# 💬 Clase 3: Natural Language Processing

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar capacidades avanzadas de procesamiento de lenguaje natural (NLP) en React Native, incluyendo análisis de sentimientos, clasificación de texto, traducción automática y creación de chatbots inteligentes.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** análisis de sentimientos y emociones
2. **Crear** sistemas de clasificación de texto
3. **Desarrollar** traducción automática
4. **Construir** chatbots inteligentes
5. **Optimizar** procesamiento de texto en tiempo real

---

## 🛠️ Configuración de NLP

### **Instalación de Dependencias**

```bash
# Instalar librerías de NLP
npm install @tensorflow/tfjs @tensorflow/tfjs-text
npm install natural compromise sentiment
npm install react-native-voice react-native-speech-to-text
npm install react-native-tts react-native-text-to-speech

# Instalar dependencias adicionales
npm install @react-native-async-storage/async-storage
npm install react-native-fs

# Para iOS
cd ios && pod install
```

### **Configuración de Permisos**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
```

```xml
<!-- ios/YourApp/Info.plist -->
<key>NSMicrophoneUsageDescription</key>
<string>Esta app necesita acceso al micrófono para reconocimiento de voz</string>
<key>NSSpeechRecognitionUsageDescription</key>
<string>Esta app necesita acceso al reconocimiento de voz</string>
```

---

## 📊 Análisis de Sentimientos

### **Implementación con Sentiment**

```javascript
// SentimentAnalyzer.js
import * as tf from '@tensorflow/tfjs';
import * as tfText from '@tensorflow/tfjs-text';

class SentimentAnalyzer {
  constructor() {
    this.model = null;
    this.tokenizer = null;
    this.isLoaded = false;
  }

  async loadModel() {
    try {
      // Cargar modelo de análisis de sentimientos
      this.model = await tf.loadLayersModel(
        'https://tfhub.dev/tensorflow/tfjs-model/sentiment/1/default/1'
      );
      
      // Cargar tokenizer
      this.tokenizer = await tfText.loadTokenizer(
        'https://tfhub.dev/tensorflow/tfjs-model/sentiment/1/default/1'
      );
      
      this.isLoaded = true;
      console.log('Modelo de análisis de sentimientos cargado');
    } catch (error) {
      console.error('Error cargando modelo de sentimientos:', error);
      throw error;
    }
  }

  async analyzeSentiment(text) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Tokenizar texto
      const tokens = await this.tokenizer.tokenize(text);
      
      // Convertir a tensor
      const inputTensor = tf.tensor2d([tokens]);
      
      // Predecir sentimiento
      const prediction = await this.model.predict(inputTensor);
      const probabilities = await prediction.data();
      
      // Interpretar resultados
      const sentiment = this.interpretSentiment(probabilities);
      
      // Limpiar tensores
      inputTensor.dispose();
      prediction.dispose();
      
      return sentiment;
    } catch (error) {
      console.error('Error analizando sentimiento:', error);
      throw error;
    }
  }

  interpretSentiment(probabilities) {
    const [negative, positive] = probabilities;
    
    if (positive > negative) {
      return {
        sentiment: 'positive',
        confidence: positive,
        score: positive - negative
      };
    } else {
      return {
        sentiment: 'negative',
        confidence: negative,
        score: negative - positive
      };
    }
  }

  async analyzeBatchTexts(texts) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      const results = [];
      
      for (const text of texts) {
        const sentiment = await this.analyzeSentiment(text);
        results.push({
          text,
          sentiment: sentiment.sentiment,
          confidence: sentiment.confidence,
          score: sentiment.score
        });
      }
      
      return results;
    } catch (error) {
      console.error('Error analizando batch de textos:', error);
      throw error;
    }
  }
}

export default SentimentAnalyzer;
```

### **Análisis Avanzado de Emociones**

```javascript
// EmotionAnalyzer.js
import * as tf from '@tensorflow/tfjs';

class EmotionAnalyzer {
  constructor() {
    this.model = null;
    this.isLoaded = false;
  }

  async loadModel() {
    try {
      // Cargar modelo de análisis de emociones
      this.model = await tf.loadLayersModel(
        'https://tfhub.dev/tensorflow/tfjs-model/emotion-detection/1/default/1'
      );
      
      this.isLoaded = true;
      console.log('Modelo de análisis de emociones cargado');
    } catch (error) {
      console.error('Error cargando modelo de emociones:', error);
      throw error;
    }
  }

