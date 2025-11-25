# Facades no Alphavel Framework

## 📖 O que são Facades?

Facades são **aliases estáticos** para classes registradas no container de injeção de dependências. Eles fornecem uma interface "estática" conveniente para classes que estão disponíveis no service container.

## ⚙️ Como Funcionam

### Sem Facade (Injeção de Dependência)
```php
use Alphavel\Database\DB;

class UserController extends Controller
{
    private $db;
    
    public function __construct(DB $db)
    {
        $this->db = $db;
    }
    
    public function index()
    {
        $users = $this->db->table('users')->get();
        return response()->json($users);
    }
}
```

### Com Facade (Acesso Estático)
```php
use DB; // Facade

class UserController extends Controller
{
    public function index()
    {
        $users = DB::table('users')->get();
        return response()->json($users);
    }
}
```

## 🎯 São Necessárias?

**NÃO!** No Alphavel Framework, facades são **completamente opcionais**.

### O Framework é Totalmente Modular

```php
// Forma 1: Usar o helper app() (recomendado)
$users = app('db')->table('users')->get();

// Forma 2: Injeção de dependências (melhor prática)
public function __construct(DB $db) {
    $this->db = $db;
}

// Forma 3: Facades (conveniência, requer geração)
$users = DB::table('users')->get();
```

## 🔧 Como Gerar Facades

Facades são geradas **sob demanda** para suporte de IDE:

```bash
# Gerar arquivo de facades para auto-complete da IDE
php alphavel ide-helper

# Isso cria: storage/framework/facades.php
```

## 📁 Arquivo `facades.php`

### Localização
```
storage/framework/facades.php
```

### Conteúdo (Exemplo)
```php
<?php

namespace {
    /**
     * @method static mixed table(string $table)
     * @method static mixed select(string $sql, array $bindings = [])
     * @see \Alphavel\Database\DB
     */
    class DB extends \Alphavel\Framework\Facade {}
    
    /**
     * @method static mixed get(string $key, mixed $default = null)
     * @method static bool put(string $key, mixed $value, int $ttl = null)
     * @see \Alphavel\Cache\Cache
     */
    class Cache extends \Alphavel\Framework\Facade {}
}
```

### Por que não é versionado?

O arquivo `facades.php` está no `.gitignore` porque:

1. **Gerado automaticamente**: Cada desenvolvedor pode gerá-lo localmente
2. **Específico do ambiente**: Depende dos pacotes instalados
3. **Opcional**: Não é necessário para o funcionamento do framework
4. **IDE-specific**: Usado apenas para auto-complete

## 🚀 Quando Usar Facades?

### ✅ Bom para:

- **Prototipagem rápida**: código mais conciso
- **Scripts simples**: menos boilerplate
- **Helpers globais**: acesso rápido em views/templates

### ❌ Evite em:

- **Aplicações grandes**: dificulta testes e manutenção
- **Código testável**: injeção de dependências é melhor
- **APIs públicas**: contratos explícitos são mais claros

## 🎓 Melhores Práticas

### 1. Prefira Injeção de Dependências

```php
// ✅ RECOMENDADO
class UserService
{
    public function __construct(
        private DB $db,
        private Cache $cache
    ) {}
    
    public function getUsers(): array
    {
        return $this->cache->remember('users', 3600, function() {
            return $this->db->table('users')->get();
        });
    }
}
```

### 2. Use Helper `app()` para Acesso Rápido

```php
// ✅ BOM para scripts e helpers
function getActiveUsers(): array
{
    return app('db')->table('users')
        ->where('active', true)
        ->get();
}
```

### 3. Facades para Protótipos

```php
// ✅ OK para MVPs e protótipos rápidos
Route::get('/users', function() {
    return DB::table('users')->get();
});
```

## 🔍 IDE Auto-Complete

### Visual Studio Code

```bash
# 1. Gerar facades
php alphavel ide-helper

# 2. Instalar PHP Intelephense (recomendado)
# Extension ID: bmewburn.vscode-intelephense-client
```

### PhpStorm

```bash
# 1. Gerar facades
php alphavel ide-helper

# 2. PhpStorm detecta automaticamente o arquivo
# Settings > PHP > Include Path > Add storage/framework/facades.php
```

## 📦 Pacotes Suportados

Facades são geradas para:

- `DB` - alphavel/database
- `Cache` - alphavel/cache
- `Log` - alphavel/logging
- `Event` - alphavel/events
- `Validator` - alphavel/validation

## 🔄 Regenerar Facades

```bash
# Limpar facades antigas
php alphavel facade:clear

# Gerar novas facades
php alphavel ide-helper

# Ou fazer tudo de uma vez
php alphavel facade:clear && php alphavel ide-helper
```

## 🐳 Docker e Facades

### Facades não são necessárias no Docker

O Dockerfile **não** cria o arquivo `facades.php` porque:

1. O container roda perfeitamente sem ele
2. Reduz o tamanho da imagem
3. Evita arquivos desnecessários em produção
4. Facades são apenas para **desenvolvimento local**

### Se precisar em desenvolvimento Docker

```bash
# Acessar container
docker exec -it alphavel-app bash

# Gerar facades
php alphavel ide-helper
```

## 🎯 Resumo

| Característica | Status |
|---------------|--------|
| **Obrigatório?** | ❌ Não |
| **Framework funciona sem?** | ✅ Sim, perfeitamente |
| **Útil para IDE?** | ✅ Sim, auto-complete |
| **Versionado no Git?** | ❌ Não (gerado localmente) |
| **Usado em produção?** | ❌ Não recomendado |
| **Bom para testes?** | ❌ Dificulta mock/stub |

## 📚 Leitura Adicional

- [Laravel Facades Documentation](https://laravel.com/docs/facades)
- [Dependency Injection vs Facades](https://laravel.com/docs/facades#facades-vs-dependency-injection)
- [Service Container](https://laravel.com/docs/container)

---

**Alphavel Framework** - Modular, rápido e flexível! 🚀
