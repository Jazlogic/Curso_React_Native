# 🤖 Clase 1: TensorFlow Lite en React Native

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a integrar TensorFlow Lite en aplicaciones React Native, configurar el entorno de desarrollo, trabajar con modelos pre-entrenados y personalizados, y optimizar el rendimiento para dispositivos móviles.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Configurar** TensorFlow Lite en React Native
2. **Integrar** modelos pre-entrenados
3. **Crear** modelos personalizados
4. **Optimizar** para dispositivos móviles
5. **Implementar** inferencia en tiempo real

---

## 🛠️ Configuración del Entorno

### **Instalación de Dependencias**

```bash
# Instalar TensorFlow Lite para React Native
npm install @tensorflow/tfjs @tensorflow/tfjs-react-native @tensorflow/tfjs-platform-react-native

# Instalar dependencias adicionales
npm install react-native-fs react-native-image-picker
npm install @react-native-async-storage/async-storage

# Para iOS
cd ios && pod install
```

### **Configuración de Android**

```gradle
// android/app/build.gradle
android {
    compileSdkVersion 33
    
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 33
    }
    
    packagingOptions {
        pickFirst '**/libc++_shared.so'
        pickFirst '**/libjsc.so'
    }
}
```

### **Configuración de iOS**

```swift
// ios/Podfile
platform :ios, '11.0'

target 'YourApp' do
  use_frameworks!
  
  pod 'TensorFlowLiteSwift'
  pod 'TensorFlowLiteObjC'
end
```

---

## 🚀 Fundamentos de TensorFlow Lite

### **¿Qué es TensorFlow Lite?**

TensorFlow Lite es una versión optimizada de TensorFlow diseñada específicamente para dispositivos móviles y embebidos:

- **Tamaño reducido**: Modelos optimizados para móviles
- **Inferencia rápida**: Optimizado para CPU, GPU y NPU
- **Bajo consumo**: Eficiente en batería y memoria
- **Cross-platform**: iOS, Android, Linux, microcontroladores

### **Arquitectura de TensorFlow Lite**

```javascript
// Estructura básica de TensorFlow Lite
const tf = require('@tensorflow/tfjs');
const tfReactNative = require('@tensorflow/tfjs-react-native');

// Inicializar TensorFlow
await tfReactNative.ready();
```

### **Tipos de Modelos**

1. **Modelos Pre-entrenados**
   - MobileNet (clasificación de imágenes)
   - PoseNet (detección de poses)
   - DeepLab (segmentación semántica)

2. **Modelos Personalizados**
   - Entrenados con TensorFlow
   - Convertidos a formato .tflite
   - Optimizados para móviles

---

## 📱 Integración con React Native

### **Configuración Inicial**

```javascript
// App.js
import React, { useEffect } from 'react';
import { Platform } from 'react-native';
import * as tf from '@tensorflow/tfjs';
import '@tensorflow/tfjs-react-native';
import '@tensorflow/tfjs-platform-react-native';

const App = () => {
  useEffect(() => {
    const initializeTensorFlow = async () => {
      // Inicializar TensorFlow
      await tf.ready();
      console.log('TensorFlow.js está listo!');
    };
    
    initializeTensorFlow();
  }, []);

  return (
    // Tu componente principal
  );
};

export default App;
```

### **Carga de Modelos**

```javascript
// ModelLoader.js
import * as tf from '@tensorflow/tfjs';
import { bundleResourceIO } from '@tensorflow/tfjs-react-native';

class ModelLoader {
  constructor() {
    this.model = null;
  }

  async loadModel(modelPath) {
    try {
      // Cargar modelo desde bundle
      this.model = await tf.loadLayersModel(
        bundleResourceIO(modelPath)
      );
      
      console.log('Modelo cargado exitosamente');
      return this.model;
    } catch (error) {
      console.error('Error cargando modelo:', error);
      throw error;
    }
  }

  async loadModelFromURL(modelUrl) {
    try {
      // Cargar modelo desde URL
      this.model = await tf.loadLayersModel(modelUrl);
      
      console.log('Modelo cargado desde URL');
      return this.model;
    } catch (error) {
      console.error('Error cargando modelo desde URL:', error);
      throw error;
    }
  }

  getModel() {
    return this.model;
  }
}

export default ModelLoader;
```

