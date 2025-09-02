# 🔗 Clase 2: GraphQL y APIs Modernas

## 📋 Objetivos de la Clase

- Comprender GraphQL vs REST APIs
- Diseñar schemas GraphQL efectivos
- Implementar Apollo Client en React Native
- Configurar subscriptions y real-time data
- Optimizar performance y caching

## 🎯 Contenido Principal

### 1. GraphQL vs REST APIs

#### Comparación de Paradigmas
```javascript
// REST API - Múltiples requests
const restExample = {
  // Para obtener datos de usuario y sus órdenes
  requests: [
    'GET /api/users/123',
    'GET /api/users/123/orders',
    'GET /api/users/123/profile',
    'GET /api/users/123/preferences'
  ],
  problems: [
    'Over-fetching: datos innecesarios',
    'Under-fetching: múltiples requests',
    'Versioning complejo',
    'Documentación dispersa'
  ]
};

// GraphQL - Un solo request
const graphqlExample = {
  query: `
    query GetUserWithOrders($userId: ID!) {
      user(id: $userId) {
        id
        name
        email
        orders {
          id
          total
          status
          items {
            name
            price
          }
        }
        profile {
          avatar
          bio
        }
        preferences {
          notifications
          theme
        }
      }
    }
  `,
  benefits: [
    'Un solo request',
    'Solo datos necesarios',
    'Type-safe',
    'Auto-documentación'
  ]
};
```

#### Ventajas de GraphQL
```javascript
// 1. Type Safety
const userSchema = `
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
    orders: [Order!]!
    createdAt: DateTime!
  }

  type Order {
    id: ID!
    total: Float!
    status: OrderStatus!
    items: [OrderItem!]!
  }

  enum OrderStatus {
    PENDING
    CONFIRMED
    SHIPPED
    DELIVERED
    CANCELLED
  }
`;

// 2. Introspection
const introspectionQuery = `
  query IntrospectionQuery {
    __schema {
      types {
        name
        kind
        description
        fields {
          name
          type {
            name
            kind
          }
        }
      }
    }
  }
`;

// 3. Real-time con Subscriptions
const subscriptionExample = `
  subscription OrderStatusUpdate($orderId: ID!) {
    orderStatusUpdate(orderId: $orderId) {
      id
      status
      updatedAt
      estimatedDelivery
    }
  }
`;
```

### 2. Schema Design y Resolvers

#### Diseño de Schema
```javascript
// Schema principal
const typeDefs = `
  type Query {
    user(id: ID!): User
    users(limit: Int, offset: Int): [User!]!
    order(id: ID!): Order
    orders(userId: ID, status: OrderStatus): [Order!]!
    restaurant(id: ID!): Restaurant
    restaurants(location: LocationInput): [Restaurant!]!
  }

  type Mutation {
    createUser(input: CreateUserInput!): User!
    updateUser(id: ID!, input: UpdateUserInput!): User!
    createOrder(input: CreateOrderInput!): Order!
    updateOrderStatus(id: ID!, status: OrderStatus!): Order!
  }

  type Subscription {
    orderStatusUpdate(orderId: ID!): Order!
    newOrder(userId: ID!): Order!
    restaurantUpdate(restaurantId: ID!): Restaurant!
  }

  input CreateUserInput {
    name: String!
    email: String!
    password: String!
    phone: String
  }

  input CreateOrderInput {
    userId: ID!
    restaurantId: ID!
    items: [OrderItemInput!]!
    deliveryAddress: AddressInput!
  }

  type User {
    id: ID!
    name: String!
    email: String!
    phone: String
    orders: [Order!]!
    profile: UserProfile
    preferences: UserPreferences
    createdAt: DateTime!
    updatedAt: DateTime!
  }

  type Order {
    id: ID!
    user: User!
    restaurant: Restaurant!
    items: [OrderItem!]!
    total: Float!
    status: OrderStatus!
    deliveryAddress: Address!
    estimatedDelivery: DateTime
    createdAt: DateTime!
    updatedAt: DateTime!
  }
