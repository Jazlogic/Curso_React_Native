# Clase 3: Performance Monitoring ⚡

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Configurar Firebase Performance** para monitoreo de rendimiento
2. **Implementar métricas personalizadas** de performance
3. **Monitorear operaciones críticas** de la aplicación
4. **Analizar y optimizar** el rendimiento de la app
5. **Crear dashboards** de performance en tiempo real

## 🎯 Conceptos Clave

### ¿Qué es Performance Monitoring?

**Performance Monitoring** es el proceso de medir, analizar y optimizar el rendimiento de una aplicación para asegurar que funcione de manera eficiente y proporcione una experiencia de usuario fluida.

```javascript
// Ejemplo básico de Firebase Performance
import perf from '@react-native-firebase/perf';

class PerformanceMonitor {
  constructor() {
    this.traces = new Map();
    this.metrics = new Map();
  }

  // Iniciar trace de performance
  startTrace(traceName) {
    try {
      const trace = perf().newTrace(traceName);
      trace.start();
      
      this.traces.set(traceName, trace);
      return trace;
    } catch (error) {
      console.error('Error iniciando trace:', error);
      return null;
    }
  }

  // Finalizar trace de performance
  stopTrace(traceName) {
    try {
      const trace = this.traces.get(traceName);
      if (trace) {
        trace.stop();
        this.traces.delete(traceName);
        return true;
      }
      return false;
    } catch (error) {
      console.error('Error finalizando trace:', error);
      return false;
    }
  }

  // Agregar métrica personalizada
  addMetric(traceName, metricName, value) {
    try {
      const trace = this.traces.get(traceName);
      if (trace) {
        trace.putMetric(metricName, value);
        return true;
      }
      return false;
    } catch (error) {
      console.error('Error agregando métrica:', error);
      return false;
    }
  }

  // Agregar atributo personalizado
  addAttribute(traceName, attributeName, value) {
    try {
      const trace = this.traces.get(traceName);
      if (trace) {
        trace.putAttribute(attributeName, String(value));
        return true;
      }
      return false;
    } catch (error) {
      console.error('Error agregando atributo:', error);
      return false;
    }
  }
}
```

### Tipos de Métricas de Performance

#### 1. **App Launch Performance**
```javascript
// Monitorear tiempo de inicio de la app
class AppLaunchMonitor {
  constructor(performanceMonitor) {
    this.performanceMonitor = performanceMonitor;
    this.launchStartTime = Date.now();
    this.setupLaunchMonitoring();
  }

  // Configurar monitoreo de inicio
  setupLaunchMonitoring() {
    // Trace de inicio de app
    const appLaunchTrace = this.performanceMonitor.startTrace('app_launch');
    
    if (appLaunchTrace) {
      // Agregar métricas de inicio
      this.performanceMonitor.addMetric('app_launch', 'launch_start_time', this.launchStartTime);
      
      // Monitorear cuando la app esté lista
      this.monitorAppReady();
    }
  }

  // Monitorear cuando la app esté lista
  monitorAppReady() {
    // Usar InteractionManager para detectar cuando la app esté lista
    const { InteractionManager } = require('react-native');
    
    InteractionManager.runAfterInteractions(() => {
      const appReadyTime = Date.now();
      const totalLaunchTime = appReadyTime - this.launchStartTime;
      
      // Agregar métricas de tiempo total
      this.performanceMonitor.addMetric('app_launch', 'total_launch_time', totalLaunchTime);
      this.performanceMonitor.addMetric('app_launch', 'app_ready_time', appReadyTime);
      
      // Finalizar trace
      this.performanceMonitor.stopTrace('app_launch');
      
      console.log(`App iniciada en ${totalLaunchTime}ms`);
    });
  }
}
```

