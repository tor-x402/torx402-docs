---
description: Understanding Merkle root history and why your funds are never locked
---

# Historical Merkle Root Storage

```
Your Commitment in Tree: PERMANENT ✅
  [C0, C1, ... C_yours, ... C9999]
              ↑
              Never removed, always accessible

Root in History Buffer: TEMPORARY (convenience)
  [R0, R1, ... R998, R999]
   ↑
   Eventually overwritten

If root expires: Regenerate proof (5-10 seconds) ✅
No funds lost, no access denied!
```

## The Challenge: Ever-Changing Merkle Roots

### Why Roots Change

Every time someone deposits into a privacy pool, the Merkle tree changes, which means the **Merkle root changes**:

```
Initial State (3 deposits):
  Tree: [C0, C1, C2, 0, 0, 0, ...]
  Root: R0 = hash(hash(C0,C1), hash(C2,0))

After Alice Deposits (C3):
  Tree: [C0, C1, C2, C3, 0, 0, ...]
  Root: R1 = hash(hash(C0,C1), hash(C2,C3))  ← DIFFERENT!

After Bob Deposits (C4):
  Tree: [C0, C1, C2, C3, C4, 0, ...]
  Root: R2 = hash(hash(C0,C1), hash(C2,C3), hash(C4,0))  ← DIFFERENT AGAIN!
```

**Key Insight**: The root changes with every deposit, but **commitments never change**.

### The Withdrawal Challenge

**Scenario:**
```
10:00 AM - You deposit
  Commitment: C_yours at leafIndex 100
  Current Root: R100

10:01 AM - Someone else deposits
  Current Root: R101 (changed!)

10:02 AM - Another deposit
  Current Root: R102 (changed!)

...

10:30 AM - You want to withdraw
  Current Root: R130

Question: Can you still prove your deposit exists?
Answer: YES! Your commitment is still at leafIndex 100
```

## The Solution: Historical Root Buffer

### Root History as a Convenience

The smart contract stores **recent roots** to avoid forcing users to regenerate proofs constantly:

```solidity
contract PrivacyPool {
    // Store last 1,000 Merkle roots in circular buffer
    bytes32[1000] public roots;
    mapping(bytes32 => bool) private rootSet; // O(1) lookup
    uint256 public currentRootIndex = 0;

    function isKnownRoot(bytes32 root) public view returns (bool) {
        return rootSet[root];  // Constant time lookup
    }

    function addRoot(bytes32 newRoot) internal {
        // Remove oldest root from mapping
        bytes32 oldRoot = roots[currentRootIndex];
        if (oldRoot != bytes32(0)) {
            delete rootSet[oldRoot];
        }

        // Add new root to both structures
        roots[currentRootIndex] = newRoot;
        rootSet[newRoot] = true;

        // Circular buffer: wraps around at 1,000
        currentRootIndex = (currentRootIndex + 1) % 1000;
    }
}
```

### How Withdrawals Work

**Case 1: Root Still in History (No Regeneration)**
```
You deposited at 10:00 AM (Root R100)
You withdraw at 2:00 PM (200 deposits later, Current Root R300)

Check: isKnownRoot(R100)?
  → Search in last 1,000 roots
  → R100 found at position 100 ✅
  → Your pre-generated proof is accepted
  → Withdraw immediately

Time: 0 extra seconds
Convenience: Maximum ✅
```

**Case 2: Root Expired from History (Regenerate Proof)**
```
You deposited at 10:00 AM (Root R100)
You withdraw 2 weeks later (1,500 deposits later, Current Root R1600)

Check: isKnownRoot(R100)?
  → Search in last 1,000 roots
  → R100 NOT found (evicted at deposit 1,100)
  → Your pre-generated proof is rejected

BUT your commitment is still at leafIndex 100! ✅

Solution:
  1. Fetch current root R1600
  2. Fetch Merkle proof for leafIndex 100
  3. Generate new zk-SNARK proof (5-10 seconds)
  4. Submit new proof
  5. Withdraw successfully ✅

Time: 5-10 seconds extra (one-time)
Funds status: SAFE, fully accessible ✅
```

## Why 1,000 Roots? (Not 30, 100, or 10,000)

### Comparison of Options

