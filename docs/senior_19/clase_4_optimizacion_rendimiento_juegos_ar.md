# Clase 4: Optimización de Rendimiento para Juegos y AR

## Objetivos de la Clase
- Comprender las técnicas de optimización específicas para juegos y AR
- Aprender sobre profiling y debugging de rendimiento
- Implementar estrategias de optimización avanzadas
- Optimizar el uso de memoria y CPU en aplicaciones intensivas

## Contenido de la Clase

### 1. Fundamentos de Optimización de Rendimiento

#### Métricas de Rendimiento
- **Frame Rate:** 60 FPS para experiencia fluida
- **Memory Usage:** Gestión eficiente de memoria
- **CPU Usage:** Optimización de cálculos
- **Battery Life:** Eficiencia energética
- **Startup Time:** Tiempo de carga inicial

#### Herramientas de Profiling
```bash
# React Native Performance
npm install --save-dev react-native-performance

# Flipper para debugging
npm install --save-dev react-native-flipper
```

### 2. Optimización de Renderizado

#### React Native Reanimated
```jsx
// OptimizedAnimation.js
import React, { useRef } from 'react';
import { View, StyleSheet, TouchableOpacity, Text } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
  runOnJS
} from 'react-native-reanimated';

const OptimizedAnimation = () => {
  const scale = useSharedValue(1);
  const opacity = useSharedValue(1);
  const rotation = useSharedValue(0);

  const animateScale = () => {
    scale.value = withSpring(scale.value === 1 ? 1.2 : 1, {
      damping: 15,
      stiffness: 150
    });
  };

  const animateOpacity = () => {
    opacity.value = withTiming(opacity.value === 1 ? 0.5 : 1, {
      duration: 300
    });
  };

  const animateRotation = () => {
    rotation.value = withTiming(rotation.value + 360, {
      duration: 1000
    });
  };

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { scale: scale.value },
      { rotate: `${rotation.value}deg` }
    ],
    opacity: opacity.value
  }));

  return (
    <View style={styles.container}>
      <Animated.View style={[styles.box, animatedStyle]} />
      
      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={animateScale}>
          <Text style={styles.buttonText}>Escala</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={animateOpacity}>
          <Text style={styles.buttonText}>Opacidad</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={animateRotation}>
          <Text style={styles.buttonText}>Rotación</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5'
  },
  box: {
    width: 100,
    height: 100,
    backgroundColor: '#4CAF50',
    borderRadius: 10,
    marginBottom: 50
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    width: '100%',
    paddingHorizontal: 20
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 10,
    minWidth: 80,
    alignItems: 'center'
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold'
  }
});

export default OptimizedAnimation;
```

#### Memoización y Optimización de Componentes
```jsx
// OptimizedGameComponent.js
import React, { memo, useCallback, useMemo } from 'react';
import { View, StyleSheet, Text } from 'react-native';

const GameEntity = memo(({ position, size, color, onPress }) => {
  const style = useMemo(() => ({
    position: 'absolute',
    left: position.x,
    top: position.y,
    width: size.width,
    height: size.height,
    backgroundColor: color
  }), [position.x, position.y, size.width, size.height, color]);

  const handlePress = useCallback(() => {
    onPress?.(position);
  }, [onPress, position]);

  return (
    <View style={style} onTouchEnd={handlePress} />
  );
});

const GameWorld = memo(({ entities, onEntityPress }) => {
  const entityComponents = useMemo(() => {
    return entities.map((entity, index) => (
      <GameEntity
        key={entity.id || index}
        position={entity.position}
        size={entity.size}
        color={entity.color}
        onPress={onEntityPress}
      />
    ));
  }, [entities, onEntityPress]);

  return (
    <View style={styles.gameWorld}>
      {entityComponents}
    </View>
  );
});

const styles = StyleSheet.create({
  gameWorld: {
    flex: 1,
    position: 'relative'
  }
});

export default GameWorld;
```

### 3. Optimización de Memoria

