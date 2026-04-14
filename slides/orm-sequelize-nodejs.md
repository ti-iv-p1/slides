---
title: ORM di Node.js dengan Sequelize
version: 1.0.0
header: ORM di Node.js dengan Sequelize
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# ORM di Node.js dengan Sequelize

---

## Tujuan Pembelajaran

- Memahami konsep ORM (Object-Relational Mapping)
- Menguasai Sequelize untuk database management
- Mampu membuat model dan migrations
- Menguasai CRUD operations dengan Sequelize
- Memahami associations (relationships)
- Menerapkan validations dan hooks
- Menggunakan transactions dan query optimization

---

## Apa itu ORM?

**ORM (Object-Relational Mapping)**

- Teknik untuk mapping antara object dan relational database
- Allows you to work with database using objects, bukan SQL

**Tanpa ORM:**

```javascript
db.query("INSERT INTO users (name, email) VALUES (?, ?)", [
  "John",
  "john@example.com",
]);
```

**Dengan ORM:**

```javascript
await User.create({ name: "John", email: "john@example.com" });
```

---

## Keuntungan ORM

✅ **Abstraksi SQL** - tidak perlu tulis raw SQL
✅ **Type Safety** - validasi data types
✅ **Database Agnostic** - support multiple databases
✅ **Migrations** - version control untuk database schema
✅ **Relationships** - mudah handle associations
✅ **Security** - prevent SQL injection
✅ **Productivity** - development lebih cepat

---

## Apa itu Sequelize?

**Sequelize** adalah promise-based ORM untuk Node.js

**Support databases:**

- PostgreSQL
- MySQL
- MariaDB
- SQLite
- Microsoft SQL Server

**Features:**

- Models, Migrations, Seeders
- Associations (1:1, 1:N, N:M)
- Validations, Hooks
- Transactions, Raw queries

---

## Install Sequelize

```bash
# Install sequelize dan CLI
npm install sequelize
npm install --save-dev sequelize-cli

# Install database driver (pilih salah satu)
npm install pg pg-hstore          # PostgreSQL
npm install mysql2                # MySQL
npm install mariadb               # MariaDB
npm install sqlite3               # SQLite
npm install tedious              # Microsoft SQL Server
```

---

## Initialize Sequelize

```bash
# Initialize project structure
npx sequelize-cli init
```

**Struktur folder yang dibuat:**

```
project/
├── config/
│   └── config.json       (Database configuration)
├── migrations/           (Database migrations)
├── models/               (Sequelize models)
│   └── index.js
└── seeders/              (Sample data)
```

---

## Database Configuration

```javascript
// config/config.json
{
  "development": {
    "username": "root",
    "password": "password",
    "database": "myapp_dev",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": "password",
    "database": "myapp_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": process.env.DB_USER,
    "password": process.env.DB_PASSWORD,
    "database": process.env.DB_NAME,
    "host": process.env.DB_HOST,
    "dialect": "postgres"
  }
}
```

---

## Connect Sequelize

```javascript
// config/database.js
const { Sequelize } = require("sequelize");

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_HOST,
    dialect: "mysql",
    logging: false, // Disable SQL logging
    pool: {
      max: 5,
      min: 0,
      acquire: 30000,
      idle: 10000,
    },
  },
);

// Test connection
async function testConnection() {
  try {
    await sequelize.authenticate();
    console.log("Database connected successfully");
  } catch (error) {
    console.error("Unable to connect:", error);
  }
}

module.exports = sequelize;
```

---

## Define Model

```javascript
// models/User.js
const { DataTypes } = require("sequelize");
const sequelize = require("../config/database");

const User = sequelize.define(
  "User",
  {
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true,
    },
    name: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
      validate: {
        isEmail: true,
      },
    },
    password: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    role: {
      type: DataTypes.ENUM("user", "admin"),
      defaultValue: "user",
    },
  },
  {
    tableName: "users",
    timestamps: true, // createdAt, updatedAt
  },
);

module.exports = User;
```

---

## Data Types

