# 📊 Clase 3: Performance Monitoring

## 🎯 Objetivos de la Clase
- Implementar monitoreo avanzado de rendimiento
- Configurar métricas de rendimiento personalizadas
- Monitorear operaciones de red y base de datos
- Optimizar la experiencia del usuario basada en métricas

---

## 📚 Contenido Teórico

### 1. ¿Qué es Performance Monitoring?

#### **Definición y Propósito**
**Performance Monitoring** es el proceso de medir y analizar el rendimiento de una aplicación para:
- **Identificar cuellos de botella** en el rendimiento
- **Optimizar tiempos de respuesta** de la UI
- **Monitorear operaciones de red** y base de datos
- **Mejorar la experiencia** del usuario
- **Detectar problemas** de rendimiento en producción

#### **Métricas Clave de Rendimiento:**
- **FCP (First Contentful Paint)**: Primer contenido visible
- **LCP (Largest Contentful Paint)**: Elemento más grande visible
- **FID (First Input Delay)**: Tiempo de respuesta a la primera interacción
- **CLS (Cumulative Layout Shift)**: Estabilidad visual
- **TTFB (Time to First Byte)**: Tiempo de respuesta del servidor

### 2. Firebase Performance Monitoring

#### **Configuración Básica**
```javascript
// config/performance.js
import perf from '@react-native-firebase/perf';

export class PerformanceMonitor {
  // Monitorear tiempo de carga de pantalla
  static startScreenLoad(screenName) {
    const trace = perf().newTrace('screen_load');
    trace.putAttribute('screen_name', screenName);
    trace.putAttribute('timestamp', Date.now().toString());
    trace.start();
    return trace;
  }

  // Monitorear operaciones de red
  static startNetworkRequest(url, method) {
    const metric = perf().newHttpMetric(url, method);
    metric.putAttribute('request_type', 'api_call');
    metric.start();
    return metric;
  }

  // Monitorear operaciones personalizadas
  static startCustomTrace(traceName, attributes = {}) {
    const trace = perf().newTrace(traceName);
    
    // Agregar atributos personalizados
    Object.entries(attributes).forEach(([key, value]) => {
      trace.putAttribute(key, value.toString());
    });
    
    trace.start();
    return trace;
  }

  // Monitorear operaciones de base de datos
  static startDatabaseOperation(operationType, tableName) {
    const trace = perf().newTrace('database_operation');
    trace.putAttribute('operation_type', operationType);
    trace.putAttribute('table_name', tableName);
    trace.start();
    return trace;
  }
}
```

#### **Hook para Monitoreo de Pantallas**
```javascript
// hooks/usePerformanceMonitoring.js
import { useEffect, useRef } from 'react';
import { PerformanceMonitor } from '../config/performance';

export const useScreenPerformance = (screenName, additionalAttributes = {}) => {
  const traceRef = useRef(null);

  useEffect(() => {
    // Iniciar monitoreo cuando la pantalla se monta
    traceRef.current = PerformanceMonitor.startScreenLoad(screenName);
    
    // Agregar atributos adicionales
    if (additionalAttributes) {
      Object.entries(additionalAttributes).forEach(([key, value]) => {
        traceRef.current.putAttribute(key, value.toString());
      });
    }

    return () => {
      // Detener monitoreo cuando la pantalla se desmonta
      if (traceRef.current) {
        traceRef.current.stop();
      }
    };
  }, [screenName, JSON.stringify(additionalAttributes)]);

  // Función para agregar métricas personalizadas
  const addMetric = (name, value) => {
    if (traceRef.current) {
      traceRef.current.putMetric(name, value);
    }
  };

  // Función para agregar atributos dinámicos
  const addAttribute = (name, value) => {
    if (traceRef.current) {
      traceRef.current.putAttribute(name, value.toString());
    }
  };

  return { addMetric, addAttribute };
};
```

### 3. Monitoreo de Operaciones de Red

