# 系统架构图（四层分层风格）

采用四层横向分层（表现层、接口层、业务逻辑层、数据持久层），展示系统整体架构和调用关系。

## 使用时机

- 用户需要展示「从前端到后端再到数据库」的整体架构分层
- 用户提到「系统架构图」「总体架构」「分层结构」「端到端调用流程」
- 设计文档、答辩 PPT 或论文中需要宏观架构示意图

## 推荐版式（黑白学术风格）

1. **自上而下四层**：表现层、接口层、业务逻辑层、数据持久层，每层为横条大矩形背景
2. **层内组件**：每层内 2–4 个白底矩形，代表关键组件
3. **垂直箭头**：表示调用流程（表现层 → 接口层 → 业务逻辑层 → 数据持久层 → 数据库）
4. **最底层**：圆柱图标（`shape=cylinder3`）表示数据库
5. **右侧流程标注**（可选）：大箭头向下，标注「调用流程」
6. **图题**：预留「图 x-y 系统架构图」

## 黑白配色（必须遵守）

- 层背景：`fillColor=#F5F5F5`，`strokeColor=#333333`
- 组件：`fillColor=#FFFFFF`，`strokeColor=#333333`
- 字体：`fontColor=#333333`，Helvetica 12 号
- 连接线：`strokeColor=#333333`，`endArrow=blockThin`
- 数据库圆柱：`shape=cylinder3`，白底深灰边框

## 工作流程

1. **识别系统分层**：从项目识别四层结构及每层关键组件
2. **梳理垂直调用关系**：明确从上到下的调用流程
3. **规划布局**：画布约 900×680px，每层高约 100px
4. **生成 .drawio**：四层大矩形、层内组件、垂直箭头、数据库圆柱
5. **文字说明与图题**：四层职责与调用流程说明

## 注意事项

- 经典 Web 系统默认四层。纯训练仓库、单进程推理或事件驱动系统若没有这四层，按真实进程边界画横向带，并在说明里写明为什么不套四层。禁止编造不存在的层。
- 保持「组件/服务」粒度，不细化到类或方法。模型内部结构走 `drawio-diagram` 的技术图，不画在这张分层图里。

## 从代码抽取

| 层 | 优先看 |
|----|--------|
| 表现层 | `src/routes`、`pages`、`app` 下的页面入口，而不是每个组件 |
| 接口层 | 路由注册、`openapi`、gRPC `.proto`、`urls` |
| 业务逻辑层 | `service`、`usecase`、训练循环、推理 `pipeline` 的公开入口 |
| 数据持久层 | ORM 会话、repository、SQL 文件、对象存储客户端 |
| 数据库 | `docker-compose`、迁移目录、连接配置里的引擎名 |

调用箭头只表达真实方向：页面调用接口、接口调用业务、业务调用存储。反向依赖（业务直接 import 页面）不要画成正向箭头，在说明里指出。

## 布局

画布 900×720，`grid="0"`，背景白。四条带 y 分别为 24、156、288、420，高 112，x=40，宽 820，填充 `#F5F5F5`。带内组件白底，高 48，y 比带顶多 32，同一带内等宽并排，间距 24。数据库 `shape=cylinder3`，x=390，y=552，宽 120，高 72。层间箭头在 x=450，`endArrow=blockThin`。图题 y=660。

## 骨架

层名和组件名换成代码里的真实名称。下面只示范一条带、一个组件和向下的箭头，其余层按同一 x/宽复制。

```xml
<mxfile host="app.diagrams.net">
  <diagram name="系统架构" id="sys-arch">
    <mxGraphModel dx="900" dy="720" grid="0" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="900" pageHeight="720" background="#FFFFFF">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <mxCell id="2" value="表现层" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#F5F5F5;strokeColor=#333333;fontColor=#333333;fontSize=12;align=left;spacingLeft=8;verticalAlign=top;" vertex="1" parent="1">
          <mxGeometry x="40" y="24" width="820" height="112" as="geometry"/>
        </mxCell>
        <mxCell id="3" value="Web 页面" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#333333;fontColor=#333333;fontSize=12;fontStyle=1;" vertex="1" parent="1">
          <mxGeometry x="80" y="56" width="200" height="48" as="geometry"/>
        </mxCell>
        <mxCell id="4" value="" style="endArrow=blockThin;html=1;strokeColor=#333333;strokeWidth=1.5;" edge="1" parent="1">
          <mxGeometry relative="1" as="geometry">
            <mxPoint x="450" y="136" as="sourcePoint"/>
            <mxPoint x="450" y="156" as="targetPoint"/>
          </mxGeometry>
        </mxCell>
        <mxCell id="5" value="数据库" style="shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;backgroundOutline=1;size=12;fillColor=#FFFFFF;strokeColor=#333333;fontColor=#333333;fontSize=12;" vertex="1" parent="1">
          <mxGeometry x="390" y="552" width="120" height="72" as="geometry"/>
        </mxCell>
        <mxCell id="6" value="图 x-y 系统架构图" style="text;html=1;align=center;fontSize=13;fontStyle=2;fontColor=#333333;" vertex="1" parent="1">
          <mxGeometry x="250" y="660" width="400" height="24" as="geometry"/>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```
