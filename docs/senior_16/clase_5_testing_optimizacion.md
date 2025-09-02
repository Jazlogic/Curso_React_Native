# 🌍 Clase 5: Testing y Optimización

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar testing completo para aplicaciones multilingües, validación de traducciones, optimización de performance, debugging de problemas de localización y mejores prácticas para aplicaciones i18n.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** testing de aplicaciones multilingües
2. **Validar** traducciones automáticamente
3. **Optimizar** performance de aplicaciones i18n
4. **Debuggear** problemas de localización
5. **Aplicar** mejores prácticas de i18n

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Testing de Aplicaciones Multilingües**

#### **Testing de Traducciones**

```javascript
// __tests__/translation.test.js
import i18n from '../i18n';
import { render, screen } from '@testing-library/react-native';
import { I18nextProvider } from 'react-i18next';
import WelcomeScreen from '../screens/WelcomeScreen';

const renderWithI18n = (component, { locale = 'en' } = {}) => {
  i18n.changeLanguage(locale);
  return render(
    <I18nextProvider i18n={i18n}>
      {component}
    </I18nextProvider>
  );
};

describe('Translation Testing', () => {
  beforeEach(() => {
    i18n.changeLanguage('en');
  });
  
  test('renders English text by default', () => {
    renderWithI18n(<WelcomeScreen />);
    expect(screen.getByText('Welcome to our app!')).toBeTruthy();
  });
  
  test('renders Spanish text when locale is es', () => {
    renderWithI18n(<WelcomeScreen />, { locale: 'es' });
    expect(screen.getByText('¡Bienvenido a nuestra aplicación!')).toBeTruthy();
  });
  
  test('handles missing translations gracefully', () => {
    i18n.changeLanguage('fr');
    renderWithI18n(<WelcomeScreen />);
    // Should fallback to English
    expect(screen.getByText('Welcome to our app!')).toBeTruthy();
  });
  
  test('interpolates variables correctly', () => {
    const { getByText } = renderWithI18n(<WelcomeScreen />);
    expect(getByText('Hello, John!')).toBeTruthy();
  });
});
```

#### **Testing de Pluralización**

```javascript
// __tests__/pluralization.test.js
import i18n from '../i18n';

describe('Pluralization Testing', () => {
  test('handles zero count', () => {
    i18n.changeLanguage('en');
    const result = i18n.t('messages.count', { count: 0 });
    expect(result).toBe('No messages');
  });
  
  test('handles singular count', () => {
    i18n.changeLanguage('en');
    const result = i18n.t('messages.count', { count: 1 });
    expect(result).toBe('1 message');
  });
  
  test('handles plural count', () => {
    i18n.changeLanguage('en');
    const result = i18n.t('messages.count', { count: 5 });
    expect(result).toBe('5 messages');
  });
  
  test('handles pluralization in different languages', () => {
    i18n.changeLanguage('es');
    const result = i18n.t('messages.count', { count: 5 });
    expect(result).toBe('5 mensajes');
  });
});
```

#### **Testing de RTL**

```javascript
// __tests__/rtl.test.js
import React from 'react';
import { render, screen } from '@testing-library/react-native';
import { I18nManager } from 'react-native';
import RTLIcon from '../components/RTLIcon';

// Mock I18nManager
jest.mock('react-native', () => ({
  ...jest.requireActual('react-native'),
  I18nManager: {
    isRTL: false,
    allowRTL: jest.fn(),
    forceRTL: jest.fn(),
  },
}));

describe('RTL Testing', () => {
  beforeEach(() => {
    I18nManager.isRTL = false;
  });
  
  test('renders LTR icon correctly', () => {
    render(<RTLIcon name="chevron-left" testID="icon" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('chevron-left');
  });
  
  test('renders RTL icon correctly', () => {
    I18nManager.isRTL = true;
    
    render(<RTLIcon name="chevron-left" testID="icon" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('chevron-right');
  });
  
  test('handles unknown icon names', () => {
    I18nManager.isRTL = true;
    
    render(<RTLIcon name="unknown-icon" testID="icon" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('unknown-icon');
  });
});
```

### **2. Validación de Traducciones**

#### **Validación de Estructura**

