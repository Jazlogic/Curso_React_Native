# 🔧 Clase 1: Flipper y React DevTools Avanzado

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase dominarás las herramientas más avanzadas de debugging para React Native: Flipper y React DevTools. Aprenderás a configurar, usar y aprovechar al máximo estas herramientas profesionales que diferencian a un desarrollador senior.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Configurar** Flipper para debugging avanzado
2. **Usar plugins** específicos para React Native
3. **Inspeccionar** network requests y responses
4. **Gestionar** bases de datos locales
5. **Analizar** performance y memoria
6. **Dominar** React DevTools Profiler

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Introducción a Flipper**

#### **¿Qué es Flipper?**
Flipper es una plataforma de debugging multiplataforma desarrollada por Facebook que permite inspeccionar, controlar y debuggear aplicaciones React Native, iOS y Android.

#### **Características Principales**
- **Multiplataforma**: iOS, Android, React Native
- **Plugins**: Extensible con plugins personalizados
- **Network Inspection**: Análisis de requests HTTP/HTTPS
- **Database Inspection**: Visualización de bases de datos
- **Performance Monitoring**: Análisis de memoria y CPU
- **Layout Inspector**: Inspección de UI nativa

#### **Arquitectura de Flipper**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Flipper App   │◄──►│   Flipper SDK   │◄──►│  React Native   │
│   (Desktop)     │    │   (Mobile)      │    │    App          │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### **2. Configuración de Flipper**

#### **Instalación**
```bash
# Instalar Flipper Desktop
# Descargar desde: https://fbflipper.com/

# Instalar Flipper SDK en React Native
npm install --save-dev react-native-flipper
```

#### **Configuración en React Native**
```javascript
// metro.config.js
const {getDefaultConfig} = require('metro-config');

module.exports = (async () => {
  const {
    resolver: {sourceExts, assetExts},
  } = await getDefaultConfig();
  return {
    transformer: {
      getTransformOptions: async () => ({
        transform: {
          experimentalImportSupport: false,
          inlineRequires: true,
        },
      }),
    },
    resolver: {
      assetExts: assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...sourceExts, 'svg'],
    },
  };
})();
```

#### **Configuración en Android**
```gradle
// android/app/build.gradle
android {
    compileSdkVersion rootProject.ext.compileSdkVersion

    defaultConfig {
        // ... existing config
        buildConfigField "boolean", "FLIPPER_ENABLED", "true"
    }
}

dependencies {
    debugImplementation 'com.facebook.flipper:flipper:0.125.0'
    debugImplementation 'com.facebook.flipper:flipper-network-plugin:0.125.0'
    debugImplementation 'com.facebook.flipper:flipper-fresco-plugin:0.125.0'
}
```

#### **Configuración en iOS**
```ruby
# ios/Podfile
def flipper_pods()
  pod 'FlipperKit', '~> 0.125.0', :configurations => ['Debug']
  pod 'FlipperKit/FlipperKitLayoutPlugin', '~> 0.125.0', :configurations => ['Debug']
  pod 'FlipperKit/FlipperKitUserDefaultsPlugin', '~> 0.125.0', :configurations => ['Debug']
  pod 'FlipperKit/FlipperKitNetworkPlugin', '~> 0.125.0', :configurations => ['Debug']
end

post_install do |installer|
  flipper_pods()
end
```

### **3. Plugins de Flipper**

#### **Network Plugin**
```javascript
// Configuración del Network Plugin
import {Flipper} from 'react-native-flipper';

// En tu aplicación
Flipper.addPlugin({
  getId() {
    return 'Network';
  },
  onConnect(connection) {
    // Configurar interceptación de network
    this.connection = connection;
  },
  onDisconnect() {
    this.connection = null;
  },
  runInBackground() {
    return true;
  },
});
```

#### **Database Plugin**
```javascript
// Plugin para AsyncStorage
import AsyncStorage from '@react-native-async-storage/async-storage';

Flipper.addPlugin({
  getId() {
    return 'AsyncStorage';
  },
  onConnect(connection) {
    this.connection = connection;
    this.setupAsyncStorageInspection();
  },
  
  setupAsyncStorageInspection() {
    // Interceptar operaciones de AsyncStorage
    const originalGetItem = AsyncStorage.getItem;
    AsyncStorage.getItem = async (key) => {
      const result = await originalGetItem(key);
      this.connection.send('asyncStorageGet', {key, value: result});
      return result;
    };
  },
});
```

#### **Performance Plugin**
```javascript
// Plugin para monitoreo de performance
Flipper.addPlugin({
  getId() {
    return 'Performance';
  },
  onConnect(connection) {
    this.connection = connection;
    this.startPerformanceMonitoring();
  },
  
  startPerformanceMonitoring() {
    // Monitorear frame rate
    setInterval(() => {
      const frameRate = this.getFrameRate();
      this.connection.send('frameRate', {frameRate});
    }, 1000);
  },
});
```

