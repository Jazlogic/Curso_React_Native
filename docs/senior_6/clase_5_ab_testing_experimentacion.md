# 🧪 Clase 5: A/B Testing y Experimentación

## 🎯 Objetivos de la Clase
- Implementar un sistema completo de A/B testing
- Configurar experimentos controlados y multivariantes
- Analizar resultados y significancia estadística
- Implementar feature flags y rollouts graduales

---

## 📚 Contenido Teórico

### 1. ¿Qué es A/B Testing?

#### **Definición y Propósito**
**A/B Testing** (también conocido como split testing) es una metodología que permite:
- **Comparar dos versiones** de una funcionalidad
- **Medir el impacto** de cambios en el comportamiento del usuario
- **Tomar decisiones** basadas en datos reales
- **Optimizar la experiencia** del usuario de manera sistemática
- **Reducir riesgos** al implementar cambios

#### **Tipos de Experimentos:**
- **A/B Testing**: Comparación de dos variantes
- **Multivariante**: Comparación de múltiples variantes
- **Split Testing**: División de tráfico entre variantes
- **Feature Flags**: Activación/desactivación de funcionalidades
- **Rollouts Graduales**: Implementación progresiva de cambios

### 2. Firebase Remote Config para A/B Testing

#### **Configuración de Remote Config**
```javascript
// config/remoteConfig.js
import { getRemoteConfig, getValue, setDefaults } from '@react-native-firebase/remote-config';

export class RemoteConfigService {
  constructor() {
    this.remoteConfig = getRemoteConfig();
    this.setupDefaults();
  }

  // Configurar valores por defecto
  setupDefaults() {
    setDefaults({
      // Configuraciones de UI
      'ui_theme': 'modern',
      'button_style': 'rounded',
      'color_scheme': 'light',
      
      // Configuraciones de funcionalidad
      'onboarding_enabled': true,
      'premium_features': false,
      'push_notifications': true,
      
      // Configuraciones de experimentos
      'experiment_variant': 'control',
      'feature_flag_new_ui': false,
      'rollout_percentage': 0
    });
  }

  // Obtener valor de configuración
  getValue(key) {
    return getValue(key);
  }

  // Obtener valor como string
  getString(key) {
    return getValue(key).asString();
  }

  // Obtener valor como boolean
  getBoolean(key) {
    return getValue(key).asBoolean();
  }

  // Obtener valor como número
  getNumber(key) {
    return getValue(key).asNumber();
  }

  // Obtener valor como JSON
  getJSON(key) {
    try {
      return JSON.parse(getValue(key).asString());
    } catch (error) {
      console.error(`Error parsing JSON for key ${key}:`, error);
      return null;
    }
  }

  // Verificar si una funcionalidad está habilitada
  isFeatureEnabled(featureKey) {
    return getValue(featureKey).asBoolean();
  }

  // Obtener porcentaje de rollout
  getRolloutPercentage(featureKey) {
    return getValue(featureKey).asNumber();
  }
}

// Instancia global del servicio
export const remoteConfigService = new RemoteConfigService();
```

#### **Hook para Remote Config**
```javascript
// hooks/useRemoteConfig.js
import { useState, useEffect } from 'react';
import { remoteConfigService } from '../config/remoteConfig';

export const useRemoteConfig = (configKeys = []) => {
  const [config, setConfig] = useState({});
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const loadConfig = async () => {
      try {
        const configValues = {};
        
        configKeys.forEach(key => {
          if (key.endsWith('_enabled') || key.endsWith('_flag')) {
            configValues[key] = remoteConfigService.getBoolean(key);
          } else if (key.endsWith('_percentage')) {
            configValues[key] = remoteConfigService.getNumber(key);
          } else if (key.endsWith('_json')) {
            configValues[key] = remoteConfigService.getJSON(key);
          } else {
            configValues[key] = remoteConfigService.getString(key);
          }
        });
        
        setConfig(configValues);
      } catch (error) {
        console.error('Error loading remote config:', error);
      } finally {
        setIsLoading(false);
      }
    };

    loadConfig();
  }, [configKeys]);

  const getValue = (key) => config[key];
  const isFeatureEnabled = (key) => config[key] === true;
  const getRolloutPercentage = (key) => config[key] || 0;

  return {
    config,
    isLoading,
    getValue,
    isFeatureEnabled,
    getRolloutPercentage
  };
};

// Uso del hook
const App = () => {
  const { config, isLoading, isFeatureEnabled } = useRemoteConfig([
    'ui_theme',
    'onboarding_enabled',
    'premium_features',
    'experiment_variant'
  ]);

  if (isLoading) {
    return <LoadingSpinner />;
  }

  return (
    <View>
      {isFeatureEnabled('onboarding_enabled') && <OnboardingFlow />}
      {isFeatureEnabled('premium_features') && <PremiumFeatures />}
      <MainContent theme={config.ui_theme} />
    </View>
  );
};
```

