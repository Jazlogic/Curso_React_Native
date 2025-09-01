# 📊 Clase 4: Analytics y User Behavior

## 🎯 Objetivos de la Clase
- Implementar un sistema completo de analytics
- Trackear comportamiento del usuario y eventos personalizados
- Configurar funnels de conversión y análisis de cohortes
- Implementar A/B testing y experimentos

---

## 📚 Contenido Teórico

### 1. ¿Qué es User Analytics?

#### **Definición y Propósito**
**User Analytics** es el proceso de recopilar, analizar e interpretar datos sobre el comportamiento de los usuarios para:
- **Entender patrones** de uso de la aplicación
- **Identificar puntos de fricción** en la experiencia del usuario
- **Optimizar flujos** de conversión y engagement
- **Tomar decisiones** basadas en datos reales
- **Personalizar la experiencia** del usuario

#### **Tipos de Analytics:**
- **Event Tracking**: Acciones específicas del usuario
- **User Journey**: Flujos de navegación completos
- **Conversion Funnels**: Procesos de conversión
- **Cohort Analysis**: Análisis de grupos de usuarios
- **A/B Testing**: Experimentos controlados

### 2. Firebase Analytics Avanzado

#### **Configuración de Eventos Personalizados**
```javascript
// config/analytics.js
import analytics from '@react-native-firebase/analytics';

export class AnalyticsService {
  // Eventos estándar de Firebase
  static logScreenView(screenName, screenClass) {
    analytics().logScreenView({
      screen_name: screenName,
      screen_class: screenClass
    });
  }

  static logEvent(eventName, parameters = {}) {
    analytics().logEvent(eventName, parameters);
  }

  // Eventos personalizados para e-commerce
  static logPurchase(productId, price, currency = 'USD') {
    analytics().logEvent('purchase', {
      currency: currency,
      value: price,
      items: [{
        item_id: productId,
        item_name: `Product ${productId}`,
        price: price
      }]
    });
  }

  static logAddToCart(productId, price, quantity = 1) {
    analytics().logEvent('add_to_cart', {
      currency: 'USD',
      value: price * quantity,
      items: [{
        item_id: productId,
        item_name: `Product ${productId}`,
        price: price,
        quantity: quantity
      }]
    });
  }

  // Eventos para engagement
  static logUserEngagement(action, context = {}) {
    analytics().logEvent('user_engagement', {
      engagement_time_msec: Date.now(),
      action: action,
      ...context
    });
  }

  // Eventos para onboarding
  static logOnboardingStep(stepNumber, stepName, completed = false) {
    analytics().logEvent('onboarding_step', {
      step_number: stepNumber,
      step_name: stepName,
      completed: completed,
      timestamp: Date.now()
    });
  }

  // Eventos para errores y problemas
  static logError(errorType, errorMessage, context = {}) {
    analytics().logEvent('app_error', {
      error_type: errorType,
      error_message: errorMessage,
      timestamp: Date.now(),
      ...context
    });
  }
}
```

