# 📚 Clase 1: ¿Qué es React Native?

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [README del Módulo](README.md)
- **➡️ Siguiente**: [Clase 2: Diferencias con React Web](clase_2_diferencias_react_web.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Comprender qué es React Native** y sus fundamentos
2. **Entender la historia** y evolución de React Native
3. **Conocer las ventajas** y desventajas de React Native
4. **Identificar cuándo usar** React Native vs otras tecnologías
5. **Comprender el ecosistema** de React Native

---

## 📚 Contenido Teórico

### **¿Qué es React Native?**

**React Native** es un framework de desarrollo móvil creado por Facebook (ahora Meta) que permite desarrollar aplicaciones nativas para iOS y Android usando JavaScript y React.

### **Características Principales:**

1. **Desarrollo Multiplataforma**: Un solo código para iOS y Android
2. **Rendimiento Nativo**: Usa componentes nativos reales
3. **Hot Reloading**: Cambios en tiempo real durante el desarrollo
4. **Comunidad Activa**: Gran ecosistema de librerías y herramientas
5. **Integración Nativa**: Acceso a APIs nativas del dispositivo

### **Historia y Evolución:**

- **2013**: Facebook comienza el desarrollo interno
- **2015**: Lanzamiento público en React Conf
- **2016**: Versión 0.20 con mejoras significativas
- **2018**: React Native 0.60 con autolinking
- **2020**: Nueva arquitectura anunciada
- **2023**: React Native 0.72 con mejoras de rendimiento

---

## 💻 Implementación Práctica

### **1. Comparación de Tecnologías Móviles**

```javascript:src/examples/TechnologyComparison.js
// Ejemplo de comparación entre tecnologías móviles
const mobileTechnologies = {
  reactNative: {
    name: 'React Native',
    language: 'JavaScript/TypeScript',
    performance: 'Nativo',
    developmentSpeed: 'Alto',
    maintenance: 'Medio',
    cost: 'Medio',
    platforms: ['iOS', 'Android'],
    pros: [
      'Desarrollo multiplataforma',
      'Rendimiento nativo',
      'Hot reloading',
      'Gran comunidad',
      'Reutilización de código web'
    ],
    cons: [
      'Curva de aprendizaje',
      'Dependencias nativas',
      'Tamaño de bundle',
      'Debugging complejo'
    ]
  },
  
  native: {
    name: 'Desarrollo Nativo',
    language: 'Swift/Objective-C (iOS), Java/Kotlin (Android)',
    performance: 'Excelente',
    developmentSpeed: 'Bajo',
    maintenance: 'Alto',
    cost: 'Alto',
    platforms: ['iOS', 'Android'],
    pros: [
      'Máximo rendimiento',
      'Acceso completo a APIs nativas',
      'Mejor experiencia de usuario',
      'Soporte oficial de plataforma'
    ],
    cons: [
      'Desarrollo separado por plataforma',
      'Mayor tiempo de desarrollo',
      'Mayor costo de mantenimiento',
      'Diferentes equipos de desarrollo'
    ]
  },
  
  flutter: {
    name: 'Flutter',
    language: 'Dart',
    performance: 'Nativo',
    developmentSpeed: 'Alto',
    maintenance: 'Medio',
    cost: 'Medio',
    platforms: ['iOS', 'Android', 'Web', 'Desktop'],
    pros: [
      'Desarrollo multiplataforma',
      'Rendimiento nativo',
      'Widgets personalizables',
      'Hot reload',
      'Soporte de Google'
    ],
    cons: [
      'Lenguaje Dart menos popular',
      'Ecosistema más pequeño',
      'Menos librerías disponibles'
    ]
  }
};

// Función para comparar tecnologías
export const compareTechnologies = (tech1, tech2) => {
  const comparison = {
    performance: tech1.performance === tech2.performance ? 'Igual' : 
                 tech1.performance === 'Nativo' ? `${tech1.name} mejor` : `${tech2.name} mejor`,
    developmentSpeed: tech1.developmentSpeed === tech2.developmentSpeed ? 'Igual' : 
                     tech1.developmentSpeed === 'Alto' ? `${tech1.name} más rápido` : `${tech2.name} más rápido`,
    maintenance: tech1.maintenance === tech2.maintenance ? 'Igual' : 
                tech1.maintenance === 'Bajo' ? `${tech1.name} más fácil` : `${tech2.name} más fácil`
  };
  
  return comparison;
};

// Función para recomendar tecnología según necesidades
export const recommendTechnology = (requirements) => {
  const { budget, timeline, performance, platforms } = requirements;
  
  if (budget === 'low' && timeline === 'short' && platforms.length > 1) {
    return 'React Native';
  } else if (performance === 'critical' && budget === 'high') {
    return 'Desarrollo Nativo';
  } else if (platforms.includes('web') && platforms.includes('desktop')) {
    return 'Flutter';
  } else {
    return 'React Native';
  }
};
```

### **2. Componente de Información de React Native**

```javascript:src/components/ReactNativeInfo.js
import React from 'react';
import {
  View,
  Text,
  ScrollView,
  StyleSheet,
  TouchableOpacity,
  Linking
} from 'react-native';

const ReactNativeInfo = () => {
  const features = [
    {
      title: 'Desarrollo Multiplataforma',
      description: 'Escribe una vez, ejecuta en iOS y Android',
      icon: '📱',
      examples: ['Instagram', 'Facebook', 'Discord', 'Skype']
    },
    {
      title: 'Rendimiento Nativo',
      description: 'Usa componentes nativos reales, no WebViews',
      icon: '⚡',
      examples: ['60 FPS', 'Animaciones fluidas', 'Acceso a hardware']
    },
    {
      title: 'Hot Reloading',
      description: 'Ve cambios en tiempo real sin reiniciar la app',
      icon: '🔥',
      examples: ['Desarrollo rápido', 'Iteración rápida', 'Debugging eficiente']
    },
    {
      title: 'Ecosistema Rico',
      description: 'Miles de librerías y herramientas disponibles',
      icon: '🌐',
      examples: ['React Navigation', 'Redux', 'AsyncStorage', 'Realm']
    }
  ];

  const openDocumentation = () => {
    Linking.openURL('https://reactnative.dev/');
  };

  const openGitHub = () => {
    Linking.openURL('https://github.com/facebook/react-native');
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>React Native</Text>
        <Text style={styles.subtitle}>
          Desarrolla aplicaciones móviles nativas con JavaScript
        </Text>
      </View>

      <View style={styles.featuresContainer}>
        {features.map((feature, index) => (
          <View key={index} style={styles.featureCard}>
            <Text style={styles.featureIcon}>{feature.icon}</Text>
            <Text style={styles.featureTitle}>{feature.title}</Text>
            <Text style={styles.featureDescription}>
              {feature.description}
            </Text>
            <View style={styles.examplesContainer}>
              <Text style={styles.examplesTitle}>Ejemplos:</Text>
              {feature.examples.map((example, idx) => (
                <Text key={idx} style={styles.example}>
                  • {example}
                </Text>
              ))}
            </View>
          </View>
        ))}
      </View>

      <View style={styles.actionsContainer}>
        <TouchableOpacity style={styles.button} onPress={openDocumentation}>
          <Text style={styles.buttonText}>📚 Documentación Oficial</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={openGitHub}>
          <Text style={styles.buttonText}>🐙 Código en GitHub</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.statsContainer}>
        <Text style={styles.statsTitle}>Estadísticas de React Native</Text>
        <View style={styles.statRow}>
          <Text style={styles.statLabel}>GitHub Stars:</Text>
          <Text style={styles.statValue}>100K+</Text>
        </View>
        <View style={styles.statRow}>
          <Text style={styles.statLabel}>Contribuidores:</Text>
          <Text style={styles.statValue}>2,000+</Text>
        </View>
        <View style={styles.statRow}>
          <Text style={styles.statLabel}>Apps en Producción:</Text>
          <Text style={styles.statValue}>Millones</Text>
        </View>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  header: {
    backgroundColor: '#007AFF',
    padding: 20,
    alignItems: 'center'
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 8
  },
  subtitle: {
    fontSize: 16,
    color: 'white',
    textAlign: 'center',
    opacity: 0.9
  },
  featuresContainer: {
    padding: 16
  },
  featureCard: {
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 20,
    marginBottom: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 4
  },
  featureIcon: {
    fontSize: 32,
    marginBottom: 12
  },
  featureTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 8
  },
  featureDescription: {
    fontSize: 16,
    color: '#666',
    lineHeight: 24,
    marginBottom: 16
  },
  examplesContainer: {
    backgroundColor: '#f8f9fa',
    padding: 12,
    borderRadius: 8
  },
  examplesTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
    marginBottom: 8
  },
  example: {
    fontSize: 14,
    color: '#666',
    marginBottom: 4
  },
  actionsContainer: {
    padding: 16,
    gap: 12
  },
  button: {
    backgroundColor: '#34C759',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center'
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600'
  },
  statsContainer: {
    backgroundColor: 'white',
    margin: 16,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 4
  },
  statsTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 16,
    textAlign: 'center'
  },
  statRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  statLabel: {
    fontSize: 16,
    color: '#666'
  },
  statValue: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#007AFF'
  }
});

export default ReactNativeInfo;
```

---

## 🧪 Casos de Uso Prácticos

### **1. Aplicación de Información de React Native**

```javascript:src/screens/ReactNativeInfoScreen.js
import React from 'react';
import { SafeAreaView, StatusBar } from 'react-native';
import ReactNativeInfo from '../components/ReactNativeInfo';

const ReactNativeInfoScreen = () => {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <StatusBar barStyle="light-content" backgroundColor="#007AFF" />
      <ReactNativeInfo />
    </SafeAreaView>
  );
};

export default ReactNativeInfoScreen;
```

### **2. Hook para Información de Tecnologías**

```javascript:src/hooks/useTechnologyInfo.js
import { useState, useEffect } from 'react';

export const useTechnologyInfo = (technology) => {
  const [info, setInfo] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchTechnologyInfo = async () => {
      try {
        setLoading(true);
        
        // Simular llamada a API
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        const technologyData = {
          reactNative: {
            name: 'React Native',
            description: 'Framework para desarrollo móvil multiplataforma',
            year: 2015,
            company: 'Facebook (Meta)',
            language: 'JavaScript/TypeScript',
            platforms: ['iOS', 'Android'],
            website: 'https://reactnative.dev/'
          },
          flutter: {
            name: 'Flutter',
            description: 'Framework de UI multiplataforma de Google',
            year: 2017,
            company: 'Google',
            language: 'Dart',
            platforms: ['iOS', 'Android', 'Web', 'Desktop'],
            website: 'https://flutter.dev/'
          },
          native: {
            name: 'Desarrollo Nativo',
            description: 'Desarrollo específico para cada plataforma',
            year: 2008,
            company: 'Apple/Google',
            language: 'Swift/Java/Kotlin',
            platforms: ['iOS', 'Android'],
            website: 'https://developer.apple.com/'
          }
        };

        setInfo(technologyData[technology] || null);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchTechnologyInfo();
  }, [technology]);

  return { info, loading, error };
};
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Investigación de Tecnologías**
Investiga y compara React Native con otras tecnologías móviles como Flutter, Ionic y desarrollo nativo.

### **Ejercicio 2: Análisis de Apps**
Identifica 5 aplicaciones populares que usen React Native y analiza por qué eligieron esta tecnología.

### **Ejercicio 3: Ventajas y Desventajas**
Crea una lista detallada de las ventajas y desventajas de React Native para diferentes tipos de proyectos.

### **Ejercicio 4: Casos de Uso**
Investiga casos de uso específicos donde React Native es la mejor opción y donde no lo es.

---

## 🎯 Proyecto de la Clase

### **App de Información de Tecnologías Móviles**

Crea una aplicación que muestre información detallada sobre diferentes tecnologías de desarrollo móvil, incluyendo React Native.

**Requisitos:**
- Comparación visual entre tecnologías
- Información detallada de cada una
- Ejemplos de aplicaciones reales
- Sistema de navegación entre tecnologías
- Diseño atractivo y responsive

---

## 📚 Recursos Adicionales

### **Documentación:**
- [React Native Official Documentation](https://reactnative.dev/)
- [React Native GitHub Repository](https://github.com/facebook/react-native)
- [React Native Community](https://github.com/react-native-community)

### **Artículos:**
- [Why React Native?](https://reactnative.dev/docs/intro-react-native-components)
- [React Native vs Flutter](https://medium.com/@openminder/react-native-vs-flutter-which-one-to-choose-aa4c3c3d2e1c)
- [The State of React Native](https://www.stateofjs.com/en-us/libraries/mobile-and-desktop/)

### **Videos:**
- [React Native Introduction](https://www.youtube.com/watch?v=0-S5a0eXPoc)
- [React Native vs Native Development](https://www.youtube.com/watch?v=9ArhRkQqy3Q)

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Definición clara** de qué es React Native
2. **Historia y evolución** del framework
3. **Ventajas y desventajas** comparadas con otras tecnologías
4. **Casos de uso** apropiados para React Native
5. **Ecosistema** y comunidad de React Native

### **🚀 Próximos Pasos:**
- Entender las diferencias con React Web
- Configurar el entorno de desarrollo
- Crear tu primera aplicación React Native

### **💡 Conceptos Clave:**
- **Framework**: Conjunto de herramientas para desarrollo
- **Multiplataforma**: Funciona en múltiples sistemas operativos
- **Nativo**: Usa componentes del sistema operativo
- **JavaScript**: Lenguaje de programación principal
- **React**: Biblioteca base para interfaces de usuario

---

**🎯 Objetivo**: Comprender qué es React Native y cuándo es la mejor opción para tu proyecto móvil.

**💡 Consejo**: React Native es excelente para aplicaciones que necesitan funcionar en iOS y Android con un solo código base.
