# Logging Package

Flexible logging with Monolog integration.

---

## Installation

```bash
alpha package:add logging
```

---

## Configuration

Edit `config/logging.php`:

```php
return [
    'default' => env('LOG_CHANNEL', 'daily'),

    'channels' => [
        'daily' => [
            'driver' => 'daily',
            'path' => storage_path('logs/alphavel.log'),
            'level' => env('LOG_LEVEL', 'debug'),
            'days' => 14,
        ],

        'single' => [
            'driver' => 'single',
            'path' => storage_path('logs/alphavel.log'),
            'level' => 'debug',
        ],
    ],
];
```

---

## Usage

```php
use Alphavel\Logging\Log;

// Log levels
Log::emergency('System is down!');
Log::alert('Action must be taken immediately');
Log::critical('Critical condition');
Log::error('Error occurred', ['user_id' => 123]);
Log::warning('Warning message');
Log::notice('Normal but significant');
Log::info('Informational message');
Log::debug('Debug information');

// With context
Log::error('User not found', [
    'user_id' => 123,
    'ip' => '192.168.1.1',
    'timestamp' => time()
]);

// Use specific channel
Log::channel('single')->info('Message to single file');
```

---

## Next Steps

- [Usage Guide →](usage.md)
- [Channels →](channels.md)
