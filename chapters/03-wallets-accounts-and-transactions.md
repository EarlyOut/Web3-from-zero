# Chapter 3: Wallets, Accounts and Transactions

> **Goal:** After this chapter you can explain keys, wallet types, the lifecycle of a transaction, gas, and MEV.

---

## 🧒 Like you're 12

Your **wallet** is like a keychain. The keys on it open lockboxes (accounts) on the blockchain. You don't actually store money *in* the wallet — the money is recorded on the blockchain. The wallet just proves it's yours.

**Gas** is like postage. Every time you send a transaction, you pay a small fee to the network for processing it. More complex transactions (like running a contract) cost more gas, just like heavier packages cost more to ship.

---

## 🛠️ Engineer view

### Account types (Ethereum model)

| | Externally Owned Account (EOA) | Contract Account |
|---|---|---|
| **Controlled by** | A private key (held by a human or hardware) | Its code (deployed once, runs forever) |
| **Has code?** | No | Yes |
| **Can initiate transactions?** | Yes | No (only responds to calls) |
| **Address** | Derived from public key | Derived from deployer address + nonce |
| **Example** | Your MetaMask wallet | The Earlyout Vault contract |

### Key hierarchy

```
Seed phrase (12 or 24 words)
    ↓ BIP-39
Master private key
    ↓ BIP-44 derivation path: m/44'/60'/0'/0/0
Private key (256-bit number)
    ↓ Elliptic curve multiplication
Public key
    ↓ Keccak-256, take last 20 bytes
Address (0x...)
```

> **Never share your seed phrase or private key.** Anyone who has it controls every account derived from it.

### Wallet types

| Type | What it is | Examples | Security |
|------|-----------|----------|----------|
| **Hot wallet** | Software wallet, keys in browser/phone | MetaMask, Rainbow | Convenient, vulnerable to malware |
| **Hardware wallet** | Keys stored on a physical device | Ledger, Trezor | Keys never leave the device |
| **Multi-sig** | Needs M-of-N signatures | Gnosis Safe | Great for teams/treasuries |
| **Smart wallet** | A contract that acts as a wallet | ERC-4337 accounts | Social recovery, gas sponsoring |
| **Developer-controlled** | A cloud-hosted key managed by an API | Circle Wallets | Good for server-side automation |

### The life of a transaction

```
1. Build           You specify: to, value, data, gas limit, max fee
        ↓
2. Sign            Your wallet signs with your private key
        ↓
3. Broadcast       Signed tx sent to a node's mempool
        ↓
4. Mempool         Transaction waits in the pending pool
        ↓
5. Inclusion       A validator picks it up and puts it in a block
        ↓
6. Execution       The EVM runs the transaction (state changes)
        ↓
7. Receipt         Success or revert, gas used, logs emitted
```

### Transaction fields

| Field | What it is | Example |
|-------|-----------|---------|
| `from` | Sender address (derived from signature) | 0xbc83...537B |
| `to` | Recipient address (or empty for deploy) | 0x7A40...2319 (Vault) |
| `value` | Native currency to send (ETH, or USDC on Arc) | 0 (we move USDC via contract calls) |
| `data` | Encoded function call (ABI) | `sell(claimId, amount, minPayout, receiver)` |
| `nonce` | Sender's transaction count (prevents replays) | 42 |
| `gasLimit` | Max gas units this tx may consume | 200000 |
| `maxFeePerGas` | Max price per gas unit | 30 gwei |
| `maxPriorityFeePerGas` | Tip to the validator | 2 gwei |

### Gas: the execution fee

**Gas** is how the network meters computation. Every EVM opcode has a gas cost:

| Operation | Approximate gas |
|-----------|----------------|
| Simple transfer | 21,000 |
| Storage write (new slot) | 20,000 |
| Storage write (existing) | 5,000 |
| Addition/subtraction | 3 |
| Keccak-256 hash | 30 + 6 per word |

**Cost = gas used × gas price**

On Ethereum mainnet, a complex DeFi transaction might cost $5–50 in gas. On Arc, it's fractions of a cent (paid in USDC).

### EIP-1559: the fee model

Since London (2021), Ethereum uses a two-part fee:
- **Base fee:** algorithmically set, burned (removed from circulation)
- **Priority fee (tip):** goes to the validator

```
You pay:  min(maxFeePerGas, baseFee + maxPriorityFeePerGas) × gasUsed
Burned:   baseFee × gasUsed
Validator: (your price - baseFee) × gasUsed
```

### MEV: Maximal Extractable Value

Validators can **reorder transactions** within a block. This creates profit opportunities:

| MEV type | What happens |
|----------|-------------|
| **Front-running** | Bot sees your trade, places the same trade first |
| **Sandwich attack** | Bot buys before you, sells after you, profiting from the price impact |
| **Back-running** | Bot places a trade right after yours to capture the arbitrage |
| **Liquidations** | Bot races to liquidate an undercollateralised position for the reward |

This is why Earlyout's `sell()` function takes a `minPayout` parameter — if the price moves against the seller (from MEV or anything else), the transaction simply reverts instead of executing at a bad price.

---

## 🔗 In Earlyout

- The **deployer** was a Circle developer-controlled wallet (server-side key management via Circle's API), which then transferred ownership to the team's ops wallet.
- When you sell a claim, the browser wallet (MetaMask) **signs** the `depositAndSell` transaction. The browser never sends the private key — only the signed transaction.
- **Gas on Arc** is paid in USDC. This means a fund investor doesn't need to acquire a separate gas token (like ETH) — they already have USDC.
- The `minPayout` guard in `Vault.sell()` protects against MEV: if someone front-runs the transaction and moves the price, it reverts.

---

## ✅ Check yourself

1. What's the difference between an EOA and a contract account?
2. Why does a transaction have a `nonce`?
3. If gas costs 21,000 units for a transfer and the gas price is 10 gwei, what's the cost in ETH?
4. What is MEV, and why should a user care?
5. What does `minPayout` protect against in `Vault.sell()`?

---

[← Previous: Cryptography and blockchains](02-cryptography-and-blockchains.md) · [Next: Smart contracts →](04-smart-contracts.md)
