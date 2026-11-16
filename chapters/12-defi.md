# Chapter 12: DeFi

> **Goal:** After this chapter you can explain lending, AMMs, vaults, yield, and composability.

---

## 🧒 Like you're 12

**DeFi** (Decentralised Finance) is doing everything a bank does — lending, borrowing, trading, earning interest — but with smart contracts instead of a bank. No one controls it, it's open 24/7, and anyone with a wallet can use it.

---

## 🛠️ Engineer view

### DeFi primitives

| Primitive | What it does | Examples | Traditional equivalent |
|-----------|-------------|----------|----------------------|
| **DEX** (Decentralised Exchange) | Trade tokens | Uniswap, Curve | Stock exchange |
| **Lending** | Deposit to earn interest; borrow against collateral | Aave, Compound | Bank savings + loans |
| **Stablecoins** | Dollar-pegged tokens | USDC, DAI | Cash |
| **Vaults / Yield aggregators** | Automatically invest deposits for yield | Yearn, ERC-4626 vaults | Money market fund |
| **Bridges** | Move assets between chains | CCTP, Wormhole | SWIFT / wire transfer |

### How lending works (Aave-style)

```
Supplier (Lender):
  1. Deposits 10,000 USDC into Aave
  2. Receives aUSDC (interest-bearing token)
  3. aUSDC balance grows every block

Borrower:
  1. Deposits 15,000 USDC of ETH as collateral
  2. Borrows 10,000 USDC
  3. Must maintain collateral > 150% of loan
  4. If ETH price drops below threshold → liquidated
```

Interest rates are set algorithmically based on utilisation:
```
Utilisation = Total borrowed / Total supplied
Higher utilisation → Higher interest rate → Encourages more deposits
```

### AMMs: automated market makers

Instead of an order book, AMMs use a **constant product formula**:

```
x × y = k

Where:
  x = amount of Token A in the pool
  y = amount of Token B in the pool
  k = constant (stays the same after each trade)
```

When you trade:
```
Pool: 100 ETH × 200,000 USDC = 20,000,000 (k)
You buy 1 ETH:
  New pool: 99 ETH × (20,000,000 / 99) USDC = 99 ETH × 202,020 USDC
  You paid: 202,020 - 200,000 = 2,020 USDC for 1 ETH
  Price impact: $2,020 instead of $2,000 (1% slippage)
```

Bigger trades → more slippage. This is why liquidity depth matters.

### Vaults and ERC-4626

A vault takes deposits, invests them according to a strategy, and returns the yield to depositors:

```
┌────────────────────────────────────────┐
│          ERC-4626 Vault                │
│                                        │
│  Depositors:                           │
│    Alice: 1000 shares                  │
│    Bob:   500 shares                   │
│                                        │
│  Total assets: 1530 USDC              │
│  Total shares: 1500 eoUSDC            │
│  Share price:  1.02 USDC              │
│                                        │
│  Strategy: buy redemption claims       │
│  at NAV - spread, collect at NAV       │
│                                        │
└────────────────────────────────────────┘
```

**Earlyout's Vault** is an ERC-4626 vault where:
- Asset = USDC
- Strategy = buy pending fund redemption claims at a discount
- Yield = the spread between buy price and settlement price
- Risk = issuer failure (mitigated by HALT + markImpaired)

### Composability: "money Legos"

The most powerful idea in DeFi: contracts can call other contracts, stacking on top of each other:

```
Layer 1: USDC (the asset)
Layer 2: Earlyout Vault (earns yield from claims)
Layer 3: Could use eoUSDC as collateral on Aave
Layer 4: Could borrow against it, deploy into another strategy
```

Each layer is a building block. You don't need permission from the layer below — it's open and permissionless.

### Yield sources in DeFi

| Source | Where yield comes from | Risk |
|--------|----------------------|------|
| **Lending** | Borrowers pay interest | Smart contract risk, bad debt |
| **LP fees** | Traders pay swap fees | Impermanent loss |
| **Staking** | Validators earn block rewards | Slashing, lockup |
| **Vault strategies** | Various (Earlyout: redemption spreads) | Strategy-specific |
| **Points/airdrops** | Protocol distributes tokens | Speculative, may not materialise |

### TVL: Total Value Locked

TVL is the standard metric for DeFi protocols — how much money is deposited. As of 2026:
- DeFi total: ~$100B+
- Aave: ~$15B
- Lido: ~$30B
- Uniswap pools: ~$5B

---

## 🔗 In Earlyout

- The Vault is a **DeFi primitive** — it could be composed with other protocols.
- LP yield comes from a real source: the spread on fund redemption claims. This isn't inflationary token rewards — it's economic value from time preference.
- The `totalAssets()` function implements the accounting: idle USDC + claims at cost (written down if impaired).
- `maxWithdraw` is capped at `idleUsdc()` — LPs can't withdraw USDC that's tied up in claims. This protects against bank runs.

---

## ✅ Check yourself

1. What's the difference between a DEX and a traditional exchange?
2. How does a constant-product AMM determine the price?
3. Where does yield come from in Earlyout's vault?
4. What is composability, and why does it matter?
5. Why does `maxWithdraw` return `min(normal amount, idleUsdc())`?

---

[← Previous: Oracles, data and indexing](11-oracles-data-and-indexing.md) · [Next: Security, risk and regulation →](13-security-risk-and-regulation.md)
