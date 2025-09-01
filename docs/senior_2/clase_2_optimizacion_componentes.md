# ⚡ Clase 2: Optimización de Componentes

## 📋 Objetivos de la Clase
- Dominar React.memo para evitar re-renders innecesarios
- Aprender useMemo para memoización de valores costosos
- Implementar useCallback para callbacks estables
- Optimizar componentes con técnicas avanzadas de memoización
- Crear componentes de alto rendimiento

## ⏱️ Duración
**2 horas**

## 🔗 Navegación
- **Anterior**: [Clase 1: Fundamentos de Performance](clase_1_fundamentos_performance.md)
- **Siguiente**: [Clase 3: Optimización de Listas y Scroll](clase_3_optimizacion_listas_scroll.md)
- **Módulo**: [README.md](README.md)
- **Inicio**: [🏠](../../README.md)

---

## 🎯 React.memo - El Guardián de Re-renders

### ¿Qué es React.memo?
`React.memo` es una función de orden superior que memoiza un componente, evitando re-renders cuando las props no han cambiado.

### Implementación Básica
```javascript
// ❌ Sin memoización - Se re-renderiza siempre
const ExpensiveComponent = ({ data, onUpdate }) => {
  console.log('🔄 Re-renderizando ExpensiveComponent');
  
  // Simular operación costosa
  const processedData = data.map(item => ({
    ...item,
    processed: true,
    timestamp: Date.now()
  }));
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Datos Procesados</Text>
      {processedData.map(item => (
        <Text key={item.id} style={styles.item}>
          {item.name} - {item.processed ? '✅' : '❌'}
        </Text>
      ))}
      <TouchableOpacity style={styles.button} onPress={onUpdate}>
        <Text style={styles.buttonText}>Actualizar</Text>
      </TouchableOpacity>
    </View>
  );
};

// ✅ Con memoización - Solo se re-renderiza si las props cambian
const OptimizedComponent = React.memo(({ data, onUpdate }) => {
  console.log('🚀 Renderizando OptimizedComponent (memoizado)');
  
  const processedData = data.map(item => ({
    ...item,
    processed: true,
    timestamp: Date.now()
  }));
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Datos Procesados (Optimizado)</Text>
      {processedData.map(item => (
        <Text key={item.id} style={styles.item}>
          {item.name} - {item.processed ? '✅' : '❌'}
        </Text>
      ))}
      <TouchableOpacity style={styles.button} onPress={onUpdate}>
        <Text style={styles.buttonText}>Actualizar</Text>
      </TouchableOpacity>
    </View>
  );
});

// Función de comparación personalizada para React.memo
const arePropsEqual = (prevProps, nextProps) => {
  // Comparar solo las propiedades que realmente importan
  const dataChanged = prevProps.data.length !== nextProps.data.length ||
    prevProps.data.some((item, index) => item.id !== nextProps.data[index]?.id);
  
  // onUpdate es una función, comparar por referencia
  const callbackChanged = prevProps.onUpdate !== nextProps.onUpdate;
  
  console.log('🔍 Comparando props:', {
    dataChanged,
    callbackChanged,
    shouldUpdate: dataChanged || callbackChanged
  });
  
  return !dataChanged && !callbackChanged;
};

// Componente con comparación personalizada
const SmartComponent = React.memo(({ data, onUpdate }) => {
  console.log('🧠 Renderizando SmartComponent (comparación personalizada)');
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Componente Inteligente</Text>
      <Text style={styles.subtitle}>
        Total de items: {data.length}
      </Text>
      <TouchableOpacity style={styles.button} onPress={onUpdate}>
        <Text style={styles.buttonText}>Actualizar</Text>
      </TouchableOpacity>
    </View>
  );
}, arePropsEqual);
```

