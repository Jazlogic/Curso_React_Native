# 🌍 Clase 1: Fundamentos de i18n

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás los conceptos fundamentales de internacionalización (i18n) en React Native, incluyendo la configuración de React Native i18next, estructura de archivos de traducción, pluralización, contextos e interpolación de variables.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Comprender** los conceptos básicos de i18n
2. **Configurar** React Native i18next en tu proyecto
3. **Crear** estructura de archivos de traducción
4. **Implementar** pluralización y contextos
5. **Usar** interpolación de variables en traducciones

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Conceptos de Internacionalización**

#### **¿Qué es i18n?**
La internacionalización (i18n) es el proceso de diseñar y desarrollar aplicaciones para que puedan adaptarse fácilmente a diferentes idiomas y regiones.

```javascript
// ❌ Sin i18n - Texto hardcodeado
const WelcomeScreen = () => (
  <View>
    <Text>Welcome to our app!</Text>
    <Text>You have 3 new messages</Text>
  </View>
);

// ✅ Con i18n - Texto traducible
const WelcomeScreen = () => {
  const { t } = useTranslation();
  
  return (
    <View>
      <Text>{t('welcome.title')}</Text>
      <Text>{t('messages.count', { count: 3 })}</Text>
    </View>
  );
};
```

#### **Diferencias entre i18n y l10n**
- **i18n (Internationalization)**: Preparar la aplicación para múltiples idiomas
- **l10n (Localization)**: Adaptar la aplicación a un idioma/región específica

#### **Beneficios de la Internacionalización**
- **Alcance global**: Llegar a mercados internacionales
- **Experiencia de usuario**: Interfaz en idioma nativo
- **Escalabilidad**: Fácil adición de nuevos idiomas
- **Mantenimiento**: Centralización de textos

### **2. React Native i18next**

#### **¿Qué es i18next?**
i18next es un framework de internacionalización para JavaScript que funciona en múltiples plataformas.

```bash
# Instalación
npm install react-i18next i18next i18next-browser-languagedetector
# o
yarn add react-i18next i18next i18next-browser-languagedetector
```

#### **Configuración Básica**

```javascript
// i18n/index.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

// Importar archivos de traducción
import en from './locales/en.json';
import es from './locales/es.json';
import fr from './locales/fr.json';

const resources = {
  en: { translation: en },
  es: { translation: es },
  fr: { translation: fr },
};

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources,
    fallbackLng: 'en',
    debug: __DEV__,
    
    interpolation: {
      escapeValue: false, // React ya escapa por defecto
    },
    
    detection: {
      order: ['device', 'storage', 'navigator'],
      caches: ['localStorage'],
    },
  });

export default i18n;
```

#### **Configuración Avanzada**

```javascript
// i18n/config.js
export const i18nConfig = {
  // Idiomas soportados
  supportedLngs: ['en', 'es', 'fr', 'de', 'ar'],
  
  // Idioma por defecto
  fallbackLng: 'en',
  
  // Namespaces para organizar traducciones
  defaultNS: 'common',
  ns: ['common', 'auth', 'profile', 'settings'],
  
  // Configuración de detección
  detection: {
    order: ['device', 'storage', 'navigator'],
    caches: ['localStorage'],
    lookupLocalStorage: 'i18nextLng',
  },
  
  // Configuración de interpolación
  interpolation: {
    escapeValue: false,
    formatSeparator: ',',
  },
  
  // Configuración de pluralización
  pluralSeparator: '_',
  contextSeparator: '_',
  
  // Configuración de desarrollo
  debug: __DEV__,
  saveMissing: __DEV__,
  missingKeyHandler: (lng, ns, key) => {
    console.warn(`Missing translation: ${lng}.${ns}.${key}`);
  },
};
```

### **3. Estructura de Archivos de Traducción**

#### **Organización de Archivos**

```
src/
├── i18n/
│   ├── index.js
│   ├── config.js
│   └── locales/
│       ├── en/
│       │   ├── common.json
│       │   ├── auth.json
│       │   └── profile.json
│       ├── es/
│       │   ├── common.json
│       │   ├── auth.json
│       │   └── profile.json
│       └── fr/
│           ├── common.json
│           ├── auth.json
│           └── profile.json
```

#### **Estructura de Archivos JSON**