```javascript
// String types
DataTypes.STRING; // VARCHAR(255)
DataTypes.STRING(100); // VARCHAR(100)
DataTypes.TEXT; // TEXT
DataTypes.TEXT("tiny"); // TINYTEXT

// Number types
DataTypes.INTEGER; // INTEGER
DataTypes.BIGINT; // BIGINT
DataTypes.FLOAT; // FLOAT
DataTypes.DECIMAL(10, 2); // DECIMAL(10,2)

// Boolean
DataTypes.BOOLEAN; // TINYINT(1)

// Date
DataTypes.DATE; // DATETIME
DataTypes.DATEONLY; // DATE

// Others
DataTypes.JSON; // JSON
DataTypes.UUID; // UUID
DataTypes.ENUM("value1", "value2");
```

---

## Sync Models

```javascript
// Sync database schema
async function syncDatabase() {
  try {
    // Create tables (development only!)
    await sequelize.sync({ force: false });
    console.log("Database synced");
  } catch (error) {
    console.error("Sync error:", error);
  }
}

// Options:
// { force: true }  - DROP tables then CREATE
// { force: false } - CREATE if not exists
// { alter: true }  - ALTER tables to match models

// ⚠️ PRODUCTION: Use migrations, JANGAN sync()
```

---

## Create (Insert)

```javascript
// Create single record
const user = await User.create({
  name: "John Doe",
  email: "john@example.com",
  password: "hashedpassword",
});

console.log(user.id); // Auto-generated ID
console.log(user.name); // 'John Doe'

// Create multiple records
const users = await User.bulkCreate([
  { name: "Jane", email: "jane@example.com", password: "pass" },
  { name: "Bob", email: "bob@example.com", password: "pass" },
]);

// Build (without saving)
const user = User.build({ name: "John" });
await user.save(); // Now save
```

---

## Read (Select)

```javascript
// Find all
const users = await User.findAll();

// Find with conditions
const users = await User.findAll({
  where: {
    role: "admin",
  },
});

// Find one
const user = await User.findOne({
  where: { email: "john@example.com" },
});

// Find by primary key
const user = await User.findByPk(1);

// Find or create
const [user, created] = await User.findOrCreate({
  where: { email: "john@example.com" },
  defaults: { name: "John", password: "pass" },
});
```

---

## Update

```javascript
// Update instance
const user = await User.findByPk(1);
user.name = "John Updated";
await user.save();

// Or use update method
await user.update({ name: "John Updated" });

// Update multiple records
await User.update({ role: "admin" }, { where: { email: "john@example.com" } });

// Increment
await User.increment("loginCount", {
  where: { id: 1 },
});

// Decrement
await User.decrement("credits", {
  by: 5,
  where: { id: 1 },
});
```

---

## Delete

```javascript
// Delete instance
const user = await User.findByPk(1);
await user.destroy();

// Delete with condition
await User.destroy({
  where: { id: 1 },
});

// Delete multiple
await User.destroy({
  where: {
    role: "guest",
  },
});

// Delete all (be careful!)
await User.destroy({ truncate: true });
```

---

## Queries - Where Operators

```javascript
const { Op } = require('sequelize');

// Equals
where: { name: 'John' }

// Multiple conditions (AND)
where: { name: 'John', role: 'admin' }

// OR
where: {
  [Op.or]: [
    { name: 'John' },
    { name: 'Jane' }
  ]
}

// Greater than, less than
where: { age: { [Op.gt]: 18 } }       // age > 18
where: { age: { [Op.gte]: 18 } }      // age >= 18
where: { age: { [Op.lt]: 65 } }       // age < 65
where: { age: { [Op.lte]: 65 } }      // age <= 65
where: { age: { [Op.between]: [18, 65] } }
```

---

## Queries - More Operators

```javascript
// IN
where: { id: { [Op.in]: [1, 2, 3] } }

// NOT IN
where: { id: { [Op.notIn]: [1, 2, 3] } }

// LIKE
where: { name: { [Op.like]: '%John%' } }
where: { name: { [Op.startsWith]: 'John' } }
where: { name: { [Op.endsWith]: 'Doe' } }

// NOT
where: { role: { [Op.ne]: 'admin' } }  // not equals

// IS NULL
where: { deletedAt: { [Op.is]: null } }

// IS NOT NULL
where: { email: { [Op.not]: null } }

// Complex
where: {
  [Op.and]: [
    { age: { [Op.gte]: 18 } },
    { role: 'user' }
  ]
}
```

---

## Select Specific Columns