#### 2. **Screen Performance**
```javascript
// Monitorear performance de pantallas
class ScreenPerformanceMonitor {
  constructor(performanceMonitor) {
    this.performanceMonitor = performanceMonitor;
    this.screenTraces = new Map();
  }

  // Iniciar monitoreo de pantalla
  startScreenTrace(screenName) {
    const traceName = `screen_${screenName}`;
    const trace = this.performanceMonitor.startTrace(traceName);
    
    if (trace) {
      this.screenTraces.set(screenName, trace);
      
      // Agregar atributos de la pantalla
      this.performanceMonitor.addAttribute(traceName, 'screen_name', screenName);
      this.performanceMonitor.addAttribute(traceName, 'start_time', Date.now());
      
      return trace;
    }
    return null;
  }

  // Finalizar monitoreo de pantalla
  stopScreenTrace(screenName) {
    const traceName = `screen_${screenName}`;
    const trace = this.screenTraces.get(screenName);
    
    if (trace) {
      // Agregar métricas finales
      const endTime = Date.now();
      const startTime = parseInt(this.performanceMonitor.getAttribute(traceName, 'start_time') || '0');
      const screenTime = endTime - startTime;
      
      this.performanceMonitor.addMetric(traceName, 'screen_time', screenTime);
      this.performanceMonitor.addAttribute(traceName, 'end_time', endTime);
      
      // Finalizar trace
      this.performanceMonitor.stopTrace(traceName);
      this.screenTraces.delete(screenName);
      
      return screenTime;
    }
    return 0;
  }

  // Monitorear renderizado de componentes
  monitorComponentRender(componentName, screenName) {
    const traceName = `component_render_${componentName}`;
    const trace = this.performanceMonitor.startTrace(traceName);
    
    if (trace) {
      this.performanceMonitor.addAttribute(traceName, 'component_name', componentName);
      this.performanceMonitor.addAttribute(traceName, 'screen_name', screenName);
      
      return trace;
    }
    return null;
  }
}
```

#### 3. **Network Performance**
```javascript
// Monitorear performance de red
class NetworkPerformanceMonitor {
  constructor(performanceMonitor) {
    this.performanceMonitor = performanceMonitor;
    this.networkTraces = new Map();
  }

  // Monitorear request HTTP
  monitorHttpRequest(url, method) {
    try {
      const trace = perf().newHttpMetric(url, method);
      trace.start();
      
      const traceId = this.generateTraceId(url, method);
      this.networkTraces.set(traceId, trace);
      
      return trace;
    } catch (error) {
      console.error('Error iniciando HTTP trace:', error);
      return null;
    }
  }

  // Finalizar monitoreo de request HTTP
  stopHttpRequest(url, method, statusCode, responseSize) {
    try {
      const traceId = this.generateTraceId(url, method);
      const trace = this.networkTraces.get(traceId);
      
      if (trace) {
        // Agregar métricas de respuesta
        trace.setHttpResponseCode(statusCode);
        trace.setResponsePayloadSize(responseSize);
        
        // Finalizar trace
        trace.stop();
        this.networkTraces.delete(traceId);
        
        return true;
      }
      return false;
    } catch (error) {
      console.error('Error finalizando HTTP trace:', error);
      return false;
    }
  }

  // Generar ID único para trace
  generateTraceId(url, method) {
    return `${method}_${url}_${Date.now()}`;
  }

  // Monitorear request con Axios
  setupAxiosMonitoring(axiosInstance) {
    axiosInstance.interceptors.request.use(
      (config) => {
        const trace = this.monitorHttpRequest(config.url, config.method);
        if (trace) {
          config.metadata = { trace };
        }
        return config;
      },
      (error) => {
        return Promise.reject(error);
      }
    );

    axiosInstance.interceptors.response.use(
      (response) => {
        const trace = response.config.metadata?.trace;
        if (trace) {
          const responseSize = JSON.stringify(response.data).length;
          this.stopHttpRequest(
            response.config.url,
            response.config.method,
            response.status,
            responseSize
          );
        }
        return response;
      },
      (error) => {
        const trace = error.config?.metadata?.trace;
        if (trace) {
          this.stopHttpRequest(
            error.config.url,
            error.config.method,
            error.response?.status || 0,
            0
          );
        }
        return Promise.reject(error);
      }
    );
  }
}
```

