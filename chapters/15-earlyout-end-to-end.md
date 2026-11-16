# Chapter 15: Earlyout End to End

> **Goal:** After this chapter you can explain every contract in Earlyout, the full sale lifecycle, the business model, and the roadmap.

---

## 🧒 Like you're 12

You now know all the building blocks. This chapter puts them all together: how Earlyout works from the moment an investor wants out of a fund to the moment everyone gets paid.

---

## 🛠️ Engineer view

### The problem, one more time

```
Traditional fund redemption:
  Day 0:   Investor requests redemption
  Day 1-89: Waits (quarterly window)
  Day 90:  Fund processes the request
  Day 95:  Cash arrives in bank account

With Earlyout:
  Second 0: Investor calls depositAndSell()
  Second 2: USDC is in their wallet (same transaction, Arc finality)
```

### Contract map

```
┌─────────────────────────────────────────────────────────────┐
│                      Arc Testnet                            │
│                                                             │
│  FundRegistry ←─── Terms per fund (delay, vol, credit, cap)│
│       ↓                                                     │
│  NavOracle ←────── CRE reports (NAV, timestamp, status)    │
│       ↓                                                     │
│  IssuerStub ←───── Stand-in: queues redemptions, pays NAV  │
│       ↓                                                     │
│  ClaimWrapper ──→ ClaimToken (ERC-1155)                    │
│       │                 ↓                                   │
│       └──────────→ Vault (ERC-4626, USDC)                  │
│                     ├── quote() / quoteFund()               │
│                     ├── sell()                              │
│                     ├── buy()                               │
│                     ├── settle()                            │
│                     └── markImpaired()                      │
│                                                             │
│  USDC ──────────── 0x3600...0000 (native on Arc)           │
└─────────────────────────────────────────────────────────────┘
```

### Contract-by-contract breakdown

#### 1. FundRegistry
- **Job:** stores the list of funds and their pricing terms
- **Key fields per fund:** `settlementDelay`, `window`, `maxAge`, `volBps`, `creditBps`, `recoveryBps`, `exposureCapUsdc`
- **Access:** only the owner can list/update funds
- **Limit:** max 10 funds (bounded loop for gas)

#### 2. NavOracle
- **Job:** receives NAV and issuer status from the CRE workflow (or the ops wallet)
- **Key function:** `onReport(metadata, report)` — only the forwarder can call it
- **Safety:** rejects stale reports, rejects future timestamps (>5 min), validates workflow metadata
- **Output:** `getNav(fund)` returns `(nav, updatedAt, status)`

#### 3. IssuerStub
- **Job:** simulates a fund's redemption desk
- **On request:** locks the fund tokens, calculates payout date from settlement delay
- **On settle:** pays NAV × amount in USDC (after the payout date)
- **Failure mode:** `setFailed(true)` blocks all settlements

#### 4. ClaimToken (ERC-1155)
- **Job:** represents redemption claims — one token ID per (fund, payout date)
- **ID generation:** `keccak256(abi.encode(fund, settleAt))`
- **Transfer restriction:** both sender and receiver must be on the allowlist
- **Minting:** only the ClaimWrapper can mint; burning is for redeeming settled claims

#### 5. ClaimWrapper
- **Job:** orchestrates the deposit-to-claim flow
- **`deposit(fund, amount)`:** takes fund tokens → files redemption → mints claim
- **`depositAndSell(fund, amount, minPayout)`:** deposit + sell to Vault in one transaction
- **`collect(claimId)`:** after the payout date, pulls USDC from the issuer
- **`redeem(claimId, amount)`:** burns claims, pays pro-rata USDC

#### 6. Vault (ERC-4626)
- **Job:** the market maker — buys claims at a discount, holds them, collects at settlement
- **For sellers:** `quote()` → `sell()` — take claims, pay USDC
- **For buyers:** `buyPrice()` → `buy()` — sell claims the vault holds to external buyers
- **For keepers:** `settle()` — after the payout date, collect from the issuer
- **For risk:** `markImpaired()` — write down a failed fund's claims
- **For LPs:** `deposit()` → `withdraw()` (ERC-4626 standard) — provide/withdraw USDC

