# Chapter 16: Glossary, Quiz Answers and Next Steps

> **Goal:** Look up any term, check your quiz answers, and plan what's next.

---

## Glossary

| Term | Definition |
|------|-----------|
| **ABI** | Application Binary Interface — JSON description of a contract's functions, used by off-chain code to encode calls |
| **Account** | An address on the blockchain, either an EOA (controlled by a private key) or a contract |
| **Address** | 20-byte identifier derived from a public key (EOA) or deployer+nonce (contract) |
| **Allowlist** | A set of approved addresses that can hold/transfer restricted tokens |
| **AMM** | Automated Market Maker — a DEX that uses a mathematical formula instead of an order book |
| **AML** | Anti-Money Laundering — regulations to prevent illicit fund flows |
| **APY / APR** | Annual Percentage Yield (with compounding) / Rate (without) |
| **Arc** | Circle's blockchain, purpose-built for stablecoin finance. USDC is the gas token |
| **Arkiv** | A data layer for writing structured, expiring entities queryable via SDK |
| **Block** | A batch of transactions bundled together and added to the chain |
| **Blockchain** | A distributed ledger where blocks are cryptographically linked, forming an append-only chain |
| **Bridge** | A mechanism to move assets between different blockchains |
| **BPS (basis points)** | 1/100th of a percent. 100 bps = 1% |
| **CCTP** | Circle Cross-Chain Transfer Protocol — burns USDC on one chain, mints on another |
| **Claim** | In Earlyout: a token representing the right to receive USDC at a future payout date |
| **CLO** | Collateralised Loan Obligation — a structured product bundling corporate loans into tranches |
| **Composability** | The ability of smart contracts to call and build on each other ("money Legos") |
| **Consensus** | The process by which nodes agree on the state of the blockchain |
| **CRE** | Chainlink Runtime Environment — a programmable workflow engine for cross-chain data |
| **Decimals** | The number of decimal places a token uses (USDC = 6, most tokens = 18) |
| **DeFi** | Decentralised Finance — financial services built on smart contracts |
| **DEX** | Decentralised Exchange — a protocol for trading tokens without an intermediary |
| **ECDSA** | Elliptic Curve Digital Signature Algorithm — the signing scheme used by Ethereum |
| **EOA** | Externally Owned Account — an account controlled by a private key |
| **ERC-20** | Standard for fungible tokens (like USDC) |
| **ERC-721** | Standard for non-fungible tokens (NFTs) |
| **ERC-1155** | Standard for multi-token contracts (Earlyout's ClaimToken) |
| **ERC-4626** | Standard for tokenized vaults (Earlyout's Vault) |
| **EVM** | Ethereum Virtual Machine — the runtime that executes smart contract bytecode |
| **Finality** | The guarantee that a transaction is irreversible |
| **Flash loan** | A loan borrowed and repaid within a single transaction |
| **Foundry** | A Solidity development toolkit: forge (compile/test), cast (CLI calls), anvil (local node) |
| **Gas** | The unit measuring computational effort on the EVM. Users pay gas × gas price |
| **Gate** | A fund's ability to limit withdrawals (e.g. max 5% per quarter) |
| **Hash** | A fixed-size fingerprint produced by a one-way function (SHA-256, Keccak-256) |
| **HALT** | Earlyout verdict: trading is stopped (issuer failure, stale NAV, impairment) |
| **Impermanent loss** | Loss from providing liquidity to an AMM when token prices diverge |
| **KYC** | Know Your Customer — identity verification for compliance |
| **L1 / L2** | Layer 1 (base chain) / Layer 2 (scaling chain built on top) |
| **Liquidity** | How easily an asset can be bought or sold at a fair price |
| **LP** | Liquidity Provider — someone who deposits assets for others to trade against |
| **MEV** | Maximal Extractable Value — profit from reordering/inserting transactions in a block |
| **Merkle tree** | A hash tree that enables efficient proofs of inclusion |
| **NAV** | Net Asset Value — total fund assets minus liabilities, divided by shares |
| **NFT** | Non-Fungible Token — a unique digital asset (ERC-721) |
| **Nonce** | Transaction counter per account, prevents replay attacks |
| **Oracle** | A service that brings off-chain data onto the blockchain |
| **Private key** | A 256-bit secret number that controls a blockchain account |
| **Public key** | Derived from the private key; can be shared freely |
| **Reentrancy** | An attack where a contract calls back into the caller before state is updated |
| **REFUSE** | Earlyout verdict: the vault can't buy right now (over cap, insufficient USDC, etc.) |
| **Rollup** | An L2 that posts transaction data/proofs to L1 for security |
| **RPC** | Remote Procedure Call — the JSON-RPC API for communicating with blockchain nodes |
| **RWA** | Real-World Asset — a physical or financial asset represented on-chain |
| **Seed phrase** | 12 or 24 words that encode a master private key (BIP-39) |
| **Slippage** | The difference between expected and actual trade price |
| **Smart contract** | Code deployed to a blockchain that executes deterministically |
| **Spread** | The difference between buy and sell price. In Earlyout: base + util + time + credit |
| **Stablecoin** | A token designed to maintain a fixed value (usually $1) |
| **Tokenization** | Representing ownership of an asset as a blockchain token |
| **TVL** | Total Value Locked — the standard metric for DeFi protocol size |
| **USDC** | USD Coin — Circle's dollar-backed stablecoin |
| **USDT** | Tether — the largest stablecoin by market cap |
| **Vault** | A smart contract that accepts deposits and executes a strategy to generate yield |
| **viem** | A TypeScript library for interacting with EVM blockchains |

---

## Quiz Answers

### Chapter 1
1. The double-spend problem — proving you can't spend the same digital money twice, without a central authority.
2. "Trustless" means you don't need to trust any single party. You verify correctness through cryptography and consensus instead.
3. Banks check balances and prevent double-spending; payment processors route transactions and charge fees.
4. Web2: a username/password managed by a platform. Web3: a private key you control.
5. Smart contract code is immutable once deployed. The developer can change the state but not the code.

### Chapter 2
1. The output changes completely (avalanche effect).
2. No — elliptic curve multiplication is a one-way function.
3. Probabilistic: more confirmations = more certainty, but never 100%. Deterministic: once in a block, it's final.
4. Earlyout needs instant USDC settlement — "USDC arrives in the same transaction" only works with fast finality.
5. A binary hash tree that lets you prove a transaction is in a block with log₂(n) hashes.

### Chapter 3
1. EOA is controlled by a private key (human). Contract account is controlled by its code.
2. It prevents replay attacks — each transaction must have a unique, incrementing nonce.
3. 21,000 × 10 gwei = 210,000 gwei = 0.00021 ETH.
4. MEV is profit from reordering transactions. Users can get worse prices from front-running and sandwich attacks.
5. It prevents the transaction from executing at a price worse than expected (slippage/MEV protection).

### Chapter 4
1. The entire transaction rolls back — all state changes are undone.
2. `view` functions only read state (free, no gas). Regular functions change state (cost gas).
3. Checks-effects-interactions pattern: update state before making external calls to prevent reentrancy.
4. Prevents reentrancy attacks — blocks re-entering the function during execution.
5. No. Contract code is immutable. Only state changes with transactions.

### Chapter 5
1. `transfer`: you send your own tokens. `transferFrom`: a pre-approved spender sends tokens on behalf of the owner.
2. Contracts can't reach into your wallet — you explicitly grant permission via `approve`.
3. 1,500,000 (1.50 × 10⁶).
4. ERC-1155 supports multiple token types (one per fund+date) with many units each — perfect for claims.
5. It increases — the same number of shares represents more assets.

### Chapter 6
1. Optimistic: assumes valid, allows fraud challenges (7-day window). ZK: generates a mathematical proof of validity (instant finality on L1).
2. Bridges hold locked funds that, if hacked, leave wrapped tokens worthless.
3. CCTP burns on source and mints on destination — no locked pool to steal.
4. So users don't need a separate gas token; everything is in USDC.
5. Chain ID is included in the transaction signature to prevent replay across chains.

### Chapter 7
1. To provide price stability for commerce and DeFi, since crypto prices are volatile.
2. USDC publishes monthly attestations with detailed composition; USDT provides less detail.
3. Yes — the USDC contract has a blocklist function. It's used for sanctions/law enforcement compliance.
4. USDC is native (no bridged copies), gas is USDC (no ETH needed), deterministic finality.
5. USDC is widely trusted by institutions, backed 1:1, and native on Arc — ideal for fund settlements.

### Chapter 8
1. NAV = (total assets - liabilities) / shares. It's the fair value of one fund share.
2. A gate limits how much can be withdrawn in a period (e.g. 5% per quarter).
3. Traditional: days to weeks. Earlyout: same transaction (<2 seconds on Arc).
4. They lend directly to companies (more risk), skip the bank middleman, and earn the spread.
5. Base (minimum fee), utilisation (vault USDC deployment), time (uncertainty over waiting period), credit (issuer risk).

### Chapter 9
1. Representing ownership of a real-world asset as a token on a blockchain.
2. Improves: settlement speed, market hours, transparency. Doesn't fix: fund rules, credit risk.
3. A list of approved addresses. Tokenized securities require KYC — only verified entities can hold them.
4. A transfer agent maintains the official record of who owns what (the register/cap table).
5. The fund token represents ownership of the fund. The claim represents the right to receive USDC at a future payout date — it's the pending redemption, not the fund itself.

### Chapter 10
1. Order book: buyers and sellers post prices, matched when they cross. AMM: a formula determines the price from pool balances.
2. There aren't enough buyers/sellers for a traditional order book. The Vault acts as an always-on buyer.
3. $10.00 × (1 - 110/10000) = $10.00 × 0.989 = $9.89.
4. Financial uncertainty grows with √t, not linearly (from option pricing theory / random walk).
5. The utilisation term (`utilBps`) increases, widening the spread.

### Chapter 11
1. Different nodes would get different responses → consensus breaks. Oracles solve this by posting one agreed-upon value on-chain.
2. Data Feeds are pre-built, continuously updated prices. CRE is a programmable workflow that can read feeds, call APIs, and write custom reports.
3. Events are log entries emitted by contracts. Only off-chain code can read them; other contracts cannot.
4. Indexers organise blockchain data into queryable databases. RPCs only let you read individual blocks/transactions — complex queries are slow.
5. Arkiv entities expire so stale data doesn't linger. Claims expire at payout; fund data expires if not refreshed.

### Chapter 12
1. A DEX is permissionless, runs on smart contracts, and operates 24/7 without KYC. Traditional exchanges have gatekeepers.
2. price = y/x. Buying Token A reduces its supply in the pool, increasing its price.
3. The spread between the discounted purchase price and the full NAV collected at settlement.
4. Contracts can call other contracts without permission — they stack like building blocks.
5. To prevent LPs from withdrawing USDC that's committed to claims, which would leave the vault unable to pay.

### Chapter 13
1. A contract calls back into itself before state is updated, draining funds. Prevented by updating state before external calls + `nonReentrant`.
2. If the price feed is wrong, every trade based on it misprices. An attacker can profit by manipulating the oracle.
3. First check inputs, then update state, then interact with other contracts.
4. HALT verdict stops trading; `markImpaired` writes down claims; LP lock pauses deposits/withdrawals during the transition.
5. If a token passes all four Howey criteria, it's a security subject to regulation. Fund shares clearly qualify.

### Chapter 14
1. `forge test` runs the test suite. `forge script` executes a script (usually for deployment).
2. `cast call` reads a contract function. It's a view call — no gas cost, no transaction.
3. A JSON schema of a contract's functions. Without it, off-chain code doesn't know how to encode arguments.
4. The first 4 bytes of `keccak256(functionSignature)`, used to route a call to the right function.
5. Try it! `cast call 0x7A40b36b012DC19b1f42dd68C13F4daE312e2319 "sharePrice()(uint256)" --rpc-url https://rpc.testnet.arc.io`

### Chapter 15
1. Seller calls `depositAndSell` → ClaimWrapper takes fund tokens → files redemption → mints claim → Vault buys claim → seller gets USDC. 90 days later, keeper calls `settle` → Vault collects full NAV from issuer → LP pool grows by the spread.
2. From the spread: buy at NAV - 1.25%, collect full NAV at settlement. Sustainable because it's based on real time-preference value, not token inflation.
3. `markImpaired` writes down claims. Trading HALTs. LP deposits/withdrawals pause. The loss is shared by all LPs in one transaction.
4. Atomic: either everything succeeds or nothing does. Two separate transactions could leave the user with an unusable claim if the sell fails.
5. The keeper calls `settle()` after the payout date to trigger collection from the issuer. It's needed because contracts can't trigger themselves.

---

## Next Steps

### Keep learning
- **Read the contracts:** the full source is in `contracts/src/` — you can now follow every line
- **Follow the transactions:** every proof link in the Earlyout README leads to a real Arc testnet transaction
- **Build something:** fork the repo, deploy your own Vault, list a custom fund
- **Join the ecosystem:** follow Ethereum, Chainlink, and Circle developer channels

### Suggested reading
- [Ethereum.org](https://ethereum.org/developers) — official developer documentation
- [Solidity docs](https://docs.soliditylang.org/) — the language reference
- [Chainlink docs](https://docs.chain.link/) — oracle and CRE documentation
- [OpenZeppelin docs](https://docs.openzeppelin.com/) — standard contract library
- [viem docs](https://viem.sh/) — the TypeScript library
- [Foundry book](https://book.getfoundry.sh/) — the Foundry toolkit

### Where AI fits in Web3
- **Code generation:** AI can write Solidity, but always review and test
- **Auditing assistance:** AI finds common patterns but misses novel attacks
- **Data analysis:** AI can process on-chain data, explain transactions, generate reports
- **UX:** AI can explain what a transaction does in plain language before the user signs
- **What AI can't do:** replace independent security audits, make governance decisions, hold private keys safely

---

**Congratulations!** You've gone from "what is a blockchain?" to understanding a production-grade DeFi system. 🎉

[← Previous: Earlyout end to end](15-earlyout-end-to-end.md) · [Back to course map →](../README.md)
