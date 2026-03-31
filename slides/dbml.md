---
marp: true
theme: default
class: lead
paginate: true
backgroundColor: #fff
---

# Tutorial DBML
## Database Markup Language
### Desain Database dengan Mudah

---

# Daftar Isi

1. Pengantar DBML
2. Mengapa Menggunakan DBML?
3. Syntax Dasar DBML
4. Mendefinisikan Tabel dan Kolom
5. Tipe Data dan Constraints
6. Mendefinisikan Relasi
7. Indexes dan Keys
8. Enum dan Table Groups
9. Tools dan Platform
10. Contoh Project Lengkap
11. Export ke SQL
12. Best Practices

---

<!-- _class: lead -->

# 1. Pengantar DBML

---

## Apa itu DBML?

**DBML (Database Markup Language)** adalah bahasa markup open-source yang dirancang khusus untuk mendefinisikan dan mendokumentasikan struktur database.

Dikembangkan oleh tim **Holistics** dengan tujuan membuat desain database lebih mudah dibaca, ditulis, dan dikomunikasikan antar tim.

---

## Apa yang Bisa Dilakukan dengan DBML?

- ✅ Mendefinisikan struktur database dengan syntax yang simple dan intuitif
- ✅ Membuat dokumentasi database yang mudah dibaca manusia
- ✅ Menggenerate diagram Entity Relationship (ERD) secara otomatis
- ✅ Export ke berbagai format SQL (PostgreSQL, MySQL, SQL Server, dll)
- ✅ Version control database schema dengan Git

---

## Keuntungan DBML

```
✓ Simple & Readable      - Syntax yang mudah dipahami
✓ Database Agnostic      - Tidak terikat vendor database tertentu
✓ Visual Diagram         - Auto-generate ERD dari kode
✓ Version Control Ready  - Perfect untuk Git workflow
✓ Team Collaboration     - Mudah direview dan didiskusikan
✓ Documentation          - Self-documenting schema
```

---

<!-- _class: lead -->

# 2. Mengapa Menggunakan DBML?

---

## Perbandingan: SQL vs DBML

### SQL Traditional
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## Perbandingan: SQL vs DBML

### DBML
```dbml
Table users {
  id int [pk, increment]
  username varchar(50) [not null, unique]
  email varchar(100) [not null, unique]
  created_at timestamp [default: `now()`]
}

Table posts {
  id int [pk, increment]
  user_id int [not null, ref: > users.id]
  title varchar(200) [not null]
  content text
  created_at timestamp [default: `now()`]
}
```

---

## Keuntungan DBML

- ✅ Lebih ringkas dan mudah dibaca
- ✅ Relasi langsung terlihat di definisi kolom
- ✅ Tidak perlu menulis verbose SQL syntax
- ✅ Otomatis generate diagram visual

---

## Use Cases

DBML sangat cocok untuk:

1. **Prototyping** - Cepat membuat mockup database structure
2. **Documentation** - Mendokumentasikan existing database
3. **Team Collaboration** - Code review database schema
4. **Learning** - Belajar database design tanpa kompleksitas SQL
5. **Migration Planning** - Merencanakan perubahan schema

---

<!-- _class: lead -->

# 3. Syntax Dasar DBML

---

## Struktur File DBML

```dbml
// Komentar single line

/*
  Komentar
  multi-line
*/

Project project_name {
  database_type: 'PostgreSQL'
  Note: 'Deskripsi project'
}

Table table_name {
  column_name data_type [settings]
  
  Note: 'Catatan untuk tabel'
}

Ref: table1.column > table2.column
```

---

## Elemen-Elemen Dasar

| Elemen | Fungsi |
|--------|--------|
| **Table** | Mendefinisikan tabel database |
| **Ref** | Mendefinisikan relasi antar tabel |
| **Enum** | Mendefinisikan tipe data enum |
| **TableGroup** | Mengelompokkan tabel terkait |
| **Project** | Metadata project database |
| **Note** | Menambahkan dokumentasi |

---

<!-- _class: lead -->

