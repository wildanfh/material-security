### [G-2] `players.length` accessed repeatedly in loop should be cached in a local variable

**Description:** In `enterRaffle`, `players.length` is read from storage on every iteration of both the outer and inner loops. Each storage read costs ~100 gas minimum.

```solidity
for (uint256 i = 0; i < players.length - 1; i++) {       // SLOAD each iteration
    for (uint256 j = i + 1; j < players.length; j++) {   // SLOAD each iteration
```

**Recommended Mitigation:** Cache the length before the loops:

```diff
+ uint256 playersLength = players.length;
- for (uint256 i = 0; i < players.length - 1; i++) {
+ for (uint256 i = 0; i < playersLength - 1; i++) {
-     for (uint256 j = i + 1; j < players.length; j++) {
+     for (uint256 j = i + 1; j < playersLength; j++) {
```

Note: the underlying O(n²) loop is a more fundamental issue (see M-1). Caching helps but doesn't fix the core problem.
