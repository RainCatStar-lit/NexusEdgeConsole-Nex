# ADR 0001: 模块身份识别方案选择

**状态**：Accepted  
**日期**：2025-11-18  
**决策者**：NEXUS 核心团队

---

## 背景

NEXUS Edge Console 需要一种机制，让 Linux 主控在模块插入 USB 背板时自动识别模块类型、版本、能力，并建立通信通道。  
主要约束：
- 主控运行 Linux（Debian/Armbian），无 Windows 原生需求
- 模块数量可控（首版 4 槽，未来 ≤8 槽）
- 希望固件实现简单、调试容易、工具链依赖少
- 预留 Windows 兼容扩展路径

---

## 备选方案

| 方案 | 描述 |
|------|------|
| **A: USB VID/PID + 标准字符串描述符** | 使用标准 USB Device Descriptor 的 `idVendor`、`idProduct`、`iManufacturer`、`iProduct`、`iSerialNumber`。Linux 通过 `lsusb`、`udev` 规则、`pyudev` 直接识别。 |
| **B: Microsoft OS 2.0 Descriptors (WCID)** | 实现 OS String Descriptor (Index 0xEE)、Extended Compat ID Descriptor、Extended Properties Descriptor。Windows 8.1+ 自动绑定 WinUSB，免 INF。复合设备需每接口独立设置 Compatible ID。 |

---

## 决策

**选择方案 A 作为默认实现，方案 B 作为可选扩展预留。**

---

## 理由

1. **Linux 原生支持最好**  
   `udev` 规则可直接匹配 VID/PID，`pyudev`/`libudev` 无需额外解析即可获取字符串描述符。开发调试时 `lsusb -v` 即可见所有信息。

2. **固件实现最简单**  
   标准 USB 设备描述符是所有 USB 设备必须实现的；方案 B 需额外实现 Vendor Request `0xEE`、OS String Descriptor、两类 Extended Descriptor，代码量约 2-3 倍。

3. **模块数量可控，无需自动驱动安装**  
   首版仅 6 种模块类型，手工维护 `udev` 规则与 PID 映射表成本极低。Windows 免驱非当前阶段刚需。

4. **向后兼容性强**  
   方案 A 仅依赖 USB 2.0 规范核心部分，任何 USB Host 栈均支持。方案 B 依赖 Microsoft 专有扩展，非 Windows 平台需额外解析库。

5. **预留扩展路径清晰**  
   固件可同时烧录方案 A 与方案 B 描述符（互不冲突），Linux 忽略 OS 描述符，Windows 检测到即自动生效。后续如需 Windows 支持，仅需在固件开启方案 B 编译选项。

---

## 后果

### 正面
- 固件开发周期缩短 ~1 周
- 调试工具链零依赖（`lsusb`、`dmesg`、`udevadm`）
- PID 映射表单一数据源，文档与代码同步维护成本低

### 负面 / 需接受
- Windows 用户需手动安装 WinUSB 驱动（通过 Zadig 或 INF），或由上层应用打包驱动
- 无法利用 Windows 自动弹出设备友好名称等 UI 特性

### 缓解措施
- 在 `docs/MODULE_PROTOCOL.md` 完整记录方案 B 实现细节，作为未来扩展参考
- 提供 `scripts/generate_udev_rules.py` 自动从 PID 表生成 `99-nexus-modules.rules`
- 预留固件编译宏 `CONFIG_MS_OS_20_DESCRIPTORS`，默认关闭

---

## 替代方案评估记录

| 标准 | 权重 | 方案 A 得分 | 方案 B 得分 |
|------|------|-------------|-------------|
| Linux 原生支持 | 30% | 10 | 6 |
| 固件实现复杂度 | 25% | 10 | 4 |
| 调试便利性 | 15% | 10 | 5 |
| Windows 免驱 | 10% | 3 | 10 |
| 扩展灵活性 | 10% | 8 | 9 |
| 文档/社区成熟度 | 10% | 9 | 6 |
| **加权总分** | **100%** | **9.1** | **6.1** |

---

## 确认

- [x] 方案 A 已在 `docs/MODULE_PROTOCOL.md` 第 3 节完整规范
- [x] 方案 B 已在同文档第 4 节作为扩展记录
- [x] PID 分配表已在同文档第 3.2 节预留
- [x] 固件模板 `firmware/module-manager/` 已包含方案 A 基础实现框架

---

## 相关文档

- `docs/MODULE_PROTOCOL.md` — 完整协议规范
- `decisions/0003-power-switch-selection.md` — 电源限流器件选型（同期决策）
- `examples/module-capability.schema.json` — 能力描述 JSON Schema