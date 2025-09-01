# Clase 4: Autenticación y Autorización

## Navegación
- **Anterior**: [Clase 3: Manejo de Errores y Caché](./clase_3_manejo_errores_cache.md)
- **Siguiente**: [Clase 5: Optimización y Performance](./clase_5_optimizacion_performance.md)
- **Módulo**: [Módulo 6: APIs y Networking](./README.md)
- **Índice**: [Índice Completo](./../INDICE_COMPLETO.md)
- **Navegación Rápida**: [Navegación Rápida](./../NAVEGACION_RAPIDA.md)

## 🎯 Objetivos de la Clase
- Implementar sistemas de autenticación robustos en React Native
- Manejar diferentes tipos de tokens (JWT, OAuth, API Keys)
- Implementar autorización basada en roles y permisos
- Gestionar sesiones y renovación automática de tokens
- Implementar seguridad adicional (biometría, 2FA)

## 📚 Contenido Teórico

### 1. Conceptos Fundamentales de Seguridad

#### Autenticación vs Autorización
- **Autenticación**: Verificar la identidad del usuario ("¿Quién eres?")
- **Autorización**: Verificar los permisos del usuario ("¿Qué puedes hacer?")

#### Tipos de Autenticación
```javascript
// Diferentes métodos de autenticación
const authMethods = {
  CREDENTIALS: 'username/password',
  TOKEN: 'JWT, OAuth, API Key',
  BIOMETRIC: 'huella dactilar, Face ID',
  TWO_FACTOR: 'SMS, email, authenticator app',
  SOCIAL: 'Google, Facebook, Apple',
  SSO: 'Single Sign-On empresarial',
};
```

### 2. JWT (JSON Web Tokens)

#### Estructura de un JWT
```javascript
// Un JWT tiene 3 partes separadas por puntos
// header.payload.signature

// Ejemplo de payload decodificado
const jwtPayload = {
  // Claims estándar
  iss: 'https://myapp.com',        // Issuer (emisor)
  sub: 'user123',                  // Subject (usuario)
  aud: 'myapp',                    // Audience (audiencia)
  exp: 1640995200,                 // Expiration (expiración)
  iat: 1640908800,                 // Issued At (emitido en)
  nbf: 1640908800,                 // Not Before (no válido antes de)
  
  // Claims personalizados
  userId: 'user123',
  email: 'user@example.com',
  role: 'admin',
  permissions: ['read', 'write', 'delete'],
};
```

