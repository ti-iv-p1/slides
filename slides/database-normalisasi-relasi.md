---
title: Tutorial Database - Normalisasi dan Relasi
version: 1.0.0
header: Tutorial Database - Normalisasi dan Relasi
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Tutorial Database

## Normalisasi dan Relasi

---

## Daftar Isi

1. Pengantar Database
2. Apa itu Normalisasi Database?
3. Bentuk-Bentuk Normal (Normal Forms)
4. Contoh Proses Normalisasi
5. Relasi Database
6. Contoh Implementasi
7. Best Practices

---

<!--
_class: lead
-->

# Pengantar Database

---

## Apa itu Database?

**Database** atau **basis data** adalah kumpulan data yang terorganisir dan tersimpan secara sistematis dalam komputer.

**Database relasional** menggunakan tabel-tabel yang saling berhubungan untuk menyimpan dan mengelola data.

---

## Mengapa Normalisasi Penting?

Tanpa normalisasi yang baik, database dapat mengalami masalah:

- ❌ **Redundansi data** (data yang berulang)
- ❌ **Anomali update** (kesulitan mengubah data)
- ❌ **Anomali insert** (kesulitan menambah data)
- ❌ **Anomali delete** (kehilangan data yang tidak diinginkan)
- ❌ **Ketidakkonsistenan data**

---

<!--
_class: lead
-->

# Apa itu Normalisasi Database?

---

## Definisi Normalisasi

**Normalisasi** adalah proses mengorganisir data dalam database untuk mengurangi redundansi dan meningkatkan integritas data.

Normalisasi dilakukan dengan **memecah tabel besar** menjadi tabel-tabel yang lebih kecil dan mendefinisikan **relasi** di antara mereka.

---

## Tujuan Normalisasi

1. ✅ Menghilangkan redundansi data
2. ✅ Memastikan dependensi data yang logis
3. ✅ Mempermudah pemeliharaan database
4. ✅ Mengoptimalkan performa query

---

<!--
_class: lead
-->

# Bentuk-Bentuk Normal

## (Normal Forms)

---

## First Normal Form (1NF)

Suatu tabel dikatakan dalam **1NF** jika:

- Setiap kolom berisi nilai **atomik** (tidak dapat dibagi lagi)
- Setiap baris memiliki data yang **unik**
- Tidak ada **grup berulang** dalam satu kolom

---

## Contoh Pelanggaran 1NF

**❌ Sebelum 1NF:**

| ID  | Nama | Hobi                     |
| --- | ---- | ------------------------ |
| 1   | Budi | Sepak bola, Musik        |
| 2   | Ani  | Membaca, Menulis, Renang |

**Masalah:** Kolom `Hobi` berisi multiple values (tidak atomik)

---

## Perbaikan ke 1NF

**✅ Setelah 1NF:**

| ID  | Nama | Hobi       |
| --- | ---- | ---------- |
| 1   | Budi | Sepak bola |
| 1   | Budi | Musik      |
| 2   | Ani  | Membaca    |
| 2   | Ani  | Menulis    |
| 2   | Ani  | Renang     |

---

## Second Normal Form (2NF)

Suatu tabel dikatakan dalam **2NF** jika:

- Sudah memenuhi **1NF**
- Semua atribut non-kunci bergantung **sepenuhnya** pada primary key
- Tidak ada **partial dependency**

---

## Contoh Pelanggaran 2NF

**❌ Sebelum 2NF:**

| ID_Siswa | ID_Mata_Kuliah | Nama_Siswa | Nilai | Nama_Mata_Kuliah |
| -------- | -------------- | ---------- | ----- | ---------------- |
| 1        | 101            | Budi       | A     | Database         |
| 1        | 102            | Budi       | B     | Pemrograman Web  |
| 2        | 101            | Ani        | A     | Database         |

**Masalah:**

- `Nama_Siswa` hanya bergantung pada `ID_Siswa`
- `Nama_Mata_Kuliah` hanya bergantung pada `ID_Mata_Kuliah`

---

## Perbaikan ke 2NF

**✅ Setelah 2NF:**

**Tabel Siswa:**
| ID_Siswa | Nama_Siswa |
|----------|------------|
| 1 | Budi |
| 2 | Ani |

**Tabel Mata_Kuliah:**
| ID_Mata_Kuliah | Nama_Mata_Kuliah |
|----------------|------------------|
| 101 | Database |
| 102 | Pemrograman Web |

---

## Perbaikan ke 2NF (lanjutan)

