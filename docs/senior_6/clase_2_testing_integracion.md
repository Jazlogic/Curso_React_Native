# Clase 2: Testing de Integración 🧪

## Objetivos de la Clase
- Implementar testing de integración para APIs y servicios
- Testing de navegación y flujos de usuario
- Testing de estado global y context
- Testing de integración entre componentes

## Contenido Teórico

### 1. Testing de APIs y Servicios

#### Servicio de Usuarios
```javascript
// src/services/userService.js
import AsyncStorage from '@react-native-async-storage/async-storage';

export class UserService {
  constructor(baseURL = 'https://api.example.com') {
    this.baseURL = baseURL;
    this.token = null;
  }

  // Configurar token de autenticación
  setAuthToken(token) {
    this.token = token;
  }

  // Obtener headers de autenticación
  getAuthHeaders() {
    const headers = {
      'Content-Type': 'application/json',
    };
    
    if (this.token) {
      headers.Authorization = `Bearer ${this.token}`;
    }
    
    return headers;
  }

  // Obtener lista de usuarios
  async getUsers(page = 1, limit = 10) {
    try {
      const response = await fetch(
        `${this.baseURL}/users?page=${page}&limit=${limit}`,
        {
          method: 'GET',
          headers: this.getAuthHeaders(),
        }
      );

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      return { success: true, data };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Obtener usuario por ID
  async getUserById(id) {
    try {
      const response = await fetch(`${this.baseURL}/users/${id}`, {
        method: 'GET',
        headers: this.getAuthHeaders(),
      });

      if (!response.ok) {
        if (response.status === 404) {
          throw new Error('User not found');
        }
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      return { success: true, data };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Crear nuevo usuario
  async createUser(userData) {
    try {
      const response = await fetch(`${this.baseURL}/users`, {
        method: 'POST',
        headers: this.getAuthHeaders(),
        body: JSON.stringify(userData),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Failed to create user');
      }

      const data = await response.json();
      return { success: true, data };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Actualizar usuario
  async updateUser(id, userData) {
    try {
      const response = await fetch(`${this.baseURL}/users/${id}`, {
        method: 'PUT',
        headers: this.getAuthHeaders(),
        body: JSON.stringify(userData),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Failed to update user');
      }

      const data = await response.json();
      return { success: true, data };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Eliminar usuario
  async deleteUser(id) {
    try {
      const response = await fetch(`${this.baseURL}/users/${id}`, {
        method: 'DELETE',
        headers: this.getAuthHeaders(),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Cache local de usuarios
  async cacheUsers(users) {
    try {
      await AsyncStorage.setItem('cached_users', JSON.stringify(users));
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Obtener usuarios del cache
  async getCachedUsers() {
    try {
      const cached = await AsyncStorage.getItem('cached_users');
      if (cached) {
        return { success: true, data: JSON.parse(cached) };
      }
      return { success: false, error: 'No cached data' };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }
}

// Instancia singleton
export const userService = new UserService();
```

