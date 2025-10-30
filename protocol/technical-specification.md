---
description: Final technical specifications for torx402 privacy-preserving HTTP micropayment protocol
---

# torx402 Technical Specifications

**Version**: 1.0
**Last Updated**: October 2025

---

## Protocol Overview

**torx402** is a privacy-preserving HTTP micropayment protocol that combines x402 (HTTP 402 status code micropayments) with Tornado Cash's zero-knowledge proof technology to enable anonymous, instant, and verifiable payments for web APIs, digital content, and AI agent commerce.

---

## Core Specifications

### Merkle Tree Configuration

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Tree Height** | **32** | 4.3 billion deposit capacity, matches Tornado Cash proven design |
| **Maximum Deposits** | **4,294,967,296** | Effectively unlimited for realistic usage |
| **Hash Function** | **MiMC-Sponge** | zk-SNARK friendly, efficient in circuits |
| **Initial Leaf Value** | **0** | All leaves start as zero, filled left-to-right |
| **Leaf Update** | **Incremental only** | Leaves never removed or modified after addition |

### Root History Configuration

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Root Storage** | **10,000 roots** | Provides ~1 hour window at 10k deposits/hour |
| **Storage Structure** | **Circular buffer** | Constant storage, oldest overwritten |
| **Lookup Method** | **Mapping (O(1))** | ~3k gas vs ~200k for iteration |
| **Storage Size** | **320 KB** | Acceptable on L2s, BSC, Solana |

### Volume Assumptions

| Scenario | Deposits/Hour | Time Window (10k roots) | Status |
|----------|---------------|------------------------|--------|
| **Conservative (Target)** | 10,000 | ~1 hour | Primary design target ✅ |
| **High** | 100,000 | ~6 minutes | Peak scenario (auto-regeneration) |
| **Medium** | 1,000 | ~10 hours | Slower pools |
| **Low** | 100 | ~4 days | Large denominations |

**Design Target**: 10,000 deposits/hour is the conservative estimate for high-volume micropayment pools (0.001-0.01 ETH denominations).

---

## Comparison to Tornado Cash

### Tornado Cash (Original, 2019)

| Specification | Value |
|---------------|-------|
| **Root Storage** | 30 roots |
| **Tree Height** | 32 |
| **Lookup Method** | Linear iteration (do-while loop) |
| **Gas Cost (root check)** | ~6,000 gas |
| **Target Pools** | 0.1, 1, 10, 100 ETH (large denominations) |
| **Typical Volume** | 5-20 deposits/hour (low frequency) |
| **Time Window** | 1.5-6 hours at typical volume |
| **Target Network** | Ethereum mainnet |

**Why 30 roots worked for Tornado**:
- Low volume (5-20/hour)
- Large denominations (users wait days for anonymity)
- 30 roots × 5-20 deposits/hour = 1.5-6 hour window (sufficient)

### torx402 (2024+)

| Specification | Value |
|---------------|-------|
| **Root Storage** | 10,000 roots (333x more) |
| **Tree Height** | 32 (same as Tornado) |
| **Lookup Method** | Mapping O(1) (optimized) |
| **Gas Cost (root check)** | ~3,000 gas (50% cheaper!) |
| **Target Pools** | 0.001, 0.01, 0.1 ETH (micropayments) |
| **Target Volume** | 10,000 deposits/hour (conservative) |
| **Peak Volume** | 100,000+ deposits/hour |
| **Time Window** | ~1 hour at target volume |
| **Target Networks** | L2s (Base, Arbitrum), BSC, Solana |

**Why torx402 needs 333x more roots**:
- High volume (10,000/hour = 500-2000x more than Tornado)
- Micropayments = constant, high-frequency usage
- AI agents operating 24/7 globally
- Need 10,000 roots to maintain ~1 hour window
- L2 gas is cheap enough to afford more storage

---

## Smart Contract Interface

### Privacy Pool Contract

