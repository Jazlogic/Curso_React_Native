# ⚡ Clase 5: Escalabilidad y Performance

## 📋 Objetivos de la Clase

- Implementar horizontal scaling y load balancing
- Configurar database sharding y partitioning
- Optimizar caching strategies (Redis, Memcached)
- Implementar CDN y content delivery
- Configurar performance monitoring y optimization

## 🎯 Contenido Principal

### 1. Horizontal Scaling y Load Balancing

#### Load Balancer Configuration
```javascript
// Nginx Load Balancer Configuration
const nginxConfig = `
upstream backend {
    least_conn;
    server user-service-1:3001 weight=3;
    server user-service-2:3001 weight=2;
    server user-service-3:3001 weight=1;
    
    # Health checks
    health_check interval=10s fails=3 passes=2;
}

upstream order-backend {
    ip_hash; # Sticky sessions para órdenes
    server order-service-1:3002;
    server order-service-2:3002;
    server order-service-3:3002;
}

server {
    listen 80;
    server_name api.deliveryapp.com;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req zone=api burst=20 nodelay;
    
    # User service routes
    location /api/users {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Timeouts
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 10s;
    }
    
    # Order service routes
    location /api/orders {
        proxy_pass http://order-backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
`;

// HAProxy Configuration
const haproxyConfig = `
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http_front
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    option httpchk GET /health
    
    server user-service-1 user-service-1:3001 check
    server user-service-2 user-service-2:3001 check
    server user-service-3 user-service-3:3001 check

backend order_servers
    balance source
    option httpchk GET /health
    
    server order-service-1 order-service-1:3002 check
    server order-service-2 order-service-2:3002 check
    server order-service-3 order-service-3:3002 check
`;
```

#### Application Load Balancing
```javascript
// Load Balancer en Node.js
const express = require('express');
const httpProxy = require('http-proxy-middleware');

class ApplicationLoadBalancer {
  constructor() {
    this.app = express();
    this.servers = new Map();
    this.healthChecks = new Map();
    this.algorithm = 'round-robin';
    this.currentIndex = 0;
  }

  addServer(serviceName, server) {
    if (!this.servers.has(serviceName)) {
      this.servers.set(serviceName, []);
    }
    
    this.servers.get(serviceName).push({
      ...server,
      healthy: true,
      requests: 0,
      responseTime: 0
    });
    
    // Iniciar health check
    this.startHealthCheck(serviceName, server);
  }

  startHealthCheck(serviceName, server) {
    const interval = setInterval(async () => {
      try {
        const start = Date.now();
        const response = await fetch(`http://${server.host}:${server.port}/health`);
        const responseTime = Date.now() - start;
        
        if (response.ok) {
          server.healthy = true;
          server.responseTime = responseTime;
        } else {
          server.healthy = false;
        }
      } catch (error) {
        server.healthy = false;
        console.error(`Health check failed for ${server.host}:${server.port}`);
      }
    }, 5000);
    
    this.healthChecks.set(`${serviceName}-${server.host}`, interval);
  }

  getHealthyServers(serviceName) {
    const servers = this.servers.get(serviceName) || [];
    return servers.filter(server => server.healthy);
  }

  selectServer(serviceName) {
    const healthyServers = this.getHealthyServers(serviceName);
    
    if (healthyServers.length === 0) {
      throw new Error(`No healthy servers available for ${serviceName}`);
    }

    switch (this.algorithm) {
      case 'round-robin':
        return this.roundRobin(healthyServers);
      case 'least-connections':
        return this.leastConnections(healthyServers);
      case 'weighted':
        return this.weightedRoundRobin(healthyServers);
      case 'fastest-response':
        return this.fastestResponse(healthyServers);
      default:
        return healthyServers[0];
    }
  }

  roundRobin(servers) {
    const server = servers[this.currentIndex % servers.length];
    this.currentIndex++;
    return server;
  }

  leastConnections(servers) {
    return servers.reduce((min, server) => 
      server.requests < min.requests ? server : min
    );
  }

  fastestResponse(servers) {
    return servers.reduce((fastest, server) => 
      server.responseTime < fastest.responseTime ? server : fastest
    );
  }

  setupRoutes() {
    // User service
    this.app.use('/api/users', (req, res, next) => {
      try {
        const server = this.selectServer('user-service');
        server.requests++;
        
        const proxy = httpProxy.createProxyMiddleware({
          target: `http://${server.host}:${server.port}`,
          changeOrigin: true,
          onProxyReq: (proxyReq, req, res) => {
            proxyReq.setHeader('X-Forwarded-For', req.ip);
          },
          onError: (err, req, res) => {
            console.error('Proxy error:', err);
            res.status(503).json({ error: 'Service unavailable' });
          }
        });
        
        proxy(req, res, next);
      } catch (error) {
        res.status(503).json({ error: 'No healthy servers available' });
      }
    });

    // Order service
    this.app.use('/api/orders', (req, res, next) => {
      try {
        const server = this.selectServer('order-service');
        server.requests++;
        
        const proxy = httpProxy.createProxyMiddleware({
          target: `http://${server.host}:${server.port}`,
          changeOrigin: true
        });
        
        proxy(req, res, next);
      } catch (error) {
        res.status(503).json({ error: 'No healthy servers available' });
      }
    });
  }
}
```

### 2. Database Sharding y Partitioning

#### Horizontal Sharding
```javascript
// Database Sharding Strategy
class DatabaseSharding {
  constructor() {
    this.shards = new Map();
    this.shardKey = 'userId'; // Clave para determinar shard
  }

