# 📱 Clase 5: Integración con APIs del Sistema

## 🎯 Objetivos de la Clase
- Integrar módulos nativos con APIs del sistema operativo
- Acceder a funcionalidades como cámara, galería y sensores
- Implementar módulos para hardware específico del dispositivo
- Crear módulos que interactúen con servicios del sistema

---

## 📚 Contenido Teórico

### 1. Módulo de Cámara Nativa

#### iOS - Implementación de Cámara
```swift
// CameraModule.swift
import Foundation
import UIKit
import AVFoundation
import React

@objc(CameraModule)
class CameraModule: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
  
  private var resolve: RCTPromiseResolveBlock?
  private var reject: RCTPromiseRejectBlock?
  private var imagePicker: UIImagePickerController?
  
  @objc
  func takePhoto(_ options: [String: Any], resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    
    self.resolve = resolve
    self.reject = reject
    
    // Verificar permisos de cámara
    let authStatus = AVCaptureDevice.authorizationStatus(for: .video)
    
    switch authStatus {
    case .authorized:
      presentImagePicker(sourceType: .camera, options: options)
    case .notDetermined:
      AVCaptureDevice.requestAccess(for: .video) { [weak self] granted in
        DispatchQueue.main.async {
          if granted {
            self?.presentImagePicker(sourceType: .camera, options: options)
          } else {
            self?.reject?("CAMERA_PERMISSION_DENIED", "Permiso de cámara denegado", nil)
          }
        }
      }
    default:
      reject("CAMERA_PERMISSION_DENIED", "Permiso de cámara denegado", nil)
    }
  }
  
  @objc
  func pickImageFromGallery(_ options: [String: Any], resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    
    self.resolve = resolve
    self.reject = reject
    
    presentImagePicker(sourceType: .photoLibrary, options: options)
  }
  
  private func presentImagePicker(sourceType: UIImagePickerController.SourceType, options: [String: Any]) {
    imagePicker = UIImagePickerController()
    imagePicker?.delegate = self
    imagePicker?.sourceType = sourceType
    
    // Configurar opciones
    if let allowsEditing = options["allowsEditing"] as? Bool {
      imagePicker?.allowsEditing = allowsEditing
    }
    
    if let quality = options["quality"] as? Float {
      // La calidad se maneja en el delegate
    }
    
    // Presentar el picker
    if let rootViewController = RCTPresentedViewController() {
      rootViewController.present(imagePicker!, animated: true)
    } else {
      reject("PRESENTATION_ERROR", "No se pudo presentar la cámara", nil)
    }
  }
  
  // MARK: - UIImagePickerControllerDelegate
  
  func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
    
    picker.dismiss(animated: true) { [weak self] in
      guard let self = self else { return }
      
      var image: UIImage?
      
      if let editedImage = info[.editedImage] as? UIImage {
        image = editedImage
      } else if let originalImage = info[.originalImage] as? UIImage {
        image = originalImage
      }
      
      guard let finalImage = image else {
        self.reject?("IMAGE_PROCESSING_ERROR", "No se pudo procesar la imagen", nil)
        return
      }
      
      // Convertir imagen a base64 o guardar en archivo
      if let imageData = finalImage.jpegData(compressionQuality: 0.8) {
        let base64String = imageData.base64EncodedString()
        
        let result: [String: Any] = [
          "base64": base64String,
          "width": finalImage.size.width,
          "height": finalImage.size.height,
          "mimeType": "image/jpeg"
        ]
        
        self.resolve?(result)
      } else {
        self.reject?("IMAGE_CONVERSION_ERROR", "No se pudo convertir la imagen", nil)
      }
    }
  }
  
  func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
    picker.dismiss(animated: true) { [weak self] in
      self?.reject?("USER_CANCELLED", "Usuario canceló la selección", nil)
    }
  }
}
```

