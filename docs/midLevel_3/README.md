# Módulo 6: APIs y Networking

## Navegación
- **Anterior**: [Módulo 5: Gestión de Estado Avanzada](./../midLevel_2/README.md)
- **Siguiente**: [Módulo 7: Almacenamiento Local](./../midLevel_4/README.md)
- **Índice**: [Índice Completo](./../INDICE_COMPLETO.md)
- **Navegación Rápida**: [Navegación Rápida](./../NAVEGACION_RAPIDA.md)

## Descripción
En este módulo aprenderás a integrar APIs externas en React Native, manejar peticiones HTTP, gestionar errores de red, y implementar patrones robustos para comunicación con servidores.

## Objetivos de Aprendizaje
- ✅ Comprender los fundamentos de las peticiones HTTP en React Native
- ✅ Implementar peticiones con Fetch API y Axios
- ✅ Manejar estados de carga, éxito y error
- ✅ Implementar interceptores y middleware para peticiones
- ✅ Gestionar autenticación y autorización en APIs
- ✅ Optimizar el rendimiento de las peticiones de red

## 📚 Clases del Módulo

### ✅ **Clase 1: Fetch API Básica** - [Ver Clase](./clase_1_fetch_api_basica.md)
- Fundamentos de Fetch API en React Native
- Implementación de peticiones HTTP básicas (GET, POST, PUT, DELETE)
- Manejo de respuestas y errores de manera efectiva
- Creación de servicios reutilizables para APIs
- Hooks personalizados para gestión de estado de APIs

### ✅ **Clase 2: Axios y Middleware** - [Ver Clase](./clase_2_axios_middleware.md)
- Ventajas de Axios sobre Fetch API
- Implementación de interceptores para requests y responses
- Middleware personalizado para logging y analytics
- Manejo de autenticación y autorización de manera elegante
- Retry logic y timeout handling

### ✅ **Clase 3: Manejo de Errores y Caché** - [Ver Clase](./clase_3_manejo_errores_cache.md)
- Estrategias robustas de manejo de errores
- Sistemas de caché inteligente para mejorar performance
- Manejo de diferentes tipos de errores de red y servidor
- Implementación de fallbacks y retry logic avanzado
- Optimización del rendimiento de las aplicaciones

### ✅ **Clase 4: Autenticación y Autorización** - [Ver Clase](./clase_4_autenticacion_autorizacion.md)
- Sistemas de autenticación robustos con JWT
- Manejo de diferentes tipos de tokens (JWT, OAuth, API Keys)
- Autorización basada en roles y permisos
- Gestión de sesiones y renovación automática de tokens
- Seguridad adicional (biometría, 2FA)

### ✅ **Clase 5: Optimización y Performance** - [Ver Clase](./clase_5_optimizacion_performance.md)
- Técnicas de optimización para React Native
- Optimización de re-renders con React.memo, useMemo y useCallback
- Implementación de virtualización para listas grandes
- Optimización de imágenes y assets
- Profiling y métricas de performance

## 🎯 Estado del Módulo: **COMPLETADO** ✅

## Contenido Teórico

### 1. Fetch API
Fetch es la API nativa de JavaScript para realizar peticiones HTTP.

#### Petición Básica
```javascript
// api/users.js
const API_BASE_URL = 'https://jsonplaceholder.typicode.com';

export const fetchUsers = async () => {
  try {
    const response = await fetch(`${API_BASE_URL}/users`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const users = await response.json();
    return users;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
};

export const createUser = async (userData) => {
  try {
    const response = await fetch(`${API_BASE_URL}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const newUser = await response.json();
    return newUser;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};
```

#### Manejo de Estados
```javascript
// hooks/useApi.js
import { useState, useCallback } from 'react';

export const useApi = (apiFunction) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await apiFunction(...args);
      setData(result);
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, [apiFunction]);

  return { data, loading, error, execute };
};
```

### 2. Axios
Axios es una librería popular que simplifica las peticiones HTTP con características adicionales.

#### Instalación y Configuración
```bash
npm install axios
```

```javascript
// api/axiosConfig.js
import axios from 'axios';

