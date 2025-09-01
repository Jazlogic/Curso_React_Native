# Clase 1: Fundamentos de CI/CD 🚀

## Objetivos de la Clase
- Comprender los conceptos fundamentales de CI/CD
- Conocer las herramientas principales para automatización
- Entender los flujos de trabajo en el desarrollo móvil
- Implementar un pipeline básico de CI/CD

## Duración Estimada
**1.5 horas**

## Contenido Teórico

### 1. ¿Qué es CI/CD?

**CI (Continuous Integration)** es una práctica de desarrollo que consiste en integrar cambios de código de forma frecuente y automática. **CD (Continuous Deployment/Delivery)** se refiere a la automatización del proceso de despliegue.

```javascript
// Ejemplo de flujo CI/CD básico
const ciCdPipeline = {
  // 1. Integración Continua
  continuousIntegration: {
    trigger: 'git push',           // Se activa con cada push
    steps: [
      'install dependencies',      // Instalar dependencias
      'run tests',                 // Ejecutar tests
      'build project',             // Compilar proyecto
      'code quality checks'        // Verificar calidad del código
    ]
  },
  
  // 2. Despliegue Continuo
  continuousDeployment: {
    trigger: 'tests passed',       // Solo si pasan los tests
    steps: [
      'create build artifacts',    // Crear artefactos de build
      'deploy to staging',         // Desplegar en staging
      'run integration tests',     // Tests de integración
      'deploy to production'       // Desplegar en producción
    ]
  }
};
```

### 2. Herramientas Principales

#### GitHub Actions
```yaml
# .github/workflows/react-native.yml
name: React Native CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm install
      - name: Build Android
        run: cd android && ./gradlew assembleRelease
```

#### Fastlane
```ruby
# fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Build and deploy Android app"
  lane :deploy do
    # 1. Incrementar versión
    increment_version_code(
      gradle_file_path: "app/build.gradle"
    )
    
    # 2. Build de la aplicación
    gradle(
      task: "clean assembleRelease",
      project_dir: "android/"
    )
    
    # 3. Subir a Google Play
    upload_to_play_store(
      track: 'internal',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab'
    )
  end
end
```

#### Jenkins
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build') {
            steps {
                sh 'cd android && ./gradlew assembleRelease'
            }
        }
    }
}
```

### 3. Flujos de Trabajo en React Native

#### Flujo de Desarrollo
```javascript
// workflow/development-flow.js
class DevelopmentWorkflow {
  constructor() {
    this.stages = [
      'development',    // Desarrollo local
      'staging',        // Entorno de pruebas
      'production'      // Entorno de producción
    ];
  }
  
  // 1. Desarrollo Local
  async developLocally() {
    console.log('🚀 Iniciando desarrollo local...');
    
    // Metro bundler
    await this.startMetro();
    
    // Simulador/Emulador
    await this.startSimulator();
    
    // Hot reload activado
    this.enableHotReload();
  }
  
  // 2. Commit y Push
  async commitAndPush() {
    console.log('📝 Creando commit...');
    
    // Verificar código
    await this.runLinting();
    await this.runTests();
    
    // Crear commit
    await this.createCommit();
    
    // Push al repositorio
    await this.pushToRemote();
  }
  
  // 3. CI/CD Pipeline
  async triggerPipeline() {
    console.log('🔄 Activando pipeline CI/CD...');
    
    // GitHub Actions se activa automáticamente
    // o Jenkins detecta cambios
    this.monitorPipeline();
  }
}
```

#### Flujo de Build
```javascript
// workflow/build-flow.js
class BuildWorkflow {
  constructor() {
    this.platforms = ['android', 'ios'];
    this.buildTypes = ['debug', 'release'];
  }
  
  // Build para Android
  async buildAndroid(type = 'debug') {
    console.log(`🔨 Construyendo Android (${type})...`);
    
    try {
      // Limpiar build anterior
      await this.cleanAndroidBuild();
      
      // Instalar dependencias
      await this.installAndroidDependencies();
      
      // Generar bundle
      if (type === 'release') {
        await this.generateReleaseBundle();
      } else {
        await this.generateDebugAPK();
      }
      
      console.log('✅ Build de Android completado');
    } catch (error) {
      console.error('❌ Error en build de Android:', error);
      throw error;
    }
  }
  
