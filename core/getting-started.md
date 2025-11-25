---
layout: default
title: Getting Started
---

# Getting Started

## Quick Start Guide

This guide will help you create your first Alphavel application in minutes.

---

## Prerequisites

Before starting, ensure you have:

- **PHP 8.1+** installed
- **Composer** installed
- **Swoole extension** (or use Docker)

---

## Installation

### Option 1: Docker (Recommended)

Perfect for development without installing Swoole locally:

```bash
# Create new project
composer create-project alphavel/skeleton my-app
cd my-app

# Start development server (installs everything automatically)
docker-compose -f docker-compose.dev.yml up
```

Access your app at: http://localhost:9999

### Option 2: Local Swoole

If you have Swoole installed:

```bash
# Create new project
composer create-project alphavel/skeleton my-app
cd my-app

# Start server
php alpha serve
```

Access your app at: http://localhost:9501

---

## Project Structure

```
my-app/
├── app/
│   └── Controllers/
│       └── ExampleController.php
├── routes/
│   └── api.php
├── config/
│   ├── app.php
│   └── swoole.php
├── bootstrap/
│   └── app.php
├── public/
│   └── index.php
├── storage/
│   ├── logs/
│   └── cache/
└── alphavel (CLI)
```

---

## Your First Route

Edit `routes/api.php`:

```php
<?php

use Alphavel\Framework\Router;
use Alphavel\Framework\Response;

/** @var Router $router */

// Raw route (ultra-fast, zero overhead)
$router->raw('/ping', 'pong');

// JSON response
$router->get('/users', function () {
    return Response::make()->json([
        'users' => [
            ['id' => 1, 'name' => 'John Doe'],
            ['id' => 2, 'name' => 'Jane Smith']
        ]
    ]);
});

// Route with parameter
$router->get('/users/{id}', function ($id) {
    return Response::make()->json([
        'user' => ['id' => $id, 'name' => 'User ' . $id]
    ]);
});
```

Test it:

```bash
curl http://localhost:9501/ping
# Output: pong

curl http://localhost:9501/users
# Output: {"users":[{"id":1,"name":"John Doe"},{"id":2,"name":"Jane Smith"}]}
```

---

## Your First Controller

Create `app/Controllers/UserController.php`:

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;

class UserController extends Controller
{
    public function index()
    {
        return Response::make()->json([
            'users' => [
                ['id' => 1, 'name' => 'John'],
                ['id' => 2, 'name' => 'Jane']
            ]
        ]);
    }

    public function show(int $id)
    {
        return Response::make()->json([
            'user' => ['id' => $id, 'name' => "User {$id}"]
        ]);
    }

    public function store(Request $request)
    {
        $data = $request->all();
        
        return Response::make()->json([
            'message' => 'User created',
            'user' => $data
        ], 201);
    }
}
```

Register routes in `routes/api.php`:

```php
$router->get('/users', 'App\Controllers\UserController@index');
$router->get('/users/{id}', 'App\Controllers\UserController@show');
$router->post('/users', 'App\Controllers\UserController@store');
```

---

## Adding Packages

Alphavel uses a modular package system. Add only what you need:

```bash
# Database (MySQL/PostgreSQL with connection pooling)
alpha package:add database

# Cache (Redis/Memcached)
alpha package:add cache

# Logging (Monolog)
alpha package:add logging

# Events
alpha package:add events

# Validation
alpha package:add validation
```

After adding, packages are auto-discovered and ready to use!

---

## Using Database

After `alpha package:add database`:

```php
<?php

use Alphavel\Database\DB;

// Configure in .env
DB_HOST=mysql
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=secret

// Use in your controller
public function index()
{
    $users = DB::query('SELECT * FROM users WHERE active = ?', [1]);
    
    return Response::make()->json(['users' => $users]);
}

// Query Builder
$user = DB::table('users')
    ->where('email', 'john@example.com')
    ->first();

// Transaction
DB::transaction(function () {
    DB::execute('INSERT INTO users (name) VALUES (?)', ['John']);
    DB::execute('INSERT INTO logs (action) VALUES (?)', ['user_created']);
});
```

---

## Configuration

### Environment Variables

Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Edit `.env`:

```env
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:9501

# Database (if using database package)
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=alphavel
DB_USERNAME=root
DB_PASSWORD=

# Swoole
SWOOLE_HOST=0.0.0.0
SWOOLE_PORT=9501
SWOOLE_WORKERS=4
```

### Config Files

Edit `config/app.php`:

```php
return [
    'name' => env('APP_NAME', 'Alphavel'),
    'env' => env('APP_ENV', 'production'),
    'debug' => env('APP_DEBUG', false),
    'url' => env('APP_URL', 'http://localhost'),
    'timezone' => 'UTC',
    
    'providers' => [
        // Auto-discovered by package system
    ],
];
```

---

## CLI Commands

```bash
# Start server
php alpha serve

# Start with custom host/port
php alpha serve --host=0.0.0.0 --port=8000

# Package management
php alpha package:add database
php alpha package:discover