### Uso en el Componente Padre
```javascript
const ParentComponent = () => {
  const [data, setData] = useState([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ]);
  
  const [counter, setCounter] = useState(0);
  
  // ❌ Callback que cambia en cada render
  const handleUpdateBad = () => {
    setData(prev => [...prev, { id: Date.now(), name: `Item ${prev.length + 1}` }]);
  };
  
  // ✅ Callback estable con useCallback
  const handleUpdateGood = useCallback(() => {
    setData(prev => [...prev, { id: Date.now(), name: `Item ${prev.length + 1}` }]);
  }, []);
  
  // ✅ Callback que solo cambia cuando data cambia
  const handleUpdateSmart = useCallback(() => {
    setData(prev => [...prev, { id: Date.now(), name: `Item ${prev.length + 1}` }]);
  }, [data.length]);
  
  return (
    <View style={styles.container}>
      <Text style={styles.counter}>Contador: {counter}</Text>
      <TouchableOpacity 
        style={styles.button} 
        onPress={() => setCounter(c => c + 1)}
      >
        <Text style={styles.buttonText}>Incrementar Contador</Text>
      </TouchableOpacity>
      
      {/* Este componente se re-renderiza cada vez que counter cambia */}
      <ExpensiveComponent data={data} onUpdate={handleUpdateBad} />
      
      {/* Este componente solo se re-renderiza cuando data o onUpdate cambian */}
      <OptimizedComponent data={data} onUpdate={handleUpdateGood} />
      
      {/* Este componente tiene comparación personalizada */}
      <SmartComponent data={data} onUpdate={handleUpdateSmart} />
    </View>
  );
};
```

---

## 🧠 useMemo - Memoización de Valores

### ¿Cuándo Usar useMemo?
- **Cálculos costosos** que se repiten en cada render
- **Objetos/arrays complejos** que se crean en cada render
- **Operaciones de transformación de datos** pesadas

### Ejemplos Prácticos
```javascript
// ❌ Sin useMemo - Se recalcula en cada render
const DataProcessor = ({ items, filterText, sortBy }) => {
  // Este procesamiento se ejecuta en cada render
  const processedItems = items
    .filter(item => item.name.toLowerCase().includes(filterText.toLowerCase()))
    .sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'date') return new Date(b.date) - new Date(a.date);
      return 0;
    })
    .map(item => ({
      ...item,
      processed: true,
      timestamp: Date.now(),
      // Simular operación costosa
      score: Math.sqrt(item.id) * Math.PI + Math.random()
    }));
  
  console.log('🔄 Procesando datos sin memoización');
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Items Procesados: {processedItems.length}</Text>
      <FlatList
        data={processedItems}
        keyExtractor={item => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text style={styles.itemName}>{item.name}</Text>
            <Text style={styles.itemScore}>Score: {item.score.toFixed(2)}</Text>
          </View>
        )}
      />
    </View>
  );
};

// ✅ Con useMemo - Solo se recalcula cuando las dependencias cambian
const OptimizedDataProcessor = ({ items, filterText, sortBy }) => {
  // Memoizar el procesamiento de datos
  const processedItems = useMemo(() => {
    console.log('🧠 Procesando datos con memoización');
    
    return items
      .filter(item => item.name.toLowerCase().includes(filterText.toLowerCase()))
      .sort((a, b) => {
        if (sortBy === 'name') return a.name.localeCompare(b.name);
        if (sortBy === 'date') return new Date(b.date) - new Date(a.date);
        return 0;
      })
      .map(item => ({
        ...item,
        processed: true,
        timestamp: Date.now(),
        score: Math.sqrt(item.id) * Math.PI + Math.random()
      }));
  }, [items, filterText, sortBy]); // Solo recalcular si estas dependencias cambian
  
  // Memoizar estadísticas calculadas
  const statistics = useMemo(() => {
    if (processedItems.length === 0) return null;
    
    const totalScore = processedItems.reduce((sum, item) => sum + item.score, 0);
    const averageScore = totalScore / processedItems.length;
    const maxScore = Math.max(...processedItems.map(item => item.score));
    const minScore = Math.min(...processedItems.map(item => item.score));
    
    return {
      total: processedItems.length,
      averageScore: averageScore.toFixed(2),
      maxScore: maxScore.toFixed(2),
      minScore: minScore.toFixed(2)
    };
  }, [processedItems]);
  
  // Memoizar estilos que dependen de datos
  const containerStyle = useMemo(() => [
    styles.container,
    {
      backgroundColor: processedItems.length > 10 ? '#e8f5e8' : '#fff3e0'
    }
  ], [processedItems.length]);
  
  return (
    <View style={containerStyle}>
      <Text style={styles.title}>Items Procesados: {processedItems.length}</Text>
      
      {statistics && (
        <View style={styles.stats}>
          <Text style={styles.statsTitle}>Estadísticas:</Text>
          <Text>Promedio: {statistics.averageScore}</Text>
          <Text>Máximo: {statistics.maxScore}</Text>
          <Text>Mínimo: {statistics.minScore}</Text>
        </View>
      )}
      
      <FlatList
        data={processedItems}
        keyExtractor={item => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text style={styles.itemName}>{item.name}</Text>
            <Text style={styles.itemScore}>Score: {item.score.toFixed(2)}</Text>
          </View>
        )}
      />
    </View>
  );
};
```