  async analyzeEmotion(text) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Preprocesar texto
      const processedText = await this.preprocessText(text);
      
      // Convertir a tensor
      const inputTensor = tf.tensor2d([processedText]);
      
      // Predecir emoción
      const prediction = await this.model.predict(inputTensor);
      const probabilities = await prediction.data();
      
      // Interpretar emociones
      const emotion = this.interpretEmotion(probabilities);
      
      // Limpiar tensores
      inputTensor.dispose();
      prediction.dispose();
      
      return emotion;
    } catch (error) {
      console.error('Error analizando emoción:', error);
      throw error;
    }
  }

  async preprocessText(text) {
    // Limpiar y normalizar texto
    const cleaned = text
      .toLowerCase()
      .replace(/[^\w\s]/g, '')
      .replace(/\s+/g, ' ')
      .trim();
    
    // Convertir a secuencia de números
    const words = cleaned.split(' ');
    const wordToIndex = await this.getWordToIndex();
    
    return words.map(word => wordToIndex[word] || 0);
  }

  async getWordToIndex() {
    // Cargar vocabulario
    const response = await fetch('https://tfhub.dev/tensorflow/tfjs-model/emotion-detection/1/default/1/vocab.json');
    return await response.json();
  }

  interpretEmotion(probabilities) {
    const emotions = ['joy', 'sadness', 'anger', 'fear', 'surprise', 'disgust'];
    const maxIndex = probabilities.indexOf(Math.max(...probabilities));
    
    return {
      emotion: emotions[maxIndex],
      confidence: probabilities[maxIndex],
      allEmotions: emotions.map((emotion, index) => ({
        emotion,
        confidence: probabilities[index]
      }))
    };
  }
}

export default EmotionAnalyzer;
```

---

## 🏷️ Clasificación de Texto

### **Implementación de Clasificador**

```javascript
// TextClassifier.js
import * as tf from '@tensorflow/tfjs';
import * as tfText from '@tensorflow/tfjs-text';

class TextClassifier {
  constructor() {
    this.model = null;
    this.tokenizer = null;
    this.isLoaded = false;
    this.categories = [];
  }

  async loadModel(modelPath, categories) {
    try {
      // Cargar modelo de clasificación
      this.model = await tf.loadLayersModel(modelPath);
      
      // Cargar tokenizer
      this.tokenizer = await tfText.loadTokenizer(modelPath);
      
      this.categories = categories;
      this.isLoaded = true;
      console.log('Modelo de clasificación de texto cargado');
    } catch (error) {
      console.error('Error cargando modelo de clasificación:', error);
      throw error;
    }
  }

  async classifyText(text) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Tokenizar texto
      const tokens = await this.tokenizer.tokenize(text);
      
      // Convertir a tensor
      const inputTensor = tf.tensor2d([tokens]);
      
      // Predecir categoría
      const prediction = await this.model.predict(inputTensor);
      const probabilities = await prediction.data();
      
      // Interpretar resultados
      const classification = this.interpretClassification(probabilities);
      
      // Limpiar tensores
      inputTensor.dispose();
      prediction.dispose();
      
      return classification;
    } catch (error) {
      console.error('Error clasificando texto:', error);
      throw error;
    }
  }

  interpretClassification(probabilities) {
    const maxIndex = probabilities.indexOf(Math.max(...probabilities));
    
    return {
      category: this.categories[maxIndex],
      confidence: probabilities[maxIndex],
      allCategories: this.categories.map((category, index) => ({
        category,
        confidence: probabilities[index]
      }))
    };
  }

  async classifyBatchTexts(texts) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      const results = [];
      
      for (const text of texts) {
        const classification = await this.classifyText(text);
        results.push({
          text,
          category: classification.category,
          confidence: classification.confidence
        });
      }
      
      return results;
    } catch (error) {
      console.error('Error clasificando batch de textos:', error);
      throw error;
    }
  }
}

export default TextClassifier;
```

### **Clasificador de Spam**

```javascript
// SpamClassifier.js
import TextClassifier from './TextClassifier';

class SpamClassifier extends TextClassifier {
  constructor() {
    super();
    this.categories = ['ham', 'spam'];
  }

