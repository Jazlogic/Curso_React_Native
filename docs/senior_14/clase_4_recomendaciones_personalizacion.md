# 🎯 Clase 4: Recomendaciones y Personalización

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar sistemas avanzados de recomendaciones y personalización en React Native, incluyendo filtrado colaborativo, filtrado basado en contenido, machine learning para personalización y A/B testing.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** sistemas de recomendación colaborativos
2. **Crear** filtrado basado en contenido y comportamiento
3. **Desarrollar** algoritmos de personalización
4. **Implementar** A/B testing para recomendaciones
5. **Crear** analytics y métricas de engagement

---

## 🛠️ Configuración del Sistema

### **Instalación de Dependencias**

```bash
# Instalar librerías de ML y recomendaciones
npm install @tensorflow/tfjs @tensorflow/tfjs-node
npm install ml-matrix ml-kmeans ml-knn
npm install react-native-sqlite-storage
npm install @react-native-async-storage/async-storage

# Instalar dependencias de analytics
npm install react-native-analytics
npm install react-native-device-info
```

### **Configuración de Base de Datos**

```javascript
// DatabaseConfig.js
import SQLite from 'react-native-sqlite-storage';

class DatabaseConfig {
  constructor() {
    this.db = null;
  }

  async initializeDatabase() {
    try {
      this.db = await SQLite.openDatabase({
        name: 'recommendations.db',
        location: 'default',
      });

      await this.createTables();
      console.log('Base de datos inicializada');
    } catch (error) {
      console.error('Error inicializando base de datos:', error);
    }
  }

  async createTables() {
    const queries = [
      // Tabla de usuarios
      `CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id TEXT UNIQUE,
        preferences TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      )`,
      
      // Tabla de items
      `CREATE TABLE IF NOT EXISTS items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        item_id TEXT UNIQUE,
        category TEXT,
        features TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      )`,
      
      // Tabla de interacciones
      `CREATE TABLE IF NOT EXISTS interactions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id TEXT,
        item_id TEXT,
        interaction_type TEXT,
        rating REAL,
        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (user_id) REFERENCES users (user_id),
        FOREIGN KEY (item_id) REFERENCES items (item_id)
      )`,
      
      // Tabla de recomendaciones
      `CREATE TABLE IF NOT EXISTS recommendations (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id TEXT,
        item_id TEXT,
        score REAL,
        algorithm TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      )`
    ];

    for (const query of queries) {
      await this.db.executeSql(query);
    }
  }

  getDatabase() {
    return this.db;
  }
}

export default DatabaseConfig;
```

---

## 🤝 Sistemas de Recomendación Colaborativos

### **Implementación de Filtrado Colaborativo**

