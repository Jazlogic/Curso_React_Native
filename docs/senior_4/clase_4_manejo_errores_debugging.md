# 📱 Clase 4: Manejo de Errores y Debugging en Módulos Nativos

## 🎯 Objetivos de la Clase
- Implementar manejo robusto de errores en módulos nativos
- Configurar logging y debugging para desarrollo
- Integrar crash reporting y monitoreo de errores
- Usar herramientas de desarrollo para debugging nativo

---

## 📚 Contenido Teórico

### 1. Manejo de Errores en Módulos Nativos

#### iOS - Manejo de Errores
```swift
// MyNativeModule.swift
import Foundation
import React

@objc(MyNativeModule)
class MyNativeModule: NSObject {
  
  // Enum para tipos de error
  enum NativeModuleError: Error, LocalizedError {
    case invalidInput(String)
    case networkError(String)
    case permissionDenied(String)
    case unknownError(String)
    
    var errorDescription: String? {
      switch self {
      case .invalidInput(let message):
        return "Input inválido: \(message)"
      case .networkError(let message):
        return "Error de red: \(message)"
      case .permissionDenied(let message):
        return "Permiso denegado: \(message)"
      case .unknownError(let message):
        return "Error desconocido: \(message)"
      }
    }
    
    var failureReason: String? {
      switch self {
      case .invalidInput:
        return "Los datos de entrada no son válidos"
      case .networkError:
        return "Problema de conectividad de red"
      case .permissionDenied:
        return "La aplicación no tiene permisos necesarios"
      case .unknownError:
        return "Error no identificado en el sistema"
      }
    }
  }
  
  // Método con manejo de errores
  @objc
  func processData(_ data: String, resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    
    // Validar entrada
    guard !data.isEmpty else {
      let error = NativeModuleError.invalidInput("Los datos no pueden estar vacíos")
      reject("INVALID_INPUT", error.localizedDescription, error)
      return
    }
    
    // Simular procesamiento
    do {
      let result = try performDataProcessing(data)
      resolve(result)
    } catch {
      // Log del error para debugging
      NSLog("Error procesando datos: \(error.localizedDescription)")
      
      if let nativeError = error as? NativeModuleError {
        reject(nativeError.localizedDescription, nativeError.failureReason, nativeError)
      } else {
        reject("PROCESSING_ERROR", "Error durante el procesamiento", error)
      }
    }
  }
  
  private func performDataProcessing(_ data: String) throws -> String {
    // Simular operación que puede fallar
    guard data.count > 3 else {
      throw NativeModuleError.invalidInput("Los datos deben tener al menos 4 caracteres")
    }
    
    // Simular error de red ocasional
    if Int.random(in: 1...10) == 1 {
      throw NativeModuleError.networkError("Conexión de red inestable")
    }
    
    return "Procesado: \(data.uppercased())"
  }
}
```

