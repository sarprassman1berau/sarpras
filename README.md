# Aset & Persediaan SMAN 1 Berau (Aplikasi SMAN 1 Berau)

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

### Status terkini dan chat tim
- **Tata letak:** menu halaman ada di panel kiri yang dibuka dengan tombol garis tiga di kiri atas, dikelompokkan (Utama, Aset & ruang, Penyerahan barang, Persediaan & keuangan, Akun). Nama halaman yang sedang dibuka tampil besar di kepala halaman (untuk Aset juga golongan KIB-nya). Titik merah di tombol menu = ada usulan menunggu validasi (master).
- **Status terkini** (ikon orang di kanan atas, angka hijau = jumlah akun online; klik untuk melihat daftarnya): daftar akun aktif, siapa yang sedang membuka aplikasi dan di menu apa, atau kapan terakhir aktif. Setiap akun menulis `online/{uid}` (`nama`, `at`, `view`, `off`) saat membuka aplikasi, setiap 90 detik selama halaman terbuka, dan saat menutup halaman (`off: true`). Dianggap online bila `at` kurang dari 4 menit lalu.
- **Chat tim** (tombol "Chat tim" di kanan bawah setiap halaman): pesan di koleksi `chat` (`by`, `nama`, `teks`, `at`), dibaca semua akun yang login, maks. 2.000 huruf. Yang ditampilkan 200 pesan terakhir. Ada penanda pesan belum dibaca dan pemberitahuan pesan baru. Hanya master yang bisa menghapus pesan.

Setiap data yang tersimpan diberi cap `_by` (yang mengubah), `_ok` (yang memvalidasi), dan `_at` (waktu). Riwayat lengkap ada di koleksi `log` (satu dokumen per hari) dan tampil di menu **Validasi → Riwayat perubahan** serta di detail setiap aset.

---

### Unggah foto: kamera atau galeri
Semua tempat unggah foto (detail aset, catatan temuan, foto ruang, BAST, BAST bantuan) di HP/tablet kini menampilkan pilihan **Ambil foto dengan kamera** atau **Pilih dari galeri / file** (pendengar klik global untuk `input[type=file][accept^=image]`: atribut `capture` dipasang hanya bila memilih kamera). Di komputer langsung membuka jendela pilih file.

### Migrasi aset tidak tercatat / BOSP Daerah ke menu lain
- **Per aset** (detail aset → Pencatatan): status *Tidak tercatat* · *BOSP Daerah* · *Sudah tercatat di SIPBMD* (isi kode aset dan no. register → pindah ke menu golongan KIB yang sesuai kode), pilihan **Golongan aset** untuk memindah golongan, dan *Padanan di SIPBMD* bila barang yang sama sudah ada di data SIPBMD (catatan sekolah disembunyikan, data SIPBMD yang dipakai).
- **Sekaligus** (centang beberapa aset → Isi sekaligus → *Migrasi / pindahkan aset*): pindahkan ke Aset tidak tercatat, Aset BOSP Daerah, atau **Sudah tercatat di SIPBMD** (kode aset sama untuk semua, no. register berurutan mulai dari angka awal; golongan mengikuti kode), dan/atau ubah golongan.
- Pemindahan hanya mengubah field `sip`, `grp`, `kode`, `noreg`, `kat` lewat `saveUpd`, sehingga riwayat, foto, pemegang, kondisi, dan batas penggunaan ikut terbawa; dari akun operator menunggu validasi.

