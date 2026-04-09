# BlackEdge — Open Items for Discussion
**Date**: March 5, 2026  
**Status**: Draft — Pending Team Resolution  
**Related**: [PRD-BlackEdge.md](PRD-BlackEdge.md)

---

## Item 1: Gas Strategy — Smart Wallet (Gasless) vs. On-Chain Gas

### Context

Every Solana transaction requires a small SOL fee ("gas"). BlackEdge users deposit USDC only — they never hold SOL. The platform must decide how transaction gas is handled for user-facing operations (deposits, allocations, trades, withdrawals).

**Assumption**: The user ultimately bears the gas cost in both options. The question is **how** — either visibly as on-chain SOL, or invisibly via a smart wallet provider (whose subscription cost is passed through or absorbed).

---

### Option A: Smart Wallet Provider (Gasless UX)

Use a third-party smart wallet infrastructure provider (e.g., **Crossmint**, **Privy**, **Turnkey**) that abstracts gas away from the user. The provider sponsors gas on behalf of the user, and BlackEdge pays the provider a subscription / per-wallet fee.

**How it works**: The provider maintains a gas treasury. Transactions are relayed through their infrastructure. Users sign intents, not raw Solana transactions — the provider wraps them, pays gas, and settles on-chain.

| Dimension | Details |
|-----------|---------|
| **User Experience** | ✅ Seamless — users never see SOL, never need to "fund gas." Pure USDC experience. Lowest possible friction for non-crypto-native users. |
| **Onboarding** | ✅ No "buy SOL first" step. Eliminates a major known churn point in crypto apps. Users sign up, deposit USDC, and go. |
| **Security** | ✅ Wallet keys managed by the provider's HSM/TEE infrastructure (e.g., Turnkey uses secure enclaves). Reduces the surface area BlackEdge needs to secure directly. However, introduces **dependency on a third party** for key custody. |
| **Cost to BlackEdge** | ⚠️ Subscription-based. Crossmint charges **~$0.05/Monthly Active Wallet (MAW)** with volume discounts. At 1,000 MAW = ~$50/mo; at 100K MAW = negotiated. Additional costs for premium features (webhooks, analytics, advanced auth). |
| **Cost to User** | ✅ Zero visible gas cost. The subscription cost can be absorbed by BlackEdge (at small scale) or passed through as part of the management/platform fee (at scale). |
| **Implementation** | ⚠️ Requires integrating the provider's SDK. Wallet creation, transaction signing, and gas sponsorship are all API calls. Less control over transaction construction. Must handle provider outages gracefully. |
| **Vendor Lock-in** | ❌ High — wallet keys may be custodied by the provider. Migrating away requires key export (if supported) or user re-onboarding. Evaluate provider key portability carefully. |
| **Limitations** | ⚠️ Provider may impose rate limits, transaction size limits, or supported program restrictions. Need to verify compatibility with Jupiter (DEX aggregator), SPL token transfers, and any custom Solana programs BlackEdge uses. |
| **Transparency** | ⚠️ Gas cost is abstracted — user doesn't see it. Fine for UX, but must be disclosed in Terms of Service if costs are passed through. |

**Known Providers (Solana-compatible)**:

| Provider | Pricing Model | Gas Sponsorship | Key Custody | Notes |
|----------|--------------|-----------------|-------------|-------|
| **Crossmint** | $0.05/MAW (volume discounts) | ✅ Built-in, app-sponsored | Provider-managed | Broad Solana support. Smart wallets with embedded signing. |
| **Privy** | Free tier + usage-based | ✅ Via embedded wallets | Provider-managed (with export) | Auth + wallet in one. Social login support. |
| **Turnkey** | Usage-based (API calls) | ⚠️ BYO gas sponsorship | HSM/TEE secure enclaves | More DIY — you build the gas layer. Most flexible, most work. |

---

### Option B: On-Chain Gas (User Pays SOL Directly)

Users' custodial wallets hold a small SOL balance for gas. BlackEdge either (a) auto-converts a small fraction of each USDC deposit to SOL, or (b) requires users to deposit SOL separately.

