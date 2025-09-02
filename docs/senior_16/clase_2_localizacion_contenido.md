# 🌍 Clase 2: Localización de Contenido

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a gestionar archivos de traducción de manera eficiente, organizar claves de traducción, manejar contenido dinámico, validar traducciones y establecer un workflow de traducción profesional.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Gestionar** archivos de traducción de manera escalable
2. **Organizar** claves de traducción de forma lógica
3. **Manejar** traducción de contenido dinámico
4. **Validar** traducciones automáticamente
5. **Establecer** un workflow de traducción profesional

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Gestión de Archivos de Traducción**

#### **Estrategias de Organización**

```javascript
// Estructura recomendada para proyectos grandes
src/
├── i18n/
│   ├── index.js
│   ├── config.js
│   ├── utils/
│   │   ├── validation.js
│   │   ├── extraction.js
│   │   └── helpers.js
│   └── locales/
│       ├── en/
│       │   ├── common.json
│       │   ├── auth.json
│       │   ├── profile.json
│       │   ├── settings.json
│       │   └── errors.json
│       ├── es/
│       │   ├── common.json
│       │   ├── auth.json
│       │   ├── profile.json
│       │   ├── settings.json
│       │   └── errors.json
│       └── fr/
│           ├── common.json
│           ├── auth.json
│           ├── profile.json
│           ├── settings.json
│           └── errors.json
```

#### **Carga Dinámica de Traducciones**

```javascript
// i18n/dynamicLoader.js
import i18n from 'i18next';

class TranslationLoader {
  constructor() {
    this.loadedNamespaces = new Set();
  }
  
  async loadNamespace(namespace, language) {
    const key = `${language}-${namespace}`;
    
    if (this.loadedNamespaces.has(key)) {
      return;
    }
    
    try {
      const translation = await import(`./locales/${language}/${namespace}.json`);
      i18n.addResourceBundle(language, namespace, translation.default);
      this.loadedNamespaces.add(key);
    } catch (error) {
      console.warn(`Failed to load ${namespace} for ${language}:`, error);
    }
  }
  
  async preloadNamespaces(namespaces, language) {
    const promises = namespaces.map(ns => this.loadNamespace(ns, language));
    await Promise.all(promises);
  }
}

export const translationLoader = new TranslationLoader();
```

#### **Gestión de Versiones**

```javascript
// i18n/versionManager.js
import AsyncStorage from '@react-native-async-storage/async-storage';

class VersionManager {
  constructor() {
    this.versionKey = 'translation_version';
    this.currentVersion = '1.0.0';
  }
  
  async checkForUpdates() {
    try {
      const storedVersion = await AsyncStorage.getItem(this.versionKey);
      
      if (storedVersion !== this.currentVersion) {
        await this.updateTranslations();
        await AsyncStorage.setItem(this.versionKey, this.currentVersion);
      }
    } catch (error) {
      console.error('Error checking translation updates:', error);
    }
  }
  
  async updateTranslations() {
    // Lógica para actualizar traducciones
    // Descargar nuevas traducciones del servidor
    // Actualizar archivos locales
  }
}

export const versionManager = new VersionManager();
```

### **2. Organización de Claves de Traducción**

#### **Convenciones de Nomenclatura**

```json
// ✅ Buena organización - Claves descriptivas y jerárquicas
{
  "auth": {
    "login": {
      "title": "Sign In",
      "form": {
        "email": {
          "label": "Email",
          "placeholder": "Enter your email",
          "error": {
            "required": "Email is required",
            "invalid": "Please enter a valid email"
          }
        },
        "password": {
          "label": "Password",
          "placeholder": "Enter your password",
          "error": {
            "required": "Password is required",
            "minLength": "Password must be at least 8 characters"
          }
        }
      },
      "button": {
        "submit": "Sign In",
        "forgot": "Forgot Password?",
        "signup": "Don't have an account? Sign Up"
      }
    }
  }
}
```

```json
// ❌ Mala organización - Claves genéricas y planas
{
  "title": "Sign In",
  "emailLabel": "Email",
  "emailPlaceholder": "Enter your email",
  "emailError": "Please enter a valid email",
  "passwordLabel": "Password",
  "passwordPlaceholder": "Enter your password",
  "passwordError": "Password must be at least 8 characters",
  "submitButton": "Sign In",
  "forgotButton": "Forgot Password?",
  "signupButton": "Don't have an account? Sign Up"
}
```

