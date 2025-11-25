---
layout: default
title: README
---

# Alphavel i18n Package

Internationalization with Swoole Table caching.

## Installation

```bash
composer require alphavel/i18n
```

## Usage

```php
// Translate
echo __('messages.welcome');
echo trans('messages.hello', ['name' => 'John']);

// Switch locale
app()->setLocale('pt_BR');
$locale = app()->getLocale();

// In views
{{ __('auth.failed') }}
```

## Language Files

`resources/lang/en/messages.php`:
```php
return [
    'welcome' => 'Welcome to our application!',
    'hello' => 'Hello, :name!',
];
```

`resources/lang/pt_BR/messages.php`:
```php
return [
    'welcome' => 'Bem-vindo à nossa aplicação!',
    'hello' => 'Olá, :name!',
];
```

## Performance
- **Swoole Table cache:** < 0.001ms lookup
- **Overhead:** < 0.05%
- **Auto-cache** on first access

## License
MIT