## 🛠️ Configuración Avanzada

### Configuración de Firebase Performance
```javascript
// Configuración avanzada de Firebase Performance
import perf from '@react-native-firebase/perf';

class AdvancedPerformanceMonitor {
  constructor() {
    this.setupAdvancedConfiguration();
  }

  // Configuración avanzada
  async setupAdvancedConfiguration() {
    try {
      // Habilitar collection de performance
      await perf().setPerformanceCollectionEnabled(true);
      
      // Configurar sampling rate
      await perf().setPerformanceCollectionEnabled(true);
      
      console.log('Firebase Performance configurado correctamente');
    } catch (error) {
      console.error('Error configurando Firebase Performance:', error);
    }
  }

  // Monitorear operaciones personalizadas
  async monitorCustomOperation(operationName, operation) {
    const trace = perf().newTrace(operationName);
    
    try {
      trace.start();
      
      // Agregar atributos de contexto
      trace.putAttribute('operation_type', 'custom');
      trace.putAttribute('start_time', Date.now().toString());
      
      // Ejecutar operación
      const result = await operation();
      
      // Agregar métricas de éxito
      trace.putMetric('success', 1);
      trace.putMetric('duration', Date.now() - parseInt(trace.getAttribute('start_time')));
      
      return result;
    } catch (error) {
      // Agregar métricas de error
      trace.putMetric('success', 0);
      trace.putAttribute('error_message', error.message);
      
      throw error;
    } finally {
      trace.stop();
    }
  }

  // Monitorear operaciones en lote
  async monitorBatchOperation(operationName, operations) {
    const trace = perf().newTrace(operationName);
    
    try {
      trace.start();
      
      trace.putAttribute('operation_type', 'batch');
      trace.putAttribute('batch_size', operations.length.toString());
      
      const startTime = Date.now();
      const results = await Promise.all(operations);
      const totalTime = Date.now() - startTime;
      
      trace.putMetric('total_operations', operations.length);
      trace.putMetric('total_time', totalTime);
      trace.putMetric('avg_time_per_operation', totalTime / operations.length);
      
      return results;
    } catch (error) {
      trace.putAttribute('error_message', error.message);
      throw error;
    } finally {
      trace.stop();
    }
  }
}
```

### Métricas Personalizadas Avanzadas
```javascript
// Sistema de métricas personalizadas avanzado
class AdvancedMetricsSystem {
  constructor(performanceMonitor) {
    this.performanceMonitor = performanceMonitor;
    this.customMetrics = new Map();
    this.performanceThresholds = new Map();
  }

  // Definir métrica personalizada
  defineMetric(metricName, options = {}) {
    const metric = {
      name: metricName,
      type: options.type || 'counter',
      unit: options.unit || 'count',
      description: options.description || '',
      thresholds: options.thresholds || {},
      ...options
    };
    
    this.customMetrics.set(metricName, metric);
    
    // Configurar thresholds por defecto
    if (metric.type === 'timer') {
      this.performanceThresholds.set(metricName, {
        warning: options.warning || 1000, // 1 segundo
        critical: options.critical || 3000  // 3 segundos
      });
    }
    
    return metric;
  }

  // Registrar valor de métrica
  recordMetric(metricName, value, context = {}) {
    const metric = this.customMetrics.get(metricName);
    if (!metric) {
      console.warn(`Métrica no definida: ${metricName}`);
      return false;
    }

    try {
      // Registrar en Firebase Performance
      if (metric.type === 'timer') {
        this.performanceMonitor.addMetric(metricName, 'duration', value);
        
        // Verificar thresholds
        this.checkThresholds(metricName, value);
      } else {
        this.performanceMonitor.addMetric(metricName, 'value', value);
      }
      
      // Agregar contexto
      Object.entries(context).forEach(([key, val]) => {
        this.performanceMonitor.addAttribute(metricName, key, val);
      });
      
      return true;
    } catch (error) {
      console.error(`Error registrando métrica ${metricName}:`, error);
      return false;
    }
  }

  // Verificar thresholds de performance
  checkThresholds(metricName, value) {
    const thresholds = this.performanceThresholds.get(metricName);
    if (!thresholds) return;
    
    if (value > thresholds.critical) {
      console.error(`🚨 CRÍTICO: ${metricName} excedió threshold crítico (${value}ms > ${thresholds.critical}ms)`);
      this.triggerAlert('critical', metricName, value, thresholds.critical);
    } else if (value > thresholds.warning) {
      console.warn(`⚠️ ADVERTENCIA: ${metricName} excedió threshold de advertencia (${value}ms > ${thresholds.warning}ms)`);
      this.triggerAlert('warning', metricName, value, thresholds.warning);
    }
  }

  // Disparar alertas
  triggerAlert(level, metricName, value, threshold) {
    // Implementar sistema de alertas
    const alert = {
      level,
      metric: metricName,
      value,
      threshold,
      timestamp: Date.now()
    };
    
    console.log('Alerta de Performance:', alert);
    
    // Aquí podrías enviar a un servicio de alertas
    // o mostrar notificación al usuario
  }
}
```

