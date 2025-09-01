# Clase 1: Testing Unitario Avanzado 🧪

## Objetivos de la Clase
- Dominar técnicas avanzadas de testing unitario con Jest
- Implementar mocks complejos para dependencias externas
- Testing de hooks personalizados y componentes complejos
- Configurar testing de componentes con diferentes estados

## Contenido Teórico

### 1. Jest Avanzado

#### Configuración Avanzada de Jest
```javascript
// jest.config.js
module.exports = {
  // Configuración básica
  preset: 'react-native',
  
  // Configuración de setup
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  
  // Configuración de transformación
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest',
  },
  
  // Configuración de mocks
  moduleNameMapper: {
    // Mock de assets
    '\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$':
      '<rootDir>/__mocks__/fileMock.js',
    
    // Mock de módulos nativos
    'react-native-vector-icons': '<rootDir>/__mocks__/vectorIconsMock.js',
  },
  
  // Configuración de cobertura
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.{js,jsx,ts,tsx}',
  ],
  
  // Umbrales de cobertura
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

#### Setup Avanzado de Jest
```javascript
// jest.setup.js
import 'react-native-gesture-handler/jestSetup';

// Mock de react-native-reanimated
jest.mock('react-native-reanimated', () => {
  const Reanimated = require('react-native-reanimated/mock');
  Reanimated.default.call = () => {};
  return Reanimated;
});

// Mock de react-native-vector-icons
jest.mock('react-native-vector-icons/MaterialIcons', () => 'Icon');

// Mock de AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);

// Mock de react-native-keychain
jest.mock('react-native-keychain', () => ({
  setInternetCredentials: jest.fn(),
  getInternetCredentials: jest.fn(),
  resetInternetCredentials: jest.fn(),
}));

// Configuración global de testing
global.console = {
  ...console,
  // Silenciar warnings en tests
  warn: jest.fn(),
  error: jest.fn(),
};
```

### 2. Mocks Complejos

#### Mock de APIs Externas
```javascript
// __mocks__/apiMock.js
export const mockApiResponse = {
  success: true,
  data: {
    users: [
      { id: 1, name: 'John Doe', email: 'john@example.com' },
      { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
    ],
  },
};

export const mockApiError = {
  success: false,
  error: 'Network error',
  status: 500,
};

// Mock de fetch global
global.fetch = jest.fn(() =>
  Promise.resolve({
    ok: true,
    json: () => Promise.resolve(mockApiResponse),
    status: 200,
  })
);

// Mock de axios
export const mockAxios = {
  get: jest.fn(() => Promise.resolve({ data: mockApiResponse })),
  post: jest.fn(() => Promise.resolve({ data: mockApiResponse })),
  put: jest.fn(() => Promise.resolve({ data: mockApiResponse })),
  delete: jest.fn(() => Promise.resolve({ data: mockApiResponse })),
};
```

#### Mock de Navegación
```javascript
// __mocks__/navigationMock.js
export const mockNavigation = {
  navigate: jest.fn(),
  goBack: jest.fn(),
  push: jest.fn(),
  pop: jest.fn(),
  reset: jest.fn(),
  setOptions: jest.fn(),
  addListener: jest.fn(() => ({ remove: jest.fn() })),
  getState: jest.fn(() => ({
    routes: [{ name: 'Home' }],
    index: 0,
  })),
};

export const mockRoute = {
  params: {},
  name: 'TestScreen',
  key: 'test-key',
};

// Mock de useNavigation hook
export const mockUseNavigation = () => mockNavigation;

// Mock de useRoute hook
export const mockUseRoute = () => mockRoute;
```

#### Mock de Context y Estado Global
```javascript
// __mocks__/contextMock.js
export const mockAuthContext = {
  user: {
    id: 1,
    name: 'Test User',
    email: 'test@example.com',
  },
  isAuthenticated: true,
  login: jest.fn(),
  logout: jest.fn(),
  updateUser: jest.fn(),
};

export const mockThemeContext = {
  theme: 'light',
  colors: {
    primary: '#007AFF',
    secondary: '#5856D6',
    background: '#FFFFFF',
    text: '#000000',
  },
  toggleTheme: jest.fn(),
  setTheme: jest.fn(),
};

// Mock de useAuth hook
export const mockUseAuth = () => mockAuthContext;

// Mock de useTheme hook
export const mockUseTheme = () => mockThemeContext;
```

### 3. Testing de Hooks Personalizados

#### Hook de Autenticación
```javascript
// src/hooks/useAuth.js
import { useState, useEffect, useCallback } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const useAuth = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Cargar usuario desde storage
  const loadUser = useCallback(async () => {
    try {
      setLoading(true);
      const userData = await AsyncStorage.getItem('user');
      if (userData) {
        setUser(JSON.parse(userData));
      }
    } catch (err) {
      setError('Error loading user data');
    } finally {
      setLoading(false);
    }
  }, []);

  // Login
  const login = useCallback(async (credentials) => {
    try {
      setLoading(true);
      setError(null);
      
      // Simular llamada a API
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials),
      });
      
      if (!response.ok) {
        throw new Error('Invalid credentials');
      }
      
      const userData = await response.json();
      setUser(userData);
      await AsyncStorage.setItem('user', JSON.stringify(userData));
      
      return { success: true };
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    } finally {
      setLoading(false);
    }
  }, []);

  // Logout
  const logout = useCallback(async () => {
    try {
      setUser(null);
      await AsyncStorage.removeItem('user');
      return { success: true };
    } catch (err) {
      setError('Error during logout');
      return { success: false, error: err.message };
    }
  }, []);

  // Cargar usuario al montar
  useEffect(() => {
    loadUser();
  }, [loadUser]);

  return {
    user,
    loading,
    error,
    login,
    logout,
    loadUser,
    isAuthenticated: !!user,
  };
};
```

#### Testing del Hook de Autenticación
```javascript
// src/hooks/__tests__/useAuth.test.js
import { renderHook, act, waitFor } from '@testing-library/react-hooks';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useAuth } from '../useAuth';

