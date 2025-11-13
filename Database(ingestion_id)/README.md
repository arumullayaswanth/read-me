#### Corrected_DataBase_Schema.md
# ✅ Corrected Database Schema with ingestion_id

## Problem Found

Your current schema queries are **missing the `ingestion_id` column**, which is **critical** for user data isolation. Without it, all users will see all data mixed together.

## ✅ Corrected Schema Queries

Here are the corrected queries with `ingestion_id` added:

```javascript
const queries = {
  customer_kpi: `
    SELECT 
      product_id, 
      product_name, 
      kpi_name, 
      period_type, 
      period, 
      value,
      ingestion_id
    FROM customer_kpi
    WHERE ingestion_id = $1
  `,

  productivity_kpi: `
    SELECT 
      kpi, 
      product_id, 
      product_name, 
      time_granularity, 
      period, 
      value,
      ingestion_id
    FROM productivity_kpi
    WHERE ingestion_id = $1
  `,

  inventory_kpi: `
    SELECT 
      kpi,
      product_id, 
      product_name, 
      period_type, 
      period, 
      value,
      ingestion_id
    FROM inventory_kpi
    WHERE ingestion_id = $1
  `,

  profitability_kpi: `
    SELECT 
      product_id, 
      product_name, 
      period_type, 
      period, 
      kpi, 
      value,
      ingestion_id
    FROM profitability_kpi
    WHERE ingestion_id = $1
  `,

  return_kpi: `
    SELECT 
      kpi, 
      period_type, 
      period, 
      product_id, 
      product_name, 
      value,
      ingestion_id
    FROM return_kpi
    WHERE ingestion_id = $1
  `,

  sales_kpi: `
    SELECT
      product_id,
      period,
      period_type,
      product_name,
      gross_sales,
      returns,
      transactions,
      units_sold,
      units_returned,
      net_sales,
      atv,
      upt,
      asp,
      gross_sales_mom_growth,
      returns_mom_growth,
      net_sales_mom_growth,
      transactions_mom_growth,
      units_sold_mom_growth,
      units_returned_mom_growth,
      atv_mom_growth,
      upt_mom_growth,
      asp_mom_growth,
      gross_sales_yoy_growth,
      returns_yoy_growth,
      net_sales_yoy_growth,
      transactions_yoy_growth,
      units_sold_yoy_growth,
      units_returned_yoy_growth,
      atv_yoy_growth,
      upt_yoy_growth,
      asp_yoy_growth,
      ingestion_id
    FROM sales_kpi
    WHERE ingestion_id = $1
  `,

  store_kpi: `
    SELECT 
      kpi, 
      period_type, 
      period, 
      entity_id, 
      entity_name, 
      value,
      ingestion_id
    FROM store_kpi
    WHERE ingestion_id = $1
  `
};
```

## 🔧 Database Migration Required

You need to add `ingestion_id` column to ALL KPI tables. Run these SQL commands:

```sql
-- Add ingestion_id column to all KPI tables
ALTER TABLE customer_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE productivity_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE inventory_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE profitability_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE return_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE sales_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE store_kpi ADD COLUMN ingestion_id VARCHAR(255);

-- Optional: Add index for better query performance
CREATE INDEX idx_customer_kpi_ingestion_id ON customer_kpi(ingestion_id);
CREATE INDEX idx_productivity_kpi_ingestion_id ON productivity_kpi(ingestion_id);
CREATE INDEX idx_inventory_kpi_ingestion_id ON inventory_kpi(ingestion_id);
CREATE INDEX idx_profitability_kpi_ingestion_id ON profitability_kpi(ingestion_id);
CREATE INDEX idx_return_kpi_ingestion_id ON return_kpi(ingestion_id);
CREATE INDEX idx_sales_kpi_ingestion_id ON sales_kpi(ingestion_id);
CREATE INDEX idx_store_kpi_ingestion_id ON store_kpi(ingestion_id);
```

## 📋 Complete Table Schemas

Here are the complete table schemas with `ingestion_id`:

### sales_kpi
```sql
CREATE TABLE sales_kpi (
  product_id VARCHAR(255),
  period DATE,
  period_type VARCHAR(50),
  product_name VARCHAR(255),
  gross_sales DECIMAL(15,2),
  returns DECIMAL(15,2),
  transactions INTEGER,
  units_sold INTEGER,
  units_returned INTEGER,
  net_sales DECIMAL(15,2),
  atv DECIMAL(10,2),
  upt DECIMAL(10,2),
  asp DECIMAL(10,2),
  gross_sales_mom_growth DECIMAL(10,2),
  returns_mom_growth DECIMAL(10,2),
  net_sales_mom_growth DECIMAL(10,2),
  transactions_mom_growth DECIMAL(10,2),
  units_sold_mom_growth DECIMAL(10,2),
  units_returned_mom_growth DECIMAL(10,2),
  atv_mom_growth DECIMAL(10,2),
  upt_mom_growth DECIMAL(10,2),
  asp_mom_growth DECIMAL(10,2),
  gross_sales_yoy_growth DECIMAL(10,2),
  returns_yoy_growth DECIMAL(10,2),
  net_sales_yoy_growth DECIMAL(10,2),
  transactions_yoy_growth DECIMAL(10,2),
  units_sold_yoy_growth DECIMAL(10,2),
  units_returned_yoy_growth DECIMAL(10,2),
  atv_yoy_growth DECIMAL(10,2),
  upt_yoy_growth DECIMAL(10,2),
  asp_yoy_growth DECIMAL(10,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

### customer_kpi
```sql
CREATE TABLE customer_kpi (
  product_id VARCHAR(255),
  product_name VARCHAR(255),
  kpi_name VARCHAR(255),
  period_type VARCHAR(50),
  period DATE,
  value DECIMAL(15,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

### productivity_kpi
```sql
CREATE TABLE productivity_kpi (
  kpi VARCHAR(255),
  product_id VARCHAR(255),
  product_name VARCHAR(255),
  time_granularity VARCHAR(50),
  period DATE,
  value DECIMAL(15,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

### inventory_kpi
```sql
CREATE TABLE inventory_kpi (
  kpi VARCHAR(255),
  product_id VARCHAR(255),
  product_name VARCHAR(255),
  period_type VARCHAR(50),
  period DATE,
  value DECIMAL(15,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

### profitability_kpi
```sql
CREATE TABLE profitability_kpi (
  product_id VARCHAR(255),
  product_name VARCHAR(255),
  period_type VARCHAR(50),
  period DATE,
  kpi VARCHAR(255),
  value DECIMAL(15,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

### return_kpi
```sql
CREATE TABLE return_kpi (
  kpi VARCHAR(255),
  period_type VARCHAR(50),
  period DATE,
  product_id VARCHAR(255),
  product_name VARCHAR(255),
  value DECIMAL(15,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

### store_kpi
```sql
CREATE TABLE store_kpi (
  kpi VARCHAR(255),
  period_type VARCHAR(50),
  period DATE,
  entity_id VARCHAR(255),
  entity_name VARCHAR(255),
  value DECIMAL(15,2),
  ingestion_id VARCHAR(255)  -- ✅ ADD THIS
);
```

## ⚠️ Important Notes

1. **Lambda Must Write ingestion_id**: When Lambda processes files and writes to these tables, it MUST include the `ingestion_id` value for each row.

2. **ingestion_id Format**: The `ingestion_id` should match the one stored in the `kpi_flags` table (e.g., `ingestion_20250111_123456_user@example.com`).

3. **Data Isolation**: With `ingestion_id` filtering:
   - User A uploads File A → Lambda writes data with `ingestion_id_A`
   - User B uploads File B → Lambda writes data with `ingestion_id_B`
   - User A queries → Only sees data with `ingestion_id_A` ✅
   - User B queries → Only sees data with `ingestion_id_B` ✅

## ✅ Summary

**Your Current Schema:** ❌ Missing `ingestion_id` column
**Corrected Schema:** ✅ Includes `ingestion_id` column in all tables

**Action Required:**
1. Add `ingestion_id` column to all 7 KPI tables (use ALTER TABLE commands above)
2. Update Lambda to write `ingestion_id` when inserting data
3. Use the corrected queries above

After this, each user will only see their own data! 🎯

