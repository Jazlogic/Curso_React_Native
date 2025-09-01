# 📱 Clase 2: Expo SDK y APIs

## 🎯 Objetivos de la Clase
- Dominar las APIs principales del SDK de Expo
- Implementar funcionalidades nativas del dispositivo
- Integrar múltiples APIs en una aplicación
- Manejar permisos y configuraciones de APIs

---

## 📚 Contenido Teórico

### 1. Introducción al Expo SDK

#### **¿Qué es el Expo SDK?**
El **Expo SDK** es un conjunto de APIs pre-construidas que proporcionan acceso a funcionalidades nativas del dispositivo:

- **APIs unificadas**: Funcionan igual en iOS y Android
- **Configuración automática**: No requiere configuración nativa manual
- **Actualizaciones OTA**: Se actualizan sin recompilar la app
- **Documentación completa**: APIs bien documentadas y ejemplos

#### **Instalación del SDK**
```bash
# Instalar SDK completo
npx expo install expo

# O instalar APIs específicas
npx expo install expo-camera
npx expo install expo-location
npx expo install expo-notifications
npx expo install expo-file-system
npx expo install expo-media-library
```

### 2. API de Cámara (expo-camera)

#### **Configuración Básica**
```javascript
// App.js
import React, { useState, useEffect } from 'react';
import { StyleSheet, Text, View, TouchableOpacity } from 'react-native';
import { Camera, CameraType } from 'expo-camera';

export default function App() {
  const [hasPermission, setHasPermission] = useState(null);
  const [type, setType] = useState(CameraType.back);

  useEffect(() => {
    (async () => {
      const { status } = await Camera.requestCameraPermissionsAsync();
      setHasPermission(status === 'granted');
    })();
  }, []);

  if (hasPermission === null) {
    return <View />;
  }
  if (hasPermission === false) {
    return <Text>No access to camera</Text>;
  }

  return (
    <View style={styles.container}>
      <Camera style={styles.camera} type={type}>
        <View style={styles.buttonContainer}>
          <TouchableOpacity
            style={styles.button}
            onPress={() => {
              setType(type === CameraType.back ? CameraType.front : CameraType.back);
            }}>
            <Text style={styles.text}>Flip Camera</Text>
          </TouchableOpacity>
        </View>
      </Camera>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  camera: {
    flex: 1,
  },
  buttonContainer: {
    flex: 1,
    backgroundColor: 'transparent',
    flexDirection: 'row',
    margin: 20,
  },
  button: {
    flex: 0.1,
    alignSelf: 'flex-end',
    alignItems: 'center',
  },
  text: {
    fontSize: 18,
    color: 'white',
  },
});
```

#### **Funcionalidades Avanzadas**
```javascript
// CameraComponent.js
import React, { useState, useRef } from 'react';
import { View, TouchableOpacity, Text, Alert } from 'react-native';
import { Camera } from 'expo-camera';

export default function CameraComponent() {
  const [isRecording, setIsRecording] = useState(false);
  const cameraRef = useRef(null);

  // Tomar foto
  const takePicture = async () => {
    if (cameraRef.current) {
      try {
        const photo = await cameraRef.current.takePictureAsync({
          quality: 0.8,
          base64: true,
        });
        console.log('Foto tomada:', photo.uri);
        // Aquí puedes guardar la foto o enviarla
      } catch (error) {
        Alert.alert('Error', 'No se pudo tomar la foto');
      }
    }
  };

  // Grabar video
  const startRecording = async () => {
    if (cameraRef.current && !isRecording) {
      try {
        setIsRecording(true);
        const video = await cameraRef.current.recordAsync({
          quality: Camera.Constants.VideoQuality['720p'],
          maxDuration: 30,
        });
        console.log('Video grabado:', video.uri);
      } catch (error) {
        Alert.alert('Error', 'No se pudo grabar el video');
      } finally {
        setIsRecording(false);
      }
    }
  };

  const stopRecording = () => {
    if (cameraRef.current && isRecording) {
      cameraRef.current.stopRecording();
    }
  };

  return (
    <View style={{ flex: 1 }}>
      <Camera
        ref={cameraRef}
        style={{ flex: 1 }}
        type={Camera.Constants.Type.back}
      >
        <View style={{ flex: 1, backgroundColor: 'transparent' }}>
          <TouchableOpacity
            style={{
              alignSelf: 'center',
              margin: 20,
              padding: 15,
              backgroundColor: 'white',
              borderRadius: 10,
            }}
            onPress={takePicture}
          >
            <Text>Tomar Foto</Text>
          </TouchableOpacity>
          
          <TouchableOpacity
            style={{
              alignSelf: 'center',
              margin: 20,
              padding: 15,
              backgroundColor: isRecording ? 'red' : 'white',
              borderRadius: 10,
            }}
            onPress={isRecording ? stopRecording : startRecording}
          >
            <Text style={{ color: isRecording ? 'white' : 'black' }}>
              {isRecording ? 'Detener' : 'Grabar'}
            </Text>
          </TouchableOpacity>
        </View>
      </Camera>
    </View>
  );
}
```

