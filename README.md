# Sarpras SMAN 1 Berau

Aplikasi pencatatan sarana dan prasarana SMAN 1 Berau: aset SIPBMD (KIB A–F dan ATB), kondisi ruangan untuk Dapodik, BAST penyerahan dan penarikan, aset per pemegang, usulan penghapusan, persediaan (masuk, keluar, mutasi), kertas kerja aset tetap (modal), surat keluar, dan rekon.

- **Alamat aplikasi:** https://sarprassman1berau.github.io/sarpras/
- **Kode aplikasi:** repositori ini (GitHub, publik). Kode tidak berisi data atau password.
- **Data:** Firebase proyek `sarpras-sman1berau` (Cloud Firestore, lokasi Jakarta). Hanya bisa dibuka setelah login.
- **Akun master:** `sarpras.sman1berau@gmail.com`

---

## Panduan penyiapan (sekali saja)

### 1. Buat akun master di Firebase
1. Buka https://console.firebase.google.com → proyek **sarpras-sman1berau**.
2. Menu **Security → Authentication → Users → Add user**.
3. Email: `sarpras.sman1berau@gmail.com`, password: buat password yang kuat. Klik **Add user**.

### 2. Pasang aturan keamanan database
1. Menu **Databases & Storage → Firestore Database → tab Rules**.
2. Hapus seluruh isi yang ada, lalu tempel isi file [`firestore.rules`](firestore.rules) dari repositori ini.
3. Klik **Publish**.

### 3. Izinkan alamat GitHub untuk login
1. Menu **Security → Authentication → tab Settings → Authorized domains**.
2. Klik **Add domain**, isi `sarprassman1berau.github.io`, lalu **Add**.

### 4. Nyalakan GitHub Pages
1. Di repositori ini: **Settings → Pages**.
2. **Source: Deploy from a branch**, Branch: **main**, folder **/ (root)** → **Save**.
3. Tunggu 1–2 menit, lalu buka https://sarprassman1berau.github.io/sarpras/ dan login dengan akun master.

### 5. Buat akun operator
Login sebagai master → **Pengaturan → Akun & akses → Buat akun operator** (nama, jabatan, **nama pengguna atau email**, password awal). Sampaikan login dan password awal kepada operator; operator bisa mengganti passwordnya sendiri di halaman yang sama. Centang **hak akses** akun tersebut (atau pilih paket Staf sarpras / Panitia penghapusan / Hanya lihat). Hak akses bisa diubah kapan saja lewat tombol **Atur akses** di daftar akun.

- **Nama pengguna** (mis. `ika`): tidak perlu email. Di Firebase tersimpan sebagai `ika@pengguna.sarpras-sman1berau.firebaseapp.com` (alamat tanpa kotak surat). Tidak bisa memakai "Lupa password"; jika lupa, master menonaktifkan akun itu lalu membuat akun baru.
- **Email**: bisa memakai "Lupa password" dan tombol "Kirim reset password".

---

## Peran dan alur kerja

| Peran | Siapa | Bisa apa |
|---|---|---|
| Master | Email di `MASTER_EMAILS` (index.html) dan di `isMaster()` (firestore.rules) | Membaca dan menyimpan semua data secara langsung, menyetujui/menolak usulan, mengimpor data, membuat/menonaktifkan akun operator, mengubah pengaturan |
| Operator | Akun yang dibuat master (dokumen `akun/{uid}` dengan `aktif: true`) | Membaca semua data dan mengisi perubahan. Setiap simpanan menjadi **usulan** yang baru berlaku setelah master menyetujuinya di menu **Validasi** |
| Lainnya | Tidak login, atau akun nonaktif | Tidak bisa membaca apa pun (dijaga aturan Firestore) |

### Hak akses operator
Disimpan di `akun/{uid}.izin` sebagai centang per bidang. Akun lama tanpa `izin` dianggap Staf sarpras (akses penuh).

