# Clase 5: Despliegue y Monitoreo 🚀

## Objetivos de la Clase
- Configurar despliegue automático a App Store y Google Play
- Implementar monitoreo de crashes y métricas de rendimiento
- Configurar alertas y notificaciones automáticas
- Crear dashboards de monitoreo en tiempo real

## Duración Estimada
**1.5 horas**

## Contenido Teórico

### 1. Despliegue a App Store Connect

#### Configuración de App Store Connect
```javascript
// deployment/app-store-connect.js
class AppStoreConnectManager {
  constructor() {
    this.apiKey = process.env.APP_STORE_CONNECT_API_KEY;
    this.teamId = process.env.DEVELOPMENT_TEAM_ID;
    this.bundleId = 'com.myapp';
  }
  
  // Subir build a TestFlight
  async uploadToTestFlight(buildPath) {
    try {
      console.log('🍎 Subiendo build a TestFlight...');
      
      // Verificar build
      await this.validateBuild(buildPath);
      
      // Subir a App Store Connect
      const uploadResult = await this.uploadBuild(buildPath);
      
      // Procesar build
      await this.processBuild(uploadResult.id);
      
      console.log('✅ Build subido exitosamente a TestFlight');
      return uploadResult;
      
    } catch (error) {
      console.error('❌ Error al subir a TestFlight:', error);
      throw error;
    }
  }
  
  // Subir a App Store
  async submitToAppStore(buildId, metadata) {
    try {
      console.log('📱 Enviando build a App Store...');
      
      // Preparar metadatos
      const appMetadata = await this.prepareMetadata(metadata);
      
      // Enviar para revisión
      const submission = await this.submitForReview(buildId, appMetadata);
      
      console.log('✅ Build enviado para revisión de App Store');
      return submission;
      
    } catch (error) {
      console.error('❌ Error al enviar a App Store:', error);
      throw error;
    }
  }
}
```

#### Fastlane para iOS
```ruby
# fastlane/Fastfile - iOS Deployment
platform :ios do
  desc "Deploy to TestFlight"
  lane :deploy_testflight do
    # Incrementar build number
    increment_build_number(
      xcodeproj: "ios/MyApp.xcodeproj"
    )
    
    # Build de la aplicación
    build_ios_app(
      workspace: "ios/MyApp.xcworkspace",
      scheme: "MyApp",
      configuration: "Release",
      export_method: "app-store",
      output_directory: "ios/build",
      output_name: "MyApp.ipa"
    )
    
    # Subir a TestFlight
    pilot(
      api_key_path: ENV['APP_STORE_CONNECT_API_KEY'],
      ipa: "ios/build/MyApp.ipa",
      skip_waiting_for_build_processing: true,
      distribute_external: true,
      notify_external_testers: true
    )
  end
  
  desc "Deploy to App Store"
  lane :deploy_appstore do
    # Build release
    build_ios_app(
      workspace: "ios/MyApp.xcworkspace",
      scheme: "MyApp",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # Subir a App Store
    deliver(
      api_key_path: ENV['APP_STORE_CONNECT_API_KEY'],
      ipa: "ios/build/MyApp.ipa",
      skip_screenshots: true,
      skip_metadata: true,
      submit_for_review: true,
      automatic_release: true
    )
  end
end
```

### 2. Despliegue a Google Play Console

#### Configuración de Google Play
```javascript
// deployment/google-play-manager.js
class GooglePlayManager {
  constructor() {
    this.serviceAccount = JSON.parse(process.env.GOOGLE_PLAY_JSON_KEY);
    this.packageName = 'com.myapp';
    this.tracks = ['internal', 'alpha', 'beta', 'production'];
  }
  
  // Subir APK/AAB a Google Play
  async uploadToPlayStore(buildPath, track = 'internal') {
    try {
      console.log(`🤖 Subiendo build a Google Play (${track})...`);
      
      // Verificar build
      await this.validateBuild(buildPath);
      
      // Crear nueva versión
      const version = await this.createVersion();
      
      // Subir archivo
      await this.uploadBundle(buildPath, version.id);
      
      // Publicar en track
      await this.publishToTrack(version.id, track);
      
      console.log(`✅ Build publicado exitosamente en ${track}`);
      return version;
      
    } catch (error) {
      console.error('❌ Error al subir a Google Play:', error);
      throw error;
    }
  }
  
  // Publicar en múltiples tracks
  async publishToMultipleTracks(buildPath, tracks) {
    const results = [];
    
    for (const track of tracks) {
      try {
        const result = await this.uploadToPlayStore(buildPath, track);
        results.push({ track, success: true, result });
      } catch (error) {
        results.push({ track, success: false, error: error.message });
      }
    }
    
    return results;
  }
}
```

