# E-R 图（Chen 记法）

采用经典 Chen 记法（矩形=实体、菱形=联系、椭圆=属性、PK 下划线），基于数据表识别实体、主键、外键及关联类型。

## 使用时机

- 用户需要数据库层面的 E-R 图
- 用户提到「E-R 图」「实体关系图」「数据库关系图」「表结构设计」
- 论文/设计文档中需展示规范的实体关系及主键、外键

## Chen 记法图形约定（必须遵守）

| 图形 | Draw.io 形状 | 含义 |
|------|-------------|------|
| 矩形 | `rounded=0` | 实体（表） |
| 菱形 | `rhombus` | 联系（关系） |
| 椭圆 | `ellipse` | 属性（字段） |

- **主键**：属性椭圆使用 `fontStyle=4`（下划线）
- **基数**：实体与联系间连线上标注 1 或 n，连线 `endArrow=none`

## 黑白学术风格

- 所有图形：`fillColor=#FFFFFF`，`strokeColor=#333333`
- 实体：13 号加粗；属性/联系：11–12 号
- 连接线：实体–联系 `strokeWidth=1.5`，属性线 `strokeWidth=1`

## 工作流程

1. **获取表结构**：从 schema.sql、ORM、db_utils、文档提取
2. **识别主键、外键与联系**：标记 PK、外键指向、联系名称与基数
3. **规划布局**：实体居中，属性从实体辐射展开，联系菱形放两实体之间
4. **生成 .drawio**：矩形、菱形、椭圆三种图形，主键下划线，基数标注
5. **文字说明与图题**：实体集合及主要联系类型

## 与数据结构图的关系

- 数据结构图：表格式，字段详情完整
- 本图：Chen 记法，概念层实体关系建模
- 可先用数据结构图梳理字段，再用本图生成正式 E-R 图

## 从代码抽取

来源与数据结构图相同。每张表一个实体。外键变成联系：联系名用动词（下单、属于），基数从唯一约束判断——外键列有唯一约束是 1，否则是 n；两端都能多则中间还有一张表，不要画成一条无中间实体的多对多，除非代码里没有中间表。

Chen 图只放主键、外键和最多 3 个能说明联系的业务属性。完整字段留给数据结构图。没有 schema 就不画。

## 布局

实体矩形 160×44，`rounded=0`，13px 加粗。联系菱形 `shape=rhombus`，140×72，放在两实体之间。属性 `ellipse`，100×40，从实体向外辐射，间距不少于 16。主键 `fontStyle=4`（下划线）。实体到联系的连线 `endArrow=none;strokeWidth=1.5`，`value` 写 `1` 或 `n`。属性连线 `strokeWidth=1;endArrow=none`。全部白底 `#FFFFFF`、描边 `#333333`。图题在最低图形之下 36px。

## 骨架

```xml
<mxfile host="app.diagrams.net">
  <diagram name="E-R" id="er-chen">
    <mxGraphModel dx="1100" dy="640" grid="0" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1100" pageHeight="640" background="#FFFFFF">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <mxCell id="2" value="用户" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontStyle=1;fontSize=13;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="80" y="200" width="160" height="44" as="geometry"/>
        </mxCell>
        <mxCell id="3" value="用户编号" style="ellipse;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontStyle=4;fontSize=12;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="40" y="80" width="100" height="40" as="geometry"/>
        </mxCell>
        <mxCell id="4" value="下单" style="rhombus;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontSize=12;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="360" y="186" width="140" height="72" as="geometry"/>
        </mxCell>
        <mxCell id="5" value="订单" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontStyle=1;fontSize=13;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="620" y="200" width="160" height="44" as="geometry"/>
        </mxCell>
        <mxCell id="6" value="1" style="endArrow=none;html=1;strokeColor=#333333;strokeWidth=1.5;fontStyle=1;fontSize=12;fontColor=#333333;" edge="1" parent="1" source="2" target="4">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="7" value="n" style="endArrow=none;html=1;strokeColor=#333333;strokeWidth=1.5;fontStyle=1;fontSize=12;fontColor=#333333;" edge="1" parent="1" source="4" target="5">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="8" value="" style="endArrow=none;html=1;strokeColor=#333333;strokeWidth=1;" edge="1" parent="1" source="3" target="2">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="9" value="图 x-y 系统 E-R 图" style="text;html=1;align=center;fontSize=13;fontStyle=2;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="300" y="560" width="400" height="24" as="geometry"/>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

主键属性用 `fontStyle=4`。外键属性不要加下划线，在属性名后加「（FK）」即可。