#### Servicio de JWT
```javascript
// services/jwtService.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import jwt_decode from 'jwt-decode';

class JWTService {
  constructor() {
    this.accessTokenKey = 'accessToken';
    this.refreshTokenKey = 'refreshToken';
    this.tokenExpiryKey = 'tokenExpiry';
  }
  
  // Guardar tokens
  async setTokens(accessToken, refreshToken) {
    try {
      const decoded = jwt_decode(accessToken);
      const expiryTime = decoded.exp * 1000; // Convertir a milisegundos
      
      await AsyncStorage.multiSet([
        [this.accessTokenKey, accessToken],
        [this.refreshTokenKey, refreshToken],
        [this.tokenExpiryKey, expiryTime.toString()],
      ]);
      
      return true;
    } catch (error) {
      console.error('Error saving tokens:', error);
      return false;
    }
  }
  
  // Obtener token de acceso
  async getAccessToken() {
    try {
      return await AsyncStorage.getItem(this.accessTokenKey);
    } catch (error) {
      console.error('Error getting access token:', error);
      return null;
    }
  }
  
  // Obtener token de refresh
  async getRefreshToken() {
    try {
      return await AsyncStorage.getItem(this.refreshTokenKey);
    } catch (error) {
      console.error('Error getting refresh token:', error);
      return null;
    }
  }
  
  // Verificar si el token está expirado
  async isTokenExpired() {
    try {
      const expiryTime = await AsyncStorage.getItem(this.tokenExpiryKey);
      if (!expiryTime) return true;
      
      const currentTime = Date.now();
      const tokenExpiry = parseInt(expiryTime);
      
      // Considerar expirado si falta menos de 5 minutos
      return currentTime >= (tokenExpiry - (5 * 60 * 1000));
    } catch (error) {
      console.error('Error checking token expiry:', error);
      return true;
    }
  }
  
  // Verificar si el token está próximo a expirar
  async isTokenExpiringSoon(minutes = 5) {
    try {
      const expiryTime = await AsyncStorage.getItem(this.tokenExpiryKey);
      if (!expiryTime) return true;
      
      const currentTime = Date.now();
      const tokenExpiry = parseInt(expiryTime);
      const threshold = minutes * 60 * 1000;
      
      return currentTime >= (tokenExpiry - threshold);
    } catch (error) {
      console.error('Error checking token expiry:', error);
      return true;
    }
  }
  
  // Decodificar token
  decodeToken(token) {
    try {
      return jwt_decode(token);
    } catch (error) {
      console.error('Error decoding token:', error);
      return null;
    }
  }
  
  // Obtener información del usuario del token
  async getUserInfo() {
    try {
      const token = await this.getAccessToken();
      if (!token) return null;
      
      const decoded = this.decodeToken(token);
      if (!decoded) return null;
      
      return {
        userId: decoded.userId,
        email: decoded.email,
        role: decoded.role,
        permissions: decoded.permissions || [],
        exp: decoded.exp,
      };
    } catch (error) {
      console.error('Error getting user info:', error);
      return null;
    }
  }
  
  // Limpiar tokens
  async clearTokens() {
    try {
      await AsyncStorage.multiRemove([
        this.accessTokenKey,
        this.refreshTokenKey,
        this.tokenExpiryKey,
      ]);
      return true;
    } catch (error) {
      console.error('Error clearing tokens:', error);
      return false;
    }
  }
  
  // Verificar si el usuario está autenticado
  async isAuthenticated() {
    try {
      const token = await this.getAccessToken();
      if (!token) return false;
      
      return !(await this.isTokenExpired());
    } catch (error) {
      console.error('Error checking authentication:', error);
      return false;
    }
  }
}

export default new JWTService();
```

### 3. Servicio de Autenticación