### Masa penggunaan barang (BAST penyerahan) dan kondisi saat penarikan
- **BAST Penyerahan** memuat **batas waktu penggunaan** (pilihan cepat: 1 tahun, akhir tahun ajaran 30 Juni, 2 tahun, atau tanpa batas). Kalimat BAST menyebut batas tersebut dan kewajiban memperbarui atau mengembalikan barang. Batas disimpan di setiap barang (`batas`).
- **Pengingat**: barang yang masih dipegang dan batasnya sudah lewat (merah) atau berakhir ≤30 hari (kuning) tampil di atas halaman BAST (per pegawai, tombol **Perbarui masa penggunaan** = BAST penyerahan baru yang sudah terisi, dan **Tarik barang**), di Rekap, di detail aset, angka merah di menu BAST, dan status kartu Aset & Persediaan di Beranda.
- **Menu Masa Penggunaan Barang** (`#masa`, `vMasa`, `masaRows`; Aset & Persediaan → Penyerahan barang, angka merah = jumlah yang lewat batas): kartu filter *Sudah lewat batas* · *Berakhir ≤30 hari* · *Semua yang berbatas* · *Dipegang tanpa batas*, pencarian, Excel. Batas bisa diubah langsung per baris (tanggal di tabel) atau sekaligus untuk barang terpilih (1 tahun / akhir TA / tanggal bebas), dan dari barang terpilih milik satu pegawai bisa langsung dibuat BAST perpanjangan atau penarikan. Batas juga bisa diubah di detail aset (isian "Batas waktu penggunaan"). Semua perubahan lewat `saveUpd` (field `batas`), jadi langsung terbaca di semua menu; dari akun operator menunggu validasi master. Halaman BAST dan Rekap hanya menampilkan ringkasan pengingat dengan tautan ke menu ini.
- **BAST Penarikan** wajib mengisi **kondisi terkini** dan **alasan penarikan** setiap barang (alasan umum dipakai bila alasan per barang kosong); keduanya tercetak di tabel BAST.
- Saat BAST penarikan disimpan, kondisi terkini langsung menjadi kondisi barang (tanggal cek = tanggal BAST, pemeriksa = Pihak Kedua, catatan = alasan), sehingga ikut ke Rekap, Pemeliharaan & Perbaikan (Rusak Sedang), Usulan Penghapusan (Rusak Berat/Hilang), Unduh KIB, Buku Induk, dan Per Pemegang. Dari akun operator perubahan ini menunggu validasi master seperti biasa.

### Aplikasi Keuangan (tahap 1)
Dibuka dari Beranda (akun operator perlu dicentang **Keuangan** di Akun & hak akses; hak tambahan: *Mengajukan usulan & KAK*, *Mengelola RKAS*, *Mencatat SPJ*). Semua isian Keuangan langsung tersimpan (tanpa validasi master) sesuai centang tersebut.
- **Ringkasan keuangan**: pagu, RKAS, realisasi, sisa per sumber dana per tahun; dokumen SPJ terakhir.
- **Standar harga (SSH/SBU)**: ±30 ribu komponen (katalog sekolah 2027, SSH barang, SBU jasa) dari file *aplikasi_bahan.xlsx*, disimpan di `data/standar-harga.json` dan dimuat saat dibutuhkan. Pencarian mengisi otomatis uraian, spesifikasi, satuan, harga, dan kode rekening.
- **Usulan & KAK**: pengusul mengisi usulan kebutuhan (dari standar harga), bidang, sumber dana, KAK; pengelola RKAS mengasistensi (centang → *Setujui → masuk RKAS* / *Minta revisi* / *Tolak* dengan catatan). Cetak **Lembar Asistensi** per bidang dan **Lembar KAK** (dengan perkiraan kebutuhan anggaran dari usulan yang memilih KAK tersebut).
- **RKAS & RKBMD**: pagu per tahun & sumber dana, kontrol *belum dianggarkan / melebihi pagu*, baris per kode rekening (manual, dari usulan, atau **impor Excel** — tabel datar ARKAS/usulan maupun matrik berjenjang SIPD/BOSP Daerah), realisasi per baris, cetak **matrik RKAS/RKBMD** berjenjang (5 → 5.1 → … → rincian) PDF dan Excel.
- **SPJ belanja barang/jasa**: sekali isi per **nomor BPU ARKAS** → **Surat Pesanan Barang** (`900.1.3.3/{BPU}.A/SPB/{BOSNAS}/SMAN1-Berau/{bulan}/{tahun}`), **Tanda Terima Barang** (`.B/TTB/`), dan **Kuitansi** (terbilang otomatis). Tiap rincian bisa dihubungkan ke baris RKAS (realisasi) dan langsung dicatat ke **Persediaan masuk** atau **Aset** (golongan dipilih).
- **Honorarium**: daftar pembayaran (kertas F4/legal mendatar) dengan PPh 21 otomatis dari golongan (PNS I–II / PPPK I–VIII 0%, III / IX–XII 5%, IV / XIII–XVII 15%, tanpa golongan 5%; bisa diubah per orang) dan jumlah diterima, plus kuitansi.
- **Perjalanan dinas**: rincian biaya (angkutan, lumpsum hari × tarif, penginapan hari × tarif, terbilang, tanda terima uang), laporan perjalanan dinas (hasil per poin), dan kuitansi.
- Penandatangan diatur di **Pengaturan surat & penerima**: Kepala Sekolah (+ pangkat), Bendahara Dana BOS (+ NIP, golongan), Pengurus/pengelola barang (TTB).