```javascript
// CollaborativeFiltering.js
import { Matrix } from 'ml-matrix';
import { KMeans } from 'ml-kmeans';

class CollaborativeFiltering {
  constructor() {
    this.userItemMatrix = null;
    this.userSimilarities = null;
    this.itemSimilarities = null;
    this.db = null;
  }

  async initialize(database) {
    this.db = database;
    await this.buildUserItemMatrix();
  }

  async buildUserItemMatrix() {
    try {
      // Obtener todas las interacciones
      const interactions = await this.getInteractions();
      
      // Obtener usuarios e items únicos
      const users = [...new Set(interactions.map(i => i.user_id))];
      const items = [...new Set(interactions.map(i => i.item_id))];
      
      // Crear matriz usuario-item
      this.userItemMatrix = new Matrix(users.length, items.length);
      
      // Llenar matriz con ratings
      interactions.forEach(interaction => {
        const userIndex = users.indexOf(interaction.user_id);
        const itemIndex = items.indexOf(interaction.item_id);
        this.userItemMatrix.set(userIndex, itemIndex, interaction.rating);
      });
      
      this.users = users;
      this.items = items;
      
      console.log('Matriz usuario-item construida');
    } catch (error) {
      console.error('Error construyendo matriz:', error);
    }
  }

  async getInteractions() {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM interactions',
          [],
          (tx, results) => {
            const interactions = [];
            for (let i = 0; i < results.rows.length; i++) {
              interactions.push(results.rows.item(i));
            }
            resolve(interactions);
          },
          (tx, error) => reject(error)
        );
      });
    });
  }

  calculateUserSimilarities() {
    if (!this.userItemMatrix) {
      throw new Error('Matriz usuario-item no construida');
    }

    const numUsers = this.userItemMatrix.rows;
    this.userSimilarities = new Matrix(numUsers, numUsers);

    for (let i = 0; i < numUsers; i++) {
      for (let j = 0; j < numUsers; j++) {
        if (i === j) {
          this.userSimilarities.set(i, j, 1);
        } else {
          const similarity = this.cosineSimilarity(
            this.userItemMatrix.getRow(i),
            this.userItemMatrix.getRow(j)
          );
          this.userSimilarities.set(i, j, similarity);
        }
      }
    }
  }

  calculateItemSimilarities() {
    if (!this.userItemMatrix) {
      throw new Error('Matriz usuario-item no construida');
    }

    const numItems = this.userItemMatrix.columns;
    this.itemSimilarities = new Matrix(numItems, numItems);

    for (let i = 0; i < numItems; i++) {
      for (let j = 0; j < numItems; j++) {
        if (i === j) {
          this.itemSimilarities.set(i, j, 1);
        } else {
          const similarity = this.cosineSimilarity(
            this.userItemMatrix.getColumn(i),
            this.userItemMatrix.getColumn(j)
          );
          this.itemSimilarities.set(i, j, similarity);
        }
      }
    }
  }

  cosineSimilarity(vectorA, vectorB) {
    let dotProduct = 0;
    let normA = 0;
    let normB = 0;

    for (let i = 0; i < vectorA.length; i++) {
      dotProduct += vectorA[i] * vectorB[i];
      normA += vectorA[i] * vectorA[i];
      normB += vectorB[i] * vectorB[i];
    }

    if (normA === 0 || normB === 0) return 0;
    
    return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
  }

  async getUserBasedRecommendations(userId, numRecommendations = 10) {
    try {
      const userIndex = this.users.indexOf(userId);
      if (userIndex === -1) {
        throw new Error('Usuario no encontrado');
      }

      // Calcular similitudes si no están calculadas
      if (!this.userSimilarities) {
        this.calculateUserSimilarities();
      }

      // Obtener usuarios similares
      const userSimilarities = this.userSimilarities.getRow(userIndex);
      const similarUsers = this.getTopSimilarUsers(userSimilarities, 10);

      // Calcular recomendaciones
      const recommendations = [];
      const userRatings = this.userItemMatrix.getRow(userIndex);

      for (let itemIndex = 0; itemIndex < this.items.length; itemIndex++) {
        if (userRatings[itemIndex] === 0) { // Item no calificado
          let weightedSum = 0;
          let similaritySum = 0;

          similarUsers.forEach(({ userIndex: similarUserIndex, similarity }) => {
            const similarUserRating = this.userItemMatrix.get(similarUserIndex, itemIndex);
            if (similarUserRating > 0) {
              weightedSum += similarUserRating * similarity;
              similaritySum += Math.abs(similarity);
            }
          });

          if (similaritySum > 0) {
            const predictedRating = weightedSum / similaritySum;
            recommendations.push({
              itemId: this.items[itemIndex],
              score: predictedRating,
              algorithm: 'user_based'
            });
          }
        }
      }

      // Ordenar por score y retornar top N
      return recommendations
        .sort((a, b) => b.score - a.score)
        .slice(0, numRecommendations);

    } catch (error) {
      console.error('Error generando recomendaciones basadas en usuario:', error);
      throw error;
    }
  }

  async getItemBasedRecommendations(userId, numRecommendations = 10) {
    try {
      const userIndex = this.users.indexOf(userId);
      if (userIndex === -1) {
        throw new Error('Usuario no encontrado');
      }

      // Calcular similitudes si no están calculadas
      if (!this.itemSimilarities) {
        this.calculateItemSimilarities();
      }

      const userRatings = this.userItemMatrix.getRow(userIndex);
      const recommendations = [];

      for (let itemIndex = 0; itemIndex < this.items.length; itemIndex++) {
        if (userRatings[itemIndex] === 0) { // Item no calificado
          let weightedSum = 0;
          let similaritySum = 0;

          for (let ratedItemIndex = 0; ratedItemIndex < this.items.length; ratedItemIndex++) {
            if (userRatings[ratedItemIndex] > 0) {
              const similarity = this.itemSimilarities.get(itemIndex, ratedItemIndex);
              weightedSum += userRatings[ratedItemIndex] * similarity;
              similaritySum += Math.abs(similarity);
            }
          }

          if (similaritySum > 0) {
            const predictedRating = weightedSum / similaritySum;
            recommendations.push({
              itemId: this.items[itemIndex],
              score: predictedRating,
              algorithm: 'item_based'
            });
          }
        }
      }

      return recommendations
        .sort((a, b) => b.score - a.score)
        .slice(0, numRecommendations);

    } catch (error) {
      console.error('Error generando recomendaciones basadas en items:', error);
      throw error;
    }
  }

  getTopSimilarUsers(userSimilarities, topK) {
    const similarities = [];
    for (let i = 0; i < userSimilarities.length; i++) {
      similarities.push({
        userIndex: i,
        similarity: userSimilarities[i]
      });
    }

    return similarities
      .sort((a, b) => b.similarity - a.similarity)
      .slice(1, topK + 1); // Excluir el usuario mismo
  }
}

export default CollaborativeFiltering;
```