#### Fastlane para Android
```ruby
# fastlane/Fastfile - Android Deployment
platform :android do
  desc "Deploy to Google Play Internal"
  lane :deploy_internal do
    # Incrementar versión
    increment_version_code(
      gradle_file_path: "android/app/build.gradle"
    )
    
    # Build de la aplicación
    gradle(
      task: "bundleRelease",
      project_dir: "android/"
    )
    
    # Subir a Google Play Internal
    upload_to_play_store(
      track: 'internal',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab',
      json_key: ENV['GOOGLE_PLAY_JSON_KEY'],
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end
  
  desc "Deploy to Google Play Production"
  lane :deploy_production do
    # Build release
    gradle(
      task: "bundleRelease",
      project_dir: "android/"
    )
    
    # Subir a Google Play Production
    upload_to_play_store(
      track: 'production',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab',
      json_key: ENV['GOOGLE_PLAY_JSON_KEY'],
      skip_upload_metadata: false,
      skip_upload_images: false,
      skip_upload_screenshots: false
    )
  end
end
```

### 3. Monitoreo de Crashes

#### Integración con Crashlytics
```javascript
// monitoring/crashlytics-manager.js
import crashlytics from '@react-native-firebase/crashlytics';

class CrashlyticsManager {
  constructor() {
    this.isEnabled = __DEV__ ? false : true;
    this.setupCrashlytics();
  }
  
  // Configurar Crashlytics
  setupCrashlytics() {
    if (this.isEnabled) {
      // Habilitar recolección de crashes
      crashlytics().setCrashlyticsCollectionEnabled(true);
      
      // Configurar usuario
      this.setUserIdentifier('user123');
      this.setUserEmail('user@example.com');
      
      // Configurar atributos personalizados
      this.setCustomAttributes({
        app_version: '1.0.0',
        build_number: '1',
        device_model: 'iPhone 14'
      });
    }
  }
  
  // Registrar crash manual
  logError(error, context = {}) {
    if (this.isEnabled) {
      crashlytics().recordError(error, context);
    }
    
    // También loggear en consola para desarrollo
    if (__DEV__) {
      console.error('Error logged:', error, context);
    }
  }
  
  // Registrar evento personalizado
  logEvent(eventName, parameters = {}) {
    if (this.isEnabled) {
      crashlytics().log(`${eventName}: ${JSON.stringify(parameters)}`);
    }
  }
  
  // Configurar identificador de usuario
  setUserIdentifier(userId) {
    if (this.isEnabled) {
      crashlytics().setUserId(userId);
    }
  }
  
  // Configurar email del usuario
  setUserEmail(email) {
    if (this.isEnabled) {
      crashlytics().setAttribute('email', email);
    }
  }
  
  // Configurar atributos personalizados
  setCustomAttributes(attributes) {
    if (this.isEnabled) {
      Object.entries(attributes).forEach(([key, value]) => {
        crashlytics().setAttribute(key, value);
      });
    }
  }
}

export default CrashlyticsManager;
```

