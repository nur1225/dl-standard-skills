# dl-standard-skills

Grok / Agent Skills 集合，聚焦 **Python 深度学习工程规范** 与 **Draw.io 专业图表生成**。

## 包含的 Skills

### 1. `dl-python-standards`
编写、重构或审查 Python 深度学习代码时使用。

- 覆盖 PyTorch 模型、损失函数、NumPy/OpenCV 前后处理、DataLoader、DDP、AMP、训练与推理指标
- ONNX 导出与 TensorRT 部署规范
- 重点检查：张量契约、数值稳定性、跨进程统计、后端输出一致性

**参考文档**：
- `references/attention.md` — 多头注意力示例与 mask 约定
- `references/loss-and-preprocess.md` — 损失与前后处理
- `references/training-inference.md` — 训练与推理工程规范
- `references/export-deployment.md` — 模型导出与部署

### 2. `codegen-diagram`
基于**当前项目/代码仓库**生成 Draw.io 图表，支持四种类型：

| 类型 | 触发关键词 |
|------|------------|
| 技术栈图 | 技术栈、组件选型 |
| 系统架构图 | 系统架构、分层结构 |
| 数据结构图 | 数据结构图、表结构图 |
| E-R 图 | E-R 图、实体关系图 |

输出符合 Draw.io 语法的 `.drawio` 文件（mxGraph XML），可直接导入编辑。采用黑白学术配色。

### 3. `drawio-diagram`
为深度学习模型、网络架构、算法流程和代码交互生成可编辑的 Draw.io 图表。

- **从零生成**：模型架构、注意力、感受野、训练/推理数据流 → `reference/tech-diagram.md`
- **风格迁移**：有参考图时只迁移版式与配色 → `reference/style-migration.md`
- **UML 时序图**：系统调用顺序、交互流程 → `reference/uml-diagram.md`

## 使用方式

将这些 skill 目录放到 Agent 的 skills 目录下即可被自动发现与调用。

## License

MIT
