# ⚡ Clase 5: Edge Computing y ML Local

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar edge computing y machine learning local en React Native, incluyendo modelos locales, sincronización, optimización de batería, privacidad y federated learning.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** edge computing vs cloud computing
2. **Crear** modelos locales y sincronización
3. **Optimizar** batería y rendimiento
4. **Implementar** privacidad y seguridad de datos
5. **Desarrollar** federated learning

---

## 🛠️ Configuración de Edge Computing

### **Instalación de Dependencias**

```bash
# Instalar librerías de edge computing
npm install @tensorflow/tfjs @tensorflow/tfjs-react-native
npm install react-native-fs react-native-background-job
npm install @react-native-async-storage/async-storage
npm install react-native-device-info react-native-battery-optimization

# Instalar dependencias de sincronización
npm install react-native-sync-storage
npm install react-native-network-info
```

### **Configuración de Permisos**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

---

## 🏠 Modelos Locales

### **Implementación de Modelo Local**

```javascript
// LocalModelManager.js
import * as tf from '@tensorflow/tfjs';
import '@tensorflow/tfjs-react-native';
import RNFS from 'react-native-fs';

class LocalModelManager {
  constructor() {
    this.models = new Map();
    this.modelCache = new Map();
    this.isOnline = true;
  }

  async loadLocalModel(modelId, modelPath) {
    try {
      // Verificar si el modelo ya está cargado
      if (this.models.has(modelId)) {
        return this.models.get(modelId);
      }

      // Cargar modelo desde almacenamiento local
      const model = await tf.loadLayersModel(`file://${modelPath}`);
      
      // Cachear modelo
      this.models.set(modelId, model);
      this.modelCache.set(modelId, {
        path: modelPath,
        loadedAt: Date.now(),
        size: await this.getModelSize(modelPath)
      });

      console.log(`Modelo local ${modelId} cargado exitosamente`);
      return model;
    } catch (error) {
      console.error(`Error cargando modelo local ${modelId}:`, error);
      throw error;
    }
  }

  async downloadAndCacheModel(modelId, modelUrl) {
    try {
      const localPath = `${RNFS.DocumentDirectoryPath}/models/${modelId}`;
      
      // Crear directorio si no existe
      await RNFS.mkdir(`${RNFS.DocumentDirectoryPath}/models`, { NSURLIsExcludedFromBackupKey: true });
      
      // Descargar modelo
      const downloadResult = await RNFS.downloadFile({
        fromUrl: modelUrl,
        toFile: localPath,
        progress: (res) => {
          console.log(`Descargando modelo ${modelId}: ${res.bytesWritten}/${res.contentLength}`);
        }
      }).promise;

      if (downloadResult.statusCode === 200) {
        console.log(`Modelo ${modelId} descargado y cacheado`);
        return localPath;
      } else {
        throw new Error(`Error descargando modelo: ${downloadResult.statusCode}`);
      }
    } catch (error) {
      console.error(`Error descargando modelo ${modelId}:`, error);
      throw error;
    }
  }

  async getModelSize(modelPath) {
    try {
      const stats = await RNFS.stat(modelPath);
      return stats.size;
    } catch (error) {
      console.error('Error obteniendo tamaño del modelo:', error);
      return 0;
    }
  }

  async predict(modelId, inputData) {
    try {
      const model = this.models.get(modelId);
      if (!model) {
        throw new Error(`Modelo ${modelId} no cargado`);
      }

      const input = tf.tensor(inputData);
      const prediction = await model.predict(input);
      const result = await prediction.data();
      
      // Limpiar tensores
      input.dispose();
      prediction.dispose();
      
      return result;
    } catch (error) {
      console.error(`Error en predicción del modelo ${modelId}:`, error);
      throw error;
    }
  }

  async updateModel(modelId, newModelUrl) {
    try {
      // Descargar nuevo modelo
      const newModelPath = await this.downloadAndCacheModel(modelId, newModelUrl);
      
      // Cargar nuevo modelo
      const newModel = await this.loadLocalModel(modelId, newModelPath);
      
      // Reemplazar modelo anterior
      this.models.set(modelId, newModel);
      
      console.log(`Modelo ${modelId} actualizado exitosamente`);
      return newModel;
    } catch (error) {
      console.error(`Error actualizando modelo ${modelId}:`, error);
      throw error;
    }
  }

  getModelInfo(modelId) {
    return this.modelCache.get(modelId);
  }

  getAllModels() {
    return Array.from(this.modelCache.entries()).map(([id, info]) => ({
      id,
      ...info
    }));
  }

  async clearModelCache() {
    try {
      // Limpiar modelos de memoria
      for (const [modelId, model] of this.models) {
        model.dispose();
      }
      this.models.clear();
      this.modelCache.clear();

      // Limpiar archivos de modelo
      const modelsDir = `${RNFS.DocumentDirectoryPath}/models`;
      const files = await RNFS.readDir(modelsDir);
      
      for (const file of files) {
        await RNFS.unlink(file.path);
      }

      console.log('Cache de modelos limpiado');
    } catch (error) {
      console.error('Error limpiando cache de modelos:', error);
    }
  }
}

