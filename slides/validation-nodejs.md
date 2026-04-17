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
- Menguasai teknik validasi di Node.js
- Menggunakan library validasi (express-validator, Joi, Zod)
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

## Validasi Tanpa Library

```javascript
app.post("/register", (req, res) => {
  const { username, email, password } = req.body;
  const errors = [];

  // Check required fields
  if (!username || username.trim() === "") {
    errors.push("Username is required");
  }

  // Check email format
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!email || !emailRegex.test(email)) {
    errors.push("Valid email is required");
  }

  // Check password length
  if (!password || password.length < 8) {
    errors.push("Password must be at least 8 characters");
  }

  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }

  // Process valid data...
});
```

❌ **Masalah**: Verbose, repetitive, error-prone

---

## Express-Validator

**Library validasi paling populer untuk Express.js**

```bash
npm install express-validator
```

**Features:**

- Built-in validators untuk kasus umum
- Chain multiple validators
- Custom validators
- Sanitization
- Integration dengan Express

---

## Express-Validator: Basic Usage

```javascript
const { body, validationResult } = require("express-validator");

app.post(
  "/register",
  [
    // Validate username
    body("username")
      .trim()
      .notEmpty()
      .withMessage("Username is required")
      .isLength({ min: 3, max: 20 })
      .withMessage("Username must be 3-20 characters"),

    // Validate email
    body("email")
      .trim()
      .notEmpty()
      .withMessage("Email is required")
      .isEmail()
      .withMessage("Must be a valid email")
      .normalizeEmail(),

    // Validate password
    body("password")
      .notEmpty()
      .withMessage("Password is required")
      .isLength({ min: 8 })
      .withMessage("Password must be at least 8 characters")
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
      .withMessage("Password must contain uppercase, lowercase, and number"),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    // Process valid data...
    res.json({ message: "Registration successful" });
  },
);
```

---

## Express-Validator: Built-in Validators

```javascript
// String validators
body("name").isString().isLength({ min: 2, max: 50 });
body("email").isEmail();
body("url").isURL();
body("phone").isMobilePhone("id-ID");

// Number validators
body("age").isInt({ min: 18, max: 100 });
body("price").isFloat({ min: 0 });

// Date validators
body("birthdate").isDate().isBefore(new Date().toISOString());

// Boolean validators
body("terms").isBoolean().equals("true");

// Array validators
body("tags").isArray({ min: 1, max: 5 });
body("tags.*").isString(); // Validate each element

// Custom regex
body("username").matches(/^[a-zA-Z0-9_]+$/);
```

---

## Express-Validator: Validation Middleware

**Reusable validation middleware:**

```javascript
// validators/userValidator.js
const { body } = require("express-validator");

const registerValidation = [
  body("username")
    .trim()
    .isLength({ min: 3, max: 20 })
    .withMessage("Username must be 3-20 characters")
    .isAlphanumeric()
    .withMessage("Username must contain only letters and numbers"),

  body("email")
    .trim()
    .isEmail()
    .withMessage("Must be a valid email")
    .normalizeEmail(),

  body("password")
    .isLength({ min: 8 })
    .withMessage("Password must be at least 8 characters")
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage("Password must contain uppercase, lowercase, and number"),
];

const loginValidation = [
  body("email").trim().isEmail().withMessage("Must be a valid email"),
  body("password").notEmpty().withMessage("Password is required"),
];

module.exports = { registerValidation, loginValidation };
```

---

## Express-Validator: Using Middleware

```javascript
// routes/userRoutes.js
const express = require("express");
const {
  registerValidation,
  loginValidation,
} = require("../validators/userValidator");
const { validationResult } = require("express-validator");
const router = express.Router();

// Middleware to check validation results
const checkValidation = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  next();
};

// Use validation middleware
router.post("/register", registerValidation, checkValidation, (req, res) => {
  // Data sudah valid di sini
  res.json({ message: "Registration successful" });
});

router.post("/login", loginValidation, checkValidation, (req, res) => {
  // Data sudah valid di sini
  res.json({ message: "Login successful" });
});

module.exports = router;
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

## Joi - Schema-Based Validation

**Joi adalah library validasi berbasis schema**

```bash
npm install joi
```

**Advantages:**

- Schema-based approach
- More readable untuk struktur kompleks
- Tidak terikat dengan Express
- Bisa digunakan untuk validasi umum

---

## Joi: Basic Usage

```javascript
const Joi = require("joi");

// Define schema
const userSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(20).required(),

  email: Joi.string().email().required(),

  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .required()
    .messages({
      "string.pattern.base":
        "Password must contain uppercase, lowercase, and number",
    }),

  age: Joi.number().integer().min(18).max(100),

  birthdate: Joi.date().max("now"),

  tags: Joi.array().items(Joi.string()).min(1).max(5),

  terms: Joi.boolean().valid(true).required(),
});

// Validate data
const { error, value } = userSchema.validate(req.body, {
  abortEarly: false, // Return all errors
});

if (error) {
  return res.status(400).json({
    errors: error.details.map((err) => err.message),
  });
}

// Use validated data
console.log(value);
```

---

## Joi: Validation Middleware

```javascript
// middleware/validateMiddleware.js
const validateRequest = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true, // Remove unknown fields
    });

    if (error) {
      return res.status(400).json({
        errors: error.details.map((err) => ({
          field: err.path.join("."),
          message: err.message,
        })),
      });
    }

    // Replace req.body with validated value
    req.body = value;
    next();
  };
};

module.exports = validateRequest;
```

```javascript
// routes/userRoutes.js
const validateRequest = require("../middleware/validateMiddleware");
const { userSchema } = require("../schemas/userSchema");

