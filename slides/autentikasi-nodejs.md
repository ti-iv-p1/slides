---
title: Autentikasi di Node.js
version: 1.0.0
header: Autentikasi di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Autentikasi di Node.js

---

## Tujuan Pembelajaran

- Memahami konsep autentikasi dan otorisasi
- Menguasai session-based authentication
- Mengimplementasikan JWT (JSON Web Token)
- Melakukan password hashing dengan bcrypt
- Menerapkan best practices keamanan
- Memahami OAuth dan social login

---

## Autentikasi vs Otorisasi

**Authentication (Autentikasi)**

- Proses memverifikasi **siapa** user
- Contoh: Login dengan username & password
- Pertanyaan: "Apakah kamu benar-benar user ini?"

**Authorization (Otorisasi)**

- Proses memeriksa **apa yang boleh** dilakukan user
- Contoh: Admin vs Regular User
- Pertanyaan: "Apakah kamu punya akses ke resource ini?"

---

## Jenis Autentikasi

1. **Session-based Authentication**
   - Server menyimpan session data
   - Client menyimpan session ID di cookie
   - Stateful (server harus ingat session)

2. **Token-based Authentication (JWT)**
   - Server generate token
   - Client simpan token (localStorage/cookie)
   - Stateless (server tidak simpan state)

---

## Password Hashing

**JANGAN PERNAH** simpan password plain text!

```javascript
// ❌ SALAH - Plain text
const user = {
  email: "user@example.com",
  password: "password123", // Bahaya!
};

// ✅ BENAR - Hashed password
const user = {
  email: "user@example.com",
  password: "$2b$10$...", // Hash dari bcrypt
};
```

---

## Bcrypt untuk Hashing

```javascript
const bcrypt = require("bcrypt");

// Hash password saat register
async function registerUser(email, password) {
  const saltRounds = 10;
  const hashedPassword = await bcrypt.hash(password, saltRounds);

  await User.create({
    email: email,
    password: hashedPassword,
  });
}

// Verify password saat login
async function loginUser(email, password) {
  const user = await User.findByEmail(email);
  const isMatch = await bcrypt.compare(password, user.password);

  if (isMatch) {
    return user;
  }
  throw new Error("Invalid credentials");
}
```

---

## Session-based Authentication

**Flow:**

1. User login dengan credentials
2. Server verify & create session
3. Session ID dikirim ke client via cookie
4. Client send cookie setiap request
5. Server verify session ID

---

## Setup Express Session

```javascript
const express = require("express");
const session = require("express-session");

const app = express();

app.use(
  session({
    secret: "your-secret-key", // Ganti dengan env variable
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: false, // true untuk HTTPS
      httpOnly: true,
      maxAge: 24 * 60 * 60 * 1000, // 24 hours
    },
  }),
);
```

---

## Login dengan Session

```javascript
app.post("/login", async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findByEmail(email);
    if (!user) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // Verify password
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // Create session
    req.session.userId = user.id;
    req.session.email = user.email;

    res.json({ message: "Login successful", user });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## Logout dengan Session

```javascript
app.post("/logout", (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: "Logout failed" });
    }
    res.clearCookie("connect.sid"); // Default session cookie name
    res.json({ message: "Logout successful" });
  });
});
```

---

## Authentication Middleware (Session)

```javascript
// middleware/auth.js
const authenticate = (req, res, next) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: "Unauthorized" });
  }
  next();
};

// Gunakan di route
app.get("/profile", authenticate, async (req, res) => {
  const user = await User.findById(req.session.userId);
  res.json({ user });
});

app.put("/profile", authenticate, async (req, res) => {
  const userId = req.session.userId;
  await User.update(userId, req.body);
  res.json({ message: "Profile updated" });
});
```

---

## JWT (JSON Web Token)

**JWT** adalah token yang berisi informasi dalam format JSON

Struktur JWT: `header.payload.signature`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VySWQiOjEyMywiZW1haWwiOiJ1c2VyQGV4YW1wbGUuY29tIn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Header** - algorithm & token type
**Payload** - data/claims
**Signature** - verify token belum diubah

---

## Install JWT

```bash
npm install jsonwebtoken
```

```javascript
const jwt = require("jsonwebtoken");

