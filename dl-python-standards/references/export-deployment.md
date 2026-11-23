# 模型导出与部署规范

## 导出前确定契约

- 记录权重版本、推理模式、输入输出名称、dtype、布局与维度约束，以及前后处理的归属。检测模型还要说明框坐标、分数含义、NMS 所在端和有效检测数。用一份确定的输入保存 PyTorch 推理基准，供各后端复用。
- 根据目标 ONNX Runtime 执行提供程序（EP）与 TensorRT 解析器支持范围选择 opset，同时记录 PyTorch、导出器、运行时、精度和目标设备版本。使用 dynamo 导出时以 `dynamic_shapes` 表达动态维，维护旧导出路径时核对其 `dynamic_axes` 配置。为多输入写清共享的 batch、序列长度等约束。

## 逐层检查算子兼容性

| 检查位置 | 操作与通过依据 |
| --- | --- |
| PyTorch → ONNX | 检查依赖输入值的 Python 分支、转为 Python 标量的尺寸、自定义算子及其 lowering。用能触发不同分支或尺寸的输入对照导出图。若导出结果将样例值固化，改用导出器支持的表达或显式收窄并记录输入契约。 |
| ONNX 图 | 用 `onnx.checker.check_model` 校验图结构，必要时用 shape inference 定位秩和类型问题。针对风险节点核对 domain、opset、属性和输入类型，例如 Resize 的坐标变换、Gather 索引与 NMS 的框格式。 |
| ONNX Runtime | 在目标 EP 上创建会话、执行基准输入并比对结果。若有算子放置要求，查看 profiling 或节点分配日志，记录 CPU fallback 节点及其影响。 |
| TensorRT | 使用目标版本的 ONNX parser 解析，并用相同精度、插件及 profile 实际构建 engine。失败时记录节点名、算子、opset、dtype、shape 与 parser/builder 错误。结合目标版本的支持表，选择等价改写、已部署插件或显式移出图的前后处理，并重新对照输出。 |

## 动态形状与精度

- 给每个动态执行输入配置 optimization profile 的 min/opt/max，涉及 shape tensor 时同时约束其取值。opt 取代表性工作负载。构建后在运行时选定 profile、设置实际输入形状，再按解析出的输出形状或运行时分配器分配输出。验证关联输入维度一致，并检查超出 profile 的请求如何处理。
- 用相同权重、输入和预处理依次比对 PyTorch、ONNX Runtime 和实际 TensorRT engine。至少覆盖任务相关的典型输入及边界，例如 batch=1、动态尺寸的最小与最大值、空检测结果。先核对输出名称、shape、dtype 和有限性，再比较数值。
- 浮点输出按模型、输出量纲、精度和任务预算预先确定 `atol`、`rtol` 和任务指标容差。离散标签按语义比较。NMS 或 Top-K 存在并列分数时，按类别及框匹配比较，记录阈值附近的差异。发生误差时先比对后处理前的输出，以区分图内计算与坐标还原或 NMS 的差异。
- FP16 检查易溢出的归约和分数。INT8 记录量化方式、尺度来源及代表性数据覆盖范围。验收记录注明模型、后端、硬件、profile、精度、输入和误差；明确哪些阶段已在目标环境执行，哪些只完成静态检查。
