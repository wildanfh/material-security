### [I-5] Magic numbers used in `PuppyRaffle::selectWinner` reduce code readability

**Description:** Literal values `80`, `20`, and `100` are used directly in prize calculations without named constants, making their purpose unclear.

```solidity
uint256 prizePool = (totalAmountCollected * 80) / 100;
uint256 fee = (totalAmountCollected * 20) / 100;
```

**Recommended Mitigation:** Define named constants:

```solidity
uint256 public constant PRIZE_POOL_PERCENTAGE = 80;
uint256 public constant FEE_PERCENTAGE = 20;
uint256 public constant POOL_PRECISION = 100;
```
