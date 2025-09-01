# 📱 Clase 1: Fundamentos de APIs Nativas

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Comprender** qué son las APIs nativas y por qué son importantes
2. **Diferenciar** entre APIs nativas y JavaScript APIs
3. **Configurar** el entorno para desarrollo con APIs nativas
4. **Crear** un Native Module básico
5. **Entender** el bridge de JavaScript a nativo

---

## 📚 Contenido Teórico

### **1. ¿Qué son las APIs Nativas?**

#### **A. Definición y Conceptos Básicos**
Las APIs nativas son interfaces que permiten a tu aplicación React Native acceder a funcionalidades específicas del sistema operativo (iOS/Android) que no están disponibles a través de JavaScript puro.

```javascript
// ❌ JavaScript puro - Limitado
const getDeviceInfo = () => {
  return {
    platform: 'unknown',
    version: 'unknown',
    model: 'unknown'
  };
};

// ✅ APIs Nativas - Acceso completo al sistema
import { NativeModules } from 'react-native';

const { DeviceInfo } = NativeModules;
const deviceInfo = DeviceInfo.getDeviceInfo();
// Retorna información real del dispositivo
```

#### **B. Por Qué Necesitamos APIs Nativas**
```javascript
// Funcionalidades que requieren APIs nativas:
const nativeFeatures = {
  camera: 'Acceso a cámara y galería',
  location: 'GPS y servicios de ubicación',
  notifications: 'Push notifications y notificaciones locales',
  biometrics: 'Touch ID, Face ID, huellas dactilares',
  bluetooth: 'Conexiones Bluetooth y dispositivos cercanos',
  sensors: 'Acelerómetro, giroscopio, brújula',
  fileSystem: 'Acceso al sistema de archivos',
  contacts: 'Lista de contactos del dispositivo',
  calendar: 'Calendario y eventos',
  audio: 'Reproducción y grabación de audio'
};
```

### **2. Arquitectura del Bridge**

#### **A. Cómo Funciona el Bridge**
```javascript
// Flujo de comunicación JavaScript → Nativo
const bridgeFlow = {
  step1: 'JavaScript llama a Native Module',
  step2: 'Bridge serializa los parámetros',
  step3: 'Parámetros se envían al hilo nativo',
  step4: 'Código nativo procesa la solicitud',
  step5: 'Resultado se serializa de vuelta',
  step6: 'JavaScript recibe el resultado'
};

// Ejemplo práctico
import { NativeModules } from 'react-native';

const { CameraModule } = NativeModules;

// Esta llamada cruza el bridge
const takePhoto = async () => {
  try {
    const photo = await CameraModule.takePhoto({
      quality: 'high',
      flash: 'auto'
    });
    return photo;
  } catch (error) {
    console.error('Error taking photo:', error);
  }
};
```

#### **B. Ventajas y Desventajas del Bridge**
```javascript
// ✅ VENTAJAS
const advantages = {
  flexibilidad: 'Acceso completo a funcionalidades nativas',
  performance: 'Código nativo es más rápido para operaciones pesadas',
  integración: 'Acceso directo a APIs del sistema',
  control: 'Control total sobre el comportamiento nativo'
};

// ❌ DESVENTAJAS
const disadvantages = {
  overhead: 'Cada llamada cruza el bridge (lento)',
  complejidad: 'Requiere conocimiento de iOS/Android',
  mantenimiento: 'Código separado para cada plataforma',
  debugging: 'Más difícil de debuggear'
};
```

### **3. Configuración del Entorno**

#### **A. Configuración para iOS**
```bash
# Instalar CocoaPods si no está instalado
sudo gem install cocoapods

# Navegar al directorio iOS
cd ios

# Instalar dependencias
pod install

# Abrir proyecto en Xcode
open YourApp.xcworkspace
```

#### **B. Configuración para Android**
```gradle
// android/app/build.gradle
android {
    compileSdkVersion 33
    buildToolsVersion "33.0.0"
    
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 33
    }
}

dependencies {
    implementation "com.facebook.react:react-native:+"
    // Otras dependencias nativas
}
```

