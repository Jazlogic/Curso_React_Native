# Clase 2: Realidad Aumentada con ARKit y ARCore

## Objetivos de la Clase
- Comprender los fundamentos de la Realidad Aumentada en React Native
- Aprender sobre ARKit (iOS) y ARCore (Android)
- Implementar aplicaciones AR básicas
- Integrar AR con React Native usando librerías especializadas

## Contenido de la Clase

### 1. Introducción a la Realidad Aumentada

#### ¿Qué es la Realidad Aumentada?
- **Definición:** Superposición de elementos digitales sobre el mundo real
- **Diferencias con VR:** AR mantiene la conexión con el entorno real
- **Aplicaciones:** Gaming, educación, retail, arquitectura, medicina

#### Tipos de AR
- **Marker-based:** Basada en marcadores visuales
- **Markerless:** Basada en reconocimiento de superficies
- **Location-based:** Basada en ubicación GPS
- **Projection-based:** Proyección de luz sobre superficies

### 2. ARKit vs ARCore

#### ARKit (iOS)
- **Características:**
  - Face tracking
  - Object detection
  - Light estimation
  - Plane detection
  - Image tracking

#### ARCore (Android)
- **Características:**
  - Motion tracking
  - Environmental understanding
  - Light estimation
  - Cloud anchors
  - Augmented images

### 3. Librerías para AR en React Native

#### React Native AR
```bash
npm install react-native-ar
```

#### ViroReact (Deprecated pero útil para referencia)
```bash
npm install react-viro
```

#### React Native ARCore
```bash
npm install react-native-arcore
```

#### React Native ARKit
```bash
npm install react-native-arkit
```

### 4. Configuración del Proyecto

#### Dependencias Necesarias
```json
{
  "dependencies": {
    "react-native-ar": "^0.1.0",
    "react-native-permissions": "^3.0.0",
    "react-native-vector-icons": "^9.0.0"
  }
}
```

#### Permisos Requeridos
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

```xml
<!-- ios/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>Esta app necesita acceso a la cámara para AR</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Esta app necesita acceso a la ubicación para AR</string>
```

### 5. Implementación Básica de AR

#### Componente AR Básico
```jsx
// ARView.js
import React, { useState, useEffect } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import { RNCamera } from 'react-native-camera';
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

const ARView = () => {
  const [hasPermission, setHasPermission] = useState(null);
  const [arObjects, setArObjects] = useState([]);

  useEffect(() => {
    requestCameraPermission();
  }, []);

  const requestCameraPermission = async () => {
    const result = await request(PERMISSIONS.ANDROID.CAMERA);
    setHasPermission(result === RESULTS.GRANTED);
  };

  const addARObject = (event) => {
    const { locationX, locationY } = event.nativeEvent;
    const newObject = {
      id: Date.now(),
      x: locationX,
      y: locationY,
      type: 'cube'
    };
    setArObjects([...arObjects, newObject]);
  };

  if (hasPermission === null) {
    return <View />;
  }
  if (hasPermission === false) {
    return <Text>No se tiene permiso para usar la cámara</Text>;
  }

  return (
    <View style={styles.container}>
      <RNCamera
        style={styles.camera}
        type={RNCamera.Constants.Type.back}
        onTouchEnd={addARObject}
      >
        {arObjects.map((obj) => (
          <View
            key={obj.id}
            style={[
              styles.arObject,
              {
                left: obj.x - 25,
                top: obj.y - 25
              }
            ]}
          />
        ))}
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
  arObject: {
    position: 'absolute',
    width: 50,
    height: 50,
    backgroundColor: 'red',
    borderRadius: 25
  }
});

export default ARView;
```

### 6. AR con React Native AR

#### Implementación Avanzada
```jsx
// AdvancedAR.js
import React, { useState, useRef } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import { ARView, ARPlane, ARBox } from 'react-native-ar';

const AdvancedAR = () => {
  const [planes, setPlanes] = useState([]);
  const [objects, setObjects] = useState([]);
  const arRef = useRef(null);

  const onPlaneDetected = (plane) => {
    setPlanes(prev => [...prev, plane]);
  };

  const addObject = (plane) => {
    const newObject = {
      id: Date.now(),
      position: plane.position,
      rotation: [0, 0, 0],
      scale: [0.1, 0.1, 0.1]
    };
    setObjects(prev => [...prev, newObject]);
  };

  const resetScene = () => {
    setObjects([]);
    setPlanes([]);
  };

  return (
    <View style={styles.container}>
      <ARView
        ref={arRef}
        style={styles.arView}
        onPlaneDetected={onPlaneDetected}
      >
        {planes.map((plane) => (
          <ARPlane
            key={plane.id}
            position={plane.position}
            rotation={plane.rotation}
            scale={plane.scale}
            onPress={() => addObject(plane)}
          />
        ))}
        
        {objects.map((obj) => (
          <ARBox
            key={obj.id}
            position={obj.position}
            rotation={obj.rotation}
            scale={obj.scale}
            color="blue"
          />
        ))}
      </ARView>
      
      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={resetScene}>
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  controls: {
    position: 'absolute',
    bottom: 50,
    left: 0,
    right: 0,
    flexDirection: 'row',
    justifyContent: 'center'
  },
  button: {
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 15,
    borderRadius: 10,
    marginHorizontal: 10
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold'
  }
});

export default AdvancedAR;
```

### 7. AR con Modelos 3D