// Secret key - simpan di environment variable
const JWT_SECRET = process.env.JWT_SECRET || "your-secret-key";
```

---

## Generate JWT Token

```javascript
const jwt = require("jsonwebtoken");

function generateToken(user) {
  const payload = {
    userId: user.id,
    email: user.email,
  };

  const options = {
    expiresIn: "24h", // Token expire dalam 24 jam
  };

  return jwt.sign(payload, JWT_SECRET, options);
}

// Penggunaan
const token = generateToken(user);
// eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Login dengan JWT

```javascript
app.post("/login", async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findByEmail(email);
    if (!user) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // Verify password
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // Generate token
    const token = generateToken(user);

    res.json({
      message: "Login successful",
      token: token,
      user: { id: user.id, email: user.email },
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## Verify JWT Token

```javascript
function verifyToken(token) {
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    return decoded;
  } catch (error) {
    throw new Error("Invalid token");
  }
}

// Penggunaan
const decoded = verifyToken(token);
console.log(decoded.userId); // 123
console.log(decoded.email); // user@example.com
```

---

## Authentication Middleware (JWT)

```javascript
// middleware/auth.js
const jwt = require("jsonwebtoken");

const authenticate = (req, res, next) => {
  try {
    // Get token from header
    const authHeader = req.headers.authorization;
    if (!authHeader) {
      return res.status(401).json({ error: "No token provided" });
    }

    // Format: "Bearer <token>"
    const token = authHeader.split(" ")[1];

    // Verify token
    const decoded = jwt.verify(token, JWT_SECRET);

    // Attach user data ke request
    req.userId = decoded.userId;
    req.email = decoded.email;

    next();
  } catch (error) {
    res.status(401).json({ error: "Invalid token" });
  }
};

module.exports = { authenticate };
```

---

## Menggunakan JWT Middleware

```javascript
const { authenticate } = require("./middleware/auth");

// Protected route
app.get("/profile", authenticate, async (req, res) => {
  const user = await User.findById(req.userId);
  res.json({ user });
});

app.put("/profile", authenticate, async (req, res) => {
  await User.update(req.userId, req.body);
  res.json({ message: "Profile updated" });
});
```

**Client mengirim request:**

```
GET /profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Refresh Token

Token utama (access token) expire cepat, refresh token lebih lama

```javascript
function generateTokens(user) {
  const accessToken = jwt.sign(
    { userId: user.id, email: user.email },
    JWT_SECRET,
    { expiresIn: "15m" }, // 15 minutes
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    REFRESH_TOKEN_SECRET,
    { expiresIn: "7d" }, // 7 days
  );

  return { accessToken, refreshToken };
}
```

---

## Endpoint Refresh Token

```javascript
app.post("/refresh", async (req, res) => {
  try {
    const { refreshToken } = req.body;

    if (!refreshToken) {
      return res.status(401).json({ error: "Refresh token required" });
    }

    // Verify refresh token
    const decoded = jwt.verify(refreshToken, REFRESH_TOKEN_SECRET);

    // Generate new access token
    const user = await User.findById(decoded.userId);
    const newAccessToken = jwt.sign(
      { userId: user.id, email: user.email },
      JWT_SECRET,
      { expiresIn: "15m" },
    );

    res.json({ accessToken: newAccessToken });
  } catch (error) {
    res.status(401).json({ error: "Invalid refresh token" });
  }
});
```

---

## Register User

```javascript
app.post("/register", async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Validasi input
    if (!email || !password) {
      return res.status(400).json({
        error: "Email and password required",
      });
    }

    // Check if user exists
    const existingUser = await User.findByEmail(email);
    if (existingUser) {
      return res.status(409).json({
        error: "Email already registered",
      });
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Create user
    const user = await User.create({
      name,
      email,
      password: hashedPassword,
    });

    res.status(201).json({
      message: "User registered successfully",
      user: { id: user.id, email: user.email },
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## Authorization (Role-based)

```javascript
// Model User dengan role
const user = {
  id: 1,
  email: "admin@example.com",
  role: "admin", // 'admin', 'user', 'moderator'
};

