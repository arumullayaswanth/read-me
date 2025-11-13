# Deployment Instructions - User Data Isolation Fix

## Summary
Fixed the issue where all users were seeing the same data. Now each user only sees their own uploaded data.

## Changes Made

### Server-Side (`Server/routes/kpi.js`)
1. ✅ Added authentication to all KPI routes
2. ✅ Added user filtering by latest ingestion ID
3. ✅ Each endpoint now gets user's email and filters data accordingly

### Database (`Server/utils/kpiDatabase.js`)
1. ✅ Updated KPI queries to filter by `ingestion_id`
2. ✅ Each user's data is isolated

### Client-Side (`Client/hooks/useKPIData.ts`)
1. ✅ Added Authorization header to all KPI requests
2. ✅ Token retrieved from localStorage

## Deployment Steps

### 1. Update Server
```bash
cd Server
pm2 restart foresightflow
```

### 2. Rebuild and Update Client
```bash
cd Client
npm run build
pm2 restart foresightflow-client
```

### 3. Verify
Check server logs:
```bash
pm2 logs foresightflow --lines 50
```

You should see:
- `📊 Fetching sales KPI data for user: user@example.com`
- `✅ Using latest ingestion ID for user@example.com: ingestion_xxx`

## Important: Database Schema

**CRITICAL:** The KPI tables must have an `ingestion_id` column for this to work!

If the column doesn't exist, you need to:
1. **Update Lambda** to write `ingestion_id` when processing data
2. **OR** run a migration to add the column:

```sql
ALTER TABLE sales_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE inventory_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE store_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE customer_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE productivity_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE profitability_kpi ADD COLUMN ingestion_id VARCHAR(255);
ALTER TABLE return_kpi ADD COLUMN ingestion_id VARCHAR(255);
```

## Testing

1. **Test with two users:**
   - User A uploads File A → Should see only File A data
   - User B uploads File B → Should see only File B data (NOT File A)

2. **Test with same user:**
   - Upload File 1 → See File 1 data
   - Upload File 2 → See File 2 data (latest)

## Troubleshooting

### Issue: "Column 'ingestion_id' does not exist"
**Solution:** Add the column to KPI tables (see above)

### Issue: "401 Unauthorized" errors
**Solution:** Make sure users are logged in and token is in localStorage

### Issue: Still seeing all users' data
**Solution:** 
1. Check if `ingestion_id` column exists in tables
2. Check if Lambda writes `ingestion_id` when processing
3. Check server logs for ingestion ID filtering

## Expected Behavior After Fix

✅ User A sees only their data
✅ User B sees only their data  
✅ No data mixing between users
✅ Latest upload is shown automatically
✅ Fast updates (5-second polling + instant refresh on upload)


Important requirement
1.The KPI tables must have an ingestion_id column. If they don't:
2.Update Lambda to write ingestion_id when processing data
3.Or run SQL migration to add the column to all KPI tables

Testing
After deployment:
User A: Upload File A → Should see only File A data
User B: Upload File B → Should see only File B data (not File A)
The fix is complete. Deploy and test with two different users to confirm each sees only their own data.