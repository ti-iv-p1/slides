---
title: Authorization di Node.js
version: 1.0.0
header: Authorization di Node.js
footer: https://github.com/ti-iv-p1/slides
paginate: true
marp: true
---

<!--
_class: lead
_paginate: skip
-->

# Authorization di Node.js

---

## Tujuan Pembelajaran

- Memahami konsep authorization dan perbedaannya dengan authentication
- Menguasai Role-Based Access Control (RBAC)
- Mengimplementasikan Permission-Based Authorization
- Memahami Access Control List (ACL)
- Menerapkan Resource-Based Authorization
- Menggunakan middleware untuk authorization

---

## Authentication vs Authorization

**Authentication** 🔐

- Memverifikasi **identitas** user
- "Siapa kamu?"
- Login dengan username/password, JWT, OAuth

**Authorization** 🔑

- Menentukan **hak akses** user
- "Apa yang boleh kamu lakukan?"
- Role, permissions, policies

**Flow:** Authentication → Authorization → Access

---

## Contoh Kasus

```
User: john@example.com
Role: Editor

✅ Dapat membaca artikel (authorized)
✅ Dapat membuat artikel (authorized)
✅ Dapat edit artikel sendiri (authorized)
❌ Tidak dapat delete artikel orang lain (not authorized)
❌ Tidak dapat manage users (not authorized - butuh Admin role)
```

---

## Jenis-jenis Authorization

1. **Role-Based Access Control (RBAC)**
   - User punya role (admin, editor, user)
   - Role menentukan akses

2. **Permission-Based**
   - User/Role punya specific permissions
   - Lebih granular dari role

3. **Attribute-Based Access Control (ABAC)**
   - Berdasarkan attributes (departemen, lokasi, dll)

4. **Resource-Based**
   - User hanya bisa akses resource miliknya

---

## Role-Based Access Control (RBAC)

**Konsep:** User diberi role, role menentukan akses

```javascript
// User model
const user = {
  id: 1,
  email: "john@example.com",
  role: "editor", // 'admin', 'editor', 'user'
};

// Roles hierarchy
admin > editor > user;
```

---

## Implementasi RBAC - Database Schema

```sql
-- Users table
CREATE TABLE users (
  id INT PRIMARY KEY,
  email VARCHAR(255),
  password VARCHAR(255),
  role VARCHAR(50) DEFAULT 'user'
);

-- Roles
-- 'admin' - full access
-- 'editor' - can create/edit content
-- 'moderator' - can review content
-- 'user' - read only
```

---

## Authorization Middleware - Single Role

```javascript
// middleware/authorize.js

const authorize = (role) => {
  return (req, res, next) => {
    // Assume req.user sudah ada dari auth middleware
    if (req.user.role !== role) {
      return res.status(403).json({
        error: "Forbidden - Insufficient permissions",
      });
    }
    next();
  };
};

module.exports = { authorize };
```

---

## Menggunakan Single Role Authorization

```javascript
const { authenticate } = require("./middleware/auth");
const { authorize } = require("./middleware/authorize");

// Only admin can access
app.delete(
  "/api/users/:id",
  authenticate,
  authorize("admin"),
  UserController.delete,
);

// Only editor can access
app.post(
  "/api/posts",
  authenticate,
  authorize("editor"),
  PostController.create,
);
```

---

## Authorization Middleware - Multiple Roles

```javascript
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: "Unauthorized" });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: "Forbidden - Insufficient permissions",
        requiredRoles: allowedRoles,
        yourRole: req.user.role,
      });
    }

    next();
  };
};
```

---

## Menggunakan Multiple Roles

```javascript
// Admin atau Editor
app.put(
  "/api/posts/:id",
  authenticate,
  authorize("admin", "editor"),
  PostController.update,
);

// Admin, Editor, atau Moderator
app.get(
  "/api/posts/pending",
  authenticate,
  authorize("admin", "editor", "moderator"),
  PostController.getPending,
);
```

---

## Role Hierarchy

```javascript
const roleHierarchy = {
  admin: 4,
  editor: 3,
  moderator: 2,
  user: 1,
};

const authorizeMinRole = (minRole) => {
  return (req, res, next) => {
    const userLevel = roleHierarchy[req.user.role] || 0;
    const minLevel = roleHierarchy[minRole];

    if (userLevel < minLevel) {
      return res.status(403).json({
        error: "Insufficient permissions",
      });
    }

    next();
  };
};

// Usage: butuh minimal editor
app.post(
  "/api/posts",
  authenticate,
  authorizeMinRole("editor"),
  PostController.create,
);
```

