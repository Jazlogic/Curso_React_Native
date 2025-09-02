# Clase 5: Patrones Avanzados y Optimización

## Objetivos de la Clase
- Implementar patrones avanzados de diseño
- Optimizar performance de componentes
- Gestionar estado complejo en sistemas de componentes
- Implementar técnicas de optimización avanzadas

## 1. Patrones Avanzados de Diseño

### Higher-Order Components (HOC)
```typescript
// patterns/HOC/withTheme.tsx
import React, { ComponentType } from 'react';
import { useTheme } from '../../hooks/useTheme';

export interface WithThemeProps {
  theme: any;
  isDark: boolean;
  toggleTheme: () => void;
}

export function withTheme<P extends object>(
  Component: ComponentType<P & WithThemeProps>
): ComponentType<P> {
  const WrappedComponent = (props: P) => {
    const theme = useTheme();
    
    return <Component {...props} {...theme} />;
  };
  
  WrappedComponent.displayName = `withTheme(${Component.displayName || Component.name})`;
  
  return WrappedComponent;
}

// Uso
const ThemedButton = withTheme(Button);
```

### Render Props Pattern
```typescript
// patterns/RenderProps/DataProvider.tsx
import React, { useState, useEffect } from 'react';

interface DataProviderProps<T> {
  url: string;
  children: (data: {
    data: T | null;
    loading: boolean;
    error: string | null;
    refetch: () => void;
  }) => React.ReactNode;
}

export function DataProvider<T>({ url, children }: DataProviderProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(url);
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  };
  
  useEffect(() => {
    fetchData();
  }, [url]);
  
  return <>{children({ data, loading, error, refetch: fetchData })}</>;
}

// Uso
<DataProvider url="/api/users">
  {({ data, loading, error, refetch }) => (
    <View>
      {loading && <Text>Loading...</Text>}
      {error && <Text>Error: {error}</Text>}
      {data && <UserList users={data} />}
      <Button title="Refresh" onPress={refetch} />
    </View>
  )}
</DataProvider>
```

### Compound Components con Context
```typescript
// patterns/Compound/Accordion.tsx
import React, { createContext, useContext, useState } from 'react';
import { View, TouchableOpacity, Text } from 'react-native';

interface AccordionContextType {
  isOpen: boolean;
  toggle: () => void;
}

const AccordionContext = createContext<AccordionContextType | undefined>(undefined);

const useAccordionContext = () => {
  const context = useContext(AccordionContext);
  if (!context) {
    throw new Error('Accordion components must be used within Accordion');
  }
  return context;
};

interface AccordionProps {
  children: React.ReactNode;
  defaultOpen?: boolean;
}

export const Accordion: React.FC<AccordionProps> = ({ children, defaultOpen = false }) => {
  const [isOpen, setIsOpen] = useState(defaultOpen);
  
  const toggle = () => setIsOpen(!isOpen);
  
  return (
    <AccordionContext.Provider value={{ isOpen, toggle }}>
      <View>{children}</View>
    </AccordionContext.Provider>
  );
};

export const AccordionHeader: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { toggle } = useAccordionContext();
  
  return (
    <TouchableOpacity onPress={toggle}>
      {children}
    </TouchableOpacity>
  );
};

export const AccordionContent: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { isOpen } = useAccordionContext();
  
  if (!isOpen) return null;
  
  return <View>{children}</View>;
};
```

### Provider Pattern
```typescript
// patterns/Provider/FeatureFlagProvider.tsx
import React, { createContext, useContext, useState, useEffect } from 'react';

interface FeatureFlag {
  name: string;
  enabled: boolean;
  description?: string;
}

interface FeatureFlagContextType {
  flags: Record<string, boolean>;
  isEnabled: (flagName: string) => boolean;
  refresh: () => Promise<void>;
}

const FeatureFlagContext = createContext<FeatureFlagContextType | undefined>(undefined);

export const useFeatureFlag = () => {
  const context = useContext(FeatureFlagContext);
  if (!context) {
    throw new Error('useFeatureFlag must be used within FeatureFlagProvider');
  }
  return context;
};

interface FeatureFlagProviderProps {
  children: React.ReactNode;
  apiUrl: string;
}

export const FeatureFlagProvider: React.FC<FeatureFlagProviderProps> = ({
  children,
  apiUrl,
}) => {
  const [flags, setFlags] = useState<Record<string, boolean>>({});
  
  const fetchFlags = async () => {
    try {
      const response = await fetch(apiUrl);
      const featureFlags: FeatureFlag[] = await response.json();
      
      const flagsMap = featureFlags.reduce((acc, flag) => {
        acc[flag.name] = flag.enabled;
        return acc;
      }, {} as Record<string, boolean>);
      
      setFlags(flagsMap);
    } catch (error) {
      console.error('Failed to fetch feature flags:', error);
    }
  };
  
  useEffect(() => {
    fetchFlags();
  }, [apiUrl]);
  
  const isEnabled = (flagName: string): boolean => {
    return flags[flagName] || false;
  };
  
  return (
    <FeatureFlagContext.Provider value={{ flags, isEnabled, refresh: fetchFlags }}>
      {children}
    </FeatureFlagContext.Provider>
  );
};
```