---

## 📊 Filtrado Basado en Contenido

### **Implementación de Content-Based Filtering**

```javascript
// ContentBasedFiltering.js
import { KMeans } from 'ml-kmeans';
import { KNN } from 'ml-knn';

class ContentBasedFiltering {
  constructor() {
    this.itemFeatures = new Map();
    this.userProfiles = new Map();
    this.db = null;
  }

  async initialize(database) {
    this.db = database;
    await this.buildItemFeatures();
    await this.buildUserProfiles();
  }

  async buildItemFeatures() {
    try {
      const items = await this.getItems();
      
      for (const item of items) {
        const features = await this.extractItemFeatures(item);
        this.itemFeatures.set(item.item_id, features);
      }
      
      console.log('Características de items construidas');
    } catch (error) {
      console.error('Error construyendo características de items:', error);
    }
  }

  async buildUserProfiles() {
    try {
      const users = await this.getUsers();
      
      for (const user of users) {
        const profile = await this.buildUserProfile(user.user_id);
        this.userProfiles.set(user.user_id, profile);
      }
      
      console.log('Perfiles de usuarios construidos');
    } catch (error) {
      console.error('Error construyendo perfiles de usuarios:', error);
    }
  }

  async extractItemFeatures(item) {
    try {
      const features = {
        category: item.category,
        price: item.price || 0,
        rating: item.average_rating || 0,
        popularity: item.popularity || 0,
        tags: item.tags ? JSON.parse(item.tags) : [],
        created_at: new Date(item.created_at).getTime()
      };

      // Agregar características adicionales basadas en el contenido
      if (item.description) {
        features.description_length = item.description.length;
        features.has_description = 1;
      } else {
        features.description_length = 0;
        features.has_description = 0;
      }

      return features;
    } catch (error) {
      console.error('Error extrayendo características del item:', error);
      return {};
    }
  }

  async buildUserProfile(userId) {
    try {
      const interactions = await this.getUserInteractions(userId);
      const profile = {
        preferred_categories: new Map(),
        average_rating: 0,
        total_interactions: interactions.length,
        last_activity: 0
      };

      if (interactions.length === 0) {
        return profile;
      }

      let totalRating = 0;
      let ratedInteractions = 0;

      for (const interaction of interactions) {
        const item = await this.getItem(interaction.item_id);
        if (item) {
          // Actualizar categorías preferidas
          const category = item.category;
          const currentCount = profile.preferred_categories.get(category) || 0;
          profile.preferred_categories.set(category, currentCount + 1);

          // Calcular rating promedio
          if (interaction.rating > 0) {
            totalRating += interaction.rating;
            ratedInteractions++;
          }

          // Actualizar última actividad
          const interactionTime = new Date(interaction.timestamp).getTime();
          if (interactionTime > profile.last_activity) {
            profile.last_activity = interactionTime;
          }
        }
      }

      profile.average_rating = ratedInteractions > 0 ? totalRating / ratedInteractions : 0;

      return profile;
    } catch (error) {
      console.error('Error construyendo perfil de usuario:', error);
      return {};
    }
  }

  async getContentBasedRecommendations(userId, numRecommendations = 10) {
    try {
      const userProfile = this.userProfiles.get(userId);
      if (!userProfile) {
        throw new Error('Perfil de usuario no encontrado');
      }

      const recommendations = [];
      const userInteractions = await this.getUserInteractions(userId);
      const interactedItemIds = new Set(userInteractions.map(i => i.item_id));

      for (const [itemId, features] of this.itemFeatures) {
        if (!interactedItemIds.has(itemId)) {
          const score = this.calculateContentSimilarity(userProfile, features);
          recommendations.push({
            itemId,
            score,
            algorithm: 'content_based'
          });
        }
      }

      return recommendations
        .sort((a, b) => b.score - a.score)
        .slice(0, numRecommendations);

    } catch (error) {
      console.error('Error generando recomendaciones basadas en contenido:', error);
      throw error;
    }
  }

  calculateContentSimilarity(userProfile, itemFeatures) {
    let score = 0;
    let totalWeight = 0;

    // Similitud basada en categorías preferidas
    const category = itemFeatures.category;
    const categoryPreference = userProfile.preferred_categories.get(category) || 0;
    const categoryWeight = 0.4;
    score += (categoryPreference / userProfile.total_interactions) * categoryWeight;
    totalWeight += categoryWeight;

    // Similitud basada en rating promedio
    const ratingSimilarity = 1 - Math.abs(userProfile.average_rating - itemFeatures.rating) / 5;
    const ratingWeight = 0.3;
    score += ratingSimilarity * ratingWeight;
    totalWeight += ratingWeight;

    // Similitud basada en popularidad
    const popularityScore = Math.min(itemFeatures.popularity / 100, 1);
    const popularityWeight = 0.2;
    score += popularityScore * popularityWeight;
    totalWeight += popularityWeight;

    // Similitud basada en recencia
    const now = Date.now();
    const itemAge = now - itemFeatures.created_at;
    const recencyScore = Math.max(0, 1 - (itemAge / (365 * 24 * 60 * 60 * 1000))); // 1 año
    const recencyWeight = 0.1;
    score += recencyScore * recencyWeight;
    totalWeight += recencyWeight;

    return totalWeight > 0 ? score / totalWeight : 0;
  }

  async getItems() {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM items',
          [],
          (tx, results) => {
            const items = [];
            for (let i = 0; i < results.rows.length; i++) {
              items.push(results.rows.item(i));
            }
            resolve(items);
          },
          (tx, error) => reject(error)
        );
      });
    });
  }

  async getUsers() {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM users',
          [],
          (tx, results) => {
            const users = [];
            for (let i = 0; i < results.rows.length; i++) {
              users.push(results.rows.item(i));
            }
            resolve(users);
          },
          (tx, error) => reject(error)
        );
      });
    });
  }

  async getUserInteractions(userId) {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM interactions WHERE user_id = ?',
          [userId],
          (tx, results) => {
            const interactions = [];
            for (let i = 0; i < results.rows.length; i++) {
              interactions.push(results.rows.item(i));
            }
            resolve(interactions);
          },
          (tx, error) => reject(error)
        );
      });
    });
  }

  async getItem(itemId) {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM items WHERE item_id = ?',
          [itemId],
          (tx, results) => {
            if (results.rows.length > 0) {
              resolve(results.rows.item(0));
            } else {
              resolve(null);
            }
          },
          (tx, error) => reject(error)
        );
      });
    });
  }
}

export default ContentBasedFiltering;
```

