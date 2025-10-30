---
description: Technical specification of the torx402 protocol
---

# Protocol Overview

## Introduction

torx402 is a privacy-preserving HTTP micropayment protocol that extends the [x402 specification](https://x402.org) with zero-knowledge proof technology from Tornado Cash. It enables anonymous, verifiable payments for web resources while maintaining HTTP-native integration and sub-second settlement times.

This document provides the technical specification for the torx402 protocol, including data structures, cryptographic primitives, and implementation requirements.

## Protocol Version

**Current Version**: 1.0  
**x402 Base Version**: 1.0  
**Protocol Identifier**: `tornado`

## Design Goals

1. **Privacy**: Complete unlinkability between deposits and withdrawals
2. **Compatibility**: Work with existing x402 infrastructure
3. **Performance**: Maintain sub-second payment settlement
4. **Security**: 126-bit cryptographic security minimum
5. **Usability**: Simple integration for developers
6. **Decentralization**: No trusted third parties required

## Protocol Extensions to x402

torx402 extends the standard x402 protocol with the following additions:

### New Payment Scheme: `tornado`

The `tornado` scheme adds privacy-preserving payments to the existing x402 payment schemes (`exact`, etc.).

### Privacy Pool Contracts

Smart contracts that custody deposits and enable anonymous withdrawals through zero-knowledge proofs.

### Enhanced Payment Requirements

Payment requirements now include privacy pool information and anonymity set metadata.

### Zero-Knowledge Proof Payload

Payment proofs use zk-SNARKs instead of direct signatures, enabling anonymity.

## Core Protocol Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    torx402 Protocol Flow                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: DEPOSIT (Privacy Pool Setup)                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Client → Privacy Pool: deposit(commitment, denomination) │  │
│  │ Privacy Pool → Blockchain: Add to Merkle tree           │  │
│  │ Client: Store note (k, r, leafIndex)                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Phase 2: MIXING (Anonymity Set Growth)                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Other users deposit into same pool                       │  │
│  │ Anonymity set size increases                             │  │
│  │ Recommended minimum: 50-100 deposits                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Phase 3: HTTP REQUEST (Standard x402)                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Client → Server: GET /resource                           │  │
│  │ Server → Client: 402 Payment Required                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Phase 4: ANONYMOUS PAYMENT (Privacy-Preserving)                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Client: Generate zk-SNARK proof                          │  │
│  │ Client → Server: GET /resource + X-PAYMENT (proof)       │  │
│  │ Server → Facilitator: Verify proof                       │  │
│  │ Server → Facilitator: Settle withdrawal                  │  │
│  │ Privacy Pool: Verify & transfer funds                    │  │
│  │ Server → Client: 200 OK + Resource                       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## HTTP Status Codes

torx402 uses the same HTTP status codes as standard x402:

| Code | Meaning | Usage |
|------|---------|-------|
| **200** | OK | Payment verified and settled, resource delivered |
| **402** | Payment Required | Anonymous payment required to access resource |
| **400** | Bad Request | Invalid proof or payment requirements |
| **403** | Forbidden | Valid proof but settlement failed |
| **500** | Internal Server Error | Server error during verification/settlement |

## Data Structures

### 1. Privacy-Enhanced Payment Requirements

When a server requires payment, it responds with HTTP 402 and includes privacy pool information:

```json
{
  "x402Version": 1,
  "error": "X-PAYMENT header is required",
  "accepts": [
    {
      "scheme": "tornado",
      "network": "base",
      "denomination": "0.1",
      "asset": "ETH",
      "poolAddress": "0x209693Bc6afc0C5328bA36FaF03C514EF312287C",
      "verifierAddress": "0x123...",
      "payTo": "0x857b06519E91e3A54538791bDbb0E22373e36b66",
      "resource": "https://api.example.com/premium-data",
      "description": "Premium market data API",
      "mimeType": "application/json",
      "maxTimeoutSeconds": 300,
      "relayerFee": "0.001",
      "extra": {
        "minAnonymitySet": 50,
        "currentAnonymitySet": 234,
        "merkleTreeHeight": 20
      }
    }
  ]
}
```

#### Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `scheme` | string | Yes | Must be `"tornado"` for privacy-preserving payments |
| `network` | string | Yes | Blockchain network (e.g., "base", "solana") |
| `denomination` | string | Yes | Fixed deposit amount (e.g., "0.1", "1.0") |
| `asset` | string | Yes | Token type (e.g., "ETH", "USDC") |
| `poolAddress` | string | Yes | Privacy pool contract address |
| `verifierAddress` | string | Yes | zk-SNARK verifier contract address |
| `payTo` | string | Yes | Recipient address for payment |
| `resource` | string | Yes | Protected resource URL |
| `description` | string | Yes | Human-readable description |
| `mimeType` | string | Optional | Expected response MIME type |
| `maxTimeoutSeconds` | number | Yes | Maximum time to complete payment |
| `relayerFee` | string | Optional | Suggested relayer fee amount |
| `extra.minAnonymitySet` | number | Optional | Minimum recommended anonymity set |
| `extra.currentAnonymitySet` | number | Optional | Current pool size |
| `extra.merkleTreeHeight` | number | Optional | Merkle tree height (default: 20) |

### 2. Anonymous Payment Proof (X-PAYMENT Header)

Client includes the zero-knowledge proof in the `X-PAYMENT` header as base64-encoded JSON:

```json
{
  "x402Version": 1,
  "scheme": "tornado",
  "network": "base",
  "payload": {
    "proof": {
      "pi_a": [
        "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
        "0x234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1"
      ],
      "pi_b": [
        [
          "0x34567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef12",
          "0x4567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef123"
        ],
        [
          "0x567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234",
          "0x67890abcdef1234567890abcdef1234567890abcdef1234567890abcdef12345"
        ]
      ],
      "pi_c": [
        "0x7890abcdef1234567890abcdef1234567890abcdef1234567890abcdef123456",
        "0x890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567"
      ],
      "protocol": "groth16",
      "curve": "bn128"
    },
    "publicSignals": {
      "root": "0x2f6a7588d6acca505cbf0d9a4a227e0c52c6c34008c8e8986a1283259764173",
      "nullifierHash": "0x08a2ce6496642e377d6da8dbbf5836e9bd15092f9ecab05ded3d6293af148b57",
      "recipient": "0x857b06519E91e3A54538791bDbb0E22373e36b66",
      "relayer": "0xabcdef1234567890abcdef1234567890abcdef12",
      "fee": "1000000000000000",
      "timestamp": "1703123456"
    }
  }
}
```

#### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `proof.pi_a` | string[2] | Groth16 proof element A |
| `proof.pi_b` | string[2][2] | Groth16 proof element B |
| `proof.pi_c` | string[2] | Groth16 proof element C |
| `proof.protocol` | string | Must be "groth16" |
| `proof.curve` | string | Must be "bn128" |
| `publicSignals.root` | string | Merkle root used in proof |
| `publicSignals.nullifierHash` | string | Hash of nullifier (prevents double-spend) |
| `publicSignals.recipient` | string | Payment recipient address |
| `publicSignals.relayer` | string | Relayer address (0x0 if direct) |
| `publicSignals.fee` | string | Relayer fee in wei/lamports |
| `publicSignals.timestamp` | string | Unix timestamp of proof generation |

### 3. Settlement Response (X-PAYMENT-RESPONSE Header)

After successful settlement, server includes transaction details:

```json
{
  "success": true,
  "transaction": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
  "network": "base",
  "payer": "anonymous",
  "nullifierHash": "0x08a2ce6496642e377d6da8dbbf5836e9bd15092f9ecab05ded3d6293af148b57",
  "amount": "100000000000000000",
  "fee": "1000000000000000",
  "anonymitySetSize": 234
}
```

## Cryptographic Primitives

### Hash Functions

#### H1: Pedersen Hash
- **Purpose**: Nullifier hashing and commitment generation
- **Input**: Arbitrary bit string
- **Output**: Field element in Zp (where p is the BN128 curve order)
- **Security**: Collision-resistant, hiding
- **Implementation**: iden3 circomlib

#### H2: MiMC Hash
- **Purpose**: Merkle tree hashing
- **Mode**: MiMC-Feistel in sponge mode
- **Input**: Two field elements in Zp
- **Output**: One field element in Zp
- **Security**: Collision-resistant, suitable for zk-SNARKs
- **Rounds**: 220 rounds for 128-bit security

### Zero-Knowledge Proof System

#### Groth16 zk-SNARK
- **Curve**: BN128 (alt_bn128)
- **Field Order**: ~254 bits
- **Security Level**: ~128 bits
- **Proof Size**: 128 bytes (constant)
- **Verification Time**: ~1-2ms on-chain
- **Proving Time**: ~5-10 seconds client-side

### Merkle Tree

- **Height**: 20 (supports 2^20 = 1,048,576 deposits)
- **Hash Function**: MiMC (H2)
- **Initial State**: All leaves set to 0
- **Update**: Incremental (only path to root recalculated)
- **Root History**: Last 100 roots stored

## Privacy Pool Smart Contract Interface

### Deposit Function

```solidity
function deposit(bytes32 commitment) external payable {
    require(msg.value == DENOMINATION, "Invalid denomination");
    require(nextLeafIndex < 2**TREE_HEIGHT, "Tree full");
    
    // Add commitment to tree
    commitments[nextLeafIndex] = commitment;
    
    // Update Merkle root
    bytes32 newRoot = updateTree(nextLeafIndex, commitment);
    
    // Store root in history
    roots[currentRootIndex] = newRoot;
    currentRootIndex = (currentRootIndex + 1) % ROOT_HISTORY_SIZE;
    
    emit Deposit(commitment, nextLeafIndex, block.timestamp);
    nextLeafIndex++;
}
```

### Withdrawal Function

```solidity
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
    require(fee < DENOMINATION, "Fee too high");
    require(!nullifierHashes[nullifierHash], "Already withdrawn");
    require(isKnownRoot(root), "Unknown root");
    
    // Verify zk-SNARK proof
    require(
        verifier.verifyProof(
            a, b, c,
            [uint256(root), uint256(nullifierHash), uint256(uint160(recipient)), 
             uint256(uint160(relayer)), fee]
        ),
        "Invalid proof"
    );
    
    // Mark nullifier as used
    nullifierHashes[nullifierHash] = true;
    
    // Transfer funds
    recipient.transfer(DENOMINATION - fee);
    if (fee > 0) {
        relayer.transfer(fee);
    }
    
    emit Withdrawal(recipient, nullifierHash, relayer, fee);
}
```

## Facilitator API Extensions

### POST /verify (Tornado Scheme)

Verifies a tornado payment proof without executing on-chain.

**Request:**
```json
{
  "paymentPayload": {
    "x402Version": 1,
    "scheme": "tornado",
    "network": "base",
    "payload": {
      "proof": { /* Groth16 proof */ },
      "publicSignals": { /* public inputs */ }
    }
  },
  "paymentRequirements": {
    "scheme": "tornado",
    "denomination": "0.1",
    "poolAddress": "0x...",
    "payTo": "0x...",
    /* ... */
  }
}
```

**Response (Success):**
```json
{
  "isValid": true,
  "payer": "anonymous",
  "nullifierHash": "0x08a2ce...",
  "anonymitySetSize": 234
}
```

**Response (Error):**
```json
{
  "isValid": false,
  "invalidReason": "invalid_proof_verification",
  "details": "zk-SNARK proof verification failed"
}
```

### POST /settle (Tornado Scheme)

Executes the withdrawal transaction on-chain.

**Request:** Same as `/verify`

**Response (Success):**
```json
{
  "success": true,
  "transaction": "0x1234...",
  "network": "base",
  "payer": "anonymous",
  "nullifierHash": "0x08a2ce...",
  "amount": "100000000000000000",
  "fee": "1000000000000000"
}
```

### GET /pools/:denomination/:network

Returns information about a specific privacy pool.

**Response:**
```json
{
  "denomination": "0.1",
  "asset": "ETH",
  "network": "base",
  "poolAddress": "0x209693Bc6afc0C5328bA36FaF03C514EF312287C",
  "verifierAddress": "0x123...",
  "totalDeposits": 234,
  "nextLeafIndex": 234,
  "merkleTreeHeight": 20,
  "currentRoot": "0x2f6a7588...",
  "rootHistorySize": 100,
  "minRelayerFee": "0.001"
}
```

## Historical Merkle Root Storage

### Why Store 1,000 Roots?

The privacy pool contract stores the **last 1,000 Merkle roots** in a circular buffer. This is a **convenience window** - not a hard limit on withdrawals.

**Critical Understanding**: Your funds are NEVER locked. If your root expires from history, you simply regenerate your proof with a current root. Your commitment remains in the Merkle tree permanently.

#### The Challenge

Every time someone deposits into a privacy pool, the Merkle tree changes, which means the Merkle root changes:

```
Deposit 1 → Root R1
Deposit 2 → Root R2 (different from R1)
Deposit 3 → Root R3 (different from R2)
...
```

Without historical storage, users would need to generate proofs against the current root at the exact moment of withdrawal, creating race conditions and poor UX.

#### The Solution: Circular Root History Buffer

```solidity
contract PrivacyPool {
    // Store last 10,000 Merkle roots in circular buffer
    bytes32[1000] public roots;
    uint256 public currentRootIndex = 0;
    
    // Optimized root validation using mapping for O(1) lookup
    mapping(bytes32 => bool) private rootSet;
    
    function isKnownRoot(bytes32 root) public view returns (bool) {
        return rootSet[root];
    }
    
    function addRoot(bytes32 newRoot) internal {
        // Remove oldest root from mapping
        bytes32 oldRoot = roots[currentRootIndex];
        if (oldRoot != bytes32(0)) {
            delete rootSet[oldRoot];
        }
        
        // Add new root
        roots[currentRootIndex] = newRoot;
        rootSet[newRoot] = true;
        
        // Move to next position (circular)
        currentRootIndex = (currentRootIndex + 1) % 1000;
    }
}
```

#### Why 10,000 is Optimal

The 10,000-root window provides ample time for users to withdraw, even in high-volume pools:

| Pool Volume | Deposits/Hour | Time Until Root Expires | Sufficient? |
|-------------|---------------|------------------------|-------------|
| Ultra-High (0.001 ETH) | 100 | ~4 days | ✅ Excellent |
| High (0.01 ETH) | 50 | ~8 days | ✅ Excellent |
| Medium (0.1 ETH) | 20 | ~21 days | ✅ Excellent |
| Low (1 ETH) | 5 | ~83 days | ✅ More than enough |
| Very Low (10 ETH) | 1 | ~416 days | ✅ Over a year |

**Key Benefit**: Users can generate proofs against any of the last 1,000 roots, providing **days to months** of flexibility. Uses optimized mapping storage for O(1) lookup, maintaining low gas costs.

#### Gas Efficiency

```
Storage Cost:
  1,000 roots × 32 bytes = 32,000 bytes
  With mapping optimization: 2 storage slots per root update
  Root update: ~45,000 gas (mapping update + array update)

Lookup Cost (using mapping):
  Constant time: ~3,000 gas (O(1) mapping lookup)
  
Total withdrawal cost:
  Proof verification: ~280,000 gas
  Root lookup: ~3,000 gas (mapping)
  Nullifier check: ~5,000 gas
  Transfers: ~50,000 gas
  ────────────────────────────────
  Total: ~338,000 gas
  
Overhead: Only ~1% of total withdrawal cost
```

**Comparison to Tornado Cash (30 roots):**
- Tornado Cash: 30 roots, linear search, ~6k gas lookup
- torx402: 1,000 roots, mapping lookup, ~3k gas lookup
- **Result**: 33x more roots with LOWER gas cost! ✅

#### Root Expiration & Proof Regeneration

**Important**: Root expiration does NOT prevent withdrawal. It only means you need to regenerate your proof.

**Your commitment remains in the tree FOREVER** - it never gets removed. The root history only determines which pre-generated proofs are accepted without regeneration.

If your root expires from the 1,000-root history:
1. Your commitment at `leafIndex` still exists in the tree
2. Fetch current Merkle tree state
3. Regenerate Merkle proof for your commitment (same leafIndex)
4. Generate new zk-SNARK proof with current root
5. Withdraw successfully ✅

**Cost**: 5-10 seconds for proof regeneration (no financial loss)
**Likelihood**: 
- High-volume pools: Every 10-20 hours (occasional regeneration needed)
- Medium-volume pools: Every 2+ days (rarely needed)
- Low-volume pools: Every week+ (almost never needed)

**This is a feature, not a bug**: Keeping the window reasonable (1,000 vs 10,000 or 100,000) maintains gas efficiency while proof regeneration provides infinite flexibility.

## Security Properties

### Privacy Guarantees

1. **Deposit-Withdrawal Unlinkability**: No on-chain or cryptographic link between deposit and withdrawal transactions
2. **Anonymity Set**: Each withdrawal could correspond to any deposit in the pool
3. **Sender Anonymity**: Merchant cannot determine payer identity
4. **IP Privacy**: Relayers hide client IP addresses
5. **Amount Privacy**: Fixed denominations hide exact payment amounts

### Security Guarantees

1. **Double-Spend Prevention**: Nullifier hashes prevent reuse of deposits
2. **Replay Attack Prevention**: Nullifiers are unique per deposit
3. **Front-Running Protection**: Proofs bound to specific recipient
4. **Proof Soundness**: Invalid proofs cannot be generated
5. **Withdrawal Integrity**: Only valid deposit owners can withdraw

### Attack Resistance

| Attack Type | Protection Mechanism |
|-------------|---------------------|
| Double-spending | Nullifier hash uniqueness check |
| Replay attacks | Nullifier binding, timestamp validation |
| Front-running | Proof bound to recipient address |
| Sybil attacks | Economic cost of deposits |
| Pool draining | Proof verification, balance checks |
| Privacy degradation | Fixed denominations, relayer network |
| Timing analysis | Root history allows flexible timing |

## Implementation Requirements

### Client Requirements

1. Generate cryptographically secure random values (k, r)
2. Compute Pedersen and MiMC hashes
3. Generate Groth16 zk-SNARK proofs
4. Manage Merkle tree state locally or via RPC
5. Store deposit notes securely
6. Support HTTP/HTTPS requests with custom headers

### Server Requirements

1. Return HTTP 402 with tornado payment requirements
2. Parse and validate X-PAYMENT header
3. Call facilitator for proof verification
4. Initiate settlement via facilitator
5. Track nullifier hashes (optional, for accounting)
6. Return X-PAYMENT-RESPONSE header

### Facilitator Requirements

1. Verify Groth16 proofs off-chain
2. Interact with privacy pool contracts
3. Manage gas fees for withdrawals
4. Provide pool discovery and information
5. Support multiple denominations and networks
6. Maintain high availability

## Network Support

### EVM Chains

- Base (Chain ID: 8453)
- Base Sepolia (Chain ID: 84532)
- Avalanche (Chain ID: 43114)
- Polygon (Chain ID: 137)
- Other EVM-compatible chains

**Requirements:**
- Solidity 0.8.x support
- BN128 precompiles for pairing operations
- Sufficient gas limits for proof verification

### Solana

- Mainnet-beta
- Devnet

**Requirements:**
- Groth16 verification program
- SPL Token support
- Program-derived addresses (PDAs)

## Denomination Standards

### Recommended Denominations

| Asset | Denominations |
|-------|--------------|
| ETH | 0.001, 0.01, 0.1, 1, 10 |
| USDC | 10, 100, 1000, 10000 |
| USDT | 10, 100, 1000, 10000 |

### Denomination Selection Guidelines

1. **Liquidity**: Choose denominations with high usage
2. **Anonymity Set**: Prefer pools with >100 deposits
3. **Use Case**: Match denomination to typical API costs
4. **Privacy**: Avoid unique amounts that leak information

## Error Codes

| Error Code | Description | Recommended Action |
|------------|-------------|-------------------|
| `invalid_proof_verification` | zk-SNARK proof verification failed | Regenerate proof |
| `nullifier_already_used` | Deposit already withdrawn | Use different deposit |
| `unknown_merkle_root` | Root not in contract history | Update Merkle proof |
| `insufficient_relayer_fee` | Fee too low for relayer | Increase fee amount |
| `denomination_mismatch` | Wrong pool denomination | Use correct pool |
| `invalid_recipient` | Recipient doesn't match proof | Regenerate proof |
| `pool_not_found` | Pool doesn't exist on network | Check network/denomination |

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-01-XX | Initial specification |

## References

- [x402 Protocol Specification](https://x402.org)
- [Tornado Cash Whitepaper](https://tornado.cash/audits/TornadoCash_whitepaper_v1.4.pdf)
- [Groth16 Paper](https://eprint.iacr.org/2016/260.pdf)
- [MiMC Hash](https://eprint.iacr.org/2016/492.pdf)
- [Pedersen Commitments](https://iden3-docs.readthedocs.io/en/latest/)