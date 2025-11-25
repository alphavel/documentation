---
layout: default
title: Controllers
---

# Controllers

Controllers handle HTTP requests and contain your application logic. Alphavel provides powerful features like automatic dependency injection, type hinting, and request/response helpers.

---

## Basic Controller

Create a controller in `app/Controllers/`:

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
                ['id' => 1, 'name' => 'John Doe'],
                ['id' => 2, 'name' => 'Jane Smith']
            ]
        ]);
    }

    public function show(int $id)
    {
        return Response::make()->json([
            'user' => ['id' => $id, 'name' => "User {$id}"]
        ]);
    }
}
```

Register routes in `routes/api.php`:

```php
$router->get('/users', 'App\Controllers\UserController@index');
$router->get('/users/{id}', 'App\Controllers\UserController@show');
```

---

## Dependency Injection

Alphavel automatically injects dependencies into controller methods using **constructor property promotion** and **method parameter injection**.

### Constructor Injection

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Response;
use Alphavel\Database\DB;
use Alphavel\Cache\Cache;
use Alphavel\Logging\Log;

class UserController extends Controller
{
    public function __construct(
        private DB $db,
        private Cache $cache,
        private Log $log
    ) {}

    public function index()
    {
        // Try cache first
        $users = $this->cache->remember('users:all', 3600, function () {
            $this->log->info('Fetching users from database');
            return $this->db->query('SELECT * FROM users');
        });

        return Response::make()->json(['users' => $users]);
    }
}
```

### Method Injection

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;
use Alphavel\Database\DB;

class UserController extends Controller
{
    // Request and route parameters automatically injected
    public function store(Request $request, DB $db)
    {
        $data = $request->all();
        
        $userId = $db->table('users')->insert([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => password_hash($data['password'], PASSWORD_BCRYPT),
            'created_at' => date('Y-m-d H:i:s')
        ]);

        return Response::make()->json([
            'id' => $userId,
            'message' => 'User created successfully'
        ], 201);
    }

    public function show(int $id, DB $db)
    {
        $user = $db->table('users')
            ->where('id', $id)
            ->first();

        if (!$user) {
            return Response::make()->json([
                'error' => 'User not found'
            ], 404);
        }

        return Response::make()->json(['user' => $user]);
    }
}
```

---

## Request Handling

### Accessing Request Data

```php
public function store(Request $request)
{
    // Get all input
    $data = $request->all();

    // Get specific fields
    $name = $request->get('name');
    $email = $request->get('email');

    // Get with default value
    $role = $request->get('role', 'user');

    // Check if field exists
    if ($request->has('email')) {
        // ...
    }

    // Get only specific fields
    $credentials = $request->only(['email', 'password']);

    // Get all except specific fields
    $data = $request->except(['password', 'password_confirmation']);

    // Get JSON payload
    $payload = $request->json();

    // Get raw body
    $body = $request->getContent();
}
```

### File Uploads

```php
public function uploadAvatar(Request $request)
{
    $file = $request->file('avatar');

    if ($file) {
        $filename = time() . '_' . $file->getClientFilename();
        $file->moveTo(storage_path('avatars/' . $filename));

        return Response::make()->json([
            'message' => 'Avatar uploaded',
            'filename' => $filename
        ]);
    }

    return Response::make()->json([
        'error' => 'No file uploaded'
    ], 400);
}
```

### Headers

```php
public function checkHeaders(Request $request)
{
    // Get header
    $auth = $request->header('Authorization');
    $userAgent = $request->header('User-Agent');

    // Get all headers
    $headers = $request->headers();

    // Check if header exists
    if ($request->hasHeader('X-API-Key')) {
        // ...
    }
}
```

---

## Response Types

### JSON Response

```php
public function index()
{
    return Response::make()->json([
        'users' => [...]
    ]);
}

// With custom status
public function create()
{
    return Response::make()->json([
        'message' => 'Created'
    ], 201);
}

// Error response
public function error()
{
    return Response::make()->json([
        'error' => 'Not found'
    ], 404);
}
```

### Success/Error Helpers

```php
// Success response (200)
return Response::success(['user' => $user]);

