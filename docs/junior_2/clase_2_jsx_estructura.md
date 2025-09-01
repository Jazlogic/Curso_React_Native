# 📚 Clase 2: JSX y Estructura

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 1: Componentes Nativos Básicos](clase_1_componentes_nativos_basicos.md)
- **➡️ Siguiente**: [Clase 3: Flexbox y Layout](clase_3_flexbox_layout.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender qué es JSX y cómo funciona en React Native
- Aprender a estructurar componentes de manera clara y legible
- Dominar la sintaxis JSX y sus reglas fundamentales
- Crear componentes con estructura lógica y organizada
- Entender la diferencia entre JSX y HTML tradicional

---

## 📚 Contenido Teórico

### **¿Qué es JSX?**

JSX (JavaScript XML) es una extensión de sintaxis para JavaScript que permite escribir código que se parece al HTML, pero con toda la potencia de JavaScript. En React Native, JSX se utiliza para describir cómo debe verse la interfaz de usuario.

#### **Características principales:**
- **Sintaxis familiar**: Similar al HTML pero con reglas específicas
- **Expresiones JavaScript**: Puedes usar variables, funciones y lógica directamente
- **Componentes**: Permite crear y reutilizar componentes personalizados
- **Transpilación**: Se convierte a JavaScript puro antes de ejecutarse

### **Reglas fundamentales de JSX:**

1. **Un solo elemento raíz**: Todo componente debe retornar un solo elemento
2. **Etiquetas de cierre**: Todas las etiquetas deben cerrarse correctamente
3. **Atributos en camelCase**: `className` en lugar de `class`, `onClick` en lugar de `onclick`
4. **Expresiones JavaScript**: Usar `{}` para insertar código JavaScript
5. **Comentarios**: Usar `{/* */}` para comentarios

### **Estructura de componentes:**

La estructura de un componente React Native debe ser:
- **Imports**: Librerías y componentes necesarios
- **Definición del componente**: Función o clase
- **Lógica del componente**: Hooks, variables, funciones
- **JSX**: La interfaz de usuario
- **Export**: Hacer el componente disponible

---

## 💻 Implementación Práctica

### **1. Componente Básico con JSX**

```javascript:src/components/BasicJSXExample.js
// Importamos los componentes necesarios de React Native
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

// Definimos nuestro componente como una función
const BasicJSXExample = () => {
  // Variables que usaremos en nuestro JSX
  const userName = 'Juan Pérez';
  const userAge = 25;
  const isActive = true;
  
  // Función que retorna un saludo personalizado
  const getGreeting = (name) => {
    return `¡Hola ${name}!`;
  };
  
  // Función que determina el estado del usuario
  const getUserStatus = (active) => {
    return active ? 'Activo' : 'Inactivo';
  };
  
  // Retornamos el JSX que describe la interfaz
  return (
    // View es el contenedor principal (equivalente a div en HTML)
    <View style={styles.container}>
      {/* Usamos {} para insertar expresiones JavaScript */}
      <Text style={styles.title}>
        {getGreeting(userName)}
      </Text>
      
      {/* Podemos usar operadores ternarios para renderizado condicional */}
      <Text style={styles.info}>
        Edad: {userAge} años
      </Text>
      
      {/* Renderizado condicional con operador lógico && */}
      {isActive && (
        <Text style={styles.status}>
          Estado: {getUserStatus(isActive)}
        </Text>
      )}
      
      {/* Lista de características usando map */}
      {['React Native', 'JSX', 'Componentes'].map((feature, index) => (
        <Text key={index} style={styles.feature}>
          ✓ {feature}
        </Text>
      ))}
    </View>
  );
};

// Estilos del componente
const styles = StyleSheet.create({
  container: {
    padding: 20,
    backgroundColor: '#f5f5f5',
    borderRadius: 10,
    margin: 10,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  info: {
    fontSize: 16,
    color: '#666',
    marginBottom: 5,
  },
  status: {
    fontSize: 16,
    color: '#28a745',
    marginBottom: 10,
    fontWeight: 'bold',
  },
  feature: {
    fontSize: 14,
    color: '#555',
    marginBottom: 3,
  },
});

// Exportamos el componente para poder usarlo en otros archivos
export default BasicJSXExample;
```

### **2. Componente con Estructura Compleja**

```javascript:src/components/ComplexJSXStructure.js
import React, { useState } from 'react';
import { 
  View, 
  Text, 
  TouchableOpacity, 
  ScrollView, 
  StyleSheet 
} from 'react-native';

const ComplexJSXStructure = () => {
  // Estado para manejar la lista de tareas
  const [tasks, setTasks] = useState([
    { id: 1, title: 'Aprender JSX', completed: true },
    { id: 2, title: 'Crear componentes', completed: false },
    { id: 3, title: 'Practicar estructura', completed: false },
  ]);
  
  // Estado para el filtro actual
  const [filter, setFilter] = useState('all'); // 'all', 'completed', 'pending'
  
  // Función para cambiar el estado de una tarea
  const toggleTask = (taskId) => {
    setTasks(prevTasks => 
      prevTasks.map(task => 
        task.id === taskId 
          ? { ...task, completed: !task.completed }
          : task
      )
    );
  };
  
  // Función para filtrar tareas
  const getFilteredTasks = () => {
    switch (filter) {
      case 'completed':
        return tasks.filter(task => task.completed);
      case 'pending':
        return tasks.filter(task => !task.completed);
      default:
        return tasks;
    }
  };
  
  // Función para obtener estadísticas
  const getStats = () => {
    const total = tasks.length;
    const completed = tasks.filter(task => task.completed).length;
    const pending = total - completed;
    
    return { total, completed, pending };
  };
  
  // Obtenemos las estadísticas
  const stats = getStats();
  
  // Obtenemos las tareas filtradas
  const filteredTasks = getFilteredTasks();
  
  return (
    <View style={styles.container}>
      {/* Header del componente */}
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Gestor de Tareas</Text>
        <Text style={styles.headerSubtitle}>
          Ejemplo de estructura JSX compleja
        </Text>
      </View>
      
      {/* Panel de estadísticas */}
      <View style={styles.statsContainer}>
        <View style={styles.statItem}>
          <Text style={styles.statNumber}>{stats.total}</Text>
          <Text style={styles.statLabel}>Total</Text>
        </View>
        <View style={styles.statItem}>
          <Text style={styles.statNumber}>{stats.completed}</Text>
          <Text style={styles.statLabel}>Completadas</Text>
        </View>
        <View style={styles.statItem}>
          <Text style={styles.statNumber}>{stats.pending}</Text>
          <Text style={styles.statLabel}>Pendientes</Text>
        </View>
      </View>
      
      {/* Filtros */}
      <View style={styles.filterContainer}>
        {['all', 'completed', 'pending'].map((filterOption) => (
          <TouchableOpacity
            key={filterOption}
            style={[
              styles.filterButton,
              filter === filterOption && styles.filterButtonActive
            ]}
            onPress={() => setFilter(filterOption)}
          >
            <Text style={[
              styles.filterButtonText,
              filter === filterOption && styles.filterButtonTextActive
            ]}>
              {filterOption === 'all' ? 'Todas' : 
               filterOption === 'completed' ? 'Completadas' : 'Pendientes'}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
      
      {/* Lista de tareas */}
      <ScrollView style={styles.tasksContainer}>
        {filteredTasks.length > 0 ? (
          filteredTasks.map((task) => (
            <TouchableOpacity
              key={task.id}
              style={[
                styles.taskItem,
                task.completed && styles.taskItemCompleted
              ]}
              onPress={() => toggleTask(task.id)}
            >
              <View style={styles.taskContent}>
                <Text style={[
                  styles.taskTitle,
                  task.completed && styles.taskTitleCompleted
                ]}>
                  {task.title}
                </Text>
                <Text style={styles.taskStatus}>
                  {task.completed ? '✓ Completada' : '○ Pendiente'}
                </Text>
              </View>
            </TouchableOpacity>
          ))
        ) : (
          <View style={styles.emptyState}>
            <Text style={styles.emptyStateText}>
              No hay tareas {filter === 'all' ? '' : 
                           filter === 'completed' ? 'completadas' : 'pendientes'}
            </Text>
          </View>
        )}
      </ScrollView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f8f9fa',
  },
  header: {
    backgroundColor: '#007bff',
    padding: 20,
    alignItems: 'center',
  },
  headerTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 5,
  },
  headerSubtitle: {
    fontSize: 16,
    color: '#e3f2fd',
  },
  statsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    padding: 20,
    backgroundColor: 'white',
    margin: 10,
    borderRadius: 10,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  statItem: {
    alignItems: 'center',
  },
  statNumber: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#007bff',
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
    marginTop: 5,
  },
  filterContainer: {
    flexDirection: 'row',
    paddingHorizontal: 10,
    marginBottom: 10,
  },
  filterButton: {
    flex: 1,
    paddingVertical: 10,
    paddingHorizontal: 15,
    marginHorizontal: 5,
    borderRadius: 20,
    backgroundColor: '#e9ecef',
    alignItems: 'center',
  },
  filterButtonActive: {
    backgroundColor: '#007bff',
  },
  filterButtonText: {
    color: '#666',
    fontWeight: '500',
  },
  filterButtonTextActive: {
    color: 'white',
  },
  tasksContainer: {
    flex: 1,
    paddingHorizontal: 10,
  },
  taskItem: {
    backgroundColor: 'white',
    padding: 15,
    marginBottom: 8,
    borderRadius: 8,
    elevation: 1,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
  },
  taskItemCompleted: {
    backgroundColor: '#f8f9fa',
    opacity: 0.7,
  },
  taskContent: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  taskTitle: {
    fontSize: 16,
    color: '#333',
    flex: 1,
  },
  taskTitleCompleted: {
    textDecorationLine: 'line-through',
    color: '#666',
  },
  taskStatus: {
    fontSize: 12,
    color: '#666',
    marginLeft: 10,
  },
  emptyState: {
    padding: 40,
    alignItems: 'center',
  },
  emptyStateText: {
    fontSize: 16,
    color: '#666',
    fontStyle: 'italic',
  },
});

export default ComplexJSXStructure;
```

### **3. Hook Personalizado para Estructura**

```javascript:src/hooks/useComponentStructure.js
import { useState, useCallback } from 'react';

// Hook personalizado para manejar la estructura de componentes
const useComponentStructure = (initialStructure = {}) => {
  // Estado para la estructura del componente
  const [structure, setStructure] = useState(initialStructure);
  
  // Estado para el historial de cambios
  const [history, setHistory] = useState([]);
  
  // Función para actualizar la estructura
  const updateStructure = useCallback((updates) => {
    setStructure(prevStructure => {
      // Guardamos el estado anterior en el historial
      setHistory(prevHistory => [...prevHistory, prevStructure]);
      
      // Aplicamos las actualizaciones
      return { ...prevStructure, ...updates };
    });
  }, []);
  
  // Función para revertir al estado anterior
  const undo = useCallback(() => {
    if (history.length > 0) {
      const previousState = history[history.length - 1];
      const newHistory = history.slice(0, -1);
      
      setStructure(previousState);
      setHistory(newHistory);
    }
  }, [history]);
  
  // Función para resetear la estructura
  const reset = useCallback(() => {
    setHistory(prevHistory => [...prevHistory, structure]);
    setStructure(initialStructure);
  }, [structure, initialStructure]);
  
  // Función para validar la estructura
  const validateStructure = useCallback(() => {
    const errors = [];
    
    // Verificamos que tenga un título
    if (!structure.title) {
      errors.push('El componente debe tener un título');
    }
    
    // Verificamos que tenga contenido
    if (!structure.content || structure.content.length === 0) {
      errors.push('El componente debe tener contenido');
    }
    
    // Verificamos que tenga estilos
    if (!structure.styles) {
      errors.push('El componente debe tener estilos definidos');
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }, [structure]);
  
  // Función para generar código JSX
  const generateJSX = useCallback(() => {
    const validation = validateStructure();
    
    if (!validation.isValid) {
      throw new Error(`Estructura inválida: ${validation.errors.join(', ')}`);
    }
    
    // Generamos el código JSX basado en la estructura
    return `
import React from 'react';
import { ${structure.imports?.join(', ') || 'View, Text'} } from 'react-native';

const ${structure.title} = () => {
  return (
    <View style={styles.container}>
      ${structure.content?.map(item => 
        `<${item.type} style={styles.${item.styleName}}>${item.text}</${item.type}>`
      ).join('\n      ') || ''}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    ${Object.entries(structure.styles?.container || {}).map(([key, value]) => 
      `${key}: ${typeof value === 'string' ? `'${value}'` : value},`
    ).join('\n    ') || ''}
  },
  ${Object.entries(structure.styles || {}).filter(([key]) => key !== 'container').map(([key, value]) => 
    `${key}: {
    ${Object.entries(value).map(([styleKey, styleValue]) => 
      `${styleKey}: ${typeof styleValue === 'string' ? `'${styleValue}'` : styleValue},`
    ).join('\n    ')}
  },`
  ).join('\n  ')}
});

