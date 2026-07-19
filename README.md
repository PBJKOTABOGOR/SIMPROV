# SIMPROV v160 — Hardening, UX, dan Optimasi Respons

Baseline: `SIMPROV_v159_UI_VERIFIKATOR_RINGKAS.zip`.

Revisi diterapkan sebagai patch pada fungsi aktif terakhir. Nama sheet, ID
kegiatan, ID paket, endpoint, URL deployment, data lama, dokumen lama, role,
dan menu utama tetap dipertahankan.

## 1. Keamanan sesi dan role

- Login memakai token sesi server dengan batas absolut 6 jam dan idle browser
  30 menit.
- Setiap request internal mengabaikan objek role dari browser dan mengambil
  ulang akun aktif dari sheet `USER` melalui cache server.
- Respons login tidak lagi mengirim kolom password atau `_row` ke browser.
- Manipulasi `currentUser.role` di browser tidak mengubah kewenangan backend.
- Password lama tetap kompatibel. Backend juga dapat membaca password berformat
  `SHA256$<hash>` untuk migrasi bertahap di kemudian hari.

## 2. Akses file Google Drive

- Semua upload baru tidak lagi otomatis memakai `ANYONE_WITH_LINK`.
- Default sharing baru: `DOMAIN_WITH_LINK`.
- Bila kebijakan domain tidak tersedia, file tetap privat.
- Dokumen lama dan URL lama tidak diubah.
- Script Property opsional `SIMPROV_DRIVE_SHARING` menerima:
  `PRIVATE`, `DOMAIN_WITH_LINK`, atau `ANYONE_WITH_LINK`.

## 3. Pencatatan Non Pengadaan selain Honorarium

- Pipeline disederhanakan menjadi lima tahap:
  Pencatatan Realisasi → Pemeriksaan PBJ → Perbaikan → Siap Diselesaikan →
  Selesai.
- Bagian teratas selalu menjelaskan status dan tindakan berikutnya.
- Tombol `Setujui Paket` dan `Setujui & Selesaikan` hanya tampil ketika seluruh
  realisasi memiliki lampiran dan seluruh lampiran sudah valid.
- Bila belum lengkap, Verifikator melihat alasan konkret mengapa paket belum
  dapat disetujui.
- Tombol `Buka Akses Edit User` diganti dengan bahasa yang lebih jelas:
  `Kembalikan untuk Perbaikan`.
- Paket selesai memakai tindakan khusus `Buka Kembali Paket` dengan alasan
  wajib dan konfirmasi tambahan.
- Pengembalian paket pada tahap siap diselesaikan tidak mereset dokumen lain
  yang sudah valid.
- Pembukaan kembali paket yang sudah selesai tetap mewajibkan pemeriksaan ulang
  seluruh nilai dan dokumen.

## 4. Perbaikan realisasi dan dokumen

- Jika User Bidang mengubah nilai, jenis realisasi, pihak, tanggal, nomor bukti,
  atau keterangan, hanya lampiran pada realisasi tersebut yang dikembalikan ke
  status `MENUNGGU VERIFIKASI DOKUMEN`.
- Lampiran realisasi lain tidak ikut direset.
- Dokumen berstatus Perlu Perbaikan memiliki tombol `Ganti Dokumen`.
- File lama tidak dihapus; statusnya menjadi `DIBATALKAN` dan tetap tersimpan
  sebagai riwayat audit.
- File pengganti langsung muncul dari state lokal tanpa reload dashboard penuh.

## 5. Kartu ringkasan per role

### User Bidang
- Total Paket
- Sedang Dicatat
- Menunggu Pemeriksaan
- Perlu Perbaikan
- Selesai

### Verifikator PBJ
- Total Paket
- Menunggu Pemeriksaan
- Perlu Perbaikan
- Siap Diselesaikan
- Selesai

### Admin/Pimpinan
- Total Paket
- Total Diajukan
- Realisasi Disetujui
- Sisa Pagu
- Selesai

Jika bidang penugasan Verifikator belum diatur, sistem menampilkan
`Penugasan Bidang: Belum diatur` dan tidak menggunakan seluruh data sebagai
fallback.

## 6. Performa

