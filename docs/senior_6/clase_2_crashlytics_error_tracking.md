# 📊 Clase 2: Crashlytics y Error Tracking

## 🎯 Objetivos de la Clase
- Implementar un sistema robusto de captura de errores
- Configurar Crashlytics para análisis detallado de crashes
- Implementar error boundaries avanzados
- Configurar alertas y notificaciones de errores

---

## 📚 Contenido Teórico

### 1. ¿Qué es Crashlytics?

#### **Definición y Propósito**
**Crashlytics** es una herramienta de Firebase que proporciona:
- **Reportes detallados** de crashes en tiempo real
- **Análisis de stack traces** para debugging
- **Agrupación inteligente** de errores similares
- **Métricas de estabilidad** de la aplicación
- **Integración** con otras herramientas de Firebase

#### **Ventajas de Crashlytics:**
- **Tiempo real**: Los crashes se reportan inmediatamente
- **Detalles completos**: Stack traces, logs, contexto del dispositivo
- **Agrupación automática**: Errores similares se agrupan automáticamente
- **Filtros avanzados**: Búsqueda por versión, dispositivo, usuario, etc.
- **Integración**: Con Analytics, Performance, y Remote Config

### 2. Configuración Avanzada de Crashlytics

#### **Configuración de Android**
```gradle
// android/app/build.gradle
android {
    buildTypes {
        debug {
            // Habilitar Crashlytics en debug para testing
            firebaseCrashlytics {
                mappingFileUploadEnabled false
            }
        }
        
        release {
            // Configuración para producción
            firebaseCrashlytics {
                mappingFileUploadEnabled true
                nativeSymbolUploadEnabled true
                unstrippedNativeLibsDir 'build/intermediates/merged_native_libs/release/out/lib'
            }
        }
    }
}

dependencies {
    implementation 'com.google.firebase:firebase-crashlytics:18.5.1'
    implementation 'com.google.firebase:firebase-analytics:21.3.0'
}
```

#### **Configuración de iOS**
```ruby
# ios/Podfile
target 'YourApp' do
  pod 'Firebase/Crashlytics'
  pod 'Firebase/Analytics'
  
  # Configuración para diferentes configuraciones
  pod 'Firebase/Crashlytics', :configuration => ['Debug', 'Release']
end
```

```swift
// ios/YourApp/AppDelegate.swift
import Firebase
import FirebaseCrashlytics

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        FirebaseApp.configure()
        
        // Configuración adicional de Crashlytics
        Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
        
        return true
    }
}
```

### 3. Implementación de Error Boundaries Avanzados

