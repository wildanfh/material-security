### [G-1] Unchanged state variables should be declared `constant` or `immutable`

**Description:** Several variables are initialized once and never changed. Reading from storage costs ~2100 gas per `SLOAD`, while `constant`/`immutable` values are embedded in bytecode and cost only ~3 gas to read.

Instances:
- `PuppyRaffle::raffleDuration` — set in constructor, never changed → `immutable`
- `PuppyRaffle::commonImageUri` — static string → `constant`
- `PuppyRaffle::rareImageUri` — static string → `constant`
- `PuppyRaffle::legendaryImageUri` — static string → `constant`

**Recommended Mitigation:**

```diff
- string private commonImageUri = "ipfs://...";
+ string private constant commonImageUri = "ipfs://...";

- uint256 public raffleDuration;
+ uint256 public immutable raffleDuration;
```
