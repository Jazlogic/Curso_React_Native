# Clase 1: Fundamentos de Monitoreo 📊

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender los conceptos fundamentales** del monitoreo de aplicaciones móviles
2. **Identificar los diferentes tipos de monitoreo** necesarios para apps React Native
3. **Conocer las herramientas principales** de monitoreo disponibles
4. **Entender la importancia del monitoreo** en el desarrollo de software
5. **Configurar un entorno básico** de monitoreo para React Native

## 🎯 Conceptos Clave

### ¿Qué es el Monitoreo?

El **monitoreo** es el proceso sistemático de observar, medir y analizar el comportamiento de una aplicación en tiempo real para detectar problemas, optimizar performance y mejorar la experiencia del usuario.

```javascript
// Ejemplo conceptual de monitoreo
class AppMonitor {
  constructor() {
    this.metrics = new Map();
    this.alerts = [];
  }

  // Monitorear performance
  trackPerformance(operation, duration) {
    this.metrics.set(operation, {
      duration,
      timestamp: Date.now(),
      count: (this.metrics.get(operation)?.count || 0) + 1
    });
  }

  // Monitorear errores
  trackError(error, context) {
    this.alerts.push({
      type: 'ERROR',
      message: error.message,
      stack: error.stack,
      context,
      timestamp: Date.now()
    });
  }

  // Generar reportes
  generateReport() {
    return {
      metrics: Object.fromEntries(this.metrics),
      alerts: this.alerts,
      summary: {
        totalOperations: this.metrics.size,
        totalErrors: this.alerts.length,
        timestamp: Date.now()
      }
    };
  }
}
```

### Tipos de Monitoreo

#### 1. **Performance Monitoring**
- **Tiempo de respuesta** de operaciones
- **Uso de memoria** y CPU
- **Tiempo de carga** de pantallas
- **Rendimiento** de animaciones

#### 2. **Error Tracking**
- **Crashes** de la aplicación
- **Errores** de JavaScript
- **Errores** de red
- **Errores** de APIs nativas

#### 3. **User Analytics**
- **Comportamiento** del usuario
- **Flujos** de navegación
- **Engagement** con features
- **Retención** de usuarios

#### 4. **Business Metrics**
- **Conversiones** y objetivos
- **Revenue** y transacciones
- **Feature adoption**
- **User satisfaction**

## 🛠️ Herramientas Principales

### Firebase Suite (Google)
```javascript
// Configuración básica de Firebase
import { initializeApp } from '@react-native-firebase/app';
import crashlytics from '@react-native-firebase/crashlytics';
import analytics from '@react-native-firebase/analytics';
import perf from '@react-native-firebase/perf';

const firebaseConfig = {
  apiKey: "tu-api-key",
  authDomain: "tu-proyecto.firebaseapp.com",
  projectId: "tu-proyecto",
  storageBucket: "tu-proyecto.appspot.com",
  messagingSenderId: "123456789",
  appId: "tu-app-id"
};

// Inicializar Firebase
const app = initializeApp(firebaseConfig);

// Habilitar Crashlytics en desarrollo
if (__DEV__) {
  crashlytics().setCrashlyticsCollectionEnabled(false);
} else {
  crashlytics().setCrashlyticsCollectionEnabled(true);
}
```

### Sentry
```javascript
// Configuración de Sentry
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'tu-dsn-de-sentry',
  environment: __DEV__ ? 'development' : 'production',
  debug: __DEV__,
  
  // Configuración de performance
  tracesSampleRate: 1.0,
  
  // Configuración de errores
  beforeSend(event) {
    // Filtrar información sensible
    if (event.user) {
      delete event.user.email;
    }
    return event;
  }
});
```

