---
title: Logging di Node.js
version: 1.0.0
header: Logging di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Logging di Node.js

---

## Tujuan Pembelajaran

- Memahami pentingnya logging dalam aplikasi
- Menguasai console logging dasar
- Memahami log levels (info, warn, error, debug)
- Menggunakan Winston untuk advanced logging
- Menggunakan Morgan untuk HTTP request logging
- Menerapkan best practices logging
- Membedakan logging untuk development vs production

---

## Apa itu Logging?

**Logging** adalah proses mencatat aktivitas dan kejadian yang terjadi dalam aplikasi

**Tujuan Logging:**

- 🐛 **Debugging** - membantu menemukan bug
- 📊 **Monitoring** - memantau performa aplikasi
- 🔍 **Troubleshooting** - membantu analisa masalah
- 📈 **Analytics** - menganalisa user behavior
- 🔒 **Security** - tracking aktivitas mencurigakan
- 📝 **Audit Trail** - record of events

---

## Mengapa Logging Penting?

✅ Membantu debugging di production
✅ Memantau kesehatan aplikasi
✅ Tracking errors dan exceptions
✅ Analisa performa aplikasi
✅ Compliance dan audit requirements
✅ Memahami user behavior

❌ Tanpa logging: sulit menemukan root cause masalah di production

---

## Console Logging Dasar

```javascript
// Basic console methods
console.log("Informational message");
console.info("Info message");
console.warn("Warning message");
console.error("Error message");
console.debug("Debug message");

// Log dengan multiple arguments
console.log("User:", user.name, "logged in at", new Date());

// Log objects
const user = { id: 1, name: "John" };
console.log("User data:", user);

// Pretty print object
console.log("User data:", JSON.stringify(user, null, 2));
```

---

## Console Logging - Advanced

```javascript
// console.table untuk array of objects
const users = [
  { id: 1, name: "John", role: "admin" },
  { id: 2, name: "Jane", role: "user" },
];
console.table(users);

// console.time untuk measure performance
console.time("database-query");
await fetchUsers();
console.timeEnd("database-query");

// console.trace untuk stack trace
function problematicFunction() {
  console.trace("Trace point");
}

// console.group untuk grouping logs
console.group("User Login Process");
console.log("Step 1: Validate credentials");
console.log("Step 2: Create session");
console.groupEnd();
```

---

## Log Levels

**Log levels** menentukan severity/tingkat kepentingan log

```javascript
// Dari paling tidak penting ke paling penting:
// DEBUG → INFO → WARN → ERROR

// DEBUG - detail informasi untuk debugging
console.debug("Variable value:", userInput);

// INFO - informasi umum
console.info("Server started on port 3000");

// WARN - peringatan, tidak critical tapi perlu perhatian
console.warn("Deprecated function used");

// ERROR - error yang perlu immediate attention
console.error("Database connection failed:", error);
```

**Production:** biasanya hanya log WARN dan ERROR
**Development:** log semua levels termasuk DEBUG

---

## Masalah dengan Console Logging

❌ Tidak ada log levels yang proper
❌ Tidak ada timestamps
❌ Tidak ada file output (hanya console)
❌ Tidak bisa filter berdasarkan severity
❌ Tidak bisa customize format
❌ Tidak production-ready

**Solusi:** Gunakan logging library seperti **Winston**

---

## Winston Logger

**Winston** adalah logging library paling populer untuk Node.js

**Instalasi:**

```bash
npm install winston
```

**Features:**

- Multiple transports (console, file, database)
- Log levels
- Custom formats
- Timestamps
- Colored output
- Log rotation

---

## Winston - Basic Setup

```javascript
// logger.js
const winston = require("winston");

const logger = winston.createLogger({
  level: "info",
  format: winston.format.json(),
  transports: [
    // Write all logs to console
    new winston.transports.Console(),
    // Write all logs to combined.log
    new winston.transports.File({ filename: "combined.log" }),
    // Write only errors to error.log
    new winston.transports.File({ filename: "error.log", level: "error" }),
  ],
});

module.exports = logger;
```

---

## Menggunakan Winston Logger

