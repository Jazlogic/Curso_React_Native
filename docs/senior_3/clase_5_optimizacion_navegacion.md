# 📱 Clase 5: Optimización de Navegación y Transiciones

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Implementar** lazy loading de pantallas y componentes
2. **Optimizar** transiciones entre pantallas
3. **Configurar** navegación con Suspense y React.lazy
4. **Implementar** pre-carga inteligente de rutas
5. **Crear** sistemas de navegación con máximo rendimiento

---

## 📚 Contenido Teórico

### **1. Lazy Loading de Pantallas**

#### **A. Lazy Loading Básico con React.lazy**
```javascript
import React, { Suspense, lazy } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

// Lazy loading de pantallas
const HomeScreen = lazy(() => import('./screens/HomeScreen'));
const ProfileScreen = lazy(() => import('./screens/ProfileScreen'));
const SettingsScreen = lazy(() => import('./screens/SettingsScreen'));
const ChatScreen = lazy(() => import('./screens/ChatScreen'));

const Stack = createStackNavigator();

const AppNavigator = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ headerShown: false }}
        />
        <Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{ headerShown: false }}
        />
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={{ headerShown: false }}
        />
        <Stack.Screen
          name="Chat"
          component={ChatScreen}
          options={{ headerShown: false }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

// Wrapper con Suspense
const App = () => {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <AppNavigator />
    </Suspense>
  );
};
```

#### **B. Lazy Loading Condicional**
```javascript
const ConditionalLazyScreen = ({ routeName }) => {
  const [ScreenComponent, setScreenComponent] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  useEffect(() => {
    const loadScreen = async () => {
      setIsLoading(true);
      try {
        let Component;
        
        switch (routeName) {
          case 'profile':
            Component = (await import('./screens/ProfileScreen')).default;
            break;
          case 'settings':
            Component = (await import('./screens/SettingsScreen')).default;
            break;
          case 'chat':
            Component = (await import('./screens/ChatScreen')).default;
            break;
          default:
            Component = (await import('./screens/HomeScreen')).default;
        }
        
        setScreenComponent(() => Component);
      } catch (error) {
        console.error('Error loading screen:', error);
      } finally {
        setIsLoading(false);
      }
    };
    
    loadScreen();
  }, [routeName]);
  
  if (isLoading) {
    return <LoadingScreen />;
  }
  
  if (!ScreenComponent) {
    return <ErrorScreen />;
  }
  
  return <ScreenComponent />;
};
```

### **2. Optimización de Transiciones**

#### **A. Transiciones Personalizadas Optimizadas**
```javascript
import { TransitionPresets } from '@react-navigation/stack';

const OptimizedStack = createStackNavigator();

const AppNavigator = () => {
  return (
    <NavigationContainer>
      <OptimizedStack.Navigator
        screenOptions={{
          // Transiciones optimizadas
          ...TransitionPresets.SlideFromRightIOS,
          gestureEnabled: true,
          gestureDirection: 'horizontal',
          
          // Optimizaciones de performance
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
              overlayStyle: {
                opacity: current.progress.interpolate({
                  inputRange: [0, 1],
                  outputRange: [0, 0.5],
                }),
              },
            };
          },
          
          // Configuraciones de performance
          cardStyle: { backgroundColor: 'white' },
          headerStyle: { backgroundColor: 'white' },
          headerTitleStyle: { color: 'black' },
        }}
      >
        <OptimizedStack.Screen name="Home" component={HomeScreen} />
        <OptimizedStack.Screen name="Profile" component={ProfileScreen} />
        <OptimizedStack.Screen name="Settings" component={SettingsScreen} />
      </OptimizedStack.Navigator>
    </NavigationContainer>
  );
};
```

#### **B. Transiciones con Animaciones Personalizadas**
```javascript
const CustomTransition = ({ current, next, layouts }) => {
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
};

const AnimatedStack = createStackNavigator();

const AnimatedNavigator = () => {
  return (
    <NavigationContainer>
      <AnimatedStack.Navigator
        screenOptions={{
          cardStyleInterpolator: CustomTransition,
          gestureEnabled: true,
          gestureDirection: 'horizontal',
          
          // Optimizaciones adicionales
          cardOverlayEnabled: false,
          cardShadowEnabled: false,
        }}
      >
        <AnimatedStack.Screen name="Home" component={HomeScreen} />
        <AnimatedStack.Screen name="Profile" component={ProfileScreen} />
      </AnimatedStack.Navigator>
    </NavigationContainer>
  );
};
```

### **3. Pre-carga Inteligente de Rutas**

