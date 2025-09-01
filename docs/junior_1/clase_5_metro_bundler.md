# 📚 Clase 5: Metro Bundler

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 4: Primer Proyecto](clase_4_primer_proyecto.md)
- **➡️ Siguiente**: [README del Módulo](README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Comprender qué es Metro Bundler** y cómo funciona
2. **Configurar Metro** para tu proyecto específico
3. **Optimizar el bundling** para mejor rendimiento
4. **Resolver problemas comunes** de Metro
5. **Personalizar la configuración** según tus necesidades

---

## 📚 Contenido Teórico

### **¿Qué es Metro Bundler?**

**Metro** es el bundler oficial de React Native que se encarga de:

#### **Funcionalidades Principales:**
- **Empaquetado de código**: Combina todos los archivos JavaScript en uno
- **Hot Reloading**: Recarga automática de cambios en tiempo real
- **Transformación de código**: Convierte JSX, TypeScript, ES6+ a código compatible
- **Resolución de módulos**: Encuentra y conecta dependencias
- **Optimización**: Minimiza y optimiza el código para producción

#### **Ventajas de Metro:**
- **Rapidez**: Optimizado para aplicaciones móviles
- **Flexibilidad**: Configurable para diferentes necesidades
- **Integración**: Perfectamente integrado con React Native
- **Performance**: Bundling incremental y caching inteligente

---

## 💻 Implementación Práctica

### **1. Configuración Básica de Metro**

```javascript:metro.config.js
// Configuración básica de Metro para React Native
const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');

/**
 * Configuración de Metro para React Native
 * @see https://facebook.github.io/metro/docs/configuration
 */
const config = {
  // Configuración del resolver
  resolver: {
    // Extensiones de archivos que Metro puede resolver
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json'],
    
    // Extensiones de plataforma específicas
    platformExts: ['ios', 'android', 'native', 'web'],
    
    // Alias para importaciones
    alias: {
      '@components': './src/components',
      '@screens': './src/screens',
      '@navigation': './src/navigation',
      '@services': './src/services',
      '@hooks': './src/hooks',
      '@utils': './src/utils',
      '@types': './src/types',
      '@assets': './src/assets',
    },
    
    // Configuración de resolución de módulos
    resolverMainFields: ['react-native', 'browser', 'main'],
    
    // Módulos que no deben ser procesados
    blacklistRE: /blacklisted-file\.js$/,
    
    // Módulos que deben ser procesados
    whitelistRE: /whitelisted-file\.js$/,
  },

  // Configuración del transformer
  transformer: {
    // Habilitar transformación de TypeScript
    babelTransformerPath: require.resolve('react-native-svg-transformer'),
    
    // Configuración de Babel
    babelTransformerPath: require.resolve('metro-react-native-babel-transformer'),
    
    // Configuración de minificación
    minifierConfig: {
      keep_fnames: true,
      mangle: {
        keep_fnames: true,
      },
    },
    
    // Configuración de source maps
    generateSourceMaps: __DEV__,
    
    // Configuración de assets
    assetPlugins: ['react-native-svg-asset-plugin'],
  },

  // Configuración del servidor
  server: {
    // Puerto del servidor de desarrollo
    port: 8081,
    
    // Host del servidor
    host: 'localhost',
    
    // Configuración de CORS
    cors: true,
    
    // Configuración de HTTPS
    https: false,
    
    // Configuración de proxy
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
    },
  },

  // Configuración de watchman
  watchFolders: [
    // Directorios adicionales para observar cambios
    './src',
    './shared',
    './node_modules',
  ],

  // Configuración de cache
  cacheStores: [
    {
      name: 'metro-cache',
      type: 'file',
      options: {
        directory: './metro-cache',
        maxAge: 24 * 60 * 60 * 1000, // 24 horas
      },
    },
  ],

  // Configuración de optimización
  optimization: {
    // Habilitar tree shaking
    treeShaking: true,
    
    // Habilitar minificación
    minify: !__DEV__,
    
    // Configuración de chunks
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 244000,
    },
  },
};

// Configuración específica para desarrollo
const developmentConfig = {
  transformer: {
    ...config.transformer,
    // Habilitar source maps en desarrollo
    generateSourceMaps: true,
    // Configuración de Babel para desarrollo
    babelTransformerPath: require.resolve('metro-react-native-babel-transformer'),
  },
  server: {
    ...config.server,
    // Configuración adicional para desarrollo
    enableDebugger: true,
    enableHotReload: true,
  },
};

// Configuración específica para producción
const productionConfig = {
  transformer: {
    ...config.transformer,
    // Deshabilitar source maps en producción
    generateSourceMaps: false,
    // Configuración de Babel para producción
    babelTransformerPath: require.resolve('metro-react-native-babel-transformer'),
  },
  optimization: {
    ...config.optimization,
    // Habilitar todas las optimizaciones en producción
    minify: true,
    treeShaking: true,
  },
};

// Exportar configuración según el entorno
module.exports = mergeConfig(
  getDefaultConfig(__dirname),
  config,
  __DEV__ ? developmentConfig : productionConfig
);
```

### **2. Configuración Avanzada de Metro**

```javascript:metro.config.advanced.js
// Configuración avanzada de Metro con plugins y optimizaciones
const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');
const path = require('path');

// Plugin personalizado para Metro
class MetroCustomPlugin {
  constructor(options = {}) {
    this.options = options;
  }

  // Método que se ejecuta antes del bundling
  preBundle(moduleGraph) {
    console.log('🔧 Pre-bundle: Analizando módulos...');
    
    // Analizar dependencias
    const dependencies = this.analyzeDependencies(moduleGraph);
    
    // Optimizar imports
    this.optimizeImports(dependencies);
    
    return moduleGraph;
  }

  // Método que se ejecuta después del bundling
  postBundle(bundle) {
    console.log('✅ Post-bundle: Optimizando bundle...');
    
    // Optimizar código
    bundle.code = this.optimizeCode(bundle.code);
    
    // Generar estadísticas
    this.generateStats(bundle);
    
    return bundle;
  }

  // Analizar dependencias del módulo
  analyzeDependencies(moduleGraph) {
    const dependencies = new Map();
    
    moduleGraph.traverse((module) => {
      if (module.dependencies) {
        dependencies.set(module.path, module.dependencies);
      }
    });
    
    return dependencies;
  }

  // Optimizar imports
  optimizeImports(dependencies) {
    dependencies.forEach((deps, modulePath) => {
      // Implementar lógica de optimización de imports
      console.log(`📦 Optimizando imports para: ${modulePath}`);
    });
  }

  // Optimizar código del bundle
  optimizeCode(code) {
    // Implementar optimizaciones de código
    return code
      .replace(/console\.log\(/g, '__DEV__ && console.log(')
      .replace(/console\.warn\(/g, '__DEV__ && console.warn(')
      .replace(/console\.error\(/g, '__DEV__ && console.error(');
  }

  // Generar estadísticas del bundle
  generateStats(bundle) {
    const stats = {
      size: bundle.code.length,
      modules: bundle.modules?.length || 0,
      timestamp: new Date().toISOString(),
    };
    
    console.log('📊 Estadísticas del bundle:', stats);
  }
}

// Configuración avanzada
const advancedConfig = {
  // Configuración del resolver avanzada
  resolver: {
    // Resolver personalizado para módulos
    resolveRequest: (context, moduleName, platform) => {
      // Resolver módulos personalizados
      if (moduleName.startsWith('@custom/')) {
        const customPath = path.resolve(__dirname, 'src', moduleName.replace('@custom/', ''));
        return {
          filePath: customPath,
          type: 'sourceFile',
        };
      }
      
      // Resolver módulos de plataforma
      if (platform) {
        const platformModule = `${moduleName}.${platform}`;
        try {
          const platformPath = context.resolveRequest(context, platformModule, platform);
          if (platformPath) {
            return platformPath;
          }
        } catch (error) {
          // Continuar con resolución normal
        }
      }
      
      // Resolución por defecto
      return context.resolveRequest(context, moduleName, platform);
    },
    
    // Configuración de alias avanzada
    alias: {
      // Alias para componentes
      '@components': './src/components',
      '@screens': './src/screens',
      '@navigation': './src/navigation',
      '@services': './src/services',
      '@hooks': './src/hooks',
      '@utils': './src/utils',
      '@types': './src/types',
      '@assets': './src/assets',
      
      // Alias para librerías externas
      'react-native-vector-icons': 'react-native-vector-icons/dist',
      'react-native-gesture-handler': 'react-native-gesture-handler/lib',
      
      // Alias para módulos personalizados
      '@custom': './src/custom',
      '@shared': './shared',
      '@config': './config',
    },
    
    // Configuración de resolución de módulos
    resolverMainFields: ['react-native', 'browser', 'main'],
    
    // Extensiones de archivos
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json', 'svg'],
    
    // Configuración de plataforma
    platformExts: ['ios', 'android', 'native', 'web'],
  },

  // Configuración del transformer avanzada
  transformer: {
    // Transformador personalizado para SVG
    babelTransformerPath: require.resolve('react-native-svg-transformer'),
    
    // Configuración de Babel
    babelTransformerPath: require.resolve('metro-react-native-babel-transformer'),
    
    // Configuración de minificación
    minifierConfig: {
      keep_fnames: true,
      mangle: {
        keep_fnames: true,
        reserved: ['__DEV__', '__METRO_GLOBAL_PREFIX__'],
      },
      compress: {
        drop_console: !__DEV__,
        drop_debugger: !__DEV__,
        pure_funcs: __DEV__ ? [] : ['console.log', 'console.warn'],
      },
    },
    
    // Configuración de source maps
    generateSourceMaps: __DEV__,
    
    // Configuración de assets
    assetPlugins: [
      'react-native-svg-asset-plugin',
      'react-native-asset-plugin',
    ],
  },

  // Configuración del servidor avanzada
  server: {
    // Puerto del servidor
    port: process.env.METRO_PORT || 8081,
    
    // Host del servidor
    host: process.env.METRO_HOST || 'localhost',
    
    // Configuración de CORS
    cors: true,
    
    // Configuración de HTTPS
    https: process.env.METRO_HTTPS === 'true',
    
    // Configuración de proxy
    proxy: {
      '/api': {
        target: process.env.API_URL || 'http://localhost:3000',
        changeOrigin: true,
        secure: false,
        pathRewrite: {
          '^/api': '',
        },
      },
      '/assets': {
        target: process.env.ASSETS_URL || 'http://localhost:3001',
        changeOrigin: true,
      },
    },
    
    // Configuración de middleware
    middleware: [
      // Middleware personalizado
      (req, res, next) => {
        // Log de requests
        console.log(`${req.method} ${req.url}`);
        next();
      },
    ],
  },

  // Configuración de watchman
  watchFolders: [
    // Directorios para observar cambios
    './src',
    './shared',
    './config',
    './assets',
    
    // Directorios de node_modules específicos
    './node_modules/react-native',
    './node_modules/@react-navigation',
  ],

  // Configuración de cache avanzada
  cacheStores: [
    {
      name: 'metro-cache',
      type: 'file',
      options: {
        directory: './metro-cache',
        maxAge: 24 * 60 * 60 * 1000, // 24 horas
        maxSize: 100 * 1024 * 1024, // 100 MB
      },
    },
    {
      name: 'metro-memory-cache',
      type: 'memory',
      options: {
        maxSize: 50 * 1024 * 1024, // 50 MB
      },
    },
  ],

  // Configuración de optimización avanzada
  optimization: {
    // Habilitar tree shaking
    treeShaking: true,
    
    // Habilitar minificación
    minify: !__DEV__,
    
    // Configuración de chunks
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 244000,
      cacheGroups: {
        vendor: {
          test: /[\\\\/]node_modules[\\\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true,
        },
      },
    },
    
    // Configuración de concatenación de módulos
    concatenateModules: true,
    
    // Configuración de side effects
    sideEffects: false,
  },

  // Plugins personalizados
  plugins: [
    new MetroCustomPlugin({
      enableOptimization: !__DEV__,
      enableStats: true,
    }),
  ],
};

// Exportar configuración
module.exports = mergeConfig(
  getDefaultConfig(__dirname),
  advancedConfig
);
```

### **3. Script de Gestión de Metro**

```javascript:src/scripts/metroManager.js
// Script para gestionar Metro Bundler
import { execSync, spawn } from 'child_process';
import { existsSync, readFileSync, writeFileSync } from 'fs';
import { join } from 'path';

class MetroManager {
  constructor(projectPath) {
    this.projectPath = projectPath;
    this.metroConfigPath = join(projectPath, 'metro.config.js');
    this.packageJsonPath = join(projectPath, 'package.json');
  }

  // Iniciar Metro Bundler
  startMetro(options = {}) {
    const {
      port = 8081,
      host = 'localhost',
      resetCache = false,
      maxWorkers = 4,
      transformer = 'metro-react-native-babel-transformer',
    } = options;

    console.log('🚀 Iniciando Metro Bundler...');
    console.log(`📍 Puerto: ${port}`);
    console.log(`🌐 Host: ${host}`);
    console.log(`🔄 Reset Cache: ${resetCache}`);
    console.log(`👥 Max Workers: ${maxWorkers}`);

    const args = [
      'react-native',
      'start',
      '--port',
      port.toString(),
      '--host',
      host,
      '--max-workers',
      maxWorkers.toString(),
    ];

    if (resetCache) {
      args.push('--reset-cache');
    }

    if (transformer) {
      args.push('--transformer', transformer);
    }

    try {
      const metroProcess = spawn('npx', args, {
        cwd: this.projectPath,
        stdio: 'inherit',
        env: {
          ...process.env,
          METRO_PORT: port.toString(),
          METRO_HOST: host,
        },
      });

      metroProcess.on('error', (error) => {
        console.error('❌ Error al iniciar Metro:', error.message);
      });

      metroProcess.on('close', (code) => {
        console.log(`🔚 Metro Bundler cerrado con código: ${code}`);
      });

      return metroProcess;
    } catch (error) {
      console.error('❌ Error al iniciar Metro:', error.message);
      return null;
    }
  }

  // Detener Metro Bundler
  stopMetro() {
    console.log('🛑 Deteniendo Metro Bundler...');
    
    try {
      // En Windows
      if (process.platform === 'win32') {
        execSync('taskkill /f /im node.exe', { stdio: 'inherit' });
      } else {
        // En Unix/Linux/macOS
        execSync('pkill -f "react-native start"', { stdio: 'inherit' });
      }
      
      console.log('✅ Metro Bundler detenido');
    } catch (error) {
      console.log('⚠️  No se encontraron procesos de Metro ejecutándose');
    }
  }

  // Limpiar cache de Metro
  clearCache() {
    console.log('🧹 Limpiando cache de Metro...');
    
    try {
      // Limpiar cache de Metro
      execSync('npx react-native start --reset-cache', {
        cwd: this.projectPath,
        stdio: 'inherit',
      });
      
      // Limpiar cache de npm
      execSync('npm cache clean --force', {
        cwd: this.projectPath,
        stdio: 'inherit',
      });
      
      // Limpiar node_modules (opcional)
      console.log('💡 Para limpiar completamente, considera eliminar node_modules y reinstalar');
      
      console.log('✅ Cache limpiado');
    } catch (error) {
      console.error('❌ Error al limpiar cache:', error.message);
    }
  }

  // Verificar configuración de Metro
  checkMetroConfig() {
    console.log('🔍 Verificando configuración de Metro...');
    
    if (!existsSync(this.metroConfigPath)) {
      console.log('⚠️  No se encontró metro.config.js');
      return false;
    }

    try {
      const config = readFileSync(this.metroConfigPath, 'utf8');
      
      // Verificar configuraciones importantes
      const checks = [
        { name: 'Resolver configurado', pattern: /resolver:/ },
        { name: 'Transformer configurado', pattern: /transformer:/ },
        { name: 'Server configurado', pattern: /server:/ },
        { name: 'Watchman configurado', pattern: /watchFolders:/ },
      ];

      checks.forEach(check => {
        if (check.pattern.test(config)) {
          console.log(`✅ ${check.name}`);
        } else {
          console.log(`⚠️  ${check.name} - No encontrado`);
        }
      });

      return true;
    } catch (error) {
      console.error('❌ Error al leer configuración:', error.message);
      return false;
    }
  }

  // Optimizar configuración de Metro
  optimizeMetroConfig() {
    console.log('⚡ Optimizando configuración de Metro...');
    
    if (!existsSync(this.metroConfigPath)) {
      console.log('📝 Creando metro.config.js optimizado...');
      this.createOptimizedConfig();
      return;
    }

    try {
      const config = readFileSync(this.metroConfigPath, 'utf8');
      
      // Agregar optimizaciones si no existen
      let optimizedConfig = config;
      
      if (!config.includes('optimization:')) {
        optimizedConfig += `
  
  // Configuración de optimización
  optimization: {
    treeShaking: true,
    minify: !__DEV__,
    concatenateModules: true,
  },`;
      }

      if (!config.includes('cacheStores:')) {
        optimizedConfig += `
  
  // Configuración de cache
  cacheStores: [
    {
      name: 'metro-cache',
      type: 'file',
      options: {
        directory: './metro-cache',
        maxAge: 24 * 60 * 60 * 1000,
      },
    },
  ],`;
      }

      writeFileSync(this.metroConfigPath, optimizedConfig);
      console.log('✅ Configuración optimizada');
    } catch (error) {
      console.error('❌ Error al optimizar configuración:', error.message);
    }
  }

  // Crear configuración optimizada
  createOptimizedConfig() {
    const optimizedConfig = `const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');

const config = {
  resolver: {
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json'],
    platformExts: ['ios', 'android', 'native', 'web'],
    alias: {
      '@components': './src/components',
      '@screens': './src/screens',
      '@navigation': './src/navigation',
      '@services': './src/services',
      '@hooks': './src/hooks',
      '@utils': './src/utils',
      '@types': './src/types',
      '@assets': './src/assets',
    },
  },
  
  transformer: {
    babelTransformerPath: require.resolve('metro-react-native-babel-transformer'),
    generateSourceMaps: __DEV__,
  },
  
  server: {
    port: 8081,
    host: 'localhost',
    cors: true,
  },
  
  watchFolders: ['./src', './shared'],
  
  optimization: {
    treeShaking: true,
    minify: !__DEV__,
    concatenateModules: true,
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);`;

    try {
      writeFileSync(this.metroConfigPath, optimizedConfig);
      console.log('✅ metro.config.js creado con configuración optimizada');
    } catch (error) {
      console.error('❌ Error al crear configuración:', error.message);
    }
  }

  // Generar estadísticas de Metro
  generateStats() {
    console.log('📊 Generando estadísticas de Metro...');
    
    try {
      // Verificar tamaño del bundle
      const bundlePath = join(this.projectPath, 'android', 'app', 'src', 'main', 'assets');
      
      if (existsSync(bundlePath)) {
        const stats = execSync(`du -sh "${bundlePath}"`, { encoding: 'utf8' });
        console.log(`📦 Tamaño del bundle: ${stats.trim()}`);
      }
      
      // Verificar dependencias
      const packageJson = JSON.parse(readFileSync(this.packageJsonPath, 'utf8'));
      const dependencies = Object.keys(packageJson.dependencies || {}).length;
      const devDependencies = Object.keys(packageJson.devDependencies || {}).length;
      
      console.log(`📚 Dependencias: ${dependencies}`);
      console.log(`🔧 Dependencias de desarrollo: ${devDependencies}`);
      
      // Verificar configuración
      this.checkMetroConfig();
      
    } catch (error) {
      console.error('❌ Error al generar estadísticas:', error.message);
    }
  }

  // Ejecutar comando personalizado
  runCommand(command, args = []) {
    console.log(`🚀 Ejecutando comando: ${command} ${args.join(' ')}`);
    
    try {
      execSync(`${command} ${args.join(' ')}`, {
        cwd: this.projectPath,
        stdio: 'inherit',
      });
    } catch (error) {
      console.error(`❌ Error al ejecutar comando: ${error.message}`);
    }
  }
}

// Función para ejecutar el gestor
export const runMetroManager = (projectPath, action, options = {}) => {
  const manager = new MetroManager(projectPath);
  
  switch (action) {
    case 'start':
      return manager.startMetro(options);
    case 'stop':
      return manager.stopMetro();
    case 'clear-cache':
      return manager.clearCache();
    case 'check-config':
      return manager.checkMetroConfig();
    case 'optimize':
      return manager.optimizeMetroConfig();
    case 'stats':
      return manager.generateStats();
    default:
      console.log('❌ Acción no válida. Acciones disponibles: start, stop, clear-cache, check-config, optimize, stats');
  }
};

// Ejecutar si se llama directamente
if (require.main === module) {
  const [,, projectPath, action, ...args] = process.argv;
  
  if (!projectPath || !action) {
    console.error('❌ Uso: node metroManager.js <ruta-proyecto> <accion> [opciones]');
    process.exit(1);
  }
  
  runMetroManager(projectPath, action, {});
}
```

---

## 🧪 Casos de Uso Prácticos

### **1. Iniciar Metro con Configuración Personalizada**

```bash
# Iniciar Metro en puerto específico
npx react-native start --port 8082

# Iniciar Metro con reset de cache
npx react-native start --reset-cache

# Iniciar Metro con configuración personalizada
npx react-native start --max-workers 8 --transformer metro-react-native-babel-transformer
```

### **2. Script de Gestión de Metro**

```bash
# Usar el gestor de Metro
node metroManager.js ./MiApp start
node metroManager.js ./MiApp stop
node metroManager.js ./MiApp clear-cache
node metroManager.js ./MiApp optimize
node metroManager.js ./MiApp stats
```

### **3. Configuración para Diferentes Entornos**

```javascript:metro.config.environment.js
// Configuración de Metro para diferentes entornos
const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');

// Configuración base
const baseConfig = {
  resolver: {
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json'],
    alias: {
      '@components': './src/components',
      '@screens': './src/screens',
    },
  },
};

// Configuración de desarrollo
const developmentConfig = {
  transformer: {
    generateSourceMaps: true,
    minifierConfig: {
      keep_fnames: true,
    },
  },
  server: {
    port: 8081,
    enableDebugger: true,
  },
};

// Configuración de producción
const productionConfig = {
  transformer: {
    generateSourceMaps: false,
    minifierConfig: {
      keep_fnames: false,
      drop_console: true,
    },
  },
  optimization: {
    minify: true,
    treeShaking: true,
  },
};

// Configuración de testing
const testingConfig = {
  resolver: {
    ...baseConfig.resolver,
    sourceExts: [...baseConfig.resolver.sourceExts, 'test.js', 'spec.js'],
  },
  transformer: {
    generateSourceMaps: true,
  },
};

// Exportar configuración según el entorno
const getConfig = () => {
  const env = process.env.NODE_ENV || 'development';
  
  switch (env) {
    case 'production':
      return mergeConfig(getDefaultConfig(__dirname), baseConfig, productionConfig);
    case 'testing':
      return mergeConfig(getDefaultConfig(__dirname), baseConfig, testingConfig);
    default:
      return mergeConfig(getDefaultConfig(__dirname), baseConfig, developmentConfig);
  }
};

module.exports = getConfig();
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Configuración Básica**
Crea un metro.config.js básico con alias para tu proyecto.

### **Ejercicio 2: Optimización de Metro**
Configura Metro para optimizar el rendimiento en desarrollo y producción.

### **Ejercicio 3: Resolución de Problemas**
Identifica y resuelve al menos 3 problemas comunes de Metro.

### **Ejercicio 4: Configuración Avanzada**
Implementa una configuración de Metro con plugins personalizados.

---

## 🎯 Proyecto de la Clase

### **Gestor de Metro Personalizado**

Crea una herramienta que permita gestionar Metro Bundler con configuraciones personalizadas.

**Requisitos:**
- Interfaz para iniciar/detener Metro
- Configuración de puertos y hosts
- Gestión de cache
- Optimización automática
- Estadísticas de rendimiento

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [Metro Documentation](https://facebook.github.io/metro/)
- [Metro Configuration](https://facebook.github.io/metro/docs/configuration)
- [Metro API Reference](https://facebook.github.io/metro/docs/api)

### **Artículos:**
- [Metro Bundler Deep Dive](https://medium.com/@reactnative/metro-bundler-deep-dive-4c5b5b5b5b5b)
- [Optimizing Metro for React Native](https://medium.com/@reactnative/optimizing-metro-for-react-native-5b5b5b5b5b5b)

### **Herramientas:**
- [Metro CLI](https://www.npmjs.com/package/metro-cli)
- [Metro React Native Babel Preset](https://www.npmjs.com/package/metro-react-native-babel-preset)

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Funcionamiento de Metro** Bundler
2. **Configuración básica y avanzada** de Metro
3. **Optimización del bundling** para mejor rendimiento
4. **Gestión de cache** y resolución de problemas
5. **Personalización** de la configuración

### **🚀 Próximos Pasos:**
- Aprender sobre componentes nativos
- Crear interfaces más complejas
- Implementar navegación en la aplicación

### **💡 Conceptos Clave:**
- **Bundler**: Empaquetador de código
- **Hot Reloading**: Recarga automática de cambios
- **Transformer**: Transformador de código
- **Resolver**: Resolvedor de módulos
- **Cache**: Almacenamiento temporal de archivos

---

**🎯 Objetivo**: Dominar Metro Bundler para optimizar el desarrollo y rendimiento de tus aplicaciones React Native.

**💡 Consejo**: Una configuración optimizada de Metro puede mejorar significativamente el tiempo de desarrollo y el rendimiento de tu aplicación.
