---
description: Glossary of terms used in torx402 documentation
---

# Glossary

## A

### Anonymity Set
The group of all deposits in a privacy pool that could potentially correspond to a given withdrawal. Larger anonymity sets provide better privacy because they make it harder to determine which deposit funded a specific withdrawal.

**Example**: If a pool has 100 deposits, each withdrawal has an anonymity set of 100 (any of the 100 deposits could be the source).

### Asset
A digital token or cryptocurrency used for payments. In torx402, this typically refers to ETH, USDC, USDT, or other ERC-20/SPL tokens.

### Authorization
In the context of EIP-3009, a signed message that authorizes a token transfer without requiring the sender to pay gas fees. Used in standard x402 for gasless payments.

## B

### BN128 (alt_bn128)
An elliptic curve used in Ethereum's precompiled contracts for pairing operations. Essential for zk-SNARK verification on EVM chains. Also known as the BN254 curve.

### Blockchain
A distributed ledger technology that records transactions in blocks linked together cryptographically. torx402 operates on multiple blockchains including Ethereum-compatible chains and Solana.

## C

### Commitment
A cryptographic value that binds a user to their secret data without revealing it. In torx402, the commitment is `C = H1(k||r)` where `k` is the nullifier and `r` is randomness.

### Circom
A circuit compiler for writing arithmetic circuits used in zero-knowledge proofs. torx402's zk-SNARK circuits are written in Circom.

### Circuit
In zero-knowledge proofs, a mathematical representation of the computation being proven. torx402's circuit proves knowledge of deposit secrets.

## D

### Denomination
A fixed amount for deposits in a privacy pool. Using fixed denominations (e.g., 0.1 ETH, 1 ETH) prevents amount-based deanonymization and improves the anonymity set.

### Deposit
The act of placing funds into a privacy pool with a secret commitment. The first phase of using torx402.

### Deposit Note
A secret string containing the information needed to withdraw a deposit: the nullifier (k), randomness (r), and leaf index. Must be kept secure.

**Format**: `tornado-eth-0.1-base-sepolia-0x1234...abcd`

### Double-Spending
Attempting to spend the same deposit twice. torx402 prevents this using nullifier hashes.

## E

### EIP-3009
Ethereum Improvement Proposal 3009, which defines a standard for gasless token transfers using signed authorizations. Used in standard x402 but not in torx402 (which uses zk-SNARKs instead).

### EIP-712
Ethereum Improvement Proposal 712, which defines a standard for typed structured data signing. Used for creating human-readable signed messages.

### EVM (Ethereum Virtual Machine)
The runtime environment for smart contracts on Ethereum and compatible blockchains. torx402 supports multiple EVM chains.

## F

### Facilitator
A server that provides infrastructure services for x402/torx402, including proof verification, blockchain interaction, and pool discovery. Merchants use facilitators to verify and settle payments.

### Fee
The amount paid to a relayer for submitting a withdrawal transaction on behalf of a user. Enables anonymous withdrawals without requiring gas in the withdrawal address.

### Front-Running
An attack where someone observes a pending transaction and submits their own transaction with a higher gas price to execute first. torx402 prevents this by binding proofs to specific recipients.

## G

### Groth16
A zero-knowledge SNARK construction that produces small, constant-size proofs with fast verification. Used by torx402 for privacy proofs.

**Properties**: 128 bytes proof size, ~1-2ms verification time, ~5-10 seconds proving time.

## H

### Hash Function
A cryptographic function that maps arbitrary data to a fixed-size output. torx402 uses Pedersen Hash (H1) and MiMC (H2).

### HTTP 402 Payment Required
An HTTP status code indicating that payment is required to access a resource. Central to both x402 and torx402 protocols.

## I

### iden3
An open-source identity and cryptography project that developed circomlib, the library of circuits used by torx402.

## K

### Key Pair (Proving/Verifying)
In zk-SNARKs, a pair of cryptographic keys generated during setup. The proving key creates proofs, the verifying key checks them. Generated through a trusted setup ceremony.

## L

### Leaf Index
The position of a commitment in the Merkle tree. Used to construct Merkle proofs for withdrawals.

## M

### Merkle Tree
A tree data structure where each non-leaf node is a hash of its children. torx402 uses Merkle trees to efficiently prove that a commitment exists in the deposit set.

**Properties**: Height 32, supports up to 2^32 (~4.3B) deposits per pool.

### Merkle Root
The top hash of a Merkle tree, representing the entire tree state. torx402 stores the last 100 roots to allow flexible proof timing.

### Merkle Proof (Opening)
A set of sibling hashes proving that a specific leaf is part of a Merkle tree with a given root. Required for withdrawal proofs.

### Merchant
A server that provides protected HTTP resources and accepts torx402 payments in exchange for access.

### MiMC Hash
A cryptographic hash function designed for efficient use in zero-knowledge circuits. Used in torx402's Merkle tree.

**Advantages**: Fewer constraints in zk-SNARKs compared to SHA-256 or Keccak.

### Micropayment
A very small payment, often fractions of a dollar. Traditional payment systems are too expensive for micropayments due to fixed fees.

## N

### Nullifier
A secret random value (k) generated during deposit. Used to compute the nullifier hash during withdrawal. Never revealed directly.