```json
// locales/en/common.json
{
  "welcome": {
    "title": "Welcome to our app!",
    "subtitle": "Discover amazing features",
    "button": "Get Started"
  },
  "navigation": {
    "home": "Home",
    "profile": "Profile",
    "settings": "Settings"
  },
  "common": {
    "loading": "Loading...",
    "error": "Something went wrong",
    "retry": "Try Again",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete"
  }
}
```

```json
// locales/es/common.json
{
  "welcome": {
    "title": "¡Bienvenido a nuestra aplicación!",
    "subtitle": "Descubre características increíbles",
    "button": "Comenzar"
  },
  "navigation": {
    "home": "Inicio",
    "profile": "Perfil",
    "settings": "Configuración"
  },
  "common": {
    "loading": "Cargando...",
    "error": "Algo salió mal",
    "retry": "Intentar de Nuevo",
    "cancel": "Cancelar",
    "save": "Guardar",
    "delete": "Eliminar"
  }
}
```

#### **Estructura Anidada**

```json
// locales/en/auth.json
{
  "login": {
    "title": "Sign In",
    "form": {
      "email": {
        "label": "Email",
        "placeholder": "Enter your email",
        "error": "Please enter a valid email"
      },
      "password": {
        "label": "Password",
        "placeholder": "Enter your password",
        "error": "Password must be at least 8 characters"
      }
    },
    "button": "Sign In",
    "forgot": "Forgot Password?",
    "signup": "Don't have an account? Sign Up"
  }
}
```

### **4. Pluralización y Contextos**

#### **Pluralización**

```json
// locales/en/messages.json
{
  "messages": {
    "count_zero": "No messages",
    "count_one": "{{count}} message",
    "count_other": "{{count}} messages"
  },
  "items": {
    "count_zero": "No items",
    "count_one": "{{count}} item",
    "count_other": "{{count}} items"
  }
}
```

```javascript
// Uso de pluralización
const MessageCount = ({ count }) => {
  const { t } = useTranslation();
  
  return (
    <Text>{t('messages.count', { count })}</Text>
  );
};

// Ejemplos de uso:
// count = 0: "No messages"
// count = 1: "1 message"
// count = 5: "5 messages"
```

#### **Contextos**

```json
// locales/en/profile.json
{
  "user": {
    "status_online": "Online",
    "status_offline": "Offline",
    "status_away": "Away"
  },
  "button": {
    "edit": "Edit",
    "save": "Save",
    "cancel": "Cancel"
  }
}
```

```javascript
// Uso de contextos
const UserStatus = ({ status }) => {
  const { t } = useTranslation();
  
  return (
    <Text>{t(`user.status_${status}`)}</Text>
  );
};

// Ejemplos de uso:
// status = "online": "Online"
// status = "offline": "Offline"
// status = "away": "Away"
```

### **5. Interpolación de Variables**

#### **Interpolación Básica**

```json
// locales/en/profile.json
{
  "welcome": "Welcome, {{name}}!",
  "lastLogin": "Last login: {{date}}",
  "profile": {
    "age": "{{name}} is {{age}} years old",
    "location": "{{name}} lives in {{city}}, {{country}}"
  }
}
```

```javascript
// Uso de interpolación
const WelcomeMessage = ({ user }) => {
  const { t } = useTranslation();
  
  return (
    <View>
      <Text>{t('welcome', { name: user.name })}</Text>
      <Text>{t('lastLogin', { date: user.lastLogin })}</Text>
      <Text>{t('profile.age', { name: user.name, age: user.age })}</Text>
    </View>
  );
};
```

#### **Interpolación con Formateo**

```json
// locales/en/currency.json
{
  "price": "Price: {{price, currency}}",
  "discount": "{{discount, percent}} off",
  "date": "Order date: {{date, date}}"
}
```

```javascript
// Configuración de formateo
i18n.init({
  interpolation: {
    format: (value, format, lng) => {
      if (format === 'currency') {
        return new Intl.NumberFormat(lng, {
          style: 'currency',
          currency: 'USD'
        }).format(value);
      }
      if (format === 'percent') {
        return new Intl.NumberFormat(lng, {
          style: 'percent'
        }).format(value / 100);
      }
      if (format === 'date') {
        return new Intl.DateTimeFormat(lng).format(new Date(value));
      }
      return value;
    }
  }
});

// Uso
const ProductCard = ({ product }) => {
  const { t } = useTranslation();
  
  return (
    <View>
      <Text>{t('price', { price: product.price })}</Text>
      <Text>{t('discount', { discount: product.discount })}</Text>
      <Text>{t('date', { date: product.date })}</Text>
    </View>
  );
};
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Configuración Básica**

```javascript
// 1. Instalar dependencias
// npm install react-i18next i18next i18next-browser-languagedetector

