# Clase 4: Testing de Rendimiento 🚀

## Objetivos de la Clase
- Implementar testing de rendimiento para aplicaciones React Native
- Medir y optimizar métricas de velocidad y memoria
- Testing de rendimiento en diferentes dispositivos y condiciones
- Análisis de bottlenecks y optimizaciones

## Contenido Teórico

### 1. Métricas de Rendimiento

#### Métricas Clave de Rendimiento
```javascript
// src/utils/performanceMetrics.js
export class PerformanceMetrics {
  constructor() {
    this.metrics = {
      appLaunch: 0,
      screenLoad: {},
      memoryUsage: [],
      networkLatency: [],
      frameRate: [],
      batteryUsage: 0,
    };
  }

  // Medir tiempo de lanzamiento de la app
  startAppLaunch() {
    this.metrics.appLaunch = performance.now();
  }

  endAppLaunch() {
    const launchTime = performance.now() - this.metrics.appLaunch;
    console.log(`App Launch Time: ${launchTime}ms`);
    return launchTime;
  }

  // Medir tiempo de carga de pantalla
  startScreenLoad(screenName) {
    this.metrics.screenLoad[screenName] = performance.now();
  }

  endScreenLoad(screenName) {
    if (this.metrics.screenLoad[screenName]) {
      const loadTime = performance.now() - this.metrics.screenLoad[screenName];
      console.log(`${screenName} Load Time: ${loadTime}ms`);
      return loadTime;
    }
    return 0;
  }

  // Medir uso de memoria
  measureMemoryUsage() {
    if (global.performance && global.performance.memory) {
      const memory = global.performance.memory;
      const memoryInfo = {
        used: memory.usedJSHeapSize,
        total: memory.totalJSHeapSize,
        limit: memory.jsHeapSizeLimit,
        timestamp: Date.now(),
      };
      
      this.metrics.memoryUsage.push(memoryInfo);
      console.log('Memory Usage:', memoryInfo);
      return memoryInfo;
    }
    return null;
  }

  // Medir latencia de red
  async measureNetworkLatency(url) {
    const startTime = performance.now();
    
    try {
      const response = await fetch(url);
      const endTime = performance.now();
      const latency = endTime - startTime;
      
      this.metrics.networkLatency.push({
        url,
        latency,
        status: response.status,
        timestamp: Date.now(),
      });
      
      console.log(`Network Latency for ${url}: ${latency}ms`);
      return latency;
    } catch (error) {
      console.error('Network latency measurement failed:', error);
      return -1;
    }
  }

  // Medir frame rate
  startFrameRateMeasurement() {
    let frameCount = 0;
    let lastTime = performance.now();
    
    const measureFrame = () => {
      frameCount++;
      const currentTime = performance.now();
      
      if (currentTime - lastTime >= 1000) { // Cada segundo
        const fps = Math.round((frameCount * 1000) / (currentTime - lastTime));
        this.metrics.frameRate.push({
          fps,
          timestamp: Date.now(),
        });
        
        console.log(`Frame Rate: ${fps} FPS`);
        frameCount = 0;
        lastTime = currentTime;
      }
      
      requestAnimationFrame(measureFrame);
    };
    
    requestAnimationFrame(measureFrame);
  }

  // Medir uso de batería (solo disponible en algunos dispositivos)
  measureBatteryUsage() {
    if (navigator.getBattery) {
      navigator.getBattery().then(battery => {
        this.metrics.batteryUsage = battery.level * 100;
        console.log(`Battery Level: ${this.metrics.batteryUsage}%`);
      });
    }
  }

  // Obtener métricas resumidas
  getMetricsSummary() {
    const summary = {
      appLaunch: this.metrics.appLaunch,
      averageScreenLoad: this.calculateAverageScreenLoad(),
      memoryTrend: this.calculateMemoryTrend(),
      averageNetworkLatency: this.calculateAverageNetworkLatency(),
      averageFrameRate: this.calculateAverageFrameRate(),
      batteryUsage: this.metrics.batteryUsage,
    };
    
    return summary;
  }

  // Calcular promedios
  calculateAverageScreenLoad() {
    const times = Object.values(this.metrics.screenLoad);
    return times.length > 0 ? times.reduce((a, b) => a + b, 0) / times.length : 0;
  }

  calculateMemoryTrend() {
    if (this.metrics.memoryUsage.length < 2) return 'insufficient_data';
    
    const recent = this.metrics.memoryUsage.slice(-5);
    const trend = recent.reduce((acc, curr, i, arr) => {
      if (i === 0) return 0;
      return acc + (curr.used - arr[i - 1].used);
    }, 0) / (recent.length - 1);
    
    if (trend > 1000000) return 'increasing';
    if (trend < -1000000) return 'decreasing';
    return 'stable';
  }

  calculateAverageNetworkLatency() {
    const latencies = this.metrics.networkLatency.map(l => l.latency);
    return latencies.length > 0 ? latencies.reduce((a, b) => a + b, 0) / latencies.length : 0;
  }

  calculateAverageFrameRate() {
    const fpsValues = this.metrics.frameRate.map(f => f.fps);
    return fpsValues.length > 0 ? fpsValues.reduce((a, b) => a + b, 0) / fpsValues.length : 0;
  }

  // Exportar métricas
  exportMetrics() {
    return JSON.stringify(this.metrics, null, 2);
  }

  // Limpiar métricas
  clearMetrics() {
    this.metrics = {
      appLaunch: 0,
      screenLoad: {},
      memoryUsage: [],
      networkLatency: [],
      frameRate: [],
      batteryUsage: 0,
    };
  }
}

// Instancia global
export const performanceMetrics = new PerformanceMetrics();
```

