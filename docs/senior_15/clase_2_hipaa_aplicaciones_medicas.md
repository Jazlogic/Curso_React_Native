# 🏥 Clase 2: HIPAA y Aplicaciones Médicas

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar el cumplimiento de HIPAA (Health Insurance Portability and Accountability Act) en aplicaciones React Native que manejan información médica. Comprenderás los requisitos de seguridad, privacidad y las medidas técnicas necesarias para proteger la información de salud protegida (PHI).

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Comprender** los requisitos de HIPAA para aplicaciones
2. **Identificar** información de salud protegida (PHI)
3. **Implementar** medidas de seguridad física y técnica
4. **Configurar** sistemas de auditoría y monitoreo
5. **Desarrollar** aplicaciones médicas compliant

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 60 minutos
- **Proyecto**: 30 minutos
- **Total**: 2.25 horas

---

## 📚 Contenido Teórico

### **1. Fundamentos de HIPAA**

#### **¿Qué es HIPAA?**
HIPAA es una ley estadounidense que establece estándares para la protección de información de salud protegida (PHI) y establece requisitos para entidades cubiertas y socios comerciales.

```javascript
// Estructura de PHI (Protected Health Information)
const phiData = {
  identificadoresDirectos: {
    nombre: "María García",
    fechaNacimiento: "1985-03-15",
    numeroSeguroSocial: "123-45-6789",
    direccion: "Calle Principal 123",
    telefono: "+1-555-0123",
    email: "maria@email.com"
  },
  identificadoresIndirectos: {
    codigoPostal: "12345",
    fechaAdmision: "2024-01-15",
    fechaAlta: "2024-01-20",
    edad: 39,
    genero: "F"
  },
  informacionMedica: {
    diagnostico: "Hipertensión arterial",
    medicamentos: ["Lisinopril 10mg", "Metformina 500mg"],
    alergias: ["Penicilina", "Mariscos"],
    historialMedico: "Diabetes tipo 2, Hipertensión",
    resultadosLaboratorio: {
      glucosa: "120 mg/dL",
      presionArterial: "140/90 mmHg"
    }
  }
};
```

#### **Reglas de HIPAA**
1. **Regla de Privacidad**: Protección de PHI
2. **Regla de Seguridad**: Medidas técnicas y administrativas
3. **Regla de Notificación de Violaciones**: Procedimientos de notificación
4. **Regla de Cumplimiento**: Sanciones y enforcement

### **2. Requisitos de Seguridad**

#### **Safeguards Administrativos**
```javascript
// Sistema de gestión de acceso HIPAA
class HIPAAAccessControl {
  constructor() {
    this.userRoles = {
      DOCTOR: 'doctor',
      NURSE: 'nurse',
      ADMIN: 'admin',
      PATIENT: 'patient'
    };
    
    this.accessLevels = {
      FULL: 'full',
      LIMITED: 'limited',
      READ_ONLY: 'read_only',
      NONE: 'none'
    };
  }

  // Asignar roles y permisos
  async assignUserRole(userId, role, permissions) {
    const userAccess = {
      userId,
      role,
      permissions: {
        viewPHI: permissions.viewPHI || false,
        editPHI: permissions.editPHI || false,
        deletePHI: permissions.deletePHI || false,
        exportPHI: permissions.exportPHI || false
      },
      assignedAt: new Date(),
      assignedBy: await this.getCurrentUser(),
      expiresAt: permissions.expiresAt || null
    };

    await this.storeUserAccess(userAccess);
    await this.logAccessChange(userId, 'role_assigned', userAccess);
    
    return userAccess;
  }

  // Verificar acceso a PHI
  async checkPHIAccess(userId, phiId, action) {
    const userAccess = await this.getUserAccess(userId);
    const phiData = await this.getPHIData(phiId);
    
    // Verificar si el usuario tiene acceso al paciente
    const hasPatientAccess = await this.checkPatientAccess(
      userId, 
      phiData.patientId
    );
    
    if (!hasPatientAccess) {
      await this.logAccessDenied(userId, phiId, action, 'no_patient_access');
      return false;
    }

    // Verificar permisos específicos
    const hasPermission = this.checkPermission(userAccess, action);
    if (!hasPermission) {
      await this.logAccessDenied(userId, phiId, action, 'insufficient_permissions');
      return false;
    }

    // Log de acceso exitoso
    await this.logAccessGranted(userId, phiId, action);
    return true;
  }

  // Verificar acceso mínimo necesario
  checkMinimumNecessary(userId, requestedData, purpose) {
    const userRole = this.getUserRole(userId);
    const allowedData = this.getAllowedDataForRole(userRole, purpose);
    
    return this.isSubsetOf(requestedData, allowedData);
  }
}
```

