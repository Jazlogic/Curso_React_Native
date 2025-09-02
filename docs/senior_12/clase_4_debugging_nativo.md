# 🔧 Clase 4: Debugging Nativo

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase dominarás el debugging nativo de aplicaciones React Native en iOS y Android. Aprenderás a usar Xcode, Android Studio, y herramientas nativas para debuggear crashes, memory leaks, y problemas de rendimiento a nivel nativo.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Debuggear** aplicaciones iOS con Xcode
2. **Debuggear** aplicaciones Android con Android Studio
3. **Analizar** crashes nativos y stack traces
4. **Profilar** performance nativo
5. **Identificar** memory leaks nativos
6. **Optimizar** código nativo

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Debugging iOS con Xcode**

#### **Configuración de Xcode para Debugging**
```bash
# Abrir proyecto en Xcode
cd ios && xed .

# Configurar scheme para debugging
# Product → Scheme → Edit Scheme → Run → Debug
```

#### **Herramientas de Debugging en Xcode**
- **LLDB Debugger**: Debugger de línea de comandos
- **Instruments**: Profiling y análisis de performance
- **Memory Graph Debugger**: Análisis de memoria
- **View Debugger**: Inspección de UI nativa
- **Network Debugger**: Análisis de requests HTTP

#### **Configuración de Breakpoints**
```swift
// En código Swift/Objective-C
func nativeFunction() {
    // Breakpoint aquí
    print("Debugging native code")
    
    // LLDB commands
    // po variableName - imprimir objeto
    // p variableName - imprimir valor
    // bt - backtrace
    // frame variable - variables del frame actual
}
```

#### **LLDB Commands Útiles**
```bash
# Comandos básicos de LLDB
(lldb) po self.view.frame
(lldb) p self.view.frame
(lldb) bt
(lldb) frame variable
(lldb) thread backtrace
(lldb) breakpoint set --name functionName
(lldb) breakpoint list
(lldb) breakpoint delete 1
```

### **2. Debugging Android con Android Studio**

#### **Configuración de Android Studio**
```bash
# Abrir proyecto en Android Studio
cd android && ./gradlew assembleDebug

# Configurar debugging
# Run → Debug 'app'
```

#### **Herramientas de Debugging en Android Studio**
- **Android Debugger**: Debugger integrado
- **Profiler**: Análisis de CPU, memoria y red
- **Memory Profiler**: Análisis de memoria
- **Network Profiler**: Análisis de requests HTTP
- **Layout Inspector**: Inspección de UI nativa

#### **Configuración de Breakpoints**
```java
// En código Java/Kotlin
public void nativeFunction() {
    // Breakpoint aquí
    Log.d("Debug", "Debugging native code");
    
    // Variables del debugger
    // this - objeto actual
    // variableName - variable local
    // System.out.println() - imprimir en consola
}
```

#### **Logcat Commands**
```bash
# Comandos de Logcat
adb logcat -s ReactNativeJS
adb logcat -s ReactNative
adb logcat -s System.err
adb logcat -s AndroidRuntime
adb logcat -c  # limpiar logs
```

### **3. Análisis de Crashes Nativos**

#### **Crash Reports en iOS**
```swift
// Configurar crash reporting
import Crashlytics

func setupCrashReporting() {
    // Configurar Crashlytics
    Crashlytics.sharedInstance().setUserIdentifier("user123")
    Crashlytics.sharedInstance().setObjectValue("v1.0", forKey: "version")
    
    // Log personalizado
    Crashlytics.sharedInstance().log("User performed action")
}

// Manejo de crashes
func handleCrash() {
    do {
        // Código que puede fallar
        try riskyOperation()
    } catch {
        // Log del error
        Crashlytics.sharedInstance().recordError(error)
    }
}
```

#### **Crash Reports en Android**
```java
// Configurar crash reporting
import com.crashlytics.android.Crashlytics;

public void setupCrashReporting() {
    // Configurar Crashlytics
    Crashlytics.setUserIdentifier("user123");
    Crashlytics.setString("version", "v1.0");
    
    // Log personalizado
    Crashlytics.log("User performed action");
}

// Manejo de crashes
public void handleCrash() {
    try {
        // Código que puede fallar
        riskyOperation();
    } catch (Exception e) {
        // Log del error
        Crashlytics.logException(e);
    }
}
```

#### **Análisis de Stack Traces**
```javascript
// En React Native
import {NativeModules} from 'react-native';

// Configurar crash reporting
NativeModules.Crashlytics.setUserIdentifier('user123');
NativeModules.Crashlytics.setString('version', 'v1.0');

// Log personalizado
NativeModules.Crashlytics.log('User performed action');

// Manejo de errores
try {
    // Código que puede fallar
    riskyOperation();
} catch (error) {
    // Log del error
    NativeModules.Crashlytics.recordError(error);
}
```

### **4. Profiling de Performance Nativo**

