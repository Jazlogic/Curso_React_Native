# 📊 Clase 2: Funnels de Conversión y Cohort Analysis

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar funnels de conversión y cohort analysis en aplicaciones React Native, incluyendo análisis de flujo de usuarios, métricas de retención, análisis de comportamiento y optimización de conversiones. Aprenderás a identificar cuellos de botella y optimizar la experiencia del usuario.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** funnels de conversión en React Native
2. **Configurar** cohort analysis y análisis de retención
3. **Analizar** comportamiento y engagement de usuarios
4. **Optimizar** métricas de conversión
5. **Identificar** cuellos de botella en el flujo de usuario

### **⏱️ Duración Estimada**
- **Total**: 2-2.5 horas
- **Teoría**: 45 minutos
- **Práctica**: 75-90 minutos

---

## 📚 Contenido de la Clase

### **1. Funnels de Conversión**

#### **¿Qué son los Funnels de Conversión?**

Los funnels de conversión son representaciones visuales del flujo que siguen los usuarios desde el primer contacto hasta la conversión final, mostrando dónde se pierden usuarios en el proceso.

```javascript
// Estructura de un funnel de conversión
const ConversionFunnel = {
  steps: [
    { name: 'App Install', conversion_rate: 100 },
    { name: 'First Open', conversion_rate: 85 },
    { name: 'Registration', conversion_rate: 60 },
    { name: 'First Purchase', conversion_rate: 25 },
    { name: 'Repeat Purchase', conversion_rate: 15 }
  ],
  metrics: {
    total_users: 10000,
    converted_users: 1500,
    overall_conversion: 15,
    drop_off_points: ['Registration', 'First Purchase']
  }
};
```

#### **Implementación de Funnels con Mixpanel**

```javascript
// Sistema de tracking de funnels
class FunnelTracker {
  constructor() {
    this.mixpanel = new Mixpanel('YOUR_PROJECT_TOKEN');
    this.funnels = new Map();
    this.userJourneys = new Map();
  }

  // Definir funnel
  defineFunnel(funnelName, steps) {
    this.funnels.set(funnelName, {
      name: funnelName,
      steps: steps,
      users: new Set(),
      conversions: new Map()
    });
  }

  // Trackear paso del funnel
  trackFunnelStep(funnelName, stepName, userId, properties = {}) {
    const funnel = this.funnels.get(funnelName);
    if (!funnel) return;

    // Agregar usuario al funnel
    funnel.users.add(userId);

    // Trackear evento
    this.mixpanel.track('Funnel Step', {
      funnel_name: funnelName,
      step_name: stepName,
      user_id: userId,
      step_number: funnel.steps.indexOf(stepName) + 1,
      ...properties
    });

    // Actualizar journey del usuario
    this.updateUserJourney(userId, funnelName, stepName);
  }

  // Actualizar journey del usuario
  updateUserJourney(userId, funnelName, stepName) {
    if (!this.userJourneys.has(userId)) {
      this.userJourneys.set(userId, new Map());
    }

    const userJourney = this.userJourneys.get(userId);
    if (!userJourney.has(funnelName)) {
      userJourney.set(funnelName, []);
    }

    userJourney.get(funnelName).push({
      step: stepName,
      timestamp: Date.now()
    });
  }

  // Calcular métricas del funnel
  calculateFunnelMetrics(funnelName) {
    const funnel = this.funnels.get(funnelName);
    if (!funnel) return null;

    const metrics = {
      total_users: funnel.users.size,
      step_conversions: [],
      drop_off_rates: [],
      overall_conversion: 0
    };

    let previousStepUsers = funnel.users.size;

    funnel.steps.forEach((step, index) => {
      const stepUsers = this.getStepUsers(funnelName, step);
      const conversionRate = (stepUsers / previousStepUsers) * 100;
      const dropOffRate = 100 - conversionRate;

      metrics.step_conversions.push({
        step: step,
        users: stepUsers,
        conversion_rate: conversionRate
      });

      metrics.drop_off_rates.push({
        step: step,
        drop_off_rate: dropOffRate
      });

      previousStepUsers = stepUsers;
    });

    metrics.overall_conversion = metrics.step_conversions[metrics.step_conversions.length - 1].conversion_rate;

    return metrics;
  }

  // Obtener usuarios que completaron un paso
  getStepUsers(funnelName, stepName) {
    let count = 0;
    this.userJourneys.forEach((journey, userId) => {
      const funnelJourney = journey.get(funnelName);
      if (funnelJourney && funnelJourney.some(step => step.step === stepName)) {
        count++;
      }
    });
    return count;
  }
}

export default new FunnelTracker();
```

#### **Funnel de E-commerce**

