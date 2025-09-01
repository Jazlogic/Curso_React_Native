# Clase 4: Seguridad de Red y API 🔐

## Información de la Clase

- **Módulo**: 14 - Seguridad
- **Clase**: 4 de 5
- **Duración**: 2 horas
- **Tipo**: Teórica + Práctica

## Navegación

- **Anterior**: [Clase 3: Encriptación y Protección de Datos](clase_3_encriptacion_proteccion_datos.md)
- **Siguiente**: [Clase 5: Auditoría y Cumplimiento](clase_5_auditoria_cumplimiento.md)
- **Arriba**: [README del Módulo](README.md)

## Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Implementar** HTTPS y certificados SSL correctamente
2. **Configurar** headers de seguridad para APIs
3. **Proteger** contra ataques comunes de red
4. **Implementar** rate limiting y protección DDoS
5. **Configurar** firewalls y filtros de red

## Contenido Teórico

### 1. HTTPS y Certificados SSL

#### A. **Configuración de HTTPS**

```typescript
// Configuración de seguridad de red
interface NetworkSecurityConfig {
  requireHTTPS: boolean;
  certificatePinning: boolean;
  allowedProtocols: string[];
  allowedCipherSuites: string[];
  minTLSVersion: string;
  maxTLSVersion: string;
}

// Clase para gestión de certificados SSL
class SSLCertificateManager {
  private config: NetworkSecurityConfig;
  private pinnedCertificates: Set<string>;
  
  constructor(config: NetworkSecurityConfig) {
    this.config = config;
    this.pinnedCertificates = new Set();
  }
  
  // Verificar certificado SSL
  async verifyCertificate(url: string): Promise<CertificateVerificationResult> {
    try {
      // Obtener información del certificado
      const certificateInfo = await this.getCertificateInfo(url);
      
      // Verificar validez del certificado
      const isValid = await this.validateCertificate(certificateInfo);
      
      // Verificar pinning si está habilitado
      const isPinned = this.config.certificatePinning ? 
        this.verifyCertificatePinning(certificateInfo) : true;
      
      return {
        isValid,
        isPinned,
        certificateInfo,
        timestamp: Date.now(),
      };
    } catch (error) {
      return {
        isValid: false,
        isPinned: false,
        error: error.message,
        timestamp: Date.now(),
      };
    }
  }
  
  // Obtener información del certificado
  private async getCertificateInfo(url: string): Promise<CertificateInfo> {
    // En producción, usar librería como react-native-ssl-pinning
    const response = await fetch(url, { method: 'HEAD' });
    
    // Simular información del certificado
    return {
      issuer: 'Let\'s Encrypt',
      subject: new URL(url).hostname,
      validFrom: new Date(),
      validTo: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000),
      serialNumber: '1234567890',
      fingerprint: 'sha256:abc123...',
    };
  }
  
  // Validar certificado
  private async validateCertificate(certInfo: CertificateInfo): Promise<boolean> {
    const now = new Date();
    return certInfo.validFrom <= now && certInfo.validTo >= now;
  }
  
  // Verificar pinning de certificado
  private verifyCertificatePinning(certInfo: CertificateInfo): boolean {
    return this.pinnedCertificates.has(certInfo.fingerprint);
  }
  
  // Agregar certificado pinneado
  addPinnedCertificate(fingerprint: string): void {
    this.pinnedCertificates.add(fingerprint);
  }
  
  // Remover certificado pinneado
  removePinnedCertificate(fingerprint: string): boolean {
    return this.pinnedCertificates.delete(fingerprint);
  }
}

// Estructuras para certificados
interface CertificateInfo {
  issuer: string;
  subject: string;
  validFrom: Date;
  validTo: Date;
  serialNumber: string;
  fingerprint: string;
}

interface CertificateVerificationResult {
  isValid: boolean;
  isPinned: boolean;
  certificateInfo?: CertificateInfo;
  error?: string;
  timestamp: number;
}
```

#### B. **Configuración de TLS**

