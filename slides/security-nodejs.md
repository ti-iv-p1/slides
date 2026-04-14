---
title: Security di Node.js dan Express.js
version: 1.0.0
header: Security di Node.js dan Express.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Security di Node.js dan Express.js

---

## Tujuan Pembelajaran

- Memahami konsep dasar web security
- Menguasai authentication dan password hashing
- Mencegah common vulnerabilities (XSS, CSRF, SQL Injection)
- Mengamankan session dan cookies
- Menggunakan Helmet.js untuk security headers
- Menerapkan input validation dan sanitization
- Mengimplementasikan rate limiting
- Menerapkan best practices security

---

## Mengapa Security Penting?

**Konsekuensi keamanan yang buruk:**

- 💰 Kehilangan data customer
- 🔓 Account takeover
- 💸 Kerugian finansial
- 📉 Kehilangan kepercayaan user
- ⚖️ Masalah legal dan compliance
- 🗞️ Reputasi rusak

**Lebih baik mencegah daripada mengobati!**

---

## OWASP Top 10 Web Vulnerabilities

1. **Broken Access Control**
2. **Cryptographic Failures**
3. **Injection** (SQL, XSS)
4. **Insecure Design**
5. **Security Misconfiguration**
6. **Vulnerable Components**
7. **Authentication Failures**
8. **Software and Data Integrity Failures**
9. **Security Logging Failures**
10. **Server-Side Request Forgery (SSRF)**

Kita akan fokus pada yang paling relevan untuk Node.js web apps

---

## Environment Variables

**JANGAN hardcode secrets di code!**

❌ **BAD:**

```javascript
const dbPassword = "mypassword123";
const sessionSecret = "secret-key";
const apiKey = "sk_live_1234567890";
```

✅ **GOOD:**

```javascript
// .env file
DATABASE_PASSWORD = mypassword123;
SESSION_SECRET = randomly_generated_secure_string;
API_KEY = sk_live_1234567890;
NODE_ENV = production;
```

```javascript
// app.js
require("dotenv").config();

const dbPassword = process.env.DATABASE_PASSWORD;
const sessionSecret = process.env.SESSION_SECRET;
```

**JANGAN commit .env ke git!** (tambahkan ke .gitignore)

---

## Password Hashing

**JANGAN simpan password dalam plaintext!**

❌ **BAD:**

```javascript
// Menyimpan password langsung - SANGAT BERBAHAYA!
const user = {
  email: "user@example.com",
  password: "mypassword123",
};
```

✅ **GOOD: Gunakan bcrypt**

```bash
npm install bcrypt
```

```javascript
const bcrypt = require("bcrypt");

// Hash password sebelum save ke database
const saltRounds = 10;
const hashedPassword = await bcrypt.hash(password, saltRounds);

const user = {
  email: "user@example.com",
  password: hashedPassword, // $2b$10$N9qo8uLOickgx2...
};
```

---

## Registration dengan Password Hashing

```javascript
const bcrypt = require("bcrypt");

app.post("/register", async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Validate input
    if (!email || !password || password.length < 8) {
      return res.render("register", {
        error: "Password must be at least 8 characters",
      });
    }

    // Hash password
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Save to database
    await User.create({
      name,
      email,
      password: hashedPassword,
    });

    res.redirect("/login");
  } catch (error) {
    res.status(500).render("errors/500");
  }
});
```

---

## Login dengan Password Verification

```javascript
const bcrypt = require("bcrypt");

app.post("/login", async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findOne({ where: { email } });

    if (!user) {
      return res.render("login", { error: "Invalid credentials" });
    }

    // Compare password dengan hash
    const isValidPassword = await bcrypt.compare(password, user.password);

    if (!isValidPassword) {
      return res.render("login", { error: "Invalid credentials" });
    }

    // Create session
    req.session.userId = user.id;
    req.session.user = { id: user.id, name: user.name, email: user.email };

    res.redirect("/dashboard");
  } catch (error) {
    res.status(500).render("errors/500");
  }
});
```

---

## Session Security

**Gunakan express-session dengan proper configuration**