### 3. API de Geolocalización (expo-location)

#### **Configuración y Uso Básico**
```javascript
// LocationComponent.js
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, Alert } from 'react-native';
import * as Location from 'expo-location';

export default function LocationComponent() {
  const [location, setLocation] = useState(null);
  const [errorMsg, setErrorMsg] = useState(null);

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        setErrorMsg('Permission to access location was denied');
        return;
      }

      let location = await Location.getCurrentPositionAsync({});
      setLocation(location);
    })();
  }, []);

  const getCurrentLocation = async () => {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        Alert.alert('Permisos', 'Se requieren permisos de ubicación');
        return;
      }

      const currentLocation = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High,
      });
      
      setLocation(currentLocation);
      console.log('Ubicación actual:', currentLocation);
    } catch (error) {
      Alert.alert('Error', 'No se pudo obtener la ubicación');
    }
  };

  const startLocationUpdates = async () => {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        Alert.alert('Permisos', 'Se requieren permisos de ubicación');
        return;
      }

      await Location.startLocationUpdatesAsync('location-task', {
        accuracy: Location.Accuracy.Balanced,
        timeInterval: 5000,
        distanceInterval: 10,
      });
    } catch (error) {
      Alert.alert('Error', 'No se pudo iniciar las actualizaciones de ubicación');
    }
  };

  let text = 'Waiting..';
  if (errorMsg) {
    text = errorMsg;
  } else if (location) {
    text = `Lat: ${location.coords.latitude}, Long: ${location.coords.longitude}`;
  }

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ fontSize: 18, marginBottom: 20 }}>{text}</Text>
      
      <TouchableOpacity
        style={{
          backgroundColor: '#007AFF',
          padding: 15,
          borderRadius: 10,
          marginBottom: 10,
        }}
        onPress={getCurrentLocation}
      >
        <Text style={{ color: 'white', fontSize: 16 }}>Obtener Ubicación</Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={{
          backgroundColor: '#34C759',
          padding: 15,
          borderRadius: 10,
        }}
        onPress={startLocationUpdates}
      >
        <Text style={{ color: 'white', fontSize: 16 }}>Iniciar Seguimiento</Text>
      </TouchableOpacity>
    </View>
  );
}
```

