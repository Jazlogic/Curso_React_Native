# 📱 Clase 1: Fundamentos de Performance en React Native

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Comprender** los conceptos fundamentales de performance en React Native
2. **Identificar** los cuellos de botella más comunes en aplicaciones móviles
3. **Aplicar** técnicas básicas de optimización de rendimiento
4. **Utilizar** herramientas de profiling para medir performance
5. **Implementar** mejores prácticas de performance desde el diseño

---

## 📚 Contenido Teórico

### **1. ¿Qué es Performance en React Native?**

Performance se refiere a qué tan rápido y eficientemente funciona tu aplicación. En React Native, esto incluye:

- **Tiempo de respuesta**: Qué tan rápido responde la UI a las interacciones
- **Fluidez**: Qué tan suave son las animaciones y transiciones
- **Uso de memoria**: Cuánta RAM consume la aplicación
- **Consumo de batería**: Qué tan eficiente es en el uso de energía
- **Tiempo de carga**: Qué tan rápido se inicia la aplicación

### **2. Por Qué la Performance es Crítica en Móviles**

#### **Limitaciones del Hardware Móvil:**
```javascript
// Los dispositivos móviles tienen limitaciones específicas
const mobileConstraints = {
  cpu: 'Limitado y compartido con el sistema',
  ram: 'Generalmente 2-8GB vs 16-32GB en desktop',
  bateria: 'Recurso finito y crítico',
  red: 'Conexiones inestables y lentas',
  pantalla: 'Tamaño limitado y diferentes densidades'
};
```

#### **Expectativas del Usuario:**
- **60 FPS**: Los usuarios esperan animaciones fluidas
- **< 100ms**: Respuesta inmediata a toques
- **< 3 segundos**: Carga inicial rápida
- **Sin lag**: Experiencia fluida en todo momento

### **3. Principales Cuellos de Botella en React Native**

#### **A. Bridge de JavaScript a Nativo**
```javascript
// ❌ PROBLEMA: Llamadas frecuentes al bridge
const updatePosition = (x, y) => {
  // Cada llamada cruza el bridge JS → Nativo
  NativeModules.Location.updatePosition(x, y);
};

// ✅ SOLUCIÓN: Batch de operaciones
const updatePositionBatch = (positions) => {
  // Una sola llamada con múltiples posiciones
  NativeModules.Location.updatePositionBatch(positions);
};
```

#### **B. Re-renders Innecesarios**
```javascript
// ❌ PROBLEMA: Componente se re-renderiza innecesariamente
const BadComponent = ({ data }) => {
  // Se re-renderiza cada vez que data cambia
  return <Text>{data.value}</Text>;
};

// ✅ SOLUCIÓN: Memoización
const GoodComponent = React.memo(({ data }) => {
  // Solo se re-renderiza si data realmente cambia
  return <Text>{data.value}</Text>;
});
```

#### **C. Operaciones en el Hilo Principal**
```javascript
// ❌ PROBLEMA: Operaciones pesadas en el hilo principal
const heavyOperation = () => {
  // Esto bloquea la UI
  const result = complexCalculation();
  setResult(result);
};

// ✅ SOLUCIÓN: Usar InteractionManager
const heavyOperation = () => {
  InteractionManager.runAfterInteractions(() => {
    // Se ejecuta después de que las animaciones terminen
    const result = complexCalculation();
    setResult(result);
  });
};
```

### **4. Herramientas de Profiling**

#### **A. Flipper (Recomendado)**
```bash
# Instalación
npm install --save-dev flipper-plugin-react-native-performance

# Configuración en metro.config.js
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
    }),
  },
};
```

#### **B. React DevTools Profiler**
```javascript
import { Profiler } from 'react';

const onRenderCallback = (id, phase, actualDuration) => {
  console.log(`Componente ${id} tomó ${actualDuration}ms en ${phase}`);
};

const MyComponent = () => (
  <Profiler id="MyComponent" onRender={onRenderCallback}>
    <ExpensiveComponent />
  </Profiler>
);
```

