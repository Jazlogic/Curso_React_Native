# Clase 4: Optimización de Rendimiento para Juegos y AR

## 🎯 Objetivos de la Clase

- Comprender los desafíos de rendimiento en juegos y AR
- Implementar técnicas de optimización de frame rate
- Gestionar memoria eficientemente
- Optimizar el uso de batería
- Aplicar técnicas de optimización nativa

## 📋 Contenido

### 1. Fundamentos de Rendimiento

#### Métricas de Rendimiento
- **Frame Rate (FPS)**: Frames por segundo
- **Frame Time**: Tiempo entre frames
- **Memory Usage**: Uso de memoria RAM
- **CPU Usage**: Uso del procesador
- **GPU Usage**: Uso de la tarjeta gráfica
- **Battery Drain**: Consumo de batería

#### Herramientas de Profiling
```javascript
import { Performance } from 'react-native-performance';

const PerformanceMonitor = () => {
  const [metrics, setMetrics] = useState({});
  
  useEffect(() => {
    const startTime = Performance.now();
    
    // Tu código aquí
    
    const endTime = Performance.now();
    const duration = endTime - startTime;
    
    setMetrics(prev => ({
      ...prev,
      lastOperation: duration
    }));
  }, []);
  
  return (
    <View>
      <Text>Tiempo de operación: {metrics.lastOperation}ms</Text>
    </View>
  );
};
```

### 2. Optimización de Frame Rate

#### Target de 60 FPS
```javascript
import { useFrameCallback } from 'react-native-reanimated';

const GameLoop = () => {
  const [frameCount, setFrameCount] = useState(0);
  const [fps, setFps] = useState(0);
  const lastTime = useRef(0);
  
  useFrameCallback((frameInfo) => {
    const { timestamp } = frameInfo;
    const deltaTime = timestamp - lastTime.current;
    
    if (deltaTime >= 16.67) { // 60 FPS = 16.67ms por frame
      setFrameCount(prev => prev + 1);
      setFps(1000 / deltaTime);
      lastTime.current = timestamp;
      
      // Lógica del juego aquí
      updateGame(deltaTime);
    }
  });
  
  return (
    <View>
      <Text>FPS: {fps.toFixed(1)}</Text>
      <Text>Frames: {frameCount}</Text>
    </View>
  );
};
```

#### Optimización de Renderizado
```javascript
import { memo, useMemo } from 'react';

const OptimizedGameObject = memo(({ position, rotation, scale }) => {
  const transform = useMemo(() => ({
    transform: [
      { translateX: position.x },
      { translateY: position.y },
      { rotate: `${rotation}deg` },
      { scale: scale }
    ]
  }), [position.x, position.y, rotation, scale]);
  
  return (
    <View style={[styles.gameObject, transform]}>
      {/* Contenido del objeto */}
    </View>
  );
});

// Uso en el juego
const GameScene = ({ objects }) => {
  const visibleObjects = useMemo(() => {
    return objects.filter(obj => isObjectVisible(obj));
  }, [objects]);
  
  return (
    <View>
      {visibleObjects.map(obj => (
        <OptimizedGameObject
          key={obj.id}
          position={obj.position}
          rotation={obj.rotation}
          scale={obj.scale}
        />
      ))}
    </View>
  );
};
```

#### Culling y LOD (Level of Detail)
```javascript
const LODSystem = () => {
  const [cameraPosition, setCameraPosition] = useState({ x: 0, y: 0, z: 0 });
  
  const getLODLevel = (objectPosition) => {
    const distance = calculateDistance(cameraPosition, objectPosition);
    
    if (distance < 10) return 'high';
    if (distance < 50) return 'medium';
    return 'low';
  };
  
  const renderObject = (object) => {
    const lodLevel = getLODLevel(object.position);
    
    switch (lodLevel) {
      case 'high':
        return <HighDetailObject {...object} />;
      case 'medium':
        return <MediumDetailObject {...object} />;
      case 'low':
        return <LowDetailObject {...object} />;
    }
  };
  
  return (
    <View>
      {gameObjects.map(obj => renderObject(obj))}
    </View>
  );
};
```

### 3. Gestión de Memoria

#### Memory Pooling
```javascript
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    this.active = new Set();
    
    // Pre-crear objetos
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
}

// Uso del pool
const bulletPool = new ObjectPool(
  () => ({ x: 0, y: 0, active: false }),
  (bullet) => { bullet.active = false; }
);

const createBullet = (x, y) => {
  const bullet = bulletPool.get();
  bullet.x = x;
  bullet.y = y;
  bullet.active = true;
  return bullet;
};
```