---

## 🎯 Modelos Pre-entrenados

### **MobileNet para Clasificación**

```javascript
// ImageClassifier.js
import * as tf from '@tensorflow/tfjs';
import { decodeJpeg } from '@tensorflow/tfjs-react-native';

class ImageClassifier {
  constructor() {
    this.model = null;
    this.labels = [];
  }

  async loadMobileNet() {
    try {
      // Cargar MobileNet pre-entrenado
      this.model = await tf.loadLayersModel(
        'https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_0.25_224/model.json'
      );
      
      // Cargar etiquetas
      const response = await fetch(
        'https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_0.25_224/labels.json'
      );
      this.labels = await response.json();
      
      console.log('MobileNet cargado exitosamente');
    } catch (error) {
      console.error('Error cargando MobileNet:', error);
    }
  }

  async classifyImage(imageUri) {
    if (!this.model) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Cargar y preprocesar imagen
      const imageAsset = require(imageUri);
      const imageTensor = decodeJpeg(imageAsset);
      
      // Redimensionar a 224x224
      const resized = tf.image.resizeBilinear(imageTensor, [224, 224]);
      
      // Normalizar valores
      const normalized = resized.div(255.0);
      
      // Agregar dimensión de batch
      const batched = normalized.expandDims(0);
      
      // Realizar predicción
      const predictions = await this.model.predict(batched);
      const topPredictions = await this.getTopPredictions(predictions, 5);
      
      // Limpiar tensores
      imageTensor.dispose();
      resized.dispose();
      normalized.dispose();
      batched.dispose();
      predictions.dispose();
      
      return topPredictions;
    } catch (error) {
      console.error('Error en clasificación:', error);
      throw error;
    }
  }

  async getTopPredictions(predictions, topK) {
    const values = await predictions.data();
    const topKIndices = tf.topk(predictions, topK).indices;
    const topKValues = tf.topk(predictions, topK).values;
    
    const indices = await topKIndices.data();
    const valuesArray = await topKValues.data();
    
    const results = [];
    for (let i = 0; i < topK; i++) {
      results.push({
        label: this.labels[indices[i]],
        confidence: valuesArray[i]
      });
    }
    
    topKIndices.dispose();
    topKValues.dispose();
    
    return results;
  }
}

export default ImageClassifier;
```

### **PoseNet para Detección de Poses**

```javascript
// PoseDetector.js
import * as tf from '@tensorflow/tfjs';
import * as posenet from '@tensorflow-models/posenet';

class PoseDetector {
  constructor() {
    this.model = null;
  }

  async loadPoseNet() {
    try {
      this.model = await posenet.load({
        architecture: 'MobileNetV1',
        outputStride: 16,
        inputResolution: { width: 640, height: 480 },
        multiplier: 0.75
      });
      
      console.log('PoseNet cargado exitosamente');
    } catch (error) {
      console.error('Error cargando PoseNet:', error);
    }
  }

  async detectPose(imageElement) {
    if (!this.model) {
      throw new Error('Modelo no cargado');
    }

    try {
      const pose = await this.model.estimateSinglePose(imageElement, {
        flipHorizontal: false,
        decodingMethod: 'single-person'
      });
      
      return pose;
    } catch (error) {
      console.error('Error detectando pose:', error);
      throw error;
    }
  }

  async detectMultiplePoses(imageElement) {
    if (!this.model) {
      throw new Error('Modelo no cargado');
    }

    try {
      const poses = await this.model.estimateMultiplePoses(imageElement, {
        flipHorizontal: false,
        decodingMethod: 'multi-person',
        maxDetections: 5,
        scoreThreshold: 0.5,
        nmsRadius: 20
      });
      
      return poses;
    } catch (error) {
      console.error('Error detectando múltiples poses:', error);
      throw error;
    }
  }
}

export default PoseDetector;
```

---

## 🎨 Modelos Personalizados

### **Creación de Modelo Simple**