#### **Sistema de Categorización**

```javascript
// i18n/categories.js
export const TRANSLATION_CATEGORIES = {
  // Contenido estático
  STATIC: 'static',
  
  // Contenido dinámico
  DYNAMIC: 'dynamic',
  
  // Mensajes de error
  ERROR: 'error',
  
  // Mensajes de éxito
  SUCCESS: 'success',
  
  // Contenido de ayuda
  HELP: 'help',
  
  // Contenido de navegación
  NAVIGATION: 'navigation',
  
  // Contenido de formularios
  FORM: 'form',
  
  // Contenido de notificaciones
  NOTIFICATION: 'notification'
};

// Ejemplo de uso
const getTranslationKey = (category, section, key) => {
  return `${category}.${section}.${key}`;
};

// Uso
const errorKey = getTranslationKey(TRANSLATION_CATEGORIES.ERROR, 'auth', 'invalidCredentials');
// Resultado: "error.auth.invalidCredentials"
```

#### **Gestión de Contextos**

```json
// locales/en/contexts.json
{
  "button": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "create": "Create",
    "update": "Update"
  },
  "status": {
    "online": "Online",
    "offline": "Offline",
    "away": "Away",
    "busy": "Busy"
  },
  "priority": {
    "low": "Low",
    "medium": "Medium",
    "high": "High",
    "urgent": "Urgent"
  }
}
```

```javascript
// Uso de contextos
const getContextualTranslation = (baseKey, context, value) => {
  return `${baseKey}.${context}.${value}`;
};

// Ejemplos
const saveButton = getContextualTranslation('button', 'action', 'save');
const onlineStatus = getContextualTranslation('status', 'connection', 'online');
const highPriority = getContextualTranslation('priority', 'level', 'high');
```

### **3. Traducción de Contenido Dinámico**

#### **Contenido del Servidor**

```javascript
// services/translationService.js
import i18n from '../i18n';

class TranslationService {
  async translateServerContent(content, targetLanguage) {
    try {
      // Si el contenido ya está traducido, devolverlo
      if (content.translations && content.translations[targetLanguage]) {
        return content.translations[targetLanguage];
      }
      
      // Si no, usar servicio de traducción automática
      const translatedContent = await this.autoTranslate(content.default, targetLanguage);
      
      // Guardar traducción para futuras consultas
      await this.saveTranslation(content.id, targetLanguage, translatedContent);
      
      return translatedContent;
    } catch (error) {
      console.error('Translation error:', error);
      return content.default; // Fallback al contenido por defecto
    }
  }
  
  async autoTranslate(text, targetLanguage) {
    // Implementar servicio de traducción automática
    // Google Translate API, Azure Translator, etc.
    const response = await fetch('/api/translate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text, targetLanguage })
    });
    
    const result = await response.json();
    return result.translatedText;
  }
  
  async saveTranslation(contentId, language, translation) {
    // Guardar traducción en base de datos
    await fetch('/api/translations', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ contentId, language, translation })
    });
  }
}

export const translationService = new TranslationService();
```

#### **Contenido de Usuario**

```javascript
// components/UserGeneratedContent.js
import React, { useState, useEffect } from 'react';
import { View, Text } from 'react-native';
import { useTranslation } from 'react-i18next';
import { translationService } from '../services/translationService';

const UserGeneratedContent = ({ content, userLanguage }) => {
  const { t } = useTranslation();
  const [translatedContent, setTranslatedContent] = useState(content);
  const [isTranslating, setIsTranslating] = useState(false);
  
  useEffect(() => {
    if (content.language !== userLanguage) {
      translateContent();
    }
  }, [content, userLanguage]);
  
  const translateContent = async () => {
    setIsTranslating(true);
    try {
      const translated = await translationService.translateServerContent(
        content,
        userLanguage
      );
      setTranslatedContent(translated);
    } catch (error) {
      console.error('Translation failed:', error);
    } finally {
      setIsTranslating(false);
    }
  };
  
  return (
    <View>
      {isTranslating ? (
        <Text>{t('common.translating')}</Text>
      ) : (
        <Text>{translatedContent}</Text>
      )}
    </View>
  );
};

export default UserGeneratedContent;
```

#### **Contenido de Tiempo Real**

