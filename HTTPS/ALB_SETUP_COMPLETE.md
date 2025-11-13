# Complete ALB Setup with Backend API Routing

## Architecture

```
Internet → ALB (HTTPS) → EC2 Instance
                    ├─ /api/* → Backend (port 5000)
                    └─ /* → Frontend (port 3000)
```

## Step-by-Step Setup

### 1. Create Frontend Target Group (Port 3000)

1. **EC2 → Target Groups → Create target group**
2. **Settings:**
   - Name: `foresightflow-frontend`
   - Target type: Instances
   - Protocol: HTTP
   - Port: **3000**
   - VPC: Your VPC
   - Health check: `/` (or `/dashboard`)
3. **Register targets:** Your EC2 instance
4. **Create target group**

### 2. Create Backend Target Group (Port 5000)

1. **EC2 → Target Groups → Create target group**
2. **Settings:**
   - Name: `foresightflow-backend`
   - Target type: Instances
   - Protocol: HTTP
   - Port: **5000**
   - VPC: Your VPC
   - Health check: `/api/health`
3. **Register targets:** Your EC2 instance
4. **Create target group**

### 3. Configure ALB Listener Rules

1. **EC2 → Load Balancers → Your ALB → Listeners → HTTPS:443**

2. **Edit rules** - Add in this order:

   **Rule 1 (Priority 1):**
   ```
   IF: Path is /api/*
   THEN: Forward to → foresightflow-backend (port 5000)
   ```

   **Rule 2 (Default):**
   ```
   IF: All other requests
   THEN: Forward to → foresightflow-frontend (port 3000)
   ```

3. **Save rules**

### 4. Update Security Groups

**EC2 Security Group:**
- Allow port 3000 from ALB Security Group
- Allow port 5000 from ALB Security Group

**ALB Security Group:**
- Allow port 443 from 0.0.0.0/0 (HTTPS)
- Allow port 80 from 0.0.0.0/0 (HTTP redirect)

### 5. Rebuild Client with HTTPS API URL

```bash
# On EC2
cd ~/ForesightFlow/Client

# Set HTTPS API URL
export NEXT_PUBLIC_API_URL=https://foresightflow.site

# Rebuild
rm -rf .next out
npm run build

# Restart
pm2 restart foresightflow-client
```

### 6. Test

```bash
# Test backend API
curl https://foresightflow.site/api/health
# Should return: {"status":"ok",...}

# Test frontend
curl -I https://foresightflow.site
# Should return: HTTP/2 200
```

## Troubleshooting

### Check Target Group Health
```bash
# In AWS Console → Target Groups
# Both should show: Status = healthy
```

### Check ALB Rules Order
- `/api/*` rule MUST be before default rule
- Rules are evaluated top to bottom

### Check Security Groups
- EC2 SG must allow ALB SG on ports 3000 and 5000
- ALB SG must allow 0.0.0.0/0 on ports 80 and 443

### Verify API URL in Client
```bash
# Check what API URL the Client is using
cd Client
grep -r "NEXT_PUBLIC_API_URL" out/_next/static/chunks/*.js | head -1
# Should show: https://foresightflow.site
```

## Final Configuration

**ALB Listener Rules (HTTPS:443):**
1. `/api/*` → Backend Target Group (5000)
2. Default → Frontend Target Group (3000)

**Client Environment:**
- `NEXT_PUBLIC_API_URL=https://foresightflow.site`

**Server:**
- Running on port 5000
- CORS allows all origins
- Listening on 0.0.0.0:5000

**Client:**
- Running on port 3000
- Built with HTTPS API URL