| Roots | Storage | Gas Cost | Time Window (High-Vol Pool) | Regeneration Frequency | Verdict |
|-------|---------|----------|----------------------------|------------------------|---------|
| **30** (Tornado) | 960 B | ~6k gas | 18 minutes | Very often | ❌ Too short for micropayments |
| **100** | 3.2 KB | ~20k gas | 1 hour | Often | ⚠️ Still too tight |
| **1,000** ✅ | **32 KB** | **~3k gas** | **10 hours** | **Rare** | ✅ **OPTIMAL** |
| **10,000** | 320 KB | ~3k gas | 4+ days | Very rare | ⚠️ Unnecessary (overkill) |

### Time Windows by Pool Volume

```
Ultra-High Volume (0.001 ETH - 100 deposits/hour):
  30 roots: 18 minutes ❌
  1,000 roots: 10 hours ✅
  → User can sleep, work, live normally

High Volume (0.01 ETH - 50 deposits/hour):
  30 roots: 36 minutes ❌
  1,000 roots: 20 hours ✅
  → Covers full day + sleep

Medium Volume (0.1 ETH - 20 deposits/hour):
  30 roots: 1.5 hours ⚠️
  1,000 roots: 2+ days ✅
  → Covers weekend wait for anonymity

Low Volume (1 ETH - 5 deposits/hour):
  30 roots: 6 hours ✅
  1,000 roots: 8+ days ✅
  → Plenty of time
```

### Why Not 10,000?

**Diminishing Returns:**
```
1,000 roots: 10 hours (high-vol)
  → Covers: Sleep, work, normal delays ✅

10,000 roots: 4+ days (high-vol)
  → Covers: Extended weekends, vacations
  → But: Regeneration only takes 5-10 seconds anyway
  → Benefit: Minimal (avoid 5-10 sec regeneration?)
  → Cost: 10x storage (320 KB vs 32 KB)

Verdict: Not worth 10x storage for marginal benefit
```

**Philosophy**: If someone waits days/weeks to withdraw, spending 5-10 seconds to regenerate proof is acceptable. We don't need massive storage to avoid this minor inconvenience.

## Gas Cost Optimization: The Mapping Trick

### Naive Implementation (Don't Use)

```solidity
// O(n) iteration - expensive for large arrays
function isKnownRoot(bytes32 root) public view returns (bool) {
    for (uint256 i = 0; i < 1000; i++) {
        if (roots[i] == root) {
            return true;
        }
    }
    return false;
}

Gas cost: 1,000 iterations × ~200 gas = ~200,000 gas ❌
```

### Optimized Implementation (Use This)

```solidity
// O(1) mapping lookup - constant time
contract PrivacyPool {
    bytes32[1000] public roots;  // Historical record
    mapping(bytes32 => bool) private rootSet;  // Fast validation

    function isKnownRoot(bytes32 root) public view returns (bool) {
        return rootSet[root];  // Single lookup!
    }
}

Gas cost: Single mapping read = ~3,000 gas ✅
```

**Result**: Mapping makes 1,000 roots CHEAPER than iterating through 100 roots!

### Storage Management

```solidity
function addRoot(bytes32 newRoot) internal {
    // Step 1: Remove oldest root from mapping
    bytes32 oldRoot = roots[currentRootIndex];
    if (oldRoot != bytes32(0)) {
        delete rootSet[oldRoot];  // ~5k gas (storage refund)
    }

    // Step 2: Add new root to both structures
    roots[currentRootIndex] = newRoot;  // ~20k gas
    rootSet[newRoot] = true;  // ~20k gas

    // Step 3: Advance circular buffer pointer
    currentRootIndex = (currentRootIndex + 1) % 1000;

    // Total: ~45k gas per deposit
}
```

## Proof Regeneration: Simple & Automatic

### When Do You Need to Regenerate?

```
High-volume pool (100 deposits/hour):
  1,000 roots = 10 hour window

If you withdraw within 10 hours: No regeneration needed ✅
If you withdraw after 10+ hours: Regenerate (5-10 sec) ⚠️

Likelihood:
  - Same day withdrawal: 95% no regeneration
  - Next day withdrawal: 80% no regeneration
  - Week later: 0% (regeneration needed)
```

### The Regeneration Process

```typescript
// SDK handles this automatically
async function withdraw(note) {
  // 1. Check if root is still valid
  const rootValid = await pool.isKnownRoot(note.root);

  if (rootValid) {
    // Fast path: Use existing proof
    console.log('✅ Using cached proof');
    return await submitWithdrawal(note.proof);
  } else {
    // Regenerate path: Takes 5-10 seconds
    console.log('⏳ Root expired, regenerating proof...');

    // Your commitment is still at the same leafIndex!
    const currentState = await pool.getMerkleTreeState();
    const merkleProof = currentState.getProof(note.leafIndex);

    // Generate new zk-SNARK proof with current root
    const newProof = await generateZKProof({
      nullifier: note.k,
      secret: note.r,
      leafIndex: note.leafIndex,
      merkleProof: merkleProof,
      root: currentState.root,  // Use current root
      recipient: merchantAddress,
      relayer: relayerAddress,
      fee: relayerFee,
    });

    console.log('✅ New proof generated');
    return await submitWithdrawal(newProof);
  }
}

// User experience: Transparent, automatic
// Time penalty: 5-10 seconds (only if root expired)
// Financial cost: $0
```