const API_BASE_URL = 'https://api.example.com';

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Interceptor de peticiones
apiClient.interceptors.request.use(
  (config) => {
    // Agregar token de autenticación
    const token = AsyncStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Interceptor de respuestas
apiClient.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error.response?.status === 401) {
      // Token expirado, redirigir al login
      // navigation.navigate('Login');
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

#### Servicios API
```javascript
// api/userService.js
import apiClient from './axiosConfig';

export const userService = {
  getUsers: async () => {
    const response = await apiClient.get('/users');
    return response.data;
  },
  
  getUserById: async (id) => {
    const response = await apiClient.get(`/users/${id}`);
    return response.data;
  },
  
  createUser: async (userData) => {
    const response = await apiClient.post('/users', userData);
    return response.data;
  },
  
  updateUser: async (id, userData) => {
    const response = await apiClient.put(`/users/${id}`, userData);
    return response.data;
  },
  
  deleteUser: async (id) => {
    const response = await apiClient.delete(`/users/${id}`);
    return response.data;
  },
};
```

### 3. Manejo de Errores Avanzado
Implementar un sistema robusto de manejo de errores.

```javascript
// utils/apiErrorHandler.js
export class ApiError extends Error {
  constructor(message, status, code) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.code = code;
  }
}

export const handleApiError = (error) => {
  if (error.response) {
    // Error de respuesta del servidor
    const { status, data } = error.response;
    
    switch (status) {
      case 400:
        return new ApiError('Datos inválidos', status, 'VALIDATION_ERROR');
      case 401:
        return new ApiError('No autorizado', status, 'UNAUTHORIZED');
      case 403:
        return new ApiError('Acceso denegado', status, 'FORBIDDEN');
      case 404:
        return new ApiError('Recurso no encontrado', status, 'NOT_FOUND');
      case 500:
        return new ApiError('Error del servidor', status, 'SERVER_ERROR');
      default:
        return new ApiError(data.message || 'Error desconocido', status, 'UNKNOWN');
    }
  } else if (error.request) {
    // Error de red
    return new ApiError('Error de conexión', 0, 'NETWORK_ERROR');
  } else {
    // Error de configuración
    return new ApiError('Error de configuración', 0, 'CONFIG_ERROR');
  }
};
```

### 4. Hooks Personalizados para APIs
Crear hooks reutilizables para manejar peticiones API.

```javascript
// hooks/useApiState.js
import { useState, useCallback } from 'react';

export const useApiState = (initialData = null) => {
  const [data, setData] = useState(initialData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const setLoadingState = useCallback((isLoading) => {
    setLoading(isLoading);
    if (isLoading) {
      setError(null);
    }
  }, []);

  const setErrorState = useCallback((errorMessage) => {
    setError(errorMessage);
    setLoading(false);
  }, []);

  const setDataState = useCallback((newData) => {
    setData(newData);
    setLoading(false);
    setError(null);
  }, []);

  const reset = useCallback(() => {
    setData(initialData);
    setLoading(false);
    setError(null);
  }, [initialData]);

  return {
    data,
    loading,
    error,
    setLoading: setLoadingState,
    setError: setErrorState,
    setData: setDataState,
    reset,
  };
};

// hooks/useApiCall.js
import { useCallback } from 'react';
import { useApiState } from './useApiState';
import { handleApiError } from '../utils/apiErrorHandler';

export const useApiCall = (apiFunction) => {
  const apiState = useApiState();

  const execute = useCallback(async (...args) => {
    apiState.setLoading(true);
    
    try {
      const result = await apiFunction(...args);
      apiState.setData(result);
      return result;
    } catch (error) {
      const apiError = handleApiError(error);
      apiState.setError(apiError.message);
      throw apiError;
    }
  }, [apiFunction, apiState]);

  return {
    ...apiState,
    execute,
  };
};
```

### 5. Cache y Optimización
Implementar estrategias de cache para mejorar el rendimiento.

```javascript
// utils/apiCache.js
class ApiCache {
  constructor() {
    this.cache = new Map();
    this.timeouts = new Map();
  }

  set(key, data, ttl = 5 * 60 * 1000) { // 5 minutos por defecto
    this.cache.set(key, data);
    
    // Limpiar cache después del TTL
    const timeout = setTimeout(() => {
      this.delete(key);
    }, ttl);
    
    this.timeouts.set(key, timeout);
  }

  get(key) {
    return this.cache.get(key);
  }

  has(key) {
    return this.cache.has(key);
  }

  delete(key) {
    this.cache.delete(key);
    const timeout = this.timeouts.get(key);
    if (timeout) {
      clearTimeout(timeout);
      this.timeouts.delete(key);
    }
  }

  clear() {
    this.cache.clear();
    this.timeouts.forEach(timeout => clearTimeout(timeout));
    this.timeouts.clear();
  }
}

export const apiCache = new ApiCache();

// hooks/useCachedApi.js
import { useCallback } from 'react';
import { useApiCall } from './useApiCall';
import { apiCache } from '../utils/apiCache';

export const useCachedApi = (apiFunction, cacheKey, ttl = 5 * 60 * 1000) => {
  const apiCall = useApiCall(apiFunction);

  const execute = useCallback(async (...args) => {
    const key = typeof cacheKey === 'function' ? cacheKey(...args) : cacheKey;
    
    // Verificar cache
    if (apiCache.has(key)) {
      const cachedData = apiCache.get(key);
      apiCall.setData(cachedData);
      return cachedData;
    }

    // Realizar petición
    const result = await apiCall.execute(...args);
    
    // Guardar en cache
    apiCache.set(key, result, ttl);
    
    return result;
  }, [apiFunction, cacheKey, ttl, apiCall]);

  return {
    ...apiCall,
    execute,
  };
};
```

### 6. Autenticación y Autorización
Implementar manejo de tokens y autenticación.

```javascript
// services/authService.js
import apiClient from '../api/axiosConfig';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const authService = {
  login: async (credentials) => {
    const response = await apiClient.post('/auth/login', credentials);
    const { token, user } = response.data;
    
    // Guardar token
    await AsyncStorage.setItem('authToken', token);
    await AsyncStorage.setItem('user', JSON.stringify(user));
    
    return { token, user };
  },
  
  logout: async () => {
    try {
      await apiClient.post('/auth/logout');
    } catch (error) {
      console.error('Error during logout:', error);
    } finally {
      // Limpiar datos locales
      await AsyncStorage.removeItem('authToken');
      await AsyncStorage.removeItem('user');
    }
  },
  
  refreshToken: async () => {
    const response = await apiClient.post('/auth/refresh');
    const { token } = response.data;
    
    await AsyncStorage.setItem('authToken', token);
    return token;
  },
  
  getCurrentUser: async () => {
    const userStr = await AsyncStorage.getItem('user');
    return userStr ? JSON.parse(userStr) : null;
  },
};
```

## Ejercicios Prácticos

### Ejercicio 1: API de Usuarios con Fetch
Crea un servicio completo para gestionar usuarios usando Fetch API.

```javascript
// services/userService.js
const API_BASE_URL = 'https://jsonplaceholder.typicode.com';

export const userService = {
  async getUsers() {
    const response = await fetch(`${API_BASE_URL}/users`);
    if (!response.ok) throw new Error('Error fetching users');
    return response.json();
  },
  
  async getUserById(id) {
    const response = await fetch(`${API_BASE_URL}/users/${id}`);
    if (!response.ok) throw new Error('User not found');
    return response.json();
  },
  
  async createUser(userData) {
    const response = await fetch(`${API_BASE_URL}/users`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData),
    });
    if (!response.ok) throw new Error('Error creating user');
    return response.json();
  },
  
  async updateUser(id, userData) {
    const response = await fetch(`${API_BASE_URL}/users/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData),
    });
    if (!response.ok) throw new Error('Error updating user');
    return response.json();
  },
  
  async deleteUser(id) {
    const response = await fetch(`${API_BASE_URL}/users/${id}`, {
      method: 'DELETE',
    });
    if (!response.ok) throw new Error('Error deleting user');
    return response.json();
  },
};
```

### Ejercicio 2: Hook Personalizado para Posts
Implementa un hook para gestionar posts con estados de carga y error.

```javascript
// hooks/usePosts.js
import { useState, useEffect, useCallback } from 'react';
import { userService } from '../services/userService';