#### Testing de Integración del Servicio
```javascript
// src/services/__tests__/userService.integration.test.js
import { UserService } from '../userService';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Mock de fetch global
global.fetch = jest.fn();

// Mock de AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);

describe('UserService Integration Tests', () => {
  let userService;

  beforeEach(() => {
    jest.clearAllMocks();
    userService = new UserService('https://test-api.com');
    AsyncStorage.clear();
  });

  describe('Autenticación', () => {
    it('debe incluir token en headers cuando está configurado', () => {
      const token = 'test-token-123';
      userService.setAuthToken(token);

      const headers = userService.getAuthHeaders();
      
      expect(headers.Authorization).toBe(`Bearer ${token}`);
      expect(headers['Content-Type']).toBe('application/json');
    });

    it('no debe incluir Authorization header cuando no hay token', () => {
      const headers = userService.getAuthHeaders();
      
      expect(headers.Authorization).toBeUndefined();
      expect(headers['Content-Type']).toBe('application/json');
    });
  });

  describe('getUsers', () => {
    it('debe hacer request exitoso con parámetros de paginación', async () => {
      const mockUsers = [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' },
      ];

      global.fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockUsers),
      });

      const result = await userService.getUsers(2, 5);

      expect(global.fetch).toHaveBeenCalledWith(
        'https://test-api.com/users?page=2&limit=5',
        {
          method: 'GET',
          headers: { 'Content-Type': 'application/json' },
        }
      );

      expect(result.success).toBe(true);
      expect(result.data).toEqual(mockUsers);
    });

    it('debe manejar errores HTTP', async () => {
      global.fetch.mockResolvedValueOnce({
        ok: false,
        status: 500,
      });

      const result = await userService.getUsers();

      expect(result.success).toBe(false);
      expect(result.error).toBe('HTTP error! status: 500');
    });

    it('debe manejar errores de red', async () => {
      global.fetch.mockRejectedValueOnce(new Error('Network error'));

      const result = await userService.getUsers();

      expect(result.success).toBe(false);
      expect(result.error).toBe('Network error');
    });
  });

  describe('getUserById', () => {
    it('debe obtener usuario exitosamente', async () => {
      const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };

      global.fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockUser),
      });

      const result = await userService.getUserById(1);

      expect(global.fetch).toHaveBeenCalledWith(
        'https://test-api.com/users/1',
        {
          method: 'GET',
          headers: { 'Content-Type': 'application/json' },
        }
      );

      expect(result.success).toBe(true);
      expect(result.data).toEqual(mockUser);
    });

    it('debe manejar usuario no encontrado', async () => {
      global.fetch.mockResolvedValueOnce({
        ok: false,
        status: 404,
      });

      const result = await userService.getUserById(999);

      expect(result.success).toBe(false);
      expect(result.error).toBe('User not found');
    });
  });

  describe('createUser', () => {
    it('debe crear usuario exitosamente', async () => {
      const userData = { name: 'New User', email: 'new@example.com' };
      const createdUser = { id: 3, ...userData };

      global.fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(createdUser),
      });

      const result = await userService.createUser(userData);

      expect(global.fetch).toHaveBeenCalledWith(
        'https://test-api.com/users',
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(userData),
        }
      );

      expect(result.success).toBe(true);
      expect(result.data).toEqual(createdUser);
    });

    it('debe manejar errores de validación', async () => {
      const userData = { name: '', email: 'invalid-email' };

      global.fetch.mockResolvedValueOnce({
        ok: false,
        status: 400,
        json: () => Promise.resolve({ message: 'Validation failed' }),
      });

      const result = await userService.createUser(userData);

      expect(result.success).toBe(false);
      expect(result.error).toBe('Validation failed');
    });
  });

  describe('Cache Integration', () => {
    it('debe guardar y recuperar usuarios del cache', async () => {
      const users = [
        { id: 1, name: 'Cached User 1' },
        { id: 2, name: 'Cached User 2' },
      ];

      // Guardar en cache
      const saveResult = await userService.cacheUsers(users);
      expect(saveResult.success).toBe(true);

      // Recuperar del cache
      const getResult = await userService.getCachedUsers();
      expect(getResult.success).toBe(true);
      expect(getResult.data).toEqual(users);
    });

    it('debe manejar cache vacío', async () => {
      const result = await userService.getCachedUsers();
      
      expect(result.success).toBe(false);
      expect(result.error).toBe('No cached data');
    });
  });
});
```

### 2. Testing de Navegación