#### Monitoreo de Performance
```javascript
// monitoring/performance-monitor.js
import perf from '@react-native-firebase/perf';

class PerformanceMonitor {
  constructor() {
    this.isEnabled = __DEV__ ? false : true;
    this.traces = new Map();
    this.setupPerformance();
  }
  
  // Configurar Performance Monitoring
  setupPerformance() {
    if (this.isEnabled) {
      perf().setPerformanceCollectionEnabled(true);
    }
  }
  
  // Iniciar trace personalizado
  startTrace(traceName) {
    if (this.isEnabled) {
      const trace = perf().newTrace(traceName);
      trace.start();
      this.traces.set(traceName, trace);
      return trace;
    }
    return null;
  }
  
  // Detener trace
  stopTrace(traceName) {
    if (this.isEnabled && this.traces.has(traceName)) {
      const trace = this.traces.get(traceName);
      trace.stop();
      this.traces.delete(traceName);
    }
  }
  
  // Medir tiempo de renderizado de pantalla
  async measureScreenRender(screenName) {
    if (this.isEnabled) {
      const trace = this.startTrace(`screen_render_${screenName}`);
      
      // Simular medición de renderizado
      await new Promise(resolve => setTimeout(resolve, 100));
      
      this.stopTrace(`screen_render_${screenName}`);
      return trace;
    }
    return null;
  }
  
  // Medir tiempo de API calls
  async measureAPICall(apiName, apiCall) {
    if (this.isEnabled) {
      const trace = this.startTrace(`api_call_${apiName}`);
      
      try {
        const result = await apiCall();
        trace.putAttribute('status', 'success');
        return result;
      } catch (error) {
        trace.putAttribute('status', 'error');
        trace.putAttribute('error_message', error.message);
        throw error;
      } finally {
        this.stopTrace(`api_call_${apiName}`);
      }
    } else {
      return await apiCall();
    }
  }
}

export default PerformanceMonitor;
```

### 4. Sistema de Alertas y Notificaciones

#### Configuración de Alertas
```javascript
// monitoring/alert-manager.js
class AlertManager {
  constructor() {
    this.alertRules = new Map();
    this.notificationChannels = new Map();
    this.setupAlertRules();
  }
  
  // Configurar reglas de alerta
  setupAlertRules() {
    // Alerta por crash rate alto
    this.alertRules.set('high_crash_rate', {
      threshold: 5, // 5% de crash rate
      timeWindow: 3600000, // 1 hora
      action: 'notify_team'
    });
    
    // Alerta por performance degradado
    this.alertRules.set('performance_degradation', {
      threshold: 2000, // 2 segundos
      timeWindow: 1800000, // 30 minutos
      action: 'notify_developers'
    });
    
    // Alerta por errores de API
    this.alertRules.set('api_errors', {
      threshold: 10, // 10 errores
      timeWindow: 900000, // 15 minutos
      action: 'notify_devops'
    });
  }
  
  // Verificar reglas de alerta
  checkAlertRules(metrics) {
    const triggeredAlerts = [];
    
    for (const [ruleName, rule] of this.alertRules) {
      if (this.shouldTriggerAlert(ruleName, rule, metrics)) {
        triggeredAlerts.push({
          rule: ruleName,
          threshold: rule.threshold,
          currentValue: metrics[ruleName],
          action: rule.action
        });
      }
    }
    
    return triggeredAlerts;
  }
  
  // Determinar si debe activarse una alerta
  shouldTriggerAlert(ruleName, rule, metrics) {
    const currentValue = metrics[ruleName];
    
    switch (ruleName) {
      case 'high_crash_rate':
        return currentValue > rule.threshold;
      case 'performance_degradation':
        return currentValue > rule.threshold;
      case 'api_errors':
        return currentValue > rule.threshold;
      default:
        return false;
    }
  }
  
  // Ejecutar acciones de alerta
  async executeAlertActions(alerts) {
    for (const alert of alerts) {
      switch (alert.action) {
        case 'notify_team':
          await this.notifyTeam(alert);
          break;
        case 'notify_developers':
          await this.notifyDevelopers(alert);
          break;
        case 'notify_devops':
          await this.notifyDevOps(alert);
          break;
      }
    }
  }
  
  // Notificar al equipo
  async notifyTeam(alert) {
    console.log(`🚨 Team Alert: ${alert.rule} threshold exceeded!`);
    // Implementar notificación por Slack, email, etc.
  }
  
  // Notificar a desarrolladores
  async notifyDevelopers(alert) {
    console.log(`⚠️ Developer Alert: ${alert.rule} needs attention!`);
    // Implementar notificación específica para desarrolladores
  }
  
  // Notificar a DevOps
  async notifyDevOps(alert) {
    console.log(`🔧 DevOps Alert: ${alert.rule} requires immediate action!`);
    // Implementar notificación urgente para DevOps
  }
}

export default AlertManager;
```

