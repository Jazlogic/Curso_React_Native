# Clase 5: Sensores y Hardware Avanzado 📱

## Objetivos de la Clase
- Implementar acceso a sensores del dispositivo (acelerómetro, giroscopio, brújula)
- Controlar funciones de hardware (vibración, linterna)
- Crear aplicaciones que respondan al movimiento del dispositivo
- Implementar detección de gestos y orientación
- Crear un sistema de monitoreo de sensores en tiempo real

## Duración Estimada
**1.5 horas**

## Contenido Teórico

### 1. Acelerómetro y Giroscopio

Para acceder a los sensores de movimiento, usamos `react-native-sensors`:

```javascript
import { accelerometer, gyroscope, magnetometer } from 'react-native-sensors';
import { Platform, PermissionsAndroid } from 'react-native';

class SensorManager {
  // Configuración por defecto para sensores
  static sensorConfig = {
    updateInterval: 100, // Actualización cada 100ms
    referenceFrame: 'device', // Frame de referencia del dispositivo
  };

  // Verificar permisos de sensores (Android)
  static async checkSensorPermissions() {
    if (Platform.OS === 'android') {
      try {
        const granted = await PermissionsAndroid.request(
          PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
          {
            title: 'Permiso de Sensores',
            message: 'La app necesita acceso a sensores para funcionalidades avanzadas',
            buttonNeutral: 'Preguntar más tarde',
            buttonNegative: 'Cancelar',
            buttonPositive: 'OK',
          }
        );
        return granted === PermissionsAndroid.RESULTS.GRANTED;
      } catch (err) {
        console.warn('Error al solicitar permiso de sensores:', err);
        return false;
      }
    }
    return true; // iOS no requiere permiso explícito para sensores
  }

  // Iniciar monitoreo del acelerómetro
  static startAccelerometer(onData, onError) {
    const subscription = accelerometer.subscribe(
      ({ x, y, z, timestamp }) => {
        const acceleration = {
          x: Math.round(x * 100) / 100, // Redondear a 2 decimales
          y: Math.round(y * 100) / 100,
          z: Math.round(z * 100) / 100,
          magnitude: Math.sqrt(x * x + y * y + z * z),
          timestamp,
        };
        onData(acceleration);
      },
      onError
    );
    
    return subscription;
  }

  // Iniciar monitoreo del giroscopio
  static startGyroscope(onData, onError) {
    const subscription = gyroscope.subscribe(
      ({ x, y, z, timestamp }) => {
        const rotation = {
          x: Math.round(x * 100) / 100,
          y: Math.round(y * 100) / 100,
          z: Math.round(z * 100) / 100,
          magnitude: Math.sqrt(x * x + y * y + z * z),
          timestamp,
        };
        onData(rotation);
      },
      onError
    );
    
    return subscription;
  }

  // Iniciar monitoreo del magnetómetro (brújula)
  static startMagnetometer(onData, onError) {
    const subscription = magnetometer.subscribe(
      ({ x, y, z, timestamp }) => {
        const magneticField = {
          x: Math.round(x * 100) / 100,
          y: Math.round(y * 100) / 100,
          z: Math.round(z * 100) / 100,
          magnitude: Math.sqrt(x * x + y * y + z * z),
          heading: this.calculateHeading(x, y),
          timestamp,
        };
        onData(magneticField);
      },
      onError
    );
    
    return subscription;
  }

  // Calcular dirección (heading) desde campos magnéticos
  static calculateHeading(mx, my) {
    let heading = Math.atan2(my, mx) * (180 / Math.PI);
    
    // Ajustar para que 0° sea el Norte
    heading = (heading + 360) % 360;
    
    return Math.round(heading);
  }

  // Detectar gestos básicos
  static detectGesture(acceleration, threshold = 2.0) {
    const { x, y, z, magnitude } = acceleration;
    
    if (magnitude > threshold) {
      if (Math.abs(x) > Math.abs(y) && Math.abs(x) > Math.abs(z)) {
        return x > 0 ? 'shake_right' : 'shake_left';
      } else if (Math.abs(y) > Math.abs(x) && Math.abs(y) > Math.abs(z)) {
        return y > 0 ? 'shake_up' : 'shake_down';
      } else {
        return 'shake_vertical';
      }
    }
    
    return null;
  }

  // Detectar orientación del dispositivo
  static detectOrientation(acceleration) {
    const { x, y, z } = acceleration;
    
    // Calcular ángulos de orientación
    const pitch = Math.atan2(-x, Math.sqrt(y * y + z * z)) * (180 / Math.PI);
    const roll = Math.atan2(y, z) * (180 / Math.PI);
    
    // Determinar orientación principal
    if (Math.abs(pitch) < 45 && Math.abs(roll) < 45) {
      return 'flat';
    } else if (Math.abs(pitch) > 45) {
      return pitch > 0 ? 'tilted_up' : 'tilted_down';
    } else {
      return roll > 0 ? 'tilted_right' : 'tilted_left';
    }
  }

  // Detener monitoreo de sensores
  static stopSensor(subscription) {
    if (subscription) {
      subscription.unsubscribe();
    }
  }
}
```