#### Android - Manejo de Errores
```kotlin
// MyNativeModule.kt
package com.miapp

import com.facebook.react.bridge.*
import com.facebook.react.modules.core.DeviceEventManagerModule

class MyNativeModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
  
  // Clases de error personalizadas
  sealed class NativeModuleException(message: String) : Exception(message) {
    object InvalidInputException : NativeModuleException("Input inválido")
    object NetworkException : NativeModuleException("Error de red")
    object PermissionException : NativeModuleException("Permiso denegado")
    object UnknownException : NativeModuleException("Error desconocido")
  }
  
  override fun getName(): String = "MyNativeModule"
  
  @ReactMethod
  fun processData(data: String, promise: Promise) {
    try {
      // Validar entrada
      if (data.isEmpty()) {
        throw NativeModuleException.InvalidInputException
      }
      
      // Procesar datos
      val result = performDataProcessing(data)
      promise.resolve(result)
      
    } catch (e: NativeModuleException) {
      // Log del error para debugging
      Log.e("MyNativeModule", "Error procesando datos: ${e.message}")
      
      when (e) {
        is NativeModuleException.InvalidInputException -> {
          promise.reject("INVALID_INPUT", "Los datos de entrada no son válidos", e)
        }
        is NativeModuleException.NetworkException -> {
          promise.reject("NETWORK_ERROR", "Error de conectividad de red", e)
        }
        is NativeModuleException.PermissionException -> {
          promise.reject("PERMISSION_DENIED", "La aplicación no tiene permisos necesarios", e)
        }
        is NativeModuleException.UnknownException -> {
          promise.reject("UNKNOWN_ERROR", "Error no identificado en el sistema", e)
        }
      }
    } catch (e: Exception) {
      // Error genérico
      Log.e("MyNativeModule", "Error inesperado: ${e.message}")
      promise.reject("UNEXPECTED_ERROR", "Error inesperado durante el procesamiento", e)
    }
  }
  
  private fun performDataProcessing(data: String): String {
    // Simular validación
    if (data.length < 4) {
      throw NativeModuleException.InvalidInputException
    }
    
    // Simular error de red ocasional
    if ((1..10).random() == 1) {
      throw NativeModuleException.NetworkException
    }
    
    return "Procesado: ${data.uppercase()}"
  }
}
```

### 2. Logging y Debugging

#### iOS - Sistema de Logging
```swift
// Logger.swift
import Foundation
import os.log

class NativeLogger {
  static let shared = NativeLogger()
  
  private let log: OSLog
  private let subsystem = Bundle.main.bundleIdentifier ?? "com.miapp"
  
  private init() {
    log = OSLog(subsystem: subsystem, category: "NativeModule")
  }
  
  func debug(_ message: String, file: String = #file, function: String = #function, line: Int = #line) {
    let fileName = (file as NSString).lastPathComponent
    let logMessage = "[\(fileName):\(line)] \(function): \(message)"
    os_log(.debug, log: log, "%{public}@", logMessage)
  }
  
  func info(_ message: String, file: String = #file, function: String = #function, line: Int = #line) {
    let fileName = (file as NSString).lastPathComponent
    let logMessage = "[\(fileName):\(line)] \(function): \(message)"
    os_log(.info, log: log, "%{public}@", logMessage)
  }
  
  func error(_ message: String, error: Error? = nil, file: String = #file, function: String = #function, line: Int = #line) {
    let fileName = (file as NSString).lastPathComponent
    var logMessage = "[\(fileName):\(line)] \(function): \(message)"
    
    if let error = error {
      logMessage += " - Error: \(error.localizedDescription)"
    }
    
    os_log(.error, log: log, "%{public}@", logMessage)
  }
  
  func critical(_ message: String, error: Error? = nil, file: String = #file, function: String = #function, line: Int = #line) {
    let fileName = (file as NSString).lastPathComponent
    var logMessage = "[\(fileName):\(line)] \(function): \(message)"
    
    if let error = error {
      logMessage += " - Error: \(error.localizedDescription)"
    }
    
    os_log(.fault, log: log, "%{public}@", logMessage)
  }
}
```

#### Android - Sistema de Logging
```kotlin
// Logger.kt
package com.miapp

import android.util.Log
import java.text.SimpleDateFormat
import java.util.*

object NativeLogger {
  private const val TAG = "NativeModule"
  private const val MAX_TAG_LENGTH = 23
  
  fun debug(message: String, tag: String = TAG, file: String = "", function: String = "", line: Int = 0) {
    val timestamp = SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault()).format(Date())
    val fileName = file.substringAfterLast("/", file)
    val logMessage = "[$timestamp][$fileName:$line] $function: $message"
    
    Log.d(tag, logMessage)
  }
  
  fun info(message: String, tag: String = TAG, file: String = "", function: String = "", line: Int = 0) {
    val timestamp = SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault()).format(Date())
    val fileName = file.substringAfterLast("/", file)
    val logMessage = "[$timestamp][$fileName:$line] $function: $message"
    
    Log.i(tag, logMessage)
  }
  
  fun error(message: String, error: Throwable? = null, tag: String = TAG, file: String = "", function: String = "", line: Int = 0) {
    val timestamp = SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault()).format(Date())
    val fileName = file.substringAfterLast("/", file)
    var logMessage = "[$timestamp][$fileName:$line] $function: $message"
    
    if (error != null) {
      logMessage += " - Error: ${error.message}"
    }
    
    Log.e(tag, logMessage, error)
  }
  
  fun critical(message: String, error: Throwable? = null, tag: String = TAG, file: String = "", function: String = "", line: Int = 0) {
    val timestamp = SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault()).format(Date())
    val fileName = file.substringAfterLast("/", file)
    var logMessage = "[$timestamp][$fileName:$line] $function: $message"
    
    if (error != null) {
      logMessage += " - Error: ${error.message}"
    }
    
    Log.wtf(tag, logMessage, error)
  }
}
```

