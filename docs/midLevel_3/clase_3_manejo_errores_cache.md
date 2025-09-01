# Clase 3: Manejo de Errores y Caché

## Navegación
- **Anterior**: [Clase 2: Axios y Middleware](./clase_2_axios_middleware.md)
- **Siguiente**: [Clase 4: Autenticación y Autorización](./clase_4_autenticacion_autorizacion.md)
- **Módulo**: [Módulo 6: APIs y Networking](./README.md)
- **Índice**: [Índice Completo](./../INDICE_COMPLETO.md)
- **Navegación Rápida**: [Navegación Rápida](./../NAVEGACION_RAPIDA.md)

## 🎯 Objetivos de la Clase
- Implementar estrategias robustas de manejo de errores
- Crear sistemas de caché para mejorar performance
- Manejar diferentes tipos de errores de red y servidor
- Implementar fallbacks y retry logic avanzado
- Optimizar el rendimiento de las aplicaciones con caché inteligente

## 📚 Contenido Teórico

### 1. Tipos de Errores en APIs

#### Errores de Red
Los **errores de red** ocurren cuando hay problemas de conectividad o infraestructura:

```javascript
// Errores comunes de red
const networkErrors = {
  NETWORK_ERROR: 'No hay conexión a internet',
  TIMEOUT_ERROR: 'La petición tardó demasiado',
  CONNECTION_REFUSED: 'El servidor rechazó la conexión',
  DNS_ERROR: 'No se pudo resolver el dominio',
  SSL_ERROR: 'Error en el certificado SSL',
};
```

#### Errores HTTP
Los **errores HTTP** son respuestas del servidor con códigos de estado específicos:

```javascript
// Categorías de errores HTTP
const httpErrorCategories = {
  // Errores del cliente (4xx)
  CLIENT_ERRORS: {
    400: 'Bad Request - Datos inválidos',
    401: 'Unauthorized - No autorizado',
    403: 'Forbidden - Acceso prohibido',
    404: 'Not Found - Recurso no encontrado',
    409: 'Conflict - Conflicto de datos',
    422: 'Unprocessable Entity - Entidad no procesable',
    429: 'Too Many Requests - Demasiadas peticiones',
  },
  
  // Errores del servidor (5xx)
  SERVER_ERRORS: {
    500: 'Internal Server Error - Error interno del servidor',
    502: 'Bad Gateway - Gateway incorrecto',
    503: 'Service Unavailable - Servicio no disponible',
    504: 'Gateway Timeout - Timeout del gateway',
  },
};
```

#### Errores de Aplicación
Los **errores de aplicación** son errores específicos del negocio:

```javascript
// Errores de aplicación comunes
const applicationErrors = {
  VALIDATION_ERROR: 'Error de validación de datos',
  AUTHENTICATION_ERROR: 'Error de autenticación',
  AUTHORIZATION_ERROR: 'Error de autorización',
  BUSINESS_LOGIC_ERROR: 'Error en la lógica de negocio',
  RESOURCE_NOT_FOUND: 'Recurso no encontrado',
  DUPLICATE_RESOURCE: 'Recurso duplicado',
};
```

### 2. Estrategias de Manejo de Errores

