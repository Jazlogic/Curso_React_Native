# Clase 5: Auditoría y Cumplimiento 🔐

## Información de la Clase

- **Módulo**: 14 - Seguridad
- **Clase**: 5 de 5
- **Duración**: 1.5 horas
- **Tipo**: Teórica + Práctica

## Navegación

- **Anterior**: [Clase 4: Seguridad de Red y API](clase_4_seguridad_red_api.md)
- **Siguiente**: Módulo 15: Testing Avanzado (pendiente)
- **Arriba**: [README del Módulo](README.md)

## Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Implementar** sistemas de auditoría de seguridad
2. **Configurar** logs de seguridad y monitoreo
3. **Aplicar** cumplimiento GDPR/CCPA
4. **Generar** reportes de cumplimiento
5. **Implementar** sistemas de alertas de seguridad

## Contenido Teórico

### 1. Fundamentos de Auditoría de Seguridad

#### A. **¿Qué es la Auditoría de Seguridad?**

La auditoría de seguridad es el proceso sistemático de evaluar y verificar que una aplicación cumple con los estándares de seguridad establecidos.

```typescript
// Configuración de auditoría
interface AuditConfig {
  enabled: boolean;
  logLevel: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  retentionPeriod: number; // días
  encryptionEnabled: boolean;
  realTimeAlerts: boolean;
  complianceStandards: string[];
}

// Clase para gestión de auditoría
class SecurityAuditor {
  private config: AuditConfig;
  private auditLogs: AuditLog[];
  
  constructor(config: AuditConfig) {
    this.config = config;
    this.auditLogs = [];
  }
  
  // Registrar evento de auditoría
  logAuditEvent(event: AuditEvent): void {
    const auditLog: AuditLog = {
      timestamp: new Date(),
      event: event,
      severity: this.calculateSeverity(event),
      metadata: this.collectMetadata(event),
    };
    
    this.auditLogs.push(auditLog);
    
    // Alertas en tiempo real
    if (this.config.realTimeAlerts && auditLog.severity === 'CRITICAL') {
      this.sendRealTimeAlert(auditLog);
    }
  }
  
  // Calcular severidad del evento
  private calculateSeverity(event: AuditEvent): AuditSeverity {
    const criticalEvents = ['LOGIN_FAILURE', 'UNAUTHORIZED_ACCESS', 'DATA_BREACH'];
    const highEvents = ['PERMISSION_VIOLATION', 'SUSPICIOUS_ACTIVITY'];
    
    if (criticalEvents.includes(event.type)) return 'CRITICAL';
    if (highEvents.includes(event.type)) return 'HIGH';
    if (event.type === 'LOGIN_SUCCESS') return 'LOW';
    
    return 'MEDIUM';
  }
  
  // Recolectar metadatos del evento
  private collectMetadata(event: AuditEvent): AuditMetadata {
    return {
      userId: event.userId,
      ipAddress: event.ipAddress,
      userAgent: event.userAgent,
      sessionId: event.sessionId,
      resource: event.resource,
    };
  }
  
  // Enviar alerta en tiempo real
  private sendRealTimeAlert(auditLog: AuditLog): void {
    // Implementar notificación (email, Slack, etc.)
    console.log('🚨 CRITICAL SECURITY ALERT:', auditLog);
  }
}

// Estructuras para auditoría
interface AuditEvent {
  type: string;
  userId?: string;
  ipAddress?: string;
  userAgent?: string;
  sessionId?: string;
  resource?: string;
  details: string;
}

interface AuditLog {
  timestamp: Date;
  event: AuditEvent;
  severity: AuditSeverity;
  metadata: AuditMetadata;
}

interface AuditMetadata {
  userId?: string;
  ipAddress?: string;
  userAgent?: string;
  sessionId?: string;
  resource?: string;
}

type AuditSeverity = 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
```

#### B. **Tipos de Eventos de Auditoría**

