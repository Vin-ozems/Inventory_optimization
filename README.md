# Inventory_optimization

## Project Overview
This project builds a MySQL-based inventory optimization system for TechElectro, integrating sales, product, and macroeconomic data to calculate data-driven reorder points, identify overstocked/understocked products, and automate inventory monitoring. The system uses rolling sales statistics and safety stock formulas to recommend when and how much to reorder for each product.

## Dataset Description
- **external_factors**: Macroeconomic indicators by date — GDP, Inflation Rate, and Seasonal Factor
- **product_data**: Product-level attributes — Product ID, Product Category, and Promotions flag (yes/no)
- **sales_data**: Daily sales records — Product ID, Sales Date, Inventory Quantity, and Product Cost

These are joined into two views (`sales_product_data`, then `Inventory_data`) to create a unified dataset combining sales, product, and economic context per product per date.

## Key Findings
- **High-demand products can still stock out**: the top ~5% of products by average sales were checked for stockout frequency (days with zero inventory), surfacing which top sellers are most at risk of lost sales due to understocking.
- **Economic conditions correlate with demand**: average sales were compared under positive vs. non-positive GDP and inflation conditions, showing that macroeconomic shifts measurably affect inventory needs per product.
- **Rolling 7-day sales and variance reveal demand volatility**: products with higher rolling variance require larger safety stock buffers to avoid stockouts, while more stable products can run leaner.
- **Overstocking and understocking coexist across the catalog**: combining average inventory value, rolling average sales, and stockout day counts per product highlighted both excess-stock and excess-stockout cases side by side.

## Recommendation / Tool
The script builds an automated **Reorder Point Engine**:
- **Lead Time Demand** = 7-day rolling average sales × 7 (lead time in days)
- **Safety Stock** = 1.645 × √(7-day rolling variance × 7) — a 95% service-level buffer
- **Reorder Point** = Lead Time Demand + Safety Stock

This is implemented as:
- A `RecalculateReorderPoint` stored procedure that recomputes the reorder point for any given product
- An `AfterInsertUnifiedTable` trigger that automatically recalculates the reorder point whenever a new inventory row is added — keeping reorder points continuously up to date without manual recalculation
- Additional monitoring procedures (`MonitorInventoryLevels`, `MonitorSalesTrends`, `MonitorStockouts`) for ongoing tracking of average inventory, sales trends, and stockout frequency per product

## Tools & Technology
- MySQL
- Window functions (rolling averages/variance via `OVER (... ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)`)
- Stored procedures & triggers for automation
- CTEs for layered calculations

## Notes
There are a few naming inconsistencies worth cleaning up before sharing publicly:
- Some queries reference `Inventory_table` before it's created (it's created later in the script, in the "Automate the Reorder Points" section) — the script likely needs reordering so `Inventory_table` exists before it's queried.
- `full_integrated_data` is referenced in `MonitorStockouts` but never created — this looks like a leftover/typo for `Inventory_table`.
- Table/column casing is inconsistent (e.g. `Sales_data` vs `sales_data`, `Inventory_data` vs `inventory_data`) — MySQL on case-sensitive filesystems (Linux) will treat these as different tables, so this should be standardized.