```typescript
// Configuración de TLS
interface TLSConfig {
  minVersion: string;
  maxVersion: string;
  cipherSuites: string[];
  keyExchange: string[];
  signatureAlgorithms: string[];
}

// Clase para configuración de TLS
class TLSConfigManager {
  private config: TLSConfig;
  
  constructor(config: TLSConfig) {
    this.config = config;
  }
  
  // Configurar cliente TLS
  configureTLSClient(): TLSClientConfig {
    return {
      minVersion: this.config.minVersion,
      maxVersion: this.config.maxVersion,
      cipherSuites: this.config.cipherSuites,
      keyExchange: this.config.keyExchange,
      signatureAlgorithms: this.config.signatureAlgorithms,
      secureProtocol: 'TLSv1_2_method',
    };
  }
  
  // Verificar compatibilidad de versión
  isVersionSupported(version: string): boolean {
    const versions = ['TLSv1.0', 'TLSv1.1', 'TLSv1.2', 'TLSv1.3'];
    const minIndex = versions.indexOf(this.config.minVersion);
    const maxIndex = versions.indexOf(this.config.maxVersion);
    const versionIndex = versions.indexOf(version);
    
    return versionIndex >= minIndex && versionIndex <= maxIndex;
  }
  
  // Obtener configuración recomendada
  getRecommendedConfig(): TLSConfig {
    return {
      minVersion: 'TLSv1.2',
      maxVersion: 'TLSv1.3',
      cipherSuites: [
        'TLS_AES_256_GCM_SHA384',
        'TLS_CHACHA20_POLY1305_SHA256',
        'TLS_AES_128_GCM_SHA256',
      ],
      keyExchange: ['ECDHE', 'DHE'],
      signatureAlgorithms: ['RSA-PSS', 'ECDSA', 'RSA'],
    };
  }
}

// Configuración del cliente TLS
interface TLSClientConfig {
  minVersion: string;
  maxVersion: string;
  cipherSuites: string[];
  keyExchange: string[];
  signatureAlgorithms: string[];
  secureProtocol: string;
}
```

### 2. Headers de Seguridad

#### A. **Headers de Seguridad Básicos**

```typescript
// Headers de seguridad para APIs
interface SecurityHeaders {
  'Content-Security-Policy': string;
  'X-Frame-Options': string;
  'X-Content-Type-Options': string;
  'X-XSS-Protection': string;
  'Referrer-Policy': string;
  'Strict-Transport-Security': string;
  'Permissions-Policy': string;
}

// Clase para gestión de headers de seguridad
class SecurityHeadersManager {
  private headers: SecurityHeaders;
  
  constructor() {
    this.headers = this.getDefaultHeaders();
  }
  
  // Obtener headers por defecto
  private getDefaultHeaders(): SecurityHeaders {
    return {
      'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';",
      'X-Frame-Options': 'DENY',
      'X-Content-Type-Options': 'nosniff',
      'X-XSS-Protection': '1; mode=block',
      'Referrer-Policy': 'strict-origin-when-cross-origin',
      'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
      'Permissions-Policy': 'geolocation=(), microphone=(), camera=()',
    };
  }
  
  // Personalizar header específico
  setHeader(header: keyof SecurityHeaders, value: string): void {
    this.headers[header] = value;
  }
  
  // Obtener todos los headers
  getAllHeaders(): SecurityHeaders {
    return { ...this.headers };
  }
  
  // Aplicar headers a respuesta HTTP
  applyHeadersToResponse(response: Response): Response {
    Object.entries(this.headers).forEach(([key, value]) => {
      response.headers.set(key, value);
    });
    return response;
  }
  
  // Validar headers
  validateHeaders(): HeaderValidationResult[] {
    const results: HeaderValidationResult[] = [];
    
    // Validar CSP
    if (!this.headers['Content-Security-Policy'].includes("default-src 'self'")) {
      results.push({
        header: 'Content-Security-Policy',
        severity: 'HIGH',
        message: 'CSP should include default-src directive',
      });
    }
    
    // Validar HSTS
    if (!this.headers['Strict-Transport-Security'].includes('max-age=')) {
      results.push({
        header: 'Strict-Transport-Security',
        severity: 'HIGH',
        message: 'HSTS should include max-age directive',
      });
    }
    
    return results;
  }
}

// Resultado de validación de headers
interface HeaderValidationResult {
  header: string;
  severity: 'LOW' | 'MEDIUM' | 'HIGH';
  message: string;
}
```

#### B. **Content Security Policy (CSP)**

