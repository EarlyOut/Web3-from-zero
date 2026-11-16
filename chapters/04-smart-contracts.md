# Chapter 4: Smart Contracts

> **Goal:** After this chapter you can read a Solidity function, understand what it does, and follow the logic of Earlyout's `Vault.sell()`.

---

## 🧒 Like you're 12

A smart contract is a **vending machine**. You put in money and press a button, and the machine does exactly what it's programmed to do — no negotiation, no "let me check with my manager". Once the machine is set up, nobody can change how it works.

The blockchain is like putting that vending machine in a glass box in a public square. Everyone can see what it does. Everyone can use it. Nobody can tamper with it.

---

## 🛠️ Engineer view

### What is a smart contract?

A smart contract is:
- **Code** deployed to a blockchain address
- **Immutable** — once deployed, the code cannot change (the state can)
- **Deterministic** — given the same input and state, it always produces the same output
- **Self-executing** — runs automatically when called, no server needed

Think of it as a **serverless function that runs on a global computer**, except:
- You pay per instruction (gas), not per request
- It's permanent and public
- Anyone can call it (unless the code restricts access)
- State persists between calls (like a database row that lives forever)

### Solidity basics

Solidity is the dominant language for Ethereum-compatible blockchains. If you know JavaScript or Java, you can read it.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;              // compiler version

contract Counter {                     // like a class
    uint256 public count;              // state variable (stored on-chain)

    function increment() external {    // anyone can call this
        count += 1;                    // costs gas (storage write)
    }

    function getCount() external view returns (uint256) {   // "view" = read-only, free
        return count;
    }
}
```

### Key concepts mapped to what you know

| Solidity concept | Closest equivalent |
|-----------------|-------------------|
| `contract` | Class (with persistent storage) |
| State variables | Database columns |
| `external` function | Public API endpoint |
| `view` / `pure` function | GET endpoint (no state change, no gas) |
| `msg.sender` | The authenticated caller (like `req.user`) |
| `msg.value` | Native currency sent with the call |
| `require(condition, "error")` | Input validation / assert |
| `revert SomeError()` | Throw an exception (all changes rolled back) |
| `emit EventName(...)` | Log entry (cheap storage, can't be read by contracts) |
| `modifier onlyOwner` | Middleware / decorator for access control |

### Reading Earlyout's `Vault.sell()`

Here's the actual function from Earlyout's deployed Vault contract, annotated:

```solidity
/// @notice Sell `amount` of a claim to the vault. The claim moves first,
///         then USDC goes to `receiver`.
function sell(
    uint256 claimId,       // which claim (identifies the fund + payout date)
    uint256 amount,        // how many claim units to sell
    uint256 minPayout,     // revert if payout is below this (MEV protection)
    address receiver       // who gets the USDC
)
    external               // callable from outside
    nonReentrant           // prevents reentrancy attacks (see Chapter 13)
    returns (uint256 payout)
{
    // 1. Get a quote — this checks HALT, REFUSE, and calculates the price
    Quote memory q = quote(claimId, amount);

    // 2. If the verdict isn't BUY, revert with the reason
    if (q.verdict != Verdict.BUY) revert NotBuyable(q.verdict, q.reason);

    // 3. Slippage protection: revert if price moved against the seller
    if (q.payout < minPayout) revert BelowMinPayout(q.payout, minPayout);
    payout = q.payout;

    // 4. Take the claim tokens from the seller FIRST (checks-effects-interactions)
    (address fund,) = claims.meta(claimId);
    claims.safeTransferFrom(msg.sender, address(this), claimId, amount, "");

    // 5. Update the vault's bookkeeping
    positions[claimId].amount += amount;    // track how many claims we hold
    positions[claimId].cost += payout;      // track what we paid
    fundCost[fund] += payout;              // track exposure per fund
    deployedUsdc += payout;                // track total USDC deployed

    // 6. Pay the seller in USDC
    usdc.safeTransfer(receiver, payout);

    // 7. Log it (indexers, the frontend, and Arkiv all read this)
    emit ClaimSold(claimId, receiver, amount, payout, q.nav,
                   q.baseBps, q.utilBps, q.timeBps, q.creditBps);
}
```

### The verdict system

Before any USDC moves, the contract runs a checklist:

```solidity
function _quote(...) internal view returns (Quote memory q) {
    // ...calculate price...

    // HALT conditions (serious — trading stops)
    if (status == STATUS_FAILED) return _verdict(q, Verdict.HALT, "Issuer reports failure");
    if (nav is stale)            return _verdict(q, Verdict.HALT, "NAV is stale");
    if (impaired[fund])          return _verdict(q, Verdict.HALT, "Fund is impaired");

    // REFUSE conditions (not dangerous, just can't do it right now)
    if (!listed)                 return _verdict(q, Verdict.REFUSE, "Fund not listed");
    if (amount == 0)             return _verdict(q, Verdict.REFUSE, "Amount is zero");
    if (already due)             return _verdict(q, Verdict.REFUSE, "Claim is already due");
    if (spread too wide)         return _verdict(q, Verdict.REFUSE, "Spread too wide");
    if (over exposure cap)       return _verdict(q, Verdict.REFUSE, "Over the fund cap");
    if (vault short of USDC)     return _verdict(q, Verdict.REFUSE, "Vault short of USDC");

    // All checks passed
    return _verdict(q, Verdict.BUY, "");
}
```

This is like an API that returns a structured response before doing anything — the frontend can call `quote()` for free (it's a `view` function) to show the user what will happen before they sign.

### Contract interactions

Contracts can call other contracts, forming a call chain:

```
User → ClaimWrapper.depositAndSell()
         → IssuerStub.requestRedemption()     // file the redemption
         → ClaimToken.mint()                  // create the claim
         → Vault.sell()                       // sell it for USDC
              → NavOracle.getNav()            // get the price
              → ClaimToken.safeTransferFrom() // take the claim
              → USDC.safeTransfer()           // pay the seller