**Tabel Nilai:**
| ID_Siswa | ID_Mata_Kuliah | Nilai |
|----------|----------------|-------|
| 1 | 101 | A |
| 1 | 102 | B |
| 2 | 101 | A |

---

## Third Normal Form (3NF)

Suatu tabel dikatakan dalam **3NF** jika:

- Sudah memenuhi **2NF**
- Tidak ada **transitive dependency**
  (atribut non-kunci tidak bergantung pada atribut non-kunci lainnya)

---

## Contoh Pelanggaran 3NF

**❌ Sebelum 3NF:**

| ID_Siswa | Nama | Kota    | Kode_Pos | Provinsi   |
| -------- | ---- | ------- | -------- | ---------- |
| 1        | Budi | Jakarta | 12345    | DKI        |
| 2        | Ani  | Bandung | 40111    | Jawa Barat |

**Masalah:** `Provinsi` bergantung pada `Kota`, bukan pada `ID_Siswa`

---

## Perbaikan ke 3NF

**✅ Setelah 3NF:**

**Tabel Siswa:**
| ID_Siswa | Nama | Kode_Pos |
|----------|------|----------|
| 1 | Budi | 12345 |
| 2 | Ani | 40111 |

**Tabel Kota:**
| Kode_Pos | Kota | Provinsi |
|----------|------|----------|
| 12345 | Jakarta | DKI |
| 40111 | Bandung | Jawa Barat |

---

## Boyce-Codd Normal Form (BCNF)

Suatu tabel dikatakan dalam **BCNF** jika:

- Sudah memenuhi **3NF**
- Untuk setiap functional dependency A → B, A harus **super key**

BCNF adalah versi yang lebih ketat dari 3NF dan menangani anomali khusus yang jarang terjadi.

---

<!--
_class: lead
-->

# Contoh Proses Normalisasi

---

## Tabel Awal (Tidak Ternormalisasi)

**Tabel Pemesanan:**

| ID_Pesanan | Tanggal    | Nama_Customer | Telp_Customer | Alamat_Customer | Produk          | Harga_Produk  | Jumlah | Total   |
| ---------- | ---------- | ------------- | ------------- | --------------- | --------------- | ------------- | ------ | ------- |
| 1          | 2026-03-01 | John Doe      | 081234567890  | Jl. Merdeka 1   | Mouse, Keyboard | 50000, 150000 | 2, 1   | 250000  |
| 2          | 2026-03-02 | Jane Smith    | 081298765432  | Jl. Sudirman 5  | Monitor         | 1000000       | 1      | 1000000 |

---

## Langkah 1: Menerapkan 1NF

Pisahkan nilai yang tidak atomik:

| ID_Pesanan | Tanggal    | Nama_Customer | Telp_Customer | Alamat_Customer | Produk   | Harga_Produk | Jumlah | Total   |
| ---------- | ---------- | ------------- | ------------- | --------------- | -------- | ------------ | ------ | ------- |
| 1          | 2026-03-01 | John Doe      | 081234567890  | Jl. Merdeka 1   | Mouse    | 50000        | 2      | 100000  |
| 1          | 2026-03-01 | John Doe      | 081234567890  | Jl. Merdeka 1   | Keyboard | 150000       | 1      | 150000  |
| 2          | 2026-03-02 | Jane Smith    | 081298765432  | Jl. Sudirman 5  | Monitor  | 1000000      | 1      | 1000000 |

---

## Langkah 2: Menerapkan 2NF (1/3)

**Tabel Pesanan:**
| ID_Pesanan | Tanggal | ID_Customer |
|------------|---------|-------------|
| 1 | 2026-03-01 | 1 |
| 2 | 2026-03-02 | 2 |

**Tabel Customer:**
| ID_Customer | Nama_Customer | Telp_Customer | Alamat_Customer |
|-------------|---------------|---------------|-----------------|
| 1 | John Doe | 081234567890 | Jl. Merdeka 1 |
| 2 | Jane Smith | 081298765432 | Jl. Sudirman 5 |

---

## Langkah 2: Menerapkan 2NF (2/3)

**Tabel Produk:**
| ID_Produk | Nama_Produk | Harga |
|-----------|-------------|-------|
| 1 | Mouse | 50000 |
| 2 | Keyboard | 150000 |
| 3 | Monitor | 1000000 |

---

## Langkah 2: Menerapkan 2NF (3/3)

