# 🔧 Clase 3: Hermes Engine y Optimización

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase dominarás Hermes Engine, el motor JavaScript optimizado de React Native. Aprenderás a configurarlo, optimizarlo y aprovechar al máximo sus capacidades para mejorar significativamente el rendimiento de tu aplicación.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Entender** los fundamentos de Hermes Engine
2. **Configurar** Hermes en proyectos React Native
3. **Optimizar** JavaScript para Hermes
4. **Gestionar** memoria y garbage collection
5. **Realizar** benchmarks y comparaciones de performance
6. **Implementar** mejores prácticas para Hermes

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Fundamentos de Hermes Engine**

#### **¿Qué es Hermes?**
Hermes es un motor JavaScript optimizado desarrollado por Facebook específicamente para React Native. Está diseñado para mejorar el rendimiento, reducir el uso de memoria y acelerar el tiempo de inicio de las aplicaciones.

#### **Características Principales**
- **Bytecode Precompilation**: Compila JavaScript a bytecode
- **Memory Efficiency**: Optimizado para uso eficiente de memoria
- **Fast Startup**: Tiempo de inicio más rápido
- **Small Binary Size**: Tamaño de binario reducido
- **Garbage Collection**: GC optimizado para móviles

#### **Arquitectura de Hermes**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   JavaScript    │───►│   Hermes Parser │───►│   Bytecode      │
│   Source Code   │    │                 │    │   Generator     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Execution     │◄───│   Hermes VM     │◄───│   Bytecode      │
│   Engine        │    │                 │    │   Interpreter   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### **2. Configuración de Hermes**

#### **Habilitar Hermes en Android**
```gradle
// android/app/build.gradle
android {
    compileSdkVersion rootProject.ext.compileSdkVersion

    defaultConfig {
        // ... existing config
        buildConfigField "boolean", "HERMES_ENABLED", "true"
    }
}

dependencies {
    // ... existing dependencies
    if (enableHermes) {
        implementation("com.facebook.react:hermes-engine:+")
    }
}
```

#### **Habilitar Hermes en iOS**
```ruby
# ios/Podfile
def use_hermes!
  pod 'hermes-engine', :path => '../node_modules/react-native/sdks/hermes'
  pod 'React-Core/Hermes', :path => '../node_modules/react-native'
end

# En el target
target 'MyApp' do
  use_hermes!
  # ... other pods
end
```

#### **Configuración en metro.config.js**
```javascript
// metro.config.js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');

const config = {
  transformer: {
    hermesParser: true,
    minifierConfig: {
      keep_fnames: true,
      mangle: {
        keep_fnames: true,
      },
    },
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

### **3. Optimizaciones de JavaScript para Hermes**

#### **Bytecode Precompilation**
```javascript
// Configuración para precompilación
const hermesConfig = {
  // Habilitar precompilación
  enableHermes: true,
  
  // Configuración de bytecode
  bytecode: {
    // Optimizaciones de bytecode
    optimize: true,
    
    // Configuración de GC
    gc: {
      type: 'concurrent',
      threshold: 0.5,
    },
  },
};
```

#### **Optimizaciones de Código**
```javascript
// ❌ Código no optimizado para Hermes
function inefficientCode() {
  // Crear objetos innecesariamente
  const obj = {};
  for (let i = 0; i < 1000; i++) {
    obj[`key${i}`] = i;
  }
  return obj;
}

// ✅ Código optimizado para Hermes
function optimizedCode() {
  // Usar arrays cuando sea posible
  const arr = new Array(1000);
  for (let i = 0; i < 1000; i++) {
    arr[i] = i;
  }
  return arr;
}
```

#### **Gestión de Memoria**
```javascript
// Gestión eficiente de memoria
class MemoryEfficientClass {
  constructor() {
    // Reutilizar objetos
    this.reusableObject = {};
    this.objectPool = [];
  }

  // Reutilizar objetos en lugar de crear nuevos
  getReusableObject() {
    if (this.objectPool.length > 0) {
      return this.objectPool.pop();
    }
    return {};
  }

  // Devolver objetos al pool
  returnObject(obj) {
    // Limpiar el objeto
    Object.keys(obj).forEach(key => delete obj[key]);
    this.objectPool.push(obj);
  }
}
```

### **4. Garbage Collection y Memory Management**

#### **Configuración de GC**
```javascript
// Configuración de Garbage Collection
const gcConfig = {
  // Tipo de GC
  type: 'concurrent', // 'concurrent' o 'stop-the-world'
  
  // Umbral de memoria
  threshold: 0.5, // 50% de memoria usada
  
  // Configuración de heap
  heap: {
    initial: 16 * 1024 * 1024, // 16MB
    maximum: 64 * 1024 * 1024, // 64MB
  },
};
```

#### **Monitoreo de Memoria**
```javascript
// Monitoreo de memoria en tiempo real
class MemoryMonitor {
  constructor() {
    this.memoryStats = [];
    this.startMonitoring();
  }

