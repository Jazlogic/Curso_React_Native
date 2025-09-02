# 📊 Clase 4: Dashboards de BI en Tiempo Real

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a crear dashboards de Business Intelligence en tiempo real para aplicaciones React Native, incluyendo visualizaciones interactivas, streaming de datos, KPI tracking y alertas automáticas. Aprenderás a construir sistemas de BI que proporcionen insights accionables en tiempo real.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Crear** dashboards de BI interactivos en React Native
2. **Implementar** streaming de datos en tiempo real
3. **Configurar** KPI tracking y alertas automáticas
4. **Desarrollar** visualizaciones de datos avanzadas
5. **Optimizar** performance de dashboards en tiempo real

### **⏱️ Duración Estimada**
- **Total**: 2-2.5 horas
- **Teoría**: 45 minutos
- **Práctica**: 75-90 minutos

---

## 📚 Contenido de la Clase

### **1. Business Intelligence para Móviles**

#### **¿Qué es Business Intelligence?**

Business Intelligence (BI) es el proceso de transformar datos en insights accionables que impulsen decisiones de negocio estratégicas. En el contexto móvil, BI se enfoca en métricas de usuario, engagement y conversión.

```javascript
// Estructura de BI para móviles
const MobileBI = {
  dataSources: {
    userBehavior: 'Comportamiento de usuarios',
    appPerformance: 'Rendimiento de la app',
    businessMetrics: 'Métricas de negocio',
    externalData: 'Datos externos'
  },
  processing: {
    realTime: 'Procesamiento en tiempo real',
    batch: 'Procesamiento por lotes',
    streaming: 'Streaming de datos',
    aggregation: 'Agregación de métricas'
  },
  visualization: {
    dashboards: 'Dashboards ejecutivos',
    charts: 'Gráficos interactivos',
    alerts: 'Alertas automáticas',
    reports: 'Reportes automáticos'
  }
};
```

#### **KPIs Clave para Móviles**

```javascript
// KPIs esenciales para aplicaciones móviles
const MobileKPIs = {
  userMetrics: {
    dau: 'Daily Active Users',
    mau: 'Monthly Active Users',
    retention: 'User Retention Rate',
    churn: 'User Churn Rate',
    ltv: 'Lifetime Value'
  },
  engagementMetrics: {
    sessionDuration: 'Average Session Duration',
    screenViews: 'Screen Views per Session',
    actions: 'Actions per Session',
    frequency: 'Usage Frequency'
  },
  businessMetrics: {
    conversion: 'Conversion Rate',
    revenue: 'Revenue per User',
    arpu: 'Average Revenue per User',
    cac: 'Customer Acquisition Cost'
  },
  technicalMetrics: {
    crashRate: 'Crash Rate',
    loadTime: 'App Load Time',
    apiResponse: 'API Response Time',
    errorRate: 'Error Rate'
  }
};
```

### **2. Dashboards Interactivos**

#### **Sistema de Dashboards**

