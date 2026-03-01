# ChainTorque - DigitalOcean Deployment Guide

## 📋 Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Deployment Options](#deployment-options)
4. [Option 1: DigitalOcean App Platform (Recommended)](#option-1-digitalocean-app-platform-recommended)
5. [Option 2: Droplet + Nginx](#option-2-droplet--nginx)
6. [Environment Variables](#environment-variables)
7. [Database Setup](#database-setup)
8. [Custom Domain Configuration](#custom-domain-configuration)
9. [Troubleshooting](#troubleshooting)

---

## 🏗️ System Architecture Overview

ChainTorque is a **monorepo** containing 4 separate services:

```
ChainTorque/
├── Backend (Express API + Hardhat Smart Contracts)
│   ├── Port: 5001
│   ├── Tech: Express.js, MongoDB, ethers.js, Hardhat
│   ├── Features: REST API, Web3 integration, IPFS storage, AI endpoints
│   └── Build: No build step (Node.js runtime)
│
├── Landing Page (Frontend)
│   ├── Port: 5000
│   ├── Tech: React 18, TypeScript, Vite, Tailwind CSS, GSAP, Clerk Auth
│   ├── Build Output: dist/
│   └── Served: Static files (SPA)
│
├── Marketplace (Frontend)
│   ├── Port: 8080
│   ├── Tech: React 18, TypeScript, Vite, Tailwind CSS, ethers.js, Clerk Auth
│   ├── Build Output: dist/
│   └── Served: Static files (SPA)
│
└── CAD Editor (Frontend)
    ├── Port: 3001
    ├── Tech: React 18, Vite, Three.js, OpenCascade.js (WASM)
    ├── Build Output: build/
    └── Served: Static files (SPA)
```

### Service Dependencies
- **Backend**: Requires MongoDB Atlas, Ethereum RPC, IPFS (Lighthouse)
- **Landing Page**: Requires Clerk Auth
- **Marketplace**: Requires Backend API, Clerk Auth, MetaMask
- **CAD Editor**: Standalone (can load models from Marketplace)

### Inter-Service Communication
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Landing   │────▶│  Clerk Auth  │◀────│ Marketplace │
│   Page      │     │  (Primary)   │     │             │
└─────────────┘     └──────────────┘     └──────┬──────┘
                                                  │
                                                  ▼
┌─────────────┐                          ┌─────────────┐
│ CAD Editor  │◀─────────────────────────│   Backend   │
│ (Optional)  │   Load Models            │   API       │
└─────────────┘                          └──────┬──────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ External Services│
                                         │ - MongoDB Atlas │
                                         │ - Ethereum RPC  │
                                         │ - IPFS/Lighthouse│
                                         └─────────────────┘
```

---

## 🛡️ Prerequisites

Before deploying to DigitalOcean, ensure you have:

### 1. **DigitalOcean Account**
   - Sign up at [digitalocean.com](https://www.digitalocean.com/)
   - Add a payment method
   - Verify your account

### 2. **Required External Services**
   - ✅ **MongoDB Atlas** (Free tier available)
     - Get connection string from [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
   - ✅ **Clerk Auth** (Free tier available)
     - Get publishable key from [clerk.com](https://clerk.com/)
   - ✅ **Lighthouse IPFS** (Free tier available)
     - Get API key from [lighthouse.storage](https://lighthouse.storage/)
   - ✅ **Ethereum RPC Provider** (Free tier available)
     - Options: Ankr, Infura, Alchemy
   - ✅ **Ethereum Wallet** with Sepolia testnet ETH
     - For deploying smart contracts

### 3. **Local Tools**
   - Git installed
   - Node.js 18+ (for local testing)
   - Bun (optional, for faster local development)

### 4. **Code Repository**
   - Push your code to GitHub/GitLab
   - Ensure `.env` is in `.gitignore` (already configured)

---

## 🚀 Deployment Options

### Comparison

| Feature | App Platform | Droplet + Nginx |
|---------|-------------|-----------------|
| **Difficulty** | Easy ⭐ | Advanced ⭐⭐⭐ |
| **Cost** | $5-20/month | $6-24/month |
| **Auto-deploy** | ✅ Yes (Git push) | ❌ Manual |
| **SSL/HTTPS** | ✅ Automatic | ⚙️ Manual (Let's Encrypt) |
| **Scaling** | ✅ Easy | ⚙️ Manual |
| **Best For** | Production | Full control |

**Recommendation**: Use **App Platform** for easier management and automatic deployments.

---

## 🌐 Option 1: DigitalOcean App Platform (Recommended)

DigitalOcean App Platform is a PaaS (Platform as a Service) similar to Render.com/Heroku.

### Step-by-Step Guide

#### 1️⃣ **Prepare Your Repository**

1. **Push your code to GitHub**:
   ```bash
   git add .
   git commit -m "Prepare for DigitalOcean deployment"
   git push origin main
   ```

2. **Create a `.do` folder** (optional, for App Platform spec):
   ```bash
   mkdir .do
   ```

3. **Create App Spec file** `.do/app.yaml`:
   ```yaml
   # Copy the content below after this code block
   ```

#### 2️⃣ **Create App Platform Project**

1. **Go to DigitalOcean Dashboard**:
   - Navigate to [cloud.digitalocean.com](https://cloud.digitalocean.com/)
   - Click **"Apps"** in the left sidebar
   - Click **"Create App"**

2. **Connect GitHub**:
   - Select **"GitHub"**
   - Authorize DigitalOcean
   - Choose your repository
   - Select `main` branch
   - Check **"Autodeploy"** (deploys on every push)

3. **Configure Services**:
   - DigitalOcean will auto-detect your services
   - Or manually add 4 services:

#### 3️⃣ **Configure Backend Service**

**Service Name**: `chaintorque-backend`
- **Type**: Web Service
- **Source**: `/backend`
- **Runtime**: Node.js 20.x
- **Build Command**:
  ```bash
  cd backend && npm install
  ```
- **Run Command**:
  ```bash
  cd backend && npm start
  ```
- **Port**: `8080` (DigitalOcean default, override with `PORT` env var)
- **HTTP Port**: Internal (private to other services)

**Environment Variables** (click "Add Environment Variable"):
```env
NODE_ENV=production
PORT=8080
MONGODB_URI=<your-mongodb-connection-string>
RPC_URL=<your-ethereum-rpc-url>
PRIVATE_KEY=<your-wallet-private-key>
CONTRACT_ADDRESS=<deployed-smart-contract-address>
LIGHTHOUSE_API_KEY=<your-lighthouse-api-key>
CLERK_PUBLISHABLE_KEY=<your-clerk-key>
```

#### 4️⃣ **Configure Landing Page**

**Service Name**: `chaintorque-landing`
- **Type**: Static Site
- **Source**: `/Landing Page (Frontend)`
- **Build Command**:
  ```bash
  cd "Landing Page (Frontend)" && npm install && npm run build
  ```
- **Output Directory**: `Landing Page (Frontend)/dist`
- **Routes**: `/*` → `index.html` (SPA routing)

**Environment Variables**:
```env
VITE_CLERK_PUBLISHABLE_KEY=<your-clerk-key>
VITE_MARKETPLACE_URL=https://<your-marketplace-app>.ondigitalocean.app
```

#### 5️⃣ **Configure Marketplace**

**Service Name**: `chaintorque-marketplace`
- **Type**: Static Site
- **Source**: `/Marketplace (Frontend)`
- **Build Command**:
  ```bash
  cd "Marketplace (Frontend)" && npm install && npm run build
  ```
- **Output Directory**: `Marketplace (Frontend)/dist`
- **Routes**: `/*` → `index.html` (SPA routing)

**Environment Variables**:
```env
VITE_CLERK_PUBLISHABLE_KEY=<your-clerk-key>
VITE_API_URL=https://<your-backend-app>.ondigitalocean.app/api
VITE_CONTRACT_ADDRESS=<your-smart-contract-address>
VITE_LANDING_URL=https://<your-landing-app>.ondigitalocean.app
```

#### 6️⃣ **Configure CAD Editor**

**Service Name**: `chaintorque-cad`
- **Type**: Static Site
- **Source**: `/CAD (Frontend)`
- **Build Command**:
  ```bash
  cd "CAD (Frontend)" && npm install && npm run build
  ```
- **Output Directory**: `CAD (Frontend)/build`
- **Routes**: `/*` → `index.html` (SPA routing)

**Environment Variables**: (None required, fully standalone)

#### 7️⃣ **Deploy Smart Contract**

Before backend starts, deploy your smart contract to Ethereum Sepolia:

```bash
# On your local machine
cd backend
npm install
npx hardhat compile
npx hardhat run scripts/deploy.js --network sepolia
```

**Save the Contract Address** from output and add to Backend env vars!

#### 8️⃣ **Launch App**

1. Click **"Review"** → Verify all 4 services
2. Click **"Create Resources"**
3. Wait 5-10 minutes for build and deployment
4. Get your URLs:
   - Landing: `https://chaintorque-landing-xxxxx.ondigitalocean.app`
   - Marketplace: `https://chaintorque-marketplace-xxxxx.ondigitalocean.app`
   - Backend: `https://chaintorque-backend-xxxxx.ondigitalocean.app`
   - CAD: `https://chaintorque-cad-xxxxx.ondigitalocean.app`

#### 9️⃣ **Update CORS and URLs**

1. **Update Backend CORS** in [server.js](backend/server.js#L30-L50):
   ```javascript
   app.use(cors({
     origin: [
       'https://chaintorque-landing-xxxxx.ondigitalocean.app',
       'https://chaintorque-marketplace-xxxxx.ondigitalocean.app',
       'https://chaintorque-cad-xxxxx.ondigitalocean.app',
       // Keep localhost for local dev
       'http://localhost:8080',
       'http://localhost:5000',
       'http://localhost:3001'
     ]
   }));
   ```

2. **Update Clerk Settings**:
   - Go to [Clerk Dashboard](https://dashboard.clerk.com/)
   - Add your DigitalOcean URLs to **Allowed Origins**
   - Configure **Satellite Mode** if using multiple domains

3. **Push changes**:
   ```bash
   git add .
   git commit -m "Update URLs for DigitalOcean"
   git push
   ```

4. Auto-deploy will trigger within 1-2 minutes

---

## 🖥️ Option 2: Droplet + Nginx

For advanced users who want full control. This requires SSH, Linux knowledge, and manual configuration.

### Prerequisites
- Understanding of Linux commands
- SSH client
- Domain name (optional but recommended)

### Step-by-Step Guide

#### 1️⃣ **Create Droplet**

1. Go to [DigitalOcean Dashboard](https://cloud.digitalocean.com/)
2. Click **"Droplets"** → **"Create Droplet"**
3. **Configuration**:
   - **OS**: Ubuntu 22.04 LTS
   - **Plan**: Basic ($6/month - 1GB RAM)
   - **Datacenter**: Choose closest to your users
   - **Authentication**: SSH Key (more secure) or Password
4. Click **"Create Droplet"**

#### 2️⃣ **SSH into Droplet**

```bash
ssh root@<your-droplet-ip>
```

#### 3️⃣ **Install Dependencies**

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Install Nginx
apt install -y nginx

# Install PM2 (process manager)
npm install -g pm2

# Install Git
apt install -y git

# Install Bun (optional, for faster builds)
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc
```

#### 4️⃣ **Clone Repository**

```bash
cd /var/www
git clone https://github.com/<your-username>/ChainTorque.git
cd ChainTorque
```

#### 5️⃣ **Set Up Environment Variables**

```bash
# Create .env file
nano .env
```

Paste your environment variables:
```env
NODE_ENV=production
PORT=5001
MONGODB_URI=<your-mongodb-uri>
RPC_URL=<your-rpc-url>
PRIVATE_KEY=<your-private-key>
CONTRACT_ADDRESS=<your-contract-address>
LIGHTHOUSE_API_KEY=<your-lighthouse-key>
CLERK_PUBLISHABLE_KEY=<your-clerk-key>
VITE_CLERK_PUBLISHABLE_KEY=<your-clerk-key>
VITE_API_URL=http://localhost:5001/api
```

Press `CTRL+X`, then `Y`, then `Enter` to save.

#### 6️⃣ **Build Frontend Apps**

```bash
# Install dependencies
npm install --legacy-peer-deps

# Build Landing Page
cd "Landing Page (Frontend)"
npm run build
cd ..

# Build Marketplace
cd "Marketplace (Frontend)"
npm run build
cd ..

# Build CAD Editor  
cd "CAD (Frontend)"
npm run build
cd ..
```

#### 7️⃣ **Start Backend with PM2**

```bash
cd backend
npm install
pm2 start server.js --name chaintorque-backend
pm2 save
pm2 startup
```

#### 8️⃣ **Configure Nginx**

```bash
nano /etc/nginx/sites-available/chaintorque
```

Paste the following configuration:

```nginx
# Backend API
server {
    listen 80;
    server_name api.yourdomain.com;  # Replace with your domain or IP

    location / {
        proxy_pass http://localhost:5001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Landing Page
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;  # Replace with your domain

    root /var/www/ChainTorque/Landing Page (Frontend)/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

# Marketplace
server {
    listen 80;
    server_name marketplace.yourdomain.com;  # Replace with your subdomain

    root /var/www/ChainTorque/Marketplace (Frontend)/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

# CAD Editor
server {
    listen 80;
    server_name cad.yourdomain.com;  # Replace with your subdomain

    root /var/www/ChainTorque/CAD (Frontend)/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Save and exit (`CTRL+X`, `Y`, `Enter`).

#### 9️⃣ **Enable Site and Restart Nginx**

```bash
ln -s /etc/nginx/sites-available/chaintorque /etc/nginx/sites-enabled/
nginx -t  # Test configuration
systemctl restart nginx
```

#### 🔟 **Set Up SSL with Let's Encrypt**

```bash
# Install Certbot
apt install -y certbot python3-certbot-nginx

# Get SSL certificates (replace with YOUR domains)
certbot --nginx -d yourdomain.com -d www.yourdomain.com
certbot --nginx -d marketplace.yourdomain.com
certbot --nginx -d cad.yourdomain.com
certbot --nginx -d api.yourdomain.com

# Auto-renewal
certbot renew --dry-run
```

#### 1️⃣1️⃣ **Configure Firewall**

```bash
ufw allow 'Nginx Full'
ufw allow OpenSSH
ufw enable
```

#### 1️⃣2️⃣ **Test Your Deployment**

Visit your domains:
- Landing: `https://yourdomain.com`
- Marketplace: `https://marketplace.yourdomain.com`
- CAD: `https://cad.yourdomain.com`
- API Health: `https://api.yourdomain.com/health`

---

## 🔐 Environment Variables

### Backend Environment Variables
```env
# REQUIRED
NODE_ENV=production
PORT=8080
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/chaintorque
RPC_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
PRIVATE_KEY=0xYOUR_WALLET_PRIVATE_KEY
CONTRACT_ADDRESS=0xYOUR_DEPLOYED_CONTRACT_ADDRESS
LIGHTHOUSE_API_KEY=your_lighthouse_api_key
CLERK_PUBLISHABLE_KEY=pk_test_xxxxx

# OPTIONAL
NODE_NO_WARNINGS=1
GROQ_API_KEY=gsk_xxxx  # For AI features
HF_TOKEN=hf_xxxx  # For Hugging Face models
```

### Frontend Environment Variables

**Landing Page**:
```env
VITE_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
VITE_MARKETPLACE_URL=https://marketplace.yourdomain.com
```

**Marketplace**:
```env
VITE_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
VITE_API_URL=https://api.yourdomain.com/api
VITE_CONTRACT_ADDRESS=0xYOUR_CONTRACT_ADDRESS
VITE_LANDING_URL=https://yourdomain.com
```

**CAD Editor**:
```env
# No environment variables required
# Fully standalone application
```

---

## 💾 Database Setup

### MongoDB Atlas (Recommended)

1. **Create Account**: [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)

2. **Create Cluster**:
   - Free tier (M0) is sufficient for testing
   - Choose region close to your DigitalOcean datacenter

3. **Database Access**:
   - Create database user
   - Save username and password

4. **Network Access**:
   - Click "Add IP Address"
   - Choose **"Allow Access from Anywhere"** (`0.0.0.0/0`)
   - Or add your DigitalOcean App IP addresses

5. **Get Connection String**:
   - Click "Connect" → "Connect your application"
   - Copy connection string
   - Replace `<password>` with your password
   - Add database name: `/chaintorque?retryWrites=true&w=majority`

Example:
```
mongodb+srv://username:password@cluster.mongodb.net/chaintorque?retryWrites=true&w=majority
```

### Collections Created Automatically

The backend creates these collections:
- `marketitems` - NFT listings
- `transactions` - Purchase history
- `users` - User profiles

---

## 🌍 Custom Domain Configuration

### DigitalOcean App Platform

1. **Buy Domain** (from Namecheap, GoDaddy, etc.)

2. **Add Domain to DigitalOcean**:
   - Go to your App → "Settings" → "Domains"
   - Click "Add Domain"
   - Enter your domain: `yourdomain.com`
   - DigitalOcean will provide DNS records

3. **Update DNS** (at your domain registrar):
   ```
   Type: CNAME
   Host: www
   Value: <your-app>.ondigitalocean.app
   
   Type: A
   Host: @
   Value: <DigitalOcean IP>
   ```

4. **Add Subdomains** (repeat for each service):
   - `marketplace.yourdomain.com` → Marketplace App
   - `cad.yourdomain.com` → CAD App
   - `api.yourdomain.com` → Backend App

5. **SSL Certificate**: Auto-provisioned by DigitalOcean

### Droplet with Nginx

Already covered in [Option 2 Step 10](#option-2-droplet--nginx) above.

---

## 🚨 Troubleshooting

### Issue: Build Fails with "Cannot find workspace"

**Solution**: Update build commands to include workspace name:
```bash
npm run --workspace=chaintorque-backend start
```

### Issue: Frontend shows "Cannot connect to API"

**Cause**: CORS or incorrect API URL

**Solution**:
1. Check `VITE_API_URL` in Marketplace env vars
2. Verify Backend CORS includes your frontend domain
3. Test API directly: `https://api.yourdomain.com/health`

### Issue: "MetaMask Not Connected"

**Cause**: Smart contract not deployed or wrong address

**Solution**:
1. Deploy contract: `npx hardhat run scripts/deploy.js --network sepolia`
2. Copy address from output
3. Update `CONTRACT_ADDRESS` in backend env
4. Update `VITE_CONTRACT_ADDRESS` in marketplace env

### Issue: MongoDB Connection Failed

**Cause**: IP whitelist or wrong connection string

**Solution**:
1. Go to MongoDB Atlas → Network Access
2. Add `0.0.0.0/0` (or specific DigitalOcean IPs)
3. Verify connection string format
4. Test connection: `mongosh "your-connection-string"`

### Issue: Images/Models Not Loading

**Cause**: IPFS/Lighthouse API key issue

**Solution**:
1. Verify `LIGHTHOUSE_API_KEY` in backend env
2. Check Lighthouse dashboard for quota
3. Try re-uploading with: `/api/marketplace/upload-files`

### Issue: SSL Certificate Error

**Cause**: Certificate not provisioned yet

**Solution**:
- **App Platform**: Wait 5-10 minutes after adding domain
- **Droplet**: Run `certbot renew` manually

### Issue: PM2 Process Crashes

**Solution**:
```bash
# Check logs
pm2 logs chaintorque-backend

# Restart
pm2 restart chaintorque-backend

# Check status
pm2 status
```

### Issue: Clerk Auth Not Working

**Cause**: Domain not added to Clerk allowlist

**Solution**:
1. Go to [Clerk Dashboard](https://dashboard.clerk.com/)
2. Go to your application
3. Add your DigitalOcean URLs to:
   - **Home URL**
   - **Allowed callback URLs**
   - **Allowed logout redirect URLs**

---

## 📊 Monitoring & Maintenance

### DigitalOcean App Platform

- **Logs**: Click your service → "Console" → "Runtime Logs"
- **Metrics**: View CPU, Memory, Bandwidth usage in Dashboard
- **Alerts**: Set up alerts for downtime

### Droplet

```bash
# View backend logs
pm2 logs chaintorque-backend

# View Nginx logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Monitor resources
htop  # Install with: apt install htop
```

---

## 💰 Pricing Estimate

### App Platform (4 Services)
- **Backend**: $5/month (Basic)
- **Landing**: $3/month (Static)
- **Marketplace**: $3/month (Static)
- **CAD**: $3/month (Static)
- **Total**: ~$14/month + bandwidth

### Droplet
- **Droplet**: $6-12/month (1-2GB RAM)
- **No additional charges** for bandwidth (1TB included)

### External Services
- **MongoDB Atlas**: Free (M0 cluster)
- **Clerk Auth**: Free (up to 10,000 users)
- **Lighthouse IPFS**: Free tier available
- **Ethereum RPC**: Free (Ankr, Infura free tiers)

**Total Monthly Cost**: $14-20/month

---

## 🎉 Deployment Complete!

Your ChainTorque platform should now be live on DigitalOcean!

### Next Steps

1. ✅ Test all features:
   - User signup/login
   - NFT listing/purchase
   - 3D model preview
   - CAD editor tools

2. ✅ Configure custom domains

3. ✅ Set up monitoring/alerts

4. ✅ Enable backups:
   - MongoDB Atlas: Automatic backups enabled by default
   - DigitalOcean Droplet: Enable weekly backups ($1.20/month)

5. ✅ Optimize performance:
   - Enable CDN for static assets
   - Configure caching headers
   - Compress images

### Support Resources

- [DigitalOcean Documentation](https://docs.digitalocean.com/)
- [DigitalOcean Community](https://www.digitalocean.com/community)
- [ChainTorque Documentation](./DOCUMENTATION.md)
- [Report Issues](https://github.com/your-repo/issues)

---

**Deployed by**: Your Name  
**Date**: March 2026  
**Version**: 1.0.0
