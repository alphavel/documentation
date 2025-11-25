---
layout: default
title: README
---

# Validation Package

Request validation with fluent API.

---

## Installation

```bash
alpha package:add validation
```

---

## Usage

```php
use Alphavel\Validation\Validator;

// In your controller
public function store(Request $request)
{
    $validator = Validator::make($request->all(), [
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
        'age' => 'required|integer|min:18',
        'password' => 'required|string|min:8|confirmed',
    ]);

    if ($validator->fails()) {
        return Response::make()->json([
            'errors' => $validator->errors()
        ], 422);
    }

    // Validation passed
    $validated = $validator->validated();
    
    // Save to database
    $userId = DB::table('users')->insert($validated);
    
    return Response::make()->json(['id' => $userId], 201);
}
```

---

## Available Rules

| Rule | Description | Example |
|------|-------------|---------|
| `required` | Field must be present | `'name' => 'required'` |
| `string` | Must be string | `'name' => 'string'` |
| `integer` | Must be integer | `'age' => 'integer'` |
| `email` | Must be valid email | `'email' => 'email'` |
| `min:X` | Minimum length/value | `'name' => 'min:3'` |
| `max:X` | Maximum length/value | `'name' => 'max:255'` |
| `confirmed` | Must match X_confirmation | `'password' => 'confirmed'` |
| `unique:table,column` | Must be unique in DB | `'email' => 'unique:users,email'` |
| `exists:table,column` | Must exist in DB | `'user_id' => 'exists:users,id'` |
| `in:val1,val2` | Must be one of values | `'role' => 'in:admin,user'` |
| `numeric` | Must be numeric | `'price' => 'numeric'` |
| `array` | Must be array | `'tags' => 'array'` |
| `boolean` | Must be boolean | `'active' => 'boolean'` |
| `date` | Must be valid date | `'birth_date' => 'date'` |
| `url` | Must be valid URL | `'website' => 'url'` |

---

## Custom Error Messages

```php
$validator = Validator::make($request->all(), [
    'name' => 'required|string|max:255',
    'email' => 'required|email',
], [
    'name.required' => 'O nome é obrigatório',
    'email.required' => 'O email é obrigatório',
    'email.email' => 'Email inválido',
]);
```

---

## Custom Rules

```php
Validator::extend('cpf', function ($attribute, $value) {
    // CPF validation logic
    return validateCPF($value);
});

// Usage
$validator = Validator::make($data, [
    'cpf' => 'required|cpf',
]);
```

---

## Next Steps

- [Available Rules →](rules.md)
- [Custom Rules →](custom-rules.md)