```javascript
// Funnel específico para e-commerce
class EcommerceFunnel {
  constructor() {
    this.funnelTracker = new FunnelTracker();
    this.setupEcommerceFunnel();
  }

  setupEcommerceFunnel() {
    this.funnelTracker.defineFunnel('ecommerce_purchase', [
      'product_view',
      'add_to_cart',
      'checkout_start',
      'payment_info',
      'purchase_complete'
    ]);
  }

  // Trackear vista de producto
  trackProductView(productId, userId, properties = {}) {
    this.funnelTracker.trackFunnelStep('ecommerce_purchase', 'product_view', userId, {
      product_id: productId,
      ...properties
    });
  }

  // Trackear agregar al carrito
  trackAddToCart(productId, userId, quantity, price, properties = {}) {
    this.funnelTracker.trackFunnelStep('ecommerce_purchase', 'add_to_cart', userId, {
      product_id: productId,
      quantity: quantity,
      price: price,
      cart_value: quantity * price,
      ...properties
    });
  }

  // Trackear inicio de checkout
  trackCheckoutStart(userId, cartValue, properties = {}) {
    this.funnelTracker.trackFunnelStep('ecommerce_purchase', 'checkout_start', userId, {
      cart_value: cartValue,
      ...properties
    });
  }

  // Trackear información de pago
  trackPaymentInfo(userId, paymentMethod, properties = {}) {
    this.funnelTracker.trackFunnelStep('ecommerce_purchase', 'payment_info', userId, {
      payment_method: paymentMethod,
      ...properties
    });
  }

  // Trackear compra completa
  trackPurchaseComplete(userId, orderId, totalAmount, properties = {}) {
    this.funnelTracker.trackFunnelStep('ecommerce_purchase', 'purchase_complete', userId, {
      order_id: orderId,
      total_amount: totalAmount,
      ...properties
    });

    // Trackear evento de conversión
    this.funnelTracker.mixpanel.track('Purchase', {
      order_id: orderId,
      revenue: totalAmount,
      user_id: userId,
      ...properties
    });
  }

  // Obtener métricas del funnel de e-commerce
  getEcommerceMetrics() {
    return this.funnelTracker.calculateFunnelMetrics('ecommerce_purchase');
  }
}

export default new EcommerceFunnel();
```

### **2. Cohort Analysis**

#### **¿Qué es Cohort Analysis?**

El cohort analysis agrupa usuarios por características compartidas (como fecha de registro) y analiza su comportamiento a lo largo del tiempo para identificar patrones de retención y engagement.

```javascript
// Estructura de cohort analysis
const CohortAnalysis = {
  cohorts: [
    {
      cohort_id: '2024-01',
      users: 1000,
      retention: {
        day_1: 85,
        day_7: 60,
        day_30: 40,
        day_90: 25
      }
    }
  ],
  metrics: {
    average_retention: 52.5,
    best_performing_cohort: '2024-01',
    retention_trend: 'increasing'
  }
};
```

#### **Implementación de Cohort Analysis**

