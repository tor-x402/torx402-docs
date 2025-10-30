---
description: Get started with x402tornado in 5 minutes
---

# Quick Start

This guide will help you make your first anonymous payment using x402tornado in just a few minutes.

## What You'll Learn

- How to deposit funds into a privacy pool
- How to make an anonymous payment for an API resource
- How to verify your transaction was successful

## Prerequisites

Before you begin, make sure you have:

- **Node.js 18+** or **Python 3.9+** installed
- A **wallet** with some ETH/USDC on a supported network (Base Sepolia recommended for testing)
- **Basic understanding** of blockchain transactions

## Choose Your Path

<tabs>
<tab title="TypeScript">

### Installation

```bash
npm install x402tornado-client
# or
pnpm add x402tornado-client
# or
yarn add x402tornado-client
```

### Step 1: Initialize Client

```typescript
import { TornadoClient } from 'x402tornado-client';

const client = new TornadoClient({
  network: 'base-sepolia',
  rpcUrl: 'https://sepolia.base.org',
  privateKey: process.env.PRIVATE_KEY, // Your wallet private key
});
```

### Step 2: Make a Deposit

```typescript
// Deposit 0.1 ETH into the privacy pool
const deposit = await client.deposit({
  denomination: '0.1',
  asset: 'ETH',
});

console.log('Deposit successful!');
console.log('Save this note securely:');
console.log(deposit.note);
// Example note: tornado-eth-0.1-base-sepolia-0x1234...abcd

// IMPORTANT: Save this note! You'll need it to spend the deposit
```

### Step 3: Wait for Anonymity

```typescript
// Check anonymity set size
const poolInfo = await client.getPoolInfo('0.1', 'ETH');

console.log(`Current anonymity set: ${poolInfo.totalDeposits}`);
console.log(`Recommended minimum: 50`);

if (poolInfo.totalDeposits < 50) {
  console.log('⏳ Wait for more deposits to improve privacy');
  console.log('You can proceed now, but privacy will be limited');
}
```

### Step 4: Make an Anonymous Payment

```typescript
// Pay for a protected resource anonymously
const response = await client.payAnonymously({
  resourceUrl: 'https://api.example.com/premium-data',
  note: deposit.note, // Or a previously saved note
  relayerUrl: 'https://relayer.x402tornado.network',
});

console.log('Payment successful!');
console.log('Resource:', response.data);
```

### Complete Example

```typescript
import { TornadoClient } from 'x402tornado-client';

async function main() {
  // Initialize
  const client = new TornadoClient({
    network: 'base-sepolia',
    rpcUrl: 'https://sepolia.base.org',
    privateKey: process.env.PRIVATE_KEY,
  });

  // Deposit
  const deposit = await client.deposit({
    denomination: '0.1',
    asset: 'ETH',
  });
  
  console.log('✅ Deposit confirmed');
  console.log('📝 Note:', deposit.note);
  console.log('⏳ Waiting for anonymity set to grow...');
  
  // Wait for minimum anonymity
  await client.waitForAnonymitySet('0.1', 'ETH', 50);
  
  console.log('✅ Anonymity set reached 50+ deposits');
  
  // Make payment
  const data = await client.payAnonymously({
    resourceUrl: 'https://api.example.com/premium-data',
    note: deposit.note,
    relayerUrl: 'https://relayer.x402tornado.network',
  });
  
  console.log('✅ Payment successful');
  console.log('📊 Data received:', data);
}

main().catch(console.error);
```

</tab>

<tab title="Python">

### Installation

```bash
pip install x402tornado
```

### Step 1: Initialize Client

```python
from x402tornado import TornadoClient
import os

client = TornadoClient(
    network='base-sepolia',
    rpc_url='https://sepolia.base.org',
    private_key=os.getenv('PRIVATE_KEY'),
)
```

### Step 2: Make a Deposit

```python
# Deposit 0.1 ETH into the privacy pool
deposit = client.deposit(
    denomination='0.1',
    asset='ETH',
)

print('Deposit successful!')
print('Save this note securely:')
print(deposit.note)
# Example note: tornado-eth-0.1-base-sepolia-0x1234...abcd

# IMPORTANT: Save this note! You'll need it to spend the deposit
```

