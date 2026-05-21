---
title: "IQ 动手实验：零售供应链"
description: 一个 60 分钟的引导演练，探索 Fabric IQ 功能，包括本体、图探索、GQL 查询和 Data Agent。
ms.date: 2026-02-16
ms.topic: tutorial
ai-usage: ai-assisted
---

## 概述

在这个 60 分钟的动手实验中，你将使用零售供应链场景探索 [Fabric IQ（预览版）](https://learn.microsoft.com/fabric/iq/overview) 的核心能力。一个设置 notebook 会自动部署所有必需的项目，因此你可以专注于理解本体概念、图可视化、GQL 查询以及 Data Agent 交互。

## 场景概览

你在一家虚构的零售公司工作，该公司跨多个区域管理订单、产品、客户、仓库和配送。公司会跟踪库存、需求信号、预测和促销活动。你的目标是探索 Fabric IQ 如何使用本体统一这些数据，将其可视化为图，并让它可以通过 GQL 和对话式 Data Agent 进行查询。

### 为什么选择零售供应链

零售供应链非常适合演示本体能力，因为它反映了真实企业面对的复杂性：

- **数据存在于多个系统中。** 订单可能流入 Eventhouse 用于实时分析，而产品目录和客户资料则保存在 Lakehouse 中用于批处理。传统方法要求分析师知道哪些系统保存了哪些数据，以及如何跨系统联接这些数据。
- **关系是洞察的核心。** 一个简单问题，例如“西南区域有哪些促销活动推动了退货？”，需要从退货遍历到产品、促销和区域。扁平表会让这个过程很痛苦，而图可以自然表达这种关系。
- **业务语言不同于技术架构。** `cust_lt_val` 或 `prod_cat_id` 这样的列名并不符合业务用户的思维方式。本体会将技术列映射到 *Customer lifetime value* 和 *Product category* 等概念，让所有人使用同一种业务语言交流。
- **多个角色都需要访问数据。** 数据工程师关注架构和沿袭，分析师希望可视化探索关系，业务用户希望用自然语言提问。本体、图和 Data Agent 都基于同一个基础来服务这些角色。

本实验使用一组逼真的零售实体和关系，带你逐层了解从原始表到语义模型，再到对话式 AI 的完整路径。

### 实体和关系

本实验的本体包括以下实体类型：

| 实体类型 | 说明 |
| --- | --- |
| *Order* | 客户购买交易 |
| *OrderLine* | 订单中的单个明细项 |
| *Customer* | 与订单关联的买家 |
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
| 20 到 35 分钟 | 在图准备完成之前探索已部署项目 | [步骤 3：探索已部署项目](#步骤-3探索已部署项目) |
| 35 到 45 分钟 | 探索图 UI | [步骤 4：探索图](#步骤-4探索图) |
| 45 到 55 分钟 | 运行 GQL 查询 | [步骤 5：运行 GQL 查询](#步骤-5运行-gql-查询) |
| 55 到 58 分钟 | Data Agent 演示 | [步骤 6：试用 Data Agent](#步骤-6试用-data-agent) |
| 58 到 60 分钟 | 总结和路线图 | [步骤 7：总结](#步骤-7总结) |

## 步骤 1：部署环境

一个 Fabric notebook 会部署本实验需要的所有项目，包括：

- 名为 **EHRetail** 的 Eventhouse，其中包含预填充的表
- 名为 **LHRetail** 的 Lakehouse，其中包含支持数据
- 一个包含实体类型、关系类型和数据绑定的本体

开始之前，请确认你的租户已启用本体。如果你在 Fabric 中看不到本体选项，请让租户管理员启用 Fabric IQ ontology 预览设置。

![显示在 Fabric 中启用本体功能的租户设置截图。](../images/1.prequsite-tenant-enable-ontology.png)

### 上传并运行设置 notebook

<!-- TODO: Add notebook source location (GitHub repo URL, pre-loaded workspace, or download link) -->

1. 下载实验讲师提供的设置 notebook 文件（*.ipynb*）。
1. 打开你的 Fabric 工作区。
1. 选择 **+ New item** > **Import notebook**。
1. 选择 **Upload**，浏览并选择你下载的 notebook 文件，然后选择 **Open**。
1. 等待上传完成。该 notebook 会显示在工作区项目列表中。
1. 打开上传的 notebook。
1. 选择 **Run all** 执行 notebook。

   notebook 会自动执行以下操作：

   - 创建 Eventhouse 和 Lakehouse 项目
   - 为所有表生成零售示例数据
   - 创建包含实体类型和关系的本体
   - 将实体类型绑定到数据源
   - 触发图刷新

   > [!IMPORTANT]
   > 图刷新会在后台运行，通常需要大约 10 到 15 分钟才能完成。你无法加速此过程。刷新运行时可以继续执行后续步骤。

1. 确认 notebook 单元格全部完成且没有错误。你应该能看到每个项目创建步骤的成功消息。

   ![显示设置 notebook 正在运行，并可使用 Copilot 帮助解决错误的截图。](../images/2.setup-notebook-with-copilot-resolve-error.png)

## 步骤 2：理解 IQ 和本体概念

当图刷新在后台运行时，请利用这段时间理解 Fabric IQ 背后的核心概念。

THIS IS PLACEHOLDER CONTENT - WE WILL PRESENT THE POWERPOINT AT THIS TIME AND ASK THE PARTICIPANTS TO FOCUS ON THE LAB LEADER.

## 步骤 3：探索已部署项目

等待图刷新完成时，探索设置 notebook 创建的项目。

### 查看 Eventhouse 表

1. 在你的 Fabric 工作区中，选择 **EHRetail** Eventhouse。
1. 打开数据库并查看以下表：

   | 表 | 说明 |
   | --- | --- |
   | *carriers* | 包含承运商 ID、服务类型代码和覆盖区域标识符的行 |
   | *customers* | 包含客户 ID、忠诚度等级代码、生命周期价值分数和联系字段的行 |
   | *demand_signals* | 包含产品和区域 ID 对、信号强度值以及时间戳的行 |
   | *forecasts* | 包含产品 ID、预测需求数量和预测周期日期的行 |
   | *inventories* | 包含仓库和产品 ID 对、库存数量以及补货阈值的行 |
   | *products* | 包含产品 ID、折扣百分比和生效日期的行（事实/事务数据） |
   | *regions* | 包含区域代码、时区偏移以及冷链标记等运营标志的行 |
   | *shipments* | 包含配送 ID、订单/承运商/仓库外键和状态代码的行 |
   | *stores* | 包含门店 ID、位置坐标和运营属性的行 |

1. 选择任意表并查看原始数据。请注意，这是“原始”运营数据，列名可能并不适合业务用户阅读，值也可能是 ID 而不是描述性名称。这正是本体要解决的问题：在数据之上叠加业务含义。

### 查看 Lakehouse

1. 返回工作区并打开 **LHRetail** Lakehouse。
1. 在 **Lakehouse explorer** 窗格中，展开 **Tables** 文件夹。你应该能看到设置 notebook 根据生成的零售数据创建的 Delta 表。这些表会作为本体的额外数据源。
1. 选择一个表（例如 *orders* 或 *returns*）预览其内容。将这里的结构与 Eventhouse 表进行比较。注意：

   - Lakehouse 表可能包含不同的列，或包含补充 Eventhouse 表的额外数据。
   - 本体中的某些实体类型会绑定到 Lakehouse 表，而不是 Eventhouse 表，这展示了本体如何跨多个存储引擎统一数据。

1. 比较两个引擎中的 *products* 数据：

   - 在 Lakehouse 中，*products* 表包含**维度数据**，也就是描述每个产品的静态属性，例如名称、类别和价格。
   - 在 Eventhouse 中，*products* 表包含**事实数据**，也就是关于不同时点应用到不同产品的折扣事务记录。
   - 这两个表代表同一个业务概念（*Product*）的不同侧面。本体会将两者统一到单一的 *Product* 实体类型下，因此 Graph 查询和 Data Agent 等下游使用者不需要知道哪个引擎保存了哪一部分数据。

1. 查看 **Table details**，了解行数、架构和列类型。这有助于你理解本体所处理的数据规模。

   > [!TIP]
   > Lakehouse 和 Eventhouse 在此场景中扮演不同角色。Eventhouse 保存面向实时查询优化的运营和流式数据，Lakehouse 保存批处理和历史数据。本体会将二者统一到同一组业务概念之下。

### 探索本体结构

1. 返回工作区并打开 **Ontology** 项目。
1. 在 **Entity Types** 窗格中，查看实体类型列表。你应该能看到[场景概览](#场景概览)中列出的全部 15 个实体类型。
1. 从选择 **Product** 实体类型开始。在前一步中，你看到产品数据存在于两个位置：Lakehouse 保存维度属性（名称、类别、价格），Eventhouse 保存事实数据（折扣记录）。现在查看本体如何把它们整合起来：

   ![显示在 Entity Types 窗格中选中 Product 实体类型并显示其属性的截图。](./media/iq-lab-retail-supply-chain/product-entity-view.png)

   1. 在 **Entity type configuration** 右侧窗格中查看属性和绑定。
   1. 查看 *Product* 实体类型上的属性列表。注意，部分属性绑定到 Lakehouse *products* 表中的列（例如产品名称、类别和单位成本），而其他属性绑定到 Eventhouse *products* 表中的列（例如折扣百分比和日期）。
   1. 选择 *Product* 的 **Bindings** 选项卡。此选项卡会显示绑定到该实体的每个源表卡片。

   ![显示 Product 实体类型 Bindings 选项卡，并包含 Lakehouse 和 Eventhouse 数据源卡片的截图。](./media/iq-lab-retail-supply-chain/entity-type-bindings.png)

   > [!NOTE]
   > 这是本体的核心价值：单一的 *Product* 实体类型呈现了该概念的统一视图，即使底层数据来自两个不同的存储引擎。本体的使用者（Graph、GQL 和 Data Agent）不需要知道每个属性来自哪里。

1. 现在探索连接到 *Product* 的关系。

   - **ProductInCategory** - 将 *Product* 链接到其 *ProductCategory*，因此你可以回答“哪些产品属于冷冻食品类别？”之类的问题。
   - **InventoryForProduct** - 将 *Inventory* 记录链接到 *Product*，显示该产品在每个仓库中的库存水平。
   - **PromotionForProduct** - 将 *Promotion* 活动链接到其目标 *Product*，把营销活动连接到具体商品。

   这些关系会将孤立的表转换为一个连通图。单个 *Product* 节点可以带你找到它的类别、跨仓库库存以及任何正在进行的促销，而无需编写联接，也无需知道哪些表位于哪个引擎中。

1. 查看其余关系类型，了解更广泛的图结构：

   - *OrderPlacedByCustomer*（下单客户）
   - *OrderFulfilledToRegion*（订单履约区域）
   - *OrderHasLineItem*（订单包含的内容）
   - *ShipmentFulfillsOrder*（哪个配送履约了订单）
   - *ShipmentDepartedFromWarehouse*（库存所在位置）

1. 返回 **Data Bindings** 视图，并探索更多实体类型的绑定。例如，*Customer* 实体类型会绑定到 EHRetail 中的 *customers* 表以及 Lakehouse 中的 *customers* 表。查看图标来理解绑定类型。

> [!TIP]
> Fabric IQ agent 会根据数据绑定类型改变其推理策略。在稍后的 Data Agent 步骤之前，理解这种区别很重要。
>
> **静态绑定**指向维度数据或缓慢变化的数据，例如客户名称、产品类别或区域时区。当 agent 使用静态绑定回答问题时，它会：
>
> - 将属性值视为当前事实。
> - 不执行时间推理，也就是说它不能解释某个值是在*何时*或*为什么*发生变化的。
> - 返回类似这样的答案：*“Customer CUST000297 的 loyalty tier 是 Gold。”*
>
> **时间序列绑定**指向随时间变化的数据，例如传感器读数、事务日志，或带时间戳的需求信号。当 agent 使用时间序列绑定回答问题时，它可以：
>
> - 跟踪按时间排序的观察结果之间的因果关系。
> - 解释导致某个状态或条件的原因。
> - 推理异常、违规和趋势。
> - 返回类似这样的答案：*“由于来自 southwest 区域的三次连续高信号读数，产品 PROD00042 的需求在 14:30 激增。”*
>
> 这种差异很重要，因为同一个问题会因绑定类型不同而产生非常不同的答案。产品状态的静态绑定会返回一个扁平事实（*“该产品有库存”*），而库存水平的时间序列绑定可以让 agent 解释库存为什么下降以及何时发生变化。
>
> 在本实验中，大多数绑定都是静态绑定（Lakehouse 中的维度表，以及 Eventhouse 中的事实快照）。当你在[步骤 6](#步骤-6试用-data-agent)中试用 Data Agent 时，请记住这个区别。agent 的响应会反映本体中可用的绑定类型。另外，Data Agent 当前实现只使用 NL->GQL。

## 步骤 4：探索图

图刷新完成后，你可以直观探索已连接的数据。

### 检查图刷新状态

继续之前，请确认设置 notebook 触发的图刷新已经完成。

1. 在左侧 Fabric 导航栏中，选择 **Monitor** 图标。
1. 在监视视图中，查找与你的本体关联的图刷新活动。

   ![导航栏监视功能截图。](./media/iq-lab-retail-supply-chain/monitor.png)

1. 如果刷新仍在进行，请等待其完成再继续。图刷新可能需要最多 15 分钟，具体取决于数据量和当前容量利用率。
1. 状态显示为 **Completed** 后，返回本体开始探索图。

### 导航图视图

1. 从 **Product** 实体打开 **Entity type overview**。

   ![显示 Product 的 Entity type overview 页面以及数据和关系预览卡片的截图。](./media/iq-lab-retail-supply-chain/entity-overview.png)

1. 浏览与此实体关联的数据的不同预览区域。
1. 在左上角卡片中，选择 **Relationship Graph** 上的 **Expand**。这会打开以 *Product* 节点为中心的图视图。

   ![显示扩展图视图的截图，其中节点和边表示实体和关系。](./media/iq-lab-retail-supply-chain/graph-view-expanded.png)

1. 图视图会将实体实例显示为节点，将关系显示为边。你可以：

   - **平移和缩放**来浏览图。
   - **选择节点**来查看其属性和连接的实体。
   - **沿边移动**来遍历关系。

1. 尝试探索你刚开始查看的 **Product** 节点如何连接到区域。节点周围会展开一组工具。选择 **+** 号以添加一个连接节点。在此例中，唯一选项是 **Order**，因此选择该节点。对 **Order** 节点重复此操作，这次添加连接的 **Region** 节点。类似地，你可以单击某个节点，然后选择 **X** 将其从图视图中移除。请注意，随着你添加和移除节点，对应的边也会随之添加和移除。

![通过从 Product 到 Order 再到 Region 的关系添加节点，然后移除 Order 节点并看到连接到 Region 的边也被移除的 GIF。](./media/iq-lab-retail-supply-chain/add-nodes.gif)

   > [!IMPORTANT]
   > 在图探索过程中不要对本体进行任何数据或架构更改。图刷新成本较高且耗时较长，任何更改都会要求再次进行漫长的刷新。

### 使用 UI 查询图

现在使用可视化查询生成器运行查询。我们将查找有关联退货的有效促销活动。

1. 从 Product 菜单栏中选择 **Clear query**。
1. 在图查询 UI 中，从右侧窗格选择 **Return** &rarr; **Product** &rarr; **Promotion** 节点来构建路径。
1. 将连接它们的边 *ReturnForProduct* 和 *PromotionForProduct* 添加到路径中。这会告诉查询引擎从退货记录开始，经过关联产品，再遍历到链接到该产品的任何促销活动。
1. 在 **Promotion** 节点上添加筛选器：将 **IsActivePromotion** 设置为 **true**。这会将结果缩小为当前仍处于活动状态的促销。

   ![显示在 Promotion 节点上将 IsActivePromotion 设置为 true 的筛选器配置截图。](./media/iq-lab-retail-supply-chain/add-filter.png)

1. 选择 **Apply** 确认查询配置。
1. 选择 **Run** 执行查询。
1. 查看结果。你应该能看到一个有效促销列表，其中促销产品也发生过退货。这类洞察，即将退货关联回营销活动，在扁平表查询中很难实现，但当数据建模为图时会变得直接。

   ![显示应用 Return 到 Product 到 Promotion 路径及有效促销筛选器后的查询结果截图。](./media/iq-lab-retail-supply-chain/query-built-in-ui.png)

1. 选择 **Query builder** &rarr; **Code editor** 查看可视化生成器生成的 GQL 查询。这会显示与你配置的路径和筛选器对应的底层 GQL 语法。进入下一步手动编写 GQL 之前，花一点时间阅读查询结构。

## 步骤 5：运行 GQL 查询

在此步骤中，你将使用 GQL（Graph Query Language，图查询语言）直接查询本体图。编写查询之前，先了解 GQL 是什么，以及为什么它适合这个场景。

### 什么是 GQL

GQL 是图数据库的 ISO 标准化查询语言（[ISO/IEC 39075](https://www.iso.org/standard/76120.html)）。开发 SQL 的同一个 ISO 工作组也开发 GQL，因此如果你写过 SQL 查询，会看到一些熟悉的概念，例如表达式、谓词、类型和筛选。

关键差异在于表达关系的方式。GQL 不使用 JOIN 语句来联接表，而是使用**可视化图模式**直接描述你希望遍历的连接。这让查询更易读，也更接近你自然思考连接数据的方式。

### 为什么使用 GQL 查询本体

传统 SQL 很擅长在单个表中查找数据，或联接少数相关表。但是，当你需要遍历许多关系时，例如查找某个区域中哪些促销活动推动了退货，SQL 查询会变成一长串复杂联接，难以编写也更难维护。

GQL 通过将关系作为一等公民来解决这个问题。一个在 SQL 中可能需要五六个 JOIN 的查询，在 GQL 中可以成为一个单一模式，读起来几乎像一句话：“查找下过订单且订单中包含特定类别产品的客户。”

在 Fabric IQ 中，GQL 让你能够：

- **自然遍历本体关系。** `(node:EntityType)-[edge:RelationshipType]->(node:EntityType)` 模式与你在本体中定义关系的方式完全一致。
- **透明地跨数据源查询。** 同一个 GQL 查询可以从 Eventhouse 中的数据遍历到 Lakehouse 中的数据，而你无需知道哪个引擎保存了哪个实体。
- **简洁表达复杂业务问题。** “西南区域中哪些客户购买了促销产品且这些产品发生过退货？”这样的问题可以成为单个可读查询。

### GQL 当前可用功能

Fabric IQ 中的 GQL 支持包括：

- 用于跨实体类型和关系类型遍历节点与边的 **MATCH 模式**
- 根据属性值筛选结果的 **WHERE 子句**
- 指定输出数据的 **RETURN 语句**
- 在单个查询中组合多个关系路径的**模式组合**
- 用于计数、求和和分组结果的**聚合**

### 当前限制

GQL 支持仍处于预览阶段，因此请预期以下限制：

- **查询复杂度限制。** 很长的遍历路径（超过六七跳）可能会超时或返回不完整结果。
- **不能通过 GQL 修改架构。** 你不能使用 GQL 创建或修改实体类型、关系类型或绑定。请使用本体 UI 进行架构更改。
- **函数支持有限。** ISO 标准中的某些高级 GQL 函数当前尚不可用。

有关 Fabric 中完整的 GQL 语言参考，请参阅 [GQL language guide](https://learn.microsoft.com/fabric/graph/gql-language-guide)。

### 运行第一个 GQL 查询

1. 选择 **Clear query** 重置图查询。
1. 将以下 GQL 查询复制并粘贴到代码编辑器中。此查询会查找西南区域中所有购买过 household 类别产品的客户。它会从 *Customer* 经由 *Order* 和 *OrderLine* 遍历到 *Product*，然后分支到 *ProductCategory* 和 *Region* 以应用筛选器。

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
1. 查看结果。查询会返回西南区域中订单包含 household 类别产品的客户。注意 GQL 模式如何映射到你之前探索的本体关系，每个 `[edge:RelationshipType]` 都对应一个在本体中定义的关系类型。

   ![显示本体 GQL 查询示例及其结果的截图。](../images/3.ontology-query-sample.png)

> [!TIP]
> 查看查询结果时，请注意 GQL 如何遍历关系。`(node:EntityType)-[edge:RelationshipType]->(node:EntityType)` 模式映射了本体结构。这就是基于图的语义模型的价值：查询会沿着业务概念的自然结构前进。

## 步骤 6：试用 Data Agent

在此步骤中，你将看到 Fabric Data Agent 如何使用本体来回答有关数据的自然语言问题。

> [!NOTE]
> Data Agent 当前处于预览阶段。请预期以下行为：
>
> - 第一次查询通常耗时更长，或者可能超时。如果发生这种情况，请再次运行。
> - 结果可能在不同运行之间不一致。
> - 复杂问题可能无法返回有用答案。
>
> 此步骤用于演示技术方向，而不是生产就绪行为。

### 创建 Data Agent

1. 返回工作区。
1. 选择 **+ New item**，然后选择 **Data Agent**。
1. 为 agent 命名（例如 *RetailSupplyChainAgent*）。
1. 在 agent 配置中，将本体添加为数据源。

### 提问

一次尝试一个问题。输入每个问题后，等待响应再提下一个问题。

**问题 1：**

> What is the loyalty tier and lifetime value for Customer CUST000297?

查看响应。agent 应该会通过本体查询并返回该客户的忠诚度等级和生命周期价值。

![显示 Fabric Data Agent 针对零售供应链问题返回示例响应的截图。](../images/4.data-agent-sample.png)

**问题 2：**

> Is Region southwest marked as cold-chain required, and what is its timezone?

这个问题测试 agent 查找 Region 实体中特定属性的能力。

**问题 3：**

> For Customer CUST000297, show the customer's region name and the latest customer engagement score.

这个问题要求 agent 从 Customer 遍历关系到 Region，展示跨实体推理能力。

> [!NOTE]
> 如果查询超时或返回错误，请尝试再次运行。新 Data Agent 会话中的首次查询经常失败。这是预览版的已知限制。

## 步骤 7：总结

你已经探索了一个 IQ 工作流，从本体创作和跨源数据绑定，到图遍历、GQL 查询和对话式 Data Agent。下面是 IQ 当前状态的概览。

### 当前可用功能

<!-- TODO: Fill in based on current GA/preview status -->

- 使用实体类型、属性和关系类型创建本体
- 绑定到 Eventhouse 和 Lakehouse 数据
- 图可视化和探索
- GQL 查询
- 使用本体作为数据源的 Data Agent

### 已知限制

<!-- TODO: Fill in based on current known issues -->

- 图刷新时间较长且不确定
- Data Agent 查询可能超时，尤其是首次运行时
- Data Agent 结果可能在不同运行之间不一致
- 复杂自然语言查询可能无法返回有用结果

### 后续计划

<!-- TODO: Fill in based on product roadmap -->

- 改进图刷新性能
- 提升 Data Agent 可靠性和查询覆盖范围
- 增强 GQL 能力

## 清理资源

如果你已经完成探索，可以删除工作区或本实验中创建的各个项目，以避免产生容量使用：

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
