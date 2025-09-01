# 📱 Clase 2: Optimización de Componentes

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Implementar** técnicas avanzadas de memoización con React.memo
2. **Optimizar** componentes usando useMemo y useCallback
3. **Aplicar** patrones de optimización para componentes pesados
4. **Implementar** lazy loading de componentes
5. **Crear** componentes optimizados para listas y grids

---

## 📚 Contenido Teórico

### **1. Memoización Avanzada con React.memo**

#### **A. React.memo Básico**
```javascript
// ❌ SIN OPTIMIZACIÓN: Se re-renderiza siempre
const ExpensiveComponent = ({ data, onPress }) => {
  console.log('ExpensiveComponent renderizado');
  return (
    <TouchableOpacity onPress={onPress}>
      <Text>{data.value}</Text>
    </TouchableOpacity>
  );
};

// ✅ CON OPTIMIZACIÓN: Solo se re-renderiza si props cambian
const OptimizedComponent = React.memo(({ data, onPress }) => {
  console.log('OptimizedComponent renderizado');
  return (
    <TouchableOpacity onPress={onPress}>
      <Text>{data.value}</Text>
    </TouchableOpacity>
  );
});
```

#### **B. React.memo con Comparación Personalizada**
```javascript
// Comparación personalizada para objetos complejos
const areEqual = (prevProps, nextProps) => {
  return (
    prevProps.data.id === nextProps.data.id &&
    prevProps.data.value === nextProps.data.value &&
    prevProps.onPress === nextProps.onPress
  );
};

const DeepOptimizedComponent = React.memo(({ data, onPress }) => {
  return (
    <TouchableOpacity onPress={onPress}>
      <Text>{data.value}</Text>
    </TouchableOpacity>
  );
}, areEqual);
```

### **2. Optimización con useMemo**

#### **A. Cálculos Costosos**
```javascript
// ❌ SIN OPTIMIZACIÓN: Se recalcula en cada render
const ExpensiveCalculation = ({ items }) => {
  const expensiveValue = items.reduce((acc, item) => {
    // Simular cálculo costoso
    return acc + Math.pow(item.value, 2) + Math.sqrt(item.value);
  }, 0);
  
  return <Text>Resultado: {expensiveValue}</Text>;
};

// ✅ CON OPTIMIZACIÓN: Solo se recalcula si items cambia
const OptimizedCalculation = ({ items }) => {
  const expensiveValue = useMemo(() => {
    return items.reduce((acc, item) => {
      return acc + Math.pow(item.value, 2) + Math.sqrt(item.value);
    }, 0);
  }, [items]);
  
  return <Text>Resultado: {expensiveValue}</Text>;
};
```

#### **B. Objetos y Arrays**
```javascript
// ❌ PROBLEMA: Se crea un nuevo objeto en cada render
const BadComponent = ({ user }) => {
  const userInfo = {
    name: user.name,
    email: user.email,
    avatar: user.avatar
  };
  
  return <UserCard user={userInfo} />;
};

// ✅ SOLUCIÓN: useMemo para objetos complejos
const GoodComponent = ({ user }) => {
  const userInfo = useMemo(() => ({
    name: user.name,
    email: user.email,
    avatar: user.avatar
  }), [user.name, user.email, user.avatar]);
  
  return <UserCard user={userInfo} />;
};
```

### **3. Optimización con useCallback**

#### **A. Funciones que se Pasan como Props**
```javascript
// ❌ PROBLEMA: Nueva función en cada render
const ParentComponent = ({ items }) => {
  const handleItemPress = (item) => {
    console.log('Item presionado:', item);
  };
  
  return (
    <FlatList
      data={items}
      renderItem={({ item }) => (
        <ItemComponent item={item} onPress={handleItemPress} />
      )}
    />
  );
};

// ✅ SOLUCIÓN: useCallback para estabilizar la función
const OptimizedParentComponent = ({ items }) => {
  const handleItemPress = useCallback((item) => {
    console.log('Item presionado:', item);
  }, []);
  
  return (
    <FlatList
      data={items}
      renderItem={({ item }) => (
        <ItemComponent item={item} onPress={handleItemPress} />
      )}
    />
  );
};
```