// Success with custom status
return Response::success(['user' => $user], 201);

// Error response (400)
return Response::error('Validation failed', 400, [
    'email' => ['Email is required']
]);

// Not found (404)
return Response::error('User not found', 404);
```

### Custom Headers

```php
public function download()
{
    return Response::make()
        ->content($fileContent)
        ->header('Content-Type', 'application/pdf')
        ->header('Content-Disposition', 'attachment; filename="report.pdf"')
        ->status(200);
}
```

### Redirect

```php
public function redirect()
{
    return Response::make()
        ->redirect('/dashboard', 302);
}
```

---

## Validation

Integrate with Validation package:

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;
use Alphavel\Validation\Validator;
use Alphavel\Database\DB;

class UserController extends Controller
{
    public function store(Request $request, DB $db)
    {
        // Validate request
        $validator = Validator::make($request->all(), [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|min:8|confirmed',
            'age' => 'required|integer|min:18',
        ]);

        if ($validator->fails()) {
            return Response::error('Validation failed', 422, $validator->errors());
        }

        // Get validated data
        $validated = $validator->validated();

        // Hash password
        $validated['password'] = password_hash($validated['password'], PASSWORD_BCRYPT);

        // Insert user
        $userId = $db->table('users')->insert($validated);

        return Response::success([
            'id' => $userId,
            'message' => 'User created successfully'
        ], 201);
    }
}
```

---

## Database Operations

### Using Query Builder

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Response;
use Alphavel\Database\DB;

class UserController extends Controller
{
    public function index(DB $db)
    {
        $users = $db->table('users')
            ->where('active', 1)
            ->orderBy('created_at', 'DESC')
            ->limit(20)
            ->get();

        return Response::make()->json(['users' => $users]);
    }

    public function show(int $id, DB $db)
    {
        $user = $db->table('users')
            ->where('id', $id)
            ->first();

        if (!$user) {
            return Response::error('User not found', 404);
        }

        return Response::make()->json(['user' => $user]);
    }

    public function store(Request $request, DB $db)
    {
        $userId = $db->table('users')->insert([
            'name' => $request->get('name'),
            'email' => $request->get('email'),
            'created_at' => date('Y-m-d H:i:s')
        ]);

        return Response::success(['id' => $userId], 201);
    }

    public function update(int $id, Request $request, DB $db)
    {
        $affected = $db->table('users')
            ->where('id', $id)
            ->update([
                'name' => $request->get('name'),
                'updated_at' => date('Y-m-d H:i:s')
            ]);

        if ($affected === 0) {
            return Response::error('User not found', 404);
        }

        return Response::success(['message' => 'User updated']);
    }