## 2. Optimización de Performance

### Memoización Avanzada
```typescript
// optimization/Memoization.tsx
import React, { memo, useMemo, useCallback, useState } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

// Memoización de componentes
const ExpensiveComponent = memo<{ data: any[]; onItemPress: (id: string) => void }>(
  ({ data, onItemPress }) => {
    const processedData = useMemo(() => {
      return data.map(item => ({
        ...item,
        processed: item.value * 2,
      }));
    }, [data]);
    
    return (
      <View>
        {processedData.map(item => (
          <TouchableOpacity
            key={item.id}
            onPress={() => onItemPress(item.id)}
          >
            <Text>{item.name}: {item.processed}</Text>
          </TouchableOpacity>
        ))}
      </View>
    );
  }
);

// Componente padre optimizado
export const OptimizedParent: React.FC = () => {
  const [data, setData] = useState([]);
  const [selectedId, setSelectedId] = useState<string | null>(null);
  
  // Memoización de callbacks
  const handleItemPress = useCallback((id: string) => {
    setSelectedId(id);
  }, []);
  
  // Memoización de datos derivados
  const filteredData = useMemo(() => {
    return data.filter(item => item.active);
  }, [data]);
  
  return (
    <View>
      <ExpensiveComponent
        data={filteredData}
        onItemPress={handleItemPress}
      />
    </View>
  );
};
```

### Virtualización
```typescript
// optimization/VirtualizedList.tsx
import React, { memo } from 'react';
import { FlatList, ListRenderItem, ViewToken } from 'react-native';

interface VirtualizedListProps<T> {
  data: T[];
  renderItem: ListRenderItem<T>;
  itemHeight: number;
  onViewableItemsChanged?: (info: { viewableItems: ViewToken[] }) => void;
}

export const VirtualizedList = memo(<T,>({
  data,
  renderItem,
  itemHeight,
  onViewableItemsChanged,
}: VirtualizedListProps<T>) => {
  const getItemLayout = useCallback(
    (_: any, index: number) => ({
      length: itemHeight,
      offset: itemHeight * index,
      index,
    }),
    [itemHeight]
  );
  
  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      getItemLayout={getItemLayout}
      onViewableItemsChanged={onViewableItemsChanged}
      viewabilityConfig={{
        itemVisiblePercentThreshold: 50,
      }}
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={10}
    />
  );
});
```

### Lazy Loading
```typescript
// optimization/LazyLoading.tsx
import React, { lazy, Suspense, useState, useEffect } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';

// Componente lazy
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Hook para lazy loading
export const useLazyLoading = (threshold: number = 0.1) => {
  const [isVisible, setIsVisible] = useState(false);
  const [ref, setRef] = useState<View | null>(null);
  
  useEffect(() => {
    if (!ref) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold }
    );
    
    observer.observe(ref);
    
    return () => observer.disconnect();
  }, [ref, threshold]);
  
  return { ref: setRef, isVisible };
};

// Componente con lazy loading
export const LazyLoadedComponent: React.FC = () => {
  const { ref, isVisible } = useLazyLoading();
  
  return (
    <View ref={ref}>
      {isVisible ? (
        <Suspense fallback={<ActivityIndicator />}>
          <HeavyComponent />
        </Suspense>
      ) : (
        <View style={{ height: 200 }}>
          <Text>Loading...</Text>
        </View>
      )}
    </View>
  );
};
```

## 3. Gestión de Estado Complejo

