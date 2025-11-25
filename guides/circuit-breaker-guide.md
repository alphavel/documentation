---
layout: default
title: Circuit Breaker Guide
---

# 🛡️ Circuit Breaker Package - Guia Completo

> Resiliência para microserviços com overhead de < 0.1ms

## 🎯 O Que É?

O `alphavel/circuit-breaker` implementa o padrão Circuit Breaker para prevenir falhas em cascata em arquiteturas de microserviços, com performance de < 0.1ms de overhead por chamada.

## ⚡ Performance

- **< 0.1ms overhead** por chamada
- **O(1) lookups** (Swoole Table lock-free)
- **80 bytes memória** por serviço
- **Lock-free** shared memory
- **20x mais rápido** que implementações tradicionais

## 📦 Instalação

```bash
composer require alphavel/circuit-breaker
```

## 🚀 Quick Start

### Uso Básico

```php
use Alphavel\CircuitBreaker\Facades\CircuitBreaker;

try {
    $result = CircuitBreaker::call('payment-api', function() {
        return Http::post('https://payment-api.com/charge', $data);
    });
} catch (CircuitOpenException $e) {
    // Circuito aberto, serviço está indisponível
    return $this->handlePaymentFailure();
}
```

### Com Fallback

```php
$result = CircuitBreaker::call('payment-api', 
    function() {
        return Http::post('https://payment-api.com/charge', $data);
    },
    fallback: function() {
        // Circuito aberto, enfileirar para depois
        dispatch(new ProcessPaymentLater($data));
        return ['status' => 'queued'];
    }
);
```

## 🔄 Estados do Circuito

### 1. CLOSED (Fechado - Normal)

- ✅ Todas as requisições passam
- ✅ Falhas são contadas
- ⚠️ Abre quando threshold de falhas atingido

```php
// Circuito fechado = operação normal
$result = CircuitBreaker::call('api', fn() => Http::get('...'));
```

### 2. OPEN (Aberto - Falhando)

- ❌ Todas as requisições falham imediatamente
- ❌ Nenhuma chamada ao serviço backend
- ⏱️ Após recovery timeout → HALF_OPEN

```php
// Após 5 falhas, circuito abre
for ($i = 0; $i < 5; $i++) {
    try {
        CircuitBreaker::call('broken-service', fn() => throw new Exception());
    } catch (Exception $e) {}
}

// Agora está OPEN
CircuitBreaker::getState('broken-service'); // CircuitState::OPEN
```

### 3. HALF_OPEN (Meio Aberto - Testando)

- 🔍 Requisições limitadas permitidas
- ✅ Se taxa de sucesso >= threshold → CLOSED
- ❌ Se qualquer falha → OPEN

```php
// Após recovery timeout, próxima chamada testa recuperação
sleep(30);
CircuitBreaker::call('broken-service', fn() => Http::get('...'));
CircuitBreaker::getState('broken-service'); // CircuitState::HALF_OPEN
```

## 💡 Casos de Uso

### 1. Comunicação entre Microserviços

```php
class OrderService
{
    public function createOrder(array $data): Order
    {
        $order = Order::create($data);
        
        // Chamar serviço de inventário com circuit breaker
        try {
            $inventory = CircuitBreaker::call('inventory-service', function() use ($data) {
                return Http::post('http://inventory/reserve', [
                    'product_id' => $data['product_id'],
                    'quantity' => $data['quantity'],
                ]);
            });
            
            $order->update(['inventory_reserved' => true]);
            
        } catch (CircuitOpenException $e) {
            Log::warning('Inventory service down', [
                'service' => $e->getServiceName(),
                'metrics' => $e->getMetrics(),
            ]);
            
            // Enfileirar para processamento posterior
            dispatch(new ReserveInventoryLater($order));
        }
        
        return $order;
    }
}
```

### 2. APIs Externas

```php
class WeatherService
{
    public function getCurrentWeather(string $city): array
    {
        return CircuitBreaker::call('weather-api',
            function() use ($city) {
                $response = Http::get("https://api.weather.com/current", [
                    'city' => $city,
                    'apiKey' => config('services.weather.key'),
                ]);
                
                return $response->json();
            },
            fallback: function() use ($city) {
                // API fora, retornar dados cacheados
                return Cache::get("weather:{$city}", [
                    'temp' => null,
                    'condition' => 'Desconhecido',
                    'cached' => true,
                ]);
            }
        );
    }
}
```

