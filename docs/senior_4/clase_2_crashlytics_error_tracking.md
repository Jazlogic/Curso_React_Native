# Clase 2: Crashlytics y Error Tracking 🚨

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Configurar Firebase Crashlytics** en aplicaciones React Native
2. **Implementar error tracking** personalizado y automático
3. **Configurar reportes de crashes** con contexto detallado
4. **Integrar Crashlytics** con sistemas de monitoreo existentes
5. **Analizar y resolver** problemas reportados por Crashlytics

## 🎯 Conceptos Clave

### ¿Qué es Crashlytics?

**Firebase Crashlytics** es una herramienta de Google que ayuda a detectar, priorizar y resolver problemas de estabilidad que afectan negativamente la experiencia del usuario.

```javascript
// Ejemplo básico de integración de Crashlytics
import crashlytics from '@react-native-firebase/crashlytics';

class CrashlyticsService {
  constructor() {
    this.isEnabled = !__DEV__; // Solo en producción
    this.userContext = {};
  }

  // Habilitar/deshabilitar Crashlytics
  setEnabled(enabled) {
    this.isEnabled = enabled;
    crashlytics().setCrashlyticsCollectionEnabled(enabled);
  }

  // Configurar contexto del usuario
  setUserContext(userId, email, displayName) {
    this.userContext = { userId, email, displayName };
    
    if (this.isEnabled) {
      crashlytics().setUserId(userId);
      crashlytics().setUserEmail(email);
      crashlytics().setUserName(displayName);
    }
  }

  // Registrar error no fatal
  logError(error, context = {}) {
    if (this.isEnabled) {
      crashlytics().log(`Error: ${error.message}`);
      crashlytics().recordError(error);
      
      // Agregar contexto adicional
      Object.entries(context).forEach(([key, value]) => {
        crashlytics().setAttribute(key, String(value));
      });
    }
  }

  // Registrar crash fatal
  logFatalError(error, context = {}) {
    if (this.isEnabled) {
      // Agregar contexto antes del crash
      Object.entries(context).forEach(([key, value]) => {
        crashlytics().setAttribute(key, String(value));
      });
      
      // Registrar el error
      crashlytics().recordError(error);
      
      // Forzar el envío
      crashlytics().sendUnsentReports();
    }
  }
}
```

### Tipos de Errores a Trackear

#### 1. **JavaScript Errors**
```javascript
// Capturar errores de JavaScript
class JavaScriptErrorHandler {
  constructor(crashlyticsService) {
    this.crashlytics = crashlyticsService;
    this.setupGlobalErrorHandling();
  }

  // Configurar manejo global de errores
  setupGlobalErrorHandling() {
    // Capturar errores no manejados
    const originalOnError = global.ErrorUtils.setGlobalHandler;
    
    global.ErrorUtils.setGlobalHandler = (error, isFatal) => {
      // Log del error
      console.error('Global Error:', error, 'Fatal:', isFatal);
      
      // Enviar a Crashlytics
      this.crashlytics.logError(error, {
        isFatal,
        source: 'global_error_handler',
        timestamp: Date.now()
      });
      
      // Llamar al handler original si existe
      if (originalOnError) {
        originalOnError(error, isFatal);
      }
    };
  }

  // Capturar errores de promesas rechazadas
  setupPromiseErrorHandling() {
    const originalUnhandledRejection = global.onunhandledrejection;
    
    global.onunhandledrejection = (event) => {
      const error = event.reason;
      
      this.crashlytics.logError(error, {
        source: 'unhandled_promise_rejection',
        timestamp: Date.now()
      });
      
      if (originalUnhandledRejection) {
        originalUnhandledRejection(event);
      }
    };
  }
}
```

