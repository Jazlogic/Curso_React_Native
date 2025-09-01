# 📱 Clase 2: Creación de Módulos Nativos

## 🎯 Objetivos de la Clase
- Crear módulos nativos básicos para iOS y Android
- Implementar métodos nativos con parámetros y valores de retorno
- Manejar tipos de datos entre JavaScript y nativo
- Configurar el proyecto para incluir módulos nativos

---

## 📚 Contenido Teórico

### 1. Estructura de un Módulo Nativo

Un módulo nativo en React Native tiene dos partes principales:

#### iOS (Swift/Objective-C)
```swift
// MyNativeModule.swift
import Foundation
import React

@objc(MyNativeModule)
class MyNativeModule: NSObject {
  
  @objc
  func constantsToExport() -> [AnyHashable : Any]! {
    return ["platform": "iOS"]
  }
  
  @objc
  func addNumbers(_ a: NSNumber, b: NSNumber, resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    let result = a.intValue + b.intValue
    resolve(result)
  }
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return false
  }
}
```

#### Android (Kotlin/Java)
```kotlin
// MyNativeModule.kt
package com.miapp

import com.facebook.react.bridge.*

class MyNativeModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
  
  override fun getName(): String {
    return "MyNativeModule"
  }
  
  override fun getConstants(): Map<String, Any>? {
    return mapOf("platform" to "Android")
  }
  
  @ReactMethod
  fun addNumbers(a: Int, b: Int, promise: Promise) {
    try {
      val result = a + b
      promise.resolve(result)
    } catch (e: Exception) {
      promise.reject("ERROR", e.message)
    }
  }
}
```

### 2. Configuración del Proyecto

#### iOS - Podfile
```ruby
# ios/Podfile
target 'MyApp' do
  # ... otras configuraciones
  
  pod 'React', :path => '../node_modules/react-native/'
  pod 'React-Core', :path => '../node_modules/react-native/'
  
  # Incluir módulos nativos personalizados
  pod 'MyNativeModule', :path => '../node_modules/my-native-module'
end
```

#### Android - build.gradle
```gradle
// android/app/build.gradle
dependencies {
    // ... otras dependencias
    
    // Incluir módulos nativos
    implementation project(':my-native-module')
}
```

### 3. Bridge de JavaScript

```typescript
// MyNativeModule.ts
import { NativeModules, Platform } from 'react-native';

const { MyNativeModule } = NativeModules;

interface MyNativeModuleInterface {
  addNumbers(a: number, b: number): Promise<number>;
  getPlatform(): string;
}

// Verificar si el módulo está disponible
if (!MyNativeModule) {
  throw new Error('MyNativeModule no está disponible en esta plataforma');
}

export default MyNativeModule as MyNativeModuleInterface;
```

### 4. Uso en React Native

```tsx
// App.tsx
import React, { useState, useEffect } from 'react';
import { View, Text, Button, Alert } from 'react-native';
import MyNativeModule from './MyNativeModule';

const App = () => {
  const [platform, setPlatform] = useState<string>('');
  const [result, setResult] = useState<number>(0);

  useEffect(() => {
    // Obtener la plataforma
    setPlatform(MyNativeModule.getPlatform());
  }, []);

  const handleAddNumbers = async () => {
    try {
      const sum = await MyNativeModule.addNumbers(5, 3);
      setResult(sum);
      Alert.alert('Resultado', `5 + 3 = ${sum}`);
    } catch (error) {
      Alert.alert('Error', 'No se pudo realizar la operación');
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ fontSize: 24, marginBottom: 20 }}>
        Plataforma: {platform}
      </Text>
      
      <Button title="Sumar 5 + 3" onPress={handleAddNumbers} />
      
      <Text style={{ fontSize: 18, marginTop: 20 }}>
        Resultado: {result}
      </Text>
    </View>
  );
};

export default App;
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Módulo de Calculadora Nativa
Crea un módulo nativo que implemente operaciones matemáticas básicas:

**Requisitos:**
- Suma, resta, multiplicación y división
- Manejo de errores para división por cero
- Retorno de resultados como Promise
- Constantes para operaciones disponibles

**Implementación iOS:**
```swift
@objc
func calculate(_ operation: String, a: NSNumber, b: NSNumber, resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
  let aValue = a.doubleValue
  let bValue = b.doubleValue
  
  switch operation {
  case "add":
    resolve(aValue + bValue)
  case "subtract":
    resolve(aValue - bValue)
  case "multiply":
    resolve(aValue * bValue)
  case "divide":
    if bValue == 0 {
      reject("DIVISION_BY_ZERO", "No se puede dividir por cero", nil)
    } else {
      resolve(aValue / bValue)
    }
  default:
    reject("INVALID_OPERATION", "Operación no válida", nil)
  }
}
```

**Implementación Android:**
```kotlin
@ReactMethod
fun calculate(operation: String, a: Double, b: Double, promise: Promise) {
  try {
    val result = when (operation) {
      "add" -> a + b
      "subtract" -> a - b
      "multiply" -> a * b
      "divide" -> {
        if (b == 0.0) {
          throw Exception("No se puede dividir por cero")
        }
        a / b
      }
      else -> throw Exception("Operación no válida")
    }
    promise.resolve(result)
  } catch (e: Exception) {
    promise.reject("CALCULATION_ERROR", e.message)
  }
}
```

### Ejercicio 2: Módulo de Sensores del Dispositivo
Crea un módulo que acceda a sensores nativos:

**Requisitos:**
- Acceso al acelerómetro
- Información de la batería
- Estado de la conexión de red
- Métodos para suscribirse a cambios

**Implementación básica:**
```typescript
interface DeviceSensors {
  getBatteryLevel(): Promise<number>;
  getNetworkType(): Promise<string>;
  startAccelerometer(callback: (x: number, y: number, z: number) => void): void;
  stopAccelerometer(): void;
}
```

---

## 🔍 Puntos Clave

1. **Tipos de Datos**: React Native convierte automáticamente entre tipos JavaScript y nativos
2. **Promesas**: Usa Promises para operaciones asíncronas
3. **Callbacks**: Implementa callbacks para eventos en tiempo real
4. **Manejo de Errores**: Siempre incluye manejo de errores robusto
5. **Configuración**: Cada plataforma requiere configuración específica

---

## 📖 Recursos Adicionales

- [React Native Native Modules Guide](https://reactnative.dev/docs/native-modules-intro)
- [iOS Native Modules](https://reactnative.dev/docs/native-modules-ios)
- [Android Native Modules](https://reactnative.dev/docs/native-modules-android)
- [TurboModules](https://reactnative.dev/docs/the-new-architecture/pillars-turbomodules)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Manejo de Eventos y Callbacks** en módulos nativos, incluyendo suscripciones a eventos del sistema y comunicación bidireccional.
