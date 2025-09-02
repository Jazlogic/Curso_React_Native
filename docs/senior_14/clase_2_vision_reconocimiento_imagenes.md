# 👁️ Clase 2: Visión y Reconocimiento de Imágenes

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar capacidades avanzadas de visión por computadora en React Native, incluyendo reconocimiento de objetos, detección facial, análisis de emociones, OCR y segmentación de imágenes.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** reconocimiento de objetos y clasificación
2. **Crear** sistemas de detección facial
3. **Desarrollar** análisis de emociones
4. **Implementar** OCR y procesamiento de texto
5. **Crear** segmentación de imágenes

---

## 🛠️ Configuración de Computer Vision

### **Instalación de Dependencias**

```bash
# Instalar librerías de visión por computadora
npm install @tensorflow-models/mobilenet @tensorflow-models/coco-ssd
npm install @tensorflow-models/posenet @tensorflow-models/face-detection
npm install @tensorflow-models/face-landmarks-detection

# Instalar dependencias de cámara
npm install react-native-vision-camera react-native-image-picker
npm install react-native-permissions

# Para iOS
cd ios && pod install
```

### **Configuración de Permisos**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

```xml
<!-- ios/YourApp/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>Esta app necesita acceso a la cámara para reconocimiento de imágenes</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Esta app necesita acceso a la galería para seleccionar imágenes</string>
```

---

## 🎯 Reconocimiento de Objetos

### **Implementación con COCO-SSD**

```javascript
// ObjectDetector.js
import * as tf from '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';

class ObjectDetector {
  constructor() {
    this.model = null;
    this.isLoaded = false;
  }

  async loadModel() {
    try {
      this.model = await cocoSsd.load();
      this.isLoaded = true;
      console.log('Modelo COCO-SSD cargado exitosamente');
    } catch (error) {
      console.error('Error cargando modelo COCO-SSD:', error);
      throw error;
    }
  }

  async detectObjects(imageElement) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      const predictions = await this.model.detect(imageElement);
      
      // Filtrar predicciones con confianza alta
      const filteredPredictions = predictions.filter(
        prediction => prediction.score > 0.5
      );

      return filteredPredictions.map(prediction => ({
        object: prediction.class,
        confidence: prediction.score,
        bbox: {
          x: prediction.bbox[0],
          y: prediction.bbox[1],
          width: prediction.bbox[2],
          height: prediction.bbox[3]
        }
      }));
    } catch (error) {
      console.error('Error detectando objetos:', error);
      throw error;
    }
  }

  async detectObjectsFromImage(imageUri) {
    try {
      // Cargar imagen desde URI
      const imageElement = await this.loadImageFromUri(imageUri);
      const objects = await this.detectObjects(imageElement);
      
      // Limpiar imagen
      if (imageElement.dispose) {
        imageElement.dispose();
      }
      
      return objects;
    } catch (error) {
      console.error('Error detectando objetos desde imagen:', error);
      throw error;
    }
  }

  async loadImageFromUri(uri) {
    return new Promise((resolve, reject) => {
      const img = new Image();
      img.crossOrigin = 'anonymous';
      img.onload = () => resolve(img);
      img.onerror = reject;
      img.src = uri;
    });
  }
}

export default ObjectDetector;
```

### **Visualización de Detecciones**

```javascript
// ObjectDetectionVisualizer.js
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

class ObjectDetectionVisualizer extends React.Component {
  render() {
    const { detections, imageWidth, imageHeight } = this.props;

    return (
      <View style={styles.container}>
        {detections.map((detection, index) => (
          <View
            key={index}
            style={[
              styles.detectionBox,
              {
                left: detection.bbox.x,
                top: detection.bbox.y,
                width: detection.bbox.width,
                height: detection.bbox.height,
              }
            ]}
          >
            <Text style={styles.detectionLabel}>
              {detection.object} ({(detection.confidence * 100).toFixed(1)}%)
            </Text>
          </View>
        ))}
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
  },
  detectionBox: {
    position: 'absolute',
    borderWidth: 2,
    borderColor: '#00FF00',
    backgroundColor: 'rgba(0, 255, 0, 0.1)',
  },
  detectionLabel: {
    position: 'absolute',
    top: -20,
    left: 0,
    backgroundColor: '#00FF00',
    color: 'black',
    padding: 2,
    fontSize: 12,
    fontWeight: 'bold',
  },
});

export default ObjectDetectionVisualizer;
```

