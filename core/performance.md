# Performance Optimization

Alphavel is already fast out of the box (520k+ req/s), but here's how to optimize it further.

---

## Production Checklist

### 1. Enable Route Caching

```bash
./alphavel route:cache
```

**Impact**: +10-20% faster routing

### 2. Use Raw Routes

```php
// Instead of this (20k req/s)
$router->get('/health', function () {
    return Response::make()->json(['status' => 'ok']);
});

// Use this (45k req/s)
$router->raw('/health', ['status' => 'ok'], 'application/json');
```

**Impact**: 2-3x faster for simple endpoints

### 3. Enable OPcache

Edit `php.ini`:

```ini
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000
opcache.validate_timestamps=0  # Production only!
```

**Impact**: +30-50% faster PHP execution

### 4. Use Connection Pooling

Already enabled by default in Database package:

```php
'pool' => [
    'min_connections' => 1,
    'max_connections' => 10,
],
```

**Impact**: 7x faster database queries

### 5. Optimize Swoole Workers

Edit `config/swoole.php`:

```php
'worker_num' => swoole_cpu_num() * 2,  # 2x CPU cores
'max_request' => 10000,  # Recycle worker after 10k requests
'enable_coroutine' => true,  # Async I/O
```

**Impact**: Better resource utilization

---

## Benchmarking

### Measure Performance

```bash
# Install Apache Bench
sudo apt install apache2-utils

# Test throughput
ab -n 10000 -c 100 http://localhost:9501/

# Test JSON endpoint
ab -n 10000 -c 100 http://localhost:9501/users

# Test raw route
ab -n 10000 -c 100 http://localhost:9501/health
```

### Expected Results

| Endpoint | Requests/sec | Latency (avg) |
|----------|--------------|---------------|
| Raw route `/health` | 45,000+ | 2.2ms |
| Static route `/users` | 20,000+ | 5ms |
| DB query | 15,000+ | 6.5ms |

---

## Optimization Techniques

### 1. Lazy Loading Providers

Service providers are lazy-loaded by default:

```php
// Only loaded when needed
class DatabaseServiceProvider extends ServiceProvider
{
    public $defer = true;
}
```

### 2. Request Pooling

Request objects are pooled and reused:

```php
// Automatic - no configuration needed
// Pool size: 1024 requests
```

**Impact**: -50% memory allocation

### 3. Fast Path for Simple Classes

Container uses fast path for classes without dependencies:

```php
// Fast path (no reflection)
$controller = new UserController();

// Slow path (reflection + DI)
$controller = $container->make(UserController::class);
```

---

## Monitoring

### Swoole Stats

```php
$router->raw('/metrics', function($request, $response) {
    $stats = swoole_get_server_stats();
    $response->header('Content-Type', 'application/json');
    $response->end(json_encode($stats));
}, 'application/json');
```

### Database Connection Pool

```php
$pool = DB::getPool();
echo "Active: {$pool->getActiveConnections()}\n";
echo "Idle: {$pool->getIdleConnections()}\n";
```

---

## Performance Comparison

| Framework | Requests/sec | Memory |
|-----------|--------------|--------|
| **Alphavel (Raw)** | **520,000** | 10MB |
| **Alphavel (Normal)** | **20,000** | 15MB |
| Laravel Octane | 12,000 | 25MB |
| Symfony Runtime | 8,000 | 30MB |
| Raw PHP-FPM | 3,000 | 5MB |

---

## Next Steps

- [Raw Routes →](raw-routes.md)
- [Connection Pooling →](../packages/database/connection-pooling.md)
- [Deployment →](../deployment/production.md)
