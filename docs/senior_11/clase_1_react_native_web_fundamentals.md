# Clase 1: React Native Web Fundamentals

## Objetivos de la Clase
- Comprender los fundamentos de React Native Web
- Configurar el entorno de desarrollo multiplataforma
- Entender las diferencias entre móvil y web
- Crear el primer proyecto multiplataforma

## 1. ¿Qué es React Native Web?

### Concepto
React Native Web es una implementación de React Native que permite ejecutar aplicaciones React Native en navegadores web usando DOM en lugar de componentes nativos.

### Beneficios
- **Code Sharing**: Reutilizar código entre móvil y web
- **Consistencia**: Misma API y componentes
- **Performance**: Optimizado para web
- **SEO**: Aplicaciones web indexables
- **PWA**: Soporte para Progressive Web Apps

### Limitaciones
- **APIs nativas**: No todas las APIs están disponibles
- **Performance**: Puede ser más lento que React puro
- **Bundle size**: Tamaño de bundle más grande
- **Complejidad**: Configuración más compleja

## 2. Configuración del Entorno

### Instalación de Dependencias
```bash
# Crear proyecto React Native
npx react-native init MyMultiplatformApp

# Instalar React Native Web
npm install react-native-web react-dom

# Instalar dependencias de desarrollo
npm install --save-dev @babel/core @babel/preset-env @babel/preset-react
npm install --save-dev webpack webpack-cli webpack-dev-server
npm install --save-dev html-webpack-plugin babel-loader
npm install --save-dev css-loader style-loader
```

### Configuración de Babel
```json
// babel.config.js
module.exports = {
  presets: [
    'module:metro-react-native-babel-preset',
    '@babel/preset-env',
    '@babel/preset-react',
  ],
  plugins: [
    'react-native-web/babel',
  ],
};
```

### Configuración de Webpack
```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './index.web.js',
  mode: 'development',
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  resolve: {
    alias: {
      'react-native$': 'react-native-web',
    },
    extensions: ['.web.js', '.js', '.web.ts', '.ts', '.web.tsx', '.tsx', '.json'],
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
  ],
  devServer: {
    static: {
      directory: path.join(__dirname, 'public'),
    },
    compress: true,
    port: 3000,
    hot: true,
  },
};
```

### Punto de Entrada Web
```javascript
// index.web.js
import { AppRegistry } from 'react-native';
import App from './App';

AppRegistry.registerComponent('MyMultiplatformApp', () => App);
AppRegistry.runApplication('MyMultiplatformApp', {
  initialProps: {},
  rootTag: document.getElementById('root'),
});
```

### HTML Template
```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Multiplatform App</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

## 3. Diferencias entre Móvil y Web

### Componentes Específicos
```typescript
// components/PlatformSpecific.tsx
import React from 'react';
import { Platform, View, Text } from 'react-native';

export const PlatformSpecific: React.FC = () => {
  return (
    <View>
      <Text>Platform: {Platform.OS}</Text>
      
      {Platform.OS === 'web' && (
        <Text>This is web-specific content</Text>
      )}
      
      {Platform.OS === 'ios' && (
        <Text>This is iOS-specific content</Text>
      )}
      
      {Platform.OS === 'android' && (
        <Text>This is Android-specific content</Text>
      )}
    </View>
  );
};
```

### Estilos Específicos por Plataforma
```typescript
// styles/PlatformStyles.ts
import { StyleSheet, Platform } from 'react-native';

export const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    ...Platform.select({
      web: {
        minHeight: '100vh',
        display: 'flex',
      },
      ios: {
        paddingTop: 44, // Status bar height
      },
      android: {
        paddingTop: 24, // Status bar height
      },
    }),
  },
  button: {
    padding: 12,
    borderRadius: 8,
    ...Platform.select({
      web: {
        cursor: 'pointer',
        userSelect: 'none',
      },
      default: {},
    }),
  },
});
```

### APIs Específicas por Plataforma
```typescript
// utils/PlatformAPI.ts
import { Platform, Linking, Alert } from 'react-native';

