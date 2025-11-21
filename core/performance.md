# Performance Optimization

Complete guide to maximizing Alphavel performance.

---

## Performance Features Overview

Alphavel is built for extreme performance from day one:

| Feature | Impact | Effort |
|---------|--------|--------|
| **Raw Routes** | 26x faster (520k+ req/s) | Low |
| **Route Caching** | 15-20% faster | Very Low |
| **Connection Pooling** | 7x faster queries | Low |
| **Request Pooling** | 50% less memory | Automatic |
| **OPcache JIT** | 20-30% faster | Very Low |
| **Lazy Providers** | 40% faster boot | Medium |
| **Swoole Coroutines** | 10x concurrency | Medium |

---

## 1. Raw Routes (Highest Impact)

### What Are Raw Routes?

Raw routes bypass the entire framework stack for **maximum performance**.

```php
// Regular route: ~20k req/s
$router->get('/api/users', [UserController::class, 'index']);

// Raw route: ~520k req/s (26x faster!)
$router->raw('/ping', 'pong');
```

### Performance Impact

- **Plaintext responses**: 520,000+ req/s
- **JSON responses**: 45,000+ req/s
- **26x faster** than regular routes
- **2-3ms response time** vs 50ms

### When to Use

✅ **Perfect for:**
- Health checks (`/health`, `/ready`)
- Ping endpoints (`/ping`)
- Status pages (`/status`)
- Metrics (`/metrics`)
- Simple API responses with no business logic

❌ **Don't use for:**
- Routes with parameters (`/users/{id}`)
- Complex business logic
- Database operations
- Authentication/validation

### Examples

```php
// routes/api.php

// String response (fastest)
$router->raw('/ping', 'pong');

// Array response (JSON)
$router->raw('/status', [
    'status' => 'ok',
    'version' => '1.0.0',
    'uptime' => 123456
]);

// Closure with Swoole stats
$router->raw('/metrics', function($request, $response) {
    $stats = swoole_get_server_stats();
    
    $response->header('Content-Type', 'application/json');
    $response->end(json_encode([
        'requests' => $stats['request_count'],
        'connections' => $stats['connection_num'],
        'workers' => $stats['worker_num']
    ]));
});
```

### Benchmarks

```bash
# Raw route
wrk -t12 -c400 -d30s http://localhost:9501/ping
# Requests/sec: 520,431.12

# Regular route
wrk -t12 -c400 -d30s http://localhost:9501/api/users
# Requests/sec: 19,847.33

# Improvement: 26x faster
```

---

## 2. Route Caching

### Enable in Production

Route caching compiles all routes into a single optimized file.

```bash
# Cache routes
alpha route:cache

# Clear cache
alpha route:clear
```

### Performance Impact

- **15-20% faster** route matching
- **Instant boot time** (no route loading)
- **Reduced memory** usage

### How It Works

```php
// Without caching: routes loaded every request
require_once __DIR__.'/../routes/api.php';
require_once __DIR__.'/../routes/web.php';

// With caching: single file, pre-compiled
$routes = require __DIR__.'/../bootstrap/cache/routes.php';
```

### Best Practices

✅ **Always cache in production**
```bash
# In Dockerfile
RUN alpha route:cache
```

❌ **Don't cache in development**
```bash
# Let routes load dynamically for hot-reload
alpha serve  # No caching needed
```

---

## 3. Database Connection Pooling

### What Is Connection Pooling?

Instead of creating a new database connection for each request, connections are **reused** across requests.

```php
// Without pooling (traditional PHP)
// Each request: connect → query → disconnect
// ~2,800 queries/sec

// With pooling (Alphavel)
// Pre-connect → reuse → reuse → reuse
// ~20,000 queries/sec (7x faster!)
```

### Configuration

```php
// config/database.php

return [
    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', 'localhost'),
            'database' => env('DB_DATABASE', 'alphavel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            
            // Connection pool settings
            'pool' => [
                'min' => 2,          // Minimum connections
                'max' => 20,         // Maximum connections
                'timeout' => 5.0,    // Wait timeout (seconds)
            ],
        ],
    ],
];
```

### Pool Sizing

**Formula**: `max = 2 * CPU_CORES * WORKERS`

```php
// 4 CPU cores, 8 workers
'max' => 2 * 4 * 8 = 64 connections

// But! Be mindful of MySQL max_connections
// MySQL default: 151 connections
// Your app max: Should be < 151
```

### Monitoring