| Dimension | Details |
|-----------|---------|
| **User Experience** | ❌ Users must understand SOL gas, see a second token in their wallet, and potentially manage a SOL balance. Adds cognitive load. |
| **Onboarding** | ❌ "You need SOL for gas" is a known friction point. Even auto-conversion adds a step and explanation. New crypto users may not understand why. |
| **Security** | ✅ Standard Solana transaction signing. No third-party dependency for key custody — BlackEdge fully controls wallet infrastructure. |
| **Cost to BlackEdge** | ✅ Zero platform cost for gas — user pays directly. |
| **Cost to User** | ⚠️ Tiny but visible. Solana gas is ~$0.001/tx (5,000 lamports). Over 1,000 transactions that's ~$1. Negligible in dollar terms, but the **experience of managing SOL** is the real cost. |
| **Implementation** | ✅ Standard Solana transaction construction. No third-party SDK dependency. Full control over transaction building, priority fees, retry logic. |
| **Vendor Lock-in** | ✅ None — standard Solana infrastructure. |
| **Limitations** | ❌ User can get "stuck" — they have USDC but no SOL to transact. Requires a rescue mechanism (platform-funded emergency gas top-up, or auto-swap). |
| **Transparency** | ✅ User sees exactly what they pay. Every transaction shows gas cost. |

**Sub-option B.1 — Auto-Convert on Deposit**:
- On each USDC deposit, auto-swap a small amount (e.g., 0.01 SOL worth) to SOL to fund the user's gas reserve.
- Pro: User never manually buys SOL. Con: Adds complexity, requires a SOL/USDC swap mechanism, and the user still "sees" SOL in their wallet.

**Sub-option B.2 — Manual SOL Deposit**:
- User deposits SOL separately. Not recommended — extremely high friction.

---

### Comparison Matrix

| Criterion | Option A (Smart Wallet) | Option B (On-Chain Gas) |
|-----------|:---:|:---:|
| User experience | ✅✅ | ❌ |
| Onboarding friction | ✅✅ | ❌ |
| Security (self-custody) | ⚠️ (third-party) | ✅✅ |
| Platform cost | ⚠️ ($0.05/MAW+) | ✅✅ |
| Implementation simplicity | ⚠️ (SDK integration) | ✅✅ |
| Vendor independence | ❌ | ✅✅ |
| Solana program compatibility | ⚠️ (verify per provider) | ✅✅ |
| User gas visibility | Hidden | Visible |

### Decision Points

> [!IMPORTANT]
> 1. **Does BlackEdge want to own wallet infrastructure or outsource it?** Smart wallet providers bundle custody + gas + auth. Going on-chain gas means BlackEdge builds its own custody layer.
> 2. **Is the $0.05/MAW cost acceptable at target scale?** At 10K users = ~$500/mo; at 100K = ~$5K/mo (before volume discounts). Is this worth the UX improvement?
> 3. **Key portability**: If we go with a smart wallet provider and later want to switch, can we export user keys? This must be confirmed before committing.
> 4. **Program compatibility**: Will the provider's smart wallet work with Jupiter, SPL token operations, and any custom programs BlackEdge deploys?

---
---

## Item 2: Fee Proration Across Partial Allocations and Unallocations

### Context

BlackEdge charges two fees per AI Fund Manager:

| Fee | Rate | Trigger |
|-----|------|---------|
| **Management Fee** | 1% annually, prorated by time | Collected upon unallocation or full close |
| **Performance Fee** | 20% of net profit | Collected upon unallocation or full close |

The fee model is straightforward for a **full close** (user unallocates 100%). The complexity arises with **partial unallocations**, **top-up allocations**, and **multiple rounds** of both over time.

### Proposed Method: Pro-Rata by Withdrawal Fraction

When a user unallocates **$X** from an AI Manager with **Current Value (CV)**, the system calculates fees on the **proportional slice** of the portfolio being withdrawn.

**Core formula**:

```
withdrawal_fraction = X / CV
proportional_basis  = withdrawal_fraction × allocation_basis
proportional_profit = X − proportional_basis
management_fee      = 1% × proportional_basis × (days_held / 365)
performance_fee     = 20% × max(0, proportional_profit)
user_receives       = X − management_fee − performance_fee
```

After each unallocation, the remaining `allocation_basis` and `blended_start_date` are updated proportionally.

---

### Scenario A: Full Close

> User allocates $1,000. After 6 months, AI grows portfolio to $1,200. User unallocates 100%.

