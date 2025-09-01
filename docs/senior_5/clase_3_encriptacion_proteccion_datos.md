# Clase 3: Encriptación y Protección de Datos 🔐

## Información de la Clase

- **Módulo**: 14 - Seguridad
- **Clase**: 3 de 5
- **Duración**: 2 horas
- **Tipo**: Teórica + Práctica

## Navegación

- **Anterior**: [Clase 2: Autenticación y Autorización](clase_2_autenticacion_autorizacion.md)
- **Siguiente**: [Clase 4: Seguridad de Red y API](clase_4_seguridad_red_api.md)
- **Arriba**: [README del Módulo](README.md)

## Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Implementar** sistemas de encriptación robustos
2. **Proteger** datos sensibles en almacenamiento local
3. **Implementar** protección de APIs con encriptación
4. **Aplicar** mejores prácticas de protección de datos
5. **Gestionar** claves de encriptación de forma segura

## Contenido Teórico

### 1. Fundamentos de Encriptación

#### A. **Tipos de Encriptación**

```typescript
// Enumeración de algoritmos de encriptación
enum EncryptionAlgorithm {
  // Encriptación simétrica
  AES_128 = 'AES-128',
  AES_256 = 'AES-256',
  
  // Encriptación asimétrica
  RSA_2048 = 'RSA-2048',
  RSA_4096 = 'RSA-4096',
  
  // Hashing
  SHA_256 = 'SHA-256',
  SHA_512 = 'SHA-512',
  
  // Hashing con salt
  BCRYPT = 'BCRYPT',
  ARGON2 = 'ARGON2',
}

// Configuración de encriptación
interface EncryptionConfig {
  algorithm: EncryptionAlgorithm;
  keySize: number;
  saltRounds: number;
  ivLength: number;
}
```

#### B. **Clase de Encriptación Base**

```typescript
// Clase para manejo de encriptación
class EncryptionManager {
  private config: EncryptionConfig;
  private secretKey: string;
  
  constructor(config: EncryptionConfig, secretKey: string) {
    this.config = config;
    this.secretKey = secretKey;
  }
  
  // Encriptar datos
  async encrypt(data: string): Promise<EncryptedData> {
    try {
      // Generar IV (Initialization Vector)
      const iv = this.generateIV();
      
      // Encriptar datos usando AES
      const encrypted = await this.encryptAES(data, iv);
      
      return {
        data: encrypted,
        iv: iv,
        algorithm: this.config.algorithm,
        timestamp: Date.now(),
      };
    } catch (error) {
      throw new Error(`Encryption failed: ${error.message}`);
    }
  }
  
  // Desencriptar datos
  async decrypt(encryptedData: EncryptedData): Promise<string> {
    try {
      const decrypted = await this.decryptAES(encryptedData.data, encryptedData.iv);
      return decrypted;
    } catch (error) {
      throw new Error(`Decryption failed: ${error.message}`);
    }
  }
  
  // Generar IV aleatorio
  private generateIV(): string {
    const array = new Uint8Array(this.config.ivLength);
    crypto.getRandomValues(array);
    return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
  }
  
  // Encriptar con AES (simulado)
  private async encryptAES(data: string, iv: string): Promise<string> {
    // En producción, usar librería como react-native-crypto-js
    return btoa(`${data}:${iv}:${this.secretKey}`);
  }
  
  // Desencriptar con AES (simulado)
  private async decryptAES(encryptedData: string, iv: string): Promise<string> {
    // En producción, usar librería como react-native-crypto-js
    const parts = atob(encryptedData).split(':');
    if (parts.length !== 3) {
      throw new Error('Invalid encrypted data format');
    }
    return parts[0];
  }
}

// Estructura de datos encriptados
interface EncryptedData {
  data: string;
  iv: string;
  algorithm: string;
  timestamp: number;
}
```

### 2. Almacenamiento Seguro de Datos

#### A. **Keychain para Datos Sensibles**