export const openURL = async (url: string) => {
  try {
    if (Platform.OS === 'web') {
      window.open(url, '_blank');
    } else {
      const supported = await Linking.canOpenURL(url);
      if (supported) {
        await Linking.openURL(url);
      } else {
        Alert.alert('Error', 'Cannot open URL');
      }
    }
  } catch (error) {
    console.error('Error opening URL:', error);
  }
};

export const shareContent = async (content: string) => {
  if (Platform.OS === 'web') {
    if (navigator.share) {
      await navigator.share({ text: content });
    } else {
      // Fallback para navegadores que no soportan Web Share API
      navigator.clipboard.writeText(content);
      alert('Content copied to clipboard');
    }
  } else {
    // Usar React Native Share para móvil
    const { share } = require('react-native');
    await share({ message: content });
  }
};
```

## 4. Configuración de Metro

### Metro Config para Web
```javascript
// metro.config.js
const { getDefaultConfig } = require('metro-config');

module.exports = (async () => {
  const {
    resolver: { sourceExts, assetExts },
  } = await getDefaultConfig();
  
  return {
    transformer: {
      babelTransformerPath: require.resolve('react-native-svg-transformer'),
    },
    resolver: {
      assetExts: assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...sourceExts, 'svg'],
      alias: {
        'react-native$': 'react-native-web',
      },
    },
  };
})();
```

### Scripts de Package.json
```json
{
  "scripts": {
    "start": "react-native start",
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "web": "webpack serve --mode development",
    "web:build": "webpack --mode production",
    "web:start": "webpack serve --mode development --open"
  }
}
```

## 5. Primer Proyecto Multiplataforma

### App Principal
```typescript
// App.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Platform,
  SafeAreaView,
} from 'react-native';

const App: React.FC = () => {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.content}>
        <Text style={styles.title}>
          Multiplatform Counter App
        </Text>
        
        <Text style={styles.platform}>
          Running on: {Platform.OS}
        </Text>
        
        <Text style={styles.counter}>
          Count: {count}
        </Text>
        
        <View style={styles.buttonContainer}>
          <TouchableOpacity
            style={[styles.button, styles.decrementButton]}
            onPress={decrement}
          >
            <Text style={styles.buttonText}>-</Text>
          </TouchableOpacity>
          
          <TouchableOpacity
            style={[styles.button, styles.incrementButton]}
            onPress={increment}
          >
            <Text style={styles.buttonText}>+</Text>
          </TouchableOpacity>
        </View>
      </View>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  content: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  platform: {
    fontSize: 16,
    color: '#666',
    marginBottom: 30,
  },
  counter: {
    fontSize: 48,
    fontWeight: 'bold',
    marginBottom: 40,
    color: '#333',
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 20,
  },
  button: {
    width: 60,
    height: 60,
    borderRadius: 30,
    justifyContent: 'center',
    alignItems: 'center',
    ...Platform.select({
      web: {
        cursor: 'pointer',
        userSelect: 'none',
      },
    }),
  },
  incrementButton: {
    backgroundColor: '#4CAF50',
  },
  decrementButton: {
    backgroundColor: '#f44336',
  },
  buttonText: {
    color: 'white',
    fontSize: 24,
    fontWeight: 'bold',
  },
});

export default App;
```

### Componente Responsive
```typescript
// components/ResponsiveComponent.tsx
import React from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';

const { width } = Dimensions.get('window');

export const ResponsiveComponent: React.FC = () => {
  const isTablet = width >= 768;
  const isDesktop = width >= 1024;

  return (
    <View style={[
      styles.container,
      isTablet && styles.tabletContainer,
      isDesktop && styles.desktopContainer,
    ]}>
      <Text style={[
        styles.text,
        isTablet && styles.tabletText,
        isDesktop && styles.desktopText,
      ]}>
        {isDesktop ? 'Desktop View' : isTablet ? 'Tablet View' : 'Mobile View'}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#e3f2fd',
    borderRadius: 8,
    margin: 8,
  },
  tabletContainer: {
    padding: 24,
    margin: 16,
  },
  desktopContainer: {
    padding: 32,
    margin: 24,
    maxWidth: 800,
    alignSelf: 'center',
  },
  text: {
    fontSize: 16,
    textAlign: 'center',
  },
  tabletText: {
    fontSize: 20,
  },
  desktopText: {
    fontSize: 24,
  },
});
```

## 6. Optimizaciones para Web

### Lazy Loading
```typescript
// components/LazyComponent.tsx
import React, { lazy, Suspense } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