  addShard(shardId, connection) {
    this.shards.set(shardId, connection);
  }

  getShard(shardKey) {
    const hash = this.hashFunction(shardKey);
    const shardId = hash % this.shards.size;
    return this.shards.get(shardId);
  }

  hashFunction(key) {
    let hash = 0;
    for (let i = 0; i < key.length; i++) {
      const char = key.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }

  async executeQuery(query, shardKey) {
    const shard = this.getShard(shardKey);
    return await shard.query(query);
  }

  async executeQueryAll(query) {
    const results = [];
    
    for (const [shardId, shard] of this.shards) {
      try {
        const result = await shard.query(query);
        results.push({ shardId, result });
      } catch (error) {
        console.error(`Error in shard ${shardId}:`, error);
      }
    }
    
    return results;
  }
}

// Sharding por rango de fechas
class DateRangeSharding {
  constructor() {
    this.shards = new Map();
  }

  addShard(dateRange, connection) {
    this.shards.set(dateRange, connection);
  }

  getShard(date) {
    const targetDate = new Date(date);
    
    for (const [range, connection] of this.shards) {
      const [startDate, endDate] = range;
      
      if (targetDate >= startDate && targetDate <= endDate) {
        return connection;
      }
    }
    
    throw new Error(`No shard found for date: ${date}`);
  }

  async getOrdersByDateRange(startDate, endDate) {
    const results = [];
    
    for (const [range, connection] of this.shards) {
      const [shardStart, shardEnd] = range;
      
      // Verificar si hay overlap
      if (startDate <= shardEnd && endDate >= shardStart) {
        const query = `
          SELECT * FROM orders 
          WHERE created_at >= ? AND created_at <= ?
        `;
        
        const result = await connection.query(query, [startDate, endDate]);
        results.push(...result);
      }
    }
    
    return results;
  }
}
```

#### Database Partitioning
```javascript
// PostgreSQL Partitioning
const partitioningSQL = `
-- Crear tabla particionada por fecha
CREATE TABLE orders (
    id SERIAL,
    user_id INTEGER,
    restaurant_id INTEGER,
    total DECIMAL(10,2),
    status VARCHAR(20),
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Crear particiones mensuales
CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE orders_2024_03 PARTITION OF orders
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Crear índices en particiones
CREATE INDEX idx_orders_2024_01_user_id ON orders_2024_01(user_id);
CREATE INDEX idx_orders_2024_01_status ON orders_2024_01(status);

-- Función para crear particiones automáticamente
CREATE OR REPLACE FUNCTION create_monthly_partition(table_name text, start_date date)
RETURNS void AS $$
DECLARE
    partition_name text;
    end_date date;
BEGIN
    partition_name := table_name || '_' || to_char(start_date, 'YYYY_MM');
    end_date := start_date + interval '1 month';
    
    EXECUTE format('CREATE TABLE %I PARTITION OF %I FOR VALUES FROM (%L) TO (%L)',
                   partition_name, table_name, start_date, end_date);
    
    EXECUTE format('CREATE INDEX idx_%s_user_id ON %I(user_id)', 
                   partition_name, partition_name);
    EXECUTE format('CREATE INDEX idx_%s_status ON %I(status)', 
                   partition_name, partition_name);
END;
$$ LANGUAGE plpgsql;
`;

// MongoDB Sharding
const mongoShardingConfig = `
// Configurar sharding
sh.enableSharding("deliveryapp")

// Shard collection por user_id
sh.shardCollection("deliveryapp.orders", { "userId": "hashed" })

// Shard collection por fecha
sh.shardCollection("deliveryapp.analytics", { "date": 1, "userId": 1 })

// Crear índices compuestos
db.orders.createIndex({ "userId": 1, "createdAt": -1 })
db.orders.createIndex({ "restaurantId": 1, "status": 1 })
db.orders.createIndex({ "status": 1, "createdAt": -1 })
`;
```

### 3. Caching Strategies

#### Redis Caching
```javascript
// Redis Cache Manager
class RedisCacheManager {
  constructor() {
    this.redis = require('redis').createClient({
      host: 'localhost',
      port: 6379
    });
    
    this.defaultTTL = 3600; // 1 hora
  }