export const usePosts = () => {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchPosts = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts');
      const data = await response.json();
      setPosts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, []);

  const createPost = useCallback(async (postData) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(postData),
      });
      const newPost = await response.json();
      setPosts(prev => [newPost, ...prev]);
      return newPost;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchPosts();
  }, [fetchPosts]);

  return {
    posts,
    loading,
    error,
    fetchPosts,
    createPost,
  };
};
```

### Ejercicio 3: Interceptor de Axios para Retry
Crea un interceptor que reintente peticiones fallidas automáticamente.

```javascript
// utils/retryInterceptor.js
import axios from 'axios';

const retryInterceptor = (maxRetries = 3, delay = 1000) => {
  return axios.interceptors.response.use(
    (response) => response,
    async (error) => {
      const { config } = error;
      
      if (!config || !config.retry) {
        config.retry = 0;
      }
      
      if (config.retry >= maxRetries) {
        return Promise.reject(error);
      }
      
      config.retry += 1;
      
      // Esperar antes de reintentar
      await new Promise(resolve => setTimeout(resolve, delay * config.retry));
      
      return axios(config);
    }
  );
};

export default retryInterceptor;
```

### Ejercicio 4: API con Cache Inteligente
Implementa un sistema de cache que invalide automáticamente datos obsoletos.

```javascript
// utils/smartCache.js
class SmartCache {
  constructor() {
    this.cache = new Map();
    this.timestamps = new Map();
  }