#### **C. Performance Monitor de React Native**
```javascript
import { PerformanceObserver } from 'react-native';

const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});

observer.observe({ entryTypes: ['measure'] });
```

### **5. Métricas Clave de Performance**

#### **A. FPS (Frames Per Second)**
```javascript
// Monitorear FPS
import { PerformanceObserver } from 'react-native';

const fpsObserver = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    if (entry.entryType === 'measure') {
      const fps = 1000 / entry.duration;
      console.log(`FPS actual: ${fps.toFixed(2)}`);
    }
  });
});
```

#### **B. Tiempo de Renderizado**
```javascript
// Medir tiempo de renderizado
const measureRenderTime = () => {
  const start = performance.now();
  
  // Renderizar componente
  renderComponent();
  
  const end = performance.now();
  console.log(`Renderizado en: ${end - start}ms`);
};
```

#### **C. Uso de Memoria**
```javascript
// Monitorear uso de memoria
import { PerformanceObserver } from 'react-native';

const memoryObserver = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    if (entry.entryType === 'memory') {
      console.log(`Memoria usada: ${entry.usedJSHeapSize} bytes`);
    }
  });
});
```

---

## 💻 Ejercicios Prácticos

### **Ejercicio 1: Identificar Cuellos de Botella**
Crea una aplicación simple con múltiples componentes y usa las herramientas de profiling para identificar cuellos de botella.

```javascript
// Componente con performance issues intencionales
const SlowComponent = () => {
  const [data, setData] = useState([]);
  
  // Simular operación pesada
  const generateData = () => {
    const newData = [];
    for (let i = 0; i < 10000; i++) {
      newData.push({ id: i, value: Math.random() });
    }
    setData(newData);
  };
  
  return (
    <View>
      <Button title="Generar Datos" onPress={generateData} />
      <FlatList
        data={data}
        renderItem={({ item }) => <Text>{item.value}</Text>}
        keyExtractor={(item) => item.id.toString()}
      />
    </View>
  );
};
```

**Tarea**: Usa Flipper para identificar cuánto tiempo toma generar los datos y renderizar la lista.

### **Ejercicio 2: Implementar Memoización Básica**
Crea un componente que se re-renderice innecesariamente y aplícale memoización.

```javascript
// Componente sin memoización
const ExpensiveComponent = ({ data, onPress }) => {
  console.log('ExpensiveComponent se re-renderizó');
  
  // Simular cálculo costoso
  const expensiveValue = useMemo(() => {
    return data.reduce((acc, item) => acc + item.value, 0);
  }, [data]);
  
  return (
    <TouchableOpacity onPress={onPress}>
      <Text>Valor: {expensiveValue}</Text>
    </TouchableOpacity>
  );
};

// Componente padre que causa re-renders
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  const [data] = useState([{ value: 1 }, { value: 2 }]);
  
  return (
    <View>
      <Text>Contador: {count}</Text>
      <Button title="Incrementar" onPress={() => setCount(c => c + 1)} />
      <ExpensiveComponent data={data} onPress={() => {}} />
    </View>
  );
};
```

**Tarea**: Aplica `React.memo` al `ExpensiveComponent` y observa la diferencia en el número de re-renders.

### **Ejercicio 3: Medir Performance de Listas**
Crea una lista con diferentes implementaciones y mide su performance.

```javascript
// Lista simple (puede ser lenta)
const SimpleList = ({ items }) => (
  <ScrollView>
    {items.map((item) => (
      <Text key={item.id}>{item.text}</Text>
    ))}
  </ScrollView>
);

// Lista optimizada con FlatList
const OptimizedList = ({ items }) => (
  <FlatList
    data={items}
    renderItem={({ item }) => <Text>{item.text}</Text>}
    keyExtractor={(item) => item.id.toString()}
    removeClippedSubviews={true}
    maxToRenderPerBatch={10}
    windowSize={10}
  />
);
```

**Tarea**: Crea 1000 items y compara el performance entre ambas implementaciones.

