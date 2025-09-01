# Clase 5: Optimización y Performance

## Navegación
- **Anterior**: [Clase 4: Autenticación y Autorización](./clase_4_autenticacion_autorizacion.md)
- **Siguiente**: [Módulo 7: Almacenamiento Local](./../midLevel_4/README.md)
- **Módulo**: [Módulo 6: APIs y Networking](./README.md)
- **Índice**: [Índice Completo](./../INDICE_COMPLETO.md)
- **Navegación Rápida**: [Navegación Rápida](./../NAVEGACION_RAPIDA.md)

## 🎯 Objetivos de la Clase
- Implementar técnicas de optimización para React Native
- Optimizar re-renders con React.memo, useMemo y useCallback
- Implementar virtualización para listas grandes
- Optimizar imágenes y assets
- Implementar profiling y métricas de performance

## 📚 Contenido Teórico

### 1. Principios de Performance en React Native

#### ¿Por qué es Importante la Performance?
- **Experiencia del Usuario**: Apps lentas frustran a los usuarios
- **Consumo de Batería**: Código ineficiente consume más recursos
- **Memoria**: Apps con memory leaks pueden crashear
- **App Store**: Apple y Google evalúan performance en reviews

#### Métricas Clave de Performance
```javascript
// Métricas importantes a monitorear
const performanceMetrics = {
  // Tiempo de carga inicial
  INITIAL_LOAD_TIME: 'Tiempo desde splash hasta primera pantalla',
  
  // Tiempo de respuesta de interacciones
  INTERACTION_RESPONSE_TIME: 'Tiempo desde toque hasta respuesta visual',
  
  // FPS (Frames Per Second)
  FRAME_RATE: 'Frames por segundo, idealmente 60 FPS',
  
  // Uso de memoria
  MEMORY_USAGE: 'Consumo de memoria RAM',
  
  // Tiempo de JavaScript
  JS_THREAD_TIME: 'Tiempo de ejecución en el hilo principal',
  
  // Tiempo de bridge
  BRIDGE_TIME: 'Tiempo de comunicación nativo-JavaScript',
};
```

### 2. Optimización de Re-renders

#### React.memo para Componentes
```javascript
// Componente optimizado con React.memo
import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

const UserCard = React.memo(({ user, onPress, isSelected }) => {
  console.log(`UserCard renderizado para usuario: ${user.id}`);
  
  return (
    <TouchableOpacity
      onPress={() => onPress(user.id)}
      style={{
        padding: 15,
        backgroundColor: isSelected ? '#e3f2fd' : '#ffffff',
        borderWidth: 1,
        borderColor: '#e0e0e0',
        borderRadius: 8,
        marginBottom: 10,
      }}
    >
      <Text style={{ fontSize: 18, fontWeight: 'bold' }}>{user.name}</Text>
      <Text style={{ fontSize: 14, color: '#666' }}>{user.email}</Text>
      <Text style={{ fontSize: 12, color: '#999' }}>{user.role}</Text>
    </TouchableOpacity>
  );
}, (prevProps, nextProps) => {
  // Función de comparación personalizada
  return (
    prevProps.user.id === nextProps.user.id &&
    prevProps.isSelected === nextProps.isSelected &&
    prevProps.onPress === nextProps.onPress
  );
});

export default UserCard;
```