---

## Permission-Based Authorization

**Lebih granular dari role**

```javascript
// User model dengan permissions
const user = {
  id: 1,
  email: "john@example.com",
  role: "editor",
  permissions: [
    "post:create",
    "post:read",
    "post:update",
    "post:delete",
    "comment:read",
  ],
};
```

---

## Database Schema untuk Permissions

```sql
-- Permissions table
CREATE TABLE permissions (
  id INT PRIMARY KEY,
  name VARCHAR(100) UNIQUE,
  description TEXT
);

-- Role_Permissions junction table
CREATE TABLE role_permissions (
  role_id INT,
  permission_id INT,
  PRIMARY KEY (role_id, permission_id)
);

-- User_Permissions (optional - untuk custom permissions)
CREATE TABLE user_permissions (
  user_id INT,
  permission_id INT,
  PRIMARY KEY (user_id, permission_id)
);
```

---

## Permission Middleware

```javascript
const hasPermission = (...requiredPermissions) => {
  return async (req, res, next) => {
    try {
      // Get user dengan permissions
      const user = await User.findById(req.userId).populate("permissions");

      const userPermissions = user.permissions.map((p) => p.name);

      // Check if user has all required permissions
      const hasAllPermissions = requiredPermissions.every((permission) =>
        userPermissions.includes(permission),
      );

      if (!hasAllPermissions) {
        return res.status(403).json({
          error: "Missing required permissions",
          required: requiredPermissions,
          yours: userPermissions,
        });
      }

      next();
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  };
};
```

---

## Menggunakan Permission-Based Authorization

```javascript
// Butuh permission 'post:delete'
app.delete(
  "/api/posts/:id",
  authenticate,
  hasPermission("post:delete"),
  PostController.delete,
);

// Butuh multiple permissions
app.post(
  "/api/posts/:id/publish",
  authenticate,
  hasPermission("post:update", "post:publish"),
  PostController.publish,
);

// Create user butuh admin permissions
app.post(
  "/api/users",
  authenticate,
  hasPermission("user:create"),
  UserController.create,
);
```

---

## Resource-Based Authorization

**User hanya bisa akses/edit resource miliknya**

```javascript
const isOwner = (resourceModel) => {
  return async (req, res, next) => {
    try {
      const resourceId = req.params.id;
      const resource = await resourceModel.findById(resourceId);

      if (!resource) {
        return res.status(404).json({ error: "Resource not found" });
      }

      // Check ownership
      if (resource.userId.toString() !== req.userId.toString()) {
        return res.status(403).json({
          error: "You can only access your own resources",
        });
      }

      // Attach resource ke request untuk digunakan di controller
      req.resource = resource;
      next();
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  };
};
```

---

## Menggunakan Resource-Based Authorization

```javascript
const Post = require("../models/Post");

// User hanya bisa update post miliknya
app.put("/api/posts/:id", authenticate, isOwner(Post), PostController.update);

// User hanya bisa delete post miliknya
app.delete(
  "/api/posts/:id",
  authenticate,
  isOwner(Post),
  PostController.delete,
);

// Admin bisa access semua
app.delete(
  "/api/posts/:id",
  authenticate,
  authorize("admin"), // Skip ownership check untuk admin
  PostController.delete,
);
```

---

## Kombinasi Role dan Ownership

```javascript
const authorizeOwnerOrRole = (resourceModel, ...roles) => {
  return async (req, res, next) => {
    try {
      // Admin bypass ownership check
      if (roles.includes(req.user.role)) {
        return next();
      }

      // Check ownership
      const resource = await resourceModel.findById(req.params.id);
      if (!resource) {
        return res.status(404).json({ error: "Not found" });
      }

      if (resource.userId.toString() !== req.userId.toString()) {
        return res.status(403).json({
          error: "Access denied",
        });
      }

      req.resource = resource;
      next();
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  };
};
```

---

## Contoh Penggunaan

```javascript
// User bisa edit post sendiri, atau Admin bisa edit semua
app.put(
  "/api/posts/:id",
  authenticate,
  authorizeOwnerOrRole(Post, "admin", "moderator"),
  PostController.update,
);
```

