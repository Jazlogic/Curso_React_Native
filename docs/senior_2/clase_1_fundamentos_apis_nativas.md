# Clase 1: Fundamentos de APIs Nativas 📱

## Objetivos de la Clase
- Comprender qué son las APIs nativas y su importancia en React Native
- Aprender a manejar permisos de dispositivo de forma segura
- Configurar APIs nativas para diferentes plataformas (iOS/Android)
- Implementar manejo robusto de errores nativos
- Crear un sistema de permisos reutilizable

## Duración Estimada
**1.5 horas**

## Contenido Teórico

### 1. ¿Qué son las APIs Nativas?

Las **APIs Nativas** son interfaces que permiten a tu aplicación React Native acceder a funcionalidades específicas del dispositivo que no están disponibles en JavaScript puro.

```javascript
// Ejemplo básico de API nativa
import { Platform, Alert } from 'react-native';

// Platform.OS nos dice en qué plataforma estamos ejecutando
if (Platform.OS === 'ios') {
  // Código específico para iOS
  console.log('Ejecutando en iOS');
} else if (Platform.OS === 'android') {
  // Código específico para Android
  console.log('Ejecutando en Android');
}
```

**¿Por qué son importantes?**
- **Acceso al hardware**: Cámara, GPS, sensores, vibración
- **Funcionalidades del sistema**: Notificaciones, archivos, contactos
- **Experiencia nativa**: Comportamiento similar a apps nativas
- **Performance**: Ejecución en código nativo, no en JavaScript

### 2. Sistema de Permisos

Los permisos son **autorizaciones** que tu app debe solicitar al usuario para acceder a funcionalidades del dispositivo.

```javascript
import { PermissionsAndroid, Platform } from 'react-native';

class PermissionManager {
  // Solicitar permiso de cámara en Android
  static async requestCameraPermission() {
    if (Platform.OS === 'android') {
      try {
        // PermissionsAndroid.PERMISSIONS.CAMERA es la constante para el permiso de cámara
        const granted = await PermissionsAndroid.request(
          PermissionsAndroid.PERMISSIONS.CAMERA,
          {
            title: 'Permiso de Cámara',
            message: 'La aplicación necesita acceso a la cámara para tomar fotos',
            buttonNeutral: 'Preguntar más tarde',
            buttonNegative: 'Cancelar',
            buttonPositive: 'OK',
          }
        );
        
        // PermissionsAndroid.RESULTS.GRANTED significa que el usuario concedió el permiso
        return granted === PermissionsAndroid.RESULTS.GRANTED;
      } catch (err) {
        console.warn('Error al solicitar permiso:', err);
        return false;
      }
    }
    
    // En iOS, los permisos se manejan de forma diferente
    return true;
  }

  // Verificar si ya tenemos un permiso
  static async checkCameraPermission() {
    if (Platform.OS === 'android') {
      // PermissionsAndroid.check() verifica si ya tenemos el permiso
      return await PermissionsAndroid.check(PermissionsAndroid.PERMISSIONS.CAMERA);
    }
    return true;
  }
}
```

**Tipos de Permisos Comunes:**
- **CAMERA**: Acceso a cámara y galería
- **ACCESS_FINE_LOCATION**: Ubicación precisa (GPS)
- **RECORD_AUDIO**: Grabación de audio
- **WRITE_EXTERNAL_STORAGE**: Escritura en almacenamiento
- **READ_CONTACTS**: Lectura de contactos

### 3. Configuración de Plataforma

React Native permite configurar comportamientos específicos para cada plataforma:

```javascript
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    // Estilos base que se aplican a ambas plataformas
    flex: 1,
    backgroundColor: '#fff',
    
    // Estilos específicos para iOS
    ...Platform.select({
      ios: {
        paddingTop: 44, // Barra de estado de iOS
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84,
      },
      // Estilos específicos para Android
      android: {
        paddingTop: 24, // Barra de estado de Android
        elevation: 5, // Sombra en Android
      },
    }),
  },
  
  button: {
    // Comportamiento específico por plataforma
    ...Platform.select({
      ios: {
        // En iOS, los botones tienen un estilo más suave
        backgroundColor: '#007AFF',
        borderRadius: 8,
      },
      android: {
        // En Android, los botones tienen un estilo más material
        backgroundColor: '#6200EE',
        borderRadius: 4,
        elevation: 2,
      },
    }),
  },
});
```