export default ${structure.title};
    `.trim();
  }, [structure, validateStructure]);
  
  return {
    structure,
    updateStructure,
    undo,
    reset,
    validateStructure,
    generateJSX,
    canUndo: history.length > 0,
  };
};

export default useComponentStructure;
```

### **4. Utilidad para Validar JSX**

```javascript:src/utils/jsxValidator.js
// Utilidad para validar la sintaxis JSX
export const validateJSX = (jsxString) => {
  const errors = [];
  const warnings = [];
  
  // Verificamos que tenga un solo elemento raíz
  const rootElements = (jsxString.match(/<[A-Z][a-zA-Z]*/g) || []);
  if (rootElements.length === 0) {
    errors.push('El JSX debe tener al menos un elemento raíz');
  }
  
  // Verificamos etiquetas de cierre
  const openTags = (jsxString.match(/<([A-Z][a-zA-Z]*)/g) || []);
  const closeTags = (jsxString.match(/<\/([A-Z][a-zA-Z]*)/g) || []);
  
  if (openTags.length !== closeTags.length) {
    errors.push('Hay etiquetas sin cerrar o etiquetas de cierre extra');
  }
  
  // Verificamos atributos en camelCase
  const attributes = (jsxString.match(/[a-z]+-[a-z]+/g) || []);
  attributes.forEach(attr => {
    warnings.push(`Considera usar camelCase para: ${attr}`);
  });
  
  // Verificamos expresiones JavaScript
  const expressions = (jsxString.match(/\{([^}]+)\}/g) || []);
  expressions.forEach(expr => {
    if (expr === '{}') {
      warnings.push('Expresión JavaScript vacía detectada');
    }
  });
  
  // Verificamos comentarios
  const comments = (jsxString.match(/\/\*[\s\S]*?\*\//g) || []);
  if (comments.length === 0) {
    warnings.push('Considera agregar comentarios para explicar la lógica compleja');
  }
  
  return {
    isValid: errors.length === 0,
    errors,
    warnings,
    score: Math.max(0, 100 - (errors.length * 20) - (warnings.length * 5))
  };
};

// Función para formatear JSX
export const formatJSX = (jsxString) => {
  let formatted = jsxString;
  
  // Agregamos indentación
  const lines = formatted.split('\n');
  let indentLevel = 0;
  
  const formattedLines = lines.map(line => {
    const trimmedLine = line.trim();
    
    if (trimmedLine.includes('</')) {
      indentLevel = Math.max(0, indentLevel - 1);
    }
    
    const formattedLine = '  '.repeat(indentLevel) + trimmedLine;
    
    if (trimmedLine.includes('<') && !trimmedLine.includes('</') && !trimmedLine.includes('/>')) {
      indentLevel += 1;
    }
    
    return formattedLine;
  });
  
  return formattedLines.join('\n');
};

// Función para extraer información del JSX
export const extractJSXInfo = (jsxString) => {
  const components = (jsxString.match(/<([A-Z][a-zA-Z]*)/g) || [])
    .map(tag => tag.slice(1))
    .filter((tag, index, arr) => arr.indexOf(tag) === index);
  
  const props = (jsxString.match(/[a-zA-Z]+=/g) || [])
    .map(prop => prop.slice(0, -1))
    .filter((prop, index, arr) => arr.indexOf(prop) === index);
  
  const expressions = (jsxString.match(/\{([^}]+)\}/g) || [])
    .map(expr => expr.slice(1, -1).trim());
  
  return {
    components,
    props,
    expressions,
    complexity: components.length + props.length + expressions.length
  };
};
```

