# SELARAS — Deskripsi Aplikasi & Panduan Gaya

**SELARAS** = **S**istem **E**lektronik **L**ayanan **A**dministrasi Persediaan dan **A**set.
Berkas utama: `siperdis-bmn (3).html` (satu file HTML mandiri: HTML + CSS + JavaScript, tanpa build step). Logo: `selaras-logo.svg`. Judul tab: `Menu - Selaras`.

## 1. Ringkasan

SELARAS adalah aplikasi web (desktop dan mobile) untuk mengelola **permintaan dan distribusi barang BMN** (persediaan dan aset), **penanganan barang rusak**, **inventaris per lokasi**, dan **pengguna** di **BKK Kelas I Tarakan** (Kementerian Kesehatan RI, Ditjen Penanggulangan Penyakit).

Alur permintaan mengikuti administrasi surat dinas:

1. Penanggung Jawab memilih barang dari katalog (beranda), lalu mengajukan **Nota Dinas (ND)**. Nomor urut ND diisi manual.
2. Petugas BMN memproses permintaan, menerbitkan **SBBK** (Surat Bukti Barang Keluar), dan mengemas barang.
3. Petugas BMN mengirim barang dan menerbitkan **BAST** (hanya memuat barang berjenis aset), dengan foto lampiran.
4. Penanggung Jawab mengonfirmasi barang diterima.
5. Semua surat dapat dicetak atau diekspor ke PDF.

Data disimpan di sisi klien (objek `DB` di localStorage kunci `siperdis_bmn_v1`, sesi `siperdis_bmn_sess`; `seed()` untuk data awal, `migrate()` untuk data lama).

## 2. Peran & Hak Akses

Hak akses terpusat di `CAN`, `ACT_PERM`, `ROUTE_PERM`, dan `MATRIX`. Akses yang tidak diizinkan ditolak walau alamat dibuka langsung.

| Peran | Kode | Tempat & tugas |
|---|---|---|
| Penanggung Jawab (PJ) | `pj` (`buat`, `konfirmasi`, `lapor`) | Berada di wilayah kerja atau ruangan kantor induk. Membuat ND, melaporkan barang rusak, memperbaiki sendiri atau mengirim ke induk, dan mengonfirmasi penerimaan. Hanya melihat data miliknya. |
| Petugas BMN | `bmn` (`proses`, `master`, `semua`, `rusak`, `lokasi`) | Berada di kantor induk. Memproses permintaan, SBBK/BAST, mengelola master barang dan lokasi, serta menindaklanjuti barang rusak. Melihat semua data. |
| Admin | `admin` (`kelola`, `semua`, `lokasi`) | Administrator sistem: pengguna, lokasi, hak akses, pengaturan. Pada permintaan dan barang rusak hanya melihat. |

Pemisahan tugas: yang mengajukan tidak memproses, dan Admin tidak ikut memproses.

## 3. Struktur Lokasi

Susunan: **Kantor / Wilayah kerja → Lantai → Ruangan → Penanggung Jawab** (`DB.lokasi`).

- Satu lokasi berjenis `induk` (kantor induk) atau `wilayah` (wilayah kerja), dan memiliki 1 sampai 3 lantai (bisa ditambah).
- Tiap lantai memiliki ruangan.
- **Wilayah kerja:** penanggung jawab melekat pada wilayah (bukan ruangan).
- **Kantor induk:** penanggung jawab melekat pada ruangan.
- Pengguna menyimpan `lokasiId` dan `ruangId`. Teks `unit` diturunkan otomatis (`lokUnit`, `syncUnits`) dan dipakai oleh ND, inventaris, dan barang rusak.
- Lokasi, lantai, atau ruangan yang masih dipakai penanggung jawab tidak dapat dihapus.
- Dikelola di halaman Lokasi oleh Petugas BMN dan Admin; pada form pengguna lokasi dipilih lewat satu pilihan "Lokasi / Ruangan".

## 4. Status

**Permintaan:** `diajukan` → `diproses` → `dikemas` → `dikirim` → `selesai`; jalur lain `ditolak`.

**Barang rusak:** PJ melapor dengan kategori rusak (ringan, sedang, berat) dan tindakan (perbaiki atau minta pengganti).
- Perbaiki sendiri → kirim bukti perbaikan → Petugas memverifikasi.
- Kirim ke induk → Petugas memeriksa. Bisa diperbaiki: diperbaiki lalu dikirim kembali. Tidak bisa: dikirim pengganti.
- Minta pengganti → Petugas mengirim pengganti, barang rusak dikirim ke induk.
- Diakhiri konfirmasi penerimaan. Jalur lain: ditolak.

## 5. Halaman & Rute (hash routing)

| Rute | Halaman | Akses |
|---|---|---|
| `#/` | Beranda (PJ: katalog gaya e-commerce; lainnya: dashboard) | Semua |
| `#/permintaan` , `#/permintaan/{id}` | Daftar dan detail permintaan, timeline, cetak surat | Semua (PJ hanya miliknya) |
| `#/baru` | Buat Nota Dinas | PJ |
| `#/rusak` | Barang Rusak: laporan (PJ) dan tindak lanjut (Petugas) | Semua |
| `#/inventaris` | Aset & Persediaan (PJ: wilayahnya; Petugas/Admin: kartu per wilayah lalu tabel) | Semua |
| `#/master` | Master Barang (barang dan stok) | Petugas BMN |
| `#/lokasi` | Lokasi: kantor/wilayah, lantai, ruangan, penanggung jawab | Petugas BMN, Admin |
| `#/pengguna` | Manajemen pengguna | Admin |
| `#/pengaturan`, `#/akses` | Pengaturan dan Hak Akses (hanya lewat menu avatar kanan atas) | Admin |
| `#/profil`, `#/notif` | Profil dan notifikasi (Profil dan Keluar lewat avatar) | Semua |