| Kunci | Arti |
|---|---|
| `aset_ubah` | Aset: diserahkan ke, peruntukan, posisi, no. registrasi, SPJ, sumber dana, padanan SIPBMD |
| `aset_kondisi` | Aset: kondisi, tanggal cek, pemeriksa, catatan kondisi, foto |
| `aset_tambah` | Tombol Tambah aset |
| `hapus` | Menu Usulan Penghapusan, nomor usulan, pengecualian hapus |
| `bast` | Menu BAST, Per Pemegang, Laporan BAST, tombol BAST di detail aset |
| `ruang` | Mengubah gedung dan ruang |
| `persediaan` | Menu Persediaan |
| `kertas` | Menu Kertas Kerja Modal dan Rekon Aset |
| `surat` | Menu Surat Keluar |
| `catatan` | Menulis catatan temuan dan mengubah status/tindak lanjutnya |

Paket: **Staf sarpras** = semua; **Panitia penghapusan** = `aset_kondisi`, `hapus`, `catatan`; **Hanya lihat** = tidak ada. Rekap, Aset SIPBMD, Kondisi Ruangan, Catatan Temuan, Validasi, dan Pengaturan selalu terlihat oleh semua akun. Pembatasan menu dan tombol dilakukan di aplikasi (`can(k)`, `canView(v)`); di sisi database, semua perubahan operator tetap hanya berupa usulan yang harus disetujui master, kecuali catatan temuan.

### Catatan temuan
Menu **Catatan Temuan** dan bagian "Catatan temuan" di detail aset, ruang, dan kartu barang persediaan. Setiap catatan tertaut ke satu data (`target: {t: aset|ruang|barang|umum, id, label}`), punya kategori (Kerusakan, Kehilangan, Perlu perbaikan / perawatan, Tidak sesuai data, Lainnya), status (Terbuka, Ditindaklanjuti, Selesai), tindak lanjut, dan foto opsional. Catatan **langsung tersimpan tanpa validasi**, supaya temuan di lapangan cepat tercatat; operator hanya bisa mengubah status dan tindak lanjutnya, isi catatan tidak bisa diubah, dan hanya master yang bisa menghapus. Halaman awal **Rekap Data Sarpras** memuat kotak tulis catatan temuan (cari aset/ruang/barang, pilih, tulis), catatan terbuka terbaru, rekap aset per golongan dan kondisi, kondisi ruangan, temuan pemeriksa, dan riwayat perubahan terbaru beserta nama akunnya. Aset dan ruang yang punya catatan terbuka diberi tanda "n temuan" di daftarnya.

Setiap data yang tersimpan diberi cap `_by` (yang mengubah), `_ok` (yang memvalidasi), dan `_at` (waktu). Riwayat lengkap ada di koleksi `log` (satu dokumen per hari) dan tampil di menu **Validasi → Riwayat perubahan** serta di detail setiap aset.

---

## Catatan teknis (untuk perbaikan dan pengembangan)

**Satu file:** seluruh aplikasi ada di `index.html`. Bagian akhir file (komentar `Firebase adapter`) menyediakan layanan data:
- `window.claude.use("db" | "user" | "assets" | "downloads")` adalah lapisan penyesuai. Aplikasi ini awalnya dibangun sebagai artifact claude.ai; adaptor ini meniru layanan yang sama di atas Firebase, sehingga kode aplikasi di atasnya tidak perlu tahu sedang berjalan di mana.
- **Lapisan simpan `W`** (cari `const W = {`): semua penulisan data lewat `W.set / W.update / W.merge / W.del`, dikelompokkan dengan `W.group(ringkasan, fn)`. Master → langsung ditulis + dicatat di `log`. Operator → dikumpulkan menjadi satu usulan di `usulan/{uid}`.
- `decide(p, ok)` menerapkan atau menolak usulan, lalu menulis `keputusan/{id}`.