#### Navegador Principal
```javascript
// src/navigation/AppNavigator.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { useAuth } from '../hooks/useAuth';

// Screens
import LoginScreen from '../screens/LoginScreen';
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import UserListScreen from '../screens/UserListScreen';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

// Navegador de tabs para usuarios autenticados
const AuthenticatedTabs = () => (
  <Tab.Navigator>
    <Tab.Screen 
      name="Home" 
      component={HomeScreen}
      options={{ title: 'Inicio' }}
    />
    <Tab.Screen 
      name="Users" 
      component={UserListScreen}
      options={{ title: 'Usuarios' }}
    />
    <Tab.Screen 
      name="Profile" 
      component={ProfileScreen}
      options={{ title: 'Perfil' }}
    />
    <Tab.Screen 
      name="Settings" 
      component={SettingsScreen}
      options={{ title: 'Configuración' }}
    />
  </Tab.Navigator>
);

// Navegador principal de la app
export const AppNavigator = () => {
  const { isAuthenticated, loading } = useAuth();

  if (loading) {
    return null; // O un componente de loading
  }

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {isAuthenticated ? (
          <Stack.Screen name="Main" component={AuthenticatedTabs} />
        ) : (
          <Stack.Screen name="Login" component={LoginScreen} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

#### Testing de Integración de Navegación
```javascript
// src/navigation/__tests__/AppNavigator.integration.test.js
import React from 'react';
import { render, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';
import { AppNavigator } from '../AppNavigator';
import { useAuth } from '../../hooks/useAuth';

// Mock de useAuth
jest.mock('../../hooks/useAuth');

// Mock de screens
jest.mock('../../screens/LoginScreen', () => 'LoginScreen');
jest.mock('../../screens/HomeScreen', () => 'HomeScreen');
jest.mock('../../screens/ProfileScreen', () => 'ProfileScreen');
jest.mock('../../screens/SettingsScreen', () => 'SettingsScreen');
jest.mock('../../screens/UserListScreen', () => 'UserListScreen');

describe('AppNavigator Integration Tests', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('Estado de autenticación', () => {
    it('debe mostrar LoginScreen cuando no está autenticado', async () => {
      useAuth.mockReturnValue({
        isAuthenticated: false,
        loading: false,
      });

      const { getByText } = render(
        <NavigationContainer>
          <AppNavigator />
        </NavigationContainer>
      );

      await waitFor(() => {
        expect(getByText('LoginScreen')).toBeTruthy();
      });
    });

    it('debe mostrar tabs cuando está autenticado', async () => {
      useAuth.mockReturnValue({
        isAuthenticated: true,
        loading: false,
      });

      const { getByText } = render(
        <NavigationContainer>
          <AppNavigator />
        </NavigationContainer>
      );

      await waitFor(() => {
        expect(getByText('HomeScreen')).toBeTruthy();
        expect(getByText('UserListScreen')).toBeTruthy();
        expect(getByText('ProfileScreen')).toBeTruthy();
        expect(getByText('SettingsScreen')).toBeTruthy();
      });
    });

    it('debe mostrar nada durante loading', () => {
      useAuth.mockReturnValue({
        isAuthenticated: false,
        loading: true,
      });

      const { container } = render(
        <NavigationContainer>
          <AppNavigator />
        </NavigationContainer>
      );

      expect(container.children).toHaveLength(0);
    });
  });

  describe('Navegación entre tabs', () => {
    it('debe permitir navegación entre tabs autenticados', async () => {
      useAuth.mockReturnValue({
        isAuthenticated: true,
        loading: false,
      });

      const { getByText } = render(
        <NavigationContainer>
          <AppNavigator />
        </NavigationContainer>
      );

      // Verificar que todos los tabs están disponibles
      await waitFor(() => {
        expect(getByText('HomeScreen')).toBeTruthy();
        expect(getByText('UserListScreen')).toBeTruthy();
        expect(getByText('ProfileScreen')).toBeTruthy();
        expect(getByText('SettingsScreen')).toBeTruthy();
      });
    });
  });
});
```

### 3. Testing de Estado Global

#### Context de Usuarios
```javascript
// src/contexts/UserContext.js
import React, { createContext, useContext, useReducer, useCallback } from 'react';
import { userService } from '../services/userService';

