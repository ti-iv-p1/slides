---
title: Perintah Terminal Shell
version: 1.0.0
theme: react
header: Perintah Terminal Shell
footer: https://github.com/ti-iv-p1/slides/terminal
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Perintah Terminal Shell

---

Shell command adalah perintah yang dieksekusi dalam shell, yaitu antarmuka command-line yang memungkinkan pengguna berinteraksi dengan sistem operasi. Shell digunakan untuk menjalankan perintah sistem, mengelola file, menjalankan program, dan mengotomatisasi tugas melalui skrip.

---

### 1. **`ls`** (List Directory Content)

Digunakan untuk melihat isi dari suatu direktori.  
 **Contoh:**

```sh
ls
```

Menampilkan daftar file dan folder dalam direktori saat ini.

```sh
ls -l
```

Menampilkan daftar file dengan informasi detail seperti izin, pemilik, ukuran, dan waktu modifikasi.

---

### 2. **`cd`** (Change Directory)

Digunakan untuk berpindah direktori.  
 **Contoh:**

```sh
cd /home/user/Documents
```

Pindah ke direktori `Documents` dalam `home/user`.

```sh
cd ..
```

Pindah ke direktori satu tingkat di atas.

---

### 3. **`pwd`** (Print Working Directory)

Menampilkan path direktori kerja saat ini.  
 **Contoh:**

```sh
pwd
```

Outputnya mungkin seperti:

```
/home/user/Documents
```

---

### 4. **`mkdir`** (Make Directory)

Digunakan untuk membuat direktori baru.  
 **Contoh:**

```sh
mkdir proyek
```

Membuat direktori bernama `proyek`.

```sh
mkdir -p proyek/web
```

Membuat direktori `proyek` dan `web` di dalamnya secara bersamaan.

---

### 5. **`rm`** (Remove File/Directory)

Digunakan untuk menghapus file atau direktori.  
 **Contoh:**

```sh
rm file.txt
```

Menghapus file `file.txt`.

```sh
rm -r folder
```

Menghapus folder dan seluruh isinya.

---

### 6. **`cp`** (Copy File/Directory)

Digunakan untuk menyalin file atau direktori.  
 **Contoh:**

```sh
cp file.txt backup.txt
```

Menyalin `file.txt` menjadi `backup.txt`.

```sh
cp -r folder1 folder2
```

Menyalin `folder1` beserta isinya ke `folder2`.

---

### 7. **`mv`** (Move/Rename File)

Digunakan untuk memindahkan atau mengganti nama file/direktori.  
 **Contoh:**

```sh
mv file.txt dokumen.txt
```

Mengganti nama `file.txt` menjadi `dokumen.txt`.

```sh
mv file.txt /home/user/Documents/
```

Memindahkan `file.txt` ke dalam folder `Documents`.

---

### 8. **`touch`** (Create File)

Digunakan untuk membuat file kosong baru.  
 **Contoh:**

```sh
touch file_baru.txt
```

Membuat file kosong bernama `file_baru.txt`.

---

### 9. **`cat`** (Concatenate & Display File)

Menampilkan isi file ke terminal.  
 **Contoh:**

```sh
cat file.txt
```

Menampilkan isi `file.txt`.
