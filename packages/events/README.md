---
layout: default
title: README
---

# Events Package

Event dispatcher for decoupled application logic.

---

## Installation

```bash
alpha package:add events
```

---

## Configuration

Edit `config/events.php`:

{% raw %}
```php
return [
    'listeners' => [
        'user.created' => [
            App\Listeners\SendWelcomeEmail::class,
            App\Listeners\CreateUserProfile::class,
        ],
        'user.deleted' => [
            App\Listeners\DeleteUserData::class,
        ],
    ],
];
```
{% endraw %}

---

## Usage

### Dispatching Events

{% raw %}
```php
use Alphavel\Events\Event;

// Dispatch simple event
Event::dispatch('user.created', ['user_id' => 123]);

// Dispatch with data
Event::dispatch('order.placed', [
    'order_id' => 456,
    'total' => 99.99,
    'user_id' => 123
]);
```
{% endraw %}

### Creating Listeners

{% raw %}
```php
<?php

namespace App\Listeners;

class SendWelcomeEmail
{
    public function handle(array $data)
    {
        $userId = $data['user_id'];
        
        // Send email logic
        Mail::send($userId, 'welcome');
    }
}
```
{% endraw %}

### Registering Listeners

In `config/events.php`:

{% raw %}
```php
'listeners' => [
    'user.created' => [
        App\Listeners\SendWelcomeEmail::class,
        App\Listeners\CreateUserProfile::class,
    ],
],
```
{% endraw %}

---

## Example

{% raw %}
```php
// In UserController
public function store(Request $request)
{
    $userId = DB::table('users')->insert($request->all());
    
    // Dispatch event
    Event::dispatch('user.created', ['user_id' => $userId]);
    
    return Response::make()->json(['id' => $userId], 201);
}

// Listeners automatically called
// - SendWelcomeEmail
// - CreateUserProfile
```
{% endraw %}

---

## Next Steps

- [Dispatching Events →](dispatching.html)
- [Event Listeners →](listeners.html)