### Permintaan Barang Persediaan (aplikasi baru)
Guru dan staf membuka aplikasi **Permintaan Barang Persediaan** di Beranda (aktif untuk semua akun, bisa dimatikan per akun), memilih barang dari stok persediaan, mengisi jumlah, unit/bagian, dan keperluan, lalu mengirim permintaan. Akun yang dicentang **"Memproses permintaan persediaan"** (dan master) melihat antrean di tab Daftar permintaan, menyesuaikan jumlah yang diserahkan dengan stok, lalu klik **Serahkan & catat keluar**: setiap barang langsung tercatat sebagai Persediaan Keluar (nilai FIFO, nomor bukti `BKP/…`, ditandai "dari aplikasi permintaan"), dan Bukti Permintaan dan Pengeluaran Barang Persediaan (PDF) terunduh dengan tanda tangan Pengurus Barang · Mengetahui Waka Sarpras · Yang meminta/menerima. Permintaan bisa ditolak dengan alasan atau dibatalkan pemohon. Tab Stok tersedia dan Rekap pengeluaran (per unit, per barang, Excel).

### BAST Penerimaan Bantuan
Menu **BAST Penerimaan Bantuan** (Aset & Persediaan → Penyerahan barang, hak "BAST & pemegang"): berita acara serah terima bantuan/hibah dari pihak luar (Pihak Pertama: instansi pemberi; Pihak Kedua: Kepala Sekolah). Rincian barang dengan jumlah, satuan, harga satuan, kondisi, dan pilihan dicatat sebagai **aset** (golongan dipilih, masuk ke Aset tidak tercatat di SIPBMD, satu catatan per unit, sumber dana dan asal usul "Hibah dari …") atau **persediaan** (Persediaan Masuk). Dokumentasi hingga 4 foto menjadi lampiran PDF. Daftar BAST tersimpan bisa dicetak ulang, diubah, dihapus, dan diunduh ke Excel.

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
| `cetak`, `pengingat` | Riwayat cetak BAST; catatan pengingat per menu |
| `pinjam` | Peminjaman Aula dan Ruang Multimedia (Jadwal Ruang) |
| `online`, `chat` | Status terkini dan chat tim (lihat di atas) |
| `usulan`, `keputusan`, `log` | Alur validasi dan riwayat perubahan |
| `meta/config` | Pengaturan surat (penandatangan, pola nomor surat) |

**Beranda (ruang aplikasi):** halaman pertama setelah login (`#home`, fungsi `vHome`, daftar `APPS`). Kartu aplikasi Sarpras (aktif), Keuangan dan Waka Sarpras (sedang disiapkan). Akses per aplikasi disimpan di `akun/{uid}.izin` dengan kunci `app_sarpras`, `app_keuangan`, `app_waka` (akun tanpa kunci ini tetap boleh membuka Sarpras); fungsi `appAccess(id)`. Akun tanpa akses Sarpras hanya melihat Beranda dan Pengaturan.

