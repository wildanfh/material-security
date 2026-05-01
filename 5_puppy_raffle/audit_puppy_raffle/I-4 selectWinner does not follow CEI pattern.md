### [I-4] `PuppyRaffle::selectWinner` does not follow CEI (Checks-Effects-Interactions) pattern

**Description:** In `selectWinner`, the ETH transfer to the winner (`winner.call{value: prizePool}`) happens before `_safeMint`. While no reentrancy exploit was identified here, the pattern violates best practice and could become exploitable if the contract is extended.

```solidity
(bool success,) = winner.call{value: prizePool}("");  // Interaction
require(success, "...");
_safeMint(winner, tokenId);                            // Effect
```

**Recommended Mitigation:** Move `_safeMint` before the external call:

```diff
+   _safeMint(winner, tokenId);
    (bool success,) = winner.call{value: prizePool}("");
    require(success, "PuppyRaffle: Failed to send prize pool to winner");
-   _safeMint(winner, tokenId);
```