export const LazyComponent: React.FC = () => {
  return (
    <View>
      <Text>This component loads immediately</Text>
      
      <Suspense fallback={
        <View style={{ padding: 20, alignItems: 'center' }}>
          <ActivityIndicator size="large" />
          <Text>Loading heavy component...</Text>
        </View>
      }>
        <HeavyComponent />
      </Suspense>
    </View>
  );
};
```

### Optimización de Bundle
```typescript
// utils/BundleOptimization.ts
import { Platform } from 'react-native';

// Cargar librerías solo cuando sea necesario
export const loadPlatformSpecificLibrary = async () => {
  if (Platform.OS === 'web') {
    const { default: webLibrary } = await import('./webLibrary');
    return webLibrary;
  } else {
    const { default: mobileLibrary } = await import('./mobileLibrary');
    return mobileLibrary;
  }
};

// Tree shaking para web
export const webOnlyFeatures = {
  // Estas funciones solo se incluirán en el bundle web
  enableWebFeatures: () => {
    if (Platform.OS === 'web') {
      // Implementar características específicas de web
      console.log('Web features enabled');
    }
  },
};
```

### Service Worker para PWA
```javascript
// public/sw.js
const CACHE_NAME = 'my-app-cache-v1';
const urlsToCache = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        if (response) {
          return response;
        }
        return fetch(event.request);
      })
  );
});
```

## 7. Testing Multiplataforma

### Test Setup
```typescript
// __tests__/setup.ts
import 'react-native-gesture-handler/jestSetup';

// Mock para React Native Web
jest.mock('react-native/Libraries/EventEmitter/NativeEventEmitter');

// Mock para APIs específicas de web
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

### Test de Componente
```typescript
// __tests__/App.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import App from '../App';

describe('App', () => {
  it('renders correctly', () => {
    const { getByText } = render(<App />);
    expect(getByText('Multiplatform Counter App')).toBeTruthy();
  });

  it('increments counter when + button is pressed', () => {
    const { getByText } = render(<App />);
    const incrementButton = getByText('+');
    
    fireEvent.press(incrementButton);
    expect(getByText('Count: 1')).toBeTruthy();
  });

  it('decrements counter when - button is pressed', () => {
    const { getByText } = render(<App />);
    const decrementButton = getByText('-');
    
    fireEvent.press(decrementButton);
    expect(getByText('Count: -1')).toBeTruthy();
  });
});
```

## 8. Mejores Prácticas

### Organización de Código
- **Separar lógica de plataforma**: Usar archivos `.web.js`, `.ios.js`, `.android.js`
- **Abstraer APIs**: Crear wrappers para APIs específicas de plataforma
- **Testing**: Probar en todas las plataformas objetivo
- **Performance**: Optimizar bundle size para web

### Consideraciones de UX
- **Responsive Design**: Adaptar UI a diferentes tamaños de pantalla
- **Gestos**: Implementar gestos apropiados para cada plataforma
- **Navegación**: Usar patrones de navegación nativos
- **Accesibilidad**: Asegurar accesibilidad en todas las plataformas

## Conclusión

React Native Web permite extender React Native al navegador web, pero requiere:

- **Configuración adecuada** del entorno de desarrollo
- **Comprensión** de las diferencias entre plataformas
- **Optimización** específica para cada plataforma
- **Testing** en múltiples entornos

En la siguiente clase exploraremos las plataformas adicionales como Windows y macOS.

## Tarea

1. Configurar un proyecto React Native Web desde cero
2. Crear un componente que se comporte diferente en web y móvil
3. Implementar lazy loading para componentes pesados
4. Configurar testing para la plataforma web
5. Optimizar el bundle size para web

## Enlaces Útiles

- [React Native Web](https://github.com/necolas/react-native-web)
- [Webpack Configuration](https://webpack.js.org/configuration/)
- [Babel Configuration](https://babeljs.io/docs/en/configuration)
- [Progressive Web Apps](https://web.dev/progressive-web-apps/)
