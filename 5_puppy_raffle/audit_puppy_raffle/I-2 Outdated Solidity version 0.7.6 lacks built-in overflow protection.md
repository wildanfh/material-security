### [I-2] Using an outdated Solidity version `0.7.6` is not recommended

**Description:** Solidity `0.7.6` lacks many improvements introduced in later versions, most critically the automatic overflow/underflow checks added in `0.8.0`. Using this version requires manual SafeMath usage or extra care with arithmetic.

**Recommended Mitigation:** Upgrade to at least `0.8.18`:

```diff
- pragma solidity ^0.7.6;
+ pragma solidity 0.8.18;
```

Note: upgrading will also require changing `uint64 totalFees` to `uint256 totalFees` and removing the unsafe cast.