### **4. React DevTools Avanzado**

#### **Instalación y Configuración**
```bash
# Instalar React DevTools
npm install --save-dev react-devtools

# Ejecutar React DevTools
npx react-devtools
```

#### **Profiler Avanzado**
```javascript
// Configuración del Profiler
import {Profiler} from 'react';

function MyComponent() {
  const onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime) => {
    console.log('Profiler:', {
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime
    });
  };

  return (
    <Profiler id="MyComponent" onRender={onRenderCallback}>
      {/* Tu componente */}
    </Profiler>
  );
}
```

#### **Component Inspector**
```javascript
// Configuración para inspección de componentes
import {unstable_trace as trace} from 'scheduler/tracing';

function ExpensiveComponent() {
  return trace('ExpensiveComponent', performance.now(), () => {
    // Componente costoso
    return <div>Expensive content</div>;
  });
}
```

### **5. Debugging Avanzado**

#### **Network Debugging**
```javascript
// Interceptar requests HTTP
import {XMLHttpRequest} from 'xmlhttprequest';

const originalOpen = XMLHttpRequest.prototype.open;
XMLHttpRequest.prototype.open = function(method, url, async, user, password) {
  console.log(`HTTP ${method} ${url}`);
  return originalOpen.call(this, method, url, async, user, password);
};

const originalSend = XMLHttpRequest.prototype.send;
XMLHttpRequest.prototype.send = function(data) {
  console.log('Request data:', data);
  return originalSend.call(this, data);
};
```

#### **State Debugging**
```javascript
// Hook para debugging de estado
import {useEffect, useRef} from 'react';

function useDebugState(state, name) {
  const prevState = useRef();
  
  useEffect(() => {
    if (prevState.current !== state) {
      console.log(`${name} changed:`, {
        previous: prevState.current,
        current: state
      });
      prevState.current = state;
    }
  }, [state, name]);
}

// Uso
function MyComponent() {
  const [count, setCount] = useState(0);
  useDebugState(count, 'count');
  
  return <div>{count}</div>;
}
```

#### **Memory Debugging**
```javascript
// Monitoreo de memoria
function useMemoryMonitor() {
  useEffect(() => {
    const interval = setInterval(() => {
      if (performance.memory) {
        console.log('Memory usage:', {
          used: performance.memory.usedJSHeapSize,
          total: performance.memory.totalJSHeapSize,
          limit: performance.memory.jsHeapSizeLimit
        });
      }
    }, 5000);
    
    return () => clearInterval(interval);
  }, []);
}
```

---

## 🛠️ Contenido Práctico

### **Ejercicio 1: Configuración de Flipper**

#### **Objetivo**
Configurar Flipper en una aplicación React Native existente.

#### **Pasos**
1. **Instalar Flipper Desktop**
2. **Configurar Flipper SDK** en el proyecto
3. **Configurar plugins** básicos
4. **Verificar conexión** entre Flipper y la app

#### **Código de Ejemplo**
```javascript
// App.js
import React from 'react';
import {View, Text, Button} from 'react-native';
import {Flipper} from 'react-native-flipper';

export default function App() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // Configurar Flipper
    Flipper.addPlugin({
      getId() {
        return 'Counter';
      },
      onConnect(connection) {
        this.connection = connection;
      },
      onDisconnect() {
        this.connection = null;
      },
    });
  }, []);

  return (
    <View style={{flex: 1, justifyContent: 'center', alignItems: 'center'}}>
      <Text>Count: {count}</Text>
      <Button title="Increment" onPress={() => setCount(count + 1)} />
    </View>
  );
}
```

### **Ejercicio 2: Network Plugin**

#### **Objetivo**
Implementar un plugin de Flipper para monitorear requests HTTP.

#### **Código de Ejemplo**
```javascript
// NetworkPlugin.js
import {Flipper} from 'react-native-flipper';

class NetworkPlugin {
  constructor() {
    this.connection = null;
    this.setupNetworkInterception();
  }

  setupNetworkInterception() {
    // Interceptar fetch
    const originalFetch = global.fetch;
    global.fetch = async (url, options) => {
      const startTime = Date.now();
      
      try {
        const response = await originalFetch(url, options);
        const endTime = Date.now();
        
        this.logRequest({
          url,
          method: options?.method || 'GET',
          status: response.status,
          duration: endTime - startTime,
          headers: options?.headers,
          body: options?.body
        });
        
        return response;
      } catch (error) {
        this.logRequest({
          url,
          method: options?.method || 'GET',
          error: error.message,
          duration: Date.now() - startTime
        });
        throw error;
      }
    };
  }

  logRequest(requestData) {
    if (this.connection) {
      this.connection.send('networkRequest', requestData);
    }
  }
}

// Configurar plugin
Flipper.addPlugin({
  getId() {
    return 'Network';
  },
  onConnect(connection) {
    this.connection = connection;
    this.networkPlugin = new NetworkPlugin();
  },
  onDisconnect() {
    this.connection = null;
    this.networkPlugin = null;
  },
});
```

