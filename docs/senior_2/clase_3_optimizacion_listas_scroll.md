# 📱 Clase 3: Optimización de Listas y Scroll

## 📋 Objetivos de la Clase
- Dominar la optimización de FlatList para listas grandes
- Implementar virtualización para mejorar el rendimiento
- Aprender lazy loading de imágenes y contenido
- Optimizar el scroll y la navegación en listas
- Crear listas de alto rendimiento para apps móviles

## ⏱️ Duración
**2 horas**

## 🔗 Navegación
- **Anterior**: [Clase 2: Optimización de Componentes](clase_2_optimizacion_componentes.md)
- **Siguiente**: [Clase 4: Optimización de Navegación y Carga](clase_4_optimizacion_navegacion_carga.md)
- **Módulo**: [README.md](README.md)
- **Inicio**: [🏠](../../README.md)

---

## 🚀 FlatList Optimizada - El Corazón del Rendimiento

### ¿Por qué FlatList es Mejor que map()?
```javascript
// ❌ Usando map() - Renderiza todos los elementos de una vez
const BadList = ({ items }) => {
  return (
    <ScrollView>
      {items.map(item => (
        <ExpensiveItemComponent 
          key={item.id} 
          item={item} 
        />
      ))}
    </ScrollView>
  );
};

// ✅ Usando FlatList - Renderiza solo elementos visibles
const GoodList = ({ items }) => {
  const renderItem = useCallback(({ item }) => (
    <ExpensiveItemComponent item={item} />
  ), []);
  
  const keyExtractor = useCallback((item) => item.id.toString(), []);
  
  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      initialNumToRender={10} // Solo renderizar 10 elementos inicialmente
      maxToRenderPerBatch={5} // Renderizar máximo 5 por lote
      windowSize={5} // Mantener 5 "ventanas" en memoria
      removeClippedSubviews={true} // Eliminar elementos fuera de pantalla
      getItemLayout={getItemLayout} // Optimización para elementos de altura fija
    />
  );
};
```

