---
title: Testing di Node.js
version: 1.0.0
header: Testing di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Testing di Node.js

---

## Tujuan Pembelajaran

- Memahami pentingnya testing dalam software development
- Mengenal berbagai jenis testing (unit, integration, e2e)
- Menguasai Jest untuk testing JavaScript/Node.js
- Mampu membuat unit test dan integration test
- Memahami mocking dan stubbing
- Menerapkan Test-Driven Development (TDD)
- Mengukur test coverage

---

## Mengapa Testing Penting?

✅ **Menemukan bug lebih cepat** - sebelum masuk production
✅ **Confidence** - yakin code bekerja dengan benar
✅ **Documentation** - test adalah dokumentasi code
✅ **Refactoring** - mudah refactor tanpa takut break
✅ **Maintenance** - mudah maintain code
✅ **Quality** - meningkatkan kualitas code

---

## Jenis-jenis Testing

**1. Unit Testing**

- Test function/method individual
- Isolated, tidak tergantung komponen lain
- Cepat dan mudah

**2. Integration Testing**

- Test interaksi antar komponen
- Database, API, services

**3. End-to-End (E2E) Testing**

- Test aplikasi secara keseluruhan
- Simulasi user behavior

---

## Testing Pyramid

```
        /\
       /  \  E2E Tests (sedikit, lambat)
      /----\
     /      \  Integration Tests (sedang)
    /--------\
   /          \  Unit Tests (banyak, cepat)
  /------------\
```

**Prinsip:**

- Banyak unit tests (cepat, murah)
- Sedang integration tests
- Sedikit E2E tests (lambat, mahal)

---

## Testing Framework

**Jest** ⭐ (Most Popular)

- All-in-one testing framework
- Built-in assertion, mocking, coverage
- Fast dan easy to use

**Mocha + Chai**

- Flexible, modular
- Butuh library tambahan

**AVA**

- Minimalist, concurrent

**Supertest**

- HTTP testing

---

## Install Jest

```bash
npm install --save-dev jest
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

```bash
# Run tests
npm test
```

---

## Struktur File Testing

```
project/
├── src/
│   ├── utils/
│   │   └── math.js
│   ├── models/
│   │   └── User.js
│   └── controllers/
│       └── userController.js
├── tests/
│   ├── unit/
│   │   ├── utils/
│   │   │   └── math.test.js
│   │   └── models/
│   │       └── User.test.js
│   └── integration/
│       └── user.test.js
└── package.json
```

---

## Unit Test Pertama

```javascript
// src/utils/math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = { add, subtract };
```

```javascript
// tests/unit/utils/math.test.js
const { add, subtract } = require("../../../src/utils/math");

describe("Math functions", () => {
  test("add should return sum of two numbers", () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
  });

  test("subtract should return difference", () => {
    expect(subtract(5, 3)).toBe(2);
  });
});
```

---

## Jest Matchers

```javascript
// Equality
expect(value).toBe(5); // strict equality (===)
expect(value).toEqual({ name: "John" }); // deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeLessThan(5);
expect(value).toBeCloseTo(0.3); // floating point

// Strings
expect(value).toMatch(/pattern/);
expect(value).toContain("substring");
```

---

## Jest Matchers (lanjutan)

```javascript
// Arrays
expect(array).toContain("item");
expect(array).toHaveLength(3);

// Objects
expect(obj).toHaveProperty("name");
expect(obj).toMatchObject({ name: "John" });

// Exceptions
expect(() => {
  throw new Error("Failed");
}).toThrow();
expect(() => func()).toThrow("Failed");

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();
```

---

## Struktur Test

```javascript
describe("User Service", () => {
  // Setup sebelum semua test
  beforeAll(() => {
    // Connect database, etc
  });

  // Setup sebelum setiap test
  beforeEach(() => {
    // Reset data, clear cache, etc
  });

  // Cleanup setelah setiap test
  afterEach(() => {
    // Clear data
  });

  // Cleanup setelah semua test
  afterAll(() => {
    // Close connections
  });

  test("should create user", () => {
    // Test code
  });
});
```

---

## Test User Model

```javascript
// models/User.js
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  validate() {
    if (!this.name || !this.email) {
      throw new Error("Name and email are required");
    }

    if (!this.email.includes("@")) {
      throw new Error("Invalid email");
    }

    return true;
  }
}