#### Gestión de Memoria para Juegos
```jsx
// MemoryManager.js
import React, { useState, useEffect, useRef } from 'react';
import { View, StyleSheet, Text, TouchableOpacity, Alert } from 'react-native';
import DeviceInfo from 'react-native-device-info';

const MemoryManager = () => {
  const [memoryUsage, setMemoryUsage] = useState(0);
  const [totalMemory, setTotalMemory] = useState(0);
  const [entities, setEntities] = useState([]);
  const memoryInterval = useRef(null);

  useEffect(() => {
    initializeMemoryMonitoring();
    return () => {
      if (memoryInterval.current) {
        clearInterval(memoryInterval.current);
      }
    };
  }, []);

  const initializeMemoryMonitoring = async () => {
    try {
      const total = await DeviceInfo.getTotalMemory();
      setTotalMemory(total);
      
      memoryInterval.current = setInterval(async () => {
        const used = await DeviceInfo.getUsedMemory();
        setMemoryUsage(used);
      }, 1000);
    } catch (error) {
      console.log('Error initializing memory monitoring:', error);
    }
  };

  const createEntities = () => {
    const newEntities = Array.from({ length: 1000 }, (_, index) => ({
      id: index,
      position: { x: Math.random() * 300, y: Math.random() * 500 },
      size: { width: 20, height: 20 },
      color: `#${Math.floor(Math.random() * 16777215).toString(16)}`
    }));
    setEntities(newEntities);
  };

  const clearEntities = () => {
    setEntities([]);
    // Forzar garbage collection si está disponible
    if (global.gc) {
      global.gc();
    }
  };

  const optimizeMemory = () => {
    // Limpiar caché y liberar memoria
    clearEntities();
    Alert.alert('Optimización', 'Memoria optimizada');
  };

  const memoryPercentage = totalMemory > 0 ? (memoryUsage / totalMemory) * 100 : 0;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Gestión de Memoria</Text>
      
      <View style={styles.memoryInfo}>
        <Text style={styles.memoryText}>
          Memoria Usada: {memoryUsage} MB
        </Text>
        <Text style={styles.memoryText}>
          Memoria Total: {totalMemory} MB
        </Text>
        <Text style={styles.memoryText}>
          Porcentaje: {memoryPercentage.toFixed(1)}%
        </Text>
        <Text style={styles.memoryText}>
          Entidades: {entities.length}
        </Text>
      </View>

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={createEntities}>
          <Text style={styles.buttonText}>Crear Entidades</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={clearEntities}>
          <Text style={styles.buttonText}>Limpiar Entidades</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={optimizeMemory}>
          <Text style={styles.buttonText}>Optimizar Memoria</Text>
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
  memoryInfo: {
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
  memoryText: {
    fontSize: 16,
    marginBottom: 5,
    color: '#333'
  },
  controls: {
    flex: 1,
    justifyContent: 'center'
  },
  button: {
    backgroundColor: '#2196F3',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    alignItems: 'center'
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  }
});

export default MemoryManager;
```

#### Object Pooling
```jsx
// ObjectPool.js
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    this.active = new Set();
    
    // Crear objetos iniciales
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }

  get() {
    let obj;
    if (this.pool.length > 0) {
      obj = this.pool.pop();
    } else {
      obj = this.createFn();
    }
    
    this.active.add(obj);
    return obj;
  }

  release(obj) {
    if (this.active.has(obj)) {
      this.active.delete(obj);
      this.resetFn(obj);
      this.pool.push(obj);
    }
  }

  releaseAll() {
    this.active.forEach(obj => {
      this.resetFn(obj);
      this.pool.push(obj);
    });
    this.active.clear();
  }

  getActiveCount() {
    return this.active.size;
  }

  getPoolSize() {
    return this.pool.length;
  }
}

// Uso del Object Pool
const bulletPool = new ObjectPool(
  () => ({ x: 0, y: 0, velocity: 0, active: false }),
  (bullet) => {
    bullet.x = 0;
    bullet.y = 0;
    bullet.velocity = 0;
    bullet.active = false;
  },
  50
);

export default ObjectPool;
```

### 4. Optimización de AR

#### Optimización de Renderizado AR
```jsx
// OptimizedARView.js
import React, { useState, useRef, useCallback } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import { ARView, ARPlane, ARBox } from 'react-native-ar';