---

## 👤 Detección Facial

### **Implementación con Face Detection**

```javascript
// FaceDetector.js
import * as faceDetection from '@tensorflow-models/face-detection';

class FaceDetector {
  constructor() {
    this.model = null;
    this.isLoaded = false;
  }

  async loadModel() {
    try {
      this.model = await faceDetection.createDetector(
        faceDetection.SupportedModels.MediaPipeFaceDetector,
        {
          runtime: 'tfjs',
          maxFaces: 10,
          detectorModelUrl: 'https://tfhub.dev/mediapipe/tfjs-model/face_detection_short_range/1/default/1'
        }
      );
      
      this.isLoaded = true;
      console.log('Modelo de detección facial cargado exitosamente');
    } catch (error) {
      console.error('Error cargando modelo de detección facial:', error);
      throw error;
    }
  }

  async detectFaces(imageElement) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      const faces = await this.model.estimateFaces(imageElement);
      
      return faces.map(face => ({
        bbox: {
          x: face.box.xCenter - face.box.width / 2,
          y: face.box.yCenter - face.box.height / 2,
          width: face.box.width,
          height: face.box.height
        },
        landmarks: face.landmarks,
        score: face.score
      }));
    } catch (error) {
      console.error('Error detectando caras:', error);
      throw error;
    }
  }

  async detectFacesFromImage(imageUri) {
    try {
      const imageElement = await this.loadImageFromUri(imageUri);
      const faces = await this.detectFaces(imageElement);
      
      if (imageElement.dispose) {
        imageElement.dispose();
      }
      
      return faces;
    } catch (error) {
      console.error('Error detectando caras desde imagen:', error);
      throw error;
    }
  }

  async loadImageFromUri(uri) {
    return new Promise((resolve, reject) => {
      const img = new Image();
      img.crossOrigin = 'anonymous';
      img.onload = () => resolve(img);
      img.onerror = reject;
      img.src = uri;
    });
  }
}

export default FaceDetector;
```

### **Análisis de Emociones**

```javascript
// EmotionAnalyzer.js
import * as tf from '@tensorflow/tfjs';
import * as faceLandmarksDetection from '@tensorflow-models/face-landmarks-detection';

class EmotionAnalyzer {
  constructor() {
    this.faceModel = null;
    this.emotionModel = null;
    this.isLoaded = false;
  }

  async loadModels() {
    try {
      // Cargar modelo de landmarks faciales
      this.faceModel = await faceLandmarksDetection.createDetector(
        faceLandmarksDetection.SupportedModels.MediaPipeFaceMesh,
        {
          runtime: 'tfjs',
          refineLandmarks: true,
          maxFaces: 1
        }
      );

      // Cargar modelo de emociones
      this.emotionModel = await tf.loadLayersModel(
        'https://tfhub.dev/tensorflow/tfjs-model/emotion-detection/1/default/1'
      );

      this.isLoaded = true;
      console.log('Modelos de análisis de emociones cargados');
    } catch (error) {
      console.error('Error cargando modelos de emociones:', error);
      throw error;
    }
  }

  async analyzeEmotion(imageElement) {
    if (!this.isLoaded) {
      throw new Error('Modelos no cargados');
    }

    try {
      // Detectar landmarks faciales
      const faces = await this.faceModel.estimateFaces(imageElement);
      
      if (faces.length === 0) {
        return { emotion: 'no_face', confidence: 0 };
      }

      const face = faces[0];
      
      // Extraer región facial
      const faceRegion = this.extractFaceRegion(imageElement, face);
      
      // Analizar emoción
      const emotion = await this.predictEmotion(faceRegion);
      
      // Limpiar tensores
      if (faceRegion.dispose) {
        faceRegion.dispose();
      }
      
      return emotion;
    } catch (error) {
      console.error('Error analizando emoción:', error);
      throw error;
    }
  }

  extractFaceRegion(imageElement, face) {
    const bbox = face.boundingBox;
    const x = Math.max(0, bbox.xCenter - bbox.width / 2);
    const y = Math.max(0, bbox.yCenter - bbox.height / 2);
    const width = Math.min(bbox.width, imageElement.width - x);
    const height = Math.min(bbox.height, imageElement.height - y);

    // Crear canvas para extraer región facial
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    canvas.width = 48; // Tamaño esperado por el modelo
    canvas.height = 48;
    
    ctx.drawImage(
      imageElement,
      x, y, width, height,
      0, 0, 48, 48
    );

    return tf.browser.fromPixels(canvas);
  }

  async predictEmotion(faceTensor) {
    try {
      // Preprocesar tensor
      const normalized = faceTensor.div(255.0);
      const batched = normalized.expandDims(0);
      
      // Predecir emoción
      const prediction = await this.emotionModel.predict(batched);
      const probabilities = await prediction.data();
      
      // Mapear a emociones
      const emotions = ['angry', 'disgust', 'fear', 'happy', 'sad', 'surprise', 'neutral'];
      const maxIndex = probabilities.indexOf(Math.max(...probabilities));
      
      // Limpiar tensores
      normalized.dispose();
      batched.dispose();
      prediction.dispose();
      
      return {
        emotion: emotions[maxIndex],
        confidence: probabilities[maxIndex]
      };
    } catch (error) {
      console.error('Error prediciendo emoción:', error);
      throw error;
    }
  }
}

export default EmotionAnalyzer;
```