```javascript
const session = require("express-session");

app.use(
  session({
    secret: process.env.SESSION_SECRET, // Gunakan secret yang kuat
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: process.env.NODE_ENV === "production", // HTTPS only di production
      httpOnly: true, // Prevent JavaScript access
      maxAge: 1000 * 60 * 60 * 24, // 24 hours
      sameSite: "strict", // CSRF protection
    },
    name: "sessionId", // Jangan gunakan default 'connect.sid'
  }),
);
```

**Key Points:**

- `httpOnly: true` - mencegah XSS attacks
- `secure: true` - hanya kirim via HTTPS
- `sameSite: 'strict'` - mencegah CSRF attacks
- Gunakan secret yang kuat dan random

---

## Authentication Middleware

```javascript
// middleware/auth.js
function isAuthenticated(req, res, next) {
  if (req.session && req.session.userId) {
    return next();
  }
  res.redirect("/login");
}

function isGuest(req, res, next) {
  if (req.session && req.session.userId) {
    return res.redirect("/dashboard");
  }
  next();
}

module.exports = { isAuthenticated, isGuest };
```

```javascript
const { isAuthenticated, isGuest } = require("./middleware/auth");

// Protected route
app.get("/dashboard", isAuthenticated, (req, res) => {
  res.render("dashboard", { user: req.session.user });
});

// Guest only route
app.get("/login", isGuest, (req, res) => {
  res.render("login");
});
```

---

## Input Validation

**SELALU validasi input dari user!**

```bash
npm install express-validator
```

```javascript
const { body, validationResult } = require("express-validator");

app.post(
  "/register",
  // Validation rules
  [
    body("name").trim().notEmpty().withMessage("Name is required"),
    body("email").isEmail().normalizeEmail().withMessage("Invalid email"),
    body("password")
      .isLength({ min: 8 })
      .withMessage("Password must be at least 8 characters")
      .matches(/\d/)
      .withMessage("Password must contain a number"),
  ],

  async (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.render("register", {
        errors: errors.array(),
        formData: req.body,
      });
    }

    // Proceed with registration
  },
);
```

---

## SQL Injection Prevention

**JANGAN concat user input ke SQL query!**

❌ **VULNERABLE:**

```javascript
// SANGAT BERBAHAYA - SQL Injection vulnerability!
app.get("/users/:id", (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
  db.query(query); // Attacker bisa inject: /users/1 OR 1=1
});
```

✅ **SAFE: Gunakan Parameterized Queries**

```javascript
// Safe - menggunakan parameterized query
app.get("/users/:id", async (req, res) => {
  const query = "SELECT * FROM users WHERE id = ?";
  const [users] = await db.query(query, [req.params.id]);
  res.render("users/show", { user: users[0] });
});

// Atau gunakan ORM seperti Sequelize
const user = await User.findByPk(req.params.id);
```

**ORM seperti Sequelize otomatis melindungi dari SQL Injection**

---

## XSS (Cross-Site Scripting) Prevention

**XSS:** Attacker inject malicious script ke website

❌ **VULNERABLE:**

```javascript
// BERBAHAYA - tidak escape HTML
app.get("/search", (req, res) => {
  const query = req.query.q;
  res.send(`<h1>Results for: ${query}</h1>`);
  // Attacker: /search?q=<script>alert('XSS')</script>
});
```

✅ **SAFE: Gunakan Template Engine**

```javascript
// EJS otomatis escape HTML dengan <%= %>
app.get("/search", (req, res) => {
  const query = req.query.q;
  res.render("search", { query });
});
```

```html
<!-- views/search.ejs - SAFE, auto-escaped -->
<h1>Results for: <%= query %></h1>

<!-- UNSAFE - raw HTML (hindari!) -->
<h1>Results for: <%- query %></h1>
```

---

## Content Security Policy (CSP)

**CSP** mencegah XSS dengan membatasi sumber script yang allowed

```javascript
const helmet = require("helmet");

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "trusted-cdn.com"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "images.example.com"],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  }),
);
```

**Ini akan block semua inline scripts kecuali dari sumber yang diizinkan**

---

## CSRF (Cross-Site Request Forgery) Prevention

**CSRF:** Attacker trick user untuk submit request tanpa sepengetahuan mereka

```bash
npm install csurf
```