### **Ejercicio 4: Optimizar Imágenes**
Implementa diferentes estrategias de carga de imágenes y mide su impacto.

```javascript
// Imagen sin optimización
const BasicImage = ({ uri }) => (
  <Image source={{ uri }} style={{ width: 200, height: 200 }} />
);

// Imagen optimizada
const OptimizedImage = ({ uri }) => (
  <Image
    source={{ uri }}
    style={{ width: 200, height: 200 }}
    resizeMode="cover"
    fadeDuration={0}
    loadingIndicatorSource={require('./placeholder.png')}
  />
);

// Imagen con lazy loading
const LazyImage = ({ uri }) => {
  const [isVisible, setIsVisible] = useState(false);
  
  return (
    <View style={{ width: 200, height: 200 }}>
      {isVisible ? (
        <Image source={{ uri }} style={{ width: 200, height: 200 }} />
      ) : (
        <View style={{ width: 200, height: 200, backgroundColor: '#ccc' }} />
      )}
    </View>
  );
};
```

**Tarea**: Mide el tiempo de carga y el uso de memoria de cada implementación.

### **Ejercicio 5: Crear Dashboard de Performance**
Implementa un sistema de monitoreo de performance en tiempo real.

```javascript
const PerformanceDashboard = () => {
  const [metrics, setMetrics] = useState({
    fps: 0,
    memory: 0,
    renderTime: 0
  });
  
  useEffect(() => {
    const interval = setInterval(() => {
      // Simular métricas de performance
      setMetrics({
        fps: Math.random() * 60 + 30,
        memory: Math.random() * 100 + 50,
        renderTime: Math.random() * 16 + 8
      });
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return (
    <View style={styles.dashboard}>
      <Text style={styles.metric}>FPS: {metrics.fps.toFixed(1)}</Text>
      <Text style={styles.metric}>Memoria: {metrics.memory.toFixed(1)}%</Text>
      <Text style={styles.metric}>Render: {metrics.renderTime.toFixed(1)}ms</Text>
    </View>
  );
};
```

**Tarea**: Integra métricas reales usando las herramientas de profiling.

---

## 🎯 Proyecto Integrador

### **App de Performance Monitoring**

Crea una aplicación que demuestre diferentes técnicas de optimización:

#### **Funcionalidades Requeridas:**
1. **Dashboard de Métricas**: Muestra FPS, memoria y tiempo de renderizado
2. **Lista Optimizada**: Implementa FlatList con todas las optimizaciones
3. **Componentes Memoizados**: Usa React.memo y useMemo apropiadamente
4. **Lazy Loading**: Implementa carga diferida de imágenes y componentes
5. **Profiling**: Integra herramientas de medición de performance

#### **Requisitos Técnicos:**
- **Métricas en Tiempo Real**: Actualización cada segundo
- **Comparación de Performance**: Muestra versión optimizada vs no optimizada
- **Alertas**: Notifica cuando la performance cae por debajo de umbrales
- **Historial**: Guarda métricas para análisis posterior

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Native Performance](https://reactnative.dev/docs/performance)
- [Flipper Documentation](https://fbflipper.com/)
- [React DevTools Profiler](https://react.dev/learn/react-developer-tools#profiler)

### **Artículos Recomendados:**
- "Performance Optimization in React Native"
- "Profiling React Native Apps"
- "Memory Management Best Practices"

---

## 🎓 Próximos Pasos

### **En la Siguiente Clase Aprenderás:**
- **Optimización de Componentes**: Técnicas avanzadas de memoización
- **Optimización de Listas**: FlatList, VirtualizedList y optimizaciones
- **Optimización de Imágenes**: Lazy loading, caching y compresión
- **Optimización de Navegación**: Lazy loading de pantallas y transiciones

---

**🎯 Objetivo de la Clase**: Comprender los fundamentos de performance en React Native y aprender a identificar y medir cuellos de botella básicos.

**💡 Consejo**: Siempre mide antes de optimizar. El profiling te mostrará exactamente dónde están los problemas reales de performance.