#### **B. Funciones con Dependencias**
```javascript
const UserProfile = ({ userId, onUpdate }) => {
  const [user, setUser] = useState(null);
  
  // ✅ Función que depende de userId
  const fetchUser = useCallback(async () => {
    try {
      const userData = await api.getUser(userId);
      setUser(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
    }
  }, [userId]);
  
  // ✅ Función que depende de user y onUpdate
  const handleSave = useCallback(() => {
    if (user) {
      onUpdate(user);
    }
  }, [user, onUpdate]);
  
  useEffect(() => {
    fetchUser();
  }, [fetchUser]);
  
  return (
    <View>
      {user && (
        <>
          <TextInput
            value={user.name}
            onChangeText={(name) => setUser({ ...user, name })}
          />
          <Button title="Guardar" onPress={handleSave} />
        </>
      )}
    </View>
  );
};
```

### **4. Patrones de Optimización para Componentes Pesados**

#### **A. Componente con Estado Interno**
```javascript
const HeavyComponent = React.memo(({ data, onPress }) => {
  const [internalState, setInternalState] = useState(0);
  const [isExpanded, setIsExpanded] = useState(false);
  
  // Memoizar cálculos basados en estado interno
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      processed: item.value * internalState
    }));
  }, [data, internalState]);
  
  // Memoizar handlers
  const handleToggle = useCallback(() => {
    setIsExpanded(prev => !prev);
  }, []);
  
  const handleInternalUpdate = useCallback(() => {
    setInternalState(prev => prev + 1);
  }, []);
  
  return (
    <View>
      <TouchableOpacity onPress={handleToggle}>
        <Text>{isExpanded ? 'Contraer' : 'Expandir'}</Text>
      </TouchableOpacity>
      
      {isExpanded && (
        <View>
          {processedData.map(item => (
            <TouchableOpacity
              key={item.id}
              onPress={() => onPress(item)}
            >
              <Text>{item.processed}</Text>
            </TouchableOpacity>
          ))}
          
          <Button title="Actualizar" onPress={handleInternalUpdate} />
        </View>
      )}
    </View>
  );
});
```

#### **B. Componente con Lista Virtualizada**
```javascript
const VirtualizedList = React.memo(({ items, onItemPress }) => {
  const [visibleRange, setVisibleRange] = useState({ start: 0, end: 10 });
  
  // Memoizar items visibles
  const visibleItems = useMemo(() => {
    return items.slice(visibleRange.start, visibleRange.end);
  }, [items, visibleRange]);
  
  // Memoizar renderItem
  const renderItem = useCallback(({ item, index }) => (
    <ListItem
      item={item}
      onPress={() => onItemPress(item)}
      index={index}
    />
  ), [onItemPress]);
  
  // Memoizar keyExtractor
  const keyExtractor = useCallback((item) => item.id.toString(), []);
  
  // Memoizar onViewableItemsChanged
  const onViewableItemsChanged = useCallback(({ viewableItems }) => {
    if (viewableItems.length > 0) {
      const first = viewableItems[0].index;
      const last = viewableItems[viewableItems.length - 1].index;
      setVisibleRange({ start: first, end: last + 1 });
    }
  }, []);
  
  return (
    <FlatList
      data={visibleItems}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      onViewableItemsChanged={onViewableItemsChanged}
      viewabilityConfig={{
        itemVisiblePercentThreshold: 50
      }}
      removeClippedSubviews={true}
      maxToRenderPerBatch={5}
      windowSize={5}
    />
  );
});
```

### **5. Lazy Loading de Componentes**

#### **A. Lazy Loading Básico**
```javascript
// Componente que se carga solo cuando es necesario
const LazyComponent = React.lazy(() => import('./LazyComponent'));

const MainComponent = () => {
  const [showLazy, setShowLazy] = useState(false);
  
  return (
    <View>
      <Button
        title="Cargar Componente"
        onPress={() => setShowLazy(true)}
      />
      
      {showLazy && (
        <Suspense fallback={<ActivityIndicator size="large" />}>
          <LazyComponent />
        </Suspense>
      )}
    </View>
  );
};
```

