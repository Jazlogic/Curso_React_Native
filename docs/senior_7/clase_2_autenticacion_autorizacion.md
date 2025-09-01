# Clase 2: Autenticación y Autorización 🔐

## Información de la Clase

- **Módulo**: 14 - Seguridad
- **Clase**: 2 de 5
- **Duración**: 2 horas
- **Tipo**: Teórica + Práctica

## Navegación

- **Anterior**: [Clase 1: Fundamentos de Seguridad](clase_1_fundamentos_seguridad.md)
- **Siguiente**: [Clase 3: Encriptación y Protección de Datos](clase_3_encriptacion_proteccion_datos.md)
- **Arriba**: [README del Módulo](README.md)

## Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Implementar** sistemas de autenticación robustos con JWT
2. **Configurar** flujos de OAuth para terceros
3. **Gestionar** roles y permisos de usuario
4. **Implementar** sistemas de sesión seguros
5. **Aplicar** mejores prácticas de autenticación

## Contenido Teórico

### 1. Fundamentos de Autenticación

#### A. **¿Qué es la Autenticación?**

La autenticación es el proceso de verificar la identidad de un usuario. En React Native, esto incluye:

```typescript
// Flujo básico de autenticación
interface AuthenticationFlow {
  // 1. Usuario ingresa credenciales
  credentials: {
    username: string;
    password: string;
  };
  
  // 2. App valida credenciales
  validation: {
    isValid: boolean;
    errors: string[];
  };
  
  // 3. Servidor verifica y responde
  serverResponse: {
    success: boolean;
    token?: string;
    user?: UserData;
    error?: string;
  };
  
  // 4. App almacena token y actualiza estado
  appState: {
    isAuthenticated: boolean;
    user: UserData | null;
    token: string | null;
  };
}
```

#### B. **Tipos de Autenticación**

```typescript
// Enumeración de tipos de autenticación
enum AuthenticationType {
  // Autenticación local (credenciales)
  LOCAL = 'LOCAL',
  
  // Autenticación biométrica
  BIOMETRIC = 'BIOMETRIC',
  
  // Autenticación de dos factores
  TWO_FACTOR = 'TWO_FACTOR',
  
  // Autenticación social (OAuth)
  SOCIAL = 'SOCIAL',
  
  // Autenticación por token
  TOKEN = 'TOKEN',
}

// Configuración de autenticación
interface AuthConfig {
  type: AuthenticationType;
  requireBiometric: boolean;
  requireTwoFactor: boolean;
  sessionTimeout: number;
  maxLoginAttempts: number;
}
```

### 2. Implementación de JWT (JSON Web Tokens)

#### A. **Estructura de JWT**

```typescript
// Estructura de un JWT
interface JWTStructure {
  // Header: Algoritmo y tipo de token
  header: {
    alg: 'HS256' | 'RS256' | 'ES256';
    typ: 'JWT';
  };
  
  // Payload: Datos del token
  payload: {
    // Claims estándar
    iss?: string;        // Issuer (emisor)
    sub: string;         // Subject (usuario)
    aud?: string;        // Audience (audiencia)
    exp: number;         // Expiration (expiración)
    iat: number;         // Issued At (emitido en)
    nbf?: number;        // Not Before (no válido antes de)
    
    // Claims personalizados
    userId: string;
    role: UserRole;
    permissions: string[];
  };
  
  // Signature: Firma digital
  signature: string;
}

// Clase para manejo de JWT
class JWTManager {
  private secretKey: string;
  private algorithm: string;
  
  constructor(secretKey: string, algorithm: string = 'HS256') {
    this.secretKey = secretKey;
    this.algorithm = algorithm;
  }
  
  // Generar token JWT
  generateToken(payload: any): string {
    const header = {
      alg: this.algorithm,
      typ: 'JWT',
    };
    
    const now = Math.floor(Date.now() / 1000);
    const tokenPayload = {
      ...payload,
      iat: now,
      exp: now + (60 * 60 * 24), // 24 horas
    };
    
    // En producción, usar librería como jsonwebtoken
    const encodedHeader = btoa(JSON.stringify(header));
    const encodedPayload = btoa(JSON.stringify(tokenPayload));
    
    // Simular firma (en producción usar HMAC o RSA)
    const signature = this.createSignature(encodedHeader, encodedPayload);
    
    return `${encodedHeader}.${encodedPayload}.${signature}`;
  }
  
  // Verificar token JWT
  verifyToken(token: string): any {
    try {
      const parts = token.split('.');
      if (parts.length !== 3) {
        throw new Error('Invalid token format');
      }
      
      const [header, payload, signature] = parts;
      
      // Verificar firma
      const expectedSignature = this.createSignature(header, payload);
      if (signature !== expectedSignature) {
        throw new Error('Invalid signature');
      }
      
      // Verificar expiración
      const decodedPayload = JSON.parse(atob(payload));
      if (decodedPayload.exp < Math.floor(Date.now() / 1000)) {
        throw new Error('Token expired');
      }
      
      return decodedPayload;
    } catch (error) {
      throw new Error(`Token verification failed: ${error.message}`);
    }
  }
  
  // Crear firma (simulada)
  private createSignature(header: string, payload: string): string {
    // En producción, usar HMAC-SHA256 o similar
    return btoa(`${header}.${payload}.${this.secretKey}`);
  }
}
```