```solidity
contract PrivacyPool {
    // Configuration
    uint256 public immutable DENOMINATION;
    uint32 public constant TREE_HEIGHT = 32;
    uint32 public constant ROOT_HISTORY_SIZE = 10000;

    // Merkle tree state
    mapping(uint256 => bytes32) public commitments;
    uint256 public nextLeafIndex;

    // Root history (circular buffer)
    bytes32[10000] public roots;
    mapping(bytes32 => bool) private rootSet;  // O(1) lookup
    uint256 public currentRootIndex;

    // Nullifier tracking (prevent double-spend)
    mapping(bytes32 => bool) public nullifierHashes;

    // Verifier contract
    IVerifier public verifier;

    /**
     * @dev Deposit funds with commitment
     * @param commitment Hash of (nullifier || secret)
     */
    function deposit(bytes32 commitment) external payable {
        require(msg.value == DENOMINATION, "Invalid denomination");
        require(nextLeafIndex < 2**TREE_HEIGHT, "Tree full");
        require(!commitments[nextLeafIndex], "Duplicate commitment");

        // Add commitment to tree
        commitments[nextLeafIndex] = commitment;

        // Update Merkle root (incremental)
        bytes32 newRoot = _updateTree(nextLeafIndex, commitment);

        // Add root to history
        _addRoot(newRoot);

        emit Deposit(commitment, nextLeafIndex, block.timestamp);
        nextLeafIndex++;
    }

    /**
     * @dev Withdraw with zero-knowledge proof
     * @param proof Groth16 proof elements (a, b, c)
     * @param root Merkle root (must be in last 10,000)
     * @param nullifierHash Hash of nullifier (prevents double-spend)
     * @param recipient Address to receive funds
     * @param relayer Address to receive fee (or zero address)
     * @param fee Amount for relayer
     */
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
        require(fee < DENOMINATION, "Fee exceeds denomination");
        require(!nullifierHashes[nullifierHash], "Already withdrawn");
        require(isKnownRoot(root), "Unknown Merkle root");

        // Verify zk-SNARK proof
        require(
            verifier.verifyProof(
                a, b, c,
                [uint256(root), uint256(nullifierHash), uint256(uint160(recipient)),
                 uint256(uint160(relayer)), fee]
            ),
            "Invalid proof"
        );

        // Mark nullifier as spent
        nullifierHashes[nullifierHash] = true;

        // Transfer funds
        recipient.transfer(DENOMINATION - fee);
        if (fee > 0) {
            relayer.transfer(fee);
        }

        emit Withdrawal(recipient, nullifierHash, relayer, fee);
    }

    /**
     * @dev Check if root is in recent history
     * @param root The Merkle root to check
     * @return bool True if root is in last 10,000 roots
     */
    function isKnownRoot(bytes32 root) public view returns (bool) {
        return rootSet[root];  // O(1) constant time lookup
    }

    /**
     * @dev Add new root to circular buffer
     * @param newRoot The new Merkle root
     */
    function _addRoot(bytes32 newRoot) internal {
        // Remove oldest root from mapping
        bytes32 oldRoot = roots[currentRootIndex];
        if (oldRoot != bytes32(0)) {
            delete rootSet[oldRoot];
        }

        // Add new root to both structures
        roots[currentRootIndex] = newRoot;
        rootSet[newRoot] = true;

        // Advance circular buffer pointer
        currentRootIndex = (currentRootIndex + 1) % ROOT_HISTORY_SIZE;
    }
}
```

---

## Cryptographic Primitives

### Hash Functions

| Function | Algorithm | Purpose | Properties |
|----------|-----------|---------|------------|
| **H1** | Pedersen Hash | Nullifier hashing, commitments | Collision-resistant, hiding |
| **H2** | MiMC-Sponge | Merkle tree hashing | zk-SNARK friendly, efficient |