#### Clasificación de Errores
```javascript
// utils/errorClassifier.js
class ErrorClassifier {
  // Clasificar error por tipo
  static classifyError(error) {
    if (error.code === 'NETWORK_ERROR') {
      return {
        type: 'NETWORK',
        severity: 'HIGH',
        retryable: true,
        userMessage: 'Error de conexión. Verifica tu internet.',
      };
    }
    
    if (error.response?.status) {
      return this.classifyHttpError(error.response.status);
    }
    
    if (error.message) {
      return this.classifyApplicationError(error.message);
    }
    
    return {
      type: 'UNKNOWN',
      severity: 'MEDIUM',
      retryable: false,
      userMessage: 'Ocurrió un error inesperado.',
    };
  }
  
  // Clasificar errores HTTP
  static classifyHttpError(status) {
    if (status >= 400 && status < 500) {
      return {
        type: 'CLIENT',
        severity: 'MEDIUM',
        retryable: false,
        userMessage: this.getClientErrorMessage(status),
      };
    }
    
    if (status >= 500) {
      return {
        type: 'SERVER',
        severity: 'HIGH',
        retryable: true,
        userMessage: 'Error del servidor. Intenta más tarde.',
      };
    }
    
    return {
      type: 'UNKNOWN',
      severity: 'LOW',
      retryable: false,
      userMessage: 'Respuesta inesperada del servidor.',
    };
  }
  
  // Clasificar errores de aplicación
  static classifyApplicationError(message) {
    if (message.includes('validation')) {
      return {
        type: 'VALIDATION',
        severity: 'LOW',
        retryable: false,
        userMessage: 'Datos inválidos. Verifica la información.',
      };
    }
    
    if (message.includes('auth') || message.includes('token')) {
      return {
        type: 'AUTHENTICATION',
        severity: 'HIGH',
        retryable: false,
        userMessage: 'Sesión expirada. Inicia sesión nuevamente.',
      };
    }
    
    return {
      type: 'APPLICATION',
      severity: 'MEDIUM',
      retryable: false,
      userMessage: 'Error en la aplicación.',
    };
  }
  
  // Obtener mensaje de error del cliente
  static getClientErrorMessage(status) {
    const messages = {
      400: 'Datos inválidos enviados.',
      401: 'No tienes autorización para acceder.',
      403: 'Acceso prohibido a este recurso.',
      404: 'El recurso solicitado no existe.',
      409: 'Conflicto con datos existentes.',
      422: 'Datos no procesables.',
      429: 'Demasiadas peticiones. Espera un momento.',
    };
    
    return messages[status] || 'Error del cliente.';
  }
}

export default ErrorClassifier;
```

#### Manejo Centralizado de Errores
```javascript
// services/errorHandlerService.js
import ErrorClassifier from '../utils/errorClassifier';

class ErrorHandlerService {
  constructor() {
    this.errorLogs = [];
    this.errorCallbacks = new Map();
  }
  
  // Registrar callback para tipos de error específicos
  registerErrorCallback(errorType, callback) {
    this.errorCallbacks.set(errorType, callback);
  }
  
  // Manejar error de manera centralizada
  async handleError(error, context = {}) {
    // Clasificar el error
    const errorInfo = ErrorClassifier.classifyError(error);
    
    // Agregar contexto
    const enrichedError = {
      ...errorInfo,
      originalError: error,
      context,
      timestamp: new Date().toISOString(),
      stack: error.stack,
    };
    
    // Loggear el error
    this.logError(enrichedError);
    
    // Ejecutar callback específico si existe
    const callback = this.errorCallbacks.get(errorInfo.type);
    if (callback) {
      try {
        await callback(enrichedError);
      } catch (callbackError) {
        console.error('Error in error callback:', callbackError);
      }
    }
    
    // Retornar información del error para el componente
    return enrichedError;
  }
  
  // Loggear error
  logError(errorInfo) {
    this.errorLogs.push(errorInfo);
    
    // Loggear en consola
    console.error('🚨 Error handled:', {
      type: errorInfo.type,
      severity: errorInfo.severity,
      message: errorInfo.userMessage,
      timestamp: errorInfo.timestamp,
      context: errorInfo.context,
    });
    
    // Enviar a servicio de logging si está disponible
    if (this.loggingService) {
      this.loggingService.logError(errorInfo);
    }
  }
  
  // Obtener logs de errores
  getErrorLogs() {
    return this.errorLogs;
  }
  
  // Limpiar logs
  clearErrorLogs() {
    this.errorLogs = [];
  }
  
  // Configurar servicio de logging
  setLoggingService(loggingService) {
    this.loggingService = loggingService;
  }
}

export default new ErrorHandlerService();
```

### 3. Sistema de Caché Inteligente

