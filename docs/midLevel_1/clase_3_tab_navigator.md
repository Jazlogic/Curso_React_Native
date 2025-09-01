# 📚 Clase 3: Tab Navigator

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 2: Stack Navigator Avanzado](clase_2_stack_navigator.md)
- **➡️ Siguiente**: [Clase 4: Drawer Navigator](clase_4_drawer_navigator.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el funcionamiento del Tab Navigator en React Native
- Aprender a crear navegación por pestañas con múltiples stacks
- Dominar la personalización de tabs y sus iconos
- Implementar navegación anidada con tabs y stacks
- Crear experiencias de usuario fluidas con tabs

---

## 📚 Contenido Teórico

### **¿Qué es el Tab Navigator?**

El Tab Navigator es un componente de React Navigation que permite crear navegación por pestañas en la parte inferior o superior de la pantalla. Es ideal para aplicaciones que tienen múltiples secciones principales que deben ser accesibles simultáneamente.

#### **Características principales:**
- **Navegación horizontal**: Cambio rápido entre secciones principales
- **Estado persistente**: Cada tab mantiene su propio estado
- **Iconos personalizables**: Representación visual de cada sección
- **Badges y notificaciones**: Indicadores de contenido nuevo
- **Navegación anidada**: Cada tab puede tener su propio stack

### **Tipos de Tab Navigator:**

#### **1. Bottom Tab Navigator:**
- Tabs en la parte inferior de la pantalla
- Estilo nativo de iOS y Android
- Fácil acceso con el pulgar
- Ideal para apps móviles

#### **2. Material Top Tab Navigator:**
- Tabs en la parte superior
- Estilo Material Design
- Deslizamiento horizontal entre tabs
- Perfecto para contenido relacionado

#### **3. Custom Tab Navigator:**
- Tabs completamente personalizados
- Posicionamiento libre
- Animaciones personalizadas
- Máxima flexibilidad

### **Ventajas del Tab Navigator:**

✅ **Acceso rápido**: Cambio inmediato entre secciones
✅ **Estado independiente**: Cada tab mantiene su navegación
✅ **UX familiar**: Patrón estándar en apps móviles
✅ **Escalabilidad**: Fácil agregar nuevas secciones
✅ **Performance**: Lazy loading de pantallas

---

## 💻 Implementación Práctica

### **1. Bottom Tab Navigator Básico**

```javascript:src/navigation/BasicTabNavigator.js
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Text, View, StyleSheet } from 'react-native';

// Importamos las pantallas
import HomeScreen from '../screens/HomeScreen';
import SearchScreen from '../screens/SearchScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';

// Creamos el navegador de tabs
const Tab = createBottomTabNavigator();

// Componente para iconos personalizados
const TabIcon = ({ name, focused, color, size }) => {
  // Emojis como iconos (en producción usarías react-native-vector-icons)
  const getIcon = () => {
    switch (name) {
      case 'Home':
        return focused ? '🏠' : '🏠';
      case 'Search':
        return focused ? '🔍' : '🔍';
      case 'Profile':
        return focused ? '👤' : '👤';
      case 'Settings':
        return focused ? '⚙️' : '⚙️';
      default:
        return '📱';
    }
  };

  return (
    <View style={styles.iconContainer}>
      <Text style={[styles.iconText, { color, fontSize: size }]}>
        {getIcon()}
      </Text>
      {/* Indicador de tab activo */}
      {focused && (
        <View style={[styles.activeIndicator, { backgroundColor: color }]} />
      )}
    </View>
  );
};

// Componente principal del navegador de tabs
const BasicTabNavigator = () => {
  return (
    <Tab.Navigator
      // Configuración global de los tabs
      screenOptions={{
        // Colores de los tabs
        tabBarActiveTintColor: '#007bff', // Color del tab activo
        tabBarInactiveTintColor: '#6c757d', // Color del tab inactivo
        
        // Estilo de la barra de tabs
        tabBarStyle: {
          backgroundColor: '#ffffff', // Color de fondo
          borderTopWidth: 1, // Grosor del borde superior
          borderTopColor: '#e9ecef', // Color del borde
          paddingBottom: 5, // Padding inferior
          paddingTop: 5, // Padding superior
          height: 60, // Altura de la barra
          elevation: 8, // Sombra en Android
          shadowColor: '#000', // Color de sombra en iOS
          shadowOffset: { width: 0, height: -2 }, // Offset de sombra
          shadowOpacity: 0.1, // Opacidad de sombra
          shadowRadius: 4, // Radio de sombra
        },
        
        // Estilo de las etiquetas de los tabs
        tabBarLabelStyle: {
          fontSize: 12, // Tamaño de fuente
          fontWeight: '600', // Peso de la fuente
          marginTop: 2, // Margen superior
        },
        
        // Estilo de los iconos de los tabs
        tabBarIconStyle: {
          marginBottom: 2, // Margen inferior
        },
        
        // Configuración del header
        headerShown: false, // Ocultar header (cada pantalla tendrá su propio header)
        
        // Configuración de gestos
        tabBarHideOnKeyboard: true, // Ocultar tabs cuando aparece el teclado
      }}
    >
      {/* Tab de inicio */}
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          title: 'Inicio', // Título del tab
          // Icono personalizado para este tab
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon name="Home" focused={focused} color={color} size={size} />
          ),
          // Configuración específica del tab
          tabBarBadge: undefined, // Sin badge por defecto
          tabBarBadgeStyle: {
            backgroundColor: '#dc3545', // Color del badge
            color: '#ffffff', // Color del texto del badge
          },
        }}
      />
      
      {/* Tab de búsqueda */}
      <Tab.Screen
        name="Search"
        component={SearchScreen}
        options={{
          title: 'Búsqueda',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon name="Search" focused={focused} color={color} size={size} />
          ),
          // Badge para mostrar resultados de búsqueda
          tabBarBadge: 5, // Número de resultados
        }}
      />
      
      {/* Tab de perfil */}
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          title: 'Perfil',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon name="Profile" focused={focused} color={color} size={size} />
          ),
          // Badge para notificaciones del perfil
          tabBarBadge: 2, // Notificaciones no leídas
        }}
      />
      
      {/* Tab de configuración */}
      <Tab.Screen
        name="Settings"
        component={SettingsScreen}
        options={{
          title: 'Config',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon name="Settings" focused={focused} color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
};

// Estilos para los iconos de los tabs
const styles = StyleSheet.create({
  iconContainer: {
    alignItems: 'center', // Centra horizontalmente
    justifyContent: 'center', // Centra verticalmente
    position: 'relative', // Para posicionar el indicador activo
  },
  
  iconText: {
    textAlign: 'center', // Centra el emoji
  },
  
  activeIndicator: {
    position: 'absolute', // Posicionamiento absoluto
    bottom: -8, // Debajo del icono
    width: 4, // Ancho del indicador
    height: 4, // Alto del indicador
    borderRadius: 2, // Forma circular
  },
});

export default BasicTabNavigator;
```

### **2. Tab Navigator con Navegación Anidada**

```javascript:src/navigation/NestedTabNavigator.js
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createStackNavigator } from '@react-navigation/stack';
import { Text, View, StyleSheet } from 'react-native';

// Importamos las pantallas
import HomeScreen from '../screens/HomeScreen';
import HomeDetailScreen from '../screens/HomeDetailScreen';
import SearchScreen from '../screens/SearchScreen';
import SearchResultsScreen from '../screens/SearchResultsScreen';
import ProfileScreen from '../screens/ProfileScreen';
import EditProfileScreen from '../screens/EditProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import PrivacyScreen from '../screens/PrivacyScreen';

// Creamos los navegadores
const Tab = createBottomTabNavigator();
const HomeStack = createStackNavigator();
const SearchStack = createStackNavigator();
const ProfileStack = createStackNavigator();
const SettingsStack = createStackNavigator();

// Stack para la pantalla de inicio
const HomeStackNavigator = () => {
  return (
    <HomeStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#007bff',
        },
        headerTintColor: '#fff',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
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
        name="HomeDetail"
        component={HomeDetailScreen}
        options={{
          title: 'Detalles',
        }}
      />
    </HomeStack.Navigator>
  );
};

// Stack para la pantalla de búsqueda
const SearchStackNavigator = () => {
  return (
    <SearchStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#28a745',
        },
        headerTintColor: '#fff',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
      }}
    >
      <SearchStack.Screen
        name="SearchMain"
        component={SearchScreen}
        options={{
          title: 'Búsqueda',
        }}
      />
      <SearchStack.Screen
        name="SearchResults"
        component={SearchResultsScreen}
        options={{
          title: 'Resultados',
        }}
      />
    </SearchStack.Navigator>
  );
};

// Stack para la pantalla de perfil
const ProfileStackNavigator = () => {
  return (
    <ProfileStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#ffc107',
        },
        headerTintColor: '#000',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
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
        headerTitleStyle: {
          fontWeight: 'bold',
        },
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
    </SettingsStack.Navigator>
  );
};

// Componente principal del navegador anidado
const NestedTabNavigator = () => {
  return (
    <Tab.Navigator
      screenOptions={{
        // Configuración global
        tabBarActiveTintColor: '#007bff',
        tabBarInactiveTintColor: '#6c757d',
        
        // Estilo de la barra de tabs
        tabBarStyle: {
          backgroundColor: '#ffffff',
          borderTopWidth: 1,
          borderTopColor: '#e9ecef',
          paddingBottom: 5,
          paddingTop: 5,
          height: 60,
          elevation: 8,
          shadowColor: '#000',
          shadowOffset: { width: 0, height: -2 },
          shadowOpacity: 0.1,
          shadowRadius: 4,
        },
        
        // Estilo de las etiquetas
        tabBarLabelStyle: {
          fontSize: 12,
          fontWeight: '600',
          marginTop: 2,
        },
        
        // Ocultar header de los tabs (cada stack tiene su propio header)
        headerShown: false,
        
        // Configuración adicional
        tabBarHideOnKeyboard: true,
        tabBarShowLabel: true,
        tabBarAllowFontScaling: false,
      }}
    >
      {/* Tab de inicio con stack anidado */}
      <Tab.Screen
        name="HomeTab"
        component={HomeStackNavigator}
        options={{
          title: 'Inicio',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon 
              name="Home" 
              focused={focused} 
              color={color} 
              size={size}
              badge={3} // Badge para contenido nuevo
            />
          ),
        }}
      />
      
      {/* Tab de búsqueda con stack anidado */}
      <Tab.Screen
        name="SearchTab"
        component={SearchStackNavigator}
        options={{
          title: 'Búsqueda',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon 
              name="Search" 
              focused={focused} 
              color={color} 
              size={size}
              badge={5} // Badge para resultados de búsqueda
            />
          ),
        }}
      />
      
      {/* Tab de perfil con stack anidado */}
      <Tab.Screen
        name="ProfileTab"
        component={ProfileStackNavigator}
        options={{
          title: 'Perfil',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon 
              name="Profile" 
              focused={focused} 
              color={color} 
              size={size}
              badge={2} // Badge para notificaciones
            />
          ),
        }}
      />
      
      {/* Tab de configuración con stack anidado */}
      <Tab.Screen
        name="SettingsTab"
        component={SettingsStackNavigator}
        options={{
          title: 'Config',
          tabBarIcon: ({ focused, color, size }) => (
            <TabIcon 
              name="Settings" 
              focused={focused} 
              color={color} 
              size={size}
            />
          ),
        }}
      />
    </Tab.Navigator>
  );
};

// Componente de icono personalizado con badge
const TabIcon = ({ name, focused, color, size, badge }) => {
  const getIcon = () => {
    switch (name) {
      case 'Home':
        return focused ? '🏠' : '🏠';
      case 'Search':
        return focused ? '🔍' : '🔍';
      case 'Profile':
        return focused ? '👤' : '👤';
      case 'Settings':
        return focused ? '⚙️' : '⚙️';
      default:
        return '📱';
    }
  };

  return (
    <View style={styles.iconContainer}>
      <Text style={[styles.iconText, { color, fontSize: size }]}>
        {getIcon()}
      </Text>
      
      {/* Indicador de tab activo */}
      {focused && (
        <View style={[styles.activeIndicator, { backgroundColor: color }]} />
      )}
      
      {/* Badge del tab */}
      {badge && (
        <View style={styles.badgeContainer}>
          <Text style={styles.badgeText}>{badge}</Text>
        </View>
      )}
    </View>
  );
};

// Estilos para los iconos y badges
const styles = StyleSheet.create({
  iconContainer: {
    alignItems: 'center',
    justifyContent: 'center',
    position: 'relative',
  },
  
  iconText: {
    textAlign: 'center',
  },
  
  activeIndicator: {
    position: 'absolute',
    bottom: -8,
    width: 4,
    height: 4,
    borderRadius: 2,
  },
  
  badgeContainer: {
    position: 'absolute',
    top: -5,
    right: -10,
    backgroundColor: '#dc3545',
    borderRadius: 10,
    minWidth: 20,
    height: 20,
    alignItems: 'center',
    justifyContent: 'center',
    borderWidth: 2,
    borderColor: '#ffffff',
  },
  
  badgeText: {
    color: '#ffffff',
    fontSize: 10,
    fontWeight: 'bold',
  },
});

export default NestedTabNavigator;
```

### **3. Hook Personalizado para Tab Navigator**

```javascript:src/hooks/useTabNavigation.js
import { useNavigation, useRoute } from '@react-navigation/native';
import { useCallback, useRef, useEffect, useState } from 'react';

// Hook personalizado para manejar la navegación de tabs
const useTabNavigation = () => {
  const navigation = useNavigation();
  const route = useRoute();
  
  // Estado para el tab activo actual
  const [activeTab, setActiveTab] = useState('Home');
  
  // Estado para el historial de tabs
  const [tabHistory, setTabHistory] = useState(['Home']);
  
  // Referencia para el estado de cada tab
  const tabStates = useRef({});
  
  // Referencia para el contador de visitas por tab
  const tabVisitCount = useRef({});
  
  // Efecto para detectar cambios de tab
  useEffect(() => {
    const currentTab = getCurrentTabFromRoute(route.name);
    
    if (currentTab && currentTab !== activeTab) {
      // Actualizar tab activo
      setActiveTab(currentTab);
      
      // Agregar a historial
      setTabHistory(prev => [...prev, currentTab]);
      
      // Incrementar contador de visitas
      tabVisitCount.current[currentTab] = (tabVisitCount.current[currentTab] || 0) + 1;
      
      // Limpiar historial si es muy largo
      if (tabHistory.length > 20) {
        setTabHistory(prev => prev.slice(-20));
      }
    }
  }, [route.name, activeTab]);
  
  // Función para obtener el tab actual desde la ruta
  const getCurrentTabFromRoute = useCallback((routeName) => {
    if (routeName.includes('Home')) return 'Home';
    if (routeName.includes('Search')) return 'Search';
    if (routeName.includes('Profile')) return 'Profile';
    if (routeName.includes('Settings')) return 'Settings';
    return null;
  }, []);
  
  // Función para navegar a un tab específico
  const navigateToTab = useCallback((tabName, screenName = null) => {
    const tabRouteMap = {
      Home: 'HomeTab',
      Search: 'SearchTab',
      Profile: 'ProfileTab',
      Settings: 'SettingsTab',
    };
    
    const targetRoute = screenName || tabRouteMap[tabName];
    
    if (targetRoute) {
      navigation.navigate(targetRoute);
      setActiveTab(tabName);
    }
  }, [navigation]);
  
  // Función para navegar al tab anterior
  const navigateToPreviousTab = useCallback(() => {
    if (tabHistory.length > 1) {
      const previousTab = tabHistory[tabHistory.length - 2];
      navigateToTab(previousTab);
    }
  }, [tabHistory, navigateToTab]);
  
  // Función para obtener el estado de un tab
  const getTabState = useCallback((tabName) => {
    return tabStates.current[tabName] || {};
  }, []);
  
  // Función para actualizar el estado de un tab
  const updateTabState = useCallback((tabName, newState) => {
    tabStates.current[tabName] = {
      ...tabStates.current[tabName],
      ...newState,
    };
  }, []);
  
  // Función para resetear el estado de un tab
  const resetTabState = useCallback((tabName) => {
    tabStates.current[tabName] = {};
  }, []);
  
  // Función para obtener estadísticas de uso de tabs
  const getTabStats = useCallback(() => {
    const stats = {
      activeTab,
      tabHistory: [...tabHistory],
      visitCount: { ...tabVisitCount.current },
      totalVisits: Object.values(tabVisitCount.current).reduce((sum, count) => sum + count, 0),
      mostVisitedTab: null,
      leastVisitedTab: null,
    };
    
    // Encontrar tab más y menos visitado
    const visitEntries = Object.entries(tabVisitCount.current);
    if (visitEntries.length > 0) {
      const sortedVisits = visitEntries.sort(([,a], [,b]) => b - a);
      stats.mostVisitedTab = sortedVisits[0][0];
      stats.leastVisitedTab = sortedVisits[sortedVisits.length - 1][0];
    }
    
    return stats;
  }, [activeTab, tabHistory, tabVisitCount]);
  
  // Función para limpiar historial de tabs
  const clearTabHistory = useCallback(() => {
    setTabHistory(['Home']);
  }, []);
  
  // Función para verificar si un tab está activo
  const isTabActive = useCallback((tabName) => {
    return activeTab === tabName;
  }, [activeTab]);
  
  // Función para obtener el tab activo
  const getActiveTab = useCallback(() => {
    return activeTab;
  }, [activeTab]);
  
  // Función para obtener el historial de tabs
  const getTabHistory = useCallback(() => {
    return [...tabHistory];
  }, [tabHistory]);
  
  // Función para navegar a un tab con parámetros
  const navigateToTabWithParams = useCallback((tabName, screenName, params = {}) => {
    const tabRouteMap = {
      Home: 'HomeTab',
      Search: 'SearchTab',
      Profile: 'ProfileTab',
      Settings: 'SettingsTab',
    };
    
    const targetRoute = screenName || tabRouteMap[tabName];
    
    if (targetRoute) {
      navigation.navigate(targetRoute, params);
      setActiveTab(tabName);
    }
  }, [navigation]);
  
  // Función para navegar a un tab y resetear su stack
  const navigateToTabAndReset = useCallback((tabName, screenName = null) => {
    const tabRouteMap = {
      Home: 'HomeTab',
      Search: 'SearchTab',
      Profile: 'ProfileTab',
      Settings: 'SettingsTab',
    };
    
    const targetRoute = screenName || tabRouteMap[tabName];
    
    if (targetRoute) {
      // Resetear el stack del tab y navegar
      navigation.reset({
        index: 0,
        routes: [{ name: targetRoute }],
      });
      
      setActiveTab(tabName);
    }
  }, [navigation]);
  
  return {
    // Estado
    activeTab,
    tabHistory,
    
    // Funciones de navegación
    navigateToTab,
    navigateToPreviousTab,
    navigateToTabWithParams,
    navigateToTabAndReset,
    
    // Funciones de estado
    getTabState,
    updateTabState,
    resetTabState,
    
    // Funciones de información
    getTabStats,
    isTabActive,
    getActiveTab,
    getTabHistory,
    
    // Funciones de utilidad
    clearTabHistory,
    
    // Objeto de navegación nativo
    navigation,
    route,
  };
};

export default useTabNavigation;
```

### **4. Utilidades para Tab Navigator**

```javascript:src/utils/tabNavigatorUtils.js
// Utilidades para Tab Navigator en React Native

// Función para crear configuración de tab personalizado
export const createTabConfig = (config) => {
  const {
    activeColor = '#007bff',
    inactiveColor = '#6c757d',
    backgroundColor = '#ffffff',
    borderColor = '#e9ecef',
    height = 60,
    elevation = 8,
    shadowColor = '#000',
    shadowOffset = { width: 0, height: -2 },
    shadowOpacity = 0.1,
    shadowRadius = 4,
  } = config;
  
  return {
    tabBarActiveTintColor: activeColor,
    tabBarInactiveTintColor: inactiveColor,
    tabBarStyle: {
      backgroundColor,
      borderTopWidth: 1,
      borderTopColor: borderColor,
      paddingBottom: 5,
      paddingTop: 5,
      height,
      elevation,
      shadowColor,
      shadowOffset,
      shadowOpacity,
      shadowRadius,
    },
  };
};

// Función para crear configuración de iconos de tabs
export const createTabIconConfig = (config) => {
  const {
    size = 24,
    activeSize = 28,
    badgeColor = '#dc3545',
    badgeTextColor = '#ffffff',
    showActiveIndicator = true,
    activeIndicatorSize = 4,
    activeIndicatorColor = null,
  } = config;
  
  return {
    tabBarIconStyle: {
      marginBottom: 2,
    },
    tabBarIcon: ({ focused, color, size: iconSize }) => {
      const finalSize = focused ? activeSize : size;
      const indicatorColor = activeIndicatorColor || color;
      
      return {
        size: finalSize,
        color,
        focused,
        badgeColor,
        badgeTextColor,
        showActiveIndicator,
        activeIndicatorSize,
        activeIndicatorColor: indicatorColor,
      };
    },
  };
};

// Función para crear configuración de badges
export const createTabBadgeConfig = (config) => {
  const {
    backgroundColor = '#dc3545',
    textColor = '#ffffff',
    fontSize = 10,
    fontWeight = 'bold',
    minWidth = 20,
    height = 20,
    borderRadius = 10,
    borderWidth = 2,
    borderColor = '#ffffff',
  } = config;
  
  return {
    tabBarBadgeStyle: {
      backgroundColor,
      color: textColor,
      fontSize,
      fontWeight,
      minWidth,
      height,
      borderRadius,
      borderWidth,
      borderColor,
    },
  };
};

// Función para crear configuración de animaciones de tabs
export const createTabAnimationConfig = (config) => {
  const {
    enableAnimations = true,
    animationDuration = 200,
    enablePressFeedback = true,
    pressFeedbackColor = 'rgba(0, 123, 255, 0.1)',
  } = config;
  
  return {
    tabBarStyle: (route) => ({
      // Animación de entrada/salida
      transform: [
        {
          translateY: route.focused ? 0 : 10,
        },
      ],
      opacity: route.focused ? 1 : 0.8,
    }),
    // Configuración de feedback táctil
    tabBarPressColor: enablePressFeedback ? pressFeedbackColor : 'transparent',
    tabBarPressOpacity: enablePressFeedback ? 0.7 : 1,
  };
};

// Función para crear configuración de tabs responsivos
export const createResponsiveTabConfig = (screenDimensions) => {
  const { width, height } = screenDimensions;
  const isLandscape = width > height;
  const isSmallScreen = width < 375;
  
  return {
    tabBarStyle: {
      height: isLandscape ? 50 : (isSmallScreen ? 55 : 60),
      paddingBottom: isLandscape ? 3 : 5,
      paddingTop: isLandscape ? 3 : 5,
    },
    tabBarLabelStyle: {
      fontSize: isSmallScreen ? 10 : 12,
      marginTop: isLandscape ? 1 : 2,
    },
    tabBarIconStyle: {
      marginBottom: isLandscape ? 1 : 2,
    },
  };
};

// Función para validar configuración de tabs
export const validateTabConfig = (tabs) => {
  const errors = [];
  
  if (!tabs || tabs.length === 0) {
    errors.push('Debe haber al menos un tab definido');
  }
  
  tabs?.forEach((tab, index) => {
    if (!tab.name) {
      errors.push(`El tab ${index} debe tener un nombre`);
    }
    
    if (!tab.component) {
      errors.push(`El tab ${tab.name} debe tener un componente`);
    }
    
    if (tab.options && typeof tab.options !== 'object') {
      errors.push(`Las opciones del tab ${tab.name} deben ser un objeto`);
    }
  });
  
  return {
    isValid: errors.length === 0,
    errors,
  };
};

// Función para crear navegador de tabs optimizado
export const createOptimizedTabNavigator = (tabs, options = {}) => {
  const {
    lazy = true,
    unmountOnBlur = false,
    freezeOnBlur = true,
    ...otherOptions
  } = options;
  
  return {
    tabs,
    options: {
      lazy,
      unmountOnBlur,
      freezeOnBlur,
      ...otherOptions,
    },
  };
};

// Función para crear configuración de deep linking para tabs
export const createTabDeepLinkingConfig = (config) => {
  const {
    prefixes = ['myapp://', 'https://myapp.com'],
    config: linkingConfig = {},
  } = config;
  
  return {
    prefixes,
    config: {
      screens: {
        // Configuración de tabs para deep linking
        HomeTab: {
          screens: linkingConfig.homeScreens || {},
        },
        SearchTab: {
          screens: linkingConfig.searchScreens || {},
        },
        ProfileTab: {
          screens: linkingConfig.profileScreens || {},
        },
        SettingsTab: {
          screens: linkingConfig.settingsScreens || {},
        },
      },
      ...linkingConfig,
    },
  };
};
```

---

## 🧪 Casos de Uso

### **Caso 1: Navegación entre Tabs con Estado**
```javascript
// Navegar a un tab específico
const handleTabChange = (tabName) => {
  navigateToTab(tabName);
};

// Navegar a un tab con parámetros
const handleProfileTab = () => {
  navigateToTabWithParams('Profile', 'EditProfile', { userId: 123 });
};
```

### **Caso 2: Gestión de Estado por Tab**
```javascript
// Actualizar estado de un tab
const updateHomeTabState = (newData) => {
  updateTabState('Home', { data: newData });
};

// Obtener estado de un tab
const homeTabState = getTabState('Home');
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear Tabs Personalizados**
Implementa un Tab Navigator con iconos personalizados y badges dinámicos.

### **Ejercicio 2: Navegación Anidada en Tabs**
Crea tabs donde cada uno tenga su propio stack de navegación.

### **Ejercicio 3: Estado Persistente por Tab**
Implementa un sistema que mantenga el estado de cada tab independientemente.

---

## 🚀 Proyecto de la Clase

### **App de Tabs Avanzada**

Crea una aplicación que demuestre:
- **Tabs personalizados**: Iconos y estilos únicos
- **Navegación anidada**: Stacks dentro de cada tab
- **Estado persistente**: Mantener datos por tab
- **Badges dinámicos**: Contadores actualizados en tiempo real

**Requisitos:**
1. Implementar al menos 4 tabs con navegación anidada
2. Crear iconos personalizados para cada tab
3. Implementar badges dinámicos
4. Usar el hook personalizado de tab navigation
5. Mantener estado independiente por tab

**Estructura sugerida:**
```
src/
├── navigation/
│   ├── BasicTabNavigator.js
│   ├── NestedTabNavigator.js
│   └── TabNavigator.js
├── screens/
│   ├── HomeScreen.js
│   ├── SearchScreen.js
│   ├── ProfileScreen.js
│   └── SettingsScreen.js
├── hooks/
│   └── useTabNavigation.js
└── utils/
    └── tabNavigatorUtils.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Navigation Tabs](https://reactnavigation.org/docs/bottom-tab-navigator)
- [Material Top Tabs](https://reactnavigation.org/docs/material-top-tab-navigator)

### **Artículos Recomendados:**
- "Cómo implementar tabs en React Native"
- "Navegación anidada con tabs y stacks"
- "Mejores prácticas para tab navigation"

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Tab Navigator**: Navegación por pestañas en la parte inferior o superior
- **Navegación anidada**: Stacks independientes dentro de cada tab
- **Estado persistente**: Mantener datos por tab
- **Badges y notificaciones**: Indicadores de contenido nuevo

### **Habilidades Desarrolladas:**
- ✅ Crear Tab Navigator básico y avanzado
- ✅ Implementar navegación anidada con tabs y stacks
- ✅ Personalizar iconos, estilos y badges de tabs
- ✅ Crear hooks personalizados para tab navigation
- ✅ Gestionar estado independiente por tab

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Drawer Navigator**, que te permitirá crear menús laterales deslizables para navegación.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Stack Navigator Avanzado](clase_2_stack_navigator.md)
- **➡️ Siguiente Clase**: [Drawer Navigator](clase_4_drawer_navigator.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
