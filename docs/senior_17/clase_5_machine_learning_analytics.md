# 🤖 Clase 5: Machine Learning para Analytics

## 📋 Objetivos de la Clase

- Implementar analytics predictivos con ML
- Crear sistemas de segmentación de usuarios
- Desarrollar modelos de predicción de churn
- Construir motores de recomendación
- Implementar detección de anomalías

## 🎯 Contenido Principal

### 1. Analytics Predictivos

#### Fundamentos de ML para Analytics
```javascript
// Configuración de TensorFlow.js para React Native
import * as tf from '@tensorflow/tfjs';
import '@tensorflow/tfjs-react-native';

// Inicialización de TensorFlow
const initializeTensorFlow = async () => {
  await tf.ready();
  console.log('TensorFlow.js está listo');
};
```

#### Modelos de Predicción
```javascript
// Modelo de predicción de conversión
class ConversionPredictionModel {
  constructor() {
    this.model = null;
  }

  async loadModel() {
    this.model = await tf.loadLayersModel('model/conversion-prediction.json');
  }

  async predictConversion(userFeatures) {
    const input = tf.tensor2d([userFeatures]);
    const prediction = this.model.predict(input);
    const probability = await prediction.data();
    return probability[0];
  }
}
```

### 2. Segmentación de Usuarios

#### Clustering con K-Means
```javascript
// Implementación de K-Means para segmentación
class UserSegmentation {
  constructor(k = 5) {
    this.k = k;
    this.centroids = [];
  }

  // Calcular distancia euclidiana
  euclideanDistance(point1, point2) {
    return Math.sqrt(
      point1.reduce((sum, val, i) => sum + Math.pow(val - point2[i], 2), 0)
    );
  }

  // Asignar usuarios a clusters
  assignToClusters(users) {
    return users.map(user => {
      let minDistance = Infinity;
      let cluster = 0;
      
      this.centroids.forEach((centroid, index) => {
        const distance = this.euclideanDistance(user.features, centroid);
        if (distance < minDistance) {
          minDistance = distance;
          cluster = index;
        }
      });
      
      return { ...user, cluster };
    });
  }
}
```

#### Análisis de Cohorts Avanzado
```javascript
// Análisis de cohorts con ML
class CohortAnalysis {
  async analyzeUserBehavior(users) {
    const features = users.map(user => [
      user.sessionDuration,
      user.pageViews,
      user.conversionRate,
      user.retentionDays
    ]);

    // Aplicar clustering
    const segmentation = new UserSegmentation(4);
    const segmentedUsers = segmentation.assignToClusters(users);

    return this.generateCohortInsights(segmentedUsers);
  }

  generateCohortInsights(segmentedUsers) {
    return segmentedUsers.reduce((insights, user) => {
      const cohort = `Cohort_${user.cluster}`;
      if (!insights[cohort]) {
        insights[cohort] = {
          size: 0,
          avgLTV: 0,
          churnRate: 0,
          characteristics: []
        };
      }
      
      insights[cohort].size++;
      insights[cohort].avgLTV += user.lifetimeValue;
      insights[cohort].churnRate += user.churnProbability;
      
      return insights;
    }, {});
  }
}
```

### 3. Predicción de Churn

#### Modelo de Churn Prediction
```javascript
// Modelo de predicción de churn
class ChurnPredictionModel {
  constructor() {
    this.features = [
      'sessionFrequency',
      'lastLoginDays',
      'supportTickets',
      'paymentFailures',
      'featureUsage'
    ];
  }

  // Extraer características del usuario
  extractFeatures(user) {
    return this.features.map(feature => {
      switch(feature) {
        case 'sessionFrequency':
          return user.sessionsLast30Days / 30;
        case 'lastLoginDays':
          return this.daysSinceLastLogin(user.lastLogin);
        case 'supportTickets':
          return user.supportTicketsLast30Days;
        case 'paymentFailures':
          return user.paymentFailuresLast30Days;
        case 'featureUsage':
          return user.featuresUsed / user.totalFeatures;
        default:
          return 0;
      }
    });
  }

  // Predecir probabilidad de churn
  async predictChurn(user) {
    const features = this.extractFeatures(user);
    const model = await this.loadChurnModel();
    
    const input = tf.tensor2d([features]);
    const prediction = model.predict(input);
    const churnProbability = await prediction.data();
    
    return {
      churnProbability: churnProbability[0],
      riskLevel: this.categorizeRisk(churnProbability[0]),
      recommendations: this.generateRecommendations(features)
    };
  }

  categorizeRisk(probability) {
    if (probability > 0.7) return 'HIGH';
    if (probability > 0.4) return 'MEDIUM';
    return 'LOW';
  }
}
```

### 4. Motores de Recomendación

#### Sistema de Recomendaciones Colaborativas
```javascript
// Motor de recomendaciones
class RecommendationEngine {
  constructor() {
    this.userItemMatrix = new Map();
    this.itemSimilarities = new Map();
  }

  // Calcular similitud coseno
  cosineSimilarity(vectorA, vectorB) {
    const dotProduct = vectorA.reduce((sum, a, i) => sum + a * vectorB[i], 0);
    const magnitudeA = Math.sqrt(vectorA.reduce((sum, a) => sum + a * a, 0));
    const magnitudeB = Math.sqrt(vectorB.reduce((sum, b) => sum + b * b, 0));
    
    return dotProduct / (magnitudeA * magnitudeB);
  }

  // Generar recomendaciones
  async generateRecommendations(userId, limit = 10) {
    const userRatings = this.userItemMatrix.get(userId) || new Map();
    const recommendations = [];

    // Encontrar usuarios similares
    const similarUsers = this.findSimilarUsers(userId);
    
    // Calcular scores de recomendación
    for (const [itemId, itemRatings] of this.itemSimilarities) {
      if (!userRatings.has(itemId)) {
        let score = 0;
        let weightSum = 0;

        similarUsers.forEach(({ userId: similarUserId, similarity }) => {
          const similarUserRatings = this.userItemMatrix.get(similarUserId);
          if (similarUserRatings && similarUserRatings.has(itemId)) {
            score += similarity * similarUserRatings.get(itemId);
            weightSum += Math.abs(similarity);
          }
        });

        if (weightSum > 0) {
          recommendations.push({
            itemId,
            score: score / weightSum,
            confidence: weightSum
          });
        }
      }
    }

    return recommendations
      .sort((a, b) => b.score - a.score)
      .slice(0, limit);
  }
}
```