  async connect() {
    await this.redis.connect();
    console.log('Connected to Redis');
  }

  // Cache con TTL
  async set(key, value, ttl = this.defaultTTL) {
    const serializedValue = JSON.stringify(value);
    await this.redis.setEx(key, ttl, serializedValue);
  }

  async get(key) {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }

  // Cache con tags para invalidación
  async setWithTags(key, value, tags, ttl = this.defaultTTL) {
    await this.set(key, value, ttl);
    
    // Agregar a sets de tags
    for (const tag of tags) {
      await this.redis.sAdd(`tag:${tag}`, key);
    }
  }

  async invalidateByTag(tag) {
    const keys = await this.redis.sMembers(`tag:${tag}`);
    
    if (keys.length > 0) {
      await this.redis.del(...keys);
      await this.redis.del(`tag:${tag}`);
    }
  }

  // Cache con patrón de actualización
  async getOrSet(key, fetchFunction, ttl = this.defaultTTL) {
    let value = await this.get(key);
    
    if (!value) {
      value = await fetchFunction();
      await this.set(key, value, ttl);
    }
    
    return value;
  }

  // Cache distribuido con locks
  async getOrSetWithLock(key, fetchFunction, ttl = this.defaultTTL) {
    let value = await this.get(key);
    
    if (!value) {
      const lockKey = `lock:${key}`;
      const lockValue = generateId();
      
      // Intentar adquirir lock
      const acquired = await this.redis.set(lockKey, lockValue, {
        EX: 10, // 10 segundos
        NX: true // Solo si no existe
      });
      
      if (acquired) {
        try {
          // Verificar nuevamente por si otro proceso ya lo creó
          value = await this.get(key);
          
          if (!value) {
            value = await fetchFunction();
            await this.set(key, value, ttl);
          }
        } finally {
          // Liberar lock
          await this.redis.del(lockKey);
        }
      } else {
        // Esperar un poco y reintentar
        await new Promise(resolve => setTimeout(resolve, 100));
        return await this.getOrSetWithLock(key, fetchFunction, ttl);
      }
    }
    
    return value;
  }
}

// Cache de aplicación
class ApplicationCache {
  constructor() {
    this.cache = new Map();
    this.maxSize = 1000;
    this.ttl = new Map(); // Time to live
  }

  set(key, value, ttlSeconds = 3600) {
    // Implementar LRU si se excede el tamaño
    if (this.cache.size >= this.maxSize) {
      this.evictLRU();
    }
    
    this.cache.set(key, value);
    this.ttl.set(key, Date.now() + (ttlSeconds * 1000));
  }

  get(key) {
    const value = this.cache.get(key);
    
    if (value && this.ttl.get(key) > Date.now()) {
      return value;
    }
    
    // Limpiar entrada expirada
    if (value) {
      this.cache.delete(key);
      this.ttl.delete(key);
    }
    
    return null;
  }