## 📊 Dashboard de Performance

### Componente de Dashboard
```javascript
// Dashboard de performance en React Native
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, StyleSheet } from 'react-native';

const PerformanceDashboard = ({ performanceMonitor }) => {
  const [metrics, setMetrics] = useState({});
  const [traces, setTraces] = useState([]);
  const [isRefreshing, setIsRefreshing] = useState(false);

  // Actualizar métricas
  const updateMetrics = async () => {
    setIsRefreshing(true);
    
    try {
      // Obtener métricas actuales
      const currentMetrics = await performanceMonitor.getCurrentMetrics();
      const currentTraces = await performanceMonitor.getActiveTraces();
      
      setMetrics(currentMetrics);
      setTraces(currentTraces);
    } catch (error) {
      console.error('Error actualizando métricas:', error);
    } finally {
      setIsRefreshing(false);
    }
  };

  // Actualizar cada 5 segundos
  useEffect(() => {
    updateMetrics();
    const interval = setInterval(updateMetrics, 5000);
    
    return () => clearInterval(interval);
  }, []);

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Dashboard de Performance</Text>
      
      {/* Métricas Generales */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Métricas Generales</Text>
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>App Launch:</Text>
          <Text style={styles.metricValue}>{metrics.appLaunch || 'N/A'}ms</Text>
        </View>
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Memory Usage:</Text>
          <Text style={styles.metricValue}>{metrics.memoryUsage || 'N/A'}MB</Text>
        </View>
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>CPU Usage:</Text>
          <Text style={styles.metricValue}>{metrics.cpuUsage || 'N/A'}%</Text>
        </View>
      </View>
      
      {/* Traces Activos */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Traces Activos</Text>
        {traces.map((trace, index) => (
          <View key={index} style={styles.traceRow}>
            <Text style={styles.traceName}>{trace.name}</Text>
            <Text style={styles.traceDuration}>{trace.duration}ms</Text>
          </View>
        ))}
      </View>
      
      {/* Métricas de Red */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance de Red</Text>
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Avg Response Time:</Text>
          <Text style={styles.metricValue}>{metrics.avgResponseTime || 'N/A'}ms</Text>
        </View>
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Success Rate:</Text>
          <Text style={styles.metricValue}>{metrics.successRate || 'N/A'}%</Text>
        </View>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center'
  },
  section: {
    backgroundColor: 'white',
    padding: 16,
    marginBottom: 16,
    borderRadius: 8,
    elevation: 2
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 12,
    color: '#333'
  },
  metricRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 8
  },
  metricLabel: {
    fontSize: 16,
    color: '#666'
  },
  metricValue: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333'
  },
  traceRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#eee'
  },
  traceName: {
    fontSize: 14,
    color: '#333'
  },
  traceDuration: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#007AFF'
  }
});

export default PerformanceDashboard;
```

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Configuración de Firebase Performance
**Objetivo**: Configurar Firebase Performance con métricas básicas

