# 🚀 Frontend Implementation Checklist

## ✅ COMPLETED

### 1. Environment Variables Updated
- ✅ Contract Address: `0.0.7158178`
- ✅ QuestCoin Token: `0.0.7158216`
- ✅ Badge NFT Token: `0.0.7158217`

File: `frontend/.env.local`

---

## 📋 TODO - NEXT STEPS

### 2. Replace ABI File (CRITICAL)
**File:** `frontend/src/config/abis/MindoraRunnerABI.ts`

**Action:** Replace ENTIRE file content with the new ABI you provided.

The new ABI includes:
- ✅ `claimTokens(stage)` function
- ✅ `claimNFT(stage)` function
- ✅ `areTokensClaimed(player, stage)` function
- ✅ `isNFTClaimed(player, stage)` function
- ✅ `getGeneralLeaderboard(limit)` function
- ✅ `TokensClaimed` event
- ✅ `NFTClaimed` event

**Steps:**
1. Open `frontend/src/config/abis/MindoraRunnerABI.ts`
2. Delete ALL content
3. Paste your new ABI starting with `export const MindoraRunnerABI = [`
4. End with `] as const;`
5. Save file

---

### 3. Complete ContractManager Updates

**File:** `frontend/src/components/ContractManager.tsx`

**Changes already made (partially):**
- ✅ Imports updated to use `useTokensClaimed`, `useNFTClaimed`, `useGeneralLeaderboard`
- ✅ Hooks initialized for all 3 stages

**Still need to complete:**

```typescript
// Around line 218-221 - Update dependencies
], [
  // ... existing deps ...
  stage1TokensClaimed.isClaimed,
  stage2TokensClaimed.isClaimed,
  stage3TokensClaimed.isClaimed,
  stage1NFTClaimed.isClaimed,
  stage2NFTClaimed.isClaimed,
  stage3NFTClaimed.isClaimed,
  address
]);
```

```typescript
// Around line 601-619 - Update claimRewards callback
claimTokens: async (stage: number): Promise<boolean> => {
  try {
    console.log(`💰 Claiming tokens for stage ${stage}...`);
    await hooksRef.current.claimTokens(stage);
    // Wait and refetch
    setTimeout(() => {
      hooksRef.current.refetchPlayer();
      stage1TokensClaimed.refetch();
      stage2TokensClaimed.refetch();
      stage3TokensClaimed.refetch();
    }, 3000);
    return true;
  } catch (error) {
    console.error('❌ Claim tokens failed:', error);
    return false;
  }
},

claimNFT: async (stage: number): Promise<boolean> => {
  try {
    console.log(`🎖️ Claiming NFT for stage ${stage}...`);
    await hooksRef.current.claimNFT(stage);
    // Wait and refetch
    setTimeout(() => {
      hooksRef.current.refetchPlayer();
      stage1NFTClaimed.refetch();
      stage2NFTClaimed.refetch();
      stage3NFTClaimed.refetch();
    }, 3000);
    return true;
  } catch (error) {
    console.error('❌ Claim NFT failed:', error);
    return false;
  }
},
```

---

### 4. Update gameStore Callbacks

**File:** `frontend/src/store/gameStore.ts`

**Around line 106-121:**

```typescript
setContractCallbacks: (callbacks: {
  registerPlayer: (username: string) => Promise<boolean>;
  saveGameSession: (...) => Promise<{ success: boolean; transactionId?: string }>;
  waitForTransactionConfirmation: (transactionId: string) => Promise<boolean>;
  loadPlayerData: (walletAddress: string) => Promise<Player | null>;
  loadLeaderboard: (stage: number, limit: number) => Promise<LeaderboardEntry[]>;
  claimTokens: (stage: number) => Promise<boolean>;  // ← ADD THIS
  claimNFT: (stage: number) => Promise<boolean>;      // ← ADD THIS
}) => void;

contractCallbacks: {
  registerPlayer?: (username: string) => Promise<boolean>;
  saveGameSession?: (...) => Promise<{ success: boolean; transactionId?: string }>;
  waitForTransactionConfirmation?: (transactionId: string) => Promise<boolean>;
  loadPlayerData?: (walletAddress: string) => Promise<Player | null>;
  loadLeaderboard?: (stage: number, limit: number) => Promise<LeaderboardEntry[]>;
  claimTokens?: (stage: number) => Promise<boolean>;  // ← ADD THIS
  claimNFT?: (stage: number) => Promise<boolean>;      // ← ADD THIS
};
```

---

