# 📡 Clase 4: Message Queues y Event Streaming

## 📋 Objetivos de la Clase

- Implementar message queues (RabbitMQ, Redis)
- Configurar event streaming (Kafka, EventStore)
- Desarrollar event sourcing y CQRS
- Crear procesamiento asíncrono
- Diseñar arquitectura event-driven

## 🎯 Contenido Principal

### 1. Message Queues

#### RabbitMQ Implementation
```javascript
// Configuración de RabbitMQ
const amqp = require('amqplib');

class RabbitMQService {
  constructor() {
    this.connection = null;
    this.channel = null;
    this.exchanges = new Map();
  }

  async connect() {
    try {
      this.connection = await amqp.connect('amqp://localhost');
      this.channel = await this.connection.createChannel();
      
      // Configurar exchanges
      await this.setupExchanges();
      
      console.log('Connected to RabbitMQ');
    } catch (error) {
      console.error('Failed to connect to RabbitMQ:', error);
      throw error;
    }
  }

  async setupExchanges() {
    // Exchange para eventos de usuario
    await this.channel.assertExchange('user.events', 'topic', { durable: true });
    
    // Exchange para eventos de órdenes
    await this.channel.assertExchange('order.events', 'topic', { durable: true });
    
    // Exchange para notificaciones
    await this.channel.assertExchange('notifications', 'fanout', { durable: true });
  }

  // Publicar mensaje
  async publish(exchange, routingKey, message) {
    try {
      const messageBuffer = Buffer.from(JSON.stringify(message));
      
      await this.channel.publish(exchange, routingKey, messageBuffer, {
        persistent: true,
        timestamp: Date.now(),
        messageId: generateId()
      });
      
      console.log(`Message published to ${exchange} with key ${routingKey}`);
    } catch (error) {
      console.error('Failed to publish message:', error);
      throw error;
    }
  }

  // Suscribirse a mensajes
  async subscribe(queueName, exchange, routingKey, handler) {
    try {
      // Crear queue
      const queue = await this.channel.assertQueue(queueName, { durable: true });
      
      // Bind queue a exchange
      await this.channel.bindQueue(queue.queue, exchange, routingKey);
      
      // Configurar consumer
      await this.channel.consume(queue.queue, async (msg) => {
        if (msg) {
          try {
            const message = JSON.parse(msg.content.toString());
            await handler(message);
            
            // Acknowledge message
            this.channel.ack(msg);
          } catch (error) {
            console.error('Error processing message:', error);
            
            // Reject message y enviar a DLQ
            this.channel.nack(msg, false, false);
          }
        }
      });
      
      console.log(`Subscribed to ${queueName} with routing key ${routingKey}`);
    } catch (error) {
      console.error('Failed to subscribe:', error);
      throw error;
    }
  }
}

// Uso de RabbitMQ
const rabbitMQ = new RabbitMQService();

// Publicar evento de usuario creado
async function publishUserCreated(userData) {
  await rabbitMQ.publish('user.events', 'user.created', {
    userId: userData.id,
    email: userData.email,
    name: userData.name,
    timestamp: new Date().toISOString()
  });
}

// Suscribirse a eventos de usuario
async function subscribeToUserEvents() {
  await rabbitMQ.subscribe(
    'user-service-queue',
    'user.events',
    'user.created',
    async (message) => {
      console.log('User created:', message);
      
      // Enviar email de bienvenida
      await sendWelcomeEmail(message.email, message.name);
      
      // Crear perfil inicial
      await createUserProfile(message.userId);
    }
  );
}
```