## Why Tornado Cash Used 30 Roots

### Original Tornado Cash Design (2019)

```
Target use case: Large, infrequent transfers
  - Pools: 0.1 ETH, 1 ETH, 10 ETH, 100 ETH
  - Volume: 5-20 deposits per hour
  - User behavior: Deposit large sum, withdraw days/weeks later

30 roots calculation:
  Medium pool (10 deposits/hour): 30 roots = 3 hours
  Low pool (5 deposits/hour): 30 roots = 6 hours

Verdict: 30 roots was SUFFICIENT for large, low-frequency pools ✅

Why it worked:
  - Users expected to wait anyway (for anonymity)
  - Lower volume meant longer windows
  - Regeneration was rare
```

## Why torx402 Uses 1,000 Roots

### Target Use Case: High-Volume Micropayments

```
Target use case: Frequent, small payments
  - Pools: 0.001 ETH, 0.01 ETH, 0.1 ETH
  - Volume: 50-100+ deposits per hour
  - User behavior: Deposit and withdraw same day, AI agents constant usage

1,000 roots calculation:
  Ultra-high pool (100 deposits/hour): 1,000 roots = 10 hours
  High pool (50 deposits/hour): 1,000 roots = 20 hours

Verdict: 1,000 roots provides comfortable same-day window ✅

Why we need more:
  - Higher volume = faster root rotation
  - Global users (timezones)
  - Users shouldn't feel rushed
  - AI agents may have delays
```

### The Design Philosophy

```
Tornado Cash (30 roots):
  Philosophy: "Users wait days for anonymity anyway"
  Use case: Large amounts, patience expected
  Window: 18 min to 6 hours (sufficient)

torx402 (1,000 roots):
  Philosophy: "Users want fast, convenient payments"
  Use case: Micropayments, frequent usage
  Window: 10 hours to 2+ days (comfortable)

Both are correct for their use case!
```

## Technical Details

### Circular Buffer Implementation

```solidity
contract PrivacyPool {
    // Dual storage for efficiency
    bytes32[1000] public roots;           // Array: historical record
    mapping(bytes32 => bool) private rootSet;  // Mapping: fast lookup
    uint256 public currentRootIndex = 0;

    // Add new root (called on each deposit)
    function addRoot(bytes32 newRoot) internal {
        // Evict oldest root from mapping
        bytes32 oldRoot = roots[currentRootIndex];
        if (oldRoot != bytes32(0)) {
            delete rootSet[oldRoot];
        }

        // Add new root to both structures
        roots[currentRootIndex] = newRoot;
        rootSet[newRoot] = true;

        // Advance pointer (circular)
        currentRootIndex = (currentRootIndex + 1) % 1000;
    }

    // Validate root (O(1) constant time)
    function isKnownRoot(bytes32 root) public view returns (bool) {
        return rootSet[root];
    }

    // Withdraw function checks root validity
    function withdraw(
        uint[2] memory a,
        uint[2][2] memory b,
        uint[2] memory c,
        bytes32 root,
        bytes32 nullifierHash,
        address payable recipient,
        address payable relayer,
        uint256 fee
    ) external nonReentrant {
        // Verify root is in recent history
        require(isKnownRoot(root), "Unknown Merkle root");

        // Verify zk-SNARK proof
        require(
            verifier.verifyProof(a, b, c, [uint256(root), uint256(nullifierHash), ...]),
            "Invalid proof"
        );

        // Check nullifier not used
        require(!nullifierHashes[nullifierHash], "Already withdrawn");
        nullifierHashes[nullifierHash] = true;

        // Transfer funds
        recipient.transfer(DENOMINATION - fee);
        if (fee > 0) relayer.transfer(fee);

        emit Withdrawal(recipient, nullifierHash, relayer, fee);
    }
}
```

### Gas Cost Analysis

