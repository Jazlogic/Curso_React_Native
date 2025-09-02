# 🔧 Clase 5: Profiling y Análisis de Performance

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase dominarás el profiling avanzado y análisis de performance de aplicaciones React Native. Aprenderás a usar React DevTools Profiler, implementar monitoring en tiempo real, analizar bundles, detectar memory leaks y crear testing automatizado de performance.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Usar** React DevTools Profiler avanzado
2. **Implementar** performance monitoring en tiempo real
3. **Analizar** bundles y optimizar tamaño
4. **Detectar** memory leaks y optimizar memoria
5. **Crear** testing automatizado de performance
6. **Implementar** métricas y dashboards de performance

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. React DevTools Profiler Avanzado**

#### **Configuración del Profiler**
```javascript
// Configuración avanzada del Profiler
import {Profiler} from 'react';

function App() {
  const onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime, interactions) => {
    // Análisis detallado de rendimiento
    console.log('Profiler data:', {
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime,
      interactions,
      // Métricas adicionales
      renderTime: actualDuration,
      commitTime: commitTime - startTime,
      totalTime: actualDuration + (commitTime - startTime),
    });
  };

  return (
    <Profiler id="App" onRender={onRenderCallback}>
      {/* Tu aplicación */}
    </Profiler>
  );
}
```

#### **Análisis de Componentes**
```javascript
// Análisis detallado de componentes
import {Profiler} from 'react';

function ComponentProfiler({children, name}) {
  const onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime) => {
    // Análisis específico del componente
    const analysis = {
      componentName: name,
      phase,
      renderTime: actualDuration,
      baseTime: baseDuration,
      efficiency: baseDuration > 0 ? (actualDuration / baseDuration) * 100 : 100,
      timestamp: Date.now(),
    };

    // Log para análisis
    console.log(`Component ${name} analysis:`, analysis);
    
    // Enviar a servicio de monitoring
    sendToMonitoring(analysis);
  };

  return (
    <Profiler id={name} onRender={onRenderCallback}>
      {children}
    </Profiler>
  );
}

// Uso
function MyComponent() {
  return (
    <ComponentProfiler name="MyComponent">
      <div>Component content</div>
    </ComponentProfiler>
  );
}
```

#### **Métricas de Performance**
```javascript
// Métricas avanzadas de performance
class PerformanceMetrics {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      renderTime: 16, // 60fps
      commitTime: 8,
      totalTime: 24,
    };
  }

  recordMetric(componentName, phase, duration) {
    const key = `${componentName}-${phase}`;
    const existing = this.metrics.get(key) || [];
    
    existing.push({
      duration,
      timestamp: Date.now(),
      phase,
    });
    
    this.metrics.set(key, existing);
    
    // Verificar thresholds
    this.checkThresholds(componentName, phase, duration);
  }

  checkThresholds(componentName, phase, duration) {
    const threshold = this.thresholds[phase];
    if (threshold && duration > threshold) {
      console.warn(`Performance warning: ${componentName} ${phase} took ${duration}ms (threshold: ${threshold}ms)`);
    }
  }

  getAverageMetrics(componentName) {
    const metrics = [];
    for (const [key, values] of this.metrics.entries()) {
      if (key.startsWith(componentName)) {
        metrics.push(...values);
      }
    }
    
    if (metrics.length === 0) return null;
    
    const total = metrics.reduce((sum, metric) => sum + metric.duration, 0);
    return {
      average: total / metrics.length,
      count: metrics.length,
      min: Math.min(...metrics.map(m => m.duration)),
      max: Math.max(...metrics.map(m => m.duration)),
    };
  }
}
```

### **2. Performance Monitoring en Tiempo Real**