#### **Safeguards Físicos**
```javascript
// Control de acceso físico y de dispositivos
class HIPAAPhysicalSecurity {
  constructor() {
    this.deviceSecurity = new DeviceSecurityManager();
    this.locationTracking = new LocationTracking();
  }

  // Verificar seguridad del dispositivo
  async verifyDeviceSecurity(deviceId) {
    const deviceInfo = await this.deviceSecurity.getDeviceInfo(deviceId);
    
    const securityChecks = {
      isJailbroken: await this.deviceSecurity.isJailbroken(deviceId),
      hasAntivirus: await this.deviceSecurity.hasAntivirus(deviceId),
      isEncrypted: await this.deviceSecurity.isEncrypted(deviceId),
      hasScreenLock: await this.deviceSecurity.hasScreenLock(deviceId),
      isInSecureLocation: await this.locationTracking.isInSecureLocation(deviceId)
    };

    const isSecure = Object.values(securityChecks).every(check => check === true);
    
    if (!isSecure) {
      await this.logSecurityViolation(deviceId, securityChecks);
      await this.restrictDeviceAccess(deviceId);
    }

    return isSecure;
  }

  // Configurar políticas de dispositivo
  async configureDevicePolicies(deviceId) {
    const policies = {
      requireScreenLock: true,
      requireEncryption: true,
      allowScreenshots: false,
      allowCopyPaste: false,
      requireVPN: true,
      autoLockTimeout: 300000, // 5 minutos
      maxLoginAttempts: 3
    };

    await this.deviceSecurity.applyPolicies(deviceId, policies);
    await this.logPolicyApplication(deviceId, policies);
  }
}
```

### **3. Safeguards Técnicos**

#### **Encriptación de PHI**
```javascript
// Encriptación específica para HIPAA
import CryptoJS from 'crypto-js';

class HIPAAEncryption {
  constructor() {
    this.algorithm = 'AES-256-GCM';
    this.keyDerivation = 'PBKDF2';
    this.iterations = 100000;
  }

  // Encriptar PHI con múltiples capas
  async encryptPHI(phiData, userId) {
    // Generar clave específica del usuario
    const userKey = await this.generateUserKey(userId);
    
    // Encriptar datos sensibles
    const encryptedPHI = {
      identificadores: await this.encryptField(phiData.identificadores, userKey),
      informacionMedica: await this.encryptField(phiData.informacionMedica, userKey),
      metadatos: {
        encryptedAt: new Date(),
        algorithm: this.algorithm,
        keyId: userKey.id,
        version: '1.0'
      }
    };

    // Agregar checksum de integridad
    encryptedPHI.integrityHash = await this.generateIntegrityHash(encryptedPHI);
    
    return encryptedPHI;
  }

  // Desencriptar PHI con verificación de integridad
  async decryptPHI(encryptedPHI, userId) {
    // Verificar integridad
    const isValid = await this.verifyIntegrityHash(encryptedPHI);
    if (!isValid) {
      throw new Error('PHI data integrity compromised');
    }

    // Obtener clave del usuario
    const userKey = await this.getUserKey(userId, encryptedPHI.metadatos.keyId);
    
    // Desencriptar datos
    const decryptedPHI = {
      identificadores: await this.decryptField(encryptedPHI.identificadores, userKey),
      informacionMedica: await this.decryptField(encryptedPHI.informacionMedica, userKey)
    };

    // Log de desencriptación
    await this.logDecryption(userId, encryptedPHI.metadatos.encryptedAt);
    
    return decryptedPHI;
  }

  // Encriptar campo específico
  async encryptField(fieldData, key) {
    const dataString = JSON.stringify(fieldData);
    const encrypted = CryptoJS.AES.encrypt(dataString, key.value).toString();
    
    return {
      encryptedData: encrypted,
      iv: CryptoJS.lib.WordArray.random(128/8).toString(),
      tag: CryptoJS.lib.WordArray.random(128/8).toString()
    };
  }
}
```