### State Machine
```typescript
// state/StateMachine.ts
export type State = 'idle' | 'loading' | 'success' | 'error';
export type Event = 'FETCH' | 'SUCCESS' | 'ERROR' | 'RESET';

interface StateMachineConfig {
  [key: string]: {
    [key: string]: State;
  };
}

const stateMachine: StateMachineConfig = {
  idle: {
    FETCH: 'loading',
  },
  loading: {
    SUCCESS: 'success',
    ERROR: 'error',
  },
  success: {
    FETCH: 'loading',
    RESET: 'idle',
  },
  error: {
    FETCH: 'loading',
    RESET: 'idle',
  },
};

export class StateMachine {
  private currentState: State = 'idle';
  private listeners: Set<(state: State) => void> = new Set();
  
  getState(): State {
    return this.currentState;
  }
  
  transition(event: Event): boolean {
    const nextState = stateMachine[this.currentState]?.[event];
    
    if (nextState) {
      this.currentState = nextState;
      this.notifyListeners();
      return true;
    }
    
    return false;
  }
  
  subscribe(listener: (state: State) => void) {
    this.listeners.add(listener);
    
    return () => {
      this.listeners.delete(listener);
    };
  }
  
  private notifyListeners() {
    this.listeners.forEach(listener => listener(this.currentState));
  }
}
```

### Custom Hook para State Machine
```typescript
// hooks/useStateMachine.ts
import { useState, useEffect, useCallback } from 'react';
import { StateMachine, State, Event } from '../state/StateMachine';

export const useStateMachine = (initialState: State = 'idle') => {
  const [stateMachine] = useState(() => new StateMachine());
  const [state, setState] = useState<State>(initialState);
  
  useEffect(() => {
    const unsubscribe = stateMachine.subscribe(setState);
    return unsubscribe;
  }, [stateMachine]);
  
  const transition = useCallback((event: Event) => {
    return stateMachine.transition(event);
  }, [stateMachine]);
  
  return {
    state,
    transition,
    isIdle: state === 'idle',
    isLoading: state === 'loading',
    isSuccess: state === 'success',
    isError: state === 'error',
  };
};
```

### Context con Reducer
```typescript
// state/AppState.tsx
import React, { createContext, useContext, useReducer, ReactNode } from 'react';

interface AppState {
  user: User | null;
  theme: 'light' | 'dark';
  notifications: Notification[];
  loading: boolean;
}

type AppAction =
  | { type: 'SET_USER'; payload: User }
  | { type: 'SET_THEME'; payload: 'light' | 'dark' }
  | { type: 'ADD_NOTIFICATION'; payload: Notification }
  | { type: 'REMOVE_NOTIFICATION'; payload: string }
  | { type: 'SET_LOADING'; payload: boolean };

const initialState: AppState = {
  user: null,
  theme: 'light',
  notifications: [],
  loading: false,
};

const appReducer = (state: AppState, action: AppAction): AppState => {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    case 'ADD_NOTIFICATION':
      return { ...state, notifications: [...state.notifications, action.payload] };
    case 'REMOVE_NOTIFICATION':
      return {
        ...state,
        notifications: state.notifications.filter(n => n.id !== action.payload),
      };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    default:
      return state;
  }
};

const AppStateContext = createContext<{
  state: AppState;
  dispatch: React.Dispatch<AppAction>;
} | undefined>(undefined);

export const AppStateProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  return (
    <AppStateContext.Provider value={{ state, dispatch }}>
      {children}
    </AppStateContext.Provider>
  );
};

export const useAppState = () => {
  const context = useContext(AppStateContext);
  if (!context) {
    throw new Error('useAppState must be used within AppStateProvider');
  }
  return context;
};
```

## 4. Optimización de Bundle

### Code Splitting
```typescript
// optimization/CodeSplitting.tsx
import React, { lazy, Suspense } from 'react';
import { View, ActivityIndicator } from 'react-native';

// Lazy loading de componentes
const HomeScreen = lazy(() => import('../screens/HomeScreen'));
const ProfileScreen = lazy(() => import('../screens/ProfileScreen'));
const SettingsScreen = lazy(() => import('../screens/SettingsScreen'));

// Componente de loading
const LoadingFallback: React.FC = () => (
  <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
    <ActivityIndicator size="large" />
  </View>
);

// Wrapper para lazy components
export const LazyScreen: React.FC<{ screen: string }> = ({ screen }) => {
  const renderScreen = () => {
    switch (screen) {
      case 'home':
        return <HomeScreen />;
      case 'profile':
        return <ProfileScreen />;
      case 'settings':
        return <SettingsScreen />;
      default:
        return <HomeScreen />;
    }
  };
  
  return (
    <Suspense fallback={<LoadingFallback />}>
      {renderScreen()}
    </Suspense>
  );
};
```