### Configuración Avanzada de FlatList
```javascript
const OptimizedFlatList = ({ items, onItemPress, onItemDelete }) => {
  // Memoizar funciones para evitar re-renders
  const renderItem = useCallback(({ item, index }) => (
    <ListItem
      item={item}
      index={index}
      onPress={() => onItemPress(item)}
      onDelete={() => onItemDelete(item.id)}
    />
  ), [onItemPress, onItemDelete]);
  
  const keyExtractor = useCallback((item) => item.id.toString(), []);
  
  // Función para obtener layout de elementos (optimización para altura fija)
  const getItemLayout = useCallback((data, index) => ({
    length: 80, // Altura fija de cada item
    offset: 80 * index,
    index,
  }), []);
  
  // Función para renderizar separadores
  const ItemSeparator = useCallback(() => (
    <View style={styles.separator} />
  ), []);
  
  // Función para renderizar header
  const ListHeaderComponent = useCallback(() => (
    <View style={styles.header}>
      <Text style={styles.headerTitle}>Lista Optimizada</Text>
      <Text style={styles.headerSubtitle}>
        Total: {items.length} elementos
      </Text>
    </View>
  ), [items.length]);
  
  // Función para renderizar footer
  const ListFooterComponent = useCallback(() => (
    <View style={styles.footer}>
      <Text style={styles.footerText}>
        Fin de la lista
      </Text>
    </View>
  ), []);
  
  // Función para renderizar componente vacío
  const ListEmptyComponent = useCallback(() => (
    <View style={styles.emptyContainer}>
      <Text style={styles.emptyText}>No hay elementos para mostrar</Text>
    </View>
  ), []);
  
  // Función para manejar scroll
  const handleScroll = useCallback((event) => {
    const offsetY = event.nativeEvent.contentOffset.y;
    console.log('📱 Scroll offset:', offsetY);
  }, []);
  
  // Función para manejar scroll al final
  const handleEndReached = useCallback(() => {
    console.log('🔚 Llegaste al final de la lista');
    // Aquí podrías cargar más datos
  }, []);
  
  // Función para refrescar
  const handleRefresh = useCallback(() => {
    console.log('🔄 Refrescando lista...');
    // Aquí podrías recargar datos
  }, []);
  
  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      ItemSeparatorComponent={ItemSeparator}
      ListHeaderComponent={ListHeaderComponent}
      ListFooterComponent={ListFooterComponent}
      ListEmptyComponent={ListEmptyComponent}
      onScroll={handleScroll}
      onEndReached={handleEndReached}
      onEndReachedThreshold={0.1}
      onRefresh={handleRefresh}
      refreshing={false}
      // Configuración de performance
      initialNumToRender={10}
      maxToRenderPerBatch={5}
      windowSize={5}
      removeClippedSubviews={true}
      updateCellsBatchingPeriod={50}
      disableVirtualization={false}
      // Configuración de scroll
      showsVerticalScrollIndicator={false}
      scrollEventThrottle={16} // 60 FPS
      decelerationRate="fast"
      // Configuración de memoria
      maintainVisibleContentPosition={{
        minIndexForVisible: 0,
        autoscrollToTopThreshold: 10,
      }}
    />
  );
};

// Componente de item optimizado
const ListItem = React.memo(({ item, index, onPress, onDelete }) => {
  console.log(`🔄 Renderizando item ${index}: ${item.name}`);
  
  return (
    <View style={styles.itemContainer}>
      <TouchableOpacity 
        style={styles.itemContent} 
        onPress={() => onPress(item)}
        activeOpacity={0.7}
      >
        <Image 
          source={{ uri: item.image }} 
          style={styles.itemImage}
          resizeMode="cover"
        />
        <View style={styles.itemText}>
          <Text style={styles.itemTitle}>{item.name}</Text>
          <Text style={styles.itemDescription}>{item.description}</Text>
        </View>
      </TouchableOpacity>
      
      <TouchableOpacity 
        style={styles.deleteButton} 
        onPress={() => onDelete(item.id)}
        activeOpacity={0.5}
      >
        <Text style={styles.deleteButtonText}>🗑️</Text>
      </TouchableOpacity>
    </View>
  );
});
```

---

## 🎯 Virtualización - Renderizado Inteligente

### ¿Qué es la Virtualización?
La virtualización es una técnica que solo renderiza los elementos que están visibles en pantalla, mejorando significativamente el rendimiento.