  // Build para iOS
  async buildIOS(type = 'debug') {
    console.log(`🍎 Construyendo iOS (${type})...`);
    
    try {
      // Limpiar build anterior
      await this.cleanIOSBuild();
      
      // Instalar pods
      await this.installPods();
      
      // Generar build
      if (type === 'release') {
        await this.generateReleaseIPA();
      } else {
        await this.generateDebugApp();
      }
      
      console.log('✅ Build de iOS completado');
    } catch (error) {
      console.error('❌ Error en build de iOS:', error);
      throw error;
    }
  }
}
```

### 4. Configuración de Entornos

#### Variables de Entorno
```javascript
// config/environments.js
class EnvironmentConfig {
  constructor() {
    this.environments = {
      development: {
        apiUrl: 'http://localhost:3000',
        appName: 'MyApp Dev',
        version: '1.0.0-dev',
        enableLogs: true,
        enableAnalytics: false
      },
      
      staging: {
        apiUrl: 'https://staging-api.myapp.com',
        appName: 'MyApp Staging',
        version: '1.0.0-staging',
        enableLogs: true,
        enableAnalytics: true
      },
      
      production: {
        apiUrl: 'https://api.myapp.com',
        appName: 'MyApp',
        version: '1.0.0',
        enableLogs: false,
        enableAnalytics: true
      }
    };
  }
  
  // Obtener configuración del entorno actual
  getCurrentConfig() {
    const env = process.env.NODE_ENV || 'development';
    return this.environments[env];
  }
  
  // Configurar variables para build
  setupBuildConfig(platform, buildType) {
    const config = this.getCurrentConfig();
    
    if (platform === 'android') {
      this.setupAndroidConfig(config, buildType);
    } else if (platform === 'ios') {
      this.setupIOSConfig(config, buildType);
    }
  }
}
```

#### Configuración de Build
```javascript
// config/build-config.js
class BuildConfig {
  constructor() {
    this.androidConfig = {
      debug: {
        applicationId: 'com.myapp.debug',
        versionName: '1.0.0-debug',
        versionCode: 1,
        debuggable: true
      },
      
      release: {
        applicationId: 'com.myapp',
        versionName: '1.0.0',
        versionCode: 1,
        debuggable: false,
        minifyEnabled: true
      }
    };
    
    this.iosConfig = {
      debug: {
        bundleIdentifier: 'com.myapp.debug',
        version: '1.0.0-debug',
        build: 1,
        configuration: 'Debug'
      },
      
      release: {
        bundleIdentifier: 'com.myapp',
        version: '1.0.0',
        build: 1,
        configuration: 'Release'
      }
    };
  }
  
  // Aplicar configuración de build
  applyBuildConfig(platform, buildType) {
    console.log(`🔧 Aplicando configuración ${buildType} para ${platform}...`);
    
    if (platform === 'android') {
      this.updateAndroidBuildGradle(buildType);
    } else if (platform === 'ios') {
      this.updateIOSProjectConfig(buildType);
    }
  }
}
```

## Ejercicios Prácticos

### Ejercicio 1: Pipeline Básico de CI/CD
Crea un pipeline básico usando GitHub Actions que:
- Se active con cada push a la rama main
- Instale dependencias
- Ejecute tests
- Construya la aplicación para Android e iOS
- Genere artefactos de build

**Solución:**
```yaml
# .github/workflows/basic-pipeline.yml
name: Basic CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  test-and-build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '11'
          
      - name: Build Android
        run: |
          cd android
          ./gradlew assembleRelease
          
      - name: Upload Android build
        uses: actions/upload-artifact@v3
        with:
          name: android-release
          path: android/app/build/outputs/apk/release/
          
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          
      - name: Install Fastlane
        run: gem install fastlane
        
      - name: Build iOS
        run: |
          cd ios
          fastlane build
          
      - name: Upload iOS build
        uses: actions/upload-artifact@v3
        with:
          name: ios-release
          path: ios/build/
```

### Ejercicio 2: Configurador de Entornos
Implementa un sistema que permita cambiar entre entornos de desarrollo, staging y producción:

**Solución:**
```javascript
// scripts/switch-environment.js
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

class EnvironmentSwitcher {
  constructor() {
    this.configPath = path.join(__dirname, '../config/environments.js');
    this.envFiles = {
      android: 'android/app/build.gradle',
      ios: 'ios/MyApp/Info.plist'
    };
  }
  