#### useMemo para Valores Calculados
```javascript
// Hook personalizado optimizado con useMemo
import React, { useState, useMemo } from 'react';
import { View, Text, FlatList } from 'react-native';

const UserList = ({ users, searchTerm, sortBy }) => {
  const [selectedUserId, setSelectedUserId] = useState(null);

  // Memoizar usuarios filtrados y ordenados
  const filteredAndSortedUsers = useMemo(() => {
    console.log('Recalculando usuarios filtrados y ordenados');
    
    let filtered = users;
    
    // Filtrar por término de búsqueda
    if (searchTerm) {
      filtered = users.filter(user =>
        user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
        user.email.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }
    
    // Ordenar usuarios
    if (sortBy) {
      filtered = [...filtered].sort((a, b) => {
        switch (sortBy) {
          case 'name':
            return a.name.localeCompare(b.name);
          case 'email':
            return a.email.localeCompare(b.email);
          case 'role':
            return a.role.localeCompare(b.role);
          default:
            return 0;
        }
      });
    }
    
    return filtered;
  }, [users, searchTerm, sortBy]); // Solo recalcular si cambian estas dependencias

  // Memoizar estadísticas
  const stats = useMemo(() => {
    const total = users.length;
    const filtered = filteredAndSortedUsers.length;
    const selected = selectedUserId ? 1 : 0;
    
    return {
      total,
      filtered,
      selected,
      percentage: total > 0 ? Math.round((filtered / total) * 100) : 0,
    };
  }, [users.length, filteredAndSortedUsers.length, selectedUserId]);

  const handleUserPress = React.useCallback((userId) => {
    setSelectedUserId(userId);
  }, []);

  const renderUser = React.useCallback(({ item }) => (
    <UserCard
      user={item}
      onPress={handleUserPress}
      isSelected={item.id === selectedUserId}
    />
  ), [handleUserPress, selectedUserId]);

  return (
    <View style={{ flex: 1 }}>
      {/* Estadísticas */}
      <View style={{ padding: 15, backgroundColor: '#f5f5f5' }}>
        <Text style={{ fontSize: 16, fontWeight: 'bold' }}>
          Usuarios: {stats.filtered}/{stats.total} ({stats.percentage}%)
        </Text>
        {stats.selected > 0 && (
          <Text style={{ fontSize: 14, color: '#666' }}>
            Seleccionado: {stats.selected}
          </Text>
        )}
      </View>

      {/* Lista de usuarios */}
      <FlatList
        data={filteredAndSortedUsers}
        renderItem={renderUser}
        keyExtractor={item => item.id.toString()}
        initialNumToRender={10}
        maxToRenderPerBatch={10}
        windowSize={10}
        removeClippedSubviews={true}
        getItemLayout={(data, index) => ({
          length: 80, // altura fija de cada item
          offset: 80 * index,
          index,
        })}
      />
    </View>
  );
};

export default UserList;
```

#### useCallback para Funciones
```javascript
// Hook personalizado con useCallback optimizado
import React, { useState, useCallback } from 'react';
import { View, Text, TouchableOpacity, Alert } from 'react-native';

const UserActions = ({ userId, onUpdate, onDelete }) => {
  const [isLoading, setIsLoading] = useState(false);

  // Memoizar función de actualización
  const handleUpdate = useCallback(async () => {
    if (isLoading) return;
    
    setIsLoading(true);
    try {
      await onUpdate(userId);
      Alert.alert('Éxito', 'Usuario actualizado correctamente');
    } catch (error) {
      Alert.alert('Error', 'No se pudo actualizar el usuario');
    } finally {
      setIsLoading(false);
    }
  }, [userId, onUpdate, isLoading]);

  // Memoizar función de eliminación
  const handleDelete = useCallback(async () => {
    if (isLoading) return;
    
    Alert.alert(
      'Confirmar eliminación',
      '¿Estás seguro de que quieres eliminar este usuario?',
      [
        { text: 'Cancelar', style: 'cancel' },
        {
          text: 'Eliminar',
          style: 'destructive',
          onPress: async () => {
            setIsLoading(true);
            try {
              await onDelete(userId);
              Alert.alert('Éxito', 'Usuario eliminado correctamente');
            } catch (error) {
              Alert.alert('Error', 'No se pudo eliminar el usuario');
            } finally {
              setIsLoading(false);
            }
          },
        },
      ]
    );
  }, [userId, onDelete, isLoading]);

  return (
    <View style={{ flexDirection: 'row', gap: 10 }}>
      <TouchableOpacity
        onPress={handleUpdate}
        disabled={isLoading}
        style={{
          padding: 10,
          backgroundColor: '#2196f3',
          borderRadius: 5,
          opacity: isLoading ? 0.6 : 1,
        }}
      >
        <Text style={{ color: 'white', fontWeight: 'bold' }}>
          {isLoading ? 'Actualizando...' : 'Actualizar'}
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        onPress={handleDelete}
        disabled={isLoading}
        style={{
          padding: 10,
          backgroundColor: '#f44336',
          borderRadius: 5,
          opacity: isLoading ? 0.6 : 1,
        }}
      >
        <Text style={{ color: 'white', fontWeight: 'bold' }}>
          {isLoading ? 'Eliminando...' : 'Eliminar'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

export default UserActions;
```

### 3. Virtualización para Listas Grandes

