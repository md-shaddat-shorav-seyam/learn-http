# learn-http
# 🌐 HTTP — Full Tutorial

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the foundation of data communication on the Web. It's a **request-response protocol** — a client sends a request, and a server sends back a response.

---

## 📦 HTTP Structure

Every HTTP communication has two parts:

### 1. Request (Client → Server)
```
METHOD /path HTTP/version
Headers

Body (optional)
```

### 2. Response (Server → Client)
```
HTTP/version STATUS_CODE STATUS_MESSAGE
Headers

Body (optional)
```

---

## 🔧 HTTP Methods

| Method | Purpose | Has Body? | Idempotent? | Safe? |
|--------|---------|-----------|-------------|-------|
| `GET` | Retrieve a resource | ❌ | ✅ | ✅ |
| `POST` | Create a new resource | ✅ | ❌ | ❌ |
| `PUT` | Replace a resource entirely | ✅ | ✅ | ❌ |
| `PATCH` | Partially update a resource | ✅ | ❌ | ❌ |
| `DELETE` | Delete a resource | ❌/✅ | ✅ | ❌ |
| `HEAD` | Like GET but no body returned | ❌ | ✅ | ✅ |
| `OPTIONS` | Returns allowed methods | ❌ | ✅ | ✅ |
| `CONNECT` | Establish a tunnel (HTTPS) | ❌ | ❌ | ❌ |
| `TRACE` | Loop-back test / debugging | ❌ | ✅ | ✅ |

> **Idempotent** = calling it multiple times gives the same result.  
> **Safe** = does not modify server state.

### Examples

```http
GET /users/42 HTTP/1.1
Host: api.example.com
```

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Seyam",
  "email": "seyam@example.com"
}
```

```http
PUT /users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Seyam Updated",
  "email": "seyam@example.com"
}
```

```http
PATCH /users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "email": "newemail@example.com"
}
```

```http
DELETE /users/42 HTTP/1.1
Host: api.example.com
```

---

## 📋 HTTP Headers

Headers carry **metadata** about the request or response. They follow the format:
```
Header-Name: value
```

### 🔵 Request Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Host` | Target server hostname (required in HTTP/1.1) | `Host: api.example.com` |
| `User-Agent` | Identifies the client/browser | `User-Agent: Mozilla/5.0` |
| `Accept` | Media types the client can handle | `Accept: application/json` |
| `Accept-Language` | Preferred language | `Accept-Language: en-US` |
| `Accept-Encoding` | Compression types supported | `Accept-Encoding: gzip, deflate, br` |
| `Content-Type` | Format of the request body | `Content-Type: application/json` |
| `Content-Length` | Size of the request body in bytes | `Content-Length: 348` |
| `Authorization` | Credentials for authentication | `Authorization: Bearer <token>` |
| `Cookie` | Sends stored cookies to server | `Cookie: sessionId=abc123` |
| `Referer` | URL of the previous page | `Referer: https://example.com/home` |
| `Origin` | Origin of a cross-site request | `Origin: https://myapp.com` |
| `Connection` | Control connection persistence | `Connection: keep-alive` |
| `Cache-Control` | Caching directives | `Cache-Control: no-cache` |
| `If-Modified-Since` | Conditional GET (check freshness) | `If-Modified-Since: Sat, 29 Oct 2024 19:43:31 GMT` |
| `If-None-Match` | Conditional GET using ETag | `If-None-Match: "686897696a7c876b7e"` |
| `Range` | Request partial content | `Range: bytes=0-1023` |

