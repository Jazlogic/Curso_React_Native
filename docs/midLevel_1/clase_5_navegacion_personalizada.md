# 📚 Clase 5: Navegación Personalizada

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 4: Drawer Navigator](clase_4_drawer_navigator.md)
- **➡️ Siguiente**: [Módulo 4: Estado y Gestión de Datos](../midLevel_2/README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender cómo crear navegación completamente personalizada
- Aprender a implementar transiciones y animaciones únicas
- Dominar la creación de navegadores personalizados
- Implementar gestos y interacciones avanzadas
- Crear experiencias de navegación únicas

---

## 📚 Contenido Teórico

### **¿Qué es la Navegación Personalizada?**

La navegación personalizada en React Native te permite crear sistemas de navegación completamente únicos, con transiciones personalizadas, gestos específicos y comportamientos que van más allá de los navegadores estándar.

#### **Características principales:**
- **Transiciones únicas**: Animaciones personalizadas entre pantallas
- **Gestos personalizados**: Interacciones táctiles específicas
- **Comportamiento único**: Lógica de navegación personalizada
- **Diseño flexible**: Estructura de navegación adaptable
- **Performance optimizada**: Navegación eficiente y fluida

---

## 💻 Implementación Práctica

### **1. Navegador Personalizado con Transiciones**

```javascript:src/navigation/CustomNavigator.js
import React, { useRef, useEffect } from 'react';
import { View, StyleSheet, Animated, PanGestureHandler, State } from 'react-native';
import { useNavigation, useRoute } from '@react-navigation/native';

// Navegador personalizado con transiciones únicas
const CustomNavigator = ({ children, screens }) => {
  const navigation = useNavigation();
  const route = useRoute();
  
  // Animaciones para transiciones
  const slideAnim = useRef(new Animated.Value(0)).current;
  const fadeAnim = useRef(new Animated.Value(1)).current;
  const scaleAnim = useRef(new Animated.Value(1)).current;
  
  // Estado de navegación
  const [currentScreen, setCurrentScreen] = useState(0);
  const [isTransitioning, setIsTransitioning] = useState(false);
  
  // Función para navegar con transición personalizada
  const navigateWithTransition = (screenIndex, direction = 'right') => {
    if (isTransitioning) return;
    
    setIsTransitioning(true);
    
    // Animación de salida
    Animated.parallel([
      Animated.timing(slideAnim, {
        toValue: direction === 'right' ? -300 : 300,
        duration: 300,
        useNativeDriver: true,
      }),
      Animated.timing(fadeAnim, {
        toValue: 0,
        duration: 300,
        useNativeDriver: true,
      }),
      Animated.timing(scaleAnim, {
        toValue: 0.8,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start(() => {
      // Cambiar pantalla
      setCurrentScreen(screenIndex);
      
      // Animación de entrada
      slideAnim.setValue(direction === 'right' ? 300 : -300);
      
      Animated.parallel([
        Animated.timing(slideAnim, {
          toValue: 0,
          duration: 300,
          useNativeDriver: true,
        }),
        Animated.timing(fadeAnim, {
          toValue: 1,
          duration: 300,
          useNativeDriver: true,
        }),
        Animated.timing(scaleAnim, {
          toValue: 1,
          duration: 300,
          useNativeDriver: true,
        }),
      ]).start(() => {
        setIsTransitioning(false);
      });
    });
  };
  
  // Gestos para navegación por swipe
  const onGestureEvent = Animated.event(
    [{ nativeEvent: { translationX: slideAnim } }],
    { useNativeDriver: true }
  );
  
  const onHandlerStateChange = (event) => {
    if (event.nativeEvent.state === State.END) {
      const { translationX } = event.nativeEvent;
      
      if (Math.abs(translationX) > 100) {
        if (translationX > 0 && currentScreen > 0) {
          // Swipe derecha - pantalla anterior
          navigateWithTransition(currentScreen - 1, 'left');
        } else if (translationX < 0 && currentScreen < screens.length - 1) {
          // Swipe izquierda - pantalla siguiente
          navigateWithTransition(currentScreen + 1, 'right');
        }
      } else {
        // Resetear posición
        Animated.spring(slideAnim, {
          toValue: 0,
          useNativeDriver: true,
        }).start();
      }
    }
  };
  
  return (
    <PanGestureHandler
      onGestureEvent={onGestureEvent}
      onHandlerStateChange={onHandlerStateChange}
    >
      <Animated.View style={styles.container}>
        <Animated.View
          style={[
            styles.screenContainer,
            {
              transform: [
                { translateX: slideAnim },
                { scale: scaleAnim },
              ],
              opacity: fadeAnim,
            },
          ]}
        >
          {screens[currentScreen]}
        </Animated.View>
        
        {/* Indicadores de navegación */}
        <View style={styles.navigationDots}>
          {screens.map((_, index) => (
            <View
              key={index}
              style={[
                styles.dot,
                index === currentScreen && styles.activeDot,
              ]}
            />
          ))}
        </View>
      </Animated.View>
    </PanGestureHandler>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000',
  },
  screenContainer: {
    flex: 1,
  },
  navigationDots: {
    position: 'absolute',
    bottom: 50,
    left: 0,
    right: 0,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
  },
  dot: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: 'rgba(255, 255, 255, 0.5)',
    marginHorizontal: 4,
  },
  activeDot: {
    backgroundColor: '#fff',
    transform: [{ scale: 1.2 }],
  },
});

export default CustomNavigator;
```

### **2. Hook de Navegación Personalizada**

```javascript:src/hooks/useCustomNavigation.js
import { useRef, useState, useCallback } from 'react';
import { Animated } from 'react-native';

// Hook para navegación personalizada
const useCustomNavigation = () => {
  const [navigationStack, setNavigationStack] = useState([]);
  const [currentRoute, setCurrentRoute] = useState(null);
  
  // Referencias para animaciones
  const slideAnim = useRef(new Animated.Value(0)).current;
  const fadeAnim = useRef(new Animated.Value(1)).current;
  
  // Navegar a una nueva pantalla
  const navigate = useCallback((routeName, params = {}) => {
    const newRoute = { name: routeName, params, timestamp: Date.now() };
    
    setNavigationStack(prev => [...prev, newRoute]);
    setCurrentRoute(newRoute);
    
    // Animación de entrada
    slideAnim.setValue(300);
    fadeAnim.setValue(0);
    
    Animated.parallel([
      Animated.timing(slideAnim, {
        toValue: 0,
        duration: 300,
        useNativeDriver: true,
      }),
      Animated.timing(fadeAnim, {
        toValue: 1,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start();
  }, []);
  
  // Volver a la pantalla anterior
  const goBack = useCallback(() => {
    if (navigationStack.length > 1) {
      const newStack = navigationStack.slice(0, -1);
      const previousRoute = newStack[newStack.length - 1];
      
      setNavigationStack(newStack);
      setCurrentRoute(previousRoute);
      
      // Animación de salida
      Animated.parallel([
        Animated.timing(slideAnim, {
          toValue: 300,
          duration: 300,
          useNativeDriver: true,
        }),
        Animated.timing(fadeAnim, {
          toValue: 0,
          duration: 300,
          useNativeDriver: true,
        }),
      ]).start();
    }
  }, [navigationStack]);
  
  // Resetear navegación
  const reset = useCallback((routeName, params = {}) => {
    const newRoute = { name: routeName, params, timestamp: Date.now() };
    setNavigationStack([newRoute]);
    setCurrentRoute(newRoute);
  }, []);
  
  return {
    navigate,
    goBack,
    reset,
    currentRoute,
    navigationStack,
    slideAnim,
    fadeAnim,
  };
};

export default useCustomNavigation;
```

---

## 🧪 Casos de Uso

### **Caso 1: Navegación con Transiciones Personalizadas**
```javascript
// Navegar con transición específica
const handleNavigation = () => {
  navigateWithTransition(1, 'right');
};
```

### **Caso 2: Gestos Personalizados**
```javascript
// Implementar gestos únicos
const onCustomGesture = (gesture) => {
  // Lógica personalizada del gesto
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Transiciones Únicas**
Implementa transiciones personalizadas entre pantallas.

### **Ejercicio 2: Gestos Avanzados**
Crea gestos personalizados para navegación.

### **Ejercicio 3: Navegador Personalizado**
Desarrolla un navegador completamente personalizado.

---

## 🚀 Proyecto de la Clase

### **App de Navegación Personalizada**

Crea una aplicación que demuestre:
- **Transiciones únicas**: Animaciones personalizadas
- **Gestos avanzados**: Interacciones táctiles únicas
- **Navegador personalizado**: Sistema de navegación único

**Requisitos:**
1. Implementar transiciones personalizadas
2. Crear gestos únicos de navegación
3. Desarrollar navegador personalizado
4. Usar hook de navegación personalizada
5. Optimizar performance de animaciones

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Navegación personalizada**: Sistemas únicos de navegación
- **Transiciones personalizadas**: Animaciones únicas entre pantallas
- **Gestos avanzados**: Interacciones táctiles específicas
- **Performance optimizada**: Navegación eficiente y fluida

### **Habilidades Desarrolladas:**
- ✅ Crear navegación completamente personalizada
- ✅ Implementar transiciones y animaciones únicas
- ✅ Desarrollar gestos y interacciones avanzadas
- ✅ Crear hooks de navegación personalizada
- ✅ Optimizar performance de navegación

### **Próximos Pasos:**
En el siguiente módulo aprenderemos sobre **Estado y Gestión de Datos**, que te permitirá manejar el estado de la aplicación de manera eficiente.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Drawer Navigator](clase_4_drawer_navigator.md)
- **➡️ Siguiente Módulo**: [Módulo 4: Estado y Gestión de Datos](../midLevel_2/README.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
