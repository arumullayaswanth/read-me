# Fix: HTTPS Not Working - Backend API Connection Issue

## Problem
- HTTP works but HTTPS doesn't
- Error: "Unexpected token '<', "<!DOCTYPE ". is not valid JSON"
- This means the frontend is getting HTML instead of JSON from the backend API

## Root Cause
The ALB is routing ALL traffic to the frontend (port 3000), but API calls (`/api/*`) need to go to the backend (port 5000).

## Solution: Configure ALB to Route API Calls to Backend

### Step 1: Create Backend Target Group

1. Go to **EC2 → Target Groups → Create target group**

2. Configure:
   - **Target type:** Instances
   - **Protocol:** HTTP
   - **Port:** 5000 (Backend port)
   - **VPC:** Your VPC
   - **Health check path:** `/api/health`

3. Click **Next**

4. **Register targets:**
   - Select your EC2 instance
   - Click **Include as pending below**
   - Click **Create target group**

5. Wait for health check to show **healthy**

### Step 2: Update ALB Listener Rules

1. Go to **EC2 → Load Balancers → Your ALB → Listeners**

2. Click on **HTTPS:443** listener → **View/Edit rules**

3. **Add a new rule at the top** (rules are evaluated in order):

   **Rule 1: API Routes to Backend**
   - **IF:** Path is `/api/*`
   - **THEN:** Forward to → Select your **Backend Target Group** (port 5000)
   - Click **Save**

4. **Default rule** (should already exist):
   - **IF:** All other requests
   - **THEN:** Forward to → Your **Frontend Target Group** (port 3000)

5. **Rule order should be:**
   ```
   1. /api/* → Backend Target Group (port 5000)
   2. Default → Frontend Target Group (port 3000)
   ```

### Step 3: Rebuild Client with HTTPS API URL

On your EC2 instance:

```bash
cd ~/ForesightFlow/Client

# Set API URL to use HTTPS
export NEXT_PUBLIC_API_URL=https://foresightflow.site

# Clean and rebuild
rm -rf .next out
npm run build

# Restart Client
pm2 restart foresightflow-client
```

### Step 4: Update Server CORS (if needed)

The Server CORS should already allow all origins, but verify:

```bash
cd ~/ForesightFlow/Server
grep -A 10 "cors" server.js
# Should show: origin: function (origin, callback) { callback(null, true); }
```

If CORS is restrictive, update it to allow your domain.

### Step 5: Verify Security Groups

**Backend Security Group:**
- Allow **port 5000** from **ALB Security Group** (not 0.0.0.0/0)

**Frontend Security Group:**
- Allow **port 3000** from **ALB Security Group**

**ALB Security Group:**
- Allow **port 80** from 0.0.0.0/0
- Allow **port 443** from 0.0.0.0/0

### Step 6: Test

1. **Test Backend API directly:**
   ```bash
   curl https://foresightflow.site/api/health
   # Should return: {"status":"ok",...}
   ```

2. **Test Frontend:**
   ```
   https://foresightflow.site
   # Should load the login page
   ```

3. **Test Login:**
   - Try logging in
   - Check browser console (F12) for errors
   - Check Network tab - API calls should go to `https://foresightflow.site/api/...`

## Alternative: Single Target Group with Nginx

If you prefer, you can use Nginx on EC2 to route traffic:

1. **Install Nginx:**
   ```bash
   sudo yum install nginx -y  # Amazon Linux
   # or
   sudo apt install nginx -y  # Ubuntu
   ```

2. **Configure Nginx:**
   ```bash
   sudo nano /etc/nginx/conf.d/foresightflow.conf
   ```

3. **Nginx Config:**
   ```nginx
   server {
       listen 80;
       server_name _;

       # Backend API
       location /api {
           proxy_pass http://localhost:5000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_cache_bypass $http_upgrade;
       }

       # Frontend
       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

4. **Start Nginx:**
   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

5. **Update ALB Target Group:**
   - Point to port 80 (Nginx)
   - Nginx will route internally

## Quick Fix Summary

**Option 1: ALB Rules (Recommended)**
- Create Backend Target Group (port 5000)
- Add ALB rule: `/api/*` → Backend Target Group
- Rebuild Client with `NEXT_PUBLIC_API_URL=https://foresightflow.site`

**Option 2: Nginx Proxy**
- Install Nginx
- Configure to route `/api` to port 5000, `/` to port 3000
- Point ALB to port 80

## Verification Checklist

- [ ] Backend Target Group created (port 5000) and healthy
- [ ] ALB rule: `/api/*` → Backend Target Group
- [ ] ALB default rule: All other → Frontend Target Group
- [ ] Client rebuilt with `NEXT_PUBLIC_API_URL=https://foresightflow.site`
- [ ] Security Groups allow ALB → EC2 on ports 3000 and 5000
- [ ] `curl https://foresightflow.site/api/health` works
- [ ] Login page loads without errors
- [ ] Browser Network tab shows API calls to `https://foresightflow.site/api/...`

## Common Issues

**Issue:** Still getting HTML instead of JSON
- **Fix:** Check ALB rule order - `/api/*` rule must be BEFORE default rule

**Issue:** CORS errors
- **Fix:** Verify Server CORS allows your domain

**Issue:** 502 Bad Gateway
- **Fix:** Check target group health, verify Security Groups

**Issue:** API calls still going to HTTP
- **Fix:** Rebuild Client with HTTPS URL, clear browser cache

