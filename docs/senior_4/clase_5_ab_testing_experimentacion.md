# Clase 5: A/B Testing y Experimentación 🧪

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Configurar Firebase Remote Config** para A/B testing
2. **Implementar experimentos controlados** en aplicaciones React Native
3. **Analizar resultados** de experimentos y tomar decisiones basadas en datos
4. **Crear sistemas de feature flags** para despliegues graduales
5. **Implementar testing multivariado** para optimización continua

## 🎯 Conceptos Clave

### ¿Qué es A/B Testing?

**A/B Testing** es una metodología de experimentación que compara dos o más versiones de una funcionalidad para determinar cuál produce mejores resultados según métricas específicas.

```javascript
// Ejemplo básico de Firebase Remote Config
import remoteConfig from '@react-native-firebase/remote-config';

class ABTestingService {
  constructor() {
    this.isEnabled = !__DEV__; // Solo en producción
    this.experiments = new Map();
    this.userAssignments = new Map();
    this.setupRemoteConfig();
  }

  // Configurar Remote Config
  async setupRemoteConfig() {
    try {
      // Configurar valores por defecto
      await remoteConfig().setDefaults({
        'welcome_message': 'Bienvenido a nuestra app!',
        'button_color': '#007AFF',
        'show_feature_x': false,
        'experiment_group': 'control'
      });

      // Configurar parámetros
      await remoteConfig().setConfigSettings({
        minimumFetchIntervalMillis: 3600000, // 1 hora
        fetchTimeoutMillis: 60000 // 1 minuto
      });

      // Obtener configuración remota
      await remoteConfig().fetchAndActivate();
      
      console.log('Remote Config configurado correctamente');
    } catch (error) {
      console.error('Error configurando Remote Config:', error);
    }
  }

  // Obtener valor de configuración
  getConfigValue(key, defaultValue = null) {
    try {
      const value = remoteConfig().getValue(key);
      return value.asString() || defaultValue;
    } catch (error) {
      console.error(`Error obteniendo valor ${key}:`, error);
      return defaultValue;
    }
  }

  // Obtener valor booleano
  getBooleanValue(key, defaultValue = false) {
    try {
      const value = remoteConfig().getValue(key);
      return value.asBoolean() ?? defaultValue;
    } catch (error) {
      console.error(`Error obteniendo valor booleano ${key}:`, error);
      return defaultValue;
    }
  }

  // Obtener valor numérico
  getNumberValue(key, defaultValue = 0) {
    try {
      const value = remoteConfig().getValue(key);
      return value.asNumber() ?? defaultValue;
    } catch (error) {
      console.error(`Error obteniendo valor numérico ${key}:`, error);
      return defaultValue;
    }
  }
}
```

### Tipos de Experimentos

#### 1. **Experimentos A/B Simples**
```javascript
// Sistema de experimentos A/B simples
class SimpleABTesting {
  constructor(abTestingService, analyticsService) {
    this.abTesting = abTestingService;
    this.analytics = analyticsService;
    this.experiments = new Map();
  }

  // Definir experimento A/B
  defineExperiment(experimentName, variants, metrics) {
    const experiment = {
      name: experimentName,
      variants: variants,
      metrics: metrics,
      startDate: Date.now(),
      isActive: true
    };
    
    this.experiments.set(experimentName, experiment);
    return experiment;
  }

  // Asignar usuario a variante
  assignUserToVariant(experimentName, userId) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment || !experiment.isActive) {
      return 'control'; // Variante por defecto
    }

    // Generar asignación determinística basada en userId
    const hash = this.hashString(userId + experimentName);
    const variantIndex = hash % experiment.variants.length;
    const assignedVariant = experiment.variants[variantIndex];
    
    // Registrar asignación
    this.analytics.logEvent('experiment_assignment', {
      experiment_name: experimentName,
      user_id: userId,
      assigned_variant: assignedVariant,
      timestamp: Date.now()
    });
    
    return assignedVariant;
  }

  // Hash simple para asignación determinística
  hashString(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convertir a entero de 32 bits
    }
    return Math.abs(hash);
  }

  // Trackear métrica del experimento
  trackExperimentMetric(experimentName, userId, metricName, value) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) return false;
    
    const variant = this.assignUserToVariant(experimentName, userId);
    
    // Registrar métrica
    this.analytics.logEvent('experiment_metric', {
      experiment_name: experimentName,
      user_id: userId,
      variant: variant,
      metric_name: metricName,
      metric_value: value,
      timestamp: Date.now()
    });
    
    return true;
  }

  // Obtener estadísticas del experimento
  async getExperimentStats(experimentName) {
    // Implementar lógica para obtener estadísticas desde analytics
    // Esta es una implementación simplificada
    return {
      experiment_name: experimentName,
      total_users: 0,
      variants: {},
      metrics: {}
    };
  }
}
```