# 4. Mendefinisikan Tabel dan Kolom

---

## Syntax Tabel Dasar

```dbml
Table users {
  id int
  username varchar
  email varchar
  age int
}
```

Syntax paling sederhana untuk mendefinisikan tabel dengan kolom-kolomnya.

---

## Tabel dengan Schema/Namespace

```dbml
Table public.users {
  id int
  username varchar
}

Table admin.users {
  id int
  username varchar
}
```

Gunakan schema/namespace untuk mengorganisir tabel dalam group yang berbeda.

---

## Aliases untuk Table

```dbml
Table users as U {
  id int [pk]
  username varchar
}

Table posts as P {
  id int [pk]
  user_id int [ref: > U.id]
}
```

Aliases memudahkan referensi tabel saat mendefinisikan relasi.

---

## Multi-line Column Definition

```dbml
Table products {
  id int [
    pk,
    increment,
    note: 'Primary key'
  ]
  
  name varchar(255) [
    not null,
    unique,
    note: 'Product name must be unique'
  ]
  
  price decimal(10,2) [
    not null,
    default: 0,
    note: 'Price in USD'
  ]
}
```

---

<!-- _class: lead -->

# 5. Tipe Data dan Constraints

---

## Tipe Data Umum (1)

```dbml
Table data_types_example {
  // Integer types
  small_int smallint
  regular_int int
  big_int bigint
  
  // Decimal types
  decimal_col decimal(10,2)
  float_col float
  double_col double
  
  // String types
  char_col char(10)
  varchar_col varchar(255)
  text_col text
```

---

## Tipe Data Umum (2)

```dbml
  // Date & Time
  date_col date
  time_col time
  datetime_col datetime
  timestamp_col timestamp
  
  // Boolean
  bool_col boolean
  
  // Binary
  blob_col blob
  
  // JSON
  json_col json
}
```

---

## Column Settings (Constraints)

```dbml
Table users {
  id int [pk]                          // Primary Key
  id int [primary key]                 // Alternative syntax
  
  username varchar [unique]            // Unique constraint
  username varchar [not null]          // Not null constraint
  
  status varchar [default: 'active']   // Default value
  created_at timestamp [default: `now()`] // Function default
  
  id int [increment]                   // Auto increment
  id int [pk, increment]               // Multiple settings
  
  email varchar [note: 'User email']   // Column note
}
```

---

## Composite Primary Key

```dbml
Table user_roles {
  user_id int
  role_id int
  
  indexes {
    (user_id, role_id) [pk]
  }
}
```

Untuk primary key yang terdiri dari multiple kolom, gunakan syntax `indexes`.

---

<!-- _class: lead -->

# 6. Mendefinisikan Relasi

---

## Relationship Syntax

### 1. Inline Relationship Reference

```dbml
Table posts {
  id int [pk]
  user_id int [ref: > users.id]  // Many-to-One
  title varchar
}

Table users {
  id int [pk]
  username varchar
}
```

---

## Relationship Syntax

### 2. Separate Ref Block

```dbml
Table posts {
  id int [pk]
  user_id int
  title varchar
}

Table users {
  id int [pk]
  username varchar
}

Ref: posts.user_id > users.id
```

---

## One-to-Many (1:N)

```dbml
// One user has many posts
Table users {
  id int [pk]
  username varchar
}

Table posts {
  id int [pk]
  user_id int [ref: > users.id]  // Many-to-one from posts to users
  title varchar
}

// Alternative syntax:
Ref: posts.user_id > users.id
```

**Symbol:** `>` atau `<`
- `posts.user_id > users.id` : Many posts → One user
- `users.id < posts.user_id` : One user ← Many posts

---

## One-to-One (1:1)

```dbml
Table users {
  id int [pk]
  username varchar
}

Table user_profiles {
  id int [pk]
  user_id int [unique, ref: - users.id]  // One-to-one
  bio text
  avatar varchar
}

// Alternative syntax:
Ref: user_profiles.user_id - users.id
```

**Symbol:** `-`

