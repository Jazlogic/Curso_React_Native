# 📱 Clase 3: Eventos y Callbacks en Módulos Nativos

## 🎯 Objetivos de la Clase
- Implementar eventos nativos que se envían a JavaScript
- Crear callbacks para comunicación bidireccional
- Manejar suscripciones a eventos del sistema
- Implementar listeners para cambios en tiempo real

---

## 📚 Contenido Teórico

### 1. Eventos Nativos (Native Events)

Los eventos nativos permiten que el código nativo envíe información a JavaScript cuando ocurren cambios o eventos.

#### iOS - Implementación de Eventos
```swift
// MyNativeModule.swift
import Foundation
import React

@objc(MyNativeModule)
class MyNativeModule: NSObject {
  
  private var eventEmitter: RCTEventEmitter!
  
  override init() {
    super.init()
    eventEmitter = RCTEventEmitter()
  }
  
  // Enviar evento a JavaScript
  @objc
  func sendEvent(_ name: String, body: [String: Any]) {
    eventEmitter.sendEvent(withName: name, body: body)
  }
  
  // Método para suscribirse a cambios de batería
  @objc
  func startBatteryMonitoring() {
    UIDevice.current.isBatteryMonitoringEnabled = true
    
    // Observar cambios de batería
    NotificationCenter.default.addObserver(
      self,
      selector: #selector(batteryLevelChanged),
      name: UIDevice.batteryLevelDidChangeNotification,
      object: nil
    )
  }
  
  @objc
  private func batteryLevelChanged() {
    let batteryLevel = UIDevice.current.batteryLevel
    let isCharging = UIDevice.current.batteryState == .charging
    
    let eventBody: [String: Any] = [
      "batteryLevel": batteryLevel,
      "isCharging": isCharging,
      "timestamp": Date().timeIntervalSince1970
    ]
    
    sendEvent("BatteryChanged", body: eventBody)
  }
  
  // Método para detener monitoreo
  @objc
  func stopBatteryMonitoring() {
    UIDevice.current.isBatteryMonitoringEnabled = false
    NotificationCenter.default.removeObserver(self)
  }
}
```

#### Android - Implementación de Eventos
```kotlin
// MyNativeModule.kt
package com.miapp

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import com.facebook.react.bridge.*
import com.facebook.react.modules.core.DeviceEventManagerModule

class MyNativeModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
  
  private val batteryReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
      intent?.let { 
        val level = it.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
        val scale = it.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
        val batteryPct = level * 100 / scale.toFloat()
        
        val isCharging = it.getIntExtra(BatteryManager.EXTRA_STATUS, -1) == BatteryManager.BATTERY_STATUS_CHARGING
        
        val eventBody = mapOf(
          "batteryLevel" to batteryPct,
          "isCharging" to isCharging,
          "timestamp" to System.currentTimeMillis()
        )
        
        sendEvent("BatteryChanged", eventBody)
      }
    }
  }
  
  override fun getName(): String = "MyNativeModule"
  
  @ReactMethod
  fun startBatteryMonitoring() {
    val filter = IntentFilter().apply {
      addAction(Intent.ACTION_BATTERY_CHANGED)
    }
    reactApplicationContext.registerReceiver(batteryReceiver, filter)
  }
  
  @ReactMethod
  fun stopBatteryMonitoring() {
    try {
      reactApplicationContext.unregisterReceiver(batteryReceiver)
    } catch (e: Exception) {
      // Manejar error si el receiver no está registrado
    }
  }
  
  private fun sendEvent(eventName: String, params: Map<String, Any>) {
    reactApplicationContext
      .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
      .emit(eventName, Arguments.fromMap(params))
  }
}
```

### 2. Callbacks y Promesas

Los callbacks permiten que JavaScript pase funciones que el código nativo puede ejecutar.

#### Implementación con Callbacks
```typescript
// MyNativeModule.ts
interface MyNativeModuleInterface {
  // Método con callback
  performOperationWithCallback(
    data: string, 
    callback: (result: string, error?: string) => void
  ): void;
  
  // Método con Promise
  performOperationAsync(data: string): Promise<string>;
  
  // Método con múltiples callbacks
  startProcess(
    onProgress: (progress: number) => void,
    onComplete: (result: any) => void,
    onError: (error: string) => void
  ): void;
}
```

