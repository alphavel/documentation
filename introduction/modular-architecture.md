---
layout: default
title: Modular Architecture
---

# Modular Architecture

Alphavel é **totalmente modular** por design, focando em performance máxima através da filosofia "instale apenas o que você precisa".

---

## 🎯 Filosofia: Zero Bloat

Diferente de frameworks monolíticos, Alphavel tem um **core mínimo** (apenas 5 arquivos essenciais):

```
alphavel/
├── Application.php      # Application container
├── Router.php          # High-performance router
├── Request.php         # HTTP request wrapper
├── Response.php        # HTTP response wrapper
└── Container.php       # Dependency injection
```

**Tamanho do Core:** ~15 KB  
**Dependências:** Zero (exceto Swoole)  
**Boot Time:** 5ms (sem packages)

---

## 📦 Sistema de Packages

Tudo além do core é um **package opcional**:

### Available Packages

| Package | Size | Boot Impact | Purpose |
|---------|------|-------------|---------|
| **alphavel/database** | 45 KB | +2ms | MySQL/PostgreSQL with pooling |
| **alphavel/cache** | 12 KB | +1ms | Redis/Memcached support |
| **alphavel/logging** | 8 KB | +0.5ms | Monolog integration |
| **alphavel/events** | 6 KB | +0.3ms | Event dispatcher |
| **alphavel/validation** | 18 KB | +1ms | Request validation |
| **alphavel/support** | 10 KB | +0.5ms | Collection & helpers |

### Installation

```bash
# Install only what you need
composer require alphavel/database
composer require alphavel/cache

# Or use CLI
alpha package:add database
alpha package:add cache
```

---

## 🚀 Performance Impact

### Core Only (Minimal Setup)

{% raw %}
```php
// bootstrap/app.php
$app = new Alphavel\Application();

// Only routing - no packages
$app->router->get('/health', fn() => ['status' => 'ok']);

return $app;
```
{% endraw %}

**Performance:**
- Boot time: **5ms**
- Memory: **2 MB**
- Requests/sec: **520,000+**
- Latency: **0.19ms**

### With Database Package

```bash
composer require alphavel/database
```

{% raw %}
```php
// config/app.php
'providers' => [
    Alphavel\Database\DatabaseServiceProvider::class,
],
```
{% endraw %}

**Performance:**
- Boot time: **7ms** (+2ms)
- Memory: **8 MB** (+6 MB for connections)
- Requests/sec: **45,000+**
- Latency: **2.2ms**

### With Full Stack

```bash
composer require alphavel/database
composer require alphavel/cache
composer require alphavel/validation
composer require alphavel/logging
composer require alphavel/events
```

**Performance:**
- Boot time: **12ms** (+7ms total)
- Memory: **12 MB** (+10 MB)
- Requests/sec: **38,000+**
- Latency: **2.6ms**

---

## 🔄 Auto-Discovery

Packages register automaticamente via `composer.json`:

### Package Structure

```json
{
  "name": "alphavel/database",
  "extra": {
    "alphavel": {
      "providers": [
        "Alphavel\\Database\\DatabaseServiceProvider"
      ]
    }
  }
}
```

### How It Works

1. **Install Package**
   ```bash
   composer require alphavel/database
   ```

2. **Auto-Discovery**
   - Alphavel reads `vendor/composer/installed.json`
   - Finds packages with `extra.alphavel.providers`
   - Caches in `storage/cache/providers.php`

3. **Ready to Use**
   ```php
   use Alphavel\Database\DB;
   
   $users = DB::table('users')->get();
   ```

**No manual registration needed!**

---

## 🎨 Modular Benefits

### 1. Minimal Memory Footprint

Only load what you use:

{% raw %}
```php
// API without database? Don't install it!
$app = new Application();
$app->router->get('/status', fn() => ['status' => 'ok']);

// Memory: 2 MB (vs 40 MB in Laravel)
```
{% endraw %}

### 2. Faster Boot Time

Fewer packages = faster startup:

| Setup | Boot Time | Comparison |
|-------|-----------|------------|
| Core only | 5ms | ⚡ Baseline |
| + Database | 7ms | +40% |
| + Cache | 8ms | +60% |
| Full stack | 12ms | +140% |
| **Laravel** | 450ms | **+9000%** 🐌 |

### 3. Microservices-Friendly

Build specialized microservices:

