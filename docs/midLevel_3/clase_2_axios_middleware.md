# Clase 2: Axios y Middleware

## Navegación
- **Anterior**: [Clase 1: Fetch API Básica](./clase_1_fetch_api_basica.md)
- **Siguiente**: [Clase 3: Manejo de Errores y Caché](./clase_3_manejo_errores_cache.md)
- **Módulo**: [Módulo 6: APIs y Networking](./README.md)
- **Índice**: [Índice Completo](./../INDICE_COMPLETO.md)
- **Navegación Rápida**: [Navegación Rápida](./../NAVEGACION_RAPIDA.md)

## 🎯 Objetivos de la Clase
- Comprender las ventajas de Axios sobre Fetch API
- Implementar interceptores para requests y responses
- Crear middleware personalizado para logging y analytics
- Manejar autenticación y autorización de manera elegante
- Implementar retry logic y timeout handling

## 📚 Contenido Teórico

### 1. ¿Qué es Axios?

**Axios** es una librería popular de JavaScript que simplifica las peticiones HTTP. Proporciona una API más intuitiva que Fetch API y incluye características avanzadas como interceptores, cancelación de peticiones y transformación automática de datos.

#### Ventajas sobre Fetch API:
- **Transformación automática**: JSON se parsea automáticamente
- **Interceptores**: Modificar requests/responses antes/después
- **Cancelación**: Cancelar peticiones en curso
- **Timeout**: Configurar timeouts automáticamente
- **Mejor manejo de errores**: Errores HTTP se capturan automáticamente
- **Soporte para navegadores antiguos**: Polyfills incluidos

### 2. Instalación y Configuración

#### Instalación
```bash
npm install axios
# o
yarn add axios
```

#### Configuración Básica
```javascript
// services/axiosConfig.js
import axios from 'axios';

// Crear instancia base de axios
const api = axios.create({
  baseURL: 'https://jsonplaceholder.typicode.com',
  timeout: 10000, // 10 segundos
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
});

export default api;
```

### 3. Métodos HTTP con Axios

#### GET - Obtener Datos
```javascript
// GET básico
const getUsers = async () => {
  try {
    const response = await api.get('/users');
    return response.data; // Axios automáticamente extrae .data
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
};

// GET con parámetros
const getUsersWithParams = async (page = 1, limit = 10) => {
  try {
    const response = await api.get('/users', {
      params: {
        _page: page,
        _limit: limit,
      },
    });
    return response.data;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
};
```

#### POST - Crear Datos
```javascript
// POST con datos
const createUser = async (userData) => {
  try {
    const response = await api.post('/users', userData);
    return response.data;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};

// POST con headers personalizados
const createUserWithCustomHeaders = async (userData, token) => {
  try {
    const response = await api.post('/users', userData, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'X-Custom-Header': 'custom-value',
      },
    });
    return response.data;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};
```

#### PUT/PATCH - Actualizar Datos
```javascript
// PUT para actualización completa
const updateUser = async (userId, userData) => {
  try {
    const response = await api.put(`/users/${userId}`, userData);
    return response.data;
  } catch (error) {
    console.error('Error updating user:', error);
    throw error;
  }
};

// PATCH para actualización parcial
const updateUserPartial = async (userId, partialData) => {
  try {
    const response = await api.patch(`/users/${userId}`, partialData);
    return response.data;
  } catch (error) {
    console.error('Error updating user:', error);
    throw error;
  }
};
```

#### DELETE - Eliminar Datos
```javascript
// DELETE básico
const deleteUser = async (userId) => {
  try {
    await api.delete(`/users/${userId}`);
    return true; // Usuario eliminado exitosamente
  } catch (error) {
    console.error('Error deleting user:', error);
    throw error;
  }
};

// DELETE con confirmación
const deleteUserWithConfirmation = async (userId, reason) => {
  try {
    await api.delete(`/users/${userId}`, {
      data: { reason }, // Axios permite enviar datos en DELETE
    });
    return true;
  } catch (error) {
    console.error('Error deleting user:', error);
    throw error;
  }
};
```

### 4. Interceptores de Axios

Los **interceptores** son funciones que se ejecutan antes de enviar una petición o después de recibir una respuesta. Permiten modificar, validar o transformar datos de manera centralizada.

#### Interceptor de Request
```javascript
// Interceptor para requests
api.interceptors.request.use(
  (config) => {
    // Modificar la configuración antes de enviar
    console.log('Request being sent:', config);
    
    // Agregar token de autenticación
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Agregar timestamp
    config.metadata = { startTime: new Date() };
    
    return config;
  },
  (error) => {
    // Manejar errores en la configuración
    console.error('Request interceptor error:', error);
    return Promise.reject(error);
  }
);
```

