# 🚀 Clase 5: Monitoreo y Métricas

## 🎯 Objetivos de la Clase
- Implementar monitoreo de builds y despliegues
- Configurar métricas de rendimiento de la aplicación
- Implementar alertas y notificaciones automáticas
- Analizar crash reports en producción

---

## 📚 Contenido Teórico

### 1. Monitoreo de Builds y Despliegues

#### **¿Por qué es importante el monitoreo?**
El monitoreo en CI/CD permite:
- **Detectar fallos** en builds y despliegues
- **Optimizar tiempos** de ejecución
- **Identificar cuellos de botella** en el pipeline
- **Mejorar la calidad** del código desplegado
- **Reducir tiempo de resolución** de problemas

#### **Métricas Clave de CI/CD**
- **Build Time**: Tiempo total de compilación
- **Success Rate**: Porcentaje de builds exitosos
- **Deployment Frequency**: Frecuencia de despliegues
- **Lead Time**: Tiempo desde commit hasta producción
- **Mean Time to Recovery (MTTR)**: Tiempo de recuperación

### 2. Monitoreo con GitHub Actions

#### **Workflow con Métricas**
```yaml
# .github/workflows/ci-cd-metrics.yml
name: CI/CD with Metrics

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
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
      
      - name: Build Android
        run: |
          cd android
          ./gradlew assembleDebug
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: app-debug
          path: android/app/build/outputs/apk/debug/
      
      - name: Record build metrics
        run: |
          echo "BUILD_TIME=$(date +%s)" >> $GITHUB_ENV
          echo "BUILD_STATUS=success" >> $GITHUB_ENV
      
      - name: Send notification
        if: always()
        run: |
          if [ "${{ job.status }}" == "success" ]; then
            echo "✅ Build exitoso"
          else
            echo "❌ Build falló"
          fi
```

#### **Workflow de Deploy con Métricas**
```yaml
# .github/workflows/deploy-metrics.yml
name: Deploy with Metrics

on:
  workflow_run:
    workflows: ["CI/CD with Metrics"]
    types: [completed]

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          bundler-cache: true
      
      - name: Deploy to Google Play
        run: bundle exec fastlane deploy
        env:
          SUPPLY_JSON_KEY: ${{ secrets.GOOGLE_PLAY_API_KEY }}
      
      - name: Record deployment metrics
        run: |
          echo "DEPLOY_TIME=$(date +%s)" >> $GITHUB_ENV
          echo "DEPLOY_STATUS=success" >> $GITHUB_ENV
      
      - name: Send deployment notification
        run: |
          echo "🚀 Deploy completado exitosamente"
```

### 3. Métricas de Rendimiento de la Aplicación

#### **Firebase Performance Monitoring**
```javascript
// config/performance.js
import perf from '@react-native-firebase/perf';

export const PerformanceMonitor = {
  // Monitorear tiempo de carga de pantalla
  startScreenLoad: (screenName) => {
    const trace = perf.newTrace('screen_load');
    trace.putAttribute('screen_name', screenName);
    trace.start();
    return trace;
  },
  
  // Monitorear operaciones de red
  startNetworkRequest: (url, method) => {
    const metric = perf.newHttpMetric(url, method);
    metric.start();
    return metric;
  },
  
  // Monitorear operaciones personalizadas
  startCustomTrace: (traceName) => {
    const trace = perf.newTrace(traceName);
    trace.start();
    return trace;
  }
};

// Uso en componentes
export const usePerformanceMonitoring = (screenName) => {
  useEffect(() => {
    const trace = PerformanceMonitor.startScreenLoad(screenName);
    
    return () => {
      trace.stop();
    };
  }, [screenName]);
};
```

#### **Integración con React Navigation**
```javascript
// navigation/PerformanceNavigation.js
import { NavigationContainer } from '@react-navigation/native';
import { PerformanceMonitor } from '../config/performance';

export const PerformanceNavigation = ({ children }) => {
  const onStateChange = (state) => {
    if (state?.routes?.length > 0) {
      const currentRoute = state.routes[state.index];
      PerformanceMonitor.startScreenLoad(currentRoute.name);
    }
  };

  return (
    <NavigationContainer onStateChange={onStateChange}>
      {children}
    </NavigationContainer>
  );
};
```

### 4. Alertas y Notificaciones Automáticas