**Platform.select()** es una función que permite seleccionar valores específicos por plataforma:

```javascript
const config = Platform.select({
  ios: {
    // Configuración específica para iOS
    apiEndpoint: 'https://api.ios.com',
    timeout: 30000,
  },
  android: {
    // Configuración específica para Android
    apiEndpoint: 'https://api.android.com',
    timeout: 25000,
  },
  default: {
    // Configuración por defecto (web, etc.)
    apiEndpoint: 'https://api.default.com',
    timeout: 20000,
  },
});
```

### 4. Manejo de Errores Nativos

El manejo de errores en APIs nativas es **crítico** para la estabilidad de tu aplicación:

```javascript
class NativeAPIError extends Error {
  constructor(message, code, platform) {
    super(message);
    this.name = 'NativeAPIError';
    this.code = code;
    this.platform = platform;
    this.timestamp = new Date();
  }
}

class NativeAPIManager {
  static async executeWithErrorHandling(apiCall, fallback = null) {
    try {
      // Intentar ejecutar la API nativa
      const result = await apiCall();
      return result;
    } catch (error) {
      // Clasificar el tipo de error
      const errorType = this.classifyError(error);
      
      // Registrar el error para debugging
      console.error('Error en API nativa:', {
        message: error.message,
        code: error.code,
        type: errorType,
        platform: Platform.OS,
        timestamp: new Date().toISOString(),
      });
      
      // Manejar el error según su tipo
      switch (errorType) {
        case 'permission_denied':
          // El usuario denegó el permiso
          Alert.alert(
            'Permiso Requerido',
            'Esta funcionalidad requiere permisos del dispositivo',
            [
              { text: 'Cancelar', style: 'cancel' },
              { text: 'Configuración', onPress: () => this.openSettings() },
            ]
          );
          break;
          
        case 'device_not_supported':
          // El dispositivo no soporta esta funcionalidad
          Alert.alert(
            'Funcionalidad No Soportada',
            'Tu dispositivo no soporta esta característica',
            [{ text: 'OK' }]
          );
          break;
          
        case 'network_error':
          // Error de red
          Alert.alert(
            'Error de Conexión',
            'Verifica tu conexión a internet e intenta nuevamente',
            [{ text: 'Reintentar', onPress: () => this.retry(apiCall) }]
          );
          break;
          
        default:
          // Error desconocido
          Alert.alert(
            'Error Inesperado',
            'Ocurrió un error inesperado. Intenta nuevamente.',
            [{ text: 'OK' }]
          );
      }
      
      // Retornar fallback si está disponible
      return fallback;
    }
  }

  // Clasificar el tipo de error basado en el código o mensaje
  static classifyError(error) {
    if (error.code === 'PERMISSION_DENIED') return 'permission_denied';
    if (error.code === 'DEVICE_NOT_SUPPORTED') return 'device_not_supported';
    if (error.message.includes('Network')) return 'network_error';
    if (error.message.includes('Permission')) return 'permission_denied';
    
    return 'unknown_error';
  }

  // Abrir configuración del dispositivo
  static openSettings() {
    if (Platform.OS === 'ios') {
      // En iOS, abrimos la configuración de la app
      Linking.openURL('app-settings:');
    } else {
      // En Android, abrimos la configuración general
      Linking.openSettings();
    }
  }

  // Reintentar una operación
  static async retry(apiCall, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await apiCall();
      } catch (error) {
        if (i === maxRetries - 1) throw error;
        // Esperar antes de reintentar (backoff exponencial)
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
      }
    }
  }
}
```

### 5. Configuración de Permisos en Android

En Android, necesitas declarar permisos en el archivo `AndroidManifest.xml`:

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <!-- Permisos básicos -->
  <uses-permission android:name="android.permission.CAMERA" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.RECORD_AUDIO" />
  
  <!-- Permisos de almacenamiento -->
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  
  <!-- Permisos de red -->
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  
  <!-- Características del dispositivo -->
  <uses-feature android:name="android.hardware.camera" android:required="false" />
  <uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />
  <uses-feature android:name="android.hardware.location" android:required="false" />
  
  <application>
    <!-- ... resto de la configuración ... -->
  </application>