```javascript
// app.js
const logger = require("./logger");

// Different log levels
logger.info("Application started");
logger.warn("Low disk space");
logger.error("Database connection failed", { error: err.message });
logger.debug("User data:", { user });

// di route handler
app.get("/users", async (req, res) => {
  try {
    logger.info("Fetching users list");
    const users = await User.findAll();
    res.render("users/index", { users });
  } catch (error) {
    logger.error("Error fetching users:", {
      error: error.message,
      stack: error.stack,
    });
    res.status(500).render("errors/500");
  }
});
```

---

## Winston - Custom Format

```javascript
const winston = require("winston");

const customFormat = winston.format.combine(
  winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
  winston.format.errors({ stack: true }),
  winston.format.splat(),
  winston.format.printf(({ level, message, timestamp, ...meta }) => {
    return `[${timestamp}] ${level.toUpperCase()}: ${message} ${
      Object.keys(meta).length ? JSON.stringify(meta) : ""
    }`;
  }),
);

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: customFormat,
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "logs/app.log" }),
  ],
});
```

---

## Winston - Colored Console Output

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: "debug",
  format: winston.format.combine(
    winston.format.colorize(),
    winston.format.timestamp({ format: "HH:mm:ss" }),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] ${level}: ${message}`;
    }),
  ),
  transports: [new winston.transports.Console()],
});

// Output dengan warna:
logger.error("This will be red");
logger.warn("This will be yellow");
logger.info("This will be green");
logger.debug("This will be blue");
```

---

## Winston - Development vs Production

```javascript
const winston = require("winston");

const isDevelopment = process.env.NODE_ENV !== "production";

const logger = winston.createLogger({
  level: isDevelopment ? "debug" : "info",
  format: isDevelopment
    ? winston.format.combine(winston.format.colorize(), winston.format.simple())
    : winston.format.json(),
  transports: [
    new winston.transports.Console(),
    // Production: simpan ke file
    ...(!isDevelopment
      ? [
          new winston.transports.File({
            filename: "logs/error.log",
            level: "error",
          }),
          new winston.transports.File({ filename: "logs/combined.log" }),
        ]
      : []),
  ],
});

module.exports = logger;
```

---

## Morgan - HTTP Request Logger

**Morgan** adalah middleware untuk logging HTTP requests

**Instalasi:**

```bash
npm install morgan
```

**Predefined Formats:**

- `combined` - Standard Apache combined log
- `common` - Standard Apache common log
- `dev` - Colored by response status (untuk development)
- `short` - Shorter than default
- `tiny` - Minimal output

---

## Morgan - Basic Usage

```javascript
const express = require("express");
const morgan = require("morgan");
const app = express();

// Development: colored output
if (process.env.NODE_ENV !== "production") {
  app.use(morgan("dev"));
}

// Production: write to file
if (process.env.NODE_ENV === "production") {
  const fs = require("fs");
  const path = require("path");

  const accessLogStream = fs.createWriteStream(
    path.join(__dirname, "logs/access.log"),
    { flags: "a" },
  );

  app.use(morgan("combined", { stream: accessLogStream }));
}

// Routes
app.get("/", (req, res) => {
  res.render("index");
});
```

---

## Morgan Format Examples

```javascript
// dev format (colored)
// GET /users 200 15.123 ms - 1234
app.use(morgan("dev"));

// tiny format
// GET /users 200 1234 - 15.123 ms
app.use(morgan("tiny"));

// combined format (Apache style)
// ::1 - - [14/Apr/2026:10:30:45 +0000] "GET /users HTTP/1.1" 200 1234
app.use(morgan("combined"));

// Custom format
app.use(
  morgan(":method :url :status :response-time ms - :res[content-length]"),
);

// Custom format dengan timestamp
morgan.token("timestamp", () => {
  return new Date().toISOString();
});
app.use(morgan(":timestamp :method :url :status"));
```

---

## Morgan + Winston Integration

```javascript
const express = require("express");
const morgan = require("morgan");
const winston = require("winston");

// Create Winston logger
const logger = winston.createLogger({
  transports: [new winston.transports.File({ filename: "logs/access.log" })],
});

// Create Morgan stream to Winston
const morganStream = {
  write: (message) => {
    logger.info(message.trim());
  },
};

const app = express();

// Use Morgan with Winston stream
app.use(morgan("combined", { stream: morganStream }));

// Regular routes
app.get("/", (req, res) => {
  res.render("index");
});
```

---

## Logging di Controllers