module.exports = User;
```

---

## User Model Test

```javascript
// tests/unit/models/User.test.js
const User = require("../../../src/models/User");

describe("User Model", () => {
  test("should create user with name and email", () => {
    const user = new User("John", "john@example.com");

    expect(user.name).toBe("John");
    expect(user.email).toBe("john@example.com");
  });

  test("validate should pass for valid user", () => {
    const user = new User("John", "john@example.com");
    expect(user.validate()).toBe(true);
  });

  test("validate should throw error for missing fields", () => {
    const user = new User("", "");

    expect(() => user.validate()).toThrow("Name and email are required");
  });

  test("validate should throw error for invalid email", () => {
    const user = new User("John", "invalid-email");

    expect(() => user.validate()).toThrow("Invalid email");
  });
});
```

---

## Async Function Testing

```javascript
// services/userService.js
async function getUserById(id) {
  const response = await fetch(`https://api.example.com/users/${id}`);
  const data = await response.json();
  return data;
}

module.exports = { getUserById };
```

```javascript
// tests/unit/services/userService.test.js
const { getUserById } = require("../../../src/services/userService");

test("getUserById should return user data", async () => {
  const user = await getUserById(1);

  expect(user).toBeDefined();
  expect(user).toHaveProperty("id");
  expect(user.id).toBe(1);
});
```

---

## Mocking

**Mock** = fake implementation untuk testing

Contoh kenapa butuh mock:

- Tidak ingin hit API real saat testing
- Tidak ingin akses database real
- Tidak ingin send email real
- Control test environment

---

## Mock Functions

```javascript
// Create mock function
const mockFn = jest.fn();

// Mock return value
mockFn.mockReturnValue(42);
mockFn(); // Returns 42

// Mock implementation
mockFn.mockImplementation((a, b) => a + b);
mockFn(2, 3); // Returns 5

// Mock resolved value (Promise)
mockFn.mockResolvedValue({ id: 1, name: "John" });
await mockFn(); // Returns { id: 1, name: 'John' }

// Check if called
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);
expect(mockFn).toHaveBeenCalledWith(2, 3);
```

---

## Mock Module

```javascript
// services/database.js
async function query(sql) {
  // Real database query
}

module.exports = { query };
```

```javascript
// tests/unit/services/userService.test.js
const db = require("../../../src/services/database");

// Mock entire module
jest.mock("../../../src/services/database");

test("should fetch users from database", async () => {
  // Mock implementation
  db.query.mockResolvedValue([
    { id: 1, name: "John" },
    { id: 2, name: "Jane" },
  ]);

  const users = await UserService.getAllUsers();

  expect(db.query).toHaveBeenCalledWith("SELECT * FROM users");
  expect(users).toHaveLength(2);
});
```

---

## Mock Axios

```javascript
// services/apiService.js
const axios = require("axios");

async function getUser(id) {
  const response = await axios.get(`/api/users/${id}`);
  return response.data;
}

module.exports = { getUser };
```

```javascript
// tests/unit/services/apiService.test.js
const axios = require("axios");
const { getUser } = require("../../../src/services/apiService");

jest.mock("axios");

test("getUser should fetch user from API", async () => {
  const mockUser = { id: 1, name: "John" };
  axios.get.mockResolvedValue({ data: mockUser });

  const user = await getUser(1);

  expect(axios.get).toHaveBeenCalledWith("/api/users/1");
  expect(user).toEqual(mockUser);
});
```

---

## Spy

**Spy** = monitor function calls tanpa mengubah behavior

```javascript
const user = {
  getName: () => "John",
  setName: (name) => {
    this.name = name;
  },
};

// Spy on method
const spy = jest.spyOn(user, "getName");

user.getName();

expect(spy).toHaveBeenCalled();
expect(spy).toHaveReturnedWith("John");

// Restore original implementation
spy.mockRestore();
```

---

## Integration Testing - API

```bash
npm install --save-dev supertest
```

```javascript
// app.js
const express = require("express");
const app = express();

app.use(express.json());

app.get("/api/users", (req, res) => {
  res.json({ users: [] });
});

app.post("/api/users", (req, res) => {
  res.status(201).json({ message: "User created" });
});

