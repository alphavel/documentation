# Production Deployment

Complete guide to deploying Alphavel in production environments.

---

## Quick Deployment Checklist

Before going to production:

- [ ] Enable route caching (`alpha route:cache`)
- [ ] Set `APP_ENV=production` in `.env`
- [ ] Set `APP_DEBUG=false` in `.env`
- [ ] Enable OPcache in `php.ini`
- [ ] Configure Swoole workers (2x CPU cores)
- [ ] Setup database connection pooling
- [ ] Configure Redis/Memcached for caching
- [ ] Setup logging and monitoring
- [ ] Configure SSL/TLS certificates
- [ ] Setup reverse proxy (nginx)
- [ ] Enable firewall rules
- [ ] Setup automated backups

---

## Docker Deployment (Recommended)

### Production Dockerfile

Already included in skeleton, optimized for production:

```dockerfile
FROM php:8.2-cli-alpine

# Install Swoole and dependencies
RUN apk add --no-cache ${PHPIZE_DEPS} \
    && pecl install swoole \
    && docker-php-ext-enable swoole

# Install OPcache
RUN docker-php-ext-install opcache

# Copy application
COPY . /app
WORKDIR /app

# Install Composer dependencies
RUN composer install --no-dev --optimize-autoloader --classmap-authoritative

# Cache routes
RUN alpha route:cache

# Set permissions
RUN chown -R www-data:www-data storage bootstrap/cache

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:9501/health || exit 1

# Expose port
EXPOSE 9501

# Start server
CMD ["alpha", "serve", "--workers=auto"]
```

### docker-compose.yml (Production)

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "9501:9501"
    environment:
      APP_ENV: production
      APP_DEBUG: false
      DB_HOST: db
      REDIS_HOST: redis
    depends_on:
      - db
      - redis
    restart: always
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
    volumes:
      - db_data:/var/lib/mysql
    restart: always

  redis:
    image: redis:7-alpine
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    restart: always

volumes:
  db_data:
```

### Deploy

```bash
# Build and start
docker-compose up -d --build

# Check logs
docker-compose logs -f app

# Scale horizontally
docker-compose up -d --scale app=4
```

---

## nginx Reverse Proxy

### nginx.conf

```nginx
upstream alphavel {
    least_conn;
    server app:9501 max_fails=3 fail_timeout=30s;
    # Add more app instances for load balancing
    # server app2:9501;
    # server app3:9501;
    keepalive 32;
}

server {
    listen 80;
    listen [::]:80;
    server_name api.example.com;

    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.example.com;

    # SSL certificates
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Logging
    access_log /var/log/nginx/alphavel-access.log;
    error_log /var/log/nginx/alphavel-error.log;

    # Proxy settings
    location / {
        proxy_pass http://alphavel;
        proxy_http_version 1.1;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Swoole uses keep-alive
        proxy_set_header Connection "";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffering
        proxy_buffering off;
        proxy_request_buffering off;
    }

    # Health check endpoint
    location /health {
        proxy_pass http://alphavel/health;
        access_log off;
    }
}
```

---

## Kubernetes Deployment

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alphavel-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: alphavel-api
  template:
    metadata:
      labels:
        app: alphavel-api
    spec:
      containers:
      - name: alphavel
        image: your-registry/alphavel:latest
        ports:
        - containerPort: 9501
        env:
        - name: APP_ENV
          value: "production"
        - name: APP_DEBUG
          value: "false"
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: host
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 9501
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 9501
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: alphavel-service
spec:
  selector:
    app: alphavel-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9501
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: alphavel-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: alphavel-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Deploy to Kubernetes

```bash
# Apply configuration
kubectl apply -f deployment.yaml

# Check status
kubectl get pods
kubectl get services

# View logs
kubectl logs -f deployment/alphavel-api

# Scale manually
kubectl scale deployment alphavel-api --replicas=10

# Rolling update
kubectl set image deployment/alphavel-api alphavel=your-registry/alphavel:v2
```

---

## Environment Configuration

### .env (Production)

```env
# Application
APP_NAME=Alphavel
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.example.com

# Database
DB_CONNECTION=mysql
DB_HOST=db.internal.example.com
DB_PORT=3306
DB_DATABASE=production_db
DB_USERNAME=prod_user
DB_PASSWORD=super_secure_password_here

# Connection Pool
DB_POOL_MIN=2
DB_POOL_MAX=20
DB_POOL_TIMEOUT=5.0

# Cache
CACHE_DRIVER=redis
REDIS_HOST=redis.internal.example.com
REDIS_PORT=6379
REDIS_PASSWORD=redis_secure_password
REDIS_DB=0

# Logging
LOG_CHANNEL=daily
LOG_LEVEL=warning  # Don't log debug in production

# Swoole
SWOOLE_HOST=0.0.0.0
SWOOLE_PORT=9501
SWOOLE_WORKERS=16  # 2x CPU cores
SWOOLE_MAX_REQUEST=10000
```

---

## Performance Optimization

### 1. PHP Configuration (php.ini)

```ini
[PHP]
; Memory
memory_limit = 512M

; Execution
max_execution_time = 60
max_input_time = 60

[OPcache]
; Enable OPcache
opcache.enable=1
opcache.enable_cli=1

; Memory
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000

; Performance
opcache.validate_timestamps=0  ; Production only!
opcache.revalidate_freq=0
opcache.save_comments=0
opcache.fast_shutdown=1