### Memoización de Objetos y Arrays
```javascript
// ❌ Sin memoización - Nuevo objeto en cada render
const BadForm = ({ onSubmit, initialData }) => {
  const [formData, setFormData] = useState(initialData);
  
  // Este objeto se recrea en cada render
  const formConfig = {
    fields: [
      { name: 'name', label: 'Nombre', required: true },
      { name: 'email', label: 'Email', required: true },
      { name: 'phone', label: 'Teléfono', required: false }
    ],
    validation: {
      email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      phone: /^\+?[\d\s-]+$/
    }
  };
  
  return (
    <View style={styles.form}>
      {formConfig.fields.map(field => (
        <View key={field.name} style={styles.field}>
          <Text style={styles.label}>{field.label}</Text>
          <TextInput
            style={styles.input}
            value={formData[field.name] || ''}
            onChangeText={(text) => setFormData(prev => ({
              ...prev,
              [field.name]: text
            }))}
            placeholder={`Ingresa tu ${field.label.toLowerCase()}`}
          />
        </View>
      ))}
      <TouchableOpacity style={styles.button} onPress={() => onSubmit(formData)}>
        <Text style={styles.buttonText}>Enviar</Text>
      </TouchableOpacity>
    </View>
  );
};

// ✅ Con memoización - Objetos estables
const OptimizedForm = ({ onSubmit, initialData }) => {
  const [formData, setFormData] = useState(initialData);
  
  // Memoizar configuración del formulario
  const formConfig = useMemo(() => ({
    fields: [
      { name: 'name', label: 'Nombre', required: true },
      { name: 'email', label: 'Email', required: true },
      { name: 'phone', label: 'Teléfono', required: false }
    ],
    validation: {
      email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      phone: /^\+?[\d\s-]+$/
    }
  }), []); // Array vacío = nunca cambia
  
  // Memoizar función de validación
  const validateForm = useMemo(() => {
    return (data) => {
      const errors = {};
      
      formConfig.fields.forEach(field => {
        if (field.required && !data[field.name]) {
          errors[field.name] = `${field.label} es requerido`;
        }
        
        if (data[field.name] && formConfig.validation[field.name]) {
          const regex = formConfig.validation[field.name];
          if (!regex.test(data[field.name])) {
            errors[field.name] = `${field.name} no es válido`;
          }
        }
      });
      
      return errors;
    };
  }, [formConfig]);
  
  // Memoizar estilos que dependen del estado
  const buttonStyle = useMemo(() => [
    styles.button,
    {
      opacity: Object.keys(formData).length > 0 ? 1 : 0.5
    }
  ], [formData]);
  
  const handleSubmit = () => {
    const errors = validateForm(formData);
    if (Object.keys(errors).length === 0) {
      onSubmit(formData);
    } else {
      console.log('❌ Errores de validación:', errors);
    }
  };
  
  return (
    <View style={styles.form}>
      {formConfig.fields.map(field => (
        <View key={field.name} style={styles.field}>
          <Text style={styles.label}>{field.label}</Text>
          <TextInput
            style={styles.input}
            value={formData[field.name] || ''}
            onChangeText={(text) => setFormData(prev => ({
              ...prev,
              [field.name]: text
            }))}
            placeholder={`Ingresa tu ${field.label.toLowerCase()}`}
          />
        </View>
      ))}
      <TouchableOpacity style={buttonStyle} onPress={handleSubmit}>
        <Text style={styles.buttonText}>Enviar</Text>
      </TouchableOpacity>
    </View>
  );
};
```