`;

// Resolvers
const resolvers = {
  Query: {
    user: async (parent, { id }, { dataSources }) => {
      return await dataSources.userAPI.getUser(id);
    },
    
    users: async (parent, { limit = 10, offset = 0 }, { dataSources }) => {
      return await dataSources.userAPI.getUsers({ limit, offset });
    },
    
    order: async (parent, { id }, { dataSources }) => {
      return await dataSources.orderAPI.getOrder(id);
    },
    
    orders: async (parent, { userId, status }, { dataSources }) => {
      return await dataSources.orderAPI.getOrders({ userId, status });
    }
  },

  Mutation: {
    createUser: async (parent, { input }, { dataSources }) => {
      return await dataSources.userAPI.createUser(input);
    },
    
    updateUser: async (parent, { id, input }, { dataSources }) => {
      return await dataSources.userAPI.updateUser(id, input);
    },
    
    createOrder: async (parent, { input }, { dataSources, pubsub }) => {
      const order = await dataSources.orderAPI.createOrder(input);
      
      // Publicar evento para subscription
      pubsub.publish('NEW_ORDER', {
        newOrder: order,
        userId: input.userId
      });
      
      return order;
    }
  },

  Subscription: {
    orderStatusUpdate: {
      subscribe: (parent, { orderId }, { pubsub }) => {
        return pubsub.asyncIterator(`ORDER_STATUS_${orderId}`);
      }
    },
    
    newOrder: {
      subscribe: (parent, { userId }, { pubsub }) => {
        return pubsub.asyncIterator('NEW_ORDER');
      },
      resolve: (payload) => {
        if (payload.userId === payload.newOrder.userId) {
          return payload.newOrder;
        }
        return null;
      }
    }
  },

  // Field resolvers
  User: {
    orders: async (parent, args, { dataSources }) => {
      return await dataSources.orderAPI.getOrdersByUser(parent.id);
    }
  },

  Order: {
    user: async (parent, args, { dataSources }) => {
      return await dataSources.userAPI.getUser(parent.userId);
    },
    
    restaurant: async (parent, args, { dataSources }) => {
      return await dataSources.restaurantAPI.getRestaurant(parent.restaurantId);
    }
  }
};
```

#### Data Sources
```javascript
// Data Source para Users
class UserAPI extends DataSource {
  constructor() {
    super();
    this.baseURL = 'http://user-service:3001';
  }

  async getUser(id) {
    const response = await this.get(`/users/${id}`);
    return this.userReducer(response);
  }

  async getUsers({ limit, offset }) {
    const response = await this.get(`/users?limit=${limit}&offset=${offset}`);
    return response.map(user => this.userReducer(user));
  }

  async createUser(input) {
    const response = await this.post('/users', input);
    return this.userReducer(response);
  }

  userReducer(user) {
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      phone: user.phone,
      createdAt: user.created_at,
      updatedAt: user.updated_at
    };
  }
}

// Data Source para Orders
class OrderAPI extends DataSource {
  constructor() {
    super();
    this.baseURL = 'http://order-service:3002';
  }

  async getOrder(id) {
    const response = await this.get(`/orders/${id}`);
    return this.orderReducer(response);
  }

  async createOrder(input) {
    const response = await this.post('/orders', input);
    return this.orderReducer(response);
  }

  async updateOrderStatus(id, status) {
    const response = await this.patch(`/orders/${id}/status`, { status });
    return this.orderReducer(response);
  }

  orderReducer(order) {
    return {
      id: order.id,
      userId: order.user_id,
      restaurantId: order.restaurant_id,
      total: order.total,
      status: order.status,
      items: order.items,
      deliveryAddress: order.delivery_address,
      estimatedDelivery: order.estimated_delivery,
      createdAt: order.created_at,
      updatedAt: order.updated_at
    };
  }
}
```