```typescript
// Configuración de CSP
interface CSPConfig {
  defaultSrc: string[];
  scriptSrc: string[];
  styleSrc: string[];
  imgSrc: string[];
  connectSrc: string[];
  fontSrc: string[];
  objectSrc: string[];
  mediaSrc: string[];
  frameSrc: string[];
}

// Clase para gestión de CSP
class CSPManager {
  private config: CSPConfig;
  
  constructor(config: CSPConfig) {
    this.config = config;
  }
  
  // Generar string de CSP
  generateCSPString(): string {
    const directives: string[] = [];
    
    if (this.config.defaultSrc.length > 0) {
      directives.push(`default-src ${this.config.defaultSrc.join(' ')}`);
    }
    
    if (this.config.scriptSrc.length > 0) {
      directives.push(`script-src ${this.config.scriptSrc.join(' ')}`);
    }
    
    if (this.config.styleSrc.length > 0) {
      directives.push(`style-src ${this.config.styleSrc.join(' ')}`);
    }
    
    if (this.config.imgSrc.length > 0) {
      directives.push(`img-src ${this.config.imgSrc.join(' ')}`);
    }
    
    if (this.config.connectSrc.length > 0) {
      directives.push(`connect-src ${this.config.connectSrc.join(' ')}`);
    }
    
    return directives.join('; ');
  }
  
  // Validar configuración de CSP
  validateCSPConfig(): CSPValidationResult[] {
    const results: CSPValidationResult[] = [];
    
    // Verificar que default-src esté configurado
    if (this.config.defaultSrc.length === 0) {
      results.push({
        directive: 'default-src',
        severity: 'HIGH',
        message: 'default-src is required for CSP',
      });
    }
    
    // Verificar que no se use 'unsafe-inline' en script-src
    if (this.config.scriptSrc.includes("'unsafe-inline'")) {
      results.push({
        directive: 'script-src',
        severity: 'MEDIUM',
        message: 'Consider avoiding unsafe-inline for better security',
      });
    }
    
    return results;
  }
  
  // Obtener configuración recomendada
  static getRecommendedConfig(): CSPConfig {
    return {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    };
  }
}

// Resultado de validación de CSP
interface CSPValidationResult {
  directive: string;
  severity: 'LOW' | 'MEDIUM' | 'HIGH';
  message: string;
}
```

### 3. Protección contra Ataques Comunes

#### A. **Rate Limiting y Protección DDoS**

```typescript
// Configuración de rate limiting
interface RateLimitConfig {
  windowMs: number;
  maxRequests: number;
  skipSuccessfulRequests: boolean;
  skipFailedRequests: boolean;
  keyGenerator: (req: any) => string;
}

// Clase para rate limiting
class RateLimiter {
  private config: RateLimitConfig;
  private requestCounts: Map<string, RequestCount>;
  
  constructor(config: RateLimitConfig) {
    this.config = config;
    this.requestCounts = new Map();
  }
  
  // Verificar si la petición está permitida
  isRequestAllowed(identifier: string): RateLimitResult {
    const now = Date.now();
    const requestCount = this.requestCounts.get(identifier);
    
    if (!requestCount) {
      // Primera petición
      this.requestCounts.set(identifier, {
        count: 1,
        resetTime: now + this.config.windowMs,
      });
      return { allowed: true, remaining: this.config.maxRequests - 1 };
    }
    
    // Verificar si la ventana de tiempo ha expirado
    if (now > requestCount.resetTime) {
      // Resetear contador
      this.requestCounts.set(identifier, {
        count: 1,
        resetTime: now + this.config.windowMs,
      });
      return { allowed: true, remaining: this.config.maxRequests - 1 };
    }
    
    // Verificar límite
    if (requestCount.count >= this.config.maxRequests) {
      return {
        allowed: false,
        remaining: 0,
        resetTime: requestCount.resetTime,
        retryAfter: Math.ceil((requestCount.resetTime - now) / 1000),
      };
    }
    
    // Incrementar contador
    requestCount.count++;
    this.requestCounts.set(identifier, requestCount);
    
    return {
      allowed: true,
      remaining: this.config.maxRequests - requestCount.count,
    };
  }
  
  // Limpiar contadores expirados
  cleanupExpiredCounters(): void {
    const now = Date.now();
    for (const [identifier, requestCount] of this.requestCounts.entries()) {
      if (now > requestCount.resetTime) {
        this.requestCounts.delete(identifier);
      }
    }
  }
  
  // Obtener estadísticas
  getStatistics(): RateLimitStats {
    const totalIdentifiers = this.requestCounts.size;
    let totalRequests = 0;
    
    for (const requestCount of this.requestCounts.values()) {
      totalRequests += requestCount.count;
    }
    
    return {
      totalIdentifiers,
      totalRequests,
      averageRequestsPerIdentifier: totalIdentifiers > 0 ? totalRequests / totalIdentifiers : 0,
    };
  }
}

// Estructuras para rate limiting
interface RequestCount {
  count: number;
  resetTime: number;
}

interface RateLimitResult {
  allowed: boolean;
  remaining: number;
  resetTime?: number;
  retryAfter?: number;
}

interface RateLimitStats {
  totalIdentifiers: number;
  totalRequests: number;
  averageRequestsPerIdentifier: number;
}
```