#### B. **Manejo de Tokens en React Native**

```typescript
// Clase para gestión de tokens
class TokenManager {
  private storage: AsyncStorage;
  private refreshTokenEndpoint: string;
  
  constructor(storage: AsyncStorage, refreshEndpoint: string) {
    this.storage = storage;
    this.refreshTokenEndpoint = refreshEndpoint;
  }
  
  // Guardar tokens
  async saveTokens(accessToken: string, refreshToken: string): Promise<void> {
    try {
      await this.storage.setItem('accessToken', accessToken);
      await this.storage.setItem('refreshToken', refreshToken);
      
      // Guardar timestamp de expiración
      const decoded = this.decodeToken(accessToken);
      if (decoded.exp) {
        await this.storage.setItem('tokenExpiry', decoded.exp.toString());
      }
    } catch (error) {
      console.error('Error saving tokens:', error);
      throw error;
    }
  }
  
  // Obtener token de acceso
  async getAccessToken(): Promise<string | null> {
    try {
      const token = await this.storage.getItem('accessToken');
      if (!token) return null;
      
      // Verificar si está expirado
      if (await this.isTokenExpired(token)) {
        return await this.refreshAccessToken();
      }
      
      return token;
    } catch (error) {
      console.error('Error getting access token:', error);
      return null;
    }
  }
  
  // Verificar si el token está expirado
  private async isTokenExpired(token: string): Promise<boolean> {
    try {
      const expiry = await this.storage.getItem('tokenExpiry');
      if (!expiry) return true;
      
      const expiryTime = parseInt(expiry) * 1000; // Convertir a milisegundos
      return Date.now() >= expiryTime;
    } catch {
      return true;
    }
  }
  
  // Refrescar token de acceso
  private async refreshAccessToken(): Promise<string | null> {
    try {
      const refreshToken = await this.storage.getItem('refreshToken');
      if (!refreshToken) return null;
      
      const response = await fetch(this.refreshTokenEndpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ refreshToken }),
      });
      
      if (!response.ok) {
        throw new Error('Failed to refresh token');
      }
      
      const { accessToken, newRefreshToken } = await response.json();
      
      // Guardar nuevos tokens
      await this.saveTokens(accessToken, newRefreshToken);
      
      return accessToken;
    } catch (error) {
      console.error('Error refreshing token:', error);
      await this.clearTokens();
      return null;
    }
  }
  
  // Limpiar tokens
  async clearTokens(): Promise<void> {
    try {
      await this.storage.removeItem('accessToken');
      await this.storage.removeItem('refreshToken');
      await this.storage.removeItem('tokenExpiry');
    } catch (error) {
      console.error('Error clearing tokens:', error);
    }
  }
  
  // Decodificar token (sin verificación)
  private decodeToken(token: string): any {
    try {
      const payload = token.split('.')[1];
      return JSON.parse(atob(payload));
    } catch {
      return {};
    }
  }
}
```

### 3. Implementación de OAuth

#### A. **Flujo de OAuth 2.0**