#### Clase de Autenticación
```javascript
// services/authService.js
import JWTService from './jwtService';
import api from './api';

class AuthService {
  constructor() {
    this.isRefreshing = false;
    this.failedQueue = [];
    this.processQueue = this.processQueue.bind(this);
  }
  
  // Login con credenciales
  async login(credentials) {
    try {
      const response = await api.post('/auth/login', credentials);
      const { accessToken, refreshToken, user } = response.data;
      
      // Guardar tokens
      await JWTService.setTokens(accessToken, refreshToken);
      
      return { success: true, user };
    } catch (error) {
      console.error('Login error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Logout
  async logout() {
    try {
      // Llamar al endpoint de logout si es necesario
      const refreshToken = await JWTService.getRefreshToken();
      if (refreshToken) {
        try {
          await api.post('/auth/logout', { refreshToken });
        } catch (error) {
          console.warn('Logout endpoint failed, continuing with local cleanup');
        }
      }
      
      // Limpiar tokens locales
      await JWTService.clearTokens();
      
      return { success: true };
    } catch (error) {
      console.error('Logout error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Refrescar token
  async refreshToken() {
    try {
      const refreshToken = await JWTService.getRefreshToken();
      if (!refreshToken) {
        throw new Error('No refresh token available');
      }
      
      const response = await api.post('/auth/refresh', { refreshToken });
      const { accessToken, newRefreshToken } = response.data;
      
      // Guardar nuevos tokens
      await JWTService.setTokens(accessToken, newRefreshToken);
      
      return { success: true, accessToken };
    } catch (error) {
      console.error('Token refresh error:', error);
      
      // Si falla el refresh, limpiar tokens y redirigir a login
      await JWTService.clearTokens();
      
      return { success: false, error: error.message };
    }
  }
  
  // Registrar usuario
  async register(userData) {
    try {
      const response = await api.post('/auth/register', userData);
      const { accessToken, refreshToken, user } = response.data;
      
      // Guardar tokens
      await JWTService.setTokens(accessToken, refreshToken);
      
      return { success: true, user };
    } catch (error) {
      console.error('Registration error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Cambiar contraseña
  async changePassword(currentPassword, newPassword) {
    try {
      const response = await api.post('/auth/change-password', {
        currentPassword,
        newPassword,
      });
      
      return { success: true, message: response.data.message };
    } catch (error) {
      console.error('Change password error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Solicitar reset de contraseña
  async requestPasswordReset(email) {
    try {
      const response = await api.post('/auth/forgot-password', { email });
      return { success: true, message: response.data.message };
    } catch (error) {
      console.error('Password reset request error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Resetear contraseña
  async resetPassword(token, newPassword) {
    try {
      const response = await api.post('/auth/reset-password', {
        token,
        newPassword,
      });
      
      return { success: true, message: response.data.message };
    } catch (error) {
      console.error('Password reset error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Verificar email
  async verifyEmail(token) {
    try {
      const response = await api.post('/auth/verify-email', { token });
      return { success: true, message: response.data.message };
    } catch (error) {
      console.error('Email verification error:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Procesar cola de peticiones fallidas
  processQueue(error, token = null) {
    this.failedQueue.forEach(({ resolve, reject }) => {
      if (error) {
        reject(error);
      } else {
        resolve(token);
      }
    });
    
    this.failedQueue = [];
  }
  
  // Agregar petición a la cola
  addToQueue(resolve, reject) {
    this.failedQueue.push({ resolve, reject });
  }
}

export default new AuthService();
```

### 4. Middleware de Autenticación para Axios

#### Interceptor de Autenticación
```javascript
// middleware/authInterceptor.js
import JWTService from '../services/jwtService';
import AuthService from '../services/authService';

class AuthInterceptor {
  constructor(api) {
    this.api = api;
    this.setupInterceptors();
  }
  
  // Configurar interceptores
  setupInterceptors() {
    // Request interceptor
    this.api.interceptors.request.use(
      async (config) => {
        // Agregar token a todas las peticiones
        const token = await JWTService.getAccessToken();
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        
        return config;
      },
      (error) => {
        return Promise.reject(error);
      }
    );
    
    // Response interceptor
    this.api.interceptors.response.use(
      (response) => {
        return response;
      },
      async (error) => {
        const originalRequest = error.config;
        
        // Si es error 401 y no se ha intentado refresh
        if (error.response?.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;
          
          try {
            // Intentar refrescar token
            const result = await AuthService.refreshToken();
            
            if (result.success) {
              // Reintentar petición original con nuevo token
              const newToken = await JWTService.getAccessToken();
              originalRequest.headers.Authorization = `Bearer ${newToken}`;
              
              return this.api(originalRequest);
            } else {
              // Si falla el refresh, redirigir a login
              this.handleAuthFailure();
              return Promise.reject(error);
            }
          } catch (refreshError) {
            // Si falla el refresh, redirigir a login
            this.handleAuthFailure();
            return Promise.reject(error);
          }
        }
        
        return Promise.reject(error);
      }
    );
  }
  
  // Manejar fallo de autenticación
  handleAuthFailure() {
    // Limpiar tokens
    JWTService.clearTokens();
    
    // Emitir evento para que la app redirija a login
    if (this.onAuthFailure) {
      this.onAuthFailure();
    }
  }
  
  // Configurar callback de fallo de autenticación
  setAuthFailureCallback(callback) {
    this.onAuthFailure = callback;
  }
}

export default AuthInterceptor;
```

### 5. Sistema de Autorización

