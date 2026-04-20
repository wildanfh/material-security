# Kapan Kita Selesai? (Time-Boxing dalam Audit)

## Pertanyaan Kunci

Setelah menyelesaikan tiga temuan dan menyusun laporan, muncul pertanyaan:  
**Apakah kita sudah selesai? Haruskah kita kembali melihat kode sekali lagi? Mungkin ada yang terlewat...**

Dalam situasi *live audit*, apa yang akan Anda lakukan?

---

## Jawaban: "Mungkin, tapi harus ada batasan"

> *"Honestly, we can always look at one more line of code. We can always further scrutinize a repo. At some point however, we have to say 'I'm done'."*

Tidak ada habisnya jika terus mencari. Kode bisa ditinjau terus-menerus tanpa batas. Pada titik tertentu, auditor harus **mengambil keputusan** bahwa pekerjaan sudah cukup.

---

## Konsep Time-Boxing (Batasan Waktu)

Dalam banyak situasi audit, terdapat **batasan waktu** (time-box) yang harus dipatuhi. Ini bisa berupa:

| Jenis Batasan | Penjelasan |
|---------------|------------|
| **Batasan keras (hard limit)** | Misalnya: kontrak audit menentukan durasi 2 minggu. Setelah waktu habis, audit selesai. |
| **Batasan yang ditetapkan sendiri** | Auditor menetapkan batas waktu untuk setiap fase atau untuk keseluruhan review, demi menjaga efisiensi dan fokus. |

> *"Time-boxing is a hard limit we impose on ourselves to assure we remain at our most efficient."*

---

## Mengapa Time-Boxing Penting?

1. **Mencegah perfeksionisme berlebihan** – tidak ada audit yang sempurna.
2. **Memastikan efisiensi** – dengan batasan waktu, auditor bekerja lebih terarah.
3. **Manajemen sumber daya** – waktu auditor terbatas, harus dialokasikan dengan bijak.
4. **Ekspektasi klien** – klien membutuhkan kepastian kapan laporan selesai.

---

## Strategi Time-Boxing (Sekilas)

Materi ini hanya menyebutkan bahwa strategi time-boxing akan dibahas lebih lanjut nanti. Beberapa pendekatan umum:

- Alokasikan waktu untuk **reconnaissance** (misal: 20% dari total waktu).
- Alokasikan waktu untuk **identifikasi kerentanan** (60%).
- Alokasikan waktu untuk **pelaporan** (20%).
- Tetapkan batas maksimum per kontrak atau per fitur.

> Detail strategi akan dibahas di pelajaran berikutnya.

---

## Kesimpulan

- **Tidak ada audit yang benar-benar "selesai"** – selalu ada baris kode lain yang bisa diperiksa.
- **Time-boxing adalah keharusan** – auditor harus menetapkan batasan waktu untuk tetap efisien.
- Pada titik tertentu, Anda harus berkata: *"I'm done."*
- Manajemen waktu adalah keterampilan kunci dalam security review profesional.

> *"Often a pressing situation comes down to time management and setting bounds on the time we spend on things."*