  async loadSpamModel() {
    try {
      await this.loadModel(
        'https://tfhub.dev/tensorflow/tfjs-model/spam-classification/1/default/1',
        this.categories
      );
      console.log('Modelo de clasificación de spam cargado');
    } catch (error) {
      console.error('Error cargando modelo de spam:', error);
      throw error;
    }
  }

  async isSpam(text) {
    try {
      const classification = await this.classifyText(text);
      return {
        isSpam: classification.category === 'spam',
        confidence: classification.confidence,
        spamScore: classification.allCategories.find(c => c.category === 'spam').confidence
      };
    } catch (error) {
      console.error('Error verificando spam:', error);
      throw error;
    }
  }

  async filterSpamMessages(messages) {
    try {
      const filteredMessages = [];
      
      for (const message of messages) {
        const spamCheck = await this.isSpam(message.text);
        
        if (!spamCheck.isSpam) {
          filteredMessages.push({
            ...message,
            spamScore: spamCheck.spamScore
          });
        }
      }
      
      return filteredMessages;
    } catch (error) {
      console.error('Error filtrando spam:', error);
      throw error;
    }
  }
}

export default SpamClassifier;
```

---

## 🌍 Traducción Automática

### **Implementación de Traductor**

```javascript
// Translator.js
import * as tf from '@tensorflow/tfjs';
import * as tfText from '@tensorflow/tfjs-text';

class Translator {
  constructor() {
    this.model = null;
    this.tokenizer = null;
    this.isLoaded = false;
    this.supportedLanguages = ['en', 'es', 'fr', 'de', 'it', 'pt'];
  }

  async loadModel(sourceLanguage, targetLanguage) {
    try {
      const modelPath = `https://tfhub.dev/tensorflow/tfjs-model/translation/${sourceLanguage}-${targetLanguage}/1/default/1`;
      
      // Cargar modelo de traducción
      this.model = await tf.loadLayersModel(modelPath);
      
      // Cargar tokenizer
      this.tokenizer = await tfText.loadTokenizer(modelPath);
      
      this.isLoaded = true;
      console.log(`Modelo de traducción ${sourceLanguage}-${targetLanguage} cargado`);
    } catch (error) {
      console.error('Error cargando modelo de traducción:', error);
      throw error;
    }
  }

  async translateText(text) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Tokenizar texto fuente
      const sourceTokens = await this.tokenizer.tokenize(text);
      
      // Convertir a tensor
      const inputTensor = tf.tensor2d([sourceTokens]);
      
      // Traducir
      const translation = await this.model.predict(inputTensor);
      const translatedTokens = await translation.data();
      
      // Decodificar traducción
      const translatedText = await this.decodeTranslation(translatedTokens);
      
      // Limpiar tensores
      inputTensor.dispose();
      translation.dispose();
      
      return translatedText;
    } catch (error) {
      console.error('Error traduciendo texto:', error);
      throw error;
    }
  }

  async decodeTranslation(tokens) {
    try {
      // Cargar vocabulario de destino
      const response = await fetch('https://tfhub.dev/tensorflow/tfjs-model/translation/vocab.json');
      const vocab = await response.json();
      
      // Convertir tokens a texto
      const words = tokens.map(token => vocab[token] || '');
      return words.join(' ').trim();
    } catch (error) {
      console.error('Error decodificando traducción:', error);
      return '';
    }
  }

  async translateBatchTexts(texts) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      const translations = [];
      
      for (const text of texts) {
        const translation = await this.translateText(text);
        translations.push({
          original: text,
          translated: translation
        });
      }
      
      return translations;
    } catch (error) {
      console.error('Error traduciendo batch de textos:', error);
      throw error;
    }
  }

  async detectLanguage(text) {
    try {
      // Implementar detección de idioma simple
      const languagePatterns = {
        'en': /[a-zA-Z]/g,
        'es': /[ñáéíóúü]/g,
        'fr': /[àâäéèêëïîôöùûüÿç]/g,
        'de': /[äöüß]/g,
        'it': /[àèéìíîòóù]/g,
        'pt': /[ãõç]/g
      };
      
      let maxScore = 0;
      let detectedLanguage = 'en';
      
      for (const [lang, pattern] of Object.entries(languagePatterns)) {
        const matches = text.match(pattern);
        const score = matches ? matches.length : 0;
        
        if (score > maxScore) {
          maxScore = score;
          detectedLanguage = lang;
        }
      }
      
      return {
        language: detectedLanguage,
        confidence: maxScore / text.length
      };
    } catch (error) {
      console.error('Error detectando idioma:', error);
      return { language: 'en', confidence: 0 };
    }
  }
}