### Tree Shaking Optimization
```typescript
// optimization/TreeShaking.ts
// Exportaciones específicas para tree shaking
export { Button } from './components/Button';
export { Input } from './components/Input';
export { Card } from './components/Card';

// Evitar exportaciones por defecto que incluyan todo
// ❌ Malo
export * from './components';

// ✅ Bueno
export { Button } from './components/Button';
export { Input } from './components/Input';

// Separar utilidades
export { formatDate } from './utils/date';
export { validateEmail } from './utils/validation';
export { debounce } from './utils/debounce';
```

## 5. Optimización de Rendering

### ShouldComponentUpdate Personalizado
```typescript
// optimization/ShouldUpdate.tsx
import React, { memo, useMemo } from 'react';
import { View, Text } from 'react-native';

interface ExpensiveComponentProps {
  data: any[];
  filter: string;
  sortBy: string;
}

// Memoización con comparación personalizada
export const ExpensiveComponent = memo<ExpensiveComponentProps>(
  ({ data, filter, sortBy }) => {
    const processedData = useMemo(() => {
      return data
        .filter(item => item.name.includes(filter))
        .sort((a, b) => a[sortBy] - b[sortBy]);
    }, [data, filter, sortBy]);
    
    return (
      <View>
        {processedData.map(item => (
          <Text key={item.id}>{item.name}</Text>
        ))}
      </View>
    );
  },
  (prevProps, nextProps) => {
    // Comparación personalizada para optimización
    return (
      prevProps.data.length === nextProps.data.length &&
      prevProps.filter === nextProps.filter &&
      prevProps.sortBy === nextProps.sortBy
    );
  }
);
```

### Optimización de Listas
```typescript
// optimization/OptimizedList.tsx
import React, { memo, useCallback, useMemo } from 'react';
import { FlatList, ListRenderItem } from 'react-native';

interface OptimizedListProps<T> {
  data: T[];
  renderItem: ListRenderItem<T>;
  keyExtractor: (item: T) => string;
  onEndReached?: () => void;
  onRefresh?: () => void;
  refreshing?: boolean;
}

export const OptimizedList = memo(<T,>({
  data,
  renderItem,
  keyExtractor,
  onEndReached,
  onRefresh,
  refreshing = false,
}: OptimizedListProps<T>) => {
  // Memoización del renderItem
  const memoizedRenderItem = useCallback(renderItem, [renderItem]);
  
  // Memoización del keyExtractor
  const memoizedKeyExtractor = useCallback(keyExtractor, [keyExtractor]);
  
  // Memoización de callbacks
  const handleEndReached = useCallback(() => {
    onEndReached?.();
  }, [onEndReached]);
  
  const handleRefresh = useCallback(() => {
    onRefresh?.();
  }, [onRefresh]);
  
  // Configuración optimizada
  const listConfig = useMemo(() => ({
    removeClippedSubviews: true,
    maxToRenderPerBatch: 10,
    windowSize: 10,
    initialNumToRender: 10,
    updateCellsBatchingPeriod: 50,
    getItemLayout: undefined, // Se puede optimizar si se conoce la altura
  }), []);
  
  return (
    <FlatList
      data={data}
      renderItem={memoizedRenderItem}
      keyExtractor={memoizedKeyExtractor}
      onEndReached={handleEndReached}
      onRefresh={handleRefresh}
      refreshing={refreshing}
      {...listConfig}
    />
  );
});
```

## 6. Profiling y Debugging

### Performance Profiler
```typescript
// optimization/PerformanceProfiler.tsx
import React, { Profiler, ReactNode } from 'react';

interface PerformanceProfilerProps {
  children: ReactNode;
  id: string;
  onRender?: (id: string, phase: string, actualDuration: number) => void;
}

export const PerformanceProfiler: React.FC<PerformanceProfilerProps> = ({
  children,
  id,
  onRender,
}) => {
  const handleRender = (
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number,
    baseDuration: number,
    startTime: number,
    commitTime: number
  ) => {
    console.log(`Performance Profiler [${id}]:`, {
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime,
    });
    
    onRender?.(id, phase, actualDuration);
  };
  
  return (
    <Profiler id={id} onRender={handleRender}>
      {children}
    </Profiler>
  );
};
```