#### **A. Pre-carga de Pantallas Comunes**
```javascript
const RoutePreloader = () => {
  useEffect(() => {
    const preloadCommonScreens = async () => {
      try {
        // Pre-cargar pantallas que se usan frecuentemente
        const preloadPromises = [
          import('./screens/ProfileScreen'),
          import('./screens/SettingsScreen'),
          import('./screens/HomeScreen'),
        ];
        
        await Promise.all(preloadPromises);
        console.log('Common screens preloaded successfully');
      } catch (error) {
        console.error('Error preloading screens:', error);
      }
    };
    
    // Pre-cargar después de un delay para no bloquear la carga inicial
    const timer = setTimeout(preloadCommonScreens, 2000);
    
    return () => clearTimeout(timer);
  }, []);
  
  return null; // Componente invisible
};

const AppWithPreloader = () => {
  return (
    <>
      <RoutePreloader />
      <App />
    </>
  );
};
```

#### **B. Pre-carga Condicional Basada en Uso**
```javascript
const SmartRoutePreloader = () => {
  const [preloadedScreens, setPreloadedScreens] = useState(new Set());
  const [userBehavior, setUserBehavior] = useState({});
  
  // Analizar comportamiento del usuario
  useEffect(() => {
    const analyzeUserBehavior = () => {
      // Simular análisis de comportamiento
      const behavior = {
        profileVisits: Math.random() > 0.5,
        settingsVisits: Math.random() > 0.3,
        chatVisits: Math.random() > 0.7,
      };
      
      setUserBehavior(behavior);
    };
    
    analyzeUserBehavior();
  }, []);
  
  // Pre-cargar basado en comportamiento
  useEffect(() => {
    const preloadBasedOnBehavior = async () => {
      const screensToPreload = [];
      
      if (userBehavior.profileVisits) {
        screensToPreload.push(import('./screens/ProfileScreen'));
      }
      
      if (userBehavior.settingsVisits) {
        screensToPreload.push(import('./screens/SettingsScreen'));
      }
      
      if (userBehavior.chatVisits) {
        screensToPreload.push(import('./screens/ChatScreen'));
      }
      
      if (screensToPreload.length > 0) {
        try {
          await Promise.all(screensToPreload);
          setPreloadedScreens(prev => new Set([...prev, ...screensToPreload]));
          console.log('Behavior-based screens preloaded');
        } catch (error) {
          console.error('Error preloading behavior-based screens:', error);
        }
      }
    };
    
    if (Object.keys(userBehavior).length > 0) {
      preloadBasedOnBehavior();
    }
  }, [userBehavior]);
  
  return null;
};
```

### **4. Optimización de Navegación con Tabs**

#### **A. Tabs con Lazy Loading**
```javascript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

const TabNavigator = () => {
  return (
    <Tab.Navigator
      screenOptions={{
        // Optimizaciones para tabs
        lazy: true, // Lazy loading por defecto
        lazyPlaceholder: () => <LoadingScreen />,
        
        // Configuraciones de performance
        tabBarStyle: { backgroundColor: 'white' },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Icon name="home" size={size} color={color} />
          ),
        }}
      />
      
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Icon name="person" size={size} color={color} />
          ),
        }}
      />
      
      <Tab.Screen
        name="Settings"
        component={SettingsScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Icon name="settings" size={size} color={color} />
          ),
        }}
      />
    </Tab.Navigator>
  );
};
```

#### **B. Tabs con Pre-carga Inteligente**
```javascript
const SmartTabNavigator = () => {
  const [activeTab, setActiveTab] = useState('Home');
  const [preloadedTabs, setPreloadedTabs] = useState(new Set(['Home']));
  
  const handleTabPress = useCallback((tabName) => {
    setActiveTab(tabName);
    
    // Pre-cargar tabs adyacentes
    const preloadAdjacentTabs = async () => {
      const tabsToPreload = [];
      
      switch (tabName) {
        case 'Home':
          tabsToPreload.push('Profile');
          break;
        case 'Profile':
          tabsToPreload.push('Home', 'Settings');
          break;
        case 'Settings':
          tabsToPreload.push('Profile');
          break;
      }
      
      for (const tab of tabsToPreload) {
        if (!preloadedTabs.has(tab)) {
          try {
            switch (tab) {
              case 'Profile':
                await import('./screens/ProfileScreen');
                break;
              case 'Settings':
                await import('./screens/SettingsScreen');
                break;
            }
            setPreloadedTabs(prev => new Set([...prev, tab]));
          } catch (error) {
            console.error(`Error preloading ${tab}:`, error);
          }
        }
      }
    };
    
    preloadAdjacentTabs();
  }, [preloadedTabs]);
  
  return (
    <Tab.Navigator
      screenOptions={{
        lazy: true,
        lazyPlaceholder: () => <LoadingScreen />,
      }}
      screenListeners={{
        tabPress: (e) => {
          const tabName = e.target.split('-')[0];
          handleTabPress(tabName);
        },
      }}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
};
```

### **5. Navegación con Suspense y Error Boundaries**