### 5. Dashboard de Monitoreo

#### Dashboard en Tiempo Real
```javascript
// monitoring/dashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, ScrollView } from 'react-native';
import { LineChart, BarChart } from 'react-native-chart-kit';

class MonitoringDashboard {
  constructor() {
    this.metrics = {
      crashRate: 0,
      performance: 0,
      apiErrors: 0,
      activeUsers: 0
    };
    
    this.historicalData = {
      crashRate: [],
      performance: [],
      apiErrors: [],
      activeUsers: []
    };
    
    this.startMonitoring();
  }
  
  // Iniciar monitoreo
  startMonitoring() {
    // Actualizar métricas cada 30 segundos
    setInterval(() => {
      this.updateMetrics();
    }, 30000);
    
    // Actualizar datos históricos cada 5 minutos
    setInterval(() => {
      this.updateHistoricalData();
    }, 300000);
  }
  
  // Actualizar métricas en tiempo real
  async updateMetrics() {
    try {
      // Obtener métricas de Crashlytics
      const crashlyticsMetrics = await this.getCrashlyticsMetrics();
      
      // Obtener métricas de Performance
      const performanceMetrics = await this.getPerformanceMetrics();
      
      // Obtener métricas de API
      const apiMetrics = await this.getAPIMetrics();
      
      // Actualizar métricas locales
      this.metrics = {
        crashRate: crashlyticsMetrics.crashRate,
        performance: performanceMetrics.avgResponseTime,
        apiErrors: apiMetrics.errorRate,
        activeUsers: crashlyticsMetrics.activeUsers
      };
      
      // Verificar alertas
      const alertManager = new AlertManager();
      const alerts = alertManager.checkAlertRules(this.metrics);
      
      if (alerts.length > 0) {
        await alertManager.executeAlertActions(alerts);
      }
      
    } catch (error) {
      console.error('Error updating metrics:', error);
    }
  }
  
  // Obtener métricas de Crashlytics
  async getCrashlyticsMetrics() {
    // Simular llamada a API de Crashlytics
    return {
      crashRate: Math.random() * 10,
      activeUsers: Math.floor(Math.random() * 10000)
    };
  }
  
  // Obtener métricas de Performance
  async getPerformanceMetrics() {
    // Simular llamada a API de Performance
    return {
      avgResponseTime: Math.random() * 3000
    };
  }
  
  // Obtener métricas de API
  async getAPIMetrics() {
    // Simular llamada a API de monitoreo
    return {
      errorRate: Math.random() * 20
    };
  }
  
  // Actualizar datos históricos
  updateHistoricalData() {
    const timestamp = Date.now();
    
    // Agregar métricas actuales a datos históricos
    Object.keys(this.metrics).forEach(key => {
      this.historicalData[key].push({
        timestamp,
        value: this.metrics[key]
      });
      
      // Mantener solo las últimas 100 mediciones
      if (this.historicalData[key].length > 100) {
        this.historicalData[key].shift();
      }
    });
  }
  
  // Obtener datos para gráficos
  getChartData(metricName) {
    const data = this.historicalData[metricName];
    
    return {
      labels: data.map(d => new Date(d.timestamp).toLocaleTimeString()),
      datasets: [{
        data: data.map(d => d.value)
      }]
    };
  }
  
  // Generar reporte de estado
  generateStatusReport() {
    const status = {
      overall: 'healthy',
      details: {}
    };
    
    // Evaluar crash rate
    if (this.metrics.crashRate > 5) {
      status.overall = 'critical';
      status.details.crashRate = 'High crash rate detected';
    } else if (this.metrics.crashRate > 2) {
      status.overall = 'warning';
      status.details.crashRate = 'Elevated crash rate';
    }
    
    // Evaluar performance
    if (this.metrics.performance > 2000) {
      status.overall = status.overall === 'healthy' ? 'warning' : status.overall;
      status.details.performance = 'Performance degradation detected';
    }
    
    // Evaluar errores de API
    if (this.metrics.apiErrors > 10) {
      status.overall = status.overall === 'healthy' ? 'warning' : status.overall;
      status.details.apiErrors = 'High API error rate';
    }
    
    return status;
  }
}

export default MonitoringDashboard;
```

