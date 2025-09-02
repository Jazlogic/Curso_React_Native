# ☁️ Clase 3: Serverless y Cloud Functions

## 📋 Objetivos de la Clase

- Comprender arquitecturas serverless para móviles
- Implementar AWS Lambda y Azure Functions
- Configurar Firebase Functions y Google Cloud
- Desarrollar programación event-driven
- Optimizar cold starts y performance

## 🎯 Contenido Principal

### 1. Serverless Architecture para Móviles

#### Fundamentos de Serverless
```javascript
// Arquitectura tradicional vs serverless
const traditionalArchitecture = {
  servers: 'Mantenimiento constante',
  scaling: 'Manual o automático complejo',
  cost: 'Fijo independiente del uso',
  deployment: 'Complejo con downtime',
  monitoring: 'Requiere configuración'
};

const serverlessArchitecture = {
  servers: 'Gestionados por el proveedor',
  scaling: 'Automático e instantáneo',
  cost: 'Pay-per-execution',
  deployment: 'Simple sin downtime',
  monitoring: 'Integrado y automático'
};

// Beneficios para aplicaciones móviles
const mobileBenefits = {
  backend: 'API endpoints sin servidor',
  processing: 'Image/video processing',
  notifications: 'Push notifications',
  authentication: 'User management',
  storage: 'File uploads y downloads',
  realtime: 'WebSocket connections'
};
```

#### Patrones Serverless
```javascript
// 1. API Gateway + Lambda
const apiPattern = {
  client: 'React Native App',
  gateway: 'API Gateway (routing, auth, rate limiting)',
  functions: [
    'user-service-lambda',
    'order-service-lambda',
    'payment-service-lambda'
  ],
  database: 'Managed database (RDS, DynamoDB)'
};

// 2. Event-Driven Architecture
const eventDrivenPattern = {
  triggers: [
    'S3 upload -> image processing',
    'DynamoDB change -> notification',
    'Scheduled -> cleanup task',
    'HTTP request -> API response'
  ],
  processing: 'Stateless functions',
  storage: 'Event store + database'
};

// 3. Microservices Serverless
const microservicesPattern = {
  services: [
    { name: 'auth-service', runtime: 'Node.js' },
    { name: 'user-service', runtime: 'Python' },
    { name: 'order-service', runtime: 'Go' },
    { name: 'payment-service', runtime: 'Java' }
  ],
  communication: 'Event-driven + API calls',
  deployment: 'Independent deployments'
};
```

### 2. AWS Lambda

#### Configuración de Lambda
```javascript
// Lambda function básica
exports.handler = async (event, context) => {
  try {
    // Parsear el evento
    const { httpMethod, path, body, headers } = event;
    
    // Logging
    console.log('Event:', JSON.stringify(event, null, 2));
    
    // Procesar request
    let response;
    switch (httpMethod) {
      case 'GET':
        response = await handleGet(path, headers);
        break;
      case 'POST':
        response = await handlePost(path, JSON.parse(body), headers);
        break;
      case 'PUT':
        response = await handlePut(path, JSON.parse(body), headers);
        break;
      case 'DELETE':
        response = await handleDelete(path, headers);
        break;
      default:
        throw new Error(`Method ${httpMethod} not allowed`);
    }
    
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,Authorization',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
      },
      body: JSON.stringify(response)
    };
    
  } catch (error) {
    console.error('Error:', error);
    
    return {
      statusCode: 500,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify({
        error: error.message,
        timestamp: new Date().toISOString()
      })
    };
  }
};

// Handlers específicos
async function handleGet(path, headers) {
  const userId = extractUserId(headers);
  
  if (path === '/users/profile') {
    return await getUserProfile(userId);
  } else if (path.startsWith('/users/orders')) {
    return await getUserOrders(userId);
  }
  
  throw new Error('Path not found');
}

async function handlePost(path, body, headers) {
  const userId = extractUserId(headers);
  
  if (path === '/orders') {
    return await createOrder(userId, body);
  } else if (path === '/payments') {
    return await processPayment(userId, body);
  }
  
  throw new Error('Path not found');
}
```

