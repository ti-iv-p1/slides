---
title: Routing di Node.js dan Express.js
version: 1.0.0
header: Routing di Node.js dan Express.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Routing di Node.js dan Express.js

---

## Tujuan Pembelajaran

- Memahami konsep routing dalam web application
- Menguasai cara membuat routes di Express.js
- Memahami HTTP methods (GET, POST, PUT, DELETE)
- Mampu mengorganisir routes dengan Express Router
- Menerapkan route parameters dan query strings

---

## Apa itu Routing?

**Routing** adalah proses menentukan bagaimana aplikasi merespons request dari client ke endpoint tertentu

**Endpoint** = URL path + HTTP method

Contoh:

- `GET /users` - mengambil daftar users
- `POST /users` - membuat user baru
- `GET /users/5` - mengambil user dengan id 5
- `DELETE /users/5` - menghapus user dengan id 5

---

## Struktur Route

```javascript
app.METHOD(PATH, HANDLER);
```

- **app** - instance Express
- **METHOD** - HTTP method (get, post, put, delete, dll)
- **PATH** - path di server
- **HANDLER** - function yang dijalankan saat route match

---

## Route Sederhana

```javascript
const express = require("express");
const app = express();

// GET request
app.get("/", (req, res) => {
  res.send("Hello World!");
});

// GET request dengan path berbeda
app.get("/about", (req, res) => {
  res.send("About Page");
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## HTTP Methods

**GET** - mengambil data

```javascript
app.get("/users", (req, res) => {
  res.json({ users: [] });
});
```

**POST** - membuat data baru

```javascript
app.post("/users", (req, res) => {
  res.json({ message: "User created" });
});
```

**PUT** - update data (full update)

```javascript
app.put("/users/:id", (req, res) => {
  res.json({ message: "User updated" });
});
```

---

## HTTP Methods (lanjutan)

**PATCH** - update data (partial update)

```javascript
app.patch("/users/:id", (req, res) => {
  res.json({ message: "User partially updated" });
});
```

**DELETE** - menghapus data

```javascript
app.delete("/users/:id", (req, res) => {
  res.json({ message: "User deleted" });
});
```

---

## Route Parameters

**Route parameters** adalah named URL segments untuk menangkap nilai

```javascript
// Pattern: /users/:id
app.get("/users/:id", (req, res) => {
  const userId = req.params.id;
  res.send(`User ID: ${userId}`);
});

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req, res) => {
  const { userId, postId } = req.params;
  res.send(`User: ${userId}, Post: ${postId}`);
});
```

**Akses:** `http://localhost:3000/users/123`
**Result:** `req.params.id = "123"`

---

## Query Strings

**Query strings** adalah parameter setelah `?` di URL

```javascript
app.get("/search", (req, res) => {
  const keyword = req.query.q;
  const page = req.query.page || 1;

  res.json({
    keyword: keyword,
    page: page,
  });
});
```

**Akses:** `http://localhost:3000/search?q=nodejs&page=2`
**Result:** `req.query.q = "nodejs"`, `req.query.page = "2"`

---

## Request Body

Untuk data dari POST/PUT/PATCH request:

```javascript
// Middleware untuk parsing JSON
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.post("/users", (req, res) => {
  const { name, email } = req.body;

  res.json({
    message: "User created",
    data: { name, email },
  });
});
```

**Client mengirim:**

```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

## Response Methods

```javascript
// Send text
res.send("Hello World");

// Send JSON
res.json({ message: "Success", data: [] });

// Send status code
res.status(404).send("Not Found");
res.status(201).json({ message: "Created" });

// Redirect
res.redirect("/login");

// Render view
res.render("index", { title: "Home" });

// Download file
res.download("/path/to/file.pdf");
```

---

## Express Router

**Express Router** adalah cara untuk mengorganisir routes menjadi modular

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.json({ users: [] });
});

router.get("/:id", (req, res) => {
  res.json({ user: { id: req.params.id } });
});

router.post("/", (req, res) => {
  res.status(201).json({ message: "User created" });
});

module.exports = router;
```

