# 🧪 **Clase 3: Testing de Hooks y Lógica** - React Native

## 📋 **Objetivos de la Clase**
- Aprender a testear hooks personalizados
- Testing de lógica de negocio compleja
- Implementar mocks para hooks nativos
- Testing de efectos secundarios y cleanup

## ⏱️ **Duración**
**2 horas**

## 🔧 **Configuración para Testing de Hooks**

### Instalación de Dependencias
```bash
npm install --save-dev @testing-library/react-hooks
npm install --save-dev react-test-renderer
```

### Configuración de Jest para Hooks
```javascript
// jest.setup.js
import '@testing-library/jest-native/extend-expect';

// Mock de react-native
jest.mock('react-native', () => {
  const RN = jest.requireActual('react-native');
  
  return {
    ...RN,
    // Mock de APIs nativas
    Alert: {
      alert: jest.fn(),
    },
    Platform: {
      OS: 'ios',
      select: jest.fn((obj) => obj.ios),
    },
    // Mock de AsyncStorage
    AsyncStorage: {
      getItem: jest.fn(),
      setItem: jest.fn(),
      removeItem: jest.fn(),
      clear: jest.fn(),
    },
  };
});

// Mock de react-native-vector-icons
jest.mock('react-native-vector-icons/MaterialIcons', () => 'Icon');
```

## 📚 **Contenido Teórico**

### 1. **Testing de Hooks Personalizados**

#### **Hook de Estado Local**
```typescript
// src/hooks/useCounter.ts
import { useState, useCallback } from 'react';

interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  setCount: (value: number) => void;
}

export const useCounter = (initialValue: number = 0): UseCounterReturn => {
  const [count, setCountState] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCountState(prev => prev + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCountState(prev => prev - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCountState(initialValue);
  }, [initialValue]);
  
  const setCount = useCallback((value: number) => {
    setCountState(value);
  }, []);
  
  return {
    count,
    increment,
    decrement,
    reset,
    setCount
  };
};
```

#### **Tests para el Hook useCounter**
```typescript
// src/hooks/__tests__/useCounter.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from '../useCounter';

describe('useCounter Hook', () => {
  test('debe inicializar con valor por defecto 0', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });
  
  test('debe inicializar con valor personalizado', () => {
    const { result } = renderHook(() => useCounter(5));
    
    expect(result.current.count).toBe(5);
  });
  
  test('debe incrementar contador', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  test('debe decrementar contador', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
  
  test('debe resetear contador al valor inicial', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
  
  test('debe establecer contador a valor específico', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.setCount(25);
    });
    
    expect(result.current.count).toBe(25);
  });
  
  test('debe mantener referencias estables de funciones', () => {
    const { result, rerender } = renderHook(() => useCounter());
    
    const firstIncrement = result.current.increment;
    const firstDecrement = result.current.decrement;
    const firstReset = result.current.reset;
    const firstSetCount = result.current.setCount;
    
    rerender();
    
    expect(result.current.increment).toBe(firstIncrement);
    expect(result.current.decrement).toBe(firstDecrement);
    expect(result.current.reset).toBe(firstReset);
    expect(result.current.setCount).toBe(firstSetCount);
  });
});
```

### 2. **Testing de Hooks con AsyncStorage**

#### **Hook de Persistencia de Datos**
```typescript
// src/hooks/useLocalStorage.ts
import { useState, useEffect, useCallback } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void, () => void] {
  const [storedValue, setStoredValue] = useState<T>(initialValue);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    const loadStoredValue = async () => {
      try {
        const item = await AsyncStorage.getItem(key);
        if (item !== null) {
          setStoredValue(JSON.parse(item));
        }
      } catch (error) {
        console.error('Error loading from AsyncStorage:', error);
      } finally {
        setIsLoading(false);
      }
    };
    
    loadStoredValue();
  }, [key]);
  
  const setValue = useCallback(async (value: T) => {
    try {
      setStoredValue(value);
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to AsyncStorage:', error);
    }
  }, [key]);
  
  const removeValue = useCallback(async () => {
    try {
      setStoredValue(initialValue);
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Error removing from AsyncStorage:', error);
    }
  }, [key, initialValue]);
  
  return [storedValue, setValue, removeValue];
}
```