### **Ejercicio 3: React DevTools Profiler**

#### **Objetivo**
Implementar profiling avanzado con React DevTools.

#### **Código de Ejemplo**
```javascript
// ProfilerComponent.js
import React, {Profiler, useState, useMemo} from 'react';

function ExpensiveList({items}) {
  const sortedItems = useMemo(() => {
    console.log('Sorting items...');
    return items.sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);

  return (
    <div>
      {sortedItems.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

function App() {
  const [items, setItems] = useState([
    {id: 1, name: 'Item 3'},
    {id: 2, name: 'Item 1'},
    {id: 3, name: 'Item 2'},
  ]);

  const onRenderCallback = (id, phase, actualDuration, baseDuration, startTime, commitTime) => {
    console.log('Profiler data:', {
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime
    });
  };

  return (
    <div>
      <Profiler id="ExpensiveList" onRender={onRenderCallback}>
        <ExpensiveList items={items} />
      </Profiler>
      <button onClick={() => setItems([...items, {id: Date.now(), name: `Item ${items.length + 1}`}])}>
        Add Item
      </button>
    </div>
  );
}
```

### **Ejercicio 4: Database Plugin**

#### **Objetivo**
Crear un plugin para inspeccionar AsyncStorage.

#### **Código de Ejemplo**
```javascript
// AsyncStoragePlugin.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import {Flipper} from 'react-native-flipper';

class AsyncStoragePlugin {
  constructor() {
    this.connection = null;
    this.setupAsyncStorageInterception();
  }

  setupAsyncStorageInterception() {
    // Interceptar getItem
    const originalGetItem = AsyncStorage.getItem;
    AsyncStorage.getItem = async (key) => {
      const result = await originalGetItem(key);
      this.logOperation('getItem', {key, value: result});
      return result;
    };

    // Interceptar setItem
    const originalSetItem = AsyncStorage.setItem;
    AsyncStorage.setItem = async (key, value) => {
      await originalSetItem(key, value);
      this.logOperation('setItem', {key, value});
    };

    // Interceptar removeItem
    const originalRemoveItem = AsyncStorage.removeItem;
    AsyncStorage.removeItem = async (key) => {
      await originalRemoveItem(key);
      this.logOperation('removeItem', {key});
    };
  }

  logOperation(operation, data) {
    if (this.connection) {
      this.connection.send('asyncStorageOperation', {
        operation,
        ...data,
        timestamp: Date.now()
      });
    }
  }

  async getAllKeys() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      if (this.connection) {
        this.connection.send('asyncStorageKeys', {keys});
      }
      return keys;
    } catch (error) {
      console.error('Error getting keys:', error);
    }
  }
}

// Configurar plugin
Flipper.addPlugin({
  getId() {
    return 'AsyncStorage';
  },
  onConnect(connection) {
    this.connection = connection;
    this.asyncStoragePlugin = new AsyncStoragePlugin();
    
    // Enviar todas las keys al conectar
    this.asyncStoragePlugin.getAllKeys();
  },
  onDisconnect() {
    this.connection = null;
    this.asyncStoragePlugin = null;
  },
});
```

---

## 🎯 Ejercicios de Evaluación

### **Ejercicio 1: Configuración Completa**
- Configurar Flipper con todos los plugins básicos
- Verificar que la conexión funcione correctamente
- Documentar la configuración

### **Ejercicio 2: Plugin Personalizado**
- Crear un plugin personalizado para monitorear un estado específico
- Implementar logging de operaciones
- Verificar que los datos se muestren en Flipper

### **Ejercicio 3: Performance Profiling**
- Implementar profiling con React DevTools
- Identificar componentes costosos
- Optimizar el rendimiento basado en los resultados

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [Flipper Documentation](https://fbflipper.com/)
- [React DevTools](https://reactjs.org/blog/2019/08/15/new-react-devtools.html)
- [Metro Configuration](https://facebook.github.io/metro/docs/configuration)

### **Plugins Útiles**
- [Flipper Plugins](https://github.com/facebook/flipper/tree/master/desktop/plugins)
- [React Native Flipper](https://github.com/facebook/flipper/tree/master/react-native)

### **Mejores Prácticas**
- Configurar Flipper solo en builds de desarrollo
- Usar plugins específicos para tu caso de uso
- Documentar configuraciones personalizadas
- Mantener actualizadas las versiones de Flipper

---

## 🚀 Siguiente Clase

En la próxima clase aprenderás sobre **Metro y Bundling Avanzado**, donde profundizarás en la configuración avanzada de Metro bundler, resolvers personalizados, code splitting y optimización de bundles.

---

**💡 Consejo**: Flipper y React DevTools son herramientas esenciales para el debugging profesional. Invierte tiempo en dominarlas, ya que te ahorrarán horas de debugging manual.