#### **Interceptor de Red con Métricas**
```javascript
// services/networkMonitor.js
import { PerformanceMonitor } from '../config/performance';

export class NetworkMonitor {
  static async monitorRequest(url, options = {}) {
    const method = options.method || 'GET';
    const metric = PerformanceMonitor.startNetworkRequest(url, method);
    
    try {
      // Agregar atributos adicionales
      metric.putAttribute('request_size', this.calculateRequestSize(options.body));
      metric.putAttribute('user_id', this.getCurrentUserId());
      
      const startTime = Date.now();
      const response = await fetch(url, options);
      const responseTime = Date.now() - startTime;
      
      // Configurar métricas de respuesta
      metric.setHttpResponseCode(response.status);
      metric.setResponseContentType(response.headers.get('content-type'));
      metric.setResponsePayloadSize(parseInt(response.headers.get('content-length') || '0'));
      
      // Agregar métricas personalizadas
      metric.putMetric('response_time_ms', responseTime);
      metric.putMetric('response_size_bytes', parseInt(response.headers.get('content-length') || '0'));
      
      // Agregar atributos de respuesta
      metric.putAttribute('response_status', response.status.toString());
      metric.putAttribute('response_time_category', this.categorizeResponseTime(responseTime));
      
      return response;
    } catch (error) {
      // Agregar métricas de error
      metric.putAttribute('error_type', error.name);
      metric.putAttribute('error_message', error.message);
      metric.putAttribute('request_failed', 'true');
      
      throw error;
    } finally {
      metric.stop();
    }
  }

  static calculateRequestSize(body) {
    if (!body) return '0';
    if (typeof body === 'string') return body.length.toString();
    if (body instanceof FormData) return 'form_data';
    return JSON.stringify(body).length.toString();
  }

  static categorizeResponseTime(responseTime) {
    if (responseTime < 100) return 'excellent';
    if (responseTime < 300) return 'good';
    if (responseTime < 1000) return 'fair';
    return 'poor';
  }

  static getCurrentUserId() {
    // Implementar lógica para obtener ID del usuario actual
    return 'user_123';
  }
}

// Uso en servicios de API
export const apiService = {
  async get(endpoint) {
    return NetworkMonitor.monitorRequest(`${API_BASE_URL}${endpoint}`, {
      method: 'GET',
      headers: { 'Authorization': `Bearer ${getToken()}` }
    });
  },

  async post(endpoint, data) {
    return NetworkMonitor.monitorRequest(`${API_BASE_URL}${endpoint}`, {
      method: 'POST',
      headers: { 
        'Authorization': `Bearer ${getToken()}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });
  }
};
```

### 4. Monitoreo de Operaciones de Base de Datos

#### **Monitoreo de SQLite/Realm**
```javascript
// services/databaseMonitor.js
import { PerformanceMonitor } from '../config/performance';

export class DatabaseMonitor {
  static async monitorQuery(operationType, query, parameters = []) {
    const trace = PerformanceMonitor.startDatabaseOperation(operationType, 'main');
    
    try {
      // Agregar atributos de la consulta
      trace.putAttribute('query_type', operationType);
      trace.putAttribute('query_length', query.length.toString());
      trace.putAttribute('parameters_count', parameters.length.toString());
      
      const startTime = Date.now();
      const result = await this.executeQuery(query, parameters);
      const executionTime = Date.now() - startTime;
      
      // Agregar métricas de rendimiento
      trace.putMetric('execution_time_ms', executionTime);
      trace.putMetric('result_count', Array.isArray(result) ? result.length : 1);
      
      // Agregar atributos de resultado
      trace.putAttribute('execution_time_category', this.categorizeExecutionTime(executionTime));
      trace.putAttribute('result_size', JSON.stringify(result).length.toString());
      
      return result;
    } catch (error) {
      // Agregar métricas de error
      trace.putAttribute('error_type', error.name);
      trace.putAttribute('error_message', error.message);
      trace.putAttribute('query_failed', 'true');
      
      throw error;
    } finally {
      trace.stop();
    }
  }

  static categorizeExecutionTime(executionTime) {
    if (executionTime < 10) return 'excellent';
    if (executionTime < 50) return 'good';
    if (executionTime < 200) return 'fair';
    return 'poor';
  }

  static async executeQuery(query, parameters) {
    // Implementar ejecución real de la consulta
    // Por ejemplo, con SQLite o Realm
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve([{ id: 1, name: 'Test' }]);
      }, Math.random() * 100);
    });
  }
}