#### **Tests para el Hook useLocalStorage**
```typescript
// src/hooks/__tests__/useLocalStorage.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useLocalStorage } from '../useLocalStorage';

// Mock de AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => ({
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
}));

describe('useLocalStorage Hook', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  test('debe inicializar con valor por defecto', async () => {
    (AsyncStorage.getItem as jest.Mock).mockResolvedValue(null);
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useLocalStorage('test-key', 'default-value')
    );
    
    expect(result.current[0]).toBe('default-value');
    
    await waitForNextUpdate();
    
    expect(result.current[0]).toBe('default-value');
  });
  
  test('debe cargar valor existente de AsyncStorage', async () => {
    const storedValue = 'stored-value';
    (AsyncStorage.getItem as jest.Mock).mockResolvedValue(JSON.stringify(storedValue));
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useLocalStorage('test-key', 'default-value')
    );
    
    expect(result.current[0]).toBe('default-value');
    
    await waitForNextUpdate();
    
    expect(result.current[0]).toBe(storedValue);
    expect(AsyncStorage.getItem).toHaveBeenCalledWith('test-key');
  });
  
  test('debe guardar valor en AsyncStorage', async () => {
    (AsyncStorage.getItem as jest.Mock).mockResolvedValue(null);
    (AsyncStorage.setItem as jest.Mock).mockResolvedValue(undefined);
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useLocalStorage('test-key', 'default-value')
    );
    
    await waitForNextUpdate();
    
    act(() => {
      result.current[1]('new-value');
    });
    
    expect(result.current[0]).toBe('new-value');
    expect(AsyncStorage.setItem).toHaveBeenCalledWith(
      'test-key',
      JSON.stringify('new-value')
    );
  });
  
  test('debe remover valor de AsyncStorage', async () => {
    (AsyncStorage.getItem as jest.Mock).mockResolvedValue(null);
    (AsyncStorage.removeItem as jest.Mock).mockResolvedValue(undefined);
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useLocalStorage('test-key', 'default-value')
    );
    
    await waitForNextUpdate();
    
    act(() => {
      result.current[2]();
    });
    
    expect(result.current[0]).toBe('default-value');
    expect(AsyncStorage.removeItem).toHaveBeenCalledWith('test-key');
  });
  
  test('debe manejar errores de AsyncStorage', async () => {
    const consoleErrorSpy = jest.spyOn(console, 'error').mockImplementation();
    (AsyncStorage.getItem as jest.Mock).mockRejectedValue(new Error('Storage error'));
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useLocalStorage('test-key', 'default-value')
    );
    
    await waitForNextUpdate();
    
    expect(result.current[0]).toBe('default-value');
    expect(consoleErrorSpy).toHaveBeenCalledWith(
      'Error loading from AsyncStorage:',
      expect.any(Error)
    );
    
    consoleErrorSpy.mockRestore();
  });
});
```

### 3. **Testing de Hooks con Lógica de Negocio**

