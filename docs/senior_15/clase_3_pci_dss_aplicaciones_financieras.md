# 💳 Clase 3: PCI DSS y Aplicaciones Financieras

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar el cumplimiento de PCI DSS (Payment Card Industry Data Security Standard) en aplicaciones React Native que procesan pagos con tarjetas. Comprenderás los requisitos de seguridad, encriptación y las medidas técnicas necesarias para proteger datos de tarjetas de crédito.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Comprender** los estándares PCI DSS y sus requisitos
2. **Implementar** encriptación y tokenización de datos de tarjetas
3. **Configurar** sistemas de auditoría y monitoreo
4. **Desarrollar** aplicaciones financieras seguras
5. **Cumplir** con los niveles de compliance PCI

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 60 minutos
- **Proyecto**: 30 minutos
- **Total**: 2.25 horas

---

## 📚 Contenido Teórico

### **1. Fundamentos de PCI DSS**

#### **¿Qué es PCI DSS?**
PCI DSS es un estándar de seguridad de la industria de tarjetas de pago que establece requisitos para organizaciones que procesan, almacenan o transmiten datos de tarjetas de crédito.

```javascript
// Estructura de datos de tarjeta de crédito
const cardData = {
  // Datos sensibles de autenticación (SAD)
  sensitiveAuthenticationData: {
    fullTrackData: "1234567890123456=1234567890123456789", // Prohibido almacenar
    cvv: "123", // Prohibido almacenar
    pin: "1234", // Prohibido almacenar
    magneticStripe: "raw_magnetic_stripe_data" // Prohibido almacenar
  },
  
  // Datos de la cuenta (PAD)
  primaryAccountData: {
    pan: "4111111111111111", // Número de tarjeta - requiere encriptación
    cardholderName: "Juan Pérez",
    expirationDate: "12/25",
    serviceCode: "123"
  },
  
  // Datos adicionales
  additionalData: {
    transactionId: "TXN123456789",
    merchantId: "MERCHANT123",
    terminalId: "TERMINAL456",
    timestamp: "2024-01-15T10:30:00Z"
  }
};
```

#### **Niveles de Compliance PCI**
1. **Nivel 1**: Más de 6 millones de transacciones anuales
2. **Nivel 2**: 1-6 millones de transacciones anuales
3. **Nivel 3**: 20,000-1 millón de transacciones anuales
4. **Nivel 4**: Menos de 20,000 transacciones anuales

### **2. Requisitos de Seguridad PCI DSS**

