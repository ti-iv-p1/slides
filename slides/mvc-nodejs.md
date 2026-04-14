---
title: MVC Pattern di Node.js
version: 1.0.0
header: MVC Pattern di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# MVC Pattern di Node.js

---

## Tujuan Pembelajaran

- Memahami konsep MVC (Model-View-Controller)
- Mengerti peran masing-masing komponen MVC
- Mampu mengimplementasikan MVC di Node.js
- Menerapkan best practices struktur aplikasi

---

## Apa itu MVC?

**MVC** adalah design pattern untuk memisahkan aplikasi menjadi 3 komponen utama:

- **Model** - mengelola data dan logika bisnis
- **View** - menampilkan data ke user
- **Controller** - menerima input dan mengatur Model & View

---

## Mengapa Menggunakan MVC?

✅ **Separation of Concerns** - setiap bagian punya tanggung jawab jelas
✅ **Maintainability** - mudah di-maintain dan di-update
✅ **Scalability** - mudah dikembangkan
✅ **Team Collaboration** - tim bisa kerja parallel
✅ **Reusability** - komponen bisa digunakan ulang
✅ **Testing** - lebih mudah melakukan unit testing

---

## Alur Kerja MVC

```
User Request
     ↓
Controller (menerima request)
     ↓
Model (ambil/proses data)
     ↓
Controller (terima data dari Model)
     ↓
View (render data)
     ↓
Response ke User
```

---

## Model

**Model** bertanggung jawab untuk:

- Mengelola data aplikasi
- Berinteraksi dengan database
- Validasi data
- Logika bisnis

**Model tidak tahu** tentang Controller atau View

---

## Contoh Model - User Model

```javascript
// models/User.js
const db = require("../config/database");

class User {
  static async findAll() {
    const [rows] = await db.query("SELECT * FROM users");
    return rows;
  }

  static async findById(id) {
    const [rows] = await db.query("SELECT * FROM users WHERE id = ?", [id]);
    return rows[0];
  }

  static async create(userData) {
    const { name, email, password } = userData;
    const [result] = await db.query(
      "INSERT INTO users (name, email, password) VALUES (?, ?, ?)",
      [name, email, password],
    );
    return result.insertId;
  }
}

module.exports = User;
```

---

## View

**View** bertanggung jawab untuk:

- Menampilkan data ke user
- User interface / presentation layer
- Template rendering (EJS, Pug, Handlebars)

**View tidak boleh** mengandung logika bisnis

---

## Contoh View - EJS Template

```html
<!-- views/users/index.ejs -->
<!DOCTYPE html>
<html>
  <head>
    <title>Daftar Users</title>
  </head>
  <body>
    <h1>Daftar Users</h1>
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Nama</th>
          <th>Email</th>
        </tr>
      </thead>
      <tbody>
        <% users.forEach(user => { %>
        <tr>
          <td><%= user.id %></td>
          <td><%= user.name %></td>
          <td><%= user.email %></td>
        </tr>
        <% }); %>
      </tbody>
    </table>
  </body>
</html>
```

---

## Controller

**Controller** bertanggung jawab untuk:

- Menerima request dari user
- Memproses input
- Memanggil Model untuk data
- Memilih View untuk render
- Mengirim response

**Controller** adalah penghubung antara Model dan View

---

## Contoh Controller

```javascript
// controllers/userController.js
const User = require("../models/User");

class UserController {
  // GET /users
  static async index(req, res) {
    try {
      const users = await User.findAll();
      res.render("users/index", { users });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  // GET /users/:id
  static async show(req, res) {
    try {
      const user = await User.findById(req.params.id);
      if (!user) {
        return res.status(404).json({ error: "User not found" });
      }
      res.render("users/show", { user });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
}

module.exports = UserController;
```

---

## Contoh Controller (lanjutan)

```javascript
// controllers/userController.js (continued)

  // POST /users
  static async create(req, res) {
    try {
      const userId = await User.create(req.body);
      res.status(201).json({
        message: 'User created',
        userId
      });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  // PUT /users/:id
  static async update(req, res) {
    try {
      await User.update(req.params.id, req.body);
      res.json({ message: 'User updated' });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
```

