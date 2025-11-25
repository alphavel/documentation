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

- **PHP 8.4+** installed
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
    // Method signature: index(Request $request = null)
    // Request is auto-injected by container
    public function index()
    {
        // Helper methods from Controller base class
        return $this->success([
            'users' => [
                ['id' => 1, 'name' => 'John'],
                ['id' => 2, 'name' => 'Jane']
            ]
        ]);
        
        // Equivalent to:
        // return Response::success(['users' => ...], 200);
    }

    // Route parameters come AFTER Request (if present)
    public function show($id)
    {
        return $this->success([
            'user' => ['id' => $id, 'name' => "User {$id}"]
        ]);
    }

    // Request is auto-injected first, then route parameters
    public function store(Request $request)
    {
        // Get all input (POST + GET merged)
        $data = $request->all();
        
        // Get specific fields only
        $userData = $request->only(['name', 'email']);
        
        // Get with default value
        $name = $request->input('name', 'Guest');
        
        // Check if field exists and is not empty
        if ($request->filled('email')) {
            // email exists and has value
        }
        
        // Return success with 201 status
        return $this->success([
            'message' => 'User created',
            'user' => $data
        ], 201);
        
        // Or return error
        // return $this->error('Validation failed', 400, ['email' => 'Invalid']);
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

## Working with Request & Response

### Request Methods (Complete API)

```php
use Alphavel\Framework\Request;

public function example(Request $request)
{
    // === Getting Input ===
    
    // Get single input (POST or GET, POST takes priority)
    $name = $request->input('name', 'default');
    
    // Get all input (POST + GET merged)
    $all = $request->all();
    
    // Get only specific fields
    $data = $request->only(['name', 'email']);
    
    // Get all except specific fields
    $data = $request->except(['password', 'token']);
    
    // Check if field exists (even if empty)
    if ($request->has('email')) { }
    
    // Check if field exists AND has value (not empty)
    if ($request->filled('email')) { }
    
    // === Query Parameters (GET) ===
    
    // Get query parameter only (ignores POST)
    $page = $request->query('page', 1);
    
    // === Headers ===
    
    // Get header (case-insensitive, auto-converts _ to -)
    $contentType = $request->header('content-type');
    $auth = $request->header('authorization');
    
    // Get Bearer token from Authorization header
    $token = $request->bearerToken(); // Returns null if not present
    
    // === JSON Handling ===
    
    // Check if request is JSON
    if ($request->isJson()) {
        // Get all JSON data as array
        $data = $request->json();
        
        // Get specific JSON field (supports dot notation)
        $email = $request->json('user.email', 'default');
    }
    
    // === HTTP Method Checks ===
    
    if ($request->isPost()) { }
    if ($request->isGet()) { }
    if ($request->isPut()) { }
    if ($request->isDelete()) { }
    
    // Get method name
    $method = $request->getMethod(); // 'GET', 'POST', etc.
    
    // Get URI
    $uri = $request->getUri(); // '/api/users'
    
    return $this->success(['received' => $data]);
}
```

### Response Methods (Complete API)

```php
use Alphavel\Framework\Response;

// === Helper Methods in Controller ===

// Success response (status 200)
return $this->success(['data' => $value]);

// Success with custom status
return $this->success(['data' => $value], 201);

// Error response
return $this->error('Something went wrong', 400);

// Error with validation errors
return $this->error('Validation failed', 422, [
    'email' => 'Email is required',
    'name' => 'Name must be at least 3 characters'
]);

// === Response Factory Methods ===

// JSON response
return Response::make()->json(['key' => 'value'], 200);

// Success helper (includes 'status' field)
return Response::success(['user' => $user], 200);
// Output: {"status":"success","data":{"user":{...}}}

// Error helper (includes 'status' and 'message')
return Response::error('Not found', 404);
// Output: {"status":"error","message":"Not found"}

// Plain text
return Response::make()
    ->content('Hello World')
    ->status(200);

// Redirect
return Response::make()->redirect('/login', 302);

// Custom headers
return Response::make()
    ->json(['data' => 'value'])
    ->header('X-Custom-Header', 'value')
    ->status(200);
```

**⚡ Performance Tip for Beginners:**

- `Response::success()` and `Response::error()` add extra fields (`status`, `message`)
- For maximum performance, use `Response::make()->json()` directly
- Difference: ~0.01ms (negligible for most apps)

---

## Adding Packages

Alphavel uses a modular package system. Add only what you need:

```bash
# Database (MySQL/PostgreSQL with Swoole connection pooling)
composer require alphavel/database

# Cache (Swoole\Table in-memory, no Redis needed)
composer require alphavel/cache

# Logging (file-based with rotation)
composer require alphavel/logging

# Events (async event dispatcher)
composer require alphavel/events

# Validation (Laravel-style rules)
composer require alphavel/validation
```

Or use the interactive wizard:

```bash
php alpha package:add database
```

After adding, packages are auto-discovered and ready to use!

---

## Using Database

After `composer require alphavel/database`:

**1. Configure `.env`:**

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=secret

# Connection Pool (optional, defaults shown)
DB_POOL_MIN=1
DB_POOL_MAX=10
DB_POOL_TIMEOUT=3.0
```

**2. Use in your controller:**

```php
<?php

use Alphavel\Database\DB;
use Alphavel\Framework\Controller;

class UserController extends Controller
{
    // Query Builder (6,700 req/s) - RECOMMENDED for APIs
    public function index()
    {
        $users = DB::table('users')
            ->where('active', 1)
            ->orderBy('created_at', 'DESC')
            ->get();
        
        return $this->success(['users' => $users]);
    }
    
    // Find by ID
    public function show($id)
    {
        $user = DB::table('users')
            ->where('id', $id)
            ->first();
        
        if (!$user) {
            return $this->error('User not found', 404);
        }
        
        return $this->success(['user' => $user]);
    }
    
    // Raw SQL (8,000+ req/s) - For complex queries
    public function search(Request $request)
    {
        $term = $request->input('q', '');
        
        $users = DB::query(
            'SELECT * FROM users WHERE name LIKE ? OR email LIKE ? LIMIT 20',
            ["%{$term}%", "%{$term}%"]
        );
        
        return $this->success(['users' => $users]);
    }
    
    // Transactions (ACID compliant)
    public function transfer(Request $request)
    {
        $fromId = $request->input('from');
        $toId = $request->input('to');
        $amount = $request->input('amount');
        
        try {
            DB::transaction(function () use ($fromId, $toId, $amount) {
                // Deduct from sender
                DB::execute(
                    'UPDATE accounts SET balance = balance - ? WHERE id = ?',
                    [$amount, $fromId]
                );
                
                // Add to receiver
                DB::execute(
                    'UPDATE accounts SET balance = balance + ? WHERE id = ?',
                    [$amount, $toId]
                );
            });
            
            return $this->success(['message' => 'Transfer successful']);
            
        } catch (\Exception $e) {
            return $this->error('Transfer failed: ' . $e->getMessage(), 400);
        }
    }
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

- [Routing →](routing.html)
- [Controllers →](controllers.html)
- [CLI Commands →](cli-commands.html)
- [Performance →](performance.html)
- [Database →](../packages/database/)

---

## Need Help?

- 📖 [Full Documentation](../)
- 💬 [Discord Community](https://discord.gg/alphavel)
- 🐛 [Issue Tracker](https://github.com/alphavel/alphavel/issues)
