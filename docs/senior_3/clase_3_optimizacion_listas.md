# 📱 Clase 3: Optimización de Listas y Scroll

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Implementar** FlatList con todas las optimizaciones disponibles
2. **Configurar** VirtualizedList para listas extremadamente largas
3. **Optimizar** el rendimiento de scroll y navegación
4. **Implementar** lazy loading y paginación eficiente
5. **Crear** listas personalizadas con máximo rendimiento

---

## 📚 Contenido Teórico

### **1. FlatList vs ScrollView: Cuándo Usar Cada Uno**

#### **A. ScrollView - Para Listas Pequeñas**
```javascript
// ✅ ScrollView: Ideal para listas pequeñas (< 50 items)
const SmallList = ({ items }) => (
  <ScrollView>
    {items.map((item) => (
      <ListItem key={item.id} item={item} />
    ))}
  </ScrollView>
);

// ❌ ScrollView: Problemas con listas grandes
const BadLargeList = ({ items }) => (
  <ScrollView>
    {items.map((item) => (
      <ListItem key={item.id} item={item} />
    ))}
  </ScrollView>
);
```

#### **B. FlatList - Para Listas de Cualquier Tamaño**
```javascript
// ✅ FlatList: Optimizado para cualquier cantidad de items
const OptimizedList = ({ items }) => (
  <FlatList
    data={items}
    renderItem={({ item }) => <ListItem item={item} />}
    keyExtractor={(item) => item.id.toString()}
    // Optimizaciones básicas
    removeClippedSubviews={true}
    maxToRenderPerBatch={10}
    windowSize={10}
    initialNumToRender={10}
  />
);
```

### **2. Configuración Avanzada de FlatList**

#### **A. Optimizaciones de Rendimiento**
```javascript
const HighPerformanceList = ({ items, onItemPress }) => {
  // Memoizar renderItem para evitar re-creaciones
  const renderItem = useCallback(({ item, index }) => (
    <ListItem
      item={item}
      index={index}
      onPress={() => onItemPress(item)}
    />
  ), [onItemPress]);

  // Memoizar keyExtractor
  const keyExtractor = useCallback((item) => item.id.toString(), []);

  // Memoizar getItemLayout para listas con altura fija
  const getItemLayout = useCallback((data, index) => ({
    length: 80, // altura fija de cada item
    offset: 80 * index,
    index,
  }), []);

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      
      // Optimizaciones de rendimiento
      removeClippedSubviews={true}
      maxToRenderPerBatch={5}
      windowSize={5}
      initialNumToRender={10}
      updateCellsBatchingPeriod={50}
      
      // Optimizaciones de scroll
      scrollEventThrottle={16}
      decelerationRate="fast"
      showsVerticalScrollIndicator={false}
      
      // Optimizaciones de memoria
      maintainVisibleContentPosition={{
        minIndexForVisible: 0,
        autoscrollToTopThreshold: 10,
      }}
    />
  );
};
```

#### **B. Configuración para Diferentes Tipos de Contenido**
```javascript
// Lista con items de altura variable
const VariableHeightList = ({ items }) => {
  const renderItem = useCallback(({ item, index }) => (
    <VariableHeightItem item={item} />
  ), []);

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      
      // Para items de altura variable
      getItemLayout={undefined} // No usar getItemLayout
      removeClippedSubviews={false} // Desactivar para altura variable
      
      // Optimizaciones específicas
      maxToRenderPerBatch={3}
      windowSize={3}
      initialNumToRender={5}
    />
  );
};

// Lista con items de altura fija
const FixedHeightList = ({ items }) => {
  const ITEM_HEIGHT = 100;
  
  const getItemLayout = useCallback((data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  }), []);

  return (
    <FlatList
      data={items}
      renderItem={({ item }) => <FixedHeightItem item={item} />}
      keyExtractor={(item) => item.id.toString()}
      getItemLayout={getItemLayout}
      
      // Optimizaciones para altura fija
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
    />
  );
};
```

### **3. VirtualizedList para Listas Extremadamente Largas**

#### **A. Implementación Básica**
```javascript
import { VirtualizedList } from 'react-native';

const UltraLongList = ({ items }) => {
  const getItem = useCallback((data, index) => data[index], []);
  const getItemCount = useCallback((data) => data.length, []);
  
  const renderItem = useCallback(({ item, index }) => (
    <ListItem item={item} index={index} />
  ), []);

  const keyExtractor = useCallback((item) => item.id.toString(), []);

  return (
    <VirtualizedList
      data={items}
      getItem={getItem}
      getItemCount={getItemCount}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      
      // Configuración para listas muy largas
      initialNumToRender={5}
      maxToRenderPerBatch={3}
      windowSize={3}
      removeClippedSubviews={true}
      
      // Optimizaciones de scroll
      scrollEventThrottle={16}
      decelerationRate="fast"
    />
  );
};
```