### 3. Sistema de A/B Testing Avanzado

#### **Gestor de Experimentos**
```javascript
// services/experimentManager.js
import analytics from '@react-native-firebase/analytics';
import { remoteConfigService } from '../config/remoteConfig';

export class ExperimentManager {
  constructor() {
    this.experiments = new Map();
    this.userAssignments = new Map();
  }

  // Registrar experimento
  registerExperiment(experimentName, config) {
    const experiment = {
      name: experimentName,
      variants: config.variants || ['control', 'variant_a'],
      trafficSplit: config.trafficSplit || [50, 50],
      startDate: config.startDate || new Date(),
      endDate: config.endDate,
      metrics: config.metrics || [],
      isActive: true
    };

    this.experiments.set(experimentName, experiment);
    
    // Registrar experimento en analytics
    analytics().logEvent('experiment_registered', {
      experiment_name: experimentName,
      variants: experiment.variants.join(','),
      traffic_split: experiment.trafficSplit.join(','),
      timestamp: Date.now()
    });

    return experiment;
  }

  // Asignar usuario a variante
  assignUserToVariant(experimentName, userId) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment || !experiment.isActive) {
      return 'control';
    }

    // Verificar si el usuario ya tiene una asignación
    const assignmentKey = `${experimentName}_${userId}`;
    if (this.userAssignments.has(assignmentKey)) {
      return this.userAssignments.get(assignmentKey);
    }

    // Asignar variante basada en el hash del userId
    const hash = this.hashUserId(userId);
    const variantIndex = hash % experiment.variants.length;
    const variant = experiment.variants[variantIndex];

    // Guardar asignación
    this.userAssignments.set(assignmentKey, variant);

    // Registrar asignación en analytics
    analytics().logEvent('experiment_assignment', {
      experiment_name: experimentName,
      user_id: userId,
      variant: variant,
      timestamp: Date.now()
    });

    return variant;
  }

  // Generar hash simple del userId
  hashUserId(userId) {
    let hash = 0;
    for (let i = 0; i < userId.length; i++) {
      const char = userId.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convertir a entero de 32 bits
    }
    return Math.abs(hash);
  }

  // Registrar evento del experimento
  logExperimentEvent(experimentName, userId, eventName, eventData = {}) {
    const variant = this.getUserVariant(experimentName, userId);
    if (variant) {
      analytics().logEvent('experiment_event', {
        experiment_name: experimentName,
        user_id: userId,
        variant: variant,
        event_name: eventName,
        timestamp: Date.now(),
        ...eventData
      });
    }
  }

  // Registrar conversión del experimento
  logExperimentConversion(experimentName, userId, conversionType, conversionData = {}) {
    const variant = this.getUserVariant(experimentName, userId);
    if (variant) {
      analytics().logEvent('experiment_conversion', {
        experiment_name: experimentName,
        user_id: userId,
        variant: variant,
        conversion_type: conversionType,
        timestamp: Date.now(),
        ...conversionData
      });
    }
  }

  // Obtener variante del usuario
  getUserVariant(experimentName, userId) {
    const assignmentKey = `${experimentName}_${userId}`;
    return this.userAssignments.get(assignmentKey) || 'control';
  }

  // Obtener estadísticas del experimento
  getExperimentStats(experimentName) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) return null;

    const stats = {
      name: experimentName,
      variants: {},
      totalUsers: 0,
      startDate: experiment.startDate,
      endDate: experiment.endDate
    };

    // Contar usuarios por variante
    this.userAssignments.forEach((variant, key) => {
      if (key.startsWith(experimentName + '_')) {
        if (!stats.variants[variant]) {
          stats.variants[variant] = { users: 0, conversions: 0 };
        }
        stats.variants[variant].users++;
        stats.totalUsers++;
      }
    });

    return stats;
  }
}

// Instancia global del gestor de experimentos
export const experimentManager = new ExperimentManager();
```