module.exports = app;
```

---

## Supertest - HTTP Testing

```javascript
// tests/integration/user.test.js
const request = require("supertest");
const app = require("../../src/app");

describe("User API", () => {
  test("GET /api/users should return users", async () => {
    const response = await request(app).get("/api/users").expect(200);

    expect(response.body).toHaveProperty("users");
    expect(Array.isArray(response.body.users)).toBe(true);
  });

  test("POST /api/users should create user", async () => {
    const newUser = {
      name: "John",
      email: "john@example.com",
    };

    const response = await request(app)
      .post("/api/users")
      .send(newUser)
      .expect(201);

    expect(response.body.message).toBe("User created");
  });
});
```

---

## Testing dengan Database

```javascript
const mongoose = require("mongoose");
const User = require("../../src/models/User");

// Setup database sebelum test
beforeAll(async () => {
  await mongoose.connect(process.env.MONGO_TEST_URL);
});

// Cleanup setelah setiap test
afterEach(async () => {
  await User.deleteMany({});
});

// Close connection setelah semua test
afterAll(async () => {
  await mongoose.connection.close();
});

test("should create user in database", async () => {
  const userData = {
    name: "John",
    email: "john@example.com",
  };

  const user = await User.create(userData);

  expect(user._id).toBeDefined();
  expect(user.name).toBe("John");
});
```

---

## Test Coverage

```bash
npm test -- --coverage
```

**Output:**

```
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
All files           |   85.71 |    66.67 |   83.33 |   85.71 |
 controllers        |   80.00 |    50.00 |   75.00 |   80.00 |
  userController.js |   80.00 |    50.00 |   75.00 |   80.00 |
 models             |   90.00 |    80.00 |   90.00 |   90.00 |
  User.js           |   90.00 |    80.00 |   90.00 |   90.00 |