### Zero-Knowledge Proof System

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Proof System** | Groth16 | Industry standard, constant-size proofs |
| **Curve** | BN128 (alt_bn128) | Supported by EVM precompiles |
| **Field Order** | ~254 bits | Prime order group |
| **Security Level** | ~128 bits | Computational security |
| **Proof Size** | 128 bytes | Constant (3 G1 + 1 G2 elements) |
| **Proving Time** | 5-10 seconds | Client-side (can be GPU accelerated) |
| **Verification Time** | ~1-2 ms | On-chain verification |
| **Verification Gas** | ~400,000 gas | Height 32 tree (32 hash operations) |

---

## Gas Costs by Network

### Target Networks

torx402 is optimized for **Layer 2 chains, BSC, and Solana** where gas is cheap enough for height 32 Merkle proofs.

#### Base / Arbitrum / Optimism (L2s)

| Operation | Gas Used | Cost (0.1 gwei) | Cost (1 gwei) |
|-----------|----------|-----------------|---------------|
| **Deposit** | ~250,000 | $0.00006 | $0.0006 |
| **Withdrawal** | ~458,000 | $0.0001 | $0.001 |
| **Root Check** | ~3,000 | $0.0000007 | $0.000007 |

**Verdict**: ✅ Extremely cheap - height 32 is no problem

#### BSC (Binance Smart Chain)

| Operation | Gas Used | Cost (3 gwei) | Cost (5 gwei) |
|-----------|----------|---------------|---------------|
| **Deposit** | ~250,000 | $0.002 | $0.003 |
| **Withdrawal** | ~458,000 | $0.003 | $0.005 |
| **Root Check** | ~3,000 | $0.00002 | $0.00004 |

**Verdict**: ✅ Very cheap - height 32 is affordable

#### Solana

| Operation | Compute Units | Cost | Notes |
|-----------|---------------|------|-------|
| **Deposit** | ~200,000 CU | ~$0.00002 | Native program implementation |
| **Withdrawal** | ~400,000 CU | ~$0.00004 | Groth16 verification optimized |
| **Root Check** | ~5,000 CU | ~$0.0000005 | Account lookup |

**Verdict**: ✅ Negligible cost - height 32 is trivial

#### Ethereum Mainnet (Not Primary Target)

| Operation | Gas Used | Cost (50 gwei) | Cost (100 gwei) |
|-----------|----------|----------------|-----------------|
| **Deposit** | ~250,000 | $0.30 | $0.60 |
| **Withdrawal** | ~458,000 | $0.55 | $1.10 |

**Verdict**: ⚠️ More expensive but still viable for larger micropayments

---

## Withdrawal Gas Breakdown (Height 32)

```
Total Withdrawal Cost: ~458,000 gas

Breakdown:
  Groth16 Proof Verification: ~400,000 gas
    - BN128 pairing operations
    - 32 MiMC hash verifications
    - Public input validation

  Root Lookup (mapping): ~3,000 gas
    - Single mapping read (O(1))
    - 333x faster than iterating 10,000 roots!

  Nullifier Check: ~5,000 gas
    - Mapping read and write
    - Prevent double-spending

  Fund Transfers: ~50,000 gas
    - Transfer to recipient
    - Transfer to relayer (if applicable)

Root History Overhead: < 1% of total cost ✅
```

---

## Volume Projections & Scaling

### Conservative Estimate (Primary Design Target)

```
10,000 deposits/hour per popular pool:
  - Micropayment denominations (0.001, 0.01 ETH)
  - AI agents making frequent payments
  - Global usage across timezones
  - Multiple services accepting torx402

Time window with 10,000 roots: ~1 hour ✅
User experience: Withdraw within hour (reasonable)
Regeneration: Rare (most users withdraw within 1 hour)
```

### Peak Estimate (High Growth Scenario)

```
100,000 deposits/hour per popular pool:
  - Mass AI agent adoption
  - Thousands of merchants accepting payments
  - Global micropayment infrastructure
  - Content, APIs, IoT, DePIN usage

Time window with 10,000 roots: ~6 minutes ⚠️
User experience: Auto-regeneration in SDK
Regeneration: More frequent (handled automatically)
Future upgrade: Increase to 100,000 roots if sustained
```

### Extreme Peak (Theoretical Maximum)