#### 2. **Network Errors**
```javascript
// Capturar errores de red
class NetworkErrorHandler {
  constructor(crashlyticsService) {
    this.crashlytics = crashlyticsService;
    this.setupNetworkErrorHandling();
  }

  // Configurar interceptores para Axios
  setupAxiosErrorHandling(axiosInstance) {
    axiosInstance.interceptors.response.use(
      (response) => response,
      (error) => {
        this.handleNetworkError(error);
        return Promise.reject(error);
      }
    );
  }

  // Manejar errores de red
  handleNetworkError(error) {
    const errorContext = {
      source: 'network_request',
      url: error.config?.url,
      method: error.config?.method,
      status: error.response?.status,
      statusText: error.response?.statusText,
      responseData: error.response?.data,
      timestamp: Date.now()
    };

    this.crashlytics.logError(error, errorContext);
  }

  // Capturar errores de fetch
  setupFetchErrorHandling() {
    const originalFetch = global.fetch;
    
    global.fetch = async (...args) => {
      try {
        const response = await originalFetch(...args);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return response;
      } catch (error) {
        this.handleNetworkError(error);
        throw error;
      }
    };
  }
}
```

#### 3. **Native Errors**
```javascript
// Capturar errores de APIs nativas
class NativeErrorHandler {
  constructor(crashlyticsService) {
    this.crashlytics = crashlyticsService;
  }

  // Wrapper para APIs nativas con manejo de errores
  async wrapNativeCall(nativeFunction, context = {}) {
    try {
      const result = await nativeFunction();
      return result;
    } catch (error) {
      this.handleNativeError(error, context);
      throw error;
    }
  }

  // Manejar errores nativos
  handleNativeError(error, context) {
    const errorContext = {
      source: 'native_api',
      ...context,
      timestamp: Date.now()
    };

    this.crashlytics.logError(error, errorContext);
  }

  // Ejemplo de uso con cámara
  async takePhoto() {
    return this.wrapNativeCall(
      async () => {
        const { launchCamera } = require('react-native-image-picker');
        return launchCamera({
          mediaType: 'photo',
          quality: 0.8
        });
      },
      { api: 'camera', action: 'take_photo' }
    );
  }
}
```

## 🛠️ Configuración Avanzada de Crashlytics

### Configuración de Android
```gradle
// android/app/build.gradle
android {
    defaultConfig {
        // Configuración de Crashlytics
        manifestPlaceholders = [
            crashlyticsCollectionEnabled: "true"
        ]
    }
    
    buildTypes {
        debug {
            // Deshabilitar en debug
            manifestPlaceholders = [
                crashlyticsCollectionEnabled: "false"
            ]
        }
        release {
            // Habilitar en release
            manifestPlaceholders = [
                crashlyticsCollectionEnabled: "true"
            ]
        }
    }
}

dependencies {
    // Crashlytics
    implementation 'com.google.firebase:firebase-crashlytics:18.4.3'
    implementation 'com.google.firebase:firebase-analytics:21.3.0'
}

// Aplicar plugins
apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.crashlytics'
```

### Configuración de iOS
```ruby
# ios/Podfile
target 'TuApp' do
  # Crashlytics
  pod 'Firebase/Crashlytics'
  pod 'Firebase/Analytics'
  
  # Configuración de build
  post_install do |installer|
    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        config.build_settings['ENABLE_BITCODE'] = 'NO'
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '12.0'
      end
    end
  end
end
```

### Configuración de JavaScript
```javascript
// Configuración avanzada de Crashlytics
import crashlytics from '@react-native-firebase/crashlytics';

class AdvancedCrashlyticsService {
  constructor() {
    this.setupAdvancedConfiguration();
  }

  // Configuración avanzada
  async setupAdvancedConfiguration() {
    try {
      // Configurar envío automático
      await crashlytics().setCrashlyticsCollectionEnabled(true);
      
      // Configurar atributos por defecto
      await crashlytics().setAttribute('app_version', this.getAppVersion());
      await crashlytics().setAttribute('build_number', this.getBuildNumber());
      await crashlytics().setAttribute('device_model', this.getDeviceModel());
      await crashlytics().setAttribute('os_version', this.getOSVersion());
      
      // Configurar logs personalizados
      await crashlytics().log('Crashlytics inicializado correctamente');
      
    } catch (error) {
      console.error('Error configurando Crashlytics:', error);
    }
  }

  // Obtener versión de la app
  getAppVersion() {
    // Implementar lógica para obtener versión
    return '1.0.0';
  }

  // Obtener número de build
  getBuildNumber() {
    // Implementar lógica para obtener build number
    return '1';
  }

  // Obtener modelo del dispositivo
  getDeviceModel() {
    // Implementar lógica para obtener modelo
    return 'Unknown';
  }

  // Obtener versión del OS
  getOSVersion() {
    // Implementar lógica para obtener versión del OS
    return 'Unknown';
  }
}
```