#### Clase Base de Caché
```javascript
// services/cacheService.js
class CacheService {
  constructor() {
    this.cache = new Map();
    this.maxSize = 100; // Máximo número de elementos en caché
    this.defaultTTL = 5 * 60 * 1000; // 5 minutos por defecto
  }
  
  // Generar clave de caché
  generateKey(url, params = {}) {
    const paramsString = JSON.stringify(params);
    return `${url}:${paramsString}`;
  }
  
  // Obtener elemento del caché
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) {
      return null;
    }
    
    // Verificar si expiró
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    // Actualizar último acceso
    item.lastAccessed = Date.now();
    return item.data;
  }
  
  // Guardar elemento en caché
  set(key, data, ttl = this.defaultTTL) {
    // Limpiar caché si está lleno
    if (this.cache.size >= this.maxSize) {
      this.cleanup();
    }
    
    const item = {
      data,
      createdAt: Date.now(),
      lastAccessed: Date.now(),
      expiresAt: Date.now() + ttl,
      accessCount: 0,
    };
    
    this.cache.set(key, item);
  }
  
  // Verificar si existe en caché
  has(key) {
    return this.cache.has(key) && Date.now() <= this.cache.get(key).expiresAt;
  }
  
  // Eliminar elemento del caché
  delete(key) {
    return this.cache.delete(key);
  }
  
  // Limpiar caché expirado
  cleanup() {
    const now = Date.now();
    const expiredKeys = [];
    
    for (const [key, item] of this.cache.entries()) {
      if (now > item.expiresAt) {
        expiredKeys.push(key);
      }
    }
    
    expiredKeys.forEach(key => this.cache.delete(key));
    
    // Si aún está lleno, eliminar elementos menos usados
    if (this.cache.size >= this.maxSize) {
      this.removeLeastUsed();
    }
  }
  
  // Eliminar elementos menos usados
  removeLeastUsed() {
    const entries = Array.from(this.cache.entries());
    entries.sort((a, b) => {
      // Priorizar por último acceso y número de accesos
      const scoreA = a[1].lastAccessed + (a[1].accessCount * 1000);
      const scoreB = b[1].lastAccessed + (b[1].accessCount * 1000);
      return scoreA - scoreB;
    });
    
    // Eliminar el 20% menos usado
    const toRemove = Math.ceil(this.cache.size * 0.2);
    entries.slice(0, toRemove).forEach(([key]) => {
      this.cache.delete(key);
    });
  }
  
  // Obtener estadísticas del caché
  getStats() {
    const now = Date.now();
    let expiredCount = 0;
    let totalAccessCount = 0;
    
    for (const item of this.cache.values()) {
      if (now > item.expiresAt) {
        expiredCount++;
      }
      totalAccessCount += item.accessCount;
    }
    
    return {
      totalItems: this.cache.size,
      expiredItems: expiredCount,
      validItems: this.cache.size - expiredCount,
      totalAccessCount,
      averageAccessCount: this.cache.size > 0 ? totalAccessCount / this.cache.size : 0,
    };
  }
  
  // Limpiar todo el caché
  clear() {
    this.cache.clear();
  }
}

export default CacheService;
```

#### Middleware de Caché para Axios
```javascript
// middleware/cacheMiddleware.js
import CacheService from '../services/cacheService';

class CacheMiddleware {
  constructor(cacheService = new CacheService()) {
    this.cache = cacheService;
    this.cacheableMethods = ['GET'];
    this.cacheableStatuses = [200, 201];
  }
  
  // Middleware para requests
  handleRequest(config) {
    // Solo cachear métodos GET
    if (!this.cacheableMethods.includes(config.method?.toUpperCase())) {
      return config;
    }
    
    // Generar clave de caché
    const cacheKey = this.cache.generateKey(config.url, config.params);
    
    // Verificar si existe en caché
    if (this.cache.has(cacheKey)) {
      const cachedData = this.cache.get(cacheKey);
      
      // Marcar como respuesta del caché
      config.cacheHit = true;
      config.cachedData = cachedData;
      
      // Agregar header para indicar que viene del caché
      config.headers = {
        ...config.headers,
        'X-Cache': 'HIT',
        'X-Cache-Key': cacheKey,
      };
    }
    
    return config;
  }
  
  // Middleware para responses
  handleResponse(response) {
    // Solo cachear respuestas exitosas
    if (!this.cacheableStatuses.includes(response.status)) {
      return response;
    }
    
    // Solo cachear métodos GET
    if (!this.cacheableMethods.includes(response.config.method?.toUpperCase())) {
      return response;
    }
  
    // Generar clave de caché
    const cacheKey = this.cache.generateKey(response.config.url, response.config.params);
    
    // Guardar en caché
    this.cache.set(cacheKey, response.data);
    
    // Agregar headers de caché
    response.headers['X-Cache'] = 'MISS';
    response.headers['X-Cache-Key'] = cacheKey;
    
    return response;
  }
  
  // Middleware para errores
  handleError(error) {
    // Si es un error de red, intentar usar caché
    if (error.code === 'NETWORK_ERROR' && error.config) {
      const cacheKey = this.cache.generateKey(error.config.url, error.config.params);
      
      if (this.cache.has(cacheKey)) {
        const cachedData = this.cache.get(cacheKey);
        
        // Crear respuesta simulada del caché
        const cachedResponse = {
          data: cachedData,
          status: 200,
          statusText: 'OK',
          headers: {
            'X-Cache': 'STALE',
            'X-Cache-Key': cacheKey,
          },
          config: error.config,
          fromCache: true,
        };
        
        // Retornar respuesta del caché en lugar del error
        return Promise.resolve(cachedResponse);
      }
    }
    
    return Promise.reject(error);
  }
  
  // Invalidar caché para endpoint específico
  invalidateCache(pattern) {
    const keysToDelete = [];
    
    for (const key of this.cache.cache.keys()) {
      if (key.includes(pattern)) {
        keysToDelete.push(key);
      }
    }
    
    keysToDelete.forEach(key => this.cache.delete(key));
  }
  
  // Configurar TTL para endpoint específico
  setTTL(pattern, ttl) {
    this.endpointTTLs = this.endpointTTLs || new Map();
    this.endpointTTLs.set(pattern, ttl);
  }
  
  // Obtener TTL para endpoint
  getTTL(url) {
    if (!this.endpointTTLs) return this.cache.defaultTTL;
    
    for (const [pattern, ttl] of this.endpointTTLs.entries()) {
      if (url.includes(pattern)) {
        return ttl;
      }
    }
    
    return this.cache.defaultTTL;
  }
}

export default CacheMiddleware;
```

