# ActiFlow

Action-conditioned flow world models for long-horizon robot execution.

基于 **RSI（Recursive Self-Improvement）+ embodied WAM** 主线的最终 A+B 方案：

- **主交付模型**：`W(x_initial, x_history, commanded_actions) → future_video`，
  从原始 Wan2.2-TI2V-5B 初始化，复现公开 BWM 双路径动作条件结构并 SFT。
- **A · 可观测目标完成验证**：双视角 RGB + proprio + 目标编码 → 逐目标完成概率（可撤销），
  低置信度回退已冻结的 FIXED160 规则。
- **B · 动作条件后果预测**：在 chunk 边界对「继续当前子目标 / 切换下一子目标」
  各生成短程未来，由冻结完成估计器打分决定是否切换。

> 状态：仓库骨架占位中，代码按计划文档（`docs/plan/`）陆续迁入。结构为公开 BWM 适配，不声称原创。

## 目录结构

```
actiflow/               主 Python 包
├── world_model/        B 的载体：动作条件视频世界模型（Wan2.2 + BWM 结构）
│   ├── models/         Wan2.2 DiT 差分、动作 encoder 双路径（context + AdaLN 调制）
│   ├── pipelines/      去噪管线、history/action 索引、自回归 rollout 契约
│   ├── training/       SFT trainer、flow-matching loss、ACTION/NULL 两臂
│   └── inference/      predict / 自回归生成 / 重载校验
├── goals/              A：目标验证
│   ├── completion/     完成估计器（冻结视觉特征 + 小 MLP，可撤销判定）
│   └── decomposition/  语言目标分解（继承 U_full400 已证收益）
├── control/            控制器
│   ├── fixed160/       最强已证基线（部署回退）
│   ├── a_only/         A-only 选择器（低置信回退 FIXED160）
│   └── ab_joint/       A+B 联合：continue/switch 候选评分与 margin/λ 冻结
├── data/               数据层
│   ├── robotwin/       RoboTwin HDF5（14D commanded joint targets，主训练）
│   ├── libero/         LIBERO（7D OSC，条件桥接 F08 解锁后使用）
│   └── normalization/  train-only p01/p99 归一化与 clip 记录
├── evaluation/         四级验收与指标（见下）
└── utils/

configs/                训练/评测配置（world_model / completion / control）
scripts/                CLI 入口（train / eval / data）
docs/
├── plan/               方案文档（总览、动作模型、最终 A+B、任务与验收）
└── paper/              论文主张边界（可以说 / 不能说）
experiments/            EXP-NNN 实验记录与产物指针（大文件不入库）
tests/
third_party/bwm/        官方 BWM 最小隔离参考副本（source commit 固定，附许可）
```

## 世界模型验收分级

`INTERFACE_READY` → `TRAIN_FIT` → `ACTION_USEFUL_HELDOUT` → `BRANCH_VALIDATED`
（达到 BRANCH_VALIDATED 后才允许插入 A+B 决策）

## 部署回退

任何新方案未过门时，交付/部署一律回退到已冻结的 FIXED160 控制器。