#### Interceptor de Response
```javascript
// Interceptor para responses
api.interceptors.response.use(
  (response) => {
    // Modificar la respuesta antes de entregarla
    console.log('Response received:', response);
    
    // Calcular tiempo de respuesta
    const endTime = new Date();
    const startTime = response.config.metadata?.startTime;
    if (startTime) {
      const duration = endTime - startTime;
      console.log(`Request took ${duration}ms`);
    }
    
    return response;
  },
  (error) => {
    // Manejar errores de respuesta
    console.error('Response interceptor error:', error);
    
    // Manejar errores de autenticación
    if (error.response?.status === 401) {
      // Redirigir a login o refrescar token
      handleUnauthorized();
    }
    
    // Manejar errores de red
    if (error.code === 'NETWORK_ERROR') {
      console.error('Network error - check your connection');
    }
    
    return Promise.reject(error);
  }
);
```

### 5. Middleware Personalizado

#### Middleware de Logging
```javascript
// middleware/loggingMiddleware.js
class LoggingMiddleware {
  constructor() {
    this.logs = [];
  }
  
  // Middleware para logging de requests
  logRequest(config) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      type: 'REQUEST',
      method: config.method?.toUpperCase(),
      url: config.url,
      headers: config.headers,
      data: config.data,
    };
    
    this.logs.push(logEntry);
    console.log('🌐 API Request:', logEntry);
    
    return config;
  }
  
  // Middleware para logging de responses
  logResponse(response) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      type: 'RESPONSE',
      status: response.status,
      statusText: response.statusText,
      url: response.config.url,
      data: response.data,
      duration: this.calculateDuration(response.config.metadata?.startTime),
    };
    
    this.logs.push(logEntry);
    console.log('✅ API Response:', logEntry);
    
    return response;
  }
  
  // Middleware para logging de errores
  logError(error) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      type: 'ERROR',
      message: error.message,
      status: error.response?.status,
      url: error.config?.url,
      data: error.response?.data,
    };
    
    this.logs.push(logEntry);
    console.error('❌ API Error:', logEntry);
    
    return Promise.reject(error);
  }
  
  // Calcular duración de la petición
  calculateDuration(startTime) {
    if (!startTime) return null;
    return new Date() - startTime;
  }
  
  // Obtener logs
  getLogs() {
    return this.logs;
  }
  
  // Limpiar logs
  clearLogs() {
    this.logs = [];
  }
}

export default LoggingMiddleware;
```

#### Middleware de Analytics
```javascript
// middleware/analyticsMiddleware.js
class AnalyticsMiddleware {
  constructor(analyticsService) {
    this.analyticsService = analyticsService;
  }
  
  // Trackear peticiones exitosas
  trackSuccess(response) {
    const eventData = {
      event: 'api_success',
      endpoint: response.config.url,
      method: response.config.method,
      status: response.status,
      duration: this.calculateDuration(response.config.metadata?.startTime),
      timestamp: new Date().toISOString(),
    };
    
    this.analyticsService.track(eventData);
    return response;
  }
  
  // Trackear errores
  trackError(error) {
    const eventData = {
      event: 'api_error',
      endpoint: error.config?.url,
      method: error.config?.method,
      status: error.response?.status,
      message: error.message,
      timestamp: new Date().toISOString(),
    };
    
    this.analyticsService.track(eventData);
    return Promise.reject(error);
  }
  
  // Trackear performance
  trackPerformance(response) {
    const duration = this.calculateDuration(response.config.metadata?.startTime);
    
    if (duration > 5000) { // Más de 5 segundos
      this.analyticsService.track({
        event: 'api_slow_response',
        endpoint: response.config.url,
        duration,
        timestamp: new Date().toISOString(),
      });
    }
    
    return response;
  }
  
  calculateDuration(startTime) {
    if (!startTime) return null;
    return new Date() - startTime;
  }
}

export default AnalyticsMiddleware;
```

### 6. Servicio de API con Middleware

