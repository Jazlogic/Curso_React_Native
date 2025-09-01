# 📊 Clase 1: Fundamentos de Monitoreo y Analytics

## 🎯 Objetivos de la Clase
- Entender la importancia del monitoreo en aplicaciones móviles
- Conocer las herramientas de monitoreo disponibles para React Native
- Implementar monitoreo básico de rendimiento y errores
- Configurar analytics para seguimiento de usuarios

---

## 📚 Contenido Teórico

### 1. ¿Por qué es importante el Monitoreo?

#### **Desafíos de las Aplicaciones Móviles:**
- **Entornos variables**: Diferentes dispositivos, sistemas operativos, conexiones
- **Experiencia offline**: Aplicaciones deben funcionar sin conexión
- **Rendimiento crítico**: Los usuarios abandonan apps lentas
- **Escalabilidad**: Miles o millones de usuarios simultáneos
- **Actualizaciones**: Nuevas versiones pueden introducir bugs

#### **Beneficios del Monitoreo:**
- **Detección temprana** de problemas
- **Mejora continua** del rendimiento
- **Experiencia de usuario** optimizada
- **Reducción de costos** de soporte
- **Toma de decisiones** basada en datos

### 2. Tipos de Monitoreo

#### **Monitoreo de Rendimiento (Performance Monitoring)**
- **Tiempo de carga** de pantallas
- **Rendimiento de red** (API calls)
- **Uso de memoria** y CPU
- **Tiempo de respuesta** de la UI
- **FPS (Frames por segundo)**

#### **Monitoreo de Errores (Error Monitoring)**
- **Crashes** de la aplicación
- **Errores de JavaScript** no capturados
- **Errores de red** y timeouts
- **Errores de renderizado** de componentes
- **Problemas de compatibilidad** entre plataformas

#### **Monitoreo de Usuarios (User Analytics)**
- **Comportamiento** del usuario
- **Flujos de navegación** más comunes
- **Funcionalidades** más utilizadas
- **Puntos de abandono** de la app
- **Métricas de retención** y engagement

### 3. Herramientas de Monitoreo para React Native

#### **Firebase (Google)**
- **Crashlytics**: Reportes de crashes detallados
- **Performance Monitoring**: Métricas de rendimiento
- **Analytics**: Comportamiento del usuario
- **Remote Config**: Configuración remota
- **A/B Testing**: Experimentos con usuarios

#### **Sentry**
- **Error Tracking**: Captura de errores en tiempo real
- **Performance Monitoring**: Métricas de rendimiento
- **Release Tracking**: Seguimiento de versiones
- **User Feedback**: Comentarios de usuarios
- **Issue Management**: Gestión de problemas

#### **Mixpanel**
- **Event Tracking**: Seguimiento de eventos personalizados
- **User Analytics**: Análisis de comportamiento
- **Funnel Analysis**: Análisis de conversión
- **Cohort Analysis**: Análisis de retención
- **A/B Testing**: Experimentos controlados

### 4. Configuración Básica de Firebase

#### **Instalación de Dependencias**
```bash
# Instalar Firebase
npm install @react-native-firebase/app
npm install @react-native-firebase/analytics
npm install @react-native-firebase/crashlytics
npm install @react-native-firebase/perf

# Para iOS (CocoaPods)
cd ios && pod install
```

#### **Configuración de Android**
```gradle
// android/app/build.gradle
dependencies {
    implementation platform('com.google.firebase:firebase-bom:32.2.0')
    implementation 'com.google.firebase:firebase-analytics'
    implementation 'com.google.firebase:firebase-crashlytics'
    implementation 'com.google.firebase:firebase-perf'
}

apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.crashlytics'
apply plugin: 'com.google.firebase.firebase-perf'
```

#### **Configuración de iOS**
```ruby
# ios/Podfile
target 'YourApp' do
  pod 'Firebase/Analytics'
  pod 'Firebase/Crashlytics'
  pod 'Firebase/Performance'
end
```

### 5. Implementación Básica de Analytics

#### **Configuración Inicial**
```javascript
// config/firebase.js
import { initializeApp } from '@react-native-firebase/app';
import analytics from '@react-native-firebase/analytics';
import crashlytics from '@react-native-firebase/crashlytics';
import perf from '@react-native-firebase/perf';

// Configuración de Firebase
const firebaseConfig = {
  apiKey: "tu-api-key",
  authDomain: "tu-app.firebaseapp.com",
  projectId: "tu-app",
  storageBucket: "tu-app.appspot.com",
  messagingSenderId: "123456789",
  appId: "tu-app-id"
};

// Inicializar Firebase
const app = initializeApp(firebaseConfig);

export { analytics, crashlytics, perf };
```