---

## 🎣 useCallback - Callbacks Estables

### ¿Por qué useCallback?
- **Evita re-renders** de componentes hijos memoizados
- **Mantiene referencias estables** de funciones
- **Optimiza componentes** que dependen de callbacks

### Implementación y Uso
```javascript
// ❌ Sin useCallback - Nueva función en cada render
const BadParent = ({ items }) => {
  const [selectedId, setSelectedId] = useState(null);
  const [filter, setFilter] = useState('');
  
  // Estas funciones se recrean en cada render
  const handleItemSelect = (id) => {
    setSelectedId(id);
  };
  
  const handleItemDelete = (id) => {
    // Simular eliminación
    console.log('Eliminando item:', id);
  };
  
  const handleFilterChange = (text) => {
    setFilter(text);
  };
  
  const filteredItems = items.filter(item => 
    item.name.toLowerCase().includes(filter.toLowerCase())
  );
  
  return (
    <View style={styles.container}>
      <TextInput
        style={styles.searchInput}
        placeholder="Filtrar items..."
        value={filter}
        onChangeText={handleFilterChange}
      />
      
      {filteredItems.map(item => (
        <ItemRow
          key={item.id}
          item={item}
          isSelected={selectedId === item.id}
          onSelect={handleItemSelect}
          onDelete={handleItemDelete}
        />
      ))}
    </View>
  );
};

// ✅ Con useCallback - Funciones estables
const OptimizedParent = ({ items }) => {
  const [selectedId, setSelectedId] = useState(null);
  const [filter, setFilter] = useState('');
  
  // Callbacks estables con useCallback
  const handleItemSelect = useCallback((id) => {
    setSelectedId(id);
  }, []); // Sin dependencias = función estable
  
  const handleItemDelete = useCallback((id) => {
    console.log('Eliminando item:', id);
  }, []); // Sin dependencias = función estable
  
  const handleFilterChange = useCallback((text) => {
    setFilter(text);
  }, []); // Sin dependencias = función estable
  
  // Callback que depende de filter
  const handleClearFilter = useCallback(() => {
    setFilter('');
  }, [filter]); // Depende de filter
  
  // Callback que depende de items
  const handleBulkDelete = useCallback(() => {
    const selectedItems = items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
    console.log('Eliminando en lote:', selectedItems.length, 'items');
  }, [items, filter]); // Depende de items y filter
  
  const filteredItems = useMemo(() => 
    items.filter(item => item.name.toLowerCase().includes(filter.toLowerCase())),
    [items, filter]
  );
  
  return (
    <View style={styles.container}>
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Filtrar items..."
          value={filter}
          onChangeText={handleFilterChange}
        />
        {filter && (
          <TouchableOpacity style={styles.clearButton} onPress={handleClearFilter}>
            <Text style={styles.clearButtonText}>✕</Text>
          </TouchableOpacity>
        )}
      </View>
      
      {filteredItems.map(item => (
        <ItemRow
          key={item.id}
          item={item}
          isSelected={selectedId === item.id}
          onSelect={handleItemSelect}
          onDelete={handleItemDelete}
        />
      ))}
      
      {filteredItems.length > 0 && (
        <TouchableOpacity style={styles.bulkButton} onPress={handleBulkDelete}>
          <Text style={styles.bulkButtonText}>
            Eliminar {filteredItems.length} items
          </Text>
        </TouchableOpacity>
      )}
    </View>
  );
};

// Componente hijo optimizado
const ItemRow = React.memo(({ item, isSelected, onSelect, onDelete }) => {
  console.log(`🔄 Renderizando ItemRow: ${item.name}`);
  
  return (
    <View style={[styles.itemRow, isSelected && styles.selectedItem]}>
      <TouchableOpacity 
        style={styles.itemContent} 
        onPress={() => onSelect(item.id)}
      >
        <Text style={styles.itemName}>{item.name}</Text>
        <Text style={styles.itemDescription}>{item.description}</Text>
      </TouchableOpacity>
      
      <TouchableOpacity 
        style={styles.deleteButton} 
        onPress={() => onDelete(item.id)}
      >
        <Text style={styles.deleteButtonText}>🗑️</Text>
      </TouchableOpacity>
    </View>
  );
});
```