</manifest>
```

### 6. Configuración de Permisos en iOS

En iOS, los permisos se configuran en el archivo `Info.plist`:

```xml
<!-- ios/YourApp/Info.plist -->
<dict>
  <!-- Permiso de cámara -->
  <key>NSCameraUsageDescription</key>
  <string>Esta aplicación necesita acceso a la cámara para tomar fotos</string>
  
  <!-- Permiso de galería -->
  <key>NSPhotoLibraryUsageDescription</key>
  <string>Esta aplicación necesita acceso a tu galería para seleccionar fotos</string>
  
  <!-- Permiso de ubicación -->
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>Esta aplicación necesita acceso a tu ubicación para mostrar servicios cercanos</string>
  
  <!-- Permiso de micrófono -->
  <key>NSMicrophoneUsageDescription</key>
  <string>Esta aplicación necesita acceso al micrófono para grabar audio</string>
  
  <!-- Permiso de contactos -->
  <key>NSContactsUsageDescription</key>
  <string>Esta aplicación necesita acceso a tus contactos para sincronizar información</string>
</dict>
```

## Ejercicios Prácticos

### Ejercicio 1: Gestor de Permisos Universal
Crea un componente que maneje todos los permisos de tu aplicación:

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Alert,
  Platform,
} from 'react-native';
import { PermissionManager } from './PermissionManager';

const PermissionHandler = () => {
  const [permissions, setPermissions] = useState({
    camera: false,
    location: false,
    microphone: false,
    contacts: false,
  });

  // Verificar permisos al cargar el componente
  useEffect(() => {
    checkAllPermissions();
  }, []);

  // Verificar todos los permisos
  const checkAllPermissions = async () => {
    const cameraPerm = await PermissionManager.checkCameraPermission();
    const locationPerm = await PermissionManager.checkLocationPermission();
    const micPerm = await PermissionManager.checkMicrophonePermission();
    const contactsPerm = await PermissionManager.checkContactsPermission();

    setPermissions({
      camera: cameraPerm,
      location: locationPerm,
      microphone: micPerm,
      contacts: contactsPerm,
    });
  };

  // Solicitar un permiso específico
  const requestPermission = async (permissionType) => {
    let granted = false;
    
    switch (permissionType) {
      case 'camera':
        granted = await PermissionManager.requestCameraPermission();
        break;
      case 'location':
        granted = await PermissionManager.requestLocationPermission();
        break;
      case 'microphone':
        granted = await PermissionManager.requestMicrophonePermission();
        break;
      case 'contacts':
        granted = await PermissionManager.requestContactsPermission();
        break;
    }

    if (granted) {
      // Actualizar el estado
      setPermissions(prev => ({
        ...prev,
        [permissionType]: true,
      }));
      
      Alert.alert('Éxito', `Permiso de ${permissionType} concedido`);
    } else {
      Alert.alert('Permiso Denegado', `No se pudo obtener el permiso de ${permissionType}`);
    }
  };

  // Renderizar botón de permiso
  const renderPermissionButton = (type, label) => (
    <TouchableOpacity
      style={[
        styles.permissionButton,
        permissions[type] ? styles.permissionGranted : styles.permissionDenied,
      ]}
      onPress={() => requestPermission(type)}
      disabled={permissions[type]}
    >
      <Text style={[
        styles.buttonText,
        permissions[type] ? styles.buttonTextGranted : styles.buttonTextDenied,
      ]}>
        {permissions[type] ? `✅ ${label}` : `❌ ${label}`}
      </Text>
    </TouchableOpacity>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Gestor de Permisos</Text>
      <Text style={styles.subtitle}>Plataforma: {Platform.OS}</Text>
      
      <View style={styles.permissionsContainer}>
        {renderPermissionButton('camera', 'Cámara')}
        {renderPermissionButton('location', 'Ubicación')}
        {renderPermissionButton('microphone', 'Micrófono')}
        {renderPermissionButton('contacts', 'Contactos')}
      </View>
      
      <TouchableOpacity
        style={styles.refreshButton}
        onPress={checkAllPermissions}
      >
        <Text style={styles.refreshButtonText}>🔄 Actualizar Estado</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 10,
    color: '#333',
  },
  subtitle: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 30,
    color: '#666',
  },
  permissionsContainer: {
    gap: 15,
    marginBottom: 30,
  },
  permissionButton: {
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  permissionGranted: {
    backgroundColor: '#4CAF50',
  },
  permissionDenied: {
    backgroundColor: '#f44336',
  },
  buttonText: {
    fontSize: 16,
    fontWeight: '600',
  },
  buttonTextGranted: {
    color: 'white',
  },
  buttonTextDenied: {
    color: 'white',
  },
  refreshButton: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  refreshButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
});

export default PermissionHandler;
```