  startMonitoring() {
    setInterval(() => {
      const stats = this.getMemoryStats();
      this.memoryStats.push(stats);
      this.logMemoryUsage(stats);
    }, 1000);
  }

  getMemoryStats() {
    if (performance.memory) {
      return {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit,
        timestamp: Date.now(),
      };
    }
    return null;
  }

  logMemoryUsage(stats) {
    if (stats) {
      console.log('Memory Usage:', {
        used: `${(stats.used / 1024 / 1024).toFixed(2)} MB`,
        total: `${(stats.total / 1024 / 1024).toFixed(2)} MB`,
        limit: `${(stats.limit / 1024 / 1024).toFixed(2)} MB`,
        usage: `${((stats.used / stats.limit) * 100).toFixed(2)}%`,
      });
    }
  }
}
```

#### **Optimización de Objetos**
```javascript
// Optimización de creación de objetos
class ObjectOptimizer {
  constructor() {
    // Pool de objetos reutilizables
    this.objectPool = new Map();
  }

  // Crear objetos de forma eficiente
  createObject(type, data) {
    let obj = this.objectPool.get(type);
    
    if (!obj) {
      obj = this.createNewObject(type);
    }
    
    // Reutilizar objeto existente
    this.populateObject(obj, data);
    return obj;
  }

  createNewObject(type) {
    switch (type) {
      case 'user':
        return { id: null, name: null, email: null };
      case 'product':
        return { id: null, title: null, price: null };
      default:
        return {};
    }
  }

  populateObject(obj, data) {
    Object.keys(data).forEach(key => {
      obj[key] = data[key];
    });
  }

  // Devolver objeto al pool
  returnObject(obj, type) {
    this.objectPool.set(type, obj);
  }
}
```

### **5. Performance Benchmarks**

#### **Benchmark de Tiempo de Inicio**
```javascript
// Benchmark de tiempo de inicio
class StartupBenchmark {
  constructor() {
    this.startTime = Date.now();
    this.milestones = [];
  }

  markMilestone(name) {
    const currentTime = Date.now();
    const elapsed = currentTime - this.startTime;
    
    this.milestones.push({
      name,
      elapsed,
      timestamp: currentTime,
    });
    
    console.log(`Milestone ${name}: ${elapsed}ms`);
  }

  getResults() {
    return {
      totalTime: Date.now() - this.startTime,
      milestones: this.milestones,
    };
  }
}

// Uso del benchmark
const benchmark = new StartupBenchmark();

// Marcar hitos importantes
benchmark.markMilestone('App Start');
benchmark.markMilestone('Bundle Loaded');
benchmark.markMilestone('First Render');
benchmark.markMilestone('Navigation Ready');
```

#### **Benchmark de Rendimiento**
```javascript
// Benchmark de rendimiento
class PerformanceBenchmark {
  constructor() {
    this.results = [];
  }

  // Benchmark de operaciones
  benchmark(name, operation, iterations = 1000) {
    const startTime = performance.now();
    
    for (let i = 0; i < iterations; i++) {
      operation();
    }
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    const average = duration / iterations;
    
    const result = {
      name,
      iterations,
      totalTime: duration,
      averageTime: average,
      operationsPerSecond: 1000 / average,
    };
    
    this.results.push(result);
    console.log(`Benchmark ${name}:`, result);
    
    return result;
  }

  // Comparar resultados
  compareResults() {
    const sorted = this.results.sort((a, b) => a.averageTime - b.averageTime);
    
    console.log('Performance Comparison:');
    sorted.forEach((result, index) => {
      console.log(`${index + 1}. ${result.name}: ${result.averageTime.toFixed(4)}ms`);
    });
  }
}

// Ejemplo de uso
const benchmark = new PerformanceBenchmark();

// Benchmark de diferentes implementaciones
benchmark.benchmark('Array Push', () => {
  const arr = [];
  for (let i = 0; i < 100; i++) {
    arr.push(i);
  }
});

benchmark.benchmark('Array Concat', () => {
  let arr = [];
  for (let i = 0; i < 100; i++) {
    arr = arr.concat([i]);
  }
});

benchmark.compareResults();
```

### **6. Mejores Prácticas para Hermes**

#### **Optimización de Imports**
```javascript
// ❌ Imports no optimizados
import * as React from 'react';
import {View, Text, StyleSheet} from 'react-native';
import {useState, useEffect, useCallback, useMemo} from 'react';

