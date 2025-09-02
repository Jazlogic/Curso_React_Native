# 🔒 Clase 1: GDPR y Protección de Datos

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás los fundamentos del GDPR (General Data Protection Regulation) y cómo implementar la protección de datos personales en aplicaciones React Native. Comprenderás los principios, derechos del usuario y las bases legales para el procesamiento de datos.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Comprender** los fundamentos del GDPR y su aplicación
2. **Implementar** principios de protección de datos
3. **Gestionar** derechos del usuario (ARCO)
4. **Configurar** consentimiento y bases legales
5. **Desarrollar** funcionalidades técnicas de GDPR

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 60 minutos
- **Proyecto**: 30 minutos
- **Total**: 2.25 horas

---

## 📚 Contenido Teórico

### **1. Fundamentos del GDPR**

#### **¿Qué es el GDPR?**
El Reglamento General de Protección de Datos (GDPR) es una regulación de la Unión Europea que establece reglas para el procesamiento de datos personales.

```javascript
// Estructura básica de datos personales
const personalData = {
  identificadores: {
    nombre: "Juan Pérez",
    email: "juan@email.com",
    teléfono: "+1234567890",
    dirección: "Calle 123, Ciudad"
  },
  datosSensibles: {
    biometría: "huella_digital",
    salud: "condición_médica",
    financieros: "datos_bancarios"
  },
  metadatos: {
    ip: "192.168.1.1",
    userAgent: "Mozilla/5.0...",
    timestamp: "2024-01-15T10:30:00Z"
  }
};
```

#### **Principios del GDPR**
1. **Licitud**: Procesamiento legal y transparente
2. **Limitación de la finalidad**: Propósito específico y legítimo
3. **Minimización**: Solo datos necesarios
4. **Exactitud**: Datos precisos y actualizados
5. **Limitación del almacenamiento**: Conservación limitada
6. **Integridad y confidencialidad**: Seguridad adecuada
7. **Responsabilidad proactiva**: Demostrar cumplimiento

### **2. Derechos del Usuario (ARCO)**

#### **Derechos Fundamentales**
```javascript
// Implementación de derechos del usuario
class GDPRUserRights {
  constructor(userId) {
    this.userId = userId;
    this.dataController = new DataController();
  }

  // Acceso - Derecho a conocer qué datos se procesan
  async getDataAccess() {
    const userData = await this.dataController.getUserData(this.userId);
    return {
      datosProcesados: userData,
      finalidad: "Procesamiento para servicios de la app",
      baseLegal: "Consentimiento del usuario",
      conservacion: "2 años desde último acceso"
    };
  }

  // Rectificación - Derecho a corregir datos inexactos
  async rectifyData(updates) {
    const validation = await this.validateData(updates);
    if (validation.isValid) {
      return await this.dataController.updateUserData(this.userId, updates);
    }
    throw new Error("Datos no válidos para rectificación");
  }

  // Cancelación - Derecho al olvido
  async deleteData() {
    // Eliminar datos personales
    await this.dataController.deleteUserData(this.userId);
    
    // Eliminar de sistemas de terceros
    await this.notifyThirdParties(this.userId);
    
    // Confirmar eliminación
    return { status: "deleted", timestamp: new Date() };
  }

  // Oposición - Derecho a oponerse al procesamiento
  async objectToProcessing(reason) {
    return await this.dataController.addProcessingObjection(
      this.userId, 
      reason
    );
  }
}
```

### **3. Consentimiento y Bases Legales**

