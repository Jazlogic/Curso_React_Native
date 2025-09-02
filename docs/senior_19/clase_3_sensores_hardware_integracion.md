# Clase 3: Integración de Sensores y Hardware

## Objetivos de la Clase
- Comprender la integración de sensores en aplicaciones React Native
- Aprender sobre el acceso a hardware del dispositivo
- Implementar funcionalidades que utilicen sensores
- Optimizar el uso de recursos del dispositivo

## Contenido de la Clase

### 1. Introducción a los Sensores en React Native

#### Tipos de Sensores Disponibles
- **Acelerómetro:** Detección de movimiento y orientación
- **Giroscopio:** Rotación y orientación espacial
- **Magnetómetro:** Campo magnético y brújula
- **Proximidad:** Detección de objetos cercanos
- **Luz:** Nivel de iluminación ambiental
- **Presión:** Altitud y presión atmosférica
- **Humedad:** Nivel de humedad ambiental
- **Temperatura:** Temperatura del dispositivo

#### Librerías para Sensores
```bash
npm install react-native-sensors
npm install @react-native-community/geolocation
npm install react-native-device-info
npm install react-native-orientation-locker
```

### 2. Configuración de Sensores

#### Permisos Necesarios
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.VIBRATE" />
```

```xml
<!-- ios/Info.plist -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Esta app necesita acceso a la ubicación para funcionalidades de sensores</string>
<key>NSCameraUsageDescription</key>
<string>Esta app necesita acceso a la cámara para funcionalidades de sensores</string>
```

### 3. Implementación de Sensores

#### Acelerómetro y Giroscopio
```jsx
// SensorManager.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { accelerometer, gyroscope, setUpdateIntervalForType, SensorTypes } from 'react-native-sensors';

const SensorManager = () => {
  const [accelerometerData, setAccelerometerData] = useState({ x: 0, y: 0, z: 0 });
  const [gyroscopeData, setGyroscopeData] = useState({ x: 0, y: 0, z: 0 });
  const [isListening, setIsListening] = useState(false);

  useEffect(() => {
    setUpdateIntervalForType(SensorTypes.accelerometer, 100);
    setUpdateIntervalForType(SensorTypes.gyroscope, 100);
  }, []);

  const startListening = () => {
    setIsListening(true);
    
    const accelerometerSubscription = accelerometer.subscribe(({ x, y, z }) => {
      setAccelerometerData({ x, y, z });
    });

    const gyroscopeSubscription = gyroscope.subscribe(({ x, y, z }) => {
      setGyroscopeData({ x, y, z });
    });

    return () => {
      accelerometerSubscription.unsubscribe();
      gyroscopeSubscription.unsubscribe();
    };
  };

  const stopListening = () => {
    setIsListening(false);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Datos de Sensores</Text>
      
      <View style={styles.sensorContainer}>
        <Text style={styles.sensorTitle}>Acelerómetro</Text>
        <Text style={styles.sensorData}>X: {accelerometerData.x.toFixed(2)}</Text>
        <Text style={styles.sensorData}>Y: {accelerometerData.y.toFixed(2)}</Text>
        <Text style={styles.sensorData}>Z: {accelerometerData.z.toFixed(2)}</Text>
      </View>

      <View style={styles.sensorContainer}>
        <Text style={styles.sensorTitle}>Giroscopio</Text>
        <Text style={styles.sensorData}>X: {gyroscopeData.x.toFixed(2)}</Text>
        <Text style={styles.sensorData}>Y: {gyroscopeData.y.toFixed(2)}</Text>
        <Text style={styles.sensorData}>Z: {gyroscopeData.z.toFixed(2)}</Text>
      </View>

      <TouchableOpacity
        style={[styles.button, isListening ? styles.stopButton : styles.startButton]}
        onPress={isListening ? stopListening : startListening}
      >
        <Text style={styles.buttonText}>
          {isListening ? 'Detener' : 'Iniciar'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30
  },
  sensorContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  sensorTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333'
  },
  sensorData: {
    fontSize: 16,
    marginBottom: 5,
    color: '#666'
  },
  button: {
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginTop: 20
  },
  startButton: {
    backgroundColor: '#4CAF50'
  },
  stopButton: {
    backgroundColor: '#f44336'
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  }
});

export default SensorManager;
```

#### Geolocalización
```jsx
// LocationManager.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, Alert } from 'react-native';
import Geolocation from '@react-native-community/geolocation';

