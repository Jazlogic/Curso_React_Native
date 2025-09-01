# 🚀 Clase 1: Fundamentos de Performance

## 📋 Objetivos de la Clase
- Comprender qué es performance y por qué es crucial en React Native
- Aprender las métricas clave de performance móvil
- Dominar las herramientas de profiling y medición
- Identificar cuellos de botella comunes en apps móviles
- Establecer una base sólida para optimización

## ⏱️ Duración
**1.5 horas**

## 🔗 Navegación
- **Anterior**: [Clase 5: Testing E2E y Performance](clase_5_testing_e2e_performance.md)
- **Siguiente**: [Clase 2: Optimización de Componentes](clase_2_optimizacion_componentes.md)
- **Módulo**: [README.md](README.md)
- **Inicio**: [🏠](../../README.md)

---

## 🎯 ¿Qué es Performance?

### Definición
Performance se refiere a qué tan rápido y eficientemente tu aplicación React Native responde a las acciones del usuario y consume recursos del dispositivo.

### ¿Por qué es Importante?
```javascript
// ❌ App lenta - Usuario frustrado
const SlowApp = () => {
  const [data, setData] = useState([]);
  
  // Carga todos los datos de una vez
  useEffect(() => {
    fetchAllData().then(setData); // Puede tomar 5-10 segundos
  }, []);
  
  return (
    <View>
      {data.map(item => (
        <HeavyComponent key={item.id} data={item} />
      ))}
    </View>
  );
};

// ✅ App optimizada - Usuario satisfecho
const FastApp = () => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  
  // Carga progresiva con indicador
  useEffect(() => {
    setLoading(true);
    fetchPaginatedData(1, 20).then(newData => {
      setData(newData);
      setLoading(false);
    });
  }, []);
  
  return (
    <View>
      {loading && <LoadingSpinner />}
      <FlatList
        data={data}
        renderItem={({item}) => <OptimizedComponent data={item} />}
        keyExtractor={item => item.id}
        onEndReached={() => loadMoreData()}
      />
    </View>
  );
};
```

---

## 📊 Métricas Clave de Performance

### 1. **Time to Interactive (TTI)**
Tiempo desde que se inicia la app hasta que el usuario puede interactuar.

```javascript
// Medición de TTI
import { PerformanceObserver } from 'react-native';

const measureTTI = () => {
  const startTime = Date.now();
  
  // Simular carga de la app
  const appReady = new Promise(resolve => {
    // App se considera lista cuando:
    // - Componentes principales renderizados
    // - Navegación inicializada
    // - Datos críticos cargados
    setTimeout(resolve, 1000);
  });
  
  appReady.then(() => {
    const tti = Date.now() - startTime;
    console.log(`TTI: ${tti}ms`);
    
    // Enviar métrica a analytics
    Analytics.track('app_tti', { value: tti });
  });
};
```

### 2. **Frame Rate (FPS)**
Frames por segundo - idealmente 60 FPS para experiencia fluida.

```javascript
// Monitoreo de FPS
import { InteractionManager } from 'react-native';

class FPSMonitor {
  constructor() {
    this.frameCount = 0;
    this.lastTime = Date.now();
    this.fps = 0;
  }
  
  start() {
    this.monitorFrame();
  }
  
  monitorFrame = () => {
    this.frameCount++;
    const currentTime = Date.now();
    
    if (currentTime - this.lastTime >= 1000) {
      this.fps = this.frameCount;
      this.frameCount = 0;
      this.lastTime = currentTime;
      
      console.log(`FPS: ${this.fps}`);
      
      // Alertar si FPS es bajo
      if (this.fps < 30) {
        console.warn('⚠️ FPS bajo detectado:', this.fps);
      }
    }
    
    // Continuar monitoreando
    requestAnimationFrame(this.monitorFrame);
  };
}

// Uso
const fpsMonitor = new FPSMonitor();
fpsMonitor.start();
```

### 3. **Memory Usage**
Consumo de memoria RAM del dispositivo.