#### Integración con DynamoDB
```javascript
// DynamoDB operations
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();

// Crear usuario
async function createUser(userData) {
  const params = {
    TableName: 'Users',
    Item: {
      userId: userData.id,
      email: userData.email,
      name: userData.name,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    },
    ConditionExpression: 'attribute_not_exists(userId)'
  };
  
  try {
    await dynamodb.put(params).promise();
    return { success: true, userId: userData.id };
  } catch (error) {
    if (error.code === 'ConditionalCheckFailedException') {
      throw new Error('User already exists');
    }
    throw error;
  }
}

// Obtener usuario
async function getUser(userId) {
  const params = {
    TableName: 'Users',
    Key: { userId }
  };
  
  const result = await dynamodb.get(params).promise();
  return result.Item;
}

// Query con índices
async function getUserOrders(userId, limit = 20) {
  const params = {
    TableName: 'Orders',
    IndexName: 'UserIdIndex',
    KeyConditionExpression: 'userId = :userId',
    ExpressionAttributeValues: {
      ':userId': userId
    },
    Limit: limit,
    ScanIndexForward: false // Orden descendente
  };
  
  const result = await dynamodb.query(params).promise();
  return result.Items;
}

// Batch operations
async function batchCreateOrders(orders) {
  const params = {
    RequestItems: {
      'Orders': orders.map(order => ({
        PutRequest: {
          Item: {
            orderId: order.id,
            userId: order.userId,
            restaurantId: order.restaurantId,
            total: order.total,
            status: 'PENDING',
            items: order.items,
            createdAt: new Date().toISOString()
          }
        }
      }))
    }
  };
  
  await dynamodb.batchWrite(params).promise();
}
```

#### S3 Integration
```javascript
// S3 operations para archivos
const s3 = new AWS.S3();

// Upload de imagen de perfil
async function uploadProfileImage(userId, imageData) {
  const key = `users/${userId}/profile/${Date.now()}.jpg`;
  
  const params = {
    Bucket: process.env.PROFILE_IMAGES_BUCKET,
    Key: key,
    Body: Buffer.from(imageData, 'base64'),
    ContentType: 'image/jpeg',
    ACL: 'public-read'
  };
  
  const result = await s3.upload(params).promise();
  
  // Actualizar URL en DynamoDB
  await updateUserProfileImage(userId, result.Location);
  
  return { imageUrl: result.Location };
}

// Generar URL firmada para upload
async function generateUploadUrl(userId, fileType) {
  const key = `users/${userId}/uploads/${Date.now()}.${fileType}`;
  
  const params = {
    Bucket: process.env.UPLOADS_BUCKET,
    Key: key,
    Expires: 300, // 5 minutos
    ContentType: `image/${fileType}`
  };
  
  const uploadUrl = s3.getSignedUrl('putObject', params);
  
  return {
    uploadUrl,
    key,
    expiresIn: 300
  };
}
```

### 3. Azure Functions

#### Configuración de Azure Functions
```javascript
// Azure Function con Node.js
const { app } = require('@azure/functions');

// HTTP trigger
app.http('userProfile', {
  methods: ['GET', 'POST'],
  authLevel: 'function',
  route: 'users/{userId}/profile',
  handler: async (request, context) => {
    context.log('HTTP function processed a request.');
    
    const userId = request.params.userId;
    const method = request.method;
    
    try {
      if (method === 'GET') {
        const profile = await getUserProfile(userId);
        return {
          status: 200,
          body: profile
        };
      } else if (method === 'POST') {
        const body = await request.json();
        const updatedProfile = await updateUserProfile(userId, body);
        return {
          status: 200,
          body: updatedProfile
        };
      }
    } catch (error) {
      context.log.error('Error:', error);
      return {
        status: 500,
        body: { error: error.message }
      };
    }
  }
});

// Timer trigger
app.timer('cleanupExpiredOrders', {
  schedule: '0 0 2 * * *', // Diario a las 2 AM
  handler: async (myTimer, context) => {
    context.log('Cleanup function executed');
    
    const expiredOrders = await getExpiredOrders();
    let cleanedCount = 0;
    
    for (const order of expiredOrders) {
      await cancelOrder(order.id);
      cleanedCount++;
    }
    
    context.log(`Cleaned up ${cleanedCount} expired orders`);
  }
});

// Blob trigger
app.storageBlob('processUploadedImage', {
  path: 'uploads/{name}',
  connection: 'AzureWebJobsStorage',
  handler: async (blob, context) => {
    context.log(`Processing blob: ${blob.name}`);
    
    // Procesar imagen
    const processedImage = await processImage(blob);
    
    // Guardar en storage
    await saveProcessedImage(processedImage);
    
    // Notificar al usuario
    await notifyUserImageProcessed(blob.name);
  }
});
```