### 3. Crash Reporting y Monitoreo

#### Integración con Crashlytics (iOS)
```swift
// CrashReporter.swift
import Foundation
import FirebaseCrashlytics

class CrashReporter {
  static let shared = CrashReporter()
  
  private init() {}
  
  func logError(_ error: Error, context: String? = nil) {
    if let context = context {
      Crashlytics.crashlytics().setCustomValue(context, forKey: "error_context")
    }
    
    Crashlytics.crashlytics().record(error: error)
  }
  
  func logMessage(_ message: String, level: String = "info") {
    Crashlytics.crashlytics().log("\(level.uppercased()): \(message)")
  }
  
  func setUserIdentifier(_ identifier: String) {
    Crashlytics.crashlytics().setUserID(identifier)
  }
  
  func setCustomValue(_ value: Any, forKey key: String) {
    Crashlytics.crashlytics().setCustomValue(value, forKey: key)
  }
}
```

#### Integración con Crashlytics (Android)
```kotlin
// CrashReporter.kt
package com.miapp

import com.google.firebase.crashlytics.FirebaseCrashlytics

object CrashReporter {
  fun logError(error: Throwable, context: String? = null) {
    context?.let { 
      FirebaseCrashlytics.getInstance().setCustomKey("error_context", it)
    }
    
    FirebaseCrashlytics.getInstance().recordException(error)
  }
  
  fun logMessage(message: String, level: String = "info") {
    FirebaseCrashlytics.getInstance().log("${level.uppercase()}: $message")
  }
  
  fun setUserIdentifier(identifier: String) {
    FirebaseCrashlytics.getInstance().setUserId(identifier)
  }
  
  fun setCustomValue(value: String, key: String) {
    FirebaseCrashlytics.getInstance().setCustomKey(key, value)
  }
}
```

### 4. Uso en React Native

