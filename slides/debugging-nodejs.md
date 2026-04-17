---
title: Debugging di Node.js
version: 1.0.0
header: Debugging di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Debugging di Node.js

---

## Tujuan Pembelajaran

- Memahami konsep debugging dan pentingnya
- Menguasai teknik debugging di Node.js
- Menggunakan console methods secara efektif
- Debugging dengan VS Code debugger
- Debugging dengan Chrome DevTools
- Menggunakan Node.js built-in debugger
- Menerapkan debugging best practices
- Troubleshooting common issues

---

## Apa itu Debugging?

**Debugging** adalah proses menemukan dan memperbaiki bug (kesalahan) dalam code

**Bug** bisa berupa:

- 🐛 Syntax errors - kesalahan penulisan code
- 🔴 Runtime errors - error saat program berjalan
- 🤔 Logic errors - code berjalan tapi hasilnya salah
- 🐌 Performance issues - code lambat
- 💾 Memory leaks - penggunaan memory tidak efisien

**"Debugging is twice as hard as writing the code in the first place"** - Brian Kernighan

---

## Mengapa Debugging Penting?

- 💰 **Menghemat waktu dan biaya** - menemukan bug lebih cepat
- 🎯 **Meningkatkan kualitas code** - memahami flow program
- 🧠 **Belajar lebih dalam** - memahami cara kerja code
- 🚀 **Meningkatkan produktivitas** - debug efisien = develop lebih cepat
- 🛡️ **Mencegah masalah production** - tangkap bug sebelum deploy

---

## Level 1: Console Methods

**console.log() dan teman-temannya**

```javascript
// Basic logging
console.log("Hello World");
console.log("User:", user);
console.log("Name:", name, "Age:", age);

// Info, warning, error
console.info("ℹ️ Info message");
console.warn("⚠️ Warning message");
console.error("❌ Error message");

// Debug message (only shown when NODE_DEBUG is set)
console.debug("🐛 Debug message");
```

**Tips**: Gunakan emoji atau prefix untuk membedakan log messages!

---

## Console Methods: Advanced

```javascript
// Group related logs
console.group("User Details");
console.log("Name:", user.name);
console.log("Email:", user.email);
console.log("Age:", user.age);
console.groupEnd();

// Table format (bagus untuk array of objects)
const users = [
  { name: "Alice", age: 25, role: "Admin" },
  { name: "Bob", age: 30, role: "User" },
];
console.table(users);

// Time measurement
console.time("fetchData");
await fetchDataFromAPI();
console.timeEnd("fetchData"); // Output: fetchData: 250ms

// Count how many times a line is executed
for (let i = 0; i < 5; i++) {
  console.count("Loop iteration");
}

// Stack trace
console.trace("Show call stack");
```

---

## Console Methods: Object Inspection

```javascript
const user = {
  name: "Alice",
  email: "alice@example.com",
  profile: {
    age: 25,
    address: {
      city: "Jakarta",
      country: "Indonesia",
    },
  },
};

// Basic log (terpotong jika nested)
console.log(user);

// Dir - inspect object dengan detail
console.dir(user, { depth: null, colors: true });

// Assert - log only if condition is false
console.assert(user.age >= 18, "User must be 18 or older");
console.assert(user.email.includes("@"), "Invalid email format");

// Clear console
console.clear();
```

---

## Console Debugging Best Practices

✅ **DO:**

```javascript
// Descriptive messages
console.log("✅ User registered:", user.email);

// Use different levels
console.error("❌ Failed to save:", error.message);
console.warn("⚠️ Deprecated method used");

// Structured logging
console.log({
  timestamp: new Date(),
  level: "info",
  message: "User login",
  userId: user.id,
});
```

❌ **DON'T:**

```javascript
// Cryptic messages
console.log("here");
console.log("test");
console.log(x);

// Too verbose
console.log("====================================");
console.log("Starting function...");
```

---

## Node.js Built-in Debugger

**Node.js memiliki built-in debugger (inspect mode)**

```bash
# Run dengan inspect mode
node inspect app.js

# Atau
node --inspect app.js

# Atau dengan break di first line
node --inspect-brk app.js
```

