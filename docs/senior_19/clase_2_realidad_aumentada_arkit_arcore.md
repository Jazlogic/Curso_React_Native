# Clase 2: Realidad Aumentada con ARKit y ARCore

## 🎯 Objetivos de la Clase

- Comprender los fundamentos de la realidad aumentada
- Configurar ARKit para iOS y ARCore para Android
- Implementar detección de planos y tracking
- Crear objetos virtuales en el mundo real
- Desarrollar una aplicación AR funcional

## 📋 Contenido

### 1. Introducción a la Realidad Aumentada

#### ¿Qué es la Realidad Aumentada?
La realidad aumentada (AR) es una tecnología que superpone información digital sobre el mundo real, creando una experiencia híbrida que combina elementos virtuales y físicos.

#### Diferencias entre AR, VR y MR
- **AR (Augmented Reality)**: Superpone contenido digital sobre el mundo real
- **VR (Virtual Reality)**: Crea un entorno completamente virtual
- **MR (Mixed Reality)**: Combina AR y VR con interacción entre objetos reales y virtuales

#### Aplicaciones de AR en Móviles
- **Retail**: Visualización de productos en casa
- **Educación**: Modelos 3D interactivos
- **Gaming**: Juegos que usan el entorno real
- **Navegación**: Direcciones superpuestas
- **Medicina**: Visualización de anatomía

### 2. ARKit para iOS

#### Configuración de ARKit
```javascript
import { ARKit } from 'react-native-arkit';

// Verificar disponibilidad
const checkARKitSupport = async () => {
  const isSupported = await ARKit.isARKitSupported();
  if (!isSupported) {
    console.log('ARKit no está disponible en este dispositivo');
    return false;
  }
  return true;
};
```

#### Configuración de la Escena AR
```javascript
import { ARKit } from 'react-native-arkit';

const ARScene = () => {
  const [arState, setArState] = useState('initializing');
  
  const onARKitError = (error) => {
    console.error('ARKit Error:', error);
    setArState('error');
  };
  
  const onARKitInitialized = () => {
    setArState('ready');
  };
  
  return (
    <ARKit
      style={{ flex: 1 }}
      onARKitError={onARKitError}
      onARKitInitialized={onARKitInitialized}
      planeDetection={['horizontal', 'vertical']}
      lightEstimation={true}
    >
      {/* Contenido AR aquí */}
    </ARKit>
  );
};
```

#### Detección de Planos
```javascript
const PlaneDetection = () => {
  const [detectedPlanes, setDetectedPlanes] = useState([]);
  
  const onPlaneDetected = (plane) => {
    setDetectedPlanes(prev => [...prev, plane]);
  };
  
  const onPlaneUpdated = (plane) => {
    setDetectedPlanes(prev => 
      prev.map(p => p.id === plane.id ? plane : p)
    );
  };
  
  return (
    <ARKit
      onPlaneDetected={onPlaneDetected}
      onPlaneUpdated={onPlaneUpdated}
    >
      {detectedPlanes.map(plane => (
        <ARKit.Plane
          key={plane.id}
          position={plane.center}
          width={plane.extent.x}
          height={plane.extent.z}
          material={{ color: 'rgba(0,255,0,0.3)' }}
        />
      ))}
    </ARKit>
  );
};
```

### 3. ARCore para Android

#### Configuración de ARCore
```javascript
import { ARCore } from 'react-native-arcore';

// Verificar disponibilidad
const checkARCoreSupport = async () => {
  const isSupported = await ARCore.isARCoreSupported();
  if (!isSupported) {
    console.log('ARCore no está disponible en este dispositivo');
    return false;
  }
  return true;
};
```

#### Configuración de la Sesión AR
```javascript
const ARCoreScene = () => {
  const [sessionState, setSessionState] = useState('initializing');
  
  const onSessionCreated = () => {
    setSessionState('created');
  };
  
  const onSessionResumed = () => {
    setSessionState('resumed');
  };
  
  const onSessionPaused = () => {
    setSessionState('paused');
  };
  
  return (
    <ARCore
      style={{ flex: 1 }}
      onSessionCreated={onSessionCreated}
      onSessionResumed={onSessionResumed}
      onSessionPaused={onSessionPaused}
      planeDetectionMode="horizontal"
      lightEstimationMode="environmentalHDR"
    >
      {/* Contenido AR aquí */}
    </ARCore>
  );
};
```

### 4. Creación de Objetos Virtuales

#### Objetos 3D Básicos
```javascript
const VirtualObjects = () => {
  const [objects, setObjects] = useState([]);
  
  const addCube = (position) => {
    const newObject = {
      id: Date.now(),
      type: 'cube',
      position: position,
      scale: [0.1, 0.1, 0.1],
      color: 'red'
    };
    setObjects(prev => [...prev, newObject]);
  };
  
  const addSphere = (position) => {
    const newObject = {
      id: Date.now(),
      type: 'sphere',
      position: position,
      scale: [0.1, 0.1, 0.1],
      color: 'blue'
    };
    setObjects(prev => [...prev, newObject]);
  };
  
  return (
    <ARKit>
      {objects.map(obj => (
        <ARKit.Box
          key={obj.id}
          position={obj.position}
          scale={obj.scale}
          material={{ color: obj.color }}
        />
      ))}
    </ARKit>
  );
};
```