**Flow:**

- Jika user role = 'admin' → allow
- Jika bukan admin → check ownership
- Jika owner → allow
- Jika bukan owner → 403 Forbidden

---

## Access Control List (ACL)

**ACL** menentukan aksi spesifik untuk resource spesifik

```javascript
const acl = {
  admin: {
    post: ["create", "read", "update", "delete"],
    user: ["create", "read", "update", "delete"],
    comment: ["create", "read", "update", "delete"],
  },
  editor: {
    post: ["create", "read", "update"],
    comment: ["create", "read", "update", "delete"],
  },
  user: {
    post: ["read"],
    comment: ["create", "read"],
  },
};
```

---

## ACL Middleware

```javascript
const checkACL = (resource, action) => {
  return (req, res, next) => {
    const role = req.user.role;

    // Check if role exists in ACL
    if (!acl[role]) {
      return res.status(403).json({ error: "Role not found" });
    }

    // Check if role has access to resource
    if (!acl[role][resource]) {
      return res.status(403).json({
        error: `No access to ${resource}`,
      });
    }

    // Check if role can perform action
    if (!acl[role][resource].includes(action)) {
      return res.status(403).json({
        error: `Cannot ${action} ${resource}`,
      });
    }

    next();
  };
};
```

---

## Menggunakan ACL

```javascript
// Check if user can create post
app.post(
  "/api/posts",
  authenticate,
  checkACL("post", "create"),
  PostController.create,
);

// Check if user can delete comment
app.delete(
  "/api/comments/:id",
  authenticate,
  checkACL("comment", "delete"),
  CommentController.delete,
);

// Check if user can update user
app.put(
  "/api/users/:id",
  authenticate,
  checkACL("user", "update"),
  UserController.update,
);
```

---

## Library: node-acl

```bash
npm install acl
```

```javascript
const ACL = require("acl");
const acl = new ACL(new ACL.memoryBackend());

// Define roles and permissions
acl.allow([
  {
    roles: "admin",
    allows: [
      { resources: "posts", permissions: "*" },
      { resources: "users", permissions: "*" },
    ],
  },
  {
    roles: "editor",
    allows: [{ resources: "posts", permissions: ["create", "read", "update"] }],
  },
  {
    roles: "user",
    allows: [{ resources: "posts", permissions: ["read"] }],
  },
]);
```

---

## Menggunakan node-acl

```javascript
// Middleware
const checkPermission = (resource, action) => {
  return (req, res, next) => {
    acl.isAllowed(req.user.role, resource, action, (err, allowed) => {
      if (err) {
        return res.status(500).json({ error: err.message });
      }

      if (!allowed) {
        return res.status(403).json({ error: "Access denied" });
      }

      next();
    });
  };
};

// Usage
app.post(
  "/api/posts",
  authenticate,
  checkPermission("posts", "create"),
  PostController.create,
);
```

---

## Attribute-Based Access Control (ABAC)

**Authorization based on attributes**

```javascript
const user = {
  id: 1,
  email: "john@example.com",
  department: "engineering",
  location: "jakarta",
  level: "senior",
};

const resource = {
  id: 100,
  type: "document",
  department: "engineering",
  classification: "internal",
};

// Policy: User can access document if same department
function canAccess(user, resource) {
  return user.department === resource.department;
}
```

---

## ABAC Middleware

```javascript
const authorizeByAttribute = (policy) => {
  return async (req, res, next) => {
    try {
      const user = req.user;
      const resource = await getResource(req.params.id);

      if (!policy(user, resource)) {
        return res.status(403).json({
          error: "Access denied by policy",
        });
      }

      req.resource = resource;
      next();
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  };
};

// Policy function
const sameDepartmentPolicy = (user, resource) => {
  return user.department === resource.department;
};

// Usage
app.get(
  "/api/documents/:id",
  authenticate,
  authorizeByAttribute(sameDepartmentPolicy),
  DocumentController.show,
);
```

---

## Complex Authorization Policy

```javascript
const complexPolicy = (user, resource, action) => {
  // Admin can do anything
  if (user.role === "admin") {
    return true;
  }

  // Owner can do anything with their resource
  if (resource.ownerId === user.id) {
    return true;
  }

  // Same department can read
  if (action === "read" && user.department === resource.department) {
    return true;
  }

  // Senior level can edit in their department
  if (
    action === "update" &&
    user.level === "senior" &&
    user.department === resource.department
  ) {
    return true;
  }

  return false;
};
```