## 📊 Reportes y Análisis

### Estructura de Reportes
```javascript
// Estructura de reportes de errores
class ErrorReportGenerator {
  constructor(crashlyticsService) {
    this.crashlytics = crashlyticsService;
  }

  // Generar reporte detallado de error
  generateErrorReport(error, context = {}) {
    const report = {
      // Información del error
      error: {
        message: error.message,
        stack: error.stack,
        name: error.name,
        code: error.code
      },
      
      // Contexto de la aplicación
      app: {
        version: this.getAppVersion(),
        build: this.getBuildNumber(),
        timestamp: Date.now()
      },
      
      // Contexto del dispositivo
      device: {
        model: this.getDeviceModel(),
        os: this.getOSVersion(),
        memory: this.getMemoryInfo(),
        battery: this.getBatteryInfo()
      },
      
      // Contexto del usuario
      user: {
        id: this.getUserId(),
        session: this.getSessionId(),
        screen: this.getCurrentScreen()
      },
      
      // Contexto adicional
      context: {
        ...context,
        networkStatus: this.getNetworkStatus(),
        appState: this.getAppState()
      }
    };

    return report;
  }

  // Enviar reporte a Crashlytics
  async sendErrorReport(error, context = {}) {
    try {
      const report = this.generateErrorReport(error, context);
      
      // Enviar a Crashlytics
      await this.crashlytics.logError(error);
      
      // Agregar atributos personalizados
      Object.entries(report.context).forEach(([key, value]) => {
        this.crashlytics.setAttribute(key, String(value));
      });
      
      // Forzar envío
      await this.crashlytics.sendUnsentReports();
      
      return true;
    } catch (sendError) {
      console.error('Error enviando reporte:', sendError);
      return false;
    }
  }
}
```

### Dashboard de Errores
```javascript
// Dashboard para visualizar errores
class ErrorDashboard {
  constructor() {
    this.errors = [];
    this.errorStats = {};
  }

  // Agregar error al dashboard
  addError(error, context = {}) {
    const errorEntry = {
      id: this.generateErrorId(),
      error,
      context,
      timestamp: Date.now(),
      count: 1
    };

    // Verificar si es un error duplicado
    const existingError = this.findSimilarError(error);
    if (existingError) {
      existingError.count++;
      existingError.lastOccurrence = Date.now();
    } else {
      this.errors.push(errorEntry);
    }

    this.updateStats();
  }

  // Encontrar error similar
  findSimilarError(error) {
    return this.errors.find(existing => 
      existing.error.message === error.message &&
      existing.error.name === error.name
    );
  }

  // Generar ID único para error
  generateErrorId() {
    return `error_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  // Actualizar estadísticas
  updateStats() {
    this.errorStats = {
      totalErrors: this.errors.length,
      totalOccurrences: this.errors.reduce((sum, e) => sum + e.count, 0),
      errorsByType: this.groupErrorsByType(),
      errorsBySource: this.groupErrorsBySource(),
      recentErrors: this.errors.slice(-10)
    };
  }

  // Agrupar errores por tipo
  groupErrorsByType() {
    const groups = {};
    this.errors.forEach(error => {
      const type = error.error.name || 'Unknown';
      groups[type] = (groups[type] || 0) + error.count;
    });
    return groups;
  }

  // Agrupar errores por fuente
  groupErrorsBySource() {
    const groups = {};
    this.errors.forEach(error => {
      const source = error.context.source || 'Unknown';
      groups[source] = (groups[source] || 0) + error.count;
    });
    return groups;
  }

  // Obtener estadísticas
  getStats() {
    return this.errorStats;
  }
}
```

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Configuración Completa de Crashlytics
**Objetivo**: Configurar Crashlytics con manejo avanzado de errores

**Requisitos**:
1. Configurar Crashlytics en Android e iOS
2. Implementar manejo global de errores
3. Configurar contexto de usuario
4. Implementar reportes personalizados

**Implementación**:
```javascript
// Implementar las clases CrashlyticsService, JavaScriptErrorHandler,
// NetworkErrorHandler y NativeErrorHandler mostradas arriba

