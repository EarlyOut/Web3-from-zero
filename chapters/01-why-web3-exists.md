# Chapter 1: Why Web3 Exists

> **Goal:** After this chapter you can explain what problem blockchains solve, the history from Bitcoin to today, and why anyone bothers.

---

## 🧒 Like you're 12

Imagine you and your friends keep score in a shared notebook. Everyone has a copy. When someone scores a point, everyone writes it down at the same time. Nobody can cheat because everyone would notice.

That's what a blockchain is — a shared scoreboard that nobody owns and nobody can secretly change.

**Why does it matter?** Before this, keeping score online always needed a referee: a bank, PayPal, or some company in the middle. The referee charges fees, can freeze your account, and might go out of business. The blockchain replaces the referee with maths.

---

## 🛠️ Engineer view

### The problem: trusted third parties

Every digital transaction today goes through an intermediary:
- Send money → your bank talks to their bank (via SWIFT, ACH, etc.)
- Buy a stock → a broker talks to an exchange talks to a clearinghouse
- Send a message → a platform stores and routes it

Each intermediary is a **single point of failure**, a **permission gate**, and a **fee collector**.

### Bitcoin's answer (2008)

Satoshi Nakamoto's whitepaper solved the **double-spend problem** without a central authority:

```
Traditional:   Alice → Bank (checks balance) → Bob
Bitcoin:       Alice → signs tx with private key → broadcast to network
               → miners validate → added to block → everyone agrees
```

**Key insight:** replace trust in an institution with trust in cryptography and economic incentives.

### The evolution

| Year | What happened | Why it matters |
|------|--------------|----------------|
| 2008 | Bitcoin whitepaper | Proved digital scarcity without a central party |
| 2009 | Bitcoin launches | First real peer-to-peer money |
| 2013 | Ethereum proposed | "What if the blockchain could run programs?" |
| 2015 | Ethereum launches | Smart contracts: code that runs on the blockchain |
| 2017 | ICO boom (and bust) | Showed tokens can represent anything, also showed the risks |
| 2018 | Stablecoins grow (USDC, DAI) | Tokens pegged to $1 — practical for commerce |
| 2020 | DeFi Summer | Lending, trading, yield — all without banks |
| 2021 | NFTs go mainstream | Digital ownership, beyond just money |
| 2022–23 | Real-world assets (RWAs) | Bonds, funds, and real estate move on-chain |
| 2024–26 | Institutional adoption, L2s, Arc | Big finance tokenises funds; purpose-built chains like Arc launch |

### Web1 → Web2 → Web3

| | Web1 (Read) | Web2 (Read + Write) | Web3 (Read + Write + Own) |
|---|---|---|---|
| **Era** | 1990s–2000s | 2000s–now | 2015–now |
| **Content** | Static pages | User-generated | User-owned |
| **Identity** | Email | Google/Facebook login | Wallet (private key) |
| **Data** | On servers you read | On platforms that own it | On a chain everyone can verify |
| **Money** | Credit cards via banks | PayPal, Stripe | Tokens, stablecoins, DeFi |
| **Trust model** | "Trust the webmaster" | "Trust the platform" | "Verify, don't trust" |

### Why should a software engineer care?

1. **New execution model.** Code on a blockchain runs deterministically on thousands of machines. Once deployed, nobody — not even the developer — can change it (without a migration).
2. **Built-in payments.** Every user has a wallet. No Stripe integration, no bank accounts, no chargebacks.
3. **Composability.** Smart contracts can call other smart contracts. It's like every API in the world being open-source, permissionless, and running 24/7.
4. **Global and censorship-resistant.** There's no kill switch, no region lock, no Terms of Service update that shuts you out.

---

## 🔗 In Earlyout

Earlyout exists because of a specific Web3 problem:

> Tokenized private credit funds (like Apollo's ACRED on Securitize) let investors out once a quarter. If you need cash before that window, you're stuck.

Earlyout builds on the blockchain (Arc, Circle's chain) to let that investor **sell their pending redemption today**, without asking anyone's permission. The blockchain guarantees the trade settles in one transaction.

This only works because:
- The fund share is a **token** on a blockchain (Chapter 5)
- The price comes from a **Chainlink oracle** (Chapter 11)
- Payment is in **USDC**, a stablecoin (Chapter 7)
- The logic runs in a **smart contract** (Chapter 4)

---

## ✅ Check yourself

1. What problem did Bitcoin solve that no one had solved before?
2. What does "trustless" actually mean? (Hint: it doesn't mean "you can't trust it")
3. Name two things an intermediary does that a blockchain replaces.
4. What's the difference between Web2 identity and Web3 identity?
5. Why can't the developer of a smart contract just change it after deploying?

---

[← Back to course map](../README.md) · [Next: Cryptography and blockchains →](02-cryptography-and-blockchains.md)