#### **Autenticación y Autorización**
```javascript
// Sistema de autenticación HIPAA
class HIPAAAuthentication {
  constructor() {
    this.mfaProvider = new MultiFactorAuth();
    this.sessionManager = new SessionManager();
  }

  // Autenticación multifactor para PHI
  async authenticateUser(credentials) {
    // Verificación básica
    const basicAuth = await this.verifyCredentials(credentials);
    if (!basicAuth.isValid) {
      await this.logFailedLogin(credentials.username, 'invalid_credentials');
      return { success: false, reason: 'invalid_credentials' };
    }

    // Verificar si requiere MFA
    const requiresMFA = await this.checkMFARequirement(basicAuth.userId);
    if (requiresMFA) {
      const mfaResult = await this.mfaProvider.verifyMFA(basicAuth.userId);
      if (!mfaResult.success) {
        await this.logFailedLogin(credentials.username, 'mfa_failed');
        return { success: false, reason: 'mfa_failed' };
      }
    }

    // Crear sesión segura
    const session = await this.sessionManager.createSecureSession({
      userId: basicAuth.userId,
      role: basicAuth.role,
      permissions: basicAuth.permissions,
      expiresAt: new Date(Date.now() + 8 * 60 * 60 * 1000) // 8 horas
    });

    await this.logSuccessfulLogin(basicAuth.userId, session.id);
    
    return {
      success: true,
      session: session,
      requiresPasswordChange: basicAuth.requiresPasswordChange
    };
  }

  // Verificar sesión para acceso a PHI
  async verifySessionForPHI(sessionId, phiId) {
    const session = await this.sessionManager.getSession(sessionId);
    
    if (!session || session.expiresAt < new Date()) {
      await this.logSessionExpired(sessionId);
      return false;
    }

    // Verificar si la sesión está activa
    if (!session.isActive) {
      await this.logInactiveSession(sessionId);
      return false;
    }

    // Verificar acceso específico al PHI
    const hasAccess = await this.checkPHIAccess(session.userId, phiId);
    if (!hasAccess) {
      await this.logPHIAccessDenied(session.userId, phiId);
      return false;
    }

    // Actualizar última actividad
    await this.sessionManager.updateLastActivity(sessionId);
    
    return true;
  }
}
```

### **4. Auditoría y Monitoreo**

#### **Sistema de Auditoría HIPAA**
```javascript
// Sistema de auditoría completo para HIPAA
class HIPAAAuditSystem {
  constructor() {
    this.auditLogger = new AuditLogger();
    this.alertSystem = new AlertSystem();
    this.complianceMonitor = new ComplianceMonitor();
  }

  // Registrar acceso a PHI
  async logPHIAccess(userId, phiId, action, details) {
    const auditEntry = {
      event: 'phi_access',
      userId,
      phiId,
      action, // 'view', 'edit', 'delete', 'export'
      details,
      timestamp: new Date(),
      ip: await this.getClientIP(),
      userAgent: await this.getUserAgent(),
      sessionId: await this.getSessionId(),
      location: await this.getUserLocation(),
      deviceId: await this.getDeviceId()
    };

    await this.auditLogger.log(auditEntry);
    
    // Verificar si requiere alerta
    if (this.requiresAlert(auditEntry)) {
      await this.alertSystem.sendAlert(auditEntry);
    }
  }

  // Monitoreo de cumplimiento en tiempo real
  async monitorCompliance() {
    const complianceChecks = {
      unauthorizedAccess: await this.checkUnauthorizedAccess(),
      dataBreaches: await this.checkDataBreaches(),
      encryptionStatus: await this.checkEncryptionStatus(),
      accessControls: await this.checkAccessControls(),
      auditLogs: await this.checkAuditLogs()
    };

    const violations = Object.entries(complianceChecks)
      .filter(([_, status]) => status.hasViolations)
      .map(([check, status]) => ({ check, violations: status.violations }));

    if (violations.length > 0) {
      await this.handleComplianceViolations(violations);
    }

    return {
      status: violations.length === 0 ? 'compliant' : 'violations_detected',
      violations,
      timestamp: new Date()
    };
  }

  // Generar reportes de auditoría
  async generateAuditReport(startDate, endDate, filters = {}) {
    const auditData = await this.auditLogger.getAuditData(startDate, endDate, filters);
    
    const report = {
      period: { startDate, endDate },
      summary: {
        totalEvents: auditData.length,
        phiAccessEvents: auditData.filter(e => e.event === 'phi_access').length,
        securityEvents: auditData.filter(e => e.event === 'security_violation').length,
        userActivity: this.aggregateUserActivity(auditData)
      },
      details: auditData,
      compliance: await this.assessCompliance(auditData),
      recommendations: await this.generateRecommendations(auditData)
    };

    return report;
  }
}
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Pantalla de Consentimiento HIPAA**

Crea un componente de consentimiento específico para aplicaciones médicas:

```javascript
// Componente de consentimiento HIPAA
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, ScrollView, Alert } from 'react-native';

