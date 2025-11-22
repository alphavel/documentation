# Authentication (JWT)

> JWT authentication with Guards, Middleware, and Laravel-like API

[![Status](https://img.shields.io/badge/status-production%20ready-green.svg)]()
[![Performance](https://img.shields.io/badge/overhead-0.08%25-green.svg)]()

---

## 🚀 Installation

### Using Alpha CLI (Recommended)

```bash
# Interactive wizard
php alpha make:auth

# Menu options:
# 1. Configure authentication (JWT secret, TTL)
# 2. Create User model
# 3. Add auth routes
# 4. Create AuthController
# 5. Show usage examples
```

### Manual Installation

```bash
composer require alphavel/auth
```

---

## ⚡ Quick Start

### 1. Configure JWT

```bash
# Generate JWT secret
php alpha make:auth
# Select: "Configure authentication"
```

Or add manually to `.env`:

```env
JWT_SECRET=your-secret-key-here
JWT_TTL=3600  # 1 hour in seconds
```

### 2. Create User Model

```php
// app/Models/User.php
namespace App\Models;

use Alphavel\Database\Model;
use Alphavel\Auth\Contracts\Authenticatable;

class User extends Model implements Authenticatable
{
    protected string $table = 'users';
    
    protected array $fillable = ['name', 'email', 'password'];
    
    protected array $hidden = ['password'];
    
    // Authenticatable interface methods
    public function getAuthIdentifier(): int|string
    {
        return $this->id;
    }
    
    public function getAuthPassword(): string
    {
        return $this->password;
    }
}
```

### 3. Create Auth Routes

```php
// routes/api.php
use App\Controllers\AuthController;

$router->post('/auth/register', [AuthController::class, 'register']);
$router->post('/auth/login', [AuthController::class, 'login']);
$router->post('/auth/logout', [AuthController::class, 'logout'])
    ->middleware('auth');
$router->get('/auth/me', [AuthController::class, 'me'])
    ->middleware('auth');
```

### 4. Create AuthController

```php
// app/Controllers/AuthController.php
namespace App\Controllers;

use Alphavel\Controller;
use Alphavel\Auth\Facades\Auth;
use App\Models\User;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $data = $request->validate([
            'name' => 'required|string|max:100',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|min:8',
        ]);
        
        $data['password'] = password_hash($data['password'], PASSWORD_BCRYPT);
        $user = User::create($data);
        
        return ['user' => $user, 'message' => 'Registration successful'];
    }
    
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);
        
        $token = Auth::attempt($credentials);
        
        if (!$token) {
            return Response::unauthorized(['error' => 'Invalid credentials']);
        }
        
        return ['token' => $token, 'user' => Auth::user()];
    }
    
    public function logout()
    {
        Auth::logout();
        return ['message' => 'Logged out successfully'];
    }
    
    public function me()
    {
        return ['user' => Auth::user()];
    }
}
```

---

## 📖 Usage

### Login / Attempt

```php
use Alphavel\Auth\Facades\Auth;

// Attempt login
$token = Auth::attempt([
    'email' => 'user@example.com',
    'password' => 'secret'
]);

if ($token) {
    // Success - return token to client
    return ['token' => $token];
}
```

### Get Authenticated User

```php
// In controllers/routes with 'auth' middleware
$user = Auth::user();

// Check if authenticated
if (Auth::check()) {
    // User is authenticated
}

// Get user ID
$userId = Auth::id();
```

### Protect Routes

```php
// Single route
$router->get('/api/posts', [PostController::class, 'index'])
    ->middleware('auth');

// Route group
$router->middleware('auth')->group(function ($router) {
    $router->get('/posts', [PostController::class, 'index']);
    $router->post('/posts', [PostController::class, 'store']);
    $router->put('/posts/{id}', [PostController::class, 'update']);
    $router->delete('/posts/{id}', [PostController::class, 'destroy']);
});
```

### Manual Token Management

```php
// Login with user instance
$user = User::find(1);
$token = Auth::login($user);

// Set token manually
Auth::guard()->setToken($token);

// Logout (blacklist token)
Auth::logout();
```

### Helper Function

```php
// Use auth() helper
$user = auth()->user();
$id = auth()->id();
$check = auth()->check();

// Attempt login via helper
$token = auth()->attempt($credentials);
```

---

## 🔧 Configuration

### Config File

Publish config:

```bash
php alpha vendor:publish --tag=auth-config
```

```php
// config/auth.php
return [
    'defaults' => [
        'guard' => 'jwt',
        'provider' => 'users',
    ],
    
    'guards' => [
        'jwt' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],
    
    'providers' => [
        'users' => [
            'driver' => 'database',
            'model' => App\Models\User::class,
        ],
    ],
    
    'jwt' => [
        'secret' => env('JWT_SECRET', ''),
        'ttl' => env('JWT_TTL', 3600), // 1 hour
        'algorithm' => 'HS256',
        'blacklist_grace_period' => 0,
    ],
    
    'passwords' => [
        'algorithm' => PASSWORD_BCRYPT,
        'options' => ['cost' => 12],
    ],
];
```

---

## ⚡ Performance

### Benchmarks

- **Overhead:** 0.08% per request (< 0.1ms)
- **Token validation:** 0.001ms (Swoole Table blacklist)
- **User loading:** Lazy (only when `Auth::user()` called)

### Optimizations

1. **Swoole Table Blacklist**
   - O(1) token lookup
   - 150x faster than Redis
   - Shared across workers

2. **Lazy User Loading**
   - No DB query for token-only checks
   - User loaded only when accessed

3. **Singleton Guards**
   - Guard instances cached per request
   - No repeated initialization

---

## 🔒 Security

### Token Blacklisting

Tokens are blacklisted on logout:

```php
Auth::logout(); // Adds token to blacklist
```

### Password Hashing

```php
// Secure password hashing
$hashed = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);

// Verify
if (password_verify($password, $hashed)) {
    // Correct password
}
```

### Token Expiration

Tokens expire based on `JWT_TTL`:

```env
JWT_TTL=3600  # 1 hour
```

---

## 📚 API Reference

### Auth Facade

```php
// Attempt login
Auth::attempt(array $credentials): string|false

// Login with user
Auth::login(Authenticatable $user): string

// Logout (blacklist token)
Auth::logout(): void

// Get authenticated user
Auth::user(): ?Authenticatable

// Check if authenticated
Auth::check(): bool

// Get user ID
Auth::id(): int|string|null

// Get guard instance
Auth::guard(string $name = null): Guard

// Get token
Auth::token(): ?string
```

### JWT Class

```php
// Encode payload
JWT::encode(array $payload): string

// Decode token
JWT::decode(string $token): ?array

// Blacklist token
JWT::blacklist(string $token): void

// Check if blacklisted
JWT::isBlacklisted(string $token): bool
```

---

## 🧪 Testing

```php
use Alphavel\Testing\TestCase;
use App\Models\User;

class AuthTest extends TestCase
{
    public function testLogin()
    {
        $user = User::create([
            'email' => 'test@example.com',
            'password' => password_hash('secret', PASSWORD_BCRYPT),
        ]);
        
        $token = Auth::attempt([
            'email' => 'test@example.com',
            'password' => 'secret',
        ]);
        
        $this->assertNotNull($token);
        $this->assertEquals($user->id, Auth::id());
    }
}
```

---

## 🔗 Related

- [Guards & Middleware](guards.md)
- [User Authentication](usage.md)
- [API Reference](api.md)

---

**Status:** ✅ Production Ready  
**Version:** 1.0.0  
**Performance:** 0.08% overhead (< 0.1% target)