#### Garbage Collection Optimization
```javascript
const OptimizedGameLoop = () => {
  const gameState = useRef({
    objects: [],
    particles: [],
    lastGC: 0
  });
  
  const updateGame = useCallback((deltaTime) => {
    const state = gameState.current;
    
    // Actualizar objetos
    state.objects.forEach(obj => {
      if (obj.active) {
        obj.update(deltaTime);
      }
    });
    
    // Limpiar objetos inactivos
    state.objects = state.objects.filter(obj => obj.active);
    
    // Garbage collection cada 5 segundos
    if (Date.now() - state.lastGC > 5000) {
      state.lastGC = Date.now();
      
      // Forzar garbage collection si está disponible
      if (global.gc) {
        global.gc();
      }
    }
  }, []);
  
  return <GameView onUpdate={updateGame} />;
};
```

### 4. Optimización de Batería

#### Battery-Aware Rendering
```javascript
import { Battery } from 'react-native-battery';

const BatteryOptimizedGame = () => {
  const [batteryLevel, setBatteryLevel] = useState(100);
  const [isCharging, setIsCharging] = useState(false);
  
  useEffect(() => {
    const subscription = Battery.addListener((battery) => {
      setBatteryLevel(battery.level);
      setIsCharging(battery.isCharging);
    });
    
    return () => subscription.remove();
  }, []);
  
  const getOptimalSettings = () => {
    if (batteryLevel < 20 && !isCharging) {
      return {
        targetFPS: 30,
        quality: 'low',
        effects: false
      };
    } else if (batteryLevel < 50) {
      return {
        targetFPS: 45,
        quality: 'medium',
        effects: true
      };
    } else {
      return {
        targetFPS: 60,
        quality: 'high',
        effects: true
      };
    }
  };
  
  const settings = getOptimalSettings();
  
  return (
    <GameView
      targetFPS={settings.targetFPS}
      quality={settings.quality}
      effects={settings.effects}
    />
  );
};
```

#### Adaptive Quality
```javascript
const AdaptiveQuality = () => {
  const [quality, setQuality] = useState('high');
  const [performance, setPerformance] = useState({ fps: 60, frameTime: 16.67 });
  
  useEffect(() => {
    const interval = setInterval(() => {
      // Monitorear rendimiento
      const currentFPS = getCurrentFPS();
      const currentFrameTime = getCurrentFrameTime();
      
      setPerformance({ fps: currentFPS, frameTime: currentFrameTime });
      
      // Ajustar calidad basado en rendimiento
      if (currentFPS < 45) {
        setQuality('low');
      } else if (currentFPS < 55) {
        setQuality('medium');
      } else {
        setQuality('high');
      }
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  const getQualitySettings = () => {
    switch (quality) {
      case 'low':
        return {
          shadowQuality: 0,
          textureQuality: 0.5,
          particleCount: 10,
          drawDistance: 50
        };
      case 'medium':
        return {
          shadowQuality: 1,
          textureQuality: 0.75,
          particleCount: 25,
          drawDistance: 100
        };
      case 'high':
        return {
          shadowQuality: 2,
          textureQuality: 1.0,
          particleCount: 50,
          drawDistance: 200
        };
    }
  };
  
  return (
    <GameView qualitySettings={getQualitySettings()} />
  );
};
```

### 5. Optimización de GPU

#### Texture Optimization
```javascript
const TextureManager = () => {
  const [textures, setTextures] = useState(new Map());
  
  const loadTexture = useCallback(async (texturePath) => {
    if (textures.has(texturePath)) {
      return textures.get(texturePath);
    }
    
    try {
      const texture = await loadTextureFromPath(texturePath);
      setTextures(prev => new Map(prev).set(texturePath, texture));
      return texture;
    } catch (error) {
      console.error('Error loading texture:', error);
      return null;
    }
  }, [textures]);
  
  const preloadTextures = useCallback(async (texturePaths) => {
    const promises = texturePaths.map(path => loadTexture(path));
    await Promise.all(promises);
  }, [loadTexture]);
  
  return { loadTexture, preloadTextures };
};
```

#### Shader Optimization
```javascript
const OptimizedShader = () => {
  const vertexShader = `
    attribute vec4 position;
    attribute vec2 texCoord;
    uniform mat4 mvpMatrix;
    varying vec2 vTexCoord;
    
    void main() {
      gl_Position = mvpMatrix * position;
      vTexCoord = texCoord;
    }
  `;
  
  const fragmentShader = `
    precision mediump float;
    varying vec2 vTexCoord;
    uniform sampler2D texture;
    uniform float alpha;
    
    void main() {
      vec4 color = texture2D(texture, vTexCoord);
      gl_FragColor = vec4(color.rgb, color.a * alpha);
    }
  `;
  
  return { vertexShader, fragmentShader };
};
```

### 6. Optimización Nativa