```typescript
// Enumeración de eventos de auditoría
enum AuditEventType {
  // Eventos de autenticación
  LOGIN_SUCCESS = 'LOGIN_SUCCESS',
  LOGIN_FAILURE = 'LOGIN_FAILURE',
  LOGOUT = 'LOGOUT',
  PASSWORD_CHANGE = 'PASSWORD_CHANGE',
  
  // Eventos de autorización
  PERMISSION_GRANTED = 'PERMISSION_GRANTED',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  ROLE_CHANGE = 'ROLE_CHANGE',
  
  // Eventos de datos
  DATA_ACCESS = 'DATA_ACCESS',
  DATA_MODIFICATION = 'DATA_MODIFICATION',
  DATA_DELETION = 'DATA_DELETION',
  DATA_EXPORT = 'DATA_EXPORT',
  
  // Eventos de seguridad
  SUSPICIOUS_ACTIVITY = 'SUSPICIOUS_ACTIVITY',
  UNAUTHORIZED_ACCESS = 'UNAUTHORIZED_ACCESS',
  DATA_BREACH = 'DATA_BREACH',
  
  // Eventos del sistema
  CONFIGURATION_CHANGE = 'CONFIGURATION_CHANGE',
  SYSTEM_STARTUP = 'SYSTEM_STARTUP',
  SYSTEM_SHUTDOWN = 'SYSTEM_SHUTDOWN',
}

// Clase para eventos de auditoría específicos
class AuditEventFactory {
  
  // Crear evento de login
  static createLoginEvent(
    userId: string,
    success: boolean,
    ipAddress: string,
    userAgent: string
  ): AuditEvent {
    return {
      type: success ? AuditEventType.LOGIN_SUCCESS : AuditEventType.LOGIN_FAILURE,
      userId,
      ipAddress,
      userAgent,
      details: success ? 'User logged in successfully' : 'Failed login attempt',
    };
  }
  
  // Crear evento de acceso a datos
  static createDataAccessEvent(
    userId: string,
    resource: string,
    action: 'READ' | 'WRITE' | 'DELETE',
    ipAddress: string
  ): AuditEvent {
    return {
      type: AuditEventType.DATA_ACCESS,
      userId,
      resource,
      ipAddress,
      details: `User ${action.toLowerCase()}ed resource: ${resource}`,
    };
  }
  
  // Crear evento de actividad sospechosa
  static createSuspiciousActivityEvent(
    userId: string,
    activity: string,
    ipAddress: string,
    riskLevel: 'LOW' | 'MEDIUM' | 'HIGH'
  ): AuditEvent {
    return {
      type: AuditEventType.SUSPICIOUS_ACTIVITY,
      userId,
      ipAddress,
      details: `Suspicious activity detected: ${activity} (Risk: ${riskLevel})`,
    };
  }
}
```

### 2. Sistema de Logs de Seguridad

#### A. **Gestión de Logs**

