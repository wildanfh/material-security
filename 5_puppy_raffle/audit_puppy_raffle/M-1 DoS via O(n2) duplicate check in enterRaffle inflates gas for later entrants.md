### [M-1] Looping through players array to check duplicates in `PuppyRaffle::enterRaffle` is a potential DoS attack, incrementing gas costs for future entrants

**Description:** `PuppyRaffle::enterRaffle` uses a nested loop to check for duplicate players. The complexity is O(n²) — as more players enter, the gas cost grows quadratically. Players who enter early pay much less gas than those who enter later.

```solidity
// @audit DoS — O(n²) complexity
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

An attacker can front-fill the array with hundreds of addresses to make participation prohibitively expensive for everyone else, effectively guaranteeing their own win.

**Impact:** Gas costs for later entrants grow dramatically, discouraging participation and creating an unfair advantage for early entrants.

**Proof of Concept:**

<details>
<summary>Proof of Code</summary>

```solidity
function testDenialOfService() public {
    vm.txGasPrice(1);
    uint256 playersNum = 100;
    address[] memory players = new address[](playersNum);
    for (uint256 i = 0; i < playersNum; i++) {
        players[i] = address(uint160(i));
    }

    uint256 gasStart = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);
    uint256 gasUsedFirst = (gasStart - gasleft()) * tx.gasprice;
    console.log("Gas cost of the first 100 players: ", gasUsedFirst);

    address[] memory playersTwo = new address[](playersNum);
    for (uint256 i = 0; i < playersNum; i++) {
        playersTwo[i] = address(uint160(i + playersNum));
    }

    uint256 gasStartTwo = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * playersNum}(playersTwo);
    uint256 gasUsedSecond = (gasStartTwo - gasleft()) * tx.gasprice;
    console.log("Gas cost of the second 100 players: ", gasUsedSecond);

    assert(gasUsedFirst < gasUsedSecond);
}
```

Results (approximate):
- First 100 players: ~6,252,048 gas
- Second 100 players: ~18,068,138 gas (3x higher)

</details>

**Recommended Mitigation:** Use a `mapping(address => bool)` for duplicate detection instead of a nested loop:

```diff
+ mapping(address => bool) private addressEntered;

  function enterRaffle(address[] memory newPlayers) public payable {
      require(msg.value == entranceFee * newPlayers.length, "...");
      for (uint256 i = 0; i < newPlayers.length; i++) {
+         require(!addressEntered[newPlayers[i]], "PuppyRaffle: Duplicate player");
+         addressEntered[newPlayers[i]] = true;
          players.push(newPlayers[i]);
      }
-     // remove the O(n²) loop entirely
  }
```
