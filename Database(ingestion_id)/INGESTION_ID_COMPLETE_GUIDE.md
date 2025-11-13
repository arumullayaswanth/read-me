# 📘 Complete Guide: Understanding and Checking ingestion_id

## 🎯 Quick Reference

**ingestion_id Format:**
```
ING + 5 Random Digits
Example: ING83738, ING00123, ING99999
Always 8 characters total
```

**How to Check:**
1. **Server Logs:** `pm2 logs foresightflow` → Look for "ingestion ID: ING83738"
2. **Database:** `SELECT * FROM kpi_flags WHERE email = 'user@example.com'`
3. **S3 Metadata:** `aws s3api head-object --bucket bucket --key file.csv`
4. **Browser:** F12 → Network → Upload response → See `ingestionId`
5. **API:** `curl /api/get-kpi-analysis` with ingestionId

---

## 🎯 What is ingestion_id?

**ingestion_id** is a **unique identifier** generated for each file upload session. It links:
- ✅ Uploaded files in S3
- ✅ Database records in KPI tables
- ✅ User's data (for data isolation)
- ✅ Processing status (PENDING/READY)

---

## 🔄 How ingestion_id is Generated

### Step 1: Generation Function

**Location:** `Server/utils/idGenerator.js`

**Code:**
```javascript
export function generateIngestionId() {
  // Format: ING + 5 random digits
  const randomBuffer = new Uint32Array(1);
  crypto.getRandomValues(randomBuffer); // Cryptographically secure random
  const randomNum = randomBuffer[0] % 100000; // 0-99999
  const paddedNum = randomNum.toString().padStart(5, '0'); // Always 5 digits
  return `ING${paddedNum}`;
}
```

**Example Output:**
```
ING83738
ING00123
ING99999
```

**Format Breakdown:**
- `ING` - Prefix (always)
- `83738` - 5 random digits (00000-99999)
- Always exactly 8 characters total: `ING` + 5 digits

### Step 2: When is it Generated?

**Generated when user uploads files:**

1. **User clicks "Upload & Analyze"**
2. **Server receives upload request** (`/api/upload` or `/api/upload-multiple`)
3. **Server calls `setKpiFlagPending(userEmail)`**
4. **Function generates new ingestion_id:**
   ```javascript
   const ingestionId = generateIngestionId();
   // Example: ING83738 (ING + 5 random digits)
   ```
5. **Stored in database:**
   ```sql
   INSERT INTO kpi_flags (email, ingestion_id, kpi_flag)
   VALUES ('user@example.com', 'ING83738', FALSE);
   ```
6. **Added to S3 metadata:**
   ```javascript
   Metadata: {
     'ingestion-id': 'ING83738',
     'user-email': 'user@example.com'
   }
   ```

---

## 📍 Where ingestion_id is Stored

### 1. Database: `kpi_flags` Table

**Table Structure:**
```sql
CREATE TABLE kpi_flags (
  email VARCHAR(255),
  ingestion_id VARCHAR(255),
  kpi_flag BOOLEAN,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

**Example Data:**
```
email              | ingestion_id | kpi_flag | created_at
-------------------|--------------|----------|-------------------
user@example.com   | ING83738     | false    | 2025-01-11 12:00:00
user@example.com   | ING12345     | true     | 2025-01-11 13:00:00
```

### 2. S3 Object Metadata

**Every uploaded file has metadata:**
```javascript
{
  'ingestion-id': 'ING83738',
  'user-email': 'user@example.com',
  'file-type': 'sales',
  'upload-timestamp': '2025-01-11T12:00:00.000Z'
}
```

### 3. Database: All KPI Tables

**Every KPI table has `ingestion_id` column:**
- `sales_kpi.ingestion_id`
- `inventory_kpi.ingestion_id`
- `customer_kpi.ingestion_id`
- `productivity_kpi.ingestion_id`
- `profitability_kpi.ingestion_id`
- `return_kpi.ingestion_id`
- `store_kpi.ingestion_id`

**Example:**
```sql
SELECT product_id, product_name, gross_sales, ingestion_id
FROM sales_kpi
WHERE ingestion_id = 'ING83738';
```

---

## 🔍 How to Check ingestion_id

### Method 1: Check Server Logs

**After uploading files, check logs:**
```bash
pm2 logs foresightflow --lines 50
```

**Look for:**
```
🔄 KPI processing flag set to PENDING for user@example.com with ingestion ID: ING83738
✅ File uploaded to S3 with ingestion-id: ING83738
```

### Method 2: Check Database (kpi_flags table)

**Connect to database:**
```bash
psql -h YOUR_DB_HOST -U YOUR_DB_USER -d foresightflow
```

**Query all ingestion IDs:**
```sql
-- See all ingestion IDs
SELECT email, ingestion_id, kpi_flag, created_at
FROM kpi_flags
ORDER BY created_at DESC;