```javascript
// Sistema de cohort analysis
class CohortAnalyzer {
  constructor() {
    this.cohorts = new Map();
    this.userActivity = new Map();
    this.retentionData = new Map();
  }

  // Crear cohort
  createCohort(cohortId, users, cohortDate) {
    this.cohorts.set(cohortId, {
      id: cohortId,
      users: new Set(users),
      cohort_date: cohortDate,
      size: users.length
    });

    // Inicializar datos de retención
    this.retentionData.set(cohortId, {
      day_1: 0,
      day_7: 0,
      day_30: 0,
      day_90: 0
    });
  }

  // Trackear actividad del usuario
  trackUserActivity(userId, activityType, timestamp = Date.now()) {
    if (!this.userActivity.has(userId)) {
      this.userActivity.set(userId, []);
    }

    this.userActivity.get(userId).push({
      type: activityType,
      timestamp: timestamp
    });
  }

  // Calcular retención de cohort
  calculateCohortRetention(cohortId, days) {
    const cohort = this.cohorts.get(cohortId);
    if (!cohort) return null;

    const cohortDate = new Date(cohort.cohort_date);
    const targetDate = new Date(cohortDate.getTime() + (days * 24 * 60 * 60 * 1000));

    let activeUsers = 0;

    cohort.users.forEach(userId => {
      const userActivities = this.userActivity.get(userId) || [];
      const hasActivityAfterTarget = userActivities.some(activity => 
        new Date(activity.timestamp) >= targetDate
      );

      if (hasActivityAfterTarget) {
        activeUsers++;
      }
    });

    const retentionRate = (activeUsers / cohort.size) * 100;

    // Actualizar datos de retención
    const retentionData = this.retentionData.get(cohortId);
    if (days === 1) retentionData.day_1 = retentionRate;
    if (days === 7) retentionData.day_7 = retentionRate;
    if (days === 30) retentionData.day_30 = retentionRate;
    if (days === 90) retentionData.day_90 = retentionRate;

    return {
      cohort_id: cohortId,
      days: days,
      active_users: activeUsers,
      total_users: cohort.size,
      retention_rate: retentionRate
    };
  }

  // Generar reporte de cohort
  generateCohortReport(cohortId) {
    const cohort = this.cohorts.get(cohortId);
    const retentionData = this.retentionData.get(cohortId);

    if (!cohort || !retentionData) return null;

    return {
      cohort_id: cohortId,
      cohort_date: cohort.cohort_date,
      cohort_size: cohort.size,
      retention: {
        day_1: retentionData.day_1,
        day_7: retentionData.day_7,
        day_30: retentionData.day_30,
        day_90: retentionData.day_90
      },
      insights: this.generateCohortInsights(cohortId, retentionData)
    };
  }

  // Generar insights del cohort
  generateCohortInsights(cohortId, retentionData) {
    const insights = [];

    // Análisis de retención temprana
    if (retentionData.day_1 > 80) {
      insights.push('Excelente retención temprana (D1 > 80%)');
    } else if (retentionData.day_1 < 60) {
      insights.push('Retención temprana baja (D1 < 60%) - Revisar onboarding');
    }

    // Análisis de retención a largo plazo
    if (retentionData.day_30 > 40) {
      insights.push('Buena retención a largo plazo (D30 > 40%)');
    } else if (retentionData.day_30 < 20) {
      insights.push('Retención a largo plazo baja (D30 < 20%) - Revisar engagement');
    }

    // Análisis de tendencia
    const retentionTrend = this.calculateRetentionTrend(retentionData);
    insights.push(`Tendencia de retención: ${retentionTrend}`);

    return insights;
  }

  // Calcular tendencia de retención
  calculateRetentionTrend(retentionData) {
    const rates = [retentionData.day_1, retentionData.day_7, retentionData.day_30, retentionData.day_90];
    const trend = rates.every((rate, index) => 
      index === 0 || rate <= rates[index - 1]
    );

    return trend ? 'decreciente (normal)' : 'anómala';
  }
}

export default new CohortAnalyzer();
```

### **3. Análisis de Comportamiento**

#### **Sistema de Análisis de Comportamiento**

