# Database Package

High-performance database layer with connection pooling for MySQL and PostgreSQL.

---

## Features

- ✅ **Connection Pooling** - Reuse connections across requests
- ✅ **Query Builder** - Fluent interface for building queries
- ✅ **Raw SQL** - Execute custom queries
- ✅ **Transactions** - ACID compliant
- ✅ **Multiple Connections** - Support for multiple databases
- ✅ **PDO** - Standard PHP Data Objects

---

## Installation

```bash
./alphavel package:add database
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

## Next Steps

- [Connection Pooling →](connection-pooling.md)
- [Query Builder →](query-builder.md)
- [Transactions →](transactions.md)
- [Models →](models.md)