#### **Real-time Performance Monitor**
```javascript
// Monitor de performance en tiempo real
class RealTimePerformanceMonitor {
  constructor() {
    this.metrics = [];
    this.isMonitoring = false;
    this.interval = null;
  }

  startMonitoring() {
    this.isMonitoring = true;
    this.interval = setInterval(() => {
      this.collectMetrics();
    }, 1000);
  }

  stopMonitoring() {
    this.isMonitoring = false;
    if (this.interval) {
      clearInterval(this.interval);
    }
  }

  collectMetrics() {
    const metrics = {
      timestamp: Date.now(),
      memory: this.getMemoryUsage(),
      frameRate: this.getFrameRate(),
      renderTime: this.getRenderTime(),
      network: this.getNetworkMetrics(),
    };

    this.metrics.push(metrics);
    
    // Mantener solo los últimos 100 registros
    if (this.metrics.length > 100) {
      this.metrics.shift();
    }

    // Enviar a servicio de monitoring
    this.sendToMonitoring(metrics);
  }

  getMemoryUsage() {
    if (performance.memory) {
      return {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit,
      };
    }
    return null;
  }

  getFrameRate() {
    // Calcular frame rate basado en requestAnimationFrame
    let frames = 0;
    const startTime = Date.now();
    
    const countFrames = () => {
      frames++;
      if (Date.now() - startTime < 1000) {
        requestAnimationFrame(countFrames);
      }
    };
    
    requestAnimationFrame(countFrames);
    return frames;
  }

  getRenderTime() {
    // Medir tiempo de render
    const startTime = performance.now();
    
    // Simular render
    requestAnimationFrame(() => {
      const endTime = performance.now();
      return endTime - startTime;
    });
  }

  getNetworkMetrics() {
    // Métricas de red
    return {
      activeConnections: navigator.connection?.effectiveType || 'unknown',
      downlink: navigator.connection?.downlink || 0,
      rtt: navigator.connection?.rtt || 0,
    };
  }

  sendToMonitoring(metrics) {
    // Enviar a servicio de monitoring
    fetch('/api/performance-metrics', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(metrics),
    }).catch(error => {
      console.error('Error sending metrics:', error);
    });
  }
}
```

#### **Performance Dashboard**
```javascript
// Dashboard de performance
import React, {useState, useEffect} from 'react';

function PerformanceDashboard() {
  const [metrics, setMetrics] = useState([]);
  const [isMonitoring, setIsMonitoring] = useState(false);

  useEffect(() => {
    const monitor = new RealTimePerformanceMonitor();
    
    if (isMonitoring) {
      monitor.startMonitoring();
    } else {
      monitor.stopMonitoring();
    }

    return () => monitor.stopMonitoring();
  }, [isMonitoring]);

  const renderMetrics = () => {
    return metrics.map((metric, index) => (
      <div key={index} className="metric-card">
        <h3>Timestamp: {new Date(metric.timestamp).toLocaleTimeString()}</h3>
        <div>Memory: {metric.memory ? `${(metric.memory.used / 1024 / 1024).toFixed(2)} MB` : 'N/A'}</div>
        <div>Frame Rate: {metric.frameRate} FPS</div>
        <div>Render Time: {metric.renderTime}ms</div>
        <div>Network: {metric.network.activeConnections}</div>
      </div>
    ));
  };

  return (
    <div className="performance-dashboard">
      <h2>Performance Dashboard</h2>
      <button onClick={() => setIsMonitoring(!isMonitoring)}>
        {isMonitoring ? 'Stop Monitoring' : 'Start Monitoring'}
      </button>
      <div className="metrics-container">
        {renderMetrics()}
      </div>
    </div>
  );
}
```

### **3. Bundle Analysis y Optimización**

