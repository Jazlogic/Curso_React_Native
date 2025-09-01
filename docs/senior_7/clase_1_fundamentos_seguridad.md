# Clase 1: Fundamentos de Seguridad 🔐

## Información de la Clase

- **Módulo**: 14 - Seguridad
- **Clase**: 1 de 5
- **Duración**: 1.5 horas
- **Tipo**: Teórica + Práctica

## Navegación

- **Anterior**: [Módulo 13: Monitoreo](../senior_4/README.md)
- **Siguiente**: [Clase 2: Autenticación y Autorización](clase_2_autenticacion_autorizacion.md)
- **Arriba**: [README del Módulo](README.md)

## Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender** los conceptos fundamentales de seguridad en aplicaciones móviles
2. **Identificar** las amenazas más comunes en React Native
3. **Aplicar** los principios básicos de seguridad
4. **Implementar** medidas de seguridad básicas en una app
5. **Evaluar** la seguridad de una aplicación existente

## Contenido Teórico

### 1. ¿Qué es la Seguridad en Aplicaciones Móviles?

La seguridad en aplicaciones móviles se refiere a la protección de:
- **Datos del usuario**: Información personal, credenciales, datos financieros
- **Integridad de la aplicación**: Prevención de manipulación o corrupción
- **Privacidad**: Control sobre qué información se comparte y cómo
- **Recursos del dispositivo**: Acceso a cámara, ubicación, contactos

```typescript
// Ejemplo de datos sensibles que requieren protección
interface UserData {
  id: string;           // Identificador único del usuario
  email: string;        // Email (dato personal)
  password: string;     // Contraseña (dato sensible)
  creditCard?: string;  // Tarjeta de crédito (dato financiero)
  location?: {          // Ubicación (dato de privacidad)
    latitude: number;
    longitude: number;
  };
}
```

### 2. Amenazas Comunes en React Native

#### A. **Inyección de Código**
```typescript
// ❌ PELIGROSO: Código vulnerable a inyección
const executeCode = (userInput: string) => {
  // Esto es peligroso - permite ejecutar código arbitrario
  eval(userInput);
};

// ✅ SEGURO: Validación y sanitización
const executeCode = (userInput: string) => {
  // Solo permitir operaciones matemáticas básicas
  const sanitizedInput = userInput.replace(/[^0-9+\-*/().]/g, '');
  try {
    return eval(sanitizedInput);
  } catch (error) {
    console.error('Invalid input:', error);
    return null;
  }
};
```

#### B. **Almacenamiento Inseguro**
```typescript
// ❌ PELIGROSO: Almacenamiento en texto plano
import AsyncStorage from '@react-native-async-storage/async-storage';

const savePassword = async (password: string) => {
  // La contraseña se guarda sin encriptar
  await AsyncStorage.setItem('userPassword', password);
};

// ✅ SEGURO: Almacenamiento encriptado
import { encrypt, decrypt } from 'react-native-crypto-js';

const savePassword = async (password: string) => {
  // Encriptar antes de guardar
  const encryptedPassword = encrypt(password, 'secretKey');
  await AsyncStorage.setItem('userPassword', encryptedPassword);
};

const getPassword = async () => {
  const encryptedPassword = await AsyncStorage.getItem('userPassword');
  if (encryptedPassword) {
    return decrypt(encryptedPassword, 'secretKey').toString();
  }
  return null;
};
```

#### C. **Comunicación No Segura**
```typescript
// ❌ PELIGROSO: HTTP sin encriptación
const fetchData = async () => {
  const response = await fetch('http://api.example.com/data');
  return response.json();
};

// ✅ SEGURO: HTTPS con certificados SSL
const fetchData = async () => {
  const response = await fetch('https://api.example.com/data', {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
    },
  });
  return response.json();
};
```

### 3. Principios de Seguridad

#### A. **Principio de Mínimo Privilegio**
```typescript
// Ejemplo de implementación del principio de mínimo privilegio
interface UserPermissions {
  canRead: boolean;
  canWrite: boolean;
  canDelete: boolean;
  canAdmin: boolean;
}

class SecureUser {
  private permissions: UserPermissions;
  
  constructor(role: 'user' | 'admin' | 'moderator') {
    switch (role) {
      case 'user':
        this.permissions = {
          canRead: true,
          canWrite: false,
          canDelete: false,
          canAdmin: false,
        };
        break;
      case 'moderator':
        this.permissions = {
          canRead: true,
          canWrite: true,
          canDelete: false,
          canAdmin: false,
        };
        break;
      case 'admin':
        this.permissions = {
          canRead: true,
          canWrite: true,
          canDelete: true,
          canAdmin: true,
        };
        break;
    }
  }
  
  canPerformAction(action: keyof UserPermissions): boolean {
    return this.permissions[action];
  }
}
```

