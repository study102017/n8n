# Deploy n8n to Render.com - Complete Guide

This guide will walk you through deploying n8n (workflow automation platform) to Render.com step by step, acting as your Render.com customer support supervisor.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Prepare Your Render Account](#step-1-prepare-your-render-account)
- [Step 2: Create PostgreSQL Database](#step-2-create-postgresql-database)
- [Step 3: Deploy n8n Web Service](#step-3-deploy-n8n-web-service)
- [Step 4: Configure Environment Variables](#step-4-configure-environment-variables)
- [Step 5: Set Up Persistent Storage](#step-5-set-up-persistent-storage)
- [Step 6: Configure Domain and SSL](#step-6-configure-domain-and-ssl)
- [Step 7: Verify Deployment](#step-7-verify-deployment)
- [Troubleshooting](#troubleshooting)
- [Maintenance and Updates](#maintenance-and-updates)
- [Cost Estimation](#cost-estimation)

---

## Prerequisites

Before you begin, ensure you have:

1. **A Render.com account** - Sign up at [render.com](https://render.com) if you don't have one
2. **A GitHub account** (optional, but recommended for automatic deployments)
3. **Basic understanding of environment variables** and Docker
4. **Credit card** (required for paid services, though you can start with free tier for testing)

---

## Step 1: Prepare Your Render Account

### 1.1 Sign Up or Log In
1. Go to [https://render.com](https://render.com)
2. Click **"Get Started"** or **"Sign In"**
3. You can sign up using:
   - GitHub (recommended for easy deployments)
   - GitLab
   - Google
   - Email

### 1.2 Verify Your Account
1. Complete email verification if signing up for the first time
2. Add a payment method in **Account Settings** → **Billing** (required for databases and persistent storage)

---

## Step 2: Create PostgreSQL Database

n8n requires a database to store workflows, credentials, and execution history. We'll use PostgreSQL.

### 2.1 Create Database
1. From your Render dashboard, click **"New +"** in the top right
2. Select **"PostgreSQL"**
3. Fill in the database details:
   - **Name**: `n8n-database` (or your preferred name)
   - **Database**: `n8n` (database name)
   - **User**: `n8n_user` (will be auto-created)
   - **Region**: Choose the region closest to your users (e.g., `Oregon (US West)`, `Frankfurt (EU Central)`)
   - **PostgreSQL Version**: Select the latest stable version (16.x recommended)
   - **Plan**: Start with **"Starter"** ($7/month) or **"Free"** for testing

4. Click **"Create Database"**

### 2.2 Save Database Credentials
1. Once the database is created, go to the **"Info"** tab
2. **Save these important values** - you'll need them later:
   - **Internal Database URL** (for connecting from Render services)
   - **External Database URL** (for external connections)
   - **Hostname**
   - **Port**
   - **Database**
   - **Username**
   - **Password**

> ⚠️ **Important**: Keep these credentials secure. You'll use the **Internal Database URL** for your n8n service.

### 2.3 Wait for Database Availability
The database typically takes 2-3 minutes to become available. Wait until the status shows **"Available"** (green indicator).

---

## Step 3: Deploy n8n Web Service

Now we'll deploy the n8n application itself.

### 3.1 Create Web Service
1. From your Render dashboard, click **"New +"**
2. Select **"Web Service"**

### 3.2 Connect Repository (Option A - Recommended)
If you want automatic deployments when the n8n repository updates:

1. Select **"Deploy an existing image from a registry"**
2. Use the official n8n Docker image: `docker.n8n.io/n8nio/n8n`
3. Click **"Continue"**

### 3.3 Manual Docker Deploy (Option B)
If you prefer manual deployments:

1. Select **"Deploy an existing image from a registry"**
2. Image URL: `docker.n8n.io/n8nio/n8n:latest`
3. Click **"Continue"**

### 3.4 Configure Service Settings
Fill in the following details:

- **Name**: `n8n-app` (or your preferred name - this will be part of your URL)
- **Region**: **MUST match your database region** for optimal performance
- **Instance Type**: 
  - **Starter**: $7/month (512MB RAM, 0.5 CPU) - **Recommended minimum**
  - **Standard**: $25/month (2GB RAM, 1 CPU) - For production use
  - Free tier is **NOT recommended** as n8n requires more resources

---

## Step 4: Configure Environment Variables

This is the most critical step. Environment variables configure how n8n connects to your database and operates.

### 4.1 Add Required Environment Variables

In the **"Environment Variables"** section, click **"Add Environment Variable"** and add each of the following:

#### Database Configuration

| Variable Name | Value | Description |
|--------------|-------|-------------|
| `DB_TYPE` | `postgresdb` | Tells n8n to use PostgreSQL |
| `DB_POSTGRESDB_DATABASE` | `n8n` | Your database name from Step 2 |
| `DB_POSTGRESDB_HOST` | `<your-db-hostname>` | From the Internal Database URL |
| `DB_POSTGRESDB_PORT` | `5432` | Default PostgreSQL port |
| `DB_POSTGRESDB_USER` | `<your-db-username>` | From database credentials |
| `DB_POSTGRESDB_PASSWORD` | `<your-db-password>` | From database credentials |
| `DB_POSTGRESDB_SCHEMA` | `public` | Database schema name |

> 💡 **Tip**: Instead of setting these individually, you can use the **Internal Database URL** directly:

| Variable Name | Value |
|--------------|-------|
| `DB_POSTGRESDB_CONNECTION_URL` | `<your-internal-database-url>` |

#### n8n Core Configuration

| Variable Name | Value | Description |
|--------------|-------|-------------|
| `N8N_HOST` | `<your-render-url>.onrender.com` | Your Render service URL (e.g., `n8n-app.onrender.com`) |
| `WEBHOOK_URL` | `https://<your-render-url>.onrender.com/` | Same as above but with https:// and trailing / |
| `N8N_PORT` | `5678` | Default n8n port (usually detected automatically) |
| `N8N_PROTOCOL` | `https` | Use HTTPS (Render provides SSL) |
| `NODE_ENV` | `production` | Run in production mode |
| `GENERIC_TIMEZONE` | `America/New_York` | Your timezone (e.g., `Europe/Berlin`, `Asia/Tokyo`) |

#### Security & Performance

| Variable Name | Value | Description |
|--------------|-------|-------------|
| `N8N_ENCRYPTION_KEY` | `<generate-random-string>` | **CRITICAL**: Generate a random 32+ character string |
| `N8N_USER_MANAGEMENT_JWT_SECRET` | `<generate-random-string>` | JWT secret for user authentication |
| `EXECUTIONS_DATA_PRUNE` | `true` | Automatically clean old execution data |
| `EXECUTIONS_DATA_MAX_AGE` | `168` | Keep executions for 7 days (168 hours) |

> 🔐 **Security Note**: Generate secure random strings for encryption keys:
> ```bash
> # On Linux/Mac, generate with:
> openssl rand -hex 32
> # Or use an online generator: https://www.random.org/strings/
> ```

#### Optional: Email Configuration

If you want to enable email notifications:

| Variable Name | Example Value | Description |
|--------------|---------------|-------------|
| `N8N_EMAIL_MODE` | `smtp` | Enable SMTP email |
| `N8N_SMTP_HOST` | `smtp.gmail.com` | Your SMTP server |
| `N8N_SMTP_PORT` | `587` | SMTP port |
| `N8N_SMTP_USER` | `your-email@gmail.com` | SMTP username |
| `N8N_SMTP_PASS` | `your-app-password` | SMTP password |
| `N8N_SMTP_SENDER` | `n8n@yourdomain.com` | Sender email address |

### 4.2 Review Configuration
Double-check all environment variables before proceeding. Incorrect database credentials are the most common issue.

---

## Step 5: Set Up Persistent Storage

n8n needs persistent storage for files and temporary data.

### 5.1 Create Disk Storage
1. Scroll down to **"Disk"** section in your web service settings
2. Click **"Add Disk"**
3. Configure:
   - **Name**: `n8n-data`
   - **Mount Path**: `/home/node/.n8n`
   - **Size**: Start with **1GB** (can be increased later)

4. Click **"Create Disk"**

> 📁 **Important**: The mount path `/home/node/.n8n` is where n8n stores:
> - Encryption keys (persisted)
> - User uploads
> - Temporary workflow data
> - SSL certificates (if any)

---

## Step 6: Configure Domain and SSL

### 6.1 Use Render's Default Domain
By default, your n8n instance will be available at:
```
https://n8n-app.onrender.com
```
(Replace `n8n-app` with your service name)

Render automatically provides:
- ✅ **Free SSL certificate** (Let's Encrypt)
- ✅ **HTTPS by default**
- ✅ **Automatic certificate renewal**

### 6.2 Use Custom Domain (Optional)

If you want to use your own domain (e.g., `n8n.yourdomain.com`):

1. Go to your web service in Render
2. Click on the **"Settings"** tab
3. Scroll to **"Custom Domain"**
4. Click **"Add Custom Domain"**
5. Enter your domain: `n8n.yourdomain.com`
6. Render will provide DNS records:
   - **CNAME record**: Point to your Render service
   
7. Add this CNAME record in your domain's DNS settings:
   ```
   n8n.yourdomain.com  CNAME  n8n-app.onrender.com
   ```

8. Wait for DNS propagation (can take 5-60 minutes)
9. Render will automatically issue an SSL certificate

> 🌐 **Update Environment Variables**: After adding a custom domain, update:
> - `N8N_HOST` → `n8n.yourdomain.com`
> - `WEBHOOK_URL` → `https://n8n.yourdomain.com/`

---

## Step 7: Verify Deployment

### 7.1 Deploy the Service
1. At the bottom of your web service configuration, click **"Create Web Service"**
2. Render will start building and deploying your n8n instance
3. Monitor the **"Logs"** tab to watch the deployment progress

### 7.2 Wait for Deployment
- Initial deployment takes **5-10 minutes**
- Watch for these log messages indicating success:
  ```
  n8n ready on 0.0.0.0:5678
  Editor is now accessible via:
  https://n8n-app.onrender.com
  ```

### 7.3 Access n8n
1. Once deployment shows **"Live"** (green status):
2. Open your browser and go to: `https://your-service-name.onrender.com`
3. You should see the n8n **setup wizard**

### 7.4 Complete n8n Setup
1. **Create your first user account**:
   - Enter your email
   - Create a strong password
   - Set your first and last name

2. **You're done!** You now have a working n8n instance on Render.com

---

## Troubleshooting

### Issue 1: "Service Unavailable" or Constant Restarting

**Symptoms**: Service keeps restarting or shows 503 error

**Solutions**:
1. Check the **Logs** tab for error messages
2. Common causes:
   - **Database connection failed**: Verify `DB_POSTGRESDB_*` variables
   - **Out of memory**: Upgrade to a larger instance type
   - **Wrong database hostname**: Use the **Internal Database URL** (not External)

**Fix**:
```bash
# Verify database connection from logs
# Look for: "Database connection established" or "Connection refused"
```

### Issue 2: Webhooks Not Working

**Symptoms**: Workflows with webhooks don't trigger

**Solutions**:
1. Verify `WEBHOOK_URL` is set correctly:
   ```
   WEBHOOK_URL=https://your-service.onrender.com/
   ```
   Note the trailing `/`

2. Ensure `N8N_PROTOCOL` is set to `https`

3. Check if your Render service is not sleeping (upgrade from free tier)

### Issue 3: "Encryption Key Missing" Error

**Symptoms**: Can't decrypt existing credentials

**Solutions**:
1. This happens when `N8N_ENCRYPTION_KEY` changes or disk storage is lost
2. **Prevention**: Always set `N8N_ENCRYPTION_KEY` explicitly in environment variables
3. **If already happened**: You'll need to re-enter all credentials

### Issue 4: Slow Performance

**Solutions**:
1. Upgrade your instance type to **Standard** or higher
2. Ensure database and web service are in the **same region**
3. Enable execution data pruning:
   ```
   EXECUTIONS_DATA_PRUNE=true
   EXECUTIONS_DATA_MAX_AGE=168
   ```

### Issue 5: Database Connection Issues

**Symptoms**: "Connection refused" or "authentication failed"

**Solutions**:
1. Use the **Internal Database URL** instead of External
2. Verify database is in **Available** state
3. Check that web service and database are in the **same region**
4. Whitelist doesn't apply to internal connections - no need to add IPs

### Issue 6: Free Tier Sleeping

**Symptoms**: Service takes 30+ seconds to respond after inactivity

**Cause**: Free tier services sleep after 15 minutes of inactivity

**Solutions**:
1. Upgrade to **Starter** ($7/month) or higher - they never sleep
2. Use an external uptime monitor to ping your service every 10 minutes (temporary workaround)

---

## Maintenance and Updates

### Updating n8n to Latest Version

Since you're using the Docker image `docker.n8n.io/n8nio/n8n:latest`, updates are automatic:

1. Go to your web service in Render
2. Click **"Manual Deploy"** → **"Deploy latest commit"**
3. Render will pull the latest Docker image and redeploy

### Manual Version Control

To use a specific version:

1. Change the image to include version tag:
   ```
   docker.n8n.io/n8nio/n8n:1.115.0
   ```
2. Update in **Settings** → **"Image URL"**
3. Save and deploy

### Monitoring

1. Enable **Health Checks** in Render:
   - Path: `/healthz`
   - This endpoint returns n8n's health status

2. Set up **Alerts** in Render dashboard for:
   - Service downtime
   - High CPU/memory usage
   - Failed deployments

### Backups

**Database Backups**:
- Render automatically backs up PostgreSQL databases daily (on paid plans)
- Access backups from **Database** → **"Backups"** tab
- Download backups regularly for extra safety

**Workflow Backups**:
- Export workflows regularly from n8n UI
- **Settings** → **"Workflows"** → Export all workflows
- Store exports in version control (GitHub, GitLab)

---

## Cost Estimation

Here's a breakdown of costs for running n8n on Render:

### Minimal Production Setup
| Service | Plan | Monthly Cost |
|---------|------|--------------|
| Web Service | Starter (512MB RAM) | $7 |
| PostgreSQL | Starter (256MB RAM) | $7 |
| Disk Storage | 1GB | Included |
| **Total** | | **$14/month** |

### Recommended Production Setup
| Service | Plan | Monthly Cost |
|---------|------|--------------|
| Web Service | Standard (2GB RAM) | $25 |
| PostgreSQL | Standard (1GB RAM) | $20 |
| Disk Storage | 10GB | $0.25/GB = $2.50 |
| **Total** | | **$47.50/month** |

### Enterprise Setup
| Service | Plan | Monthly Cost |
|---------|------|--------------|
| Web Service | Pro (4GB RAM) | $85 |
| PostgreSQL | Standard Plus (4GB RAM) | $50 |
| Disk Storage | 50GB | $12.50 |
| **Total** | | **$147.50/month** |

> 💡 **Free Testing**: You can test with free tiers, but they have limitations:
> - Services sleep after 15 minutes of inactivity
> - Limited resources (512MB RAM)
> - Not suitable for production webhooks

---

## Additional Configuration Options

### Enable Basic Authentication (Optional)

Add extra security layer before reaching n8n:

```
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=your-username
N8N_BASIC_AUTH_PASSWORD=your-password
```

### Configure Log Level

For debugging:

```
N8N_LOG_LEVEL=debug
N8N_LOG_OUTPUT=console
```

### Enable Metrics (Advanced)

```
N8N_METRICS=true
N8N_METRICS_INCLUDE_API_ENDPOINTS=true
```

### Queue Mode (For High Load)

For production with many concurrent workflows:

```
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=<redis-host>
QUEUE_BULL_REDIS_PORT=6379
```

(Requires setting up Redis - add another service on Render)

---

## Support and Resources

### Official n8n Resources
- 📚 [n8n Documentation](https://docs.n8n.io)
- 💬 [Community Forum](https://community.n8n.io)
- 🐛 [GitHub Issues](https://github.com/n8n-io/n8n/issues)
- 📺 [YouTube Tutorials](https://www.youtube.com/c/n8n-io)

### Render.com Resources
- 📘 [Render Documentation](https://render.com/docs)
- 💬 [Render Community](https://community.render.com)
- 🆘 [Render Support](https://render.com/support)

### Getting Help
1. Check the [n8n Community Forum](https://community.n8n.io) first
2. For Render-specific issues, contact [Render Support](https://render.com/support)
3. For n8n bugs, report on [GitHub](https://github.com/n8n-io/n8n/issues)

---

## Security Best Practices

1. ✅ **Use strong passwords** for your n8n account
2. ✅ **Set a unique `N8N_ENCRYPTION_KEY`** - never use the default
3. ✅ **Enable HTTPS** (automatic on Render)
4. ✅ **Regularly update n8n** to get security patches
5. ✅ **Use environment variables** for sensitive data, never hardcode
6. ✅ **Enable database backups** (included in paid plans)
7. ✅ **Restrict database access** to internal network only (default on Render)
8. ✅ **Use strong PostgreSQL password** (auto-generated by Render)
9. ✅ **Consider IP whitelisting** if handling sensitive data
10. ✅ **Review n8n user permissions** regularly

---

## Conclusion

Congratulations! 🎉 You've successfully deployed n8n to Render.com. You now have:

- ✅ A fully functional n8n instance
- ✅ PostgreSQL database for workflow storage
- ✅ HTTPS with automatic SSL certificates
- ✅ Persistent storage for your data
- ✅ A production-ready automation platform

### Next Steps
1. **Create your first workflow** - Try the examples in n8n
2. **Connect integrations** - Link your favorite apps and services
3. **Set up notifications** - Configure email or Slack alerts
4. **Invite team members** - Add collaborators to your instance
5. **Explore templates** - Browse [n8n.io/workflows](https://n8n.io/workflows) for inspiration

### Need Help?
If you run into issues or have questions:
- 💬 Join the [n8n Community](https://community.n8n.io)
- 📧 Contact Render Support through your dashboard
- 📖 Review this guide's [Troubleshooting](#troubleshooting) section

Happy automating! 🚀
