---
layout: default
title: Helpers And Facades
---

# Helper Functions & Facades

Alphavel provides Laravel-style helper functions and facades for elegant, expressive code.

---

## Helper Functions

### Application Helpers

#### `app()`

Access the application container:

{% raw %}
```php
// Get application instance
$app = app();

// Resolve from container
$router = app('router');
$db = app('db');

// With type hint
/** @var Router $router */
$router = app(Router::class);
```
{% endraw %}

#### `config()`

Access configuration values:

{% raw %}
```php
// Get config value
$name = config('app.name');

// With default
$env = config('app.env', 'production');

// Nested values (dot notation)
$host = config('database.connections.mysql.host');
```
{% endraw %}

#### `env()`

Get environment variables:

{% raw %}
```php
// Get environment variable
$debug = env('APP_DEBUG', false);

// Common usage
$dbHost = env('DB_HOST', 'localhost');
$apiKey = env('API_KEY');
```
{% endraw %}

---

### Array Helpers

#### `array_get()`

Get array value using dot notation:

{% raw %}
```php
$array = [
    'user' => [
        'profile' => [
            'name' => 'John'
        ]
    ]
];

$name = array_get($array, 'user.profile.name');
// Result: 'John'

$email = array_get($array, 'user.email', 'default@example.com');
// Result: 'default@example.com'
```
{% endraw %}

#### `array_only()`

Get only specified keys:

{% raw %}
```php
$array = [
    'id' => 1,
    'name' => 'John',
    'email' => 'john@example.com',
    'password' => 'secret'
];

$safe = array_only($array, ['id', 'name', 'email']);
// Result: ['id' => 1, 'name' => 'John', 'email' => 'john@example.com']
```
{% endraw %}

#### `array_except()`

Remove specified keys:

{% raw %}
```php
$array = [
    'id' => 1,
    'name' => 'John',
    'password' => 'secret',
    'password_confirmation' => 'secret'
];

$safe = array_except($array, ['password', 'password_confirmation']);
// Result: ['id' => 1, 'name' => 'John']
```
{% endraw %}

---

### Data Helpers

#### `data_get()`

Get value from array or object using dot notation:

{% raw %}
```php
// Array
$array = ['user' => ['name' => 'John']];
$name = data_get($array, 'user.name');

// Object
$object = (object) ['user' => (object) ['name' => 'John']];
$name = data_get($object, 'user.name');

// With default
$email = data_get($object, 'user.email', 'default@example.com');
```
{% endraw %}

#### `blank()`

Check if value is empty:

{% raw %}
```php
blank(null);           // true
blank('');             // true
blank('   ');          // true
blank([]);             // true
blank(0);              // false
blank(false);          // false
blank('content');      // false
```
{% endraw %}

#### `filled()`

Check if value is not empty:

{% raw %}
```php
filled('content');     // true
filled(0);             // true
filled(false);         // true
filled(null);          // false
filled('');            // false
filled([]);            // false
```
{% endraw %}

---

### String Helpers

#### `str_contains()`

Check if string contains substring:

{% raw %}
```php
str_contains('Hello World', 'World');  // true
str_contains('Hello World', 'PHP');    // false
```
{% endraw %}

#### `str_starts_with()`

Check if string starts with prefix:

{% raw %}
```php
str_starts_with('Hello World', 'Hello');  // true
str_starts_with('Hello World', 'World');  // false
```
{% endraw %}

---

### Date Helpers

#### `now()`

Get current datetime:

{% raw %}
```php
$now = now();
// Result: '2025-11-21 15:30:45'

// Usage
DB::table('users')->insert([
    'name' => 'John',
    'created_at' => now()
]);
```
{% endraw %}

#### `today()`

Get current date:

{% raw %}
```php
$today = today();
// Result: '2025-11-21'

// Usage
$orders = DB::table('orders')
    ->where('created_at', '>=', today())
    ->get();
```
{% endraw %}

---

## Facades

Facades provide a static interface to services registered in the container.

### How Facades Work

{% raw %}
```php
// Behind the scenes
Cache::put('key', 'value', 3600);

// Is equivalent to
app('cache')->put('key', 'value', 3600);
```
{% endraw %}

**Performance:** Zero overhead - facade resolves to singleton instance (resolved once per request).

---

### Creating Facades

#### Step 1: Create Facade Class

{% raw %}
```php
// app/Facades/Cache.php
namespace App\Facades;

use Alphavel\Framework\Facade;

class Cache extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'cache';  // Container binding name
    }
}
```
{% endraw %}

#### Step 2: Register in Container

{% raw %}
```php
// app/Providers/AppServiceProvider.php
public function register(): void
{
    $this->app->singleton('cache', function ($app) {
        return new \Alphavel\Cache\CacheManager($app);
    });
}
```
{% endraw %}

#### Step 3: Use Facade

{% raw %}
```php
use App\Facades\Cache;

// Static calls
Cache::put('user.1', $user, 3600);
$user = Cache::get('user.1');
Cache::forget('user.1');
```
{% endraw %}

---

### Built-In Facades

Alphavel packages provide facades out of the box:

#### Database Facade