const HIPAAConsentScreen = ({ patientId, onConsentGiven }) => {
  const [consents, setConsents] = useState({
    dataCollection: false,
    dataSharing: false,
    treatment: false,
    payment: false,
    healthcareOperations: false
  });

  const consentTypes = [
    {
      key: 'dataCollection',
      title: 'Recopilación de Información',
      description: 'Recopilar y almacenar su información médica para proporcionar atención médica',
      required: true,
      legalBasis: 'Necesario para el tratamiento médico'
    },
    {
      key: 'dataSharing',
      title: 'Compartir Información',
      description: 'Compartir información con otros proveedores de atención médica involucrados en su cuidado',
      required: false,
      legalBasis: 'Consentimiento del paciente'
    },
    {
      key: 'treatment',
      title: 'Tratamiento Médico',
      description: 'Usar su información para proporcionar, coordinar y gestionar su atención médica',
      required: true,
      legalBasis: 'Necesario para el tratamiento médico'
    },
    {
      key: 'payment',
      title: 'Procesamiento de Pagos',
      description: 'Usar su información para procesar pagos y facturación de servicios médicos',
      required: true,
      legalBasis: 'Necesario para el procesamiento de pagos'
    },
    {
      key: 'healthcareOperations',
      title: 'Operaciones de Atención Médica',
      description: 'Usar su información para mejorar la calidad de la atención y operaciones del centro médico',
      required: false,
      legalBasis: 'Interés legítimo del proveedor'
    }
  ];

  const handleConsentChange = (key, value) => {
    setConsents(prev => ({
      ...prev,
      [key]: value
    }));
  };

  const handleSubmitConsent = async () => {
    // Verificar que los consentimientos requeridos estén marcados
    const requiredConsents = consentTypes.filter(c => c.required);
    const missingRequired = requiredConsents.some(c => !consents[c.key]);
    
    if (missingRequired) {
      Alert.alert(
        'Consentimiento Requerido',
        'Debe aceptar todos los consentimientos requeridos para continuar.'
      );
      return;
    }

    try {
      const consentRecord = {
        patientId,
        consents,
        timestamp: new Date(),
        version: '1.0',
        ip: await getClientIP(),
        userAgent: await getUserAgent()
      };

      await onConsentGiven(consentRecord);
      Alert.alert('Éxito', 'Consentimiento registrado correctamente');
    } catch (error) {
      Alert.alert('Error', 'No se pudo registrar el consentimiento');
    }
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Aviso de Prácticas de Privacidad</Text>
        <Text style={styles.subtitle}>
          HIPAA - Health Insurance Portability and Accountability Act
        </Text>
      </View>

      <View style={styles.consentSection}>
        <Text style={styles.sectionTitle}>Uso y Divulgación de Información Médica</Text>
        <Text style={styles.sectionDescription}>
          Su información médica puede ser utilizada y divulgada para los siguientes propósitos:
        </Text>

        {consentTypes.map(consent => (
          <View key={consent.key} style={styles.consentItem}>
            <View style={styles.consentHeader}>
              <Text style={styles.consentTitle}>{consent.title}</Text>
              <TouchableOpacity
                style={[
                  styles.checkbox,
                  consents[consent.key] && styles.checkboxChecked
                ]}
                onPress={() => handleConsentChange(consent.key, !consents[consent.key])}
              >
                {consents[consent.key] && <Text style={styles.checkmark}>✓</Text>}
              </TouchableOpacity>
            </View>
            <Text style={styles.consentDescription}>
              {consent.description}
            </Text>
            <Text style={styles.legalBasis}>
              Base Legal: {consent.legalBasis}
            </Text>
            {consent.required && (
              <Text style={styles.requiredText}>* Requerido</Text>
            )}
          </View>
        ))}
      </View>

      <View style={styles.rightsSection}>
        <Text style={styles.sectionTitle}>Sus Derechos</Text>
        <Text style={styles.rightsText}>
          • Derecho a acceder a su información médica
        </Text>
        <Text style={styles.rightsText}>
          • Derecho a solicitar correcciones
        </Text>
        <Text style={styles.rightsText}>
          • Derecho a restringir el uso y divulgación
        </Text>
        <Text style={styles.rightsText}>
          • Derecho a recibir una copia de este aviso
        </Text>
      </View>

      <TouchableOpacity 
        style={styles.submitButton}
        onPress={handleSubmitConsent}
      >
        <Text style={styles.submitButtonText}>
          Aceptar y Continuar
        </Text>
      </TouchableOpacity>
    </ScrollView>
  );
};