```javascript
const csrf = require("csurf");
const csrfProtection = csrf({ cookie: true });

// Parse cookies
app.use(require("cookie-parser")());

// Apply CSRF protection ke semua routes
app.use(csrfProtection);

// Pass CSRF token ke semua views
app.use((req, res, next) => {
  res.locals.csrfToken = req.csrfToken();
  next();
});
```

**Dalam form:**

```html
<form method="POST" action="/users">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>" />
  <input type="text" name="name" />
  <button type="submit">Submit</button>
</form>
```

---

## Helmet.js - Security Headers

**Helmet** sets various HTTP headers untuk melindungi app

```bash
npm install helmet
```

```javascript
const helmet = require("helmet");

// Apply default helmet protections
app.use(helmet());

// Atau customize
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
      },
    },
    hsts: {
      maxAge: 31536000, // 1 year
      includeSubDomains: true,
    },
  }),
);
```

**Headers yang diset:**

- X-Content-Type-Options
- X-Frame-Options
- X-XSS-Protection
- Strict-Transport-Security
- Content-Security-Policy

---

## Rate Limiting

**Mencegah brute force attacks**

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require("express-rate-limit");

// General rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: "Too many requests, please try again later.",
});

app.use(limiter);

// Stricter limiter untuk login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: "Too many login attempts, please try again later.",
});

app.post("/login", loginLimiter, async (req, res) => {
  // Login logic
});
```

---

## Sanitization

**Bersihkan input dari user**

```javascript
const { body } = require("express-validator");

app.post(
  "/users",
  [
    body("name").trim().escape(), // Remove HTML tags
    body("email").trim().normalizeEmail(),
    body("bio").trim().escape(),
  ],
  (req, res) => {
    // Input sudah di-sanitize
  },
);
```

**Atau gunakan DOMPurify untuk HTML content:**

```bash
npm install isomorphic-dompurify
```

```javascript
const createDOMPurify = require("isomorphic-dompurify");
const DOMPurify = createDOMPurify();

const cleanHTML = DOMPurify.sanitize(dirtyHTML);
```

---

## File Upload Security

```bash
npm install multer
```

```javascript
const multer = require("multer");
const path = require("path");

// Whitelist allowed file types
const allowedTypes = ["image/jpeg", "image/png", "image/gif"];
const maxSize = 5 * 1024 * 1024; // 5MB

const storage = multer.diskStorage({
  destination: "./public/uploads/",
  filename: (req, file, cb) => {
    // Generate unique filename
    const uniqueName = Date.now() + "-" + Math.round(Math.random() * 1e9);
    const ext = path.extname(file.originalname);
    cb(null, uniqueName + ext);
  },
});

const upload = multer({
  storage: storage,
  limits: { fileSize: maxSize },
  fileFilter: (req, file, cb) => {
    if (!allowedTypes.includes(file.mimetype)) {
      return cb(new Error("Invalid file type"), false);
    }
    cb(null, true);
  },
});
```

---

## File Upload Route

```javascript
app.post("/upload", upload.single("avatar"), (req, res) => {
  if (!req.file) {
    return res.render("upload", { error: "No file uploaded" });
  }

  // Additional validation
  const allowedExtensions = [".jpg", ".jpeg", ".png", ".gif"];
  const fileExt = path.extname(req.file.originalname).toLowerCase();

  if (!allowedExtensions.includes(fileExt)) {
    // Delete uploaded file
    fs.unlinkSync(req.file.path);
    return res.render("upload", { error: "Invalid file extension" });
  }

  // Save file path to database
  const filePath = "/uploads/" + req.file.filename;

  res.redirect("/profile");
});
```

**Key Points:**

- Validate file type (MIME type dan extension)
- Limit file size
- Generate unique filenames
- Store outside web root jika memungkinkan

---

## Secure Cookies

```javascript
const cookieParser = require("cookie-parser");

app.use(cookieParser());

// Set secure cookie
res.cookie("token", "value", {
  httpOnly: true, // Prevent JavaScript access
  secure: process.env.NODE_ENV === "production", // HTTPS only
  sameSite: "strict", // CSRF protection
  maxAge: 3600000, // 1 hour
  signed: true, // Sign cookie
});