#### Hook de Rendimiento
```javascript
// src/hooks/usePerformance.js
import { useEffect, useRef, useCallback } from 'react';
import { performanceMetrics } from '../utils/performanceMetrics';

export const usePerformance = (screenName) => {
  const startTimeRef = useRef(null);
  const memoryIntervalRef = useRef(null);

  // Iniciar medición de pantalla
  const startScreenMeasurement = useCallback(() => {
    performanceMetrics.startScreenLoad(screenName);
    startTimeRef.current = performance.now();
  }, [screenName]);

  // Finalizar medición de pantalla
  const endScreenMeasurement = useCallback(() => {
    if (startTimeRef.current) {
      const loadTime = performanceMetrics.endScreenLoad(screenName);
      return loadTime;
    }
    return 0;
  }, [screenName]);

  // Iniciar monitoreo de memoria
  const startMemoryMonitoring = useCallback((intervalMs = 5000) => {
    memoryIntervalRef.current = setInterval(() => {
      performanceMetrics.measureMemoryUsage();
    }, intervalMs);
  }, []);

  // Detener monitoreo de memoria
  const stopMemoryMonitoring = useCallback(() => {
    if (memoryIntervalRef.current) {
      clearInterval(memoryIntervalRef.current);
      memoryIntervalRef.current = null;
    }
  }, []);

  // Medir rendimiento de función
  const measureFunctionPerformance = useCallback((fn, functionName) => {
    const startTime = performance.now();
    const result = fn();
    const endTime = performance.now();
    const executionTime = endTime - startTime;
    
    console.log(`${functionName} execution time: ${executionTime}ms`);
    return { result, executionTime };
  }, []);

  // Medir rendimiento de función asíncrona
  const measureAsyncFunctionPerformance = useCallback(async (fn, functionName) => {
    const startTime = performance.now();
    const result = await fn();
    const endTime = performance.now();
    const executionTime = endTime - startTime;
    
    console.log(`${functionName} execution time: ${executionTime}ms`);
    return { result, executionTime };
  }, []);

  useEffect(() => {
    // Iniciar medición cuando se monta la pantalla
    startScreenMeasurement();
    
    // Iniciar monitoreo de memoria
    startMemoryMonitoring();
    
    // Iniciar medición de frame rate
    performanceMetrics.startFrameRateMeasurement();
    
    return () => {
      // Finalizar medición cuando se desmonta
      endScreenMeasurement();
      stopMemoryMonitoring();
    };
  }, [startScreenMeasurement, endScreenMeasurement, startMemoryMonitoring, stopMemoryMonitoring]);

  return {
    startScreenMeasurement,
    endScreenMeasurement,
    startMemoryMonitoring,
    stopMemoryMonitoring,
    measureFunctionPerformance,
    measureAsyncFunctionPerformance,
    getMetrics: () => performanceMetrics.getMetricsSummary(),
  };
};
```