-- Output:
-- email              | ingestion_id | kpi_flag | created_at
-- -------------------|--------------|----------|-------------------
-- user@example.com   | ING83738     | false    | 2025-01-11 12:00:00
-- user@example.com   | ING12345     | true     | 2025-01-11 13:00:00
```

**Query for specific user:**
```sql
-- Get all ingestion IDs for a user
SELECT ingestion_id, kpi_flag, created_at, updated_at
FROM kpi_flags
WHERE email = 'user@example.com'
ORDER BY created_at DESC;

-- Output:
-- ingestion_id | kpi_flag | created_at          | updated_at
-- -------------|----------|---------------------|-------------------
-- ING12345     | true     | 2025-01-11 13:00:00 | 2025-01-11 13:15:00
-- ING83738     | false    | 2025-01-11 12:00:00 | 2025-01-11 12:00:00
```

**Get latest completed ingestion:**
```sql
-- Get latest completed ingestion for user
SELECT ingestion_id, kpi_flag, updated_at
FROM kpi_flags
WHERE email = 'user@example.com' 
  AND kpi_flag = true
ORDER BY updated_at DESC
LIMIT 1;

-- Output:
-- ingestion_id | kpi_flag | updated_at
-- -------------|----------|-------------------
-- ING12345     | true     | 2025-01-11 13:15:00
```

### Method 3: Check S3 Object Metadata

**Using AWS CLI:**
```bash
# List all files in S3
aws s3 ls s3://your-bucket-name/ --recursive

# Get metadata for specific file
aws s3api head-object \
  --bucket your-bucket-name \
  --key uploads/sales/sales_20250111_120000.csv

# Output will show:
# "Metadata": {
#   "ingestion-id": "ING83738",
#   "user-email": "user@example.com",
#   "file-type": "sales"
# }
```

**Using AWS Console:**
1. Go to S3 → Your Bucket
2. Click on uploaded file
3. Click "Metadata" tab
4. See `ingestion-id` in metadata

### Method 4: Check Browser (Frontend)

**After upload, check browser console (F12):**
```javascript
// In browser console
// The ingestion_id is returned in the upload response
```

**Check Network Tab (F12 → Network):**
1. Find `POST /api/upload` request
2. Click on it
3. Go to "Response" tab
4. See:
```json
{
  "success": true,
  "data": {
    "ingestionId": "ING83738",
    "kpiStatus": "PENDING"
  }
}
```

**Check localStorage (F12 → Application → Local Storage):**
- May store ingestion_id temporarily during upload

### Method 5: Check API Response

**Test upload endpoint:**
```bash
# Upload a file (need JWT token)
TOKEN="your-jwt-token"

curl -X POST http://localhost:5000/api/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@sales.csv" \
  -F "fileType=sales"

# Response:
# {
#   "success": true,
#   "data": {
#     "ingestionId": "ING83738",
#     "kpiStatus": "PENDING"
#   }
# }
```

**Check KPI status:**
```bash
# Check status for specific ingestion_id
curl -X POST http://localhost:5000/api/get-kpi-analysis \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ingestionId": "ING83738"}'

# Response:
# {
#   "status": "PENDING" or "READY",
#   "ingestionId": "ING83738"
# }
```

---

## 🔄 Complete Flow: ingestion_id Lifecycle

### Step 1: Upload Files
```
User uploads files
    ↓
Server generates: ING83738
    ↓
Stored in kpi_flags: (email, ING83738, FALSE)
    ↓
Added to S3 metadata: ingestion-id = ING83738
    ↓
Returned to frontend: { ingestionId: "ING83738" }
```

### Step 2: Lambda Processing
```
Lambda triggered by S3 event
    ↓
Lambda reads S3 metadata: ingestion-id = ING83738
    ↓
Lambda processes files
    ↓
Lambda writes to database:
  INSERT INTO sales_kpi (..., ingestion_id)
  VALUES (..., 'ING83738')
    ↓
Lambda updates flag:
  UPDATE kpi_flags 
  SET kpi_flag = TRUE 
  WHERE ingestion_id = 'ING83738'
```

### Step 3: Dashboard Display
```
User opens dashboard
    ↓
Frontend requests: GET /api/kpi/sales
    ↓
Server gets user email from JWT token
    ↓
Server queries:
  SELECT ingestion_id FROM kpi_flags 
  WHERE email = 'user@example.com' AND kpi_flag = TRUE
  ORDER BY updated_at DESC LIMIT 1
    ↓