// Estado inicial
const initialState = {
  users: [],
  currentUser: null,
  loading: false,
  error: null,
  filters: {
    search: '',
    status: 'all',
    role: 'all',
  },
  pagination: {
    page: 1,
    limit: 10,
    total: 0,
  },
};

// Tipos de acciones
const USER_ACTIONS = {
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR',
  SET_USERS: 'SET_USERS',
  SET_CURRENT_USER: 'SET_CURRENT_USER',
  ADD_USER: 'ADD_USER',
  UPDATE_USER: 'UPDATE_USER',
  DELETE_USER: 'DELETE_USER',
  SET_FILTERS: 'SET_FILTERS',
  SET_PAGINATION: 'SET_PAGINATION',
  CLEAR_ERROR: 'CLEAR_ERROR',
};

// Reducer
const userReducer = (state, action) => {
  switch (action.type) {
    case USER_ACTIONS.SET_LOADING:
      return { ...state, loading: action.payload };

    case USER_ACTIONS.SET_ERROR:
      return { ...state, error: action.payload, loading: false };

    case USER_ACTIONS.SET_USERS:
      return { 
        ...state, 
        users: action.payload.users,
        pagination: action.payload.pagination || state.pagination,
        loading: false,
        error: null,
      };

    case USER_ACTIONS.SET_CURRENT_USER:
      return { ...state, currentUser: action.payload };

    case USER_ACTIONS.ADD_USER:
      return { 
        ...state, 
        users: [action.payload, ...state.users],
        pagination: {
          ...state.pagination,
          total: state.pagination.total + 1,
        },
      };

    case USER_ACTIONS.UPDATE_USER:
      return {
        ...state,
        users: state.users.map(user =>
          user.id === action.payload.id ? action.payload : user
        ),
        currentUser: state.currentUser?.id === action.payload.id 
          ? action.payload 
          : state.currentUser,
      };

    case USER_ACTIONS.DELETE_USER:
      return {
        ...state,
        users: state.users.filter(user => user.id !== action.payload),
        pagination: {
          ...state.pagination,
          total: Math.max(0, state.pagination.total - 1),
        },
      };

    case USER_ACTIONS.SET_FILTERS:
      return { 
        ...state, 
        filters: { ...state.filters, ...action.payload },
        pagination: { ...state.pagination, page: 1 }, // Reset page
      };

    case USER_ACTIONS.SET_PAGINATION:
      return { ...state, pagination: { ...state.pagination, ...action.payload } };

    case USER_ACTIONS.CLEAR_ERROR:
      return { ...state, error: null };

    default:
      return state;
  }
};

// Context
const UserContext = createContext();