---

## 📝 OCR y Procesamiento de Texto

### **Implementación de OCR**

```javascript
// OCRProcessor.js
import * as tf from '@tensorflow/tfjs';
import * as tfText from '@tensorflow/tfjs-text';

class OCRProcessor {
  constructor() {
    this.model = null;
    this.isLoaded = false;
  }

  async loadModel() {
    try {
      // Cargar modelo de OCR
      this.model = await tf.loadLayersModel(
        'https://tfhub.dev/tensorflow/tfjs-model/ocr/1/default/1'
      );
      
      this.isLoaded = true;
      console.log('Modelo de OCR cargado exitosamente');
    } catch (error) {
      console.error('Error cargando modelo de OCR:', error);
      throw error;
    }
  }

  async extractText(imageElement) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Preprocesar imagen
      const processedImage = await this.preprocessImage(imageElement);
      
      // Extraer texto
      const textRegions = await this.detectTextRegions(processedImage);
      const extractedText = await this.recognizeText(processedImage, textRegions);
      
      // Limpiar tensores
      if (processedImage.dispose) {
        processedImage.dispose();
      }
      
      return extractedText;
    } catch (error) {
      console.error('Error extrayendo texto:', error);
      throw error;
    }
  }

  async preprocessImage(imageElement) {
    try {
      // Convertir a tensor
      const tensor = tf.browser.fromPixels(imageElement);
      
      // Redimensionar
      const resized = tf.image.resizeBilinear(tensor, [512, 512]);
      
      // Normalizar
      const normalized = resized.div(255.0);
      
      // Limpiar tensor original
      tensor.dispose();
      resized.dispose();
      
      return normalized;
    } catch (error) {
      console.error('Error preprocesando imagen:', error);
      throw error;
    }
  }

  async detectTextRegions(imageTensor) {
    try {
      // Detectar regiones de texto
      const textDetection = await this.model.predict(imageTensor);
      const regions = await textDetection.data();
      
      textDetection.dispose();
      
      return this.parseTextRegions(regions);
    } catch (error) {
      console.error('Error detectando regiones de texto:', error);
      throw error;
    }
  }

  parseTextRegions(regions) {
    const textRegions = [];
    
    // Procesar regiones detectadas
    for (let i = 0; i < regions.length; i += 4) {
      if (regions[i + 3] > 0.5) { // Confianza > 0.5
        textRegions.push({
          x: regions[i],
          y: regions[i + 1],
          width: regions[i + 2],
          height: regions[i + 3]
        });
      }
    }
    
    return textRegions;
  }

  async recognizeText(imageTensor, textRegions) {
    const extractedText = [];
    
    for (const region of textRegions) {
      try {
        // Extraer región de texto
        const textRegion = this.extractTextRegion(imageTensor, region);
        
        // Reconocer texto
        const text = await this.recognizeTextInRegion(textRegion);
        
        if (text) {
          extractedText.push({
            text,
            region,
            confidence: 0.8 // Placeholder
          });
        }
        
        // Limpiar tensor
        if (textRegion.dispose) {
          textRegion.dispose();
        }
      } catch (error) {
        console.error('Error reconociendo texto en región:', error);
      }
    }
    
    return extractedText;
  }

  extractTextRegion(imageTensor, region) {
    const x = Math.floor(region.x);
    const y = Math.floor(region.y);
    const width = Math.floor(region.width);
    const height = Math.floor(region.height);
    
    return imageTensor.slice([y, x, 0], [height, width, 3]);
  }

  async recognizeTextInRegion(textRegion) {
    try {
      // Redimensionar a tamaño estándar
      const resized = tf.image.resizeBilinear(textRegion, [32, 128]);
      
      // Normalizar
      const normalized = resized.div(255.0);
      
      // Agregar dimensión de batch
      const batched = normalized.expandDims(0);
      
      // Predecir texto
      const prediction = await this.model.predict(batched);
      const text = await this.decodeText(prediction);
      
      // Limpiar tensores
      resized.dispose();
      normalized.dispose();
      batched.dispose();
      prediction.dispose();
      
      return text;
    } catch (error) {
      console.error('Error reconociendo texto:', error);
      return null;
    }
  }

  async decodeText(prediction) {
    try {
      const probabilities = await prediction.data();
      
      // Decodificar secuencia de caracteres
      const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
      let text = '';
      
      for (let i = 0; i < probabilities.length; i += characters.length) {
        const charProbs = probabilities.slice(i, i + characters.length);
        const maxIndex = charProbs.indexOf(Math.max(...charProbs));
        
        if (maxIndex < characters.length) {
          text += characters[maxIndex];
        }
      }
      
      return text.trim();
    } catch (error) {
      console.error('Error decodificando texto:', error);
      return '';
    }
  }
}

export default OCRProcessor;
```