**Tabel Detail_Pesanan:**
| ID_Pesanan | ID_Produk | Jumlah | Subtotal |
|------------|-----------|--------|----------|
| 1 | 1 | 2 | 100000 |
| 1 | 2 | 1 | 150000 |
| 2 | 3 | 1 | 1000000 |

---

## Langkah 3: Menerapkan 3NF

Dalam kasus ini, tabel sudah memenuhi 3NF karena tidak ada transitive dependency.

`Subtotal` sebenarnya dapat dihitung (`Jumlah × Harga`), jadi bisa dihapus untuk menghindari redundansi:

**Tabel Detail_Pesanan (Final):**
| ID_Pesanan | ID_Produk | Jumlah |
|------------|-----------|--------|
| 1 | 1 | 2 |
| 1 | 2 | 1 |
| 2 | 3 | 1 |

---

<!--
_class: lead
-->

# Relasi Database

---

## Jenis-Jenis Relasi

Ada tiga jenis relasi utama dalam database:

1. **One-to-One (1:1)**
2. **One-to-Many (1:N)**
3. **Many-to-Many (M:N)**

---

## One-to-One (1:1)

**Satu record** dalam tabel A berhubungan dengan **tepat satu record** dalam tabel B.

**Contoh:** Satu siswa memiliki satu KTP.

```
Siswa (1) ←→ (1) KTP
```

---

## Contoh One-to-One

**Tabel Siswa:**
| ID_Siswa | Nama | Email |
|----------|------|-------|
| 1 | Budi | budi@email.com |
| 2 | Ani | ani@email.com |

**Tabel KTP:**
| ID_KTP | ID_Siswa | Nomor_KTP | Alamat_KTP |
|--------|----------|-----------|------------|
| 1 | 1 | 3201010101010001 | Jl. Merdeka 1 |
| 2 | 2 | 3202020202020002 | Jl. Sudirman 5 |

---

## One-to-Many (1:N)

**Satu record** dalam tabel A dapat berhubungan dengan **banyak record** dalam tabel B, tetapi satu record dalam tabel B hanya berhubungan dengan **satu record** dalam tabel A.

**Contoh:** Satu dosen dapat mengajar banyak mata kuliah, tetapi satu mata kuliah hanya diajar oleh satu dosen.

```
Dosen (1) ←→ (N) Mata_Kuliah
```

---

## Contoh One-to-Many

**Tabel Dosen:**
| ID_Dosen | Nama_Dosen | Email |
|----------|------------|-------|
| 1 | Dr. Ahmad | ahmad@univ.ac.id |
| 2 | Dr. Siti | siti@univ.ac.id |

**Tabel Mata_Kuliah:**
| ID_Mata_Kuliah | Nama_Mata_Kuliah | ID_Dosen | SKS |
|----------------|------------------|----------|-----|
| 101 | Database | 1 | 3 |
| 102 | Pemrograman Web | 1 | 3 |
| 103 | Jaringan | 2 | 3 |

---

## Many-to-Many (M:N)

**Banyak record** dalam tabel A dapat berhubungan dengan **banyak record** dalam tabel B, dan sebaliknya.

**Contoh:** Banyak siswa dapat mengambil banyak mata kuliah, dan satu mata kuliah dapat diambil oleh banyak siswa.

**Implementasi:** Relasi M:N membutuhkan **junction table** (tabel penghubung).

```
Siswa (M) ←→ Siswa_Mata_Kuliah ←→ (N) Mata_Kuliah
```

---

## Contoh Many-to-Many (1/2)

**Tabel Siswa:**
| ID_Siswa | Nama | NIM |
|----------|------|---------|
| 1 | Budi | 20230001 |
| 2 | Ani | 20230002 |
| 3 | Dedi | 20230003 |

**Tabel Mata_Kuliah:**
| ID_Mata_Kuliah | Nama_Mata_Kuliah | SKS |
|----------------|------------------|-----|
| 101 | Database | 3 |
| 102 | Pemrograman Web | 3 |
| 103 | Jaringan | 3 |

---

## Contoh Many-to-Many (2/2)

**Tabel Siswa_Mata_Kuliah (Junction Table):**
| ID_Siswa | ID_Mata_Kuliah | Nilai | Semester |
|----------|----------------|-------|----------|
| 1 | 101 | A | 3 |
| 1 | 102 | B | 3 |
| 2 | 101 | A | 3 |
| 2 | 103 | B | 3 |
| 3 | 102 | A | 3 |

---

<!--
_class: lead
-->

# Contoh Implementasi SQL