#### 2. **Experimentos Multivariados**
```javascript
// Sistema de experimentos multivariados
class MultivariateTesting {
  constructor(abTestingService, analyticsService) {
    this.abTesting = abTestingService;
    this.analytics = analyticsService;
    this.experiments = new Map();
  }

  // Definir experimento multivariado
  defineMultivariateExperiment(experimentName, factors, combinations) {
    const experiment = {
      name: experimentName,
      factors: factors, // Array de factores a probar
      combinations: combinations, // Combinaciones específicas a probar
      startDate: Date.now(),
      isActive: true,
      userAssignments: new Map()
    };
    
    this.experiments.set(experimentName, experiment);
    return experiment;
  }

  // Asignar usuario a combinación
  assignUserToCombination(experimentName, userId) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment || !experiment.isActive) {
      return this.getDefaultCombination(experiment);
    }

    // Verificar si el usuario ya tiene asignación
    if (experiment.userAssignments.has(userId)) {
      return experiment.userAssignments.get(userId);
    }

    // Generar asignación aleatoria pero determinística
    const hash = this.hashString(userId + experimentName);
    const combinationIndex = hash % experiment.combinations.length;
    const assignedCombination = experiment.combinations[combinationIndex];
    
    // Guardar asignación
    experiment.userAssignments.set(userId, assignedCombination);
    
    // Registrar asignación
    this.analytics.logEvent('multivariate_assignment', {
      experiment_name: experimentName,
      user_id: userId,
      assigned_combination: assignedCombination,
      timestamp: Date.now()
    });
    
    return assignedCombination;
  }

  // Obtener combinación por defecto
  getDefaultCombination(experiment) {
    // Retornar la primera combinación como control
    return experiment.combinations[0];
  }

  // Aplicar combinación a componente
  applyCombinationToComponent(experimentName, userId, componentConfig) {
    const combination = this.assignUserToCombination(experimentName, userId);
    
    // Aplicar configuración de la combinación
    const appliedConfig = { ...componentConfig };
    
    Object.entries(combination).forEach(([factor, value]) => {
      appliedConfig[factor] = value;
    });
    
    return appliedConfig;
  }

  // Trackear métrica multivariada
  trackMultivariateMetric(experimentName, userId, metricName, value) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) return false;
    
    const combination = this.assignUserToCombination(experimentName, userId);
    
    // Registrar métrica
    this.analytics.logEvent('multivariate_metric', {
      experiment_name: experimentName,
      user_id: userId,
      combination: JSON.stringify(combination),
      metric_name: metricName,
      metric_value: value,
      timestamp: Date.now()
    });
    
    return true;
  }
}
```