#### **Hook para Experimentos**
```javascript
// hooks/useExperiment.js
import { useState, useEffect, useCallback } from 'react';
import { experimentManager } from '../services/experimentManager';

export const useExperiment = (experimentName, userId) => {
  const [variant, setVariant] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    if (userId) {
      const userVariant = experimentManager.assignUserToVariant(experimentName, userId);
      setVariant(userVariant);
      setIsLoading(false);
    }
  }, [experimentName, userId]);

  const logEvent = useCallback((eventName, eventData = {}) => {
    if (userId) {
      experimentManager.logExperimentEvent(experimentName, userId, eventName, eventData);
    }
  }, [experimentName, userId]);

  const logConversion = useCallback((conversionType, conversionData = {}) => {
    if (userId) {
      experimentManager.logExperimentConversion(experimentName, userId, conversionType, conversionData);
    }
  }, [experimentName, userId]);

  return {
    variant,
    isLoading,
    logEvent,
    logConversion
  };
};

// Uso del hook
const ProductScreen = () => {
  const { variant, isLoading, logEvent, logConversion } = useExperiment('product_ui', 'user123');

  useEffect(() => {
    if (variant) {
      logEvent('screen_view', { screen_name: 'product' });
    }
  }, [variant, logEvent]);

  const handlePurchase = () => {
    logConversion('purchase', { amount: 29.99, product_id: '123' });
  };

  if (isLoading) {
    return <LoadingSpinner />;
  }

  return (
    <View>
      {variant === 'modern' ? (
        <ModernProductUI onPurchase={handlePurchase} />
      ) : (
        <ClassicProductUI onPurchase={handlePurchase} />
      )}
    </View>
  );
};
```

### 4. Feature Flags y Rollouts Graduales