**Koleksi Firestore:**

| Koleksi | Isi |
|---|---|
| `seed` | Data impor SIPBMD, dipotong per ±220 KB (`kind: "aset"`, `items: [...]`). Kunci aset = Kode Aset + No. Register yang dinormalkan (`k`) |
| `impor` | Riwayat impor SIPBMD (dokumen `seed` mana saja yang dibuat) |
| `updb` | Isian internal per aset SIPBMD (pemegang, posisi, kondisi, foto, dll.), dikelompokkan dalam 32 dokumen `bNN` berisi peta `m` supaya hemat kuota baca. Di kode aplikasi tetap terlihat sebagai koleksi `upd` |
| `aset_baru` | Aset yang tidak berasal dari impor SIPBMD (tambah manual, hasil kertas kerja) |
| `kk` | Kertas kerja aset tetap (modal): satu dokumen per pekerjaan, `items` = rincian barang. Sinkron otomatis ke `aset_baru` dan `psd_trx` (`kkSync`) |
| `trx`, `surat` | BAST penyerahan/penarikan dan buku surat keluar |
| `psd_barang`, `psd_trx` | Persediaan: master barang dan transaksi masuk/keluar (nilai keluar FIFO) |
| `gdg`, `ruang` | Kondisi ruangan per gedung untuk Dapodik (terpisah dari aset KIB C) |
| `penerima` | Daftar pilihan "Diserahkan ke" |
| `foto` | Foto (JPEG dikompres, disimpan sebagai data URL, maks. ±420 KB per foto) |
| `akun` | Akun aplikasi: nama, jabatan, email, peran, aktif, hak akses (`izin`) |
| `catatan` | Catatan temuan (lihat di atas) |
| `usulan`, `keputusan`, `log` | Alur validasi dan riwayat perubahan |
| `meta/config` | Pengaturan surat (penandatangan, pola nomor surat) |

**Keputusan penting:**
- Kondisi aset: Baik, Rusak Sedang, Rusak Berat, Hilang. Rusak Berat/Hilang otomatis masuk usulan penghapusan kecuali dicentang "Jangan usulkan hapus".
- Kondisi ruang (Dapodik): Baik, Rusak Ringan, Rusak Sedang, Rusak Berat.
- Batas aset tetap (kebijakan akuntansi Kaltim): Peralatan & Mesin ≥ Rp1.500.000, Gedung & Bangunan ≥ Rp20.000.000. Di bawahnya disarankan reklas ke persediaan di kertas kerja.
- Golongan dari kode rekening belanja: 5.2.02 → KIB B, 5.2.03 → KIB C, 5.2.04 → KIB D, 5.2.05 → KIB E, 5.2.06 → ATB.
- Foto disimpan di Firestore (bukan Cloud Storage) karena Cloud Storage mewajibkan paket Blaze berbayar. Kuota gratis Firestore: 1 GiB, 50.000 baca dan 20.000 tulis per hari.

**Memperbarui aplikasi:**
- Lewat Claude: sambungkan GitHub di claude.ai (Settings → Connectors → GitHub, masuk dengan akun `sarprassman1berau`), lalu minta perubahan di chat project "Ika Sarpras". Claude mengubah `index.html` dan GitHub Pages memperbarui situs dalam 1–2 menit.
- Manual: unggah `index.html` baru lewat **Add file → Upload files** di repositori ini.
- Perubahan aturan akses: ubah `firestore.rules` di repositori, lalu tempel ulang di Firebase → Firestore → Rules → Publish.
- Mengganti email master: ubah `MASTER_EMAILS` di `index.html` **dan** daftar di `isMaster()` pada `firestore.rules`, lalu publish ulang aturannya.

**Cadangan:** gunakan tombol Unduh Excel di tiap menu secara berkala. Data di Firebase tidak ikut terhapus ketika kode aplikasi diperbarui.