  set(key, data, ttl = 5 * 60 * 1000) {
    this.cache.set(key, data);
    this.timestamps.set(key, Date.now() + ttl);
  }

  get(key) {
    if (!this.cache.has(key)) return null;
    
    const timestamp = this.timestamps.get(key);
    if (Date.now() > timestamp) {
      this.delete(key);
      return null;
    }
    
    return this.cache.get(key);
  }

  delete(key) {
    this.cache.delete(key);
    this.timestamps.delete(key);
  }

  clear() {
    this.cache.clear();
    this.timestamps.clear();
  }

  // Invalidar cache basado en patrones
  invalidatePattern(pattern) {
    const regex = new RegExp(pattern);
    for (const key of this.cache.keys()) {
      if (regex.test(key)) {
        this.delete(key);
      }
    }
  }
}

export const smartCache = new SmartCache();
```

### Ejercicio 5: Hook para Paginación
Crea un hook que maneje paginación automáticamente.

```javascript
// hooks/usePagination.js
import { useState, useCallback } from 'react';

export const usePagination = (fetchFunction, pageSize = 10) => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    
    setLoading(true);
    setError(null);
    
    try {
      const newData = await fetchFunction(page, pageSize);
      
      if (newData.length < pageSize) {
        setHasMore(false);
      }
      
      setData(prev => [...prev, ...newData]);
      setPage(prev => prev + 1);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [fetchFunction, page, pageSize, loading, hasMore]);

  const refresh = useCallback(async () => {
    setData([]);
    setPage(1);
    setHasMore(true);
    setError(null);
    await loadMore();
  }, [loadMore]);

  return {
    data,
    loading,
    error,
    hasMore,
    loadMore,
    refresh,
  };
};
```

### Ejercicio 6: Servicio de Upload de Archivos
Implementa un servicio para subir archivos con progreso.

```javascript
// services/uploadService.js
import apiClient from '../api/axiosConfig';