#### **B. Lazy Loading Condicional**
```javascript
const ConditionalLazyComponent = ({ type }) => {
  const LazyComponent = useMemo(() => {
    switch (type) {
      case 'chart':
        return React.lazy(() => import('./ChartComponent'));
      case 'map':
        return React.lazy(() => import('./MapComponent'));
      case 'table':
        return React.lazy(() => import('./TableComponent'));
      default:
        return null;
    }
  }, [type]);
  
  if (!LazyComponent) return null;
  
  return (
    <Suspense fallback={<ActivityIndicator size="large" />}>
      <LazyComponent />
    </Suspense>
  );
};
```

---

## 💻 Ejercicios Prácticos

### **Ejercicio 1: Optimizar Componente de Lista**
Crea un componente de lista que use todas las técnicas de optimización.

```javascript
const OptimizedList = React.memo(({ items, onItemPress, onItemLongPress }) => {
  // Implementa aquí todas las optimizaciones
  // - React.memo
  // - useMemo para items procesados
  // - useCallback para handlers
  // - FlatList optimizado
});
```

**Tarea**: Implementa el componente con todas las optimizaciones y mide el performance.

### **Ejercicio 2: Componente con Estado Complejo**
Crea un componente que maneje estado interno complejo de manera optimizada.

```javascript
const ComplexStateComponent = React.memo(({ initialData, onSave }) => {
  // Implementa estado interno optimizado
  // - Múltiples estados relacionados
  // - Cálculos memoizados
  // - Handlers optimizados
});
```

**Tarea**: Maneja al menos 3 estados internos con optimizaciones.

### **Ejercicio 3: Lazy Loading de Pantallas**
Implementa lazy loading para diferentes pantallas de una aplicación.

```javascript
const AppNavigator = () => {
  // Implementa lazy loading para:
  // - Pantalla de perfil
  // - Pantalla de configuración
  // - Pantalla de estadísticas
};
```

**Tarea**: Crea 3 pantallas lazy-loaded con Suspense.

---

## 🎯 Proyecto Integrador

### **Componente de Dashboard Optimizado**

Crea un componente de dashboard que demuestre todas las técnicas de optimización:

#### **Funcionalidades Requeridas:**
1. **Múltiples Widgets**: Cada uno optimizado individualmente
2. **Estado Compartido**: Entre widgets con optimizaciones
3. **Lazy Loading**: De widgets pesados
4. **Memoización**: De cálculos y funciones
5. **Performance Monitoring**: Métricas en tiempo real

#### **Requisitos Técnicos:**
- **React.memo**: Para todos los widgets
- **useMemo**: Para cálculos costosos
- **useCallback**: Para handlers y funciones
- **Lazy Loading**: Para widgets complejos
- **Estado Optimizado**: Sin re-renders innecesarios

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React.memo](https://react.dev/reference/react/memo)
- [useMemo](https://react.dev/reference/react/useMemo)
- [useCallback](https://react.dev/reference/react/useCallback)

### **Artículos Recomendados:**
- "Advanced React Performance Patterns"
- "Memoization Strategies in React"
- "Component Optimization Best Practices"

---

## 🎓 Próximos Pasos

### **En la Siguiente Clase Aprenderás:**
- **Optimización de Listas**: FlatList, VirtualizedList y optimizaciones
- **Optimización de Imágenes**: Lazy loading, caching y compresión
- **Optimización de Navegación**: Lazy loading de pantallas y transiciones

---

**🎯 Objetivo de la Clase**: Dominar las técnicas avanzadas de optimización de componentes usando React.memo, useMemo y useCallback.

**💡 Consejo**: La memoización es poderosa, pero úsala sabiamente. No todo necesita ser memoizado, solo los componentes y cálculos que realmente impactan el performance.
