# Chapter 14: Developer Stack and Hands-On

> **Goal:** After this chapter you can use Foundry, viem, and a block explorer; and run read-only calls on a testnet.

---

## 🧒 Like you're 12

Time to get your hands dirty! This chapter is like a workshop. You'll learn the tools that Web3 developers use every day, and you'll actually talk to a real blockchain.

---

## 🛠️ Engineer view

### The developer toolkit

| Tool | What it does | Equivalent in Web2 |
|------|-------------|-------------------|
| **Foundry** | Compile, test, deploy Solidity contracts | Compiler + test runner (like `cargo` or `go test`) |
| **viem** | TypeScript library to talk to EVM chains | HTTP client / ORM |
| **ethers.js** | Older TypeScript library (similar to viem) | Same (older) |
| **Hardhat** | JS-based alternative to Foundry | Same as Foundry, but slower |
| **Block explorer** | Web UI to inspect transactions and contracts | Database admin tool / CloudWatch |
| **MetaMask** | Browser wallet | SSH key / auth token |

### Foundry: the Solidity toolkit

```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Create a new project
forge init my-project
cd my-project

# Compile
forge build

# Run tests
forge test                    # all tests
forge test -vvvv              # verbose (shows call traces)
forge test --match-test testSell  # specific test

# Deploy
forge script script/Deploy.s.sol --rpc-url $RPC_URL --broadcast

# Read a contract
cast call $VAULT_ADDRESS "sharePrice()" --rpc-url $RPC_URL
```

### Writing a Foundry test

```solidity
// test/Vault.t.sol
import {Test} from "forge-std/Test.sol";
import {Vault} from "../src/Vault.sol";

contract VaultTest is Test {
    Vault vault;

    function setUp() public {
        // Deploy contracts, set up state
        vault = new Vault(...);
    }

    function testSellHappyPath() public {
        // Arrange
        uint256 amount = 1e18;  // 1 token (18 decimals)

        // Act
        uint256 payout = vault.sell(claimId, amount, 0, seller);

        // Assert
        assertGt(payout, 0, "Payout should be positive");
        assertEq(usdc.balanceOf(seller), payout, "Seller should receive USDC");
    }

    function testSellRevertsWhenHalted() public {
        // Set issuer to failed
        oracle.setStatus(fund, oracle.STATUS_FAILED());

        // Expect revert
        vm.expectRevert();
        vault.sell(claimId, 1e18, 0, seller);
    }
}
```

### viem: talking to chains from TypeScript

```typescript
import { createPublicClient, http } from 'viem';

// Create a client for Arc testnet
const client = createPublicClient({
  transport: http('https://rpc.testnet.arc.io'),
});

// Read the vault's share price
const sharePrice = await client.readContract({
  address: '0x7A40b36b012DC19b1f42dd68C13F4daE312e2319',
  abi: vaultAbi,
  functionName: 'sharePrice',
});
console.log('Share price:', sharePrice);  // e.g. 1011360n (= $1.011360)

// Read a quote
const quote = await client.readContract({
  address: '0x7A40b36b012DC19b1f42dd68C13F4daE312e2319',
  abi: vaultAbi,
  functionName: 'quoteFund',
  args: [fundAddress, amount],
});
console.log('Verdict:', quote.verdict);  // 0 = BUY
console.log('Payout:', quote.payout);
```

### Block explorers

Go to [explorer.testnet.arc.io](https://explorer.testnet.arc.io) and try:

1. **Search an address:** paste `0x7A40b36b012DC19b1f42dd68C13F4daE312e2319` (the Vault)
   - See the contract's code, transactions, and events
2. **Search a transaction:** paste `0x12aefa9df1553daadc4c026566387f0fdde7cf9af6a0153a5eac5e141973a5ee`
   - See the sale: who called what, what events were emitted, gas used
3. **Read contract functions:** on the contract page, go to "Read Contract"
   - Call `sharePrice()`, `idleUsdc()`, `totalAssets()` — no wallet needed

### Hands-on exercises

**Exercise 1: Read the Vault's state using `cast`**

```bash
# Get the share price
cast call 0x7A40b36b012DC19b1f42dd68C13F4daE312e2319 \
  "sharePrice()(uint256)" \
  --rpc-url https://rpc.testnet.arc.io

# Get idle USDC
cast call 0x7A40b36b012DC19b1f42dd68C13F4daE312e2319 \
  "idleUsdc()(uint256)" \
  --rpc-url https://rpc.testnet.arc.io

# Get total assets
cast call 0x7A40b36b012DC19b1f42dd68C13F4daE312e2319 \
  "totalAssets()(uint256)" \
  --rpc-url https://rpc.testnet.arc.io
```

**Exercise 2: Decode a transaction**

```bash
# Decode the calldata of a sell transaction
cast 4byte-decode 0x$(cast tx 0x12aefa9df1553daadc4c026566387f0fdde7cf9af6a0153a5eac5e141973a5ee input --rpc-url https://rpc.testnet.arc.io | cut -c3-)
```

**Exercise 3: Try it in the browser**

1. Go to [earlyout.vercel.app/sell](https://earlyout.vercel.app/sell)
2. Connect MetaMask on Arc testnet (chain 5042002, RPC: https://rpc.testnet.arc.io)
3. Click **Try as judge** to get demo tokens and gas
4. Sell 0.001 ACRED and watch the quote breakdown
5. Check the [Book](https://earlyout.vercel.app/book) — your sale should appear

### ABI: Application Binary Interface

The ABI is a JSON description of a contract's functions. It's how off-chain code knows how to encode function calls:

```json
{
  "name": "sell",
  "type": "function",
  "inputs": [
    { "name": "claimId", "type": "uint256" },
    { "name": "amount", "type": "uint256" },
    { "name": "minPayout", "type": "uint256" },
    { "name": "receiver", "type": "address" }
  ],
  "outputs": [
    { "name": "payout", "type": "uint256" }
  ]
}
```

The first 4 bytes of the encoded call are the **function selector** (keccak256 of the function signature, truncated):
```
keccak256("sell(uint256,uint256,uint256,address)") → 0x12345678...
Call data = 0x12345678 + encoded args
```

---

## 🔗 In Earlyout

| Tool | How Earlyout uses it |
|------|---------------------|
| Foundry | Compiles, tests (63 tests), and deploys the contracts |
| viem | The web app reads from Arc and encodes transactions |
| Arc explorer | Every proof link in the README |
| MetaMask | Sellers sign transactions in the browser |
| Arc Studio | Reviewed, tested, and deployed the contracts |

---

## ✅ Check yourself

1. What's the difference between `forge test` and `forge script`?
2. What does `cast call` do, and does it cost gas?
3. What is an ABI, and why do you need it?
4. What's a function selector?
5. Run `cast call` to read the Vault's `sharePrice()` on Arc testnet.

---

[← Previous: Security, risk and regulation](13-security-risk-and-regulation.md) · [Next: Earlyout end to end →](15-earlyout-end-to-end.md)