#### **Hook de Gestión de Formulario**
```typescript
// src/hooks/useForm.ts
import { useState, useCallback, useMemo } from 'react';

interface ValidationRule {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  custom?: (value: any) => boolean | string;
}

interface FormField {
  value: any;
  error?: string;
  touched: boolean;
}

interface UseFormReturn<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isValid: boolean;
  setFieldValue: (field: keyof T, value: any) => void;
  setFieldTouched: (field: keyof T) => void;
  handleSubmit: (onSubmit: (values: T) => void) => void;
  resetForm: () => void;
}

export function useForm<T extends Record<string, any>>(
  initialValues: T,
  validationRules?: Partial<Record<keyof T, ValidationRule>>
): UseFormReturn<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  
  const validateField = useCallback((field: keyof T, value: any): string | undefined => {
    if (!validationRules || !validationRules[field]) return undefined;
    
    const rule = validationRules[field]!;
    
    if (rule.required && (!value || value.toString().trim() === '')) {
      return 'Este campo es requerido';
    }
    
    if (rule.minLength && value && value.toString().length < rule.minLength) {
      return `Mínimo ${rule.minLength} caracteres`;
    }
    
    if (rule.maxLength && value && value.toString().length > rule.maxLength) {
      return `Máximo ${rule.maxLength} caracteres`;
    }
    
    if (rule.pattern && value && !rule.pattern.test(value.toString())) {
      return 'Formato inválido';
    }
    
    if (rule.custom) {
      const customResult = rule.custom(value);
      if (typeof customResult === 'string') {
        return customResult;
      }
      if (!customResult) {
        return 'Valor inválido';
      }
    }
    
    return undefined;
  }, [validationRules]);
  
  const setFieldValue = useCallback((field: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [field]: value }));
    
    // Validar campo cuando cambia el valor
    const error = validateField(field, value);
    setErrors(prev => ({
      ...prev,
      [field]: error
    }));
  }, [validateField]);
  
  const setFieldTouched = useCallback((field: keyof T) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  }, []);
  
  const isValid = useMemo(() => {
    if (!validationRules) return true;
    
    return Object.keys(validationRules).every(field => {
      const error = validateField(field as keyof T, values[field as keyof T]);
      return !error;
    });
  }, [values, validationRules, validateField]);
  
  const handleSubmit = useCallback((onSubmit: (values: T) => void) => {
    // Marcar todos los campos como touched
    const allTouched = Object.keys(initialValues).reduce((acc, key) => {
      acc[key as keyof T] = true;
      return acc;
    }, {} as Partial<Record<keyof T, boolean>>);
    
    setTouched(allTouched);
    
    if (isValid) {
      onSubmit(values);
    }
  }, [isValid, values, initialValues]);
  
  const resetForm = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);
  
  return {
    values,
    errors,
    touched,
    isValid,
    setFieldValue,
    setFieldTouched,
    handleSubmit,
    resetForm
  };
}
```