#### Clase de Autorización
```javascript
// services/authorizationService.js
import JWTService from './jwtService';

class AuthorizationService {
  constructor() {
    this.roleHierarchy = {
      user: 1,
      moderator: 2,
      admin: 3,
      superadmin: 4,
    };
    
    this.permissionMatrix = {
      user: ['read:own', 'write:own'],
      moderator: ['read:own', 'write:own', 'read:all', 'moderate'],
      admin: ['read:all', 'write:all', 'delete:all', 'manage_users'],
      superadmin: ['read:all', 'write:all', 'delete:all', 'manage_users', 'system_config'],
    };
  }
  
  // Verificar si el usuario tiene un rol específico
  async hasRole(requiredRole) {
    try {
      const userInfo = await JWTService.getUserInfo();
      if (!userInfo) return false;
      
      const userRoleLevel = this.roleHierarchy[userInfo.role] || 0;
      const requiredRoleLevel = this.roleHierarchy[requiredRole] || 0;
      
      return userRoleLevel >= requiredRoleLevel;
    } catch (error) {
      console.error('Error checking role:', error);
      return false;
    }
  }
  
  // Verificar si el usuario tiene un permiso específico
  async hasPermission(requiredPermission) {
    try {
      const userInfo = await JWTService.getUserInfo();
      if (!userInfo) return false;
      
      const userPermissions = userInfo.permissions || [];
      const rolePermissions = this.permissionMatrix[userInfo.role] || [];
      
      // Combinar permisos del token y del rol
      const allPermissions = [...userPermissions, ...rolePermissions];
      
      return allPermissions.includes(requiredPermission);
    } catch (error) {
      console.error('Error checking permission:', error);
      return false;
    }
  }
  
  // Verificar si el usuario puede acceder a un recurso
  async canAccessResource(resourceType, resourceId, action) {
    try {
      const userInfo = await JWTService.getUserInfo();
      if (!userInfo) return false;
      
      // Superadmin puede acceder a todo
      if (userInfo.role === 'superadmin') return true;
      
      // Admin puede acceder a todo excepto configuración del sistema
      if (userInfo.role === 'admin' && resourceType !== 'system') return true;
      
      // Moderador puede acceder a recursos públicos y moderar
      if (userInfo.role === 'moderator') {
        if (action === 'moderate') return true;
        if (resourceType === 'public') return true;
      }
      
      // Usuario normal solo puede acceder a sus propios recursos
      if (userInfo.role === 'user') {
        if (action === 'read:own' || action === 'write:own') {
          // Verificar si el recurso pertenece al usuario
          return this.isResourceOwner(resourceType, resourceId, userInfo.userId);
        }
      }
      
      return false;
    } catch (error) {
      console.error('Error checking resource access:', error);
      return false;
    }
  }
  
  // Verificar si el usuario es dueño del recurso
  async isResourceOwner(resourceType, resourceId, userId) {
    try {
      // Esta función debería consultar la base de datos
      // Por ahora, asumimos que el resourceId contiene el userId
      return resourceId.toString().includes(userId.toString());
    } catch (error) {
      console.error('Error checking resource ownership:', error);
      return false;
    }
  }
  
  // Obtener permisos del usuario
  async getUserPermissions() {
    try {
      const userInfo = await JWTService.getUserInfo();
      if (!userInfo) return [];
      
      const rolePermissions = this.permissionMatrix[userInfo.role] || [];
      const tokenPermissions = userInfo.permissions || [];
      
      return [...new Set([...rolePermissions, ...tokenPermissions])];
    } catch (error) {
      console.error('Error getting user permissions:', error);
      return [];
    }
  }
  
  // Verificar si el usuario puede realizar una acción en un recurso
  async canPerformAction(action, resourceType, resourceId) {
    try {
      const userInfo = await JWTService.getUserInfo();
      if (!userInfo) return false;
      
      // Construir el permiso requerido
      const requiredPermission = `${action}:${resourceType}`;
      
      // Verificar permiso básico
      if (!(await this.hasPermission(requiredPermission))) {
        return false;
      }
      
      // Verificar acceso al recurso específico
      return await this.canAccessResource(resourceType, resourceId, requiredPermission);
    } catch (error) {
      console.error('Error checking action permission:', error);
      return false;
    }
  }
}

export default new AuthorizationService();
```

