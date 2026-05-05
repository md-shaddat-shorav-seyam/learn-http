# 🚀 HTTP in Real Development — Express.js

## Setup

```bash
npm init -y
npm install express cookie-parser cors jsonwebtoken bcryptjs express-rate-limit
```

```javascript
// server.js
const express = require('express');
const app = express();

app.use(express.json());         // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse form data

app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## 📁 Project Structure

```
project/
├── server.js
├── routes/
│   ├── auth.js
│   ├── users.js
│   └── posts.js
├── middleware/
│   ├── auth.js
│   ├── errorHandler.js
│   └── rateLimiter.js
└── utils/
    └── response.js
```

---

## 🔧 HTTP Methods in Express

### GET — Fetch Resources

```javascript
const express = require('express');
const router = express.Router();

const users = [
  { id: 1, name: 'Seyam', email: 'seyam@example.com' },
  { id: 2, name: 'Alice', email: 'alice@example.com' },
];

// GET all users (with query params)
// GET /users?page=1&limit=10&sort=name
router.get('/users', (req, res) => {
  const { page = 1, limit = 10, sort = 'id' } = req.query;

  let result = [...users];

  // Sort
  result.sort((a, b) => (a[sort] > b[sort] ? 1 : -1));

  // Paginate
  const start = (page - 1) * limit;
  const paginated = result.slice(start, start + Number(limit));

  res.status(200).json({
    data: paginated,
    meta: { page: Number(page), limit: Number(limit), total: users.length },
  });
});

