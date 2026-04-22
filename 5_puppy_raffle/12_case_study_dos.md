# Case Study: DoS Attacks di Dunia Nyata

## Pendahuluan

Sekarang kita akan mempelajari dua **real-world DoS vulnerabilities** yang ditemukan selama security review. Case studies ini dishare oleh **Owen** – Founder dari **Guardian Audits**.

Guardian Audits telah menemukan ratusan vulnerabilities dan membuat Web3 lebih aman.

---

## Case Study 1: Bridges Exchange

### Vulnerability

**Unbounded for-loop** di fungsi `distributeDividends` menyebabkan risiko DoS.

### Kode Vulnerability

```solidity
function distributeDividends(uint amount) public payable lock {
    require(amount == msg.value, "don't cheat");
    uint length = users.length;
    amount = amount.mul(magnitude);
    for (uint i; i < length; i++) {
        if (users[i] != address(0)) {
            UserInfo storage user = userInfo[users[i]];
            user.rewards += (amount.mul(IERC20(address(this).balanceOf(users[i])).div(totalSupply.sub(MINIMUM_LIQUIDITY))));
        }
    }
}
```

### Masalah

- Loop melalui `users[]` array – panjang tak terbatas
- Jika array sangat panjang, gas exceeds block limit
- Fungsi menjadi tidak bisa dipanggil

### Attack Vector

Cari tahu bagaimana `users[]` bisa diperbesar:

```solidity
function mint(address to) external lock returns (uint liquidity) {
    // ...
    if (IERC20(address(this).balanceOf(to) == 0)) {
        users.push(to);
    }
}
```

### Attack Scenario

1. Attacker generate banyak wallet addresses baru
2. Mint jumlah minimal token untuk cetak address
3. Array `users[]` membengkak
4. `distributeDividends` tidak bisa dipanggil (DoS)

### Resolution

Refactor agar for-loop tidak diperlukan.

---

## Case Study 2: GMX V2

### Vulnerability

**Boolean flag** `shouldUnwrapNativeToken` memungkinkan posisi yang tidak bisa di-liquidate.

### Kode Vulnerability

Dalam `DecreaseOrderUtils.sol` – fungsi `processOrder`:

```solidity
function transferNativeToken(DataStore dataStore, address receiver, uint256 amount) internal {
    if (amount == 0) { return; }

    uint256 gasLimit = dataStore.getUint(keys.NATIVE_TOKEN_TRANSFER_GAS_LIMIT);

    (bool success, bytes memory data) = payable(receiver).call{ value: amount, gas: gasLimit }("");

    if (success) { return; }

    string memory reason = string(abi.encode(data));
    emit NativeTokenTransferReverted(reason);

    revert NativeTokenTransferError(receiver, amount);
}
```

### Masalah

1. Jika `receiver` adalah contract yang tidak bisa terima ETH → reverts
2. Liquidation/ADL order gagal
3. Protocol kehilangan fungsi kritis

### Attack Scenario

1. Buat contract yang tidak bisa terima native token
2. Buka posisi dengan `shouldUnwrapNativeToken = true`
3. Ketika di-liquidate → transaction revert setiap saat
4. Posisi tidak bisa di-liquidate = stuck funds

### Additional Issue

- `gasLimit` juga bisa diexploit
- Contract dengan receive function boros gas → reverts

---

## Pattern DoS yang Perlu Diperhatikan

### 1. For-Loops

Selalu tanya:

| Pertanyaan | Implikasi |
|------------|----------|
| Apakah iterable entity bounded? | Jika tidak → risk DoS |
| Bisakah user menambah arbitrary items? | Attack vector |
| Berapa biaya user untuk add? | Attack feasibility |

### 2. External Calls

- Transfer ETH ke contract lain
- Calling third-party contracts
- Evaluasi cara call bisa fail

---

## Jenis DoS Attacks

| Tipe | Penyebab | Contoh |
|------|----------|--------|
| **Unbounded Loop** | for-loop tanpa batas | Bridges Exchange |
| **Unreceiveable Token** | receiver tidak bisa terima | GMX V2 |
| **Gas Limit** | external call boros gas | GMX V2 |
| **Block Gas Limit** | array terlalu besar | PuppyRaffle |

---

## Kesimpulan

- **DoS** = denial fungsi protokol
- Bisa multiple sources:
  1. **Unbounded for-loops**
  2. **External calls** yang bisa gagal
- Selalu evaluasi:
  - Apakah loop bounded?
  - Apakah external call bisa gagal?
- Severity: bisa sangat critical jika fungsi kritis terdampak