```javascript
// controllers/userController.js
const logger = require("../config/logger");
const User = require("../models/User");

class UserController {
  static async index(req, res) {
    try {
      logger.info("Fetching all users", {
        requestedBy: req.session.userId,
      });

      const users = await User.findAll();

      logger.info(`Found ${users.length} users`);

      res.render("users/index", { users });
    } catch (error) {
      logger.error("Error in UserController.index", {
        error: error.message,
        stack: error.stack,
        requestedBy: req.session.userId,
      });

      res.status(500).render("errors/500");
    }
  }
}

module.exports = UserController;
```

---

## Logging User Actions

```javascript
// Log user login
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  logger.info("Login attempt", { email });

  try {
    const user = await authenticateUser(email, password);

    if (user) {
      req.session.userId = user.id;

      logger.info("Login successful", {
        userId: user.id,
        email: user.email,
        ip: req.ip,
      });

      res.redirect("/dashboard");
    } else {
      logger.warn("Login failed - invalid credentials", {
        email,
        ip: req.ip,
      });

      res.render("login", { error: "Invalid credentials" });
    }
  } catch (error) {
    logger.error("Login error", {
      email,
      error: error.message,
    });

    res.status(500).render("errors/500");
  }
});
```

---

## Error Logging

```javascript
// Global error handler dengan logging
app.use((err, req, res, next) => {
  // Log error dengan detail
  logger.error("Unhandled error", {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    ip: req.ip,
    userAgent: req.get("user-agent"),
    userId: req.session ? req.session.userId : null,
  });

  // Response ke user
  const statusCode = err.statusCode || 500;

  res.status(statusCode).render("errors/500", {
    error:
      process.env.NODE_ENV === "development"
        ? err.message
        : "Internal Server Error",
  });
});
```

---

## Logging Database Queries

```javascript
// Sequelize logging example
const { Sequelize } = require("sequelize");
const logger = require("./logger");

const sequelize = new Sequelize(process.env.DATABASE_URL, {
  logging: (msg) => logger.debug(msg),
  // Atau disable di production
  // logging: process.env.NODE_ENV === 'development' ? console.log : false
});

// Manual query logging
app.get("/users", async (req, res) => {
  try {
    logger.debug("Executing database query: SELECT * FROM users");

    console.time("db-query");
    const users = await User.findAll();
    console.timeEnd("db-query");

    logger.debug(`Query returned ${users.length} results`);

    res.render("users/index", { users });
  } catch (error) {
    logger.error("Database query failed", { error: error.message });
    res.status(500).render("errors/500");
  }
});
```

---

## Log Rotation

**Log rotation** mencegah file log terlalu besar

**Instalasi:**

```bash
npm install winston-daily-rotate-file
```

**Setup:**

```javascript
const winston = require("winston");
require("winston-daily-rotate-file");

const transport = new winston.transports.DailyRotateFile({
  filename: "logs/application-%DATE%.log",
  datePattern: "YYYY-MM-DD",
  maxSize: "20m", // Rotate when file reaches 20MB
  maxFiles: "14d", // Keep logs for 14 days
  format: winston.format.json(),
});

const logger = winston.createLogger({
  transports: [transport],
});
```

**Hasil:** `application-2026-04-14.log`, `application-2026-04-15.log`, dll

---

## Structured Logging

**Structured logging** menggunakan format JSON untuk mudah di-parse

```javascript
const logger = require("./logger");

// ❌ BAD - Hard to parse
logger.info("User John with ID 123 logged in from 192.168.1.1");

// ✅ GOOD - Structured, easy to parse
logger.info("User login", {
  event: "user_login",
  userId: 123,
  username: "John",
  ip: "192.168.1.1",
  timestamp: new Date().toISOString(),
});

// Easy to search/filter:
// - All login events: event === 'user_login'
// - All actions from specific user: userId === 123
// - All actions from specific IP: ip === '192.168.1.1'
```

---

## Logging Best Practices

1. **Use appropriate log levels**

   ```javascript
   logger.debug("Variable value:", data); // Development only
   logger.info("User created"); // Informational
   logger.warn("Deprecated API used"); // Warning
   logger.error("Database error", { error }); // Errors
   ```

2. **Include context**

   ```javascript
   logger.info("Order created", {
     orderId: order.id,
     userId: user.id,
     amount: order.total,
     timestamp: new Date(),
   });
   ```

