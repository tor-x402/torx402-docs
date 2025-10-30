---
description: Understanding anonymity sets and how deposit volume affects privacy timing
---

# Anonymity Sets

## TL;DR: High Volume = Instant Privacy

### Privacy Timeline by Pool Volume

| Pool Volume | Deposits/Hour | Current Pool Size | Wait Time for Safe Withdrawal | Example Scenario |
|-------------|---------------|-------------------|-------------------------------|------------------|
| **🚀 Instant** | Any | **500+** | **< 2 seconds** ⚡ | Pool already has 1,247 deposits - withdraw immediately! |
| **⚡ Very High** | 50+ | 100-500 | **Minutes** | AI agents using 0.001 ETH pool - 15 min wait |
| **✅ High** | 20-50 | 50-100 | **1-2 hours** | Popular 0.01 ETH on Base - lunch break wait |
| **⏰ Medium** | 5-20 | 20-50 | **4-12 hours** | 0.1 ETH pool - overnight wait |
| **😴 Low** | 1-5 | 10-20 | **1-3 days** | Unpopular denomination - weekend wait |
| **💤 Very Low** | <1 | <10 | **Weeks+** | 10 ETH pool - DON'T USE for fast privacy |

### The 2-Second Withdrawal Example

```
🌪️ Ultra-Popular Pool: 0.001 ETH on Base

Current Status:
  📊 Pool Size: 1,247 deposits
  📈 Growth Rate: 73 deposits/hour
  🌐 Network: Base (low fees, high activity)
  💰 Denomination: 0.001 ETH (~$2)

Your Action:
  10:00:00 - You deposit
  10:00:02 - You withdraw! ✅

Privacy Analysis:
  ✅ Anonymity Set: 1,247 (EXCELLENT)
  ✅ Your deposit is 1 of 1,247 possibilities
  ✅ Probability of identification: 0.08%
  ✅ Information entropy: 10.3 bits

Why This Works:
  - Pool ALREADY has massive anonymity set
  - No waiting needed - privacy exists immediately
  - Every second, ~0.02 new deposits join
  - Your deposit instantly lost in crowd of 1,247

Real-World Use Case:
  - AI agent needs data feed NOW
  - Deposits 0.001 ETH
  - Immediately withdraws to pay merchant
  - Total time: 2 seconds
  - Privacy: Excellent (1 of 1,247)
```

**Key Insight:** You don't need to wait if the pool is already large! Privacy comes from the crowd size, and if the crowd is already huge (500+), you can withdraw instantly.

---

## What is an Anonymity Set?

An **anonymity set** is the group of all deposits in a privacy pool that could potentially be the source of a given withdrawal. It's the fundamental measure of privacy in torx402.

### The Basic Principle

When you withdraw from a privacy pool, observers can see:
- ✅ A withdrawal happened
- ✅ The amount (denomination)
- ✅ The recipient address

But they **cannot** determine:
- ❌ Which deposit was spent
- ❌ Who made the original deposit
- ❌ When the deposit was made

**Your privacy = How many deposits you could be confused with**

### Mathematical Definition

```
Anonymity Set Size = Total Deposits in Pool at Withdrawal Time

Privacy Level = 1 / Anonymity Set Size

Example:
  - Pool has 100 deposits
  - Your anonymity set = 100
  - Probability of identifying you = 1/100 = 1%
```

## The Volume-Privacy-Time Relationship

This is the KEY insight that answers your question:

### High Volume Pool: Fast Privacy ✅

```
Scenario: Popular 0.1 ETH pool on Base

Time 0:00  - You deposit (Total: 234 deposits)
Time 0:05  - 12 new deposits (Total: 246)
Time 0:10  - 15 new deposits (Total: 261)
Time 0:15  - 8 new deposits  (Total: 269)

✅ After 15 minutes: Anonymity set = 269
✅ You can withdraw immediately with STRONG privacy!
✅ Even if someone was watching, 269 possible sources!
```

**Why This Works:**
- High volume means rapid anonymity set growth
- More deposits = more "noise" to hide in
- Time becomes less important
- You get privacy through **crowd size**, not waiting

### Low Volume Pool: Slow Privacy ⏰