const LocationManager = () => {
  const [location, setLocation] = useState(null);
  const [isTracking, setIsTracking] = useState(false);
  const [watchId, setWatchId] = useState(null);

  const getCurrentLocation = () => {
    Geolocation.getCurrentPosition(
      (position) => {
        setLocation({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          accuracy: position.coords.accuracy,
          timestamp: position.timestamp
        });
      },
      (error) => {
        Alert.alert('Error', 'No se pudo obtener la ubicación');
        console.log(error);
      },
      { enableHighAccuracy: true, timeout: 15000, maximumAge: 10000 }
    );
  };

  const startTracking = () => {
    const id = Geolocation.watchPosition(
      (position) => {
        setLocation({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          accuracy: position.coords.accuracy,
          timestamp: position.timestamp
        });
      },
      (error) => {
        Alert.alert('Error', 'Error en el seguimiento de ubicación');
        console.log(error);
      },
      { enableHighAccuracy: true, distanceFilter: 10 }
    );
    setWatchId(id);
    setIsTracking(true);
  };

  const stopTracking = () => {
    if (watchId) {
      Geolocation.clearWatch(watchId);
      setWatchId(null);
      setIsTracking(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Geolocalización</Text>
      
      {location && (
        <View style={styles.locationContainer}>
          <Text style={styles.locationTitle}>Ubicación Actual</Text>
          <Text style={styles.locationData}>
            Latitud: {location.latitude.toFixed(6)}
          </Text>
          <Text style={styles.locationData}>
            Longitud: {location.longitude.toFixed(6)}
          </Text>
          <Text style={styles.locationData}>
            Precisión: {location.accuracy.toFixed(2)}m
          </Text>
          <Text style={styles.locationData}>
            Timestamp: {new Date(location.timestamp).toLocaleString()}
          </Text>
        </View>
      )}

      <View style={styles.buttonContainer}>
        <TouchableOpacity style={styles.button} onPress={getCurrentLocation}>
          <Text style={styles.buttonText}>Obtener Ubicación</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.button, isTracking ? styles.stopButton : styles.startButton]}
          onPress={isTracking ? stopTracking : startTracking}
        >
          <Text style={styles.buttonText}>
            {isTracking ? 'Detener Seguimiento' : 'Iniciar Seguimiento'}
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30
  },
  locationContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  locationTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333'
  },
  locationData: {
    fontSize: 16,
    marginBottom: 5,
    color: '#666'
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around'
  },
  button: {
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    flex: 0.45
  },
  startButton: {
    backgroundColor: '#4CAF50'
  },
  stopButton: {
    backgroundColor: '#f44336'
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold'
  }
});

export default LocationManager;
```

### 4. Integración con Hardware

#### Cámara y Flash
```jsx
// CameraManager.js
import React, { useState, useRef } from 'react';
import { View, StyleSheet, TouchableOpacity, Text, Alert } from 'react-native';
import { RNCamera } from 'react-native-camera';
import { Vibration } from 'react-native';

const CameraManager = () => {
  const [flashMode, setFlashMode] = useState(RNCamera.Constants.FlashMode.off);
  const [cameraType, setCameraType] = useState(RNCamera.Constants.Type.back);
  const cameraRef = useRef(null);

  const takePicture = async () => {
    if (cameraRef.current) {
      try {
        const options = { quality: 0.5, base64: true };
        const data = await cameraRef.current.takePictureAsync(options);
        Alert.alert('Foto tomada', 'La foto se ha guardado correctamente');
        Vibration.vibrate(100);
      } catch (error) {
        Alert.alert('Error', 'No se pudo tomar la foto');
        console.log(error);
      }
    }
  };

  const toggleFlash = () => {
    setFlashMode(
      flashMode === RNCamera.Constants.FlashMode.off
        ? RNCamera.Constants.FlashMode.on
        : RNCamera.Constants.FlashMode.off
    );
  };

  const toggleCamera = () => {
    setCameraType(
      cameraType === RNCamera.Constants.Type.back
        ? RNCamera.Constants.Type.front
        : RNCamera.Constants.Type.back
    );
  };

  return (
    <View style={styles.container}>
      <RNCamera
        ref={cameraRef}
        style={styles.camera}
        type={cameraType}
        flashMode={flashMode}
        androidCameraPermissionOptions={{
          title: 'Permiso para usar la cámara',
          message: 'Necesitamos tu permiso para usar la cámara',
          buttonPositive: 'Ok',
          buttonNegative: 'Cancelar'
        }}
      >
        <View style={styles.controls}>
          <TouchableOpacity style={styles.controlButton} onPress={toggleFlash}>
            <Text style={styles.controlButtonText}>
              Flash: {flashMode === RNCamera.Constants.FlashMode.off ? 'Off' : 'On'}
            </Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={styles.captureButton} onPress={takePicture}>
            <Text style={styles.captureButtonText}>Capturar</Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={styles.controlButton} onPress={toggleCamera}>
            <Text style={styles.controlButtonText}>Cambiar</Text>
          </TouchableOpacity>
        </View>
      </RNCamera>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  camera: {
    flex: 1
  },
  controls: {
    position: 'absolute',
    bottom: 50,
    left: 0,
    right: 0,
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center'
  },
  controlButton: {
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 15,
    borderRadius: 10
  },
  controlButtonText: {
    color: 'white',
    fontWeight: 'bold'
  },
  captureButton: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 50,
    borderWidth: 3,
    borderColor: '#333'
  },
  captureButtonText: {
    color: '#333',
    fontWeight: 'bold',
    fontSize: 16
  }
});