#### Redis Message Queue
```javascript
// Redis como message queue
const redis = require('redis');

class RedisQueue {
  constructor() {
    this.client = redis.createClient({
      host: 'localhost',
      port: 6379
    });
    
    this.subscriber = redis.createClient({
      host: 'localhost',
      port: 6379
    });
  }

  async connect() {
    await this.client.connect();
    await this.subscriber.connect();
    console.log('Connected to Redis');
  }

  // Enviar mensaje a queue
  async enqueue(queueName, message) {
    try {
      const messageData = {
        id: generateId(),
        data: message,
        timestamp: Date.now(),
        retries: 0
      };
      
      await this.client.lPush(queueName, JSON.stringify(messageData));
      console.log(`Message enqueued to ${queueName}`);
    } catch (error) {
      console.error('Failed to enqueue message:', error);
      throw error;
    }
  }

  // Procesar mensajes de queue
  async dequeue(queueName, handler) {
    try {
      while (true) {
        const result = await this.client.brPop(queueName, 0);
        
        if (result) {
          const messageData = JSON.parse(result.element);
          
          try {
            await handler(messageData.data);
            console.log(`Message processed from ${queueName}`);
          } catch (error) {
            console.error('Error processing message:', error);
            
            // Reintentar si no excede límite
            if (messageData.retries < 3) {
              messageData.retries++;
              await this.client.lPush(`${queueName}:retry`, JSON.stringify(messageData));
            } else {
              // Enviar a dead letter queue
              await this.client.lPush(`${queueName}:dlq`, JSON.stringify(messageData));
            }
          }
        }
      }
    } catch (error) {
      console.error('Error in dequeue:', error);
    }
  }

  // Pub/Sub
  async publish(channel, message) {
    await this.client.publish(channel, JSON.stringify(message));
  }

  async subscribe(channel, handler) {
    await this.subscriber.subscribe(channel, (message) => {
      const data = JSON.parse(message);
      handler(data);
    });
  }
}

// Uso de Redis Queue
const redisQueue = new RedisQueue();

// Procesar órdenes
async function processOrders() {
  await redisQueue.dequeue('order-processing', async (orderData) => {
    console.log('Processing order:', orderData);
    
    // Validar orden
    await validateOrder(orderData);
    
    // Procesar pago
    await processPayment(orderData);
    
    // Notificar al restaurante
    await notifyRestaurant(orderData);
  });
}
```

### 2. Event Streaming con Kafka

#### Configuración de Kafka
```javascript
// Kafka Producer
const kafka = require('kafkajs');

const kafkaClient = kafka({
  clientId: 'delivery-app',
  brokers: ['localhost:9092']
});

class KafkaProducer {
  constructor() {
    this.producer = kafkaClient.producer();
  }

  async connect() {
    await this.producer.connect();
    console.log('Kafka producer connected');
  }

  async publish(topic, message) {
    try {
      await this.producer.send({
        topic,
        messages: [{
          key: message.id || generateId(),
          value: JSON.stringify(message),
          timestamp: Date.now().toString()
        }]
      });
      
      console.log(`Message published to topic ${topic}`);
    } catch (error) {
      console.error('Failed to publish to Kafka:', error);
      throw error;
    }
  }

  async disconnect() {
    await this.producer.disconnect();
  }
}

// Kafka Consumer
class KafkaConsumer {
  constructor(groupId) {
    this.consumer = kafkaClient.consumer({ groupId });
  }

  async connect() {
    await this.consumer.connect();
    console.log('Kafka consumer connected');
  }

  async subscribe(topics) {
    await this.consumer.subscribe({ topics });
  }

  async run(handler) {
    await this.consumer.run({
      eachMessage: async ({ topic, partition, message }) => {
        try {
          const messageData = JSON.parse(message.value.toString());
          await handler(topic, messageData);
        } catch (error) {
          console.error('Error processing Kafka message:', error);
        }
      }
    });
  }

  async disconnect() {
    await this.consumer.disconnect();
  }
}

// Uso de Kafka
const kafkaProducer = new KafkaProducer();
const kafkaConsumer = new KafkaConsumer('delivery-app-group');

// Publicar evento de orden
async function publishOrderEvent(orderData) {
  await kafkaProducer.publish('order-events', {
    type: 'OrderCreated',
    orderId: orderData.id,
    userId: orderData.userId,
    restaurantId: orderData.restaurantId,
    total: orderData.total,
    timestamp: new Date().toISOString()
  });
}

// Consumir eventos de orden
async function consumeOrderEvents() {
  await kafkaConsumer.subscribe(['order-events']);
  
  await kafkaConsumer.run(async (topic, message) => {
    console.log(`Received message from ${topic}:`, message);
    
    switch (message.type) {
      case 'OrderCreated':
        await handleOrderCreated(message);
        break;
      case 'OrderConfirmed':
        await handleOrderConfirmed(message);
        break;
      case 'OrderCancelled':
        await handleOrderCancelled(message);
        break;
    }
  });
}
```