```
Scenario: Unpopular 10 ETH pool on small network

Time 0:00   - You deposit (Total: 8 deposits)
Time 1:00   - 1 new deposit (Total: 9)
Time 6:00   - 2 new deposits (Total: 11)
Time 24:00  - 3 new deposits (Total: 14)
Time 48:00  - Still only 23 deposits...

⚠️ After 48 hours: Anonymity set = 23
⚠️ WEAK privacy - only 1 of 23 possibilities
⚠️ Should wait longer or accept lower privacy
```

**Why This Is Slow:**
- Few users = slow growth
- Small anonymity set = weak privacy
- Must wait days/weeks for sufficient mixing
- Time is your only tool for privacy

## Privacy Thresholds

### Anonymity Set Guidelines

| Set Size | Privacy Level | Risk Level | Recommendation |
|----------|---------------|------------|----------------|
| **1-5** | ⛔ Critical | Very High | AVOID - Trivial to trace |
| **5-10** | ⚠️ Very Low | High | Only for testing |
| **10-25** | ⚠️ Low | Medium-High | Not recommended |
| **25-50** | ⚠️ Weak | Medium | Minimum for low-value |
| **50-100** | ✅ Acceptable | Low-Medium | OK for most use cases |
| **100-250** | ✅ Good | Low | Recommended minimum |
| **250-500** | ✅ Strong | Very Low | Good privacy |
| **500-1000** | ✅ Very Strong | Minimal | Strong privacy |
| **1000+** | ✅ Excellent | Negligible | Excellent privacy |

### Real-World Examples

#### Example 0: Instant Withdrawal (< 2 seconds)
```
Use Case: AI agent needs real-time data NOW
Amount: 0.001 ETH (~$2)
Frequency: Every few minutes
Privacy Need: HIGH (pattern protection)

Chosen Pool:
  ✅ 0.001 ETH denomination (micropayments)
  ✅ Current size: 1,247 deposits (EXCELLENT)
  ✅ Ultra-high volume pool (Base mainnet)

Wait Time:
  - Pool already has 1,247 deposits
  - Minimum threshold: 100 (✅ Far exceeded!)
  - Required wait: 0 seconds
  - Actual withdrawal time: < 2 seconds ⚡

Why This Is Possible:
  1. Popular denomination (micropayments)
  2. High-traffic network (Base)
  3. Constant usage (AI agents, content, APIs)
  4. Pool grown over weeks to 1,247
  5. New user benefits from existing crowd

Privacy Level: EXCELLENT
  - 1 of 1,247 = 0.08% chance of identification
  - Even sophisticated analysis fails
  - Perfect for time-sensitive payments
```

#### Example 1: AI Agent Paying for API
```
Use Case: Trading bot buying market data
Amount: 0.01 ETH (~$20)
Frequency: Every few hours
Privacy Need: HIGH (strategy protection)

Recommended Pool:
  ✅ 0.01 ETH denomination (exact match)
  ✅ Minimum 250+ anonymity set
  ✅ High volume pool (Base or Ethereum mainnet)

Wait Time with High Volume:
  - Deposits/hour: ~30
  - Current pool: 180
  - Need: 250
  - Wait: (250-180)/30 = 2.3 hours ✅
```

#### Example 2: Content Payment
```
Use Case: User buying article access
Amount: 0.001 ETH (~$2)
Frequency: Occasional
Privacy Need: MEDIUM (browsing habits)

Recommended Pool:
  ✅ 0.001 ETH denomination
  ✅ Minimum 100+ anonymity set
  ✅ Any network with volume

Wait Time with Medium Volume:
  - Deposits/hour: ~10
  - Current pool: 85
  - Need: 100
  - Wait: (100-85)/10 = 1.5 hours ✅
```

#### Example 3: Large B2B Payment
```
Use Case: Company paying for enterprise API
Amount: 10 ETH (~$20,000)
Frequency: Monthly
Privacy Need: CRITICAL (competitive intelligence)

Recommended Pool:
  ⚠️ 10 ETH pool likely has LOW volume

Strategy:
  Option A: Split into smaller denominations
    - 100x 0.1 ETH deposits
    - Use high-volume pool
    - Faster anonymity growth

  Option B: Use 10 ETH pool but WAIT
    - May take weeks to reach 100+ anonymity
    - Higher privacy when you do withdraw

  ✅ BEST: Split + High Volume Pool
```

## Volume Dynamics

### What Creates High Volume?

