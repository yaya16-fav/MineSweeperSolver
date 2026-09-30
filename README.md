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

### 1. Tampilan awal permainan

<img width="300" alt="(Wǒ)Men-Sweeper" src="https://github.com/user-attachments/assets/92d036e7-1dd8-4f42-9695-e4911a3e9fc1" />

*Gambar 1. Papan Minesweeper 6×6 dengan 6 bom sebelum ada sel yang dibuka. Tombol AI SOLVE dan penghitung bom ada di panel atas, sedangkan petunjuk klik kiri (buka sel) dan klik kanan (bendera) ada di bawah papan.*


### 2. Jendela log AI

<img width="540" height="736" alt="Screenshot 2026-09-30 172013" src="https://github.com/user-attachments/assets/947bf4df-084c-47dd-9e11-d7fb81328c6a" />

*Gambar 2. Jendela log yang mencatat setiap langkah AI beserta alasannya. Tiap baris menunjukkan nomor langkah, aksi (OPEN atau FLAG), koordinat sel, jenis aturan yang dipakai, dan sel bernomor yang memicu aturan tersebut. Mode Manual (tombol Langkah Berikutnya) dan Mode Otomatis bisa dipilih dari jendela ini.*

### 3. Langkah pasti dan tebakan
<img width="502" height="152" alt="Screenshot 2026-09-30 172429" src="https://github.com/user-attachments/assets/aa44164b-94b6-4587-9d9b-5aa75357be4b" />
n.png)

*Gambar 3. Perbedaan dua jenis langkah AI. Sel dengan highlight **hijau** adalah langkah pasti hasil penalaran (aturan dasar atau subset method), sedangkan highlight **oranye** adalah tebakan yang dipakai saat tidak ada kesimpulan logis tersisa.*

### 4. Hasil akhir permainan

<img width="546" height="731" alt="Screenshot 2026-09-30 172135" src="https://github.com/user-attachments/assets/f6edb668-503a-4029-8096-a2f74613cba0" />


*Gambar 4a. Popup kemenangan setelah semua sel aman berhasil dibuka oleh AI. Jumlah langkah yang dibutuhkan ditampilkan di label status.*

<img width="543" height="727" alt="Screenshot 2026-09-30 172233" src="https://github.com/user-attachments/assets/d1c78632-ef0c-44d0-ae87-23b9744532a5" />

*Gambar 4b. Kondisi kalah saat AI membuka sel berbom, biasanya setelah harus menebak. Sel bom yang diinjak ditandai merah dan bom lainnya ditampilkan.*




## Referensi dan Sitasi