#### FlatList Optimizada
```javascript
// Componente de lista virtualizada optimizada
import React, { useState, useMemo, useCallback } from 'react';
import {
  View,
  Text,
  FlatList,
  TextInput,
  ActivityIndicator,
  Dimensions,
} from 'react-native';

const { width, height } = Dimensions.get('window');

const VirtualizedUserList = ({ users, onUserPress, onLoadMore }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);

  // Memoizar usuarios filtrados
  const filteredUsers = useMemo(() => {
    if (!searchTerm) return users;
    
    return users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      user.email.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [users, searchTerm]);

  // Memoizar renderItem
  const renderItem = useCallback(({ item, index }) => (
    <UserListItem
      user={item}
      index={index}
      onPress={() => onUserPress(item)}
    />
  ), [onUserPress]);

  // Memoizar keyExtractor
  const keyExtractor = useCallback((item) => item.id.toString(), []);

  // Memoizar getItemLayout para altura fija
  const getItemLayout = useCallback((data, index) => ({
    length: 100, // altura fija de cada item
    offset: 100 * index,
    index,
  }), []);

  // Memoizar separador
  const ItemSeparator = useCallback(() => (
    <View style={{ height: 1, backgroundColor: '#e0e0e0' }} />
  ), []);

  // Memoizar header
  const ListHeaderComponent = useCallback(() => (
    <View style={{ padding: 15, backgroundColor: '#f5f5f5' }}>
      <Text style={{ fontSize: 18, fontWeight: 'bold', marginBottom: 10 }}>
        Usuarios ({filteredUsers.length})
      </Text>
      <TextInput
        placeholder="Buscar usuarios..."
        value={searchTerm}
        onChangeText={setSearchTerm}
        style={{
          padding: 10,
          backgroundColor: 'white',
          borderRadius: 5,
          borderWidth: 1,
          borderColor: '#ddd',
        }}
      />
    </View>
  ), [searchTerm, filteredUsers.length]);

  // Memoizar footer
  const ListFooterComponent = useCallback(() => (
    isLoading ? (
      <View style={{ padding: 20, alignItems: 'center' }}>
        <ActivityIndicator size="small" color="#666" />
        <Text style={{ marginTop: 10, color: '#666' }}>Cargando más usuarios...</Text>
      </View>
    ) : null
  ), [isLoading]);

  // Memoizar onEndReached
  const handleEndReached = useCallback(() => {
    if (!isLoading && onLoadMore) {
      onLoadMore();
    }
  }, [isLoading, onLoadMore]);

  // Memoizar onRefresh
  const handleRefresh = useCallback(async () => {
    setRefreshing(true);
    try {
      // Simular refresh
      await new Promise(resolve => setTimeout(resolve, 1000));
    } finally {
      setRefreshing(false);
    }
  }, []);

  return (
    <FlatList
      data={filteredUsers}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      ItemSeparatorComponent={ItemSeparator}
      ListHeaderComponent={ListHeaderComponent}
      ListFooterComponent={ListFooterComponent}
      onEndReached={handleEndReached}
      onEndReachedThreshold={0.1}
      onRefresh={handleRefresh}
      refreshing={refreshing}
      // Configuraciones de performance
      initialNumToRender={10}
      maxToRenderPerBatch={10}
      windowSize={10}
      removeClippedSubviews={true}
      updateCellsBatchingPeriod={50}
      // Configuraciones de scroll
      showsVerticalScrollIndicator={false}
      scrollEventThrottle={16}
      // Configuraciones de memoria
      maintainVisibleContentPosition={{
        minIndexForVisible: 0,
        autoscrollToTopThreshold: 10,
      }}
    />
  );
};

// Componente de item optimizado
const UserListItem = React.memo(({ user, index, onPress }) => {
  const handlePress = useCallback(() => {
    onPress(user);
  }, [user, onPress]);

  return (
    <View style={{
      padding: 15,
      backgroundColor: index % 2 === 0 ? '#fafafa' : '#ffffff',
      minHeight: 100,
      justifyContent: 'center',
    }}>
      <Text style={{ fontSize: 16, fontWeight: 'bold' }}>{user.name}</Text>
      <Text style={{ fontSize: 14, color: '#666', marginTop: 5 }}>{user.email}</Text>
      <Text style={{ fontSize: 12, color: '#999', marginTop: 5 }}>{user.role}</Text>
      <TouchableOpacity
        onPress={handlePress}
        style={{
          marginTop: 10,
          padding: 8,
          backgroundColor: '#2196f3',
          borderRadius: 5,
          alignSelf: 'flex-start',
        }}
      >
        <Text style={{ color: 'white', fontSize: 12 }}>Ver detalles</Text>
      </TouchableOpacity>
    </View>
  );
});

export default VirtualizedUserList;
```