---

## Many-to-Many (M:N)

```dbml
Table students {
  id int [pk]
  name varchar
}

Table courses {
  id int [pk]
  name varchar
}

// Junction table
Table student_courses {
  student_id int [ref: > students.id]
  course_id int [ref: > courses.id]
  enrolled_at timestamp
  
  indexes {
    (student_id, course_id) [pk]
  }
}
```

---

## Relationship Settings

```dbml
Ref name_optional: table1.column > table2.column [
  delete: cascade,
  update: cascade
]

// Example:
Ref: posts.user_id > users.id [
  delete: cascade,
  update: restrict
]
```

**Delete/Update Actions:**
- `cascade` - Cascade action
- `restrict` - Restrict action
- `set null` - Set NULL
- `set default` - Set default value
- `no action` - No action

---

<!-- _class: lead -->

# 7. Indexes dan Keys

---

## Single Column Index

```dbml
Table users {
  id int [pk]
  username varchar
  email varchar
  created_at timestamp
  
  indexes {
    username
    email [unique]
    created_at [name: 'idx_created']
  }
}
```

---

## Composite Index

```dbml
Table posts {
  id int [pk]
  user_id int
  category_id int
  created_at timestamp
  
  indexes {
    (user_id, created_at) [name: 'idx_user_date']
    (category_id, created_at)
  }
}
```

Composite index digunakan untuk query yang melibatkan multiple kolom.

---

## Index Settings

```dbml
Table products {
  id int [pk]
  name varchar
  description text
  price decimal
  
  indexes {
    name [unique, name: 'unique_product_name']
    price [name: 'idx_price', type: btree]
    description [type: hash]
    (name, price) [unique]
  }
}
```

**Index Types:**
- `btree` (default)
- `hash`
- `gin`
- `gist`

---

<!-- _class: lead -->

# 8. Enum dan Table Groups

---

## Enum Types

```dbml
enum user_status {
  active
  inactive
  suspended
  deleted
}

enum order_status {
  pending [note: 'Order placed']
  processing [note: 'Being processed']
  shipped [note: 'Shipped to customer']
  delivered [note: 'Delivered']
  cancelled [note: 'Cancelled by user or admin']
}
```

---

## Menggunakan Enum

```dbml
Table users {
  id int [pk]
  username varchar
  status user_status [default: 'active']
}

Table orders {
  id int [pk]
  status order_status [default: 'pending']
  total decimal
}
```

Enum membuat status fields lebih type-safe dan terstruktur.

---

## Table Groups

```dbml
TableGroup user_management {
  users
  user_profiles
  user_roles
}

TableGroup e_commerce {
  products
  orders
  order_items
  payments
}

Table users {
  id int [pk]
  username varchar
}

Table user_profiles {
  id int [pk]
  user_id int [ref: - users.id]
}
```

---

<!-- _class: lead -->

# 9. Tools dan Platform

---

## 1. dbdiagram.io

**dbdiagram.io** adalah platform online gratis untuk membuat database diagram menggunakan DBML.

**Fitur:**
- ✅ Real-time diagram preview
- ✅ Export ke PNG, PDF, SQL
- ✅ Share diagram dengan link
- ✅ Import dari SQL
- ✅ Collaboration features

**Website:** https://dbdiagram.io

---

## 2. DBML CLI

Install DBML command-line tool:

```bash
# Install via npm
npm install -g @dbml/cli

# Convert DBML to SQL
dbml2sql schema.dbml --postgres > schema.sql
dbml2sql schema.dbml --mysql > schema.sql

# Convert SQL to DBML
sql2dbml schema.sql --postgres > schema.dbml

# Validate DBML file
dbml2sql schema.dbml --validate
```

---

## 3. VS Code Extensions

**DBML Language Support:**
- ✅ Syntax highlighting
- ✅ Auto-completion
- ✅ Error checking
- ✅ Format document

Search **"DBML"** di VS Code Extensions Marketplace.

---

## 4. Integration dengan Tools Lain

DBML dapat diintegrasikan dengan:

- **Git** - Version control untuk database schema
- **CI/CD** - Automated schema validation
- **Documentation Tools** - Generate docs dari DBML
- **ORM Tools** - Generate model dari DBML

---

<!-- _class: lead -->

# 10. Contoh Project Lengkap

---

## E-Commerce Database Schema (1)

```dbml
Project ecommerce_db {
  database_type: 'PostgreSQL'
  Note: '''
    # E-Commerce Database
    Complete database schema for e-commerce platform
  '''
}

// Enums
enum user_role {
  customer
  admin
  vendor
}

enum order_status {
  pending
  processing
  shipped
  delivered
  cancelled
  refunded
}
```

---

## E-Commerce Database Schema (2)

```dbml
enum payment_status {
  pending
  completed
  failed
  refunded
}

// Users & Authentication
Table users {
  id int [pk, increment]
  email varchar(255) [unique, not null]
  password_hash varchar(255) [not null]
  first_name varchar(100)
  last_name varchar(100)
  role user_role [default: 'customer']
  is_active boolean [default: true]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
```

---

## E-Commerce Database Schema (3)

```dbml
  indexes {
    email
    (first_name, last_name)
  }
  
  Note: 'User accounts and authentication'
}

Table user_addresses {
  id int [pk, increment]
  user_id int [not null, ref: > users.id]
  address_line1 varchar(255) [not null]
  address_line2 varchar(255)
  city varchar(100) [not null]
  state varchar(100)
  postal_code varchar(20)
  country varchar(100) [not null]
  is_default boolean [default: false]
  created_at timestamp [default: `now()`]
```

---

## E-Commerce Database Schema (4)

```dbml
  indexes {
    user_id
  }
}

// Products
Table categories {
  id int [pk, increment]
  name varchar(100) [unique, not null]
  slug varchar(100) [unique, not null]
  description text
  parent_id int [ref: > categories.id]
  created_at timestamp [default: `now()`]
  
  indexes {
    slug
    parent_id
  }
}
```

---

## E-Commerce Database Schema (5)

```dbml
Table products {
  id int [pk, increment]
  category_id int [ref: > categories.id]
  name varchar(255) [not null]
  slug varchar(255) [unique, not null]
  description text
  price decimal(10,2) [not null]
  compare_at_price decimal(10,2)
  cost_price decimal(10,2)
  sku varchar(100) [unique]
  barcode varchar(100)
  quantity int [default: 0]
  weight decimal(8,2)
  is_active boolean [default: true]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
```

---

## E-Commerce Database Schema (6)

```dbml
  indexes {
    slug
    category_id
    sku
    is_active
    (name, is_active)
  }
  
  Note: 'Product catalog'
}

Table product_images {
  id int [pk, increment]
  product_id int [not null, ref: > products.id]
  image_url varchar(500) [not null]
  alt_text varchar(255)
  position int [default: 0]
  created_at timestamp [default: `now()`]
```

---

## E-Commerce Database Schema (7)

```dbml
  indexes {
    product_id
  }
}

// Orders
Table orders {
  id int [pk, increment]
  user_id int [not null, ref: > users.id]
  order_number varchar(50) [unique, not null]
  status order_status [default: 'pending']
  subtotal decimal(10,2) [not null]
  tax_amount decimal(10,2) [default: 0]
  shipping_amount decimal(10,2) [default: 0]
  discount_amount decimal(10,2) [default: 0]
  total_amount decimal(10,2) [not null]
```

---

## E-Commerce Database Schema (8)

```dbml
  // Shipping address
  shipping_address_line1 varchar(255)
  shipping_address_line2 varchar(255)
  shipping_city varchar(100)
  shipping_state varchar(100)
  shipping_postal_code varchar(20)
  shipping_country varchar(100)
  
  notes text
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
  
  indexes {
    user_id
    order_number
    status
    created_at
  }
  
  Note: 'Customer orders'
}
```

---

## E-Commerce Database Schema (9)

