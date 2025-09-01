# 📚 Clase 3: Redux

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 2: Context API](clase_2_context_api.md)
- **➡️ Siguiente**: [Clase 4: Hooks Personalizados Avanzados](clase_4_hooks_personalizados_avanzados.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender los principios fundamentales de Redux
- Aprender a implementar store, reducers y actions
- Dominar el uso de Redux Toolkit para simplificar el código
- Implementar middleware personalizado y asíncrono
- Crear aplicaciones con gestión de estado predecible

---

## 📚 Contenido Teórico

### **¿Qué es Redux?**

Redux es una librería de gestión de estado predecible para aplicaciones JavaScript. Proporciona un contenedor de estado centralizado que sigue principios específicos para hacer que el estado sea predecible y fácil de debuggear.

#### **Principios fundamentales:**
- **Estado único**: Todo el estado de la aplicación se almacena en un solo objeto
- **Estado inmutable**: El estado nunca se modifica directamente, se crea uno nuevo
- **Cambios predecibles**: Solo las actions pueden cambiar el estado
- **Funciones puras**: Los reducers son funciones puras que calculan el nuevo estado

### **Arquitectura de Redux:**

#### **1. Store:**
- Contenedor central del estado
- Proporciona métodos para acceder y actualizar el estado
- Permite suscribirse a cambios del estado

#### **2. Actions:**
- Objetos que describen qué pasó
- Deben tener una propiedad `type` obligatoria
- Pueden contener datos adicionales (payload)

#### **3. Reducers:**
- Funciones puras que especifican cómo cambia el estado
- Reciben el estado actual y una action
- Devuelven el nuevo estado

#### **4. Middleware:**
- Funciones que interceptan actions antes de llegar al reducer
- Permiten efectos secundarios (API calls, logging, etc.)
- Se ejecutan en cadena

---

## 💻 Implementación Práctica

### **1. Store Principal de la Aplicación**

```javascript:src/store/index.js
import { configureStore } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { createLogger } from 'redux-logger';
import { createTransform } from 'redux-persist';

// Importar slices (reducers)
import authSlice from './slices/authSlice';
import userSlice from './slices/userSlice';
import themeSlice from './slices/themeSlice';
import notificationSlice from './slices/notificationSlice';
import settingsSlice from './slices/settingsSlice';

// Transform personalizado para datos sensibles
const sensitiveDataTransform = createTransform(
  // Transformar al almacenamiento (outbound)
  (inboundState, key) => {
    if (key === 'auth') {
      // Remover datos sensibles antes de persistir
      const { password, token, ...safeState } = inboundState;
      return safeState;
    }
    return inboundState;
  },
  // Transformar desde el almacenamiento (inbound)
  (outboundState, key) => {
    if (key === 'auth') {
      // Restaurar estado pero sin datos sensibles
      return {
        ...outboundState,
        password: '',
        token: null,
        isAuthenticated: false,
      };
    }
    return outboundState;
  },
  { whitelist: ['auth'] }
);

// Configuración de persistencia
const persistConfig = {
  key: 'root', // Clave para el almacenamiento
  storage: AsyncStorage, // Motor de almacenamiento
  whitelist: ['auth', 'user', 'theme', 'settings'], // Solo persistir estos slices
  blacklist: ['notification'], // No persistir notificaciones
  transforms: [sensitiveDataTransform], // Aplicar transformaciones
  timeout: 10000, // Timeout para operaciones de almacenamiento
  debug: __DEV__, // Solo debug en desarrollo
};

// Configuración del logger de Redux
const logger = createLogger({
  collapsed: true, // Colapsar logs por defecto
  duration: true, // Mostrar duración de las actions
  timestamp: true, // Mostrar timestamp
  colors: {
    title: () => '#139BFE', // Color para títulos
    prevState: () => '#9E9E9E', // Color para estado anterior
    action: () => '#149945', // Color para actions
    nextState: () => '#A47104', // Color para siguiente estado
    error: () => '#FF0000', // Color para errores
  },
  predicate: (getState, action) => {
    // Solo loggear en desarrollo y para actions específicas
    if (__DEV__) {
      // Filtrar actions que no queremos loggear
      const blacklistedActions = ['persist/REHYDRATE', 'persist/PERSIST'];
      return !blacklistedActions.includes(action.type);
    }
    return false;
  },
});

// Configuración del store
const store = configureStore({
  // Combinar todos los reducers
  reducer: {
    auth: authSlice,
    user: userSlice,
    theme: themeSlice,
    notification: notificationSlice,
    settings: settingsSlice,
  },
  
  // Middleware personalizado
  middleware: (getDefaultMiddleware) => {
    const defaultMiddleware = getDefaultMiddleware({
      // Configuración del middleware por defecto
      serializableCheck: {
        // Ignorar actions de persistencia
        ignoredActions: [
          'persist/PERSIST',
          'persist/REHYDRATE',
          'persist/PAUSE',
          'persist/PURGE',
          'persist/REGISTER',
          'persist/FLUSH',
        ],
        // Ignorar paths específicos del estado
        ignoredPaths: ['auth.token', 'auth.password'],
      },
      // Configuración de thunk
      thunk: {
        extraArgument: {
          // Argumentos extra para thunks
          api: 'https://api.ejemplo.com',
          version: '1.0.0',
        },
      },
    });
    
    // Agregar logger solo en desarrollo
    if (__DEV__) {
      defaultMiddleware.push(logger);
    }
    
    return defaultMiddleware;
  },
  
  // Configuración del store
  devTools: __DEV__, // Solo en desarrollo
  preloadedState: undefined, // Estado inicial
  enhancers: [], // Enhancers personalizados
});

// Configurar persistencia
const persistor = persistStore(store, {
  manualPersist: false, // Persistencia automática
  debug: __DEV__, // Debug solo en desarrollo
});

// Función para limpiar el store
export const clearStore = () => {
  persistor.purge(); // Limpiar almacenamiento persistente
  store.dispatch({ type: 'RESET_STORE' }); // Resetear estado
};

// Función para obtener estado específico
export const getStateSlice = (sliceName) => {
  const state = store.getState();
  return state[sliceName];
};

// Función para suscribirse a cambios del store
export const subscribeToStore = (callback) => {
  return store.subscribe(callback);
};

// Función para dispatch de actions
export const dispatchAction = (action) => {
  return store.dispatch(action);
};

// Función para obtener el store completo
export const getStore = () => store;

// Función para obtener el persistor
export const getPersistor = () => persistor;

// Tipos de exportación
export { store, persistor };
export default store;
```

### **2. Slice de Autenticación con Redux Toolkit**

```javascript:src/store/slices/authSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Estado inicial del slice de autenticación
const initialState = {
  // Estado del usuario
  user: null,
  token: null,
  refreshToken: null,
  
  // Estado de autenticación
  isAuthenticated: false,
  isInitialized: false,
  isLoading: false,
  
  // Estado de sesión
  sessionExpiry: null,
  lastActivity: null,
  loginAttempts: 0,
  
  // Estado de seguridad
  isLocked: false,
  lockoutTime: null,
  lockoutReason: null,
  
  // Estado de errores
  error: null,
  errorCode: null,
  errorDetails: null,
  
  // Estado de validación
  validationErrors: {},
  fieldErrors: {},
  
  // Estado de biometría
  biometricEnabled: false,
  biometricType: null,
  biometricSupported: false,
  
  // Estado de recordar sesión
  rememberSession: false,
  autoLogin: false,
  
  // Estado de verificación
  requiresVerification: false,
  verificationMethod: null,
  verificationCode: null,
  
  // Estado de recuperación
  recoveryMode: false,
  recoveryEmail: null,
  recoveryPhone: null,
};

// Thunk para inicializar autenticación
export const initializeAuth = createAsyncThunk(
  'auth/initialize',
  async (_, { rejectWithValue }) => {
    try {
      // Intentar recuperar datos de autenticación
      const [storedToken, storedUser, storedRefreshToken] = await Promise.all([
        AsyncStorage.getItem('auth_token'),
        AsyncStorage.getItem('auth_user'),
        AsyncStorage.getItem('auth_refresh_token'),
      ]);
      
      if (storedToken && storedUser) {
        const user = JSON.parse(storedUser);
        const tokenData = JSON.parse(storedToken);
        
        // Verificar si el token no ha expirado
        if (tokenData.expiry && new Date(tokenData.expiry) > new Date()) {
          return {
            user,
            token: tokenData.token,
            refreshToken: storedRefreshToken,
            isAuthenticated: true,
          };
        } else {
          // Token expirado, limpiar almacenamiento
          await Promise.all([
            AsyncStorage.removeItem('auth_token'),
            AsyncStorage.removeItem('auth_user'),
            AsyncStorage.removeItem('auth_refresh_token'),
          ]);
          
          throw new Error('Token expirado');
        }
      }
      
      return {
        user: null,
        token: null,
        refreshToken: null,
        isAuthenticated: false,
      };
    } catch (error) {
      return rejectWithValue({
        message: 'Error al inicializar autenticación',
        details: error.message,
      });
    }
  }
);

// Thunk para iniciar sesión
export const loginUser = createAsyncThunk(
  'auth/login',
  async (credentials, { rejectWithValue, getState }) => {
    try {
      const state = getState();
      
      // Verificar si la cuenta está bloqueada
      if (state.auth.isLocked && state.auth.lockoutTime) {
        const now = new Date();
        const lockoutEnd = new Date(state.auth.lockoutTime);
        
        if (now < lockoutEnd) {
          const remainingTime = Math.ceil((lockoutEnd - now) / 1000 / 60);
          throw new Error(`Cuenta bloqueada. Intente nuevamente en ${remainingTime} minutos.`);
        }
      }
      
      // Simular llamada a API de login
      const response = await simulateLoginAPI(credentials);
      
      if (response.success) {
        const { user, token, refreshToken } = response.data;
        
        // Calcular expiración del token
        const tokenExpiry = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 horas
        
        // Guardar en almacenamiento local
        await Promise.all([
          AsyncStorage.setItem('auth_token', JSON.stringify({
            token,
            expiry: tokenExpiry.toISOString(),
          })),
          AsyncStorage.setItem('auth_user', JSON.stringify(user)),
          AsyncStorage.setItem('auth_refresh_token', refreshToken),
        ]);
        
        return {
          user,
          token,
          refreshToken,
          sessionExpiry: tokenExpiry.toISOString(),
          lastActivity: new Date().toISOString(),
        };
      } else {
        throw new Error(response.error || 'Error de autenticación');
      }
    } catch (error) {
      return rejectWithValue({
        message: error.message,
        code: 'LOGIN_FAILED',
        details: error.stack,
      });
    }
  }
);

// Thunk para cerrar sesión
export const logoutUser = createAsyncThunk(
  'auth/logout',
  async (_, { rejectWithValue }) => {
    try {
      // Limpiar almacenamiento local
      await Promise.all([
        AsyncStorage.removeItem('auth_token'),
        AsyncStorage.removeItem('auth_user'),
        AsyncStorage.removeItem('auth_refresh_token'),
      ]);
      
      return true;
    } catch (error) {
      return rejectWithValue({
        message: 'Error al cerrar sesión',
        details: error.message,
      });
    }
  }
);

// Thunk para renovar token
export const refreshAuthToken = createAsyncThunk(
  'auth/refreshToken',
  async (_, { rejectWithValue, getState }) => {
    try {
      const state = getState();
      const refreshToken = state.auth.refreshToken;
      
      if (!refreshToken) {
        throw new Error('No hay token de renovación disponible');
      }
      
      // Simular llamada a API para renovar token
      const response = await simulateRefreshTokenAPI(refreshToken);
      
      if (response.success) {
        const { token, newRefreshToken } = response.data;
        const tokenExpiry = new Date(Date.now() + 24 * 60 * 60 * 1000);
        
        // Actualizar almacenamiento
        await Promise.all([
          AsyncStorage.setItem('auth_token', JSON.stringify({
            token,
            expiry: tokenExpiry.toISOString(),
          })),
          AsyncStorage.setItem('auth_refresh_token', newRefreshToken),
        ]);
        
        return {
          token,
          refreshToken: newRefreshToken,
          sessionExpiry: tokenExpiry.toISOString(),
        };
      } else {
        throw new Error(response.error || 'Error al renovar token');
      }
    } catch (error) {
      return rejectWithValue({
        message: 'Error al renovar token',
        details: error.message,
      });
    }
  }
);

// Thunk para verificar sesión
export const checkSession = createAsyncThunk(
  'auth/checkSession',
  async (_, { rejectWithValue, getState }) => {
    try {
      const state = getState();
      const { sessionExpiry, lastActivity } = state.auth;
      
      if (!sessionExpiry || !lastActivity) {
        throw new Error('Sesión no válida');
      }
      
      const now = new Date();
      const expiry = new Date(sessionExpiry);
      const lastActivityDate = new Date(lastActivity);
      
      // Verificar si la sesión ha expirado
      if (now > expiry) {
        throw new Error('Sesión expirada');
      }
      
      // Verificar inactividad (30 minutos)
      const inactivityLimit = new Date(lastActivityDate.getTime() + 30 * 60 * 1000);
      if (now > inactivityLimit) {
        throw new Error('Sesión inactiva');
      }
      
      // Actualizar última actividad
      const newLastActivity = new Date().toISOString();
      await AsyncStorage.setItem('auth_last_activity', newLastActivity);
      
      return {
        lastActivity: newLastActivity,
        isValid: true,
      };
    } catch (error) {
      return rejectWithValue({
        message: 'Sesión inválida',
        details: error.message,
      });
    }
  }
);

// Slice de autenticación
const authSlice = createSlice({
  name: 'auth',
  initialState,
  
  // Reducers síncronos
  reducers: {
    // Limpiar errores
    clearErrors: (state) => {
      state.error = null;
      state.errorCode = null;
      state.errorDetails = null;
      state.validationErrors = {};
      state.fieldErrors = {};
    },
    
    // Actualizar usuario
    updateUser: (state, action) => {
      if (state.user) {
        state.user = { ...state.user, ...action.payload };
      }
    },
    
    // Establecer error de validación
    setValidationError: (state, action) => {
      const { field, message } = action.payload;
      state.fieldErrors[field] = message;
    },
    
    // Limpiar error de validación
    clearValidationError: (state, action) => {
      const { field } = action.payload;
      delete state.fieldErrors[field];
    },
    
    // Incrementar intentos de login
    incrementLoginAttempts: (state) => {
      state.loginAttempts += 1;
      
      // Bloquear cuenta después de 5 intentos
      if (state.loginAttempts >= 5) {
        state.isLocked = true;
        state.lockoutTime = new Date(Date.now() + 30 * 60 * 1000).toISOString(); // 30 minutos
        state.lockoutReason = 'Demasiados intentos de login';
      }
    },
    
    // Resetear intentos de login
    resetLoginAttempts: (state) => {
      state.loginAttempts = 0;
      state.isLocked = false;
      state.lockoutTime = null;
      state.lockoutReason = null;
    },
    
    // Desbloquear cuenta
    unlockAccount: (state) => {
      state.isLocked = false;
      state.lockoutTime = null;
      state.lockoutReason = null;
      state.loginAttempts = 0;
    },
    
    // Establecer modo de recuperación
    setRecoveryMode: (state, action) => {
      state.recoveryMode = action.payload.enabled;
      state.recoveryEmail = action.payload.email || null;
      state.recoveryPhone = action.payload.phone || null;
    },
    
    // Establecer verificación requerida
    setVerificationRequired: (state, action) => {
      state.requiresVerification = action.payload.required;
      state.verificationMethod = action.payload.method;
      state.verificationCode = action.payload.code || null;
    },
    
    // Actualizar configuración de biometría
    updateBiometricConfig: (state, action) => {
      const { enabled, type, supported } = action.payload;
      state.biometricEnabled = enabled;
      state.biometricType = type;
      state.biometricSupported = supported;
    },
    
    // Actualizar configuración de sesión
    updateSessionConfig: (state, action) => {
      const { remember, autoLogin } = action.payload;
      state.rememberSession = remember;
      state.autoLogin = autoLogin;
    },
    
    // Actualizar última actividad
    updateLastActivity: (state) => {
      state.lastActivity = new Date().toISOString();
    },
    
    // Resetear estado de autenticación
    resetAuth: () => initialState,
  },
  
  // Extra reducers para thunks
  extraReducers: (builder) => {
    // Inicialización
    builder
      .addCase(initializeAuth.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(initializeAuth.fulfilled, (state, action) => {
        state.isLoading = false;
        state.isInitialized = true;
        state.isAuthenticated = action.payload.isAuthenticated;
        state.user = action.payload.user;
        state.token = action.payload.token;
        state.refreshToken = action.payload.refreshToken;
      })
      .addCase(initializeAuth.rejected, (state, action) => {
        state.isLoading = false;
        state.isInitialized = true;
        state.error = action.payload?.message || 'Error de inicialización';
        state.errorCode = 'INIT_FAILED';
      });
    
    // Login
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
        state.validationErrors = {};
        state.fieldErrors = {};
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.isAuthenticated = true;
        state.user = action.payload.user;
        state.token = action.payload.token;
        state.refreshToken = action.payload.refreshToken;
        state.sessionExpiry = action.payload.sessionExpiry;
        state.lastActivity = action.payload.lastActivity;
        state.loginAttempts = 0;
        state.isLocked = false;
        state.lockoutTime = null;
        state.lockoutReason = null;
        state.error = null;
        state.errorCode = null;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload?.message || 'Error de login';
        state.errorCode = action.payload?.code || 'LOGIN_FAILED';
        state.errorDetails = action.payload?.details;
        
        // Incrementar intentos de login
        state.loginAttempts += 1;
        
        // Bloquear cuenta si es necesario
        if (state.loginAttempts >= 5) {
          state.isLocked = true;
          state.lockoutTime = new Date(Date.now() + 30 * 60 * 1000).toISOString();
          state.lockoutReason = 'Demasiados intentos de login';
        }
      });
    
    // Logout
    builder
      .addCase(logoutUser.fulfilled, (state) => {
        // Resetear todo el estado
        Object.assign(state, initialState);
        state.isInitialized = true;
      })
      .addCase(logoutUser.rejected, (state, action) => {
        state.error = action.payload?.message || 'Error al cerrar sesión';
        state.errorCode = 'LOGOUT_FAILED';
      });
    
    // Renovar token
    builder
      .addCase(refreshAuthToken.fulfilled, (state, action) => {
        state.token = action.payload.token;
        state.refreshToken = action.payload.refreshToken;
        state.sessionExpiry = action.payload.sessionExpiry;
        state.error = null;
        state.errorCode = null;
      })
      .addCase(refreshAuthToken.rejected, (state, action) => {
        state.error = action.payload?.message || 'Error al renovar token';
        state.errorCode = 'REFRESH_FAILED';
        // Forzar logout si no se puede renovar el token
        state.isAuthenticated = false;
        state.user = null;
        state.token = null;
        state.refreshToken = null;
      });
    
    // Verificar sesión
    builder
      .addCase(checkSession.fulfilled, (state, action) => {
        state.lastActivity = action.payload.lastActivity;
        state.error = null;
        state.errorCode = null;
      })
      .addCase(checkSession.rejected, (state, action) => {
        state.error = action.payload?.message || 'Error al verificar sesión';
        state.errorCode = 'SESSION_CHECK_FAILED';
        // Forzar logout si la sesión no es válida
        state.isAuthenticated = false;
        state.user = null;
        state.token = null;
        state.refreshToken = null;
      });
  },
});

// Exportar actions
export const {
  clearErrors,
  updateUser,
  setValidationError,
  clearValidationError,
  incrementLoginAttempts,
  resetLoginAttempts,
  unlockAccount,
  setRecoveryMode,
  setVerificationRequired,
  updateBiometricConfig,
  updateSessionConfig,
  updateLastActivity,
  resetAuth,
} = authSlice.actions;

// Exportar selectors
export const selectAuth = (state) => state.auth;
export const selectUser = (state) => state.auth.user;
export const selectIsAuthenticated = (state) => state.auth.isAuthenticated;
export const selectIsLoading = (state) => state.auth.isLoading;
export const selectToken = (state) => state.auth.token;
export const selectAuthError = (state) => state.auth.error;
export const selectIsLocked = (state) => state.auth.isLocked;
export const selectLoginAttempts = (state) => state.auth.loginAttempts;

// Exportar reducer
export default authSlice.reducer;

// Funciones auxiliares para simular APIs
const simulateLoginAPI = async (credentials) => {
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  if (credentials.email === 'usuario@ejemplo.com' && credentials.password === 'password123') {
    return {
      success: true,
      data: {
        user: {
          id: 1,
          email: credentials.email,
          name: 'Usuario Ejemplo',
          role: 'user',
          avatar: 'https://via.placeholder.com/100',
          createdAt: new Date().toISOString(),
        },
        token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        refreshToken: 'refresh_token_123...',
      },
    };
  } else {
    return {
      success: false,
      error: 'Credenciales inválidas',
    };
  }
};

const simulateRefreshTokenAPI = async (refreshToken) => {
  await new Promise(resolve => setTimeout(resolve, 500));
  
  if (refreshToken) {
    return {
      success: true,
      data: {
        token: 'new_token_456...',
        newRefreshToken: 'new_refresh_token_456...',
      },
    };
  } else {
    return {
      success: false,
      error: 'Token de renovación inválido',
    };
  }
};
```

### **3. Middleware Personalizado para Redux**

```javascript:src/store/middleware/customMiddleware.js
import { createListenerMiddleware } from '@reduxjs/toolkit';

// Middleware para logging personalizado
export const loggingMiddleware = (store) => (next) => (action) => {
  // Loggear antes de procesar la action
  if (__DEV__) {
    console.group(`🚀 Action: ${action.type}`);
    console.log('📤 Action:', action);
    console.log('📊 State anterior:', store.getState());
  }
  
  // Procesar la action
  const result = next(action);
  
  // Loggear después de procesar la action
  if (__DEV__) {
    console.log('📥 State posterior:', store.getState());
    console.groupEnd();
  }
  
  return result;
};

// Middleware para persistencia automática
export const autoPersistMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // Persistir automáticamente ciertas actions
  const actionsToPersist = [
    'auth/loginUser/fulfilled',
    'auth/logoutUser/fulfilled',
    'user/updateProfile/fulfilled',
    'theme/setTheme',
    'settings/updateSettings',
  ];
  
  if (actionsToPersist.includes(action.type)) {
    // Guardar en AsyncStorage
    const state = store.getState();
    const dataToPersist = {
      auth: {
        user: state.auth.user,
        isAuthenticated: state.auth.isAuthenticated,
        biometricEnabled: state.auth.biometricEnabled,
        rememberSession: state.auth.rememberSession,
      },
      user: state.user,
      theme: state.theme,
      settings: state.settings,
    };
    
    AsyncStorage.setItem('redux_persist', JSON.stringify(dataToPersist))
      .catch(error => {
        console.error('Error al persistir estado:', error);
      });
  }
  
  return result;
};

// Middleware para manejo de errores
export const errorHandlingMiddleware = (store) => (next) => (action) => {
  try {
    const result = next(action);
    
    // Verificar si hay errores en el estado
    const state = store.getState();
    const errors = [];
    
    // Verificar errores de autenticación
    if (state.auth.error) {
      errors.push({
        source: 'auth',
        message: state.auth.error,
        code: state.auth.errorCode,
      });
    }
    
    // Verificar errores de usuario
    if (state.user.error) {
      errors.push({
        source: 'user',
        message: state.user.error,
        code: state.user.errorCode,
      });
    }
    
    // Manejar errores si existen
    if (errors.length > 0) {
      handleErrors(errors, store);
    }
    
    return result;
  } catch (error) {
    // Manejar errores del middleware
    console.error('Error en middleware:', error);
    
    // Dispatch action de error global
    store.dispatch({
      type: 'global/setError',
      payload: {
        message: 'Error interno del sistema',
        details: error.message,
        timestamp: new Date().toISOString(),
      },
    });
    
    throw error;
  }
};

// Middleware para analytics
export const analyticsMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // Trackear actions importantes
  const actionsToTrack = [
    'auth/loginUser/fulfilled',
    'auth/logoutUser/fulfilled',
    'user/updateProfile/fulfilled',
    'theme/setTheme',
    'settings/updateSettings',
  ];
  
  if (actionsToTrack.includes(action.type)) {
    trackEvent(action.type, {
      timestamp: new Date().toISOString(),
      userId: store.getState().auth.user?.id,
      actionData: action.payload,
    });
  }
  
  return result;
};

// Middleware para sincronización offline
export const offlineSyncMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // Verificar si hay acciones pendientes de sincronización
  const pendingActions = store.getState().offline?.pendingActions || [];
  
  if (pendingActions.length > 0 && navigator.onLine) {
    // Intentar sincronizar acciones pendientes
    syncPendingActions(pendingActions, store);
  }
  
  return result;
};

// Middleware para validación de estado
export const stateValidationMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // Validar estado después de cada action
  const state = store.getState();
  validateState(state, action);
  
  return result;
};

// Middleware para performance monitoring
export const performanceMiddleware = (store) => (next) => (action) => {
  const startTime = performance.now();
  
  const result = next(action);
  
  const endTime = performance.now();
  const duration = endTime - startTime;
  
  // Loggear actions lentas
  if (duration > 100) { // Más de 100ms
    console.warn(`⚠️ Action lenta detectada: ${action.type} (${duration.toFixed(2)}ms)`);
  }
  
  // Trackear métricas de performance
  trackPerformance(action.type, duration);
  
  return result;
};

// Funciones auxiliares
const handleErrors = (errors, store) => {
  // Agrupar errores por tipo
  const errorGroups = errors.reduce((groups, error) => {
    const group = error.code || 'UNKNOWN';
    if (!groups[group]) {
      groups[group] = [];
    }
    groups[group].push(error);
    return groups;
  }, {});
  
  // Manejar cada grupo de errores
  Object.entries(errorGroups).forEach(([code, groupErrors]) => {
    switch (code) {
      case 'AUTH_FAILED':
        // Manejar errores de autenticación
        handleAuthErrors(groupErrors, store);
        break;
      
      case 'NETWORK_ERROR':
        // Manejar errores de red
        handleNetworkErrors(groupErrors, store);
        break;
      
      case 'VALIDATION_ERROR':
        // Manejar errores de validación
        handleValidationErrors(groupErrors, store);
        break;
      
      default:
        // Manejar errores desconocidos
        handleUnknownErrors(groupErrors, store);
        break;
    }
  });
};

const handleAuthErrors = (errors, store) => {
  // Limpiar estado de autenticación
  store.dispatch({ type: 'auth/clearErrors' });
  
  // Mostrar notificación de error
  store.dispatch({
    type: 'notification/show',
    payload: {
      type: 'error',
      title: 'Error de Autenticación',
      message: errors[0].message,
      duration: 5000,
    },
  });
};

const handleNetworkErrors = (errors, store) => {
  // Marcar como offline
  store.dispatch({
    type: 'global/setOffline',
    payload: { isOffline: true, lastError: errors[0].message },
  });
  
  // Mostrar notificación de error de red
  store.dispatch({
    type: 'notification/show',
    payload: {
      type: 'warning',
      title: 'Error de Conexión',
      message: 'Verifique su conexión a internet',
      duration: 10000,
    },
  });
};

const handleValidationErrors = (errors, store) => {
  // Mostrar errores de validación
  errors.forEach(error => {
    if (error.field) {
      store.dispatch({
        type: 'auth/setValidationError',
        payload: { field: error.field, message: error.message },
      });
    }
  });
};

const handleUnknownErrors = (errors, store) => {
  // Loggear errores desconocidos
  console.error('Errores desconocidos:', errors);
  
  // Mostrar notificación genérica
  store.dispatch({
    type: 'notification/show',
    payload: {
      type: 'error',
      title: 'Error del Sistema',
      message: 'Ha ocurrido un error inesperado',
      duration: 5000,
    },
  });
};

const trackEvent = (eventName, data) => {
  // Implementar tracking de analytics
  if (__DEV__) {
    console.log('📊 Analytics Event:', eventName, data);
  }
  
  // Aquí iría la implementación real de analytics
  // Por ejemplo: Firebase Analytics, Mixpanel, etc.
};

const trackPerformance = (actionType, duration) => {
  // Implementar tracking de performance
  if (__DEV__) {
    console.log('⚡ Performance:', actionType, `${duration.toFixed(2)}ms`);
  }
  
  // Aquí iría la implementación real de performance monitoring
  // Por ejemplo: Sentry, Bugsnag, etc.
};

const syncPendingActions = async (actions, store) => {
  try {
    // Implementar sincronización de acciones pendientes
    console.log('🔄 Sincronizando acciones pendientes:', actions.length);
    
    // Aquí iría la lógica de sincronización real
    
    // Limpiar acciones sincronizadas
    store.dispatch({
      type: 'offline/clearPendingActions',
    });
  } catch (error) {
    console.error('Error al sincronizar acciones:', error);
  }
};

const validateState = (state, action) => {
  // Validar estructura del estado
  const requiredSlices = ['auth', 'user', 'theme', 'settings'];
  
  requiredSlices.forEach(sliceName => {
    if (!state[sliceName]) {
      console.error(`❌ Slice requerido faltante: ${sliceName}`);
    }
  });
  
  // Validar estado de autenticación
  if (state.auth.isAuthenticated && !state.auth.user) {
    console.error('❌ Estado inconsistente: autenticado pero sin usuario');
  }
  
  if (!state.auth.isAuthenticated && state.auth.user) {
    console.error('❌ Estado inconsistente: usuario pero no autenticado');
  }
};

// Exportar todos los middlewares
export const customMiddlewares = [
  loggingMiddleware,
  autoPersistMiddleware,
  errorHandlingMiddleware,
  analyticsMiddleware,
  offlineSyncMiddleware,
  stateValidationMiddleware,
  performanceMiddleware,
];
```

---

## 🧪 Casos de Uso

### **Caso 1: Usar Redux en un Componente**
```javascript
import { useSelector, useDispatch } from 'react-redux';
import { loginUser, selectAuth, selectIsLoading } from '../store/slices/authSlice';

const LoginScreen = () => {
  const dispatch = useDispatch();
  const { error, isAuthenticated } = useSelector(selectAuth);
  const isLoading = useSelector(selectIsLoading);
  
  const handleLogin = (credentials) => {
    dispatch(loginUser(credentials));
  };
  
  return (
    // Componente de login
  );
};
```

### **Caso 2: Middleware Personalizado**
```javascript
// En store/index.js
import { customMiddlewares } from './middleware/customMiddleware';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => [
    ...getDefaultMiddleware(),
    ...customMiddlewares,
  ],
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Store de Usuario**
Implementa un slice para gestionar datos del usuario con Redux Toolkit.

### **Ejercicio 2: Middleware de Logging**
Crea un middleware personalizado para logging de actions y cambios de estado.

### **Ejercicio 3: Persistencia Selectiva**
Implementa persistencia selectiva para ciertos slices del store.

---

## 🚀 Proyecto de la Clase

### **App con Redux Completo**

Crea una aplicación que demuestre:
- **Store centralizado**: Con múltiples slices
- **Actions asíncronas**: Usando createAsyncThunk
- **Middleware personalizado**: Para logging, analytics y manejo de errores
- **Persistencia selectiva**: Solo para datos importantes
- **Validación de estado**: Verificación de consistencia

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Store centralizado**: Estado único y predecible
- **Actions y Reducers**: Patrón unidireccional de datos
- **Redux Toolkit**: Simplificación de la implementación
- **Middleware**: Interceptación y procesamiento de actions
- **Persistencia**: Almacenamiento local del estado

### **Habilidades Desarrolladas:**
- ✅ Implementar store de Redux con múltiples slices
- ✅ Crear actions asíncronas con createAsyncThunk
- ✅ Implementar middleware personalizado
- ✅ Configurar persistencia selectiva del estado
- ✅ Manejar errores y validaciones en Redux

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Hooks Personalizados Avanzados**, que te permitirán crear lógica reutilizable y compleja para tus aplicaciones.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Context API](clase_2_context_api.md)
- **➡️ Siguiente Clase**: [Hooks Personalizados Avanzados](clase_4_hooks_personalizados_avanzados.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
