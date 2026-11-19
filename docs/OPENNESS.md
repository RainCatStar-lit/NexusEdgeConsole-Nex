# NEXUS Edge Console：治理、贡献与沟通

本文档列出项目的治理原则、贡献流程、沟通渠道、决策公开规则以及已知边界。详细信息参见各模块文档和仓库配置。

## 1. 治理原则

- **透明**：所有 ADR（Architecture Decision Records）记录在 `decisions/` 下，供公开查阅。对新旧决策的更动需同步更新对应文档。
- **可追溯**：每个技术决策（模块协议选择、电源器件选型、主控方案等）均有对应的 ADR，记录“为什么选 A 不选 B”的理由。
- **可修正**：欢迎基于测试数据推翻旧决策，并在 `decisions/` 新增记录或修改现有记录。PR 标题请使用 `adr: <编号> - <简短说明>` 格式。
- **分层许可**：硬件采用 CERN-OHL-P-2.0（宽松，类 Apache 2.0）或 CERN-OHL-W-2.0（弱互惠，类 LGPL）；软件/固件采用 MIT License。最终以各子目录 `LICENSE` 文件为准。

## 2. 贡献流程

### 2.1 六类参与者入口

| 参与者 | 可以做什么 | 入口 |
|---|---|---|
| **新手** | 复刻某个子模块、提交装配反馈、报告中文翻译错误 | `replication-report.md` / `docs/ASSEMBLY.md` |
| **硬件** | 评审背板电源、提出替代料、完善 BOM 备选料表 | `docs/BOM.md` / `module-proposal.md` |
| **固件** | 贡献模块 MCU 固件、ESP-SR 命令词扩展、改进心跳/降级逻辑 | `firmware/` / `docs/MODULE_PROTOCOL.md` |
| **软件** | 改进 API Gateway、模块管理服务、新增 Cline Base URL 示例 | `software/` / `examples/cline-base-url.md` |
| **文档** | 完善装配说明、添加测试用例、更新脚注引用、翻译文档 | `docs/ASSEMBLY.md` / `docs/BOM.md` / `docs/TEST_REPORT.md` |
| **测试** | 提交长时间运行与故障注入结果、填写测试报告模板 | `docs/TEST_REPORT.md` |

### 2.2 PR 流程概览

1. Fork 仓库，新建分支 `feature/<模块>/<描述>` 或 `fix/<模块>/<问题>`。
2. 按对应模板补充文档（`docs/ASSEMBLY.md`、`docs/BOM.md`、`docs/MODULE_PROTOCOL.md` 等）。
3. 如涉及代码/固件修改，确保通过 `mkdocs build` 与 `markdownlint` 检查。
4. 提交 PR，描述变更内容、涉及的 ADR 编号以及测试结果。
5. 维护者审查后合并，合并后自动触发 `mkdocs build` 发布至 GitHub Pages。

### 2.3 Issue 模板覆盖

- `module-proposal.md`：新模块提案（包含功能描述、粗略 BOM、预计成本）
- `bug-report.md`：缺陷报告（重现步骤、预期 vs 实际、环境信息）
- `replication-report.md`：复刻测试报告（实付成本、坑点、改进建议）
- `feature-request.md`：功能需求（使用场景、验收标准）

## 3. 沟通渠道

- **主要渠道**：GitHub Issues（缺陷、需求、问题） / GitHub Discussions（设计讨论、问答）
- **响应预期**：Issues 在 48 小时内获得首次回复，PR 在 1 周内获得审查意见。未承诺 SLA，但会尽力及时响应。
- **不接受的沟通方式**：私信、非仓库内的电子邮件（除个别维护者主动联系外）。所有技术讨论请在 GitHub 公开进行，以保证可追溯。
- **社区行为**：遵守 `CODE_OF_CONDUCT.md`（若已创建）或统一的开源硬件社区自律原则。冲突时以 GitHub 官方指南为准。

## 4. 决策公开规则

- **ADR 制度**：所有关键技术决策（模块识别方案、电源器件选型、主控平台等）在 `decisions/` 下记录为 Markdown 文件。每个 ADR 包含背景、选项分析、决策理由、后果及相关文档链接。
- **修正机制**：若测试数据证决策错误，可在 `decisions/` 新增修正 ADR，原 ADR 保留并标记 `superseded`。新旧 ADR 在文中通过链接互相引用。
- **公开透明**：所有 ADR 一旦接受，即对所有人开放。决策过程中的讨论记录（GitHub Discussions）将在 PR 合并后归档至 `decisions/` 对应文件。
- **变更记录**：重大变更（影响兼容性）需同步更新 `docs/ASSEMBLY.md`、`docs/BOM.md` 与 `docs/MODULE_PROTOCOL.md`，并在 README `## 参考文献` 小节更新脚注引用。

## 5. 已知边界与风险

- **当前阶段**：Pre-development（硬件研究/预开发），接口可能在 Stage 0–1 期间调整。
- **成本边界**：`¥400–550` 为目标预算值，实际随采购批次、打样费用、替代料可得性波动。最终以采购记录为准。
- **已知风险**（详见 `docs/TEST_REPORT.md` 与 `docs/MODULE_PROTOCOL.md`）：
  1. USB HUB 兼容性：不同品牌（SL2.1A / CH334）的枚举行为差异，部分需手动安装 WinUSB 驱动。
  2. 槽位限流与瞬态保护：TPS2553/SY6280 的热关断响应时间可能在突电情况下产生短暂电压波动。
  3. ESP-SR 命令词数量限制：MultiNet 支持 ≤200 条自定义命令词，上限可能受语料库质量影响。
  4. 结构公差：2020 铝型材与 3D 打印托盘的配合间隙受打印机精度与材料收缩率影响。
  5. 替代料可得性：`docs/BOM.md` 中列出的 ≥2 个替代型号可能因供应链变动而实际不可得，以实测采购为准。
- **未验证内容**：在 `docs/TEST_REPORT.md` 中标记状态，尚未通过首版 MVP 验收清单的项目不承诺可用性。

## 6. 下一步（可验证目标）

1. 冻结主控与模块接口选型，更新 `decisions/0001-module-identification.md` 与 `decisions/0003-power-switch-selection.md`（目标日期：[2025-12]）
2. 完成 API Gateway 与 Cline 接入示例，更新 `examples/cline-base-url.md`（目标日期：[2025-12]）
3. 完成首版模块协议草案与心跳/降级测试，更新 `docs/MODULE_PROTOCOL.md`（目标日期：[2025-12]）

## 7. 参考文献

- `.readme/REFERENCES.md` — 完整参考文献列表，含 URL 与访问日期
- `decisions/0001-module-identification.md` — 为什么默认选方案 A
- `decisions/0003-power-switch-selection.md` — 限流器件选型记录
- `docs/MODULE_PROTOCOL.md` — 模块协议规范，含方案 A/B 对比
- `docs/ASSEMBLY.md` — 模块装配与测试步骤
- `docs/BOM.md` — 分阶段 BOM 与替代料
- `examples/module-capability.schema.json` — 能力描述 JSON Schema
- OHAI (Open Hardware Assembly Instructions) — Open Hardware Assembly Instructions (Opulo)
- DocuBricks — Modular XML-based documentation standard for Open Source Hardware (IEEE Open)
- CERN Open Hardware Licence v2 — CERN-OHL-P / CERN-OHL-W / CERN-OHL-S
- KiCad Project Structure Best Practices — KiCad 官方 / 社区

---
> 本文档为 `README.md` 的治理与开放性摘录。详细规则、贡献模板、测试报告等请参见对应仓库目录与 `docs/` 下文档。