### 2. Testing de Rendimiento de Componentes

#### Componente con Testing de Rendimiento
```javascript
// src/components/PerformanceTestComponent.js
import React, { useState, useCallback, useMemo } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  FlatList,
  StyleSheet,
  Alert,
} from 'react-native';
import { usePerformance } from '../hooks/usePerformance';

export const PerformanceTestComponent = () => {
  const [items, setItems] = useState([]);
  const [filter, setFilter] = useState('all');
  const [sortBy, setSortBy] = useState('name');
  
  const { measureFunctionPerformance, getMetrics } = usePerformance('PerformanceTest');

  // Generar datos de prueba
  const generateTestData = useCallback((count) => {
    return measureFunctionPerformance(() => {
      const data = [];
      for (let i = 0; i < count; i++) {
        data.push({
          id: i,
          name: `Item ${i}`,
          description: `Description for item ${i}`,
          value: Math.random() * 1000,
          category: i % 3 === 0 ? 'A' : i % 3 === 1 ? 'B' : 'C',
        });
      }
      return data;
    }, 'generateTestData');
  }, [measureFunctionPerformance]);

  // Filtrar items
  const filteredItems = useMemo(() => {
    return measureFunctionPerformance(() => {
      if (filter === 'all') return items;
      return items.filter(item => item.category === filter);
    }, 'filterItems');
  }, [items, filter, measureFunctionPerformance]);

  // Ordenar items
  const sortedItems = useMemo(() => {
    return measureFunctionPerformance(() => {
      return [...filteredItems].sort((a, b) => {
        if (sortBy === 'name') return a.name.localeCompare(b.name);
        if (sortBy === 'value') return a.value - b.value;
        return 0;
      });
    }, 'sortItems');
  }, [filteredItems, sortBy, measureFunctionPerformance]);

  // Agregar items
  const addItems = useCallback(() => {
    const { result } = generateTestData(1000);
    setItems(prev => [...prev, ...result]);
  }, [generateTestData]);

  // Limpiar items
  const clearItems = useCallback(() => {
    setItems([]);
  }, []);

  // Mostrar métricas
  const showMetrics = useCallback(() => {
    const metrics = getMetrics();
    Alert.alert(
      'Performance Metrics',
      `App Launch: ${metrics.appLaunch}ms\n` +
      `Screen Load: ${metrics.averageScreenLoad}ms\n` +
      `Memory Trend: ${metrics.memoryTrend}\n` +
      `Network Latency: ${metrics.averageNetworkLatency}ms\n` +
      `Frame Rate: ${metrics.averageFrameRate} FPS\n` +
      `Battery: ${metrics.batteryUsage}%`
    );
  }, [getMetrics]);

  // Renderizar item
  const renderItem = useCallback(({ item }) => (
    <View style={styles.item}>
      <Text style={styles.itemName}>{item.name}</Text>
      <Text style={styles.itemDescription}>{item.description}</Text>
      <Text style={styles.itemValue}>Value: {item.value.toFixed(2)}</Text>
      <Text style={styles.itemCategory}>Category: {item.category}</Text>
    </View>
  ), []);

  // Key extractor optimizado
  const keyExtractor = useCallback((item) => item.id.toString(), []);

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Performance Test</Text>
        <Text style={styles.subtitle}>Items: {items.length}</Text>
      </View>

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={addItems}>
          <Text style={styles.buttonText}>Add 1000 Items</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={clearItems}>
          <Text style={styles.buttonText}>Clear All</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={showMetrics}>
          <Text style={styles.buttonText}>Show Metrics</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.filters}>
        <TouchableOpacity
          style={[styles.filterButton, filter === 'all' && styles.activeFilter]}
          onPress={() => setFilter('all')}
        >
          <Text style={styles.filterText}>All</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.filterButton, filter === 'A' && styles.activeFilter]}
          onPress={() => setFilter('A')}
        >
          <Text style={styles.filterText}>Category A</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.filterButton, filter === 'B' && styles.activeFilter]}
          onPress={() => setFilter('B')}
        >
          <Text style={styles.filterText}>Category B</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.filterButton, filter === 'C' && styles.activeFilter]}
          onPress={() => setFilter('C')}
        >
          <Text style={styles.filterText}>Category C</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.sortControls}>
        <TouchableOpacity
          style={[styles.sortButton, sortBy === 'name' && styles.activeSort]}
          onPress={() => setSortBy('name')}
        >
          <Text style={styles.sortText}>Sort by Name</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.sortButton, sortBy === 'value' && styles.activeSort]}
          onPress={() => setSortBy('value')}
        >
          <Text style={styles.sortText}>Sort by Value</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={sortedItems}
        renderItem={renderItem}
        keyExtractor={keyExtractor}
        style={styles.list}
        initialNumToRender={10}
        maxToRenderPerBatch={10}
        windowSize={10}
        removeClippedSubviews={true}
        getItemLayout={(data, index) => ({
          length: 80,
          offset: 80 * index,
          index,
        })}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    padding: 20,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
  controls: {
    flexDirection: 'row',
    padding: 15,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 8,
    marginRight: 10,
  },
  buttonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
  filters: {
    flexDirection: 'row',
    padding: 15,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  filterButton: {
    padding: 8,
    borderRadius: 6,
    marginRight: 10,
    backgroundColor: '#f0f0f0',
  },
  activeFilter: {
    backgroundColor: '#007AFF',
  },
  filterText: {
    color: '#333',
  },
  sortControls: {
    flexDirection: 'row',
    padding: 15,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  sortButton: {
    padding: 8,
    borderRadius: 6,
    marginRight: 10,
    backgroundColor: '#f0f0f0',
  },
  activeSort: {
    backgroundColor: '#28a745',
  },
  sortText: {
    color: '#333',
  },
  list: {
    flex: 1,
  },
  item: {
    backgroundColor: '#fff',
    padding: 15,
    marginHorizontal: 15,
    marginVertical: 5,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  itemName: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  itemDescription: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
  itemValue: {
    fontSize: 14,
    color: '#007AFF',
    marginBottom: 5,
  },
  itemCategory: {
    fontSize: 12,
    color: '#999',
  },
});
```