```typescript
// Clase para almacenamiento seguro usando Keychain
class SecureStorage {
  private encryptionManager: EncryptionManager;
  
  constructor(encryptionManager: EncryptionManager) {
    this.encryptionManager = encryptionManager;
  }
  
  // Guardar datos sensibles
  async saveSecureData(key: string, data: string): Promise<void> {
    try {
      // Encriptar datos antes de guardar
      const encrypted = await this.encryptionManager.encrypt(data);
      
      // Guardar en Keychain (iOS) o Keystore (Android)
      await this.saveToKeychain(key, JSON.stringify(encrypted));
    } catch (error) {
      throw new Error(`Failed to save secure data: ${error.message}`);
    }
  }
  
  // Obtener datos sensibles
  async getSecureData(key: string): Promise<string | null> {
    try {
      // Obtener datos encriptados del Keychain
      const encryptedData = await this.getFromKeychain(key);
      if (!encryptedData) return null;
      
      // Desencriptar datos
      const encrypted = JSON.parse(encryptedData);
      return await this.encryptionManager.decrypt(encrypted);
    } catch (error) {
      console.error('Failed to get secure data:', error);
      return null;
    }
  }
  
  // Eliminar datos sensibles
  async removeSecureData(key: string): Promise<void> {
    try {
      await this.removeFromKeychain(key);
    } catch (error) {
      throw new Error(`Failed to remove secure data: ${error.message}`);
    }
  }
  
  // Guardar en Keychain (simulado)
  private async saveToKeychain(key: string, value: string): Promise<void> {
    // En producción, usar react-native-keychain
    await AsyncStorage.setItem(`keychain_${key}`, value);
  }
  
  // Obtener del Keychain (simulado)
  private async getFromKeychain(key: string): Promise<string | null> {
    // En producción, usar react-native-keychain
    return await AsyncStorage.getItem(`keychain_${key}`);
  }
  
  // Remover del Keychain (simulado)
  private async removeFromKeychain(key: string): Promise<void> {
    // En producción, usar react-native-keychain
    await AsyncStorage.removeItem(`keychain_${key}`);
  }
}
```

#### B. **Gestión de Claves de Encriptación**

```typescript
// Clase para gestión segura de claves
class KeyManager {
  private masterKey: string;
  private keyDerivationSalt: string;
  
  constructor() {
    this.masterKey = this.generateMasterKey();
    this.keyDerivationSalt = this.generateSalt();
  }
  
  // Generar clave maestra
  private generateMasterKey(): string {
    const array = new Uint8Array(32);
    crypto.getRandomValues(array);
    return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
  }
  
  // Generar salt para derivación de claves
  private generateSalt(): string {
    const array = new Uint8Array(16);
    crypto.getRandomValues(array);
    return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
  }
  
  // Derivar clave específica
  deriveKey(purpose: string, length: number = 32): string {
    // En producción, usar PBKDF2 o Argon2
    const input = `${this.masterKey}:${purpose}:${this.keyDerivationSalt}`;
    return this.hashString(input).substring(0, length);
  }
  
  // Hash de string (simulado)
  private hashString(input: string): string {
    // En producción, usar SHA-256
    let hash = 0;
    for (let i = 0; i < input.length; i++) {
      const char = input.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convertir a entero de 32 bits
    }
    return Math.abs(hash).toString(16);
  }
  
  // Rotar claves
  async rotateKeys(): Promise<void> {
    try {
      // Generar nueva clave maestra
      this.masterKey = this.generateMasterKey();
      this.keyDerivationSalt = this.generateSalt();
      
      // Re-encriptar datos existentes con nueva clave
      await this.reEncryptExistingData();
    } catch (error) {
      throw new Error(`Key rotation failed: ${error.message}`);
    }
  }
  
  // Re-encriptar datos existentes
  private async reEncryptExistingData(): Promise<void> {
    // Implementar lógica para re-encriptar datos
    console.log('Re-encrypting existing data with new keys...');
  }
}
```

### 3. Protección de APIs

#### A. **Encriptación de Payloads**