export default CameraManager;
```

#### Vibración y Sonido
```jsx
// HapticFeedback.js
import React, { useState } from 'react';
import { View, StyleSheet, TouchableOpacity, Text, Alert } from 'react-native';
import { Vibration } from 'react-native';
import Sound from 'react-native-sound';

const HapticFeedback = () => {
  const [sound, setSound] = useState(null);

  const playSound = () => {
    const newSound = new Sound('notification.mp3', Sound.MAIN_BUNDLE, (error) => {
      if (error) {
        Alert.alert('Error', 'No se pudo cargar el sonido');
        console.log(error);
      } else {
        newSound.play((success) => {
          if (success) {
            console.log('Sonido reproducido correctamente');
          } else {
            console.log('Error al reproducir el sonido');
          }
        });
      }
    });
    setSound(newSound);
  };

  const vibrate = (pattern) => {
    Vibration.vibrate(pattern);
  };

  const stopSound = () => {
    if (sound) {
      sound.stop();
      sound.release();
      setSound(null);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Feedback Háptico y Sonido</Text>
      
      <View style={styles.buttonContainer}>
        <TouchableOpacity
          style={styles.button}
          onPress={() => vibrate(100)}
        >
          <Text style={styles.buttonText}>Vibración Corta</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.button}
          onPress={() => vibrate([0, 100, 100, 100])}
        >
          <Text style={styles.buttonText}>Vibración Patrón</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.button}
          onPress={playSound}
        >
          <Text style={styles.buttonText}>Reproducir Sonido</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.button}
          onPress={stopSound}
        >
          <Text style={styles.buttonText}>Detener Sonido</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30
  },
  buttonContainer: {
    flex: 1,
    justifyContent: 'center'
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    alignItems: 'center'
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  }
});

export default HapticFeedback;
```

### 5. Optimización de Recursos

#### Gestión de Memoria
```jsx
// ResourceManager.js
import React, { useState, useEffect, useCallback } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import DeviceInfo from 'react-native-device-info';

const ResourceManager = () => {
  const [deviceInfo, setDeviceInfo] = useState({});
  const [memoryUsage, setMemoryUsage] = useState(0);

  useEffect(() => {
    getDeviceInfo();
    startMemoryMonitoring();
  }, []);

  const getDeviceInfo = async () => {
    try {
      const info = {
        brand: await DeviceInfo.getBrand(),
        model: await DeviceInfo.getModel(),
        systemVersion: await DeviceInfo.getSystemVersion(),
        totalMemory: await DeviceInfo.getTotalMemory(),
        usedMemory: await DeviceInfo.getUsedMemory()
      };
      setDeviceInfo(info);
    } catch (error) {
      console.log('Error getting device info:', error);
    }
  };

  const startMemoryMonitoring = () => {
    const interval = setInterval(async () => {
      try {
        const used = await DeviceInfo.getUsedMemory();
        setMemoryUsage(used);
      } catch (error) {
        console.log('Error monitoring memory:', error);
      }
    }, 1000);

    return () => clearInterval(interval);
  };

  const optimizeMemory = useCallback(() => {
    // Limpiar caché y liberar memoria
    if (global.gc) {
      global.gc();
    }
    Alert.alert('Optimización', 'Memoria optimizada');
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Información del Dispositivo</Text>
      
      <View style={styles.infoContainer}>
        <Text style={styles.infoTitle}>Información del Dispositivo</Text>
        <Text style={styles.infoText}>Marca: {deviceInfo.brand}</Text>
        <Text style={styles.infoText}>Modelo: {deviceInfo.model}</Text>
        <Text style={styles.infoText}>Versión del Sistema: {deviceInfo.systemVersion}</Text>
        <Text style={styles.infoText}>Memoria Total: {deviceInfo.totalMemory} MB</Text>
        <Text style={styles.infoText}>Memoria Usada: {memoryUsage} MB</Text>
      </View>

      <TouchableOpacity style={styles.button} onPress={optimizeMemory}>
        <Text style={styles.buttonText}>Optimizar Memoria</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30
  },
  infoContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  infoTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333'
  },
  infoText: {
    fontSize: 16,
    marginBottom: 5,
    color: '#666'
  },
  button: {
    backgroundColor: '#4CAF50',
    padding: 20,
    borderRadius: 10,
    alignItems: 'center'
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  }
});