### 3. Réplicas de Leitura (Database)

```php
class UserRepository
{
    public function find(int $id): ?User
    {
        // Tentar réplica de leitura com circuit breaker
        try {
            return CircuitBreaker::call('read-replica', function() use ($id) {
                return DB::connection('read')->table('users')->find($id);
            });
        } catch (CircuitOpenException $e) {
            // Réplica fora, fallback para primary
            Log::info('Read replica down, using primary');
            return DB::connection('primary')->table('users')->find($id);
        }
    }
}
```

### 4. Serviço de Email

```php
class NotificationService
{
    public function sendEmail(User $user, string $subject, string $body): bool
    {
        return CircuitBreaker::call('smtp-service',
            function() use ($user, $subject, $body) {
                Mail::to($user)->send(new GenericEmail($subject, $body));
                return true;
            },
            fallback: function() use ($user, $subject, $body) {
                // SMTP fora, enfileirar email
                dispatch(new SendEmailLater($user, $subject, $body));
                return true; // Enfileirado com sucesso
            }
        );
    }
}
```

### 5. Cache (Redis)

```php
class CacheService
{
    public function get(string $key): mixed
    {
        return CircuitBreaker::call('redis-cache',
            function() use ($key) {
                return Redis::get($key);
            },
            fallback: fn() => null // Cache miss se Redis fora
        );
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): bool
    {
        try {
            return CircuitBreaker::call('redis-cache', function() use ($key, $value, $ttl) {
                return Redis::setex($key, $ttl, $value);
            });
        } catch (CircuitOpenException $e) {
            // Redis fora, pular cache (não crítico)
            return false;
        }
    }
}
```

## ⚙️ Configuração

`config/circuit-breaker.php`:

```php
return [
    'default' => 'swoole-table',
    
    'thresholds' => [
        'failure_threshold' => 5,       // Falhas antes de abrir
        'success_threshold' => 80,      // % sucesso para fechar
        'timeout' => 60,                // Janela de reset (segundos)
        'recovery_timeout' => 30,       // Espera antes de half-open
        'half_open_requests' => 3,      // Requests de teste em half-open
    ],
    
    // Configuração por serviço
    'services' => [
        'payment-api' => [
            'failure_threshold' => 3,
            'recovery_timeout' => 60,
        ],
        
        'analytics-api' => [
            'failure_threshold' => 20,  // Mais tolerante
            'timeout' => 300,
        ],
    ],
];
```

### Perfis de Configuração

#### Agressivo (Fail Fast)

```php
'critical-api' => [
    'failure_threshold' => 2,       // Abre após 2 falhas
    'timeout' => 30,                // Janela de 30 seg
    'recovery_timeout' => 10,       // Tenta recovery em 10 seg
    'half_open_requests' => 1,      // Uma única tentativa
],
```

#### Tolerante (Retry More)

```php
'flaky-api' => [
    'failure_threshold' => 10,      // Tolera 10 falhas
    'timeout' => 120,               // Janela de 2 min
    'recovery_timeout' => 60,       // Espera 1 min
    'half_open_requests' => 5,      // Testa com 5 requests
    'success_threshold' => 60,      // Apenas 60% necessário
],
```

#### Não Crítico (Best Effort)

```php
'analytics-api' => [
    'failure_threshold' => 20,      // Muito tolerante
    'timeout' => 300,               // Janela de 5 min
    'recovery_timeout' => 120,      // Espera 2 min
],
```

## 📊 Monitoramento

### Ver Estatísticas

```bash
# Todos os serviços
php alpha circuit-breaker:stats

# Output:
# Circuit Breaker Statistics
#
# payment-api: CLOSED (failures: 0, success rate: 100%)
# inventory-service: HALF_OPEN (failures: 1, success rate: 75%)
# notification-service: OPEN (failures: 5, success rate: 45%)

# Serviço específico
php alpha circuit-breaker:stats payment-api

# Output:
# Circuit Breaker: payment-api
#
#   State: CLOSED
#   Failures: 0
#   Successes: 1500
#   Success Rate: 100%
#   Total Requests: 1500
#   Last Success: 2025-11-22 15:30:00
```

### Stats via Código

```php
// Stats de um serviço
$stats = CircuitBreaker::getStats('payment-api');

// Retorna:
[
    'service' => 'payment-api',
    'state' => 'closed',
    'failures' => 2,
    'successes' => 150,
    'success_rate' => 98.68,
    'total_requests' => 152,
    'last_failure_time' => 1732291200,
    'last_success_time' => 1732295400,
]

// Todos os serviços
$allStats = CircuitBreaker::getAllStats();

foreach ($allStats as $service => $stats) {
    echo "{$service}: {$stats['state']} ({$stats['success_rate']}%)\n";
}
```