# Performance (production)
php alpha route:cache
php alpha route:clear
```

---

## Docker Commands

```bash
# Development
docker-compose -f docker-compose.dev.yml up
docker-compose -f docker-compose.dev.yml down

# Production
docker-compose up -d
docker-compose logs -f app
docker-compose exec app bash

# Rebuild
docker-compose build --no-cache
```

---

## Advanced Features

### Helper Functions

Alphavel includes Laravel-style helper functions:

```php
// Application helpers
$app = app();                    // Get Application instance
$router = app('router');         // Resolve from container
$value = config('app.name');     // Get config value
$env = env('APP_ENV', 'local');  // Get environment variable

// Array helpers
$value = array_get($arr, 'key.nested', 'default');
$filtered = array_only($arr, ['id', 'name']);
$removed = array_except($arr, ['password']);

// Data helpers
$value = data_get($object, 'property.nested');
$empty = blank($value);          // Check if empty
$filled = filled($value);        // Check if not empty

// String helpers
str_contains($str, 'needle');    // Check if contains
str_starts_with($str, 'prefix'); // Check if starts with

// Date helpers
$now = now();                    // Current datetime
$today = today();                // Current date
```

---

### Facades

Create static interfaces to services:

```php
// Create a facade (app/Facades/Cache.php)
namespace App\Facades;

use Alphavel\Framework\Facade;

class Cache extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}

// Usage
use App\Facades\Cache;

Cache::put('key', 'value', 3600);
$value = Cache::get('key');
```

**Performance:** Zero overhead - resolved once per request via singleton container.

---

### Package Auto-Discovery

Alphavel automatically discovers packages from `composer.json`:

```json
{
    "extra": {
        "alphavel": {
            "providers": [
                "Vendor\\Package\\ServiceProvider"
            ],
            "aliases": {
                "MyFacade": "Vendor\\Package\\Facades\\MyFacade"
            }
        }
    }
}
```

**Caching:** Results are cached in `storage/cache/providers.php` for performance.

---

### Middleware Pipeline

Create custom middleware:

```php
// app/Middleware/LogRequests.php
namespace App\Middleware;

use Alphavel\Request;
use Closure;

class LogRequests
{
    public function handle(Request $request, Closure $next)
    {
        // Before request
        $start = microtime(true);
        
        $response = $next($request);
        
        // After request
        $duration = microtime(true) - $start;
        error_log("Request to {$request->getUri()} took {$duration}s");
        
        return $response;
    }
}

// routes/api.php
$router->get('/users', [UserController::class, 'index'])
    ->middleware(LogRequests::class);
```

---

### Request Pooling

Alphavel automatically reuses Request objects across requests:

```php
// Automatic - no configuration needed
// Request objects are pooled (max 1024)
// 50% less memory allocation
// Faster garbage collection
```

**Performance Impact:**
- 50% reduction in memory allocation
- Faster GC (fewer objects to trace)
- Zero configuration required

---

### Deferred Service Providers

Service providers can be lazy-loaded:

```php
namespace App\Providers;

use Alphavel\Framework\ServiceProvider;

class HeavyServiceProvider extends ServiceProvider
{
    // Only loads when needed
    protected bool $defer = true;
    
    public function register(): void
    {
        $this->app->singleton('heavy', function () {
            return new HeavyService();
        });
    }
}
```

**Impact:** 40% faster boot time on average.

---

### Reflection Caching

Container caches reflection data for autowiring:

```php
// First resolution: Reflection runs
$controller = app(UserController::class);

// Subsequent resolutions: Uses cached data
$controller = app(UserController::class); // ⚡ Instant
```

**Cache types:**
- Constructor parameters cached per class
- Simple classes (no dependencies) marked for fast path
- Memory-efficient: Only reflection data cached, not instances

---

### Advanced CLI Commands

```bash
# Code generation
php alpha make:controller UserController --resource
php alpha make:model User
php alpha make:middleware AuthMiddleware
php alpha make:migration create_users_table
php alpha make:seeder UserSeeder
php alpha make:test UserTest
php alpha make:request StoreUserRequest
php alpha make:command SendEmails

# Optimization
php alpha optimize              # Cache config + routes
php alpha optimize:clear        # Clear all caches
php alpha config:cache          # Cache configuration
php alpha config:clear          # Clear config cache
php alpha route:list            # List all routes
php alpha ide-helper            # Generate IDE helper files

# Views
php alpha cache:clear           # Clear application cache
php alpha facade:clear          # Clear facade cache
```

---

## Next Steps

- [Routing →](routing.md)
- [Controllers →](controllers.md)
- [CLI Commands →](cli-commands.md)
- [Performance →](performance.md)
- [Database →](../packages/database/getting-started.md)
- [Performance →](performance.md)

---

## Need Help?

- 📖 [Full Documentation](../README.md)
- 💬 [Discord Community](https://discord.gg/alphavel)
- 🐛 [Issue Tracker](https://github.com/alphavel/alphavel/issues)
