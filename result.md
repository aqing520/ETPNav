# ETPNav R2R-CE / RxR-CE Evaluation Results

评测日期：2026-04-20

## Setup

- 项目路径：`/data1/code/embAI_sup/awzy/ETPNav`
- R2R checkpoint：`data/logs/checkpoints/release_r2r/ckpt.iter12000.pth`
- RxR checkpoint：`data/logs/checkpoints/release_rxr/ckpt.iter19600.pth`
- DD-PPO 权重：`data/ddppo-models/gibson-2plus-resnet50.pth`
- R2R 评测 ckpt id：`ckpt_59`
- RxR 评测 ckpt id：`ckpt_97`
- 评测范围：`EVAL.EPISODE_COUNT=-1`
- R2R `val_unseen` episodes：1839
- R2R `val_seen` episodes：778
- RxR `val_unseen` episodes：11006
- R2R 主 baseline 配置：`val_unseen`，默认 `control` 回退，`ALLOW_SLIDING=True`，`merge_ghost=True`，`use_sprels=True`
- RxR 主 baseline 配置：`val_unseen`，`IL.back_algo=control`，`ALLOW_SLIDING=False`，4 GPU eval

## Main Results

表中 `SR`、`OSR`、`SPL`、`nDTW`、`SDTW` 为百分比；`NE` 为 `distance_to_goal`；`TL` 为 `path_length`。

| Dataset | Experiment | Split | Variant | Episodes | SR | OSR | SPL | nDTW | SDTW | NE | TL | Steps | Collisions | Ghost Cnt |
|---|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| R2R-CE | `awzy_r2r_12000_full_20260420_0110` | `val_unseen` | baseline | 1839 | 56.12 | 63.30 | 48.15 | 62.38 | 46.15 | 4.78 | 11.41 | 79.25 | 0.196 | 23.92 |
| R2R-CE | `awzy_r2r_12000_valseen_20260420_1125` | `val_seen` | baseline | 778 | 69.41 | 75.58 | 61.53 | 71.88 | 58.93 | 3.52 | 11.15 | 73.06 | 0.163 | 24.12 |
| R2R-CE | `awzy_r2r_12000_unseen_teleport_20260420_1125` | `val_unseen` | `IL.back_algo=teleport` | 1839 | 56.01 | 63.78 | 47.90 | 63.66 | 46.83 | 4.76 | 11.27 | 88.01 | 0.207 | 24.09 |
| R2R-CE | `awzy_r2r_12000_unseen_noslide_20260420_1125` | `val_unseen` | `ALLOW_SLIDING=False` | 1839 | 57.59 | 64.06 | 48.61 | 62.13 | 47.12 | 4.77 | 11.76 | 112.41 | 0.166 | 24.24 |
| R2R-CE | `awzy_r2r_12000_unseen_mergeghost_off_20260420_1128` | `val_unseen` | `MODEL.merge_ghost=False` | 1839 | 57.10 | 62.81 | 49.33 | 63.17 | 47.34 | 4.66 | 11.22 | 79.53 | 0.222 | 31.78 |
| R2R-CE | `awzy_r2r_12000_unseen_sprels_off_20260420_1128` | `val_unseen` | `MODEL.use_sprels=False` | 1839 | 56.01 | 63.08 | 48.29 | 62.67 | 46.30 | 4.77 | 11.34 | 78.78 | 0.196 | 23.82 |
| RxR-CE | `release_rxr` | `val_unseen` | baseline | 11006 | 54.02 | 62.51 | 44.16 | 61.12 | 44.54 | 5.96 | 18.37 | 175.87 | 0.393 | 33.69 |

## Val Unseen Ablation Deltas

相对 `awzy_r2r_12000_full_20260420_0110` baseline。百分比指标使用百分点差值。