## Ejercicios Prácticos

### Ejercicio 1: Pipeline de Despliegue Completo
Implementa un pipeline que despliegue automáticamente a ambas plataformas:

**Solución:**
```yaml
# .github/workflows/auto-deploy.yml
name: Auto Deploy

on:
  push:
    tags: ['v*']
  workflow_dispatch:
    inputs:
      platform:
        description: 'Platform to deploy'
        required: true
        default: 'all'
        type: choice
        options:
        - android
        - ios
        - all

jobs:
  deploy-android:
    name: Deploy Android
    runs-on: ubuntu-latest
    if: github.event.inputs.platform == 'android' || github.event.inputs.platform == 'all'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '11'
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          gem install fastlane
          
      - name: Deploy to Google Play
        run: |
          cd android
          fastlane deploy_production
        env:
          GOOGLE_PLAY_JSON_KEY: ${{ secrets.GOOGLE_PLAY_JSON_KEY }}

  deploy-ios:
    name: Deploy iOS
    runs-on: macos-latest
    if: github.event.inputs.platform == 'ios' || github.event.inputs.platform == 'all'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install && cd ..
          gem install fastlane
          
      - name: Deploy to App Store
        run: |
          cd ios
          fastlane deploy_appstore
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          DEVELOPMENT_TEAM_ID: ${{ secrets.DEVELOPMENT_TEAM_ID }}

  notify-deployment:
    name: Notify Deployment
    runs-on: ubuntu-latest
    needs: [deploy-android, deploy-ios]
    if: always()
    
    steps:
      - name: Check deployment status
        run: |
          if [ "${{ needs.deploy-android.result }}" == "success" ] && [ "${{ needs.deploy-ios.result }}" == "success" ]; then
            echo "status=success" >> $GITHUB_OUTPUT
            echo "message=All platforms deployed successfully!" >> $GITHUB_OUTPUT
          else
            echo "status=failure" >> $GITHUB_OUTPUT
            echo "message=Some deployments failed!" >> $GITHUB_OUTPUT
          fi
          
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ steps.check-deployment-status.outputs.status }}
          text: |
            ${{ steps.check-deployment-status.outputs.message }}
            
            **Version**: ${{ github.ref_name }}
            **Android**: ${{ needs.deploy-android.result }}
            **iOS**: ${{ needs.deploy-ios.result }}
            
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Ejercicio 2: Sistema de Monitoreo Inteligente
Implementa un sistema que monitoree y alerte automáticamente:

**Solución:**
```javascript
// monitoring/intelligent-monitor.js
import CrashlyticsManager from './crashlytics-manager';
import PerformanceMonitor from './performance-monitor';
import AlertManager from './alert-manager';

class IntelligentMonitor {
  constructor() {
    this.crashlytics = new CrashlyticsManager();
    this.performance = new PerformanceMonitor();
    this.alerts = new AlertManager();
    
    this.monitoringInterval = null;
    this.alertThresholds = {
      crashRate: 5,
      responseTime: 2000,
      apiErrors: 10,
      memoryUsage: 80
    };
    
    this.startIntelligentMonitoring();
  }
  
  // Iniciar monitoreo inteligente
  startIntelligentMonitoring() {
    // Monitorear cada 30 segundos
    this.monitoringInterval = setInterval(() => {
      this.performHealthCheck();
    }, 30000);
    
    // Monitoreo de memoria cada 5 minutos
    setInterval(() => {
      this.checkMemoryUsage();
    }, 300000);
    
    // Monitoreo de red cada minuto
    setInterval(() => {
      this.checkNetworkHealth();
    }, 60000);
  }
  
  // Realizar verificación de salud
  async performHealthCheck() {
    try {
      const healthMetrics = await this.gatherHealthMetrics();
      const healthScore = this.calculateHealthScore(healthMetrics);
      
      // Registrar métricas
      this.logHealthMetrics(healthMetrics, healthScore);
      
      // Verificar si se necesitan alertas
      if (healthScore < 70) {
        await this.triggerHealthAlert(healthScore, healthMetrics);
      }
      
      // Ajustar umbrales dinámicamente
      this.adjustThresholds(healthMetrics);
      
    } catch (error) {
      console.error('Health check failed:', error);
      this.crashlytics.logError(error, { context: 'health_check' });
    }
  }
  
