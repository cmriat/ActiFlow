# data · 数据层

- `robotwin/` — RoboTwin `demo_clean` 三任务（handover_block / stack_blocks_two /
  pick_dual_bottles）各 50 条 HDF5；按完整 episode 划分（建议 32 train / 8 val / 10 test），
  动作取 `action/*joint_states` 的 14D commanded joint targets；须核验 `o_t → command_t → o_{t+1}` 时序对齐。
- `libero/` — LIBERO 7D OSC（条件桥接用）；仅使用 train 角色 root，dense RGB/proprio 由真实重放采集。
- `normalization/` — p01/p99 只从 train episodes 计算并映射 [-1,1]；记录 clip 比率与常量维处理；
  val/test 不得参与；gripper 单位与方向单独注明。

root 分配/冻结清单见 `experiments/` 记录；split 与 norm 冻结后不再变更。