Server gets: ING12345 (latest completed)
    ↓
Server queries:
  SELECT * FROM sales_kpi 
  WHERE ingestion_id = 'ING12345'
    ↓
Server returns only user's data
```

---

## ✅ Verification Checklist

### After Upload:
- [ ] Check server logs for ingestion_id
- [ ] Check database `kpi_flags` table
- [ ] Check S3 metadata has ingestion-id
- [ ] Check API response has ingestionId

### After Lambda Processing:
- [ ] Check `kpi_flag` changed to TRUE
- [ ] Check KPI tables have data with ingestion_id
- [ ] Verify data count matches uploaded files

### For Dashboard:
- [ ] Check server logs show correct ingestion_id
- [ ] Verify only user's data is returned
- [ ] Check database queries filter by ingestion_id

---

## 🧪 Testing ingestion_id

### Test 1: Generate New ingestion_id
```javascript
// In Node.js console
const { generateIngestionId } = require('./Server/utils/idGenerator.js');
const id = generateIngestionId();
console.log(id);
// Output: ING83738 (or similar, 5 random digits)
```

### Test 2: Check Database
```sql
-- See all ingestion IDs
SELECT * FROM kpi_flags;

-- Count records per ingestion
SELECT ingestion_id, COUNT(*) as record_count
FROM sales_kpi
GROUP BY ingestion_id;
```

### Test 3: Verify Data Isolation
```sql
-- User A's data
SELECT COUNT(*) FROM sales_kpi 
WHERE ingestion_id IN (
  SELECT ingestion_id FROM kpi_flags 
  WHERE email = 'userA@example.com' AND kpi_flag = true
);

-- User B's data
SELECT COUNT(*) FROM sales_kpi 
WHERE ingestion_id IN (
  SELECT ingestion_id FROM kpi_flags 
  WHERE email = 'userB@example.com' AND kpi_flag = true
);

-- Should be different counts!
```

---

## 📝 Key Points

### ✅ Uniqueness
- Each upload gets a **unique** ingestion_id
- Format: `ING` + timestamp + random number
- Guaranteed unique (timestamp + random)

### ✅ Persistence
- Stored in database (`kpi_flags` table)
- Stored in S3 metadata
- Used in all KPI tables

### ✅ Data Isolation
- Each user's data has different ingestion_id
- Dashboard filters by ingestion_id
- Users only see their own data

### ✅ Status Tracking
- `kpi_flag = FALSE` → Processing (PENDING)
- `kpi_flag = TRUE` → Complete (READY)
- Dashboard uses latest completed ingestion_id

---

## 🔧 Troubleshooting

### Problem: ingestion_id Not Generated
**Check:**
```bash
# Check server logs
pm2 logs foresightflow --lines 50

# Look for error:
# ❌ Error setting KPI flag to PENDING
```

**Solution:**
- Check database connection
- Verify `kpi_flags` table exists
- Check `idGenerator.js` is imported correctly

### Problem: ingestion_id Not in S3
**Check:**
```bash
# Check S3 metadata
aws s3api head-object --bucket your-bucket --key path/to/file.csv
```

**Solution:**
- Verify upload code includes metadata
- Check S3 permissions

### Problem: Lambda Can't Read ingestion_id
**Check:**
- Lambda logs in AWS Console
- Verify S3 metadata is accessible

**Solution:**
- Ensure Lambda has S3 read permissions
- Check metadata key is `ingestion-id` (with hyphen)

### Problem: Dashboard Shows Wrong Data
**Check:**
```sql
-- Verify ingestion_id filtering
SELECT ingestion_id, COUNT(*) 
FROM sales_kpi 
GROUP BY ingestion_id;
```

**Solution:**
- Check `getLatestIngestionId()` function
- Verify `kpi_flag = TRUE` for completed ingestions
- Check server logs for which ingestion_id is used

---

## 📚 Summary

**ingestion_id Format:**
```
ING + 5 Random Digits (00000-99999)
Example: ING83738, ING00123, ING99999
Always 8 characters: ING + exactly 5 digits
```

**Where it's stored:**
1. ✅ Database: `kpi_flags` table
2. ✅ S3: Object metadata
3. ✅ Database: All KPI tables

**How to check:**
1. ✅ Server logs: `pm2 logs foresightflow`
2. ✅ Database: `SELECT * FROM kpi_flags`
3. ✅ S3: `aws s3api head-object`
4. ✅ Browser: Network tab, API response
5. ✅ API: Test endpoints with curl

**Purpose:**
- ✅ Links uploaded files to processed data
- ✅ Isolates data by user
- ✅ Tracks processing status
- ✅ Enables multi-user support

---

**Everything you need to know about ingestion_id!** 🎯

