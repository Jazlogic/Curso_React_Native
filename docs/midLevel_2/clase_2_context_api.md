# 📚 Clase 2: Context API

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 1: Gestión de Estado Básica](clase_1_gestion_estado_basica.md)
- **➡️ Siguiente**: [Clase 3: Redux](clase_3_redux.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el funcionamiento de Context API en React Native
- Aprender a crear y usar contextos para compartir estado
- Dominar la implementación de providers y consumers
- Implementar gestión de estado global con Context API
- Crear aplicaciones con estado compartido eficiente

---

## 📚 Contenido Teórico

### **¿Qué es Context API?**

Context API es una característica de React que permite compartir datos entre componentes sin tener que pasar props manualmente a través de cada nivel de la jerarquía de componentes. Es ideal para datos que se consideran "globales" para un árbol de componentes.

#### **Características principales:**
- **Compartir estado**: Acceso a datos desde cualquier componente
- **Evitar prop drilling**: No pasar props a través de múltiples niveles
- **Estado global**: Datos accesibles en toda la aplicación
- **Reactivo**: Los cambios se propagan automáticamente
- **Nativo de React**: No requiere librerías externas

### **Componentes de Context API:**

#### **1. createContext:**
- Crea un nuevo contexto
- Define la estructura de datos
- Establece valores por defecto

#### **2. Provider:**
- Envuelve componentes que necesitan acceso al contexto
- Proporciona el valor actual del contexto
- Se actualiza cuando cambia el estado

#### **3. Consumer:**
- Consume el valor del contexto
- Se re-renderiza cuando cambia el contexto
- Puede usar hooks o componentes

### **Casos de Uso de Context API:**

✅ **Tema de la aplicación**: Colores, fuentes, estilos
✅ **Autenticación**: Estado del usuario, tokens
✅ **Configuración**: Preferencias, idioma
✅ **Estado de la app**: Modo offline, notificaciones
✅ **Datos compartidos**: Listas, filtros comunes

---

## 💻 Implementación Práctica

### **1. Contexto de Autenticación**

```javascript:src/contexts/AuthContext.js
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Crear el contexto de autenticación
const AuthContext = createContext();

// Estado inicial del contexto
const initialState = {
  user: null, // Usuario actual
  token: null, // Token de autenticación
  isAuthenticated: false, // Estado de autenticación
  isLoading: true, // Estado de carga
  isInitialized: false, // Si ya se inicializó
  error: null, // Errores de autenticación
  lastLogin: null, // Último inicio de sesión
  loginAttempts: 0, // Intentos de login
  isLocked: false, // Si la cuenta está bloqueada
  lockoutTime: null, // Tiempo de bloqueo
};

// Tipos de acciones para el reducer
const AUTH_ACTIONS = {
  // Inicialización
  INITIALIZE: 'INITIALIZE',
  
  // Autenticación
  LOGIN_START: 'LOGIN_START',
  LOGIN_SUCCESS: 'LOGIN_SUCCESS',
  LOGIN_FAILURE: 'LOGIN_FAILURE',
  
  // Cerrar sesión
  LOGOUT: 'LOGOUT',
  
  // Actualizar usuario
  UPDATE_USER: 'UPDATE_USER',
  
  // Limpiar errores
  CLEAR_ERROR: 'CLEAR_ERROR',
  
  // Bloquear cuenta
  LOCK_ACCOUNT: 'LOCK_ACCOUNT',
  UNLOCK_ACCOUNT: 'UNLOCK_ACCOUNT',
  
  // Resetear intentos
  RESET_LOGIN_ATTEMPTS: 'RESET_LOGIN_ATTEMPTS',
};

// Reducer para manejar el estado de autenticación
const authReducer = (state, action) => {
  switch (action.type) {
    case AUTH_ACTIONS.INITIALIZE:
      return {
        ...state,
        isLoading: false,
        isInitialized: true,
        user: action.payload.user,
        token: action.payload.token,
        isAuthenticated: !!action.payload.token,
      };
      
    case AUTH_ACTIONS.LOGIN_START:
      return {
        ...state,
        isLoading: true,
        error: null,
      };
      
    case AUTH_ACTIONS.LOGIN_SUCCESS:
      return {
        ...state,
        isLoading: false,
        isAuthenticated: true,
        user: action.payload.user,
        token: action.payload.token,
        lastLogin: new Date().toISOString(),
        loginAttempts: 0,
        error: null,
        isLocked: false,
        lockoutTime: null,
      };
      
    case AUTH_ACTIONS.LOGIN_FAILURE:
      const newLoginAttempts = state.loginAttempts + 1;
      const isLocked = newLoginAttempts >= 5;
      const lockoutTime = isLocked ? new Date(Date.now() + 30 * 60 * 1000) : null; // 30 minutos
      
      return {
        ...state,
        isLoading: false,
        isAuthenticated: false,
        user: null,
        token: null,
        error: action.payload.error,
        loginAttempts: newLoginAttempts,
        isLocked,
        lockoutTime,
      };
      
    case AUTH_ACTIONS.LOGOUT:
      return {
        ...state,
        isAuthenticated: false,
        user: null,
        token: null,
        lastLogin: null,
        error: null,
      };
      
    case AUTH_ACTIONS.UPDATE_USER:
      return {
        ...state,
        user: {
          ...state.user,
          ...action.payload.updates,
        },
      };
      
    case AUTH_ACTIONS.CLEAR_ERROR:
      return {
        ...state,
        error: null,
      };
      
    case AUTH_ACTIONS.LOCK_ACCOUNT:
      return {
        ...state,
        isLocked: true,
        lockoutTime: action.payload.lockoutTime,
      };
      
    case AUTH_ACTIONS.UNLOCK_ACCOUNT:
      return {
        ...state,
        isLocked: false,
        lockoutTime: null,
        loginAttempts: 0,
      };
      
    case AUTH_ACTIONS.RESET_LOGIN_ATTEMPTS:
      return {
        ...state,
        loginAttempts: 0,
        isLocked: false,
        lockoutTime: null,
      };
      
    default:
      return state;
  }
};

// Provider del contexto de autenticación
export const AuthProvider = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);
  
  // Función para inicializar el contexto
  const initializeAuth = async () => {
    try {
      // Intentar recuperar datos de autenticación del almacenamiento
      const [storedToken, storedUser] = await Promise.all([
        AsyncStorage.getItem('auth_token'),
        AsyncStorage.getItem('auth_user'),
      ]);
      
      if (storedToken && storedUser) {
        const user = JSON.parse(storedUser);
        
        // Verificar si el token no ha expirado
        if (user.tokenExpiry && new Date(user.tokenExpiry) > new Date()) {
          dispatch({
            type: AUTH_ACTIONS.INITIALIZE,
            payload: { user, token: storedToken },
          });
        } else {
          // Token expirado, limpiar almacenamiento
          await Promise.all([
            AsyncStorage.removeItem('auth_token'),
            AsyncStorage.removeItem('auth_user'),
          ]);
          
          dispatch({
            type: AUTH_ACTIONS.INITIALIZE,
            payload: { user: null, token: null },
          });
        }
      } else {
        dispatch({
          type: AUTH_ACTIONS.INITIALIZE,
          payload: { user: null, token: null },
        });
      }
    } catch (error) {
      console.error('Error al inicializar autenticación:', error);
      dispatch({
        type: AUTH_ACTIONS.INITIALIZE,
        payload: { user: null, token: null },
      });
    }
  };
  
  // Función para iniciar sesión
  const login = async (credentials) => {
    try {
      dispatch({ type: AUTH_ACTIONS.LOGIN_START });
      
      // Verificar si la cuenta está bloqueada
      if (state.isLocked && state.lockoutTime) {
        const now = new Date();
        const lockoutEnd = new Date(state.lockoutTime);
        
        if (now < lockoutEnd) {
          const remainingTime = Math.ceil((lockoutEnd - now) / 1000 / 60);
          throw new Error(`Cuenta bloqueada. Intente nuevamente en ${remainingTime} minutos.`);
        } else {
          // Desbloquear cuenta si ya pasó el tiempo
          dispatch({ type: AUTH_ACTIONS.UNLOCK_ACCOUNT });
        }
      }
      
      // Simular llamada a API de login
      const response = await simulateLoginAPI(credentials);
      
      if (response.success) {
        const { user, token } = response.data;
        
        // Guardar en almacenamiento local
        await Promise.all([
          AsyncStorage.setItem('auth_token', token),
          AsyncStorage.setItem('auth_user', JSON.stringify({
            ...user,
            tokenExpiry: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(), // 24 horas
          })),
        ]);
        
        dispatch({
          type: AUTH_ACTIONS.LOGIN_SUCCESS,
          payload: { user, token },
        });
        
        return { success: true };
      } else {
        throw new Error(response.error || 'Error de autenticación');
      }
    } catch (error) {
      dispatch({
        type: AUTH_ACTIONS.LOGIN_FAILURE,
        payload: { error: error.message },
      });
      
      return { success: false, error: error.message };
    }
  };
  
  // Función para cerrar sesión
  const logout = async () => {
    try {
      // Limpiar almacenamiento local
      await Promise.all([
        AsyncStorage.removeItem('auth_token'),
        AsyncStorage.removeItem('auth_user'),
      ]);
      
      dispatch({ type: AUTH_ACTIONS.LOGOUT });
    } catch (error) {
      console.error('Error al cerrar sesión:', error);
    }
  };
  
  // Función para actualizar datos del usuario
  const updateUser = async (updates) => {
    try {
      // Simular llamada a API para actualizar usuario
      const response = await simulateUpdateUserAPI(state.token, updates);
      
      if (response.success) {
        const updatedUser = { ...state.user, ...updates };
        
        // Actualizar almacenamiento local
        await AsyncStorage.setItem('auth_user', JSON.stringify(updatedUser));
        
        dispatch({
          type: AUTH_ACTIONS.UPDATE_USER,
          payload: { updates },
        });
        
        return { success: true };
      } else {
        throw new Error(response.error || 'Error al actualizar usuario');
      }
    } catch (error) {
      console.error('Error al actualizar usuario:', error);
      return { success: false, error: error.message };
    }
  };
  
  // Función para limpiar errores
  const clearError = () => {
    dispatch({ type: AUTH_ACTIONS.CLEAR_ERROR });
  };
  
  // Función para desbloquear cuenta manualmente
  const unlockAccount = () => {
    dispatch({ type: AUTH_ACTIONS.UNLOCK_ACCOUNT });
  };
  
  // Función para resetear intentos de login
  const resetLoginAttempts = () => {
    dispatch({ type: AUTH_ACTIONS.RESET_LOGIN_ATTEMPTS });
  };
  
  // Función para verificar si la cuenta está bloqueada
  const checkAccountLocked = () => {
    if (state.isLocked && state.lockoutTime) {
      const now = new Date();
      const lockoutEnd = new Date(state.lockoutTime);
      
      if (now >= lockoutEnd) {
        // Desbloquear automáticamente si ya pasó el tiempo
        dispatch({ type: AUTH_ACTIONS.UNLOCK_ACCOUNT });
        return false;
      }
      
      return true;
    }
    
    return false;
  };
  
  // Función para obtener tiempo restante de bloqueo
  const getRemainingLockoutTime = () => {
    if (state.isLocked && state.lockoutTime) {
      const now = new Date();
      const lockoutEnd = new Date(state.lockoutTime);
      const remaining = Math.ceil((lockoutEnd - now) / 1000 / 60);
      return Math.max(0, remaining);
    }
    return 0;
  };
  
  // Efecto para inicializar la autenticación
  useEffect(() => {
    initializeAuth();
  }, []);
  
  // Efecto para verificar bloqueo de cuenta
  useEffect(() => {
    if (state.isLocked && state.lockoutTime) {
      const interval = setInterval(() => {
        checkAccountLocked();
      }, 60000); // Verificar cada minuto
      
      return () => clearInterval(interval);
    }
  }, [state.isLocked, state.lockoutTime]);
  
  // Valor del contexto
  const contextValue = {
    // Estado
    ...state,
    
    // Funciones
    login,
    logout,
    updateUser,
    clearError,
    unlockAccount,
    resetLoginAttempts,
    checkAccountLocked,
    getRemainingLockoutTime,
    
    // Constantes
    AUTH_ACTIONS,
  };
  
  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
};

// Hook personalizado para usar el contexto de autenticación
export const useAuth = () => {
  const context = useContext(AuthContext);
  
  if (!context) {
    throw new Error('useAuth debe ser usado dentro de un AuthProvider');
  }
  
  return context;
};

// Función para simular API de login
const simulateLoginAPI = async (credentials) => {
  // Simular delay de red
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  // Simular validación de credenciales
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
      },
    };
  } else {
    return {
      success: false,
      error: 'Credenciales inválidas',
    };
  }
};

// Función para simular API de actualización de usuario
const simulateUpdateUserAPI = async (token, updates) => {
  // Simular delay de red
  await new Promise(resolve => setTimeout(resolve, 500));
  
  // Simular validación de token
  if (token) {
    return {
      success: true,
      data: updates,
    };
  } else {
    return {
      success: false,
      error: 'Token inválido',
    };
  }
};

export default AuthContext;
```

### **2. Contexto de Tema de la Aplicación**

```javascript:src/contexts/ThemeContext.js
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Crear el contexto de tema
const ThemeContext = createContext();

// Temas disponibles
const THEMES = {
  light: {
    name: 'light',
    colors: {
      primary: '#007bff',
      secondary: '#6c757d',
      success: '#28a745',
      danger: '#dc3545',
      warning: '#ffc107',
      info: '#17a2b8',
      light: '#f8f9fa',
      dark: '#343a40',
      white: '#ffffff',
      black: '#000000',
      gray: '#6c757d',
      grayLight: '#f8f9fa',
      grayDark: '#343a40',
      border: '#dee2e6',
      background: '#ffffff',
      surface: '#ffffff',
      text: '#212529',
      textSecondary: '#6c757d',
      textMuted: '#6c757d',
      shadow: 'rgba(0, 0, 0, 0.1)',
      overlay: 'rgba(0, 0, 0, 0.5)',
    },
    fonts: {
      regular: 'System',
      medium: 'System',
      bold: 'System',
      light: 'System',
    },
    sizes: {
      xs: 12,
      sm: 14,
      md: 16,
      lg: 18,
      xl: 20,
      xxl: 24,
      xxxl: 32,
    },
    spacing: {
      xs: 4,
      sm: 8,
      md: 16,
      lg: 24,
      xl: 32,
      xxl: 48,
    },
    borderRadius: {
      sm: 4,
      md: 8,
      lg: 12,
      xl: 16,
      round: 50,
    },
    shadows: {
      sm: {
        shadowColor: 'rgba(0, 0, 0, 0.1)',
        shadowOffset: { width: 0, height: 1 },
        shadowOpacity: 0.2,
        shadowRadius: 2,
        elevation: 2,
      },
      md: {
        shadowColor: 'rgba(0, 0, 0, 0.1)',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 4,
        elevation: 4,
      },
      lg: {
        shadowColor: 'rgba(0, 0, 0, 0.1)',
        shadowOffset: { width: 0, height: 4 },
        shadowOpacity: 0.3,
        shadowRadius: 8,
        elevation: 8,
      },
    },
  },
  
  dark: {
    name: 'dark',
    colors: {
      primary: '#4dabf7',
      secondary: '#adb5bd',
      success: '#69db7c',
      danger: '#ff6b6b',
      warning: '#ffd43b',
      info: '#74c0fc',
      light: '#495057',
      dark: '#f8f9fa',
      white: '#212529',
      black: '#ffffff',
      gray: '#adb5bd',
      grayLight: '#495057',
      grayDark: '#f8f9fa',
      border: '#495057',
      background: '#121212',
      surface: '#1e1e1e',
      text: '#f8f9fa',
      textSecondary: '#adb5bd',
      textMuted: '#6c757d',
      shadow: 'rgba(0, 0, 0, 0.3)',
      overlay: 'rgba(0, 0, 0, 0.7)',
    },
    fonts: {
      regular: 'System',
      medium: 'System',
      bold: 'System',
      light: 'System',
    },
    sizes: {
      xs: 12,
      sm: 14,
      md: 16,
      lg: 18,
      xl: 20,
      xxl: 24,
      xxxl: 32,
    },
    spacing: {
      xs: 4,
      sm: 8,
      md: 16,
      lg: 24,
      xl: 32,
      xxl: 48,
    },
    borderRadius: {
      sm: 4,
      md: 8,
      lg: 12,
      xl: 16,
      round: 50,
    },
    shadows: {
      sm: {
        shadowColor: 'rgba(0, 0, 0, 0.3)',
        shadowOffset: { width: 0, height: 1 },
        shadowOpacity: 0.4,
        shadowRadius: 2,
        elevation: 2,
      },
      md: {
        shadowColor: 'rgba(0, 0, 0, 0.3)',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.5,
        shadowRadius: 4,
        elevation: 4,
      },
      lg: {
        shadowColor: 'rgba(0, 0, 0, 0.3)',
        shadowOffset: { width: 0, height: 4 },
        shadowOpacity: 0.6,
        shadowRadius: 8,
        elevation: 8,
      },
    },
  },
};

// Estado inicial
const initialState = {
  currentTheme: 'light',
  theme: THEMES.light,
  isDarkMode: false,
  autoTheme: true, // Cambiar automáticamente según hora del día
  customColors: {}, // Colores personalizados
  fontSize: 'md', // Tamaño de fuente base
  highContrast: false, // Modo alto contraste
  reducedMotion: false, // Reducir animaciones
};

// Tipos de acciones
const THEME_ACTIONS = {
  SET_THEME: 'SET_THEME',
  TOGGLE_THEME: 'TOGGLE_THEME',
  SET_AUTO_THEME: 'SET_AUTO_THEME',
  SET_CUSTOM_COLORS: 'SET_CUSTOM_COLORS',
  SET_FONT_SIZE: 'SET_FONT_SIZE',
  TOGGLE_HIGH_CONTRAST: 'TOGGLE_HIGH_CONTRAST',
  TOGGLE_REDUCED_MOTION: 'TOGGLE_REDUCED_MOTION',
  RESET_THEME: 'RESET_THEME',
};

// Reducer del tema
const themeReducer = (state, action) => {
  switch (action.type) {
    case THEME_ACTIONS.SET_THEME:
      const themeName = action.payload;
      return {
        ...state,
        currentTheme: themeName,
        theme: THEMES[themeName],
        isDarkMode: themeName === 'dark',
      };
      
    case THEME_ACTIONS.TOGGLE_THEME:
      const newTheme = state.currentTheme === 'light' ? 'dark' : 'light';
      return {
        ...state,
        currentTheme: newTheme,
        theme: THEMES[newTheme],
        isDarkMode: newTheme === 'dark',
      };
      
    case THEME_ACTIONS.SET_AUTO_THEME:
      return {
        ...state,
        autoTheme: action.payload,
      };
      
    case THEME_ACTIONS.SET_CUSTOM_COLORS:
      return {
        ...state,
        customColors: {
          ...state.customColors,
          ...action.payload,
        },
      };
      
    case THEME_ACTIONS.SET_FONT_SIZE:
      return {
        ...state,
        fontSize: action.payload,
      };
      
    case THEME_ACTIONS.TOGGLE_HIGH_CONTRAST:
      return {
        ...state,
        highContrast: !state.highContrast,
      };
      
    case THEME_ACTIONS.TOGGLE_REDUCED_MOTION:
      return {
        ...state,
        reducedMotion: !state.reducedMotion,
      };
      
    case THEME_ACTIONS.RESET_THEME:
      return {
        ...initialState,
        currentTheme: 'light',
        theme: THEMES.light,
      };
      
    default:
      return state;
  }
};

// Provider del contexto de tema
export const ThemeProvider = ({ children }) => {
  const [state, dispatch] = useReducer(themeReducer, initialState);
  
  // Función para cambiar tema
  const setTheme = (themeName) => {
    if (THEMES[themeName]) {
      dispatch({ type: THEME_ACTIONS.SET_THEME, payload: themeName });
    }
  };
  
  // Función para alternar tema
  const toggleTheme = () => {
    dispatch({ type: THEME_ACTIONS.TOGGLE_THEME });
  };
  
  // Función para establecer auto tema
  const setAutoTheme = (enabled) => {
    dispatch({ type: THEME_ACTIONS.SET_AUTO_THEME, payload: enabled });
  };
  
  // Función para establecer colores personalizados
  const setCustomColors = (colors) => {
    dispatch({ type: THEME_ACTIONS.SET_CUSTOM_COLORS, payload: colors });
  };
  
  // Función para establecer tamaño de fuente
  const setFontSize = (size) => {
    if (['xs', 'sm', 'md', 'lg', 'xl', 'xxl', 'xxxl'].includes(size)) {
      dispatch({ type: THEME_ACTIONS.SET_FONT_SIZE, payload: size });
    }
  };
  
  // Función para alternar alto contraste
  const toggleHighContrast = () => {
    dispatch({ type: THEME_ACTIONS.TOGGLE_HIGH_CONTRAST });
  };
  
  // Función para alternar movimiento reducido
  const toggleReducedMotion = () => {
    dispatch({ type: THEME_ACTIONS.TOGGLE_REDUCED_MOTION });
  };
  
  // Función para resetear tema
  const resetTheme = () => {
    dispatch({ type: THEME_ACTIONS.RESET_THEME });
  };
  
  // Función para obtener tema con colores personalizados
  const getThemeWithCustomColors = () => {
    if (Object.keys(state.customColors).length === 0) {
      return state.theme;
    }
    
    return {
      ...state.theme,
      colors: {
        ...state.theme.colors,
        ...state.customColors,
      },
    };
  };
  
  // Función para obtener tema con alto contraste
  const getThemeWithHighContrast = () => {
    const theme = getThemeWithCustomColors();
    
    if (!state.highContrast) {
      return theme;
    }
    
    // Aplicar colores de alto contraste
    return {
      ...theme,
      colors: {
        ...theme.colors,
        text: '#000000',
        background: '#ffffff',
        surface: '#ffffff',
        border: '#000000',
        primary: '#000000',
        secondary: '#000000',
      },
    };
  };
  
  // Función para detectar tema automático según hora del día
  const detectAutoTheme = () => {
    if (!state.autoTheme) return;
    
    const hour = new Date().getHours();
    const isDayTime = hour >= 6 && hour < 18;
    const newTheme = isDayTime ? 'light' : 'dark';
    
    if (newTheme !== state.currentTheme) {
      setTheme(newTheme);
    }
  };
  
  // Efecto para cargar tema guardado
  useEffect(() => {
    const loadSavedTheme = async () => {
      try {
        const savedTheme = await AsyncStorage.getItem('app_theme');
        if (savedTheme) {
          const themeData = JSON.parse(savedTheme);
          setTheme(themeData.currentTheme);
          
          if (themeData.customColors) {
            setCustomColors(themeData.customColors);
          }
          
          if (themeData.fontSize) {
            setFontSize(themeData.fontSize);
          }
          
          if (themeData.highContrast !== undefined) {
            if (themeData.highContrast) {
              toggleHighContrast();
            }
          }
          
          if (themeData.reducedMotion !== undefined) {
            if (themeData.reducedMotion) {
              toggleReducedMotion();
            }
          }
        }
      } catch (error) {
        console.error('Error al cargar tema guardado:', error);
      }
    };
    
    loadSavedTheme();
  }, []);
  
  // Efecto para guardar cambios de tema
  useEffect(() => {
    const saveTheme = async () => {
      try {
        const themeData = {
          currentTheme: state.currentTheme,
          customColors: state.customColors,
          fontSize: state.fontSize,
          highContrast: state.highContrast,
          reducedMotion: state.reducedMotion,
        };
        
        await AsyncStorage.setItem('app_theme', JSON.stringify(themeData));
      } catch (error) {
        console.error('Error al guardar tema:', error);
      }
    };
    
    saveTheme();
  }, [state.currentTheme, state.customColors, state.fontSize, state.highContrast, state.reducedMotion]);
  
  // Efecto para detectar tema automático
  useEffect(() => {
    if (state.autoTheme) {
      detectAutoTheme();
      
      const interval = setInterval(detectAutoTheme, 60000); // Verificar cada minuto
      return () => clearInterval(interval);
    }
  }, [state.autoTheme]);
  
  // Valor del contexto
  const contextValue = {
    // Estado
    ...state,
    
    // Funciones
    setTheme,
    toggleTheme,
    setAutoTheme,
    setCustomColors,
    setFontSize,
    toggleHighContrast,
    toggleReducedMotion,
    resetTheme,
    getThemeWithCustomColors,
    getThemeWithHighContrast,
    
    // Constantes
    THEMES,
    THEME_ACTIONS,
  };
  
  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
};

// Hook personalizado para usar el contexto de tema
export const useTheme = () => {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('useTheme debe ser usado dentro de un ThemeProvider');
  }
  
  return context;
};

export default ThemeContext;
```

### **3. Hook Personalizado para Contextos**

```javascript:src/hooks/useContextState.js
import { useContext, useCallback, useMemo } from 'react';

// Hook personalizado para usar contextos de manera más eficiente
const useContextState = (context, selector = null) => {
  const contextValue = useContext(context);
  
  // Si no hay selector, devolver todo el contexto
  if (!selector) {
    return contextValue;
  }
  
  // Si hay selector, devolver solo los valores seleccionados
  const selectedValues = useMemo(() => {
    if (typeof selector === 'function') {
      return selector(contextValue);
    }
    
    if (Array.isArray(selector)) {
      return selector.reduce((acc, key) => {
        if (contextValue.hasOwnProperty(key)) {
          acc[key] = contextValue[key];
        }
        return acc;
      }, {});
    }
    
    return {};
  }, [contextValue, selector]);
  
  // Función para actualizar contexto
  const updateContext = useCallback((updates) => {
    if (contextValue.updateContext) {
      contextValue.updateContext(updates);
    }
  }, [contextValue]);
  
  // Función para resetear contexto
  const resetContext = useCallback(() => {
    if (contextValue.resetContext) {
      contextValue.resetContext();
    }
  }, [contextValue]);
  
  return {
    ...selectedValues,
    updateContext,
    resetContext,
    fullContext: contextValue,
  };
};

export default useContextState;
```

---

## 🧪 Casos de Uso

### **Caso 1: Usar Contexto de Autenticación**
```javascript
// En un componente
const { user, isAuthenticated, login, logout } = useAuth();

// Verificar autenticación
if (isAuthenticated) {
  // Usuario autenticado
}
```

### **Caso 2: Usar Contexto de Tema**
```javascript
// En un componente
const { theme, toggleTheme, isDarkMode } = useTheme();

// Aplicar tema
const styles = StyleSheet.create({
  container: {
    backgroundColor: theme.colors.background,
  },
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Contexto de Usuario**
Implementa un contexto para gestionar datos del usuario.

### **Ejercicio 2: Contexto de Configuración**
Crea un contexto para configuraciones de la aplicación.

### **Ejercicio 3: Contexto de Notificaciones**
Desarrolla un contexto para gestionar notificaciones.

---

## 🚀 Proyecto de la Clase

### **App con Contextos Múltiples**

Crea una aplicación que demuestre:
- **Contexto de autenticación**: Login/logout y gestión de usuario
- **Contexto de tema**: Cambio entre temas claro/oscuro
- **Contexto de configuración**: Preferencias de la aplicación
- **Integración de contextos**: Uso combinado de múltiples contextos

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Context API**: Compartir estado entre componentes
- **Providers y Consumers**: Patrón para distribución de datos
- **Estado global**: Acceso a datos desde cualquier lugar
- **Hooks personalizados**: Simplificar el uso de contextos

### **Habilidades Desarrolladas:**
- ✅ Crear y usar contextos en React Native
- ✅ Implementar providers y consumers
- ✅ Gestionar estado global con Context API
- ✅ Crear hooks personalizados para contextos
- ✅ Integrar múltiples contextos en una aplicación

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Redux**, que te permitirá gestionar estado de manera más avanzada y predecible.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Gestión de Estado Básica](clase_1_gestion_estado_basica.md)
- **➡️ Siguiente Clase**: [Redux](clase_3_redux.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
