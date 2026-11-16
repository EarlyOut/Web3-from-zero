# Chapter 6: Chains, Scaling and Bridges

> **Goal:** After this chapter you can tell an L1 from an L2, explain rollups, bridges, and CCTP.

---

## 🧒 Like you're 12

Imagine a highway (Ethereum) that's really secure but gets traffic jams. **L2s** are express lanes built above the highway — they handle the traffic faster and cheaper, but periodically post a summary back to the main highway for security.

**Bridges** are like ferry boats between islands (different blockchains). You lock your money on one island, and a copy appears on the other. CCTP (Circle's bridge) is a special ferry just for USDC — it burns USDC on one chain and mints fresh USDC on the other, so there's no "copy" problem.

---

## 🛠️ Engineer view

### L1 vs L2

| | L1 (Layer 1) | L2 (Layer 2) |
|---|---|---|
| **What** | The base blockchain | A chain built on top of an L1 |
| **Security** | Its own validators/miners | Inherits security from the L1 |
| **Speed** | Slower (all nodes process everything) | Faster (fewer nodes, batch to L1) |
| **Cost** | Higher gas | Much lower gas |
| **Examples** | Ethereum, Bitcoin, Solana | Arbitrum, Optimism, Base, zkSync |

### Rollups: the dominant L2 design

Rollups execute transactions off the main chain but post the data (or a proof) back to L1:

**Optimistic Rollups** (Arbitrum, Optimism, Base):
```
1. Execute transactions on L2
2. Post transaction data to L1
3. Assume everything is correct (optimistic)
4. 7-day challenge window: anyone can submit a fraud proof
5. If fraud is proven, the batch is rolled back
```

**ZK Rollups** (zkSync, StarkNet, Scroll):
```
1. Execute transactions on L2
2. Generate a zero-knowledge proof (mathematical guarantee)
3. Post the proof + compressed data to L1
4. L1 verifies the proof (instant finality on L1)
```

| | Optimistic | ZK |
|---|---|---|
| Proof type | Fraud proof (reactive) | Validity proof (proactive) |
| Withdrawal to L1 | ~7 days | Minutes |
| Computation cost | Cheap on L2 | Expensive to generate proof |
| EVM compatibility | Very high | Getting better |

### Sidechains and app-chains

Not all chains are L2s:

| Type | Relationship to Ethereum | Example |
|------|-------------------------|---------|
| **Sidechain** | Independent chain, bridged | Polygon PoS |
| **App-chain** | Purpose-built for one use case | Arc (stablecoin finance) |
| **Alt-L1** | Completely independent | Solana, Avalanche |

**Arc** (Circle's chain) is an **app-chain for stablecoin finance**. USDC is the native gas token — you don't need ETH. Deterministic finality in ~2 seconds. It's not an L2 (it doesn't post proofs to Ethereum), but it's EVM-compatible.

### Bridges

Moving assets between chains requires a bridge. The core problem: a token on Chain A doesn't natively exist on Chain B.

**Lock-and-mint bridges:**
```
Chain A:  Lock 100 USDC in bridge contract
Bridge:   Observes the lock
Chain B:  Mint 100 "wrapped USDC" on Chain B
```

Risk: if the bridge is hacked, the locked assets are stolen but the wrapped tokens still exist → they become worthless.

**Notable bridge hacks:**
- Ronin Bridge (2022): $625M stolen
- Wormhole (2022): $320M stolen
- Nomad (2022): $190M stolen

### CCTP: Circle Cross-Chain Transfer Protocol

CCTP solves the bridge problem for USDC by using **burn-and-mint** instead of lock-and-mint:

```
Chain A:  Burn 100 USDC (destroyed)
Circle:   Attestation service confirms the burn
Chain B:  Mint 100 fresh native USDC (newly created by Circle)
```

**Why this is better:** there's no pool of locked USDC that can be stolen. The USDC on Chain B is real, native USDC — not a wrapped copy. Circle is the issuer on both chains.

This matters for Earlyout because a fund investor whose USDC lives on Ethereum can use CCTP to move it to Arc without trusting a third-party bridge.

### Chain IDs

Every EVM chain has a unique chain ID to prevent replay attacks (a transaction signed for one chain shouldn't work on another):

| Chain | Chain ID |
|-------|---------|
| Ethereum mainnet | 1 |
| Arbitrum One | 42161 |
| Base | 8453 |
| Polygon | 137 |
| Arc testnet | 5042002 |

---

## 🔗 In Earlyout

- **Earlyout runs on Arc** (chain ID 5042002) because USDC is the native gas token and finality is deterministic.
- The **Chainlink CRE workflow** reads NAV feeds from **Ethereum mainnet** and writes them to **Arc** — a cross-chain data bridge.
- Sellers whose USDC is on another chain would use **CCTP** to move it to Arc.
- The contracts are EVM-compatible and chain-agnostic — the same deploy script works on any EVM chain, just with a different RPC URL.

---

## ✅ Check yourself

1. What's the key difference between an optimistic rollup and a ZK rollup?
2. Why are bridges the most attacked part of the crypto ecosystem?
3. How does CCTP avoid the "locked pool" risk of traditional bridges?
4. Why does Arc use USDC as the gas token instead of a native token?
5. What's a chain ID for, and why does it matter?

---

[← Previous: Tokens and standards](05-tokens-and-standards.md) · [Next: Stablecoins, Circle and Arc →](07-stablecoins-circle-and-arc.md)