---

## Routes

**Routes** menghubungkan URL dengan Controller

```javascript
// routes/userRoutes.js
const express = require("express");
const router = express.Router();
const UserController = require("../controllers/userController");

router.get("/users", UserController.index);
router.get("/users/:id", UserController.show);
router.post("/users", UserController.create);
router.put("/users/:id", UserController.update);
router.delete("/users/:id", UserController.delete);

module.exports = router;
```

---

## Struktur Folder MVC

```
project/
├── app.js (entry point)
├── config/
│   ├── database.js
│   └── config.js
├── models/
│   ├── User.js
│   └── Product.js
├── views/
│   ├── users/
│   │   ├── index.ejs
│   │   └── show.ejs
│   └── layouts/
│       └── main.ejs
├── controllers/
│   ├── userController.js
│   └── productController.js
├── routes/
│   ├── userRoutes.js
│   └── productRoutes.js
└── public/
    ├── css/
    ├── js/
    └── images/
```

---

## Setup Express dengan MVC

```javascript
// app.js
const express = require("express");
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));

// View engine setup
app.set("view engine", "ejs");
app.set("views", "./views");

// Routes
const userRoutes = require("./routes/userRoutes");
app.use("/api", userRoutes);

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## Model dengan Mongoose (MongoDB)

```javascript
// models/User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
    },
    password: {
      type: String,
      required: true,
    },
  },
  {
    timestamps: true,
  },
);

module.exports = mongoose.model("User", userSchema);
```

---

## Controller dengan Mongoose

```javascript
// controllers/userController.js
const User = require("../models/User");