```typescript
// Configuración de OAuth
interface OAuthConfig {
  clientId: string;
  clientSecret: string;
  redirectUri: string;
  scope: string[];
  authorizationEndpoint: string;
  tokenEndpoint: string;
  userInfoEndpoint: string;
}

// Clase para manejo de OAuth
class OAuthManager {
  private config: OAuthConfig;
  private authState: string;
  
  constructor(config: OAuthConfig) {
    this.config = config;
    this.authState = this.generateRandomState();
  }
  
  // Generar estado aleatorio para seguridad
  private generateRandomState(): string {
    return Math.random().toString(36).substring(2, 15);
  }
  
  // Iniciar flujo de autorización
  async initiateAuth(): Promise<string> {
    const params = new URLSearchParams({
      client_id: this.config.clientId,
      redirect_uri: this.config.redirectUri,
      scope: this.config.scope.join(' '),
      response_type: 'code',
      state: this.authState,
    });
    
    const authUrl = `${this.config.authorizationEndpoint}?${params.toString()}`;
    return authUrl;
  }
  
  // Intercambiar código por token
  async exchangeCodeForToken(authCode: string, state: string): Promise<OAuthTokens> {
    // Verificar estado para prevenir CSRF
    if (state !== this.authState) {
      throw new Error('Invalid state parameter');
    }
    
    const response = await fetch(this.config.tokenEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: new URLSearchParams({
        grant_type: 'authorization_code',
        client_id: this.config.clientId,
        client_secret: this.config.clientSecret,
        redirect_uri: this.config.redirectUri,
        code: authCode,
      }),
    });
    
    if (!response.ok) {
      throw new Error('Failed to exchange code for token');
    }
    
    return await response.json();
  }
  
  // Obtener información del usuario
  async getUserInfo(accessToken: string): Promise<UserProfile> {
    const response = await fetch(this.config.userInfoEndpoint, {
      headers: {
        'Authorization': `Bearer ${accessToken}`,
      },
    });
    
    if (!response.ok) {
      throw new Error('Failed to get user info');
    }
    
    return await response.json();
  }
  
  // Refrescar token
  async refreshToken(refreshToken: string): Promise<OAuthTokens> {
    const response = await fetch(this.config.tokenEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        client_id: this.config.clientId,
        client_secret: this.config.clientSecret,
        refresh_token: refreshToken,
      }),
    });
    
    if (!response.ok) {
      throw new Error('Failed to refresh token');
    }
    
    return await response.json();
  }
}

// Tipos para OAuth
interface OAuthTokens {
  access_token: string;
  refresh_token: string;
  expires_in: number;
  token_type: string;
  scope: string;
}

interface UserProfile {
  id: string;
  email: string;
  name: string;
  picture?: string;
  verified_email: boolean;
}
```

#### B. **Integración con Proveedores Sociales**

```typescript
// Configuración para diferentes proveedores
const OAuthProviders = {
  GOOGLE: {
    clientId: 'your-google-client-id',
    clientSecret: 'your-google-client-secret',
    authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
    tokenEndpoint: 'https://oauth2.googleapis.com/token',
    userInfoEndpoint: 'https://www.googleapis.com/oauth2/v2/userinfo',
    scope: ['openid', 'email', 'profile'],
  },
  
  FACEBOOK: {
    clientId: 'your-facebook-client-id',
    clientSecret: 'your-facebook-client-secret',
    authorizationEndpoint: 'https://www.facebook.com/v12.0/dialog/oauth',
    tokenEndpoint: 'https://graph.facebook.com/v12.0/oauth/access_token',
    userInfoEndpoint: 'https://graph.facebook.com/me',
    scope: ['email', 'public_profile'],
  },
  
  APPLE: {
    clientId: 'your-apple-client-id',
    clientSecret: 'your-apple-client-secret',
    authorizationEndpoint: 'https://appleid.apple.com/auth/authorize',
    tokenEndpoint: 'https://appleid.apple.com/auth/token',
    userInfoEndpoint: 'https://appleid.apple.com/auth/userinfo',
    scope: ['name', 'email'],
  },
} as const;

// Clase para autenticación social
class SocialAuthManager {
  private providers: Map<string, OAuthManager>;
  
  constructor() {
    this.providers = new Map();
    
    // Inicializar proveedores
    Object.entries(OAuthProviders).forEach(([name, config]) => {
      this.providers.set(name, new OAuthManager(config));
    });
  }
  
  // Autenticación con proveedor específico
  async authenticateWithProvider(providerName: string): Promise<AuthResult> {
    const provider = this.providers.get(providerName);
    if (!provider) {
      throw new Error(`Provider ${providerName} not supported`);
    }
    
    try {
      // Iniciar flujo de autorización
      const authUrl = await provider.initiateAuth();
      
      // En React Native, usar Linking para abrir URL
      const supported = await Linking.canOpenURL(authUrl);
      if (!supported) {
        throw new Error('Cannot open OAuth URL');
      }
      
      // Abrir URL de autorización
      await Linking.openURL(authUrl);
      
      // Nota: En una implementación real, necesitarías manejar
      // el callback de OAuth usando deep linking o URL schemes
      
      return {
        success: true,
        message: 'OAuth flow initiated',
      };
    } catch (error) {
      return {
        success: false,
        error: error.message,
      };
    }
  }
  
  // Obtener proveedor por nombre
  getProvider(providerName: string): OAuthManager | undefined {
    return this.providers.get(providerName);
  }
  
  // Listar proveedores disponibles
  getAvailableProviders(): string[] {
    return Array.from(this.providers.keys());
  }
}
```