```dbml
Table order_items {
  id int [pk, increment]
  order_id int [not null, ref: > orders.id]
  product_id int [not null, ref: > products.id]
  product_name varchar(255) [not null]
  product_sku varchar(100)
  quantity int [not null]
  unit_price decimal(10,2) [not null]
  total_price decimal(10,2) [not null]
  created_at timestamp [default: `now()`]
  
  indexes {
    order_id
    product_id
  }
  
  Note: 'Line items in orders'
}
```

---

## E-Commerce Database Schema (10)

```dbml
// Payments
Table payments {
  id int [pk, increment]
  order_id int [not null, ref: > orders.id]
  payment_method varchar(50) [not null]
  amount decimal(10,2) [not null]
  status payment_status [default: 'pending']
  transaction_id varchar(255)
  payment_gateway varchar(100)
  paid_at timestamp
  created_at timestamp [default: `now()`]
  
  indexes {
    order_id
    status
    transaction_id
  }
}
```

---

## E-Commerce Database Schema (11)

```dbml
// Reviews
Table product_reviews {
  id int [pk, increment]
  product_id int [not null, ref: > products.id]
  user_id int [not null, ref: > users.id]
  rating int [not null, note: 'Rating from 1 to 5']
  title varchar(255)
  comment text
  is_verified_purchase boolean [default: false]
  is_approved boolean [default: false]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
  
  indexes {
    product_id
    user_id
    rating
    (product_id, user_id) [unique]
  }
}
```

---

## E-Commerce Database Schema (12)

```dbml
// Shopping Cart
Table cart_items {
  id int [pk, increment]
  user_id int [ref: > users.id]
  product_id int [not null, ref: > products.id]
  quantity int [not null, default: 1]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
  
  indexes {
    user_id
    product_id
    (user_id, product_id) [unique]
  }
}
```

---

## E-Commerce Database Schema (13)

```dbml
// Relationships with settings
Ref: order_items.order_id > orders.id [delete: cascade]
Ref: order_items.product_id > products.id [delete: restrict]
Ref: payments.order_id > orders.id [delete: cascade]

// Table Groups
TableGroup user_management {
  users
  user_addresses
}

TableGroup product_catalog {
  categories
  products
  product_images
  product_reviews
}

TableGroup order_management {
  orders
  order_items
  payments
}
```

---

<!-- _class: lead -->

# 11. Export ke SQL

---

## Export menggunakan dbdiagram.io

1. Buka https://dbdiagram.io
2. Paste DBML code
3. Klik **Export** → Pilih database (PostgreSQL, MySQL, SQL Server, dll)
4. Download file SQL

---

## Export menggunakan CLI

```bash
# PostgreSQL
dbml2sql schema.dbml --postgres -o schema.sql

# MySQL
dbml2sql schema.dbml --mysql -o schema.sql

# SQL Server
dbml2sql schema.dbml --mssql -o schema.sql
```

---

## Contoh Output SQL (PostgreSQL) (1)

```sql
-- Auto-generated from DBML

CREATE TYPE "user_role" AS ENUM (
  'customer',
  'admin',
  'vendor'
);

CREATE TYPE "order_status" AS ENUM (
  'pending',
  'processing',
  'shipped',
  'delivered',
  'cancelled',
  'refunded'
);
```

---

## Contoh Output SQL (PostgreSQL) (2)

```sql
CREATE TABLE "users" (
  "id" SERIAL PRIMARY KEY,
  "email" varchar(255) UNIQUE NOT NULL,
  "password_hash" varchar(255) NOT NULL,
  "first_name" varchar(100),
  "last_name" varchar(100),
  "role" user_role DEFAULT 'customer',
  "is_active" boolean DEFAULT true,
  "created_at" timestamp DEFAULT (now()),
  "updated_at" timestamp DEFAULT (now())
);

CREATE TABLE "products" (
  "id" SERIAL PRIMARY KEY,
  "category_id" int,
  "name" varchar(255) NOT NULL,
  "slug" varchar(255) UNIQUE NOT NULL,
  "description" text,
  "price" decimal(10,2) NOT NULL
);
```