```typescript
// Clase para gestión de logs de seguridad
class SecurityLogger {
  private logs: SecurityLog[];
  private maxLogs: number;
  private encryptionKey: string;
  
  constructor(maxLogs: number = 10000, encryptionKey: string) {
    this.logs = [];
    this.maxLogs = maxLogs;
    this.encryptionKey = encryptionKey;
  }
  
  // Agregar log de seguridad
  addLog(log: SecurityLog): void {
    // Encriptar datos sensibles
    const encryptedLog = this.encryptSensitiveData(log);
    
    this.logs.push(encryptedLog);
    
    // Mantener límite de logs
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }
  }
  
  // Buscar logs por criterios
  searchLogs(criteria: LogSearchCriteria): SecurityLog[] {
    return this.logs.filter(log => {
      if (criteria.userId && log.userId !== criteria.userId) return false;
      if (criteria.eventType && log.eventType !== criteria.eventType) return false;
      if (criteria.severity && log.severity !== criteria.severity) return false;
      if (criteria.startDate && log.timestamp < criteria.startDate) return false;
      if (criteria.endDate && log.timestamp > criteria.endDate) return false;
      if (criteria.ipAddress && log.ipAddress !== criteria.ipAddress) return false;
      
      return true;
    });
  }
  
  // Generar reporte de logs
  generateLogReport(timeRange: TimeRange): LogReport {
    const logsInRange = this.logs.filter(log => 
      log.timestamp >= timeRange.start && log.timestamp <= timeRange.end
    );
    
    const eventTypeCounts = this.countEventTypes(logsInRange);
    const severityCounts = this.countSeverities(logsInRange);
    const topUsers = this.getTopUsers(logsInRange);
    const topIPs = this.getTopIPs(logsInRange);
    
    return {
      timeRange,
      totalLogs: logsInRange.length,
      eventTypeCounts,
      severityCounts,
      topUsers,
      topIPs,
      generatedAt: new Date(),
    };
  }
  
  // Encriptar datos sensibles
  private encryptSensitiveData(log: SecurityLog): SecurityLog {
    // En producción, usar librería de encriptación real
    return {
      ...log,
      userId: log.userId ? this.encrypt(log.userId) : undefined,
      ipAddress: log.ipAddress ? this.encrypt(log.ipAddress) : undefined,
    };
  }
  
  // Encriptar string (simulado)
  private encrypt(text: string): string {
    return btoa(text);
  }
  
  // Contar tipos de eventos
  private countEventTypes(logs: SecurityLog[]): Record<string, number> {
    const counts: Record<string, number> = {};
    logs.forEach(log => {
      counts[log.eventType] = (counts[log.eventType] || 0) + 1;
    });
    return counts;
  }
  
  // Contar severidades
  private countSeverities(logs: SecurityLog[]): Record<string, number> {
    const counts: Record<string, number> = {};
    logs.forEach(log => {
      counts[log.severity] = (counts[log.severity] || 0) + 1;
    });
    return counts;
  }
  
  // Obtener usuarios más activos
  private getTopUsers(logs: SecurityLog[], limit: number = 10): UserActivity[] {
    const userCounts: Record<string, number> = {};
    logs.forEach(log => {
      if (log.userId) {
        userCounts[log.userId] = (userCounts[log.userId] || 0) + 1;
      }
    });
    
    return Object.entries(userCounts)
      .map(([userId, count]) => ({ userId, count }))
      .sort((a, b) => b.count - a.count)
      .slice(0, limit);
  }
  
  // Obtener IPs más activas
  private getTopIPs(logs: SecurityLog[], limit: number = 10): IPActivity[] {
    const ipCounts: Record<string, number> = {};
    logs.forEach(log => {
      if (log.ipAddress) {
        ipCounts[log.ipAddress] = (ipCounts[log.ipAddress] || 0) + 1;
      }
    });
    
    return Object.entries(ipCounts)
      .map(([ip, count]) => ({ ip, count }))
      .sort((a, b) => b.count - a.count)
      .slice(0, limit);
  }
}

// Estructuras para logs
interface SecurityLog {
  timestamp: Date;
  eventType: string;
  severity: string;
  userId?: string;
  ipAddress?: string;
  userAgent?: string;
  details: string;
  metadata?: Record<string, any>;
}

interface LogSearchCriteria {
  userId?: string;
  eventType?: string;
  severity?: string;
  startDate?: Date;
  endDate?: Date;
  ipAddress?: string;
}

interface TimeRange {
  start: Date;
  end: Date;
}

interface LogReport {
  timeRange: TimeRange;
  totalLogs: number;
  eventTypeCounts: Record<string, number>;
  severityCounts: Record<string, number>;
  topUsers: UserActivity[];
  topIPs: IPActivity[];
  generatedAt: Date;
}

interface UserActivity {
  userId: string;
  count: number;
}

interface IPActivity {
  ip: string;
  count: number;
}
```

### 3. Cumplimiento GDPR/CCPA

#### A. **Gestión de Consentimiento**