**Nama aplikasi:** aplikasi pertama bernama **Aset & Persediaan** (di kode `app_sarpras`, `id:"sarpras"`). Halaman login dan Beranda memakai nama portal "Aplikasi SMAN 1 Berau".

**Pemeliharaan & Perbaikan** (`#pelihara`, `vPelihara`, `plRows`): aset dan ruang berkondisi Rusak Sedang otomatis masuk; ruang Rusak Berat juga masuk sebagai perlu rehabilitasi (ruang tidak bisa dihapus); aset Rusak Berat/Hilang tetap ke Usulan Penghapusan. Data perbaikan disimpan di field `pl` aset (lewat `saveUpd`) atau dokumen `ruang` (`pl: {st, tgl, oleh, biaya, sumber, ket, selesai}`); status: Belum ditangani, Dijadwalkan, Sedang diperbaiki, Menunggu dana / suku cadang, Selesai. "Selesai diperbaiki" mengubah kondisi menjadi Baik. Hak akses edit: kunci `pelihara`.

**Jadwal Penggunaan Aula dan Multimedia** (aplikasi tersendiri, `#jadwal`, `vJadwal`, koleksi `pinjam`): kalender mingguan Aula dan Ruang Multimedia berdampingan, form pengajuan dengan cek bentrok otomatis (dan saran pindah ke ruang lain yang kosong), daftar pengajuan, setujui/tolak (dengan alasan)/batalkan, unduh Excel. Status: Diajukan, Disetujui, Ditolak, Dibatalkan. Semua akun terdaftar bisa melihat (kunci aplikasi `app_jadwal`, aktif secara bawaan). Mengajukan butuh centang `jadwal_ajukan`, menyetujui butuh `jadwal_setuju` (keduanya harus dicentang khusus; akun lama tidak otomatis punya). Pengajuan langsung tersimpan tanpa validasi master; aturan Firebase `pinjam` membatasi siapa yang boleh membuat dan mengubah.

**Pembagian aplikasi (Beranda):** Aset & Persediaan (`app_sarpras`), Jadwal Penggunaan Aula dan Multimedia (`app_jadwal`), Keuangan (`app_keuangan`, disiapkan), Waka Sarpras (`app_waka`), Outsourcing (`app_outsourcing`, disiapkan). Menu **Akun & hak akses** (`#akun`, `vAkunPage`) berada di aplikasi Waka Sarpras; master bisa membuat, mengubah (nama, jabatan, hak akses; login tidak bisa diubah), dan menonaktifkan akun. Akun lain membuka `#akun` lewat "Akun saya" untuk ganti password. Akun yang sudah dinonaktifkan bisa **dihapus** (tombol Hapus, dengan konfirmasi): dokumen `akun/{uid}` diberi `dihapus: true` (tidak benar-benar dibuang, supaya riwayat perubahan tetap menampilkan namanya), hilang dari daftar akun dan status terkini, dan dokumen `online/{uid}`-nya dihapus. Login lama tetap tidak bisa masuk dan nama penggunanya tidak bisa dipakai lagi untuk akun baru (batas Firebase gratis). **Validasi** dan **Pengaturan surat & penerima** (`#atur`) tetap di aplikasi Aset & Persediaan; validasi khusus perubahan data aplikasi itu.

**Format KIB A (Tanah) & KIB C (Gedung dan Bangunan):** `KIB_FMT` menampilkan kolom SIPBMD yang tersimpan di `extra` dalam susunan kartu SIPBMD (Tanah: data barang, lokasi, dokumen kepemilikan, tanda batas; Gedung: tanggal & nomor dokumen, bertingkat, beton, luas bangunan, lokasi, status tanah, asal usul). Daftar KIB A menampilkan luas dan dokumen kepemilikan; KIB C menampilkan luas bangunan, konstruksi, dan status tanah. Kolom dicocokkan dari judul kolom file impor SIPBMD, dan semua kolom SIPBMD ikut di Unduh Excel.

