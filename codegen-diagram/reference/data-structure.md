# 数据结构图（表格式）

采用表格式展示核心业务实体及其字段列表（字段名、类型、约束），用连线标注实体间关系与基数。

## 使用时机

- 用户希望整理核心业务实体（表）及其字段列表
- 用户提到「数据结构图」「表结构图」「实体字段关系图」
- 需要在文档或论文中展示主要实体及字段详情，比 E-R 图更直观

## 推荐版式（黑白学术风格）

1. **每个实体/表**：矩形框分为表头（表名）+ 表体（字段列表）
2. **字段格式**：三列 `字段名 | 类型 | 约束`（如 `id | int | PK`）
3. **实体间连线**：实线箭头连接，箭头指向「多」方，线上标注基数（1、n）
4. **图题**：预留「图 x-y 系统数据结构图」

## 黑白配色

- 表头：`fillColor=#FFFFFF`，`fontStyle=1`（加粗）
- 表体：`fillColor=#F5F5F5` 或 `#FFFFFF`
- 连接线：`strokeColor=#333333`，`endArrow=block`
- 基数标注：`fontStyle=1`（加粗）

## 工作流程

1. **获取表结构**：从 schema.sql、ORM 模型、文档提取表名、字段、类型、主键、外键
2. **识别关系**：主键、外键、1:1/1:N/M:N、基数
3. **规划布局**：画布 1100–1200px 宽，主表在左、从表在右
4. **生成 .drawio**：表头+表体矩形、实线箭头、edgeLabel 基数
5. **文字说明与图题**：表及字段总览、关系说明

## 与 E-R 图的关系

- 本图偏重**表结构与字段详情**，字段信息完整、直观
- E-R 图偏重**实体关系建模**，采用 Chen 记法，强调概念层
- 两者可配合：E-R 图做概念建模，数据结构图展示物理表结构

## 从代码抽取

来源优先级：迁移 SQL、`schema.sql`、ORM 模型、原型注释。只画核心表。超过 8 张时画聚合根和它的直接外键表，其余在文字说明里列出名字。

- 字段顺序：主键、业务字段、外键。类型照 DDL 或 ORM，不改写成口头语。
- 约束只写代码里有的：`PK`、`FK`、`UK`、`NOT NULL`。没有就留空。
- 多对多画中间表，不要收成一条线。箭头指向「多」的一侧，基数写在线上。
- 找不到 schema 时不要发明表。

## 布局

画布宽 1100–1200。主表在左，从表在右，表间距不少于 80。用 Draw.io 表格：父级 `shape=table`，每行一个 `shape=tableRow`，行内三格 `shape=partialRectangle`，列宽约 120 / 100 / 80。表之间的连线连到表（父单元格），不要连到行。图题在最低表底边之下 36px。

## 骨架

一行字段示范。其余字段复制 `tableRow`，并把父表的高度加上每行 28px。`y` 是相对父表的。

```xml
<mxCell id="2" value="orders" style="shape=table;startSize=28;container=1;collapsible=0;childLayout=tableLayout;fixedRows=1;rowLines=0;fontStyle=1;strokeColor=#333333;fillColor=#FFFFFF;fontSize=13;fontColor=#333333;" vertex="1" parent="1">
  <mxGeometry x="40" y="40" width="300" height="56" as="geometry"/>
</mxCell>
<mxCell id="3" value="" style="shape=tableRow;horizontal=0;startSize=0;swimlaneHead=0;swimlaneBody=0;fillColor=#F5F5F5;collapsible=0;dropTarget=0;points=[[0,0.5],[1,0.5]];portConstraint=eastwest;top=0;left=0;right=0;bottom=0;" vertex="1" parent="2">
  <mxGeometry y="28" width="300" height="28" as="geometry"/>
</mxCell>
<mxCell id="4" value="id" style="shape=partialRectangle;connectable=0;fillColor=none;top=0;left=0;bottom=0;right=0;align=left;spacingLeft=6;fontSize=12;fontColor=#333333;" vertex="1" parent="3">
  <mxGeometry width="120" height="28" as="geometry"/>
</mxCell>
<mxCell id="5" value="bigint" style="shape=partialRectangle;connectable=0;fillColor=none;top=0;left=0;bottom=0;right=0;align=left;spacingLeft=6;fontSize=12;fontColor=#333333;" vertex="1" parent="3">
  <mxGeometry x="120" width="100" height="28" as="geometry"/>
</mxCell>
<mxCell id="6" value="PK" style="shape=partialRectangle;connectable=0;fillColor=none;top=0;left=0;bottom=0;right=0;align=left;spacingLeft=6;fontSize=12;fontStyle=1;fontColor=#333333;" vertex="1" parent="3">
  <mxGeometry x="220" width="80" height="28" as="geometry"/>
</mxCell>
```

基数写在连线的 `value` 上，例如 `value="1..n"`，样式 `endArrow=block;strokeColor=#333333;fontStyle=1;fontSize=12`。箭头指向多的一侧。