**Debugger commands:**

- `cont`, `c` - Continue execution
- `next`, `n` - Step next (tidak masuk ke function)
- `step`, `s` - Step into (masuk ke function)
- `out`, `o` - Step out (keluar dari function)
- `pause` - Pause execution
- `watch('variable')` - Watch variable value
- `repl` - Open REPL untuk inspect variables

---

## Debugger Statement

**Gunakan `debugger` keyword untuk set breakpoint di code**

```javascript
function calculateTotal(items) {
  let total = 0;

  debugger; // Execution will pause here when debugging

  for (const item of items) {
    total += item.price * item.quantity;
    debugger; // Pause di setiap iteration
  }

  return total;
}

const cart = [
  { name: "Laptop", price: 10000000, quantity: 1 },
  { name: "Mouse", price: 100000, quantity: 2 },
];

const total = calculateTotal(cart);
console.log("Total:", total);
```

```bash
# Run with inspect
node inspect app.js
```

**Note**: `debugger` statement diabaikan jika tidak dalam debug mode

---

## VS Code Debugger - Setup

**VS Code memiliki debugger yang powerful dan user-friendly**

**Setup:**

1. Buka file yang ingin di-debug
2. Set breakpoint dengan klik di sebelah kiri line number (red dot)
3. Press `F5` atau Run > Start Debugging
4. Atau buat `.vscode/launch.json` untuk konfigurasi custom

**Shortcut keys:**

- `F5` - Continue
- `F10` - Step over (next)
- `F11` - Step into
- `Shift+F11` - Step out
- `Ctrl+Shift+F5` - Restart
- `Shift+F5` - Stop

---

## VS Code Debugger - launch.json

**Buat `.vscode/launch.json` untuk konfigurasi debugging:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/app.js"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Launch with nodemon",
      "runtimeExecutable": "nodemon",
      "program": "${workspaceFolder}/app.js",
      "restart": true,
      "console": "integratedTerminal"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Tests",
      "program": "${workspaceFolder}/node_modules/jest/bin/jest",
      "args": ["--runInBand"],
      "console": "integratedTerminal"
    }
  ]
}
```

---

## VS Code Debugger - Features

**Debug Console:**

- Evaluate expressions saat paused
- Access variables in current scope
- Execute code

**Watch Variables:**

- Add expressions to watch
- Auto-update saat execution

**Call Stack:**

- Lihat function call hierarchy
- Navigate ke different stack frames

**Breakpoints:**

- Regular breakpoint - pause setiap kali
- Conditional breakpoint - pause jika kondisi true
- Logpoint - log message tanpa pause

---

## VS Code Debugger - Conditional Breakpoint

**Breakpoint yang hanya trigger jika kondisi terpenuhi:**

```javascript
function processUsers(users) {
  for (let i = 0; i < users.length; i++) {
    const user = users[i];

    // Set conditional breakpoint di line ini:
    // Right click > Add Conditional Breakpoint
    // Condition: user.age < 18 || user.role === 'admin'
    processUser(user);
  }
}
```

**Logpoint:**

```javascript
// Right click > Add Logpoint
// Message: User {user.name} has role {user.role}
// Tidak pause execution, hanya log message
processUser(user);
```

---

## Chrome DevTools Debugging

**Node.js bisa di-debug dengan Chrome DevTools**

**Step 1**: Run Node.js dengan `--inspect` flag

```bash
node --inspect app.js
# atau
node --inspect-brk app.js  # Break at first line
```

**Output:**

```
Debugger listening on ws://127.0.0.1:9229/[uuid]
For help, see: https://nodejs.org/en/docs/inspector
```

**Step 2**: Buka Chrome dan ketik:

```
chrome://inspect
```

**Step 3**: Click "inspect" pada target yang muncul

---

## Chrome DevTools - Features

**Sama seperti debugging di browser:**

- 🔍 **Sources tab** - lihat code, set breakpoints
- 👁️ **Watch** - monitor variable values
- 📞 **Call Stack** - lihat function calls
- 🔬 **Scope** - local, closure, global variables
- 💾 **Memory profiler** - analyze memory usage
- ⚡ **Performance profiler** - analyze performance
- 🌐 **Network** - monitor HTTP requests (dengan libraries tertentu)

---

## Debugging Asynchronous Code

**Async code bisa tricky untuk di-debug:**

```javascript
async function fetchUserData(userId) {
  try {
    debugger; // Breakpoint 1

    const response = await fetch(`/api/users/${userId}`);
    debugger; // Breakpoint 2 - setelah fetch selesai

    const user = await response.json();
    debugger; // Breakpoint 3 - setelah parsing JSON

    return user;
  } catch (error) {
    debugger; // Breakpoint 4 - jika ada error
    console.error("Error fetching user:", error);
    throw error;
  }
}

