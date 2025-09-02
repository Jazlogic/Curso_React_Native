# 🔧 Clase 2: Metro y Bundling Avanzado

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase dominarás la configuración avanzada de Metro bundler, resolvers personalizados, transformaciones, code splitting y optimización de bundles. Aprenderás a personalizar completamente el proceso de bundling para maximizar el rendimiento de tu aplicación.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Configurar** Metro bundler avanzado
2. **Crear resolvers** personalizados
3. **Implementar** transformaciones custom
4. **Optimizar** code splitting y lazy loading
5. **Analizar** y optimizar bundle size
6. **Crear plugins** personalizados para Metro

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Arquitectura de Metro**

#### **¿Qué es Metro?**
Metro es el bundler oficial de React Native desarrollado por Facebook. Es responsable de transformar, resolver y empaquetar tu código JavaScript y assets.

#### **Componentes Principales**
- **Resolver**: Encuentra y resuelve módulos
- **Transformer**: Transforma código (Babel, TypeScript, etc.)
- **Serializer**: Genera el bundle final
- **Watcher**: Monitorea cambios en archivos

#### **Flujo de Metro**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Resolver  │───►│ Transformer │───►│ Serializer  │───►│   Bundle    │
│             │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### **2. Configuración Avanzada de Metro**

#### **metro.config.js Completo**
```javascript
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');
const path = require('path');

// Configuración personalizada
const customConfig = {
  resolver: {
    // Extensiones de archivo soportadas
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json', 'svg'],
    assetExts: ['png', 'jpg', 'jpeg', 'gif', 'webp', 'svg'],
    
    // Alias para imports
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@screens': path.resolve(__dirname, 'src/screens'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@assets': path.resolve(__dirname, 'src/assets'),
    },
    
    // Resolvers personalizados
    resolverMainFields: ['react-native', 'browser', 'main'],
    
    // Plataformas soportadas
    platforms: ['ios', 'android', 'native', 'web'],
  },
  
  transformer: {
    // Configuración de Babel
    babelTransformerPath: require.resolve('react-native-svg-transformer'),
    
    // Configuración de assets
    assetPlugins: ['react-native-svg-asset-plugin'],
    
    // Minificación
    minifierConfig: {
      keep_fnames: true,
      mangle: {
        keep_fnames: true,
      },
    },
  },
  
  serializer: {
    // Configuración de serialización
    getModulesRunBeforeMainModule: () => [
      require.resolve('react-native/Libraries/Core/InitializeCore'),
    ],
    
    // Custom serializer
    customSerializer: require.resolve('./custom-serializer.js'),
  },
  
  // Configuración de cache
  cacheStores: [
    {
      name: 'filesystem',
      path: path.resolve(__dirname, '.metro-cache'),
    },
  ],
  
  // Configuración de watchman
  watchFolders: [
    path.resolve(__dirname, 'node_modules'),
    path.resolve(__dirname, 'src'),
  ],
};

module.exports = mergeConfig(getDefaultConfig(__dirname), customConfig);
```

### **3. Resolvers Personalizados**

#### **Resolver para Módulos Nativos**
```javascript
// custom-resolver.js
const path = require('path');

class CustomResolver {
  constructor(options) {
    this.options = options;
  }

  resolve(context, moduleName, platform) {
    // Resolver módulos nativos por plataforma
    if (moduleName.startsWith('react-native-')) {
      const platformModule = `${moduleName}/${platform}`;
      try {
        return context.resolveRequest(context, platformModule, platform);
      } catch (error) {
        // Fallback al módulo base
        return context.resolveRequest(context, moduleName, platform);
      }
    }

    // Resolver módulos con alias
    if (moduleName.startsWith('@/')) {
      const aliasPath = moduleName.replace('@/', 'src/');
      return context.resolveRequest(context, aliasPath, platform);
    }

    // Resolver por defecto
    return context.resolveRequest(context, moduleName, platform);
  }
}

module.exports = CustomResolver;
```

