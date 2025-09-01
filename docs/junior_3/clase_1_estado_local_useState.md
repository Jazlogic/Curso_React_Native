# 📚 Clase 1: Estado Local y useState

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Módulo 2: Componentes Básicos](../junior_2/README.md)
- **➡️ Siguiente**: [Clase 2: useEffect y Ciclo de Vida](clase_2_useEffect_ciclo_vida.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el concepto de estado en React Native
- Dominar el hook useState para gestión de estado local
- Implementar patrones de estado para diferentes tipos de datos
- Crear componentes interactivos con estado
- Aprender mejores prácticas para el manejo de estado

---

## 📚 Contenido Teórico

### **¿Qué es el Estado en React Native?**

El estado es la información que puede cambiar durante la ejecución de una aplicación. En React Native, el estado determina cómo se renderiza un componente y cómo responde a las interacciones del usuario.

#### **Características del estado:**
- **Reactivo**: Cambios en el estado provocan re-renderizados
- **Local**: Cada componente mantiene su propio estado
- **Inmutable**: Nunca se modifica directamente, solo a través de funciones
- **Asíncrono**: Las actualizaciones pueden ser agrupadas por React

---

## 💻 Implementación Práctica

### **1. Hook useState Básico**

```javascript:src/components/BasicStateExample.js
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Componente que demuestra el uso básico de useState
const BasicStateExample = () => {
  // Estado simple para un contador
  const [count, setCount] = useState(0);
  
  // Estado para texto
  const [message, setMessage] = useState('Hola Mundo');
  
  // Estado booleano
  const [isVisible, setIsVisible] = useState(true);

  // Función para incrementar contador
  const incrementCount = () => {
    setCount(prevCount => prevCount + 1);
  };

  // Función para decrementar contador
  const decrementCount = () => {
    setCount(prevCount => prevCount - 1);
  };

  // Función para resetear contador
  const resetCount = () => {
    setCount(0);
  };

  // Función para cambiar mensaje
  const changeMessage = () => {
    setMessage('¡Estado actualizado!');
  };

  // Función para alternar visibilidad
  const toggleVisibility = () => {
    setIsVisible(prev => !prev);
  };

  return (
    <View style={styles.container}>
      {/* Sección del contador */}
      <View style={styles.section}>
        <Text style={styles.title}>Contador: {count}</Text>
        
        <View style={styles.buttonRow}>
          <TouchableOpacity 
            style={styles.button} 
            onPress={decrementCount}
          >
            <Text style={styles.buttonText}>-</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={styles.button} 
            onPress={incrementCount}
          >
            <Text style={styles.buttonText}>+</Text>
          </TouchableOpacity>
        </View>
        
        <TouchableOpacity 
          style={styles.resetButton} 
          onPress={resetCount}
        >
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
      </View>

      {/* Sección del mensaje */}
      <View style={styles.section}>
        <Text style={styles.subtitle}>{message}</Text>
        <TouchableOpacity 
          style={styles.button} 
          onPress={changeMessage}
        >
          <Text style={styles.buttonText}>Cambiar Mensaje</Text>
        </TouchableOpacity>
      </View>

      {/* Sección de visibilidad */}
      <View style={styles.section}>
        <TouchableOpacity 
          style={styles.button} 
          onPress={toggleVisibility}
        >
          <Text style={styles.buttonText}>
            {isVisible ? 'Ocultar' : 'Mostrar'}
          </Text>
        </TouchableOpacity>
        
        {isVisible && (
          <Text style={styles.subtitle}>¡Este texto es visible!</Text>
        )}
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
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#2c3e50',
  },
  subtitle: {
    fontSize: 18,
    textAlign: 'center',
    marginBottom: 15,
    color: '#34495e',
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
    minWidth: 80,
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#e74c3c',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default BasicStateExample;
```

### **2. Estado con Objetos y Arrays**

```javascript:src/components/ComplexStateExample.js
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, TextInput, StyleSheet, ScrollView } from 'react-native';

// Componente que demuestra estado complejo con objetos y arrays
const ComplexStateExample = () => {
  // Estado para objeto de usuario
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: '',
    isActive: false,
  });

  // Estado para lista de tareas
  const [tasks, setTasks] = useState([
    { id: 1, text: 'Aprender React Native', completed: false },
    { id: 2, text: 'Practicar useState', completed: true },
    { id: 3, text: 'Crear componentes', completed: false },
  ]);

  // Estado para contador de tareas
  const [taskCount, setTaskCount] = useState(3);

  // Función para actualizar campo del usuario
  const updateUserField = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value,
    }));
  };

  // Función para alternar estado activo del usuario
  const toggleUserStatus = () => {
    setUser(prevUser => ({
      ...prevUser,
      isActive: !prevUser.isActive,
    }));
  };

  // Función para agregar nueva tarea
  const addTask = () => {
    if (user.name.trim()) {
      const newTask = {
        id: taskCount + 1,
        text: user.name,
        completed: false,
      };
      
      setTasks(prevTasks => [...prevTasks, newTask]);
      setTaskCount(prevCount => prevCount + 1);
      setUser(prevUser => ({ ...prevUser, name: '' }));
    }
  };

  // Función para alternar estado de tarea
  const toggleTask = (taskId) => {
    setTasks(prevTasks =>
      prevTasks.map(task =>
        task.id === taskId
          ? { ...task, completed: !task.completed }
          : task
      )
    );
  };

  // Función para eliminar tarea
  const deleteTask = (taskId) => {
    setTasks(prevTasks => prevTasks.filter(task => task.id !== taskId));
  };

  // Función para limpiar tareas completadas
  const clearCompleted = () => {
    setTasks(prevTasks => prevTasks.filter(task => !task.completed));
  };

  // Calcular estadísticas
  const completedTasks = tasks.filter(task => task.completed).length;
  const pendingTasks = tasks.length - completedTasks;

  return (
    <ScrollView style={styles.container}>
      {/* Sección de usuario */}
      <View style={styles.section}>
        <Text style={styles.title}>Información del Usuario</Text>
        
        <TextInput
          style={styles.input}
          placeholder="Nombre"
          value={user.name}
          onChangeText={(text) => updateUserField('name', text)}
        />
        
        <TextInput
          style={styles.input}
          placeholder="Email"
          value={user.email}
          onChangeText={(text) => updateUserField('email', text)}
          keyboardType="email-address"
        />
        
        <TextInput
          style={styles.input}
          placeholder="Edad"
          value={user.age}
          onChangeText={(text) => updateUserField('age', text)}
          keyboardType="numeric"
        />
        
        <TouchableOpacity 
          style={[styles.button, user.isActive && styles.activeButton]} 
          onPress={toggleUserStatus}
        >
          <Text style={styles.buttonText}>
            {user.isActive ? 'Usuario Activo' : 'Usuario Inactivo'}
          </Text>
        </TouchableOpacity>
        
        <Text style={styles.statusText}>
          Estado: {user.isActive ? '✅ Activo' : '❌ Inactivo'}
        </Text>
      </View>

      {/* Sección de tareas */}
      <View style={styles.section}>
        <Text style={styles.title}>Gestor de Tareas</Text>
        
        <View style={styles.addTaskRow}>
          <TextInput
            style={[styles.input, styles.taskInput]}
            placeholder="Nueva tarea"
            value={user.name}
            onChangeText={(text) => updateUserField('name', text)}
          />
          <TouchableOpacity style={styles.addButton} onPress={addTask}>
            <Text style={styles.buttonText}>+</Text>
          </TouchableOpacity>
        </View>
        
        <View style={styles.statsRow}>
          <Text style={styles.statsText}>Total: {tasks.length}</Text>
          <Text style={styles.statsText}>Completadas: {completedTasks}</Text>
          <Text style={styles.statsText}>Pendientes: {pendingTasks}</Text>
        </View>
        
        {tasks.map(task => (
          <View key={task.id} style={styles.taskRow}>
            <TouchableOpacity 
              style={styles.taskButton}
              onPress={() => toggleTask(task.id)}
            >
              <Text style={styles.taskButtonText}>
                {task.completed ? '✅' : '⭕'}
              </Text>
            </TouchableOpacity>
            
            <Text style={[
              styles.taskText, 
              task.completed && styles.completedTask
            ]}>
              {task.text}
            </Text>
            
            <TouchableOpacity 
              style={styles.deleteButton}
              onPress={() => deleteTask(task.id)}
            >
              <Text style={styles.deleteButtonText}>🗑️</Text>
            </TouchableOpacity>
          </View>
        ))}
        
        <TouchableOpacity style={styles.clearButton} onPress={clearCompleted}>
          <Text style={styles.buttonText}>Limpiar Completadas</Text>
        </TouchableOpacity>
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
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
    color: '#2c3e50',
    textAlign: 'center',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    marginBottom: 15,
    fontSize: 16,
  },
  taskInput: {
    flex: 1,
    marginBottom: 0,
    marginRight: 10,
  },
  button: {
    backgroundColor: '#3498db',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 10,
  },
  activeButton: {
    backgroundColor: '#27ae60',
  },
  addButton: {
    backgroundColor: '#27ae60',
    padding: 15,
    borderRadius: 8,
    width: 50,
    alignItems: 'center',
  },
  clearButton: {
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
  statusText: {
    fontSize: 16,
    textAlign: 'center',
    color: '#7f8c8d',
  },
  addTaskRow: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 15,
  },
  statsRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20,
    paddingVertical: 10,
    backgroundColor: '#ecf0f1',
    borderRadius: 8,
  },
  statsText: {
    fontSize: 14,
    color: '#2c3e50',
    fontWeight: '500',
  },
  taskRow: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#ecf0f1',
  },
  taskButton: {
    marginRight: 15,
  },
  taskButtonText: {
    fontSize: 20,
  },
  taskText: {
    flex: 1,
    fontSize: 16,
    color: '#2c3e50',
  },
  completedTask: {
    textDecorationLine: 'line-through',
    color: '#7f8c8d',
  },
  deleteButton: {
    padding: 5,
  },
  deleteButtonText: {
    fontSize: 18,
  },
});

export default ComplexStateExample;
```

### **3. Hook Personalizado para Estado**

```javascript:src/hooks/useCounter.js
import { useState, useCallback } from 'react';

// Hook personalizado para gestión de contador
const useCounter = (initialValue = 0, options = {}) => {
  const {
    min = -Infinity,
    max = Infinity,
    step = 1,
    allowNegative = true,
    autoReset = false,
    resetValue = 0,
  } = options;

  // Estado del contador
  const [count, setCount] = useState(initialValue);
  
  // Estado para historial
  const [history, setHistory] = useState([initialValue]);
  
  // Estado para estadísticas
  const [stats, setStats] = useState({
    increments: 0,
    decrements: 0,
    resets: 0,
    maxReached: 0,
    minReached: 0,
  });

  // Función para incrementar
  const increment = useCallback(() => {
    setCount(prevCount => {
      const newCount = prevCount + step;
      
      if (newCount <= max) {
        // Actualizar historial
        setHistory(prev => [...prev, newCount]);
        
        // Actualizar estadísticas
        setStats(prev => ({
          ...prev,
          increments: prev.increments + 1,
          maxReached: newCount === max ? prev.maxReached + 1 : prev.maxReached,
        }));
        
        return newCount;
      }
      
      // Si se alcanza el máximo y autoReset está habilitado
      if (autoReset && newCount > max) {
        setHistory(prev => [...prev, resetValue]);
        setStats(prev => ({
          ...prev,
          increments: prev.increments + 1,
          maxReached: prev.maxReached + 1,
          resets: prev.resets + 1,
        }));
        return resetValue;
      }
      
      return prevCount;
    });
  }, [step, max, autoReset, resetValue]);

  // Función para decrementar
  const decrement = useCallback(() => {
    setCount(prevCount => {
      const newCount = prevCount - step;
      
      if (newCount >= min && (allowNegative || newCount >= 0)) {
        setHistory(prev => [...prev, newCount]);
        
        setStats(prev => ({
          ...prev,
          decrements: prev.decrements + 1,
          minReached: newCount === min ? prev.minReached + 1 : prev.minReached,
        }));
        
        return newCount;
      }
      
      if (autoReset && newCount < min) {
        setHistory(prev => [...prev, resetValue]);
        setStats(prev => ({
          ...prev,
          decrements: prev.decrements + 1,
          minReached: prev.minReached + 1,
          resets: prev.resets + 1,
        }));
        return resetValue;
      }
      
      return prevCount;
    });
  }, [step, min, allowNegative, autoReset, resetValue]);

  // Función para resetear
  const reset = useCallback(() => {
    setCount(resetValue);
    setHistory(prev => [...prev, resetValue]);
    setStats(prev => ({
      ...prev,
      resets: prev.resets + 1,
    }));
  }, [resetValue]);

  // Función para establecer valor específico
  const setValue = useCallback((value) => {
    const clampedValue = Math.max(min, Math.min(max, value));
    setCount(clampedValue);
    setHistory(prev => [...prev, clampedValue]);
  }, [min, max]);

  // Función para obtener estadísticas
  const getStats = useCallback(() => ({
    current: count,
    history: history,
    totalChanges: history.length - 1,
    average: history.reduce((sum, val) => sum + val, 0) / history.length,
    minValue: Math.min(...history),
    maxValue: Math.max(...history),
    ...stats,
  }), [count, history, stats]);

  // Función para limpiar historial
  const clearHistory = useCallback(() => {
    setHistory([count]);
  }, [count]);

  return {
    // Estado
    count,
    
    // Funciones
    increment,
    decrement,
    reset,
    setValue,
    clearHistory,
    
    // Utilidades
    isAtMax: count === max,
    isAtMin: count === min,
    canIncrement: count < max,
    canDecrement: count > min && (allowNegative || count > 0),
    
    // Estadísticas
    getStats,
  };
};

export default useCounter;
```

---

## 🧪 Casos de Uso

### **Caso 1: Contador Simple**
```javascript
const { count, increment, decrement } = useCounter(0);
```

### **Caso 2: Contador con Límites**
```javascript
const { count, increment, decrement } = useCounter(0, {
  min: 0,
  max: 100,
  step: 5,
  autoReset: true
});
```

### **Caso 3: Estado de Usuario**
```javascript
const [user, setUser] = useState({
  name: '',
  email: '',
  preferences: {}
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Contador con Historial**
Crea un contador que mantenga un historial de los últimos 10 valores.

### **Ejercicio 2: Formulario de Usuario**
Implementa un formulario que gestione estado de usuario con validación básica.

### **Ejercicio 3: Lista de Compras**
Crea una lista de compras donde puedas agregar, eliminar y marcar items como comprados.

---

## 🚀 Proyecto de la Clase

### **App de Gestión de Estado**

Crea una aplicación que demuestre:
- **Estado simple**: Contadores y toggles
- **Estado complejo**: Objetos y arrays
- **Hooks personalizados**: Lógica reutilizable
- **Patrones de estado**: Mejores prácticas

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **useState**: Hook fundamental para estado local
- **Inmutabilidad**: Nunca modificar estado directamente
- **Patrones de estado**: Objetos, arrays y valores primitivos
- **Hooks personalizados**: Lógica reutilizable

### **Habilidades Desarrolladas:**
- ✅ Usar useState para estado simple y complejo
- ✅ Implementar patrones de estado efectivos
- ✅ Crear hooks personalizados
- ✅ Gestionar estado de formularios

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **useEffect y Ciclo de Vida**, que te permitirá manejar efectos secundarios en tus componentes.

---

## 🔗 Enlaces de Navegación

- **⬅️ Módulo Anterior**: [Módulo 2: Componentes Básicos](../junior_2/README.md)
- **➡️ Siguiente Clase**: [useEffect y Ciclo de Vida](clase_2_useEffect_ciclo_vida.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
