# Alphavel Ecosystem

O ecossistema Alphavel consiste em **dois projetos complementares** que trabalham juntos para entregar uma experiência de desenvolvimento completa.

---

## 🏗️ Arquitetura do Ecossistema

```
┌─────────────────────────────────────────────────────────┐
│                  Alphavel Ecosystem                      │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌────────────────────┐      ┌─────────────────────┐   │
│  │  Alphavel Core     │      │  Alpha CLI Tool     │   │
│  │  (Framework)       │◄─────┤  (Code Generator)   │   │
│  │                    │      │                     │   │
│  │  - Router          │      │  - make:controller  │   │
│  │  - Container       │      │  - make:model       │   │
│  │  - Request/Response│      │  - make:resource    │   │
│  │  - Pipeline        │      │  - add package      │   │
│  │  - ServiceProviders│      │  - inspect schema   │   │
│  └────────────────────┘      └─────────────────────┘   │
│           ▲                           │                  │
│           │                           ▼                  │
│           │         ┌──────────────────────┐            │
│           └─────────┤  Your Application    │            │
│                     │  (generated code)    │            │
│                     └──────────────────────┘            │
└─────────────────────────────────────────────────────────┘
```

---

## 📦 1. Alphavel Core (Framework)

**Repositório:** `alphavel/alphavel`  
**Namespace:** `Alphavel\Framework\`  
**Propósito:** Runtime framework para aplicações de alta performance

### Características

- ⚡ **Core Mínimo:** 15 KB, 5 arquivos essenciais
- 🚀 **Performance:** 20-45k req/s (standard routes), 520k+ req/s (raw routes)*
- 🔄 **Swoole-Powered:** Servidor HTTP assíncrono
- 📦 **Modular:** Packages opcionais (database, cache, logging, etc.)
- 🎯 **PSR-Compliant:** PSR-7, PSR-11, PSR-3

### Componentes Core

```php
Alphavel\Framework\
├── Application.php         # Application container
├── Router.php             # High-performance router
├── Request.php            # PSR-7 Request
├── Response.php           # PSR-7 Response
├── Container.php          # PSR-11 DI Container
├── Pipeline.php           # Middleware pipeline
├── ServiceProvider.php    # Service registration
├── Facade.php             # Static interface
└── helpers.php            # Helper functions
```

### CLI Commands (Built-in)

O **framework core** inclui 19 comandos básicos:

```bash
# Make commands (geração básica)
make:controller         # Controller simples
make:model             # Model básico
make:middleware        # Middleware
make:migration         # Migration
make:seeder            # Seeder
make:test              # Test
make:request           # Form Request
make:command           # CLI Command

# Optimization commands
optimize               # Optimize for production
optimize:clear         # Clear optimization
cache:clear            # Clear application cache
config:cache           # Cache config
config:clear           # Clear config cache
facade:clear           # Clear facade cache
route:cache            # Cache routes
route:clear            # Clear route cache
route:list             # List routes

# Development
serve                  # Start Swoole server
ide-helper             # Generate IDE helper
```

**Limitação:** Comandos `make:*` são básicos, sem schema awareness.

### Installation

```bash
composer require alphavel/alphavel
```

---

## 🛠️ 2. Alpha CLI Tool (Code Generator)

**Repositório:** `alphavel/alpha`  
**Namespace:** `Alphavel\Alpha\`  
**Propósito:** Ferramenta inteligente de geração de código

### Características

- 🧠 **Schema-Aware:** Lê estrutura do banco de dados
- ✅ **Validation Generation:** SQL types → Validation rules
- 🔗 **Relationship Detection:** Foreign keys → Model relationships
- 🚀 **Smart Controllers:** CRUD based on table structure
- 📊 **Schema Inspector:** Analyze database structure

### Commands

```bash
# Intelligent code generation
alpha make:controller UserController --model=User    # Schema-aware controller
alpha make:model User --table=users                  # Model from DB schema
alpha make:resource User                             # Full resource (model + controller)