### 🟢 Response Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Format of the response body | `Content-Type: application/json; charset=utf-8` |
| `Content-Length` | Size of response body | `Content-Length: 1024` |
| `Content-Encoding` | Compression used | `Content-Encoding: gzip` |
| `Set-Cookie` | Sets a cookie on the client | `Set-Cookie: sessionId=abc; HttpOnly; Secure` |
| `Cache-Control` | How the response should be cached | `Cache-Control: max-age=3600` |
| `ETag` | Unique version tag for a resource | `ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"` |
| `Last-Modified` | Last time the resource was modified | `Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT` |
| `Location` | Redirect target URL | `Location: https://example.com/new-page` |
| `Server` | Server software info | `Server: nginx/1.18.0` |
| `WWW-Authenticate` | Auth challenge | `WWW-Authenticate: Basic realm="Access"` |
| `Access-Control-Allow-Origin` | CORS allowed origins | `Access-Control-Allow-Origin: *` |
| `Strict-Transport-Security` | Enforce HTTPS (HSTS) | `Strict-Transport-Security: max-age=31536000` |
| `X-Content-Type-Options` | Prevent MIME sniffing | `X-Content-Type-Options: nosniff` |
| `X-Frame-Options` | Clickjacking protection | `X-Frame-Options: DENY` |
| `Retry-After` | When to retry after 429/503 | `Retry-After: 120` |

---

## 🔢 HTTP Status Codes

### 1xx — Informational
| Code | Name | Meaning |
|------|------|---------|
| `100` | Continue | Initial part of request received, client may continue |
| `101` | Switching Protocols | Server is switching protocols (e.g., WebSocket) |

### 2xx — Success
| Code | Name | Meaning |
|------|------|---------|
| `200` | OK | Request succeeded |
| `201` | Created | Resource was created successfully |
| `202` | Accepted | Request accepted but not yet processed |
| `204` | No Content | Success, but no body returned (e.g., DELETE) |
| `206` | Partial Content | Partial resource returned (Range request) |

### 3xx — Redirection
| Code | Name | Meaning |
|------|------|---------|
| `301` | Moved Permanently | Resource permanently moved to new URL |
| `302` | Found | Temporary redirect |
| `304` | Not Modified | Cached version is still valid |
| `307` | Temporary Redirect | Redirect preserving the HTTP method |
| `308` | Permanent Redirect | Like 301, but method is preserved |

### 4xx — Client Errors
| Code | Name | Meaning |
|------|------|---------|
| `400` | Bad Request | Malformed or invalid request |
| `401` | Unauthorized | Authentication required or failed |
| `403` | Forbidden | Authenticated but not authorized |
| `404` | Not Found | Resource does not exist |
| `405` | Method Not Allowed | HTTP method not supported for this endpoint |
| `408` | Request Timeout | Server timed out waiting for request |
| `409` | Conflict | State conflict (e.g., duplicate entry) |
| `410` | Gone | Resource permanently deleted |
| `413` | Payload Too Large | Body exceeds server's limit |
| `415` | Unsupported Media Type | Content-Type not supported |
| `422` | Unprocessable Entity | Syntactically correct but semantically invalid |
| `429` | Too Many Requests | Rate limit exceeded |

### 5xx — Server Errors
| Code | Name | Meaning |
|------|------|---------|
| `500` | Internal Server Error | Generic server-side failure |
| `501` | Not Implemented | Feature not yet supported |
| `502` | Bad Gateway | Upstream server returned an invalid response |
| `503` | Service Unavailable | Server temporarily down or overloaded |
| `504` | Gateway Timeout | Upstream server timed out |

---

## 🔄 HTTP Versions

| Version | Key Features |
|---------|-------------|
| **HTTP/1.0** | New connection per request (no persistent connections) |
| **HTTP/1.1** | Persistent connections (`keep-alive`), pipelining, chunked transfer |
| **HTTP/2** | Binary framing, multiplexing, header compression (HPACK), server push |
| **HTTP/3** | Built on **QUIC** (UDP), faster handshake, no head-of-line blocking |

---

## 🔐 HTTPS

HTTPS = HTTP + **TLS (Transport Layer Security)**