### Custom Monitoring
```javascript
// Sistema de monitoreo personalizado
class CustomMonitor {
  constructor() {
    this.events = [];
    this.performance = new Map();
    this.errors = [];
  }

  // Trackear eventos personalizados
  trackEvent(name, properties = {}) {
    const event = {
      name,
      properties,
      timestamp: Date.now(),
      sessionId: this.getSessionId()
    };
    
    this.events.push(event);
    this.sendToAnalytics(event);
  }

  // Trackear performance
  trackPerformance(name, startTime) {
    const duration = Date.now() - startTime;
    
    this.performance.set(name, {
      duration,
      timestamp: Date.now(),
      count: (this.performance.get(name)?.count || 0) + 1
    });
  }

  // Trackear errores
  trackError(error, context = {}) {
    const errorEvent = {
      message: error.message,
      stack: error.stack,
      context,
      timestamp: Date.now(),
      sessionId: this.getSessionId()
    };
    
    this.errors.push(errorEvent);
    this.sendToErrorTracking(errorEvent);
  }

  // Obtener ID de sesión
  getSessionId() {
    if (!this.sessionId) {
      this.sessionId = `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    }
    return this.sessionId;
  }

  // Enviar a analytics
  sendToAnalytics(event) {
    // Implementar envío a servicio de analytics
    console.log('Analytics Event:', event);
  }

  // Enviar a error tracking
  sendToErrorTracking(errorEvent) {
    // Implementar envío a servicio de error tracking
    console.log('Error Event:', errorEvent);
  }
}
```

## 🔧 Configuración del Entorno

### 1. **Instalación de Dependencias**
```bash
# Firebase
npm install @react-native-firebase/app
npm install @react-native-firebase/crashlytics
npm install @react-native-firebase/analytics
npm install @react-native-firebase/perf

# Sentry (alternativa)
npm install @sentry/react-native

# Herramientas de desarrollo
npm install --save-dev react-native-flipper
```

### 2. **Configuración de Android**
```gradle
// android/app/build.gradle
dependencies {
    // Firebase
    implementation 'com.google.firebase:firebase-crashlytics:18.4.3'
    implementation 'com.google.firebase:firebase-analytics:21.3.0'
    implementation 'com.google.firebase:firebase-perf:20.4.1'
}

// Aplicar plugin de Firebase
apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.crashlytics'
apply plugin: 'com.google.firebase.firebase-perf'
```

### 3. **Configuración de iOS**
```ruby
# ios/Podfile
target 'TuApp' do
  # Firebase
  pod 'Firebase/Crashlytics'
  pod 'Firebase/Analytics'
  pod 'Firebase/Performance'
end
```

### 4. **Configuración de Metro**
```javascript
// metro.config.js
const { getDefaultConfig } = require('metro-config');

module.exports = (async () => {
  const {
    resolver: { sourceExts, assetExts }
  } = await getDefaultConfig();
  
  return {
    transformer: {
      getTransformOptions: async () => ({
        transform: {
          experimentalImportSupport: false,
          inlineRequires: true,
        },
      }),
    },
    resolver: {
      assetExts: assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...sourceExts, 'svg']
    }
  };
})();
```

## 📊 Métricas Básicas a Monitorear

### Performance Metrics
```javascript
// Métricas de performance básicas
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.startTimes = new Map();
  }

  // Iniciar medición
  startTimer(operation) {
    this.startTimes.set(operation, Date.now());
  }

  // Finalizar medición
  endTimer(operation) {
    const startTime = this.startTimes.get(operation);
    if (startTime) {
      const duration = Date.now() - startTime;
      this.recordMetric(operation, duration);
      this.startTimes.delete(operation);
    }
  }

  // Registrar métrica
  recordMetric(operation, value) {
    if (!this.metrics.has(operation)) {
      this.metrics.set(operation, []);
    }
    
    this.metrics.get(operation).push({
      value,
      timestamp: Date.now()
    });
  }

  // Obtener estadísticas
  getStats(operation) {
    const values = this.metrics.get(operation) || [];
    
    if (values.length === 0) return null;
    
    const sum = values.reduce((acc, v) => acc + v.value, 0);
    const avg = sum / values.length;
    const min = Math.min(...values.map(v => v.value));
    const max = Math.max(...values.map(v => v.value));
    
    return { sum, avg, min, max, count: values.length };
  }
}
```

### Error Metrics
```javascript
// Métricas de errores
class ErrorMonitor {
  constructor() {
    this.errors = [];
    this.errorCounts = new Map();
  }

  // Registrar error
  recordError(error, context = {}) {
    const errorInfo = {
      message: error.message,
      stack: error.stack,
      context,
      timestamp: Date.now(),
      type: error.constructor.name
    };
    
    this.errors.push(errorInfo);
    
    // Contar por tipo
    const errorType = errorInfo.type;
    this.errorCounts.set(errorType, (this.errorCounts.get(errorType) || 0) + 1);
  }