export default LocalModelManager;
```

---

## 🔄 Sincronización de Modelos

### **Sistema de Sincronización**

```javascript
// ModelSyncManager.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import NetInfo from '@react-native-community/netinfo';

class ModelSyncManager {
  constructor() {
    this.syncQueue = [];
    this.isSyncing = false;
    this.syncInterval = null;
    this.lastSyncTime = null;
  }

  async initialize() {
    // Verificar conectividad
    const netInfo = await NetInfo.fetch();
    this.isOnline = netInfo.isConnected;
    
    // Configurar listener de conectividad
    NetInfo.addEventListener(state => {
      this.isOnline = state.isConnected;
      if (this.isOnline && this.syncQueue.length > 0) {
        this.startSync();
      }
    });

    // Cargar última sincronización
    this.lastSyncTime = await AsyncStorage.getItem('lastModelSync');
  }

  async syncModel(modelId, modelData) {
    try {
      if (!this.isOnline) {
        // Agregar a cola de sincronización
        this.syncQueue.push({ modelId, modelData, timestamp: Date.now() });
        await this.saveSyncQueue();
        return false;
      }

      // Sincronizar modelo
      const response = await fetch(`https://api.example.com/models/${modelId}/sync`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          modelId,
          data: modelData,
          timestamp: Date.now()
        })
      });

      if (response.ok) {
        this.lastSyncTime = Date.now();
        await AsyncStorage.setItem('lastModelSync', this.lastSyncTime.toString());
        console.log(`Modelo ${modelId} sincronizado exitosamente`);
        return true;
      } else {
        throw new Error(`Error sincronizando modelo: ${response.status}`);
      }
    } catch (error) {
      console.error(`Error sincronizando modelo ${modelId}:`, error);
      // Agregar a cola de sincronización
      this.syncQueue.push({ modelId, modelData, timestamp: Date.now() });
      await this.saveSyncQueue();
      return false;
    }
  }

  async startSync() {
    if (this.isSyncing || !this.isOnline) {
      return;
    }

    this.isSyncing = true;
    console.log('Iniciando sincronización de modelos...');

    try {
      // Procesar cola de sincronización
      while (this.syncQueue.length > 0) {
        const syncItem = this.syncQueue.shift();
        await this.syncModel(syncItem.modelId, syncItem.modelData);
      }

      // Limpiar cola guardada
      await AsyncStorage.removeItem('syncQueue');
      
      console.log('Sincronización completada');
    } catch (error) {
      console.error('Error en sincronización:', error);
    } finally {
      this.isSyncing = false;
    }
  }

  async saveSyncQueue() {
    try {
      await AsyncStorage.setItem('syncQueue', JSON.stringify(this.syncQueue));
    } catch (error) {
      console.error('Error guardando cola de sincronización:', error);
    }
  }

  async loadSyncQueue() {
    try {
      const queueData = await AsyncStorage.getItem('syncQueue');
      if (queueData) {
        this.syncQueue = JSON.parse(queueData);
      }
    } catch (error) {
      console.error('Error cargando cola de sincronización:', error);
    }
  }

  startPeriodicSync(intervalMinutes = 30) {
    if (this.syncInterval) {
      clearInterval(this.syncInterval);
    }

    this.syncInterval = setInterval(() => {
      if (this.isOnline && this.syncQueue.length > 0) {
        this.startSync();
      }
    }, intervalMinutes * 60 * 1000);
  }

  stopPeriodicSync() {
    if (this.syncInterval) {
      clearInterval(this.syncInterval);
      this.syncInterval = null;
    }
  }

  getSyncStatus() {
    return {
      isOnline: this.isOnline,
      isSyncing: this.isSyncing,
      queueLength: this.syncQueue.length,
      lastSyncTime: this.lastSyncTime
    };
  }
}