| Variant | Delta SR | Delta OSR | Delta SPL | Delta nDTW | Delta SDTW | Delta NE | Delta TL | Delta Steps | Delta Collisions | Delta Ghost Cnt |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `IL.back_algo=teleport` | -0.11 | +0.49 | -0.25 | +1.27 | +0.69 | -0.02 | -0.14 | +8.76 | +0.012 | +0.17 |
| `ALLOW_SLIDING=False` | +1.47 | +0.76 | +0.46 | -0.25 | +0.98 | -0.01 | +0.35 | +33.16 | -0.029 | +0.32 |
| `MODEL.merge_ghost=False` | +0.98 | -0.49 | +1.17 | +0.79 | +1.19 | -0.12 | -0.19 | +0.28 | +0.026 | +7.86 |
| `MODEL.use_sprels=False` | -0.11 | -0.22 | +0.13 | +0.29 | +0.15 | -0.01 | -0.07 | -0.47 | +0.000 | -0.10 |

## Notes

- `val_unseen` baseline 结果为 `SR=56.12`、`SPL=48.15`，和论文 R2R-CE `val_unseen` 约 `SR=57`、`SPL=49` 的量级基本对齐，差距约 1 个百分点。
- `val_seen` 明显高于 `val_unseen`，`SR=69.41`、`SPL=61.53`，符合 seen split 难度更低的预期。
- RxR-CE `val_unseen` baseline 结果为 `SR=54.02`、`SPL=44.16`、`nDTW=61.12`、`SDTW=44.54`，和论文 RxR-CE `val_unseen` ETPNav 行 `SR=54.79`、`SPL=44.89`、`nDTW=61.90`、`SDTW=45.33` 基本对齐，主要指标差距均小于 1 个百分点。
- RxR-CE 当前 `NE=5.96`，论文 ETPNav `NE=5.64`，差距约 `+0.32m`，略差但不异常。
- `ALLOW_SLIDING=False` 的 `SR` 和 `SPL` 略高，但 `Steps` 大幅增加到 `112.41`，说明路径执行代价变高；`Collisions` 是归一化统计，不能直接理解为绝对碰撞次数下降。
- `MODEL.merge_ghost=False` 在本次结果里 `SPL`、`nDTW`、`SDTW` 都有小幅提升，但 `Ghost Cnt` 明显增大，需要结合轨迹可视化或更多 seed 判断是否稳定。
- `MODEL.use_sprels=False` 与 baseline 基本持平，说明该 checkpoint 在当前评测上对 spatial relation 开关不敏感，差异可能在评测噪声范围内。
- `IL.back_algo=teleport` 对 `SR`、`SPL` 基本无提升，但 `nDTW` 和 `SDTW` 更高，主要影响轨迹形状而不是最终成功率。
- `diag_rxr_ddp` 是 16 episode 的 DDP smoke/diagnostic run，不计入正式结果表。

## Result Files

- Baseline `val_unseen`：`data/logs/eval_results/awzy_r2r_12000_full_20260420_0110/stats_ckpt_59_val_unseen.json`
- Baseline `val_seen`：`data/logs/eval_results/awzy_r2r_12000_valseen_20260420_1125/stats_ckpt_59_val_seen.json`
- Teleport ablation：`data/logs/eval_results/awzy_r2r_12000_unseen_teleport_20260420_1125/stats_ckpt_59_val_unseen.json`
- No sliding ablation：`data/logs/eval_results/awzy_r2r_12000_unseen_noslide_20260420_1125/stats_ckpt_59_val_unseen.json`
- Merge ghost off ablation：`data/logs/eval_results/awzy_r2r_12000_unseen_mergeghost_off_20260420_1128/stats_ckpt_59_val_unseen.json`
- Spatial relation off ablation：`data/logs/eval_results/awzy_r2r_12000_unseen_sprels_off_20260420_1128/stats_ckpt_59_val_unseen.json`
- RxR baseline `val_unseen`：`data/logs/eval_results/release_rxr/stats_ckpt_97_val_unseen.json`