### 2. Control de Vibración y Linterna

```javascript
import { Vibration, Platform } from 'react-native';
import Torch from 'react-native-torch';

class HardwareController {
  // Patrones de vibración predefinidos
  static vibrationPatterns = {
    short: 100, // Vibración corta
    medium: 300, // Vibración media
    long: 500, // Vibración larga
    double: [0, 100, 200, 100], // Doble vibración
    morse: [0, 100, 200, 100, 400, 300, 600, 100, 800, 300], // Patrón morse
    heartbeat: [0, 100, 100, 100, 100, 100, 200, 100, 200, 100], // Latido
  };

  // Activar vibración
  static vibrate(pattern = 'short') {
    if (typeof pattern === 'string') {
      const duration = this.vibrationPatterns[pattern];
      if (duration) {
        Vibration.vibrate(duration);
      }
    } else if (Array.isArray(pattern)) {
      Vibration.vibrate(pattern);
    } else {
      Vibration.vibrate(pattern);
    }
  }

  // Detener vibración
  static stopVibration() {
    Vibration.cancel();
  }

  // Vibración personalizada
  static customVibration(duration, intensity = 1.0) {
    // En Android, podemos usar diferentes intensidades
    if (Platform.OS === 'android') {
      // Simular intensidad con múltiples vibraciones
      const pattern = [];
      for (let i = 0; i < intensity * 3; i++) {
        pattern.push(0, duration / (intensity * 3));
      }
      Vibration.vibrate(pattern);
    } else {
      Vibration.vibrate(duration);
    }
  }

  // Controlar linterna
  static async toggleTorch() {
    try {
      const isEnabled = await Torch.switchState();
      return isEnabled;
    } catch (error) {
      console.error('Error al controlar linterna:', error);
      return false;
    }
  }

  // Activar linterna
  static async turnOnTorch() {
    try {
      await Torch.switchState(true);
      return true;
    } catch (error) {
      console.error('Error al encender linterna:', error);
      return false;
    }
  }

  // Desactivar linterna
  static async turnOffTorch() {
    try {
      await Torch.switchState(false);
      return true;
    } catch (error) {
      console.error('Error al apagar linterna:', error);
      return false;
    }
  }

  // Verificar si la linterna está disponible
  static async isTorchAvailable() {
    try {
      return await Torch.isSupported();
    } catch (error) {
      console.error('Error al verificar linterna:', error);
      return false;
    }
  }

  // Controlar intensidad de linterna (si está soportado)
  static async setTorchIntensity(intensity) {
    try {
      if (Platform.OS === 'android') {
        // En Android, podemos usar diferentes niveles de intensidad
        await Torch.setTorchMode(intensity);
        return true;
      }
      return false; // iOS no soporta control de intensidad
    } catch (error) {
      console.error('Error al establecer intensidad de linterna:', error);
      return false;
    }
  }
}
```

### 3. Componente de Monitoreo de Sensores