```typescript
// Clase para protección de APIs
class APISecurityManager {
  private encryptionManager: EncryptionManager;
  private keyManager: KeyManager;
  
  constructor(encryptionManager: EncryptionManager, keyManager: KeyManager) {
    this.encryptionManager = encryptionManager;
    this.keyManager = keyManager;
  }
  
  // Encriptar payload antes de enviar
  async encryptPayload(payload: any): Promise<EncryptedPayload> {
    try {
      const jsonPayload = JSON.stringify(payload);
      const encrypted = await this.encryptionManager.encrypt(jsonPayload);
      
      return {
        encryptedData: encrypted.data,
        iv: encrypted.iv,
        timestamp: Date.now(),
        signature: this.generateSignature(encrypted.data),
      };
    } catch (error) {
      throw new Error(`Payload encryption failed: ${error.message}`);
    }
  }
  
  // Desencriptar respuesta del servidor
  async decryptResponse(encryptedResponse: EncryptedPayload): Promise<any> {
    try {
      // Verificar firma
      if (!this.verifySignature(encryptedResponse.encryptedData, encryptedResponse.signature)) {
        throw new Error('Invalid response signature');
      }
      
      // Desencriptar datos
      const encryptedData: EncryptedData = {
        data: encryptedResponse.encryptedData,
        iv: encryptedResponse.iv,
        algorithm: 'AES-256',
        timestamp: encryptedResponse.timestamp,
      };
      
      const decrypted = await this.encryptionManager.decrypt(encryptedData);
      return JSON.parse(decrypted);
    } catch (error) {
      throw new Error(`Response decryption failed: ${error.message}`);
    }
  }
  
  // Generar firma digital
  private generateSignature(data: string): string {
    const key = this.keyManager.deriveKey('api-signature', 16);
    return this.hmac(data, key);
  }
  
  // Verificar firma digital
  private verifySignature(data: string, signature: string): boolean {
    const expectedSignature = this.generateSignature(data);
    return signature === expectedSignature;
  }
  
  // HMAC (simulado)
  private hmac(data: string, key: string): string {
    // En producción, usar HMAC-SHA256
    return btoa(`${data}:${key}`).substring(0, 32);
  }
}

// Estructura de payload encriptado
interface EncryptedPayload {
  encryptedData: string;
  iv: string;
  timestamp: number;
  signature: string;
}
```

#### B. **Middleware de Seguridad para APIs**

```typescript
// Middleware para proteger llamadas a API
const withAPISecurity = (apiCall: Function) => {
  return async (...args: any[]) => {
    try {
      // Encriptar argumentos sensibles
      const encryptedArgs = await encryptSensitiveArgs(args);
      
      // Realizar llamada a API
      const response = await apiCall(...encryptedArgs);
      
      // Desencriptar respuesta si es necesario
      if (response.encrypted) {
        return await decryptResponse(response);
      }
      
      return response;
    } catch (error) {
      console.error('API call failed:', error);
      throw error;
    }
  };
};

// Función para encriptar argumentos sensibles
const encryptSensitiveArgs = async (args: any[]): Promise<any[]> => {
  const encryptionManager = new EncryptionManager(
    { algorithm: EncryptionAlgorithm.AES_256, keySize: 256, saltRounds: 12, ivLength: 16 },
    'secret-key'
  );
  
  return await Promise.all(
    args.map(async (arg) => {
      if (typeof arg === 'string' && isSensitive(arg)) {
        return await encryptionManager.encrypt(arg);
      }
      return arg;
    })
  );
};

// Función para determinar si un argumento es sensible
const isSensitive = (value: string): boolean => {
  const sensitivePatterns = [
    /password/i,
    /token/i,
    /secret/i,
    /key/i,
    /credit.?card/i,
    /ssn/i,
  ];
  
  return sensitivePatterns.some(pattern => pattern.test(value));
};
```

## Ejercicios Prácticos

### Ejercicio 1: Sistema de Encriptación Completo

**Objetivo**: Implementar un sistema completo de encriptación para datos sensibles.

