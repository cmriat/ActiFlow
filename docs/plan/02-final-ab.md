# 02 · 最终 A+B 设计

> 提炼自旧库 `68-Wan22_BWM_Action_Input_and_Final_AB_Design.md` 与
> `executor_tasks/2026-10-09_round9_final/03_FINAL_AB.md`。

## A · 可观测的剩余目标验证

- 双视角 RGB + proprio + 原任务目标编码 → 共享视觉编码 / 小 MLP → **每目标当前完成概率**；
  必要时两帧差分，只选一个 backbone 配方。
- 在学生实际错切/重复操作状态上训练，不只按时间推阶段。
- **可撤销**：预测器不得用"曾经完成"永久锁住已被后续动作破坏的目标。
- 训练标签可来自 sim 谓词；推理禁止谓词/未来状态/成功 flag/episode ID/计时器。
- 低置信度沿用冻结的 FIXED160；选择器记录为何切换。
- 先测 A-only 对 FULL 与 FIXED160 的完整执行、保持与额外耗时；
  **无稳定增益则部署仍保留 FIXED160**，不得以分类准确率强行宣布 A 成功。

## B · 动作条件预测：比较切换前后的后果

- 在 chunk 边界，同一冻结 actor 分别对 continue / switch 子指令提出完整候选动作 chunk；
  世界模型生成各自短程未来，**冻结完成估计器**打分：

  `score(candidate) = predicted_remaining_goal_gain − λ · predicted_loss_of_completed_goals`

- 默认最多 2 候选、同 seed 去噪；λ 与最小优势 margin 只在 controller_val 选一次。
- 差值小 / 预测校准差 / 超时 → 回退 A-only / FIXED160。
- 禁止用真实执行后的标签回填本回合选择；真实环境只在执行已选动作后给下一轮观测。
- 世界模型未过同起点合法动作分支验收（BRANCH_VALIDATED）前，**不允许插入已有效系统**。
- 候选生成/预测的额外 actor 查询与 GPU 秒全部计成本；增加推理预算的对照不可省。

## 同平台联合的最小桥接（条件任务）

- 14D RoboTwin 模型与 7D Cosmos/LIBERO actor **动作空间不同，不能直接连**。
- 前置：14D 模型 val 已学会利用动作，且 A-only 有收益（或明显缩小 oracle 差距）。
- 做法：复制 BWM 结构为 `action_dim=7`，迁移本轮 14D 的 Wan DiT 差分，新 7D encoder 随机初始化；
  在 LIBERO train 角色 RGB 双视角、真实 7D OSC 命令上 SFT；7D 有独立 train-only stats 与分支验收。
- 联合世界模型用 **25 总帧 = 9 条件 + 16 未来**（对齐 Cosmos 每 chunk 16 动作），双视角都预测。
- 桥接失败或 A 无收益 → 停止联合扩张，写明 "A+B 联合主张未成立"。

## 最终消融表（同一冻结 actor、同一根状态）

1. FULL
2. FIXED160（最强已证基线）
3. A-only
4. A+B（仅 F08 桥接验收后）
5. 若 A+B 胜出：等额 actor 查询/无世界模型的计算对照 + B 机制消融（错误动作/固定输出）

成功增量、保持、推理耗时分别报告。A-only 与 A+B 持平 → 选 A-only；FIXED160 更好 → 交付 FIXED160。
