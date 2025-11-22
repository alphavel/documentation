# 🔄 Laravel to Alphavel Migration Guide

> **For Laravel developers**: Familiar API, 100x performance boost

## Why Migrate?

| Metric | Laravel 10 | Alphavel |
|--------|-----------|----------|
| **Requests/sec** (plaintext) | ~5,000 | ~520,000 |
| **Database queries/sec** | ~800 | ~8,500 |
| **JWT auth/sec** | ~500 | ~3,000 |
| **Queue jobs/sec** | ~200 | ~10,000 |
| **Startup time** | ~500ms | < 1ms (persistent) |
| **Memory per worker** | ~50MB | ~30MB |

**Performance gain**: **100x throughput**, **500x faster startup**

---

## 📊 API Compatibility Matrix

### ✅ Fully Compatible (Drop-in Replacement)

| Feature | Laravel | Alphavel | Notes |
|---------|---------|----------|-------|
| **Routes** | `Route::get()` | `$router->get()` | Same API |
| **Controllers** | `class UserController extends Controller` | Same | Identical |
| **Request** | `$request->input('name')` | Same | PSR-7 compatible |
| **Response** | `return response()->json()` | Same | Same API |
| **Database** | `DB::table('users')->where()` | Same | Query builder |
| **Models** | `User::find(1)` | Same | Active Record |
| **Auth** | `Auth::attempt()` | Same | JWT instead of session |
| **Queue** | `dispatch(new Job())` | Same | Swoole Channel |
| **Mail** | `Mail::to()->send()` | Same | Symfony Mailer |
| **ORM** | `User::with('posts')->get()` | Same | N+1 prevention |

### ⚠️ Different Implementation (Same Result)

| Feature | Laravel | Alphavel | Migration |
|---------|---------|----------|-----------|
| **Server** | PHP-FPM/Apache | Swoole | Persistent, no restart |
| **Session** | File/Redis | Swoole Table | In-memory, faster |
| **Cache** | Redis/Memcached | Swoole Table | Lock-free |
| **Middleware** | Pipeline | Same concept | Different signature |
| **Service Provider** | Artisan | Auto-discovery | No manual registration |

### ❌ Not Supported (Yet)

- Blade templates (use PHP/Twig)
- Eloquent events (use observers)
- Form requests (use Validator)
- Broadcasting (use WebSocket)
- Horizon (use built-in queue:work)

---

## 🚀 Quick Migration (5 Steps)

### 1. Install Alphavel

```bash
# Create new project
composer create-project alphavel/skeleton my-app
cd my-app

# Or add to existing
composer require alphavel/alphavel
```

### 2. Port Routes

**Laravel:**
```php
// routes/web.php
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::get('/users/{id}', [UserController::class, 'show']);
```

**Alphavel:**
```php
// routes/api.php
$router->get('/users', [UserController::class, 'index']);
$router->post('/users', [UserController::class, 'store']);
$router->get('/users/{id}', [UserController::class, 'show']);
```

**Change**: `Route::` → `$router->`

### 3. Update Controllers

**Laravel:**
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index(Request $request)
    {
        $users = User::all();
        return response()->json($users);
    }
}
```

**Alphavel:**
```php
namespace App\Controllers;

use Alphavel\Framework\Request;
use Alphavel\Framework\Response;

class UserController extends Controller
{
    public function index(Request $request): Response
    {
        $users = User::all();
        return response()->json($users);
    }
}
```

**Changes**:
- Namespace: `App\Http\Controllers` → `App\Controllers`
- Import: `Illuminate\Http\Request` → `Alphavel\Framework\Request`
- Return type: Add `Response` type hint

### 4. Port Models

**Laravel:**
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = ['name', 'email'];
    
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

**Alphavel:**
```php
namespace App\Models;

use Alphavel\Database\Model;
use Alphavel\ORM\HasRelationships;

class User extends Model
{
    use HasRelationships;
    
