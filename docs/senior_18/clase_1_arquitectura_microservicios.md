# 🏗️ Clase 1: Arquitectura de Microservicios

## 📋 Objetivos de la Clase

- Comprender los fundamentos de microservicios
- Diseñar arquitecturas distribuidas escalables
- Implementar service discovery y load balancing
- Configurar API Gateway y routing
- Implementar circuit breakers y resiliencia

## 🎯 Contenido Principal

### 1. Fundamentos de Microservicios

#### ¿Qué son los Microservicios?
```javascript
// Arquitectura monolítica vs microservicios
// Monolítico: Una aplicación grande
const monolithicApp = {
  users: require('./modules/users'),
  orders: require('./modules/orders'),
  payments: require('./modules/payments'),
  notifications: require('./modules/notifications')
};

// Microservicios: Servicios independientes
const userService = {
  port: 3001,
  database: 'users_db',
  endpoints: ['/users', '/auth', '/profile']
};

const orderService = {
  port: 3002,
  database: 'orders_db',
  endpoints: ['/orders', '/cart', '/checkout']
};
```

#### Principios de Microservicios
```javascript
// 1. Single Responsibility Principle
class UserService {
  constructor() {
    this.responsibilities = [
      'User registration',
      'Authentication',
      'Profile management',
      'User preferences'
    ];
  }
}

// 2. Database per Service
class ServiceDatabase {
  constructor(serviceName) {
    this.serviceName = serviceName;
    this.database = `${serviceName}_db`;
    this.isolation = 'Complete';
  }
}

// 3. Decentralized Governance
class ServiceGovernance {
  constructor() {
    this.technologies = {
      userService: 'Node.js + Express',
      orderService: 'Python + FastAPI',
      paymentService: 'Java + Spring Boot',
      notificationService: 'Go + Gin'
    };
  }
}
```

### 2. Diseño de Arquitectura Distribuida

#### Patrón de Comunicación
```javascript
// Comunicación síncrona (HTTP/REST)
class SynchronousCommunication {
  async getUserOrders(userId) {
    try {
      // Llamada directa al servicio de órdenes
      const response = await fetch(`http://order-service:3002/users/${userId}/orders`);
      return await response.json();
    } catch (error) {
      console.error('Error calling order service:', error);
      throw error;
    }
  }
}

// Comunicación asíncrona (Message Queues)
class AsynchronousCommunication {
  constructor() {
    this.messageQueue = new MessageQueue();
  }

  async publishUserCreated(userData) {
    await this.messageQueue.publish('user.created', {
      userId: userData.id,
      email: userData.email,
      timestamp: new Date().toISOString()
    });
  }

  async subscribeToEvents() {
    this.messageQueue.subscribe('order.created', this.handleOrderCreated.bind(this));
    this.messageQueue.subscribe('payment.processed', this.handlePaymentProcessed.bind(this));
  }
}
```

#### Service Mesh
```javascript
// Configuración de Service Mesh con Istio
const serviceMeshConfig = {
  apiVersion: 'networking.istio.io/v1alpha3',
  kind: 'VirtualService',
  metadata: {
    name: 'user-service'
  },
  spec: {
    hosts: ['user-service'],
    http: [
      {
        route: [
          {
            destination: {
              host: 'user-service',
              subset: 'v1'
            },
            weight: 90
          },
          {
            destination: {
              host: 'user-service',
              subset: 'v2'
            },
            weight: 10
          }
        ]
      }
    ]
  }
};
```

### 3. Service Discovery

#### Consul Service Discovery
```javascript
// Configuración de Consul
const consul = require('consul')();

class ServiceDiscovery {
  constructor() {
    this.consul = consul;
  }

  // Registrar servicio
  async registerService(serviceConfig) {
    const service = {
      id: serviceConfig.id,
      name: serviceConfig.name,
      address: serviceConfig.address,
      port: serviceConfig.port,
      check: {
        http: `http://${serviceConfig.address}:${serviceConfig.port}/health`,
        interval: '10s'
      }
    };

    await this.consul.agent.service.register(service);
    console.log(`Service ${serviceConfig.name} registered`);
  }

  // Descubrir servicios
  async discoverService(serviceName) {
    const services = await this.consul.health.service({
      service: serviceName,
      passing: true
    });

    return services[0].Service;
  }