### 6. Hook de Autenticación

#### Hook useAuth
```javascript
// hooks/useAuth.js
import { useState, useEffect, useCallback } from 'react';
import JWTService from '../services/jwtService';
import AuthService from '../services/authService';
import AuthorizationService from '../services/authorizationService';

export const useAuth = () => {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  const [permissions, setPermissions] = useState([]);

  // Verificar estado de autenticación
  const checkAuthStatus = useCallback(async () => {
    try {
      setIsLoading(true);
      
      const authenticated = await JWTService.isAuthenticated();
      setIsAuthenticated(authenticated);
      
      if (authenticated) {
        const userInfo = await JWTService.getUserInfo();
        const userPermissions = await AuthorizationService.getUserPermissions();
        
        setUser(userInfo);
        setPermissions(userPermissions);
      } else {
        setUser(null);
        setPermissions([]);
      }
    } catch (error) {
      console.error('Error checking auth status:', error);
      setIsAuthenticated(false);
      setUser(null);
      setPermissions([]);
    } finally {
      setIsLoading(false);
    }
  }, []);

  // Login
  const login = useCallback(async (credentials) => {
    try {
      const result = await AuthService.login(credentials);
      
      if (result.success) {
        await checkAuthStatus();
        return { success: true, user: result.user };
      } else {
        return { success: false, error: result.error };
      }
    } catch (error) {
      console.error('Login error:', error);
      return { success: false, error: error.message };
    }
  }, [checkAuthStatus]);

  // Logout
  const logout = useCallback(async () => {
    try {
      const result = await AuthService.logout();
      
      if (result.success) {
        setUser(null);
        setIsAuthenticated(false);
        setPermissions([]);
        return { success: true };
      } else {
        return { success: false, error: result.error };
      }
    } catch (error) {
      console.error('Logout error:', error);
      return { success: false, error: error.message };
    }
  }, []);

  // Verificar rol
  const hasRole = useCallback(async (role) => {
    return await AuthorizationService.hasRole(role);
  }, []);

  // Verificar permiso
  const hasPermission = useCallback(async (permission) => {
    return await AuthorizationService.hasPermission(permission);
  }, []);

  // Verificar acceso a recurso
  const canAccessResource = useCallback(async (resourceType, resourceId, action) => {
    return await AuthorizationService.canAccessResource(resourceType, resourceId, action);
  }, []);

  // Verificar acción
  const canPerformAction = useCallback(async (action, resourceType, resourceId) => {
    return await AuthorizationService.canPerformAction(action, resourceType, resourceId);
  }, []);

  // Efecto para verificar estado inicial
  useEffect(() => {
    checkAuthStatus();
  }, [checkAuthStatus]);

  return {
    user,
    isAuthenticated,
    isLoading,
    permissions,
    login,
    logout,
    hasRole,
    hasPermission,
    canAccessResource,
    canPerformAction,
    checkAuthStatus,
  };
};
```

### 7. Componente de Protección de Rutas