#### 3. **Feature Flags y Rollouts Graduales**
```javascript
// Sistema de feature flags para rollouts graduales
class FeatureFlagService {
  constructor(abTestingService, analyticsService) {
    this.abTesting = abTestingService;
    this.analytics = analyticsService;
    this.featureFlags = new Map();
    this.userEligibility = new Map();
  }

  // Definir feature flag
  defineFeatureFlag(featureName, options = {}) {
    const featureFlag = {
      name: featureName,
      isEnabled: options.isEnabled || false,
      rolloutPercentage: options.rolloutPercentage || 0,
      targetAudience: options.targetAudience || 'all',
      startDate: options.startDate || Date.now(),
      endDate: options.endDate || null,
      conditions: options.conditions || []
    };
    
    this.featureFlags.set(featureName, featureFlag);
    return featureFlag;
  }

  // Verificar si feature está habilitado para usuario
  isFeatureEnabled(featureName, userId, userContext = {}) {
    const featureFlag = this.featureFlags.get(featureName);
    if (!featureFlag) return false;
    
    // Verificar si está habilitado globalmente
    if (!featureFlag.isEnabled) return false;
    
    // Verificar fecha de inicio
    if (Date.now() < featureFlag.startDate) return false;
    
    // Verificar fecha de fin
    if (featureFlag.endDate && Date.now() > featureFlag.endDate) return false;
    
    // Verificar condiciones específicas
    if (!this.checkUserConditions(featureFlag.conditions, userContext)) {
      return false;
    }
    
    // Verificar rollout gradual
    if (featureFlag.rolloutPercentage < 100) {
      return this.isUserInRollout(featureName, userId, featureFlag.rolloutPercentage);
    }
    
    return true;
  }

  // Verificar condiciones del usuario
  checkUserConditions(conditions, userContext) {
    if (!conditions || conditions.length === 0) return true;
    
    return conditions.every(condition => {
      const { field, operator, value } = condition;
      const userValue = userContext[field];
      
      switch (operator) {
        case 'equals':
          return userValue === value;
        case 'not_equals':
          return userValue !== value;
        case 'contains':
          return userValue && userValue.includes(value);
        case 'greater_than':
          return userValue > value;
        case 'less_than':
          return userValue < value;
        default:
          return true;
      }
    });
  }

  // Verificar si usuario está en rollout
  isUserInRollout(featureName, userId, rolloutPercentage) {
    const hash = this.hashString(userId + featureName);
    const userPercentage = (hash % 100) + 1;
    
    return userPercentage <= rolloutPercentage;
  }

  // Hash simple para asignación determinística
  hashString(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }

  // Trackear uso de feature flag
  trackFeatureFlagUsage(featureName, userId, isEnabled, userContext = {}) {
    const featureFlag = this.featureFlags.get(featureName);
    if (!featureFlag) return false;
    
    // Registrar uso
    this.analytics.logEvent('feature_flag_usage', {
      feature_name: featureName,
      user_id: userId,
      is_enabled: isEnabled,
      rollout_percentage: featureFlag.rolloutPercentage,
      user_context: JSON.stringify(userContext),
      timestamp: Date.now()
    });
    
    return true;
  }
}
```

## 🛠️ Configuración Avanzada

### Configuración de Firebase Remote Config
```javascript
// Configuración avanzada de Firebase Remote Config
import remoteConfig from '@react-native-firebase/remote-config';

class AdvancedRemoteConfigService {
  constructor() {
    this.setupAdvancedConfiguration();
  }

  // Configuración avanzada
  async setupAdvancedConfiguration() {
    try {
      // Configurar valores por defecto
      await this.setDefaultValues();
      
      // Configurar parámetros de fetch
      await this.setFetchParameters();
      
      // Configurar listeners de cambios
      this.setupConfigChangeListeners();
      
      console.log('Advanced Remote Config configurado correctamente');
    } catch (error) {
      console.error('Error configurando Advanced Remote Config:', error);
    }
  }

  // Configurar valores por defecto
  async setDefaultValues() {
    const defaultValues = {
      // Configuración de UI
      'ui_theme': 'light',
      'primary_color': '#007AFF',
      'secondary_color': '#5856D6',
      
      // Configuración de features
      'enable_chat': false,
      'enable_push_notifications': true,
      'enable_analytics': true,
      
      // Configuración de experimentos
      'experiment_welcome_flow': 'control',
      'experiment_button_style': 'rounded',
      'experiment_layout': 'standard',
      
      // Configuración de performance
      'cache_duration': 3600,
      'max_retry_attempts': 3,
      'timeout_duration': 30000
    };
    
    await remoteConfig().setDefaults(defaultValues);
  }

  // Configurar parámetros de fetch
  async setFetchParameters() {
    await remoteConfig().setConfigSettings({
      minimumFetchIntervalMillis: 1800000, // 30 minutos
      fetchTimeoutMillis: 60000, // 1 minuto
      maximumFetchTimeoutMillis: 300000 // 5 minutos
    });
  }

  // Configurar listeners de cambios
  setupConfigChangeListeners() {
    remoteConfig().onConfigUpdated(() => {
      console.log('Remote Config actualizado');
      this.handleConfigUpdate();
    });
  }

  // Manejar actualización de configuración
  async handleConfigUpdate() {
    try {
      // Activar nueva configuración
      await remoteConfig().activate();
      
      // Notificar a componentes sobre cambios
      this.notifyConfigChange();
      
    } catch (error) {
      console.error('Error activando nueva configuración:', error);
    }
  }

  // Notificar cambios de configuración
  notifyConfigChange() {
    // Implementar sistema de notificaciones
    // para que los componentes sepan que la configuración cambió
    console.log('Configuración actualizada, notificando a componentes');
  }

  // Obtener configuración con fallback
  getConfigWithFallback(key, fallbackValue) {
    try {
      const value = remoteConfig().getValue(key);
      
      // Intentar obtener como string primero
      if (value.asString()) {
        return value.asString();
      }
      
      // Intentar como booleano
      if (typeof value.asBoolean() === 'boolean') {
        return value.asBoolean();
      }
      
      // Intentar como número
      if (typeof value.asNumber() === 'number') {
        return value.asNumber();
      }
      
      // Retornar fallback si no se puede convertir
      return fallbackValue;
      
    } catch (error) {
      console.error(`Error obteniendo configuración ${key}:`, error);
      return fallbackValue;
    }
  }
}
```