#### **Instruments en iOS**
```swift
// Configuración para profiling
import os.signpost

func profileFunction() {
    let signpostID = OSSignpostID(log: OSLog(subsystem: "com.app", category: "performance"))
    
    os_signpost(.begin, log: OSLog(subsystem: "com.app", category: "performance"), name: "Function", signpostID: signpostID)
    
    // Código a perfilar
    expensiveOperation()
    
    os_signpost(.end, log: OSLog(subsystem: "com.app", category: "performance"), name: "Function", signpostID: signpostID)
}
```

#### **Android Profiler**
```java
// Configuración para profiling
import android.os.Trace;

public void profileFunction() {
    Trace.beginSection("Function");
    
    try {
        // Código a perfilar
        expensiveOperation();
    } finally {
        Trace.endSection();
    }
}
```

#### **Métricas de Performance**
```javascript
// En React Native
import {NativeModules} from 'react-native';

// Métricas de performance
const performanceMetrics = {
    startTime: Date.now(),
    endTime: null,
    duration: null,
};

// Iniciar profiling
function startProfiling() {
    performanceMetrics.startTime = Date.now();
}

// Finalizar profiling
function endProfiling() {
    performanceMetrics.endTime = Date.now();
    performanceMetrics.duration = performanceMetrics.endTime - performanceMetrics.startTime;
    
    console.log('Performance metrics:', performanceMetrics);
}
```

### **5. Memory Leaks y Optimización**

#### **Memory Leaks en iOS**
```swift
// Detectar memory leaks
import Foundation

class MemoryLeakDetector {
    private var objects: [AnyObject] = []
    
    func trackObject(_ object: AnyObject) {
        objects.append(object)
    }
    
    func checkForLeaks() {
        for object in objects {
            if CFGetRetainCount(object) > 1 {
                print("Potential memory leak: \(object)")
            }
        }
    }
}

// Uso
let detector = MemoryLeakDetector()
detector.trackObject(someObject)
detector.checkForLeaks()
```

#### **Memory Leaks en Android**
```java
// Detectar memory leaks
import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.List;

public class MemoryLeakDetector {
    private List<WeakReference<Object>> objects = new ArrayList<>();
    
    public void trackObject(Object object) {
        objects.add(new WeakReference<>(object));
    }
    
    public void checkForLeaks() {
        for (WeakReference<Object> ref : objects) {
            if (ref.get() != null) {
                System.out.println("Potential memory leak: " + ref.get());
            }
        }
    }
}
```

#### **Optimización de Memoria**
```javascript
// En React Native
import {NativeModules} from 'react-native';

// Optimización de memoria
class MemoryOptimizer {
    constructor() {
        this.objectPool = new Map();
    }
    
    // Reutilizar objetos
    getObject(type) {
        if (this.objectPool.has(type)) {
            return this.objectPool.get(type);
        }
        return this.createObject(type);
    }
    
    // Devolver objeto al pool
    returnObject(type, object) {
        this.objectPool.set(type, object);
    }
    
    // Limpiar pool
    clearPool() {
        this.objectPool.clear();
    }
}
```

### **6. Herramientas de Debugging Avanzadas**

#### **React Native Debugger**
```javascript
// Configuración de React Native Debugger
import {NativeModules} from 'react-native';

// Habilitar debugging
if (__DEV__) {
    NativeModules.DevSettings.setIsDebuggingRemotely(true);
}

// Configurar Redux DevTools
import {createStore} from 'redux';
import {composeWithDevTools} from 'redux-devtools-extension';

const store = createStore(
    rootReducer,
    composeWithDevTools()
);
```

#### **Flipper Integration**
```javascript
// Configuración de Flipper
import {Flipper} from 'react-native-flipper';

// Plugin personalizado
Flipper.addPlugin({
    getId() {
        return 'CustomPlugin';
    },
    onConnect(connection) {
        this.connection = connection;
    },
    onDisconnect() {
        this.connection = null;
    },
});
```

#### **Chrome DevTools**
```javascript
// Configuración de Chrome DevTools
import {NativeModules} from 'react-native';

// Habilitar debugging
if (__DEV__) {
    NativeModules.DevSettings.setIsDebuggingRemotely(true);
}

// Configurar breakpoints
debugger; // Breakpoint en JavaScript
```

---

## 🛠️ Contenido Práctico

### **Ejercicio 1: Debugging iOS**

#### **Objetivo**
Configurar y usar Xcode para debugging de una aplicación React Native.

#### **Pasos**
1. **Abrir proyecto** en Xcode
2. **Configurar breakpoints** en código nativo
3. **Usar LLDB** para inspeccionar variables
4. **Analizar** stack traces

#### **Código de Ejemplo**
```swift
// NativeModule.swift
import Foundation
import React

@objc(NativeModule)
class NativeModule: NSObject {
    
    @objc
    func nativeFunction(_ callback: @escaping RCTResponseSenderBlock) {
        // Breakpoint aquí
        let result = performNativeOperation()
        callback([result])
    }
    
    private func performNativeOperation() -> String {
        // Código nativo
        return "Native operation completed"
    }
}
```

