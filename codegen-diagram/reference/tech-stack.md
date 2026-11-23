# 技术栈结构图

从前端、后端、模型服务、基础设施等维度梳理所用技术与依赖关系。

## 使用时机

- 用户需要说明系统用到了哪些技术、库、框架和基础设施
- 用户提到「技术栈结构图」「整体技术栈」「组件选型示意图」
- 撰写设计文档、汇报 PPT 或论文时需要总览性技术栈图

## 推荐版式

1. **顶层**：3 个主要组件横向排列（如「前端应用」「后端服务」「模型服务」），圆角矩形，白底深灰边框
2. **组件之间**：标注通信方式（HTTP/HTTPS、RESTful API、JSON 等），浅灰底 #F5F5F5
3. **每个组件下方**：虚线连接到「技术栈」框，框内列表形式列出该组件所用技术
4. **图题**：预留「图 x-y 系统技术栈结构图」

## 工作流程

1. **识别技术栈组件**：从项目结构和配置提取，归纳为 3 个顶层组件，为每个列出技术栈清单
2. **梳理通信关系**：明确组件间通信方式
3. **生成 .drawio**：按版式生成完整 mxGraph XML
4. **文字总览与图题**：学术风格概述，建议图题与引用写法

## 样式要点

- 顶层组件：`rounded=1;fillColor=#FFFFFF;strokeColor=#333333;fontStyle=1`
- 通信标注：`fillColor=#F5F5F5`
- 虚线：`dashed=1;strokeColor=#333333;endArrow=none`
- 技术栈框：标题「技术栈」，框内多行列表，左对齐

## 从代码抽取

按这个顺序找，只保留直接依赖里的框架和运行时，不展开传递依赖：

| 来源 | 看什么 |
|------|--------|
| `package.json`、`pyproject.toml`、`requirements.txt`、`go.mod`、`Cargo.toml` | 框架、运行时、模型库 |
| `Dockerfile`、`docker-compose`、部署清单 | 独立进程、数据库、队列、对象存储 |
| 入口（`main`、`server`、`train.py`、`serve`） | 顶层怎么切块 |

有界面时三块为「前端 / 业务后端 / 模型或数据服务」。纯训练仓库改为「数据与特征 / 训练 / 导出与推理」。某一块在仓库里不存在就不要硬凑，改成两块并写明原因。通信只写代码里出现的协议；猜的标「待确认」。

## 布局

画布 1000×640，背景白，`grid="0"`。三块顶层 y=40、高 56、宽 200，x 为 40、380、720。技术栈框与顶层同 x，y=220、宽 240，左对齐，`spacingLeft=8`。顶层底边中点到技术栈框顶边用虚线，`dashed=1;endArrow=none;strokeColor=#333333`。块之间的连线标签写协议，12px。图题 y=560。

## 骨架

名称换成仓库里的真实技术。虚线不带箭头，协议写在实线标签上。

```xml
<mxfile host="app.diagrams.net">
  <diagram name="技术栈" id="tech-stack">
    <mxGraphModel dx="1000" dy="640" grid="0" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1000" pageHeight="640" background="#FFFFFF">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <mxCell id="2" value="业务后端" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontStyle=1;fontSize=13;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="380" y="40" width="200" height="56" as="geometry"/>
        </mxCell>
        <mxCell id="3" value="模型服务" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontStyle=1;fontSize=13;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="720" y="40" width="200" height="56" as="geometry"/>
        </mxCell>
        <mxCell id="4" value="HTTP / JSON" style="endArrow=blockThin;html=1;strokeColor=#333333;fontSize=12;fontColor=#333333;labelBackgroundColor=#F5F5F5;" edge="1" parent="1" source="2" target="3">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="5" value="技术栈&#xa;FastAPI&#xa;PyTorch" style="rounded=0;whiteSpace=wrap;html=1;align=left;spacingLeft=8;fillColor=#FFFFFF;strokeColor=#333333;fontSize=12;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="380" y="220" width="240" height="88" as="geometry"/>
        </mxCell>
        <mxCell id="6" value="" style="endArrow=none;dashed=1;html=1;strokeColor=#333333;exitX=0.5;exitY=1;exitDx=0;exitDy=0;entryX=0.5;entryY=0;entryDx=0;entryDy=0;" edge="1" parent="1" source="2" target="5">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="7" value="图 x-y 系统技术栈结构图" style="text;html=1;align=center;fontSize=13;fontStyle=2;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="280" y="560" width="440" height="24" as="geometry"/>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```
