# 多头自注意力示例

此例使用 PyTorch 2.0+ 的缩放点积注意力（SDPA）。`True` 表示允许读取该键。示例约定某个查询的键全部被屏蔽时，该查询的模块输出为零。集成已有模型时先核对其约定，尤其是 `nn.MultiheadAttention` 的布尔 `attn_mask` 中 `True` 表示屏蔽。

```python
from typing import Optional

import torch
from torch import nn
from torch.nn import functional as F


class MultiHeadSelfAttention(nn.Module):
    """在同一序列上按头独立计算注意力，再投影回原嵌入空间。

    Args:
        embed_dim: 输入和输出的嵌入维度 D。
        num_heads: 头数 H，每头维度 d = D / H。
    """

    def __init__(self, embed_dim: int, num_heads: int = 8) -> None:
        super().__init__()
        if num_heads <= 0 or embed_dim <= 0 or embed_dim % num_heads:
            raise ValueError("头数和嵌入维度必须为正整数，且嵌入维度能被头数整除")
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.qkv_proj = nn.Linear(embed_dim, 3 * embed_dim, bias=False)
        self.out_proj = nn.Linear(embed_dim, embed_dim)

    def forward(
        self, x: torch.Tensor, attn_mask: Optional[torch.Tensor] = None
    ) -> torch.Tensor:
        """聚合每个查询允许读取的键所对应的值。

        Args:
            x: 浮点特征 [B, N, D]，B 为批量大小，N 为序列长度。
            attn_mask: 与 x 同设备的布尔张量 [B, 1, N, N] 或 [1, 1, N, N]。
                所有头共享掩码，最后两轴依次为查询和键。True 表示允许。

        Returns:
            投影后的特征 [B, N, D]。整行无有效键的查询输出全零。
        """
        batch_size, seq_len, embed_dim = x.shape
        # [B, N, 3D] → [B, N, 3, H, d] → [3, B, H, N, d]
        qkv = self.qkv_proj(x).reshape(
            batch_size, seq_len, 3, self.num_heads, self.head_dim
        ).permute(2, 0, 3, 1, 4)
        query, key, value = qkv.unbind(dim=0)

        valid_rows = None
        if attn_mask is not None:
            if attn_mask.dtype != torch.bool or attn_mask.device != x.device:
                raise ValueError("attn_mask 必须为布尔张量且与 x 同设备")
            if attn_mask.shape not in (
                (1, 1, seq_len, seq_len),
                (batch_size, 1, seq_len, seq_len),
            ):
                raise ValueError("attn_mask 的形状必须为 [B 或 1, 1, N, N]")
            # [B 或 1, 1, N, N] → [B 或 1, 1, N, 1]，沿键轴检测空行。
            valid_rows = attn_mask.any(dim=-1, keepdim=True)
            # 空行临时开放所有键，使 softmax 有定义。投影后再清零以消除偏置。
            attn_mask = attn_mask | ~valid_rows

        context = F.scaled_dot_product_attention(query, key, value, attn_mask=attn_mask)
        # [B, H, N, d] → [B, N, H, d] → [B, N, D]
        output = self.out_proj(context.transpose(1, 2).reshape(batch_size, seq_len, embed_dim))
        if valid_rows is not None:
            # [B 或 1, 1, N, 1] → [B 或 1, N, 1]，沿 D 广播。
            output = output.masked_fill(~valid_rows.squeeze(1), 0.0)
        return output
```

验证时可将 Q、K、V 和输出投影的权重分别设为单位矩阵。为输出层设置非零偏置，构造一行仅允许一个键的掩码和一行全屏蔽的掩码。前一行应等于选中的值加偏置，后一行应精确为零。再用足以让 FP16 朴素点积溢出的有限输入，在目标设备和实际 AMP 配置下与 FP32 参考输出比较。

布尔 `attn_mask` 通常不能留在 FlashAttention 内核里，会回退到 math 或 memory-efficient 实现。因果掩码优先只设 `is_causal=True`，不要在未核对当前 PyTorch 版本时再同时传入 `attn_mask`。需要和 `nn.MultiheadAttention` 对照时，记住它的布尔 `attn_mask` 里 `True` 表示屏蔽，与本例相反。