```javascript
// CustomModel.js
import * as tf from '@tensorflow/tfjs';

class CustomModel {
  constructor() {
    this.model = null;
  }

  createSimpleModel() {
    // Crear modelo secuencial
    this.model = tf.sequential({
      layers: [
        // Capa de entrada
        tf.layers.dense({
          inputShape: [784], // 28x28 píxeles
          units: 128,
          activation: 'relu'
        }),
        
        // Capa oculta
        tf.layers.dense({
          units: 64,
          activation: 'relu'
        }),
        
        // Capa de salida
        tf.layers.dense({
          units: 10, // 10 clases
          activation: 'softmax'
        })
      ]
    });

    // Compilar modelo
    this.model.compile({
      optimizer: 'adam',
      loss: 'categoricalCrossentropy',
      metrics: ['accuracy']
    });

    console.log('Modelo personalizado creado');
    return this.model;
  }

  async trainModel(trainingData, labels) {
    if (!this.model) {
      throw new Error('Modelo no creado');
    }

    try {
      // Convertir datos a tensores
      const xs = tf.tensor2d(trainingData);
      const ys = tf.oneHot(tf.tensor1d(labels, 'int32'), 10);

      // Entrenar modelo
      const history = await this.model.fit(xs, ys, {
        epochs: 10,
        batchSize: 32,
        validationSplit: 0.2,
        callbacks: {
          onEpochEnd: (epoch, logs) => {
            console.log(`Epoch ${epoch}: loss = ${logs.loss.toFixed(4)}, accuracy = ${logs.acc.toFixed(4)}`);
          }
        }
      });

      // Limpiar tensores
      xs.dispose();
      ys.dispose();

      return history;
    } catch (error) {
      console.error('Error entrenando modelo:', error);
      throw error;
    }
  }

  async predict(inputData) {
    if (!this.model) {
      throw new Error('Modelo no creado');
    }

    try {
      const input = tf.tensor2d([inputData]);
      const prediction = await this.model.predict(input);
      const result = await prediction.data();
      
      input.dispose();
      prediction.dispose();
      
      return result;
    } catch (error) {
      console.error('Error en predicción:', error);
      throw error;
    }
  }

  async saveModel(path) {
    if (!this.model) {
      throw new Error('Modelo no creado');
    }

    try {
      await this.model.save(`file://${path}`);
      console.log('Modelo guardado exitosamente');
    } catch (error) {
      console.error('Error guardando modelo:', error);
      throw error;
    }
  }
}

export default CustomModel;
```

### **Transfer Learning**

```javascript
// TransferLearning.js
import * as tf from '@tensorflow/tfjs';

class TransferLearning {
  constructor() {
    this.baseModel = null;
    this.model = null;
  }

  async loadBaseModel() {
    try {
      // Cargar MobileNet como modelo base
      this.baseModel = await tf.loadLayersModel(
        'https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_0.25_224/model.json'
      );
      
      // Congelar capas del modelo base
      this.baseModel.trainable = false;
      
      console.log('Modelo base cargado');
    } catch (error) {
      console.error('Error cargando modelo base:', error);
    }
  }

  createTransferModel(numClasses) {
    if (!this.baseModel) {
      throw new Error('Modelo base no cargado');
    }

    // Crear nuevo modelo con capas adicionales
    this.model = tf.sequential({
      layers: [
        // Usar modelo base (sin la última capa)
        ...this.baseModel.layers.slice(0, -1),
        
        // Agregar capas personalizadas
        tf.layers.globalAveragePooling2d(),
        tf.layers.dropout({ rate: 0.5 }),
        tf.layers.dense({
          units: numClasses,
          activation: 'softmax'
        })
      ]
    });

    // Compilar modelo
    this.model.compile({
      optimizer: tf.train.adam(0.001),
      loss: 'categoricalCrossentropy',
      metrics: ['accuracy']
    });

    console.log('Modelo de transfer learning creado');
    return this.model;
  }

  async fineTune(trainingData, labels, epochs = 10) {
    if (!this.model) {
      throw new Error('Modelo no creado');
    }

    try {
      const xs = tf.tensor4d(trainingData);
      const ys = tf.oneHot(tf.tensor1d(labels, 'int32'), labels.length);

      const history = await this.model.fit(xs, ys, {
        epochs,
        batchSize: 16,
        validationSplit: 0.2,
        callbacks: {
          onEpochEnd: (epoch, logs) => {
            console.log(`Epoch ${epoch}: loss = ${logs.loss.toFixed(4)}, accuracy = ${logs.acc.toFixed(4)}`);
          }
        }
      });

      xs.dispose();
      ys.dispose();

      return history;
    } catch (error) {
      console.error('Error en fine-tuning:', error);
      throw error;
    }
  }
}

