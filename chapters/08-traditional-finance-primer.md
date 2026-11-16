# Chapter 8: Traditional Finance Primer

> **Goal:** After this chapter you can explain bonds, yields, funds, NAV, private credit, CLOs, and gates.

---

## 🧒 Like you're 12

When you put money in a savings account, the bank pays you interest because they lend your money to someone else. A **bond** is basically an IOU: you lend money to a company or government, and they promise to pay you back with interest.

A **fund** is when a bunch of people pool their money and a professional manager invests it. The fund's **NAV** (Net Asset Value) is the total value of everything the fund owns, divided by the number of shares.

**Private credit** is lending to companies directly instead of through a bank. It pays higher interest but your money is locked up — you can't just sell whenever you want.

---

## 🛠️ Engineer view

### Bonds: the simplest debt

```
Day 0:    You lend $1,000 to a company (buy the bond)
Every 6 months: The company pays you $25 (coupon = 5% annual / 2)
Day 365:  The company returns your $1,000 (face value / principal)
```

Key terms:
- **Face value / Par:** the amount paid back at maturity ($1,000)
- **Coupon:** the periodic interest payment (5% = $50/year)
- **Maturity:** when the bond expires and principal is returned
- **Yield:** your effective return, considering the price you paid
- **Credit risk:** the chance the borrower doesn't pay back

### Yield: what you actually earn

| Term | Meaning |
|------|---------|
| **Coupon rate** | The fixed interest rate on the bond (e.g. 5%) |
| **Current yield** | Coupon ÷ current market price |
| **Yield to maturity (YTM)** | Total return if held to maturity, including price gain/loss |
| **APY** | Annual Percentage Yield — includes compounding |
| **APR** | Annual Percentage Rate — does not include compounding |

**US Treasury bonds** are considered "risk-free" — the US government has never defaulted. Their yield is the baseline everything else is measured against.

### Funds: pooled investments

| Type | What it invests in | Liquidity | Minimum |
|------|-------------------|-----------|---------|
| **Mutual fund** | Stocks, bonds | Daily redemption | $1,000–$100K |
| **ETF** | Stocks, bonds, commodities | Trades on exchange all day | 1 share |
| **Hedge fund** | Anything (long, short, leverage) | Quarterly or annual | $1M+ |
| **Private credit fund** | Direct loans to companies | Quarterly, with gates | $100K+ |

### NAV: Net Asset Value

```
NAV per share = Total assets of the fund - Total liabilities
                ────────────────────────────────────────────
                       Number of outstanding shares
```

For a fund holding $100M in bonds with 10M shares, NAV = $10 per share. Earlyout reads NAV from **Chainlink price feeds** that report it daily from real data sources.

### Private credit: why it matters for Earlyout

Private credit funds lend directly to companies (not through banks). They earn 8–12% yields but with restrictions:

- **Lock-up period:** money is locked for 1–3 years
- **Redemption windows:** you can only request to withdraw once per quarter
- **Gates:** the fund can limit how much can be withdrawn (e.g. max 5% per quarter)
- **NAV lag:** the fund's NAV is calculated monthly or quarterly, not in real-time

**Apollo's ACRED** (Securitize) is a tokenized version of Apollo's credit fund. It has quarterly redemption windows and a minimum 5% repurchase per quarter.

**The problem Earlyout solves:** An ACRED holder who needs cash before the next quarterly window is stuck. Earlyout lets them sell their pending redemption *now*, taking the discount (the spread) instead of waiting 90 days.

### CLOs: Collateralised Loan Obligations

A CLO is a structured product that bundles many corporate loans together and slices them into tranches:

```
AAA Tranche (senior): gets paid first, lowest yield, lowest risk
AA Tranche:           next in line
BBB Tranche:          higher yield, higher risk
Equity Tranche:       gets whatever's left, highest risk and return
```

This is relevant because many private credit funds invest in or manage CLOs.

### Settlement in traditional finance

```
You sell stock today (Trade date: T)
Settlement: T+1 (US stocks since 2024, was T+2 before)
Your cash arrives: next business day

You sell a fund share (Redemption request)
Fund calculates NAV: end of month or quarter
Fund processes redemption: next window
Your cash arrives: days to weeks after the window
```

**On-chain with Earlyout:**
```
You sell a claim (depositAndSell)
USDC arrives: same transaction (< 2 seconds on Arc)
```

This is the fundamental improvement: settlement goes from days/weeks to seconds.

---

## 🔗 In Earlyout

- The Vault prices claims using **NAV** from Chainlink feeds, the same number a traditional fund uses.
- **FundRegistry** stores each fund's settlement delay (90 days for ACRED, 7 days for JAAA, etc.) and volatility terms.
- The **spread** = base + utilisation + volatility × √(time to payout) + credit — these are the same risk factors a traditional finance desk would use.
- **IssuerStub** simulates a fund's redemption desk: queue the request, pay at NAV after the settlement delay.

---

## ✅ Check yourself

1. What is NAV, and why does it matter for pricing a fund share?
2. What's a "gate" in the context of a private credit fund?
3. How long does traditional fund redemption take vs. Earlyout?
4. Why do private credit funds offer higher yields than a savings account?
5. What are the four terms in Earlyout's spread formula, and what risk does each represent?

---

[← Previous: Stablecoins, Circle and Arc](07-stablecoins-circle-and-arc.md) · [Next: RWA and tokenization →](09-rwa-and-tokenization.md)