#### **C. Configuración de Metro**
```javascript
// metro.config.js
module.exports = {
  resolver: {
    platforms: ['ios', 'android', 'native', 'web'],
  },
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
    }),
  },
};
```

### **4. Creando tu Primer Native Module**

#### **A. Native Module para iOS (Swift)**
```swift
// ios/DeviceInfoModule.swift
import Foundation
import React

@objc(DeviceInfoModule)
class DeviceInfoModule: NSObject {
  
  @objc
  func getDeviceInfo(_ resolve: @escaping RCTPromiseResolveBlock,
                     rejecter reject: @escaping RCTPromiseRejectBlock) {
    let deviceInfo: [String: Any] = [
      "name": UIDevice.current.name,
      "model": UIDevice.current.model,
      "systemName": UIDevice.current.systemName,
      "systemVersion": UIDevice.current.systemVersion,
      "identifierForVendor": UIDevice.current.identifierForVendor?.uuidString ?? "",
      "batteryLevel": UIDevice.current.batteryLevel,
      "batteryState": UIDevice.current.batteryState.rawValue
    ]
    
    resolve(deviceInfo)
  }
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return false
  }
}
```

#### **B. Bridge para iOS**
```objc
// ios/DeviceInfoModule.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(DeviceInfoModule, NSObject)

RCT_EXTERN_METHOD(getDeviceInfo:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)

@end
```

#### **C. Native Module para Android (Kotlin)**
```kotlin
// android/app/src/main/java/com/yourapp/DeviceInfoModule.kt
package com.yourapp

import com.facebook.react.bridge.*
import com.facebook.react.modules.core.DeviceEventManagerModule
import android.os.Build
import android.content.Context

class DeviceInfoModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
  
  override fun getName(): String {
    return "DeviceInfoModule"
  }
  
  @ReactMethod
  fun getDeviceInfo(promise: Promise) {
    try {
      val deviceInfo = Arguments.createMap().apply {
        putString("name", Build.MODEL)
        putString("model", Build.MODEL)
        putString("systemName", "Android")
        putString("systemVersion", Build.VERSION.RELEASE)
        putString("sdkVersion", Build.VERSION.SDK_INT.toString())
        putString("manufacturer", Build.MANUFACTURER)
        putString("brand", Build.BRAND)
      }
      
      promise.resolve(deviceInfo)
    } catch (e: Exception) {
      promise.reject("ERROR", e.message)
    }
  }
}
```

#### **D. Package para Android**
```kotlin
// android/app/src/main/java/com/yourapp/DeviceInfoPackage.kt
package com.yourapp

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class DeviceInfoPackage : ReactPackage {
  override fun createNativeModules(reactContext: ReactApplicationContext): List<NativeModule> {
    return listOf(DeviceInfoModule(reactContext))
  }
  
  override fun createViewManagers(reactContext: ReactApplicationContext): List<ViewManager<*, *>> {
    return emptyList()
  }
}
```

### **5. Usando el Native Module en React Native**

#### **A. Importación y Uso**
```javascript
// App.js
import { NativeModules } from 'react-native';

const { DeviceInfoModule } = NativeModules;

const App = () => {
  const [deviceInfo, setDeviceInfo] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const getDeviceInfo = async () => {
    setLoading(true);
    try {
      const info = await DeviceInfoModule.getDeviceInfo();
      setDeviceInfo(info);
    } catch (error) {
      console.error('Error getting device info:', error);
    } finally {
      setLoading(false);
    }
  };
  
  useEffect(() => {
    getDeviceInfo();
  }, []);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Información del Dispositivo</Text>
      
      {loading && <ActivityIndicator size="large" />}
      
      {deviceInfo && (
        <View style={styles.infoContainer}>
          <Text>Nombre: {deviceInfo.name}</Text>
          <Text>Modelo: {deviceInfo.model}</Text>
          <Text>Sistema: {deviceInfo.systemName}</Text>
          <Text>Versión: {deviceInfo.systemVersion}</Text>
          <Text>Fabricante: {deviceInfo.manufacturer}</Text>
        </View>
      )}
      
      <Button title="Actualizar Info" onPress={getDeviceInfo} />
    </View>
  );
};
```

