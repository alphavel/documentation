---
layout: default
title: Routing
---

# Routing

Alphavel provides a powerful and fast routing system with two types of routes: **Raw Routes** (zero overhead) and **Standard Routes** (full framework stack).

---

## Basic Routing

Routes are defined in `routes/api.php`:

```php
<?php

use Alphavel\Framework\Router;
use Alphavel\Framework\Response;

/** @var Router $router */

// GET route
$router->get('/users', function () {
    return Response::make()->json(['users' => []]);
});

// POST route
$router->post('/users', function (Request $request) {
    $data = $request->all();
    return Response::make()->json(['user' => $data], 201);
});

// PUT route
$router->put('/users/{id}', function ($id, Request $request) {
    return Response::make()->json(['id' => $id]);
});

// DELETE route
$router->delete('/users/{id}', function ($id) {
    return Response::make()->json(['deleted' => true]);
});

// PATCH route
$router->patch('/users/{id}', function ($id) {
    return Response::make()->json(['patched' => true]);
});
```

---

## Raw Routes (Ultra-Fast)

Raw routes bypass the entire framework stack for maximum performance:

```php
// Plain text (520k+ req/s)
$router->raw('/ping', 'pong');

// JSON (45k+ req/s)
$router->raw('/health', ['status' => 'ok'], 'application/json');

// Custom closure with Swoole access
$router->raw('/metrics', function($request, $response) {
    $stats = swoole_get_server_stats();
    $response->header('Content-Type', 'text/plain');
    $response->end("requests: {$stats['request_count']}");
}, 'text/plain');
```

**When to use**: Health checks, metrics, static responses

[Learn more about Raw Routes →](raw-routes.html)

---

## Route Parameters

```php
// Single parameter
$router->get('/users/{id}', function ($id) {
    return Response::make()->json(['user_id' => $id]);
});

// Multiple parameters
$router->get('/posts/{postId}/comments/{commentId}', function ($postId, $commentId) {
    return Response::make()->json([
        'post_id' => $postId,
        'comment_id' => $commentId
    ]);
});

// Optional parameters (coming soon)
$router->get('/search/{query?}', function ($query = null) {
    return Response::make()->json(['query' => $query]);
});
```

---

## Controller Routes

```php
// Basic controller route
$router->get('/users', 'App\Controllers\UserController@index');
$router->post('/users', 'App\Controllers\UserController@store');
$router->get('/users/{id}', 'App\Controllers\UserController@show');
$router->put('/users/{id}', 'App\Controllers\UserController@update');
$router->delete('/users/{id}', 'App\Controllers\UserController@destroy');

// RESTful resource (shorthand - coming soon)
$router->resource('/users', 'App\Controllers\UserController');
```

---

## Route Groups (Coming Soon)

```php
// Prefix
$router->group(['prefix' => '/api/v1'], function ($router) {
    $router->get('/users', 'UserController@index');
    $router->get('/posts', 'PostController@index');
});

// Middleware
$router->group(['middleware' => ['auth']], function ($router) {
    $router->get('/dashboard', 'DashboardController@index');
    $router->get('/profile', 'ProfileController@show');
});

// Combined
$router->group(['prefix' => '/admin', 'middleware' => ['auth', 'admin']], function ($router) {
    $router->get('/users', 'AdminController@users');
    $router->get('/settings', 'AdminController@settings');
});
```

---

## Middleware (Coming Soon)

```php
// Single middleware
$router->get('/dashboard', 'DashboardController@index')
    ->middleware('auth');

// Multiple middlewares
$router->get('/admin', 'AdminController@index')
    ->middleware(['auth', 'admin']);
```

---

## Route Caching

For production, compile routes for maximum performance:

```bash
alpha route:cache
```

This creates `bootstrap/cache/routes.php` with pre-compiled routes:

```
Routes cached successfully!
  Raw routes (zero overhead): 4
  Static routes: 5
  Dynamic routes: 3
  Total: 12

⚡ Route lookup is now optimized for production!
```

Clear cache:

```bash
alpha route:clear
```

---

## Route Performance

| Route Type | Lookup Time | Use Case |
|------------|-------------|----------|
| **Raw** | O(1) ~0.001ms | Health checks, metrics |
| **Static** | O(1) ~0.01ms | Exact paths `/users` |
| **Dynamic** | O(n) ~0.1ms | Parameters `/users/{id}` |

---

## Best Practices

### ✅ DO

```php
// Use raw routes for simple endpoints
$router->raw('/health', ['status' => 'ok'], 'application/json');

// Use static routes when possible
$router->get('/about', 'PageController@about');

// Use controllers for complex logic
$router->get('/users', 'UserController@index');
```

### ❌ DON'T

```php
// Don't use raw routes for complex logic
$router->raw('/users', function($req, $res) {
    $users = DB::query('SELECT * FROM users'); // ❌
    $res->end(json_encode($users));
});

// Don't put business logic in route closures
$router->get('/orders', function () {
    // 100 lines of code... ❌
});
```

---

## API Reference

```php
// HTTP methods
$router->get(string $path, string|Closure $handler): Route
$router->post(string $path, string|Closure $handler): Route
$router->put(string $path, string|Closure $handler): Route
$router->delete(string $path, string|Closure $handler): Route
$router->patch(string $path, string|Closure $handler): Route
$router->any(string $path, string|Closure $handler): Route

// Raw routes
$router->raw(
    string $path,
    string|array|Closure $handler,
    string $contentType = 'text/plain',
    string $method = 'GET'
): void
```

---

## Next Steps

- [Raw Routes →](raw-routes.html)
- [Controllers →](controllers.html)
- [Route Caching →](route-caching.html)