export default TransferLearning;
```

---

## ⚡ Optimización para Dispositivos Móviles

### **Quantización de Modelos**

```javascript
// ModelOptimizer.js
import * as tf from '@tensorflow/tfjs';

class ModelOptimizer {
  constructor() {
    this.originalModel = null;
    this.optimizedModel = null;
  }

  async quantizeModel(model) {
    try {
      // Quantización post-entrenamiento
      const quantizedModel = await tf.quantization.quantizeModel(model, {
        weightBits: 8,
        activationBits: 8
      });

      console.log('Modelo quantizado exitosamente');
      return quantizedModel;
    } catch (error) {
      console.error('Error quantizando modelo:', error);
      throw error;
    }
  }

  async pruneModel(model, sparsity = 0.5) {
    try {
      // Pruning para reducir tamaño
      const prunedModel = await tf.pruning.pruneModel(model, {
        sparsity,
        beginStep: 0,
        endStep: 1000,
        frequency: 100
      });

      console.log('Modelo podado exitosamente');
      return prunedModel;
    } catch (error) {
      console.error('Error podando modelo:', error);
      throw error;
    }
  }

  async optimizeForMobile(model) {
    try {
      // Optimizaciones múltiples
      const optimizedModel = await tf.graphModel.optimizeForMobile(model, {
        quantization: true,
        pruning: true,
        sparsity: 0.3
      });

      console.log('Modelo optimizado para móvil');
      return optimizedModel;
    } catch (error) {
      console.error('Error optimizando modelo:', error);
      throw error;
    }
  }
}

export default ModelOptimizer;
```

### **Gestión de Memoria**

```javascript
// MemoryManager.js
import * as tf from '@tensorflow/tfjs';

class MemoryManager {
  constructor() {
    this.tensors = new Set();
  }

  trackTensor(tensor) {
    this.tensors.add(tensor);
    return tensor;
  }

  disposeTensor(tensor) {
    if (tensor && !tensor.isDisposed) {
      tensor.dispose();
      this.tensors.delete(tensor);
    }
  }

  disposeAllTensors() {
    this.tensors.forEach(tensor => {
      if (!tensor.isDisposed) {
        tensor.dispose();
      }
    });
    this.tensors.clear();
  }

  getMemoryInfo() {
    return {
      numTensors: tf.memory().numTensors,
      numBytes: tf.memory().numBytes,
      numDataBuffers: tf.memory().numDataBuffers
    };
  }

  cleanup() {
    // Limpiar memoria no utilizada
    tf.disposeVariables();
    this.disposeAllTensors();
  }
}

export default MemoryManager;
```

---

## 🎯 Implementación Práctica

### **Componente de Clasificación de Imágenes**

```javascript
// ImageClassificationScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Image,
  StyleSheet,
  Alert,
  ActivityIndicator
} from 'react-native';
import ImagePicker from 'react-native-image-picker';
import ImageClassifier from './ImageClassifier';