### Ejercicio 2: Configurador de Plataforma
Crea un hook personalizado que maneje configuraciones específicas por plataforma:

```javascript
import { useState, useEffect } from 'react';
import { Platform, Dimensions } from 'react-native';

export const usePlatformConfig = () => {
  const [config, setConfig] = useState({});

  useEffect(() => {
    const platformConfig = Platform.select({
      ios: {
        // Configuración específica para iOS
        statusBarHeight: 44,
        tabBarHeight: 83,
        safeAreaInsets: { top: 44, bottom: 34, left: 0, right: 0 },
        colors: {
          primary: '#007AFF',
          secondary: '#5856D6',
          background: '#F2F2F7',
          text: '#000000',
        },
        fonts: {
          regular: 'SF Pro Text',
          medium: 'SF Pro Text-Medium',
          bold: 'SF Pro Text-Bold',
        },
        animations: {
          duration: 300,
          easing: 'easeInOut',
        },
      },
      android: {
        // Configuración específica para Android
        statusBarHeight: 24,
        tabBarHeight: 56,
        safeAreaInsets: { top: 24, bottom: 0, left: 0, right: 0 },
        colors: {
          primary: '#6200EE',
          secondary: '#03DAC6',
          background: '#FFFFFF',
          text: '#000000',
        },
        fonts: {
          regular: 'Roboto',
          medium: 'Roboto-Medium',
          bold: 'Roboto-Bold',
        },
        animations: {
          duration: 250,
          easing: 'easeOut',
        },
      },
      default: {
        // Configuración por defecto
        statusBarHeight: 0,
        tabBarHeight: 60,
        safeAreaInsets: { top: 0, bottom: 0, left: 0, right: 0 },
        colors: {
          primary: '#000000',
          secondary: '#666666',
          background: '#FFFFFF',
          text: '#000000',
        },
        fonts: {
          regular: 'System',
          medium: 'System',
          bold: 'System',
        },
        animations: {
          duration: 300,
          easing: 'ease',
        },
      },
    });

    // Agregar información adicional de la pantalla
    const screenInfo = {
      width: Dimensions.get('window').width,
      height: Dimensions.get('window').height,
      scale: Dimensions.get('window').scale,
      fontScale: Dimensions.get('window').fontScale,
    };

    setConfig({
      ...platformConfig,
      screen: screenInfo,
      isIOS: Platform.OS === 'ios',
      isAndroid: Platform.OS === 'android',
      version: Platform.Version,
    });
  }, []);

  return config;
};
```

### Ejercicio 3: Manejador de Errores Robusto
Implementa un sistema completo de manejo de errores para APIs nativas:

```javascript
import { Alert, Platform, Linking } from 'react-native';

class ErrorHandler {
  // Tipos de errores comunes en APIs nativas
  static ERROR_TYPES = {
    PERMISSION_DENIED: 'permission_denied',
    DEVICE_NOT_SUPPORTED: 'device_not_supported',
    NETWORK_ERROR: 'network_error',
    TIMEOUT_ERROR: 'timeout_error',
    UNKNOWN_ERROR: 'unknown_error',
  };

  // Manejar error con recuperación automática
  static async handleError(error, context = '') {
    // Clasificar el error
    const errorType = this.classifyError(error);
    
    // Registrar el error
    this.logError(error, errorType, context);
    
    // Intentar recuperación automática
    const recovered = await this.attemptRecovery(errorType, context);
    
    if (!recovered) {
      // Mostrar mensaje al usuario
      this.showUserFriendlyError(errorType, context);
    }
    
    return recovered;
  }

  // Clasificar el tipo de error
  static classifyError(error) {
    if (error.code === 'PERMISSION_DENIED' || error.message.includes('Permission')) {
      return this.ERROR_TYPES.PERMISSION_DENIED;
    }
    
    if (error.code === 'DEVICE_NOT_SUPPORTED' || error.message.includes('not supported')) {
      return this.ERROR_TYPES.DEVICE_NOT_SUPPORTED;
    }
    
    if (error.code === 'NETWORK_ERROR' || error.message.includes('Network')) {
      return this.ERROR_TYPES.NETWORK_ERROR;
    }
    
    if (error.code === 'TIMEOUT' || error.message.includes('timeout')) {
      return this.ERROR_TYPES.TIMEOUT_ERROR;
    }
    
    return this.ERROR_TYPES.UNKNOWN_ERROR;
  }

  // Intentar recuperación automática
  static async attemptRecovery(errorType, context) {
    switch (errorType) {
      case this.ERROR_TYPES.PERMISSION_DENIED:
        // Para permisos, no podemos recuperar automáticamente
        return false;
        
      case this.ERROR_TYPES.NETWORK_ERROR:
        // Para errores de red, esperar y reintentar
        await this.delay(2000);
        return true; // Permitir reintento
        
      case this.ERROR_TYPES.TIMEOUT_ERROR:
        // Para timeouts, aumentar el timeout y reintentar
        await this.delay(1000);
        return true; // Permitir reintento
        
      case this.ERROR_TYPES.DEVICE_NOT_SUPPORTED:
        // Para dispositivos no soportados, no hay recuperación
        return false;
        
      default:
        return false;
    }
  }

  // Mostrar error amigable al usuario
  static showUserFriendlyError(errorType, context) {
    const messages = {
      [this.ERROR_TYPES.PERMISSION_DENIED]: {
        title: 'Permiso Requerido',
        message: 'Esta funcionalidad necesita permisos del dispositivo',
        actions: [
          { text: 'Cancelar', style: 'cancel' },
          { text: 'Configuración', onPress: () => this.openSettings() },
        ],
      },
      [this.ERROR_TYPES.DEVICE_NOT_SUPPORTED]: {
        title: 'Funcionalidad No Soportada',
        message: 'Tu dispositivo no soporta esta característica',
        actions: [{ text: 'OK' }],
      },
      [this.ERROR_TYPES.NETWORK_ERROR]: {
        title: 'Error de Conexión',
        message: 'Verifica tu conexión a internet e intenta nuevamente',
        actions: [{ text: 'Reintentar', onPress: () => this.retry(context) }],
      },
      [this.ERROR_TYPES.TIMEOUT_ERROR]: {
        title: 'Tiempo de Espera Agotado',
        message: 'La operación tardó demasiado. Intenta nuevamente.',
        actions: [{ text: 'Reintentar', onPress: () => this.retry(context) }],
      },
      [this.ERROR_TYPES.UNKNOWN_ERROR]: {
        title: 'Error Inesperado',
        message: 'Ocurrió un error inesperado. Intenta nuevamente.',
        actions: [{ text: 'OK' }],
      },
    };

    const errorConfig = messages[errorType] || messages[this.ERROR_TYPES.UNKNOWN_ERROR];
    
    Alert.alert(
      errorConfig.title,
      errorConfig.message,
      errorConfig.actions
    );
  }

  // Abrir configuración del dispositivo
  static openSettings() {
    if (Platform.OS === 'ios') {
      Linking.openURL('app-settings:');
    } else {
      Linking.openSettings();
    }
  }

  // Reintentar operación
  static retry(context) {
    // Emitir evento para reintentar
    console.log(`Reintentando operación en contexto: ${context}`);
  }

  // Delay helper
  static delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // Logging de errores
  static logError(error, errorType, context) {
    const errorLog = {
      timestamp: new Date().toISOString(),
      platform: Platform.OS,
      version: Platform.Version,
      errorType,
      context,
      message: error.message,
      code: error.code,
      stack: error.stack,
    };
    
    console.error('Error en API nativa:', errorLog);
    
    // Aquí podrías enviar el error a un servicio de logging
    // this.sendToLoggingService(errorLog);
  }
}

export default ErrorHandler;
```

## Resumen de la Clase

En esta clase hemos aprendido:

1. **APIs Nativas**: Interfaces para acceder a funcionalidades del dispositivo
2. **Sistema de Permisos**: Manejo seguro de autorizaciones del usuario
3. **Configuración de Plataforma**: Comportamientos específicos para iOS y Android
4. **Manejo de Errores**: Sistema robusto para errores nativos
5. **Configuración de Archivos**: AndroidManifest.xml e Info.plist

## Navegación
- **Anterior**: [Módulo 10: Performance](../senior_2/README.md)
- **Siguiente**: [Clase 2: Cámara y Galería](clase_2_camara_galeria.md)
- **Inicio**: [Índice del Módulo](../senior_2/README.md)

## Próxima Clase
En la siguiente clase aprenderemos a implementar **Cámara y Galería**, incluyendo captura de fotos, grabación de video, y acceso a la galería del dispositivo.