  // Obtener estadísticas de errores
  getErrorStats() {
    return {
      totalErrors: this.errors.length,
      errorCounts: Object.fromEntries(this.errorCounts),
      recentErrors: this.errors.slice(-10), // Últimos 10 errores
      errorRate: this.calculateErrorRate()
    };
  }

  // Calcular tasa de errores
  calculateErrorRate() {
    // Implementar lógica de tasa de errores
    return this.errors.length / 1000; // Por cada 1000 operaciones
  }
}
```

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Configuración Básica de Firebase
**Objetivo**: Configurar Firebase en un proyecto React Native

**Pasos**:
1. Crear proyecto en Firebase Console
2. Descargar archivos de configuración
3. Instalar dependencias necesarias
4. Configurar Crashlytics y Analytics
5. Verificar que los eventos se envíen correctamente

**Código de Verificación**:
```javascript
// Verificar que Firebase esté funcionando
import crashlytics from '@react-native-firebase/crashlytics';
import analytics from '@react-native-firebase/analytics';

// Test de Crashlytics
const testCrashlytics = async () => {
  try {
    await crashlytics().log('Testing Crashlytics integration');
    console.log('✅ Crashlytics funcionando correctamente');
  } catch (error) {
    console.error('❌ Error en Crashlytics:', error);
  }
};

// Test de Analytics
const testAnalytics = async () => {
  try {
    await analytics().logEvent('test_event', {
      test_param: 'test_value'
    });
    console.log('✅ Analytics funcionando correctamente');
  } catch (error) {
    console.error('❌ Error en Analytics:', error);
  }
};
```

### Ejercicio 2: Sistema de Monitoreo Personalizado
**Objetivo**: Crear un sistema básico de monitoreo personalizado

**Requisitos**:
- Trackear eventos personalizados
- Medir performance de operaciones
- Registrar errores con contexto
- Generar reportes básicos

**Implementación Sugerida**:
```javascript
// Implementar la clase CustomMonitor mostrada arriba
// y crear una instancia global para usar en toda la app

const globalMonitor = new CustomMonitor();

// Usar en componentes
const MyComponent = () => {
  useEffect(() => {
    globalMonitor.trackEvent('screen_view', { screen: 'MyComponent' });
    
    const startTime = Date.now();
    globalMonitor.trackPerformance('component_mount', startTime);
    
    return () => {
      globalMonitor.endTimer('component_mount');
    };
  }, []);

  const handleError = (error) => {
    globalMonitor.trackError(error, { 
      component: 'MyComponent',
      action: 'user_interaction' 
    });
  };

  // ... resto del componente
};
```

### Ejercicio 3: Dashboard de Métricas
**Objetivo**: Crear una pantalla que muestre métricas en tiempo real

**Características**:
- Métricas de performance
- Conteo de errores
- Eventos recientes
- Gráficos básicos (opcional)

## 📚 Resumen de la Clase

### **Conceptos Clave Aprendidos**:
1. **Monitoreo** es esencial para detectar problemas y optimizar apps
2. **Tipos de monitoreo**: Performance, Error Tracking, Analytics, Business Metrics
3. **Herramientas principales**: Firebase Suite, Sentry, Custom Solutions
4. **Configuración básica** del entorno de monitoreo
5. **Métricas fundamentales** a trackear

### **Próximos Pasos**:
- Configurar Firebase en tu proyecto
- Implementar sistema básico de monitoreo
- Comenzar a trackear métricas básicas

### **Recursos Adicionales**:
- [Firebase Documentation](https://firebase.google.com/docs)
- [Sentry React Native](https://docs.sentry.io/platforms/react-native/)
- [React Native Performance](https://reactnative.dev/docs/performance)

---

## 🧭 Navegación

- **Clase Anterior**: [Módulo 12: CI/CD](../senior_3/)
- **Clase Siguiente**: [Clase 2: Crashlytics y Error Tracking](clase_2_crashlytics_error_tracking.md)
- **[🏠 Volver al Índice Principal](../../INDICE_COMPLETO.md)**
- **[📚 Volver al README del Módulo](../README.md)**

---

**💡 Consejo**: Comienza implementando el monitoreo básico antes de agregar funcionalidades avanzadas. Es mejor tener métricas simples pero confiables que un sistema complejo que no funciona.
