---
description: Understanding the torx402 system architecture and components
---

# Architecture Overview

## Introduction

torx402's architecture combines two proven protocols—x402 for HTTP-native micropayments and Tornado Cash for zero-knowledge privacy—into a unified system that enables anonymous, verifiable payments over standard HTTP infrastructure.

This page provides a comprehensive overview of the system architecture, core components, and how they interact to provide privacy-preserving micropayments.

## Instant Withdrawal: The High-Volume Advantage

**Can you withdraw in under 2 seconds?** YES! When pools have large existing anonymity sets.

```
⚡ INSTANT WITHDRAWAL SCENARIO (< 2 seconds)

Ultra-High Volume Pool: 0.001 ETH on Base
Current State: 1,247 deposits already in pool

┌─────────────────────────────────────────────────────────────┐
│                    PRIVACY POOL (1,247 deposits)            │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐       │
│  │  │ │  │ │  │ │  │ │  │ │  │ │  │ │  │ │  │ │  │ ...   │
│  └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘       │
│   ... 1,247 existing deposits ...                           │
│                         ↓          ↑                         │
│                    (instant in/out)                          │
└─────────────────────────────────────────────────────────────┘
                            ↓          ↑
      Time 0.0s: Deposit ──┘          └── Time 2.0s: Withdraw
                            
┌──────────┐                                    ┌──────────┐
│  Client  │────── < 2 seconds total ─────────→│ Merchant │
└──────────┘                                    └──────────┘

Privacy Achieved:
  ✅ Anonymity Set: 1,247 (EXCELLENT)
  ✅ Identification Probability: 0.08%
  ✅ No waiting needed
  ✅ Perfect for real-time AI agent payments
  ✅ Time-sensitive transactions enabled

Timeline:
  10:00:00.000 - Client deposits to pool
  10:00:00.500 - Deposit confirmed on-chain
  10:00:01.000 - Generate zk-SNARK proof
  10:00:01.500 - Submit withdrawal via relayer
  10:00:02.000 - Payment received by merchant ✅
  
Total Time: 2 seconds
Privacy: Your deposit is 1 of 1,247 possibilities
```

**Key Insight:** You don't wait for OTHER people to deposit - you benefit from deposits that ALREADY HAPPENED. High-volume pools with 500+ existing deposits enable instant privacy!

### Privacy Timeline Comparison

| Pool Volume | Current Size | Your Wait Time | Use Case |
|-------------|--------------|----------------|----------|
| **🚀 Ultra-High** | **1,247** | **< 2 seconds** ⚡ | Instant AI payments |
| **⚡ Very High** | 500 | < 2 seconds | Real-time transactions |
| **✅ High** | 100-250 | Minutes | Popular pools |
| **⏰ Medium** | 50-100 | 1-2 hours | Standard usage |
| **😴 Low** | 10-50 | Days | Avoid if possible |

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        torx402 Ecosystem                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐         ┌──────────────┐         ┌──────────┐       │
│  │          │         │              │         │          │       │
│  │  Client  │◄───────►│ Privacy Pool │◄───────►│ Merchant │       │
│  │          │         │  (Contract)  │         │  Server  │       │
│  └────┬─────┘         └──────┬───────┘         └────┬─────┘       │
│       │                      │                      │              │
│       │                      │                      │              │
│       │                ┌─────┴──────┐              │              │
│       └───────────────►│            │◄─────────────┘              │
│                        │ Facilitator│                              │
│                        │   Server   │                              │
│                        └─────┬──────┘                              │
│                              │                                      │
│                        ┌─────┴──────┐                              │
│                        │            │                              │
│                        │ Relayer(s) │                              │
│                        │            │                              │
│                        └────────────┘                              │
│                                                                      │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │
                         ┌─────────┴──────────┐
                         │                    │
                         │   Blockchain       │
                         │  (EVM / Solana)    │
                         │                    │
                         └────────────────────┘
```

## Core Components

### 1. Client

**Role**: Initiates payment requests and consumes protected resources.

**Responsibilities**:
- Generate deposit commitments (secrets `k` and `r`)
- Deposit funds into privacy pools
- Request protected HTTP resources
- Generate zero-knowledge proofs for withdrawals
- Construct anonymous payment payloads
- Manage deposit notes and secrets securely

**Technology Stack**:
- TypeScript/JavaScript (browser or Node.js)
- Python (for AI agents and scripts)
- Cryptographic libraries (circomlib, snarkjs)
- HTTP client libraries (fetch, axios, httpx, requests)

**Key Operations**:
```
DEPOSIT:
  1. Generate random secrets (k, r)
  2. Compute commitment C = H1(k||r)
  3. Send transaction to Privacy Pool with N tokens + C
  4. Store deposit note securely