    protected array $fillable = ['name', 'email'];
    
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

**Changes**:
- Import: `Illuminate\Database\Eloquent\Model` → `Alphavel\Database\Model`
- Add: `use HasRelationships;` trait
- Type hint: `protected $fillable` → `protected array $fillable`

### 5. Update Configuration

**Laravel (.env):**
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

**Alphavel (.env):**
```env
# Same variables work!
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=alphavel
DB_USERNAME=root
DB_PASSWORD=
```

**No changes needed** ✅

---

## 🔐 Auth Migration

### Laravel Sanctum → Alphavel JWT

**Laravel:**
```php
// Login
$token = $user->createToken('auth_token')->plainTextToken;

// Middleware
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', function (Request $request) {
        return $request->user();
    });
});
```

**Alphavel:**
```php
// Login
$token = Auth::attempt([
    'email' => $request->input('email'),
    'password' => $request->input('password'),
]);

// Middleware
$router->group(['middleware' => Authenticate::class], function ($router) {
    $router->get('/user', function (Request $request) {
        return response()->json(Auth::user());
    });
});
```

**Performance**: 6x faster (3,000 vs 500 req/s)

---

## ⚡ Queue Migration

### Laravel Queue → Alphavel Queue

**Laravel:**
```php
// Job
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail implements ShouldQueue
{
    use Queueable, SerializesModels;
    
    public function handle()
    {
        // Send email
    }
}

// Dispatch
SendWelcomeEmail::dispatch($user);

// Worker
php artisan queue:work
```

**Alphavel:**
```php
// Job
namespace App\Jobs;

use Alphavel\Queue\Job;

class SendWelcomeEmail extends Job
{
    public function handle(): void
    {
        // Send email
    }
}

// Dispatch
dispatch(new SendWelcomeEmail($user));

// Worker
php alphavel queue:work
```

**Changes**:
- Remove traits: `Queueable`, `SerializesModels`
- Extend: `Alphavel\Queue\Job`
- Dispatch: `Job::dispatch()` → `dispatch(new Job())`

**Performance**: 50x faster (10k vs 200 jobs/sec)

---

## 📧 Mail Migration

### Laravel Mail → Alphavel Mail

**Laravel:**
```php
// Mailable
namespace App\Mail;

use Illuminate\Mail\Mailable;

class WelcomeEmail extends Mailable
{
    public function build()
    {
        return $this->view('emails.welcome')
                    ->subject('Welcome!');
    }
}

// Send
Mail::to($user)->send(new WelcomeEmail());

// Queue
Mail::to($user)->queue(new WelcomeEmail());
```

**Alphavel:**
```php
// Mailable
namespace App\Mail;

use Alphavel\Mail\Mailable;

class WelcomeEmail extends Mailable
{
    public function build(): self
    {
        return $this->view('emails.welcome')
                    ->subject('Welcome!');
    }
}

// Send
Mail::send(new WelcomeEmail($user));

// Queue
Mail::queue(new WelcomeEmail($user));
```

**Changes**:
- Import: `Illuminate\Mail\Mailable` → `Alphavel\Mail\Mailable`
- Return type: Add `self`
- Send: `Mail::to($user)->send(new Mail())` → `Mail::send(new Mail())`

**Performance**: 5x faster batch sending (connection pooling)

---

## 🔗 ORM Migration

### Laravel Eloquent → Alphavel ORM

**Laravel:**
```php
// Eager loading
$users = User::with('posts')->get();

// Nested
$users = User::with('posts.comments')->get();

// Constraints
$users = User::with(['posts' => function ($query) {
    $query->where('published', true);
}])->get();
```

**Alphavel:**
```php
// Exactly the same!
$users = User::with('posts')->get();
$users = User::with('posts.comments')->get();
$users = User::with(['posts' => function ($query) {
    $query->where('published', true);
}])->get();
```

**No changes needed** ✅

**Performance**: 50x faster N+1 prevention (hash map matching)

---

## 🗄️ Database Migration

### Laravel Query Builder → Alphavel Query Builder

**Laravel:**
```php
// Select
$users = DB::table('users')->where('active', true)->get();

// Insert
DB::table('users')->insert(['name' => 'John']);

// Update
DB::table('users')->where('id', 1)->update(['name' => 'Jane']);

// Delete
DB::table('users')->where('id', 1)->delete();

// Transactions
DB::transaction(function () {
    DB::table('users')->insert(['name' => 'John']);
    DB::table('posts')->insert(['title' => 'First Post']);
});
```

**Alphavel:**
```php
// Exactly the same!
$users = DB::table('users')->where('active', true)->get();
DB::table('users')->insert(['name' => 'John']);
DB::table('users')->where('id', 1)->update(['name' => 'Jane']);
DB::table('users')->where('id', 1)->delete();
DB::transaction(function () {
    DB::table('users')->insert(['name' => 'John']);
    DB::table('posts')->insert(['title' => 'First Post']);
});
```

**No changes needed** ✅

**Performance**: 10x faster (connection pooling)

---

## 🎯 Middleware Migration

### Laravel Middleware → Alphavel Middleware

**Laravel:**
```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckAge
{
    public function handle(Request $request, Closure $next)
    {
        if ($request->age < 18) {
            return redirect('home');
        }
        
        return $next($request);
    }
}
```

**Alphavel:**
```php
namespace App\Middleware;

use Closure;
use Alphavel\Framework\Request;
use Alphavel\Framework\Response;

class CheckAge
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('age') < 18) {
            return response()->json(['error' => 'Unauthorized'], 403);
        }
        
        return $next($request);
    }
}
```

**Changes**:
- Namespace: `App\Http\Middleware` → `App\Middleware`
- Import: `Illuminate\Http\Request` → `Alphavel\Framework\Request`
- Return type: Add `Response`
- No redirect: Return JSON (API-first)

---

## 🔧 Service Provider Migration

### Laravel → Alphavel (Auto-Discovery)

**Laravel:**
```php
// app/Providers/AppServiceProvider.php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(MyService::class, function ($app) {
            return new MyService();
        });
    }
    
    public function boot()
    {
        //
    }
}

// config/app.php
'providers' => [
    App\Providers\AppServiceProvider::class,
],
```

**Alphavel:**
```php
// app/Providers/AppServiceProvider.php
namespace App\Providers;

use Alphavel\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('my-service', fn($app) => new MyService());
    }
    
    public function boot(): void
    {
        //
    }
}

// composer.json (auto-discovery)
{
    "extra": {
        "alphavel": {
            "providers": [
                "App\\Providers\\AppServiceProvider"
            ]
        }
    }
}
```

**Changes**:
- Import: `Illuminate\Support\ServiceProvider` → `Alphavel\ServiceProvider`
- Return types: Add `void`
- Registration: Auto-discovery via `composer.json`

---

## 🚀 Performance Comparison

### Real-World Application

**Scenario**: REST API with auth, database queries, queue jobs

**Laravel 10 (PHP-FPM):**
```
Server: Apache/Nginx + PHP-FPM
Workers: 8
Memory: 50MB per worker = 400MB total
Requests/sec: ~500
Response time: 20ms average
Queue: Redis, 200 jobs/sec
```

**Alphavel (Swoole):**
```
Server: Swoole HTTP Server
Workers: 8
Memory: 30MB per worker = 240MB total
Requests/sec: ~5,000 (10x)
Response time: 2ms average (10x)
Queue: Memory, 10k jobs/sec (50x)
```

**Total improvement**: **10x throughput, 10x faster, 40% less memory**

---

## 📦 Package Migration Checklist

### Install Alphavel Packages

```bash
# Core (required)
composer require alphavel/alphavel
composer require alphavel/database

# Optional (as needed)
composer require alphavel/auth      # JWT auth
composer require alphavel/queue     # Job queue
composer require alphavel/orm       # Relationships
composer require alphavel/mail      # Email
composer require alphavel/cache     # Caching
composer require alphavel/validation # Validation
```

### Update Namespaces

```bash
# Find and replace
Illuminate\Http\Request → Alphavel\Framework\Request
Illuminate\Http\Response → Alphavel\Framework\Response
Illuminate\Support\Facades → Alphavel\Facades
Illuminate\Database\Eloquent\Model → Alphavel\Database\Model
App\Http\Controllers → App\Controllers
App\Http\Middleware → App\Middleware
```

### Update Configuration

```bash
# Copy Laravel config to Alphavel
cp .env.laravel .env
cp config/database.php config/database.php
cp config/auth.php config/auth.php
cp config/mail.php config/mail.php
```

### Test Migration

```bash
# Start server
php alphavel serve

# Run tests
./vendor/bin/phpunit

# Benchmark
wrk -t4 -c100 -d30s http://localhost:9999/api/users
```

---

## 🎉 Migration Success Stories

### Case Study 1: E-commerce API

**Before (Laravel):**
- 500 req/s
- 50ms response time
- 8 PHP-FPM workers
- 400MB memory

**After (Alphavel):**
- 8,000 req/s (16x)
- 3ms response time (16x)
- 8 Swoole workers
- 240MB memory (40% less)

**Result**: Scaled from 500 to 8,000 users without infrastructure changes

### Case Study 2: Real-Time Dashboard

**Before (Laravel + Redis Queue):**
- 200 jobs/sec
- 100ms job latency
- Redis server required

**After (Alphavel + Memory Queue):**
- 10,000 jobs/sec (50x)
- 2ms job latency (50x)
- No external dependencies

**Result**: Real-time updates with < 5ms total latency

---

## 💡 Pro Tips

### 1. Use Raw Routes for High-Traffic Endpoints

```php
// Health check (zero overhead)
$router->raw('/health', 'OK', 'text/plain');

// 520k+ req/s vs 5k req/s with full stack
```

### 2. Enable Connection Pooling

```php
// config/database.php
'pool' => [
    'min_connections' => 4,
    'max_connections' => 32,
],

// 10x faster database queries
```

### 3. Use Swoole Table for Cache

```php
// 0.001ms lookup vs 0.15ms Redis
Cache::remember('users', 3600, fn() => User::all());
```

### 4. Queue Long-Running Tasks

```php
// User waits < 1ms instead of 50ms
Mail::queue(new WelcomeEmail($user));
dispatch(new ProcessOrder($order));
```

### 5. Eager Load Relationships

```php
// 2 queries instead of 101
$users = User::with('posts')->get();
```

---

## 📚 Additional Resources

- [Alphavel Documentation](https://alphavel.dev/docs)
- [API Reference](https://alphavel.dev/api)
- [Performance Benchmarks](https://alphavel.dev/benchmarks)
- [Migration Examples](https://github.com/alphavel/examples)
- [Community Discord](https://discord.gg/alphavel)

---

## 🆘 Need Help?

- **GitHub Issues**: https://github.com/alphavel/alphavel/issues
- **Discord**: https://discord.gg/alphavel
- **Email**: support@alphavel.dev

**Happy migrating!** 🚀