  // Recolectar métricas de salud
  async gatherHealthMetrics() {
    const metrics = {
      timestamp: Date.now(),
      crashRate: await this.getCrashRate(),
      responseTime: await this.getAverageResponseTime(),
      apiErrors: await this.getAPIErrorRate(),
      memoryUsage: await this.getMemoryUsage(),
      networkLatency: await this.getNetworkLatency(),
      batteryLevel: await this.getBatteryLevel()
    };
    
    return metrics;
  }
  
  // Calcular puntuación de salud
  calculateHealthScore(metrics) {
    let score = 100;
    
    // Reducir puntuación por métricas problemáticas
    if (metrics.crashRate > this.alertThresholds.crashRate) {
      score -= 20;
    }
    
    if (metrics.responseTime > this.alertThresholds.responseTime) {
      score -= 15;
    }
    
    if (metrics.apiErrors > this.alertThresholds.apiErrors) {
      score -= 15;
    }
    
    if (metrics.memoryUsage > this.alertThresholds.memoryUsage) {
      score -= 10;
    }
    
    return Math.max(0, score);
  }
  
  // Registrar métricas de salud
  logHealthMetrics(metrics, healthScore) {
    console.log(`🏥 Health Check - Score: ${healthScore}/100`);
    console.log('Metrics:', metrics);
    
    // Enviar a Crashlytics
    this.crashlytics.logEvent('health_check', {
      score: healthScore,
      metrics: metrics
    });
  }
  
  // Activar alerta de salud
  async triggerHealthAlert(healthScore, metrics) {
    const alertLevel = healthScore < 50 ? 'critical' : 'warning';
    
    console.log(`🚨 ${alertLevel.toUpperCase()} Health Alert! Score: ${healthScore}`);
    
    // Enviar alerta
    await this.alerts.executeAlertActions([{
      rule: 'health_check',
      threshold: 70,
      currentValue: healthScore,
      action: `notify_${alertLevel}`,
      metrics: metrics
    }]);
  }
  
  // Ajustar umbrales dinámicamente
  adjustThresholds(metrics) {
    // Ajustar umbral de crash rate basado en tendencias
    if (metrics.crashRate < 1) {
      this.alertThresholds.crashRate = Math.max(1, this.alertThresholds.crashRate - 0.5);
    } else if (metrics.crashRate > 8) {
      this.alertThresholds.crashRate = Math.min(10, this.alertThresholds.crashRate + 0.5);
    }
    
    // Ajustar umbral de response time
    if (metrics.responseTime < 1000) {
      this.alertThresholds.responseTime = Math.max(1000, this.alertThresholds.responseTime - 100);
    } else if (metrics.responseTime > 3000) {
      this.alertThresholds.responseTime = Math.min(5000, this.alertThresholds.responseTime + 100);
    }
  }
  
  // Verificar uso de memoria
  async checkMemoryUsage() {
    const memoryUsage = await this.getMemoryUsage();
    
    if (memoryUsage > 90) {
      console.warn('⚠️ High memory usage detected:', memoryUsage + '%');
      
      // Intentar liberar memoria
      this.attemptMemoryCleanup();
    }
  }
  
  // Verificar salud de red
  async checkNetworkHealth() {
    const latency = await this.getNetworkLatency();
    
    if (latency > 5000) {
      console.warn('🌐 High network latency detected:', latency + 'ms');
      
      // Notificar degradación de red
      this.crashlytics.logEvent('network_degradation', { latency });
    }
  }
  
  // Intentar limpieza de memoria
  attemptMemoryCleanup() {
    // Limpiar cachés
    if (global.ImageCache) {
      global.ImageCache.clearCache();
    }
    
    // Forzar garbage collection si está disponible
    if (global.gc) {
      global.gc();
    }
    
    console.log('🧹 Memory cleanup attempted');
  }
  
  // Métodos auxiliares para obtener métricas
  async getCrashRate() {
    // Simular obtención de crash rate
    return Math.random() * 10;
  }
  
  async getAverageResponseTime() {
    // Simular obtención de tiempo de respuesta promedio
    return Math.random() * 3000;
  }
  