#### B. **Protección contra Ataques de Inyección**

```typescript
// Clase para protección contra inyección
class InjectionProtection {
  
  // Sanitizar entrada de usuario
  static sanitizeInput(input: string): string {
    return input
      .replace(/[<>]/g, '') // Remover < y >
      .replace(/javascript:/gi, '') // Remover javascript:
      .replace(/on\w+\s*=/gi, '') // Remover event handlers
      .trim();
  }
  
  // Validar entrada SQL
  static validateSQLInput(input: string): boolean {
    const sqlPatterns = [
      /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION)\b)/i,
      /(\b(OR|AND)\s+\d+\s*=\s*\d+)/i,
      /(\b(OR|AND)\s+['"]\w+['"]\s*=\s*['"]\w+['"])/i,
      /(--|\/\*|\*\/)/,
      /(\bxp_cmdshell\b)/i,
    ];
    
    return !sqlPatterns.some(pattern => pattern.test(input));
  }
  
  // Validar entrada XSS
  static validateXSSInput(input: string): boolean {
    const xssPatterns = [
      /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi,
      /javascript:/gi,
      /on\w+\s*=/gi,
      /<iframe\b[^<]*(?:(?!<\/iframe>)<[^<]*)*<\/iframe>/gi,
      /<object\b[^<]*(?:(?!<\/object>)<[^<]*)*<\/object>/gi,
    ];
    
    return !xssPatterns.some(pattern => pattern.test(input));
  }
  
  // Escapar HTML
  static escapeHTML(input: string): string {
    const htmlEscapes: Record<string, string> = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#x27;',
      '/': '&#x2F;',
    };
    
    return input.replace(/[&<>"'/]/g, (match) => htmlEscapes[match]);
  }
  
  // Validar URL
  static validateURL(url: string): boolean {
    try {
      const parsed = new URL(url);
      return ['http:', 'https:'].includes(parsed.protocol);
    } catch {
      return false;
    }
  }
}
```

### 4. Firewalls y Filtros de Red

#### A. **Firewall de Aplicación**