```javascript
// Sistema de dashboards interactivos
class DashboardSystem {
  constructor() {
    this.dashboards = new Map();
    this.widgets = new Map();
    this.dataStreams = new Map();
    this.alerts = new Map();
  }

  // Crear dashboard
  createDashboard(dashboardId, name, description, layout) {
    this.dashboards.set(dashboardId, {
      id: dashboardId,
      name: name,
      description: description,
      layout: layout,
      widgets: [],
      filters: {},
      refreshInterval: 30000, // 30 segundos
      isRealTime: true,
      created_at: Date.now()
    });
  }

  // Agregar widget al dashboard
  addWidget(dashboardId, widgetId, widgetType, config) {
    const dashboard = this.dashboards.get(dashboardId);
    if (!dashboard) return;

    const widget = {
      id: widgetId,
      type: widgetType,
      config: config,
      data: null,
      lastUpdated: null,
      isRealTime: config.realTime || false
    };

    dashboard.widgets.push(widget);
    this.widgets.set(widgetId, widget);
  }

  // Actualizar datos del widget
  updateWidgetData(widgetId, data) {
    const widget = this.widgets.get(widgetId);
    if (!widget) return;

    widget.data = data;
    widget.lastUpdated = Date.now();

    // Notificar a suscriptores
    this.notifyWidgetSubscribers(widgetId, data);
  }

  // Obtener datos del dashboard
  getDashboardData(dashboardId) {
    const dashboard = this.dashboards.get(dashboardId);
    if (!dashboard) return null;

    const widgetsData = dashboard.widgets.map(widget => ({
      id: widget.id,
      type: widget.type,
      config: widget.config,
      data: widget.data,
      lastUpdated: widget.lastUpdated
    }));

    return {
      dashboard: dashboard,
      widgets: widgetsData,
      lastUpdated: Date.now()
    };
  }

  // Configurar filtros del dashboard
  setDashboardFilters(dashboardId, filters) {
    const dashboard = this.dashboards.get(dashboardId);
    if (!dashboard) return;

    dashboard.filters = filters;
    this.applyFiltersToWidgets(dashboardId, filters);
  }

  // Aplicar filtros a widgets
  applyFiltersToWidgets(dashboardId, filters) {
    const dashboard = this.dashboards.get(dashboardId);
    if (!dashboard) return;

    dashboard.widgets.forEach(widget => {
      this.updateWidgetWithFilters(widget.id, filters);
    });
  }

  // Actualizar widget con filtros
  updateWidgetWithFilters(widgetId, filters) {
    const widget = this.widgets.get(widgetId);
    if (!widget) return;

    // Aplicar filtros a los datos del widget
    const filteredData = this.applyFiltersToData(widget.data, filters);
    widget.data = filteredData;
    widget.lastUpdated = Date.now();
  }

  // Aplicar filtros a datos
  applyFiltersToData(data, filters) {
    if (!data || !filters) return data;

    let filteredData = { ...data };

    // Aplicar filtro de fecha
    if (filters.dateRange) {
      filteredData = this.filterByDateRange(filteredData, filters.dateRange);
    }

    // Aplicar filtro de segmento
    if (filters.segment) {
      filteredData = this.filterBySegment(filteredData, filters.segment);
    }

    // Aplicar filtro de plataforma
    if (filters.platform) {
      filteredData = this.filterByPlatform(filteredData, filters.platform);
    }

    return filteredData;
  }

  // Notificar suscriptores del widget
  notifyWidgetSubscribers(widgetId, data) {
    // Implementar sistema de notificaciones
    // En una implementación real, esto usaría WebSockets o Server-Sent Events
    console.log(`Widget ${widgetId} updated:`, data);
  }
}

export default new DashboardSystem();
```

#### **Componentes de Widgets**