// Uso en repositorios
export class UserRepository {
  static async findById(id) {
    return DatabaseMonitor.monitorQuery(
      'SELECT',
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
  }

  static async create(userData) {
    return DatabaseMonitor.monitorQuery(
      'INSERT',
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [userData.name, userData.email]
    );
  }
}
```

### 5. Monitoreo de Rendimiento de Componentes

#### **HOC para Monitoreo de Rendimiento**
```javascript
// components/withPerformanceMonitoring.js
import React, { useEffect, useRef } from 'react';
import { PerformanceMonitor } from '../config/performance';

export const withPerformanceMonitoring = (WrappedComponent, componentName) => {
  return function PerformanceMonitoredComponent(props) {
    const traceRef = useRef(null);
    const renderCountRef = useRef(0);
    const lastRenderTimeRef = useRef(Date.now());

    useEffect(() => {
      // Iniciar monitoreo del componente
      traceRef.current = PerformanceMonitor.startCustomTrace('component_render', {
        component_name: componentName,
        render_count: renderCountRef.current.toString()
      });

      // Agregar métricas de renderizado
      const currentTime = Date.now();
      const timeSinceLastRender = currentTime - lastRenderTimeRef.current;
      
      traceRef.current.putMetric('time_since_last_render_ms', timeSinceLastRender);
      traceRef.current.putMetric('total_render_count', renderCountRef.current);

      // Actualizar contadores
      renderCountRef.current++;
      lastRenderTimeRef.current = currentTime;

      return () => {
        if (traceRef.current) {
          traceRef.current.stop();
        }
      };
    });

    // Monitorear tiempo de renderizado
    const startRenderTime = Date.now();
    const result = <WrappedComponent {...props} />;
    const renderTime = Date.now() - startRenderTime;

    // Agregar métrica de tiempo de renderizado
    if (traceRef.current) {
      traceRef.current.putMetric('render_time_ms', renderTime);
      traceRef.current.putAttribute('render_time_category', 
        renderTime < 16 ? 'excellent' : 
        renderTime < 33 ? 'good' : 
        renderTime < 50 ? 'fair' : 'poor'
      );
    }

    return result;
  };
};

// Uso del HOC
const MonitoredUserList = withPerformanceMonitoring(UserList, 'UserList');
```

#### **Monitoreo de Listas y Scroll**
```javascript
// hooks/useListPerformance.js
import { useRef, useCallback } from 'react';
import { PerformanceMonitor } from '../config/performance';

export const useListPerformance = (listName, itemCount) => {
  const scrollStartTimeRef = useRef(null);
  const scrollTraceRef = useRef(null);

  const onScrollBeginDrag = useCallback(() => {
    scrollStartTimeRef.current = Date.now();
    scrollTraceRef.current = PerformanceMonitor.startCustomTrace('list_scroll', {
      list_name: listName,
      item_count: itemCount.toString()
    });
  }, [listName, itemCount]);

  const onScrollEndDrag = useCallback(() => {
    if (scrollStartTimeRef.current && scrollTraceRef.current) {
      const scrollDuration = Date.now() - scrollStartTimeRef.current;
      
      scrollTraceRef.current.putMetric('scroll_duration_ms', scrollDuration);
      scrollTraceRef.current.putAttribute('scroll_type', 'user_scroll');
      
      scrollTraceRef.current.stop();
      
      // Resetear referencias
      scrollStartTimeRef.current = null;
      scrollTraceRef.current = null;
    }
  }, []);

  const onMomentumScrollBegin = useCallback(() => {
    scrollStartTimeRef.current = Date.now();
    scrollTraceRef.current = PerformanceMonitor.startCustomTrace('list_scroll', {
      list_name: listName,
      item_count: itemCount.toString()
    });
  }, [listName, itemCount]);

  const onMomentumScrollEnd = useCallback(() => {
    if (scrollStartTimeRef.current && scrollTraceRef.current) {
      const scrollDuration = Date.now() - scrollStartTimeRef.current;
      
      scrollTraceRef.current.putMetric('scroll_duration_ms', scrollDuration);
      scrollTraceRef.current.putAttribute('scroll_type', 'momentum_scroll');
      
      scrollTraceRef.current.stop();
      
      // Resetear referencias
      scrollStartTimeRef.current = null;
      scrollTraceRef.current = null;
    }
  }, []);

  return {
    onScrollBeginDrag,
    onScrollEndDrag,
    onMomentumScrollBegin,
    onMomentumScrollEnd
  };
};

// Uso en FlatList
const UserList = () => {
  const scrollHandlers = useListPerformance('UserList', users.length);

  return (
    <FlatList
      data={users}
      renderItem={renderUser}
      {...scrollHandlers}
    />
  );
};
```

### 6. Métricas de Rendimiento Personalizadas

#### **Sistema de Métricas Personalizadas**
```javascript
// services/customMetrics.js
import { PerformanceMonitor } from '../config/performance';

export class CustomMetrics {
  static trackUserAction(actionName, context = {}) {
    const trace = PerformanceMonitor.startCustomTrace('user_action', {
      action_name: actionName,
      timestamp: Date.now().toString()
    });

    // Agregar contexto de la acción
    Object.entries(context).forEach(([key, value]) => {
      trace.putAttribute(key, value.toString());
    });

    return trace;
  }