#### B. **Defensa en Profundidad**
```typescript
// Implementación de múltiples capas de seguridad
class SecureDataManager {
  // Capa 1: Validación de entrada
  private validateInput(data: any): boolean {
    if (!data || typeof data !== 'object') return false;
    if (!data.id || !data.content) return false;
    return true;
  }
  
  // Capa 2: Sanitización
  private sanitizeData(data: any): any {
    return {
      id: data.id.toString().replace(/[^a-zA-Z0-9]/g, ''),
      content: data.content.toString().trim(),
      timestamp: new Date().toISOString(),
    };
  }
  
  // Capa 3: Encriptación
  private encryptData(data: any): string {
    return encrypt(JSON.stringify(data), 'secretKey');
  }
  
  // Capa 4: Verificación de integridad
  private verifyIntegrity(encryptedData: string): boolean {
    try {
      const decrypted = decrypt(encryptedData, 'secretKey');
      const data = JSON.parse(decrypted);
      return this.validateInput(data);
    } catch {
      return false;
    }
  }
  
  // Método principal que aplica todas las capas
  async saveData(data: any): Promise<boolean> {
    try {
      // Aplicar todas las capas de seguridad
      if (!this.validateInput(data)) return false;
      
      const sanitized = this.sanitizeData(data);
      const encrypted = this.encryptData(sanitized);
      
      // Guardar encriptado
      await AsyncStorage.setItem(`data_${sanitized.id}`, encrypted);
      
      return true;
    } catch (error) {
      console.error('Error saving data:', error);
      return false;
    }
  }
}
```

### 4. Herramientas de Seguridad

#### A. **Librerías de Encriptación**
```bash
# Instalación de librerías de seguridad
npm install react-native-crypto-js
npm install react-native-keychain
npm install react-native-biometrics
npm install @react-native-async-storage/async-storage
```

#### B. **Configuración de Seguridad**
```typescript
// Configuración de seguridad para la aplicación
export const SecurityConfig = {
  // Configuración de encriptación
  encryption: {
    algorithm: 'AES-256-GCM',
    keySize: 256,
    saltRounds: 12,
  },
  
  // Configuración de sesión
  session: {
    timeout: 30 * 60 * 1000, // 30 minutos
    maxRetries: 3,
    lockOnBackground: true,
  },
  
  // Configuración de red
  network: {
    requireHTTPS: true,
    certificatePinning: true,
    timeout: 10000, // 10 segundos
  },
  
  // Configuración de almacenamiento
  storage: {
    encryptSensitiveData: true,
    autoWipeOnTamper: true,
    backupProtection: true,
  },
};
```

## Ejercicios Prácticos

### Ejercicio 1: Implementar Validación de Entrada

**Objetivo**: Crear un sistema de validación robusto para datos de usuario.

```typescript
// Implementa la clase SecureInputValidator
class SecureInputValidator {
  // Valida email
  static validateEmail(email: string): boolean {
    // TODO: Implementar validación de email
    // Debe verificar formato, longitud y caracteres permitidos
  }
  
  // Valida contraseña
  static validatePassword(password: string): boolean {
    // TODO: Implementar validación de contraseña
    // Debe verificar longitud mínima, caracteres especiales, números, etc.
  }
  
  // Valida número de tarjeta de crédito
  static validateCreditCard(cardNumber: string): boolean {
    // TODO: Implementar algoritmo de Luhn
    // Debe verificar que el número sea válido matemáticamente
  }
  
  // Sanitiza texto
  static sanitizeText(text: string): string {
    // TODO: Implementar sanitización
    // Debe remover caracteres peligrosos y limitar longitud
  }
}

// Prueba tu implementación
const testValidator = () => {
  console.log('Email válido:', SecureInputValidator.validateEmail('test@example.com'));
  console.log('Contraseña válida:', SecureInputValidator.validatePassword('SecurePass123!'));
  console.log('Tarjeta válida:', SecureInputValidator.validateCreditCard('4532015112830366'));
  console.log('Texto sanitizado:', SecureInputValidator.sanitizeText('<script>alert("xss")</script>'));
};
```

### Ejercicio 2: Sistema de Logs de Seguridad

**Objetivo**: Implementar un sistema de logging para eventos de seguridad.