- Token dan identitas user aktif dibaca melalui cache server.
- Pemeriksaan header sheet v158 menggunakan cache selama enam jam.
- Update status, nilai, dan dokumen tetap dilakukan pada state lokal setelah
  aksi berhasil.
- Tidak ada reload seluruh dashboard setelah koreksi nilai, penggantian
  dokumen, verifikasi dokumen, persetujuan, pengembalian, atau penyelesaian
  paket Non Honorarium.
- Cache bust frontend diperbarui menjadi `1600-hardening-ux-performa`.

## File yang berubah

- `app.js`
- `Code.gs`
- `style.css`
- `index.html`
- `README.md`

## Deployment

`Code.gs` berubah dan wajib **Deploy New Version** pada Web App Apps Script.
URL `/exec` dan endpoint lama tetap digunakan.

Setelah deployment, seluruh user harus login ulang satu kali untuk memperoleh
token sesi v160.

## Pemeriksaan

- Sintaks `app.js`: lolos `node --check`.
- Sintaks `Code.gs`: lolos pemeriksaan parser JavaScript.
- Smoke test login: password tidak dikirim ke browser.
- Smoke test spoofing role: role palsu dari browser diabaikan.
- Smoke test tombol persetujuan: tersembunyi sebelum dokumen lengkap dan valid.
- Smoke test pipeline: lima tahap.
- Smoke test penugasan kosong: tidak membuka seluruh data sebagai fallback.
- Pencarian public sharing: hanya tersisa pada helper konfigurasi eksplisit.
- Data dan URL Drive lama tidak dimigrasi atau dihapus.


# SIMPROV v161 — Fix 5 Koreksi Uji Lapangan

## Perbaikan

- Struktur Anggaran Verifikator PBJ/Keuangan menampilkan bidang yang diberikan pada Manajemen Akses, bukan nama role sebagai bidang.
- ID pada tabel Data Perencanaan dapat diklik untuk membuka detail lengkap paket, termasuk sumber harga, jenis pengadaan, cara pelaksanaan, standar biaya, nilai, status, dan riwayat.
- Refresh pada Pencatatan Non Pengadaan menggunakan endpoint baca ringan sehingga tidak mengambil ulang seluruh dashboard.
- Form Data Pembayaran Honorarium mengabaikan baris kosong yang tidak sengaja tersisa, tetap mewajibkan data penerima yang benar, dan memvalidasi total bruto agar tidak melampaui Nilai Perencanaan di frontend serta backend.
- Pipeline Honorarium hanya menandai satu tahap aktif. Status awal paket yang sudah disetujui dinormalisasi menjadi Menunggu Data Honorarium atau Menunggu Pencatatan Realisasi.
- Cache bust frontend diperbarui menjadi `1610-fix-5-koreksi`.

## File yang berubah

- `app.js`
- `Code.gs`
- `style.css`
- `index.html`
- `README.md`

## Deployment

`Code.gs` berubah dan wajib **Deploy New Version** pada Web App Apps Script. URL `/exec`, nama sheet, endpoint lama, ID kegiatan, dan data lama tidak diubah.

## Kompatibilitas

Patch bersifat aditif. Data lama, dokumen Drive lama, role, sheet, dan alur yang sudah berjalan tetap dipertahankan.

# SIMPROV v162 — Honorarium UX dan Validasi Dokumen

## Perbaikan

- ID pada tabel Data Perencanaan tampil sebagai teks klik sederhana tanpa kotak, border, atau efek tombol.
- Metadata teknis seperti kategori, cara pelaksanaan, dan sumber harga tidak lagi disisipkan ke kolom `keterangan`.
- Detail record lama membersihkan metadata legacy ketika ditampilkan; jika tidak ada keterangan pengguna, tampil `-`.
- Form tambah dan edit perencanaan tetap menyimpan sumber harga, standar biaya, jenis pengadaan, dan cara pelaksanaan melalui kolom khusus.
- Nomor urut pada ruang tanda tangan penerima di Daftar Pembayaran Honorarium dihapus.
- User Bidang tidak dapat menyimpan realisasi Honorarium sebelum Dokumen Honorarium TTD, Tanda Terima, dan Bukti Potong Pajak selesai diunggah.
- Pesan pada Pencatatan Realisasi diubah menjadi `Pastikan Dokumen Wajib sudah diunggah sebelum mencatat realisasi.`
- Pemeriksaan nilai Honorarium oleh Verifikator PBJ disederhanakan menjadi dua tombol: `Simpan` dan `Tolak`.
- Jika nilai diubah, tombol Simpan otomatis meminta catatan koreksi. Jika nilai tidak berubah, sistem meminta konfirmasi persetujuan.
- Tombol Tolak meminta alasan dan mengembalikan realisasi kepada User Bidang.
- Form koreksi terpisah dan form pengembalian lama tidak lagi ditampilkan.
- Semua aksi tetap memperbarui state lokal tanpa reload seluruh dashboard.