---

## 🎨 Segmentación de Imágenes

### **Implementación de Segmentación**

```javascript
// ImageSegmenter.js
import * as tf from '@tensorflow/tfjs';

class ImageSegmenter {
  constructor() {
    this.model = null;
    this.isLoaded = false;
  }

  async loadModel() {
    try {
      // Cargar modelo de segmentación
      this.model = await tf.loadLayersModel(
        'https://tfhub.dev/tensorflow/tfjs-model/deeplab/1/default/1'
      );
      
      this.isLoaded = true;
      console.log('Modelo de segmentación cargado exitosamente');
    } catch (error) {
      console.error('Error cargando modelo de segmentación:', error);
      throw error;
    }
  }

  async segmentImage(imageElement) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Preprocesar imagen
      const processedImage = await this.preprocessImage(imageElement);
      
      // Realizar segmentación
      const segmentation = await this.model.predict(processedImage);
      const mask = await this.createSegmentationMask(segmentation);
      
      // Limpiar tensores
      if (processedImage.dispose) {
        processedImage.dispose();
      }
      segmentation.dispose();
      
      return mask;
    } catch (error) {
      console.error('Error segmentando imagen:', error);
      throw error;
    }
  }

  async preprocessImage(imageElement) {
    try {
      // Convertir a tensor
      const tensor = tf.browser.fromPixels(imageElement);
      
      // Redimensionar a 513x513 (tamaño esperado por DeepLab)
      const resized = tf.image.resizeBilinear(tensor, [513, 513]);
      
      // Normalizar valores
      const normalized = resized.div(255.0);
      
      // Agregar dimensión de batch
      const batched = normalized.expandDims(0);
      
      // Limpiar tensores
      tensor.dispose();
      resized.dispose();
      normalized.dispose();
      
      return batched;
    } catch (error) {
      console.error('Error preprocesando imagen:', error);
      throw error;
    }
  }

  async createSegmentationMask(segmentation) {
    try {
      // Obtener predicciones
      const predictions = await segmentation.data();
      
      // Crear máscara de segmentación
      const mask = new Uint8Array(513 * 513 * 4); // RGBA
      
      for (let i = 0; i < 513 * 513; i++) {
        const classIndex = Math.round(predictions[i]);
        const color = this.getClassColor(classIndex);
        
        mask[i * 4] = color.r;     // Red
        mask[i * 4 + 1] = color.g; // Green
        mask[i * 4 + 2] = color.b; // Blue
        mask[i * 4 + 3] = 128;     // Alpha (semi-transparente)
      }
      
      return mask;
    } catch (error) {
      console.error('Error creando máscara de segmentación:', error);
      throw error;
    }
  }

  getClassColor(classIndex) {
    // Colores para diferentes clases de segmentación
    const colors = [
      { r: 0, g: 0, b: 0 },       // background
      { r: 128, g: 0, b: 0 },     // aeroplane
      { r: 0, g: 128, b: 0 },     // bicycle
      { r: 128, g: 128, b: 0 },   // bird
      { r: 0, g: 0, b: 128 },     // boat
      { r: 128, g: 0, b: 128 },   // bottle
      { r: 0, g: 128, b: 128 },   // bus
      { r: 128, g: 128, b: 128 }, // car
      { r: 64, g: 0, b: 0 },      // cat
      { r: 192, g: 0, b: 0 },     // chair
      { r: 64, g: 128, b: 0 },    // cow
      { r: 192, g: 128, b: 0 },   // dining table
      { r: 64, g: 0, b: 128 },    // dog
      { r: 192, g: 0, b: 128 },   // horse
      { r: 64, g: 128, b: 128 },  // motorbike
      { r: 192, g: 128, b: 128 }, // person
      { r: 0, g: 64, b: 0 },      // potted plant
      { r: 128, g: 64, b: 0 },    // sheep
      { r: 0, g: 192, b: 0 },     // sofa
      { r: 128, g: 192, b: 0 },   // train
      { r: 0, g: 64, b: 128 },    // tv/monitor
    ];
    
    return colors[classIndex] || { r: 0, g: 0, b: 0 };
  }
}

export default ImageSegmenter;
```

