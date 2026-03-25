# OpenVLA-LoRA 微调项目复盘（完结）

本仓库记录我在 OpenVLA 上进行 LoRA 微调与 LIBERO-Spatial 评估的完整过程。

## 项目结论（最终）

- `8000 steps`：评估脚本成功率 `38%`，肉眼观察约 `58%`。
- `6500 steps`：评估脚本成功率 `52%`，肉眼观察约 `65%`。
- 结论：`6500 steps` 综合表现更好，本项目在该阶段收尾。

---

## 关键现象

在 `8000 steps` 的结果中出现明显分化：

- 成功样本：抓取动作连贯、执行流畅。
- 失败样本：机械臂“完全不动”（非“抓偏”，而是直接冻结）。

这类现象可归纳为：

- `Freezing`
- `Action Collapse`

即策略在不确定状态下收缩到“近零动作”的局部最优。

---

## 原因分析

### A. 过拟合导致不确定性抑制（Overfitting to Specific States）

训练步数增加到 `8000` 后，模型对训练轨迹记忆增强；在评估时若初始状态发生轻微扰动（视角、物体初始位姿等），模型可能输出大量接近零速度的 action token，从而表现为“不敢动”。

### B. 学到“静止”伪相关（Spurious Correlation with Stop）

数据中常含起始/结束静止片段，模型可能把“静止”赋予过高先验权重。一旦早期视觉信号不够强，策略会落入“保持静止”的局部最优解。

---

## 为什么 5000 与 8000 差异大？

这是典型的 Bias-Variance 权衡：

- `5000 steps`：更敢动，但抓取精度不稳定（失败多为抓偏/掉落）。
- `8000 steps`：熟悉场景很稳；陌生初始状态下易冻结（直接不动）。

简化理解：

- `5000` 像“会做但粗心”；
- `8000` 像“会的全对，不会就交白卷”。

---

## 成功率差异说明（脚本 38% vs 观察 58%）

LIBERO 评估脚本通常采用严格几何判据（位置、姿态、稳定时长等），其成功定义是 benchmark 级“一致标准”。

- 肉眼“功能上完成”不一定满足脚本全部阈值。
- 因此 `58%`（观察）高于 `38%`（脚本）是合理现象。

建议：报告时以脚本成功率为主；分析时同时保留人工观察结论。

---

## 视频示例（2026_03_24）

以下挑选了若干 `成功/失败` 样本，便于快速对比：

### 成功样本

- [episode=7, success=True](./2026_03_24/2026_03_24-17_59_48--episode=7--success=True--task=pick_up_the_black_bowl_between_the_plate_and_the_r.mp4)
- [episode=16, success=True](./2026_03_24/2026_03_24-17_59_48--episode=16--success=True--task=pick_up_the_black_bowl_between_the_plate_and_the_r.mp4)
- [episode=9, success=True](./2026_03_24/2026_03_24-18_17_59--episode=9--success=True--task=pick_up_the_black_bowl_in_the_top_drawer_of_the_wo.mp4)

### 失败样本（典型冻结/不动）

- [episode=2, success=False](./2026_03_24/2026_03_24-17_59_48--episode=2--success=False--task=pick_up_the_black_bowl_between_the_plate_and_the_r.mp4)
- [episode=14, success=False](./2026_03_24/2026_03_24-17_59_48--episode=14--success=False--task=pick_up_the_black_bowl_between_the_plate_and_the_r.mp4)
- [episode=11, success=False](./2026_03_24/2026_03_24-18_17_59--episode=11--success=False--task=pick_up_the_black_bowl_on_the_ramekin_and_place_it.mp4)

> 注：`2026_03_24` 文件夹中保留了完整视频记录，这里仅展示对比样本。

---

## 下一步优化建议（供后续复现实验参考）

1. 评估中间 checkpoint（`6000~7000`）做早停选择。
2. 若重训，建议使用 `dropout=0.1`（避免 `dropout=0.0` 带来的过拟合）。
3. 评估时增加 episode 数量（如 `20/50`）以降低单一初始化偏差。

---

## 最终结论

`6500 steps` 在“稳定性 + 泛化”上优于 `8000 steps`，并显著缓解“失败即不动”的策略坍缩倾向。

本次 OpenVLA-LoRA 微调项目至此结束。

感谢所有阅读到这里的同学，我们一起加油。
