# Chapter 5: Tokens and Standards

> **Goal:** After this chapter you can explain ERC-20, ERC-721, ERC-1155, ERC-4626, approvals, and decimals.

---

## 🧒 Like you're 12

Tokens are like different types of tickets:

- **ERC-20** = movie tickets — they're all the same, interchangeable. "I have 100 USDC" is like "I have 100 identical tickets".
- **ERC-721** = baseball cards — each one is unique. "I own CryptoPunk #3100" is like "I own the one-of-a-kind rookie card".
- **ERC-1155** = concert tickets by section — "I have 50 units of Section-A tickets". Same within a group, but different groups exist.

---

## 🛠️ Engineer view

### Why standards?

A token is just a **smart contract that tracks balances**. Without standards, every token would have different function names, and wallets/exchanges couldn't support them. ERC (Ethereum Request for Comments) standards define a common interface.

### ERC-20: Fungible tokens

The most common standard. Think of it as a database table:

```
balances:  { 0xAlice: 1000, 0xBob: 500, 0xVault: 50000 }
```

Key functions:

```solidity
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}
```

**The approval pattern** is crucial:

```
Step 1: Alice calls USDC.approve(Vault, 1000)
        → "Vault, you may spend up to 1000 of my USDC"

Step 2: Vault calls USDC.transferFrom(Alice, Vault, 500)
        → "Take 500 of Alice's USDC" (works because she approved it)
```

This two-step pattern exists because contracts can't reach into your wallet — you have to explicitly grant permission.

### Decimals: the gotcha

Tokens don't have decimal points. They use integers with a `decimals` field:

| Token | Decimals | 1.00 token stored as |
|-------|----------|---------------------|
| USDC | 6 | 1_000_000 |
| ETH/WETH | 18 | 1_000_000_000_000_000_000 |
| DAI | 18 | 1_000_000_000_000_000_000 |

**Common bug:** treating a 6-decimal token like an 18-decimal one (or vice versa) gives you a number off by 10¹².

In Earlyout, claim amounts are 18 decimals but USDC is 6, so the conversion is:
```
usdcPaid = amount * nav / 1e30
// 18 (amount) + 18 (nav) - 6 (usdc) = 30
```

### ERC-721: Non-fungible tokens (NFTs)

Each token has a unique `tokenId`. Ownership is a mapping:

```
owners: { 1: 0xAlice, 2: 0xBob, 3: 0xAlice }
```

```solidity
interface IERC721 {
    function ownerOf(uint256 tokenId) external view returns (address);
    function transferFrom(address from, address to, uint256 tokenId) external;
    function approve(address to, uint256 tokenId) external;
}
```

### ERC-1155: Multi-token standard

Combines fungible and non-fungible in one contract. Each `id` is a token type, and each type can have multiple units:

```
balances: {
    id 1 (ACRED-2027-Q1): { 0xAlice: 50, 0xVault: 200 },
    id 2 (JAAA-2027-Jan):  { 0xBob: 100 }
}
```

**Why Earlyout uses ERC-1155:** each claim is identified by (fund, payout date). Many investors can sell the same fund for the same payout date, so there can be many units of the same claim ID. ERC-1155 handles this naturally — one contract, many claim types, each with multiple units.

```solidity
// From Earlyout's ClaimToken
function claimId(address fund, uint64 settleAt) public pure returns (uint256) {
    return uint256(keccak256(abi.encode(fund, settleAt)));
}
```

### ERC-4626: Tokenized vaults

A standard for yield-bearing vaults. Deposit an asset, get shares back. The shares represent your proportional ownership of the vault's assets.

```
Deposit: 1000 USDC → receive 1000 eoUSDC shares (at 1:1 initially)
         The vault invests the USDC (buys claims)
         Claims settle, vault earns the spread
         1000 eoUSDC now redeemable for 1012 USDC

Withdraw: burn 1000 eoUSDC → receive 1012 USDC
```

Key functions:

```solidity
interface IERC4626 {
    function deposit(uint256 assets, address receiver) external returns (uint256 shares);
    function withdraw(uint256 assets, address receiver, address owner) external returns (uint256 shares);
    function convertToAssets(uint256 shares) external view returns (uint256 assets);
    function totalAssets() external view returns (uint256);
}
```

**Earlyout's Vault is an ERC-4626 vault** where:
- The **asset** is USDC
- LPs deposit USDC and get `eoUSDC` shares
- The vault uses that USDC to buy claims at a discount
- When claims settle, the vault collects the full NAV amount
- The difference (the spread) is the LP's yield

### Approval best practices

| Pattern | Risk |
|---------|------|
| `approve(spender, MAX_UINT256)` | Convenient but dangerous — the spender can drain everything |
| `approve(spender, exactAmount)` | Safer — only the needed amount |
| `permit` (EIP-2612) | Sign an off-chain approval (saves gas, one tx instead of two) |

---

## 🔗 In Earlyout

| Standard | Where | Why |
|----------|-------|-----|
| ERC-20 | USDC, FundToken | Fungible: dollars and fund shares |
| ERC-1155 | ClaimToken | One ID per (fund, payout date), many units per ID |
| ERC-4626 | Vault | LPs deposit USDC, earn yield from spreads |
| Allowlists | FundToken, ClaimToken | Only KYC'd addresses can hold/transfer |

---

## ✅ Check yourself

1. What's the difference between `transfer` and `transferFrom`?
2. Why do you need to call `approve` before a contract can move your tokens?
3. If USDC has 6 decimals, what integer represents $1.50?
4. Why does Earlyout use ERC-1155 instead of ERC-20 for claims?
5. In an ERC-4626 vault, what happens to the share price when the vault earns yield?

---

[← Previous: Smart contracts](04-smart-contracts.md) · [Next: Chains, scaling and bridges →](06-chains-scaling-and-bridges.md)