```javascript
// utils/translationValidator.js
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
// utils/contentValidator.js
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
        const invalidChars = value.split('').filter(char => !allowedChars.test(char));
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
    const match = variable.match(/\{\{([^}]+)\}\}/);
    if (!match) return false;
    
    const varName = match[1];
    return /^[a-zA-Z_][a-zA-Z0-9_]*$/.test(varName);
  }
}

export const contentValidator = new ContentValidator();
```

### **3. Optimización de Performance**

#### **Lazy Loading de Traducciones**

```javascript
// utils/translationLoader.js
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

#### **Optimización de Bundle**

```javascript
// i18n/optimizedConfig.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

const resources = {
  en: {
    common: require('./locales/en/common.json'),
    auth: require('./locales/en/auth.json'),
  },
  es: {
    common: require('./locales/es/common.json'),
    auth: require('./locales/es/auth.json'),
  },
};

i18n
  .use(initReactI18next)
  .init({
    resources,
    fallbackLng: 'en',
    debug: __DEV__,
    
    // Optimizaciones de performance
    load: 'languageOnly', // Cargar solo idioma, no región
    cleanCode: true, // Limpiar códigos de idioma
    nonExplicitSupportedLngs: true, // Soporte implícito de idiomas
    
    // Configuración de interpolación
    interpolation: {
      escapeValue: false,
    },
    
    // Configuración de pluralización
    pluralSeparator: '_',
    contextSeparator: '_',
    
    // Configuración de desarrollo
    saveMissing: __DEV__,
    missingKeyHandler: (lng, ns, key) => {
      if (__DEV__) {
        console.warn(`Missing translation: ${lng}.${ns}.${key}`);
      }
    },
  });

export default i18n;
```

#### **Memoización de Traducciones**

```javascript
// hooks/useOptimizedTranslation.js
import { useMemo } from 'react';
import { useTranslation } from 'react-i18next';

export const useOptimizedTranslation = (namespace) => {
  const { t, i18n } = useTranslation(namespace);
  
  const memoizedT = useMemo(() => {
    return (key, options) => {
      // Cache de traducciones frecuentes
      const cacheKey = `${i18n.language}-${namespace}-${key}-${JSON.stringify(options)}`;
      
      if (this.translationCache && this.translationCache[cacheKey]) {
        return this.translationCache[cacheKey];
      }
      
      const result = t(key, options);
      
      if (!this.translationCache) {
        this.translationCache = {};
      }
      
      this.translationCache[cacheKey] = result;
      return result;
    };
  }, [t, i18n.language, namespace]);
  
  return { t: memoizedT, i18n };
};
```

### **4. Debugging de Problemas de Localización**

#### **Herramientas de Debugging**

```javascript
// utils/i18nDebugger.js
class I18nDebugger {
  constructor() {
    this.debugMode = __DEV__;
    this.logs = [];
  }
  
  log(message, data = null) {
    if (this.debugMode) {
      const logEntry = {
        timestamp: new Date().toISOString(),
        message,
        data
      };
      
      this.logs.push(logEntry);
      console.log(`[i18n] ${message}`, data);
    }
  }
  
  logMissingTranslation(language, namespace, key) {
    this.log(`Missing translation: ${language}.${namespace}.${key}`);
  }
  
  logLanguageChange(from, to) {
    this.log(`Language changed from ${from} to ${to}`);
  }
  
  logNamespaceLoad(namespace, language) {
    this.log(`Namespace loaded: ${namespace} for ${language}`);
  }
  
  logRTLChange(isRTL) {
    this.log(`RTL mode changed to: ${isRTL}`);
  }
  
  getLogs() {
    return this.logs;
  }
  
  clearLogs() {
    this.logs = [];
  }
  
  exportLogs() {
    return JSON.stringify(this.logs, null, 2);
  }
}

export const i18nDebugger = new I18nDebugger();
```

#### **Componente de Debugging**

```javascript
// components/I18nDebugger.js
import React, { useState, useEffect } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  ScrollView, 
  TouchableOpacity 
} from 'react-native';
import { useTranslation } from 'react-i18next';
import { i18nDebugger } from '../utils/i18nDebugger';