#### **B. Manejo de Errores**
```javascript
const SafeNativeModule = {
  async callNativeMethod(methodName, ...args) {
    try {
      // Verificar si el módulo está disponible
      if (!NativeModules[methodName]) {
        throw new Error(`Native module ${methodName} not found`);
      }
      
      // Llamar al método nativo
      const result = await NativeModules[methodName](...args);
      return result;
      
    } catch (error) {
      console.error(`Error calling ${methodName}:`, error);
      
      // Fallback o manejo de error
      if (error.code === 'UNAVAILABLE') {
        // Módulo no disponible, usar alternativa
        return this.getFallbackResult(methodName);
      }
      
      throw error;
    }
  },
  
  getFallbackResult(methodName) {
    // Implementar fallbacks para métodos no disponibles
    const fallbacks = {
      getDeviceInfo: () => ({
        name: 'Unknown Device',
        model: 'Unknown Model',
        systemName: 'Unknown System',
        systemVersion: 'Unknown Version'
      })
    };
    
    return fallbacks[methodName] ? fallbacks[methodName]() : null;
  }
};
```

---

## 💻 Ejercicios Prácticos

### **Ejercicio 1: Native Module Básico**
Crea un Native Module que retorne información básica del dispositivo.

```javascript
// Implementa:
// - Native Module para iOS (Swift/Objective-C)
// - Native Module para Android (Kotlin/Java)
// - Bridge y package
// - Uso en React Native
```

**Tarea**: Crea un módulo que retorne nombre, modelo y versión del sistema.

### **Ejercicio 2: Native Module con Parámetros**
Crea un Native Module que acepte parámetros y retorne resultados procesados.

```javascript
// Implementa:
// - Método que acepte parámetros
// - Validación de parámetros
// - Procesamiento nativo
// - Retorno de resultados
```

**Tarea**: Crea un módulo que calcule el factorial de un número usando código nativo.

### **Ejercicio 3: Native Module con Callbacks**
Implementa un Native Module que use callbacks para comunicación asíncrona.

```javascript
// Implementa:
// - Callbacks para éxito y error
// - Operaciones asíncronas
// - Manejo de errores
// - Limpieza de recursos
```

**Tarea**: Crea un módulo que simule una operación larga y notifique el progreso.

---

## 🎯 Proyecto Integrador

### **App de Información del Sistema**

Crea una aplicación que use múltiples Native Modules:

#### **Funcionalidades Requeridas:**
1. **Información del Dispositivo**: Modelo, sistema, versión
2. **Información de Hardware**: CPU, memoria, batería
3. **Información de Red**: Tipo de conexión, velocidad
4. **Información de Sensores**: Acelerómetro, giroscopio
5. **Información de Apps**: Apps instaladas, uso de espacio

#### **Requisitos Técnicos:**
- **Múltiples Native Modules**: Para diferentes funcionalidades
- **Manejo de Errores**: Con fallbacks y mensajes claros
- **UI Responsiva**: Que muestre toda la información
- **Actualización en Tiempo Real**: Para datos que cambian
- **Documentación**: De todos los módulos nativos

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [Native Modules](https://reactnative.dev/docs/native-modules-ios)
- [Android Native Modules](https://reactnative.dev/docs/native-modules-android)
- [Bridge Documentation](https://reactnative.dev/docs/communication-ios)

### **Artículos Recomendados:**
- "Building Native Modules in React Native"
- "iOS vs Android Native Development"
- "Performance Optimization with Native Modules"

---

## 🎓 Próximos Pasos

### **En la Siguiente Clase Aprenderás:**
- **Cámara y Galería**: Acceso a cámara, fotos y videos
- **Permisos**: Manejo de permisos del sistema
- **Integración**: Con librerías nativas existentes

---

**🎯 Objetivo de la Clase**: Comprender los fundamentos de las APIs nativas y crear tu primer Native Module funcional.

**💡 Consejo**: Las APIs nativas te dan poder ilimitado, pero úsalas sabiamente. No todo necesita ser nativo, solo las funcionalidades que realmente requieren acceso al sistema.