```
1,000,000 deposits/hour per pool:
  - Visa-scale transaction volume (16% of Visa globally!)
  - Would require significant protocol upgrades
  - Pool sharding (multiple pools per denomination)
  - Or 1,000,000 root history (32 MB storage)

Currently: Not designed for this scale
Future: Can be addressed with sharding or larger root history
```

---

## Proof Regeneration

### When Regeneration is Needed

```
Root expires when:
  (Current deposits - Your deposit) > 10,000

Example at 10k deposits/hour:
  Deposit at 10:00 AM (Root R5000)
  Current time: 11:00 AM (Root R15000)
  Elapsed deposits: 10,000
  Root R5000 evicted from history
  → Need regeneration
```

### Regeneration Process

```typescript
// Automatic in SDK - user doesn't see this
async function ensureValidProof(note) {
    const rootValid = await pool.isKnownRoot(note.root);

    if (rootValid) {
        // Fast path: use cached proof
        return note.proof;
    }

    // Slow path: regenerate with current root
    console.log('Regenerating proof...');
    const currentState = await pool.getMerkleTreeState();
    const merkleProof = currentState.getProof(note.leafIndex);

    const newProof = await generateZKProof({
        nullifier: note.k,
        secret: note.r,
        leafIndex: note.leafIndex,
        merkleProof: merkleProof,
        root: currentState.root,  // Current root
        recipient: recipient,
        relayer: relayer,
        fee: fee,
    });

    return newProof;
}

Time cost: 5-10 seconds
Financial cost: $0
Handled: Automatically by SDK
```

### Regeneration Frequency Estimates

| Volume | Window Duration | Likelihood if Withdraw After... |
|--------|-----------------|--------------------------------|
| 10k/hour | 1 hour | 1 hour: 50% | 6 hours: 100% | 24 hours: 100% |
| 100k/hour | 6 minutes | 10 min: 90% | 1 hour: 100% | 6 hours: 100% |
| 1k/hour | 10 hours | 12 hours: 20% | 24 hours: 70% | Week: 100% |

**SDK Behavior**: Auto-regenerate when needed, transparent to user

---

## Network-Specific Implementation

### EVM Chains (Base, Arbitrum, Optimism, Polygon)

```solidity
// Solidity 0.8.x
contract PrivacyPoolEVM {
    using SafeMath for uint256;

    uint32 public constant TREE_HEIGHT = 32;
    bytes32[10000] public roots;
    mapping(bytes32 => bool) private rootSet;

    // EVM uses precompiled contracts for BN128 pairing
    // Groth16 verification via Verifier.sol (circom output)
}
```

**Gas Costs on Base L2 (0.1 gwei)**:
- Deposit: ~$0.00006
- Withdrawal: ~$0.0001
- **Total round-trip: ~$0.00016** ✅

### BSC (Binance Smart Chain)

```solidity
// Same Solidity implementation as EVM
contract PrivacyPoolBSC {
    uint32 public constant TREE_HEIGHT = 32;
    bytes32[10000] public roots;
    mapping(bytes32 => bool) private rootSet;

    // BSC is EVM-compatible, uses same verifier
}
```

**Gas Costs on BSC (3 gwei)**:
- Deposit: ~$0.002
- Withdrawal: ~$0.003
- **Total round-trip: ~$0.005** ✅

### Solana

```rust
// Anchor/Native Rust program
pub struct PrivacyPool {
    pub denomination: u64,
    pub tree_height: u8,  // 32
    pub next_leaf_index: u64,
    pub current_root_index: u64,
    pub root_history_size: u64,  // 10,000
}

// Root storage in separate account (PDA)
pub struct RootHistory {
    pub roots: [Hash; 10000],
}

// Nullifier tracking in separate account
pub struct NullifierSet {
    pub nullifiers: HashMap<Hash, bool>,
}
```

**Compute Unit Costs**:
- Deposit: ~200,000 CU (~$0.00002)
- Withdrawal: ~400,000 CU (~$0.00004)
- **Total round-trip: ~$0.00006** ✅