// Provider
export const UserProvider = ({ children }) => {
  const [state, dispatch] = useReducer(userReducer, initialState);

  // Cargar usuarios
  const loadUsers = useCallback(async (page = 1, limit = 10) => {
    try {
      dispatch({ type: USER_ACTIONS.SET_LOADING, payload: true });
      
      const result = await userService.getUsers(page, limit);
      
      if (result.success) {
        dispatch({
          type: USER_ACTIONS.SET_USERS,
          payload: {
            users: result.data.users,
            pagination: {
              page,
              limit,
              total: result.data.total || 0,
            },
          },
        });
      } else {
        dispatch({ type: USER_ACTIONS.SET_ERROR, payload: result.error });
      }
    } catch (error) {
      dispatch({ type: USER_ACTIONS.SET_ERROR, payload: error.message });
    }
  }, []);

  // Cargar usuario por ID
  const loadUserById = useCallback(async (id) => {
    try {
      dispatch({ type: USER_ACTIONS.SET_LOADING, payload: true });
      
      const result = await userService.getUserById(id);
      
      if (result.success) {
        dispatch({ type: USER_ACTIONS.SET_CURRENT_USER, payload: result.data });
      } else {
        dispatch({ type: USER_ACTIONS.SET_ERROR, payload: result.error });
      }
    } catch (error) {
      dispatch({ type: USER_ACTIONS.SET_ERROR, payload: error.message });
    }
  }, []);

  // Crear usuario
  const createUser = useCallback(async (userData) => {
    try {
      dispatch({ type: USER_ACTIONS.SET_LOADING, payload: true });
      
      const result = await userService.createUser(userData);
      
      if (result.success) {
        dispatch({ type: USER_ACTIONS.ADD_USER, payload: result.data });
        return { success: true, data: result.data };
      } else {
        dispatch({ type: USER_ACTIONS.SET_ERROR, payload: result.error });
        return { success: false, error: result.error };
      }
    } catch (error) {
      dispatch({ type: USER_ACTIONS.SET_ERROR, payload: error.message });
      return { success: false, error: error.message };
    }
  }, []);

  // Actualizar usuario
  const updateUser = useCallback(async (id, userData) => {
    try {
      dispatch({ type: USER_ACTIONS.SET_LOADING, payload: true });
      
      const result = await userService.updateUser(id, userData);
      
      if (result.success) {
        dispatch({ type: USER_ACTIONS.UPDATE_USER, payload: result.data });
        return { success: true, data: result.data };
      } else {
        dispatch({ type: USER_ACTIONS.SET_ERROR, payload: result.error });
        return { success: false, error: result.error };
      }
    } catch (error) {
      dispatch({ type: USER_ACTIONS.SET_ERROR, payload: error.message });
      return { success: false, error: error.message };
    }
  }, []);

  // Eliminar usuario
  const deleteUser = useCallback(async (id) => {
    try {
      dispatch({ type: USER_ACTIONS.SET_LOADING, payload: true });
      
      const result = await userService.deleteUser(id);
      
      if (result.success) {
        dispatch({ type: USER_ACTIONS.DELETE_USER, payload: id });
        return { success: true };
      } else {
        dispatch({ type: USER_ACTIONS.SET_ERROR, payload: result.error });
        return { success: false, error: result.error };
      }
    } catch (error) {
      dispatch({ type: USER_ACTIONS.SET_ERROR, payload: error.message });
      return { success: false, error: error.message };
    }
  }, []);

  // Aplicar filtros
  const applyFilters = useCallback((filters) => {
    dispatch({ type: USER_ACTIONS.SET_FILTERS, payload: filters });
  }, []);

  // Cambiar página
  const changePage = useCallback((page) => {
    dispatch({ type: USER_ACTIONS.SET_PAGINATION, payload: { page } });
  }, []);

  // Limpiar error
  const clearError = useCallback(() => {
    dispatch({ type: USER_ACTIONS.CLEAR_ERROR });
  }, []);

  const value = {
    ...state,
    loadUsers,
    loadUserById,
    createUser,
    updateUser,
    deleteUser,
    applyFilters,
    changePage,
    clearError,
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
};

// Hook personalizado
export const useUsers = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUsers must be used within a UserProvider');
  }
  return context;
};
```

#### Testing de Integración del Context
```javascript
// src/contexts/__tests__/UserContext.integration.test.js
import React from 'react';
import { render, act, waitFor } from '@testing-library/react-native';
import { UserProvider, useUsers } from '../UserContext';
import { userService } from '../../services/userService';

// Mock de userService
jest.mock('../../services/userService');