  // Cambiar entorno
  switchEnvironment(env) {
    console.log(`🔄 Cambiando a entorno: ${env}`);
    
    if (!['development', 'staging', 'production'].includes(env)) {
      throw new Error(`Entorno no válido: ${env}`);
    }
    
    // Actualizar configuración
    this.updateEnvironmentConfig(env);
    
    // Actualizar archivos nativos
    this.updateNativeConfigs(env);
    
    console.log(`✅ Entorno cambiado a: ${env}`);
  }
  
  // Actualizar configuración de entorno
  updateEnvironmentConfig(env) {
    const configContent = fs.readFileSync(this.configPath, 'utf8');
    
    // Reemplazar NODE_ENV
    const updatedContent = configContent.replace(
      /NODE_ENV\s*=\s*['"][^'"]*['"]/,
      `NODE_ENV = '${env}'`
    );
    
    fs.writeFileSync(this.configPath, updatedContent);
  }
  
  // Actualizar configuraciones nativas
  updateNativeConfigs(env) {
    // Android
    this.updateAndroidConfig(env);
    
    // iOS
    this.updateIOSConfig(env);
  }
  
  // Actualizar configuración de Android
  updateAndroidConfig(env) {
    const buildGradlePath = this.envFiles.android;
    
    if (fs.existsSync(buildGradlePath)) {
      let content = fs.readFileSync(buildGradlePath, 'utf8');
      
      // Actualizar applicationId según entorno
      const appIdMap = {
        development: 'com.myapp.debug',
        staging: 'com.myapp.staging',
        production: 'com.myapp'
      };
      
      content = content.replace(
        /applicationId\s+["'][^"']*["']/,
        `applicationId "${appIdMap[env]}"`
      );
      
      fs.writeFileSync(buildGradlePath, content);
    }
  }
  
  // Actualizar configuración de iOS
  updateIOSConfig(env) {
    const infoPlistPath = this.envFiles.ios;
    
    if (fs.existsSync(infoPlistPath)) {
      let content = fs.readFileSync(infoPlistPath, 'utf8');
      
      // Actualizar bundle identifier según entorno
      const bundleIdMap = {
        development: 'com.myapp.debug',
        staging: 'com.myapp.staging',
        production: 'com.myapp'
      };
      
      content = content.replace(
        /<key>CFBundleIdentifier<\/key>\s*<string>[^<]*<\/string>/,
        `<key>CFBundleIdentifier</key>\n\t<string>${bundleIdMap[env]}</string>`
      );
      
      fs.writeFileSync(infoPlistPath, content);
    }
  }
}

// Uso desde línea de comandos
if (require.main === module) {
  const env = process.argv[2];
  
  if (!env) {
    console.error('❌ Uso: node switch-environment.js <environment>');
    console.error('Entornos disponibles: development, staging, production');
    process.exit(1);
  }
  
  const switcher = new EnvironmentSwitcher();
  switcher.switchEnvironment(env);
}

module.exports = EnvironmentSwitcher;
```

### Ejercicio 3: Monitor de Pipeline
Crea un sistema que monitoree el estado de los pipelines de CI/CD:

**Solución:**
```javascript
// monitoring/pipeline-monitor.js
class PipelineMonitor {
  constructor() {
    this.pipelines = new Map();
    this.status = {
      success: '✅',
      failure: '❌',
      running: '🔄',
      pending: '⏳'
    };
  }
  
  // Agregar pipeline para monitoreo
  addPipeline(name, config) {
    this.pipelines.set(name, {
      ...config,
      status: 'pending',
      lastRun: null,
      duration: 0,
      history: []
    });
    
    console.log(`📊 Pipeline "${name}" agregado al monitoreo`);
  }
  
  // Actualizar estado del pipeline
  updatePipelineStatus(name, status, details = {}) {
    const pipeline = this.pipelines.get(name);
    
    if (pipeline) {
      const previousStatus = pipeline.status;
      pipeline.status = status;
      pipeline.lastRun = new Date();
      
      if (details.duration) {
        pipeline.duration = details.duration;
      }
      
      // Agregar al historial
      pipeline.history.push({
        timestamp: new Date(),
        status,
        duration: pipeline.duration,
        details
      });
      
      // Mantener solo los últimos 10 registros
      if (pipeline.history.length > 10) {
        pipeline.history.shift();
      }
      
      console.log(`📊 Pipeline "${name}": ${previousStatus} → ${this.status[status]}`);
      
      // Notificar si hay cambio de estado
      if (previousStatus !== status) {
        this.notifyStatusChange(name, previousStatus, status);
      }
    }
  }
  