export default ModelSyncManager;
```

---

## 🔋 Optimización de Batería

### **Sistema de Optimización de Batería**

```javascript
// BatteryOptimizer.js
import DeviceInfo from 'react-native-device-info';
import { AppState } from 'react-native';

class BatteryOptimizer {
  constructor() {
    this.batteryLevel = 100;
    this.isCharging = false;
    this.powerMode = 'normal';
    this.optimizationSettings = {
      lowBatteryThreshold: 20,
      criticalBatteryThreshold: 10,
      modelUpdateInterval: 300000, // 5 minutos
      syncInterval: 600000, // 10 minutos
      backgroundProcessing: true
    };
  }

  async initialize() {
    // Obtener información de batería
    this.batteryLevel = await DeviceInfo.getBatteryLevel();
    this.isCharging = await DeviceInfo.isBatteryCharging();
    
    // Configurar listener de batería
    DeviceInfo.addBatteryLevelListener((batteryLevel) => {
      this.batteryLevel = batteryLevel;
      this.updatePowerMode();
    });

    DeviceInfo.addBatteryChargingListener((isCharging) => {
      this.isCharging = isCharging;
      this.updatePowerMode();
    });

    // Configurar listener de estado de la app
    AppState.addEventListener('change', this.handleAppStateChange.bind(this));
  }

  updatePowerMode() {
    if (this.batteryLevel <= this.optimizationSettings.criticalBatteryThreshold) {
      this.powerMode = 'critical';
    } else if (this.batteryLevel <= this.optimizationSettings.lowBatteryThreshold) {
      this.powerMode = 'low';
    } else if (this.isCharging) {
      this.powerMode = 'charging';
    } else {
      this.powerMode = 'normal';
    }

    this.applyPowerOptimizations();
  }

  applyPowerOptimizations() {
    switch (this.powerMode) {
      case 'critical':
        this.enableCriticalMode();
        break;
      case 'low':
        this.enableLowPowerMode();
        break;
      case 'charging':
        this.enableChargingMode();
        break;
      default:
        this.enableNormalMode();
    }
  }

  enableCriticalMode() {
    console.log('Modo crítico de batería activado');
    // Desactivar procesamiento en segundo plano
    this.optimizationSettings.backgroundProcessing = false;
    // Reducir frecuencia de actualizaciones
    this.optimizationSettings.modelUpdateInterval = 1800000; // 30 minutos
    this.optimizationSettings.syncInterval = 3600000; // 1 hora
  }

  enableLowPowerMode() {
    console.log('Modo de bajo consumo activado');
    // Reducir procesamiento en segundo plano
    this.optimizationSettings.backgroundProcessing = true;
    // Aumentar intervalos
    this.optimizationSettings.modelUpdateInterval = 900000; // 15 minutos
    this.optimizationSettings.syncInterval = 1800000; // 30 minutos
  }

  enableChargingMode() {
    console.log('Modo de carga activado');
    // Permitir procesamiento intensivo
    this.optimizationSettings.backgroundProcessing = true;
    // Reducir intervalos para sincronización rápida
    this.optimizationSettings.modelUpdateInterval = 60000; // 1 minuto
    this.optimizationSettings.syncInterval = 300000; // 5 minutos
  }

  enableNormalMode() {
    console.log('Modo normal activado');
    // Configuración estándar
    this.optimizationSettings.backgroundProcessing = true;
    this.optimizationSettings.modelUpdateInterval = 300000; // 5 minutos
    this.optimizationSettings.syncInterval = 600000; // 10 minutos
  }

  handleAppStateChange(nextAppState) {
    if (nextAppState === 'background') {
      this.handleBackgroundMode();
    } else if (nextAppState === 'active') {
      this.handleForegroundMode();
    }
  }