```typescript
// Clase para gestión de consentimiento GDPR/CCPA
class ConsentManager {
  private consents: UserConsent[];
  private dataProcessingActivities: DataProcessingActivity[];
  
  constructor() {
    this.consents = [];
    this.dataProcessingActivities = [];
  }
  
  // Registrar consentimiento del usuario
  recordConsent(consent: UserConsent): void {
    // Verificar si ya existe un consentimiento previo
    const existingIndex = this.consents.findIndex(c => 
      c.userId === consent.userId && c.purpose === consent.purpose
    );
    
    if (existingIndex >= 0) {
      // Actualizar consentimiento existente
      this.consents[existingIndex] = {
        ...consent,
        updatedAt: new Date(),
        version: this.consents[existingIndex].version + 1,
      };
    } else {
      // Agregar nuevo consentimiento
      this.consents.push({
        ...consent,
        version: 1,
        createdAt: new Date(),
        updatedAt: new Date(),
      });
    }
  }
  
  // Verificar consentimiento válido
  hasValidConsent(userId: string, purpose: string): boolean {
    const consent = this.consents.find(c => 
      c.userId === userId && 
      c.purpose === purpose && 
      c.status === 'GRANTED'
    );
    
    if (!consent) return false;
    
    // Verificar que no haya expirado
    if (consent.expiresAt && consent.expiresAt < new Date()) {
      return false;
    }
    
    return true;
  }
  
  // Revocar consentimiento
  revokeConsent(userId: string, purpose: string): boolean {
    const consent = this.consents.find(c => 
      c.userId === userId && c.purpose === purpose
    );
    
    if (consent) {
      consent.status = 'REVOKED';
      consent.updatedAt = new Date();
      return true;
    }
    
    return false;
  }
  
  // Obtener historial de consentimientos
  getConsentHistory(userId: string): UserConsent[] {
    return this.consents
      .filter(c => c.userId === userId)
      .sort((a, b) => b.updatedAt.getTime() - a.updatedAt.getTime());
  }
  
  // Exportar datos del usuario (GDPR Art. 20)
  exportUserData(userId: string): UserDataExport {
    const userConsents = this.consents.filter(c => c.userId === userId);
    const userActivities = this.dataProcessingActivities.filter(a => a.userId === userId);
    
    return {
      userId,
      consents: userConsents,
      dataProcessingActivities: userActivities,
      exportedAt: new Date(),
      format: 'JSON',
    };
  }
  
  // Eliminar datos del usuario (GDPR Art. 17)
  deleteUserData(userId: string): boolean {
    try {
      // Eliminar consentimientos
      this.consents = this.consents.filter(c => c.userId !== userId);
      
      // Eliminar actividades de procesamiento
      this.dataProcessingActivities = this.dataProcessingActivities.filter(a => a.userId !== userId);
      
      return true;
    } catch (error) {
      console.error('Error deleting user data:', error);
      return false;
    }
  }
}

// Estructuras para consentimiento
interface UserConsent {
  userId: string;
  purpose: string;
  status: 'GRANTED' | 'DENIED' | 'REVOKED';
  legalBasis: 'CONSENT' | 'LEGITIMATE_INTEREST' | 'CONTRACT' | 'LEGAL_OBLIGATION';
  expiresAt?: Date;
  version: number;
  createdAt: Date;
  updatedAt: Date;
  ipAddress?: string;
  userAgent?: string;
}

interface DataProcessingActivity {
  userId: string;
  activity: string;
  purpose: string;
  dataCategories: string[];
  timestamp: Date;
  legalBasis: string;
}

interface UserDataExport {
  userId: string;
  consents: UserConsent[];
  dataProcessingActivities: DataProcessingActivity[];
  exportedAt: Date;
  format: string;
}
```

#### B. **Reportes de Cumplimiento**