// Middleware untuk check role
const authorize = (...roles) => {
  return async (req, res, next) => {
    const user = await User.findById(req.userId);

    if (!roles.includes(user.role)) {
      return res.status(403).json({
        error: "Forbidden - Insufficient permissions",
      });
    }

    next();
  };
};
```

---

## Menggunakan Authorization

```javascript
// Only admin can access
app.delete(
  "/users/:id",
  authenticate,
  authorize("admin"),
  UserController.delete,
);

// Admin or moderator
app.put(
  "/posts/:id/approve",
  authenticate,
  authorize("admin", "moderator"),
  PostController.approve,
);

// All authenticated users
app.get("/profile", authenticate, UserController.profile);
```

---

## Protect Password di Response

```javascript
// ❌ SALAH - Expose password
const user = await User.findById(userId);
res.json({ user }); // Password terkirim!

// ✅ BENAR - Exclude password
const user = await User.findById(userId);
const { password, ...userWithoutPassword } = user;
res.json({ user: userWithoutPassword });

// ✅ Atau gunakan select di query
const user = await User.findById(userId).select("-password");
res.json({ user });
```

---

## Email Verification

```javascript
const crypto = require("crypto");

// Generate verification token
function generateVerificationToken() {
  return crypto.randomBytes(32).toString("hex");
}