#### **Geocodificación y Mapas**
```javascript
// GeocodingComponent.js
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert } from 'react-native';
import * as Location from 'expo-location';

export default function GeocodingComponent() {
  const [address, setAddress] = useState('');
  const [coordinates, setCoordinates] = useState(null);

  const geocodeAddress = async () => {
    if (!address.trim()) {
      Alert.alert('Error', 'Por favor ingresa una dirección');
      return;
    }

    try {
      const results = await Location.geocodeAsync(address);
      if (results.length > 0) {
        const result = results[0];
        setCoordinates({
          latitude: result.latitude,
          longitude: result.longitude,
        });
        console.log('Coordenadas encontradas:', result);
      } else {
        Alert.alert('Error', 'No se encontró la dirección');
      }
    } catch (error) {
      Alert.alert('Error', 'No se pudo geocodificar la dirección');
    }
  };

  const reverseGeocode = async () => {
    if (!coordinates) {
      Alert.alert('Error', 'No hay coordenadas para geocodificar');
      return;
    }

    try {
      const results = await Location.reverseGeocodeAsync({
        latitude: coordinates.latitude,
        longitude: coordinates.longitude,
      });
      
      if (results.length > 0) {
        const result = results[0];
        const fullAddress = `${result.street}, ${result.city}, ${result.region}, ${result.country}`;
        Alert.alert('Dirección', fullAddress);
      }
    } catch (error) {
      Alert.alert('Error', 'No se pudo obtener la dirección');
    }
  };

  return (
    <View style={{ flex: 1, padding: 20 }}>
      <Text style={{ fontSize: 20, marginBottom: 20, textAlign: 'center' }}>
        Geocodificación
      </Text>

      <TextInput
        style={{
          borderWidth: 1,
          borderColor: '#ccc',
          borderRadius: 5,
          padding: 10,
          marginBottom: 20,
        }}
        placeholder="Ingresa una dirección"
        value={address}
        onChangeText={setAddress}
      />

      <TouchableOpacity
        style={{
          backgroundColor: '#007AFF',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={geocodeAddress}
      >
        <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
          Obtener Coordenadas
        </Text>
      </TouchableOpacity>

      {coordinates && (
        <View style={{ marginBottom: 20 }}>
          <Text style={{ fontSize: 16, marginBottom: 10 }}>
            Coordenadas encontradas:
          </Text>
          <Text>Latitud: {coordinates.latitude}</Text>
          <Text>Longitud: {coordinates.longitude}</Text>
        </View>
      )}

      {coordinates && (
        <TouchableOpacity
          style={{
            backgroundColor: '#34C759',
            padding: 15,
            borderRadius: 10,
          }}
          onPress={reverseGeocode}
        >
          <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
            Obtener Dirección
          </Text>
        </TouchableOpacity>
      )}
    </View>
  );
}
```

### 4. API de Notificaciones (expo-notifications)

#### **Configuración de Notificaciones Push**
```javascript
// NotificationComponent.js
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, TouchableOpacity, Alert } from 'react-native';
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

// Configurar comportamiento de notificaciones
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export default function NotificationComponent() {
  const [expoPushToken, setExpoPushToken] = useState('');
  const [notification, setNotification] = useState(false);
  const notificationListener = useRef();
  const responseListener = useRef();

  useEffect(() => {
    registerForPushNotificationsAsync().then(token => setExpoPushToken(token));

    notificationListener.current = Notifications.addNotificationReceivedListener(notification => {
      setNotification(notification);
    });

    responseListener.current = Notifications.addNotificationResponseReceivedListener(response => {
      console.log(response);
    });

    return () => {
      Notifications.removeNotificationSubscription(notificationListener.current);
      Notifications.removeNotificationSubscription(responseListener.current);
    };
  }, []);

  const registerForPushNotificationsAsync = async () => {
    let token;

    if (Device.isDevice) {
      const { status: existingStatus } = await Notifications.getPermissionsAsync();
      let finalStatus = existingStatus;
      if (existingStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }
      if (finalStatus !== 'granted') {
        alert('Failed to get push token for push notification!');
        return;
      }
      token = (await Notifications.getExpoPushTokenAsync()).data;
      console.log(token);
    } else {
      alert('Must use physical device for Push Notifications');
    }

    if (Platform.OS === 'android') {
      Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: '#FF231F7C',
      });
    }

    return token;
  };

  const sendLocalNotification = async () => {
    await Notifications.scheduleNotificationAsync({
      content: {
        title: "Notificación Local",
        body: "Esta es una notificación local de prueba",
        data: { data: 'goes here' },
      },
      trigger: { seconds: 2 },
    });
  };

  const sendPushNotification = async () => {
    const message = {
      to: expoPushToken,
      sound: 'default',
      title: 'Notificación Push',
      body: 'Esta es una notificación push de prueba',
      data: { someData: 'goes here' },
    };

    await fetch('https://exp.host/--/api/v2/push/send', {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Accept-encoding': 'gzip, deflate',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(message),
    });
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ fontSize: 20, marginBottom: 20 }}>
        Notificaciones
      </Text>

      <TouchableOpacity
        style={{
          backgroundColor: '#007AFF',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={sendLocalNotification}
      >
        <Text style={{ color: 'white', fontSize: 16 }}>
          Enviar Notificación Local
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={{
          backgroundColor: '#34C759',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={sendPushNotification}
      >
        <Text style={{ color: 'white', fontSize: 16 }}>
          Enviar Notificación Push
        </Text>
      </TouchableOpacity>

      {expoPushToken && (
        <Text style={{ fontSize: 12, textAlign: 'center', marginTop: 20 }}>
          Token: {expoPushToken}
        </Text>
      )}
    </View>
  );
}
```