#### **Sistema de Feature Flags**
```javascript
// services/featureFlags.js
import { remoteConfigService } from '../config/remoteConfig';
import analytics from '@react-native-firebase/analytics';

export class FeatureFlagsService {
  constructor() {
    this.flags = new Map();
    this.rolloutConfigs = new Map();
  }

  // Registrar feature flag
  registerFeatureFlag(flagName, config) {
    const featureFlag = {
      name: flagName,
      enabled: config.enabled || false,
      rolloutPercentage: config.rolloutPercentage || 0,
      targetAudience: config.targetAudience || 'all',
      conditions: config.conditions || {},
      startDate: config.startDate || new Date(),
      endDate: config.endDate
    };

    this.flags.set(flagName, featureFlag);
    return featureFlag;
  }

  // Verificar si una funcionalidad está habilitada para un usuario
  isFeatureEnabled(flagName, userId, userContext = {}) {
    const flag = this.flags.get(flagName);
    if (!flag) return false;

    // Verificar si la funcionalidad está habilitada globalmente
    if (!flag.enabled) return false;

    // Verificar condiciones de fecha
    const now = new Date();
    if (flag.startDate && now < flag.startDate) return false;
    if (flag.endDate && now > flag.endDate) return false;

    // Verificar condiciones de usuario
    if (!this.checkUserConditions(flag, userId, userContext)) return false;

    // Verificar rollout gradual
    if (flag.rolloutPercentage < 100) {
      return this.isUserInRollout(flagName, userId, flag.rolloutPercentage);
    }

    return true;
  }

  // Verificar condiciones del usuario
  checkUserConditions(flag, userId, userContext) {
    if (flag.targetAudience === 'all') return true;
    if (flag.targetAudience === 'premium' && userContext.isPremium) return true;
    if (flag.targetAudience === 'new_users' && userContext.isNewUser) return true;

    // Verificar condiciones personalizadas
    for (const [key, value] of Object.entries(flag.conditions)) {
      if (userContext[key] !== value) return false;
    }

    return true;
  }

  // Verificar si el usuario está en el rollout
  isUserInRollout(flagName, userId, rolloutPercentage) {
    const hash = this.hashUserId(userId);
    const userPercentage = hash % 100;
    return userPercentage < rolloutPercentage;
  }

  // Generar hash del userId
  hashUserId(userId) {
    let hash = 0;
    for (let i = 0; i < userId.length; i++) {
      const char = userId.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }

  // Registrar uso de feature flag
  logFeatureFlagUsage(flagName, userId, enabled, userContext = {}) {
    analytics().logEvent('feature_flag_usage', {
      flag_name: flagName,
      user_id: userId,
      enabled: enabled,
      timestamp: Date.now(),
      ...userContext
    });
  }

  // Obtener estado de todos los feature flags
  getAllFeatureFlags() {
    return Array.from(this.flags.values());
  }

  // Obtener feature flags habilitados para un usuario
  getEnabledFeaturesForUser(userId, userContext = {}) {
    const enabledFeatures = [];
    
    this.flags.forEach((flag, flagName) => {
      if (this.isFeatureEnabled(flagName, userId, userContext)) {
        enabledFeatures.push(flagName);
      }
    });

    return enabledFeatures;
  }
}

// Instancia global del servicio de feature flags
export const featureFlagsService = new FeatureFlagsService();
```

#### **Hook para Feature Flags**
```javascript
// hooks/useFeatureFlag.js
import { useState, useEffect, useCallback } from 'react';
import { featureFlagsService } from '../services/featureFlags';

export const useFeatureFlag = (flagName, userId, userContext = {}) => {
  const [isEnabled, setIsEnabled] = useState(false);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    if (userId) {
      const enabled = featureFlagsService.isFeatureEnabled(flagName, userId, userContext);
      setIsEnabled(enabled);
      setIsLoading(false);

      // Registrar uso del feature flag
      featureFlagsService.logFeatureFlagUsage(flagName, userId, enabled, userContext);
    }
  }, [flagName, userId, userContext]);

  const checkFeatureFlag = useCallback(() => {
    if (userId) {
      return featureFlagsService.isFeatureEnabled(flagName, userId, userContext);
    }
    return false;
  }, [flagName, userId, userContext]);

  return {
    isEnabled,
    isLoading,
    checkFeatureFlag
  };
};

// Hook para múltiples feature flags
export const useFeatureFlags = (flagNames, userId, userContext = {}) => {
  const [flags, setFlags] = useState({});
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    if (userId) {
      const flagStates = {};
      
      flagNames.forEach(flagName => {
        flagStates[flagName] = featureFlagsService.isFeatureEnabled(flagName, userId, userContext);
      });
      
      setFlags(flagStates);
      setIsLoading(false);
    }
  }, [flagNames, userId, userContext]);

  return {
    flags,
    isLoading
  };
};

// Uso de los hooks
const App = () => {
  const { isEnabled: isNewUIEnabled } = useFeatureFlag('new_ui_enabled', 'user123', { isPremium: true });
  const { flags } = useFeatureFlags(['beta_features', 'advanced_analytics'], 'user123');

  return (
    <View>
      {isNewUIEnabled ? <NewUI /> : <ClassicUI />}
      {flags.beta_features && <BetaFeatures />}
      {flags.advanced_analytics && <AdvancedAnalytics />}
    </View>
  );
};
```

### 5. Análisis de Resultados y Significancia Estadística

