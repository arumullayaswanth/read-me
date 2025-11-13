# 🚀 ForesightFlow - AWS EC2 Deployment Guide

Complete guide to deploy ForesightFlow on AWS EC2 instance.

## 📋 Prerequisites

- AWS EC2 instance (Ubuntu 20.04/22.04 or Amazon Linux 2)
- SSH access to EC2 instance
- Node.js 18+ installed
- PostgreSQL database (can be RDS or EC2 instance)
- AWS S3 bucket configured
- Domain name (optional, for production)

## 🔧 Step 1: Initial Server Setup

### 1.1 Connect to EC2 Instance
```bash
ssh -i your-key.pem ubuntu@your-ec2-ip
# or
ssh -i your-key.pem ec2-user@your-ec2-ip  # For Amazon Linux
```

### 1.2 Update System
```bash
sudo apt update && sudo apt upgrade -y  # Ubuntu
# or
sudo yum update -y  # Amazon Linux
```

### 1.3 Install Node.js 18+
```bash
# Ubuntu
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Amazon Linux
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Verify
node --version  # Should be v18.x or higher
npm --version
```

### 1.4 Install PM2 Globally
```bash
sudo npm install -g pm2
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

### 1.5 Install Git
```bash
sudo apt install git -y  # Ubuntu
# or
sudo yum install git -y  # Amazon Linux
```

## 📦 Step 2: Clone and Setup Project

### 2.1 Clone Repository
```bash
cd ~
git clone <your-repository-url> ForesightFlow
cd ForesightFlow
```

### 2.2 Setup Server

```bash
cd Server

# Install dependencies
npm install

# Create .env file
nano .env
```

**Server .env file content:**
```env
# Database Configuration
DB_HOST=your-database-host
DB_PORT=5432
DB_NAME=foresightflow
DB_USERNAME=your-db-username
DB_PASSWORD=your-db-password
DB_SSL=false

# Database Reader (if using read replica, otherwise same as DB_HOST)
DB_READER_HOST=your-database-host
DB_READER_PORT=5432

# Server Configuration
PORT=5000
NODE_ENV=production

# AWS S3 Configuration
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_S3_BUCKET_NAME=your-bucket-name

# Session Secret (generate a random string)
SESSION_SECRET=your-random-session-secret-here

# Mock Data (set to false for production)
USE_MOCK_DATA=false
```

### 2.3 Setup Client

```bash
cd ../Client

# Install dependencies
npm install

# Get Server IP address
SERVER_IP=$(hostname -I | awk '{print $1}')
echo "Server IP: $SERVER_IP"

# Create .env.local file with Server URL
echo "NEXT_PUBLIC_API_URL=http://$SERVER_IP:5000" > .env.local
# OR if you have a domain:
# echo "NEXT_PUBLIC_API_URL=https://your-domain.com" > .env.local

# Build Client (this will take a few minutes)
npm run build
```

## 🚀 Step 3: Start Services with PM2

### 3.1 Start Server
```bash
cd ~/ForesightFlow/Server

# Create logs directory
mkdir -p logs

# Start with PM2
pm2 start ecosystem.config.cjs

# Save PM2 configuration
pm2 save
```

### 3.2 Start Client
```bash
cd ~/ForesightFlow/Client

# Create logs directory
mkdir -p logs

# Start with PM2
pm2 start ecosystem.config.cjs

# Save PM2 configuration
pm2 save
```

### 3.3 Setup PM2 to Start on Boot
```bash
pm2 startup
# Run the command that PM2 outputs (usually something like):
# sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

## 🔒 Step 4: Configure Security Groups

### 4.1 AWS Security Group Rules

In AWS Console → EC2 → Security Groups, allow:

**Inbound Rules:**
- Port 22 (SSH) - Your IP only
- Port 3000 (Client) - 0.0.0.0/0 (or your IP)
- Port 5000 (Server) - 0.0.0.0/0 (or your IP)
- Port 80 (HTTP) - 0.0.0.0/0 (if using Nginx)
- Port 443 (HTTPS) - 0.0.0.0/0 (if using SSL)

### 4.2 Configure Firewall (UFW)
```bash
#ubuntu
sudo apt update
sudo apt install ufw -y
sudo ufw allow 22/tcp
sudo ufw allow 3000/tcp
sudo ufw allow 5000/tcp
sudo ufw enable

# Amazon2023
# For Amazon Linux 2023 (check if firewalld is active)
sudo systemctl status firewalld

# If active, allow ports:
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

```
3. Check firewall
```bash
# On EC2 - test locally
curl http://localhost:5000/api/health

# Check what it's listening on
sudo netstat -tulpn | grep 5000
# Should show: 0.0.0.0:5000 (not 127.0.0.1:5000)
```