---

## 🎯 Sistema de Personalización Avanzado

### **Implementación de Personalización**

```javascript
// PersonalizationEngine.js
import CollaborativeFiltering from './CollaborativeFiltering';
import ContentBasedFiltering from './ContentBasedFiltering';

class PersonalizationEngine {
  constructor() {
    this.collaborativeFiltering = new CollaborativeFiltering();
    this.contentBasedFiltering = new ContentBasedFiltering();
    this.db = null;
    this.userPreferences = new Map();
  }

  async initialize(database) {
    this.db = database;
    await this.collaborativeFiltering.initialize(database);
    await this.contentBasedFiltering.initialize(database);
    await this.loadUserPreferences();
  }

  async loadUserPreferences() {
    try {
      const users = await this.getUsers();
      
      for (const user of users) {
        const preferences = user.preferences ? JSON.parse(user.preferences) : {};
        this.userPreferences.set(user.user_id, preferences);
      }
      
      console.log('Preferencias de usuarios cargadas');
    } catch (error) {
      console.error('Error cargando preferencias de usuarios:', error);
    }
  }

  async getPersonalizedRecommendations(userId, options = {}) {
    try {
      const {
        numRecommendations = 10,
        algorithms = ['collaborative', 'content_based'],
        weights = { collaborative: 0.6, content_based: 0.4 }
      } = options;

      const recommendations = [];

      // Obtener recomendaciones de cada algoritmo
      if (algorithms.includes('collaborative')) {
        const collaborativeRecs = await this.collaborativeFiltering.getUserBasedRecommendations(
          userId, numRecommendations
        );
        recommendations.push(...collaborativeRecs.map(rec => ({
          ...rec,
          weight: weights.collaborative
        })));
      }

      if (algorithms.includes('content_based')) {
        const contentRecs = await this.contentBasedFiltering.getContentBasedRecommendations(
          userId, numRecommendations
        );
        recommendations.push(...contentRecs.map(rec => ({
          ...rec,
          weight: weights.content_based
        })));
      }

      // Combinar y rankear recomendaciones
      const combinedRecommendations = this.combineRecommendations(recommendations);
      
      // Aplicar filtros personalizados
      const filteredRecommendations = await this.applyPersonalFilters(
        userId, combinedRecommendations
      );

      // Guardar recomendaciones
      await this.saveRecommendations(userId, filteredRecommendations);

      return filteredRecommendations.slice(0, numRecommendations);

    } catch (error) {
      console.error('Error generando recomendaciones personalizadas:', error);
      throw error;
    }
  }

  combineRecommendations(recommendations) {
    const itemScores = new Map();

    recommendations.forEach(rec => {
      const itemId = rec.itemId;
      const currentScore = itemScores.get(itemId) || 0;
      const newScore = currentScore + (rec.score * rec.weight);
      itemScores.set(itemId, newScore);
    });

    return Array.from(itemScores.entries())
      .map(([itemId, score]) => ({
        itemId,
        score,
        algorithm: 'hybrid'
      }))
      .sort((a, b) => b.score - a.score);
  }

  async applyPersonalFilters(userId, recommendations) {
    try {
      const userPrefs = this.userPreferences.get(userId) || {};
      const filteredRecommendations = [];

      for (const rec of recommendations) {
        let shouldInclude = true;

        // Filtro por categorías preferidas
        if (userPrefs.preferred_categories && userPrefs.preferred_categories.length > 0) {
          const item = await this.getItem(rec.itemId);
          if (item && !userPrefs.preferred_categories.includes(item.category)) {
            shouldInclude = false;
          }
        }

        // Filtro por precio
        if (userPrefs.max_price) {
          const item = await this.getItem(rec.itemId);
          if (item && item.price > userPrefs.max_price) {
            shouldInclude = false;
          }
        }

        // Filtro por rating mínimo
        if (userPrefs.min_rating) {
          const item = await this.getItem(rec.itemId);
          if (item && item.average_rating < userPrefs.min_rating) {
            shouldInclude = false;
          }
        }

        if (shouldInclude) {
          filteredRecommendations.push(rec);
        }
      }

      return filteredRecommendations;
    } catch (error) {
      console.error('Error aplicando filtros personales:', error);
      return recommendations;
    }
  }

  async updateUserPreferences(userId, preferences) {
    try {
      const currentPrefs = this.userPreferences.get(userId) || {};
      const updatedPrefs = { ...currentPrefs, ...preferences };
      
      this.userPreferences.set(userId, updatedPrefs);

      // Guardar en base de datos
      await this.saveUserPreferences(userId, updatedPrefs);

      console.log('Preferencias de usuario actualizadas');
    } catch (error) {
      console.error('Error actualizando preferencias:', error);
    }
  }

  async saveUserPreferences(userId, preferences) {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'UPDATE users SET preferences = ? WHERE user_id = ?',
          [JSON.stringify(preferences), userId],
          (tx, results) => resolve(results),
          (tx, error) => reject(error)
        );
      });
    });
  }

  async saveRecommendations(userId, recommendations) {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        // Limpiar recomendaciones anteriores
        tx.executeSql(
          'DELETE FROM recommendations WHERE user_id = ?',
          [userId],
          (tx, results) => {
            // Insertar nuevas recomendaciones
            let completed = 0;
            const total = recommendations.length;

            if (total === 0) {
              resolve();
              return;
            }

            recommendations.forEach(rec => {
              tx.executeSql(
                'INSERT INTO recommendations (user_id, item_id, score, algorithm) VALUES (?, ?, ?, ?)',
                [userId, rec.itemId, rec.score, rec.algorithm],
                (tx, results) => {
                  completed++;
                  if (completed === total) {
                    resolve();
                  }
                },
                (tx, error) => reject(error)
              );
            });
          },
          (tx, error) => reject(error)
        );
      });
    });
  }

  async getUsers() {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM users',
          [],
          (tx, results) => {
            const users = [];
            for (let i = 0; i < results.rows.length; i++) {
              users.push(results.rows.item(i));
            }
            resolve(users);
          },
          (tx, error) => reject(error)
        );
      });
    });
  }

  async getItem(itemId) {
    return new Promise((resolve, reject) => {
      this.db.transaction(tx => {
        tx.executeSql(
          'SELECT * FROM items WHERE item_id = ?',
          [itemId],
          (tx, results) => {
            if (results.rows.length > 0) {
              resolve(results.rows.item(0));
            } else {
              resolve(null);
            }
          },
          (tx, error) => reject(error)
        );
      });
    });
  }
}

export default PersonalizationEngine;
```