```javascript
// Monitoreo de memoria
import { NativeModules } from 'react-native';

const { PerformanceModule } = NativeModules;

class MemoryMonitor {
  async getMemoryInfo() {
    try {
      const memoryInfo = await PerformanceModule.getMemoryInfo();
      
      console.log('📱 Memoria disponible:', memoryInfo.availableMemory, 'MB');
      console.log('💾 Memoria usada por la app:', memoryInfo.appMemory, 'MB');
      console.log('🔋 Memoria total del dispositivo:', memoryInfo.totalMemory, 'MB');
      
      // Calcular porcentaje de uso
      const usagePercentage = (memoryInfo.appMemory / memoryInfo.totalMemory) * 100;
      
      if (usagePercentage > 80) {
        console.warn('⚠️ Uso de memoria alto:', usagePercentage.toFixed(1) + '%');
      }
      
      return memoryInfo;
    } catch (error) {
      console.error('Error obteniendo información de memoria:', error);
    }
  }
  
  startMonitoring(interval = 5000) {
    setInterval(() => {
      this.getMemoryInfo();
    }, interval);
  }
}

// Uso
const memoryMonitor = new MemoryMonitor();
memoryMonitor.startMonitoring();
```

### 4. **Bundle Size**
Tamaño del JavaScript bundle que se descarga.

```javascript
// Verificación de bundle size
const checkBundleSize = () => {
  // En desarrollo, mostrar advertencias
  if (__DEV__) {
    console.log('📦 Bundle size en desarrollo');
    console.log('⚠️ Recuerda verificar el bundle de producción');
  }
  
  // En producción, monitorear tamaño
  if (!__DEV__) {
    // Métricas de descarga
    const downloadStart = performance.now();
    
    // Simular descarga del bundle
    fetch('/bundle.js')
      .then(response => {
        const downloadEnd = performance.now();
        const downloadTime = downloadEnd - downloadStart;
        
        console.log(`⏱️ Tiempo de descarga: ${downloadTime.toFixed(2)}ms`);
        console.log(`📊 Tamaño del bundle: ${response.headers.get('content-length')} bytes`);
      });
  }
};
```

---

## 🛠️ Herramientas de Profiling

### 1. **React DevTools Profiler**
```javascript
// Componente con profiling
import { Profiler } from 'react';

const ProfiledComponent = ({ children }) => {
  const onRenderCallback = (
    id, // ID único del componente
    phase, // "mount" (primera vez) o "update" (re-render)
    actualDuration, // Tiempo que tomó renderizar
    baseDuration, // Tiempo estimado para renderizar
    startTime, // Cuándo comenzó el render
    commitTime // Cuándo se completó el render
  ) => {
    console.log(`🔍 Profiling: ${id}`);
    console.log(`📊 Fase: ${phase}`);
    console.log(`⏱️ Duración: ${actualDuration}ms`);
    console.log(`📈 Duración base: ${baseDuration}ms`);
    
    // Alertar si el render es lento
    if (actualDuration > 16) { // 16ms = 60 FPS
      console.warn(`⚠️ Render lento en ${id}: ${actualDuration}ms`);
    }
  };
  
  return (
    <Profiler id="ProfiledComponent" onRender={onRenderCallback}>
      {children}
    </Profiler>
  );
};

// Uso
const App = () => (
  <ProfiledComponent>
    <HeavyComponent />
  </ProfiledComponent>
);
```

### 2. **Performance API**
```javascript
// Medición de performance personalizada
class PerformanceTracker {
  constructor() {
    this.marks = new Map();
    this.measures = new Map();
  }
  
  // Marcar un punto en el tiempo
  mark(name) {
    const timestamp = performance.now();
    this.marks.set(name, timestamp);
    console.log(`📍 Mark: ${name} en ${timestamp.toFixed(2)}ms`);
  }
  
  // Medir tiempo entre dos marks
  measure(name, startMark, endMark) {
    const start = this.marks.get(startMark);
    const end = this.marks.get(endMark);
    
    if (start && end) {
      const duration = end - start;
      this.measures.set(name, duration);
      
      console.log(`⏱️ Measure: ${name} = ${duration.toFixed(2)}ms`);
      
      // Alertar si la operación es lenta
      if (duration > 100) {
        console.warn(`🐌 Operación lenta: ${name} (${duration.toFixed(2)}ms)`);
      }
      
      return duration;
    } else {
      console.error('❌ Marks no encontrados para la medida');
    }
  }
  
  // Obtener todas las medidas
  getMeasures() {
    return Object.fromEntries(this.measures);
  }
  
  // Limpiar datos
  clear() {
    this.marks.clear();
    this.measures.clear();
  }
}

// Uso en componentes
const OptimizedComponent = () => {
  const perfTracker = useMemo(() => new PerformanceTracker(), []);
  
  useEffect(() => {
    perfTracker.mark('component-start');
    
    // Simular operación pesada
    const heavyOperation = () => {
      perfTracker.mark('operation-start');
      
      // Simular trabajo intensivo
      let result = 0;
      for (let i = 0; i < 1000000; i++) {
        result += Math.random();
      }
      
      perfTracker.mark('operation-end');
      perfTracker.measure('heavy-operation', 'operation-start', 'operation-end');
      
      return result;
    };
    
    const result = heavyOperation();
    
    perfTracker.mark('component-end');
    perfTracker.measure('total-render', 'component-start', 'component-end');
    
    console.log('📊 Medidas de performance:', perfTracker.getMeasures());
  }, []);
  
  return <View><Text>Componente Optimizado</Text></View>;
};
```