export default Translator;
```

---

## 🤖 Chatbots Inteligentes

### **Implementación de Chatbot**

```javascript
// IntelligentChatbot.js
import * as tf from '@tensorflow/tfjs';
import * as tfText from '@tensorflow/tfjs-text';

class IntelligentChatbot {
  constructor() {
    this.model = null;
    this.tokenizer = null;
    this.isLoaded = false;
    this.conversationHistory = [];
    this.context = {};
  }

  async loadModel() {
    try {
      // Cargar modelo de chatbot
      this.model = await tf.loadLayersModel(
        'https://tfhub.dev/tensorflow/tfjs-model/chatbot/1/default/1'
      );
      
      // Cargar tokenizer
      this.tokenizer = await tfText.loadTokenizer(
        'https://tfhub.dev/tensorflow/tfjs-model/chatbot/1/default/1'
      );
      
      this.isLoaded = true;
      console.log('Modelo de chatbot cargado');
    } catch (error) {
      console.error('Error cargando modelo de chatbot:', error);
      throw error;
    }
  }

  async generateResponse(userMessage) {
    if (!this.isLoaded) {
      throw new Error('Modelo no cargado');
    }

    try {
      // Agregar mensaje del usuario al historial
      this.conversationHistory.push({
        role: 'user',
        message: userMessage,
        timestamp: Date.now()
      });

      // Preprocesar mensaje
      const processedMessage = await this.preprocessMessage(userMessage);
      
      // Generar respuesta
      const response = await this.generateBotResponse(processedMessage);
      
      // Agregar respuesta del bot al historial
      this.conversationHistory.push({
        role: 'bot',
        message: response,
        timestamp: Date.now()
      });

      return response;
    } catch (error) {
      console.error('Error generando respuesta:', error);
      throw error;
    }
  }

  async preprocessMessage(message) {
    try {
      // Limpiar mensaje
      const cleaned = message
        .toLowerCase()
        .replace(/[^\w\s]/g, '')
        .replace(/\s+/g, ' ')
        .trim();
      
      // Tokenizar
      const tokens = await this.tokenizer.tokenize(cleaned);
      
      return tokens;
    } catch (error) {
      console.error('Error preprocesando mensaje:', error);
      throw error;
    }
  }

  async generateBotResponse(processedMessage) {
    try {
      // Convertir a tensor
      const inputTensor = tf.tensor2d([processedMessage]);
      
      // Generar respuesta
      const response = await this.model.predict(inputTensor);
      const responseTokens = await response.data();
      
      // Decodificar respuesta
      const responseText = await this.decodeResponse(responseTokens);
      
      // Limpiar tensores
      inputTensor.dispose();
      response.dispose();
      
      return responseText;
    } catch (error) {
      console.error('Error generando respuesta del bot:', error);
      throw error;
    }
  }

  async decodeResponse(tokens) {
    try {
      // Cargar vocabulario
      const response = await fetch('https://tfhub.dev/tensorflow/tfjs-model/chatbot/1/default/1/vocab.json');
      const vocab = await response.json();
      
      // Convertir tokens a texto
      const words = tokens.map(token => vocab[token] || '');
      return words.join(' ').trim();
    } catch (error) {
      console.error('Error decodificando respuesta:', error);
      return 'Lo siento, no pude procesar tu mensaje.';
    }
  }

  async updateContext(userMessage, botResponse) {
    try {
      // Actualizar contexto basado en la conversación
      this.context = {
        ...this.context,
        lastUserMessage: userMessage,
        lastBotResponse: botResponse,
        conversationLength: this.conversationHistory.length,
        lastUpdate: Date.now()
      };
    } catch (error) {
      console.error('Error actualizando contexto:', error);
    }
  }

  getConversationHistory() {
    return this.conversationHistory;
  }

  clearHistory() {
    this.conversationHistory = [];
    this.context = {};
  }

  async getContextualResponse(userMessage) {
    try {
      // Analizar contexto de la conversación
      const contextAnalysis = await this.analyzeContext();
      
      // Generar respuesta contextual
      const response = await this.generateResponse(userMessage);
      
      // Actualizar contexto
      await this.updateContext(userMessage, response);
      
      return {
        response,
        context: contextAnalysis
      };
    } catch (error) {
      console.error('Error generando respuesta contextual:', error);
      throw error;
    }
  }

