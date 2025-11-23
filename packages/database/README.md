# Database Package

High-performance database layer with **Laravel-style API** and Swoole coroutine support.

> 💡 **Laravel-compatible**: If you know Laravel's Query Builder, you already know Alphavel Database!

---

## 🚀 Features

- ✅ **🎯 Laravel-Style API** - 100% familiar syntax for Laravel developers
- ✅ **⚡ Persistent Connections** - +1,769% performance boost (enabled by default)
- ✅ **📦 Batch Queries** - New `findMany()` helper (+627% performance)
- ✅ **🔄 Connection Pooling** - Reuse connections across requests (zero overhead)
- ✅ **💾 Statement Cache** - Automatic prepared statement caching (+15-30%)
- ✅ **🚀 Query Builder Statement Cache (v1.3.0 - NEW!)** - Automatic QB caching (+557% performance)
- ✅ **🏗️ Query Builder** - Fluent interface identical to Laravel
- ✅ **🔐 Transactions** - ACID compliant with single-connection guarantee
- ✅ **🔒 Coroutine-Safe** - Context isolation per coroutine
- ✅ **♻️ Auto-Release** - Automatic connection release after request

---

## 📚 Documentation

- **[Laravel-Style Guide](LARAVEL_STYLE_GUIDE.md)** - Complete guide for Laravel developers
- **[Performance Optimizations](PERFORMANCE_OPTIMIZATIONS.md)** - Deep dive into +2,674% performance gains
- **[Configuration Template](.env.performance)** - Optimized .env settings

---

## 🎯 Quick Start (Laravel Developers)

```php
use Alphavel\Database\DB;

// 🔍 Queries (Laravel-style)
$users = DB::table('users')
    ->where('status', 'active')
    ->whereIn('role', ['admin', 'moderator'])
    ->orderBy('created_at', 'DESC')
    ->get();

// 📦 NEW: Batch queries (627% faster!)
$worlds = DB::findMany('World', [1, 2, 3, 4, 5]);
// SELECT * FROM World WHERE id IN (1,2,3,4,5)

// 🔄 Transactions
DB::transaction(function() {
    DB::execute('UPDATE accounts SET balance = balance - 100 WHERE id = ?', [1]);
    DB::execute('UPDATE accounts SET balance = balance + 100 WHERE id = ?', [2]);
});
```

---

## 🚀 Performance Optimizations

Alphavel Database includes **4 native performance optimizations**:

### 1. ⚡ Persistent Connections (+1,769%)
```php
// config/database.php - ENABLED BY DEFAULT
'persistent' => true,  // PDO::ATTR_PERSISTENT
```

**Benchmark**: 350 → 6,541 req/s (+1,769%) 🔥

### 2. 📦 Batch Queries (+627%)
```php
// ❌ BAD: 20 queries (312 req/s)
foreach ($ids as $id) {
    $world = DB::table('World')->where('id', $id)->first();
}

// ✅ GOOD: 1 query (2,269 req/s)
$worlds = DB::findMany('World', $ids);
```

**Benchmark**: 312 → 2,269 req/s (+627%) 🔥

### 3. 💾 Statement Cache (+15-30%)
Automatic prepared statement caching - **no configuration needed**!

### 3.1 🚀 Query Builder Statement Cache (v1.3.1 - FIXED!)
**Revolutionary feature**: Query Builder now caches compiled SQL automatically!

```php
// v1.2.0: 274 req/s
// v1.3.1: 1,109-1,434 req/s (+305-423%) 🔥
// Zero code changes required!

$results = DB::table('users')
    ->where('age', '>=', $minAge)
    ->where('city', $city)
    ->get();
```

**Performance Impact**:
- Low concurrency (10 conn): 274 → 1,434 req/s (**+423%** 🔥)
- High concurrency (100 conn): 274 → 1,109 req/s (**+305%** 🔥)
- Gap vs findOne(): Reduced from 23x to 4.5x (**80% reduction** ✅)

> **v1.3.1 Fix**: Caches SQL strings (not PDOStatements) to avoid race conditions in Swoole.

**How it works**: Queries with the same structure but different values reuse the same compiled SQL string. PDO prepare() is fast; SQL compilation is the expensive part.

**Optional management**:
```php
// View cache statistics
$stats = DB::getQueryBuilderCacheStats();

// Clear cache (useful for testing)
DB::clearQueryBuilderCache();

// Adjust max size (default: 500)
DB::setMaxQueryBuilderStatements(1000);
```

### 4. 🔄 Connection Pooling (+200-400%)
Swoole connection pool - **automatic** with configuration:

```env
# .env
SWOOLE_WORKER_NUM=4    # CPU cores
DB_POOL_MAX=20         # 4 * 5
DB_POOL_MIN=8          # 4 * 2
DB_PERSISTENT=true
```