#### **Servicio de Análisis Estadístico**
```javascript
// services/statisticalAnalysis.js
export class StatisticalAnalysis {
  // Calcular tasa de conversión
  static calculateConversionRate(conversions, totalUsers) {
    if (totalUsers === 0) return 0;
    return (conversions / totalUsers) * 100;
  }

  // Calcular diferencia porcentual
  static calculatePercentageDifference(controlRate, variantRate) {
    if (controlRate === 0) return 0;
    return ((variantRate - controlRate) / controlRate) * 100;
  }

  // Calcular significancia estadística (test de chi-cuadrado)
  static calculateStatisticalSignificance(controlData, variantData) {
    const { conversions: controlConversions, users: controlUsers } = controlData;
    const { conversions: variantConversions, users: variantUsers } = variantData;

    const controlRate = controlConversions / controlUsers;
    const variantRate = variantConversions / variantUsers;

    // Test de chi-cuadrado simplificado
    const expectedControl = controlUsers * ((controlConversions + variantConversions) / (controlUsers + variantUsers));
    const expectedVariant = variantUsers * ((controlConversions + variantConversions) / (controlUsers + variantUsers));

    const chiSquare = Math.pow(controlConversions - expectedControl, 2) / expectedControl +
                     Math.pow(variantConversions - expectedVariant, 2) / expectedVariant;

    // Para 1 grado de libertad, chi-cuadrado crítico para p < 0.05 es 3.841
    const isSignificant = chiSquare > 3.841;
    const pValue = this.estimatePValue(chiSquare);

    return {
      chiSquare,
      pValue,
      isSignificant,
      controlRate,
      variantRate,
      improvement: this.calculatePercentageDifference(controlRate * 100, variantRate * 100)
    };
  }

  // Estimar valor p basado en chi-cuadrado
  static estimatePValue(chiSquare) {
    // Aproximación simple del valor p
    if (chiSquare > 10.83) return 0.001;
    if (chiSquare > 6.64) return 0.01;
    if (chiSquare > 3.84) return 0.05;
    if (chiSquare > 2.71) return 0.1;
    return 0.5;
  }

  // Calcular tamaño de muestra requerido
  static calculateRequiredSampleSize(controlRate, mde, confidenceLevel = 0.95, power = 0.8) {
    const alpha = 1 - confidenceLevel;
    const beta = 1 - power;
    
    // Valores críticos para z
    const zAlpha = 1.96; // Para alpha = 0.05
    const zBeta = 0.84;  // Para power = 0.8
    
    const p = controlRate / 100;
    const delta = mde / 100;
    
    const numerator = Math.pow(zAlpha + zBeta, 2) * (2 * p * (1 - p));
    const denominator = Math.pow(delta, 2);
    
    return Math.ceil(numerator / denominator);
  }

  // Verificar si el experimento tiene suficiente poder estadístico
  static hasStatisticalPower(controlData, variantData, mde = 5) {
    const { users: controlUsers } = controlData;
    const { users: variantUsers } = variantData;
    
    const totalUsers = controlUsers + variantUsers;
    const requiredSampleSize = this.calculateRequiredSampleSize(10, mde); // Asumiendo 10% de conversión base
    
    return totalUsers >= requiredSampleSize;
  }
}

// Uso del análisis estadístico
export const analyzeExperimentResults = (experimentName, controlData, variantData) => {
  const analysis = StatisticalAnalysis.calculateStatisticalSignificance(controlData, variantData);
  const hasPower = StatisticalAnalysis.hasStatisticalPower(controlData, variantData);
  
  return {
    experimentName,
    ...analysis,
    hasStatisticalPower: hasPower,
    recommendation: getRecommendation(analysis, hasPower)
  };
};

const getRecommendation = (analysis, hasPower) => {
  if (!hasPower) {
    return 'Necesita más usuarios para resultados confiables';
  }
  
  if (!analysis.isSignificant) {
    return 'No hay diferencia estadísticamente significativa';
  }
  
  if (analysis.improvement > 0) {
    return `Implementar variante (mejora del ${analysis.improvement.toFixed(2)}%)`;
  } else {
    return 'Mantener control (la variante no mejora la conversión)';
  }
};
```

