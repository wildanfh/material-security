# Reentrancy: Cross-Function Accounting Corruption

**Source:** Notional Finance v4 — AbstractYieldStrategy bug report  
**Severity:** Critical  
**Pattern:** CEI violation → double deduction → frozen funds

---

## Bug Summary

`redeemNative()` → `_redeemShares()` → `_executeTrade()` melakukan external call (Uniswap swap) di tengah `_burnShares()`. Malicious token di swap path reenter via `transfer()` dan panggil `initiateWithdraw()`, menyebabkan double deduction pada `s_yieldTokenBalance`.

---

## Code Pattern (Reconstructed from Report)

```solidity
// VULNERABLE
function _burnShares(...) {
    uint256 yieldTokensBefore = ERC20(yieldToken).balanceOf(address(this)); // snapshot

    _executeTrade();  // ← external call (Uniswap swap) ← reentry masuk sini
                      // initiateWithdraw() jalan → s_yieldTokenBalance -= N
                      // yieldTokensAfter sudah reflect N yang dipindahkan

    uint256 yieldTokensAfter = ERC20(yieldToken).balanceOf(address(this));
    uint256 yieldTokensRedeemed = yieldTokensBefore - yieldTokensAfter; // ← stale yieldTokensBefore
    s_yieldTokenBalance -= yieldTokensRedeemed; // ← double deduction N
}
```

```solidity
// SAFE (CEI)
function _burnShares(...) {
    uint256 yieldTokensBefore = ERC20(yieldToken).balanceOf(address(this));
    s_yieldTokenBalance -= expectedDelta; // Effect DULU sebelum external call
    _executeTrade();                      // Interaction terakhir
}
```

---

## Attack Flow

```
Vault                    MaliciousToken           LendingRouter
  |                           |                        |
  |── redeemNative() ────────>|                        |
  |── _burnShares()           |                        |
  |   yieldTokensBefore = X   |                        |
  |── _executeTrade() ───────>|                        |
  |                    transfer() ──────────────────>  |
  |                           |    initiateWithdraw()  |
  |                           |<── move N tokens ──────|
  |                           |    s_yieldTokenBalance -= N
  |<── return ────────────────|                        |
  |   yieldTokensAfter = X - N (sudah include reentrancy)
  |   delta = X - (X-N) = N                            |
  |   s_yieldTokenBalance -= N  ← double deduct!       |
```

**Net effect:** `s_yieldTokenBalance` berkurang 2N, actual balance berkurang N → N tokens frozen.

---

## Kenapa `initiateWithdraw()` Berhasil di Dalam Reentrancy

Attacker pre-approve malicious token sebelum attack:

```solidity
lendingRouter.setApproval(maliciousToken, true);
```

Saat `transfer()` reenter dan panggil `initiateWithdraw(attacker, vault, "")`, router melihat approval valid → eksekusi berhasil.

**Lesson:** reentrancy tidak harus self-reentrant. Attacker pisahkan *siapa yang approve* vs *siapa yang reenter*.

---

## Violated Invariant

> Setelah `_burnShares()` selesai: `s_yieldTokenBalance == ERC20(yieldToken).balanceOf(vault)`

Cara latih: untuk setiap fungsi yang dibaca, tanya — *"state apa yang harus konsisten sebelum dan sesudah fungsi ini?"*

---

## Impact Cascade

```
s_yieldTokenBalance < actual balance
    → share price distorted (per-user)
    → health factor drop
    → liquidation triggered
    → repeated exploitation possible
    → full vault fund freezing
```

---

## `collectFees()` — Variant dengan Impact Lebih Kecil

Mekanisme sama: `collectFees()` juga kurangi `s_yieldTokenBalance`. Tapi impact lebih kecil karena fee collection bergerak dengan jumlah token yang terbatas → N kecil → double deduction kecil.

---

## Fix Analysis

| Fix | Proteksi | Kekuatan |
|-----|----------|----------|
| Validasi path (2-hop only) | blok vektor ini saja | Symptomatic |
| `nonReentrant` di semua entry point | blok semua reentrancy | Structural |

**Rule:** fix structural > fix symptomatic. Ideal: keduanya (defense in depth).

Path validation bisa dibypass via ERC777 callback, fee-on-transfer token, atau callback lain yang lolos validasi path.

---

## Grep Patterns — Cara Temukan di Codebase Baru

```bash
# Snapshot sebelum external call
grep -n "balanceOf" contracts/ -r | grep -i "before\|snapshot\|pre"

# External call yang bisa trigger reentry
grep -n "\.transfer\|\.call\|\.swap" contracts/ -r

# Fungsi publik/external tanpa nonReentrant
grep -rn "function.*external\|function.*public" contracts/ | \
  grep -v "nonReentrant\|view\|pure"
```

---

## Pattern Library Entry

```
PATTERN : Cross-function reentrancy via external swap
ROOT    : CEI violation — snapshot sebelum, state update setelah external call
GREP    : balanceOf.*before + external call + state -= delta (same function)
IMPACT  : accounting corruption → frozen funds → liquidation cascade
FIX     : nonReentrant + move state update BEFORE external call
ANALOG  : ERC777 callback reentrancy, fee-on-transfer token exploit
```

---

## Drill: 3 Pertanyaan Wajib Bisa Dijawab Tanpa Lihat Report

1. Kenapa `yieldTokensBefore` stale saat `_burnShares()` hitung delta? *(nilai mana yang stale, kenapa)*
2. Kenapa `initiateWithdraw()` di dalam reentrancy bisa berhasil?
3. Fix mana yang lebih kuat — `nonReentrant` atau validasi path Uniswap? Kenapa?

---

## Meta-Lesson: Cara Belajar dari Report

1. **Baca finding** → tulis hipotesis sendiri DULU sebelum lihat attack path
2. **Decompose 3 layer:** primitif → mekanisme → efek
3. **Cari violated invariant** — bukan cuma label "reentrancy"
4. **Gambar flow** sebelum tulis PoC
5. **Bedakan:** *"saya lihat ini di kode"* vs *"saya tebak ini"* — di audit nyata, tebak = false positive