```typescript
// Implementa la clase SecureDataManager
class SecureDataManager {
  private encryptionManager: EncryptionManager;
  private secureStorage: SecureStorage;
  
  constructor() {
    const config: EncryptionConfig = {
      algorithm: EncryptionAlgorithm.AES_256,
      keySize: 256,
      saltRounds: 12,
      ivLength: 16,
    };
    
    const encryptionManager = new EncryptionManager(config, 'your-secret-key');
    const secureStorage = new SecureStorage(encryptionManager);
    
    this.encryptionManager = encryptionManager;
    this.secureStorage = secureStorage;
  }
  
  // Guardar datos sensibles
  async saveSensitiveData(key: string, data: any): Promise<void> {
    // TODO: Implementar guardado seguro
    // Debe incluir:
    // - Validación de datos
    // - Encriptación
    // - Almacenamiento seguro
  }
  
  // Obtener datos sensibles
  async getSensitiveData(key: string): Promise<any | null> {
    // TODO: Implementar obtención segura
    // Debe incluir:
    // - Obtención del almacenamiento
    // - Desencriptación
    // - Validación de integridad
  }
  
  // Eliminar datos sensibles
  async removeSensitiveData(key: string): Promise<void> {
    // TODO: Implementar eliminación segura
    // Debe incluir:
    // - Eliminación del almacenamiento
    // - Limpieza de memoria
  }
  
  // Listar claves de datos sensibles
  async listSensitiveDataKeys(): Promise<string[]> {
    // TODO: Implementar listado de claves
    // Debe incluir:
    // - Obtención de todas las claves
    // - Filtrado de claves sensibles
  }
}

// Prueba tu implementación
const testSecureDataManager = async () => {
  const secureManager = new SecureDataManager();
  
  try {
    // Guardar datos sensibles
    await secureManager.saveSensitiveData('user_credentials', {
      username: 'john_doe',
      password: 'secure_password_123',
      email: 'john@example.com',
    });
    
    // Obtener datos sensibles
    const credentials = await secureManager.getSensitiveData('user_credentials');
    console.log('Credentials retrieved:', credentials);
    
    // Listar claves
    const keys = await secureManager.listSensitiveDataKeys();
    console.log('Sensitive data keys:', keys);
    
  } catch (error) {
    console.error('Error testing secure data manager:', error);
  }
};
```

### Ejercicio 2: Protección de API con Encriptación

**Objetivo**: Implementar protección completa de APIs con encriptación de payloads.

```typescript
// Implementa la clase SecureAPIClient
class SecureAPIClient {
  private baseURL: string;
  private apiSecurityManager: APISecurityManager;
  
  constructor(baseURL: string, apiSecurityManager: APISecurityManager) {
    this.baseURL = baseURL;
    this.apiSecurityManager = apiSecurityManager;
  }
  
  // Petición GET segura
  async secureGet(endpoint: string, headers?: Record<string, string>): Promise<any> {
    // TODO: Implementar GET seguro
    // Debe incluir:
    // - Headers de seguridad
    // - Verificación de certificados
    // - Desencriptación de respuesta
  }
  
  // Petición POST segura
  async securePost(endpoint: string, data: any, headers?: Record<string, string>): Promise<any> {
    // TODO: Implementar POST seguro
    // Debe incluir:
    // - Encriptación de payload
    // - Headers de seguridad
    // - Verificación de respuesta
  }
  
  // Petición PUT segura
  async securePut(endpoint: string, data: any, headers?: Record<string, string>): Promise<any> {
    // TODO: Implementar PUT seguro
    // Debe incluir:
    // - Encriptación de payload
    // - Headers de seguridad
    // - Verificación de respuesta
  }
  
  // Petición DELETE segura
  async secureDelete(endpoint: string, headers?: Record<string, string>): Promise<any> {
    // TODO: Implementar DELETE seguro
    // Debe incluir:
    // - Headers de seguridad
    // - Verificación de respuesta
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
}

// Prueba tu implementación
const testSecureAPIClient = async () => {
  const encryptionManager = new EncryptionManager(
    { algorithm: EncryptionAlgorithm.AES_256, keySize: 256, saltRounds: 12, ivLength: 16 },
    'api-secret-key'
  );
  
  const keyManager = new KeyManager();
  const apiSecurityManager = new APISecurityManager(encryptionManager, keyManager);
  
  const secureClient = new SecureAPIClient('https://api.example.com', apiSecurityManager);
  
  try {
    // POST seguro
    const response = await secureClient.securePost('/users', {
      name: 'John Doe',
      email: 'john@example.com',
      password: 'secure_password',
    });
    
    console.log('Secure POST response:', response);
    
    // GET seguro
    const users = await secureClient.secureGet('/users');
    console.log('Secure GET response:', users);
    
  } catch (error) {
    console.error('Error testing secure API client:', error);
  }
};
```

