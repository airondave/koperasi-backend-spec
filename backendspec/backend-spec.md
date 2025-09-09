# Backend API Specification - Sistem Koperasi Merah Putih

Dokumen ini mendefinisikan kebutuhan backend untuk sistem koperasi, termasuk endpoint API, data model, business logic, serta role & permission untuk setiap modul.

---

## 1. Authentication & User Management

### Data Model: user
- `id` (int, PK)
- `nama` (string)
- `email` (string, unique)
- `password` (hashed)
- `role` (enum: admin, manager, kasir, anggota)
- `created_at` (datetime)

### Endpoint
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/logout`
- `GET /auth/me`

### Business Logic
- Password harus di-hash (bcrypt).
- Autentikasi pakai JWT token.
- Role menentukan izin akses modul lain.

---

## 2. Website Company Profile & Publikasi

### Data Model
- **artikel**
  - `id`, `judul`, `isi`, `penulis`, `created_at`
- **galeri**
  - `id`, `url_gambar`, `deskripsi`
- **newsletter**
  - `id`, `email`
- **faq**
  - `id`, `pertanyaan`, `jawaban`

### Endpoint
- `GET /artikel`
- `GET /artikel/:id`
- `POST /artikel` (admin)
- `PUT /artikel/:id` (admin)
- `DELETE /artikel/:id` (admin)

- `GET /galeri`
- `POST /newsletter` (public)
- `GET /faq`

### Business Logic
- Artikel & galeri hanya admin yang bisa CRUD.
- Newsletter terbuka untuk publik.

---

## 3. Manajemen Keuangan

### Data Model
- **transaksi**
  - `id`, `jenis` (pemasukan/pengeluaran), `jumlah`, `deskripsi`, `tanggal`
- **anggaran**
  - `id`, `tahun`, `kategori`, `jumlah`
- **shu**
  - `id`, `tahun`, `total_shu`, `dibagikan`

### Endpoint
- `POST /transaksi`
- `GET /transaksi`
- `GET /laporan/keuangan`
- `POST /anggaran`
- `GET /shu/distribusi`

### Business Logic
- SHU dihitung otomatis berdasarkan transaksi.
- Laporan bisa diexport (PDF/Excel).

---

## 4. Manajemen Anggota

### Data Model
- **anggota**
  - `id`, `nama`, `email`, `password`, `alamat`, `telepon`, `status`, `tanggal_daftar`

### Endpoint
- `POST /anggota/register`
- `GET /anggota`
- `GET /anggota/:id`
- `PUT /anggota/:id`
- `DELETE /anggota/:id`

### Business Logic
- Email unik.
- Soft delete → status jadi inactive.
- Anggota bisa akses profil sendiri.

---

## 5. Pinjaman

### Data Model
- **pinjaman**
  - `id`, `anggota_id`, `jumlah`, `bunga`, `jangka_waktu`, `status`, `tanggal_pengajuan`, `tanggal_disetujui`

### Endpoint
- `POST /pinjaman/apply`
- `GET /pinjaman`
- `GET /pinjaman/:id`
- `PUT /pinjaman/:id/approve`
- `PUT /pinjaman/:id/reject`
- `POST /pinjaman/:id/bayar`

### Business Logic
- Validasi kelayakan (misalnya cek riwayat).
- Bunga dihitung otomatis.
- Status = pending → approved/rejected.

---

## 6. Tabungan & Nabung

### Data Model
- **tabungan**
  - `id`, `anggota_id`, `saldo`, `jenis`, `tanggal_buka`
- **transaksi_tabungan**
  - `id`, `tabungan_id`, `jenis` (setor/tarik), `jumlah`, `tanggal`

### Endpoint
- `POST /tabungan`
- `GET /tabungan/:id`
- `POST /tabungan/:id/setor`
- `POST /tabungan/:id/tarik`
- `GET /tabungan/:id/laporan`

### Business Logic
- Tidak boleh tarik lebih besar dari saldo.
- Laporan saldo per anggota tersedia.

---

## 7. Pembagian SHU

### Data Model
- **shu_distribusi**
  - `id`, `anggota_id`, `tahun`, `jumlah`

### Endpoint
- `POST /shu/hitung`
- `GET /shu/distribusi/:tahun`

### Business Logic
- Perhitungan berdasarkan kontribusi transaksi.
- Hasil distribusi transparan per anggota.

---

## 8. Pengajuan Online

### Data Model
- **pengajuan**
  - `id`, `anggota_id`, `jenis` (pinjaman/produk/layanan), `status`, `tanggal`

### Endpoint
- `POST /pengajuan`
- `GET /pengajuan`
- `GET /pengajuan/:id`
- `PUT /pengajuan/:id/approve`
- `PUT /pengajuan/:id/reject`

### Business Logic
- Notifikasi dikirim ke anggota saat status berubah.

---

## 9. POS (Point of Sale)

### Data Model
- **produk**
  - `id`, `nama`, `barcode`, `stok`, `harga`
- **penjualan**
  - `id`, `kasir_id`, `tanggal`, `total`
- **penjualan_detail**
  - `id`, `penjualan_id`, `produk_id`, `jumlah`, `harga_satuan`

### Endpoint
- `POST /pos/penjualan`
- `GET /pos/penjualan/:id`
- `GET /pos/laporan`
- `POST /produk`
- `PUT /produk/:id`
- `GET /produk`

### Business Logic
- Stok otomatis berkurang saat transaksi.
- Diskon bisa per produk/kategori.
- Mendukung pembayaran tunai, e-wallet, transfer.

---

## 10. Kebutuhan Sistem Umum

- **Integrasi**: API payment gateway, bank, e-wallet.  
- **Keamanan**: JWT auth, HTTPS, enkripsi data sensitif.  
- **Role**: Admin, Manager, Kasir, Anggota.  
- **Audit Trail**: setiap transaksi dicatat.  
- **Scalability**: siap untuk multi-cabang koperasi.  