### 4. Retry Logic y Fallbacks

#### Middleware de Retry
```javascript
// middleware/retryMiddleware.js
class RetryMiddleware {
  constructor(options = {}) {
    this.maxRetries = options.maxRetries || 3;
    this.retryDelay = options.retryDelay || 1000;
    this.backoffMultiplier = options.backoffMultiplier || 2;
    this.retryableStatuses = options.retryableStatuses || [500, 502, 503, 504];
    this.retryableErrors = options.retryableErrors || ['NETWORK_ERROR', 'TIMEOUT_ERROR'];
  }
  
  // Middleware para requests
  handleRequest(config) {
    config.retryCount = 0;
    config.maxRetries = this.maxRetries;
    return config;
  }
  
  // Middleware para responses
  handleResponse(response) {
    return response;
  }
  
  // Middleware para errores
  async handleError(error) {
    const config = error.config;
    
    // Verificar si se puede reintentar
    if (!this.shouldRetry(error, config)) {
      return Promise.reject(error);
    }
    
    // Verificar si no se ha excedido el número máximo de reintentos
    if (config.retryCount >= config.maxRetries) {
      console.warn(`Max retries (${config.maxRetries}) exceeded for ${config.url}`);
      return Promise.reject(error);
    }
    
    // Incrementar contador de reintentos
    config.retryCount++;
    
    // Calcular delay con backoff exponencial
    const delay = this.calculateDelay(config.retryCount);
    
    console.log(`Retrying request to ${config.url} (attempt ${config.retryCount}/${config.maxRetries}) in ${delay}ms`);
    
    // Esperar antes de reintentar
    await this.sleep(delay);
    
    // Reintentar la petición
    try {
      const response = await this.retryRequest(config);
      return response;
    } catch (retryError) {
      // Si falla el reintento, continuar con la cadena de reintentos
      return this.handleError(retryError);
    }
  }
  
  // Verificar si se debe reintentar
  shouldRetry(error, config) {
    // Reintentar errores de red
    if (this.retryableErrors.includes(error.code)) {
      return true;
    }
    
    // Reintentar errores HTTP específicos
    if (error.response && this.retryableStatuses.includes(error.response.status)) {
      return true;
    }
    
    // No reintentar errores de autenticación o autorización
    if (error.response && [401, 403].includes(error.response.status)) {
      return false;
    }
    
    // No reintentar errores de validación
    if (error.response && [400, 422].includes(error.response.status)) {
      return false;
    }
    
    return false;
  }
  
  // Calcular delay con backoff exponencial
  calculateDelay(retryCount) {
    return this.retryDelay * Math.pow(this.backoffMultiplier, retryCount - 1);
  }
  
  // Esperar por un tiempo específico
  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Reintentar la petición
  async retryRequest(config) {
    // Crear nueva instancia de axios para evitar problemas de estado
    const axios = require('axios');
    
    // Limpiar configuraciones que no se deben reutilizar
    const cleanConfig = { ...config };
    delete cleanConfig.retryCount;
    delete cleanConfig.maxRetries;
    
    return axios(cleanConfig);
  }
  
  // Configurar reintentos para endpoint específico
  setRetryConfig(pattern, config) {
    this.endpointRetryConfigs = this.endpointRetryConfigs || new Map();
    this.endpointRetryConfigs.set(pattern, config);
  }
  
  // Obtener configuración de reintentos para endpoint
  getRetryConfig(url) {
    if (!this.endpointRetryConfigs) return null;
    
    for (const [pattern, config] of this.endpointRetryConfigs.entries()) {
      if (url.includes(pattern)) {
        return config;
      }
    }
    
    return null;
  }
}

export default RetryMiddleware;
```