const styles = {
  container: {
    flex: 1,
    backgroundColor: '#f8f9fa'
  },
  header: {
    backgroundColor: '#007AFF',
    padding: 20,
    alignItems: 'center'
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    color: 'white',
    textAlign: 'center',
    marginBottom: 5
  },
  subtitle: {
    fontSize: 14,
    color: 'white',
    textAlign: 'center',
    opacity: 0.9
  },
  consentSection: {
    padding: 20
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 10,
    color: '#333'
  },
  sectionDescription: {
    fontSize: 14,
    color: '#666',
    marginBottom: 20,
    lineHeight: 20
  },
  consentItem: {
    backgroundColor: 'white',
    padding: 15,
    marginBottom: 15,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  consentHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8
  },
  consentTitle: {
    fontSize: 16,
    fontWeight: '600',
    flex: 1,
    color: '#333'
  },
  checkbox: {
    width: 24,
    height: 24,
    borderWidth: 2,
    borderColor: '#007AFF',
    borderRadius: 4,
    alignItems: 'center',
    justifyContent: 'center'
  },
  checkboxChecked: {
    backgroundColor: '#007AFF'
  },
  checkmark: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold'
  },
  consentDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
    marginBottom: 8
  },
  legalBasis: {
    fontSize: 12,
    color: '#007AFF',
    fontStyle: 'italic',
    marginBottom: 5
  },
  requiredText: {
    fontSize: 12,
    color: '#FF3B30',
    fontWeight: '600'
  },
  rightsSection: {
    padding: 20,
    backgroundColor: '#E3F2FD'
  },
  rightsText: {
    fontSize: 14,
    color: '#1976D2',
    marginBottom: 5,
    lineHeight: 20
  },
  submitButton: {
    backgroundColor: '#007AFF',
    margin: 20,
    padding: 15,
    borderRadius: 10,
    alignItems: 'center'
  },
  submitButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600'
  }
};

export default HIPAAConsentScreen;
```

### **Ejercicio 2: Dashboard de Seguridad HIPAA**

Implementa un dashboard para monitorear el cumplimiento de HIPAA:

```javascript
// Dashboard de seguridad HIPAA
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TouchableOpacity, Alert } from 'react-native';

