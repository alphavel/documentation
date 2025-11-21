# CLI Commands (Alpha)

The Alphavel CLI (`alphavel` or `./alphavel`) is your command center for managing applications, packages, and performance optimization.

---

## Overview

```bash
./alphavel [command] [options]
```

**Available Commands:**
- `serve` - Start Swoole HTTP server
- `package:add` - Install and configure packages
- `package:discover` - Discover installed packages
- `route:cache` - Compile routes for production
- `route:clear` - Clear route cache

---

## Server Commands

### `serve` - Start Server

Start the Swoole HTTP server for development or production.

**Basic Usage:**

```bash
./alphavel serve
```

**Output:**
```
🚀 Alphavel HTTP Server
┌─────────────────────────────────────────┐
│ Server started successfully!            │
│ Host:     0.0.0.0                       │
│ Port:     9501                          │
│ Workers:  4                             │
│ Mode:     development                   │
└─────────────────────────────────────────┘

Press Ctrl+C to stop
```

**Custom Host/Port:**

```bash
./alphavel serve --host=127.0.0.1 --port=8000
```

**Custom Workers:**

```bash
# Use more workers for production
./alphavel serve --workers=8

# Auto-detect (2x CPU cores)
./alphavel serve --workers=auto
```

**Production Mode:**

```bash
# Disable debug, enable optimizations
APP_ENV=production ./alphavel serve
```

**Options:**

| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `--host` | Bind host | `0.0.0.0` | `--host=127.0.0.1` |
| `--port` | Bind port | `9501` | `--port=8000` |
| `--workers` | Worker processes | `4` or auto | `--workers=8` |
| `--daemon` | Run as daemon | `false` | `--daemon` |

**Examples:**

```bash
# Development (single worker for debugging)
./alphavel serve --workers=1

# Production (high traffic)
./alphavel serve --workers=16 --daemon

# Custom configuration
./alphavel serve --host=0.0.0.0 --port=80 --workers=auto
```

---

## Package Management

### `package:add` - Install Package

Install and auto-configure packages with one command.

**Usage:**

```bash
./alphavel package:add [package-name]
```

**Available Packages:**

```bash
# Database (MySQL/PostgreSQL + connection pooling)
./alphavel package:add database

# Cache (Redis/Memcached)
./alphavel package:add cache

# Logging (Monolog)
./alphavel package:add logging

# Events (Event dispatcher)
./alphavel package:add events

# Validation (Request validation)
./alphavel package:add validation

# Support (Helper utilities)
./alphavel package:add support
```

**What Happens:**

1. ✅ Installs package via Composer
2. ✅ Publishes configuration files
3. ✅ Registers service provider
4. ✅ Updates package manifest
5. ✅ Runs post-install recipe (if any)

**Example Output:**

```bash
$ ./alphavel package:add database

📦 Installing alphavel/database...

✓ Package installed via Composer
✓ Configuration published to config/database.php
✓ Service provider registered
✓ Package manifest updated

🎉 Package alphavel/database installed successfully!

Next steps:
  1. Configure your database in .env:
     DB_HOST=127.0.0.1
     DB_DATABASE=myapp
     DB_USERNAME=root
     DB_PASSWORD=

  2. Use in your controllers:
     use Alphavel\Database\DB;
     
     $users = DB::query('SELECT * FROM users');
```

**Package Aliases:**

Instead of full package names, use short aliases:

```bash
./alphavel package:add db        # alias for database
./alphavel package:add redis     # alias for cache
./alphavel package:add log       # alias for logging
```

**Bulk Installation:**

```bash
# Install multiple packages
./alphavel package:add database cache logging
```

---

### `package:discover` - Discover Packages

Scan vendor directory and rebuild package manifest.

**Usage:**

```bash
./alphavel package:discover
```

**When to Use:**

- After manually installing packages via Composer
- After updating composer dependencies
- When service providers aren't loading

**Output:**

```bash
$ ./alphavel package:discover

🔍 Discovering packages...

Found packages:
  ✓ alphavel/database (v1.0.0)
    - Providers: DatabaseServiceProvider
    - Aliases: DB
  
  ✓ alphavel/cache (v1.0.0)
    - Providers: CacheServiceProvider
    - Aliases: Cache
  
  ✓ alphavel/logging (v1.0.0)
    - Providers: LoggingServiceProvider
    - Aliases: Log

✓ Package manifest updated: bootstrap/cache/packages.php

3 packages discovered
```

---

## Performance Commands

### `route:cache` - Compile Routes

Pre-compile routes for zero-overhead routing in production.

**Usage:**

```bash
./alphavel route:cache
```

**What It Does:**

1. Loads all routes from `routes/api.php`
2. Separates into categories (raw, static, dynamic)
3. Serializes to PHP array
4. Writes to `bootstrap/cache/routes.php`

**Output:**

```bash
$ ./alphavel route:cache

⚡ Caching routes...

✓ Routes loaded from routes/api.php
✓ Routes compiled and optimized
✓ Cache written to bootstrap/cache/routes.php

Routes cached successfully!
  Raw routes (zero overhead): 4
  Static routes: 8
  Dynamic routes: 5
  Total: 17

⚡ Route lookup is now optimized for production!

Performance impact:
  • Route lookup: +15-20% faster
  • Memory usage: -5MB
  • Startup time: -50ms
```

**Impact:**

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Route lookup | 0.12ms | 0.01ms | **12x faster** |
| Memory | 25MB | 20MB | -5MB |
| Startup | 100ms | 50ms | -50% |