```typescript
// Implementa la clase SecurityLogger
class SecurityLogger {
  private logs: SecurityEvent[] = [];
  
  // Tipos de eventos de seguridad
  static readonly EVENT_TYPES = {
    LOGIN_ATTEMPT: 'LOGIN_ATTEMPT',
    LOGIN_SUCCESS: 'LOGIN_SUCCESS',
    LOGIN_FAILURE: 'LOGIN_FAILURE',
    DATA_ACCESS: 'DATA_ACCESS',
    PERMISSION_DENIED: 'PERMISSION_DENIED',
    SUSPICIOUS_ACTIVITY: 'SUSPICIOUS_ACTIVITY',
  } as const;
  
  // Niveles de severidad
  static readonly SEVERITY_LEVELS = {
    LOW: 'LOW',
    MEDIUM: 'MEDIUM',
    HIGH: 'HIGH',
    CRITICAL: 'CRITICAL',
  } as const;
  
  // Log de evento de seguridad
  logEvent(
    type: keyof typeof SecurityLogger.EVENT_TYPES,
    severity: keyof typeof SecurityLogger.SEVERITY_LEVELS,
    details: string,
    userId?: string,
    metadata?: Record<string, any>
  ): void {
    // TODO: Implementar logging de eventos
    // Debe incluir timestamp, tipo, severidad, detalles, usuario y metadatos
  }
  
  // Obtener logs por tipo
  getLogsByType(type: keyof typeof SecurityLogger.EVENT_TYPES): SecurityEvent[] {
    // TODO: Implementar filtrado por tipo
  }
  
  // Obtener logs por severidad
  getLogsBySeverity(severity: keyof typeof SecurityLogger.SEVERITY_LEVELS): SecurityEvent[] {
    // TODO: Implementar filtrado por severidad
  }
  
  // Limpiar logs antiguos
  cleanOldLogs(maxAge: number): void {
    // TODO: Implementar limpieza de logs antiguos
    // maxAge en milisegundos
  }
  
  // Exportar logs para auditoría
  exportLogs(): string {
    // TODO: Implementar exportación de logs
    // Debe retornar JSON formateado
  }
}

// Prueba tu implementación
const testSecurityLogger = () => {
  const logger = new SecurityLogger();
  
  logger.logEvent(
    SecurityLogger.EVENT_TYPES.LOGIN_ATTEMPT,
    SecurityLogger.SEVERITY_LEVELS.MEDIUM,
    'Usuario intentó acceder con credenciales incorrectas',
    'user123',
    { ip: '192.168.1.1', userAgent: 'Mozilla/5.0...' }
  );
  
  console.log('Logs de login:', logger.getLogsByType(SecurityLogger.EVENT_TYPES.LOGIN_ATTEMPT));
  console.log('Logs de alta severidad:', logger.getLogsBySeverity(SecurityLogger.SEVERITY_LEVELS.HIGH));
};
```

### Ejercicio 3: Configuración de Seguridad de Red

**Objetivo**: Configurar medidas de seguridad para comunicaciones de red.

```typescript
// Implementa la clase SecureNetworkManager
class SecureNetworkManager {
  private config: NetworkSecurityConfig;
  
  constructor(config: NetworkSecurityConfig) {
    this.config = config;
  }
  
  // Realizar petición segura
  async secureRequest(
    url: string,
    options: RequestOptions
  ): Promise<SecureResponse> {
    // TODO: Implementar petición segura
    // Debe incluir:
    // - Verificación de HTTPS
    // - Validación de certificados
    // - Timeout configurable
    // - Headers de seguridad
    // - Manejo de errores
  }
  
  // Verificar certificado SSL
  private verifySSLCertificate(url: string): Promise<boolean> {
    // TODO: Implementar verificación de certificado
    // Debe verificar que el certificado sea válido y confiable
  }
  
  // Configurar headers de seguridad
  private getSecurityHeaders(): Record<string, string> {
    // TODO: Implementar headers de seguridad
    // Debe incluir:
    // - Content-Security-Policy
    // - X-Frame-Options
    // - X-Content-Type-Options
    // - Referrer-Policy
  }
  
  // Validar URL
  private validateURL(url: string): boolean {
    // TODO: Implementar validación de URL
    // Debe verificar:
    // - Protocolo HTTPS
    // - Formato válido
    // - Dominio permitido
  }
}

// Prueba tu implementación
const testSecureNetwork = async () => {
  const config: NetworkSecurityConfig = {
    requireHTTPS: true,
    timeout: 10000,
    maxRetries: 3,
    allowedDomains: ['api.example.com', 'secure.example.com'],
  };
  
  const networkManager = new SecureNetworkManager(config);
  
  try {
    const response = await networkManager.secureRequest(
      'https://api.example.com/data',
      { method: 'GET' }
    );
    console.log('Respuesta segura:', response);
  } catch (error) {
    console.error('Error de red:', error);
  }
};
```

## Resumen de la Clase

En esta clase hemos cubierto:

✅ **Conceptos fundamentales** de seguridad en aplicaciones móviles
✅ **Amenazas comunes** y cómo prevenirlas
✅ **Principios de seguridad** (mínimo privilegio, defensa en profundidad)
✅ **Herramientas básicas** para implementar seguridad
✅ **Ejercicios prácticos** para aplicar los conceptos

## Próximos Pasos

En la siguiente clase aprenderemos sobre:
- **Autenticación y Autorización**
- **JWT y OAuth**
- **Gestión de roles y permisos**
- **Sistemas de sesión seguros**

## Recursos Adicionales

- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [React Native Security Guidelines](https://reactnative.dev/docs/security)
- [Mobile App Security Testing](https://owasp.org/www-project-mobile-security-testing-guide/)

---

**Nota**: La seguridad es un proceso continuo, no un estado final. Siempre mantente actualizado con las últimas amenazas y mejores prácticas.