```
Client                          Server
  |                               |
  |------- TLS ClientHello ------>|
  |<------ TLS ServerHello -------|
  |<------ Certificate -----------|
  |------- Key Exchange --------->|
  |<====== Encrypted Channel ====>|
  |------- GET /index.html ------>|
  |<------ 200 OK + HTML ---------|
```

**TLS provides:**
- **Encryption** — Data is unreadable to eavesdroppers
- **Authentication** — Server identity verified via certificate
- **Integrity** — Data cannot be tampered with in transit

---

## 🍪 Cookies

Cookies are small pieces of data stored by the browser and sent with every request.

```http
# Server sets a cookie
Set-Cookie: sessionId=xyz789; Path=/; HttpOnly; Secure; SameSite=Strict; Max-Age=86400

# Browser sends it back automatically
Cookie: sessionId=xyz789
```

| Attribute | Meaning |
|-----------|---------|
| `HttpOnly` | Not accessible via JavaScript (XSS protection) |
| `Secure` | Only sent over HTTPS |
| `SameSite=Strict` | Not sent in cross-site requests (CSRF protection) |
| `Max-Age` | Expiry in seconds |
| `Path` | Which paths the cookie applies to |

---

## 🌍 CORS (Cross-Origin Resource Sharing)

When a web page at `https://app.com` requests `https://api.other.com`, the browser performs a **CORS check**.

### Preflight Request (for non-simple methods)
```http
OPTIONS /api/data HTTP/1.1
Host: api.other.com
Origin: https://app.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization
```

### Preflight Response
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

---

## 🔑 Authentication Methods

### 1. Basic Auth
```http
Authorization: Basic base64(username:password)
```

### 2. Bearer Token (JWT)
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 3. API Key
```http
X-API-Key: my-secret-api-key-here
```

### 4. OAuth 2.0 Flow (simplified)
```
User → App → Authorization Server → Access Token → API
```

---

## 🗃️ Caching

```http
# Response tells browser to cache for 1 hour
Cache-Control: public, max-age=3600

# No caching at all
Cache-Control: no-store

# Revalidate every time (use ETag/Last-Modified)
Cache-Control: no-cache

# ETag-based conditional request
If-None-Match: "abc123"
# Server replies with 304 Not Modified if unchanged
```

---

## 📡 Content Negotiation

```http
# Client wants JSON, falls back to XML
Accept: application/json, application/xml;q=0.9, */*;q=0.8

# Client wants English, falls back to French
Accept-Language: en-US, en;q=0.9, fr;q=0.7

# Server responds
Content-Type: application/json
Content-Language: en
```

---

## 🔁 Full Request-Response Example

### Request
```http
POST /api/login HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0
Accept: application/json
Content-Type: application/json
Content-Length: 51
Connection: keep-alive

{
  "username": "seyam",
  "password": "secret123"
}
```

### Response
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 127
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
Cache-Control: no-store
Date: Tue, 05 May 2026 10:00:00 GMT

{
  "status": "success",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600
}
```

---

## ⚡ Quick Reference Cheatsheet

```
┌─────────────────────────────────────────────────────────┐
│                    HTTP METHODS                         │
│  GET    → Read        POST   → Create                   │
│  PUT    → Replace     PATCH  → Update                   │
│  DELETE → Delete      HEAD   → Metadata only            │
├─────────────────────────────────────────────────────────┤
│                  STATUS CODES                           │
│  2xx → Success        3xx → Redirect                    │
│  4xx → Client Error   5xx → Server Error                │
├─────────────────────────────────────────────────────────┤
│               KEY REQUEST HEADERS                       │
│  Host, Authorization, Content-Type, Accept, Cookie      │
├─────────────────────────────────────────────────────────┤
│               KEY RESPONSE HEADERS                      │
│  Content-Type, Set-Cookie, Cache-Control, Location      │
└─────────────────────────────────────────────────────────┘
```

---

This covers the full foundation of HTTP. Let me know if you want a deep dive into any specific section — like WebSockets, HTTP/2 internals, JWT in detail, or REST API design patterns!
