# 01 · Wan2.2 + BWM 结构的动作输入世界模型

> 提炼自旧库 `executor_tasks/2026-10-09_round9_final/02_WAN_ACTION_MODEL.md`（实现细节以该文为准）。

## 结构（公开 BWM 双路径，source commit 固定）

设动作 `a ∈ R[B,T,14]`，T 满足 `4k+1`：

1. **context 路径**：`Linear(14,D) → GELU → Linear(D,D)` 得逐步 context；text off，context 即动作 token。
2. **调制路径**：首帧 repeat 3 次 padding 后按 4 步展平为 `((T+3)/4)×56`，
   `Linear(56,4D) → SiLU → Linear(4D,D)` 得逐 latent 时间的动作调制；
   重复到同时刻空间 tokens，**先加进 time embedding 再过 time_projection** 生成 6 路 block 调制。
3. 每个 DiT block 对动作 context cross-attend + 上述 AdaLN；视频 self-attn/FFN 继承 Wan2.2。
4. 条件 = 初始帧 + 最近 8 帧历史；正式 81 总帧（未来 72）。每去噪步恢复固定条件 latent，
   初始帧 clean、历史固定低噪声；**只对未来 latent 算 flow-matching loss**。
5. 动作缺失/长度非法必须显式报错，禁止静默退回无动作。

## 初始化与动作接口

- 从**原始 Wan2.2-TI2V-5B** 加载 DiT/VAE（记录 shard/config hash），action encoder 随机初始化；
  不得用官方已训 BWM 权重当初始化。
- 动作 = RoboTwin **14D commanded joint targets**（左右各 6 关节 + 夹爪，hdf5 `action/*joint_states`）；
  与官方 14D EEF 绝对命令维度/拓扑一致、**语义不同**，model card 必须写明。
- 禁止把未来测量 state 当动作输入。
- 归一化：train-only p01/p99 → [-1,1]，记录 clip 比率；零归一化 ≠ 物理静止。

## 数据

- RoboTwin `demo_clean` 三任务各 50 条 HDF5；按完整 episode 划分 32/8/10，overlap clip 留在同 split。
- 主用 head RGB，288×384 起训可到 480×640；多视图 latent 拼接接口保留但不阻塞主交付。
- 必须实测核验 `o_t → command_t → o_{t+1}` 对齐；动作 token 数与未来 transition 数的边界要有显式索引表。

## SFT 阶梯

1. 20 步 smoke（数值/梯度/形状/泄漏/重载一致）——不用于判断效能。
2. 8–16 clips、200–400 步可学性 + free-run ≥4 条。
3. 正式两臂 `ACTION_BWM` / `NULL_ACTION`：同初始化、同 train 列表、同噪声流、同优化步。
4. 默认 3000 optimizer updates（前 1000 步 33 帧、后 2000 步 81 帧）；val 仍改善可到 6000。
   bf16；encoder 全训 lr 1e-4 + DiT LoRA rank32 lr 2e-5；VAE 冻结、text off。
5. checkpoint 选择 = 固定规则下最低 val 未来误差且不丢动作响应；test 只用选定版本。

## 验收（四级）

`INTERFACE_READY`（接口通、重载一致）→ `TRAIN_FIT`（train 拟合）→
`ACTION_USEFUL_HELDOUT`（heldout 动态误差较 NULL 相对降 ≥5%，正确动作优于错动作 episode 比例 ≥65%）→
`BRANCH_VALIDATED`（12 root × 3 合法分支，预测更接近自己分支真实未来）。

评测细则（ROI 指标、50 步去噪、自回归 8ep×3chunk、bootstrap 区间）见 `actiflow/evaluation/README.md`。

## 交付格式

独立目录：权重（或 base+delta）、action_norm、schema、训练配置/manifest、
`predict.py / train.py / eval.py` 真实 CLI、示例输入与结果 JSON/视频；
新进程重载必须通过。无动作输出头须写明 "action→video world model"，不是完整策略。
