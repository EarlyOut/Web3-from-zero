# Chapter 7: Stablecoins, Circle and Arc

> **Goal:** After this chapter you can answer "What does Circle do?", compare USDC and USDT, and explain Arc.

---

## 🧒 Like you're 12

Crypto prices go up and down wildly. Imagine trying to price a lemonade at "0.0003 Bitcoin" when Bitcoin might be worth $30,000 today and $25,000 tomorrow. That's why **stablecoins** exist — they're tokens designed to always be worth $1.

**USDC** is like a gift card backed by real dollars in a bank. Circle (the company) holds $1 of cash and US Treasury bonds for every 1 USDC in circulation. You can always trade them back.

---

## 🛠️ Engineer view

### What is a stablecoin?

A token whose value is pegged to a reference asset (usually $1 USD).

| Type | How it works | Examples | Risk |
|------|-------------|----------|------|
| **Fiat-backed** | 1:1 reserves in bank accounts | USDC, USDT | Bank risk, regulatory risk |
| **Over-collateralised** | Locked crypto worth more than the stablecoin | DAI, LUSD | Liquidation if collateral drops |
| **Algorithmic** | Supply/demand algorithms maintain the peg | (UST, failed) | Death spiral risk — UST lost $40B |

### USDC vs USDT

| | USDC (Circle) | USDT (Tether) |
|---|---|---|
| **Issuer** | Circle (US-based, regulated) | Tether (BVI-based) |
| **Market cap** | ~$55B | ~$120B |
| **Reserves** | Monthly third-party attestations. Cash + short-term US Treasuries | Quarterly attestations. Mix of assets including commercial paper (historically) |
| **Transparency** | Published reserve composition | Less detailed |
| **Regulatory stance** | Embraces US regulation | Operates offshore |
| **Chains** | 15+ chains natively | 10+ chains |
| **ERC-20?** | Yes, with blocklist and admin functions | Yes, with similar controls |

**Both can freeze accounts.** USDC and USDT contracts have an admin function that can blocklist addresses (used for law enforcement compliance). This is a feature for institutions and a concern for cypherpunks.

### Circle's products

| Product | What it does |
|---------|-------------|
| **USDC** | The stablecoin — a dollar on-chain |
| **CCTP** | Cross-chain transfer protocol — burn on one chain, mint on another |
| **Circle Wallets** | Developer-controlled wallets via API (programmatic key management) |
| **Arc** | A blockchain purpose-built for stablecoin finance |
| **Arc Studio** | An IDE for building on Arc (deployed Earlyout's contracts) |

### Arc: Circle's blockchain

Arc is a chain built specifically for stablecoin-based finance:

| Feature | Why it matters |
|---------|---------------|
| **USDC is the gas token** | No ETH needed. Investors only deal in dollars. |
| **Deterministic finality** | A transaction in a block is *final*. No "wait for confirmations". |
| **EVM-compatible** | Standard Solidity, standard tooling (Foundry, Hardhat, viem) |
| **Known validator set** | More centralised than Ethereum, but institutional-grade uptime |
| **Native USDC** | Not bridged — Circle mints directly on Arc |

```
On Ethereum:   Gas is ETH.  USDC is an ERC-20 contract.
On Arc:        Gas is USDC. USDC is the native currency (precompiled at 0x3600...0000).
```

**The USDC address on Arc:** `0x3600000000000000000000000000000000000000` (6 decimals for balances, but native gas accounting uses 18 decimals — never use the native balance for financial accounting).

### Why institutions care about USDC on Arc

1. **Same-currency settlement.** A fund investor sells a claim and gets USDC. The gas fee is in USDC. The LP earns yield in USDC. No currency conversion, no volatile gas token.
2. **Compliance built in.** Circle can comply with sanctions and law enforcement. The validator set is known. This makes regulators more comfortable than a permissionless chain.
3. **Interoperability via CCTP.** USDC on Arc is the same USDC as on Ethereum, Arbitrum, or Base — movable via CCTP without a third-party bridge.

---

## 🔗 In Earlyout

- **Every balance** in Earlyout is USDC on Arc.
- The Vault is an ERC-4626 vault where the **asset is USDC**.
- The **gas** for every transaction is paid in USDC — a fund investor doesn't need to acquire a separate token.
- **Arc Studio** tested, security-reviewed, and deployed all Earlyout contracts from a Circle developer-controlled wallet.
- The deployer wallet was then transferred to the team's ops wallet — the Circle wallet owns nothing post-deployment.

---

## ✅ Check yourself

1. Why do stablecoins exist? What problem do they solve?
2. What's the main difference between USDC's reserves and USDT's reserves?
3. Can Circle freeze a specific address's USDC? Why or why not?
4. What makes Arc different from Ethereum for financial applications?
5. Why does Earlyout use USDC as the settlement currency instead of ETH or DAI?

---

[← Previous: Chains, scaling and bridges](06-chains-scaling-and-bridges.md) · [Next: Traditional finance primer →](08-traditional-finance-primer.md)
