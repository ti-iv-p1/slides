---
title: Error Handling di Node.js
version: 1.0.0
header: Error Handling di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Error Handling di Node.js

---

## Tujuan Pembelajaran

- Memahami konsep error dan exception di Node.js
- Mengenal berbagai jenis error
- Menguasai teknik error handling yang efektif
- Menerapkan best practices dalam penanganan error

---

## Apa itu Error?

**Error** adalah kondisi tidak normal yang terjadi saat program berjalan

- **Error** bisa disebabkan oleh:
  - Kesalahan logika program
  - Input yang tidak valid
  - Masalah koneksi (network, database)
  - Resource tidak tersedia (file, memory)

---

## Jenis-jenis Error di Node.js

1. **Standard JavaScript Errors**
   - `Error` - error umum
   - `SyntaxError` - kesalahan syntax
   - `ReferenceError` - variabel tidak ditemukan
   - `TypeError` - tipe data salah
2. **System Errors**
   - `ENOENT` - file/directory tidak ditemukan
   - `ECONNREFUSED` - koneksi ditolak

---

## Try-Catch untuk Synchronous Code

```javascript
try {
  // Code yang mungkin error
  const data = JSON.parse("invalid json");
  console.log(data);
} catch (error) {
  // Tangani error
  console.error("Error parsing JSON:", error.message);
} finally {
  // Code yang selalu dijalankan
  console.log("Selesai");
}
```

---

## Error pada Asynchronous Code - Callback

```javascript
const fs = require("fs");

fs.readFile("file.txt", "utf8", (error, data) => {
  if (error) {
    console.error("Error membaca file:", error.message);
    return;
  }
  console.log("Data:", data);
});
```

**Konvensi Node.js**: Parameter pertama callback adalah `error`

---

## Error pada Promises

```javascript
const fs = require("fs").promises;

fs.readFile("file.txt", "utf8")
  .then((data) => {
    console.log("Data:", data);
  })
  .catch((error) => {
    console.error("Error:", error.message);
  });
```

---

## Error pada Async/Await

```javascript
const fs = require("fs").promises;

async function readFileData() {
  try {
    const data = await fs.readFile("file.txt", "utf8");
    console.log("Data:", data);
    return data;
  } catch (error) {
    console.error("Error membaca file:", error.message);
    throw error; // Re-throw jika perlu
  }
}

readFileData();
```

---

## Membuat Custom Error

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
    this.statusCode = 400;
  }
}

// Penggunaan
function validateUser(user) {
  if (!user.email) {
    throw new ValidationError("Email wajib diisi");
  }
}
```

---

## Error Handling di Express.js

```javascript
const express = require("express");
const app = express();

// Route dengan error
app.get("/user/:id", async (req, res, next) => {
  try {
    const user = await getUserById(req.params.id);
    if (!user) {
      throw new Error("User tidak ditemukan");
    }
    res.json(user);
  } catch (error) {
    next(error); // Teruskan ke error handler
  }
});
```

---

## Error Middleware di Express.js

```javascript
// Error handler middleware (harus di akhir)
app.use((err, req, res, next) => {
  console.error(err.stack);

  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    error: {
      message: err.message,
      status: statusCode,
    },
  });
});
```

---

## Uncaught Exception

```javascript
// Tangani error yang tidak tertangani
process.on("uncaughtException", (error) => {
  console.error("Uncaught Exception:", error);
  // Log error, cleanup, lalu exit
  process.exit(1);
});

// Tangani rejected promise yang tidak tertangani
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection:", reason);
  // Log error, cleanup, lalu exit
  process.exit(1);
});
```

---

## Best Practices Error Handling

1. **Selalu tangani error** - jangan biarkan error tidak tertangani
2. **Gunakan try-catch untuk async/await**
3. **Buat custom error class** untuk jenis error spesifik
4. **Logging yang baik** - catat detail error untuk debugging
5. **Jangan expose detail error** ke user (security)
6. **Cleanup resources** - tutup koneksi, file handle, dll

---

## Contoh Struktur Error yang Baik

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true; // Operational error
    Error.captureStackTrace(this, this.constructor);
  }
}

// Penggunaan
if (!user) {
  throw new AppError("User tidak ditemukan", 404);
}
```

---

## Operational vs Programming Errors

**Operational Errors** _(dapat diprediksi)_

- Invalid user input
- Network timeout
- Database connection error

**Programming Errors** _(bugs)_

- Syntax error
- Undefined variable
- Type error

---

## Error Logging

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: "error",
  format: winston.format.json(),
  transports: [new winston.transports.File({ filename: "error.log" })],
});

app.use((err, req, res, next) => {
  logger.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
  });
  res.status(500).json({ error: "Internal Server Error" });
});
```

---

## Contoh Lengkap - API dengan Error Handling

```javascript
const express = require("express");
const app = express();

app.use(express.json());

// Custom error class
class ApiError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

// Route
app.post("/api/users", async (req, res, next) => {
  try {
    const { name, email } = req.body;

    if (!name || !email) {
      throw new ApiError("Name dan email wajib diisi", 400);
    }

    // Proses create user...
    res.status(201).json({ message: "User created" });
  } catch (error) {
    next(error);
  }
});
```

---

## Contoh Lengkap - Error Middleware

```javascript
// 404 handler
app.use((req, res, next) => {
  next(new ApiError("Route tidak ditemukan", 404));
});

// Global error handler
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || "Internal Server Error";

  // Log error
  console.error({
    error: message,
    stack: err.stack,
    path: req.path,
  });

  // Response
  res.status(statusCode).json({
    success: false,
    error: message,
  });
});
```

---

## Tips Debugging Error

1. **Baca error message dengan teliti**
2. **Perhatikan stack trace** - lihat dari mana error berasal
3. **Gunakan console.log / debugger** strategis
4. **Check dokumentasi** API/library yang digunakan
5. **Gunakan tools**: Node.js debugger, VS Code debugger
6. **Test error scenarios** - jangan hanya test happy path

---

## Kesalahan Umum yang Harus Dihindari

❌ Mengabaikan error dalam callback
❌ Tidak menangani rejected promise
❌ Expose stack trace ke client (production)
❌ Catch error tanpa melakukan apa-apa
❌ Tidak log error untuk debugging

---

## Resources & Referensi

- [Node.js Error Documentation](https://nodejs.org/api/errors.html)
- [Express Error Handling Guide](https://expressjs.com/en/guide/error-handling.html)
- [JavaScript Error Reference (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Happy Coding! 🚀