const OptimizedARView = () => {
  const [planes, setPlanes] = useState([]);
  const [objects, setObjects] = useState([]);
  const [isTracking, setIsTracking] = useState(false);
  const arRef = useRef(null);
  const frameCount = useRef(0);
  const lastFrameTime = useRef(Date.now());

  const onPlaneDetected = useCallback((plane) => {
    setPlanes(prev => {
      // Limitar el número de planos para optimizar rendimiento
      if (prev.length >= 10) {
        return [...prev.slice(1), plane];
      }
      return [...prev, plane];
    });
  }, []);

  const addObject = useCallback((plane) => {
    setObjects(prev => {
      // Limitar el número de objetos
      if (prev.length >= 20) {
        return [...prev.slice(1), {
          id: Date.now(),
          position: plane.position,
          rotation: [0, 0, 0],
          scale: [0.1, 0.1, 0.1]
        }];
      }
      return [...prev, {
        id: Date.now(),
        position: plane.position,
        rotation: [0, 0, 0],
        scale: [0.1, 0.1, 0.1]
      }];
    });
  }, []);

  const onFrameUpdate = useCallback(() => {
    frameCount.current++;
    const now = Date.now();
    
    // Calcular FPS cada segundo
    if (now - lastFrameTime.current >= 1000) {
      const fps = frameCount.current;
      frameCount.current = 0;
      lastFrameTime.current = now;
      
      // Log FPS para debugging
      console.log(`FPS: ${fps}`);
    }
  }, []);

  const resetScene = useCallback(() => {
    setObjects([]);
    setPlanes([]);
  }, []);

  const toggleTracking = useCallback(() => {
    setIsTracking(prev => !prev);
  }, []);

  return (
    <View style={styles.container}>
      <ARView
        ref={arRef}
        style={styles.arView}
        onPlaneDetected={onPlaneDetected}
        onFrameUpdate={onFrameUpdate}
        enablePlaneDetection={isTracking}
      >
        {planes.map((plane) => (
          <ARPlane
            key={plane.id}
            position={plane.position}
            rotation={plane.rotation}
            scale={plane.scale}
            onPress={() => addObject(plane)}
            opacity={0.3}
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
        <TouchableOpacity
          style={[styles.button, isTracking ? styles.activeButton : styles.inactiveButton]}
          onPress={toggleTracking}
        >
          <Text style={styles.buttonText}>
            {isTracking ? 'Detener Tracking' : 'Iniciar Tracking'}
          </Text>
        </TouchableOpacity>
        
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
    justifyContent: 'space-around',
    paddingHorizontal: 20
  },
  button: {
    padding: 15,
    borderRadius: 10,
    minWidth: 120,
    alignItems: 'center'
  },
  activeButton: {
    backgroundColor: '#f44336'
  },
  inactiveButton: {
    backgroundColor: '#4CAF50'
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
    fontSize: 14
  }
});

export default OptimizedARView;
```

### 5. Optimización de Red y Datos

#### Caching y Lazy Loading
```jsx
// DataManager.js
import AsyncStorage from '@react-native-async-storage/async-storage';

class DataManager {
  constructor() {
    this.cache = new Map();
    this.cacheExpiry = new Map();
    this.maxCacheSize = 100;
    this.cacheTimeout = 5 * 60 * 1000; // 5 minutos
  }

  async get(key) {
    // Verificar si está en caché y no ha expirado
    if (this.cache.has(key)) {
      const expiry = this.cacheExpiry.get(key);
      if (Date.now() < expiry) {
        return this.cache.get(key);
      } else {
        // Limpiar entrada expirada
        this.cache.delete(key);
        this.cacheExpiry.delete(key);
      }
    }

    // Intentar cargar desde AsyncStorage
    try {
      const stored = await AsyncStorage.getItem(key);
      if (stored) {
        const data = JSON.parse(stored);
        this.set(key, data);
        return data;
      }
    } catch (error) {
      console.log('Error loading from AsyncStorage:', error);
    }

    return null;
  }

  set(key, value) {
    // Limpiar caché si está lleno
    if (this.cache.size >= this.maxCacheSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
      this.cacheExpiry.delete(firstKey);
    }

    this.cache.set(key, value);
    this.cacheExpiry.set(key, Date.now() + this.cacheTimeout);

    // Guardar en AsyncStorage
    AsyncStorage.setItem(key, JSON.stringify(value)).catch(error => {
      console.log('Error saving to AsyncStorage:', error);
    });
  }

  clear() {
    this.cache.clear();
    this.cacheExpiry.clear();
    AsyncStorage.clear().catch(error => {
      console.log('Error clearing AsyncStorage:', error);
    });
  }

  getCacheStats() {
    return {
      size: this.cache.size,
      maxSize: this.maxCacheSize,
      keys: Array.from(this.cache.keys())
    };
  }
}

export default DataManager;
```

### 6. Profiling y Debugging

#### Performance Monitor
```jsx
// PerformanceMonitor.js
import React, { useState, useEffect, useRef } from 'react';
import { View, StyleSheet, Text, TouchableOpacity } from 'react-native';
import DeviceInfo from 'react-native-device-info';

const PerformanceMonitor = () => {
  const [metrics, setMetrics] = useState({
    fps: 0,
    memory: 0,
    cpu: 0,
    battery: 0
  });
  const [isMonitoring, setIsMonitoring] = useState(false);
  const frameCount = useRef(0);
  const lastTime = useRef(Date.now());
  const monitoringInterval = useRef(null);

  useEffect(() => {
    if (isMonitoring) {
      startMonitoring();
    } else {
      stopMonitoring();
    }

    return () => stopMonitoring();
  }, [isMonitoring]);

  const startMonitoring = () => {
    monitoringInterval.current = setInterval(async () => {
      try {
        const memory = await DeviceInfo.getUsedMemory();
        const battery = await DeviceInfo.getBatteryLevel();
        
        setMetrics(prev => ({
          ...prev,
          memory,
          battery: battery * 100
        }));
      } catch (error) {
        console.log('Error monitoring performance:', error);
      }
    }, 1000);
  };

  const stopMonitoring = () => {
    if (monitoringInterval.current) {
      clearInterval(monitoringInterval.current);
      monitoringInterval.current = null;
    }
  };

  const onFrameUpdate = () => {
    frameCount.current++;
    const now = Date.now();
    
    if (now - lastTime.current >= 1000) {
      const fps = frameCount.current;
      frameCount.current = 0;
      lastTime.current = now;
      
      setMetrics(prev => ({
        ...prev,
        fps
      }));
    }
  };

  const toggleMonitoring = () => {
    setIsMonitoring(prev => !prev);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Monitor de Rendimiento</Text>
      
      <View style={styles.metricsContainer}>
        <View style={styles.metric}>
          <Text style={styles.metricLabel}>FPS</Text>
          <Text style={styles.metricValue}>{metrics.fps}</Text>
        </View>
        
        <View style={styles.metric}>
          <Text style={styles.metricLabel}>Memoria (MB)</Text>
          <Text style={styles.metricValue}>{metrics.memory.toFixed(0)}</Text>
        </View>
        
        <View style={styles.metric}>
          <Text style={styles.metricLabel}>Batería (%)</Text>
          <Text style={styles.metricValue}>{metrics.battery.toFixed(0)}</Text>
        </View>
      </View>

      <TouchableOpacity
        style={[styles.button, isMonitoring ? styles.stopButton : styles.startButton]}
        onPress={toggleMonitoring}
      >
        <Text style={styles.buttonText}>
          {isMonitoring ? 'Detener Monitoreo' : 'Iniciar Monitoreo'}
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
  metricsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 30
  },
  metric: {
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  metricLabel: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5
  },
  metricValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333'
  },
  button: {
    padding: 20,
    borderRadius: 10,
    alignItems: 'center'
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

export default PerformanceMonitor;
```

## Recursos Adicionales

### Documentación
- [React Native Performance](https://reactnative.dev/docs/performance)
- [React Native Reanimated](https://docs.swmansion.com/react-native-reanimated/)
- [Flipper](https://fbflipper.com/)

### Herramientas
- [React Native Performance](https://github.com/oblador/react-native-performance)
- [React Native Flipper](https://github.com/facebook/flipper)

### Tutoriales
- [Optimizing React Native Performance](https://blog.logrocket.com/optimizing-react-native-performance/)
- [React Native Performance Best Practices](https://reactnative.dev/docs/performance)

## Ejercicios Prácticos

### Ejercicio 1: Optimización de Animaciones
Implementar animaciones optimizadas usando React Native Reanimated.

### Ejercicio 2: Gestión de Memoria
Crear un sistema de gestión de memoria para un juego.

### Ejercicio 3: Monitor de Rendimiento
Implementar un monitor de rendimiento en tiempo real.

## Evaluación

### Criterios de Evaluación
- **Rendimiento (40%):** Optimización implementada correctamente
- **Código (30%):** Estructura y calidad del código
- **Funcionalidad (20%):** La aplicación funciona correctamente
- **Documentación (10%):** Documentación de optimizaciones

### Entregables
- Código fuente optimizado
- Documentación de optimizaciones
- Métricas de rendimiento
- Demo de la aplicación funcionando

---

**Duración estimada:** 2.5 horas  
**Dificultad:** Avanzada  
**Prerrequisitos:** Conocimientos sólidos de React Native, experiencia con optimización