**Kelengkapan KIB:** isian KIB A/KIB C yang kosong di SIPBMD bisa dilengkapi lewat tombol "Lengkapi … isian yang belum tercatat" di detail aset (hak `aset_ubah`); disimpan di field `kib` (peta label → nilai) lewat `saveUpd`, ditandai "diisi sekolah", tidak mengubah data SIPBMD. **Kondisi belum dikonfirmasi:** baris aset tanpa kondisi diwarnai merah dan detailnya diberi peringatan. **Detail aset** KIB B, KIB E, dan aset tidak tercatat menampilkan "Diserahkan ke" dan "Lokasi" di kepala detail; Catatan temuan selalu tampil paling atas di detail aset. **Menu Rekon** menggabungkan Kertas Kerja Modal (`#kertas`) dan Rekon Aset (`#rekon`) dengan tab di atas halaman.

**Lengkapi / koreksi data aset (semua golongan):** detail aset menampilkan data per golongan (`KIB_FMT`; KIB B/D/E/F/ATB memakai `GEN_FMT`). Tombol "Lengkapi data" membuka isian; nilai yang berbeda dari SIPBMD disimpan di `kib` (label → nilai) lewat validasi master. Setelah divalidasi, tampil label "diisi sekolah" (SIPBMD kosong) atau "beda dari SIPBMD" beserta nilai asli SIPBMD. Nilai perolehan dan kondisi SIPBMD tidak bisa diubah. **Riwayat** setiap aset menampilkan siapa mengubah, siapa memvalidasi, dan rincian perubahan yang masih menunggu. **Catatan temuan** dari operator diberi `val:false` dan baru masuk rekap setelah master memvalidasinya (di kartu catatan atau menu Validasi). **Unduh KIB** (`#unduhkib`): per item (PDF kartu / Excel; juga tombol di detail aset), per golongan, per periode tahun perolehan, dan custom (filter + pilihan kelompok kolom); Excel satu sheet per golongan.

**Impor daftar pegawai:** Pengaturan surat & penerima → "Impor daftar pegawai (Excel)" (master). Kolom Nama, NIP, Jabatan, Status (Kepegawaian), Kelompok dikenali dari judulnya; dicocokkan ke penerima yang ada lewat NIP lalu nama; jabatan dan peruntukan yang sudah diisi dipertahankan. Disimpan di koleksi `penerima` (field tambahan `kel`).

**Surat & PDF:** semua surat memakai `pdfKop` (Pemprov Kaltim · Dinas Pendidikan dan Kebudayaan · SMA Negeri 1 Berau + alamat, logo kiri-kanan), `pdfJudul` (judul tebal bergaris bawah + nomor), `pdfPara`, `signBlock` (tempat/tanggal, nama bergaris bawah, label rata bawah; `row3` untuk BAST: Pihak Pertama · Mengetahui Kepala Sekolah · Pihak Kedua), dan `pdfFinish` (catatan kaki + nomor halaman). **BAST** bisa disematkan foto dokumentasi (maks. 2 foto barang + 2 foto proses, disimpan di `trx.fotos` `{id, j:"barang"|"proses"}`); foto dicetak di halaman **Lampiran Dokumentasi** (kop, judul, nomor BAST, foto besar per kelompok, tanda tangan kedua pihak) dalam file PDF yang sama. Pratinjau BAST di aplikasi berbentuk lembar kertas dengan susunan yang sama. Pencarian "Tambah barang" menampilkan tahun perolehan dan lokasi.

