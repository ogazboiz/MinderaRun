# 🚀 Mindora Runner - Complete Deployment Guide

This guide walks you through deploying your game with proper NFT images and metadata.

---

## 📋 Prerequisites

- [ ] Node.js installed
- [ ] Hedera Testnet account with HBAR
- [ ] 3 Badge images designed (PNG/JPG format, recommended 512x512px)
- [ ] IPFS account or cloud storage (for hosting images)

---

## 🎨 STEP 1: Prepare NFT Images

### Create Your Badge Images

You need 3 different images:

1. **Stage 1 - Explorer Badge** 🎯
   - File: `explorer-badge.png`
   - Theme: Beginner, exploration, first steps
   - Reward: 20 QuestCoins

2. **Stage 2 - Adventurer Badge** ⚔️
   - File: `adventurer-badge.png`
   - Theme: Adventure, courage, progress
   - Reward: 50 QuestCoins

3. **Stage 3 - Master Badge** 👑
   - File: `master-badge.png`
   - Theme: Mastery, legendary, champion
   - Reward: 100 QuestCoins

### Upload Images to IPFS/Storage

**Option A: Using IPFS (Recommended for Web3)**
```bash
# Using Pinata (https://pinata.cloud)
1. Create account on Pinata
2. Upload each badge image
3. Copy the IPFS URLs (e.g., https://gateway.pinata.cloud/ipfs/QmXXX...)
```

**Option B: Using Cloud Storage**
```bash
# Using any cloud service (AWS S3, Cloudflare, etc.)
1. Upload images to your storage
2. Make sure they're publicly accessible
3. Copy the public URLs
```

---

## 📝 STEP 2: Setup NFT Metadata

### Update Metadata Script

1. Open `scripts/setupNFTMetadata.js`
2. Replace the `IMAGE_URLS` with your uploaded image URLs:

```javascript
const IMAGE_URLS = {
  stage1: "https://gateway.pinata.cloud/ipfs/YOUR_STAGE1_HASH",
  stage2: "https://gateway.pinata.cloud/ipfs/YOUR_STAGE2_HASH",
  stage3: "https://gateway.pinata.cloud/ipfs/YOUR_STAGE3_HASH"
};
```

### Generate Metadata Files

```bash
cd scripts
node setupNFTMetadata.js
```

This creates:
- `nft-metadata/stage1-metadata.json`
- `nft-metadata/stage2-metadata.json`
- `nft-metadata/stage3-metadata.json`
- `nft-metadata/badge-config.json`

### Upload Metadata to IPFS/Storage

Upload the generated JSON files the same way you uploaded images:

```bash
# You'll get URLs like:
# https://gateway.pinata.cloud/ipfs/METADATA1_HASH (Stage 1)
# https://gateway.pinata.cloud/ipfs/METADATA2_HASH (Stage 2)
# https://gateway.pinata.cloud/ipfs/METADATA3_HASH (Stage 3)
```

**SAVE THESE METADATA URLs - YOU'LL NEED THEM!**

---

## 🔧 STEP 3: Update Hedera Service

Open `frontend/src/services/hederaService.ts` and update the `mintNFTBadge` function to use the correct metadata:

```typescript
const BADGE_METADATA_URLS = {
  "Explorer Badge": "https://your-storage/stage1-metadata.json",
  "Adventurer Badge": "https://your-storage/stage2-metadata.json",
  "Master Badge": "https://your-storage/stage3-metadata.json"
};

// In mintNFTBadge function:
const metadataURL = BADGE_METADATA_URLS[badgeType];
```

---

## 📦 STEP 4: Compile Smart Contract

### Option A: Using Remix IDE (Easiest)

1. Go to https://remix.ethereum.org
2. Create new file: `MindoraRunnerFinal.sol`
3. Copy your contract code from `Smart-contract/MindoraRunnerFinal.sol`
4. Compile with Solidity 0.8.19
5. Download the ABI and bytecode

### Option B: Using Hardhat/Foundry

```bash
# If using Hardhat
npx hardhat compile

# If using Foundry
forge build
```

**SAVE THE ABI - YOU'LL NEED IT!**

---

## 🚀 STEP 5: Deploy to Hedera

### Setup Environment Variables

Create `.env` file in `scripts/` directory:

```bash
HEDERA_OPERATOR_ID=0.0.YOUR_ACCOUNT_ID
HEDERA_OPERATOR_KEY=YOUR_PRIVATE_KEY
```

### Update Deployment Script

Open `scripts/deployGame.js` and update line 40 with your compiled bytecode:

```javascript
const contractBytecode = "0xYOUR_COMPILED_BYTECODE_HERE";
```

### Run Deployment

```bash
cd scripts
node deployGame.js
```

This will:
1. ✅ Deploy smart contract
2. ✅ Create QuestCoin token (fungible)
3. ✅ Create Badge NFT token (non-fungible)
4. ✅ Create HCS topic for game events
5. ✅ Generate `.env.local` file for frontend

**Output:**
```
✅ Smart Contract deployed: 0.0.XXXXXX
✅ QuestCoin Token created: 0.0.YYYYYY
✅ Badge NFT Token created: 0.0.ZZZZZZ
✅ HCS Topic created: 0.0.WWWWWW
```

---

## 🔄 STEP 6: Update Frontend

### 1. Copy New ABI

Replace `frontend/src/config/abis/MindoraRunnerABI.ts` with the new ABI from compilation.

### 2. Update Contract Address

The deployment script should have created `frontend/.env.local`:

```bash
NEXT_PUBLIC_CONTRACT_ADDRESS=0.0.XXXXXX
NEXT_PUBLIC_QUESTCOIN_TOKEN_ID=0.0.YYYYYY
NEXT_PUBLIC_BADGE_NFT_TOKEN_ID=0.0.ZZZZZZ
NEXT_PUBLIC_GAME_EVENTS_TOPIC=0.0.WWWWWW
```

### 3. Verify All URLs Are Set

Check that your metadata URLs are configured in `hederaService.ts`.

---

## ✅ STEP 7: Test Everything

### 1. Start Frontend

```bash
cd frontend
npm install
npm run dev
```

### 2. Test Flow

1. ✅ Connect wallet
2. ✅ Register player
3. ✅ Play Stage 1
4. ✅ Complete Stage 1
5. ✅ Check badge shows "🔓 UNLOCKED"
6. ✅ Click "MINT NFT + TOKENS"
7. ✅ Verify tokens minted
8. ✅ Verify NFT minted with correct image
9. ✅ Badge shows "✅ REWARDS CLAIMED"
10. ✅ Check leaderboard shows scores

### 3. Test Partial Claim

1. Simulate NFT mint failure
2. Verify tokens still marked as claimed
3. Retry NFT mint
4. Verify both now claimed

---

## 🎯 Summary Checklist

- [ ] 3 badge images created and uploaded
- [ ] Image URLs obtained
- [ ] Metadata generated with `setupNFTMetadata.js`
- [ ] Metadata JSONs uploaded
- [ ] Metadata URLs obtained
- [ ] `hederaService.ts` updated with metadata URLs
- [ ] Smart contract compiled
- [ ] ABI saved
- [ ] Bytecode obtained
- [ ] `.env` configured in scripts folder
- [ ] `deployGame.js` updated with bytecode
- [ ] Deployment executed successfully
- [ ] Frontend ABI updated
- [ ] Frontend `.env.local` updated
- [ ] Frontend tested
- [ ] All claim flows tested
- [ ] Leaderboard working

---

## 📊 Expected Costs (Hedera Testnet)

- Smart Contract Deployment: ~20 HBAR
- QuestCoin Token Creation: ~20 HBAR
- Badge NFT Token Creation: ~20 HBAR
- HCS Topic Creation: ~5 HBAR
- **Total: ~65 HBAR** (approximately $3-5 USD)

---

## 🐛 Troubleshooting

### Issue: "Metadata not loading"
- Check if metadata URLs are publicly accessible
- Verify JSON format is correct
- Check browser console for errors

### Issue: "NFT mint fails"
- Verify treasury account has supply key
- Check if user account is associated with token
- Enable auto-association if available

### Issue: "Tokens claimed but NFT shows not claimed"
- This is expected behavior! They're tracked separately
- User can retry NFT mint without re-minting tokens

---

## 🎉 You're Done!

Your game is now fully deployed with:
- ✅ Separate token & NFT claim tracking
- ✅ Unique images for each stage badge
- ✅ General leaderboard across all stages
- ✅ Proper metadata for all NFTs
- ✅ No double-claiming possible

**Happy Gaming! 🎮**