router.post("/register", validateRequest(userSchema), (req, res) => {
  // req.body sudah valid dan clean
  res.json({ message: "Registration successful" });
});
```

---

## Joi: Advanced Features

```javascript
const Joi = require("joi");

const advancedSchema = Joi.object({
  // Conditional validation
  hasAccount: Joi.boolean(),
  accountId: Joi.when("hasAccount", {
    is: true,
    then: Joi.string().required(),
    otherwise: Joi.forbidden(),
  }),

  // Alternative values
  deliveryMethod: Joi.string().valid("email", "sms", "whatsapp"),

  // Nested objects
  address: Joi.object({
    street: Joi.string().required(),
    city: Joi.string().required(),
    zipcode: Joi.string().pattern(/^\d{5}$/),
  }),

  // Custom validation
  customField: Joi.string().custom((value, helpers) => {
    if (value === "forbidden") {
      return helpers.error("any.invalid");
    }
    return value;
  }),

  // References to other fields
  password: Joi.string().min(8),
  passwordConfirmation: Joi.string().valid(Joi.ref("password")).required(),
});
```

---

## Zod - TypeScript-First Validation

**Zod adalah modern schema validation library dengan TypeScript support**

```bash
npm install zod
```

**Advantages:**

- TypeScript-first with automatic type inference
- Zero dependencies
- Composable schemas
- Parsing & transformation

---

## Zod: Basic Usage

```javascript
const { z } = require("zod");

// Define schema
const userSchema = z.object({
  username: z.string().min(3).max(20),

  email: z.string().email(),

  password: z
    .string()
    .min(8)
    .regex(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
      "Must contain uppercase, lowercase, and number",
    ),

  age: z.number().int().min(18).max(100),

  tags: z.array(z.string()).min(1).max(5),

  terms: z.boolean().refine((val) => val === true, {
    message: "Terms must be accepted",
  }),
});

// Validate data
try {
  const validatedData = userSchema.parse(req.body);
  // validatedData is fully typed in TypeScript
  res.json({ message: "Success", data: validatedData });
} catch (error) {
  if (error instanceof z.ZodError) {
    return res.status(400).json({
      errors: error.errors.map((err) => ({
        field: err.path.join("."),
        message: err.message,
      })),
    });
  }
}
```

---

## Sanitization - Pembersihan Input

**Sanitization** membersihkan input dari karakter berbahaya

```javascript
const { body } = require("express-validator");

app.post(
  "/submit",
  [
    // Trim whitespace
    body("name").trim(),

    // Lowercase
    body("email").trim().normalizeEmail().toLowerCase(),

    // Remove HTML tags
    body("comment").trim().escape(),

    // Blacklist characters
    body("username").trim().blacklist(" @#$%"),

    // Whitelist characters only
    body("code").trim().whitelist("A-Za-z0-9"),

    // Custom sanitization
    body("phone").customSanitizer((value) => {
      return value.replace(/[^\d]/g, ""); // Keep digits only
    }),
  ],
  (req, res) => {
    // Data sudah clean
  },
);
```

---

## Validasi dengan Template Engine

**Menampilkan error di form HTML:**

```javascript
app.post(
  "/register",
  [
    body("username").notEmpty().withMessage("Username required"),
    body("email").isEmail().withMessage("Valid email required"),
    body("password").isLength({ min: 8 }).withMessage("Password min 8 chars"),
  ],
  (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      // Re-render form dengan error messages
      return res.render("register", {
        errors: errors.array(),
        formData: req.body, // Keep user input
      });
    }

    // Process registration...
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

  <div>
    <label>Email:</label>
    <input type="email" name="email" value="{{formData.email}}" />
    {{#each errors}}
      {{#if (eq this.path "email")}}
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

## Validation Best Practices

✅ **DO:**

- Validasi di server-side (wajib)
- Gunakan validation library (express-validator, Joi, Zod)
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
  body("name")
    .trim()
    .notEmpty()
    .withMessage("Product name required")
    .isLength({ max: 100 }),

  body("price")
    .notEmpty()
    .withMessage("Price required")
    .isFloat({ min: 0 })
    .withMessage("Price must be positive"),

  body("stock")
    .notEmpty()
    .isInt({ min: 0 })
    .withMessage("Stock must be non-negative integer"),

  body("category")
    .notEmpty()
    .isIn(["electronics", "clothing", "food"])
    .withMessage("Invalid category"),

  body("description").optional().trim().isLength({ max: 1000 }),
];

module.exports = { createProductValidation };
```

---

## Contoh Lengkap: Route Implementation

```javascript
// routes/productRoutes.js
const express = require("express");
const { validationResult } = require("express-validator");
const { createProductValidation } = require("../validators/productValidator");
const router = express.Router();

router.post("/products", createProductValidation, async (req, res) => {
  // Check validation results
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array().map((err) => ({
        field: err.path,
        message: err.msg,
      })),
    });
  }

  try {
    // Data is valid, save to database
    const product = await Product.create(req.body);
    res.status(201).json({
      success: true,
      data: product,
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: "Server error",
    });
  }
});

module.exports = router;
```

---

## Summary

- **Validation** penting untuk security, data integrity, dan UX
- **Server-side validation** adalah wajib, client-side adalah bonus
- **Express-validator** bagus untuk Express, **Joi** untuk schema-based, **Zod** untuk TypeScript
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

- express-validator: https://express-validator.github.io
- Joi: https://joi.dev
- Zod: https://zod.dev
- OWASP Input Validation: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