### The full sale lifecycle

```
Step 1: SELLER calls depositAndSell(ACRED, 1e18, minPayout)
        │
        ├── ClaimWrapper takes 1.0 ACRED from the seller
        ├── ClaimWrapper calls IssuerStub.requestRedemption(ACRED, 1e18)
        │   └── IssuerStub records: "1.0 ACRED, payout in 90 days"
        ├── ClaimWrapper calls ClaimToken.mint(wrapper, ACRED, settleAt, 1e18)
        │   └── ClaimToken creates claim #XYZ with 1.0 units
        └── ClaimWrapper calls Vault.sell(#XYZ, 1e18, minPayout, seller)
            │
            ├── Vault calls NavOracle.getNav(ACRED) → $10.50, OK
            ├── Vault checks: not halted, not over cap, has USDC
            ├── Vault calculates spread: 50 + 12 + 38 + 25 = 125 bps
            ├── Vault calculates payout: $10.50 × 0.9875 = $10.37
            ├── Vault takes the claim from ClaimWrapper
            ├── Vault sends $10.37 USDC to the seller
            └── Vault emits ClaimSold(...)

Step 2: Backend writes the sale to Arkiv (earlyout.claims entity, expires at payout)

Step 3: 90 days later, KEEPER calls Vault.settle(#XYZ)
        │
        ├── Vault calls ClaimWrapper.collect(#XYZ)
        │   └── ClaimWrapper calls IssuerStub.settle(ACRED, settleAt)
        │       └── IssuerStub pays full NAV: $10.50 USDC
        ├── ClaimWrapper.redeem(#XYZ, 1e18) → burns claim, sends USDC to Vault
        ├── Vault's idle USDC increases by $10.50
        ├── Vault's deployed USDC decreases by $10.37 (the cost)
        └── Net gain to LP pool: $0.13 per unit (the spread)
```

### The business model

```
Revenue = Spread × Volume

The spread goes to LPs (and potentially to the protocol via a future fee split):
  Spread = baseBps + utilBps + timeBps + creditBps

Example for 1 ACRED at $10.50 NAV, 90-day payout:
  Spread = 50 + 12 + 38 + 25 = 125 bps = 1.25%
  Discount = $0.13 per unit
  LP earns: $0.13 on $10.37 deployed for ~90 days
  Annualised: ~5.1%
```

### Who uses Earlyout

| User | What they do | What they get |
|------|-------------|--------------|
| **Seller** (fund investor) | Sells a pending redemption | Cash now instead of waiting |
| **LP** (liquidity provider) | Deposits USDC into the Vault | Yield from spreads |
| **Buyer** | Buys a discounted claim from the Vault | The claim at a smaller discount than the seller got |
| **Keeper** | Calls `settle()` at the payout date | Gas refund (future) |
| **Distributor** | Manages the allowlist | Earlyout adds liquidity to their fund |

### Architecture diagram

```
Ethereum mainnet                    Issuer status API
  │ (Chainlink NAV feeds)              │
  └──────────┐          ┌──────────────┘
             ↓          ↓
         Chainlink CRE workflow
             │
             ↓ signed report
Arc testnet: NavOracle
             │
  FundRegistry ──→ Vault (ERC-4626, USDC) ←── LPs
                     ↑         │
  ClaimWrapper ──→ ClaimToken  │
       ↑                       ↓
  Seller (MetaMask)     Earlyout Backend (Next.js, Vercel)
                              │              │
                              ↓              ↓
                         Arkiv (Book)   Web App (Sell, Vault, Book screens)
```

---

## ✅ Check yourself

1. Walk through the full lifecycle of a sale, from deposit to settlement.
2. Where does LP yield come from? Is it sustainable?
3. What happens if the issuer fails between sale and settlement?
4. Why is `depositAndSell` better than `deposit` + `sell` as separate transactions?
5. What does the keeper do, and why is it needed?

---

[← Previous: Developer stack and hands-on](14-developer-stack-and-hands-on.md) · [Next: Glossary, quiz answers and next steps →](16-glossary-quiz-and-next-steps.md)