#### **Tests para el Hook useForm**
```typescript
// src/hooks/__tests__/useForm.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import { useForm } from '../useForm';

interface TestForm {
  name: string;
  email: string;
  age: number;
}

describe('useForm Hook', () => {
  const initialValues: TestForm = {
    name: '',
    email: '',
    age: 0
  };
  
  const validationRules = {
    name: { required: true, minLength: 2 },
    email: { required: true, pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
    age: { required: true, custom: (value: number) => value >= 18 || 'Debe ser mayor de edad' }
  };
  
  test('debe inicializar con valores por defecto', () => {
    const { result } = renderHook(() => useForm(initialValues));
    
    expect(result.current.values).toEqual(initialValues);
    expect(result.current.errors).toEqual({});
    expect(result.current.touched).toEqual({});
    expect(result.current.isValid).toBe(true);
  });
  
  test('debe actualizar valor de campo', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldValue('name', 'Juan');
    });
    
    expect(result.current.values.name).toBe('Juan');
  });
  
  test('debe validar campo requerido', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldValue('name', '');
    });
    
    expect(result.current.errors.name).toBe('Este campo es requerido');
  });
  
  test('debe validar longitud mínima', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldValue('name', 'A');
    });
    
    expect(result.current.errors.name).toBe('Mínimo 2 caracteres');
  });
  
  test('debe validar patrón de email', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldValue('email', 'invalid-email');
    });
    
    expect(result.current.errors.email).toBe('Formato inválido');
  });
  
  test('debe validar email válido', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldValue('email', 'test@email.com');
    });
    
    expect(result.current.errors.email).toBeUndefined();
  });
  
  test('debe validar función personalizada', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldValue('age', 16);
    });
    
    expect(result.current.errors.age).toBe('Debe ser mayor de edad');
  });
  
  test('debe marcar campo como touched', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    act(() => {
      result.current.setFieldTouched('name');
    });
    
    expect(result.current.touched.name).toBe(true);
  });
  
  test('debe calcular isValid correctamente', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    // Formulario inválido inicialmente
    expect(result.current.isValid).toBe(false);
    
    // Hacer válido el formulario
    act(() => {
      result.current.setFieldValue('name', 'Juan Pérez');
      result.current.setFieldValue('email', 'juan@email.com');
      result.current.setFieldValue('age', 25);
    });
    
    expect(result.current.isValid).toBe(true);
  });
  
  test('debe manejar submit exitoso', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    const mockOnSubmit = jest.fn();
    
    // Hacer válido el formulario
    act(() => {
      result.current.setFieldValue('name', 'Juan Pérez');
      result.current.setFieldValue('email', 'juan@email.com');
      result.current.setFieldValue('age', 25);
    });
    
    act(() => {
      result.current.handleSubmit(mockOnSubmit);
    });
    
    expect(mockOnSubmit).toHaveBeenCalledWith({
      name: 'Juan Pérez',
      email: 'juan@email.com',
      age: 25
    });
  });
  
  test('debe marcar todos los campos como touched en submit', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    const mockOnSubmit = jest.fn();
    
    act(() => {
      result.current.handleSubmit(mockOnSubmit);
    });
    
    expect(result.current.touched.name).toBe(true);
    expect(result.current.touched.email).toBe(true);
    expect(result.current.touched.age).toBe(true);
  });
  
  test('debe resetear formulario', () => {
    const { result } = renderHook(() => useForm(initialValues, validationRules));
    
    // Cambiar valores y crear errores
    act(() => {
      result.current.setFieldValue('name', 'Juan');
      result.current.setFieldTouched('name');
    });
    
    expect(result.current.values.name).toBe('Juan');
    expect(result.current.touched.name).toBe(true);
    
    // Resetear
    act(() => {
      result.current.resetForm();
    });
    
    expect(result.current.values).toEqual(initialValues);
    expect(result.current.errors).toEqual({});
    expect(result.current.touched).toEqual({});
  });
});
```

### 4. **Testing de Hooks con Efectos Secundarios**

#### **Hook de Fetch de Datos**
```typescript
// src/hooks/useFetch.ts
import { useState, useEffect, useCallback } from 'react';

interface UseFetchReturn<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

export function useFetch<T>(
  url: string,
  options?: RequestInit
): UseFetchReturn<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error desconocido');
    } finally {
      setLoading(false);
    }
  }, [url, options]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return {
    data,
    loading,
    error,
    refetch: fetchData
  };
}
```