#### EventStore Implementation
```javascript
// EventStore para event sourcing
const { EventStoreClient } = require('@eventstore/db-client');

class EventStore {
  constructor() {
    this.client = new EventStoreClient({
      endpoint: 'localhost:2113'
    });
  }

  async connect() {
    await this.client.connect();
    console.log('Connected to EventStore');
  }

  // Append eventos a stream
  async appendEvents(streamName, events, expectedVersion) {
    try {
      const eventData = events.map(event => ({
        type: event.type,
        data: event.data,
        metadata: {
          timestamp: new Date().toISOString(),
          version: event.version || 1
        }
      }));

      await this.client.appendToStream(streamName, eventData, {
        expectedRevision: expectedVersion
      });

      console.log(`Events appended to stream ${streamName}`);
    } catch (error) {
      console.error('Failed to append events:', error);
      throw error;
    }
  }

  // Leer eventos de stream
  async readEvents(streamName, fromVersion = 0) {
    try {
      const events = [];
      
      for await (const event of this.client.readStream(streamName, {
        fromRevision: fromVersion
      })) {
        events.push({
          id: event.event.id,
          type: event.event.type,
          data: event.event.data,
          metadata: event.event.metadata,
          version: event.event.revision
        });
      }

      return events;
    } catch (error) {
      console.error('Failed to read events:', error);
      throw error;
    }
  }

  // Subscribe a eventos
  async subscribeToStream(streamName, handler) {
    try {
      const subscription = this.client.subscribeToStream(streamName, {
        fromRevision: 'end'
      });

      for await (const event of subscription) {
        await handler(event);
      }
    } catch (error) {
      console.error('Subscription error:', error);
    }
  }
}

// Uso de EventStore
const eventStore = new EventStore();

// Crear orden con event sourcing
async function createOrderWithEventSourcing(orderData) {
  const streamName = `order-${orderData.id}`;
  
  const events = [
    {
      type: 'OrderCreated',
      data: {
        orderId: orderData.id,
        userId: orderData.userId,
        restaurantId: orderData.restaurantId,
        items: orderData.items,
        total: orderData.total
      }
    }
  ];

  await eventStore.appendEvents(streamName, events, 'no-stream');
}

// Leer historial de orden
async function getOrderHistory(orderId) {
  const streamName = `order-${orderId}`;
  const events = await eventStore.readEvents(streamName);
  
  // Reconstruir estado desde eventos
  return replayEvents(events);
}
```

### 3. Event Sourcing

#### Event Store Implementation
```javascript
// Event Store personalizado
class CustomEventStore {
  constructor() {
    this.events = [];
    this.snapshots = new Map();
  }

  async appendEvent(streamId, event) {
    const eventRecord = {
      id: generateId(),
      streamId,
      eventType: event.type,
      eventData: event.data,
      timestamp: new Date().toISOString(),
      version: this.getNextVersion(streamId)
    };

    this.events.push(eventRecord);
    
    // Crear snapshot si es necesario
    if (this.shouldCreateSnapshot(streamId)) {
      await this.createSnapshot(streamId);
    }

    return eventRecord;
  }

  async getEvents(streamId, fromVersion = 0) {
    return this.events
      .filter(e => e.streamId === streamId && e.version > fromVersion)
      .sort((a, b) => a.version - b.version);
  }

  async getSnapshot(streamId) {
    return this.snapshots.get(streamId);
  }

  shouldCreateSnapshot(streamId) {
    const eventCount = this.events.filter(e => e.streamId === streamId).length;
    return eventCount % 100 === 0; // Snapshot cada 100 eventos
  }

  async createSnapshot(streamId) {
    const events = await this.getEvents(streamId);
    const aggregate = this.replayEvents(events);
    
    this.snapshots.set(streamId, {
      version: events.length,
      data: aggregate,
      timestamp: new Date().toISOString()
    });
  }

  replayEvents(events) {
    let aggregate = { version: 0 };
    
    for (const event of events) {
      aggregate = this.applyEvent(aggregate, event);
      aggregate.version = event.version;
    }
    
    return aggregate;
  }

  applyEvent(aggregate, event) {
    switch (event.eventType) {
      case 'OrderCreated':
        return {
          ...aggregate,
          id: event.eventData.orderId,
          userId: event.eventData.userId,
          restaurantId: event.eventData.restaurantId,
          items: event.eventData.items,
          total: event.eventData.total,
          status: 'CREATED'
        };
      
      case 'OrderConfirmed':
        return {
          ...aggregate,
          status: 'CONFIRMED',
          confirmedAt: event.timestamp
        };
      
      case 'OrderShipped':
        return {
          ...aggregate,
          status: 'SHIPPED',
          shippedAt: event.timestamp
        };
      
      default:
        return aggregate;
    }
  }
}
```