WITHDRAWAL/PAYMENT:
  1. Select deposit to spend
  2. Compute nullifier hash h = H1(k)
  3. Generate Merkle proof for commitment
  4. Create zk-SNARK proof
  5. Submit proof via relayer or directly
```

### 2. Privacy Pool (Smart Contract)

**Role**: Custodies deposits and enables anonymous withdrawals through zero-knowledge proofs.

**Responsibilities**:
- Accept fixed-denomination deposits with commitments
- Maintain Merkle tree of all deposit commitments
- Store historical Merkle roots (last 100)
- Verify zk-SNARK proofs for withdrawals
- Prevent double-spending via nullifier hash tracking
- Execute withdrawals to designated recipients
- Emit events for deposit and withdrawal tracking

**Data Structures**:
```solidity
// Merkle tree of commitments
mapping(uint256 => bytes32) public commitments;
uint256 public nextLeafIndex;

// Historical roots for proof flexibility
bytes32[100] public roots;
uint256 public currentRootIndex;

// Nullifier tracking (prevent double-spend)
mapping(bytes32 => bool) public nullifierHashes;

// Denomination
uint256 public immutable DENOMINATION;
```

**State Transitions**:
```
INITIAL STATE: Empty tree (all zeros)

AFTER DEPOSIT:
  - Add commitment C to tree at next leaf index
  - Recalculate Merkle root
  - Add new root to history array
  - Increment leaf index

AFTER WITHDRAWAL:
  - Verify zk-SNARK proof
  - Check nullifier hash is unused
  - Mark nullifier as spent
  - Transfer tokens to recipient
```

### 3. Merchant Server

**Role**: Provides protected HTTP resources in exchange for anonymous payment.

**Responsibilities**:
- Host protected API endpoints
- Return HTTP `402 Payment Required` with requirements
- Receive anonymous payment proofs via `X-PAYMENT` header
- Verify proofs through Facilitator
- Settle payments on-chain
- Deliver resources upon successful payment
- Log transactions for accounting (without revealing payer identity)

**Integration Points**:
- Express middleware (`torx402-express`)
- FastAPI middleware (`torx402-fastapi`)
- Flask middleware (`torx402-flask`)
- Hono middleware (`torx402-hono`)

**Payment Flow**:
```
1. Client → GET /protected-resource
2. Server → 402 Payment Required + privacy pool info
3. Client → GET /protected-resource + X-PAYMENT header
4. Server → Verify proof via Facilitator
5. Server → Settle via Privacy Pool
6. Server → 200 OK + Resource
```

### 4. Facilitator Server

**Role**: Provides infrastructure services for proof verification and blockchain interaction.

**Responsibilities**:
- Verify zk-SNARK proofs off-chain
- Interact with Privacy Pool contracts
- Provide `/verify` endpoint for payment validation
- Provide `/settle` endpoint for withdrawal execution
- Provide `/supported` endpoint for capability discovery
- Provide `/discovery/resources` for marketplace
- Manage RPC connections to multiple chains
- Handle gas fee management for settlements

**API Endpoints**:
```
POST /verify
  Input: { paymentPayload, paymentRequirements }
  Output: { isValid, payer, invalidReason? }

POST /settle
  Input: { paymentPayload, paymentRequirements }
  Output: { success, transaction, network, payer }

GET /supported
  Output: { kinds: [{ x402Version, scheme, network }] }

GET /discovery/resources
  Output: { items: [...], pagination: {...} }

GET /pools/:denomination/:network
  Output: { contractAddress, anonymitySetSize, ... }
```

### 5. Relayer Network

**Role**: Enables completely anonymous withdrawals by submitting transactions on behalf of clients.

**Responsibilities**:
- Accept withdrawal requests from clients
- Submit withdrawal transactions to Privacy Pool
- Pay blockchain gas fees (covered by withdrawal fee)
- Provide IP address anonymity
- Maintain uptime and reliability
- Earn fees for service provision

**Why Relayers?**:
Without relayers, clients would need to:
- Have ETH/SOL in their withdrawal address for gas
- Reveal their IP address when submitting transactions
- Potentially link deposits to withdrawals through timing analysis

**Relayer Economics**:
```
Fee Structure:
  - Client specifies fee in payment request
  - Typical fees: 0.5-2% of withdrawal amount
  - Fee covers gas costs + relayer profit
  