---

## Menggunakan Router di App

```javascript
// app.js
const express = require("express");
const app = express();

// Import router
const userRoutes = require("./routes/users");
const productRoutes = require("./routes/products");

// Use router dengan prefix
app.use("/api/users", userRoutes);
app.use("/api/products", productRoutes);

app.listen(3000);
```

**Hasil:**

- `/api/users` → GET all users
- `/api/users/:id` → GET user by id
- `/api/products` → GET all products

---

## Route Chaining

Menggabungkan multiple methods untuk satu path:

```javascript
app
  .route("/users")
  .get((req, res) => {
    res.json({ users: [] });
  })
  .post((req, res) => {
    res.status(201).json({ message: "Created" });
  });

app
  .route("/users/:id")
  .get((req, res) => {
    res.json({ user: {} });
  })
  .put((req, res) => {
    res.json({ message: "Updated" });
  })
  .delete((req, res) => {
    res.json({ message: "Deleted" });
  });
```

---

## Middleware di Routes

**Middleware** adalah function yang punya akses ke `req`, `res`, dan `next`

```javascript
// Logger middleware
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // Lanjut ke handler berikutnya
};

// Gunakan middleware
app.get("/users", logger, (req, res) => {
  res.json({ users: [] });
});

// Atau gunakan untuk semua routes
app.use(logger);
```

---

## Authentication Middleware

```javascript
// middleware/auth.js
const authenticateToken = (req, res, next) => {
  const token = req.headers["authorization"];

  if (!token) {
    return res.status(401).json({ error: "Unauthorized" });
  }

  // Verify token (contoh sederhana)
  if (token === "valid-token") {
    next();
  } else {
    res.status(403).json({ error: "Forbidden" });
  }
};

// Gunakan di route
app.get("/protected", authenticateToken, (req, res) => {
  res.json({ message: "This is protected" });
});
```

---

## Route dengan Multiple Middleware

```javascript
const validate = (req, res, next) => {
  if (!req.body.name) {
    return res.status(400).json({ error: "Name is required" });
  }
  next();
};

const sanitize = (req, res, next) => {
  req.body.name = req.body.name.trim();
  next();
};

app.post("/users", validate, sanitize, (req, res) => {
  res.json({ message: "User created" });
});
```

**Flow:** validate → sanitize → handler

---

## Struktur Folder Routes

```
project/
├── app.js
├── routes/
│   ├── index.js       (main router)
│   ├── users.js       (user routes)
│   ├── products.js    (product routes)
│   └── auth.js        (auth routes)
├── controllers/
│   ├── userController.js
│   └── productController.js
└── middleware/
    ├── auth.js
    └── validation.js
```

---

## Main Router

```javascript
// routes/index.js
const express = require("express");
const router = express.Router();

const userRoutes = require("./users");
const productRoutes = require("./products");
const authRoutes = require("./auth");

router.use("/users", userRoutes);
router.use("/products", productRoutes);
router.use("/auth", authRoutes);

module.exports = router;
```

```javascript
// app.js
const routes = require("./routes");
app.use("/api", routes);
```

---

## User Routes dengan Controller

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();
const UserController = require("../controllers/userController");
const { authenticate } = require("../middleware/auth");

router.get("/", UserController.index);
router.get("/:id", UserController.show);
router.post("/", authenticate, UserController.create);
router.put("/:id", authenticate, UserController.update);
router.delete("/:id", authenticate, UserController.delete);

module.exports = router;
```

Lebih clean dan terorganisir!

---

## Route Pattern Matching

```javascript
// Exact match
app.get("/users", handler);

// Parameter
app.get("/users/:id", handler);

// Optional parameter (?)
app.get("/users/:id?", handler);
// Match: /users dan /users/123