```
withdrawal_fraction = $1,200 / $1,200 = 100%
proportional_basis  = 100% × $1,000 = $1,000
proportional_profit = $1,200 − $1,000 = $200
```

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Management Fee | 1% × $1,000 × (6/12) | **$5.00** |
| Performance Fee | 20% × $200 | **$40.00** |
| **User Receives** | $1,200 − $5 − $40 | **$1,155.00** |

✅ Baseline case — already defined in the PRD.

---

### Scenario B: Partial Unallocation

#### B.1 — Partial Close, Profitable

> User allocates $1,000. After 6 months, AI grows portfolio to $1,200 (CV). User unallocates $600 (50% of CV).

```
withdrawal_fraction = $600 / $1,200 = 50%
proportional_basis  = 50% × $1,000 = $500
proportional_profit = $600 − $500 = $100
```

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Management Fee | 1% × $500 × (6/12) | **$2.50** |
| Performance Fee | 20% × $100 | **$20.00** |
| **User Receives** | $600 − $2.50 − $20 | **$577.50** |

**Post-unallocation state**:
- Remaining CV: $600
- Remaining allocation basis: $500
- Start date: unchanged (Month 0)

When the user later unallocates the remaining $600, fees are calculated on the remaining $500 basis from the original start date.

#### B.2 — Partial Close, At a Loss

> User allocates $1,000. After 6 months, AI drops portfolio to $800. User unallocates $400 (50% of CV).

```
withdrawal_fraction = $400 / $800 = 50%
proportional_basis  = 50% × $1,000 = $500
proportional_profit = $400 − $500 = −$100 (loss)
```

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Management Fee | 1% × $500 × (6/12) | **$2.50** |
| Performance Fee | 20% × max($0, −$100) | **$0.00** |
| **User Receives** | $400 − $2.50 | **$397.50** |

✅ No performance fee when the proportional slice is at a loss.

#### B.3 — Partial Close, Then Full Close of Remainder (With Market Drop)

> Month 0: Allocate $1,000.
> Month 6: CV = $1,200. Unallocate $600 (50%).
> Month 12: Remaining CV = $800 (market dropped). Unallocate 100% of remainder.

**First unallocation (Month 6)**:

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Proportional basis | 50% × $1,000 | $500 |
| Proportional profit | $600 − $500 | $100 |
| Management Fee | 1% × $500 × (6/12) | **$2.50** |
| Performance Fee | 20% × $100 | **$20.00** |
| **User Receives** | $600 − $2.50 − $20 | **$577.50** |

Post-state: Remaining basis = $500, start = Month 0.

**Second unallocation (Month 12)** — full close of remaining:

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Remaining basis | — | $500 |
| Profit | $800 − $500 | $300 |
| Management Fee | 1% × $500 × (12/12) | **$5.00** |
| Performance Fee | 20% × $300 | **$60.00** |
| **User Receives** | $800 − $5 − $60 | **$735.00** |

**Integrity check**:
| | Amount |
|--|--------|
| Total CV withdrawn | $600 + $800 = **$1,400** |
| Total fees paid | $2.50 + $20 + $5 + $60 = **$87.50** |
| Total received | $577.50 + $735.00 = **$1,312.50** |
| Check | $1,312.50 + $87.50 = $1,400 ✅ |

---

### Scenario C: Multiple Allocations + Unallocations

#### C.1 — Allocate Top-Up, Then Partial Unallocate

> Month 0: Allocate $1,000 (Tranche 1).
> Month 3: AI grows CV to $1,100. User allocates $500 more (Tranche 2). New total allocated = $1,500.
> Month 6: AI grows total CV to $1,800. User unallocates $900 (50% of CV).

**On top-up (Month 3)**, the system recalculates a **blended allocation basis** and **blended start date** (weighted by capital):

| | Tranche 1 | Tranche 2 | Blended |
|--|-----------|-----------|---------|
| Capital | $1,000 | $500 | **$1,500** |
| Start | Month 0 | Month 3 | weighted avg |
| Weight | 66.7% | 33.3% | — |
| **Blended start** | — | — | 0×0.667 + 3×0.333 = **Month 1** |

**Unallocation at Month 6**:

```
withdrawal_fraction = $900 / $1,800 = 50%
proportional_basis  = 50% × $1,500 = $750
proportional_profit = $900 − $750 = $150
time_held           = Month 6 − Month 1 = 5 months
```

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Management Fee | 1% × $750 × (5/12) | **$3.13** |
| Performance Fee | 20% × $150 | **$30.00** |
| **User Receives** | $900 − $3.13 − $30 | **$866.87** |

Post-state: Remaining basis = $750, blended start = Month 1.

#### C.2 — Multiple Allocations, Multiple Unallocations

> Month 0: Allocate $1,000.
> Month 3: CV = $1,050. Allocate $500. Blended basis = $1,500, blended start = Month 1.
> Month 6: CV = $1,800. Unallocate $600 (33.3% of CV).
> Month 9: CV = $1,400 (remaining). Allocate $200. Re-blend.
> Month 12: CV = $1,900. Unallocate all (100%).

**Unallocation 1 (Month 6)** — 33.3% of CV:

```
proportional_basis  = 33.3% × $1,500 = $500
proportional_profit = $600 − $500 = $100
time_held           = Month 6 − Month 1 = 5 months
```

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Management Fee | 1% × $500 × (5/12) | **$2.08** |
| Performance Fee | 20% × $100 | **$20.00** |
| **User Receives** | $600 − $2.08 − $20 | **$577.92** |

Post-state: Remaining basis = $1,000, blended start = Month 1, remaining CV = $1,200.

**Re-allocation (Month 9)** — $200 top-up into remaining $1,400 CV:

New blended basis = $1,000 + $200 = **$1,200**.
New blended start = weighted avg of Month 1 (weight $1,000) and Month 9 (weight $200):
= (1 × 1000 + 9 × 200) / 1200 = **Month 2.33**.

**Unallocation 2 (Month 12)** — full close of $1,900:

```
proportional_basis  = 100% × $1,200 = $1,200
proportional_profit = $1,900 − $1,200 = $700
time_held           = Month 12 − Month 2.33 = 9.67 months
```

| Line Item | Calculation | Amount |
|-----------|-------------|--------|
| Management Fee | 1% × $1,200 × (9.67/12) | **$9.67** |
| Performance Fee | 20% × $700 | **$140.00** |
| **User Receives** | $1,900 − $9.67 − $140 | **$1,750.33** |

#### C.3 — Loss Offset: Per-Manager Net P&L

> The AI buys AAPLx at $180 (now worth $200, +$20 unrealized gain) and TSLAx at $250 (now worth $230, −$20 unrealized loss). User unallocates.

**Question**: Does the TSLAx loss offset the AAPLx gain for performance fee purposes?

| Approach | Behavior | Performance Fee |
|----------|----------|:---:|
| **Per-asset** | Charge 20% on AAPLx profit ($20), ignore TSLAx loss | $4.00 |
| **Per-manager net** | Net P&L = +$20 − $20 = $0. No profit. | **$0.00** |

✅ **Recommendation**: **Per-manager net P&L**. Losses offset gains. Performance fee only applies when the manager's overall return is positive at time of unallocation. The per-asset approach would charge users a fee even when they made no money overall — this is unfair and would erode trust.

---

### Implementation Requirements

The backend must track the following per AI Manager per user:

| Field | Type | Purpose |
|-------|------|---------|
| `allocation_basis` | decimal | Running total of capital allocated; reduced proportionally on each partial unallocation |
| `blended_start_date` | timestamp | Weighted-average start date across all allocation tranches; re-blended on each new allocation |
| `cumulative_fees_paid` | decimal | Total management + performance fees collected to date; guards against double-charging |

---

### Decisions Required

> [!IMPORTANT]
> 1. **Confirm pro-rata by withdrawal fraction** as the method for fee proration on partial unallocations.
> 2. **Confirm weighted-average blended basis** for calculating allocation basis and start date when users allocate additional funds.
> 3. **Confirm per-manager net P&L** (not per-asset) for performance fee calculation.
> 4. **High-water mark (future consideration)**: Should performance fees only apply on *new highs* above the previous peak? This prevents users from being charged twice after a drawdown and recovery. Not required for Phase 1, but the data model should accommodate it.

---

*Document maintained by: Product Team*  
*Created: March 5, 2026*