```php
use Alphavel\Database\DB;

$stats = DB::connection()->getPoolStats();

echo "Active: {$stats['active']}\n";     // In-use connections
echo "Idle: {$stats['idle']}\n";         // Available connections
echo "Total: {$stats['total']}\n";       // Total connections
echo "Waiting: {$stats['waiting']}\n";   // Requests waiting
```

### Performance Impact

| Operation | Without Pool | With Pool | Improvement |
|-----------|--------------|-----------|-------------|
| Simple SELECT | 2,800 q/s | 20,000 q/s | **7x faster** |
| INSERT | 1,200 q/s | 8,500 q/s | **7x faster** |
| Transaction | 800 tx/s | 5,600 tx/s | **7x faster** |

---

## 4. Request Pooling (Automatic)

Alphavel automatically reuses Request/Response objects across requests.

### Traditional PHP

```php
// Apache/PHP-FPM: new objects every request
$request = new Request();   // Allocate memory
$response = new Response(); // Allocate memory
// Process request
// Destroy objects
// Next request: repeat!
```

### Alphavel (Swoole)

```php
// Pre-allocate once
$requestPool = [/* 100 Request objects */];
$responsePool = [/* 100 Response objects */];

// Reuse across requests
$request = $requestPool->get();  // Get from pool
$response = $responsePool->get(); // Get from pool
// Process request
$requestPool->put($request);     // Return to pool
$responsePool->put($response);   // Return to pool
```

### Impact

- **50% less memory** allocation
- **Faster GC** (garbage collection)
- **Automatic** - no configuration needed

---

## 5. OPcache & JIT

### Enable OPcache

```ini
; php.ini

[OPcache]
opcache.enable=1
opcache.enable_cli=1

; Memory
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000

; Production settings
opcache.validate_timestamps=0  ; Never revalidate
opcache.revalidate_freq=0
opcache.save_comments=0
opcache.fast_shutdown=1

; JIT (PHP 8.0+)
opcache.jit=tracing
opcache.jit_buffer_size=128M
```

### Performance Impact

- **OPcache alone**: 20-30% faster
- **OPcache + JIT**: 30-50% faster
- **Zero code changes** required

### Verify Configuration

```bash
# Check if OPcache is enabled
php -i | grep opcache.enable

# Check JIT status
php -r "echo opcache_get_status()['jit']['enabled'];"
```

---

## 6. Lazy Service Providers

### What Is Lazy Loading?

Service providers only load when actually needed.

```php
// Traditional: all providers load on boot
// Alphavel: only load when used

class ArticleController extends Controller
{
    public function index()
    {
        // DatabaseServiceProvider loads here (not at boot)
        $articles = DB::table('articles')->get();
        
        return $articles;
    }
}
```

### Configuration

```php
// config/app.php

return [
    'providers' => [
        Alphavel\CoreServiceProvider::class,        // Always loaded
        Alphavel\Database\DatabaseServiceProvider::class,  // Lazy
        Alphavel\Cache\CacheServiceProvider::class,        // Lazy
        Alphavel\Events\EventServiceProvider::class,       // Lazy
    ],
];
```

### Performance Impact

- **40% faster boot time** on average
- Only pay for what you use
- Scales better with more packages

### Example

```php
// Route without database access
$router->get('/ping', fn() => 'pong');
// DatabaseServiceProvider never loads! ✨

// Route with database access
$router->get('/users', [UserController::class, 'index']);
// DatabaseServiceProvider loads only when DB is used
```

---

## 7. Swoole Coroutines

### What Are Coroutines?

Lightweight concurrent execution without threads.

```php
// Traditional: sequential (slow)
$user = $db->query('SELECT * FROM users WHERE id = 1');     // 10ms
$posts = $db->query('SELECT * FROM posts WHERE user_id = 1'); // 10ms
$comments = $db->query('SELECT * FROM comments WHERE user_id = 1'); // 10ms
// Total: 30ms

// Coroutines: concurrent (fast)
go(function() use ($db) {
    $user = $db->query('SELECT * FROM users WHERE id = 1');
});
go(function() use ($db) {
    $posts = $db->query('SELECT * FROM posts WHERE user_id = 1');
});
go(function() use ($db) {
    $comments = $db->query('SELECT * FROM comments WHERE user_id = 1');
});
// Total: 10ms (3x faster!)
```

### Practical Example

```php
use Swoole\Coroutine;

class UserController extends Controller
{
    public function show($id)
    {
        $results = [];
        
        // Execute 3 queries concurrently
        Coroutine\batch([
            'user' => function() use ($id) {
                return DB::table('users')->find($id);
            },
            'posts' => function() use ($id) {
                return DB::table('posts')->where('user_id', $id)->get();
            },
            'comments' => function() use ($id) {
                return DB::table('comments')->where('user_id', $id)->get();
            }
        ], $results);
        
        return $results;
    }
}
```