---

## Membuat Tabel Mahasiswa

```sql
CREATE TABLE Mahasiswa (
    id_mahasiswa INT PRIMARY KEY AUTO_INCREMENT,
    nim VARCHAR(10) UNIQUE NOT NULL,
    nama VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    tanggal_lahir DATE,
    alamat TEXT,
    id_jurusan INT,
    FOREIGN KEY (id_jurusan) REFERENCES Jurusan(id_jurusan)
);
```

---

## Membuat Tabel Jurusan

```sql
-- One-to-Many dengan Mahasiswa
CREATE TABLE Jurusan (
    id_jurusan INT PRIMARY KEY AUTO_INCREMENT,
    kode_jurusan VARCHAR(10) UNIQUE NOT NULL,
    nama_jurusan VARCHAR(100) NOT NULL,
    fakultas VARCHAR(100)
);
```

---

## Membuat Tabel Dosen dan Mata Kuliah

```sql
CREATE TABLE Dosen (
    id_dosen INT PRIMARY KEY AUTO_INCREMENT,
    nip VARCHAR(20) UNIQUE NOT NULL,
    nama VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    spesialisasi VARCHAR(100)
);

CREATE TABLE Mata_Kuliah (
    id_mata_kuliah INT PRIMARY KEY AUTO_INCREMENT,
    kode_mk VARCHAR(10) UNIQUE NOT NULL,
    nama_mk VARCHAR(100) NOT NULL,
    sks INT NOT NULL,
    id_dosen INT,
    FOREIGN KEY (id_dosen) REFERENCES Dosen(id_dosen)
);
```

---

## Membuat Junction Table

```sql
-- Many-to-Many relationship
CREATE TABLE Pengambilan_MK (
    id_pengambilan INT PRIMARY KEY AUTO_INCREMENT,
    id_mahasiswa INT,
    id_mata_kuliah INT,
    semester INT,
    tahun_ajaran VARCHAR(10),
    nilai_angka DECIMAL(5,2),
    nilai_huruf CHAR(2),
    FOREIGN KEY (id_mahasiswa) REFERENCES Mahasiswa(id_mahasiswa),
    FOREIGN KEY (id_mata_kuliah) REFERENCES Mata_Kuliah(id_mata_kuliah),
    UNIQUE KEY unique_pengambilan (id_mahasiswa, id_mata_kuliah,
                                   semester, tahun_ajaran)
);
```

---

## Contoh Query: JOIN

```sql
-- Menampilkan semua mahasiswa dengan jurusannya
SELECT
    m.nim,
    m.nama,
    m.email,
    j.nama_jurusan,
    j.fakultas
FROM Mahasiswa m
JOIN Jurusan j ON m.id_jurusan = j.id_jurusan
ORDER BY m.nama;
```

---

## Contoh Query: Multiple JOIN

```sql
-- Menampilkan mata kuliah yang diambil oleh seorang mahasiswa
SELECT
    m.nama AS nama_mahasiswa,
    mk.kode_mk,
    mk.nama_mk,
    mk.sks,
    p.nilai_huruf,
    d.nama AS nama_dosen
FROM Pengambilan_MK p
JOIN Mahasiswa m ON p.id_mahasiswa = m.id_mahasiswa
JOIN Mata_Kuliah mk ON p.id_mata_kuliah = mk.id_mata_kuliah
JOIN Dosen d ON mk.id_dosen = d.id_dosen
WHERE m.nim = '20230001'
ORDER BY mk.nama_mk;
```

---

## Contoh Query: Agregasi

```sql
-- Menghitung IPK mahasiswa
SELECT
    m.nim,
    m.nama,
    AVG(
        CASE p.nilai_huruf
            WHEN 'A' THEN 4.0
            WHEN 'AB' THEN 3.5
            WHEN 'B' THEN 3.0
            WHEN 'BC' THEN 2.5
            WHEN 'C' THEN 2.0
            WHEN 'D' THEN 1.0
            WHEN 'E' THEN 0.0
            ELSE 0
        END
    ) AS IPK
FROM Mahasiswa m
JOIN Pengambilan_MK p ON m.id_mahasiswa = p.id_mahasiswa
GROUP BY m.id_mahasiswa, m.nim, m.nama
ORDER BY IPK DESC;
```

---

## Contoh Query: COUNT dengan LEFT JOIN