Revenue Model:
  - Fee per withdrawal transaction
  - Competitive market drives fee discovery
  - High-volume relayers more profitable
```

### 6. Blockchain Layer

**Role**: Provides immutable settlement and storage for the privacy protocol.

**Supported Chains**:
- **EVM Chains**: Base, Avalanche, Polygon, IoTeX, Peaq, Sei
- **Solana**: Mainnet and Devnet

**On-Chain Components**:
```
EVM:
  - Privacy Pool Contracts (per denomination)
  - EIP-3009 Token Contracts (USDC, etc.)
  - Verifier Contracts (Groth16 verification)

Solana:
  - Privacy Pool Programs
  - SPL Token Accounts
  - Program-derived Addresses (PDAs)
```

## Component Interaction Flow

### Deposit Flow

```
┌────────┐                ┌──────────────┐              ┌────────────┐
│ Client │                │ Privacy Pool │              │ Blockchain │
└───┬────┘                └──────┬───────┘              └─────┬──────┘
    │                            │                            │
    │ 1. Generate (k, r)         │                            │
    │    C = H1(k||r)            │                            │
    │                            │                            │
    │ 2. deposit(C, N tokens)    │                            │
    ├───────────────────────────>│                            │
    │                            │ 3. Add C to Merkle tree    │
    │                            │    Update root             │
    │                            │                            │
    │                            │ 4. Submit transaction      │
    │                            ├───────────────────────────>│
    │                            │                            │
    │                            │ 5. Transaction confirmed   │
    │                            │<───────────────────────────┤
    │                            │                            │
    │ 6. Emit Deposit event      │                            │
    │<───────────────────────────┤                            │
    │                            │                            │
    │ 7. Store note (k, r, l)    │                            │
    │    locally                 │                            │
    │                            │                            │
```

### Anonymous Payment Flow

```
┌────────┐  ┌────────┐  ┌────────────┐  ┌──────────┐  ┌──────────────┐
│ Client │  │Relayer │  │ Facilitator│  │ Merchant │  │ Privacy Pool │
└───┬────┘  └───┬────┘  └─────┬──────┘  └────┬─────┘  └──────┬───────┘
    │           │              │              │               │
    │ 1. GET /api              │              │               │
    ├──────────────────────────┼──────────────>               │
    │           │              │              │               │
    │           │              │  2. 402 Payment Required     │
    │<──────────────────────────┼──────────────┤               │
    │           │              │              │               │
    │ 3. Generate proof        │              │               │
    │    P = Prove(k,r,l,...)  │              │               │
    │           │              │              │               │
    │ 4. Send withdrawal req   │              │               │
    ├──────────>               │              │               │
    │           │              │              │               │
    │           │ 5. GET /api + X-PAYMENT: P  │               │
    │           ├──────────────┼──────────────>               │
    │           │              │              │               │
    │           │              │  6. POST /verify             │
    │           │              │<─────────────┤               │
    │           │              │              │               │
    │           │              │  7. { isValid: true }        │
    │           │              ├─────────────>│               │
    │           │              │              │               │
    │           │              │  8. POST /settle             │
    │           │              │<─────────────┤               │
    │           │              │              │               │
    │           │              │  9. withdraw(proof, ...)     │
    │           │              ├──────────────┼───────────────>
    │           │              │              │               │
    │           │              │              │  10. Verify proof
    │           │              │              │      Check nullifier
    │           │              │              │      Transfer tokens
    │           │              │              │               │
    │           │              │  11. { success, tx }         │
    │           │              │<─────────────┼───────────────┤
    │           │              │              │               │
    │           │              │  12. Settlement confirmed    │
    │           │              ├─────────────>│               │
    │           │              │              │               │
    │           │  13. 200 OK + Resource      │               │
    │           │<─────────────┼──────────────┤               │
    │           │              │              │               │
    │ 14. Resource delivered   │              │               │
    │<──────────┤              │              │               │
    │           │              │              │               │