### 3. **Flipper para React Native**
```javascript
// Configuración para Flipper
import { addPlugin } from 'react-native-flipper';

// Plugin de performance
class PerformancePlugin {
  constructor() {
    this.id = 'performance-plugin';
    this.title = 'Performance Monitor';
  }
  
  // Enviar métricas a Flipper
  sendMetric(name, value, unit = 'ms') {
    if (global.flipperFlipperPlugin) {
      global.flipperFlipperPlugin.send('performance-metric', {
        name,
        value,
        unit,
        timestamp: Date.now()
      });
    }
  }
  
  // Monitorear renders de componentes
  monitorComponent(componentName) {
    return {
      onRender: (duration) => {
        this.sendMetric(`${componentName}-render`, duration);
      },
      onUpdate: (duration) => {
        this.sendMetric(`${componentName}-update`, duration);
      }
    };
  }
}

// Uso
const perfPlugin = new PerformancePlugin();
addPlugin(perfPlugin);
```

---

## 🚨 Cuellos de Botella Comunes

### 1. **Re-renders Innecesarios**
```javascript
// ❌ Problema: Re-renders innecesarios
const BadComponent = ({ data, onUpdate }) => {
  // Este componente se re-renderiza cada vez que el padre se actualiza
  // incluso si data no cambió
  
  return (
    <View>
      <Text>{data.name}</Text>
      <Button 
        title="Actualizar" 
        onPress={onUpdate} // Nueva función cada render
      />
    </View>
  );
};

// ✅ Solución: Memoización y callbacks estables
const GoodComponent = React.memo(({ data, onUpdate }) => {
  // Solo se re-renderiza si data cambia
  
  return (
    <View>
      <Text>{data.name}</Text>
      <Button 
        title="Actualizar" 
        onPress={onUpdate} // Función estable
      />
    </View>
  );
});

// En el componente padre
const ParentComponent = () => {
  const [data, setData] = useState({ name: 'Test' });
  
  // Callback estable con useCallback
  const handleUpdate = useCallback(() => {
    setData(prev => ({ ...prev, updated: true }));
  }, []);
  
  return <GoodComponent data={data} onUpdate={handleUpdate} />;
};
```

### 2. **Operaciones Síncronas en el Thread Principal**
```javascript
// ❌ Problema: Operación pesada en el thread principal
const SlowComponent = () => {
  const [result, setResult] = useState(0);
  
  const heavyCalculation = () => {
    // Esto bloquea la UI por 2-3 segundos
    let sum = 0;
    for (let i = 0; i < 100000000; i++) {
      sum += Math.sqrt(i);
    }
    return sum;
  };
  
  const handlePress = () => {
    const result = heavyCalculation(); // Bloquea la UI
    setResult(result);
  };
  
  return (
    <View>
      <Text>Resultado: {result}</Text>
      <Button title="Calcular" onPress={handlePress} />
    </View>
  );
};

// ✅ Solución: Usar InteractionManager o requestIdleCallback
const FastComponent = () => {
  const [result, setResult] = useState(0);
  const [calculating, setCalculating] = useState(false);
  
  const heavyCalculation = () => {
    let sum = 0;
    for (let i = 0; i < 100000000; i++) {
      sum += Math.sqrt(i);
    }
    return sum;
  };
  
  const handlePress = () => {
    setCalculating(true);
    
    // Ejecutar cuando la UI esté libre
    InteractionManager.runAfterInteractions(() => {
      const result = heavyCalculation();
      setResult(result);
      setCalculating(false);
    });
  };
  
  return (
    <View>
      <Text>Resultado: {result}</Text>
      <Button 
        title={calculating ? "Calculando..." : "Calcular"} 
        onPress={handlePress}
        disabled={calculating}
      />
      {calculating && <ActivityIndicator />}
    </View>
  );
};
```