#### Aggregate Pattern
```javascript
// Order Aggregate
class OrderAggregate {
  constructor(id) {
    this.id = id;
    this.version = 0;
    this.status = 'CREATED';
    this.userId = null;
    this.restaurantId = null;
    this.items = [];
    this.total = 0;
    this.events = [];
  }

  static fromEvents(events) {
    const order = new OrderAggregate(events[0].eventData.orderId);
    
    for (const event of events) {
      order.applyEvent(event);
    }
    
    return order;
  }

  create(userId, restaurantId, items, total) {
    if (this.status !== 'CREATED') {
      throw new Error('Order already exists');
    }

    const event = {
      type: 'OrderCreated',
      data: {
        orderId: this.id,
        userId,
        restaurantId,
        items,
        total
      }
    };

    this.applyEvent(event);
    this.events.push(event);
  }

  confirm() {
    if (this.status !== 'CREATED') {
      throw new Error('Order cannot be confirmed');
    }

    const event = {
      type: 'OrderConfirmed',
      data: {
        orderId: this.id
      }
    };

    this.applyEvent(event);
    this.events.push(event);
  }

  ship() {
    if (this.status !== 'CONFIRMED') {
      throw new Error('Order must be confirmed before shipping');
    }

    const event = {
      type: 'OrderShipped',
      data: {
        orderId: this.id
      }
    };

    this.applyEvent(event);
    this.events.push(event);
  }

  applyEvent(event) {
    switch (event.type) {
      case 'OrderCreated':
        this.userId = event.data.userId;
        this.restaurantId = event.data.restaurantId;
        this.items = event.data.items;
        this.total = event.data.total;
        this.status = 'CREATED';
        break;
      
      case 'OrderConfirmed':
        this.status = 'CONFIRMED';
        break;
      
      case 'OrderShipped':
        this.status = 'SHIPPED';
        break;
    }
    
    this.version++;
  }

  getUncommittedEvents() {
    return this.events;
  }

  markEventsAsCommitted() {
    this.events = [];
  }
}
```

### 4. CQRS Pattern

#### Command Side
```javascript
// Command Handlers
class OrderCommandHandler {
  constructor(eventStore) {
    this.eventStore = eventStore;
  }

  async handle(command) {
    switch (command.type) {
      case 'CreateOrder':
        return await this.createOrder(command.data);
      case 'ConfirmOrder':
        return await this.confirmOrder(command.data);
      case 'ShipOrder':
        return await this.shipOrder(command.data);
      case 'CancelOrder':
        return await this.cancelOrder(command.data);
    }
  }

  async createOrder(data) {
    const orderId = generateId();
    const streamName = `order-${orderId}`;
    
    // Crear aggregate
    const order = new OrderAggregate(orderId);
    order.create(data.userId, data.restaurantId, data.items, data.total);
    
    // Guardar eventos
    const events = order.getUncommittedEvents();
    await this.eventStore.appendEvents(streamName, events, 'no-stream');
    
    order.markEventsAsCommitted();
    
    return { orderId, status: 'CREATED' };
  }

  async confirmOrder(data) {
    const streamName = `order-${data.orderId}`;
    
    // Cargar aggregate desde eventos
    const events = await this.eventStore.getEvents(streamName);
    const order = OrderAggregate.fromEvents(events);
    
    // Ejecutar comando
    order.confirm();
    
    // Guardar nuevos eventos
    const newEvents = order.getUncommittedEvents();
    await this.eventStore.appendEvents(streamName, newEvents, events.length - 1);
    
    order.markEventsAsCommitted();
    
    return { orderId: data.orderId, status: 'CONFIRMED' };
  }
}
```