### 6. Dashboard de Experimentos

#### **Sistema de Reportes de Experimentos**
```javascript
// services/experimentReporting.js
import { StatisticalAnalysis } from './statisticalAnalysis';

export class ExperimentReporting {
  // Generar reporte completo del experimento
  static async generateExperimentReport(experimentName, timeRange = '7d') {
    // En una implementación real, esto se conectaría con la API de Firebase
    const experimentData = await this.fetchExperimentData(experimentName, timeRange);
    
    if (!experimentData) {
      return { error: 'No se encontraron datos para el experimento' };
    }

    const analysis = StatisticalAnalysis.calculateStatisticalSignificance(
      experimentData.control,
      experimentData.variant
    );

    const hasPower = StatisticalAnalysis.hasStatisticalPower(
      experimentData.control,
      experimentData.variant
    );

    return {
      experimentName,
      timeRange,
      control: {
        ...experimentData.control,
        conversionRate: StatisticalAnalysis.calculateConversionRate(
          experimentData.control.conversions,
          experimentData.control.users
        )
      },
      variant: {
        ...experimentData.variant,
        conversionRate: StatisticalAnalysis.calculateConversionRate(
          experimentData.variant.conversions,
          experimentData.variant.users
        )
      },
      analysis: {
        ...analysis,
        hasStatisticalPower: hasPower,
        recommendation: this.getRecommendation(analysis, hasPower)
      },
      generatedAt: new Date().toISOString()
    };
  }

  // Obtener datos del experimento (simulado)
  static async fetchExperimentData(experimentName, timeRange) {
    // Simular datos de experimento
    return {
      control: {
        users: 1000,
        conversions: 120,
        revenue: 1200
      },
      variant: {
        users: 1000,
        conversions: 135,
        revenue: 1350
      }
    };
  }

  // Generar recomendación basada en análisis
  static getRecommendation(analysis, hasPower) {
    if (!hasPower) {
      return {
        action: 'wait',
        message: 'Necesita más usuarios para resultados confiables',
        priority: 'medium'
      };
    }
    
    if (!analysis.isSignificant) {
      return {
        action: 'maintain',
        message: 'No hay diferencia estadísticamente significativa',
        priority: 'low'
      };
    }
    
    if (analysis.improvement > 0) {
      return {
        action: 'implement',
        message: `Implementar variante (mejora del ${analysis.improvement.toFixed(2)}%)`,
        priority: 'high'
      };
    } else {
      return {
        action: 'reject',
        message: 'Mantener control (la variante no mejora la conversión)',
        priority: 'medium'
      };
    }
  }

  // Generar reporte de todos los experimentos activos
  static async generateAllExperimentsReport() {
    const experiments = ['ui_design', 'onboarding_flow', 'pricing_display'];
    const reports = await Promise.all(
      experiments.map(exp => this.generateExperimentReport(exp))
    );

    return {
      totalExperiments: experiments.length,
      activeExperiments: reports.filter(r => !r.error).length,
      experiments: reports,
      summary: this.generateSummary(reports),
      generatedAt: new Date().toISOString()
    };
  }

  // Generar resumen de todos los experimentos
  static generateSummary(reports) {
    const validReports = reports.filter(r => !r.error);
    
    const totalUsers = validReports.reduce((sum, r) => 
      sum + r.control.users + r.variant.users, 0
    );
    
    const significantExperiments = validReports.filter(r => 
      r.analysis.isSignificant
    ).length;
    
    const recommendedImplementations = validReports.filter(r => 
      r.analysis.recommendation.action === 'implement'
    ).length;

    return {
      totalUsers,
      significantExperiments,
      recommendedImplementations,
      overallSuccessRate: (recommendedImplementations / validReports.length) * 100
    };
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Sistema de A/B Testing Completo
Implementa un sistema completo de A/B testing:

**Requisitos:**
- Gestión de experimentos
- Asignación de usuarios a variantes
- Tracking de eventos y conversiones
- Análisis estadístico básico

**Implementación:**
```javascript
class CompleteABTestingSystem {
  constructor() {
    this.experimentManager = new ExperimentManager();
    this.featureFlagsService = new FeatureFlagsService();
  }