#### **Hook para Analytics Avanzado**
```javascript
// hooks/useAdvancedAnalytics.js
import { useCallback, useEffect, useRef } from 'react';
import { AnalyticsService } from '../config/analytics';

export const useAdvancedAnalytics = (screenName) => {
  const sessionStartTime = useRef(Date.now());
  const interactionCount = useRef(0);

  useEffect(() => {
    // Registrar vista de pantalla
    AnalyticsService.logScreenView(screenName, 'Screen');
    
    // Registrar inicio de sesión en pantalla
    AnalyticsService.logEvent('screen_session_start', {
      screen_name: screenName,
      timestamp: Date.now()
    });

    return () => {
      // Registrar fin de sesión en pantalla
      const sessionDuration = Date.now() - sessionStartTime.current;
      AnalyticsService.logEvent('screen_session_end', {
        screen_name: screenName,
        duration_ms: sessionDuration,
        interaction_count: interactionCount.current
      });
    };
  }, [screenName]);

  const trackInteraction = useCallback((interactionType, details = {}) => {
    interactionCount.current++;
    
    AnalyticsService.logEvent('user_interaction', {
      screen_name: screenName,
      interaction_type: interactionType,
      interaction_count: interactionCount.current,
      timestamp: Date.now(),
      ...details
    });
  }, [screenName]);

  const trackFeatureUsage = useCallback((featureName, usageData = {}) => {
    AnalyticsService.logEvent('feature_usage', {
      screen_name: screenName,
      feature_name: featureName,
      timestamp: Date.now(),
      ...usageData
    });
  }, [screenName]);

  const trackUserJourney = useCallback((journeyStep, stepData = {}) => {
    AnalyticsService.logEvent('user_journey', {
      screen_name: screenName,
      journey_step: journeyStep,
      timestamp: Date.now(),
      ...stepData
    });
  }, [screenName]);

  return {
    trackInteraction,
    trackFeatureUsage,
    trackUserJourney
  };
};
```

### 3. Funnel Analysis y Conversion Tracking

#### **Sistema de Funnels de Conversión**
```javascript
// services/funnelAnalytics.js
import analytics from '@react-native-firebase/analytics';

export class FunnelAnalytics {
  constructor(funnelName) {
    this.funnelName = funnelName;
    this.steps = [];
    this.currentStep = 0;
    this.startTime = Date.now();
  }

  // Agregar paso al funnel
  addStep(stepName, stepOrder, required = true) {
    this.steps.push({
      name: stepName,
      order: stepOrder,
      required: required,
      completed: false,
      timestamp: null
    });
    
    // Ordenar pasos por orden
    this.steps.sort((a, b) => a.order - b.order);
  }

  // Marcar paso como completado
  completeStep(stepName, stepData = {}) {
    const step = this.steps.find(s => s.name === stepName);
    if (step) {
      step.completed = true;
      step.timestamp = Date.now();
      step.data = stepData;

      // Registrar evento de paso completado
      analytics().logEvent('funnel_step_completed', {
        funnel_name: this.funnelName,
        step_name: stepName,
        step_order: step.order,
        timestamp: Date.now(),
        ...stepData
      });

      // Verificar si es el último paso
      if (this.isFunnelCompleted()) {
        this.completeFunnel();
      }
    }
  }

  // Verificar si el funnel está completado
  isFunnelCompleted() {
    const requiredSteps = this.steps.filter(s => s.required);
    return requiredSteps.every(s => s.completed);
  }

  // Completar el funnel
  completeFunnel() {
    const totalTime = Date.now() - this.startTime;
    const completedSteps = this.steps.filter(s => s.completed);
    
    analytics().logEvent('funnel_completed', {
      funnel_name: this.funnelName,
      total_time_ms: totalTime,
      steps_completed: completedSteps.length,
      total_steps: this.steps.length,
      timestamp: Date.now()
    });
  }

  // Obtener métricas del funnel
  getFunnelMetrics() {
    const completedSteps = this.steps.filter(s => s.completed);
    const conversionRate = (completedSteps.length / this.steps.length) * 100;
    
    return {
      funnelName: this.funnelName,
      totalSteps: this.steps.length,
      completedSteps: completedSteps.length,
      conversionRate: conversionRate,
      totalTime: Date.now() - this.startTime
    };
  }
}

// Uso del sistema de funnels
export const createEcommerceFunnel = () => {
  const funnel = new FunnelAnalytics('ecommerce_purchase');
  
  funnel.addStep('product_view', 1, true);
  funnel.addStep('add_to_cart', 2, true);
  funnel.addStep('checkout_start', 3, true);
  funnel.addStep('payment_info', 4, true);
  funnel.addStep('purchase_complete', 5, true);
  
  return funnel;
};
```