Akun contoh: `arman` (PJ), `bmn` (Petugas BMN), `admin` (Admin).

## 6. Fitur Utama

- **Katalog dan keranjang** untuk PJ, dengan animasi barang masuk keranjang dan peringatan stok.
- **Penomoran surat:** kode di `settings` (`kodeND`, `kodeSBBK`); nomor urut diisi manual.
- **Dokumen cetak** ND, SBBK, BAST (font Times New Roman), lengkap dengan tanda tangan dan cap. Tanda tangan dan cap otomatis dibuat transparan (`clearBg`).
- **Foto:** banyak foto sekaligus, dikompres di canvas dan dapat diperbesar (lightbox). Foto master barang dapat dicari otomatis dari internet.
- **Pagination, filter, dan pencarian** di semua daftar (`pgOf`, `ffOf`, `ftoolbar`). Hanya bagian daftar yang dimuat ulang.
- **Penanda "Baru/Update"** per pengguna, juga pada badge menu.
- **Notifikasi** per pengguna.
- **Loader:** splash kardus berisolasi saat aplikasi dibuka, dan loader mini (kardus buka-tutup) saat pindah halaman.
- **Mobile:** menu bawah (bottom nav), bilah atas dua baris, modal berupa bottom-sheet.

## 7. Sistem Desain

### Warna (CSS variable di `:root`)
| Token | Nilai | Pemakaian |
|---|---|---|
| `--p` | `#ff5a0a` | Warna utama (oranye) |
| `--p2` | `#e04e05` | Hover warna utama |
| `--p3` | `#ffe8dc` | Latar oranye lembut |
| `--bg` | `#f1f5f9` | Latar halaman |
| `--tx` | `#0f172a` | Teks |
| `--mu` | `#64748b` | Teks redup |
| `--ok` / `--warn` / `--err` | `#16a34a` / `#d97706` / `#dc2626` | Status |

**Label status** (`.stag.s-*` di pojok kartu, `.chip.sc`): teks putih, kecerahan setara oranye utama. Warna: `#ea9606`, `#ff7b00`, `#ff5a0a`, `#ec3b13`, hijau `#26a065`, ditolak `#b8203f`. Label kategori rusak: ringan krem, sedang terakota, berat merah muda.

### Tipografi
Inter, fallback `system-ui`, "Segoe UI", Roboto, Arial. Ukuran dasar 14px. Judul tebal (800–900).

### Bentuk & komponen
- Radius besar (kartu 16–20px, panel 24px, tombol dan input pill). Permukaan putih dan abu lembut `#f6f7fb`, hampir tanpa bayangan; bayangan hanya untuk tombol oranye dan hover kartu.
- Ikon: set SVG sendiri (`IC`, lewat `ic('nama')`), stroke 2px, `currentColor`.
- Tanpa garis pemisah yang kasar; utamakan blok warna lembut.

## 8. Aturan Tetap

1. **Animasi:** semua fitur/interaksi baru wajib beranimasi ringan. Hanya `transform` dan `opacity`, durasi sekitar 150–350 ms, hormati `prefers-reduced-motion`, jangan render ulang seluruh halaman demi animasi.
2. **Tanpa UI bawaan browser:** select, tanggal, datalist, dan konfirmasi memakai komponen sendiri (`enhAll`, `ask()`).
3. **Modal berisi isian** hanya boleh ditutup lewat tombol (Batal/Simpan), bukan klik latar atau Esc.
4. **Responsif:** uji pada desktop dan mobile (<900px); tata letak harus rapi, tidak dipaksakan.
5. **Hak akses:** fitur baru didaftarkan di `CAN`/`ACT_PERM`/`ROUTE_PERM` dan `MATRIX`.
6. **Menu baru** ditambahkan lewat `navItems()` (isi `k`, `l`, `i`).
7. **Peran dan tempat:** PJ berada di wilayah kerja atau ruangan, Petugas BMN di kantor induk, Admin hanya mengelola sistem.

## 9. Struktur Kode (di dalam satu HTML)

1. **CSS**: dasar, komponen, katalog (`.k*`), kartu, media query, animasi.
2. **Ikon & util**: `IC`, `ic()`, `esc()`, `readImg()`, `trimImg()`, `clearBg()`, `tgl()`.
3. **Data**: `ST`, `RS`, `ROLE`, `CAN`, `ACT_PERM`, `ROUTE_PERM`, `seed()`, `migrate()`, `DB`.
4. **Navigasi**: `navItems()`, `topbar()`, `kpage()`, loader halaman.
5. **Tampilan**: `dashView`, `listView`, `detailView`, `newView`, `rusakView`, `inventarisView`, `masterView`, `lokasiView`, `penggunaView`, `pengaturanView`, `aksesView`, `profilView`, `notifView`.
6. **Render & aksi**: `render()` memilih halaman dari hash; klik ditangani lewat `data-act` pada objek `actions`, izin dicek di `ACT_PERM`.