---

## 🌐 Step 5: Setup Nginx (Optional but Recommended)

### 5.1 Install Nginx
```bash
sudo apt install nginx -y  # Ubuntu
# or
sudo yum install nginx -y  # Amazon Linux
```

### 5.2 Configure Nginx
```bash
sudo vim /etc/nginx/sites-available/foresightflow
```

**Nginx Configuration:**
```nginx
server {
    listen 80;
    server_name your-domain.com;  # or your EC2 IP

    # Client (Frontend)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Server API
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
}
```

### 5.3 Enable and Start Nginx
```bash
sudo ln -s /etc/nginx/sites-available/foresightflow /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl enable nginx
```




## ✅ Step 6: Verify Deployment

### 6.1 Check PM2 Status
```bash
pm2 status
# Both processes should be "online"
```

### 6.2 Check Logs
```bash
# Server logs
pm2 logs foresightflow --lines 50

# Client logs
pm2 logs foresightflow-client --lines 50
```

### 6.3 Test Endpoints
```bash
# Test Server health
curl http://localhost:5000/api/health

# Test from browser
# Open: http://your-ec2-ip:3000
# Or: http://your-domain.com (if Nginx configured)
```

## 🔄 Step 7: Update Code (After Git Pull)

```bash
cd ~/ForesightFlow

# Pull latest code
git pull

# Update Server
cd Server
npm install
pm2 restart foresightflow

# Update Client (rebuild required)
cd ../Client
npm install

# Get Server IP
SERVER_IP=$(hostname -I | awk '{print $1}')

# Rebuild with correct API URL
NEXT_PUBLIC_API_URL=http://$SERVER_IP:5000 npm run build

# Restart Client
pm2 restart foresightflow-client
```

## 📊 Useful PM2 Commands

```bash
# View status
pm2 status

# View logs
pm2 logs
pm2 logs foresightflow
pm2 logs foresightflow-client

# Restart
pm2 restart foresightflow
pm2 restart foresightflow-client
pm2 restart all

# Stop
pm2 stop foresightflow

# Delete
pm2 delete foresightflow

# Monitor
pm2 monit
```

## 🐛 Troubleshooting

### Server not starting?
```bash
# Check logs
pm2 logs foresightflow --err

# Check if port is in use
sudo netstat -tulpn | grep 5000

# Verify .env file
cat Server/.env
```

### Client not loading?
```bash
# Check if build exists
ls -la Client/out

# Rebuild Client
cd Client
rm -rf .next out
NEXT_PUBLIC_API_URL=http://YOUR_SERVER_IP:5000 npm run build
pm2 restart foresightflow-client
```

### Database connection issues?
```bash
# Test database connection
cd Server
node -e "require('dotenv').config(); const {Pool} = require('pg'); const pool = new Pool({host: process.env.DB_HOST, port: process.env.DB_PORT, database: process.env.DB_NAME, user: process.env.DB_USERNAME, password: process.env.DB_PASSWORD}); pool.query('SELECT NOW()').then(r => console.log('Connected:', r.rows[0])).catch(e => console.error('Error:', e.message));"
```

### Port already in use?
```bash
# Find process using port
sudo lsof -i :5000
sudo lsof -i :3000

# Kill process
sudo kill -9 <PID>
```

## 🔐 Security Best Practices