### 3. Apollo Client en React Native

#### Configuración de Apollo Client
```javascript
// Configuración de Apollo Client
import { ApolloClient, InMemoryCache, createHttpLink, split } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';
import { onError } from '@apollo/client/link/error';
import { WebSocketLink } from '@apollo/client/link/ws';
import { getMainDefinition } from '@apollo/client/utilities';

// HTTP Link para queries y mutations
const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
});

// WebSocket Link para subscriptions
const wsLink = new WebSocketLink({
  uri: 'ws://localhost:4000/graphql',
  options: {
    reconnect: true,
    connectionParams: {
      authorization: `Bearer ${getToken()}`,
    },
  },
});

// Auth Link
const authLink = setContext((_, { headers }) => {
  const token = getToken();
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : "",
    }
  }
});

// Error Link
const errorLink = onError(({ graphQLErrors, networkError, operation, forward }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
      )
    );
  }

  if (networkError) {
    console.log(`[Network error]: ${networkError}`);
    
    // Retry logic
    if (networkError.statusCode === 401) {
      // Redirect to login
      redirectToLogin();
    }
  }
});

// Split Link (HTTP for queries/mutations, WS for subscriptions)
const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  authLink.concat(httpLink)
);

// Apollo Client
const client = new ApolloClient({
  link: errorLink.concat(splitLink),
  cache: new InMemoryCache({
    typePolicies: {
      User: {
        fields: {
          orders: {
            merge(existing = [], incoming) {
              return [...existing, ...incoming];
            }
          }
        }
      }
    }
  }),
  defaultOptions: {
    watchQuery: {
      errorPolicy: 'all',
    },
    query: {
      errorPolicy: 'all',
    },
  },
});
```

#### Hooks de Apollo Client
```javascript
// Componente con useQuery
import { useQuery, useMutation, useSubscription } from '@apollo/client';

const UserProfile = ({ userId }) => {
  const { data, loading, error, refetch } = useQuery(GET_USER, {
    variables: { id: userId },
    errorPolicy: 'all',
    notifyOnNetworkStatusChange: true
  });

  const [updateUser] = useMutation(UPDATE_USER, {
    onCompleted: (data) => {
      console.log('User updated:', data);
    },
    onError: (error) => {
      console.error('Update error:', error);
    },
    update: (cache, { data }) => {
      // Actualizar cache local
      cache.modify({
        id: cache.identify({ __typename: 'User', id: userId }),
        fields: {
          name: () => data.updateUser.name,
          email: () => data.updateUser.email
        }
      });
    }
  });

  // Subscription para updates en tiempo real
  const { data: orderData } = useSubscription(ORDER_STATUS_UPDATE, {
    variables: { orderId: '123' },
    onSubscriptionData: ({ subscriptionData }) => {
      console.log('Order status updated:', subscriptionData.data);
    }
  });

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <View>
      <Text>{data.user.name}</Text>
      <Text>{data.user.email}</Text>
      <Button 
        title="Update Profile" 
        onPress={() => updateUser({ 
          variables: { 
            id: userId, 
            input: { name: 'New Name' } 
          } 
        })} 
      />
    </View>
  );
};
```

### 4. Subscriptions y Real-time Data