```javascript
// Select specific columns
const users = await User.findAll({
  attributes: ["id", "name", "email"],
});

// Exclude columns
const users = await User.findAll({
  attributes: { exclude: ["password"] },
});

// Alias
const users = await User.findAll({
  attributes: [
    "id",
    ["name", "fullName"], // AS fullName
    [sequelize.fn("COUNT", sequelize.col("posts")), "postCount"],
  ],
});
```

---

## Ordering and Pagination

```javascript
// Order by
const users = await User.findAll({
  order: [
    ["createdAt", "DESC"],
    ["name", "ASC"],
  ],
});

// Limit and Offset (pagination)
const users = await User.findAll({
  limit: 10,
  offset: 20, // Skip 20 records
});

// Pagination helper
const page = 2;
const limit = 10;
const users = await User.findAll({
  limit: limit,
  offset: (page - 1) * limit,
});

// Count and pagination
const { count, rows } = await User.findAndCountAll({
  limit: 10,
  offset: 0,
});
```

---

## Aggregation

```javascript
// Count
const count = await User.count();
const adminCount = await User.count({
  where: { role: "admin" },
});

// Max, Min, Sum
const maxAge = await User.max("age");
const minAge = await User.min("age");
const totalScore = await User.sum("score");

// Group by
const result = await User.findAll({
  attributes: ["role", [sequelize.fn("COUNT", sequelize.col("id")), "count"]],
  group: ["role"],
});
```

---

## Associations - One-to-One

```javascript
// models/User.js
const User = sequelize.define("User", {
  /* ... */
});

// models/Profile.js
const Profile = sequelize.define("Profile", {
  bio: DataTypes.TEXT,
  avatar: DataTypes.STRING,
  userId: {
    type: DataTypes.INTEGER,
    references: {
      model: "users",
      key: "id",
    },
  },
});

// Define association
User.hasOne(Profile, { foreignKey: "userId" });
Profile.belongsTo(User, { foreignKey: "userId" });

module.exports = { User, Profile };
```

---

## Associations - One-to-Many

```javascript
// models/User.js
const User = sequelize.define("User", {
  /* ... */
});

// models/Post.js
const Post = sequelize.define("Post", {
  title: DataTypes.STRING,
  content: DataTypes.TEXT,
  userId: DataTypes.INTEGER,
});

// A user has many posts
User.hasMany(Post, {
  foreignKey: "userId",
  as: "posts",
});

// A post belongs to a user
Post.belongsTo(User, {
  foreignKey: "userId",
  as: "author",
});
```

---

## Associations - Many-to-Many

```javascript
// models/User.js
const User = sequelize.define("User", {
  /* ... */
});

// models/Course.js
const Course = sequelize.define("Course", {
  name: DataTypes.STRING,
});

// Junction table
const UserCourse = sequelize.define("UserCourse", {
  enrolledAt: DataTypes.DATE,
});

// Many-to-many association
User.belongsToMany(Course, {
  through: UserCourse,
  foreignKey: "userId",
});

Course.belongsToMany(User, {
  through: UserCourse,
  foreignKey: "courseId",
});
```

---

## Query with Associations

```javascript
// Include associated data (JOIN)
const users = await User.findAll({
  include: [
    {
      model: Post,
      as: "posts",
    },
  ],
});

// Multiple includes
const users = await User.findAll({
  include: [{ model: Profile }, { model: Post, as: "posts" }],
});

// Nested include
const users = await User.findAll({
  include: [
    {
      model: Post,
      include: [{ model: Comment }],
    },
  ],
});

// Where on association
const users = await User.findAll({
  include: [
    {
      model: Post,
      where: { published: true },
    },
  ],
});
```

---

## Migrations

```bash
# Create migration
npx sequelize-cli migration:generate --name create-users-table
```

```javascript
// migrations/20240101000000-create-users-table.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable("users", {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true,
      },
      createdAt: {
        type: Sequelize.DATE,
        allowNull: false,
      },
      updatedAt: {
        type: Sequelize.DATE,
        allowNull: false,
      },
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable("users");
  },
};
```

---

## Run Migrations

```bash
# Run all pending migrations
npx sequelize-cli db:migrate

# Undo last migration
npx sequelize-cli db:migrate:undo

# Undo all migrations
npx sequelize-cli db:migrate:undo:all

# Migration status
npx sequelize-cli db:migrate:status
```

---

## Add Column Migration

```javascript
// migration: add-role-to-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn("users", "role", {
      type: Sequelize.ENUM("user", "admin"),
      defaultValue: "user",
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn("users", "role");
  },
};
```

