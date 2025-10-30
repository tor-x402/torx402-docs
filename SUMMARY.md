# Summary

## Introduction

* [Welcome to x402tornado](README.md)
* [What is x402tornado?](introduction/what-is-x402tornado.md)
* [Why Privacy-Preserving Micropayments?](introduction/why-privacy.md)
* [Key Features](introduction/key-features.md)
* [Quick Start](introduction/quick-start.md)

## Core Concepts

* [Understanding x402 Protocol](core-concepts/x402-protocol.md)
* [Understanding Tornado Cash](core-concepts/tornado-cash.md)
* [Privacy in Micropayments](core-concepts/privacy-in-micropayments.md)
* [Zero-Knowledge Proofs Basics](core-concepts/zero-knowledge-proofs.md)
* [Anonymity Sets](core-concepts/anonymity-sets.md)

## How x402tornado Works

* [Architecture Overview](how-it-works/architecture.md)
* [Payment Flow](how-it-works/payment-flow.md)
* [Deposit Phase](how-it-works/deposit-phase.md)
* [Withdrawal Phase](how-it-works/withdrawal-phase.md)
* [Privacy Guarantees](how-it-works/privacy-guarantees.md)
* [Relayer System](how-it-works/relayer-system.md)

## Protocol Specification

* [Protocol Overview](protocol/overview.md)
* [HTTP Status Codes](protocol/http-status-codes.md)
* [Data Structures](protocol/data-structures.md)
  * [Privacy-Enhanced Payment Requirements](protocol/data-structures.md#payment-requirements)
  * [Anonymous Payment Proof](protocol/data-structures.md#payment-proof)
  * [Settlement Response](protocol/data-structures.md#settlement-response)
* [Cryptographic Primitives](protocol/cryptographic-primitives.md)
  * [Hash Functions](protocol/cryptographic-primitives.md#hash-functions)
  * [Merkle Trees](protocol/cryptographic-primitives.md#merkle-trees)
  * [zk-SNARKs](protocol/cryptographic-primitives.md#zk-snarks)
* [Smart Contract Interface](protocol/smart-contract.md)
* [Facilitator API](protocol/facilitator-api.md)

## Implementation Guide

### For Merchants (Servers)

* [Server Setup](implementation/servers/setup.md)
* [Accepting Anonymous Payments](implementation/servers/accepting-payments.md)
* [TypeScript/Express](implementation/servers/typescript-express.md)
* [TypeScript/Hono](implementation/servers/typescript-hono.md)
* [Python/FastAPI](implementation/servers/python-fastapi.md)
* [Python/Flask](implementation/servers/python-flask.md)

### For Clients (Buyers)

* [Client Setup](implementation/clients/setup.md)
* [Making Anonymous Payments](implementation/clients/making-payments.md)
* [Deposit Process](implementation/clients/deposit.md)
* [Withdrawal Process](implementation/clients/withdrawal.md)
* [TypeScript Client](implementation/clients/typescript.md)
* [Python Client](implementation/clients/python.md)

### For Relayers

* [Relayer Overview](implementation/relayers/overview.md)
* [Running a Relayer](implementation/relayers/running-relayer.md)
* [Relayer Economics](implementation/relayers/economics.md)

## Advanced Topics

* [Denomination Pools](advanced/denomination-pools.md)
* [Multi-Chain Support](advanced/multi-chain.md)
* [Optimizing Anonymity Sets](advanced/optimizing-anonymity.md)
* [Mixer Strategies](advanced/mixer-strategies.md)
* [Batch Withdrawals](advanced/batch-withdrawals.md)
* [Compliance Considerations](advanced/compliance.md)

## Security

* [Security Model](security/security-model.md)
* [Threat Analysis](security/threat-analysis.md)
* [Replay Attack Prevention](security/replay-prevention.md)
* [Front-Running Protection](security/front-running.md)
* [Double-Spending Prevention](security/double-spending.md)
* [Privacy Leakage Vectors](security/privacy-leakage.md)
* [Best Practices](security/best-practices.md)
* [Audits](security/audits.md)

## Use Cases

* [Anonymous AI Agent Commerce](use-cases/ai-agents.md)
* [Privacy-Preserving API Monetization](use-cases/api-monetization.md)
* [Anonymous Content Access](use-cases/content-access.md)
* [Confidential B2B Payments](use-cases/b2b-payments.md)
* [DePIN & IoT Micropayments](use-cases/depin-iot.md)

## Network Support

* [Supported Blockchains](networks/supported-chains.md)
* [Network-Specific Details](networks/network-details.md)
* [Solana Implementation](networks/solana.md)
* [EVM Chains](networks/evm-chains.md)

## Developer Resources

* [SDK Reference](resources/sdk-reference.md)
* [API Reference](resources/api-reference.md)
* [Code Examples](resources/code-examples.md)
* [Testing Guide](resources/testing.md)
* [Troubleshooting](resources/troubleshooting.md)
* [FAQ](resources/faq.md)

## Ecosystem

* [Official Links](ecosystem/official-links.md)
* [Facilitators](ecosystem/facilitators.md)
* [Community Relayers](ecosystem/community-relayers.md)
* [Integration Partners](ecosystem/partners.md)

## Contributing

* [How to Contribute](contributing/how-to-contribute.md)
* [Development Setup](contributing/development-setup.md)
* [Code Style Guide](contributing/code-style.md)
* [Submitting Issues](contributing/issues.md)
* [Pull Request Process](contributing/pull-requests.md)

## Legal & Compliance

* [Terms of Use](legal/terms.md)
* [Privacy Policy](legal/privacy-policy.md)
* [Regulatory Considerations](legal/regulatory.md)
* [Disclaimer](legal/disclaimer.md)

## Appendix

* [Glossary](appendix/glossary.md)
* [Mathematical Notation](appendix/notation.md)
* [Circuit Diagrams](appendix/circuits.md)
* [Version History](appendix/version-history.md)
* [References](appendix/references.md)