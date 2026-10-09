# control · 控制器层

同一冻结 actor、同一根状态下的候选：

1. `fixed160/` — FIXED160，最强已证基线，**部署回退**。
2. `a_only/` — 用 A（完成估计）替代时钟决定切换；无稳定增益则不替换基线。
3. `ab_joint/` — A+B 联合（仅当世界模型 BRANCH_VALIDATED 且 A-only 有收益）：
   chunk 边界由同一冻结 actor 提出 continue/switch 两个候选动作 chunk，
   世界模型生成各自短程未来，冻结完成估计器打分
   `score = predicted_remaining_goal_gain − λ · predicted_loss_of_completed_goals`，
   margin/λ 只在 controller_val 选一次；差异小、校准差或超时一律回退。

最终消融表与选择规则见 `docs/plan/02-final-ab.md`。
