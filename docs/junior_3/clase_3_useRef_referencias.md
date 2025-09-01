# 📚 Clase 3: useRef y Referencias

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 2: useEffect y Ciclo de Vida](clase_2_useEffect_ciclo_vida.md)
- **➡️ Siguiente**: [Clase 4: Manejo de Eventos y Formularios](clase_4_manejo_eventos_formularios.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el hook useRef y sus casos de uso
- Dominar el acceso directo a elementos del DOM
- Implementar referencias para valores persistentes
- Gestionar focus, scroll y mediciones de elementos
- Crear componentes con control directo de elementos

---

## 📚 Contenido Teórico

### **¿Qué es useRef?**

`useRef` es un hook que devuelve un objeto mutable cuya propiedad `.current` se inicializa con el argumento pasado. El objeto devuelto persistirá durante toda la vida del componente.

#### **Casos de uso principales:**
- **Referencias a elementos**: Acceder directamente a componentes nativos
- **Valores persistentes**: Almacenar valores que no causan re-renderizados
- **Focus y scroll**: Controlar el comportamiento de inputs y scrollviews
- **Mediciones**: Obtener dimensiones y posiciones de elementos

---

## 💻 Implementación Práctica

### **1. useRef Básico**

```javascript:src/components/BasicUseRefExample.js
import React, { useRef, useState } from 'react';
import { View, Text, TouchableOpacity, TextInput, StyleSheet, ScrollView } from 'react-native';

// Componente que demuestra useRef básico
const BasicUseRefExample = () => {
  // Referencias a elementos
  const inputRef = useRef(null);
  const scrollViewRef = useRef(null);
  const textRef = useRef(null);
  
  // Referencias para valores persistentes
  const renderCountRef = useRef(0);
  const previousValueRef = useRef('');
  const timerRef = useRef(null);
  
  // Estado para demostrar funcionalidad
  const [inputValue, setInputValue] = useState('');
  const [focusCount, setFocusCount] = useState(0);
  const [scrollPosition, setScrollPosition] = useState(0);

  // Incrementar contador de renderizados
  renderCountRef.current += 1;

  // Función para enfocar el input
  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
      setFocusCount(prev => prev + 1);
    }
  };

  // Función para limpiar el input
  const clearInput = () => {
    if (inputRef.current) {
      inputRef.current.clear();
      setInputValue('');
      previousValueRef.current = '';
    }
  };

  // Función para hacer scroll al inicio
  const scrollToTop = () => {
    if (scrollViewRef.current) {
      scrollViewRef.current.scrollTo({ y: 0, animated: true });
    }
  };

  // Función para hacer scroll al final
  const scrollToBottom = () => {
    if (scrollViewRef.current) {
      scrollViewRef.current.scrollToEnd({ animated: true });
    }
  };

  // Función para obtener información del elemento de texto
  const getTextInfo = () => {
    if (textRef.current) {
      // En React Native, podemos obtener información básica
      console.log('Referencia al elemento de texto:', textRef.current);
    }
  };

  // Función para simular timer con ref
  const startTimer = () => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
    }
    
    timerRef.current = setInterval(() => {
      console.log('Timer ejecutándose...');
    }, 1000);
  };

  // Función para detener timer
  const stopTimer = () => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
      timerRef.current = null;
      console.log('Timer detenido');
    }
  };

  // Función para manejar cambio de input
  const handleInputChange = (text) => {
    previousValueRef.current = inputValue;
    setInputValue(text);
  };

  return (
    <ScrollView 
      ref={scrollViewRef}
      style={styles.container}
      onScroll={(event) => {
        setScrollPosition(event.nativeEvent.contentOffset.y);
      }}
      scrollEventThrottle={16}
    >
      <View style={styles.section}>
        <Text style={styles.title}>useRef Básico</Text>
        <Text style={styles.subtitle}>
          Demostración de referencias a elementos y valores persistentes
        </Text>
      </View>

      {/* Sección de Input con Ref */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>📝 Input con Referencia</Text>
        
        <TextInput
          ref={inputRef}
          style={styles.input}
          placeholder="Escribe algo aquí..."
          value={inputValue}
          onChangeText={handleInputChange}
          onFocus={() => console.log('Input enfocado')}
          onBlur={() => console.log('Input desenfocado')}
        />
        
        <View style={styles.buttonRow}>
          <TouchableOpacity style={styles.button} onPress={focusInput}>
            <Text style={styles.buttonText}>Enfocar</Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={styles.button} onPress={clearInput}>
            <Text style={styles.buttonText}>Limpiar</Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.infoText}>
          Valor actual: {inputValue || 'vacío'}{'\n'}
          Valor anterior: {previousValueRef.current || 'ninguno'}{'\n'}
          Veces enfocado: {focusCount}
        </Text>
      </View>

      {/* Sección de Scroll con Ref */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>📜 Control de Scroll</Text>
        
        <View style={styles.buttonRow}>
          <TouchableOpacity style={styles.button} onPress={scrollToTop}>
            <Text style={styles.buttonText}>Ir al Inicio</Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={styles.button} onPress={scrollToBottom}>
            <Text style={styles.buttonText}>Ir al Final</Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.infoText}>
          Posición de scroll: {scrollPosition.toFixed(0)}px
        </Text>
      </View>

      {/* Sección de Timer con Ref */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>⏱️ Timer con Referencia</Text>
        
        <View style={styles.buttonRow}>
          <TouchableOpacity style={styles.button} onPress={startTimer}>
            <Text style={styles.buttonText}>Iniciar Timer</Text>
          </TouchableOpacity>
          
          <TouchableOpacity style={styles.button} onPress={stopTimer}>
            <Text style={styles.buttonText}>Detener Timer</Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.infoText}>
          Timer activo: {timerRef.current ? 'Sí' : 'No'}
        </Text>
      </View>

      {/* Sección de Información */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>ℹ️ Información del Componente</Text>
        
        <Text style={styles.infoText}>
          Renderizados: {renderCountRef.current}{'\n'}
          Referencia al texto: {textRef.current ? 'Disponible' : 'No disponible'}
        </Text>
        
        <TouchableOpacity style={styles.infoButton} onPress={getTextInfo}>
          <Text style={styles.buttonText}>Obtener Info del Texto</Text>
        </TouchableOpacity>
      </View>

      {/* Elemento de texto con ref para demostración */}
      <Text 
        ref={textRef}
        style={styles.hiddenText}
        onLayout={(event) => {
          console.log('Layout del texto:', event.nativeEvent.layout);
        }}
      >
        Este texto tiene una referencia
      </Text>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 10,
    color: '#2c3e50',
  },
  subtitle: {
    fontSize: 16,
    textAlign: 'center',
    color: '#7f8c8d',
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#2c3e50',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    marginBottom: 15,
    fontSize: 16,
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 15,
  },
  button: {
    backgroundColor: '#3498db',
    padding: 15,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  infoButton: {
    backgroundColor: '#9b59b6',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 10,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  infoText: {
    fontSize: 14,
    color: '#7f8c8d',
    lineHeight: 20,
    textAlign: 'center',
  },
  hiddenText: {
    opacity: 0,
    height: 0,
  },
});

export default BasicUseRefExample;
```

### **2. useRef para Mediciones y Animaciones**

```javascript:src/components/AdvancedUseRefExample.js
import React, { useRef, useState, useEffect } from 'react';
import { 
  View, 
  Text, 
  TouchableOpacity, 
  StyleSheet, 
  Animated, 
  PanGestureHandler,
  State,
  Dimensions
} from 'react-native';

// Componente que demuestra useRef avanzado para mediciones y animaciones
const AdvancedUseRefExample = () => {
  // Referencias para elementos
  const containerRef = useRef(null);
  const animatedViewRef = useRef(null);
  const textRef = useRef(null);
  
  // Referencias para valores persistentes
  const panRef = useRef(null);
  const animationRef = useRef(null);
  const measurementRef = useRef(null);
  
  // Estado para mediciones
  const [measurements, setMeasurements] = useState({
    container: null,
    animatedView: null,
    text: null,
  });
  
  // Estado para animaciones
  const [isAnimating, setIsAnimating] = useState(false);
  const [animationType, setAnimationType] = useState('');

  // Valores animados
  const translateX = useRef(new Animated.Value(0)).current;
  const translateY = useRef(new Animated.Value(0)).current;
  const scale = useRef(new Animated.Value(1)).current;
  const opacity = useRef(new Animated.Value(1)).current;

  // Función para medir elementos
  const measureElement = (ref, name) => {
    if (ref.current) {
      ref.current.measure((x, y, width, height, pageX, pageY) => {
        const measurement = { x, y, width, height, pageX, pageY };
        setMeasurements(prev => ({
          ...prev,
          [name]: measurement,
        }));
        console.log(`Medición de ${name}:`, measurement);
      });
    }
  };

  // Función para medir todos los elementos
  const measureAllElements = () => {
    measureElement(containerRef, 'container');
    measureElement(animatedViewRef, 'animatedView');
    measureElement(textRef, 'text');
  };

  // Función para animación de fade
  const animateFade = () => {
    setIsAnimating(true);
    setAnimationType('Fade');
    
    Animated.sequence([
      Animated.timing(opacity, {
        toValue: 0,
        duration: 1000,
        useNativeDriver: true,
      }),
      Animated.timing(opacity, {
        toValue: 1,
        duration: 1000,
        useNativeDriver: true,
      }),
    ]).start(() => {
      setIsAnimating(false);
      setAnimationType('');
    });
  };

  // Función para animación de escala
  const animateScale = () => {
    setIsAnimating(true);
    setAnimationType('Scale');
    
    Animated.sequence([
      Animated.timing(scale, {
        toValue: 1.5,
        duration: 500,
        useNativeDriver: true,
      }),
      Animated.timing(scale, {
        toValue: 1,
        duration: 500,
        useNativeDriver: true,
      }),
    ]).start(() => {
      setIsAnimating(false);
      setAnimationType('');
    });
  };

  // Función para animación de movimiento
  const animateMovement = () => {
    setIsAnimating(true);
    setAnimationType('Movement');
    
    const screenWidth = Dimensions.get('window').width;
    const moveDistance = screenWidth * 0.3;
    
    Animated.sequence([
      Animated.timing(translateX, {
        toValue: moveDistance,
        duration: 1000,
        useNativeDriver: true,
      }),
      Animated.timing(translateX, {
        toValue: -moveDistance,
        duration: 1000,
        useNativeDriver: true,
      }),
      Animated.timing(translateX, {
        toValue: 0,
        duration: 500,
        useNativeDriver: true,
      }),
    ]).start(() => {
      setIsAnimating(false);
      setAnimationType('');
    });
  };

  // Función para resetear animaciones
  const resetAnimations = () => {
    translateX.setValue(0);
    translateY.setValue(0);
    scale.setValue(1);
    opacity.setValue(1);
    setIsAnimating(false);
    setAnimationType('');
  };

  // Función para obtener estadísticas de mediciones
  const getMeasurementStats = () => {
    const stats = {
      totalElements: Object.keys(measurements).length,
      measuredElements: Object.values(measurements).filter(m => m !== null).length,
      totalArea: 0,
      averageWidth: 0,
      averageHeight: 0,
    };

    const validMeasurements = Object.values(measurements).filter(m => m !== null);
    
    if (validMeasurements.length > 0) {
      stats.totalArea = validMeasurements.reduce((sum, m) => sum + (m.width * m.height), 0);
      stats.averageWidth = validMeasurements.reduce((sum, m) => sum + m.width, 0) / validMeasurements.length;
      stats.averageHeight = validMeasurements.reduce((sum, m) => sum + m.height, 0) / validMeasurements.length;
    }

    return stats;
  };

  return (
    <View style={styles.container} ref={containerRef}>
      <View style={styles.header}>
        <Text style={styles.title}>useRef Avanzado</Text>
        <Text style={styles.subtitle}>
          Mediciones, animaciones y control directo de elementos
        </Text>
      </View>

      {/* Sección de Mediciones */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>📏 Mediciones de Elementos</Text>
        
        <TouchableOpacity style={styles.button} onPress={measureAllElements}>
          <Text style={styles.buttonText}>Medir Todos los Elementos</Text>
        </TouchableOpacity>
        
        <View style={styles.measurementsContainer}>
          {Object.entries(measurements).map(([name, measurement]) => (
            <View key={name} style={styles.measurementItem}>
              <Text style={styles.measurementTitle}>{name}:</Text>
              {measurement ? (
                <Text style={styles.measurementText}>
                  {measurement.width.toFixed(0)} × {measurement.height.toFixed(0)}
                </Text>
              ) : (
                <Text style={styles.measurementText}>No medido</Text>
              )}
            </View>
          ))}
        </View>
        
        <TouchableOpacity style={styles.infoButton} onPress={() => {
          const stats = getMeasurementStats();
          console.log('Estadísticas de mediciones:', stats);
        }}>
          <Text style={styles.buttonText}>Ver Estadísticas</Text>
        </TouchableOpacity>
      </View>

      {/* Sección de Animaciones */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>🎭 Animaciones con Referencias</Text>
        
        <Animated.View
          ref={animatedViewRef}
          style={[
            styles.animatedView,
            {
              transform: [
                { translateX },
                { translateY },
                { scale },
              ],
              opacity,
            },
          ]}
        >
          <Text style={styles.animatedText}>Elemento Animado</Text>
        </Animated.View>
        
        <View style={styles.animationControls}>
          <TouchableOpacity 
            style={[styles.animButton, isAnimating && styles.disabledButton]} 
            onPress={animateFade}
            disabled={isAnimating}
          >
            <Text style={styles.buttonText}>Fade</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={[styles.animButton, isAnimating && styles.disabledButton]} 
            onPress={animateScale}
            disabled={isAnimating}
          >
            <Text style={styles.buttonText}>Scale</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={[styles.animButton, isAnimating && styles.disabledButton]} 
            onPress={animateMovement}
            disabled={isAnimating}
          >
            <Text style={styles.buttonText}>Move</Text>
          </TouchableOpacity>
        </View>
        
        <TouchableOpacity style={styles.resetButton} onPress={resetAnimations}>
          <Text style={styles.buttonText}>Reset Animaciones</Text>
        </TouchableOpacity>
        
        <Text style={styles.animationStatus}>
          Estado: {isAnimating ? `🔄 ${animationType}` : '⏸️ Pausado'}
        </Text>
      </View>

      {/* Elemento de texto para mediciones */}
      <Text 
        ref={textRef}
        style={styles.hiddenText}
        onLayout={(event) => {
          const { x, y, width, height } = event.nativeEvent.layout;
          setMeasurements(prev => ({
            ...prev,
            text: { x, y, width, height, pageX: x, pageY: y },
          }));
        }}
      >
        Texto oculto para mediciones
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  header: {
    marginBottom: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 10,
    color: '#2c3e50',
  },
  subtitle: {
    fontSize: 16,
    textAlign: 'center',
    color: '#7f8c8d',
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#2c3e50',
  },
  button: {
    backgroundColor: '#3498db',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 15,
  },
  infoButton: {
    backgroundColor: '#9b59b6',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 10,
  },
  resetButton: {
    backgroundColor: '#e74c3c',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 15,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  measurementsContainer: {
    marginBottom: 15,
  },
  measurementItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 5,
    borderBottomWidth: 1,
    borderBottomColor: '#ecf0f1',
  },
  measurementTitle: {
    fontSize: 14,
    fontWeight: '500',
    color: '#2c3e50',
  },
  measurementText: {
    fontSize: 14,
    color: '#7f8c8d',
  },
  animatedView: {
    backgroundColor: '#3498db',
    padding: 20,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 20,
    minHeight: 80,
  },
  animatedText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  animationControls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 15,
  },
  animButton: {
    backgroundColor: '#27ae60',
    padding: 12,
    borderRadius: 8,
    minWidth: 80,
    alignItems: 'center',
  },
  disabledButton: {
    backgroundColor: '#bdc3c7',
  },
  animationStatus: {
    fontSize: 14,
    textAlign: 'center',
    color: '#7f8c8d',
    fontStyle: 'italic',
  },
  hiddenText: {
    opacity: 0,
    height: 0,
  },
});

export default AdvancedUseRefExample;
```

### **3. Hook Personalizado para Referencias**

```javascript:src/hooks/useElementRef.js
import { useRef, useCallback, useEffect, useState } from 'react';

// Hook personalizado para gestión avanzada de referencias
const useElementRef = (options = {}) => {
  const {
    autoMeasure = false, // Medir automáticamente al montar
    trackChanges = false, // Rastrear cambios de dimensiones
    onMeasure, // Callback cuando se mide
    onLayout, // Callback cuando cambia el layout
  } = options;

  // Referencia al elemento
  const elementRef = useRef(null);
  
  // Estado para mediciones
  const [measurements, setMeasurements] = useState(null);
  
  // Estado para cambios de layout
  const [layout, setLayout] = useState(null);
  
  // Referencia para el último layout
  const lastLayoutRef = useRef(null);

  // Función para medir el elemento
  const measure = useCallback(() => {
    if (elementRef.current) {
      elementRef.current.measure((x, y, width, height, pageX, pageY) => {
        const measurement = { x, y, width, height, pageX, pageY };
        setMeasurements(measurement);
        
        if (onMeasure) {
          onMeasure(measurement);
        }
      });
    }
  }, [onMeasure]);

  // Función para obtener información del layout
  const getLayout = useCallback(() => {
    return layout;
  }, [layout]);

  // Función para obtener mediciones
  const getMeasurements = useCallback(() => {
    return measurements;
  }, [measurements]);

  // Función para verificar si el elemento está visible
  const isVisible = useCallback(() => {
    if (!measurements) return false;
    return measurements.width > 0 && measurements.height > 0;
  }, [measurements]);

  // Función para obtener el centro del elemento
  const getCenter = useCallback(() => {
    if (!measurements) return null;
    
    return {
      x: measurements.x + measurements.width / 2,
      y: measurements.y + measurements.height / 2,
    };
  }, [measurements]);

  // Función para verificar si un punto está dentro del elemento
  const containsPoint = useCallback((pointX, pointY) => {
    if (!measurements) return false;
    
    return (
      pointX >= measurements.x &&
      pointX <= measurements.x + measurements.width &&
      pointY >= measurements.y &&
      pointY <= measurements.y + measurements.height
    );
  }, [measurements]);

  // Función para obtener el área del elemento
  const getArea = useCallback(() => {
    if (!measurements) return 0;
    return measurements.width * measurements.height;
  }, [measurements]);

  // Función para obtener la relación de aspecto
  const getAspectRatio = useCallback(() => {
    if (!measurements || measurements.height === 0) return 0;
    return measurements.width / measurements.height;
  }, [measurements]);

  // Función para verificar cambios de layout
  const hasLayoutChanged = useCallback(() => {
    if (!layout || !lastLayoutRef.current) return false;
    
    const last = lastLayoutRef.current;
    return (
      layout.width !== last.width ||
      layout.height !== last.height ||
      layout.x !== last.x ||
      layout.y !== last.y
    );
  }, [layout]);

  // Efecto para medición automática
  useEffect(() => {
    if (autoMeasure) {
      measure();
    }
  }, [autoMeasure, measure]);

  // Efecto para rastrear cambios
  useEffect(() => {
    if (trackChanges && layout) {
      if (hasLayoutChanged()) {
        lastLayoutRef.current = layout;
        measure(); // Re-medir cuando cambia el layout
      }
    }
  }, [trackChanges, layout, hasLayoutChanged, measure]);

  // Función para manejar cambios de layout
  const handleLayout = useCallback((event) => {
    const newLayout = event.nativeEvent.layout;
    setLayout(newLayout);
    
    if (onLayout) {
      onLayout(newLayout);
    }
  }, [onLayout]);

  // Función para hacer scroll al elemento
  const scrollToElement = useCallback((scrollViewRef, animated = true) => {
    if (measurements && scrollViewRef?.current) {
      scrollViewRef.current.scrollTo({
        y: measurements.pageY,
        animated,
      });
    }
  }, [measurements]);

  // Función para enfocar el elemento (si es un input)
  const focus = useCallback(() => {
    if (elementRef.current?.focus) {
      elementRef.current.focus();
    }
  }, []);

  // Función para hacer blur del elemento (si es un input)
  const blur = useCallback(() => {
    if (elementRef.current?.blur) {
      elementRef.current.blur();
    }
  }, []);

  return {
    // Referencia
    ref: elementRef,
    
    // Estado
    measurements,
    layout,
    
    // Funciones de medición
    measure,
    getMeasurements,
    getLayout,
    
    // Funciones de utilidad
    isVisible,
    getCenter,
    containsPoint,
    getArea,
    getAspectRatio,
    hasLayoutChanged,
    
    // Funciones de control
    scrollToElement,
    focus,
    blur,
    
    // Event handler
    onLayout: handleLayout,
  };
};

export default useElementRef;
```

---

## 🧪 Casos de Uso

### **Caso 1: Referencia Básica**
```javascript
const inputRef = useRef(null);
// Enfocar input
inputRef.current?.focus();
```

### **Caso 2: Referencia para Mediciones**
```javascript
const { ref, measure, getMeasurements } = useElementRef();
// Medir elemento
measure();
```

### **Caso 3: Referencia para Animaciones**
```javascript
const animatedRef = useRef(new Animated.Value(0)).current;
// Animar valor
Animated.timing(animatedRef, { toValue: 1, duration: 1000 }).start();
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Input con Auto-focus**
Crea un input que se enfoque automáticamente al montar el componente.

### **Ejercicio 2: Medidor de Elementos**
Implementa un componente que mida y muestre las dimensiones de otros elementos.

### **Ejercicio 3: Animación Controlada**
Crea una animación que se controle mediante referencias y botones.

---

## 🚀 Proyecto de la Clase

### **App de Mediciones y Animaciones**

Crea una aplicación que demuestre:
- **Mediciones en tiempo real**: Dimensiones y posiciones de elementos
- **Animaciones controladas**: Fade, scale, movement con referencias
- **Control directo**: Focus, scroll y manipulación de elementos
- **Hooks personalizados**: Lógica reutilizable para referencias

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **useRef**: Hook para referencias mutables
- **Referencias a elementos**: Acceso directo a componentes nativos
- **Valores persistentes**: Datos que no causan re-renderizados
- **Mediciones y animaciones**: Control preciso de elementos

### **Habilidades Desarrolladas:**
- ✅ Usar useRef para referencias a elementos
- ✅ Implementar mediciones y control de layout
- ✅ Crear animaciones controladas por referencias
- ✅ Desarrollar hooks personalizados para referencias

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Manejo de Eventos y Formularios**, que te permitirá crear interfaces interactivas y manejar la entrada del usuario.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [useEffect y Ciclo de Vida](clase_2_useEffect_ciclo_vida.md)
- **➡️ Siguiente Clase**: [Manejo de Eventos y Formularios](clase_4_manejo_eventos_formularios.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