#### **12 Requisitos Principales**
```javascript
// Sistema de cumplimiento PCI DSS
class PCIDSSCompliance {
  constructor() {
    this.requirements = {
      // Construir y mantener una red segura
      requirement1: {
        title: "Firewall y Configuración de Red",
        controls: [
          "firewall_configuration",
          "default_passwords",
          "network_segmentation"
        ]
      },
      
      // Proteger datos de titulares de tarjetas
      requirement2: {
        title: "Configuración de Seguridad",
        controls: [
          "vendor_defaults",
          "security_parameters",
          "system_configuration"
        ]
      },
      
      // Mantener un programa de gestión de vulnerabilidades
      requirement3: {
        title: "Protección de Datos de Titulares",
        controls: [
          "data_protection",
          "data_encryption",
          "data_retention"
        ]
      },
      
      // Implementar medidas de control de acceso
      requirement4: {
        title: "Encriptación de Datos Transmitidos",
        controls: [
          "transmission_encryption",
          "secure_protocols",
          "key_management"
        ]
      },
      
      // Monitorear y probar redes regularmente
      requirement5: {
        title: "Protección contra Malware",
        controls: [
          "antivirus_software",
          "malware_protection",
          "regular_updates"
        ]
      },
      
      // Mantener una política de seguridad de la información
      requirement6: {
        title: "Desarrollo de Sistemas Seguros",
        controls: [
          "secure_development",
          "vulnerability_management",
          "change_management"
        ]
      },
      
      requirement7: {
        title: "Restricción de Acceso por Necesidad de Negocio",
        controls: [
          "access_control",
          "user_management",
          "privilege_management"
        ]
      },
      
      requirement8: {
        title: "Identificación y Autenticación",
        controls: [
          "user_identification",
          "authentication_controls",
          "password_management"
        ]
      },
      
      requirement9: {
        title: "Restricción del Acceso Físico",
        controls: [
          "physical_access_control",
          "media_handling",
          "device_management"
        ]
      },
      
      requirement10: {
        title: "Monitoreo y Pruebas de Redes",
        controls: [
          "network_monitoring",
          "log_management",
          "intrusion_detection"
        ]
      },
      
      requirement11: {
        title: "Pruebas de Seguridad Regulares",
        controls: [
          "vulnerability_scanning",
          "penetration_testing",
          "security_testing"
        ]
      },
      
      requirement12: {
        title: "Política de Seguridad de la Información",
        controls: [
          "security_policy",
          "incident_response",
          "security_awareness"
        ]
      }
    };
  }

  // Evaluar cumplimiento de un requisito
  async evaluateRequirement(requirementId, controls) {
    const requirement = this.requirements[requirementId];
    if (!requirement) {
      throw new Error(`Requisito ${requirementId} no encontrado`);
    }

    const evaluation = {
      requirementId,
      title: requirement.title,
      controls: {},
      complianceScore: 0,
      status: 'non_compliant'
    };

    // Evaluar cada control
    for (const control of requirement.controls) {
      const controlResult = await this.evaluateControl(control, controls[control]);
      evaluation.controls[control] = controlResult;
    }

    // Calcular puntuación de cumplimiento
    const totalControls = Object.keys(evaluation.controls).length;
    const compliantControls = Object.values(evaluation.controls)
      .filter(result => result.status === 'compliant').length;
    
    evaluation.complianceScore = (compliantControls / totalControls) * 100;
    evaluation.status = evaluation.complianceScore >= 100 ? 'compliant' : 'non_compliant';

    return evaluation;
  }

  // Evaluar control específico
  async evaluateControl(controlId, controlData) {
    const controlEvaluators = {
      data_encryption: () => this.evaluateDataEncryption(controlData),
      access_control: () => this.evaluateAccessControl(controlData),
      network_monitoring: () => this.evaluateNetworkMonitoring(controlData),
      vulnerability_scanning: () => this.evaluateVulnerabilityScanning(controlData)
    };

    const evaluator = controlEvaluators[controlId];
    if (!evaluator) {
      return {
        status: 'not_evaluated',
        message: 'Evaluador no disponible para este control'
      };
    }

    return await evaluator();
  }
}
```

### **3. Encriptación y Tokenización**