; JIT (PHP 8+)
opcache.jit=tracing
opcache.jit_buffer_size=128M
```

### 2. Swoole Configuration (config/swoole.php)

```php
return [
    'host' => env('SWOOLE_HOST', '0.0.0.0'),
    'port' => env('SWOOLE_PORT', 9501),
    
    // Workers (2x CPU cores)
    'worker_num' => 16,
    
    // Task workers for async operations
    'task_worker_num' => 4,
    
    // Max requests per worker before restart
    'max_request' => 10000,
    'max_conn' => 10000,
    
    // Enable coroutines
    'enable_coroutine' => true,
    'max_coroutine' => 10000,
    
    // Buffer sizes
    'package_max_length' => 10 * 1024 * 1024, // 10MB
    'buffer_output_size' => 4 * 1024 * 1024, // 4MB
    
    // Timeouts
    'heartbeat_check_interval' => 60,
    'heartbeat_idle_time' => 600,
    
    // Performance
    'enable_static_handler' => true,
    'document_root' => base_path('public'),
];
```

### 3. Cache Routes

```bash
alpha route:cache
```

---

## Monitoring & Logging

### Prometheus Metrics

Already included via raw route:

```php
$router->raw('/metrics', function($request, $response) {
    $stats = swoole_get_server_stats();
    
    $metrics = <<<METRICS
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total {$stats['request_count']}

# HELP http_connections_active Active connections
# TYPE http_connections_active gauge
http_connections_active {$stats['connection_num']}

# HELP swoole_workers Workers count
# TYPE swoole_workers gauge
swoole_workers {$stats['worker_num']}
METRICS;

    $response->header('Content-Type', 'text/plain');
    $response->end($metrics);
}, 'text/plain');
```

### Grafana Dashboard

Monitor:
- Requests per second
- Response time (p50, p95, p99)
- Error rate
- Active connections
- Memory usage
- CPU usage

### Application Logging

```php
// Structured logging
Log::info('Order created', [
    'order_id' => $orderId,
    'user_id' => $userId,
    'total' => $total,
    'timestamp' => time()
]);

// Error tracking
Log::error('Payment failed', [
    'order_id' => $orderId,
    'error' => $e->getMessage(),
    'trace' => $e->getTraceAsString()
]);
```

---

## Security

### SSL/TLS

```bash
# Let's Encrypt (free SSL)
sudo apt install certbot
sudo certbot certonly --standalone -d api.example.com

# Configure nginx (see nginx.conf above)
```

### Firewall

```bash
# UFW (Ubuntu)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 22/tcp  # SSH
sudo ufw enable

# Block direct access to Swoole port
sudo ufw deny 9501/tcp
```

### Rate Limiting

```nginx
# In nginx.conf
limit_req_zone $binary_remote_addr zone=api:10m rate=100r/s;

location /api {
    limit_req zone=api burst=200 nodelay;
    proxy_pass http://alphavel;
}
```

---

## Backup Strategy

### Database Backups

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/mysql"
DB_NAME="production_db"

# Create backup
docker-compose exec -T db mysqldump -u root -p${DB_PASSWORD} ${DB_NAME} | gzip > "${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz"

# Keep only last 7 days
find ${BACKUP_DIR} -type f -mtime +7 -delete

echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"
```

### Cron Job

```bash
# Daily backup at 2 AM
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

## Zero-Downtime Deployment

### Blue-Green Deployment

```bash
#!/bin/bash
# deploy.sh

# Build new version
docker build -t alphavel:green .

# Start green environment
docker-compose -f docker-compose.green.yml up -d

# Wait for health check
sleep 10

# Switch nginx to green
nginx -s reload

# Stop blue environment
docker-compose -f docker-compose.blue.yml down

echo "Deployment complete!"
```

### Rolling Update (Kubernetes)

```bash
# Update image
kubectl set image deployment/alphavel-api alphavel=alphavel:v2

# Monitor rollout
kubectl rollout status deployment/alphavel-api

# Rollback if needed
kubectl rollout undo deployment/alphavel-api
```

---

## Troubleshooting

### High Memory Usage

```bash
# Check worker memory
docker stats

# Reduce workers
SWOOLE_WORKERS=8 alpha serve

# Enable worker recycling
# config/swoole.php: max_request = 5000
```

### High CPU Usage

```bash
# Check worker count
# Should be 2x CPU cores, not more

# Monitor with top
docker exec app top

# Profile slow queries
# Enable slow query log in MySQL
```

### Connection Issues

```bash
# Check if server is running
curl http://localhost:9501/health

# Check Docker logs
docker-compose logs app

# Check nginx logs
tail -f /var/log/nginx/error.log
```

---

## Cost Optimization

### AWS Deployment (Example)

**Small API (< 1M req/day):**
- 1x t3.medium (2 vCPU, 4GB RAM) - $30/month
- RDS MySQL t3.micro - $15/month
- ElastiCache Redis t3.micro - $12/month
- **Total: ~$60/month**

**Medium API (< 10M req/day):**
- 2x t3.large (2 vCPU, 8GB RAM) - $120/month
- RDS MySQL t3.small - $30/month
- ElastiCache Redis t3.small - $25/month
- **Total: ~$175/month**

**High Traffic API (100M+ req/day):**
- 4x c5.2xlarge (8 vCPU, 16GB RAM) - $500/month
- RDS MySQL r5.large - $180/month
- ElastiCache Redis r5.large - $150/month
- **Total: ~$830/month**

Compare with Laravel FPM (same traffic): **$3,200+/month** 🤯

---

## Next Steps

- [Benchmarks →](../introduction/benchmarks.md)
- [Performance Guide →](../core/performance.md)
- [Monitoring →](monitoring.md)