```javascript
// Sistema de análisis de comportamiento de usuarios
class BehaviorAnalyzer {
  constructor() {
    this.userSessions = new Map();
    this.userActions = new Map();
    this.behaviorPatterns = new Map();
  }

  // Iniciar sesión de usuario
  startUserSession(userId, sessionId) {
    this.userSessions.set(sessionId, {
      user_id: userId,
      start_time: Date.now(),
      actions: [],
      screens_visited: new Set(),
      total_time: 0
    });
  }

  // Finalizar sesión de usuario
  endUserSession(sessionId) {
    const session = this.userSessions.get(sessionId);
    if (!session) return;

    session.end_time = Date.now();
    session.total_time = session.end_time - session.start_time;

    // Analizar comportamiento de la sesión
    this.analyzeSessionBehavior(sessionId);
  }

  // Trackear acción del usuario
  trackUserAction(userId, actionType, properties = {}) {
    const action = {
      user_id: userId,
      action_type: actionType,
      timestamp: Date.now(),
      properties: properties
    };

    // Agregar a acciones del usuario
    if (!this.userActions.has(userId)) {
      this.userActions.set(userId, []);
    }
    this.userActions.get(userId).push(action);

    // Agregar a sesión activa
    this.addActionToActiveSession(userId, action);
  }

  // Agregar acción a sesión activa
  addActionToActiveSession(userId, action) {
    this.userSessions.forEach((session, sessionId) => {
      if (session.user_id === userId && !session.end_time) {
        session.actions.push(action);
        
        if (action.properties.screen) {
          session.screens_visited.add(action.properties.screen);
        }
      }
    });
  }

  // Analizar comportamiento de sesión
  analyzeSessionBehavior(sessionId) {
    const session = this.userSessions.get(sessionId);
    if (!session) return;

    const behavior = {
      session_id: sessionId,
      user_id: session.user_id,
      duration: session.total_time,
      action_count: session.actions.length,
      screens_visited: session.screens_visited.size,
      engagement_score: this.calculateEngagementScore(session),
      behavior_pattern: this.identifyBehaviorPattern(session)
    };

    this.behaviorPatterns.set(sessionId, behavior);
    return behavior;
  }

  // Calcular score de engagement
  calculateEngagementScore(session) {
    const duration = session.total_time / 1000; // en segundos
    const actionCount = session.actions.length;
    const screenCount = session.screens_visited.size;

    // Fórmula de engagement (ajustable según necesidades)
    const engagementScore = Math.min(100, 
      (duration * 0.1) + (actionCount * 2) + (screenCount * 5)
    );

    return Math.round(engagementScore);
  }

  // Identificar patrón de comportamiento
  identifyBehaviorPattern(session) {
    const patterns = [];

    // Patrón de navegación
    if (session.screens_visited.size > 5) {
      patterns.push('heavy_navigation');
    } else if (session.screens_visited.size < 2) {
      patterns.push('minimal_navigation');
    }

    // Patrón de duración
    if (session.total_time > 600000) { // 10 minutos
      patterns.push('long_session');
    } else if (session.total_time < 30000) { // 30 segundos
      patterns.push('quick_exit');
    }

    // Patrón de acciones
    if (session.actions.length > 20) {
      patterns.push('high_activity');
    } else if (session.actions.length < 5) {
      patterns.push('low_activity');
    }

    return patterns;
  }

  // Obtener métricas de comportamiento
  getBehaviorMetrics(userId) {
    const userActions = this.userActions.get(userId) || [];
    const userSessions = Array.from(this.userSessions.values())
      .filter(session => session.user_id === userId);

    return {
      total_actions: userActions.length,
      total_sessions: userSessions.length,
      average_session_duration: this.calculateAverageSessionDuration(userSessions),
      most_common_actions: this.getMostCommonActions(userActions),
      behavior_patterns: this.getUserBehaviorPatterns(userId)
    };
  }

  // Calcular duración promedio de sesión
  calculateAverageSessionDuration(sessions) {
    if (sessions.length === 0) return 0;

    const totalDuration = sessions.reduce((sum, session) => 
      sum + (session.total_time || 0), 0
    );

    return totalDuration / sessions.length;
  }

  // Obtener acciones más comunes
  getMostCommonActions(actions) {
    const actionCounts = {};
    
    actions.forEach(action => {
      actionCounts[action.action_type] = (actionCounts[action.action_type] || 0) + 1;
    });

    return Object.entries(actionCounts)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 5)
      .map(([action, count]) => ({ action, count }));
  }

  // Obtener patrones de comportamiento del usuario
  getUserBehaviorPatterns(userId) {
    const userSessions = Array.from(this.userSessions.values())
      .filter(session => session.user_id === userId);

    const patterns = new Set();
    userSessions.forEach(session => {
      const behavior = this.behaviorPatterns.get(session.session_id);
      if (behavior && behavior.behavior_pattern) {
        behavior.behavior_pattern.forEach(pattern => patterns.add(pattern));
      }
    });

    return Array.from(patterns);
  }
}

export default new BehaviorAnalyzer();
```

### **4. Métricas de Conversión**

#### **Sistema de Métricas de Conversión**

