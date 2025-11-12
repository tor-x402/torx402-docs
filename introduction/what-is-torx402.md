# What is torx402?

## Overview

**torx402** is a privacy-preserving payment protocol that combines the HTTP-native micropayment capabilities of [x402](https://x402.org) with the zero-knowledge privacy technology from [Tornado Cash](https://tornado.ws/). It enables **anonymous, instant, and verifiable payments** for web APIs, digital content, and autonomous agent commerce.

In simple terms: torx402 lets you pay for things on the internet without revealing who you are, while cryptographically proving that payment was made.

## The Genesis: Two Powerful Protocols

### x402: HTTP-Native Micropayments

The x402 protocol revived the HTTP `402 Payment Required` status code to enable seamless pay-per-use payments for web resources. It solved the problem of micropayment integration by embedding payment directly into the HTTP request-response cycle.

**What x402 provides:**
- Native HTTP integration (no custom protocols)
- Instant settlement (<1 second)
- No merchant fees
- Pay-per-request pricing
- Agent-friendly design

**What x402 lacks:**
- Privacy protection
- Transaction anonymity
- Protection against usage pattern analysis

### Tornado Cash: Privacy Through Zero-Knowledge

Tornado Cash pioneered privacy-preserving transactions on Ethereum using zero-knowledge proofs (zk-SNARKs). It broke the on-chain link between deposit and withdrawal addresses by creating anonymity sets where any deposit could correspond to any withdrawal.

**What Tornado provides:**
- Complete transaction privacy
- Cryptographic anonymity guarantees
- Deposit-withdrawal unlinkability
- No trusted third parties

**What Tornado lacks:**
- HTTP integration
- Micropayment optimization
- API monetization features

### torx402: The Best of Both Worlds

torx402 merges these protocols to create a **privacy-preserving HTTP micropayment system** that provides:

✅ HTTP-native integration from x402
✅ Zero-knowledge privacy from Tornado
✅ Instant settlement with anonymity
✅ Verifiable payments without identity exposure
✅ Agent-ready commerce infrastructure

## How torx402 Works (very very High Level)

```
┌─────────────────────────────────────────────────────────────┐
│                    PRIVACY POOL                             │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐           │
│  │ 0.1Ξ │  │ 0.1Ξ │  │ 0.1Ξ │  │ 0.1Ξ │  │ 0.1Ξ │  ...      │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘           │
│     ↑                              ↓                        │
│     │                              │                        │
│  DEPOSIT                      WITHDRAW (anonymous)          │
│     │                              │                        │
└─────┼──────────────────────────────┼────────────────────────┘
      │                              │
      │                              │ zk-SNARK proof
   ┌──┴───┐            HTTP           ┌─────────┐         ┌───┴────┐
   │Client│ ─────────────────────────>│ Relayer │ ──────> │Merchant│
   └──────┘   402 / X-PAYMENT         └─────────┘         └────────┘
```

### The Process

1. **Deposit Phase**: Client deposits a fixed amount (e.g., 0.1 ETH) into a privacy pool with a secret commitment.
2. **Mixing Period**: Deposit joins an anonymity set with other deposits.
3. **Request (via Relayer)**: Client → Relayer: GET `/api`; Relayer → Merchant: forward the request.
4. **Payment Required (via Relayer)**: Merchant → Relayer → Client: HTTP `402 Payment Required` with torx402 payment requirements.
5. **Proof Generation**: Client generates a zk‑SNARK proof (bound to recipient and relayer/fee). If the cached root expired, the SDK regenerates the proof against a current root.
6. **Anonymous Payment (via Relayer)**: Client → Relayer: send proof; Relayer → Merchant: GET `/api` with `X-PAYMENT` (base64 JSON payload).
7. **Verify & Settle**: Merchant → Facilitator: `/verify` → valid; then `/settle`. Facilitator settles on‑chain via the Privacy Pool (transfer to Merchant, relayer fee).
8. **Resource Delivery (via Relayer)**: Merchant → Relayer → Client: `200 OK` + resource (optionally with `X-PAYMENT-RESPONSE`).

**The Privacy Guarantee**: The merchant receives payment and can verify it's legitimate, but cannot determine which deposit in the pool it came from.

## Key Differentiators

### vs. Traditional Payments (Credit Cards, PayPal)

| Feature | Traditional | torx402 |
|---------|------------|-------------|
| Transaction Speed | Minutes-Days | <1 second |
| Fees | 2-3% + fixed | ~0.1% network |
| Privacy | None | Complete |
| Accounts Required | Yes | No |
| Micropayment Viable | No | Yes |
| Chargebacks | Yes (risk) | No (cryptographic) |

### vs. Standard Cryptocurrency Payments

| Feature | Standard Crypto | torx402 |
|---------|----------------|-------------|
| Privacy | Pseudonymous | Anonymous |
| HTTP Integration | Complex | Native |
| Sender Visible | Yes | No |
| Usage Patterns | Trackable | Hidden |
| Anonymity Set | 1 | Entire pool |

### vs. Standard x402 Protocol

| Feature | x402 | torx402 |
|---------|------|-------------|
| Payment Privacy | None | Complete |
| Sender Anonymity | Exposed | Protected |
| Usage Tracking | Possible | Prevented |
| Zero-Knowledge Proofs | No | Yes |
| Anonymity Sets | N/A | Multi-user pools |

## Architecture Principles

torx402 is built on these core principles:

### 🔐 Privacy by Design
Privacy is not an add-on—it's fundamental to the protocol architecture. Zero-knowledge proofs ensure privacy at the cryptographic level.

### 🌐 HTTP-Native
Works with standard HTTP infrastructure. No custom protocols, no special requirements beyond torx402 support.

### 🚀 Performance-First
Sub-second settlement times make micropayments practical. Optimized circuits keep proof generation fast.

### 🔓 Open & Decentralized
Fully open-source protocol with no central authority. Anyone can verify the code, run a relayer, or build integrations.

### 🛡️ Secure by Default
126-bit cryptographic security. Multiple layers of protection against attacks (replay, front-running, double-spend).

### 🤝 Agent-Compatible
Designed for both human and AI agent use. Simple enough for autonomous decision-making.

## Real-World Applications

### Anonymous AI Trading Bots
A trading bot purchases real-time market data from multiple providers without revealing its data sources or trading strategy to competitors.

### Privacy-Preserving News Access
Readers pay for individual articles without creating accounts or being tracked across publications. Publishers get paid, readers stay anonymous.

### Confidential B2B API Integration
A startup integrates with enterprise APIs for market research without alerting competitors to their product development direction.

### DePIN Bandwidth Marketplace
IoT devices purchase bandwidth from peer devices using anonymous micropayments, preventing network topology analysis.

### Private AI Model Inference
AI applications query expensive LLM APIs anonymously, preventing model providers from training on query patterns or building user profiles.

## Technical Foundations

torx402 builds on proven cryptographic primitives:

- **zk-SNARKs (Groth16)**: Zero-knowledge proof system for privacy
- **Pedersen Hash**: Commitment scheme for nullifiers
- **MiMC Hash**: zk-SNARK-friendly hash for Merkle trees
- **Merkle Trees**: Efficient accumulator for deposit commitments
- **EIP-3009**: Gasless token transfers for EVM chains
- **Solana Programs**: Native support for SVM chains

## Getting Started

Ready to integrate privacy-preserving payments?

- 📚 [Learn Core Concepts](../core-concepts/privacy-in-micropayments.md)

- 🏗️ [Architecture Overview](../how-it-works/architecture.md)
- 💻 [Implementation Guide](../implementation/servers/setup.md)

## Next Steps

<table data-view="cards">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th data-hidden data-card-target data-type="content-ref"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Why Privacy?</strong></td>
      <td>Understand the importance of payment privacy</td>
      <td><a href="why-privacy.md">why-privacy.md</a></td>
    </tr>
    <tr>
      <td><strong>Key Features</strong></td>
      <td>Explore torx402's capabilities</td>
      <td><a href="key-features.md">key-features.md</a></td>
    </tr>

  </tbody>
</table>