```sql
-- Menampilkan mata kuliah dengan jumlah mahasiswanya
SELECT
    mk.kode_mk,
    mk.nama_mk,
    d.nama AS dosen_pengampu,
    COUNT(p.id_mahasiswa) AS jumlah_mahasiswa
FROM Mata_Kuliah mk
LEFT JOIN Pengambilan_MK p ON mk.id_mata_kuliah = p.id_mata_kuliah
JOIN Dosen d ON mk.id_dosen = d.id_dosen
GROUP BY mk.id_mata_kuliah, mk.kode_mk, mk.nama_mk, d.nama
ORDER BY jumlah_mahasiswa DESC;
```

---

## Entity Relationship Diagram (ERD)

```
┌─────────────┐         ┌──────────────┐
│  Jurusan    │         │    Dosen     │
├─────────────┤         ├──────────────┤
│ id_jurusan  │         │ id_dosen     │
│ kode_jurusan│         │ nip          │
│ nama_jurusan│         │ nama         │
│ fakultas    │         │ spesialisasi │
└──────┬──────┘         └──────┬───────┘
       │ 1:N                   │ 1:N
       │                       │
┌──────┴──────┐         ┌──────┴──────────┐
│  Mahasiswa  │         │  Mata_Kuliah    │
├─────────────┤         ├─────────────────┤
│id_mahasiswa │         │ id_mata_kuliah  │
│ nim         │         │ kode_mk         │
│ nama        │    M:N  │ nama_mk         │
│ email       │◄───────►│ sks             │
│id_jurusan(FK)│        │ id_dosen (FK)   │
└─────────────┘         └─────────────────┘
                   │
           ┌───────┴──────────┐
           │ Pengambilan_MK   │
           ├──────────────────┤
           │ id_mahasiswa (FK)│
           │id_mata_kuliah(FK)│
           │ semester         │
           │ nilai_huruf      │
           └──────────────────┘
```

---

<!--
_class: lead
-->

# Best Practices

---

## 1. Normalisasi yang Seimbang

Meskipun normalisasi penting, terkadang **denormalisasi** diperlukan untuk performa.

**Pertimbangkan:**

- **Query yang sering dilakukan:** Jika join terlalu banyak memperlambat query, pertimbangkan menyimpan beberapa data redundan
- **Read vs Write:** Database dengan banyak read benefit dari denormalisasi, sedangkan database dengan banyak write benefit dari normalisasi

---

## 2. Penggunaan Foreign Key

Selalu gunakan **foreign key constraint** untuk memastikan referential integrity:

```sql
FOREIGN KEY (id_column) REFERENCES parent_table(id_column)
    ON DELETE CASCADE
    ON UPDATE CASCADE
```

---

## 3. Indexing

Buat **index** pada kolom yang sering digunakan untuk:

- **Primary key** (otomatis)
- **Foreign key**
- Kolom yang sering di-query (WHERE, JOIN)

```sql
CREATE INDEX idx_mahasiswa_nim ON Mahasiswa(nim);
CREATE INDEX idx_mata_kuliah_kode ON Mata_Kuliah(kode_mk);
```

---

## 4. Naming Convention

Gunakan **naming convention** yang konsisten:

- **Tabel:** Gunakan singular atau plural, tetapi konsisten
- **Primary Key:** `id_nama_tabel` atau hanya `id`
- **Foreign Key:** `id_nama_tabel_referensi`
- **Junction Table:** `tabel1_tabel2` atau nama yang deskriptif

---

<!--
_class: lead
-->

# Kesimpulan

---

## Ringkasan

Normalisasi database adalah teknik fundamental dalam desain database yang bertujuan untuk:

1. ✅ Mengurangi redundansi dan menghemat storage
2. ✅ Meningkatkan integritas data dengan menghindari anomali
3. ✅ Mempermudah maintenance database
4. ✅ Mengoptimalkan performa query

---

## Poin-Poin Penting

- **1NF:** Nilai atomik, tidak ada grup berulang
- **2NF:** 1NF + tidak ada partial dependency
- **3NF:** 2NF + tidak ada transitive dependency
- **Relasi:** One-to-One, One-to-Many, Many-to-Many
- **Junction Table:** Diperlukan untuk relasi Many-to-Many

Dengan memahami normalisasi dan relasi, Anda dapat mendesain database yang **efisien**, **scalable**, dan **mudah dimaintain**.

---

<!--
_class: lead
-->

# Terima Kasih!

**Referensi:**

- Tutorial: https://ti-iv-p1.github.io/blog/
- GitHub: https://github.com/ti-iv-p1/slides

---
