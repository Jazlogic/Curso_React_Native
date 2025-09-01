# 🧭 Clase 4: Optimización de Navegación y Carga

## 📋 Objetivos de la Clase
- Implementar lazy loading de pantallas para mejorar el tiempo de carga
- Aprender técnicas de preloading para contenido crítico
- Optimizar transiciones entre pantallas
- Crear navegación fluida y responsiva
- Implementar estrategias de carga inteligente

## ⏱️ Duración
**2 horas**

## 🔗 Navegación
- **Anterior**: [Clase 3: Optimización de Listas y Scroll](clase_3_optimizacion_listas_scroll.md)
- **Siguiente**: [Clase 5: Performance Avanzada y Monitoreo](clase_5_performance_avanzada_monitoreo.md)
- **Módulo**: [README.md](README.md)
- **Inicio**: [🏠](../../README.md)

---

## 🚀 Lazy Loading de Pantallas

### ¿Por qué Lazy Loading de Pantallas?
- **Reduce el bundle inicial** al cargar solo pantallas necesarias
- **Mejora el tiempo de inicio** de la aplicación
- **Optimiza el uso de memoria** del dispositivo

### Implementación con React Navigation
```javascript
// ❌ Sin lazy loading - Todas las pantallas se cargan al inicio
import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';
import SettingsScreen from './screens/SettingsScreen';
import ChatScreen from './screens/ChatScreen';
import GalleryScreen from './screens/GalleryScreen';

const Stack = createStackNavigator();

const BadNavigation = () => (
  <Stack.Navigator>
    <Stack.Screen name="Home" component={HomeScreen} />
    <Stack.Screen name="Profile" component={ProfileScreen} />
    <Stack.Screen name="Settings" component={SettingsScreen} />
    <Stack.Screen name="Chat" component={ChatScreen} />
    <Stack.Screen name="Gallery" component={GalleryScreen} />
  </Stack.Navigator>
);

// ✅ Con lazy loading - Las pantallas se cargan cuando se necesitan
const GoodNavigation = () => {
  const Stack = createStackNavigator();
  
  return (
    <Stack.Navigator>
      <Stack.Screen 
        name="Home" 
        component={HomeScreen} // Pantalla principal siempre cargada
      />
      <Stack.Screen 
        name="Profile" 
        getComponent={() => import('./screens/ProfileScreen').then(m => m.default)}
        options={{
          // Configuración de transición optimizada
          transitionSpec: {
            open: {
              animation: 'timing',
              config: { duration: 300, easing: Easing.out(Easing.poly(4)) }
            },
            close: {
              animation: 'timing',
              config: { duration: 300, easing: Easing.in(Easing.poly(4)) }
            }
          }
        }}
      />
      <Stack.Screen 
        name="Settings" 
        getComponent={() => import('./screens/SettingsScreen').then(m => m.default)}
      />
      <Stack.Screen 
        name="Chat" 
        getComponent={() => import('./screens/ChatScreen').then(m => m.default)}
      />
      <Stack.Screen 
        name="Gallery" 
        getComponent={() => import('./screens/GalleryScreen').then(m => m.default)}
      />
    </Stack.Navigator>
  );
};
```

### Lazy Loading Avanzado con Suspense
```javascript
import React, { Suspense, lazy } from 'react';
import { View, ActivityIndicator, Text } from 'react-native';

// Componente de carga personalizado
const LoadingScreen = ({ message = 'Cargando...' }) => (
  <View style={styles.loadingContainer}>
    <ActivityIndicator size="large" color="#007AFF" />
    <Text style={styles.loadingText}>{message}</Text>
  </View>
);

// Pantallas lazy con Suspense
const LazyHomeScreen = lazy(() => import('./screens/HomeScreen'));
const LazyProfileScreen = lazy(() => import('./screens/ProfileScreen'));
const LazySettingsScreen = lazy(() => import('./screens/SettingsScreen'));
const LazyChatScreen = lazy(() => import('./screens/ChatScreen'));
const LazyGalleryScreen = lazy(() => import('./screens/GalleryScreen'));

// Navegación con Suspense
const SuspenseNavigation = () => {
  const Stack = createStackNavigator();
  
  return (
    <Suspense fallback={<LoadingScreen message="Inicializando app..." />}>
      <Stack.Navigator>
        <Stack.Screen 
          name="Home" 
          component={LazyHomeScreen}
          options={{
            headerShown: false
          }}
        />
        <Stack.Screen 
          name="Profile" 
          component={LazyProfileScreen}
          options={{
            headerTitle: 'Perfil',
            headerBackTitle: 'Atrás'
          }}
        />
        <Stack.Screen 
          name="Settings" 
          component={LazySettingsScreen}
          options={{
            headerTitle: 'Configuración',
            headerBackTitle: 'Atrás'
          }}
        />
        <Stack.Screen 
          name="Chat" 
          component={LazyChatScreen}
          options={{
            headerTitle: 'Chat',
            headerBackTitle: 'Atrás'
          }}
        />
        <Stack.Screen 
          name="Gallery" 
          component={LazyGalleryScreen}
          options={{
            headerTitle: 'Galería',
            headerBackTitle: 'Atrás'
          }}
        />
      </Stack.Navigator>
    </Suspense>
  );
};
```