---

## 🧪 Casos de Uso

### **Caso 1: Componente de Perfil de Usuario**
```javascript
// Estructura clara para un perfil de usuario
const UserProfile = ({ user }) => {
  return (
    <View style={styles.profileContainer}>
      {/* Información básica */}
      <View style={styles.basicInfo}>
        <Image source={{ uri: user.avatar }} style={styles.avatar} />
        <Text style={styles.name}>{user.name}</Text>
        <Text style={styles.email}>{user.email}</Text>
      </View>
      
      {/* Estadísticas */}
      <View style={styles.stats}>
        <View style={styles.statItem}>
          <Text style={styles.statNumber}>{user.posts}</Text>
          <Text style={styles.statLabel}>Publicaciones</Text>
        </View>
        <View style={styles.statItem}>
          <Text style={styles.statNumber}>{user.followers}</Text>
          <Text style={styles.statLabel}>Seguidores</Text>
        </View>
      </View>
      
      {/* Acciones */}
      <View style={styles.actions}>
        <TouchableOpacity style={styles.actionButton}>
          <Text style={styles.actionText}>Editar Perfil</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};
```

### **Caso 2: Lista de Elementos con Renderizado Condicional**
```javascript
const ItemList = ({ items, showCompleted = false }) => {
  const filteredItems = showCompleted 
    ? items 
    : items.filter(item => !item.completed);
  
  if (filteredItems.length === 0) {
    return (
      <View style={styles.emptyState}>
        <Text style={styles.emptyText}>
          {showCompleted ? 'No hay elementos completados' : 'No hay elementos pendientes'}
        </Text>
      </View>
    );
  }
  
  return (
    <ScrollView style={styles.listContainer}>
      {filteredItems.map((item, index) => (
        <View key={item.id || index} style={styles.listItem}>
          <Text style={[
            styles.itemText,
            item.completed && styles.completedText
          ]}>
            {item.title}
          </Text>
          {item.description && (
            <Text style={styles.itemDescription}>
              {item.description}
            </Text>
          )}
        </View>
      ))}
    </ScrollView>
  );
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear un Componente de Tarjeta**
Crea un componente `Card` que tenga:
- Título
- Descripción
- Imagen (opcional)
- Botón de acción
- Estilos personalizables

### **Ejercicio 2: Implementar Renderizado Condicional**
Crea un componente que muestre diferentes contenidos basado en:
- Estado de carga
- Estado de error
- Estado de éxito
- Estado vacío

### **Ejercicio 3: Estructura de Formulario**
Crea un formulario con:
- Campos de entrada
- Validación
- Botones de acción
- Mensajes de estado

---

## 🚀 Proyecto de la Clase

### **App de Galería de Fotos con JSX Estructurado**

Crea una aplicación que muestre una galería de fotos con:
- **Componente principal**: `PhotoGallery`
- **Componente de foto**: `PhotoCard`
- **Componente de filtros**: `GalleryFilters`
- **Componente de estadísticas**: `GalleryStats`

**Requisitos:**
1. Usar JSX bien estructurado y organizado
2. Implementar renderizado condicional
3. Usar expresiones JavaScript en JSX
4. Crear componentes reutilizables
5. Implementar navegación entre vistas

**Estructura sugerida:**
```
src/
├── components/
│   ├── PhotoGallery.js
│   ├── PhotoCard.js
│   ├── GalleryFilters.js
│   └── GalleryStats.js
├── hooks/
│   └── usePhotoGallery.js
├── utils/
│   └── photoUtils.js
└── screens/
    └── GalleryScreen.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Native JSX](https://reactnative.dev/docs/intro-react-native-components)
- [JSX en React](https://es.reactjs.org/docs/introducing-jsx.html)

### **Artículos Recomendados:**
- "Mejores prácticas para estructurar componentes React Native"
- "JSX vs HTML: Diferencias y similitudes"
- "Cómo organizar código JSX para mejor legibilidad"

### **Herramientas:**
- [Prettier](https://prettier.io/) - Formateador de código
- [ESLint](https://eslint.org/) - Linter para JavaScript/JSX
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **JSX**: Sintaxis que combina JavaScript con XML para crear interfaces
- **Estructura**: Organización lógica de componentes para mejor legibilidad
- **Expresiones**: Uso de `{}` para insertar código JavaScript en JSX
- **Renderizado condicional**: Mostrar contenido basado en condiciones
- **Componentes reutilizables**: Crear piezas de UI que se pueden reutilizar

### **Habilidades Desarrolladas:**
- ✅ Escribir JSX válido y bien estructurado
- ✅ Organizar componentes de manera lógica
- ✅ Usar expresiones JavaScript en JSX
- ✅ Implementar renderizado condicional
- ✅ Crear componentes reutilizables

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Flexbox y Layout**, que te permitirá crear interfaces más complejas y responsivas usando el sistema de layout de React Native.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Componentes Nativos Básicos](clase_1_componentes_nativos_basicos.md)
- **➡️ Siguiente Clase**: [Flexbox y Layout](clase_3_flexbox_layout.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