```javascript
// Sistema de métricas de conversión
class ConversionMetrics {
  constructor() {
    this.conversions = new Map();
    this.conversionGoals = new Map();
    this.metrics = {
      total_conversions: 0,
      conversion_rate: 0,
      revenue: 0,
      average_order_value: 0
    };
  }

  // Definir objetivo de conversión
  defineConversionGoal(goalId, goalName, goalType, targetValue = 0) {
    this.conversionGoals.set(goalId, {
      id: goalId,
      name: goalName,
      type: goalType, // 'purchase', 'signup', 'download', etc.
      target_value: targetValue,
      conversions: 0,
      conversion_rate: 0
    });
  }

  // Trackear conversión
  trackConversion(userId, goalId, value = 0, properties = {}) {
    const conversion = {
      user_id: userId,
      goal_id: goalId,
      value: value,
      timestamp: Date.now(),
      properties: properties
    };

    // Agregar conversión
    if (!this.conversions.has(goalId)) {
      this.conversions.set(goalId, []);
    }
    this.conversions.get(goalId).push(conversion);

    // Actualizar métricas
    this.updateConversionMetrics(goalId, value);

    // Trackear evento
    this.trackConversionEvent(conversion);
  }

  // Actualizar métricas de conversión
  updateConversionMetrics(goalId, value) {
    const goal = this.conversionGoals.get(goalId);
    if (!goal) return;

    goal.conversions++;
    this.metrics.total_conversions++;
    this.metrics.revenue += value;

    // Calcular tasa de conversión (necesitarías el total de usuarios)
    // goal.conversion_rate = (goal.conversions / totalUsers) * 100;

    // Calcular valor promedio de orden
    if (goal.type === 'purchase') {
      const purchases = this.conversions.get(goalId) || [];
      this.metrics.average_order_value = 
        purchases.reduce((sum, conv) => sum + conv.value, 0) / purchases.length;
    }
  }

  // Trackear evento de conversión
  trackConversionEvent(conversion) {
    // Enviar a analytics
    const analytics = new AnalyticsService();
    analytics.trackEvent('Conversion', {
      goal_id: conversion.goal_id,
      value: conversion.value,
      user_id: conversion.user_id,
      ...conversion.properties
    });
  }

  // Calcular métricas de funnel
  calculateFunnelMetrics(funnelSteps, totalUsers) {
    const funnelMetrics = {
      steps: [],
      overall_conversion: 0,
      drop_off_analysis: []
    };

    let previousStepUsers = totalUsers;

    funnelSteps.forEach((step, index) => {
      const stepConversions = this.getStepConversions(step.goal_id);
      const stepUsers = stepConversions.length;
      const conversionRate = (stepUsers / previousStepUsers) * 100;
      const dropOffRate = 100 - conversionRate;

      funnelMetrics.steps.push({
        step_name: step.name,
        users: stepUsers,
        conversion_rate: conversionRate,
        drop_off_rate: dropOffRate
      });

      funnelMetrics.drop_off_analysis.push({
        step: step.name,
        drop_off_rate: dropOffRate,
        impact: dropOffRate > 50 ? 'high' : dropOffRate > 25 ? 'medium' : 'low'
      });

      previousStepUsers = stepUsers;
    });

    funnelMetrics.overall_conversion = funnelMetrics.steps[funnelMetrics.steps.length - 1].conversion_rate;

    return funnelMetrics;
  }

  // Obtener conversiones de un paso
  getStepConversions(goalId) {
    return this.conversions.get(goalId) || [];
  }

  // Generar reporte de conversión
  generateConversionReport(goalId) {
    const goal = this.conversionGoals.get(goalId);
    const conversions = this.conversions.get(goalId) || [];

    if (!goal) return null;

    return {
      goal: goal,
      total_conversions: conversions.length,
      total_revenue: conversions.reduce((sum, conv) => sum + conv.value, 0),
      average_value: conversions.length > 0 ? 
        conversions.reduce((sum, conv) => sum + conv.value, 0) / conversions.length : 0,
      conversion_trend: this.calculateConversionTrend(conversions),
      top_converting_sources: this.getTopConvertingSources(conversions)
    };
  }

  // Calcular tendencia de conversión
  calculateConversionTrend(conversions) {
    if (conversions.length < 2) return 'insufficient_data';

    const recent = conversions.slice(-7); // últimos 7 días
    const previous = conversions.slice(-14, -7); // 7 días anteriores

    const recentAvg = recent.length / 7;
    const previousAvg = previous.length / 7;

    if (recentAvg > previousAvg * 1.1) return 'increasing';
    if (recentAvg < previousAvg * 0.9) return 'decreasing';
    return 'stable';
  }

  // Obtener fuentes de conversión más efectivas
  getTopConvertingSources(conversions) {
    const sourceCounts = {};

    conversions.forEach(conversion => {
      const source = conversion.properties.source || 'unknown';
      sourceCounts[source] = (sourceCounts[source] || 0) + 1;
    });

    return Object.entries(sourceCounts)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 5)
      .map(([source, count]) => ({ source, conversions: count }));
  }
}

export default new ConversionMetrics();
```

### **5. Optimización de Conversiones**

#### **Sistema de Optimización**

