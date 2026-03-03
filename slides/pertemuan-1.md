---
title: Pengenalan Pemrograman Web, Konsep Client-Server, dan JavaScript
version: 1.0.0
header: Pengenalan Pemrograman Web, Konsep Client-Server, dan JavaScript
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Pengenalan Pemrograman Web, Konsep Client-Server, dan JavaScript

---

## Tujuan Pertemuan 1

- Memahami perbedaan _client-side_ dan _server-side_
- Mengetahui peran JavaScript dalam ekosistem web modern
- Melihat hubungan antara HTML, CSS, dan JavaScript

---

## Perkembangan Pemrograman Web

- Website statis
- Aplikasi dinamis
- Aplikasi backend dan frontend

---

## Jenis Web Development

- _Frontend development_: UI/UX interaktif dengan framework modern
- _Backend development_: API, logic, database
- _Full-stack development_: skill menguasai keduanya

---

## Peran JavaScript di Dunia Modern

- Bahasa wajib untuk web browser
- Satu bahasa bisa dipakai _client_ dan _server_
- Didukung oleh ekosistem luas (library, framework, tools)

---

## Arsitektur Aplikasi Web

- Web berbasis arsitektur client-server
- Browser: mengakses konten
- Server: menyediakan konten

---

## Konsep Client-Server Model

- Client mengirim request
- Server memproses & mengirim response
- Proses berulang setiap interaksi

---

## Peran Client-Side

- Menampilkan antarmuka ke pengguna
- Mengelola input user
- Menjalankan script (JavaScript) untuk interaktivitas

---

## Peran Server-Side

- Verifikasi & proses data
- Interaksi dengan database
- Menyediakan API atau halaman dinamis

---

## Sejarah Singkat JavaScript

- Dibuat oleh Brendan Eich (1995)
- Awalnya untuk menambah interaktivitas di halaman statis
- Kini menjadi bahasa paling populer dunia (StackOverflow survey)

---

## Kenapa JavaScript?

- Dijalankan di semua browser tanpa tambahan software
- Dukungan komunitas global
- Fleksibel: dapat digunakan untuk frontend, backend, hingga mobile (React Native)

---

## Contoh Kombinasi HTML+CSS+JS

- HTML → konten
- CSS → desain menarik
- JS → tombol "klik saya" ubah warna teks

---

## Peran JavaScript di Frontend

- Manipulasi DOM (ubah teks, gambar, style)
- Menangani interaksi pengguna (klik, input, animasi)
- Komunikasi data lewat API

---

## Peran JavaScript di Backend

- Node.js memungkinkan server-side JS
- Framework populer: Express.js, NestJS
- Digunakan untuk REST API, WebSocket, microservices

---

## Frontend Framework

- React, Vue, Angular
- Fokus pengelolaan state UI dan rendering cepat
- Contoh: SPA (Single Page Application)

---

## Backend Framework

- Express.js, Django, Spring Boot
- Mengatur request, response, API
- Skala besar dan fleksibel

---

## Integrasi API dan Data

- Backend menyediakan API JSON
- Frontend konsumsi API via JavaScript Fetch
- Membuat web real-time dan dinamis

---

## Demo JavaScript di Console Browser

- Buka Chrome → Inspect → Console
- Ketik: `console.log("Halo Web!");`

---

## Menyisipkan Script JavaScript di HTML

```html
<script>
  alert("Halo dari JavaScript!");
</script>
```

---

## Interaksi DOM

```html
<button onclick="document.getElementById('demo').innerHTML='Klik Berhasil!'">
  Klik Saya!
</button>
<p id="demo"></p>
```

---

## Ringkasan Pertemuan

- Pemrograman web = kombinasi HTML, CSS, JS
- Konsep _client-server_ dasar web modern
- JavaScript memegang peran di frontend & backend