```typescript
// Configuración de firewall
interface FirewallConfig {
  enabled: boolean;
  blockList: string[];
  allowList: string[];
  maxRequestSize: number;
  blockSuspiciousPatterns: boolean;
  logBlockedRequests: boolean;
}

// Clase para firewall de aplicación
class ApplicationFirewall {
  private config: FirewallConfig;
  private blockedIPs: Set<string>;
  private requestLogs: RequestLog[];
  
  constructor(config: FirewallConfig) {
    this.config = config;
    this.blockedIPs = new Set();
    this.requestLogs = [];
  }
  
  // Verificar si la petición está permitida
  isRequestAllowed(request: RequestInfo): FirewallResult {
    try {
      // Verificar IP bloqueada
      const clientIP = this.extractClientIP(request);
      if (this.blockedIPs.has(clientIP)) {
        this.logBlockedRequest(request, 'IP_BLOCKED');
        return { allowed: false, reason: 'IP_BLOCKED' };
      }
      
      // Verificar lista de bloqueo
      if (this.config.blockList.includes(clientIP)) {
        this.blockedIPs.add(clientIP);
        this.logBlockedRequest(request, 'IP_IN_BLOCKLIST');
        return { allowed: false, reason: 'IP_IN_BLOCKLIST' };
      }
      
      // Verificar lista de permitidos
      if (this.config.allowList.length > 0 && !this.config.allowList.includes(clientIP)) {
        this.logBlockedRequest(request, 'IP_NOT_IN_ALLOWLIST');
        return { allowed: false, reason: 'IP_NOT_IN_ALLOWLIST' };
      }
      
      // Verificar patrones sospechosos
      if (this.config.blockSuspiciousPatterns && this.hasSuspiciousPatterns(request)) {
        this.logBlockedRequest(request, 'SUSPICIOUS_PATTERNS');
        return { allowed: false, reason: 'SUSPICIOUS_PATTERNS' };
      }
      
      return { allowed: true };
    } catch (error) {
      this.logBlockedRequest(request, 'ERROR');
      return { allowed: false, reason: 'ERROR', error: error.message };
    }
  }
  
  // Extraer IP del cliente
  private extractClientIP(request: RequestInfo): string {
    // En producción, extraer IP real del request
    return '192.168.1.1'; // Simulado
  }
  
  // Verificar patrones sospechosos
  private hasSuspiciousPatterns(request: RequestInfo): boolean {
    const suspiciousPatterns = [
      /\.\.\//, // Directory traversal
      /<script/i, // XSS
      /union\s+select/i, // SQL injection
      /eval\s*\(/i, // Code injection
    ];
    
    const requestString = JSON.stringify(request);
    return suspiciousPatterns.some(pattern => pattern.test(requestString));
  }
  
  // Bloquear IP
  blockIP(ip: string): void {
    this.blockedIPs.add(ip);
  }
  
  // Desbloquear IP
  unblockIP(ip: string): boolean {
    return this.blockedIPs.delete(ip);
  }
  
  // Obtener IPs bloqueadas
  getBlockedIPs(): string[] {
    return Array.from(this.blockedIPs);
  }
  
  // Log de petición bloqueada
  private logBlockedRequest(request: RequestInfo, reason: string): void {
    if (this.config.logBlockedRequests) {
      this.requestLogs.push({
        timestamp: new Date(),
        request: request,
        reason: reason,
        clientIP: this.extractClientIP(request),
      });
    }
  }
  
  // Obtener logs de peticiones bloqueadas
  getBlockedRequestLogs(): RequestLog[] {
    return [...this.requestLogs];
  }
}

// Estructuras para firewall
interface FirewallResult {
  allowed: boolean;
  reason?: string;
  error?: string;
}

interface RequestLog {
  timestamp: Date;
  request: RequestInfo;
  reason: string;
  clientIP: string;
}
```

## Ejercicios Prácticos

### Ejercicio 1: Implementar Verificación de Certificados SSL

**Objetivo**: Crear un sistema completo de verificación de certificados SSL.

```typescript
// Implementa la clase SSLSecurityManager
class SSLSecurityManager {
  private certificateManager: SSLCertificateManager;
  private tlsConfigManager: TLSConfigManager;
  
  constructor() {
    const networkConfig: NetworkSecurityConfig = {
      requireHTTPS: true,
      certificatePinning: true,
      allowedProtocols: ['TLSv1.2', 'TLSv1.3'],
      allowedCipherSuites: ['TLS_AES_256_GCM_SHA384', 'TLS_CHACHA20_POLY1305_SHA256'],
      minTLSVersion: 'TLSv1.2',
      maxTLSVersion: 'TLSv1.3',
    };
    
    const tlsConfig: TLSConfig = {
      minVersion: 'TLSv1.2',
      maxVersion: 'TLSv1.3',
      cipherSuites: ['TLS_AES_256_GCM_SHA384', 'TLS_CHACHA20_POLY1305_SHA256'],
      keyExchange: ['ECDHE', 'DHE'],
      signatureAlgorithms: ['RSA-PSS', 'ECDSA'],
    };
    
    this.certificateManager = new SSLCertificateManager(networkConfig);
    this.tlsConfigManager = new TLSConfigManager(tlsConfig);
  }
  
  // Verificar seguridad de URL
  async verifyURLSecurity(url: string): Promise<SecurityVerificationResult> {
    // TODO: Implementar verificación completa
    // Debe incluir:
    // - Verificación de certificado SSL
    // - Verificación de versión TLS
    // - Verificación de pinning
    // - Análisis de headers de seguridad
  }
  
  // Configurar certificados pinneados
  configureCertificatePinning(certificates: string[]): void {
    // TODO: Implementar configuración de pinning
    // Debe incluir:
    // - Validación de certificados
    // - Almacenamiento seguro
    // - Verificación de integridad
  }
  
  // Verificar configuración TLS
  verifyTLSConfiguration(): TLSVerificationResult {
    // TODO: Implementar verificación de TLS
    // Debe incluir:
    // - Verificación de versiones soportadas
    // - Verificación de cipher suites
    // - Verificación de algoritmos de firma
  }
}

// Prueba tu implementación
const testSSLSecurityManager = async () => {
  const sslManager = new SSLSecurityManager();
  
  try {
    // Verificar seguridad de URL
    const securityResult = await sslManager.verifyURLSecurity('https://api.example.com');
    console.log('Security verification result:', securityResult);
    
    // Configurar certificados pinneados
    sslManager.configureCertificatePinning([
      'sha256:abc123...',
      'sha256:def456...',
    ]);
    
    // Verificar configuración TLS
    const tlsResult = sslManager.verifyTLSConfiguration();
    console.log('TLS verification result:', tlsResult);
    
  } catch (error) {
    console.error('Error testing SSL security manager:', error);
  }
};
```

