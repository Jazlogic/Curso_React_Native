# 📚 Clase 1: Navegación Básica

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Módulo 2: Componentes Básicos](../junior_2/README.md)
- **➡️ Siguiente**: [Clase 2: Stack Navigator](clase_2_stack_navigator.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender los conceptos fundamentales de navegación en React Native
- Aprender a configurar React Navigation en un proyecto
- Dominar la navegación básica entre pantallas
- Crear una estructura de navegación simple
- Entender la diferencia entre navegación web y móvil

---

## 📚 Contenido Teórico

### **¿Qué es la Navegación en React Native?**

La navegación en React Native es el sistema que permite a los usuarios moverse entre diferentes pantallas o vistas dentro de una aplicación móvil. A diferencia de la navegación web tradicional, la navegación móvil debe ser intuitiva, rápida y seguir las convenciones de cada plataforma.

#### **Características principales:**
- **Navegación nativa**: Sigue los patrones de iOS y Android
- **Transiciones suaves**: Animaciones fluidas entre pantallas
- **Gestos táctiles**: Navegación mediante gestos del usuario
- **Historial de navegación**: Mantiene el estado de navegación
- **Deep linking**: Permite abrir pantallas específicas desde enlaces externos

### **Tipos de Navegación:**

#### **1. Navegación por Stack (Pila):**
- Pantallas se apilan una encima de otra
- Navegación hacia adelante y hacia atrás
- Ideal para flujos lineales (login → home → detalles)

#### **2. Navegación por Tabs:**
- Múltiples pantallas accesibles simultáneamente
- Navegación horizontal entre secciones
- Perfecto para apps con funcionalidades independientes

#### **3. Navegación por Drawer:**
- Menú lateral deslizable
- Acceso a todas las secciones principales
- Común en apps empresariales o con muchas funcionalidades

### **React Navigation vs Navegación Nativa:**

**React Navigation:**
- ✅ Fácil de implementar
- ✅ Multiplataforma
- ✅ Personalizable
- ✅ Gran comunidad
- ❌ Puede tener problemas de rendimiento en apps complejas

**Navegación Nativa:**
- ✅ Mejor rendimiento
- ✅ Comportamiento nativo perfecto
- ✅ Acceso completo a APIs nativas
- ❌ Más complejo de implementar
- ❌ Requiere código separado para cada plataforma

---

## 💻 Implementación Práctica

### **1. Configuración Inicial de React Navigation**

```bash:terminal
# Instalar dependencias necesarias
npm install @react-navigation/native @react-navigation/stack
npm install react-native-screens react-native-safe-area-context
npm install react-native-gesture-handler

# Para iOS, instalar pods
cd ios && pod install && cd ..
```

### **2. Configuración del Navegador Principal**

```javascript:src/navigation/AppNavigator.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

// Importamos las pantallas de la aplicación
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';

// Creamos el navegador de stack
const Stack = createStackNavigator();

// Componente principal del navegador
const AppNavigator = () => {
  return (
    // NavigationContainer es el contenedor principal que envuelve toda la navegación
    <NavigationContainer>
      {/* Stack.Navigator define cómo se organizan las pantallas */}
      <Stack.Navigator
        // Pantalla inicial de la aplicación
        initialRouteName="Home"
        // Configuración global del navegador
        screenOptions={{
          // Estilo del header por defecto
          headerStyle: {
            backgroundColor: '#007bff', // Color de fondo del header
            elevation: 4, // Sombra en Android
            shadowColor: '#000', // Color de sombra en iOS
            shadowOffset: { width: 0, height: 2 }, // Offset de la sombra
            shadowOpacity: 0.25, // Opacidad de la sombra
            shadowRadius: 4, // Radio de la sombra
          },
          // Estilo del texto del header
          headerTintColor: '#fff', // Color del texto del header
          // Estilo del título del header
          headerTitleStyle: {
            fontWeight: 'bold', // Peso de la fuente
            fontSize: 18, // Tamaño de la fuente
          },
          // Configuración del botón de retroceso
          headerBackTitle: 'Atrás', // Texto del botón de retroceso (iOS)
          headerBackTitleVisible: true, // Mostrar texto del botón de retroceso
        }}
      >
        {/* Definimos cada pantalla de la aplicación */}
        
        {/* Pantalla de inicio */}
        <Stack.Screen
          name="Home" // Nombre único de la pantalla
          component={HomeScreen} // Componente que se renderiza
          options={{
            title: 'Inicio', // Título que aparece en el header
            // Configuración específica para esta pantalla
            headerRight: () => (
              <TouchableOpacity
                style={styles.headerButton}
                onPress={() => {
                  // Acción del botón derecho
                  console.log('Botón derecho presionado');
                }}
              >
                <Text style={styles.headerButtonText}>⚙️</Text>
              </TouchableOpacity>
            ),
          }}
        />
        
        {/* Pantalla de perfil */}
        <Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{
            title: 'Mi Perfil',
            // Configuración específica para esta pantalla
            headerStyle: {
              backgroundColor: '#28a745', // Color diferente para esta pantalla
            },
          }}
        />
        
        {/* Pantalla de configuración */}
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={{
            title: 'Configuración',
            // Configuración específica para esta pantalla
            headerStyle: {
              backgroundColor: '#6c757d', // Color diferente para esta pantalla
            },
          }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

// Estilos para los botones del header
const styles = StyleSheet.create({
  headerButton: {
    marginRight: 15, // Margen derecho del botón
    padding: 8, // Padding interno del botón
  },
  headerButtonText: {
    fontSize: 20, // Tamaño del emoji
    color: '#fff', // Color del emoji
  },
});

export default AppNavigator;
```

### **3. Pantalla de Inicio con Navegación**

```javascript:src/screens/HomeScreen.js
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  SafeAreaView,
} from 'react-native';

// Componente de la pantalla de inicio
const HomeScreen = ({ navigation }) => {
  // Función para navegar al perfil
  const navigateToProfile = () => {
    // navigation.navigate() navega a una pantalla específica
    // Si la pantalla ya está en el stack, regresa a ella
    navigation.navigate('Profile', {
      // Parámetros que se pasan a la pantalla de destino
      userId: 123,
      userName: 'Juan Pérez',
    });
  };

  // Función para navegar a configuración
  const navigateToSettings = () => {
    // navigation.push() siempre crea una nueva instancia de la pantalla
    // Útil cuando quieres múltiples instancias de la misma pantalla
    navigation.push('Settings');
  };

  // Función para ir a la pantalla anterior
  const goBack = () => {
    // navigation.goBack() regresa a la pantalla anterior
    if (navigation.canGoBack()) {
      navigation.goBack();
    }
  };

  // Función para ir a la primera pantalla del stack
  const goToFirstScreen = () => {
    // navigation.popToTop() regresa a la primera pantalla del stack
    navigation.popToTop();
  };

  return (
    // SafeAreaView asegura que el contenido no se superponga con el notch o la barra de estado
    <SafeAreaView style={styles.container}>
      {/* Contenido principal de la pantalla */}
      <View style={styles.content}>
        {/* Título de bienvenida */}
        <Text style={styles.title}>¡Bienvenido a la App!</Text>
        
        {/* Descripción */}
        <Text style={styles.description}>
          Esta es la pantalla de inicio. Desde aquí puedes navegar a otras pantallas de la aplicación.
        </Text>
        
        {/* Botones de navegación */}
        <View style={styles.buttonContainer}>
          {/* Botón para ir al perfil */}
          <TouchableOpacity
            style={[styles.button, styles.primaryButton]}
            onPress={navigateToProfile}
          >
            <Text style={styles.buttonText}>Ver Mi Perfil</Text>
          </TouchableOpacity>
          
          {/* Botón para ir a configuración */}
          <TouchableOpacity
            style={[styles.button, styles.secondaryButton]}
            onPress={navigateToSettings}
          >
            <Text style={styles.buttonText}>Configuración</Text>
          </TouchableOpacity>
        </View>
        
        {/* Botones de navegación adicionales */}
        <View style={styles.navigationContainer}>
          <Text style={styles.sectionTitle}>Navegación Avanzada:</Text>
          
          {/* Botón para regresar */}
          <TouchableOpacity
            style={[styles.button, styles.infoButton]}
            onPress={goBack}
          >
            <Text style={styles.buttonText}>Regresar</Text>
          </TouchableOpacity>
          
          {/* Botón para ir al inicio del stack */}
          <TouchableOpacity
            style={[styles.button, styles.warningButton]}
            onPress={goToFirstScreen}
          >
            <Text style={styles.buttonText}>Ir al Inicio del Stack</Text>
          </TouchableOpacity>
        </View>
        
        {/* Información del estado de navegación */}
        <View style={styles.infoContainer}>
          <Text style={styles.infoTitle}>Información de Navegación:</Text>
          <Text style={styles.infoText}>
            • Puedes regresar: {navigation.canGoBack() ? 'Sí' : 'No'}
          </Text>
          <Text style={styles.infoText}>
            • Pantalla actual: {navigation.getCurrentRoute()?.name}
          </Text>
        </View>
      </View>
    </SafeAreaView>
  );
};

// Estilos del componente
const styles = StyleSheet.create({
  // Contenedor principal
  container: {
    flex: 1, // Ocupa toda la pantalla disponible
    backgroundColor: '#f8f9fa', // Color de fondo
  },
  
  // Contenido principal
  content: {
    flex: 1, // Ocupa todo el espacio disponible
    padding: 20, // Padding interno
    justifyContent: 'center', // Centra el contenido verticalmente
  },
  
  // Título principal
  title: {
    fontSize: 28, // Tamaño de fuente grande
    fontWeight: 'bold', // Peso de la fuente
    color: '#333', // Color del texto
    textAlign: 'center', // Centra el texto
    marginBottom: 20, // Espacio después del título
  },
  
  // Descripción
  description: {
    fontSize: 16, // Tamaño de fuente mediano
    color: '#666', // Color del texto
    textAlign: 'center', // Centra el texto
    lineHeight: 24, // Altura de línea para mejor legibilidad
    marginBottom: 40, // Espacio después de la descripción
  },
  
  // Contenedor de botones principales
  buttonContainer: {
    marginBottom: 40, // Espacio después de los botones
  },
  
  // Botón individual
  button: {
    paddingVertical: 15, // Padding vertical del botón
    paddingHorizontal: 30, // Padding horizontal del botón
    borderRadius: 10, // Bordes redondeados
    marginBottom: 15, // Espacio entre botones
    alignItems: 'center', // Centra el contenido horizontalmente
    elevation: 3, // Sombra en Android
    shadowColor: '#000', // Color de sombra en iOS
    shadowOffset: { width: 0, height: 2 }, // Offset de la sombra
    shadowOpacity: 0.25, // Opacidad de la sombra
    shadowRadius: 4, // Radio de la sombra
  },
  
  // Variantes de botones
  primaryButton: {
    backgroundColor: '#007bff', // Color azul primario
  },
  
  secondaryButton: {
    backgroundColor: '#6c757d', // Color gris secundario
  },
  
  infoButton: {
    backgroundColor: '#17a2b8', // Color azul informativo
  },
  
  warningButton: {
    backgroundColor: '#ffc107', // Color amarillo de advertencia
  },
  
  // Texto del botón
  buttonText: {
    color: '#fff', // Color del texto
    fontSize: 16, // Tamaño de fuente
    fontWeight: '600', // Peso de la fuente
  },
  
  // Contenedor de navegación avanzada
  navigationContainer: {
    marginBottom: 30, // Espacio después de la sección
  },
  
  // Título de sección
  sectionTitle: {
    fontSize: 18, // Tamaño de fuente
    fontWeight: '600', // Peso de la fuente
    color: '#333', // Color del texto
    marginBottom: 15, // Espacio después del título
    textAlign: 'center', // Centra el texto
  },
  
  // Contenedor de información
  infoContainer: {
    backgroundColor: '#e9ecef', // Color de fondo
    padding: 15, // Padding interno
    borderRadius: 10, // Bordes redondeados
    borderWidth: 1, // Grosor del borde
    borderColor: '#dee2e6', // Color del borde
  },
  
  // Título de información
  infoTitle: {
    fontSize: 16, // Tamaño de fuente
    fontWeight: '600', // Peso de la fuente
    color: '#333', // Color del texto
    marginBottom: 10, // Espacio después del título
  },
  
  // Texto de información
  infoText: {
    fontSize: 14, // Tamaño de fuente
    color: '#666', // Color del texto
    marginBottom: 5, // Espacio entre líneas
  },
});

export default HomeScreen;
```

### **4. Hook Personalizado para Navegación**

```javascript:src/hooks/useNavigation.js
import { useNavigation, useRoute } from '@react-navigation/native';
import { useCallback, useRef, useEffect } from 'react';

// Hook personalizado para manejar la navegación
const useCustomNavigation = () => {
  // Hook nativo de React Navigation
  const navigation = useNavigation();
  const route = useRoute();
  
  // Referencia para almacenar el historial de navegación
  const navigationHistory = useRef([]);
  
  // Referencia para almacenar parámetros de navegación
  const navigationParams = useRef({});
  
  // Efecto para registrar la navegación actual
  useEffect(() => {
    const currentRoute = {
      name: route.name,
      params: route.params,
      timestamp: Date.now(),
    };
    
    // Agregar la ruta actual al historial
    navigationHistory.current.push(currentRoute);
    
    // Limpiar historial si es muy largo (máximo 50 entradas)
    if (navigationHistory.current.length > 50) {
      navigationHistory.current = navigationHistory.current.slice(-50);
    }
  }, [route.name, route.params]);
  
  // Función para navegar a una pantalla con parámetros
  const navigateTo = useCallback((screenName, params = {}) => {
    // Guardar parámetros para uso posterior
    navigationParams.current[screenName] = params;
    
    // Navegar a la pantalla
    navigation.navigate(screenName, params);
  }, [navigation]);
  
  // Función para navegar y reemplazar la pantalla actual
  const navigateAndReplace = useCallback((screenName, params = {}) => {
    // Guardar parámetros
    navigationParams.current[screenName] = params;
    
    // Reemplazar la pantalla actual
    navigation.replace(screenName, params);
  }, [navigation]);
  
  // Función para navegar y resetear el stack
  const navigateAndReset = useCallback((screenName, params = {}) => {
    // Guardar parámetros
    navigationParams.current[screenName] = params;
    
    // Resetear el stack y navegar
    navigation.reset({
      index: 0,
      routes: [{ name: screenName, params }],
    });
  }, [navigation]);
  
  // Función para regresar a una pantalla específica
  const goBackTo = useCallback((screenName) => {
    // Buscar la pantalla en el historial
    const targetIndex = navigationHistory.current.findIndex(
      route => route.name === screenName
    );
    
    if (targetIndex !== -1) {
      // Calcular cuántas pantallas regresar
      const stepsBack = navigationHistory.current.length - targetIndex - 1;
      
      // Regresar la cantidad de pasos necesaria
      for (let i = 0; i < stepsBack; i++) {
        if (navigation.canGoBack()) {
          navigation.goBack();
        }
      }
    }
  }, [navigation]);
  
  // Función para obtener parámetros de una pantalla
  const getScreenParams = useCallback((screenName) => {
    return navigationParams.current[screenName] || {};
  }, []);
  
  // Función para obtener el historial de navegación
  const getNavigationHistory = useCallback(() => {
    return [...navigationHistory.current];
  }, []);
  
  // Función para limpiar el historial
  const clearHistory = useCallback(() => {
    navigationHistory.current = [];
    navigationParams.current = {};
  }, []);
  
  // Función para verificar si se puede regresar
  const canGoBack = useCallback(() => {
    return navigation.canGoBack();
  }, [navigation]);
  
  // Función para obtener la pantalla actual
  const getCurrentScreen = useCallback(() => {
    return {
      name: route.name,
      params: route.params,
    };
  }, [route.name, route.params]);
  
  // Función para obtener la pantalla anterior
  const getPreviousScreen = useCallback(() => {
    if (navigationHistory.current.length > 1) {
      return navigationHistory.current[navigationHistory.current.length - 2];
    }
    return null;
  }, []);
  
  return {
    // Funciones de navegación
    navigateTo,
    navigateAndReplace,
    navigateAndReset,
    goBackTo,
    
    // Funciones de información
    getScreenParams,
    getNavigationHistory,
    getCurrentScreen,
    getPreviousScreen,
    
    // Funciones de utilidad
    canGoBack,
    clearHistory,
    
    // Objeto de navegación nativo (para casos especiales)
    navigation,
    
    // Ruta actual
    route,
  };
};

export default useCustomNavigation;
```

---

## 🧪 Casos de Uso

### **Caso 1: Navegación con Parámetros**
```javascript
// En la pantalla de origen
const handleUserPress = (userId) => {
  navigation.navigate('UserProfile', {
    userId: userId,
    showEditButton: true,
  });
};

// En la pantalla de destino
const UserProfileScreen = ({ route, navigation }) => {
  const { userId, showEditButton } = route.params;
  
  return (
    <View>
      <Text>Perfil del usuario {userId}</Text>
      {showEditButton && (
        <Button title="Editar" onPress={() => {}} />
      )}
    </View>
  );
};
```

### **Caso 2: Navegación Condicional**
```javascript
const handleLogin = async (credentials) => {
  try {
    const user = await loginUser(credentials);
    
    if (user.isFirstTime) {
      // Usuario nuevo, ir a onboarding
      navigation.replace('Onboarding');
    } else {
      // Usuario existente, ir a home
      navigation.replace('Home');
    }
  } catch (error) {
    // Mostrar error
    Alert.alert('Error', error.message);
  }
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear un Flujo de Navegación Simple**
Crea una app con 3 pantallas:
- Pantalla de bienvenida
- Pantalla de login
- Pantalla principal

### **Ejercicio 2: Implementar Navegación con Parámetros**
Crea una lista de usuarios que al presionar navegue a un perfil con información del usuario.

### **Ejercicio 3: Navegación Condicional**
Implementa un sistema que navegue a diferentes pantallas según el estado del usuario (logueado/no logueado).

---

## 🚀 Proyecto de la Clase

### **App de Navegación Básica**

Crea una aplicación que demuestre:
- **Navegación entre pantallas**: Home, Profile, Settings
- **Paso de parámetros**: Entre pantallas
- **Navegación condicional**: Según el estado del usuario
- **Historial de navegación**: Mostrar rutas visitadas

**Requisitos:**
1. Configurar React Navigation correctamente
2. Crear al menos 3 pantallas con navegación
3. Implementar paso de parámetros entre pantallas
4. Usar el hook personalizado de navegación
5. Manejar estados de navegación

**Estructura sugerida:**
```
src/
├── navigation/
│   └── AppNavigator.js
├── screens/
│   ├── HomeScreen.js
│   ├── ProfileScreen.js
│   └── SettingsScreen.js
├── hooks/
│   └── useNavigation.js
└── App.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Navigation](https://reactnavigation.org/)
- [Getting Started](https://reactnavigation.org/docs/getting-started)

### **Artículos Recomendados:**
- "Guía completa de React Navigation"
- "Mejores prácticas para navegación móvil"
- "Cómo manejar parámetros entre pantallas"

### **Herramientas:**
- [React Navigation DevTools](https://github.com/react-navigation/devtools)
- [Flipper](https://fbflipper.com/) - Para debugging de navegación

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Navegación**: Sistema para moverse entre pantallas en apps móviles
- **React Navigation**: Librería principal para navegación en React Native
- **Stack Navigator**: Navegación por pila de pantallas
- **Parámetros**: Datos que se pasan entre pantallas
- **Historial**: Registro de pantallas visitadas

### **Habilidades Desarrolladas:**
- ✅ Configurar React Navigation en un proyecto
- ✅ Crear navegadores básicos con Stack Navigator
- ✅ Navegar entre pantallas con parámetros
- ✅ Crear hooks personalizados para navegación
- ✅ Manejar estados de navegación

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Stack Navigator Avanzado**, que te permitirá crear navegación más compleja con transiciones personalizadas y opciones avanzadas.

---

## 🔗 Enlaces de Navegación

- **⬅️ Módulo Anterior**: [Componentes Básicos](../junior_2/README.md)
- **➡️ Siguiente Clase**: [Stack Navigator](clase_2_stack_navigator.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