```javascript
// Componentes de widgets para React Native
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';
import { LineChart, BarChart, PieChart } from 'react-native-chart-kit';

// Widget de métrica KPI
const KPIMetricWidget = ({ widgetId, config, data }) => {
  const [currentValue, setCurrentValue] = useState(0);
  const [previousValue, setPreviousValue] = useState(0);
  const [trend, setTrend] = useState('stable');

  useEffect(() => {
    if (data) {
      setCurrentValue(data.current);
      setPreviousValue(data.previous);
      setTrend(data.trend);
    }
  }, [data]);

  const calculateChange = () => {
    if (previousValue === 0) return 0;
    return ((currentValue - previousValue) / previousValue) * 100;
  };

  const getTrendColor = () => {
    switch (trend) {
      case 'up': return '#4CAF50';
      case 'down': return '#F44336';
      default: return '#757575';
    }
  };

  return (
    <View style={styles.kpiWidget}>
      <Text style={styles.kpiTitle}>{config.title}</Text>
      <Text style={styles.kpiValue}>{currentValue.toLocaleString()}</Text>
      <Text style={[styles.kpiChange, { color: getTrendColor() }]}>
        {calculateChange() > 0 ? '+' : ''}{calculateChange().toFixed(1)}%
      </Text>
    </View>
  );
};

// Widget de gráfico de líneas
const LineChartWidget = ({ widgetId, config, data }) => {
  const [chartData, setChartData] = useState(null);

  useEffect(() => {
    if (data && data.datasets) {
      setChartData({
        labels: data.labels,
        datasets: data.datasets.map(dataset => ({
          data: dataset.data,
          color: (opacity = 1) => dataset.color || `rgba(134, 65, 244, ${opacity})`,
          strokeWidth: 2
        }))
      });
    }
  }, [data]);

  if (!chartData) return <View style={styles.loading}><Text>Loading...</Text></View>;

  return (
    <View style={styles.chartWidget}>
      <Text style={styles.chartTitle}>{config.title}</Text>
      <LineChart
        data={chartData}
        width={Dimensions.get('window').width - 40}
        height={220}
        chartConfig={{
          backgroundColor: '#ffffff',
          backgroundGradientFrom: '#ffffff',
          backgroundGradientTo: '#ffffff',
          decimalPlaces: 0,
          color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
          labelColor: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
          style: {
            borderRadius: 16
          },
          propsForDots: {
            r: '6',
            strokeWidth: '2',
            stroke: '#ffa726'
          }
        }}
        bezier
        style={styles.chart}
      />
    </View>
  );
};

// Widget de gráfico de barras
const BarChartWidget = ({ widgetId, config, data }) => {
  const [chartData, setChartData] = useState(null);

  useEffect(() => {
    if (data && data.labels && data.data) {
      setChartData({
        labels: data.labels,
        datasets: [{
          data: data.data
        }]
      });
    }
  }, [data]);

  if (!chartData) return <View style={styles.loading}><Text>Loading...</Text></View>;

  return (
    <View style={styles.chartWidget}>
      <Text style={styles.chartTitle}>{config.title}</Text>
      <BarChart
        data={chartData}
        width={Dimensions.get('window').width - 40}
        height={220}
        chartConfig={{
          backgroundColor: '#ffffff',
          backgroundGradientFrom: '#ffffff',
          backgroundGradientTo: '#ffffff',
          decimalPlaces: 0,
          color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
          labelColor: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
          style: {
            borderRadius: 16
          }
        }}
        style={styles.chart}
      />
    </View>
  );
};

// Widget de gráfico circular
const PieChartWidget = ({ widgetId, config, data }) => {
  const [chartData, setChartData] = useState(null);

  useEffect(() => {
    if (data && data.data) {
      setChartData(data.data.map((item, index) => ({
        name: item.name,
        population: item.value,
        color: item.color || `hsl(${index * 60}, 70%, 50%)`,
        legendFontColor: '#7F7F7F',
        legendFontSize: 12
      })));
    }
  }, [data]);

  if (!chartData) return <View style={styles.loading}><Text>Loading...</Text></View>;

  return (
    <View style={styles.chartWidget}>
      <Text style={styles.chartTitle}>{config.title}</Text>
      <PieChart
        data={chartData}
        width={Dimensions.get('window').width - 40}
        height={220}
        chartConfig={{
          color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`
        }}
        accessor="population"
        backgroundColor="transparent"
        paddingLeft="15"
        style={styles.chart}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  kpiWidget: {
    backgroundColor: '#ffffff',
    padding: 16,
    borderRadius: 8,
    margin: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  kpiTitle: {
    fontSize: 14,
    color: '#666',
    marginBottom: 8
  },
  kpiValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 4
  },
  kpiChange: {
    fontSize: 12,
    fontWeight: '500'
  },
  chartWidget: {
    backgroundColor: '#ffffff',
    padding: 16,
    borderRadius: 8,
    margin: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  chartTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 16
  },
  chart: {
    marginVertical: 8,
    borderRadius: 16
  },
  loading: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    height: 200
  }
});

export { KPIMetricWidget, LineChartWidget, BarChartWidget, PieChartWidget };
```

### **3. Real-time Analytics y Streaming**

#### **Sistema de Streaming de Datos**

```javascript
// Sistema de streaming de datos en tiempo real
class RealTimeDataStream {
  constructor() {
    this.connections = new Map();
    this.subscribers = new Map();
    this.dataCache = new Map();
    this.websocket = null;
  }

  // Conectar a WebSocket
  connect(websocketUrl) {
    this.websocket = new WebSocket(websocketUrl);
    
    this.websocket.onopen = () => {
      console.log('WebSocket connected');
      this.authenticate();
    };

    this.websocket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleIncomingData(data);
    };

    this.websocket.onclose = () => {
      console.log('WebSocket disconnected');
      this.reconnect();
    };

    this.websocket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  // Autenticar conexión
  authenticate() {
    const authMessage = {
      type: 'auth',
      token: this.getAuthToken()
    };
    this.websocket.send(JSON.stringify(authMessage));
  }

