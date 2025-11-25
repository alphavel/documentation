# 🔧 Troubleshooting - Alphavel Framework

Guia de solução de problemas comuns ao trabalhar com Alphavel.

---

## 📋 Índice

1. [Problemas de Permissão](#problemas-de-permissão)
2. [Problemas com Docker](#problemas-com-docker)
3. [Problemas com Composer](#problemas-com-composer)
4. [Problemas de Performance](#problemas-de-performance)
5. [Problemas de Banco de Dados](#problemas-de-banco-de-dados)

---

## 🔒 Problemas de Permissão

### Erro: "Permission denied" ao editar arquivos

**Sintoma:**
```
Permission denied: /var/www/app/Controllers/UserController.php
```

**Causa:**  
Arquivos foram criados/modificados dentro do container com usuário diferente (root ou www-data), impedindo edição no host.

**Solução 1 - Comando rápido:**
```bash
make fix-permissions
```

**Solução 2 - Manual:**
```bash
docker run --rm -v $(pwd):/app -w /app alpine:latest sh -c "\
    chown -R $(id -u):$(id -g) storage bootstrap/cache && \
    chmod -R 775 storage bootstrap/cache"
```

**Prevenção:**  
O Dockerfile agora usa ARG USER_ID/GROUP_ID para coincidir com o usuário do host automaticamente.

---

### Erro: "storage/logs não tem permissão de escrita"

**Sintoma:**
```
Unable to write to /var/www/storage/logs/alphavel.log
```

**Solução:**
```bash
# Via Makefile
make fix-permissions

# Ou manualmente
chmod -R 775 storage bootstrap/cache
```

---

## 🐳 Problemas com Docker

### Container marcado como "unhealthy"

**Sintoma:**
```bash
docker ps
# Mostra: (unhealthy) alphavel-app
```

**Causa:**  
Versões antigas do healthcheck tentavam acessar a rota `/` que não existe.

**Solução:**  
✅ **Já corrigido** - Versão atual usa `/json` endpoint.

**Verificar manualmente:**
```bash
curl http://localhost:9999/json
# Deve retornar: {"message":"Hello, Alphavel!"}
```

Se retornar 200 OK mas container está unhealthy, reconstrua:
```bash
make rebuild
```

---

### "Bind for 0.0.0.0:9999 failed: port is already allocated"

**Causa:**  
Porta 9999 já está em uso por outro processo.

**Solução 1 - Mudar porta:**
```bash
# Edite .env
APP_PORT=8080

# Reinicie
make restart
```

**Solução 2 - Matar processo na porta:**
```bash
# Descobrir processo
lsof -i :9999

# Matar processo (Linux)
sudo kill -9 $(lsof -t -i:9999)
```

---

### Containers não iniciam após rebuild

**Solução:**
```bash
# Parar tudo
docker-compose down -v

# Limpar cache do Docker
docker system prune -a --volumes

# Rebuild limpo
make rebuild
```

---

## 📦 Problemas com Composer

### "Your requirements could not be resolved"

**Causa comum:** Swoole extension não detectada.

**Solução:**
```bash
# Instalar dentro do container
docker-compose exec app composer install --ignore-platform-req=ext-swoole
```

**Ou adicione ao composer.json:**
```json
{
    "config": {
        "platform": {
            "ext-swoole": "5.1.0"
        }
    }
}
```

---

### Composer extremamente lento

**Solução - Habilitar cache do Composer:**
```bash
# No docker-compose.yml, adicione volume:
volumes:
  - ~/.composer:/tmp/composer

# Ou use modo paralelo
docker-compose exec app composer install --prefer-dist --optimize-autoloader
```

---

## ⚡ Problemas de Performance

### "Call to undefined method DatabaseServiceProvider::register()"

**Causa:**  
ServiceProvider antigo não estende `Alphavel\Framework\ServiceProvider`.

**Sintoma:**
```
PHP Fatal error: Call to undefined method Alphavel\Database\DatabaseServiceProvider::register()
```

**Solução - Corrigir estrutura do ServiceProvider:**

```php
<?php

namespace Alphavel\Database;

use Alphavel\Framework\ServiceProvider; // ← DEVE estender esta classe

class DatabaseServiceProvider extends ServiceProvider
{
    public function register(): void // ← Método register() OBRIGATÓRIO
    {
        $this->app->singleton('db', function ($app) {
            return new Database($app->config['database']);
        });
    }

    public function boot(): void
    {
        // Lógica de inicialização
    }
}
```

**Padrão correto:**
- ✅ Estende `Alphavel\Framework\ServiceProvider`
- ✅ Implementa `register(): void`
- ✅ Pode implementar `boot(): void` (opcional)
- ❌ **NÃO use** métodos estáticos

---

### Swoole não está carregando

**Verificar instalação:**
```bash
docker-compose exec app php -m | grep swoole
```

**Se não aparecer, reinstale:**
```bash
docker-compose exec app pecl install swoole
docker-compose exec app docker-php-ext-enable swoole
```

---

### OPcache JIT não funciona

**Verificar:**
```bash
docker-compose exec app php -i | grep jit
```

**Deve mostrar:**
```
opcache.jit => tracing
opcache.jit_buffer_size => 128M
```

**Se não estiver habilitado:**
```bash
# Edite Dockerfile e rebuilde
make rebuild
```

---

## 🗄️ Problemas de Banco de Dados

### "Connection refused" ao conectar no MySQL

**Causa:** Container MySQL não iniciou completamente.

**Solução:**
```bash
# Verificar logs do banco
make logs-db

# Aguardar healthcheck
docker-compose ps

# Deve mostrar: (healthy) alphavel-db
```

---

### "Access denied for user 'alphavel'@'%'"

**Verificar credenciais no .env:**
```env
DB_HOST=db
DB_PORT=3306
DB_DATABASE=alphavel
DB_USERNAME=alphavel
DB_PASSWORD=alphavel
```

**Recriar banco:**
```bash
make db-fresh
```

---

### Migrations não funcionam

**Sintoma:**
```
Migration command not yet implemented
```

**Causa:**  
Alphavel ainda não tem sistema de migrations integrado.

**Solução temporária:**
```bash
# Conectar no MySQL manualmente
make shell-db

# Executar SQL diretamente
CREATE TABLE users (...);
```

---

## 🚀 Dicas de Performance

### Cache de rotas/config (futuro)

```bash
# Quando implementado:
php alpha route:cache
php alpha config:cache
```

### Otimizar autoload

```bash
make composer-dump
```

### Usar cache APCu/Redis

```php
// config/cache.php
return [
    'driver' => 'redis', // ou 'apcu'
    'connection' => [
        'host' => 'redis',
        'port' => 6379,
    ],
];
```

---

## 📞 Ainda com problemas?

1. **Logs da aplicação:**
   ```bash
   make logs
   ```

2. **Logs do banco:**
   ```bash
   make logs-db
   ```

3. **Status dos containers:**
   ```bash
   make status
   ```

4. **Limpar tudo e recomeçar:**
   ```bash
   make clean
   make rebuild
   ```

5. **Reportar bug:**  
   Abra uma issue em: [https://github.com/alphavel/alphavel/issues](https://github.com/alphavel/alphavel/issues)

---

## 🎯 Comandos de Emergência

```bash
# Resetar tudo (CUIDADO: apaga dados)
make clean && make rebuild

# Corrigir permissões
make fix-permissions

# Ver todos os comandos disponíveis
make help

# Acessar shell do container
make shell

# Reinstalar dependências
make composer-install

# Backup do banco antes de experimentos
make backup-db
```

---

**Última atualização:** 20 de novembro de 2025  
**Versão:** 1.0.0