### Sistema de Análisis de Experimentos
```javascript
// Sistema para analizar resultados de experimentos
class ExperimentAnalysisService {
  constructor(analyticsService) {
    this.analytics = analyticsService;
    this.experimentResults = new Map();
  }

  // Calcular estadísticas del experimento
  calculateExperimentStats(experimentName, variant, metrics) {
    const results = this.experimentResults.get(experimentName) || {};
    const variantData = results[variant] || { users: [], metrics: {} };
    
    const stats = {
      variant,
      total_users: variantData.users.length,
      metrics: {}
    };
    
    // Calcular estadísticas para cada métrica
    Object.keys(metrics).forEach(metricName => {
      const metricValues = variantData.metrics[metricName] || [];
      
      if (metricValues.length > 0) {
        stats.metrics[metricName] = {
          count: metricValues.length,
          sum: metricValues.reduce((a, b) => a + b, 0),
          mean: metricValues.reduce((a, b) => a + b, 0) / metricValues.length,
          min: Math.min(...metricValues),
          max: Math.max(...metricValues),
          median: this.calculateMedian(metricValues)
        };
      }
    });
    
    return stats;
  }

  // Calcular mediana
  calculateMedian(values) {
    const sorted = values.slice().sort((a, b) => a - b);
    const middle = Math.floor(sorted.length / 2);
    
    if (sorted.length % 2 === 0) {
      return (sorted[middle - 1] + sorted[middle]) / 2;
    }
    
    return sorted[middle];
  }

  // Calcular significancia estadística
  calculateStatisticalSignificance(controlStats, variantStats, metricName) {
    const control = controlStats.metrics[metricName];
    const variant = variantStats.metrics[metricName];
    
    if (!control || !variant) return null;
    
    // Implementar test t de Student para comparar medias
    const tStat = this.calculateTStatistic(control, variant);
    const pValue = this.calculatePValue(tStat, control.count + variant.count - 2);
    
    return {
      t_statistic: tStat,
      p_value: pValue,
      significant: pValue < 0.05, // 95% de confianza
      improvement: ((variant.mean - control.mean) / control.mean) * 100
    };
  }

  // Calcular estadístico t
  calculateTStatistic(control, variant) {
    const pooledVariance = this.calculatePooledVariance(control, variant);
    const standardError = Math.sqrt(pooledVariance * (1/control.count + 1/variant.count));
    
    return (variant.mean - control.mean) / standardError;
  }

  // Calcular varianza agrupada
  calculatePooledVariance(control, variant) {
    const controlVariance = this.calculateVariance(control.mean, control.values);
    const variantVariance = this.calculateVariance(variant.mean, variant.values);
    
    return ((control.count - 1) * controlVariance + (variant.count - 1) * variantVariance) / 
           (control.count + variant.count - 2);
  }

  // Calcular varianza
  calculateVariance(mean, values) {
    const squaredDifferences = values.map(value => Math.pow(value - mean, 2));
    return squaredDifferences.reduce((a, b) => a + b, 0) / (values.length - 1);
  }

  // Calcular valor p (simplificado)
  calculatePValue(tStat, degreesOfFreedom) {
    // Implementación simplificada del cálculo del valor p
    // En producción, usar librerías especializadas como jStat o similar
    
    const absT = Math.abs(tStat);
    
    // Aproximación simple para valores p comunes
    if (absT > 3.291) return 0.001; // 99.9% confianza
    if (absT > 2.576) return 0.01;  // 99% confianza
    if (absT > 1.96) return 0.05;   // 95% confianza
    if (absT > 1.645) return 0.1;   // 90% confianza
    
    return 0.2; // No significativo
  }

  // Generar reporte completo del experimento
  generateExperimentReport(experimentName) {
    const results = this.experimentResults.get(experimentName);
    if (!results) return null;
    
    const report = {
      experiment_name: experimentName,
      start_date: results.startDate,
      end_date: results.endDate,
      variants: {},
      analysis: {},
      recommendations: []
    };
    
    // Analizar cada variante
    Object.keys(results).forEach(variant => {
      if (variant === 'startDate' || variant === 'endDate') return;
      
      const variantStats = this.calculateExperimentStats(experimentName, variant, results.metrics);
      report.variants[variant] = variantStats;
    });
    
    // Comparar variantes
    const controlVariant = report.variants['control'];
    if (controlVariant) {
      Object.keys(report.variants).forEach(variant => {
        if (variant === 'control') return;
        
        const variantStats = report.variants[variant];
        const significance = this.calculateStatisticalSignificance(
          controlVariant, 
          variantStats, 
          'conversion_rate'
        );
        
        report.analysis[variant] = significance;
        
        // Generar recomendaciones
        if (significance && significance.significant) {
          if (significance.improvement > 0) {
            report.recommendations.push(
              `La variante ${variant} muestra una mejora significativa del ${significance.improvement.toFixed(2)}%`
            );
          } else {
            report.recommendations.push(
              `La variante ${variant} muestra una disminución significativa del ${Math.abs(significance.improvement).toFixed(2)}%`
            );
          }
        }
      });
    }
    
    return report;
  }
}
```

