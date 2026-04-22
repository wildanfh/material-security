# Recon: Membaca Kode Sumber

## Pendahuluan

Setelah memahami dokumentasi, langkah selanjutnya dalam **Recon** adalah membaca kode sumber. Fokus awal pada **main entry point** – fungsi yang dapat dipanggil dari luar (external/public) yang berpotensi menjadi vektor serangan.

---

## Menemukan Entry Point

Gunakan **Solidity: Metrics** atau perintah berikut di Foundry:

```bash
forge inspect PuppyRaffle methods
```

Fokus pada fungsi yang:
- `public` atau `external`
- `modify state` (mengubah storage)
- `payable` (menerima ETH)

---

## Analisis: Fungsi enterRaffle

Mari analyze fungsi utama `enterRaffle`:

```solidity
function enterRaffle(address[] memory newPlayers) public payable {
    require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
    for (uint256 i = 0; i < newPlayers.length; i++) {
        players.push(newPlayers[i]);
    }

    // Check for duplicates
    for (uint256 i = 0; i < players.length - 1; i++) {
        for (uint256 j = i + 1; j < players.length; j++) {
            require(players[i] != players[j], "PuppyRaffle: Duplicate player");
        }
    }
    emit RaffleEnter(newPlayers);
}
```

### Pertanyaan dari NatSpec

Dari komentar NatSpec, muncul beberapa pertanyaan:

1. **"What's meant by # of players?"**
   - Apakah `newPlayers.length` dihitung per array yang passed, atau ada batasan lainnya?

2. **"How does the function prevent duplicate entrants?"**
   - Loop check duplikat berjalan setelah players ditambahkan, apakah ada off-by-one error?

---

## Membuat Catatan Audit

Selama analisis, catat pertanyaan dan temuan di `notes.md`:

```markdown
## Informational

`PuppyRaffle::entranceFee` is immutable and should follow a more clear naming convention
    ie. `i_entranceFee` or `ENTRANCE_FEE`
```

Atau gunakan anotasi inline:

```solidity
/// @audit question: Bagaimana jika array kosong passed?
/// @audit issue: Check duplikat berjalan setelah push - perlu verifikasi
```

---

## Naming Convention Issue

Satu hal yang perlu diperhatikan – naming convention tidak konsisten:

- `entranceFee` adalah variabel immutable
- Seharusnya: `i_entranceFee` atau `ENTRANCE_FEE`

Ini adalah **Informational** finding – tidak rentan tapi perlu diperbaiki untuk klaritas kode.

---

## Navigasi di VS Code

Tips navigasi cepat:

| OS | Shortcut | Fungsi |
|----|----------|--------|
| Windows | `Alt + Left Arrow` | Previous cursor |
| Windows | `Alt + Right Arrow` | Next cursor |
| Mac | `Control + '-'` | Previous cursor |
| Mac | `Control + Shift + '-'` | Next cursor |

---

## Urutan Analisis yang Disarankan

1. **Entry point** – fungsi public/external
2. **State variables** – siapa yang bisa modify
3. **Access control** – modifier seperti `onlyOwner`
4. **External calls** – interaksi dengan kontrak lain
5. **Business logic** – alur utama protokol

---

## Kesimpulan

- Mulai dari **main entry point** – fungsi yang dapat dipanggil publik.
- Fokus pada fungsi yang **modify state** dan **payable**.
- Catat pertanyaan dari NatSpec di `notes.md`.
- Identifikasi **Informational** findings seperti naming convention.
- Navigasi cepat dengan shortcut keyboard.