// Crear instancia global
const crashlyticsService = new CrashlyticsService();
const jsErrorHandler = new JavaScriptErrorHandler(crashlyticsService);
const networkErrorHandler = new NetworkErrorHandler(crashlyticsService);
const nativeErrorHandler = new NativeErrorHandler(crashlyticsService);

// Configurar en App.js
const App = () => {
  useEffect(() => {
    // Configurar contexto del usuario
    crashlyticsService.setUserContext('user123', 'user@example.com', 'John Doe');
    
    // Configurar manejo de errores
    jsErrorHandler.setupGlobalErrorHandling();
    jsErrorHandler.setupPromiseErrorHandling();
    networkErrorHandler.setupFetchErrorHandling();
  }, []);

  // ... resto del componente
};
```

### Ejercicio 2: Sistema de Reportes Personalizados
**Objetivo**: Crear sistema de reportes de errores con contexto detallado

**Requisitos**:
- Generar reportes estructurados
- Incluir contexto de app, dispositivo y usuario
- Enviar reportes a Crashlytics
- Crear dashboard de errores

**Implementación**:
```javascript
// Implementar ErrorReportGenerator y ErrorDashboard

// Crear instancia global
const errorReportGenerator = new ErrorReportGenerator(crashlyticsService);
const errorDashboard = new ErrorDashboard();

// Usar en componentes
const MyComponent = () => {
  const handleError = async (error) => {
    const context = {
      component: 'MyComponent',
      action: 'user_interaction',
      screen: 'HomeScreen'
    };

    // Agregar al dashboard
    errorDashboard.addError(error, context);
    
    // Enviar a Crashlytics
    await errorReportGenerator.sendErrorReport(error, context);
  };

  // ... resto del componente
};
```

### Ejercicio 3: Dashboard de Errores en Tiempo Real
**Objetivo**: Crear pantalla que muestre errores y estadísticas

**Características**:
- Lista de errores recientes
- Estadísticas por tipo y fuente
- Filtros y búsqueda
- Gráficos de tendencias

## 📚 Resumen de la Clase

### **Conceptos Clave Aprendidos**:
1. **Crashlytics** es esencial para tracking de errores en producción
2. **Tipos de errores**: JavaScript, Network, Native
3. **Configuración avanzada** para Android, iOS y JavaScript
4. **Reportes estructurados** con contexto detallado
5. **Dashboard de errores** para análisis en tiempo real

### **Próximos Pasos**:
- Configurar Crashlytics en tu proyecto
- Implementar manejo global de errores
- Crear sistema de reportes personalizados
- Desarrollar dashboard de errores

### **Recursos Adicionales**:
- [Firebase Crashlytics Documentation](https://firebase.google.com/docs/crashlytics)
- [React Native Error Handling](https://reactnative.dev/docs/error-boundaries)
- [JavaScript Error Handling Best Practices](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)

---

## 🧭 Navegación

- **Clase Anterior**: [Clase 1: Fundamentos de Monitoreo](clase_1_fundamentos_monitoreo.md)
- **Clase Siguiente**: [Clase 3: Performance Monitoring](clase_3_performance_monitoring.md)
- **[🏠 Volver al Índice Principal](../../INDICE_COMPLETO.md)**
- **[📚 Volver al README del Módulo](../README.md)**

---

**💡 Consejo**: Configura Crashlytics desde el inicio del proyecto. Es más fácil implementar el manejo de errores desde el principio que agregarlo después cuando ya tienes una app en producción.
