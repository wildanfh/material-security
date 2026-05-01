### [I-1] Floating pragma `^0.7.6` allows compilation with unintended Solidity versions

**Description:** The contract uses `pragma solidity ^0.7.6`, which allows compilation with any `0.7.x` version above 0.7.6. Different compiler versions may behave differently or introduce subtle bugs.

**Recommended Mitigation:** Lock to a specific version:

```diff
- pragma solidity ^0.7.6;
+ pragma solidity 0.7.6;
```