---

## 🔧 Técnicas Avanzadas de Optimización

### 1. **useMemo para Funciones Costosas**
```javascript
const ExpensiveCalculator = ({ numbers, operation }) => {
  // Memoizar función de cálculo
  const calculateFunction = useMemo(() => {
    switch (operation) {
      case 'sum':
        return (nums) => nums.reduce((sum, num) => sum + num, 0);
      case 'product':
        return (nums) => nums.reduce((prod, num) => prod * num, 1);
      case 'average':
        return (nums) => nums.reduce((sum, num) => sum + num, 0) / nums.length;
      case 'fibonacci':
        return (nums) => {
          const fib = (n) => n <= 1 ? n : fib(n - 1) + fib(n - 2);
          return nums.map(num => fib(num));
        };
      default:
        return (nums) => nums;
    }
  }, [operation]);
  
  // Memoizar resultado del cálculo
  const result = useMemo(() => {
    console.log(`🧮 Calculando ${operation} para ${numbers.length} números`);
    return calculateFunction(numbers);
  }, [calculateFunction, numbers]);
  
  return (
    <View style={styles.calculator}>
      <Text style={styles.title}>Calculadora: {operation}</Text>
      <Text style={styles.numbers}>Números: {numbers.join(', ')}</Text>
      <Text style={styles.result}>Resultado: {result}</Text>
    </View>
  );
};
```

### 2. **Optimización de Context**
```javascript
// ❌ Context sin optimización
const BadThemeContext = createContext();

const BadThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  // Este objeto se recrea en cada render
  const themeValue = {
    theme,
    setTheme,
    colors: theme === 'light' ? lightColors : darkColors,
    isDark: theme === 'dark'
  };
  
  return (
    <BadThemeContext.Provider value={themeValue}>
      {children}
    </BadThemeContext.Provider>
  );
};

// ✅ Context optimizado
const OptimizedThemeContext = createContext();

const OptimizedThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  // Memoizar valores del context
  const themeValue = useMemo(() => ({
    theme,
    setTheme,
    colors: theme === 'light' ? lightColors : darkColors,
    isDark: theme === 'dark'
  }), [theme]);
  
  // Memoizar función de cambio de tema
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  // Memoizar objeto completo del context
  const contextValue = useMemo(() => ({
    ...themeValue,
    toggleTheme
  }), [themeValue, toggleTheme]);
  
  return (
    <OptimizedThemeContext.Provider value={contextValue}>
      {children}
    </OptimizedThemeContext.Provider>
  );
};

// Hook personalizado optimizado
const useTheme = () => {
  const context = useContext(OptimizedThemeContext);
  if (!context) {
    throw new Error('useTheme debe usarse dentro de ThemeProvider');
  }
  return context;
};
```

