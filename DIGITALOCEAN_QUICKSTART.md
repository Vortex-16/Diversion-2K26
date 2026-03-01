# DigitalOcean Quick Start Guide

> **🚀 Deploy ChainTorque to DigitalOcean in under 30 minutes!**

---

## 📚 Documentation Index

This project now includes comprehensive documentation for DigitalOcean deployment:

### 1. [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md)
**Read this first** to understand how the entire system works:
- Complete technical overview
- Service-by-service breakdown
- Data flow diagrams
- Database schema
- Smart contract architecture
- API endpoints reference

### 2. [DIGITALOCEAN_DEPLOYMENT.md](./DIGITALOCEAN_DEPLOYMENT.md)
**Your deployment bible** with step-by-step instructions:
- Two deployment options (App Platform vs Droplet)
- Complete environment variable guide
- Database setup instructions
- Custom domain configuration
- Troubleshooting section
- Cost estimates

### 3. [.do/app.yaml](./.do/app.yaml)
**Ready-to-use App Platform spec file**:
- Pre-configured for all 4 services
- Just replace placeholder values with your keys
- One-click deployment support

---

## ⚡ Quick Start (5 minutes)

### Prerequisites
Before you start, gather these:

1. **DigitalOcean Account** → [Sign up](https://www.digitalocean.com/)
2. **MongoDB Atlas** (Free) → [Get URI](https://www.mongodb.com/cloud/atlas)
3. **Clerk Auth** (Free) → [Get key](https://clerk.com/)
4. **Lighthouse IPFS** (Free) → [Get API key](https://lighthouse.storage/)
5. **Ethereum RPC** (Free) → [Ankr](https://www.ankr.com/) or [Alchemy](https://www.alchemy.com/)
6. **Wallet Private Key** → For smart contract deployment

### Step 1: Prepare Your Code

```bash
# Push to GitHub
git add .
git commit -m "Prepare for DigitalOcean deployment"
git push origin main
```

### Step 2: Deploy Smart Contract

```bash
cd backend
npm install
npx hardhat compile
npx hardhat run scripts/deploy.js --network sepolia
```

**Save the contract address!** You'll need it in Step 4.

### Step 3: Create DigitalOcean App

1. Go to [DigitalOcean Apps](https://cloud.digitalocean.com/apps)
2. Click **"Create App"**
3. Connect your GitHub repository
4. Choose `main` branch
5. DigitalOcean will detect 4 services automatically

### Step 4: Configure Environment Variables

For **Backend** service:
```env
NODE_ENV=production
PORT=8080
MONGODB_URI=<your-mongodb-uri>
RPC_URL=<your-ethereum-rpc>
PRIVATE_KEY=<your-wallet-private-key>
CONTRACT_ADDRESS=<your-contract-address>
LIGHTHOUSE_API_KEY=<your-lighthouse-key>
CLERK_PUBLISHABLE_KEY=<your-clerk-key>
```

For **Landing Page**:
```env
VITE_CLERK_PUBLISHABLE_KEY=<your-clerk-key>
```

For **Marketplace**:
```env
VITE_CLERK_PUBLISHABLE_KEY=<your-clerk-key>
VITE_API_URL=https://<your-backend-url>/api
VITE_CONTRACT_ADDRESS=<your-contract-address>
```

For **CAD Editor**: No environment variables needed.

### Step 5: Deploy!

1. Click **"Create Resources"**
2. Wait 5-10 minutes for build
3. Get your URLs:
   - `https://chaintorque-landing-xxxxx.ondigitalocean.app`
   - `https://chaintorque-marketplace-xxxxx.ondigitalocean.app`
   - `https://chaintorque-cad-xxxxx.ondigitalocean.app`
   - `https://chaintorque-backend-xxxxx.ondigitalocean.app`

### Step 6: Update CORS

In [backend/server.js](backend/server.js), update the CORS origins with your DigitalOcean URLs:

```javascript
app.use(cors({
  origin: [
    'https://chaintorque-landing-xxxxx.ondigitalocean.app',
    'https://chaintorque-marketplace-xxxxx.ondigitalocean.app',
    'https://chaintorque-cad-xxxxx.ondigitalocean.app',
    // Keep localhost for dev
    'http://localhost:8080',
    'http://localhost:5000',
    'http://localhost:3001'
  ]
}));
```

Commit and push - auto-deploy will trigger!

### Step 7: Update Clerk Settings

1. Go to [Clerk Dashboard](https://dashboard.clerk.com/)
2. Add your DigitalOcean URLs to:
   - **Allowed Origins**
   - **Redirect URLs**

### Step 8: Test Your Deployment! 🎉

Visit your landing page and try:
- ✅ Sign in with Google/MetaMask
- ✅ Browse marketplace
- ✅ Open CAD editor
- ✅ Upload a 3D model

---

## 🏗️ System Overview

ChainTorque is a **monorepo** with 4 independent services:

```
┌─────────────────────────────────────────────────────────┐
│                  1. Landing Page                        │
│  Marketing site with Clerk authentication               │
│  Tech: React + TypeScript + Vite + Tailwind            │
│  Port: 5000                                             │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  2. Marketplace                         │
│  NFT marketplace with Web3 integration                  │
│  Tech: React + TypeScript + Vite + ethers.js           │
│  Port: 8080                                             │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  3. CAD Editor                          │
│  Browser-based 3D modeling tool                         │
│  Tech: React + Vite + Three.js + OpenCascade.js        │
│  Port: 3001                                             │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  4. Backend API                         │
│  Express API + Smart Contracts + IPFS                   │
│  Tech: Express + MongoDB + Hardhat + ethers.js         │
│  Port: 5001                                             │
└─────────────────────────────────────────────────────────┘
```

**How they communicate**:
- Landing Page → Clerk Auth → Marketplace
- Marketplace → Backend API → MongoDB & Blockchain
- CAD Editor → Standalone (can load from Marketplace)

---

## 🔐 Environment Variables Explained

### What is an Environment Variable?
A secret value that your app needs but shouldn't be in your code (like passwords).

### Where to Set Them?
- **DigitalOcean App Platform**: In each service's settings
- **Local Development**: In `.env` file in root directory

### Critical Variables:

| Variable | Purpose | Where to Get |
|----------|---------|--------------|
| `MONGODB_URI` | Database connection | MongoDB Atlas dashboard |
| `CLERK_PUBLISHABLE_KEY` | Authentication | Clerk dashboard |
| `CONTRACT_ADDRESS` | Smart contract | After deploying contract |
| `LIGHTHOUSE_API_KEY` | IPFS storage | Lighthouse website |
| `RPC_URL` | Blockchain connection | Ankr/Alchemy/Infura |
| `PRIVATE_KEY` | Wallet for backend | MetaMask (export key) |

⚠️ **Never commit your `.env` file to Git!** (Already in `.gitignore`)

---

## 💡 Using DigitalOcean

### What is DigitalOcean?
A cloud hosting platform (like AWS, Google Cloud, but simpler and cheaper).

### Two Deployment Options:

#### Option 1: App Platform (Recommended) ⭐
**What it is**: Like Heroku/Render - automatic deployments from GitHub

**Pros**:
- ✅ Easy setup (click & configure)
- ✅ Auto-deploy on git push
- ✅ Automatic SSL/HTTPS
- ✅ No server management

**Cons**:
- ❌ Slightly more expensive ($14-20/month)
- ❌ Less control

**Best for**: Beginners, production apps

#### Option 2: Droplet + Nginx (Advanced) 🛠️
**What it is**: Your own Linux server (VPS)

**Pros**:
- ✅ Full control
- ✅ Cheaper ($6-12/month)
- ✅ Can run anything

**Cons**:
- ❌ Requires Linux knowledge
- ❌ Manual deployment
- ❌ Must configure SSL yourself

**Best for**: Advanced users, learning DevOps

---

## 📊 Cost Breakdown

### DigitalOcean App Platform
- Backend: $5/month
- Landing Page: $3/month
- Marketplace: $3/month
- CAD Editor: $3/month
- **Total: ~$14/month**

### Droplet
- Server: $6-12/month (1-2GB RAM)
- **Total: ~$6-12/month**

### External Services (All Free Tiers):
- ✅ MongoDB Atlas: Free (512MB storage)
- ✅ Clerk Auth: Free (10,000 users)
- ✅ Lighthouse IPFS: Free tier available
- ✅ Ethereum RPC: Free (limited requests)

**Grand Total**: $14-20/month for full production setup! 💰

---

## 🆘 Common Issues & Solutions

### "Cannot connect to MongoDB"
**Solution**: 
1. Go to MongoDB Atlas → Network Access
2. Add IP Address → "Allow from Anywhere" (`0.0.0.0/0`)
3. Verify connection string format

### "Smart contract not found"
**Solution**: 
1. Deploy contract: `npx hardhat run scripts/deploy.js --network sepolia`
2. Copy address from output
3. Add to `CONTRACT_ADDRESS` env var

### "CORS error" in browser console
**Solution**: 
1. Update [backend/server.js](backend/server.js) CORS origins
2. Add your DigitalOcean URLs
3. Push to GitHub (auto-deploy)

### "Clerk authentication not working"
**Solution**: 
1. Go to Clerk Dashboard
2. Add your DigitalOcean URLs to Allowed Origins
3. Configure Satellite Mode if using multiple domains

### "Build failed" on DigitalOcean
**Solution**: 
1. Check build logs
2. Verify all environment variables are set
3. Try with `--legacy-peer-deps` flag in build command

---

## 📚 Next Steps

1. ✅ **Read Full Documentation**:
   - [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) - How everything works
   - [DIGITALOCEAN_DEPLOYMENT.md](./DIGITALOCEAN_DEPLOYMENT.md) - Detailed deployment guide

2. ✅ **Configure Custom Domain**:
   - Buy domain from Namecheap/GoDaddy
   - Add to DigitalOcean (see deployment guide)

3. ✅ **Enable Monitoring**:
   - Set up alerts in DigitalOcean dashboard
   - Monitor errors and performance

4. ✅ **Optimize Performance**:
   - Enable CDN for static assets
   - Add caching headers
   - Compress images

5. ✅ **Deploy to Mainnet**:
   - Deploy contract to Ethereum Mainnet or Polygon
   - Update RPC_URL and CONTRACT_ADDRESS
   - Test thoroughly before launch!

---

## 🤝 Support

Need help? Check these resources:

- 📖 **Documentation**: [DOCUMENTATION.md](./DOCUMENTATION.md)
- 🚀 **Deployment Guide**: [DIGITALOCEAN_DEPLOYMENT.md](./DIGITALOCEAN_DEPLOYMENT.md)
- 🏗️ **Architecture**: [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md)
- 💬 **DigitalOcean Community**: [community.digitalocean.com](https://www.digitalocean.com/community)
- 🐛 **Report Bug**: Open an issue on GitHub

---

## ✅ Pre-Deployment Checklist

Before deploying to production:

- [ ] Smart contract deployed to Sepolia/Mainnet
- [ ] MongoDB Atlas cluster created
- [ ] All API keys obtained (Clerk, Lighthouse, RPC)
- [ ] `.env` file configured locally (for testing)
- [ ] Code pushed to GitHub
- [ ] Tested locally: `bun run dev`
- [ ] All 4 services start successfully
- [ ] Smart contract functions work (create, buy, relist)
- [ ] IPFS uploads work
- [ ] Authentication works

If all checked ✅, you're ready to deploy!

---

**Good luck with your deployment! 🚀**

If you run into any issues, consult the full deployment guide or open an issue.

---

**Last Updated**: March 2026  
**Project**: ChainTorque - Web3 Engineering Platform  
**Maintainer**: Archisman Pal