  evictLRU() {
    // Implementación simple de LRU
    const firstKey = this.cache.keys().next().value;
    this.cache.delete(firstKey);
    this.ttl.delete(firstKey);
  }

  clear() {
    this.cache.clear();
    this.ttl.clear();
  }
}
```

#### Cache Patterns
```javascript
// Cache-Aside Pattern
class CacheAsideService {
  constructor(cache, database) {
    this.cache = cache;
    this.database = database;
  }

  async getUser(userId) {
    // Intentar obtener del cache
    let user = await this.cache.get(`user:${userId}`);
    
    if (!user) {
      // Obtener de la base de datos
      user = await this.database.getUser(userId);
      
      if (user) {
        // Guardar en cache
        await this.cache.set(`user:${userId}`, user, 3600);
      }
    }
    
    return user;
  }

  async updateUser(userId, userData) {
    // Actualizar en base de datos
    const updatedUser = await this.database.updateUser(userId, userData);
    
    // Invalidar cache
    await this.cache.del(`user:${userId}`);
    
    return updatedUser;
  }
}

// Write-Through Pattern
class WriteThroughService {
  constructor(cache, database) {
    this.cache = cache;
    this.database = database;
  }

  async setUser(userId, userData) {
    // Escribir en cache y base de datos simultáneamente
    await Promise.all([
      this.cache.set(`user:${userId}`, userData, 3600),
      this.database.setUser(userId, userData)
    ]);
  }

  async getUser(userId) {
    return await this.cache.get(`user:${userId}`);
  }
}

// Write-Behind Pattern
class WriteBehindService {
  constructor(cache, database) {
    this.cache = cache;
    this.database = database;
    this.writeQueue = [];
    this.isProcessing = false;
  }

  async setUser(userId, userData) {
    // Escribir inmediatamente en cache
    await this.cache.set(`user:${userId}`, userData, 3600);
    
    // Agregar a cola de escritura
    this.writeQueue.push({ userId, userData });
    
    // Procesar cola si no está en proceso
    if (!this.isProcessing) {
      this.processWriteQueue();
    }
  }

  async processWriteQueue() {
    this.isProcessing = true;
    
    while (this.writeQueue.length > 0) {
      const { userId, userData } = this.writeQueue.shift();
      
      try {
        await this.database.setUser(userId, userData);
      } catch (error) {
        console.error('Error writing to database:', error);
        // Re-agregar a la cola para reintento
        this.writeQueue.push({ userId, userData });
      }
    }
    
    this.isProcessing = false;
  }
}
```

### 4. CDN y Content Delivery

#### CDN Configuration
```javascript
// CloudFront Configuration
const cloudFrontConfig = {
  DistributionConfig: {
    Origins: {
      Quantity: 2,
      Items: [
        {
          Id: 'user-service-origin',
          DomainName: 'user-service.deliveryapp.com',
          CustomOriginConfig: {
            HTTPPort: 80,
            HTTPSPort: 443,
            OriginProtocolPolicy: 'https-only'
          }
        },
        {
          Id: 's3-origin',
          DomainName: 'deliveryapp-assets.s3.amazonaws.com',
          S3OriginConfig: {
            OriginAccessIdentity: 'origin-access-identity/cloudfront/ABCDEFGHIJKLMN'
          }
        }
      ]
    },
    DefaultCacheBehavior: {
      TargetOriginId: 'user-service-origin',
      ViewerProtocolPolicy: 'redirect-to-https',
      CachePolicyId: '4135ea2d-6df8-44a3-9df3-4b5a84be39ad', // Managed-CachingOptimized
      Compress: true
    },
    CacheBehaviors: {
      Quantity: 2,
      Items: [
        {
          PathPattern: '/api/*',
          TargetOriginId: 'user-service-origin',
          ViewerProtocolPolicy: 'https-only',
          CachePolicyId: '4135ea2d-6df8-44a3-9df3-4b5a84be39ad',
          TTL: {
            DefaultTTL: 0,
            MaxTTL: 0,
            MinTTL: 0
          }
        },
        {
          PathPattern: '/assets/*',
          TargetOriginId: 's3-origin',
          ViewerProtocolPolicy: 'redirect-to-https',
          CachePolicyId: '658327ea-f89d-4fab-a63d-7e88639e58f6', // Managed-CachingDisabled
          TTL: {
            DefaultTTL: 86400, // 24 horas
            MaxTTL: 31536000,  // 1 año
            MinTTL: 0
          }
        }
      ]
    }
  }
};

// CDN Service
class CDNService {
  constructor() {
    this.cloudFront = new AWS.CloudFront();
    this.s3 = new AWS.S3();
  }