// Wildcard (*)
app.get("/files/*", handler);
// Match: /files/any/path/here

// Regular expression
app.get(/.*flight$/, handler);
// Match: /flight, /airflight, dll
```

---

## Route Priority

Routes dievaluasi **dalam urutan definisi**:

```javascript
// ❌ SALAH - Route spesifik setelah wildcard
app.get("/users/:id", handler);
app.get("/users/me", handler); // Tidak akan tercapai!

// ✅ BENAR - Route spesifik terlebih dahulu
app.get("/users/me", handler);
app.get("/users/:id", handler);
```

Route yang lebih spesifik harus didefinisikan terlebih dahulu!

---

## Error Handling di Routes

```javascript
// Synchronous error
app.get("/users/:id", (req, res) => {
  const user = findUserById(req.params.id);
  if (!user) {
    return res.status(404).json({ error: "User not found" });
  }
  res.json({ user });
});

// Asynchronous error
app.get("/users", async (req, res, next) => {
  try {
    const users = await fetchUsers();
    res.json({ users });
  } catch (error) {
    next(error); // Pass ke error handler
  }
});
```

---

## 404 Handler

```javascript
// Tangani route yang tidak ditemukan
// Letakkan di akhir setelah semua routes
app.use((req, res, next) => {
  res.status(404).json({
    error: "Route not found",
    path: req.url,
  });
});
```

---

## Global Error Handler

```javascript
// Error handler middleware (harus 4 parameter)
app.use((err, req, res, next) => {
  console.error(err.stack);

  const statusCode = err.statusCode || 500;
  const message = err.message || "Internal Server Error";

  res.status(statusCode).json({
    success: false,
    error: message,
    ...(process.env.NODE_ENV === "development" && {
      stack: err.stack,
    }),
  });
});
```

---

## RESTful API Routes

```javascript
// routes/products.js
const router = require("express").Router();

// GET /api/products - List all products
router.get("/", ProductController.index);

// GET /api/products/:id - Get single product
router.get("/:id", ProductController.show);

// POST /api/products - Create product
router.post("/", ProductController.create);

// PUT /api/products/:id - Update product
router.put("/:id", ProductController.update);

// DELETE /api/products/:id - Delete product
router.delete("/:id", ProductController.delete);

module.exports = router;
```

---

## Nested Routes

```javascript
// routes/users.js
const router = require("express").Router();
const postRoutes = require("./posts");

router.get("/:userId", UserController.show);

// Nested resource: /users/:userId/posts
router.use(
  "/:userId/posts",
  (req, res, next) => {
    req.userId = req.params.userId; // Pass userId
    next();
  },
  postRoutes,
);

module.exports = router;
```

```javascript
// routes/posts.js
router.get("/", (req, res) => {
  const userId = req.userId; // Dari parent router
  res.json({ posts: [], userId });
});
```

---

## Route Versioning

```javascript
// Versioning dengan prefix
const v1Routes = require("./routes/v1");
const v2Routes = require("./routes/v2");

app.use("/api/v1", v1Routes);
app.use("/api/v2", v2Routes);
```

```javascript
// routes/v1/users.js
router.get("/", (req, res) => {
  res.json({ version: 1, users: [] });
});

// routes/v2/users.js
router.get("/", (req, res) => {
  res.json({ version: 2, users: [], meta: {} });
});
```

---

## Route dengan Validation

```javascript
const { body, param, validationResult } = require("express-validator");

router.post(
  "/users",
  // Validation rules
  body("email").isEmail().normalizeEmail(),
  body("name").trim().notEmpty(),
  body("age").isInt({ min: 1, max: 120 }),

  // Check validation
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  },

  // Handler
  UserController.create,
);
```

---

## Rate Limiting

```javascript
const rateLimit = require("express-rate-limit");

// Limit 100 requests per 15 minutes
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: "Too many requests from this IP",
});