// Mock de fetch
global.fetch = jest.fn();

// Mock de AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);

describe('useAuth Hook', () => {
  beforeEach(() => {
    // Limpiar todos los mocks
    jest.clearAllMocks();
    AsyncStorage.clear();
  });

  describe('Estado inicial', () => {
    it('debe tener estado inicial correcto', () => {
      const { result } = renderHook(() => useAuth());

      expect(result.current.user).toBeNull();
      expect(result.current.loading).toBe(true);
      expect(result.current.error).toBeNull();
      expect(result.current.isAuthenticated).toBe(false);
    });
  });

  describe('loadUser', () => {
    it('debe cargar usuario desde AsyncStorage', async () => {
      const mockUser = { id: 1, name: 'Test User' };
      await AsyncStorage.setItem('user', JSON.stringify(mockUser));

      const { result } = renderHook(() => useAuth());

      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      expect(result.current.user).toEqual(mockUser);
      expect(result.current.isAuthenticated).toBe(true);
    });

    it('debe manejar errores de AsyncStorage', async () => {
      // Mock de error en AsyncStorage
      AsyncStorage.getItem.mockRejectedValueOnce(new Error('Storage error'));

      const { result } = renderHook(() => useAuth());

      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      expect(result.current.error).toBe('Error loading user data');
    });
  });

  describe('login', () => {
    it('debe hacer login exitoso', async () => {
      const mockUser = { id: 1, name: 'Test User' };
      const credentials = { email: 'test@example.com', password: 'password' };

      // Mock de respuesta exitosa
      global.fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockUser),
      });

      const { result } = renderHook(() => useAuth());

      let loginResult;
      await act(async () => {
        loginResult = await result.current.login(credentials);
      });

      expect(loginResult.success).toBe(true);
      expect(result.current.user).toEqual(mockUser);
      expect(result.current.isAuthenticated).toBe(true);
      expect(result.current.error).toBeNull();
    });

    it('debe manejar errores de login', async () => {
      const credentials = { email: 'test@example.com', password: 'wrong' };

      // Mock de respuesta de error
      global.fetch.mockResolvedValueOnce({
        ok: false,
        status: 401,
      });

      const { result } = renderHook(() => useAuth());

      let loginResult;
      await act(async () => {
        loginResult = await result.current.login(credentials);
      });

      expect(loginResult.success).toBe(false);
      expect(loginResult.error).toBe('Invalid credentials');
      expect(result.current.error).toBe('Invalid credentials');
      expect(result.current.user).toBeNull();
    });
  });

  describe('logout', () => {
    it('debe hacer logout exitoso', async () => {
      const mockUser = { id: 1, name: 'Test User' };
      await AsyncStorage.setItem('user', JSON.stringify(mockUser));

      const { result } = renderHook(() => useAuth());

      // Esperar a que se cargue el usuario
      await waitFor(() => {
        expect(result.current.isAuthenticated).toBe(true);
      });

      let logoutResult;
      await act(async () => {
        logoutResult = await result.current.logout();
      });

      expect(logoutResult.success).toBe(true);
      expect(result.current.user).toBeNull();
      expect(result.current.isAuthenticated).toBe(false);
    });
  });
});
```

### 4. Testing de Componentes Complejos

#### Componente de Login
```javascript
// src/components/LoginForm.js
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
} from 'react-native';
import { useAuth } from '../hooks/useAuth';