  async uploadAsset(file, key) {
    // Subir a S3
    const uploadParams = {
      Bucket: 'deliveryapp-assets',
      Key: key,
      Body: file,
      ContentType: file.mimetype,
      ACL: 'public-read'
    };

    await this.s3.upload(uploadParams).promise();

    // Invalidar cache de CloudFront
    await this.invalidateCache(`/assets/${key}`);

    return `https://cdn.deliveryapp.com/assets/${key}`;
  }

  async invalidateCache(paths) {
    const invalidationParams = {
      DistributionId: 'E1234567890ABC',
      InvalidationBatch: {
        Paths: {
          Quantity: paths.length,
          Items: paths
        },
        CallerReference: Date.now().toString()
      }
    };

    await this.cloudFront.createInvalidation(invalidationParams).promise();
  }

  async getSignedUrl(key, expiresIn = 3600) {
    const params = {
      Bucket: 'deliveryapp-assets',
      Key: key,
      Expires: expiresIn
    };

    return this.s3.getSignedUrl('getObject', params);
  }
}
```

#### Image Optimization
```javascript
// Image Processing Service
class ImageProcessingService {
  constructor() {
    this.sharp = require('sharp');
    this.sizes = [150, 300, 600, 1200];
    this.formats = ['webp', 'jpeg', 'png'];
  }

  async processImage(inputBuffer, options = {}) {
    const {
      width,
      height,
      quality = 80,
      format = 'webp',
      fit = 'cover'
    } = options;

    let pipeline = this.sharp(inputBuffer);

    if (width || height) {
      pipeline = pipeline.resize(width, height, { fit });
    }

    switch (format) {
      case 'webp':
        pipeline = pipeline.webp({ quality });
        break;
      case 'jpeg':
        pipeline = pipeline.jpeg({ quality });
        break;
      case 'png':
        pipeline = pipeline.png({ quality });
        break;
    }

    return await pipeline.toBuffer();
  }

  async generateResponsiveImages(inputBuffer, baseKey) {
    const results = [];

    for (const size of this.sizes) {
      for (const format of this.formats) {
        const processedImage = await this.processImage(inputBuffer, {
          width: size,
          quality: 80,
          format
        });

        const key = `${baseKey}_${size}.${format}`;
        
        // Subir a S3
        await this.uploadToS3(processedImage, key);
        
        results.push({
          size,
          format,
          key,
          url: `https://cdn.deliveryapp.com/assets/${key}`
        });
      }
    }

    return results;
  }

  async uploadToS3(buffer, key) {
    const params = {
      Bucket: 'deliveryapp-assets',
      Key: key,
      Body: buffer,
      ContentType: this.getContentType(key),
      ACL: 'public-read',
      CacheControl: 'max-age=31536000' // 1 año
    };

    await this.s3.upload(params).promise();
  }

  getContentType(key) {
    const extension = key.split('.').pop().toLowerCase();
    
    switch (extension) {
      case 'webp': return 'image/webp';
      case 'jpeg':
      case 'jpg': return 'image/jpeg';
      case 'png': return 'image/png';
      default: return 'application/octet-stream';
    }
  }
}
```

### 5. Performance Monitoring

#### Application Performance Monitoring
```javascript
// APM Service
class APMService {
  constructor() {
    this.metrics = new Map();
    this.traces = [];
    this.alerts = [];
  }

  // Métricas de performance
  recordMetric(name, value, tags = {}) {
    const timestamp = Date.now();
    const metric = {
      name,
      value,
      tags,
      timestamp
    };

    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }

    this.metrics.get(name).push(metric);

    // Verificar alertas
    this.checkAlerts(name, value, tags);
  }

