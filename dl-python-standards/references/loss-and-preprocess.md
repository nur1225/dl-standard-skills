# 损失与前后处理

## 损失

- 接口上写清预测和标签的形状、含义、数值域。分类写 logits 还是概率；框写编码前还是解码后；掩码写概率、logits 还是类别索引。
- `CrossEntropyLoss` 收 logits，标签是类别索引 `long`。不要先 softmax，也不要传 one-hot，除非改用与 one-hot 匹配的另一接口。`ignore_index` 不计入默认 mean 的分母；不要再手写 one-hot 后乘掩码，否则分母会把忽略位置算进去。
- `BCEWithLogitsLoss` 收 logits。已经 sigmoid 过再用它，等于激活做了两次。`pos_weight` 的形状是 `[C]`，不是 `[N,C]`。
- 标签平滑只加在损失里，或只加在标签生成里，二选一。类权重和过采样会叠乘，同时用时在注释里写明这是有意的。
- Dice、IoU 写明在哪些轴上求和，以及「预测和标签都为空」记 1 还是 0。`smooth` 已经加在分母上时，不要再无说明地加一个 `eps`。空集合的约定在训练和导出验收里用同一个。
- `reduction="none"` 之后自己做 mean，分母必须和损失定义同一个有效掩码。有效元素数为 0 时，各 rank 的返回值要一致，避免有的返回 0、有的返回 NaN。
- 检测损失写清哪些匹配框进入分类和回归的分母，未匹配的预测是背景类还是直接不参与。不要把「乘了 0」说成「未参与」——乘 0 仍可能留下无匹配时的梯度噪声，取决于实现。

## NumPy / OpenCV 前后处理

- 在预处理函数的 docstring 里固定输入布局、通道顺序、dtype 和数值范围。`cv2.imread` 是 BGR、uint8、HWC。`cv2.resize` 的 `dsize` 是 `(width, height)`，不是 `(height, width)`。
- 图像用 `INTER_LINEAR` 或 `INTER_CUBIC`。标签和掩码只用 `INTER_NEAREST`，并且不要做减均值。
- 归一化的 mean/std 必须和像素在同一数值范围、同一通道顺序。ImageNet 的 0.485 这组是 RGB、0–1；直接减到 0–255 的 BGR 上是错的。
- 先 BGR→RGB，再换成浮点并除以 255，再减 mean 除 std，再 HWC→CHW。`transpose` 之后用 `ascontiguousarray`，再 `torch.from_numpy`。`from_numpy` 与数组共享内存，函数返回前不要让调用方还能改到这块数组。
- 框写明 xyxy 还是 xywh、绝对像素还是相对边长、原点在左上还是中心。letterbox 要记下缩放和 padding，把网络输出的框映回原图时用同一组数的逆变换。训练和导出走同一套，不要训练用拉伸、推理用 letterbox。

```python
import cv2
import numpy as np
import torch


def preprocess_bgr_uint8(
    image: np.ndarray, mean_rgb: np.ndarray, std_rgb: np.ndarray
) -> torch.Tensor:
    """把 OpenCV 读入的图像变成 RGB NCHW float32。

    Args:
        image: HWC、3 通道、uint8 的 BGR 图像。
        mean_rgb: 形状 [3]，定义在 0–1 的 RGB 上。
        std_rgb: 形状 [3]，与 mean_rgb 同一色彩空间和数值范围。

    Returns:
        形状 [1, 3, H, W] 的 float32 张量，不与 image 共享内存。
    """
    if image.ndim != 3 or image.shape[2] != 3 or image.dtype != np.uint8:
        raise ValueError("需要 HWC、3 通道、uint8 的 BGR 图像")
    rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB).astype(np.float32) / 255.0
    normalized = (rgb - mean_rgb) / std_rgb
    chw = np.ascontiguousarray(normalized.transpose(2, 0, 1))
    return torch.from_numpy(chw).unsqueeze(0)
```
