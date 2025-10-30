# 🎨 How NFT Metadata Works in Mindora Runner

This guide explains exactly how your NFT badges get their images and metadata.

---

## 📖 The Complete Flow

### **Phase 1: Token Creation (One Time Setup)**

```bash
node scripts/createTokens.js
```

**What happens:**
```
Creates NFT Token ID: 0.0.XXXXXX
  ├─ Token Name: "Mindora Runner Badges"
  ├─ Token Type: NFT (Non-Fungible)
  ├─ Supply: Unlimited
  ├─ Metadata Key: Set (allows adding metadata later!)
  └─ Status: EMPTY (no NFTs minted yet)
```

**This is just creating the "container" for your NFTs. No images, no metadata yet!**

---

### **Phase 2: Upload Badge Images (Before Playing)**

**You need 3 images:**

1. **Explorer Badge** (Stage 1) 🎯
   - File: `explorer-badge.png`
   - Size: 512x512px recommended
   - Upload to IPFS (Pinata.cloud)
   - Get URL: `https://gateway.pinata.cloud/ipfs/QmXXX...`

2. **Adventurer Badge** (Stage 2) ⚔️
   - File: `adventurer-badge.png`
   - Upload to IPFS
   - Get URL: `https://gateway.pinata.cloud/ipfs/QmYYY...`

3. **Master Badge** (Stage 3) 👑
   - File: `master-badge.png`
   - Upload to IPFS
   - Get URL: `https://gateway.pinata.cloud/ipfs/QmZZZ...`

---

### **Phase 3: Update Frontend with Image URLs**

Open `frontend/src/services/hederaService.ts` around line 1093:

```typescript
const BADGE_METADATA = {
  "Explorer Badge": {
    name: "Explorer Badge",
    description: "Stage 1 achievement - Mindora Runner",
    image: "https://gateway.pinata.cloud/ipfs/QmXXX...", // ← YOUR IMAGE
    attributes: [
      { trait_type: "Stage", value: "1" },
      { trait_type: "Reward", value: "20 QuestCoins" },
      { trait_type: "Rarity", value: "Common" }
    ]
  },
  "Adventurer Badge": {
    image: "https://gateway.pinata.cloud/ipfs/QmYYY...", // ← YOUR IMAGE
    // ...
  },
  "Master Badge": {
    image: "https://gateway.pinata.cloud/ipfs/QmZZZ...", // ← YOUR IMAGE
    // ...
  }
};
```

---

### **Phase 4: Player Claims Badge (Runtime)**

**When a player clicks "MINT NFT + TOKENS":**

```
1. Player completes Stage 2 ⚔️
2. Badge shows "🔓 UNLOCKED"
3. Player clicks "MINT NFT + TOKENS"
   │
   ├─ Step 1: Mint 50 QuestCoins ✅
   │
   └─ Step 2: Mint NFT Badge
       │
       ├─ Get badge type: "Adventurer Badge"
       │
       ├─ Load metadata from BADGE_METADATA object:
       │   {
       │     name: "Adventurer Badge",
       │     description: "Stage 2 achievement",
       │     image: "https://gateway.pinata.cloud/ipfs/QmYYY...",
       │     attributes: [...]
       │   }
       │
       ├─ Convert to Buffer
       │
       └─ Call TokenMintTransaction
           .setTokenId(NFT_TOKEN_ID)
           .setMetadata([metadata])  ← THIS IS WHERE IMAGE URL GETS SET!
           .execute()
```

---

## 🎯 Key Points

### ✅ **What GETS Stored On-Chain:**

The metadata JSON (including image URL) is stored directly in the NFT:

```json
{
  "name": "Adventurer Badge",
  "description": "Stage 2 achievement - Mindora Runner",
  "image": "https://gateway.pinata.cloud/ipfs/QmYYY...",
  "attributes": [
    { "trait_type": "Stage", "value": "2" },
    { "trait_type": "Reward", "value": "50 QuestCoins" },
    { "trait_type": "Rarity", "value": "Rare" }
  ]
}
```

### ✅ **Each NFT Minted is Unique:**

```
NFT #1 (Serial 1): Explorer Badge → Image: ipfs://QmXXX
NFT #2 (Serial 2): Adventurer Badge → Image: ipfs://QmYYY
NFT #3 (Serial 3): Adventurer Badge → Image: ipfs://QmYYY (same as #2)
NFT #4 (Serial 4): Master Badge → Image: ipfs://QmZZZ
```

### ✅ **Different Images Per Stage:**

- All Stage 1 badges → Show `explorer-badge.png`
- All Stage 2 badges → Show `adventurer-badge.png`
- All Stage 3 badges → Show `master-badge.png`

---

## 🔧 Step-by-Step Setup Checklist

- [ ] **1. Create 3 badge images** (PNG, 512x512px)
- [ ] **2. Upload images to IPFS** (use Pinata.cloud)
- [ ] **3. Copy image URLs** from IPFS
- [ ] **4. Update `hederaService.ts`** with image URLs (line 1093)
- [ ] **5. Deploy smart contract** (use Remix)
- [ ] **6. Run `createTokens.js`** (creates NFT token)
- [ ] **7. Test minting** (play game, claim badge)
- [ ] **8. Check Hashscan** - verify image appears!

---

## 🌐 Where You'll See the Images

### **During Minting:**
```
Frontend → mintNFTBadge() → Gets metadata with image URL → Mints to blockchain
```

### **After Minting:**
```
1. Hashscan: https://hashscan.io/testnet/token/0.0.XXXXXX/1
   └─ Shows NFT with image from IPFS

2. Wallet: MetaMask/HashPack
   └─ Displays NFT with image

3. Marketplaces: (if listed)
   └─ Shows NFT collection with all badges
```

---

## 📊 Visual Diagram

```
┌─────────────────┐
│  Upload Images  │
│   to IPFS       │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Get IPFS URLs   │
│ QmXXX, QmYYY... │
└────────┬────────┘
         │
         ↓
┌─────────────────────┐
│ Update hederaService│
│  with image URLs    │
└────────┬────────────┘
         │
         ↓
┌─────────────────────┐
│  Create NFT Token   │
│  (scripts/create    │
│   Tokens.js)        │
└────────┬────────────┘
         │
         ↓
    [Game Ready!]
         │
         ↓
┌─────────────────────┐
│  Player Completes   │
│  Stage & Claims     │
└────────┬────────────┘
         │
         ↓
┌─────────────────────┐
│  Frontend Mints NFT │
│  WITH metadata      │
│  (includes image)   │
└────────┬────────────┘
         │
         ↓
┌─────────────────────┐
│  NFT Created        │
│  ✅ Name            │
│  ✅ Description     │
│  ✅ Image (IPFS)    │
│  ✅ Attributes      │
└─────────────────────┘
```

---

## ❓ Common Questions

### Q: When do I set the metadata?
**A:** When you MINT the NFT (runtime), NOT when creating the token!

### Q: Can I use different images for different players?
**A:** No - all Stage 2 badges use the same image. But Stage 1, 2, and 3 each have DIFFERENT images.

### Q: Where is the image stored?
**A:** Image is on IPFS. Only the **URL** is stored in the NFT metadata on-chain.

### Q: Can I change the image later?
**A:** Only if you set a `metadataKey` when creating the token (which the script does!).

### Q: What if I don't upload images?
**A:** The NFT will still work, but won't show any image in wallets/explorers.

---

## 🚀 You're Ready!

Once you update the image URLs in `hederaService.ts`, your NFTs will automatically mint with the correct images for each stage!

**No more confusion! 🎉**