#### **Resolver para Assets**
```javascript
// asset-resolver.js
const path = require('path');
const fs = require('fs');

class AssetResolver {
  constructor(options) {
    this.options = options;
  }

  resolve(context, moduleName, platform) {
    // Resolver assets con diferentes densidades
    if (this.isAsset(moduleName)) {
      const assetPath = this.resolveAsset(moduleName, platform);
      if (assetPath) {
        return {
          type: 'asset',
          path: assetPath,
          name: path.basename(assetPath, path.extname(assetPath)),
        };
      }
    }

    return null;
  }

  isAsset(moduleName) {
    const assetExtensions = ['.png', '.jpg', '.jpeg', '.gif', '.webp', '.svg'];
    return assetExtensions.some(ext => moduleName.endsWith(ext));
  }

  resolveAsset(moduleName, platform) {
    const basePath = path.dirname(moduleName);
    const baseName = path.basename(moduleName, path.extname(moduleName));
    const extension = path.extname(moduleName);

    // Buscar assets con diferentes densidades
    const densities = ['@3x', '@2x', '@1x', ''];
    
    for (const density of densities) {
      const assetPath = path.join(basePath, `${baseName}${density}${extension}`);
      if (fs.existsSync(assetPath)) {
        return assetPath;
      }
    }

    return null;
  }
}

module.exports = AssetResolver;
```

### **4. Transformaciones Personalizadas**

#### **Transformer para SVG**
```javascript
// svg-transformer.js
const {transform} = require('@babel/core');
const svgToComponent = require('svg-to-component');

class SVGTransformer {
  constructor(options) {
    this.options = options;
  }

  transform({src, filename, options}) {
    // Convertir SVG a componente React
    const componentCode = svgToComponent(src, {
      componentName: this.getComponentName(filename),
      typescript: filename.endsWith('.tsx'),
    });

    // Transformar con Babel
    const result = transform(componentCode, {
      filename,
      presets: ['@babel/preset-react', '@babel/preset-typescript'],
      plugins: ['@babel/plugin-proposal-class-properties'],
    });

    return {
      ast: result.ast,
      code: result.code,
      map: result.map,
    };
  }

  getComponentName(filename) {
    const baseName = path.basename(filename, path.extname(filename));
    return baseName.charAt(0).toUpperCase() + baseName.slice(1);
  }
}

module.exports = SVGTransformer;
```

#### **Transformer para TypeScript**
```javascript
// typescript-transformer.js
const {transform} = require('@babel/core');
const typescript = require('typescript');

class TypeScriptTransformer {
  constructor(options) {
    this.options = options;
  }

  transform({src, filename, options}) {
    // Compilar TypeScript
    const result = typescript.transpile(src, {
      target: typescript.ScriptTarget.ES2017,
      module: typescript.ModuleKind.ESNext,
      jsx: typescript.JsxEmit.React,
      allowSyntheticDefaultImports: true,
      esModuleInterop: true,
    });

    // Transformar con Babel
    const babelResult = transform(result, {
      filename,
      presets: ['@babel/preset-react'],
      plugins: ['@babel/plugin-proposal-class-properties'],
    });

    return {
      ast: babelResult.ast,
      code: babelResult.code,
      map: babelResult.map,
    };
  }
}

module.exports = TypeScriptTransformer;
```

### **5. Code Splitting y Lazy Loading**

#### **Configuración de Code Splitting**
```javascript
// metro.config.js
const customConfig = {
  serializer: {
    // Configurar code splitting
    createModuleIdFactory: () => {
      let nextId = 0;
      return (path) => {
        // Crear IDs únicos para módulos
        return nextId++;
      };
    },
    
    // Configurar chunks
    getRunModuleStatement: (moduleId) => {
      return `__r(${moduleId});`;
    },
  },
};
```

#### **Lazy Loading de Componentes**
```javascript
// LazyComponent.js
import React, {lazy, Suspense} from 'react';
import {View, Text, ActivityIndicator} from 'react-native';

// Lazy loading de componentes
const LazyScreen = lazy(() => import('./LazyScreen'));
const LazyModal = lazy(() => import('./LazyModal'));

function LazyComponent() {
  return (
    <View>
      <Suspense fallback={<ActivityIndicator />}>
        <LazyScreen />
      </Suspense>
      
      <Suspense fallback={<Text>Loading modal...</Text>}>
        <LazyModal />
      </Suspense>
    </View>
  );
}

export default LazyComponent;
```

