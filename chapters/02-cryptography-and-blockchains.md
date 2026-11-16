# Chapter 2: Cryptography and Blockchains

> **Goal:** After this chapter you can explain hashes, digital signatures, blocks, consensus mechanisms, and finality.

---

## 🧒 Like you're 12

### Hashing: the fingerprint machine

Imagine a machine where you put in any document and it spits out a unique fingerprint — a short string of letters and numbers. Even changing one comma gives a completely different fingerprint. And you can never reverse-engineer the document from the fingerprint.

That's a **hash function**.

### Signatures: proving it's you

You have a secret stamp (private key) that only you have. When you stamp a letter, anyone can hold it up to the light and see your watermark (public key) — but nobody can copy your stamp.

That's a **digital signature**.

### Blocks: pages in the notebook

Every 12 seconds (on Ethereum), all the new transactions get bundled into a **block**. Each block includes the fingerprint of the previous block, so they form a chain. If you try to change an old block, its fingerprint changes, which breaks every block after it.

---

## 🛠️ Engineer view

### Hash functions

A cryptographic hash function takes arbitrary input and produces a fixed-size output:

```
SHA-256("Hello") → 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
SHA-256("hello") → 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9825  (completely different)
```

Properties that matter:
- **Deterministic:** same input → same output, always
- **One-way:** output → input is computationally infeasible
- **Collision-resistant:** finding two inputs with the same output is impractical
- **Avalanche effect:** tiny change in input → completely different output

Ethereum uses **Keccak-256** (a SHA-3 variant). Bitcoin uses **SHA-256** (doubled).

### Digital signatures (ECDSA)

```
                    ┌─────────────┐
  message ─────────►│   Sign()    │──────► signature
  private key ─────►│             │
                    └─────────────┘

                    ┌─────────────┐
  message ─────────►│  Verify()   │──────► true/false
  signature ───────►│             │
  public key ──────►│             │
                    └─────────────┘
```

- **Private key:** a 256-bit random number. This IS your identity on the blockchain.
- **Public key:** derived from the private key (elliptic curve multiplication, one-way).
- **Address:** the last 20 bytes of the Keccak-256 hash of the public key (on Ethereum).

If you know some cryptography: Ethereum uses **secp256k1** (same curve as Bitcoin). The signing algorithm is ECDSA, though newer accounts can use other schemes.

### Block structure

```
Block N                              Block N+1
┌──────────────────────┐             ┌──────────────────────┐
│ Header               │             │ Header               │
│  ├ parentHash ───────────────────────► hash of Block N    │
│  ├ stateRoot         │             │  ├ stateRoot         │
│  ├ timestamp         │             │  ├ timestamp         │
│  └ ...               │             │  └ ...               │
│                      │             │                      │
│ Transactions         │             │ Transactions         │
│  ├ tx0: Alice→Bob    │             │  ├ tx0: Carol→Dave   │
│  ├ tx1: Eve→Frank    │             │  └ tx1: ...          │
│  └ ...               │             │                      │
│                      │             │                      │
│ hash: 0xabc123...    │             │ hash: 0xdef456...    │
└──────────────────────┘             └──────────────────────┘
```

**Why it's tamper-proof:** changing any transaction in Block N changes its hash, which breaks the `parentHash` in Block N+1, which changes *its* hash, breaking N+2, and so on. To rewrite history, you'd need to redo every block from the change forward — faster than everyone else.

### Consensus mechanisms

How thousands of computers agree on which block comes next:

| Mechanism | How it works | Used by | Trade-offs |
|-----------|-------------|---------|------------|
| **Proof of Work (PoW)** | Miners race to find a hash below a target. First one wins. Costs electricity. | Bitcoin | Very secure, very wasteful |
| **Proof of Stake (PoS)** | Validators lock up tokens as collateral. Random selection weighted by stake. | Ethereum (since 2022) | Energy efficient, stake can be slashed |
| **Delegated PoS / Authority** | A known set of validators take turns. | Arc, BNB Chain | Fast + cheap, more centralised |

### Finality

**Finality** = the guarantee that a transaction will never be reversed.

| Chain | Finality type | Time |
|-------|--------------|------|
| Bitcoin | Probabilistic (6 confirmations) | ~60 minutes |
| Ethereum | Probabilistic → deterministic (2 epochs) | ~13 minutes |
| Arc | Deterministic (single block) | ~2 seconds |

**Why finality matters for real money:** if you sell a fund claim for $100,000 in USDC, you need to know the payment is *final*, not "probably final after a few minutes". This is why Earlyout builds on Arc.

### Merkle trees (bonus)

Transactions within a block are organised in a **Merkle tree** — a binary tree of hashes. This lets you prove a transaction is in a block by providing a short "proof path" (log₂(n) hashes) instead of downloading the whole block.

```
        Root Hash
       /         \
    Hash01      Hash23
    /   \       /   \
  H(tx0) H(tx1) H(tx2) H(tx3)
```

---

## 🔗 In Earlyout

- Every transaction on Arc is signed by the user's **private key** via their wallet (MetaMask).
- The **NavOracle** contract rejects reports with an `updatedAt` older than the stored value — using timestamps (not hashes) to prevent replay, but the same "only accept newer" principle.
- Arc's **deterministic finality** means the USDC payment in a `sell()` transaction is final the moment it's in a block. The app can show "Done" immediately, not "Pending 6 confirmations".

---

## ✅ Check yourself

1. If you change one byte of the input to a hash function, what happens to the output?
2. Can you derive a private key from a public key? Why or why not?
3. What's the difference between probabilistic and deterministic finality?
4. Why does Earlyout care about fast finality?
5. What's a Merkle tree, and why is it useful?

---

[← Previous: Why Web3 exists](01-why-web3-exists.md) · [Next: Wallets, accounts and transactions →](03-wallets-accounts-and-transactions.md)