#### **Configuración de Slack Notifications**
```yaml
# .github/workflows/slack-notifications.yml
name: Slack Notifications

on:
  workflow_run:
    workflows: ["CI/CD with Metrics", "Deploy with Metrics"]
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          text: |
            🚀 Workflow: ${{ github.event.workflow_run.name }}
            📊 Status: ${{ github.event.workflow_run.conclusion }}
            🔗 URL: ${{ github.event.workflow_run.html_url }}
            👤 Triggered by: ${{ github.event.workflow_run.actor.login }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### **Configuración de Email Notifications**
```yaml
# .github/workflows/email-notifications.yml
name: Email Notifications

on:
  workflow_run:
    workflows: ["Deploy with Metrics"]
    types: [completed]

jobs:
  notify-email:
    runs-on: ubuntu-latest
    steps:
      - name: Send email notification
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.gmail.com
          server_port: 587
          username: ${{ secrets.EMAIL_USERNAME }}
          password: ${{ secrets.EMAIL_PASSWORD }}
          subject: "Deploy Status: ${{ github.event.workflow_run.conclusion }}"
          to: ${{ secrets.NOTIFICATION_EMAIL }}
          from: "CI/CD Bot"
          body: |
            Workflow: ${{ github.event.workflow_run.name }}
            Status: ${{ github.event.workflow_run.conclusion }}
            URL: ${{ github.event.workflow_run.html_url }}
            Triggered by: ${{ github.event.workflow_run.actor.login }}
```

### 5. Dashboard de Métricas

#### **Configuración de Grafana**
```yaml
# docker-compose.grafana.yml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-storage:/prometheus

volumes:
  grafana-storage:
  prometheus-storage:
```

#### **Configuración de Prometheus**
```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'github-actions'
    static_configs:
      - targets: ['localhost:9090']
    
  - job_name: 'react-native-app'
    static_configs:
      - targets: ['localhost:8080']
    metrics_path: '/metrics'
```

### 6. Análisis de Crash Reports

#### **Integración con Firebase Crashlytics**
```javascript
// config/crashlytics.js
import crashlytics from '@react-native-firebase/crashlytics';

export const CrashlyticsService = {
  // Registrar error personalizado
  logError: (error, context = {}) => {
    crashlytics().recordError(error);
    
    // Agregar contexto adicional
    Object.entries(context).forEach(([key, value]) => {
      crashlytics().setAttribute(key, value);
    });
  },
  
  // Registrar información del usuario
  setUserInfo: (userId, userEmail, userName) => {
    crashlytics().setUserId(userId);
    crashlytics().setAttribute('user_email', userEmail);
    crashlytics().setAttribute('user_name', userName);
  },
  
  // Registrar evento personalizado
  logEvent: (eventName, parameters = {}) => {
    crashlytics().log(`${eventName}: ${JSON.stringify(parameters)}`);
  }
};

// Uso en error boundaries
export const useCrashlytics = () => {
  const logError = useCallback((error, context) => {
    CrashlyticsService.logError(error, context);
  }, []);
  
  return { logError };
};
```

#### **Error Boundary con Métricas**
```jsx
// components/ErrorBoundary.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { CrashlyticsService } from '../config/crashlytics';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Registrar en Crashlytics
    CrashlyticsService.logError(error, {
      component: this.constructor.name,
      errorInfo: JSON.stringify(errorInfo)
    });
    
    // Enviar métricas de error
    this.sendErrorMetrics(error);
  }

  sendErrorMetrics = (error) => {
    // Implementar envío de métricas
    console.log('Error metrics sent:', error.message);
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          <Text>Algo salió mal 😕</Text>
          <Button 
            title="Reintentar" 
            onPress={() => this.setState({ hasError: false })} 
          />
        </View>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

### 7. Métricas de Negocio

#### **Configuración de Analytics**
```javascript
// config/analytics.js
import analytics from '@react-native-firebase/analytics';

export const AnalyticsService = {
  // Registrar evento de pantalla
  logScreenView: (screenName, screenClass) => {
    analytics().logScreenView({
      screen_name: screenName,
      screen_class: screenClass
    });
  },
  
  // Registrar evento personalizado
  logEvent: (eventName, parameters = {}) => {
    analytics().logEvent(eventName, parameters);
  },
  
  // Registrar conversión
  logConversion: (conversionType, value = 0) => {
    analytics().logEvent('conversion', {
      conversion_type: conversionType,
      value: value
    });
  },
  
  // Configurar propiedades del usuario
  setUserProperty: (name, value) => {
    analytics().setUserProperty(name, value);
  }
};

// Hook para analytics
export const useAnalytics = () => {
  const logScreenView = useCallback((screenName, screenClass) => {
    AnalyticsService.logScreenView(screenName, screenClass);
  }, []);
  
  const logEvent = useCallback((eventName, parameters) => {
    AnalyticsService.logEvent(eventName, parameters);
  }, []);
  
  return { logScreenView, logEvent };
};
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Dashboard de Métricas Completo
Crea un dashboard que muestre métricas clave de CI/CD:

**Requisitos:**
- Métricas de builds (tiempo, éxito, fallos)
- Métricas de deploys (frecuencia, tiempo)
- Métricas de rendimiento de la app
- Alertas automáticas para problemas

**Implementación:**
```yaml
# .github/workflows/metrics-dashboard.yml
name: Metrics Dashboard

on:
  schedule:
    - cron: '0 */6 * * *'  # Cada 6 horas

jobs:
  collect-metrics:
    runs-on: ubuntu-latest
    steps:
      - name: Collect build metrics
        run: |
          # Obtener métricas de GitHub API
          echo "BUILD_SUCCESS_RATE=$(curl -s ...)"
          echo "AVERAGE_BUILD_TIME=$(curl -s ...)"
          echo "DEPLOYMENT_FREQUENCY=$(curl -s ...)"
      
      - name: Send to monitoring service
        run: |
          # Enviar métricas a servicio de monitoreo
          curl -X POST "https://monitoring.service.com/metrics" \
            -H "Content-Type: application/json" \
            -d '{
              "build_success_rate": "${{ env.BUILD_SUCCESS_RATE }}",
              "average_build_time": "${{ env.AVERAGE_BUILD_TIME }}",
              "deployment_frequency": "${{ env.DEPLOYMENT_FREQUENCY }}"
            }'