export const uploadService = {
  uploadFile: async (file, onProgress) => {
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await apiClient.post('/upload', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
      onUploadProgress: (progressEvent) => {
        const percentCompleted = Math.round(
          (progressEvent.loaded * 100) / progressEvent.total
        );
        onProgress?.(percentCompleted);
      },
    });
    
    return response.data;
  },
  
  uploadMultipleFiles: async (files, onProgress) => {
    const formData = new FormData();
    files.forEach((file, index) => {
      formData.append(`files[${index}]`, file);
    });
    
    const response = await apiClient.post('/upload/multiple', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
      onUploadProgress: (progressEvent) => {
        const percentCompleted = Math.round(
          (progressEvent.loaded * 100) / progressEvent.total
        );
        onProgress?.(percentCompleted);
      },
    });
    
    return response.data;
  },
};
```

### Ejercicio 7: API con WebSocket
Implementa comunicación en tiempo real con WebSocket.

```javascript
// services/websocketService.js
class WebSocketService {
  constructor(url) {
    this.url = url;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.listeners = new Map();
  }

  connect() {
    try {
      this.ws = new WebSocket(this.url);
      
      this.ws.onopen = () => {
        console.log('WebSocket connected');
        this.reconnectAttempts = 0;
      };
      
      this.ws.onmessage = (event) => {
        const data = JSON.parse(event.data);
        this.notifyListeners(data);
      };
      
      this.ws.onclose = () => {
        console.log('WebSocket disconnected');
        this.attemptReconnect();
      };
      
      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error);
      };
    } catch (error) {
      console.error('Error connecting to WebSocket:', error);
    }
  }

  attemptReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      setTimeout(() => {
        console.log(`Attempting to reconnect... (${this.reconnectAttempts})`);
        this.connect();
      }, 1000 * this.reconnectAttempts);
    }
  }

  send(data) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    }
  }

  subscribe(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }

  unsubscribe(event, callback) {
    if (this.listeners.has(event)) {
      const callbacks = this.listeners.get(event);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  notifyListeners(data) {
    const { event, payload } = data;
    if (this.listeners.has(event)) {
      this.listeners.get(event).forEach(callback => callback(payload));
    }
  }

  disconnect() {
    if (this.ws) {
      this.ws.close();
    }
  }
}

export const chatWebSocket = new WebSocketService('wss://api.example.com/chat');
```

### Ejercicio 8: Hook para Polling
Crea un hook que realice polling automático de datos.

```javascript
// hooks/usePolling.js
import { useState, useEffect, useRef } from 'react';

export const usePolling = (fetchFunction, interval = 5000, enabled = true) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const intervalRef = useRef(null);

  const fetchData = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await fetchFunction();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (enabled) {
      fetchData(); // Fetch inicial
      
      intervalRef.current = setInterval(fetchData, interval);
      
      return () => {
        if (intervalRef.current) {
          clearInterval(intervalRef.current);
        }
      };
    }
  }, [fetchFunction, interval, enabled]);

  const stopPolling = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  const startPolling = () => {
    if (!intervalRef.current) {
      intervalRef.current = setInterval(fetchData, interval);
    }
  };

  return {
    data,
    loading,
    error,
    stopPolling,
    startPolling,
    refetch: fetchData,
  };
};
```

### Ejercicio 9: API con Offline Support
Implementa un sistema que funcione offline usando cache local.

```javascript
// services/offlineApiService.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import NetInfo from '@react-native-community/netinfo';

class OfflineApiService {
  constructor() {
    this.pendingRequests = [];
    this.isOnline = true;
    this.setupNetworkListener();
  }

  setupNetworkListener() {
    NetInfo.addEventListener(state => {
      const wasOffline = !this.isOnline;
      this.isOnline = state.isConnected;
      
      if (wasOffline && this.isOnline) {
        this.processPendingRequests();
      }
    });
  }

  async request(endpoint, options = {}) {
    const requestData = { endpoint, options, timestamp: Date.now() };
    
    if (!this.isOnline) {
      // Guardar petición para procesar cuando esté online
      this.pendingRequests.push(requestData);
      return this.getCachedData(endpoint);
    }
    
    try {
      const response = await fetch(endpoint, options);
      const data = await response.json();
      
      // Cachear respuesta
      await this.cacheData(endpoint, data);
      
      return data;
    } catch (error) {
      // Si falla, intentar con cache
      const cachedData = await this.getCachedData(endpoint);
      if (cachedData) {
        return cachedData;
      }
      throw error;
    }
  }