  async analyzeContext() {
    try {
      const recentMessages = this.conversationHistory.slice(-5);
      const userMessages = recentMessages.filter(msg => msg.role === 'user');
      
      return {
        conversationLength: this.conversationHistory.length,
        recentUserMessages: userMessages.length,
        lastMessageTime: recentMessages[recentMessages.length - 1]?.timestamp,
        context: this.context
      };
    } catch (error) {
      console.error('Error analizando contexto:', error);
      return {};
    }
  }
}

export default IntelligentChatbot;
```

---

## 🎯 Implementación Práctica

### **Componente de NLP**

```javascript
// NLPScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator,
  ScrollView,
  FlatList
} from 'react-native';
import SentimentAnalyzer from './SentimentAnalyzer';
import TextClassifier from './TextClassifier';
import Translator from './Translator';
import IntelligentChatbot from './IntelligentChatbot';

const NLPScreen = () => {
  const [inputText, setInputText] = useState('');
  const [results, setResults] = useState({});
  const [loading, setLoading] = useState(false);
  const [chatMessages, setChatMessages] = useState([]);
  
  // Modelos
  const [sentimentAnalyzer, setSentimentAnalyzer] = useState(null);
  const [textClassifier, setTextClassifier] = useState(null);
  const [translator, setTranslator] = useState(null);
  const [chatbot, setChatbot] = useState(null);

  useEffect(() => {
    initializeModels();
  }, []);

  const initializeModels = async () => {
    try {
      setLoading(true);
      
      // Inicializar analizador de sentimientos
      const sentiment = new SentimentAnalyzer();
      await sentiment.loadModel();
      setSentimentAnalyzer(sentiment);

      // Inicializar clasificador de texto
      const classifier = new TextClassifier();
      await classifier.loadModel(
        'https://tfhub.dev/tensorflow/tfjs-model/text-classification/1/default/1',
        ['positive', 'negative', 'neutral']
      );
      setTextClassifier(classifier);

      // Inicializar traductor
      const trans = new Translator();
      await trans.loadModel('en', 'es');
      setTranslator(trans);

      // Inicializar chatbot
      const bot = new IntelligentChatbot();
      await bot.loadModel();
      setChatbot(bot);

    } catch (error) {
      Alert.alert('Error', 'No se pudieron cargar los modelos');
    } finally {
      setLoading(false);
    }
  };

  const analyzeText = async () => {
    if (!inputText.trim()) {
      Alert.alert('Error', 'Ingresa un texto para analizar');
      return;
    }

    try {
      setLoading(true);
      const analysisResults = {};

      // Análisis de sentimientos
      if (sentimentAnalyzer) {
        try {
          const sentiment = await sentimentAnalyzer.analyzeSentiment(inputText);
          analysisResults.sentiment = sentiment;
        } catch (error) {
          console.error('Error analizando sentimiento:', error);
        }
      }

      // Clasificación de texto
      if (textClassifier) {
        try {
          const classification = await textClassifier.classifyText(inputText);
          analysisResults.classification = classification;
        } catch (error) {
          console.error('Error clasificando texto:', error);
        }
      }

      // Traducción
      if (translator) {
        try {
          const translation = await translator.translateText(inputText);
          analysisResults.translation = translation;
        } catch (error) {
          console.error('Error traduciendo texto:', error);
        }
      }

      setResults(analysisResults);
    } catch (error) {
      Alert.alert('Error', 'No se pudo analizar el texto');
    } finally {
      setLoading(false);
    }
  };

  const sendChatMessage = async () => {
    if (!inputText.trim() || !chatbot) {
      return;
    }

    try {
      setLoading(true);
      
      // Agregar mensaje del usuario
      const userMessage = {
        id: Date.now(),
        text: inputText,
        isUser: true,
        timestamp: new Date()
      };
      
      setChatMessages(prev => [...prev, userMessage]);
      
      // Generar respuesta del bot
      const botResponse = await chatbot.generateResponse(inputText);
      
      // Agregar respuesta del bot
      const botMessage = {
        id: Date.now() + 1,
        text: botResponse,
        isUser: false,
        timestamp: new Date()
      };
      
      setChatMessages(prev => [...prev, botMessage]);
      
      // Limpiar input
      setInputText('');
    } catch (error) {
      Alert.alert('Error', 'No se pudo enviar el mensaje');
    } finally {
      setLoading(false);
    }
  };

  const renderChatMessage = ({ item }) => (
    <View style={[
      styles.chatMessage,
      item.isUser ? styles.userMessage : styles.botMessage
    ]}>
      <Text style={[
        styles.messageText,
        item.isUser ? styles.userMessageText : styles.botMessageText
      ]}>
        {item.text}
      </Text>
      <Text style={styles.messageTime}>
        {item.timestamp.toLocaleTimeString()}
      </Text>
    </View>
  );

  const renderResults = () => {
    if (Object.keys(results).length === 0) {
      return null;
    }

    return (
      <ScrollView style={styles.resultsContainer}>
        {results.sentiment && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Análisis de Sentimientos:</Text>
            <Text style={styles.resultText}>
              Sentimiento: {results.sentiment.sentiment}
            </Text>
            <Text style={styles.resultText}>
              Confianza: {(results.sentiment.confidence * 100).toFixed(1)}%
            </Text>
            <Text style={styles.resultText}>
              Score: {results.sentiment.score.toFixed(3)}
            </Text>
          </View>
        )}

        {results.classification && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Clasificación:</Text>
            <Text style={styles.resultText}>
              Categoría: {results.classification.category}
            </Text>
            <Text style={styles.resultText}>
              Confianza: {(results.classification.confidence * 100).toFixed(1)}%
            </Text>
          </View>
        )}

        {results.translation && (
          <View style={styles.resultSection}>
            <Text style={styles.sectionTitle}>Traducción:</Text>
            <Text style={styles.resultText}>
              {results.translation}
            </Text>
          </View>
        )}
      </ScrollView>
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Procesamiento de Lenguaje Natural</Text>
      
      {loading && <ActivityIndicator size="large" />}
      
      <TextInput
        style={styles.textInput}
        placeholder="Ingresa un texto para analizar..."
        value={inputText}
        onChangeText={setInputText}
        multiline
        numberOfLines={4}
      />

      <TouchableOpacity style={styles.button} onPress={analyzeText}>
        <Text style={styles.buttonText}>Analizar Texto</Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.button} onPress={sendChatMessage}>
        <Text style={styles.buttonText}>Enviar al Chatbot</Text>
      </TouchableOpacity>

      {renderResults()}

      <View style={styles.chatContainer}>
        <Text style={styles.chatTitle}>Chatbot</Text>
        <FlatList
          data={chatMessages}
          renderItem={renderChatMessage}
          keyExtractor={(item) => item.id.toString()}
          style={styles.chatList}
        />
      </View>
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
  textInput: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 10,
    padding: 15,
    marginBottom: 15,
    backgroundColor: 'white',
    textAlignVertical: 'top',
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
    maxHeight: 200,
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
  chatContainer: {
    flex: 1,
    marginTop: 20,
  },
  chatTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  chatList: {
    flex: 1,
  },
  chatMessage: {
    padding: 10,
    marginBottom: 10,
    borderRadius: 10,
    maxWidth: '80%',
  },
  userMessage: {
    backgroundColor: '#007AFF',
    alignSelf: 'flex-end',
  },
  botMessage: {
    backgroundColor: '#E5E5EA',
    alignSelf: 'flex-start',
  },
  messageText: {
    fontSize: 16,
    marginBottom: 5,
  },
  userMessageText: {
    color: 'white',
  },
  botMessageText: {
    color: 'black',
  },
  messageTime: {
    fontSize: 12,
    color: '#666',
  },
});