### 4. Gestión de Roles y Permisos

#### A. **Sistema de Roles**

```typescript
// Definición de roles y permisos
enum UserRole {
  GUEST = 'GUEST',
  USER = 'USER',
  MODERATOR = 'MODERATOR',
  ADMIN = 'ADMIN',
  SUPER_ADMIN = 'SUPER_ADMIN',
}

// Permisos disponibles
enum Permission {
  // Permisos de lectura
  READ_PUBLIC_CONTENT = 'READ_PUBLIC_CONTENT',
  READ_PRIVATE_CONTENT = 'READ_PRIVATE_CONTENT',
  READ_USER_PROFILES = 'READ_USER_PROFILES',
  
  // Permisos de escritura
  CREATE_CONTENT = 'CREATE_CONTENT',
  UPDATE_OWN_CONTENT = 'UPDATE_OWN_CONTENT',
  UPDATE_ANY_CONTENT = 'UPDATE_ANY_CONTENT',
  
  // Permisos de administración
  DELETE_CONTENT = 'DELETE_CONTENT',
  MANAGE_USERS = 'MANAGE_USERS',
  MANAGE_SYSTEM = 'MANAGE_SYSTEM',
  
  // Permisos especiales
  MODERATE_CONTENT = 'MODERATE_CONTENT',
  VIEW_ANALYTICS = 'VIEW_ANALYTICS',
  EXPORT_DATA = 'EXPORT_DATA',
}

// Mapeo de roles a permisos
const RolePermissions: Record<UserRole, Permission[]> = {
  [UserRole.GUEST]: [
    Permission.READ_PUBLIC_CONTENT,
  ],
  
  [UserRole.USER]: [
    Permission.READ_PUBLIC_CONTENT,
    Permission.READ_PRIVATE_CONTENT,
    Permission.CREATE_CONTENT,
    Permission.UPDATE_OWN_CONTENT,
  ],
  
  [UserRole.MODERATOR]: [
    Permission.READ_PUBLIC_CONTENT,
    Permission.READ_PRIVATE_CONTENT,
    Permission.READ_USER_PROFILES,
    Permission.CREATE_CONTENT,
    Permission.UPDATE_OWN_CONTENT,
    Permission.UPDATE_ANY_CONTENT,
    Permission.MODERATE_CONTENT,
    Permission.VIEW_ANALYTICS,
  ],
  
  [UserRole.ADMIN]: [
    Permission.READ_PUBLIC_CONTENT,
    Permission.READ_PRIVATE_CONTENT,
    Permission.READ_USER_PROFILES,
    Permission.CREATE_CONTENT,
    Permission.UPDATE_OWN_CONTENT,
    Permission.UPDATE_ANY_CONTENT,
    Permission.DELETE_CONTENT,
    Permission.MANAGE_USERS,
    Permission.MODERATE_CONTENT,
    Permission.VIEW_ANALYTICS,
    Permission.EXPORT_DATA,
  ],
  
  [UserRole.SUPER_ADMIN]: [
    ...Object.values(Permission), // Todos los permisos
  ],
};

// Clase para gestión de permisos
class PermissionManager {
  private userRole: UserRole;
  private userPermissions: Permission[];
  
  constructor(userRole: UserRole) {
    this.userRole = userRole;
    this.userPermissions = RolePermissions[userRole] || [];
  }
  
  // Verificar si el usuario tiene un permiso específico
  hasPermission(permission: Permission): boolean {
    return this.userPermissions.includes(permission);
  }
  
  // Verificar si el usuario tiene múltiples permisos
  hasAnyPermission(permissions: Permission[]): boolean {
    return permissions.some(permission => this.hasPermission(permission));
  }
  
  // Verificar si el usuario tiene todos los permisos
  hasAllPermissions(permissions: Permission[]): boolean {
    return permissions.every(permission => this.hasPermission(permission));
  }
  
  // Obtener rol del usuario
  getUserRole(): UserRole {
    return this.userRole;
  }
  
  // Obtener permisos del usuario
  getUserPermissions(): Permission[] {
    return [...this.userPermissions];
  }
  
  // Verificar si el usuario puede realizar una acción
  canPerformAction(action: string): boolean {
    // Mapeo de acciones a permisos
    const actionPermissions: Record<string, Permission[]> = {
      'view_profile': [Permission.READ_USER_PROFILES],
      'edit_profile': [Permission.UPDATE_OWN_CONTENT],
      'create_post': [Permission.CREATE_CONTENT],
      'edit_post': [Permission.UPDATE_OWN_CONTENT, Permission.UPDATE_ANY_CONTENT],
      'delete_post': [Permission.DELETE_CONTENT],
      'moderate_comments': [Permission.MODERATE_CONTENT],
      'manage_users': [Permission.MANAGE_USERS],
      'view_analytics': [Permission.VIEW_ANALYTICS],
      'export_data': [Permission.EXPORT_DATA],
    };
    
    const requiredPermissions = actionPermissions[action];
    if (!requiredPermissions) return false;
    
    return this.hasAnyPermission(requiredPermissions);
  }
}
```

