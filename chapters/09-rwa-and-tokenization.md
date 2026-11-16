# Chapter 9: RWA and Tokenization

> **Goal:** After this chapter you can explain how a fund becomes a token, who's involved, what it fixes, and what it doesn't.

---

## 🧒 Like you're 12

Imagine you own a piece of a building (through a fund). Right now, that ownership is a line in a spreadsheet at a fund company. If you want to sell it, you call the company, fill out forms, wait days or weeks.

**Tokenization** is like turning that spreadsheet row into a card in your wallet — a digital token on a blockchain. You can still only sell it to approved people, but the transfer happens in seconds instead of days, and the record is on a shared ledger that everyone can verify.

---

## 🛠️ Engineer view

### What is RWA tokenization?

**RWA** = Real-World Asset. Tokenization is the process of representing ownership of a real-world asset as a token on a blockchain.

```
Physical / Legal world                 Blockchain world
━━━━━━━━━━━━━━━━━━━━━                 ━━━━━━━━━━━━━━━━━
Fund share in a cap table              ERC-20 token
Real estate deed                       NFT (ERC-721)
Bond certificate                       ERC-20 or ERC-1155
Invoice / receivable                   ERC-721
Carbon credit                          ERC-20
```

### Who's involved

| Role | What they do | Example |
|------|-------------|---------|
| **Asset originator** | Creates or owns the underlying asset | Apollo (private credit fund) |
| **Tokenization platform** | Issues the token, handles compliance | Securitize, Ondo, Hashnote |
| **Transfer agent** | Maintains the register of ownership | Securitize |
| **Custodian** | Holds the underlying asset safely | Bank of New York Mellon |
| **Oracle provider** | Brings off-chain data (NAV, status) on-chain | Chainlink |
| **Secondary market** | Where tokens can be traded | Earlyout (for redemption claims) |

### The token lifecycle

```
1. Fund exists off-chain (Apollo manages $600B in credit assets)
        ↓
2. Tokenization platform creates an ERC-20 representing fund shares
   (Securitize issues ACRED tokens on Ethereum)
        ↓
3. Only KYC/AML-approved addresses can hold the token
   (allowlist enforced in the smart contract)
        ↓
4. NAV is published daily via Chainlink price feeds
   (ACRED feed: 0x35DD...07EB on Ethereum, 8 decimals)
        ↓
5. Redemption follows the fund's rules
   (quarterly window, 5% minimum, days to settle)
```

### What tokenization fixes

| Problem | How tokens help |
|---------|----------------|
| **Settlement speed** | T+2 days → same transaction |
| **Market hours** | 9–4 weekdays → 24/7/365 |
| **Minimum investment** | $100K → potentially $100 (fractionalisation) |
| **Transparency** | Monthly PDF reports → real-time on-chain data |
| **Composability** | Can't use fund shares in DeFi → can (deposit as collateral, etc.) |
| **Geography** | Country-restricted → global (with compliance) |

### What tokenization does NOT fix

| It doesn't change | Why |
|-------------------|-----|
| **The fund's rules** | A quarterly fund still redeems quarterly. The token is a wrapper, not a workaround |
| **Credit risk** | If the borrowers default, the token's value drops |
| **Regulatory requirements** | KYC/AML still applies — the allowlist enforces it on-chain |
| **Liquidity** | A token doesn't magically create buyers. You need a market (like Earlyout) |
| **NAV accuracy** | The token reflects what the oracle reports. Garbage in, garbage out |

### Real RWA projects today

| Project | What it tokenizes | TVL/AUM |
|---------|------------------|---------|
| BlackRock BUIDL | US Treasury money market fund | $500M+ |
| Securitize / ACRED | Apollo credit fund | $1B+ |
| Ondo USDY | US Treasuries yield | $300M+ |
| Hashnote USYC | US Treasury bills | $200M+ |
| Centrifuge | Invoices, real estate loans | $200M+ |

### The allowlist pattern

Tokenized securities can't trade freely like USDC. Transfers are restricted:

```solidity
// From Earlyout's FundToken contract
function _update(address from, address to, uint256 value) internal override {
    if (from != address(0) && to != address(0)) {
        // Both sender and receiver must be on the allowlist
        if (!allowlisted[from]) revert NotAllowlisted(from);
        if (!allowlisted[to]) revert NotAllowlisted(to);
    }
    super._update(from, to, value);
}
```

The same pattern is on Earlyout's `ClaimToken` — claims can only move between approved addresses.

---

## 🔗 In Earlyout

- **FundToken** is a stand-in for a tokenized fund (like ACRED). Real ones use Securitize's contracts.
- The **ClaimWrapper** accepts fund tokens, files a redemption with the issuer, and mints a claim.
- The claim is tokenized as an ERC-1155 — it represents the *right to receive USDC at the payout date*.
- The **Vault** buys these claims at a discount (NAV minus spread) and collects the full amount at settlement.
- **What Earlyout doesn't change:** the fund still redeems quarterly. What it changes: the investor doesn't have to *wait* for that quarter.

---

## ✅ Check yourself

1. What does "tokenization" mean in one sentence?
2. Name three things tokenization improves and two things it doesn't fix.
3. What is an allowlist, and why do tokenized securities need one?
4. What role does a transfer agent play?
5. How is Earlyout's claim different from the fund token itself?

---

[← Previous: Traditional finance primer](08-traditional-finance-primer.md) · [Next: Liquidity, markets and pricing →](10-liquidity-markets-and-pricing.md)