#### **Hook para Tracking de Funnels**
```javascript
// hooks/useFunnelTracking.js
import { useCallback, useRef } from 'react';
import { FunnelAnalytics } from '../services/funnelAnalytics';

export const useFunnelTracking = (funnelName, steps) => {
  const funnelRef = useRef(null);

  useEffect(() => {
    // Crear instancia del funnel
    funnelRef.current = new FunnelAnalytics(funnelName);
    
    // Agregar pasos del funnel
    steps.forEach(step => {
      funnelRef.current.addStep(step.name, step.order, step.required);
    });
  }, [funnelName, steps]);

  const completeStep = useCallback((stepName, stepData = {}) => {
    if (funnelRef.current) {
      funnelRef.current.completeStep(stepName, stepData);
    }
  }, []);

  const getFunnelMetrics = useCallback(() => {
    if (funnelRef.current) {
      return funnelRef.current.getFunnelMetrics();
    }
    return null;
  }, []);

  return {
    completeStep,
    getFunnelMetrics
  };
};

// Uso en componentes
const ProductScreen = () => {
  const { completeStep } = useFunnelTracking('product_purchase', [
    { name: 'product_view', order: 1, required: true },
    { name: 'add_to_cart', order: 2, required: true }
  ]);

  const handleAddToCart = () => {
    completeStep('add_to_cart', {
      product_id: '123',
      product_name: 'Sample Product',
      price: 29.99
    });
  };

  return (
    <View>
      <Button title="Add to Cart" onPress={handleAddToCart} />
    </View>
  );
};
```

### 4. Cohort Analysis y User Segmentation

#### **Sistema de Análisis de Cohortes**
```javascript
// services/cohortAnalytics.js
import analytics from '@react-native-firebase/analytics';

export class CohortAnalytics {
  // Registrar usuario en cohorte
  static registerUserInCohort(userId, cohortType, cohortValue) {
    analytics().setUserProperty('cohort_type', cohortType);
    analytics().setUserProperty('cohort_value', cohortValue);
    analytics().setUserId(userId);
    
    // Registrar evento de cohorte
    analytics().logEvent('user_cohort_assigned', {
      user_id: userId,
      cohort_type: cohortType,
      cohort_value: cohortValue,
      timestamp: Date.now()
    });
  }

  // Registrar comportamiento de cohorte
  static trackCohortBehavior(cohortType, cohortValue, behavior, data = {}) {
    analytics().logEvent('cohort_behavior', {
      cohort_type: cohortType,
      cohort_value: cohortValue,
      behavior: behavior,
      timestamp: Date.now(),
      ...data
    });
  }

  // Registrar retención de cohorte
  static trackCohortRetention(cohortType, cohortValue, retentionDay, isRetained) {
    analytics().logEvent('cohort_retention', {
      cohort_type: cohortType,
      cohort_value: cohortValue,
      retention_day: retentionDay,
      is_retained: isRetained,
      timestamp: Date.now()
    });
  }
}

// Tipos de cohortes comunes
export const COHORT_TYPES = {
  SIGNUP_DATE: 'signup_date',
  USER_TYPE: 'user_type',
  FEATURE_USAGE: 'feature_usage',
  PURCHASE_HISTORY: 'purchase_history',
  GEOGRAPHIC: 'geographic'
};

// Uso del sistema de cohortes
export const assignUserToCohort = (userId, userData) => {
  // Determinar cohorte basada en datos del usuario
  const signupDate = new Date(userData.signupDate);
  const month = signupDate.getMonth() + 1;
  const year = signupDate.getFullYear();
  const cohortValue = `${year}-${month.toString().padStart(2, '0')}`;
  
  CohortAnalytics.registerUserInCohort(
    userId,
    COHORT_TYPES.SIGNUP_DATE,
    cohortValue
  );
  
  // Asignar cohorte por tipo de usuario
  if (userData.isPremium) {
    CohortAnalytics.registerUserInCohort(
      userId,
      COHORT_TYPES.USER_TYPE,
      'premium'
    );
  }
};
```

