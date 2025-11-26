---
layout: default
title: Performance Architecture
---

# Performance Architecture 🚀

Alphavel is designed with a unique "Hot Path" architecture that allows it to bypass traditional framework overhead for critical operations, achieving performance levels comparable to raw PHP scripts while maintaining a developer-friendly API.

## The "Hot Path" Philosophy

In most frameworks, every database query goes through layers of abstraction:
1.  Query Builder (AST construction)
2.  Grammar Compilation (SQL generation)
3.  Connection Management
4.  PDO Execution
5.  Result Hydration (Array/Object creation)

Alphavel identifies "Hot Paths" — operations that are executed frequently and require maximum speed — and provides optimized shortcuts.

### 1. Global Statement Cache

Alphavel implements a **Global Statement Cache** that is thread-safe and shared across requests within the same Swoole worker.

*   **Traditional:** `prepare()` is called on every request (~100-200µs overhead).
*   **Alphavel:** `prepare()` is called once per worker lifetime. Subsequent requests reuse the prepared statement handle.

This is automatically handled by the `QueryBuilder` and `DB` facade.

### 2. Read Connection Singleton

For read-only operations (SELECT), Alphavel uses a singleton connection that bypasses the connection pool lookup overhead.

*   **Write/Transaction:** Uses an isolated connection from the pool (safe for ACID).
*   **Read:** Uses a shared, lock-free connection (safe for concurrent SELECTs).

### 3. Native Query Builder Optimization

The `QueryBuilder` in Alphavel is "Turbo-charged". It automatically detects if it's running inside a transaction.

```php
// Standard Laravel-style code:
$users = DB::table('users')->where('active', 1)->get();
```

**Under the hood in Alphavel:**
1.  Checks `DB::inTransaction()`.
2.  **If False (Read Path):**
    *   Compiles SQL (or retrieves from SQL cache).
    *   Retrieves prepared statement from **Global Cache**.
    *   Executes on **Read Singleton Connection**.
    *   Returns raw array (no object hydration overhead).
3.  **If True (Write Path):**
    *   Uses isolated connection for current coroutine.
    *   Ensures transaction safety.

This means you get **raw PDO performance** with **Eloquent-like syntax**.

## Infrastructure Philosophy: "Less is More"

Our benchmarks proved that aggressive server tuning (JIT, Huge Pages) often degrades performance for I/O-bound web applications. Alphavel's production architecture focuses on simplicity:

*   **No JIT:** Disabled by default. The PHP 8.4 JIT adds complexity and compilation overhead that doesn't benefit typical database-driven apps.
*   **Root/Default User:** We recommend running containers as the default user to avoid permission overheads, unless strict compliance requires otherwise.
*   **OPcache:** We rely solely on `opcache.validate_timestamps=0` for code caching.

## Benchmarks

| Operation | Alphavel (Hot Path) | Traditional Framework | Gain |
| :--- | :--- | :--- | :--- |
| **Single Read** | ~17,000 req/s | ~6,500 req/s | **+161%** |
| **Batch Read** | ~10,500 req/s | ~3,000 req/s | **+250%** |
| **Complex Query** | ~8,700 req/s | ~1,200 req/s | **+625%** |

## Best Practices for Maximum Speed

1.  **Use `DB::findOne($table, $id)`:** The fastest way to fetch a single record by ID.
2.  **Use `DB::batchFetch($table, $ids)`:** The fastest way to fetch multiple records by ID.
3.  **Avoid `DB::raw()`:** Raw queries bypass some optimizations. Use the Query Builder methods whenever possible.
4.  **Keep Transactions Short:** Long transactions lock the connection to a specific coroutine, preventing it from being returned to the pool.
