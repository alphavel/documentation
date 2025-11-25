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

{% raw %}
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
{% endraw %}

## Language Files

`resources/lang/en/messages.php`:
{% raw %}
```php
return [
    'welcome' => 'Welcome to our application!',
    'hello' => 'Hello, :name!',
];
```
{% endraw %}

`resources/lang/pt_BR/messages.php`:
{% raw %}
```php
return [
    'welcome' => 'Bem-vindo à nossa aplicação!',
    'hello' => 'Olá, :name!',
];
```
{% endraw %}

## Performance
- **Swoole Table cache:** < 0.001ms lookup
- **Overhead:** < 0.05%
- **Auto-cache** on first access

## License
MIT