  // Notificar cambio de estado
  notifyStatusChange(name, previousStatus, newStatus) {
    const emoji = this.status[newStatus];
    const message = `Pipeline "${name}" cambió de ${previousStatus} a ${newStatus} ${emoji}`;
    
    // Aquí podrías integrar con Slack, email, etc.
    console.log(`🔔 ${message}`);
    
    // Ejemplo de notificación por email
    if (newStatus === 'failure') {
      this.sendFailureNotification(name, message);
    }
  }
  
  // Enviar notificación de fallo
  sendFailureNotification(pipelineName, message) {
    // Implementar notificación por email, Slack, etc.
    console.log(`📧 Enviando notificación de fallo para: ${pipelineName}`);
  }
  
  // Obtener resumen de pipelines
  getSummary() {
    const summary = {
      total: this.pipelines.size,
      success: 0,
      failure: 0,
      running: 0,
      pending: 0
    };
    
    for (const pipeline of this.pipelines.values()) {
      summary[pipeline.status]++;
    }
    
    return summary;
  }
  
  // Mostrar dashboard
  showDashboard() {
    console.log('\n📊 DASHBOARD DE PIPELINES');
    console.log('=' .repeat(50));
    
    const summary = this.getSummary();
    console.log(`Total: ${summary.total} | ✅ ${summary.success} | ❌ ${summary.failure} | 🔄 ${summary.running} | ⏳ ${summary.pending}`);
    console.log('');
    
    for (const [name, pipeline] of this.pipelines) {
      const status = this.status[pipeline.status];
      const lastRun = pipeline.lastRun ? pipeline.lastRun.toLocaleTimeString() : 'Nunca';
      const duration = pipeline.duration ? `(${pipeline.duration}s)` : '';
      
      console.log(`${status} ${name} - Última ejecución: ${lastRun} ${duration}`);
    }
    
    console.log('');
  }
  
  // Simular ejecución de pipeline
  simulatePipelineRun(name) {
    const pipeline = this.pipelines.get(name);
    
    if (pipeline) {
      console.log(`🔄 Simulando ejecución de pipeline: ${name}`);
      
      // Cambiar a running
      this.updatePipelineStatus(name, 'running');
      
      // Simular duración aleatoria
      const duration = Math.floor(Math.random() * 60) + 30; // 30-90 segundos
      
      setTimeout(() => {
        // Simular éxito o fallo
        const success = Math.random() > 0.3; // 70% éxito
        const finalStatus = success ? 'success' : 'failure';
        
        this.updatePipelineStatus(name, finalStatus, { duration });
      }, duration * 1000);
    }
  }
}

// Ejemplo de uso
const monitor = new PipelineMonitor();

// Agregar pipelines
monitor.addPipeline('Android Build', { platform: 'android' });
monitor.addPipeline('iOS Build', { platform: 'ios' });
monitor.addPipeline('Testing', { platform: 'all' });
monitor.addPipeline('Deployment', { platform: 'all' });

// Simular ejecuciones
setInterval(() => {
  const pipelines = Array.from(monitor.pipelines.keys());
  const randomPipeline = pipelines[Math.floor(Math.random() * pipelines.length)];
  monitor.simulatePipelineRun(randomPipeline);
}, 10000); // Cada 10 segundos

// Mostrar dashboard cada 5 segundos
setInterval(() => {
  monitor.showDashboard();
}, 5000);
```

## Resumen de la Clase

En esta clase hemos cubierto:

1. **Conceptos fundamentales** de CI/CD y su importancia en el desarrollo móvil
2. **Herramientas principales** como GitHub Actions, Fastlane y Jenkins
3. **Flujos de trabajo** específicos para React Native
4. **Configuración de entornos** para diferentes etapas de desarrollo
5. **Ejercicios prácticos** que implementan pipelines básicos y monitoreo

## Próximos Pasos

En la siguiente clase aprenderemos a configurar **GitHub Actions** específicamente para React Native, incluyendo workflows avanzados y optimizaciones.

## Navegación
- **Clase Anterior**: [Clase 5: Sensores y Hardware Avanzado](clase_5_sensores_hardware_avanzado.md)
- **Clase Siguiente**: [Clase 2: GitHub Actions para React Native](clase_2_github_actions.md)
- **Volver al Módulo**: [README del Módulo 12](README.md)
- **Volver al Índice**: [Índice Completo](../../INDICE_COMPLETO.md)