## File yang berubah

- `app.js`
- `Code.gs`
- `style.css`
- `index.html`
- `README.md`

## Deployment

`Code.gs` berubah dan wajib **Deploy New Version** pada Web App Apps Script. URL `/exec`, nama sheet, endpoint lama, ID kegiatan, data lama, dan dokumen lama tidak diubah.

# SIMPROV v163 — Konfirmasi, ID Paket, Kalkulasi Honorarium, dan Kartu Realisasi

## Perbaikan

- Verifikator PBJ memperoleh popup konfirmasi lengkap sebelum menyetujui perencanaan. Popup menampilkan nama kegiatan, nilai perencanaan, dan dampak setelah persetujuan.
- Daftar pada menu Pengadaan Langsung, Pencatatan Pengadaan, dan Pencatatan Non Pengadaan memiliki kolom ID di sebelah kiri Nama Paket.
- ID paket tampil sebagai teks klik sederhana dan membuka detail paket yang sama dengan klik Nama Paket.
- Bruto, pajak, dan netto pada setiap penerima Honorarium dihitung realtime saat volume, kategori pajak, atau tarif PPh 21 diubah.
- Hasil perhitungan tampil pada card penerima yang sama dan total keseluruhan tetap diperbarui otomatis.
- Card `Total Nilai` pada menu Perencanaan User Bidang diubah menjadi `Total Pagu`.
- Menu Pengadaan Langsung, Pencatatan Pengadaan, dan Pencatatan Non Pengadaan menampilkan card `Total Realisasi` berdasarkan nilai yang sudah sah/disetujui.
- Enam card tetap berada dalam satu baris pada layar desktop dan berubah responsif pada layar lebih kecil.
- Seluruh perubahan kartu memakai state lokal; tidak ada request tambahan atau reload dashboard penuh setelah aksi kecil.

## File yang berubah

- `app.js`
- `style.css`
- `index.html`
- `README.md`

## Deployment

`Code.gs` tidak berubah dari v162. Jika Code.gs v162 sudah di-deploy, tidak perlu Deploy New Version untuk patch v163 ini. Frontend GitHub Pages tetap harus diperbarui agar cache-bust v163 aktif.

## Kompatibilitas

Patch bersifat aditif dan tidak mengubah endpoint, nama sheet, ID kegiatan, struktur data, role, maupun dokumen lama.

# SIMPROV v164 — Realtime Card Surat dan Navigasi Paket Pembayaran

## Perbaikan

- Card `Total Surat`, `Draft`, `Menunggu Proses`, `Selesai`, dan `Perlu Perbaikan` langsung diperbarui setelah surat disimpan, diajukan, diproses, dihapus, atau selesai disinkronkan dari server.
- Pembaruan card memakai `suratWorkspaceV133` yang sudah diterima frontend; tidak mengambil ulang seluruh dashboard dan tidak reload halaman.
- Setiap kartu pada `Daftar Pengajuan Pembayaran` menampilkan `ID Paket` di samping judul sebagai teks klik sederhana tanpa border.
- Ditambahkan tombol `Lihat Paket` pada detail pengajuan pembayaran.
- Klik ID Paket atau tombol Lihat Paket langsung membuka paket sumber pada menu yang sesuai:
  - Non Pengadaan → Pencatatan Non Pengadaan;
  - Pengadaan Langsung/Tender Manual → Pengadaan Langsung;
  - Belanja Langsung → Pencatatan Pengadaan.