```javascript
// Sistema de optimización de conversiones
class ConversionOptimizer {
  constructor() {
    this.conversionMetrics = new ConversionMetrics();
    this.optimizationExperiments = new Map();
    this.optimizationResults = new Map();
  }

  // Crear experimento de optimización
  createOptimizationExperiment(experimentId, experimentName, hypothesis, variants) {
    this.optimizationExperiments.set(experimentId, {
      id: experimentId,
      name: experimentName,
      hypothesis: hypothesis,
      variants: variants,
      start_date: Date.now(),
      status: 'active',
      results: {
        variant_a: { users: 0, conversions: 0, conversion_rate: 0 },
        variant_b: { users: 0, conversions: 0, conversion_rate: 0 }
      }
    });
  }

  // Asignar usuario a variante
  assignUserToVariant(userId, experimentId) {
    const experiment = this.optimizationExperiments.get(experimentId);
    if (!experiment || experiment.status !== 'active') return null;

    // Asignación aleatoria simple (en producción usaría un algoritmo más sofisticado)
    const variant = Math.random() < 0.5 ? 'variant_a' : 'variant_b';
    
    // Trackear asignación
    this.trackVariantAssignment(userId, experimentId, variant);
    
    return variant;
  }

  // Trackear asignación de variante
  trackVariantAssignment(userId, experimentId, variant) {
    const experiment = this.optimizationExperiments.get(experimentId);
    if (!experiment) return;

    experiment.results[variant].users++;

    // Trackear evento
    const analytics = new AnalyticsService();
    analytics.trackEvent('Experiment Assignment', {
      experiment_id: experimentId,
      variant: variant,
      user_id: userId
    });
  }

  // Trackear conversión en experimento
  trackExperimentConversion(userId, experimentId, goalId, value = 0) {
    const experiment = this.optimizationExperiments.get(experimentId);
    if (!experiment) return;

    // Obtener variante del usuario
    const variant = this.getUserVariant(userId, experimentId);
    if (!variant) return;

    // Actualizar resultados
    experiment.results[variant].conversions++;
    experiment.results[variant].conversion_rate = 
      (experiment.results[variant].conversions / experiment.results[variant].users) * 100;

    // Trackear conversión
    this.conversionMetrics.trackConversion(userId, goalId, value, {
      experiment_id: experimentId,
      variant: variant
    });
  }

  // Obtener variante del usuario
  getUserVariant(userId, experimentId) {
    // En una implementación real, esto se almacenaría en base de datos
    // Por simplicidad, usamos una función determinística
    const hash = this.hashUserId(userId + experimentId);
    return hash % 2 === 0 ? 'variant_a' : 'variant_b';
  }

  // Hash simple para asignación determinística
  hashUserId(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convertir a 32-bit integer
    }
    return Math.abs(hash);
  }

  // Analizar resultados del experimento
  analyzeExperimentResults(experimentId) {
    const experiment = this.optimizationExperiments.get(experimentId);
    if (!experiment) return null;

    const results = experiment.results;
    const variantA = results.variant_a;
    const variantB = results.variant_b;

    // Calcular significancia estadística (simplificado)
    const significance = this.calculateStatisticalSignificance(variantA, variantB);

    return {
      experiment_id: experimentId,
      experiment_name: experiment.name,
      hypothesis: experiment.hypothesis,
      results: {
        variant_a: variantA,
        variant_b: variantB,
        winner: variantA.conversion_rate > variantB.conversion_rate ? 'variant_a' : 'variant_b',
        improvement: Math.abs(variantA.conversion_rate - variantB.conversion_rate),
        statistical_significance: significance
      },
      recommendation: this.generateOptimizationRecommendation(experiment, results)
    };
  }

  // Calcular significancia estadística (simplificado)
  calculateStatisticalSignificance(variantA, variantB) {
    // Implementación simplificada - en producción usaría test estadístico real
    const minSampleSize = 100;
    const minDifference = 5; // 5% de diferencia mínima

    if (variantA.users < minSampleSize || variantB.users < minSampleSize) {
      return 'insufficient_sample_size';
    }

    const difference = Math.abs(variantA.conversion_rate - variantB.conversion_rate);
    return difference > minDifference ? 'significant' : 'not_significant';
  }

  // Generar recomendación de optimización
  generateOptimizationRecommendation(experiment, results) {
    const variantA = results.variant_a;
    const variantB = results.variant_b;

    if (variantA.conversion_rate > variantB.conversion_rate) {
      return {
        action: 'implement_variant_a',
        reason: `Variant A shows ${(variantA.conversion_rate - variantB.conversion_rate).toFixed(2)}% higher conversion rate`,
        confidence: 'high'
      };
    } else if (variantB.conversion_rate > variantA.conversion_rate) {
      return {
        action: 'implement_variant_b',
        reason: `Variant B shows ${(variantB.conversion_rate - variantA.conversion_rate).toFixed(2)}% higher conversion rate`,
        confidence: 'high'
      };
    } else {
      return {
        action: 'no_change',
        reason: 'No significant difference between variants',
        confidence: 'low'
      };
    }
  }
}

export default new ConversionOptimizer();
```

---

## 🛠️ Implementación Práctica

### **Ejemplo: Sistema Completo de Funnels y Cohort**