---

## 🧪 A/B Testing para Recomendaciones

### **Implementación de A/B Testing**

```javascript
// ABTesting.js
class ABTesting {
  constructor() {
    this.experiments = new Map();
    this.userAssignments = new Map();
  }

  createExperiment(experimentId, variants, trafficAllocation = 1.0) {
    const experiment = {
      id: experimentId,
      variants: variants,
      trafficAllocation: trafficAllocation,
      startDate: new Date(),
      endDate: null,
      status: 'active',
      metrics: {
        impressions: 0,
        clicks: 0,
        conversions: 0
      }
    };

    this.experiments.set(experimentId, experiment);
    return experiment;
  }

  assignUserToVariant(userId, experimentId) {
    if (this.userAssignments.has(`${userId}_${experimentId}`)) {
      return this.userAssignments.get(`${userId}_${experimentId}`);
    }

    const experiment = this.experiments.get(experimentId);
    if (!experiment || experiment.status !== 'active') {
      return null;
    }

    // Asignar usuario a variante basado en hash
    const hash = this.hashUserId(userId, experimentId);
    const variantIndex = Math.floor(hash * experiment.variants.length);
    const variant = experiment.variants[variantIndex];

    this.userAssignments.set(`${userId}_${experimentId}`, variant);
    return variant;
  }

  hashUserId(userId, experimentId) {
    const str = `${userId}_${experimentId}`;
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash) / 2147483647; // Normalize to 0-1
  }

  trackImpression(userId, experimentId, variant) {
    const experiment = this.experiments.get(experimentId);
    if (experiment) {
      experiment.metrics.impressions++;
    }
  }

  trackClick(userId, experimentId, variant) {
    const experiment = this.experiments.get(experimentId);
    if (experiment) {
      experiment.metrics.clicks++;
    }
  }

  trackConversion(userId, experimentId, variant) {
    const experiment = this.experiments.get(experimentId);
    if (experiment) {
      experiment.metrics.conversions++;
    }
  }

  getExperimentResults(experimentId) {
    const experiment = this.experiments.get(experimentId);
    if (!experiment) {
      return null;
    }

    const clickThroughRate = experiment.metrics.impressions > 0 
      ? experiment.metrics.clicks / experiment.metrics.impressions 
      : 0;

    const conversionRate = experiment.metrics.clicks > 0 
      ? experiment.metrics.conversions / experiment.metrics.clicks 
      : 0;

    return {
      experimentId,
      status: experiment.status,
      metrics: experiment.metrics,
      clickThroughRate,
      conversionRate,
      startDate: experiment.startDate,
      endDate: experiment.endDate
    };
  }

  endExperiment(experimentId) {
    const experiment = this.experiments.get(experimentId);
    if (experiment) {
      experiment.status = 'ended';
      experiment.endDate = new Date();
    }
  }
}

export default ABTesting;
```