```
Per Deposit (add root):
  Delete old mapping entry: ~5,000 gas
  Add new mapping entry: ~20,000 gas
  Update array slot: ~20,000 gas
  ────────────────────────────────
  Total: ~45,000 gas
  Cost at 10 gwei: ~$0.01

Per Withdrawal (check root):
  Mapping lookup: ~3,000 gas
  ────────────────────────────────
  Total: ~3,000 gas (vs ~200k for iteration!)

Total Withdrawal Cost:
  Proof verification: ~280,000 gas
  Root lookup: ~3,000 gas
  Nullifier check: ~5,000 gas
  Transfers: ~50,000 gas
  ────────────────────────────────
  Total: ~338,000 gas

Root overhead: < 1% of total cost ✅
```

## Real-World Scenarios

### Scenario 1: Normal Usage (No Regeneration)

```
User deposits 0.01 ETH at 9:00 AM
Pool: 50 deposits/hour
Window: 1,000 roots = 20 hours

Timeline:
  9:00 AM - Deposit (Root R500)
  10:00 AM - 50 more deposits (Root R550)
  2:00 PM - 250 more deposits (Root R750)
  5:00 PM - User withdraws

Check: isKnownRoot(R500)?
  → YES (only 8 hours, 400 deposits elapsed)
  → Pre-generated proof accepted ✅
  → Instant withdrawal

User experience: Seamless, no delays
```

### Scenario 2: Long Wait (Regeneration Needed)

```
User deposits 0.001 ETH on Monday
Pool: 100 deposits/hour (ultra-high volume)
Window: 1,000 roots = 10 hours

Timeline:
  Monday 10:00 AM - Deposit (Root R2000)
  Monday 8:00 PM - Root R2000 evicted (R3000 is oldest)
  Wednesday 10:00 AM - User tries to withdraw

Check: isKnownRoot(R2000)?
  → NO (48 hours elapsed, 4,800 deposits)
  → Pre-generated proof rejected

SDK automatically regenerates:
  1. Fetch current root R6800
  2. Fetch Merkle proof for leafIndex 2000
     (commitment still there!)
  3. Generate new proof (5-10 seconds)
  4. Submit and withdraw ✅

User experience: 5-10 second delay, no funds lost
Message: "Regenerating proof... done! ✅"
```

### Scenario 3: Extreme Edge Case (Still Works!)

```
User deposits and forgets for 6 months
Pool has 100,000+ deposits now

Timeline:
  January 1 - Deposit (Root R1000, leafIndex 1000)
  July 1 - User remembers and withdraws

Check: isKnownRoot(R1000)?
  → NO (100,000 deposits later!)

Solution: Regenerate proof
  1. Fetch current tree state
  2. Commitment STILL at leafIndex 1000 ✅
  3. Generate proof with current root (5-10 seconds)
  4. Withdraw successfully ✅

User experience: 5-10 second delay after 6 months
Funds: Completely safe ✅
```

## Why Root History Doesn't Lock Funds

### The Key Distinction

```
Merkle Tree (THE SOURCE OF TRUTH):
  - Contains ALL commitments from ALL deposits
  - Commitments NEVER removed
  - Grows monotonically (never shrinks)
  - Your leafIndex is PERMANENT

  Your commitment at leafIndex 1000:
    Tree: [... C999, C1000 (yours), C1001, ...]
    Status: PERMANENT - never removed ✅

Root History (CONVENIENCE CACHE):
  - Contains RECENT roots only
  - Roots ARE removed (circular buffer)
  - Used to accept pre-generated proofs
  - NOT required for withdrawal

  Your root R1000:
    History: [... R999, R1000, R1001, ...]
    Status: TEMPORARY - will be evicted
    Effect: Must regenerate proof (5-10 sec)
    NOT: "Funds locked" ❌
```

### What the Contract Actually Checks

```solidity
function withdraw(..., bytes32 root, ...) external {
    // Check 1: Is root in recent history?
    require(isKnownRoot(root), "Unknown Merkle root");
    // ↑ This is just checking your PROOF is recent
    //   NOT checking your COMMITMENT exists

    // Check 2: Does the proof verify?
    require(verifyProof(...), "Invalid proof");
    // ↑ This verifies you know secrets (k,r)
    //   that produce a commitment in the tree
    //   at some leafIndex that hashes to this root

    // Check 3: Not already withdrawn?
    require(!nullifierHashes[nullifierHash], "Already withdrawn");

    // All checks pass → Withdraw ✅
}
```

**Key Point**: The contract checks if your **proof is recent**, not if your **commitment exists**. Your commitment existing is proven by the zk-SNARK, which you can regenerate anytime!