    public function destroy(int $id, DB $db)
    {
        $affected = $db->table('users')
            ->where('id', $id)
            ->delete();

        if ($affected === 0) {
            return Response::error('User not found', 404);
        }

        return Response::success(['message' => 'User deleted']);
    }
}
```

### Using Raw Queries

```php
public function complexQuery(DB $db)
{
    $results = $db->query('
        SELECT 
            u.id, 
            u.name, 
            COUNT(o.id) as order_count,
            SUM(o.total) as total_spent
        FROM users u
        LEFT JOIN orders o ON o.user_id = u.id
        WHERE u.active = ?
        GROUP BY u.id, u.name
        HAVING total_spent > ?
        ORDER BY total_spent DESC
        LIMIT 10
    ', [1, 1000]);

    return Response::make()->json(['top_customers' => $results]);
}
```

### Transactions

```php
public function transferFunds(Request $request, DB $db)
{
    try {
        $db->transaction(function () use ($db, $request) {
            $fromId = $request->get('from_user_id');
            $toId = $request->get('to_user_id');
            $amount = $request->get('amount');

            // Debit from sender
            $db->execute(
                'UPDATE wallets SET balance = balance - ? WHERE user_id = ?',
                [$amount, $fromId]
            );

            // Credit to receiver
            $db->execute(
                'UPDATE wallets SET balance = balance + ? WHERE user_id = ?',
                [$amount, $toId]
            );

            // Log transaction
            $db->table('transactions')->insert([
                'from_user_id' => $fromId,
                'to_user_id' => $toId,
                'amount' => $amount,
                'created_at' => date('Y-m-d H:i:s')
            ]);
        });

        return Response::success(['message' => 'Transfer successful']);
    } catch (\Exception $e) {
        return Response::error('Transfer failed: ' . $e->getMessage(), 400);
    }
}
```

---

## Caching

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Response;
use Alphavel\Database\DB;
use Alphavel\Cache\Cache;

class ProductController extends Controller
{
    public function index(DB $db, Cache $cache)
    {
        // Cache for 1 hour
        $products = $cache->remember('products:all', 3600, function () use ($db) {
            return $db->query('SELECT * FROM products WHERE active = 1');
        });

        return Response::make()->json(['products' => $products]);
    }

    public function show(int $id, DB $db, Cache $cache)
    {
        $cacheKey = "products:{$id}";

        // Try cache first
        $product = $cache->get($cacheKey);

        if (!$product) {
            $product = $db->table('products')->where('id', $id)->first();
            
            if ($product) {
                $cache->set($cacheKey, $product, 3600);
            }
        }

        if (!$product) {
            return Response::error('Product not found', 404);
        }

        return Response::make()->json(['product' => $product]);
    }

    public function update(int $id, Request $request, DB $db, Cache $cache)
    {
        $affected = $db->table('products')
            ->where('id', $id)
            ->update($request->only(['name', 'price']));

        if ($affected > 0) {
            // Invalidate cache
            $cache->delete("products:{$id}");
            $cache->delete('products:all');
        }

        return Response::success(['message' => 'Product updated']);
    }
}
```

---

## Logging

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;
use Alphavel\Logging\Log;
use Alphavel\Database\DB;

class OrderController extends Controller
{
    public function store(Request $request, DB $db, Log $log)
    {
        $log->info('Creating new order', [
            'user_id' => $request->get('user_id'),
            'total' => $request->get('total')
        ]);

        try {
            $orderId = $db->table('orders')->insert([
                'user_id' => $request->get('user_id'),
                'total' => $request->get('total'),
                'status' => 'pending',
                'created_at' => date('Y-m-d H:i:s')
            ]);

            $log->info('Order created successfully', ['order_id' => $orderId]);

            return Response::success(['order_id' => $orderId], 201);
        } catch (\Exception $e) {
            $log->error('Failed to create order', [
                'error' => $e->getMessage(),
                'user_id' => $request->get('user_id')
            ]);

            return Response::error('Failed to create order', 500);
        }
    }
}
```

---

## Events

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;
use Alphavel\Database\DB;
use Alphavel\Events\Event;

class UserController extends Controller
{
    public function store(Request $request, DB $db)
    {
        $userId = $db->table('users')->insert([
            'name' => $request->get('name'),
            'email' => $request->get('email'),
            'created_at' => date('Y-m-d H:i:s')
        ]);

        // Dispatch event (listeners handle email, profile creation, etc)
        Event::dispatch('user.created', [
            'user_id' => $userId,
            'email' => $request->get('email')
        ]);

        return Response::success(['id' => $userId], 201);
    }

    public function destroy(int $id, DB $db)
    {
        $db->table('users')->where('id', $id)->delete();

        // Dispatch cleanup event
        Event::dispatch('user.deleted', ['user_id' => $id]);

        return Response::success(['message' => 'User deleted']);
    }
}
```

---

## RESTful Resource Controller

Complete CRUD controller example:

```php
<?php

namespace App\Controllers;

use Alphavel\Framework\Controller;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;
use Alphavel\Database\DB;
use Alphavel\Validation\Validator;
use Alphavel\Cache\Cache;
use Alphavel\Logging\Log;

class ArticleController extends Controller
{
    /**
     * List all articles
     * GET /articles
     */
    public function index(DB $db, Cache $cache)
    {
        $articles = $cache->remember('articles:all', 600, function () use ($db) {
            return $db->table('articles')
                ->where('published', 1)
                ->orderBy('created_at', 'DESC')
                ->limit(50)
                ->get();
        });

        return Response::make()->json([
            'articles' => $articles,
            'total' => count($articles)
        ]);
    }

    /**
     * Show single article
     * GET /articles/{id}
     */
    public function show(int $id, DB $db, Cache $cache)
    {
        $article = $cache->remember("articles:{$id}", 600, function () use ($db, $id) {
            return $db->table('articles')->where('id', $id)->first();
        });

        if (!$article) {
            return Response::error('Article not found', 404);
        }

        // Increment view count (async, don't wait)
        $db->execute('UPDATE articles SET views = views + 1 WHERE id = ?', [$id]);

        return Response::make()->json(['article' => $article]);
    }

    /**
     * Create new article
     * POST /articles
     */
    public function store(Request $request, DB $db, Cache $cache, Log $log)
    {
        // Validate
        $validator = Validator::make($request->all(), [
            'title' => 'required|string|max:255',
            'content' => 'required|string',
            'author_id' => 'required|integer|exists:users,id',
            'published' => 'boolean',
        ]);

        if ($validator->fails()) {
            return Response::error('Validation failed', 422, $validator->errors());
        }

        $data = $validator->validated();
        $data['created_at'] = date('Y-m-d H:i:s');

        $articleId = $db->table('articles')->insert($data);

        $log->info('Article created', ['id' => $articleId, 'title' => $data['title']]);

        // Clear cache
        $cache->delete('articles:all');

        return Response::success([
            'id' => $articleId,
            'message' => 'Article created successfully'
        ], 201);
    }

    /**
     * Update article
     * PUT /articles/{id}
     */
    public function update(int $id, Request $request, DB $db, Cache $cache, Log $log)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'string|max:255',
            'content' => 'string',
            'published' => 'boolean',
        ]);

        if ($validator->fails()) {
            return Response::error('Validation failed', 422, $validator->errors());
        }

        $data = $validator->validated();
        $data['updated_at'] = date('Y-m-d H:i:s');

        $affected = $db->table('articles')
            ->where('id', $id)
            ->update($data);

        if ($affected === 0) {
            return Response::error('Article not found', 404);
        }

        $log->info('Article updated', ['id' => $id]);

        // Invalidate cache
        $cache->delete("articles:{$id}");
        $cache->delete('articles:all');

        return Response::success(['message' => 'Article updated successfully']);
    }

    /**
     * Delete article
     * DELETE /articles/{id}
     */
    public function destroy(int $id, DB $db, Cache $cache, Log $log)
    {
        $affected = $db->table('articles')
            ->where('id', $id)
            ->delete();

        if ($affected === 0) {
            return Response::error('Article not found', 404);
        }

        $log->warning('Article deleted', ['id' => $id]);

        // Clear cache
        $cache->delete("articles:{$id}");
        $cache->delete('articles:all');

        return Response::success(['message' => 'Article deleted successfully']);
    }
}
```

**Routes:**

```php
$router->get('/articles', 'App\Controllers\ArticleController@index');
$router->get('/articles/{id}', 'App\Controllers\ArticleController@show');
$router->post('/articles', 'App\Controllers\ArticleController@store');
$router->put('/articles/{id}', 'App\Controllers\ArticleController@update');
$router->delete('/articles/{id}', 'App\Controllers\ArticleController@destroy');
```

---

## Best Practices

### ✅ DO

```php
// Use dependency injection
public function index(DB $db, Cache $cache) { }

// Validate input
$validator = Validator::make($request->all(), [...]);

// Use type hints
public function show(int $id, DB $db) { }

// Cache expensive queries
$cache->remember('key', 3600, fn() => $db->query(...));

// Log important actions
$log->info('User created', ['user_id' => $id]);

// Return proper HTTP status codes
return Response::make()->json([...], 201);
```

### ❌ DON'T

```php
// Don't instantiate dependencies manually
$db = new Database(); // ❌

// Don't skip validation
$db->insert($request->all()); // ❌ Unsafe!

// Don't use global state
global $db; // ❌

// Don't put business logic in routes
$router->get('/users', function() {
    // 100 lines of code... ❌
});

// Don't return raw arrays
return ['users' => $users]; // ❌ Use Response object
```

---

## Next Steps

- [Dependency Injection →](autowiring.html)
- [Validation →](../packages/validation/)
- [Database →](../packages/database/)