  handleBackgroundMode() {
    if (this.powerMode === 'critical' || this.powerMode === 'low') {
      // Pausar procesamiento en segundo plano
      this.pauseBackgroundProcessing();
    }
  }

  handleForegroundMode() {
    // Reanudar procesamiento
    this.resumeBackgroundProcessing();
  }

  pauseBackgroundProcessing() {
    console.log('Pausando procesamiento en segundo plano');
    // Implementar lógica para pausar procesamiento
  }

  resumeBackgroundProcessing() {
    console.log('Reanudando procesamiento en segundo plano');
    // Implementar lógica para reanudar procesamiento
  }

  getOptimizationSettings() {
    return this.optimizationSettings;
  }

  getBatteryStatus() {
    return {
      level: this.batteryLevel,
      isCharging: this.isCharging,
      powerMode: this.powerMode
    };
  }
}

export default BatteryOptimizer;
```

---

## 🔒 Privacidad y Seguridad

### **Sistema de Privacidad**

```javascript
// PrivacyManager.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import CryptoJS from 'crypto-js';

class PrivacyManager {
  constructor() {
    this.encryptionKey = null;
    this.privacySettings = {
      dataEncryption: true,
      localProcessing: true,
      dataRetention: 30, // días
      anonymizeData: true
    };
  }

  async initialize() {
    // Generar o cargar clave de encriptación
    this.encryptionKey = await this.getOrCreateEncryptionKey();
    
    // Cargar configuraciones de privacidad
    await this.loadPrivacySettings();
  }

  async getOrCreateEncryptionKey() {
    try {
      let key = await AsyncStorage.getItem('encryptionKey');
      if (!key) {
        key = CryptoJS.lib.WordArray.random(256/8).toString();
        await AsyncStorage.setItem('encryptionKey', key);
      }
      return key;
    } catch (error) {
      console.error('Error generando clave de encriptación:', error);
      throw error;
    }
  }

  async loadPrivacySettings() {
    try {
      const settings = await AsyncStorage.getItem('privacySettings');
      if (settings) {
        this.privacySettings = { ...this.privacySettings, ...JSON.parse(settings) };
      }
    } catch (error) {
      console.error('Error cargando configuraciones de privacidad:', error);
    }
  }

  async savePrivacySettings(settings) {
    try {
      this.privacySettings = { ...this.privacySettings, ...settings };
      await AsyncStorage.setItem('privacySettings', JSON.stringify(this.privacySettings));
    } catch (error) {
      console.error('Error guardando configuraciones de privacidad:', error);
    }
  }

  encryptData(data) {
    if (!this.privacySettings.dataEncryption) {
      return data;
    }

    try {
      const encrypted = CryptoJS.AES.encrypt(JSON.stringify(data), this.encryptionKey).toString();
      return encrypted;
    } catch (error) {
      console.error('Error encriptando datos:', error);
      return data;
    }
  }

  decryptData(encryptedData) {
    if (!this.privacySettings.dataEncryption) {
      return encryptedData;
    }

    try {
      const decrypted = CryptoJS.AES.decrypt(encryptedData, this.encryptionKey);
      return JSON.parse(decrypted.toString(CryptoJS.enc.Utf8));
    } catch (error) {
      console.error('Error desencriptando datos:', error);
      return encryptedData;
    }
  }

  anonymizeUserData(userData) {
    if (!this.privacySettings.anonymizeData) {
      return userData;
    }

    try {
      const anonymized = {
        ...userData,
        userId: this.hashUserId(userData.userId),
        email: this.hashEmail(userData.email),
        phone: this.hashPhone(userData.phone)
      };

      return anonymized;
    } catch (error) {
      console.error('Error anonimizando datos:', error);
      return userData;
    }
  }

  hashUserId(userId) {
    return CryptoJS.SHA256(userId).toString();
  }

  hashEmail(email) {
    return CryptoJS.SHA256(email).toString();
  }

  hashPhone(phone) {
    return CryptoJS.SHA256(phone).toString();
  }