---

## 🔄 Preloading Inteligente

### ¿Qué es el Preloading?
El preloading es una técnica que carga contenido por adelantado para mejorar la experiencia del usuario.

### Implementación de Preloading
```javascript
class PreloadManager {
  constructor() {
    this.preloadQueue = new Map();
    this.preloadedScreens = new Map();
    this.isPreloading = false;
  }
  
  // Agregar pantalla a la cola de preload
  addToPreloadQueue(screenName, importFn, priority = 'normal') {
    this.preloadQueue.set(screenName, {
      importFn,
      priority,
      timestamp: Date.now()
    });
    
    console.log(`📋 Agregada a cola de preload: ${screenName}`);
  }
  
  // Iniciar preloading
  async startPreloading() {
    if (this.isPreloading) return;
    
    this.isPreloading = true;
    console.log('🚀 Iniciando preloading...');
    
    try {
      // Ordenar por prioridad
      const sortedQueue = Array.from(this.preloadQueue.entries())
        .sort(([, a], [, b]) => {
          const priorityOrder = { high: 3, normal: 2, low: 1 };
          return priorityOrder[b.priority] - priorityOrder[a.priority];
        });
      
      // Preload en paralelo con límite de concurrencia
      const concurrencyLimit = 3;
      const chunks = this.chunkArray(sortedQueue, concurrencyLimit);
      
      for (const chunk of chunks) {
        await Promise.all(
          chunk.map(([screenName, { importFn }]) => 
            this.preloadScreen(screenName, importFn)
          )
        );
      }
      
      console.log('✅ Preloading completado');
    } catch (error) {
      console.error('❌ Error en preloading:', error);
    } finally {
      this.isPreloading = false;
    }
  }
  
  // Preload de una pantalla específica
  async preloadScreen(screenName, importFn) {
    try {
      console.log(`🔄 Preload: ${screenName}`);
      
      const startTime = performance.now();
      const module = await importFn();
      const loadTime = performance.now() - startTime;
      
      this.preloadedScreens.set(screenName, {
        component: module.default || module,
        loadTime,
        timestamp: Date.now()
      });
      
      console.log(`✅ Preload completado: ${screenName} (${loadTime.toFixed(2)}ms)`);
    } catch (error) {
      console.error(`❌ Error preload: ${screenName}`, error);
    }
  }
  
  // Obtener pantalla preloaded
  getPreloadedScreen(screenName) {
    return this.preloadedScreens.get(screenName);
  }
  
  // Limpiar pantallas preloaded antiguas
  cleanupOldPreloadedScreens(maxAge = 5 * 60 * 1000) { // 5 minutos
    const now = Date.now();
    
    for (const [screenName, { timestamp }] of this.preloadedScreens.entries()) {
      if (now - timestamp > maxAge) {
        this.preloadedScreens.delete(screenName);
        console.log(`🧹 Limpiado preload antiguo: ${screenName}`);
      }
    }
  }
  
  // Dividir array en chunks
  chunkArray(array, size) {
    const chunks = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }
  
  // Obtener estadísticas de preload
  getStats() {
    return {
      queueSize: this.preloadQueue.size,
      preloadedCount: this.preloadedScreens.size,
      isPreloading: this.isPreloading,
      preloadedScreens: Array.from(this.preloadedScreens.keys())
    };
  }
}

// Hook para usar preloading
const usePreloading = () => {
  const preloadManager = useMemo(() => new PreloadManager(), []);
  
  const preloadScreen = useCallback((screenName, importFn, priority = 'normal') => {
    preloadManager.addToPreloadQueue(screenName, importFn, priority);
  }, [preloadManager]);
  
  const startPreloading = useCallback(() => {
    preloadManager.startPreloading();
  }, [preloadManager]);
  
  const getPreloadedScreen = useCallback((screenName) => {
    return preloadManager.getPreloadedScreen(screenName);
  }, [preloadManager]);
  
  const getStats = useCallback(() => {
    return preloadManager.getStats();
  }, [preloadManager]);
  
  // Limpiar preloads antiguos
  useEffect(() => {
    const cleanupInterval = setInterval(() => {
      preloadManager.cleanupOldPreloadedScreens();
    }, 60000); // Cada minuto
    
    return () => clearInterval(cleanupInterval);
  }, [preloadManager]);
  
  return {
    preloadScreen,
    startPreloading,
    getPreloadedScreen,
    getStats
  };
};

// Uso en navegación
const PreloadNavigation = () => {
  const { preloadScreen, startPreloading, getPreloadedScreen } = usePreloading();
  const Stack = createStackNavigator();
  
  // Preload de pantallas críticas
  useEffect(() => {
    // Preload inmediato de pantallas de alta prioridad
    preloadScreen('Profile', () => import('./screens/ProfileScreen'), 'high');
    preloadScreen('Settings', () => import('./screens/SettingsScreen'), 'high');
    
    // Iniciar preloading después de un delay
    const preloadTimer = setTimeout(() => {
      startPreloading();
    }, 2000);
    
    return () => clearTimeout(preloadTimer);
  }, [preloadScreen, startPreloading]);
  
  // Componente que usa preloading
  const LazyScreenWithPreload = ({ screenName, importFn, fallback }) => {
    const [Component, setComponent] = useState(null);
    
    useEffect(() => {
      // Intentar obtener pantalla preloaded primero
      const preloaded = getPreloadedScreen(screenName);
      
      if (preloaded) {
        setComponent(() => preloaded.component);
        console.log(`🚀 Usando pantalla preloaded: ${screenName}`);
      } else {
        // Cargar normalmente si no está preloaded
        importFn().then(module => {
          setComponent(() => module.default || module);
        });
      }
    }, [screenName, importFn, getPreloadedScreen]);
    
    if (!Component) {
      return fallback || <LoadingScreen />;
    }
    
    return <Component />;
  };
  
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={LazyHomeScreen} />
      <Stack.Screen 
        name="Profile" 
        component={() => (
          <LazyScreenWithPreload
            screenName="Profile"
            importFn={() => import('./screens/ProfileScreen')}
            fallback={<LoadingScreen message="Cargando perfil..." />}
          />
        )}
      />
      <Stack.Screen 
        name="Settings" 
        component={() => (
          <LazyScreenWithPreload
            screenName="Settings"
            importFn={() => import('./screens/SettingsScreen')}
            fallback={<LoadingScreen message="Cargando configuración..." />}
          />
        )}
      />
    </Stack.Navigator>
  );
};
```