  async cacheData(key, data) {
    try {
      await AsyncStorage.setItem(`cache_${key}`, JSON.stringify({
        data,
        timestamp: Date.now(),
      }));
    } catch (error) {
      console.error('Error caching data:', error);
    }
  }

  async getCachedData(key) {
    try {
      const cached = await AsyncStorage.getItem(`cache_${key}`);
      if (cached) {
        const { data, timestamp } = JSON.parse(cached);
        const age = Date.now() - timestamp;
        
        // Cache válido por 1 hora
        if (age < 60 * 60 * 1000) {
          return data;
        }
      }
    } catch (error) {
      console.error('Error getting cached data:', error);
    }
    return null;
  }

  async processPendingRequests() {
    const requests = [...this.pendingRequests];
    this.pendingRequests = [];
    
    for (const request of requests) {
      try {
        await this.request(request.endpoint, request.options);
      } catch (error) {
        console.error('Error processing pending request:', error);
      }
    }
  }
}

export const offlineApi = new OfflineApiService();
```

### Ejercicio 10: API Rate Limiting
Implementa rate limiting para evitar exceder límites de API.

```javascript
// utils/rateLimiter.js
class RateLimiter {
  constructor(maxRequests = 100, timeWindow = 60000) { // 100 requests per minute
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindow;
    this.requests = [];
  }

  canMakeRequest() {
    const now = Date.now();
    
    // Limpiar requests antiguos
    this.requests = this.requests.filter(
      timestamp => now - timestamp < this.timeWindow
    );
    
    return this.requests.length < this.maxRequests;
  }

  recordRequest() {
    this.requests.push(Date.now());
  }

  getTimeUntilReset() {
    if (this.requests.length === 0) return 0;
    
    const oldestRequest = Math.min(...this.requests);
    return Math.max(0, this.timeWindow - (Date.now() - oldestRequest));
  }
}

// Hook para usar rate limiting
export const useRateLimitedApi = (apiFunction, maxRequests = 100, timeWindow = 60000) => {
  const rateLimiter = new RateLimiter(maxRequests, timeWindow);
  
  return async (...args) => {
    if (!rateLimiter.canMakeRequest()) {
      const waitTime = rateLimiter.getTimeUntilReset();
      throw new Error(`Rate limit exceeded. Try again in ${Math.ceil(waitTime / 1000)} seconds.`);
    }
    
    rateLimiter.recordRequest();
    return apiFunction(...args);
  };
};
```

## Proyecto Integrador: Social Media App

### Descripción
Desarrolla una aplicación de redes sociales que integre múltiples APIs y servicios:
- API de usuarios y autenticación
- API de posts y comentarios
- API de notificaciones en tiempo real
- API de mensajería
- API de búsqueda y filtros

### Estructura del Proyecto
```
src/
├── api/
│   ├── axiosConfig.js
│   ├── userService.js
│   ├── postService.js
│   └── notificationService.js
├── hooks/
│   ├── useApiCall.js
│   ├── usePagination.js
│   └── usePolling.js
├── services/
│   ├── authService.js
│   ├── uploadService.js
│   └── websocketService.js
├── utils/
│   ├── apiErrorHandler.js
│   ├── apiCache.js
│   └── rateLimiter.js
└── screens/
    ├── FeedScreen.js
    ├── ProfileScreen.js
    ├── ChatScreen.js
    └── NotificationScreen.js
```

### Funcionalidades Clave
1. **Feed de Posts**: Paginación infinita con cache
2. **Chat en Tiempo Real**: WebSocket para mensajería
3. **Notificaciones**: Polling para notificaciones push
4. **Upload de Imágenes**: Subida con progreso
5. **Búsqueda**: API con rate limiting
6. **Modo Offline**: Cache local para funcionalidad offline

### Criterios de Evaluación
- Implementación correcta de múltiples APIs
- Manejo robusto de errores y estados de carga
- Optimización de rendimiento con cache
- Funcionalidad offline
- Código limpio y bien estructurado
- Documentación de APIs

---

**Siguiente módulo**: [Módulo 7: Almacenamiento Local](./../midLevel_4/README.md)