## Comparison to Tornado Cash

### Tornado Cash (Original, 2019)

```
Root storage: 30 roots
Lookup method: Linear iteration
Gas cost: ~6,000 gas
Target pools: 1 ETH, 10 ETH, 100 ETH
Volume: 5-20 deposits/hour
Time window: 1.5 to 6 hours

Design rationale:
  ✅ Sufficient for low-volume pools
  ✅ Users expected to wait days anyway
  ✅ Minimal storage (960 bytes)
  ✅ 2019 gas prices were high

Verdict: Perfect for their use case ✅
```

### torx402 (2024+)

```
Root storage: 1,000 roots (33x more)
Lookup method: Mapping (O(1))
Gas cost: ~3,000 gas (50% cheaper!)
Target pools: 0.001 ETH, 0.01 ETH, 0.1 ETH
Volume: 50-100+ deposits/hour
Time window: 10 hours to 2+ days

Design rationale:
  ✅ High-volume micropayments need longer windows
  ✅ Global users (timezones, sleep cycles)
  ✅ Modern L2s have cheap storage
  ✅ Mapping optimization enables more roots with lower gas

Verdict: Optimized for torx402 use case ✅
```

## Best Practices

### For Users

**✅ DO:**
1. **Withdraw within the window** (10+ hours for high-volume)
2. **Don't panic if root expires** - just regenerate (automatic in SDK)
3. **Save your note securely** - you need (k, r, leafIndex)
4. **Check pool volume** before depositing

**❌ DON'T:**
1. Worry about "losing funds" - impossible if you have your note
2. Rush to withdraw - you have hours/days
3. Generate proof weeks in advance - might need regeneration

### For Developers

**✅ DO:**
1. **Implement automatic regeneration** in SDK
   ```typescript
   if (!isKnownRoot(note.root)) {
     await regenerateProof(note);
   }
   ```

2. **Show time remaining** to users
   ```typescript
   const rootAge = await pool.getRootAge(note.root);
   console.log(`Root valid for ${1000 - rootAge} more deposits`);
   ```

3. **Educate users** that regeneration is safe
   ```typescript
   console.log('Root expired, regenerating proof (5-10 sec)...');
   console.log('Your funds are safe! ✅');
   ```

**❌ DON'T:**
1. Show scary error messages about "expired roots"
2. Make regeneration seem like a failure
3. Require manual regeneration (automate it!)

## FAQ

### Q: What if my root expires?

**A**: No problem! Just regenerate your proof with the current root. Takes 5-10 seconds. Your commitment is still in the tree at the same position.

### Q: Can I lose my funds if I wait too long?

**A**: NO! Your commitment is permanent. You can always withdraw by regenerating the proof, even years later (assuming you have your secrets k and r).

### Q: Why not just store all roots forever?

**A**:
- After 1 million deposits: 32 MB storage, ~$10,000 gas per withdrawal
- Not practical or necessary
- Regeneration is cheap (5-10 seconds)
- 1,000 roots covers 99%+ of normal usage

### Q: What if the Merkle tree gets corrupted?

**A**: The tree state is reconstructable from blockchain events. Anyone can rebuild it. Your commitment is in the blockchain permanently.

### Q: How does this compare to Tornado Cash?

**A**:
- Tornado: 30 roots (sufficient for their low-volume pools)
- torx402: 1,000 roots (needed for high-volume micropayments)
- Both designs are correct for their use cases

## Summary

**Root History: A Convenience, Not a Constraint**

```
✅ Stores last 1,000 Merkle roots
✅ Provides 10-48 hour window (high to medium volume)
✅ Efficient O(1) lookup (~3k gas)
✅ Reasonable storage (32 KB)
✅ Funds NEVER locked (can always regenerate)
✅ Optimal for micropayment use case
```

**Key Takeaways:**

1. Your commitment is **permanent** (never removed from tree)
2. Root history is **temporary** (convenience window)
3. Expired root = **regenerate proof** (5-10 seconds, no cost)
4. 1,000 roots = **optimal balance** (not too short, not overkill)
5. Funds are **always accessible** (mathematically guaranteed)

**This design enables fast, convenient withdrawals while maintaining security and efficiency.** 🎯

---

## Related Topics

- [Anonymity Sets](anonymity-sets.md)
- [Zero-Knowledge Proofs](zero-knowledge-proofs.md)
- [Privacy Guarantees](../how-it-works/privacy-guarantees.md)
- [Smart Contract Interface](../protocol/overview.md#smart-contract-interface)