### 3. Testing de Rendimiento con Jest

#### Tests de Rendimiento
```javascript
// src/components/__tests__/PerformanceTestComponent.performance.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { PerformanceTestComponent } from '../PerformanceTestComponent';

// Mock de performance API
global.performance = {
  now: jest.fn(() => Date.now()),
  memory: {
    usedJSHeapSize: 1000000,
    totalJSHeapSize: 2000000,
    jsHeapSizeLimit: 10000000,
  },
};

// Mock de requestAnimationFrame
global.requestAnimationFrame = jest.fn((cb) => setTimeout(cb, 16));

describe('PerformanceTestComponent Performance Tests', () => {
  beforeEach(() => {
    jest.clearAllMocks();
    // Reset performance.now
    global.performance.now.mockReturnValue(0);
  });

  describe('Rendering Performance', () => {
    it('debe renderizar inicialmente en menos de 100ms', async () => {
      const startTime = performance.now();
      
      const { getByText } = render(<PerformanceTestComponent />);
      
      // Esperar a que se renderice
      await waitFor(() => {
        expect(getByText('Performance Test')).toBeTruthy();
      });
      
      const renderTime = performance.now() - startTime;
      expect(renderTime).toBeLessThan(100);
    });

    it('debe manejar 1000 items eficientemente', async () => {
      const { getByText, getByTestId } = render(<PerformanceTestComponent />);
      
      // Simular tiempo para agregar items
      global.performance.now
        .mockReturnValueOnce(0)    // start
        .mockReturnValueOnce(50);  // end
      
      const addButton = getByText('Add 1000 Items');
      fireEvent.press(addButton);
      
      // Verificar que se agregaron los items
      await waitFor(() => {
        expect(getByText('Items: 1000')).toBeTruthy();
      });
      
      // Verificar que la lista se renderiza
      await waitFor(() => {
        expect(getByText('Item 0')).toBeTruthy();
        expect(getByText('Item 999')).toBeTruthy();
      });
    });
  });

  describe('Memory Usage', () => {
    it('debe monitorear uso de memoria', async () => {
      const { getByText } = render(<PerformanceTestComponent />);
      
      // Simular medición de memoria
      global.performance.memory.usedJSHeapSize = 1500000;
      
      const metricsButton = getByText('Show Metrics');
      fireEvent.press(metricsButton);
      
      // Verificar que se muestran las métricas
      // (En un test real, esto verificaría el Alert)
    });
  });

  describe('Filtering Performance', () => {
    it('debe filtrar items eficientemente', async () => {
      const { getByText } = render(<PerformanceTestComponent />);
      
      // Agregar items primero
      const addButton = getByText('Add 1000 Items');
      fireEvent.press(addButton);
      
      await waitFor(() => {
        expect(getByText('Items: 1000')).toBeTruthy();
      });
      
      // Medir tiempo de filtrado
      global.performance.now
        .mockReturnValueOnce(100)  // start filter
        .mockReturnValueOnce(120); // end filter
      
      const categoryAButton = getByText('Category A');
      fireEvent.press(categoryAButton);
      
      // Verificar que se aplicó el filtro
      await waitFor(() => {
        // Solo deberían mostrarse items de categoría A
        expect(getByText('Category: A')).toBeTruthy();
      });
    });
  });

  describe('Sorting Performance', () => {
    it('debe ordenar items eficientemente', async () => {
      const { getByText } = render(<PerformanceTestComponent />);
      
      // Agregar items
      const addButton = getByText('Add 1000 Items');
      fireEvent.press(addButton);
      
      await waitFor(() => {
        expect(getByText('Items: 1000')).toBeTruthy();
      });
      
      // Medir tiempo de ordenamiento
      global.performance.now
        .mockReturnValueOnce(200)  // start sort
        .mockReturnValueOnce(230); // end sort
      
      const sortByNameButton = getByText('Sort by Name');
      fireEvent.press(sortByNameButton);
      
      // Verificar que se ordenó
      await waitFor(() => {
        expect(getByText('Item 0')).toBeTruthy();
      });
    });
  });

  describe('List Performance', () => {
    it('debe usar optimizaciones de FlatList', async () => {
      const { getByText } = render(<PerformanceTestComponent />);
      
      // Agregar muchos items
      const addButton = getByText('Add 1000 Items');
      fireEvent.press(addButton);
      
      await waitFor(() => {
        expect(getByText('Items: 1000')).toBeTruthy();
      });
      
      // Verificar que se renderizan solo los items visibles
      // FlatList debería usar windowing para optimizar
      await waitFor(() => {
        expect(getByText('Item 0')).toBeTruthy();
        expect(getByText('Item 10')).toBeTruthy();
        // Item 999 no debería estar renderizado inicialmente
      });
    });
  });
});
```