### 📊 Combined Results
| Configuration | Req/s | Improvement |
|--------------|-------|-------------|
| Baseline | 350 | - |
| All optimizations | 9,712 | **+2,674%** 🚀 |

**📖 Full guide**: See [PERFORMANCE_OPTIMIZATIONS.md](PERFORMANCE_OPTIMIZATIONS.md)

---

## Installation

```bash
alpha package:add database
```

This automatically:
1. Installs the package
2. Publishes `config/database.php`
3. Registers the service provider

---

## Configuration

Edit `config/database.php`:

```php
<?php

return [
    'default' => env('DB_CONNECTION', 'mysql'),

    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', 3306),
            'database' => env('DB_DATABASE', 'alphavel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'options' => [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            ],
        ],

        'pgsql' => [
            'driver' => 'pgsql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', 5432),
            'database' => env('DB_DATABASE', 'alphavel'),
            'username' => env('DB_USERNAME', 'postgres'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8',
            'schema' => 'public',
        ],
    ],

    'pool' => [
        'min_connections' => env('DB_POOL_MIN', 1),
        'max_connections' => env('DB_POOL_MAX', 10),
        'wait_timeout' => env('DB_POOL_TIMEOUT', 3.0),
    ],
];
```

Set environment variables in `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=secret

# Connection Pool
DB_POOL_MIN=1
DB_POOL_MAX=10
DB_POOL_TIMEOUT=3.0
```

---

## Basic Usage

### Raw Queries

```php
<?php

use Alphavel\Database\DB;

// SELECT query
$users = DB::query('SELECT * FROM users WHERE active = ?', [1]);

// INSERT
$affected = DB::execute(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    ['John Doe', 'john@example.com']
);

// Get last insert ID
$id = DB::lastInsertId();

// UPDATE
$affected = DB::execute(
    'UPDATE users SET name = ? WHERE id = ?',
    ['Jane Doe', 1]
);

// DELETE
$affected = DB::execute('DELETE FROM users WHERE id = ?', [1]);
```

---

## Query Builder

### SELECT Queries

```php
// Get all
$users = DB::table('users')->get();

// Get first
$user = DB::table('users')->where('email', 'john@example.com')->first();

// Where conditions
$users = DB::table('users')
    ->where('active', 1)
    ->where('age', '>', 18)
    ->get();

// Multiple where
$users = DB::table('users')
    ->where([
        'active' => 1,
        'role' => 'admin'
    ])
    ->get();

// Or where
$users = DB::table('users')
    ->where('role', 'admin')
    ->orWhere('role', 'moderator')
    ->get();

// Where In
$users = DB::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();

// Where Null
$users = DB::table('users')
    ->whereNull('deleted_at')
    ->get();

// Order by
$users = DB::table('users')
    ->orderBy('created_at', 'DESC')
    ->get();

// Limit and offset
$users = DB::table('users')
    ->limit(10)
    ->offset(20)
    ->get();

// Select specific columns
$users = DB::table('users')
    ->select(['id', 'name', 'email'])
    ->get();
```

### INSERT

```php
// Single insert
$id = DB::table('users')->insert([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'created_at' => date('Y-m-d H:i:s')
]);

// Get last insert ID
echo "New user ID: {$id}";
```

### UPDATE

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update([
        'name' => 'Jane Doe',
        'updated_at' => date('Y-m-d H:i:s')
    ]);

echo "Updated {$affected} rows";
```

### DELETE

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->delete();

// Or direct
$affected = DB::table('users')->delete(1);
```

---

## Transactions

```php
use Alphavel\Database\DB;

try {
    DB::transaction(function () {
        // Insert user
        $userId = DB::table('users')->insert([
            'name' => 'John Doe',
            'email' => 'john@example.com'
        ]);

        // Insert user profile
        DB::table('profiles')->insert([
            'user_id' => $userId,
            'bio' => 'Software Developer'
        ]);

        // Insert log
        DB::table('logs')->insert([
            'user_id' => $userId,
            'action' => 'user_created'
        ]);
    });

    echo "Transaction successful!";
} catch (\Exception $e) {
    echo "Transaction failed: " . $e->getMessage();
}
```

---

## Connection Pooling

Connection pooling automatically reuses database connections across requests, dramatically improving performance.

### How it Works

```
Request 1:
  ↓ Get connection from pool
  ↓ Execute query
  ↓ Return connection to pool

Request 2:
  ↓ Reuse connection from pool (no reconnect!)
  ↓ Execute query
  ↓ Return connection to pool
```

### Performance Impact

| Method | Requests/sec | Connect Time |
|--------|--------------|--------------|
| **Connection Pool** | **20,000+** | 0ms (reused) |
| Traditional | 3,000 | 5-10ms per request |

**7x faster!** 🚀

### Configuration