const I18nDebugger = () => {
  const { i18n } = useTranslation();
  const [logs, setLogs] = useState([]);
  const [isVisible, setIsVisible] = useState(false);
  
  useEffect(() => {
    if (__DEV__) {
      const interval = setInterval(() => {
        setLogs(i18nDebugger.getLogs());
      }, 1000);
      
      return () => clearInterval(interval);
    }
  }, []);
  
  if (!__DEV__) {
    return null;
  }
  
  return (
    <View style={styles.container}>
      <TouchableOpacity 
        style={styles.toggleButton}
        onPress={() => setIsVisible(!isVisible)}
      >
        <Text style={styles.toggleButtonText}>
          {isVisible ? 'Hide' : 'Show'} i18n Debug
        </Text>
      </TouchableOpacity>
      
      {isVisible && (
        <View style={styles.debugPanel}>
          <Text style={styles.title}>i18n Debug Information</Text>
          
          <View style={styles.infoSection}>
            <Text style={styles.infoTitle}>Current Language:</Text>
            <Text style={styles.infoValue}>{i18n.language}</Text>
          </View>
          
          <View style={styles.infoSection}>
            <Text style={styles.infoTitle}>Available Languages:</Text>
            <Text style={styles.infoValue}>{i18n.languages.join(', ')}</Text>
          </View>
          
          <View style={styles.infoSection}>
            <Text style={styles.infoTitle}>Loaded Namespaces:</Text>
            <Text style={styles.infoValue}>
              {Object.keys(i18n.getResourceBundle(i18n.language, 'translation')).join(', ')}
            </Text>
          </View>
          
          <ScrollView style={styles.logsContainer}>
            <Text style={styles.logsTitle}>Debug Logs:</Text>
            {logs.map((log, index) => (
              <Text key={index} style={styles.logEntry}>
                {log.timestamp}: {log.message}
              </Text>
            ))}
          </ScrollView>
          
          <TouchableOpacity 
            style={styles.clearButton}
            onPress={() => {
              i18nDebugger.clearLogs();
              setLogs([]);
            }}
          >
            <Text style={styles.clearButtonText}>Clear Logs</Text>
          </TouchableOpacity>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    position: 'absolute',
    top: 50,
    right: 10,
    zIndex: 1000,
  },
  toggleButton: {
    backgroundColor: '#007AFF',
    padding: 8,
    borderRadius: 4,
  },
  toggleButtonText: {
    color: '#fff',
    fontSize: 12,
  },
  debugPanel: {
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,
    maxHeight: 400,
    width: 300,
  },
  title: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  infoSection: {
    marginBottom: 12,
  },
  infoTitle: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#333',
  },
  infoValue: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
  logsContainer: {
    maxHeight: 200,
    marginTop: 16,
  },
  logsTitle: {
    fontSize: 14,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  logEntry: {
    fontSize: 12,
    color: '#666',
    marginBottom: 4,
  },
  clearButton: {
    backgroundColor: '#FF3B30',
    padding: 8,
    borderRadius: 4,
    marginTop: 16,
  },
  clearButtonText: {
    color: '#fff',
    textAlign: 'center',
    fontSize: 12,
  },
});

export default I18nDebugger;
```

### **5. Mejores Prácticas de i18n**

#### **Estructura de Proyecto**

```javascript
// Estructura recomendada
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
│       │   └── profile.json
│       └── es/
│           ├── common.json
│           ├── auth.json
│           └── profile.json
├── components/
│   ├── LocalizedText.js
│   ├── RTLIcon.js
│   └── AdaptiveLayout.js
└── hooks/
    ├── useTranslation.js
    └── useRegionalFormatting.js
```

#### **Convenciones de Nomenclatura**

```javascript
// ✅ Buena nomenclatura
const translationKeys = {
  'auth.login.title': 'Sign In',
  'auth.login.form.email.label': 'Email',
  'auth.login.form.email.placeholder': 'Enter your email',
  'auth.login.form.email.error.required': 'Email is required',
  'auth.login.form.email.error.invalid': 'Please enter a valid email',
  'auth.login.button.submit': 'Sign In',
  'auth.login.button.forgot': 'Forgot Password?',
  'auth.login.button.signup': 'Don\'t have an account? Sign Up'
};

// ❌ Mala nomenclatura
const badTranslationKeys = {
  'title': 'Sign In',
  'emailLabel': 'Email',
  'emailPlaceholder': 'Enter your email',
  'emailError': 'Please enter a valid email',
  'submitButton': 'Sign In',
  'forgotButton': 'Forgot Password?',
  'signupButton': 'Don\'t have an account? Sign Up'
};
```

#### **Gestión de Estado**

```javascript
// hooks/useI18nState.js
import { useState, useEffect } from 'react';
import { useTranslation } from 'react-i18next';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const useI18nState = () => {
  const { i18n } = useTranslation();
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    initializeI18n();
  }, []);
  
  const initializeI18n = async () => {
    try {
      setIsLoading(true);
      
      // Cargar idioma guardado
      const savedLanguage = await AsyncStorage.getItem('language');
      if (savedLanguage) {
        await i18n.changeLanguage(savedLanguage);
      }
      
      // Precargar namespaces críticos
      await preloadCriticalNamespaces();
      
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  
  const changeLanguage = async (language) => {
    try {
      await i18n.changeLanguage(language);
      await AsyncStorage.setItem('language', language);
    } catch (err) {
      setError(err.message);
    }
  };
  
  const preloadCriticalNamespaces = async () => {
    const criticalNamespaces = ['common', 'auth'];
    const promises = criticalNamespaces.map(ns => 
      i18n.loadNamespaces(ns)
    );
    await Promise.all(promises);
  };
  
  return {
    currentLanguage: i18n.language,
    availableLanguages: i18n.languages,
    isLoading,
    error,
    changeLanguage,
    isRTL: i18n.dir() === 'rtl'
  };
};
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Suite de Testing Completa**

```javascript
// __tests__/i18n.test.js
import React from 'react';
import { render, screen } from '@testing-library/react-native';
import { I18nextProvider } from 'react-i18next';
import i18n from '../i18n';
import WelcomeScreen from '../screens/WelcomeScreen';

const renderWithI18n = (component, { locale = 'en' } = {}) => {
  i18n.changeLanguage(locale);
  return render(
    <I18nextProvider i18n={i18n}>
      {component}
    </I18nextProvider>
  );
};

describe('i18n Testing Suite', () => {
  beforeEach(() => {
    i18n.changeLanguage('en');
  });
  
  describe('Translation Rendering', () => {
    test('renders English text by default', () => {
      renderWithI18n(<WelcomeScreen />);
      expect(screen.getByText('Welcome to our app!')).toBeTruthy();
    });
    
    test('renders Spanish text when locale is es', () => {
      renderWithI18n(<WelcomeScreen />, { locale: 'es' });
      expect(screen.getByText('¡Bienvenido a nuestra aplicación!')).toBeTruthy();
    });
    
    test('handles missing translations gracefully', () => {
      i18n.changeLanguage('fr');
      renderWithI18n(<WelcomeScreen />);
      expect(screen.getByText('Welcome to our app!')).toBeTruthy();
    });
  });
  
  describe('Pluralization', () => {
    test('handles zero count', () => {
      const result = i18n.t('messages.count', { count: 0 });
      expect(result).toBe('No messages');
    });
    
    test('handles singular count', () => {
      const result = i18n.t('messages.count', { count: 1 });
      expect(result).toBe('1 message');
    });
    
    test('handles plural count', () => {
      const result = i18n.t('messages.count', { count: 5 });
      expect(result).toBe('5 messages');
    });
  });
  
  describe('Interpolation', () => {
    test('interpolates variables correctly', () => {
      const result = i18n.t('welcome.message', { name: 'John' });
      expect(result).toBe('Hello, John!');
    });
    
    test('handles missing variables gracefully', () => {
      const result = i18n.t('welcome.message', {});
      expect(result).toBe('Hello, {{name}}!');
    });
  });
});
```

### **Ejercicio 2: Validación Automática**

```javascript
// scripts/validateTranslations.js
import { translationValidator } from '../utils/translationValidator';
import { contentValidator } from '../utils/contentValidator';

const validateAllTranslations = async () => {
  console.log('Starting translation validation...');
  
  const baseLanguage = 'en';
  const targetLanguages = ['es', 'fr', 'de', 'ar'];
  
  // Validar estructura
  const structureValidation = translationValidator.validateStructure(
    baseLanguage, 
    targetLanguages
  );
  
  if (!structureValidation.isValid) {
    console.error('Structure validation failed:');
    structureValidation.errors.forEach(error => console.error(error));
  }
  
  // Validar contenido
  const contentValidation = contentValidator.validateTranslations({
    en: require('../locales/en/common.json'),
    es: require('../locales/es/common.json'),
    fr: require('../locales/fr/common.json'),
  });
  
  if (contentValidation.length > 0) {
    console.warn('Content validation issues:');
    contentValidation.forEach(issue => console.warn(issue));
  }
  
  console.log('Translation validation completed');
};

validateAllTranslations();
```

### **Ejercicio 3: Dashboard de Performance**

```javascript
// screens/PerformanceDashboard.js
import React, { useState, useEffect } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  ScrollView,
  TouchableOpacity 
} from 'react-native';
import { useTranslation } from 'react-i18next';
import { i18nDebugger } from '../utils/i18nDebugger';

const PerformanceDashboard = () => {
  const { i18n } = useTranslation();
  const [performanceMetrics, setPerformanceMetrics] = useState({});
  const [logs, setLogs] = useState([]);
  
  useEffect(() => {
    const interval = setInterval(() => {
      updatePerformanceMetrics();
      setLogs(i18nDebugger.getLogs());
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  const updatePerformanceMetrics = () => {
    const metrics = {
      currentLanguage: i18n.language,
      availableLanguages: i18n.languages.length,
      loadedNamespaces: Object.keys(i18n.getResourceBundle(i18n.language, 'translation')).length,
      memoryUsage: getMemoryUsage(),
      translationCache: getTranslationCacheSize(),
    };
    
    setPerformanceMetrics(metrics);
  };
  
  const getMemoryUsage = () => {
    // Simular uso de memoria
    return Math.random() * 100;
  };
  
  const getTranslationCacheSize = () => {
    // Simular tamaño de caché
    return Math.floor(Math.random() * 1000);
  };
  
  const clearCache = () => {
    // Limpiar caché de traducciones
    i18nDebugger.clearLogs();
    setLogs([]);
  };
  
  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Performance Dashboard</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Current Status</Text>
        <Text style={styles.metric}>
          Language: {performanceMetrics.currentLanguage}
        </Text>
        <Text style={styles.metric}>
          Available Languages: {performanceMetrics.availableLanguages}
        </Text>
        <Text style={styles.metric}>
          Loaded Namespaces: {performanceMetrics.loadedNamespaces}
        </Text>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance Metrics</Text>
        <Text style={styles.metric}>
          Memory Usage: {performanceMetrics.memoryUsage?.toFixed(2)}%
        </Text>
        <Text style={styles.metric}>
          Translation Cache: {performanceMetrics.translationCache} entries
        </Text>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Debug Logs</Text>
        <ScrollView style={styles.logsContainer}>
          {logs.map((log, index) => (
            <Text key={index} style={styles.logEntry}>
              {log.timestamp}: {log.message}
            </Text>
          ))}
        </ScrollView>
        
        <TouchableOpacity style={styles.clearButton} onPress={clearCache}>
          <Text style={styles.clearButtonText}>Clear Cache</Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    margin: 20,
  },
  section: {
    backgroundColor: '#fff',
    margin: 20,
    padding: 20,
    borderRadius: 12,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  metric: {
    fontSize: 16,
    marginBottom: 8,
    color: '#333',
  },
  logsContainer: {
    maxHeight: 200,
    marginBottom: 16,
  },
  logEntry: {
    fontSize: 12,
    color: '#666',
    marginBottom: 4,
  },
  clearButton: {
    backgroundColor: '#FF3B30',
    padding: 12,
    borderRadius: 8,
  },
  clearButtonText: {
    color: '#fff',
    textAlign: 'center',
    fontWeight: 'bold',
  },
});

export default PerformanceDashboard;
```

---

## 🧪 Testing

### **Testing de Performance**

```javascript
// __tests__/performance.test.js
import { performance } from 'perf_hooks';
import { translationLoader } from '../utils/translationLoader';

describe('Performance Testing', () => {
  test('loads namespaces efficiently', async () => {
    const start = performance.now();
    
    await translationLoader.preloadNamespaces(['common', 'auth'], 'en');
    
    const end = performance.now();
    const duration = end - start;
    
    expect(duration).toBeLessThan(1000); // Should load in less than 1 second
  });
  
  test('caches translations correctly', () => {
    const start = performance.now();
    
    // First call
    i18n.t('common.welcome');
    
    const firstCall = performance.now();
    
    // Second call (should be cached)
    i18n.t('common.welcome');
    
    const secondCall = performance.now();
    
    const firstDuration = firstCall - start;
    const secondDuration = secondCall - firstCall;
    
    expect(secondDuration).toBeLessThan(firstDuration);
  });
});
```

---

## 📱 Ejemplo Completo

### **Aplicación con Testing y Optimización**

```javascript
// App.js - Aplicación optimizada
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useTranslation } from 'react-i18next';
import './i18n';
import { i18nDebugger } from './utils/i18nDebugger';
import I18nDebugger from './components/I18nDebugger';

import WelcomeScreen from './screens/WelcomeScreen';
import PerformanceDashboard from './screens/PerformanceDashboard';

const Stack = createStackNavigator();

const App = () => {
  const { i18n } = useTranslation();
  
  useEffect(() => {
    initializeI18n();
  }, []);
  
  const initializeI18n = async () => {
    try {
      i18nDebugger.log('Initializing i18n...');
      
      // Configurar i18n
      await i18n.init();
      
      i18nDebugger.log('i18n initialized successfully');
      
    } catch (error) {
      i18nDebugger.log('i18n initialization failed', error);
    }
  };
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Welcome" component={WelcomeScreen} />
        <Stack.Screen name="Performance" component={PerformanceDashboard} />
      </Stack.Navigator>
      
      {__DEV__ && <I18nDebugger />}
    </NavigationContainer>
  );
};

export default App;
```

---

## 🎯 Resumen de la Clase

### **Conceptos Clave Aprendidos**

1. **Testing multilingüe**: Verificación de traducciones y funcionalidad
2. **Validación automática**: Verificación de estructura y contenido
3. **Optimización de performance**: Lazy loading y memoización
4. **Debugging**: Herramientas para diagnosticar problemas
5. **Mejores prácticas**: Estructura y convenciones recomendadas

### **Habilidades Desarrolladas**

- ✅ Testing de aplicaciones multilingües
- ✅ Validación automática de traducciones
- ✅ Optimización de performance
- ✅ Debugging de problemas de localización
- ✅ Aplicación de mejores prácticas

### **Próximos Pasos**

Has completado el Módulo 25 de Internacionalización. Los siguientes módulos cubrirán:
- Analytics Avanzados y Business Intelligence
- Microservicios y Backend para Móvil
- Gaming y Realidad Aumentada

---

## 📚 Recursos Adicionales

### **Herramientas de Testing**
- [Jest](https://jestjs.io/) - Framework de testing
- [React Testing Library](https://testing-library.com/docs/react-native-testing-library/intro/) - Testing de componentes
- [Detox](https://github.com/wix/Detox) - Testing E2E

### **Herramientas de Performance**
- [React DevTools Profiler](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)
- [Flipper](https://fbflipper.com/) - Herramienta de debugging
- [Metro Bundle Analyzer](https://github.com/facebook/metro/tree/main/packages/metro-symbolicate)

### **Recursos de i18n**
- [i18next Best Practices](https://www.i18next.com/overview/best-practices)
- [React Native i18n Guide](https://reactnative.dev/docs/internationalization)
- [RTL Support Guide](https://reactnative.dev/docs/internationalization#right-to-left-layouts)

---

**🎯 Objetivo Alcanzado**: Has aprendido a implementar testing completo, validación automática, optimización de performance y debugging para aplicaciones multilingües.

**💡 Consejo**: El testing y la optimización son fundamentales para aplicaciones i18n. Una aplicación bien probada y optimizada garantiza una experiencia de usuario consistente en todos los idiomas.

**🚀 ¡Has completado el Módulo 25 de Internacionalización! ¡Continúa con los siguientes módulos para seguir desarrollando tus habilidades!**