const HIPAASecurityDashboard = ({ userId }) => {
  const [securityStatus, setSecurityStatus] = useState(null);
  const [auditLogs, setAuditLogs] = useState([]);
  const [complianceScore, setComplianceScore] = useState(0);

  useEffect(() => {
    loadSecurityData();
  }, []);

  const loadSecurityData = async () => {
    try {
      const [status, logs, score] = await Promise.all([
        getSecurityStatus(),
        getRecentAuditLogs(),
        getComplianceScore()
      ]);
      
      setSecurityStatus(status);
      setAuditLogs(logs);
      setComplianceScore(score);
    } catch (error) {
      Alert.alert('Error', 'No se pudo cargar la información de seguridad');
    }
  };

  const securityMetrics = [
    {
      title: 'Accesos a PHI',
      value: securityStatus?.phiAccesses || 0,
      status: 'normal',
      icon: '👁️'
    },
    {
      title: 'Intentos Fallidos',
      value: securityStatus?.failedAttempts || 0,
      status: securityStatus?.failedAttempts > 5 ? 'warning' : 'normal',
      icon: '⚠️'
    },
    {
      title: 'Dispositivos Activos',
      value: securityStatus?.activeDevices || 0,
      status: 'normal',
      icon: '📱'
    },
    {
      title: 'Puntuación Compliance',
      value: `${complianceScore}%`,
      status: complianceScore >= 90 ? 'good' : complianceScore >= 70 ? 'warning' : 'critical',
      icon: '📊'
    }
  ];

  const getStatusColor = (status) => {
    switch (status) {
      case 'good': return '#4CAF50';
      case 'warning': return '#FF9800';
      case 'critical': return '#F44336';
      default: return '#2196F3';
    }
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Dashboard de Seguridad HIPAA</Text>
        <Text style={styles.subtitle}>
          Monitoreo en tiempo real del cumplimiento
        </Text>
      </View>

      <View style={styles.metricsContainer}>
        {securityMetrics.map((metric, index) => (
          <View key={index} style={styles.metricCard}>
            <Text style={styles.metricIcon}>{metric.icon}</Text>
            <Text style={styles.metricValue}>{metric.value}</Text>
            <Text style={styles.metricTitle}>{metric.title}</Text>
            <View style={[
              styles.statusIndicator,
              { backgroundColor: getStatusColor(metric.status) }
            ]} />
          </View>
        ))}
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Estado de Seguridad</Text>
        <View style={styles.statusCard}>
          <Text style={styles.statusTitle}>
            {securityStatus?.overallStatus === 'secure' ? '🔒 Seguro' : '⚠️ Atención Requerida'}
          </Text>
          <Text style={styles.statusDescription}>
            {securityStatus?.overallStatus === 'secure' 
              ? 'Todos los sistemas de seguridad están funcionando correctamente'
              : 'Se detectaron problemas de seguridad que requieren atención'
            }
          </Text>
        </View>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Actividad Reciente</Text>
        {auditLogs.map((log, index) => (
          <View key={index} style={styles.logItem}>
            <View style={styles.logHeader}>
              <Text style={styles.logEvent}>{log.event}</Text>
              <Text style={styles.logTime}>
                {new Date(log.timestamp).toLocaleTimeString()}
              </Text>
            </View>
            <Text style={styles.logDetails}>
              Usuario: {log.userId} | IP: {log.ip}
            </Text>
            {log.details && (
              <Text style={styles.logDescription}>{log.details}</Text>
            )}
          </View>
        ))}
      </View>

      <View style={styles.actionsContainer}>
        <TouchableOpacity 
          style={styles.actionButton}
          onPress={() => generateSecurityReport()}
        >
          <Text style={styles.actionButtonText}>Generar Reporte</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={[styles.actionButton, styles.secondaryButton]}
          onPress={() => runSecurityScan()}
        >
          <Text style={[styles.actionButtonText, styles.secondaryButtonText]}>
            Escaneo de Seguridad
          </Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
};

const styles = {
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  header: {
    backgroundColor: '#007AFF',
    padding: 20,
    alignItems: 'center'
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 5
  },
  subtitle: {
    fontSize: 14,
    color: 'white',
    opacity: 0.9
  },
  metricsContainer: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 15,
    justifyContent: 'space-between'
  },
  metricCard: {
    backgroundColor: 'white',
    width: '48%',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  metricIcon: {
    fontSize: 24,
    marginBottom: 5
  },
  metricValue: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5
  },
  metricTitle: {
    fontSize: 12,
    color: '#666',
    textAlign: 'center'
  },
  statusIndicator: {
    width: 8,
    height: 8,
    borderRadius: 4,
    marginTop: 5
  },
  section: {
    padding: 15
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 10,
    color: '#333'
  },
  statusCard: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  statusTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 5,
    color: '#333'
  },
  statusDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20
  },
  logItem: {
    backgroundColor: 'white',
    padding: 15,
    marginBottom: 10,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  logHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 5
  },
  logEvent: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333'
  },
  logTime: {
    fontSize: 12,
    color: '#666'
  },
  logDetails: {
    fontSize: 12,
    color: '#666',
    marginBottom: 5
  },
  logDescription: {
    fontSize: 12,
    color: '#999',
    fontStyle: 'italic'
  },
  actionsContainer: {
    padding: 15,
    flexDirection: 'row',
    justifyContent: 'space-between'
  },
  actionButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
    flex: 0.48,
    alignItems: 'center'
  },
  secondaryButton: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#007AFF'
  },
  actionButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: '600'
  },
  secondaryButtonText: {
    color: '#007AFF'
  }
};

