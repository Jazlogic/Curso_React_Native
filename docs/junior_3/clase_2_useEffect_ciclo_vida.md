# 📚 Clase 2: useEffect y Ciclo de Vida

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 1: Estado Local y useState](clase_1_estado_local_useState.md)
- **➡️ Siguiente**: [Clase 3: useRef y Referencias](clase_3_useRef_referencias.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el hook useEffect y sus casos de uso
- Dominar el ciclo de vida de componentes funcionales
- Implementar efectos secundarios de manera eficiente
- Gestionar suscripciones y limpieza de recursos
- Crear componentes con comportamiento controlado

---

## 📚 Contenido Teórico

### **¿Qué es useEffect?**

`useEffect` es un hook que permite ejecutar código con efectos secundarios en componentes funcionales. Es el equivalente a `componentDidMount`, `componentDidUpdate` y `componentWillUnmount` combinados.

#### **Casos de uso principales:**
- **Llamadas a APIs**: Obtener datos del servidor
- **Suscripciones**: Event listeners, timers, WebSockets
- **Sincronización**: Actualizar estado basado en props o estado
- **Efectos de limpieza**: Limpiar recursos cuando el componente se desmonta

---

## 💻 Implementación Práctica

### **1. useEffect Básico**

```javascript:src/components/BasicUseEffectExample.js
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Componente que demuestra useEffect básico
const BasicUseEffectExample = () => {
  // Estado para contador
  const [count, setCount] = useState(0);
  
  // Estado para mostrar información del efecto
  const [effectInfo, setEffectInfo] = useState('');

  // useEffect que se ejecuta en cada renderizado
  useEffect(() => {
    setEffectInfo(`useEffect ejecutado - Contador: ${count}`);
    console.log('useEffect ejecutado - Contador:', count);
  });

  // useEffect que se ejecuta solo al montar el componente
  useEffect(() => {
    setEffectInfo('Componente montado - useEffect inicial');
    console.log('Componente montado');
    
    // Simular llamada a API
    const timer = setTimeout(() => {
      setEffectInfo('Datos cargados después de 2 segundos');
    }, 2000);

    // Función de limpieza
    return () => {
      clearTimeout(timer);
      console.log('Componente desmontado - Timer limpiado');
    };
  }, []); // Array vacío = solo al montar

  // useEffect que se ejecuta cuando cambia count
  useEffect(() => {
    if (count > 0) {
      setEffectInfo(`Contador actualizado a: ${count}`);
      console.log('Contador actualizado:', count);
    }
  }, [count]); // Dependencia: count

  // Función para incrementar contador
  const incrementCount = () => {
    setCount(prevCount => prevCount + 1);
  };

  // Función para resetear contador
  const resetCount = () => {
    setCount(0);
  };

  return (
    <View style={styles.container}>
      <View style={styles.section}>
        <Text style={styles.title}>useEffect Básico</Text>
        <Text style={styles.subtitle}>Contador: {count}</Text>
        
        <View style={styles.buttonRow}>
          <TouchableOpacity 
            style={styles.button} 
            onPress={incrementCount}
          >
            <Text style={styles.buttonText}>Incrementar</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={styles.resetButton} 
            onPress={resetCount}
          >
            <Text style={styles.buttonText}>Reset</Text>
          </TouchableOpacity>
        </View>
      </View>

      <View style={styles.section}>
        <Text style={styles.title}>Información del Efecto</Text>
        <Text style={styles.effectText}>{effectInfo}</Text>
      </View>

      <View style={styles.section}>
        <Text style={styles.title}>Explicación</Text>
        <Text style={styles.explanationText}>
          • Primer useEffect: Se ejecuta en cada renderizado{'\n'}
          • Segundo useEffect: Solo al montar (componentDidMount){'\n'}
          • Tercer useEffect: Cuando cambia count{'\n'}
          • Función de retorno: Limpieza al desmontar
        </Text>
      </View>
    </View>
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
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#2c3e50',
    textAlign: 'center',
  },
  subtitle: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#3498db',
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  button: {
    backgroundColor: '#3498db',
    padding: 15,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#e74c3c',
    padding: 15,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  effectText: {
    fontSize: 16,
    textAlign: 'center',
    color: '#27ae60',
    fontStyle: 'italic',
    padding: 15,
    backgroundColor: '#f8f9fa',
    borderRadius: 8,
  },
  explanationText: {
    fontSize: 14,
    color: '#7f8c8d',
    lineHeight: 20,
  },
});

export default BasicUseEffectExample;
```

### **2. useEffect con Suscripciones**

```javascript:src/components/SubscriptionExample.js
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, ScrollView } from 'react-native';

// Componente que demuestra useEffect con suscripciones
const SubscriptionExample = () => {
  // Estado para diferentes tipos de datos
  const [time, setTime] = useState(new Date());
  const [location, setLocation] = useState({ x: 0, y: 0 });
  const [networkStatus, setNetworkStatus] = useState('online');
  const [batteryLevel, setBatteryLevel] = useState(100);
  
  // Referencias para limpiar suscripciones
  const timeIntervalRef = useRef(null);
  const locationIntervalRef = useRef(null);
  const networkIntervalRef = useRef(null);
  const batteryIntervalRef = useRef(null);

  // Estado para controlar suscripciones
  const [subscriptions, setSubscriptions] = useState({
    time: false,
    location: false,
    network: false,
    battery: false,
  });

  // useEffect para suscripción de tiempo
  useEffect(() => {
    if (subscriptions.time) {
      // Crear intervalo para actualizar tiempo cada segundo
      timeIntervalRef.current = setInterval(() => {
        setTime(new Date());
      }, 1000);

      // Función de limpieza
      return () => {
        if (timeIntervalRef.current) {
          clearInterval(timeIntervalRef.current);
          timeIntervalRef.current = null;
        }
      };
    }
  }, [subscriptions.time]);

  // useEffect para suscripción de ubicación simulada
  useEffect(() => {
    if (subscriptions.location) {
      // Simular actualización de ubicación cada 2 segundos
      locationIntervalRef.current = setInterval(() => {
        setLocation(prev => ({
          x: prev.x + (Math.random() - 0.5) * 10,
          y: prev.y + (Math.random() - 0.5) * 10,
        }));
      }, 2000);

      return () => {
        if (locationIntervalRef.current) {
          clearInterval(locationIntervalRef.current);
          locationIntervalRef.current = null;
        }
      };
    }
  }, [subscriptions.location]);

  // useEffect para suscripción de estado de red
  useEffect(() => {
    if (subscriptions.network) {
      // Simular cambios de estado de red cada 3 segundos
      networkIntervalRef.current = setInterval(() => {
        setNetworkStatus(prev => prev === 'online' ? 'offline' : 'online');
      }, 3000);

      return () => {
        if (networkIntervalRef.current) {
          clearInterval(networkIntervalRef.current);
          networkIntervalRef.current = null;
        }
      };
    }
  }, [subscriptions.network]);

  // useEffect para suscripción de nivel de batería
  useEffect(() => {
    if (subscriptions.battery) {
      // Simular descarga de batería cada 4 segundos
      batteryIntervalRef.current = setInterval(() => {
        setBatteryLevel(prev => Math.max(0, prev - 1));
      }, 4000);

      return () => {
        if (batteryIntervalRef.current) {
          clearInterval(batteryIntervalRef.current);
          batteryIntervalRef.current = null;
        }
      };
    }
  }, [subscriptions.battery]);

  // Función para alternar suscripción
  const toggleSubscription = (type) => {
    setSubscriptions(prev => ({
      ...prev,
      [type]: !prev[type],
    }));
  };

  // Función para limpiar todas las suscripciones
  const clearAllSubscriptions = () => {
    setSubscriptions({
      time: false,
      location: false,
      network: false,
      battery: false,
    });
  };

  // Función para activar todas las suscripciones
  const activateAllSubscriptions = () => {
    setSubscriptions({
      time: true,
      location: true,
      network: true,
      battery: true,
    });
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.section}>
        <Text style={styles.title}>Gestión de Suscripciones</Text>
        <Text style={styles.subtitle}>
          Demostración de useEffect con diferentes tipos de suscripciones
        </Text>
      </View>

      {/* Suscripción de Tiempo */}
      <View style={styles.section}>
        <View style={styles.headerRow}>
          <Text style={styles.sectionTitle}>⏰ Reloj en Tiempo Real</Text>
          <TouchableOpacity 
            style={[
              styles.toggleButton, 
              subscriptions.time && styles.activeToggle
            ]}
            onPress={() => toggleSubscription('time')}
          >
            <Text style={styles.toggleText}>
              {subscriptions.time ? 'ON' : 'OFF'}
            </Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.dataText}>
          {time.toLocaleTimeString()}
        </Text>
        
        <Text style={styles.statusText}>
          Estado: {subscriptions.time ? '🟢 Activo' : '🔴 Inactivo'}
        </Text>
      </View>

      {/* Suscripción de Ubicación */}
      <View style={styles.section}>
        <View style={styles.headerRow}>
          <Text style={styles.sectionTitle}>📍 Ubicación Simulada</Text>
          <TouchableOpacity 
            style={[
              styles.toggleButton, 
              subscriptions.location && styles.activeToggle
            ]}
            onPress={() => toggleSubscription('location')}
          >
            <Text style={styles.toggleText}>
              {subscriptions.location ? 'ON' : 'OFF'}
            </Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.dataText}>
          X: {location.x.toFixed(2)}, Y: {location.y.toFixed(2)}
        </Text>
        
        <Text style={styles.statusText}>
          Estado: {subscriptions.location ? '🟢 Activo' : '🔴 Inactivo'}
        </Text>
      </View>

      {/* Suscripción de Red */}
      <View style={styles.section}>
        <View style={styles.headerRow}>
          <Text style={styles.sectionTitle}>🌐 Estado de Red</Text>
          <TouchableOpacity 
            style={[
              styles.toggleButton, 
              subscriptions.network && styles.activeToggle
            ]}
            onPress={() => toggleSubscription('network')}
          >
            <Text style={styles.toggleText}>
              {subscriptions.network ? 'ON' : 'OFF'}
            </Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.dataText}>
          {networkStatus.toUpperCase()}
        </Text>
        
        <Text style={styles.statusText}>
          Estado: {subscriptions.network ? '🟢 Activo' : '🔴 Inactivo'}
        </Text>
      </View>

      {/* Suscripción de Batería */}
      <View style={styles.section}>
        <View style={styles.headerRow}>
          <Text style={styles.sectionTitle}>🔋 Nivel de Batería</Text>
          <TouchableOpacity 
            style={[
              styles.toggleButton, 
              subscriptions.battery && styles.activeToggle
            ]}
            onPress={() => toggleSubscription('battery')}
          >
            <Text style={styles.toggleText}>
              {subscriptions.battery ? 'ON' : 'OFF'}
            </Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.dataText}>
          {batteryLevel}%
        </Text>
        
        <Text style={styles.statusText}>
          Estado: {subscriptions.battery ? '🟢 Activo' : '🔴 Inactivo'}
        </Text>
      </View>

      {/* Controles Globales */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>🎛️ Controles Globales</Text>
        
        <View style={styles.buttonRow}>
          <TouchableOpacity 
            style={styles.controlButton} 
            onPress={activateAllSubscriptions}
          >
            <Text style={styles.buttonText}>Activar Todas</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={styles.controlButton} 
            onPress={clearAllSubscriptions}
          >
            <Text style={styles.buttonText}>Desactivar Todas</Text>
          </TouchableOpacity>
        </View>
      </View>

      {/* Información */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>ℹ️ Información</Text>
        <Text style={styles.infoText}>
          • Cada suscripción se gestiona con useEffect{'\n'}
          • Las funciones de limpieza evitan memory leaks{'\n'}
          • Los intervalos se crean/eliminan según el estado{'\n'}
          • Las suscripciones son independientes entre sí
        </Text>
      </View>
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
  headerRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#2c3e50',
    flex: 1,
  },
  toggleButton: {
    backgroundColor: '#e74c3c',
    paddingHorizontal: 20,
    paddingVertical: 8,
    borderRadius: 20,
    minWidth: 60,
    alignItems: 'center',
  },
  activeToggle: {
    backgroundColor: '#27ae60',
  },
  toggleText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  dataText: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    color: '#3498db',
    marginBottom: 10,
  },
  statusText: {
    fontSize: 14,
    textAlign: 'center',
    color: '#7f8c8d',
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginTop: 15,
  },
  controlButton: {
    backgroundColor: '#9b59b6',
    padding: 15,
    borderRadius: 8,
    minWidth: 140,
    alignItems: 'center',
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
  },
});

export default SubscriptionExample;
```

### **3. Hook Personalizado para Efectos**

```javascript:src/hooks/useInterval.js
import { useEffect, useRef } from 'react';

// Hook personalizado para intervalos con useEffect
const useInterval = (callback, delay, options = {}) => {
  const {
    immediate = false, // Ejecutar inmediatamente
    pauseOnBlur = false, // Pausar cuando la app no está en foco
    maxExecutions = null, // Número máximo de ejecuciones
  } = options;

  const savedCallback = useRef();
  const intervalRef = useRef(null);
  const executionCount = useRef(0);
  const isPaused = useRef(false);

  // Guardar callback en ref para evitar recrear el efecto
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Efecto principal para el intervalo
  useEffect(() => {
    // Función que se ejecutará en cada intervalo
    const tick = () => {
      if (isPaused.current) return;

      // Verificar límite de ejecuciones
      if (maxExecutions && executionCount.current >= maxExecutions) {
        clearInterval(intervalRef.current);
        return;
      }

      // Ejecutar callback
      if (savedCallback.current) {
        savedCallback.current();
        executionCount.current += 1;
      }
    };

    // Ejecutar inmediatamente si está habilitado
    if (immediate && delay !== null) {
      tick();
    }

    // Crear intervalo
    if (delay !== null) {
      intervalRef.current = setInterval(tick, delay);
    }

    // Función de limpieza
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
    };
  }, [delay, immediate, maxExecutions]);

  // Efecto para manejar pausa en blur (opcional)
  useEffect(() => {
    if (!pauseOnBlur) return;

    const handleAppStateChange = (nextAppState) => {
      if (nextAppState === 'active') {
        isPaused.current = false;
      } else if (nextAppState === 'background' || nextAppState === 'inactive') {
        isPaused.current = true;
      }
    };

    // Aquí normalmente usarías AppState de React Native
    // Para este ejemplo, simulamos con un listener personalizado
    
    return () => {
      // Limpiar listener si es necesario
    };
  }, [pauseOnBlur]);

  // Función para pausar manualmente
  const pause = () => {
    isPaused.current = true;
  };

  // Función para reanudar manualmente
  const resume = () => {
    isPaused.current = false;
  };

  // Función para resetear contador de ejecuciones
  const reset = () => {
    executionCount.current = 0;
  };

  // Función para obtener estadísticas
  const getStats = () => ({
    executionCount: executionCount.current,
    isPaused: isPaused.current,
    isActive: intervalRef.current !== null,
    maxExecutions,
    hasReachedMax: maxExecutions ? executionCount.current >= maxExecutions : false,
  });

  return {
    pause,
    resume,
    reset,
    getStats,
  };
};

export default useInterval;
```

---

## 🧪 Casos de Uso

### **Caso 1: useEffect Básico**
```javascript
useEffect(() => {
  console.log('Componente montado');
  return () => console.log('Componente desmontado');
}, []);
```

### **Caso 2: useEffect con Dependencias**
```javascript
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

### **Caso 3: useEffect con Limpieza**
```javascript
useEffect(() => {
  const subscription = api.subscribe();
  return () => subscription.unsubscribe();
}, []);
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Timer con Pausa**
Crea un timer que se pueda pausar y reanudar usando useEffect.

### **Ejercicio 2: Fetch de Datos**
Implementa un componente que obtenga datos de una API y los muestre.

### **Ejercicio 3: Geolocalización Simulada**
Crea un componente que simule actualizaciones de ubicación cada 5 segundos.

---

## 🚀 Proyecto de la Clase

### **App de Monitoreo en Tiempo Real**

Crea una aplicación que demuestre:
- **Múltiples suscripciones**: Tiempo, ubicación, red, batería
- **Gestión de recursos**: Limpieza automática de suscripciones
- **Hooks personalizados**: Lógica reutilizable para efectos
- **Control de suscripciones**: Activar/desactivar dinámicamente

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **useEffect**: Hook para efectos secundarios
- **Ciclo de vida**: Montaje, actualización y desmontaje
- **Limpieza**: Función de retorno para evitar memory leaks
- **Dependencias**: Control de cuándo se ejecuta el efecto

### **Habilidades Desarrolladas:**
- ✅ Implementar useEffect para diferentes casos de uso
- ✅ Gestionar suscripciones y recursos
- ✅ Crear hooks personalizados para efectos
- ✅ Controlar el ciclo de vida de componentes

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **useRef y Referencias**, que te permitirá acceder directamente a elementos del DOM y mantener valores persistentes.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Estado Local y useState](clase_1_estado_local_useState.md)
- **➡️ Siguiente Clase**: [useRef y Referencias](clase_3_useRef_referencias.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