```typescript
// Clase para reportes de cumplimiento
class ComplianceReporter {
  private consentManager: ConsentManager;
  private securityLogger: SecurityLogger;
  
  constructor(consentManager: ConsentManager, securityLogger: SecurityLogger) {
    this.consentManager = consentManager;
    this.securityLogger = securityLogger;
  }
  
  // Generar reporte de cumplimiento GDPR
  generateGDPRReport(timeRange: TimeRange): GDPRComplianceReport {
    const consents = this.consentManager.consents.filter(c => 
      c.createdAt >= timeRange.start && c.createdAt <= timeRange.end
    );
    
    const consentStats = this.analyzeConsentStats(consents);
    const dataProcessingStats = this.analyzeDataProcessing(timeRange);
    const securityIncidents = this.analyzeSecurityIncidents(timeRange);
    
    return {
      timeRange,
      consentStatistics: consentStats,
      dataProcessingStatistics: dataProcessingStats,
      securityIncidents,
      complianceScore: this.calculateComplianceScore(consentStats, securityIncidents),
      recommendations: this.generateRecommendations(consentStats, securityIncidents),
      generatedAt: new Date(),
    };
  }
  
  // Generar reporte de cumplimiento CCPA
  generateCCPAReport(timeRange: TimeRange): CCPAComplianceReport {
    const consents = this.consentManager.consents.filter(c => 
      c.createdAt >= timeRange.start && c.createdAt <= timeRange.end
    );
    
    const optOutRequests = consents.filter(c => c.status === 'REVOKED');
    const dataSales = this.analyzeDataSales(timeRange);
    
    return {
      timeRange,
      optOutRequests: optOutRequests.length,
      dataSales: dataSales,
      consumerRights: this.verifyConsumerRights(),
      generatedAt: new Date(),
    };
  }
  
  // Analizar estadísticas de consentimiento
  private analyzeConsentStats(consents: UserConsent[]): ConsentStatistics {
    const total = consents.length;
    const granted = consents.filter(c => c.status === 'GRANTED').length;
    const denied = consents.filter(c => c.status === 'DENIED').length;
    const revoked = consents.filter(c => c.status === 'REVOKED').length;
    
    return {
      total,
      granted,
      denied,
      revoked,
      grantRate: total > 0 ? (granted / total) * 100 : 0,
      revocationRate: total > 0 ? (revoked / total) * 100 : 0,
    };
  }
  
  // Analizar actividades de procesamiento de datos
  private analyzeDataProcessing(timeRange: TimeRange): DataProcessingStatistics {
    const activities = this.consentManager.dataProcessingActivities.filter(a => 
      a.timestamp >= timeRange.start && a.timestamp <= timeRange.end
    );
    
    const purposes = new Set(activities.map(a => a.purpose));
    const dataCategories = new Set(activities.flatMap(a => a.dataCategories));
    
    return {
      totalActivities: activities.length,
      uniquePurposes: purposes.size,
      uniqueDataCategories: dataCategories.size,
      purposes: Array.from(purposes),
      dataCategories: Array.from(dataCategories),
    };
  }
  
  // Analizar incidentes de seguridad
  private analyzeSecurityIncidents(timeRange: TimeRange): SecurityIncidentSummary {
    const logs = this.securityLogger.searchLogs({
      startDate: timeRange.start,
      endDate: timeRange.end,
      severity: 'HIGH',
    });
    
    const criticalIncidents = logs.filter(log => log.severity === 'CRITICAL');
    const highIncidents = logs.filter(log => log.severity === 'HIGH');
    
    return {
      totalIncidents: logs.length,
      criticalIncidents: criticalIncidents.length,
      highIncidents: highIncidents.length,
      incidentTypes: this.countIncidentTypes(logs),
    };
  }
  
  // Calcular puntuación de cumplimiento
  private calculateComplianceScore(
    consentStats: ConsentStatistics,
    securityIncidents: SecurityIncidentSummary
  ): number {
    let score = 100;
    
    // Reducir puntuación por incidentes de seguridad
    score -= securityIncidents.criticalIncidents * 20;
    score -= securityIncidents.highIncidents * 10;
    
    // Reducir puntuación por altas tasas de revocación
    if (consentStats.revocationRate > 50) {
      score -= 20;
    }
    
    return Math.max(0, score);
  }
  
  // Generar recomendaciones
  private generateRecommendations(
    consentStats: ConsentStatistics,
    securityIncidents: SecurityIncidentSummary
  ): string[] {
    const recommendations: string[] = [];
    
    if (securityIncidents.criticalIncidents > 0) {
      recommendations.push('Investigar y resolver incidentes críticos de seguridad inmediatamente');
    }
    
    if (consentStats.revocationRate > 30) {
      recommendations.push('Revisar políticas de consentimiento para reducir tasas de revocación');
    }
    
    if (consentStats.grantRate < 70) {
      recommendations.push('Mejorar la experiencia de consentimiento para aumentar tasas de aceptación');
    }
    
    return recommendations;
  }
  
  // Contar tipos de incidentes
  private countIncidentTypes(logs: SecurityLog[]): Record<string, number> {
    const counts: Record<string, number> = {};
    logs.forEach(log => {
      counts[log.eventType] = (counts[log.eventType] || 0) + 1;
    });
    return counts;
  }
  
  // Verificar derechos del consumidor (CCPA)
  private verifyConsumerRights(): ConsumerRightsVerification {
    return {
      rightToKnow: true,
      rightToDelete: true,
      rightToOptOut: true,
      nonDiscrimination: true,
      verification: 'VERIFIED',
    };
  }
  
  // Analizar ventas de datos (CCPA)
  private analyzeDataSales(timeRange: TimeRange): DataSalesAnalysis {
    // Implementar análisis de ventas de datos
    return {
      totalSales: 0,
      categories: [],
      thirdParties: [],
    };
  }
}

// Estructuras para reportes
interface ConsentStatistics {
  total: number;
  granted: number;
  denied: number;
  revoked: number;
  grantRate: number;
  revocationRate: number;
}

interface DataProcessingStatistics {
  totalActivities: number;
  uniquePurposes: number;
  uniqueDataCategories: number;
  purposes: string[];
  dataCategories: string[];
}

interface SecurityIncidentSummary {
  totalIncidents: number;
  criticalIncidents: number;
  highIncidents: number;
  incidentTypes: Record<string, number>;
}

interface GDPRComplianceReport {
  timeRange: TimeRange;
  consentStatistics: ConsentStatistics;
  dataProcessingStatistics: DataProcessingStatistics;
  securityIncidents: SecurityIncidentSummary;
  complianceScore: number;
  recommendations: string[];
  generatedAt: Date;
}

interface CCPAComplianceReport {
  timeRange: TimeRange;
  optOutRequests: number;
  dataSales: DataSalesAnalysis;
  consumerRights: ConsumerRightsVerification;
  generatedAt: Date;
}

interface ConsumerRightsVerification {
  rightToKnow: boolean;
  rightToDelete: boolean;
  rightToOptOut: boolean;
  nonDiscrimination: boolean;
  verification: string;
}

interface DataSalesAnalysis {
  totalSales: number;
  categories: string[];
  thirdParties: string[];
}
```