---

## ⚡ Optimización de Transiciones

### Transiciones Personalizadas
```javascript
// Configuración de transiciones optimizadas
const optimizedTransitions = {
  // Transición rápida para navegación simple
  fast: {
    transitionSpec: {
      open: {
        animation: 'timing',
        config: { 
          duration: 200, 
          easing: Easing.out(Easing.quad) 
        }
      },
      close: {
        animation: 'timing',
        config: { 
          duration: 200, 
          easing: Easing.in(Easing.quad) 
        }
      }
    },
    cardStyleInterpolator: CardStyleInterpolators.forHorizontalIOS
  },
  
  // Transición suave para pantallas importantes
  smooth: {
    transitionSpec: {
      open: {
        animation: 'timing',
        config: { 
          duration: 350, 
          easing: Easing.out(Easing.back(1.2)) 
        }
      },
      close: {
        animation: 'timing',
        config: { 
          duration: 300, 
          easing: Easing.in(Easing.back(1.2)) 
        }
      }
    },
    cardStyleInterpolator: CardStyleInterpolators.forHorizontalIOS
  },
  
  // Transición modal para pantallas de configuración
  modal: {
    transitionSpec: {
      open: {
        animation: 'timing',
        config: { 
          duration: 400, 
          easing: Easing.out(Easing.back(1.5)) 
        }
      },
      close: {
        animation: 'timing',
        config: { 
          duration: 350, 
          easing: Easing.in(Easing.back(1.5)) 
        }
      }
    },
    cardStyleInterpolator: CardStyleInterpolators.forModalPresentationIOS
  }
};

// Navegación con transiciones optimizadas
const OptimizedNavigation = () => {
  const Stack = createStackNavigator();
  
  return (
    <Stack.Navigator
      screenOptions={{
        // Transición por defecto
        ...optimizedTransitions.fast,
        headerShown: false
      }}
    >
      <Stack.Screen 
        name="Home" 
        component={LazyHomeScreen}
        options={{
          // Sin transición para la pantalla inicial
          animationEnabled: false
        }}
      />
      
      <Stack.Screen 
        name="Profile" 
        component={LazyProfileScreen}
        options={{
          ...optimizedTransitions.smooth,
          headerShown: true,
          headerTitle: 'Perfil',
          headerBackTitle: 'Atrás'
        }}
      />
      
      <Stack.Screen 
        name="Settings" 
        component={LazySettingsScreen}
        options={{
          ...optimizedTransitions.modal,
          headerShown: true,
          headerTitle: 'Configuración',
          presentation: 'modal'
        }}
      />
      
      <Stack.Screen 
        name="Chat" 
        component={LazyChatScreen}
        options={{
          ...optimizedTransitions.fast,
          headerShown: true,
          headerTitle: 'Chat',
          headerBackTitle: 'Atrás'
        }}
      />
    </Stack.Navigator>
  );
};
```

