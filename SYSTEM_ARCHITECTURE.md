# ChainTorque System Architecture & Technical Overview

> **Last Updated**: March 2026  
> **Version**: 1.0.0

---

## 📖 Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Service-by-Service Breakdown](#service-by-service-breakdown)
5. [Data Flow & Communication](#data-flow--communication)
6. [Database Schema](#database-schema)
7. [Smart Contract Architecture](#smart-contract-architecture)
8. [Authentication & Authorization](#authentication--authorization)
9. [File Storage & IPFS](#file-storage--ipfs)
10. [Development Workflow](#development-workflow)
11. [Testing Strategy](#testing-strategy)
12. [Security Considerations](#security-considerations)
13. [Performance Optimization](#performance-optimization)
14. [Deployment Architecture](#deployment-architecture)

---

## 🎯 Executive Summary

**ChainTorque** is a decentralized Web3 engineering platform that combines:
- **Blockchain Technology** (NFT-based licensing)
- **Professional CAD Tools** (Browser-based 3D modeling)
- **Marketplace** (Buy/sell 3D models as NFTs)
- **AI Assistance** (Natural language CAD commands)

### Key Features
✅ NFT-based model ownership & licensing  
✅ Browser-based CAD editor with OpenCascade.js  
✅ Decentralized storage via IPFS (Lighthouse)  
✅ Smart contract royalties (2.5% platform + customizable creator royalties)  
✅ Interactive 3D previews  
✅ Wallet-based authentication (MetaMask + Clerk)  

### Architecture Type
**Monorepo Microservices** - 4 independent services in a single repository:
1. Backend API (Express + MongoDB + Web3)
2. Landing Page (Marketing site)
3. Marketplace (NFT marketplace)
4. CAD Editor (3D modeling tool)

---

## 🏗️ System Architecture

### High-Level Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER (Browser)                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌─────────────────┐  ┌──────────────────┐  ┌─────────────────────────┐│
│  │  Landing Page   │  │   Marketplace    │  │     CAD Editor          ││
│  │  React + Vite   │  │  React + Vite    │  │    React + Vite         ││
│  │  Port: 5000     │  │  Port: 8080      │  │    Port: 3001           ││
│  │                 │  │                  │  │                         ││
│  │  • Marketing    │  │  • NFT Catalog   │  │  • 2D Sketching         ││
│  │  • Features     │  │  • 3D Preview    │  │  • 3D Modeling          ││
│  │  • Clerk Auth   │  │  • Web3 (ethers) │  │  • WASM (OpenCascade)   ││
│  │  • GSAP Anims   │  │  • Upload Models │  │  • Three.js Rendering   ││
│  └────────┬────────┘  └────────┬─────────┘  └───────────┬─────────────┘│
│           │                    │                         │              │
└───────────┼────────────────────┼─────────────────────────┼──────────────┘
            │                    │                         │
            └────────────────────▼─────────────────────────┘
                                 │
                                 │ HTTP/REST API
                                 │ WebSocket (optional)
                                 │
┌────────────────────────────────▼─────────────────────────────────────────┐
│                        APPLICATION LAYER                                 │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                  Backend API (Express.js)                         │  │
│  │                        Port: 5001                                 │  │
│  ├───────────────────────────────────────────────────────────────────┤  │
│  │  Routes:                                                          │  │
│  │    • /api/marketplace  ───→  Marketplace CRUD operations         │  │
│  │    • /api/user         ───→  User management                     │  │
│  │    • /api/ai           ───→  AI copilot (Groq, Hugging Face)    │  │
│  │    • /api/proxy-model  ───→  CORS bypass for IPFS               │  │
│  │    • /api/web3/status  ───→  Blockchain connection status        │  │
│  │                                                                   │  │
│  │  Middleware:                                                      │  │
│  │    • CORS (whitelist origins)                                    │  │
│  │    • Multer (file uploads)                                       │  │
│  │    • Express.json (body parsing)                                 │  │
│  │                                                                   │  │
│  │  Services:                                                        │  │
│  │    • Web3Manager      ───→  Smart contract interaction           │  │
│  │    • LighthouseStorage ───→ IPFS uploads                        │  │
│  │    • EventListener     ───→ Blockchain event monitoring         │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                           │
└───────────────────────────────┬───────────────────────────────────────────┘
                                │
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
│  DATA LAYER      │  │ BLOCKCHAIN LAYER │  │  STORAGE LAYER       │
├──────────────────┤  ├──────────────────┤  ├──────────────────────┤
│                  │  │                  │  │                      │
│ MongoDB Atlas    │  │ Ethereum Network │  │ IPFS (Lighthouse)    │
│                  │  │                  │  │                      │
│ Collections:     │  │ • Sepolia (Test) │  │ • 3D Model Files     │
│ • marketitems    │  │ • Mainnet (Prod) │  │ • Images/Thumbnails  │
│ • transactions   │  │                  │  │ • Metadata JSON      │
│ • users          │  │ Smart Contract:  │  │                      │
│                  │  │ ChainTorque      │  │ CIDs (Content IDs):  │
│ Indexes:         │  │ Marketplace.sol  │  │ • Permanent Storage  │
│ • tokenId        │  │                  │  │ • Decentralized      │
│ • seller         │  │ Functions:       │  │ • Censorship-resist  │
│ • category       │  │ • createToken    │  │                      │
│ • status         │  │ • buyToken       │  │ Gateway URLs:        │
│                  │  │ • relistToken    │  │ gateway.lighthouse   │
│                  │  │ • burn           │  │ .storage             │
└──────────────────┘  └──────────────────┘  └──────────────────────┘
                                │
                                │
                        ┌───────▼────────┐
                        │  RPC Provider  │
                        │  (Ankr/Alchemy)│
                        └────────────────┘
```

### Component Interaction Flow

```
┌──────────────┐
│    User      │
└──────┬───────┘
       │
       │ 1. Visit Landing Page
       ▼
┌──────────────────┐
│  Landing Page    │──────┐
│  (React SPA)     │      │ 2. Sign In with Clerk
└──────────────────┘      │
                          ▼
                   ┌──────────────────┐
                   │  Clerk Auth      │
                   │  (OAuth/Wallet)  │
                   └──────┬───────────┘
       ┌──────────────────┘
       │ 3. Redirect to Marketplace
       ▼
┌──────────────────┐
│  Marketplace     │
│  (React SPA)     │
└──────┬───────────┘
       │
       │ 4. Fetch NFTs
       ▼
┌──────────────────┐         ┌──────────────────┐
│  Backend API     │────────▶│  MongoDB         │
│  /api/marketplace│         │  marketitems     │
└──────┬───────────┘         └──────────────────┘
       │
       │ 5. User Uploads Model
       ▼
┌──────────────────┐         ┌──────────────────┐
│  POST /upload    │────────▶│  Lighthouse IPFS │
│  (Multer)        │         │  (File Storage)  │
└──────┬───────────┘         └──────────────────┘
       │                              │
       │ 6. Get IPFS CID              │
       ◀──────────────────────────────┘
       │
       │ 7. Return tokenURI to Frontend
       ▼
┌──────────────────┐
│  Marketplace UI  │
└──────┬───────────┘
       │
       │ 8. Connect MetaMask
       │ 9. Call Smart Contract
       ▼
┌──────────────────┐         ┌──────────────────┐
│  MetaMask Wallet │────────▶│  Smart Contract  │
│  (ethers.js)     │         │  createToken()   │
└──────────────────┘         └──────┬───────────┘
                                    │
       ┌────────────────────────────┘
       │ 10. Event: MarketItemCreated
       ▼
┌──────────────────┐         ┌──────────────────┐
│  Backend Listens │────────▶│  Save to MongoDB │
│  (EventListener) │         │  (Sync)          │
└──────────────────┘         └──────────────────┘
```

---

## 🛠️ Technology Stack

### Frontend Technologies

#### Landing Page
```json
{
  "framework": "React 18.2",
  "build": "Vite 6.0",
  "language": "TypeScript 5.0",
  "styling": "Tailwind CSS 3.4",
  "animations": "GSAP 3.14, Framer Motion 12.34",
  "routing": "React Router 7.11",
  "auth": "Clerk React 5.59",
  "scrolling": "Lenis 1.3"
}
```

**Key Dependencies**:
- `gsap` - Professional animations
- `lenis` - Smooth scrolling
- `framer-motion` - React animations
- `@clerk/clerk-react` - Authentication

#### Marketplace
```json
{
  "framework": "React 18.3",
  "build": "Vite 6.0",
  "language": "TypeScript 5.5",
  "styling": "Tailwind CSS 3.4",
  "ui": "Radix UI (shadcn/ui components)",
  "web3": "ethers.js 6.13",
  "3d": "Three.js (for model preview)",
  "state": "React Query (TanStack Query 5.51)",
  "auth": "Clerk React 5.2"
}
```

**Key Dependencies**:
- `ethers` - Ethereum library
- `@clerk/clerk-react` - Authentication
- `@tanstack/react-query` - Server state management
- `lucide-react` - Icon library
- `sonner` - Toast notifications

#### CAD Editor
```json
{
  "framework": "React 18.3",
  "build": "Vite 6.0",
  "language": "JavaScript (ES6+)",
  "3d": "Three.js 0.179",
  "cad": "OpenCascade.js 1.1 (WASM)",
  "icons": "React Icons 5.5"
}
```

**Key Dependencies**:
- `opencascade.js` - CAD kernel (BREP geometry)
- `three` - 3D rendering
- `react-icons` - UI icons

### Backend Technologies

```json
{
  "runtime": "Node.js 20.x (Bun compatible)",
  "framework": "Express 5.2",
  "database": "MongoDB 9.0 (Mongoose ODM)",
  "blockchain": "ethers.js 6.16, Hardhat 2.26",
  "storage": "IPFS (Lighthouse SDK 0.4)",
  "ai": "Groq SDK 0.37, Gradio Client 2.1"
}
```

**Key Dependencies**:
- `express` - Web framework
- `mongoose` - MongoDB ODM
- `ethers` - Ethereum library
- `hardhat` - Smart contract development
- `@lighthouse-web3/sdk` - IPFS uploads
- `multer` - File uploads
- `cors` - Cross-origin requests
- `dotenv` - Environment variables

### Smart Contract

```solidity
// Solidity 0.8.19
// OpenZeppelin Contracts 4.9.6

ChainTorqueMarketplace.sol
├── ERC721URIStorage  (NFT standard)
├── Ownable           (Access control)
├── ReentrancyGuard   (Security)
└── Pausable          (Emergency stop)
```

**Functions**:
- `createToken()` - Mint new NFT
- `batchCreateTokens()` - Bulk minting
- `buyToken()` - Purchase NFT
- `relistToken()` - Resell NFT
- `burn()` - Destroy NFT

### Development Tools

```yaml
Package Manager: Bun 1.x (npm compatible)
Monorepo Manager: npm workspaces
Task Runner: Concurrently
Linter: ESLint
Formatter: Prettier (optional)
Version Control: Git
CI/CD: GitHub Actions (optional)
```

---

## 📦 Service-by-Service Breakdown

### 1. Backend Service

**Purpose**: Central API for all frontends + smart contract interaction

**Directory**: `/backend`

**Key Files**:
```
backend/
├── server.js                    # Main Express app (229 lines)
├── web3.js                      # Web3Manager class (382 lines)
├── hardhat.config.js            # Hardhat configuration
├── nodemon.json                 # Dev auto-reload config
├── package.json                 # Dependencies
│
├── contracts/
│   └── ChainTorqueMarketplace.sol  # Solidity smart contract (375 lines)
│
├── scripts/
│   ├── deploy.js                # Contract deployment (108 lines)
│   └── wipe_db.js               # Database reset
│
├── models/                      # Mongoose schemas
│   ├── MarketItem.js
│   ├── Transaction.js
│   └── User.js
│
├── routes/                      # API routes
│   ├── marketplace.js           # NFT CRUD (548 lines)
│   ├── user.js                  # User management
│   └── ai.js                    # AI copilot
│
├── services/
│   ├── eventListener.js         # Blockchain event sync
│   └── lighthouseStorage.js     # IPFS uploads
│
└── middleware/
    └── upload.js                # Multer file handling
```

**API Endpoints**:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/api/marketplace` | GET | Get all NFTs |
| `/api/marketplace/upload-files` | POST | Upload to IPFS |
| `/api/marketplace/create-token` | POST | Mint NFT (deprecated) |
| `/api/marketplace/:id` | GET | Get single NFT |
| `/api/user/profile` | GET/PUT | User profile |
| `/api/ai/chat` | POST | AI copilot |
| `/api/proxy-model` | GET | IPFS CORS proxy |
| `/api/web3/status` | GET | Web3 status |
| `/api/web3/balance/:address` | GET | Wallet balance |

**Environment Variables**:
- `MONGODB_URI` - MongoDB connection string
- `RPC_URL` - Ethereum RPC endpoint
- `PRIVATE_KEY` - Wallet private key (server signer)
- `CONTRACT_ADDRESS` - Deployed smart contract address
- `LIGHTHOUSE_API_KEY` - IPFS API key
- `CLERK_PUBLISHABLE_KEY` - Clerk auth key
- `GROQ_API_KEY` - AI API key (optional)
- `HF_TOKEN` - Hugging Face token (optional)

**Port**: 5001 (development), `$PORT` (production)

**Startup Command**:
```bash
# Development
npm run dev  # Uses nodemon

# Production
npm start    # node server.js
```

### 2. Landing Page

**Purpose**: Marketing website and entry point

**Directory**: `/Landing Page (Frontend)`

**Key Files**:
```
Landing Page (Frontend)/
├── index.html
├── vite.config.ts              # Vite configuration
├── tailwind.config.ts          # Tailwind CSS config
├── package.json
│
└── src/
    ├── main.tsx                # Entry point
    ├── App.tsx                 # Main component (155 lines)
    ├── index.css               # Global styles
    │
    └── components/             # React components
        ├── Header.tsx
        ├── Hero.tsx
        ├── ModelShowcase.tsx
        ├── Collections.tsx
        ├── Features.tsx
        ├── HowItWorks.tsx
        ├── Gallery.tsx
        ├── Testimonials.tsx
        ├── DetailedFeatures.tsx
        ├── Pricing.tsx
        ├── Footer.tsx
        └── BackToTop.tsx
```

**Features**:
- ✅ Smooth scrolling (Lenis)
- ✅ GSAP animations with ScrollTrigger
- ✅ Clerk authentication integration
- ✅ Dark mode support
- ✅ Responsive design
- ✅ SEO optimized

**Environment Variables**:
- `VITE_CLERK_PUBLISHABLE_KEY` - Clerk auth
- `VITE_MARKETPLACE_URL` - Marketplace link (optional)

**Port**: 5000 (development), `$PORT` (production)

**Build Output**: `dist/`

**Startup Commands**:
```bash
# Development
npm run dev    # Vite dev server

# Build
npm run build  # TypeScript + Vite build

# Preview
npm run preview
```

### 3. Marketplace

**Purpose**: NFT marketplace for buying/selling 3D models

**Directory**: `/Marketplace (Frontend)`

**Key Files**:
```
Marketplace (Frontend)/
├── index.html
├── vite.config.ts
├── tailwind.config.ts
├── package.json
│
└── src/
    ├── main.tsx
    ├── App.tsx                 # Main component (155 lines)
    ├── silence-warnings.ts     # Console warning filter
    │
    ├── pages/                  # Route pages
    │   ├── Index.tsx           # Home/catalog
    │   ├── ProductDetail.tsx   # NFT detail page
    │   ├── Upload.tsx          # Upload model
    │   ├── Dashboard.tsx       # User dashboard
    │   ├── Edit.tsx            # Edit profile
    │   ├── SignUp.tsx          # Sign up page
    │   └── NotFound.tsx        # 404 page
    │
    ├── components/             # Reusable components
    │   ├── BackendStatus.tsx   # API status indicator
    │   ├── ui/                 # shadcn/ui components
    │   └── ...
    │
    ├── contexts/               # React contexts
    │   ├── AuthContext.tsx     # Clerk + Web3 auth
    │   └── StatusPanelContext.tsx
    │
    ├── hooks/                  # Custom hooks
    │   └── ...
    │
    ├── services/               # API services
    │   └── api.ts
    │
    └── lib/                    # Utilities
        ├── urls.ts
        └── utils.ts
```

**Features**:
- ✅ Browse NFT catalog
- ✅ 3D model preview (Three.js)
- ✅ MetaMask integration
- ✅ Smart contract interaction
- ✅ Upload models to IPFS
- ✅ Purchase NFTs
- ✅ User dashboard
- ✅ Clerk authentication
- ✅ Responsive design

**Environment Variables**:
- `VITE_CLERK_PUBLISHABLE_KEY` - Clerk auth
- `VITE_API_URL` - Backend API URL
- `VITE_CONTRACT_ADDRESS` - Smart contract address
- `VITE_LANDING_URL` - Landing page URL (for auth)

**Port**: 8080 (development), `$PORT` (production)

**Build Output**: `dist/`

**Startup Commands**:
```bash
# Development
npm run dev

# Build
npm run build   # TypeScript + Vite build

# Lint
npm run lint
```

### 4. CAD Editor

**Purpose**: Browser-based 3D modeling tool

**Directory**: `/CAD (Frontend)`

**Key Files**:
```
CAD (Frontend)/
├── index.html
├── vite.config.js
├── package.json
│
└── src/
    ├── main.jsx
    ├── App.jsx                 # Main component (999 lines)
    ├── App.css                 # Component styles
    ├── index.css               # Global styles
    │
    ├── components/
    │   ├── ViewportManager.jsx # 3D viewport + 2D canvas
    │   ├── FeatureTree.jsx     # Model hierarchy
    │   ├── CADOperations.jsx   # Boolean operations
    │   ├── ImageTo3D.jsx       # AI image-to-3D
    │   ├── ThreeViewer.jsx     # Three.js scene
    │   └── UploadToMarketplaceModal.jsx
    │
    ├── cad/                    # CAD core logic
    │   ├── CADGeometryService.js    # OpenCascade wrapper
    │   ├── OpenCascadeLoader.js     # WASM loader
    │   ├── useCAD.js                # Custom hook
    │   └── index.js
    │
    └── utils/
        └── geometryUtils.js
```

**Features**:
- ✅ 2D sketching (line, circle, polygon)
- ✅ 3D extrusion (OpenCascade.js)
- ✅ Boolean operations (union, cut, intersect)
- ✅ Real-time 3D preview (Three.js)
- ✅ Camera controls (orbit, zoom, pan)
- ✅ AI copilot (Torquy)
- ✅ AI image-to-3D
- ✅ Export to marketplace
- ✅ Keyboard shortcuts

**Environment Variables**: None (fully client-side)

**Port**: 3001 (development), `$PORT` (production)

**Build Output**: `build/`

**Startup Commands**:
```bash
# Development
npm run dev

# Build
npm run build   # Vite build
```

---

## 🔄 Data Flow & Communication

### 1. User Authentication Flow

```
┌──────────┐   1. Visit Landing    ┌─────────────┐
│  User    │ ──────────────────────▶│ Landing Page│
└────┬─────┘                        └──────┬──────┘
     │                                     │
     │                              2. Click "Sign In"
     │                                     │
     │                                     ▼
     │                              ┌─────────────┐
     │         3. OAuth Flow        │    Clerk    │
     │◀─────────────────────────────│   (Auth)    │
     │         (Google/Wallet)      └─────────────┘
     │
     │ 4. Get JWT Token
     │
     │                              ┌─────────────┐
     └─────────────────────────────▶│ Marketplace │
       5. Redirect with Token       └─────────────┘
                                           │
                                    6. API Calls with JWT
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   Backend   │
                                    │ (Verifies   │
                                    │   Token)    │
                                    └─────────────┘
```

### 2. NFT Creation Flow

```
┌──────────┐   1. Upload Files      ┌─────────────┐
│  User    │ ──────────────────────▶│ Marketplace │
└──────────┘                        │   /upload   │
                                    └──────┬──────┘
                                           │
                                    2. POST /api/marketplace/upload-files
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   Backend   │
                                    │   (Multer)  │
                                    └──────┬──────┘
                                           │
                                    3. Upload to IPFS
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │ Lighthouse  │
                                    │    IPFS     │
                                    └──────┬──────┘
                                           │
                                    4. Return CID
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   Backend   │
                                    │ (Create JSON│
                                    │  Metadata)  │
                                    └──────┬──────┘
                                           │
                                    5. Upload metadata to IPFS
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │ Lighthouse  │
                                    │    IPFS     │
                                    └──────┬──────┘
                                           │
                                    6. Return tokenURI (metadata CID)
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │ Marketplace │
                                    │  (Frontend) │
                                    └──────┬──────┘
                                           │
                                    7. Connect MetaMask
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   MetaMask  │
                                    │   Wallet    │
                                    └──────┬──────┘
                                           │
                                    8. Call createToken(tokenURI, price, category, royalty)
                                           │
                                           ▼
                                    ┌─────────────────────┐
                                    │  Smart Contract     │
                                    │  (Blockchain)       │
                                    │  - Mint NFT         │
                                    │  - Transfer to      │
                                    │    marketplace      │
                                    │  - Emit Event       │
                                    └──────┬──────────────┘
                                           │
                                    9. Event: MarketItemCreated
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   Backend   │
                                    │ (Event      │
                                    │  Listener)  │
                                    └──────┬──────┘
                                           │
                                    10. Save to MongoDB
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   MongoDB   │
                                    │ marketitems │
                                    └─────────────┘
```

### 3. NFT Purchase Flow

```
┌──────────┐   1. Browse NFTs       ┌─────────────┐
│  Buyer   │ ──────────────────────▶│ Marketplace │
└────┬─────┘                        └──────┬──────┘
     │                                     │
     │                              2. GET /api/marketplace
     │                                     │
     │                                     ▼
     │                              ┌─────────────┐
     │       3. NFT List            │   Backend   │
     │◀─────────────────────────────│  (MongoDB)  │
     │                              └─────────────┘
     │
     │ 4. Click "Buy Now"
     │
     ▼
┌─────────────┐   5. Connect Wallet ┌─────────────┐
│ MetaMask    │◀────────────────────│ Marketplace │
│  Popup      │                     └─────────────┘
└──────┬──────┘
       │
       │ 6. Approve Transaction
       │
       ▼
┌─────────────────────┐
│  Smart Contract     │
│  buyToken(tokenId)  │
│  - Check price      │
│  - Transfer ETH     │
│  - Pay royalties    │
│  - Pay platform fee │
│  - Transfer NFT     │
│  - Update state     │
│  - Emit Event       │
└──────┬──────────────┘
       │
       │ 7. Event: MarketItemSold
       │
       ▼
┌─────────────┐
│   Backend   │
│ (Listener)  │
└──────┬──────┘
       │
       │ 8. Update MongoDB (status: sold)
       │
       ▼
┌─────────────┐         ┌─────────────┐
│  MongoDB    │         │ Transaction │
│ marketitems │         │  Collection │
└─────────────┘         └─────────────┘
```

---

## 💾 Database Schema

### MongoDB Collections

#### 1. `marketitems` Collection

```javascript
{
  _id: ObjectId("..."),
  tokenId: 42,                          // Blockchain token ID (unique)
  title: "Gear Assembly",
  description: "High-precision gear...",
  category: "Electronics",
  price: "0.05",                        // ETH price (string)
  royalty: 500,                         // 5.0% (basis points)
  seller: "0xABC...123",                // Wallet address
  owner: "0xDEF...456",                 // Current owner
  creator: "0xABC...123",               // Original creator
  status: "active",                     // active | sold | burned
  images: [
    "https://gateway.lighthouse.storage/ipfs/Qm..."
  ],
  modelUrl: "https://gateway.lighthouse.storage/ipfs/Qm...",
  tokenURI: "ipfs://Qm...",            // Metadata URI
  contractAddress: "0x123...",
  createdAt: ISODate("2026-03-01T..."),
  updatedAt: ISODate("2026-03-01T..."),
  sold: false,
  blockchainSynced: true
}
```

**Indexes**:
- `tokenId` (unique)
- `seller`
- `status`
- `category`
- `createdAt` (descending)

#### 2. `transactions` Collection

```javascript
{
  _id: ObjectId("..."),
  tokenId: 42,
  transactionHash: "0xabc123...",      // Blockchain tx hash
  from: "0xBuyer...",
  to: "0xSeller...",
  price: "0.05",
  type: "sale",                         // sale | listing | transfer
  timestamp: ISODate("2026-03-01T..."),
  blockNumber: 12345678,
  gasUsed: "123456",
  gasPrice: "20000000000"
}
```

**Indexes**:
- `tokenId`
- `from`
- `to`
- `timestamp` (descending)

#### 3. `users` Collection

```javascript
{
  _id: ObjectId("..."),
  clerkId: "user_2abc123...",          // Clerk user ID
  walletAddress: "0xABC...123",        // Primary wallet
  email: "user@example.com",
  username: "creator123",
  displayName: "John Doe",
  avatar: "https://...",
  bio: "3D artist and engineer",
  createdNFTs: [42, 43, 44],           // Token IDs
  ownedNFTs: [1, 5, 42],               // Token IDs
  favorites: [7, 8, 9],                // Token IDs
  totalSales: 5,
  totalRevenue: "0.25",                // ETH (string)
  joinedAt: ISODate("2026-01-01T..."),
  lastActive: ISODate("2026-03-01T...")
}
```

**Indexes**:
- `clerkId` (unique)
- `walletAddress` (unique)
- `username` (unique)

---

## 🔐 Smart Contract Architecture

### ChainTorqueMarketplace.sol

**Inheritance**:
```solidity
ChainTorqueMarketplace is ERC721URIStorage, Ownable, ReentrancyGuard, Pausable
```

**State Variables**:
```solidity
// Constants
uint256 public constant LISTING_PRICE = 0.00025 ether;
uint256 public constant PLATFORM_FEE_BPS = 250;  // 2.5%

// Storage
uint256 private _currentTokenId;
uint256 private _totalItemsSold;

// Mappings
mapping(uint256 => MarketItem) private _marketItems;
mapping(address => uint256[]) private _userTokens;
mapping(uint32 => uint256[]) private _categoryTokens;
mapping(address => bool) private _authorizedCreators;
```

**Struct**:
```solidity
struct MarketItem {
    uint256 tokenId;
    uint128 price;
    uint64 createdAt;
    uint32 category;
    uint24 royalty;      // Basis points (5% = 500)
    bool sold;
    address seller;
    address owner;
    address creator;     // For royalty distribution
}
```

**Key Functions**:

1. **createToken** (Public):
   ```solidity
   function createToken(
       string calldata tokenURI,
       uint128 price,
       uint32 category,
       uint24 royalty
   ) external payable nonReentrant whenNotPaused returns (uint256)
   ```
   - Requires `LISTING_PRICE` (0.00025 ETH)
   - Mints NFT
   - Transfers to marketplace contract
   - Emits `MarketItemCreated`

2. **buyToken** (Public):
   ```solidity
   function buyToken(uint256 tokenId) external payable nonReentrant whenNotPaused
   ```
   - Validates price
   - Calculates platform fee (2.5%)
   - Calculates creator royalty (if resale)
   - Transfers ETH to seller/creator/platform
   - Transfers NFT to buyer
   - Emits `MarketItemSold`

3. **relistToken** (Public):
   ```solidity
   function relistToken(uint256 tokenId, uint128 price) external payable
   ```
   - Requires `LISTING_PRICE`
   - Transfers NFT back to marketplace
   - Updates price and seller
   - Emits `MarketItemRelisted`

4. **batchCreateTokens** (Authorized):
   ```solidity
   function batchCreateTokens(
       string[] calldata tokenURIs,
       uint128[] calldata prices,
       uint32[] calldata categories,
       uint24[] calldata royalties
   ) external payable
   ```
   - Bulk minting (up to 50 NFTs)
   - Gas optimized

**Events**:
```solidity
event MarketItemCreated(uint256 indexed tokenId, address indexed seller, uint128 indexed price, uint32 category, uint256 timestamp);
event MarketItemSold(uint256 indexed tokenId, address indexed seller, address indexed buyer, uint128 price);
event MarketItemRelisted(uint256 indexed tokenId, address indexed seller, uint128 indexed price);
event MarketItemBurned(uint256 indexed tokenId, address indexed owner, uint64 timestamp);
event RoyaltyUpdated(uint256 indexed tokenId, uint24 royalty);
```

**Security Features**:
- ✅ ReentrancyGuard (prevents reentrancy attacks)
- ✅ Pausable (emergency stop)
- ✅ Ownable (admin functions)
- ✅ Input validation (price > 0, royalty <= 10%)
- ✅ Safe ETH transfers (`call` instead of `transfer`)

---

## 🔑 Authentication & Authorization

### Clerk + Wallet Auth

**Primary Method**: Clerk (OAuth + Web3 wallet sign-in)

**Supported Methods**:
- Google OAuth
- GitHub OAuth
- MetaMask wallet
- WalletConnect
- Email + Password

**Flow**:
```
1. User visits Landing Page
2. Clicks "Sign In" → Redirects to Clerk
3. Clerk handles authentication
4. Returns JWT token
5. Frontend stores token in localStorage
6. All API requests include: Authorization: Bearer <token>
7. Backend verifies token (optional, Clerk handles it)
```

**Clerk Configuration**:
```javascript
// Landing Page (Primary)
<ClerkProvider
  publishableKey={VITE_CLERK_PUBLISHABLE_KEY}
  afterSignOutUrl="/"
>
  {children}
</ClerkProvider>

// Marketplace (Satellite)
<ClerkProvider
  publishableKey={VITE_CLERK_PUBLISHABLE_KEY}
  isSatellite={true}
  domain="landing.yourdomain.com"
  signInUrl="https://landing.yourdomain.com/sign-in"
  signUpUrl="https://landing.yourdomain.com/sign-up"
>
  {children}
</ClerkProvider>
```

**Satellite Mode**: Marketplace is a "satellite" app that relies on Landing Page for auth.

### Web3 Wallet Integration

**Library**: ethers.js v6

**Usage**:
```typescript
import { BrowserProvider } from 'ethers';

// Connect MetaMask
async function connectWallet() {
  if (!window.ethereum) {
    alert('MetaMask not installed');
    return;
  }
  
  const provider = new BrowserProvider(window.ethereum);
  const accounts = await provider.send('eth_requestAccounts', []);
  const signer = await provider.getSigner();
  
  return { provider, signer, address: accounts[0] };
}

// Call smart contract
const contract = new ethers.Contract(
  CONTRACT_ADDRESS,
  ABI,
  signer
);

await contract.buyToken(tokenId, { value: price });
```

**Authorization Levels**:
1. **Public** (no auth required):
   - View NFT catalog
   - View contract info
   
2. **Authenticated User** (Clerk):
   - Upload models
   - Favorite NFTs
   - View profile
   
3. **Wallet Connected** (MetaMask):
   - Buy NFTs
   - Mint NFTs
   - Resell NFTs
   
4. **Admin** (contract owner):
   - Pause contract
   - Set authorized creators
   - Withdraw funds

---

## 📁 File Storage & IPFS

### Lighthouse Storage

**SDK**: `@lighthouse-web3/sdk`

**API Key**: Required (get from [lighthouse.storage](https://lighthouse.storage))

**Upload Flow**:
```javascript
import lighthouse from '@lighthouse-web3/sdk';

// Upload file
const uploadFile = async (filePath) => {
  const result = await lighthouse.upload(
    filePath,
    process.env.LIGHTHOUSE_API_KEY
  );
  
  return {
    cid: result.data.Hash,
    url: `https://gateway.lighthouse.storage/ipfs/${result.data.Hash}`
  };
};

// Upload JSON metadata
const uploadMetadata = async (metadata) => {
  const result = await lighthouse.uploadText(
    JSON.stringify(metadata),
    process.env.LIGHTHOUSE_API_KEY
  );
  
  return {
    cid: result.data.Hash,
    url: `ipfs://${result.data.Hash}`  // tokenURI format
  };
};
```

**Metadata Format** (ERC-721 standard):
```json
{
  "name": "Gear Assembly",
  "description": "High-precision gear for robotics",
  "image": "https://gateway.lighthouse.storage/ipfs/Qm...",
  "images": [
    "https://gateway.lighthouse.storage/ipfs/Qm...",
    "https://gateway.lighthouse.storage/ipfs/Qm..."
  ],
  "animation_url": "https://gateway.lighthouse.storage/ipfs/Qm.../model.glb",
  "external_url": "https://chaintorque.com",
  "attributes": [
    {
      "trait_type": "Category",
      "value": "Electronics"
    },
    {
      "trait_type": "File Type",
      "value": "GLB"
    },
    {
      "trait_type": "Created",
      "value": "2026-03-01"
    }
  ],
  "properties": {
    "category": "Electronics",
    "fileType": "glb",
    "creator": "0xABC...123"
  }
}
```

**Gateway URLs**:
- **Official**: `https://gateway.lighthouse.storage/ipfs/<CID>`
- **Backup**: `https://ipfs.io/ipfs/<CID>`
- **Custom**: Configure your own gateway

---

## 🛠️ Development Workflow

### Local Setup

1. **Clone repository**:
   ```bash
   git clone https://github.com/your-username/ChainTorque.git
   cd ChainTorque
   ```

2. **Install dependencies**:
   ```bash
   # Using Bun (recommended)
   bun install
   
   # Or using npm
   npm install --legacy-peer-deps
   ```

3. **Configure environment**:
   ```bash
   # Copy example env
   cp .env.example .env
   
   # Edit with your values
   nano .env
   ```

4. **Start MongoDB**:
   - Use MongoDB Atlas (cloud)
   - Or local: `mongod --dbpath ./data`

5. **Deploy smart contract**:
   ```bash
   cd backend
   npx hardhat compile
   npx hardhat node  # Terminal 1 - local blockchain
   npx hardhat run scripts/deploy.js --network localhost  # Terminal 2
   ```

6. **Start all services**:
   ```bash
   # From root directory
   bun run dev
   
   # Or individually
   bun run dev:backend      # Port 5001
   bun run dev:landing      # Port 5000
   bun run dev:marketplace  # Port 8080
   bun run dev:cad          # Port 3001
   ```

7. **Access apps**:
   - Landing: http://localhost:5000
   - Marketplace: http://localhost:8080
   - CAD: http://localhost:3001
   - Backend: http://localhost:5001/health

### Development Commands

```bash
# Format code
bun run format

# Lint frontend
bun run lint

# Test smart contracts
bun run test:contracts

# Build all frontends
bun run build

# Clean node_modules
bun run clean
```

---

## ✅ Testing Strategy

### Smart Contract Testing

```bash
cd backend
npx hardhat test
```

**Test Coverage**:
- ✅ Token creation
- ✅ Token purchase
- ✅ Royalty distribution
- ✅ Platform fees
- ✅ Access control
- ✅ Reentrancy protection

### API Testing

**Tools**: Postman, cURL

**Example**:
```bash
# Health check
curl http://localhost:5001/health

# Get marketplace items
curl http://localhost:5001/api/marketplace

# Upload file (requires Postman/Insomnia)
POST http://localhost:5001/api/marketplace/upload-files
Content-Type: multipart/form-data
Body: { image: [file], model: [file], title: "...", description: "..." }
```

### Frontend Testing

**Manual Testing**:
1. User authentication flow
2. NFT browsing
3. MetaMask connection
4. NFT purchase
5. Model upload
6. CAD editor tools

---

## 🔒 Security Considerations

### Smart Contract Security
- ✅ **ReentrancyGuard**: Prevents reentrancy attacks
- ✅ **Pausable**: Emergency stop mechanism
- ✅ **Input Validation**: Price > 0, royalty <= 10%
- ✅ **Safe Transfers**: Using `call` instead of `transfer`
- ✅ **Access Control**: Ownable pattern

### Backend Security
- ✅ **CORS**: Whitelist origins
- ✅ **Environment Variables**: Secrets in `.env`
- ✅ **File Upload Limits**: Max file size
- ✅ **Rate Limiting**: (TODO)
- ✅ **Input Sanitization**: Mongoose validation

### Frontend Security
- ✅ **HTTPS**: Required in production
- ✅ **CSP Headers**: Content Security Policy
- ✅ **XSS Protection**: React auto-escapes
- ✅ **JWT Validation**: Clerk handles it
- ✅ **Wallet Validation**: ethers.js

### Best Practices
- 🔐 Never commit `.env` files
- 🔐 Use separate wallets for dev/prod
- 🔐 Rotate API keys regularly
- 🔐 Monitor smart contract events
- 🔐 Audit smart contracts before mainnet

---

## ⚡ Performance Optimization

### Frontend Optimizations
- ✅ **Code Splitting**: Vite auto-splits
- ✅ **Lazy Loading**: React.lazy for routes
- ✅ **Image Optimization**: Use CDN
- ✅ **Bundle Size**: Tree shaking (Vite)
- ✅ **Caching**: Service Worker (TODO)

### Backend Optimizations
- ✅ **Database Indexing**: MongoDB indexes
- ✅ **Connection Pooling**: Mongoose default
- ✅ **Compression**: gzip middleware (TODO)
- ✅ **Caching**: Redis (TODO)
- ✅ **CDN**: CloudFlare for static assets (TODO)

### Smart Contract Optimizations
- ✅ **Batch Operations**: `batchCreateTokens`
- ✅ **Packed Structs**: Reduced storage costs
- ✅ **Events Over Storage**: Cheaper
- ✅ **Gas Optimization**: Solidity 0.8.19 features

---

## 🚀 Deployment Architecture

See [DIGITALOCEAN_DEPLOYMENT.md](./DIGITALOCEAN_DEPLOYMENT.md) for full guide.

### Production Checklist

- [ ] Smart contract deployed to mainnet
- [ ] MongoDB Atlas cluster created
- [ ] Environment variables configured
- [ ] CORS origins updated
- [ ] Clerk domains added
- [ ] SSL certificates provisioned
- [ ] Custom domains configured
- [ ] Monitoring enabled
- [ ] Backups enabled
- [ ] Rate limiting enabled
- [ ] CDN configured
- [ ] Analytics added

---

## 📞 Support & Resources

- **Documentation**: [DOCUMENTATION.md](./DOCUMENTATION.md)
- **Deployment**: [DIGITALOCEAN_DEPLOYMENT.md](./DIGITALOCEAN_DEPLOYMENT.md)
- **GitHub**: [Repository](https://github.com/your-username/ChainTorque)
- **Issues**: [Report Bug](https://github.com/your-username/ChainTorque/issues)

---

**Last Updated**: March 2026  
**Maintainer**: Archisman Pal  
**License**: ISC