---

## Denomination Pools

### Recommended Denominations

| Network | Asset | Denominations | Expected Volume |
|---------|-------|---------------|-----------------|
| **Base/Arbitrum** | ETH | 0.001, 0.01, 0.1, 1 | High (10k-100k/hr) |
| **Base** | USDC | 1, 10, 100, 1000 | High (10k-100k/hr) |
| **BSC** | BNB | 0.01, 0.1, 1 | Medium (1k-10k/hr) |
| **BSC** | USDT | 10, 100, 1000 | Medium (1k-10k/hr) |
| **Solana** | SOL | 0.1, 1, 10 | High (10k-100k/hr) |
| **Solana** | USDC | 1, 10, 100 | High (10k-100k/hr) |

### Pool Lifetime Estimate

```
At 10,000 deposits/hour (target volume):
  4.3 billion capacity ÷ 10,000/hour = 430,000 hours = 49 years ✅

At 100,000 deposits/hour (peak):
  4.3 billion capacity ÷ 100,000/hour = 43,000 hours = 4.9 years ✅

Height 32 provides DECADES of capacity even at extreme volume
```

---

## Security Properties

### Privacy Guarantees

1. **Deposit-Withdrawal Unlinkability**: Zero-knowledge proofs ensure no cryptographic or on-chain link
2. **Anonymity Set**: Each withdrawal could correspond to any deposit in the pool
3. **Sender Anonymity**: Merchant cannot determine payer identity
4. **IP Privacy**: Relayers hide client IP addresses (optional)
5. **Amount Privacy**: Fixed denominations hide exact payment amounts

### Security Guarantees

1. **Double-Spend Prevention**: Nullifier hash uniqueness enforced on-chain
2. **Replay Attack Prevention**: Nullifiers are unique per deposit
3. **Front-Running Protection**: Proofs cryptographically bound to recipient
4. **Proof Soundness**: Invalid proofs cannot be generated (cryptographic guarantee)
5. **Withdrawal Integrity**: Only deposit owner (with k, r) can withdraw
6. **No Fund Loss**: Commitment permanent in tree, always withdrawable

### Cryptographic Security

- **zk-SNARK Security**: ~128 bits (Groth16 over BN128)
- **Hash Security**: 126 bits minimum (Pedersen, MiMC)
- **Nullifier Entropy**: 248 bits
- **Secret Entropy**: 248 bits
- **Combined Entropy**: 496 bits

---

## Implementation Requirements

### Client SDK Requirements

**Must Support:**
- Generate cryptographically secure random values (k, r) - 248 bits each
- Compute Pedersen hash (H1) for commitments and nullifiers
- Compute MiMC hash (H2) for Merkle tree operations
- Generate Groth16 zk-SNARK proofs (height 32 circuit)
- Manage Merkle tree state (local cache or via RPC)
- Store deposit notes securely (encrypted storage)
- Auto-regenerate proofs when roots expire
- Support HTTP requests with X-PAYMENT header

**Recommended Libraries:**
- circomlib (hash functions)
- snarkjs (proof generation)
- ethers.js / web3.js (EVM interaction)
- @solana/web3.js (Solana interaction)

### Server/Merchant Requirements

**Must Support:**
- Return HTTP 402 with tornado payment requirements
- Parse and validate X-PAYMENT header (base64 JSON)
- Call facilitator for proof verification
- Initiate settlement via facilitator
- Return X-PAYMENT-RESPONSE header
- Track nullifiers for accounting (optional)

**Recommended Middleware:**
- torx402-express (Express.js)
- torx402-hono (Hono)
- torx402-fastapi (FastAPI)
- torx402-flask (Flask)

### Facilitator Requirements

**Must Support:**
- Verify Groth16 proofs off-chain (pre-validation)
- Interact with privacy pool contracts on multiple chains
- Manage gas fees for withdrawals
- Provide pool discovery and information APIs
- Support multiple denominations and networks
- Handle 10,000+ requests/hour (high volume)
- Maintain low latency (<100ms verification)