// Apply to all routes
app.use("/api/", limiter);

// Apply to specific routes
app.post("/api/login", limiter, AuthController.login);
```

---

## CORS di Routes

```javascript
const cors = require("cors");

// Enable CORS untuk semua routes
app.use(cors());

// Enable CORS untuk routes tertentu
app.use("/api", cors(), apiRoutes);

// Custom CORS options
const corsOptions = {
  origin: "https://example.com",
  methods: ["GET", "POST"],
  credentials: true,
};

app.use("/api", cors(corsOptions), apiRoutes);
```

---

## Route Documentation

```javascript
/**
 * @route   GET /api/users
 * @desc    Get all users
 * @access  Public
 */
router.get("/", UserController.index);

/**
 * @route   POST /api/users
 * @desc    Create new user
 * @access  Private (Admin only)
 */
router.post("/", authenticate, isAdmin, UserController.create);

/**
 * @route   PUT /api/users/:id
 * @desc    Update user by ID
 * @access  Private
 */
router.put("/:id", authenticate, UserController.update);
```

---

## Testing Routes

```javascript
// tests/users.test.js
const request = require("supertest");
const app = require("../app");

describe("User Routes", () => {
  test("GET /api/users should return users", async () => {
    const response = await request(app).get("/api/users").expect(200);

    expect(response.body).toHaveProperty("users");
  });

  test("POST /api/users should create user", async () => {
    const response = await request(app)
      .post("/api/users")
      .send({ name: "John", email: "john@example.com" })
      .expect(201);

    expect(response.body.message).toBe("User created");
  });
});
```

---

## Best Practices

1. **Gunakan Express Router** untuk modularitas
2. **RESTful naming** - gunakan kata benda, bukan kata kerja
3. **Versioning API** - untuk backward compatibility
4. **Consistent response format** - JSON dengan struktur sama
5. **Error handling** - gunakan try-catch dan error middleware
6. **Validation** - validasi semua input
7. **Documentation** - dokumentasikan setiap route
8. **Security** - gunakan auth, rate limiting, CORS

---

## Naming Convention

✅ **GOOD:**

```
GET    /users           - Get all users
GET    /users/:id       - Get user by ID
POST   /users           - Create user
PUT    /users/:id       - Update user
DELETE /users/:id       - Delete user
```

❌ **BAD:**

```
GET    /getUsers
POST   /createUser
GET    /user-detail/:id
DELETE /deleteUser/:id
```

---

## Response Format Guidelines

```javascript
// Success response
res.json({
  success: true,
  data: {},
  message: "Operation successful",
});

// Error response
res.status(400).json({
  success: false,
  error: "Error message",
  errors: [], // Detail errors (optional)
});

// List response with pagination
res.json({
  success: true,
  data: [],
  meta: {
    page: 1,
    limit: 10,
    total: 100,
  },
});
```

---

## Kesalahan Umum

❌ Tidak menangani error
❌ Route terlalu spesifik di awal
❌ Tidak menggunakan Router untuk modularitas
❌ Tidak konsisten dalam naming
❌ Lupa middleware `next()`
❌ Tidak validasi input
❌ Expose sensitive data di response
❌ Tidak menggunakan HTTP status code yang tepat

---

## HTTP Status Codes

**2xx Success:**

- 200 OK - Request berhasil
- 201 Created - Resource created
- 204 No Content - Success, no response body

**4xx Client Error:**

- 400 Bad Request - Invalid input
- 401 Unauthorized - Not authenticated
- 403 Forbidden - No permission
- 404 Not Found - Resource not found

**5xx Server Error:**

- 500 Internal Server Error

---

## Resources & Referensi

- [Express Routing Guide](https://expressjs.com/en/guide/routing.html)
- [Express Router API](https://expressjs.com/en/4x/api.html#router)
- [RESTful API Design](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [express-validator](https://express-validator.github.io/)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Happy Coding! 🚀