#### Integración de Modelos 3D
```jsx
// ARModelViewer.js
import React, { useState } from 'react';
import { View, StyleSheet, Text } from 'react-native';
import { ARView, ARModel } from 'react-native-ar';

const ARModelViewer = () => {
  const [modelLoaded, setModelLoaded] = useState(false);

  const onModelLoad = () => {
    setModelLoaded(true);
  };

  return (
    <View style={styles.container}>
      <ARView style={styles.arView}>
        <ARModel
          source="https://example.com/model.obj"
          position={[0, 0, -1]}
          rotation={[0, 0, 0]}
          scale={[0.5, 0.5, 0.5]}
          onLoad={onModelLoad}
        />
      </ARView>
      
      {!modelLoaded && (
        <View style={styles.loading}>
          <Text style={styles.loadingText}>Cargando modelo 3D...</Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  loading: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.5)'
  },
  loadingText: {
    color: 'white',
    fontSize: 18
  }
});

export default ARModelViewer;
```

### 8. AR con Reconocimiento de Imágenes

#### Image Tracking
```jsx
// ARImageTracking.js
import React, { useState } from 'react';
import { View, StyleSheet, Text } from 'react-native';
import { ARView, ARImage, ARBox } from 'react-native-ar';

const ARImageTracking = () => {
  const [imageDetected, setImageDetected] = useState(false);

  const onImageDetected = (image) => {
    setImageDetected(true);
  };

  return (
    <View style={styles.container}>
      <ARView style={styles.arView}>
        <ARImage
          source="https://example.com/marker.jpg"
          onDetected={onImageDetected}
        >
          {imageDetected && (
            <ARBox
              position={[0, 0, 0]}
              rotation={[0, 0, 0]}
              scale={[0.1, 0.1, 0.1]}
              color="green"
            />
          )}
        </ARImage>
      </ARView>
      
      <View style={styles.status}>
        <Text style={styles.statusText}>
          {imageDetected ? 'Imagen detectada' : 'Buscando imagen...'}
        </Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  status: {
    position: 'absolute',
    top: 50,
    left: 0,
    right: 0,
    alignItems: 'center'
  },
  statusText: {
    color: 'white',
    fontSize: 16,
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 10,
    borderRadius: 5
  }
});

export default ARImageTracking;
```

### 9. Optimización y Mejores Prácticas

#### Rendimiento en AR
- **Optimización de modelos 3D:** Reducir polígonos y texturas
- **Gestión de memoria:** Liberar recursos no utilizados
- **Frame rate:** Mantener 60 FPS para experiencia fluida
- **Batería:** Optimizar uso de sensores y cámara

#### Mejores Prácticas
- **UX/UI:** Interfaz intuitiva y no intrusiva
- **Accesibilidad:** Considerar usuarios con discapacidades
- **Privacidad:** Manejar datos de cámara y ubicación
- **Testing:** Probar en diferentes dispositivos y condiciones

### 10. Casos de Uso Avanzados

#### AR para E-commerce
```jsx
// ARProductViewer.js
import React, { useState } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import { ARView, ARModel } from 'react-native-ar';

const ARProductViewer = ({ product }) => {
  const [modelScale, setModelScale] = useState([1, 1, 1]);

  const scaleUp = () => {
    setModelScale(prev => [prev[0] * 1.1, prev[1] * 1.1, prev[2] * 1.1]);
  };

  const scaleDown = () => {
    setModelScale(prev => [prev[0] * 0.9, prev[1] * 0.9, prev[2] * 0.9]);
  };

  return (
    <View style={styles.container}>
      <ARView style={styles.arView}>
        <ARModel
          source={product.modelUrl}
          position={[0, 0, -1]}
          rotation={[0, 0, 0]}
          scale={modelScale}
        />
      </ARView>
      
      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={scaleUp}>
          <Text style={styles.buttonText}>+</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.button} onPress={scaleDown}>
          <Text style={styles.buttonText}>-</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  controls: {
    position: 'absolute',
    bottom: 50,
    right: 20,
    flexDirection: 'column'
  },
  button: {
    backgroundColor: 'rgba(0,0,0,0.7)',
    width: 50,
    height: 50,
    borderRadius: 25,
    justifyContent: 'center',
    alignItems: 'center',
    marginVertical: 5
  },
  buttonText: {
    color: 'white',
    fontSize: 20,
    fontWeight: 'bold'
  }
});

export default ARProductViewer;
```

## Recursos Adicionales

### Documentación
- [ARKit Documentation](https://developer.apple.com/documentation/arkit)
- [ARCore Documentation](https://developers.google.com/ar)
- [React Native AR](https://github.com/react-native-ar/react-native-ar)

### Herramientas
- [Blender](https://www.blender.org/) - Para crear modelos 3D
- [Vuforia](https://developer.vuforia.com/) - Para reconocimiento de imágenes
- [Unity](https://unity.com/) - Para desarrollo de AR avanzado

### Tutoriales
- [AR Development with React Native](https://blog.logrocket.com/ar-development-react-native/)
- [Building AR Apps with ARKit](https://developer.apple.com/arkit/)

## Ejercicios Prácticos

### Ejercicio 1: AR Básico
Crear una aplicación AR que permita colocar objetos 3D en el mundo real.

### Ejercicio 2: Image Tracking
Implementar reconocimiento de imágenes para activar contenido AR.

### Ejercicio 3: AR E-commerce
Crear un visualizador de productos en AR para una tienda online.

## Evaluación

### Criterios de Evaluación
- **Funcionalidad (40%):** La aplicación AR funciona correctamente
- **Rendimiento (30%):** Optimización y fluidez
- **Código (20%):** Estructura y calidad del código
- **Innovación (10%):** Elementos únicos de la aplicación

### Entregables
- Código fuente de la aplicación AR
- Demo de la aplicación funcionando
- Documentación de optimizaciones
- Reflexión sobre el proceso de desarrollo

---

**Duración estimada:** 2.5 horas  
**Dificultad:** Avanzada  
**Prerrequisitos:** Conocimientos sólidos de React Native, experiencia con APIs nativas