#### **Hook para Tracking de Cohortes**
```javascript
// hooks/useCohortTracking.js
import { useCallback, useEffect } from 'react';
import { CohortAnalytics, COHORT_TYPES } from '../services/cohortAnalytics';

export const useCohortTracking = (userId, userData) => {
  useEffect(() => {
    if (userId && userData) {
      // Asignar usuario a cohortes automáticamente
      assignUserToCohort(userId, userData);
    }
  }, [userId, userData]);

  const trackCohortBehavior = useCallback((behavior, data = {}) => {
    if (userId) {
      // Obtener cohortes del usuario desde analytics
      const cohortType = 'signup_date'; // En una app real, esto vendría de analytics
      const cohortValue = '2024-01'; // En una app real, esto vendría de analytics
      
      CohortAnalytics.trackCohortBehavior(cohortType, cohortValue, behavior, data);
    }
  }, [userId]);

  const trackRetention = useCallback((retentionDay, isRetained) => {
    if (userId) {
      const cohortType = 'signup_date';
      const cohortValue = '2024-01';
      
      CohortAnalytics.trackCohortRetention(cohortType, cohortValue, retentionDay, isRetained);
    }
  }, [userId]);

  return {
    trackCohortBehavior,
    trackRetention
  };
};
```

### 5. A/B Testing y Experimentos

#### **Sistema de A/B Testing**
```javascript
// services/abTesting.js
import analytics from '@react-native-firebase/analytics';
import { getRemoteConfig, getValue } from '@react-native-firebase/remote-config';

export class ABTestingService {
  constructor() {
    this.experiments = new Map();
    this.remoteConfig = getRemoteConfig();
  }

  // Configurar experimento
  async setupExperiment(experimentName, variants, defaultVariant) {
    try {
      // Obtener configuración remota
      await this.remoteConfig.fetchAndActivate();
      
      const experimentValue = getValue(experimentName);
      const variant = experimentValue.asString() || defaultVariant;
      
      // Registrar experimento
      analytics().logEvent('experiment_view', {
        experiment_name: experimentName,
        variant: variant,
        timestamp: Date.now()
      });
      
      this.experiments.set(experimentName, {
        name: experimentName,
        variant: variant,
        variants: variants,
        defaultVariant: defaultVariant
      });
      
      return variant;
    } catch (error) {
      console.error('Error setting up experiment:', error);
      return defaultVariant;
    }
  }

  // Obtener variante del experimento
  getExperimentVariant(experimentName) {
    const experiment = this.experiments.get(experimentName);
    return experiment ? experiment.variant : null;
  }

  // Registrar conversión del experimento
  logExperimentConversion(experimentName, conversionType, conversionData = {}) {
    const experiment = this.experiments.get(experimentName);
    if (experiment) {
      analytics().logEvent('experiment_conversion', {
        experiment_name: experimentName,
        variant: experiment.variant,
        conversion_type: conversionType,
        timestamp: Date.now(),
        ...conversionData
      });
    }
  }

  // Registrar evento del experimento
  logExperimentEvent(experimentName, eventName, eventData = {}) {
    const experiment = this.experiments.get(experimentName);
    if (experiment) {
      analytics().logEvent('experiment_event', {
        experiment_name: experimentName,
        variant: experiment.variant,
        event_name: eventName,
        timestamp: Date.now(),
        ...eventData
      });
    }
  }
}

// Uso del sistema de A/B Testing
export const abTestingService = new ABTestingService();

// Configurar experimentos
export const setupExperiments = async () => {
  // Experimento de UI
  await abTestingService.setupExperiment(
    'ui_design',
    ['modern', 'classic'],
    'modern'
  );
  
  // Experimento de onboarding
  await abTestingService.setupExperiment(
    'onboarding_flow',
    ['guided', 'minimal'],
    'guided'
  );
  
  // Experimento de precios
  await abTestingService.setupExperiment(
    'pricing_display',
    ['monthly', 'annual'],
    'monthly'
  );
};
```

