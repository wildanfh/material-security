## Apa Itu Storage?

**Storage** adalah area penyimpanan permanen dalam smart contract yang dialokasikan untuk variabel state. Setiap variabel state yang dideklarasikan akan menempati **slot storage** tersendiri.

- Storage bersifat **persisten** – nilai tetap tersimpan di blockchain.
- Setiap slot storage berukuran 32 byte (256 bit).

## Variabel yang Tidak Menempati Storage

**Constants** dan **immutable** variabel **tidak** menggunakan slot storage. Keduanya disimpan langsung di dalam **bytecode** kontrak.

| Jenis Variabel | Menempati Storage? | Keterangan |
|----------------|---------------------|-------------|
| State variable biasa | ✅ Ya | Contoh: `uint256 public number1;` |
| `constant` | ❌ Tidak | Nilai ditentukan saat kompilasi, disimpan di bytecode. |
| `immutable` | ❌ Tidak | Nilai ditentukan di constructor, disimpan di bytecode. |

## Contoh Kode

```solidity
contract Counter {
    uint256 public number1;    // slot 0
    uint256 public number2;    // slot 1
    uint256 public constant number3 = 123;  // tidak pakai storage
    uint256 public number4;    // slot 2
}
```

## Melihat Storage dengan Foundry

Gunakan perintah `forge inspect` untuk melihat alokasi storage suatu kontrak.

```bash
forge inspect Counter storage
```

### Contoh Output

```json
"storage": [
    {
      "label": "number1",
      "slot": "0",
      "type": "t_uint256"
    },
    {
      "label": "number2",
      "slot": "1",
      "type": "t_uint256"
    },
    {
      "label": "number4",
      "slot": "2",
      "type": "t_uint256"
    }
]
```

> **Catatan:** `number3` tidak muncul dalam output karena merupakan constant.

## Poin Penting

- Setiap variabel state mendapat slot storage unik, dimulai dari slot 0, 1, 2, dst.
- Variabel `constant` dan `immutable` lebih hemat gas karena tidak perlu baca/tulis storage.
- Gunakan `forge inspect` untuk debugging dan memahami tata letak storage kontrak Anda.

## Prinsip Belajar

> *"Always experiment with code, because it's in the doing that we grasp the most complex concepts."*

Eksperimen langsung dengan kode dan perintah Foundry adalah cara terbaik untuk memahami storage secara mendalam.
