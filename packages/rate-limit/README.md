---
layout: default
title: README
---

# Rate Limiting (alphavel/rate-limit)

High-performance rate limiting package for Alphavel using Swoole Table.

---

## 🎯 Overview

The `alphavel/rate-limit` package provides ultra-fast rate limiting with **0.001ms latency** (1000x faster than Redis) using Swoole's shared memory (Swoole Table).

### Key Features

- ⚡ **0.001ms latency** - Atomic operations in shared memory
- 🚀 **Zero dependencies** - Uses Swoole built-in Table
- 🔒 **Thread-safe** - Atomic operations, no race conditions
- 💾 **Shared memory** - Works across all Swoole workers
- 🎯 **Multiple levels** - IP, User, API Key, Endpoint, Session, Global
- 🛡️ **DDoS protection** - Global rate limiting
- 📊 **CLI tools** - Stats, list, reset, block commands

---

## 📦 Installation

```bash
composer require alphavel/rate-limit
```

---

## ⚙️ Configuration

### 1. Publish Config

```bash
php alpha vendor:publish --tag=rate-limit-config
```

### 2. Environment Variables

```bash
# .env

RATE_LIMIT_DRIVER=swoole
RATE_LIMIT_MAX_ENTRIES=100000  # Max unique IPs/users (1.6MB for 100k)
RATE_LIMIT_DEFAULT=1000        # Default: 1000 req/min
RATE_LIMIT_WINDOW=60           # Default: 60 seconds

# Global Rate Limiting (DDoS Protection)
RATE_LIMIT_ENABLE_GLOBAL=false
RATE_LIMIT_GLOBAL=10000        # 10k req/s globally
```

### 3. Initialize (Critical!)

Initialize Swoole Table **BEFORE** `$server->start()`:

{% raw %}
```php
// bootstrap/server.php

use Alphavel\RateLimit\Drivers\SwooleTableDriver;
use Swoole\Http\Server;

require __DIR__ . '/../vendor/autoload.php';

$app = require_once __DIR__ . '/app.php';

// ⚠️ CRITICAL: Initialize BEFORE server start
SwooleTableDriver::init(config('rate_limit.swoole.max_entries', 100000));

$server = new Server('0.0.0.0', 8087);

$server->on('request', function ($request, $response) use ($app) {
    $app->handle($request, $response);
});

$server->start();
```
{% endraw %}

---

## 🚀 Usage

### Basic Rate Limiting

{% raw %}
```php
// routes/api.php

// 100 requests per minute per IP
$router->middleware('rate_limit:100,60,ip')->group(function ($router) {
    $router->post('/auth/login', [AuthController::class, 'login']);
    $router->post('/auth/register', [AuthController::class, 'register']);
});

// 1000 requests per minute per authenticated user
$router->middleware(['auth', 'rate_limit:1000,60,user'])->group(function ($router) {
    $router->get('/users', [UserController::class, 'index']);
    $router->put('/users/me', [UserController::class, 'update']);
});

// 10 requests per minute per IP on this endpoint
$router->middleware('rate_limit:10,60,endpoint')->post('/reports/generate', [ReportController::class, 'generate']);
```
{% endraw %}

### Multiple Levels (Defense in Depth)

{% raw %}
```php
// Apply multiple rate limits
$router->middleware([
    'rate_limit:1000,60,ip',      // 1000/min per IP
    'rate_limit:100,60,user',      // 100/min per user
    'rate_limit:10,60,endpoint'    // 10/min on this endpoint
])->post('/ai/generate', [AIController::class, 'generate']);
```
{% endraw %}

### Available Levels

| Level | Key Format | Description |
|-------|-----------|-------------|
| `ip` | `ip:192.168.1.1` | Rate limit by IP address |
| `user` | `user:123` | Rate limit by authenticated user ID |
| `api_key` | `key:abc123` | Rate limit by API key (X-API-Key header) |
| `endpoint` | `endpoint:192.168.1.1:/api/test` | Rate limit by IP + endpoint path |
| `session` | `session:sess_xyz` | Rate limit by session ID |
| `global` | `global` | Global rate limit (all requests) |

---

## 🛡️ DDoS Protection

Enable global rate limiting to protect against DDoS:

```bash
# .env
RATE_LIMIT_ENABLE_GLOBAL=true
RATE_LIMIT_GLOBAL=10000  # 10k requests/second globally
```

This is applied **before** individual rate limits and returns HTTP 503 when exceeded.

---

## 🎯 Whitelist

Add trusted IPs that are never rate limited:

{% raw %}
```php
// config/rate_limit.php

'whitelist' => [
    '127.0.0.1',
    '::1',
    '10.0.0.0/8',        // Private network (CIDR notation)
    '192.168.1.100',     // Load balancer
    '203.0.113.0/24',    // Office network
],
```
{% endraw %}

---

## 📊 Response Headers

### Successful Request

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1700000000
```

### Rate Limit Exceeded

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 42
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1700000042

{
  "error": "rate_limit_exceeded",
  "message": "Rate limit of 100 requests per minute exceeded for your IP.",
  "retry_after": 42,
  "reset_at": 1700000042
}
```

---

## 🖥️ CLI Commands

### Show Statistics

```bash
php alpha rate-limit:stats
```

**Output:**
```
Rate Limit Statistics
+----------------+--------+
| Metric         | Value  |
+----------------+--------+
| Total Entries  | 1,245  |
| Active Limits  | 892    |
| Blocked IPs    | 23     |
| Memory Usage   | 1.87 MB|
+----------------+--------+
```

