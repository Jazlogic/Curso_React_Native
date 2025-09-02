# 📊 Clase 3: A/B Testing Avanzado y MVT

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar A/B testing avanzado y Multivariate Testing (MVT) en aplicaciones React Native, incluyendo significancia estadística, testing de hipótesis, optimización continua de conversiones y herramientas avanzadas de testing. Aprenderás a diseñar experimentos robustos que generen insights accionables.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** A/B testing avanzado en React Native
2. **Configurar** Multivariate Testing (MVT)
3. **Calcular** significancia estadística y sample sizes
4. **Diseñar** testing de hipótesis y experimentación
5. **Optimizar** conversiones de manera continua

### **⏱️ Duración Estimada**
- **Total**: 2-2.5 horas
- **Teoría**: 45 minutos
- **Práctica**: 75-90 minutos

---

## 📚 Contenido de la Clase

### **1. A/B Testing vs Multivariate Testing (MVT)**

#### **¿Qué es A/B Testing?**

El A/B testing es un método de experimentación que compara dos versiones de una aplicación para determinar cuál funciona mejor. Es la base de la optimización basada en datos.

```javascript
// Estructura de A/B Testing
const ABTest = {
  experiment: {
    id: 'checkout_button_color',
    name: 'Checkout Button Color Test',
    hypothesis: 'Red buttons will increase conversions by 15%',
    variants: {
      control: { name: 'Blue Button', color: '#007AFF' },
      treatment: { name: 'Red Button', color: '#FF3B30' }
    }
  },
  metrics: {
    sample_size: 10000,
    confidence_level: 95,
    statistical_power: 80,
    minimum_detectable_effect: 15
  }
};
```

#### **¿Qué es Multivariate Testing (MVT)?**

El MVT permite probar múltiples variables simultáneamente, creando combinaciones de diferentes elementos para encontrar la combinación óptima.

```javascript
// Estructura de MVT
const MVTTest = {
  experiment: {
    id: 'landing_page_optimization',
    name: 'Landing Page MVT',
    hypothesis: 'Combination of headline + CTA + image will increase conversions',
    variables: {
      headline: ['Original', 'Benefit-focused', 'Urgency-driven'],
      cta: ['Get Started', 'Try Free', 'Sign Up Now'],
      image: ['Product', 'People', 'Abstract']
    },
    combinations: 27 // 3 × 3 × 3
  },
  metrics: {
    total_sample_size: 27000,
    per_variant_sample: 1000,
    confidence_level: 95
  }
};
```

### **2. Implementación de A/B Testing**

#### **Sistema de A/B Testing**

