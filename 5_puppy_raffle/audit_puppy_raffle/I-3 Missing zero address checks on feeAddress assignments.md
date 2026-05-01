### [I-3] Missing checks for `address(0)` when assigning values to address state variables

**Description:** Multiple places assign to address state variables without checking for `address(0)`. Setting `feeAddress` to the zero address would silently burn all fees.

Locations:
- `constructor` — initializes `feeAddress`
- `changeFeeAddress` — updates `feeAddress`
- `selectWinner` — sets `previousWinner`

**Recommended Mitigation:** Add zero-address guards:

```diff
  function changeFeeAddress(address newFeeAddress) external onlyOwner {
+     require(newFeeAddress != address(0), "PuppyRaffle: Fee address cannot be zero");
      feeAddress = newFeeAddress;
      emit FeeAddressChanged(newFeeAddress);
  }
```