### 3. **Imágenes No Optimizadas**
```javascript
// ❌ Problema: Imágenes grandes sin optimización
const BadImageComponent = () => {
  return (
    <View>
      <Image 
        source={{ uri: 'https://example.com/large-image.jpg' }}
        style={{ width: 300, height: 200 }}
        // Sin lazy loading, sin placeholder, sin cache
      />
    </View>
  );
};

// ✅ Solución: Imágenes optimizadas
const GoodImageComponent = () => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [imageError, setImageError] = useState(false);
  
  return (
    <View>
      {!imageLoaded && !imageError && (
        <View style={styles.placeholder}>
          <ActivityIndicator size="large" />
        </View>
      )}
      
      <Image 
        source={{ 
          uri: 'https://example.com/optimized-image.jpg',
          // Parámetros de optimización
          width: 300,
          height: 200,
          cache: 'force-cache'
        }}
        style={[
          styles.image,
          { opacity: imageLoaded ? 1 : 0 }
        ]}
        onLoadStart={() => setImageLoaded(false)}
        onLoad={() => setImageLoaded(true)}
        onError={() => setImageError(true)}
        // Lazy loading automático
        loadingIndicatorSource={require('./placeholder.png')}
        // Progreso de carga
        onProgress={(event) => {
          const progress = event.nativeEvent.loaded / event.nativeEvent.total;
          console.log(`📸 Carga de imagen: ${(progress * 100).toFixed(1)}%`);
        }}
      />
      
      {imageError && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>Error cargando imagen</Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  placeholder: {
    width: 300,
    height: 200,
    backgroundColor: '#f0f0f0',
    justifyContent: 'center',
    alignItems: 'center'
  },
  image: {
    width: 300,
    height: 200,
    borderRadius: 8
  },
  errorContainer: {
    width: 300,
    height: 200,
    backgroundColor: '#ffebee',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 8
  },
  errorText: {
    color: '#c62828',
    fontSize: 16
  }
});
```

---

## 📱 Ejercicios Prácticos

### Ejercicio 1: Medidor de Performance
Crea un componente que mida y muestre métricas de performance en tiempo real.

```javascript
// Tu código aquí
const PerformanceMeter = () => {
  // Implementa:
  // 1. Medición de FPS
  // 2. Monitoreo de memoria
  // 3. Tiempo de render
  // 4. Alertas cuando las métricas sean bajas
};
```

### Ejercicio 2: Profiler de Componentes
Implementa un sistema de profiling que identifique componentes lentos.

```javascript
// Tu código aquí
const ComponentProfiler = ({ children, name }) => {
  // Implementa:
  // 1. Medición de tiempo de render
  // 2. Detección de re-renders innecesarios
  // 3. Reporte de performance
};
```

### Ejercicio 3: Optimizador de Imágenes
Crea un componente que optimice automáticamente las imágenes.

```javascript
// Tu código aquí
const OptimizedImage = ({ source, style, placeholder }) => {
  // Implementa:
  // 1. Lazy loading
  // 2. Placeholder mientras carga
  // 3. Cache de imágenes
  // 4. Redimensionado automático
};
```

---

## 🔍 Resumen de la Clase

### ✅ Lo que Aprendiste
- **Performance** es crucial para la experiencia del usuario
- **Métricas clave**: TTI, FPS, Memory Usage, Bundle Size
- **Herramientas**: React DevTools, Performance API, Flipper
- **Cuellos de botella**: Re-renders, operaciones pesadas, imágenes no optimizadas

### 🎯 Próximos Pasos
En la siguiente clase aprenderás:
- **React.memo** para evitar re-renders innecesarios
- **useMemo** y **useCallback** para optimización
- Técnicas avanzadas de memoización de componentes

### 💡 Consejos Clave
1. **Mide primero, optimiza después** - No optimices sin datos
2. **60 FPS es el objetivo** - Cada frame debe completarse en <16ms
3. **Monitorea en producción** - Las métricas de desarrollo no siempre reflejan la realidad
4. **Optimiza progresivamente** - Enfócate en los cuellos de botella más grandes primero

---

## 📚 Recursos Adicionales
- [React Native Performance](https://reactnative.dev/docs/performance)
- [React Profiler API](https://reactjs.org/docs/profiler.html)
- [Performance Best Practices](https://reactnative.dev/docs/performance#my-components-re-render-too-often-what-can-i-do)
- [Flipper Documentation](https://fbflipper.com/)

---

**¿Tienes alguna pregunta sobre los fundamentos de performance? ¿Te gustaría que profundice en algún aspecto específico antes de continuar con la siguiente clase?**