### Ejercicio 2: Sistema de Rate Limiting Avanzado

**Objetivo**: Implementar un sistema robusto de rate limiting con múltiples estrategias.

```typescript
// Implementa la clase AdvancedRateLimiter
class AdvancedRateLimiter {
  private limiters: Map<string, RateLimiter>;
  private globalLimiter: RateLimiter;
  
  constructor() {
    this.limiters = new Map();
    this.globalLimiter = new RateLimiter({
      windowMs: 60000, // 1 minuto
      maxRequests: 1000, // 1000 peticiones por minuto globalmente
      skipSuccessfulRequests: false,
      skipFailedRequests: false,
      keyGenerator: () => 'global',
    });
  }
  
  // Crear rate limiter para endpoint específico
  createEndpointLimiter(endpoint: string, config: RateLimitConfig): void {
    // TODO: Implementar creación de limitador
    // Debe incluir:
    // - Validación de configuración
    // - Creación de limitador
    // - Almacenamiento en mapa
  }
  
  // Verificar rate limit para endpoint
  checkEndpointRateLimit(endpoint: string, identifier: string): RateLimitResult {
    // TODO: Implementar verificación de endpoint
    // Debe incluir:
    // - Verificación de limitador específico
    // - Verificación de limitador global
    // - Retorno de resultado combinado
  }
  
  // Obtener estadísticas de rate limiting
  getRateLimitStatistics(): RateLimitStatistics {
    // TODO: Implementar estadísticas
    // Debe incluir:
    // - Estadísticas por endpoint
    // - Estadísticas globales
    // - IPs más activas
  }
  
  // Configurar rate limiting dinámico
  configureDynamicRateLimiting(endpoint: string, loadFactor: number): void {
    // TODO: Implementar configuración dinámica
    // Debe incluir:
    // - Ajuste basado en carga del sistema
    // - Ajuste basado en comportamiento del usuario
    // - Ajuste basado en recursos disponibles
  }
}

// Prueba tu implementación
const testAdvancedRateLimiter = () => {
  const rateLimiter = new AdvancedRateLimiter();
  
  try {
    // Crear limitador para endpoint de login
    rateLimiter.createEndpointLimiter('/auth/login', {
      windowMs: 300000, // 5 minutos
      maxRequests: 5, // 5 intentos de login por 5 minutos
      skipSuccessfulRequests: true,
      skipFailedRequests: false,
      keyGenerator: (req) => req.ip || 'unknown',
    });
    
    // Crear limitador para endpoint de API
    rateLimiter.createEndpointLimiter('/api/data', {
      windowMs: 60000, // 1 minuto
      maxRequests: 100, // 100 peticiones por minuto
      skipSuccessfulRequests: false,
      skipFailedRequests: false,
      keyGenerator: (req) => req.userId || req.ip || 'anonymous',
    });
    
    // Verificar rate limit
    const loginResult = rateLimiter.checkEndpointRateLimit('/auth/login', '192.168.1.1');
    console.log('Login rate limit result:', loginResult);
    
    const apiResult = rateLimiter.checkEndpointRateLimit('/api/data', 'user123');
    console.log('API rate limit result:', apiResult);
    
    // Obtener estadísticas
    const stats = rateLimiter.getRateLimitStatistics();
    console.log('Rate limit statistics:', stats);
    
  } catch (error) {
    console.error('Error testing advanced rate limiter:', error);
  }
};
```

