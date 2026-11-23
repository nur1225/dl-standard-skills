# 训练与推理工程规范

## DataLoader 与样本覆盖

- 在 dataset 或 `collate_fn` 的边界记录单样本和单批次的结构、dtype、变长字段的 padding 值、mask 极性和样本 ID。检查最后一个不足批量的 batch；分割任务同时核对标签插值方式和忽略值。
- 对按索引读取的数据集，DDP 训练通常使用 `DistributedSampler`。由 sampler 管理 shuffle，训练每轮在创建迭代器前调用 `sampler.set_epoch(epoch)`。自定义 `IterableDataset` 在 rank 和 worker 两层分片，用样本 ID 验证覆盖与重复。
- 注明采样器和 DataLoader 各自的 `drop_last` 影响哪一层。分布式验证时，默认 sampler 可能补齐并重复尾部样本；使用无重复的评估分片，或保留 ID 和有效标记排除补齐项。若各 rank 的步数不同，确认循环内的 DDP 前向和自定义通信都能匹配。
- 需要复现增强或采样时，记录基础种子以及 worker、rank 和 epoch 的派生方式，覆盖实际使用的 Python、NumPy 或独立 RNG。启用 `persistent_workers` 后，检查新 epoch 的状态如何传到已有 worker。按设备、数据大小和测量结果选择 worker 数、锁页内存与异步拷贝。

## DDP、梯度累积与 AMP

- 对常见的单机或多机多 GPU 训练，一个进程绑定一张本地设备。用 local rank 选择设备，用 global rank 决定谁写共享日志或权重。注释说明每设备 batch、进程数和累积步数，以及尾部窗口的实际大小。
- 所有 rank 以相同顺序参与 collective，包括本地有效样本数为零的 rank。结果仅由主进程写出时，先在所有 rank 完成归约。采用 `no_sync()` 累积梯度时，把对应微批次的前向与反向一同放入上下文，在窗口最后一个微批次恢复同步。
- 写清 loss 分母是样本数、有效 token 数、有效像素数还是权重之和。等权微批次可按实际累积次数缩放。若目标是全局有效元素均值且各 rank 的分母不同，默认 DDP 会平均各 rank 的梯度；可将本地 loss 求和，用整个累积窗口的全局分母归一化，并补偿 DDP 的进程数因子。全局分母为零时让各 rank 一致处理该步。
- 用与目标设备匹配的 `torch.autocast` 包住前向和 loss。按后端与精度选择 `torch.amp.GradScaler`。启用缩放时，在整个窗口累积缩放后的梯度，更新边界按 `unscale_`（需要裁剪或检查梯度时）、梯度裁剪、`scaler.step`、`scaler.update` 的顺序执行。按更新步推进的调度器与实际成功的优化器更新对齐。

## 推理模式与指标统计

- 标准验证或推理同时使用 `model.eval()` 与 `torch.inference_mode()`。需要输入梯度的解释或优化任务保留 autograd。训练期间穿插验证时恢复先前的训练模式，并保持部署时相同的预处理。
- 在指标定义旁写清统计单位、忽略条件、分母和平均方式。Accuracy 累加正确数与有效数，micro Dice 累加交集和基数，macro Dice 按约定的样本或类别先分别计算。精确 AUROC、AP 等依赖全局排序的指标汇总预测与标签，或注明所用近似算法。
- 例如两个 rank 分别预测正确 1/1 和 1/3，全局准确率应为 2/4。下面的函数接收整个评估周期的本地累计计数。所有 rank 都调用；NCCL 使用对应的 CUDA 设备，Gloo CPU 评估使用 CPU。此例把空验证集的准确率记为未定义。

```python
import torch
import torch.distributed as dist


def global_accuracy(correct: int, valid: int, device: torch.device) -> float:
    """按有效样本数计算全局准确率。

    Args:
        correct: 本 rank 预测正确的有效样本数。
        valid: 本 rank 参与评估的有效样本数，已排除忽略项及补齐项。
        device: 当前进程组支持的计数张量设备。

    Returns:
        准确率。所有 rank 合计无有效样本时返回 NaN。
    """
    counts = torch.tensor([correct, valid], dtype=torch.int64, device=device)
    if dist.is_available() and dist.is_initialized():
        dist.all_reduce(counts, op=dist.ReduceOp.SUM)
    total_correct, total_valid = counts.tolist()
    return total_correct / total_valid if total_valid else float("nan")
```
