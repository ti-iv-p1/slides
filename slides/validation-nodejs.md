---
title: Validation di Node.js dan Express.js
version: 1.0.0
header: Validation di Node.js dan Express.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Validation di Node.js dan Express.js

---

## Tujuan Pembelajaran

- Memahami pentingnya validasi input data
- Menguasai teknik validasi dengan vanilla JavaScript
- Menggunakan library Express Validator
- Menerapkan validasi untuk form, API, dan database
- Membuat custom validator
- Menangani validation errors dengan baik
- Menerapkan best practices validation

---

## Apa itu Validation?

**Validation** adalah proses memeriksa apakah data yang diterima sesuai dengan kriteria yang ditentukan

**Mengapa validation penting?**

- 🛡️ **Security** - mencegah injeksi dan serangan
- 🐛 **Data integrity** - memastikan data konsisten
- 💾 **Database safety** - mencegah data corrupt
- 👤 **User experience** - memberikan feedback yang jelas
- 🎯 **Business logic** - memastikan aturan bisnis terpenuhi

---

## Client-Side vs Server-Side Validation

**Client-Side Validation** (Browser/JavaScript):

- ✅ Feedback cepat untuk user
- ✅ Mengurangi beban server
- ❌ Mudah dibypass
- ❌ Tidak aman untuk security

**Server-Side Validation** (Node.js):

- ✅ Aman dan terpercaya
- ✅ Tidak bisa dibypass
- ✅ Validasi final sebelum database
- ❌ Response lebih lambat

**BEST PRACTICE**: Gunakan keduanya! Client untuk UX, Server untuk security

---

## Jenis-jenis Validasi

1. **Type validation** - tipe data (string, number, boolean)
2. **Format validation** - format khusus (email, URL, phone)
3. **Length validation** - panjang karakter (min/max)
4. **Range validation** - rentang nilai numerik
5. **Pattern validation** - regex pattern
6. **Custom validation** - logika bisnis khusus
7. **Existence validation** - required/optional
8. **Uniqueness validation** - cek di database

---

## Validasi dengan Vanilla JavaScript

### Contoh 1: Validasi Required (Wajib Diisi)