### 5. API de Sistema de Archivos (expo-file-system)

#### **Manejo de Archivos**
```javascript
// FileSystemComponent.js
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, TextInput, Alert } from 'react-native';
import * as FileSystem from 'expo-file-system';

export default function FileSystemComponent() {
  const [fileName, setFileName] = useState('');
  const [fileContent, setFileContent] = useState('');
  const [savedFiles, setSavedFiles] = useState([]);

  const createFile = async () => {
    if (!fileName.trim() || !fileContent.trim()) {
      Alert.alert('Error', 'Por favor completa todos los campos');
      return;
    }

    try {
      const fileUri = FileSystem.documentDirectory + fileName + '.txt';
      await FileSystem.writeAsStringAsync(fileUri, fileContent);
      
      setSavedFiles(prev => [...prev, { name: fileName, uri: fileUri }]);
      setFileName('');
      setFileContent('');
      
      Alert.alert('Éxito', 'Archivo creado correctamente');
    } catch (error) {
      Alert.alert('Error', 'No se pudo crear el archivo');
    }
  };

  const readFile = async (fileUri) => {
    try {
      const content = await FileSystem.readAsStringAsync(fileUri);
      Alert.alert('Contenido del archivo', content);
    } catch (error) {
      Alert.alert('Error', 'No se pudo leer el archivo');
    }
  };

  const deleteFile = async (fileUri) => {
    try {
      await FileSystem.deleteAsync(fileUri);
      setSavedFiles(prev => prev.filter(file => file.uri !== fileUri));
      Alert.alert('Éxito', 'Archivo eliminado correctamente');
    } catch (error) {
      Alert.alert('Error', 'No se pudo eliminar el archivo');
    }
  };

  const listFiles = async () => {
    try {
      const files = await FileSystem.readDirectoryAsync(FileSystem.documentDirectory);
      console.log('Archivos en el directorio:', files);
    } catch (error) {
      Alert.alert('Error', 'No se pudo listar los archivos');
    }
  };

  return (
    <View style={{ flex: 1, padding: 20 }}>
      <Text style={{ fontSize: 20, marginBottom: 20, textAlign: 'center' }}>
        Sistema de Archivos
      </Text>

      <TextInput
        style={{
          borderWidth: 1,
          borderColor: '#ccc',
          borderRadius: 5,
          padding: 10,
          marginBottom: 10,
        }}
        placeholder="Nombre del archivo"
        value={fileName}
        onChangeText={setFileName}
      />

      <TextInput
        style={{
          borderWidth: 1,
          borderColor: '#ccc',
          borderRadius: 5,
          padding: 10,
          marginBottom: 20,
          height: 100,
          textAlignVertical: 'top',
        }}
        placeholder="Contenido del archivo"
        value={fileContent}
        onChangeText={setFileContent}
        multiline
      />

      <TouchableOpacity
        style={{
          backgroundColor: '#007AFF',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={createFile}
      >
        <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
          Crear Archivo
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={{
          backgroundColor: '#34C759',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={listFiles}
      >
        <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
          Listar Archivos
        </Text>
      </TouchableOpacity>

      {savedFiles.map((file, index) => (
        <View key={index} style={{ marginBottom: 10, padding: 10, backgroundColor: '#f0f0f0' }}>
          <Text style={{ fontSize: 16, marginBottom: 5 }}>{file.name}</Text>
          <View style={{ flexDirection: 'row' }}>
            <TouchableOpacity
              style={{
                backgroundColor: '#007AFF',
                padding: 8,
                borderRadius: 5,
                marginRight: 10,
              }}
              onPress={() => readFile(file.uri)}
            >
              <Text style={{ color: 'white', fontSize: 12 }}>Leer</Text>
            </TouchableOpacity>
            
            <TouchableOpacity
              style={{
                backgroundColor: '#FF3B30',
                padding: 8,
                borderRadius: 5,
              }}
              onPress={() => deleteFile(file.uri)}
            >
              <Text style={{ color: 'white', fontSize: 12 }}>Eliminar</Text>
            </TouchableOpacity>
          </View>
        </View>
      ))}
    </View>
  );
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: App de Cámara Completa
Crea una aplicación que integre múltiples funcionalidades de la cámara:

**Requisitos:**
- Cambio entre cámara frontal y trasera
- Captura de fotos con diferentes calidades
- Grabación de video con límite de tiempo
- Galería de fotos capturadas
- Guardado en el sistema de archivos

### Ejercicio 2: App de Ubicación y Mapas
Desarrolla una aplicación de ubicación completa:

**Requisitos:**
- Obtener ubicación actual
- Geocodificación de direcciones
- Seguimiento de ubicación en tiempo real
- Historial de ubicaciones visitadas
- Notificaciones cuando se llega a un destino

### Ejercicio 3: Sistema de Notificaciones
Implementa un sistema completo de notificaciones:

**Requisitos:**
- Notificaciones locales programadas
- Notificaciones push
- Diferentes tipos de notificaciones
- Manejo de respuestas del usuario
- Configuración de canales (Android)

---

## 🔍 Puntos Clave

1. **Expo SDK** proporciona APIs unificadas para funcionalidades nativas
2. **expo-camera** permite captura de fotos y video con configuración avanzada
3. **expo-location** ofrece geolocalización, geocodificación y seguimiento
4. **expo-notifications** maneja notificaciones locales y push
5. **expo-file-system** proporciona acceso completo al sistema de archivos
6. **Los permisos** son esenciales para el funcionamiento de las APIs
7. **La configuración** varía entre iOS y Android para algunas funcionalidades

---

## 📖 Recursos Adicionales

- [Expo Camera Documentation](https://docs.expo.dev/versions/latest/sdk/camera/)
- [Expo Location Documentation](https://docs.expo.dev/versions/latest/sdk/location/)
- [Expo Notifications Documentation](https://docs.expo.dev/versions/latest/sdk/notifications/)
- [Expo File System Documentation](https://docs.expo.dev/versions/latest/sdk/filesystem/)

---

## 🎉 ¡Clase Completada!

Has completado exitosamente la **Clase 2: Expo SDK y APIs** 🚀

**Lo que has aprendido:**
- ✅ APIs principales del SDK de Expo
- ✅ Implementación de cámara y captura multimedia
- ✅ Geolocalización y servicios de ubicación
- ✅ Sistema de notificaciones local y push
- ✅ Manejo del sistema de archivos

**Próximos pasos:**
- Practicar con los ejercicios integradores
- Explorar más APIs del SDK
- Continuar con la siguiente clase sobre herramientas de desarrollo

¡Excelente trabajo! 🎯

---

## 🔗 Navegación del Módulo

### **Clase Anterior**: [Clase 1: Fundamentos de Expo](clase_1_fundamentos_expo.md)
### **Clase Siguiente**: [Clase 3: Herramientas de Desarrollo](clase_3_herramientas_desarrollo.md)
### **Volver al Módulo**: [Módulo 8: Expo y Desarrollo Rápido](README.md)
### **Volver al Inicio**: [Índice Completo](../INDICE_COMPLETO.md)