---

## Validations

```javascript
const User = sequelize.define("User", {
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true,
      notEmpty: true,
    },
  },
  age: {
    type: DataTypes.INTEGER,
    validate: {
      min: 18,
      max: 120,
    },
  },
  username: {
    type: DataTypes.STRING,
    validate: {
      is: /^[a-z0-9_]+$/i, // Regex
      len: [3, 20], // Length
      notIn: [["admin", "root"]],
    },
  },
  password: {
    type: DataTypes.STRING,
    validate: {
      len: [8, 100],
      isStrongPassword(value) {
        if (!/[A-Z]/.test(value)) {
          throw new Error("Need uppercase");
        }
      },
    },
  },
});
```

---

## Custom Validations

```javascript
const User = sequelize.define(
  "User",
  {
    // ... fields
  },
  {
    validate: {
      // Model-level validation
      bothEmailAndUsername() {
        if (!this.email && !this.username) {
          throw new Error("Either email or username required");
        }
      },
    },
  },
);

// Usage
try {
  await User.create({ name: "John" });
} catch (error) {
  console.error(error.errors); // Validation errors
}
```

---

## Hooks (Lifecycle Events)

```javascript
const User = sequelize.define(
  "User",
  {
    // ... fields
  },
  {
    hooks: {
      // Before create
      beforeCreate: async (user) => {
        user.email = user.email.toLowerCase();
      },

      // After create
      afterCreate: async (user) => {
        console.log("User created:", user.id);
        await sendWelcomeEmail(user.email);
      },

      // Before update
      beforeUpdate: async (user) => {
        if (user.changed("password")) {
          user.password = await bcrypt.hash(user.password, 10);
        }
      },

      // Before destroy
      beforeDestroy: async (user) => {
        await user
          .getPosts()
          .then((posts) => posts.forEach((post) => post.destroy()));
      },
    },
  },
);
```

---

## Available Hooks

```javascript
// Create hooks
(beforeCreate, afterCreate);
(beforeBulkCreate, afterBulkCreate);

// Update hooks
(beforeUpdate, afterUpdate);
(beforeBulkUpdate, afterBulkUpdate);

// Destroy hooks
(beforeDestroy, afterDestroy);
(beforeBulkDestroy, afterBulkDestroy);

// Find hooks
(beforeFind, afterFind);

// Save hooks (create or update)
(beforeSave, afterSave);

// Validate hooks
(beforeValidate, afterValidate);
```

---

## Transactions

**Transaction** = group of operations, all or nothing

```javascript
const { sequelize } = require("./models");

// Managed transaction
try {
  const result = await sequelize.transaction(async (t) => {
    // Create user
    const user = await User.create(
      {
        name: "John",
        email: "john@example.com",
      },
      { transaction: t },
    );

    // Create profile
    await Profile.create(
      {
        userId: user.id,
        bio: "Hello",
      },
      { transaction: t },
    );

    return user;
  });

  console.log("Transaction successful");
} catch (error) {
  console.log("Transaction rolled back");
}
```

---

## Unmanaged Transaction

```javascript
const t = await sequelize.transaction();

try {
  const user = await User.create(
    {
      name: "John",
    },
    { transaction: t },
  );

  await Profile.create(
    {
      userId: user.id,
    },
    { transaction: t },
  );

  // Commit transaction
  await t.commit();
} catch (error) {
  // Rollback transaction
  await t.rollback();
}
```

---

## Scopes

**Scopes** = predefined query configurations

```javascript
const User = sequelize.define(
  "User",
  {
    // ... fields
  },
  {
    scopes: {
      // Active users only
      active: {
        where: { isActive: true },
      },

      // Admin users
      admin: {
        where: { role: "admin" },
      },

      // With posts
      withPosts: {
        include: [{ model: Post }],
      },

      // Exclude password
      safe: {
        attributes: { exclude: ["password"] },
      },
    },
  },
);

// Usage
const activeUsers = await User.scope("active").findAll();
const admins = await User.scope(["active", "admin"]).findAll();
```

---

## Default Scope

```javascript
const User = sequelize.define(
  "User",
  {
    // ... fields
  },
  {
    defaultScope: {
      attributes: { exclude: ["password"] },
    },
    scopes: {
      withPassword: {
        attributes: { include: ["password"] },
      },
    },
  },
);

// Uses default scope (no password)
const users = await User.findAll();

// Override default scope
const usersWithPassword = await User.scope("withPassword").findAll();

// Remove all scopes
const allUsers = await User.unscoped().findAll();
```

