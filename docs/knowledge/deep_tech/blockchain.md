# Blockchain & Smart Contract Security

> A blockchain is a database everyone can audit and no one can edit. The security problems are not in the chain — they are in the code people deploy on top of it.

---

## First Principles: What a Blockchain Actually Is

Strip away the hype:

```
A blockchain is:
  Linked list of blocks
  + Each block contains a hash of the previous block  ← The "chain"
  + Distributed across many nodes
  + With a consensus rule for which chain is canonical
  
A block contains:
  - Previous block hash (links the chain)
  - Merkle root of transactions (tamper-evident)
  - Nonce (proof of work) or validator signature (proof of stake)
  - Timestamp
  - Transaction data
```

**Why tampering is hard:**
Changing any historical block changes its hash, which invalidates the next block's "previous hash" field, which cascades forward. An attacker would need to recompute all subsequent blocks *and* outpace the rest of the network (51% attack).

**What this means practically:**
Immutability is real but has costs. Bugs in smart contracts are also immutable — you cannot patch deployed code.

---

## Ethereum & Smart Contracts

**Smart contracts** are code that runs on the blockchain. They execute exactly as written, cannot be stopped once deployed, and handle real money.

```
Ethereum Virtual Machine (EVM):
  - Stack-based virtual machine
  - Deterministic: same input → same output on every node
  - Gas: computation costs Ether, prevents infinite loops
  - State: contracts have persistent storage on-chain
  
Solidity → Compiled → EVM Bytecode → Deployed to address
```

### Smart Contract Anatomy

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    // State variables (stored on-chain, expensive to write)
    mapping(address => uint256) public balances;
    address public owner;
    
    // Events (logged, cheaper than storage)
    event Deposit(address indexed user, uint256 amount);
    event Withdrawal(address indexed user, uint256 amount);
    
    constructor() {
        owner = msg.sender;  // Deployer is owner
    }
    
    // payable = can receive Ether
    function deposit() external payable {
        balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }
    
    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;              // ← Update state FIRST
        (bool success,) = msg.sender.call{value: amount}("");  // Then send
        require(success, "Transfer failed");
        emit Withdrawal(msg.sender, amount);
    }
}
```

---

## Smart Contract Vulnerabilities

These vulnerabilities have collectively caused **billions of dollars** in losses.

### 1. Reentrancy

**The DAO hack (2016): $60 million stolen.**

**How it works:** Contract A calls Contract B. Before A updates its state, B calls back into A. A thinks B hasn't been paid yet.

```solidity
// VULNERABLE — state not updated before external call
function withdraw(uint256 amount) external {
    require(balances[msg.sender] >= amount);
    (bool success,) = msg.sender.call{value: amount}("");  // ← External call FIRST
    require(success);
    balances[msg.sender] -= amount;  // ← State update SECOND — too late!
}

// ATTACK CONTRACT
contract Attacker {
    VulnerableVault victim;
    
    function attack() external payable {
        victim.deposit{value: 1 ether}();
        victim.withdraw(1 ether);
    }
    
    // Called every time victim sends Ether
    receive() external payable {
        if (address(victim).balance >= 1 ether) {
            victim.withdraw(1 ether);  // Re-enter before balance is updated!
        }
    }
}
```

**Fix:** Checks-Effects-Interactions pattern — update state before external calls.

```solidity
function withdraw(uint256 amount) external {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;   // ✓ State update FIRST (Effects)
    (bool success,) = msg.sender.call{value: amount}("");  // Then external call (Interactions)
    require(success);
}
// Or use OpenZeppelin's ReentrancyGuard modifier
```

### 2. Integer Overflow/Underflow

Pre-Solidity 0.8.0, arithmetic operations could silently wrap around:

```solidity
// Vulnerable (Solidity < 0.8.0)
uint256 balance = 0;
balance -= 1;  // Wraps to 2^256 - 1 (massive number)

// Fix: Solidity 0.8+ has built-in overflow checks
// Or use OpenZeppelin SafeMath for older code
```

### 3. tx.origin Authentication

`tx.origin` is the original transaction sender (your wallet). `msg.sender` is the immediate caller (might be a contract). Confusing them enables phishing attacks.

```solidity
// VULNERABLE
function transfer(address to, uint amount) external {
    require(tx.origin == owner);  // ← Can be bypassed by malicious intermediary contract
    // ...
}