const ImageClassificationScreen = () => {
  const [classifier, setClassifier] = useState(null);
  const [imageUri, setImageUri] = useState(null);
  const [predictions, setPredictions] = useState([]);
  const [loading, setLoading] = useState(false);
  const [modelLoaded, setModelLoaded] = useState(false);

  useEffect(() => {
    initializeClassifier();
  }, []);

  const initializeClassifier = async () => {
    try {
      setLoading(true);
      const newClassifier = new ImageClassifier();
      await newClassifier.loadMobileNet();
      setClassifier(newClassifier);
      setModelLoaded(true);
    } catch (error) {
      Alert.alert('Error', 'No se pudo cargar el modelo');
    } finally {
      setLoading(false);
    }
  };

  const selectImage = () => {
    const options = {
      title: 'Seleccionar Imagen',
      storageOptions: {
        skipBackup: true,
        path: 'images',
      },
    };

    ImagePicker.showImagePicker(options, (response) => {
      if (response.didCancel || response.error) {
        return;
      }

      setImageUri(response.uri);
      setPredictions([]);
    });
  };

  const classifyImage = async () => {
    if (!classifier || !imageUri) {
      Alert.alert('Error', 'Selecciona una imagen primero');
      return;
    }

    try {
      setLoading(true);
      const results = await classifier.classifyImage(imageUri);
      setPredictions(results);
    } catch (error) {
      Alert.alert('Error', 'No se pudo clasificar la imagen');
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Clasificación de Imágenes</Text>
      
      {loading && <ActivityIndicator size="large" />}
      
      {!modelLoaded && (
        <Text style={styles.status}>Cargando modelo...</Text>
      )}

      {imageUri && (
        <Image source={{ uri: imageUri }} style={styles.image} />
      )}

      <TouchableOpacity style={styles.button} onPress={selectImage}>
        <Text style={styles.buttonText}>Seleccionar Imagen</Text>
      </TouchableOpacity>

      {imageUri && (
        <TouchableOpacity style={styles.button} onPress={classifyImage}>
          <Text style={styles.buttonText}>Clasificar</Text>
        </TouchableOpacity>
      )}

      {predictions.length > 0 && (
        <View style={styles.predictions}>
          <Text style={styles.predictionsTitle}>Predicciones:</Text>
          {predictions.map((prediction, index) => (
            <Text key={index} style={styles.prediction}>
              {prediction.label}: {(prediction.confidence * 100).toFixed(2)}%
            </Text>
          ))}
        </View>
      )}
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
  status: {
    textAlign: 'center',
    marginBottom: 20,
    color: '#666',
  },
  image: {
    width: 200,
    height: 200,
    alignSelf: 'center',
    marginBottom: 20,
    borderRadius: 10,
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
  predictions: {
    marginTop: 20,
    padding: 15,
    backgroundColor: 'white',
    borderRadius: 10,
  },
  predictionsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  prediction: {
    fontSize: 16,
    marginBottom: 5,
  },
});

export default ImageClassificationScreen;
```

---

## 🧪 Testing y Debugging

### **Testing de Modelos**

```javascript
// ModelTester.js
import * as tf from '@tensorflow/tfjs';

class ModelTester {
  constructor() {
    this.testResults = [];
  }

  async testModelAccuracy(model, testData, testLabels) {
    try {
      const predictions = await model.predict(testData);
      const predictedLabels = await predictions.argMax(1).data();
      
      let correct = 0;
      const total = testLabels.length;
      
      for (let i = 0; i < total; i++) {
        if (predictedLabels[i] === testLabels[i]) {
          correct++;
        }
      }
      
      const accuracy = correct / total;
      
      this.testResults.push({
        test: 'accuracy',
        result: accuracy,
        timestamp: new Date()
      });
      
      predictions.dispose();
      
      return accuracy;
    } catch (error) {
      console.error('Error en test de precisión:', error);
      throw error;
    }
  }

  async testModelPerformance(model, inputData, iterations = 100) {
    try {
      const startTime = Date.now();
      
      for (let i = 0; i < iterations; i++) {
        const prediction = await model.predict(inputData);
        prediction.dispose();
      }
      
      const endTime = Date.now();
      const avgTime = (endTime - startTime) / iterations;
      
      this.testResults.push({
        test: 'performance',
        result: avgTime,
        iterations,
        timestamp: new Date()
      });
      
      return avgTime;
    } catch (error) {
      console.error('Error en test de rendimiento:', error);
      throw error;
    }
  }

  getTestResults() {
    return this.testResults;
  }

  clearResults() {
    this.testResults = [];
  }
}

export default ModelTester;
```

---

## 📊 Métricas y Monitoreo

### **Sistema de Métricas**

```javascript
// MetricsCollector.js
class MetricsCollector {
  constructor() {
    this.metrics = {
      inferenceTime: [],
      memoryUsage: [],
      accuracy: [],
      errors: []
    };
  }

  recordInferenceTime(time) {
    this.metrics.inferenceTime.push({
      time,
      timestamp: Date.now()
    });
  }

  recordMemoryUsage() {
    const memoryInfo = tf.memory();
    this.metrics.memoryUsage.push({
      numTensors: memoryInfo.numTensors,
      numBytes: memoryInfo.numBytes,
      timestamp: Date.now()
    });
  }

  recordAccuracy(accuracy) {
    this.metrics.accuracy.push({
      accuracy,
      timestamp: Date.now()
    });
  }

  recordError(error) {
    this.metrics.errors.push({
      error: error.message,
      timestamp: Date.now()
    });
  }

  getAverageInferenceTime() {
    if (this.metrics.inferenceTime.length === 0) return 0;
    
    const total = this.metrics.inferenceTime.reduce((sum, item) => sum + item.time, 0);
    return total / this.metrics.inferenceTime.length;
  }

  getAverageAccuracy() {
    if (this.metrics.accuracy.length === 0) return 0;
    
    const total = this.metrics.accuracy.reduce((sum, item) => sum + item.accuracy, 0);
    return total / this.metrics.accuracy.length;
  }

  getMetrics() {
    return {
      ...this.metrics,
      averageInferenceTime: this.getAverageInferenceTime(),
      averageAccuracy: this.getAverageAccuracy()
    };
  }
}

export default MetricsCollector;
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Clasificador de Emociones**

```javascript
// Ejercicio: Crear un clasificador de emociones
// 1. Cargar modelo pre-entrenado
// 2. Preprocesar imagen de rostro
// 3. Clasificar emoción
// 4. Mostrar resultado con confianza

const emotionClassifier = {
  async classifyEmotion(imageUri) {
    // Implementar clasificación de emociones
    // Retornar: { emotion: 'happy', confidence: 0.85 }
  }
};
```

### **Ejercicio 2: Detector de Objetos**

```javascript
// Ejercicio: Crear detector de objetos
// 1. Cargar modelo de detección
// 2. Procesar imagen
// 3. Detectar objetos con bounding boxes
// 4. Mostrar resultados visualmente

const objectDetector = {
  async detectObjects(imageUri) {
    // Implementar detección de objetos
    // Retornar: [{ object: 'car', confidence: 0.9, bbox: {...} }]
  }
};
```

---

## 🚀 Próximos Pasos

### **Lo que Aprendiste**

1. **Configuración** de TensorFlow Lite en React Native
2. **Integración** de modelos pre-entrenados
3. **Creación** de modelos personalizados
4. **Optimización** para dispositivos móviles
5. **Implementación** de inferencia en tiempo real

### **Preparación para la Siguiente Clase**

En la próxima clase aprenderás sobre:
- **Computer Vision** avanzada
- **Reconocimiento de objetos** y clasificación
- **Detección facial** y análisis de emociones
- **OCR** y procesamiento de texto
- **Segmentación** de imágenes

### **Tarea para Casa**

1. **Implementar** un clasificador de imágenes personalizado
2. **Optimizar** un modelo para tu dispositivo
3. **Crear** una aplicación que use TensorFlow Lite
4. **Documentar** el rendimiento y métricas

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [TensorFlow Lite](https://tensorflow.org/lite)
- [TensorFlow.js](https://tensorflow.org/js)
- [React Native TensorFlow](https://github.com/tensorflow/tfjs/tree/master/tfjs-react-native)

### **Modelos Pre-entrenados**
- [TensorFlow Hub](https://tfhub.dev/)
- [Model Zoo](https://github.com/tensorflow/models)
- [MediaPipe Models](https://google.github.io/mediapipe/solutions/models.html)

### **Herramientas de Desarrollo**
- [TensorBoard](https://tensorflow.org/tensorboard)
- [Model Converter](https://tensorflow.org/lite/convert)
- [Benchmark Tool](https://tensorflow.org/lite/performance/benchmarks)

---

**🎯 Objetivo de la Clase**: Dominar la integración de TensorFlow Lite en React Native, desde la configuración básica hasta la implementación de modelos complejos optimizados para dispositivos móviles.

**💡 Consejo**: Comienza siempre con modelos pre-entrenados y optimízalos gradualmente. La cuantización y el pruning son esenciales para el rendimiento en móviles.

---

**🚀 ¡Has completado la Clase 1 de Machine Learning e IA! Continúa con la Clase 2 para dominar la visión por computadora.**