#### Query Side
```javascript
// Read Models
class OrderReadModel {
  constructor() {
    this.orders = new Map();
  }

  async handleEvent(event) {
    switch (event.eventType) {
      case 'OrderCreated':
        await this.handleOrderCreated(event);
        break;
      case 'OrderConfirmed':
        await this.handleOrderConfirmed(event);
        break;
      case 'OrderShipped':
        await this.handleOrderShipped(event);
        break;
    }
  }

  async handleOrderCreated(event) {
    const order = {
      id: event.eventData.orderId,
      userId: event.eventData.userId,
      restaurantId: event.eventData.restaurantId,
      items: event.eventData.items,
      total: event.eventData.total,
      status: 'CREATED',
      createdAt: event.timestamp
    };

    this.orders.set(order.id, order);
  }

  async handleOrderConfirmed(event) {
    const order = this.orders.get(event.eventData.orderId);
    if (order) {
      order.status = 'CONFIRMED';
      order.confirmedAt = event.timestamp;
    }
  }

  async handleOrderShipped(event) {
    const order = this.orders.get(event.eventData.orderId);
    if (order) {
      order.status = 'SHIPPED';
      order.shippedAt = event.timestamp;
    }
  }

  // Query methods
  async getOrder(orderId) {
    return this.orders.get(orderId);
  }

  async getOrdersByUser(userId) {
    return Array.from(this.orders.values())
      .filter(order => order.userId === userId);
  }

  async getOrdersByStatus(status) {
    return Array.from(this.orders.values())
      .filter(order => order.status === status);
  }
}

// Query Handlers
class OrderQueryHandler {
  constructor(readModel) {
    this.readModel = readModel;
  }

  async handle(query) {
    switch (query.type) {
      case 'GetOrder':
        return await this.getOrder(query.data);
      case 'GetOrdersByUser':
        return await this.getOrdersByUser(query.data);
      case 'GetOrdersByStatus':
        return await this.getOrdersByStatus(query.data);
    }
  }

  async getOrder(data) {
    return await this.readModel.getOrder(data.orderId);
  }

  async getOrdersByUser(data) {
    return await this.readModel.getOrdersByUser(data.userId);
  }

  async getOrdersByStatus(data) {
    return await this.readModel.getOrdersByStatus(data.status);
  }
}
```

### 5. Event-Driven Architecture

#### Event Bus
```javascript
// Event Bus central
class EventBus {
  constructor() {
    this.handlers = new Map();
    this.middleware = [];
  }

  // Registrar handler para evento
  subscribe(eventType, handler) {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    
    this.handlers.get(eventType).push(handler);
  }

  // Publicar evento
  async publish(event) {
    try {
      // Aplicar middleware
      for (const middleware of this.middleware) {
        await middleware(event);
      }

      const handlers = this.handlers.get(event.type) || [];
      
      // Ejecutar handlers en paralelo
      await Promise.all(
        handlers.map(handler => this.executeHandler(handler, event))
      );
      
      console.log(`Event ${event.type} published successfully`);
    } catch (error) {
      console.error('Error publishing event:', error);
      throw error;
    }
  }

  async executeHandler(handler, event) {
    try {
      await handler(event);
    } catch (error) {
      console.error('Error in event handler:', error);
      // No re-throw para evitar que un handler falle afecte otros
    }
  }

  // Agregar middleware
  use(middleware) {
    this.middleware.push(middleware);
  }
}

// Uso del Event Bus
const eventBus = new EventBus();

// Middleware de logging
eventBus.use(async (event) => {
  console.log(`Processing event: ${event.type}`, event);
});

// Handlers
eventBus.subscribe('OrderCreated', async (event) => {
  console.log('Order created:', event.data);
  
  // Notificar al restaurante
  await notifyRestaurant(event.data.restaurantId, event.data);
  
  // Enviar confirmación al usuario
  await sendOrderConfirmation(event.data.userId, event.data);
});

eventBus.subscribe('OrderCreated', async (event) => {
  // Calcular tiempo estimado
  const estimatedTime = await calculateDeliveryTime(event.data);
  
  // Publicar evento de tiempo estimado
  await eventBus.publish({
    type: 'OrderEstimated',
    data: {
      orderId: event.data.orderId,
      estimatedTime
    }
  });
});
```