### 3. **Optimización de Event Handlers**
```javascript
const OptimizedForm = ({ onSubmit, validationRules }) => {
  const [formData, setFormData] = useState({});
  const [errors, setErrors] = useState({});
  
  // Memoizar función de validación
  const validateField = useCallback((name, value) => {
    const rule = validationRules[name];
    if (!rule) return '';
    
    if (rule.required && !value) {
      return `${name} es requerido`;
    }
    
    if (rule.pattern && !rule.pattern.test(value)) {
      return rule.message || `${name} no es válido`;
    }
    
    return '';
  }, [validationRules]);
  
  // Memoizar función de cambio de campo
  const handleFieldChange = useCallback((name, value) => {
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Validar campo en tiempo real
    const error = validateField(name, value);
    setErrors(prev => ({ ...prev, [name]: error }));
  }, [validateField]);
  
  // Memoizar función de envío
  const handleSubmit = useCallback(() => {
    const newErrors = {};
    
    // Validar todos los campos
    Object.keys(validationRules).forEach(fieldName => {
      const error = validateField(fieldName, formData[fieldName]);
      if (error) {
        newErrors[fieldName] = error;
      }
    });
    
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      onSubmit(formData);
    }
  }, [formData, validationRules, validateField, onSubmit]);
  
  // Memoizar función de reset
  const handleReset = useCallback(() => {
    setFormData({});
    setErrors({});
  }, []);
  
  return (
    <View style={styles.form}>
      {Object.keys(validationRules).map(fieldName => (
        <View key={fieldName} style={styles.field}>
          <Text style={styles.label}>{fieldName}</Text>
          <TextInput
            style={[
              styles.input,
              errors[fieldName] && styles.inputError
            ]}
            value={formData[fieldName] || ''}
            onChangeText={(text) => handleFieldChange(fieldName, text)}
            placeholder={`Ingresa ${fieldName}`}
          />
          {errors[fieldName] && (
            <Text style={styles.errorText}>{errors[fieldName]}</Text>
          )}
        </View>
      ))}
      
      <View style={styles.buttons}>
        <TouchableOpacity style={styles.submitButton} onPress={handleSubmit}>
          <Text style={styles.buttonText}>Enviar</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.resetButton} onPress={handleReset}>
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};
```

---

## 📱 Ejercicios Prácticos

### Ejercicio 1: Componente de Lista Optimizada
Crea un componente de lista que use React.memo, useMemo y useCallback para optimizar el rendimiento.

```javascript
// Tu código aquí
const OptimizedList = ({ items, onItemSelect, onItemDelete, filterText }) => {
  // Implementa:
  // 1. React.memo para el componente
  // 2. useMemo para filtrar items
  // 3. useCallback para handlers
  // 4. Optimización de re-renders
};
```

### Ejercicio 2: Calculadora con Memoización
Implementa una calculadora que memoice operaciones costosas.

```javascript
// Tu código aquí
const MemoizedCalculator = ({ numbers, operation, precision }) => {
  // Implementa:
  // 1. useMemo para funciones de cálculo
  // 2. useMemo para resultados
  // 3. useCallback para operaciones
  // 4. Optimización de cálculos repetitivos
};
```

### Ejercicio 3: Formulario Dinámico Optimizado
Crea un formulario que se adapte dinámicamente y use todas las técnicas de optimización.

```javascript
// Tu código aquí
const DynamicForm = ({ fields, onSubmit, validation }) => {
  // Implementa:
  // 1. React.memo para campos del formulario
  // 2. useMemo para configuración
  // 3. useCallback para handlers
  // 4. Optimización de validación
};
```

---

## 🔍 Resumen de la Clase

### ✅ Lo que Aprendiste
- **React.memo** evita re-renders innecesarios de componentes
- **useMemo** es ideal para cálculos costosos y objetos que se recrean
- **useCallback** es esencial para funciones que se pasan como props
- **Técnicas avanzadas** para optimización de context y event handlers

### 🎯 Próximos Pasos
En la siguiente clase aprenderás:
- **Optimización de FlatList** y virtualización
- **Lazy loading** de imágenes y contenido
- **Técnicas de scroll** optimizado

### 💡 Consejos Clave
1. **Usa React.memo** para componentes que reciben las mismas props frecuentemente
2. **useMemo** es ideal para cálculos costosos y objetos que se recrean
3. **useCallback** es esencial para funciones que se pasan como props
4. **Mide el impacto** antes y después de optimizar
5. **No optimices prematuramente** - solo cuando sea necesario

---

## 📚 Recursos Adicionales
- [React.memo Documentation](https://reactjs.org/docs/react-api.html#reactmemo)
- [useMemo Hook](https://reactjs.org/docs/hooks-reference.html#usememo)
- [useCallback Hook](https://reactjs.org/docs/hooks-reference.html#usecallback)
- [React Performance Optimization](https://reactjs.org/docs/optimizing-performance.html)

---

**¿Tienes alguna pregunta sobre la optimización de componentes? ¿Te gustaría que profundice en algún aspecto específico antes de continuar con la siguiente clase?**