### Implementación de Virtualización Personalizada
```javascript
class VirtualizedList extends Component {
  constructor(props) {
    super(props);
    this.state = {
      visibleRange: { start: 0, end: 10 },
      scrollOffset: 0,
      containerHeight: 0,
      itemHeight: 80
    };
    
    this.scrollViewRef = null;
    this.measurements = new Map();
  }
  
  // Calcular qué elementos están visibles
  calculateVisibleRange = (scrollOffset, containerHeight) => {
    const { itemHeight } = this.state;
    const startIndex = Math.floor(scrollOffset / itemHeight);
    const endIndex = Math.min(
      startIndex + Math.ceil(containerHeight / itemHeight) + 1,
      this.props.data.length
    );
    
    return {
      start: Math.max(0, startIndex - 2), // Buffer de 2 elementos
      end: Math.min(this.props.data.length, endIndex + 2)
    };
  };
  
  // Manejar scroll
  handleScroll = (event) => {
    const scrollOffset = event.nativeEvent.contentOffset.y;
    const { containerHeight } = this.state;
    
    const visibleRange = this.calculateVisibleRange(scrollOffset, containerHeight);
    
    this.setState({
      visibleRange,
      scrollOffset
    });
  };
  
  // Medir altura del contenedor
  onLayout = (event) => {
    const { height } = event.nativeEvent.layout;
    this.setState({ containerHeight: height });
  };
  
  // Renderizar solo elementos visibles
  renderVisibleItems = () => {
    const { data, renderItem } = this.props;
    const { visibleRange, itemHeight, scrollOffset } = this.state;
    
    const items = [];
    
    for (let i = visibleRange.start; i < visibleRange.end; i++) {
      if (i >= data.length) break;
      
      const item = data[i];
      const top = i * itemHeight;
      
      items.push(
        <View
          key={item.id || i}
          style={[
            styles.virtualizedItem,
            {
              position: 'absolute',
              top,
              height: itemHeight,
              left: 0,
              right: 0
            }
          ]}
        >
          {renderItem({ item, index: i })}
        </View>
      );
    }
    
    return items;
  };
  
  render() {
    const { data, itemHeight } = this.props;
    const { containerHeight } = this.state;
    
    const totalHeight = data.length * itemHeight;
    
    return (
      <View style={styles.container} onLayout={this.onLayout}>
        <ScrollView
          ref={ref => this.scrollViewRef = ref}
          style={styles.scrollView}
          contentContainerStyle={{
            height: totalHeight
          }}
          onScroll={this.handleScroll}
          scrollEventThrottle={16}
          showsVerticalScrollIndicator={false}
        >
          {this.renderVisibleItems()}
        </ScrollView>
        
        {/* Indicador de elementos visibles */}
        <View style={styles.debugInfo}>
          <Text style={styles.debugText}>
            Visible: {this.state.visibleRange.start}-{this.state.visibleRange.end}
          </Text>
          <Text style={styles.debugText}>
            Total: {data.length}
          </Text>
        </View>
      </View>
    );
  }
}

// Uso del componente virtualizado
const VirtualizedExample = () => {
  const [data, setData] = useState([]);
  
  useEffect(() => {
    // Generar datos de ejemplo
    const generateData = () => {
      const items = [];
      for (let i = 0; i < 10000; i++) {
        items.push({
          id: i,
          name: `Item ${i}`,
          description: `Descripción del item ${i}`,
          image: `https://picsum.photos/60/60?random=${i}`
        });
      }
      return items;
    };
    
    setData(generateData());
  }, []);
  
  const renderItem = useCallback(({ item, index }) => (
    <View style={styles.item}>
      <Image source={{ uri: item.image }} style={styles.itemImage} />
      <View style={styles.itemContent}>
        <Text style={styles.itemTitle}>{item.name}</Text>
        <Text style={styles.itemDescription}>{item.description}</Text>
      </View>
    </View>
  ), []);
  
  return (
    <VirtualizedList
      data={data}
      renderItem={renderItem}
      itemHeight={80}
    />
  );
};
```

---

## 🖼️ Lazy Loading de Imágenes

### ¿Por qué Lazy Loading?
- **Reduce el uso de memoria** al cargar solo imágenes visibles
- **Mejora el tiempo de carga** inicial
- **Optimiza el uso de ancho de banda**

### Implementación de Lazy Loading
```javascript
const LazyImage = ({ source, style, placeholder, onLoad, onError }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isError, setIsError] = useState(false);
  const [isInView, setIsInView] = useState(false);
  
  const imageRef = useRef(null);
  
  // Verificar si la imagen está en vista
  const checkIfInView = useCallback(() => {
    if (!imageRef.current) return;
    
    imageRef.current.measure((x, y, width, height, pageX, pageY) => {
      const screenHeight = Dimensions.get('window').height;
      const isVisible = pageY < screenHeight && pageY + height > 0;
      
      if (isVisible && !isInView) {
        setIsInView(true);
      }
    });
  }, [isInView]);
  
  // Cargar imagen cuando esté en vista
  useEffect(() => {
    if (isInView && !isLoaded && !isError) {
      // Simular carga de imagen
      const loadImage = async () => {
        try {
          // En una app real, aquí cargarías la imagen
          await new Promise(resolve => setTimeout(resolve, 500));
          setIsLoaded(true);
          onLoad?.();
        } catch (error) {
          setIsError(true);
          onError?.(error);
        }
      };
      
      loadImage();
    }
  }, [isInView, isLoaded, isError, onLoad, onError]);
  
  // Verificar visibilidad en scroll
  useEffect(() => {
    const checkVisibility = () => {
      checkIfInView();
    };
    
    // Verificar cada 100ms durante scroll
    const interval = setInterval(checkVisibility, 100);
    
    return () => clearInterval(interval);
  }, [checkIfInView]);
  
  return (
    <View 
      ref={imageRef} 
      style={[styles.imageContainer, style]}
      onLayout={checkIfInView}
    >
      {/* Placeholder mientras carga */}
      {!isLoaded && !isError && (
        <View style={[styles.placeholder, style]}>
          {placeholder || (
            <View style={styles.defaultPlaceholder}>
              <ActivityIndicator size="small" color="#666" />
            </View>
          )}
        </View>
      )}
      
      {/* Imagen real */}
      {isInView && (
        <Image
          source={source}
          style={[
            styles.image,
            style,
            { opacity: isLoaded ? 1 : 0 }
          ]}
          onLoad={() => setIsLoaded(true)}
          onError={() => setIsError(true)}
          resizeMode="cover"
          // Configuración de cache
          cache="force-cache"
          // Progreso de carga
          onProgress={(event) => {
            const progress = event.nativeEvent.loaded / event.nativeEvent.total;
            console.log(`📸 Carga de imagen: ${(progress * 100).toFixed(1)}%`);
          }}
        />
      )}
      
      {/* Estado de error */}
      {isError && (
        <View style={[styles.errorContainer, style]}>
          <Text style={styles.errorText}>❌ Error</Text>
        </View>
      )}
    </View>
  );
};