---

## 🎯 Implementación Práctica

### **Componente de Visión por Computadora**

```javascript
// ComputerVisionScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Image,
  StyleSheet,
  Alert,
  ActivityIndicator,
  ScrollView
} from 'react-native';
import ImagePicker from 'react-native-image-picker';
import ObjectDetector from './ObjectDetector';
import FaceDetector from './FaceDetector';
import EmotionAnalyzer from './EmotionAnalyzer';
import OCRProcessor from './OCRProcessor';

const ComputerVisionScreen = () => {
  const [imageUri, setImageUri] = useState(null);
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState({});
  
  // Modelos
  const [objectDetector, setObjectDetector] = useState(null);
  const [faceDetector, setFaceDetector] = useState(null);
  const [emotionAnalyzer, setEmotionAnalyzer] = useState(null);
  const [ocrProcessor, setOCRProcessor] = useState(null);

  useEffect(() => {
    initializeModels();
  }, []);

  const initializeModels = async () => {
    try {
      setLoading(true);
      
      // Inicializar todos los modelos
      const objDetector = new ObjectDetector();
      await objDetector.loadModel();
      setObjectDetector(objDetector);

      const fDetector = new FaceDetector();
      await fDetector.loadModel();
      setFaceDetector(fDetector);

      const eAnalyzer = new EmotionAnalyzer();
      await eAnalyzer.loadModels();
      setEmotionAnalyzer(eAnalyzer);

      const ocr = new OCRProcessor();
      await ocr.loadModel();
      setOCRProcessor(ocr);

    } catch (error) {
      Alert.alert('Error', 'No se pudieron cargar los modelos');
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
      setResults({});
    });
  };

  const analyzeImage = async () => {
    if (!imageUri) {
      Alert.alert('Error', 'Selecciona una imagen primero');
      return;
    }

    try {
      setLoading(true);
      const analysisResults = {};

      // Detectar objetos
      if (objectDetector) {
        try {
          const objects = await objectDetector.detectObjectsFromImage(imageUri);
          analysisResults.objects = objects;
        } catch (error) {
          console.error('Error detectando objetos:', error);
        }
      }

      // Detectar caras
      if (faceDetector) {
        try {
          const faces = await faceDetector.detectFacesFromImage(imageUri);
          analysisResults.faces = faces;
        } catch (error) {
          console.error('Error detectando caras:', error);
        }
      }

      // Analizar emociones
      if (emotionAnalyzer) {
        try {
          const emotion = await emotionAnalyzer.analyzeEmotion(imageUri);
          analysisResults.emotion = emotion;
        } catch (error) {
          console.error('Error analizando emociones:', error);
        }
      }

      // Extraer texto
      if (ocrProcessor) {
        try {
          const text = await ocrProcessor.extractText(imageUri);
          analysisResults.text = text;
        } catch (error) {
          console.error('Error extrayendo texto:', error);
        }
      }

      setResults(analysisResults);
    } catch (error) {
      Alert.alert('Error', 'No se pudo analizar la imagen');
    } finally {
      setLoading(false);
    }
  };

  const renderResults = () => {
    if (Object.keys(results).length === 0) {
      return null;
    }

    return (
      <ScrollView style={styles.resultsContainer}>
        {results.objects && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Objetos Detectados:</Text>
            {results.objects.map((obj, index) => (
              <Text key={index} style={styles.resultText}>
                {obj.object} ({(obj.confidence * 100).toFixed(1)}%)
              </Text>
            ))}
          </View>
        )}

        {results.faces && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Caras Detectadas:</Text>
            <Text style={styles.resultText}>
              {results.faces.length} cara(s) encontrada(s)
            </Text>
          </View>
        )}

        {results.emotion && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Emoción Detectada:</Text>
            <Text style={styles.resultText}>
              {results.emotion.emotion} ({(results.emotion.confidence * 100).toFixed(1)}%)
            </Text>
          </View>
        )}

        {results.text && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Texto Extraído:</Text>
            {results.text.map((text, index) => (
              <Text key={index} style={styles.resultText}>
                {text.text}
              </Text>
            ))}
          </View>
        )}
      </ScrollView>
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Visión por Computadora</Text>
      
      {loading && <ActivityIndicator size="large" />}
      
      {imageUri && (
        <Image source={{ uri: imageUri }} style={styles.image} />
      )}

      <TouchableOpacity style={styles.button} onPress={selectImage}>
        <Text style={styles.buttonText}>Seleccionar Imagen</Text>
      </TouchableOpacity>

      {imageUri && (
        <TouchableOpacity style={styles.button} onPress={analyzeImage}>
          <Text style={styles.buttonText}>Analizar Imagen</Text>
        </TouchableOpacity>
      )}

      {renderResults()}
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
  image: {
    width: 300,
    height: 300,
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
  resultsContainer: {
    marginTop: 20,
    maxHeight: 300,
  },
  resultSection: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  resultText: {
    fontSize: 16,
    marginBottom: 5,
    color: '#666',
  },
});

export default ComputerVisionScreen;
```