export const LoginForm = ({ onSuccess, onError }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);
  
  const { login } = useAuth();

  const handleLogin = async () => {
    if (!email || !password) {
      Alert.alert('Error', 'Please fill in all fields');
      return;
    }

    setLoading(true);
    try {
      const result = await login({ email, password });
      
      if (result.success) {
        onSuccess?.(result);
      } else {
        onError?.(result.error);
      }
    } catch (error) {
      onError?.(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>
      
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
        testID="email-input"
      />
      
      <TextInput
        style={styles.input}
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        testID="password-input"
      />
      
      <TouchableOpacity
        style={[styles.button, loading && styles.buttonDisabled]}
        onPress={handleLogin}
        disabled={loading}
        testID="login-button"
      >
        <Text style={styles.buttonText}>
          {loading ? 'Loading...' : 'Login'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    marginBottom: 16,
    fontSize: 16,
  },
  button: {
    backgroundColor: '#007AFF',
    borderRadius: 8,
    padding: 16,
    alignItems: 'center',
  },
  buttonDisabled: {
    backgroundColor: '#ccc',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

#### Testing del Componente de Login
```javascript
// src/components/__tests__/LoginForm.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { Alert } from 'react-native';
import { LoginForm } from '../LoginForm';
import { useAuth } from '../../hooks/useAuth';

// Mock de useAuth
jest.mock('../../hooks/useAuth');

// Mock de Alert
jest.spyOn(Alert, 'alert');

describe('LoginForm Component', () => {
  const mockLogin = jest.fn();
  const mockOnSuccess = jest.fn();
  const mockOnError = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
    useAuth.mockReturnValue({
      login: mockLogin,
    });
  });

  describe('Renderizado', () => {
    it('debe renderizar correctamente', () => {
      const { getByText, getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      expect(getByText('Login')).toBeTruthy();
      expect(getByTestId('email-input')).toBeTruthy();
      expect(getByTestId('password-input')).toBeTruthy();
      expect(getByTestId('login-button')).toBeTruthy();
    });

    it('debe mostrar botón deshabilitado cuando loading', () => {
      const { getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      const button = getByTestId('login-button');
      expect(button.props.disabled).toBe(false);
    });
  });

  describe('Validación de campos', () => {
    it('debe mostrar alerta si campos están vacíos', () => {
      const { getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      const loginButton = getByTestId('login-button');
      fireEvent.press(loginButton);

      expect(Alert.alert).toHaveBeenCalledWith('Error', 'Please fill in all fields');
      expect(mockLogin).not.toHaveBeenCalled();
    });

    it('debe mostrar alerta si solo email está vacío', () => {
      const { getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      const passwordInput = getByTestId('password-input');
      const loginButton = getByTestId('login-button');

      fireEvent.changeText(passwordInput, 'password123');
      fireEvent.press(loginButton);

      expect(Alert.alert).toHaveBeenCalledWith('Error', 'Please fill in all fields');
    });
  });

  describe('Login exitoso', () => {
    it('debe llamar onSuccess cuando login es exitoso', async () => {
      mockLogin.mockResolvedValue({ success: true, user: { id: 1 } });

      const { getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      const emailInput = getByTestId('email-input');
      const passwordInput = getByTestId('password-input');
      const loginButton = getByTestId('login-button');

      fireEvent.changeText(emailInput, 'test@example.com');
      fireEvent.changeText(passwordInput, 'password123');
      fireEvent.press(loginButton);

      await waitFor(() => {
        expect(mockLogin).toHaveBeenCalledWith({
          email: 'test@example.com',
          password: 'password123',
        });
      });

      expect(mockOnSuccess).toHaveBeenCalledWith({ success: true, user: { id: 1 } });
      expect(mockOnError).not.toHaveBeenCalled();
    });
  });

  describe('Login fallido', () => {
    it('debe llamar onError cuando login falla', async () => {
      mockLogin.mockResolvedValue({ success: false, error: 'Invalid credentials' });

      const { getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      const emailInput = getByTestId('email-input');
      const passwordInput = getByTestId('password-input');
      const loginButton = getByTestId('login-button');

      fireEvent.changeText(emailInput, 'test@example.com');
      fireEvent.changeText(passwordInput, 'wrongpassword');
      fireEvent.press(loginButton);

      await waitFor(() => {
        expect(mockLogin).toHaveBeenCalled();
      });

      expect(mockOnError).toHaveBeenCalledWith('Invalid credentials');
      expect(mockOnSuccess).not.toHaveBeenCalled();
    });
  });

  describe('Estados de loading', () => {
    it('debe mostrar loading en el botón durante login', async () => {
      // Mock de login que nunca se resuelve para simular loading
      mockLogin.mockImplementation(() => new Promise(() => {}));

      const { getByTestId } = render(
        <LoginForm onSuccess={mockOnSuccess} onError={mockOnError} />
      );

      const emailInput = getByTestId('email-input');
      const passwordInput = getByTestId('password-input');
      const loginButton = getByTestId('login-button');

      fireEvent.changeText(emailInput, 'test@example.com');
      fireEvent.changeText(passwordInput, 'password123');
      fireEvent.press(loginButton);

      await waitFor(() => {
        expect(loginButton.props.disabled).toBe(true);
      });
    });
  });
});
```

## Ejercicios Prácticos

### Ejercicio 1: Testing de Hook Personalizado
Crea un hook personalizado `useCounter` y escribe tests completos para él:

```javascript
// Hook a implementar
export const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => setCount(prev => prev + 1), []);
  const decrement = useCallback(() => setCount(prev => prev - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  const setValue = useCallback((value) => setCount(value), []);
  
  return { count, increment, decrement, reset, setValue };
};
```

**Requisitos del testing:**
- Test de estado inicial
- Test de incremento y decremento
- Test de reset
- Test de setValue
- Test de memoización de funciones

### Ejercicio 2: Testing de Componente con Estado Complejo
Crea un componente `TodoList` y escribe tests para:

```javascript
// Componente a implementar
export const TodoList = () => {
  const [todos, setTodos] = useState([]);
  const [inputText, setInputText] = useState('');
  const [filter, setFilter] = useState('all'); // all, active, completed
  
  const addTodo = () => { /* implementar */ };
  const toggleTodo = (id) => { /* implementar */ };
  const deleteTodo = (id) => { /* implementar */ };
  const clearCompleted = () => { /* implementar */ };
  
  const filteredTodos = /* implementar filtrado */;
  
  return (/* JSX del componente */);
};
```

**Requisitos del testing:**
- Test de renderizado inicial
- Test de agregar todo
- Test de toggle todo
- Test de eliminar todo
- Test de filtros
- Test de clear completed
- Test de estado de input

### Ejercicio 3: Mock Complejo de API
Crea un sistema de mocks para una API de usuarios:

```javascript
// API a mockear
export const userApi = {
  getUsers: () => fetch('/api/users').then(res => res.json()),
  getUser: (id) => fetch(`/api/users/${id}`).then(res => res.json()),
  createUser: (userData) => fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData),
  }).then(res => res.json()),
  updateUser: (id, userData) => fetch(`/api/users/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData),
  }).then(res => res.json()),
  deleteUser: (id) => fetch(`/api/users/${id}`, {
    method: 'DELETE',
  }).then(res => res.json()),
};
```

**Requisitos del testing:**
- Mock de fetch global
- Mock de respuestas exitosas y de error
- Mock de diferentes códigos de estado HTTP
- Test de timeout y errores de red
- Test de diferentes formatos de respuesta

## Resumen de la Clase

En esta clase hemos aprendido:

1. **Configuración Avanzada de Jest**: Configuración completa con mocks, transformaciones y cobertura
2. **Mocks Complejos**: Creación de mocks para APIs, navegación, context y estado global
3. **Testing de Hooks**: Testing completo de hooks personalizados con diferentes estados
4. **Testing de Componentes**: Testing de componentes complejos con estado y interacciones

## Navegación
- **Anterior**: [README del Módulo](../README.md)
- **Siguiente**: [Clase 2: Testing de Integración](clase_2_testing_integracion.md)
- **Inicio**: [Índice del Curso](../../INDICE_COMPLETO.md)
