# 📚 Clase 2: Stack Navigator Avanzado

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 1: Navegación Básica](clase_1_navegacion_basica.md)
- **➡️ Siguiente**: [Clase 3: Tab Navigator](clase_3_tab_navigator.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Dominar las opciones avanzadas del Stack Navigator
- Aprender a personalizar transiciones y animaciones
- Implementar navegación con múltiples stacks
- Crear navegación anidada y compleja
- Optimizar el rendimiento de la navegación

---

## 📚 Contenido Teórico

### **¿Qué es el Stack Navigator Avanzado?**

El Stack Navigator avanzado es una extensión del Stack Navigator básico que permite configuraciones más sofisticadas, transiciones personalizadas, navegación anidada y optimizaciones de rendimiento. Es ideal para aplicaciones complejas que requieren un control granular sobre la navegación.

#### **Características avanzadas:**
- **Transiciones personalizadas**: Animaciones únicas entre pantallas
- **Navegación anidada**: Stacks dentro de otros stacks
- **Opciones dinámicas**: Cambiar configuración según el estado
- **Gestión de estado**: Control avanzado del historial de navegación
- **Optimización**: Lazy loading y memoización de pantallas

### **Tipos de Transiciones:**

#### **1. Transiciones por Defecto:**
- **Slide from right**: Deslizamiento desde la derecha (iOS)
- **Slide from bottom**: Deslizamiento desde abajo (Android)
- **Fade**: Desvanecimiento suave

#### **2. Transiciones Personalizadas:**
- **Modal**: Aparece desde abajo como modal
- **Card**: Efecto de tarjeta que se desliza
- **Slide from left**: Deslizamiento desde la izquierda
- **Scale**: Escalado con zoom

### **Navegación Anidada:**

La navegación anidada permite tener múltiples stacks de navegación funcionando simultáneamente. Esto es útil para:
- **Apps con tabs**: Cada tab tiene su propio stack
- **Flujos independientes**: Login, onboarding, contenido principal
- **Modales**: Pantallas que aparecen sobre el contenido principal

---

## 💻 Implementación Práctica

### **1. Stack Navigator con Transiciones Personalizadas**

```javascript:src/navigation/AdvancedStackNavigator.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator, TransitionPresets } from '@react-navigation/stack';

// Importamos las pantallas
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import ModalScreen from '../screens/ModalScreen';
import DetailScreen from '../screens/DetailScreen';

// Creamos el navegador de stack
const Stack = createStackNavigator();

// Componente principal del navegador avanzado
const AdvancedStackNavigator = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        // Configuración global del navegador
        screenOptions={{
          // Estilo del header por defecto
          headerStyle: {
            backgroundColor: '#007bff',
            elevation: 8,
            shadowColor: '#000',
            shadowOffset: { width: 0, height: 4 },
            shadowOpacity: 0.3,
            shadowRadius: 8,
          },
          headerTintColor: '#fff',
          headerTitleStyle: {
            fontWeight: 'bold',
            fontSize: 18,
          },
          // Configuración de transiciones globales
          ...TransitionPresets.SlideFromRightIOS, // Transición por defecto para iOS
          // Configuración de gestos
          gestureEnabled: true, // Habilitar gestos de navegación
          gestureDirection: 'horizontal', // Dirección de los gestos
          // Configuración de animaciones
          cardStyleInterpolator: ({ current, layouts }) => {
            return {
              cardStyle: {
                transform: [
                  {
                    translateX: current.progress.interpolate({
                      inputRange: [0, 1],
                      outputRange: [layouts.screen.width, 0],
                    }),
                  },
                ],
              },
            };
          },
        }}
      >
        {/* Pantalla de inicio con transición personalizada */}
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{
            title: 'Inicio',
            // Transición personalizada para esta pantalla
            ...TransitionPresets.SlideFromRightIOS,
            // Configuración específica del header
            headerRight: () => (
              <TouchableOpacity
                style={styles.headerButton}
                onPress={() => {
                  // Navegar a modal
                  navigation.navigate('Modal');
                }}
              >
                <Text style={styles.headerButtonText}>📱</Text>
              </TouchableOpacity>
            ),
          }}
        />
        
        {/* Pantalla de perfil con transición de modal */}
        <Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{
            title: 'Mi Perfil',
            // Transición de modal (aparece desde abajo)
            ...TransitionPresets.ModalSlideFromBottomIOS,
            // Configuración específica para modal
            presentation: 'modal', // Tipo de presentación
            headerStyle: {
              backgroundColor: '#28a745',
            },
          }}
        />
        
        {/* Pantalla de configuración con transición de tarjeta */}
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={{
            title: 'Configuración',
            // Transición de tarjeta (efecto 3D)
            ...TransitionPresets.SlideFromRightIOS,
            // Configuración de gestos específica
            gestureDirection: 'horizontal-inverted', // Gestos invertidos
            headerStyle: {
              backgroundColor: '#6c757d',
            },
          }}
        />
        
        {/* Pantalla modal con transición personalizada */}
        <Stack.Screen
          name="Modal"
          component={ModalScreen}
          options={{
            title: 'Modal',
            // Transición completamente personalizada
            presentation: 'modal',
            // Configuración de modal
            modalPresentationStyle: 'pageSheet', // Estilo de modal (iOS)
            // Transición personalizada
            cardStyleInterpolator: ({ current, next, layouts }) => {
              return {
                cardStyle: {
                  transform: [
                    {
                      translateY: current.progress.interpolate({
                        inputRange: [0, 1],
                        outputRange: [layouts.screen.height, 0],
                      }),
                    },
                    {
                      scale: current.progress.interpolate({
                        inputRange: [0, 1],
                        outputRange: [0.8, 1],
                      }),
                    },
                  ],
                  opacity: current.progress.interpolate({
                    inputRange: [0, 1],
                    outputRange: [0, 1],
                  }),
                },
                overlayStyle: {
                  opacity: current.progress.interpolate({
                    inputRange: [0, 1],
                    outputRange: [0, 0.5],
                  }),
                },
              };
            },
            // Configuración de gestos para modal
            gestureEnabled: true,
            gestureDirection: 'vertical',
            gestureResponseDistance: {
              vertical: 200, // Distancia mínima para activar gesto
            },
          }}
        />
        
        {/* Pantalla de detalles con transición de zoom */}
        <Stack.Screen
          name="Detail"
          component={DetailScreen}
          options={{
            title: 'Detalles',
            // Transición de zoom
            cardStyleInterpolator: ({ current, layouts }) => {
              return {
                cardStyle: {
                  transform: [
                    {
                      scale: current.progress.interpolate({
                        inputRange: [0, 1],
                        outputRange: [0.5, 1],
                      }),
                    },
                    {
                      translateY: current.progress.interpolate({
                        inputRange: [0, 1],
                        outputRange: [layouts.screen.height * 0.5, 0],
                      }),
                    },
                  ],
                  opacity: current.progress.interpolate({
                    inputRange: [0, 1],
                    outputRange: [0, 1],
                  }),
                },
              };
            },
            // Configuración de header transparente
            headerTransparent: true,
            headerTintColor: '#fff',
            headerStyle: {
              backgroundColor: 'transparent',
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
    marginRight: 15,
    padding: 8,
    borderRadius: 20,
    backgroundColor: 'rgba(255, 255, 255, 0.2)',
  },
  headerButtonText: {
    fontSize: 18,
    color: '#fff',
  },
});

export default AdvancedStackNavigator;
```

### **2. Navegación Anidada con Múltiples Stacks**

```javascript:src/navigation/NestedStackNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

// Importamos las pantallas
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import FeedScreen from '../screens/FeedScreen';
import SearchScreen from '../screens/SearchScreen';
import NotificationsScreen from '../screens/NotificationsScreen';

// Creamos los navegadores
const MainStack = createStackNavigator();
const HomeStack = createStackNavigator();
const ProfileStack = createStackNavigator();
const SettingsStack = createStackNavigator();
const TabNavigator = createBottomTabNavigator();

// Stack para la pantalla de inicio
const HomeStackNavigator = () => {
  return (
    <HomeStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#007bff',
        },
        headerTintColor: '#fff',
      }}
    >
      <HomeStack.Screen
        name="HomeMain"
        component={HomeScreen}
        options={{
          title: 'Inicio',
        }}
      />
      <HomeStack.Screen
        name="Feed"
        component={FeedScreen}
        options={{
          title: 'Feed',
        }}
      />
    </HomeStack.Navigator>
  );
};

// Stack para la pantalla de perfil
const ProfileStackNavigator = () => {
  return (
    <ProfileStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#28a745',
        },
        headerTintColor: '#fff',
      }}
    >
      <ProfileStack.Screen
        name="ProfileMain"
        component={ProfileScreen}
        options={{
          title: 'Mi Perfil',
        }}
      />
      <ProfileStack.Screen
        name="EditProfile"
        component={EditProfileScreen}
        options={{
          title: 'Editar Perfil',
        }}
      />
    </ProfileStack.Navigator>
  );
};

// Stack para la pantalla de configuración
const SettingsStackNavigator = () => {
  return (
    <SettingsStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#6c757d',
        },
        headerTintColor: '#fff',
      }}
    >
      <SettingsStack.Screen
        name="SettingsMain"
        component={SettingsScreen}
        options={{
          title: 'Configuración',
        }}
      />
      <SettingsStack.Screen
        name="Privacy"
        component={PrivacyScreen}
        options={{
          title: 'Privacidad',
        }}
      />
      <SettingsStack.Screen
        name="Security"
        component={SecurityScreen}
        options={{
          title: 'Seguridad',
        }}
      />
    </SettingsStack.Navigator>
  );
};

// Navegador de tabs
const TabNavigatorComponent = () => {
  return (
    <TabNavigator.Navigator
      screenOptions={{
        // Configuración global de tabs
        tabBarActiveTintColor: '#007bff',
        tabBarInactiveTintColor: '#6c757d',
        tabBarStyle: {
          backgroundColor: '#fff',
          borderTopWidth: 1,
          borderTopColor: '#e9ecef',
          elevation: 8,
          shadowColor: '#000',
          shadowOffset: { width: 0, height: -2 },
          shadowOpacity: 0.1,
          shadowRadius: 4,
        },
        // Configuración de iconos
        tabBarIconStyle: {
          fontSize: 24,
        },
      }}
    >
      {/* Tab de inicio */}
      <TabNavigator.Screen
        name="HomeTab"
        component={HomeStackNavigator}
        options={{
          title: 'Inicio',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ color, fontSize: size }}>🏠</Text>
          ),
          // Ocultar header del tab (ya que cada stack tiene su propio header)
          headerShown: false,
        }}
      />
      
      {/* Tab de búsqueda */}
      <TabNavigator.Screen
        name="Search"
        component={SearchScreen}
        options={{
          title: 'Búsqueda',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ color, fontSize: size }}>🔍</Text>
          ),
        }}
      />
      
      {/* Tab de notificaciones */}
      <TabNavigator.Screen
        name="Notifications"
        component={NotificationsScreen}
        options={{
          title: 'Notificaciones',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ color, fontSize: size }}>🔔</Text>
          ),
          // Badge para notificaciones no leídas
          tabBarBadge: 3,
        }}
      />
      
      {/* Tab de perfil */}
      <TabNavigator.Screen
        name="ProfileTab"
        component={ProfileStackNavigator}
        options={{
          title: 'Perfil',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ color, fontSize: size }}>👤</Text>
          ),
          headerShown: false,
        }}
      />
      
      {/* Tab de configuración */}
      <TabNavigator.Screen
        name="SettingsTab"
        component={SettingsStackNavigator}
        options={{
          title: 'Config',
          tabBarIcon: ({ color, size }) => (
            <Text style={{ color, fontSize: size }}>⚙️</Text>
          ),
          headerShown: false,
        }}
      />
    </TabNavigator.Navigator>
  );
};

// Navegador principal que contiene todo
const NestedStackNavigator = () => {
  return (
    <MainStack.Navigator
      screenOptions={{
        headerShown: false, // Ocultar header principal
      }}
    >
      {/* Pantalla principal con tabs */}
      <MainStack.Screen
        name="MainTabs"
        component={TabNavigatorComponent}
      />
      
      {/* Pantallas modales que aparecen sobre los tabs */}
      <MainStack.Screen
        name="Modal"
        component={ModalScreen}
        options={{
          presentation: 'modal',
          ...TransitionPresets.ModalSlideFromBottomIOS,
        }}
      />
      
      {/* Pantalla de login (fuera de los tabs) */}
      <MainStack.Screen
        name="Login"
        component={LoginScreen}
        options={{
          ...TransitionPresets.SlideFromRightIOS,
        }}
      />
    </MainStack.Navigator>
  );
};

export default NestedStackNavigator;
```

### **3. Hook Personalizado para Navegación Avanzada**

```javascript:src/hooks/useAdvancedNavigation.js
import { useNavigation, useRoute } from '@react-navigation/native';
import { useCallback, useRef, useEffect, useState } from 'react';

// Hook personalizado para navegación avanzada
const useAdvancedNavigation = () => {
  const navigation = useNavigation();
  const route = useRoute();
  
  // Estado para el historial de navegación
  const [navigationHistory, setNavigationHistory] = useState([]);
  
  // Estado para las opciones de pantalla
  const [screenOptions, setScreenOptions] = useState({});
  
  // Referencia para parámetros de navegación
  const navigationParams = useRef({});
  
  // Referencia para callbacks de navegación
  const navigationCallbacks = useRef({});
  
  // Efecto para registrar la navegación actual
  useEffect(() => {
    const currentRoute = {
      name: route.name,
      params: route.params,
      timestamp: Date.now(),
      options: route.params?.navigationOptions || {},
    };
    
    setNavigationHistory(prev => [...prev, currentRoute]);
    
    // Limpiar historial si es muy largo
    if (navigationHistory.length > 50) {
      setNavigationHistory(prev => prev.slice(-50));
    }
  }, [route.name, route.params]);
  
  // Función para navegar con transición personalizada
  const navigateWithTransition = useCallback((
    screenName, 
    params = {}, 
    transition = 'default'
  ) => {
    // Configurar transición personalizada
    const transitionConfig = getTransitionConfig(transition);
    
    // Navegar con configuración personalizada
    navigation.navigate(screenName, {
      ...params,
      navigationOptions: transitionConfig,
    });
  }, [navigation]);
  
  // Función para obtener configuración de transición
  const getTransitionConfig = useCallback((transition) => {
    switch (transition) {
      case 'modal':
        return {
          presentation: 'modal',
          ...TransitionPresets.ModalSlideFromBottomIOS,
        };
      case 'slideFromLeft':
        return {
          ...TransitionPresets.SlideFromLeftIOS,
        };
      case 'fade':
        return {
          cardStyleInterpolator: ({ current }) => ({
            cardStyle: {
              opacity: current.progress,
            },
          }),
        };
      case 'scale':
        return {
          cardStyleInterpolator: ({ current, layouts }) => ({
            cardStyle: {
              transform: [
                {
                  scale: current.progress.interpolate({
                    inputRange: [0, 1],
                    outputRange: [0.5, 1],
                  }),
                },
              ],
            },
          }),
        };
      default:
        return {};
    }
  }, []);
  
  // Función para navegar y reemplazar con animación
  const replaceWithAnimation = useCallback((
    screenName, 
    params = {}, 
    animation = 'slideFromRight'
  ) => {
    const transitionConfig = getTransitionConfig(animation);
    
    navigation.replace(screenName, {
      ...params,
      navigationOptions: transitionConfig,
    });
  }, [navigation, getTransitionConfig]);
  
  // Función para navegar y resetear con configuración
  const resetAndNavigate = useCallback((
    screenName, 
    params = {}, 
    resetConfig = {}
  ) => {
    const defaultResetConfig = {
      index: 0,
      routes: [{ name: screenName, params }],
    };
    
    navigation.reset({
      ...defaultResetConfig,
      ...resetConfig,
    });
  }, [navigation]);
  
  // Función para navegar a múltiples pantallas
  const navigateToMultiple = useCallback((screens) => {
    if (screens.length === 0) return;
    
    // Navegar a la primera pantalla
    navigation.navigate(screens[0].name, screens[0].params);
    
    // Programar navegación a las siguientes pantallas
    screens.slice(1).forEach((screen, index) => {
      setTimeout(() => {
        navigation.navigate(screen.name, screen.params);
      }, (index + 1) * 100); // Delay de 100ms entre pantallas
    });
  }, [navigation]);
  
  // Función para navegar con callback
  const navigateWithCallback = useCallback((
    screenName, 
    params = {}, 
    callback
  ) => {
    // Guardar callback
    if (callback) {
      navigationCallbacks.current[screenName] = callback;
    }
    
    // Navegar a la pantalla
    navigation.navigate(screenName, {
      ...params,
      onReturn: callback,
    });
  }, [navigation]);
  
  // Función para ejecutar callback de retorno
  const executeReturnCallback = useCallback((screenName, data = {}) => {
    const callback = navigationCallbacks.current[screenName];
    
    if (callback && typeof callback === 'function') {
      callback(data);
      
      // Limpiar callback después de ejecutarlo
      delete navigationCallbacks.current[screenName];
    }
  }, []);
  
  // Función para obtener pantalla anterior con filtro
  const getPreviousScreen = useCallback((filter = () => true) => {
    const filteredHistory = navigationHistory.filter(filter);
    
    if (filteredHistory.length > 1) {
      return filteredHistory[filteredHistory.length - 2];
    }
    
    return null;
  }, [navigationHistory]);
  
  // Función para verificar si una pantalla está en el historial
  const isScreenInHistory = useCallback((screenName) => {
    return navigationHistory.some(route => route.name === screenName);
  }, [navigationHistory]);
  
  // Función para obtener todas las pantallas de un tipo
  const getScreensByType = useCallback((screenType) => {
    return navigationHistory.filter(route => 
      route.name.includes(screenType)
    );
  }, [navigationHistory]);
  
  // Función para limpiar historial de pantallas específicas
  const clearScreenHistory = useCallback((screenNames) => {
    setNavigationHistory(prev => 
      prev.filter(route => !screenNames.includes(route.name))
    );
  }, []);
  
  // Función para obtener estadísticas de navegación
  const getNavigationStats = useCallback(() => {
    const stats = {
      totalScreens: navigationHistory.length,
      uniqueScreens: new Set(navigationHistory.map(r => r.name)).size,
      mostVisited: {},
      averageTime: 0,
    };
    
    // Contar visitas por pantalla
    navigationHistory.forEach(route => {
      stats.mostVisited[route.name] = (stats.mostVisited[route.name] || 0) + 1;
    });
    
    // Calcular tiempo promedio
    if (navigationHistory.length > 1) {
      const totalTime = navigationHistory.reduce((acc, route, index) => {
        if (index > 0) {
          return acc + (route.timestamp - navigationHistory[index - 1].timestamp);
        }
        return acc;
      }, 0);
      
      stats.averageTime = totalTime / (navigationHistory.length - 1);
    }
    
    return stats;
  }, [navigationHistory]);
  
  return {
    // Funciones de navegación avanzada
    navigateWithTransition,
    replaceWithAnimation,
    resetAndNavigate,
    navigateToMultiple,
    navigateWithCallback,
    executeReturnCallback,
    
    // Funciones de información
    getPreviousScreen,
    isScreenInHistory,
    getScreensByType,
    getNavigationStats,
    
    // Funciones de gestión
    clearScreenHistory,
    
    // Estado
    navigationHistory,
    screenOptions,
    
    // Objeto de navegación nativo
    navigation,
    route,
  };
};

export default useAdvancedNavigation;
```

### **4. Utilidades para Navegación Avanzada**

```javascript:src/utils/navigationUtils.js
// Utilidades para navegación avanzada en React Native

// Función para crear configuración de transición personalizada
export const createCustomTransition = (config) => {
  const {
    type = 'slide',
    direction = 'right',
    duration = 300,
    easing = 'ease',
    scale = false,
    opacity = true,
  } = config;
  
  let cardStyleInterpolator;
  
  switch (type) {
    case 'slide':
      cardStyleInterpolator = ({ current, layouts, next }) => {
        const translateX = current.progress.interpolate({
          inputRange: [0, 1],
          outputRange: [layouts.screen.width, 0],
        });
        
        const translateY = current.progress.interpolate({
          inputRange: [0, 1],
          outputRange: [layouts.screen.height, 0],
        });
        
        return {
          cardStyle: {
            transform: [
              {
                translateX: direction === 'horizontal' ? translateX : 0,
              },
              {
                translateY: direction === 'vertical' ? translateY : 0,
              },
              ...(scale ? [{
                scale: current.progress.interpolate({
                  inputRange: [0, 1],
                  outputRange: [0.8, 1],
                }),
              }] : []),
            ],
            ...(opacity ? {
              opacity: current.progress.interpolate({
                inputRange: [0, 1],
                outputRange: [0, 1],
              }),
            } : {}),
          },
          overlayStyle: {
            opacity: current.progress.interpolate({
              inputRange: [0, 1],
              outputRange: [0, 0.5],
            }),
          },
        };
      };
      break;
      
    case 'fade':
      cardStyleInterpolator = ({ current }) => ({
        cardStyle: {
          opacity: current.progress,
        },
      });
      break;
      
    case 'scale':
      cardStyleInterpolator = ({ current, layouts }) => ({
        cardStyle: {
          transform: [
            {
              scale: current.progress.interpolate({
                inputRange: [0, 1],
                outputRange: [0, 1],
              }),
            },
            {
              translateY: current.progress.interpolate({
                inputRange: [0, 1],
                outputRange: [layouts.screen.height * 0.5, 0],
              }),
            },
          ],
          opacity: current.progress.interpolate({
            inputRange: [0, 1],
            outputRange: [0, 1],
          }),
        },
      });
      break;
      
    default:
      cardStyleInterpolator = ({ current, layouts }) => ({
        cardStyle: {
          transform: [
            {
              translateX: current.progress.interpolate({
                inputRange: [0, 1],
                outputRange: [layouts.screen.width, 0],
              }),
            },
          ],
        },
      });
  }
  
  return {
    cardStyleInterpolator,
    transitionSpec: {
      open: {
        animation: 'timing',
        config: {
          duration,
          easing: easing === 'ease' ? Easing.ease : Easing.linear,
        },
      },
      close: {
        animation: 'timing',
        config: {
          duration,
          easing: easing === 'ease' ? Easing.ease : Easing.linear,
        },
      },
    },
  };
};

// Función para crear configuración de header personalizado
export const createCustomHeader = (config) => {
  const {
    backgroundColor = '#007bff',
    titleColor = '#fff',
    showBackButton = true,
    backButtonText = 'Atrás',
    rightButton = null,
    leftButton = null,
    transparent = false,
    elevation = 4,
  } = config;
  
  return {
    headerStyle: {
      backgroundColor: transparent ? 'transparent' : backgroundColor,
      elevation: transparent ? 0 : elevation,
      shadowColor: transparent ? 'transparent' : '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: transparent ? 0 : 0.25,
      shadowRadius: 4,
    },
    headerTintColor: titleColor,
    headerTitleStyle: {
      fontWeight: 'bold',
      fontSize: 18,
    },
    headerBackTitle: showBackButton ? backButtonText : null,
    headerBackTitleVisible: showBackButton,
    headerRight: rightButton,
    headerLeft: leftButton,
    headerTransparent: transparent,
  };
};

// Función para crear configuración de gestos
export const createGestureConfig = (config) => {
  const {
    enabled = true,
    direction = 'horizontal',
    responseDistance = 50,
    velocityThreshold = 500,
    activeOffsetX = [-20, 20],
    activeOffsetY = [-20, 20],
  } = config;
  
  return {
    gestureEnabled: enabled,
    gestureDirection: direction,
    gestureResponseDistance: responseDistance,
    gestureVelocityImpact: 0.3,
    gestureHandlerProps: {
      activeOffsetX,
      activeOffsetY,
      velocityThreshold,
    },
  };
};

// Función para validar configuración de navegación
export const validateNavigationConfig = (config) => {
  const errors = [];
  
  if (!config.screens || config.screens.length === 0) {
    errors.push('La configuración debe tener al menos una pantalla');
  }
  
  config.screens?.forEach((screen, index) => {
    if (!screen.name) {
      errors.push(`La pantalla ${index} debe tener un nombre`);
    }
    
    if (!screen.component) {
      errors.push(`La pantalla ${screen.name} debe tener un componente`);
    }
    
    if (screen.options && typeof screen.options !== 'object') {
      errors.push(`Las opciones de ${screen.name} deben ser un objeto`);
    }
  });
  
  return {
    isValid: errors.length === 0,
    errors,
  };
};

// Función para crear navegador optimizado
export const createOptimizedNavigator = (screens, options = {}) => {
  const {
    lazy = true,
    unmountOnBlur = false,
    freezeOnBlur = true,
    ...otherOptions
  } = options;
  
  return {
    screens,
    options: {
      lazy,
      unmountOnBlur,
      freezeOnBlur,
      ...otherOptions,
    },
  };
};

// Función para crear configuración de deep linking
export const createDeepLinkingConfig = (config) => {
  const {
    prefixes = ['myapp://', 'https://myapp.com'],
    config: linkingConfig = {},
  } = config;
  
  return {
    prefixes,
    config: {
      screens: linkingConfig.screens || {},
      ...linkingConfig,
    },
  };
};
```

---

## 🧪 Casos de Uso

### **Caso 1: Navegación con Transiciones Personalizadas**
```javascript
// Navegar con transición de modal
const handleOpenModal = () => {
  navigateWithTransition('Modal', { data: 'example' }, 'modal');
};

// Navegar con transición de escala
const handleOpenDetail = () => {
  navigateWithTransition('Detail', { id: 123 }, 'scale');
};
```

### **Caso 2: Navegación Anidada Compleja**
```javascript
// Navegar a múltiples pantallas en secuencia
const handleCompleteFlow = () => {
  navigateToMultiple([
    { name: 'Step1', params: { step: 1 } },
    { name: 'Step2', params: { step: 2 } },
    { name: 'Step3', params: { step: 3 } },
  ]);
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear Transiciones Personalizadas**
Implementa al menos 3 transiciones personalizadas diferentes entre pantallas.

### **Ejercicio 2: Navegación Anidada**
Crea una app con tabs, cada tab con su propio stack de navegación.

### **Ejercicio 3: Navegación con Callbacks**
Implementa un sistema donde una pantalla pueda enviar datos de vuelta a la anterior.

---

## 🚀 Proyecto de la Clase

### **App de Navegación Avanzada**

Crea una aplicación que demuestre:
- **Transiciones personalizadas**: Múltiples tipos de animaciones
- **Navegación anidada**: Stacks dentro de tabs
- **Gestos personalizados**: Navegación mediante gestos
- **Optimización**: Lazy loading y memoización

**Requisitos:**
1. Implementar al menos 5 tipos de transiciones diferentes
2. Crear navegación anidada con tabs y stacks
3. Usar el hook personalizado de navegación avanzada
4. Implementar gestos personalizados
5. Optimizar el rendimiento de la navegación

**Estructura sugerida:**
```
src/
├── navigation/
│   ├── AdvancedStackNavigator.js
│   ├── NestedStackNavigator.js
│   └── TabNavigator.js
├── screens/
│   ├── HomeScreen.js
│   ├── ProfileScreen.js
│   ├── SettingsScreen.js
│   ├── ModalScreen.js
│   └── DetailScreen.js
├── hooks/
│   └── useAdvancedNavigation.js
└── utils/
    └── navigationUtils.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Navigation Stack](https://reactnavigation.org/docs/stack-navigator)
- [Custom Transitions](https://reactnavigation.org/docs/stack-navigator#custom-transitions)

### **Artículos Recomendados:**
- "Navegación avanzada en React Native"
- "Cómo crear transiciones personalizadas"
- "Optimización de navegación en apps móviles"

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Transiciones personalizadas**: Animaciones únicas entre pantallas
- **Navegación anidada**: Múltiples stacks funcionando simultáneamente
- **Gestos personalizados**: Control granular de la navegación
- **Optimización**: Lazy loading y memoización para mejor rendimiento

### **Habilidades Desarrolladas:**
- ✅ Crear transiciones personalizadas complejas
- ✅ Implementar navegación anidada con múltiples stacks
- ✅ Configurar gestos personalizados de navegación
- ✅ Optimizar el rendimiento de la navegación
- ✅ Crear hooks personalizados para navegación avanzada

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Tab Navigator**, que te permitirá crear navegación por pestañas con múltiples stacks independientes.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Navegación Básica](clase_1_navegacion_basica.md)
- **➡️ Siguiente Clase**: [Tab Navigator](clase_3_tab_navigator.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
