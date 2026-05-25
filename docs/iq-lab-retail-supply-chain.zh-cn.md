---
title: "IQ 动手实验：零售供应链"
description: 一个 60 分钟的动手实验，带你体验 Fabric IQ 的本体、图探索、GQL 查询和 Data Agent。
ms.date: 2026-02-16
ms.topic: tutorial
ai-usage: ai-assisted
---

## 概述

在这个约 60 分钟的动手实验中，你会通过一个零售供应链场景，体验 [Fabric IQ（预览版）](https://learn.microsoft.com/fabric/iq/overview) 的核心能力。环境初始化 notebook 会自动创建实验所需的项目，你只需要把重点放在本体概念、图可视化、GQL 查询以及 Data Agent 交互上。

## 场景概览

假设你在一家虚构的零售公司工作，公司在多个区域管理订单、产品、客户、仓库和配送，同时还会跟踪库存、需求信号、预测和促销活动。本实验会带你看看 Fabric IQ 如何用本体把这些数据统一起来，以图的形式呈现，并支持通过 GQL 和对话式 Data Agent 进行查询。

### 为什么选择零售供应链

零售供应链很适合用来演示本体能力，因为它能代表企业里常见的几类数据问题：

- **数据散落在多个系统里。** 订单可能进入 Eventhouse 做实时分析，而产品目录和客户资料可能放在 Lakehouse 里做批处理。传统做法下，分析师得先搞清楚哪些数据在哪个系统里，再自己想办法跨系统关联。
- **关系才是洞察的关键。** 例如“西南区域有哪些促销活动带来了退货？”这样一个问题，就需要从退货一路关联到产品、促销和区域。用普通表来做会比较绕，而图可以直接表达这些关系。
- **业务语言和技术字段不是一回事。** `cust_lt_val` 或 `prod_cat_id` 这样的列名不符合业务用户平时的表达方式。本体可以把这些技术字段映射成 *Customer lifetime value*、*Product category* 这样的业务概念，让大家用同一套语言沟通。
- **不同角色都需要使用这些数据。** 数据工程师关心架构和沿袭，分析师想用可视化方式探索关系，业务用户则希望直接用自然语言提问。本体、图和 Data Agent 可以基于同一套数据，满足不同角色的使用方式。

本实验会用一组接近真实业务的零售实体和关系，带你从原始表一路看到语义模型，再看看对话式 AI 如何使用这些数据。

### 实体和关系

本实验的本体包括以下实体类型：

| 实体类型 | 说明 |
| --- | --- |
| *Order* | 客户购买交易 |
| *OrderLine* | 订单中的单个明细项 |
| *Customer* | 与订单关联的下单客户 |
| *Product* | 可供销售的商品 |
| *ProductCategory* | 相关产品的分组 |
| *Store* | 产生订单的零售门店 |
| *Region* | 包含门店和仓库的地理区域 |
| *Warehouse* | 发出订单的履约中心 |
| *Carrier* | 处理配送的物流服务商 |
| *Shipment* | 将订单关联到物流服务商的配送记录 |
| *Inventory* | 仓库中的库存水平 |
| *Forecast* | 产品的预测需求 |
| *DemandSignal* | 客户需求的实时指标 |
| *Promotion* | 影响产品销售的营销活动 |
| *Return* | 与订单关联的退货项目 |

关键关系包括 *Order* &rarr; *Customer*、*Order* &rarr; *Region*、*OrderLine* &rarr; *Product*、*Product* &rarr; *ProductCategory*、*Shipment* &rarr; *Carrier* 和 *Shipment* &rarr; *Warehouse*。

## 实验时间线

| 时间 | 活动 | 章节 |
| --- | --- | --- |
| 0 到 5 分钟 | 运行设置 notebook | [步骤 1：部署环境](#步骤-1部署环境) |
| 5 到 20 分钟 | 介绍 IQ 和本体概念 | [步骤 2：理解 IQ 和本体概念](#步骤-2理解-iq-和本体概念) |
| 20 到 35 分钟 | 图刷新期间，先探索已部署项目 | [步骤 3：探索已部署项目](#步骤-3探索已部署项目) |
| 35 到 45 分钟 | 探索图 UI | [步骤 4：探索图](#步骤-4探索图) |
| 45 到 55 分钟 | 运行 GQL 查询 | [步骤 5：运行 GQL 查询](#步骤-5运行-gql-查询) |
| 55 到 58 分钟 | Data Agent 演示 | [步骤 6：试用 Data Agent](#步骤-6试用-data-agent) |
| 58 到 60 分钟 | 总结和路线图 | [步骤 7：总结](#步骤-7总结) |

## 步骤 1：部署环境

一个 Fabric notebook 会一次性创建本实验需要的项目，包括：

- 名为 **EHRetail** 的 Eventhouse，其中包含预置好的表
- 名为 **LHRetail** 的 Lakehouse，其中包含辅助数据
- 一个包含实体类型、关系类型和数据绑定的本体

开始之前，请先确认你的租户已经启用本体功能。如果你在 Fabric 里看不到本体相关选项，请让租户管理员开启 Fabric IQ ontology 预览设置。

![显示在 Fabric 中启用本体功能的租户设置截图。](../images/1.prequsite-tenant-enable-ontology.png)

### 上传并运行设置 notebook

<!-- TODO: Add notebook source location (GitHub repo URL, pre-loaded workspace, or download link) -->

1. 下载实验讲师提供的设置 notebook 文件（*.ipynb*）。
1. 打开你的 Fabric 工作区。
1. 选择 **+ New item** > **Import notebook**。
1. 选择 **Upload**，浏览并选择你下载的 notebook 文件，然后选择 **Open**。
1. 等待上传完成。上传后，该 notebook 会显示在工作区项目列表中。
1. 打开上传的 notebook。
1. 选择 **Run all** 执行 notebook。

   notebook 会自动完成以下操作：

   - 创建 Eventhouse 和 Lakehouse 项目
   - 为所有表生成零售示例数据
   - 创建包含实体类型和关系类型的本体
   - 将实体类型绑定到数据源
   - 触发图刷新

   > [!IMPORTANT]
   > 图刷新会在后台运行，通常需要大约 10 到 15 分钟。这个过程无法手动加速，刷新期间你可以继续完成后面的步骤。

1. 确认 notebook 的所有单元格都已成功运行且没有报错。你应该能看到每个项目创建步骤的成功消息。

   ![显示设置 notebook 正在运行，并可使用 Copilot 帮助解决错误的截图。](../images/2.setup-notebook-with-copilot-resolve-error.png)

## 步骤 2：理解 IQ 和本体概念

图刷新在后台运行时，可以先花一点时间了解 Fabric IQ 背后的核心概念。

这里是占位内容。现场实验时，讲师会在这个时间段讲解 PowerPoint，请学员跟随讲师节奏。

## 步骤 3：探索已部署项目

等待图刷新完成时，可以先看看 notebook 创建了哪些项目。

### 查看 Eventhouse 表

1. 在 Fabric 工作区中，选择 **EHRetail** Eventhouse。
1. 打开数据库并查看以下表：

   | 表 | 说明 |
   | --- | --- |
   | *carriers* | 承运商 ID、服务类型代码和覆盖区域标识符 |
   | *customers* | 客户 ID、忠诚度等级代码、生命周期价值分数和联系方式字段 |
   | *demand_signals* | 产品和区域 ID 组合、信号强度值以及时间戳 |
   | *forecasts* | 产品 ID、预测需求数量和预测周期日期 |
   | *inventories* | 仓库和产品 ID 组合、库存数量以及补货阈值 |
   | *products* | 产品 ID、折扣百分比和生效日期（事实/事务数据） |
   | *regions* | 区域代码、时区偏移以及冷链标记等运营标志 |
   | *shipments* | 配送 ID、订单/承运商/仓库外键和状态代码 |
   | *stores* | 门店 ID、位置坐标和运营属性 |

1. 选择任意表并查看原始数据。注意，这些是“原始”的运营数据：列名不一定适合业务用户阅读，很多值也可能只是 ID，而不是有业务含义的名称。这正是本体要解决的问题：在原始数据之上补上一层业务语义。

### 查看 Lakehouse

1. 返回工作区并打开 **LHRetail** Lakehouse。
1. 在 **Lakehouse explorer** 窗格中展开 **Tables** 文件夹。你应该能看到 notebook 根据零售示例数据创建的 Delta 表。这些表会作为本体的额外数据源。
1. 选择一个表（例如 *orders* 或 *returns*）预览其内容。将这里的结构与 Eventhouse 表进行比较。注意：

   - Lakehouse 表可能包含不同的列，也可能提供 Eventhouse 表中没有的补充数据。
   - 本体中的某些实体类型会绑定到 Lakehouse 表，而不是 Eventhouse 表。这个设计可以展示本体如何统一多个存储引擎中的数据。

1. 比较两个引擎中的 *products* 数据：

   - 在 Lakehouse 中，*products* 表包含**维度数据**，也就是描述每个产品的静态属性，例如名称、类别和价格。
   - 在 Eventhouse 中，*products* 表包含**事实数据**，也就是不同时间点应用到不同产品上的折扣记录。
   - 这两个表其实是在描述同一个业务概念（*Product*）的不同侧面。本体会把它们统一到同一个 *Product* 实体类型下，因此 Graph 查询和 Data Agent 等下游使用者不需要关心每个字段来自哪个引擎。

1. 查看 **Table details**，了解行数、架构和列类型。这有助于你理解本体所处理的数据规模。

   > [!TIP]
   > Lakehouse 和 Eventhouse 在这个场景中各有分工。Eventhouse 存放适合实时查询的运营和流式数据，Lakehouse 存放批处理和历史数据。本体会把它们统一到同一组业务概念之下。

### 探索本体结构

1. 返回工作区并打开 **Ontology** 项目。
1. 在 **Entity Types** 窗格中查看实体类型列表。你应该能看到[场景概览](#场景概览)中列出的全部 15 个实体类型。
1. 先选择 **Product** 实体类型。前面你已经看到，产品数据分布在两个位置：Lakehouse 保存维度属性（名称、类别、价格），Eventhouse 保存事实数据（折扣记录）。现在看看本体如何把它们整合起来：

   ![显示在 Entity Types 窗格中选中 Product 实体类型并显示其属性的截图。](./media/iq-lab-retail-supply-chain/product-entity-view.png)

   1. 在 **Entity type configuration** 右侧窗格中查看属性和绑定。
   1. 查看 *Product* 实体类型的属性列表。注意，部分属性绑定到 Lakehouse *products* 表中的列（例如产品名称、类别和单位成本），其他属性则绑定到 Eventhouse *products* 表中的列（例如折扣百分比和日期）。
   1. 选择 *Product* 的 **Bindings** 选项卡。此选项卡会显示绑定到该实体的每个源表卡片。

   ![显示 Product 实体类型 Bindings 选项卡，并包含 Lakehouse 和 Eventhouse 数据源卡片的截图。](./media/iq-lab-retail-supply-chain/entity-type-bindings.png)

   > [!NOTE]
   > 这就是本体的核心价值：一个 *Product* 实体类型就能呈现这个业务概念的统一视图，即使背后的数据来自两个不同的存储引擎。本体的使用者（Graph、GQL 和 Data Agent）不需要知道每个属性具体来自哪里。

1. 现在探索连接到 *Product* 的关系。

   - **ProductInCategory** - 将 *Product* 链接到对应的 *ProductCategory*，这样你就能回答“哪些产品属于冷冻食品类别？”之类的问题。
   - **InventoryForProduct** - 将 *Inventory* 记录链接到 *Product*，显示该产品在各个仓库中的库存水平。
   - **PromotionForProduct** - 将 *Promotion* 活动链接到目标 *Product*，把营销活动和具体商品关联起来。

   这些关系会把原本分散的表连成一张图。你从一个 *Product* 节点出发，就可以找到它的类别、各仓库库存，以及正在进行的促销活动，不需要自己写 JOIN，也不需要知道每张表存在哪个引擎里。

1. 查看其余关系类型，了解更广泛的图结构：

   - *OrderPlacedByCustomer*（下单客户）
   - *OrderFulfilledToRegion*（订单履约区域）
   - *OrderHasLineItem*（订单包含的内容）
   - *ShipmentFulfillsOrder*（哪个配送履约了订单）
   - *ShipmentDepartedFromWarehouse*（库存所在位置）

1. 返回 **Data Bindings** 视图，继续查看更多实体类型的绑定。例如，*Customer* 实体类型会绑定到 EHRetail 中的 *customers* 表，也会绑定到 Lakehouse 中的 *customers* 表。你可以通过图标来判断绑定类型。

> [!TIP]
> Fabric IQ agent 会根据数据绑定类型调整推理方式。在后面的 Data Agent 步骤之前，先理解这个差异会很有帮助。
>
> **静态绑定**指向维度数据或变化不频繁的数据，例如客户名称、产品类别或区域时区。当 agent 使用静态绑定回答问题时，它会：
>
> - 把属性值当作当前事实。
> - 不做时间维度上的推理，也就是说它不能解释某个值是*何时*变化的，也不能解释*为什么*变化。
> - 返回类似这样的答案：*“Customer CUST000297 的 loyalty tier 是 Gold。”*
>
> **时间序列绑定**指向随时间变化的数据，例如传感器读数、事务日志，或带时间戳的需求信号。当 agent 使用时间序列绑定回答问题时，它可以：
>
> - 跟踪按时间顺序排列的观察结果之间的因果关系。
> - 解释某个状态或条件出现的原因。
> - 推理异常、违规和趋势。
> - 返回类似这样的答案：*“由于来自 southwest 区域的三次连续高信号读数，产品 PROD00042 的需求在 14:30 激增。”*
>
> 这种差异很重要，因为同一个问题在不同绑定类型下可能得到完全不同的答案。产品状态的静态绑定只会返回一个简单事实（*“该产品有库存”*），而库存水平的时间序列绑定则可以让 agent 解释库存为什么下降，以及下降发生在什么时候。
>
> 在本实验中，大多数绑定都是静态绑定（Lakehouse 中的维度表，以及 Eventhouse 中的事实快照）。当你在[步骤 6](#步骤-6试用-data-agent)中试用 Data Agent 时，请记住这个区别。agent 的回答会受到本体中可用绑定类型的影响。另外，目前 Data Agent 只使用 NL->GQL。

## 步骤 4：探索图

图刷新完成后，你就可以通过可视化界面探索这些已经连接起来的数据。

### 检查图刷新状态

继续之前，请先确认设置 notebook 触发的图刷新已经完成。

1. 在左侧 Fabric 导航栏中，选择 **Monitor** 图标。
1. 在监视视图中，找到与你的本体相关的图刷新活动。

   ![导航栏监视功能截图。](./media/iq-lab-retail-supply-chain/monitor.png)

1. 如果刷新仍在进行，请等待完成后再继续。图刷新最多可能需要 15 分钟，具体时间取决于数据量和当前容量使用情况。
1. 状态显示为 **Completed** 后，返回本体开始探索图。

### 导航图视图

1. 从 **Product** 实体打开 **Entity type overview**。

   ![显示 Product 的 Entity type overview 页面以及数据和关系预览卡片的截图。](./media/iq-lab-retail-supply-chain/entity-overview.png)

1. 浏览页面中与此实体相关的各个预览区域。
1. 在左上角卡片中，选择 **Relationship Graph** 上的 **Expand**。这会打开一个以 *Product* 节点为中心的图视图。

   ![显示扩展图视图的截图，其中节点和边表示实体和关系。](./media/iq-lab-retail-supply-chain/graph-view-expanded.png)

1. 图视图会把实体实例显示为节点，把关系显示为连接节点的边。你可以：

   - **平移和缩放**来浏览图。
   - **选择节点**来查看其属性和连接的实体。
   - **沿着边移动**来遍历关系。

1. 试着看看当前 **Product** 节点如何连接到区域。选中节点后，节点周围会展开一组工具。选择 **+** 号添加一个相连节点。在这个例子中，唯一可选项是 **Order**，选择它即可。接着对 **Order** 节点重复同样操作，这次添加相连的 **Region** 节点。类似地，你也可以单击某个节点，然后选择 **X** 将其从图视图中移除。注意，添加或移除节点时，对应的边也会一起添加或移除。

![显示从 Product 到 Order 再到 Region 添加节点，然后移除 Order 节点并看到连接到 Region 的边也被移除的 GIF。](./media/iq-lab-retail-supply-chain/add-nodes.gif)

   > [!IMPORTANT]
   > 在图探索过程中，请不要对本体做任何数据或架构更改。图刷新成本较高且耗时较长，任何更改都可能触发新一轮刷新。

### 使用 UI 查询图

现在使用可视化查询生成器运行查询。我们要查找的是：哪些仍在进行的促销活动，关联到了发生退货的产品。

1. 从 Product 菜单栏中选择 **Clear query**。
1. 在图查询 UI 中，从右侧窗格选择 **Return** &rarr; **Product** &rarr; **Promotion** 节点来构建路径。
1. 将连接它们的边 *ReturnForProduct* 和 *PromotionForProduct* 添加到路径中。这样查询引擎就会从退货记录出发，经过关联产品，再遍历到该产品对应的促销活动。
1. 在 **Promotion** 节点上添加筛选器：将 **IsActivePromotion** 设置为 **true**。这样结果就只会保留当前仍在进行的促销。

   ![显示在 Promotion 节点上将 IsActivePromotion 设置为 true 的筛选器配置截图。](./media/iq-lab-retail-supply-chain/add-filter.png)

1. 选择 **Apply** 确认查询配置。
1. 选择 **Run** 执行查询。
1. 查看结果。你应该能看到一组仍在进行的促销活动，而且这些促销对应的产品也发生过退货。像这样把退货关联回营销活动，在普通表查询里通常比较麻烦；但当数据建模成图以后，就会直观很多。

   ![显示应用 Return 到 Product 到 Promotion 路径及有效促销筛选器后的查询结果截图。](./media/iq-lab-retail-supply-chain/query-built-in-ui.png)

1. 选择 **Query builder** &rarr; **Code editor** 查看可视化生成器生成的 GQL 查询。这里会显示与你配置的路径和筛选器对应的 GQL 语法。进入下一步手写 GQL 之前，可以先花一点时间看看查询结构。

## 步骤 5：运行 GQL 查询

在此步骤中，你将使用 GQL（Graph Query Language，图查询语言）直接查询本体图。写查询之前，先简单了解一下 GQL 是什么，以及为什么适合这个场景。

### 什么是 GQL

GQL 是面向图数据库的 ISO 标准查询语言（[ISO/IEC 39075](https://www.iso.org/standard/76120.html)）。负责 SQL 的同一个 ISO 工作组也在制定 GQL，所以如果你写过 SQL，会看到一些熟悉的概念，例如表达式、谓词、类型和筛选。

关键差异在于关系的表达方式。GQL 不需要用 JOIN 语句把表关联起来，而是使用**可视化图模式**直接描述你想遍历的连接。这样查询更易读，也更贴近我们理解关联数据的方式。

### 为什么使用 GQL 查询本体

传统 SQL 很擅长在单个表中查找数据，或者关联少量相关表。但是，当你需要连续遍历很多关系时，例如查找某个区域中哪些促销活动带来了退货，SQL 查询往往会变成一长串复杂 JOIN，写起来累，后续维护也不轻松。

GQL 的思路是把关系放到查询的核心位置。一个在 SQL 中可能需要五六个 JOIN 的查询，在 GQL 中可以写成一个图模式，读起来几乎像一句话：“查找下过订单，并且订单中包含特定类别产品的客户。”

在 Fabric IQ 中，GQL 让你能够：

- **自然遍历本体关系。** `(node:EntityType)-[edge:RelationshipType]->(node:EntityType)` 这种模式，和你在本体中定义关系的方式是一致的。
- **不用关心数据源差异。** 同一个 GQL 查询可以从 Eventhouse 中的数据遍历到 Lakehouse 中的数据，你不需要知道哪个实体存在哪个引擎里。
- **简洁表达复杂业务问题。** “西南区域中哪些客户购买了促销产品，而且这些产品发生过退货？”这样的问题，可以写成一个可读性很高的查询。

### GQL 目前支持的功能

Fabric IQ 中的 GQL 目前支持：

- 使用 **MATCH 模式**跨实体类型和关系类型遍历节点与边
- 使用 **WHERE 子句**按属性值筛选结果
- 使用 **RETURN 语句**指定输出内容
- 在单个查询中组合多条关系路径
- 用于计数、求和和分组结果的**聚合**

### 当前限制

GQL 支持仍处于预览阶段，需要注意以下限制：

- **查询复杂度限制。** 很长的遍历路径（超过六七跳）可能会超时，或者返回不完整结果。
- **不能通过 GQL 修改架构。** 你不能使用 GQL 创建或修改实体类型、关系类型或绑定。架构变更请通过本体 UI 完成。
- **函数支持有限。** ISO 标准中的某些高级 GQL 函数目前还不可用。

如需查看 Fabric 中完整的 GQL 语言参考，请参阅 [GQL language guide](https://learn.microsoft.com/fabric/graph/gql-language-guide)。

### 运行第一个 GQL 查询

1. 选择 **Clear query** 重置图查询。
1. 将以下 GQL 查询复制并粘贴到代码编辑器中。这个查询会查找西南区域中所有购买过 household 类别产品的客户。它会从 *Customer* 经由 *Order* 和 *OrderLine* 遍历到 *Product*，然后再关联到 *ProductCategory* 和 *Region* 来应用筛选条件。

   ```gql
   MATCH (node_Order:`Order`)-[edge1_OrderFulfilledToRegion:`OrderFulfilledToRegion`]->(node_Region:`Region`),
       (node_Order:`Order`)-[edge2_OrderPlacedByCustomer:`OrderPlacedByCustomer`]->(node_Customer:`Customer`),
       (node_OrderLine:`OrderLine`)-[edge3_OrderHasLineItem:`OrderHasLineItem`]->(node_Order:`Order`),
       (node_OrderLine:`OrderLine`)-[edge4_OrderLineReferencesProduct:`OrderLineReferencesProduct`]->(node_Product:`Product`),
       (node_Product:`Product`)-[edge5_ProductInCategory:`ProductInCategory`]->(node_ProductCategory:`ProductCategory`)
   WHERE node_Region.`RegionName` = "southwest"
       AND node_ProductCategory.`CategoryName` = "household"
   RETURN TO_JSON_STRING(node_Customer) AS `Customer`
   ```

1. 选择 **Run** 执行查询。
1. 查看结果。查询会返回西南区域中订单包含 household 类别产品的客户。注意 GQL 模式是如何映射到你之前探索过的本体关系的：每个 `[edge:RelationshipType]` 都对应本体中定义的一个关系类型。

   ![显示本体 GQL 查询示例及其结果的截图。](../images/3.ontology-query-sample.png)

> [!TIP]
> 查看查询结果时，请留意 GQL 是如何遍历关系的。`(node:EntityType)-[edge:RelationshipType]->(node:EntityType)` 这种模式对应的就是本体结构。这就是图语义模型的价值：查询可以沿着业务概念之间的自然关系向外展开。

## 步骤 6：试用 Data Agent

在此步骤中，你会看到 Fabric Data Agent 如何借助本体回答自然语言问题。

> [!NOTE]
> Data Agent 当前处于预览阶段，需要注意以下行为：
>
> - 第一次查询通常会更慢，也可能超时。如果遇到这种情况，请再运行一次。
> - 同一个问题在不同运行之间，结果可能不完全一致。
> - 复杂问题可能得不到理想答案。
>
> 这个步骤主要用于演示产品能力的发展方向，还不能当作生产就绪能力来理解。

### 创建 Data Agent

1. 返回工作区。
1. 选择 **+ New item**，然后选择 **Data Agent**。
1. 为 agent 命名（例如 *RetailSupplyChainAgent*）。
1. 在 agent 配置中，将本体添加为数据源。

### 提问

一次尝试一个问题。输入每个问题后，等待返回结果，再继续提出下一个问题。

**问题 1：**

> What is the loyalty tier and lifetime value for Customer CUST000297?

查看返回结果。agent 应该会通过本体查询，并返回该客户的忠诚度等级和生命周期价值。

![显示 Fabric Data Agent 针对零售供应链问题返回示例响应的截图。](../images/4.data-agent-sample.png)

**问题 2：**

> Is Region southwest marked as cold-chain required, and what is its timezone?

这个问题用来测试 agent 查找 Region 实体中特定属性的能力。

**问题 3：**

> For Customer CUST000297, show the customer's region name and the latest customer engagement score.

这个问题要求 agent 从 Customer 沿关系遍历到 Region，用来展示跨实体推理能力。

> [!NOTE]
> 如果查询超时或返回错误，请再运行一次。新的 Data Agent 会话中，第一次查询经常失败，这是预览版的已知限制。

## 步骤 7：总结

你已经完整体验了一条 IQ 工作流：从本体创建和跨源数据绑定，到图遍历、GQL 查询，再到对话式 Data Agent。下面简单总结一下 IQ 目前可以完成的工作。

### 目前可用的功能

<!-- TODO: Fill in based on current GA/preview status -->

- 使用实体类型、属性和关系类型创建本体
- 绑定 Eventhouse 和 Lakehouse 中的数据
- 图可视化和探索
- GQL 查询
- 使用本体作为数据源的 Data Agent

### 已知限制

<!-- TODO: Fill in based on current known issues -->

- 图刷新耗时较长，而且完成时间不完全可预测
- Data Agent 查询可能超时，尤其是首次运行时
- Data Agent 的结果在不同运行之间可能不完全一致
- 复杂自然语言查询可能得不到理想结果

### 后续方向

<!-- TODO: Fill in based on product roadmap -->

- 改进图刷新性能
- 提升 Data Agent 的可靠性和查询覆盖范围
- 增强 GQL 能力

## 清理资源

如果你已经完成实验，可以删除工作区，或删除本实验中创建的各个项目，避免继续占用容量：

1. 导航到你的工作区。
1. 选择实验期间创建的项目（**EHRetail**、Lakehouse、本体和 Data Agent）。
1. 删除选中的项目。

## 相关内容

- [什么是 Fabric IQ（预览版）？](https://learn.microsoft.com/fabric/iq/overview)
- [什么是本体（预览版）？](https://learn.microsoft.com/fabric/iq/ontology/overview)
- [本体（预览版）教程](https://learn.microsoft.com/fabric/iq/ontology/tutorial-0-introduction)
- [Microsoft Fabric 中的图](https://learn.microsoft.com/fabric/graph/overview)
- [GQL 语言指南](https://learn.microsoft.com/fabric/graph/gql-language-guide)
- [GQL 表达式、谓词和函数](https://learn.microsoft.com/fabric/graph/gql-expressions)
- [Fabric data agent](https://learn.microsoft.com/fabric/data-science/concept-data-agent)