#### B. **Middleware de Autorización**

```typescript
// Middleware para verificar permisos en componentes
const withPermission = (requiredPermission: Permission) => {
  return (WrappedComponent: React.ComponentType<any>) => {
    return (props: any) => {
      const { userRole } = useAuth(); // Hook personalizado
      const permissionManager = new PermissionManager(userRole);
      
      if (!permissionManager.hasPermission(requiredPermission)) {
        return <AccessDenied />;
      }
      
      return <WrappedComponent {...props} />;
    };
  };
};

// Hook personalizado para autenticación
const useAuth = () => {
  const [authState, setAuthState] = useState<AuthState>({
    isAuthenticated: false,
    user: null,
    token: null,
    loading: true,
  });
  
  const login = useCallback(async (credentials: LoginCredentials) => {
    try {
      setAuthState(prev => ({ ...prev, loading: true }));
      
      const response = await authService.login(credentials);
      
      if (response.success) {
        setAuthState({
          isAuthenticated: true,
          user: response.user,
          token: response.token,
          loading: false,
        });
        
        // Guardar tokens
        await tokenManager.saveTokens(response.token, response.refreshToken);
      } else {
        throw new Error(response.error);
      }
    } catch (error) {
      setAuthState(prev => ({ ...prev, loading: false }));
      throw error;
    }
  }, []);
  
  const logout = useCallback(async () => {
    try {
      await tokenManager.clearTokens();
      setAuthState({
        isAuthenticated: false,
        user: null,
        token: null,
        loading: false,
      });
    } catch (error) {
      console.error('Error during logout:', error);
    }
  }, []);
  
  const checkAuth = useCallback(async () => {
    try {
      const token = await tokenManager.getAccessToken();
      if (token) {
        const user = await authService.getCurrentUser(token);
        setAuthState({
          isAuthenticated: true,
          user,
          token,
          loading: false,
        });
      } else {
        setAuthState(prev => ({ ...prev, loading: false }));
      }
    } catch (error) {
      setAuthState(prev => ({ ...prev, loading: false }));
    }
  }, []);
  
  useEffect(() => {
    checkAuth();
  }, [checkAuth]);
  
  return {
    ...authState,
    login,
    logout,
    checkAuth,
  };
};
```