#### Servicio Base con Middleware
```javascript
// services/ApiServiceWithMiddleware.js
import axios from 'axios';
import LoggingMiddleware from '../middleware/loggingMiddleware';
import AnalyticsMiddleware from '../middleware/analyticsMiddleware';

class ApiServiceWithMiddleware {
  constructor(baseURL, analyticsService = null) {
    // Crear instancia de axios
    this.api = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
    });
    
    // Inicializar middleware
    this.loggingMiddleware = new LoggingMiddleware();
    this.analyticsMiddleware = analyticsService ? new AnalyticsMiddleware(analyticsService) : null;
    
    // Configurar interceptores
    this.setupInterceptors();
  }
  
  // Configurar interceptores
  setupInterceptors() {
    // Request interceptor
    this.api.interceptors.request.use(
      (config) => {
        // Agregar timestamp para calcular duración
        config.metadata = { startTime: new Date() };
        
        // Aplicar middleware de logging
        return this.loggingMiddleware.logRequest(config);
      },
      (error) => {
        return Promise.reject(error);
      }
    );
    
    // Response interceptor
    this.api.interceptors.response.use(
      (response) => {
        // Aplicar middleware de logging
        response = this.loggingMiddleware.logResponse(response);
        
        // Aplicar middleware de analytics si está disponible
        if (this.analyticsMiddleware) {
          response = this.analyticsMiddleware.trackSuccess(response);
          response = this.analyticsMiddleware.trackPerformance(response);
        }
        
        return response;
      },
      (error) => {
        // Aplicar middleware de logging
        error = this.loggingMiddleware.logError(error);
        
        // Aplicar middleware de analytics si está disponible
        if (this.analyticsMiddleware) {
          error = this.analyticsMiddleware.trackError(error);
        }
        
        return Promise.reject(error);
      }
    );
  }
  
  // Métodos HTTP
  async get(endpoint, config = {}) {
    return this.api.get(endpoint, config);
  }
  
  async post(endpoint, data, config = {}) {
    return this.api.post(endpoint, data, config);
  }
  
  async put(endpoint, data, config = {}) {
    return this.api.put(endpoint, data, config);
  }
  
  async patch(endpoint, data, config = {}) {
    return this.api.patch(endpoint, data, config);
  }
  
  async delete(endpoint, config = {}) {
    return this.api.delete(endpoint, config);
  }
  
  // Obtener logs
  getLogs() {
    return this.loggingMiddleware.getLogs();
  }
  
  // Limpiar logs
  clearLogs() {
    this.loggingMiddleware.clearLogs();
  }
}

export default ApiServiceWithMiddleware;
```

### 7. Manejo de Autenticación

#### Middleware de Autenticación
```javascript
// middleware/authMiddleware.js
class AuthMiddleware {
  constructor(tokenService) {
    this.tokenService = tokenService;
  }
  
  // Agregar token a requests
  addAuthToken(config) {
    const token = this.tokenService.getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  }
  
  // Refrescar token si expiró
  async handleTokenExpiry(error) {
    if (error.response?.status === 401) {
      try {
        const newToken = await this.tokenService.refreshToken();
        if (newToken) {
          // Reintentar la petición original con el nuevo token
          const originalRequest = error.config;
          originalRequest.headers.Authorization = `Bearer ${newToken}`;
          return axios(originalRequest);
        }
      } catch (refreshError) {
        // Token de refresh también expiró, redirigir a login
        this.tokenService.clearTokens();
        window.location.href = '/login';
      }
    }
    
    return Promise.reject(error);
  }
  
  // Verificar si el token está próximo a expirar
  shouldRefreshToken(token) {
    if (!token) return false;
    
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      const expirationTime = payload.exp * 1000; // Convertir a milisegundos
      const currentTime = Date.now();
      const timeUntilExpiry = expirationTime - currentTime;
      
      // Refrescar si expira en menos de 5 minutos
      return timeUntilExpiry < 5 * 60 * 1000;
    } catch (error) {
      return false;
    }
  }
}

export default AuthMiddleware;
```

#### Servicio de Token
```javascript
// services/tokenService.js
class TokenService {
  constructor() {
    this.accessTokenKey = 'accessToken';
    this.refreshTokenKey = 'refreshToken';
  }
  
  // Obtener token de acceso
  getToken() {
    return localStorage.getItem(this.accessTokenKey);
  }
  
  // Obtener token de refresh
  getRefreshToken() {
    return localStorage.getItem(this.refreshTokenKey);
  }
  
  // Guardar tokens
  setTokens(accessToken, refreshToken) {
    localStorage.setItem(this.accessTokenKey, accessToken);
    localStorage.setItem(this.refreshTokenKey, refreshToken);
  }
  
  // Limpiar tokens
  clearTokens() {
    localStorage.removeItem(this.accessTokenKey);
    localStorage.removeItem(this.refreshTokenKey);
  }
  
  // Refrescar token
  async refreshToken() {
    try {
      const refreshToken = this.getRefreshToken();
      if (!refreshToken) {
        throw new Error('No refresh token available');
      }
      
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ refreshToken }),
      });
      
      if (!response.ok) {
        throw new Error('Failed to refresh token');
      }
      
      const { accessToken, newRefreshToken } = await response.json();
      this.setTokens(accessToken, newRefreshToken);
      
      return accessToken;
    } catch (error) {
      console.error('Error refreshing token:', error);
      this.clearTokens();
      throw error;
    }
  }
  
  // Verificar si el usuario está autenticado
  isAuthenticated() {
    const token = this.getToken();
    if (!token) return false;
    
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      const currentTime = Date.now() / 1000;
      return payload.exp > currentTime;
    } catch (error) {
      return false;
    }
  }
}

export default TokenService;
```