#### **A. Error Boundary para Navegación**
```javascript
class NavigationErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Navigation error:', error, errorInfo);
    
    // Reportar error a servicio de monitoreo
    // reportError(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorTitle}>Algo salió mal</Text>
          <Text style={styles.errorMessage}>
            Ha ocurrido un error en la navegación
          </Text>
          <Button
            title="Reintentar"
            onPress={() => this.setState({ hasError: false, error: null })}
          />
        </View>
      );
    }
    
    return this.props.children;
  }
}

const AppWithErrorBoundary = () => {
  return (
    <NavigationErrorBoundary>
      <Suspense fallback={<LoadingScreen />}>
        <AppNavigator />
      </Suspense>
    </NavigationErrorBoundary>
  );
};
```

#### **B. Suspense con Fallbacks Inteligentes**
```javascript
const SmartSuspense = ({ children, fallback, timeout = 5000 }) => {
  const [isLoading, setIsLoading] = useState(true);
  const [hasTimedOut, setHasTimedOut] = useState(false);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setHasTimedOut(true);
    }, timeout);
    
    return () => clearTimeout(timer);
  }, [timeout]);
  
  useEffect(() => {
    // Simular que la carga se completó
    const timer = setTimeout(() => {
      setIsLoading(false);
    }, 1000);
    
    return () => clearTimeout(timer);
  }, []);
  
  if (isLoading && !hasTimedOut) {
    return fallback;
  }
  
  if (hasTimedOut) {
    return (
      <View style={styles.timeoutContainer}>
        <Text style={styles.timeoutTitle}>Carga lenta</Text>
        <Text style={styles.timeoutMessage}>
          La aplicación está tardando en cargar. Verifica tu conexión.
        </Text>
        <Button
          title="Reintentar"
          onPress={() => {
            setHasTimedOut(false);
            setIsLoading(true);
          }}
        />
      </View>
    );
  }
  
  return children;
};

const AppWithSmartSuspense = () => {
  return (
    <SmartSuspense
      fallback={<LoadingScreen />}
      timeout={3000}
    >
      <AppNavigator />
    </SmartSuspense>
  );
};
```

---

## 💻 Ejercicios Prácticos

### **Ejercicio 1: Navegación con Lazy Loading**
Implementa una navegación que use lazy loading para todas las pantallas.

```javascript
const LazyNavigation = () => {
  // Implementa:
  // - Lazy loading de todas las pantallas
  // - Suspense con fallback
  // - Error boundary
  // - Pre-carga de pantallas comunes
};
```

**Tarea**: Crea 5 pantallas con lazy loading y navegación entre ellas.

### **Ejercicio 2: Transiciones Optimizadas**
Implementa transiciones personalizadas y optimizadas.

```javascript
const OptimizedTransitions = () => {
  // Implementa:
  // - Transiciones personalizadas
  // - Gestos optimizados
  // - Animaciones fluidas
  // - Configuraciones de performance
};
```

**Tarea**: Crea 3 tipos diferentes de transiciones con máximo rendimiento.

### **Ejercicio 3: Pre-carga Inteligente**
Implementa un sistema de pre-carga basado en comportamiento del usuario.

```javascript
const SmartPreloader = () => {
  // Implementa:
  // - Análisis de comportamiento
  // - Pre-carga condicional
  // - Métricas de pre-carga
  // - Fallbacks para errores
};
```

**Tarea**: Crea un sistema que pre-cargue pantallas basándose en patrones de uso.

---

## 🎯 Proyecto Integrador

### **App de Navegación Ultra-Optimizada**

Crea una aplicación que demuestre todas las técnicas de optimización de navegación:

#### **Funcionalidades Requeridas:**
1. **Lazy Loading**: De todas las pantallas y componentes
2. **Transiciones Optimizadas**: Con animaciones fluidas
3. **Pre-carga Inteligente**: Basada en comportamiento del usuario
4. **Error Handling**: Con error boundaries y fallbacks
5. **Performance Monitoring**: Métricas de navegación en tiempo real

#### **Requisitos Técnicos:**
- **Lazy Loading**: Con React.lazy y Suspense
- **Transiciones**: Personalizadas y optimizadas
- **Pre-carga**: Condicional e inteligente
- **Error Handling**: Con error boundaries
- **Métricas**: Monitoreo de performance de navegación

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Navigation](https://reactnavigation.org/)
- [React.lazy](https://react.dev/reference/react/lazy)
- [Suspense](https://react.dev/reference/react/Suspense)

### **Artículos Recomendados:**
- "Navigation Performance in React Native"
- "Lazy Loading Best Practices"
- "Navigation Optimization Strategies"

---

## 🎓 Próximos Pasos

### **Después de Completar Este Módulo:**
- **Módulo 11**: APIs Nativas y Integración
- **Módulo 12**: CI/CD y Deployment
- **Módulo 13**: Monitoreo y Analytics

---

**🎯 Objetivo de la Clase**: Dominar la optimización de navegación en React Native para crear aplicaciones con transiciones fluidas y carga rápida de pantallas.

**💡 Consejo**: La navegación es la columna vertebral de tu app. Optimizarla correctamente mejora dramáticamente la experiencia del usuario.