#### **Tipos de Consentimiento**
```javascript
// Gestión de consentimiento GDPR
class GDPRConsentManager {
  constructor() {
    this.consentTypes = {
      NECESARIO: "necessary",
      FUNCIONAL: "functional", 
      ANALITICO: "analytics",
      PUBLICITARIO: "advertising",
      MARKETING: "marketing"
    };
  }

  // Solicitar consentimiento granular
  async requestConsent(userId, consentData) {
    const consent = {
      userId,
      timestamp: new Date(),
      version: "1.0",
      consents: {
        [this.consentTypes.NECESARIO]: {
          granted: true, // Siempre necesario
          purpose: "Funcionalidad básica de la app",
          legalBasis: "Interés legítimo"
        },
        [this.consentTypes.FUNCIONAL]: {
          granted: consentData.functional || false,
          purpose: "Mejoras de funcionalidad",
          legalBasis: "Consentimiento"
        },
        [this.consentTypes.ANALITICO]: {
          granted: consentData.analytics || false,
          purpose: "Análisis de uso y rendimiento",
          legalBasis: "Consentimiento"
        },
        [this.consentTypes.PUBLICITARIO]: {
          granted: consentData.advertising || false,
          purpose: "Publicidad personalizada",
          legalBasis: "Consentimiento"
        }
      }
    };

    await this.storeConsent(consent);
    return consent;
  }

  // Verificar consentimiento
  async checkConsent(userId, consentType) {
    const consent = await this.getConsent(userId);
    return consent?.consents[consentType]?.granted || false;
  }

  // Retirar consentimiento
  async withdrawConsent(userId, consentType) {
    const consent = await this.getConsent(userId);
    if (consent && consent.consents[consentType]) {
      consent.consents[consentType].granted = false;
      consent.consents[consentType].withdrawnAt = new Date();
      await this.storeConsent(consent);
    }
  }
}
```

### **4. Implementación Técnica**

#### **Encriptación de Datos Personales**
```javascript
// Encriptación para GDPR
import CryptoJS from 'crypto-js';

class GDPREncryption {
  constructor() {
    this.algorithm = 'AES-256-GCM';
    this.keySize = 256;
  }

  // Encriptar datos personales
  encryptPersonalData(data, key) {
    const dataString = JSON.stringify(data);
    const encrypted = CryptoJS.AES.encrypt(dataString, key).toString();
    
    return {
      encryptedData: encrypted,
      algorithm: this.algorithm,
      timestamp: new Date(),
      keyId: this.generateKeyId(key)
    };
  }

  // Desencriptar datos personales
  decryptPersonalData(encryptedData, key) {
    try {
      const decrypted = CryptoJS.AES.decrypt(encryptedData.encryptedData, key);
      const dataString = decrypted.toString(CryptoJS.enc.Utf8);
      return JSON.parse(dataString);
    } catch (error) {
      throw new Error("Error al desencriptar datos personales");
    }
  }

  // Generar clave de encriptación
  generateEncryptionKey() {
    return CryptoJS.lib.WordArray.random(256/8).toString();
  }
}
```

#### **Logging de Auditoría**
```javascript
// Sistema de auditoría para GDPR
class GDPRAuditLogger {
  constructor() {
    this.logLevels = {
      INFO: "info",
      WARNING: "warning", 
      ERROR: "error",
      CRITICAL: "critical"
    };
  }

  // Registrar acceso a datos personales
  async logDataAccess(userId, dataType, purpose, legalBasis) {
    const auditLog = {
      event: "data_access",
      userId,
      dataType,
      purpose,
      legalBasis,
      timestamp: new Date(),
      ip: await this.getClientIP(),
      userAgent: await this.getUserAgent(),
      sessionId: await this.getSessionId()
    };

    await this.storeAuditLog(auditLog);
    return auditLog;
  }

  // Registrar modificación de datos
  async logDataModification(userId, dataType, changes, reason) {
    const auditLog = {
      event: "data_modification",
      userId,
      dataType,
      changes,
      reason,
      timestamp: new Date(),
      ip: await this.getClientIP(),
      userAgent: await this.getUserAgent()
    };

    await this.storeAuditLog(auditLog);
  }

  // Registrar eliminación de datos
  async logDataDeletion(userId, dataType, reason) {
    const auditLog = {
      event: "data_deletion",
      userId,
      dataType,
      reason,
      timestamp: new Date(),
      ip: await this.getClientIP(),
      userAgent: await this.getUserAgent()
    };

    await this.storeAuditLog(auditLog);
  }
}
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Consentimiento Granular**

Crea un componente de consentimiento GDPR:

```javascript
// Componente de consentimiento GDPR
import React, { useState, useEffect } from 'react';
import { View, Text, Switch, TouchableOpacity, Alert } from 'react-native';