#### Cosmos DB Integration
```javascript
// Cosmos DB operations
const { CosmosClient } = require('@azure/cosmos');

const client = new CosmosClient({
  endpoint: process.env.COSMOS_ENDPOINT,
  key: process.env.COSMOS_KEY
});

const database = client.database('DeliveryApp');
const usersContainer = database.container('Users');
const ordersContainer = database.container('Orders');

// Crear usuario
async function createUser(userData) {
  const user = {
    id: userData.id,
    email: userData.email,
    name: userData.name,
    type: 'user',
    createdAt: new Date().toISOString()
  };
  
  const { resource } = await usersContainer.items.create(user);
  return resource;
}

// Query con SQL
async function getUserOrders(userId, status = null) {
  let query = 'SELECT * FROM c WHERE c.userId = @userId';
  const parameters = [{ name: '@userId', value: userId }];
  
  if (status) {
    query += ' AND c.status = @status';
    parameters.push({ name: '@status', value: status });
  }
  
  query += ' ORDER BY c.createdAt DESC';
  
  const { resources } = await ordersContainer.items
    .query({ query, parameters })
    .fetchAll();
    
  return resources;
}

// Stored procedure
async function updateOrderStatus(orderId, status) {
  const { resource } = await ordersContainer.scripts
    .storedProcedure('updateOrderStatus')
    .execute([orderId, status]);
    
  return resource;
}
```

### 4. Firebase Functions

#### Configuración de Firebase Functions
```javascript
// Firebase Functions con TypeScript
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

admin.initializeApp();

// HTTP function
export const createUser = functions.https.onRequest(async (req, res) => {
  try {
    const { email, password, name } = req.body;
    
    // Crear usuario en Authentication
    const userRecord = await admin.auth().createUser({
      email,
      password,
      displayName: name
    });
    
    // Crear documento en Firestore
    await admin.firestore().collection('users').doc(userRecord.uid).set({
      name,
      email,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
      preferences: {
        notifications: true,
        theme: 'light'
      }
    });
    
    res.status(201).json({
      success: true,
      userId: userRecord.uid
    });
    
  } catch (error) {
    console.error('Error creating user:', error);
    res.status(500).json({ error: error.message });
  }
});

// Firestore trigger
export const onOrderCreated = functions.firestore
  .document('orders/{orderId}')
  .onCreate(async (snap, context) => {
    const order = snap.data();
    const orderId = context.params.orderId;
    
    console.log('New order created:', orderId);
    
    // Notificar al restaurante
    await notifyRestaurant(order.restaurantId, order);
    
    // Calcular tiempo estimado
    const estimatedTime = await calculateDeliveryTime(order);
    
    // Actualizar orden con tiempo estimado
    await snap.ref.update({
      estimatedDelivery: estimatedTime,
      status: 'CONFIRMED'
    });
    
    // Enviar notificación push al usuario
    await sendPushNotification(order.userId, {
      title: 'Order Confirmed',
      body: `Your order will be delivered in ${estimatedTime} minutes`
    });
  });

// Authentication trigger
export const onUserCreated = functions.auth.user().onCreate(async (user) => {
  console.log('New user created:', user.uid);
  
  // Crear perfil básico
  await admin.firestore().collection('userProfiles').doc(user.uid).set({
    email: user.email,
    displayName: user.displayName,
    photoURL: user.photoURL,
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
    lastLogin: admin.firestore.FieldValue.serverTimestamp()
  });
  
  // Enviar email de bienvenida
  await sendWelcomeEmail(user.email, user.displayName);
});
```