**1. Popular Denominations**
```
Most Popular (High Volume):
  - 0.001 ETH - Micropayments
  - 0.01 ETH  - Small purchases
  - 0.1 ETH   - Medium transactions
  - 1 ETH     - Large transactions

Less Popular (Low Volume):
  - 5 ETH, 10 ETH, 100 ETH - Rare
  - Odd amounts (0.37 ETH) - Never use!
```

**2. Network Effects**
```
More users → More deposits → Faster mixing → Better UX → More users
                                  ↑__________________________|
```

**3. Active Blockchains**
```
High Volume Networks:
  - Base (low fees, high activity)
  - Ethereum Mainnet (most liquidity)
  - Solana (fast, cheap)

Lower Volume:
  - Smaller L2s
  - Testnets
  - New networks
```

### Monitoring Pool Volume

Check pool statistics before depositing:

```typescript
// Example: Check pool health
const poolInfo = await client.getPoolInfo('0.1', 'ETH', 'base');

console.log(`
Pool Analysis:
--------------
Denomination: ${poolInfo.denomination} ${poolInfo.asset}
Network: ${poolInfo.network}
Total Deposits: ${poolInfo.totalDeposits}
Deposits (24h): ${poolInfo.deposits24h}
Avg Deposits/hour: ${poolInfo.depositsPerHour}

Privacy Assessment:
  Current Anonymity: ${poolInfo.totalDeposits}
  Estimated Wait for 100+: ${(100 - poolInfo.totalDeposits) / poolInfo.depositsPerHour} hours
  Estimated Wait for 250+: ${(250 - poolInfo.totalDeposits) / poolInfo.depositsPerHour} hours

Recommendation: ${
  poolInfo.totalDeposits >= 250 ? '✅ Withdraw anytime' :
  poolInfo.depositsPerHour > 10 ? '✅ Wait a few hours' :
  poolInfo.depositsPerHour > 2 ? '⚠️ Wait 12-24 hours' :
  '⚠️ Low volume - consider different pool'
}
`);
```

## The "Fast After" Question Answered

### Yes! High Volume = Fast Withdrawals

**Your Original Question:**
> "If there is a high volume of deposits (and all being of low amounts), does it mean that withdraw could be done very fast after?"

**Answer: ABSOLUTELY YES! ✅**

**Even Better:** If the pool ALREADY has high volume (500+ deposits), you can withdraw **under 2 seconds**! You don't even need to wait - the anonymity set already exists.

### Why This Is TRUE:

**1. Privacy is About Crowd Size, Not Time**
```
Traditional thinking: "Wait longer = more private"
torx402 reality: "Bigger crowd = more private"

High volume pool:
  - 50 deposits/hour
  - After 2 hours: +100 deposits
  - Your deposit lost in crowd of 100+
  - Privacy achieved in 2 hours! ✅
```

**2. Small Amounts → More Participants**
```
0.001 ETH pool: Thousands of users
  - Micropayments
  - AI agents
  - Content access
  - Frequent use
  → High volume, fast mixing

10 ETH pool: Dozens of users
  - Large transfers
  - Rare use
  - Few participants
  → Low volume, slow mixing
```

**3. The Mathematics**

Privacy is measured by **information entropy**:

```
Entropy = log₂(Anonymity Set Size)

Small anonymity set:
  - 10 deposits = 3.3 bits of privacy
  - Easy to narrow down

Large anonymity set:
  - 100 deposits = 6.6 bits of privacy
  - 1000 deposits = 10 bits of privacy
  - Exponentially harder to trace
```

### Practical Examples: Real-Time Scenarios

#### Scenario 1: Instant Withdrawal (< 2 seconds)
```
🌪️ torx402 0.001 ETH Pool on Base (Micropayments)

10:00:00 - You deposit (Pool: 1,247 deposits)
           ✅ MASSIVE anonymity set already!

10:00:02 - You withdraw! 🎉⚡

Privacy achieved: Your deposit is 1 of 1,247
Time elapsed: 2 seconds
Recommendation: ✅ INSTANT - withdraw NOW!

Why this works:
  - Huge baseline (1,247 already in pool)
  - No waiting needed
  - Anonymity set far exceeds minimum (100+)
  - Perfect for time-sensitive payments
  - Your deposit is 0.08% identifiable
```