```

## Privacy Pool Architecture

### Denomination-Based Pools

torx402 uses **fixed-denomination pools** to maximize anonymity sets:

```
┌─────────────────────────────────────────────────────────────┐
│                    DENOMINATION POOLS                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  0.001 ETH Pool      0.01 ETH Pool       0.1 ETH Pool      │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │ 150 deposits │   │ 450 deposits │   │ 89 deposits  │   │
│  │              │   │              │   │              │   │
│  │ Anonymity    │   │ Anonymity    │   │ Anonymity    │   │
│  │ Set: 150     │   │ Set: 450     │   │ Set: 89      │   │
│  └──────────────┘   └──────────────┘   └──────────────┘   │
│                                                              │
│  1 ETH Pool          10 ETH Pool         100 ETH Pool      │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │ 34 deposits  │   │ 12 deposits  │   │ 3 deposits   │   │
│  │              │   │              │   │              │   │
│  │ Anonymity    │   │ Anonymity    │   │ Anonymity    │   │
│  │ Set: 34      │   │ Set: 12      │   │ Set: 3       │   │
│  └──────────────┘   └──────────────┘   └──────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Rule: Larger anonymity sets = Better privacy
Recommendation: Use pools with >100 deposits for strong privacy
```

### Merkle Tree Structure

Each pool maintains a Merkle tree of commitments:

```
                        Root (R)
                       /        \
                      /          \
                   h23            h67
                  /    \         /    \
                 /      \       /      \
               h01      h45   h23     h89
              /  \     /  \   /  \    /  \
             /    \   /    \  \   \  /    \
           C0    C1 C2    C3 C4  C5 C6   C7 ...
           
Height: 20 (supports 2^20 = ~1M deposits per pool)
Hash Function: MiMC (zk-SNARK friendly)
Leaf Values: Commitments C = H1(k||r)
```

**Why Merkle Trees?**:
- Efficient proof of membership (log N proof size)
- zk-SNARK friendly (MiMC hash)
- Supports incremental updates
- Historical roots enable flexible proof timing

## Security Architecture

### Cryptographic Security Layers

```
Layer 1: Zero-Knowledge Proofs (zk-SNARKs)
  ↓
  - Proves knowledge of (k, r) without revealing them
  - Proves commitment is in Merkle tree
  - Binding to recipient address and fee
  
Layer 2: Nullifier System
  ↓
  - Prevents double-spending
  - Unlinkable to commitment
  - Deterministic from secret k
  
Layer 3: Merkle Tree
  ↓
  - Accumulates all deposits
  - Proves membership efficiently
  - Historical roots for timing flexibility
  
Layer 4: Smart Contract
  ↓
  - Enforces protocol rules
  - Verifies proofs on-chain
  - Manages state transitions
  
Layer 5: Blockchain
  ↓
  - Immutable settlement
  - Consensus security
  - Censorship resistance
```

### Attack Prevention

| Attack Type | Prevention Mechanism |
|-------------|---------------------|
| **Double-Spend** | Nullifier hash tracking in contract |
| **Replay Attack** | Nullifier uniqueness, time bounds |
| **Front-Running** | Proof bound to specific recipient |
| **Privacy Leak** | Relayer network, fixed denominations |
| **Sybil Attack** | Economic cost of deposits |
| **Pool Draining** | Proof verification, nullifier checks |

## Scalability Considerations

### Horizontal Scaling

```
Multiple Pools per Chain:
  - Different denominations
  - Independent anonymity sets
  - Parallel processing

Multiple Chains:
  - EVM: Base, Avalanche, Polygon, etc.
  - Solana: Different economics
  - Cross-chain future expansion

Multiple Facilitators:
  - Decentralized verification
  - Load balancing
  - Redundancy

Relayer Network:
  - Geographic distribution
  - Competitive fee market
  - No single point of failure
```

### Performance Optimization

```
Client-Side:
  - Proof generation: ~5-10 seconds
  - Local caching of Merkle trees
  - Parallel proof computation

Server-Side:
  - Proof verification: <100ms
  - Cached RPC responses
  - Batch processing support

On-Chain:
  - Optimized verifier contracts
  - Gas-efficient Merkle updates
  - Minimal storage footprint
```

## Future Architecture Evolution

### Planned Enhancements

1. **Multi-Asset Support**: Privacy pools for different tokens (USDC, DAI, USDT)
2. **Cross-Chain Bridges**: Deposit on Chain A, withdraw on Chain B
3. **Trusted Setup Ceremony**: Community-driven parameter generation
4. **Recursive Proofs**: Smaller proof sizes, faster verification
5. **Mobile SDKs**: Native iOS/Android support
6. **Browser Extensions**: One-click anonymous payments
7. **Compliance Layer**: Optional identity disclosure for regulated entities

## Next Steps

- [Payment Flow Details](payment-flow.md)
- [Deposit Phase](deposit-phase.md)
- [Withdrawal Phase](withdrawal-phase.md)
- [Privacy Guarantees](privacy-guarantees.md)
- [Protocol Specification](../protocol/overview.md)