### Ejercicio 3: Gestión de Claves de Encriptación

**Objetivo**: Implementar un sistema robusto de gestión de claves.

```typescript
// Implementa la clase AdvancedKeyManager
class AdvancedKeyManager {
  private keys: Map<string, KeyInfo>;
  private keyRotationSchedule: Map<string, number>;
  
  constructor() {
    this.keys = new Map();
    this.keyRotationSchedule = new Map();
    this.initializeDefaultKeys();
  }
  
  // Inicializar claves por defecto
  private initializeDefaultKeys(): void {
    // TODO: Implementar inicialización de claves
    // Debe incluir:
    // - Clave maestra
    // - Claves para diferentes propósitos
    // - Programación de rotación
  }
  
  // Generar nueva clave
  generateKey(purpose: string, algorithm: EncryptionAlgorithm, keySize: number): string {
    // TODO: Implementar generación de clave
    // Debe incluir:
    // - Generación criptográficamente segura
    // - Almacenamiento de metadatos
    // - Programación de rotación
  }
  
  // Obtener clave por propósito
  getKey(purpose: string): string | null {
    // TODO: Implementar obtención de clave
    // Debe incluir:
    // - Verificación de validez
    // - Rotación automática si es necesario
    // - Logging de acceso
  }
  
  // Rotar clave específica
  async rotateKey(purpose: string): Promise<void> {
    // TODO: Implementar rotación de clave
    // Debe incluir:
    // - Generación de nueva clave
    // - Re-encriptación de datos
    // - Actualización de metadatos
  }
  
  // Verificar estado de claves
  checkKeyHealth(): KeyHealthReport {
    // TODO: Implementar verificación de salud
    // Debe incluir:
    // - Verificación de expiración
    // - Verificación de uso
    // - Recomendaciones de rotación
  }
}

// Prueba tu implementación
const testAdvancedKeyManager = async () => {
  const keyManager = new AdvancedKeyManager();
  
  try {
    // Generar clave para encriptación de datos
    const dataKey = keyManager.generateKey('data_encryption', EncryptionAlgorithm.AES_256, 256);
    console.log('Data encryption key generated:', dataKey);
    
    // Generar clave para firmas
    const signatureKey = keyManager.generateKey('api_signature', EncryptionAlgorithm.SHA_256, 256);
    console.log('Signature key generated:', signatureKey);
    
    // Verificar salud de claves
    const healthReport = keyManager.checkKeyHealth();
    console.log('Key health report:', healthReport);
    
    // Rotar clave de datos
    await keyManager.rotateKey('data_encryption');
    console.log('Data encryption key rotated successfully');
    
  } catch (error) {
    console.error('Error testing advanced key manager:', error);
  }
};
```

## Resumen de la Clase

En esta clase hemos cubierto:

✅ **Sistemas de encriptación** con algoritmos AES y RSA
✅ **Almacenamiento seguro** usando Keychain y encriptación
✅ **Protección de APIs** con encriptación de payloads
✅ **Gestión de claves** con rotación automática
✅ **Ejercicios prácticos** para implementar sistemas completos

## Próximos Pasos

En la siguiente clase aprenderemos sobre:
- **Seguridad de Red y API**
- **HTTPS y certificados SSL**
- **Protección contra ataques comunes**
- **Headers de seguridad**

## Recursos Adicionales

- [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [React Native Security Best Practices](https://reactnative.dev/docs/security)
- [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
- [Keychain Services](https://developer.apple.com/documentation/security/keychain_services)

---

**Nota**: La encriptación es fundamental para proteger datos sensibles, pero debe implementarse correctamente. Siempre usa algoritmos estándar de la industria y gestiona las claves de forma segura.