## 📊 Dashboard de Experimentos

### Componente de Dashboard
```javascript
// Dashboard de experimentos en React Native
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, StyleSheet, TouchableOpacity } from 'react-native';

const ExperimentDashboard = ({ experimentService, analysisService }) => {
  const [experiments, setExperiments] = useState([]);
  const [selectedExperiment, setSelectedExperiment] = useState(null);
  const [experimentReport, setExperimentReport] = useState(null);

  // Cargar experimentos
  useEffect(() => {
    loadExperiments();
  }, []);

  // Cargar experimentos
  const loadExperiments = async () => {
    try {
      const activeExperiments = await experimentService.getActiveExperiments();
      setExperiments(activeExperiments);
    } catch (error) {
      console.error('Error cargando experimentos:', error);
    }
  };

  // Seleccionar experimento
  const selectExperiment = async (experimentName) => {
    try {
      const report = await analysisService.generateExperimentReport(experimentName);
      setSelectedExperiment(experimentName);
      setExperimentReport(report);
    } catch (error) {
      console.error('Error generando reporte:', error);
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Dashboard de Experimentos</Text>
      
      {/* Lista de Experimentos */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Experimentos Activos</Text>
        {experiments.map((experiment, index) => (
          <TouchableOpacity
            key={index}
            style={[
              styles.experimentCard,
              selectedExperiment === experiment.name && styles.selectedExperimentCard
            ]}
            onPress={() => selectExperiment(experiment.name)}
          >
            <Text style={styles.experimentName}>{experiment.name}</Text>
            <Text style={styles.experimentStatus}>
              Estado: {experiment.isActive ? 'Activo' : 'Inactivo'}
            </Text>
            <Text style={styles.experimentVariants}>
              Variantes: {experiment.variants.length}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
      
      {/* Reporte del Experimento Seleccionado */}
      {experimentReport && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>
            Reporte: {experimentReport.experiment_name}
          </Text>
          
          {/* Resumen de Variantes */}
          <View style={styles.variantsSection}>
            <Text style={styles.subsectionTitle}>Variantes</Text>
            {Object.entries(experimentReport.variants).map(([variant, stats]) => (
              <View key={variant} style={styles.variantCard}>
                <Text style={styles.variantName}>{variant}</Text>
                <Text style={styles.variantStats}>
                  Usuarios: {stats.total_users}
                </Text>
                {stats.metrics.conversion_rate && (
                  <Text style={styles.variantStats}>
                    Conversión: {stats.metrics.conversion_rate.mean.toFixed(2)}%
                  </Text>
                )}
              </View>
            ))}
          </View>
          
          {/* Análisis de Significancia */}
          <View style={styles.analysisSection}>
            <Text style={styles.subsectionTitle}>Análisis de Significancia</Text>
            {Object.entries(experimentReport.analysis).map(([variant, analysis]) => (
              <View key={variant} style={styles.analysisCard}>
                <Text style={styles.analysisVariant}>{variant} vs Control</Text>
                <Text style={styles.analysisResult}>
                  P-Value: {analysis.p_value.toFixed(4)}
                </Text>
                <Text style={styles.analysisResult}>
                  Significativo: {analysis.significant ? 'Sí' : 'No'}
                </Text>
                {analysis.improvement && (
                  <Text style={[
                    styles.analysisResult,
                    { color: analysis.improvement > 0 ? '#4CAF50' : '#F44336' }
                  ]}>
                    Mejora: {analysis.improvement > 0 ? '+' : ''}{analysis.improvement.toFixed(2)}%
                  </Text>
                )}
              </View>
            ))}
          </View>
          
          {/* Recomendaciones */}
          {experimentReport.recommendations.length > 0 && (
            <View style={styles.recommendationsSection}>
              <Text style={styles.subsectionTitle}>Recomendaciones</Text>
              {experimentReport.recommendations.map((recommendation, index) => (
                <Text key={index} style={styles.recommendation}>
                  • {recommendation}
                </Text>
              ))}
            </View>
          )}
        </View>
      )}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center'
  },
  section: {
    backgroundColor: 'white',
    padding: 16,
    marginBottom: 16,
    borderRadius: 8,
    elevation: 2
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 16,
    color: '#333'
  },
  experimentCard: {
    backgroundColor: '#f8f9fa',
    padding: 16,
    borderRadius: 8,
    marginBottom: 12,
    borderWidth: 2,
    borderColor: 'transparent'
  },
  selectedExperimentCard: {
    borderColor: '#007AFF',
    backgroundColor: '#E3F2FD'
  },
  experimentName: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 4,
    color: '#333'
  },
  experimentStatus: {
    fontSize: 14,
    color: '#666',
    marginBottom: 2
  },
  experimentVariants: {
    fontSize: 14,
    color: '#007AFF'
  },
  variantsSection: {
    marginBottom: 20
  },
  subsectionTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 12,
    color: '#333'
  },
  variantCard: {
    backgroundColor: '#f8f9fa',
    padding: 12,
    borderRadius: 8,
    marginBottom: 8
  },
  variantName: {
    fontSize: 14,
    fontWeight: 'bold',
    marginBottom: 4,
    color: '#333'
  },
  variantStats: {
    fontSize: 12,
    color: '#666'
  },
  analysisSection: {
    marginBottom: 20
  },
  analysisCard: {
    backgroundColor: '#f8f9fa',
    padding: 12,
    borderRadius: 8,
    marginBottom: 8
  },
  analysisVariant: {
    fontSize: 14,
    fontWeight: 'bold',
    marginBottom: 4,
    color: '#333'
  },
  analysisResult: {
    fontSize: 12,
    color: '#666'
  },
  recommendationsSection: {
    marginBottom: 20
  },
  recommendation: {
    fontSize: 14,
    color: '#333',
    marginBottom: 4,
    lineHeight: 20
  }
});

export default ExperimentDashboard;
```

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Configuración de Firebase Remote Config
**Objetivo**: Configurar Remote Config con experimentos básicos