#### **Bundle Analyzer**
```javascript
// Analizador de bundle
class BundleAnalyzer {
  constructor() {
    this.modules = [];
    this.dependencies = new Map();
  }

  analyzeBundle(bundleContent) {
    // Extraer módulos del bundle
    const moduleRegex = /__d\((\d+),\[([^\]]*)\],function\([^)]*\)\{([^}]*)\}/g;
    let match;

    while ((match = moduleRegex.exec(bundleContent)) !== null) {
      const module = {
        id: match[1],
        dependencies: match[2].split(',').filter(Boolean),
        code: match[3],
        size: match[0].length,
      };
      
      this.modules.push(module);
      this.analyzeDependencies(module);
    }

    return this.generateReport();
  }

  analyzeDependencies(module) {
    module.dependencies.forEach(dep => {
      if (!this.dependencies.has(dep)) {
        this.dependencies.set(dep, []);
      }
      this.dependencies.get(dep).push(module.id);
    });
  }

  generateReport() {
    const totalSize = this.modules.reduce((sum, module) => sum + module.size, 0);
    const averageSize = totalSize / this.modules.length;
    
    const largestModules = this.modules
      .sort((a, b) => b.size - a.size)
      .slice(0, 10);

    const duplicateModules = this.findDuplicateModules();
    const unusedModules = this.findUnusedModules();

    return {
      totalSize,
      averageSize,
      moduleCount: this.modules.length,
      largestModules,
      duplicateModules,
      unusedModules,
      recommendations: this.generateRecommendations(),
    };
  }

  findDuplicateModules() {
    const duplicates = [];
    const seen = new Set();

    this.modules.forEach(module => {
      if (seen.has(module.code)) {
        duplicates.push(module);
      } else {
        seen.add(module.code);
      }
    });

    return duplicates;
  }

  findUnusedModules() {
    const used = new Set();
    
    // Marcar módulos usados
    this.modules.forEach(module => {
      module.dependencies.forEach(dep => {
        used.add(dep);
      });
    });

    // Encontrar módulos no usados
    return this.modules.filter(module => !used.has(module.id));
  }

  generateRecommendations() {
    const recommendations = [];

    if (this.modules.length > 1000) {
      recommendations.push('Consider code splitting for large applications');
    }

    if (this.findDuplicateModules().length > 0) {
      recommendations.push('Remove duplicate modules');
    }

    if (this.findUnusedModules().length > 0) {
      recommendations.push('Remove unused modules');
    }

    return recommendations;
  }
}
```

#### **Bundle Optimization**
```javascript
// Optimizador de bundle
class BundleOptimizer {
  constructor() {
    this.optimizations = [];
  }

  optimizeBundle(bundleContent) {
    let optimized = bundleContent;

    // Aplicar optimizaciones
    optimized = this.removeUnusedCode(optimized);
    optimized = this.minifyCode(optimized);
    optimized = this.optimizeImports(optimized);
    optimized = this.compressCode(optimized);

    return optimized;
  }

  removeUnusedCode(bundle) {
    // Remover código no usado
    const unusedCodeRegex = /\/\* unused \*\/[^}]*\}/g;
    return bundle.replace(unusedCodeRegex, '');
  }

  minifyCode(bundle) {
    // Minificar código
    return bundle
      .replace(/\s+/g, ' ')
      .replace(/;\s*}/g, '}')
      .replace(/{\s*/g, '{')
      .replace(/\s*}/g, '}');
  }

  optimizeImports(bundle) {
    // Optimizar imports
    const importRegex = /import\s+{[^}]*}\s+from\s+['"][^'"]*['"];?/g;
    return bundle.replace(importRegex, (match) => {
      // Optimizar imports específicos
      return match.replace(/\s+/g, ' ').trim();
    });
  }

  compressCode(bundle) {
    // Comprimir código
    return bundle
      .replace(/\/\*[^*]*\*+(?:[^/*][^*]*\*+)*\//g, '') // Remover comentarios
      .replace(/\/\/.*$/gm, '') // Remover comentarios de línea
      .replace(/\s+/g, ' ') // Comprimir espacios
      .trim();
  }
}
```

### **4. Memory Leak Detection**