### Ejercicio 3: Firewall de Aplicación Completo

**Objetivo**: Implementar un firewall de aplicación con múltiples capas de protección.

```typescript
// Implementa la clase ComprehensiveFirewall
class ComprehensiveFirewall {
  private firewall: ApplicationFirewall;
  private injectionProtection: InjectionProtection;
  private securityHeaders: SecurityHeadersManager;
  
  constructor() {
    const firewallConfig: FirewallConfig = {
      enabled: true,
      blockList: ['192.168.1.100', '10.0.0.50'],
      allowList: [],
      maxRequestSize: 10 * 1024 * 1024, // 10MB
      blockSuspiciousPatterns: true,
      logBlockedRequests: true,
    };
    
    this.firewall = new ApplicationFirewall(firewallConfig);
    this.injectionProtection = new InjectionProtection();
    this.securityHeaders = new SecurityHeadersManager();
  }
  
  // Procesar petición a través del firewall
  async processRequest(request: RequestInfo): Promise<FirewallProcessingResult> {
    // TODO: Implementar procesamiento completo
    // Debe incluir:
    // - Verificación de firewall básico
    // - Protección contra inyección
    // - Validación de headers
    // - Análisis de contenido
  }
  
  // Configurar reglas personalizadas
  addCustomRule(rule: CustomFirewallRule): void {
    // TODO: Implementar reglas personalizadas
    // Debe incluir:
    // - Validación de regla
    // - Almacenamiento de regla
    // - Aplicación de regla
  }
  
  // Analizar tráfico en tiempo real
  analyzeTraffic(): TrafficAnalysisResult {
    // TODO: Implementar análisis de tráfico
    // Debe incluir:
    // - Análisis de patrones
    // - Detección de anomalías
    // - Estadísticas de tráfico
  }
  
  // Generar reporte de seguridad
  generateSecurityReport(): SecurityReport {
    // TODO: Implementar reporte de seguridad
    // Debe incluir:
    // - Resumen de amenazas bloqueadas
    // - Estadísticas de tráfico
    // - Recomendaciones de seguridad
  }
}

// Prueba tu implementación
const testComprehensiveFirewall = async () => {
  const firewall = new ComprehensiveFirewall();
  
  try {
    // Procesar petición
    const request = {
      method: 'POST',
      url: '/api/users',
      headers: { 'Content-Type': 'application/json' },
      body: { name: 'John', email: 'john@example.com' },
    };
    
    const result = await firewall.processRequest(request);
    console.log('Firewall processing result:', result);
    
    // Agregar regla personalizada
    firewall.addCustomRule({
      name: 'block_suspicious_user_agents',
      pattern: /bot|crawler|spider/i,
      action: 'BLOCK',
      priority: 'HIGH',
    });
    
    // Analizar tráfico
    const trafficAnalysis = firewall.analyzeTraffic();
    console.log('Traffic analysis:', trafficAnalysis);
    
    // Generar reporte de seguridad
    const securityReport = firewall.generateSecurityReport();
    console.log('Security report:', securityReport);
    
  } catch (error) {
    console.error('Error testing comprehensive firewall:', error);
  }
};
```

## Resumen de la Clase

En esta clase hemos cubierto:

✅ **HTTPS y certificados SSL** con verificación y pinning
✅ **Headers de seguridad** incluyendo CSP y HSTS
✅ **Protección contra ataques** de inyección y XSS
✅ **Rate limiting y protección DDoS** con múltiples estrategias
✅ **Firewalls de aplicación** con reglas personalizables

## Próximos Pasos

En la siguiente clase aprenderemos sobre:
- **Auditoría y Cumplimiento**
- **Logs de seguridad**
- **Cumplimiento GDPR/CCPA**
- **Sistemas de monitoreo de seguridad**

## Recursos Adicionales

- [OWASP Transport Layer Protection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [Mozilla Security Guidelines](https://infosec.mozilla.org/guidelines/)
- [Security Headers](https://securityheaders.com/)
- [SSL Labs](https://www.ssllabs.com/)

---

**Nota**: La seguridad de red es fundamental para proteger las comunicaciones de tu aplicación. Siempre implementa HTTPS, configura headers de seguridad apropiados y monitorea el tráfico en busca de amenazas.