#### **Hook para A/B Testing**
```javascript
// hooks/useABTesting.js
import { useCallback, useEffect, useState } from 'react';
import { abTestingService } from '../services/abTesting';

export const useABTesting = (experimentName) => {
  const [variant, setVariant] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const loadExperiment = async () => {
      try {
        const experimentVariant = abTestingService.getExperimentVariant(experimentName);
        setVariant(experimentVariant);
      } catch (error) {
        console.error('Error loading experiment:', error);
      } finally {
        setIsLoading(false);
      }
    };

    loadExperiment();
  }, [experimentName]);

  const logConversion = useCallback((conversionType, conversionData = {}) => {
    abTestingService.logExperimentConversion(experimentName, conversionType, conversionData);
  }, [experimentName]);

  const logEvent = useCallback((eventName, eventData = {}) => {
    abTestingService.logExperimentEvent(experimentName, eventName, eventData);
  }, [experimentName]);

  return {
    variant,
    isLoading,
    logConversion,
    logEvent
  };
};

// Uso en componentes
const OnboardingScreen = () => {
  const { variant, isLoading, logConversion } = useABTesting('onboarding_flow');

  useEffect(() => {
    if (variant) {
      // Registrar vista del experimento
      logEvent('screen_view', { screen_name: 'onboarding' });
    }
  }, [variant]);

  const handleOnboardingComplete = () => {
    logConversion('onboarding_complete', {
      time_spent: Date.now() - startTime,
      steps_completed: 5
    });
  };

  if (isLoading) {
    return <LoadingSpinner />;
  }

  return (
    <View>
      {variant === 'guided' ? (
        <GuidedOnboarding onComplete={handleOnboardingComplete} />
      ) : (
        <MinimalOnboarding onComplete={handleOnboardingComplete} />
      )}
    </View>
  );
};
```

### 6. Dashboard de Analytics Personalizado

#### **Sistema de Reportes de Analytics**
```javascript
// services/analyticsReporting.js
export class AnalyticsReporting {
  // Obtener resumen de eventos
  static async getEventSummary(timeRange = '7d') {
    // En una implementación real, esto se conectaría con la API de Firebase
    // Por ahora, simulamos los datos
    return {
      totalEvents: 15420,
      uniqueUsers: 3240,
      topEvents: [
        { name: 'screen_view', count: 5430 },
        { name: 'user_interaction', count: 3210 },
        { name: 'purchase', count: 890 }
      ],
      conversionRate: 12.5,
      averageSessionDuration: 180
    };
  }

  // Obtener métricas de funnel
  static async getFunnelMetrics(funnelName, timeRange = '7d') {
    return {
      funnelName: funnelName,
      totalUsers: 1000,
      stepMetrics: [
        { step: 'product_view', users: 1000, conversion: 100 },
        { step: 'add_to_cart', users: 450, conversion: 45 },
        { step: 'checkout_start', users: 200, conversion: 20 },
        { step: 'purchase_complete', users: 120, conversion: 12 }
      ],
      overallConversion: 12
    };
  }

  // Obtener métricas de cohortes
  static async getCohortMetrics(cohortType, timeRange = '30d') {
    return {
      cohortType: cohortType,
      cohorts: [
        {
          name: '2024-01',
          totalUsers: 500,
          retention: [100, 85, 72, 68, 65, 62, 60]
        },
        {
          name: '2024-02',
          totalUsers: 600,
          retention: [100, 88, 75, 70, 68, 65, 63]
        }
      ]
    };
  }

  // Obtener métricas de experimentos
  static async getExperimentMetrics(experimentName, timeRange = '7d') {
    return {
      experimentName: experimentName,
      variants: [
        {
          name: 'control',
          users: 1000,
          conversions: 120,
          conversionRate: 12
        },
        {
          name: 'variant_a',
          users: 1000,
          conversions: 135,
          conversionRate: 13.5
        }
      ],
      statisticalSignificance: 0.05
    };
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Sistema de Analytics Completo
Implementa un sistema de analytics que cubra todos los aspectos:

**Requisitos:**
- Tracking de eventos personalizados
- Sistema de funnels de conversión
- Análisis de cohortes
- A/B testing básico

**Implementación:**
```javascript
class ComprehensiveAnalytics {
  constructor() {
    this.funnels = new Map();
    this.experiments = new Map();
  }