#### **Memory Leak Detector**
```javascript
// Detector de memory leaks
class MemoryLeakDetector {
  constructor() {
    this.snapshots = [];
    this.leaks = [];
    this.isMonitoring = false;
  }

  startMonitoring() {
    this.isMonitoring = true;
    this.takeSnapshot();
    
    // Tomar snapshots cada 30 segundos
    this.interval = setInterval(() => {
      this.takeSnapshot();
      this.detectLeaks();
    }, 30000);
  }

  stopMonitoring() {
    this.isMonitoring = false;
    if (this.interval) {
      clearInterval(this.interval);
    }
  }

  takeSnapshot() {
    if (performance.memory) {
      const snapshot = {
        timestamp: Date.now(),
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit,
      };
      
      this.snapshots.push(snapshot);
      
      // Mantener solo los últimos 20 snapshots
      if (this.snapshots.length > 20) {
        this.snapshots.shift();
      }
    }
  }

  detectLeaks() {
    if (this.snapshots.length < 2) return;

    const current = this.snapshots[this.snapshots.length - 1];
    const previous = this.snapshots[this.snapshots.length - 2];
    
    const memoryIncrease = current.used - previous.used;
    const timeIncrease = current.timestamp - previous.timestamp;
    
    // Detectar memory leak si el uso de memoria aumenta consistentemente
    if (memoryIncrease > 0 && timeIncrease > 0) {
      const leakRate = memoryIncrease / timeIncrease;
      
      if (leakRate > 1000) { // 1KB por segundo
        this.leaks.push({
          timestamp: current.timestamp,
          memoryIncrease,
          leakRate,
          severity: this.calculateSeverity(leakRate),
        });
        
        console.warn('Memory leak detected:', {
          memoryIncrease: `${(memoryIncrease / 1024).toFixed(2)} KB`,
          leakRate: `${(leakRate / 1024).toFixed(2)} KB/s`,
          severity: this.calculateSeverity(leakRate),
        });
      }
    }
  }

  calculateSeverity(leakRate) {
    if (leakRate > 10000) return 'critical';
    if (leakRate > 5000) return 'high';
    if (leakRate > 1000) return 'medium';
    return 'low';
  }

  getLeakReport() {
    return {
      totalLeaks: this.leaks.length,
      leaks: this.leaks,
      averageLeakRate: this.leaks.reduce((sum, leak) => sum + leak.leakRate, 0) / this.leaks.length,
      criticalLeaks: this.leaks.filter(leak => leak.severity === 'critical').length,
    };
  }
}
```

#### **Memory Optimization**
```javascript
// Optimizador de memoria
class MemoryOptimizer {
  constructor() {
    this.objectPool = new Map();
    this.weakRefs = new WeakMap();
  }

  // Pool de objetos reutilizables
  getObject(type) {
    if (this.objectPool.has(type)) {
      const pool = this.objectPool.get(type);
      if (pool.length > 0) {
        return pool.pop();
      }
    }
    return this.createObject(type);
  }

  returnObject(type, object) {
    if (!this.objectPool.has(type)) {
      this.objectPool.set(type, []);
    }
    
    // Limpiar objeto
    this.clearObject(object);
    
    // Devolver al pool
    this.objectPool.get(type).push(object);
  }

  createObject(type) {
    switch (type) {
      case 'user':
        return { id: null, name: null, email: null };
      case 'product':
        return { id: null, title: null, price: null };
      default:
        return {};
    }
  }

  clearObject(object) {
    Object.keys(object).forEach(key => {
      delete object[key];
    });
  }

  // Weak references para evitar memory leaks
  setWeakRef(key, value) {
    this.weakRefs.set(key, new WeakRef(value));
  }

  getWeakRef(key) {
    const weakRef = this.weakRefs.get(key);
    if (weakRef) {
      return weakRef.deref();
    }
    return null;
  }

  // Limpiar referencias débiles
  cleanupWeakRefs() {
    for (const [key, weakRef] of this.weakRefs.entries()) {
      if (!weakRef.deref()) {
        this.weakRefs.delete(key);
      }
    }
  }
}
```

### **5. Automated Performance Testing**