#### Android - Implementación de Cámara
```kotlin
// CameraModule.kt
package com.miapp

import android.Manifest
import android.app.Activity
import android.content.Intent
import android.content.pm.PackageManager
import android.graphics.Bitmap
import android.net.Uri
import android.provider.MediaStore
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import com.facebook.react.bridge.*
import com.facebook.react.modules.core.ActivityEventListener
import com.facebook.react.modules.core.PermissionAwareActivity
import java.io.ByteArrayOutputStream
import java.io.File
import java.util.*

class CameraModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext), ActivityEventListener {
  
  private val CAMERA_PERMISSION_REQUEST = 1001
  private val CAMERA_REQUEST = 1002
  private val GALLERY_REQUEST = 1003
  
  private var resolve: Promise? = null
  private var reject: Promise? = null
  private var currentOptions: Map<String, Any>? = null
  
  init {
    reactContext.addActivityEventListener(this)
  }
  
  override fun getName(): String = "CameraModule"
  
  @ReactMethod
  fun takePhoto(options: ReadableMap, promise: Promise) {
    this.resolve = promise
    this.reject = promise
    this.currentOptions = options.toHashMap()
    
    val activity = currentActivity
    if (activity == null) {
      promise.reject("ACTIVITY_ERROR", "No hay actividad disponible", null)
      return
    }
    
    if (checkCameraPermission(activity)) {
      openCamera(activity)
    } else {
      requestCameraPermission(activity)
    }
  }
  
  @ReactMethod
  fun pickImageFromGallery(options: ReadableMap, promise: Promise) {
    this.resolve = promise
    this.reject = promise
    this.currentOptions = options.toHashMap()
    
    val activity = currentActivity
    if (activity == null) {
      promise.reject("ACTIVITY_ERROR", "No hay actividad disponible", null)
      return
    }
    
    openGallery(activity)
  }
  
  private fun checkCameraPermission(activity: Activity): Boolean {
    return ContextCompat.checkSelfPermission(activity, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED
  }
  
  private fun requestCameraPermission(activity: Activity) {
    ActivityCompat.requestPermissions(activity, arrayOf(Manifest.permission.CAMERA), CAMERA_PERMISSION_REQUEST)
  }
  
  private fun openCamera(activity: Activity) {
    val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
    activity.startActivityForResult(intent, CAMERA_REQUEST)
  }
  
  private fun openGallery(activity: Activity) {
    val intent = Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI)
    activity.startActivityForResult(intent, GALLERY_REQUEST)
  }
  
  override fun onActivityResult(activity: Activity, requestCode: Int, resultCode: Int, data: Intent?) {
    if (resultCode == Activity.RESULT_OK) {
      when (requestCode) {
        CAMERA_REQUEST -> {
          val imageBitmap = data?.extras?.get("data") as? Bitmap
          if (imageBitmap != null) {
            processImage(imageBitmap)
          } else {
            reject?.reject("IMAGE_ERROR", "No se pudo obtener la imagen de la cámara", null)
          }
        }
        GALLERY_REQUEST -> {
          val imageUri = data?.data
          if (imageUri != null) {
            val imageBitmap = MediaStore.Images.Media.getBitmap(activity.contentResolver, imageUri)
            processImage(imageBitmap)
          } else {
            reject?.reject("IMAGE_ERROR", "No se pudo obtener la imagen de la galería", null)
          }
        }
      }
    } else if (resultCode == Activity.RESULT_CANCELED) {
      reject?.reject("USER_CANCELLED", "Usuario canceló la selección", null)
    }
  }
  
  private fun processImage(bitmap: Bitmap) {
    try {
      val quality = (currentOptions?.get("quality") as? Double)?.toFloat() ?: 0.8f
      val byteArrayOutputStream = ByteArrayOutputStream()
      bitmap.compress(Bitmap.CompressFormat.JPEG, (quality * 100).toInt(), byteArrayOutputStream)
      
      val imageBytes = byteArrayOutputStream.toByteArray()
      val base64String = android.util.Base64.encodeToString(imageBytes, android.util.Base64.DEFAULT)
      
      val result = Arguments.createMap().apply {
        putString("base64", base64String)
        putDouble("width", bitmap.width.toDouble())
        putDouble("height", bitmap.height.toDouble())
        putString("mimeType", "image/jpeg")
      }
      
      resolve?.resolve(result)
    } catch (e: Exception) {
      reject?.reject("IMAGE_PROCESSING_ERROR", "Error procesando la imagen: ${e.message}", e)
    }
  }
  
  override fun onNewIntent(intent: Intent?) {
    // No es necesario para este módulo
  }
}
```