const GDPRConsentScreen = ({ onConsentUpdate }) => {
  const [consents, setConsents] = useState({
    necessary: true, // Siempre activo
    functional: false,
    analytics: false,
    advertising: false,
    marketing: false
  });

  const consentOptions = [
    {
      key: 'necessary',
      title: 'Funcionalidad Necesaria',
      description: 'Datos necesarios para el funcionamiento básico de la app',
      required: true
    },
    {
      key: 'functional',
      title: 'Funcionalidad Mejorada',
      description: 'Datos para mejorar la experiencia del usuario',
      required: false
    },
    {
      key: 'analytics',
      title: 'Análisis y Estadísticas',
      description: 'Datos para analizar el uso y rendimiento de la app',
      required: false
    },
    {
      key: 'advertising',
      title: 'Publicidad Personalizada',
      description: 'Datos para mostrar publicidad relevante',
      required: false
    },
    {
      key: 'marketing',
      title: 'Marketing y Comunicaciones',
      description: 'Datos para enviar ofertas y noticias',
      required: false
    }
  ];

  const handleConsentChange = (key, value) => {
    if (key === 'necessary') return; // No se puede desactivar
    
    setConsents(prev => ({
      ...prev,
      [key]: value
    }));
  };

  const handleSaveConsent = async () => {
    try {
      await onConsentUpdate(consents);
      Alert.alert('Éxito', 'Consentimiento guardado correctamente');
    } catch (error) {
      Alert.alert('Error', 'No se pudo guardar el consentimiento');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Configuración de Privacidad</Text>
      <Text style={styles.subtitle}>
        Controla cómo utilizamos tus datos personales
      </Text>

      {consentOptions.map(option => (
        <View key={option.key} style={styles.consentItem}>
          <View style={styles.consentHeader}>
            <Text style={styles.consentTitle}>{option.title}</Text>
            <Switch
              value={consents[option.key]}
              onValueChange={(value) => handleConsentChange(option.key, value)}
              disabled={option.required}
            />
          </View>
          <Text style={styles.consentDescription}>
            {option.description}
          </Text>
          {option.required && (
            <Text style={styles.requiredText}>Requerido</Text>
          )}
        </View>
      ))}

      <TouchableOpacity 
        style={styles.saveButton}
        onPress={handleSaveConsent}
      >
        <Text style={styles.saveButtonText}>Guardar Configuración</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = {
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'center'
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginBottom: 30,
    textAlign: 'center'
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
    fontSize: 18,
    fontWeight: '600',
    flex: 1
  },
  consentDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20
  },
  requiredText: {
    fontSize: 12,
    color: '#007AFF',
    fontWeight: '600',
    marginTop: 5
  },
  saveButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    marginTop: 20
  },
  saveButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600',
    textAlign: 'center'
  }
};

export default GDPRConsentScreen;
```

### **Ejercicio 2: Gestión de Derechos del Usuario**

Implementa un sistema de gestión de derechos ARCO:

```javascript
// Gestión de derechos del usuario
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, Alert, ScrollView } from 'react-native';

const UserRightsManager = ({ userId, onDataRequest }) => {
  const [loading, setLoading] = useState(false);

  const userRights = [
    {
      id: 'access',
      title: 'Derecho de Acceso',
      description: 'Obtener información sobre qué datos personales procesamos',
      action: 'Solicitar copia de datos'
    },
    {
      id: 'rectification',
      title: 'Derecho de Rectificación',
      description: 'Corregir datos personales inexactos o incompletos',
      action: 'Solicitar corrección'
    },
    {
      id: 'erasure',
      title: 'Derecho al Olvido',
      description: 'Eliminar datos personales cuando ya no sean necesarios',
      action: 'Solicitar eliminación'
    },
    {
      id: 'portability',
      title: 'Portabilidad de Datos',
      description: 'Recibir datos en formato estructurado y transferible',
      action: 'Solicitar portabilidad'
    },
    {
      id: 'objection',
      title: 'Derecho de Oposición',
      description: 'Oponerse al procesamiento de datos personales',
      action: 'Solicitar oposición'
    }
  ];

  const handleRightRequest = async (rightId) => {
    setLoading(true);
    try {
      const result = await onDataRequest(userId, rightId);
      Alert.alert(
        'Solicitud Enviada',
        'Tu solicitud ha sido procesada. Recibirás una respuesta en 30 días.'
      );
    } catch (error) {
      Alert.alert('Error', 'No se pudo procesar tu solicitud');
    } finally {
      setLoading(false);
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Tus Derechos de Privacidad</Text>
      <Text style={styles.subtitle}>
        Ejerce tus derechos bajo el GDPR
      </Text>

      {userRights.map(right => (
        <View key={right.id} style={styles.rightItem}>
          <Text style={styles.rightTitle}>{right.title}</Text>
          <Text style={styles.rightDescription}>
            {right.description}
          </Text>
          <TouchableOpacity
            style={styles.actionButton}
            onPress={() => handleRightRequest(right.id)}
            disabled={loading}
          >
            <Text style={styles.actionButtonText}>
              {right.action}
            </Text>
          </TouchableOpacity>
        </View>
      ))}

      <View style={styles.infoBox}>
        <Text style={styles.infoTitle}>Información Importante</Text>
        <Text style={styles.infoText}>
          • Las solicitudes se procesan en un plazo máximo de 30 días
        </Text>
        <Text style={styles.infoText}>
          • Puedes ejercer estos derechos de forma gratuita
        </Text>
        <Text style={styles.infoText}>
          • Contacta con nosotros si tienes dudas
        </Text>
      </View>
    </ScrollView>
  );
};

const styles = {
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'center'
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginBottom: 30,
    textAlign: 'center'
  },
  rightItem: {
    backgroundColor: 'white',
    padding: 20,
    marginBottom: 15,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  rightTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 8,
    color: '#333'
  },
  rightDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
    marginBottom: 15
  },
  actionButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
    alignItems: 'center'
  },
  actionButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600'
  },
  infoBox: {
    backgroundColor: '#E3F2FD',
    padding: 15,
    borderRadius: 10,
    marginTop: 20
  },
  infoTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 10,
    color: '#1976D2'
  },
  infoText: {
    fontSize: 14,
    color: '#1976D2',
    marginBottom: 5
  }
};