export default HIPAASecurityDashboard;
```

---

## 🎯 Proyecto Práctico

### **Objetivo**
Crear una aplicación de telemedicina que cumpla completamente con HIPAA, incluyendo consentimiento del paciente, gestión segura de PHI, auditoría completa y dashboard de monitoreo.

### **Requisitos**
1. **Sistema de consentimiento** HIPAA específico
2. **Gestión segura de PHI** con encriptación
3. **Control de acceso** basado en roles
4. **Sistema de auditoría** completo
5. **Dashboard de seguridad** en tiempo real

### **Entregables**
- Pantalla de consentimiento HIPAA
- Sistema de gestión de PHI
- Control de acceso y autenticación
- Sistema de auditoría y logging
- Dashboard de monitoreo de seguridad

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [HIPAA Regulations](https://www.hhs.gov/hipaa/for-professionals/index.html)
- [HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html)
- [HIPAA Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/index.html)

### **Herramientas de Desarrollo**
- [HIPAA Compliance Tools](https://www.hhs.gov/hipaa/for-professionals/compliance-enforcement/index.html)
- [Healthcare Security Frameworks](https://www.healthit.gov/topic/health-it-basics/health-information-privacy-security-and-your-ehr)

### **Testing y Validación**
- [HIPAA Risk Assessment](https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html)
- [Healthcare Security Testing](https://www.healthit.gov/topic/health-it-basics/health-information-privacy-security-and-your-ehr)

---

## ✅ Checklist de Aprendizaje

### **Conceptos Teóricos**
- [ ] Comprender los requisitos de HIPAA
- [ ] Identificar información de salud protegida (PHI)
- [ ] Conocer las reglas de privacidad y seguridad
- [ ] Entender los safeguards administrativos, físicos y técnicos
- [ ] Comprender los requisitos de auditoría

### **Implementación Técnica**
- [ ] Implementar consentimiento HIPAA
- [ ] Crear sistema de gestión de PHI
- [ ] Configurar control de acceso basado en roles
- [ ] Implementar encriptación de datos médicos
- [ ] Crear sistema de auditoría completo

### **Mejores Prácticas**
- [ ] Aplicar principio de mínimo acceso necesario
- [ ] Implementar autenticación multifactor
- [ ] Configurar monitoreo continuo
- [ ] Documentar todos los procesos
- [ ] Preparar para auditorías de cumplimiento

---

**🎯 Objetivo de la Clase**: Dominar la implementación de HIPAA en aplicaciones React Native médicas, creando sistemas seguros que protejan la información de salud de los pacientes y cumplan con todas las regulaciones de salud.

**💡 Consejo**: HIPAA no es solo cumplimiento, es protección de vidas. Cada medida de seguridad implementada protege información que puede salvar vidas y mejorar la atención médica.

---

**🚀 ¡Comienza a proteger la información médica con los más altos estándares de seguridad!**