### Transiciones Personalizadas Avanzadas
```javascript
// Transición personalizada con interpolación
const customTransition = {
  transitionSpec: {
    open: {
      animation: 'timing',
      config: { 
        duration: 500, 
        easing: Easing.bezier(0.25, 0.46, 0.45, 0.94) 
      }
    },
    close: {
      animation: 'timing',
      config: { 
        duration: 400, 
        easing: Easing.bezier(0.55, 0.055, 0.675, 0.19) 
      }
    }
  },
  
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
  }
};

// Transición de fade para pantallas de carga
const fadeTransition = {
  transitionSpec: {
    open: {
      animation: 'timing',
      config: { duration: 300 }
    },
    close: {
      animation: 'timing',
      config: { duration: 300 }
    }
  },
  
  cardStyleInterpolator: ({ current }) => ({
    cardStyle: {
      opacity: current.progress
    }
  })
};

// Navegación con transiciones personalizadas
const CustomTransitionNavigation = () => {
  const Stack = createStackNavigator();
  
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false
      }}
    >
      <Stack.Screen 
        name="Home" 
        component={LazyHomeScreen}
        options={{
          animationEnabled: false
        }}
      />
      
      <Stack.Screen 
        name="Profile" 
        component={LazyProfileScreen}
        options={{
          ...customTransition,
          headerShown: true,
          headerTitle: 'Perfil'
        }}
      />
      
      <Stack.Screen 
        name="Loading" 
        component={LoadingScreen}
        options={{
          ...fadeTransition,
          headerShown: false
        }}
      />
    </Stack.Navigator>
  );
};
```

---

## 📱 Ejercicios Prácticos

### Ejercicio 1: Sistema de Preloading Inteligente
Crea un sistema que preload pantallas basándose en el comportamiento del usuario.

```javascript
// Tu código aquí
const SmartPreloader = ({ userBehavior, navigationHistory }) => {
  // Implementa:
  // 1. Análisis de comportamiento del usuario
  // 2. Predicción de pantallas a preload
  // 3. Priorización inteligente
  // 4. Gestión de memoria
};
```

### Ejercicio 2: Transiciones Adaptativas
Implementa transiciones que se adapten al contexto de la navegación.

```javascript
// Tu código aquí
const AdaptiveTransitions = ({ navigationType, screenContext }) => {
  // Implementa:
  // 1. Transiciones basadas en contexto
  // 2. Animaciones personalizadas
  // 3. Optimización de rendimiento
  // 4. Gestión de estados de transición
};
```

### Ejercicio 3: Navegación con Cache
Crea un sistema de navegación que mantenga el estado de las pantallas.

```javascript
// Tu código aquí
const CachedNavigation = ({ screens, cacheStrategy }) => {
  // Implementa:
  // 1. Cache de estado de pantallas
  // 2. Restauración de estado
  // 3. Gestión de memoria
  // 4. Sincronización de datos
};
```

---

## 🔍 Resumen de la Clase

### ✅ Lo que Aprendiste
- **Lazy loading de pantallas** para reducir el bundle inicial
- **Preloading inteligente** para mejorar la experiencia del usuario
- **Transiciones optimizadas** para navegación fluida
- **Técnicas avanzadas** de carga y navegación

### 🎯 Próximos Pasos
En la siguiente clase aprenderás:
- **Bundle splitting** y code splitting
- **Monitoreo en producción** y métricas
- **Optimización avanzada** de performance

### 💡 Consejos Clave
1. **Usa lazy loading** para pantallas no críticas
2. **Implementa preloading** para pantallas frecuentemente accedidas
3. **Optimiza transiciones** para una experiencia fluida
4. **Mide el impacto** del preloading en la experiencia del usuario
5. **Balancea preloading** con uso de memoria

---

## 📚 Recursos Adicionales
- [React Navigation Performance](https://reactnavigation.org/docs/performance)
- [Lazy Loading in React](https://reactjs.org/docs/code-splitting.html)
- [Navigation Transitions](https://reactnavigation.org/docs/stack-navigator#transitions)
- [Preloading Strategies](https://web.dev/preload-critical-assets/)

---

**¿Tienes alguna pregunta sobre la optimización de navegación y carga? ¿Te gustaría que profundice en algún aspecto específico antes de continuar con la siguiente clase?**