  // Configurar funnel
  setupFunnel(funnelName, steps) {
    const funnel = new FunnelAnalytics(funnelName);
    steps.forEach(step => {
      funnel.addStep(step.name, step.order, step.required);
    });
    this.funnels.set(funnelName, funnel);
    return funnel;
  }

  // Configurar experimento
  async setupExperiment(experimentName, variants, defaultVariant) {
    const variant = await abTestingService.setupExperiment(
      experimentName,
      variants,
      defaultVariant
    );
    this.experiments.set(experimentName, variant);
    return variant;
  }

  // Trackear evento con contexto completo
  trackEvent(eventName, eventData = {}) {
    const context = {
      timestamp: Date.now(),
      session_id: this.getSessionId(),
      user_id: this.getUserId(),
      ...eventData
    };

    analytics().logEvent(eventName, context);
  }
}
```

### Ejercicio 2: Dashboard de Analytics
Crea un dashboard que muestre métricas clave:

**Requisitos:**
- Resumen de eventos y usuarios
- Métricas de funnels de conversión
- Análisis de cohortes
- Resultados de A/B testing

**Implementación:**
```javascript
class AnalyticsDashboard {
  static async generateReport(timeRange = '7d') {
    const [eventSummary, funnelMetrics, cohortMetrics, experimentMetrics] = await Promise.all([
      AnalyticsReporting.getEventSummary(timeRange),
      AnalyticsReporting.getFunnelMetrics('ecommerce_purchase', timeRange),
      AnalyticsReporting.getCohortMetrics('signup_date', timeRange),
      AnalyticsReporting.getExperimentMetrics('ui_design', timeRange)
    ]);

    return {
      eventSummary,
      funnelMetrics,
      cohortMetrics,
      experimentMetrics,
      generatedAt: new Date().toISOString()
    };
  }

  static formatMetricsForDisplay(metrics) {
    // Formatear métricas para visualización
    return {
      ...metrics,
      formattedConversionRate: `${metrics.conversionRate}%`,
      formattedSessionDuration: `${Math.round(metrics.averageSessionDuration / 60)}m`
    };
  }
}
```

---

## 🔍 Puntos Clave

1. **User Analytics** proporciona insights valiosos sobre el comportamiento del usuario
2. **Funnel Analysis** ayuda a identificar puntos de fricción en la conversión
3. **Cohort Analysis** permite entender patrones de retención y engagement
4. **A/B Testing** permite tomar decisiones basadas en datos reales
5. **Dashboard personalizado** facilita el análisis y la toma de decisiones

---

## 📖 Recursos Adicionales

- [Firebase Analytics](https://firebase.google.com/docs/analytics)
- [Google Analytics 4](https://developers.google.com/analytics/devguides/collection/ga4)
- [A/B Testing Best Practices](https://www.optimizely.com/optimization-glossary/ab-testing/)
- [Cohort Analysis Guide](https://mixpanel.com/blog/cohort-analysis-made-simple/)

---

## ➡️ Siguiente Clase
En la última clase aprenderemos sobre **A/B Testing y Experimentación** y cómo implementar un sistema completo de experimentos para optimizar la experiencia del usuario.