---

## Network Support

### Supported Blockchains

| Network | Type | Chain ID | Status | Gas Cost |
|---------|------|----------|--------|----------|
| **Base** | L2 | 8453 | ✅ Primary | ~$0.0001/tx |
| **Base Sepolia** | L2 Testnet | 84532 | ✅ Testing | Free |
| **Arbitrum** | L2 | 42161 | 🔄 Planned | ~$0.0001/tx |
| **Optimism** | L2 | 10 | 🔄 Planned | ~$0.0001/tx |
| **Polygon** | Sidechain | 137 | 🔄 Planned | ~$0.001/tx |
| **BSC** | Alt L1 | 56 | ✅ Primary | ~$0.003/tx |
| **Solana** | Alt L1 | - | ✅ Primary | ~$0.00004/tx |
| **Ethereum** | L1 | 1 | ⚠️ Secondary | ~$0.50/tx |

**Priority**: L2s (Base, Arbitrum), BSC, and Solana - all have cheap enough gas for height 32 proofs.

---

## Data Structures

### Deposit Note Format

```
tornado-eth-0.01-base-0xABCDEF123456...

Structure:
  protocol: tornado
  asset: eth
  denomination: 0.01
  network: base
  secrets: 0xABCDEF123456... (encoded k, r, leafIndex)
```

### Payment Proof Format (X-PAYMENT Header)

```json
{
  "x402Version": 1,
  "scheme": "tornado",
  "network": "base",
  "payload": {
    "proof": {
      "pi_a": ["0x...", "0x..."],
      "pi_b": [["0x...", "0x..."], ["0x...", "0x..."]],
      "pi_c": ["0x...", "0x..."],
      "protocol": "groth16",
      "curve": "bn128"
    },
    "publicSignals": {
      "root": "0x...",
      "nullifierHash": "0x...",
      "recipient": "0x...",
      "relayer": "0x...",
      "fee": "1000000000000000"
    }
  }
}
```

---

## Performance Targets

### Client Performance

| Operation | Target | Notes |
|-----------|--------|-------|
| **Proof Generation** | < 10 seconds | Can be GPU accelerated to ~2 seconds |
| **Merkle Proof Fetch** | < 500ms | Cached locally or via fast RPC |
| **Deposit Transaction** | < 2 seconds | Network confirmation time |
| **Total Deposit Flow** | < 15 seconds | End-to-end user experience |

### Server Performance

| Operation | Target | Notes |
|-----------|--------|-------|
| **Proof Verification (off-chain)** | < 100ms | Facilitator pre-validation |
| **Settlement** | < 2 seconds | Blockchain confirmation |
| **Total Payment Flow** | < 5 seconds | From request to resource delivery |

### Network Throughput

| Network | Deposits/Second | Withdrawals/Second | Notes |
|---------|----------------|-------------------|-------|
| **Base L2** | ~50 | ~30 | L2 block time ~2s |
| **Arbitrum** | ~50 | ~30 | Similar to Base |
| **BSC** | ~20 | ~15 | 3s block time |
| **Solana** | ~1,000 | ~500 | 400ms slots |

**System Capacity**: Limited by blockchain throughput, not protocol design

---

## Future Upgrades

### Planned Enhancements

1. **Dynamic Root History** (Q2 2024)
   - Governance-controlled root history size
   - Increase from 10k → 100k if volume exceeds projections
   - No contract migration needed

2. **Recursive Proofs** (Q3 2024)
   - Reduce proof size (128 bytes → 64 bytes)
   - Faster verification (~200k gas vs ~400k)
   - Better for height 32 trees

3. **Batch Withdrawals** (Q3 2024)
   - Withdraw multiple deposits in one transaction
   - Amortize gas costs across multiple payments
   - Better for high-frequency users

4. **Cross-Chain Deposits** (Q4 2024)
   - Deposit on Base, withdraw on Solana
   - Unified liquidity pools
   - Better privacy (larger anonymity sets)