### 4. Optimización de Imágenes

#### Componente de Imagen Optimizada
```javascript
// Componente de imagen con lazy loading y caché
import React, { useState, useCallback, useMemo } from 'react';
import {
  View,
  Image,
  ActivityIndicator,
  Dimensions,
  TouchableOpacity,
} from 'react-native';
import FastImage from 'react-native-fast-image';

const { width: screenWidth } = Dimensions.get('window');

const OptimizedImage = ({
  source,
  style,
  placeholder,
  resizeMode = 'cover',
  onPress,
  onLoad,
  onError,
  priority = 'normal',
  cache = 'immutable',
  ...props
}) => {
  const [isLoading, setIsLoading] = useState(true);
  const [hasError, setHasError] = useState(false);

  // Memoizar estilos
  const imageStyle = useMemo(() => [
    style,
    { opacity: isLoading ? 0 : 1 }
  ], [style, isLoading]);

  // Memoizar placeholder
  const placeholderStyle = useMemo(() => [
    style,
    {
      position: 'absolute',
      opacity: isLoading ? 1 : 0,
    }
  ], [style, isLoading]);

  // Memoizar source para FastImage
  const fastImageSource = useMemo(() => ({
    uri: typeof source === 'string' ? source : source?.uri,
    priority: FastImage.priority[priority.toUpperCase()],
    cache: FastImage.cacheControl[cache.toUpperCase()],
  }), [source, priority, cache]);

  // Memoizar callbacks
  const handleLoad = useCallback(() => {
    setIsLoading(false);
    setHasError(false);
    onLoad?.();
  }, [onLoad]);

  const handleError = useCallback(() => {
    setIsLoading(false);
    setHasError(true);
    onError?.();
  }, [onError]);

  const handlePress = useCallback(() => {
    if (onPress) {
      onPress();
    }
  }, [onPress]);

  // Si hay error, mostrar placeholder o imagen por defecto
  if (hasError && placeholder) {
    return (
      <TouchableOpacity onPress={handlePress} disabled={!onPress}>
        <Image
          source={placeholder}
          style={style}
          resizeMode={resizeMode}
          {...props}
        />
      </TouchableOpacity>
    );
  }

  return (
    <TouchableOpacity onPress={handlePress} disabled={!onPress}>
      <View style={style}>
        {/* Placeholder mientras carga */}
        {isLoading && placeholder && (
          <Image
            source={placeholder}
            style={placeholderStyle}
            resizeMode={resizeMode}
          />
        )}

        {/* Indicador de carga */}
        {isLoading && !placeholder && (
          <View style={[
            style,
            {
              position: 'absolute',
              justifyContent: 'center',
              alignItems: 'center',
              backgroundColor: '#f0f0f0',
            }
          ]}>
            <ActivityIndicator size="small" color="#666" />
          </View>
        )}

        {/* Imagen principal */}
        <FastImage
          source={fastImageSource}
          style={imageStyle}
          resizeMode={FastImage.resizeMode[resizeMode.toUpperCase()]}
          onLoad={handleLoad}
          onError={handleError}
          {...props}
        />
      </View>
    </TouchableOpacity>
  );
};

// Componente de imagen con lazy loading para listas
const LazyImage = React.memo(({ source, style, ...props }) => {
  const [isVisible, setIsVisible] = useState(false);

  const handleViewRef = useCallback((ref) => {
    if (ref) {
      // Usar Intersection Observer o similar para detectar visibilidad
      // Por simplicidad, asumimos que es visible
      setIsVisible(true);
    }
  }, []);

  if (!isVisible) {
    return (
      <View style={[style, { backgroundColor: '#f0f0f0' }]}>
        <ActivityIndicator size="small" color="#666" />
      </View>
    );
  }

  return (
    <OptimizedImage
      ref={handleViewRef}
      source={source}
      style={style}
      {...props}
    />
  );
});

export { OptimizedImage, LazyImage };
```

### 5. Profiling y Métricas de Performance