---

## 📊 Analytics y Métricas

### **Sistema de Analytics**

```javascript
// RecommendationAnalytics.js
class RecommendationAnalytics {
  constructor() {
    this.metrics = {
      impressions: 0,
      clicks: 0,
      conversions: 0,
      revenue: 0,
      userEngagement: new Map(),
      algorithmPerformance: new Map()
    };
  }

  trackImpression(userId, itemId, algorithm) {
    this.metrics.impressions++;
    this.updateUserEngagement(userId, 'impression');
    this.updateAlgorithmPerformance(algorithm, 'impression');
  }

  trackClick(userId, itemId, algorithm) {
    this.metrics.clicks++;
    this.updateUserEngagement(userId, 'click');
    this.updateAlgorithmPerformance(algorithm, 'click');
  }

  trackConversion(userId, itemId, algorithm, value = 0) {
    this.metrics.conversions++;
    this.metrics.revenue += value;
    this.updateUserEngagement(userId, 'conversion');
    this.updateAlgorithmPerformance(algorithm, 'conversion');
  }

  updateUserEngagement(userId, action) {
    const userEngagement = this.metrics.userEngagement.get(userId) || {
      impressions: 0,
      clicks: 0,
      conversions: 0,
      lastActivity: new Date()
    };

    userEngagement[action]++;
    userEngagement.lastActivity = new Date();
    this.metrics.userEngagement.set(userId, userEngagement);
  }

  updateAlgorithmPerformance(algorithm, action) {
    const performance = this.metrics.algorithmPerformance.get(algorithm) || {
      impressions: 0,
      clicks: 0,
      conversions: 0
    };

    performance[action]++;
    this.metrics.algorithmPerformance.set(algorithm, performance);
  }

  getClickThroughRate() {
    return this.metrics.impressions > 0 
      ? this.metrics.clicks / this.metrics.impressions 
      : 0;
  }

  getConversionRate() {
    return this.metrics.clicks > 0 
      ? this.metrics.conversions / this.metrics.clicks 
      : 0;
  }

  getRevenuePerUser() {
    const activeUsers = this.metrics.userEngagement.size;
    return activeUsers > 0 ? this.metrics.revenue / activeUsers : 0;
  }

  getAlgorithmPerformance() {
    const performance = {};
    
    for (const [algorithm, metrics] of this.metrics.algorithmPerformance) {
      performance[algorithm] = {
        ...metrics,
        clickThroughRate: metrics.impressions > 0 ? metrics.clicks / metrics.impressions : 0,
        conversionRate: metrics.clicks > 0 ? metrics.conversions / metrics.clicks : 0
      };
    }
    
    return performance;
  }

  getUserEngagement(userId) {
    return this.metrics.userEngagement.get(userId) || {
      impressions: 0,
      clicks: 0,
      conversions: 0,
      lastActivity: null
    };
  }

  getOverallMetrics() {
    return {
      ...this.metrics,
      clickThroughRate: this.getClickThroughRate(),
      conversionRate: this.getConversionRate(),
      revenuePerUser: this.getRevenuePerUser(),
      algorithmPerformance: this.getAlgorithmPerformance()
    };
  }
}

export default RecommendationAnalytics;
```