  static trackFeatureUsage(featureName, usageData = {}) {
    const trace = PerformanceMonitor.startCustomTrace('feature_usage', {
      feature_name: featureName,
      usage_count: '1'
    });

    // Agregar datos de uso
    Object.entries(usageData).forEach(([key, value]) => {
      trace.putAttribute(key, value.toString());
    });

    return trace;
  }

  static trackMemoryUsage(componentName) {
    const trace = PerformanceMonitor.startCustomTrace('memory_usage', {
      component_name: componentName
    });

    // Simular métricas de memoria (en una app real usarías APIs nativas)
    const memoryUsage = Math.random() * 100;
    trace.putMetric('memory_mb', memoryUsage);
    trace.putAttribute('memory_category', 
      memoryUsage < 10 ? 'low' : 
      memoryUsage < 50 ? 'medium' : 'high'
    );

    return trace;
  }

  static trackBatteryImpact(operationName, batteryData = {}) {
    const trace = PerformanceMonitor.startCustomTrace('battery_impact', {
      operation_name: operationName
    });

    // Agregar datos de batería
    Object.entries(batteryData).forEach(([key, value]) => {
      trace.putAttribute(key, value.toString());
    });

    return trace;
  }
}

// Uso en componentes
export const useCustomMetrics = () => {
  const trackAction = useCallback((actionName, context) => {
    const trace = CustomMetrics.trackUserAction(actionName, context);
    
    return {
      stop: () => trace.stop(),
      addAttribute: (key, value) => trace.putAttribute(key, value.toString()),
      addMetric: (key, value) => trace.putMetric(key, value)
    };
  }, []);

  return { trackAction };
};
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Sistema de Monitoreo Completo
Implementa un sistema de monitoreo que cubra todos los aspectos:

**Requisitos:**
- Monitoreo de pantallas
- Monitoreo de operaciones de red
- Monitoreo de base de datos
- Métricas personalizadas

**Implementación:**
```javascript
class ComprehensivePerformanceMonitor {
  static async monitorScreen(screenName, callback) {
    const trace = PerformanceMonitor.startScreenLoad(screenName);
    
    try {
      const result = await callback();
      trace.putAttribute('screen_load_success', 'true');
      return result;
    } catch (error) {
      trace.putAttribute('screen_load_success', 'false');
      trace.putAttribute('error_message', error.message);
      throw error;
    } finally {
      trace.stop();
    }
  }

  static async monitorApiCall(url, options, callback) {
    const metric = PerformanceMonitor.startNetworkRequest(url, options.method);
    
    try {
      const result = await callback();
      metric.putAttribute('api_call_success', 'true');
      return result;
    } catch (error) {
      metric.putAttribute('api_call_success', 'false');
      metric.putAttribute('error_type', error.name);
      throw error;
    } finally {
      metric.stop();
    }
  }
}
```

### Ejercicio 2: Dashboard de Rendimiento
Crea un dashboard que muestre métricas de rendimiento:

**Requisitos:**
- Métricas de tiempo de carga
- Métricas de operaciones de red
- Métricas de base de datos
- Gráficos de tendencias

**Implementación:**
```javascript
class PerformanceDashboard {
  static async getPerformanceSummary() {
    const metrics = await this.fetchPerformanceMetrics();
    
    return {
      screenLoadTimes: this.aggregateScreenMetrics(metrics.screens),
      networkPerformance: this.aggregateNetworkMetrics(metrics.network),
      databasePerformance: this.aggregateDatabaseMetrics(metrics.database),
      userActions: this.aggregateUserActionMetrics(metrics.actions)
    };
  }

  static aggregateScreenMetrics(screenMetrics) {
    return screenMetrics.reduce((acc, metric) => {
      if (!acc[metric.screen_name]) {
        acc[metric.screen_name] = [];
      }
      acc[metric.screen_name].push(metric.load_time);
      return acc;
    }, {});
  }
}
```

---

## 🔍 Puntos Clave

1. **Performance Monitoring** es esencial para optimizar la experiencia del usuario
2. **Firebase Performance** proporciona herramientas robustas de monitoreo
3. **Métricas personalizadas** permiten monitorear aspectos específicos de la app
4. **Monitoreo de red y base de datos** ayuda a identificar cuellos de botella
5. **HOCs y hooks** facilitan la implementación del monitoreo en componentes

---

## 📖 Recursos Adicionales

- [Firebase Performance Monitoring](https://firebase.google.com/docs/perf)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [Web Performance Metrics](https://web.dev/metrics/)
- [Performance Monitoring Best Practices](https://firebase.google.com/docs/perf/guides)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Analytics y User Behavior** y cómo implementar un sistema completo de análisis del comportamiento del usuario.
