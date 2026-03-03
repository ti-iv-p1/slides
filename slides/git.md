---
title: Git
version: 1.0.0
header: Git
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Git

---

Git adalah sistem kontrol versi terdistribusi (distributed version control system - DVCS) yang digunakan untuk melacak perubahan dalam kode sumber selama pengembangan perangkat lunak.

---

📌 Fungsi utama Git:

- Mencatat perubahan kode dari waktu ke waktu.
- Memungkinkan banyak pengembang bekerja dalam satu proyek tanpa konflik.
- Membantu mengembalikan versi sebelumnya jika terjadi kesalahan.

---

📌 Cara kerja Git:

- **Local Repository** → Git bekerja secara lokal di komputer pengembang tanpa memerlukan koneksi internet.
- **Staging Area** → Perubahan yang dilakukan perlu ditambahkan ke staging sebelum di-commit.
- **Commit** → Menyimpan perubahan sebagai versi baru.
- **Remote Repository** → Perubahan bisa diunggah ke server seperti GitHub, GitLab, atau Bitbucket.

---

## **Konfigurasi Git (`git config`)**

Mengatur nama dan email pengguna untuk commit.

```sh
git config --global user.name "Nama Anda"
git config --global user.email "email@example.com"
```

📌 **Cek konfigurasi yang sudah diset:**

```sh
git config --list
```

---

## **Inisialisasi Repositori (`git init`)**

Membuat repository Git baru di dalam folder proyek.

```sh
git init
```

📌 **Setelah ini, folder akan memiliki folder `.git` untuk menyimpan metadata Git.**

---

## **Melihat Status Repository (`git status`)**

Menampilkan status perubahan file dalam repository.

```sh
git status
```

📌 **Output bisa menunjukkan file yang belum ditambahkan ke staging atau yang sudah dimodifikasi.**

---

## **Menambahkan File ke Staging (`git add`)**

Menandai file agar siap untuk di-commit.

📌 **Menambahkan satu file:**

```sh
git add nama_file.txt
```

📌 **Menambahkan semua file yang berubah:**

```sh
git add .
```

---

## **Menyimpan Perubahan (`git commit`)**

Menyimpan perubahan yang ada di staging area.

```sh
git commit -m "Pesan commit yang menjelaskan perubahan"
```

📌 **Contoh:**

```sh
git commit -m "Menambahkan fitur login"
```

---

## **Melihat Log Commit (`git log`)**

Menampilkan daftar commit yang telah dibuat.

```sh
git log
```

📌 **Untuk tampilan lebih ringkas:**

```sh
git log --oneline
```

---

## **Melihat Perbedaan (`git diff`)**

Menampilkan perbedaan antara perubahan yang dibuat dengan versi terakhir.

📌 **Melihat perubahan sebelum di-commit:**

```sh
git diff
```

📌 **Melihat perubahan setelah `git add`:**

```sh
git diff --staged
```

---

## **Membatalkan Perubahan (`git reset`)**

📌 **Membatalkan `git add`:**

```sh
git reset nama_file.txt
```

---

## **Menghapus Perubahan Terakhir (`git revert`)**

Menghapus commit terakhir dengan membuat commit baru.

```sh
git revert HEAD
```

---

## Apa itu GitHub?

GitHub adalah platform hosting untuk repository Git yang digunakan untuk menyimpan, mengelola, dan berkolaborasi dalam proyek perangkat lunak secara online.

---

📌 Fungsi utama GitHub:

- Menyimpan repository Git di cloud (mirip seperti Google Drive untuk kode).
- Memfasilitasi kerja tim melalui fitur seperti Pull Request, Issues, dan Collaborator.
- Menyediakan CI/CD (Continuous Integration/Continuous Deployment) untuk otomatisasi.
- Memungkinkan open-source development, di mana orang lain bisa berkontribusi ke proyek kita.

---

📌 Perbedaan Git dan GitHub:

| Git                              | GitHub                                                |
| -------------------------------- | ----------------------------------------------------- |
| Software kontrol versi           | Platform hosting untuk Git                            |
| Bekerja secara lokal di komputer | Bekerja secara online di cloud                        |
| Tidak memerlukan internet        | Memerlukan internet untuk mengakses repositori online |
| Bisa digunakan tanpa GitHub      | GitHub tidak bisa bekerja tanpa Git                   |

---

## **Melihat Remote Repository (`git remote -v`)**

Menampilkan daftar repository remote yang terhubung.

```sh
git remote -v
```

---

## **Mengatur Remote Repository (`git remote add`)**

Menambahkan remote repository baru.

```sh
git remote add origin https://github.com/user/repository.git
```

---

## **Mengunggah Perubahan ke Repository (`git push`)**

Mengirim commit ke repository online (GitHub, GitLab, dll.).

```sh
git push origin branch_name
```

📌 **Contoh:**

```sh
git push origin main
```

---

## **Mengambil Perubahan dari Repository (`git pull`)**

Menarik update terbaru dari repository online.

```sh
git pull origin branch_name
```

📌 **Contoh:**

```sh
git pull origin main
```

---

## **Menyambungkan Branch Lokal ke Remote (`git push --set-upstream`)**

Jika ingin mengirim branch baru ke remote pertama kali:

```sh
git push --set-upstream origin nama_branch
```

---

## **Menyalin Repositori (`git clone`)**

Menduplikasi repository dari server ke komputer lokal.

```sh
git clone https://github.com/user/repository.git
```

📌 **Contoh:**

```sh
git clone https://github.com/torvalds/linux.git
```

---

## Github Pages

Deploy web static kamu dengan Github Pages

```sh
git checkout --orphan gh-pages
git add .
git commit -m "Deploy to GitHub Pages"
git push -u origin gh-pages
```
