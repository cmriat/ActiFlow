# 03 · F00–F11 任务与验收要点

> 提炼自旧库 `executor_tasks/2026-10-09_round9_final/04_TASKS_AND_ACCEPTANCE.md`。
> 工程/证据依赖关系；保留负结果；每编号对应协议、实际 EXP、raw 与判定。

| 任务 | 内容 | 要点 |
|---|---|---|
| F00 | 恢复、固定阳性产物与接管清理 | 核验 EXP-134；冻结 FIXED160 spec；新 root 分配（train/val/confirmation 登记在先） |
| F01 | 动作条件结构与数据契约 | 固定 source commit；`architecture_parity.md`；冻结 split/norm/时序索引表/ROI 规则 |
| F02 | 可训练实现 + 20 步 smoke + 过拟合检查 | factory/data/loss 兼容修复；两路径梯度；free-run 4 条 |
| F03 | 正式动作 SFT + NULL 对照 | 3000 步 33→81 帧；两臂同预算；固定 val 选 checkpoint |
| F04 | 独立动作效能 + 真实分支 | 60 clip 四路对照；12 root × 3 分支；8ep×3chunk 自回归 |
| F05 | 收尾 A：完成估计替代时钟 | 同刻谓词标签；低 false-complete、可撤销；dev 上 A-only vs FULL/FIXED160 |
| F06 | 训练不足的一次有依据修复 | 仅依 F03 val 决定一次；NULL 公平匹配 |
| F07 | 控制收益最后独立确认 | 新 root 2 任务 × 8 + 8 保持，48 episode；配对统计 |
| F08 | 条件桥接：7D LIBERO 动作模型 | 严格前置；1000 步档；分支能区分 continue/switch 才解锁 F09 |
| F09 | 最后一次 A+B 同平台判别 | margin/λ 冻结；成本计入；须超过计算对照 |
| F10 | 模型、控制器、证据与论文边界封装 | MODEL_DELIVERY；新进程重载核验；一页"可以说/不能说" |
| F11 | 巡检与终态作业清理 | 每 20 分钟巡检；Failed/Complete 留证后 delete；负结果永久保留 |

## 完成定义

必做训练/评测/封装完成后，清楚说明：动作模型达到哪一级（四级验收）、控制器是否新赢、联合是否成立。
不把"任务板全终态"当替代交付；不伪造成功。