### 5. Hook Personalizado con Caché y Retry

#### Hook useApiWithCache
```javascript
// hooks/useApiWithCache.js
import { useState, useCallback, useRef, useEffect } from 'react';
import CacheService from '../services/cacheService';
import ErrorHandlerService from '../services/errorHandlerService';

export const useApiWithCache = (apiFunction, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [fromCache, setFromCache] = useState(false);
  
  const cacheService = useRef(new CacheService());
  const abortControllerRef = useRef(null);
  
  const {
    cacheKey,
    ttl = 5 * 60 * 1000, // 5 minutos
    enableCache = true,
    enableRetry = true,
    maxRetries = 3,
    retryDelay = 1000,
  } = options;

  const execute = useCallback(async (...args) => {
    // Cancelar petición anterior si existe
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    // Crear nuevo abort controller
    abortControllerRef.current = new AbortController();
    
    setLoading(true);
    setError(null);
    setFromCache(false);
    
    try {
      // Intentar obtener del caché si está habilitado
      if (enableCache && cacheKey) {
        const cachedData = cacheService.current.get(cacheKey);
        if (cachedData) {
          setData(cachedData);
          setFromCache(true);
          setLoading(false);
          return cachedData;
        }
      }
      
      // Realizar petición a la API
      const result = await apiFunction(...args);
      
      // Guardar en caché si está habilitado
      if (enableCache && cacheKey) {
        cacheService.current.set(cacheKey, result, ttl);
      }
      
      setData(result);
      return result;
    } catch (err) {
      if (err.name === 'CanceledError') {
        console.log('Request was canceled');
        return;
      }
      
      // Manejar error con retry si está habilitado
      if (enableRetry && maxRetries > 0) {
        return await handleRetry(apiFunction, args, maxRetries, retryDelay);
      }
      
      // Manejar error con el servicio centralizado
      const errorInfo = await ErrorHandlerService.handleError(err, {
        function: apiFunction.name,
        args,
        cacheKey,
      });
      
      setError(errorInfo);
      throw errorInfo;
    } finally {
      setLoading(false);
    }
  }, [apiFunction, cacheKey, ttl, enableCache, enableRetry, maxRetries, retryDelay]);

  // Lógica de retry
  const handleRetry = async (apiFunction, args, maxRetries, retryDelay) => {
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        console.log(`Retry attempt ${attempt}/${maxRetries}`);
        
        // Esperar antes de reintentar (excepto en el primer intento)
        if (attempt > 1) {
          await new Promise(resolve => setTimeout(resolve, retryDelay * (attempt - 1)));
        }
        
        const result = await apiFunction(...args);
        
        // Guardar en caché si está habilitado
        if (enableCache && cacheKey) {
          cacheService.current.set(cacheKey, result, ttl);
        }
        
        setData(result);
        return result;
      } catch (err) {
        lastError = err;
        console.warn(`Retry attempt ${attempt} failed:`, err.message);
      }
    }
    
    // Si todos los reintentos fallaron, manejar el error
    const errorInfo = await ErrorHandlerService.handleError(lastError, {
      function: apiFunction.name,
      args,
      cacheKey,
      retryAttempts: maxRetries,
    });
    
    setError(errorInfo);
    throw errorInfo;
  };

  const cancel = useCallback(() => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
  }, []);

  const refresh = useCallback(() => {
    if (cacheKey) {
      cacheService.current.delete(cacheKey);
    }
    return execute(...args);
  }, [execute, cacheKey]);

  const clearCache = useCallback(() => {
    if (cacheKey) {
      cacheService.current.delete(cacheKey);
    }
  }, [cacheKey]);

  // Cleanup al desmontar
  useEffect(() => {
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);

  return {
    data,
    loading,
    error,
    fromCache,
    execute,
    cancel,
    refresh,
    clearCache,
  };
};
```

