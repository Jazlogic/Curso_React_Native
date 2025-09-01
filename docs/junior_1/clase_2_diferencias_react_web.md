# 📚 Clase 2: Diferencias con React Web

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 1: ¿Qué es React Native?](clase_1_que_es_react_native.md)
- **➡️ Siguiente**: [Clase 3: Configuración del Entorno](clase_3_configuracion_entorno.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Entender las diferencias fundamentales** entre React Native y React Web
2. **Comprender cómo funciona** el bridge de React Native
3. **Identificar componentes nativos** vs componentes web
4. **Conocer las limitaciones** y ventajas de cada plataforma
5. **Aplicar patrones de desarrollo** apropiados para cada caso

---

## 📚 Contenido Teórico

### **React Web vs React Native: Fundamentos**

Aunque ambos usan React, **React Native** y **React Web** son tecnologías fundamentalmente diferentes:

#### **React Web:**
- **Entorno**: Navegador web
- **Renderizado**: DOM (Document Object Model)
- **Componentes**: HTML estándar (div, span, p, etc.)
- **Estilos**: CSS
- **Navegación**: React Router, History API

#### **React Native:**
- **Entorno**: Dispositivo móvil nativo
- **Renderizado**: Componentes nativos (UIView, ViewGroup)
- **Componentes**: Componentes nativos (View, Text, Image, etc.)
- **Estilos**: JavaScript objects con Flexbox
- **Navegación**: React Navigation, APIs nativas

---

## 💻 Implementación Práctica

### **1. Comparación de Componentes**

```javascript:src/examples/ComponentComparison.js
// Ejemplo de comparación entre componentes de React Web y React Native

// ===== REACT WEB =====
const WebComponent = () => {
  return (
    <div className="container">
      <h1 className="title">Título de la Página</h1>
      <p className="description">
        Esta es una descripción en React Web usando HTML estándar.
      </p>
      <button className="button" onClick={() => alert('Click!')}>
        Botón Web
      </button>
      <input 
        type="text" 
        className="input" 
        placeholder="Escribe algo..."
      />
      <ul className="list">
        <li>Elemento 1</li>
        <li>Elemento 2</li>
        <li>Elemento 3</li>
      </ul>
    </div>
  );
};

// ===== REACT NATIVE =====
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  TextInput,
  FlatList,
  StyleSheet
} from 'react-native';

const NativeComponent = () => {
  const listItems = ['Elemento 1', 'Elemento 2', 'Elemento 3'];

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Título de la Página</Text>
      <Text style={styles.description}>
        Esta es una descripción en React Native usando componentes nativos.
      </Text>
      <TouchableOpacity 
        style={styles.button} 
        onPress={() => alert('Click!')}
      >
        <Text style={styles.buttonText}>Botón Nativo</Text>
      </TouchableOpacity>
      <TextInput 
        style={styles.input} 
        placeholder="Escribe algo..."
      />
      <FlatList
        data={listItems}
        keyExtractor={(item, index) => index.toString()}
        renderItem={({ item }) => (
          <Text style={styles.listItem}>{item}</Text>
        )}
        style={styles.list}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 12
  },
  description: {
    fontSize: 16,
    color: '#666',
    lineHeight: 24,
    marginBottom: 20
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 16
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600'
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    marginBottom: 16,
    backgroundColor: 'white'
  },
  list: {
    flex: 1
  },
  listItem: {
    fontSize: 16,
    color: '#333',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#eee'
  }
});

export { WebComponent, NativeComponent };
```

### **2. Mapeo de Componentes Web a Nativos**

```javascript:src/utils/componentMapping.js
// Mapeo de componentes HTML a componentes React Native
export const webToNativeMapping = {
  // Elementos de texto
  'h1': 'Text',
  'h2': 'Text', 
  'h3': 'Text',
  'h4': 'Text',
  'h5': 'Text',
  'h6': 'Text',
  'p': 'Text',
  'span': 'Text',
  'strong': 'Text',
  'em': 'Text',
  'label': 'Text',
  
  // Contenedores
  'div': 'View',
  'section': 'View',
  'article': 'View',
  'header': 'View',
  'footer': 'View',
  'main': 'View',
  'aside': 'View',
  'nav': 'View',
  
  // Listas
  'ul': 'FlatList',
  'ol': 'FlatList',
  'li': 'Text',
  
  // Formularios
  'form': 'View',
  'input': 'TextInput',
  'textarea': 'TextInput',
  'select': 'Picker',
  'button': 'TouchableOpacity',
  'a': 'TouchableOpacity',
  
  // Imágenes y multimedia
  'img': 'Image',
  'video': 'Video',
  'audio': 'Audio',
  
  // Tablas
  'table': 'View',
  'thead': 'View',
  'tbody': 'View',
  'tr': 'View',
  'th': 'Text',
  'td': 'Text'
};

// Función para convertir estilos CSS a estilos de React Native
export const convertCSSStyles = (cssStyles) => {
  const styleMap = {
    // Dimensiones
    'width': 'width',
    'height': 'height',
    'min-width': 'minWidth',
    'min-height': 'minHeight',
    'max-width': 'maxWidth',
    'max-height': 'maxHeight',
    
    // Márgenes y padding
    'margin': 'margin',
    'margin-top': 'marginTop',
    'margin-right': 'marginRight',
    'margin-bottom': 'marginBottom',
    'margin-left': 'marginLeft',
    'padding': 'padding',
    'padding-top': 'paddingTop',
    'padding-right': 'paddingRight',
    'padding-bottom': 'paddingBottom',
    'padding-left': 'paddingLeft',
    
    // Colores
    'background-color': 'backgroundColor',
    'color': 'color',
    'border-color': 'borderColor',
    
    // Bordes
    'border': 'borderWidth',
    'border-width': 'borderWidth',
    'border-style': 'borderStyle',
    'border-radius': 'borderRadius',
    
    // Tipografía
    'font-size': 'fontSize',
    'font-weight': 'fontWeight',
    'font-family': 'fontFamily',
    'text-align': 'textAlign',
    'line-height': 'lineHeight',
    
    // Flexbox
    'display': 'display',
    'flex-direction': 'flexDirection',
    'justify-content': 'justifyContent',
    'align-items': 'alignItems',
    'flex': 'flex',
    'flex-grow': 'flexGrow',
    'flex-shrink': 'flexShrink',
    'flex-basis': 'flexBasis'
  };
  
  const convertedStyles = {};
  
  Object.keys(cssStyles).forEach(cssProperty => {
    const reactNativeProperty = styleMap[cssProperty];
    if (reactNativeProperty) {
      convertedStyles[reactNativeProperty] = cssStyles[cssProperty];
    }
  });
  
  return convertedStyles;
};

// Función para convertir valores CSS a valores de React Native
export const convertCSSValues = (cssValue, property) => {
  // Convertir unidades CSS a números
  if (typeof cssValue === 'string') {
    // Remover unidades como px, em, rem, %
    const numericValue = parseFloat(cssValue);
    
    if (!isNaN(numericValue)) {
      // Para dimensiones, mantener el valor numérico
      if (['width', 'height', 'margin', 'padding', 'fontSize'].includes(property)) {
        return numericValue;
      }
      
      // Para otros valores, mantener el string
      return cssValue;
    }
  }
  
  return cssValue;
};
```

### **3. Hook para Detectar Plataforma**

```javascript:src/hooks/usePlatform.js
import { useState, useEffect } from 'react';
import { Platform } from 'react-native';

export const usePlatform = () => {
  const [platform, setPlatform] = useState(null);
  const [isWeb, setIsWeb] = useState(false);
  const [isIOS, setIsIOS] = useState(false);
  const [isAndroid, setIsAndroid] = useState(false);

  useEffect(() => {
    // Detectar si estamos en React Native
    if (Platform.OS) {
      setPlatform(Platform.OS);
      setIsIOS(Platform.OS === 'ios');
      setIsAndroid(Platform.OS === 'android');
      setIsWeb(false);
    } else {
      // Estamos en React Web
      setPlatform('web');
      setIsWeb(true);
      setIsIOS(false);
      setIsAndroid(false);
    }
  }, []);

  // Función para renderizar componentes según la plataforma
  const renderByPlatform = (components) => {
    const { web, ios, android, native } = components;
    
    if (isWeb && web) return web;
    if (isIOS && ios) return ios;
    if (isAndroid && android) return android;
    if (native) return native;
    
    // Fallback al componente web si existe
    return web || null;
  };

  // Función para aplicar estilos según la plataforma
  const getPlatformStyles = (styles) => {
    const { web, ios, android, native } = styles;
    
    if (isWeb && web) return web;
    if (isIOS && ios) return ios;
    if (isAndroid && android) return android;
    if (native) return native;
    
    // Fallback a estilos nativos si existen
    return native || {};
  };

  return {
    platform,
    isWeb,
    isIOS,
    isAndroid,
    renderByPlatform,
    getPlatformStyles
  };
};
```

### **4. Componente Adaptativo Multiplataforma**

```javascript:src/components/AdaptiveComponent.js
import React from 'react';
import { usePlatform } from '../hooks/usePlatform';

const AdaptiveComponent = ({ 
  webComponent, 
  iosComponent, 
  androidComponent, 
  nativeComponent,
  webStyles,
  iosStyles,
  androidStyles,
  nativeStyles,
  children,
  ...props 
}) => {
  const { renderByPlatform, getPlatformStyles } = usePlatform();

  // Renderizar componente según plataforma
  const Component = renderByPlatform({
    web: webComponent,
    ios: iosComponent,
    android: androidComponent,
    native: nativeComponent
  });

  // Obtener estilos según plataforma
  const styles = getPlatformStyles({
    web: webStyles,
    ios: iosStyles,
    android: androidStyles,
    native: nativeStyles
  });

  if (!Component) {
    return null;
  }

  return (
    <Component style={styles} {...props}>
      {children}
    </Component>
  );
};

export default AdaptiveComponent;
```

---

## 🧪 Casos de Uso Prácticos

### **1. Componente de Botón Adaptativo**

```javascript:src/components/AdaptiveButton.js
import React from 'react';
import { usePlatform } from '../hooks/usePlatform';

const AdaptiveButton = ({ 
  children, 
  onPress, 
  style, 
  webProps = {}, 
  nativeProps = {} 
}) => {
  const { isWeb, isIOS, isAndroid } = usePlatform();

  if (isWeb) {
    // Renderizar botón HTML para web
    return (
      <button 
        onClick={onPress}
        style={{
          padding: '12px 24px',
          backgroundColor: '#007AFF',
          color: 'white',
          border: 'none',
          borderRadius: '8px',
          fontSize: '16px',
          cursor: 'pointer',
          ...style,
          ...webProps
        }}
      >
        {children}
      </button>
    );
  }

  // Renderizar TouchableOpacity para React Native
  const { TouchableOpacity, Text } = require('react-native');
  
  return (
    <TouchableOpacity
      onPress={onPress}
      style={{
        padding: 12,
        backgroundColor: '#007AFF',
        borderRadius: 8,
        alignItems: 'center',
        justifyContent: 'center',
        ...style,
        ...nativeProps
      }}
    >
      <Text style={{
        color: 'white',
        fontSize: 16,
        fontWeight: '600'
      }}>
        {children}
      </Text>
    </TouchableOpacity>
  );
};

export default AdaptiveButton;
```

### **2. Hook para Navegación Adaptativa**

```javascript:src/hooks/useAdaptiveNavigation.js
import { useCallback } from 'react';
import { usePlatform } from './usePlatform';

export const useAdaptiveNavigation = () => {
  const { isWeb, isIOS, isAndroid } = usePlatform();

  const navigate = useCallback((route, params = {}) => {
    if (isWeb) {
      // Usar React Router para web
      const { useNavigate } = require('react-router-dom');
      const navigate = useNavigate();
      navigate(route, { state: params });
    } else {
      // Usar React Navigation para React Native
      const { useNavigation } = require('@react-navigation/native');
      const navigation = useNavigation();
      navigation.navigate(route, params);
    }
  }, [isWeb]);

  const goBack = useCallback(() => {
    if (isWeb) {
      // Usar React Router para web
      const { useNavigate } = require('react-router-dom');
      const navigate = useNavigate();
      navigate(-1);
    } else {
      // Usar React Navigation para React Native
      const { useNavigation } = require('@react-navigation/native');
      const navigation = useNavigation();
      navigation.goBack();
    }
  }, [isWeb]);

  return { navigate, goBack };
};
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Conversión de Componentes**
Convierte un componente de React Web a React Native, manteniendo la misma funcionalidad.

### **Ejercicio 2: Estilos Adaptativos**
Crea un sistema de estilos que se adapte automáticamente entre web y móvil.

### **Ejercicio 3: Navegación Multiplataforma**
Implementa un sistema de navegación que funcione tanto en web como en móvil.

### **Ejercicio 4: Componente Universal**
Crea un componente que se renderice de manera óptima en ambas plataformas.

---

## 🎯 Proyecto de la Clase

### **App Multiplataforma Adaptativa**

Crea una aplicación que funcione tanto en React Web como en React Native, demostrando las diferencias y similitudes.

**Requisitos:**
- Componentes que se adapten a cada plataforma
- Estilos específicos para web y móvil
- Navegación adaptativa
- Funcionalidades específicas de cada plataforma
- Diseño responsive y nativo

---

## 📚 Recursos Adicionales

### **Documentación:**
- [React Native Components](https://reactnative.dev/docs/components-and-apis)
- [React Web Components](https://reactjs.org/docs/components-and-props.html)
- [Platform-Specific Code](https://reactnative.dev/docs/platform-specific-code)

### **Artículos:**
- [React Native vs React Web](https://medium.com/@openminder/react-native-vs-react-web-which-one-to-choose-aa4c3c3d2e1c)
- [Building Cross-Platform Apps](https://reactnative.dev/docs/platform-specific-code)

### **Herramientas:**
- [React Native Web](https://github.com/necolas/react-native-web)
- [Expo](https://expo.dev/) - Para desarrollo multiplataforma

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Diferencias fundamentales** entre React Web y React Native
2. **Mapeo de componentes** entre ambas plataformas
3. **Sistema de estilos** adaptativo
4. **Patrones de desarrollo** multiplataforma
5. **Hooks y utilidades** para detectar plataforma

### **🚀 Próximos Pasos:**
- Configurar el entorno de desarrollo
- Crear tu primera aplicación React Native
- Aprender sobre componentes nativos

### **💡 Conceptos Clave:**
- **Bridge**: Comunicación entre JavaScript y código nativo
- **Componentes Nativos**: Elementos específicos de cada plataforma
- **Estilos Adaptativos**: CSS vs JavaScript objects
- **Plataforma**: Web vs iOS vs Android
- **Multiplataforma**: Código que funciona en múltiples entornos

---

**🎯 Objetivo**: Comprender las diferencias entre React Web y React Native para desarrollar aplicaciones multiplataforma efectivas.

**💡 Consejo**: Aunque React Native y React Web comparten conceptos, trátalos como tecnologías diferentes con sus propias mejores prácticas.
