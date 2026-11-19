# NEXUS Edge Console - 完整参考文献列表

最后更新：2025-11-18  
维护：自动同步自 README 脚注与 docs/ 各技术文档

---

## 1. USB 与 Windows 兼容

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [1] | Generating USB Device Descriptors for Windows | Microsoft Learn | 2025-11-18 | Microsoft OS 2.0 Descriptors、WCID、WinUSB 免 INF 加载机制概览 |
| [2] | Microsoft OS 2.0 Descriptors Specification | Microsoft | 2025-11-18 | OS 字符串描述符、扩展兼容 ID、复合设备每功能独立 compatible ID 设置方式 |
| [3] | WCID Devices (Windows Compatible ID) | GitHub (pbatard/libwdi) | 2025-11-18 | 实现参考、Zadig 工具链、INF 示例 |

---

## 2. 语音识别 (ESP-SR / ESP32-S3)

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [4] | ESP-SR Speech Recognition Framework | Espressif 官方文档 | 2025-11-18 | WakeNet 唤醒词、MultiNet 命令词（≤200 条）、16 kHz/16-bit/mono 输入要求 |
| [5] | 实战派 ESP32-S3 教程：语音唤醒与语音命令 | 立创开发板 / LCSC | 2025-11-18 | 完整 WakeNet + MultiNet 例程、I²S 采样率与模型输入格式对应关系 |
| [6] | ESP32-S3 Datasheet | Espressif | 2025-11-18 | 引脚定义、音频 I²S 外设、内存布局、功耗参数 |

---

## 3. 电源管理与热插拔

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [7] | TPS2553 Datasheet | Texas Instruments | 2025-11-18 | 可调限流（外部电阻）、热关断、反向电压比较器、USB 端口保护典型应用 |
| [8] | SY6280AAC Datasheet | Silergy | 2025-11-18 | 过流/短路/过温全保护、自动恢复、固定档位限流、低成本替代方案 |
| [9] | CH217K Datasheet | Nanjing ChipHope | 2025-11-18 | 基础过流保护、极低成本、无反向电压保护 |
| [10] | USB 2.0 Specification - Chapter 7.2.1 (Inrush Current) | USB-IF | 2025-11-18 | 热插拔浪涌电流限制、去耦电容要求 |

---

## 4. 开源硬件许可证

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [11] | CERN Open Hardware Licence v2 | CERN | 2025-11-18 | CERN-OHL-P (宽松/类 Apache 2.0)、CERN-OHL-W (弱互惠/类 LGPL)、CERN-OHL-S (强互惠/类 GPL) 完整文本与 FAQ |
| [12] | CERN-OHL v2 FAQ | CERN | 2025-11-18 | 许可证选择指南、兼容性矩阵、商业使用界定 |

---

## 5. 仓库结构与装配文档标准

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [13] | Open Hardware Documentation Template | S. Faez (GitHub) | 2025-11-18 | 根 README 入口、子目录 `_readme.md` 约定、Build/Docs/Software 三层分离 |
| [14] | OHAI — Open Hardware Assembly Instructions | Opulo (GitHub) | 2025-11-18 | MkDocs 组织分步装配文档、BOM 引用、装配指令、测试/校准节 |
| [15] | DocuBricks: Modular XML-based documentation standard | IEEE Open / DocuBricks | 2025-11-18 | Brick = 组件 + 装配指令 + 自定义指令(校准/测试/安全) + BOM 引用 |
| [16] | KiCad Project Structure Best Practices | KiCad 官方 / 社区 | 2025-11-18 | `kicad/`、`production/`、`mechanical/` 三级分离制造文件与设计源文件 |

---

## 6. 架构决策记录 (ADR) 参考

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [17] | Architecture Decision Records (ADR) | M. Nygard / adr.github.io | 2025-11-18 | ADR 格式模板、决策记录最佳实践 |
| [18] | Sustainable Architectural Decisions | InfoQ / Michael Keeling | 2025-11-18 | ADR 在长期项目中的演进与治理 |

---

## 7. 机械设计与制造

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [19] | 2020 铝型材标准规格 | Misumi / Bosch Rexroth | 2025-11-18 | 槽宽、螺纹、连接件选型、公差配合 |
| [20] | 3D 打印公差设计指南 | Prusa / Formlabs | 2025-11-18 | 孔径补偿、配合间隙、支撑策略、各向异性强度 |

---

## 8. 软件与固件生态

| # | 标题 | 来源 | 访问日期 | 关键内容 |
|---|------|------|----------|----------|
| [21] | New API Documentation | New API 官方 | 2025-11-18 | 多渠道管理、OpenAI 兼容接口、Docker 部署、Token/成本统计 |
| [22] | Caddy Documentation | Caddy 官方 | 2025-11-18 | 自动 HTTPS、反向代理、Docker 集成 |
| [23] | ESP-IDF Programming Guide | Espressif | 2025-11-18 | FreeRTOS 任务、USB CDC-ACM、NVS、OTA 升级 |
| [24] | CH32V003 Reference Manual | WCH | 2025-11-18 | RISC-V 核心、GPIO、UART、I²C、ADC、低功耗模式 |

---

## 维护说明

- 本列表由 `.readme/REFERENCES.md` 与 `docs/REFERENCES.md` 双向同步
- 新增引用时：在 README 正文添加脚注编号 `[n]`，同时在此文件追加对应行
- 失效链接或版本更新：PR 标题格式 `refs: update [n] - <reason>`
- 访问日期每季度复核一次，过期链接标记 `⚠️` 并寻找替代