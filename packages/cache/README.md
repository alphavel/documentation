---
layout: default
title: README
---

# Cache Package

High-performance caching layer with Redis and Memcached support.

---

## Installation

```bash
alpha package:add cache
```

---

## Configuration

Edit `config/cache.php`:

```php
return [
    'default' => env('CACHE_DRIVER', 'redis'),

    'stores' => [
        'redis' => [
            'driver' => 'redis',
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'port' => env('REDIS_PORT', 6379),
            'password' => env('REDIS_PASSWORD', null),
            'database' => env('REDIS_DB', 0),
        ],

        'memcached' => [
            'driver' => 'memcached',
            'servers' => [
                ['host' => '127.0.0.1', 'port' => 11211, 'weight' => 100],
            ],
        ],
    ],
];
```

---

## Usage

```php
use Alphavel\Cache\Cache;

// Set cache (expires in 3600 seconds)
Cache::set('user:1', ['name' => 'John'], 3600);

// Get cache
$user = Cache::get('user:1');

// Get with default
$user = Cache::get('user:1', ['name' => 'Guest']);

// Check if exists
if (Cache::has('user:1')) {
    // ...
}

// Delete
Cache::delete('user:1');

// Clear all
Cache::clear();

// Remember (get or set)
$users = Cache::remember('users:all', 3600, function () {
    return DB::query('SELECT * FROM users');
});
```

---

## Next Steps

- [Usage Guide →](usage.md)
- [Drivers →](drivers.md)
