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
- [Database](packages/database/README.md) - MySQL/PostgreSQL with connection pooling
- [Cache](packages/cache/README.md) - Redis/Memcached support
- [Logging](packages/logging/README.md) - Monolog integration
- [Events](packages/events/README.md) - Event dispatcher
- [Validation](packages/validation/README.md) - Request validation

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
alpha serve
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
alpha package:add database
alpha package:add cache
alpha package:add validation
```

---

## 📊 Performance

| Benchmark | Requests/sec | Latency |
|-----------|--------------|---------|
| Raw Routes | 520,000+ | 0.19ms |
| JSON API | 45,000+ | 2.2ms |
| Database Query | 20,000+ | 5ms |

**Hardware**: 4 CPU cores, 8GB RAM

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

| Package | Description | Version |
|---------|-------------|---------|
| [alphavel/alphavel](https://github.com/alphavel/alphavel) | Core framework | ^1.0 |
| [alphavel/alpha](https://github.com/alphavel/alpha) | CLI tools | ^1.0 |
| [alphavel/database](https://github.com/alphavel/database) | Database layer | ^1.0 |
| [alphavel/cache](https://github.com/alphavel/cache) | Caching | ^1.0 |
| [alphavel/logging](https://github.com/alphavel/logging) | Logging | ^1.0 |
| [alphavel/events](https://github.com/alphavel/events) | Events | ^1.0 |
| [alphavel/validation](https://github.com/alphavel/validation) | Validation | ^1.0 |
| [alphavel/skeleton](https://github.com/alphavel/skeleton) | App skeleton | ^1.0 |

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