#### **B. Lista con Paginación**
```javascript
const PaginatedList = ({ items, onLoadMore, hasMore }) => {
  const [refreshing, setRefreshing] = useState(false);
  
  const handleLoadMore = useCallback(() => {
    if (hasMore && !refreshing) {
      onLoadMore();
    }
  }, [hasMore, refreshing, onLoadMore]);

  const handleRefresh = useCallback(async () => {
    setRefreshing(true);
    try {
      await onLoadMore();
    } finally {
      setRefreshing(false);
    }
  }, [onLoadMore]);

  const renderItem = useCallback(({ item, index }) => (
    <ListItem item={item} index={index} />
  ), []);

  const keyExtractor = useCallback((item) => item.id.toString(), []);

  const ListFooterComponent = useCallback(() => (
    hasMore ? (
      <View style={styles.loadingFooter}>
        <ActivityIndicator size="small" />
        <Text>Cargando más items...</Text>
      </View>
    ) : null
  ), [hasMore]);

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      
      // Paginación
      onEndReached={handleLoadMore}
      onEndReachedThreshold={0.1}
      
      // Pull to refresh
      refreshing={refreshing}
      onRefresh={handleRefresh}
      
      // Footer para indicar carga
      ListFooterComponent={ListFooterComponent}
      
      // Optimizaciones
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={20}
    />
  );
};
```

### **4. Optimización de Scroll y Navegación**

#### **A. Scroll Suave y Responsivo**
```javascript
const SmoothScrollList = ({ items }) => {
  const flatListRef = useRef(null);
  
  // Scroll a posición específica
  const scrollToIndex = useCallback((index) => {
    flatListRef.current?.scrollToIndex({
      index,
      animated: true,
      viewPosition: 0.5, // Centrar el item
    });
  }, []);

  // Scroll al top
  const scrollToTop = useCallback(() => {
    flatListRef.current?.scrollToOffset({
      offset: 0,
      animated: true,
    });
  }, []);

  // Scroll a item específico
  const scrollToItem = useCallback((itemId) => {
    const index = items.findIndex(item => item.id === itemId);
    if (index !== -1) {
      scrollToIndex(index);
    }
  }, [items, scrollToIndex]);

  return (
    <View>
      <View style={styles.controls}>
        <Button title="Top" onPress={scrollToTop} />
        <Button title="Item 5" onPress={() => scrollToIndex(4)} />
        <Button title="Último" onPress={() => scrollToIndex(items.length - 1)} />
      </View>
      
      <FlatList
        ref={flatListRef}
        data={items}
        renderItem={({ item }) => <ListItem item={item} />}
        keyExtractor={(item) => item.id.toString()}
        
        // Configuración para scroll suave
        scrollEventThrottle={16}
        decelerationRate="normal"
        showsVerticalScrollIndicator={true}
        
        // Optimizaciones
        removeClippedSubviews={true}
        maxToRenderPerBatch={10}
        windowSize={10}
      />
    </View>
  );
};
```

#### **B. Scroll con Indicadores de Posición**
```javascript
const PositionAwareList = ({ items }) => {
  const [visibleItems, setVisibleItems] = useState([]);
  const [scrollPosition, setScrollPosition] = useState(0);
  
  const onViewableItemsChanged = useCallback(({ viewableItems }) => {
    setVisibleItems(viewableItems);
  }, []);

  const onScroll = useCallback((event) => {
    const offset = event.nativeEvent.contentOffset.y;
    setScrollPosition(offset);
  }, []);

  const viewabilityConfig = useMemo(() => ({
    itemVisiblePercentThreshold: 50,
    minimumViewTime: 100,
  }), []);

  return (
    <View>
      <View style={styles.scrollInfo}>
        <Text>Posición: {scrollPosition.toFixed(0)}</Text>
        <Text>Items visibles: {visibleItems.length}</Text>
        <Text>Primer visible: {visibleItems[0]?.index || 'N/A'}</Text>
        <Text>Último visible: {visibleItems[visibleItems.length - 1]?.index || 'N/A'}</Text>
      </View>
      
      <FlatList
        data={items}
        renderItem={({ item, index }) => (
          <ListItem 
            item={item} 
            index={index}
            isVisible={visibleItems.some(vi => vi.index === index)}
          />
        )}
        keyExtractor={(item) => item.id.toString()}
        
        // Callbacks de scroll
        onViewableItemsChanged={onViewableItemsChanged}
        onScroll={onScroll}
        viewabilityConfig={viewabilityConfig}
        
        // Optimizaciones
        scrollEventThrottle={16}
        removeClippedSubviews={true}
        maxToRenderPerBatch={10}
        windowSize={10}
      />
    </View>
  );
};
```

### **5. Lazy Loading y Carga Diferida**