1. **Use Environment Variables**: Never commit `.env` files
2. **Use HTTPS**: Setup SSL certificate (Let's Encrypt)
3. **Restrict Security Groups**: Only allow necessary IPs
4. **Regular Updates**: Keep system and dependencies updated
5. **Firewall**: Use UFW or similar
6. **Database**: Use RDS with VPC, not public access
7. **Secrets**: Use AWS Secrets Manager for sensitive data

## 📝 Notes

- **Client uses static export**: Must rebuild after changing `NEXT_PUBLIC_API_URL`
- **Server uses real data**: `USE_MOCK_DATA=false` in production
- **PM2 auto-restart**: Services restart automatically on server reboot
- **Logs location**: `Server/logs/` and `Client/logs/`

## 🎯 Quick Reference

```bash
# Start everything
cd ~/ForesightFlow/Server && pm2 start ecosystem.config.cjs
cd ~/ForesightFlow/Client && pm2 start ecosystem.config.cjs
pm2 save

# Stop everything
pm2 stop all

# Restart everything
pm2 restart all

# View all logs
pm2 logs

# Check status
pm2 status
```

---

**Need help?** Check logs: `pm2 logs` or check individual service logs.







# Troubleshoot: Can't Access Server on AWS EC2

## Problem
Server is running with PM2 but can't be accessed from browser: `ERR_CONNECTION_TIMED_OUT`

## Quick Fix Steps

### Step 1: Check if Server is Listening on All Interfaces

The Server should listen on `0.0.0.0`, not just `localhost`. Check your server.js:

```bash
cd Server
grep -n "app.listen" server.js
# Should show something like: app.listen(PORT, '0.0.0.0', ...)
# OR: app.listen(PORT) which defaults to 0.0.0.0
```

If it's listening on `localhost` only, you need to change it to `0.0.0.0`.

### Step 2: Check AWS Security Group Rules

**In AWS Console:**
1. Go to **EC2 → Instances**
2. Select your instance
3. Click **Security** tab
4. Click on the Security Group name
5. Click **Edit inbound rules**

**Add these rules if missing:**
- **Type**: Custom TCP
- **Port**: 5000
- **Source**: 0.0.0.0/0 (or your specific IP for security)
- **Description**: ForesightFlow Server API

- **Type**: Custom TCP  
- **Port**: 3000
- **Source**: 0.0.0.0/0 (or your specific IP)
- **Description**: ForesightFlow Client

- **Type**: SSH
- **Port**: 22
- **Source**: Your IP only (for security)

### Step 3: Check Firewall (UFW) on EC2

```bash
# Check UFW status
sudo ufw status

# If it's active and blocking, allow the ports:
sudo ufw allow 5000/tcp
sudo ufw allow 3000/tcp
sudo ufw allow 22/tcp

# If UFW is not installed (Amazon Linux 2023), use firewalld:
sudo systemctl status firewalld

# If firewalld is active, allow ports:
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

### Step 4: Test Server Locally on EC2

```bash
# Test if Server is responding locally
curl http://localhost:5000/api/health

# Test if Server is listening on all interfaces
sudo netstat -tulpn | grep 5000
# Should show: 0.0.0.0:5000 (not 127.0.0.1:5000)

# Or use ss command
sudo ss -tulpn | grep 5000
```

### Step 5: Check Server Logs

```bash
pm2 logs foresightflow --lines 50
# Look for: "Server running on port 5000"
# Check for any errors
```

### Step 6: Verify Server is Accessible from Outside

```bash
# From your local machine (not EC2), test:
curl http://172.31.10.248:5000/api/health

# If this works, the Server is accessible
# If it times out, check Security Groups and Firewall
```

## Common Issues

### Issue 1: Server Only Listening on Localhost

**Fix:**
```bash
cd Server
# Check server.js - the listen call should be:
# app.listen(PORT, '0.0.0.0', ...)
# OR just: app.listen(PORT) which defaults to 0.0.0.0

# If it says: app.listen(PORT, 'localhost', ...)
# Change it to: app.listen(PORT, '0.0.0.0', ...)
```

### Issue 2: Security Group Not Configured

**Fix:**
- Go to AWS Console → EC2 → Security Groups
- Add inbound rule for port 5000 (and 3000 for Client)
- Source: 0.0.0.0/0 (or your IP for better security)

### Issue 3: Firewall Blocking

**For Ubuntu:**
```bash
sudo ufw allow 5000/tcp
sudo ufw allow 3000/tcp
```

**For Amazon Linux 2023:**
```bash
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

### Issue 4: Wrong Port in URL

Make sure you're accessing:
- Server API: `http://172.31.10.248:5000/api/health`
- Client: `http://172.31.10.248:3000` (after starting Client)

## Quick Test Commands

```bash
# On EC2 - Test locally
curl http://localhost:5000/api/health

# On EC2 - Check what's listening
sudo netstat -tulpn | grep -E ':(3000|5000)'

# On EC2 - Check firewall
sudo ufw status
# OR
sudo firewall-cmd --list-all

# From your local machine - Test external access
curl http://172.31.10.248:5000/api/health
```

## Complete Checklist

- [ ] Server is running: `pm2 status` shows "online"
- [ ] Server listens on 0.0.0.0: `sudo netstat -tulpn | grep 5000`
- [ ] Security Group allows port 5000: Check AWS Console
- [ ] Firewall allows port 5000: `sudo ufw status` or `sudo firewall-cmd --list-all`
- [ ] Server responds locally: `curl http://localhost:5000/api/health`
- [ ] Server responds externally: `curl http://172.31.10.248:5000/api/health` (from your machine)

## Next Steps After Fixing

Once Server is accessible:

1. **Start Client:**
```bash
cd ~/ForesightFlow/Client
pm2 start ecosystem.config.cjs
pm2 save
```

2. **Access Application:**
- Client: `http://172.31.10.248:3000`
- Server API: `http://172.31.10.248:5000/api/health`