export default NLPScreen;
```

---

## 🧪 Testing y Optimización

### **Testing de Modelos NLP**

```javascript
// NLPModelTester.js
class NLPModelTester {
  constructor() {
    this.testResults = [];
  }

  async testSentimentAnalysis(analyzer, testTexts) {
    const results = [];
    
    for (const testCase of testTexts) {
      try {
        const startTime = Date.now();
        const sentiment = await analyzer.analyzeSentiment(testCase.text);
        const endTime = Date.now();
        
        results.push({
          text: testCase.text,
          expected: testCase.expectedSentiment,
          predicted: sentiment.sentiment,
          confidence: sentiment.confidence,
          inferenceTime: endTime - startTime,
          correct: testCase.expectedSentiment === sentiment.sentiment
        });
      } catch (error) {
        results.push({
          text: testCase.text,
          error: error.message
        });
      }
    }
    
    this.testResults.push({
      test: 'sentiment_analysis',
      results,
      accuracy: this.calculateAccuracy(results),
      timestamp: new Date()
    });
    
    return results;
  }

  async testTextClassification(classifier, testTexts) {
    const results = [];
    
    for (const testCase of testTexts) {
      try {
        const startTime = Date.now();
        const classification = await classifier.classifyText(testCase.text);
        const endTime = Date.now();
        
        results.push({
          text: testCase.text,
          expected: testCase.expectedCategory,
          predicted: classification.category,
          confidence: classification.confidence,
          inferenceTime: endTime - startTime,
          correct: testCase.expectedCategory === classification.category
        });
      } catch (error) {
        results.push({
          text: testCase.text,
          error: error.message
        });
      }
    }
    
    this.testResults.push({
      test: 'text_classification',
      results,
      accuracy: this.calculateAccuracy(results),
      timestamp: new Date()
    });
    
    return results;
  }