#### Uso en React Native
```tsx
// App.tsx
import React, { useState } from 'react';
import { View, Text, Button, Alert } from 'react-native';
import MyNativeModule from './MyNativeModule';

const App = () => {
  const [result, setResult] = useState<string>('');
  const [progress, setProgress] = useState<number>(0);

  const handleCallbackExample = () => {
    MyNativeModule.performOperationWithCallback(
      "datos de prueba",
      (result, error) => {
        if (error) {
          Alert.alert('Error', error);
        } else {
          setResult(result);
          Alert.alert('Éxito', result);
        }
      }
    );
  };

  const handlePromiseExample = async () => {
    try {
      const result = await MyNativeModule.performOperationAsync("datos async");
      setResult(result);
      Alert.alert('Éxito', result);
    } catch (error) {
      Alert.alert('Error', error.message);
    }
  };

  const handleProcessExample = () => {
    MyNativeModule.startProcess(
      (progress) => {
        setProgress(progress);
        console.log(`Progreso: ${progress}%`);
      },
      (result) => {
        setResult(JSON.stringify(result));
        Alert.alert('Completado', 'Proceso terminado exitosamente');
      },
      (error) => {
        Alert.alert('Error', error);
      }
    );
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 }}>
      <Text style={{ fontSize: 24, marginBottom: 20 }}>
        Módulo Nativo - Eventos y Callbacks
      </Text>
      
      <Button title="Ejemplo con Callback" onPress={handleCallbackExample} />
      <Button title="Ejemplo con Promise" onPress={handlePromiseExample} />
      <Button title="Proceso con Progreso" onPress={handleProcessExample} />
      
      {progress > 0 && (
        <Text style={{ marginTop: 20 }}>
          Progreso: {progress}%
        </Text>
      )}
      
      {result && (
        <Text style={{ marginTop: 20, textAlign: 'center' }}>
          Resultado: {result}
        </Text>
      )}
    </View>
  );
};

export default App;
```

### 3. Suscripciones a Eventos del Sistema

#### Implementación de Location Updates
```swift
// LocationModule.swift
import CoreLocation
import React

@objc(LocationModule)
class LocationModule: NSObject, CLLocationManagerDelegate {
  
  private let locationManager = CLLocationManager()
  private var eventEmitter: RCTEventEmitter!
  
  override init() {
    super.init()
    locationManager.delegate = self
    locationManager.desiredAccuracy = kCLLocationAccuracyBest
  }
  
  @objc
  func startLocationUpdates() {
    let authStatus = CLLocationManager.authorizationStatus()
    
    switch authStatus {
    case .notDetermined:
      locationManager.requestWhenInUseAuthorization()
    case .authorizedWhenInUse, .authorizedAlways:
      locationManager.startUpdatingLocation()
    default:
      // Manejar caso de autorización denegada
      break
    }
  }
  
  @objc
  func stopLocationUpdates() {
    locationManager.stopUpdatingLocation()
  }
  
  // CLLocationManagerDelegate
  func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.last else { return }
    
    let eventBody: [String: Any] = [
      "latitude": location.coordinate.latitude,
      "longitude": location.coordinate.longitude,
      "accuracy": location.horizontalAccuracy,
      "timestamp": location.timestamp.timeIntervalSince1970
    ]
    
    eventEmitter.sendEvent(withName: "LocationUpdated", body: eventBody)
  }
  
  func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
    let eventBody: [String: Any] = [
      "error": error.localizedDescription
    ]
    
    eventEmitter.sendEvent(withName: "LocationError", body: eventBody)
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Módulo de Notificaciones Push
Crea un módulo que maneje notificaciones push nativas:

**Requisitos:**
- Registrar el dispositivo para notificaciones
- Recibir notificaciones en primer plano
- Enviar eventos cuando lleguen notificaciones
- Manejar diferentes tipos de notificaciones

**Implementación básica:**
```typescript
interface PushNotificationModule {
  registerForPushNotifications(): Promise<string>;
  unregisterFromPushNotifications(): void;
  onNotificationReceived(callback: (notification: any) => void): void;
  onNotificationOpened(callback: (notification: any) => void): void;
}
```

### Ejercicio 2: Módulo de Sensores de Movimiento
Implementa un módulo que use el acelerómetro y giroscopio:

**Requisitos:**
- Iniciar/detener monitoreo de sensores
- Enviar datos de movimiento en tiempo real
- Calcular orientación del dispositivo
- Detectar gestos básicos (shake, tilt)

**Implementación:**
```typescript
interface MotionSensorsModule {
  startMotionTracking(): void;
  stopMotionTracking(): void;
  onMotionUpdate(callback: (data: MotionData) => void): void;
  onGestureDetected(callback: (gesture: string) => void): void;
}

interface MotionData {
  acceleration: { x: number; y: number; z: number };
  rotation: { x: number; y: number; z: number };
  timestamp: number;
}
```

---

## 🔍 Puntos Clave

1. **Eventos**: Usa `sendEvent` para comunicar cambios desde nativo a JavaScript
2. **Callbacks**: Implementa callbacks para operaciones asíncronas
3. **Promesas**: Usa Promises para operaciones que retornan valores
4. **Suscripciones**: Implementa listeners para eventos del sistema
5. **Limpieza**: Siempre limpia listeners y observers cuando se detengan

---

## 📖 Recursos Adicionales

- [React Native Event Emitter](https://reactnative.dev/docs/native-modules-ios#sending-events-to-javascript)
- [Android Event Emitter](https://reactnative.dev/docs/native-modules-android#sending-events-to-javascript)
- [Core Location Framework](https://developer.apple.com/documentation/corelocation)
- [Android Location Services](https://developer.android.com/training/location)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Manejo de Errores y Debugging** en módulos nativos, incluyendo logging, crash reporting y herramientas de desarrollo.