// 2. Crear configuración i18n
// i18n/index.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

const resources = {
  en: {
    translation: {
      welcome: "Welcome to our app!",
      loading: "Loading...",
      error: "Something went wrong"
    }
  },
  es: {
    translation: {
      welcome: "¡Bienvenido a nuestra aplicación!",
      loading: "Cargando...",
      error: "Algo salió mal"
    }
  }
};

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources,
    fallbackLng: 'en',
    debug: __DEV__,
    interpolation: {
      escapeValue: false,
    },
  });

export default i18n;
```

### **Ejercicio 2: Componente con Traducciones**

```javascript
// components/WelcomeScreen.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { useTranslation } from 'react-i18next';

const WelcomeScreen = () => {
  const { t, i18n } = useTranslation();
  
  const changeLanguage = (lng) => {
    i18n.changeLanguage(lng);
  };
  
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ fontSize: 24, marginBottom: 20 }}>
        {t('welcome')}
      </Text>
      
      <Button
        title="English"
        onPress={() => changeLanguage('en')}
      />
      <Button
        title="Español"
        onPress={() => changeLanguage('es')}
      />
    </View>
  );
};

export default WelcomeScreen;
```

### **Ejercicio 3: Pluralización Avanzada**

```json
// locales/en/notifications.json
{
  "notifications": {
    "count_zero": "No notifications",
    "count_one": "{{count}} notification",
    "count_other": "{{count}} notifications"
  },
  "friends": {
    "count_zero": "No friends",
    "count_one": "{{count}} friend",
    "count_other": "{{count}} friends"
  }
}
```

```javascript
// components/NotificationBadge.js
import React from 'react';
import { View, Text } from 'react-native';
import { useTranslation } from 'react-i18next';

const NotificationBadge = ({ count }) => {
  const { t } = useTranslation();
  
  return (
    <View>
      <Text>{t('notifications.count', { count })}</Text>
    </View>
  );
};

export default NotificationBadge;
```

### **Ejercicio 4: Interpolación Compleja**

```json
// locales/en/user.json
{
  "profile": {
    "welcome": "Welcome back, {{name}}!",
    "lastActivity": "Last activity: {{date, date}}",
    "stats": {
      "posts": "{{count}} posts",
      "followers": "{{count}} followers",
      "following": "{{count}} following"
    }
  }
}
```

```javascript
// components/UserProfile.js
import React from 'react';
import { View, Text } from 'react-native';
import { useTranslation } from 'react-i18next';

const UserProfile = ({ user }) => {
  const { t } = useTranslation();
  
  return (
    <View>
      <Text>{t('profile.welcome', { name: user.name })}</Text>
      <Text>{t('profile.lastActivity', { date: user.lastActivity })}</Text>
      <Text>{t('profile.stats.posts', { count: user.posts })}</Text>
      <Text>{t('profile.stats.followers', { count: user.followers })}</Text>
      <Text>{t('profile.stats.following', { count: user.following })}</Text>
    </View>
  );
};

export default UserProfile;
```

---

## 🧪 Testing

### **Testing de Traducciones**

```javascript
// __tests__/i18n.test.js
import i18n from '../i18n';
import { render, screen } from '@testing-library/react-native';
import { I18nextProvider } from 'react-i18next';
import WelcomeScreen from '../components/WelcomeScreen';

const renderWithI18n = (component, { locale = 'en' } = {}) => {
  i18n.changeLanguage(locale);
  return render(
    <I18nextProvider i18n={i18n}>
      {component}
    </I18nextProvider>
  );
};

describe('i18n Testing', () => {
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
});
```

### **Testing de Pluralización**

```javascript
// __tests__/pluralization.test.js
import i18n from '../i18n';