3. **Don't log sensitive data**
   ```javascript
   // ❌ BAD
   logger.info("Login", { password: user.password });
   // ✅ GOOD
   logger.info("Login", { userId: user.id });
   ```

---

## Logging Best Practices (lanjutan)

4. **Use log rotation** - hindari file log terlalu besar

5. **Different configs for dev/prod**

   ```javascript
   const level = process.env.NODE_ENV === "production" ? "info" : "debug";
   ```

6. **Log errors with stack traces**

   ```javascript
   logger.error("Error occurred", {
     error: err.message,
     stack: err.stack,
   });
   ```

7. **Use correlation IDs** untuk tracking request

   ```javascript
   const requestId = uuid();
   logger.info("Request started", { requestId, url: req.url });
   // ... dalam proses request
   logger.error("Error", { requestId, error });
   ```

---

## What NOT to Log

❌ **Passwords dan credentials**

```javascript
// BAD
logger.info("User data", { password: user.password });
```

❌ **Credit card numbers dan PII (Personal Identifiable Information)**

```javascript
// BAD
logger.info("Payment", { cardNumber: "1234-5678-9012-3456" });
```

❌ **API Keys dan secrets**

```javascript
// BAD
logger.info("API call", { apiKey: process.env.API_KEY });
```

❌ **Session tokens**

```javascript
// BAD
logger.info("Session", { token: req.session.token });
```

---

## Production Logging Setup

```javascript
// config/logger.js
const winston = require("winston");
require("winston-daily-rotate-file");

const isProduction = process.env.NODE_ENV === "production";

// File transport with rotation
const fileTransport = new winston.transports.DailyRotateFile({
  filename: "logs/app-%DATE%.log",
  datePattern: "YYYY-MM-DD",
  maxSize: "20m",
  maxFiles: "30d",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
});

// Error file transport
const errorFileTransport = new winston.transports.DailyRotateFile({
  filename: "logs/error-%DATE%.log",
  datePattern: "YYYY-MM-DD",
  level: "error",
  maxSize: "20m",
  maxFiles: "30d",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
});
```

---

## Production Logging Setup (lanjutan)

```javascript
// Console transport untuk development
const consoleTransport = new winston.transports.Console({
  format: winston.format.combine(
    winston.format.colorize(),
    winston.format.timestamp({ format: "HH:mm:ss" }),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] ${level}: ${message}`;
    }),
  ),
});

const logger = winston.createLogger({
  level: isProduction ? "info" : "debug",
  transports: isProduction
    ? [fileTransport, errorFileTransport]
    : [consoleTransport],
});

module.exports = logger;
```

---

## Complete Express App dengan Logging

```javascript
// app.js
const express = require("express");
const { engine } = require("express-handlebars");
const morgan = require("morgan");
const logger = require("./config/logger");

const app = express();

// Setup view engine
app.engine("handlebars", engine());
app.set("view engine", "handlebars");
app.use(express.static("public"));
app.use(express.urlencoded({ extended: true }));

// HTTP request logging dengan Morgan
if (process.env.NODE_ENV !== "production") {
  app.use(morgan("dev"));
} else {
  app.use(
    morgan("combined", {
      stream: { write: (msg) => logger.info(msg.trim()) },
    }),
  );
}

// Application logging
logger.info("Application started", {
  nodeEnv: process.env.NODE_ENV,
  port: process.env.PORT || 3000,
});
```

---

## Complete Express App (lanjutan)

```javascript
// Routes
const userRoutes = require("./routes/users");
app.use("/users", userRoutes);

// 404 handler
app.use((req, res) => {
  logger.warn("404 Not Found", {
    url: req.url,
    method: req.method,
    ip: req.ip,
  });
  res.status(404).render("errors/404");
});

// Error handler
app.use((err, req, res, next) => {
  logger.error("Unhandled error", {
    error: err.message,
    stack: err.stack,
    url: req.url,
  });
  res.status(500).render("errors/500");
});

app.listen(3000, () => {
  logger.info("Server listening on port 3000");
});
```

---

## Monitoring Logs

**Development:**

```bash
# Watch logs in real-time
tail -f logs/app.log

# Search for errors
grep "ERROR" logs/app.log

