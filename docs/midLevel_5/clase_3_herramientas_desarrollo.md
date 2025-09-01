# 🛠️ Clase 3: Herramientas de Desarrollo

## 🎯 Objetivos de la Clase
- Utilizar las herramientas de desarrollo de Expo eficientemente
- Configurar un entorno de desarrollo optimizado
- Implementar debugging y hot reloading
- Optimizar el flujo de desarrollo

---

## 📚 Contenido Teórico

### 1. Expo CLI - Herramienta Principal

#### **Instalación y Configuración**
```bash
# Instalación global (opcional)
npm install -g @expo/cli

# Verificar versión
npx @expo/cli --version

# Configurar alias (opcional)
alias expo="npx @expo/cli"
```

#### **Comandos Principales**
```bash
# Iniciar proyecto
npx expo start

# Iniciar con opciones específicas
npx expo start --tunnel        # Túnel para desarrollo remoto
npx expo start --localhost     # Solo localhost
npx expo start --lan          # Red local
npx expo start --offline      # Modo offline
npx expo start --web          # Previsualización web

# Gestión de proyectos
npx expo install              # Instalar dependencias compatibles
npx expo upgrade              # Actualizar SDK y dependencias
npx expo doctor               # Verificar configuración del proyecto
npx expo prebuild             # Generar código nativo (bare workflow)

# Builds y publicación
npx expo build:android        # Build para Android
npx expo build:ios            # Build para iOS
npx expo publish              # Publicar actualización OTA
```

#### **Configuración de Expo CLI**
```bash
# Configurar cuenta de Expo
npx expo login

# Ver información de la cuenta
npx expo whoami

# Cerrar sesión
npx expo logout

# Ver proyectos
npx expo projects:list
```

### 2. Expo DevTools - Herramientas del Navegador

#### **Acceso a DevTools**
```bash
# Iniciar con DevTools automáticamente
npx expo start

# Abrir DevTools manualmente
npx expo start --dev-client
```

#### **Funcionalidades de DevTools**
- **Metro Bundler**: Visualización del bundler en tiempo real
- **Logs**: Visualización de console.log y errores
- **Network**: Monitoreo de requests HTTP
- **Elements**: Inspector de componentes React Native
- **Performance**: Métricas de rendimiento
- **Storage**: Visualización de AsyncStorage

#### **Configuración de DevTools**
```javascript
// app.json
{
  "expo": {
    "developmentClient": {
      "silenceLaunchWarnings": true
    },
    "hooks": {
      "postPublish": [
        {
          "file": "sentry-expo/upload-sourcemaps",
          "config": {
            "organization": "your-org",
            "project": "your-project"
          }
        }
      ]
    }
  }
}
```

### 3. Debugging Avanzado

#### **Configuración de Debugging**
```javascript
// App.js
import { LogBox } from 'react-native';

// Ignorar warnings específicos
LogBox.ignoreLogs(['Warning: ...']);

// Configurar debugging en desarrollo
if (__DEV__) {
  console.log = (...args) => {
    // Logging personalizado para desarrollo
    if (args.length > 0) {
      console.info('🔍 DEBUG:', ...args);
    }
  };
}
```

#### **React Native Debugger**
```bash
# Instalar React Native Debugger
# macOS
brew install --cask react-native-debugger

# Windows/Linux
# Descargar desde: https://github.com/jhen0409/react-native-debugger/releases
```

#### **Configuración de Flipper**
```javascript
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

// Habilitar Flipper
config.resolver.platforms = ['ios', 'android', 'native', 'web'];

module.exports = config;
```

#### **Debugging con Chrome DevTools**
```javascript
// Habilitar debugging remoto
// En la app: Shake device -> Debug JS Remotely

// O programáticamente
import { DevSettings } from 'react-native';

const enableRemoteDebugging = () => {
  DevSettings.setIsShakeToShowDevMenuEnabled(true);
  DevSettings.setIsHotLoadingEnabled(true);
};
```

### 4. Hot Reloading y Fast Refresh

#### **Configuración de Fast Refresh**
```javascript
// app.json
{
  "expo": {
    "developmentClient": {
      "silenceLaunchWarnings": true
    },
    "hooks": {
      "postPublish": [
        {
          "file": "sentry-expo/upload-sourcemaps",
          "config": {
            "organization": "your-org",
            "project": "your-project"
          }
        }
      ]
    }
  }
}
```

#### **Optimización del Hot Reloading**
```javascript
// Componente optimizado para hot reloading
import React, { memo } from 'react';
import { View, Text, StyleSheet } from 'react-native';

const OptimizedComponent = memo(({ title, subtitle }) => {
  console.log('Componente renderizado:', title);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>{title}</Text>
      <Text style={styles.subtitle}>{subtitle}</Text>
    </View>
  );
});

const styles = StyleSheet.create({
  container: {
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
  },
});

export default OptimizedComponent;
```

#### **Configuración de Metro Bundler**
```javascript
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

// Configuración para desarrollo
config.transformer = {
  ...config.transformer,
  getTransformOptions: async () => ({
    transform: {
      experimentalImportSupport: false,
      inlineRequires: true,
    },
  }),
};

// Configuración de caché
config.cacheStores = [
  {
    name: 'metro-cache',
    type: 'file',
    options: {
      maxAge: 1000 * 60 * 60 * 24, // 24 horas
    },
  },
];

module.exports = config;
```

### 5. Entorno de Desarrollo Optimizado

#### **Configuración de Variables de Entorno**
```bash
# .env.development
EXPO_PUBLIC_API_URL=https://dev-api.miapp.com
EXPO_PUBLIC_ENVIRONMENT=development
EXPO_PUBLIC_DEBUG_MODE=true

# .env.production
EXPO_PUBLIC_API_URL=https://api.miapp.com
EXPO_PUBLIC_ENVIRONMENT=production
EXPO_PUBLIC_DEBUG_MODE=false
```