#### Uso del Hook
```javascript
// Componente usando useApiWithCache
import React from 'react';
import { View, Text, Button, ActivityIndicator } from 'react-native';
import { useApiWithCache } from '../hooks/useApiWithCache';
import userService from '../services/userService';

const UserList = () => {
  const {
    data: users,
    loading,
    error,
    fromCache,
    execute: fetchUsers,
    cancel,
    refresh,
    clearCache,
  } = useApiWithCache(userService.getUsers, {
    cacheKey: 'users-list',
    ttl: 10 * 60 * 1000, // 10 minutos
    enableCache: true,
    enableRetry: true,
    maxRetries: 3,
    retryDelay: 1000,
  });

  React.useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

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
        <Text style={{ color: 'red' }}>Error: {error.userMessage}</Text>
        <Text style={{ fontSize: 12, color: 'gray' }}>
          Tipo: {error.type} | Severidad: {error.severity}
        </Text>
        <Button title="Reintentar" onPress={refresh} />
      </View>
    );
  }

  return (
    <View style={{ flex: 1, padding: 20 }}>
      <View style={{ flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', marginBottom: 20 }}>
        <Text style={{ fontSize: 24, fontWeight: 'bold' }}>
          Lista de Usuarios
        </Text>
        {fromCache && (
          <Text style={{ fontSize: 12, color: 'orange', fontStyle: 'italic' }}>
            Desde caché
          </Text>
        )}
      </View>
      
      {users?.map(user => (
        <View key={user.id} style={{ marginBottom: 10, padding: 10, backgroundColor: '#f0f0f0' }}>
          <Text style={{ fontWeight: 'bold' }}>{user.name}</Text>
          <Text>{user.email}</Text>
        </View>
      ))}
      
      <View style={{ flexDirection: 'row', justifyContent: 'space-around', marginTop: 20 }}>
        <Button title="Actualizar" onPress={refresh} />
        <Button title="Limpiar Caché" onPress={clearCache} />
      </View>
    </View>
  );
};

export default UserList;
```

## 🧪 Ejercicios Prácticos

### Ejercicio 1: Sistema de Caché Avanzado
Implementa un sistema de caché con las siguientes características:

```javascript
// El sistema debe incluir:
// - Caché por capas (memoria, AsyncStorage, SQLite)
// - Invalidación inteligente por patrones
// - Compresión de datos para ahorrar espacio
// - Estadísticas de hit/miss ratio
// - Limpieza automática por LRU
```

### Ejercicio 2: Middleware de Fallback
Crea un middleware que implemente fallbacks:

```javascript
// El middleware debe:
// - Intentar API principal
// - Si falla, usar API de respaldo
// - Si ambas fallan, usar datos del caché
// - Implementar health check de APIs
// - Loggear fallbacks utilizados
```

### Ejercicio 3: Sistema de Notificaciones de Error
Implementa un sistema de notificaciones para errores:

```javascript
// El sistema debe:
// - Mostrar errores según severidad
// - Agrupar errores similares
// - Permitir reportar errores
// - Implementar rate limiting para notificaciones
// - Integrar con servicios de monitoreo
```

## 📝 Resumen de la Clase

### **Conceptos Clave Aprendidos:**
1. **Clasificación de Errores**: Red, HTTP y aplicación con estrategias específicas
2. **Manejo Centralizado**: Servicio unificado para clasificar y procesar errores
3. **Sistema de Caché**: Caché inteligente con TTL, limpieza automática y estadísticas
4. **Retry Logic**: Reintentos con backoff exponencial y configuración flexible
5. **Fallbacks**: Estrategias para manejar fallos de APIs con datos alternativos

### **Habilidades Desarrolladas:**
- Implementar manejo robusto de errores en aplicaciones React Native
- Crear sistemas de caché eficientes para mejorar performance
- Implementar lógica de reintentos inteligente
- Manejar fallbacks y degradación graceful
- Optimizar el rendimiento de las aplicaciones

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Autenticación y Autorización**, que nos permitirá:
- Implementar sistemas de autenticación robustos
- Manejar diferentes tipos de tokens (JWT, OAuth)
- Implementar autorización basada en roles
- Gestionar sesiones y permisos de manera segura

---

**💡 Consejo**: Experimenta con diferentes estrategias de caché y retry para encontrar la combinación óptima para tu aplicación. El caché puede mejorar significativamente la experiencia del usuario, pero también puede ocultar problemas de la API.