### 5. Detección de Anomalías

#### Sistema de Detección de Fraude
```javascript
// Detección de anomalías en tiempo real
class AnomalyDetection {
  constructor() {
    this.normalPatterns = new Map();
    this.threshold = 0.8;
  }

  // Detectar anomalías en transacciones
  async detectAnomalies(transaction) {
    const features = this.extractTransactionFeatures(transaction);
    const anomalyScore = await this.calculateAnomalyScore(features);
    
    return {
      isAnomaly: anomalyScore > this.threshold,
      score: anomalyScore,
      riskFactors: this.identifyRiskFactors(features),
      recommendation: this.getRecommendation(anomalyScore)
    };
  }

  extractTransactionFeatures(transaction) {
    return [
      transaction.amount,
      transaction.hourOfDay,
      transaction.dayOfWeek,
      transaction.locationDistance,
      transaction.deviceFingerprint,
      transaction.userBehaviorScore
    ];
  }

  // Algoritmo de detección de outliers
  calculateAnomalyScore(features) {
    const mean = this.calculateMean(features);
    const stdDev = this.calculateStandardDeviation(features, mean);
    
    // Z-score para cada característica
    const zScores = features.map(feature => 
      Math.abs((feature - mean) / stdDev)
    );
    
    // Score de anomalía promedio
    return zScores.reduce((sum, score) => sum + score, 0) / zScores.length;
  }
}
```

## 🛠️ Implementación Práctica

### Dashboard de ML Analytics
```javascript
// Componente principal del dashboard
const MLAnalyticsDashboard = () => {
  const [predictions, setPredictions] = useState({});
  const [recommendations, setRecommendations] = useState([]);
  const [anomalies, setAnomalies] = useState([]);

  useEffect(() => {
    loadMLInsights();
  }, []);

  const loadMLInsights = async () => {
    // Cargar predicciones de churn
    const churnPredictions = await churnModel.predictChurn(currentUser);
    
    // Generar recomendaciones
    const userRecommendations = await recommendationEngine
      .generateRecommendations(currentUser.id);
    
    // Detectar anomalías
    const recentTransactions = await getRecentTransactions();
    const anomalyResults = await Promise.all(
      recentTransactions.map(tx => anomalyDetection.detectAnomalies(tx))
    );

    setPredictions(churnPredictions);
    setRecommendations(userRecommendations);
    setAnomalies(anomalyResults.filter(result => result.isAnomaly));
  };

  return (
    <ScrollView style={styles.container}>
      <ChurnPredictionCard prediction={predictions} />
      <RecommendationCard recommendations={recommendations} />
      <AnomalyDetectionCard anomalies={anomalies} />
      <UserSegmentationCard />
    </ScrollView>
  );
};
```

## 📊 Métricas y KPIs

### KPIs de ML Analytics
- **Precisión de Predicciones**: 85%+
- **Tiempo de Respuesta**: <200ms
- **Cobertura de Recomendaciones**: 90%+
- **Detección de Anomalías**: 95%+ accuracy
- **ROI de ML**: Incremento en conversión 15%+

## 🔧 Herramientas y Librerías

### ML Libraries
- **TensorFlow.js**: Modelos de ML en JavaScript
- **ML Kit**: ML nativo para móviles
- **Core ML**: ML para iOS
- **Azure ML**: Servicios de ML en la nube
- **AWS SageMaker**: Plataforma de ML

### Analytics Tools
- **Mixpanel**: Event tracking avanzado
- **Amplitude**: Behavioral analytics
- **Segment**: Data pipeline
- **Optimizely**: Experimentación
- **Tableau**: Visualización de datos

## 🎯 Proyecto Integrador

### Sistema de ML Analytics Completo
Desarrollar una aplicación que integre:

1. **Predicción de Churn** con notificaciones proactivas
2. **Motor de Recomendaciones** personalizado
3. **Detección de Anomalías** en tiempo real
4. **Segmentación Automática** de usuarios
5. **Dashboard de Insights** con visualizaciones

## 📚 Recursos Adicionales

### Documentación
- [TensorFlow.js Documentation](https://www.tensorflow.org/js)
- [ML Kit for Firebase](https://firebase.google.com/docs/ml-kit)
- [Core ML Documentation](https://developer.apple.com/documentation/coreml)

### Cursos Recomendados
- Machine Learning Fundamentals
- Advanced Analytics with Python
- Deep Learning for Mobile
- Data Science for Business

## ✅ Evaluación

### Criterios de Evaluación
- Implementación correcta de modelos ML
- Precisión de predicciones >80%
- Performance de recomendaciones
- Detección efectiva de anomalías
- Dashboard funcional y responsive

---

**Siguiente**: [Módulo 27: Microservicios y Backend para Móvil](../senior_18/README.md)
**Anterior**: [Clase 4: Dashboards de BI en Tiempo Real](./clase_4_dashboards_bi_tiempo_real.md)