  // Manejar datos entrantes
  handleIncomingData(data) {
    switch (data.type) {
      case 'metric_update':
        this.updateMetric(data.metric, data.value);
        break;
      case 'kpi_update':
        this.updateKPI(data.kpi, data.value);
        break;
      case 'alert':
        this.handleAlert(data.alert);
        break;
      case 'dashboard_update':
        this.updateDashboard(data.dashboardId, data.data);
        break;
    }
  }

  // Actualizar métrica
  updateMetric(metricId, value) {
    this.dataCache.set(metricId, {
      value: value,
      timestamp: Date.now()
    });

    // Notificar suscriptores
    const subscribers = this.subscribers.get(metricId) || [];
    subscribers.forEach(callback => callback(value));
  }

  // Actualizar KPI
  updateKPI(kpiId, value) {
    const kpiData = {
      current: value.current,
      previous: value.previous,
      trend: value.trend,
      timestamp: Date.now()
    };

    this.dataCache.set(kpiId, kpiData);

    // Notificar suscriptores
    const subscribers = this.subscribers.get(kpiId) || [];
    subscribers.forEach(callback => callback(kpiData));
  }

  // Actualizar dashboard
  updateDashboard(dashboardId, data) {
    this.dataCache.set(`dashboard_${dashboardId}`, {
      data: data,
      timestamp: Date.now()
    });

    // Notificar suscriptores del dashboard
    const subscribers = this.subscribers.get(`dashboard_${dashboardId}`) || [];
    subscribers.forEach(callback => callback(data));
  }

  // Suscribirse a métrica
  subscribeToMetric(metricId, callback) {
    if (!this.subscribers.has(metricId)) {
      this.subscribers.set(metricId, []);
    }
    this.subscribers.get(metricId).push(callback);

    // Enviar datos actuales si están disponibles
    const currentData = this.dataCache.get(metricId);
    if (currentData) {
      callback(currentData);
    }

    // Solicitar suscripción al servidor
    this.sendSubscriptionRequest(metricId);
  }

  // Desuscribirse de métrica
  unsubscribeFromMetric(metricId, callback) {
    const subscribers = this.subscribers.get(metricId);
    if (subscribers) {
      const index = subscribers.indexOf(callback);
      if (index > -1) {
        subscribers.splice(index, 1);
      }
    }
  }

  // Enviar solicitud de suscripción
  sendSubscriptionRequest(metricId) {
    const message = {
      type: 'subscribe',
      metric: metricId
    };
    this.websocket.send(JSON.stringify(message));
  }

  // Manejar alerta
  handleAlert(alert) {
    console.log('Alert received:', alert);
    
    // Notificar a suscriptores de alertas
    const alertSubscribers = this.subscribers.get('alerts') || [];
    alertSubscribers.forEach(callback => callback(alert));
  }

  // Reconectar
  reconnect() {
    setTimeout(() => {
      console.log('Attempting to reconnect...');
      this.connect(this.websocket.url);
    }, 5000);
  }

  // Obtener token de autenticación
  getAuthToken() {
    // Implementar lógica de autenticación
    return 'your-auth-token';
  }
}

export default new RealTimeDataStream();
```

#### **Sistema de KPI Tracking**

```javascript
// Sistema de tracking de KPIs
class KPITrackingSystem {
  constructor() {
    this.kpis = new Map();
    this.metrics = new Map();
    this.alerts = new Map();
    this.dataStream = new RealTimeDataStream();
  }

  // Definir KPI
  defineKPI(kpiId, name, description, calculation, thresholds) {
    this.kpis.set(kpiId, {
      id: kpiId,
      name: name,
      description: description,
      calculation: calculation,
      thresholds: thresholds,
      currentValue: 0,
      previousValue: 0,
      trend: 'stable',
      lastUpdated: null
    });
  }

  // Actualizar KPI
  updateKPI(kpiId, value) {
    const kpi = this.kpis.get(kpiId);
    if (!kpi) return;

    kpi.previousValue = kpi.currentValue;
    kpi.currentValue = value;
    kpi.trend = this.calculateTrend(kpi.previousValue, kpi.currentValue);
    kpi.lastUpdated = Date.now();

    // Verificar umbrales
    this.checkThresholds(kpiId, value);

    // Notificar actualización
    this.dataStream.updateKPI(kpiId, {
      current: kpi.currentValue,
      previous: kpi.previousValue,
      trend: kpi.trend
    });
  }