#### **Tests para el Hook useFetch**
```typescript
// src/hooks/__tests__/useFetch.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import { useFetch } from '../useFetch';

// Mock de fetch global
global.fetch = jest.fn();

describe('useFetch Hook', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  test('debe hacer fetch inicial automáticamente', async () => {
    const mockData = { id: 1, name: 'Test' };
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useFetch('https://api.test.com/data')
    );
    
    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBe(null);
    expect(result.current.error).toBe(null);
    
    await waitForNextUpdate();
    
    expect(result.current.loading).toBe(false);
    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBe(null);
  });
  
  test('debe manejar errores de fetch', async () => {
    (global.fetch as jest.Mock).mockRejectedValueOnce(new Error('Network error'));
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useFetch('https://api.test.com/data')
    );
    
    await waitForNextUpdate();
    
    expect(result.current.loading).toBe(false);
    expect(result.current.data).toBe(null);
    expect(result.current.error).toBe('Network error');
  });
  
  test('debe manejar respuestas HTTP no exitosas', async () => {
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: false,
      status: 404
    });
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useFetch('https://api.test.com/data')
    );
    
    await waitForNextUpdate();
    
    expect(result.current.loading).toBe(false);
    expect(result.current.data).toBe(null);
    expect(result.current.error).toBe('HTTP error! status: 404');
  });
  
  test('debe permitir refetch manual', async () => {
    const mockData1 = { id: 1, name: 'Test 1' };
    const mockData2 = { id: 2, name: 'Test 2' };
    
    (global.fetch as jest.Mock)
      .mockResolvedValueOnce({
        ok: true,
        json: async () => mockData1
      })
      .mockResolvedValueOnce({
        ok: true,
        json: async () => mockData2
      });
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useFetch('https://api.test.com/data')
    );
    
    await waitForNextUpdate();
    expect(result.current.data).toEqual(mockData1);
    
    act(() => {
      result.current.refetch();
    });
    
    expect(result.current.loading).toBe(true);
    
    await waitForNextUpdate();
    
    expect(result.current.data).toEqual(mockData2);
  });
  
  test('debe usar opciones de fetch personalizadas', async () => {
    const mockData = { id: 1, name: 'Test' };
    const options = {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ test: true })
    };
    
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });
    
    const { result, waitForNextUpdate } = renderHook(() =>
      useFetch('https://api.test.com/data', options)
    );
    
    await waitForNextUpdate();
    
    expect(global.fetch).toHaveBeenCalledWith(
      'https://api.test.com/data',
      options
    );
  });
});
```

## 🧪 **Ejercicios Prácticos**

### **Ejercicio 1: Hook de Validación de Formulario**
Crea un hook personalizado para validación de formularios:

```typescript
// Hook a implementar
export function useFormValidation<T>(
  initialValues: T,
  validationSchema: ValidationSchema<T>
): UseFormValidationReturn<T> {
  // Implementa:
  // - Validación en tiempo real
  // - Validación de campos específicos
  // - Validación de todo el formulario
  // - Manejo de errores personalizados
}

// Escribe tests que cubran:
// - Validación de campos individuales
// - Validación de formulario completo
// - Manejo de errores
// - Performance de validaciones
```

### **Ejercicio 2: Hook de Cache de Datos**
Crea un hook para cachear datos con TTL:

```typescript
// Hook a implementar
export function useDataCache<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 5 * 60 * 1000 // 5 minutos
): UseDataCacheReturn<T> {
  // Implementa:
  // - Cache en memoria
// - TTL automático
// - Invalidador de cache
// - Persistencia opcional
}

// Escribe tests que cubran:
// - Cache de datos
// - Expiración de TTL
// - Invalidador de cache
// - Manejo de errores
```

## 📝 **Resumen de la Clase**

### **Conceptos Clave Aprendidos**
1. **Testing de Hooks**: Uso de @testing-library/react-hooks
2. **Testing de Lógica de Negocio**: Validaciones, cálculos, transformaciones
3. **Mocks de APIs Nativas**: AsyncStorage, fetch, etc.
4. **Testing de Efectos Secundarios**: useEffect, async operations
5. **Testing de Estado Complejo**: Formularios, validaciones, cache
6. **Performance de Tests**: Referencias estables, memoización

### **Próximos Pasos**
- Testing de integración y APIs
- Testing e2e y performance
- Estrategias de testing y mejores prácticas

## 🔗 **Navegación**
- [⬅️ Clase Anterior](clase_2_testing_componentes.md) - Testing de Componentes
- [🏠 Inicio](../../README.md)
- [📚 Índice Completo](../../INDICE_COMPLETO.md)
- [➡️ Siguiente Clase](clase_4_testing_integracion_apis.md) - Testing de Integración y APIs

---

**💡 Tip**: Usa `act()` para envolver todas las operaciones que cambien el estado del hook. Esto asegura que los tests se comporten como en un entorno real de React.