### 4. Testing de Rendimiento en Diferentes Dispositivos

#### Configuración de Testing Multi-Dispositivo
```javascript
// e2e/performance/devicePerformance.e2e.js
import { device, element, by, expect } from 'detox';

describe('Device Performance Testing', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  describe('Low-end Device Performance', () => {
    it('debe funcionar en dispositivos de gama baja', async () => {
      // Simular dispositivo de gama baja
      await device.setURLBlacklist(['.*']);
      
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Verificar que se completa en tiempo razonable
      await expect(element(by.text('Welcome'))).toBeVisible();
      
      // Restaurar conexión normal
      await device.setURLBlacklist([]);
    });

    it('debe manejar memoria limitada', async () => {
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Navegar a pantalla con muchos datos
      await element(by.text('Users')).tap();
      
      // Agregar muchos usuarios para probar memoria
      for (let i = 0; i < 100; i++) {
        await element(by.id('add-user-button')).tap();
        await element(by.id('user-name-input')).typeText(`User ${i}`);
        await element(by.id('user-email-input')).typeText(`user${i}@example.com`);
        await element(by.id('save-user-button')).tap();
      }
      
      // Verificar que la app sigue funcionando
      await expect(element(by.text('Users List'))).toBeVisible();
    });
  });

  describe('High-end Device Performance', () => {
    it('debe aprovechar hardware de alta gama', async () => {
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Navegar rápidamente entre pantallas
      const screens = ['Home', 'Users', 'Profile', 'Settings'];
      
      for (const screen of screens) {
        await element(by.text(screen)).tap();
        await expect(element(by.text(screen))).toBeVisible();
      }
      
      // Verificar que las transiciones son fluidas
      await expect(element(by.text('Home'))).toBeVisible();
    });

    it('debe manejar animaciones complejas', async () => {
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Ir a perfil para probar animaciones
      await element(by.text('Profile')).tap();
      
      // Probar diferentes animaciones
      await element(by.id('profile-image')).tap();
      await element(by.text('Take Photo')).tap();
      
      // Verificar que las animaciones son fluidas
      await expect(element(by.text('Camera'))).toBeVisible();
    });
  });

  describe('Network Performance', () => {
    it('debe manejar conexiones lentas', async () => {
      // Simular conexión lenta
      await device.setURLBlacklist(['.*']);
      
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Verificar que se muestra loading
      await expect(element(by.text('Loading...'))).toBeVisible();
      
      // Restaurar conexión
      await device.setURLBlacklist([]);
      
      // Verificar que se completa
      await expect(element(by.text('Welcome'))).toBeVisible();
    });

    it('debe manejar conexiones inestables', async () => {
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Ir a Users
      await element(by.text('Users')).tap();
      
      // Simular conexión intermitente
      for (let i = 0; i < 5; i++) {
        await device.setURLBlacklist(['.*']);
        await new Promise(resolve => setTimeout(resolve, 1000));
        await device.setURLBlacklist([]);
        await new Promise(resolve => setTimeout(resolve, 1000));
      }
      
      // Verificar que la app sigue funcionando
      await expect(element(by.text('Users List'))).toBeVisible();
    });
  });
});
```