#### **Performance Test Suite**
```javascript
// Suite de testing de performance
class PerformanceTestSuite {
  constructor() {
    this.tests = [];
    this.results = [];
  }

  addTest(name, testFunction, threshold) {
    this.tests.push({
      name,
      testFunction,
      threshold,
    });
  }

  async runTests() {
    this.results = [];
    
    for (const test of this.tests) {
      const result = await this.runTest(test);
      this.results.push(result);
    }
    
    return this.generateReport();
  }

  async runTest(test) {
    const startTime = performance.now();
    
    try {
      await test.testFunction();
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      return {
        name: test.name,
        duration,
        threshold: test.threshold,
        passed: duration <= test.threshold,
        error: null,
      };
    } catch (error) {
      return {
        name: test.name,
        duration: null,
        threshold: test.threshold,
        passed: false,
        error: error.message,
      };
    }
  }

  generateReport() {
    const totalTests = this.results.length;
    const passedTests = this.results.filter(r => r.passed).length;
    const failedTests = totalTests - passedTests;
    
    const averageDuration = this.results
      .filter(r => r.duration !== null)
      .reduce((sum, r) => sum + r.duration, 0) / passedTests;

    return {
      totalTests,
      passedTests,
      failedTests,
      averageDuration,
      results: this.results,
    };
  }
}

// Ejemplo de uso
const testSuite = new PerformanceTestSuite();

// Test de render time
testSuite.addTest('Render Time', async () => {
  const component = render(<MyComponent />);
  await waitFor(() => {
    expect(component).toBeInTheDocument();
  });
}, 100); // 100ms threshold

// Test de memory usage
testSuite.addTest('Memory Usage', async () => {
  const initialMemory = performance.memory?.usedJSHeapSize || 0;
  
  // Ejecutar operación
  await performOperation();
  
  const finalMemory = performance.memory?.usedJSHeapSize || 0;
  const memoryIncrease = finalMemory - initialMemory;
  
  if (memoryIncrease > 1024 * 1024) { // 1MB
    throw new Error(`Memory increase too high: ${memoryIncrease} bytes`);
  }
}, 0);

// Ejecutar tests
testSuite.runTests().then(report => {
  console.log('Performance test report:', report);
});
```

#### **Performance CI/CD Integration**
```javascript
// Integración con CI/CD
class PerformanceCI {
  constructor() {
    this.baseline = null;
    this.thresholds = {
      renderTime: 0.1, // 10% increase
      memoryUsage: 0.2, // 20% increase
      bundleSize: 0.05, // 5% increase
    };
  }

  async runPerformanceTests() {
    const results = await this.collectMetrics();
    const comparison = this.compareWithBaseline(results);
    
    if (comparison.failed) {
      this.failBuild(comparison);
    } else {
      this.passBuild(comparison);
    }
    
    return comparison;
  }

  async collectMetrics() {
    return {
      renderTime: await this.measureRenderTime(),
      memoryUsage: await this.measureMemoryUsage(),
      bundleSize: await this.measureBundleSize(),
    };
  }

  compareWithBaseline(current) {
    if (!this.baseline) {
      this.baseline = current;
      return { passed: true, message: 'Baseline established' };
    }

    const comparison = {
      renderTime: this.compareMetric(current.renderTime, this.baseline.renderTime, this.thresholds.renderTime),
      memoryUsage: this.compareMetric(current.memoryUsage, this.baseline.memoryUsage, this.thresholds.memoryUsage),
      bundleSize: this.compareMetric(current.bundleSize, this.baseline.bundleSize, this.thresholds.bundleSize),
    };

    const failed = Object.values(comparison).some(c => !c.passed);
    
    return {
      passed: !failed,
      failed,
      comparison,
    };
  }

  compareMetric(current, baseline, threshold) {
    const increase = (current - baseline) / baseline;
    const passed = increase <= threshold;
    
    return {
      current,
      baseline,
      increase,
      threshold,
      passed,
    };
  }

  failBuild(comparison) {
    console.error('Performance regression detected:', comparison);
    process.exit(1);
  }

  passBuild(comparison) {
    console.log('Performance tests passed:', comparison);
  }
}
```

---

## 🛠️ Contenido Práctico

### **Ejercicio 1: Performance Profiler**

#### **Objetivo**
Implementar un profiler de performance avanzado.