exports.getAllUsers = async (req, res) => {
  try {
    const users = await User.find();
    res.render("users/index", { users });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.createUser = async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json({ message: "User created", user });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};
```

---

## MVC untuk API (tanpa View)

Untuk REST API, kita bisa menggunakan **MC Pattern**:

```javascript
// controllers/apiUserController.js
const User = require("../models/User");

exports.index = async (req, res) => {
  try {
    const users = await User.findAll();
    res.json({ success: true, data: users });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
};
```

Response dalam format JSON, bukan HTML

---

## Middleware dalam MVC

```javascript
// middleware/auth.js
const jwt = require("jsonwebtoken");

exports.authenticate = (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(" ")[1];
    if (!token) {
      return res.status(401).json({ error: "Unauthorized" });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
  } catch (error) {
    res.status(401).json({ error: "Invalid token" });
  }
};
```

---

## Menggunakan Middleware di Routes

```javascript
// routes/userRoutes.js
const express = require("express");
const router = express.Router();
const UserController = require("../controllers/userController");
const { authenticate } = require("../middleware/auth");

// Public routes
router.post("/login", UserController.login);
router.post("/register", UserController.register);

// Protected routes
router.get("/users", authenticate, UserController.index);
router.get("/users/:id", authenticate, UserController.show);
router.put("/users/:id", authenticate, UserController.update);

module.exports = router;
```

---

## Service Layer (Optional)

Untuk aplikasi kompleks, tambahkan **Service Layer**:

```javascript
// services/userService.js
const User = require("../models/User");
const bcrypt = require("bcrypt");

class UserService {
  static async registerUser(userData) {
    // Hash password
    const hashedPassword = await bcrypt.hash(userData.password, 10);

    // Create user
    const user = await User.create({
      ...userData,
      password: hashedPassword,
    });

    return user;
  }

  static async authenticateUser(email, password) {
    const user = await User.findByEmail(email);
    if (!user) {
      throw new Error("User not found");
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      throw new Error("Invalid password");
    }

    return user;
  }
}

module.exports = UserService;
```

---

## Controller dengan Service Layer

```javascript
// controllers/userController.js
const UserService = require("../services/userService");

class UserController {
  static async register(req, res) {
    try {
      const user = await UserService.registerUser(req.body);
      res.status(201).json({
        message: "Registration successful",
        user,
      });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }

  static async login(req, res) {
    try {
      const { email, password } = req.body;
      const user = await UserService.authenticateUser(email, password);
      // Generate token...
      res.json({ message: "Login successful", user });
    } catch (error) {
      res.status(401).json({ error: error.message });
    }
  }
}
```

---

## Struktur Lengkap dengan Service Layer

```
project/
├── models/          (Database schema & queries)
├── services/        (Business logic)
├── controllers/     (Request handling)
├── routes/          (URL routing)
├── middleware/      (Auth, validation, etc)
├── views/           (Templates)
├── config/          (Configuration)
└── utils/           (Helper functions)
```

**Flow:** Routes → Controller → Service → Model

---

## Validation dalam MVC

```javascript
// middleware/validation.js
const { body, validationResult } = require("express-validator");

exports.validateUser = [
  body("name")
    .trim()
    .notEmpty()
    .withMessage("Name is required")
    .isLength({ min: 3 })
    .withMessage("Min 3 characters"),

  body("email").isEmail().withMessage("Invalid email format").normalizeEmail(),

  body("password").isLength({ min: 6 }).withMessage("Min 6 characters"),

  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  },
];
```

---

## Menggunakan Validation di Routes

```javascript
// routes/userRoutes.js
const { validateUser } = require("../middleware/validation");

router.post("/users", validateUser, UserController.create);
```

Validation berjalan sebelum Controller

---

## Best Practices MVC

1. **Keep Controllers Thin** - logika bisnis di Service/Model
2. **Single Responsibility** - satu function satu tugas
3. **Don't Repeat Yourself (DRY)** - reuse code
4. **Use Middleware** untuk validasi, auth, logging
5. **Error Handling** yang konsisten
6. **Naming Convention** yang jelas
7. **Environment Variables** untuk config sensitif

---

## Contoh Error Handling Terpusat

```javascript
// middleware/errorHandler.js
module.exports = (err, req, res, next) => {
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
};

// Di app.js
app.use(errorHandler);
```

---

## Testing dalam MVC

**Unit Test** untuk Model:

```javascript
// tests/models/User.test.js
const User = require("../../models/User");

describe("User Model", () => {
  test("should create a new user", async () => {
    const userData = {
      name: "John",
      email: "john@example.com",
    };
    const user = await User.create(userData);
    expect(user.name).toBe("John");
  });
});
```

**Integration Test** untuk Controller:

```javascript
const request = require("supertest");
const app = require("../../app");

test("GET /users should return users", async () => {
  const response = await request(app).get("/users");
  expect(response.statusCode).toBe(200);
});
```

---

## Environment Variables

```javascript
// config/config.js
require("dotenv").config();

module.exports = {
  port: process.env.PORT || 3000,
  database: {
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
  },
  jwtSecret: process.env.JWT_SECRET,
};
```

```env
# .env
PORT=3000
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=secret
DB_NAME=myapp
JWT_SECRET=your-secret-key
```

---

## MVC vs Other Patterns

**MVC** - Traditional web apps dengan server-side rendering

**MVVM** - Frontend frameworks (Vue, Angular)

**MVP** - Mobile development

**REST API** - Controller + Model (no View)

**Microservices** - Distributed MVC

---

## Tips Implementasi

1. **Start Small** - mulai dengan struktur sederhana
2. **Refactor Gradually** - pindahkan logika ke layer yang tepat
3. **Document Your Code** - beri komentar yang jelas
4. **Version Control** - gunakan Git dengan baik
5. **Code Review** - review struktur dan pattern
6. **Follow Standards** - ikuti convention tim/komunitas

---

## Kesalahan Umum

❌ Terlalu banyak logika di Controller
❌ Model yang tahu tentang HTTP request/response
❌ View yang ngakses Model langsung
❌ Tidak menggunakan environment variables
❌ Tidak ada error handling
❌ Struktur folder yang tidak konsisten
❌ Mixing concerns (logika bisnis di route)

---

## Resources & Referensi

- [Express.js Best Practices](https://expressjs.com/en/advanced/best-practice-performance.html)
- [Node.js Design Patterns](https://www.nodejsdesignpatterns.com/)
- [Clean Architecture in Node.js](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [REST API Design Guide](https://restfulapi.net/)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Happy Coding! 🚀