- Navigasi menggunakan state lokal `paketAktifV95` dan tidak melakukan reload dashboard penuh.
- Cache bust frontend diperbarui menjadi `1640-realtime-surat-lihat-paket`.

## File yang berubah

- `app.js`
- `style.css`
- `index.html`
- `README.md`

## Deployment

`Code.gs` tidak berubah dari v162/v163. Jika backend v162 sudah di-deploy, tidak perlu Deploy New Version Apps Script. Perbarui file frontend pada GitHub Pages.


# SIMPROV v164.1 — Pembayaran Terkelompok dan Progress Bendahara

## Perbaikan

- Menggunakan kembali baseline SIMPROV v164 yang diunggah pengguna.
- Daftar Pengajuan Pembayaran dipisahkan menjadi dua tab yang mudah dipahami: `Pengadaan Langsung` dan `Pencatatan Pengadaan`.
- Setiap tab menampilkan jumlah pengajuan dan keterangan singkat sumber paket. Perpindahan tab menggunakan state lokal tanpa request tambahan.
- Progress upload bukti pembayaran Bendahara bergerak dari 0% hingga 100% saat file dibaca, dikirim, disimpan, dan paket diselesaikan.
- Setelah pembayaran berhasil, status paket langsung diperbarui pada state lokal lalu disinkronkan ringan ke server tanpa reload dashboard penuh.
- Tombol `Buka Bukti Pembayaran` disamakan dengan style `SPTJM` dan `Lembar Verifikasi`.
- Tombol `Lihat Paket` disembunyikan khusus role Bendahara. ID paket tetap tampil sesuai baseline v164.

## File yang berubah

- `app.js`
- `style.css`
- `index.html`
- `README.md`

## Deployment

`Code.gs` tidak berubah dari baseline v164 yang diunggah. Tidak perlu Deploy New Version Apps Script. Cukup perbarui file frontend GitHub Pages dan pastikan cache-bust v164.1 aktif.


## SIMPROV v164.2 — UI Pembayaran, Riwayat, Refresh, dan Logout

- Tab Pengadaan Langsung dan Pencatatan Pengadaan menggunakan latar terang dan teks kontras.
- Riwayat Verifikasi Keuangan, Antrean Pembayaran Bendahara, dan Riwayat Pembayaran Bendahara dibagi menurut sumber paket tanpa request tambahan saat berpindah tab.
- Badge Antrean Pembayaran Bendahara berwarna putih saat 0 dan merah saat lebih dari 0.
- Tombol Buka Bukti Pembayaran mengikuti tampilan tombol Nota Dinas, SPTJM, dan Lembar Verifikasi.
- Tombol Refresh pada halaman riwayat/antrean mengambil workspace pembayaran secara ringan lalu merender halaman aktif.
- Logout membersihkan tampilan dan sesi lokal segera; permintaan pemutusan sesi server dijalankan tanpa menahan UI.
- Code.gs tidak diubah dari baseline v164.1.


## SIMPROV v164.3 — Tahap Input Nilai Realisasi Pengadaan Langsung

- Klik Tahap Pembayaran langsung membuka dan menyorot kartu pengajuan pembayaran paket terkait.
- Tahapan Pengadaan Langsung menjadi 8 tahap; Tahap 8 khusus Input Nilai Realisasi.
- Pembayaran Bendahara tidak lagi otomatis menutup paket Pengadaan Langsung.
- Setelah pembayaran selesai, User Bidang mengisi nilai realisasi sebenarnya; Nilai HPS hanya referensi dan tidak dipakai sebagai realisasi.
- Nilai realisasi dibatasi tidak melebihi Pagu Perencanaan dan diperiksa Verifikator PBJ.
- Setelah nilai disetujui, paket menjadi SELESAI dan nilai masuk ke Total Realisasi.
- Paket lama yang sudah berstatus SELESAI tetapi belum memiliki realisasi tetap diarahkan ke Tahap 8 tanpa migrasi destruktif.
- Aksi simpan dan pemeriksaan memperbarui state lokal tanpa reload dashboard penuh.

### Deployment

Code.gs berubah dan wajib Deploy New Version Apps Script. Frontend GitHub Pages juga harus diperbarui agar cache-bust v164.3 aktif.
