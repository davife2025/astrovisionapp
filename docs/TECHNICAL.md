# AstroVision Technical Documentation

**Version**: 1.0.0  
**Last Updated**: February 2026  
**Status**: Production Deployed

This document provides a deep technical dive into AstroVision's architecture, implementation details, and deployment procedures.

---

## 📑 Table of Contents

- [Architecture Overview](#architecture-overview)
- [System Components](#system-components)
- [Data Flow](#data-flow)
- [Technology Deep Dive](#technology-deep-dive)
- [Setup Guide](#setup-guide)
- [Deployment](#deployment)
- [API Documentation](#api-documentation)
- [Smart Contract Details](#smart-contract-details)
- [Performance & Scaling](#performance--scaling)
- [Security](#security)
- [Troubleshooting](#troubleshooting)

---

## 🏗️ Architecture Overview

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER (Browser)                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                     React SPA (Port 3000)                     │  │
│  │  ┌────────────┬────────────┬────────────┬────────────┐       │  │
│  │  │Observation │ Community  │    DAO     │ Space Lab  │       │  │
│  │  │    Tab     │   Board    │ Dashboard  │ Playground │       │  │
│  │  └────────────┴────────────┴────────────┴────────────┘       │  │
│  │           │              │              │                     │  │
│  │           └──────────────┴──────────────┘                     │  │
│  │                          │                                     │  │
│  │                  ┌───────▼────────┐                           │  │
│  │                  │ WalletContext  │                           │  │
│  │                  │  (EIP-6963)    │                           │  │
│  │                  └───────┬────────┘                           │  │
│  └────────────────────────────┬─────────────────────────────────┘  │
└─────────────────────────────────┼──────────────────────────────────┘
                                  │
        ┌─────────────────────────┴─────────────────────────┐
        │                                                     │
        ▼                                                     ▼
┌──────────────────┐                              ┌──────────────────┐
│  APPLICATION     │                              │   BLOCKCHAIN     │
│     LAYER        │                              │      LAYER       │
│                  │                              │                  │
│  ┌────────────┐ │                              │ ┌──────────────┐ │
│  │  Node.js   │ │                              │ │  AstroDAO    │ │
│  │  Express   │ │                              │ │   Contract   │ │
│  │ (Port 3001)│ │                              │ │ (Solidity)   │ │
│  └─────┬──────┘ │                              │ └──────────────┘ │
│        │        │                              │                  │
│  ┌─────▼──────┐ │                              │   BNB Chain      │
│  │   Routes   │ │                              │   (Testnet)      │
│  ├────────────┤ │                              └──────────────────┘
│  │ AI Service │ │
│  │ Astrometry │ │
│  │ DAO API    │ │
│  │ Profiles   │ │
│  └─────┬──────┘ │
└────────┼────────┘
         │
    ┌────┴────┬────────────┬──────────────┐
    │         │            │              │
    ▼         ▼            ▼              ▼
┌────────┐┌────────┐┌──────────┐┌──────────────┐
│Supabase││HuggingF││Astrometry││  NASA        │
│   DB   ││  ace   ││   .net   ││  SkyView     │
└────────┘└────────┘└──────────┘└──────────────┘
│Storage │ │  AI    │ │ Coords  │ │ Historical │
│Buckets │ │ Models │ │ Solving │ │   Images   │
└────────┘ └────────┘ └──────────┘ └──────────────┘
```

### Technology Stack Summary

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React 18, Three.js, MediaPipe, ethers.js v6, react-icons |
| **Backend** | Node.js 18+, Express 4.18, Multer, Axios, Jimp, Pixelmatch |
| **Database** | Supabase (PostgreSQL 15) |
| **Storage** | Supabase Storage (S3-compatible) |
| **Blockchain** | Solidity 0.8.20, OpenZeppelin 5.0, ethers.js |
| **AI/ML** | HuggingFace Inference API, AstroSage-8B, Kimi-K2.5 |
| **External APIs** | Astrometry.net, NASA SkyView, Twitter OAuth 2.0 |
| **Infrastructure** | Render (backend), Vercel/Netlify (frontend), BNB Chain |

---

## 🔧 System Components

### 1. Frontend Architecture (React SPA)

#### Component Hierarchy

```
App.jsx (Root)
├── SpaceBackground.jsx (Three.js canvas - always rendered)
├── Header
│   ├── Logo
│   ├── Navigation tabs
│   └── Menu toggle
│
├── Routes (tab-based, no React Router)
│   ├── Observation Tab
│   │   ├── ObservationTab.jsx (container)
│   │   ├── InputArea.jsx (image upload + prompt)
│   │   └── Response display (AI results)
│   │
│   ├── Community Tab
│   │   ├── dao.jsx (main feed)
│   │   ├── Post creation form
│   │   ├── Post cards (with nested comments)
│   │   └── Image previews
│   │
│   ├── DAO Dashboard Tab
│   │   ├── daoDashboard.jsx (container)
│   │   ├── WalletContext (provider)
│   │   ├── ProposalCard.jsx (voting UI)
│   │   ├── CreateProposalForm.jsx (modal)
│   │   └── UserProfile.jsx (sidebar)
│   │
│   ├── Space Lab Tab
│   │   ├── SpaceSimulation.jsx (Three.js scene)
│   │   ├── useHandTracking hook (MediaPipe)
│   │   └── Shape selector
│   │
│   └── Playground Tab
│       ├── Playground.jsx (Three.js text morphing)
│       └── Console controls (color, size, image upload)
│
├── Profile Modal
│   └── profile.jsx (user stats, edit form)
│
└── SignIn Modal
    └── signin.jsx (Twitter OAuth)
```

#### State Management Strategy

**No Redux/Zustand** — Uses React's built-in state management:

- **Local component state** (`useState`) for UI interactions
- **Context API** for cross-cutting concerns:
  - `WalletContext` (wallet connection, provider, account)
- **Props drilling** for parent-child communication
- **Refs** (`useRef`) for:
  - File inputs
  - Canvas elements
  - User ID persistence (localStorage)

#### Key React Patterns Used

```jsx
// 1. Custom hooks for complex logic
const { handTrackingEnabled, handStatus, toggleHandTracking } = useHandTracking();

// 2. useCallback for stable function references (avoid re-renders)
const loadProposals = useCallback(async () => {
  // Expensive blockchain read
}, [provider]); // Only recreate if provider changes

// 3. useEffect with cleanup for subscriptions
useEffect(() => {
  const subscription = subscribeToPostsChannel(() => loadPosts());
  return () => subscription.unsubscribe(); // Cleanup on unmount
}, []);

// 4. Controlled components for forms
<input 
  value={formData.title} 
  onChange={e => setFormData(prev => ({ ...prev, title: e.target.value }))}
/>
```

---

### 2. Backend Architecture (Node.js/Express)

#### Server Structure

```javascript
// server.js - Single-file architecture (603 lines)

// 1. Imports & Configuration
const express = require('express');
const cors = require('cors');
const multer = require('multer');
const { createClient } = require('@supabase/supabase-js');

// 2. Lazy initialization patterns
let _supabase = null;
function getSupabase() {
  if (_supabase) return _supabase;
  // Initialize only when first called
  _supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_SERVICE_KEY);
  return _supabase;
}

// 3. Middleware setup
app.use(cors({ origin: process.env.FRONTEND_URL }));
app.use(express.json({ limit: '10mb' }));

// 4. Route groups
// - AI Services (/api/chat, /api/identify)
// - Astrometry (/api/analyze-discovery)
// - DAO (/api/dao/posts, /api/dao/comments)
// - Profiles (/api/users)
// - Health (/health, /)

// 5. Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});
```

#### Route Organization

```
GET    /                       → Health check + API info
GET    /health                 → Uptime status

POST   /api/chat               → AI chat (AstroSage-8B)
POST   /api/identify           → Vision AI (Kimi-K2.5)
POST   /api/analyze-discovery  → Full astrometry pipeline

GET    /api/dao/posts          → Fetch all posts
POST   /api/dao/posts          → Create post (with image upload)
POST   /api/dao/posts/:id/like → Toggle post like
GET    /api/dao/posts/:id/comments → Fetch nested comments
POST   /api/dao/posts/:id/comments → Create comment/reply
POST   /api/dao/comments/:id/like  → Toggle comment like

GET    /api/users              → List all users
GET    /api/users/:userId      → Get user profile
PUT    /api/users/:userId      → Update profile
```

#### Key Backend Patterns

**1. Lazy Resource Initialization**
```javascript
// Prevents server crash if env vars missing
let _supabase = null;
function getSupabase() {
  if (_supabase) return _supabase;
  if (!process.env.SUPABASE_URL) throw new Error('Missing SUPABASE_URL');
  _supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_SERVICE_KEY);
  return _supabase;
}
```

**2. Memory Storage → Stream Upload**
```javascript
const upload = multer({ storage: multer.memoryStorage(), limits: { fileSize: 5 * 1024 * 1024 } });

app.post('/api/dao/posts', upload.single('image'), async (req, res) => {
  if (req.file) {
    const filePath = `${Date.now()}-${Math.random().toString(36).slice(2)}.${ext}`;
    await getSupabase().storage
      .from('dao-images')
      .upload(filePath, req.file.buffer, { contentType: req.file.mimetype });
  }
});
```

**3. Error Response Standardization**
```javascript
try {
  // Operation
  res.json({ success: true, data });
} catch (err) {
  console.error('Operation failed:', err);
  res.status(500).json({ success: false, error: err.message });
}
```

---

### 3. Smart Contract Architecture (Solidity)

#### Contract Inheritance Hierarchy

```
AstroDAO
├── Ownable (OpenZeppelin)
│   └── Owner-only admin functions
├── ReentrancyGuard (OpenZeppelin)
│   └── Prevents reentrancy attacks
└── Pausable (OpenZeppelin)
    └── Emergency pause mechanism
```

#### State Variables Layout

```solidity
// Governance parameters (mutable by owner)
uint256 public votingPeriod = 7 days;
uint256 public quickVotePeriod = 3 days;
uint256 public executionDelay = 2 days;
uint256 public minVotesRequired = 5;
uint256 public quorumPercentage = 10;
uint256 public proposalThreshold = 3;

// Security limits (immutable)
uint256 public constant MAX_REPUTATION = 10000;

// User data
mapping(address => uint256) public userReputation;
mapping(address => uint256) public userVoteCount;
mapping(address => address) public delegates;

// Proposal storage
mapping(uint256 => Proposal) public proposals;
uint256 public proposalCount;

// Voter tracking
mapping(address => bool) public isRegisteredVoter;
uint256 public totalVoterCount;
```

#### Gas Optimization Techniques

**1. Tight Struct Packing**
```solidity
struct Proposal {
    uint256 id;                // 32 bytes
    address proposer;          // 20 bytes
    ProposalType proposalType; // 1 byte (enum stored as uint8)
    // ... strings stored in separate slots
    uint256 votesFor;
    uint256 votesAgainst;
    uint256 votesAbstain;
    uint256 endTime;
    uint256 executionTime;
    ProposalStatus status;     // 1 byte
    bool executed;             // 1 byte (same slot as status)
}
```

**2. Custom Errors vs require strings**
```solidity
// ❌ Old way (expensive)
require(userReputation[msg.sender] >= 3, "Insufficient reputation");

// ✅ New way (saves ~50 gas)
error InsufficientReputation();
if (userReputation[msg.sender] < 3) revert InsufficientReputation();
```

**3. Calldata vs Memory for Read-Only Params**
```solidity
// ✅ Cheaper for external functions (no copy to memory)
function createProposal(
    ProposalType _type,
    string calldata _title,      // ← calldata
    string calldata _description // ← calldata
) external { }
```

---

## 🔄 Data Flow

### 1. Observation Pipeline

```
User uploads image
        ↓
Frontend compresses (Jimp)
        ↓
POST /api/identify
        ↓
Backend forwards to HuggingFace Kimi-K2.5
        ↓
Response: "This is the Andromeda Galaxy (M31)"
        ↓
Display in ObservationTab
        ↓
User clicks "Run Discovery Pipeline"
        ↓
POST /api/analyze-discovery
        ↓
Backend executes:
   1. Astrometry.net login
   2. Image upload
   3. Coordinate solving (RA/Dec)
   4. NASA SkyView historical fetch
   5. Pixel-diff anomaly detection
        ↓
Response: { coords, discovery, type, score }
        ↓
Display coordinates + anomaly alert
```

### 2. Community Post Flow

```
User writes post + uploads image
        ↓
FormData: { text, userId, author, image }
        ↓
POST /api/dao/posts (multipart/form-data)
        ↓
Multer intercepts file
        ↓
Backend uploads to Supabase Storage
        ↓
Get public URL
        ↓
Insert into Supabase DB:
   INSERT INTO posts (user_id, author, text, image, likes, liked_by)
        ↓
Response: { success: true, post }
        ↓
Frontend adds to local state
        ↓
8-second polling re-fetches all posts
        ↓
New post appears in feed
```

### 3. DAO Vote Flow

```
User clicks "For" button on proposal
        ↓
Frontend calls contract.vote(proposalId, 1)
        ↓
Wallet prompts for signature
        ↓
Transaction sent to BNB Chain
        ↓
Smart contract checks:
   - Has user already voted? → revert AlreadyVoted()
   - Is user delegating? → revert AlreadyDelegated()
   - Is voting still active? → revert ProposalNotActive()
        ↓
Record vote:
   proposal.hasVoted[msg.sender] = true
   proposal.votesFor++
        ↓
Award reputation:
   userReputation[msg.sender] += 1
        ↓
Emit VoteCast event
        ↓
Frontend waits for tx.wait()
        ↓
Reload proposals from contract
        ↓
Updated vote count displays
```

### 4. Nested Comment Thread Flow

```
User clicks "Reply" on a comment
        ↓
Reply form appears inline
        ↓
POST /api/dao/posts/:postId/comments
Body: { userId, author, text, parentId: commentId }
        ↓
Backend inserts:
   INSERT INTO comments (post_id, parent_id, user_id, author, text)
        ↓
Response: { success: true, comment }
        ↓
GET /api/dao/posts/:postId/comments
        ↓
Backend fetches all comments for post
        ↓
buildNestedComments() helper:
   - Creates hash map by comment ID
   - Iterates and nests children under parents
   - Returns tree structure
        ↓
Frontend renders recursively:
   Comment
     └── Reply
          └── Reply to reply
               └── ...
```

---

## 💻 Technology Deep Dive

### Frontend Technologies

#### Three.js Implementation

**SpaceBackground.jsx**
```javascript
useEffect(() => {
  // Scene setup
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  
  // Starfield
  const starsGeometry = new THREE.BufferGeometry();
  const starsMaterial = new THREE.PointsMaterial({ color: 0xffffff, size: 0.7 });
  const starVertices = [];
  for (let i = 0; i < 10000; i++) {
    const x = (Math.random() - 0.5) * 2000;
    const y = (Math.random() - 0.5) * 2000;
    const z = (Math.random() - 0.5) * 2000;
    starVertices.push(x, y, z);
  }
  starsGeometry.setAttribute('position', new THREE.Float32BufferAttribute(starVertices, 3));
  const stars = new THREE.Points(starsGeometry, starsMaterial);
  scene.add(stars);
  
  // Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    stars.rotation.y += 0.0002;
    renderer.render(scene, camera);
  };
  animate();
  
  // Cleanup
  return () => {
    renderer.dispose();
    starsGeometry.dispose();
    starsMaterial.dispose();
  };
}, []);
```

**Playground Text Morphing**
```javascript
// Text → 3D particles
const loader = new THREE.FontLoader();
loader.load('/fonts/helvetiker_regular.typeface.json', (font) => {
  const textGeometry = new THREE.TextGeometry(text, {
    font: font,
    size: size,
    height: 5,
  });
  
  // Sample points from geometry
  const vertices = [];
  const positions = textGeometry.attributes.position.array;
  for (let i = 0; i < positions.length; i += 3) {
    vertices.push(new THREE.Vector3(positions[i], positions[i+1], positions[i+2]));
  }
  
  // Create particles
  const particleGeometry = new THREE.BufferGeometry().setFromPoints(vertices);
  const particleMaterial = new THREE.PointsMaterial({ color, size: 0.5 });
  const particles = new THREE.Points(particleGeometry, particleMaterial);
  scene.add(particles);
  
  // Animate dispersion
  const animate = () => {
    particles.rotation.y += 0.01;
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  };
  animate();
});
```

#### MediaPipe Hand Tracking

**useHandTracking.js**
```javascript
import { Hands, HAND_CONNECTIONS } from '@mediapipe/hands';
import { Camera } from '@mediapipe/camera_utils';

export function useHandTracking() {
  const [handStatus, setHandStatus] = useState({ handCount: 0, scale: 1 });
  
  useEffect(() => {
    const hands = new Hands({
      locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
    });
    
    hands.setOptions({
      maxNumHands: 2,
      modelComplexity: 1,
      minDetectionConfidence: 0.5,
      minTrackingConfidence: 0.5
    });
    
    hands.onResults((results) => {
      if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
        const hand = results.multiHandLandmarks[0];
        
        // Calculate hand size (distance between wrist and middle finger tip)
        const wrist = hand[0];
        const middleTip = hand[12];
        const distance = Math.sqrt(
          Math.pow(middleTip.x - wrist.x, 2) + 
          Math.pow(middleTip.y - wrist.y, 2)
        );
        
        // Map distance to scale (closer hand = larger scale)
        const scale = 1 + (distance * 5);
        
        setHandStatus({ handCount: results.multiHandLandmarks.length, scale });
      } else {
        setHandStatus({ handCount: 0, scale: 1 });
      }
    });
    
    const camera = new Camera(videoRef.current, {
      onFrame: async () => {
        await hands.send({ image: videoRef.current });
      },
      width: 640,
      height: 480
    });
    camera.start();
    
    return () => camera.stop();
  }, []);
  
  return { handStatus, handTrackingEnabled, toggleHandTracking };
}
```

#### EIP-6963 Multi-Wallet Discovery

**WalletContext.jsx**
```javascript
function getInjectedProviders() {
  return new Promise((resolve) => {
    const providers = [];
    const seen = new Set();
    
    const handler = (e) => {
      const { info, provider } = e.detail;
      if (!seen.has(info.uuid)) {
        seen.add(info.uuid);
        providers.push({ info, provider });
      }
    };
    
    // Listen for wallet announcements
    window.addEventListener('eip6963:announceProvider', handler);
    window.dispatchEvent(new Event('eip6963:requestProvider'));
    
    // Wait 300ms for all wallets to respond
    setTimeout(() => {
      window.removeEventListener('eip6963:announceProvider', handler);
      
      // Fallback to window.ethereum if no EIP-6963 wallets
      if (providers.length === 0 && window.ethereum) {
        providers.push({
          info: { name: 'Browser Wallet', icon: '', uuid: 'injected' },
          provider: window.ethereum
        });
      }
      
      resolve(providers);
    }, 300);
  });
}
```

### Backend Technologies

#### Astrometry.net Pipeline

**Complete workflow:**
```javascript
async function analyzeDiscovery(imageBase64) {
  // 1. Login to Astrometry.net
  const sessionKey = await axios.post('http://nova.astrometry.net/api/login', {
    request-json: JSON.stringify({ apikey: process.env.ASTROMETRY_API_KEY })
  }).then(res => res.data.session);
  
  // 2. Upload image
  const uploadRes = await axios.post('http://nova.astrometry.net/api/upload', {
    request-json: JSON.stringify({
      session: sessionKey,
      allow_commercial_use: 'd',
      allow_modifications: 'd',
      publicly_visible: 'n'
    }),
    file: Buffer.from(imageBase64, 'base64')
  }, { headers: { 'Content-Type': 'multipart/form-data' } });
  
  const subid = uploadRes.data.subid;
  
  // 3. Poll for results
  let jobId = null;
  for (let i = 0; i < 30; i++) {
    const statusRes = await axios.get(`http://nova.astrometry.net/api/submissions/${subid}`);
    if (statusRes.data.jobs && statusRes.data.jobs.length > 0) {
      jobId = statusRes.data.jobs[0];
      break;
    }
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  // 4. Get calibration
  const calibRes = await axios.get(`http://nova.astrometry.net/api/jobs/${jobId}/calibration/`);
  const { ra, dec } = calibRes.data;
  
  // 5. Fetch historical image from NASA SkyView
  const skyviewUrl = `https://skyview.gsfc.nasa.gov/cgi-bin/images?Survey=DSS&position=${ra},${dec}&Size=0.5&Pixels=500&Return=JPEG`;
  const nasaImage = await axios.get(skyviewUrl, { responseType: 'arraybuffer' });
  
  // 6. Pixel-diff anomaly detection
  const userImg = await Jimp.read(Buffer.from(imageBase64, 'base64'));
  const nasaImg = await Jimp.read(Buffer.from(nasaImage.data));
  
  userImg.resize(500, 500).greyscale();
  nasaImg.resize(500, 500).greyscale();
  
  const diffBuffer = Buffer.alloc(500 * 500 * 4);
  const diffPixels = pixelmatch(
    userImg.bitmap.data,
    nasaImg.bitmap.data,
    diffBuffer,
    500,
    500,
    { threshold: 0.15 }
  );
  
  const anomalyScore = (diffPixels / (500 * 500)) * 100;
  
  // 7. Classify discovery
  const discovery = anomalyScore > 5 
    ? `Potential SUPERNOVA detected at RA: ${ra}, Dec: ${dec}` 
    : `Coordinates: RA: ${ra}, Dec: ${dec}`;
  
  return {
    coords: { ra, dec },
    historicalImage: nasaImage.data.toString('base64'),
    discovery,
    type: anomalyScore > 5 ? 'SUPERNOVA' : 'GALAXY',
    rawScore: anomalyScore
  };
}
```

#### Supabase Integration

**Nested comments builder:**
```javascript
function buildNestedComments(rows) {
  const map = {};
  const roots = [];
  
  // Build hash map
  rows.forEach(comment => {
    map[comment.id] = {
      ...comment,
      replies: [],
      timeString: formatTimeString(comment.created_at)
    };
  });
  
  // Nest children under parents
  rows.forEach(comment => {
    if (comment.parent_id && map[comment.parent_id]) {
      map[comment.parent_id].replies.push(map[comment.id]);
    } else {
      roots.push(map[comment.id]);
    }
  });
  
  return roots;
}
```

---

## 🚀 Setup Guide

### Prerequisites

```bash
# Check versions
node --version   # v18+ required
npm --version    # v9+ required
git --version    # v2.30+ required
```

### 1. Clone & Install

```bash
# Clone repository
git clone https://github.com/yourusername/astrovision.git
cd astrovision

# Backend setup
cd backend
npm install

# Frontend setup
cd ../frontend
npm install
```

### 2. Environment Configuration

#### Backend `.env`

```bash
# Server
PORT=3001

# Supabase (get from supabase.com dashboard)
SUPABASE_URL=https://xxxxxxxxxxx.supabase.co
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3...  # SERVICE ROLE KEY

# HuggingFace (free tier: https://huggingface.co/settings/tokens)
HF_API_KEY=hf_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Astrometry.net (free: http://nova.astrometry.net/api_help)
ASTROMETRY_API_KEY=xxxxxxxxxxxxxxxx

# Twitter OAuth (optional - https://developer.twitter.com)
TWITTER_CLIENT_ID=your_client_id
TWITTER_CLIENT_SECRET=your_client_secret

# CORS
FRONTEND_URL=http://localhost:3000
```

#### Frontend `.env`

```bash
# Backend URL (no trailing slash)
REACT_APP_API_URL=http://localhost:3001

# Smart contract address (deploy first)
REACT_APP_DAO_CONTRACT_ADDRESS=0x0000000000000000000000000000000000000000
```

### 3. Database Setup (Supabase)

```sql
-- 1. Create posts table
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT NOT NULL,
  author TEXT NOT NULL,
  text TEXT,
  image TEXT,
  likes INTEGER DEFAULT 0,
  liked_by TEXT[] DEFAULT ARRAY[]::TEXT[],
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_posts_created ON posts(created_at DESC);
CREATE INDEX idx_posts_user ON posts(user_id);

-- 2. Create comments table
CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
  user_id TEXT NOT NULL,
  author TEXT NOT NULL,
  text TEXT NOT NULL,
  likes INTEGER DEFAULT 0,
  liked_by TEXT[] DEFAULT ARRAY[]::TEXT[],
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_comments_post ON comments(post_id, created_at);
CREATE INDEX idx_comments_parent ON comments(parent_id);

-- 3. Enable Row Level Security (optional)
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

-- Allow public read
CREATE POLICY "Allow public read posts" ON posts FOR SELECT USING (true);
CREATE POLICY "Allow public read comments" ON comments FOR SELECT USING (true);

-- Allow authenticated insert (if using Supabase Auth)
CREATE POLICY "Allow authenticated insert posts" ON posts FOR INSERT WITH CHECK (true);
CREATE POLICY "Allow authenticated insert comments" ON comments FOR INSERT WITH CHECK (true);
```

### 4. Storage Setup (Supabase)

```bash
# In Supabase Dashboard:
# 1. Go to Storage → Create bucket
# 2. Bucket name: dao-images
# 3. Public: Enabled
# 4. File size limit: 5MB
# 5. Allowed MIME types: image/jpeg, image/png, image/webp
```

### 5. Smart Contract Deployment

```bash
# Install Hardhat
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox

# Create hardhat.config.js
cat > hardhat.config.js << 'EOF'
require("@nomicfoundation/hardhat-toolbox");
require('dotenv').config();

module.exports = {
  solidity: "0.8.20",
  networks: {
    bscTestnet: {
      url: "https://data-seed-prebsc-1-s1.binance.org:8545",
      chainId: 97,
      accounts: [process.env.PRIVATE_KEY]
    }
  }
};
EOF

# Deploy script
mkdir -p scripts
cat > scripts/deploy.js << 'EOF'
const hre = require("hardhat");

async function main() {
  const AstroDAO = await hre.ethers.getContractFactory("AstroDAO");
  const astrodao = await AstroDAO.deploy();
  await astrodao.waitForDeployment();
  
  console.log("AstroDAO deployed to:", await astrodao.getAddress());
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
EOF

# Deploy
npx hardhat run scripts/deploy.js --network bscTestnet

# Update frontend .env with contract address
```

### 6. Run Development Servers

```bash
# Terminal 1: Backend
cd backend
npm start
# Server running on http://localhost:3001

# Terminal 2: Frontend
cd frontend
npm start
# App running on http://localhost:3000

# Terminal 3: Blockchain (optional - if testing contract changes)
npx hardhat node
```

### 7. Verify Setup

```bash
# Test backend health
curl http://localhost:3001/health

# Expected response:
{
  "status": "ok",
  "version": "4.0.0",
  "uptime": 123,
  "supabase": "✓",
  "dao_endpoints": ["/api/dao/posts", ...]
}

# Test frontend (open browser)
open http://localhost:3000
```

---

## 🌐 Deployment

### Backend Deployment (Render)

**render.yaml** (auto-deploy on push):
```yaml
services:
  - type: web
    name: astrovision-backend
    env: node
    buildCommand: npm install
    startCommand: npm start
    envVars:
      - key: NODE_ENV
        value: production
      - key: PORT
        value: 3001
      - key: SUPABASE_URL
        sync: false
      - key: SUPABASE_SERVICE_KEY
        sync: false
      - key: HF_API_KEY
        sync: false
      - key: ASTROMETRY_API_KEY
        sync: false
      - key: FRONTEND_URL
        value: https://your-app.vercel.app
```

**Manual Render Setup:**
1. Go to [render.com](https://render.com)
2. New → Web Service
3. Connect GitHub repo
4. Build command: `npm install`
5. Start command: `npm start`
6. Add environment variables from backend `.env`
7. Deploy

### Frontend Deployment (Vercel)

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
cd frontend
vercel

# Add environment variables in Vercel dashboard:
# REACT_APP_API_URL=https://your-backend.onrender.com
# REACT_APP_DAO_CONTRACT_ADDRESS=0x...

# Production deployment
vercel --prod
```

### Smart Contract Deployment (BNB Mainnet)

```bash
# Update hardhat.config.js
networks: {
  bscMainnet: {
    url: "https://bsc-dataseed.binance.org/",
    chainId: 56,
    accounts: [process.env.PRIVATE_KEY]
  }
}

# Deploy
npx hardhat run scripts/deploy.js --network bscMainnet

# Verify on BscScan
npx hardhat verify --network bscMainnet DEPLOYED_CONTRACT_ADDRESS
```

---

## 📡 API Documentation

See full API reference in main [README.md](#api-reference)

**Quick Reference:**

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/api/chat` | POST | None | AI chat |
| `/api/identify` | POST | None | Vision ID |
| `/api/analyze-discovery` | POST | None | Full pipeline |
| `/api/dao/posts` | GET | None | Fetch posts |
| `/api/dao/posts` | POST | None | Create post |
| `/api/dao/posts/:id/like` | POST | None | Toggle like |
| `/api/dao/posts/:id/comments` | GET | None | Get comments |
| `/api/dao/posts/:id/comments` | POST | None | Add comment |

---

## 📜 Smart Contract Details

### Key Functions

```solidity
// Create proposal (requires 3+ reputation)
function createProposal(
    ProposalType _type,        // 0-4
    string calldata _title,
    string calldata _description,
    string calldata _ipfsHash,
    bool _quickVote            // 3 days vs 7 days
) external returns (uint256 proposalId)

// Vote (earns +1 reputation)
function vote(
    uint256 _proposalId,
    VoteChoice _choice         // 0=AGAINST, 1=FOR, 2=ABSTAIN
) external

// Finalize after voting ends
function finalizeProposal(uint256 _proposalId) external

// Execute after time lock (earns +5 reputation for proposer)
function executeProposal(uint256 _proposalId) external

// Delegate voting power
function delegate(address _delegatee) external
function undelegate() external
```

### Gas Costs (BNB Testnet)

| Operation | Gas Used | Cost @ 3 Gwei |
|-----------|----------|---------------|
| Create Proposal | ~150,000 | $0.45 |
| Vote | ~80,000 | $0.24 |
| Finalize | ~60,000 | $0.18 |
| Execute | ~50,000 | $0.15 |
| Delegate | ~45,000 | $0.14 |

---

## ⚡ Performance & Scaling

### Current Bottlenecks

1. **8-second polling** for community posts (Supabase)
   - **Solution**: Migrate to WebSocket subscriptions (Phase 2)

2. **Loop-based proposal loading** (frontend)
   - **Solution**: Implement `getProposals(start, end)` contract function

3. **No pagination** on DAO feed
   - **Solution**: Implement infinite scroll + backend cursor pagination

4. **Image compression** done client-side
   - **Solution**: Move to backend + use Sharp instead of Jimp

### Scalability Targets

| Metric | Current | Q2 2026 | Q4 2026 |
|--------|---------|---------|---------|
| Concurrent users | 100 | 10,000 | 100,000 |
| Posts/day | 50 | 5,000 | 50,000 |
| Proposals/day | 5 | 500 | 5,000 |
| AI requests/day | 200 | 20,000 | 200,000 |

### Optimization Plan

**Database**:
- Add indexes on `created_at`, `user_id`, `post_id`
- Implement query result caching (Redis)
- Connection pooling (currently 1 connection)

**Backend**:
- Horizontal scaling (multiple Render instances)
- CDN for static assets (Cloudflare)
- Rate limiting per IP (express-rate-limit)

**Frontend**:
- Code splitting (React.lazy + Suspense)
- Image lazy loading (react-lazyload)
- Virtualized lists for long feeds (react-window)

**Blockchain**:
- Migrate to opBNB for 10x lower gas costs
- Batch proposal reads (multicall contract)

---

## 🔒 Security

### Backend Security

**1. Environment Variable Protection**
```javascript
// ❌ Never do this
const apiKey = "hf_123456789";

// ✅ Always use env vars
const apiKey = process.env.HF_API_KEY;
if (!apiKey) throw new Error('Missing HF_API_KEY');
```

**2. Input Validation**
```javascript
// Validate image uploads
const upload = multer({
  limits: { fileSize: 5 * 1024 * 1024 }, // 5 MB max
  fileFilter: (req, file, cb) => {
    if (!file.mimetype.startsWith('image/')) {
      return cb(new Error('Only images allowed'), false);
    }
    cb(null, true);
  }
});
```

**3. SQL Injection Prevention**
```javascript
// ✅ Supabase uses parameterized queries by default
await supabase.from('posts').select('*').eq('id', postId);

// ❌ Never build raw SQL strings
const query = `SELECT * FROM posts WHERE id = '${postId}'`; // VULNERABLE
```

**4. CORS Configuration**
```javascript
app.use(cors({
  origin: process.env.FRONTEND_URL, // Whitelist only your frontend
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true
}));
```

### Smart Contract Security

**1. Reentrancy Protection**
```solidity
// ✅ Uses OpenZeppelin ReentrancyGuard
function vote(uint256 _proposalId, VoteChoice _choice) external nonReentrant {
  // Safe from reentrancy attacks
}
```

**2. Access Control**
```solidity
// ✅ Owner-only functions
function awardReputation(address _user, uint256 _amount) external onlyOwner {
  _earnReputation(_user, _amount, "Awarded by admin");
}

// ✅ Proposer-or-owner execution
function executeProposal(uint256 _proposalId) external {
  if (msg.sender != proposal.proposer && msg.sender != owner()) {
    require(block.timestamp >= proposal.executionTime + 1 days);
  }
}
```

**3. Integer Overflow Prevention**
```solidity
// ✅ Solidity 0.8+ has built-in overflow checks
userReputation[msg.sender] += 1; // Reverts on overflow

// ✅ Additional cap check
function _earnReputation(address _user, uint256 _amount, string memory _reason) internal {
  uint256 newRep = userReputation[_user] + _amount;
  if (newRep > MAX_REPUTATION) {
    newRep = MAX_REPUTATION;
  }
  userReputation[_user] = newRep;
}
```

**4. Emergency Controls**
```solidity
// ✅ Pausable pattern
function createProposal(...) external whenNotPaused {
  // Can be paused in emergency
}

function pause() external onlyOwner {
  _pause();
}
```

### Frontend Security

**1. XSS Prevention**
```jsx
// ✅ React escapes by default
<p>{userInput}</p>  // Safe

// ❌ Dangerous: dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />  // VULNERABLE
```

**2. Private Key Storage**
```javascript
// ❌ Never store private keys
localStorage.setItem('privateKey', key); // NEVER DO THIS

// ✅ Let wallet handle keys
const signer = await provider.getSigner(); // Wallet prompts user
```

**3. Transaction Verification**
```javascript
// ✅ Always verify contract address
if (!ethers.isAddress(contractAddress)) {
  throw new Error('Invalid contract address');
}

// ✅ Always wait for confirmation
const tx = await contract.vote(proposalId, choice);
await tx.wait(); // Wait for block confirmation
```

---

## 🐛 Troubleshooting

### Common Issues

#### Backend won't start
```bash
# Error: SUPABASE_URL not defined
# Solution: Check .env file exists and has correct format
cd backend
cat .env  # Verify variables present
source .env  # Load into shell (Linux/Mac)
npm start
```

#### Frontend can't connect to backend
```bash
# Error: Network Error / CORS
# Solution: Check FRONTEND_URL in backend .env
# Backend .env should have:
FRONTEND_URL=http://localhost:3000

# Frontend .env should have:
REACT_APP_API_URL=http://localhost:3001
```

#### Wallet won't connect
```javascript
// Error: "No wallet detected"
// Solution: Install MetaMask or check EIP-6963 support
// In browser console:
window.ethereum  // Should return object, not undefined
```

#### Contract calls fail
```javascript
// Error: "Contract address not configured"
// Solution: Deploy contract first, then update .env
// Frontend .env:
REACT_APP_DAO_CONTRACT_ADDRESS=0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb  // Your deployed address
```

#### Supabase queries fail
```bash
# Error: "Invalid API key"
# Solution: Use SERVICE ROLE key, not ANON key
# In Supabase Dashboard → Settings → API
# Copy the "service_role" key (starts with eyJhbG...)
```

### Debug Mode

**Enable verbose logging:**

Backend:
```javascript
// Add to server.js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});
```

Frontend:
```javascript
// In WalletContext.jsx
console.log('Wallet providers found:', providers);
console.log('Connecting to:', wallet.info.name);
```

Smart Contract:
```solidity
// Add events for debugging
event Debug(string message, uint256 value);
emit Debug("Vote count", proposal.votesFor);
```

---

## 📞 Support

### Resources
- **Documentation**: [docs.astrovision.app](https://docs.astrovision.app)
- **GitHub**: [github.com/astrovision](https://github.com/astrovision)
- **Discord**: [discord.gg/astrovision](https://discord.gg/astrovision)

### Reporting Bugs
```markdown
**Bug Report Template:**

**Environment:**
- OS: [e.g., Windows 11, macOS 13, Ubuntu 22.04]
- Browser: [e.g., Chrome 120, Firefox 121]
- Node version: [e.g., v18.17.0]
- Network: [e.g., BNB Testnet]

**Steps to Reproduce:**
1. Go to '...'
2. Click on '...'
3. See error

**Expected Behavior:**
[What should happen]

**Actual Behavior:**
[What actually happens]

**Screenshots:**
[If applicable]

**Console Errors:**
[Browser console or terminal output]
```

---

**Last Updated**: February 2026  
**Maintained by**: AstroVision Core Team  
**License**: MIT