#### Modelos 3D Complejos
```javascript
const Model3D = ({ modelPath, position, scale }) => {
  return (
    <ARKit.Model
      position={position}
      scale={scale}
      model={{
        file: modelPath,
        scale: [1, 1, 1]
      }}
      material={{
        color: 'white',
        metalness: 0.8,
        roughness: 0.2
      }}
    />
  );
};

// Uso del modelo
<Model3D
  modelPath="models/chair.usdz"
  position={[0, 0, -0.5]}
  scale={[0.5, 0.5, 0.5]}
/>
```

### 5. Interacciones y Gestos

#### Detección de Toques
```javascript
const TouchInteraction = () => {
  const onTap = (event) => {
    const { position, hitTest } = event;
    
    // Realizar hit test para encontrar planos
    const hitTestResults = hitTest(position);
    
    if (hitTestResults.length > 0) {
      const hit = hitTestResults[0];
      const worldPosition = hit.worldTransform.position;
      
      // Crear objeto en la posición tocada
      addObjectAtPosition(worldPosition);
    }
  };
  
  return (
    <ARKit
      onTap={onTap}
    >
      {/* Contenido AR */}
    </ARKit>
  );
};
```

#### Gestos de Pizca y Rotación
```javascript
const GestureHandling = () => {
  const [selectedObject, setSelectedObject] = useState(null);
  
  const onPinch = (event) => {
    if (selectedObject) {
      const scale = event.scale;
      updateObjectScale(selectedObject.id, scale);
    }
  };
  
  const onRotation = (event) => {
    if (selectedObject) {
      const rotation = event.rotation;
      updateObjectRotation(selectedObject.id, rotation);
    }
  };
  
  return (
    <ARKit
      onPinch={onPinch}
      onRotation={onRotation}
    >
      {/* Contenido AR */}
    </ARKit>
  );
};
```

### 6. Iluminación y Materiales

#### Estimación de Luz
```javascript
const LightingSystem = () => {
  const [lightEstimate, setLightEstimate] = useState(null);
  
  const onLightEstimate = (estimate) => {
    setLightEstimate(estimate);
  };
  
  return (
    <ARKit
      onLightEstimate={onLightEstimate}
      lightEstimation={true}
    >
      {lightEstimate && (
        <ARKit.Light
          type="directional"
          intensity={lightEstimate.ambientIntensity}
          color={lightEstimate.ambientColorTemperature}
        />
      )}
    </ARKit>
  );
};
```

#### Materiales Realistas
```javascript
const RealisticMaterials = () => {
  const materials = {
    metal: {
      color: 'silver',
      metalness: 1.0,
      roughness: 0.1
    },
    plastic: {
      color: 'red',
      metalness: 0.0,
      roughness: 0.5
    },
    glass: {
      color: 'white',
      metalness: 0.0,
      roughness: 0.0,
      transparency: 0.8
    }
  };
  
  return (
    <ARKit>
      <ARKit.Box
        position={[-0.2, 0, -0.5]}
        material={materials.metal}
      />
      <ARKit.Sphere
        position={[0, 0, -0.5]}
        material={materials.plastic}
      />
      <ARKit.Cylinder
        position={[0.2, 0, -0.5]}
        material={materials.glass}
      />
    </ARKit>
  );
};
```

### 7. Proyecto Práctico: AR Furniture App

#### Estructura del Proyecto
```
src/
├── components/
│   ├── ARScene.js
│   ├── FurnitureItem.js
│   ├── PlaneDetector.js
│   └── ObjectManipulator.js
├── models/
│   ├── chair.usdz
│   ├── table.usdz
│   └── lamp.usdz
├── screens/
│   ├── ARView.js
│   └── FurnitureCatalog.js
└── utils/
    ├── arUtils.js
    └── modelLoader.js
```

#### Implementación de la App
```javascript
const ARFurnitureApp = () => {
  const [selectedFurniture, setSelectedFurniture] = useState(null);
  const [placedObjects, setPlacedObjects] = useState([]);
  
  const placeFurniture = (position) => {
    if (selectedFurniture) {
      const newObject = {
        id: Date.now(),
        model: selectedFurniture.model,
        position: position,
        scale: [0.5, 0.5, 0.5]
      };
      setPlacedObjects(prev => [...prev, newObject]);
    }
  };
  
  return (
    <View style={{ flex: 1 }}>
      <ARScene onTap={placeFurniture}>
        {placedObjects.map(obj => (
          <Model3D
            key={obj.id}
            modelPath={obj.model}
            position={obj.position}
            scale={obj.scale}
          />
        ))}
      </ARScene>
      
      <FurnitureCatalog
        onSelect={setSelectedFurniture}
        selected={selectedFurniture}
      />
    </View>
  );
};
```

## 🎮 Ejercicios Prácticos

### Ejercicio 1: Detector de Planos
Implementa detección de planos horizontales y verticales.

### Ejercicio 2: Objetos Interactivos
Crea objetos que respondan a toques y gestos.

### Ejercicio 3: App de Muebles AR
Desarrolla una aplicación para visualizar muebles en AR.

## 📚 Recursos Adicionales

- [ARKit Documentation](https://developer.apple.com/augmented-reality/arkit/)
- [ARCore Documentation](https://developers.google.com/ar)
- [React Native AR](https://github.com/react-native-ar/react-native-ar)
- [AR Design Guidelines](https://developer.apple.com/design/human-interface-guidelines/ar/)

## ✅ Checklist de la Clase

- [ ] Configurar ARKit y ARCore
- [ ] Implementar detección de planos
- [ ] Crear objetos virtuales 3D
- [ ] Manejar interacciones y gestos
- [ ] Configurar iluminación y materiales
- [ ] Completar la app de muebles AR

---

**Siguiente Clase**: Integración de Sensores y Hardware