// Test
fetchUserData(123)
  .then((user) => {
    debugger; // Breakpoint 5 - success
    console.log("User:", user);
  })
  .catch((error) => {
    debugger; // Breakpoint 6 - error
    console.error("Failed:", error);
  });
```

---

## Debugging Promises

```javascript
function getUserOrders(userId) {
  return fetchUser(userId)
    .then((user) => {
      debugger; // Step 1: inspect user
      console.log("User fetched:", user);
      return fetchOrders(user.id);
    })
    .then((orders) => {
      debugger; // Step 2: inspect orders
      console.log("Orders fetched:", orders);
      return orders;
    })
    .catch((error) => {
      debugger; // Catch any error
      console.error("Error in promise chain:", error);
      throw error;
    });
}
```

**Tip**: Gunakan `async/await` instead of `.then()` untuk debugging yang lebih mudah!

---

## Debugging Express.js Applications

```javascript
const express = require("express");
const app = express();

// Middleware untuk logging
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  console.log("Headers:", req.headers);
  console.log("Body:", req.body);
  debugger; // Debug setiap request
  next();
});

app.get("/users/:id", async (req, res) => {
  debugger; // Set breakpoint di sini

  const userId = req.params.id;
  console.log("Fetching user:", userId);

  try {
    const user = await User.findById(userId);
    debugger; // Inspect user data

    if (!user) {
      debugger; // Debug not found case
      return res.status(404).json({ error: "User not found" });
    }

    res.json(user);
  } catch (error) {
    debugger; // Debug error case
    console.error("Error:", error);
    res.status(500).json({ error: "Server error" });
  }
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## Debug Environment Variables

```javascript
// Cek semua environment variables
console.log("All env vars:", process.env);

// Cek specific variables
console.log({
  NODE_ENV: process.env.NODE_ENV,
  PORT: process.env.PORT,
  DATABASE_URL: process.env.DATABASE_URL,
  API_KEY: process.env.API_KEY ? "***SET***" : "NOT SET", // Don't log secrets!
});

// Debug env loading
require("dotenv").config();
console.log("After loading .env:");
console.log("PORT:", process.env.PORT);

// Assert env vars are set
if (!process.env.DATABASE_URL) {
  console.error("❌ DATABASE_URL not set!");
  process.exit(1);
}
```

---

## Debugging Database Queries

**Sequelize:**

```javascript
const { Sequelize } = require("sequelize");

const sequelize = new Sequelize(process.env.DATABASE_URL, {
  logging: console.log, // Log all SQL queries
  // atau
  logging: (sql, timing) => {
    console.log("📊 SQL:", sql);
    console.log("⏱️ Time:", timing, "ms");
  },
  // atau disable
  logging: false,
});

// Debug specific query
const users = await User.findAll({
  where: { age: { [Op.gt]: 18 } },
  logging: console.log, // Log hanya query ini
});
```

**Mongoose:**

```javascript
const mongoose = require("mongoose");

// Enable debug mode
mongoose.set("debug", true);

// Custom debug function
mongoose.set("debug", (collectionName, method, query, doc) => {
  console.log(`${collectionName}.${method}`, JSON.stringify(query), doc);
});
```

---

## Debugging with Logging Libraries

**Winston - Professional logging:**

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: "debug",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.colorize(),
    winston.format.printf(({ timestamp, level, message, ...meta }) => {
      return `${timestamp} [${level}]: ${message} ${
        Object.keys(meta).length ? JSON.stringify(meta, null, 2) : ""
      }`;
    }),
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "debug.log", level: "debug" }),
    new winston.transports.File({ filename: "error.log", level: "error" }),
  ],
});