### Step 3: Wait for Anonymity

```python
# Check anonymity set size
pool_info = client.get_pool_info('0.1', 'ETH')

print(f'Current anonymity set: {pool_info.total_deposits}')
print(f'Recommended minimum: 50')

if pool_info.total_deposits < 50:
    print('⏳ Wait for more deposits to improve privacy')
    print('You can proceed now, but privacy will be limited')
```

### Step 4: Make an Anonymous Payment

```python
# Pay for a protected resource anonymously
response = client.pay_anonymously(
    resource_url='https://api.example.com/premium-data',
    note=deposit.note,  # Or a previously saved note
    relayer_url='https://relayer.x402tornado.network',
)

print('Payment successful!')
print('Resource:', response.data)
```

### Complete Example

```python
from x402tornado import TornadoClient
import os

def main():
    # Initialize
    client = TornadoClient(
        network='base-sepolia',
        rpc_url='https://sepolia.base.org',
        private_key=os.getenv('PRIVATE_KEY'),
    )

    # Deposit
    deposit = client.deposit(
        denomination='0.1',
        asset='ETH',
    )
    
    print('✅ Deposit confirmed')
    print(f'📝 Note: {deposit.note}')
    print('⏳ Waiting for anonymity set to grow...')
    
    # Wait for minimum anonymity
    client.wait_for_anonymity_set('0.1', 'ETH', min_size=50)
    
    print('✅ Anonymity set reached 50+ deposits')
    
    # Make payment
    data = client.pay_anonymously(
        resource_url='https://api.example.com/premium-data',
        note=deposit.note,
        relayer_url='https://relayer.x402tornado.network',
    )
    
    print('✅ Payment successful')
    print(f'📊 Data received: {data}')

if __name__ == '__main__':
    main()
```

</tab>
</tabs>

## Understanding Your Note

The deposit note is your **secret key** to spending the deposit. It contains:

```
tornado-eth-0.1-base-sepolia-0x1234567890abcdef...
│       │   │   │            │
│       │   │   │            └─ Secrets (k, r, leafIndex)
│       │   │   └─ Network
│       │   └─ Denomination
│       └─ Asset
└─ Protocol
```

**⚠️ CRITICAL: Keep your note secret and secure!**

- Store it in a password manager
- Never share it with anyone
- Back it up in multiple secure locations
- Anyone with the note can spend the deposit

## Testing the Flow

### Use the Demo Merchant

We provide a demo merchant for testing:

```bash
# Test endpoint that requires 0.001 ETH payment
curl https://demo.x402tornado.network/api/echo

# Response: 402 Payment Required
```

### Make a Test Payment

<tabs>
<tab title="TypeScript">

```typescript
// Small deposit for testing
const deposit = await client.deposit({
  denomination: '0.001',
  asset: 'ETH',
});

// Wait a bit (or check anonymity set)
await new Promise(resolve => setTimeout(resolve, 60000)); // 1 minute

// Test payment
const response = await client.payAnonymously({
  resourceUrl: 'https://demo.x402tornado.network/api/echo',
  note: deposit.note,
  relayerUrl: 'https://relayer.x402tornado.network',
});

console.log('Echo response:', response.data);
// { message: "Payment successful! This is your echo response." }
```

</tab>

<tab title="Python">

```python
import time

# Small deposit for testing
deposit = client.deposit(
    denomination='0.001',
    asset='ETH',
)

# Wait a bit (or check anonymity set)
time.sleep(60)  # 1 minute

# Test payment
response = client.pay_anonymously(
    resource_url='https://demo.x402tornado.network/api/echo',
    note=deposit.note,
    relayer_url='https://relayer.x402tornado.network',
)

print('Echo response:', response.data)
# { "message": "Payment successful! This is your echo response." }
```

</tab>
</tabs>

## Building a Merchant Server

Want to accept anonymous payments in your API?

<tabs>
<tab title="Express">