  // Trazado de requests
  startTrace(operationName, tags = {}) {
    const traceId = generateId();
    const spanId = generateId();
    
    const trace = {
      traceId,
      spanId,
      operationName,
      startTime: Date.now(),
      tags,
      children: []
    };

    this.traces.push(trace);
    return trace;
  }

  finishTrace(trace, tags = {}) {
    trace.endTime = Date.now();
    trace.duration = trace.endTime - trace.startTime;
    trace.tags = { ...trace.tags, ...tags };

    // Enviar a sistema de monitoreo
    this.sendTrace(trace);
  }

  // Monitoreo de base de datos
  async monitorDatabase(query, params = []) {
    const startTime = Date.now();
    
    try {
      const result = await this.database.query(query, params);
      const duration = Date.now() - startTime;
      
      this.recordMetric('database.query.duration', duration, {
        query: query.substring(0, 50),
        success: 'true'
      });
      
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      
      this.recordMetric('database.query.duration', duration, {
        query: query.substring(0, 50),
        success: 'false',
        error: error.message
      });
      
      throw error;
    }
  }

  // Monitoreo de cache
  async monitorCache(operation, key, fn) {
    const startTime = Date.now();
    
    try {
      const result = await fn();
      const duration = Date.now() - startTime;
      
      this.recordMetric('cache.operation.duration', duration, {
        operation,
        key: key.substring(0, 20),
        hit: result !== null ? 'true' : 'false'
      });
      
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      
      this.recordMetric('cache.operation.duration', duration, {
        operation,
        key: key.substring(0, 20),
        error: error.message
      });
      
      throw error;
    }
  }

  // Alertas
  addAlert(metricName, condition, threshold, action) {
    this.alerts.push({
      metricName,
      condition,
      threshold,
      action
    });
  }

  checkAlerts(metricName, value, tags) {
    for (const alert of this.alerts) {
      if (alert.metricName === metricName) {
        let triggered = false;
        
        switch (alert.condition) {
          case 'greater_than':
            triggered = value > alert.threshold;
            break;
          case 'less_than':
            triggered = value < alert.threshold;
            break;
          case 'equals':
            triggered = value === alert.threshold;
            break;
        }
        
        if (triggered) {
          this.triggerAlert(alert, value, tags);
        }
      }
    }
  }

  triggerAlert(alert, value, tags) {
    console.log(`ALERT: ${alert.metricName} ${alert.condition} ${alert.threshold}`);
    console.log(`Current value: ${value}`, tags);
    
    // Ejecutar acción de alerta
    if (alert.action) {
      alert.action(value, tags);
    }
  }
}
```

#### Health Checks
```javascript
// Health Check Service
class HealthCheckService {
  constructor() {
    this.checks = new Map();
    this.results = new Map();
  }

  addCheck(name, checkFunction, interval = 30000) {
    this.checks.set(name, {
      function: checkFunction,
      interval,
      lastCheck: null,
      status: 'unknown'
    });

    // Ejecutar check inicial
    this.runCheck(name);

    // Programar checks periódicos
    setInterval(() => {
      this.runCheck(name);
    }, interval);
  }

  async runCheck(name) {
    const check = this.checks.get(name);
    if (!check) return;

    try {
      const startTime = Date.now();
      const result = await check.function();
      const duration = Date.now() - startTime;

      this.results.set(name, {
        status: 'healthy',
        result,
        duration,
        lastCheck: new Date().toISOString(),
        error: null
      });

      check.status = 'healthy';
    } catch (error) {
      this.results.set(name, {
        status: 'unhealthy',
        result: null,
        duration: null,
        lastCheck: new Date().toISOString(),
        error: error.message
      });

      check.status = 'unhealthy';
    }
  }

  getHealthStatus() {
    const overall = Array.from(this.results.values()).every(
      result => result.status === 'healthy'
    ) ? 'healthy' : 'unhealthy';

    return {
      status: overall,
      checks: Object.fromEntries(this.results),
      timestamp: new Date().toISOString()
    };
  }

  // Checks específicos
  async databaseHealthCheck() {
    await this.database.ping();
    return { message: 'Database connection healthy' };
  }