## Ejercicios Prácticos

### Ejercicio 1: Sistema de Auditoría Completo

**Objetivo**: Implementar un sistema completo de auditoría de seguridad.

```typescript
// Implementa la clase ComprehensiveSecurityAuditor
class ComprehensiveSecurityAuditor {
  private auditor: SecurityAuditor;
  private logger: SecurityLogger;
  private consentManager: ConsentManager;
  
  constructor() {
    const auditConfig: AuditConfig = {
      enabled: true,
      logLevel: 'HIGH',
      retentionPeriod: 365,
      encryptionEnabled: true,
      realTimeAlerts: true,
      complianceStandards: ['GDPR', 'CCPA', 'ISO27001'],
    };
    
    this.auditor = new SecurityAuditor(auditConfig);
    this.logger = new SecurityLogger(10000, 'audit-secret-key');
    this.consentManager = new ConsentManager();
  }
  
  // Auditar evento de seguridad
  auditSecurityEvent(event: SecurityEvent): void {
    // TODO: Implementar auditoría completa
    // Debe incluir:
    // - Registro en auditor
    // - Log de seguridad
    // - Verificación de cumplimiento
    // - Alertas si es necesario
  }
  
  // Generar reporte de auditoría
  generateAuditReport(timeRange: TimeRange): ComprehensiveAuditReport {
    // TODO: Implementar reporte completo
    // Debe incluir:
    // - Resumen de eventos
    // - Análisis de tendencias
    // - Verificación de cumplimiento
    // - Recomendaciones
  }
  
  // Verificar cumplimiento de estándares
  verifyComplianceStandards(): ComplianceVerificationResult {
    // TODO: Implementar verificación
    // Debe incluir:
    // - Verificación GDPR
    // - Verificación CCPA
    // - Verificación ISO27001
    // - Puntuación general
  }
}

// Prueba tu implementación
const testComprehensiveAuditor = () => {
  const auditor = new ComprehensiveSecurityAuditor();
  
  try {
    // Auditar evento de login
    auditor.auditSecurityEvent({
      type: 'LOGIN_SUCCESS',
      userId: 'user123',
      ipAddress: '192.168.1.1',
      timestamp: new Date(),
      details: 'User logged in successfully',
    });
    
    // Generar reporte
    const timeRange = {
      start: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000), // 30 días atrás
      end: new Date(),
    };
    
    const report = auditor.generateAuditReport(timeRange);
    console.log('Audit report:', report);
    
    // Verificar cumplimiento
    const compliance = auditor.verifyComplianceStandards();
    console.log('Compliance verification:', compliance);
    
  } catch (error) {
    console.error('Error testing comprehensive auditor:', error);
  }
};
```