### 2. Módulo de Sensores del Sistema

#### Implementación de Sensores
```swift
// SensorsModule.swift
import Foundation
import CoreMotion
import CoreLocation
import React

@objc(SensorsModule)
class SensorsModule: NSObject {
  
  private let motionManager = CMMotionManager()
  private let locationManager = CLLocationManager()
  private var eventEmitter: RCTEventEmitter!
  
  override init() {
    super.init()
    setupLocationManager()
  }
  
  @objc
  func startAccelerometer(_ updateInterval: Double, resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    
    guard motionManager.isAccelerometerAvailable else {
      reject("SENSOR_NOT_AVAILABLE", "Acelerómetro no disponible", nil)
      return
    }
    
    motionManager.accelerometerUpdateInterval = updateInterval
    
    motionManager.startAccelerometerUpdates(to: .main) { [weak self] data, error in
      if let error = error {
        self?.eventEmitter.sendEvent(withName: "AccelerometerError", body: ["error": error.localizedDescription])
        return
      }
      
      guard let data = data else { return }
      
      let eventBody: [String: Any] = [
        "x": data.acceleration.x,
        "y": data.acceleration.y,
        "z": data.acceleration.z,
        "timestamp": data.timestamp
      ]
      
      self?.eventEmitter.sendEvent(withName: "AccelerometerData", body: eventBody)
    }
    
    resolve(true)
  }
  
  @objc
  func stopAccelerometer() {
    motionManager.stopAccelerometerUpdates()
  }
  
  @objc
  func startGyroscope(_ updateInterval: Double, resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseRejectBlock) {
    
    guard motionManager.isGyroAvailable else {
      reject("SENSOR_NOT_AVAILABLE", "Giroscopio no disponible", nil)
      return
    }
    
    motionManager.gyroUpdateInterval = updateInterval
    
    motionManager.startGyroUpdates(to: .main) { [weak self] data, error in
      if let error = error {
        self?.eventEmitter.sendEvent(withName: "GyroscopeError", body: ["error": error.localizedDescription])
        return
      }
      
      guard let data = data else { return }
      
      let eventBody: [String: Any] = [
        "x": data.rotationRate.x,
        "y": data.rotationRate.y,
        "z": data.rotationRate.z,
        "timestamp": data.timestamp
      ]
      
      self?.eventEmitter.sendEvent(withName: "GyroscopeData", body: eventBody)
    }
    
    resolve(true)
  }
  
  @objc
  func stopGyroscope() {
    motionManager.stopGyroUpdates()
  }
  
  private func setupLocationManager() {
    locationManager.delegate = self
    locationManager.desiredAccuracy = kCLLocationAccuracyBest
  }
  
  @objc
  func getCurrentLocation(_ resolver resolve: @escaping RCTPromiseResolveBlock, rejecter reject: @escaping RCTPromiseResolveBlock) {
    
    let authStatus = CLLocationManager.authorizationStatus()
    
    switch authStatus {
    case .notDetermined:
      locationManager.requestWhenInUseAuthorization()
      reject("LOCATION_PERMISSION_PENDING", "Esperando permiso de ubicación", nil)
    case .denied, .restricted:
      reject("LOCATION_PERMISSION_DENIED", "Permiso de ubicación denegado", nil)
    case .authorizedWhenInUse, .authorizedAlways:
      locationManager.requestLocation()
      // El resultado se maneja en el delegate
    @unknown default:
      reject("LOCATION_PERMISSION_UNKNOWN", "Estado de permiso desconocido", nil)
    }
  }
}

// MARK: - CLLocationManagerDelegate
extension SensorsModule: CLLocationManagerDelegate {
  
  func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.last else { return }
    
    let eventBody: [String: Any] = [
      "latitude": location.coordinate.latitude,
      "longitude": location.coordinate.longitude,
      "accuracy": location.horizontalAccuracy,
      "altitude": location.altitude,
      "speed": location.speed,
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
  
  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    let eventBody: [String: Any] = [
      "status": status.rawValue,
      "statusString": status.description
    ]
    
    eventEmitter.sendEvent(withName: "LocationPermissionChanged", body: eventBody)
  }
}
```