#### **Error Boundary con Contexto Detallado**
```jsx
// components/AdvancedErrorBoundary.js
import React from 'react';
import { View, Text, Button, ScrollView, StyleSheet } from 'react-native';
import { crashlytics } from '../config/firebase';

class AdvancedErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null, 
      errorInfo: null,
      errorId: null 
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Generar ID único para el error
    const errorId = `error_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    // Registrar en Crashlytics con contexto detallado
    crashlytics().recordError(error);
    
    // Agregar atributos para mejor análisis
    crashlytics().setAttribute('error_boundary', 'advanced');
    crashlytics().setAttribute('error_id', errorId);
    crashlytics().setAttribute('component_name', this.props.componentName || 'Unknown');
    crashlytics().setAttribute('error_type', error.name);
    crashlytics().setAttribute('error_message', error.message);
    crashlytics().setAttribute('component_stack', errorInfo.componentStack);
    
    // Registrar información del usuario si está disponible
    if (this.props.userId) {
      crashlytics().setUserId(this.props.userId);
    }
    
    // Registrar logs adicionales
    crashlytics().log(`Error occurred in ${this.props.componentName || 'Unknown component'}`);
    crashlytics().log(`Error details: ${error.stack}`);
    
    this.setState({ 
      error, 
      errorInfo, 
      errorId 
    });
    
    // Notificar al equipo (opcional)
    this.notifyTeam(error, errorId);
  }

  notifyTeam = (error, errorId) => {
    // Implementar notificación al equipo
    console.log(`Critical error occurred: ${errorId}`);
    // Aquí podrías enviar un email, Slack notification, etc.
  };

  resetError = () => {
    this.setState({ 
      hasError: false, 
      error: null, 
      errorInfo: null,
      errorId: null 
    });
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorTitle}>Algo salió mal 😕</Text>
          <Text style={styles.errorSubtitle}>
            Error ID: {this.state.errorId}
          </Text>
          
          <ScrollView style={styles.errorDetails}>
            <Text style={styles.errorText}>
              {this.state.error?.message || 'Error desconocido'}
            </Text>
            
            {__DEV__ && this.state.errorInfo && (
              <Text style={styles.stackTrace}>
                {this.state.errorInfo.componentStack}
              </Text>
            )}
          </ScrollView>
          
          <View style={styles.buttonContainer}>
            <Button 
              title="Reintentar" 
              onPress={this.resetError}
              color="#007AFF"
            />
            <Button 
              title="Reportar Error" 
              onPress={() => this.reportError()}
              color="#FF3B30"
            />
          </View>
        </View>
      );
    }

    return this.props.children;
  }

  reportError = () => {
    // Implementar reporte manual del error
    crashlytics().log('User manually reported error');
    alert('Error reportado. Gracias por tu feedback.');
  };
}

const styles = StyleSheet.create({
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#F8F9FA'
  },
  errorTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#FF3B30'
  },
  errorSubtitle: {
    fontSize: 16,
    marginBottom: 20,
    color: '#8E8E93'
  },
  errorDetails: {
    maxHeight: 200,
    marginBottom: 20
  },
  errorText: {
    fontSize: 16,
    marginBottom: 10,
    color: '#000'
  },
  stackTrace: {
    fontSize: 12,
    color: '#8E8E93',
    fontFamily: 'monospace'
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 10
  }
});

export default AdvancedErrorBoundary;
```

#### **Error Boundary Específico para Navegación**
```jsx
// components/NavigationErrorBoundary.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { crashlytics } from '../config/firebase';

class NavigationErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, currentRoute: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Registrar error específico de navegación
    crashlytics().recordError(error);
    crashlytics().setAttribute('error_category', 'navigation');
    crashlytics().setAttribute('current_route', this.state.currentRoute || 'unknown');
    crashlytics().setAttribute('navigation_stack', JSON.stringify(this.props.navigationState));
    
    // Log adicional para debugging
    crashlytics().log(`Navigation error on route: ${this.state.currentRoute}`);
  }

  onNavigationStateChange = (state) => {
    if (state?.routes?.length > 0) {
      const currentRoute = state.routes[state.index];
      this.setState({ currentRoute: currentRoute.name });
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          <Text>Error de navegación detectado</Text>
          <Text>Ruta: {this.state.currentRoute}</Text>
          <Button 
            title="Reiniciar Navegación" 
            onPress={() => this.setState({ hasError: false })} 
          />
        </View>
      );
    }

    return (
      <NavigationContainer onStateChange={this.onNavigationStateChange}>
        {this.props.children}
      </NavigationContainer>
    );
  }
}

export default NavigationErrorBoundary;
```

### 4. Captura de Errores Específicos

#### **Manejo de Errores de API**
```javascript
// services/errorTracking.js
import { crashlytics } from '../config/firebase';

export class ErrorTracker {
  static trackApiError(error, context = {}) {
    // Registrar error de API
    crashlytics().recordError(error);
    
    // Agregar contexto específico de API
    crashlytics().setAttribute('error_category', 'api');
    crashlytics().setAttribute('api_url', context.url || 'unknown');
    crashlytics().setAttribute('api_method', context.method || 'unknown');
    crashlytics().setAttribute('api_status', context.status || 'unknown');
    crashlytics().setAttribute('api_response_time', context.responseTime || 'unknown');
    
    // Agregar información del usuario si está disponible
    if (context.userId) {
      crashlytics().setUserId(context.userId);
    }
    
    // Log adicional
    crashlytics().log(`API Error: ${error.message} on ${context.url}`);
  }

