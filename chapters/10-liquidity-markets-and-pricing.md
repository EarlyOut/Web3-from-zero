# Chapter 10: Liquidity, Markets and Pricing

> **Goal:** After this chapter you can explain liquidity, spreads, order books vs AMMs, and Earlyout's pricing maths.

---

## 🧒 Like you're 12

**Liquidity** is how easy it is to sell something for a fair price, right now.

Your house is **illiquid** — it takes months to sell. A dollar bill is perfectly liquid. A tokenized fund share is somewhere in between — the token moves instantly, but someone has to want to buy it.

The **spread** is the gap between what a buyer will pay and what a seller wants. The bigger the spread, the more it costs you to trade. Earlyout's spread tells you exactly *why* it costs what it costs — four reasons, each visible.

---

## 🛠️ Engineer view

### Liquidity

```
High liquidity:  USDC, US Treasury bonds
                 → Many buyers and sellers, tight spreads, fast execution

Low liquidity:   Tokenized private credit claim
                 → Few buyers, wide spreads, slow to exit
```

Earlyout creates liquidity for something that has almost none: a pending fund redemption.

### Order books vs AMMs

**Order book** (traditional):
```
Sellers (asks):   $10.05 × 100 units
                  $10.03 × 50 units
                  ────────────────────
                  $10.01 × 200 units    ← best ask
                  $10.00 × 150 units    ← best bid
                  ────────────────────
Buyers (bids):    $9.98 × 100 units
                  $9.95 × 300 units

Spread = $10.01 - $10.00 = $0.01 (1 cent)
```

**AMM** (Automated Market Maker, like Uniswap):
```
Pool: 1,000 ETH + 2,000,000 USDC
Invariant: x × y = k
Price = y/x = $2,000 per ETH
When you buy ETH, x decreases, y increases → price goes up
```

**Earlyout** uses neither. It's a **direct buyer** — the Vault itself buys your claim at a calculated price. No matching engine, no liquidity pool. The price is deterministic: NAV minus a spread that's computed on-chain.

### Earlyout's spread formula

```
spread = baseBps + utilBps + timeBps + creditBps
price  = NAV × (1 - spread/10000)
```

Each term in basis points (1 bps = 0.01%):

| Term | What it represents | How it's calculated |
|------|-------------------|-------------------|
| **baseBps** | Minimum fee for running the protocol | Fixed (global setting, e.g. 50 bps = 0.5%) |
| **utilBps** | Vault is running low on USDC | `utilSlopeBps × deployedUsdc / totalAssets` — rises as the vault's USDC is deployed |
| **timeBps** | Longer wait = more risk | `volBps × √(seconds to payout / seconds in a year)` — scales with square root of time |
| **creditBps** | Riskier issuer = bigger haircut | Fixed per fund (e.g. 25 bps for ACRED) |

**The square root of time** in `timeBps` comes from financial option pricing (Black-Scholes model): uncertainty grows with the square root of time, not linearly.

### Reading the price in Solidity

```solidity
// From Vault._quote()
uint256 secs = settleAt > block.timestamp ? settleAt - block.timestamp : 0;
q.timeBps = _toBps(uint256(t.volBps) * Math.sqrt(secs * 1e18 / YEAR) / 1e9);

uint256 spread = uint256(q.baseBps) + q.utilBps + q.timeBps + q.creditBps;
uint256 value = amount * nav / 1e30;
q.payout = spread >= BPS ? 0 : value * (BPS - spread) / BPS;
```

**Example:** selling 1.0 ACRED at NAV = $10.50, 85 days to payout:
- baseBps = 50 (0.50%)
- utilBps = 12 (0.12%, vault is 12% utilised)
- timeBps = 38 (0.38%, 85 days × volatility)
- creditBps = 25 (0.25%)
- **Total spread = 125 bps = 1.25%**
- **Payout = $10.50 × (1 - 0.0125) = $10.37**

The seller gets $10.37 today instead of waiting 85 days for $10.50.

### Why the spread self-corrects

- **More sellers** → utilBps rises (vault deploys more USDC) → spread widens → fewer sellers
- **Payout date approaches** → timeBps drops → spread narrows → more sellers
- **LP deposits more USDC** → utilBps drops → spread narrows
- **Issuer issues a warning** → status flips to FAILED → trading HALTs entirely

The system finds its own equilibrium without an order book or matching engine.

---

## 🔗 In Earlyout

- The Vault is not an exchange — it's a **unilateral buyer**. It always takes the other side of the trade.
- LPs provide the USDC liquidity. Their yield comes from the spread: buy at NAV minus spread, collect full NAV at settlement.
- The `quote()` function returns all four spread terms so the UI can show them before the user signs.
- The `quoteFund()` function lets the UI show the price *before* the claim even exists — the user sees the verdict and breakdown before depositing.

---

## ✅ Check yourself

1. What's the difference between an order book and an AMM?
2. Why does Earlyout use a direct-buyer model instead of an order book?
3. If `baseBps = 50`, `utilBps = 20`, `timeBps = 30`, `creditBps = 10`, and NAV = $10.00, what's the payout per unit?
4. Why does the time term use the square root of time?
5. What happens to the spread as the vault deploys more of its USDC?

---

[← Previous: RWA and tokenization](09-rwa-and-tokenization.md) · [Next: Oracles, data and indexing →](11-oracles-data-and-indexing.md)