## Ejercicios Prácticos

### Ejercicio 1: Sistema de Métricas de Rendimiento
Crea un sistema completo de métricas de rendimiento:

```javascript
// Sistema a implementar
export class AdvancedPerformanceMetrics {
  constructor() {
    this.metrics = {};
  }

  // Métricas de renderizado
  measureRenderTime(componentName) { /* implementar */ }
  
  // Métricas de memoria
  measureMemoryLeaks() { /* implementar */ }
  
  // Métricas de red
  measureNetworkEfficiency() { /* implementar */ }
  
  // Métricas de batería
  measureBatteryImpact() { /* implementar */ }
  
  // Generar reportes
  generateReport() { /* implementar */ }
}
```

**Requisitos del testing:**
- Test de medición de tiempo de renderizado
- Test de detección de memory leaks
- Test de eficiencia de red
- Test de impacto en batería
- Test de generación de reportes

### Ejercicio 2: Optimización de Listas
Crea un componente de lista optimizado y escribe tests de rendimiento:

```javascript
// Componente a implementar
export const OptimizedList = ({ data, renderItem, keyExtractor }) => {
  // Implementar optimizaciones:
  // - Virtualización
  // - Lazy loading
  // - Memoización de items
  // - Optimización de scroll
  
  return (/* JSX optimizado */);
};
```

**Requisitos del testing:**
- Test de rendimiento con 10,000+ items
- Test de scroll suave
- Test de uso de memoria
- Test de frame rate durante scroll
- Test de carga lazy

### Ejercicio 3: Testing de Rendimiento en CI/CD
Crea un pipeline de testing de rendimiento automatizado:

```javascript
// Pipeline a implementar
describe('CI/CD Performance Pipeline', () => {
  it('debe ejecutar tests de rendimiento en diferentes dispositivos', async () => {
    // 1. Test en iOS simulator
    // 2. Test en Android emulator
    // 3. Test de rendimiento de build
    // 4. Test de métricas de bundle
    // 5. Generar reporte de rendimiento
  });
});
```

**Requisitos del testing:**
- Test automatizado en múltiples dispositivos
- Test de rendimiento de build
- Test de métricas de bundle
- Generación de reportes automáticos
- Alertas por degradación de rendimiento

## Resumen de la Clase

En esta clase hemos aprendido:

1. **Métricas de Rendimiento**: Medición de tiempo, memoria, red y frame rate
2. **Testing de Componentes**: Testing de rendimiento de componentes individuales
3. **Testing con Jest**: Tests de rendimiento automatizados
4. **Testing Multi-Dispositivo**: Testing en diferentes tipos de hardware

## Navegación
- **Anterior**: [Clase 3: Testing E2E con Detox](clase_3_testing_e2e_detox.md)
- **Siguiente**: [Clase 5: Testing Automatizado y CI/CD](clase_5_testing_automatizado_cicd.md)
- **Inicio**: [Índice del Curso](../../INDICE_COMPLETO.md)
