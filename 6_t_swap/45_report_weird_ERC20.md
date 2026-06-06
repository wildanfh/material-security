# Ringkasan Materi: Weird ERC20s dan Dampaknya pada TSwap

## Pendahuluan

**Weird ERC20** adalah token yang mengimplementasikan standar ERC20 tetapi dengan perilaku tidak standar (menyimpang dari spesifikasi). Perilaku ini dapat menyebabkan kerentanan serius pada protokol DeFi yang mengintegrasikan token tersebut tanpa antisipasi yang tepat.

Sumber daya penting: [GitHub Repo Weird ERC20 oleh d-xo](https://github.com/d-xo/weird-erc20)

---

## Contoh Weird ERC20 yang Relevan dengan TSwap

| Jenis | Perilaku | Risiko pada TSwap |
|-------|----------|-------------------|
| **Fee-on-transfer** | Memotong sebagian kecil nilai setiap transfer (misal 10% setiap 10 transaksi). | Melanggar invariant `x * y = k` karena saldo pool berubah tanpa melalui mekanisme swap yang sah. |
| **Rebasing tokens** | Saldo pengguna berubah secara otomatis (biasanya periodik) tanpa transfer eksplisit. | Protokol yang melacak saldo berdasarkan jumlah token dapat kehilangan akurasi. |
| **ERC-777** | Mengizinkan *hooks* (callback) yang dapat memanggil kembali fungsi kontrak. | Potensi *reentrancy* dan pelanggaran invariant. |

---

## Kerentanan di TSwap Terkait Weird ERC20

Dalam latihan stateful fuzzing sebelumnya, kita menemukan bahwa token `YieldERC20` (mock token dengan fee-on-transfer) menyebabkan kegagalan invariant test:

```solidity
function statefulFuzz_testInvariantBreakHandler() public {
    vm.startPrank(owner);
    handlerStatefulFuzzCatches.withdrawToken(mockUSDC);
    handlerStatefulFuzzCatches.withdrawToken(yieldERC20);
    vm.stopPrank();

    assert(mockUSDC.balanceOf(address(handlerStatefulFuzzCatches)) == 0);
    assert(yieldERC20.balanceOf(address(handlerStatefulFuzzCatches)) == 0);
    assert(mockUSDC.balanceOf(owner) == startingAmount);
    assert(yieldERC20.balanceOf(owner) == startingAmount);
}
```

Token `YieldERC20` mengirim 10% dari nilai transaksi sebagai fee setiap 10 transfer. Ini menyebabkan kontrak `HandlerStatefulFuzzCatches` tidak dapat menarik seluruh saldo karena kekurangan token untuk membayar fee – analog dengan apa yang akan terjadi di TSwap jika berintegrasi dengan token serupa.

---

## Kasus Nyata: Audit Uniswap V1

Dalam audit Uniswap V1 oleh Consensys, ditemukan kerentanan serupa di mana token ERC-777 memungkinkan *reentrancy* yang dapat mencuri likuiditas dari pool.  
[Baca laporan lengkapnya di sini](https://github.com/Consensys/Uniswap-audit-report-2018-12?tab=readme-ov-file#31-liquidity-pool-can-be-stolen-in-some-tokens-eg-erc-777-29)

---

## Dampak pada TSwap

- **Pelanggaran invariant** – Token fee-on-transfer atau rebasing mengubah saldo pool tanpa melalui `_swap` yang menghormati `x * y = k`.
- **Potensi pengurasan dana** – Penyerang dapat menggunakan token aneh untuk mengganggu keseimbangan pool dan mengambil keuntungan.
- **Kegagalan fungsi** – `withdraw` atau `swap` dapat revert karena ketidakcocokan jumlah token.

---

## Rekomendasi

- **Whitelist token** yang didukung – hanya izinkan token dengan perilaku yang telah diverifikasi.
- **Gunakan `SafeERC20`** (sudah digunakan TSwap) – membantu menghadapi token yang tidak mengembalikan boolean, tetapi tidak mengatasi fee-on-transfer.
- **Uji dengan mock token aneh** – buat token dengan fee, rebasing, dan callback untuk menguji ketahanan protokol.
- **Dokumentasikan risiko** – dalam laporan audit, cantumkan bahwa protokol tidak kompatibel dengan token fee-on-transfer atau rebasing tanpa modifikasi.

---

## Tantangan untuk Auditor

Buatlah laporan temuan untuk kerentanan ini dengan judul:

### [M-3] Rebase, fee-on-transfer, and ERC-777 tokens break protocol invariant

Gunakan mock token yang sudah dibuat dalam kursus (`YieldERC20`) dan test yang sudah ada untuk menyusun **Proof of Concept**. Tulis deskripsi, dampak, PoC, dan rekomendasi mitigasi.