## 🔧 Controle Manual

### Abrir Circuito Manualmente

```php
// Forçar abertura (ex: durante manutenção)
CircuitBreaker::breaker()->open('payment-api');
```

### Fechar Circuito Manualmente

```php
// Forçar fechamento (ex: após correção manual)
CircuitBreaker::breaker()->close('payment-api');
```

### Reset

```php
// Reset para CLOSED e limpar contadores
CircuitBreaker::breaker()->reset('payment-api');
```

### Via CLI

```bash
php alpha circuit-breaker:reset payment-api

# Output: Circuit breaker for 'payment-api' has been reset.
```

## 📈 Health Check

### Endpoint de Saúde

```php
// routes/api.php
Route::get('/health/circuit-breakers', function() {
    $stats = CircuitBreaker::getAllStats();
    
    $healthy = true;
    $issues = [];
    
    foreach ($stats as $service => $stat) {
        if ($stat['state'] === 'open') {
            $healthy = false;
            $issues[] = $service;
        }
    }
    
    return response()->json([
        'healthy' => $healthy,
        'issues' => $issues,
        'services' => $stats,
    ], $healthy ? 200 : 503);
});
```

### Alertas

```php
// Monitorar e alertar
$stats = CircuitBreaker::getAllStats();

foreach ($stats as $service => $stat) {
    if ($stat['state'] === 'open') {
        // Alertar equipe de operações
        Notification::route('slack', config('slack.ops_channel'))
            ->notify(new CircuitOpenAlert($service, $stat));
    }
}
```

## 🚀 Best Practices

### 1. Sempre Use Fallbacks

```php
// ❌ Ruim: Sem fallback
try {
    $result = CircuitBreaker::call('api', fn() => Http::get('...'));
} catch (CircuitOpenException $e) {
    throw $e; // Usuário vê erro
}

// ✅ Bom: Degradação graciosa
$result = CircuitBreaker::call('api',
    fn() => Http::get('...'),
    fallback: fn() => Cache::get('api:last_known_good')
);
```

### 2. Configure Por Serviço

```php
// Payment: Crítico, fail fast
'payment-api' => [
    'failure_threshold' => 3,
    'recovery_timeout' => 60,
],

// Analytics: Não crítico, tolerante
'analytics-api' => [
    'failure_threshold' => 20,
    'recovery_timeout' => 30,
],
```

### 3. Monitore Estados

```php
// Log quando circuito abre
try {
    $result = CircuitBreaker::call('api', fn() => Http::get('...'));
} catch (CircuitOpenException $e) {
    Log::warning('Circuit breaker open', [
        'service' => $e->getServiceName(),
        'metrics' => $e->getMetrics(),
    ]);
}
```

### 4. Teste Comportamento

```php
// Testar que circuito abre
for ($i = 0; $i < 5; $i++) {
    try {
        CircuitBreaker::call('test-service', fn() => throw new Exception());
    } catch (Exception $e) {}
}

$this->assertEquals('open', CircuitBreaker::getState('test-service'));
```

## ❓ FAQ

**Q: Quando usar circuit breakers?**  
A: Para qualquer dependência remota: APIs externas, microserviços, databases, cache, email, SMS.

**Q: Qual o impacto na performance?**  
A: < 0.1ms de overhead. Negligível para maioria dos casos.

**Q: Posso usar com código existente?**  
A: Sim! Basta envolver chamadas existentes.

**Q: E se o circuito abrir em produção?**  
A: Use fallbacks para degradação graciosa.

**Q: Circuitos se recuperam automaticamente?**  
A: Sim! Após recovery_timeout, move para HALF_OPEN e testa.

**Q: Compatível com Laravel?**  
A: Sim! Funciona perfeitamente com HTTP client, filas, cache, etc.

## 🔗 Links

- [Repositório GitHub](https://github.com/alphavel/circuit-breaker)
- [README Completo](https://github.com/alphavel/circuit-breaker/blob/master/README.md)
- [Circuit Breaker Pattern](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Documentação Alphavel](https://alphavel.com/docs)

---

**Performance: < 0.1ms overhead | Resiliência: Production-ready | Pattern: Battle-tested**
