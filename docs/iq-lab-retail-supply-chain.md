---
title: "IQ hands-on lab: Retail supply chain"
description: A guided 60-minute hands-on lab exploring Fabric IQ capabilities, including ontology, graph exploration, GQL queries, and Data Agent.
ms.date: 02/16/2026
ms.topic: tutorial
ai-usage: ai-assisted
---

# IQ hands-on lab: Retail supply chain

In this 60-minute hands-on lab, you explore the core capabilities of [Fabric IQ (preview)](https://learn.microsoft.com/fabric/iq/overview) using a retail supply chain scenario. A single setup notebook deploys all required artifacts automatically, so you can focus on exploring ontology concepts, graph visualization, GQL queries, and Data Agent interactions.

## Scenario overview

You work for a fictional retail company that manages orders, products, customers, warehouses, and shipments across multiple regions. The company tracks inventory, demand signals, forecasts, and promotions. Your goal is to explore how Fabric IQ unifies this data using an ontology, visualizes it as a graph, and makes it queryable through GQL and conversational Data Agent.

### Why retail supply chain?

Retail supply chain is an ideal scenario for demonstrating ontology capabilities because it reflects the complexity that real enterprises face:

- **Data lives in multiple systems.** Orders might stream into an Eventhouse for real-time analytics, while product catalogs and customer profiles sit in a Lakehouse for batch processing. Traditional approaches require analysts to know which system holds which data and how to join across them.
- **Relationships are central to insight.** A single question like "Which promotions drove returns in the southwest region?" requires traversing from returns to products to promotions to regions. Flat tables make this painful; a graph makes it natural.
- **Business language differs from technical schemas.** Column names like `cust_lt_val` or `prod_cat_id` don't match how business users think. An ontology maps technical columns to concepts like *Customer lifetime value* and *Product category*, so everyone speaks the same language.
- **Multiple personas need access.** Data engineers care about schema and lineage. Analysts want to explore relationships visually. Business users want to ask questions in plain English. Ontology, graph, and Data Agent serve all three from the same foundation.

This lab walks you through each layer—from raw tables to semantic model to conversational AI—using a realistic set of retail entities and relationships.

### Entities and relationships

The ontology for this lab includes the following entity types:

| Entity type | Description |
|---|---|
| *Order* | A customer purchase transaction |
| *OrderLine* | Individual line items within an order |
| *Customer* | The buyer associated with an order |
| *Product* | Items available for sale |
| *ProductCategory* | Groupings of related products |
| *Store* | Retail locations where orders originate |
| *Region* | Geographic areas containing stores and warehouses |
| *Warehouse* | Fulfillment centers that ship orders |
| *Carrier* | Logistics providers handling shipments |
| *Shipment* | Delivery records linking orders to carriers |
| *Inventory* | Stock levels at warehouses |
| *Forecast* | Predicted demand for products |
| *DemandSignal* | Real-time indicators of customer demand |
| *Promotion* | Marketing campaigns affecting product sales |
| *Return* | Returned items linked back to orders |

Key relationships include *Order* &rarr; *Customer*, *Order* &rarr; *Region*, *OrderLine* &rarr; *Product*, *Product* &rarr; *ProductCategory*, *Shipment* &rarr; *Carrier*, and *Shipment* &rarr; *Warehouse*.

## Lab timeline

| Time | Activity | Section |
|---|---|---|
| 0–5 min | Run setup notebook | [Step 1: Deploy the environment](#step-1-deploy-the-environment) |
| 5–20 min | Introduction to IQ and ontology concepts | [Step 2: Understand IQ and ontology concepts](#step-2-understand-iq-and-ontology-concepts) |
| 20–35 min | Explore artifacts before graph is ready | [Step 3: Explore deployed artifacts](#step-3-explore-deployed-artifacts) |
| 35–45 min | Graph UI exploration | [Step 4: Explore the graph](#step-4-explore-the-graph) |
| 45–55 min | Run GQL queries | [Step 5: Run GQL queries](#step-5-run-gql-queries) |
| 55–58 min | Data Agent demo | [Step 6: Try the Data Agent](#step-6-try-the-data-agent) |
| 58–60 min | Wrap-up and roadmap | [Step 7: Wrap-up](#step-7-wrap-up) |

## Step 1: Deploy the environment

A single Fabric notebook deploys all the artifacts you need for this lab, including:

- An Eventhouse named **EHRetail** with pre-populated tables
- A Lakehouse named **LHRetail** with supporting data
- An ontology with entity types, relationship types, and data bindings

Before you start, confirm that ontology is enabled for your tenant. If you don't see ontology options in Fabric, ask your tenant administrator to enable the Fabric IQ ontology preview setting.

![Screenshot showing the tenant setting used to enable ontology capabilities in Fabric.](../images/1.prequsite-tenant-enable-ontology.png)

### Upload and run the setup notebook

<!-- TODO: Add notebook source location (GitHub repo URL, pre-loaded workspace, or download link) -->

1. Download the setup notebook file (*.ipynb*) provided by your lab instructor.
1. Open your Fabric workspace.
1. Select **+ New item** > **Import notebook**.
1. Select **Upload** and browse to the notebook file you downloaded, then select **Open**.
1. Wait for the upload to complete. The notebook appears in your workspace item list.
1. Open the uploaded notebook.
1. Select **Run all** to execute the notebook.

   The notebook performs the following actions automatically:
   - Creates the Eventhouse and Lakehouse artifacts
   - Generates sample retail data across all tables
   - Creates the ontology with entity types and relationships
   - Binds entity types to data sources
   - Triggers a graph refresh

   > [!IMPORTANT]
   > The graph refresh runs in the background and takes approximately 10–15 minutes to complete. You cannot accelerate this process. Continue to the next steps while it runs.

1. Confirm that the notebook cells complete without errors. You should see success messages for each artifact creation step.

   ![Screenshot showing the setup notebook running with Copilot available to help resolve an error.](../images/2.setup-notebook-with-copilot-resolve-error.png)

## Step 2: Understand IQ and ontology concepts

While the graph refresh runs in the background, take this time to understand the core concepts behind Fabric IQ.

THIS IS PLACEHOLDER CONTENT - WE WILL PRESENT THE POWERPOINT AT THIS TIME AND ASK THE PARTICIPANTS TO FOCUS ON THE LAB LEADER.

## Step 3: Explore deployed artifacts

While you wait for the graph to finish refreshing, explore the artifacts the setup notebook created.

### Inspect Eventhouse tables

1. In your Fabric workspace, select the **EHRetail** Eventhouse.
1. Open the database and review the following tables:

   | Table | Description |
   |---|---|
   | *carriers* | Rows with carrier IDs, service type codes, and coverage area identifiers |
   | *customers* | Rows with customer IDs, loyalty tier codes, lifetime value scores, and contact fields |
   | *demand_signals* | Rows with product and region ID pairs, signal strength values, and timestamps |
   | *forecasts* | Rows with product IDs, predicted demand quantities, and forecast period dates |
   | *inventories* | Rows with warehouse and product ID pairs, stock quantities, and reorder thresholds |
   | *products* | Rows with product IDs, discount percentages, and effective dates (fact/transactional data) |
   | *regions* | Rows with region codes, timezone offsets, and operational flags like cold-chain indicators |
   | *shipments* | Rows with shipment IDs, order/carrier/warehouse foreign keys, and status codes |
   | *stores* | Rows with store IDs, location coordinates, and operational attributes |

1. Select any table and review the raw data. Notice that this is "raw" operational data—column names might not be human-friendly, and values might include IDs rather than descriptive names. This is exactly what the ontology helps resolve by layering business meaning on top.

### Inspect the Lakehouse

1. Return to your workspace and open the **LHRetail** Lakehouse.
1. In the **Lakehouse explorer** pane, expand the **Tables** folder. You should see Delta tables that the setup notebook created from the generated retail data. These tables serve as additional data sources for the ontology.
1. Select a table (for example, *orders* or *returns*) to preview its contents. Compare the structure here with what you saw in the Eventhouse tables. Notice that:
   - The Lakehouse tables may contain different columns or additional data that complements the Eventhouse tables.
   - Some entity types in the ontology bind to Lakehouse tables rather than Eventhouse tables, demonstrating how ontology can unify data across multiple storage engines.

1. Compare the *products* data across both engines:
   - In the Lakehouse, the *products* table contains **dimensional data**—static attributes that describe each product, such as name, category, and pricing.
   - In the Eventhouse, the *products* table contains **fact data**—transactional records about discounts applied to different products over time.
   - These two tables represent different facets of the same business concept (*Product*). The ontology unifies both under a single *Product* entity type, so downstream consumers like Graph queries and Data Agent don't need to know which engine stores which aspect of the data.

1. Check the **Table details** to see the row count, schema, and column types. This helps you understand the scale of data the ontology operates over.

   > [!TIP]
   > The Lakehouse and Eventhouse serve different roles in this scenario. Eventhouse holds operational and streaming data optimized for real-time queries, while the Lakehouse stores batch and historical data. The ontology unifies both under a single set of business concepts.

### Explore ontology structure

1. Return to your workspace and open the **Ontology** item.
1. In the **Entity Types** pane, review the list of entity types. You should see all 15 entity types listed in the [Scenario overview](#scenario-overview).
1. Start by selecting the **Product** entity type. In the previous step, you saw that product data lives in two places: the Lakehouse holds dimensional attributes (name, category, pricing) and the Eventhouse holds fact data (discount records). Now see how the ontology brings these together:

   ![Screenshot showing the Product entity type selected in the Entity Types pane with its properties displayed.](./media/iq-lab-retail-supply-chain/product-entity-view.png)

   1. Look on the righthand pane of **Entity type configuration** to view the properties and bindings. 
   1. Review the list of properties on the *Product* entity type. Notice that some properties are bound to columns in the Lakehouse *products* table (for example, product name, category, and unit cost) while other properties are bound to columns in the Eventhouse *products* table (for example, discount percent and date).
   1. Select the **Bindings** tab for *Product*. This tab shows cards for each source table that is bound to this entity.

   ![Screenshot showing the Bindings tab for the Product entity type with cards for Lakehouse and Eventhouse data sources.](./media/iq-lab-retail-supply-chain/entity-type-bindings.png)
    
    > [!NOTE] This is the core value of an ontology: a single *Product* entity type presents a unified view of the concept, even though the underlying data comes from two different storage engines. Consumers of the ontology—Graph, GQL, and Data Agent—don't need to know where each property originates.

1. Now explore the relationships connected to *Product*. 
   - **ProductInCategory** - Links a *Product* to its *ProductCategory*, so you can answer questions like "Which products belong to the frozen goods category?"
   - **InventoryForProduct** - Links *Inventory* records to a *Product*, showing stock levels at each warehouse for that product.
   - **PromotionForProduct** - Links *Promotion* campaigns to the *Product* they target, connecting marketing activity to specific items.

   These relationships turn isolated tables into a connected graph. A single *Product* node can lead you to its category, its inventory across warehouses, and any active promotions—all without writing joins or knowing which tables live in which engine.

1. Review the remaining relationship types to see the broader graph structure:
   - *OrderPlacedByCustomer* (which customer placed the order)
   - *OrderFulfilledToRegion* (where the order was fulfilled)
   - *OrderHasLineItem* (what was ordered)
   - *ShipmentFulfillsOrder* (what shipment fulfilled the order)
   - *ShipmentDepartedFromWarehouse* (where inventory is stored)

1. Return to the **Data Bindings** view and explore bindings for a few more entity types. For example, the *Customer* entity type binds to the *customers* table in EHRetail and the *customers* table in the lakehouse. Look at the icon to understand which type of 

> [!TIP]
> Fabric IQ agents change their reasoning strategy based on the type of data binding. This distinction is important to understand before you reach the Data Agent step later in the lab.
>
> **Static bindings** point to dimensional or slowly changing data—attributes like customer name, product category, or region timezone. When an agent answers questions using static bindings, it:
>
> - Treats property values as current facts.
> - Doesn't perform temporal reasoning (it can't explain *when* or *why* something changed).
> - Returns answers like: *"The loyalty tier for Customer CUST000297 is Gold."*
>
> **Time-series bindings** point to data that changes over time—sensor readings, transaction logs, or demand signals with timestamps. When an agent answers questions using time-series bindings, it:
>
> - Can trace causality across time-ordered observations.
> - Can explain what led to a particular state or condition.
> - Can reason about exceptions, violations, and trends.
> - Returns answers like: *"Demand for product PROD00042 spiked at 14:30 due to three consecutive high-signal readings from the southwest region."*
>
> This difference matters because the same question can produce very different answers depending on the binding type. A static binding for a product's condition returns a flat fact (*"The product is in stock"*), while a time-series binding for inventory levels enables the agent to explain *why* stock dropped and *when* the change occurred.
>
> In this lab, most bindings are static (dimensional tables in the Lakehouse, fact snapshots in the Eventhouse). As you try the Data Agent in [Step 6](#step-6-try-the-data-agent), keep this distinction in mind—the agent's responses reflect the binding types available in the ontology. Additionally, the current implementation of the Data Agent only uses NL->GQL.

## Step 4: Explore the graph

Once the graph refresh completes, you can visually explore the connected data.

### Check graph refresh status

Before you proceed, confirm that the graph refresh triggered by the setup notebook has finished.

1. In the Fabric navigation bar on the left, select the **Monitor** icon.
1. In the monitoring view, look for the graph refresh activity associated with your ontology. 

   ![Screenshot of navigation bar monitoring.](./media/iq-lab-retail-supply-chain/monitor.png)

1. If the refresh is still in progress, wait for it to complete before continuing. Graph refresh can take up to 15 minutes depending on the volume of data and current capacity utilization.
1. Once the status shows **Completed**, return to your ontology to begin exploring the graph.

### Navigate the graph view

1. From the **Product** entity, open the **Entity type overview**.

   ![Screenshot showing the Entity type overview page for Product with preview cards for data and relationships.](./media/iq-lab-retail-supply-chain/entity-overview.png)

1. Take a look around the different preview aspects of the data associated with this entity.
1. In the top left card, select **Expand** on the **Relationship Graph**. This opens the graph view centered on *Product* nodes.

   ![Screenshot showing the expanded graph view with nodes and edges representing entities and relationships.](./media/iq-lab-retail-supply-chain/graph-view-expanded.png)

1. The graph view displays entity instances as nodes and relationships as edges. You can:
   - **Pan and zoom** to navigate through the graph.
   - **Select a node** to view its properties and connected entities.
   - **Follow edges** to traverse relationships. 

1. Try exploring how the **Product** node you have started exploring is connected to regions. A selection of tools appears fanning out from the node. Select the **+** sign to add a connected node. In this case, the only option is **Order**, so select that node. Repeat for the **Order** node, this time adding the connected **Region** node. Similarly, you can click on a node and then select **X** to remove it from the graph view. Notice that the corresponding edges are also added and removed as you add and remove nodes.

![GIF of adding nodes to the graph view by following relationships from Product to Order to Region, then removing the Order node and seeing the edge to Region also removed.](./media/iq-lab-retail-supply-chain/add-nodes.gif)

   > [!IMPORTANT]
   > Don't make any data or schema changes to the ontology during Graph exploration. Graph refresh is expensive and time-consuming, so changes would require another lengthy refresh cycle.

### Query the graph using the UI

Now use the visual query builder to run queries. We are going to find active promotions that had returns associated with them.

1. Select **Clear query** from the Product menu bar.
1. In the graph query UI, build a path by selecting the  **Return** &rarr; **Product** &rarr; **Promotion** nodes from the righthand pane.
1. Add their connecting edges *ReturnForProduct* and *PromotionForProduct* to the path. This tells the query engine to traverse from return records, through the associated product, to any promotions linked to that product.
1. Add a filter on the **Promotion** node: set **IsActivePromotion** equal to **true**. This narrows results to only promotions that are currently active.

   ![Screenshot showing the filter configuration with IsActivePromotion set to true on the Promotion node.](./media/iq-lab-retail-supply-chain/add-filter.png)

1. Select **Apply** to confirm the query configuration.
1. Select **Run** to execute the query.
1. Review the results. You should see a list of active promotions where the promoted product also had returns. This type of insight—connecting returns back to marketing campaigns—is difficult to achieve with flat table queries but straightforward when the data is modeled as a graph.

   ![Screenshot showing the query results with the Return to Product to Promotion path and active promotion filter applied.](./media/iq-lab-retail-supply-chain/query-built-in-ui.png)

1. Select **Query builder** &rarr; **Code editor** to view the GQL query that the visual builder generated. This shows the underlying GQL syntax that corresponds to the path and filter you configured. Take a moment to read through the query structure before moving on to writing GQL manually in the next step.

## Step 5: Run GQL queries

In this step, you use GQL (Graph Query Language) to query the ontology graph directly. Before writing queries, take a moment to understand what GQL is and why it's the right tool for this scenario.

### What is GQL?

GQL is the ISO-standardized query language for graph databases ([ISO/IEC 39075](https://www.iso.org/standard/76120.html)). The same ISO working group that develops SQL also develops GQL, so you'll find familiar concepts if you've written SQL queries before—expressions, predicates, types, and filtering all work similarly.

The key difference is how you express relationships. Instead of writing JOIN statements that link tables together, GQL uses **visual graph patterns** that directly describe the connections you want to traverse. This makes queries more readable and closer to how you naturally think about connected data.

### Why use GQL for ontology queries?

Traditional SQL excels at finding data within a single table or joining a small number of related tables. But when you need to traverse many relationships—like finding which promotions drove returns in a specific region—SQL queries become complex chains of joins that are difficult to write and harder to maintain.

GQL solves this problem by treating relationships as first-class citizens. A query that would require five or six JOIN statements in SQL becomes a single pattern that reads almost like a sentence: "find customers who placed orders that contained products in a specific category."

In the context of Fabric IQ, GQL lets you:

- **Traverse ontology relationships naturally.** The pattern `(node:EntityType)-[edge:RelationshipType]->(node:EntityType)` mirrors exactly how you defined relationships in the ontology.
- **Query across data sources transparently.** The same GQL query can traverse from data in the Eventhouse to data in the Lakehouse without you needing to know which engine holds which entity.
- **Express complex business questions concisely.** Questions like "Which customers in the southwest region ordered products from promotions that had returns?" become single, readable queries.

### What works today

GQL support in Fabric IQ includes:

- **MATCH patterns** for traversing nodes and edges across entity types and relationship types
- **WHERE clauses** for filtering results based on property values
- **RETURN statements** for specifying which data to output
- **Pattern composition** for combining multiple relationship paths in a single query
- **Aggregations** for counting, summing, and grouping results

### Current limitations

GQL support is in preview, so expect the following:

- **Query complexity limits.** Very long traversal paths (more than six or seven hops) may time out or return incomplete results.
- **No schema modifications from GQL.** You can't create or modify entity types, relationship types, or bindings using GQL—use the ontology UI for schema changes.
- **Limited function support.** Some advanced GQL functions from the ISO standard aren't yet available.

For the full GQL language reference in Fabric, see [GQL language guide](https://learn.microsoft.com/fabric/graph/gql-language-guide).

### Run your first GQL query

1. Select **Clear query** to reset the graph query.
1. Copy and paste the following GQL query into the code editor. This query finds all customers in the southwest region who placed orders for products in the household category. It traverses from *Customer* through *Order* and *OrderLine* to *Product*, then branches to both *ProductCategory* and *Region* to apply filters.

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

1. Select **Run** to execute the query.
1. Review the results. The query returns customers from the southwest region whose orders included products in the household category. Notice how the GQL pattern mirrors the ontology relationships you explored earlier—each `[edge:RelationshipType]` corresponds to a relationship type you defined in the ontology.

   ![Screenshot showing a sample ontology GQL query and its results.](../images/3.ontology-query-sample.png)

> [!TIP]
> As you review query results, pay attention to how GQL traverses relationships. The pattern `(node:EntityType)-[edge:RelationshipType]->(node:EntityType)` mirrors the ontology structure. This is the power of a graph-based semantic model—queries follow the natural structure of your business concepts.

## Step 6: Try the Data Agent

In this step, you see how a Fabric Data Agent uses the ontology to answer natural-language questions about your data.

> [!NOTE]
> Data Agent is in preview. Expect the following behavior:
>
> - The first query often takes longer or might time out. Run it again if this happens.
> - Results might be inconsistent between runs.
> - Complex questions might not return useful answers.
>
> This step is illustrative. It demonstrates the *direction* of the technology, not production-ready behavior.

### Create a Data Agent

1. Return to your workspace.
1. Select **+ New item** and choose **Data Agent**.
1. Name the agent (for example, *RetailSupplyChainAgent*).
1. In the agent configuration, add the ontology as a data source.

### Ask questions

Try the following questions one at a time. After entering each question, wait for the response before asking the next one.

**Question 1:**

> What is the loyalty tier and lifetime value for Customer CUST000297?

Review the response. The agent should return the customer's loyalty tier and lifetime value by querying through the ontology.

![Screenshot showing a sample Fabric Data Agent response for a retail supply chain question.](../images/4.data-agent-sample.png)

**Question 2:**

> Is Region southwest marked as cold-chain required, and what is its timezone?

This question tests the agent's ability to find specific attributes on a Region entity.

**Question 3:**

> For Customer CUST000297, show the customer's region name and the latest customer engagement score.

This question requires the agent to traverse a relationship from Customer to Region, demonstrating cross-entity reasoning.

> [!NOTE]
> If a query times out or returns an error, try running it again. First queries in a new Data Agent session often fail. This is a known limitation of the preview.

## Step 7: Wrap-up

You've now explored an IQ workflow—from ontology authoring and cross-source data bindings to graph traversal, GQL queries, and conversational Data Agent. Here's a snapshot of the current state of IQ.

### What works today

<!-- TODO: Fill in based on current GA/preview status -->

- Ontology creation with entity types, properties, and relationship types
- Data bindings to Eventhouse and Lakehouse
- Graph visualization and exploration
- GQL querying
- Data Agent with ontology as a source

### Known limitations

<!-- TODO: Fill in based on current known issues -->

- Graph refresh times are long and non-deterministic
- Data Agent queries can time out, especially on first run
- Data Agent results might be inconsistent between runs
- Complex natural-language queries might not return useful results

### What's coming next

<!-- TODO: Fill in based on product roadmap -->

- Improvements to graph refresh performance
- Enhanced Data Agent reliability and query coverage
- Additional GQL capabilities

## Clean up resources

If you're done exploring, you can delete the workspace or the individual items created during this lab to avoid incurring capacity usage:

1. Navigate to your workspace.
1. Select the items created during the lab (**EHRetail**, the Lakehouse, the ontology, and the Data Agent).
1. Delete the selected items.

## Related content

- [What is Fabric IQ (preview)?](https://learn.microsoft.com/fabric/iq/overview)
- [What is ontology (preview)?](https://learn.microsoft.com/fabric/iq/ontology/overview)
- [Ontology (preview) tutorial](https://learn.microsoft.com/fabric/iq/ontology/tutorial-0-introduction)
- [Graph in Microsoft Fabric](https://learn.microsoft.com/fabric/graph/overview)
- [GQL language guide](https://learn.microsoft.com/fabric/graph/gql-language-guide)
- [GQL expressions, predicates, and functions](https://learn.microsoft.com/fabric/graph/gql-expressions)
- [Fabric data agent](https://learn.microsoft.com/fabric/data-science/concept-data-agent)
