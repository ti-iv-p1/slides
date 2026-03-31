# Rencana Pembelajaran Semester (RPS)

- Mata Kuliah: Pemrograman Web II

- Semester: IV

## Rincian Pertemuan

### Pertemuan 1

- Slides :
  - https://ti-iv-p1.github.io/slides/git
  - https://ti-iv-p1.github.io/slides/emmet
  - https://ti-iv-p1.github.io/slides/terminal
  - https://ti-iv-p1.github.io/slides/pertemuan-1

## Pertemuan 2

Review Node.js dengan Handlebars dan SQLite

### Tujuan Pertemuan

- Memahami cara kerja web dan alur request-response
- Membedakan jenis-jenis web: statis, dinamis, SPA, MPA
- Memahami perbedaan CSR dan SSR
- Menguasai Node.js untuk backend development
- Menguasai Handlebars sebagai template engine SSR
- Menggunakan SQLite dengan better-sqlite3
- Membuat aplikasi web full-stack sederhana

https://ti-iv-p1.github.io/slides/pertemuan-2

---

## Pertemuan 3

### 📊 Database - Normalisasi dan Relasi

**Link:** https://ti-iv-p1.github.io/slides/database-normalisasi-relasi

**Key Takeaways:**

- Normalisasi = proses mengorganisir data untuk mengurangi redundansi
- Bentuk Normal: 1NF (atomik) → 2NF (no partial dependency) → 3NF (no transitive dependency) → BCNF
- Jenis Relasi: One-to-One (1:1), One-to-Many (1:N), Many-to-Many (M:N dengan junction table)
- Selalu gunakan foreign key constraint dan index pada kolom yang sering diquery

### 📝 DBML - Database Markup Language

**Link:** https://ti-iv-p1.github.io/slides/dbml

**Key Takeaways:**

- DBML = bahasa markup untuk mendefinisikan struktur database dengan syntax simple
- Keuntungan: readable, database agnostic, auto-generate ERD, version control ready
- Syntax: `Table name { columns }`, `Ref: table1.col > table2.col`, `Enum`, `TableGroup`
- Tools: dbdiagram.io (online), DBML CLI (`dbml2sql`, `sql2dbml`), VS Code extensions
- Use cases: prototyping, dokumentasi, collaboration, learning
