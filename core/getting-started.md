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
./alphavel serve
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
./alphavel package:add database

# Cache (Redis/Memcached)
./alphavel package:add cache

# Logging (Monolog)
./alphavel package:add logging

# Events
./alphavel package:add events

# Validation
./alphavel package:add validation
```

After adding, packages are auto-discovered and ready to use!

---

## Using Database

After `./alphavel package:add database`:

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
./alphavel serve

# Start with custom host/port
./alphavel serve --host=0.0.0.0 --port=8000

# Package management
./alphavel package:add database
./alphavel package:discover

# Performance (production)
./alphavel route:cache
./alphavel route:clear
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

## Next Steps

- [Routing →](routing.md)
- [Controllers →](controllers.md)
- [Database →](../packages/database/getting-started.md)
- [Performance →](performance.md)

---

## Need Help?

- 📖 [Full Documentation](../README.md)
- 💬 [Discord Community](https://discord.gg/alphavel)
- 🐛 [Issue Tracker](https://github.com/alphavel/alphavel/issues)