#### **Dynamic Imports**
```javascript
// DynamicImports.js
import React, {useState, useEffect} from 'react';

function DynamicImports() {
  const [Component, setComponent] = useState(null);
  const [loading, setLoading] = useState(false);

  const loadComponent = async (componentName) => {
    setLoading(true);
    try {
      const module = await import(`./components/${componentName}`);
      setComponent(() => module.default);
    } catch (error) {
      console.error('Error loading component:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <View>
      <Button title="Load Component" onPress={() => loadComponent('MyComponent')} />
      {loading && <ActivityIndicator />}
      {Component && <Component />}
    </View>
  );
}
```

### **6. Bundle Analysis y Optimización**

#### **Bundle Analyzer**
```javascript
// bundle-analyzer.js
const fs = require('fs');
const path = require('path');

class BundleAnalyzer {
  constructor(options) {
    this.options = options;
  }

  analyze(bundlePath) {
    const bundle = fs.readFileSync(bundlePath, 'utf8');
    const modules = this.extractModules(bundle);
    const stats = this.generateStats(modules);
    
    return {
      totalSize: bundle.length,
      moduleCount: modules.length,
      stats,
      recommendations: this.generateRecommendations(stats),
    };
  }

  extractModules(bundle) {
    // Extraer información de módulos del bundle
    const moduleRegex = /__d\((\d+),\[([^\]]*)\],function\([^)]*\)\{([^}]*)\}/g;
    const modules = [];
    let match;

    while ((match = moduleRegex.exec(bundle)) !== null) {
      modules.push({
        id: match[1],
        dependencies: match[2].split(',').filter(Boolean),
        code: match[3],
        size: match[0].length,
      });
    }

    return modules;
  }

  generateStats(modules) {
    const stats = {
      totalModules: modules.length,
      totalSize: modules.reduce((sum, module) => sum + module.size, 0),
      averageSize: 0,
      largestModules: [],
      duplicateModules: [],
    };

    stats.averageSize = stats.totalSize / stats.totalModules;
    stats.largestModules = modules
      .sort((a, b) => b.size - a.size)
      .slice(0, 10);

    return stats;
  }

  generateRecommendations(stats) {
    const recommendations = [];

    if (stats.averageSize > 1000) {
      recommendations.push('Consider code splitting for large modules');
    }

    if (stats.duplicateModules.length > 0) {
      recommendations.push('Remove duplicate modules');
    }

    return recommendations;
  }
}

module.exports = BundleAnalyzer;
```

---

## 🛠️ Contenido Práctico

### **Ejercicio 1: Configuración Avanzada de Metro**

#### **Objetivo**
Configurar Metro con resolvers personalizados y transformaciones.

#### **Pasos**
1. **Crear metro.config.js** avanzado
2. **Configurar alias** para imports
3. **Implementar resolvers** personalizados
4. **Configurar transformaciones** para SVG y TypeScript

#### **Código de Ejemplo**
```javascript
// metro.config.js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');
const path = require('path');

const customConfig = {
  resolver: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@screens': path.resolve(__dirname, 'src/screens'),
      '@utils': path.resolve(__dirname, 'src/utils'),
    },
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json', 'svg'],
    assetExts: ['png', 'jpg', 'jpeg', 'gif', 'webp'],
  },
  
  transformer: {
    babelTransformerPath: require.resolve('react-native-svg-transformer'),
    assetPlugins: ['react-native-svg-asset-plugin'],
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), customConfig);
```

### **Ejercicio 2: Resolver Personalizado**

#### **Objetivo**
Crear un resolver personalizado para módulos nativos.

#### **Código de Ejemplo**
```javascript
// custom-resolver.js
const path = require('path');

class CustomResolver {
  constructor(options) {
    this.options = options;
  }

  resolve(context, moduleName, platform) {
    // Resolver módulos nativos por plataforma
    if (moduleName.startsWith('react-native-')) {
      const platformModule = `${moduleName}/${platform}`;
      try {
        return context.resolveRequest(context, platformModule, platform);
      } catch (error) {
        return context.resolveRequest(context, moduleName, platform);
      }
    }

    // Resolver alias
    if (moduleName.startsWith('@/')) {
      const aliasPath = moduleName.replace('@/', 'src/');
      return context.resolveRequest(context, aliasPath, platform);
    }

    return context.resolveRequest(context, moduleName, platform);
  }
}

module.exports = CustomResolver;
```

