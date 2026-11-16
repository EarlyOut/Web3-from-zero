# Chapter 13: Security, Risk and Regulation

> **Goal:** After this chapter you can name the main attacks and risks in Web3, and the rules that apply.

---

## 🧒 Like you're 12

Smart contracts hold real money, so bugs are expensive. Imagine if a typo in a vending machine let people get unlimited free snacks — except the snacks are worth millions of dollars. That's why security in Web3 is taken very seriously.

**Regulation** is the government's rulebook. Even though blockchains are global and permissionless, the people using them still live in countries with laws. If you're selling securities (like fund shares), those laws definitely apply.

---

## 🛠️ Engineer view

### Common attack vectors

| Attack | How it works | Damage (notable incidents) |
|--------|-------------|----------------------------|
| **Reentrancy** | A contract calls back into the caller before the first call finishes, draining funds | The DAO hack (2016, $60M) |
| **Oracle manipulation** | Attacker manipulates the price feed a contract reads | Mango Markets (2022, $114M) |
| **Flash loan attacks** | Borrow millions for one transaction, manipulate a market, pay it back | bZx (2020, $8M) |
| **Access control** | Missing `onlyOwner` or wrong permissions | Parity wallet (2017, $280M frozen forever) |
| **Front-running / MEV** | Bots see your pending transaction and trade ahead of you | Constant ongoing losses |
| **Bridge hacks** | Vulnerabilities in cross-chain bridges | Ronin ($625M), Wormhole ($320M) |
| **Private key compromise** | Attacker steals the deployer/admin key | Multiple DeFi protocols |

### Reentrancy: the classic

```solidity
// VULNERABLE CODE — DO NOT USE
function withdraw(uint256 amount) external {
    require(balances[msg.sender] >= amount);
    (bool ok,) = msg.sender.call{value: amount}("");  // ← sends ETH
    require(ok);
    balances[msg.sender] -= amount;  // ← state update AFTER the call
}
```

The attacker's contract:
```solidity
receive() external payable {
    if (address(victim).balance >= 1 ether) {
        victim.withdraw(1 ether);  // ← re-enters before balance is updated!
    }
}
```

**Fix:** Checks-Effects-Interactions pattern + `nonReentrant` modifier:

```solidity
// SAFE
function withdraw(uint256 amount) external nonReentrant {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;      // ← update state FIRST
    usdc.safeTransfer(msg.sender, amount); // ← then interact
}
```

Earlyout's `sell()` does exactly this: it transfers the claim *into* the vault and updates bookkeeping *before* sending USDC out.

### How Earlyout handles security

| Risk | Mitigation |
|------|-----------|
| **Reentrancy** | `nonReentrant` on `sell`, `buy`, `settle` |
| **Oracle manipulation** | Only the authorised CRE forwarder can write to NavOracle; stale/future reports rejected |
| **Overspending** | `minPayout` slippage protection on `sell`; `maxCost` on `buy` |
| **Exposure concentration** | Per-fund exposure caps in FundRegistry |
| **Issuer failure** | HALT verdict + `markImpaired` writes down claims for all LPs at once |
| **LP sniping** | Known issue (High): depositing just before settlement and withdrawing right after captures yield funded by earlier LPs. Fix designed but not deployed for testnet. |
| **Stale NAV** | `maxAge` per fund (96 hours); if exceeded, trading HALTs |
| **Unauthorised transfers** | Allowlists on ClaimToken and FundToken |
| **Admin key risk** | Deployer wallet transferred ownership post-deploy |

### Smart contract auditing

| Level | What it involves | Cost |
|-------|-----------------|------|
| **Self-review** | Developer reads their own code carefully | Free |
| **AI-assisted review** | Arc Studio's agent found 5 Medium findings | Included in Studio |
| **Peer review** | Another developer reviews | Time |
| **Professional audit** | Firms like Trail of Bits, OpenZeppelin, Spearbit | $50K–$500K+ |
| **Formal verification** | Mathematical proof that the code matches its spec | Expensive, slow |
| **Bug bounty** | Public reward for finding vulnerabilities | Ongoing cost |

### OpenZeppelin: the standard library

Most Solidity projects use **OpenZeppelin Contracts** — audited, battle-tested implementations of common patterns:

| OpenZeppelin contract | Used in Earlyout |
|----------------------|------------------|
| `ERC20` | FundToken |
| `ERC1155`, `ERC1155Supply` | ClaimToken |
| `ERC4626` | Vault |
| `Ownable` | All contracts (access control) |
| `ReentrancyGuard` | Vault |
| `SafeERC20` | Vault, ClaimWrapper, IssuerStub |

### Regulation

| Concept | What it means | Applies to |
|---------|--------------|-----------|
| **KYC** (Know Your Customer) | Verify identity before allowing access | Tokenized securities platforms |
| **AML** (Anti-Money Laundering) | Monitor for suspicious transactions | Exchanges, stablecoin issuers |
| **Securities law** | Rules about selling investment contracts | Fund tokens, most DeFi yield products |
| **MiCA** (EU) | Markets in Crypto-Assets regulation | All crypto service providers in EU |
| **SEC** (US) | Securities and Exchange Commission | Tokens that might be securities |

**The Howey Test** (US): is it a security?
1. Investment of money ✓
2. In a common enterprise ✓
3. With expectation of profits ✓
4. Derived from efforts of others ✓
→ If all four are true, it's probably a security.

**Tokenized fund shares** (like ACRED) are unambiguously securities. That's why:
- Only KYC'd addresses can hold them (allowlist)
- A licensed transfer agent maintains the register
- The fund follows existing securities regulations
- Earlyout's claim tokens inherit the same restrictions

---

## 🔗 In Earlyout

The contract's safety rules are not afterthoughts — they're core to the design:
- The **verdict system** (BUY/REFUSE/HALT) ensures no USDC moves unless every check passes.
- The **LP lock** (`_lpLock()`) pauses deposits and withdrawals when the oracle and the impairment flag disagree — preventing front-running of impairment events.
- The **exposure cap** per fund limits concentration risk.
- The `checks-effects-interactions` pattern is followed in every function that moves tokens.

---

## ✅ Check yourself

1. What is a reentrancy attack, and how is it prevented?
2. Why is oracle manipulation dangerous?
3. What is the checks-effects-interactions pattern?
4. Name three things Earlyout does to handle issuer failure risk.
5. What is the Howey Test, and why does it matter for tokenized fund shares?

---

[← Previous: DeFi](12-defi.md) · [Next: Developer stack and hands-on →](14-developer-stack-and-hands-on.md)