---

## 🎯 Implementación Práctica

### **Componente de Recomendaciones**

```javascript
// RecommendationScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator
} from 'react-native';
import PersonalizationEngine from './PersonalizationEngine';
import ABTesting from './ABTesting';
import RecommendationAnalytics from './RecommendationAnalytics';

const RecommendationScreen = () => {
  const [recommendations, setRecommendations] = useState([]);
  const [loading, setLoading] = useState(false);
  const [userId] = useState('user_123'); // En una app real, esto vendría del auth
  
  // Sistemas
  const [personalizationEngine, setPersonalizationEngine] = useState(null);
  const [abTesting, setABTesting] = useState(null);
  const [analytics, setAnalytics] = useState(null);

  useEffect(() => {
    initializeSystems();
  }, []);

  const initializeSystems = async () => {
    try {
      // Inicializar sistemas
      const engine = new PersonalizationEngine();
      const ab = new ABTesting();
      const analytics = new RecommendationAnalytics();

      // Configurar experimento A/B
      ab.createExperiment('recommendation_algorithm', [
        { id: 'collaborative', name: 'Collaborative Filtering' },
        { id: 'content_based', name: 'Content-Based Filtering' },
        { id: 'hybrid', name: 'Hybrid Approach' }
      ]);

      setPersonalizationEngine(engine);
      setABTesting(ab);
      setAnalytics(analytics);

      // Cargar recomendaciones iniciales
      await loadRecommendations();
    } catch (error) {
      Alert.alert('Error', 'No se pudieron inicializar los sistemas');
    }
  };

  const loadRecommendations = async () => {
    if (!personalizationEngine || !abTesting || !analytics) {
      return;
    }

    try {
      setLoading(true);

      // Asignar usuario a variante del experimento
      const variant = abTesting.assignUserToVariant(userId, 'recommendation_algorithm');
      
      // Generar recomendaciones basadas en la variante
      const options = getRecommendationOptions(variant);
      const recs = await personalizationEngine.getPersonalizedRecommendations(userId, options);
      
      setRecommendations(recs);

      // Trackear impresión
      recs.forEach(rec => {
        analytics.trackImpression(userId, rec.itemId, rec.algorithm);
      });

    } catch (error) {
      Alert.alert('Error', 'No se pudieron cargar las recomendaciones');
    } finally {
      setLoading(false);
    }
  };

  const getRecommendationOptions = (variant) => {
    switch (variant.id) {
      case 'collaborative':
        return { algorithms: ['collaborative'], weights: { collaborative: 1.0 } };
      case 'content_based':
        return { algorithms: ['content_based'], weights: { content_based: 1.0 } };
      case 'hybrid':
        return { algorithms: ['collaborative', 'content_based'], weights: { collaborative: 0.6, content_based: 0.4 } };
      default:
        return { algorithms: ['collaborative', 'content_based'], weights: { collaborative: 0.6, content_based: 0.4 } };
    }
  };

  const handleItemClick = (item) => {
    // Trackear click
    analytics.trackClick(userId, item.itemId, item.algorithm);
    
    // Navegar a detalles del item
    // navigation.navigate('ItemDetails', { itemId: item.itemId });
  };

  const handleItemPurchase = (item) => {
    // Trackear conversión
    analytics.trackConversion(userId, item.itemId, item.algorithm, item.price || 0);
    
    Alert.alert('Compra', `Has comprado ${item.itemId}`);
  };

  const renderRecommendationItem = ({ item }) => (
    <TouchableOpacity 
      style={styles.itemContainer}
      onPress={() => handleItemClick(item)}
    >
      <Text style={styles.itemTitle}>{item.itemId}</Text>
      <Text style={styles.itemScore}>Score: {item.score.toFixed(3)}</Text>
      <Text style={styles.itemAlgorithm}>Algorithm: {item.algorithm}</Text>
      
      <TouchableOpacity 
        style={styles.buyButton}
        onPress={() => handleItemPurchase(item)}
      >
        <Text style={styles.buyButtonText}>Comprar</Text>
      </TouchableOpacity>
    </TouchableOpacity>
  );

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
        <Text>Cargando recomendaciones...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Recomendaciones Personalizadas</Text>
      
      <FlatList
        data={recommendations}
        renderItem={renderRecommendationItem}
        keyExtractor={(item) => item.itemId}
        style={styles.list}
      />
      
      <TouchableOpacity style={styles.refreshButton} onPress={loadRecommendations}>
        <Text style={styles.refreshButtonText}>Actualizar Recomendaciones</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  list: {
    flex: 1,
  },
  itemContainer: {
    backgroundColor: 'white',
    padding: 15,
    marginBottom: 10,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  itemTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  itemScore: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
  itemAlgorithm: {
    fontSize: 12,
    color: '#999',
    marginBottom: 10,
  },
  buyButton: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 5,
    alignSelf: 'flex-start',
  },
  buyButtonText: {
    color: 'white',
    fontWeight: 'bold',
  },
  refreshButton: {
    backgroundColor: '#34C759',
    padding: 15,
    borderRadius: 10,
    marginTop: 10,
  },
  refreshButtonText: {
    color: 'white',
    textAlign: 'center',
    fontWeight: 'bold',
  },
});

export default RecommendationScreen;
```