---

## Field-Level Authorization

**Control access to specific fields**

```javascript
const filterUserFields = (req, res, next) => {
  const role = req.user.role;

  // Define field access per role
  const allowedFields = {
    admin: ["id", "email", "name", "password", "role", "salary"],
    manager: ["id", "email", "name", "role"],
    user: ["id", "email", "name"],
  };

  req.allowedFields = allowedFields[role] || [];
  next();
};

// Controller
async function getUser(req, res) {
  const user = await User.findById(req.params.id);

  // Filter based on allowed fields
  const filteredUser = {};
  req.allowedFields.forEach((field) => {
    if (user[field] !== undefined) {
      filteredUser[field] = user[field];
    }
  });

  res.json({ user: filteredUser });
}
```

---

## Dynamic Permissions

```javascript
// User can have dynamic permissions based on time/context
const checkDynamicPermission = async (req, res, next) => {
  const user = req.user;
  const now = new Date();

  // Check if user has temporary elevated permissions
  const tempPermission = await TempPermission.findOne({
    userId: user.id,
    expiresAt: { $gt: now },
  });

  if (tempPermission) {
    req.user.temporaryPermissions = tempPermission.permissions;
  }

  next();
};
```

---

## Rate Limiting per Role

```javascript
const rateLimit = require("express-rate-limit");

const createRateLimiter = (maxRequests) => {
  return rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: maxRequests,
    message: "Too many requests",
  });
};

const rateLimitByRole = (req, res, next) => {
  const limits = {
    admin: createRateLimiter(1000),
    editor: createRateLimiter(500),
    user: createRateLimiter(100),
  };

  const limiter = limits[req.user.role] || limits.user;
  limiter(req, res, next);
};

app.use("/api", authenticate, rateLimitByRole);
```

---

## Authorization Logger

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  transports: [new winston.transports.File({ filename: "authorization.log" })],
});

const logAuthorization = (req, res, next) => {
  const originalSend = res.send;

  res.send = function (data) {
    logger.info({
      user: req.user?.email,
      role: req.user?.role,
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      authorized: res.statusCode !== 403,
      timestamp: new Date(),
    });

    originalSend.call(this, data);
  };

  next();
};

app.use(authenticate, logAuthorization);
```

---

## Testing Authorization

```javascript
const request = require("supertest");
const app = require("../app");

describe("Authorization", () => {
  let adminToken, userToken;

  beforeAll(async () => {
    adminToken = await getTokenForRole("admin");
    userToken = await getTokenForRole("user");
  });

  test("Admin can delete user", async () => {
    const res = await request(app)
      .delete("/api/users/123")
      .set("Authorization", `Bearer ${adminToken}`);

    expect(res.statusCode).toBe(200);
  });

  test("Regular user cannot delete user", async () => {
    const res = await request(app)
      .delete("/api/users/123")
      .set("Authorization", `Bearer ${userToken}`);

    expect(res.statusCode).toBe(403);
  });
});
```

---

## Error Messages

```javascript
// ❌ JANGAN expose terlalu banyak detail
res.status(403).json({
  error: "You need admin role to access this",
});

// ✅ Generic message
res.status(403).json({
  error: "Forbidden",
});

// ✅ Atau sedikit lebih detail (development mode)
if (process.env.NODE_ENV === "development") {
  res.status(403).json({
    error: "Forbidden",
    required: ["admin"],
    current: req.user.role,
  });
} else {
  res.status(403).json({ error: "Forbidden" });
}
```

---

## Best Practices

1. **Principle of Least Privilege** - beri akses minimal yang dibutuhkan
2. **Default Deny** - deny by default, allow explicitly
3. **Centralized Authorization** - gunakan middleware terpusat
4. **Audit Logging** - log semua authorization decisions
5. **Regular Review** - review permissions secara berkala
6. **Separation of Concerns** - pisahkan authentication & authorization
7. **Test Thoroughly** - test semua permission scenarios

---

## Struktur Folder yang Baik

```
project/
├── middleware/
│   ├── authenticate.js     (who are you?)
│   ├── authorize.js        (what can you do?)
│   └── permissions.js      (check permissions)
├── models/
│   ├── User.js
│   ├── Role.js
│   └── Permission.js
├── policies/
│   ├── postPolicy.js
│   └── userPolicy.js
├── services/
│   └── authorizationService.js
└── config/
    ├── roles.js
    └── permissions.js
