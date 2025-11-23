# Alphavel Session Package

Session management with multiple drivers optimized for Swoole.

## Installation

```bash
composer require alphavel/session
```

## Usage

```php
// Get
$value = session('key');
$value = session()->get('key', 'default');

// Set
session(['key' => 'value']);
session()->put('key', 'value');

// Has
if (session()->has('key')) {
    //
}

// Delete
session()->forget('key');

// Flash (next request only)
session()->flash('message', 'Success!');
```

## Drivers
- File (default)
- Redis
- Database
- Swoole Table (ultra-fast, in-memory)

## Performance
- **Swoole Table driver:** < 0.001ms access time
- **Overhead:** < 0.1%

## License
MIT
