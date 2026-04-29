# Mishandling of ETH di `PuppyRaffle::withdrawFees`

## Kode Rentan

```solidity
require(address(this).balance == uint256(totalFees), "PuppyRaffle: There are currently players active!");
```

## Root Cause

Fungsi `withdrawFees` memerlukan saldo kontrak (`address(this).balance`) sama persis dengan `totalFees` agar penarikan dapat dilakukan. Pemeriksaan ini terlalu kaku dan tidak mempertimbangkan kemungkinan adanya ETH yang dikirim ke kontrak melalui cara lain (misal: `selfdestruct` atau transfer langsung).

## Severity

| Faktor | Penilaian | Alasan |
|--------|-----------|--------|
| **Impact** | Medium | Dapat membuat fee tidak bisa ditarik atau membuka celah griefing. |
| **Likelihood** | Low | Membutuhkan kondisi spesifik (misal: akuntansi fee rusak atau ada pengiriman ETH paksa). |

**Severity:** **Medium** (bisa juga Informational tergantung konteks)

## Dampak

- Jika akuntansi fee rusak (misal karena overflow atau unsafe casting), `totalFees` tidak akan sama dengan saldo kontrak → fee tidak bisa ditarik selamanya.
- Protokol rentan terhadap *griefing attack*: siapa pun dapat mengirim sejumlah kecil ETH ke kontrak, membuat saldo tidak cocok dan menghalangi penarikan fee.
- Pemeriksaan ini juga mengasumsikan tidak ada ETH yang masuk selain dari fee undian, yang tidak benar di blockchain publik.

## Rekomendasi Mitigasi

Hapus pemeriksaan keseimbangan tersebut. Cukup pastikan bahwa:

- Hanya `feeAddress` yang bisa menarik fee.
- Jumlah yang ditarik tidak melebihi `totalFees` yang tercatat.

```diff
- require(address(this).balance == uint256(totalFees), "PuppyRaffle: There are currently players active!");
+ require(totalFees > 0, "PuppyRaffle: No fees to withdraw");
+ uint256 amount = totalFees;
+ totalFees = 0;
+ (bool success, ) = msg.sender.call{value: amount}("");
+ require(success, "Transfer failed");
```

Dengan ini, `feeAddress` dapat menarik semua fee yang tercatat tanpa terganggu oleh ETH asing yang masuk ke kontrak.