**Requisitos**:
1. Configurar Firebase Remote Config en el proyecto
2. Implementar sistema básico de A/B testing
3. Configurar feature flags simples
4. Verificar que la configuración se aplique correctamente

**Implementación**:
```javascript
// Implementar las clases básicas de A/B testing
// y configurar en App.js

const App = () => {
  const abTestingService = new ABTestingService();
  const simpleABTesting = new SimpleABTesting(abTestingService, analyticsService);
  const featureFlagService = new FeatureFlagService(abTestingService, analyticsService);

  useEffect(() => {
    // Configurar experimentos básicos
    setupBasicExperiments();
    
    // Configurar feature flags
    setupFeatureFlags();
  }, []);

  const setupBasicExperiments = () => {
    // Experimento de mensaje de bienvenida
    simpleABTesting.defineExperiment('welcome_message', ['control', 'variant_a', 'variant_b'], [
      'conversion_rate',
      'session_duration',
      'user_engagement'
    ]);
  };

  const setupFeatureFlags = () => {
    // Feature flag para chat
    featureFlagService.defineFeatureFlag('enable_chat', {
      isEnabled: true,
      rolloutPercentage: 50,
      targetAudience: 'premium_users'
    });
  };

  // ... resto del componente
};
```

### Ejercicio 2: Sistema de Experimentos Multivariados
**Objetivo**: Crear sistema completo de experimentos multivariados