### Ejercicio 2: Sistema de Cumplimiento GDPR/CCPA

**Objetivo**: Implementar un sistema completo de cumplimiento normativo.

```typescript
// Implementa la clase ComplianceManager
class ComplianceManager {
  private consentManager: ConsentManager;
  private reporter: ComplianceReporter;
  private dataInventory: DataInventory;
  
  constructor() {
    this.consentManager = new ConsentManager();
    this.reporter = new ComplianceReporter(this.consentManager, new SecurityLogger(1000, 'key'));
    this.dataInventory = new DataInventory();
  }
  
  // Gestionar solicitud de derechos del usuario
  handleUserRightsRequest(request: UserRightsRequest): UserRightsResponse {
    // TODO: Implementar manejo de solicitudes
    // Debe incluir:
    // - Verificación de identidad
    // - Procesamiento de solicitud
    // - Respuesta apropiada
    // - Registro de actividad
  }
  
  // Realizar evaluación de impacto de protección de datos
  performDPIA(dataProcessing: DataProcessingActivity): DPIAResult {
    // TODO: Implementar DPIA
    // Debe incluir:
    // - Análisis de riesgos
    // - Evaluación de impacto
    // - Medidas de mitigación
    // - Recomendaciones
  }
  
  // Generar reporte de cumplimiento
  generateComplianceReport(standard: string, timeRange: TimeRange): ComplianceReport {
    // TODO: Implementar reporte
    // Debe incluir:
    // - Análisis de cumplimiento
    // - Identificación de brechas
    // - Plan de acción
    // - Métricas de cumplimiento
  }
}

// Prueba tu implementación
const testComplianceManager = () => {
  const complianceManager = new ComplianceManager();
  
  try {
    // Manejar solicitud de derechos
    const request: UserRightsRequest = {
      userId: 'user123',
      right: 'RIGHT_TO_DELETE',
      reason: 'User requested data deletion',
      timestamp: new Date(),
    };
    
    const response = complianceManager.handleUserRightsRequest(request);
    console.log('User rights response:', response);
    
    // Realizar DPIA
    const dpiActivity: DataProcessingActivity = {
      userId: 'user123',
      activity: 'PROFILE_ANALYSIS',
      purpose: 'PERSONALIZATION',
      dataCategories: ['PROFILE_DATA', 'BEHAVIOR_DATA'],
      timestamp: new Date(),
      legalBasis: 'LEGITIMATE_INTEREST',
    };
    
    const dpiaResult = complianceManager.performDPIA(dpiActivity);
    console.log('DPIA result:', dpiaResult);
    
    // Generar reporte de cumplimiento
    const timeRange = {
      start: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000), // 90 días atrás
      end: new Date(),
    };
    
    const report = complianceManager.generateComplianceReport('GDPR', timeRange);
    console.log('Compliance report:', report);
    
  } catch (error) {
    console.error('Error testing compliance manager:', error);
  }
};
```

### Ejercicio 3: Sistema de Alertas de Seguridad

**Objetivo**: Implementar un sistema de alertas en tiempo real para incidentes de seguridad.