  async cleanupOldData() {
    try {
      const cutoffDate = new Date();
      cutoffDate.setDate(cutoffDate.getDate() - this.privacySettings.dataRetention);

      // Limpiar datos antiguos
      await this.cleanupOldInteractions(cutoffDate);
      await this.cleanupOldModels(cutoffDate);
      await this.cleanupOldLogs(cutoffDate);

      console.log('Limpieza de datos antiguos completada');
    } catch (error) {
      console.error('Error limpiando datos antiguos:', error);
    }
  }

  async cleanupOldInteractions(cutoffDate) {
    // Implementar limpieza de interacciones antiguas
    console.log('Limpiando interacciones antiguas...');
  }

  async cleanupOldModels(cutoffDate) {
    // Implementar limpieza de modelos antiguos
    console.log('Limpiando modelos antiguos...');
  }

  async cleanupOldLogs(cutoffDate) {
    // Implementar limpieza de logs antiguos
    console.log('Limpiando logs antiguos...');
  }

  getPrivacySettings() {
    return this.privacySettings;
  }

  async exportUserData(userId) {
    try {
      const userData = {
        interactions: await this.getUserInteractions(userId),
        preferences: await this.getUserPreferences(userId),
        models: await this.getUserModels(userId)
      };

      return this.encryptData(userData);
    } catch (error) {
      console.error('Error exportando datos del usuario:', error);
      throw error;
    }
  }

  async deleteUserData(userId) {
    try {
      await this.deleteUserInteractions(userId);
      await this.deleteUserPreferences(userId);
      await this.deleteUserModels(userId);
      
      console.log(`Datos del usuario ${userId} eliminados`);
    } catch (error) {
      console.error('Error eliminando datos del usuario:', error);
      throw error;
    }
  }
}

export default PrivacyManager;
```

---

## 🤝 Federated Learning

### **Implementación de Federated Learning**

```javascript
// FederatedLearning.js
import * as tf from '@tensorflow/tfjs';

class FederatedLearning {
  constructor() {
    this.localModel = null;
    this.globalModel = null;
    this.trainingData = [];
    this.isTraining = false;
    this.roundNumber = 0;
  }

  async initialize(modelPath) {
    try {
      // Cargar modelo base
      this.localModel = await tf.loadLayersModel(modelPath);
      this.globalModel = await tf.loadLayersModel(modelPath);
      
      console.log('Federated Learning inicializado');
    } catch (error) {
      console.error('Error inicializando Federated Learning:', error);
      throw error;
    }
  }

  async addTrainingData(input, output) {
    this.trainingData.push({ input, output });
  }

  async trainLocalModel(epochs = 5) {
    if (this.trainingData.length === 0) {
      throw new Error('No hay datos de entrenamiento');
    }

    try {
      this.isTraining = true;
      
      // Preparar datos de entrenamiento
      const inputs = this.trainingData.map(d => d.input);
      const outputs = this.trainingData.map(d => d.output);
      
      const inputTensor = tf.tensor2d(inputs);
      const outputTensor = tf.tensor2d(outputs);
      
      // Entrenar modelo local
      const history = await this.localModel.fit(inputTensor, outputTensor, {
        epochs,
        batchSize: 32,
        validationSplit: 0.2,
        callbacks: {
          onEpochEnd: (epoch, logs) => {
            console.log(`Epoch ${epoch}: loss = ${logs.loss.toFixed(4)}`);
          }
        }
      });
      
      // Limpiar tensores
      inputTensor.dispose();
      outputTensor.dispose();
      
      this.isTraining = false;
      return history;
    } catch (error) {
      this.isTraining = false;
      console.error('Error entrenando modelo local:', error);
      throw error;
    }
  }

  async getModelUpdates() {
    if (!this.localModel || !this.globalModel) {
      throw new Error('Modelos no inicializados');
    }

    try {
      const updates = [];
      const localWeights = this.localModel.getWeights();
      const globalWeights = this.globalModel.getWeights();
      
      // Calcular diferencias de pesos
      for (let i = 0; i < localWeights.length; i++) {
        const update = localWeights[i].sub(globalWeights[i]);
        updates.push(update);
      }
      
      return updates;
    } catch (error) {
      console.error('Error obteniendo actualizaciones del modelo:', error);
      throw error;
    }
  }