```bash
# Auth Service (needs database + cache)
composer require alphavel/database alphavel/cache

# Logging Service (needs logging + events)
composer require alphavel/logging alphavel/events

# Gateway Service (core only - proxies requests)
# No extra packages needed!
```

### 4. Production Optimization

Deploy only required dependencies:

```dockerfile
# Dockerfile - Payment Service
FROM php:8.2-swoole

# Only database package (no cache, logging, etc)
RUN composer require alphavel/database

# Result: Smaller image, faster deployments
```

---

## 🏗️ Architecture Layers

### Layer 1: Core Framework

```
Application Container
├── Router (route matching)
├── Request (HTTP input)
├── Response (HTTP output)
└── Container (DI)
```

**Always loaded** - Essential for any application.

### Layer 2: Service Providers

{% raw %}
```php
// Loaded based on installed packages
DatabaseServiceProvider::class
CacheServiceProvider::class
ValidationServiceProvider::class
LoggingServiceProvider::class
EventServiceProvider::class
```
{% endraw %}

**Conditionally loaded** - Only if package is installed.

### Layer 3: Application Code

```
app/
├── Controllers/
├── Models/
├── Services/
└── Middleware/
```

**Your code** - Business logic.

---

## 🔧 Custom Packages

Create your own modular packages:

### Step 1: Package Structure

```
my-package/
├── composer.json
├── src/
│   ├── MyService.php
│   └── MyServiceProvider.php
└── config/
    └── my-package.php
```

### Step 2: composer.json

```json
{
  "name": "company/my-package",
  "autoload": {
    "psr-4": {
      "Company\\MyPackage\\": "src/"
    }
  },
  "extra": {
    "alphavel": {
      "providers": [
        "Company\\MyPackage\\MyServiceProvider"
      ]
    }
  }
}
```

### Step 3: Service Provider

{% raw %}
```php
namespace Company\MyPackage;

use Alphavel\Framework\ServiceProvider;

class MyServiceProvider extends ServiceProvider
{
    protected bool $defer = true;  // Lazy load
    
    public function register(): void
    {
        $this->app->singleton('my-service', function ($app) {
            return new MyService($app->config('my-package'));
        });
    }
    
    public function provides(): array
    {
        return ['my-service'];
    }
}
```
{% endraw %}

### Step 4: Install & Use

```bash
composer require company/my-package
```

{% raw %}
```php
// Auto-discovered, ready to use!
$result = app('my-service')->doSomething();
```
{% endraw %}

---

## 📊 Real-World Comparison

### Laravel (Monolithic)

```bash
# Laravel installation
composer create-project laravel/laravel app

# Installs EVERYTHING by default:
# - Database (Eloquent ORM)
# - Queue system
# - Email system
# - Broadcasting
# - File storage
# - Session management
# - Authentication scaffolding
# - Cache layer
# - Redis
# - etc...

# Result:
# - 80+ vendor packages
# - 120 MB installation
# - 450ms boot time
# - 40 MB memory (minimum)
```

**Problem:** Can't remove unwanted features easily.

### Alphavel (Modular)

```bash
# Alphavel installation
composer create-project alphavel/skeleton app

# Core only:
# - Router
# - Request/Response
# - Container

# Result:
# - 5 vendor packages
# - 2 MB installation
# - 5ms boot time
# - 2 MB memory

# Add features as needed:
composer require alphavel/database  # +2ms boot
composer require alphavel/cache     # +1ms boot
```

**Benefit:** Full control over performance/features trade-off.

---

## 🎯 Use Case Examples

### Case 1: High-Traffic API Gateway

**Requirements:**
- Route requests to microservices
- No database needed
- Maximum performance

**Solution:**
```bash
# Core only
composer create-project alphavel/skeleton gateway

# Optional: Add logging for debugging
composer require alphavel/logging
```

**Result:** 520k req/s with 2 MB memory footprint.

### Case 2: CRUD Microservice

**Requirements:**
- Database operations
- Cache layer
- Input validation

**Solution:**
```bash
composer create-project alphavel/skeleton crud-service
composer require alphavel/database
composer require alphavel/cache
composer require alphavel/validation
```

**Result:** 38k req/s with 12 MB memory footprint.

### Case 3: Event-Driven Worker

**Requirements:**
- Process events from queue
- Log processing
- No HTTP routes

**Solution:**
```bash
composer create-project alphavel/skeleton worker
composer require alphavel/events
composer require alphavel/logging
composer require alphavel/database
```

**Result:** Process 10k+ events/sec with 8 MB memory.