  static trackNetworkError(error, context = {}) {
    crashlytics().recordError(error);
    crashlytics().setAttribute('error_category', 'network');
    crashlytics().setAttribute('network_type', context.networkType || 'unknown');
    crashlytics().setAttribute('connection_quality', context.connectionQuality || 'unknown');
    
    crashlytics().log(`Network Error: ${error.message}`);
  }

  static trackRenderError(error, context = {}) {
    crashlytics().recordError(error);
    crashlytics().setAttribute('error_category', 'render');
    crashlytics().setAttribute('component_name', context.componentName || 'unknown');
    crashlytics().setAttribute('screen_name', context.screenName || 'unknown');
    
    crashlytics().log(`Render Error in ${context.componentName}: ${error.message}`);
  }

  static trackAsyncError(error, context = {}) {
    crashlytics().recordError(error);
    crashlytics().setAttribute('error_category', 'async');
    crashlytics().setAttribute('operation_type', context.operationType || 'unknown');
    crashlytics().setAttribute('operation_duration', context.duration || 'unknown');
    
    crashlytics().log(`Async Error in ${context.operationType}: ${error.message}`);
  }
}

// Uso en servicios de API
export const apiService = {
  async makeRequest(url, options = {}) {
    const startTime = Date.now();
    
    try {
      const response = await fetch(url, options);
      const responseTime = Date.now() - startTime;
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      // Trackear error con contexto completo
      ErrorTracker.trackApiError(error, {
        url,
        method: options.method || 'GET',
        status: error.status,
        responseTime,
        userId: getCurrentUserId() // Función que obtiene el ID del usuario actual
      });
      
      throw error;
    }
  }
};
```

### 5. Configuración de Alertas y Notificaciones

#### **Configuración de Alertas en Firebase Console**
1. **Ir a Firebase Console > Crashlytics**
2. **Configurar alertas** para diferentes tipos de errores
3. **Establecer umbrales** para notificaciones
4. **Configurar canales** de notificación (email, Slack, etc.)

#### **Alertas Automáticas por Severidad**
```javascript
// services/alertService.js
export class AlertService {
  static async checkErrorSeverity(error, context = {}) {
    // Determinar severidad del error
    const severity = this.calculateSeverity(error, context);
    
    if (severity === 'critical') {
      await this.sendCriticalAlert(error, context);
    } else if (severity === 'high') {
      await this.sendHighPriorityAlert(error, context);
    }
  }

  static calculateSeverity(error, context) {
    // Lógica para determinar severidad
    if (error.message.includes('crash') || error.message.includes('fatal')) {
      return 'critical';
    }
    
    if (context.affectsUserData || context.affectsPayment) {
      return 'high';
    }
    
    return 'medium';
  }

  static async sendCriticalAlert(error, context) {
    // Enviar alerta crítica al equipo
    console.log('🚨 CRITICAL ERROR ALERT 🚨');
    console.log('Error:', error.message);
    console.log('Context:', context);
    
    // Aquí implementarías el envío real de la alerta
    // Por ejemplo: Slack webhook, email, SMS, etc.
  }

  static async sendHighPriorityAlert(error, context) {
    console.log('⚠️ HIGH PRIORITY ERROR ⚠️');
    console.log('Error:', error.message);
    console.log('Context:', context);
  }
}
```

### 6. Análisis y Reportes de Errores

#### **Dashboard de Errores Personalizado**
```javascript
// services/errorAnalytics.js
export class ErrorAnalytics {
  static async getErrorSummary(timeRange = '7d') {
    // Obtener resumen de errores del período
    const errors = await this.fetchErrors(timeRange);
    
    return {
      totalErrors: errors.length,
      criticalErrors: errors.filter(e => e.severity === 'critical').length,
      highPriorityErrors: errors.filter(e => e.severity === 'high').length,
      mostCommonErrors: this.getMostCommonErrors(errors),
      errorTrends: this.calculateErrorTrends(errors),
      affectedUsers: this.getAffectedUsersCount(errors)
    };
  }