```javascript
// hooks/useRealTimeTranslation.js
import { useState, useEffect, useCallback } from 'react';
import { useTranslation } from 'react-i18next';
import { translationService } from '../services/translationService';

export const useRealTimeTranslation = (content, targetLanguage) => {
  const { i18n } = useTranslation();
  const [translatedContent, setTranslatedContent] = useState(content);
  const [isTranslating, setIsTranslating] = useState(false);
  
  const translate = useCallback(async (text) => {
    if (!text || text === translatedContent) return;
    
    setIsTranslating(true);
    try {
      const translated = await translationService.autoTranslate(text, targetLanguage);
      setTranslatedContent(translated);
    } catch (error) {
      console.error('Real-time translation failed:', error);
      setTranslatedContent(text); // Fallback
    } finally {
      setIsTranslating(false);
    }
  }, [targetLanguage, translatedContent]);
  
  useEffect(() => {
    if (content && content !== translatedContent) {
      translate(content);
    }
  }, [content, translate]);
  
  return {
    translatedContent,
    isTranslating,
    retryTranslation: () => translate(content)
  };
};
```

### **4. Validación de Traducciones**

#### **Validación de Estructura**

```javascript
// i18n/validation.js
import fs from 'fs';
import path from 'path';

class TranslationValidator {
  constructor() {
    this.errors = [];
    this.warnings = [];
  }
  
  validateStructure(baseLanguage, targetLanguages) {
    const baseTranslations = this.loadTranslations(baseLanguage);
    
    targetLanguages.forEach(language => {
      const targetTranslations = this.loadTranslations(language);
      this.compareStructures(baseTranslations, targetTranslations, language);
    });
    
    return {
      errors: this.errors,
      warnings: this.warnings,
      isValid: this.errors.length === 0
    };
  }
  
  compareStructures(base, target, language) {
    this.compareKeys(base, target, language, '');
  }
  
  compareKeys(base, target, language, prefix) {
    Object.keys(base).forEach(key => {
      const fullKey = prefix ? `${prefix}.${key}` : key;
      
      if (!target.hasOwnProperty(key)) {
        this.errors.push(`Missing key: ${fullKey} in ${language}`);
        return;
      }
      
      if (typeof base[key] === 'object' && typeof target[key] === 'object') {
        this.compareKeys(base[key], target[key], language, fullKey);
      } else if (typeof base[key] !== typeof target[key]) {
        this.errors.push(`Type mismatch: ${fullKey} in ${language}`);
      }
    });
  }
  
  loadTranslations(language) {
    const filePath = path.join(__dirname, `../locales/${language}/common.json`);
    return JSON.parse(fs.readFileSync(filePath, 'utf8'));
  }
}

export const translationValidator = new TranslationValidator();
```

#### **Validación de Contenido**

