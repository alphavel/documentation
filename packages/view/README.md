---
layout: default
title: README
---

# Alphavel View Package

Blade-like template engine with compilation cache.

## Installation

```bash
composer require alphavel/view
```

## Usage

```php
// Render view
return view('welcome', ['name' => 'John']);

// In routes
$router->get('/', function () {
    return view('home');
});

// Controller
class HomeController extends Controller
{
    public function index()
    {
        return view('home', ['user' => Auth::user()]);
    }
}
```

## Blade Syntax

```blade
{{-- Variables --}}
{{ $name }}
{!! $html !!}

{{-- Control Structures --}}
@if ($user->isAdmin())
    <p>Admin</p>
@else
    <p>User</p>
@endif

@foreach ($users as $user)
    <li>{{ $user->name }}</li>
@endforeach

{{-- Template Inheritance --}}
@extends('layouts.app')

@section('content')
    <h1>Welcome</h1>
@endsection

{{-- Components --}}
@component('alert', ['type' => 'success'])
    Operation completed!
@endcomponent
```

## Performance
- **Compiled cache:** Views compiled once, cached forever
- **Zero overhead** after compilation
- **Auto-recompilation** in development

## License
MIT