### Memory Leak Detection
```typescript
// optimization/MemoryLeakDetector.ts
export class MemoryLeakDetector {
  private static instance: MemoryLeakDetector;
  private listeners: Map<string, Set<() => void>> = new Map();
  
  static getInstance(): MemoryLeakDetector {
    if (!MemoryLeakDetector.instance) {
      MemoryLeakDetector.instance = new MemoryLeakDetector();
    }
    return MemoryLeakDetector.instance;
  }
  
  registerListener(componentId: string, cleanup: () => void) {
    if (!this.listeners.has(componentId)) {
      this.listeners.set(componentId, new Set());
    }
    this.listeners.get(componentId)!.add(cleanup);
  }
  
  unregisterListener(componentId: string, cleanup: () => void) {
    const componentListeners = this.listeners.get(componentId);
    if (componentListeners) {
      componentListeners.delete(cleanup);
      if (componentListeners.size === 0) {
        this.listeners.delete(componentId);
      }
    }
  }
  
  getStats() {
    return {
      totalComponents: this.listeners.size,
      totalListeners: Array.from(this.listeners.values())
        .reduce((sum, set) => sum + set.size, 0),
    };
  }
}
```

## 7. Optimización de Imágenes

### Image Optimization
```typescript
// optimization/ImageOptimization.tsx
import React, { memo, useState, useCallback } from 'react';
import { Image, ImageProps, View, ActivityIndicator } from 'react-native';

interface OptimizedImageProps extends Omit<ImageProps, 'source'> {
  source: { uri: string };
  placeholder?: string;
  fallback?: string;
  quality?: number;
  width?: number;
  height?: number;
}

export const OptimizedImage = memo<OptimizedImageProps>(({
  source,
  placeholder,
  fallback,
  quality = 80,
  width,
  height,
  style,
  ...props
}) => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);
  
  const handleLoad = useCallback(() => {
    setLoading(false);
    setError(false);
  }, []);
  
  const handleError = useCallback(() => {
    setLoading(false);
    setError(true);
  }, []);
  
  const getOptimizedUri = useCallback((uri: string) => {
    const url = new URL(uri);
    
    if (quality) {
      url.searchParams.set('quality', quality.toString());
    }
    
    if (width) {
      url.searchParams.set('width', width.toString());
    }
    
    if (height) {
      url.searchParams.set('height', height.toString());
    }
    
    return url.toString();
  }, [quality, width, height]);
  
  const getImageSource = () => {
    if (error && fallback) {
      return { uri: fallback };
    }
    
    if (loading && placeholder) {
      return { uri: placeholder };
    }
    
    return { uri: getOptimizedUri(source.uri) };
  };
  
  return (
    <View style={style}>
      <Image
        source={getImageSource()}
        onLoad={handleLoad}
        onError={handleError}
        style={style}
        {...props}
      />
      {loading && (
        <View style={{ position: 'absolute', top: 0, left: 0, right: 0, bottom: 0, justifyContent: 'center', alignItems: 'center' }}>
          <ActivityIndicator />
        </View>
      )}
    </View>
  );
});
```

## 8. Mejores Prácticas de Optimización

### Reglas de Optimización
1. **Medir antes de optimizar**: Usar herramientas de profiling
2. **Optimizar gradualmente**: No optimizar todo de una vez
3. **Priorizar bottlenecks**: Enfocarse en los problemas más críticos
4. **Mantener legibilidad**: No sacrificar legibilidad por performance
5. **Documentar optimizaciones**: Explicar por qué se hizo cada optimización

### Checklist de Performance
- [ ] Componentes memoizados cuando es necesario
- [ ] Callbacks memoizados con useCallback
- [ ] Datos derivados memoizados con useMemo
- [ ] Listas virtualizadas para grandes datasets
- [ ] Imágenes optimizadas y lazy loaded
- [ ] Bundle size optimizado
- [ ] Memory leaks detectados y corregidos
- [ ] Tests de performance implementados

## Conclusión

Los patrones avanzados y la optimización son fundamentales para crear sistemas de componentes de alta calidad. Permiten:

- **Performance superior** en aplicaciones complejas
- **Escalabilidad** del sistema
- **Mantenibilidad** del código
- **Experiencia de usuario** optimizada

## Tarea

1. Implementar al menos 3 patrones avanzados de diseño
2. Optimizar performance de componentes existentes
3. Implementar un sistema de state machine
4. Crear herramientas de profiling y debugging
5. Optimizar bundle size y implementar code splitting

## Enlaces Útiles

- [React Performance](https://reactjs.org/docs/optimizing-performance.html)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [Flipper](https://fbflipper.com/)
- [React DevTools](https://reactjs.org/blog/2019/08/15/new-react-devtools.html)