```javascript
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  Alert,
} from 'react-native';
import { SensorManager } from './SensorManager';
import { HardwareController } from './HardwareController';

const SensorMonitor = () => {
  const [accelerometerData, setAccelerometerData] = useState(null);
  const [gyroscopeData, setGyroscopeData] = useState(null);
  const [magnetometerData, setMagnetometerData] = useState(null);
  const [isMonitoring, setIsMonitoring] = useState(false);
  const [torchEnabled, setTorchEnabled] = useState(false);
  const [detectedGesture, setDetectedGesture] = useState(null);
  const [deviceOrientation, setDeviceOrientation] = useState(null);
  
  const sensorSubscriptions = useRef({});

  useEffect(() => {
    checkPermissions();
    return () => {
      stopAllSensors();
    };
  }, []);

  // Verificar permisos
  const checkPermissions = async () => {
    const hasPermission = await SensorManager.checkSensorPermissions();
    if (!hasPermission) {
      Alert.alert('Permiso Denegado', 'No se puede acceder a los sensores');
    }
  };

  // Iniciar monitoreo de todos los sensores
  const startMonitoring = () => {
    if (isMonitoring) return;

    try {
      // Acelerómetro
      sensorSubscriptions.current.accelerometer = SensorManager.startAccelerometer(
        (data) => {
          setAccelerometerData(data);
          
          // Detectar gestos
          const gesture = SensorManager.detectGesture(data);
          if (gesture) {
            setDetectedGesture(gesture);
            // Vibración de feedback
            HardwareController.vibrate('short');
          }
          
          // Detectar orientación
          const orientation = SensorManager.detectOrientation(data);
          setDeviceOrientation(orientation);
        },
        (error) => console.error('Error en acelerómetro:', error)
      );

      // Giroscopio
      sensorSubscriptions.current.gyroscope = SensorManager.startGyroscope(
        (data) => setGyroscopeData(data),
        (error) => console.error('Error en giroscopio:', error)
      );

      // Magnetómetro
      sensorSubscriptions.current.magnetometer = SensorManager.startMagnetometer(
        (data) => setMagnetometerData(data),
        (error) => console.error('Error en magnetómetro:', error)
      );

      setIsMonitoring(true);
      Alert.alert('Éxito', 'Monitoreo de sensores iniciado');
    } catch (error) {
      console.error('Error al iniciar monitoreo:', error);
      Alert.alert('Error', 'No se pudo iniciar el monitoreo');
    }
  };

  // Detener monitoreo de sensores
  const stopMonitoring = () => {
    if (!isMonitoring) return;

    stopAllSensors();
    setIsMonitoring(false);
    setAccelerometerData(null);
    setGyroscopeData(null);
    setMagnetometerData(null);
    setDetectedGesture(null);
    setDeviceOrientation(null);
    
    Alert.alert('Éxito', 'Monitoreo de sensores detenido');
  };

  // Detener todos los sensores
  const stopAllSensors = () => {
    Object.values(sensorSubscriptions.current).forEach(subscription => {
      SensorManager.stopSensor(subscription);
    });
    sensorSubscriptions.current = {};
  };

  // Cambiar estado de linterna
  const toggleTorch = async () => {
    try {
      const isAvailable = await HardwareController.isTorchAvailable();
      if (!isAvailable) {
        Alert.alert('No Soportado', 'La linterna no está disponible en este dispositivo');
        return;
      }

      const newState = await HardwareController.toggleTorch();
      setTorchEnabled(newState);
    } catch (error) {
      console.error('Error al cambiar linterna:', error);
      Alert.alert('Error', 'No se pudo controlar la linterna');
    }
  };

  // Probar diferentes patrones de vibración
  const testVibration = (pattern) => {
    HardwareController.vibrate(pattern);
  };

  // Renderizar datos del sensor
  const renderSensorData = (title, data, unit = '') => {
    if (!data) return null;

    return (
      <View style={styles.sensorCard}>
        <Text style={styles.sensorTitle}>{title}</Text>
        <View style={styles.sensorValues}>
          <Text style={styles.sensorValue}>X: {data.x}{unit}</Text>
          <Text style={styles.sensorValue}>Y: {data.y}{unit}</Text>
          <Text style={styles.sensorValue}>Z: {data.z}{unit}</Text>
          {data.magnitude && (
            <Text style={styles.sensorValue}>Magnitud: {data.magnitude}{unit}</Text>
          )}
          {data.heading && (
            <Text style={styles.sensorValue}>Dirección: {data.heading}°</Text>
          )}
        </View>
      </View>
    );
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Monitor de Sensores</Text>
      
      {/* Controles principales */}
      <View style={styles.controls}>
        <TouchableOpacity
          style={[
            styles.controlButton,
            isMonitoring ? styles.stopButton : styles.startButton,
          ]}
          onPress={isMonitoring ? stopMonitoring : startMonitoring}
        >
          <Text style={styles.controlButtonText}>
            {isMonitoring ? '🛑 Detener' : '▶️ Iniciar'} Monitoreo
          </Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[
            styles.torchButton,
            torchEnabled && styles.torchButtonActive,
          ]}
          onPress={toggleTorch}
        >
          <Text style={styles.torchButtonText}>
            {torchEnabled ? '💡' : '🔦'} Linterna
          </Text>
        </TouchableOpacity>
      </View>

      {/* Estado del sistema */}
      <View style={styles.statusSection}>
        <Text style={styles.statusTitle}>Estado del Sistema</Text>
        <View style={styles.statusRow}>
          <Text style={styles.statusLabel}>Monitoreo:</Text>
          <Text style={[
            styles.statusValue,
            isMonitoring ? styles.statusActive : styles.statusInactive,
          ]}>
            {isMonitoring ? 'Activo' : 'Inactivo'}
          </Text>
        </View>
        <View style={styles.statusRow}>
          <Text style={styles.statusLabel}>Linterna:</Text>
          <Text style={[
            styles.statusValue,
            torchEnabled ? styles.statusActive : styles.statusInactive,
          ]}>
            {torchEnabled ? 'Encendida' : 'Apagada'}
          </Text>
        </View>
        {detectedGesture && (
          <View style={styles.statusRow}>
            <Text style={styles.statusLabel}>Gesto Detectado:</Text>
            <Text style={styles.statusValue}>{detectedGesture}</Text>
          </View>
        )}
        {deviceOrientation && (
          <View style={styles.statusRow}>
            <Text style={styles.statusLabel}>Orientación:</Text>
            <Text style={styles.statusValue}>{deviceOrientation}</Text>
          </View>
        )}
      </View>

      {/* Datos de sensores */}
      {isMonitoring && (
        <View style={styles.sensorsSection}>
          <Text style={styles.sectionTitle}>Datos de Sensores</Text>
          
          {renderSensorData('Acelerómetro', accelerometerData, ' m/s²')}
          {renderSensorData('Giroscopio', gyroscopeData, ' rad/s')}
          {renderSensorData('Magnetómetro', magnetometerData, ' μT')}
        </View>
      )}

      {/* Pruebas de vibración */}
      <View style={styles.vibrationSection}>
        <Text style={styles.sectionTitle}>Pruebas de Vibración</Text>
        <View style={styles.vibrationButtons}>
          <TouchableOpacity
            style={styles.vibrationButton}
            onPress={() => testVibration('short')}
          >
            <Text style={styles.vibrationButtonText}>Corta</Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.vibrationButton}
            onPress={() => testVibration('medium')}
          >
            <Text style={styles.vibrationButtonText}>Media</Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.vibrationButton}
            onPress={() => testVibration('long')}
          >
            <Text style={styles.vibrationButtonText}>Larga</Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.vibrationButton}
            onPress={() => testVibration('double')}
          >
            <Text style={styles.vibrationButtonText}>Doble</Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.vibrationButton}
            onPress={() => testVibration('heartbeat')}
          >
            <Text style={styles.vibrationButtonText}>Latido</Text>
          </TouchableOpacity>
        </View>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginVertical: 20,
    color: '#333',
  },
  controls: {
    flexDirection: 'row',
    padding: 20,
    gap: 15,
  },
  controlButton: {
    flex: 1,
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  startButton: {
    backgroundColor: '#4CAF50',
  },
  stopButton: {
    backgroundColor: '#f44336',
  },
  controlButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  torchButton: {
    backgroundColor: '#FF9800',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    minWidth: 100,
  },
  torchButtonActive: {
    backgroundColor: '#FFC107',
  },
  torchButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  statusSection: {
    backgroundColor: 'white',
    margin: 20,
    padding: 20,
    borderRadius: 8,
  },
  statusTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  statusRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 10,
  },
  statusLabel: {
    fontSize: 16,
    color: '#666',
  },
  statusValue: {
    fontSize: 16,
    fontWeight: '600',
  },
  statusActive: {
    color: '#4CAF50',
  },
  statusInactive: {
    color: '#f44336',
  },
  sensorsSection: {
    margin: 20,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  sensorCard: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 15,
  },
  sensorTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  sensorValues: {
    gap: 5,
  },
  sensorValue: {
    fontSize: 14,
    color: '#666',
  },
  vibrationSection: {
    backgroundColor: 'white',
    margin: 20,
    padding: 20,
    borderRadius: 8,
  },
  vibrationButtons: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    gap: 10,
  },
  vibrationButton: {
    backgroundColor: '#2196F3',
    padding: 10,
    borderRadius: 6,
    minWidth: 80,
    alignItems: 'center',
  },
  vibrationButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: '600',
  },
});

export default SensorMonitor;
```