// Componente de prueba
const TestComponent = () => {
  const { 
    users, 
    loading, 
    error, 
    loadUsers, 
    createUser,
    updateUser,
    deleteUser,
    applyFilters,
    changePage,
  } = useUsers();

  return (
    <div>
      <div data-testid="loading">{loading ? 'Loading' : 'Not Loading'}</div>
      <div data-testid="error">{error || 'No Error'}</div>
      <div data-testid="users-count">{users.length}</div>
      <button data-testid="load-users" onPress={() => loadUsers()}>
        Load Users
      </button>
      <button data-testid="create-user" onPress={() => createUser({ name: 'Test' })}>
        Create User
      </button>
      <button data-testid="update-user" onPress={() => updateUser(1, { name: 'Updated' })}>
        Update User
      </button>
      <button data-testid="delete-user" onPress={() => deleteUser(1)}>
        Delete User
      </button>
      <button data-testid="apply-filters" onPress={() => applyFilters({ search: 'test' })}>
        Apply Filters
      </button>
      <button data-testid="change-page" onPress={() => changePage(2)}>
        Change Page
      </button>
    </div>
  );
};

describe('UserContext Integration Tests', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('Estado inicial', () => {
    it('debe tener estado inicial correcto', () => {
      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      expect(getByTestId('loading').children[0]).toBe('Not Loading');
      expect(getByTestId('error').children[0]).toBe('No Error');
      expect(getByTestId('users-count').children[0]).toBe('0');
    });
  });

  describe('loadUsers', () => {
    it('debe cargar usuarios exitosamente', async () => {
      const mockUsers = [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' },
      ];

      userService.getUsers.mockResolvedValue({
        success: true,
        data: { users: mockUsers, total: 2 },
      });

      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      const loadButton = getByTestId('load-users');
      
      await act(async () => {
        loadButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('loading').children[0]).toBe('Not Loading');
        expect(getByTestId('users-count').children[0]).toBe('2');
        expect(getByTestId('error').children[0]).toBe('No Error');
      });
    });

    it('debe manejar errores de carga', async () => {
      userService.getUsers.mockResolvedValue({
        success: false,
        error: 'Failed to load users',
      });

      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      const loadButton = getByTestId('load-users');
      
      await act(async () => {
        loadButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('error').children[0]).toBe('Failed to load users');
        expect(getByTestId('loading').children[0]).toBe('Not Loading');
      });
    });
  });

  describe('createUser', () => {
    it('debe crear usuario exitosamente', async () => {
      const newUser = { id: 3, name: 'Test User' };
      
      userService.createUser.mockResolvedValue({
        success: true,
        data: newUser,
      });

      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      const createButton = getByTestId('create-user');
      
      await act(async () => {
        createButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('users-count').children[0]).toBe('1');
      });
    });
  });

  describe('updateUser', () => {
    it('debe actualizar usuario exitosamente', async () => {
      // Primero cargar usuarios
      const mockUsers = [{ id: 1, name: 'John Doe' }];
      userService.getUsers.mockResolvedValue({
        success: true,
        data: { users: mockUsers, total: 1 },
      });

      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      // Cargar usuarios
      const loadButton = getByTestId('load-users');
      await act(async () => {
        loadButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('users-count').children[0]).toBe('1');
      });

      // Actualizar usuario
      const updatedUser = { id: 1, name: 'John Updated' };
      userService.updateUser.mockResolvedValue({
        success: true,
        data: updatedUser,
      });

      const updateButton = getByTestId('update-user');
      await act(async () => {
        updateButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('users-count').children[0]).toBe('1');
      });
    });
  });

  describe('deleteUser', () => {
    it('debe eliminar usuario exitosamente', async () => {
      // Primero cargar usuarios
      const mockUsers = [{ id: 1, name: 'John Doe' }];
      userService.getUsers.mockResolvedValue({
        success: true,
        data: { users: mockUsers, total: 1 },
      });

      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      // Cargar usuarios
      const loadButton = getByTestId('load-users');
      await act(async () => {
        loadButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('users-count').children[0]).toBe('1');
      });

      // Eliminar usuario
      userService.deleteUser.mockResolvedValue({
        success: true,
      });

      const deleteButton = getByTestId('delete-user');
      await act(async () => {
        deleteButton.props.onPress();
      });

      await waitFor(() => {
        expect(getByTestId('users-count').children[0]).toBe('0');
      });
    });
  });

  describe('Filtros y paginación', () => {
    it('debe aplicar filtros correctamente', async () => {
      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      const filterButton = getByTestId('apply-filters');
      
      await act(async () => {
        filterButton.props.onPress();
      });

      // Los filtros se aplican inmediatamente
      expect(getByTestId('users-count').children[0]).toBe('0');
    });

    it('debe cambiar página correctamente', async () => {
      const { getByTestId } = render(
        <UserProvider>
          <TestComponent />
        </UserProvider>
      );

      const pageButton = getByTestId('change-page');
      
      await act(async () => {
        pageButton.props.onPress();
      });

      // La página se cambia inmediatamente
      expect(getByTestId('users-count').children[0]).toBe('0');
    });
  });
});
```

## Ejercicios Prácticos

### Ejercicio 1: Testing de Integración de API Completa
Crea un servicio de productos y escribe tests de integración para:

```javascript
// Servicio a implementar
export class ProductService {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async getProducts(filters = {}) { /* implementar */ }
  async getProduct(id) { /* implementar */ }
  async createProduct(productData) { /* implementar */ }
  async updateProduct(id, productData) { /* implementar */ }
  async deleteProduct(id) { /* implementar */ }
  async searchProducts(query) { /* implementar */ }
}
```

**Requisitos del testing:**
- Test de todas las operaciones CRUD
- Test de manejo de errores HTTP
- Test de timeout y errores de red
- Test de diferentes códigos de estado
- Test de validación de respuestas

### Ejercicio 2: Testing de Navegación Compleja
Crea un navegador con múltiples stacks y escribe tests para:

```javascript
// Navegador a implementar
export const AppNavigator = () => {
  const { isAuthenticated, userRole } = useAuth();
  
  if (isAuthenticated) {
    if (userRole === 'admin') {
      return <AdminNavigator />;
    } else {
      return <UserNavigator />;
    }
  }
  
  return <AuthNavigator />;
};
```

**Requisitos del testing:**
- Test de navegación para usuarios no autenticados
- Test de navegación para usuarios normales
- Test de navegación para administradores
- Test de cambio de roles
- Test de navegación entre stacks

### Ejercicio 3: Testing de Context Complejo
Crea un context de carrito de compras y escribe tests para:

```javascript
// Context a implementar
export const CartContext = createContext();

