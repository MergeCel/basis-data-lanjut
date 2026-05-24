# Normalisasi Database Sistem Akademik hingga BCNF

Mata Kuliah: Basis Data Lanjut  
Topik: Normalisasi Database hingga BCNF  
Studi Kasus: Sistem Akademik Kampus


# Struktur Project

```plaintext
basis-data-lanjut/

├── docker-compose.yml
├── normalisasi.sql
├── README.md
└── screenshots/
```

---

# Tabel Tidak Normal

## Nama Tabel

```sql
akademik_tidak_normal
```

## Struktur Tabel

| nim | nama_mahasiswa | prodi | kode_mk | nama_mk | dosen | nilai |
|---|---|---|---|---|---|---|
| 221001 | Andi | Informatika | IF101 | Basis Data | Budi | A |
| 221001 | Andi | Informatika | IF102 | Pemrograman | Sinta | B |
| 221002 | Rina | Sistem Informasi | IF101 | Basis Data | Budi | A |

---

# Identifikasi Anomali

## 1. Update Anomaly

Jika nama dosen berubah, maka seluruh baris yang memiliki nama dosen tersebut harus diubah.

Contoh:

```sql
UPDATE akademik_tidak_normal
SET dosen = 'Dr. Budi'
WHERE dosen = 'Budi';
```

Masalah:
- Data dapat menjadi tidak konsisten jika ada baris yang tidak terupdate.

---

## 2. Insert Anomaly

Tidak dapat menambahkan mata kuliah baru tanpa adanya data mahasiswa.

Contoh:
- Mata kuliah AI tidak bisa dimasukkan jika belum ada mahasiswa yang mengambil mata kuliah tersebut.

---

## 3. Delete Anomaly

Jika data mahasiswa yang mengambil suatu mata kuliah dihapus, maka informasi mata kuliah juga ikut hilang.

Contoh:

```sql
DELETE FROM akademik_tidak_normal
WHERE kode_mk = 'IF102';
```

---

# Proses Normalisasi

## 1NF (First Normal Form)

Tabel sudah memenuhi 1NF karena:
- Data bersifat atomik
- Tidak ada multivalue attribute

---

## 2NF (Second Normal Form)

Masalah:
- Terdapat partial dependency

Contoh:
- nama_mahasiswa bergantung pada nim
- nama_mk dan dosen bergantung pada kode_mk

Solusi:
- Memisahkan data mahasiswa dan mata kuliah ke tabel terpisah

---

## 3NF (Third Normal Form)

Masalah:
- Terdapat transitive dependency

Contoh:
- kode_mk → dosen

Solusi:
- Membuat tabel dosen terpisah

---

## BCNF (Boyce-Codd Normal Form)

Database telah memenuhi BCNF karena:
- Setiap determinant adalah candidate key
- Tidak terdapat partial dependency
- Tidak terdapat transitive dependency

---

# Struktur Tabel Hasil BCNF

## Tabel mahasiswa

| nim | nama_mahasiswa | prodi |
|---|---|---|

---

## Tabel dosen

| id_dosen | nama_dosen |
|---|---|

---

## Tabel mata_kuliah

| kode_mk | nama_mk | id_dosen |
|---|---|---|

---

## Tabel krs

| nim | kode_mk | nilai |
|---|---|---|

---

# Implementasi SQL

## Membuat Database

```sql
CREATE DATABASE akademik_db;

USE akademik_db;
```

---

## Tabel Tidak Normal

```sql
CREATE TABLE akademik_tidak_normal (
    nim VARCHAR(10),
    nama_mahasiswa VARCHAR(100),
    prodi VARCHAR(100),
    kode_mk VARCHAR(10),
    nama_mk VARCHAR(100),
    dosen VARCHAR(100),
    nilai CHAR(2)
);
```

---

## Insert Data Awal

```sql
INSERT INTO akademik_tidak_normal VALUES
('221001', 'Andi', 'Informatika', 'IF101', 'Basis Data', 'Budi', 'A'),
('221001', 'Andi', 'Informatika', 'IF102', 'Pemrograman', 'Sinta', 'B'),
('221002', 'Rina', 'Sistem Informasi', 'IF101', 'Basis Data', 'Budi', 'A');
```

---

# Tabel Hasil BCNF

## Tabel mahasiswa

```sql
CREATE TABLE mahasiswa (
    nim VARCHAR(10) PRIMARY KEY,
    nama_mahasiswa VARCHAR(100),
    prodi VARCHAR(100)
);
```

---

## Tabel dosen

```sql
CREATE TABLE dosen (
    id_dosen INT AUTO_INCREMENT PRIMARY KEY,
    nama_dosen VARCHAR(100)
);
```

---

## Tabel mata_kuliah

```sql
CREATE TABLE mata_kuliah (
    kode_mk VARCHAR(10) PRIMARY KEY,
    nama_mk VARCHAR(100),
    id_dosen INT,
    FOREIGN KEY (id_dosen) REFERENCES dosen(id_dosen)
);
```

---

## Tabel krs

```sql
CREATE TABLE krs (
    nim VARCHAR(10),
    kode_mk VARCHAR(10),
    nilai CHAR(2),
    PRIMARY KEY (nim, kode_mk),
    FOREIGN KEY (nim) REFERENCES mahasiswa(nim),
    FOREIGN KEY (kode_mk) REFERENCES mata_kuliah(kode_mk)
);
```

---

# Insert Data Hasil Normalisasi

## Mahasiswa

```sql
INSERT INTO mahasiswa VALUES
('221001', 'Andi', 'Informatika'),
('221002', 'Rina', 'Sistem Informasi');
```

---

## Dosen

```sql
INSERT INTO dosen (nama_dosen) VALUES
('Budi'),
('Sinta');
```

---

## Mata Kuliah

```sql
INSERT INTO mata_kuliah VALUES
('IF101', 'Basis Data', 1),
('IF102', 'Pemrograman', 2);
```

---

## KRS

```sql
INSERT INTO krs VALUES
('221001', 'IF101', 'A'),
('221001', 'IF102', 'B'),
('221002', 'IF101', 'A');
```

---

# Query Menampilkan Hasil Normalisasi

```sql
SELECT 
    m.nim,
    m.nama_mahasiswa,
    m.prodi,
    mk.kode_mk,
    mk.nama_mk,
    d.nama_dosen,
    k.nilai
FROM krs k
JOIN mahasiswa m ON k.nim = m.nim
JOIN mata_kuliah mk ON k.kode_mk = mk.kode_mk
JOIN dosen d ON mk.id_dosen = d.id_dosen;
```




# Apakah Perlu Denormalisasi?

Pada studi kasus ini denormalisasi belum diperlukan karena:

- Jumlah data masih kecil
- Query masih sederhana
- Join antar tabel masih ringan

Denormalisasi biasanya digunakan jika:
- Data sangat besar
- Query terlalu kompleks
- Sistem membutuhkan performa baca sangat cepat

Untuk sistem akademik sederhana, BCNF lebih optimal karena:
- Mengurangi redundansi
- Menghindari anomali
- Menjaga konsistensi data