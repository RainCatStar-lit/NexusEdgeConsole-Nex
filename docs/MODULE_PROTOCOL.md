# NEXUS 模块协议规范

版本：0.1.0-draft  
状态：草案，供 Stage 0/1 评审  
维护者：NEXUS 核心团队

---

## 1. 概述

本文档定义 NEXUS Edge Console 与功能模块之间的通信协议、身份识别机制、心跳与降级规则、电源管理接口。  
目标：让任意符合协议的模块"插即用"，无需修改主控软件。

---

## 2. 身份识别方案对比

| 维度 | 方案 A：USB VID/PID + 字符串描述符 | 方案 B：Microsoft OS 2.0 Descriptors (WCID) |
|------|-----------------------------------|---------------------------------------------|
| 适用场景 | Linux 主控、模块数量可控、无需 Windows 免驱 | 需在 Windows 8.1+ 免 INF 自动加载 WinUSB |
| 实现复杂度 | 低（标准 USB 描述符） | 中（需额外 OS 描述符、复合设备每功能独立 compatible ID） |
| Linux 支持 | 原生 `lsusb`、`udev` 规则即可识别 | 需额外解析 OS 描述符 |
| Windows 支持 | 需手动安装 INF 或 WinUSB 驱动 | 免驱，自动绑定 WinUSB |
| 推荐用途 | **默认方案**（本项目 Linux 主控为主） | Windows 兼容扩展，预留实现 |

**决策记录**：见 `decisions/0001-module-identification.md`。

---

## 3. 方案 A 详细规范（默认）

### 3.1 USB 描述符要求

每个模块必须实现以下标准 USB 描述符：

| 描述符 | 关键字段 | 说明 |
|--------|----------|------|
| Device Descriptor | `idVendor` (VID) | 申请或使用实验性 VID（如 0x1209） |
| | `idProduct` (PID) | 模块类型唯一 PID，见下表 |
| | `iManufacturer` | "NEXUS" |
| | `iProduct` | 模块商品名，如 "NEXUS Voice Module" |
| | `iSerialNumber` | 唯一序列号，建议格式 `NEXUS-<type>-<YYYYMMDD>-<XXXX>` |
| String Descriptor (Index 0xEE) | Microsoft OS 2.0 Signature | 仅方案 B 需要，方案 A 可省略 |

### 3.2 预分配 PID 表（示例）

| 模块类型 | PID (Hex) | 备注 |
|----------|-----------|------|
| Voice Module | 0x0001 | ESP32-S3 + ESP-SR |
| Sensor Module | 0x0002 | CH32V003 / ESP32-C3 |
| I/O Module | 0x0003 | CH32V003 / CH32V203 |
| IR Module | 0x0004 | MCU + IR Rx/Tx |
| Relay Module | 0x0005 | MCU + Relay Driver |
| Display Module | 0x0006 | ST7735S / ILI9341 |

> 正式 VID/PID 分配后更新此表，保持向后兼容。

### 3.3 能力描述 JSON（设备端上报）

模块枚举后，通过 USB 串口（CDC-ACM）或自定义 HID 报告发送 JSON：

```json
{
  "module_id": "voice_001",
  "module_type": "voice",
  "hardware_version": "1.0",
  "firmware_version": "0.1.0",
  "status": "online",
  "heartbeat_ms": 1000,
  "capabilities": [
    "wake_word",
    "speech_command"
  ],
  "capability_details": {
    "wake_word": { "model": "WakeNet8", "sensitivity": 0.5 },
    "speech_command": { "model": "MultiNet", "max_commands": 200, "language": "zh-CN" }
  },
  "power": {
    "voltage_mv": 5000,
    "max_current_ma": 300,
    "inrush_current_ma": 800
  }
}
```

**字段说明**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| module_id | string | 是 | 全局唯一，建议 `<type>_<序号>` |
| module_type | string | 是 | 对应 PID 表中的类型名 |
| hardware_version | string | 是 | 主版本.次版本 |
| firmware_version | string | 是 | 语义化版本 |
| status | enum | 是 | `online` / `offline` / `degraded` / `booting` |
| heartbeat_ms | integer | 是 | 心跳上报周期，默认 1000 |
| capabilities | string[] | 是 | 标准能力标识符列表 |
| capability_details | object | 否 | 能力细节，供上层调度参考 |
| power | object | 是 | 供电需求，供背板限流策略参考 |

---

## 4. 方案 B 扩展规范（Windows 免驱）

仅当模块需在 Windows 主控上免驱运行时实现。

### 4.1 Microsoft OS 2.0 描述符集

| 描述符 | 关键内容 |
|--------|----------|
| OS String Descriptor (Index 0xEE) | `MSFT100` 签名 + Vendor Code |
| Extended Compat ID Descriptor | 每个功能接口的 `Compatible ID` = `WINUSB` |
| Extended Properties Descriptor | 可选：设备接口 GUID、友好名称等 |

### 4.2 复合设备处理

模块若为复合设备（如 CDC-ACM + HID），**每个功能接口必须独立设置** `Compatible ID = WINUSB`，否则 Windows 仅对第一个接口自动加载 WinUSB。

