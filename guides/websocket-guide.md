---
layout: default
title: Websocket Guide
---

# 📡 WebSocket Package - Guia Completo

> Real-time WebSocket com 500k+ mensagens/segundo e 100k+ conexões simultâneas

## 🎯 O Que É?

O `alphavel/websocket` é um package de WebSocket de alta performance construído sobre o Swoole, oferecendo comunicação bidirecional em tempo real com API familiar ao Laravel.

## ⚡ Performance

- **500,000+ mensagens/segundo**
- **100,000+ conexões simultâneas**
- **< 1ms de latência**
- **4KB memória por conexão**
- **30x mais rápido** que Laravel Echo + Pusher

## 📦 Instalação

```bash
composer require alphavel/websocket
```

## 🚀 Quick Start

### 1. Iniciar Servidor

```bash
php alpha websocket:serve
```

### 2. Conectar Cliente (JavaScript)

```javascript
const ws = new WebSocket('ws://localhost:9501?token=YOUR_JWT_TOKEN');

ws.onopen = () => {
    // Subscrever a um canal
    ws.send(JSON.stringify({
        event: 'subscribe',
        data: { channel: 'chat.room.1' }
    }));
};

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Mensagem recebida:', data);
};
```

### 3. Broadcast do Servidor (PHP)

```php
use Alphavel\WebSocket\Facades\WebSocket;

// Broadcast para canal
WebSocket::toChannel('chat.room.1')->push([
    'event' => 'new-message',
    'data' => [
        'user' => 'João',
        'message' => 'Olá a todos!',
        'timestamp' => time()
    ]
]);

// Broadcast para usuário específico
WebSocket::toUser(123)->push([
    'event' => 'notification',
    'data' => ['title' => 'Novo pedido', 'body' => 'Pedido #1234']
]);
```

## 💡 Casos de Uso

### Chat em Tempo Real

```php
// Controller
public function sendMessage(Request $request)
{
    $message = Message::create([
        'user_id' => $request->user()->id,
        'room_id' => $request->room_id,
        'text' => $request->text,
    ]);
    
    WebSocket::toChannel("chat.room.{$request->room_id}")
        ->push([
            'event' => 'new-message',
            'data' => [
                'id' => $message->id,
                'user' => $request->user()->name,
                'text' => $message->text,
                'timestamp' => $message->created_at->timestamp,
            ]
        ]);
    
    return response()->json($message);
}
```

### Dashboard ao Vivo

```php
// Atualizar métricas a cada segundo
$server = app('websocket.server');

$server->on('workerStart', function() {
    \Swoole\Timer::tick(1000, function() {
        $metrics = [
            'vendas' => Sale::today()->sum('amount'),
            'pedidos' => Order::today()->count(),
            'usuarios_online' => WebSocket::getPresence('presence-dashboard')->count(),
        ];
        
        WebSocket::toChannel('dashboard')->push([
            'event' => 'metrics-update',
            'data' => $metrics
        ]);
    });
});
```

### Presence Channels (Quem Está Online)

```php
// Obter usuários online
$online = WebSocket::getPresence('presence-chat.room.1');
// Retorna: [
//     ['id' => 123, 'connected_at' => 1732291200],
//     ['id' => 456, 'connected_at' => 1732291210],
// ]

// Stats do canal
$stats = WebSocket::getChannelStats('presence-chat.room.1');
// Retorna: [
//     'name' => 'presence-chat.room.1',
//     'subscribers' => 42,
//     'type' => 'presence'
// ]
```

## 🔒 Tipos de Canais

### Canais Públicos
Qualquer um pode subscrever:
```javascript
ws.send(JSON.stringify({
    event: 'subscribe',
    data: { channel: 'public-updates' }
}));
```

### Canais Privados
Requerem autorização:
```javascript
ws.send(JSON.stringify({
    event: 'subscribe',
    data: { channel: 'private-user.123' }
}));
```

### Canais de Presença
Rastreiam quem está online:
```javascript
ws.send(JSON.stringify({
    event: 'subscribe',
    data: { channel: 'presence-chat.room.1' }
}));

// Eventos automáticos:
// - member_added: Quando alguém entra
// - member_removed: Quando alguém sai
```

## ⚙️ Configuração

`config/websocket.php`:

```php
return [
    'host' => env('WEBSOCKET_HOST', '0.0.0.0'),
    'port' => env('WEBSOCKET_PORT', 9501),
    
    'options' => [
        'worker_num' => swoole_cpu_num(), // Workers = núcleos CPU
        'max_connection' => 100000,        // Máx conexões
        'heartbeat_check_interval' => 60,  // Keep-alive
    ],
    
    'auth' => [
        'enabled' => true,
        'guard' => 'api',
    ],
];
```

