# 📝 Lambda Function - How to Add ingestion_id

## Quick Overview

When Lambda processes files from S3 and writes to database, it must include `ingestion_id` in every INSERT statement.

## Step 1: Get ingestion_id from S3 Metadata

When file is uploaded, `ingestion_id` is stored in S3 object metadata. Lambda needs to read it:

### Python (AWS Lambda)
```python
import boto3

def lambda_handler(event, context):
    # Get S3 bucket and key from event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Get S3 object metadata
    s3_client = boto3.client('s3')
    response = s3_client.head_object(Bucket=bucket, Key=key)
    
    # Extract ingestion_id from metadata
    ingestion_id = response['Metadata'].get('ingestion-id', None)
    
    if not ingestion_id:
        print("⚠️ WARNING: ingestion_id not found in S3 metadata")
        # You might want to generate one or fail
        return
    
    print(f"✅ Found ingestion_id: {ingestion_id}")
    
    # Now use this ingestion_id when inserting data
```

## Step 2: Include ingestion_id in All INSERT Statements

When inserting data into KPI tables, add `ingestion_id` to every INSERT:

### Example: Insert into sales_kpi

**❌ BEFORE (Wrong - Missing ingestion_id):**
```python
insert_query = """
    INSERT INTO sales_kpi (
        product_id, product_name, period, period_type,
        gross_sales, returns, transactions, units_sold,
        net_sales, atv, upt, asp
    ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
"""
cursor.execute(insert_query, (
    product_id, product_name, period, period_type,
    gross_sales, returns, transactions, units_sold,
    net_sales, atv, upt, asp
))
```

**✅ AFTER (Correct - With ingestion_id):**
```python
insert_query = """
    INSERT INTO sales_kpi (
        product_id, product_name, period, period_type,
        gross_sales, returns, transactions, units_sold,
        net_sales, atv, upt, asp,
        ingestion_id  -- ✅ ADD THIS
    ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
"""
cursor.execute(insert_query, (
    product_id, product_name, period, period_type,
    gross_sales, returns, transactions, units_sold,
    net_sales, atv, upt, asp,
    ingestion_id  # ✅ ADD THIS
))
```

## Step 3: Apply to All KPI Tables

Do the same for ALL 7 KPI tables:

### 1. sales_kpi
```python
INSERT INTO sales_kpi (
    product_id, period, period_type, product_name,
    gross_sales, returns, transactions, units_sold,
    units_returned, net_sales, atv, upt, asp,
    gross_sales_mom_growth, returns_mom_growth,
    net_sales_mom_growth, transactions_mom_growth,
    units_sold_mom_growth, units_returned_mom_growth,
    atv_mom_growth, upt_mom_growth, asp_mom_growth,
    gross_sales_yoy_growth, returns_yoy_growth,
    net_sales_yoy_growth, transactions_yoy_growth,
    units_sold_yoy_growth, units_returned_yoy_growth,
    atv_yoy_growth, upt_yoy_growth, asp_yoy_growth,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

### 2. customer_kpi
```python
INSERT INTO customer_kpi (
    product_id, product_name, kpi_name, period_type,
    period, value,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

### 3. productivity_kpi
```python
INSERT INTO productivity_kpi (
    kpi, product_id, product_name, time_granularity,
    period, value,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

### 4. inventory_kpi
```python
INSERT INTO inventory_kpi (
    kpi, product_id, product_name, period_type,
    period, value,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

### 5. profitability_kpi
```python
INSERT INTO profitability_kpi (
    product_id, product_name, period_type, period,
    kpi, value,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

### 6. return_kpi
```python
INSERT INTO return_kpi (
    kpi, period_type, period, product_id,
    product_name, value,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

### 7. store_kpi
```python
INSERT INTO store_kpi (
    kpi, period_type, period, entity_id,
    entity_name, value,
    ingestion_id  -- ✅ ADD THIS
) VALUES (..., ingestion_id)
```

## Complete Example Lambda Function

```python
import boto3
import psycopg2
import json

def lambda_handler(event, context):
    # Step 1: Get S3 object details
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Step 2: Get ingestion_id from metadata
    s3_client = boto3.client('s3')
    response = s3_client.head_object(Bucket=bucket, Key=key)
    ingestion_id = response['Metadata'].get('ingestion-id')
    
    if not ingestion_id:
        print("❌ ERROR: ingestion_id not found!")
        return {'statusCode': 400, 'body': 'Missing ingestion_id'}
    
    print(f"✅ Processing with ingestion_id: {ingestion_id}")
    
    # Step 3: Connect to database
    conn = psycopg2.connect(
        host=os.environ['DB_HOST'],
        database=os.environ['DB_NAME'],
        user=os.environ['DB_USER'],
        password=os.environ['DB_PASSWORD']
    )
    cursor = conn.cursor()
    
    # Step 4: Process file and insert data
    # ... your file processing logic ...
    
    # Step 5: Insert with ingestion_id
    for row in processed_data:
        insert_query = """
            INSERT INTO sales_kpi (
                product_id, product_name, period, period_type,
                gross_sales, returns, transactions, units_sold,
                net_sales, atv, upt, asp,
                ingestion_id  -- ✅ Include ingestion_id
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """
        cursor.execute(insert_query, (
            row['product_id'], row['product_name'], row['period'],
            row['period_type'], row['gross_sales'], row['returns'],
            row['transactions'], row['units_sold'], row['net_sales'],
            row['atv'], row['upt'], row['asp'],
            ingestion_id  # ✅ Pass ingestion_id here
        ))
    
    conn.commit()
    cursor.close()
    conn.close()
    
    print(f"✅ Data inserted with ingestion_id: {ingestion_id}")
    return {'statusCode': 200, 'body': 'Success'}
```

## Key Points

1. **Get ingestion_id from S3 metadata** - It's stored when file is uploaded
2. **Add ingestion_id to ALL INSERT statements** - Every row must have it
3. **Use same ingestion_id for all rows** - All data from same upload uses same ID
4. **Check if ingestion_id exists** - If missing, log error or generate one

## Testing

After updating Lambda:
1. Upload a file
2. Check database: `SELECT DISTINCT ingestion_id FROM sales_kpi;`
3. Should see the ingestion_id values
4. Each user's data should have different ingestion_id

## Summary

**What to Change:**
- ✅ Read `ingestion_id` from S3 metadata
- ✅ Add `ingestion_id` column to all INSERT statements
- ✅ Pass `ingestion_id` value when executing INSERT

**Result:**
- Each user's data will have unique `ingestion_id`
- Dashboard will filter by `ingestion_id`
- Users will only see their own data! 🎯