### 4.3 共存策略

- 固件同时包含方案 A 与方案 B 描述符
- Linux 主控忽略 OS 描述符，仅用 VID/PID + 字符串识别
- Windows 主控检测到 OS 2.0 描述符 → 自动绑定 WinUSB

---

## 5. 心跳与状态机

### 5.1 心跳机制

- 周期：`heartbeat_ms`（默认 1000 ms）
- 传输：USB 串口文本行 `HEARTBEAT\n` 或 JSON `{"event":"heartbeat"}`
- 主控连续丢失 **3 次** 心跳 → 判定 `offline`
- 模块重新插入 → 主控检测到 USB 枚举 → 请求能力描述 → 恢复 `online`

### 5.2 状态迁移图

```
booting --枚举成功/能力描述上报--> online
online --连续 3 次无心跳--> offline
online --模块上报错误/异常--> degraded
degraded --错误恢复/重新枚举--> online
offline --USB 重新枚举--> booting
```

### 5.3 降级策略

| 故障现象 | 主控动作 | 用户可见 |
|----------|----------|----------|
| 单模块心跳丢失 | 标记 `offline`，禁用该模块能力，保留其它模块 | 对应槽位 LED 红灯 |
| 模块上报 `degraded` | 仅禁用故障能力，其它能力保留 | LED 红灯闪烁 |
| USB Hub 故障 | 所有模块 `offline`，核心 API 服务不受影响 | 全部槽位 LED 熄灭 |
| 主控重启 | 保留模块注册信息，启动后自动重连 | 启动期间 LED 绿闪 |

**关键原则**：Linux 主控保留基础 USB 设备管理能力，**模块管理 MCU 不作为单点故障依赖**。

---

## 6. 电源与热插拔接口

### 6.1 槽位电源规格

| 参数 | 典型值 | 备注 |
|------|--------|------|
| 电压 | 5.0 V ±5% | 来自背板统一 5V 轨 |
| 连续电流限流 | 500 mA / 1 A 可配 | TPS2553 可调，SY6280 固定 1 A/2 A 档位 |
| 浪涌电流 | ≤2 A / 100 µs | 局部储能电容 100 µF + 10 µF |
| 过压保护 | 5.5 V TVS | 单向 TVS 二极管 |
| 反向电压保护 | 是 | TPS2553 内部比较器 / SY6280 内部 MOSFET |

### 6.2 热插拔行为（首版目标）

- **机械快拆**：模块卡匣导轨 + 定位柱，插拔力 < 15 N
- **电源先断后通**：USB 连接器长度分级（电源触点最长），或由背板 MCU 控制限流开关时序
- **数据通道独立**：USB 数据线不经过限流开关，故障隔离粒度到单槽
- **不宣称完整带电热插拔**：首版实现"带电插拔不损坏硬件、软件自动恢复"，不保证零中断

### 6.3 限流器件选型对比（见 `decisions/0003-power-switch-selection.md`）

| 器件 | 可调限流 | 热关断 | 反向电压保护 | 成本 | 推荐场景 |
|------|----------|--------|--------------|------|----------|
| TPS2553 | 是（外部电阻） | 是 | 是（外部 MOSFET） | ~¥3.5 | 需精确限流、高可靠 |
| SY6280AAC | 固定档位 | 是 | 内部集成 | ~¥1.2 | 成本敏感、标准 1A/2A 档位足够 |
| CH217K | 否 | 是 | 否 | ~¥0.8 | 极低成本、仅基础过流保护 |

---

## 7. 模块能力标识符注册表

| 标识符 | 含义 | 典型模块 |
|--------|------|----------|
| `wake_word` | 本地唤醒词检测 | Voice Module |
| `speech_command` | 本地固定命令识别 | Voice Module |
| `sensor_temperature` | 温度采集 | Sensor Module |
| `sensor_humidity` | 湿度采集 | Sensor Module |
| `sensor_pressure` | 气压采集 | Sensor Module |
| `gpio_input` | GPIO 输入 | I/O Module |
| `gpio_output` | GPIO 输出 | I/O Module |
| `uart_bridge` | UART 透传 | I/O Module |
| `i2c_bridge` | I²C 透传 | I/O Module |
| `ir_learn` | 红外学习 | IR Module |
| `ir_transmit` | 红外发射 | IR Module |
| `relay_control` | 继电器控制 | Relay Module |
| `display_render` | 仪表盘渲染 | Display Module |

> 新能力提交 PR 时需同步更新此表与 `examples/module-capability.schema.json`。

---

## 8. JSON Schema（能力描述验证）

见 `examples/module-capability.schema.json`，主控服务启动时加载并验证每个模块上报的 JSON。

---

## 9. 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 0.1.0-draft | 2025-11-18 | 初始草案，含方案 A/B 对比、心跳、电源、能力注册表 |

---

## 10. 相关文档

- `decisions/0001-module-identification.md` — 为什么默认选方案 A
- `decisions/0003-power-switch-selection.md` — 限流器件选型记录
- `examples/module-capability.schema.json` — 能力描述 JSON Schema
- `docs/ASSEMBLY.md` — 模块装配与测试步骤