---

## 🧪 Testing y Optimización

### **Testing de Modelos de Visión**

```javascript
// VisionModelTester.js
class VisionModelTester {
  constructor() {
    this.testResults = [];
  }

  async testObjectDetection(detector, testImages) {
    const results = [];
    
    for (const image of testImages) {
      try {
        const startTime = Date.now();
        const detections = await detector.detectObjectsFromImage(image.uri);
        const endTime = Date.now();
        
        results.push({
          image: image.name,
          detections: detections.length,
          inferenceTime: endTime - startTime,
          accuracy: this.calculateAccuracy(detections, image.expectedObjects)
        });
      } catch (error) {
        results.push({
          image: image.name,
          error: error.message
        });
      }
    }
    
    this.testResults.push({
      test: 'object_detection',
      results,
      timestamp: new Date()
    });
    
    return results;
  }

  async testFaceDetection(detector, testImages) {
    const results = [];
    
    for (const image of testImages) {
      try {
        const startTime = Date.now();
        const faces = await detector.detectFacesFromImage(image.uri);
        const endTime = Date.now();
        
        results.push({
          image: image.name,
          faces: faces.length,
          inferenceTime: endTime - startTime,
          accuracy: this.calculateFaceAccuracy(faces, image.expectedFaces)
        });
      } catch (error) {
        results.push({
          image: image.name,
          error: error.message
        });
      }
    }
    
    this.testResults.push({
      test: 'face_detection',
      results,
      timestamp: new Date()
    });
    
    return results;
  }

  calculateAccuracy(detections, expected) {
    if (!expected || expected.length === 0) return 0;
    
    let correct = 0;
    for (const detection of detections) {
      if (expected.includes(detection.object)) {
        correct++;
      }
    }
    
    return correct / expected.length;
  }

  calculateFaceAccuracy(faces, expectedFaces) {
    if (!expectedFaces) return 0;
    
    const detected = faces.length;
    const expected = expectedFaces;
    
    return Math.min(detected, expected) / Math.max(detected, expected);
  }

  getTestResults() {
    return this.testResults;
  }
}

export default VisionModelTester;
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Detector de Productos**

```javascript
// Ejercicio: Crear un detector de productos para e-commerce
// 1. Detectar productos en imágenes
// 2. Clasificar por categoría
// 3. Extraer características visuales
// 4. Generar recomendaciones

