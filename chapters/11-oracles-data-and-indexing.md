# Chapter 11: Oracles, Data and Indexing

> **Goal:** After this chapter you can explain Chainlink, CRE, events, RPCs, indexers, and Arkiv.

---

## 🧒 Like you're 12

Smart contracts live on the blockchain, but they're **blind** — they can't see the outside world. They don't know the price of anything, the weather, or whether a company is still in business. They can only see what's on-chain.

An **oracle** is a messenger that brings outside information *into* the blockchain. Chainlink is the most popular oracle network — think of it as a trusted news agency that smart contracts subscribe to.

An **indexer** does the opposite: it watches what happens on the blockchain and organises it into a searchable database that apps can query easily.

---

## 🛠️ Engineer view

### The oracle problem

Smart contracts are deterministic: given the same state and input, every node must produce the same output. This means a contract can't just call an API — different nodes might get different responses at different times.

```
❌ Contract calls api.price.com/gold → Node A gets $1,950, Node B gets $1,951
   → Nodes disagree → Consensus breaks

✅ Oracle network reads the price, reaches consensus, posts it on-chain
   → Every node reads the same on-chain value → Deterministic
```

### Chainlink: the standard oracle network

Chainlink has two main products:

**1. Data Feeds (Price Feeds)**
Pre-built, continuously updated price feeds. Any contract can read them:

```solidity
// Reading ACRED NAV from Chainlink (Ethereum mainnet)
(, int256 answer, , uint256 updatedAt, ) = AggregatorV3Interface(
    0x35DDfB90011E686CCf837a6819562705076207EB  // ACRED feed
).latestRoundData();
// answer = 1041000000 (8 decimals = $10.41)
```

**2. CRE (Chainlink Runtime Environment)**
A programmable workflow engine that:
- Reads data from blockchains (`EVMClient.callContract`)
- Calls external APIs (`HTTPClient`)
- Processes and transforms data
- Writes results on-chain via a signed report

### Earlyout's CRE workflow: `nav-bridge`

This is the pipeline that brings NAV data from Ethereum to Arc:

```
Cron trigger (every X minutes)
    ↓
For each of 10 listed funds:
    ├── Read Chainlink NAV feed on Ethereum mainnet
    │   (EVMClient.callContract → latestRoundData)
    │
    ├── Read issuer status from HTTP API
    │   (HTTPClient in node mode → consensus)
    │
    ├── Encode (fund, nav, decimals, updatedAt, status)
    │
    ├── Sign as a CRE report (runtime.report)
    │
    └── Write to NavOracle on Arc (writeReport → forwarder)
```

Each fund runs in its own try/catch, so one failed feed doesn't stop the others.

### NavOracle: receiving the data on Arc

```solidity
function onReport(bytes calldata metadata, bytes calldata report) external override {
    if (msg.sender != forwarder) revert InvalidSender(msg.sender);
    _checkWorkflow(metadata);

    (address fund, uint256 nav, uint8 navDecimals, uint64 updatedAt, uint8 status) =
        abi.decode(report, (address, uint256, uint8, uint64, uint8));

    // Reject stale or future reports
    if (updatedAt < _data[fund].updatedAt) revert StaleReport(...);
    if (updatedAt > block.timestamp + MAX_FUTURE_DRIFT) revert FutureReport(...);

    // Normalise to 18 decimals and store
    _data[fund].nav = normalised;
    _data[fund].updatedAt = updatedAt;
    _data[fund].status = status;
}
```

### Events: how contracts talk to the outside world

Events are cheap log entries that contracts emit. They can't be read by other contracts (only by off-chain code):

```solidity
// Emitted when a sale happens
event ClaimSold(
    uint256 indexed claimId,     // "indexed" = searchable
    address indexed receiver,
    uint256 amount,
    uint256 usdcPaid,
    uint256 nav,
    uint16 baseBps, uint16 utilBps, uint16 timeBps, uint16 creditBps
);
```

Events are how indexers know what happened on-chain.

### RPCs: talking to a node

**RPC** (Remote Procedure Call) is the JSON-RPC API that every Ethereum node exposes:

```bash
# Get the latest block number
curl -X POST https://rpc.testnet.arc.io \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Call a contract function (read-only)
curl -X POST https://rpc.testnet.arc.io \
  -d '{"jsonrpc":"2.0","method":"eth_call","params":[{
    "to": "0x7A40b36b012DC19b1f42dd68C13F4daE312e2319",
    "data": "0x..."
  },"latest"],"id":1}'
```

Libraries like **viem** and **ethers.js** wrap these calls in a developer-friendly API.

### Indexers: organised off-chain data

Reading events from an RPC is slow and expensive for complex queries. Indexers scan every block, decode events, and store them in a database:

| Indexer | How it works |
|---------|-------------|
| **The Graph** | Subgraphs: GraphQL schema + event handlers → hosted or decentralised |
| **Dune** | SQL analytics over all major chains |
| **Arkiv** | Write structured entities with TTL (expiry), queryable via SDK |

### Arkiv in Earlyout

Arkiv is Earlyout's public data layer. After every sale, the backend writes a structured entity:

```typescript
// Writing a claim to Arkiv
await arkiv.createEntity({
    schema: "earlyout.claims",
    data: {
        claimId, fund, symbol, settleAt, status: "open",
        amount, usdcPaid, nav, baseBps, utilBps, timeBps, creditBps,
        seller, holder, arcTx
    },
    expiresAt: settleAt  // entity disappears on its payout date
});
```

The Book screen reads from Arkiv with a simple query:
```typescript
select().from(earlyout.claims)
  .where(eq("status", "open"))
  .ownedBy(writerWallet)
  .orderBy(asc("settleAt"))
```

**Key insight:** Arkiv entities expire automatically. A claim entity that settles on January 15 disappears from the Book on January 15. No cleanup code needed.

---

## 🔗 In Earlyout

| Data flow | From → To | Mechanism |
|-----------|-----------|-----------|
| NAV price | Chainlink feed (Ethereum) → NavOracle (Arc) | CRE workflow + signed report |
| Issuer status | HTTP API → NavOracle (Arc) | CRE workflow + signed report |
| Sale events | Vault (Arc) → Arkiv | Backend reads receipt, writes entity |
| Book data | Arkiv → Frontend | Arkiv SDK query |
| Quotes/balances | Vault (Arc) → Frontend | Backend calls Arc RPC, serves via API |

---

## ✅ Check yourself

1. Why can't a smart contract just call an external API?
2. What's the difference between a Chainlink Data Feed and CRE?
3. What are events, and can other contracts read them?
4. What does an indexer do that an RPC can't?
5. Why do Arkiv entities have an expiry time?

---

[← Previous: Liquidity, markets and pricing](10-liquidity-markets-and-pricing.md) · [Next: DeFi →](12-defi.md)