### 8. Hook Personalizado con Axios

#### Hook useAxios
```javascript
// hooks/useAxios.js
import { useState, useCallback, useRef } from 'react';
import api from '../services/axiosConfig';

export const useAxios = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const abortControllerRef = useRef(null);

  const execute = useCallback(async (config) => {
    // Cancelar petición anterior si existe
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    // Crear nuevo abort controller
    abortControllerRef.current = new AbortController();
    
    setLoading(true);
    setError(null);
    
    try {
      const response = await api({
        ...config,
        signal: abortControllerRef.current.signal,
      });
      
      setData(response.data);
      return response.data;
    } catch (err) {
      if (err.name === 'CanceledError') {
        console.log('Request was canceled');
        return;
      }
      
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  const cancel = useCallback(() => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
  }, []);

  return { data, loading, error, execute, cancel };
};
```

#### Uso del Hook
```javascript
// Componente usando useAxios
import React from 'react';
import { View, Text, Button, ActivityIndicator } from 'react-native';
import { useAxios } from '../hooks/useAxios';

const UserList = () => {
  const { data: users, loading, error, execute: fetchUsers, cancel } = useAxios();

  React.useEffect(() => {
    fetchUsers({
      method: 'GET',
      url: '/users',
    });
    
    // Cleanup: cancelar petición al desmontar
    return () => cancel();
  }, [fetchUsers, cancel]);

  const handleRefresh = () => {
    fetchUsers({
      method: 'GET',
      url: '/users',
    });
  };

  if (loading) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" color="#0000ff" />
        <Text>Cargando usuarios...</Text>
        <Button title="Cancelar" onPress={cancel} />
      </View>
    );
  }

  if (error) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text style={{ color: 'red' }}>Error: {error}</Text>
        <Button title="Reintentar" onPress={handleRefresh} />
      </View>
    );
  }

  return (
    <View style={{ flex: 1, padding: 20 }}>
      <Text style={{ fontSize: 24, fontWeight: 'bold', marginBottom: 20 }}>
        Lista de Usuarios
      </Text>
      {users?.map(user => (
        <View key={user.id} style={{ marginBottom: 10, padding: 10, backgroundColor: '#f0f0f0' }}>
          <Text style={{ fontWeight: 'bold' }}>{user.name}</Text>
          <Text>{user.email}</Text>
        </View>
      ))}
      <Button title="Actualizar" onPress={handleRefresh} />
    </View>
  );
};

export default UserList;
```

## 🧪 Ejercicios Prácticos

### Ejercicio 1: Middleware de Caché
Crea un middleware que implemente caché básico para respuestas GET:

```javascript
// El middleware debe:
// - Cachear respuestas GET por URL
// - Configurar tiempo de expiración
// - Invalidar caché en POST/PUT/DELETE
// - Permitir limpiar caché manualmente
```

### Ejercicio 2: Middleware de Retry
Implementa un middleware que reintente peticiones fallidas:

```javascript
// El middleware debe:
// - Reintentar peticiones con error 5xx
// - Implementar backoff exponencial
// - Limitar número máximo de reintentos
// - Loggear intentos de reintento
```

### Ejercicio 3: Middleware de Rate Limiting
Crea un middleware que limite la frecuencia de peticiones:

```javascript
// El middleware debe:
// - Limitar peticiones por endpoint
// - Implementar cola de peticiones
// - Rechazar peticiones que excedan el límite
// - Permitir configuración por endpoint
```

## 📝 Resumen de la Clase

### **Conceptos Clave Aprendidos:**
1. **Axios**: Librería avanzada para peticiones HTTP con características superiores a Fetch
2. **Interceptores**: Modificar requests/responses de manera centralizada
3. **Middleware Personalizado**: Logging, analytics y autenticación
4. **Manejo de Autenticación**: Tokens, refresh automático y manejo de expiración
5. **Cancelación de Peticiones**: AbortController para cancelar peticiones en curso

### **Habilidades Desarrolladas:**
- Implementar peticiones HTTP avanzadas con Axios
- Crear middleware personalizado para diferentes propósitos
- Manejar autenticación de manera robusta
- Implementar logging y analytics para APIs
- Gestionar cancelación y timeouts de peticiones

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Manejo de Errores y Caché**, que nos permitirá:
- Implementar estrategias robustas de manejo de errores
- Crear sistemas de caché para mejorar performance
- Manejar diferentes tipos de errores de red
- Implementar fallbacks y retry logic avanzado

---

**💡 Consejo**: Experimenta creando diferentes tipos de middleware para entender cómo se pueden combinar y extender. Los interceptores de Axios son muy poderosos y pueden resolver muchos problemas comunes en el desarrollo de APIs.