```javascript
// Sistema completo de A/B Testing
class ABTestingSystem {
  constructor() {
    this.experiments = new Map();
    this.userAssignments = new Map();
    this.results = new Map();
    this.analytics = new AnalyticsService();
  }

  // Crear experimento A/B
  createExperiment(experimentId, experimentName, hypothesis, variants, metrics) {
    this.experiments.set(experimentId, {
      id: experimentId,
      name: experimentName,
      hypothesis: hypothesis,
      variants: variants,
      metrics: metrics,
      status: 'active',
      startDate: Date.now(),
      endDate: null,
      results: {
        control: { users: 0, conversions: 0, conversionRate: 0 },
        treatment: { users: 0, conversions: 0, conversionRate: 0 }
      }
    });
  }

  // Asignar usuario a variante
  assignUserToVariant(userId, experimentId) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment || experiment.status !== 'active') return null;

    // Verificar si ya está asignado
    if (this.userAssignments.has(`${userId}_${experimentId}`)) {
      return this.userAssignments.get(`${userId}_${experimentId}`);
    }

    // Asignación determinística basada en hash
    const variant = this.getVariantForUser(userId, experimentId);
    
    // Almacenar asignación
    this.userAssignments.set(`${userId}_${experimentId}`, variant);
    
    // Trackear asignación
    this.trackVariantAssignment(userId, experimentId, variant);
    
    return variant;
  }

  // Obtener variante para usuario (determinística)
  getVariantForUser(userId, experimentId) {
    const hash = this.hashUserId(userId + experimentId);
    const variants = Object.keys(this.experiments.get(experimentId).variants);
    const variantIndex = hash % variants.length;
    return variants[variantIndex];
  }

  // Hash determinístico
  hashUserId(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }

  // Trackear asignación de variante
  trackVariantAssignment(userId, experimentId, variant) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment) return;

    // Actualizar contador de usuarios
    experiment.results[variant].users++;

    // Trackear evento
    this.analytics.trackEvent('Experiment Assignment', {
      experiment_id: experimentId,
      variant: variant,
      user_id: userId,
      assignment_timestamp: Date.now()
    });
  }

  // Trackear conversión en experimento
  trackConversion(userId, experimentId, conversionValue = 0, properties = {}) {
    const variant = this.userAssignments.get(`${userId}_${experimentId}`);
    if (!variant) return;

    const experiment = this.experiments.get(experimentId);
    if (!experiment) return;

    // Actualizar resultados
    experiment.results[variant].conversions++;
    experiment.results[variant].conversionRate = 
      (experiment.results[variant].conversions / experiment.results[variant].users) * 100;

    // Trackear conversión
    this.analytics.trackEvent('Experiment Conversion', {
      experiment_id: experimentId,
      variant: variant,
      user_id: userId,
      conversion_value: conversionValue,
      ...properties
    });
  }

  // Analizar resultados del experimento
  analyzeResults(experimentId) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment) return null;

    const results = experiment.results;
    const control = results.control;
    const treatment = results.treatment;

    // Calcular métricas estadísticas
    const analysis = {
      experiment_id: experimentId,
      experiment_name: experiment.name,
      hypothesis: experiment.hypothesis,
      results: {
        control: control,
        treatment: treatment,
        difference: treatment.conversionRate - control.conversionRate,
        relative_improvement: ((treatment.conversionRate - control.conversionRate) / control.conversionRate) * 100
      },
      statistical_analysis: this.performStatisticalAnalysis(control, treatment),
      recommendation: this.generateRecommendation(control, treatment)
    };

    return analysis;
  }

  // Análisis estadístico
  performStatisticalAnalysis(control, treatment) {
    const n1 = control.users;
    const n2 = treatment.users;
    const p1 = control.conversionRate / 100;
    const p2 = treatment.conversionRate / 100;

    // Pooled proportion
    const p = (control.conversions + treatment.conversions) / (n1 + n2);
    
    // Standard error
    const se = Math.sqrt(p * (1 - p) * (1/n1 + 1/n2));
    
    // Z-score
    const z = (p2 - p1) / se;
    
    // P-value (aproximado)
    const pValue = this.calculatePValue(z);
    
    // Confidence interval
    const ci = this.calculateConfidenceInterval(p1, p2, se);

    return {
      z_score: z,
      p_value: pValue,
      confidence_interval: ci,
      is_significant: pValue < 0.05,
      statistical_power: this.calculateStatisticalPower(n1, n2, p1, p2)
    };
  }

  // Calcular p-value (aproximado)
  calculatePValue(z) {
    // Aproximación simple para p-value
    const absZ = Math.abs(z);
    if (absZ > 2.58) return 0.01;
    if (absZ > 1.96) return 0.05;
    if (absZ > 1.65) return 0.10;
    return 0.20;
  }

  // Calcular intervalo de confianza
  calculateConfidenceInterval(p1, p2, se) {
    const margin = 1.96 * se; // 95% confidence
    const difference = p2 - p1;
    return {
      lower: difference - margin,
      upper: difference + margin,
      margin: margin
    };
  }

  // Calcular poder estadístico
  calculateStatisticalPower(n1, n2, p1, p2) {
    // Implementación simplificada
    const effectSize = Math.abs(p2 - p1);
    const avgN = (n1 + n2) / 2;
    
    if (effectSize > 0.1 && avgN > 1000) return 'high';
    if (effectSize > 0.05 && avgN > 500) return 'medium';
    return 'low';
  }

  // Generar recomendación
  generateRecommendation(control, treatment) {
    const analysis = this.performStatisticalAnalysis(control, treatment);
    
    if (!analysis.is_significant) {
      return {
        action: 'continue_testing',
        reason: 'No significant difference detected',
        confidence: 'low'
      };
    }

    if (treatment.conversionRate > control.conversionRate) {
      return {
        action: 'implement_treatment',
        reason: `Treatment shows ${((treatment.conversionRate - control.conversionRate) / control.conversionRate * 100).toFixed(1)}% improvement`,
        confidence: 'high'
      };
    } else {
      return {
        action: 'keep_control',
        reason: 'Control performs better',
        confidence: 'high'
      };
    }
  }
}

export default new ABTestingSystem();
```

### **3. Multivariate Testing (MVT)**

#### **Sistema de MVT**

