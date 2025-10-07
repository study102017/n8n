# Render.com Quick Start Guide for n8n

This is a condensed version of the [complete deployment guide](./DEPLOY_TO_RENDER.md). Use this for quick reference if you're already familiar with Render.com.

## 🚀 Quick Deploy (5 minutes)

### Step 1: Create PostgreSQL Database
```
Dashboard → New + → PostgreSQL
- Name: n8n-database
- Plan: Starter ($7/mo)
- Save the Internal Database URL
```

### Step 2: Create Web Service
```
Dashboard → New + → Web Service
- Deploy from: docker.n8n.io/n8nio/n8n
- Name: n8n-app
- Instance: Starter ($7/mo)
```

### Step 3: Essential Environment Variables
```bash
# Database (choose one method)
# Method 1: Use connection URL (easier)
DB_TYPE=postgresdb
DB_POSTGRESDB_CONNECTION_URL=<your-internal-database-url>

# Method 2: Individual settings
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=<hostname>
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=<username>
DB_POSTGRESDB_PASSWORD=<password>
DB_POSTGRESDB_SCHEMA=public

# Core Configuration
N8N_HOST=<your-app>.onrender.com
WEBHOOK_URL=https://<your-app>.onrender.com/
N8N_PROTOCOL=https
NODE_ENV=production

# Security (CRITICAL - generate unique values!)
N8N_ENCRYPTION_KEY=<generate-random-32-chars>
N8N_USER_MANAGEMENT_JWT_SECRET=<generate-random-32-chars>

# Optional but recommended
GENERIC_TIMEZONE=America/New_York
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=168
```

### Step 4: Add Disk Storage
```
Settings → Disk → Add Disk
- Mount Path: /home/node/.n8n
- Size: 1GB
```

### Step 5: Deploy
```
Click "Create Web Service" → Wait 5-10 minutes
Access at: https://your-app.onrender.com
```

## 🔐 Generate Encryption Keys

**Linux/Mac:**
```bash
openssl rand -hex 32
```

**PowerShell:**
```powershell
-join ((48..57) + (65..90) + (97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})
```

**Online:**
- https://www.random.org/strings/ (32 characters, alphanumeric)

## 💰 Cost Breakdown

| Setup | Monthly Cost |
|-------|--------------|
| Minimal (Testing) | $14 (Starter + DB) |
| Recommended (Production) | $47.50 (Standard + DB + Storage) |
| Enterprise | $147.50 (Pro + DB + Storage) |

## 🛠️ Common Issues

**Service won't start?**
- Check database connection in logs
- Ensure web service and database in same region
- Use Internal Database URL (not External)

**Webhooks not working?**
- Set `WEBHOOK_URL` with `https://` and trailing `/`
- Upgrade from free tier (free tier services sleep)

**Can't decrypt credentials?**
- `N8N_ENCRYPTION_KEY` changed or missing
- Ensure disk storage is properly mounted at `/home/node/.n8n`

**Slow performance?**
- Upgrade instance to Standard or higher
- Ensure database and service in same region
- Enable execution pruning

## 📚 Full Documentation

For detailed instructions, troubleshooting, and advanced configuration:
👉 [Complete Render.com Deployment Guide](./DEPLOY_TO_RENDER.md)

## 🆘 Need Help?

- 💬 [n8n Community Forum](https://community.n8n.io)
- 📧 [Render Support](https://render.com/support)
- 📖 [n8n Documentation](https://docs.n8n.io)