// FIX
require(msg.sender == owner);  // Always use msg.sender for auth
```

### 4. Front-Running

Transactions sit in the mempool (public pending pool) before inclusion. Attackers can see your transaction and pay higher gas to execute first.

**Example:** You submit a DEX trade. A bot sees it, submits their own trade with higher gas (executes first, moves price), then you execute at worse price. This is **MEV (Maximal Extractable Value)**.

```
Mitigation:
- Commit-reveal schemes (submit hash, then reveal)
- Slippage protection (specify minimum acceptable price)
- Private mempools (Flashbots Protect)
```

### 5. Oracle Manipulation

Smart contracts cannot access off-chain data natively. They use **oracles** (e.g., Chainlink) to get prices. If you can manipulate the oracle, you can manipulate the contract.

```
Flash loan attack pattern:
1. Borrow enormous amount of assets (uncollateralized flash loan)
2. Use them to manipulate a price oracle (crash/pump a token price)
3. Exploit a DeFi protocol that trusts the manipulated price
4. Repay flash loan
5. Keep profit
→ Multiple $100M+ exploits have used this pattern
```

---

## Smart Contract Auditing

### Manual Review Checklist

```
Access Control:
□ Who can call each function?
□ Is ownership transfer safe (2-step)?
□ Are privileged functions protected?

Arithmetic:
□ Are all arithmetic operations safe from overflow?
□ Division rounding — does rounding always favor the protocol?

External Interactions:
□ Does state update before external calls? (reentrancy)
□ Are return values from external calls checked?
□ What happens if a called contract reverts?

Oracle Usage:
□ Can the price feed be manipulated in one transaction?
□ Is there a TWAP (Time Weighted Average Price) instead of spot?

Logic:
□ Can a single user drain the contract?
□ What happens with 0 inputs?
□ What happens with max uint256 inputs?
```

### Automated Tools

```bash
# Slither — static analyzer (free, Python)
pip install slither-analyzer
slither contract.sol

# Mythril — symbolic execution
pip install mythril
myth analyze contract.sol

# Foundry — testing framework (write tests in Solidity)
curl -L https://foundry.paradigm.xyz | bash
forge init my-project
# Write invariant fuzz tests:
# forge test --fuzz-runs 10000
```

### Practice Platforms

| Platform | Type | Cost |
| :--- | :--- | :--- |
| [Ethernaut (OpenZeppelin)](https://ethernaut.openzeppelin.com/) | CTF — Solidity security | Free |
| [Damn Vulnerable DeFi](https://damnvulnerabledefi.xyz/) | CTF — DeFi attacks | Free |
| [Capture The Ether](https://capturetheether.com/) | CTF | Free |
| [Code4rena](https://code4rena.com/) | Competitive audit contests | Free (paid for findings) |
| [Sherlock](https://audits.sherlock.xyz/) | Audit contests | Free (paid for findings) |

---

## Blockchain Forensics

On-chain data is public and permanent — ideal for investigation.

```python
# pip install web3
from web3 import Web3

# Connect to Ethereum (use your own RPC or Infura/Alchemy)
w3 = Web3(Web3.HTTPProvider('https://mainnet.infura.io/v3/YOUR_KEY'))

# Trace funds from an address
address = '0xYourAddressHere'
balance = w3.eth.get_balance(address)
print(f"Balance: {w3.from_wei(balance, 'ether')} ETH")

# Get transactions (use Etherscan API for full history)
# https://api.etherscan.io/api?module=account&action=txlist&address=...
```

**Chain analysis tools:**

| Tool | Purpose | Cost |
| :--- | :--- | :--- |
| [Etherscan](https://etherscan.io) | Block explorer, transaction tracing | Free |
| [Breadcrumbs.app](https://breadcrumbs.app) | Visual wallet tracing | Free tier |
| [Phalcon](https://phalcon.blocksec.com/) | Transaction attack tracing | Free |
| [Tenderly](https://tenderly.co/) | Transaction simulation and debugging | Free tier |
| [Arkham Intelligence](https://arkhamintelligence.com/) | Entity labeling, fund tracing | Free tier |

---

## Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Ethereum Documentation](https://ethereum.org/en/developers/docs/) | Official docs | Free |
| [Solidity Documentation](https://docs.soliditylang.org/) | Language reference | Free |
| [Smart Contract Security by Trail of Bits](https://github.com/crytic/building-secure-contracts) | Security guide | Free |
| [Secureum](https://secureum.xyz/) | Audit training | Free |
| [The Defiant](https://thedefiant.io/) | DeFi news/research | Free tier |
| [awesome-blockchain](https://github.com/yjjnls/awesome-blockchain) | Curated blockchain resource list | Free |
| [Learn Me a Bitcoin](https://learnmeabitcoin.com/) | Visual Bitcoin/blockchain explainer from first principles | Free |
