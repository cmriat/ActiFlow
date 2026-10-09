# docs/plan · 方案文档索引

本目录是 ActiFlow 的实现依据，提炼自内部研究计划（`AcitFlow/docs/plan` 的 v32 → Round9 Final 链，
最终 A+B 设计见旧库 `68-Wan22_BWM_Action_Input_and_Final_AB_Design.md` 与
`executor_tasks/2026-10-09_round9_final/` 任务包）。

| 文档 | 内容 |
|---|---|
| [00-overview.md](00-overview.md) | 总览：RSI + embodied WAM 主线、A/B 定义、交付边界 |
| [01-wan-action-model.md](01-wan-action-model.md) | 必做：Wan2.2 + BWM 结构的动作输入世界模型（结构/数据/SFT 阶梯/验收） |
| [02-final-ab.md](02-final-ab.md) | 最终 A+B：完成验证、动作后果预测、条件桥接、消融表 |
| [03-tasks-and-acceptance.md](03-tasks-and-acceptance.md) | F00–F11 任务与验收要点 |

规则：代码迁入以本目录为唯一方案依据；方案变更先改这里再动代码。
