 # AstroVision

**Decentralized Space Exploration, Governed by Its Community**

AstroVision is a full-stack Web3 application that bridges amateur astronomy, AI-powered analysis, and on-chain governance. Built on BNB Chain during the 2026 Hackathon.

---

## 📖 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Smart Contract](#smart-contract)
- [API Reference](#api-reference)
- [Contributing](#contributing)
- [Roadmap](#roadmap)

---

## 🎯 Overview

### The Problem
Space science is locked inside expensive institutions. Amateur astronomers lack structured ways to share discoveries, vote on research priorities, or contribute to a broader mission. Meanwhile, Web3 DAOs suffer from low participation because governance feels abstract and disconnected from real-world impact.

### The Solution
AstroVision creates a complete workflow: **Observe → Analyze → Share → Govern**

1. **Observe**: Upload telescope images
2. **Analyze**: AI identifies objects + Astrometry solves coordinates + NASA cross-reference detects anomalies
3. **Share**: Post discoveries to a real-time community board (Supabase-backed)
4. **Govern**: Vote on research proposals via on-chain DAO with reputation weighting

---

## ✨ Key Features

### 🔭 Observation & Analysis
- **AI Vision ID**: Upload images → Get instant celestial object identification (AstroSage-8B + Kimi-K2.5)
- **Astrometry Pipeline**: Full coordinate solving via Astrometry.net API
- **Anomaly Detection**: Pixel-diff comparison against NASA SkyView historical data
- **Discovery Records**: RA/Dec coordinates, anomaly scores, object classifications

### 🌐 Community Board (DAO Feed)
- **Real-time Posts**: Supabase-backed feed with 8-second polling
- **Nested Comments**: Threaded discussions with infinite reply depth
- **Image Attachments**: Upload to Supabase Storage, display inline
- **Likes System**: Track engagement across posts and comments
- **User Profiles**: Avatars, bio, post/comment counters

### 🏛️ On-Chain Governance (AstroDAO)
- **Smart Contract**: Deployed on BNB Chain (opBNB-ready)
- **Proposal Types**: Weekly Theme, Research Discovery, Community, Knowledge Sharing, Collaboration
- **Reputation System**: Earn via voting (+1), executing proposals (+5), capped at 10,000
- **Voting Delegation**: Delegate voting power to trusted addresses
- **Multi-Wallet Support**: EIP-6963 integration (MetaMask, Coinbase, Brave, Rabby, any injected wallet)
- **Time-Locked Execution**: 2-day delay after voting, proposer can execute early

### 🎮 Interactive Features
- **Space Lab**: Three.js 3D simulation with MediaPipe hand tracking (control space objects with physical gestures)
- **Playground**: Text morphing canvas with real-time color/size controls
- **Twitter OAuth**: Social login with on-chain identity mapping

---

## 🏗️ Architecture

### High-Level Flow

```
┌──────────────┐
│   Browser    │
│  (React SPA) │
└──────┬───────┘
       │
       ├─────────────────────┐
       │                     │
       ▼                     ▼
┌─────────────┐      ┌─────────────┐
│   Node.js   │      │  BNB Chain  │
│   Backend   │      │   (Web3)    │
│  (Render)   │      │             │
└──────┬──────┘      └─────────────┘
       │
       ├──────────────┬──────────────┐
       │              │              │
       ▼              ▼              ▼
┌──────────┐   ┌──────────┐   ┌──────────┐
│ Supabase │   │HuggingFace│   │Astrometry│
│   (DB)   │   │   (AI)    │   │  (Coords)│
└──────────┘   └──────────┘   └──────────┘
```

### Component Architecture

```
Frontend (React)
├── Observation Tab
│   ├── Image Upload
│   ├── AI Analysis
│   └── Discovery Pipeline
├── Community Tab (DAO Feed)
│   ├── Post Creation
│   ├── Nested Comments
│   └── Like System
├── DAO Dashboard
│   ├── Proposal Management
│   ├── Voting Interface
│   └── User Profile Sidebar
├── Space Lab
│   ├── 3D Simulation (Three.js)
│   └── Hand Tracking (MediaPipe)
└── Playground
    └── Text Morphing Canvas

Backend (Node.js/Express)
├── AI Services
│   ├── Vision ID (Kimi-K2.5)
│   └── Chat (AstroSage-8B)
├── Astrometry Pipeline
│   ├── Login → Upload → Solve
│   └── NASA SkyView Cross-Reference
├── DAO API (6 Supabase endpoints)
│   ├── Posts (GET/POST)
│   ├── Likes (POST)
│   └── Comments (GET/POST, nested)
├── User Profiles
│   └── Twitter OAuth
└── Image Storage
    └── Supabase Storage

Smart Contract (Solidity)
├── Proposal Management
│   ├── Create/Vote/Finalize/Execute
│   └── 5 Proposal Types
├── Reputation System
│   ├── Voting rewards
│   └── Execution rewards
├── Delegation
│   ├── Delegate voting power
│   └── Undelegate
└── Security
    ├── ReentrancyGuard
    ├── Pausable
    └── Reputation cap (10,000)
```

---

## 🛠️ Tech Stack

### Frontend
| Technology | Purpose |
|------------|---------|
| React 18 | UI framework |
| Three.js | 3D space simulation |
| MediaPipe | Hand tracking for Space Lab |
| ethers.js | Blockchain interaction |
| react-icons (Material Design) | Icon system |

### Backend
| Technology | Purpose |
|------------|---------|
| Node.js + Express | REST API server |
| Supabase | PostgreSQL database + storage |
| Multer | File upload handling (images) |
| Axios | HTTP client for external APIs |
| Jimp + Pixelmatch | Image processing + anomaly detection |
| dotenv | Environment configuration |

### Blockchain
| Technology | Purpose |
|------------|---------|
| Solidity 0.8.20 | Smart contract language |
| OpenZeppelin | Security libraries (Ownable, ReentrancyGuard, Pausable) |
| BNB Chain | Deployment network |
| EIP-6963 | Multi-wallet discovery standard |

### External APIs
| Service | Purpose |
|---------|---------|
| HuggingFace | AI models (AstroSage-8B, Kimi-K2.5) |
| Astrometry.net | RA/Dec coordinate solving |
| NASA SkyView | Historical sky survey images |
| Twitter OAuth | Social authentication |

### Infrastructure
| Service | Purpose |
|---------|---------|
| Render | Backend hosting |
| Supabase | Database + file storage |
| GitHub | Version control |
| BNB Testnet/Mainnet | Smart contract deployment |

---

## 📁 Project Structure

```
astrovision/
├── frontend/
│   ├── public/
│   │   ├── community.ico
│   │   └── index.html
│   ├── src/
│   │   ├── components/
│   │   │   ├── CreateProposalForm.jsx
│   │   │   ├── daoDashboard.jsx
│   │   │   ├── InputArea.jsx
│   │   │   ├── observationTab.jsx
│   │   │   ├── ProposalCard.jsx
│   │   │   ├── spaceBackground.jsx
│   │   │   ├── spaceSimulation.jsx
│   │   │   └── UserProfile.jsx
│   │   ├── context/
│   │   │   └── WalletContext.jsx          # EIP-6963 multi-wallet
│   │   ├── contracts/
│   │   │   └── AstroDAO-ABI.js            # Contract ABI
│   │   ├── hooks/
│   │   │   ├── useHandTracking.js         # MediaPipe integration
│   │   │   └── useSpaceSimulation.js
│   │   ├── pages/
│   │   │   ├── dao.jsx                    # Community board
│   │   │   ├── playground.jsx
│   │   │   ├── profile.jsx
│   │   │   └── signin.jsx
│   │   ├── services/
│   │   │   ├── aiServices.js              # HuggingFace APIs
│   │   │   ├── daoService.jsx             # Supabase DAO backend
│   │   │   └── imageServices.js
│   │   ├── utils/
│   │   │   ├── constants.js
│   │   │   └── helpers.js
│   │   ├── App.css
│   │   ├── App.jsx                        # Main app component
│   │   └── index.js
│   ├── .env
│   └── package.json
│
├── backend/
│   ├── server.js                          # Main Express server
│   ├── .env
│   └── package.json
│   |
    |- contracts/
│           └── AstroDAOSecure.sol                 # Solidity smart contract
│
├── docs/
│   ├── PROJECT.md              # problem, solution ...
│   ├── TECHNICAL.md             # Achietechture ,...
│                         
└── README.md
```

---

## 🚀 Installation

### Prerequisites
- Node.js 18+
- npm or yarn
- MetaMask or any Web3 wallet
- Supabase account (free tier)
- HuggingFace API key (free)
- Twitter Developer credentials (optional)

### 1. Clone Repository
```bash
git clone https://github.com/yourusername/astrovision.git
cd astrovision
```

### 2. Backend Setup
```bash
cd backend
npm install

# Create .env file
cat > .env << EOF
PORT=3001
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_KEY=your-service-role-key
HF_API_KEY=your-huggingface-key
ASTROMETRY_API_KEY=your-astrometry-key
TWITTER_CLIENT_ID=your-twitter-client-id
TWITTER_CLIENT_SECRET=your-twitter-secret
FRONTEND_URL=http://localhost:3000
EOF

npm start
```

### 3. Frontend Setup
```bash
cd frontend
npm install

# Create .env file
cat > .env << EOF
REACT_APP_API_URL=http://localhost:3001
REACT_APP_DAO_CONTRACT_ADDRESS=0xYourContractAddress
EOF

npm start
```

### 4. Smart Contract Deployment
```bash
# Install Hardhat/Truffle/Foundry (your choice)
cd contracts

# Deploy to BNB Testnet
npx hardhat run scripts/deploy.js --network bscTestnet

# Update frontend .env with contract address
```

---

## ⚙️ Configuration

### Supabase Setup

#### 1. Create Tables
```sql
-- Posts table
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

-- Comments table
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
```

#### 2. Create Storage Bucket
- Bucket name: `dao-images`
- Public access: Enabled
- File size limit: 5MB

#### 3. Enable RLS Policies (optional)
```sql
-- Allow read access to all
CREATE POLICY "Allow public read" ON posts FOR SELECT USING (true);
CREATE POLICY "Allow public read" ON comments FOR SELECT USING (true);

-- Allow insert for authenticated users (if using Supabase Auth)
CREATE POLICY "Allow authenticated insert" ON posts FOR INSERT WITH CHECK (true);
CREATE POLICY "Allow authenticated insert" ON comments FOR INSERT WITH CHECK (true);
```

### Environment Variables

#### Backend (.env)
| Variable | Description | Required |
|----------|-------------|----------|
| `PORT` | Server port | No (default: 3001) |
| `SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_SERVICE_KEY` | Service role key (not anon key) | Yes |
| `HF_API_KEY` | HuggingFace API key | Yes |
| `ASTROMETRY_API_KEY` | Astrometry.net key | Yes |
| `TWITTER_CLIENT_ID` | Twitter OAuth client ID | No |
| `TWITTER_CLIENT_SECRET` | Twitter OAuth secret | No |
| `FRONTEND_URL` | Frontend URL for CORS | Yes |

#### Frontend (.env)
| Variable | Description | Required |
|----------|-------------|----------|
| `REACT_APP_API_URL` | Backend URL (no trailing slash) | Yes |
| `REACT_APP_DAO_CONTRACT_ADDRESS` | Deployed contract address | Yes |

---

## 💻 Usage

### 1. Observation Tab
1. Click "Upload Image" button
2. Select telescope image (JPG/PNG)
3. Enter a prompt (e.g., "Identify this galaxy")
4. Click submit
5. AI returns object identification
6. Click "Run Discovery Pipeline" for full analysis (RA/Dec + anomaly detection)

### 2. Community Board
1. Navigate to "Community" tab
2. Write a post (text + optional image)
3. Click "Post" — appears in real-time feed
4. Click comments icon to add threaded replies
5. Like posts/comments by clicking heart icon

### 3. DAO Governance
1. Navigate to "Vote" tab
2. Click "Connect Wallet" (any EIP-6963 wallet)
3. Browse active proposals
4. Click "For", "Against", or "Abstain" to vote
5. Click "Create Proposal" to submit new proposals (requires 3+ reputation)
6. After voting ends, click "Finalize Proposal"
7. After 2-day delay, click "Execute Proposal" to activate

### 4. Space Lab
1. Navigate to "Space Lab" tab
2. Toggle "Hand Tracking" on
3. Move your hand in front of webcam
4. Hand size controls zoom/scale
5. Select different shapes from dropdown

### 5. Playground
1. Navigate to "Playground" tab
2. Type text in console input
3. Click "MORPH" — text appears as 3D particles
4. Adjust color picker and size slider
5. Upload images to add to canvas

---

## 📜 Smart Contract

### Deployed Addresses
- **BNB Testnet**: `0x...` (coming soon)
- **BNB Mainnet**: Not yet deployed

### Key Functions

#### Governance
```solidity
// Create a proposal (requires 3+ reputation)
function createProposal(
    ProposalType _type,
    string calldata _title,
    string calldata _description,
    string calldata _ipfsHash,
    bool _quickVote
) external returns (uint256)

// Cast a vote
function vote(uint256 _proposalId, VoteChoice _choice) external

// Finalize after voting ends
function finalizeProposal(uint256 _proposalId) external

// Execute after time lock
function executeProposal(uint256 _proposalId) external
```

#### Delegation
```solidity
// Delegate voting power
function delegate(address _delegatee) external

// Remove delegation
function undelegate() external
```

#### View Functions
```solidity
// Get proposal details
function getProposal(uint256 _proposalId) external view returns (...)

// Check quorum
function isQuorumReached(uint256 _proposalId) public view returns (bool)

// Get user stats
function userReputation(address) public view returns (uint256)
function userVoteCount(address) public view returns (uint256)
```

### Governance Parameters
| Parameter | Value | Adjustable |
|-----------|-------|------------|
| Voting Period (standard) | 7 days | Yes (owner) |
| Quick Vote Period | 3 days | Yes (owner) |
| Execution Delay | 2 days | Yes (owner) |
| Minimum Votes | 5 | Yes (owner) |
| Quorum | 10% of voters | Yes (owner) |
| Proposal Threshold | 3 reputation | Yes (owner) |
| Max Reputation | 10,000 | No (hardcoded) |

### Reputation Rewards
- **Vote on proposal**: +1 reputation
- **Execute proposal**: +5 reputation
- **Admin award**: Variable (owner only)

---

## 🔌 API Reference

### Backend Endpoints

#### AI Services
```
POST /api/chat
Body: { prompt: string, maxTokens?: number, temperature?: number }
Returns: { choices: [{ text: string }] }

POST /api/identify
Body: { image: base64_string }
Returns: { description: string }
```

#### Astrometry Pipeline
```
POST /api/analyze-discovery
Body: { imageBase64: string }
Returns: {
  coords: { ra: string, dec: string },
  historicalImage: string,
  discovery: string,
  type: "SUPERNOVA" | "GALAXY",
  rawScore: number
}
```

#### DAO Endpoints (Supabase)
```
GET /api/dao/posts
Returns: { success: true, posts: Post[] }

POST /api/dao/posts
Body: FormData { text, userId, author, image? }
Returns: { success: true, post: Post }

POST /api/dao/posts/:id/like
Body: { userId: string }
Returns: { success: true, post: Post }

GET /api/dao/posts/:id/comments
Returns: { success: true, comments: Comment[] }

POST /api/dao/posts/:id/comments
Body: { userId, author, text, parentId? }
Returns: { success: true, comment: Comment }

POST /api/dao/comments/:id/like
Body: { userId: string }
Returns: { success: true, comment: Comment }
```

#### User Profiles
```
GET /api/users/:userId
Returns: User

PUT /api/users/:userId
Body: { username?, bio?, avatar? }
Returns: User

GET /api/users
Returns: User[]
```

---

## 🤝 Contributing

We welcome contributions! Please follow these steps:

### 1. Fork & Clone
```bash
git clone https://github.com/yourusername/astrovision.git
cd astrovision
git checkout -b feature/your-feature-name
```

### 2. Make Changes
- Follow existing code style
- Add comments for complex logic
- Test thoroughly

### 3. Submit PR
- Write clear commit messages
- Reference any related issues
- Include screenshots for UI changes

### Code Style
- **Frontend**: React functional components, hooks, JSX
- **Backend**: Express REST conventions, async/await
- **Smart Contracts**: Solidity 0.8.20, OpenZeppelin patterns

---

## 🗺️ Roadmap

### Phase 1: Foundation (✅ COMPLETE)
- [x] React SPA with 5 main tabs
- [x] Node.js backend deployed on Render
- [x] Supabase DAO feed (posts, comments, likes)
- [x] AstroDAO smart contract with reputation
- [x] AI integration (vision + chat)
- [x] Astrometry pipeline + anomaly detection
- [x] EIP-6963 multi-wallet support
- [x] Material Design icons
- [x] Three.js Space Lab with hand tracking
- [x] BNB testnet → mainnet migration
- [x] ENS / BNB NS username resolution
- [x] Supabase real-time WebSocket (replace polling)
- [x] IPFS integration for immutable storage
- [x] Mobile-optimized layouts + PWA
- [x] Social login → on-chain identity bridge

### Phase 2: Scale (Q2 2026)
- [ ] Native BEP-20 AstroToken for staking
- [ ] Multi-chain support (opBNB for L2)
- [ ] Institutional API for universities
- [ ] AI model fine-tuning on community data
- [ ] Cross-DAO collaboration proposals

### Phase 3: Ecosystem (Q3 2026)
- [ ] Open-source AstroVision SDK
- [ ] Grant programme (DAO-governed)
- [ ] Partnerships 
- [ ] Physical hardware integration (smart telescopes)
- [ ] Reference implementation for science DAOs

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details

---

## 🙏 Acknowledgments

- **BNB Chain** for hackathon infrastructure
- **OpenZeppelin** for security libraries
- **Supabase** for database + storage
- **HuggingFace** for AI model hosting
- **Astrometry.net** for coordinate solving
- **NASA SkyView** for historical survey data

---

## 📧 Contact

- **GitHub**: [github.com/yourusername/astrovision](https://github.com/davife2025/astrovisionapp)
- **Twitter**: [@astrovision](https://twitter.com/astrovisionx)


---

**Built with ❤️ by the AstroVision team during BNB Chain Hackathon 2026**

*The universe is too big to explore alone. Join us in building the future of decentralized science.*