#### Scenario 2: Fast Withdrawal (15 minutes)
```
🌪️ torx402 0.01 ETH Pool on Base

09:00 AM - You deposit (Pool: 287 deposits)
           ✅ Already good privacy!

09:05 AM - +8 deposits  (Pool: 295)
09:10 AM - +12 deposits (Pool: 307)
09:15 AM - +7 deposits  (Pool: 314)

09:15 AM - You withdraw! 🎉

Privacy achieved: Your deposit is 1 of 314
Time elapsed: 15 minutes
Recommendation: ✅ EXCELLENT - withdraw immediately!

Why this works:
  - High baseline (287 already in pool)
  - Active mixing (27 new deposits in 15 min)
  - Large anonymity set (314 total)
  - Your deposit completely lost in crowd
```

## Strategic Waiting vs. Volume Waiting

### Two Types of Waiting

**Volume Waiting (What You Asked About)**
```
Goal: Wait for ENOUGH deposits
Strategy: Monitor pool growth
Success: When anonymity set reaches threshold

High Volume Pool:
  ⏱️ Minutes to hours
  ✅ Predictable
  ✅ Fast

Low Volume Pool:
  ⏱️ Days to weeks
  ⚠️ Unpredictable
  ⚠️ Slow
```

**Strategic Waiting (Additional Privacy)**
```
Goal: Randomize timing patterns
Strategy: Wait beyond minimum threshold
Success: Harder to correlate deposit/withdrawal

Even with 250+ anonymity:
  - Wait random time (6-48 hours)
  - Withdraw at random hour
  - Breaks timing correlation
  - Extra privacy layer
```

### Optimal Strategy

**For High Volume Pools:**
```
1. Deposit
2. Check pool size
3. If already >100: Can withdraw immediately
4. If <100: Wait until threshold reached
5. Optional: Add random delay for extra privacy
6. Withdraw via relayer
```

**For Low Volume Pools:**
```
1. DON'T USE if you need fast privacy
2. If you must use:
   - Deposit
   - Wait days/weeks
   - Monitor growth
   - Withdraw only when >100
3. Better: Split to smaller denominations in high-volume pool
```

## Common Misconceptions

### ❌ Myth 1: "Must Wait X Days"
**Reality:** No fixed waiting period. Wait until anonymity set is large enough.

### ❌ Myth 2: "Newer Deposits Are Suspicious"
**Reality:** In a pool of 300, timestamp doesn't matter. Any of 300 could be source.

### ❌ Myth 3: "More Waiting Always Better"
**Reality:** 200 vs 250 anonymity set is marginal. 10 vs 100 is critical.

### ❌ Myth 4: "Volume Doesn't Matter If I Wait"
**Reality:** Low volume means you might wait weeks. High volume = hours.

## Best Practices

### ✅ DO:
1. **Check pool volume BEFORE depositing**
2. **Use popular denominations** (0.001, 0.01, 0.1, 1 ETH)
3. **Choose high-volume networks** (Base, Ethereum, Solana)
4. **Monitor pool growth** in real-time
5. **Withdraw when threshold reached** (100+)
6. **Use relayers** for extra privacy

### ❌ DON'T:
1. **Use empty or tiny pools** (<10 deposits)
2. **Withdraw immediately** if pool <50
3. **Use unpopular denominations**
4. **Assume time alone = privacy**
5. **Forget to check current anonymity set**

## Future: Volume Incentives

torx402 may implement **liquidity mining** for privacy pools:

```
Concept: Reward early depositors for improving anonymity sets

Example:
  - Deposit in new pool (size: 5)
  - Earn tokens for bootstrapping liquidity
  - Pool grows to 100
  - Everyone benefits from strong privacy
  - Early depositors rewarded for waiting/risk
```

## Summary

**Your Question:** High volume + low amounts = fast withdrawals?

**Answer:** YES! ✅

**Key Insights:**
1. Privacy = Anonymity set size (not time)
2. High volume = Rapid anonymity growth
3. Small amounts = More participants = Higher volume
4. Can withdraw in minutes/hours vs days/weeks
5. Always check pool size before withdrawing

**Golden Rule:**
> "Privacy comes from crowds, not clocks. A high-volume pool gives you a crowd in minutes."

**Practical Advice:**
- Use 0.01 or 0.1 ETH pools on Base
- These typically have 100+ deposits/day
- Wait 1-6 hours for strong privacy
- Much faster than low-volume pools!

---

## Next Steps

- [Understanding Zero-Knowledge Proofs](zero-knowledge-proofs.md)
- [Optimizing Anonymity](../advanced/optimizing-anonymity.md)
- [Denomination Pool Strategy](../advanced/denomination-pools.md)