  async getAPIErrorRate() {
    // Simular obtención de tasa de errores de API
    return Math.random() * 20;
  }
  
  async getMemoryUsage() {
    // Simular obtención de uso de memoria
    return Math.random() * 100;
  }
  
  async getNetworkLatency() {
    // Simular obtención de latencia de red
    return Math.random() * 10000;
  }
  
  async getBatteryLevel() {
    // Simular obtención de nivel de batería
    return Math.random() * 100;
  }
  
  // Detener monitoreo
  stopMonitoring() {
    if (this.monitoringInterval) {
      clearInterval(this.monitoringInterval);
      this.monitoringInterval = null;
    }
  }
}

export default IntelligentMonitor;
```

### Ejercicio 3: Dashboard de Métricas en Tiempo Real
Crea un dashboard que muestre métricas en tiempo real:

**Solución:**
```javascript
// monitoring/real-time-dashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, ScrollView, Dimensions } from 'react-native';
import { LineChart, BarChart, PieChart } from 'react-native-chart-kit';

const RealTimeDashboard = () => {
  const [metrics, setMetrics] = useState({
    crashRate: 0,
    performance: 0,
    apiErrors: 0,
    activeUsers: 0,
    memoryUsage: 0,
    batteryLevel: 0
  });
  
  const [historicalData, setHistoricalData] = useState({
    crashRate: [],
    performance: [],
    apiErrors: []
  });
  
  const [status, setStatus] = useState('healthy');
  
  useEffect(() => {
    // Iniciar monitoreo en tiempo real
    const monitoringInterval = setInterval(updateMetrics, 5000);
    
    return () => clearInterval(monitoringInterval);
  }, []);
  
  // Actualizar métricas
  const updateMetrics = async () => {
    try {
      const newMetrics = await fetchMetrics();
      setMetrics(newMetrics);
      
      // Actualizar datos históricos
      updateHistoricalData(newMetrics);
      
      // Actualizar estado general
      updateStatus(newMetrics);
      
    } catch (error) {
      console.error('Error updating metrics:', error);
    }
  };
  
  // Obtener métricas del servidor
  const fetchMetrics = async () => {
    // Simular llamada a API
    return {
      crashRate: Math.random() * 10,
      performance: Math.random() * 3000,
      apiErrors: Math.random() * 20,
      activeUsers: Math.floor(Math.random() * 10000),
      memoryUsage: Math.random() * 100,
      batteryLevel: Math.random() * 100
    };
  };
  
  // Actualizar datos históricos
  const updateHistoricalData = (newMetrics) => {
    const timestamp = Date.now();
    
    setHistoricalData(prev => ({
      crashRate: [...prev.crashRate.slice(-19), { timestamp, value: newMetrics.crashRate }],
      performance: [...prev.performance.slice(-19), { timestamp, value: newMetrics.performance }],
      apiErrors: [...prev.apiErrors.slice(-19), { timestamp, value: newMetrics.apiErrors }]
    }));
  };
  
  // Actualizar estado general
  const updateStatus = (newMetrics) => {
    let newStatus = 'healthy';
    
    if (newMetrics.crashRate > 5 || newMetrics.performance > 2000 || newMetrics.apiErrors > 10) {
      newStatus = 'warning';
    }
    
    if (newMetrics.crashRate > 8 || newMetrics.performance > 3000 || newMetrics.apiErrors > 20) {
      newStatus = 'critical';
    }
    
    setStatus(newStatus);
  };
  
  // Configuración de gráficos
  const chartConfig = {
    backgroundColor: '#ffffff',
    backgroundGradientFrom: '#ffffff',
    backgroundGradientTo: '#ffffff',
    decimalPlaces: 2,
    color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
    style: {
      borderRadius: 16
    }
  };
  
  // Preparar datos para gráficos
  const prepareChartData = (data, label) => {
    return {
      labels: data.map(d => new Date(d.timestamp).toLocaleTimeString()),
      datasets: [{
        data: data.map(d => d.value),
        color: (opacity = 1) => `rgba(0, 122, 255, ${opacity})`,
        strokeWidth: 2
      }]
    };
  };
  
  return (
    <ScrollView style={styles.container}>
      {/* Header con estado general */}
      <View style={[styles.statusHeader, styles[status]]}>
        <Text style={styles.statusText}>
          Status: {status.toUpperCase()}
        </Text>
        <Text style={styles.timestamp}>
          Last updated: {new Date().toLocaleTimeString()}
        </Text>
      </View>
      
      {/* Métricas principales */}
      <View style={styles.metricsGrid}>
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>Crash Rate</Text>
          <Text style={styles.metricValue}>{metrics.crashRate.toFixed(2)}%</Text>
        </View>
        
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>Response Time</Text>
          <Text style={styles.metricValue}>{metrics.performance.toFixed(0)}ms</Text>
        </View>
        
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>API Errors</Text>
          <Text style={styles.metricValue}>{metrics.apiErrors.toFixed(0)}</Text>
        </View>
        
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>Active Users</Text>
          <Text style={styles.metricValue}>{metrics.activeUsers.toLocaleString()}</Text>
        </View>
      </View>
      
      {/* Gráfico de Crash Rate */}
      <View style={styles.chartContainer}>
        <Text style={styles.chartTitle}>Crash Rate Trend</Text>
        <LineChart
          data={prepareChartData(historicalData.crashRate, 'Crash Rate')}
          width={Dimensions.get('window').width - 32}
          height={220}
          chartConfig={chartConfig}
          bezier
          style={styles.chart}
        />
      </View>
      
      {/* Gráfico de Performance */}
      <View style={styles.chartContainer}>
        <Text style={styles.chartTitle}>Response Time Trend</Text>
        <LineChart
          data={prepareChartData(historicalData.performance, 'Response Time')}
          width={Dimensions.get('window').width - 32}
          height={220}
          chartConfig={chartConfig}
          bezier
          style={styles.chart}
        />
      </View>
      
      {/* Gráfico de API Errors */}
      <View style={styles.chartContainer}>
        <Text style={styles.chartTitle}>API Error Rate</Text>
        <BarChart
          data={prepareChartData(historicalData.apiErrors, 'API Errors')}
          width={Dimensions.get('window').width - 32}
          height={220}
          chartConfig={chartConfig}
          style={styles.chart}
        />
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  statusHeader: {
    padding: 16,
    margin: 16,
    borderRadius: 8,
    alignItems: 'center'
  },
  healthy: {
    backgroundColor: '#d4edda',
    borderColor: '#c3e6cb'
  },
  warning: {
    backgroundColor: '#fff3cd',
    borderColor: '#ffeaa7'
  },
  critical: {
    backgroundColor: '#f8d7da',
    borderColor: '#f5c6cb'
  },
  statusText: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 4
  },
  timestamp: {
    fontSize: 12,
    color: '#666'
  },
  metricsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 16
  },
  metricCard: {
    width: '48%',
    backgroundColor: 'white',
    padding: 16,
    margin: '1%',
    borderRadius: 8,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  metricLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 8
  },
  metricValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333'
  },
  chartContainer: {
    backgroundColor: 'white',
    margin: 16,
    padding: 16,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  chartTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 16,
    textAlign: 'center'
  },
  chart: {
    marginVertical: 8,
    borderRadius: 16
  }
});

export default RealTimeDashboard;
```

## Resumen de la Clase

En esta clase hemos cubierto:

1. **Despliegue automático** a App Store Connect y Google Play Console
2. **Monitoreo de crashes** con Crashlytics y métricas de rendimiento
3. **Sistema de alertas** inteligente y notificaciones automáticas
4. **Dashboard de monitoreo** en tiempo real con métricas visuales
5. **Integración completa** de CI/CD con monitoreo y alertas

## Próximos Pasos

Con esto hemos completado el **Módulo 12: CI/CD**. En el siguiente módulo aprenderemos sobre **Testing Avanzado** para React Native.

## Navegación
- **Clase Anterior**: [Clase 4: Testing Automatizado en CI/CD](clase_4_testing_automatizado.md)
- **Clase Siguiente**: [Módulo 13: Testing Avanzado](../senior_4/)
- **Volver al Módulo**: [README del Módulo 12](README.md)
- **Volver al Índice**: [Índice Completo](../../INDICE_COMPLETO.md)
