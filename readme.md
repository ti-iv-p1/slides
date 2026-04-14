# Rencana Pembelajaran Semester (RPS)

- Mata Kuliah: Pemrograman Web II

- Semester: IV

## Rincian Pertemuan

### Pertemuan 1

**Topik:** Pengenalan Pemrograman Web, Konsep Client-Server, dan JavaScript

**Link Slides:**

- [Git](https://ti-iv-p1.github.io/slides/git)
- [Emmet (Visual Studio Code)](https://ti-iv-p1.github.io/slides/emmet)
- [Perintah Terminal Shell](https://ti-iv-p1.github.io/slides/terminal)
- [Pertemuan 1](https://ti-iv-p1.github.io/slides/pertemuan-1)

**Tujuan Pembelajaran:**

- Memahami perbedaan _client-side_ dan _server-side_
- Mengetahui peran JavaScript dalam ekosistem web modern
- Melihat hubungan antara HTML, CSS, dan JavaScript
- Menguasai Git untuk version control
- Menggunakan Emmet untuk produktivitas coding
- Menguasai perintah-perintah dasar terminal/shell

---

## Pertemuan 2

**Topik:** Node.js dengan Handlebars dan SQLite

**Link:** https://ti-iv-p1.github.io/slides/pertemuan-2

**Tujuan Pembelajaran:**

- Memahami cara kerja web dan alur request-response
- Membedakan jenis-jenis web: statis, dinamis, SPA, MPA
- Memahami perbedaan CSR dan SSR
- Menguasai Node.js untuk backend development
- Menguasai Handlebars sebagai template engine SSR
- Menggunakan SQLite dengan better-sqlite3
- Membuat aplikasi web full-stack sederhana

---

## Pertemuan 3

### Database - Normalisasi dan Relasi

**Link:** https://ti-iv-p1.github.io/slides/database-normalisasi-relasi

**Tujuan Pembelajaran:**

- Memahami konsep normalisasi database
- Menguasai bentuk-bentuk normal (1NF, 2NF, 3NF, BCNF)
- Memahami jenis-jenis relasi: One-to-One, One-to-Many, Many-to-Many
- Mampu mendesain database dengan normalisasi yang tepat
- Menerapkan foreign key constraint dan indexing

**Key Takeaways:**

- Normalisasi = proses mengorganisir data untuk mengurangi redundansi
- Bentuk Normal: 1NF (atomik) → 2NF (no partial dependency) → 3NF (no transitive dependency) → BCNF
- Jenis Relasi: One-to-One (1:1), One-to-Many (1:N), Many-to-Many (M:N dengan junction table)
- Selalu gunakan foreign key constraint dan index pada kolom yang sering diquery

---

## Pertemuan 4

---

### DBML - Database Markup Language

**Link:** https://ti-iv-p1.github.io/slides/dbml

**Tujuan Pembelajaran:**

- Memahami konsep DBML (Database Markup Language)
- Menguasai syntax DBML untuk mendefinisikan database schema
- Mampu membuat ERD menggunakan DBML
- Menggunakan tools DBML (dbdiagram.io, CLI)
- Menerapkan DBML untuk dokumentasi dan prototyping database

**Key Takeaways:**

- DBML = bahasa markup untuk mendefinisikan struktur database dengan syntax simple
- Keuntungan: readable, database agnostic, auto-generate ERD, version control ready
- Syntax: `Table name { columns }`, `Ref: table1.col > table2.col`, `Enum`, `TableGroup`
- Tools: dbdiagram.io (online), DBML CLI (`dbml2sql`, `sql2dbml`), VS Code extensions
- Use cases: prototyping, dokumentasi, collaboration, learning

---

### Error Handling di Node.js

**Link:** https://ti-iv-p1.github.io/slides/error-handling-nodejs

**Tujuan Pembelajaran:**

- Memahami konsep error dan exception di Node.js
- Mengenal berbagai jenis error
- Menguasai teknik error handling yang efektif
- Menerapkan best practices dalam penanganan error

### MVC Pattern di Node.js

**Link:** https://ti-iv-p1.github.io/slides/mvc-nodejs

**Tujuan Pembelajaran:**

- Memahami konsep MVC (Model-View-Controller)
- Mengerti peran masing-masing komponen MVC
- Mampu mengimplementasikan MVC di Node.js
- Menerapkan best practices struktur aplikasi

### Routing di Node.js dan Express.js

**Link:** https://ti-iv-p1.github.io/slides/router-nodejs

**Tujuan Pembelajaran:**

- Memahami konsep routing dalam web application
- Menguasai cara membuat routes di Express.js
- Memahami HTTP methods (GET, POST, PUT, DELETE)
- Mampu mengorganisir routes dengan Express Router
- Menerapkan route parameters dan query strings

---

### Autentikasi di Node.js

**Link:** https://ti-iv-p1.github.io/slides/autentikasi-nodejs

**Tujuan Pembelajaran:**

- Memahami konsep autentikasi dan otorisasi
- Menguasai session-based authentication
- Mengimplementasikan JWT (JSON Web Token)
- Melakukan password hashing dengan bcrypt
- Menerapkan best practices keamanan
- Memahami OAuth dan social login

### Authorization di Node.js

**Link:** https://ti-iv-p1.github.io/slides/authorization-nodejs

**Tujuan Pembelajaran:**

- Memahami konsep authorization dan perbedaannya dengan authentication
- Menguasai Role-Based Access Control (RBAC)
- Mengimplementasikan Permission-Based Authorization
- Memahami Access Control List (ACL)
- Menerapkan Resource-Based Authorization
- Menggunakan middleware untuk authorization

---

### Testing di Node.js

**Link:** https://ti-iv-p1.github.io/slides/testing-nodejs

**Tujuan Pembelajaran:**

- Memahami pentingnya testing dalam software development
- Mengenal berbagai jenis testing (unit, integration, e2e)
- Menguasai Jest untuk testing JavaScript/Node.js
- Mampu membuat unit test dan integration test
- Memahami mocking dan stubbing
- Menerapkan Test-Driven Development (TDD)
- Mengukur test coverage

---

### ORM di Node.js dengan Sequelize

**Link:** https://ti-iv-p1.github.io/slides/orm-sequelize-nodejs

**Tujuan Pembelajaran:**

- Memahami konsep ORM (Object-Relational Mapping)
- Menguasai Sequelize untuk database management
- Mampu membuat model dan migrations
- Menguasai CRUD operations dengan Sequelize
- Memahami associations (relationships)
- Menerapkan validations dan hooks
- Menggunakan transactions dan query optimization
