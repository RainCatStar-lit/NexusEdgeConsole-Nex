# NEXUS Edge Console 装配文档大纲

遵循 **OHAI (Open Hardware Assembly Instructions)** 与 **DocuBricks** 模块化结构。  
每个 brick 包含：组件清单 → 分步装配指令 → 测试/校准说明 → BOM 引用。

---

## 文档结构

```
docs/assembly/
├── index.md                    # 本文件：总览与导航
├── 00-prerequisites.md         # 前置条件：工具、环境、安全
├── 01-main-controller/         # 主控单元
│   ├── components.md           # 组件清单（PCB、散热、存储、跳线）
│   ├── assembly.md             # 分步装配
│   ├── test.md                 # 上电测试、启动验证
│   └── calibration.md          # 无（主控无需校准）
├── 02-usb-backplane/           # USB 背板
│   ├── components.md
│   ├── assembly.md
│   ├── test.md                 # 通电、限流、Hub 枚举测试
│   └── calibration.md          # 限流阈值校准（TPS2553 外部电阻）
├── 03-power-management/        # 电源管理模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md
│   └── calibration.md
├── 04-module-cartridge/        # 模块卡匣（通用）
│   ├── components.md
│   ├── assembly.md
│   ├── test.md                 # 机械插拔、USB 连通性
│   └── calibration.md
├── 05-voice-module/            # 语音模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md                 # 麦克风、扬声器、ESP-SR 固件烧录验证
│   └── calibration.md          # 唤醒词灵敏度、增益校准
├── 06-sensor-module/           # 传感器模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md
│   └── calibration.md          # 温湿度传感器偏移校准
├── 07-io-module/               # I/O 模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md
│   └── calibration.md
├── 08-ir-module/               # 红外模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md
│   └── calibration.md          # 红外发射功率、接收灵敏度
├── 09-relay-module/            # 继电器模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md
│   └── calibration.md
├── 10-display-module/          # 显示模块
│   ├── components.md
│   ├── assembly.md
│   ├── test.md
│   └── calibration.md          # SPI 时序、色彩/亮度
├── 11-mechanical/              # 机械结构
│   ├── 11a-enclosure/          # 机箱框架
│   ├── 11b-tray/               # 模块托盘
│   ├── 11c-rail/               # 导轨
│   └── 11d-panel/              # 面板、导光
├── 12-final-integration/       # 最终集成
│   ├── assembly.md             # 全机装配顺序
│   ├── test.md                 # 系统级功能测试清单
│   └── calibration.md          # 全链路校准
└── 13-packaging/               # 包装与发运
    ├── components.md
    ├── assembly.md
    └── test.md
```

---

## 通用 Brick 模板（每个子目录复制此结构）

### `components.md`
```markdown
# [模块名] 组件清单

## PCB 与电子元件
| 项 | 描述 | 数量 | 备选料 | 备注 |
|----|------|------|--------|------|
| 1 | PCB 主板 | 1 | - | KiCad 源文件见 `hardware/xxx/kicad/` |
| 2 | MCU (ESP32-S3-WROOM-1) | 1 | ESP32-S3-WROOM-1U | 已烧录引导程序 |
| ... | ... | ... | ... | ... |

## 机械件
| 项 | 描述 | 数量 | 材料/工艺 | 来源 |
|----|------|------|-----------|------|
| 1 | 模块外壳上盖 | 1 | 3D 打印 PLA/PETG | `mechanical/tray/upper.stl` |
| 2 | 模块外壳下盖 | 1 | 3D 打印 PLA/PETG | `mechanical/tray/lower.stl` |
| ... | ... | ... | ... | ... |

## 紧固件与辅料
| 项 | 规格 | 数量 | 备注 |
|----|------|------|------|
| 1 | M2×5 螺丝 | 4 | 尼龙或不锈钢 |
| 2 | M2.5×6 螺丝 | 2 | - |
| ... | ... | ... | ... |

## BOM 引用
完整 BOM 见 `docs/BOM.md#xxx-module`，含 LCSC/立创货号、单价、替代料。
```

### `assembly.md`
```markdown
# [模块名] 分步装配指令

> ⚠️ 开始前请阅读 `00-prerequisites.md` 安全须知。

## 步骤 1：PCB 检查
- 目视检查 PCB 无短路、焊桥、元件漏装
- 万用表测量电源网对地电阻 > 1 MΩ

## 步骤 2：SMT 元件焊接（如适用）
- 按 `hardware/xxx/kicad/bom.csv` 顺序贴装
- 关键器件（MCU、稳压芯片）建议热风枪/回流焊