```

### Ejercicio 2: Sistema de Alertas Inteligente
Implementa un sistema de alertas basado en umbrales:

**Requisitos:**
- Alertas para builds fallidos consecutivos
- Alertas para tiempo de build excesivo
- Alertas para métricas de rendimiento
- Notificaciones en múltiples canales

**Implementación:**
```javascript
// services/alertService.js
export class AlertService {
  constructor() {
    this.thresholds = {
      buildFailureThreshold: 3,
      buildTimeThreshold: 300, // 5 minutos
      performanceThreshold: 0.8
    };
  }
  
  checkBuildFailures(failureCount) {
    if (failureCount >= this.thresholds.buildFailureThreshold) {
      this.sendAlert('BUILD_FAILURE', {
        message: `Builds fallidos consecutivos: ${failureCount}`,
        severity: 'HIGH'
      });
    }
  }
  
  checkBuildTime(buildTime) {
    if (buildTime > this.thresholds.buildTimeThreshold) {
      this.sendAlert('BUILD_TIME', {
        message: `Tiempo de build excesivo: ${buildTime}s`,
        severity: 'MEDIUM'
      });
    }
  }
  
  sendAlert(type, data) {
    // Enviar a Slack
    this.sendSlackAlert(type, data);
    
    // Enviar email
    this.sendEmailAlert(type, data);
    
    // Registrar en sistema de monitoreo
    this.logAlert(type, data);
  }
}
```

---

## 🔍 Puntos Clave

1. **Monitoreo continuo** es esencial para CI/CD efectivo
2. **Métricas de rendimiento** ayudan a optimizar la aplicación
3. **Alertas automáticas** permiten respuesta rápida a problemas
4. **Dashboards** proporcionan visibilidad del estado del sistema
5. **Crash reporting** es crucial para aplicaciones en producción

---

## 📖 Recursos Adicionales

- [GitHub Actions Monitoring](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows)
- [Firebase Performance](https://firebase.google.com/docs/perf)
- [Firebase Crashlytics](https://firebase.google.com/docs/crashlytics)
- [Grafana Dashboards](https://grafana.com/docs/grafana/latest/dashboards/)

---

## 🎉 ¡Módulo Completado!

Has completado exitosamente el **Módulo 12: CI/CD y Deployment** del curso de React Native. 

**Lo que has aprendido:**
- ✅ Fundamentos de CI/CD y su importancia para React Native
- ✅ Generación de APK/AAB con Gradle y configuración de builds
- ✅ Configuración de flavors y variantes de build
- ✅ Automatización de despliegue con Fastlane
- ✅ Monitoreo y métricas para CI/CD

**Próximo paso:** Continuar con el **Módulo 13: Monitoreo y Analytics** para aprender a implementar sistemas completos de monitoreo y analytics en aplicaciones React Native.