#### Native Modules para Performance
```javascript
import { NativeModules } from 'react-native';

const { PerformanceModule } = NativeModules;

const NativeOptimization = () => {
  const [nativeMetrics, setNativeMetrics] = useState({});
  
  useEffect(() => {
    const getNativeMetrics = async () => {
      try {
        const metrics = await PerformanceModule.getMetrics();
        setNativeMetrics(metrics);
      } catch (error) {
        console.error('Error getting native metrics:', error);
      }
    };
    
    getNativeMetrics();
  }, []);
  
  const optimizeNative = async () => {
    try {
      await PerformanceModule.optimizeMemory();
      await PerformanceModule.setThreadPriority('high');
    } catch (error) {
      console.error('Error optimizing native:', error);
    }
  };
  
  return (
    <View>
      <Text>CPU Usage: {nativeMetrics.cpuUsage}%</Text>
      <Text>Memory Usage: {nativeMetrics.memoryUsage}MB</Text>
      <TouchableOpacity onPress={optimizeNative}>
        <Text>Optimizar Nativamente</Text>
      </TouchableOpacity>
    </View>
  );
};
```

#### JSI (JavaScript Interface) Optimization
```javascript
import { TurboModuleRegistry } from 'react-native';

const PerformanceTurboModule = TurboModuleRegistry.get('PerformanceTurboModule');

const JSIOptimization = () => {
  const [isOptimized, setIsOptimized] = useState(false);
  
  const enableJSI = async () => {
    try {
      await PerformanceTurboModule.enableJSI();
      setIsOptimized(true);
    } catch (error) {
      console.error('Error enabling JSI:', error);
    }
  };
  
  const fastCalculation = (data) => {
    if (isOptimized) {
      return PerformanceTurboModule.fastCalculation(data);
    } else {
      // Fallback a JavaScript
      return slowCalculation(data);
    }
  };
  
  return (
    <View>
      <TouchableOpacity onPress={enableJSI}>
        <Text>Habilitar JSI</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### 7. Proyecto Práctico: Game Performance Monitor

#### Estructura del Proyecto
```
src/
├── components/
│   ├── PerformanceMonitor.js
│   ├── FPSDisplay.js
│   ├── MemoryGraph.js
│   └── BatteryIndicator.js
├── hooks/
│   ├── usePerformance.js
│   ├── useMemory.js
│   └── useBattery.js
├── utils/
│   ├── performanceUtils.js
│   ├── memoryUtils.js
│   └── optimizationUtils.js
└── screens/
    ├── PerformanceDashboard.js
    └── OptimizationSettings.js
```

#### Implementación del Monitor
```javascript
const PerformanceDashboard = () => {
  const [metrics, setMetrics] = useState({
    fps: 0,
    frameTime: 0,
    memoryUsage: 0,
    cpuUsage: 0,
    batteryLevel: 100
  });
  
  const [optimizations, setOptimizations] = useState({
    targetFPS: 60,
    quality: 'high',
    memoryLimit: 100,
    batteryOptimization: false
  });
  
  useEffect(() => {
    const interval = setInterval(() => {
      const newMetrics = {
        fps: getCurrentFPS(),
        frameTime: getCurrentFrameTime(),
        memoryUsage: getMemoryUsage(),
        cpuUsage: getCPUUsage(),
        batteryLevel: getBatteryLevel()
      };
      
      setMetrics(newMetrics);
      
      // Aplicar optimizaciones automáticas
      applyAutomaticOptimizations(newMetrics, optimizations);
    }, 1000);
    
    return () => clearInterval(interval);
  }, [optimizations]);
  
  return (
    <View style={styles.dashboard}>
      <FPSDisplay fps={metrics.fps} target={optimizations.targetFPS} />
      <MemoryGraph usage={metrics.memoryUsage} limit={optimizations.memoryLimit} />
      <BatteryIndicator level={metrics.batteryLevel} />
      
      <OptimizationSettings
        settings={optimizations}
        onSettingsChange={setOptimizations}
      />
    </View>
  );
};
```

## 🎮 Ejercicios Prácticos

### Ejercicio 1: FPS Monitor
Implementa un monitor de FPS en tiempo real.

### Ejercicio 2: Memory Pool
Crea un sistema de pool de objetos para optimizar memoria.

### Ejercicio 3: Adaptive Quality
Desarrolla un sistema de calidad adaptativa.

## 📚 Recursos Adicionales

- [React Native Performance](https://reactnative.dev/docs/performance)
- [Flipper Performance](https://fbflipper.com/)
- [Android Profiler](https://developer.android.com/studio/profile)
- [Xcode Instruments](https://developer.apple.com/xcode/)

## ✅ Checklist de la Clase

- [ ] Implementar monitor de FPS
- [ ] Optimizar gestión de memoria
- [ ] Configurar optimización de batería
- [ ] Aplicar técnicas de GPU
- [ ] Implementar optimización nativa
- [ ] Completar el monitor de rendimiento

---

**Siguiente Clase**: Casos de Uso Avanzados y Proyectos