#### Configuración de Subscriptions
```javascript
// Server-side subscription setup
const { PubSub } = require('graphql-subscriptions');
const pubsub = new PubSub();

// Resolver de subscription
const resolvers = {
  Subscription: {
    orderStatusUpdate: {
      subscribe: (parent, { orderId }, { pubsub }) => {
        return pubsub.asyncIterator(`ORDER_STATUS_${orderId}`);
      }
    },
    
    newOrder: {
      subscribe: (parent, { userId }, { pubsub }) => {
        return pubsub.asyncIterator('NEW_ORDER');
      },
      resolve: (payload) => {
        return payload.newOrder;
      }
    },
    
    restaurantUpdate: {
      subscribe: (parent, { restaurantId }, { pubsub }) => {
        return pubsub.asyncIterator(`RESTAURANT_${restaurantId}`);
      }
    }
  }
};

// Publicar eventos
class OrderService {
  async updateOrderStatus(orderId, status) {
    const order = await this.updateStatus(orderId, status);
    
    // Publicar evento
    pubsub.publish(`ORDER_STATUS_${orderId}`, {
      orderStatusUpdate: order
    });
    
    return order;
  }
  
  async createOrder(orderData) {
    const order = await this.create(orderData);
    
    // Publicar evento
    pubsub.publish('NEW_ORDER', {
      newOrder: order,
      userId: orderData.userId
    });
    
    return order;
  }
}
```

#### React Native con Subscriptions
```javascript
// Componente con subscription
const OrderTracker = ({ orderId }) => {
  const { data, loading, error } = useSubscription(ORDER_STATUS_UPDATE, {
    variables: { orderId },
    onSubscriptionData: ({ subscriptionData }) => {
      const order = subscriptionData.data.orderStatusUpdate;
      
      // Mostrar notificación
      if (order.status === 'DELIVERED') {
        showNotification('Order Delivered!', 'Your order has been delivered.');
      }
    }
  });

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  const order = data?.orderStatusUpdate;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Order Status</Text>
      <Text style={styles.status}>{order?.status}</Text>
      <Text style={styles.estimated}>
        Estimated Delivery: {order?.estimatedDelivery}
      </Text>
    </View>
  );
};

// Hook personalizado para subscriptions
const useOrderTracking = (orderId) => {
  const [order, setOrder] = useState(null);
  
  const { data, loading, error } = useSubscription(ORDER_STATUS_UPDATE, {
    variables: { orderId },
    onSubscriptionData: ({ subscriptionData }) => {
      setOrder(subscriptionData.data.orderStatusUpdate);
    }
  });

  return { order, loading, error };
};
```

### 5. Performance y Caching

#### Optimización de Queries
```javascript
// Query optimization
const GET_USER_ORDERS = gql`
  query GetUserOrders($userId: ID!, $limit: Int, $offset: Int) {
    user(id: $userId) {
      id
      name
      orders(limit: $limit, offset: $offset) {
        id
        total
        status
        items {
          name
          price
        }
        restaurant {
          name
          rating
        }
      }
    }
  }
`;

// Pagination con fetchMore
const UserOrders = ({ userId }) => {
  const { data, loading, fetchMore } = useQuery(GET_USER_ORDERS, {
    variables: { userId, limit: 10, offset: 0 }
  });

  const loadMore = () => {
    fetchMore({
      variables: {
        offset: data.user.orders.length
      },
      updateQuery: (prev, { fetchMoreResult }) => {
        if (!fetchMoreResult) return prev;
        
        return {
          ...prev,
          user: {
            ...prev.user,
            orders: [...prev.user.orders, ...fetchMoreResult.user.orders]
          }
        };
      }
    });
  };

  return (
    <FlatList
      data={data?.user?.orders || []}
      renderItem={({ item }) => <OrderItem order={item} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
    />
  );
};
```