  // Calcular tendencia
  calculateTrend(previous, current) {
    if (previous === 0) return 'stable';
    
    const change = ((current - previous) / previous) * 100;
    
    if (change > 5) return 'up';
    if (change < -5) return 'down';
    return 'stable';
  }

  // Verificar umbrales
  checkThresholds(kpiId, value) {
    const kpi = this.kpis.get(kpiId);
    if (!kpi || !kpi.thresholds) return;

    const thresholds = kpi.thresholds;
    
    // Verificar umbral crítico
    if (thresholds.critical && value <= thresholds.critical) {
      this.triggerAlert(kpiId, 'critical', value, thresholds.critical);
    }
    
    // Verificar umbral de advertencia
    if (thresholds.warning && value <= thresholds.warning) {
      this.triggerAlert(kpiId, 'warning', value, thresholds.warning);
    }
    
    // Verificar umbral de éxito
    if (thresholds.success && value >= thresholds.success) {
      this.triggerAlert(kpiId, 'success', value, thresholds.success);
    }
  }

  // Disparar alerta
  triggerAlert(kpiId, level, currentValue, threshold) {
    const kpi = this.kpis.get(kpiId);
    if (!kpi) return;

    const alert = {
      id: Date.now(),
      kpiId: kpiId,
      kpiName: kpi.name,
      level: level,
      currentValue: currentValue,
      threshold: threshold,
      message: this.generateAlertMessage(kpi.name, level, currentValue, threshold),
      timestamp: Date.now()
    };

    this.alerts.set(alert.id, alert);
    
    // Enviar alerta a través del stream
    this.dataStream.handleAlert(alert);
  }

  // Generar mensaje de alerta
  generateAlertMessage(kpiName, level, currentValue, threshold) {
    switch (level) {
      case 'critical':
        return `CRITICAL: ${kpiName} is ${currentValue}, below critical threshold of ${threshold}`;
      case 'warning':
        return `WARNING: ${kpiName} is ${currentValue}, below warning threshold of ${threshold}`;
      case 'success':
        return `SUCCESS: ${kpiName} is ${currentValue}, above success threshold of ${threshold}`;
      default:
        return `${kpiName} alert: ${currentValue}`;
    }
  }

  // Obtener KPI
  getKPI(kpiId) {
    return this.kpis.get(kpiId);
  }

  // Obtener todos los KPIs
  getAllKPIs() {
    return Array.from(this.kpis.values());
  }

  // Obtener alertas
  getAlerts(limit = 10) {
    const alerts = Array.from(this.alerts.values())
      .sort((a, b) => b.timestamp - a.timestamp)
      .slice(0, limit);
    
    return alerts;
  }

  // Suscribirse a KPI
  subscribeToKPI(kpiId, callback) {
    this.dataStream.subscribeToMetric(kpiId, callback);
  }

  // Suscribirse a alertas
  subscribeToAlerts(callback) {
    this.dataStream.subscribeToMetric('alerts', callback);
  }
}

export default new KPITrackingSystem();
```

### **4. Data Visualization**

#### **Sistema de Visualización**

```javascript
// Sistema de visualización de datos
class DataVisualizationSystem {
  constructor() {
    this.charts = new Map();
    this.themes = new Map();
    this.animations = new Map();
  }

  // Crear gráfico
  createChart(chartId, type, data, config) {
    const chart = {
      id: chartId,
      type: type,
      data: data,
      config: config,
      theme: config.theme || 'default',
      animation: config.animation || 'fade'
    };

    this.charts.set(chartId, chart);
    return chart;
  }

  // Actualizar datos del gráfico
  updateChartData(chartId, newData) {
    const chart = this.charts.get(chartId);
    if (!chart) return;

    chart.data = newData;
    chart.lastUpdated = Date.now();

    // Aplicar animación si está configurada
    if (chart.animation) {
      this.animateChart(chartId, chart.animation);
    }
  }