  calculateAccuracy(results) {
    const correctResults = results.filter(r => r.correct);
    return correctResults.length / results.length;
  }

  getTestResults() {
    return this.testResults;
  }
}

export default NLPModelTester;
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Analizador de Reseñas**

```javascript
// Ejercicio: Crear un analizador de reseñas de productos
// 1. Analizar sentimientos de reseñas
// 2. Clasificar por categorías
// 3. Extraer palabras clave
// 4. Generar resumen automático

const reviewAnalyzer = {
  async analyzeReview(reviewText) {
    // Implementar análisis de reseñas
    // Retornar: { sentiment: 'positive', category: 'product', keywords: [...] }
  }
};
```

### **Ejercicio 2: Asistente Virtual**

```javascript
// Ejercicio: Crear un asistente virtual inteligente
// 1. Procesar comandos de voz
// 2. Generar respuestas contextuales
// 3. Mantener conversación
// 4. Aprender de interacciones

const virtualAssistant = {
  async processCommand(command) {
    // Implementar procesamiento de comandos
    // Retornar: { response: '...', action: '...', confidence: 0.95 }
  }
};
```

---

## 🚀 Próximos Pasos

### **Lo que Aprendiste**

1. **Implementación** de análisis de sentimientos
2. **Clasificación** de texto y categorización
3. **Traducción** automática
4. **Chatbots** inteligentes
5. **Optimización** de procesamiento de texto

### **Preparación para la Siguiente Clase**

En la próxima clase aprenderás sobre:
- **Sistemas de recomendación** colaborativos
- **Filtrado basado en contenido** y comportamiento
- **Machine Learning** para personalización
- **A/B testing** y optimización
- **Analytics** y métricas de engagement

### **Tarea para Casa**

1. **Implementar** un analizador de sentimientos personalizado
2. **Crear** un chatbot para tu aplicación
3. **Desarrollar** un sistema de traducción
4. **Optimizar** el rendimiento de los modelos NLP

---

## 📚 Recursos Adicionales

### **Modelos Pre-entrenados**
- [TensorFlow.js Text Models](https://github.com/tensorflow/tfjs-models/tree/master/text)
- [Hugging Face Models](https://huggingface.co/models)
- [Google AI Models](https://ai.google.dev/models)

### **Herramientas de Desarrollo**
- [Natural Language Toolkit](https://www.nltk.org/)
- [spaCy](https://spacy.io/)
- [Transformers](https://huggingface.co/transformers/)

### **Documentación**
- [TensorFlow.js Text Guide](https://tensorflow.org/js/guide/concepts)
- [NLP Best Practices](https://tensorflow.org/js/guide/best_practices)
- [Chatbot Development](https://tensorflow.org/js/guide/chatbot)

---

**🎯 Objetivo de la Clase**: Dominar la implementación de capacidades avanzadas de procesamiento de lenguaje natural en React Native, desde análisis de sentimientos hasta chatbots inteligentes.

**💡 Consejo**: Combina múltiples técnicas de NLP para crear aplicaciones más inteligentes. La optimización de rendimiento es crucial para la experiencia del usuario.

---

**🚀 ¡Has completado la Clase 3 de Natural Language Processing! Continúa con la Clase 4 para dominar los sistemas de recomendaciones.**