---

## Contoh Output SQL (PostgreSQL) (3)

```sql
-- Indexes
CREATE INDEX ON "users" ("email");
CREATE INDEX ON "products" ("slug");

-- Foreign Keys
ALTER TABLE "products" 
  ADD FOREIGN KEY ("category_id") 
  REFERENCES "categories" ("id");
```

---

<!-- _class: lead -->

# 12. Best Practices

---

## 1. Naming Conventions (1)

```dbml
// ✅ Good: Consistent naming
Table users {
  id int [pk]
  first_name varchar
  last_name varchar
  created_at timestamp
}

Table user_profiles {
  id int [pk]
  user_id int [ref: > users.id]
}
```

---

## 1. Naming Conventions (2)

```dbml
// ❌ Bad: Inconsistent naming
Table Users {
  ID int [pk]
  FirstName varchar
  LastName varchar
  createdAt timestamp
}
```

**Rekomendasi:**
- Gunakan `snake_case` untuk tabel dan kolom
- Nama tabel: plural atau singular (konsisten)
- Foreign key: `{table_singular}_id`
- Junction table: `{table1}_{table2}`

---

## 2. Dokumentasi dengan Notes

```dbml
Table orders {
  id int [pk, note: 'Unique order identifier']
  user_id int [ref: > users.id, note: 'Customer who placed the order']
  status order_status [note: 'Current order status']
  total_amount decimal(10,2) [note: 'Total including tax and shipping']
  
  Note: '''
    Orders table stores all customer orders.
    Status workflow: pending -> processing -> shipped -> delivered
  '''
}
```

---

## 3. Indexes untuk Performance

```dbml
Table posts {
  id int [pk]
  user_id int [ref: > users.id]
  title varchar
  content text
  published_at timestamp
  
  indexes {
    user_id                      // Foreign key index
    published_at                 // Filter/sort index
    (user_id, published_at)      // Composite for user's posts query
  }
}
```

**Index Guidelines:**
- Primary keys (automatic)
- Foreign keys
- Columns sering di-filter (WHERE)
- Columns sering di-sort (ORDER BY)
- Composite index untuk query kombinasi

---

## 4. Proper Relationship Definitions

```dbml
// ✅ Good: Clear relationship with cascade
Table posts {
  id int [pk]
  user_id int [not null, ref: > users.id]
}

Ref: posts.user_id > users.id [delete: cascade, update: cascade]

// ✅ Good: Preventing orphaned records
Table order_items {
  id int [pk]
  order_id int [not null, ref: > orders.id]
  product_id int [not null, ref: > products.id]
}

Ref: order_items.order_id > orders.id [delete: cascade]
Ref: order_items.product_id > products.id [delete: restrict]
```

---

## 5. Use Enums untuk Status (1)

```dbml
// ✅ Good: Type-safe status
enum order_status {
  pending
  processing
  shipped
  delivered
  cancelled
}

Table orders {
  id int [pk]
  status order_status [default: 'pending']
}
```

---

## 5. Use Enums untuk Status (2)

```dbml
// ❌ Bad: String without constraints
Table orders {
  id int [pk]
  status varchar [default: 'pending']  // Typo-prone
}
```

---

## 6. Organize dengan Table Groups

```dbml
TableGroup authentication {
  users
  user_sessions
  password_resets
}

TableGroup content {
  posts
  comments
  tags
}

TableGroup settings {
  site_settings
  user_preferences
}
```

---

## 7. Version Control

```bash
# Initialize git in your project
git init

# Track DBML files
git add schema.dbml

# Commit with meaningful messages
git commit -m "Add user authentication tables"

# Create branches for schema changes
git checkout -b feature/add-notifications-table
```

---

## 8. Project Metadata

```dbml
Project my_app {
  database_type: 'PostgreSQL'
  Note: '''
    # My Application Database
    
    Version: 2.0
    Last Updated: 2026-03-31
    
    ## Schema Overview
    - User Management
    - Product Catalog
    - Order Processing
    - Payment Integration
    
    ## Conventions
    - All timestamps in UTC
    - Soft deletes using is_deleted flag
    - Created/updated timestamps on all tables
  '''
}
```