**Requisitos**:
- Definir experimentos con múltiples factores
- Implementar asignación de usuarios a combinaciones
- Sistema de análisis de resultados
- Dashboard de experimentos en tiempo real

**Implementación**:
```javascript
// Implementar MultivariateTesting y ExperimentAnalysisService

const multivariateTesting = new MultivariateTesting(abTestingService, analyticsService);

// Definir experimento multivariado
multivariateTesting.defineMultivariateExperiment('ui_optimization', 
  ['button_style', 'color_scheme', 'layout_type'],
  [
    { button_style: 'rounded', color_scheme: 'blue', layout_type: 'standard' },
    { button_style: 'square', color_scheme: 'blue', layout_type: 'standard' },
    { button_style: 'rounded', color_scheme: 'green', layout_type: 'compact' },
    { button_style: 'square', color_scheme: 'green', layout_type: 'compact' }
  ]
);

// Usar en componentes
const OptimizedButton = ({ userId, experimentName }) => {
  const combination = multivariateTesting.assignUserToCombination(experimentName, userId);
  const buttonConfig = multivariateTesting.applyCombinationToComponent(
    experimentName, 
    userId, 
    { style: 'default', color: '#007AFF' }
  );

  return (
    <TouchableOpacity
      style={[
        styles.button,
        { 
          borderRadius: combination.button_style === 'rounded' ? 25 : 8,
          backgroundColor: combination.color_scheme === 'blue' ? '#007AFF' : '#4CAF50'
        }
      ]}
      onPress={() => {
        // Trackear métrica del experimento
        multivariateTesting.trackMultivariateMetric(
          experimentName, 
          userId, 
          'button_click', 
          1
        );
      }}
    >
      <Text style={styles.buttonText}>Click Me</Text>
    </TouchableOpacity>
  );
};
```

### Ejercicio 3: Dashboard de Experimentos Completo
**Objetivo**: Crear dashboard completo con análisis y recomendaciones

**Características**:
- Lista de experimentos activos
- Análisis de significancia estadística
- Comparación de variantes
- Recomendaciones automáticas
- Exportación de resultados

## 📚 Resumen de la Clase

### **Conceptos Clave Aprendidos**:
1. **A/B Testing** es esencial para optimización basada en datos
2. **Firebase Remote Config** proporciona configuración remota y experimentos
3. **Experimentos multivariados** para probar múltiples factores
4. **Feature flags** para rollouts graduales y control de features
5. **Análisis estadístico** para tomar decisiones informadas

### **Próximos Pasos**:
- Configurar Firebase Remote Config
- Implementar experimentos A/B básicos
- Crear sistema de feature flags
- Desarrollar dashboard de experimentos

### **Recursos Adicionales**:
- [Firebase Remote Config Documentation](https://firebase.google.com/docs/remote-config)
- [A/B Testing Best Practices](https://firebase.google.com/docs/remote-config/use-config-ios)
- [Statistical Significance in A/B Testing](https://www.optimizely.com/optimization-glossary/statistical-significance/)

---

## 🧭 Navegación

- **Clase Anterior**: [Clase 4: Analytics y User Behavior](clase_4_analytics_user_behavior.md)
- **Clase Siguiente**: [Módulo 14: Seguridad](../senior_5/)
- **[🏠 Volver al Índice Principal](../../INDICE_COMPLETO.md)**
- **[📚 Volver al README del Módulo](../README.md)**

---

**💡 Consejo**: Comienza con experimentos simples y métricas claras antes de implementar testing multivariado complejo. Es mejor tener un experimento bien ejecutado que muchos experimentos mal diseñados.