// Usage
logger.debug("Debug message", { userId: 123, action: "login" });
logger.info("User logged in", { userId: 123 });
logger.warn("Deprecated method called");
logger.error("Failed to connect to database", { error: err.message });
```

---

## Morgan - HTTP Request Logging

```javascript
const express = require("express");
const morgan = require("morgan");
const app = express();

// Predefined formats
app.use(morgan("dev")); // :method :url :status :response-time ms
// atau
app.use(morgan("combined")); // Apache combined format
// atau
app.use(morgan("tiny")); // Minimal format

// Custom format
morgan.token("body", (req) => JSON.stringify(req.body));
app.use(
  morgan(":method :url :status :response-time ms - :body", {
    skip: (req, res) => res.statusCode < 400, // Log only errors
  }),
);

// Write to file
const fs = require("fs");
const path = require("path");
const accessLogStream = fs.createWriteStream(
  path.join(__dirname, "access.log"),
  {
    flags: "a",
  },
);
app.use(morgan("combined", { stream: accessLogStream }));
```

---

## Debug Module

**debug npm package - standard untuk library logging:**

```javascript
const debug = require("debug");

// Buat debugger dengan namespace
const debugMain = debug("app:main");
const debugDB = debug("app:database");
const debugAPI = debug("app:api");

debugMain("Application starting...");
debugDB("Connecting to database...");
debugAPI("API server listening on port 3000");

// Nested namespaces
const debugUserCreate = debug("app:user:create");
const debugUserDelete = debug("app:user:delete");

debugUserCreate("Creating user with email:", email);
debugUserDelete("Deleting user with id:", userId);
```

**Enable debug output:**

```bash
# Enable all
DEBUG=* node app.js

# Enable specific namespace
DEBUG=app:database node app.js

# Enable multiple namespaces
DEBUG=app:database,app:api node app.js

# Enable all under namespace
DEBUG=app:* node app.js

# Exclude namespace
DEBUG=*,-app:verbose node app.js
```

---

## Common Debugging Scenarios

**1. Undefined variable:**

```javascript
function greetUser(user) {
  console.log("greetUser called with:", user); // Debug input
  console.log("user.name:", user.name); // Check property
  return `Hello, ${user.name}!`;
}

// TypeError: Cannot read property 'name' of undefined
const message = greetUser(); // Missing argument!
```

**2. Wrong data type:**

```javascript
function calculateTotal(price, quantity) {
  console.log("price:", price, typeof price); // Debug: "10" string!
  console.log("quantity:", quantity, typeof quantity);

  const total = price * quantity;
  console.log("total:", total); // NaN if price is string

  return total;
}

// Fix: parse to number
const total = Number(price) * Number(quantity);
```

---

## Common Debugging Scenarios (cont.)

**3. Async timing issue:**

```javascript
let data = null;

// ❌ Wrong: accessing data before it's ready
fetchData().then((result) => {
  data = result;
});
console.log(data); // null! Promise belum resolve

// ✅ Right: wait for promise
fetchData().then((result) => {
  data = result;
  console.log(data); // Correct timing
});

// ✅ Better: use async/await
async function main() {
  const data = await fetchData();
  console.log(data); // Correct timing
}
```

**4. Mutation issue:**

```javascript
const user = { name: "Alice", age: 25 };
const updatedUser = user; // ❌ Same reference!

updatedUser.age = 26;
console.log(user.age); // 26 - original mutated!