# Search for specific user
grep "userId: 123" logs/app.log
```

**Production:**
Gunakan tools seperti:

- **ELK Stack** (Elasticsearch, Logstash, Kibana)
- **Splunk**
- **CloudWatch** (AWS)
- **Loggly**
- **Papertrail**

---

## Debugging dengan Logs

```javascript
// Add strategic log points
app.post("/users", async (req, res) => {
  logger.debug("POST /users started", { body: req.body });

  try {
    logger.debug("Validating user data");
    const validatedData = validateUser(req.body);

    logger.debug("Saving user to database", { data: validatedData });
    const user = await User.create(validatedData);

    logger.info("User created successfully", {
      userId: user.id,
      email: user.email,
    });

    res.redirect(`/users/${user.id}`);
  } catch (error) {
    logger.error("Error creating user", {
      error: error.message,
      stack: error.stack,
      body: req.body,
    });
    res.status(500).render("errors/500");
  }
});
```

---

## Performance Monitoring

```javascript
// Middleware untuk logging request duration
app.use((req, res, next) => {
  const start = Date.now();

  res.on("finish", () => {
    const duration = Date.now() - start;

    logger.info("Request completed", {
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
    });

    // Warning untuk slow requests
    if (duration > 1000) {
      logger.warn("Slow request detected", {
        method: req.method,
        url: req.url,
        duration: `${duration}ms`,
      });
    }
  });

  next();
});
```

---

## Log File Structure

```
project/
├── app.js
├── logs/
│   ├── app-2026-04-14.log
│   ├── app-2026-04-15.log
│   ├── error-2026-04-14.log
│   ├── error-2026-04-15.log
│   └── access.log
├── config/
│   └── logger.js
└── .gitignore  # Tambahkan logs/ ke .gitignore
```

**.gitignore:**

```
logs/
*.log
```

**JANGAN commit log files ke git!**

---

## Environment-Based Logging

```javascript
// .env.development
NODE_ENV = development;
LOG_LEVEL = debug;

// .env.production
NODE_ENV = production;
LOG_LEVEL = info;
```

```javascript
// config/logger.js
require("dotenv").config();

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  // ... rest of config
});
```

---

## Custom Logger Helper

```javascript
// utils/loggerHelper.js
const logger = require("../config/logger");

class LoggerHelper {
  static logUserAction(action, userId, details = {}) {
    logger.info("User action", {
      event: "user_action",
      action,
      userId,
      timestamp: new Date().toISOString(),
      ...details,
    });
  }

  static logDatabaseOperation(operation, table, details = {}) {
    logger.debug("Database operation", {
      event: "db_operation",
      operation,
      table,
      ...details,
    });
  }

  static logSecurityEvent(event, details = {}) {
    logger.warn("Security event", {
      event: "security",
      type: event,
      timestamp: new Date().toISOString(),
      ...details,
    });
  }
}

module.exports = LoggerHelper;
```

---

## Using Logger Helper

```javascript
const LoggerHelper = require("../utils/loggerHelper");

// Log user action
app.post("/users/:id/update", async (req, res) => {
  const userId = req.params.id;

  LoggerHelper.logUserAction("update_profile", userId, {
    fields: Object.keys(req.body),
  });

  // ... update user
});

// Log security event
app.post("/login", async (req, res) => {
  // ... after failed login
  LoggerHelper.logSecurityEvent("failed_login", {
    email: req.body.email,
    ip: req.ip,
  });
});

// Log database operation
LoggerHelper.logDatabaseOperation("INSERT", "users", {
  recordCount: 1,
});
```

---

## Summary

**Console Methods:**

- `console.log()`, `console.error()`, `console.warn()`, `console.debug()`
- Good for development, not production

**Winston:**

- Professional logging library
- Multiple transports (console, file)
- Log levels, custom formats
- Log rotation

**Morgan:**

- HTTP request logging
- Predefined formats
- Integration dengan Winston

**Best Practices:**

- Appropriate log levels
- Structured logging
- Don't log sensitive data
- Use log rotation
- Different configs for dev/prod

---

## Resources & Referensi

- [Winston Documentation](https://github.com/winstonjs/winston)
- [Morgan Documentation](https://github.com/expressjs/morgan)
- [winston-daily-rotate-file](https://github.com/winstonjs/winston-daily-rotate-file)
- [Logging Best Practices](https://www.datadoghq.com/blog/node-logging-best-practices/)
- [12 Factor App - Logs](https://12factor.net/logs)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Happy Logging! 📝