  // Aplicar tema
  applyTheme(chartId, themeName) {
    const chart = this.charts.get(chartId);
    if (!chart) return;

    const theme = this.themes.get(themeName);
    if (theme) {
      chart.config = { ...chart.config, ...theme };
      chart.theme = themeName;
    }
  }

  // Animar gráfico
  animateChart(chartId, animationType) {
    const chart = this.charts.get(chartId);
    if (!chart) return;

    const animation = this.animations.get(animationType);
    if (animation) {
      animation.apply(chart);
    }
  }

  // Obtener configuración de gráfico
  getChartConfig(chartId) {
    const chart = this.charts.get(chartId);
    return chart ? chart.config : null;
  }

  // Exportar gráfico
  exportChart(chartId, format = 'png') {
    const chart = this.charts.get(chartId);
    if (!chart) return null;

    // Implementar lógica de exportación
    return {
      chartId: chartId,
      format: format,
      data: chart.data,
      config: chart.config
    };
  }
}

// Temas predefinidos
const defaultThemes = {
  light: {
    backgroundColor: '#ffffff',
    textColor: '#333333',
    gridColor: '#e0e0e0',
    primaryColor: '#2196F3',
    secondaryColor: '#FF9800',
    successColor: '#4CAF50',
    warningColor: '#FF9800',
    errorColor: '#F44336'
  },
  dark: {
    backgroundColor: '#121212',
    textColor: '#ffffff',
    gridColor: '#333333',
    primaryColor: '#90CAF9',
    secondaryColor: '#FFB74D',
    successColor: '#81C784',
    warningColor: '#FFB74D',
    errorColor: '#E57373'
  },
  corporate: {
    backgroundColor: '#ffffff',
    textColor: '#2c3e50',
    gridColor: '#ecf0f1',
    primaryColor: '#3498db',
    secondaryColor: '#e74c3c',
    successColor: '#27ae60',
    warningColor: '#f39c12',
    errorColor: '#e74c3c'
  }
};

// Animaciones predefinidas
const defaultAnimations = {
  fade: {
    apply: (chart) => {
      // Implementar animación de desvanecimiento
      console.log('Applying fade animation to chart:', chart.id);
    }
  },
  slide: {
    apply: (chart) => {
      // Implementar animación de deslizamiento
      console.log('Applying slide animation to chart:', chart.id);
    }
  },
  scale: {
    apply: (chart) => {
      // Implementar animación de escala
      console.log('Applying scale animation to chart:', chart.id);
    }
  }
};

export default new DataVisualizationSystem();
```

### **5. Performance Optimization**

#### **Optimización de Dashboards**

```javascript
// Sistema de optimización de dashboards
class DashboardOptimization {
  constructor() {
    this.performanceMetrics = new Map();
    this.optimizationRules = new Map();
    this.cache = new Map();
  }

  // Optimizar dashboard
  optimizeDashboard(dashboardId) {
    const dashboard = this.dashboards.get(dashboardId);
    if (!dashboard) return;

    const optimizations = [];

    // Optimizar widgets
    dashboard.widgets.forEach(widget => {
      const widgetOptimizations = this.optimizeWidget(widget);
      optimizations.push(...widgetOptimizations);
    });

    // Optimizar datos
    const dataOptimizations = this.optimizeData(dashboardId);
    optimizations.push(...dataOptimizations);

    // Optimizar renderizado
    const renderOptimizations = this.optimizeRendering(dashboardId);
    optimizations.push(...renderOptimizations);

    return optimizations;
  }

  // Optimizar widget
  optimizeWidget(widget) {
    const optimizations = [];

    // Optimizar datos del widget
    if (widget.data && widget.data.length > 1000) {
      optimizations.push({
        type: 'data_sampling',
        widget: widget.id,
        description: 'Reduce data points for better performance',
        impact: 'high'
      });
    }

    // Optimizar actualizaciones
    if (widget.isRealTime && widget.refreshInterval < 1000) {
      optimizations.push({
        type: 'refresh_interval',
        widget: widget.id,
        description: 'Increase refresh interval to reduce CPU usage',
        impact: 'medium'
      });
    }

    return optimizations;
  }