#### **Encriptación de Datos de Tarjetas**
```javascript
// Sistema de encriptación PCI DSS
import CryptoJS from 'crypto-js';

class PCIDSSEncryption {
  constructor() {
    this.algorithm = 'AES-256-GCM';
    this.keySize = 256;
    this.ivSize = 128;
  }

  // Encriptar PAN (Primary Account Number)
  async encryptPAN(pan, keyId) {
    // Validar formato de PAN
    if (!this.isValidPAN(pan)) {
      throw new Error('PAN inválido');
    }

    // Generar IV único
    const iv = CryptoJS.lib.WordArray.random(this.ivSize / 8);
    
    // Encriptar PAN
    const encrypted = CryptoJS.AES.encrypt(pan, keyId, {
      iv: iv,
      mode: CryptoJS.mode.GCM,
      padding: CryptoJS.pad.NoPadding
    });

    return {
      encryptedPAN: encrypted.toString(),
      iv: iv.toString(),
      keyId: keyId,
      algorithm: this.algorithm,
      timestamp: new Date(),
      hash: this.generateHash(pan)
    };
  }

  // Tokenizar PAN para almacenamiento
  async tokenizePAN(pan, tokenId) {
    // Generar token seguro
    const token = await this.generateSecureToken(pan, tokenId);
    
    // Almacenar mapeo token-PAN de forma segura
    await this.storeTokenMapping(token, pan);
    
    return {
      token: token,
      tokenId: tokenId,
      maskedPAN: this.maskPAN(pan),
      createdAt: new Date(),
      expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000) // 1 año
    };
  }

  // Desencriptar PAN
  async decryptPAN(encryptedData, keyId) {
    try {
      const decrypted = CryptoJS.AES.decrypt(
        encryptedData.encryptedPAN, 
        keyId, 
        {
          iv: CryptoJS.enc.Hex.parse(encryptedData.iv),
          mode: CryptoJS.mode.GCM,
          padding: CryptoJS.pad.NoPadding
        }
      );

      const pan = decrypted.toString(CryptoJS.enc.Utf8);
      
      // Verificar integridad
      if (this.generateHash(pan) !== encryptedData.hash) {
        throw new Error('Integridad de datos comprometida');
      }

      return pan;
    } catch (error) {
      throw new Error('Error al desencriptar PAN');
    }
  }

  // Validar formato de PAN
  isValidPAN(pan) {
    // Remover espacios y guiones
    const cleanPAN = pan.replace(/[\s-]/g, '');
    
    // Verificar longitud (13-19 dígitos)
    if (cleanPAN.length < 13 || cleanPAN.length > 19) {
      return false;
    }

    // Verificar que solo contenga dígitos
    if (!/^\d+$/.test(cleanPAN)) {
      return false;
    }

    // Aplicar algoritmo de Luhn
    return this.luhnCheck(cleanPAN);
  }

  // Algoritmo de Luhn para validación
  luhnCheck(pan) {
    let sum = 0;
    let isEven = false;

    for (let i = pan.length - 1; i >= 0; i--) {
      let digit = parseInt(pan[i]);

      if (isEven) {
        digit *= 2;
        if (digit > 9) {
          digit -= 9;
        }
      }

      sum += digit;
      isEven = !isEven;
    }

    return sum % 10 === 0;
  }

  // Enmascarar PAN para display
  maskPAN(pan) {
    if (pan.length < 8) return pan;
    
    const firstFour = pan.substring(0, 4);
    const lastFour = pan.substring(pan.length - 4);
    const middle = '*'.repeat(pan.length - 8);
    
    return `${firstFour}${middle}${lastFour}`;
  }
}
```

#### **Tokenización Segura**
```javascript
// Sistema de tokenización PCI DSS
class PCIDSSTokenization {
  constructor() {
    this.tokenVault = new SecureTokenVault();
    this.encryptionService = new PCIDSSEncryption();
  }

  // Crear token para PAN
  async createToken(pan, metadata = {}) {
    // Validar PAN
    if (!this.isValidPAN(pan)) {
      throw new Error('PAN inválido para tokenización');
    }

    // Generar token único
    const token = await this.generateUniqueToken();
    
    // Encriptar PAN para almacenamiento
    const encryptedPAN = await this.encryptionService.encryptPAN(pan);
    
    // Almacenar en vault seguro
    const tokenRecord = {
      token: token,
      encryptedPAN: encryptedPAN,
      metadata: {
        ...metadata,
        createdAt: new Date(),
        lastUsed: new Date(),
        usageCount: 0
      },
      status: 'active'
    };

    await this.tokenVault.store(tokenRecord);
    
    return {
      token: token,
      maskedPAN: this.maskPAN(pan),
      expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000)
    };
  }

  // Detokenizar (obtener PAN del token)
  async detokenize(token, requesterId) {
    // Verificar permisos de detokenización
    if (!await this.hasDetokenizationPermission(requesterId)) {
      throw new Error('Sin permisos para detokenización');
    }

    // Obtener registro del token
    const tokenRecord = await this.tokenVault.retrieve(token);
    if (!tokenRecord) {
      throw new Error('Token no encontrado');
    }

    // Verificar estado del token
    if (tokenRecord.status !== 'active') {
      throw new Error('Token inactivo o expirado');
    }

    // Desencriptar PAN
    const pan = await this.encryptionService.decryptPAN(tokenRecord.encryptedPAN);
    
    // Actualizar metadatos de uso
    await this.updateTokenUsage(token, requesterId);
    
    // Log de detokenización
    await this.logDetokenization(token, requesterId);
    
    return pan;
  }

  // Generar token único y seguro
  async generateUniqueToken() {
    let token;
    let isUnique = false;
    
    while (!isUnique) {
      // Generar token de 16 caracteres alfanuméricos
      token = this.generateRandomString(16);
      isUnique = await this.tokenVault.isTokenUnique(token);
    }
    
    return token;
  }

  // Generar string aleatorio
  generateRandomString(length) {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let result = '';
    
    for (let i = 0; i < length; i++) {
      result += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    
    return result;
  }
}
```