#### HOC para Protección de Rutas
```javascript
// components/ProtectedRoute.js
import React from 'react';
import { View, Text, ActivityIndicator } from 'react-native';
import { useAuth } from '../hooks/useAuth';

const ProtectedRoute = ({ 
  children, 
  requiredRole = null, 
  requiredPermission = null,
  fallback = null,
  loadingComponent = null 
}) => {
  const { 
    isAuthenticated, 
    isLoading, 
    user, 
    hasRole, 
    hasPermission 
  } = useAuth();

  // Mostrar loading mientras se verifica autenticación
  if (isLoading) {
    return loadingComponent || (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" color="#0000ff" />
        <Text>Verificando autenticación...</Text>
      </View>
    );
  }

  // Si no está autenticado, mostrar fallback o redirigir
  if (!isAuthenticated) {
    return fallback || (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>Debes iniciar sesión para acceder a esta página</Text>
      </View>
    );
  }

  // Si se requiere un rol específico
  if (requiredRole && !hasRole(requiredRole)) {
    return fallback || (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>No tienes permisos para acceder a esta página</Text>
        <Text>Rol requerido: {requiredRole}</Text>
        <Text>Tu rol: {user?.role}</Text>
      </View>
    );
  }

  // Si se requiere un permiso específico
  if (requiredPermission && !hasPermission(requiredPermission)) {
    return fallback || (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>No tienes permisos para acceder a esta página</Text>
        <Text>Permiso requerido: {requiredPermission}</Text>
      </View>
    );
  }

  // Si pasa todas las verificaciones, mostrar el contenido
  return children;
};

export default ProtectedRoute;
```

#### Uso del Componente Protegido
```javascript
// Ejemplo de uso en navegación
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import ProtectedRoute from '../components/ProtectedRoute';
import LoginScreen from '../screens/LoginScreen';
import DashboardScreen from '../screens/DashboardScreen';
import AdminScreen from '../screens/AdminScreen';

const Stack = createStackNavigator();

const AppNavigator = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Login" component={LoginScreen} />
        
        <Stack.Screen name="Dashboard">
          {props => (
            <ProtectedRoute>
              <DashboardScreen {...props} />
            </ProtectedRoute>
          )}
        </Stack.Screen>
        
        <Stack.Screen name="Admin">
          {props => (
            <ProtectedRoute requiredRole="admin">
              <AdminScreen {...props} />
            </ProtectedRoute>
          )}
        </Stack.Screen>
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default AppNavigator;
```

## 🧪 Ejercicios Prácticos

### Ejercicio 1: Sistema de Roles Avanzado
Implementa un sistema de roles con las siguientes características:

```javascript
// El sistema debe incluir:
// - Roles anidados (role inheritance)
// - Permisos granulares por recurso
// - Permisos temporales (time-based)
// - Auditoría de accesos
// - Rate limiting por rol
```

### Ejercicio 2: Autenticación Biométrica
Implementa autenticación biométrica en React Native:

```javascript
// Debes implementar:
// - Detección de capacidades biométricas
// - Autenticación con huella dactilar
// - Autenticación con Face ID
// - Fallback a PIN/password
// - Integración con el sistema de JWT
```

### Ejercicio 3: Sistema de 2FA
Crea un sistema de autenticación de dos factores:

```javascript
// El sistema debe incluir:
// - Generación de códigos TOTP
// - Escaneo de códigos QR
// - Backup codes
// - Notificaciones push
// - Integración con Google Authenticator
```

## 📝 Resumen de la Clase

### **Conceptos Clave Aprendidos:**
1. **JWT**: Estructura, claims, manejo de expiración y refresh automático
2. **Autenticación**: Login, logout, registro y manejo de sesiones
3. **Autorización**: Roles, permisos y control de acceso a recursos
4. **Seguridad**: Middleware de autenticación y protección de rutas
5. **Hooks**: Hook personalizado para gestión de estado de autenticación

### **Habilidades Desarrolladas:**
- Implementar sistemas de autenticación robustos con JWT
- Crear middleware de autenticación para Axios
- Implementar autorización basada en roles y permisos
- Proteger rutas y componentes según permisos del usuario
- Gestionar tokens y renovación automática

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Optimización y Performance**, que nos permitirá:
- Implementar lazy loading y code splitting
- Optimizar re-renders con React.memo y useMemo
- Implementar virtualización para listas grandes
- Optimizar imágenes y assets
- Implementar profiling y métricas de performance

---

**💡 Consejo**: La seguridad es fundamental en aplicaciones móviles. Siempre valida tanto en el cliente como en el servidor, y nunca confíes únicamente en la validación del lado del cliente. Implementa logging de seguridad para detectar intentos de acceso no autorizado.