### 4. Aplicaciones Prácticas de Sensores

```javascript
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, Alert } from 'react-native';
import { SensorManager } from './SensorManager';
import { HardwareController } from './HardwareController';

// Aplicación de nivel de burbuja
const BubbleLevel = () => {
  const [levelData, setLevelData] = useState({ x: 0, y: 0 });
  const [isActive, setIsActive] = useState(false);
  const subscription = useRef(null);

  useEffect(() => {
    return () => {
      if (subscription.current) {
        SensorManager.stopSensor(subscription.current);
      }
    };
  }, []);

  const startLevel = () => {
    subscription.current = SensorManager.startAccelerometer(
      (data) => {
        const level = {
          x: Math.round(data.x * 100) / 100,
          y: Math.round(data.y * 100) / 100,
        };
        setLevelData(level);
        
        // Vibración cuando está nivelado
        if (Math.abs(level.x) < 0.1 && Math.abs(level.y) < 0.1) {
          HardwareController.vibrate('short');
        }
      },
      (error) => console.error('Error en nivel:', error)
    );
    setIsActive(true);
  };

  const stopLevel = () => {
    if (subscription.current) {
      SensorManager.stopSensor(subscription.current);
      subscription.current = null;
    }
    setIsActive(false);
  };

  const getLevelColor = (value) => {
    const absValue = Math.abs(value);
    if (absValue < 0.1) return '#4CAF50'; // Verde (nivelado)
    if (absValue < 0.3) return '#FF9800'; // Naranja (casi nivelado)
    return '#f44336'; // Rojo (no nivelado)
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Nivel de Burbuja</Text>
      
      <View style={styles.levelDisplay}>
        <View style={styles.levelAxis}>
          <Text style={styles.axisLabel}>X: {levelData.x}°</Text>
          <View style={styles.levelBar}>
            <View
              style={[
                styles.levelIndicator,
                {
                  left: `${50 + (levelData.x * 50)}%`,
                  backgroundColor: getLevelColor(levelData.x),
                },
              ]}
            />
          </View>
        </View>
        
        <View style={styles.levelAxis}>
          <Text style={styles.axisLabel}>Y: {levelData.y}°</Text>
          <View style={styles.levelBar}>
            <View
              style={[
                styles.levelIndicator,
                {
                  left: `${50 + (levelData.y * 50)}%`,
                  backgroundColor: getLevelColor(levelData.y),
                },
              ]}
            />
          </View>
        </View>
      </View>
      
      <TouchableOpacity
        style={[
          styles.controlButton,
          isActive ? styles.stopButton : styles.startButton,
        ]}
        onPress={isActive ? stopLevel : startLevel}
      >
        <Text style={styles.controlButtonText}>
          {isActive ? '🛑 Detener' : '▶️ Iniciar'} Nivel
        </Text>
      </TouchableOpacity>
    </View>
  );
};

// Aplicación de brújula
const Compass = () => {
  const [heading, setHeading] = useState(0);
  const [isActive, setIsActive] = useState(false);
  const subscription = useRef(null);

  useEffect(() => {
    return () => {
      if (subscription.current) {
        SensorManager.stopSensor(subscription.current);
      }
    };
  }, []);

  const startCompass = () => {
    subscription.current = SensorManager.startMagnetometer(
      (data) => {
        setHeading(data.heading);
      },
      (error) => console.error('Error en brújula:', error)
    );
    setIsActive(true);
  };

  const stopCompass = () => {
    if (subscription.current) {
      SensorManager.stopSensor(subscription.current);
      subscription.current = null;
    }
    setIsActive(false);
  };

  const getDirection = (degrees) => {
    const directions = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
    const index = Math.round(degrees / 45) % 8;
    return directions[index];
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Brújula</Text>
      
      <View style={styles.compassDisplay}>
        <Text style={styles.headingText}>{heading}°</Text>
        <Text style={styles.directionText}>{getDirection(heading)}</Text>
        
        <View style={styles.compassNeedle}>
          <View
            style={[
              styles.needle,
              { transform: [{ rotate: `${heading}deg` }] },
            ]}
          />
        </View>
      </View>
      
      <TouchableOpacity
        style={[
          styles.controlButton,
          isActive ? styles.stopButton : styles.startButton,
        ]}
        onPress={isActive ? stopCompass : startCompass}
      >
        <Text style={styles.controlButtonText}>
          {isActive ? '🛑 Detener' : '▶️ Iniciar'} Brújula
        </Text>
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
    marginBottom: 30,
    color: '#333',
  },
  levelDisplay: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 8,
    marginBottom: 30,
  },
  levelAxis: {
    marginBottom: 20,
  },
  axisLabel: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 10,
    textAlign: 'center',
  },
  levelBar: {
    height: 20,
    backgroundColor: '#e0e0e0',
    borderRadius: 10,
    position: 'relative',
  },
  levelIndicator: {
    position: 'absolute',
    top: 0,
    width: 4,
    height: 20,
    borderRadius: 2,
  },
  controlButton: {
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  startButton: {
    backgroundColor: '#4CAF50',
  },
  stopButton: {
    backgroundColor: '#f44336',
  },
  controlButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  compassDisplay: {
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 40,
    borderRadius: 8,
    marginBottom: 30,
  },
  headingText: {
    fontSize: 48,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  directionText: {
    fontSize: 24,
    fontWeight: '600',
    color: '#666',
    marginBottom: 30,
  },
  compassNeedle: {
    width: 200,
    height: 200,
    borderRadius: 100,
    borderWidth: 3,
    borderColor: '#333',
    justifyContent: 'center',
    alignItems: 'center',
    position: 'relative',
  },
  needle: {
    width: 4,
    height: 80,
    backgroundColor: '#f44336',
    borderRadius: 2,
  },
});

export { BubbleLevel, Compass };
```

