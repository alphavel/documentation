---
layout: default
title: README
---

# Alphavel Testing Package

PHPUnit helpers and testing utilities for Alphavel.

## Installation

```bash
composer require alphavel/testing --dev
```

## Usage

{% raw %}
```php
use Alphavel\Testing\TestCase;

class UserTest extends TestCase
{
    public function test_user_can_register()
    {
        $response = $this->post('/api/register', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password',
        ]);
        
        $response->assertStatus(201)
                 ->assertJson(['message' => 'User created']);
                 
        $this->assertDatabaseHas('users', [
            'email' => 'john@example.com',
        ]);
    }
    
    public function test_authenticated_user_can_update_profile()
    {
        $user = $this->actingAs(User::factory()->create());
        
        $response = $this->put('/api/profile', [
            'name' => 'Jane Doe',
        ]);
        
        $response->assertStatus(200);
    }
}
```
{% endraw %}

## Features
- HTTP assertions (`assertStatus`, `assertJson`)
- Database assertions (`assertDatabaseHas`)
- Authentication helpers (`actingAs`)
- Factory support
- Faker integration

## License
MIT