#### **Código de Ejemplo**
```javascript
// AdvancedProfiler.js
import React, {Profiler} from 'react';

class AdvancedProfiler {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      renderTime: 16,
      commitTime: 8,
      totalTime: 24,
    };
  }

  onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime) => {
    const metric = {
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime,
      timestamp: Date.now(),
    };

    this.recordMetric(metric);
    this.checkThresholds(metric);
  };

  recordMetric(metric) {
    const key = `${metric.id}-${metric.phase}`;
    const existing = this.metrics.get(key) || [];
    existing.push(metric);
    this.metrics.set(key, existing);
  }

  checkThresholds(metric) {
    const threshold = this.thresholds[metric.phase];
    if (threshold && metric.actualDuration > threshold) {
      console.warn(`Performance warning: ${metric.id} ${metric.phase} took ${metric.actualDuration}ms`);
    }
  }

  getReport() {
    const report = {};
    
    for (const [key, metrics] of this.metrics.entries()) {
      const total = metrics.reduce((sum, m) => sum + m.actualDuration, 0);
      report[key] = {
        average: total / metrics.length,
        count: metrics.length,
        min: Math.min(...metrics.map(m => m.actualDuration)),
        max: Math.max(...metrics.map(m => m.actualDuration)),
      };
    }
    
    return report;
  }
}

// Uso
const profiler = new AdvancedProfiler();

function App() {
  return (
    <Profiler id="App" onRender={profiler.onRenderCallback}>
      <MyComponent />
    </Profiler>
  );
}
```

### **Ejercicio 2: Memory Leak Detector**

#### **Objetivo**
Implementar un detector de memory leaks.

#### **Código de Ejemplo**
```javascript
// MemoryLeakDetector.js
class MemoryLeakDetector {
  constructor() {
    this.snapshots = [];
    this.leaks = [];
    this.isMonitoring = false;
  }

  startMonitoring() {
    this.isMonitoring = true;
    this.takeSnapshot();
    
    this.interval = setInterval(() => {
      this.takeSnapshot();
      this.detectLeaks();
    }, 30000);
  }

  stopMonitoring() {
    this.isMonitoring = false;
    if (this.interval) {
      clearInterval(this.interval);
    }
  }

  takeSnapshot() {
    if (performance.memory) {
      const snapshot = {
        timestamp: Date.now(),
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit,
      };
      
      this.snapshots.push(snapshot);
      
      if (this.snapshots.length > 20) {
        this.snapshots.shift();
      }
    }
  }

  detectLeaks() {
    if (this.snapshots.length < 2) return;

    const current = this.snapshots[this.snapshots.length - 1];
    const previous = this.snapshots[this.snapshots.length - 2];
    
    const memoryIncrease = current.used - previous.used;
    const timeIncrease = current.timestamp - previous.timestamp;
    
    if (memoryIncrease > 0 && timeIncrease > 0) {
      const leakRate = memoryIncrease / timeIncrease;
      
      if (leakRate > 1000) {
        this.leaks.push({
          timestamp: current.timestamp,
          memoryIncrease,
          leakRate,
          severity: this.calculateSeverity(leakRate),
        });
        
        console.warn('Memory leak detected:', {
          memoryIncrease: `${(memoryIncrease / 1024).toFixed(2)} KB`,
          leakRate: `${(leakRate / 1024).toFixed(2)} KB/s`,
        });
      }
    }
  }

  calculateSeverity(leakRate) {
    if (leakRate > 10000) return 'critical';
    if (leakRate > 5000) return 'high';
    if (leakRate > 1000) return 'medium';
    return 'low';
  }

  getReport() {
    return {
      totalLeaks: this.leaks.length,
      leaks: this.leaks,
      averageLeakRate: this.leaks.reduce((sum, leak) => sum + leak.leakRate, 0) / this.leaks.length,
      criticalLeaks: this.leaks.filter(leak => leak.severity === 'critical').length,
    };
  }
}

// Uso
const detector = new MemoryLeakDetector();
detector.startMonitoring();
```

### **Ejercicio 3: Performance Dashboard**

#### **Objetivo**
Crear un dashboard de performance en tiempo real.

