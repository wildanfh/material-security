### [L-1] `PuppyRaffle::getActivePlayerIndex` returns 0 for both the player at index 0 and a non-existent player, causing confusion

**Description:** If a player is at index 0 in the `players` array, `getActivePlayerIndex` returns 0. The function also returns 0 if the player is not found. This contradicts the NatSpec which states that 0 is returned when the player "is not active."

```solidity
function getActivePlayerIndex(address player) external view returns (uint256) {
    for (uint256 i = 0; i < players.length; i++) {
        if (players[i] == player) {
            return i;
        }
    }
    return 0; // @audit: same as a valid index
}
```

**Impact:** A player at index 0 may incorrectly believe they are not registered and attempt to re-enter, wasting gas.

**Recommended Mitigation:** Revert instead of returning 0 when not found, or return a sentinel value like `type(uint256).max`:

```diff
  function getActivePlayerIndex(address player) external view returns (uint256) {
      for (uint256 i = 0; i < players.length; i++) {
          if (players[i] == player) {
              return i;
          }
      }
-     return 0;
+     revert("PuppyRaffle: Player not found");
  }
```