  // Configurar experimento completo
  async setupCompleteExperiment(experimentName, config) {
    // Registrar experimento
    const experiment = this.experimentManager.registerExperiment(experimentName, config);
    
    // Configurar feature flags relacionados
    config.variants.forEach(variant => {
      this.featureFlagsService.registerFeatureFlag(
        `${experimentName}_${variant}`,
        { enabled: true, rolloutPercentage: 100 }
      );
    });
    
    return experiment;
  }

  // Ejecutar experimento
  runExperiment(experimentName, userId, userContext = {}) {
    const variant = this.experimentManager.assignUserToVariant(experimentName, userId);
    const isEnabled = this.featureFlagsService.isFeatureEnabled(
      `${experimentName}_${variant}`,
      userId,
      userContext
    );
    
    return { variant, isEnabled };
  }
}
```

### Ejercicio 2: Dashboard de Experimentos
Crea un dashboard que muestre resultados de experimentos:

**Requisitos:**
- Lista de experimentos activos
- Métricas de conversión por variante
- Análisis de significancia estadística
- Recomendaciones de acción

**Implementación:**
```javascript
class ExperimentDashboard {
  static async generateDashboard() {
    const allExperimentsReport = await ExperimentReporting.generateAllExperimentsReport();
    
    return {
      overview: allExperimentsReport.summary,
      experiments: allExperimentsReport.experiments.map(exp => ({
        name: exp.experimentName,
        status: exp.analysis.recommendation.action,
        improvement: exp.analysis.improvement,
        significance: exp.analysis.isSignificant,
        recommendation: exp.analysis.recommendation.message
      })),
      insights: this.generateInsights(allExperimentsReport.experiments)
    };
  }

  static generateInsights(experiments) {
    const insights = [];
    
    experiments.forEach(exp => {
      if (exp.analysis.improvement > 10) {
        insights.push(`🚀 ${exp.experimentName} muestra una mejora significativa del ${exp.analysis.improvement.toFixed(2)}%`);
      }
      
      if (!exp.analysis.hasStatisticalPower) {
        insights.push(`⚠️ ${exp.experimentName} necesita más usuarios para resultados confiables`);
      }
    });
    
    return insights;
  }
}
```

---

## 🔍 Puntos Clave

1. **A/B Testing** permite tomar decisiones basadas en datos reales
2. **Remote Config** facilita la configuración dinámica de experimentos
3. **Feature Flags** permiten rollouts graduales y controlados
4. **Análisis estadístico** es crucial para interpretar resultados
5. **Dashboard de experimentos** facilita el seguimiento y la toma de decisiones

---

## 📖 Recursos Adicionales

- [Firebase Remote Config](https://firebase.google.com/docs/remote-config)
- [A/B Testing Best Practices](https://www.optimizely.com/optimization-glossary/ab-testing/)
- [Statistical Significance in A/B Testing](https://www.optimizely.com/optimization-glossary/statistical-significance/)
- [Feature Flag Best Practices](https://launchdarkly.com/blog/feature-flag-best-practices/)

---

## 🎉 ¡Módulo Completado!

Has completado exitosamente el **Módulo 13: Monitoreo y Analytics** 🚀

**Lo que has aprendido:**
- ✅ Fundamentos de monitoreo y analytics
- ✅ Implementación de Crashlytics y error tracking
- ✅ Performance monitoring avanzado
- ✅ Analytics y análisis del comportamiento del usuario
- ✅ A/B testing y experimentación

**Próximos pasos:**
- Implementar estos sistemas en tu aplicación
- Configurar dashboards personalizados
- Continuar con el siguiente módulo del curso

¡Excelente trabajo! 🎯