export default UserRightsManager;
```

---

## 🎯 Proyecto Práctico

### **Objetivo**
Crear un sistema completo de gestión GDPR para una aplicación de e-commerce que incluya consentimiento granular, gestión de derechos del usuario y logging de auditoría.

### **Requisitos**
1. **Pantalla de consentimiento** con opciones granulares
2. **Gestión de derechos ARCO** del usuario
3. **Sistema de auditoría** para todas las operaciones
4. **Encriptación** de datos personales
5. **Dashboard** de compliance

### **Entregables**
- Componente de consentimiento GDPR
- Gestor de derechos del usuario
- Sistema de logging de auditoría
- Configuración de encriptación
- Documentación de implementación

---

## 📚 Recursos Adicionales

### **Documentación Oficial**
- [GDPR Text](https://gdpr-info.eu/)
- [ICO GDPR Guide](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/)
- [EDPB Guidelines](https://edpb.europa.eu/our-work-tools/general-guidance/gdpr-guidelines-recommendations-best-practices_en)

### **Herramientas de Desarrollo**
- [React Native GDPR](https://github.com/react-native-gdpr)
- [Privacy by Design](https://www.privacybydesign.ca/)
- [GDPR Compliance Tools](https://gdpr.eu/compliance/)

### **Testing y Validación**
- [GDPR Compliance Checker](https://gdpr.eu/compliance/)
- [Privacy Impact Assessment](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/accountability-and-governance/data-protection-impact-assessments/)

---

## ✅ Checklist de Aprendizaje

### **Conceptos Teóricos**
- [ ] Comprender los principios del GDPR
- [ ] Identificar tipos de datos personales
- [ ] Conocer los derechos del usuario (ARCO)
- [ ] Entender las bases legales del procesamiento
- [ ] Comprender el concepto de consentimiento

### **Implementación Técnica**
- [ ] Implementar consentimiento granular
- [ ] Crear sistema de gestión de derechos
- [ ] Configurar logging de auditoría
- [ ] Implementar encriptación de datos
- [ ] Crear dashboard de compliance

### **Mejores Prácticas**
- [ ] Aplicar principio de minimización
- [ ] Implementar seguridad por diseño
- [ ] Documentar procesos de compliance
- [ ] Configurar monitoreo continuo
- [ ] Preparar para auditorías

---

**🎯 Objetivo de la Clase**: Dominar la implementación del GDPR en aplicaciones React Native, creando sistemas que respeten la privacidad del usuario y cumplan con las regulaciones europeas de protección de datos.

**💡 Consejo**: El GDPR no es solo una regulación, es una oportunidad para construir confianza con los usuarios. Implementa la privacidad como una característica, no como una limitación.

---

**🚀 ¡Comienza a implementar la protección de datos en tus aplicaciones!**
