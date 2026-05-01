### [I-6] Missing events for critical state changes, and dead code `_isActivePlayer`

**Description (Missing Events):** Key state changes in `selectWinner` and `withdrawFees` do not emit events, making off-chain monitoring (subgraphs, front-ends) unable to track these changes:
- `totalFees` update in `selectWinner`
- `raffleStartTime` reset in `selectWinner`
- `totalFees` reset in `withdrawFees`

**Recommended Mitigation:** Add events:

```solidity
event FeesUpdated(uint256 newTotalFees);
event RaffleReset(uint256 newStartTime);
```

---

**Description (Dead Code):** The function `_isActivePlayer` is declared but never called anywhere in the contract. It adds bytecode size and complexity with no benefit.

```solidity
function _isActivePlayer() internal view returns (bool) { ... }
```

**Recommended Mitigation:** Remove the function entirely.