---

## 🔍 Performance Analysis

### Memory Usage by Package

{% raw %}
```php
// Measure memory impact
$before = memory_get_usage(true);

require 'vendor/autoload.php';
$app = new Alphavel\Application();

// Core only
$coreMemory = memory_get_usage(true) - $before;
echo "Core: " . ($coreMemory / 1024 / 1024) . " MB\n";
// Output: Core: 2 MB

// Add database
$app->register(new DatabaseServiceProvider($app));
$dbMemory = memory_get_usage(true) - $before - $coreMemory;
echo "Database: " . ($dbMemory / 1024 / 1024) . " MB\n";
// Output: Database: 6 MB (includes connection pool)

// Add cache
$app->register(new CacheServiceProvider($app));
$cacheMemory = memory_get_usage(true) - $before - $coreMemory - $dbMemory;
echo "Cache: " . ($cacheMemory / 1024 / 1024) . " MB\n";
// Output: Cache: 2 MB
```
{% endraw %}

### Boot Time by Package

{% raw %}
```php
// Measure boot time impact
$start = microtime(true);

$app = new Alphavel\Application();
$coreTime = (microtime(true) - $start) * 1000;
echo "Core boot: {$coreTime}ms\n";
// Output: Core boot: 5ms

$app->register(new DatabaseServiceProvider($app));
$dbTime = (microtime(true) - $start) * 1000 - $coreTime;
echo "Database boot: {$dbTime}ms\n";
// Output: Database boot: 2ms

$app->register(new CacheServiceProvider($app));
$cacheTime = (microtime(true) - $start) * 1000 - $coreTime - $dbTime;
echo "Cache boot: {$cacheTime}ms\n";
// Output: Cache boot: 1ms
```
{% endraw %}

---

## 🚀 Production Optimization

### Step 1: Audit Packages

```bash
# List installed packages
composer show alphavel/*

# Remove unused packages
composer remove alphavel/events  # Not using events? Remove it!
```

### Step 2: Cache Everything

```bash
# Cache service providers
alpha optimize

# Cache routes
alpha route:cache

# Cache config
alpha config:cache
```

### Step 3: Measure Impact

```bash
# Before optimization
alpha benchmark
# Result: 35k req/s, 15ms boot, 18 MB memory

# After removing unused packages + caching
alpha benchmark
# Result: 45k req/s, 8ms boot, 10 MB memory
```

**Performance Gain:** +28% throughput, -47% boot time, -44% memory

---

## 📋 Package Selection Guide

### When to Install Each Package

| Package | Install If... | Skip If... |
|---------|---------------|------------|
| **database** | Need MySQL/PostgreSQL | Using external API only |
| **cache** | Need Redis/Memcached | Stateless application |
| **validation** | User input validation | Internal microservice |
| **logging** | Need structured logs | Using external logging |
| **events** | Event-driven architecture | Simple CRUD only |
| **support** | Need collections/helpers | Minimal dependencies |

### Recommended Combinations

**API Gateway:**
```bash
# Core only (or + logging for debugging)
composer require alphavel/logging
```

**REST API:**
```bash
composer require alphavel/database
composer require alphavel/validation
composer require alphavel/cache
```

**Event Processor:**
```bash
composer require alphavel/events
composer require alphavel/database
composer require alphavel/logging
```

**Webhook Handler:**
```bash
composer require alphavel/validation
composer require alphavel/logging
```

---

## 🎓 Best Practices

### ✅ DO

- **Start minimal** - Add packages as requirements grow
- **Profile first** - Measure before adding packages
- **Use deferred providers** - Lazy load services
- **Remove unused packages** - Audit regularly

### ❌ DON'T

- **Install everything** - "Just in case" bloats your app
- **Ignore boot time** - Each package adds overhead
- **Skip optimization** - Cache routes/config in production
- **Copy Laravel setup** - Alphavel is different by design

---

## 🔗 Next Steps

- [Package System →](../core/packages.html)
- [Service Providers →](../core/service-providers.html)
- [Performance Optimization →](../core/performance.html)
- [Production Deployment →](../deployment/production.html)

---

## 💡 Key Takeaway

> **Alphavel's modular architecture lets you choose between maximum performance (core only) and developer convenience (full packages). Start minimal, add features as needed, and maintain the performance advantage that matters for your use case.**

**Remember:** Every package you don't install is microseconds saved on every request. In high-traffic scenarios, this compounds to significant cost savings and better user experience.