### **Ejercicio 3: Code Splitting**

#### **Objetivo**
Implementar code splitting y lazy loading.

#### **Código de Ejemplo**
```javascript
// App.js
import React, {lazy, Suspense} from 'react';
import {View, ActivityIndicator} from 'react-native';

// Lazy loading de pantallas
const HomeScreen = lazy(() => import('./screens/HomeScreen'));
const ProfileScreen = lazy(() => import('./screens/ProfileScreen'));
const SettingsScreen = lazy(() => import('./screens/SettingsScreen'));

function App() {
  const [currentScreen, setCurrentScreen] = useState('home');

  const renderScreen = () => {
    switch (currentScreen) {
      case 'home':
        return <HomeScreen />;
      case 'profile':
        return <ProfileScreen />;
      case 'settings':
        return <SettingsScreen />;
      default:
        return <HomeScreen />;
    }
  };

  return (
    <View style={{flex: 1}}>
      <Suspense fallback={<ActivityIndicator />}>
        {renderScreen()}
      </Suspense>
    </View>
  );
}

export default App;
```

### **Ejercicio 4: Bundle Analysis**

#### **Objetivo**
Crear un script para analizar el bundle.

#### **Código de Ejemplo**
```javascript
// analyze-bundle.js
const fs = require('fs');
const path = require('path');

function analyzeBundle(bundlePath) {
  const bundle = fs.readFileSync(bundlePath, 'utf8');
  const size = bundle.length;
  
  console.log('Bundle Analysis:');
  console.log(`Total size: ${(size / 1024).toFixed(2)} KB`);
  
  // Analizar módulos
  const moduleRegex = /__d\((\d+),\[([^\]]*)\],function\([^)]*\)\{([^}]*)\}/g;
  const modules = [];
  let match;

  while ((match = moduleRegex.exec(bundle)) !== null) {
    modules.push({
      id: match[1],
      dependencies: match[2].split(',').filter(Boolean),
      size: match[0].length,
    });
  }

  console.log(`Module count: ${modules.length}`);
  console.log(`Average module size: ${(modules.reduce((sum, m) => sum + m.size, 0) / modules.length).toFixed(2)} bytes`);
  
  // Módulos más grandes
  const largestModules = modules
    .sort((a, b) => b.size - a.size)
    .slice(0, 5);
    
  console.log('\nLargest modules:');
  largestModules.forEach((module, index) => {
    console.log(`${index + 1}. Module ${module.id}: ${module.size} bytes`);
  });
}

// Ejecutar análisis
const bundlePath = path.join(__dirname, 'android/app/build/generated/assets/react/release/index.android.bundle');
analyzeBundle(bundlePath);
```

---

## 🎯 Ejercicios de Evaluación

### **Ejercicio 1: Configuración Completa**
- Configurar Metro con resolvers personalizados
- Implementar transformaciones para SVG
- Verificar que los alias funcionen correctamente

### **Ejercicio 2: Code Splitting**
- Implementar lazy loading de componentes
- Configurar code splitting en Metro
- Medir la mejora en el tiempo de carga

### **Ejercicio 3: Bundle Analysis**
- Crear script de análisis de bundle
- Identificar módulos más grandes
- Implementar optimizaciones basadas en el análisis

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [Metro Documentation](https://facebook.github.io/metro/)
- [Metro Configuration](https://facebook.github.io/metro/docs/configuration)
- [Metro Plugins](https://facebook.github.io/metro/docs/plugins)

### **Herramientas Útiles**
- [Bundle Analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)
- [Metro Bundle Analyzer](https://github.com/facebook/metro/tree/main/packages/metro-bundle-analyzer)

### **Mejores Prácticas**
- Usar alias para imports largos
- Implementar code splitting para pantallas grandes
- Analizar bundles regularmente
- Mantener actualizada la configuración de Metro

---

## 🚀 Siguiente Clase

En la próxima clase aprenderás sobre **Hermes Engine y Optimización**, donde profundizarás en el motor JavaScript de Hermes, su configuración y optimizaciones para mejorar el rendimiento de tu aplicación.

---

**💡 Consejo**: Metro es una herramienta poderosa que puede ser personalizada completamente. Invierte tiempo en entender su configuración para maximizar el rendimiento de tu aplicación.