  static async getMostCommonErrors(errors) {
    const errorCounts = {};
    
    errors.forEach(error => {
      const key = `${error.type}:${error.message}`;
      errorCounts[key] = (errorCounts[key] || 0) + 1;
    });
    
    return Object.entries(errorCounts)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 10)
      .map(([key, count]) => {
        const [type, message] = key.split(':');
        return { type, message, count };
      });
  }

  static calculateErrorTrends(errors) {
    // Calcular tendencias de errores por día
    const dailyErrors = {};
    
    errors.forEach(error => {
      const date = new Date(error.timestamp).toDateString();
      dailyErrors[date] = (dailyErrors[date] || 0) + 1;
    });
    
    return dailyErrors;
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Error Boundary Avanzado
Implementa un error boundary que capture diferentes tipos de errores:

**Requisitos:**
- Captura de errores de renderizado
- Captura de errores de navegación
- Captura de errores de API
- Reporte detallado a Crashlytics

**Implementación:**
```jsx
class ComprehensiveErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Categorizar el error
    const errorCategory = this.categorizeError(error, errorInfo);
    
    // Registrar en Crashlytics con contexto completo
    crashlytics().recordError(error);
    crashlytics().setAttribute('error_category', errorCategory);
    crashlytics().setAttribute('error_boundary_type', 'comprehensive');
    
    // Agregar información del dispositivo
    this.addDeviceInfo();
    
    // Agregar información del usuario
    this.addUserInfo();
  }
  
  categorizeError(error, errorInfo) {
    if (errorInfo.componentStack.includes('Navigation')) {
      return 'navigation';
    } else if (error.message.includes('API')) {
      return 'api';
    } else if (error.message.includes('Render')) {
      return 'render';
    }
    return 'unknown';
  }
}
```

### Ejercicio 2: Sistema de Alertas de Errores
Implementa un sistema que envíe alertas automáticas:

**Requisitos:**
- Clasificación de severidad de errores
- Alertas automáticas por email/Slack
- Dashboard de resumen de errores
- Notificaciones en tiempo real

**Implementación:**
```javascript
class ErrorAlertSystem {
  static async processError(error, context) {
    const severity = this.assessSeverity(error, context);
    
    if (severity === 'critical') {
      await this.sendImmediateAlert(error, context);
      await this.createIncident(error, context);
    }
    
    // Registrar para análisis posterior
    await this.logError(error, context, severity);
  }
  
  static assessSeverity(error, context) {
    let score = 0;
    
    if (error.message.includes('crash')) score += 10;
    if (context.affectsUserData) score += 8;
    if (context.affectsPayment) score += 7;
    if (context.userCount > 100) score += 5;
    
    if (score >= 15) return 'critical';
    if (score >= 10) return 'high';
    if (score >= 5) return 'medium';
    return 'low';
  }
}
```

---

## 🔍 Puntos Clave

1. **Crashlytics** proporciona análisis detallado de crashes en tiempo real
2. **Error boundaries** son esenciales para capturar errores de React
3. **Contexto detallado** mejora significativamente el debugging
4. **Alertas automáticas** permiten respuesta rápida a problemas críticos
5. **Análisis de tendencias** ayuda a identificar patrones de errores

---

## 📖 Recursos Adicionales

- [Firebase Crashlytics Documentation](https://firebase.google.com/docs/crashlytics)
- [React Error Boundaries](https://reactjs.org/docs/error-boundaries.html)
- [Error Tracking Best Practices](https://sentry.io/for/react-native/)
- [Firebase Console](https://console.firebase.google.com/)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Performance Monitoring** y cómo implementar monitoreo avanzado de rendimiento para optimizar la experiencia del usuario.