// Read signed cookie
const token = req.signedCookies.token;
```

**Cookie options:**

- `httpOnly` - mencegah XSS
- `secure` - HTTPS only
- `sameSite` - mencegah CSRF
- `signed` - detect tampering

---

## Preventing Timing Attacks

**Timing attacks:** Attacker measure response time untuk get information

❌ **VULNERABLE:**

```javascript
// Attacker bisa detect apakah email exists based on response time
app.post("/login", async (req, res) => {
  const user = await User.findOne({ where: { email: req.body.email } });

  if (!user) {
    return res.render("login", { error: "Invalid email" }); // Fast response
  }

  const isValid = await bcrypt.compare(req.body.password, user.password);
  if (!isValid) {
    return res.render("login", { error: "Invalid password" }); // Slow response
  }
});
```

✅ **BETTER:**

```javascript
app.post("/login", async (req, res) => {
  const user = await User.findOne({ where: { email: req.body.email } });

  // Always compare password, even if user not found
  const userPassword = user ? user.password : "$2b$10$invalidhash";
  const isValid = await bcrypt.compare(req.body.password, userPassword);

  if (!user || !isValid) {
    return res.render("login", { error: "Invalid credentials" }); // Same message
  }
});
```

---

## NoSQL Injection Prevention

**Relevant jika menggunakan MongoDB**

❌ **VULNERABLE:**

```javascript
// BERBAHAYA dengan MongoDB
app.post("/login", async (req, res) => {
  const user = await User.findOne({
    email: req.body.email,
    password: req.body.password,
  });
  // Attacker: { "email": { "$gt": "" }, "password": { "$gt": "" } }
});
```

✅ **SAFE:**

```javascript
// Validate dan sanitize input
const { body, validationResult } = require("express-validator");

app.post(
  "/login",
  [body("email").isEmail(), body("password").isString()],
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.render("login", { error: "Invalid input" });
    }

    const user = await User.findOne({ email: req.body.email });
    // Compare password dengan bcrypt
  },
);
```

---

## Dependency Security

**Check for vulnerable dependencies**

```bash
# Check for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix

# Force fix (may have breaking changes)
npm audit fix --force
```

**Keep dependencies up to date:**

```bash
# Check outdated packages
npm outdated

# Update packages
npm update

# Update to latest (careful!)
npm install package@latest
```

**Use package-lock.json** untuk ensure consistent versions

---

## Security Headers Summary

```javascript
const helmet = require("helmet");

app.use(helmet());

// Which sets these headers:
// X-DNS-Prefetch-Control: off
// X-Frame-Options: SAMEORIGIN
// Strict-Transport-Security: max-age=15552000; includeSubDomains
// X-Download-Options: noopen
// X-Content-Type-Options: nosniff
// X-XSS-Protection: 0
```

**Manually set additional headers:**

```javascript
app.use((req, res, next) => {
  res.setHeader("X-Powered-By", ""); // Hide Express
  res.setHeader("X-Content-Type-Options", "nosniff");
  res.setHeader("X-Frame-Options", "DENY");
  res.setHeader("X-XSS-Protection", "1; mode=block");
  next();
});
```

---

## HTTPS in Production

**SELALU gunakan HTTPS di production!**

```javascript
const https = require("https");
const fs = require("fs");

const options = {
  key: fs.readFileSync("path/to/private-key.pem"),
  cert: fs.readFileSync("path/to/certificate.pem"),
};

https.createServer(options, app).listen(443, () => {
  console.log("HTTPS Server running on port 443");
});

// Redirect HTTP to HTTPS
const http = require("http");
http
  .createServer((req, res) => {
    res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
    res.end();
  })
  .listen(80);