export default ResourceManager;
```

### 6. Casos de Uso Avanzados

#### Aplicación de Fitness con Sensores
```jsx
// FitnessTracker.js
import React, { useState, useEffect } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import { accelerometer, setUpdateIntervalForType, SensorTypes } from 'react-native-sensors';

const FitnessTracker = () => {
  const [steps, setSteps] = useState(0);
  const [isTracking, setIsTracking] = useState(false);
  const [lastAcceleration, setLastAcceleration] = useState({ x: 0, y: 0, z: 0 });

  useEffect(() => {
    setUpdateIntervalForType(SensorTypes.accelerometer, 100);
  }, []);

  const startTracking = () => {
    setIsTracking(true);
    
    const subscription = accelerometer.subscribe(({ x, y, z }) => {
      const acceleration = { x, y, z };
      const magnitude = Math.sqrt(x * x + y * y + z * z);
      
      // Detectar pasos basado en cambios de aceleración
      if (magnitude > 10 && Math.abs(acceleration.y - lastAcceleration.y) > 2) {
        setSteps(prev => prev + 1);
      }
      
      setLastAcceleration(acceleration);
    });

    return () => subscription.unsubscribe();
  };

  const stopTracking = () => {
    setIsTracking(false);
  };

  const resetSteps = () => {
    setSteps(0);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Contador de Pasos</Text>
      
      <View style={styles.stepsContainer}>
        <Text style={styles.stepsText}>{steps}</Text>
        <Text style={styles.stepsLabel}>Pasos</Text>
      </View>

      <View style={styles.buttonContainer}>
        <TouchableOpacity
          style={[styles.button, isTracking ? styles.stopButton : styles.startButton]}
          onPress={isTracking ? stopTracking : startTracking}
        >
          <Text style={styles.buttonText}>
            {isTracking ? 'Detener' : 'Iniciar'}
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={resetSteps}>
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30
  },
  stepsContainer: {
    alignItems: 'center',
    marginBottom: 50
  },
  stepsText: {
    fontSize: 72,
    fontWeight: 'bold',
    color: '#4CAF50'
  },
  stepsLabel: {
    fontSize: 18,
    color: '#666'
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around'
  },
  button: {
    padding: 20,
    borderRadius: 10,
    alignItems: 'center',
    flex: 0.45
  },
  startButton: {
    backgroundColor: '#4CAF50'
  },
  stopButton: {
    backgroundColor: '#f44336'
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  }
});

export default FitnessTracker;
```

## Recursos Adicionales

### Documentación
- [React Native Sensors](https://github.com/react-native-sensors/react-native-sensors)
- [React Native Device Info](https://github.com/react-native-device-info/react-native-device-info)
- [React Native Camera](https://github.com/react-native-camera/react-native-camera)

### Herramientas
- [Flipper](https://fbflipper.com/) - Para debugging de sensores
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)

### Tutoriales
- [Working with Sensors in React Native](https://blog.logrocket.com/working-sensors-react-native/)
- [React Native Camera Tutorial](https://reactnative.dev/docs/camera)

## Ejercicios Prácticos

### Ejercicio 1: Contador de Pasos
Crear una aplicación que cuente pasos usando el acelerómetro.

### Ejercicio 2: Brújula Digital
Implementar una brújula usando el magnetómetro.

### Ejercicio 3: Aplicación de Fitness
Crear una aplicación completa de fitness con múltiples sensores.

## Evaluación

### Criterios de Evaluación
- **Funcionalidad (40%):** Los sensores funcionan correctamente
- **Rendimiento (30%):** Optimización del uso de recursos
- **Código (20%):** Estructura y calidad del código
- **Innovación (10%):** Uso creativo de los sensores

### Entregables
- Código fuente de la aplicación
- Demo de la aplicación funcionando
- Documentación de optimizaciones
- Reflexión sobre el proceso de desarrollo

---

**Duración estimada:** 2.5 horas  
**Dificultad:** Intermedia  
**Prerrequisitos:** Conocimientos sólidos de React Native, experiencia con APIs nativas
