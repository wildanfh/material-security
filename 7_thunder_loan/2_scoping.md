# Ringkasan Materi: Phase 1 – Scoping (ThunderLoan)

## Pendahuluan

Langkah pertama dalam audit ThunderLoan adalah **scoping** – mengumpulkan konteks dan pemahaman sebanyak mungkin tentang protokol. Seperti biasa, kita mulai dengan membaca README.

---

## Scope dan Detail Audit

Protokol telah menyediakan detail audit yang jelas:

- **Commit Hash:** `8803f851f6b37e99eab2e94b4690c8b70e26b3f6` (untuk kursus ini, cukup gunakan branch `main`).
- **Lingkup (In Scope):**
  ```
  #-- interfaces
  |   #-- IFlashLoanReceiver.sol
  |   #-- IPoolFactory.sol
  |   #-- ITSwapPool.sol
  |   #-- IThunderLoan.sol
  #-- protocol
  |   #-- AssetToken.sol
  |   #-- OracleUpgradeable.sol
  |   #-- ThunderLoan.sol
  #-- upgradedProtocol
      #-- ThunderLoanUpgraded.sol
  ```
- **Versi Solidity:** 0.8.20
- **Rantai target:** Ethereum
- **Token ERC20 yang didukung:** USDC, DAI, LINK, WETH

> Mengetahui token yang digunakan membantu auditor mengesampingkan risiko `Weird ERC20s` yang tidak relevan.

---

## Roles (Peran)

Dokumentasi juga menjelaskan peran dalam protokol:

| Peran | Deskripsi |
|-------|-----------|
| **Owner** | Pemilik protokol yang memiliki hak untuk meningkatkan (upgrade) implementasi. |
| **Liquidity Provider (LP)** | Pengguna yang menyetor aset ke protokol untuk mendapatkan bunga. |
| **User** | Pengguna yang mengambil flash loan dari protokol. |

---

## Known Issues

Untuk pertama kalinya, README mencantumkan **Known Issues** – kerentanan yang sudah diketahui tim protokol. Auditor perlu kembali ke bagian ini nanti untuk memastikan tidak ada temuan duplikat.

---

## Praktik Testing

Sekilas melihat repo proyek menunjukkan:
- **Praktik baik:** Ada konfigurasi `Slither` dan `Aderyn` (terlihat di `Makefile`).
- **Praktik kurang:** Tidak ada invariant test suite (perlu dibuat oleh auditor).

---

## Menilai Ukuran dan Kompleksitas dengan Solidity Metrics

Klik kanan pada folder `src` di VS Code → pilih `Solidity: Metrics`.

**Hasil:**
- **nSLOC (source lines of code):** 391
- **Complexity:** 327

ThunderLoan adalah **codebase terbesar** yang kita hadapi sejauh ini. Sebagian besar logika dan kompleksitas berada di:
- `ThunderLoan.sol`
- `ThunderLoanUpgraded.sol`

> Mengecek metrik ini membantu memperkirakan durasi audit berdasarkan ukuran dan tingkat keterampilan saat ini.

---

## Kesimpulan

- Scoping memberikan fondasi penting: scope, roles, token, rantai, dan known issues.
- Ukuran codebase (nSLOC 391) menandakan audit akan lebih intensif.
- Langkah selanjutnya: **Phase 2 – Reconnaissance** (pengintaian).