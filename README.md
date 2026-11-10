# NEXUS Edge Console [pre-develpement]

**A Modular Desktop AI Gateway & Edge Control Platform**

一个面向个人开发者、AI 工具用户和 DIY 爱好者的模块化桌面边缘控制平台。

NEXUS Edge Console 将 Linux 计算主控、云端 AI API 网关、语音交互、设备控制和可插拔功能模块整合在一个紧凑的桌面设备中。

它不是一台用于本地运行大型语言模型的计算服务器，而是一个以云端 AI 为主要推理能力、以本地 Linux 主控为服务中枢、以模块化硬件为扩展方式的个人 AI 控制终端。

---

## 1. 项目介绍

随着 AI 编程工具和云端大模型 API 的普及，用户需要管理越来越多的模型服务、API Key、调用渠道和外围设备。

NEXUS Edge Console 希望将这些能力整合到一台独立的桌面设备中。

它负责：

- 统一管理 OpenAI 兼容 API 服务。
- 为 Cline 等 AI 编程工具提供统一 API 接入端点。
- 管理 API 调用、Token 消耗、响应时间和成本。
- 通过本地语音唤醒和固定指令控制设备。
- 通过云端 LLM 解析复杂自然语言意图。
- 使用可插拔模块扩展传感器、红外、继电器和其他外围功能。
- 通过小型显示屏实时展示系统运行状态。

**核心理念：计算集中、功能分散、模块可替换、系统可持续扩展。**

## 2. 项目目标

NEXUS Edge Console 围绕五个核心方向构建。

| 方向 | 目标 |
|---|---|
| AI Gateway | 统一管理云端模型 API |
| Edge Computing | 本地运行轻量 Linux 服务 |
| Modular Hardware | 可插拔、可替换的异构功能模块 |
| Voice Control | 本地语音唤醒与设备控制 |
| System Dashboard | 实时展示 API 和硬件运行状态 |

项目不追求在低成本硬件上运行大型语言模型，也不追求通过大量低性能 MCU 堆叠实现集群算力。

本地硬件主要负责服务编排、设备控制、数据管理和通信。

## 3. 硬件架构

### 3.1 主控平台

主控采用低功耗 Linux 单板计算机，负责运行整个系统的软件服务。

首选平台：

- Orange Pi Zero 3（H618，2GB）
- KICKPI K2B（H618，2GB）

候选平台根据采购价格、存储配置、接口资源和 Linux 支持情况确定。

基本要求：

- 2GB RAM 起步。
- 有线以太网。
- Debian / Ubuntu / Armbian。
- 支持 Docker。
- USB Host 扩展。
- GPIO、SPI、UART 等外围接口。

ESP32 系列 MCU 不作为系统主控，也不承担 API Gateway 服务。

### 3.2 模块化背板

采用 USB 2.0 HUB 作为模块通信总线，独立设计电源管理背板。

首版采用四槽设计：

- USB HUB 负责数据通信。
- 每个槽位独立电源控制。
- 每个槽位具备独立限流或负载开关。
- 电源入口具有过流和瞬态保护。
- 每个槽位配置局部储能电容和去耦电容。
- 支持模块在位检测。
- 每个槽位具有独立状态指示灯。

候选 USB HUB 芯片：

- SL2.1A
- CH334S / CH334R

候选电源管理器件：

- CH217K
- TPS2553
- SY6280

USB 数据通道和槽位电源管理相互独立。

首版实现机械快拆、模块识别和故障降级，不宣称支持完整带电热插拔。

### 3.3 功能模块

采用统一尺寸的模块卡匣，支持异构功能模块。

规划中的模块包括：

| 模块 | 主要功能 | 候选硬件 |
|---|---|---|
| Voice Module | 语音唤醒、命令识别 | ESP32-S3 |
| Sensor Module | 温湿度、环境数据采集 | CH32V003 / ESP32-C3 |
| I/O Module | GPIO、UART、I²C | CH32V003 / CH32V203 |
| IR Module | 红外学习与发射 | MCU + IR Receiver / LED |
| Relay Module | 外部设备控制 | MCU + 继电器驱动 |
| Display Module | 系统状态展示 | ST7735S / ILI9341 |

以上为规划中的模块类型，具体硬件型号可根据成本和接口需求调整。

### 3.4 机械结构

采用桌面 NAS / 硬盘托架式设计。

初步结构方案：

- 2020 铝型材框架。
- 黑色亚克力或喷砂铝前面板。
- 3D 打印模块托盘。
- 独立导轨和定位结构。
- 四个模块槽位。
- 前面板双色 LED 状态指示。
- 后面板集中布置电源、网络和调试接口。