```tsx
// App.tsx
import React, { useState, useEffect } from 'react';
import { View, Text, Button, Alert, TextInput } from 'react-native';
import MyNativeModule from './MyNativeModule';

const App = () => {
  const [inputData, setInputData] = useState<string>('');
  const [result, setResult] = useState<string>('');
  const [isProcessing, setIsProcessing] = useState<boolean>(false);

  const handleProcessData = async () => {
    if (!inputData.trim()) {
      Alert.alert('Error', 'Por favor ingresa algún dato');
      return;
    }

    setIsProcessing(true);
    setResult('');

    try {
      const processedResult = await MyNativeModule.processData(inputData);
      setResult(processedResult);
      Alert.alert('Éxito', 'Datos procesados correctamente');
    } catch (error: any) {
      console.error('Error procesando datos:', error);
      
      // Mostrar error específico
      let errorMessage = 'Error desconocido';
      if (error.code === 'INVALID_INPUT') {
        errorMessage = 'Los datos de entrada no son válidos';
      } else if (error.code === 'NETWORK_ERROR') {
        errorMessage = 'Error de conectividad de red';
      } else if (error.code === 'PERMISSION_DENIED') {
        errorMessage = 'La aplicación no tiene permisos necesarios';
      }
      
      Alert.alert('Error', errorMessage);
    } finally {
      setIsProcessing(false);
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 }}>
      <Text style={{ fontSize: 24, marginBottom: 30, textAlign: 'center' }}>
        Módulo Nativo - Manejo de Errores
      </Text>
      
      <TextInput
        style={{
          borderWidth: 1,
          borderColor: '#ccc',
          borderRadius: 8,
          padding: 15,
          width: '100%',
          marginBottom: 20,
          fontSize: 16
        }}
        placeholder="Ingresa datos para procesar..."
        value={inputData}
        onChangeText={setInputData}
        multiline
      />
      
      <Button
        title={isProcessing ? "Procesando..." : "Procesar Datos"}
        onPress={handleProcessData}
        disabled={isProcessing || !inputData.trim()}
      />
      
      {result && (
        <View style={{ marginTop: 20, padding: 15, backgroundColor: '#e8f5e8', borderRadius: 8, width: '100%' }}>
          <Text style={{ fontSize: 16, fontWeight: 'bold', marginBottom: 10 }}>
            Resultado:
          </Text>
          <Text style={{ fontSize: 14 }}>
            {result}
          </Text>
        </View>
      )}
      
      <Text style={{ marginTop: 30, fontSize: 12, color: '#666', textAlign: 'center' }}>
        Esta app demuestra el manejo robusto de errores en módulos nativos,
        incluyendo logging, crash reporting y debugging avanzado.
      </Text>
    </View>
  );
};

export default App;
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Sistema de Logging Avanzado
Implementa un sistema de logging que incluya:

**Requisitos:**
- Diferentes niveles de log (debug, info, warning, error, critical)
- Rotación de logs automática
- Filtrado por categorías
- Exportación de logs para debugging

**Implementación:**
```typescript
interface AdvancedLogger {
  setLogLevel(level: 'debug' | 'info' | 'warning' | 'error' | 'critical'): void;
  log(level: string, category: string, message: string, data?: any): void;
  getLogs(category?: string, level?: string): Promise<string[]>;
  exportLogs(): Promise<string>;
  clearLogs(): void;
}
```

### Ejercicio 2: Módulo de Monitoreo de Salud
Crea un módulo que monitoree la salud del sistema:

**Requisitos:**
- Monitoreo de memoria y CPU
- Detección de memory leaks
- Alertas de rendimiento
- Métricas de salud del sistema

**Implementación:**
```typescript
interface SystemHealthModule {
  startMonitoring(): void;
  stopMonitoring(): void;
  getSystemMetrics(): Promise<SystemMetrics>;
  onHealthAlert(callback: (alert: HealthAlert) => void): void;
}

interface SystemMetrics {
  memoryUsage: number;
  cpuUsage: number;
  batteryLevel: number;
  networkStatus: string;
  timestamp: number;
}

interface HealthAlert {
  type: 'memory' | 'cpu' | 'battery' | 'network';
  severity: 'low' | 'medium' | 'high' | 'critical';
  message: string;
  metrics: SystemMetrics;
}
```

---

## 🔍 Puntos Clave

1. **Manejo de Errores**: Implementa enums/classes para errores específicos del dominio
2. **Logging**: Usa sistemas de logging nativos para debugging efectivo
3. **Crash Reporting**: Integra herramientas como Crashlytics para monitoreo en producción
4. **Validación**: Valida siempre los datos de entrada antes de procesarlos
5. **Contexto**: Proporciona contexto útil en logs y reportes de error

---

## 📖 Recursos Adicionales

- [iOS Logging](https://developer.apple.com/documentation/os/logging)
- [Android Logging](https://developer.android.com/reference/android/util/Log)
- [Firebase Crashlytics](https://firebase.google.com/docs/crashlytics)
- [React Native Error Handling](https://reactnative.dev/docs/error-boundaries)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Integración con APIs del Sistema** y cómo acceder a funcionalidades nativas avanzadas como cámara, galería, sensores y más.