  // Optimizar datos
  optimizeData(dashboardId) {
    const optimizations = [];

    // Implementar caché de datos
    optimizations.push({
      type: 'data_caching',
      dashboard: dashboardId,
      description: 'Implement data caching to reduce API calls',
      impact: 'high'
    });

    // Implementar paginación
    optimizations.push({
      type: 'data_pagination',
      dashboard: dashboardId,
      description: 'Implement data pagination for large datasets',
      impact: 'medium'
    });

    return optimizations;
  }

  // Optimizar renderizado
  optimizeRendering(dashboardId) {
    const optimizations = [];

    // Implementar lazy loading
    optimizations.push({
      type: 'lazy_loading',
      dashboard: dashboardId,
      description: 'Implement lazy loading for widgets',
      impact: 'high'
    });

    // Implementar virtualización
    optimizations.push({
      type: 'virtualization',
      dashboard: dashboardId,
      description: 'Implement virtualization for large lists',
      impact: 'medium'
    });

    return optimizations;
  }

  // Monitorear performance
  monitorPerformance(dashboardId) {
    const startTime = Date.now();
    
    return {
      dashboardId: dashboardId,
      loadTime: 0,
      renderTime: 0,
      dataFetchTime: 0,
      memoryUsage: 0,
      timestamp: startTime
    };
  }

  // Obtener métricas de performance
  getPerformanceMetrics(dashboardId) {
    return this.performanceMetrics.get(dashboardId) || {
      loadTime: 0,
      renderTime: 0,
      dataFetchTime: 0,
      memoryUsage: 0,
      lastUpdated: null
    };
  }
}

export default new DashboardOptimization();
```

---

## 🛠️ Implementación Práctica

### **Ejemplo: Dashboard Completo de BI**

```javascript
// Dashboard completo de Business Intelligence
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, ScrollView, RefreshControl } from 'react-native';
import { KPIMetricWidget, LineChartWidget, BarChartWidget, PieChartWidget } from './Widgets';

const BIDashboard = ({ dashboardId }) => {
  const [dashboardData, setDashboardData] = useState(null);
  const [isRefreshing, setIsRefreshing] = useState(false);
  const [filters, setFilters] = useState({});

  useEffect(() => {
    loadDashboardData();
    setupRealTimeUpdates();
  }, [dashboardId]);

  const loadDashboardData = async () => {
    try {
      const data = await dashboardSystem.getDashboardData(dashboardId);
      setDashboardData(data);
    } catch (error) {
      console.error('Error loading dashboard data:', error);
    }
  };

  const setupRealTimeUpdates = () => {
    // Suscribirse a actualizaciones en tiempo real
    dataStream.subscribeToMetric(`dashboard_${dashboardId}`, (data) => {
      setDashboardData(prevData => ({
        ...prevData,
        widgets: prevData.widgets.map(widget => ({
          ...widget,
          data: data[widget.id] || widget.data,
          lastUpdated: Date.now()
        }))
      }));
    });
  };

  const onRefresh = async () => {
    setIsRefreshing(true);
    await loadDashboardData();
    setIsRefreshing(false);
  };

  const renderWidget = (widget) => {
    switch (widget.type) {
      case 'kpi_metric':
        return <KPIMetricWidget key={widget.id} {...widget} />;
      case 'line_chart':
        return <LineChartWidget key={widget.id} {...widget} />;
      case 'bar_chart':
        return <BarChartWidget key={widget.id} {...widget} />;
      case 'pie_chart':
        return <PieChartWidget key={widget.id} {...widget} />;
      default:
        return <View key={widget.id}><Text>Unknown widget type</Text></View>;
    }
  };

  if (!dashboardData) {
    return (
      <View style={styles.loading}>
        <Text>Loading dashboard...</Text>
      </View>
    );
  }

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={isRefreshing} onRefresh={onRefresh} />
      }
    >
      <View style={styles.header}>
        <Text style={styles.title}>{dashboardData.dashboard.name}</Text>
        <Text style={styles.description}>{dashboardData.dashboard.description}</Text>
      </View>

      <View style={styles.widgetsContainer}>
        {dashboardData.widgets.map(renderWidget)}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  loading: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  header: {
    padding: 16,
    backgroundColor: '#ffffff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333'
  },
  description: {
    fontSize: 14,
    color: '#666',
    marginTop: 4
  },
  widgetsContainer: {
    padding: 8
  }
});

export default BIDashboard;
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear Dashboard Básico**