#### **A. Lazy Loading de Imágenes en Listas**
```javascript
const LazyImageList = ({ items }) => {
  const [visibleImages, setVisibleImages] = useState(new Set());
  
  const onViewableItemsChanged = useCallback(({ viewableItems }) => {
    const newVisibleImages = new Set();
    viewableItems.forEach(({ item }) => {
      if (item.imageUrl) {
        newVisibleImages.add(item.imageUrl);
      }
    });
    setVisibleImages(newVisibleImages);
  }, []);

  const renderItem = useCallback(({ item, index }) => (
    <ListItem
      item={item}
      showImage={visibleImages.has(item.imageUrl)}
      index={index}
    />
  ), [visibleImages]);

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      onViewableItemsChanged={onViewableItemsChanged}
      viewabilityConfig={{
        itemVisiblePercentThreshold: 50,
        minimumViewTime: 100,
      }}
      
      // Optimizaciones
      removeClippedSubviews={true}
      maxToRenderPerBatch={5}
      windowSize={5}
      initialNumToRender={10}
    />
  );
};
```

#### **B. Carga Diferida de Contenido**
```javascript
const LazyContentList = ({ items }) => {
  const [loadedItems, setLoadedItems] = useState(new Set());
  
  const loadItemContent = useCallback(async (itemId) => {
    try {
      // Simular carga de contenido
      await new Promise(resolve => setTimeout(resolve, 100));
      setLoadedItems(prev => new Set([...prev, itemId]));
    } catch (error) {
      console.error('Error loading item content:', error);
    }
  }, []);

  const onViewableItemsChanged = useCallback(({ viewableItems }) => {
    viewableItems.forEach(({ item }) => {
      if (!loadedItems.has(item.id)) {
        loadItemContent(item.id);
      }
    });
  }, [loadedItems, loadItemContent]);

  const renderItem = useCallback(({ item }) => (
    <ListItem
      item={item}
      isLoaded={loadedItems.has(item.id)}
      onLoad={() => loadItemContent(item.id)}
    />
  ), [loadedItems, loadItemContent]);

  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      onViewableItemsChanged={onViewableItemsChanged}
      viewabilityConfig={{
        itemVisiblePercentThreshold: 50,
        minimumViewTime: 200,
      }}
      
      // Optimizaciones para carga diferida
      removeClippedSubviews={true}
      maxToRenderPerBatch={3}
      windowSize={3}
      initialNumToRender={5}
    />
  );
};
```

---

## 💻 Ejercicios Prácticos

### **Ejercicio 1: Lista Optimizada con FlatList**
Crea una lista que use todas las optimizaciones de FlatList.

```javascript
const OptimizedFlatList = ({ items, onItemPress }) => {
  // Implementa:
  // - renderItem memoizado
  // - keyExtractor memoizado
  // - getItemLayout para altura fija
  // - Todas las optimizaciones de rendimiento
  // - Scroll suave y responsivo
};
```

**Tarea**: Crea una lista de 1000 items y mide el performance.

### **Ejercicio 2: Lista con Paginación**
Implementa una lista que cargue items por páginas.

```javascript
const PaginatedList = ({ initialItems, onLoadMore }) => {
  // Implementa:
  // - Estado de items
  // - Función de carga de más items
  // - Indicador de carga
  // - Pull to refresh
  // - Scroll infinito
};
```

**Tarea**: Crea un sistema que cargue 20 items por página.

### **Ejercicio 3: Lista con Lazy Loading**
Crea una lista que cargue contenido de manera diferida.

```javascript
const LazyLoadingList = ({ items }) => {
  // Implementa:
  // - Lazy loading de imágenes
  // - Carga diferida de contenido
  // - Indicadores de carga
  // - Optimizaciones de memoria
};
```

**Tarea**: Implementa lazy loading para imágenes y contenido.

---

## 🎯 Proyecto Integrador

### **Lista de Productos Ultra-Optimizada**

Crea una lista de productos que demuestre todas las técnicas de optimización:

#### **Funcionalidades Requeridas:**
1. **Lista Virtualizada**: Con FlatList optimizado
2. **Lazy Loading**: De imágenes y contenido
3. **Paginación**: Carga infinita de productos
4. **Scroll Suave**: Con controles de navegación
5. **Performance Monitoring**: Métricas de rendimiento

#### **Requisitos Técnicos:**
- **FlatList Optimizado**: Con todas las configuraciones
- **Lazy Loading**: Imágenes y contenido
- **Paginación**: Sistema de carga infinita
- **Scroll Responsivo**: Con indicadores de posición
- **Métricas**: Monitoreo de performance en tiempo real

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [FlatList](https://reactnative.dev/docs/flatlist)
- [VirtualizedList](https://reactnative.dev/docs/virtualizedlist)
- [Performance](https://reactnative.dev/docs/performance)

### **Artículos Recomendados:**
- "FlatList Performance Optimization"
- "Virtualized Lists in React Native"
- "Scroll Performance Best Practices"

---

## 🎓 Próximos Pasos

### **En la Siguiente Clase Aprenderás:**
- **Optimización de Imágenes**: Lazy loading, caching y compresión
- **Optimización de Navegación**: Lazy loading de pantallas y transiciones

---

**🎯 Objetivo de la Clase**: Dominar la optimización de listas y scroll en React Native para crear aplicaciones con máximo rendimiento.

**💡 Consejo**: Las listas son el componente más crítico para el performance. Invierte tiempo en optimizarlas correctamente desde el inicio.