---

## Raw Queries

```javascript
// Execute raw SQL
const [results, metadata] = await sequelize.query(
  "SELECT * FROM users WHERE role = ?",
  {
    replacements: ["admin"],
    type: sequelize.QueryTypes.SELECT,
  },
);

// Named replacements
const users = await sequelize.query("SELECT * FROM users WHERE age > :minAge", {
  replacements: { minAge: 18 },
  type: sequelize.QueryTypes.SELECT,
});

// Model instance
const users = await sequelize.query("SELECT * FROM users", {
  model: User,
  mapToModel: true,
});
```

---

## Seeders

```bash
# Create seeder
npx sequelize-cli seed:generate --name demo-users
```

```javascript
// seeders/20240101-demo-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.bulkInsert("users", [
      {
        name: "John Doe",
        email: "john@example.com",
        password: "hashedpass",
        createdAt: new Date(),
        updatedAt: new Date(),
      },
      {
        name: "Jane Doe",
        email: "jane@example.com",
        password: "hashedpass",
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    ]);
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.bulkDelete("users", null, {});
  },
};
```

---

## Run Seeders

```bash
# Run all seeders
npx sequelize-cli db:seed:all

# Run specific seeder
npx sequelize-cli db:seed --seed 20240101-demo-users.js

# Undo last seeder
npx sequelize-cli db:seed:undo

# Undo all seeders
npx sequelize-cli db:seed:undo:all
```

---

## Indexes

```javascript
const User = sequelize.define(
  "User",
  {
    email: DataTypes.STRING,
    username: DataTypes.STRING,
  },
  {
    indexes: [
      // Unique index
      {
        unique: true,
        fields: ["email"],
      },
      // Regular index
      {
        fields: ["username"],
      },
      // Composite index
      {
        fields: ["firstName", "lastName"],
      },
      // Named index
      {
        name: "user_role_index",
        fields: ["role"],
      },
    ],
  },
);
```

---

## Getters and Setters

```javascript
const User = sequelize.define("User", {
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING,

  // Virtual field
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
      const names = value.split(" ");
      this.setDataValue("firstName", names[0]);
      this.setDataValue("lastName", names[1]);
    },
  },

  email: {
    type: DataTypes.STRING,
    get() {
      return this.getDataValue("email").toLowerCase();
    },
    set(value) {
      this.setDataValue("email", value.toLowerCase());
    },
  },
});

// Usage
user.fullName = "John Doe";
console.log(user.firstName); // 'John'
```

---

## Instance Methods

```javascript
const User = sequelize.define("User", {
  password: DataTypes.STRING,
});

// Instance method
User.prototype.validatePassword = async function (password) {
  return await bcrypt.compare(password, this.password);
};

// Usage
const user = await User.findOne({ where: { email: "john@example.com" } });
const isValid = await user.validatePassword("password123");
```

---

## Class Methods

```javascript
const User = sequelize.define("User", {
  // ... fields
});

// Class method
User.findByEmail = async function (email) {
  return await this.findOne({
    where: { email: email.toLowerCase() },
  });
};

User.countAdmins = async function () {
  return await this.count({ where: { role: "admin" } });
};

// Usage
const user = await User.findByEmail("john@example.com");
const adminCount = await User.countAdmins();
```

---

## Soft Delete

```javascript
const User = sequelize.define(
  "User",
  {
    // ... fields
  },
  {
    paranoid: true, // Enable soft delete
  },
);

// Creates deletedAt column automatically

// Soft delete (sets deletedAt)
await user.destroy();

// Find only non-deleted
const users = await User.findAll();

// Find including deleted
const allUsers = await User.findAll({ paranoid: false });

// Restore deleted record
await user.restore();

// Permanent delete
await user.destroy({ force: true });
```

---

## Use with Express

```javascript
// controllers/userController.js
const User = require("../models/User");

class UserController {
  static async index(req, res) {
    try {
      const users = await User.findAll({
        attributes: { exclude: ["password"] },
      });
      res.json({ users });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  static async show(req, res) {
    try {
      const user = await User.findByPk(req.params.id, {
        include: [{ model: Post, as: "posts" }],
      });

      if (!user) {
        return res.status(404).json({ error: "User not found" });
      }

      res.json({ user });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
}

module.exports = UserController;
```