### **4. Auditoría y Monitoreo PCI**

#### **Sistema de Auditoría PCI DSS**
```javascript
// Sistema de auditoría para PCI DSS
class PCIDSSAuditSystem {
  constructor() {
    this.auditLogger = new AuditLogger();
    this.alertSystem = new AlertSystem();
    this.complianceMonitor = new ComplianceMonitor();
  }

  // Registrar acceso a datos de tarjeta
  async logCardDataAccess(userId, token, action, details) {
    const auditEntry = {
      event: 'card_data_access',
      userId,
      token: this.maskToken(token),
      action, // 'view', 'process', 'tokenize', 'detokenize'
      details,
      timestamp: new Date(),
      ip: await this.getClientIP(),
      userAgent: await this.getUserAgent(),
      sessionId: await this.getSessionId(),
      complianceLevel: 'PCI_DSS'
    };

    await this.auditLogger.log(auditEntry);
    
    // Verificar si requiere alerta
    if (this.requiresAlert(auditEntry)) {
      await this.alertSystem.sendAlert(auditEntry);
    }
  }

  // Monitoreo de cumplimiento PCI
  async monitorPCICompliance() {
    const complianceChecks = {
      dataEncryption: await this.checkDataEncryption(),
      accessControls: await this.checkAccessControls(),
      networkSecurity: await this.checkNetworkSecurity(),
      vulnerabilityManagement: await this.checkVulnerabilityManagement(),
      auditLogs: await this.checkAuditLogs(),
      securityTesting: await this.checkSecurityTesting()
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
      complianceScore: this.calculateComplianceScore(complianceChecks),
      timestamp: new Date()
    };
  }

  // Verificar encriptación de datos
  async checkDataEncryption() {
    const checks = {
      panEncryption: await this.verifyPANEncryption(),
      transmissionEncryption: await this.verifyTransmissionEncryption(),
      keyManagement: await this.verifyKeyManagement(),
      tokenization: await this.verifyTokenization()
    };

    const violations = Object.entries(checks)
      .filter(([_, status]) => !status.isCompliant)
      .map(([check, status]) => ({ check, issue: status.issue }));

    return {
      hasViolations: violations.length > 0,
      violations,
      overallStatus: violations.length === 0 ? 'compliant' : 'non_compliant'
    };
  }

  // Generar reporte de cumplimiento PCI
  async generatePCIReport(startDate, endDate) {
    const reportData = await this.auditLogger.getAuditData(startDate, endDate);
    
    const report = {
      period: { startDate, endDate },
      summary: {
        totalTransactions: reportData.filter(e => e.event === 'transaction').length,
        cardDataAccess: reportData.filter(e => e.event === 'card_data_access').length,
        securityEvents: reportData.filter(e => e.event === 'security_violation').length,
        complianceScore: this.calculateComplianceScore(reportData)
      },
      requirements: await this.evaluatePCIRequirements(reportData),
      recommendations: await this.generateRecommendations(reportData),
      details: reportData
    };

    return report;
  }
}
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Componente de Pago Seguro**

Crea un componente de pago que cumpla con PCI DSS:

```javascript
// Componente de pago PCI DSS compliant
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Alert } from 'react-native';