// ✅ Imports optimizados para Hermes
import React, {useState, useEffect, useCallback, useMemo} from 'react';
import {View, Text, StyleSheet} from 'react-native';
```

#### **Optimización de Funciones**
```javascript
// ❌ Función no optimizada
function inefficientFunction() {
  const result = [];
  for (let i = 0; i < 1000; i++) {
    result.push(i * 2);
  }
  return result;
}

// ✅ Función optimizada para Hermes
function optimizedFunction() {
  const result = new Array(1000);
  for (let i = 0; i < 1000; i++) {
    result[i] = i * 2;
  }
  return result;
}
```

#### **Optimización de Objetos**
```javascript
// ❌ Creación ineficiente de objetos
function createUser(name, email) {
  return {
    id: Date.now(),
    name: name,
    email: email,
    createdAt: new Date(),
  };
}

// ✅ Creación eficiente de objetos
const userFactory = {
  create(name, email) {
    const user = this.getReusableUser();
    user.id = Date.now();
    user.name = name;
    user.email = email;
    user.createdAt = new Date();
    return user;
  },

  getReusableUser() {
    if (this.userPool.length > 0) {
      return this.userPool.pop();
    }
    return { id: null, name: null, email: null, createdAt: null };
  },

  userPool: [],
};
```

---

## 🛠️ Contenido Práctico

### **Ejercicio 1: Configuración de Hermes**

#### **Objetivo**
Configurar Hermes en un proyecto React Native existente.

#### **Pasos**
1. **Habilitar Hermes** en Android
2. **Habilitar Hermes** en iOS
3. **Configurar Metro** para Hermes
4. **Verificar** que Hermes esté funcionando

#### **Código de Ejemplo**
```javascript
// App.js
import React, {useEffect} from 'react';
import {View, Text, Button} from 'react-native';

export default function App() {
  useEffect(() => {
    // Verificar si Hermes está habilitado
    if (global.HermesInternal) {
      console.log('Hermes is enabled');
    } else {
      console.log('Hermes is not enabled');
    }
  }, []);

  return (
    <View style={{flex: 1, justifyContent: 'center', alignItems: 'center'}}>
      <Text>Hermes Engine Test</Text>
      <Button title="Test Performance" onPress={testPerformance} />
    </View>
  );
}

function testPerformance() {
  const startTime = performance.now();
  
  // Operación costosa
  const result = [];
  for (let i = 0; i < 100000; i++) {
    result.push(i * 2);
  }
  
  const endTime = performance.now();
  console.log(`Performance test: ${endTime - startTime}ms`);
}
```

### **Ejercicio 2: Memory Monitor**

#### **Objetivo**
Implementar un monitor de memoria para Hermes.

#### **Código de Ejemplo**
```javascript
// MemoryMonitor.js
import React, {useState, useEffect} from 'react';
import {View, Text, StyleSheet} from 'react-native';