---

## 🚀 Próximos Pasos

### **Lo que Aprendiste**

1. **Implementación** de sistemas de recomendación colaborativos
2. **Filtrado basado en contenido** y comportamiento
3. **Algoritmos de personalización** avanzados
4. **A/B testing** para optimización
5. **Analytics** y métricas de engagement

### **Preparación para la Siguiente Clase**

En la próxima clase aprenderás sobre:
- **Edge Computing** vs cloud computing
- **Modelos locales** y sincronización
- **Optimización** de batería y rendimiento
- **Privacidad** y seguridad de datos
- **Federated Learning** y aprendizaje distribuido

### **Tarea para Casa**

1. **Implementar** un sistema de recomendaciones personalizado
2. **Crear** experimentos A/B para optimización
3. **Desarrollar** analytics para métricas de engagement
4. **Optimizar** algoritmos de recomendación

---

## 📚 Recursos Adicionales

### **Algoritmos de Recomendación**
- [Collaborative Filtering](https://en.wikipedia.org/wiki/Collaborative_filtering)
- [Content-Based Filtering](https://en.wikipedia.org/wiki/Content-based_filtering)
- [Matrix Factorization](https://en.wikipedia.org/wiki/Matrix_factorization_(recommender_systems))

### **Herramientas de ML**
- [ML-Matrix](https://github.com/mljs/matrix)
- [ML-KMeans](https://github.com/mljs/kmeans)
- [TensorFlow.js](https://tensorflow.org/js)

### **A/B Testing**
- [A/B Testing Best Practices](https://www.optimizely.com/optimization-glossary/ab-testing/)
- [Statistical Significance](https://en.wikipedia.org/wiki/Statistical_significance)

---

**🎯 Objetivo de la Clase**: Dominar la implementación de sistemas avanzados de recomendaciones y personalización en React Native, desde algoritmos colaborativos hasta A/B testing.

**💡 Consejo**: Combina múltiples algoritmos de recomendación para obtener mejores resultados. El A/B testing es esencial para optimizar el rendimiento.

---

**🚀 ¡Has completado la Clase 4 de Recomendaciones y Personalización! Continúa con la Clase 5 para dominar el Edge Computing.**