**Buku Induk:** tombol "Unduh Buku Induk" di setiap halaman golongan aset (termasuk Aset tidak tercatat; `biPanel`, `biItems`, `biPdf`, `biExcel`). Filter: tahun perolehan, sumber dana, status pencatatan, kondisi, lokasi, pemegang, hanya berfoto. PDF A4 lanskap dengan kop, judul "BUKU INDUK BARANG MILIK DAERAH", identitas (kuasa pengguna, pengguna barang, filter, jumlah & nilai, tanggal), tabel lengkap dengan foto pertama tiap aset, dan tanda tangan Kepala Sekolah (kiri) · Waka Sarpras · Pengurus Barang. Excel berisi data lengkap tanpa foto.

**Isi sekaligus:** di daftar aset, centang beberapa aset (atau "Pilih semua hasil filter"), lalu "Isi sekaligus untuk N aset" (`selBar`, `bulkPanel`, `bulkApply`). Kolom yang diisi diterapkan ke semua aset terpilih: kelengkapan data KIB (bila satu golongan), diserahkan ke, lokasi, peruntukan, sumber dana, no. SPJ, kondisi, tanggal cek, pemeriksa; kolom kosong tidak mengubah apa pun. Semua perubahan masuk satu usulan validasi (operator) atau langsung tersimpan (master); isian yang sama dengan SIPBMD tidak disimpan sebagai isian sekolah.

**Aset/Modal:** menu aset (dulu "Aset SIPBMD"). Kartu golongan KIB A–ATB diberi tanda **SIPBMD** dan hanya berisi aset tercatat SIPBMD; aset lain ada di **Aset tidak tercatat** (`#aset-tak`) dan **Aset BOSP Daerah belum tercatat di SIPBMD** (`#aset-bospda`, field `grp:"bospda"`, belanja cabang dinas mulai 2025). Status pencatatan di detail aset punya tiga pilihan (Tidak tercatat / BOSP Daerah · menunggu SIPBMD / Sudah tercatat); mengubahnya memindahkan aset ke menu yang sesuai. Helper: `subItems`, `subMeta`, `isSpecialSub`.

**BAST:** memilih pegawai menampilkan barang yang tercatat diserahkan kepadanya (dengan tanda sudah/belum ber-BAST) untuk dipilih. Foto pertama setiap barang dari data aset otomatis masuk lampiran ("Foto barang (dari data aset)"), ditambah foto tambahan dan foto proses. **Riwayat cetak** di koleksi `cetak` (`jenis`, `nomor`, `ke`, `ulang`, `by`, `at`); mencetak BAST yang sudah pernah dicetak meminta konfirmasi "Cetak ulang". Pratinjau berbentuk lembar A4 (rasio 210:297).

**Catatan pengingat per menu:** akun master bisa menulis pengingat di setiap menu (koleksi `pengingat`, id = nama menu, untuk Aset/Persediaan per sub-menu), tampil untuk semua akun.

**Surat Keluar** dipindah ke aplikasi Waka Sarpras (butuh akses `app_waka` dan hak `surat`). Tombol impor di semua menu kini bernama "Import".

**Import di Aset tidak tercatat & Aset BOSP Daerah:** tombol "Import" (hak `aset_tambah`; `aiPanel`, `aiRead`, `aiDocs`, `aiGo`). Membaca Excel dengan kolom Nama Barang, Merek, Spesifikasi, Serial Number, Bahan, Sumber, Tahun Pengadaan/Penerimaan, Tanggal Penerimaan, Nomor Registrasi, Kode SIMDA, Jumlah, Harga per Unit/Nilai, Keterangan, Catatan, Lokasi, Pemegang, Golongan (judul dikenali otomatis, kolom ganda diambil yang pertama). Opsi pecah per unit (default) dan golongan bawaan. Disimpan ke `aset_baru` (`grp:"bospda"` untuk BOSP Daerah, `impBatch` untuk penanda impor); dari operator menjadi satu usulan (maks. 300 aset). **Buku Induk PDF:** pilihan "Dengan foto" / "Tanpa foto" (`UI.bi.foto2`; tanpa foto = kolom Foto tidak dicetak, kolom spesifikasi dilebarkan). Tanda tangan selalu satu halaman dengan tabel: bila ruang tidak cukup, beberapa baris terakhir dipindah ke halaman berikutnya bersama tanda tangan (PDF disusun ulang). **Buku Induk gabungan:** pilihan "Gabungkan semua aset" menyusun satu buku induk untuk semua golongan SIPBMD, aset tidak tercatat, dan aset BOSP Daerah (bagian per kelompok dengan subjumlah dan jumlah seluruh aset; Excel satu sheet per kelompok).