## 🛠️ Implementación Práctica

### Sistema Event-Driven Completo
```javascript
// Configuración del sistema
class EventDrivenSystem {
  constructor() {
    this.eventStore = new CustomEventStore();
    this.eventBus = new EventBus();
    this.commandHandler = new OrderCommandHandler(this.eventStore);
    this.queryHandler = new OrderQueryHandler(new OrderReadModel());
    
    this.setupEventHandlers();
  }

  setupEventHandlers() {
    // Handler para actualizar read models
    this.eventBus.subscribe('OrderCreated', async (event) => {
      await this.queryHandler.readModel.handleEvent(event);
    });

    this.eventBus.subscribe('OrderConfirmed', async (event) => {
      await this.queryHandler.readModel.handleEvent(event);
    });

    // Handler para notificaciones
    this.eventBus.subscribe('OrderCreated', async (event) => {
      await this.sendNotifications(event);
    });
  }

  async sendNotifications(event) {
    // Enviar push notification
    await sendPushNotification(event.data.userId, {
      title: 'Order Created',
      body: 'Your order has been created successfully'
    });

    // Enviar email
    await sendEmail(event.data.userId, 'order-created', event.data);
  }
}
```

## 📊 Métricas y KPIs

### Event Streaming Performance
- **Throughput**: >10,000 events/second
- **Latency**: <100ms end-to-end
- **Durability**: 99.99% message delivery
- **Ordering**: Event ordering garantizado
- **Replay**: Capacidad de replay completo

## 🔧 Herramientas y Librerías

### Message Queues
- **RabbitMQ**: Message broker robusto
- **Redis**: In-memory data store
- **Amazon SQS**: Managed message queue
- **Azure Service Bus**: Enterprise messaging

### Event Streaming
- **Apache Kafka**: Distributed streaming platform
- **EventStore**: Event sourcing database
- **Apache Pulsar**: Cloud-native messaging
- **NATS**: Cloud native messaging system

## 🎯 Proyecto Integrador

### Sistema Event-Driven para Delivery App
Implementar un sistema completo con:

1. **Message Queues** con RabbitMQ y Redis
2. **Event Streaming** con Kafka
3. **Event Sourcing** con EventStore
4. **CQRS Pattern** implementado
5. **Event Bus** centralizado
6. **Read Models** optimizados
7. **Event Handlers** robustos

## 📚 Recursos Adicionales

### Documentación
- [RabbitMQ Documentation](https://www.rabbitmq.com/documentation.html)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [EventStore Documentation](https://developers.eventstore.com/)

### Cursos Recomendados
- Event-Driven Architecture
- Apache Kafka Fundamentals
- Event Sourcing Patterns
- CQRS Implementation

## ✅ Evaluación

### Criterios de Evaluación
- Implementación correcta de message queues
- Event streaming funcional
- Event sourcing implementado
- CQRS pattern aplicado
- Event-driven architecture robusta

---

**Siguiente**: [Clase 5: Escalabilidad y Performance](./clase_5_escalabilidad_performance.md)
**Anterior**: [Clase 3: Serverless y Cloud Functions](./clase_3_serverless_cloud_functions.md)
