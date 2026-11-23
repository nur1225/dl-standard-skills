# 图表生成通用约束

本文件只保留与版式无关的通用约束。视觉规范（颜色、字号、布局）以具体 reference（tech-diagram、style-migration、uml-diagram）为准。

## 输出格式

- 始终输出完整、未压缩的 Draw.io XML（mxGraphModel）。
- 不要输出 base64 压缩的 diagram。
- 文件扩展名建议 `.drawio`。

## XML 基本要求

- 根结构：`<mxfile>` → `<diagram>` → `<mxGraphModel>` → `<root>`。
- 根单元格固定为 `<mxCell id="0"/>` 和 `<mxCell id="1" parent="0"/>`。
- 所有其他元素的 `parent` 默认为 `"1"`（除非明确挂在父节点上）。
- `id` 必须唯一。
- 节点使用 `vertex="1"`，连线使用 `edge="1"`。
- 特殊字符转义：`&` → `&`，`<` → `<`，`>` → `>`。只转义一次。
- 换行优先使用 `&#xa;`。

## 内容原则

- 只画有依据的内容（代码、论文、用户明确描述）。
- 不确定的地方标注「假设」或「待确认」。
- 不要编造不存在的模块、维度或依赖。