describe('Pluralization Testing', () => {
  test('handles zero count', () => {
    i18n.changeLanguage('en');
    const result = i18n.t('notifications.count', { count: 0 });
    expect(result).toBe('No notifications');
  });
  
  test('handles singular count', () => {
    i18n.changeLanguage('en');
    const result = i18n.t('notifications.count', { count: 1 });
    expect(result).toBe('1 notification');
  });
  
  test('handles plural count', () => {
    i18n.changeLanguage('en');
    const result = i18n.t('notifications.count', { count: 5 });
    expect(result).toBe('5 notifications');
  });
});
```

---

## 📱 Ejemplo Completo

### **Aplicación de Ejemplo: App Multilingüe**

```javascript
// App.js
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useTranslation } from 'react-i18next';
import './i18n';

import WelcomeScreen from './screens/WelcomeScreen';
import ProfileScreen from './screens/ProfileScreen';
import SettingsScreen from './screens/SettingsScreen';

const Stack = createStackNavigator();

const App = () => {
  const { i18n } = useTranslation();
  
  useEffect(() => {
    // Cargar idioma guardado
    const savedLanguage = AsyncStorage.getItem('language');
    if (savedLanguage) {
      i18n.changeLanguage(savedLanguage);
    }
  }, []);
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Welcome" component={WelcomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

```javascript
// screens/SettingsScreen.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useTranslation } from 'react-i18next';
import AsyncStorage from '@react-native-async-storage/async-storage';

const SettingsScreen = () => {
  const { t, i18n } = useTranslation();
  
  const changeLanguage = async (lng) => {
    await i18n.changeLanguage(lng);
    await AsyncStorage.setItem('language', lng);
  };
  
  const languages = [
    { code: 'en', name: 'English' },
    { code: 'es', name: 'Español' },
    { code: 'fr', name: 'Français' },
  ];
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>{t('settings.language')}</Text>
      
      {languages.map((lang) => (
        <TouchableOpacity
          key={lang.code}
          style={[
            styles.languageButton,
            i18n.language === lang.code && styles.selectedLanguage
          ]}
          onPress={() => changeLanguage(lang.code)}
        >
          <Text style={styles.languageText}>{lang.name}</Text>
        </TouchableOpacity>
      ))}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  languageButton: {
    padding: 15,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    marginBottom: 10,
  },
  selectedLanguage: {
    backgroundColor: '#007AFF',
  },
  languageText: {
    fontSize: 16,
  },
});

export default SettingsScreen;
```

---

## 🎯 Resumen de la Clase

### **Conceptos Clave Aprendidos**

1. **Internacionalización (i18n)**: Proceso de preparar aplicaciones para múltiples idiomas
2. **React Native i18next**: Framework principal para i18n en React Native
3. **Estructura de archivos**: Organización de traducciones por idioma y namespace
4. **Pluralización**: Manejo de formas singulares y plurales
5. **Interpolación**: Inserción de variables dinámicas en traducciones

### **Habilidades Desarrolladas**

- ✅ Configuración de i18next en React Native
- ✅ Creación de archivos de traducción estructurados
- ✅ Implementación de pluralización
- ✅ Uso de interpolación de variables
- ✅ Testing de traducciones

### **Próximos Pasos**

En la siguiente clase aprenderás sobre:
- Gestión avanzada de archivos de traducción
- Organización de claves de traducción
- Workflow de traducción
- Validación de traducciones

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [i18next Documentation](https://www.i18next.com/)
- [React i18next](https://react.i18next.com/)
- [i18next Browser Language Detector](https://github.com/i18next/i18next-browser-languagedetector)

### **Herramientas Útiles**
- [i18next Scanner](https://github.com/i18next/i18next-scanner) - Extrae claves de traducción automáticamente
- [i18next Ally](https://github.com/lokalise/i18next-ally) - Extensión de VS Code para i18n
- [i18next Testing](https://github.com/i18next/i18next-testing) - Utilidades para testing

### **Ejemplos y Tutoriales**
- [React Native i18n Tutorial](https://reactnative.dev/docs/internationalization)
- [i18next Best Practices](https://www.i18next.com/overview/best-practices)
- [Pluralization Rules](https://www.i18next.com/translation-function/plurals)

---

**🎯 Objetivo Alcanzado**: Has aprendido los fundamentos de internacionalización en React Native y puedes configurar un sistema básico de traducciones.

**💡 Consejo**: La internacionalización es más que traducir texto. Es adaptar toda la experiencia del usuario a diferentes culturas y regiones.

**🚀 ¡Continúa con la siguiente clase para profundizar en la localización de contenido!**
