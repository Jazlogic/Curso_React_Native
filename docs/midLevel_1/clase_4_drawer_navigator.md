# 📚 Clase 4: Drawer Navigator

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 3: Tab Navigator](clase_3_tab_navigator.md)
- **➡️ Siguiente**: [Clase 5: Navegación Personalizada](clase_5_navegacion_personalizada.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el funcionamiento del Drawer Navigator en React Native
- Aprender a crear menús laterales deslizables
- Dominar la personalización del drawer y sus opciones
- Implementar navegación anidada con drawer y otros navegadores
- Crear experiencias de usuario intuitivas con menús laterales

---

## 📚 Contenido Teórico

### **¿Qué es el Drawer Navigator?**

El Drawer Navigator es un componente de React Navigation que permite crear menús laterales deslizables desde el borde izquierdo o derecho de la pantalla. Es ideal para aplicaciones que tienen muchas secciones y necesitan un acceso centralizado a todas las funcionalidades.

#### **Características principales:**
- **Menú lateral**: Acceso a todas las secciones principales
- **Gestos táctiles**: Deslizamiento para abrir/cerrar
- **Personalización completa**: Estilos, iconos y contenido
- **Navegación anidada**: Puede contener otros navegadores
- **Accesibilidad**: Fácil acceso a todas las funcionalidades

### **Tipos de Drawer:**

#### **1. Left Drawer (Izquierdo):**
- Se abre desde el borde izquierdo
- Estilo estándar en la mayoría de apps
- Fácil acceso con el pulgar derecho
- Ideal para navegación principal

#### **2. Right Drawer (Derecho):**
- Se abre desde el borde derecho
- Menos común pero útil para acciones secundarias
- Acceso rápido a herramientas o configuraciones
- Perfecto para menús contextuales

#### **3. Custom Drawer:**
- Posicionamiento libre
- Animaciones personalizadas
- Contenido completamente personalizable
- Máxima flexibilidad de diseño

### **Ventajas del Drawer Navigator:**

✅ **Acceso centralizado**: Todas las secciones en un lugar
✅ **Escalabilidad**: Fácil agregar nuevas opciones
✅ **UX familiar**: Patrón estándar en apps móviles
✅ **Organización clara**: Jerarquía visual de la navegación
✅ **Flexibilidad**: Puede contener cualquier tipo de contenido

---

## 💻 Implementación Práctica

### **1. Drawer Navigator Básico**

```javascript:src/navigation/BasicDrawerNavigator.js
import React from 'react';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { Text, View, StyleSheet, Image } from 'react-native';

// Importamos las pantallas
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import HelpScreen from '../screens/HelpScreen';
import AboutScreen from '../screens/AboutScreen';

// Creamos el navegador de drawer
const Drawer = createDrawerNavigator();

// Componente personalizado para el header del drawer
const CustomDrawerHeader = () => {
  return (
    <View style={styles.drawerHeader}>
      {/* Avatar del usuario */}
      <View style={styles.avatarContainer}>
        <Image
          source={{ uri: 'https://via.placeholder.com/80' }}
          style={styles.avatar}
        />
        <View style={styles.onlineIndicator} />
      </View>
      
      {/* Información del usuario */}
      <View style={styles.userInfo}>
        <Text style={styles.userName}>Juan Pérez</Text>
        <Text style={styles.userEmail}>juan@example.com</Text>
        <Text style={styles.userStatus}>En línea</Text>
      </View>
      
      {/* Botón de editar perfil */}
      <View style={styles.editButton}>
        <Text style={styles.editButtonText}>✏️</Text>
      </View>
    </View>
  );
};

// Componente personalizado para los items del drawer
const CustomDrawerItem = ({ label, icon, focused, onPress }) => {
  return (
    <View style={[styles.drawerItem, focused && styles.drawerItemFocused]}>
      <Text style={[styles.drawerItemIcon, { color: focused ? '#007bff' : '#666' }]}>
        {icon}
      </Text>
      <Text style={[styles.drawerItemLabel, { color: focused ? '#007bff' : '#333' }]}>
        {label}
      </Text>
      {focused && <View style={styles.drawerItemIndicator} />}
    </View>
  );
};

// Componente principal del navegador de drawer
const BasicDrawerNavigator = () => {
  return (
    <Drawer.Navigator
      // Configuración global del drawer
      screenOptions={{
        // Estilo del header del drawer
        headerShown: true, // Mostrar header en las pantallas
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
        
        // Estilo del drawer
        drawerStyle: {
          backgroundColor: '#ffffff',
          width: 280, // Ancho del drawer
          elevation: 16,
          shadowColor: '#000',
          shadowOffset: { width: 4, height: 0 },
          shadowOpacity: 0.3,
          shadowRadius: 8,
        },
        
        // Estilo del overlay del drawer
        overlayColor: 'rgba(0, 0, 0, 0.5)',
        
        // Configuración de gestos
        gestureEnabled: true, // Habilitar gestos
        gestureHandlerProps: {
          velocityThreshold: 0.3, // Velocidad mínima para activar
          directionalOffsetThreshold: 80, // Distancia mínima para activar
        },
        
        // Configuración de animaciones
        drawerType: 'front', // Tipo de drawer (front, back, slide)
        drawerPosition: 'left', // Posición del drawer (left, right)
        
        // Configuración del header del drawer
        drawerHeader: () => <CustomDrawerHeader />,
        
        // Configuración de los items del drawer
        drawerActiveTintColor: '#007bff', // Color del item activo
        drawerInactiveTintColor: '#666', // Color del item inactivo
        drawerActiveBackgroundColor: 'rgba(0, 123, 255, 0.1)', // Fondo del item activo
        drawerInactiveBackgroundColor: 'transparent', // Fondo del item inactivo
        
        // Estilo de las etiquetas
        drawerLabelStyle: {
          fontSize: 16,
          fontWeight: '500',
          marginLeft: 8,
        },
        
        // Estilo de los iconos
        drawerIconStyle: {
          width: 24,
          height: 24,
        },
      }}
    >
      {/* Pantalla de inicio */}
      <Drawer.Screen
        name="Home"
        component={HomeScreen}
        options={{
          title: 'Inicio',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              🏠
            </Text>
          ),
          // Configuración específica del drawer
          drawerLabel: 'Inicio',
          drawerItemStyle: {
            marginVertical: 2,
          },
        }}
      />
      
      {/* Pantalla de perfil */}
      <Drawer.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          title: 'Mi Perfil',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              👤
            </Text>
          ),
          drawerLabel: 'Mi Perfil',
          // Badge para notificaciones del perfil
          drawerBadge: 3,
          drawerBadgeStyle: {
            backgroundColor: '#dc3545',
            color: '#ffffff',
            fontSize: 12,
            fontWeight: 'bold',
          },
        }}
      />
      
      {/* Pantalla de configuración */}
      <Drawer.Screen
        name="Settings"
        component={SettingsScreen}
        options={{
          title: 'Configuración',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              ⚙️
            </Text>
          ),
          drawerLabel: 'Configuración',
        }}
      />
      
      {/* Separador en el drawer */}
      <Drawer.Screen
        name="Separator"
        component={SeparatorScreen}
        options={{
          drawerLabel: () => null,
          drawerItemStyle: {
            height: 1,
            backgroundColor: '#e9ecef',
            marginVertical: 10,
          },
          drawerIcon: () => null,
        }}
      />
      
      {/* Pantalla de ayuda */}
      <Drawer.Screen
        name="Help"
        component={HelpScreen}
        options={{
          title: 'Ayuda',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              ❓
            </Text>
          ),
          drawerLabel: 'Ayuda',
        }}
      />
      
      {/* Pantalla de acerca de */}
      <Drawer.Screen
        name="About"
        component={AboutScreen}
        options={{
          title: 'Acerca de',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              ℹ️
            </Text>
          ),
          drawerLabel: 'Acerca de',
        }}
      />
    </Drawer.Navigator>
  );
};

// Pantalla separadora (no se renderiza)
const SeparatorScreen = () => null;

// Estilos para el drawer
const styles = StyleSheet.create({
  // Header del drawer
  drawerHeader: {
    backgroundColor: '#f8f9fa',
    padding: 20,
    borderBottomWidth: 1,
    borderBottomColor: '#e9ecef',
    flexDirection: 'row',
    alignItems: 'center',
  },
  
  // Contenedor del avatar
  avatarContainer: {
    position: 'relative',
    marginRight: 15,
  },
  
  // Avatar del usuario
  avatar: {
    width: 60,
    height: 60,
    borderRadius: 30,
    borderWidth: 3,
    borderColor: '#007bff',
  },
  
  // Indicador de estado en línea
  onlineIndicator: {
    position: 'absolute',
    bottom: 2,
    right: 2,
    width: 16,
    height: 16,
    borderRadius: 8,
    backgroundColor: '#28a745',
    borderWidth: 2,
    borderColor: '#ffffff',
  },
  
  // Información del usuario
  userInfo: {
    flex: 1,
  },
  
  // Nombre del usuario
  userName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 4,
  },
  
  // Email del usuario
  userEmail: {
    fontSize: 14,
    color: '#666',
    marginBottom: 2,
  },
  
  // Estado del usuario
  userStatus: {
    fontSize: 12,
    color: '#28a745',
    fontWeight: '500',
  },
  
  // Botón de editar
  editButton: {
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: '#e9ecef',
    justifyContent: 'center',
    alignItems: 'center',
  },
  
  // Texto del botón de editar
  editButtonText: {
    fontSize: 16,
  },
  
  // Item del drawer
  drawerItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 12,
    paddingHorizontal: 16,
    marginHorizontal: 8,
    borderRadius: 8,
    position: 'relative',
  },
  
  // Item del drawer cuando está enfocado
  drawerItemFocused: {
    backgroundColor: 'rgba(0, 123, 255, 0.1)',
  },
  
  // Icono del item del drawer
  drawerItemIcon: {
    fontSize: 20,
    marginRight: 12,
    width: 24,
    textAlign: 'center',
  },
  
  // Etiqueta del item del drawer
  drawerItemLabel: {
    fontSize: 16,
    fontWeight: '500',
    flex: 1,
  },
  
  // Indicador del item activo
  drawerItemIndicator: {
    position: 'absolute',
    left: 0,
    top: 0,
    bottom: 0,
    width: 4,
    backgroundColor: '#007bff',
    borderTopRightRadius: 2,
    borderBottomRightRadius: 2,
  },
  
  // Icono del drawer
  drawerIcon: {
    textAlign: 'center',
  },
});

export default BasicDrawerNavigator;
```

### **2. Drawer Navigator con Navegación Anidada**

```javascript:src/navigation/NestedDrawerNavigator.js
import React from 'react';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Text, View, StyleSheet } from 'react-native';

// Importamos las pantallas
import HomeScreen from '../screens/HomeScreen';
import HomeDetailScreen from '../screens/HomeDetailScreen';
import ProfileScreen from '../screens/ProfileScreen';
import EditProfileScreen from '../screens/EditProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import PrivacyScreen from '../screens/PrivacyScreen';
import SecurityScreen from '../screens/SecurityScreen';
import NotificationsScreen from '../screens/NotificationsScreen';
import MessagesScreen from '../screens/MessagesScreen';

// Creamos los navegadores
const Drawer = createDrawerNavigator();
const HomeStack = createStackNavigator();
const ProfileStack = createStackNavigator();
const SettingsStack = createStackNavigator();
const NotificationsStack = createStackNavigator();
const MessagesStack = createStackNavigator();

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

// Stack para la pantalla de perfil
const ProfileStackNavigator = () => {
  return (
    <ProfileStack.Navigator
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

// Stack para notificaciones
const NotificationsStackNavigator = () => {
  return (
    <NotificationsStack.Navigator
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
      <NotificationsStack.Screen
        name="NotificationsMain"
        component={NotificationsScreen}
        options={{
          title: 'Notificaciones',
        }}
      />
    </NotificationsStack.Navigator>
  );
};

// Stack para mensajes
const MessagesStackNavigator = () => {
  return (
    <MessagesStack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#17a2b8',
        },
        headerTintColor: '#fff',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
      }}
    >
      <MessagesStack.Screen
        name="MessagesMain"
        component={MessagesScreen}
        options={{
          title: 'Mensajes',
        }}
      />
    </MessagesStack.Navigator>
  );
};

// Componente principal del navegador anidado
const NestedDrawerNavigator = () => {
  return (
    <Drawer.Navigator
      screenOptions={{
        // Configuración global
        headerShown: false, // Ocultar header (cada stack tiene su propio header)
        
        // Estilo del drawer
        drawerStyle: {
          backgroundColor: '#ffffff',
          width: 300,
          elevation: 16,
          shadowColor: '#000',
          shadowOffset: { width: 4, height: 0 },
          shadowOpacity: 0.3,
          shadowRadius: 8,
        },
        
        // Configuración de gestos
        gestureEnabled: true,
        gestureHandlerProps: {
          velocityThreshold: 0.3,
          directionalOffsetThreshold: 80,
        },
        
        // Configuración de animaciones
        drawerType: 'front',
        drawerPosition: 'left',
        
        // Configuración del header del drawer
        drawerHeader: () => <CustomDrawerHeader />,
        
        // Configuración de los items
        drawerActiveTintColor: '#007bff',
        drawerInactiveTintColor: '#666',
        drawerActiveBackgroundColor: 'rgba(0, 123, 255, 0.1)',
        drawerInactiveBackgroundColor: 'transparent',
        
        // Estilo de las etiquetas
        drawerLabelStyle: {
          fontSize: 16,
          fontWeight: '500',
          marginLeft: 8,
        },
      }}
    >
      {/* Pantalla de inicio con stack anidado */}
      <Drawer.Screen
        name="HomeDrawer"
        component={HomeStackNavigator}
        options={{
          title: 'Inicio',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              🏠
            </Text>
          ),
          drawerLabel: 'Inicio',
        }}
      />
      
      {/* Pantalla de perfil con stack anidado */}
      <Drawer.Screen
        name="ProfileDrawer"
        component={ProfileStackNavigator}
        options={{
          title: 'Mi Perfil',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              👤
            </Text>
          ),
          drawerLabel: 'Mi Perfil',
          drawerBadge: 2, // Notificaciones del perfil
        }}
      />
      
      {/* Pantalla de configuración con stack anidado */}
      <Drawer.Screen
        name="SettingsDrawer"
        component={SettingsStackNavigator}
        options={{
          title: 'Configuración',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              ⚙️
            </Text>
          ),
          drawerLabel: 'Configuración',
        }}
      />
      
      {/* Separador */}
      <Drawer.Screen
        name="Separator1"
        component={SeparatorScreen}
        options={{
          drawerLabel: () => null,
          drawerItemStyle: {
            height: 1,
            backgroundColor: '#e9ecef',
            marginVertical: 10,
          },
          drawerIcon: () => null,
        }}
      />
      
      {/* Pantalla de notificaciones */}
      <Drawer.Screen
        name="NotificationsDrawer"
        component={NotificationsStackNavigator}
        options={{
          title: 'Notificaciones',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              🔔
            </Text>
          ),
          drawerLabel: 'Notificaciones',
          drawerBadge: 5, // Notificaciones no leídas
        }}
      />
      
      {/* Pantalla de mensajes */}
      <Drawer.Screen
        name="MessagesDrawer"
        component={MessagesStackNavigator}
        options={{
          title: 'Mensajes',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              💬
            </Text>
          ),
          drawerLabel: 'Mensajes',
          drawerBadge: 3, // Mensajes no leídos
        }}
      />
      
      {/* Separador */}
      <Drawer.Screen
        name="Separator2"
        component={SeparatorScreen}
        options={{
          drawerLabel: () => null,
          drawerItemStyle: {
            height: 1,
            backgroundColor: '#e9ecef',
            marginVertical: 10,
          },
          drawerIcon: () => null,
        }}
      />
      
      {/* Pantallas adicionales */}
      <Drawer.Screen
        name="Help"
        component={HelpScreen}
        options={{
          title: 'Ayuda',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              ❓
            </Text>
          ),
          drawerLabel: 'Ayuda',
        }}
      />
      
      <Drawer.Screen
        name="About"
        component={AboutScreen}
        options={{
          title: 'Acerca de',
          drawerIcon: ({ focused, color, size }) => (
            <Text style={[styles.drawerIcon, { color, fontSize: size }]}>
              ℹ️
            </Text>
          ),
          drawerLabel: 'Acerca de',
        }}
      />
    </Drawer.Navigator>
  );
};

// Componente del header del drawer
const CustomDrawerHeader = () => {
  return (
    <View style={styles.drawerHeader}>
      <View style={styles.avatarContainer}>
        <Text style={styles.avatarPlaceholder}>👤</Text>
        <View style={styles.onlineIndicator} />
      </View>
      
      <View style={styles.userInfo}>
        <Text style={styles.userName}>Usuario Demo</Text>
        <Text style={styles.userEmail}>usuario@demo.com</Text>
        <Text style={styles.userStatus}>En línea</Text>
      </View>
    </View>
  );
};

// Pantalla separadora
const SeparatorScreen = () => null;

// Pantallas adicionales
const HelpScreen = () => (
  <View style={styles.screen}>
    <Text style={styles.screenTitle}>Ayuda</Text>
  </View>
);

const AboutScreen = () => (
  <View style={styles.screen}>
    <Text style={styles.screenTitle}>Acerca de</Text>
  </View>
);

// Estilos
const styles = StyleSheet.create({
  drawerHeader: {
    backgroundColor: '#f8f9fa',
    padding: 20,
    borderBottomWidth: 1,
    borderBottomColor: '#e9ecef',
    flexDirection: 'row',
    alignItems: 'center',
  },
  
  avatarContainer: {
    position: 'relative',
    marginRight: 15,
  },
  
  avatarPlaceholder: {
    fontSize: 40,
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: '#e9ecef',
    textAlign: 'center',
    lineHeight: 60,
  },
  
  onlineIndicator: {
    position: 'absolute',
    bottom: 2,
    right: 2,
    width: 16,
    height: 16,
    borderRadius: 8,
    backgroundColor: '#28a745',
    borderWidth: 2,
    borderColor: '#ffffff',
  },
  
  userInfo: {
    flex: 1,
  },
  
  userName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 4,
  },
  
  userEmail: {
    fontSize: 14,
    color: '#666',
    marginBottom: 2,
  },
  
  userStatus: {
    fontSize: 12,
    color: '#28a745',
    fontWeight: '500',
  },
  
  drawerIcon: {
    textAlign: 'center',
  },
  
  screen: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f8f9fa',
  },
  
  screenTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
});

export default NestedDrawerNavigator;
```

### **3. Hook Personalizado para Drawer Navigator**

```javascript:src/hooks/useDrawerNavigation.js
import { useNavigation, useRoute } from '@react-navigation/native';
import { useCallback, useRef, useEffect, useState } from 'react';

// Hook personalizado para manejar la navegación del drawer
const useDrawerNavigation = () => {
  const navigation = useNavigation();
  const route = useRoute();
  
  // Estado para el drawer
  const [isDrawerOpen, setIsDrawerOpen] = useState(false);
  const [drawerHistory, setDrawerHistory] = useState([]);
  
  // Referencia para el estado del drawer
  const drawerState = useRef({
    isOpen: false,
    lastOpened: null,
    openCount: 0,
  });
  
  // Referencia para el historial de navegación del drawer
  const drawerNavigationHistory = useRef([]);
  
  // Efecto para detectar cambios en el drawer
  useEffect(() => {
    // Escuchar eventos de navegación para detectar cambios en el drawer
    const unsubscribe = navigation.addListener('state', (e) => {
      const currentRoute = e.data.state.routes[e.data.state.index];
      const currentDrawerRoute = currentRoute.name;
      
      // Detectar si el drawer está abierto
      const isOpen = currentDrawerRoute.includes('Drawer') || 
                     currentDrawerRoute === 'Home' ||
                     currentDrawerRoute === 'Profile' ||
                     currentDrawerRoute === 'Settings';
      
      if (isOpen !== isDrawerOpen) {
        setIsDrawerOpen(isOpen);
        drawerState.current.isOpen = isOpen;
        
        if (isOpen) {
          // Drawer se abrió
          drawerState.current.openCount += 1;
          drawerState.current.lastOpened = Date.now();
          
          // Agregar a historial
          setDrawerHistory(prev => [...prev, {
            route: currentDrawerRoute,
            timestamp: Date.now(),
            action: 'opened'
          }]);
        }
      }
    });
    
    return unsubscribe;
  }, [navigation, isDrawerOpen]);
  
  // Función para abrir el drawer
  const openDrawer = useCallback(() => {
    navigation.openDrawer();
    setIsDrawerOpen(true);
    drawerState.current.isOpen = true;
  }, [navigation]);
  
  // Función para cerrar el drawer
  const closeDrawer = useCallback(() => {
    navigation.closeDrawer();
    setIsDrawerOpen(false);
    drawerState.current.isOpen = false;
  }, [navigation]);
  
  // Función para alternar el drawer
  const toggleDrawer = useCallback(() => {
    if (isDrawerOpen) {
      closeDrawer();
    } else {
      openDrawer();
    }
  }, [isDrawerOpen, openDrawer, closeDrawer]);
  
  // Función para navegar a una pantalla del drawer
  const navigateToDrawerScreen = useCallback((screenName, params = {}) => {
    // Cerrar el drawer antes de navegar
    closeDrawer();
    
    // Navegar a la pantalla
    navigation.navigate(screenName, params);
    
    // Registrar en historial
    drawerNavigationHistory.current.push({
      screen: screenName,
      params,
      timestamp: Date.now(),
    });
  }, [navigation, closeDrawer]);
  
  // Función para navegar y abrir el drawer
  const navigateAndOpenDrawer = useCallback((screenName, params = {}) => {
    // Navegar a la pantalla
    navigation.navigate(screenName, params);
    
    // Abrir el drawer después de un pequeño delay
    setTimeout(() => {
      openDrawer();
    }, 100);
  }, [navigation, openDrawer]);
  
  // Función para obtener estadísticas del drawer
  const getDrawerStats = useCallback(() => {
    return {
      isOpen: isDrawerOpen,
      openCount: drawerState.current.openCount,
      lastOpened: drawerState.current.lastOpened,
      history: [...drawerHistory],
      navigationHistory: [...drawerNavigationHistory.current],
    };
  }, [isDrawerOpen, drawerHistory]);
  
  // Función para limpiar historial del drawer
  const clearDrawerHistory = useCallback(() => {
    setDrawerHistory([]);
    drawerNavigationHistory.current = [];
  }, []);
  
  // Función para verificar si una ruta está en el drawer
  const isDrawerRoute = useCallback((routeName) => {
    const drawerRoutes = [
      'Home', 'Profile', 'Settings', 'Notifications', 'Messages',
      'Help', 'About'
    ];
    
    return drawerRoutes.some(route => routeName.includes(route));
  }, []);
  
  // Función para obtener la ruta actual del drawer
  const getCurrentDrawerRoute = useCallback(() => {
    const currentRoute = route.name;
    
    if (currentRoute.includes('Drawer')) {
      return currentRoute.replace('Drawer', '');
    }
    
    return currentRoute;
  }, [route.name]);
  
  // Función para navegar al drawer anterior
  const navigateToPreviousDrawer = useCallback(() => {
    if (drawerHistory.length > 1) {
      const previousDrawer = drawerHistory[drawerHistory.length - 2];
      navigateToDrawerScreen(previousDrawer.route);
    }
  }, [drawerHistory, navigateToDrawerScreen]);
  
  // Función para resetear el drawer
  const resetDrawer = useCallback(() => {
    closeDrawer();
    clearDrawerHistory();
    drawerState.current = {
      isOpen: false,
      lastOpened: null,
      openCount: 0,
    };
  }, [closeDrawer, clearDrawerHistory]);
  
  return {
    // Estado
    isDrawerOpen,
    drawerHistory,
    
    // Funciones de control
    openDrawer,
    closeDrawer,
    toggleDrawer,
    
    // Funciones de navegación
    navigateToDrawerScreen,
    navigateAndOpenDrawer,
    navigateToPreviousDrawer,
    
    // Funciones de información
    getDrawerStats,
    isDrawerRoute,
    getCurrentDrawerRoute,
    
    // Funciones de utilidad
    clearDrawerHistory,
    resetDrawer,
    
    // Objeto de navegación nativo
    navigation,
    route,
  };
};

export default useDrawerNavigation;
```

### **4. Utilidades para Drawer Navigator**

```javascript:src/utils/drawerNavigatorUtils.js
// Utilidades para Drawer Navigator en React Native

// Función para crear configuración del drawer
export const createDrawerConfig = (config) => {
  const {
    width = 280,
    backgroundColor = '#ffffff',
    elevation = 16,
    shadowColor = '#000',
    shadowOffset = { width: 4, height: 0 },
    shadowOpacity = 0.3,
    shadowRadius = 8,
    overlayColor = 'rgba(0, 0, 0, 0.5)',
    gestureEnabled = true,
    gestureHandlerProps = {},
    drawerType = 'front',
    drawerPosition = 'left',
  } = config;
  
  return {
    drawerStyle: {
      backgroundColor,
      width,
      elevation,
      shadowColor,
      shadowOffset,
      shadowOpacity,
      shadowRadius,
    },
    overlayColor,
    gestureEnabled,
    gestureHandlerProps,
    drawerType,
    drawerPosition,
  };
};

// Función para crear configuración del header del drawer
export const createDrawerHeaderConfig = (config) => {
  const {
    backgroundColor = '#f8f9fa',
    borderColor = '#e9ecef',
    avatarSize = 60,
    avatarBorderColor = '#007bff',
    userName = 'Usuario',
    userEmail = 'usuario@example.com',
    userStatus = 'En línea',
    showEditButton = true,
  } = config;
  
  return {
    drawerHeader: () => ({
      backgroundColor,
      borderColor,
      avatarSize,
      avatarBorderColor,
      userName,
      userEmail,
      userStatus,
      showEditButton,
    }),
  };
};

// Función para crear configuración de items del drawer
export const createDrawerItemConfig = (config) => {
  const {
    activeTintColor = '#007bff',
    inactiveTintColor = '#666',
    activeBackgroundColor = 'rgba(0, 123, 255, 0.1)',
    inactiveBackgroundColor = 'transparent',
    labelStyle = {},
    iconStyle = {},
    showActiveIndicator = true,
    activeIndicatorColor = '#007bff',
  } = config;
  
  return {
    drawerActiveTintColor: activeTintColor,
    drawerInactiveTintColor: inactiveTintColor,
    drawerActiveBackgroundColor: activeBackgroundColor,
    drawerInactiveBackgroundColor: inactiveBackgroundColor,
    drawerLabelStyle: {
      fontSize: 16,
      fontWeight: '500',
      marginLeft: 8,
      ...labelStyle,
    },
    drawerIconStyle: {
      width: 24,
      height: 24,
      ...iconStyle,
    },
    showActiveIndicator,
    activeIndicatorColor,
  };
};

// Función para crear configuración de gestos del drawer
export const createDrawerGestureConfig = (config) => {
  const {
    enabled = true,
    velocityThreshold = 0.3,
    directionalOffsetThreshold = 80,
    minSwipeDistance = 50,
    minSwipeVelocity = 500,
  } = config;
  
  return {
    gestureEnabled: enabled,
    gestureHandlerProps: {
      velocityThreshold,
      directionalOffsetThreshold,
      minSwipeDistance,
      minSwipeVelocity,
    },
  };
};

// Función para crear configuración de animaciones del drawer
export const createDrawerAnimationConfig = (config) => {
  const {
    type = 'front',
    position = 'left',
    animationType = 'slide',
    duration = 300,
    easing = 'ease',
  } = config;
  
  return {
    drawerType: type,
    drawerPosition: position,
    animationType,
    duration,
    easing,
  };
};

// Función para validar configuración del drawer
export const validateDrawerConfig = (screens) => {
  const errors = [];
  
  if (!screens || screens.length === 0) {
    errors.push('El drawer debe tener al menos una pantalla');
  }
  
  screens?.forEach((screen, index) => {
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

// Función para crear navegador de drawer optimizado
export const createOptimizedDrawerNavigator = (screens, options = {}) => {
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

// Función para crear configuración de deep linking para drawer
export const createDrawerDeepLinkingConfig = (config) => {
  const {
    prefixes = ['myapp://', 'https://myapp.com'],
    config: linkingConfig = {},
  } = config;
  
  return {
    prefixes,
    config: {
      screens: {
        // Configuración del drawer para deep linking
        HomeDrawer: {
          screens: linkingConfig.homeScreens || {},
        },
        ProfileDrawer: {
          screens: linkingConfig.profileScreens || {},
        },
        SettingsDrawer: {
          screens: linkingConfig.settingsScreens || {},
        },
        NotificationsDrawer: {
          screens: linkingConfig.notificationsScreens || {},
        },
        MessagesDrawer: {
          screens: linkingConfig.messagesScreens || {},
        },
        Help: {},
        About: {},
      },
      ...linkingConfig,
    },
  };
};
```

---

## 🧪 Casos de Uso

### **Caso 1: Navegación con Drawer**
```javascript
// Abrir/cerrar drawer
const handleDrawerToggle = () => {
  toggleDrawer();
};

// Navegar a una pantalla del drawer
const handleProfileNavigation = () => {
  navigateToDrawerScreen('ProfileDrawer');
};
```

### **Caso 2: Gestión de Estado del Drawer**
```javascript
// Verificar si el drawer está abierto
if (isDrawerOpen) {
  // Realizar acciones cuando el drawer está abierto
}

// Obtener estadísticas del drawer
const drawerStats = getDrawerStats();
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear Drawer Personalizado**
Implementa un Drawer Navigator con header personalizado y items estilizados.

### **Ejercicio 2: Navegación Anidada en Drawer**
Crea un drawer donde cada item tenga su propio stack de navegación.

### **Ejercicio 3: Gestión de Estado del Drawer**
Implementa un sistema que mantenga el estado del drawer y su historial.

---

## 🚀 Proyecto de la Clase

### **App de Drawer Avanzada**

Crea una aplicación que demuestre:
- **Drawer personalizado**: Header y items únicos
- **Navegación anidada**: Stacks dentro del drawer
- **Gestos personalizados**: Control del drawer mediante gestos
- **Estado persistente**: Mantener información del drawer

**Requisitos:**
1. Implementar Drawer Navigator con navegación anidada
2. Crear header personalizado del drawer
3. Implementar gestos personalizados
4. Usar el hook personalizado de drawer navigation
5. Mantener estado del drawer

**Estructura sugerida:**
```
src/
├── navigation/
│   ├── BasicDrawerNavigator.js
│   ├── NestedDrawerNavigator.js
│   └── DrawerNavigator.js
├── screens/
│   ├── HomeScreen.js
│   ├── ProfileScreen.js
│   ├── SettingsScreen.js
│   └── NotificationsScreen.js
├── hooks/
│   └── useDrawerNavigation.js
└── utils/
    └── drawerNavigatorUtils.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Navigation Drawer](https://reactnavigation.org/docs/drawer-navigator)
- [Custom Drawer Content](https://reactnavigation.org/docs/drawer-navigator#custom-drawer-content)

### **Artículos Recomendados:**
- "Cómo implementar drawer navigation en React Native"
- "Navegación anidada con drawer y stacks"
- "Mejores prácticas para drawer navigation"

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Drawer Navigator**: Menú lateral deslizable para navegación
- **Navegación anidada**: Stacks independientes dentro del drawer
- **Gestos personalizados**: Control del drawer mediante gestos táctiles
- **Header personalizado**: Contenido personalizable en la parte superior

### **Habilidades Desarrolladas:**
- ✅ Crear Drawer Navigator básico y avanzado
- ✅ Implementar navegación anidada con drawer y stacks
- ✅ Personalizar header, items y estilos del drawer
- ✅ Crear hooks personalizados para drawer navigation
- ✅ Gestionar estado y gestos del drawer

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Navegación Personalizada**, que te permitirá crear sistemas de navegación completamente personalizados y únicos.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Tab Navigator](clase_3_tab_navigator.md)
- **➡️ Siguiente Clase**: [Navegación Personalizada](clase_5_navegacion_personalizada.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