#### Hook de Performance
```javascript
// Hook para medir performance de componentes
import { useEffect, useRef, useCallback } from 'react';
import { PerformanceObserver, PerformanceEntry } from 'react-native';

export const usePerformance = (componentName) => {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(0);
  const observer = useRef(null);

  // Contar renders
  useEffect(() => {
    renderCount.current += 1;
    lastRenderTime.current = Date.now();
    
    console.log(`🔄 ${componentName} renderizado ${renderCount.current} veces`);
  });

  // Medir tiempo de render
  const measureRender = useCallback((callback) => {
    const startTime = performance.now();
    
    const result = callback();
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    console.log(`⏱️ ${componentName} render tomó ${duration.toFixed(2)}ms`);
    
    return result;
  }, [componentName]);

  // Medir operación asíncrona
  const measureAsync = useCallback(async (operation, operationName) => {
    const startTime = performance.now();
    
    try {
      const result = await operation();
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      console.log(`⚡ ${componentName} - ${operationName} tomó ${duration.toFixed(2)}ms`);
      
      return result;
    } catch (error) {
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      console.error(`❌ ${componentName} - ${operationName} falló después de ${duration.toFixed(2)}ms:`, error);
      
      throw error;
    }
  }, [componentName]);

  // Configurar observer de performance
  useEffect(() => {
    if (PerformanceObserver && PerformanceObserver.supportedEntryTypes?.includes('measure')) {
      observer.current = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          console.log(`📊 ${componentName} - ${entry.name}: ${entry.duration.toFixed(2)}ms`);
        });
      });
      
      observer.current.observe({ entryTypes: ['measure'] });
    }

    return () => {
      if (observer.current) {
        observer.current.disconnect();
      }
    };
  }, [componentName]);

  return {
    renderCount: renderCount.current,
    lastRenderTime: lastRenderTime.current,
    measureRender,
    measureAsync,
  };
};
```

#### Componente de Profiling
```javascript
// Componente wrapper para profiling
import React, { Profiler } from 'react';
import { View, Text } from 'react-native';

const ProfilerWrapper = ({ id, children, onRender }) => {
  const handleProfilerRender = useCallback((
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime
  ) => {
    const metrics = {
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime,
      timestamp: Date.now(),
    };

    console.log('📊 Profiler metrics:', metrics);
    
    // Enviar métricas a servicio de analytics
    if (onRender) {
      onRender(metrics);
    }
    
    // Alertar si el render es muy lento
    if (actualDuration > 16) { // Más de 16ms (60 FPS)
      console.warn(`⚠️ ${id} render lento: ${actualDuration.toFixed(2)}ms`);
    }
  }, [onRender]);

  return (
    <Profiler id={id} onRender={handleProfilerRender}>
      {children}
    </Profiler>
  );
};

// HOC para profiling automático
export const withProfiling = (WrappedComponent, componentName) => {
  return React.forwardRef((props, ref) => (
    <ProfilerWrapper id={componentName}>
      <WrappedComponent {...props} ref={ref} />
    </ProfilerWrapper>
  ));
};

export default ProfilerWrapper;
```

### 6. Optimización de Bundle y Code Splitting

#### Lazy Loading de Componentes
```javascript
// Lazy loading de pantallas
import React, { Suspense, lazy } from 'react';
import { View, ActivityIndicator, Text } from 'react-native';

// Componente de loading
const LoadingScreen = () => (
  <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
    <ActivityIndicator size="large" color="#0000ff" />
    <Text style={{ marginTop: 10 }}>Cargando...</Text>
  </View>
);

// Lazy loading de pantallas
const HomeScreen = lazy(() => import('../screens/HomeScreen'));
const ProfileScreen = lazy(() => import('../screens/ProfileScreen'));
const SettingsScreen = lazy(() => import('../screens/SettingsScreen'));
const AdminScreen = lazy(() => import('../screens/AdminScreen'));

// Navegador con lazy loading
const AppNavigator = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home">
          {props => (
            <Suspense fallback={<LoadingScreen />}>
              <HomeScreen {...props} />
            </Suspense>
          )}
        </Stack.Screen>
        
        <Stack.Screen name="Profile">
          {props => (
            <Suspense fallback={<LoadingScreen />}>
              <ProfileScreen {...props} />
            </Suspense>
          )}
        </Stack.Screen>
        
        <Stack.Screen name="Settings">
          {props => (
            <Suspense fallback={<LoadingScreen />}>
              <SettingsScreen {...props} />
            </Suspense>
          )}
        </Stack.Screen>
        
        <Stack.Screen name="Admin">
          {props => (
            <Suspense fallback={<LoadingScreen />}>
              <AdminScreen {...props} />
            </Suspense>
          )}
        </Stack.Screen>
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default AppNavigator;
```