```javascript
// i18n/contentValidation.js
class ContentValidator {
  validateTranslations(translations) {
    const issues = [];
    
    Object.entries(translations).forEach(([language, content]) => {
      issues.push(...this.validateLanguage(language, content));
    });
    
    return issues;
  }
  
  validateLanguage(language, content) {
    const issues = [];
    
    // Validar longitud de texto
    issues.push(...this.validateTextLength(language, content));
    
    // Validar caracteres especiales
    issues.push(...this.validateSpecialCharacters(language, content));
    
    // Validar interpolación
    issues.push(...this.validateInterpolation(language, content));
    
    return issues;
  }
  
  validateTextLength(language, content) {
    const issues = [];
    const maxLength = this.getMaxLengthForLanguage(language);
    
    this.traverseContent(content, (key, value) => {
      if (typeof value === 'string' && value.length > maxLength) {
        issues.push({
          type: 'warning',
          message: `Text too long: ${key} (${value.length} chars, max: ${maxLength})`,
          language,
          key
        });
      }
    });
    
    return issues;
  }
  
  validateSpecialCharacters(language, content) {
    const issues = [];
    const allowedChars = this.getAllowedCharacters(language);
    
    this.traverseContent(content, (key, value) => {
      if (typeof value === 'string') {
        const invalidChars = value.split('').filter(char => !allowedChars.includes(char));
        if (invalidChars.length > 0) {
          issues.push({
            type: 'error',
            message: `Invalid characters in ${key}: ${invalidChars.join(', ')}`,
            language,
            key
          });
        }
      }
    });
    
    return issues;
  }
  
  validateInterpolation(language, content) {
    const issues = [];
    
    this.traverseContent(content, (key, value) => {
      if (typeof value === 'string') {
        const variables = value.match(/\{\{([^}]+)\}\}/g);
        if (variables) {
          variables.forEach(variable => {
            if (!this.isValidVariable(variable)) {
              issues.push({
                type: 'error',
                message: `Invalid interpolation variable: ${variable} in ${key}`,
                language,
                key
              });
            }
          });
        }
      }
    });
    
    return issues;
  }
  
  traverseContent(content, callback, prefix = '') {
    Object.entries(content).forEach(([key, value]) => {
      const fullKey = prefix ? `${prefix}.${key}` : key;
      
      if (typeof value === 'object' && value !== null) {
        this.traverseContent(value, callback, fullKey);
      } else {
        callback(fullKey, value);
      }
    });
  }
  
  getMaxLengthForLanguage(language) {
    const limits = {
      'en': 100,
      'es': 120,
      'fr': 110,
      'de': 130,
      'ar': 80
    };
    return limits[language] || 100;
  }
  
  getAllowedCharacters(language) {
    const charSets = {
      'en': /[a-zA-Z0-9\s.,!?;:'"()-]/,
      'es': /[a-zA-ZáéíóúñüÁÉÍÓÚÑÜ0-9\s.,!?;:'"()-]/,
      'fr': /[a-zA-ZàâäéèêëïîôöùûüÿçÀÂÄÉÈÊËÏÎÔÖÙÛÜŸÇ0-9\s.,!?;:'"()-]/,
      'de': /[a-zA-ZäöüßÄÖÜ0-9\s.,!?;:'"()-]/,
      'ar': /[\u0600-\u06FF\s.,!?;:'"()-]/
    };
    return charSets[language] || charSets['en'];
  }
  
  isValidVariable(variable) {
    // Validar formato de variable: {{variableName}}
    const match = variable.match(/\{\{([^}]+)\}\}/);
    if (!match) return false;
    
    const varName = match[1];
    return /^[a-zA-Z_][a-zA-Z0-9_]*$/.test(varName);
  }
}

export const contentValidator = new ContentValidator();
```

### **5. Workflow de Traducción**

#### **Automatización de Extracción**