// ✅ Fix: create new object
const updatedUser = { ...user, age: 26 };
```

---

## Debugging Memory Leaks

**Check memory usage:**

```javascript
// Log memory usage
setInterval(() => {
  const usage = process.memoryUsage();
  console.log({
    rss: `${Math.round(usage.rss / 1024 / 1024)} MB`, // Resident Set Size
    heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)} MB`,
    heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)} MB`,
    external: `${Math.round(usage.external / 1024 / 1024)} MB`,
  });
}, 5000);
```

**Common memory leak causes:**

- Global variables yang terus bertambah
- Event listeners tidak di-remove
- Closures yang menyimpan reference
- Timers (setInterval, setTimeout) tidak di-clear
- Cache tanpa eviction strategy

---

## Debugging Memory Leaks (cont.)

**Example memory leak:**

```javascript
// ❌ Memory leak: event listeners terus ditambah
function setupListener(element) {
  element.addEventListener("click", () => {
    console.log("Clicked!");
  });
}

// Dipanggil berkali-kali = memory leak!
setInterval(() => {
  setupListener(document.getElementById("button"));
}, 1000);

// ✅ Fix: simpan reference dan remove old listener
let clickHandler = null;

function setupListener(element) {
  if (clickHandler) {
    element.removeEventListener("click", clickHandler);
  }

  clickHandler = () => console.log("Clicked!");
  element.addEventListener("click", clickHandler);
}
```

---

## Debugging Performance Issues

**Measure execution time:**

```javascript
// Method 1: console.time
console.time("queryDatabase");
const users = await User.findAll();
console.timeEnd("queryDatabase"); // queryDatabase: 245ms

// Method 2: performance API
const { performance } = require("perf_hooks");

const start = performance.now();
const users = await User.findAll();
const end = performance.now();
console.log(`Query took ${end - start}ms`);

// Method 3: benchmark multiple operations
async function benchmark() {
  const results = [];

  for (let i = 0; i < 100; i++) {
    const start = performance.now();
    await someOperation();
    const end = performance.now();
    results.push(end - start);
  }

  const avg = results.reduce((a, b) => a + b) / results.length;
  console.log(`Average: ${avg}ms`);
  console.log(`Min: ${Math.min(...results)}ms`);
  console.log(`Max: ${Math.max(...results)}ms`);
}
```

---

## Node.js Profiling

**CPU Profiling:**

```bash
# Generate CPU profile
node --prof app.js

# Process results
node --prof-process isolate-0x*.log > processed.txt
```

**Heap Snapshot:**

```javascript
const v8 = require("v8");
const fs = require("fs");

// Take heap snapshot
function takeHeapSnapshot() {
  const filename = `heap-${Date.now()}.heapsnapshot`;
  const snapshot = v8.writeHeapSnapshot(filename);
  console.log("Heap snapshot written to", snapshot);
}

// Take snapshot periodically
setInterval(takeHeapSnapshot, 10000);
```

**Analyze di Chrome DevTools:**

1. Open Chrome DevTools
2. Memory tab
3. Load snapshot file
4. Analyze memory usage

---

## Debugging Tools & Utilities

**Nodemon** - Auto-restart on file changes:

```bash
npm install -g nodemon
nodemon app.js
```

**Node Inspector** - Legacy debugger UI:

```bash
npm install -g node-inspector
node-debug app.js
```

**ndb** - Improved debugging with Chrome DevTools:

```bash
npm install -g ndb
ndb app.js
```

**Clinic.js** - Performance profiling:

```bash
npm install -g clinic
clinic doctor -- node app.js
clinic bubbleprof -- node app.js
clinic flame -- node app.js
```

---

## Error Stack Traces

**Understanding stack traces:**

```javascript
function functionA() {
  functionB();
}

function functionB() {
  functionC();
}

function functionC() {
  throw new Error("Something went wrong!");
}

try {
  functionA();
} catch (error) {
  console.error(error.stack);
}
```

**Output:**

```
Error: Something went wrong!
    at functionC (/path/to/app.js:10:9)
    at functionB (/path/to/app.js:6:3)
    at functionA (/path/to/app.js:2:3)
    at Object.<anonymous> (/path/to/app.js:15:3)
```

**Read from bottom to top untuk trace flow!**

---

## Source Maps

**Untuk TypeScript atau transpiled code:**

```json
// tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSourceMap": false,
    "sourceRoot": "./src"
  }
}
```

**Enable source maps in Node.js:**

```bash
node --enable-source-maps dist/app.js
```

**Stack trace akan menunjuk ke original source code!**

---

## Remote Debugging

**Debug aplikasi yang berjalan di server remote:**

```bash
# Di server, run dengan --inspect
node --inspect=0.0.0.0:9229 app.js