const productDetector = {
  async detectProducts(imageUri) {
    // Implementar detección de productos
    // Retornar: [{ product: 'shirt', category: 'clothing', features: {...} }]
  }
};
```

### **Ejercicio 2: Sistema de Seguridad**

```javascript
// Ejercicio: Crear sistema de seguridad con reconocimiento facial
// 1. Detectar caras en tiempo real
// 2. Reconocer personas conocidas
// 3. Alertar sobre personas desconocidas
// 4. Registrar accesos

const securitySystem = {
  async recognizePerson(imageUri) {
    // Implementar reconocimiento facial
    // Retornar: { person: 'John Doe', confidence: 0.95, access: true }
  }
};
```

---

## 🚀 Próximos Pasos

### **Lo que Aprendiste**

1. **Implementación** de reconocimiento de objetos
2. **Detección facial** y análisis de emociones
3. **OCR** y procesamiento de texto
4. **Segmentación** de imágenes
5. **Optimización** para dispositivos móviles

### **Preparación para la Siguiente Clase**

En la próxima clase aprenderás sobre:
- **Natural Language Processing** en React Native
- **Análisis de sentimientos** y emociones
- **Clasificación de texto** y categorización
- **Traducción** automática
- **Chatbots** inteligentes

### **Tarea para Casa**

1. **Implementar** un detector de objetos personalizado
2. **Crear** un sistema de análisis de emociones
3. **Desarrollar** una aplicación de OCR
4. **Optimizar** el rendimiento de los modelos

---

## 📚 Recursos Adicionales

### **Modelos Pre-entrenados**
- [COCO Dataset](https://cocodataset.org/)
- [Face Detection Models](https://github.com/tensorflow/tfjs-models/tree/master/face-detection)
- [OCR Models](https://tfhub.dev/s?q=ocr)
- [Segmentation Models](https://tfhub.dev/s?q=segmentation)

### **Herramientas de Desarrollo**
- [TensorFlow.js Models](https://github.com/tensorflow/tfjs-models)
- [MediaPipe](https://mediapipe.dev/)
- [OpenCV.js](https://docs.opencv.org/3.4/d5/d10/tutorial_js_root.html)

### **Documentación**
- [Computer Vision Guide](https://tensorflow.org/js/guide/concepts)
- [Face Detection API](https://github.com/tensorflow/tfjs-models/tree/master/face-detection)
- [Object Detection API](https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd)

---

**🎯 Objetivo de la Clase**: Dominar la implementación de capacidades avanzadas de visión por computadora en React Native, desde detección de objetos hasta análisis de emociones.

**💡 Consejo**: Combina múltiples modelos de visión para crear aplicaciones más inteligentes. La optimización de rendimiento es crucial para la experiencia del usuario.

---

**🚀 ¡Has completado la Clase 2 de Visión por Computadora! Continúa con la Clase 3 para dominar el procesamiento de lenguaje natural.**