export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState([]);
  const [loading, setLoading] = useState(false);
  
  const addItem = (item) => { /* implementar */ };
  const removeItem = (itemId) => { /* implementar */ };
  const updateQuantity = (itemId, quantity) => { /* implementar */ };
  const clearCart = () => { /* implementar */ };
  const getTotal = () => { /* implementar */ };
  
  return (
    <CartContext.Provider value={{
      cart, loading, addItem, removeItem, updateQuantity, clearCart, getTotal
    }}>
      {children}
    </CartContext.Provider>
  );
};
```

**Requisitos del testing:**
- Test de estado inicial del carrito
- Test de agregar productos
- Test de remover productos
- Test de actualizar cantidades
- Test de limpiar carrito
- Test de cálculo de total
- Test de persistencia de datos

## Resumen de la Clase

En esta clase hemos aprendido:

1. **Testing de APIs**: Testing completo de servicios con mocks de fetch y manejo de errores
2. **Testing de Navegación**: Testing de flujos de navegación y cambios de estado
3. **Testing de Estado Global**: Testing de context y reducers con estado complejo
4. **Testing de Integración**: Testing de la interacción entre diferentes partes del sistema

## Navegación
- **Anterior**: [Clase 1: Testing Unitario Avanzado](clase_1_testing_unitario_avanzado.md)
- **Siguiente**: [Clase 3: Testing E2E con Detox](clase_3_testing_e2e_detox.md)
- **Inicio**: [Índice del Curso](../../INDICE_COMPLETO.md)