**Filter tahun perolehan** (`thSelect`, `thMatch`, berdasarkan `tahunOf`): di setiap halaman Aset/Modal (golongan, Aset tidak tercatat, BOSP Daerah), Usulan Penghapusan, dan Pemeliharaan & Perbaikan, termasuk pilihan "Tanpa tahun". Daftar, Unduh Excel, Unduh KIR, Isi sekaligus (pilih semua hasil filter), dan lampiran penghapusan mengikuti filter ini; Buku Induk otomatis memakai tahun yang dipilih.

**Cover cetak** (`pdfCover`, `COVER_ID`; `pdfFinish(doc, {cover:true})` melewati cover saat memberi nomor halaman): Buku Induk (centang "Sertakan cover", `UI.bi.cover`), Kertas Kerja Modal (centang "Cover", `UI.kk.cover`), dan **Berkas Rekon Aset PDF** baru (tombol "Unduh PDF berkas rekon", `rekonPdf`, centang "Cover", `UI.rekon.cover`). Cover memuat logo Pemprov Kaltim dan sekolah, judul, cakupan/periode, jumlah dan nilai, kotak identitas (satuan pendidikan, NPSN/NSS, kuasa pengguna, pengguna barang, pemerintah daerah, alamat), dan nama instansi + tahun di bawah. Berkas Rekon PDF (A4 tegak): ringkasan, A. per sumber dana, B. masuk menurut SIPBMD, C. masuk menurut catatan sekolah, tanda tangan Kepala Sekolah · Waka Sarpras · Pengurus Barang (tetap satu halaman dengan tabel terakhir).

**Buku Induk – gabung buku/ATL identik** (`biMerge`, `noRange`, `UI.bi.gab`, default aktif; centang "Gabungkan buku/ATL yang identik" muncul bila ada aset ATL): catatan KIB E dengan kode, nama, isian KIB (judul, pengarang, merek, spesifikasi, dst.), tanggal/tahun, sumber dana, asal usul, harga satuan, kondisi, pemegang, lokasi, dan status SIPBMD yang sama dicetak satu baris: No. Register berupa rentang (mis. "1–25", "30, 32, 34"), keterangan "(n register digabung)", nilai total dengan "(n eks @ harga)". Tabel Buku Induk (PDF dan Excel) kini punya kolom terpisah **Jumlah · Satuan · Harga Satuan (Rp) · Total Nilai Perolehan (Rp)** untuk semua baris (`biJml`, `biSatuan`: `x.satuan` bila ada, selain itu ATL buku = "eksemplar", ATL lain = "buah", tanah = "bidang", ATB = "paket", lainnya "unit"); baris JUMLAH/SUBJUMLAH menampilkan total barang dan total nilai, identitas menulis "n barang (dicetak dalam m baris) · total nilai". Berlaku di PDF dan Excel, termasuk buku induk gabungan; golongan lain tetap per register.

**BAST Penerimaan Bantuan** (`vBantuan`, `btSave`, `btPdf`, koleksi `bantuan`, lewat lapisan W sehingga usulan operator menunggu validasi): `dicatat:true` setelah barang dibuat sebagai `aset_baru` (`bantuan:id`, `asal`, `sumber`) dan/atau `psd_trx` masuk (`bantuan:id`, `penyedia` = pemberi). Nomor memakai `nextNomor("serah")` (penomoran ikut memperhitungkan BAST bantuan). `signBlock` kini menerima `opt.ksLabel` dan `p.nip === null` (tanpa baris NIP) serta `p.idLabel`.