## Ejercicios Prácticos

### Ejercicio 1: Aplicación de Nivel de Burbuja
Crea una aplicación que use el acelerómetro para funcionar como un nivel de burbuja digital.

### Ejercicio 2: Brújula Digital
Implementa una brújula que use el magnetómetro para mostrar la dirección norte.

### Ejercicio 3: Control por Gestos
Crea una aplicación que responda a gestos del dispositivo (agitar, inclinar, etc.).

## Resumen de la Clase

Hemos aprendido:
1. **Sensores de Movimiento**: Acelerómetro, giroscopio y magnetómetro
2. **Control de Hardware**: Vibración y linterna
3. **Detección de Gestos**: Identificación de movimientos del dispositivo
4. **Aplicaciones Prácticas**: Nivel de burbuja y brújula digital
5. **Monitoreo en Tiempo Real**: Sistema completo de sensores

## Navegación
- **Anterior**: [Clase 4: Notificaciones Push y Locales](clase_4_notificaciones_push_locales.md)
- **Siguiente**: [Módulo 12: Seguridad y Autenticación](../senior_2/README.md)
- **Inicio**: [Índice del Módulo](../senior_2/README.md)

## Próximo Módulo
En el siguiente módulo aprenderemos sobre **Seguridad y Autenticación**, incluyendo JWT, OAuth, encriptación y mejores prácticas de seguridad.