#### Lazy Loading de Funciones
```javascript
// Lazy loading de funciones pesadas
import { useCallback, useMemo } from 'react';

const useLazyFunction = (importFunction, deps = []) => {
  const [functionModule, setFunctionModule] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  const loadFunction = useCallback(async () => {
    if (functionModule) return functionModule;
    
    setIsLoading(true);
    try {
      const module = await importFunction();
      setFunctionModule(module);
      return module;
    } catch (error) {
      console.error('Error loading function:', error);
      throw error;
    } finally {
      setIsLoading(false);
    }
  }, [importFunction, functionModule]);

  const executeFunction = useCallback(async (...args) => {
    const module = await loadFunction();
    return module.default(...args);
  }, [loadFunction]);

  return {
    execute: executeFunction,
    isLoading,
    isLoaded: !!functionModule,
  };
};

// Ejemplo de uso
const UserList = () => {
  const { execute: processUsers, isLoading } = useLazyFunction(
    () => import('../utils/userProcessor')
  );

  const handleProcessUsers = useCallback(async (users) => {
    try {
      const result = await processUsers(users);
      console.log('Usuarios procesados:', result);
    } catch (error) {
      console.error('Error procesando usuarios:', error);
    }
  }, [processUsers]);

  return (
    <View>
      <TouchableOpacity
        onPress={() => handleProcessUsers(userList)}
        disabled={isLoading}
      >
        <Text>
          {isLoading ? 'Procesando...' : 'Procesar Usuarios'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};
```

## 🧪 Ejercicios Prácticos

### Ejercicio 1: Optimización de Lista Virtualizada
Implementa una lista virtualizada con las siguientes características:

```javascript
// La lista debe incluir:
// - Virtualización con FlatList optimizada
// - Lazy loading de imágenes
// - Infinite scroll
// - Pull to refresh
// - Búsqueda en tiempo real
// - Filtros avanzados
```

### Ejercicio 2: Sistema de Caché de Imágenes
Crea un sistema de caché de imágenes:

```javascript
// El sistema debe incluir:
// - Caché en memoria y disco
// - Lazy loading automático
// - Prefetching inteligente
// - Compresión de imágenes
// - Limpieza automática de caché
```

### Ejercicio 3: Profiler de Performance
Implementa un sistema de profiling completo:

```javascript
// El sistema debe incluir:
// - Métricas de render por componente
// - Análisis de re-renders innecesarios
// - Alertas de performance
// - Exportación de métricas
// - Dashboard de performance
```

## 📝 Resumen de la Clase

### **Conceptos Clave Aprendidos:**
1. **Optimización de Re-renders**: React.memo, useMemo, useCallback para evitar renders innecesarios
2. **Virtualización**: FlatList optimizada para listas grandes con configuraciones de performance
3. **Optimización de Imágenes**: Lazy loading, caché y compresión para mejorar tiempos de carga
4. **Profiling**: Herramientas para medir y analizar performance de componentes
5. **Code Splitting**: Lazy loading de componentes y funciones para reducir bundle size

### **Habilidades Desarrolladas:**
- Implementar técnicas de optimización en React Native
- Crear listas virtualizadas eficientes para grandes datasets
- Optimizar imágenes y assets para mejor performance
- Implementar profiling y métricas de performance
- Aplicar lazy loading y code splitting

### **Próximos Pasos:**
Con esto hemos completado el **Módulo 6: APIs y Networking**. En el siguiente módulo aprenderemos sobre **Almacenamiento Local**, que nos permitirá:
- Implementar diferentes estrategias de almacenamiento local
- Manejar bases de datos SQLite y NoSQL
- Implementar sincronización offline/online
- Gestionar migraciones y versionado de datos
- Crear sistemas de backup y restauración

---

**💡 Consejo**: La optimización de performance es un proceso iterativo. Siempre mide antes de optimizar, optimiza lo que realmente importa, y vuelve a medir. Las herramientas de profiling te ayudarán a identificar los cuellos de botella reales en tu aplicación.
