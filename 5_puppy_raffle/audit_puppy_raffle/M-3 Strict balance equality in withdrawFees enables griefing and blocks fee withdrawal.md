### [M-3] Strict balance equality check in `PuppyRaffle::withdrawFees` enables griefing attacks and permanently locks fees

**Description:** `withdrawFees` requires the contract balance to exactly equal `totalFees`. Any ETH sent to the contract via `selfdestruct` or direct transfer (which bypasses the `payable` functions) will cause this check to fail permanently.

```solidity
require(address(this).balance == uint256(totalFees), "PuppyRaffle: There are currently players active!");
```

This check is also broken if `totalFees` overflows (see H-3), making it impossible to ever withdraw fees even when no players are active.

**Impact:** An attacker can send as little as 1 wei via `selfdestruct` to permanently prevent the `feeAddress` from ever withdrawing fees.

**Proof of Concept:**
1. A raffle completes, `totalFees = 0.2 ETH`, `address(this).balance = 0.2 ETH`.
2. Attacker deploys a contract with 1 wei and calls `selfdestruct(address(puppyRaffle))`.
3. `address(this).balance = 0.2 ETH + 1 wei`.
4. `withdrawFees` reverts — the check can never pass again.

**Recommended Mitigation:** Replace the strict equality check with a `>=` comparison, or simply remove it since `totalFees` already tracks the exact withdrawable amount:

```diff
  function withdrawFees() external {
-     require(address(this).balance == uint256(totalFees), "PuppyRaffle: There are currently players active!");
      uint256 feesToWithdraw = totalFees;
      totalFees = 0;
      (bool success,) = feeAddress.call{value: feesToWithdraw}("");
      require(success, "PuppyRaffle: Failed to withdraw fees");
  }
```