```javascript
// Sistema completo de análisis de conversión
import { Platform } from 'react-native';

class ConversionAnalytics {
  constructor() {
    this.funnelTracker = new FunnelTracker();
    this.cohortAnalyzer = new CohortAnalyzer();
    this.behaviorAnalyzer = new BehaviorAnalyzer();
    this.conversionMetrics = new ConversionMetrics();
    this.optimizer = new ConversionOptimizer();
    
    this.initialize();
  }

  async initialize() {
    // Configurar funnels por defecto
    this.setupDefaultFunnels();
    
    // Configurar objetivos de conversión
    this.setupConversionGoals();
    
    // Configurar experimentos
    this.setupOptimizationExperiments();
  }

  // Configurar funnels por defecto
  setupDefaultFunnels() {
    // Funnel de onboarding
    this.funnelTracker.defineFunnel('onboarding', [
      'app_install',
      'first_open',
      'permissions_granted',
      'profile_created',
      'first_action'
    ]);

    // Funnel de compra
    this.funnelTracker.defineFunnel('purchase', [
      'product_view',
      'add_to_cart',
      'checkout_start',
      'payment_info',
      'purchase_complete'
    ]);
  }

  // Configurar objetivos de conversión
  setupConversionGoals() {
    this.conversionMetrics.defineConversionGoal('signup', 'User Registration', 'signup');
    this.conversionMetrics.defineConversionGoal('purchase', 'Product Purchase', 'purchase');
    this.conversionMetrics.defineConversionGoal('subscription', 'Premium Subscription', 'subscription', 9.99);
  }

  // Configurar experimentos de optimización
  setupOptimizationExperiments() {
    this.optimizer.createOptimizationExperiment(
      'checkout_flow',
      'Checkout Flow Optimization',
      'Simplified checkout flow will increase conversion rate',
      ['current_flow', 'simplified_flow']
    );
  }

  // Trackear evento completo
  async trackEvent(eventName, properties = {}) {
    const userId = properties.user_id;
    const sessionId = properties.session_id;

    // Trackear en funnel si aplica
    if (this.isFunnelEvent(eventName)) {
      this.funnelTracker.trackFunnelStep(this.getFunnelName(eventName), eventName, userId, properties);
    }

    // Trackear en cohort si aplica
    if (this.isCohortEvent(eventName)) {
      this.cohortAnalyzer.trackUserActivity(userId, eventName);
    }

    // Trackear comportamiento
    this.behaviorAnalyzer.trackUserAction(userId, eventName, properties);

    // Trackear conversión si aplica
    if (this.isConversionEvent(eventName)) {
      this.conversionMetrics.trackConversion(userId, this.getConversionGoal(eventName), properties.value || 0, properties);
    }
  }

  // Verificar si es evento de funnel
  isFunnelEvent(eventName) {
    const funnelEvents = ['app_install', 'first_open', 'product_view', 'add_to_cart', 'checkout_start', 'purchase_complete'];
    return funnelEvents.includes(eventName);
  }

  // Obtener nombre del funnel
  getFunnelName(eventName) {
    const funnelMapping = {
      'app_install': 'onboarding',
      'first_open': 'onboarding',
      'product_view': 'purchase',
      'add_to_cart': 'purchase',
      'checkout_start': 'purchase',
      'purchase_complete': 'purchase'
    };
    return funnelMapping[eventName];
  }

  // Verificar si es evento de cohort
  isCohortEvent(eventName) {
    return ['app_open', 'screen_view', 'user_action'].includes(eventName);
  }

  // Verificar si es evento de conversión
  isConversionEvent(eventName) {
    return ['signup', 'purchase', 'subscription'].includes(eventName);
  }

  // Obtener objetivo de conversión
  getConversionGoal(eventName) {
    const goalMapping = {
      'signup': 'signup',
      'purchase': 'purchase',
      'subscription': 'subscription'
    };
    return goalMapping[eventName];
  }

  // Obtener métricas completas
  getCompleteMetrics() {
    return {
      funnels: {
        onboarding: this.funnelTracker.calculateFunnelMetrics('onboarding'),
        purchase: this.funnelTracker.calculateFunnelMetrics('purchase')
      },
      cohorts: this.getCohortMetrics(),
      behavior: this.getBehaviorMetrics(),
      conversions: this.getConversionMetrics(),
      optimization: this.getOptimizationMetrics()
    };
  }

  // Obtener métricas de cohort
  getCohortMetrics() {
    const cohorts = ['2024-01', '2024-02', '2024-03'];
    return cohorts.map(cohortId => 
      this.cohortAnalyzer.generateCohortReport(cohortId)
    ).filter(report => report !== null);
  }

  // Obtener métricas de comportamiento
  getBehaviorMetrics() {
    // Implementar lógica para obtener métricas de comportamiento
    return {
      average_session_duration: 0,
      most_common_actions: [],
      engagement_score: 0
    };
  }

  // Obtener métricas de conversión
  getConversionMetrics() {
    const goals = ['signup', 'purchase', 'subscription'];
    return goals.map(goalId => 
      this.conversionMetrics.generateConversionReport(goalId)
    ).filter(report => report !== null);
  }

  // Obtener métricas de optimización
  getOptimizationMetrics() {
    const experiments = ['checkout_flow'];
    return experiments.map(experimentId => 
      this.optimizer.analyzeExperimentResults(experimentId)
    ).filter(result => result !== null);
  }
}

export default new ConversionAnalytics();
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Implementar Funnel de E-commerce**

```javascript
// Implementar funnel completo de e-commerce
const exercise1 = {
  task: 'Crear funnel de conversión para e-commerce',
  steps: [
    '1. Definir pasos del funnel',
    '2. Implementar tracking de cada paso',
    '3. Calcular métricas de conversión',
    '4. Identificar cuellos de botella'
  ],
  expectedResult: 'Funnel funcional con métricas de conversión'
};
```

### **Ejercicio 2: Cohort Analysis**

```javascript
// Implementar análisis de cohort
const exercise2 = {
  task: 'Crear sistema de cohort analysis',
  steps: [
    '1. Crear cohorts por fecha de registro',
    '2. Trackear actividad de usuarios',
    '3. Calcular métricas de retención',
    '4. Generar reportes de cohort'
  ],
  expectedResult: 'Sistema de cohort analysis funcional'
};
```

### **Ejercicio 3: Optimización de Conversiones**

```javascript
// Implementar sistema de optimización
const exercise3 = {
  task: 'Crear sistema de optimización de conversiones',
  steps: [
    '1. Definir experimentos de optimización',
    '2. Implementar asignación de variantes',
    '3. Trackear conversiones por variante',
    '4. Analizar resultados y generar recomendaciones'
  ],
  expectedResult: 'Sistema de optimización funcional'
};
```

---

## 📊 Métricas y KPIs

### **Métricas de Funnel**

```javascript
const FunnelMetrics = {
  conversion: {
    step_conversion_rate: 'Tasa de conversión por paso',
    overall_conversion_rate: 'Tasa de conversión general',
    drop_off_rate: 'Tasa de abandono por paso',
    time_to_convert: 'Tiempo promedio para conversión'
  },
  optimization: {
    conversion_improvement: 'Mejora en conversión',
    statistical_significance: 'Significancia estadística',
    confidence_level: 'Nivel de confianza',
    sample_size: 'Tamaño de muestra'
  }
};
```

### **Métricas de Cohort**

```javascript
const CohortMetrics = {
  retention: {
    day_1_retention: 'Retención D1',
    day_7_retention: 'Retención D7',
    day_30_retention: 'Retención D30',
    day_90_retention: 'Retención D90'
  },
  engagement: {
    average_session_duration: 'Duración promedio de sesión',
    actions_per_session: 'Acciones por sesión',
    screens_per_session: 'Pantallas por sesión',
    engagement_score: 'Score de engagement'
  }
};
```

---

## 🔧 Herramientas y Recursos

### **Herramientas de Funnel Analysis**

- **Mixpanel Funnels**: [mixpanel.com/funnels](https://mixpanel.com/funnels)
- **Amplitude Funnels**: [amplitude.com/funnels](https://amplitude.com/funnels)
- **Google Analytics Funnels**: [analytics.google.com](https://analytics.google.com)

### **Herramientas de Cohort Analysis**

- **Mixpanel Cohorts**: [mixpanel.com/cohorts](https://mixpanel.com/cohorts)
- **Amplitude Cohorts**: [amplitude.com/cohorts](https://amplitude.com/cohorts)
- **Google Analytics Cohorts**: [analytics.google.com](https://analytics.google.com)

### **Recursos de Aprendizaje**

- **Funnel Analysis Guide**: [mixpanel.com/learn/funnel-analysis](https://mixpanel.com/learn/funnel-analysis)
- **Cohort Analysis Tutorial**: [amplitude.com/learn/cohort-analysis](https://amplitude.com/learn/cohort-analysis)
- **Conversion Optimization**: [optimizely.com/optimization-glossary](https://www.optimizely.com/optimization-glossary)

---

## 🚀 Próximos Pasos

### **Lo que sigue**

1. **Clase 3**: A/B Testing Avanzado y MVT
2. **Clase 4**: Dashboards de BI en Tiempo Real
3. **Clase 5**: Machine Learning para Analytics

### **Preparación**

- Configurar herramientas de funnel analysis
- Preparar datos de cohort para análisis
- Revisar métricas de conversión actuales
- Configurar experimentos de optimización

---

**🎯 Objetivo de la Clase**: Dominar el análisis de funnels de conversión y cohort analysis para optimizar la experiencia del usuario y aumentar las conversiones.

**💡 Consejo**: Los funnels y cohorts son herramientas poderosas para entender el comportamiento del usuario. Úsalos para identificar oportunidades de optimización y medir el impacto de los cambios.

---

**🚀 ¡Comienza tu viaje hacia la maestría en análisis de conversión!**
