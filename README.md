-- Remove records with NULL values
DELETE FROM sales WHERE sales IS NULL OR cost IS NULL OR profit IS NULL;
DELETE FROM inventory WHERE inventory_days IS NULL OR stock_units IS NULL;

-- Total Sales, Cost, Profit
SELECT 
    SUM(sales) AS total_sales,
    SUM(cost) AS total_cost,
    SUM(profit) AS total_profit
FROM sales;

-- Profit Margin by Category
SELECT 
    category,
    SUM(sales) AS total_sales,
    SUM(profit) AS total_profit,
    ROUND((SUM(profit) / SUM(sales)) * 100, 2) AS profit_margin_percent
FROM sales
GROUP BY category
ORDER BY profit_margin_percent ASC;

-- Profit Margin by Sub-Category
SELECT 
    category,
    sub_category,
    SUM(sales) AS total_sales,
    SUM(profit) AS total_profit,
    ROUND((SUM(profit) / SUM(sales)) * 100, 2) AS profit_margin_percent
FROM sales
GROUP BY category, sub_category
ORDER BY profit_margin_percent ASC;

-- Inventory Turnover Ratio = Sales / Avg Inventory
SELECT 
    s.product_id,
    p.category,
    SUM(s.sales) AS total_sales,
    AVG(i.stock_units) AS avg_inventory,
    ROUND(SUM(s.sales) / NULLIF(AVG(i.stock_units), 0), 2) AS inventory_turnover_ratio
FROM sales s
JOIN inventory i ON s.product_id = i.product_id
JOIN products p ON s.product_id = p.product_id
GROUP BY s.product_id, p.category
ORDER BY inventory_turnover_ratio ASC;

-- Identify slow-moving products
SELECT 
    s.product_id,
    p.category,
    i.inventory_days,
    SUM(s.sales) AS total_sales
FROM sales s
JOIN inventory i ON s.product_id = i.product_id
JOIN products p ON s.product_id = p.product_id
GROUP BY s.product_id, p.category, i.inventory_days
HAVING i.inventory_days > 60 AND SUM(s.sales) < 1000
ORDER BY i.inventory_days DESC;

-- Identify overstocked items
SELECT 
    s.product_id,
    p.category,
    SUM(i.stock_units) AS total_stock,
    SUM(s.profit) AS total_profit
FROM sales s
JOIN inventory i ON s.product_id = i.product_id
JOIN products p ON s.product_id = p.product_id
GROUP BY s.product_id, p.category
HAVING SUM(i.stock_units) > 500 AND SUM(s.profit) < 500
ORDER BY total_stock DESC;
# Retail-Business-Performance-Profitability-Analysis