{% raw %}
```php
use Alphavel\Database\DB;

// Query builder
$users = DB::table('users')
    ->where('active', true)
    ->get();

// Raw queries
$users = DB::query('SELECT * FROM users WHERE id = ?', [1]);

// Transactions
DB::beginTransaction();
try {
    DB::table('users')->insert([...]);
    DB::table('logs')->insert([...]);
    DB::commit();
} catch (\Exception $e) {
    DB::rollBack();
}
```
{% endraw %}

#### Cache Facade

{% raw %}
```php
use Alphavel\Cache\Cache;

// Store
Cache::put('key', 'value', 3600);

// Retrieve
$value = Cache::get('key', 'default');

// Remember pattern
$users = Cache::remember('users.all', 3600, function () {
    return DB::table('users')->get();
});

// Tags
Cache::tags(['users', 'api'])->put('users.all', $users, 3600);
Cache::tags(['users'])->flush();
```
{% endraw %}

#### Log Facade

{% raw %}
```php
use Alphavel\Logging\Log;

// Levels
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);

// With context
Log::info('User logged in', [
    'user_id' => $userId,
    'ip' => $request->ip()
]);
```
{% endraw %}

#### Event Facade

{% raw %}
```php
use Alphavel\Events\Event;

// Dispatch event
Event::dispatch('user.created', ['user' => $user]);

// Listen
Event::listen('user.created', function ($data) {
    // Send welcome email
    Mail::send($data['user']);
});
```
{% endraw %}

---

### Custom Facade Example

Complete example creating a custom service with facade:

{% raw %}
```php
// 1. Create service
// app/Services/PaymentService.php
namespace App\Services;

class PaymentService
{
    public function charge(float $amount, string $token): array
    {
        // Payment logic
        return ['success' => true, 'transaction_id' => '123'];
    }
}

// 2. Create facade
// app/Facades/Payment.php
namespace App\Facades;

use Alphavel\Framework\Facade;

class Payment extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'payment';
    }
}

// 3. Register in service provider
// app/Providers/AppServiceProvider.php
public function register(): void
{
    $this->app->singleton('payment', function ($app) {
        return new \App\Services\PaymentService(
            $app->config('services.stripe.key')
        );
    });
}

// 4. Use facade
use App\Facades\Payment;

$result = Payment::charge(99.99, $token);
if ($result['success']) {
    // Payment successful
}
```
{% endraw %}

---

### Facade Benefits

| Benefit | Description |
|---------|-------------|
| **Clean Syntax** | Static calls instead of verbose container resolution |
| **Testable** | Easy to mock in tests |
| **Discoverable** | IDE autocomplete works with static methods |
| **Zero Overhead** | Resolves to singleton (no performance cost) |
| **Type Safety** | Can add PHPDoc for type hints |

---

### Facade vs. Dependency Injection

**Use Facades when:**
- ✅ Writing helper classes or utilities
- ✅ Need quick access without DI
- ✅ Working in routes, views, or helpers

**Use Dependency Injection when:**
- ✅ Writing testable controllers
- ✅ Need explicit dependencies
- ✅ Following SOLID principles

**Example:**

{% raw %}
```php
// Dependency Injection (recommended for controllers)
class UserController extends Controller
{
    public function __construct(
        private CacheManager $cache,
        private UserRepository $users
    ) {}
    
    public function index()
    {
        return $this->cache->remember('users', 3600, fn() =>
            $this->users->all()
        );
    }
}

// Facades (convenient for helpers/utilities)
class UserHelper
{
    public static function activeUsers()
    {
        return Cache::remember('users.active', 3600, fn() =>
            DB::table('users')->where('active', true)->get()
        );
    }
}
```
{% endraw %}

---

## Real-World Examples

### API Controller with Helpers

{% raw %}
```php
namespace App\Controllers;

use Alphavel\Controller;
use Alphavel\Request;
use Alphavel\Response;
use App\Facades\Cache;
use App\Facades\DB;
use App\Facades\Log;

class ArticleController extends Controller
{
    public function index(Request $request): Response
    {
        $page = $request->input('page', 1);
        $limit = $request->input('limit', 20);
        
        // Helper functions + facades
        $cacheKey = "articles.page.{$page}";
        
        $articles = Cache::remember($cacheKey, 3600, function () use ($limit) {
            return DB::table('articles')
                ->select(array_only([...], ['id', 'title', 'excerpt']))
                ->limit($limit)
                ->get();
        });
        
        Log::info('Articles fetched', [
            'page' => $page,
            'count' => count($articles),
            'cached' => Cache::has($cacheKey),
            'timestamp' => now()
        ]);
        
        return Response::json(data_get($articles, 'data', []));
    }
}
```
{% endraw %}

---

## Helper Functions Cheat Sheet

| Category | Function | Description |
|----------|----------|-------------|
| **App** | `app()` | Get application/service |
| | `config()` | Get configuration |
| | `env()` | Get environment variable |
| **Array** | `array_get()` | Get with dot notation |
| | `array_only()` | Get specific keys |
| | `array_except()` | Remove specific keys |
| **Data** | `data_get()` | Get from array/object |
| | `blank()` | Check if empty |
| | `filled()` | Check if not empty |
| **String** | `str_contains()` | Check contains |
| | `str_starts_with()` | Check starts with |
| **Date** | `now()` | Current datetime |
| | `today()` | Current date |

---

## Next Steps

- [Routing →](routing.html)
- [Controllers →](controllers.html)
- [Dependency Injection →](dependency-injection.html)
- [Performance →](performance.html)
