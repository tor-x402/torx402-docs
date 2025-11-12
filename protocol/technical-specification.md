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

### torx402

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

## Network Support

### Supported Blockchains

| Network | Type | Chain ID | Status | Gas Cost |
|---------|------|----------|--------|----------|
| **Ethereum Sepolia** | L1 Testnet | 11155111 | ✅ Testing | Faucet |
| **Base Sepolia** | L2 Testnet | 84532 | ✅ Testing | Free |
| **BSC Testnet** | Alt L1 Testnet | 97 | ✅ Testing | Faucet |
| **Polygon Amoy Testnet** | Sidechain Testnet | 80002 | ✅ Testing | Faucet |

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

1. **Dynamic Root History** (Q4 2025)
   - Governance-controlled root history size
   - Increase from 10k → 100k if volume exceeds projections
   - No contract migration needed

2. **Recursive Proofs** (Q1 2026)
   - Reduce proof size (128 bytes → 64 bytes)
   - Faster verification (~200k gas vs ~400k)
   - Better for height 32 trees

3. **Batch Withdrawals** (Q1 2026)
   - Withdraw multiple deposits in one transaction
   - Amortize gas costs across multiple payments
   - Better for high-frequency users

4. **Cross-Chain Deposits** (Q2 2026)
   - Deposit on Base, withdraw on Solana
   - Unified liquidity pools
   - Better privacy (larger anonymity sets)

5. **Pool Sharding** (If needed at extreme scale)
   - Multiple pools per denomination
   - Load balancing across pools
   - For 1M+ deposits/hour scenarios

---
**Last Updated**: October 2025
**Next Review**: After mainnet deployment and volume analysis
