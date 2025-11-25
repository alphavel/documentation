---
layout: default
title: Local Development
---

# 🚀 Quick Guide: Local Development with Alphavel

## For Developers Without Swoole Installed

### Quick Start

```bash
# 1. Create or clone project
composer create-project alphavel/skeleton my-project
cd my-project

# 2. Start development environment
make dev

# Or manually:
docker-compose -f docker-compose.dev.yml up
```

**First run:** May take 2-3 minutes (automatic Swoole installation)  
**Subsequent runs:** Instant

**Access:** http://localhost:9999

---

## What Happens Automatically?

When you run `make dev`, the container:

1. ✅ Installs system dependencies (curl, git, unzip, etc)
2. ✅ Installs required PHP extensions (zip, etc)
3. ✅ Installs and activates Swoole extension
4. ✅ Installs Composer
5. ✅ Installs all project dependencies (`composer install`)
6. ✅ Creates directory structure (storage, bootstrap/cache)
7. ✅ Sets correct permissions
8. ✅ Generates facades.php file
9. ✅ Copies .env.example to .env
10. ✅ Starts Swoole server on port 9999

---

## Useful Commands

### Via Makefile (Recommended)

```bash
# Start development environment
make dev

# Stop environment
make dev-stop

# View real-time logs
make dev-logs

# Access container shell
make dev-shell

# Reinstall/rebuild everything
make dev-rebuild
```

### Via docker-compose

```bash
# Start (foreground, see logs)
docker-compose -f docker-compose.dev.yml up

# Start (background)
docker-compose -f docker-compose.dev.yml up -d

# Stop
docker-compose -f docker-compose.dev.yml down

# View logs
docker-compose -f docker-compose.dev.yml logs -f app

# Access shell
docker-compose -f docker-compose.dev.yml exec app bash

# Run commands
docker-compose -f docker-compose.dev.yml exec app composer require alphavel/database
docker-compose -f docker-compose.dev.yml exec app php -v
```

---

## Port Structure

| Service | Host Port | Container Port | Description |
|---------|-----------|----------------|-------------|
| Application | 9999 | 9999 | Swoole Server |
| MySQL | 3307 | 3306 | Database (dev) |

**Note:** Port 3307 on host to avoid conflicts with local MySQL

---

## Develop in Container

### Install Packages

```bash
# Via make
make dev-shell
composer require alphavel/database

# Or directly
docker-compose -f docker-compose.dev.yml exec app composer require alphavel/database
```

### Run Tests

```bash
docker-compose -f docker-compose.dev.yml exec app vendor/bin/phpunit
```

### Execute Scripts

```bash
docker-compose -f docker-compose.dev.yml exec app php artisan migrate
```

---

## Differences: Dev vs Production

### docker-compose.dev.yml (Development)
- ✅ Installs Swoole automatically
- ✅ Installs dependencies automatically
- ✅ Uses base image `php:8.2-cli`
- ✅ No build required
- ✅ Mounted volumes (synced code)
- ✅ Port 3307 for MySQL (avoids conflict)
- ✅ Verbose logs
- ⚠️ First initialization slower

### docker-compose.yml (Production)
- ✅ Optimized build with Dockerfile
- ✅ Dependencies already in build
- ✅ Production-ready image
- ✅ Faster execution
- ✅ Standard port 3306 for MySQL
- ⚠️ Requires rebuild after code changes

---

## Troubleshooting

### Container doesn't start / stuck on installation

```bash
# View detailed logs
docker-compose -f docker-compose.dev.yml logs -f

# Rebuild from scratch
docker-compose -f docker-compose.dev.yml down -v
docker-compose -f docker-compose.dev.yml up
```

### Permission errors

```bash
# Inside container
docker-compose -f docker-compose.dev.yml exec app bash
chmod -R 777 storage bootstrap/cache
```

### Swoole not installed

```bash
# Force reinstall
docker-compose -f docker-compose.dev.yml exec app pecl install swoole
docker-compose -f docker-compose.dev.yml exec app docker-php-ext-enable swoole
docker-compose -f docker-compose.dev.yml restart app
```

### Port 8080 already in use

Edit `.env` and change port:

```env
APP_PORT=8081
```

Or specify when starting:

```bash
APP_PORT=8081 docker-compose -f docker-compose.dev.yml up
```

---

## Clean Everything

```bash
# Stop and remove containers + volumes
docker-compose -f docker-compose.dev.yml down -v

# Remove vendor and local cache
rm -rf vendor storage/cache/* storage/logs/* bootstrap/cache/*
```

---

## Workflow Comparison

### Without docker-compose.dev.yml (Old)

```bash
# 1. Install Swoole on machine (complex)
sudo pecl install swoole

# 2. Configure PHP
echo "extension=swoole.so" >> /etc/php/8.2/cli/conf.d/20-swoole.ini

# 3. Install dependencies
composer install

# 4. Configure environment
cp .env.example .env
mkdir -p storage/framework storage/logs storage/cache bootstrap/cache
chmod -R 777 storage bootstrap/cache

# 5. Start server
php public/index.php
```

**Problems:**
- ❌ Swoole may not work on macOS/Windows
- ❌ Conflicts with other PHP versions
- ❌ Configuration varies by OS
- ❌ ~10-15 minutes of manual setup

### With docker-compose.dev.yml (New)

```bash
# 1. Start (everything automatic)
make dev
```

**Benefits:**
- ✅ Works on any OS (Linux, macOS, Windows)
- ✅ Isolated and consistent environment
- ✅ No conflicts with local installations
- ✅ ~2-3 minutes of automatic setup
- ✅ Easy to share with team

---

## Best Practices

### For Daily Development

1. Always use `make dev` or `docker-compose -f docker-compose.dev.yml up`
2. Keep container running (don't recreate on every change)
3. Code is synced automatically via volumes
4. For composer.json changes, run `composer install` inside container

### For Committing Code

1. Don't commit generated files (vendor, storage/*, etc)
2. .gitignore is already configured correctly
3. Other developers will use the same docker-compose.dev.yml

### Para CI/CD

1. Use `docker-compose.yml` (production) in the pipeline
2. `docker-compose.dev.yml` is only for local development
3. Tests can run in either environment

---

## FAQ

**Q: Do I need to install Swoole on my machine?**  
A: No! docker-compose.dev.yml installs everything inside the container.

**Q: Are code changes reflected automatically?**  
A: Yes! The code is mounted via volume, changes are instant.

**Q: Can I use my favorite IDE?**  
A: Yes! Edit files normally. The container only executes the code.

**Q: How to debug the code?**  
A: Configure Xdebug (separate instructions) or use `var_dump()` and check logs.

**Q: Is the database persistent?**  
A: Yes! Data is stored in Docker volume (`dbdata-dev`).

**Q: Can I use Redis/Postgres?**  
A: Yes! Add more services to docker-compose.dev.yml as needed.

**Q: Is it slower than local execution?**  
A: Not significantly. Swoole compensates with superior performance.

**Q: Does it work on Windows?**  
A: Yes! As long as you have Docker Desktop installed.

---

## Next Steps

1. ✅ Start environment: `make dev`
2. ✅ Access http://localhost:9999
3. ✅ Read full documentation in README.md
4. ✅ Install additional packages: `composer require alphavel/database`
5. ✅ Start developing!

---

**Questions?** Check full documentation in [README.md](README.html)  
**Issues?** Open an issue on [GitHub](https://github.com/alphavel/skeleton/issues)