### 5. Gestión de Sesiones Seguras

#### A. **Configuración de Sesión**

```typescript
// Configuración de sesión
interface SessionConfig {
  timeout: number;           // Timeout en milisegundos
  maxRetries: number;        // Máximo de intentos de login
  lockOnBackground: boolean; // Bloquear al ir a background
  requireBiometric: boolean; // Requerir biometría para desbloquear
  autoLogout: boolean;       // Logout automático al expirar
}

// Clase para gestión de sesiones
class SessionManager {
  private config: SessionConfig;
  private tokenManager: TokenManager;
  private lastActivity: number;
  private activityTimer: NodeJS.Timeout | null;
  
  constructor(config: SessionConfig, tokenManager: TokenManager) {
    this.config = config;
    this.tokenManager = tokenManager;
    this.lastActivity = Date.now();
    this.activityTimer = null;
    
    this.setupActivityTracking();
  }
  
  // Configurar seguimiento de actividad
  private setupActivityTracking(): void {
    // Actualizar timestamp de última actividad
    const updateActivity = () => {
      this.lastActivity = Date.now();
    };
    
    // Eventos que indican actividad del usuario
    const events = ['touchstart', 'scroll', 'keypress', 'click'];
    events.forEach(event => {
      document.addEventListener(event, updateActivity, true);
    });
    
    // Verificar timeout de sesión
    this.startSessionTimeoutCheck();
  }
  
  // Iniciar verificación de timeout de sesión
  private startSessionTimeoutCheck(): void {
    this.activityTimer = setInterval(() => {
      this.checkSessionTimeout();
    }, 60000); // Verificar cada minuto
  }
  
  // Verificar si la sesión ha expirado
  private checkSessionTimeout(): void {
    const timeSinceLastActivity = Date.now() - this.lastActivity;
    
    if (timeSinceLastActivity >= this.config.timeout) {
      this.handleSessionTimeout();
    }
  }
  
  // Manejar timeout de sesión
  private async handleSessionTimeout(): Promise<void> {
    if (this.config.autoLogout) {
      await this.logout();
    } else {
      this.lockSession();
    }
  }
  
  // Bloquear sesión
  private lockSession(): void {
    // Implementar lógica de bloqueo
    // Por ejemplo, mostrar pantalla de login
    console.log('Session locked due to inactivity');
  }
  
  // Renovar sesión
  async renewSession(): Promise<boolean> {
    try {
      const token = await this.tokenManager.getAccessToken();
      if (token) {
        this.lastActivity = Date.now();
        return true;
      }
      return false;
    } catch (error) {
      console.error('Error renewing session:', error);
      return false;
    }
  }
  
  // Verificar si la sesión está activa
  isSessionActive(): boolean {
    const timeSinceLastActivity = Date.now() - this.lastActivity;
    return timeSinceLastActivity < this.config.timeout;
  }
  
  // Obtener tiempo restante de sesión
  getSessionTimeRemaining(): number {
    const timeSinceLastActivity = Date.now() - this.lastActivity;
    return Math.max(0, this.config.timeout - timeSinceLastActivity);
  }
  
  // Logout
  async logout(): Promise<void> {
    try {
      await this.tokenManager.clearTokens();
      this.lastActivity = 0;
      
      if (this.activityTimer) {
        clearInterval(this.activityTimer);
        this.activityTimer = null;
      }
      
      console.log('User logged out successfully');
    } catch (error) {
      console.error('Error during logout:', error);
    }
  }
  
  // Limpiar recursos
  cleanup(): void {
    if (this.activityTimer) {
      clearInterval(this.activityTimer);
      this.activityTimer = null;
    }
  }
}
```

## Ejercicios Prácticos

### Ejercicio 1: Implementar Sistema de Autenticación Completo

**Objetivo**: Crear un sistema completo de autenticación con JWT.