## 步骤 3：THT/连接器焊接
- USB Type-C 公头 / 排针 / 排母
- 注意方向、共面度

## 步骤 4：机械组装
1. 将 PCB 放入下盖定位柱
2. 装入侧向导轨锁扣
3. 盖上盖，锁付 M2×5 螺丝 ×4
4. 检查 USB 接口伸出长度、对中度

## 步骤 5：标签与记录
- 贴上模块序列号标签（格式 `NEXUS-<type>-<YYYYMMDD>-<XXXX>`）
- 记录至 `assembly-log.csv`
```

### `test.md`
```markdown
# [模块名] 测试与验收

## 上电前检查
- [ ] 电源网对地电阻 > 1 MΩ
- [ ] 关键信号网无短路
- [ ] 机械结构无干涉

## 功能测试步骤
1. **通电测试**：接入 5V/1A 限流电源，测量各电压轨
   - 5V 轨：4.9–5.1 V
   - 3.3V 轨：3.2–3.4 V
   - 电流 < 额定最大值
2. **USB 枚举**：插入 Linux 主控，`lsusb` 可见 VID/PID
3. **固件烧录**：按 `firmware/xxx/README.md` 烧录最新固件
4. **能力自检**：模块上报 JSON，主控验证通过
5. **专项测试**（各模块特有，见下）

## 专项测试（示例：Voice Module）
- [ ] 麦克风采样率 16 kHz，单声道，16-bit
- [ ] 唤醒词 "NEXUS" 10 次触发 ≥ 9 次
- [ ] 5 条固定命令各测 5 次，识别率 100%
- [ ] 噪声环境（60 dB 白噪声）下唤醒率 ≥ 80%

## 记录模板
| 测试项 | 预期结果 | 实测结果 | 通过/失败 | 备注 |
|--------|----------|----------|-----------|------|
| 5V 电压 | 4.9–5.1 V | 5.02 V | ✅ | - |
| ... | ... | ... | ... | ... |
```

### `calibration.md`（按需）
```markdown
# [模块名] 校准说明

## 需校准参数
| 参数 | 校准方法 | 目标值 | 频率 |
|------|----------|--------|------|
| 唤醒词灵敏度 | 播放标准音频，调整 `WakeNet` 阈值 | 触发率 ≥ 90% / 误触 < 1 次/天 | 出厂一次，OTA 可更新 |
| 麦克风增益 | 注入 94 dB SPL 1 kHz 正弦波，调整 ADC 增益 | RMS = -26 dBFS ±1 dB | 出厂一次 |
| 限流阈值 (TPS2553) | 调节外部电阻 R_ILIM，实测触发电流 | 500 mA ±5% / 1 A ±5% | PCB 版本变更时 |
| 温湿度偏移 | 对比标准温湿度计，写入校准系数 | ±0.3°C / ±2% RH | 出厂一次，建议每年复校 |

## 校准记录
每次校准生成 `calibration-report-<module_id>-<date>.json`，归档至 `logs/calibration/`。
```

---

## 交叉引用索引

| 文档 | 位置 | 用途 |
|------|------|------|
| `docs/BOM.md` | 全局 | 统一 BOM、替代料、采购链接 |
| `docs/MODULE_PROTOCOL.md` | 协议层 | 模块能力 JSON、电源规格、心跳规则 |
| `hardware/*/kicad/` | 设计源文件 | 原理图、PCB、封装、BOM CSV |
| `hardware/*/production/` | 制造文件 | Gerber、坐标文件、贴片图、钢网文件 |
| `firmware/*/README.md` | 固件层 | 编译、烧录、配置、测试固件 |
| `examples/module-capability.schema.json` | 验证层 | 能力描述 JSON Schema 自动校验 |

---

## 版本控制与变更管理

- 装配文档随硬件版本发布，采用语义化版本 `v<PCB版本>.<装配版本>`（如 `v1.0.2`）
- 变更记录在 `docs/assembly/CHANGELOG.md`
- 重大变更（影响兼容性）需同步更新 `decisions/` 对应 ADR

---

## 贡献指南

1. Fork 仓库，新建分支 `assembly/<module>-<description>`
2. 按上述模板补充对应模块的 4 个文件
3. 本地实测通过后提交 PR，CI 自动跑 `markdownlint` + 交叉引用检查
4. 合并后自动触发 `mkdocs build` 发布至 GitHub Pages

---

## 相关标准

- **OHAI** — Open Hardware Assembly Instructions (Opulo, mkDocs-based)
- **DocuBricks** — Modular XML-based documentation standard (IEEE Open)
- **Open Hardware Documentation Template** — Root README + per-dir `_readme.md` + Build/Docs/Software separation (Faez, S.)