```

**Di production, biasanya gunakan reverse proxy (Nginx, Apache) untuk handle SSL**

---

## Error Handling - Don't Expose Details

❌ **BAD:**

```javascript
app.use((err, req, res, next) => {
  // JANGAN expose error details di production!
  res.status(500).json({
    error: err.message,
    stack: err.stack, // Dangerous!
  });
});
```

✅ **GOOD:**

```javascript
app.use((err, req, res, next) => {
  // Log error secara internal
  console.error(err);

  // Send generic error ke user
  res.status(500).render("errors/500", {
    error:
      process.env.NODE_ENV === "development"
        ? err.message
        : "Internal Server Error",
  });
});
```

**Di production, jangan expose:**

- Stack traces
- Database errors
- File paths
- Internal system information

---

## Logout Properly

```javascript
app.post("/logout", (req, res) => {
  // Destroy session
  req.session.destroy((err) => {
    if (err) {
      console.error("Logout error:", err);
    }

    // Clear session cookie
    res.clearCookie("sessionId");

    // Redirect to home
    res.redirect("/");
  });
});
```

**Form logout (bukan link untuk prevent CSRF):**

```html
<form method="POST" action="/logout">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>" />
  <button type="submit">Logout</button>
</form>
```

---

## Security Checklist

✅ **Authentication & Sessions:**

- [ ] Hash passwords dengan bcrypt
- [ ] Secure session configuration (httpOnly, secure, sameSite)
- [ ] Implement proper logout
- [ ] Rate limit login attempts

✅ **Input Validation:**

- [ ] Validate semua user input
- [ ] Sanitize input
- [ ] Use parameterized queries (prevent SQL injection)
- [ ] Validate file uploads

✅ **Headers & HTTPS:**

- [ ] Use Helmet.js
- [ ] Enable HTTPS
- [ ] Set Content Security Policy
- [ ] CSRF protection

---

## Security Checklist (lanjutan)

✅ **Data Protection:**

- [ ] Don't expose sensitive data di error messages
- [ ] Use environment variables untuk secrets
- [ ] Don't commit .env files
- [ ] Encrypt sensitive data at rest

✅ **Dependencies:**

- [ ] Regularly run `npm audit`
- [ ] Keep dependencies updated
- [ ] Review dependency licenses
- [ ] Remove unused dependencies

✅ **Monitoring:**

- [ ] Log security events
- [ ] Monitor for suspicious activity
- [ ] Set up alerts untuk failed login attempts
- [ ] Regular security audits

---

## Example: Secure Express App Setup

```javascript
// app.js
const express = require("express");
const helmet = require("helmet");
const session = require("express-session");
const csrf = require("csurf");
const rateLimit = require("express-rate-limit");
require("dotenv").config();

const app = express();

// Security middleware
app.use(helmet());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
});
app.use(limiter);

// Body parsing
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
```

---

## Example: Secure Express App Setup (lanjutan)

```javascript
// Session management
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "strict",
      maxAge: 1000 * 60 * 60 * 24,
    },
  }),
);

// CSRF protection
app.use(csrf({ cookie: true }));
app.use((req, res, next) => {
  res.locals.csrfToken = req.csrfToken();
  next();
});

// View engine
app.set("view engine", "ejs");

// Routes
app.use("/", require("./routes"));

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).render("errors/500");
});

app.listen(3000);
```

---

## Password Reset Security

```javascript
const crypto = require("crypto");

// Generate secure reset token
app.post("/forgot-password", async (req, res) => {
  const user = await User.findOne({ where: { email: req.body.email } });

  if (!user) {
    // Don't reveal if email exists
    return res.render("forgot-password", {
      message: "If email exists, reset link has been sent",
    });
  }

  // Generate secure random token
  const resetToken = crypto.randomBytes(32).toString("hex");
  const resetTokenExpiry = Date.now() + 3600000; // 1 hour

  // Hash token before saving
  const hashedToken = crypto
    .createHash("sha256")
    .update(resetToken)
    .digest("hex");

  await user.update({
    resetPasswordToken: hashedToken,
    resetPasswordExpires: resetTokenExpiry,
  });

  // Send email with resetToken (not hashedToken)
  // sendResetEmail(user.email, resetToken);

  res.render("forgot-password", { message: "Reset link sent" });
});
```

---

## Password Reset Verification

```javascript
app.post("/reset-password/:token", async (req, res) => {
  // Hash the token from URL
  const hashedToken = crypto
    .createHash("sha256")
    .update(req.params.token)
    .digest("hex");

  // Find user with valid token
  const user = await User.findOne({
    where: {
      resetPasswordToken: hashedToken,
      resetPasswordExpires: { [Op.gt]: Date.now() },
    },
  });

  if (!user) {
    return res.render("reset-password", {
      error: "Invalid or expired token",
    });
  }

  // Update password
  const hashedPassword = await bcrypt.hash(req.body.password, 10);
  await user.update({
    password: hashedPassword,
    resetPasswordToken: null,
    resetPasswordExpires: null,
  });

  res.redirect("/login");
});
```

---

## Two-Factor Authentication (2FA)

```bash
npm install speakeasy qrcode
```

```javascript
const speakeasy = require("speakeasy");
const QRCode = require("qrcode");