```typescript
// Implementa la clase AuthenticationService
class AuthenticationService {
  private apiUrl: string;
  private tokenManager: TokenManager;
  
  constructor(apiUrl: string, tokenManager: TokenManager) {
    this.apiUrl = apiUrl;
    this.tokenManager = tokenManager;
  }
  
  // Login de usuario
  async login(credentials: LoginCredentials): Promise<LoginResponse> {
    // TODO: Implementar login
    // Debe incluir:
    // - Validación de credenciales
    // - Petición al servidor
    // - Manejo de respuesta
    // - Almacenamiento de tokens
  }
  
  // Registro de usuario
  async register(userData: RegisterData): Promise<RegisterResponse> {
    // TODO: Implementar registro
    // Debe incluir:
    // - Validación de datos
    // - Petición al servidor
    // - Manejo de respuesta
  }
  
  // Logout
  async logout(): Promise<void> {
    // TODO: Implementar logout
    // Debe incluir:
    // - Petición al servidor para invalidar token
    // - Limpieza de tokens locales
    // - Limpieza de estado de la app
  }
  
  // Verificar token
  async verifyToken(): Promise<boolean> {
    // TODO: Implementar verificación
    // Debe incluir:
    // - Obtener token local
    // - Verificar con servidor
    // - Manejar token expirado
  }
  
  // Cambiar contraseña
  async changePassword(
    currentPassword: string,
    newPassword: string
  ): Promise<boolean> {
    // TODO: Implementar cambio de contraseña
    // Debe incluir:
    // - Verificación de contraseña actual
    // - Validación de nueva contraseña
    // - Petición al servidor
  }
}

// Prueba tu implementación
const testAuthService = async () => {
  const tokenManager = new TokenManager(AsyncStorage, 'https://api.example.com/refresh');
  const authService = new AuthenticationService('https://api.example.com', tokenManager);
  
  try {
    // Login
    const loginResult = await authService.login({
      email: 'test@example.com',
      password: 'password123',
    });
    console.log('Login exitoso:', loginResult);
    
    // Verificar token
    const isValid = await authService.verifyToken();
    console.log('Token válido:', isValid);
    
  } catch (error) {
    console.error('Error de autenticación:', error);
  }
};
```

### Ejercicio 2: Sistema de Roles y Permisos

**Objetivo**: Implementar un sistema completo de gestión de roles y permisos.

```typescript
// Implementa la clase RoleBasedAccessControl
class RoleBasedAccessControl {
  private roles: Map<string, Set<string>>;
  private userRoles: Map<string, string[]>;
  
  constructor() {
    this.roles = new Map();
    this.userRoles = new Map();
    this.initializeDefaultRoles();
  }
  
  // Inicializar roles por defecto
  private initializeDefaultRoles(): void {
    // TODO: Implementar roles por defecto
    // Debe incluir:
    // - Role GUEST con permisos básicos
    // - Role USER con permisos estándar
    // - Role MODERATOR con permisos de moderación
    // - Role ADMIN con permisos administrativos
  }
  
  // Agregar rol
  addRole(roleName: string, permissions: string[]): void {
    // TODO: Implementar agregar rol
    // Debe incluir:
    // - Validación de nombre de rol
    // - Validación de permisos
    // - Almacenamiento del rol
  }
  
  // Asignar rol a usuario
  assignRoleToUser(userId: string, roleName: string): boolean {
    // TODO: Implementar asignación de rol
    // Debe incluir:
    // - Verificación de existencia del rol
    // - Asignación al usuario
    // - Manejo de roles múltiples
  }
  
  // Verificar permiso de usuario
  userHasPermission(userId: string, permission: string): boolean {
    // TODO: Implementar verificación de permiso
    // Debe incluir:
    // - Obtener roles del usuario
    // - Verificar permisos de cada rol
    // - Retornar resultado
  }
  
  // Obtener permisos del usuario
  getUserPermissions(userId: string): string[] {
    // TODO: Implementar obtención de permisos
    // Debe incluir:
    // - Obtener roles del usuario
    // - Consolidar todos los permisos
    // - Eliminar duplicados
  }
  
  // Remover rol de usuario
  removeRoleFromUser(userId: string, roleName: string): boolean {
    // TODO: Implementar remoción de rol
    // Debe incluir:
    // - Verificación de existencia del rol
    // - Remoción del rol
    // - Actualización de permisos
  }
}

// Prueba tu implementación
const testRBAC = () => {
  const rbac = new RoleBasedAccessControl();
  
  // Agregar roles personalizados
  rbac.addRole('EDITOR', ['READ_CONTENT', 'CREATE_CONTENT', 'UPDATE_OWN_CONTENT']);
  rbac.addRole('REVIEWER', ['READ_CONTENT', 'MODERATE_CONTENT', 'APPROVE_CONTENT']);
  
  // Asignar roles a usuarios
  rbac.assignRoleToUser('user1', 'EDITOR');
  rbac.assignRoleToUser('user2', 'REVIEWER');
  rbac.assignRoleToUser('user2', 'EDITOR'); // Usuario con múltiples roles
  
  // Verificar permisos
  console.log('User1 puede crear contenido:', rbac.userHasPermission('user1', 'CREATE_CONTENT'));
  console.log('User2 puede moderar:', rbac.userHasPermission('user2', 'MODERATE_CONTENT'));
  console.log('Permisos de User2:', rbac.getUserPermissions('user2'));
};
```