  async applyGlobalUpdates(globalUpdates) {
    if (!this.localModel) {
      throw new Error('Modelo local no inicializado');
    }

    try {
      const currentWeights = this.localModel.getWeights();
      const newWeights = [];
      
      // Aplicar actualizaciones globales
      for (let i = 0; i < currentWeights.length; i++) {
        const newWeight = currentWeights[i].add(globalUpdates[i]);
        newWeights.push(newWeight);
      }
      
      // Actualizar pesos del modelo local
      this.localModel.setWeights(newWeights);
      
      console.log('Actualizaciones globales aplicadas al modelo local');
    } catch (error) {
      console.error('Error aplicando actualizaciones globales:', error);
      throw error;
    }
  }

  async participateInFederatedRound() {
    try {
      // Entrenar modelo local
      await this.trainLocalModel();
      
      // Obtener actualizaciones
      const updates = await this.getModelUpdates();
      
      // Enviar actualizaciones al servidor
      const response = await fetch('https://api.example.com/federated-learning/updates', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          roundNumber: this.roundNumber,
          updates: updates.map(update => update.dataSync())
        })
      });
      
      if (response.ok) {
        const result = await response.json();
        
        // Aplicar actualizaciones globales
        if (result.globalUpdates) {
          await this.applyGlobalUpdates(result.globalUpdates);
        }
        
        this.roundNumber++;
        console.log(`Ronda federada ${this.roundNumber} completada`);
      } else {
        throw new Error(`Error en ronda federada: ${response.status}`);
      }
    } catch (error) {
      console.error('Error participando en ronda federada:', error);
      throw error;
    }
  }

  async predict(inputData) {
    if (!this.localModel) {
      throw new Error('Modelo local no inicializado');
    }

    try {
      const input = tf.tensor2d([inputData]);
      const prediction = await this.localModel.predict(input);
      const result = await prediction.data();
      
      input.dispose();
      prediction.dispose();
      
      return result;
    } catch (error) {
      console.error('Error en predicción:', error);
      throw error;
    }
  }

  getTrainingStatus() {
    return {
      isTraining: this.isTraining,
      trainingDataSize: this.trainingData.length,
      roundNumber: this.roundNumber
    };
  }
}

export default FederatedLearning;
```

---

## 🎯 Implementación Práctica

### **Componente de Edge Computing**

```javascript
// EdgeComputingScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator,
  ScrollView
} from 'react-native';
import LocalModelManager from './LocalModelManager';
import ModelSyncManager from './ModelSyncManager';
import BatteryOptimizer from './BatteryOptimizer';
import PrivacyManager from './PrivacyManager';
import FederatedLearning from './FederatedLearning';