  async redisHealthCheck() {
    await this.redis.ping();
    return { message: 'Redis connection healthy' };
  }

  async externalAPIHealthCheck() {
    const response = await fetch('https://api.external.com/health');
    if (!response.ok) {
      throw new Error(`External API returned ${response.status}`);
    }
    return { message: 'External API healthy' };
  }
}
```

## 🛠️ Implementación Práctica

### Sistema de Performance Completo
```javascript
// Performance Service integrado
class PerformanceService {
  constructor() {
    this.apm = new APMService();
    this.healthChecks = new HealthCheckService();
    this.cache = new RedisCacheManager();
    this.cdn = new CDNService();
    
    this.setupMonitoring();
    this.setupHealthChecks();
  }

  setupMonitoring() {
    // Métricas de aplicación
    this.apm.addAlert('response.time', 'greater_than', 1000, (value) => {
      console.log(`High response time: ${value}ms`);
    });

    this.apm.addAlert('error.rate', 'greater_than', 0.05, (value) => {
      console.log(`High error rate: ${value * 100}%`);
    });

    // Middleware de Express para métricas
    this.setupExpressMiddleware();
  }

  setupExpressMiddleware() {
    return (req, res, next) => {
      const startTime = Date.now();
      const trace = this.apm.startTrace(`${req.method} ${req.path}`);

      res.on('finish', () => {
        const duration = Date.now() - startTime;
        
        this.apm.recordMetric('http.request.duration', duration, {
          method: req.method,
          path: req.path,
          status: res.statusCode
        });

        this.apm.finishTrace(trace, {
          status: res.statusCode,
          duration
        });
      });

      next();
    };
  }

  setupHealthChecks() {
    this.healthChecks.addCheck('database', this.databaseHealthCheck.bind(this));
    this.healthChecks.addCheck('redis', this.redisHealthCheck.bind(this));
    this.healthChecks.addCheck('external-api', this.externalAPIHealthCheck.bind(this));
  }
}
```

## 📊 Métricas y KPIs

### Performance KPIs
- **Response Time**: <200ms (95th percentile)
- **Throughput**: >1000 requests/second
- **Error Rate**: <0.1%
- **Availability**: 99.9%
- **Cache Hit Rate**: >90%

## 🔧 Herramientas y Tecnologías

### Load Balancing
- **Nginx**: Web server y load balancer
- **HAProxy**: High availability load balancer
- **AWS ALB**: Application Load Balancer
- **Azure Load Balancer**: Managed load balancer

### Caching
- **Redis**: In-memory data store
- **Memcached**: Distributed memory caching
- **AWS ElastiCache**: Managed caching service
- **Azure Cache for Redis**: Managed Redis service

### CDN
- **CloudFront**: AWS CDN
- **Azure CDN**: Microsoft CDN
- **Cloudflare**: Global CDN
- **Fastly**: Edge computing platform

## 🎯 Proyecto Integrador

### Sistema Escalable para Delivery App
Implementar un sistema completo con:

1. **Load Balancing** con Nginx y HAProxy
2. **Database Sharding** horizontal y vertical
3. **Caching Strategy** multi-nivel
4. **CDN Configuration** para assets
5. **Performance Monitoring** completo
6. **Health Checks** automatizados
7. **Auto-scaling** basado en métricas

## 📚 Recursos Adicionales

### Documentación
- [Nginx Load Balancing](https://nginx.org/en/docs/http/load_balancing.html)
- [Redis Caching Patterns](https://redis.io/docs/manual/patterns/)
- [AWS CloudFront](https://docs.aws.amazon.com/cloudfront/)

### Cursos Recomendados
- High Performance Web Applications
- Database Scaling Strategies
- CDN and Content Delivery
- Performance Monitoring and Optimization

## ✅ Evaluación

### Criterios de Evaluación
- Implementación correcta de load balancing
- Database sharding funcional
- Caching strategy efectiva
- CDN configurado correctamente
- Performance monitoring implementado

---

**Siguiente**: [Módulo 28: Gaming y Realidad Aumentada](../senior_19/README.md)
**Anterior**: [Clase 4: Message Queues y Event Streaming](./clase_4_message_queues_event_streaming.md)
