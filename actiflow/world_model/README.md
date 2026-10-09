# world_model · 动作条件视频世界模型（B 的载体）

`W(x_initial, x_history, commanded_actions) → future_video`

- 从本地原始 **Wan2.2-TI2V-5B**（DiT/VAE）初始化，action encoder 随机初始化。
- 复现公开 **BWM 双路径**动作条件：逐帧 context cross-attn + 4 步展平 AdaLN 调制。
- 主训练：RoboTwin **14D commanded joint targets**（与官方 14D EEF 维度/拓扑一致、语义不同，model card 须注明）。
- 两臂公平对照：`ACTION_BWM` vs `NULL_ACTION`（同初始化/数据/噪声流/预算）。
- 条件桥接（F08）通过后才在此派生 `action_dim=7` 的 LIBERO 变体（DiT 差分迁移 + 新 7D encoder，独立 norm）。

验收分级：`INTERFACE_READY` → `TRAIN_FIT` → `ACTION_USEFUL_HELDOUT` → `BRANCH_VALIDATED`。

详细方案见 `docs/plan/01-wan-action-model.md`。
