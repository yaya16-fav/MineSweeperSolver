# Implementasi Knowledge-Based System untuk Menyelesaikan Permainan Minesweeper dengan Forward Chaining dan Constraint Propagation
Solver Minesweeper 6×6 berbasis **Knowledge-Based System**, lengkap dengan GUI yang menampilkan penjelasan setiap langkah algoritma AI.

Proyek Akhir Mata Kuliah Artificial Intelligence, DTETI Universitas Gadjah Mada.

Anggota Tim: Nisa Faizatul Azkiya, Fadya Aviqa, Muhammad Arsya Gifary

## Ringkasan
Game Minesweeper dimainkan oleh AI yang menalar dari angka petunjuk di papan. Setiap langkah AI (buka sel / pasang _flag_) dicatat di _pop-up log_ beserta alasan logisnya, sehingga rantai inferensinya bisa ditelusuri.

## Fitur

- Game engine Minesweeper 6×6 dengan 6 bom, flood-fill otomatis, dan  klik pertama dipastikan aman (sel pertama dan semua tetangganya bebas bom).
- Mode main manual (klik kiri = buka, klik kanan = bendera).
- Tombol **AI SOLVE** dengan dua mode:
  - **Manual**: AI jalan satu langkah setiap tombol "Langkah Berikutnya" ditekan.
  - **Otomatis**: AI jalan sendiri dengan jeda antar langkah, bisa di-pause.
- _Pop-up log_ berisi alasan tiap langkah, misalnya sel angka mana yang memicu aturan.
- Highlight sel: **hijau** = langkah pasti (hasil penalaran), **oranye** = tebakan.
- Popup Game Over / Menang, tombol Main Lagi, dan background hutan-tambang yang digenerate prosedural dengan Pillow.

## Knowledge-Based System

**Knowledge base.** Fakta berasal dari papan yang sudah terlihat: sel terbuka berisi angka *n* berarti tepat *n* dari 8 tetangganya adalah bom. Untuk tiap sel bernomor dihitung:

- `hidden` = tetangga yang masih tertutup
- `remaining` = `n` − jumlah bendera di sekitarnya

**Aturan inferensi (forward chaining).** Setiap kali AI dipanggil, seluruh papan dipindai dan aturan diterapkan berurutan:

| # | Kondisi | Kesimpulan |
|---|---------|------------|
| R1 | `remaining == 0` | semua `hidden` **aman** → OPEN |
| R2 | `remaining == len(hidden)` | semua `hidden` **bom** → FLAG |
| R3 (subset method) | himpunan constraint A ⊂ B, dengan selisih bom `k_B − k_A` | jika selisih 0 → sel `B \ A` aman; jika selisih = `|B \ A|` → sel `B \ A` bom |

R3 membandingkan pasangan constraint antar-sel bernomor. Constraint baru hasil selisih ikut dipakai di putaran berikutnya (maksimal 10 putaran). R3 hanya dijalankan kalau R1 dan R2 tidak menghasilkan aksi.

**Fallback.** Kalau tidak ada aturan yang menghasilkan kesimpulan, AI menebak secara acak, dengan prioritas sel tertutup yang bersebelahan dengan sel terbuka (perbatasan).

**Langkah pertama.** AI selalu membuka sel `(0, 0)`.

## Arsitektur

```
minesweeper-kbs/
├── minesweeper_ai.py  
├── minesweeper_gui.py   
├── requirements.txt
└── README.md
```

- `MinesweeperAI` (`minesweeper_ai.py`) hanya punya representasi papan sendiri. Ia tidak pernah menyentuh game engine.
- Antarmuka: `get_action()` mengembalikan `('open'|'flag', r, c)`, lalu simulator melaporkan hasilnya lewat `report_open(r, c, value)` dan `report_flag(r, c)`. Untuk cascade, `report_open` dipanggil untuk **setiap** sel yang terbuka.
- `TrackedMinesweeperAI` (di GUI) adalah subclass tipis yang hanya mencatat sumber langkah (`basic` / `csp` / `guess`) untuk keperluan log dan warna highlight. Algoritmanya tidak diubah.

## Cara Menjalankan

Kebutuhan: Python 3.9+ dengan Tkinter (bawaan Python di Windows/macOS), dan Pillow.

```bash
git clone <URL repo ini>
cd minesweeper-kbs
pip install -r requirements.txt
python minesweeper_gui.py
```

`requirements.txt`:

```
pillow
```

Catatan: font yang dipakai adalah Segoe UI / Segoe UI Emoji (tampilan paling pas di Windows). Di OS lain Tkinter memakai font pengganti, jadi tampilan bisa sedikit berbeda.

## Cara Pakai

1. Jalankan aplikasi, lalu main manual atau tekan **🤖 AI SOLVE**.
2. Di jendela log, pilih mode **Manual** atau **Otomatis**.
3. Perhatikan log: tiap langkah menampilkan jenis aturan (aturan dasar / subset / tebakan) dan sel pemicunya.
4. Tekan **Hentikan AI** untuk berhenti, atau tombol restart untuk papan baru.

## Hasil dan Demo

<Tempel screenshot: papan awal, jendela log, langkah pasti vs tebakan, kondisi menang/kalah>

<Opsional: tabel hasil eksperimen, misalnya jumlah run, win rate, dan rata-rata langkah, isi dari data yang benar-benar kamu jalankan>

Demo berupa aplikasi desktop, jadi tidak ada link live/deployed.

## Keterbatasan

- Ukuran papan dan jumlah bom tetap (6×6, 6 bom).
- Tebakan bersifat acak di antara sel perbatasan, tidak berbasis probabilitas, jadi AI bisa kalah saat kondisi benar-benar buntu secara logis.
- Subset method dibatasi 10 putaran dan hanya membandingkan pasangan constraint, jadi tidak menangkap semua pola inferensi lanjutan.
- Solver belum memakai informasi total jumlah bom (`total_bombs`) untuk inferensi.
- <Tambahkan keterbatasan lain dari hasil pengujian kamu>

## Latar Belakang Kasus Nyata

<Isi sendiri: kenapa Minesweeper adalah kasus nyata, dan kenapa KBS + forward chaining dipilih dibanding algoritma lain>

## Referensi dan Sitasi

- <Materi kuliah / Lecture 4: Knowledge-Based Systems>
- <Sumber aturan Minesweeper / literatur solver yang dipakai>
- <Kode atau aset yang dipakai ulang, kalau ada>