## 📊 Monitoramento

### Ver Estatísticas

```bash
php alpha websocket:stats
```

**Output:**
```
WebSocket Server Statistics

Total Connections: 1,234
Total Channels: 42

Active Channels:
  chat.room.1 (public): 156 subscribers
  presence-dashboard (presence): 12 subscribers
  private-user.123 (private): 1 subscribers
```

### Estatísticas via Código

```php
// Total de conexões
$count = app('websocket.server')->connections()->count();

// Lista de canais
$channels = WebSocket::getChannels();

// Stats de canal específico
$stats = WebSocket::getChannelStats('chat.room.1');
```

## 🚀 Deploy em Produção

### Supervisor

`/etc/supervisor/conf.d/websocket.conf`:

```ini
[program:alphavel-websocket]
command=php /var/www/html/alpha websocket:serve
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/websocket.log
```

### Docker

```dockerfile
FROM php:8.2-cli

RUN pecl install swoole
RUN docker-php-ext-enable swoole

COPY . /app
WORKDIR /app

RUN composer install --no-dev

EXPOSE 9501

CMD ["php", "alpha", "websocket:serve"]
```

### Nginx (Proxy Reverso)

```nginx
upstream websocket {
    server 127.0.0.1:9501;
}

server {
    listen 80;
    server_name ws.example.com;
    
    location / {
        proxy_pass http://websocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 3600s;
    }
}
```

## 🎨 Integração com Frontend

### React

```jsx
import { useEffect, useState } from 'react';

function Chat() {
    const [ws, setWs] = useState(null);
    const [messages, setMessages] = useState([]);
    
    useEffect(() => {
        const socket = new WebSocket('ws://localhost:9501?token=' + token);
        
        socket.onopen = () => {
            socket.send(JSON.stringify({
                event: 'subscribe',
                data: { channel: 'chat.room.1' }
            }));
        };
        
        socket.onmessage = (event) => {
            const data = JSON.parse(event.data);
            if (data.event === 'new-message') {
                setMessages(prev => [...prev, data.data]);
            }
        };
        
        setWs(socket);
        return () => socket.close();
    }, []);
    
    return (
        <div>
            {messages.map(msg => (
                <div key={msg.id}>
                    <strong>{msg.user}:</strong> {msg.text}
                </div>
            ))}
        </div>
    );
}
```

### Vue.js

```vue
<template>
    <div>
        <div v-for="msg in messages" :key="msg.id">
            <strong>{{ msg.user }}:</strong> {{ msg.text }}
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            ws: null,
            messages: []
        }
    },
    mounted() {
        this.ws = new WebSocket('ws://localhost:9501?token=' + this.token);
        
        this.ws.onopen = () => {
            this.ws.send(JSON.stringify({
                event: 'subscribe',
                data: { channel: 'chat.room.1' }
            }));
        };
        
        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            if (data.event === 'new-message') {
                this.messages.push(data.data);
            }
        };
    },
    beforeUnmount() {
        this.ws?.close();
    }
}
</script>
```

## 🔧 Troubleshooting

### Porta já em uso

```bash
# Ver o que está usando a porta 9501
sudo lsof -i :9501

# Matar processo
sudo kill -9 PID
```

### Conexões não persistem

Verifique configuração de heartbeat:

```php
'options' => [
    'heartbeat_check_interval' => 60,
    'heartbeat_idle_time' => 600,
],
```

### Autenticação falhando

```php
// Desabilitar auth para testes
'auth' => [
    'enabled' => false,
],
```

## ❓ FAQ

**Q: Quantas conexões suporta?**  
A: 100k+ em um único servidor (16GB RAM, 8 cores).

**Q: Funciona com Laravel Echo?**  
A: Parcialmente. Recomendamos usar nosso client JavaScript ou adaptar protocolo.

**Q: Como escalar para milhões?**  
A: Use Redis broadcasting + múltiplos servidores WebSocket atrás de load balancer.

**Q: Precisa de autenticação?**  
A: Não. Configure `'auth' => ['enabled' => false]` se não precisar.

**Q: Funciona com React/Vue/Angular?**  
A: Sim! API WebSocket padrão, funciona com qualquer framework.

## 🔗 Links

- [Repositório GitHub](https://github.com/alphavel/websocket)
- [README Completo](https://github.com/alphavel/websocket/blob/master/README.html)
- [Documentação Alphavel](https://alphavel.com/docs)

---

**Performance: 500k+ msgs/s | Conexões: 100k+ | Latência: < 1ms**