```php
// config/database.php

'pool' => [
    'min_connections' => 1,    // Always keep 1 connection alive
    'max_connections' => 10,   // Max 10 concurrent connections
    'wait_timeout' => 3.0,     // Wait 3s for available connection
],
```

### Best Practices

```php
// ✅ DO: Release connection after use (automatic)
public function index()
{
    $users = DB::query('SELECT * FROM users');
    return Response::make()->json(['users' => $users]);
    // Connection automatically released here
}

// ❌ DON'T: Hold connections for long operations
public function process()
{
    $users = DB::query('SELECT * FROM users');
    
    foreach ($users as $user) {
        sleep(1); // ❌ Don't hold connection while sleeping!
        // Process user...
    }
}

// ✅ DO: Release early, process later
public function process()
{
    $users = DB::query('SELECT * FROM users');
    DB::release(); // Release connection immediately
    
    foreach ($users as $user) {
        sleep(1); // ✅ Connection already released
        // Process user...
    }
}
```

---

## Multiple Connections

```php
// Use specific connection
$users = DB::connection('pgsql')
    ->query('SELECT * FROM users');

// Query builder with specific connection
$orders = DB::connection('orders_db')
    ->table('orders')
    ->where('status', 'pending')
    ->get();
```

---

## Models (Coming Soon)

Active Record pattern for database tables:

```php
class User extends Model
{
    protected $table = 'users';
    protected $fillable = ['name', 'email'];
}

// Usage
$user = User::find(1);
$user->name = 'New Name';
$user->save();

$users = User::where('active', 1)->get();
```

---

## Performance Tips

1. **Use Connection Pooling** - Already enabled by default
2. **Use Prepared Statements** - Prevents SQL injection and improves performance
3. **Limit Results** - Use `limit()` to avoid fetching too much data
4. **Index Your Tables** - Create indexes on frequently queried columns
5. **Release Connections** - Call `DB::release()` after long operations

---

## API Reference

### DB Class

```php
// Raw queries
DB::query(string $sql, array $params = []): array
DB::execute(string $sql, array $params = []): int
DB::lastInsertId(): string|int

// Query builder
DB::table(string $table): QueryBuilder
DB::connection(string $name): Database

// Transactions
DB::transaction(Closure $callback): mixed
DB::beginTransaction(): void
DB::commit(): void
DB::rollBack(): void

// Connection management
DB::release(): void
```

### QueryBuilder Class

```php
// SELECT
->get(): array
->first(): ?array
->select(array $columns): self
->where(string $column, mixed $value): self
->whereIn(string $column, array $values): self
->whereNull(string $column): self
->orWhere(string $column, mixed $value): self
->orderBy(string $column, string $direction = 'ASC'): self
->limit(int $limit): self
->offset(int $offset): self

// INSERT
->insert(array $data): int  // Returns last insert ID

// UPDATE
->update(array $data): int  // Returns affected rows

// DELETE
->delete(int $id = null): int  // Returns affected rows
```

---

## Troubleshooting

### Connection Failed

```bash
# Check if database is running
docker ps | grep mysql

# Check credentials in .env
DB_HOST=127.0.0.1  # or 'mysql' for Docker
DB_USERNAME=root
DB_PASSWORD=secret
```

### Pool Timeout

```php
// Increase timeout in config/database.php
'pool' => [
    'wait_timeout' => 5.0,  // Increase from 3.0
],
```

### Too Many Connections

```php
// Reduce max connections
'pool' => [
    'max_connections' => 5,  // Reduce from 10
],
```

---

## 📚 Next Steps

- **[Laravel-Style Guide →](LARAVEL_STYLE_GUIDE.md)** - Complete Laravel-compatible API guide
- **[Performance Optimizations →](PERFORMANCE_OPTIMIZATIONS.md)** - +2,674% performance guide
- **[Configuration Template →](.env.performance)** - Optimized settings
- [Connection Pooling →](connection-pooling.md)
- [Query Builder →](query-builder.md)
- [Transactions →](transactions.md)

---

## 🆚 Laravel vs Alphavel

| Feature | Laravel | Alphavel | Compatible? |
|---------|---------|----------|-------------|
| `DB::query()` | ✅ | ✅ | ✅ 100% |
| `DB::table()` | ✅ | ✅ | ✅ 100% |
| `where()` | ✅ | ✅ | ✅ 100% |
| `whereIn()` | ✅ | ✅ | ✅ 100% |
| `join()` | ✅ | ✅ | ✅ 100% |
| `orderBy()` | ✅ | ✅ | ✅ 100% |
| `transaction()` | ✅ | ✅ | ✅ 100% |
| **`findMany()`** | ❌ | ✅ 🆕 | - |
| **Persistent Connections** | ❌ Manual | ✅ Default | - |
| **Performance** | Standard | +2,674% | 🚀 |

**Alphavel Database: Laravel-like API + Swoole Performance = ❤️**