// Generate secret untuk user
app.get("/setup-2fa", isAuthenticated, (req, res) => {
  const secret = speakeasy.generateSecret({
    name: `MyApp (${req.session.user.email})`,
  });

  // Generate QR code
  QRCode.toDataURL(secret.otpauth_url, (err, dataUrl) => {
    res.render("setup-2fa", {
      qrCode: dataUrl,
      secret: secret.base32,
    });
  });
});

// Verify 2FA token
app.post("/verify-2fa", isAuthenticated, async (req, res) => {
  const verified = speakeasy.totp.verify({
    secret: user.twoFactorSecret,
    encoding: "base32",
    token: req.body.token,
  });

  if (verified) {
    req.session.twoFactorVerified = true;
    res.redirect("/dashboard");
  } else {
    res.render("verify-2fa", { error: "Invalid code" });
  }
});
```

---

## Secure File Serving

```javascript
const path = require("path");
const fs = require("fs");

// JANGAN biarkan user access semua files!
app.get("/download/:filename", isAuthenticated, async (req, res) => {
  const filename = req.params.filename;

  // Validate filename - prevent directory traversal
  if (
    filename.includes("..") ||
    filename.includes("/") ||
    filename.includes("\\")
  ) {
    return res.status(400).send("Invalid filename");
  }

  // Check if file belongs to user
  const file = await File.findOne({
    where: {
      filename: filename,
      userId: req.session.userId,
    },
  });

  if (!file) {
    return res.status(404).send("File not found");
  }

  const filePath = path.join(__dirname, "uploads", filename);

  // Check if file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).send("File not found");
  }

  res.download(filePath);
});
```

---

## Security Best Practices Summary

1. **Never trust user input** - always validate and sanitize
2. **Use HTTPS** - especially in production
3. **Hash passwords** - use bcrypt, never plaintext
4. **Secure sessions** - httpOnly, secure, sameSite
5. **Prevent injections** - use parameterized queries/ORM
6. **Set security headers** - use Helmet.js
7. **Implement CSRF protection** - untuk form submissions
8. **Rate limiting** - prevent brute force
9. **Keep dependencies updated** - run npm audit regularly
10. **Don't expose secrets** - use environment variables
11. **Proper error handling** - don't leak internal details
12. **Logging** - log security events untuk monitoring

---

## Common Mistakes to Avoid

❌ Hardcoding secrets dalam code
❌ Menyimpan passwords dalam plaintext
❌ Concatenating user input ke SQL queries
❌ Tidak validasi file uploads
❌ Exposing stack traces di production
❌ Menggunakan session defaults tanpa configuration
❌ Tidak implement rate limiting
❌ Tidak menggunakan HTTPS
❌ Commit .env files ke git
❌ Tidak update dependencies
❌ Tidak sanitize HTML output
❌ Menggunakan GET untuk actions yang modify data

---

## Security Testing Tools

**Development:**

```bash
# Check dependencies
npm audit

# Scan for secrets in code
npm install -g trufflehog

# Security linting
npm install -g eslint-plugin-security
```

**Online Tools:**

- **OWASP ZAP** - vulnerability scanner
- **Burp Suite** - security testing
- **Snyk** - dependency scanning
- **SonarQube** - code quality & security

**Manual Testing:**

- Test with malicious input
- Try SQL injection
- Test XSS vulnerabilities
- Try accessing unauthorized resources

---

## Resources & Referensi

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [Express Security Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)
- [Helmet.js Documentation](https://helmetjs.github.io/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [npm audit](https://docs.npmjs.com/cli/v8/commands/npm-audit)
- [Snyk Vulnerability Database](https://snyk.io/vuln/)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Stay Secure! 🔒