#### Cache Management
```javascript
// Cache policies
const cache = new InMemoryCache({
  typePolicies: {
    User: {
      fields: {
        orders: {
          merge(existing = [], incoming, { args }) {
            // Merge strategy para pagination
            if (args?.offset > 0) {
              return [...existing, ...incoming];
            }
            return incoming;
          }
        }
      }
    },
    
    Order: {
      fields: {
        items: {
          merge: true // Merge arrays
        }
      }
    }
  }
});

// Cache updates
const [createOrder] = useMutation(CREATE_ORDER, {
  update: (cache, { data }) => {
    const { createOrder } = data;
    
    // Actualizar cache de usuario
    cache.modify({
      id: cache.identify({ __typename: 'User', id: createOrder.userId }),
      fields: {
        orders: (existingOrders = []) => {
          return [...existingOrders, createOrder];
        }
      }
    });
  }
});

// Optimistic updates
const [updateOrderStatus] = useMutation(UPDATE_ORDER_STATUS, {
  optimisticResponse: {
    updateOrderStatus: {
      __typename: 'Order',
      id: orderId,
      status: newStatus,
      updatedAt: new Date().toISOString()
    }
  },
  update: (cache, { data }) => {
    cache.modify({
      id: cache.identify({ __typename: 'Order', id: orderId }),
      fields: {
        status: () => data.updateOrderStatus.status,
        updatedAt: () => data.updateOrderStatus.updatedAt
      }
    });
  }
});
```

## 🛠️ Implementación Práctica

### Sistema GraphQL Completo
```javascript
// App.js con Apollo Provider
import React from 'react';
import { ApolloProvider } from '@apollo/client';
import { client } from './apollo/client';
import AppNavigator from './navigation/AppNavigator';

const App = () => {
  return (
    <ApolloProvider client={client}>
      <AppNavigator />
    </ApolloProvider>
  );
};

export default App;

// GraphQL queries y mutations
export const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      phone
      profile {
        avatar
        bio
      }
      preferences {
        notifications
        theme
      }
    }
  }
`;

export const CREATE_ORDER = gql`
  mutation CreateOrder($input: CreateOrderInput!) {
    createOrder(input: $input) {
      id
      total
      status
      estimatedDelivery
      items {
        name
        price
        quantity
      }
    }
  }
`;

export const ORDER_STATUS_UPDATE = gql`
  subscription OrderStatusUpdate($orderId: ID!) {
    orderStatusUpdate(orderId: $orderId) {
      id
      status
      estimatedDelivery
      updatedAt
    }
  }
`;
```

## 📊 Métricas y KPIs

### GraphQL Performance
- **Query Response Time**: <200ms
- **Cache Hit Rate**: >80%
- **Subscription Latency**: <100ms
- **Error Rate**: <1%
- **Client Bundle Size**: Optimizado

## 🔧 Herramientas y Librerías

### GraphQL Tools
- **Apollo Server**: GraphQL server
- **Apollo Client**: GraphQL client
- **GraphQL Playground**: IDE para GraphQL
- **GraphQL Code Generator**: Code generation
- **Prisma**: Database ORM

### Development Tools
- **GraphQL Inspector**: Schema validation
- **GraphQL Metrics**: Performance monitoring
- **GraphQL Voyager**: Schema visualization

## 🎯 Proyecto Integrador

### API GraphQL para Delivery App
Implementar un sistema completo con:

1. **Schema Design** completo y type-safe
2. **Resolvers** optimizados con data sources
3. **Apollo Client** en React Native
4. **Subscriptions** para real-time updates
5. **Caching** estratégico y optimizado
6. **Error Handling** robusto
7. **Performance** optimizado

## 📚 Recursos Adicionales

### Documentación
- [GraphQL.org](https://graphql.org/)
- [Apollo Documentation](https://www.apollographql.com/docs/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)

### Cursos Recomendados
- GraphQL Fundamentals
- Apollo Client for React Native
- GraphQL Subscriptions
- GraphQL Performance Optimization

## ✅ Evaluación

### Criterios de Evaluación
- Diseño correcto de schema GraphQL
- Implementación de resolvers eficientes
- Configuración de Apollo Client
- Subscriptions funcionando correctamente
- Optimización de performance y caching

---

**Siguiente**: [Clase 3: Serverless y Cloud Functions](./clase_3_serverless_cloud_functions.md)
**Anterior**: [Clase 1: Arquitectura de Microservicios](./clase_1_arquitectura_microservicios.md)