5. **Pool Sharding** (If needed at extreme scale)
   - Multiple pools per denomination
   - Load balancing across pools
   - For 1M+ deposits/hour scenarios

---

## Comparison Summary

### torx402 vs Tornado Cash

| Specification | Tornado Cash | torx402 | Difference |
|---------------|--------------|---------|------------|
| **Root History** | 30 | 10,000 | 333x more |
| **Tree Height** | 32 | 32 | Same ✅ |
| **Capacity** | 4.3B | 4.3B | Same ✅ |
| **Lookup Method** | Iteration | Mapping | 6x faster |
| **Gas (root check)** | ~6k | ~3k | 50% cheaper |
| **Target Volume** | 5-20/hr | 10k/hr | 500-2000x more |
| **Window (target vol)** | 1.5-6 hours | ~1 hour | Scaled appropriately |
| **Use Case** | Large transfers | Micropayments | Different markets |
| **Networks** | ETH mainnet | L2s, BSC, Solana | Multi-chain |

### Key Insight

Both protocols use the **same fundamental design** (height 32 tree, circular root buffer), but torx402 scales the root history appropriately for **500-2000x higher transaction volume** expected from micropayments and AI agent commerce.

---

## Design Decisions Summary

### Why Height 32?

✅ **4.3 billion capacity** - Lasts 49 years at 10k deposits/hour
✅ **Proven design** - Tornado Cash battle-tested since 2019
✅ **Industry standard** - All privacy protocols use height 32
✅ **Sufficient capacity** - Effectively unlimited for realistic usage
✅ **Affordable on L2s** - Gas cost negligible on target networks

**Rejected Alternatives:**
- Height 20: Only 1M capacity, fills in 100 hours at 10k/hour ❌
- Height 64: Unnecessary (490 years at 1M deposits/hour), larger proofs ❌

### Why 10,000 Roots?

✅ **~1 hour window** - At 10k deposits/hour (target volume)
✅ **Reasonable storage** - 320 KB (affordable on L2s)
✅ **O(1) lookup** - Mapping optimization, ~3k gas
✅ **Rare regeneration** - Most users withdraw within 1 hour
✅ **Auto-recovery** - SDK handles peak volume regeneration

**Rejected Alternatives:**
- 30 roots: Only 2 minutes at 10k/hour (far too short) ❌
- 100 roots: Only 6 minutes at 10k/hour (still too short) ❌
- 1,000 roots: Only 6 minutes at 10k/hour (tight) ❌
- 100,000 roots: 10 hours at 10k/hour (overkill, wastes storage) ❌

### Why L2s/BSC/Solana?

✅ **Cheap gas** - Height 32 proofs cost $0.0001-$0.003
✅ **High throughput** - Can handle 10k-100k deposits/hour
✅ **Fast finality** - 2-3 second confirmations
✅ **Growing ecosystems** - Where AI agents will operate

**Ethereum Mainnet**: Secondary target (more expensive but still viable)

---

## Conclusion

**torx402 Final Specifications:**

- **Tree Height**: 32 (4.3 billion capacity)
- **Root History**: 10,000 roots (~1 hour at target volume)
- **Target Volume**: 10,000 deposits/hour (conservative)
- **Peak Volume**: 100,000+ deposits/hour (with auto-regeneration)
- **Networks**: L2s (Base, Arbitrum), BSC, Solana
- **Gas Cost**: $0.0001-$0.003 per transaction

**Design Philosophy**: Scale Tornado Cash's proven architecture for 500-2000x higher volume micropayment usage, optimized for cheap L2/BSC/Solana gas costs.

**Inherited from Tornado Cash**: Height 32 tree structure, incremental updates, nullifier scheme, zk-SNARK verification

**Innovations for torx402**: 10,000 root history (333x more), mapping optimization (O(1)), HTTP 402 integration, multi-chain support, micropayment focus

**Status**: Ready for implementation and deployment ✅

---

**Last Updated**: January 2024
**Next Review**: After mainnet deployment and volume analysis