### Ejercicio 3: Middleware de Seguridad

**Objetivo**: Crear middleware para proteger rutas y componentes.

```typescript
// Implementa el middleware de seguridad
class SecurityMiddleware {
  private authService: AuthenticationService;
  private permissionManager: PermissionManager;
  
  constructor(authService: AuthenticationService, permissionManager: PermissionManager) {
    this.authService = authService;
    this.permissionManager = permissionManager;
  }
  
  // Middleware de autenticación
  requireAuth = (Component: React.ComponentType<any>) => {
    return (props: any) => {
      // TODO: Implementar middleware de autenticación
      // Debe incluir:
      // - Verificación de estado de autenticación
      // - Redirección si no está autenticado
      // - Renderizado del componente si está autenticado
    };
  };
  
  // Middleware de permisos
  requirePermission = (permission: string) => {
    return (Component: React.ComponentType<any>) => {
      return (props: any) => {
        // TODO: Implementar middleware de permisos
        // Debe incluir:
        // - Verificación de autenticación
        // - Verificación de permisos
        // - Renderizado condicional
      };
    };
  };
  
  // Middleware de roles
  requireRole = (role: string) => {
    return (Component: React.ComponentType<any>) => {
      return (props: any) => {
        // TODO: Implementar middleware de roles
        // Debe incluir:
        // - Verificación de autenticación
        // - Verificación de rol
        // - Renderizado condicional
      };
    };
  };
  
  // Middleware de validación de entrada
  validateInput = (schema: any) => {
    return (Component: React.ComponentType<any>) => {
      return (props: any) => {
        // TODO: Implementar middleware de validación
        // Debe incluir:
        // - Validación de props según schema
        // - Manejo de errores de validación
        // - Renderizado del componente si es válido
      };
    };
  };
}

// Prueba tu implementación
const testSecurityMiddleware = () => {
  const authService = new AuthenticationService('https://api.example.com', tokenManager);
  const permissionManager = new PermissionManager(UserRole.USER);
  const middleware = new SecurityMiddleware(authService, permissionManager);
  
  // Componente protegido por autenticación
  const ProtectedComponent = middleware.requireAuth(MyComponent);
  
  // Componente protegido por permisos
  const AdminComponent = middleware.requirePermission('MANAGE_USERS')(AdminPanel);
  
  // Componente protegido por rol
  const ModeratorComponent = middleware.requireRole('MODERATOR')(ModeratorPanel);
  
  console.log('Middleware configurado correctamente');
};
```

## Resumen de la Clase

En esta clase hemos cubierto:

✅ **Sistemas de autenticación** con JWT y OAuth
✅ **Gestión de roles y permisos** para control de acceso
✅ **Middleware de seguridad** para proteger componentes
✅ **Gestión de sesiones** con timeout y renovación automática
✅ **Ejercicios prácticos** para implementar sistemas completos

## Próximos Pasos

En la siguiente clase aprenderemos sobre:
- **Encriptación y Protección de Datos**
- **Almacenamiento seguro**
- **Protección de APIs**
- **Manejo de datos sensibles**

## Recursos Adicionales

- [JWT.io - JSON Web Tokens](https://jwt.io/)
- [OAuth 2.0 Specification](https://oauth.net/2/)
- [React Native Security Best Practices](https://reactnative.dev/docs/security)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

---

**Nota**: La autenticación y autorización son la base de cualquier sistema seguro. Asegúrate de implementar múltiples capas de seguridad y seguir las mejores prácticas de la industria.