export default function MemoryMonitor() {
  const [memoryStats, setMemoryStats] = useState(null);

  useEffect(() => {
    const interval = setInterval(() => {
      if (performance.memory) {
        const stats = {
          used: performance.memory.usedJSHeapSize,
          total: performance.memory.totalJSHeapSize,
          limit: performance.memory.jsHeapSizeLimit,
        };
        setMemoryStats(stats);
      }
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  if (!memoryStats) {
    return <Text>Memory monitoring not available</Text>;
  }

  const usagePercentage = (memoryStats.used / memoryStats.limit) * 100;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Memory Usage</Text>
      <Text>Used: {(memoryStats.used / 1024 / 1024).toFixed(2)} MB</Text>
      <Text>Total: {(memoryStats.total / 1024 / 1024).toFixed(2)} MB</Text>
      <Text>Limit: {(memoryStats.limit / 1024 / 1024).toFixed(2)} MB</Text>
      <Text>Usage: {usagePercentage.toFixed(2)}%</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    backgroundColor: '#f0f0f0',
    margin: 10,
    borderRadius: 8,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
});
```

### **Ejercicio 3: Performance Benchmark**

#### **Objetivo**
Crear un benchmark de rendimiento para comparar diferentes implementaciones.

#### **Código de Ejemplo**
```javascript
// PerformanceBenchmark.js
import React, {useState} from 'react';
import {View, Text, Button, StyleSheet} from 'react-native';

export default function PerformanceBenchmark() {
  const [results, setResults] = useState([]);

  const runBenchmark = (name, operation, iterations = 1000) => {
    const startTime = performance.now();
    
    for (let i = 0; i < iterations; i++) {
      operation();
    }
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    const average = duration / iterations;
    
    const result = {
      name,
      iterations,
      totalTime: duration,
      averageTime: average,
      operationsPerSecond: 1000 / average,
    };
    
    setResults(prev => [...prev, result]);
  };

  const testArrayOperations = () => {
    // Test 1: Array push
    runBenchmark('Array Push', () => {
      const arr = [];
      for (let i = 0; i < 100; i++) {
        arr.push(i);
      }
    });

    // Test 2: Array concat
    runBenchmark('Array Concat', () => {
      let arr = [];
      for (let i = 0; i < 100; i++) {
        arr = arr.concat([i]);
      }
    });

    // Test 3: Array spread
    runBenchmark('Array Spread', () => {
      let arr = [];
      for (let i = 0; i < 100; i++) {
        arr = [...arr, i];
      }
    });
  };

  const testObjectOperations = () => {
    // Test 1: Object creation
    runBenchmark('Object Creation', () => {
      const obj = {};
      for (let i = 0; i < 100; i++) {
        obj[`key${i}`] = i;
      }
    });

    // Test 2: Object.assign
    runBenchmark('Object.assign', () => {
      let obj = {};
      for (let i = 0; i < 100; i++) {
        obj = Object.assign({}, obj, {[`key${i}`]: i});
      }
    });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Performance Benchmark</Text>
      
      <Button title="Test Array Operations" onPress={testArrayOperations} />
      <Button title="Test Object Operations" onPress={testObjectOperations} />
      
      {results.map((result, index) => (
        <View key={index} style={styles.result}>
          <Text style={styles.resultName}>{result.name}</Text>
          <Text>Total: {result.totalTime.toFixed(2)}ms</Text>
          <Text>Average: {result.averageTime.toFixed(4)}ms</Text>
          <Text>Ops/sec: {result.operationsPerSecond.toFixed(0)}</Text>
        </View>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  result: {
    backgroundColor: '#f0f0f0',
    padding: 10,
    margin: 5,
    borderRadius: 5,
  },
  resultName: {
    fontWeight: 'bold',
    fontSize: 16,
  },
});
```

### **Ejercicio 4: Object Pool**

#### **Objetivo**
Implementar un pool de objetos para optimizar la gestión de memoria.

#### **Código de Ejemplo**
```javascript
// ObjectPool.js
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    this.initialSize = initialSize;
    
    // Inicializar pool
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }

  get() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createFn();
  }

  release(obj) {
    if (this.resetFn) {
      this.resetFn(obj);
    }
    this.pool.push(obj);
  }

  getStats() {
    return {
      poolSize: this.pool.length,
      totalCreated: this.initialSize + (this.initialSize - this.pool.length),
    };
  }
}

// Uso del ObjectPool
const userPool = new ObjectPool(
  () => ({ id: null, name: null, email: null }),
  (user) => {
    user.id = null;
    user.name = null;
    user.email = null;
  }
);

// Crear usuario
const user = userPool.get();
user.id = 1;
user.name = 'John Doe';
user.email = 'john@example.com';

// Usar usuario...

// Devolver al pool
userPool.release(user);

console.log('Pool stats:', userPool.getStats());
```

---

## 🎯 Ejercicios de Evaluación

### **Ejercicio 1: Configuración Completa**
- Configurar Hermes en Android e iOS
- Verificar que esté funcionando correctamente
- Medir el tiempo de inicio de la aplicación

### **Ejercicio 2: Memory Optimization**
- Implementar un monitor de memoria
- Identificar memory leaks
- Optimizar el uso de memoria

### **Ejercicio 3: Performance Benchmark**
- Crear benchmarks para operaciones comunes
- Comparar rendimiento con y sin Hermes
- Implementar optimizaciones basadas en los resultados

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [Hermes Documentation](https://hermesengine.dev/)
- [React Native Hermes](https://reactnative.dev/docs/hermes)
- [Hermes GitHub](https://github.com/facebook/hermes)

### **Herramientas Útiles**
- [Hermes CLI](https://hermesengine.dev/docs/cli)
- [Hermes Debugger](https://hermesengine.dev/docs/debugging)

### **Mejores Prácticas**
- Habilitar Hermes desde el inicio del proyecto
- Optimizar código para Hermes
- Monitorear memoria regularmente
- Realizar benchmarks de rendimiento

---

## 🚀 Siguiente Clase

En la próxima clase aprenderás sobre **Debugging Nativo**, donde profundizarás en el debugging de aplicaciones React Native en iOS y Android usando herramientas nativas.

---

**💡 Consejo**: Hermes puede mejorar significativamente el rendimiento de tu aplicación. Invierte tiempo en entender sus optimizaciones y configurarlo correctamente.