  // Load balancing round-robin
  async getServiceInstance(serviceName) {
    const services = await this.discoverService(serviceName);
    const instances = services.map(s => s.Service);
    
    // Round-robin selection
    const index = Math.floor(Math.random() * instances.length);
    return instances[index];
  }
}
```

#### Eureka Service Registry
```javascript
// Configuración de Eureka
class EurekaClient {
  constructor() {
    this.eureka = require('eureka-js-client').Eureka;
    this.client = new this.eureka({
      instance: {
        app: 'user-service',
        hostName: 'localhost',
        ipAddr: '127.0.0.1',
        port: {
          '$': 3001,
          '@enabled': 'true'
        },
        vipAddress: 'user-service',
        dataCenterInfo: {
          '@class': 'com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo',
          name: 'MyOwn'
        }
      },
      eureka: {
        host: 'localhost',
        port: 8761,
        servicePath: '/eureka/apps/'
      }
    });
  }

  start() {
    this.client.start();
  }

  stop() {
    this.client.stop();
  }
}
```

### 4. API Gateway

#### Kong API Gateway
```javascript
// Configuración de Kong
const kongConfig = {
  services: [
    {
      name: 'user-service',
      url: 'http://user-service:3001'
    },
    {
      name: 'order-service',
      url: 'http://order-service:3002'
    }
  ],
  routes: [
    {
      name: 'user-routes',
      service: 'user-service',
      paths: ['/api/users']
    },
    {
      name: 'order-routes',
      service: 'order-service',
      paths: ['/api/orders']
    }
  ],
  plugins: [
    {
      name: 'rate-limiting',
      config: {
        minute: 100,
        hour: 1000
      }
    },
    {
      name: 'cors',
      config: {
        origins: ['*'],
        methods: ['GET', 'POST', 'PUT', 'DELETE']
      }
    }
  ]
};
```

#### API Gateway con Express
```javascript
// API Gateway personalizado
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

class APIGateway {
  constructor() {
    this.app = express();
    this.services = new Map();
    this.setupMiddleware();
    this.setupRoutes();
  }

  setupMiddleware() {
    // Rate limiting
    this.app.use('/api', this.rateLimiter());
    
    // Authentication
    this.app.use('/api', this.authenticate());
    
    // Logging
    this.app.use('/api', this.logger());
  }

  setupRoutes() {
    // User service routes
    this.app.use('/api/users', createProxyMiddleware({
      target: 'http://user-service:3001',
      changeOrigin: true,
      pathRewrite: {
        '^/api/users': '/users'
      }
    }));

    // Order service routes
    this.app.use('/api/orders', createProxyMiddleware({
      target: 'http://order-service:3002',
      changeOrigin: true,
      pathRewrite: {
        '^/api/orders': '/orders'
      }
    }));
  }

  rateLimiter() {
    return (req, res, next) => {
      const clientId = req.ip;
      const requests = this.getRequestCount(clientId);
      
      if (requests > 100) {
        return res.status(429).json({ error: 'Rate limit exceeded' });
      }
      
      this.incrementRequestCount(clientId);
      next();
    };
  }
}
```

### 5. Circuit Breakers y Resiliencia

#### Hystrix Circuit Breaker
```javascript
// Implementación de Circuit Breaker
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.timeout = options.timeout || 60000;
    this.resetTimeout = options.resetTimeout || 30000;
    
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.nextAttempt = Date.now();
  }

  async execute(operation) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.resetTimeout;
    }
  }
}

// Uso del Circuit Breaker
const userServiceBreaker = new CircuitBreaker({
  failureThreshold: 3,
  resetTimeout: 30000
});

async function getUserData(userId) {
  return await userServiceBreaker.execute(async () => {
    const response = await fetch(`http://user-service:3001/users/${userId}`);
    if (!response.ok) {
      throw new Error(`User service error: ${response.status}`);
    }
    return await response.json();
  });
}
```

#### Bulkhead Pattern
```javascript
// Implementación de Bulkhead Pattern
class BulkheadExecutor {
  constructor(pools = {}) {
    this.pools = {
      userService: new WorkerPool(5),
      orderService: new WorkerPool(3),
      paymentService: new WorkerPool(2),
      ...pools
    };
  }

  async execute(serviceName, operation) {
    const pool = this.pools[serviceName];
    if (!pool) {
      throw new Error(`No pool found for service: ${serviceName}`);
    }

    return await pool.execute(operation);
  }
}

class WorkerPool {
  constructor(size) {
    this.size = size;
    this.workers = [];
    this.queue = [];
    
    for (let i = 0; i < size; i++) {
      this.workers.push(new Worker());
    }
  }

  async execute(operation) {
    return new Promise((resolve, reject) => {
      this.queue.push({ operation, resolve, reject });
      this.processQueue();
    });
  }

