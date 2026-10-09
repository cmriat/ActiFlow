# evaluation · 验收与指标

世界模型四级验收：`INTERFACE_READY` → `TRAIN_FIT` → `ACTION_USEFUL_HELDOUT` → `BRANCH_VALIDATED`。

- heldout 四路对照：ACTION 正确动作 / ACTION 错动作（预冻结置换）/ NULL / 初始模型参考；
  同样本、同 seed、同去噪步数（正式 50 步）。
- 主指标：动态 ROI 感知误差（LPIPS 或冻结视觉特征距离）；全帧 PSNR/SSIM 辅助；
  机器人/物体轨迹与接触/完成事件误差；episode 级聚合 + bootstrap 区间。
- 分支验收：独立 root 上同起点多合法动作分支，预测须更接近自己分支的真实未来。
- 自回归：≥8 个 heldout episode × 3 chunk，只用预测 history，与 teacher-forced 分开报告。
- 推进门槛：heldout 动态误差较 NULL 相对降低 ≥5%；正确动作优于错动作的 episode 比例 ≥65%。

控制侧：配对 wins/losses、原始成功率、bootstrap/精确检验、保持与额外耗时分别报告。