```javascript
// Crear dashboard básico con KPIs
const exercise1 = {
  task: 'Crear dashboard básico con métricas KPI',
  steps: [
    '1. Definir estructura del dashboard',
    '2. Crear widgets de métricas KPI',
    '3. Implementar actualizaciones en tiempo real',
    '4. Configurar filtros y refrescado'
  ],
  expectedResult: 'Dashboard funcional con KPIs en tiempo real'
};
```

### **Ejercicio 2: Visualizaciones Avanzadas**

```javascript
// Implementar visualizaciones avanzadas
const exercise2 = {
  task: 'Crear visualizaciones avanzadas de datos',
  steps: [
    '1. Implementar gráficos interactivos',
    '2. Configurar temas y animaciones',
    '3. Optimizar performance de renderizado',
    '4. Implementar exportación de datos'
  ],
  expectedResult: 'Sistema de visualización avanzado'
};
```

### **Ejercicio 3: Sistema de Alertas**

```javascript
// Implementar sistema de alertas
const exercise3 = {
  task: 'Crear sistema de alertas automáticas',
  steps: [
    '1. Configurar umbrales de KPIs',
    '2. Implementar notificaciones en tiempo real',
    '3. Crear dashboard de alertas',
    '4. Configurar escalación de alertas'
  ],
  expectedResult: 'Sistema de alertas funcional'
};
```

---

## 📊 Métricas y KPIs

### **Métricas de Dashboard**

```javascript
const DashboardMetrics = {
  performance: {
    load_time: 'Tiempo de carga del dashboard',
    render_time: 'Tiempo de renderizado',
    data_fetch_time: 'Tiempo de obtención de datos',
    memory_usage: 'Uso de memoria'
  },
  usage: {
    views: 'Número de vistas',
    unique_users: 'Usuarios únicos',
    session_duration: 'Duración de sesión',
    bounce_rate: 'Tasa de rebote'
  }
};
```

### **KPIs de BI**

```javascript
const BIKPIs = {
  user_engagement: {
    dau: 'Daily Active Users',
    session_duration: 'Average Session Duration',
    screen_views: 'Screen Views per Session',
    retention_rate: 'User Retention Rate'
  },
  business_metrics: {
    conversion_rate: 'Conversion Rate',
    revenue: 'Total Revenue',
    arpu: 'Average Revenue per User',
    ltv: 'Lifetime Value'
  }
};
```

---

## 🔧 Herramientas y Recursos

### **Herramientas de BI**

- **Tableau**: [tableau.com](https://www.tableau.com/)
- **Power BI**: [powerbi.microsoft.com](https://powerbi.microsoft.com/)
- **Looker**: [looker.com](https://looker.com/)
- **Metabase**: [metabase.com](https://www.metabase.com/)

### **Herramientas de Visualización**

- **D3.js**: [d3js.org](https://d3js.org/)
- **Chart.js**: [chartjs.org](https://www.chartjs.org/)
- **React Native Chart Kit**: [github.com/indiespirit/react-native-chart-kit](https://github.com/indiespirit/react-native-chart-kit)
- **Victory**: [formidable.com/open-source/victory](https://formidable.com/open-source/victory/)

### **Recursos de Aprendizaje**

- **BI Best Practices**: [tableau.com/learn/articles/business-intelligence](https://www.tableau.com/learn/articles/business-intelligence)
- **Data Visualization Guide**: [chartjs.org/docs](https://www.chartjs.org/docs/)
- **Real-time Analytics**: [kafka.apache.org](https://kafka.apache.org/)

---

## 🚀 Próximos Pasos

### **Lo que sigue**

1. **Clase 5**: Machine Learning para Analytics

### **Preparación**

- Configurar herramientas de BI
- Revisar conceptos de visualización de datos
- Preparar datos para dashboards
- Configurar streaming de datos

---

**🎯 Objetivo de la Clase**: Dominar la creación de dashboards de BI en tiempo real para aplicaciones React Native que proporcionen insights accionables.

**💡 Consejo**: Los dashboards efectivos combinan datos en tiempo real con visualizaciones claras y alertas proactivas. Enfócate en la usabilidad y el performance.

---

**🚀 ¡Comienza tu viaje hacia la maestría en Business Intelligence!**