**Requisitos**:
1. Configurar Firebase Performance en el proyecto
2. Implementar monitoreo de app launch
3. Monitorear performance de pantallas
4. Crear métricas personalizadas básicas

**Implementación**:
```javascript
// Implementar las clases básicas de performance monitoring
// y configurar en App.js

const App = () => {
  const performanceMonitor = new PerformanceMonitor();
  const appLaunchMonitor = new AppLaunchMonitor(performanceMonitor);
  const screenMonitor = new ScreenPerformanceMonitor(performanceMonitor);

  useEffect(() => {
    // Monitorear inicio de la app
    appLaunchMonitor.setupLaunchMonitoring();
  }, []);

  // ... resto del componente
};
```

### Ejercicio 2: Sistema de Métricas Avanzadas
**Objetivo**: Crear sistema de métricas personalizadas con thresholds

**Requisitos**:
- Definir métricas personalizadas
- Implementar thresholds de performance
- Sistema de alertas automáticas
- Dashboard de métricas en tiempo real

**Implementación**:
```javascript
// Implementar AdvancedMetricsSystem y PerformanceDashboard

const metricsSystem = new AdvancedMetricsSystem(performanceMonitor);

// Definir métricas
metricsSystem.defineMetric('api_response_time', {
  type: 'timer',
  unit: 'ms',
  description: 'Tiempo de respuesta de APIs',
  warning: 1000,
  critical: 3000
});

// Usar en componentes
const MyComponent = () => {
  const handleApiCall = async () => {
    const startTime = Date.now();
    
    try {
      const result = await apiCall();
      const duration = Date.now() - startTime;
      
      metricsSystem.recordMetric('api_response_time', duration, {
        endpoint: '/api/data',
        success: true
      });
      
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      
      metricsSystem.recordMetric('api_response_time', duration, {
        endpoint: '/api/data',
        success: false,
        error: error.message
      });
      
      throw error;
    }
  };

  // ... resto del componente
};
```

### Ejercicio 3: Dashboard de Performance Completo
**Objetivo**: Crear dashboard completo con gráficos y alertas

**Características**:
- Métricas en tiempo real
- Gráficos de tendencias
- Alertas automáticas
- Filtros por tipo de métrica
- Exportación de datos

## 📚 Resumen de la Clase

### **Conceptos Clave Aprendidos**:
1. **Performance Monitoring** es esencial para optimizar apps
2. **Firebase Performance** proporciona métricas automáticas
3. **Métricas personalizadas** para operaciones específicas
4. **Thresholds y alertas** para problemas de performance
5. **Dashboard en tiempo real** para monitoreo continuo

### **Próximos Pasos**:
- Configurar Firebase Performance
- Implementar métricas personalizadas
- Crear sistema de alertas
- Desarrollar dashboard completo

### **Recursos Adicionales**:
- [Firebase Performance Documentation](https://firebase.google.com/docs/perf-mon)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [Performance Monitoring Best Practices](https://firebase.google.com/docs/perf-mon/performance-monitoring-best-practices)

---

## 🧭 Navegación

- **Clase Anterior**: [Clase 2: Crashlytics y Error Tracking](clase_2_crashlytics_error_tracking.md)
- **Clase Siguiente**: [Clase 4: Analytics y User Behavior](clase_4_analytics_user_behavior.md)
- **[🏠 Volver al Índice Principal](../../INDICE_COMPLETO.md)**
- **[📚 Volver al README del Módulo](../README.md)**

---

**💡 Consejo**: Comienza monitoreando las métricas más críticas para tu app (app launch, navegación, APIs) antes de agregar métricas más específicas. Es mejor tener pocas métricas útiles que muchas métricas que no usas.