#### Cloud Storage Integration
```javascript
// Storage trigger para procesamiento de imágenes
export const processProfileImage = functions.storage
  .object()
  .onFinalize(async (object) => {
    const filePath = object.name;
    const bucketName = object.bucket;
    
    console.log('Processing image:', filePath);
    
    // Verificar que es una imagen de perfil
    if (!filePath.startsWith('profile-images/')) {
      return;
    }
    
    const bucket = admin.storage().bucket(bucketName);
    const file = bucket.file(filePath);
    
    // Generar thumbnails
    const sizes = [150, 300, 600];
    
    for (const size of sizes) {
      const thumbnailPath = filePath.replace('profile-images/', `thumbnails/${size}/`);
      const thumbnailFile = bucket.file(thumbnailPath);
      
      await generateThumbnail(file, thumbnailFile, size);
    }
    
    // Actualizar URL en Firestore
    const userId = filePath.split('/')[1];
    const imageUrl = `https://storage.googleapis.com/${bucketName}/${filePath}`;
    
    await admin.firestore().collection('users').doc(userId).update({
      profileImageUrl: imageUrl,
      profileImageUpdated: admin.firestore.FieldValue.serverTimestamp()
    });
  });

// Función para generar thumbnails
async function generateThumbnail(sourceFile, destFile, size) {
  const sharp = require('sharp');
  
  const [sourceData] = await sourceFile.download();
  
  const thumbnailData = await sharp(sourceData)
    .resize(size, size, { fit: 'cover' })
    .jpeg({ quality: 80 })
    .toBuffer();
  
  await destFile.save(thumbnailData, {
    metadata: {
      contentType: 'image/jpeg'
    }
  });
}
```

### 5. Event-Driven Programming

#### Event Sourcing Pattern
```javascript
// Event Store
class EventStore {
  constructor() {
    this.events = [];
  }
  
  async appendEvent(streamId, event) {
    const eventRecord = {
      streamId,
      eventId: generateId(),
      eventType: event.type,
      eventData: event.data,
      timestamp: new Date().toISOString(),
      version: this.getNextVersion(streamId)
    };
    
    this.events.push(eventRecord);
    
    // Publicar evento
    await this.publishEvent(eventRecord);
    
    return eventRecord;
  }
  
  async getEvents(streamId, fromVersion = 0) {
    return this.events
      .filter(e => e.streamId === streamId && e.version > fromVersion)
      .sort((a, b) => a.version - b.version);
  }
  
  async publishEvent(eventRecord) {
    // Publicar a message queue o event bus
    console.log('Publishing event:', eventRecord);
  }
}

// Event Handlers
class OrderEventHandler {
  async handle(event) {
    switch (event.eventType) {
      case 'OrderCreated':
        await this.handleOrderCreated(event);
        break;
      case 'OrderConfirmed':
        await this.handleOrderConfirmed(event);
        break;
      case 'OrderCancelled':
        await this.handleOrderCancelled(event);
        break;
    }
  }
  
  async handleOrderCreated(event) {
    const { orderId, userId, restaurantId, items } = event.eventData;
    
    // Notificar al restaurante
    await this.notifyRestaurant(restaurantId, { orderId, items });
    
    // Calcular tiempo estimado
    const estimatedTime = await this.calculateDeliveryTime(restaurantId, items);
    
    // Emitir evento OrderEstimated
    await eventStore.appendEvent(`order-${orderId}`, {
      type: 'OrderEstimated',
      data: { orderId, estimatedTime }
    });
  }
}
```

#### CQRS Pattern
```javascript
// Command Side
class OrderCommandHandler {
  async handle(command) {
    switch (command.type) {
      case 'CreateOrder':
        return await this.createOrder(command.data);
      case 'UpdateOrderStatus':
        return await this.updateOrderStatus(command.data);
      case 'CancelOrder':
        return await this.cancelOrder(command.data);
    }
  }
  
  async createOrder(data) {
    const orderId = generateId();
    
    // Validar datos
    await this.validateOrderData(data);
    
    // Crear evento
    await eventStore.appendEvent(`order-${orderId}`, {
      type: 'OrderCreated',
      data: { orderId, ...data }
    });
    
    return { orderId, status: 'CREATED' };
  }
}

// Query Side
class OrderQueryHandler {
  async getOrder(orderId) {
    const events = await eventStore.getEvents(`order-${orderId}`);
    
    // Reconstruir estado desde eventos
    const order = this.replayEvents(events);
    
    return order;
  }
  