const EdgeComputingScreen = () => {
  const [loading, setLoading] = useState(false);
  const [status, setStatus] = useState({});
  
  // Sistemas
  const [localModelManager, setLocalModelManager] = useState(null);
  const [syncManager, setSyncManager] = useState(null);
  const [batteryOptimizer, setBatteryOptimizer] = useState(null);
  const [privacyManager, setPrivacyManager] = useState(null);
  const [federatedLearning, setFederatedLearning] = useState(null);

  useEffect(() => {
    initializeSystems();
  }, []);

  const initializeSystems = async () => {
    try {
      setLoading(true);
      
      // Inicializar sistemas
      const localManager = new LocalModelManager();
      const sync = new ModelSyncManager();
      const battery = new BatteryOptimizer();
      const privacy = new PrivacyManager();
      const federated = new FederatedLearning();

      await sync.initialize();
      await battery.initialize();
      await privacy.initialize();
      await federated.initialize('https://api.example.com/models/base-model.json');

      setLocalModelManager(localManager);
      setSyncManager(sync);
      setBatteryOptimizer(battery);
      setPrivacyManager(privacy);
      setFederatedLearning(federated);

      // Actualizar estado
      updateStatus();
    } catch (error) {
      Alert.alert('Error', 'No se pudieron inicializar los sistemas');
    } finally {
      setLoading(false);
    }
  };

  const updateStatus = () => {
    const newStatus = {
      battery: batteryOptimizer?.getBatteryStatus(),
      sync: syncManager?.getSyncStatus(),
      privacy: privacyManager?.getPrivacySettings(),
      federated: federatedLearning?.getTrainingStatus()
    };
    setStatus(newStatus);
  };

  const handleDownloadModel = async () => {
    try {
      setLoading(true);
      const modelPath = await localModelManager.downloadAndCacheModel('test-model', 'https://api.example.com/models/test-model.json');
      await localModelManager.loadLocalModel('test-model', modelPath);
      Alert.alert('Éxito', 'Modelo descargado y cargado');
    } catch (error) {
      Alert.alert('Error', 'No se pudo descargar el modelo');
    } finally {
      setLoading(false);
    }
  };

  const handleSyncModels = async () => {
    try {
      setLoading(true);
      await syncManager.startSync();
      Alert.alert('Éxito', 'Sincronización completada');
    } catch (error) {
      Alert.alert('Error', 'No se pudo sincronizar');
    } finally {
      setLoading(false);
    }
  };

  const handleFederatedRound = async () => {
    try {
      setLoading(true);
      await federatedLearning.participateInFederatedRound();
      Alert.alert('Éxito', 'Ronda federada completada');
    } catch (error) {
      Alert.alert('Error', 'No se pudo completar la ronda federada');
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
        <Text>Inicializando sistemas...</Text>
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Edge Computing y ML Local</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Estado del Sistema</Text>
        <Text>Batería: {status.battery?.level}% ({status.battery?.powerMode})</Text>
        <Text>Conectividad: {status.sync?.isOnline ? 'Online' : 'Offline'}</Text>
        <Text>Datos de entrenamiento: {status.federated?.trainingDataSize}</Text>
      </View>

      <TouchableOpacity style={styles.button} onPress={handleDownloadModel}>
        <Text style={styles.buttonText}>Descargar Modelo</Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.button} onPress={handleSyncModels}>
        <Text style={styles.buttonText}>Sincronizar Modelos</Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.button} onPress={handleFederatedRound}>
        <Text style={styles.buttonText}>Participar en Ronda Federada</Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.button} onPress={updateStatus}>
        <Text style={styles.buttonText}>Actualizar Estado</Text>
      </TouchableOpacity>
    </ScrollView>
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
  section: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
    fontWeight: 'bold',
  },
});

export default EdgeComputingScreen;
```

---

## 🚀 Próximos Pasos

### **Lo que Aprendiste**

1. **Implementación** de edge computing vs cloud computing
2. **Modelos locales** y sincronización
3. **Optimización** de batería y rendimiento
4. **Privacidad** y seguridad de datos
5. **Federated Learning** y aprendizaje distribuido

### **Preparación para el Siguiente Módulo**

En el siguiente módulo aprenderás sobre:
- **Compliance** y regulaciones
- **GDPR** y privacidad de datos
- **HIPAA** y seguridad médica
- **PCI DSS** y seguridad de pagos
- **SOC 2** y controles de seguridad

### **Tarea para Casa**

1. **Implementar** un sistema de edge computing completo
2. **Crear** modelos locales optimizados
3. **Desarrollar** sistema de privacidad
4. **Optimizar** para diferentes dispositivos

---

## 📚 Recursos Adicionales

### **Edge Computing**
- [Edge Computing Guide](https://www.ibm.com/cloud/edge-computing)
- [TensorFlow Lite](https://tensorflow.org/lite)
- [ONNX Runtime](https://onnxruntime.ai/)

### **Federated Learning**
- [Federated Learning Paper](https://arxiv.org/abs/1602.05629)
- [TensorFlow Federated](https://tensorflow.org/federated)
- [PySyft](https://github.com/OpenMined/PySyft)

### **Privacidad y Seguridad**
- [GDPR Guide](https://gdpr.eu/)
- [Privacy by Design](https://privacybydesign.ca/)
- [Data Protection](https://ico.org.uk/for-organisations/guide-to-data-protection/)

---

**🎯 Objetivo de la Clase**: Dominar la implementación de edge computing y machine learning local en React Native, desde modelos locales hasta federated learning.

**💡 Consejo**: El edge computing es el futuro del ML móvil. Prioriza la privacidad y optimiza para batería desde el inicio.

---

**🚀 ¡Has completado la Clase 5 de Edge Computing y ML Local! Has terminado el Módulo 23 de Machine Learning e IA.**