const SecurePaymentForm = ({ onPaymentProcessed }) => {
  const [cardData, setCardData] = useState({
    number: '',
    expiry: '',
    cvv: '',
    name: ''
  });
  const [isProcessing, setIsProcessing] = useState(false);

  const handleCardNumberChange = (text) => {
    // Formatear número de tarjeta
    const formatted = text.replace(/\D/g, '').replace(/(\d{4})(?=\d)/g, '$1 ');
    setCardData(prev => ({ ...prev, number: formatted }));
  };

  const handleExpiryChange = (text) => {
    // Formatear fecha de expiración
    const formatted = text.replace(/\D/g, '').replace(/(\d{2})(?=\d)/g, '$1/');
    setCardData(prev => ({ ...prev, expiry: formatted }));
  };

  const handleCVVChange = (text) => {
    // Solo permitir 3-4 dígitos
    const formatted = text.replace(/\D/g, '').substring(0, 4);
    setCardData(prev => ({ ...prev, cvv: formatted }));
  };

  const validateCardData = () => {
    const { number, expiry, cvv, name } = cardData;
    
    // Validar número de tarjeta
    const cleanNumber = number.replace(/\s/g, '');
    if (!this.luhnCheck(cleanNumber)) {
      return { isValid: false, error: 'Número de tarjeta inválido' };
    }

    // Validar fecha de expiración
    const [month, year] = expiry.split('/');
    if (!month || !year || month.length !== 2 || year.length !== 2) {
      return { isValid: false, error: 'Fecha de expiración inválida' };
    }

    const expDate = new Date(2000 + parseInt(year), parseInt(month) - 1);
    if (expDate < new Date()) {
      return { isValid: false, error: 'Tarjeta expirada' };
    }

    // Validar CVV
    if (cvv.length < 3) {
      return { isValid: false, error: 'CVV inválido' };
    }

    // Validar nombre
    if (name.trim().length < 2) {
      return { isValid: false, error: 'Nombre del titular requerido' };
    }

    return { isValid: true };
  };

  const handlePayment = async () => {
    const validation = validateCardData();
    if (!validation.isValid) {
      Alert.alert('Error', validation.error);
      return;
    }

    setIsProcessing(true);
    
    try {
      // Tokenizar datos de tarjeta
      const tokenizedCard = await tokenizeCardData(cardData);
      
      // Procesar pago con token
      const paymentResult = await processPayment(tokenizedCard);
      
      // Log de transacción
      await logPaymentTransaction(paymentResult);
      
      onPaymentProcessed(paymentResult);
      
    } catch (error) {
      Alert.alert('Error', 'No se pudo procesar el pago');
    } finally {
      setIsProcessing(false);
    }
  };

  const tokenizeCardData = async (cardData) => {
    // Enviar datos a servicio de tokenización
    const response = await fetch('/api/tokenize', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${await getAuthToken()}`
      },
      body: JSON.stringify({
        pan: cardData.number.replace(/\s/g, ''),
        expiry: cardData.expiry,
        cvv: cardData.cvv,
        name: cardData.name
      })
    });

    if (!response.ok) {
      throw new Error('Error en tokenización');
    }

    return await response.json();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Información de Pago</Text>
      
      <View style={styles.inputGroup}>
        <Text style={styles.label}>Número de Tarjeta</Text>
        <TextInput
          style={styles.input}
          value={cardData.number}
          onChangeText={handleCardNumberChange}
          placeholder="1234 5678 9012 3456"
          keyboardType="numeric"
          maxLength={19}
        />
      </View>

      <View style={styles.row}>
        <View style={[styles.inputGroup, styles.halfWidth]}>
          <Text style={styles.label}>Fecha de Expiración</Text>
          <TextInput
            style={styles.input}
            value={cardData.expiry}
            onChangeText={handleExpiryChange}
            placeholder="MM/YY"
            keyboardType="numeric"
            maxLength={5}
          />
        </View>

        <View style={[styles.inputGroup, styles.halfWidth]}>
          <Text style={styles.label}>CVV</Text>
          <TextInput
            style={styles.input}
            value={cardData.cvv}
            onChangeText={handleCVVChange}
            placeholder="123"
            keyboardType="numeric"
            maxLength={4}
            secureTextEntry
          />
        </View>
      </View>

      <View style={styles.inputGroup}>
        <Text style={styles.label}>Nombre del Titular</Text>
        <TextInput
          style={styles.input}
          value={cardData.name}
          onChangeText={(text) => setCardData(prev => ({ ...prev, name: text }))}
          placeholder="Juan Pérez"
          autoCapitalize="words"
        />
      </View>

      <TouchableOpacity 
        style={[styles.payButton, isProcessing && styles.payButtonDisabled]}
        onPress={handlePayment}
        disabled={isProcessing}
      >
        <Text style={styles.payButtonText}>
          {isProcessing ? 'Procesando...' : 'Pagar Ahora'}
        </Text>
      </TouchableOpacity>

      <View style={styles.securityInfo}>
        <Text style={styles.securityText}>
          🔒 Tus datos están protegidos con encriptación de nivel bancario
        </Text>
        <Text style={styles.securityText}>
          ✅ Cumplimos con los estándares PCI DSS
        </Text>
      </View>
    </View>
  );
};

const styles = {
  container: {
    padding: 20,
    backgroundColor: '#f8f9fa'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
    color: '#333'
  },
  inputGroup: {
    marginBottom: 15
  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between'
  },
  halfWidth: {
    width: '48%'
  },
  label: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 8,
    color: '#333'
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    backgroundColor: 'white'
  },
  payButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginTop: 20
  },
  payButtonDisabled: {
    backgroundColor: '#ccc'
  },
  payButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600'
  },
  securityInfo: {
    marginTop: 20,
    padding: 15,
    backgroundColor: '#E8F5E8',
    borderRadius: 10
  },
  securityText: {
    fontSize: 14,
    color: '#2E7D32',
    marginBottom: 5
  }
};

export default SecurePaymentForm;
```

### **Ejercicio 2: Dashboard de Cumplimiento PCI**

Implementa un dashboard para monitorear el cumplimiento de PCI DSS:

```javascript
// Dashboard de cumplimiento PCI DSS
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TouchableOpacity, Alert } from 'react-native';

const PCIDSSDashboard = ({ merchantId }) => {
  const [complianceData, setComplianceData] = useState(null);
  const [securityMetrics, setSecurityMetrics] = useState({});
  const [recentTransactions, setRecentTransactions] = useState([]);

  useEffect(() => {
    loadComplianceData();
  }, []);

  const loadComplianceData = async () => {
    try {
      const [compliance, metrics, transactions] = await Promise.all([
        getPCIComplianceStatus(merchantId),
        getSecurityMetrics(merchantId),
        getRecentTransactions(merchantId)
      ]);
      
      setComplianceData(compliance);
      setSecurityMetrics(metrics);
      setRecentTransactions(transactions);
    } catch (error) {
      Alert.alert('Error', 'No se pudo cargar la información de cumplimiento');
    }
  };

  const pciRequirements = [
    {
      id: 1,
      title: 'Firewall y Configuración de Red',
      status: complianceData?.requirements?.requirement1?.status || 'unknown',
      score: complianceData?.requirements?.requirement1?.score || 0
    },
    {
      id: 2,
      title: 'Configuración de Seguridad',
      status: complianceData?.requirements?.requirement2?.status || 'unknown',
      score: complianceData?.requirements?.requirement2?.score || 0
    },
    {
      id: 3,
      title: 'Protección de Datos de Titulares',
      status: complianceData?.requirements?.requirement3?.status || 'unknown',
      score: complianceData?.requirements?.requirement3?.score || 0
    },
    {
      id: 4,
      title: 'Encriptación de Datos Transmitidos',
      status: complianceData?.requirements?.requirement4?.status || 'unknown',
      score: complianceData?.requirements?.requirement4?.score || 0
    },
    {
      id: 5,
      title: 'Protección contra Malware',
      status: complianceData?.requirements?.requirement5?.status || 'unknown',
      score: complianceData?.requirements?.requirement5?.score || 0
    },
    {
      id: 6,
      title: 'Desarrollo de Sistemas Seguros',
      status: complianceData?.requirements?.requirement6?.status || 'unknown',
      score: complianceData?.requirements?.requirement6?.score || 0
    }
  ];

  const getStatusColor = (status) => {
    switch (status) {
      case 'compliant': return '#4CAF50';
      case 'non_compliant': return '#F44336';
      case 'partial': return '#FF9800';
      default: return '#9E9E9E';
    }
  };

  const getStatusIcon = (status) => {
    switch (status) {
      case 'compliant': return '✅';
      case 'non_compliant': return '❌';
      case 'partial': return '⚠️';
      default: return '❓';
    }
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Dashboard PCI DSS</Text>
        <Text style={styles.subtitle}>
          Cumplimiento: {complianceData?.overallScore || 0}%
        </Text>
      </View>

      <View style={styles.overviewCard}>
        <Text style={styles.overviewTitle}>Resumen de Cumplimiento</Text>
        <View style={styles.overviewMetrics}>
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {complianceData?.compliantRequirements || 0}
            </Text>
            <Text style={styles.metricLabel}>Requisitos Cumplidos</Text>
          </View>
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {complianceData?.totalRequirements || 12}
            </Text>
            <Text style={styles.metricLabel}>Total Requisitos</Text>
          </View>
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {complianceData?.violations || 0}
            </Text>
            <Text style={styles.metricLabel}>Violaciones</Text>
          </View>
        </View>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Requisitos PCI DSS</Text>
        {pciRequirements.map(requirement => (
          <View key={requirement.id} style={styles.requirementCard}>
            <View style={styles.requirementHeader}>
              <Text style={styles.requirementNumber}>
                {requirement.id}
              </Text>
              <View style={styles.requirementInfo}>
                <Text style={styles.requirementTitle}>
                  {requirement.title}
                </Text>
                <View style={styles.requirementStatus}>
                  <Text style={styles.statusIcon}>
                    {getStatusIcon(requirement.status)}
                  </Text>
                  <Text style={[
                    styles.statusText,
                    { color: getStatusColor(requirement.status) }
                  ]}>
                    {requirement.status.toUpperCase()}
                  </Text>
                  <Text style={styles.scoreText}>
                    {requirement.score}%
                  </Text>
                </View>
              </View>
            </View>
            <View style={styles.progressBar}>
              <View style={[
                styles.progressFill,
                { 
                  width: `${requirement.score}%`,
                  backgroundColor: getStatusColor(requirement.status)
                }
              ]} />
            </View>
          </View>
        ))}
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Métricas de Seguridad</Text>
        <View style={styles.metricsGrid}>
          <View style={styles.metricCard}>
            <Text style={styles.metricIcon}>🔒</Text>
            <Text style={styles.metricValue}>
              {securityMetrics.encryptedTransactions || 0}
            </Text>
            <Text style={styles.metricLabel}>Transacciones Encriptadas</Text>
          </View>
          <View style={styles.metricCard}>
            <Text style={styles.metricIcon}>🛡️</Text>
            <Text style={styles.metricValue}>
              {securityMetrics.failedAttempts || 0}
            </Text>
            <Text style={styles.metricLabel}>Intentos Fallidos</Text>
          </View>
          <View style={styles.metricCard}>
            <Text style={styles.metricIcon}>📊</Text>
            <Text style={styles.metricValue}>
              {securityMetrics.tokenizedCards || 0}
            </Text>
            <Text style={styles.metricLabel}>Tarjetas Tokenizadas</Text>
          </View>
          <View style={styles.metricCard}>
            <Text style={styles.metricIcon}>🔍</Text>
            <Text style={styles.metricValue}>
              {securityMetrics.securityScans || 0}
            </Text>
            <Text style={styles.metricLabel}>Escaneos de Seguridad</Text>
          </View>
        </View>
      </View>

      <View style={styles.actionsContainer}>
        <TouchableOpacity 
          style={styles.actionButton}
          onPress={() => runComplianceScan()}
        >
          <Text style={styles.actionButtonText}>Ejecutar Escaneo</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={[styles.actionButton, styles.secondaryButton]}
          onPress={() => generateComplianceReport()}
        >
          <Text style={[styles.actionButtonText, styles.secondaryButtonText]}>
            Generar Reporte
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
  overviewCard: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  overviewTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 15,
    color: '#333'
  },
  overviewMetrics: {
    flexDirection: 'row',
    justifyContent: 'space-around'
  },
  metric: {
    alignItems: 'center'
  },
  metricValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#007AFF'
  },
  metricLabel: {
    fontSize: 12,
    color: '#666',
    textAlign: 'center'
  },
  section: {
    padding: 15
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 15,
    color: '#333'
  },
  requirementCard: {
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
  requirementHeader: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 10
  },
  requirementNumber: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#007AFF',
    marginRight: 15,
    minWidth: 30
  },
  requirementInfo: {
    flex: 1
  },
  requirementTitle: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
    marginBottom: 5
  },
  requirementStatus: {
    flexDirection: 'row',
    alignItems: 'center'
  },
  statusIcon: {
    fontSize: 16,
    marginRight: 5
  },
  statusText: {
    fontSize: 12,
    fontWeight: '600',
    marginRight: 10
  },
  scoreText: {
    fontSize: 12,
    color: '#666'
  },
  progressBar: {
    height: 4,
    backgroundColor: '#E0E0E0',
    borderRadius: 2,
    overflow: 'hidden'
  },
  progressFill: {
    height: '100%',
    borderRadius: 2
  },
  metricsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
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

export default PCIDSSDashboard;
```

---

## 🎯 Proyecto Práctico

### **Objetivo**
Crear una aplicación de e-commerce que cumpla completamente con PCI DSS, incluyendo procesamiento seguro de pagos, tokenización de datos de tarjetas, encriptación de datos y monitoreo de cumplimiento.

### **Requisitos**
1. **Formulario de pago seguro** con validación PCI
2. **Sistema de tokenización** para datos de tarjetas
3. **Encriptación de datos** en tránsito y reposo
4. **Sistema de auditoría** completo
5. **Dashboard de cumplimiento** PCI DSS

### **Entregables**
- Componente de pago seguro
- Sistema de tokenización
- Encriptación de datos de tarjetas
- Sistema de auditoría y logging
- Dashboard de monitoreo de cumplimiento

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [PCI DSS Standards](https://www.pcisecuritystandards.org/document_library/)
- [PCI DSS Requirements](https://www.pcisecuritystandards.org/pci_security/maintaining_payment_security)
- [PCI DSS Testing Procedures](https://www.pcisecuritystandards.org/pci_security/testing_procedures)

### **Herramientas de Desarrollo**
- [PCI DSS Compliance Tools](https://www.pcisecuritystandards.org/pci_security/compliance_tools)
- [Payment Security Resources](https://www.pcisecuritystandards.org/pci_security/payment_security_resources)

### **Testing y Validación**
- [PCI DSS Self-Assessment](https://www.pcisecuritystandards.org/pci_security/self_assessment_questionnaire)
- [Payment Security Testing](https://www.pcisecuritystandards.org/pci_security/testing_procedures)

---

## ✅ Checklist de Aprendizaje

### **Conceptos Teóricos**
- [ ] Comprender los estándares PCI DSS
- [ ] Conocer los 12 requisitos principales
- [ ] Identificar datos sensibles de autenticación (SAD)
- [ ] Entender la diferencia entre encriptación y tokenización
- [ ] Comprender los niveles de compliance PCI

### **Implementación Técnica**
- [ ] Implementar encriptación de datos de tarjetas
- [ ] Crear sistema de tokenización seguro
- [ ] Configurar validación de tarjetas (Luhn)
- [ ] Implementar logging de auditoría PCI
- [ ] Crear dashboard de cumplimiento

### **Mejores Prácticas**
- [ ] Nunca almacenar datos sensibles de autenticación
- [ ] Implementar encriptación de extremo a extremo
- [ ] Usar tokenización para datos de tarjetas
- [ ] Configurar monitoreo continuo
- [ ] Realizar testing de seguridad regular

---

**🎯 Objetivo de la Clase**: Dominar la implementación de PCI DSS en aplicaciones React Native financieras, creando sistemas seguros que protejan los datos de tarjetas de crédito y cumplan con los más altos estándares de seguridad de pagos.

**💡 Consejo**: PCI DSS no es solo cumplimiento, es protección de confianza. Cada medida de seguridad implementada protege tanto a los comerciantes como a los consumidores de fraudes y violaciones de datos.

---

**🚀 ¡Comienza a procesar pagos de forma segura y compliant!**