---

<!-- _class: lead -->

# Latihan

---

## Soal 1: Konversi SQL ke DBML

Konversi SQL berikut ke DBML:

```sql
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    author_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    isbn VARCHAR(13) UNIQUE,
    published_date DATE,
    FOREIGN KEY (author_id) REFERENCES authors(id)
);

CREATE INDEX idx_books_author ON books(author_id);
CREATE INDEX idx_books_published ON books(published_date);
```

---

## Soal 2: Desain Database dengan DBML

Buatlah DBML schema untuk **sistem management sekolah** dengan requirements:

- Siswa memiliki nama, NIS, tanggal lahir, alamat
- Guru memiliki nama, NIP, mata pelajaran
- Kelas memiliki nama kelas, tahun ajaran
- Siswa bisa masuk ke satu kelas
- Kelas diajar oleh banyak guru
- Nilai siswa untuk setiap mata pelajaran
- Absensi siswa per hari

**Include:**
- Tabel dengan relasi yang benar
- Indexes yang appropriate
- Enum untuk status/type
- Table notes untuk dokumentasi

---

## Soal 3: Optimasi Schema (1)

Review DBML schema berikut dan identifikasi masalah:

```dbml
Table user {
  id int
  name varchar
  email varchar
}

Table post {
  id int
  userid int
  title varchar
  body text
}
```

---

## Soal 3: Optimasi Schema (2)

**Pertanyaan:**

1. Apa yang kurang dari definisi tabel ini?
2. Index apa yang seharusnya ditambahkan?
3. Constraint apa yang missing?
4. Bagaimana cara memperbaikinya?

---

<!-- _class: lead -->

# Kesimpulan

---

## Keuntungan Utama DBML

1. **Simplicity** - Syntax yang simple dan intuitif
2. **Readability** - Mudah dibaca dan dipahami tim
3. **Visual** - Auto-generate ERD diagram
4. **Portable** - Export ke berbagai SQL dialects
5. **Version Control** - Perfect untuk Git workflow
6. **Collaboration** - Mudah untuk code review

---

## Kapan Menggunakan DBML

- ✅ Design database baru
- ✅ Dokumentasi existing database
- ✅ Prototyping dan brainstorming
- ✅ Team discussion tentang schema
- ✅ Learning database design

---

## Best Practices Summary

- ✅ Gunakan naming convention yang konsisten
- ✅ Tambahkan notes untuk dokumentasi
- ✅ Define indexes untuk performa
- ✅ Gunakan enums untuk status fields
- ✅ Organize dengan table groups
- ✅ Version control dengan Git
- ✅ Review dan validate schema secara berkala

---

## Tools Rekomendasi

- **dbdiagram.io** - Online editor dengan visual diagram
- **@dbml/cli** - Command-line tool untuk conversion
- **VS Code Extension** - Developer experience yang lebih baik

---

## Dengan Menguasai DBML, Anda Dapat:

- ✅ Mendesain database lebih cepat dan efisien
- ✅ Berkomunikasi lebih baik dengan tim
- ✅ Maintain database schema dengan mudah
- ✅ Generate dokumentasi otomatis
- ✅ Improve database development workflow

---

<!-- _class: lead -->

# Resources

---

## Official Resources

- **DBML Homepage:** https://www.dbml.org
- **Documentation:** https://www.dbml.org/docs/
- **GitHub Repository:** https://github.com/holistics/dbml
- **dbdiagram.io:** https://dbdiagram.io

---

## Related Tools

- **Prisma Schema** - Similar tool for ORM
- **PlantUML** - General purpose diagram tool
- **SchemaSpy** - Database documentation tool
- **Liquibase** - Database migration tool

---

<!-- _class: lead -->

# Terima Kasih!
## Selamat Belajar DBML

Tags: #dbml #database #schema-design #erd #database-design
