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
- Memahami HTTP methods untuk web (GET, POST)
- Mampu mengorganisir routes dengan Express Router
- Menerapkan route parameters, query strings, dan form handling
- Mengintegrasikan routing dengan template engine

---

## Apa itu Routing?

**Routing** adalah proses menentukan bagaimana aplikasi merespons request dari client ke endpoint tertentu

**Endpoint** = URL path + HTTP method

Contoh:

- `GET /` - menampilkan halaman home
- `GET /users` - menampilkan halaman daftar users
- `GET /users/5` - menampilkan halaman detail user dengan id 5
- `POST /users` - memproses form tambah user baru

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

// Set view engine
app.set("view engine", "ejs");

// GET request - render home page
app.get("/", (req, res) => {
  res.render("index", { title: "Home" });
});

// GET request - render about page
app.get("/about", (req, res) => {
  res.render("about", { title: "About Us" });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## HTTP Methods untuk Web Application

**GET** - menampilkan halaman atau data

```javascript
app.get("/users", (req, res) => {
  const users = [
    { id: 1, name: "John" },
    { id: 2, name: "Jane" },
  ];
  res.render("users/index", { users });
});
```

**POST** - memproses form submission

```javascript
app.post("/users", (req, res) => {
  const { name, email } = req.body;
  // Simpan user ke database
  // Redirect setelah berhasil (Post-Redirect-Get pattern)
  res.redirect("/users");
});
```

---

## Static Files

Serve static files (CSS, JavaScript, images):

```javascript
const express = require("express");
const app = express();

// Serve static files dari folder 'public'
app.use(express.static("public"));

// Akses: http://localhost:3000/css/style.css
// File: public/css/style.css
```

**Struktur folder:**

```
public/
├── css/
│   └── style.css
├── js/
│   └── script.js
└── images/
    └── logo.png
```

---

## Route Parameters

**Route parameters** adalah named URL segments untuk menangkap nilai

```javascript
// Pattern: /users/:id
app.get("/users/:id", (req, res) => {
  const userId = req.params.id;
  // Ambil data user dari database berdasarkan id
  const user = { id: userId, name: "John Doe" };
  res.render("users/show", { user });
});

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req, res) => {
  const { userId, postId } = req.params;
  // Render halaman detail post dari user tertentu
  res.render("posts/show", { userId, postId });
});
```

**Akses:** `http://localhost:3000/users/123`
**Result:** Menampilkan halaman detail user dengan id 123

---

## Query Strings

**Query strings** adalah parameter setelah `?` di URL

```javascript
app.get("/products", (req, res) => {
  const category = req.query.category || "all";
  const page = req.query.page || 1;

  // Filter products berdasarkan category
  const products = getProductsByCategory(category, page);

  res.render("products/index", {
    products,
    category,
    currentPage: page,
  });
});
```

**Akses:** `http://localhost:3000/products?category=electronics&page=2`
**Result:** Menampilkan halaman produk kategori electronics halaman 2

---

## Form Handling

Memproses data dari HTML form:

```javascript
// Middleware untuk parsing form data
app.use(express.urlencoded({ extended: true }));

// GET - Tampilkan form
app.get("/users/new", (req, res) => {
  res.render("users/new");
});

// POST - Proses form
app.post("/users", (req, res) => {
  const { name, email } = req.body;
  // Simpan ke database
  // Redirect ke halaman users (Post-Redirect-Get pattern)
  res.redirect("/users");
});
```

**HTML Form:**

```html
<form action="/users" method="POST">
  <input type="text" name="name" placeholder="Name" />
  <input type="email" name="email" placeholder="Email" />
  <button type="submit">Submit</button>
</form>
```

---

## Response Methods untuk Web App

```javascript
// Render template (halaman web)
res.render("index", { title: "Home", user: currentUser });

// Redirect ke halaman lain
res.redirect("/login");
res.redirect("/users/" + userId);

// Send HTML string
res.send("<h1>Hello World</h1>");

// Send status code dengan halaman custom
res.status(404).render("errors/404");
res.status(500).render("errors/500");

// Download file
res.download("/files/report.pdf");
```

---

## Express Router

**Express Router** adalah cara untuk mengorganisir routes menjadi modular

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();

// GET /users - Tampilkan daftar users
router.get("/", (req, res) => {
  const users = []; // Ambil dari database
  res.render("users/index", { users });
});

// GET /users/:id - Tampilkan detail user
router.get("/:id", (req, res) => {
  const user = {}; // Ambil dari database
  res.render("users/show", { user });
});

// POST /users - Proses form tambah user
router.post("/", (req, res) => {
  // Simpan user
  res.redirect("/users");
});

module.exports = router;
```

---

## Menggunakan Router di App

```javascript
// app.js
const express = require("express");
const app = express();

// Set view engine
app.set("view engine", "ejs");
app.use(express.static("public"));
app.use(express.urlencoded({ extended: true }));

// Import router
const userRoutes = require("./routes/users");
const productRoutes = require("./routes/products");

// Use router dengan prefix
app.use("/users", userRoutes);
app.use("/products", productRoutes);

app.listen(3000);
```

**Hasil:**

- `/users` → Halaman daftar users
- `/users/:id` → Halaman detail user
- `/products` → Halaman daftar produk

---

## Post-Redirect-Get Pattern

**PRG Pattern** mencegah form re-submission saat refresh:

```javascript
// GET - Tampilkan form
app.get("/users/new", (req, res) => {
  res.render("users/new");
});

// POST - Proses form, lalu redirect
app.post("/users", (req, res) => {
  const { name, email } = req.body;
  const userId = saveUserToDatabase(name, email);
  // Redirect setelah POST berhasil
  res.redirect(`/users/${userId}`);
});

// GET - Tampilkan hasil
app.get("/users/:id", (req, res) => {
  const user = getUserFromDatabase(req.params.id);
  res.render("users/show", { user });
});
```

✅ Menghindari duplicate submission saat user refresh halaman

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
  const users = [];
  res.render("users/index", { users });
});

// Atau gunakan untuk semua routes
app.use(logger);
```

---

## Authentication Middleware

```javascript
// middleware/auth.js
const isAuthenticated = (req, res, next) => {
  // Cek session atau cookie
  if (req.session && req.session.userId) {
    next(); // User sudah login
  } else {
    // Redirect ke login page
    res.redirect("/login");
  }
};

// Gunakan di route yang perlu authentication
app.get("/dashboard", isAuthenticated, (req, res) => {
  res.render("dashboard", { user: req.session.user });
});

app.get("/profile", isAuthenticated, (req, res) => {
  res.render("profile", { user: req.session.user });
});
```

---

## Route dengan Multiple Middleware

```javascript
const validate = (req, res, next) => {
  if (!req.body.name || !req.body.email) {
    // Render form lagi dengan error message
    return res.render("users/new", {
      error: "Name and email are required",
    });
  }
  next();
};

const sanitize = (req, res, next) => {
  req.body.name = req.body.name.trim();
  req.body.email = req.body.email.trim().toLowerCase();
  next();
};

app.post("/users", validate, sanitize, (req, res) => {
  // Simpan user
  res.redirect("/users");
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
const { isAuthenticated } = require("../middleware/auth");

// Tampilkan halaman daftar users
router.get("/", UserController.index);

// Tampilkan form tambah user
router.get("/new", isAuthenticated, UserController.new);

// Tampilkan detail user
router.get("/:id", UserController.show);

// Proses form tambah user
router.post("/", isAuthenticated, UserController.create);

// Tampilkan form edit user
router.get("/:id/edit", isAuthenticated, UserController.edit);

// Proses form edit user
router.post("/:id", isAuthenticated, UserController.update);

module.exports = router;
```

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
// Synchronous error - render error page
app.get("/users/:id", (req, res) => {
  const user = findUserById(req.params.id);
  if (!user) {
    return res.status(404).render("errors/404", {
      message: "User not found",
    });
  }
  res.render("users/show", { user });
});

// Asynchronous error
app.get("/users", async (req, res, next) => {
  try {
    const users = await fetchUsers();
    res.render("users/index", { users });
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
  res.status(404).render("errors/404", {
    url: req.url,
  });
});
```

**views/errors/404.ejs:**

```html
<h1>404 - Page Not Found</h1>
<p>The page <%= url %> does not exist.</p>
<a href="/">Go to Home</a>
```

---

## Global Error Handler

```javascript
// Error handler middleware (harus 4 parameter)
app.use((err, req, res, next) => {
  console.error(err.stack);

  const statusCode = err.statusCode || 500;
  const message = err.message || "Internal Server Error";

  res.status(statusCode).render("errors/500", {
    error: message,
    // Tampilkan stack trace hanya di development
    stack: process.env.NODE_ENV === "development" ? err.stack : null,
  });
});
```

**views/errors/500.ejs:**

```html
<h1>500 - Internal Server Error</h1>
<p><%= error %></p>
<% if (stack) { %>
<pre><%= stack %></pre>
<% } %>
```

---

## Complete CRUD Routes Example

```javascript
// routes/products.js
const router = require("express").Router();
const ProductController = require("../controllers/productController");

// GET /products - Tampilkan daftar produk
router.get("/", ProductController.index);

// GET /products/new - Form tambah produk
router.get("/new", ProductController.new);

// POST /products - Proses form tambah
router.post("/", ProductController.create);

// GET /products/:id - Detail produk
router.get("/:id", ProductController.show);

// GET /products/:id/edit - Form edit produk
router.get("/:id/edit", ProductController.edit);

// POST /products/:id - Proses form edit
router.post("/:id", ProductController.update);

// POST /products/:id/delete - Hapus produk
router.post("/:id/delete", ProductController.delete);

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
  const userId = req.userId;
  const posts = getPostsByUserId(userId);
  res.render("posts/index", { posts, userId });
});
```

---

## Template Engine Integration

Menggunakan EJS sebagai template engine:

```javascript
// app.js
const express = require("express");
const app = express();

// Set EJS sebagai view engine
app.set("view engine", "ejs");
app.set("views", "./views");

// Route
app.get("/", (req, res) => {
  res.render("index", {
    title: "Welcome",
    user: { name: "John" },
  });
});
```

**views/index.ejs:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
  </head>
  <body>
    <h1>Hello, <%= user.name %>!</h1>
  </body>
</html>
```

---

## Form Validation

```javascript
const { body, validationResult } = require("express-validator");

// GET form
router.get("/users/new", (req, res) => {
  res.render("users/new", { errors: null });
});

// POST with validation
router.post(
  "/users",
  // Validation rules
  body("email").isEmail().normalizeEmail(),
  body("name").trim().notEmpty(),
  body("password").isLength({ min: 6 }),

  // Check validation
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      // Render form lagi dengan error messages
      return res.render("users/new", {
        errors: errors.array(),
        formData: req.body,
      });
    }
    // Simpan user dan redirect
    saveUser(req.body);
    res.redirect("/users");
  },
);
```

---

## Flash Messages

Menampilkan pesan sukses/error setelah redirect:

```javascript
const session = require("express-session");
const flash = require("connect-flash");

app.use(session({ secret: "secret-key" }));
app.use(flash());

// Middleware untuk pass flash messages ke views
app.use((req, res, next) => {
  res.locals.success = req.flash("success");
  res.locals.error = req.flash("error");
  next();
});

// Di controller
router.post("/users", (req, res) => {
  saveUser(req.body);
  req.flash("success", "User created successfully!");
  res.redirect("/users");
});
```

**views/layout.ejs:**

```html
<% if (success.length > 0) { %>
<div class="alert success"><%= success %></div>
<% } %>
```

---

## File Upload Handling

```javascript
const multer = require("multer");

// Configure multer untuk menyimpan file
const storage = multer.diskStorage({
  destination: "./public/uploads/",
  filename: (req, file, cb) => {
    cb(null, Date.now() + "-" + file.originalname);
  },
});

const upload = multer({ storage });

// Route untuk upload
router.post("/upload", upload.single("avatar"), (req, res) => {
  // req.file berisi info file yang diupload
  const filePath = "/uploads/" + req.file.filename;
  res.redirect("/profile");
});
```

**HTML Form:**

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="avatar" />
  <button type="submit">Upload</button>
</form>
```

---

## Session Management

```javascript
const session = require("express-session");

app.use(
  session({
    secret: "your-secret-key",
    resave: false,
    saveUninitialized: false,
    cookie: { maxAge: 3600000 }, // 1 hour
  }),
);

// Login route
app.post("/login", (req, res) => {
  const { email, password } = req.body;
  const user = authenticateUser(email, password);

  if (user) {
    req.session.userId = user.id;
    req.session.user = user;
    res.redirect("/dashboard");
  } else {
    res.render("login", { error: "Invalid credentials" });
  }
});

// Logout route
app.post("/logout", (req, res) => {
  req.session.destroy();
  res.redirect("/login");
});
```

---

## Layout dan Partials

**views/layout.ejs:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel="stylesheet" href="/css/style.css" />
  </head>
  <body>
    <%- include('partials/header') %>
    <main><%- body %></main>
    <%- include('partials/footer') %>
  </body>
</html>
```

**views/partials/header.ejs:**

```html
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/products">Products</a>
    <a href="/about">About</a>
  </nav>
</header>
```

**views/partials/footer.ejs:**

```html
<footer>
  <p>&copy; 2026 My Website</p>
</footer>
```

---

## Best Practices

1. **Gunakan Express Router** untuk modularitas
2. **Post-Redirect-Get pattern** - hindari duplicate submission
3. **Validation** - validasi semua form input
4. **Error handling** - render error pages yang informatif
5. **Session management** - untuk authentication
6. **Flash messages** - untuk feedback ke user
7. **Consistent naming** - gunakan kata benda untuk resources
8. **Security** - sanitize input, gunakan CSRF protection

---

## Naming Convention

✅ **GOOD:**

```
GET    /users           - Halaman daftar users
GET    /users/new       - Form tambah user
GET    /users/:id       - Halaman detail user
POST   /users           - Proses form tambah
GET    /users/:id/edit  - Form edit user
POST   /users/:id       - Proses form edit
POST   /users/:id/delete - Hapus user
```

❌ **BAD:**

```
GET    /getUsers
POST   /createUser
GET    /user-detail/:id
GET    /deleteUser/:id
```

---

## Pagination untuk Web App

```javascript
app.get("/products", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = 10;
  const offset = (page - 1) * limit;

  const products = await getProducts(limit, offset);
  const totalProducts = await getTotalProducts();
  const totalPages = Math.ceil(totalProducts / limit);

  res.render("products/index", {
    products,
    currentPage: page,
    totalPages,
    hasNextPage: page < totalPages,
    hasPrevPage: page > 1,
  });
});
```

**View pagination:**

```html
<% if (hasPrevPage) { %>
<a href="?page=<%= currentPage - 1 %>">Previous</a>
<% } %> <%= currentPage %> of <%= totalPages %> <% if (hasNextPage) { %>
<a href="?page=<%= currentPage + 1 %>">Next</a>
<% } %>
```

---

## Kesalahan Umum

❌ Tidak menangani error dengan error pages
❌ Route terlalu spesifik di awal
❌ Tidak menggunakan Router untuk modularitas
❌ Tidak menggunakan Post-Redirect-Get pattern
❌ Lupa middleware `next()`
❌ Tidak validasi form input
❌ Tidak menggunakan flash messages untuk feedback
❌ Lupa setup static files
❌ Tidak sanitize user input

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
- [EJS Template Engine](https://ejs.co/)
- [express-session](https://github.com/expressjs/session)
- [express-validator](https://express-validator.github.io/)
- [Multer (File Upload)](https://github.com/expressjs/multer)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Happy Coding! 🚀