#### **Configuración de Scripts**
```json
{
  "scripts": {
    "start": "expo start",
    "start:dev": "expo start --dev-client",
    "start:prod": "expo start --no-dev",
    "start:tunnel": "expo start --tunnel",
    "start:web": "expo start --web",
    "android": "expo run:android",
    "ios": "expo run:ios",
    "web": "expo start --web",
    "build:android": "eas build --platform android",
    "build:ios": "eas build --platform ios",
    "publish": "expo publish",
    "doctor": "expo doctor",
    "upgrade": "expo upgrade"
  }
}
```

#### **Configuración de TypeScript**
```typescript
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@screens/*": ["src/screens/*"],
      "@services/*": ["src/services/*"],
      "@utils/*": ["src/utils/*"]
    }
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".expo/types/**/*.ts",
    "expo-env.d.ts"
  ]
}
```

### 6. Herramientas de Testing y Calidad

#### **ESLint y Prettier**
```bash
# Instalar herramientas de calidad
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier

# Configurar ESLint
npx eslint --init
```

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'expo',
    '@typescript-eslint/recommended',
    'prettier'
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error',
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'warn'
  }
};
```

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

#### **Husky para Git Hooks**
```bash
# Instalar Husky
npm install --save-dev husky lint-staged

# Configurar Husky
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración de Entorno Optimizado
Configura un entorno de desarrollo completamente optimizado:

**Requisitos:**
- Configurar Expo CLI con alias y shortcuts
- Configurar DevTools para debugging avanzado
- Implementar hot reloading optimizado
- Configurar variables de entorno por ambiente

**Implementación:**
```bash
# 1. Crear alias para Expo CLI
echo 'alias expo="npx @expo/cli"' >> ~/.bashrc
echo 'alias expo-dev="npx @expo/cli start --dev-client"' >> ~/.bashrc
echo 'alias expo-tunnel="npx @expo/cli start --tunnel"' >> ~/.bashrc

# 2. Configurar variables de entorno
touch .env.development .env.production .env.local

# 3. Configurar scripts personalizados
# Editar package.json con scripts avanzados
```

### Ejercicio 2: Sistema de Debugging Completo
Implementa un sistema de debugging robusto:

**Requisitos:**
- Configurar React Native Debugger
- Implementar logging personalizado
- Configurar Flipper para debugging nativo
- Implementar error boundaries

**Implementación:**
```javascript
// utils/logger.js
class Logger {
  static info(message, ...args) {
    if (__DEV__) {
      console.info(`ℹ️ INFO: ${message}`, ...args);
    }
  }

  static warn(message, ...args) {
    if (__DEV__) {
      console.warn(`⚠️ WARNING: ${message}`, ...args);
    }
  }

  static error(message, ...args) {
    if (__DEV__) {
      console.error(`❌ ERROR: ${message}`, ...args);
    }
  }

  static debug(message, ...args) {
    if (__DEV__) {
      console.debug(`🔍 DEBUG: ${message}`, ...args);
    }
  }
}

export default Logger;
```

### Ejercicio 3: Configuración de Calidad de Código
Configura herramientas de calidad automáticas:

**Requisitos:**
- Configurar ESLint con reglas personalizadas
- Implementar Prettier para formateo automático
- Configurar Husky para pre-commit hooks
- Implementar lint-staged para archivos modificados

**Implementación:**
```bash
# 1. Configurar ESLint
npx eslint --init

# 2. Configurar Prettier
echo '{"semi": true, "singleQuote": true}' > .prettierrc

# 3. Configurar Husky
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"

# 4. Configurar lint-staged en package.json
```

---

## 🔍 Puntos Clave

1. **Expo CLI** es la herramienta principal para gestionar proyectos Expo
2. **DevTools** proporcionan debugging avanzado en el navegador
3. **Fast Refresh** mejora significativamente la experiencia de desarrollo
4. **Metro Bundler** es configurable para optimizar el desarrollo
5. **Herramientas de calidad** automatizan la revisión de código
6. **Variables de entorno** permiten configuraciones por ambiente
7. **Git hooks** aseguran calidad antes de cada commit

---

## 📖 Recursos Adicionales

- [Expo CLI Documentation](https://docs.expo.dev/workflow/expo-cli/)
- [Expo DevTools](https://docs.expo.dev/workflow/development-builds/)
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)
- [Flipper Documentation](https://fbflipper.com/docs/)

---

## 🎉 ¡Clase Completada!

Has completado exitosamente la **Clase 3: Herramientas de Desarrollo** 🚀

**Lo que has aprendido:**
- ✅ Configuración avanzada de Expo CLI
- ✅ Uso de DevTools para debugging
- ✅ Optimización de hot reloading
- ✅ Configuración de entorno de desarrollo
- ✅ Herramientas de calidad de código

**Próximos pasos:**
- Practicar con las herramientas de debugging
- Optimizar tu flujo de desarrollo
- Continuar con la siguiente clase sobre servicios de Expo

¡Excelente trabajo! 🎯

---

## 🔗 Navegación del Módulo

### **Clase Anterior**: [Clase 2: Expo SDK y APIs](clase_2_expo_sdk_apis.md)
### **Clase Siguiente**: [Clase 4: Servicios de Expo](clase_4_servicios_expo.md)
### **Volver al Módulo**: [Módulo 8: Expo y Desarrollo Rápido](README.md)
### **Volver al Inicio**: [Índice Completo](../INDICE_COMPLETO.md)