```

If **anything** in this chain fails (a revert), the **entire transaction** rolls back. All or nothing. This is why you can trust `depositAndSell` — either everything happens, or nothing does.

### Deployment and immutability

```bash
# Compile
forge build

# Deploy (creates a new contract at a new address)
forge script script/Deploy.s.sol --rpc-url arc_testnet --broadcast
```

After deployment, the **code is frozen**. The contract's state (balances, positions, etc.) changes with every transaction, but the logic is permanent. If you need to change the logic, you deploy a new contract and migrate.

---

## 🔗 In Earlyout

| Contract | What it does | Lines of Solidity |
|----------|-------------|------------------|
| `FundRegistry` | Lists funds and their pricing terms | ~88 |
| `NavOracle` | Receives price feeds from Chainlink CRE | ~122 |
| `IssuerStub` | Stand-in issuer: queues redemptions, pays at NAV | ~83 |
| `ClaimToken` | ERC-1155 — one token per fund+payout date | ~71 |
| `ClaimWrapper` | Deposits fund tokens, mints claims, collects at settlement | ~128 |
| `Vault` | The core: quote, sell, buy, settle, markImpaired | ~336 |

All six are deployed on Arc testnet and can be read on the [Arc explorer](https://explorer.testnet.arc.io).

---

## ✅ Check yourself

1. What happens if any step in a multi-contract call chain reverts?
2. What's the difference between a `view` function and a regular function?
3. In `sell()`, why does the contract transfer the claim tokens *before* sending USDC?
4. What is the `nonReentrant` modifier for?
5. Can the owner of the Vault contract change the `sell()` logic after deployment?

---

[← Previous: Wallets, accounts and transactions](03-wallets-accounts-and-transactions.md) · [Next: Tokens and standards →](05-tokens-and-standards.md)