单模块托盘初始设计尺寸：

`70 × 100 × 12 mm`

尺寸后续根据 PCB、连接器和结构公差调整。

每个模块托盘具有独立双色 LED：

| 状态 | 指示 |
|---|---|
| 在线、正常供电 | 绿色 |
| 心跳正常 | 绿色闪烁 |
| 模块故障 | 红色 |
| 模块离线 | 熄灭 |

### 3.5 状态显示屏

规划采用 SPI TFT 小屏。

候选型号：

- ST7735S：1.8 英寸，128 × 160。
- ILI9341：2.4 英寸，240 × 320。

屏幕用于显示结构化系统状态：

- 当前 API 服务状态。
- 模型和渠道。
- API 响应时间。
- 输入与输出 Token。
- 调用成本。
- 上下文摘要。
- 模块在线数量。
- 语音系统状态。

屏幕仅负责仪表盘显示，不运行浏览器或 WebGL。

## 4. 软件架构

主控运行 Debian 系列 Linux，并通过 Docker Compose 管理服务。

### 4.1 核心软件

| 软件 | 职责 |
|---|---|
| Debian / Armbian | 操作系统 |
| Docker Compose | 容器编排 |
| Caddy | 反向代理与 TLS |
| New API | API 网关 |
| SQLite | 轻量数据存储 |
| Python / Go | 模块管理、数据处理 |
| Pillow / SPI 驱动 | 屏幕显示 |

LiteLLM 可作为后续模型路由扩展，不要求首版同时部署多个 API 网关。

### 4.2 API Gateway

统一管理外部模型 API。

主要能力：

- API Key 管理。
- 多模型渠道管理。
- 请求转发。
- 使用额度管理。
- 调用日志。
- Token 统计。
- 成本统计。
- OpenAI 兼容接口。

目标是为 Cline 和其他兼容 OpenAI API 的工具提供统一的 Base URL。

### 4.3 模块管理服务

Linux 主控负责发现、注册和管理所有 USB 模块。

统一模块信息：

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
  ]
}
```

模块管理服务负责：

- USB 设备枚举。
- 模块身份识别。
- 模块注册。
- 心跳监测。
- 状态更新。
- 故障检测。
- 服务降级。
- 模块重新插入后的自动恢复。

管理 MCU 不作为 Linux 主控的单点故障依赖。Linux 主控保留基础 USB 设备管理和系统服务能力。

### 4.4 语音控制

语音模块采用 ESP32-S3 + ESP-SR。

处理分为三个层级：

1. 本地唤醒词检测。
2. 本地固定命令识别。
3. 云端 LLM 复杂意图解析。

例如：

```text
用户：NEXUS，打开桌面灯。

ESP32-S3：
唤醒词检测 + 固定命令识别。

Linux：
解析设备 ID 和动作。

设备控制服务：
检查权限与白名单。

执行：
打开桌面灯。
```

对于复杂指令：

```text
用户：帮我把办公室调整到适合工作的状态。

Linux：
调用云端 LLM 解析意图。

LLM：
返回结构化设备动作。

Linux：
执行安全检查和权限验证。