// Register dengan email verification
app.post("/register", async (req, res) => {
  const verificationToken = generateVerificationToken();

  const user = await User.create({
    email: req.body.email,
    password: hashedPassword,
    verificationToken: verificationToken,
    isVerified: false,
  });

  // Send email dengan link verification
  const verificationLink = `${BASE_URL}/verify-email?token=${verificationToken}`;
  await sendEmail(user.email, verificationLink);

  res.status(201).json({
    message: "Check your email for verification",
  });
});
```

---

## Verify Email Endpoint

```javascript
app.get("/verify-email", async (req, res) => {
  try {
    const { token } = req.query;

    const user = await User.findOne({ verificationToken: token });
    if (!user) {
      return res.status(400).json({ error: "Invalid token" });
    }

    // Update user
    await User.update(user.id, {
      isVerified: true,
      verificationToken: null,
    });

    res.json({ message: "Email verified successfully" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## Password Reset

```javascript
app.post("/forgot-password", async (req, res) => {
  try {
    const { email } = req.body;
    const user = await User.findByEmail(email);

    if (!user) {
      // Jangan expose apakah email terdaftar
      return res.json({
        message: "If email exists, reset link sent",
      });
    }

    // Generate reset token
    const resetToken = crypto.randomBytes(32).toString("hex");
    const resetExpires = Date.now() + 3600000; // 1 hour

    await User.update(user.id, {
      resetToken: resetToken,
      resetExpires: resetExpires,
    });

    // Send email
    const resetLink = `${BASE_URL}/reset-password?token=${resetToken}`;
    await sendEmail(user.email, resetLink);

    res.json({ message: "If email exists, reset link sent" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## Reset Password Endpoint

```javascript
app.post("/reset-password", async (req, res) => {
  try {
    const { token, newPassword } = req.body;

    const user = await User.findOne({
      resetToken: token,
      resetExpires: { $gt: Date.now() },
    });

    if (!user) {
      return res.status(400).json({
        error: "Invalid or expired token",
      });
    }

    // Hash new password
    const hashedPassword = await bcrypt.hash(newPassword, 10);

    // Update password & clear reset token
    await User.update(user.id, {
      password: hashedPassword,
      resetToken: null,
      resetExpires: null,
    });

    res.json({ message: "Password reset successful" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## OAuth / Social Login (Google)

```javascript
const { OAuth2Client } = require("google-auth-library");
const client = new OAuth2Client(GOOGLE_CLIENT_ID);

app.post("/auth/google", async (req, res) => {
  try {
    const { token } = req.body;

    // Verify Google token
    const ticket = await client.verifyIdToken({
      idToken: token,
      audience: GOOGLE_CLIENT_ID,
    });

    const payload = ticket.getPayload();
    const email = payload.email;

    // Find or create user
    let user = await User.findByEmail(email);
    if (!user) {
      user = await User.create({
        email: email,
        name: payload.name,
        googleId: payload.sub,
        isVerified: true,
      });
    }

    // Generate JWT
    const jwtToken = generateToken(user);
    res.json({ token: jwtToken, user });
  } catch (error) {
    res.status(401).json({ error: "Invalid Google token" });
  }
});
```

---

## Passport.js

**Passport** adalah authentication middleware yang populer

```bash
npm install passport passport-local passport-jwt
```

```javascript
const passport = require("passport");
const LocalStrategy = require("passport-local").Strategy;

passport.use(
  new LocalStrategy(
    { usernameField: "email" },
    async (email, password, done) => {
      try {
        const user = await User.findByEmail(email);
        if (!user) {
          return done(null, false, { message: "Invalid credentials" });
        }

        const isValid = await bcrypt.compare(password, user.password);
        if (!isValid) {
          return done(null, false, { message: "Invalid credentials" });
        }

        return done(null, user);
      } catch (error) {
        return done(error);
      }
    },
  ),
);
```

---

## Login dengan Passport

```javascript
app.use(passport.initialize());

app.post(
  "/login",
  passport.authenticate("local", { session: false }),
  (req, res) => {
    const token = generateToken(req.user);
    res.json({
      message: "Login successful",
      token: token,
      user: req.user,
    });
  },
);
```

---

## Security Best Practices

1. **HTTPS** - selalu gunakan HTTPS di production
2. **Environment Variables** - simpan secret di `.env`
3. **Rate Limiting** - batasi request untuk prevent brute force
4. **Input Validation** - validasi semua input
5. **SQL Injection Prevention** - gunakan parameterized queries
6. **XSS Prevention** - sanitize user input
7. **CSRF Protection** - gunakan CSRF tokens
8. **Secure Cookies** - `httpOnly`, `secure`, `sameSite`

---

## Rate Limiting untuk Login

```javascript
const rateLimit = require("express-rate-limit");

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: "Too many login attempts, try again later",
  standardHeaders: true,
  legacyHeaders: false,
});

app.post("/login", loginLimiter, async (req, res) => {
  // Login logic...
});
```

---

## Environment Variables

```javascript
// .env
JWT_SECRET=your-super-secret-key-here
JWT_REFRESH_SECRET=another-secret-key
DB_PASSWORD=database-password
GOOGLE_CLIENT_ID=your-google-client-id
SMTP_PASSWORD=email-password
```

```javascript
// Load .env
require("dotenv").config();

// Use
const JWT_SECRET = process.env.JWT_SECRET;
const DB_PASSWORD = process.env.DB_PASSWORD;
```

**JANGAN** commit `.env` ke Git! Tambahkan ke `.gitignore`

---

## Helmet.js untuk Security Headers

```javascript
const helmet = require("helmet");

app.use(helmet());

// Set security headers:
// - X-Content-Type-Options: nosniff
// - X-Frame-Options: DENY
// - X-XSS-Protection: 1; mode=block
// - Strict-Transport-Security
// - dll
```

---

## CORS Configuration

```javascript
const cors = require("cors");

const corsOptions = {
  origin: "https://yourdomain.com", // Specific domain
  credentials: true, // Allow cookies
  optionsSuccessStatus: 200,
};

app.use(cors(corsOptions));

// Untuk development, allow all:
// app.use(cors());
```

---

## Input Validation & Sanitization

```javascript
const { body, validationResult } = require("express-validator");

const validateRegister = [
  body("email").isEmail().withMessage("Invalid email").normalizeEmail(),
  body("password")
    .isLength({ min: 8 })
    .withMessage("Min 8 characters")
    .matches(/\d/)
    .withMessage("Must contain a number"),
  body("name").trim().escape().notEmpty().withMessage("Name required"),
];

app.post("/register", validateRegister, (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Process registration...
});
```

---

## Logging Authentication Events

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  transports: [new winston.transports.File({ filename: "auth.log" })],
});

app.post("/login", async (req, res) => {
  try {
    // Login logic...

    logger.info({
      event: "login_success",
      email: user.email,
      ip: req.ip,
      timestamp: new Date(),
    });

    res.json({ token });
  } catch (error) {
    logger.warn({
      event: "login_failed",
      email: req.body.email,
      ip: req.ip,
      timestamp: new Date(),
    });

    res.status(401).json({ error: "Invalid credentials" });
  }
});
```

---

## Multi-Factor Authentication (MFA)

```javascript
const speakeasy = require("speakeasy");
const qrcode = require("qrcode");

// Generate MFA secret
app.post("/mfa/setup", authenticate, async (req, res) => {
  const secret = speakeasy.generateSecret({
    name: `YourApp (${req.email})`,
  });

  await User.update(req.userId, {
    mfaSecret: secret.base32,
    mfaEnabled: false, // Enable after verification
  });

  // Generate QR code
  const qrCodeUrl = await qrcode.toDataURL(secret.otpauth_url);

  res.json({ qrCode: qrCodeUrl, secret: secret.base32 });
});
```

---

## Verify MFA Token

```javascript
app.post("/mfa/verify", authenticate, async (req, res) => {
  const { token } = req.body;
  const user = await User.findById(req.userId);

  const verified = speakeasy.totp.verify({
    secret: user.mfaSecret,
    encoding: "base32",
    token: token,
  });

  if (verified) {
    await User.update(req.userId, { mfaEnabled: true });
    res.json({ message: "MFA enabled successfully" });
  } else {
    res.status(400).json({ error: "Invalid token" });
  }
});
```

---

## Complete Auth Example Structure

```
project/
├── controllers/
│   └── authController.js
├── middleware/
│   ├── auth.js
│   └── validation.js
├── models/
│   └── User.js
├── routes/
│   └── auth.js
├── services/
│   ├── authService.js
│   └── emailService.js
├── utils/
│   ├── jwt.js
│   └── bcrypt.js
└── .env
```

---

## Testing Authentication

```javascript
const request = require("supertest");
const app = require("../app");

describe("Authentication", () => {
  test("POST /register should create user", async () => {
    const res = await request(app).post("/register").send({
      email: "test@example.com",
      password: "password123",
    });

    expect(res.statusCode).toBe(201);
    expect(res.body).toHaveProperty("message");
  });

  test("POST /login should return token", async () => {
    const res = await request(app).post("/login").send({
      email: "test@example.com",
      password: "password123",
    });

    expect(res.statusCode).toBe(200);
    expect(res.body).toHaveProperty("token");
  });
});
```

---

## Kesalahan Umum

❌ Simpan password plain text
❌ Expose sensitive data di response
❌ Tidak validasi input
❌ Secret key hardcoded di code
❌ Token tidak expire
❌ Tidak gunakan HTTPS
❌ Tidak implement rate limiting
❌ Tidak log authentication events
❌ Allow weak passwords

---

## Session vs JWT Comparison

**Session-based:**
✅ Mudah revoke (logout semua device)
✅ Tidak perlu simpan data di client
❌ Butuh session storage
❌ Sulit untuk distributed systems

**JWT:**
✅ Stateless - scalable
✅ Good for microservices
❌ Sulit revoke token
❌ Token bisa besar jika banyak data

**Pilihan:** Tergantung use case Anda!

---

## Resources & Referensi

- [Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [JWT.io](https://jwt.io/)
- [Passport.js Documentation](http://www.passportjs.org/)
- [bcrypt Documentation](https://www.npmjs.com/package/bcrypt)
- [Express Security Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Stay Secure! 🔒