```javascript
// Sistema de Multivariate Testing
class MVTTestingSystem {
  constructor() {
    this.experiments = new Map();
    this.userAssignments = new Map();
    this.results = new Map();
    this.analytics = new AnalyticsService();
  }

  // Crear experimento MVT
  createMVTExperiment(experimentId, experimentName, hypothesis, variables, metrics) {
    const combinations = this.generateCombinations(variables);
    
    this.experiments.set(experimentId, {
      id: experimentId,
      name: experimentName,
      hypothesis: hypothesis,
      variables: variables,
      combinations: combinations,
      metrics: metrics,
      status: 'active',
      startDate: Date.now(),
      results: this.initializeResults(combinations)
    });
  }

  // Generar todas las combinaciones
  generateCombinations(variables) {
    const variableNames = Object.keys(variables);
    const combinations = [];

    const generateCombination = (index, currentCombination) => {
      if (index === variableNames.length) {
        combinations.push({
          id: combinations.length,
          combination: { ...currentCombination }
        });
        return;
      }

      const variableName = variableNames[index];
      const values = variables[variableName];

      values.forEach(value => {
        currentCombination[variableName] = value;
        generateCombination(index + 1, currentCombination);
      });
    };

    generateCombination(0, {});
    return combinations;
  }

  // Inicializar resultados
  initializeResults(combinations) {
    const results = {};
    combinations.forEach(combo => {
      results[combo.id] = {
        users: 0,
        conversions: 0,
        conversionRate: 0,
        combination: combo.combination
      };
    });
    return results;
  }

  // Asignar usuario a combinación
  assignUserToCombination(userId, experimentId) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment || experiment.status !== 'active') return null;

    // Verificar si ya está asignado
    if (this.userAssignments.has(`${userId}_${experimentId}`)) {
      return this.userAssignments.get(`${userId}_${experimentId}`);
    }

    // Asignación determinística
    const combinationId = this.getCombinationForUser(userId, experimentId);
    
    // Almacenar asignación
    this.userAssignments.set(`${userId}_${experimentId}`, combinationId);
    
    // Trackear asignación
    this.trackCombinationAssignment(userId, experimentId, combinationId);
    
    return combinationId;
  }

  // Obtener combinación para usuario
  getCombinationForUser(userId, experimentId) {
    const experiment = this.experiments.get(experimentId);
    const hash = this.hashUserId(userId + experimentId);
    return hash % experiment.combinations.length;
  }

  // Hash determinístico
  hashUserId(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }

  // Trackear asignación de combinación
  trackCombinationAssignment(userId, experimentId, combinationId) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment) return;

    // Actualizar contador de usuarios
    experiment.results[combinationId].users++;

    // Trackear evento
    this.analytics.trackEvent('MVT Assignment', {
      experiment_id: experimentId,
      combination_id: combinationId,
      combination: experiment.results[combinationId].combination,
      user_id: userId,
      assignment_timestamp: Date.now()
    });
  }

  // Trackear conversión en MVT
  trackConversion(userId, experimentId, conversionValue = 0, properties = {}) {
    const combinationId = this.userAssignments.get(`${userId}_${experimentId}`);
    if (combinationId === undefined) return;

    const experiment = this.experiments.get(experimentId);
    if (!experiment) return;

    // Actualizar resultados
    experiment.results[combinationId].conversions++;
    experiment.results[combinationId].conversionRate = 
      (experiment.results[combinationId].conversions / experiment.results[combinationId].users) * 100;

    // Trackear conversión
    this.analytics.trackEvent('MVT Conversion', {
      experiment_id: experimentId,
      combination_id: combinationId,
      combination: experiment.results[combinationId].combination,
      user_id: userId,
      conversion_value: conversionValue,
      ...properties
    });
  }

  // Analizar resultados MVT
  analyzeMVTResults(experimentId) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment) return null;

    const results = experiment.results;
    const combinations = Object.values(results);
    
    // Encontrar la mejor combinación
    const bestCombination = combinations.reduce((best, current) => 
      current.conversionRate > best.conversionRate ? current : best
    );

    // Análisis de varianza (simplificado)
    const anova = this.performANOVA(combinations);

    // Análisis de efectos principales
    const mainEffects = this.analyzeMainEffects(experiment.variables, results);

    return {
      experiment_id: experimentId,
      experiment_name: experiment.name,
      hypothesis: experiment.hypothesis,
      best_combination: bestCombination,
      all_combinations: combinations,
      anova: anova,
      main_effects: mainEffects,
      recommendation: this.generateMVTRecommendation(bestCombination, combinations)
    };
  }

  // Análisis de varianza (simplificado)
  performANOVA(combinations) {
    const conversionRates = combinations.map(c => c.conversionRate);
    const mean = conversionRates.reduce((sum, rate) => sum + rate, 0) / conversionRates.length;
    
    const betweenGroupVariance = combinations.reduce((sum, combo) => 
      sum + Math.pow(combo.conversionRate - mean, 2), 0
    );

    const withinGroupVariance = combinations.reduce((sum, combo) => {
      const variance = combo.conversionRate * (1 - combo.conversionRate / 100) / combo.users;
      return sum + variance;
    }, 0);

    const fStatistic = betweenGroupVariance / withinGroupVariance;

    return {
      f_statistic: fStatistic,
      between_group_variance: betweenGroupVariance,
      within_group_variance: withinGroupVariance,
      is_significant: fStatistic > 2.0 // Threshold simplificado
    };
  }

  // Análisis de efectos principales
  analyzeMainEffects(variables, results) {
    const mainEffects = {};

    Object.keys(variables).forEach(variableName => {
      const values = variables[variableName];
      const effects = {};

      values.forEach(value => {
        const combinationsWithValue = Object.values(results).filter(result => 
          result.combination[variableName] === value
        );

        const avgConversionRate = combinationsWithValue.reduce((sum, combo) => 
          sum + combo.conversionRate, 0
        ) / combinationsWithValue.length;

        effects[value] = avgConversionRate;
      });

      mainEffects[variableName] = effects;
    });

    return mainEffects;
  }

  // Generar recomendación MVT
  generateMVTRecommendation(bestCombination, allCombinations) {
    const avgConversionRate = allCombinations.reduce((sum, combo) => 
      sum + combo.conversionRate, 0
    ) / allCombinations.length;

    const improvement = bestCombination.conversionRate - avgConversionRate;

    if (improvement > 5) {
      return {
        action: 'implement_best_combination',
        reason: `Best combination shows ${improvement.toFixed(1)}% improvement over average`,
        confidence: 'high',
        best_combination: bestCombination.combination
      };
    } else {
      return {
        action: 'continue_testing',
        reason: 'No significant improvement detected',
        confidence: 'low'
      };
    }
  }
}

export default new MVTTestingSystem();
```