```typescript
// Implementa la clase SecurityAlertSystem
class SecurityAlertSystem {
  private alertRules: SecurityAlertRule[];
  private alertHistory: SecurityAlert[];
  private notificationChannels: NotificationChannel[];
  
  constructor() {
    this.alertRules = [];
    this.alertHistory = [];
    this.notificationChannels = [];
    this.initializeDefaultRules();
  }
  
  // Inicializar reglas por defecto
  private initializeDefaultRules(): void {
    // TODO: Implementar reglas por defecto
    // Debe incluir:
    // - Reglas para intentos de login fallidos
    // - Reglas para acceso no autorizado
    // - Reglas para actividad sospechosa
    // - Reglas para cambios de configuración
  }
  
  // Agregar regla de alerta personalizada
  addAlertRule(rule: SecurityAlertRule): void {
    // TODO: Implementar agregar regla
    // Debe incluir:
    // - Validación de regla
    // - Almacenamiento de regla
    // - Configuración de prioridad
  }
  
  // Evaluar evento contra reglas
  evaluateEvent(event: SecurityEvent): SecurityAlert[] {
    // TODO: Implementar evaluación
    // Debe incluir:
    // - Evaluación contra todas las reglas
    // - Generación de alertas
    // - Priorización de alertas
    // - Activación de canales de notificación
  }
  
  // Configurar canal de notificación
  addNotificationChannel(channel: NotificationChannel): void {
    // TODO: Implementar canal
    // Debe incluir:
    // - Validación de configuración
    // - Prueba de conectividad
    // - Almacenamiento de canal
  }
  
  // Obtener historial de alertas
  getAlertHistory(filters?: AlertHistoryFilters): SecurityAlert[] {
    // TODO: Implementar filtrado
    // Debe incluir:
    // - Filtrado por severidad
    // - Filtrado por tipo
    // - Filtrado por fecha
    // - Paginación de resultados
  }
}

// Prueba tu implementación
const testSecurityAlertSystem = () => {
  const alertSystem = new SecurityAlertSystem();
  
  try {
    // Agregar regla personalizada
    alertSystem.addAlertRule({
      name: 'Multiple Failed Logins',
      description: 'Alert when user has multiple failed login attempts',
      condition: (event) => event.type === 'LOGIN_FAILURE' && event.failedAttempts > 5,
      severity: 'HIGH',
      actions: ['EMAIL', 'SLACK'],
      enabled: true,
    });
    
    // Configurar canal de notificación
    alertSystem.addNotificationChannel({
      type: 'EMAIL',
      config: {
        recipients: ['security@company.com'],
        template: 'security_alert',
      },
      enabled: true,
    });
    
    // Evaluar evento
    const event: SecurityEvent = {
      type: 'LOGIN_FAILURE',
      userId: 'user123',
      ipAddress: '192.168.1.1',
      timestamp: new Date(),
      details: 'Failed login attempt',
      failedAttempts: 6,
    };
    
    const alerts = alertSystem.evaluateEvent(event);
    console.log('Generated alerts:', alerts);
    
    // Obtener historial
    const history = alertSystem.getAlertHistory({
      severity: 'HIGH',
      startDate: new Date(Date.now() - 24 * 60 * 60 * 1000), // Últimas 24 horas
    });
    
    console.log('Alert history:', history);
    
  } catch (error) {
    console.error('Error testing security alert system:', error);
  }
};
```

## Resumen de la Clase

En esta clase hemos cubierto:

✅ **Sistemas de auditoría** de seguridad con logging y monitoreo
✅ **Gestión de logs** con encriptación y análisis
✅ **Cumplimiento GDPR/CCPA** con gestión de consentimiento
✅ **Reportes de cumplimiento** con métricas y recomendaciones
✅ **Sistemas de alertas** en tiempo real para incidentes

## Próximos Pasos

Con esto hemos completado el **Módulo 14: Seguridad**. En el siguiente módulo aprenderemos sobre:
- **Módulo 15: Testing Avanzado**
- **Testing de seguridad**
- **Testing de rendimiento**
- **Testing de accesibilidad**

## Recursos Adicionales

- [GDPR Compliance Guide](https://gdpr.eu/)
- [CCPA Compliance Guide](https://oag.ca.gov/privacy/ccpa)
- [OWASP Security Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [ISO 27001 Information Security](https://www.iso.org/isoiec-27001-information-security.html)

---

**Nota**: La auditoría y el cumplimiento normativo son fundamentales para mantener la confianza de los usuarios y cumplir con las regulaciones legales. Implementa sistemas robustos de logging y monitoreo para detectar y responder a amenazas de seguridad de manera efectiva.