设备控制服务：
执行允许的操作。
```

云端模型不得绕过本地安全策略直接控制继电器或其他设备。

### 4.5 系统仪表盘

API 网关和模块管理服务向统一 Telemetry Service 提供结构化事件。

屏幕渲染服务订阅事件，展示：

- API 调用信息。
- 模型名称。
- 响应时间。
- Token 数量。
- 调用成本。
- 上下文摘要。
- 模块状态。
- 语音状态。

不将模型内部推理过程或不可观测的上下文信息伪装成真实监控数据。

## 5. 项目结构

规划采用以下仓库结构：

```text
NEXUS-Edge-Console/
│
├── hardware/
│   ├── main-controller/
│   ├── usb-backplane/
│   ├── power-management/
│   └── module-cartridge/
│
├── firmware/
│   ├── module-manager/
│   ├── voice-module/
│   ├── sensor-module/
│   └── io-module/
│
├── software/
│   ├── api-gateway/
│   ├── module-manager/
│   ├── device-control/
│   ├── telemetry/
│   └── display/
│
├── mechanical/
│   ├── enclosure/
│   ├── tray/
│   ├── rail/
│   └── panel/
│
├── docker/
│   ├── compose.yaml
│   └── configs/
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── HARDWARE.md
│   ├── SOFTWARE.md
│   ├── MODULE_PROTOCOL.md
│   ├── BOM.md
│   ├── ASSEMBLY.md
│   └── TEST_REPORT.md
│
├── examples/
│
├── LICENSE
└── README.md
```

## 6. 开发路线

项目分阶段推进，先建立可工作的核心系统，再增加模块和外观。

| 阶段 | 主要工作 | 验收目标 |
|---|---|---|
| Stage 0 | 主控和器件选型 | BOM 与接口冻结 |
| Stage 1 | Linux 主控与 API Gateway | Docker 服务稳定运行 |
| Stage 2 | USB HUB 与独立电源背板 | 四槽通信和供电 |
| Stage 3 | 模块协议与管理服务 | 模块识别、降级、恢复 |
| Stage 4 | 语音模块 | 本地唤醒与命令控制 |
| Stage 5 | SPI 仪表盘 | API 与模块状态显示 |
| Stage 6 | 机械结构 | 四槽卡匣式机箱 |
| Stage 7 | 系统集成 | 长时间运行与故障测试 |
| Stage 8 | 开源发布 | BOM、工程文件和装配文档 |

### 首版 MVP

首版仅安装两个功能模块，另外两个槽位保留。

**模块 01：Voice Module**
- ESP32-S3。
- ESP-SR。
- 麦克风输入。
- 本地唤醒和固定命令识别。

**模块 02：Sensor / I/O Module**
- CH32 系列 MCU。
- 温湿度传感器。
- 红外收发。
- GPIO 扩展。

系统必须完成以下测试：

- API Gateway 连续运行。
- Cline 通过统一 API 地址接入。
- USB 模块正常枚举。
- 模块拔出后服务降级。
- 模块插回后自动恢复。
- LED 正确反映模块状态。
- 屏幕显示真实 API 调用信息。
- 语音完成至少五个固定命令。

## 7. 成本目标

首版采用预算敏感的 DIY 方案。

| 项目 | 目标预算 |
|---|---:|
| Linux 主控 | ¥120–160 |
| USB HUB 与电源背板 | ¥35–60 |
| MCU 功能模块 | ¥20–40 |
| 语音模块 | ¥35–60 |
| SPI 屏幕 | ¥8–20 |
| 存储、电源和辅料 | ¥50–80 |
| 基础结构 | ¥20–40 |
| **功能原型目标** | **¥400–550** |

完整演示机的铝型材、面板、托盘和导光结构单独核算，预计额外增加约 ¥100–250。

以上是目标预算，实际成本取决于主控、PCB 打样和采购渠道。

## 8. 设计原则

NEXUS Edge Console 遵循以下原则：

1. **Linux First**：所有核心服务运行在 Linux 主控上。
2. **Cloud AI First**：复杂推理交给云端，本地负责控制和调度。
3. **Modular by Design**：模块统一接口，支持异构扩展。
4. **Independent Power Management**：模块供电独立于 USB 数据 HUB。
5. **Fault Tolerance**：单个模块故障不影响核心 API 服务。
6. **Simple Hardware**：优先采用成熟器件和可复刻开源工程。
7. **Observable System**：所有关键状态可监测、可记录、可展示。
8. **Reproducible Build**：硬件、固件、软件和装配资料均可复现。

## 9. 开源与社区

项目计划开放：

- PCB 原理图和 Gerber。
- 机械结构 CAD / STL。
- MCU 固件。
- Linux 管理软件。
- Docker 部署配置。
- BOM 与采购替代清单。
- 装配说明和测试报告。

硬件许可证计划采用 CERN-OHL-P 或 CERN-OHL-W 系列，软件和固件倾向 MIT License。最终以各子项目的 LICENSE 文件为准。

欢迎 DIY 爱好者、嵌入式开发者和 AI 工具用户参与。

## 10. 项目愿景

NEXUS Edge Console 希望成为一个可以放在桌面上、长期运行、持续扩展的个人 AI 服务与设备控制中心。

它不需要拥有庞大的本地算力，也不需要复杂昂贵的硬件。

一块低功耗 Linux 主控，配合统一的模块接口、可靠的供电背板和开放的软件生态，就可以将云端 AI、桌面设备和个人自动化工作流连接起来。

**One Core. Multiple Modules. One Unified AI Gateway.**

---

**Project:** NEXUS Edge Console  
**Status:** Hardware Research / Pre-development  
**Target:** DIY / Open Hardware / Personal Edge AI  
**Budget:** ¥400–550 (Prototype Target)