  processQueue() {
    if (this.queue.length === 0) return;
    
    const availableWorker = this.workers.find(w => !w.busy);
    if (!availableWorker) return;
    
    const { operation, resolve, reject } = this.queue.shift();
    availableWorker.execute(operation)
      .then(resolve)
      .catch(reject)
      .finally(() => {
        this.processQueue();
      });
  }
}
```

## 🛠️ Implementación Práctica

### Sistema de Microservicios Completo
```javascript
// Configuración principal del sistema
class MicroservicesSystem {
  constructor() {
    this.services = new Map();
    this.gateway = new APIGateway();
    this.serviceDiscovery = new ServiceDiscovery();
    this.circuitBreakers = new Map();
  }

  async initialize() {
    // Registrar servicios
    await this.registerServices();
    
    // Configurar circuit breakers
    this.setupCircuitBreakers();
    
    // Iniciar API Gateway
    await this.gateway.start();
    
    console.log('Microservices system initialized');
  }

  async registerServices() {
    const services = [
      { name: 'user-service', port: 3001 },
      { name: 'order-service', port: 3002 },
      { name: 'payment-service', port: 3003 },
      { name: 'notification-service', port: 3004 }
    ];

    for (const service of services) {
      await this.serviceDiscovery.registerService(service);
      this.services.set(service.name, service);
    }
  }

  setupCircuitBreakers() {
    for (const [serviceName] of this.services) {
      this.circuitBreakers.set(serviceName, new CircuitBreaker({
        failureThreshold: 5,
        resetTimeout: 30000
      }));
    }
  }
}
```

## 📊 Métricas y Monitoreo

### Health Checks
```javascript
// Health check para cada servicio
class HealthCheck {
  constructor(serviceName) {
    this.serviceName = serviceName;
    this.checks = new Map();
  }

  addCheck(name, checkFunction) {
    this.checks.set(name, checkFunction);
  }

  async getHealth() {
    const results = {};
    let overall = 'healthy';

    for (const [name, check] of this.checks) {
      try {
        const result = await check();
        results[name] = { status: 'healthy', result };
      } catch (error) {
        results[name] = { status: 'unhealthy', error: error.message };
        overall = 'unhealthy';
      }
    }

    return {
      service: this.serviceName,
      status: overall,
      checks: results,
      timestamp: new Date().toISOString()
    };
  }
}

// Uso de health checks
const userServiceHealth = new HealthCheck('user-service');
userServiceHealth.addCheck('database', async () => {
  // Verificar conexión a base de datos
  return await database.ping();
});

userServiceHealth.addCheck('external-api', async () => {
  // Verificar API externa
  const response = await fetch('https://api.external.com/health');
  return response.ok;
});
```

## 🔧 Herramientas y Tecnologías

### Service Mesh
- **Istio**: Service mesh completo
- **Linkerd**: Service mesh ligero
- **Consul Connect**: Service mesh de HashiCorp

### API Gateway
- **Kong**: API Gateway open source
- **AWS API Gateway**: Gateway gestionado
- **Azure API Management**: Gateway de Microsoft

### Service Discovery
- **Consul**: Service discovery de HashiCorp
- **Eureka**: Service registry de Netflix
- **etcd**: Distributed key-value store

## 🎯 Proyecto Integrador

### Sistema de Delivery con Microservicios
Implementar un sistema completo con:

1. **User Service**: Autenticación y perfiles
2. **Restaurant Service**: Catálogos y menús
3. **Order Service**: Gestión de pedidos
4. **Payment Service**: Procesamiento de pagos
5. **Notification Service**: Push notifications
6. **API Gateway**: Routing y rate limiting
7. **Service Discovery**: Registro de servicios
8. **Circuit Breakers**: Resiliencia y fallbacks

## 📚 Recursos Adicionales

### Documentación
- [Microservices.io](https://microservices.io/)
- [Istio Documentation](https://istio.io/docs/)
- [Kong Documentation](https://docs.konghq.com/)

### Cursos Recomendados
- Microservices Architecture
- Service Mesh Fundamentals
- API Gateway Patterns
- Distributed Systems Design

## ✅ Evaluación

### Criterios de Evaluación
- Diseño correcto de arquitectura de microservicios
- Implementación de service discovery
- Configuración de API Gateway
- Implementación de circuit breakers
- Sistema resiliente y escalable

---

**Siguiente**: [Clase 2: GraphQL y APIs Modernas](./clase_2_graphql_apis_modernas.md)
**Anterior**: [Módulo 26: Analytics Avanzados](../senior_17/README.md)