```

---

## Authorization Service

```javascript
// services/authorizationService.js

class AuthorizationService {
  static async canUser(userId, action, resource) {
    const user = await User.findById(userId)
      .populate("role")
      .populate("permissions");

    // Check role permissions
    const rolePermissions = user.role.permissions;
    if (rolePermissions.includes(`${resource}:${action}`)) {
      return true;
    }

    // Check user specific permissions
    const userPermissions = user.permissions.map((p) => p.name);
    if (userPermissions.includes(`${resource}:${action}`)) {
      return true;
    }

    return false;
  }

  static async canAccessResource(userId, resourceId, action) {
    const resource = await Resource.findById(resourceId);
    const user = await User.findById(userId);

    // Admin can do anything
    if (user.role === "admin") return true;

    // Owner can do anything with their resource
    if (resource.ownerId.equals(userId)) return true;

    // Check specific permission
    return await this.canUser(userId, action, "resource");
  }
}

module.exports = AuthorizationService;
```

---

## Policy Pattern

```javascript
// policies/postPolicy.js

class PostPolicy {
  static canCreate(user) {
    return ["admin", "editor"].includes(user.role);
  }

  static canUpdate(user, post) {
    // Admin can update any post
    if (user.role === "admin") return true;

    // Owner can update their post
    if (post.authorId.equals(user.id)) return true;

    return false;
  }

  static canDelete(user, post) {
    // Only admin or owner can delete
    return user.role === "admin" || post.authorId.equals(user.id);
  }

  static canPublish(user) {
    return ["admin", "editor"].includes(user.role);
  }
}

module.exports = PostPolicy;
```

---

## Menggunakan Policy

```javascript
const PostPolicy = require("../policies/postPolicy");

// Controller
class PostController {
  static async update(req, res) {
    try {
      const post = await Post.findById(req.params.id);

      // Check authorization
      if (!PostPolicy.canUpdate(req.user, post)) {
        return res.status(403).json({ error: "Forbidden" });
      }

      // Update post
      await post.update(req.body);
      res.json({ message: "Post updated", post });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
}
```

---

## Kesalahan Umum

❌ Melakukan authorization di frontend saja
❌ Lupa check ownership untuk resource-based auth
❌ Authorization logic tersebar di banyak tempat
❌ Tidak log authorization failures
❌ Expose terlalu banyak info di error message
❌ Hardcode roles/permissions di code
❌ Tidak test authorization scenarios
❌ Mixing authentication & authorization logic

---

## Frontend Authorization

```javascript
// Backend mengirim user dengan permissions
res.json({
  user: {
    id: 1,
    email: "user@example.com",
    role: "editor",
    permissions: ["post:create", "post:update"],
  },
});

// Frontend hide/show UI based on permissions
// TAPI tetap validate di backend!
if (user.permissions.includes("post:create")) {
  // Show create button
}
```

**PENTING:** Frontend authorization hanya untuk UX,
backend HARUS selalu validate!

---

## Caching Authorization

```javascript
const NodeCache = require('node-cache');
const authCache = new NodeCache({ stdTTL: 300 }); // 5 minutes

const checkPermissionCached = (userId, resource, action) => {
  const cacheKey = `${userId}:${resource}:${action}`;

  // Check cache
  const cached = authCache.get(cacheKey);
  if (cached !== undefined) {
    return cached;
  }

  // Check permission
  const hasPermission = await checkPermission(userId, resource, action);

  // Store in cache
  authCache.set(cacheKey, hasPermission);

  return hasPermission;
};
```

---

## Resources & Referensi

- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [node-acl Documentation](https://www.npmjs.com/package/acl)
- [RBAC vs ABAC](https://www.okta.com/identity-101/role-based-access-control-vs-attribute-based-access-control/)
- [Casbin - Authorization Library](https://casbin.org/)
- [Authorization Patterns](https://auth0.com/docs/authorization)

---

<!--
_class: lead
_paginate: skip
-->

# Terima Kasih

## Control Your Access! 🔑
