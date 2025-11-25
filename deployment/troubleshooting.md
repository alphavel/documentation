------

layout: defaultlayout: default

title: Troubleshootingtitle: Troubleshooting

------



# 🔧 Troubleshooting - Alphavel Framework# 🔧 Troubleshooting - Alphavel Framework



Common troubleshooting guide for working with Alphavel.Guia de solução de problemas comuns ao trabalhar com Alphavel.



------



## 📋 Table of Contents## 📋 Índice



1. [Permission Issues](#permission-issues)1. [Problemas de Permissão](#problemas-de-permissão)

2. [Docker Issues](#docker-issues)2. [Problemas com Docker](#problemas-com-docker)

3. [Composer Issues](#composer-issues)3. [Problemas com Composer](#problemas-com-composer)

4. [Performance Issues](#performance-issues)4. [Problemas de Performance](#problemas-de-performance)

5. [Database Issues](#database-issues)5. [Problemas de Banco de Dados](#problemas-de-banco-de-dados)



------



## 🔒 Permission Issues## 🔒 Problemas de Permissão



### Error: "Permission denied" when editing files### Erro: "Permission denied" ao editar arquivos



**Symptom:****Sintoma:**

``````

Permission denied: /var/www/app/Controllers/UserController.phpPermission denied: /var/www/app/Controllers/UserController.php

``````



**Cause:**  **Causa:**  

Files were created/modified inside the container with a different user (root or www-data), preventing editing on the host.Arquivos foram criados/modificados dentro do container com usuário diferente (root ou www-data), impedindo edição no host.



**Solution 1 - Quick command:****Solução 1 - Comando rápido:**

```bash```bash

make fix-permissionsmake fix-permissions

``````



**Solution 2 - Manual:****Solução 2 - Manual:**

```bash```bash

docker run --rm -v $(pwd):/app -w /app alpine:latest sh -c "\docker run --rm -v $(pwd):/app -w /app alpine:latest sh -c "\

    chown -R $(id -u):$(id -g) storage bootstrap/cache && \    chown -R $(id -u):$(id -g) storage bootstrap/cache && \

    chmod -R 775 storage bootstrap/cache"    chmod -R 775 storage bootstrap/cache"

``````



**Prevention:**  **Prevenção:**  

The Dockerfile now uses ARG USER_ID/GROUP_ID to automatically match the host user.O Dockerfile agora usa ARG USER_ID/GROUP_ID para coincidir com o usuário do host automaticamente.



------



### Error: "storage/logs doesn't have write permission"### Erro: "storage/logs não tem permissão de escrita"



**Symptom:****Sintoma:**

``````

Unable to write to /var/www/storage/logs/alphavel.logUnable to write to /var/www/storage/logs/alphavel.log

``````



**Solution:****Solução:**

```bash```bash

# Via Makefile# Via Makefile

make fix-permissionsmake fix-permissions



# Or manually# Ou manualmente

chmod -R 775 storage bootstrap/cachechmod -R 775 storage bootstrap/cache

``````



------



## 🐳 Docker Issues## 🐳 Problemas com Docker



### Container marked as "unhealthy"### Container marcado como "unhealthy"



**Symptom:****Sintoma:**

```bash```bash

docker psdocker ps

# Shows: (unhealthy) alphavel-app# Mostra: (unhealthy) alphavel-app

``````



**Cause:**  **Causa:**  

Old healthcheck versions tried to access the `/` route which doesn't exist.Versões antigas do healthcheck tentavam acessar a rota `/` que não existe.



**Solution:**  **Solução:**  

✅ **Already fixed** - Current version uses `/json` endpoint.✅ **Já corrigido** - Versão atual usa `/json` endpoint.



**Manual verification:****Verificar manualmente:**

```bash```bash

curl http://localhost:9999/jsoncurl http://localhost:9999/json

# Should return: {"message":"Hello, Alphavel!"}# Deve retornar: {"message":"Hello, Alphavel!"}

``````



If it returns 200 OK but container is unhealthy, rebuild:Se retornar 200 OK mas container está unhealthy, reconstrua:

```bash```bash

make rebuildmake rebuild

``````



------



### "Bind for 0.0.0.0:9999 failed: port is already allocated"### "Bind for 0.0.0.0:9999 failed: port is already allocated"



**Cause:**  **Causa:**  

Port 9999 is already in use by another process.Porta 9999 já está em uso por outro processo.



**Solution 1 - Change port:****Solução 1 - Mudar porta:**

```bash```bash

# Edit .env# Edite .env

APP_PORT=8080APP_PORT=8080



# Restart# Reinicie

make restartmake restart

``````



**Solution 2 - Kill process on port:****Solução 2 - Matar processo na porta:**

```bash```bash

# Find process# Descobrir processo

lsof -i :9999lsof -i :9999



# Kill process (Linux)# Matar processo (Linux)

sudo kill -9 $(lsof -t -i:9999)sudo kill -9 $(lsof -t -i:9999)

``````



------



### Containers don't start after rebuild### Containers não iniciam após rebuild



**Solution:****Solução:**

```bash```bash

# Stop everything# Parar tudo

docker-compose down -vdocker-compose down -v



# Clean Docker cache# Limpar cache do Docker

docker system prune -a --volumesdocker system prune -a --volumes



# Clean rebuild# Rebuild limpo

make rebuildmake rebuild

``````



------



## 📦 Composer Issues## 📦 Problemas com Composer



### "Your requirements could not be resolved"### "Your requirements could not be resolved"



**Common cause:** Swoole extension not detected.**Causa comum:** Swoole extension não detectada.



**Solution:****Solução:**

```bash```bash

# Install inside container# Instalar dentro do container

docker-compose exec app composer install --ignore-platform-req=ext-swooledocker-compose exec app composer install --ignore-platform-req=ext-swoole

``````



**Or add to composer.json:****Ou adicione ao composer.json:**

```json```json

{{

    "config": {    "config": {

        "platform": {        "platform": {

            "ext-swoole": "5.1.0"            "ext-swoole": "5.1.0"

        }        }

    }    }

}}

``````



------



### Composer extremely slow### Composer extremamente lento



**Solution - Enable Composer cache:****Solução - Habilitar cache do Composer:**

```bash```bash

# In docker-compose.yml, add volume:# No docker-compose.yml, adicione volume:

volumes:volumes:

  - ~/.composer:/tmp/composer  - ~/.composer:/tmp/composer



# Or use parallel mode# Ou use modo paralelo

docker-compose exec app composer install --prefer-dist --optimize-autoloaderdocker-compose exec app composer install --prefer-dist --optimize-autoloader

``````



------



## ⚡ Performance Issues## ⚡ Problemas de Performance



### "Call to undefined method DatabaseServiceProvider::register()"### "Call to undefined method DatabaseServiceProvider::register()"



**Cause:**  **Causa:**  

Old ServiceProvider doesn't extend `Alphavel\Framework\ServiceProvider`.ServiceProvider antigo não estende `Alphavel\Framework\ServiceProvider`.



**Symptom:****Sintoma:**

``````

PHP Fatal error: Call to undefined method Alphavel\Database\DatabaseServiceProvider::register()PHP Fatal error: Call to undefined method Alphavel\Database\DatabaseServiceProvider::register()

``````



**Solution - Fix ServiceProvider structure:****Solução - Corrigir estrutura do ServiceProvider:**



{% raw %}{% raw %}

```php```php

<?php<?php



namespace Alphavel\Database;namespace Alphavel\Database;



use Alphavel\Framework\ServiceProvider; // ← MUST extend this classuse Alphavel\Framework\ServiceProvider; // ← DEVE estender esta classe



class DatabaseServiceProvider extends ServiceProviderclass DatabaseServiceProvider extends ServiceProvider

{{

    public function register(): void // ← register() method REQUIRED    public function register(): void // ← Método register() OBRIGATÓRIO

    {    {

        $this->app->singleton('db', function ($app) {        $this->app->singleton('db', function ($app) {

            return new Database($app->config['database']);            return new Database($app->config['database']);

        });        });

    }    }



    public function boot(): void    public function boot(): void

    {    {

        // Initialization logic        // Lógica de inicialização

    }    }

}}

``````

{% endraw %}{% endraw %}



**Correct pattern:****Padrão correto:**

- ✅ Extends `Alphavel\Framework\ServiceProvider`- ✅ Estende `Alphavel\Framework\ServiceProvider`

- ✅ Implements `register(): void`- ✅ Implementa `register(): void`

- ✅ Can implement `boot(): void` (optional)- ✅ Pode implementar `boot(): void` (opcional)

- ❌ **DO NOT use** static methods- ❌ **NÃO use** métodos estáticos



------



### Swoole is not loading### Swoole não está carregando



**Check installation:****Verificar instalação:**

```bash```bash

docker-compose exec app php -m | grep swooledocker-compose exec app php -m | grep swoole

``````



**If it doesn't appear, reinstall:****Se não aparecer, reinstale:**

```bash```bash

docker-compose exec app pecl install swooledocker-compose exec app pecl install swoole

docker-compose exec app docker-php-ext-enable swooledocker-compose exec app docker-php-ext-enable swoole

``````



------



### OPcache JIT not working### OPcache JIT não funciona



**Check:****Verificar:**

```bash```bash

docker-compose exec app php -i | grep jitdocker-compose exec app php -i | grep jit

``````



**Should show:****Deve mostrar:**

``````

opcache.jit => tracingopcache.jit => tracing

opcache.jit_buffer_size => 128Mopcache.jit_buffer_size => 128M

``````



**If not enabled:****Se não estiver habilitado:**

```bash```bash

# Edit Dockerfile and rebuild# Edite Dockerfile e rebuilde

make rebuildmake rebuild

``````



------



## 🗄️ Database Issues## 🗄️ Problemas de Banco de Dados



### "Connection refused" when connecting to MySQL### "Connection refused" ao conectar no MySQL



**Cause:** MySQL container didn't fully start.**Causa:** Container MySQL não iniciou completamente.



**Solution:****Solução:**

```bash```bash

# Check database logs# Verificar logs do banco

make logs-dbmake logs-db



# Wait for healthcheck# Aguardar healthcheck

docker-compose psdocker-compose ps



# Should show: (healthy) alphavel-db# Deve mostrar: (healthy) alphavel-db

``````



------



### "Access denied for user 'alphavel'@'%'"### "Access denied for user 'alphavel'@'%'"



**Check credentials in .env:****Verificar credenciais no .env:**

```env```env

DB_HOST=dbDB_HOST=db

DB_PORT=3306DB_PORT=3306

DB_DATABASE=alphavelDB_DATABASE=alphavel

DB_USERNAME=alphavelDB_USERNAME=alphavel

DB_PASSWORD=alphavelDB_PASSWORD=alphavel

``````



**Recreate database:****Recriar banco:**

```bash```bash

make db-freshmake db-fresh

``````



------



### Migrations don't work### Migrations não funcionam



**Symptom:****Sintoma:**

``````

Migration command not yet implementedMigration command not yet implemented

``````



**Cause:**  **Causa:**  

Alphavel doesn't have an integrated migration system yet.Alphavel ainda não tem sistema de migrations integrado.



**Temporary solution:****Solução temporária:**

```bash```bash

# Connect to MySQL manually# Conectar no MySQL manualmente

make shell-dbmake shell-db



# Execute SQL directly# Executar SQL diretamente

CREATE TABLE users (...);CREATE TABLE users (...);

``````



------



## 🚀 Performance Tips## 🚀 Dicas de Performance



### Route/config cache (future)### Cache de rotas/config (futuro)



```bash```bash

# When implemented:# Quando implementado:

php alpha route:cachephp alpha route:cache

php alpha config:cachephp alpha config:cache

``````



### Optimize autoload### Otimizar autoload



```bash```bash

make composer-dumpmake composer-dump

``````



### Use APCu/Redis cache### Usar cache APCu/Redis



{% raw %}{% raw %}

```php```php

// config/cache.php// config/cache.php

return [return [

    'driver' => 'redis', // or 'apcu'    'driver' => 'redis', // ou 'apcu'

    'connection' => [    'connection' => [

        'host' => 'redis',        'host' => 'redis',

        'port' => 6379,        'port' => 6379,

    ],    ],

];];

``````

{% endraw %}{% endraw %}



------



## 📞 Still having problems?## 📞 Ainda com problemas?



1. **Application logs:**1. **Logs da aplicação:**

   ```bash   ```bash

   make logs   make logs

   ```   ```



2. **Database logs:**2. **Logs do banco:**

   ```bash   ```bash

   make logs-db   make logs-db

   ```   ```



3. **Container status:**3. **Status dos containers:**

   ```bash   ```bash

   make status   make status

   ```   ```



4. **Clean everything and restart:**4. **Limpar tudo e recomeçar:**

   ```bash   ```bash

   make clean   make clean

   make rebuild   make rebuild

   ```   ```



5. **Report bug:**  5. **Reportar bug:**  

   Open an issue at: [https://github.com/alphavel/alphavel/issues](https://github.com/alphavel/alphavel/issues)   Abra uma issue em: [https://github.com/alphavel/alphavel/issues](https://github.com/alphavel/alphavel/issues)



------



## 🎯 Emergency Commands## 🎯 Comandos de Emergência



```bash```bash

# Reset everything (WARNING: deletes data)# Resetar tudo (CUIDADO: apaga dados)

make clean && make rebuildmake clean && make rebuild



# Fix permissions# Corrigir permissões

make fix-permissionsmake fix-permissions



# See all available commands# Ver todos os comandos disponíveis

make helpmake help



# Access container shell# Acessar shell do container

make shellmake shell



# Reinstall dependencies# Reinstalar dependências

make composer-installmake composer-install



# Database backup before experiments# Backup do banco antes de experimentos

make backup-dbmake backup-db

``````



------



**Last updated:** November 20, 2025  **Última atualização:** 20 de novembro de 2025  

**Version:** 1.0.0**Versão:** 1.0.0