  async getOrdersByUser(userId) {
    // Query desde read model
    const orders = await this.readModel.getOrdersByUser(userId);
    return orders;
  }
  
  replayEvents(events) {
    let order = { status: 'CREATED' };
    
    for (const event of events) {
      switch (event.eventType) {
        case 'OrderCreated':
          order = { ...order, ...event.eventData };
          break;
        case 'OrderConfirmed':
          order.status = 'CONFIRMED';
          break;
        case 'OrderShipped':
          order.status = 'SHIPPED';
          break;
        case 'OrderDelivered':
          order.status = 'DELIVERED';
          break;
      }
    }
    
    return order;
  }
}
```

## 🛠️ Implementación Práctica

### Sistema Serverless Completo
```javascript
// Configuración de deployment
const serverlessConfig = {
  service: 'delivery-app-backend',
  provider: {
    name: 'aws',
    runtime: 'nodejs18.x',
    region: 'us-east-1',
    environment: {
      DYNAMODB_TABLE: 'DeliveryApp',
      S3_BUCKET: 'delivery-app-uploads'
    },
    iam: {
      role: {
        statements: [
          {
            Effect: 'Allow',
            Action: [
              'dynamodb:Query',
              'dynamodb:Scan',
              'dynamodb:GetItem',
              'dynamodb:PutItem',
              'dynamodb:UpdateItem',
              'dynamodb:DeleteItem'
            ],
            Resource: 'arn:aws:dynamodb:${opt:region}:*:table/DeliveryApp'
          }
        ]
      }
    }
  },
  functions: {
    userService: {
      handler: 'src/user.handler',
      events: [
        {
          http: {
            path: 'users/{proxy+}',
            method: 'ANY',
            cors: true
          }
        }
      ]
    },
    orderService: {
      handler: 'src/order.handler',
      events: [
        {
          http: {
            path: 'orders/{proxy+}',
            method: 'ANY',
            cors: true
          }
        }
      ]
    },
    processImage: {
      handler: 'src/image.handler',
      events: [
        {
          s3: {
            bucket: 'delivery-app-uploads',
            event: 's3:ObjectCreated:*'
          }
        }
      ]
    }
  }
};
```

## 📊 Métricas y KPIs

### Serverless Performance
- **Cold Start Time**: <1s
- **Warm Start Time**: <100ms
- **Memory Usage**: Optimizado
- **Error Rate**: <0.1%
- **Cost per Request**: Minimizado

## 🔧 Herramientas y Librerías

### Serverless Frameworks
- **Serverless Framework**: Deployment y gestión
- **AWS SAM**: Serverless Application Model
- **Azure Functions Core Tools**: CLI para Azure
- **Firebase CLI**: Herramientas de Firebase

### Monitoring
- **AWS CloudWatch**: Monitoreo de Lambda
- **Azure Monitor**: Monitoreo de Functions
- **Firebase Analytics**: Analytics de Functions
- **Datadog**: APM para serverless

## 🎯 Proyecto Integrador

### Backend Serverless para Delivery App
Implementar un sistema completo con:

1. **AWS Lambda Functions** para APIs
2. **Azure Functions** para procesamiento
3. **Firebase Functions** para notificaciones
4. **Event-Driven Architecture** con SQS/SNS
5. **CQRS Pattern** con Event Sourcing
6. **Cold Start Optimization**
7. **Monitoring y Logging** completo

## 📚 Recursos Adicionales

### Documentación
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Azure Functions Documentation](https://docs.microsoft.com/azure/azure-functions/)
- [Firebase Functions Documentation](https://firebase.google.com/docs/functions)

### Cursos Recomendados
- Serverless Architecture Fundamentals
- AWS Lambda Deep Dive
- Azure Functions Development
- Firebase Functions Mastery

## ✅ Evaluación

### Criterios de Evaluación
- Implementación correcta de serverless functions
- Optimización de cold starts
- Event-driven architecture funcional
- Integración con servicios cloud
- Performance y escalabilidad

---

**Siguiente**: [Clase 4: Message Queues y Event Streaming](./clase_4_message_queues_event_streaming.md)
**Anterior**: [Clase 2: GraphQL y APIs Modernas](./clase_2_graphql_apis_modernas.md)