```javascript
app.post("/register", (req, res) => {
  const { username } = req.body;
  const errors = [];

  if (!username || username.trim() === "") {
    errors.push({ field: "username", message: "Username is required" });
  }

  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Email

```javascript
app.post("/subscribe", (req, res) => {
  const { email } = req.body;
  const emailRegex = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;

  if (!email || !emailRegex.test(email)) {
    return res.status(400).json({
      error: "Valid email is required",
    });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Length (Min/Max)

```javascript
app.post("/create-username", (req, res) => {
  const { username } = req.body;

  if (!username || username.length < 3 || username.length > 20) {
    return res.status(400).json({
      error: "Username must be 3-20 characters",
    });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Number (Integer & Range)

```javascript
app.post("/update-age", (req, res) => {
  const age = Number(req.body.age);

  if (!Number.isInteger(age) || age < 18 || age > 100) {
    return res.status(400).json({
      error: "Age must be an integer between 18-100",
    });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi URL

```javascript
app.post("/add-website", (req, res) => {
  const { website } = req.body;
  const urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6}).*$/;

  if (!website || !urlRegex.test(website)) {
    return res.status(400).json({ error: "Valid URL is required" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Phone Number

```javascript
app.post("/add-phone", (req, res) => {
  const { phone } = req.body;
  const phoneRegex = /^(\+62|62|0)[0-9]{9,12}$/;
  const cleanPhone = phone?.replace(/[\s-]/g, "");

  if (!cleanPhone || !phoneRegex.test(cleanPhone)) {
    return res.status(400).json({ error: "Invalid phone number" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Password (Strong)

```javascript
app.post("/change-password", (req, res) => {
  const { password } = req.body;
  const strongRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&]).{8,}$/;

  if (!password || !strongRegex.test(password)) {
    return res.status(400).json({
      error:
        "Password must be 8+ chars with uppercase, lowercase, number, special char",
    });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Date

```javascript
app.post("/book-appointment", (req, res) => {
  const date = new Date(req.body.appointmentDate);
  const today = new Date();
  today.setHours(0, 0, 0, 0);

  if (isNaN(date.getTime()) || date < today) {
    return res.status(400).json({ error: "Invalid or past date" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Boolean

```javascript
app.post("/accept-terms", (req, res) => {
  const { termsAccepted } = req.body;

  if (termsAccepted !== true && termsAccepted !== "true") {
    return res.status(400).json({ error: "Must accept terms" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Array

```javascript
app.post("/add-tags", (req, res) => {
  const { tags } = req.body;
  const isValid =
    Array.isArray(tags) &&
    tags.length >= 1 &&
    tags.length <= 5 &&
    tags.every((tag) => typeof tag === "string");

  if (!isValid) {
    return res.status(400).json({ error: "Tags must be 1-5 strings" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Alphanumeric

```javascript
app.post("/create-code", (req, res) => {
  const { code } = req.body;
  const alphanumericRegex = /^[a-zA-Z0-9]+$/;

  if (!code || !alphanumericRegex.test(code)) {
    return res.status(400).json({ error: "Code must be alphanumeric" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Credit Card

```javascript
app.post("/add-card", (req, res) => {
  const cleanNumber = req.body.cardNumber?.replace(/[\s-]/g, "");
  const isValid = /^\d{13,19}$/.test(cleanNumber);

  if (!isValid) {
    return res.status(400).json({ error: "Invalid card number" });
  }

  res.json({ success: true });
});
```

---

## Vanilla JavaScript: Validasi Lengkap (Kombinasi)

```javascript
app.post("/register", (req, res) => {
  const { username, email, password } = req.body;

  // Validasi kombinasi dengan regex
  const isValidUsername = username && /^[a-zA-Z0-9_]{3,20}$/.test(username);
  const isValidEmail =
    email && /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(email);
  const isValidPassword =
    password && /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/.test(password);

  if (!isValidUsername || !isValidEmail || !isValidPassword) {
    return res.status(400).json({ error: "Invalid input" });
  }

  res.json({ success: true });
});
```

**Masalah dengan Vanilla JS**: Verbose, repetitive, error-prone

---

## Express-Validator

**Library validasi paling populer untuk Express.js**

**Mengapa menggunakan Express-Validator?**

- ✅ Built-in validators untuk banyak kasus (email, URL, phone, dll)
- ✅ Chainable API - mudah dibaca dan ditulis
- ✅ Custom validators untuk logika bisnis
- ✅ Sanitization built-in
- ✅ Integration sempurna dengan Express
- ✅ Async validation support
- ✅ Production-ready dan battle-tested

---

## Express-Validator: Instalasi

```bash
npm install express-validator
```

**Import yang diperlukan:**

```javascript
const { body, validationResult } = require("express-validator");
```

---

## Express-Validator: Validasi Required

```javascript
const { body, validationResult } = require("express-validator");

app.post(
  "/register",
  [
    body("username").notEmpty().withMessage("Username is required"),
    body("email").notEmpty().withMessage("Email is required"),
    body("password").notEmpty().withMessage("Password is required"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

**Metode lain:** `.notEmpty()`, `.exists()`, `.not().isEmpty()`

---

## Express-Validator: Validasi Email

```javascript
app.post(
  "/subscribe",
  [
    body("email")
      .notEmpty()
      .withMessage("Email is required")
      .isEmail()
      .withMessage("Invalid email format")
      .normalizeEmail(),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

**Fitur tambahan:**

- `.normalizeEmail()` - normalisasi format email (lowercase, remove dots di Gmail)
- `.isEmail({ allow_display_name: false })` - konfigurasi tambahan

---

## Express-Validator: Validasi Length (Min/Max)

```javascript
app.post(
  "/create-post",
  [
    body("title").isLength({ min: 5 }).withMessage("Title min 5 chars"),
    body("description")
      .isLength({ max: 500 })
      .withMessage("Description max 500 chars"),
    body("username")
      .isLength({ min: 3, max: 20 })
      .withMessage("Username 3-20 chars"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Number (Integer & Range)

```javascript
app.post(
  "/update-profile",
  [
    body("age").isInt({ min: 18, max: 100 }).withMessage("Age must be 18-100"),
    body("price").isFloat({ min: 0 }).withMessage("Price must be positive"),
    body("rating")
      .isFloat({ min: 1.0, max: 5.0 })
      .withMessage("Rating 1.0-5.0"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi URL

```javascript
app.post(
  "/add-website",
  [
    body("website")
      .notEmpty()
      .withMessage("URL is required")
      .isURL()
      .withMessage("Invalid URL format"),
    body("profileUrl")
      .isURL({ protocols: ["http", "https"], require_protocol: true })
      .withMessage("URL must have http/https"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Phone Number

```javascript
app.post(
  "/add-phone",
  [
    body("phone")
      .isMobilePhone("id-ID")
      .withMessage("Invalid Indonesian phone"),
    body("internationalPhone")
      .optional()
      .isMobilePhone("any")
      .withMessage("Invalid phone format"),
    body("customPhone")
      .matches(/^(\+62|62|0)[0-9]{9,12}$/)
      .withMessage("Invalid format"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

**Locale yang didukung:**

- `id-ID` - Indonesia
- `en-US` - USA
- `en-GB` - UK
- `any` - Semua format

---

## Express-Validator: Validasi Password (Strong)

```javascript
app.post(
  "/change-password",
  [
    body("password")
      .isStrongPassword({
        minLength: 8,
        minLowercase: 1,
        minUppercase: 1,
        minNumbers: 1,
        minSymbols: 1,
      })
      .withMessage(
        "Password must be strong (8+ chars, upper, lower, number, symbol)",
      ),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Date

```javascript
app.post(
  "/book-appointment",
  [
    body("appointmentDate").isDate().withMessage("Invalid date format"),
    body("futureDate")
      .isDate()
      .custom((value) => {
        if (new Date(value) <= new Date())
          throw new Error("Must be future date");
        return true;
      }),
    body("birthdate")
      .isDate()
      .isBefore(new Date().toISOString())
      .withMessage("Must be past date"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Boolean

```javascript
app.post(
  "/accept-terms",
  [
    body("termsAccepted").isBoolean().withMessage("Must be boolean"),
    body("agreedToTerms").equals("true").withMessage("Must accept terms"),
    body("newsletter").optional().isBoolean().toBoolean(),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Array

```javascript
app.post(
  "/add-tags",
  [
    body("tags")
      .isArray({ min: 1, max: 5 })
      .withMessage("Tags must be 1-5 items"),
    body("tags.*")
      .isString()
      .trim()
      .notEmpty()
      .withMessage("Tags must be non-empty strings"),
    body("recipients").isArray().withMessage("Recipients must be array"),
    body("recipients.*")
      .isEmail()
      .withMessage("Each recipient must be valid email"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Alphanumeric

```javascript
app.post(
  "/create-code",
  [
    body("code")
      .notEmpty()
      .isAlphanumeric()
      .withMessage("Code must be alphanumeric"),
    body("name").isAlpha().withMessage("Name must contain only letters"),
    body("pin")
      .isNumeric()
      .isLength({ min: 4, max: 6 })
      .withMessage("PIN must be 4-6 digits"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Credit Card

```javascript
app.post(
  "/add-card",
  [
    // Validasi credit card number
    body("cardNumber")
      .notEmpty()
      .withMessage("Card number is required")
      .isCreditCard()
      .withMessage("Invalid credit card number"),

    // Validasi CVV
    body("cvv")
      .isNumeric()
      .withMessage("CVV must be numeric")
      .isLength({ min: 3, max: 4 })
      .withMessage("CVV must be 3-4 digits"),

    // Validasi expiry date (MM/YY format)
    body("expiryDate")
      .matches(/^(0[1-9]|1[0-2])\/([0-9]{2})$/)
      .withMessage("Expiry date must be in MM/YY format"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ success: false, errors: errors.array() });
    }

    res.json({ success: true, message: "Card added" });
  },
);
```

---

## Express-Validator: Validasi JSON

```javascript
app.post(
  "/save-config",
  [
    body("config").isJSON().withMessage("Config must be valid JSON"),
    body("settings").custom((value) => {
      const parsed = JSON.parse(value);
      if (!parsed.hasOwnProperty("theme")) throw new Error("Must have theme");
      return true;
    }),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi IP Address

```javascript
app.post(
  "/whitelist-ip",
  [
    body("ipv4").isIP(4).withMessage("Must be valid IPv4"),
    body("ipv6").optional().isIP(6).withMessage("Must be valid IPv6"),
    body("ipAddress").isIP().withMessage("Must be valid IP address"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi dengan Optional

```javascript
app.post(
  "/update-profile",
  [
    body("name").notEmpty().withMessage("Name is required"),
    body("bio")
      .optional()
      .isLength({ max: 200 })
      .withMessage("Bio max 200 chars"),
    body("newsletter").optional({ checkFalsy: true }).isBoolean().toBoolean(),
    body("middleName").optional({ nullable: true }).isString(),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validasi Conditional

```javascript
app.post(
  "/checkout",
  [
    // Validasi berdasarkan kondisi
    body("shippingAddress")
      .if(body("deliveryType").equals("delivery"))
      .notEmpty()
      .withMessage("Shipping address required for delivery"),

    // Validasi saling bergantung
    body("password")
      .if((value, { req }) => req.body.changePassword === true)
      .notEmpty()
      .withMessage("Password required when changing password"),

    // Nested object validation
    body("address.street")
      .if(body("hasAddress").equals("true"))
      .notEmpty()
      .withMessage("Street is required"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ success: false, errors: errors.array() });
    }

    res.json({ success: true, message: "Checkout successful" });
  },
);
```

---

## Express-Validator: Validasi Lengkap (Kombinasi)

```javascript
app.post(
  "/register",
  [
    body("username").trim().isLength({ min: 3, max: 20 }).isAlphanumeric(),
    body("email").trim().isEmail().normalizeEmail(),
    body("password")
      .isLength({ min: 8 })
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
    body("age").isInt({ min: 18, max: 100 }),
    body("phone").isMobilePhone("id-ID"),
    body("terms").equals("true"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty())
      return res.status(400).json({ errors: errors.array() });
    res.json({ success: true });
  },
);
```

✅ **Keuntungan Express-Validator**: Clean, chainable, reusable

---

## Perbandingan: Vanilla vs Express-Validator

| Aspek                | Vanilla JavaScript         | Express-Validator               |
| -------------------- | -------------------------- | ------------------------------- |
| **Setup**            | Tidak perlu install        | `npm install express-validator` |
| **Kode**             | Verbose, banyak if-else    | Chainable, concise              |
| **Reusability**      | Harus manual refactor      | Built-in middleware pattern     |
| **Validators**       | Harus buat sendiri         | 40+ built-in validators         |
| **Error Messages**   | Manual array push          | `.withMessage()` chainable      |
| **Sanitization**     | Manual regex/replace       | Built-in sanitizers             |
| **Async Validation** | Manual promise handling    | Built-in async support          |
| **Maintainability**  | Susah maintain             | Mudah maintain                  |
| **Best untuk**       | Pembelajaran, proyek kecil | Production, proyek besar        |

---

## Kapan Menggunakan Vanilla JavaScript?

✅ **Gunakan Vanilla JavaScript ketika:**

- Belajar konsep dasar validation
- Proyek sangat sederhana (1-2 form)
- Tidak mau tambah dependency
- Custom validation yang sangat spesifik
- Butuh kontrol penuh atas proses

❌ **Hindari Vanilla JavaScript ketika:**

- Proyek production/enterprise
- Banyak form dengan validation kompleks
- Butuh consistency di seluruh aplikasi
- Tim besar yang perlu standar

---

## Kapan Menggunakan Express-Validator?

✅ **Gunakan Express-Validator ketika:**

- Proyek production/enterprise
- Banyak endpoint dengan validation
- Butuh validation library yang proven
- Tim yang perlu standar validation
- Perlu async validation (cek database)
- Mau kode yang clean dan maintainable

❌ **Hindari Express-Validator ketika:**

- Bukan menggunakan Express.js
- Proyek sangat minimal (overkill)
- Mau zero dependencies

---

## Contoh Perbandingan: Email Validation

**Vanilla JavaScript:**

```javascript
app.post("/subscribe", (req, res) => {
  const { email } = req.body;
  const errors = [];

  const emailRegex = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;

  if (!email || email.trim() === "") {
    errors.push({ field: "email", message: "Email is required" });
  } else if (!emailRegex.test(email)) {
    errors.push({ field: "email", message: "Email format is invalid" });
  }

  if (errors.length > 0) {
    return res.status(400).json({ success: false, errors });
  }

  res.json({ success: true });
});
```

**Express-Validator:**

```javascript
const { body, validationResult } = require("express-validator");

app.post(
  "/subscribe",
  [
    body("email")
      .notEmpty()
      .withMessage("Email is required")
      .isEmail()
      .withMessage("Email format is invalid"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ success: false, errors: errors.array() });
    }

    res.json({ success: true });
  },
);
```

---

## Express-Validator: Daftar Validators Populer

| Validator           | Kegunaan                 | Contoh                                       |
| ------------------- | ------------------------ | -------------------------------------------- |
| `.notEmpty()`       | Field tidak boleh kosong | `body('name').notEmpty()`                    |
| `.isEmail()`        | Validasi format email    | `body('email').isEmail()`                    |
| `.isURL()`          | Validasi format URL      | `body('website').isURL()`                    |
| `.isInt()`          | Validasi integer         | `body('age').isInt({ min: 18 })`             |
| `.isFloat()`        | Validasi decimal         | `body('price').isFloat({ min: 0 })`          |
| `.isLength()`       | Validasi panjang string  | `body('name').isLength({ min: 3, max: 20 })` |
| `.isAlpha()`        | Hanya huruf              | `body('name').isAlpha()`                     |
| `.isAlphanumeric()` | Huruf dan angka          | `body('username').isAlphanumeric()`          |
| `.isNumeric()`      | Hanya angka              | `body('pin').isNumeric()`                    |
| `.isMobilePhone()`  | Validasi nomor HP        | `body('phone').isMobilePhone('id-ID')`       |
| `.isDate()`         | Validasi tanggal         | `body('birthdate').isDate()`                 |
| `.isBoolean()`      | Validasi boolean         | `body('terms').isBoolean()`                  |
| `.isArray()`        | Validasi array           | `body('tags').isArray()`                     |
| `.isCreditCard()`   | Validasi kartu kredit    | `body('card').isCreditCard()`                |
| `.isIP()`           | Validasi IP address      | `body('ip').isIP()`                          |

---

## Express-Validator: Daftar Sanitizers Populer

| Sanitizer            | Kegunaan                       | Contoh                                |
| -------------------- | ------------------------------ | ------------------------------------- |
| `.trim()`            | Hapus whitespace di awal/akhir | `body('name').trim()`                 |
| `.escape()`          | Escape HTML characters         | `body('comment').escape()`            |
| `.normalizeEmail()`  | Normalisasi format email       | `body('email').normalizeEmail()`      |
| `.toLowerCase()`     | Konversi ke huruf kecil        | `body('email').toLowerCase()`         |
| `.toUpperCase()`     | Konversi ke huruf besar        | `body('code').toUpperCase()`          |
| `.toBoolean()`       | Konversi ke boolean            | `body('active').toBoolean()`          |
| `.toInt()`           | Konversi ke integer            | `body('age').toInt()`                 |
| `.toFloat()`         | Konversi ke float              | `body('price').toFloat()`             |
| `.toDate()`          | Konversi ke Date object        | `body('date').toDate()`               |
| `.whitelist()`       | Hanya karakter yang diizinkan  | `body('code').whitelist('A-Za-z0-9')` |
| `.blacklist()`       | Hapus karakter tertentu        | `body('name').blacklist('!@#$')`      |
| `.customSanitizer()` | Custom sanitization            | `body('phone').customSanitizer(...)`  |

---

## Express-Validator: Sanitization (Pembersihan Input)

**Sanitization** membersihkan input dari karakter berbahaya

```javascript
const { body } = require("express-validator");

app.post(
  "/submit",
  [
    // Trim whitespace
    body("name").trim().notEmpty(),

    // Lowercase
    body("email").trim().normalizeEmail().toLowerCase(),

    // Remove HTML tags (escape)
    body("comment").trim().escape(),

    // Blacklist characters
    body("username").trim().blacklist(" @#$%"),

    // Whitelist characters only
    body("code").trim().whitelist("A-Za-z0-9"),

    // Convert to boolean
    body("newsletter").toBoolean(),

    // Convert to integer
    body("age").toInt(),

    // Custom sanitization
    body("phone").customSanitizer((value) => {
      return value.replace(/[^\d]/g, ""); // Keep digits only
    }),
  ],
  (req, res) => {
    // Data sudah clean dan sanitized
    res.json({ success: true });
  },
);
```

---

## Express-Validator: Validation Middleware

**Reusable validation middleware:**

```javascript
// validators/userValidator.js
const { body } = require("express-validator");

const registerValidation = [
  body("username").trim().isLength({ min: 3, max: 20 }).isAlphanumeric(),
  body("email").trim().isEmail().normalizeEmail(),
  body("password")
    .isLength({ min: 8 })
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
];

module.exports = { registerValidation };
```

---

## Express-Validator: Using Middleware

```javascript
const { registerValidation } = require("../validators/userValidator");
const { validationResult } = require("express-validator");

const checkValidation = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  next();
};

router.post("/register", registerValidation, checkValidation, (req, res) => {
  res.json({ message: "Registration successful" });
});
```

---

## Express-Validator: Custom Validators

```javascript
const { body } = require("express-validator");
const User = require("../models/User");

const registerValidation = [
  body("username")
    .trim()
    .isLength({ min: 3, max: 20 })
    .custom(async (value) => {
      // Check if username already exists
      const existingUser = await User.findOne({ username: value });
      if (existingUser) {
        throw new Error("Username already taken");
      }
      return true;
    }),

  body("email")
    .trim()
    .isEmail()
    .normalizeEmail()
    .custom(async (value) => {
      // Check if email already exists
      const existingUser = await User.findOne({ email: value });
      if (existingUser) {
        throw new Error("Email already registered");
      }
      return true;
    }),

  body("passwordConfirmation").custom((value, { req }) => {
    // Check if password and confirmation match
    if (value !== req.body.password) {
      throw new Error("Password confirmation does not match");
    }
    return true;
  }),
];
```

---

## Express-Validator: Custom Validators

```javascript
const { body } = require("express-validator");
const User = require("../models/User");

const registerValidation = [
  body("username")
    .trim()
    .isLength({ min: 3, max: 20 })
    .custom(async (value) => {
      // Check if username already exists
      const existingUser = await User.findOne({ username: value });
      if (existingUser) {
        throw new Error("Username already taken");
      }
      return true;
    }),

  body("email")
    .trim()
    .isEmail()
    .normalizeEmail()
    .custom(async (value) => {
      // Check if email already exists
      const existingUser = await User.findOne({ email: value });
      if (existingUser) {
        throw new Error("Email already registered");
      }
      return true;
    }),

  body("passwordConfirmation").custom((value, { req }) => {
    // Check if password and confirmation match
    if (value !== req.body.password) {
      throw new Error("Password confirmation does not match");
    }
    return true;
  }),
];
```

---

## File Upload Validation

```javascript
const multer = require("multer");
const path = require("path");

const storage = multer.diskStorage({
  destination: "uploads/",
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + "-" + Math.round(Math.random() * 1e9);
    cb(
      null,
      file.fieldname + "-" + uniqueSuffix + path.extname(file.originalname),
    );
  },
});

const upload = multer({
  storage: storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB max
  },
  fileFilter: (req, file, cb) => {
    // Validate file type
    const allowedTypes = /jpeg|jpg|png|gif/;
    const extname = allowedTypes.test(
      path.extname(file.originalname).toLowerCase(),
    );
    const mimetype = allowedTypes.test(file.mimetype);

    if (extname && mimetype) {
      return cb(null, true);
    } else {
      cb(new Error("Only images are allowed (jpeg, jpg, png, gif)"));
    }
  },
});

app.post("/upload", upload.single("avatar"), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: "No file uploaded" });
  }
  res.json({ message: "File uploaded", filename: req.file.filename });
});
```

---

## Validasi dengan Template Engine

**Menampilkan error di form HTML:**

```javascript
app.post(
  "/register",
  [body("username").notEmpty(), body("email").isEmail()],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.render("register", {
        errors: errors.array(),
        formData: req.body,
      });
    }
    res.redirect("/success");
  },
);
```

---

## Validasi dengan Template Engine (cont.)

**Template (Handlebars):**

```handlebars
<form method="POST" action="/register">
  <div>
    <label>Username:</label>
    <input type="text" name="username" value="{{formData.username}}" />
    {{#each errors}}
      {{#if (eq this.path "username")}}
        <span class="error">{{this.msg}}</span>
      {{/if}}
    {{/each}}
  </div>
  <button type="submit">Register</button>
</form>
```

---

## File Upload Validation

```javascript
const multer = require("multer");

const upload = multer({
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB max
  fileFilter: (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif/;
    const isValid = allowedTypes.test(file.mimetype);
    cb(isValid ? null : new Error("Only images allowed"), isValid);
  },
});

app.post("/upload", upload.single("avatar"), (req, res) => {
  if (!req.file) return res.status(400).json({ error: "No file" });
  res.json({ filename: req.file.filename });
});
```

---

## Validation Best Practices

✅ **DO:**

- Validasi di server-side (wajib)
- Gunakan validation library (express-validator untuk production)
- Berikan pesan error yang jelas dan spesifik
- Sanitize input untuk mencegah XSS
- Validasi sebelum database operation
- Return appropriate HTTP status codes (400 untuk validation error)
- Batasi panjang input (DoS prevention)
- Whitelist dibanding blacklist

---

## Validation Best Practices (cont.)

❌ **DON'T:**

- Hanya validasi di client-side
- Trust user input tanpa validasi
- Hardcode validation logic berulang-ulang
- Gunakan pesan error terlalu teknis untuk user
- Expose sensitive info di error messages
- Validasi dengan eval() atau Function()
- Lupa sanitize input yang akan ditampilkan
- Over-validate hingga UX jelek

---

## Error Response Format

**Consistent error format untuk API:**

```javascript
// ✅ Good error response
{
  "success": false,
  "errors": [
    {
      "field": "email",
      "message": "Email is already registered",
      "code": "EMAIL_DUPLICATE"
    },
    {
      "field": "password",
      "message": "Password must be at least 8 characters",
      "code": "PASSWORD_TOO_SHORT"
    }
  ],
  "timestamp": "2026-04-17T10:30:00Z"
}

// ❌ Bad error response
{
  "error": "Bad Request"
}
```

---

## Contoh Lengkap: Validation Flow

```javascript
// validators/productValidator.js
const { body } = require("express-validator");

const createProductValidation = [
  body("name").trim().notEmpty().isLength({ max: 100 }),
  body("price").isFloat({ min: 0 }),
  body("stock").isInt({ min: 0 }),
  body("category").isIn(["electronics", "clothing", "food"]),
  body("description").optional().trim().isLength({ max: 1000 }),
];

module.exports = { createProductValidation };
```

---

## Contoh Lengkap: Route Implementation

```javascript
const { validationResult } = require("express-validator");
const { createProductValidation } = require("../validators/productValidator");

router.post("/products", createProductValidation, async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  const product = await Product.create(req.body);
  res.status(201).json({ success: true, data: product });
});
```

---

## Summary

- **Validation** penting untuk security, data integrity, dan UX
- **Server-side validation** adalah wajib, client-side adalah bonus
- **Vanilla JavaScript** - manual tapi memberikan kontrol penuh
- **Express-validator** - clean, chainable, dan production-ready
- **Sanitization** membersihkan input dari karakter berbahaya
- **Custom validators** untuk logika bisnis khusus
- **Organized validation** dengan middleware yang reusable
- **Clear error messages** membantu user dan debugging
- Selalu validasi sebelum database operation

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih!

## Questions?

**Resources:**

- Express-validator: https://express-validator.github.io
- Express-validator GitHub: https://github.com/express-validator/express-validator
- OWASP Input Validation: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- MDN Regular Expressions: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions
