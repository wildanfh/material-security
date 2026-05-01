### [H-3] Integer overflow on `PuppyRaffle::totalFees` causes collected fees to be permanently stuck in the contract

**Description:** `PuppyRaffle` uses Solidity `^0.7.6`, which does not have built-in overflow protection. The `totalFees` variable is typed as `uint64`, with a maximum value of ~18.44 ETH. If cumulative fees exceed this value, `totalFees` wraps around to a small number.

```solidity
uint64 public totalFees = 0;
// ...
totalFees = totalFees + uint64(fee); // unsafe cast + potential overflow
```

When overflow occurs, `totalFees` no longer matches `address(this).balance`, causing `withdrawFees` to revert permanently:

```solidity
require(address(this).balance == uint256(totalFees), "PuppyRaffle: There are currently players active!");
```

**Impact:** The protocol owner can never withdraw accumulated fees. All ETH representing fees is permanently locked in the contract.

**Proof of Concept:**

<details>
<summary>Proof of Code</summary>

```solidity
function testTotalFeesOverflow() public playersEntered {
    vm.warp(block.timestamp + duration + 1);
    vm.roll(block.number + 1);
    puppyRaffle.selectWinner();
    uint256 startingTotalFees = puppyRaffle.totalFees();

    uint256 playersNum = 89;
    address[] memory players = new address[](playersNum);
    for (uint256 i = 0; i < playersNum; i++) {
        players[i] = address(uint160(i));
    }
    puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);

    vm.warp(block.timestamp + duration + 1);
    vm.roll(block.number + 1);
    puppyRaffle.selectWinner();

    uint256 endingTotalFees = puppyRaffle.totalFees();
    assert(endingTotalFees < startingTotalFees); // overflow occurred

    vm.prank(puppyRaffle.feeAddress());
    vm.expectRevert("PuppyRaffle: There are currently players active!");
    puppyRaffle.withdrawFees();
}
```

</details>

**Recommended Mitigation:** Upgrade to Solidity `^0.8.0` for automatic overflow checks, and change `totalFees` to `uint256`:

```diff
- pragma solidity ^0.7.6;
+ pragma solidity ^0.8.18;

- uint64 public totalFees = 0;
+ uint256 public totalFees = 0;

- totalFees = totalFees + uint64(fee);
+ totalFees = totalFees + fee;
```