### **Ejercicio 2: Debugging Android**

#### **Objetivo**
Configurar y usar Android Studio para debugging de una aplicación React Native.

#### **Pasos**
1. **Abrir proyecto** en Android Studio
2. **Configurar breakpoints** en código nativo
3. **Usar debugger** para inspeccionar variables
4. **Analizar** logs en Logcat

#### **Código de Ejemplo**
```java
// NativeModule.java
package com.app;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Callback;

public class NativeModule extends ReactContextBaseJavaModule {
    
    public NativeModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    
    @Override
    public String getName() {
        return "NativeModule";
    }
    
    @ReactMethod
    public void nativeFunction(Callback callback) {
        // Breakpoint aquí
        String result = performNativeOperation();
        callback.invoke(result);
    }
    
    private String performNativeOperation() {
        // Código nativo
        return "Native operation completed";
    }
}
```

### **Ejercicio 3: Crash Analysis**

#### **Objetivo**
Implementar crash reporting y análisis de crashes.

#### **Código de Ejemplo**
```javascript
// CrashReporting.js
import {NativeModules} from 'react-native';

class CrashReporting {
    constructor() {
        this.setupCrashReporting();
    }
    
    setupCrashReporting() {
        // Configurar crash reporting
        NativeModules.Crashlytics.setUserIdentifier('user123');
        NativeModules.Crashlytics.setString('version', 'v1.0');
        
        // Manejar errores no capturados
        global.ErrorUtils.setGlobalHandler((error, isFatal) => {
            this.logError(error, isFatal);
        });
    }
    
    logError(error, isFatal) {
        // Log del error
        NativeModules.Crashlytics.logException(error);
        
        if (isFatal) {
            // Error fatal
            console.error('Fatal error:', error);
        } else {
            // Error no fatal
            console.warn('Non-fatal error:', error);
        }
    }
    
    logEvent(eventName, parameters) {
        // Log de evento
        NativeModules.Crashlytics.log(eventName);
        
        if (parameters) {
            Object.keys(parameters).forEach(key => {
                NativeModules.Crashlytics.setString(key, parameters[key]);
            });
        }
    }
}

export default new CrashReporting();
```

### **Ejercicio 4: Performance Profiling**

#### **Objetivo**
Implementar profiling de performance nativo.

#### **Código de Ejemplo**
```javascript
// PerformanceProfiler.js
import {NativeModules} from 'react-native';

class PerformanceProfiler {
    constructor() {
        this.metrics = new Map();
    }
    
    startProfiling(name) {
        const startTime = Date.now();
        this.metrics.set(name, {startTime});
    }
    
    endProfiling(name) {
        const metric = this.metrics.get(name);
        if (metric) {
            const endTime = Date.now();
            const duration = endTime - metric.startTime;
            
            metric.endTime = endTime;
            metric.duration = duration;
            
            console.log(`Performance: ${name} took ${duration}ms`);
            
            // Log en crash reporting
            NativeModules.Crashlytics.log(`Performance: ${name} - ${duration}ms`);
        }
    }
    
    getMetrics() {
        return Array.from(this.metrics.entries()).map(([name, metric]) => ({
            name,
            duration: metric.duration,
        }));
    }
    
    clearMetrics() {
        this.metrics.clear();
    }
}

export default new PerformanceProfiler();
```

---

## 🎯 Ejercicios de Evaluación

### **Ejercicio 1: Debugging Setup**
- Configurar Xcode para debugging iOS
- Configurar Android Studio para debugging Android
- Verificar que los breakpoints funcionen correctamente

### **Ejercicio 2: Crash Analysis**
- Implementar crash reporting
- Simular un crash y analizar el reporte
- Configurar alertas para crashes críticos

### **Ejercicio 3: Performance Profiling**
- Implementar profiling de performance
- Identificar bottlenecks de rendimiento
- Optimizar código basado en los resultados

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [Xcode Debugging](https://developer.apple.com/documentation/xcode/debugging)
- [Android Studio Debugging](https://developer.android.com/studio/debug)
- [React Native Debugging](https://reactnative.dev/docs/debugging)

### **Herramientas Útiles**
- [LLDB Documentation](https://lldb.llvm.org/)
- [Android Profiler](https://developer.android.com/studio/profile)
- [Instruments](https://developer.apple.com/documentation/xcode/instruments)

### **Mejores Prácticas**
- Usar breakpoints estratégicamente
- Analizar crashes regularmente
- Profilar performance en builds de release
- Documentar problemas y soluciones

---

## 🚀 Siguiente Clase

En la próxima clase aprenderás sobre **Profiling y Análisis de Performance**, donde profundizarás en el análisis avanzado de rendimiento, métricas de performance y optimización de aplicaciones React Native.

---

**💡 Consejo**: El debugging nativo es esencial para aplicaciones React Native complejas. Invierte tiempo en dominar las herramientas nativas para resolver problemas avanzados.