---

## Create with Validation

```javascript
static async create(req, res) {
  try {
    const { name, email, password } = req.body;

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Create user
    const user = await User.create({
      name,
      email,
      password: hashedPassword
    });

    // Return without password
    const { password: _, ...userWithoutPassword } = user.toJSON();

    res.status(201).json({
      message: 'User created',
      user: userWithoutPassword
    });
  } catch (error) {
    if (error.name === 'SequelizeValidationError') {
      return res.status(400).json({
        errors: error.errors.map(e => e.message)
      });
    }
    res.status(500).json({ error: error.message });
  }
}
```

---

## Best Practices

1. **Use migrations** - jangan sync() di production
2. **Validate data** - gunakan validations di model
3. **Exclude sensitive fields** - jangan expose password
4. **Use transactions** - untuk multiple operations
5. **Index frequently queried fields**
6. **Use scopes** - untuk reusable queries
7. **Handle errors properly** - validation, unique constraint
8. **Use connection pooling**
9. **Paginate results** - jangan return semua data

---

## Performance Tips

```javascript
// ❌ BAD - N+1 query problem
const users = await User.findAll();
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } });
}

// ✅ GOOD - Use include (JOIN)
const users = await User.findAll({
  include: [{ model: Post, as: "posts" }],
});

// ✅ Select only needed columns
const users = await User.findAll({
  attributes: ["id", "name", "email"],
});

// ✅ Use pagination
const users = await User.findAll({
  limit: 20,
  offset: 0,
});
```

---

## Error Handling

```javascript
try {
  await User.create({ email: "duplicate@example.com" });
} catch (error) {
  if (error.name === "SequelizeUniqueConstraintError") {
    console.log("Email already exists");
  } else if (error.name === "SequelizeValidationError") {
    console.log("Validation failed:", error.errors);
  } else if (error.name === "SequelizeForeignKeyConstraintError") {
    console.log("Foreign key violation");
  } else {
    console.log("Database error:", error.message);
  }
}
```

---

## Testing with Sequelize

```javascript
// tests/models/User.test.js
const { sequelize, User } = require("../../models");

beforeAll(async () => {
  await sequelize.sync({ force: true });
});

afterEach(async () => {
  await User.destroy({ truncate: true });
});

afterAll(async () => {
  await sequelize.close();
});

describe("User Model", () => {
  test("should create user", async () => {
    const user = await User.create({
      name: "John",
      email: "john@example.com",
      password: "password",
    });

    expect(user.id).toBeDefined();
    expect(user.email).toBe("john@example.com");
  });
});
```

---

## Environment-based Config

```javascript
// config/config.js
require("dotenv").config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: "mysql",
    logging: console.log,
  },
  test: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_TEST_NAME,
    host: process.env.DB_HOST,
    dialect: "mysql",
    logging: false,
  },
  production: {
    use_env_variable: "DATABASE_URL",
    dialect: "postgres",
    logging: false,
    dialectOptions: {
      ssl: {
        require: true,
        rejectUnauthorized: false,
      },
    },
  },
};
```

---

## Kesalahan Umum

❌ Menggunakan `sync({ force: true })` di production
❌ Tidak handle validation errors
❌ Expose password di response
❌ N+1 query problem
❌ Tidak menggunakan transactions untuk multiple operations
❌ Hardcode database credentials
❌ Tidak use indexes untuk frequently queried fields
❌ Tidak pagination untuk large datasets

---

## Sequelize vs Raw SQL

**Sequelize:**
✅ Type safety
✅ Database agnostic
✅ Easy maintenance
❌ Learning curve
❌ Abstraction overhead

**Raw SQL:**
✅ Full control
✅ Better performance (complex queries)
✅ No abstraction
❌ Database specific
❌ More code
❌ SQL injection risk

**Best:** Kombinasi - Sequelize untuk CRUD, Raw SQL untuk complex queries

---

## Resources & Referensi

- [Sequelize Documentation](https://sequelize.org/)
- [Sequelize API Reference](https://sequelize.org/api/)
- [Sequelize CLI](https://github.com/sequelize/cli)
- [Sequelize Best Practices](https://sequelize.org/docs/v6/other-topics/optimistic-locking/)
- [Database Design Guide](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-database-design/)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Happy ORMing! 🗄️