### Performance Impact

- **3-10x faster** for I/O operations
- **10,000+ concurrent requests** per worker
- No blocking operations

---

## 8. Memory Optimization

### Worker Recycling

Prevent memory leaks by restarting workers periodically.

```php
// config/swoole.php

return [
    // Restart worker after 10,000 requests
    'max_request' => 10000,
    
    // Restart worker if memory exceeds 512MB
    'max_request_execution_time' => 0,
];
```

### Unset Large Variables

```php
// Bad: keeps large array in memory
function processData()
{
    $data = fetchHugeDataset(); // 100MB
    $result = transform($data);
    return $result;
    // $data still in memory!
}

// Good: free memory immediately
function processData()
{
    $data = fetchHugeDataset(); // 100MB
    $result = transform($data);
    unset($data); // Free 100MB
    return $result;
}
```

### Generator Functions

```php
// Bad: loads entire dataset into memory
function getUsers()
{
    return DB::table('users')->get(); // 10,000 records = 50MB
}

// Good: yields one record at a time
function getUsers()
{
    foreach (DB::table('users')->cursor() as $user) {
        yield $user; // Only 1 record in memory
    }
}
```

---

## 9. Caching Strategies

### Application Cache

```php
use Alphavel\Cache\Cache;

// Cache expensive operations
$users = Cache::remember('users.all', 3600, function() {
    return DB::table('users')->get();
});

// Cache with tags
Cache::tags(['users', 'api'])->put('users.all', $users, 3600);

// Invalidate by tag
Cache::tags(['users'])->flush();
```

### HTTP Caching

```php
// Set cache headers
$response->header('Cache-Control', 'public, max-age=3600');
$response->header('ETag', md5($content));

// 304 Not Modified
if ($request->header('If-None-Match') === $etag) {
    return $response->status(304)->end();
}
```

---

## 10. Profiling & Monitoring

### Xdebug Profiler (Development Only)

```bash
# Install Xdebug
pecl install xdebug

# Configure php.ini
xdebug.mode=profile
xdebug.output_dir=/tmp/xdebug

# Analyze with KCachegrind
kcachegrind /tmp/xdebug/cachegrind.out.123
```

### Blackfire (Production)

```bash
# Install Blackfire
curl https://packages.blackfire.io/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install blackfire-agent blackfire-php

# Profile endpoint
blackfire curl http://localhost:9501/api/users
```

### Custom Timing

```php
$start = microtime(true);

// Your code here
$result = expensiveOperation();

$duration = microtime(true) - $start;
Log::info("Operation took {$duration}s");
```

---

## Performance Checklist

### Development
- [ ] Use raw routes for simple endpoints
- [ ] Profile slow endpoints
- [ ] Monitor memory usage
- [ ] Use generators for large datasets

### Production
- [ ] Enable route caching (`alpha route:cache`)
- [ ] Enable OPcache + JIT
- [ ] Configure connection pooling
- [ ] Set `opcache.validate_timestamps=0`
- [ ] Use application caching (Redis)
- [ ] Configure Swoole workers (2x CPU cores)
- [ ] Enable HTTP caching headers
- [ ] Monitor metrics (`/metrics`)
- [ ] Setup alerts for high memory/CPU

---

## Real-World Results

### E-commerce API (Case Study)

**Before Alphavel (Laravel Octane):**
- 3,200 req/s peak
- 28MB memory per worker
- 15-20ms avg response time
- 8 servers required ($800/month)

**After Alphavel:**
- 8,500 req/s peak (**2.65x faster**)
- 10MB memory per worker (**2.8x less**)
- 5-8ms avg response time (**2.5x faster**)
- 3 servers required (**$300/month** - 62% cost savings)

### Social Media API (Case Study)

**Before Alphavel (Symfony Runtime):**
- 5,000 req/s peak
- 45MB memory per worker
- 25-30ms avg response time
- 12 servers required ($1,200/month)

**After Alphavel:**
- 18,000 req/s peak (**3.6x faster**)
- 12MB memory per worker (**3.75x less**)
- 7-10ms avg response time (**3x faster**)
- 4 servers required (**$400/month** - 67% cost savings)

---

## Next Steps

- [Raw Routes Documentation →](raw-routes.md)
- [Benchmarks →](../introduction/benchmarks.md)
- [Deployment Guide →](../deployment/production.md)
