---
layout: default
title: README
---

# Cache Package

High-performance caching layer using **Swoole\Table** (in-memory, zero-copy shared memory).

**Performance:** 520k+ req/s (cached responses)

---

## Installation

```bash
composer require alphavel/cache
```

Or via CLI wizard:

```bash
php alpha package:add cache
```

---

## Configuration

The cache uses **Swoole\Table** by default (no Redis/Memcached required):

**In `bootstrap/app.php` (already configured):**

{% raw %}
```php
use Alphavel\Cache\Cache;

// Initialize cache with Swoole\Table
$cache = Cache::getInstance(
    size: 1024,      // Max entries (increase for more data)
    valueSize: 4096  // Max value size in bytes
);
```
{% endraw %}

**Environment variables (.env):**

```env
# Cache Configuration
CACHE_SIZE=1024        # Max number of cache entries
CACHE_VALUE_SIZE=4096  # Max size per entry (bytes)
```

**⚠️ Important for Beginners:**

- `CACHE_SIZE`: Total number of keys you can store (e.g., 1024 = ~1000 users cached)
- `CACHE_VALUE_SIZE`: Maximum size of each cached value (4KB default = ~4000 characters)
- **Memory usage**: `CACHE_SIZE × CACHE_VALUE_SIZE` = 1024 × 4096 = 4MB RAM
- Exceeding limits: Old entries are NOT evicted automatically (set() returns false)

---

## Basic Usage

{% raw %}
```php
use Alphavel\Cache\Cache;

$cache = Cache::getInstance();

// Set cache (expires in 3600 seconds)
$cache->set('user:1', ['name' => 'John'], 3600);

// Get cache
$user = $cache->get('user:1');

// Get with default value
$user = $cache->get('user:1', ['name' => 'Guest']);

// Check if exists (respects TTL)
if ($cache->has('user:1')) {
    // Key exists and not expired
}

// Delete specific key
$cache->delete('user:1');

// Clear all cache
$cache->clear();

// Remember pattern (get from cache or execute callback)
$users = $cache->remember('users:all', 3600, function () {
    return DB::query('SELECT * FROM users');
});

// Pull (get and delete)
$value = $cache->pull('temp:data');

// Increment/Decrement (atomic operations)
$cache->increment('page:views');
$cache->decrement('stock:item:5', 3);

// Get cache statistics
$count = $cache->count(); // Number of entries
```
{% endraw %}

---

## ⚡ Performance Guide for Beginners

### When to Use Cache

| Scenario | Cache? | TTL | Example |
|----------|--------|-----|---------|
| **Database queries** (rarely change) | ✅ Yes | 3600s | User profiles, product catalog |
| **API responses** (external APIs) | ✅ Yes | 1800s | Weather data, exchange rates |
| **Computed results** (expensive) | ✅ Yes | 7200s | Reports, statistics, aggregations |
| **User sessions** | ❌ No | - | Use `alphavel/session` instead |
| **Real-time data** (always fresh) | ❌ No | - | Live scores, stock prices |

### Real Performance Impact

{% raw %}
```php
// ❌ WITHOUT CACHE: 6,700 req/s
Route::get('/api/users', function() {
    return DB::table('users')->get();
});

// ✅ WITH CACHE: 520,000+ req/s (77x faster!)
Route::get('/api/users', function() {
    $cache = Cache::getInstance();
    return $cache->remember('users:all', 60, function() {
        return DB::table('users')->get();
    });
});
```
{% endraw %}

### Best Practices for Beginners

**1. Use Short TTLs for Starting**
{% raw %}
```php
// ❌ BAD: 1 day cache (data gets stale)
$cache->set('products', $data, 86400);

// ✅ GOOD: 5 minutes (balance freshness vs performance)
$cache->set('products', $data, 300);
```
{% endraw %}

**2. Always Provide Default Values**
{% raw %}
```php
// ❌ BAD: No default (might return null)
$user = $cache->get('user:' . $id);

// ✅ GOOD: Default value prevents null checks
$user = $cache->get('user:' . $id, ['name' => 'Unknown']);
```
{% endraw %}

**3. Use Namespaced Keys**
{% raw %}
```php
// ❌ BAD: Key collision risk
$cache->set('1', $data);

// ✅ GOOD: Namespaced keys
$cache->set('user:1', $data);
$cache->set('product:1', $data);
```
{% endraw %}

**4. Cache Expensive Operations Only**
{% raw %}
```php
// ❌ BAD: Caching simple operations (no benefit)
$cache->remember('sum', 3600, fn() => 1 + 1);

// ✅ GOOD: Cache database queries or API calls
$cache->remember('users:report', 3600, function() {
    return DB::query('SELECT ... FROM users JOIN orders ...'); // Expensive
});
```
{% endraw %}

**5. Invalidate Cache on Updates**
{% raw %}
```php
// Update user in database
DB::table('users')->where('id', $id)->update($data);

// ✅ GOOD: Invalidate cache immediately
$cache->delete('user:' . $id);
$cache->delete('users:all'); // Also clear list cache
```
{% endraw %}

### Memory Management

**Calculate your memory needs:**
```
Total RAM = CACHE_SIZE × CACHE_VALUE_SIZE
Example: 10,000 entries × 4KB = 40MB RAM
```

**Increase cache size for more data:**
{% raw %}
```php
// In bootstrap/app.php
$cache = Cache::getInstance(
    size: 10000,     // 10k entries instead of 1024
    valueSize: 8192  // 8KB per entry instead of 4KB
);
```
{% endraw %}

**Warning:** Exceeding `CACHE_SIZE` limit means `set()` returns `false` (no automatic eviction).

### Cache vs Database Performance

| Operation | Database | Cached | Speedup |
|-----------|----------|--------|---------|
| Simple SELECT | 6,700 req/s | 520,000 req/s | 77x |
| JOIN query | 2,000 req/s | 520,000 req/s | 260x |
| Aggregation | 1,000 req/s | 520,000 req/s | 520x |

**Rule of thumb:** If a query runs more than 2x per minute, cache it!

---

## Next Steps

- [Usage Guide →](usage.html)
- [Drivers →](drivers.html)