# Package management
alpha add database                                    # Install package with alias
alpha add alphavel/cache                             # Install full package name

# Schema inspection
alpha inspect:schema users                           # Show table structure

# Route management
alpha route:cache                                     # Cache routes
alpha route:clear                                     # Clear route cache
```

### Schema Intelligence Example

**Database:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INT UNSIGNED,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Generated Model (`User`):**
```php
namespace App\Models;

use Alphavel\Database\Model;

class User extends Model
{
    protected string $table = 'users';
    
    protected array $fillable = [
        'name', 'email', 'age', 'status'
    ];
    
    // Auto-detected relationship
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

**Generated Validation Rules:**
```php
[
    'name' => 'required|string|max:100',
    'email' => 'required|email|max:255|unique:users,email',
    'age' => 'integer|min:0',
    'status' => 'in:active,inactive'
]
```

**Generated Controller:**
```php
namespace App\Controllers;

use Alphavel\Controller;
use App\Models\User;

class UserController extends Controller
{
    public function index()
    {
        return User::all();
    }
    
    public function show(int $id)
    {
        return User::find($id) ?? Response::notFound();
    }
    
    public function store(Request $request)
    {
        // Validation rules auto-generated from schema
        $validated = $request->validate([
            'name' => 'required|string|max:100',
            'email' => 'required|email|max:255|unique:users,email',
            'age' => 'integer|min:0',
            'status' => 'in:active,inactive'
        ]);
        
        return User::create($validated);
    }
    
    // update() and destroy() methods also generated...
}
```

### Installation

```bash
# As project dependency (recommended)
composer require alphavel/alpha --dev

# Or global installation
composer global require alphavel/alpha
```

---

## 🔄 Como os Dois Trabalham Juntos

### Development Workflow

```bash
# 1. Create new Alphavel project
composer create-project alphavel/skeleton my-app
cd my-app

# 2. Install Alpha CLI (dev dependency)
composer require alphavel/alpha --dev

# 3. Install packages using Alpha
php alpha add database
php alpha add cache
php alpha add validation

# 4. Generate code using Alpha
php alpha make:resource User

# 5. Run application using Core
php alpha serve   # Uses core's ServeCommand
# or
alpha serve       # Uses Alpha CLI's serve command (if implemented)
```

### Command Resolution

Quando você executa `php alpha <command>`, o sistema:

1. **Verifica Alpha CLI primeiro** (se instalado via `alphavel/alpha`)
2. **Fallback para Core commands** (de `alphavel/alphavel`)

**Exemplo:**

```bash
# Uses Alpha CLI (intelligent generation)
php alpha make:controller UserController --model=User

# Uses Core (basic generation if Alpha not installed)
php alpha make:controller UserController
```

---

## 📊 Performance Clarification

### Raw Routes (Plaintext Benchmark)

**Contexto:** TechEmpower Benchmark (plaintext test)

```php
// routes/api.php
$router->raw('/plaintext', 'Hello, World!', 'text/plain');
```

**Performance:** **520,000+ req/s**

**Condições:**
- Swoole Server otimizado
- Raw route (zero overhead)
- Response estático (sem lógica)
- Hardware: 12 cores, 32 GB RAM
- Benchmark: `wrk -t12 -c400`

### Standard Routes (JSON API)

**Contexto:** Aplicações reais com lógica de negócio

```php
// routes/api.php
$router->get('/users', 'UserController@index');
```

**Performance:** **20-45k req/s**

**Condições:**
- Framework stack completo
- Controller + DI
- Database queries (com pooling)
- JSON encoding
- Middleware pipeline

### Production-Constrained (Real-World)

**Contexto:** Microserviços com recursos limitados

**Hardware:** 0.5 CPU, 512 MB RAM (Kubernetes pod)

**Performance:** **5,042 req/s** (Alphavel) vs **1,050 req/s** (Hyperf)

**Vantagem:** 4.8x mais rápido em ambientes restritos

---

## 🎯 When to Use What

### Use Alphavel Core Only

✅ **Scenarios:**
- Ultra-high performance APIs (45k+ req/s target)
- Microservices with minimal dependencies
- Container-constrained environments
- Budget-conscious deployments

✅ **Commands Available:**
- Basic `make:*` commands
- Optimization commands
- `serve`, `route:cache`, etc.

### Use Alphavel Core + Alpha CLI

✅ **Scenarios:**
- Rapid application development
- Database-driven applications
- Team projects (consistency)
- Complex schemas with relationships

✅ **Benefits:**
- Intelligent code generation
- Schema-aware validation
- Relationship detection
- Package management with recipes

---

## 📦 Package Ecosystem

Both projects support the same modular packages:

| Package | Purpose | Size | Boot Impact |
|---------|---------|------|-------------|
| `alphavel/database` | MySQL/PostgreSQL + pooling | 45 KB | +2ms |
| `alphavel/cache` | Redis/Memcached | 12 KB | +1ms |
| `alphavel/logging` | Monolog integration | 8 KB | +0.5ms |
| `alphavel/events` | Event dispatcher | 6 KB | +0.3ms |
| `alphavel/validation` | Request validation | 18 KB | +1ms |
| `alphavel/support` | Collections & helpers | 10 KB | +0.5ms |

**Installation:**

```bash
# Using Alpha CLI (recommended - with recipes)
php alpha add database

# Using Composer directly
composer require alphavel/database
```

---

## 🚀 Quick Start Comparison

### Without Alpha CLI

```bash
# 1. Create project
composer create-project alphavel/skeleton my-app
cd my-app

# 2. Install packages manually
composer require alphavel/database
composer require alphavel/cache

# 3. Create files manually
php alpha make:controller UserController
php alpha make:model User

# 4. Write code manually
# - Add validation rules
# - Add relationships
# - Configure routes
```

**Time:** ~30 minutes for simple CRUD

### With Alpha CLI

```bash
# 1. Create project
composer create-project alphavel/skeleton my-app
cd my-app

# 2. Install Alpha CLI
composer require alphavel/alpha --dev

# 3. Install packages with recipes
php alpha add database
php alpha add cache

# 4. Generate complete resource
php alpha make:resource User
```

**Time:** ~5 minutes (6x faster!)  
**Output:** Model + Controller + Validation + Routes

---

## 📋 Command Cheat Sheet

### Alphavel Core Commands

```bash
# Generation (basic)
php alpha make:controller UserController
php alpha make:model User
php alpha make:middleware AuthMiddleware

# Optimization
php alpha optimize
php alpha route:cache
php alpha config:cache

# Development
php alpha serve
php alpha route:list
```

### Alpha CLI Commands

```bash
# Generation (intelligent)
php alpha make:controller UserController --model=User
php alpha make:model User --table=users
php alpha make:resource User

# Package Management
php alpha add database
php alpha add cache

# Schema Inspection
php alpha inspect:schema users
```

---

## 🔗 Documentation References

- **Core Framework:** [Getting Started →](../core/getting-started.md)
- **Modular Architecture:** [Architecture →](modular-architecture.md)
- **Performance:** [Benchmarks →](benchmarks.md)
- **CLI Commands:** [CLI Guide →](../core/cli-commands.md)

---

## 💡 Key Takeaway

> **Alphavel Core** é o engine de performance.  
> **Alpha CLI** é o turbo para produtividade.  
> Juntos, entregam **velocidade de execução + velocidade de desenvolvimento**.

Use Core only para máxima performance. Use Core + Alpha para máxima produtividade. A escolha é sua! 🚀

---

\* *Números de performance variam conforme hardware, configuração e tipo de aplicação. Valores citados são benchmarks sob condições otimizadas.*