### 3. Uso en React Native

```tsx
// App.tsx
import React, { useState, useEffect } from 'react';
import { View, Text, Button, Alert, ScrollView, StyleSheet } from 'react-native';
import { CameraModule, SensorsModule } from './NativeModules';

const App = () => {
  const [cameraResult, setCameraResult] = useState<any>(null);
  const [sensorData, setSensorData] = useState<any>({});
  const [isMonitoring, setIsMonitoring] = useState<boolean>(false);

  useEffect(() => {
    // Configurar listeners para eventos de sensores
    const subscription = SensorsModule.addListener('AccelerometerData', (data) => {
      setSensorData(prev => ({ ...prev, accelerometer: data }));
    });

    return () => subscription?.remove();
  }, []);

  const handleTakePhoto = async () => {
    try {
      const result = await CameraModule.takePhoto({
        allowsEditing: true,
        quality: 0.8
      });
      
      setCameraResult(result);
      Alert.alert('Éxito', 'Foto tomada correctamente');
    } catch (error: any) {
      if (error.code === 'CAMERA_PERMISSION_DENIED') {
        Alert.alert('Permiso Denegado', 'Necesitas dar permiso para usar la cámara');
      } else if (error.code === 'USER_CANCELLED') {
        console.log('Usuario canceló la foto');
      } else {
        Alert.alert('Error', error.message || 'Error al tomar la foto');
      }
    }
  };

  const handlePickImage = async () => {
    try {
      const result = await CameraModule.pickImageFromGallery({
        allowsEditing: true,
        quality: 0.8
      });
      
      setCameraResult(result);
      Alert.alert('Éxito', 'Imagen seleccionada correctamente');
    } catch (error: any) {
      if (error.code === 'USER_CANCELLED') {
        console.log('Usuario canceló la selección');
      } else {
        Alert.alert('Error', error.message || 'Error al seleccionar imagen');
      }
    }
  };

  const handleStartSensors = async () => {
    try {
      await SensorsModule.startAccelerometer(0.1); // 100ms
      await SensorsModule.startGyroscope(0.1);
      setIsMonitoring(true);
      Alert.alert('Sensores', 'Monitoreo de sensores iniciado');
    } catch (error: any) {
      Alert.alert('Error', error.message || 'Error al iniciar sensores');
    }
  };

  const handleStopSensors = () => {
    SensorsModule.stopAccelerometer();
    SensorsModule.stopGyroscope();
    setIsMonitoring(false);
    Alert.alert('Sensores', 'Monitoreo de sensores detenido');
  };

  const handleGetLocation = async () => {
    try {
      await SensorsModule.getCurrentLocation();
      Alert.alert('Ubicación', 'Solicitando ubicación actual...');
    } catch (error: any) {
      if (error.code === 'LOCATION_PERMISSION_DENIED') {
        Alert.alert('Permiso Denegado', 'Necesitas dar permiso para acceder a la ubicación');
      } else {
        Alert.alert('Error', error.message || 'Error al obtener ubicación');
      }
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>
        APIs Nativas del Sistema
      </Text>
      
      {/* Sección de Cámara */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>📷 Cámara y Galería</Text>
        <Button title="Tomar Foto" onPress={handleTakePhoto} />
        <Button title="Seleccionar de Galería" onPress={handlePickImage} />
        
        {cameraResult && (
          <View style={styles.resultContainer}>
            <Text style={styles.resultTitle}>Resultado:</Text>
            <Text>Dimensiones: {cameraResult.width} x {cameraResult.height}</Text>
            <Text>Tipo: {cameraResult.mimeType}</Text>
            <Text>Base64: {cameraResult.base64.substring(0, 50)}...</Text>
          </View>
        )}
      </View>
      
      {/* Sección de Sensores */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>📡 Sensores del Sistema</Text>
        
        {!isMonitoring ? (
          <Button title="Iniciar Monitoreo de Sensores" onPress={handleStartSensors} />
        ) : (
          <Button title="Detener Monitoreo de Sensores" onPress={handleStopSensors} />
        )}
        
        <Button title="Obtener Ubicación Actual" onPress={handleGetLocation} />
        
        {sensorData.accelerometer && (
          <View style={styles.resultContainer}>
            <Text style={styles.resultTitle}>Acelerómetro:</Text>
            <Text>X: {sensorData.accelerometer.x.toFixed(3)}</Text>
            <Text>Y: {sensorData.accelerometer.y.toFixed(3)}</Text>
            <Text>Z: {sensorData.accelerometer.z.toFixed(3)}</Text>
          </View>
        )}
      </View>
      
      <Text style={styles.footer}>
        Esta app demuestra la integración con APIs nativas del sistema,
        incluyendo cámara, sensores y ubicación.
      </Text>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333'
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  sectionTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333'
  },
  resultContainer: {
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 8,
    marginTop: 15
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#495057'
  },
  footer: {
    fontSize: 12,
    color: '#666',
    textAlign: 'center',
    marginTop: 20,
    lineHeight: 18
  }
});

export default App;
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Módulo de Notificaciones Push
Implementa un módulo nativo para notificaciones push:

**Requisitos:**
- Registrar dispositivo para notificaciones
- Recibir notificaciones en primer plano
- Manejar diferentes tipos de notificaciones
- Enviar eventos a JavaScript

**Implementación:**
```typescript
interface PushNotificationModule {
  registerForPushNotifications(): Promise<string>;
  unregisterFromPushNotifications(): void;
  onNotificationReceived(callback: (notification: any) => void): void;
  onNotificationOpened(callback: (notification: any) => void): void;
  getBadgeCount(): Promise<number>;
  setBadgeCount(count: number): void;
}
```

### Ejercicio 2: Módulo de Biometría
Crea un módulo para autenticación biométrica:

**Requisitos:**
- Verificar disponibilidad de biometría
- Autenticar usuario con huella dactilar/face ID
- Manejar diferentes tipos de biometría
- Configurar políticas de seguridad

**Implementación:**
```typescript
interface BiometricModule {
  isBiometricAvailable(): Promise<boolean>;
  getBiometricType(): Promise<'fingerprint' | 'face' | 'iris' | 'none'>;
  authenticate(reason: string): Promise<boolean>;
  onAuthenticationSuccess(callback: () => void): void;
  onAuthenticationError(callback: (error: string) => void): void;
}
```

---

## 🔍 Puntos Clave

1. **Permisos**: Siempre verifica y solicita permisos antes de usar APIs del sistema
2. **Eventos**: Usa eventos para comunicar cambios en tiempo real desde nativo a JavaScript
3. **Manejo de Errores**: Implementa manejo robusto de errores para APIs del sistema
4. **Configuración**: Configura correctamente los managers nativos (cámara, sensores, ubicación)
5. **Limpieza**: Detén y limpia recursos cuando no se necesiten

---

## 📖 Recursos Adicionales

- [iOS Camera Framework](https://developer.apple.com/documentation/avfoundation)
- [Android Camera API](https://developer.android.com/guide/topics/media/camera)
- [Core Motion Framework](https://developer.apple.com/documentation/coremotion)
- [Android Sensors](https://developer.android.com/guide/topics/sensors)

---

## 🎉 ¡Módulo Completado!

Has completado exitosamente el **Módulo 11: APIs Nativas** del curso de React Native. 

**Lo que has aprendido:**
- ✅ Fundamentos de módulos nativos
- ✅ Creación de módulos nativos para iOS y Android
- ✅ Eventos y callbacks bidireccionales
- ✅ Manejo robusto de errores y debugging
- ✅ Integración con APIs del sistema

**Próximo paso:** Continuar con el **Módulo 12: CI/CD y Deployment** para aprender a generar APKs y desplegar aplicaciones en producción.