// GET single user by ID
// GET /users/1
router.get('/users/:id', (req, res) => {
  const user = users.find((u) => u.id === Number(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.status(200).json({ data: user });
});
```

---

### POST — Create Resource

```javascript
// POST /users
router.post('/users', (req, res) => {
  const { name, email } = req.body;

  // Validate
  if (!name || !email) {
    return res.status(400).json({
      error: 'Bad Request',
      message: 'name and email are required',
    });
  }

  // Check duplicate
  const exists = users.find((u) => u.email === email);
  if (exists) {
    return res.status(409).json({
      error: 'Conflict',
      message: 'Email already registered',
    });
  }

  const newUser = { id: users.length + 1, name, email };
  users.push(newUser);

  // 201 Created + Location header pointing to new resource
  res
    .status(201)
    .location(`/users/${newUser.id}`)
    .json({ data: newUser });
});
```

---

### PUT — Replace Resource Entirely

```javascript
// PUT /users/1  (replaces ALL fields)
router.put('/users/:id', (req, res) => {
  const index = users.findIndex((u) => u.id === Number(req.params.id));

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  const { name, email } = req.body;

  if (!name || !email) {
    return res.status(400).json({ error: 'PUT requires all fields' });
  }

  // Full replacement
  users[index] = { id: users[index].id, name, email };

  res.status(200).json({ data: users[index] });
});
```

---

### PATCH — Partial Update

```javascript
// PATCH /users/1  (updates only provided fields)
router.patch('/users/:id', (req, res) => {
  const index = users.findIndex((u) => u.id === Number(req.params.id));

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  // Merge only what was sent
  users[index] = { ...users[index], ...req.body };

  res.status(200).json({ data: users[index] });
});
```

---

### DELETE — Remove Resource

```javascript
// DELETE /users/1
router.delete('/users/:id', (req, res) => {
  const index = users.findIndex((u) => u.id === Number(req.params.id));

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  users.splice(index, 1);

  // 204 No Content — success with no body
  res.status(204).send();
});
```

---

### HEAD — Check Resource Existence

```javascript
// HEAD /users/1  (same as GET but no body — used for existence checks)
router.head('/users/:id', (req, res) => {
  const user = users.find((u) => u.id === Number(req.params.id));

  if (!user) return res.status(404).send();

  res
    .status(200)
    .set('X-Resource-Exists', 'true')
    .set('Content-Type', 'application/json')
    .send();
});
```

---

### OPTIONS — Advertise Allowed Methods

```javascript
// OPTIONS /users
router.options('/users', (req, res) => {
  res
    .status(204)
    .set('Allow', 'GET, POST, OPTIONS')
    .set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
    .send();
});
```

---

## 📋 Headers in Express

### Setting Response Headers

```javascript
router.get('/headers-demo', (req, res) => {
  res
    // Single header
    .set('X-Custom-Header', 'MyValue')

    // Multiple at once
    .set({
      'Content-Type': 'application/json',
      'X-Powered-By': 'Express',
      'X-Request-Id': crypto.randomUUID(),
      'Cache-Control': 'no-store',
    })

    .status(200)
    .json({ message: 'Headers set!' });
});
```

### Reading Request Headers

```javascript
router.get('/read-headers', (req, res) => {
  const userAgent    = req.get('User-Agent');
  const contentType  = req.get('Content-Type');
  const authHeader   = req.get('Authorization');
  const acceptLang   = req.get('Accept-Language');
  const customHeader = req.get('X-Custom-Header');

  console.log({ userAgent, contentType, authHeader, acceptLang });

  res.json({ received: { userAgent, contentType, acceptLang } });
});
```

---

## 🔐 Authentication Scenarios

### Scenario 1 — JWT Auth (Bearer Token)

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const SECRET = process.env.JWT_SECRET || 'supersecret';

// Middleware to protect routes
function authenticate(req, res, next) {
  const authHeader = req.get('Authorization');

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Authorization header with Bearer token is required',
    });
  }

  const token = authHeader.split(' ')[1];

  try {
    const decoded = jwt.verify(token, SECRET);
    req.user = decoded; // Attach user to request
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
}

module.exports = authenticate;
```

```javascript
// routes/auth.js
const express = require('express');
const jwt     = require('jsonwebtoken');
const bcrypt  = require('bcryptjs');
const router  = express.Router();

const SECRET = process.env.JWT_SECRET || 'supersecret';

// Mock DB
const users = [];

// POST /auth/register
router.post('/auth/register', async (req, res) => {
  const { username, password } = req.body;

  if (!username || !password) {
    return res.status(400).json({ error: 'username and password required' });
  }

  const exists = users.find((u) => u.username === username);
  if (exists) {
    return res.status(409).json({ error: 'Username already taken' });
  }

  const hashed = await bcrypt.hash(password, 10);
  const user   = { id: users.length + 1, username, password: hashed };
  users.push(user);

  res.status(201).json({ message: 'Registered successfully' });
});

// POST /auth/login
router.post('/auth/login', async (req, res) => {
  const { username, password } = req.body;

  const user = users.find((u) => u.username === username);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const valid = await bcrypt.compare(password, user.password);
  if (!valid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Issue access token (short-lived) + refresh token (long-lived)
  const accessToken = jwt.sign(
    { id: user.id, username: user.username },
    SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { id: user.id },
    SECRET + '_refresh',
    { expiresIn: '7d' }
  );

  // Store refresh token in HttpOnly cookie
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days in ms
  });

  res.status(200).json({ accessToken });
});

// POST /auth/refresh
router.post('/auth/refresh', (req, res) => {
  const token = req.cookies?.refreshToken;

  if (!token) {
    return res.status(401).json({ error: 'No refresh token' });
  }

  try {
    const decoded     = jwt.verify(token, SECRET + '_refresh');
    const accessToken = jwt.sign(
      { id: decoded.id },
      SECRET,
      { expiresIn: '15m' }
    );
    res.status(200).json({ accessToken });
  } catch {
    return res.status(403).json({ error: 'Invalid or expired refresh token' });
  }
});

// POST /auth/logout
router.post('/auth/logout', (req, res) => {
  res.clearCookie('refreshToken');
  res.status(200).json({ message: 'Logged out successfully' });
});
```

---

### Scenario 2 — API Key Auth

```javascript
// middleware/apiKey.js
const VALID_KEYS = new Set(['key-abc-123', 'key-xyz-789']);

function apiKeyAuth(req, res, next) {
  // Accept from header OR query param
  const key = req.get('X-API-Key') || req.query.api_key;

  if (!key || !VALID_KEYS.has(key)) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Valid API key required',
    });
  }

  next();
}

module.exports = apiKeyAuth;
```

```javascript
// Usage
const apiKeyAuth = require('./middleware/apiKey');

// Protect only specific routes
app.get('/v1/data', apiKeyAuth, (req, res) => {
  res.json({ data: 'Secret data!' });
});
```

---

## 🍪 Cookies in Express

```javascript
const cookieParser = require('cookie-parser');
app.use(cookieParser());

// Set a cookie
router.get('/set-cookie', (req, res) => {
  res.cookie('theme', 'dark', {
    maxAge : 30 * 24 * 60 * 60 * 1000, // 30 days
    httpOnly: false,                    // Accessible by JS (for theme pref)
    secure  : false,                    // true in production (HTTPS only)
    sameSite: 'lax',
  });

  res.json({ message: 'Cookie set!' });
});

// Read a cookie
router.get('/read-cookie', (req, res) => {
  const theme = req.cookies.theme;
  res.json({ theme: theme || 'light' });
});

// Delete a cookie
router.get('/clear-cookie', (req, res) => {
  res.clearCookie('theme');
  res.json({ message: 'Cookie cleared' });
});
```

---

## 🌍 CORS Configuration

```javascript
const cors = require('cors');

// Simple — allow all origins (not recommended for production)
app.use(cors());

// Production-grade CORS
const corsOptions = {
  origin(origin, callback) {
    const whitelist = [
      'https://myapp.com',
      'https://admin.myapp.com',
    ];

    // Allow Postman / server-to-server (no origin)
    if (!origin || whitelist.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods         : ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders  : ['Content-Type', 'Authorization', 'X-API-Key'],
  exposedHeaders  : ['X-Total-Count', 'X-Request-Id'],
  credentials     : true,   // Allow cookies cross-origin
  maxAge          : 86400,  // Cache preflight for 24h
};

app.use(cors(corsOptions));
```

---

## 🗃️ Caching Headers

```javascript
// No caching (sensitive data)
router.get('/profile', authenticate, (req, res) => {
  res.set('Cache-Control', 'no-store');
  res.json({ user: req.user });
});

// Cache for 1 hour (public static-like data)
router.get('/countries', (req, res) => {
  res.set('Cache-Control', 'public, max-age=3600');
  res.json({ data: ['Bangladesh', 'USA', 'UK'] });
});

// ETag-based conditional caching
router.get('/posts/:id', (req, res) => {
  const post    = { id: 1, title: 'Hello World', content: '...' };
  const etag    = `"${Buffer.from(JSON.stringify(post)).toString('base64')}"`;
  const ifNone  = req.get('If-None-Match');

  if (ifNone === etag) {
    // Client already has the latest version
    return res.status(304).send();
  }

  res
    .set('ETag', etag)
    .set('Cache-Control', 'private, must-revalidate')
    .json({ data: post });
});
```

---

## 📡 Content Negotiation

```javascript
// Respond in different formats based on Accept header
router.get('/report', (req, res) => {
  const data = { name: 'Seyam', score: 95 };

  switch (req.accepts(['json', 'html', 'text'])) {
    case 'json':
      res.set('Content-Type', 'application/json');
      return res.json(data);

    case 'html':
      res.set('Content-Type', 'text/html');
      return res.send(`<h1>${data.name} — Score: ${data.score}</h1>`);

    case 'text':
      res.set('Content-Type', 'text/plain');
      return res.send(`${data.name}: ${data.score}`);

    default:
      // 406 Not Acceptable
      return res.status(406).json({ error: 'Unsupported format' });
  }
});
```

---

## ⚡ Rate Limiting

```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

// General API limiter
const apiLimiter = rateLimit({
  windowMs : 15 * 60 * 1000, // 15 minutes
  max      : 100,            // 100 requests per window
  standardHeaders: true,     // Send RateLimit-* headers
  legacyHeaders  : false,
  handler(req, res) {
    res.status(429).json({
      error     : 'Too Many Requests',
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000),
    });
  },
});

// Stricter limiter for auth endpoints
const authLimiter = rateLimit({
  windowMs : 15 * 60 * 1000,
  max      : 5,             // Only 5 login attempts per 15 min
  message  : { error: 'Too many login attempts, try again later' },
});

module.exports = { apiLimiter, authLimiter };
```

```javascript
const { apiLimiter, authLimiter } = require('./middleware/rateLimiter');

app.use('/api/', apiLimiter);          // Apply to all /api/ routes
app.use('/auth/login', authLimiter);   // Stricter on login
```

---

## 🔴 Status Code Scenarios

```javascript
// routes/posts.js — Full real-world status code usage

// 200 OK
router.get('/posts', (req, res) => {
  res.status(200).json({ data: posts });
});

// 201 Created
router.post('/posts', authenticate, (req, res) => {
  const post = { id: Date.now(), ...req.body, author: req.user.id };
  posts.push(post);
  res.status(201).location(`/posts/${post.id}`).json({ data: post });
});

// 204 No Content
router.delete('/posts/:id', authenticate, (req, res) => {
  const idx = posts.findIndex((p) => p.id === Number(req.params.id));
  if (idx === -1) return res.status(404).json({ error: 'Not found' });
  posts.splice(idx, 1);
  res.status(204).send();
});

// 400 Bad Request
router.post('/posts', (req, res) => {
  if (!req.body.title) {
    return res.status(400).json({ error: 'title is required' });
  }
});

// 401 Unauthorized
// (Handled inside authenticate middleware)

// 403 Forbidden — Auth OK but no permission
router.delete('/admin/users/:id', authenticate, (req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Admin access only' });
  }
  // proceed...
});

// 409 Conflict
router.post('/users', (req, res) => {
  if (userExists(req.body.email)) {
    return res.status(409).json({ error: 'Email already exists' });
  }
});

// 422 Unprocessable Entity
router.post('/orders', (req, res) => {
  if (req.body.quantity < 1) {
    return res.status(422).json({ error: 'quantity must be at least 1' });
  }
});
```

---

## 🛡️ Security Headers (Helmet)

```bash
npm install helmet
```

```javascript
const helmet = require('helmet');

// Sets all security headers automatically
app.use(helmet());

// What helmet sets:
// X-Content-Type-Options: nosniff
// X-Frame-Options: SAMEORIGIN
// Strict-Transport-Security: max-age=15552000
// X-XSS-Protection: 0
// Content-Security-Policy: default-src 'self'
// Referrer-Policy: no-referrer
// Permissions-Policy: ...

// Custom override
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc : ["'self'"],
      scriptSrc  : ["'self'", 'cdn.jsdelivr.net'],
      imgSrc     : ["'self'", 'data:', 'https:'],
    },
  })
);
```

---

## ⚠️ Global Error Handler

```javascript
// middleware/errorHandler.js

function errorHandler(err, req, res, next) {
  console.error(`[${new Date().toISOString()}] ERROR:`, err.message);

  // CORS error
  if (err.message === 'Not allowed by CORS') {
    return res.status(403).json({ error: 'CORS: Origin not allowed' });
  }

  // JWT error
  if (err.name === 'UnauthorizedError') {
    return res.status(401).json({ error: 'Invalid token' });
  }

  // Validation error
  if (err.name === 'ValidationError') {
    return res.status(400).json({ error: err.message });
  }

  // Default 500
  res.status(err.status || 500).json({
    error  : err.message || 'Internal Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
}

// Must be last middleware
app.use(errorHandler);
```

---

## 🔁 Redirect Scenarios

```javascript
// 301 — Permanent redirect (SEO-friendly)
app.get('/old-route', (req, res) => {
  res.redirect(301, '/new-route');
});

// 302 — Temporary redirect
app.get('/sale', (req, res) => {
  res.redirect(302, '/products?discount=true');
});

// Redirect after login
app.post('/auth/login', async (req, res) => {
  // ... authenticate ...
  const returnTo = req.query.returnTo || '/dashboard';
  res.redirect(302, returnTo);
});
```

---

## 🧩 Standardized Response Utility

```javascript
// utils/response.js
const response = {
  success(res, data, statusCode = 200, meta = {}) {
    return res.status(statusCode).json({
      success: true,
      data,
      meta,
      timestamp: new Date().toISOString(),
    });
  },

  error(res, message, statusCode = 500, details = null) {
    return res.status(statusCode).json({
      success  : false,
      error    : message,
      details,
      timestamp: new Date().toISOString(),
    });
  },

  paginated(res, data, page, limit, total) {
    return res.status(200).json({
      success: true,
      data,
      meta: {
        page        : Number(page),
        limit       : Number(limit),
        total,
        totalPages  : Math.ceil(total / limit),
        hasNext     : page * limit < total,
        hasPrev     : page > 1,
      },
    });
  },
};

module.exports = response;
```

```javascript
// Usage in routes
const R = require('../utils/response');

router.get('/users', (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  const users = fetchUsers();
  return R.paginated(res, users, page, limit, users.length);
});

router.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) return R.error(res, 'User not found', 404);
  return R.success(res, user);
});
```

---

## 🗺️ Full server.js Assembly

```javascript
const express    = require('express');
const cors       = require('cors');
const helmet     = require('helmet');
const cookieParser = require('cookie-parser');

const authRoutes  = require('./routes/auth');
const userRoutes  = require('./routes/users');
const postRoutes  = require('./routes/posts');
const errorHandler = require('./middleware/errorHandler');
const { apiLimiter } = require('./middleware/rateLimiter');

const app = express();

// ── Security ────────────────────────────────────────────
app.use(helmet());
app.use(cors({ origin: 'https://myapp.com', credentials: true }));

// ── Body Parsers ─────────────────────────────────────────
app.use(express.json({ limit: '10kb' }));       // Limit body size
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

// ── Rate Limiting ────────────────────────────────────────
app.use('/api/', apiLimiter);

// ── Request Logging ──────────────────────────────────────
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});

// ── Routes ───────────────────────────────────────────────
app.use('/api', authRoutes);
app.use('/api', userRoutes);
app.use('/api', postRoutes);

// ── 404 Fallback ─────────────────────────────────────────
app.use((req, res) => {
  res.status(404).json({ error: `Route ${req.method} ${req.url} not found` });
});

// ── Global Error Handler ─────────────────────────────────
app.use(errorHandler);

app.listen(3000, () => console.log('🚀 Server on http://localhost:3000'));
```

---

## 📊 Quick Scenario Reference

| Scenario | Method | Status | Key Headers |
|----------|--------|--------|-------------|
| Fetch list | `GET` | `200` | `Cache-Control`, `ETag` |
| Create resource | `POST` | `201` | `Location` |
| Full update | `PUT` | `200` | — |
| Partial update | `PATCH` | `200` | — |
| Delete | `DELETE` | `204` | — |
| Login success | `POST` | `200` | `Set-Cookie` |
| Unauthorized | any | `401` | `WWW-Authenticate` |
| Forbidden | any | `403` | — |
| Not found | any | `404` | — |
| Validation fail | `POST/PATCH` | `400/422` | — |
| Rate limited | any | `429` | `Retry-After` |
| Redirect | `GET` | `301/302` | `Location` |
| Cached resource | `GET` | `304` | `ETag`, `If-None-Match` |

---

This covers every major HTTP scenario you'll face in real Express.js development. Let me know if you want deeper coverage of any topic — WebSocket upgrades, file uploads (`multipart/form-data`), streaming responses, or versioned APIs!