```typescript
import express from 'express';
import { tornadoMiddleware } from 'x402tornado-express';

const app = express();

app.use(
  tornadoMiddleware({
    facilitatorUrl: 'https://facilitator.x402tornado.network',
    payTo: '0xYourAddress',
    routes: {
      '/premium-data': {
        denomination: '0.1',
        asset: 'ETH',
        network: 'base-sepolia',
      },
    },
  })
);

app.get('/premium-data', (req, res) => {
  res.json({
    data: 'This is premium data!',
    timestamp: Date.now(),
  });
});

app.listen(3000);
```

</tab>

<tab title="FastAPI">

```python
from fastapi import FastAPI
from x402tornado.fastapi import tornado_middleware

app = FastAPI()

app.middleware("http")(
    tornado_middleware(
        facilitator_url='https://facilitator.x402tornado.network',
        pay_to_address='0xYourAddress',
        path='/premium-data',
        denomination='0.1',
        asset='ETH',
        network='base-sepolia',
    )
)

@app.get("/premium-data")
async def premium_data():
    return {
        "data": "This is premium data!",
        "timestamp": 1703123456
    }
```

</tab>
</tabs>

## Understanding Privacy

### Anonymity Set Size Matters

| Anonymity Set | Privacy Level | When to Withdraw |
|---------------|---------------|------------------|
| 1-10 | ⚠️ Very Low | Avoid if possible |
| 10-50 | ⚠️ Low | Only for testing |
| 50-100 | ✅ Medium | Acceptable |
| 100-500 | ✅ Good | Recommended |
| 500+ | ✅ Excellent | Best privacy |

### Privacy Tips

1. **Wait for mixing**: Let your deposit sit for hours/days as others deposit
2. **Use popular pools**: Larger denomination pools have better anonymity
3. **Use relayers**: Don't withdraw directly to avoid IP address leakage
4. **Vary timing**: Don't withdraw immediately after depositing
5. **Multiple deposits**: Split large amounts across multiple deposits

## Common Issues

### "Insufficient anonymity set"

**Problem**: Pool has too few deposits for good privacy.

**Solution**: 
- Wait for more users to deposit
- Use a more popular denomination
- Accept lower privacy for testing

### "Invalid proof verification"

**Problem**: Your zero-knowledge proof is invalid.

**Solution**:
- Ensure you're using the correct note
- Check the Merkle tree is synced
- Verify network matches the deposit
- Regenerate the proof

### "Nullifier already used"

**Problem**: You're trying to spend a deposit that's already been withdrawn.

**Solution**:
- You can only withdraw each deposit once
- Check if you already spent this note
- Use a different deposit

### "Unknown Merkle root"

**Problem**: The Merkle root in your proof is too old.

**Solution**:
- Fetch the latest Merkle tree state
- Regenerate your proof with current root
- Ensure your client is in sync

## Next Steps

Now that you've completed the quick start:

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
      <td><strong>Understand Privacy</strong></td>
      <td>Learn how x402tornado protects your privacy</td>
      <td><a href="../core-concepts/privacy-in-micropayments.md">privacy-in-micropayments.md</a></td>
    </tr>
    <tr>
      <td><strong>Architecture Deep Dive</strong></td>
      <td>Explore the technical architecture</td>
      <td><a href="../how-it-works/architecture.md">architecture.md</a></td>
    </tr>
    <tr>
      <td><strong>Build a Merchant</strong></td>
      <td>Accept anonymous payments in your API</td>
      <td><a href="../implementation/servers/setup.md">setup.md</a></td>
    </tr>
    <tr>
      <td><strong>Advanced Features</strong></td>
      <td>Multi-chain, batch payments, and more</td>
      <td><a href="../advanced/denomination-pools.md">denomination-pools.md</a></td>
    </tr>
  </tbody>
</table>

## Get Help

- 💬 **Discord**: [Join our community](https://discord.gg/x402tornado)
- 📖 **Docs**: Browse the full documentation
- 🐛 **GitHub**: [Report issues](https://github.com/x402tornado/x402tornado/issues)
- 🐦 **Twitter**: [@x402tornado](https://twitter.com/x402tornado)

---

**Ready to build privacy-preserving applications?** Continue to the [Core Concepts](../core-concepts/privacy-in-micropayments.md) to deepen your understanding.