### List Active Limits

```bash
# List all active rate limits
php alpha rate-limit:list

# Show only blocked entries
php alpha rate-limit:list --blocked
```

### Reset Rate Limit

```bash
# Reset for specific IP
php alpha rate-limit:reset ip:192.168.1.1

# Reset for user
php alpha rate-limit:reset user:123
```

### Block IP/User

```bash
# Block for 1 hour (default)
php alpha rate-limit:block ip:192.168.1.1

# Block for specific duration
php alpha rate-limit:block ip:192.168.1.1 --duration=3600
```

---

## 📊 Performance

### Benchmark Results

```
Test Setup:
- Hardware: 4 cores, 8GB RAM
- Tool: wrk -t4 -c100 -d20s

Results:
Baseline (no rate limit):  5,042 req/s  31.00ms latency
With rate limit:            5,038 req/s  31.02ms latency
Overhead:                   0.08%        +0.02ms

✅ Negligible performance impact!
```

### Memory Usage

```
Swoole Table: 16 bytes per entry

100k entries  = 1.6 MB
500k entries  = 8 MB
1M entries    = 16 MB
```

**Recommendation:** Configure `max_entries` based on:
- Expected unique IPs per hour
- Number of authenticated users
- Number of API keys
- Available memory

---

## 🏗️ Architecture

### How It Works

```
Request
  ↓
Middleware checks whitelist
  ↓
Global rate limit check (if enabled)
  ↓
Specific rate limit check (IP/User/etc)
  ↓
Swoole Table atomic operations
  ├── First request → Create entry, allow
  ├── Within limit → Increment counter, allow
  ├── Exceeded limit → Block, return 429
  └── Window expired → Reset counter, allow
  ↓
Response with X-RateLimit-* headers
```

### Swoole Table Benefits

| Feature | Swoole Table | Redis |
|---------|-------------|-------|
| **Latency** | 0.001ms | ~0.15ms |
| **Network** | None (in-memory) | TCP overhead |
| **Dependencies** | None (Swoole built-in) | External service |
| **Thread-safe** | Yes (atomic ops) | Yes |
| **Persistence** | No (volatile) | Yes |
| **Shared workers** | Yes | Yes |
| **Best for** | Rate limiting | General caching |

---

## 🧪 Testing

### Unit Tests

```bash
cd rate-limit
composer install
composer test
```

### Integration Testing

```bash
# Start server
php alpha serve

# Test rate limiting
for i in {1..150}; do
  curl http://localhost:8087/api/test-limited
done

# Should see:
# - First 100 requests: 200 OK
# - Next 50 requests: 429 Too Many Requests
```

---

## 🔍 Troubleshooting

### Error: "SwooleTableDriver not initialized"

**Cause:** Swoole Table not initialized before `$server->start()`

**Solution:**
{% raw %}
```php
// bootstrap/server.php

// ✅ Correct: BEFORE server start
SwooleTableDriver::init(100000);

$server->start();
```
{% endraw %}

### Rate Limit Not Shared Between Workers

**Cause:** Table is not static

**Fix:** Ensure driver uses `private static ?Table $table`

### Memory Exhausted

**Cause:** `max_entries` too high

**Solution:** Reduce in config:
{% raw %}
```php
'max_entries' => 50000, // Reduces to 800KB
```
{% endraw %}

---

## 🎓 Best Practices

### ✅ DO

- Initialize Swoole Table before server start
- Use IP-based limiting for public endpoints
- Use user-based limiting for authenticated endpoints
- Whitelist monitoring/health check IPs
- Use global limiting for DDoS protection
- Monitor stats regularly

### ❌ DON'T

- Set max_entries too high (wastes memory)
- Forget to initialize Swoole Table
- Use rate limiting for health checks (whitelist them)
- Set limits too low (frustrates users)
- Use for permanent blocking (use firewall)

---

## 📚 Related Documentation

- [Swoole Table Documentation](https://www.swoole.co.uk/docs/modules/swoole-table)
- [HTTP 429 Status Code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)
- [Rate Limiting Best Practices](https://cloud.google.com/architecture/rate-limiting-strategies-techniques)

---

## 🔗 Package Repository

https://github.com/alphavel/rate-limit

---

## 💡 Use Cases

### 1. API Protection

{% raw %}
```php
// Prevent API abuse
$router->middleware('rate_limit:1000,60,api_key')->group(function ($router) {
    $router->get('/api/users', [ApiController::class, 'users']);
    $router->get('/api/orders', [ApiController::class, 'orders']);
});
```
{% endraw %}

### 2. Login Brute Force Protection

{% raw %}
```php
// 5 login attempts per minute
$router->middleware('rate_limit:5,60,ip')->post('/auth/login', [AuthController::class, 'login']);
```
{% endraw %}

### 3. Heavy Operations

{% raw %}
```php
// 10 exports per hour per user
$router->middleware(['auth', 'rate_limit:10,3600,user'])->post('/exports/csv', [ExportController::class, 'csv']);
```
{% endraw %}

### 4. Public API with Tiered Limits

{% raw %}
```php
// Free tier: 100/day
$router->middleware('rate_limit:100,86400,api_key')->group(function ($router) {
    $router->get('/api/free/*');
});

// Pro tier: 10000/day
$router->middleware('rate_limit:10000,86400,api_key')->group(function ($router) {
    $router->get('/api/pro/*');
});
```
{% endraw %}

---

**Built with ❤️ using Swoole Table for maximum performance** 🚀