### **4. Statistical Significance y Sample Sizes**

#### **Calculadora de Sample Size**

```javascript
// Calculadora de tamaño de muestra
class SampleSizeCalculator {
  constructor() {
    this.zScores = {
      90: 1.645,
      95: 1.96,
      99: 2.576
    };
  }

  // Calcular tamaño de muestra para A/B test
  calculateSampleSize(baselineRate, minimumDetectableEffect, confidenceLevel = 95, power = 80) {
    const zAlpha = this.zScores[confidenceLevel];
    const zBeta = this.getZScoreForPower(power);
    
    const p1 = baselineRate / 100;
    const p2 = p1 + (minimumDetectableEffect / 100);
    const p = (p1 + p2) / 2;

    const numerator = Math.pow(zAlpha * Math.sqrt(2 * p * (1 - p)) + zBeta * Math.sqrt(p1 * (1 - p1) + p2 * (1 - p2)), 2);
    const denominator = Math.pow(p2 - p1, 2);

    const sampleSize = Math.ceil(numerator / denominator);
    
    return {
      per_variant: sampleSize,
      total: sampleSize * 2,
      baseline_rate: baselineRate,
      minimum_detectable_effect: minimumDetectableEffect,
      confidence_level: confidenceLevel,
      power: power,
      estimated_duration: this.estimateTestDuration(sampleSize * 2)
    };
  }

  // Obtener Z-score para poder estadístico
  getZScoreForPower(power) {
    const powerMap = {
      80: 0.84,
      85: 1.04,
      90: 1.28,
      95: 1.64
    };
    return powerMap[power] || 0.84;
  }

  // Estimar duración del test
  estimateTestDuration(totalSampleSize) {
    const dailyTraffic = 1000; // Asumir 1000 usuarios diarios
    const days = Math.ceil(totalSampleSize / dailyTraffic);
    
    return {
      days: days,
      weeks: Math.ceil(days / 7),
      daily_traffic: dailyTraffic
    };
  }

  // Calcular significancia estadística
  calculateStatisticalSignificance(controlUsers, controlConversions, treatmentUsers, treatmentConversions) {
    const p1 = controlConversions / controlUsers;
    const p2 = treatmentConversions / treatmentUsers;
    const p = (controlConversions + treatmentConversions) / (controlUsers + treatmentUsers);

    const se = Math.sqrt(p * (1 - p) * (1/controlUsers + 1/treatmentUsers));
    const z = (p2 - p1) / se;
    const pValue = this.calculatePValue(z);

    const confidenceInterval = this.calculateConfidenceInterval(p1, p2, se);

    return {
      z_score: z,
      p_value: pValue,
      is_significant: pValue < 0.05,
      confidence_interval: confidenceInterval,
      effect_size: p2 - p1,
      relative_improvement: ((p2 - p1) / p1) * 100
    };
  }

  // Calcular p-value
  calculatePValue(z) {
    const absZ = Math.abs(z);
    if (absZ > 2.58) return 0.01;
    if (absZ > 1.96) return 0.05;
    if (absZ > 1.65) return 0.10;
    return 0.20;
  }

  // Calcular intervalo de confianza
  calculateConfidenceInterval(p1, p2, se) {
    const margin = 1.96 * se;
    const difference = p2 - p1;
    return {
      lower: difference - margin,
      upper: difference + margin,
      margin: margin
    };
  }
}

export default new SampleSizeCalculator();
```

