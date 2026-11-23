---
name: drawio-diagram
description: 为深度学习模型、网络架构、算法流程和代码交互生成可编辑的 Draw.io（.drawio / mxGraph XML）。从零生成走技术图规范；有参考图时只迁移风格；系统调用顺序走 UML 时序图。用户提到架构图、流程图、注意力、感受野、时序图时使用。
---

# Draw.io 图表

生成可在 Draw.io 中直接打开的 `.drawio` 文件。只覆盖两类图：AI 算法与模型结构，以及代码/服务交互。版式以对应 reference 为准，本文件只规定路由和 XML 底线。

## Step 0：任务识别

| 条件 | 执行 |
|------|------|
| 有参考图，且要「按这张图的风格」画 | `reference/style-migration.md` |
| 时序图、调用顺序、UML、Sequence | `reference/uml-diagram.md` |
| 其余从零生成（模型架构、算法流程、注意力、感受野、训练/推理数据流） | `reference/tech-diagram.md` |

`reference/generation.md` 只保留与版式无关的约束，视觉规范不要用它。项目技术栈、分层架构、表结构和 E-R 图走 `codegen-diagram`，不要用本 Skill 的配色重画一遍。

## 通用 XML 底线

- 根结构是 `<mxfile>` → `<diagram>` → `<mxGraphModel>` → `<root>`，根单元格固定为 `<mxCell id="0"/>` 和 `<mxCell id="1" parent="0"/>`。
- 节点 `vertex="1"`，连线 `edge="1"`。除 0 和 1 外都写 `parent="1"`（表行、UML 隔间等明确挂在父节点上的除外）。
- `id` 唯一即可，不必连续。连线的 `source` / `target` 必须指向已有节点；用 `mxPoint` 时不要同时写错误的 source。
- 手写文件时，`&` `<` `>` 各转义恰好一次，写成 `&amp;` `&lt;` `&gt;`。换行用 `&#xa;`。HTML 换行的一次转义是 `&lt;br&gt;`。再转义成 `&amp;lt;br&amp;gt;` 会在图上露出标签文本。属性里写原始小于号会让 XML 解析失败。只有 XML 库会自动转义时，才把未转义字符串交给库。
- 输出未压缩、可读的 XML，不要 base64 压缩 diagram。
- 颜色、字号、背景、网格以所执行的 reference 为准，不要混用两套配色。

## 输出

1. 两三句说明：图里有什么、依据是代码、论文还是用户描述，哪些是假设。
2. 完整 XML。
3. 打开方式：diagrams.net → Extras → Edit Diagram，或把文件拖进去；再导出 PNG / SVG / PDF。
4. 图题放在图下方。论文引用给一句可直接粘贴的「如图 x 所示，……」。