**Aplikasi Permintaan Barang Persediaan** (`app_minta`, default aktif; `vMinta`, `mtKirim`, `mtSerah`, `mtPdf`, koleksi `minta`; hak khusus `minta_proses` di `MT_IZIN`): penulisan langsung ke Firestore (bukan lewat usulan) — aturan `minta` (buat oleh pemohon dengan `st:'Diajukan'`; ubah oleh `minta_proses` atau pembatalan oleh pemohon) dan `psd_trx` (buat `jenis:'keluar'` dengan field `minta` oleh `minta_proses`). Status: Diajukan → Diserahkan / Ditolak / Dibatalkan.

**Keuangan** (`vKeu*`, koleksi `keu_usul`, `keu_kak`, `keu_rkas`, `keu_pagu`, `keu_spj`, `keu_honor`, `keu_perjadin`; izin khusus `app_keuangan`, `keu_usul`, `keu_rkas`, `keu_spj` di `KEU_IZIN`): ditulis langsung ke Firestore (`keuSet`/`keuDel`). Aturan: koleksi `keu_*` hanya **dibaca** akun bercentang Keuangan (aturan umum kini `match /{col}/{docId=**}` dengan `col.matches('keu_.*')`), usulan/KAK oleh pengusulnya selama belum disetujui, RKAS/pagu oleh `keu_rkas`, SPJ/honor/perjadin oleh `keu_spj`. Standar harga statis `data/standar-harga.json` (`SH`, `shLoad`, `shFind`; daftar rekening dipadatkan di `rl`). Impor RKAS `rkasParse` (deteksi judul kolom Uraian + Harga/Tarif, judul dua baris, kode rekening terbawa, baris tanpa volume angka dilewati). Rute `#keu-…` dikenali utuh (rute lain tetap `view-sub`). Realisasi = jumlah SPJ/honor/perjadin per tahun & sumber dana; per baris RKAS lewat tautan `rkas`.

**Pembaruan nama dan data inti aset:** isian "Nama barang" (`@nama`) kini ada di urutan pertama setiap format KIB. Nilai yang diisi sekolah (lewat Lengkapi data / Isi sekaligus, setelah divalidasi) menggantikan nilai tampil di semua tempat — daftar, detail, BAST, KIR, Buku Induk, Unduh KIB, Excel, Rekon (`buildItems` menimpa `x[field]` dari `x.kib[label]`, nilai asli SIPBMD disimpan di `x.orig`, dan `kibVal` membandingkan dengan `x.orig` sehingga tanda "beda dari SIPBMD" tetap muncul). Impor ulang SIPBMD memperbarui nilai asli; bila sekolah sudah mengisi nama sendiri, nama sekolah tetap dipakai (kosongkan isian untuk kembali ke nama SIPBMD). Unduh KIB bagian Identitas menambahkan kolom "Nama di SIPBMD" bila berbeda.

**Aset tidak tercatat:** kartu "Aset tidak tercatat" di deretan golongan (alamat `#aset-tak`) menampilkan semua aset yang `sip` = false dari semua golongan. Di detail aset ada pilihan **Status pencatatan** (Tidak tercatat / Sudah tercatat di SIPBMD); mengubah ke tercatat wajib mengisi kode aset dan no. register SIPBMD (tersimpan di `sip`, `kode`, `noreg`, lewat validasi master seperti perubahan lain), atau pasangkan ke data impor SIPBMD lewat "Padanan di SIPBMD".

**Logo:** logo SMAN 1 Berau dan logo Pemprov Kaltim disimpan di `index.html` sebagai gambar PNG ringan (konstanta `LOGO`). Dipakai sebagai ikon aplikasi, di halaman login, di menu kiri, dan di kop surat (Kaltim kiri, sekolah kanan, tinggi sama) pada pratinjau dan PDF (`pdfKop`). Untuk mengganti logo, ganti isi `LOGO` dan dua tag `<link rel="icon">`.

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