### **5. Testing de Hipótesis y Experimentación**

#### **Sistema de Testing de Hipótesis**

```javascript
// Sistema de testing de hipótesis
class HypothesisTestingSystem {
  constructor() {
    this.hypotheses = new Map();
    this.experiments = new Map();
    this.results = new Map();
  }

  // Crear hipótesis
  createHypothesis(hypothesisId, description, nullHypothesis, alternativeHypothesis, successMetrics) {
    this.hypotheses.set(hypothesisId, {
      id: hypothesisId,
      description: description,
      null_hypothesis: nullHypothesis,
      alternative_hypothesis: alternativeHypothesis,
      success_metrics: successMetrics,
      status: 'draft',
      created_at: Date.now()
    });
  }

  // Diseñar experimento para hipótesis
  designExperiment(hypothesisId, experimentType, variants, sampleSize, duration) {
    const hypothesis = this.hypotheses.get(hypothesisId);
    if (!hypothesis) return null;

    const experiment = {
      hypothesis_id: hypothesisId,
      type: experimentType, // 'ab_test', 'mvt', 'sequential'
      variants: variants,
      sample_size: sampleSize,
      duration: duration,
      status: 'designed',
      created_at: Date.now()
    };

    this.experiments.set(hypothesisId, experiment);
    hypothesis.status = 'experiment_designed';

    return experiment;
  }

  // Ejecutar experimento
  executeExperiment(hypothesisId, startDate, endDate) {
    const experiment = this.experiments.get(hypothesisId);
    const hypothesis = this.hypotheses.get(hypothesisId);
    
    if (!experiment || !hypothesis) return null;

    experiment.status = 'running';
    experiment.start_date = startDate;
    experiment.end_date = endDate;
    hypothesis.status = 'experiment_running';

    return experiment;
  }

  // Analizar resultados de hipótesis
  analyzeHypothesisResults(hypothesisId) {
    const hypothesis = this.hypotheses.get(hypothesisId);
    const experiment = this.experiments.get(hypothesisId);
    
    if (!hypothesis || !experiment) return null;

    // Obtener datos del experimento
    const experimentData = this.getExperimentData(hypothesisId);
    
    // Realizar test estadístico
    const statisticalTest = this.performStatisticalTest(experimentData, hypothesis);
    
    // Evaluar hipótesis
    const evaluation = this.evaluateHypothesis(hypothesis, statisticalTest);
    
    // Generar conclusiones
    const conclusions = this.generateConclusions(hypothesis, statisticalTest, evaluation);

    const analysis = {
      hypothesis_id: hypothesisId,
      hypothesis: hypothesis,
      experiment: experiment,
      statistical_test: statisticalTest,
      evaluation: evaluation,
      conclusions: conclusions,
      recommendation: this.generateRecommendation(evaluation, statisticalTest)
    };

    this.results.set(hypothesisId, analysis);
    hypothesis.status = 'completed';

    return analysis;
  }

  // Obtener datos del experimento
  getExperimentData(hypothesisId) {
    // En una implementación real, esto obtendría datos de la base de datos
    // Por simplicidad, retornamos datos simulados
    return {
      control: { users: 5000, conversions: 250 },
      treatment: { users: 5000, conversions: 300 }
    };
  }

  // Realizar test estadístico
  performStatisticalTest(data, hypothesis) {
    const { control, treatment } = data;
    
    const p1 = control.conversions / control.users;
    const p2 = treatment.conversions / treatment.users;
    const p = (control.conversions + treatment.conversions) / (control.users + treatment.users);

    const se = Math.sqrt(p * (1 - p) * (1/control.users + 1/treatment.users));
    const z = (p2 - p1) / se;
    const pValue = this.calculatePValue(z);

    return {
      test_type: 'two_proportion_z_test',
      z_score: z,
      p_value: pValue,
      effect_size: p2 - p1,
      confidence_interval: this.calculateConfidenceInterval(p1, p2, se),
      is_significant: pValue < 0.05
    };
  }

  // Evaluar hipótesis
  evaluateHypothesis(hypothesis, statisticalTest) {
    const { null_hypothesis, alternative_hypothesis } = hypothesis;
    const { p_value, is_significant } = statisticalTest;

    if (is_significant) {
      return {
        decision: 'reject_null',
        conclusion: 'Reject null hypothesis in favor of alternative hypothesis',
        confidence: p_value < 0.01 ? 'very_high' : 'high'
      };
    } else {
      return {
        decision: 'fail_to_reject_null',
        conclusion: 'Fail to reject null hypothesis',
        confidence: 'low'
      };
    }
  }

  // Generar conclusiones
  generateConclusions(hypothesis, statisticalTest, evaluation) {
    const conclusions = [];

    conclusions.push(`Hypothesis: ${hypothesis.description}`);
    conclusions.push(`Statistical Test: ${statisticalTest.test_type}`);
    conclusions.push(`P-value: ${statisticalTest.p_value.toFixed(4)}`);
    conclusions.push(`Decision: ${evaluation.decision}`);
    conclusions.push(`Conclusion: ${evaluation.conclusion}`);

    if (statisticalTest.is_significant) {
      conclusions.push(`Effect Size: ${(statisticalTest.effect_size * 100).toFixed(2)}%`);
      conclusions.push(`Confidence: ${evaluation.confidence}`);
    }

    return conclusions;
  }

  // Generar recomendación
  generateRecommendation(evaluation, statisticalTest) {
    if (evaluation.decision === 'reject_null') {
      return {
        action: 'implement_treatment',
        reason: 'Statistical evidence supports the alternative hypothesis',
        confidence: evaluation.confidence,
        effect_size: statisticalTest.effect_size
      };
    } else {
      return {
        action: 'continue_testing',
        reason: 'Insufficient evidence to reject null hypothesis',
        confidence: 'low'
      };
    }
  }

  // Calcular p-value
  calculatePValue(z) {
    const absZ = Math.abs(z);
    if (absZ > 2.58) return 0.01;
    if (absZ > 1.96) return 0.05;
    if (absZ > 1.65) return 0.10;
    return 0.20;
  }

  // Calcular intervalo de confianza
  calculateConfidenceInterval(p1, p2, se) {
    const margin = 1.96 * se;
    const difference = p2 - p1;
    return {
      lower: difference - margin,
      upper: difference + margin,
      margin: margin
    };
  }
}

export default new HypothesisTestingSystem();
```