```javascript
// scripts/extractTranslations.js
import fs from 'fs';
import path from 'path';
import { glob } from 'glob';

class TranslationExtractor {
  constructor() {
    this.extractedKeys = new Set();
    this.patterns = [
      /t\(['"`]([^'"`]+)['"`]\)/g,
      /t\(['"`]([^'"`]+)['"`],\s*\{[^}]*\}/g,
      /useTranslation\(\)[^}]*t\(['"`]([^'"`]+)['"`]\)/g
    ];
  }
  
  async extractFromProject() {
    const files = await glob('src/**/*.{js,jsx,ts,tsx}');
    
    files.forEach(file => {
      const content = fs.readFileSync(file, 'utf8');
      this.extractFromFile(content, file);
    });
    
    return Array.from(this.extractedKeys);
  }
  
  extractFromFile(content, filePath) {
    this.patterns.forEach(pattern => {
      let match;
      while ((match = pattern.exec(content)) !== null) {
        const key = match[1];
        this.extractedKeys.add(key);
        console.log(`Found translation key: ${key} in ${filePath}`);
      }
    });
  }
  
  generateTranslationTemplate(keys) {
    const template = {};
    
    keys.forEach(key => {
      const parts = key.split('.');
      let current = template;
      
      parts.forEach((part, index) => {
        if (index === parts.length - 1) {
          current[part] = `[TRANSLATE] ${key}`;
        } else {
          if (!current[part]) {
            current[part] = {};
          }
          current = current[part];
        }
      });
    });
    
    return template;
  }
  
  async saveTemplate(template, language) {
    const filePath = path.join(__dirname, `../locales/${language}/template.json`);
    fs.writeFileSync(filePath, JSON.stringify(template, null, 2));
  }
}

// Uso
const extractor = new TranslationExtractor();
extractor.extractFromProject().then(keys => {
  const template = extractor.generateTranslationTemplate(keys);
  extractor.saveTemplate(template, 'en');
});
```

#### **Integración con Herramientas de Traducción**

```javascript
// scripts/translationWorkflow.js
import { LokaliseApi } from '@lokalise/node-api';
import fs from 'fs';
import path from 'path';

class TranslationWorkflow {
  constructor() {
    this.api = new LokaliseApi({
      apiKey: process.env.LOKALISE_API_KEY
    });
    this.projectId = process.env.LOKALISE_PROJECT_ID;
  }
  
  async uploadTranslations() {
    try {
      // Leer archivos de traducción locales
      const translations = await this.readLocalTranslations();
      
      // Subir a Lokalise
      await this.api.files.upload(this.projectId, {
        data: translations,
        lang_iso: 'en',
        replace_modified: true
      });
      
      console.log('Translations uploaded successfully');
    } catch (error) {
      console.error('Upload failed:', error);
    }
  }
  
  async downloadTranslations() {
    try {
      // Descargar traducciones de Lokalise
      const response = await this.api.files.download(this.projectId, {
        format: 'json',
        original_filenames: true
      });
      
      // Guardar archivos locales
      await this.saveTranslations(response);
      
      console.log('Translations downloaded successfully');
    } catch (error) {
      console.error('Download failed:', error);
    }
  }
  
  async readLocalTranslations() {
    const translations = {};
    const localesDir = path.join(__dirname, '../locales');
    const languages = fs.readdirSync(localesDir);
    
    languages.forEach(language => {
      const languageDir = path.join(localesDir, language);
      const files = fs.readdirSync(languageDir);
      
      files.forEach(file => {
        if (file.endsWith('.json')) {
          const filePath = path.join(languageDir, file);
          const content = fs.readFileSync(filePath, 'utf8');
          const namespace = file.replace('.json', '');
          
          if (!translations[language]) {
            translations[language] = {};
          }
          
          translations[language][namespace] = JSON.parse(content);
        }
      });
    });
    
    return translations;
  }
  
  async saveTranslations(translations) {
    Object.entries(translations).forEach(([language, namespaces]) => {
      Object.entries(namespaces).forEach(([namespace, content]) => {
        const filePath = path.join(__dirname, `../locales/${language}/${namespace}.json`);
        fs.writeFileSync(filePath, JSON.stringify(content, null, 2));
      });
    });
  }
}

export const translationWorkflow = new TranslationWorkflow();
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Sistema de Carga Dinámica**

```javascript
// components/LazyTranslatedComponent.js
import React, { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';
import { useTranslation } from 'react-i18next';
import { translationLoader } from '../i18n/dynamicLoader';

const LazyTranslatedComponent = ({ namespace, translationKey, fallback }) => {
  const { t, i18n } = useTranslation();
  const [isLoaded, setIsLoaded] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  
  useEffect(() => {
    loadNamespace();
  }, [namespace, i18n.language]);
  
  const loadNamespace = async () => {
    setIsLoading(true);
    try {
      await translationLoader.loadNamespace(namespace, i18n.language);
      setIsLoaded(true);
    } catch (error) {
      console.error('Failed to load namespace:', error);
    } finally {
      setIsLoading(false);
    }
  };
  
  if (isLoading) {
    return <ActivityIndicator size="small" />;
  }
  
  if (!isLoaded) {
    return <Text>{fallback}</Text>;
  }
  
  return <Text>{t(translationKey)}</Text>;
};

export default LazyTranslatedComponent;
```

### **Ejercicio 2: Validación en Tiempo Real**

```javascript
// hooks/useTranslationValidation.js
import { useState, useEffect } from 'react';
import { useTranslation } from 'react-i18next';
import { contentValidator } from '../i18n/contentValidation';

export const useTranslationValidation = () => {
  const { i18n } = useTranslation();
  const [validationResults, setValidationResults] = useState({});
  const [isValidating, setIsValidating] = useState(false);
  
  const validateCurrentLanguage = async () => {
    setIsValidating(true);
    try {
      const currentLanguage = i18n.language;
      const translations = i18n.getResourceBundle(currentLanguage, 'translation');
      
      const issues = contentValidator.validateLanguage(currentLanguage, translations);
      
      setValidationResults(prev => ({
        ...prev,
        [currentLanguage]: issues
      }));
    } catch (error) {
      console.error('Validation failed:', error);
    } finally {
      setIsValidating(false);
    }
  };
  
  useEffect(() => {
    validateCurrentLanguage();
  }, [i18n.language]);
  
  return {
    validationResults,
    isValidating,
    validateCurrentLanguage,
    hasErrors: Object.values(validationResults).some(issues => 
      issues.some(issue => issue.type === 'error')
    ),
    hasWarnings: Object.values(validationResults).some(issues => 
      issues.some(issue => issue.type === 'warning')
    )
  };
};
```

### **Ejercicio 3: Sistema de Traducción Automática**

```javascript
// components/AutoTranslatedText.js
import React, { useState, useEffect } from 'react';
import { Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useTranslation } from 'react-i18next';
import { translationService } from '../services/translationService';

const AutoTranslatedText = ({ 
  text, 
  sourceLanguage, 
  targetLanguage, 
  style,
  showTranslationButton = false 
}) => {
  const { t } = useTranslation();
  const [translatedText, setTranslatedText] = useState(text);
  const [isTranslating, setIsTranslating] = useState(false);
  const [translationError, setTranslationError] = useState(null);
  
  useEffect(() => {
    if (sourceLanguage !== targetLanguage) {
      translateText();
    }
  }, [text, sourceLanguage, targetLanguage]);
  
  const translateText = async () => {
    setIsTranslating(true);
    setTranslationError(null);
    
    try {
      const translated = await translationService.autoTranslate(text, targetLanguage);
      setTranslatedText(translated);
    } catch (error) {
      setTranslationError(error.message);
      setTranslatedText(text); // Fallback al texto original
    } finally {
      setIsTranslating(false);
    }
  };
  
  const handleRetryTranslation = () => {
    translateText();
  };
  
  return (
    <TouchableOpacity 
      style={styles.container}
      onPress={showTranslationButton ? handleRetryTranslation : undefined}
      disabled={!showTranslationButton}
    >
      <Text style={style}>
        {isTranslating ? t('common.translating') : translatedText}
      </Text>
      
      {translationError && (
        <Text style={styles.errorText}>
          {t('common.translationError')}
        </Text>
      )}
      
      {showTranslationButton && translationError && (
        <Text style={styles.retryText}>
          {t('common.tapToRetry')}
        </Text>
      )}
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  errorText: {
    color: 'red',
    fontSize: 12,
    marginTop: 4,
  },
  retryText: {
    color: 'blue',
    fontSize: 12,
    marginTop: 4,
  },
});

export default AutoTranslatedText;
```

### **Ejercicio 4: Dashboard de Traducción**

```javascript
// screens/TranslationDashboard.js
import React, { useState, useEffect } from 'react';
import { 
  View, 
  Text, 
  ScrollView, 
  TouchableOpacity, 
  StyleSheet,
  Alert 
} from 'react-native';
import { useTranslation } from 'react-i18next';
import { useTranslationValidation } from '../hooks/useTranslationValidation';
import { translationWorkflow } from '../scripts/translationWorkflow';

const TranslationDashboard = () => {
  const { t, i18n } = useTranslation();
  const { 
    validationResults, 
    isValidating, 
    hasErrors, 
    hasWarnings 
  } = useTranslationValidation();
  
  const [isUploading, setIsUploading] = useState(false);
  const [isDownloading, setIsDownloading] = useState(false);
  
  const handleUploadTranslations = async () => {
    setIsUploading(true);
    try {
      await translationWorkflow.uploadTranslations();
      Alert.alert(t('dashboard.uploadSuccess'));
    } catch (error) {
      Alert.alert(t('dashboard.uploadError'), error.message);
    } finally {
      setIsUploading(false);
    }
  };
  
  const handleDownloadTranslations = async () => {
    setIsDownloading(true);
    try {
      await translationWorkflow.downloadTranslations();
      Alert.alert(t('dashboard.downloadSuccess'));
    } catch (error) {
      Alert.alert(t('dashboard.downloadError'), error.message);
    } finally {
      setIsDownloading(false);
    }
  };
  
  const renderValidationResults = () => {
    const currentLanguage = i18n.language;
    const issues = validationResults[currentLanguage] || [];
    
    if (issues.length === 0) {
      return (
        <View style={styles.successContainer}>
          <Text style={styles.successText}>
            {t('dashboard.noIssues')}
          </Text>
        </View>
      );
    }
    
    return (
      <View style={styles.issuesContainer}>
        {issues.map((issue, index) => (
          <View 
            key={index} 
            style={[
              styles.issueItem,
              issue.type === 'error' ? styles.errorIssue : styles.warningIssue
            ]}
          >
            <Text style={styles.issueText}>{issue.message}</Text>
            <Text style={styles.issueKey}>{issue.key}</Text>
          </View>
        ))}
      </View>
    );
  };
  
  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>{t('dashboard.title')}</Text>
        <Text style={styles.subtitle}>
          {t('dashboard.currentLanguage')}: {i18n.language}
        </Text>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>{t('dashboard.validation')}</Text>
        {isValidating ? (
          <Text>{t('dashboard.validating')}</Text>
        ) : (
          renderValidationResults()
        )}
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>{t('dashboard.workflow')}</Text>
        
        <TouchableOpacity 
          style={[styles.button, isUploading && styles.buttonDisabled]}
          onPress={handleUploadTranslations}
          disabled={isUploading}
        >
          <Text style={styles.buttonText}>
            {isUploading ? t('dashboard.uploading') : t('dashboard.upload')}
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={[styles.button, isDownloading && styles.buttonDisabled]}
          onPress={handleDownloadTranslations}
          disabled={isDownloading}
        >
          <Text style={styles.buttonText}>
            {isDownloading ? t('dashboard.downloading') : t('dashboard.download')}
          </Text>
        </TouchableOpacity>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>{t('dashboard.statistics')}</Text>
        <Text style={styles.statText}>
          {t('dashboard.totalLanguages')}: {i18n.languages.length}
        </Text>
        <Text style={styles.statText}>
          {t('dashboard.currentLanguage')}: {i18n.language}
        </Text>
        <Text style={styles.statText}>
          {t('dashboard.fallbackLanguage')}: {i18n.options.fallbackLng}
        </Text>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    padding: 20,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
  section: {
    margin: 20,
    padding: 20,
    backgroundColor: '#fff',
    borderRadius: 8,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  successContainer: {
    padding: 16,
    backgroundColor: '#d4edda',
    borderRadius: 8,
  },
  successText: {
    color: '#155724',
    textAlign: 'center',
  },
  issuesContainer: {
    gap: 12,
  },
  issueItem: {
    padding: 12,
    borderRadius: 8,
    borderLeftWidth: 4,
  },
  errorIssue: {
    backgroundColor: '#f8d7da',
    borderLeftColor: '#dc3545',
  },
  warningIssue: {
    backgroundColor: '#fff3cd',
    borderLeftColor: '#ffc107',
  },
  issueText: {
    fontSize: 14,
    marginBottom: 4,
  },
  issueKey: {
    fontSize: 12,
    color: '#666',
    fontFamily: 'monospace',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    marginBottom: 12,
  },
  buttonDisabled: {
    backgroundColor: '#ccc',
  },
  buttonText: {
    color: '#fff',
    textAlign: 'center',
    fontWeight: 'bold',
  },
  statText: {
    fontSize: 16,
    marginBottom: 8,
  },
});

export default TranslationDashboard;
```

---

## 🧪 Testing

### **Testing de Validación**

```javascript
// __tests__/translationValidation.test.js
import { contentValidator } from '../i18n/contentValidation';

describe('Content Validation', () => {
  test('validates text length correctly', () => {
    const translations = {
      en: {
        short: 'Hi',
        long: 'This is a very long text that exceeds the maximum length for English language which should trigger a warning'
      }
    };
    
    const issues = contentValidator.validateTranslations(translations);
    const lengthIssues = issues.filter(issue => issue.message.includes('too long'));
    
    expect(lengthIssues.length).toBeGreaterThan(0);
    expect(lengthIssues[0].type).toBe('warning');
  });
  
  test('validates special characters correctly', () => {
    const translations = {
      en: {
        valid: 'Hello World!',
        invalid: 'Hello 世界!' // Chinese characters in English
      }
    };
    
    const issues = contentValidator.validateTranslations(translations);
    const charIssues = issues.filter(issue => issue.message.includes('Invalid characters'));
    
    expect(charIssues.length).toBeGreaterThan(0);
    expect(charIssues[0].type).toBe('error');
  });
  
  test('validates interpolation variables correctly', () => {
    const translations = {
      en: {
        valid: 'Hello {{name}}!',
        invalid: 'Hello {{invalid-var}}!' // Invalid variable name
      }
    };
    
    const issues = contentValidator.validateTranslations(translations);
    const interpolationIssues = issues.filter(issue => 
      issue.message.includes('Invalid interpolation variable')
    );
    
    expect(interpolationIssues.length).toBeGreaterThan(0);
    expect(interpolationIssues[0].type).toBe('error');
  });
});
```

### **Testing de Carga Dinámica**

```javascript
// __tests__/dynamicLoader.test.js
import { translationLoader } from '../i18n/dynamicLoader';
import i18n from '../i18n';

describe('Translation Loader', () => {
  beforeEach(() => {
    // Reset i18n state
    i18n.removeResourceBundle('en', 'test');
  });
  
  test('loads namespace correctly', async () => {
    const namespace = 'test';
    const language = 'en';
    
    await translationLoader.loadNamespace(namespace, language);
    
    const resource = i18n.getResourceBundle(language, namespace);
    expect(resource).toBeDefined();
  });
  
  test('handles loading errors gracefully', async () => {
    const namespace = 'nonexistent';
    const language = 'en';
    
    await expect(
      translationLoader.loadNamespace(namespace, language)
    ).resolves.not.toThrow();
  });
  
  test('preloads multiple namespaces', async () => {
    const namespaces = ['common', 'auth'];
    const language = 'en';
    
    await translationLoader.preloadNamespaces(namespaces, language);
    
    namespaces.forEach(namespace => {
      const resource = i18n.getResourceBundle(language, namespace);
      expect(resource).toBeDefined();
    });
  });
});
```

---

## 📱 Ejemplo Completo

### **Sistema de Traducción Completo**

```javascript
// App.js - Configuración completa
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useTranslation } from 'react-i18next';
import AsyncStorage from '@react-native-async-storage/async-storage';
import './i18n';
import { versionManager } from './i18n/versionManager';
import { translationLoader } from './i18n/dynamicLoader';

import WelcomeScreen from './screens/WelcomeScreen';
import TranslationDashboard from './screens/TranslationDashboard';

const Stack = createStackNavigator();

const App = () => {
  const { i18n } = useTranslation();
  
  useEffect(() => {
    initializeTranslations();
  }, []);
  
  const initializeTranslations = async () => {
    try {
      // Cargar idioma guardado
      const savedLanguage = await AsyncStorage.getItem('language');
      if (savedLanguage) {
        await i18n.changeLanguage(savedLanguage);
      }
      
      // Verificar actualizaciones de traducción
      await versionManager.checkForUpdates();
      
      // Precargar namespaces críticos
      await translationLoader.preloadNamespaces(['common', 'auth'], i18n.language);
      
    } catch (error) {
      console.error('Translation initialization failed:', error);
    }
  };
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Welcome" component={WelcomeScreen} />
        <Stack.Screen name="Dashboard" component={TranslationDashboard} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 🎯 Resumen de la Clase

### **Conceptos Clave Aprendidos**

1. **Gestión de archivos**: Organización escalable de traducciones
2. **Organización de claves**: Convenciones y categorización
3. **Contenido dinámico**: Traducción de contenido del servidor y usuario
4. **Validación**: Verificación automática de traducciones
5. **Workflow**: Automatización del proceso de traducción

### **Habilidades Desarrolladas**

- ✅ Gestión avanzada de archivos de traducción
- ✅ Organización lógica de claves de traducción
- ✅ Manejo de contenido dinámico
- ✅ Validación automática de traducciones
- ✅ Establecimiento de workflow profesional

### **Próximos Pasos**

En la siguiente clase aprenderás sobre:
- Soporte RTL (Right-to-Left)
- Layouts adaptativos para diferentes idiomas
- Navegación y flujos RTL
- Testing de interfaces RTL

---

## 📚 Recursos Adicionales

### **Herramientas de Traducción**
- [Lokalise](https://lokalise.com/) - Plataforma de gestión de traducciones
- [Crowdin](https://crowdin.com/) - Herramienta de localización
- [POEditor](https://poeditor.com/) - Editor de traducciones
- [Transifex](https://www.transifex.com/) - Plataforma de localización

### **Automatización**
- [i18next Scanner](https://github.com/i18next/i18next-scanner) - Extracción automática
- [i18next Ally](https://github.com/lokalise/i18next-ally) - Extensión VS Code
- [i18next Testing](https://github.com/i18next/i18next-testing) - Utilidades de testing

### **APIs de Traducción**
- [Google Translate API](https://cloud.google.com/translate)
- [Azure Translator](https://azure.microsoft.com/en-us/services/cognitive-services/translator/)
- [DeepL API](https://www.deepl.com/en/docs-api)

---

**🎯 Objetivo Alcanzado**: Has aprendido a gestionar traducciones de manera profesional y escalable, con validación automática y workflow optimizado.

**💡 Consejo**: Un buen sistema de traducción es la base para aplicaciones verdaderamente globales. La automatización y validación son clave para mantener la calidad.

**🚀 ¡Continúa con la siguiente clase para aprender sobre soporte RTL y layouts adaptativos!**