// Lista con lazy loading de imágenes
const LazyImageList = ({ images }) => {
  const renderItem = useCallback(({ item, index }) => (
    <View style={styles.listItem}>
      <LazyImage
        source={{ uri: item.url }}
        style={styles.listItemImage}
        placeholder={
          <View style={styles.listItemPlaceholder}>
            <Text style={styles.placeholderText}>Cargando...</Text>
          </View>
        }
        onLoad={() => console.log(`✅ Imagen ${index} cargada`)}
        onError={(error) => console.log(`❌ Error cargando imagen ${index}:`, error)}
      />
      <Text style={styles.listItemTitle}>{item.title}</Text>
    </View>
  ), []);
  
  return (
    <FlatList
      data={images}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      numColumns={2}
      columnWrapperStyle={styles.row}
      showsVerticalScrollIndicator={false}
      // Configuración de performance
      initialNumToRender={6}
      maxToRenderPerBatch={4}
      windowSize={3}
      removeClippedSubviews={true}
    />
  );
};
```

---

## 🔄 Lazy Loading de Contenido

### Implementación de Lazy Loading para Contenido
```javascript
const LazyContentLoader = ({ children, threshold = 100, placeholder }) => {
  const [isVisible, setIsVisible] = useState(false);
  const [isLoaded, setIsLoaded] = useState(false);
  
  const contentRef = useRef(null);
  
  // Verificar si el contenido está en vista
  const checkVisibility = useCallback(() => {
    if (!contentRef.current) return;
    
    contentRef.current.measure((x, y, width, height, pageX, pageY) => {
      const screenHeight = Dimensions.get('window').height;
      const isInView = pageY < screenHeight + threshold;
      
      if (isInView && !isVisible) {
        setIsVisible(true);
        
        // Simular carga de contenido
        setTimeout(() => {
          setIsLoaded(true);
        }, 300);
      }
    });
  }, [isVisible, threshold]);
  
  // Verificar visibilidad en scroll
  useEffect(() => {
    const checkVisibilityInterval = setInterval(checkVisibility, 100);
    
    return () => clearInterval(checkVisibilityInterval);
  }, [checkVisibility]);
  
  return (
    <View ref={contentRef} style={styles.lazyContainer}>
      {!isLoaded ? (
        <View style={styles.placeholderContainer}>
          {placeholder || (
            <View style={styles.defaultPlaceholder}>
              <ActivityIndicator size="large" color="#666" />
              <Text style={styles.placeholderText}>Cargando contenido...</Text>
            </View>
          )}
        </View>
      ) : (
        <View style={styles.contentContainer}>
          {children}
        </View>
      )}
    </View>
  );
};

