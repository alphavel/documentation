---
layout: default
title: README
---

# Alphavel Framework Documentation

> High-performance PHP framework powered by Swoole, achieving 520k+ req/s

[![PHP Version](https://img.shields.io/badge/php-%3E%3D8.1-blue.svg)](https://php.net)
[![Swoole](https://img.shields.io/badge/swoole-required-red.svg)](https://www.swoole.co.uk/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

---

## 🚀 What is Alphavel?

Alphavel is a **high-performance microservice framework** built on Swoole, designed for building ultra-fast API applications. It combines Laravel-like developer experience with raw Swoole performance.

### Key Features

- ⚡ **520k+ req/s** on plaintext benchmarks
- 🚀 **Raw Routes** with zero overhead (45k+ req/s)
- 🔄 **Connection Pooling** for MySQL/PostgreSQL
- 📦 **Modular Packages** - install only what you need
- 🎯 **Auto-Discovery** - packages register automatically
- 🛠️ **Developer Experience** - Laravel-like API
- 🐳 **Docker Ready** - production-grade containers
- ⚙️ **Route Caching** - compile routes for production

---

## 📖 Documentation Structure

### Core Framework
- [Getting Started](core/getting-started.md)
- [Installation](core/installation.md)
- [Configuration](core/configuration.md)
- [Routing](core/routing.md)
- [Raw Routes](core/raw-routes.md)
- [Controllers](core/controllers.md)
- [Request & Response](core/request-response.md)
- [Middleware](core/middleware.md)
- [Dependency Injection](core/container.md)
- [Service Providers](core/service-providers.md)
- [CLI Commands](core/cli-commands.md)
- [Performance](core/performance.md)

### Packages

#### Core Packages (TIER 1)
- **[Database](packages/database/README.md)** - MySQL/PostgreSQL (+2,674% performance)
  - [Laravel-Style Guide](packages/database/LARAVEL_STYLE_GUIDE.md) 💚
  - [Performance Optimizations](packages/database/PERFORMANCE_OPTIMIZATIONS.md) ⚡
  - [Aggressive Caching](packages/database/AGGRESSIVE_CACHING_GUIDE.md) 🔥
- **[Support](packages/support/README.md)** - Collections and helper utilities
- **[Cache](packages/cache/README.md)** - Redis/Memcached support
- **[Logging](packages/logging/README.md)** - Monolog integration
- **[Events](packages/events/README.md)** - Event dispatcher
- **[Validation](packages/validation/README.md)** - Request validation

#### High-Performance Packages (TIER 2)
- **[WebSocket](packages/websocket/README.md)** 📡 - Real-time server (500k+ msgs/s, <1ms)
- **[Circuit Breaker](packages/circuit-breaker/README.md)** 🛡️ - Resilience pattern (<0.1ms overhead)
- **[Rate Limit](packages/rate-limit/README.md)** - Swoole Table rate limiting

#### Optional Packages (TIER 2)
- **[Auth](packages/auth/README.md)** - JWT authentication with Guards
- **[Queue](packages/queue/README.md)** - Async job queue with Swoole coroutines
- **[Mail](packages/mail/README.md)** - Email sending with Symfony Mailer
- **[Session](packages/session/README.md)** - Session management (File/Redis/Memory)
- **[View](packages/view/README.md)** - Blade template engine
- **[i18n](packages/i18n/README.md)** - Internationalization/translations
- **[Testing](packages/testing/README.md)** - Testing utilities and helpers
- **[ORM](packages/orm/README.md)** - Advanced ORM with relationships

### Deployment
- [Docker](deployment/docker.md)
- [Production](deployment/production.md)
- [Performance Tuning](deployment/performance-tuning.md)
- [Troubleshooting](deployment/troubleshooting.md)

---

## ⚡ Quick Start

### Installation

```bash
composer create-project alphavel/skeleton my-app
cd my-app

# Docker (recommended)
docker-compose -f docker-compose.dev.yml up

# Or local Swoole
php alpha serve
```

### Your First Route

```php
// routes/api.php

// Raw route (zero overhead)
$router->raw('/health', ['status' => 'ok'], 'application/json');

// Standard route
$router->get('/users', 'App\Controllers\UserController@index');
```

### Add Packages

```bash
# Using Alpha CLI Wizards (recommended - interactive setup)
php alpha make:auth          # JWT authentication
php alpha make:queue         # Async job queue
php alpha make:mail          # Email configuration
php alpha make:session       # Session setup
php alpha make:view          # Template creation
php alpha make:translation   # i18n setup
php alpha make:test          # Testing utilities
php alpha make:relationship  # ORM relationships

# Or install packages directly
composer require alphavel/auth
composer require alphavel/queue
composer require alphavel/mail
```

> **Note:** CLI wizards detect if packages are installed and offer installation + configuration. Install Alpha CLI with `composer require alphavel/alpha --dev`.

---

## 📊 Performance

### High-Performance Hardware (Theoretical Max)
| Benchmark | Requests/sec | Latency | Hardware |
|-----------|--------------|---------|----------|
| Raw Routes | 520,000+ | 0.19ms | 4 cores, 8GB |
| JSON API | 45,000+ | 2.2ms | 4 cores, 8GB |
| Database Query | 20,000+ | 5ms | 4 cores, 8GB |

### Production Constraints (Real-World)
| Framework | Requests/sec | Hardware |
|-----------|--------------|----------|
| Alphavel | 5,042 | 0.5 CPU, 512MB |
| Hyperf | 1,050 | 0.5 CPU, 512MB |
| **Advantage** | **4.8x faster** | Same resources |

> **See [Benchmarks](introduction/benchmarks.md) for detailed performance analysis.**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│         Swoole HTTP Server              │
│         (Event-Driven, Non-Blocking)    │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│         Application Router              │
│  ┌────────────┬──────────┬─────────┐   │
│  │ Raw Routes │  Static  │ Dynamic │   │
│  │ (0 overhead│  (O(1))  │ (Regex) │   │
│  └────────────┴──────────┴─────────┘   │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│         Middleware Pipeline             │
│         (Auth, CORS, etc)               │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│         Controller Layer                │
│         (Autowiring, DI)                │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│         Response                        │
│         (JSON, HTML, etc)               │
└─────────────────────────────────────────┘
```

---

## 🎯 Use Cases

### ✅ Perfect For:
- 🔥 High-traffic APIs
- 🚀 Microservices
- ⚡ Real-time applications
- 📊 Data processing services
- 🎮 Game backends
- 📱 Mobile app backends

### ⚠️ Not Ideal For:
- Traditional CMS (WordPress-like)
- Applications requiring shared hosting
- Projects without Swoole support

---

## 📦 Ecosystem

### Core Packages
| Package | Description | Version |
|---------|-------------|---------|
| [alphavel/alphavel](https://github.com/alphavel/alphavel) | Core framework | ^1.0 |
| [alphavel/alpha](https://github.com/alphavel/alpha) | CLI tools | ^1.0 |
| [alphavel/database](https://github.com/alphavel/database) | Database layer | ^1.0 |
| [alphavel/cache](https://github.com/alphavel/cache) | Caching | ^1.0 |
| [alphavel/logging](https://github.com/alphavel/logging) | Logging | ^1.0 |
| [alphavel/events](https://github.com/alphavel/events) | Events | ^1.0 |
| [alphavel/validation](https://github.com/alphavel/validation) | Validation | ^1.0 |
| [alphavel/rate-limit](https://github.com/alphavel/rate-limit) | Rate limiting | ^1.0 |
| [alphavel/skeleton](https://github.com/alphavel/skeleton) | App skeleton | ^1.0 |

### Optional Packages (New in v1.0)
| Package | Description | Version | Status |
|---------|-------------|---------|--------|
| [alphavel/auth](https://github.com/alphavel/auth) | JWT authentication | ^1.0 | ✅ Ready |
| [alphavel/queue](https://github.com/alphavel/queue) | Async job queue | ^1.0 | 🔄 TIER 1 |
| [alphavel/mail](https://github.com/alphavel/mail) | Email sending | ^1.0 | 🔄 TIER 1 |
| [alphavel/session](https://github.com/alphavel/session) | Session management | ^1.0 | 🔄 TIER 1 |
| [alphavel/view](https://github.com/alphavel/view) | Blade templates | ^1.0 | 🔄 TIER 1 |
| [alphavel/i18n](https://github.com/alphavel/i18n) | Internationalization | ^1.0 | 🔄 TIER 1 |
| [alphavel/testing](https://github.com/alphavel/testing) | Testing utilities | ^1.0 | 🔄 TIER 1 |
| [alphavel/orm](https://github.com/alphavel/orm) | Advanced ORM | ^1.0 | 🔄 TIER 1 |

> **Note:** TIER 1 packages have structure + documentation. Full implementation coming soon.

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---

## 📝 License

Alphavel is open-source software licensed under the [MIT license](LICENSE).

---

## 🔗 Links

- **GitHub**: https://github.com/alphavel
- **Documentation**: https://docs.alphavel.dev
- **Community**: https://discord.gg/alphavel
- **Twitter**: @alphavel

---

## 🆘 Support

- 📖 [Documentation](https://docs.alphavel.dev)
- 💬 [Discord Community](https://discord.gg/alphavel)
- 🐛 [Issue Tracker](https://github.com/alphavel/alphavel/issues)
- 📧 [Email Support](mailto:support@alphavel.dev)

---

**Built with ❤️ by the Alphavel Team**