# Di local machine, forward port (SSH)
ssh -L 9229:localhost:9229 user@server-ip

# Buka Chrome DevTools
chrome://inspect
```

**Docker debugging:**

```dockerfile
# Dockerfile
EXPOSE 9229
CMD ["node", "--inspect=0.0.0.0:9229", "app.js"]
```

```bash
# Run dengan port mapping
docker run -p 3000:3000 -p 9229:9229 myapp
```

---

## Debugging Best Practices

✅ **DO:**

- Use descriptive variable names untuk debugging mudah
- Add strategic console.logs di critical points
- Use debugger statement untuk complex logic
- Test edge cases dan error scenarios
- Remove atau disable debug logs di production
- Use logging levels (debug, info, warn, error)
- Keep debug logs organized dengan timestamps
- Use source control untuk track changes
- Document common debugging scenarios
- Setup proper error monitoring (Sentry, LogRocket)

---

## Debugging Best Practices (cont.)

❌ **DON'T:**

- Debug dengan print statements di mana-mana
- Commit code dengan banyak console.log
- Ignore warning messages
- Debug production code tanpa proper logging
- Hard-code values untuk testing
- Skip error handling
- Forget to check async operation results
- Over-complicate debugging dengan terlalu banyak tools
- Panic saat menemukan bug - debug systematically!

---

## Debugging Workflow

**Systematic approach untuk solve bugs:**

1. **🎯 Reproduce the bug** - buat bug terjadi secara konsisten
2. **🔍 Isolate the problem** - temukan code yang menyebabkan bug
3. **📝 Understand the code** - baca dan pahami logic
4. **🐛 Add debugging** - console.log, breakpoints, dll
5. **🧪 Test hypothesis** - verify what's wrong
6. **🔧 Fix the bug** - implement solution
7. **✅ Test the fix** - make sure bug resolved
8. **📚 Document** - note what caused it & solution
9. **🛡️ Prevent recurrence** - add tests, improve code

---

## Debugging Checklist

**Saat debugging, check:**

- ✅ Are all variables defined?
- ✅ Are all variables the right type?
- ✅ Are all async operations awaited?
- ✅ Are promises properly handled with catch?
- ✅ Are error cases handled?
- ✅ Is the function called with correct arguments?
- ✅ Are environment variables set?
- ✅ Is database connected?
- ✅ Are all required modules installed?
- ✅ Is the code running the latest version?
- ✅ Are there any typos in variable/function names?

---

## Debugging Quiz

**What's wrong with this code?**

```javascript
async function getUser(id) {
  const user = fetch(`/api/users/${id}`);
  console.log(user.name);
  return user;
}
```

<details>
<summary>Click untuk answer</summary>

**Problem**: Missing `await` keyword!

```javascript
async function getUser(id) {
  const user = await fetch(`/api/users/${id}`);
  const userData = await user.json();
  console.log(userData.name);
  return userData;
}
```

</details>

---

## Summary

- **Debugging** adalah skill penting untuk developer
- **console methods** untuk quick debugging
- **VS Code debugger** untuk interactive debugging
- **Chrome DevTools** untuk advanced debugging
- **Breakpoints** untuk pause dan inspect code
- **Async debugging** memerlukan perhatian khusus
- **Logging libraries** untuk professional applications
- **Performance profiling** untuk optimize code
- **Systematic approach** solve bugs lebih cepat
- **Practice makes perfect** - semakin sering debug, semakin mahir!

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih!

## Questions?

**Resources:**

- Node.js Debugging Guide: https://nodejs.org/en/docs/guides/debugging-getting-started/
- VS Code Debugging: https://code.visualstudio.com/docs/nodejs/nodejs-debugging
- Chrome DevTools: https://developer.chrome.com/docs/devtools/
- Debug module: https://www.npmjs.com/package/debug
- Winston logging: https://github.com/winstonjs/winston