#### **Tracking de Pantallas**
```javascript
// hooks/useAnalytics.js
import { useEffect } from 'react';
import { analytics } from '../config/firebase';

export const useScreenTracking = (screenName, screenClass) => {
  useEffect(() => {
    // Registrar vista de pantalla
    analytics().logScreenView({
      screen_name: screenName,
      screen_class: screenClass
    });
  }, [screenName, screenClass]);
};

// Uso en componentes
export const useAnalytics = () => {
  const logEvent = (eventName, parameters = {}) => {
    analytics().logEvent(eventName, parameters);
  };

  const logScreenView = (screenName, screenClass) => {
    analytics().logScreenView({
      screen_name: screenName,
      screen_class: screenClass
    });
  };

  return { logEvent, logScreenView };
};
```

### 6. Monitoreo de Errores Básico

#### **Error Boundary con Crashlytics**
```jsx
// components/ErrorBoundary.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { crashlytics } from '../config/firebase';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Registrar error en Crashlytics
    crashlytics().recordError(error);
    
    // Agregar contexto adicional
    crashlytics().setAttribute('error_boundary', 'true');
    crashlytics().setAttribute('component_stack', errorInfo.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          <Text>Algo salió mal 😕</Text>
          <Button 
            title="Reintentar" 
            onPress={() => this.setState({ hasError: false })} 
          />
        </View>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

#### **Manejo de Errores de Red**
```javascript
// services/api.js
import { crashlytics } from '../config/firebase';

export const apiService = {
  async makeRequest(url, options = {}) {
    try {
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      // Registrar error en Crashlytics
      crashlytics().recordError(error);
      
      // Agregar contexto
      crashlytics().setAttribute('api_url', url);
      crashlytics().setAttribute('api_method', options.method || 'GET');
      crashlytics().setAttribute('api_status', error.message);
      
      throw error;
    }
  }
};
```

### 7. Monitoreo de Rendimiento Básico

#### **Performance Monitoring**
```javascript
// hooks/usePerformance.js
import { useEffect } from 'react';
import { perf } from '../config/firebase';

export const usePerformanceMonitoring = (screenName) => {
  useEffect(() => {
    // Crear trace de rendimiento
    const trace = perf().newTrace('screen_load');
    trace.putAttribute('screen_name', screenName);
    
    // Iniciar medición
    trace.start();
    
    return () => {
      // Detener medición cuando el componente se desmonta
      trace.stop();
    };
  }, [screenName]);
};

// Monitoreo de operaciones de red
export const useNetworkMonitoring = () => {
  const monitorRequest = async (url, method, requestData) => {
    const metric = perf().newHttpMetric(url, method);
    
    try {
      metric.start();
      
      const response = await fetch(url, {
        method,
        body: requestData ? JSON.stringify(requestData) : undefined,
        headers: {
          'Content-Type': 'application/json',
        },
      });
      
      metric.setHttpResponseCode(response.status);
      metric.setResponseContentType(response.headers.get('content-type'));
      
      return response;
    } finally {
      metric.stop();
    }
  };

  return { monitorRequest };
};
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración Básica de Firebase
Configura Firebase en tu proyecto React Native:

**Requisitos:**
- Instalación de dependencias
- Configuración de Android e iOS
- Implementación de analytics básico
- Tracking de pantallas

**Implementación:**
```javascript
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { useScreenTracking } from './hooks/useAnalytics';

const App = () => {
  return (
    <NavigationContainer>
      <MainNavigator />
    </NavigationContainer>
  );
};

// hooks/useAnalytics.js
export const useScreenTracking = (screenName) => {
  useEffect(() => {
    analytics().logScreenView({
      screen_name: screenName,
      screen_class: 'Screen'
    });
  }, [screenName]);
};
```

### Ejercicio 2: Error Boundary con Crashlytics
Implementa un error boundary que capture errores:

**Requisitos:**
- Captura de errores de componentes
- Registro en Crashlytics
- UI de fallback para errores
- Botón de reintento

**Implementación:**
```jsx
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    crashlytics().recordError(error);
    crashlytics().setAttribute('error_boundary', 'true');
    crashlytics().setAttribute('component_stack', errorInfo.componentStack);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.errorContainer}>
          <Text>Algo salió mal</Text>
          <Button title="Reintentar" onPress={this.resetError} />
        </View>
      );
    }
    
    return this.props.children;
  }
}
```

---

## 🔍 Puntos Clave

1. **El monitoreo es esencial** para aplicaciones móviles de calidad
2. **Firebase** proporciona herramientas completas de monitoreo
3. **Error boundaries** capturan errores de React de manera efectiva
4. **Performance monitoring** ayuda a optimizar la experiencia del usuario
5. **Analytics** proporciona insights valiosos sobre el comportamiento del usuario

---

## 📖 Recursos Adicionales

- [Firebase Documentation](https://firebase.google.com/docs)
- [React Native Firebase](https://rnfirebase.io/)
- [Sentry React Native](https://docs.sentry.io/platforms/react-native/)
- [Mixpanel React Native](https://developer.mixpanel.com/docs/react-native)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Crashlytics y Error Tracking** y cómo implementar un sistema robusto de captura y análisis de errores en producción.