**Production Workflow:**

```bash
# 1. Test locally
./alphavel route:cache
./alphavel serve

# 2. Deploy
git add bootstrap/cache/routes.php
git commit -m "chore: Cache routes for production"
git push

# 3. On production server
docker-compose up -d
```

---

### `route:clear` - Clear Route Cache

Remove cached routes and reload from source.

**Usage:**

```bash
./alphavel route:clear
```

**When to Use:**

- After modifying routes
- When routes aren't updating
- Debugging routing issues

**Output:**

```bash
$ ./alphavel route:clear

✓ Route cache cleared: bootstrap/cache/routes.php

Routes will be loaded from source on next request.
```

---

## Advanced Usage

### Chaining Commands

```bash
# Clear cache, re-cache, and start server
./alphavel route:clear && ./alphavel route:cache && ./alphavel serve
```

### Environment-Specific Commands

```bash
# Development
APP_ENV=local ./alphavel serve --workers=1

# Staging
APP_ENV=staging ./alphavel serve --workers=4

# Production
APP_ENV=production ./alphavel serve --workers=16 --daemon
```

### Docker Integration

```bash
# Inside Docker container
docker-compose exec app ./alphavel route:cache
docker-compose exec app ./alphavel package:add database
docker-compose exec app ./alphavel serve
```

---

## Creating Custom Commands

Create your own CLI commands by extending the `Command` class.

**1. Create Command File:**

`app/Console/Commands/SyncUsersCommand.php`:

```php
<?php

namespace App\Console\Commands;

use Alphavel\Alpha\Console\Command;
use Alphavel\Database\DB;

class SyncUsersCommand extends Command
{
    protected string $signature = 'users:sync';
    protected string $description = 'Sync users from external API';

    public function handle(): int
    {
        $this->info('Syncing users...');

        // Your logic here
        $users = file_get_contents('https://api.example.com/users');
        $users = json_decode($users, true);

        foreach ($users as $user) {
            DB::table('users')->insert([
                'name' => $user['name'],
                'email' => $user['email'],
            ]);
            
            $this->comment("✓ Synced: {$user['name']}");
        }

        $this->success("Synced " . count($users) . " users!");

        return self::SUCCESS;
    }
}
```

**2. Register Command:**

`config/app.php`:

```php
'commands' => [
    App\Console\Commands\SyncUsersCommand::class,
],
```

**3. Run Command:**

```bash
./alphavel users:sync
```

**Output:**

```bash
$ ./alphavel users:sync

ℹ Syncing users...
✓ Synced: John Doe
✓ Synced: Jane Smith
✓ Synced: Bob Johnson

✅ Synced 3 users!
```

**Command Methods:**

```php
// Output methods
$this->info('Information message');
$this->success('Success message');
$this->error('Error message');
$this->warning('Warning message');
$this->comment('Comment message');

// Ask for input
$name = $this->ask('What is your name?');
$confirmed = $this->confirm('Are you sure?');

// Progress bar
$this->progressStart(100);
for ($i = 0; $i < 100; $i++) {
    $this->progressAdvance();
}
$this->progressFinish();

// Table
$this->table(
    ['ID', 'Name', 'Email'],
    [
        [1, 'John', 'john@example.com'],
        [2, 'Jane', 'jane@example.com'],
    ]
);
```

---

## Shell Scripts Integration

### Automated Deployment

`deploy.sh`:

```bash
#!/bin/bash

echo "🚀 Deploying Alphavel application..."

# Pull latest code
git pull origin main

# Install dependencies
composer install --no-dev --optimize-autoloader

# Cache routes
./alphavel route:cache

# Restart server
docker-compose restart app

echo "✅ Deployment complete!"
```

### Cron Jobs

`crontab -e`:

```bash
# Run every hour
0 * * * * cd /path/to/app && ./alphavel users:sync >> /var/log/sync.log 2>&1

# Clear cache daily
0 0 * * * cd /path/to/app && ./alphavel route:clear && ./alphavel route:cache
```

---

## Troubleshooting

### Command Not Found

```bash
$ ./alphavel serve
bash: ./alphavel: Permission denied
```

**Solution:**

```bash
chmod +x alphavel
./alphavel serve
```

### Swoole Not Installed

```bash
$ ./alphavel serve
Error: Swoole extension not installed
```

**Solution:**

```bash
# Using Docker (recommended)
docker-compose -f docker-compose.dev.yml up

# Or install Swoole
pecl install swoole
```

### Port Already in Use

```bash
$ ./alphavel serve
Error: Address already in use (port 9501)
```

**Solution:**

```bash
# Use different port
./alphavel serve --port=9502

# Or kill existing process
lsof -ti:9501 | xargs kill -9
```

---

## Performance Tips

### Production Optimization

```bash
# 1. Cache routes
./alphavel route:cache

# 2. Optimize Composer autoloader
composer dump-autoload --optimize --classmap-authoritative

# 3. Enable OPcache (php.ini)
opcache.enable=1
opcache.validate_timestamps=0

# 4. Use more workers
./alphavel serve --workers=16

# 5. Run as daemon
./alphavel serve --daemon
```

### Monitoring

```bash
# Check Swoole stats
curl http://localhost:9501/metrics

# Watch logs
tail -f storage/logs/alphavel.log

# Monitor resources
docker stats
```

---

## Next Steps

- [Performance Guide →](performance.md)
- [Deployment →](../deployment/production.md)
- [Custom Commands →](custom-commands.md)