---

## 🛠️ Implementación Práctica

### **Ejemplo: Sistema Completo de A/B Testing y MVT**

```javascript
// Sistema completo de testing avanzado
import { Platform } from 'react-native';

class AdvancedTestingSystem {
  constructor() {
    this.abTesting = new ABTestingSystem();
    this.mvtTesting = new MVTTestingSystem();
    this.sampleCalculator = new SampleSizeCalculator();
    this.hypothesisTesting = new HypothesisTestingSystem();
    this.analytics = new AnalyticsService();
    
    this.initialize();
  }

  async initialize() {
    // Configurar experimentos por defecto
    this.setupDefaultExperiments();
    
    // Configurar hipótesis
    this.setupDefaultHypotheses();
  }

  // Configurar experimentos por defecto
  setupDefaultExperiments() {
    // A/B Test: Checkout Button Color
    this.abTesting.createExperiment(
      'checkout_button_color',
      'Checkout Button Color Test',
      'Red buttons will increase conversions by 15%',
      {
        control: { name: 'Blue Button', color: '#007AFF' },
        treatment: { name: 'Red Button', color: '#FF3B30' }
      },
      {
        sample_size: 10000,
        confidence_level: 95,
        minimum_detectable_effect: 15
      }
    );

    // MVT: Landing Page Optimization
    this.mvtTesting.createMVTExperiment(
      'landing_page_mvt',
      'Landing Page MVT',
      'Combination of headline + CTA + image will increase conversions',
      {
        headline: ['Original', 'Benefit-focused', 'Urgency-driven'],
        cta: ['Get Started', 'Try Free', 'Sign Up Now'],
        image: ['Product', 'People', 'Abstract']
      },
      {
        total_sample_size: 27000,
        per_variant_sample: 1000,
        confidence_level: 95
      }
    );
  }

  // Configurar hipótesis por defecto
  setupDefaultHypotheses() {
    this.hypothesisTesting.createHypothesis(
      'checkout_optimization',
      'Red checkout buttons will increase conversion rate by at least 15%',
      'H0: Red buttons have no effect on conversion rate (p1 = p2)',
      'H1: Red buttons increase conversion rate (p2 > p1)',
      ['conversion_rate', 'revenue_per_user']
    );
  }

  // Obtener variante para usuario
  getVariantForUser(userId, experimentId, experimentType = 'ab') {
    if (experimentType === 'ab') {
      return this.abTesting.assignUserToVariant(userId, experimentId);
    } else if (experimentType === 'mvt') {
      return this.mvtTesting.assignUserToCombination(userId, experimentId);
    }
    return null;
  }

  // Trackear conversión
  trackConversion(userId, experimentId, experimentType, conversionValue = 0, properties = {}) {
    if (experimentType === 'ab') {
      this.abTesting.trackConversion(userId, experimentId, conversionValue, properties);
    } else if (experimentType === 'mvt') {
      this.mvtTesting.trackConversion(userId, experimentId, conversionValue, properties);
    }

    // Trackear en analytics general
    this.analytics.trackEvent('Experiment Conversion', {
      experiment_id: experimentId,
      experiment_type: experimentType,
      user_id: userId,
      conversion_value: conversionValue,
      ...properties
    });
  }

  // Analizar resultados completos
  getCompleteResults() {
    return {
      ab_tests: this.getABTestResults(),
      mvt_tests: this.getMVTResults(),
      hypotheses: this.getHypothesisResults(),
      recommendations: this.generateOverallRecommendations()
    };
  }

  // Obtener resultados de A/B tests
  getABTestResults() {
    const experiments = ['checkout_button_color'];
    return experiments.map(experimentId => 
      this.abTesting.analyzeResults(experimentId)
    ).filter(result => result !== null);
  }

  // Obtener resultados de MVT
  getMVTResults() {
    const experiments = ['landing_page_mvt'];
    return experiments.map(experimentId => 
      this.mvtTesting.analyzeMVTResults(experimentId)
    ).filter(result => result !== null);
  }

  // Obtener resultados de hipótesis
  getHypothesisResults() {
    const hypotheses = ['checkout_optimization'];
    return hypotheses.map(hypothesisId => 
      this.hypothesisTesting.analyzeHypothesisResults(hypothesisId)
    ).filter(result => result !== null);
  }

  // Generar recomendaciones generales
  generateOverallRecommendations() {
    const abResults = this.getABTestResults();
    const mvtResults = this.getMVTResults();
    const hypothesisResults = this.getHypothesisResults();

    const recommendations = [];

    // Recomendaciones de A/B tests
    abResults.forEach(result => {
      if (result.recommendation.action === 'implement_treatment') {
        recommendations.push({
          type: 'ab_test',
          experiment: result.experiment_name,
          action: 'implement',
          reason: result.recommendation.reason,
          confidence: result.recommendation.confidence
        });
      }
    });

    // Recomendaciones de MVT
    mvtResults.forEach(result => {
      if (result.recommendation.action === 'implement_best_combination') {
        recommendations.push({
          type: 'mvt',
          experiment: result.experiment_name,
          action: 'implement',
          reason: result.recommendation.reason,
          confidence: result.recommendation.confidence,
          best_combination: result.recommendation.best_combination
        });
      }
    });

    return recommendations;
  }

  // Calcular tamaño de muestra para nuevo experimento
  calculateSampleSizeForExperiment(baselineRate, minimumDetectableEffect, confidenceLevel = 95, power = 80) {
    return this.sampleCalculator.calculateSampleSize(
      baselineRate,
      minimumDetectableEffect,
      confidenceLevel,
      power
    );
  }

  // Verificar significancia estadística
  checkStatisticalSignificance(controlUsers, controlConversions, treatmentUsers, treatmentConversions) {
    return this.sampleCalculator.calculateStatisticalSignificance(
      controlUsers,
      controlConversions,
      treatmentUsers,
      treatmentConversions
    );
  }
}

export default new AdvancedTestingSystem();
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Implementar A/B Test**

```javascript
// Implementar A/B test completo
const exercise1 = {
  task: 'Crear A/B test para optimización de conversión',
  steps: [
    '1. Definir hipótesis y métricas',
    '2. Calcular tamaño de muestra',
    '3. Implementar asignación de variantes',
    '4. Trackear conversiones',
    '5. Analizar resultados estadísticos'
  ],
  expectedResult: 'A/B test funcional con análisis estadístico'
};
```

### **Ejercicio 2: Multivariate Testing**

```javascript
// Implementar MVT
const exercise2 = {
  task: 'Crear experimento MVT para landing page',
  steps: [
    '1. Definir variables y combinaciones',
    '2. Implementar asignación de combinaciones',
    '3. Trackear conversiones por combinación',
    '4. Realizar análisis de varianza',
    '5. Identificar mejor combinación'
  ],
  expectedResult: 'MVT funcional con análisis de efectos'
};
```

### **Ejercicio 3: Testing de Hipótesis**

```javascript
// Implementar testing de hipótesis
const exercise3 = {
  task: 'Crear sistema de testing de hipótesis',
  steps: [
    '1. Formular hipótesis nula y alternativa',
    '2. Diseñar experimento',
    '3. Ejecutar test estadístico',
    '4. Evaluar resultados',
    '5. Generar conclusiones'
  ],
  expectedResult: 'Sistema de testing de hipótesis funcional'
};
```

---

## 📊 Métricas y KPIs

### **Métricas de A/B Testing**

```javascript
const ABTestingMetrics = {
  statistical: {
    p_value: 'Valor p del test',
    confidence_interval: 'Intervalo de confianza',
    statistical_power: 'Poder estadístico',
    effect_size: 'Tamaño del efecto'
  },
  business: {
    conversion_rate: 'Tasa de conversión',
    revenue_impact: 'Impacto en ingresos',
    user_engagement: 'Engagement del usuario',
    retention_rate: 'Tasa de retención'
  }
};
```

### **Métricas de MVT**

```javascript
const MVTMetrics = {
  statistical: {
    f_statistic: 'Estadístico F',
    anova_p_value: 'Valor p del ANOVA',
    main_effects: 'Efectos principales',
    interaction_effects: 'Efectos de interacción'
  },
  optimization: {
    best_combination: 'Mejor combinación',
    improvement_over_baseline: 'Mejora sobre baseline',
    confidence_level: 'Nivel de confianza',
    sample_size_per_variant: 'Tamaño de muestra por variante'
  }
};
```

---

## 🔧 Herramientas y Recursos

### **Herramientas de A/B Testing**

- **Optimizely**: [optimizely.com](https://www.optimizely.com/)
- **VWO**: [vwo.com](https://vwo.com/)
- **Google Optimize**: [optimize.google.com](https://optimize.google.com/)
- **Apptimize**: [apptimize.com](https://apptimize.com/)

### **Herramientas de MVT**

- **Optimizely MVT**: [optimizely.com/multivariate-testing](https://www.optimizely.com/multivariate-testing)
- **VWO MVT**: [vwo.com/multivariate-testing](https://vwo.com/multivariate-testing)
- **Adobe Target**: [adobe.com/target](https://www.adobe.com/target)

### **Recursos de Aprendizaje**

- **A/B Testing Guide**: [optimizely.com/optimization-glossary](https://www.optimizely.com/optimization-glossary)
- **Statistical Significance**: [vwo.com/blog/statistical-significance](https://vwo.com/blog/statistical-significance)
- **MVT Best Practices**: [optimizely.com/learn/multivariate-testing](https://www.optimizely.com/learn/multivariate-testing)

---

## 🚀 Próximos Pasos

### **Lo que sigue**

1. **Clase 4**: Dashboards de BI en Tiempo Real
2. **Clase 5**: Machine Learning para Analytics

### **Preparación**

- Configurar herramientas de A/B testing
- Revisar conceptos de estadística
- Preparar datos para experimentos
- Configurar métricas de conversión

---

**🎯 Objetivo de la Clase**: Dominar A/B testing avanzado y MVT para optimizar aplicaciones React Native basándose en evidencia estadística sólida.

**💡 Consejo**: El testing estadístico es la base de la optimización basada en datos. Siempre asegúrate de tener suficiente poder estadístico y significancia antes de tomar decisiones.

---

**🚀 ¡Comienza tu viaje hacia la maestría en testing estadístico!**