### 5. GameUI Already Updated! ✅

**File:** `frontend/src/components/GameUI.tsx`

Already updated with:
- ✅ Stage 2 & 3 badges show claimed status
- ✅ Calls `contractCallbacks.claimTokens()` after token mint
- ✅ Calls `contractCallbacks.claimNFT()` after NFT mint
- ✅ Shows different messages for partial claims
- ✅ Leaderboard title changed to "🏆 Global Leaderboard"

---

## 🧪 TESTING PLAN

### Phase 1: Basic Connection
1. Start dev server: `npm run dev`
2. Connect MetaMask
3. Verify wallet connection works
4. Check console for errors

### Phase 2: Player Registration
1. Click "Register" button
2. Enter username
3. Sign transaction
4. Verify registration success

### Phase 3: Stage 1 Gameplay
1. Play Stage 1
2. Complete the stage
3. Check badge shows "🔓 UNLOCKED"
4. Verify `stageCompleted[player][1]` is true

### Phase 4: Stage 2 Minting (THE BIG TEST!)
1. Complete Stage 2
2. Badge shows "🔓 UNLOCKED"
3. Click "MINT NFT + TOKENS"
4. **Expected Flow:**
   - Mints 50 QuestCoin tokens ✅
   - Calls `claimTokens(2)` → `tokensClaimed[player][2] = true` ✅
   - Mints NFT badge ✅
   - Calls `claimNFT(2)` → `nftClaimed[player][2] = true` ✅
   - Badge shows "✅ REWARDS CLAIMED"

### Phase 5: Partial Claim Test
1. Simulate NFT mint failure (disconnect before NFT mints)
2. Verify tokens still marked as claimed
3. Retry minting
4. Only NFT should mint (tokens already claimed)
5. Both should be marked claimed after

### Phase 6: Leaderboard Test
1. Open leaderboard
2. Verify title shows "🏆 Global Leaderboard"
3. Verify shows scores from ALL stages
4. Play different stages, check leaderboard updates

---

## 🎨 NFT METADATA (Later)

After everything works:

### 1. Create Badge Images
- Stage 1: Explorer Badge 🎯 (512x512px PNG)
- Stage 2: Adventurer Badge ⚔️
- Stage 3: Master Badge 👑

### 2. Upload to IPFS
- Use Pinata.cloud
- Get 3 image URLs

### 3. Update hederaService.ts
**File:** `frontend/src/services/hederaService.ts`
**Line:** ~1097, 1107, 1117

```typescript
const BADGE_METADATA = {
  "Explorer Badge": {
    image: "https://gateway.pinata.cloud/ipfs/YOUR_HASH_1",
    // ...
  },
  "Adventurer Badge": {
    image: "https://gateway.pinata.cloud/ipfs/YOUR_HASH_2",
    // ...
  },
  "Master Badge": {
    image: "https://gateway.pinata.cloud/ipfs/YOUR_HASH_3",
    // ...
  }
};
```

---

## ⚠️ IMPORTANT NOTES

1. **Replace ABI FIRST** - Nothing will work without the new ABI!
2. **Test incrementally** - Don't skip testing steps
3. **Check console logs** - Lots of helpful debug info
4. **Hashscan is your friend** - Verify transactions on https://hashscan.io/testnet
5. **NFT images can wait** - Get functionality working first!

---

## 🆘 If Things Break

### Contract calls fail:
- Check contract address in `.env.local`
- Verify ABI matches deployed contract
- Check wallet has HBAR for gas

### Hooks not working:
- Check imports in `ContractManager.tsx`
- Verify hook names match (`useTokensClaimed` not `useRewardsClaimed`)
- Check console for hook errors

### Claims not showing:
- Verify `claimedStages` array includes both tokens AND NFT claimed
- Check `player.claimedStages` in React DevTools
- Verify contract functions `areTokensClaimed` and `isNFTClaimed` return correct values

---

## 📊 Progress Tracker

- [x] Update .env.local with new contract/token IDs
- [ ] Replace ABI file completely
- [ ] Complete ContractManager dependencies
- [ ] Add claimTokens/claimNFT callbacks
- [ ] Test wallet connection
- [ ] Test registration
- [ ] Test Stage 1 completion
- [ ] Test Stage 2 minting
- [ ] Test partial claims
- [ ] Test leaderboard
- [ ] Upload badge images
- [ ] Update NFT metadata URLs

---

**YOU'RE ALMOST THERE! 🎉**

The hard work (smart contract + scripts) is done. Now just connect the frontend to your deployed contract!
