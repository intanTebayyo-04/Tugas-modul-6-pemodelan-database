# Laporan Tugas Mandiri: Perancangan ERD E-Library Kampus
**Mata Kuliah:** Pemrograman Website  
**Nama:** Intan  
**NIM:** D121241086  
**Program Studi:** Teknik Informatika  
**Universitas Hasanuddin**  

---

## 1. Skenario Sistem E-Library Kampus
Sistem Basis Data Perpustakaan Kampus (*E-Library*) dirancang untuk mengelola sirkulasi peminjaman dan pengembalian buku oleh mahasiswa. Sistem ini merekam data entitas utama yang meliputi **Mahasiswa**, **Penerbit**, **Buku**, serta aktivitas **Transaksi Peminjaman**. Setiap mahasiswa dapat meminjam beberapa buku, dan satu buku dapat diterbitkan oleh satu penerbit tertentu.

---

## 2. Simulasi Normalisasi (UNF ke 3NF)

### A. Bentuk Tidak Normal (Unnormalized Form - UNF)
Tabel awal mencakup seluruh atribut yang saling berulang dan belum dipecah ke dalam struktur atomik.

| NIM | Nama_Mhs | Alamat_Mhs | ID_Buku | Judul_Buku | Tahun_Terbit | Nama_Penerbit | Kota_Penerbit | Tanggal_Pinjam | Tanggal_Kembali | Status_Pinjam |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| D121241086 | Intan | Tamalanrea | B001, B002 | Algoritma, Basis Data | 2023, 2024 | Penerbit Andi, Gramedia | Yogyakarta, Jakarta | 2026-10-01 | 2026-10-08 | Dipinjam |

---

### B. Bentuk Normal Pertama (1NF)
Menghilangkan kelompok atribut berulang dalam satu sel dan memastikan setiap sel bernilai atomik (tunggal). Kunci primer komposit sementara menggunakan gabungan `NIM` dan `ID_Buku`.

| NIM (PK-1) | Nama_Mhs | Alamat_Mhs | ID_Buku (PK-2) | Judul_Buku | Tahun_Terbit | Nama_Penerbit | Kota_Penerbit | Tanggal_Pinjam | Tanggal_Kembali | Status_Pinjam |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| D121241086 | Intan | Tamalanrea | B001 | Algoritma | 2023 | Penerbit Andi | Yogyakarta | 2026-10-01 | 2026-10-08 | Dipinjam |
| D121241086 | Intan | Tamalanrea | B002 | Basis Data | 2024 | Gramedia | Jakarta | 2026-10-02 | 2026-10-09 | Selesai |

---

### C. Bentuk Normal Kedua (2NF)
Memenuhi 1NF dan menghilangkan **ketergantungan parsial**, di mana atribut bukan kunci hanya bergantung pada sebagian dari kunci komposit. Atribut terkait mahasiswa dipisah ke tabel `mahasiswa`, dan atribut penerbit dipisah untuk menghindari redundansi penerbit.

* **Tabel `mahasiswa`** (PK: `nim`)
* **Tabel `penerbit`** (PK: `id_penerbit`)
* **Tabel `buku`** (PK: `id_buku`, FK: `id_penerbit`)
* **Tabel `peminjaman`** (PK Komposit: `nim` + `id_buku` + `tanggal_pinjam`)

---

### D. Bentuk Normal Ketiga (3NF)
Memenuhi 2NF serta **menghilangkan ketergantungan transitif** (atribut bukan kunci bergantung pada atribut bukan kunci lainnya). Pada tahap ini, informasi penerbit dipetakan secara penuh melalui relasi `id_penerbit` di dalam tabel `buku`, dan entitas transaksi dipisahkan dengan jelas menjadi tabel relasional mandiri (`peminjaman`).

---

## 3. Visualisasi Relasi Kunci (Diagram Mermaid)

```mermaid
erDiagram
    PENERBIT {
        string id_penerbit PK
        string nama_penerbit
        string kota_penerbit
    }
    BUKU {
        string id_buku PK
        string judul_buku
        int tahun_terbit
        string id_penerbit FK
    }
    MAHASISWA {
        string nim PK
        string nama_mhs
        text alamat_mhs
    }
    PEMINJAMAN {
        string id_transaksi PK
        string nim FK
        string id_buku FK
        date tanggal_pinjam
        date tanggal_kembali
        string status_pinjam
    }

    PENERBIT ||--o{ BUKU : "menerbitkan"
    MAHASISWA ||--o{ PEMINJAMAN : "melakukan"
    BUKU ||--o{ PEMINJAMAN : "dipinjam_dalam" 

'''sql
-- Skema DDL E-Library Kampus
CREATE DATABASE elibrary_db;
USE elibrary_db;

CREATE TABLE penerbit (
    id_penerbit VARCHAR(10) PRIMARY KEY,
    nama_penerbit VARCHAR(100) NOT NULL,
    kota_penerbit VARCHAR(50)
);

CREATE TABLE buku (
    id_buku VARCHAR(10) PRIMARY KEY,
    judul_buku VARCHAR(150) NOT NULL,
    tahun_terbit INT NOT NULL,
    id_penerbit VARCHAR(10),
    FOREIGN KEY (id_penerbit) REFERENCES penerbit(id_penerbit) ON DELETE SET NULL
);

CREATE TABLE mahasiswa (
    nim VARCHAR(10) PRIMARY KEY,
    nama_mhs VARCHAR(100) NOT NULL,
    alamat_mhs TEXT
);

CREATE TABLE peminjaman (
    id_transaksi VARCHAR(15) PRIMARY KEY,
    nim VARCHAR(10) NOT NULL,
    id_buku VARCHAR(10) NOT NULL,
    tanggal_pinjam DATE NOT NULL,
    tanggal_kembali DATE,
    status_pinjam VARCHAR(20) NOT NULL,
    FOREIGN KEY (nim) REFERENCES mahasiswa(nim) ON DELETE CASCADE,
    FOREIGN KEY (id_buku) REFERENCES buku(id_buku) ON DELETE CASCADE
);