#### **Código de Ejemplo**
```javascript
// PerformanceDashboard.js
import React, {useState, useEffect} from 'react';

function PerformanceDashboard() {
  const [metrics, setMetrics] = useState([]);
  const [isMonitoring, setIsMonitoring] = useState(false);

  useEffect(() => {
    const monitor = new RealTimePerformanceMonitor();
    
    if (isMonitoring) {
      monitor.startMonitoring();
    } else {
      monitor.stopMonitoring();
    }

    return () => monitor.stopMonitoring();
  }, [isMonitoring]);

  const renderMetrics = () => {
    return metrics.map((metric, index) => (
      <div key={index} className="metric-card">
        <h3>Timestamp: {new Date(metric.timestamp).toLocaleTimeString()}</h3>
        <div>Memory: {metric.memory ? `${(metric.memory.used / 1024 / 1024).toFixed(2)} MB` : 'N/A'}</div>
        <div>Frame Rate: {metric.frameRate} FPS</div>
        <div>Render Time: {metric.renderTime}ms</div>
        <div>Network: {metric.network.activeConnections}</div>
      </div>
    ));
  };

  return (
    <div className="performance-dashboard">
      <h2>Performance Dashboard</h2>
      <button onClick={() => setIsMonitoring(!isMonitoring)}>
        {isMonitoring ? 'Stop Monitoring' : 'Start Monitoring'}
      </button>
      <div className="metrics-container">
        {renderMetrics()}
      </div>
    </div>
  );
}

export default PerformanceDashboard;
```

### **Ejercicio 4: Automated Performance Testing**

#### **Objetivo**
Implementar testing automatizado de performance.

#### **Código de Ejemplo**
```javascript
// PerformanceTestSuite.js
class PerformanceTestSuite {
  constructor() {
    this.tests = [];
    this.results = [];
  }

  addTest(name, testFunction, threshold) {
    this.tests.push({
      name,
      testFunction,
      threshold,
    });
  }

  async runTests() {
    this.results = [];
    
    for (const test of this.tests) {
      const result = await this.runTest(test);
      this.results.push(result);
    }
    
    return this.generateReport();
  }

  async runTest(test) {
    const startTime = performance.now();
    
    try {
      await test.testFunction();
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      return {
        name: test.name,
        duration,
        threshold: test.threshold,
        passed: duration <= test.threshold,
        error: null,
      };
    } catch (error) {
      return {
        name: test.name,
        duration: null,
        threshold: test.threshold,
        passed: false,
        error: error.message,
      };
    }
  }

  generateReport() {
    const totalTests = this.results.length;
    const passedTests = this.results.filter(r => r.passed).length;
    const failedTests = totalTests - passedTests;
    
    const averageDuration = this.results
      .filter(r => r.duration !== null)
      .reduce((sum, r) => sum + r.duration, 0) / passedTests;

    return {
      totalTests,
      passedTests,
      failedTests,
      averageDuration,
      results: this.results,
    };
  }
}

// Uso
const testSuite = new PerformanceTestSuite();

testSuite.addTest('Render Time', async () => {
  const component = render(<MyComponent />);
  await waitFor(() => {
    expect(component).toBeInTheDocument();
  });
}, 100);

testSuite.runTests().then(report => {
  console.log('Performance test report:', report);
});
```

---

## 🎯 Ejercicios de Evaluación

### **Ejercicio 1: Performance Profiler**
- Implementar un profiler de performance avanzado
- Configurar thresholds y alertas
- Generar reportes de performance

### **Ejercicio 2: Memory Leak Detection**
- Implementar detector de memory leaks
- Configurar monitoring en tiempo real
- Generar reportes de memory leaks

### **Ejercicio 3: Performance Dashboard**
- Crear dashboard de performance
- Implementar métricas en tiempo real
- Configurar alertas de performance

### **Ejercicio 4: Automated Testing**
- Implementar testing automatizado de performance
- Configurar CI/CD integration
- Establecer baselines de performance

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [React DevTools Profiler](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)
- [Performance Monitoring](https://reactnative.dev/docs/performance)
- [Bundle Analysis](https://webpack.js.org/guides/code-splitting/)

### **Herramientas Útiles**
- [React DevTools](https://github.com/facebook/react-devtools)
- [Bundle Analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)
- [Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance)

### **Mejores Prácticas**
- Profilar regularmente durante el desarrollo
- Establecer métricas de performance
- Implementar monitoring en producción
- Automatizar testing de performance

---

## 🚀 Siguiente Módulo

En el próximo módulo aprenderás sobre **PWA y Funcionalidades Web**, donde profundizarás en el desarrollo de Progressive Web Apps, service workers, offline functionality y optimización para web.

---

**💡 Consejo**: El profiling y análisis de performance son esenciales para aplicaciones React Native de calidad. Invierte tiempo en implementar herramientas de monitoring y testing automatizado.
