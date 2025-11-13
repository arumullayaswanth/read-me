#### Complete_step_by_step_GUIDE.md

# 📘 Complete End-to-End Guide: From Setup to Dashboard Results

## 🎯 Overview

This guide covers **everything** from starting the application to seeing dashboard results. Complete step-by-step instructions with verification at each stage.

---

## 📋 Table of Contents

1. [Application Setup & Start](#1-application-setup--start)
2. [User Registration & Login](#2-user-registration--login)
3. [Upload Data Files](#3-upload-data-files)
4. [Lambda Processing](#4-lambda-processing)
5. [Dashboard Display](#5-dashboard-display)
6. [How to Stop Application](#6-how-to-stop-application)
7. [How to Check Results](#7-how-to-check-results)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Application Setup & Start

### Step 1.1: Prerequisites Check

**Before starting, verify you have:**
- ✅ Node.js 18+ installed
- ✅ PostgreSQL database running
- ✅ AWS S3 bucket configured
- ✅ Environment variables set (.env files)
- ✅ PM2 installed globally

**Check Prerequisites:**
```bash
# Check Node.js version
node --version  # Should be v18.x or higher

# Check npm version
npm --version

# Check PM2 installation
pm2 --version

# Check if database is accessible
# (Test connection from Server directory)
cd Server
node -e "require('dotenv').config(); console.log('DB_HOST:', process.env.DB_HOST)"
```

### Step 1.2: Start Server (Backend)

**Navigate to Server directory:**
```bash
cd ~/ForesightFlow/Server
# or
cd C:\Users\YourName\Downloads\ForesightFlow\Server  # Windows
```

**Check environment variables:**
```bash
# Verify .env file exists
ls -la .env  # Linux/Mac
dir .env     # Windows

# Check key variables
cat .env | grep DB_HOST
cat .env | grep PORT
cat .env | grep AWS_S3_BUCKET
```

**Install dependencies (if needed):**
```bash
npm install
```

**Start Server with PM2:**
```bash
pm2 start ecosystem.config.cjs
```

**Verify Server Started:**
```bash
# Check PM2 status
pm2 status

# Should show:
# ┌────┬──────────────────┬─────────┬─────────┬──────────┐
# │ id │ name             │ status  │ cpu     │ memory   │
# ├────┼──────────────────┼─────────┼─────────┼──────────┤
# │ 0  │ foresightflow    │ online  │ 0%      │ 50mb     │
# └────┴──────────────────┴─────────┴─────────┴──────────┘

# Check Server logs
pm2 logs foresightflow --lines 20

# Should see:
# ✅ Server running on port 5000
# ✅ Database connection established
# ✅ S3 Bucket: your-bucket-name
```

**Test Server Health:**
```bash
# Test from terminal
curl http://localhost:5000/api/health

# Expected response:
# {"status":"ok","timestamp":"2025-01-11T12:00:00.000Z"}

# Or open in browser:
# http://localhost:5000/api/health
```

### Step 1.3: Start Client (Frontend)

**Navigate to Client directory:**
```bash
cd ~/ForesightFlow/Client
# or
cd C:\Users\YourName\Downloads\ForesightFlow\Client  # Windows
```

**Check if build exists:**
```bash
# Check if 'out' folder exists (required for static export)
ls -la out  # Linux/Mac
dir out     # Windows

# If 'out' folder doesn't exist, you need to build first
```

**Build Client (if needed):**
```bash
# Get Server IP/URL
# For local: http://localhost:5000
# For AWS: http://YOUR_EC2_IP:5000
# For production: https://api.yourdomain.com

# Build with correct API URL
NEXT_PUBLIC_API_URL=http://localhost:5000 npm run build

# Wait for build to complete (2-5 minutes)
# Should see: ✓ Compiled successfully
```

**Start Client with PM2:**
```bash
pm2 start ecosystem.config.cjs
```

**Verify Client Started:**
```bash
# Check PM2 status
pm2 status

# Should show both:
# ┌────┬─────────────────────────┬─────────┬─────────┬──────────┐
# │ id │ name                    │ status  │ cpu     │ memory   │
# ├────┼─────────────────────────┼─────────┼─────────┼──────────┤
# │ 0  │ foresightflow           │ online  │ 0%      │ 50mb     │
# │ 1  │ foresightflow-client    │ online  │ 0%      │ 30mb     │
# └────┴─────────────────────────┴─────────┴─────────┴──────────┘

# Check Client logs
pm2 logs foresightflow-client --lines 20

# Should see:
# HTTP Server running on port 3000
```

**Test Client:**
```bash
# Open in browser:
# http://localhost:3000
# or
# http://YOUR_EC2_IP:3000

# Should see the login/signup page
```

**Save PM2 Configuration:**
```bash
# Save current PM2 processes
pm2 save

# Setup PM2 to start on system reboot
pm2 startup
# Follow the instructions shown
```

### Step 1.4: Verify Everything is Running

**Complete Status Check:**
```bash
# Check all PM2 processes
pm2 status

# Check Server logs
pm2 logs foresightflow --lines 10

# Check Client logs
pm2 logs foresightflow-client --lines 10

# Test Server API
curl http://localhost:5000/api/health

# Test Client (open in browser)
# http://localhost:3000
```

**Expected Results:**
- ✅ Server: `online` status, running on port 5000
- ✅ Client: `online` status, running on port 3000
- ✅ Health endpoint: Returns `{"status":"ok"}`
- ✅ Browser: Shows login/signup page

---

## 2. User Registration & Login

### Step 2.1: Open Application in Browser

**Open Browser:**
```
http://localhost:3000        # Local development
http://YOUR_EC2_IP:3000      # AWS EC2
https://yourdomain.com        # Production
```

**Verify Page Loads:**
- ✅ Page should load without errors
- ✅ Should see "Sign Up" or "Sign In" buttons
- ✅ No console errors in browser DevTools (F12)

### Step 2.2: Register New Account

**Fill Registration Form:**
1. Click "Sign Up" button
2. Enter details:
   - **Email:** `test@example.com`
   - **Password:** `password123` (min 6 characters)
   - **Full Name:** `Test User`
   - **Phone:** `+1234567890`
   - **Business Name:** `Test Business`
   - **Business Type:** Select from dropdown
3. Click "Register" or "Submit"

**Verify Registration:**
```bash
# Check Server logs
pm2 logs foresightflow --lines 30

# Should see:
# ✅ Account created successfully
# ✅ User registered: test@example.com
```

**Check Browser:**
- ✅ Should redirect to `/connect-data` page
- ✅ Should see upload page (not login page)
- ✅ Check browser console (F12) - no errors

**Verify in Database (Optional):**
```sql
-- Connect to PostgreSQL
psql -h YOUR_DB_HOST -U YOUR_DB_USER -d foresightflow

-- Check user was created
SELECT id, email, name, business_name, created_at 
FROM users 
WHERE email = 'test@example.com';

-- Should return 1 row with user details
```

### Step 2.3: Login (If Already Registered)

**Fill Login Form:**
1. Click "Sign In" button
2. Enter:
   - **Email:** `test@example.com`
   - **Password:** `password123`
3. Click "Login"

**Verify Login:**
```bash
# Check Server logs
pm2 logs foresightflow --lines 20

# Should see:
# ✅ Login successful
# ✅ User authenticated: test@example.com
```

**Check Browser:**
- ✅ Should redirect to `/connect-data` page
- ✅ Check localStorage (F12 → Application → Local Storage):
  - Should have `auth_token` (JWT token)
  - Should have `userInfo` (user details)

**Verify Token:**
```javascript
// In browser console (F12)
console.log(localStorage.getItem('auth_token'));
// Should show JWT token (long string)

console.log(JSON.parse(localStorage.getItem('userInfo')));
// Should show user object with name, email, etc.
```

---

## 3. Upload Data Files

### Step 3.1: Prepare CSV Files

**Required Files:**
- `sales.csv`
- `product.csv`
- `customer.csv`
- `discount.csv`
- `date_dimension.csv`
- `return.csv`
- `store.csv`
- `inventory.csv`
- `festival.csv`

**Verify Files:**
- ✅ All files are CSV format
- ✅ Files have headers (column names)
- ✅ Files have data rows
- ✅ Files are not corrupted

### Step 3.2: Upload Files

**Navigate to Upload Page:**
- Should be at `/connect-data` after login
- If not, manually go to: `http://localhost:3000/connect-data`

**Upload Each File:**
1. Click on upload zone for "Sales Data"
2. Select `sales.csv` file
3. Repeat for all 9 files:
   - Sales Data → `sales.csv`
   - Product Data → `product.csv`
   - Customer Data → `customer.csv`
   - Discount Data → `discount.csv`
   - Date Dimension → `date_dimension.csv`
   - Return Data → `return.csv`
   - Store Data → `store.csv`
   - Inventory Data → `inventory.csv`
   - Festival Data → `festival.csv`
4. Click "Upload & Analyze" button

**Verify Upload Started:**
```bash
# Check Server logs
pm2 logs foresightflow --lines 50

# Should see for each file:
# ✅ File uploaded: sales.csv
# ✅ Ingestion ID generated: ingestion_20250111_123456_test@example.com
# ✅ File uploaded to S3: s3://bucket-name/path/to/file
```

**Check Browser:**
- ✅ Progress bar should appear
- ✅ Should show "Processing..." status
- ✅ Should show percentage (0% → 100%)

### Step 3.3: Verify Upload Completion

**Check Server Logs:**
```bash
pm2 logs foresightflow --lines 100

# Should see:
# ✅ All files uploaded successfully
# ✅ Ingestion ID: ingestion_20250111_123456_test@example.com
# ✅ KPI flag set to PENDING
```

**Check S3 Bucket:**
```bash
# Using AWS CLI
aws s3 ls s3://your-bucket-name/ --recursive

# Should see uploaded files with timestamps
```

**Check Database:**
```sql
-- Check kpi_flags table
SELECT email, ingestion_id, kpi_flag, created_at
FROM kpi_flags
WHERE email = 'test@example.com'
ORDER BY created_at DESC;

-- Should see:
-- email              | ingestion_id                          | kpi_flag | created_at
-- test@example.com   | ingestion_20250111_123456_...        | false    | 2025-01-11 12:00:00
```

**Check Browser:**
- ✅ Should show "Upload Complete!"
- ✅ Should show ingestion ID
- ✅ Should start polling for KPI status

---

## 4. Lambda Processing

### Step 4.1: Verify Lambda Triggered

**Check Lambda Logs (AWS Console):**
1. Go to AWS Console → Lambda
2. Find your Lambda function
3. Click "Monitor" → "View logs in CloudWatch"
4. Check recent log streams

**Expected Logs:**
```
✅ Lambda triggered by S3 event
✅ Processing file: sales.csv
✅ Ingestion ID: ingestion_20250111_123456_test@example.com
✅ Reading metadata from S3
```

### Step 4.2: Monitor Lambda Processing

**Watch Lambda Logs:**
```bash
# Using AWS CLI (if configured)
aws logs tail /aws/lambda/your-function-name --follow

# Or check in AWS Console
```

**Expected Progress:**
```
✅ Processing sales.csv...
✅ Processing product.csv...
✅ Processing customer.csv...
✅ Calculating KPIs...
✅ Writing to database...
```

**Processing Time:**
- Small files (< 1,000 rows): 1-2 minutes
- Medium files (1,000-10,000 rows): 5-10 minutes
- Large files (10,000-50,000 rows): 10-15 minutes
- Very large files (> 50,000 rows): 15-30 minutes

### Step 4.3: Verify Lambda Completion

**Check Lambda Logs:**
```
✅ All files processed successfully
✅ Data written to database with ingestion_id
✅ KPI flag updated to TRUE
```

**Check Database:**
```sql
-- Check kpi_flags (should be TRUE now)
SELECT email, ingestion_id, kpi_flag, updated_at
FROM kpi_flags
WHERE email = 'test@example.com'
ORDER BY updated_at DESC
LIMIT 1;

-- Should show:
-- kpi_flag: true
-- updated_at: recent timestamp

-- Check data was inserted
SELECT COUNT(*) as total_records, ingestion_id
FROM sales_kpi
WHERE ingestion_id = 'ingestion_20250111_123456_test@example.com'
GROUP BY ingestion_id;

-- Should return count > 0
```

**Check Browser:**
- ✅ Should show "Processing Complete!"
- ✅ Status should change from "PENDING" to "READY"
- ✅ Should redirect or show success message

---

## 5. Dashboard Display

### Step 5.1: Navigate to Dashboard

**Open Dashboard:**
1. After upload completes, click "View Dashboard"
2. Or manually navigate to: `http://localhost:3000/dashboard/sales`

**Verify Page Loads:**
- ✅ Dashboard page should load
- ✅ Should see loading spinner initially
- ✅ Should see charts and metrics after loading

### Step 5.2: Verify Data Display

**Check Browser Console (F12):**
```javascript
// Should see:
// ✅ Fetching sales KPI data for user: test@example.com
// ✅ Using latest ingestion ID: ingestion_20250111_123456_test@example.com
// ✅ Dashboard data loaded
```

**Check Network Tab (F12 → Network):**
```
GET /api/kpi/sales
Status: 200 OK
Response: {
  "success": true,
  "data": {
    "sales": [...],  // Array of sales records
    "summary": {...}  // Summary statistics
  }
}
```

**Check Server Logs:**
```bash
pm2 logs foresightflow --lines 30

# Should see:
# ✅ Fetching sales KPI data for user: test@example.com
# ✅ Using latest ingestion ID: ingestion_20250111_123456_test@example.com
# ✅ Sales data fetched: X records
```

### Step 5.3: Verify Dashboard Content

**Check Dashboard Shows:**
- ✅ Charts are rendered (not blank)
- ✅ Metrics display numbers (not 0 or null)
- ✅ Data matches uploaded files
- ✅ Summary statistics are correct

**Verify Data Isolation:**
```sql
-- Check only user's data is returned
SELECT COUNT(*) 
FROM sales_kpi 
WHERE ingestion_id = 'ingestion_20250111_123456_test@example.com';

-- Compare with dashboard count
-- Should match!
```

**Test Other Dashboards:**
- Navigate to `/dashboard/inventory`
- Navigate to `/dashboard/products`
- Navigate to `/dashboard/financial`
- All should show data

---

## 6. How to Stop Application

### Step 6.1: Stop Client (Frontend)

**Stop Client:**
```bash
# Stop Client only
pm2 stop foresightflow-client

# Verify stopped
pm2 status
# Should show: status: stopped
```

**Or Delete Client Process:**
```bash
pm2 delete foresightflow-client
```

### Step 6.2: Stop Server (Backend)

**Stop Server:**
```bash
# Stop Server only
pm2 stop foresightflow

# Verify stopped
pm2 status
# Should show: status: stopped
```

**Or Delete Server Process:**
```bash
pm2 delete foresightflow
```

### Step 6.3: Stop All Applications

**Stop Everything:**
```bash
# Stop all PM2 processes
pm2 stop all

# Verify all stopped
pm2 status
# All should show: status: stopped
```

**Or Delete All:**
```bash
pm2 delete all
```

### Step 6.4: Verify Everything Stopped

**Check Status:**
```bash
pm2 status

# Should show empty or all stopped:
# ┌────┬──────┬─────────┬─────────┬──────────┐
# │ id │ name │ status  │ cpu     │ memory   │
# └────┴──────┴─────────┴─────────┴──────────┘
```

**Test Endpoints:**
```bash
# Should fail (connection refused)
curl http://localhost:5000/api/health
# Error: Connection refused

# Browser should show error
# http://localhost:3000
# Error: This site can't be reached
```

---

## 7. How to Check Results

### Step 7.1: Check Application Status

**PM2 Status:**
```bash
pm2 status

# Should show:
# ✅ Both processes online
# ✅ CPU and memory usage normal
# ✅ No restarts (restart count = 0)
```

**Process Health:**
```bash
# Check if processes are running
ps aux | grep node
# Should see node processes

# Check ports are listening
netstat -tulpn | grep 5000  # Server
netstat -tulpn | grep 3000  # Client
# Or
lsof -i :5000  # Server
lsof -i :3000  # Client
```

### Step 7.2: Check Server Logs

**View Recent Logs:**
```bash
# Server logs
pm2 logs foresightflow --lines 50

# Client logs
pm2 logs foresightflow-client --lines 50

# All logs
pm2 logs --lines 50
```

**Check for Errors:**
```bash
# Server errors only
pm2 logs foresightflow --err --lines 50

# Client errors only
pm2 logs foresightflow-client --err --lines 50
```

**Expected Logs (No Errors):**
- ✅ Server running on port 5000
- ✅ Database connection established
- ✅ No error messages
- ✅ API requests logged successfully

### Step 7.3: Check API Endpoints

**Test Health Endpoint:**
```bash
curl http://localhost:5000/api/health

# Expected:
# {"status":"ok","timestamp":"2025-01-11T12:00:00.000Z"}
```

**Test KPI Endpoint (with auth):**
```bash
# Get token from browser localStorage first
TOKEN="your-jwt-token-here"

curl -H "Authorization: Bearer $TOKEN" \
     http://localhost:5000/api/kpi/sales

# Expected:
# {"success":true,"data":{"sales":[...],"summary":{...}}}
```

### Step 7.4: Check Database

**Connect to Database:**
```bash
psql -h YOUR_DB_HOST -U YOUR_DB_USER -d foresightflow
```

**Check Users:**
```sql
SELECT COUNT(*) as total_users FROM users;
SELECT email, name, created_at FROM users ORDER BY created_at DESC LIMIT 5;
```

**Check KPI Flags:**
```sql
SELECT email, ingestion_id, kpi_flag, created_at, updated_at
FROM kpi_flags
ORDER BY created_at DESC
LIMIT 10;
```

**Check KPI Data:**
```sql
-- Check sales data
SELECT COUNT(*) as total_records, 
       COUNT(DISTINCT ingestion_id) as unique_ingestions
FROM sales_kpi;

-- Check data for specific user
SELECT COUNT(*) as user_records
FROM sales_kpi
WHERE ingestion_id IN (
  SELECT ingestion_id 
  FROM kpi_flags 
  WHERE email = 'test@example.com' AND kpi_flag = true
);
```

### Step 7.5: Check S3 Bucket

**List Uploaded Files:**
```bash
aws s3 ls s3://your-bucket-name/ --recursive

# Should see:
# - Uploaded CSV files
# - Files with correct timestamps
# - Files with metadata (ingestion-id)
```

**Check File Metadata:**
```bash
aws s3api head-object \
  --bucket your-bucket-name \
  --key path/to/sales.csv

# Should show metadata:
# "Metadata": {
#   "ingestion-id": "ingestion_20250111_123456_test@example.com",
#   "user-email": "test@example.com"
# }
```

### Step 7.6: Check Browser

**Open Browser DevTools (F12):**
- **Console Tab:** No red errors
- **Network Tab:** All requests return 200 OK
- **Application Tab:** 
  - `auth_token` exists in localStorage
  - `userInfo` exists in localStorage

**Test User Flow:**
1. ✅ Can login/register
2. ✅ Can upload files
3. ✅ Can see dashboard
4. ✅ Data displays correctly
5. ✅ No console errors

---

## 8. Troubleshooting

### Problem 1: Server Won't Start

**Symptoms:**
- `pm2 status` shows `errored` or `stopped`
- Port 5000 not accessible

**Solutions:**
```bash
# Check logs for errors
pm2 logs foresightflow --err --lines 50

# Common issues:
# 1. Port already in use
sudo lsof -i :5000
sudo kill -9 <PID>

# 2. Database connection failed
# Check .env file has correct DB credentials
cat Server/.env | grep DB_

# 3. Missing dependencies
cd Server
npm install

# 4. Restart server
pm2 restart foresightflow
```

### Problem 2: Client Won't Start

**Symptoms:**
- `pm2 status` shows `errored`
- Port 3000 not accessible
- Browser shows error

**Solutions:**
```bash
# Check logs
pm2 logs foresightflow-client --err --lines 50

# Common issues:
# 1. 'out' folder missing (need to build)
cd Client
NEXT_PUBLIC_API_URL=http://localhost:5000 npm run build
pm2 restart foresightflow-client

# 2. Port already in use
sudo lsof -i :3000
sudo kill -9 <PID>

# 3. Missing dependencies
cd Client
npm install
```

### Problem 3: Dashboard Shows No Data

**Symptoms:**
- Dashboard loads but shows empty/zero data
- Charts are blank

**Solutions:**
```bash
# 1. Check if data exists in database
psql -h YOUR_DB_HOST -U YOUR_DB_USER -d foresightflow
SELECT COUNT(*) FROM sales_kpi WHERE ingestion_id = 'your-ingestion-id';

# 2. Check server logs
pm2 logs foresightflow --lines 50
# Look for: "Using latest ingestion ID"

# 3. Check browser console (F12)
# Look for API errors

# 4. Verify authentication
# Check localStorage has auth_token
```

### Problem 4: Upload Fails

**Symptoms:**
- Files don't upload
- Error message shown

**Solutions:**
```bash
# 1. Check server logs
pm2 logs foresightflow --lines 100

# 2. Check S3 permissions
aws s3 ls s3://your-bucket-name/

# 3. Check file size (max 50MB)
ls -lh your-file.csv

# 4. Check authentication
# Verify JWT token in browser localStorage
```

### Problem 5: Lambda Not Processing

**Symptoms:**
- Files uploaded but no data in dashboard
- KPI flag stays FALSE

**Solutions:**
```bash
# 1. Check Lambda logs in AWS Console
# AWS Console → Lambda → Your Function → Monitor → Logs

# 2. Check S3 event trigger
# AWS Console → S3 → Your Bucket → Properties → Event Notifications

# 3. Check database for errors
SELECT * FROM kpi_flags WHERE kpi_flag = false ORDER BY created_at DESC;

# 4. Verify Lambda has correct permissions
# - S3 read access
# - Database write access
```

---

## 🔄 Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│              STEP 1: START APPLICATION                     │
│  1. cd Server && pm2 start ecosystem.config.cjs            │
│  2. cd Client && pm2 start ecosystem.config.cjs            │
│  3. pm2 status (verify both online)                        │
│  4. Test: curl http://localhost:5000/api/health           │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              STEP 2: USER REGISTRATION                     │
│  1. Open http://localhost:3000                             │
│  2. Click "Sign Up"                                        │
│  3. Fill form and submit                                   │
│  4. Verify: Check localStorage for auth_token             │
│  5. Verify: Check database for new user                    │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              STEP 3: UPLOAD FILES                          │
│  1. Navigate to /connect-data                              │
│  2. Upload all 9 CSV files                                 │
│  3. Click "Upload & Analyze"                              │
│  4. Verify: Check server logs for ingestion_id            │
│  5. Verify: Check S3 bucket for uploaded files            │
│  6. Verify: Check database kpi_flags (kpi_flag = false)    │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              STEP 4: LAMBDA PROCESSING                     │
│  1. Lambda triggered by S3 event                          │
│  2. Lambda processes files                                 │
│  3. Lambda writes to database with ingestion_id            │
│  4. Lambda sets kpi_flag = TRUE                           │
│  5. Verify: Check Lambda logs in AWS Console               │
│  6. Verify: Check database kpi_flags (kpi_flag = true)    │
│  7. Verify: Check database has data with ingestion_id     │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              STEP 5: VIEW DASHBOARD                        │
│  1. Navigate to /dashboard/sales                            │
│  2. Dashboard fetches data from /api/kpi/sales             │
│  3. Server filters by user's ingestion_id                  │
│  4. Dashboard displays data                                │
│  5. Verify: Check browser console (no errors)              │
│  6. Verify: Check network tab (200 OK)                     │
│  7. Verify: Check dashboard shows data                     │
└────────────────────┬──────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              STEP 6: CHECK RESULTS                          │
│  1. pm2 status (verify both online)                        │
│  2. pm2 logs (check for errors)                            │
│  3. curl /api/health (test API)                            │
│  4. Check database (verify data)                           │
│  5. Check browser (verify display)                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 📝 Quick Reference Commands

### Start Application
```bash
# Start Server
cd Server && pm2 start ecosystem.config.cjs

# Start Client
cd Client && pm2 start ecosystem.config.cjs

# Save configuration
pm2 save
```

### Stop Application
```bash
# Stop all
pm2 stop all

# Stop individual
pm2 stop foresightflow
pm2 stop foresightflow-client
```

### Check Status
```bash
# PM2 status
pm2 status

# View logs
pm2 logs
pm2 logs foresightflow --lines 50
pm2 logs foresightflow-client --lines 50

# Monitor
pm2 monit
```

### Restart Application
```bash
# Restart all
pm2 restart all

# Restart individual
pm2 restart foresightflow
pm2 restart foresightflow-client
```

### Test Endpoints
```bash
# Health check
curl http://localhost:5000/api/health

# KPI endpoint (need token)
curl -H "Authorization: Bearer YOUR_TOKEN" \
     http://localhost:5000/api/kpi/sales
```

---

## ✅ Verification Checklist

### Application Setup
- [ ] Server starts successfully
- [ ] Client starts successfully
- [ ] Both show `online` in `pm2 status`
- [ ] Health endpoint returns `{"status":"ok"}`
- [ ] Browser shows login page

### User Registration
- [ ] Can register new account
- [ ] Redirected to upload page
- [ ] `auth_token` in localStorage
- [ ] `userInfo` in localStorage
- [ ] User exists in database

### File Upload
- [ ] Can upload all 9 files
- [ ] Progress bar shows
- [ ] Server logs show ingestion_id
- [ ] Files in S3 bucket
- [ ] `kpi_flags` record created (kpi_flag = false)

### Lambda Processing
- [ ] Lambda triggered
- [ ] Lambda processes files
- [ ] Lambda logs show success
- [ ] `kpi_flag` updated to true
- [ ] Data in database with ingestion_id

### Dashboard Display
- [ ] Dashboard loads
- [ ] Data displays correctly
- [ ] Charts render
- [ ] No console errors
- [ ] API returns 200 OK

---

## 📚 Related Files

- **Database Schema:** `CORRECTED_DATABASE_SCHEMA.md`
- **Lambda Guide:** `LAMBDA_INGESTION_ID_GUIDE.md`
- **Performance:** `LARGE_DATA_PERFORMANCE_GUIDE.md`
- **Deployment:** `AWS_EC2_DEPLOYMENT.md`
- **ALB Setup:** `ALB_SETUP_COMPLETE.md`

---

## ✅ Summary

**Complete End-to-End Flow:**
1. **Start** → Server + Client with PM2
2. **Register** → Create account, get JWT token
3. **Upload** → Upload CSV files, get ingestion_id
4. **Process** → Lambda processes, writes to database
5. **Display** → Dashboard shows user's data
6. **Verify** → Check logs, database, browser
7. **Stop** → Stop PM2 processes when done

**Result:** Complete working system from start to finish! 🎯
