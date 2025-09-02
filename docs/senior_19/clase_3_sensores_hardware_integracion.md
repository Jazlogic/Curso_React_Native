# Clase 3: Integración de Sensores y Hardware

## 🎯 Objetivos de la Clase

- Comprender los sensores disponibles en dispositivos móviles
- Integrar acelerómetro, giroscopio y magnetómetro
- Implementar controles basados en sensores
- Utilizar cámara y micrófono para interacciones
- Crear experiencias inmersivas con hardware

## 📋 Contenido

### 1. Introducción a los Sensores Móviles

#### Tipos de Sensores Disponibles
- **Acelerómetro**: Detecta aceleración y orientación
- **Giroscopio**: Mide rotación angular
- **Magnetómetro**: Detecta campos magnéticos (brújula)
- **Sensor de Proximidad**: Detecta objetos cercanos
- **Sensor de Luz**: Mide iluminación ambiental
- **Barómetro**: Mide presión atmosférica
- **Humedad**: Detecta humedad ambiental

#### Permisos y Configuración
```javascript
import { PermissionsAndroid, Platform } from 'react-native';

const requestSensorPermissions = async () => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.requestMultiple([
        PermissionsAndroid.PERMISSIONS.CAMERA,
        PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
        PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION
      ]);
      
      return Object.values(granted).every(
        permission => permission === PermissionsAndroid.RESULTS.GRANTED
      );
    } catch (err) {
      console.warn(err);
      return false;
    }
  }
  return true;
};
```

### 2. Acelerómetro y Giroscopio

#### Configuración del Acelerómetro
```javascript
import { Accelerometer } from 'react-native-sensors';

const AccelerometerComponent = () => {
  const [acceleration, setAcceleration] = useState({ x: 0, y: 0, z: 0 });
  
  useEffect(() => {
    const subscription = Accelerometer.subscribe(({ x, y, z }) => {
      setAcceleration({ x, y, z });
    });
    
    return () => subscription.unsubscribe();
  }, []);
  
  return (
    <View>
      <Text>X: {acceleration.x.toFixed(2)}</Text>
      <Text>Y: {acceleration.y.toFixed(2)}</Text>
      <Text>Z: {acceleration.z.toFixed(2)}</Text>
    </View>
  );
};
```

#### Configuración del Giroscopio
```javascript
import { Gyroscope } from 'react-native-sensors';

const GyroscopeComponent = () => {
  const [rotation, setRotation] = useState({ x: 0, y: 0, z: 0 });
  
  useEffect(() => {
    const subscription = Gyroscope.subscribe(({ x, y, z }) => {
      setRotation({ x, y, z });
    });
    
    return () => subscription.unsubscribe();
  }, []);
  
  return (
    <View>
      <Text>Rotación X: {rotation.x.toFixed(2)}</Text>
      <Text>Rotación Y: {rotation.y.toFixed(2)}</Text>
      <Text>Rotación Z: {rotation.z.toFixed(2)}</Text>
    </View>
  );
};
```

#### Controles de Movimiento
```javascript
const MotionControls = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const subscription = Accelerometer.subscribe(({ x, y }) => {
      // Convertir aceleración a movimiento
      const deltaX = x * 0.1;
      const deltaY = y * 0.1;
      
      setPosition(prev => ({
        x: Math.max(0, Math.min(300, prev.x + deltaX)),
        y: Math.max(0, Math.min(500, prev.y + deltaY))
      }));
    });
    
    return () => subscription.unsubscribe();
  }, []);
  
  return (
    <View style={styles.container}>
      <View
        style={[
          styles.ball,
          {
            left: position.x,
            top: position.y
          }
        ]}
      />
    </View>
  );
};
```

### 3. Magnetómetro y Brújula

#### Implementación de Brújula
```javascript
import { Magnetometer } from 'react-native-sensors';

const CompassComponent = () => {
  const [heading, setHeading] = useState(0);
  
  useEffect(() => {
    const subscription = Magnetometer.subscribe(({ x, y }) => {
      // Calcular dirección magnética
      const angle = Math.atan2(y, x) * (180 / Math.PI);
      const normalizedAngle = (angle + 360) % 360;
      setHeading(normalizedAngle);
    });
    
    return () => subscription.unsubscribe();
  }, []);
  
  return (
    <View style={styles.compass}>
      <View
        style={[
          styles.needle,
          {
            transform: [{ rotate: `${heading}deg` }]
          }
        ]}
      />
      <Text style={styles.headingText}>
        {heading.toFixed(0)}°
      </Text>
    </View>
  );
};
```

#### Navegación con Brújula
```javascript
const NavigationCompass = ({ targetHeading }) => {
  const [currentHeading, setCurrentHeading] = useState(0);
  
  useEffect(() => {
    const subscription = Magnetometer.subscribe(({ x, y }) => {
      const angle = Math.atan2(y, x) * (180 / Math.PI);
      setCurrentHeading((angle + 360) % 360);
    });
    
    return () => subscription.unsubscribe();
  }, []);
  
  const getDirectionToTarget = () => {
    const diff = targetHeading - currentHeading;
    return (diff + 360) % 360;
  };
  
  const direction = getDirectionToTarget();
  
  return (
    <View style={styles.navigation}>
      <View
        style={[
          styles.arrow,
          {
            transform: [{ rotate: `${direction}deg` }]
          }
        ]}
      />
      <Text>Dirección al objetivo: {direction.toFixed(0)}°</Text>
    </View>
  );
};
```

### 4. Sensor de Proximidad

#### Detección de Proximidad
```javascript
import { DeviceEventEmitter } from 'react-native';

const ProximitySensor = () => {
  const [isNear, setIsNear] = useState(false);
  
  useEffect(() => {
    const subscription = DeviceEventEmitter.addListener(
      'proximitySensorDidChange',
      (data) => {
        setIsNear(data.proximity);
      }
    );
    
    return () => subscription.remove();
  }, []);
  
  return (
    <View style={styles.proximityContainer}>
      <Text style={[
        styles.proximityText,
        { color: isNear ? 'red' : 'green' }
      ]}>
        {isNear ? 'Objeto cerca' : 'Objeto lejos'}
      </Text>
    </View>
  );
};
```

#### Control de Pantalla con Proximidad
```javascript
const ProximityScreenControl = () => {
  const [screenOn, setScreenOn] = useState(true);
  
  useEffect(() => {
    const subscription = DeviceEventEmitter.addListener(
      'proximitySensorDidChange',
      (data) => {
        if (data.proximity) {
          // Apagar pantalla cuando hay proximidad
          setScreenOn(false);
        } else {
          // Encender pantalla cuando no hay proximidad
          setScreenOn(true);
        }
      }
    );
    
    return () => subscription.remove();
  }, []);
  
  return (
    <View style={[
      styles.screen,
      { opacity: screenOn ? 1 : 0.3 }
    ]}>
      <Text>Contenido de la pantalla</Text>
    </View>
  );
};
```

### 5. Integración de Cámara

#### Configuración de Cámara
```javascript
import { RNCamera } from 'react-native-camera';

const CameraComponent = () => {
  const [cameraRef, setCameraRef] = useState(null);
  const [isRecording, setIsRecording] = useState(false);
  
  const takePicture = async () => {
    if (cameraRef) {
      const options = {
        quality: 0.8,
        base64: true,
        skipProcessing: true
      };
      
      try {
        const data = await cameraRef.takePictureAsync(options);
        console.log('Foto tomada:', data.uri);
      } catch (error) {
        console.error('Error al tomar foto:', error);
      }
    }
  };
  
  const startRecording = async () => {
    if (cameraRef) {
      try {
        const video = await cameraRef.recordAsync({
          quality: RNCamera.Constants.VideoQuality['720p'],
          maxDuration: 30
        });
        console.log('Video grabado:', video.uri);
        setIsRecording(false);
      } catch (error) {
        console.error('Error al grabar video:', error);
      }
    }
  };
  
  return (
    <RNCamera
      ref={setCameraRef}
      style={styles.camera}
      type={RNCamera.Constants.Type.back}
      flashMode={RNCamera.Constants.FlashMode.auto}
    >
      <View style={styles.cameraControls}>
        <TouchableOpacity onPress={takePicture}>
          <Text>Tomar Foto</Text>
        </TouchableOpacity>
        
        <TouchableOpacity onPress={startRecording}>
          <Text>{isRecording ? 'Grabando...' : 'Grabar Video'}</Text>
        </TouchableOpacity>
      </View>
    </RNCamera>
  );
};
```

#### Detección de Objetos con Cámara
```javascript
import { Camera } from 'react-native-vision-camera';

const ObjectDetection = () => {
  const [detectedObjects, setDetectedObjects] = useState([]);
  
  const onFrameProcessed = (frame) => {
    // Procesar frame para detección de objetos
    // Aquí integrarías con TensorFlow Lite o ML Kit
    const objects = processFrameForObjects(frame);
    setDetectedObjects(objects);
  };
  
  return (
    <View style={styles.container}>
      <Camera
        style={styles.camera}
        onFrameProcessed={onFrameProcessed}
      />
      
      {detectedObjects.map((obj, index) => (
        <View
          key={index}
          style={[
            styles.objectBox,
            {
              left: obj.x,
              top: obj.y,
              width: obj.width,
              height: obj.height
            }
          ]}
        >
          <Text style={styles.objectLabel}>{obj.label}</Text>
        </View>
      ))}
    </View>
  );
};
```

### 6. Integración de Audio

#### Grabación de Audio
```javascript
import AudioRecorderPlayer from 'react-native-audio-recorder-player';

const AudioRecorder = () => {
  const [isRecording, setIsRecording] = useState(false);
  const [audioPath, setAudioPath] = useState('');
  const audioRecorderPlayer = new AudioRecorderPlayer();
  
  const startRecording = async () => {
    try {
      const result = await audioRecorderPlayer.startRecorder();
      setAudioPath(result);
      setIsRecording(true);
    } catch (error) {
      console.error('Error al iniciar grabación:', error);
    }
  };
  
  const stopRecording = async () => {
    try {
      const result = await audioRecorderPlayer.stopRecorder();
      setIsRecording(false);
      console.log('Audio guardado en:', result);
    } catch (error) {
      console.error('Error al detener grabación:', error);
    }
  };
  
  return (
    <View style={styles.audioContainer}>
      <TouchableOpacity
        onPress={isRecording ? stopRecording : startRecording}
        style={[
          styles.recordButton,
          { backgroundColor: isRecording ? 'red' : 'green' }
        ]}
      >
        <Text style={styles.buttonText}>
          {isRecording ? 'Detener' : 'Grabar'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};
```

#### Reconocimiento de Voz
```javascript
import Voice from '@react-native-voice/voice';

const VoiceRecognition = () => {
  const [isListening, setIsListening] = useState(false);
  const [recognizedText, setRecognizedText] = useState('');
  
  useEffect(() => {
    Voice.onSpeechStart = () => setIsListening(true);
    Voice.onSpeechEnd = () => setIsListening(false);
    Voice.onSpeechResults = (e) => {
      setRecognizedText(e.value[0]);
    };
    
    return () => {
      Voice.destroy().then(Voice.removeAllListeners);
    };
  }, []);
  
  const startListening = async () => {
    try {
      await Voice.start('es-ES');
    } catch (error) {
      console.error('Error al iniciar reconocimiento:', error);
    }
  };
  
  const stopListening = async () => {
    try {
      await Voice.stop();
    } catch (error) {
      console.error('Error al detener reconocimiento:', error);
    }
  };
  
  return (
    <View style={styles.voiceContainer}>
      <TouchableOpacity
        onPress={isListening ? stopListening : startListening}
        style={styles.voiceButton}
      >
        <Text>{isListening ? 'Detener' : 'Escuchar'}</Text>
      </TouchableOpacity>
      
      <Text style={styles.recognizedText}>
        {recognizedText}
      </Text>
    </View>
  );
};
```

### 7. Vibración y Feedback Háptico

#### Configuración de Vibración
```javascript
import { Vibration } from 'react-native';

const VibrationComponent = () => {
  const vibrate = () => {
    Vibration.vibrate(1000); // Vibrar por 1 segundo
  };
  
  const vibratePattern = () => {
    Vibration.vibrate([0, 500, 200, 500]); // Patrón de vibración
  };
  
  return (
    <View style={styles.vibrationContainer}>
      <TouchableOpacity onPress={vibrate}>
        <Text>Vibración Simple</Text>
      </TouchableOpacity>
      
      <TouchableOpacity onPress={vibratePattern}>
        <Text>Patrón de Vibración</Text>
      </TouchableOpacity>
    </View>
  );
};
```

#### Feedback Háptico Avanzado
```javascript
import { HapticFeedback } from 'react-native-haptic-feedback';

const HapticFeedbackComponent = () => {
  const triggerHaptic = (type) => {
    HapticFeedback.trigger(type);
  };
  
  return (
    <View style={styles.hapticContainer}>
      <TouchableOpacity onPress={() => triggerHaptic('impactLight')}>
        <Text>Impacto Ligero</Text>
      </TouchableOpacity>
      
      <TouchableOpacity onPress={() => triggerHaptic('impactMedium')}>
        <Text>Impacto Medio</Text>
      </TouchableOpacity>
      
      <TouchableOpacity onPress={() => triggerHaptic('impactHeavy')}>
        <Text>Impacto Fuerte</Text>
      </TouchableOpacity>
      
      <TouchableOpacity onPress={() => triggerHaptic('notificationSuccess')}>
        <Text>Notificación de Éxito</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### 8. Proyecto Práctico: App de Sensores

#### Estructura del Proyecto
```
src/
├── components/
│   ├── SensorDisplay.js
│   ├── MotionController.js
│   ├── CompassView.js
│   └── CameraView.js
├── hooks/
│   ├── useAccelerometer.js
│   ├── useGyroscope.js
│   └── useMagnetometer.js
├── screens/
│   ├── SensorDashboard.js
│   └── MotionGame.js
└── utils/
    ├── sensorUtils.js
    └── calibration.js
```

#### Implementación de la App
```javascript
const SensorApp = () => {
  const [activeSensor, setActiveSensor] = useState('accelerometer');
  
  const sensorComponents = {
    accelerometer: <AccelerometerDisplay />,
    gyroscope: <GyroscopeDisplay />,
    magnetometer: <CompassView />,
    camera: <CameraView />
  };
  
  return (
    <View style={styles.container}>
      <View style={styles.sensorTabs}>
        {Object.keys(sensorComponents).map(sensor => (
          <TouchableOpacity
            key={sensor}
            onPress={() => setActiveSensor(sensor)}
            style={[
              styles.tab,
              activeSensor === sensor && styles.activeTab
            ]}
          >
            <Text style={styles.tabText}>
              {sensor.charAt(0).toUpperCase() + sensor.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
      
      <View style={styles.sensorContent}>
        {sensorComponents[activeSensor]}
      </View>
    </View>
  );
};
```

## 🎮 Ejercicios Prácticos

### Ejercicio 1: Control de Movimiento
Crea un juego que use el acelerómetro para controlar un personaje.

### Ejercicio 2: Brújula Digital
Implementa una brújula que muestre la dirección actual.

### Ejercicio 3: App de Sensores
Desarrolla una aplicación que muestre datos de todos los sensores.

## 📚 Recursos Adicionales

- [React Native Sensors](https://github.com/react-native-sensors/react-native-sensors)
- [React Native Camera](https://github.com/react-native-camera/react-native-camera)
- [React Native Voice](https://github.com/react-native-voice/voice)
- [Haptic Feedback](https://github.com/mkuczera/react-native-haptic-feedback)

## ✅ Checklist de la Clase

- [ ] Configurar sensores básicos
- [ ] Implementar controles de movimiento
- [ ] Crear brújula digital
- [ ] Integrar cámara y audio
- [ ] Implementar feedback háptico
- [ ] Completar la app de sensores

---

**Siguiente Clase**: Optimización de Rendimiento para Juegos y AR