// Uso en lista con lazy loading de contenido
const LazyContentList = ({ sections }) => {
  const renderSection = useCallback(({ item: section, index }) => (
    <LazyContentLoader
      key={section.id}
      threshold={200}
      placeholder={
        <View style={styles.sectionPlaceholder}>
          <View style={styles.sectionPlaceholderHeader} />
          <View style={styles.sectionPlaceholderContent} />
        </View>
      }
    >
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>{section.title}</Text>
        <Text style={styles.sectionDescription}>{section.description}</Text>
        
        {section.items && (
          <FlatList
            data={section.items}
            horizontal
            showsHorizontalScrollIndicator={false}
            renderItem={({ item }) => (
              <View style={styles.sectionItem}>
                <Image source={{ uri: item.image }} style={styles.sectionItemImage} />
                <Text style={styles.sectionItemTitle}>{item.title}</Text>
              </View>
            )}
            keyExtractor={(item) => item.id.toString()}
          />
        )}
      </View>
    </LazyContentLoader>
  ), []);
  
  return (
    <FlatList
      data={sections}
      renderItem={renderSection}
      keyExtractor={(item) => item.id.toString()}
      showsVerticalScrollIndicator={false}
      // Configuración para contenido pesado
      initialNumToRender={2}
      maxToRenderPerBatch={1}
      windowSize={3}
      removeClippedSubviews={true}
    />
  );
};
```

---

## 📱 Ejercicios Prácticos

### Ejercicio 1: Lista Virtualizada Personalizada
Crea una lista virtualizada que maneje elementos de diferentes alturas.

```javascript
// Tu código aquí
const VariableHeightList = ({ items, renderItem }) => {
  // Implementa:
  // 1. Cálculo dinámico de altura de elementos
  // 2. Virtualización con alturas variables
  // 3. Optimización de scroll
  // 4. Gestión de memoria
};
```

### Ejercicio 2: Lazy Loading Inteligente
Implementa un sistema de lazy loading que pre-cargue contenido cercano.

```javascript
// Tu código aquí
const SmartLazyLoader = ({ children, preloadDistance = 300 }) => {
  // Implementa:
  // 1. Pre-carga de contenido cercano
  // 2. Gestión de prioridades de carga
  // 3. Cache inteligente
  // 4. Cancelación de cargas innecesarias
};
```

### Ejercicio 3: Lista con Filtros Optimizada
Crea una lista que mantenga el rendimiento con filtros en tiempo real.

```javascript
// Tu código aquí
const FilteredList = ({ items, filters, searchText }) => {
  // Implementa:
  // 1. Filtrado optimizado con useMemo
  // 2. Búsqueda en tiempo real
  // 3. Paginación virtual
  // 4. Cache de resultados filtrados
};
```

---

## 🔍 Resumen de la Clase

### ✅ Lo que Aprendiste
- **FlatList optimizada** con configuración avanzada de performance
- **Virtualización** para renderizar solo elementos visibles
- **Lazy loading** de imágenes y contenido
- **Técnicas de scroll** optimizado para listas grandes

### 🎯 Próximos Pasos
En la siguiente clase aprenderás:
- **Lazy loading de pantallas** y navegación
- **Preloading** de contenido
- **Optimización de transiciones** entre pantallas

### 💡 Consejos Clave
1. **Usa FlatList** en lugar de map() para listas grandes
2. **Implementa virtualización** cuando tengas miles de elementos
3. **Lazy loading** es esencial para imágenes y contenido pesado
4. **Configura FlatList** según tus necesidades específicas
5. **Mide el rendimiento** antes y después de optimizar

---

## 📚 Recursos Adicionales
- [FlatList Performance](https://reactnative.dev/docs/flatlist#performance-optimization)
- [Virtualization in React Native](https://reactnative.dev/docs/virtualizedlist)
- [Image Performance](https://reactnative.dev/docs/image#performance)
- [ScrollView Optimization](https://reactnative.dev/docs/scrollview#performance-considerations)

---

**¿Tienes alguna pregunta sobre la optimización de listas y scroll? ¿Te gustaría que profundice en algún aspecto específico antes de continuar con la siguiente clase?**
