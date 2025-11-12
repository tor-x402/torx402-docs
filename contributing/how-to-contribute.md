---
title: Contributing to torx402
description: How to contribute code, docs, and ideas to torx402 responsibly and effectively.
---

# Contributing to torx402

Thanks for your interest in contributing! torx402 is an open-source, privacy-preserving HTTP micropayment protocol built on top of the battle‑tested Tornado.cash design and the x402 ecosystem. We welcome contributions of all kinds: code, documentation, examples, bug reports, feature proposals, and security reviews.

If you’re new to the project, this guide will help you get productive quickly while protecting user privacy and keeping changes easy to review and ship.

---

## Ways you can help

- Documentation
  - Improve or clarify existing docs
  - Add examples (TypeScript/Python)
  - Fix typos, broken links, or outdated content
- SDKs & Middlewares
  - TypeScript/Node client library
  - Python client for AI agents
  - Express/Hono/FastAPI/Flask middlewares
- Protocol & Infrastructure
  - Facilitator (verification/settlement service)
  - Relayer tooling (privacy-first HTTP forwarder + settlement)
  - Smart contracts/programs (EVM/Solana)
- Frontend & DevEx
  - Landing page copy, diagrams, dev tooling, CI improvements
- Issue triage & testing
  - Reproduce bugs, write failing tests, verify fixes, improve coverage

---

## Principles

- Privacy by default: do not log PII (e.g., wallet addresses, IPs) in production paths.
- Safety first: never include private keys, secrets, or real funds in examples/tests.
- Minimal surface: prefer small, self-contained changes; avoid large refactors unless justified.
- Progressive disclosure: start with simple interfaces and deepen where needed (docs and APIs).
- Interoperability: preserve compatibility across L2s, BSC, and Solana where feasible.

---

## Getting started

### 1) Discuss before you build
- Open a GitHub issue to propose a change (bug fix, feature, docs improvement).
- For larger features, start a short “RFC” issue with:
  - Problem statement and motivation
  - Proposed design/API
  - Alternatives considered
  - Expected impact and risks

### 2) Pick up an issue
- Look for issues labeled `good first issue`, `help wanted`, or `docs`.
- Comment to claim an issue so others know you’re working on it.

### 3) Local setup (typical)
Depending on the package you work on, you’ll likely need:

- Node.js ≥ 18 and npm/pnpm/yarn
- TypeScript ≥ 5, ESLint, Prettier
- For EVM contracts: Foundry or Hardhat
- For Solana: Rust/Cargo + Anchor (if applicable)
- Circom/snark tooling for circuits (if applicable)

Example (TypeScript SDK):
```bash
# Install dependencies
npm install

# Build
npm run build

# Test
npm test
```

---

## Code guidelines

### Commit messages
We prefer [Conventional Commits](https://www.conventionalcommits.org/):
- `feat(sdk): add torx402 payAnonymously helper`
- `fix(relayer): handle expired root with auto-regeneration`
- `docs(readme): update architecture diagram to route via relayer`
- `chore(ci): speed up linting`

### Branching
- Create feature branches from `main`
- Keep PRs small and focused (ideal size: < 300 lines)

### Code style
- TypeScript: ESLint + Prettier defaults
- Solidity: NatSpec for public interfaces, explicit visibility, clear storage layout notes
- Rust (Solana): idiomatic Rust, clippy clean
- Test-first: include unit/integration tests for new functionality

### Security & privacy
- Do not log secrets, wallet addresses, or IPs in non-debug paths
- Avoid adding analytics that can tie user actions to identities
- Prefer E2EE for HTTP relaying patterns in examples where feasible
- Follow least-privilege and keep dangerous helpers behind explicit flags

---

## Documentation guidelines

- Tone: clear, technical, and honest (no hype)
- Structure: progressive disclosure (overview → details → references)
- Diagrams: prefer Mermaid for sequence/architecture diagrams
- Examples: runnable, minimal, and realistic
- Privacy: remind readers to avoid using real funds/keys; prefer testnets

When adding docs:
- Update `SUMMARY.md` to include your page
- Use consistent headings and terminology:
  - “Privacy Pool” (smart contract/program)
  - “Facilitator” (verify/settle service)
  - “Relayer” (HTTP forwarder + blockchain submitter)
- Prefer `bash`, `typescript`, `solidity`, `rust` fenced code blocks for syntax highlighting

---

## Submitting an issue

- Use templates
- For bugs:
  - Version info and environment (OS, Node/Rust, chain)
  - Steps to reproduce
  - Expected vs actual behavior
  - Minimal logs or stack traces (strip PII)
- For features:
  - Motivation and use case
  - Proposed interface
  - Alternatives considered

For security issues, see “Responsible Disclosure” below.

---

## Submitting a pull request

### PR checklist
- [ ] Small, focused, and rebased on `main`
- [ ] Tests added/updated and passing
- [ ] Lint/format runs clean
- [ ] Docs updated (if user-facing change)
- [ ] No secrets or real funds in examples/tests
- [ ] Privacy and security implications considered

### Review expectations
- Be responsive and collaborative
- Address review comments or explain trade-offs
- Keep discussions technical and respectful

By submitting a PR, you agree that your contribution is licensed under the project’s MIT license.

---

## Project areas & expectations

### Facilitator
- Verify zk-SNARK proofs off-chain
- Settle on-chain with idempotency and retries
- No PII logs; structured logs for ops
- Multi-chain support (L2s, BSC, Solana)

### Relayer
- HTTP forwarder + settlement submitter
- Optional E2EE pass-through (opaque bodies)
- Rate limiting, backoff, and privacy-first logs
- Split-role relayers (HTTP vs blockchain) encouraged for stronger privacy

### Contracts/Programs
- Height 32 Merkle tree (capacity: 4.3B)
- 10,000 root circular buffer (mapping-based O(1) lookup)
- Fixed denominations per pool; nullifier-based double-spend prevention
- Thorough tests: membership, nullifier reuse, binding, root expiry regeneration

### SDKs & Middlewares
- Auto-regenerate proofs on root expiry (5–10s)
- Minimal, predictable APIs
- Clear errors and actionable messages
- Examples for Node/TS + Python (agents)

---

## Responsible disclosure

If you believe you’ve found a security vulnerability, **do not open a public issue**.

- Email: security@torx402.network
- Provide:
  - Description and potential impact
  - Steps to reproduce (clear, minimal)
  - Affected versions/targets
  - Suggested remediation (if any)
- We will respond promptly and coordinate a fix & disclosure timeline.

---

## Community & communication

- X/Twitter: https://twitter.com/torx402
- GitHub Issues: technical bugs, feature requests
- Discussions/RFC issues: design proposals and bigger changes

---

## FAQ

**Q: Can I contribute externally hosted relayers/facilitators?**
Yes. Please clearly document your privacy guarantees (no IP logs, retention policy, E2EE support).

**Q: Do I need to run mainnet to test?**
No. Please use testnets (e.g., Base Sepolia, BSC Testnet, Solana Devnet) and fake keys/funds.

**Q: Are large refactors welcome?**
Prefer small, incremental changes. For large refactors, open an RFC issue first.

---

## Thank you

Whether you fix a typo, improve the docs, or add a feature—your contribution makes torx402 better for everyone. We appreciate your time, care, and attention to privacy.

Welcome aboard! 🙌