### Nullifier Hash
The hash of the nullifier: `h = H1(k)`. Revealed during withdrawal to prevent double-spending. Unlinkable to the original commitment.

## O

### Opening
See Merkle Proof.

## P

### Pairing
A bilinear map used in elliptic curve cryptography, essential for zk-SNARK verification. EVM chains provide precompiled contracts for BN128 pairings.

### Pedersen Hash
A cryptographically secure hash function based on elliptic curve operations. Used in torx402 for commitments and nullifier hashing (H1).

### Privacy Pool
A smart contract that holds deposits and enables anonymous withdrawals through zero-knowledge proofs. Organized by denomination and network.

### Proof (zk-SNARK)
A cryptographic proof that a statement is true without revealing why it's true. In torx402, proves deposit ownership without revealing which deposit.

**Structure**: Three elements (pi_a, pi_b, pi_c) in Groth16 format.

### Public Signals
The public inputs to a zero-knowledge proof that are revealed to the verifier. In torx402: root, nullifier hash, recipient, relayer, fee.

## R

### Randomness
A secret random value (r) generated during deposit, combined with the nullifier to create the commitment. Never revealed.

### Relayer
A third-party service that submits withdrawal transactions on behalf of users. Enables complete anonymity by hiding the user's IP address and preventing the need for gas in the withdrawal address.

### Replay Attack
An attack where a valid proof is reused. Prevented by nullifier hash uniqueness and time-bound authorizations.

### Root History
An array of the last Y Merkle roots stored in the privacy pool contract. Allows users to generate proofs against recent roots even as new deposits are added.

## S

### Scheme
A payment method in x402/torx402. Standard x402 supports "exact" (EIP-3009), while torx402 adds "tornado" (zk-SNARK based).

### Settlement
The final execution of a payment on the blockchain, transferring funds from the privacy pool to the recipient.

### Smart Contract
Self-executing code deployed on a blockchain. torx402's privacy pools are implemented as smart contracts.

### Solidity
The programming language for writing smart contracts on EVM chains.

### SNARK
Succinct Non-interactive ARgument of Knowledge. A type of zero-knowledge proof that is small and fast to verify.

### Sponge Construction
A cryptographic construction for hash functions that processes input data in blocks. MiMC uses sponge construction in torx402.

## T

### Tornado Cash
A privacy protocol for Ethereum that pioneered the use of zk-SNARKs for transaction privacy. torx402 adapts this technology for HTTP micropayments.

### Trusted Setup
A one-time ceremony to generate zk-SNARK proving and verifying keys. Requires trust that participants destroy their "toxic waste" secret values.

**Note**: torx402 plans a multi-party trusted setup ceremony for decentralized trust.

## U

### Unlinkability
The property that two events (e.g., deposit and withdrawal) cannot be connected. Core privacy guarantee of torx402.

### USDC
USD Coin, a stablecoin pegged to the US dollar. Commonly used in torx402 for predictable payment amounts.

## V

### Verification
The process of checking a zk-SNARK proof's validity. Done by the smart contract on-chain and/or by the facilitator off-chain.

### Verifier Contract
A smart contract that verifies zk-SNARK proofs. Automatically generated from the circuit and verifying key.

## W

### Withdrawal
The act of removing funds from a privacy pool by generating a zero-knowledge proof of deposit ownership. The second phase of using torx402.

### Witness
In zero-knowledge proofs, the private inputs known only to the prover. In torx402: nullifier (k), randomness (r), leaf index (l), Merkle proof.

## X

### x402
An HTTP-native micropayment protocol that uses the 402 status code for payment-required responses. The foundation for torx402.

### torx402
A privacy-preserving extension of x402 that uses zero-knowledge proofs to enable anonymous payments while maintaining HTTP compatibility.

### X-PAYMENT Header
An HTTP header containing the payment proof in x402/torx402. Base64-encoded JSON with proof data.

### X-PAYMENT-RESPONSE Header
An HTTP header returned by the merchant containing settlement information. Base64-encoded JSON with transaction details.

## Z

### Zero-Knowledge Proof
A cryptographic method to prove knowledge of information without revealing the information itself.

**Example**: Proving you know a password without revealing the password.

### zk-SNARK
Zero-Knowledge Succinct Non-interactive ARgument of Knowledge. The specific type of zero-knowledge proof used by torx402.

**Properties**:
- **Zero-Knowledge**: Reveals nothing beyond the statement's truth
- **Succinct**: Proofs are small (constant size)
- **Non-interactive**: No back-and-forth between prover and verifier
- **Argument**: Computationally sound (secure against polynomial-time adversaries)
- **of Knowledge**: Prover must actually know the witness

---

## Additional Resources

- [x402 Protocol Specification](https://x402.org)
- [Tornado Cash Whitepaper](https://tornado.cash/audits/TornadoCash_whitepaper_v1.4.pdf)
- [Groth16 Paper](https://eprint.iacr.org/2016/260.pdf)
- [ZK-SNARK Explainer](https://z.cash/technology/zksnarks/)
- [Circom Documentation](https://docs.circom.io/)
- [EIP-3009 Specification](https://eips.ethereum.org/EIPS/eip-3009)

## Notation Reference

For mathematical notation used in the protocol specification, see [Mathematical Notation](notation.md).

For circuit diagrams and visualizations, see [Circuit Diagrams](circuits.md).