--------------------|---------|----------|---------|---------|
```

**Target:** Minimal 80% coverage

---

## Jest Configuration

```javascript
// jest.config.js
module.exports = {
  testEnvironment: "node",
  coverageDirectory: "coverage",
  collectCoverageFrom: ["src/**/*.js", "!src/index.js", "!**/node_modules/**"],
  testMatch: ["**/tests/**/*.test.js"],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

---

## Test-Driven Development (TDD)

**Workflow:**

1. **Red** - Write failing test
2. **Green** - Write minimum code to pass
3. **Refactor** - Improve code

```javascript
// 1. RED - Write test first
test("should multiply two numbers", () => {
  expect(multiply(2, 3)).toBe(6);
});

// 2. GREEN - Write minimum code
function multiply(a, b) {
  return a * b;
}

// 3. REFACTOR - Improve if needed
```

---

## TDD Example - Password Validator

```javascript
// Step 1: Write test
describe("Password Validator", () => {
  test("should return false for short password", () => {
    expect(isValidPassword("12345")).toBe(false);
  });

  test("should return false without uppercase", () => {
    expect(isValidPassword("password123")).toBe(false);
  });

  test("should return false without number", () => {
    expect(isValidPassword("Password")).toBe(false);
  });

  test("should return true for valid password", () => {
    expect(isValidPassword("Password123")).toBe(true);
  });
});
```

---

## TDD Example - Implementation

```javascript
// Step 2: Write code to pass tests
function isValidPassword(password) {
  // Min 8 characters
  if (password.length < 8) {
    return false;
  }

  // Has uppercase
  if (!/[A-Z]/.test(password)) {
    return false;
  }

  // Has number
  if (!/\d/.test(password)) {
    return false;
  }

  return true;
}

module.exports = { isValidPassword };
```

---

## Snapshot Testing

**Snapshot** = capture output, compare di test berikutnya

```javascript
const formatUser = (user) => {
  return {
    displayName: `${user.firstName} ${user.lastName}`,
    email: user.email.toLowerCase(),
    createdAt: user.createdAt.toISOString(),
  };
};

test("formatUser snapshots", () => {
  const user = {
    firstName: "John",
    lastName: "Doe",
    email: "JOHN@EXAMPLE.COM",
    createdAt: new Date("2024-01-01"),
  };

  expect(formatUser(user)).toMatchSnapshot();
});
```

---

## Testing Error Handling

```javascript
// userService.js
async function createUser(userData) {
  if (!userData.email) {
    throw new Error("Email is required");
  }

  if (await User.exists({ email: userData.email })) {
    throw new Error("Email already exists");
  }

  return await User.create(userData);
}
```

```javascript
// userService.test.js
test("should throw error if email missing", async () => {
  await expect(createUser({ name: "John" })).rejects.toThrow(
    "Email is required",
  );
});

test("should throw error if email exists", async () => {
  await User.create({ email: "john@example.com" });

  await expect(createUser({ email: "john@example.com" })).rejects.toThrow(
    "Email already exists",
  );
});
```

---

## Testing Authentication

```javascript
describe("Authentication", () => {
  test("should login with valid credentials", async () => {
    // Create user
    await request(app)
      .post("/api/register")
      .send({ email: "john@example.com", password: "Password123" });

    // Login
    const response = await request(app)
      .post("/api/login")
      .send({ email: "john@example.com", password: "Password123" })
      .expect(200);

    expect(response.body).toHaveProperty("token");
  });

  test("should reject invalid credentials", async () => {
    const response = await request(app)
      .post("/api/login")
      .send({ email: "wrong@example.com", password: "wrong" })
      .expect(401);

    expect(response.body.error).toBeDefined();
  });
});
```

---

## Testing Protected Routes

```javascript
describe("Protected Routes", () => {
  let token;

  beforeEach(async () => {
    // Register & login to get token
    await request(app)
      .post("/api/register")
      .send({ email: "john@example.com", password: "Password123" });

    const loginRes = await request(app)
      .post("/api/login")
      .send({ email: "john@example.com", password: "Password123" });

    token = loginRes.body.token;
  });

  test("should access protected route with token", async () => {
    const response = await request(app)
      .get("/api/profile")
      .set("Authorization", `Bearer ${token}`)
      .expect(200);

    expect(response.body.user).toBeDefined();
  });

  test("should reject without token", async () => {
    await request(app).get("/api/profile").expect(401);
  });
});
```

---

## Mock Environment Variables

```javascript
describe("Config", () => {
  const originalEnv = process.env;

  beforeEach(() => {
    jest.resetModules();
    process.env = { ...originalEnv };
  });

  afterAll(() => {
    process.env = originalEnv;
  });

  test("should use production database", () => {
    process.env.NODE_ENV = "production";
    process.env.DB_URL = "mongodb://prod-db";

    const config = require("../../src/config");
    expect(config.dbUrl).toBe("mongodb://prod-db");
  });

  test("should use test database", () => {
    process.env.NODE_ENV = "test";
    process.env.DB_URL = "mongodb://test-db";

    const config = require("../../src/config");
    expect(config.dbUrl).toBe("mongodb://test-db");
  });
});
```

---

## Test Fixtures

```javascript
// tests/fixtures/users.js
const users = {
  validUser: {
    name: "John Doe",
    email: "john@example.com",
    password: "Password123",
  },
  adminUser: {
    name: "Admin",
    email: "admin@example.com",
    password: "AdminPass123",
    role: "admin",
  },
};

module.exports = users;
```

```javascript
// tests/integration/user.test.js
const fixtures = require("../fixtures/users");

test("should create user", async () => {
  const response = await request(app)
    .post("/api/users")
    .send(fixtures.validUser)
    .expect(201);

  expect(response.body.user.email).toBe(fixtures.validUser.email);
});
```

---

## Testing Utilities

```javascript
// tests/utils/testHelpers.js

// Generate test user
async function createTestUser(app, userData = {}) {
  const defaultData = {
    name: "Test User",
    email: "test@example.com",
    password: "Password123",
  };

  const response = await request(app)
    .post("/api/register")
    .send({ ...defaultData, ...userData });

  return response.body.user;
}

// Get auth token
async function getAuthToken(app, email, password) {
  const response = await request(app)
    .post("/api/login")
    .send({ email, password });

  return response.body.token;
}

module.exports = { createTestUser, getAuthToken };
```

---

## Parallel vs Sequential Tests

```javascript
// Parallel (default) - faster
describe("User Tests", () => {
  test("test 1", async () => {});
  test("test 2", async () => {});
});

// Sequential - satu per satu
describe.serial("Sequential Tests", () => {
  test("test 1", async () => {});
  test("test 2", async () => {});
});

// Run only this test
test.only("this test only", () => {});

// Skip this test
test.skip("skip this", () => {});
```

---

## Debugging Tests

```javascript
// Debug single test
test.only("debug this", () => {
  console.log("Debug info");
  debugger; // Breakpoint
  expect(true).toBe(true);
});
```

```bash
# Run with debugger
node --inspect-brk node_modules/.bin/jest --runInBand

# Run specific test file
npm test -- user.test.js

# Run tests matching pattern
npm test -- --testNamePattern="should create"
```

---

## Performance Testing

```javascript
test("should complete in reasonable time", async () => {
  const start = Date.now();

  await heavyOperation();

  const duration = Date.now() - start;
  expect(duration).toBeLessThan(1000); // < 1 second
});

// Set timeout untuk slow tests
test("slow test", async () => {
  await slowOperation();
}, 10000); // 10 seconds timeout
```

---

## Best Practices

1. **AAA Pattern** - Arrange, Act, Assert
2. **One assertion per test** (or few related)
3. **Descriptive test names** - "should do X when Y"
4. **Test isolation** - tests tidak depend satu sama lain
5. **Use beforeEach/afterEach** untuk setup/cleanup
6. **Mock external dependencies** - API, database
7. **Test edge cases** - empty, null, undefined
8. **Keep tests simple** - mudah dibaca dan dipahami

---

## AAA Pattern

```javascript
test("should calculate discount correctly", () => {
  // ARRANGE - Setup
  const product = { price: 100 };
  const discountPercent = 20;

  // ACT - Execute
  const finalPrice = calculateDiscount(product, discountPercent);

  // ASSERT - Verify
  expect(finalPrice).toBe(80);
});
```

---

## Good Test Names

```javascript
// ❌ BAD
test("user test", () => {});
test("test 1", () => {});

// ✅ GOOD
test("should create user with valid data", () => {});
test("should throw error when email is missing", () => {});
test("should return 401 for invalid credentials", () => {});
test("should update user only if owner or admin", () => {});

// Pattern: "should [expected behavior] when [condition]"
```

---

## CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Run Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: "18"

      - run: npm install
      - run: npm test
      - run: npm run test:coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

---

## Test Organization

```
tests/
├── unit/               # Unit tests
│   ├── models/
│   ├── services/
│   └── utils/
├── integration/        # Integration tests
│   ├── api/
│   └── database/
├── e2e/               # End-to-end tests
│   └── flows/
├── fixtures/          # Test data
│   └── users.js
└── utils/             # Test utilities
    └── testHelpers.js
```

---

## Kesalahan Umum

❌ Tidak test edge cases
❌ Test terlalu kompleks
❌ Test bergantung pada urutan
❌ Tidak cleanup setelah test
❌ Mock terlalu banyak (test implementation, bukan behavior)
❌ Test terlalu lambat (tidak mock eksternal)
❌ Tidak test error cases
❌ Copy-paste test tanpa modify

---

## When NOT to Test

🤔 **Tidak perlu test:**

- Third-party libraries
- Framework code
- Simple getters/setters
- Configuration files
- Auto-generated code

✅ **Focus test pada:**

- Business logic
- Complex algorithms
- Error handling
- Edge cases
- Critical paths

---

## Mocking vs Real Dependencies

**Mock ketika:**

- External API calls
- Database operations (untuk unit tests)
- Email/SMS services
- File system operations
- Time-dependent code

**Real ketika:**

- Integration tests
- Critical business logic
- Need to verify actual behavior

---

## Test Maintenance

```javascript
// Refactor common setup ke helper
function setupTestUser() {
  return {
    name: "Test User",
    email: "test@example.com",
    password: "Password123",
  };
}

// DRY - Don't Repeat Yourself
beforeEach(async () => {
  await clearDatabase();
  testUser = await createTestUser();
});

// Update tests saat code berubah
// Delete obsolete tests
// Keep tests readable dan maintainable
```

---

## Resources & Referensi

- [Jest Documentation](https://jestjs.io/)
- [Testing Best Practices](https://testingjavascript.com/)
- [Supertest Documentation](https://github.com/visionmedia/supertest)
- [Test Driven Development (TDD)](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
- [Testing JavaScript by Kent C. Dodds](https://testingjavascript.com/)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Keep Testing! ✅
