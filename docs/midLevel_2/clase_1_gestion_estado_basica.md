# 📚 Clase 1: Gestión de Estado Básica

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Módulo 3: Navegación y Routing](../midLevel_1/README.md)
- **➡️ Siguiente**: [Clase 2: Context API](clase_2_context_api.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender los conceptos fundamentales de estado en React Native
- Aprender a usar useState y useEffect para gestión básica de estado
- Dominar el patrón de elevación de estado
- Implementar gestión de estado local en componentes
- Crear aplicaciones con estado simple y eficiente

---

## 📚 Contenido Teórico

### **¿Qué es el Estado en React Native?**

El estado en React Native es la información que puede cambiar durante la vida útil de un componente. Es la base para crear aplicaciones interactivas y dinámicas que responden a las acciones del usuario.

#### **Características principales:**
- **Mutable**: Puede cambiar durante la ejecución
- **Local**: Pertenece a un componente específico
- **Reactivo**: Los cambios desencadenan re-renderizados
- **Persistente**: Mantiene información entre renderizados
- **Estructurado**: Puede ser cualquier tipo de dato

### **Tipos de Estado:**

#### **1. Estado Local:**
- Pertenece a un solo componente
- Se gestiona con `useState`
- Cambios solo afectan al componente
- Ideal para formularios y UI local

#### **2. Estado Compartido:**
- Compartido entre múltiples componentes
- Requiere elevación de estado
- Se pasa como props
- Útil para datos comunes

#### **3. Estado Global:**
- Accesible en toda la aplicación
- Requiere Context API o Redux
- Ideal para configuración y datos de usuario
- Complejo de gestionar

### **Ventajas de la Gestión de Estado:**

✅ **Reactividad**: UI se actualiza automáticamente
✅ **Predictibilidad**: Flujo de datos unidireccional
✅ **Debugging**: Fácil rastrear cambios de estado
✅ **Performance**: Re-renderizados optimizados
✅ **Mantenibilidad**: Código organizado y claro

---

## 💻 Implementación Práctica

### **1. Gestión de Estado Local con useState**

```javascript:src/components/LocalStateExample.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ScrollView,
} from 'react-native';

// Componente que demuestra gestión de estado local
const LocalStateExample = () => {
  // Estado para el formulario de usuario
  const [userForm, setUserForm] = useState({
    name: '', // Nombre del usuario
    email: '', // Email del usuario
    age: '', // Edad del usuario
    phone: '', // Teléfono del usuario
  });
  
  // Estado para validación de campos
  const [errors, setErrors] = useState({});
  
  // Estado para mostrar/ocultar contraseña
  const [showPassword, setShowPassword] = useState(false);
  
  // Estado para el modo de edición
  const [isEditing, setIsEditing] = useState(false);
  
  // Estado para el contador de cambios
  const [changeCount, setChangeCount] = useState(0);
  
  // Estado para el historial de cambios
  const [changeHistory, setChangeHistory] = useState([]);
  
  // Efecto para validar campos cuando cambian
  useEffect(() => {
    // Validar nombre (mínimo 2 caracteres)
    if (userForm.name.length > 0 && userForm.name.length < 2) {
      setErrors(prev => ({
        ...prev,
        name: 'El nombre debe tener al menos 2 caracteres'
      }));
    } else {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors.name;
        return newErrors;
      });
    }
    
    // Validar email (formato básico)
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (userForm.email.length > 0 && !emailRegex.test(userForm.email)) {
      setErrors(prev => ({
        ...prev,
        email: 'Ingrese un email válido'
      }));
    } else {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors.email;
        return newErrors;
      });
    }
    
    // Validar edad (número entre 1 y 120)
    if (userForm.age.length > 0) {
      const age = parseInt(userForm.age);
      if (isNaN(age) || age < 1 || age > 120) {
        setErrors(prev => ({
          ...prev,
          age: 'La edad debe ser un número entre 1 y 120'
        }));
      } else {
        setErrors(prev => {
          const newErrors = { ...prev };
          delete newErrors.age;
          return newErrors;
        });
      }
    }
    
    // Validar teléfono (mínimo 10 dígitos)
    if (userForm.phone.length > 0 && userForm.phone.length < 10) {
      setErrors(prev => ({
        ...prev,
        phone: 'El teléfono debe tener al menos 10 dígitos'
      }));
    } else {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors.phone;
        return newErrors;
      });
    }
  }, [userForm]); // Se ejecuta cada vez que cambia userForm
  
  // Función para actualizar un campo específico
  const updateField = (field, value) => {
    // Actualizar el formulario
    setUserForm(prev => ({
      ...prev,
      [field]: value
    }));
    
    // Incrementar contador de cambios
    setChangeCount(prev => prev + 1);
    
    // Agregar al historial de cambios
    setChangeHistory(prev => [
      ...prev,
      {
        field,
        oldValue: userForm[field],
        newValue: value,
        timestamp: new Date().toLocaleTimeString()
      }
    ]);
  };
  
  // Función para limpiar el formulario
  const clearForm = () => {
    setUserForm({
      name: '',
      email: '',
      age: '',
      phone: ''
    });
    setErrors({});
    setChangeCount(0);
    setChangeHistory([]);
    setIsEditing(false);
  };
  
  // Función para guardar el formulario
  const saveForm = () => {
    // Verificar si hay errores
    if (Object.keys(errors).length > 0) {
      Alert.alert(
        'Errores de Validación',
        'Por favor corrija los errores antes de guardar',
        [{ text: 'OK' }]
      );
      return;
    }
    
    // Verificar que todos los campos estén llenos
    const requiredFields = ['name', 'email', 'age', 'phone'];
    const emptyFields = requiredFields.filter(field => !userForm[field]);
    
    if (emptyFields.length > 0) {
      Alert.alert(
        'Campos Requeridos',
        `Los siguientes campos son obligatorios: ${emptyFields.join(', ')}`,
        [{ text: 'OK' }]
      );
      return;
    }
    
    // Simular guardado exitoso
    Alert.alert(
      'Éxito',
      'Formulario guardado correctamente',
      [{ text: 'OK' }]
    );
    
    // Cambiar a modo de visualización
    setIsEditing(false);
  };
  
  // Función para alternar modo de edición
  const toggleEditMode = () => {
    setIsEditing(prev => !prev);
  };
  
  // Función para deshacer último cambio
  const undoLastChange = () => {
    if (changeHistory.length > 0) {
      const lastChange = changeHistory[changeHistory.length - 1];
      
      // Revertir el cambio
      setUserForm(prev => ({
        ...prev,
        [lastChange.field]: lastChange.oldValue
      }));
      
      // Remover del historial
      setChangeHistory(prev => prev.slice(0, -1));
      
      // Decrementar contador
      setChangeCount(prev => Math.max(0, prev - 1));
    }
  };
  
  // Función para resetear contador de cambios
  const resetChangeCount = () => {
    setChangeCount(0);
    setChangeHistory([]);
  };
  
  return (
    <ScrollView style={styles.container}>
      {/* Header del formulario */}
      <View style={styles.header}>
        <Text style={styles.title}>Gestión de Estado Local</Text>
        <Text style={styles.subtitle}>Ejemplo con useState y useEffect</Text>
      </View>
      
      {/* Estadísticas del formulario */}
      <View style={styles.statsContainer}>
        <View style={styles.statItem}>
          <Text style={styles.statLabel}>Cambios</Text>
          <Text style={styles.statValue}>{changeCount}</Text>
        </View>
        <View style={styles.statItem}>
          <Text style={styles.statLabel}>Modo</Text>
          <Text style={styles.statValue}>{isEditing ? 'Edición' : 'Vista'}</Text>
        </View>
        <View style={styles.statItem}>
          <Text style={styles.statLabel}>Errores</Text>
          <Text style={styles.statValue}>{Object.keys(errors).length}</Text>
        </View>
      </View>
      
      {/* Formulario */}
      <View style={styles.formContainer}>
        {/* Campo Nombre */}
        <View style={styles.fieldContainer}>
          <Text style={styles.fieldLabel}>Nombre *</Text>
          <TextInput
            style={[
              styles.textInput,
              errors.name && styles.errorInput,
              !isEditing && styles.disabledInput
            ]}
            value={userForm.name}
            onChangeText={(value) => updateField('name', value)}
            placeholder="Ingrese su nombre"
            editable={isEditing}
            maxLength={50}
          />
          {errors.name && <Text style={styles.errorText}>{errors.name}</Text>}
        </View>
        
        {/* Campo Email */}
        <View style={styles.fieldContainer}>
          <Text style={styles.fieldLabel}>Email *</Text>
          <TextInput
            style={[
              styles.textInput,
              errors.email && styles.errorInput,
              !isEditing && styles.disabledInput
            ]}
            value={userForm.email}
            onChangeText={(value) => updateField('email', value)}
            placeholder="usuario@ejemplo.com"
            editable={isEditing}
            keyboardType="email-address"
            autoCapitalize="none"
          />
          {errors.email && <Text style={styles.errorText}>{errors.email}</Text>}
        </View>
        
        {/* Campo Edad */}
        <View style={styles.fieldContainer}>
          <Text style={styles.fieldLabel}>Edad *</Text>
          <TextInput
            style={[
              styles.textInput,
              errors.age && styles.errorInput,
              !isEditing && styles.disabledInput
            ]}
            value={userForm.age}
            onChangeText={(value) => updateField('age', value)}
            placeholder="Ingrese su edad"
            editable={isEditing}
            keyboardType="numeric"
            maxLength={3}
          />
          {errors.age && <Text style={styles.errorText}>{errors.age}</Text>}
        </View>
        
        {/* Campo Teléfono */}
        <View style={styles.fieldContainer}>
          <Text style={styles.fieldLabel}>Teléfono *</Text>
          <TextInput
            style={[
              styles.textInput,
              errors.phone && styles.errorInput,
              !isEditing && styles.disabledInput
            ]}
            value={userForm.phone}
            onChangeText={(value) => updateField('phone', value)}
            placeholder="Ingrese su teléfono"
            editable={isEditing}
            keyboardType="phone-pad"
            maxLength={15}
          />
          {errors.phone && <Text style={styles.errorText}>{errors.phone}</Text>}
        </View>
      </View>
      
      {/* Botones de acción */}
      <View style={styles.buttonContainer}>
        {isEditing ? (
          <>
            <TouchableOpacity style={styles.saveButton} onPress={saveForm}>
              <Text style={styles.buttonText}>Guardar</Text>
            </TouchableOpacity>
            <TouchableOpacity style={styles.cancelButton} onPress={toggleEditMode}>
              <Text style={styles.buttonText}>Cancelar</Text>
            </TouchableOpacity>
          </>
        ) : (
          <TouchableOpacity style={styles.editButton} onPress={toggleEditMode}>
            <Text style={styles.buttonText}>Editar</Text>
          </TouchableOpacity>
        )}
        
        <TouchableOpacity style={styles.clearButton} onPress={clearForm}>
          <Text style={styles.buttonText}>Limpiar</Text>
        </TouchableOpacity>
      </View>
      
      {/* Botones de utilidad */}
      <View style={styles.utilityButtonContainer}>
        <TouchableOpacity 
          style={styles.utilityButton} 
          onPress={undoLastChange}
          disabled={changeHistory.length === 0}
        >
          <Text style={styles.utilityButtonText}>Deshacer</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={styles.utilityButton} 
          onPress={resetChangeCount}
          disabled={changeCount === 0}
        >
          <Text style={styles.utilityButtonText}>Resetear Contador</Text>
        </TouchableOpacity>
      </View>
      
      {/* Historial de cambios */}
      {changeHistory.length > 0 && (
        <View style={styles.historyContainer}>
          <Text style={styles.historyTitle}>Historial de Cambios</Text>
          <ScrollView style={styles.historyList}>
            {changeHistory.slice(-5).reverse().map((change, index) => (
              <View key={index} style={styles.historyItem}>
                <Text style={styles.historyField}>{change.field}</Text>
                <Text style={styles.historyChange}>
                  {change.oldValue} → {change.newValue}
                </Text>
                <Text style={styles.historyTime}>{change.timestamp}</Text>
              </View>
            ))}
          </ScrollView>
        </View>
      )}
    </ScrollView>
  );
};

// Estilos del componente
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f8f9fa',
    padding: 20,
  },
  
  header: {
    alignItems: 'center',
    marginBottom: 20,
  },
  
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
  
  statsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 10,
    marginBottom: 20,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  
  statItem: {
    alignItems: 'center',
  },
  
  statLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 5,
  },
  
  statValue: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#007bff',
  },
  
  formContainer: {
    backgroundColor: '#fff',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  
  fieldContainer: {
    marginBottom: 15,
  },
  
  fieldLabel: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
    marginBottom: 8,
  },
  
  textInput: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    backgroundColor: '#fff',
  },
  
  errorInput: {
    borderColor: '#dc3545',
    backgroundColor: '#fff5f5',
  },
  
  disabledInput: {
    backgroundColor: '#f8f9fa',
    color: '#666',
  },
  
  errorText: {
    color: '#dc3545',
    fontSize: 14,
    marginTop: 5,
  },
  
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20,
  },
  
  saveButton: {
    backgroundColor: '#28a745',
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  
  editButton: {
    backgroundColor: '#007bff',
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  
  cancelButton: {
    backgroundColor: '#6c757d',
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  
  clearButton: {
    backgroundColor: '#dc3545',
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
  },
  
  utilityButtonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20,
  },
  
  utilityButton: {
    backgroundColor: '#6c757d',
    paddingVertical: 10,
    paddingHorizontal: 16,
    borderRadius: 6,
    minWidth: 120,
    alignItems: 'center',
  },
  
  utilityButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: '500',
  },
  
  historyContainer: {
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 10,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  
  historyTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 15,
    textAlign: 'center',
  },
  
  historyList: {
    maxHeight: 200,
  },
  
  historyItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  
  historyField: {
    fontSize: 14,
    fontWeight: '600',
    color: '#007bff',
    flex: 1,
  },
  
  historyChange: {
    fontSize: 14,
    color: '#666',
    flex: 2,
    textAlign: 'center',
  },
  
  historyTime: {
    fontSize: 12,
    color: '#999',
    flex: 1,
    textAlign: 'right',
  },
});

export default LocalStateExample;
```

### **2. Hook Personalizado para Gestión de Estado**

```javascript:src/hooks/useFormState.js
import { useState, useEffect, useCallback, useRef } from 'react';

// Hook personalizado para gestión de formularios
const useFormState = (initialState = {}, validationRules = {}) => {
  // Estado del formulario
  const [formData, setFormData] = useState(initialState);
  
  // Estado de errores de validación
  const [errors, setErrors] = useState({});
  
  // Estado de campos tocados/modificados
  const [touched, setTouched] = useState({});
  
  // Estado de validación en tiempo real
  const [isValidating, setIsValidating] = useState(false);
  
  // Referencia para el estado anterior
  const previousFormData = useRef(initialState);
  
  // Referencia para el contador de cambios
  const changeCount = useRef(0);
  
  // Referencia para el historial de cambios
  const changeHistory = useRef([]);
  
  // Función para actualizar un campo específico
  const updateField = useCallback((field, value) => {
    const oldValue = formData[field];
    
    // Actualizar el formulario
    setFormData(prev => ({
      ...prev,
      [field]: value
    }));
    
    // Marcar como tocado
    setTouched(prev => ({
      ...prev,
      [field]: true
    }));
    
    // Incrementar contador de cambios
    changeCount.current += 1;
    
    // Agregar al historial
    changeHistory.current.push({
      field,
      oldValue,
      newValue: value,
      timestamp: Date.now()
    });
    
    // Limpiar error del campo si existe
    if (errors[field]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[field];
        return newErrors;
      });
    }
  }, [formData, errors]);
  
  // Función para actualizar múltiples campos
  const updateMultipleFields = useCallback((updates) => {
    const oldValues = {};
    const newValues = {};
    
    Object.keys(updates).forEach(field => {
      oldValues[field] = formData[field];
      newValues[field] = updates[field];
    });
    
    // Actualizar el formulario
    setFormData(prev => ({
      ...prev,
      ...updates
    }));
    
    // Marcar campos como tocados
    setTouched(prev => ({
      ...prev,
      ...Object.keys(updates).reduce((acc, field) => {
        acc[field] = true;
        return acc;
      }, {})
    }));
    
    // Incrementar contador
    changeCount.current += 1;
    
    // Agregar al historial
    changeHistory.current.push({
      fields: Object.keys(updates),
      oldValues,
      newValues,
      timestamp: Date.now()
    });
  }, [formData]);
  
  // Función para resetear el formulario
  const resetForm = useCallback(() => {
    setFormData(initialState);
    setErrors({});
    setTouched({});
    changeCount.current = 0;
    changeHistory.current = [];
    previousFormData.current = initialState;
  }, [initialState]);
  
  // Función para establecer el formulario
  const setForm = useCallback((newData) => {
    setFormData(newData);
    setTouched({});
    setErrors({});
  }, []);
  
  // Función para validar un campo específico
  const validateField = useCallback((field, value) => {
    if (!validationRules[field]) return null;
    
    const rule = validationRules[field];
    let error = null;
    
    // Validación requerida
    if (rule.required && (!value || value.trim() === '')) {
      error = rule.requiredMessage || 'Este campo es requerido';
    }
    
    // Validación de longitud mínima
    if (rule.minLength && value && value.length < rule.minLength) {
      error = rule.minLengthMessage || `Mínimo ${rule.minLength} caracteres`;
    }
    
    // Validación de longitud máxima
    if (rule.maxLength && value && value.length > rule.maxLength) {
      error = rule.maxLengthMessage || `Máximo ${rule.maxLength} caracteres`;
    }
    
    // Validación de patrón (regex)
    if (rule.pattern && value && !rule.pattern.test(value)) {
      error = rule.patternMessage || 'Formato inválido';
    }
    
    // Validación personalizada
    if (rule.custom && value) {
      const customError = rule.custom(value, formData);
      if (customError) {
        error = customError;
      }
    }
    
    return error;
  }, [validationRules, formData]);
  
  // Función para validar todo el formulario
  const validateForm = useCallback(() => {
    setIsValidating(true);
    
    const newErrors = {};
    let isValid = true;
    
    Object.keys(validationRules).forEach(field => {
      const error = validateField(field, formData[field]);
      if (error) {
        newErrors[field] = error;
        isValid = false;
      }
    });
    
    setErrors(newErrors);
    setIsValidating(false);
    
    return isValid;
  }, [validationRules, formData, validateField]);
  
  // Función para obtener estadísticas del formulario
  const getFormStats = useCallback(() => {
    return {
      changeCount: changeCount.current,
      changeHistory: [...changeHistory.current],
      touchedFields: Object.keys(touched),
      errorCount: Object.keys(errors).length,
      isValid: Object.keys(errors).length === 0,
      hasChanges: changeCount.current > 0,
    };
  }, [touched, errors]);
  
  // Función para deshacer último cambio
  const undoLastChange = useCallback(() => {
    if (changeHistory.current.length > 0) {
      const lastChange = changeHistory.current[changeHistory.current.length - 1];
      
      if (lastChange.fields) {
        // Cambio múltiple
        setFormData(prev => ({
          ...prev,
          ...lastChange.oldValues
        }));
      } else {
        // Cambio simple
        setFormData(prev => ({
          ...prev,
          [lastChange.field]: lastChange.oldValue
        }));
      }
      
      // Remover del historial
      changeHistory.current.pop();
      
      // Decrementar contador
      changeCount.current = Math.max(0, changeCount.current - 1);
    }
  }, []);
  
  // Función para limpiar historial
  const clearHistory = useCallback(() => {
    changeHistory.current = [];
    changeCount.current = 0;
  }, []);
  
  // Efecto para validación automática
  useEffect(() => {
    if (Object.keys(touched).length > 0) {
      const newErrors = {};
      
      Object.keys(touched).forEach(field => {
        const error = validateField(field, formData[field]);
        if (error) {
          newErrors[field] = error;
        }
      });
      
      setErrors(newErrors);
    }
  }, [formData, touched, validateField]);
  
  // Efecto para guardar estado anterior
  useEffect(() => {
    previousFormData.current = formData;
  }, [formData]);
  
  return {
    // Estado
    formData,
    errors,
    touched,
    isValidating,
    
    // Funciones de actualización
    updateField,
    updateMultipleFields,
    
    // Funciones de validación
    validateField,
    validateForm,
    
    // Funciones de utilidad
    resetForm,
    setForm,
    getFormStats,
    undoLastChange,
    clearHistory,
    
    // Información adicional
    changeCount: changeCount.current,
    hasChanges: changeCount.current > 0,
    previousData: previousFormData.current,
  };
};

export default useFormState;
```

### **3. Utilidades para Gestión de Estado**

```javascript:src/utils/stateUtils.js
// Utilidades para gestión de estado en React Native

// Función para crear reglas de validación
export const createValidationRules = (rules) => {
  const validationRules = {};
  
  Object.keys(rules).forEach(field => {
    const rule = rules[field];
    validationRules[field] = {
      required: rule.required || false,
      requiredMessage: rule.requiredMessage || 'Este campo es requerido',
      
      minLength: rule.minLength || null,
      minLengthMessage: rule.minLengthMessage || `Mínimo ${rule.minLength} caracteres`,
      
      maxLength: rule.maxLength || null,
      maxLengthMessage: rule.maxLengthMessage || `Máximo ${rule.maxLength} caracteres`,
      
      pattern: rule.pattern || null,
      patternMessage: rule.patternMessage || 'Formato inválido',
      
      custom: rule.custom || null,
    };
  });
  
  return validationRules;
};

// Función para validar email
export const validateEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Función para validar teléfono
export const validatePhone = (phone) => {
  const phoneRegex = /^[\+]?[1-9][\d]{0,15}$/;
  return phoneRegex.test(phone.replace(/\s/g, ''));
};

// Función para validar contraseña
export const validatePassword = (password) => {
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
  
  const errors = [];
  
  if (password.length < minLength) {
    errors.push(`Mínimo ${minLength} caracteres`);
  }
  
  if (!hasUpperCase) {
    errors.push('Al menos una mayúscula');
  }
  
  if (!hasLowerCase) {
    errors.push('Al menos una minúscula');
  }
  
  if (!hasNumbers) {
    errors.push('Al menos un número');
  }
  
  if (!hasSpecialChar) {
    errors.push('Al menos un carácter especial');
  }
  
  return errors.length === 0 ? null : errors.join(', ');
};

// Función para crear estado inicial
export const createInitialState = (fields) => {
  const initialState = {};
  
  fields.forEach(field => {
    if (typeof field === 'string') {
      initialState[field] = '';
    } else if (typeof field === 'object') {
      initialState[field.name] = field.defaultValue || '';
    }
  });
  
  return initialState;
};

// Función para limpiar estado
export const cleanState = (state, fieldsToKeep = []) => {
  const cleanedState = {};
  
  if (fieldsToKeep.length > 0) {
    fieldsToKeep.forEach(field => {
      if (state.hasOwnProperty(field)) {
        cleanedState[field] = state[field];
      }
    });
  } else {
    Object.keys(state).forEach(key => {
      if (state[key] !== null && state[key] !== undefined) {
        cleanedState[key] = state[key];
      }
    });
  }
  
  return cleanedState;
};

// Función para comparar estados
export const compareStates = (state1, state2, fieldsToCompare = null) => {
  const fields = fieldsToCompare || Object.keys(state1);
  const differences = {};
  
  fields.forEach(field => {
    if (state1[field] !== state2[field]) {
      differences[field] = {
        old: state1[field],
        new: state2[field]
      };
    }
  });
  
  return {
    hasChanges: Object.keys(differences).length > 0,
    differences,
    changeCount: Object.keys(differences).length
  };
};

// Función para crear estado persistente
export const createPersistentState = (key, initialState) => {
  // En una implementación real, usarías AsyncStorage
  const stored = localStorage.getItem(key);
  return stored ? JSON.parse(stored) : initialState;
};

// Función para guardar estado persistente
export const savePersistentState = (key, state) => {
  // En una implementación real, usarías AsyncStorage
  localStorage.setItem(key, JSON.stringify(state));
};

// Función para crear estado con debounce
export const createDebouncedState = (initialState, delay = 300) {
  let timeoutId = null;
  
  const setState = (newState) => {
    if (timeoutId) {
      clearTimeout(timeoutId);
    }
    
    timeoutId = setTimeout(() => {
      // Aquí implementarías la lógica de actualización
      console.log('Estado actualizado:', newState);
    }, delay);
  };
  
  return {
    setState,
    cancel: () => {
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
    }
  };
};
```

---

## 🧪 Casos de Uso

### **Caso 1: Formulario de Usuario**
```javascript
// Usar el hook personalizado
const { formData, errors, updateField, validateForm } = useFormState(
  { name: '', email: '', age: '' },
  {
    name: { required: true, minLength: 2 },
    email: { required: true, pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
    age: { required: true, minLength: 1, maxLength: 3 }
  }
);
```

### **Caso 2: Validación en Tiempo Real**
```javascript
// Validar campo al cambiar
const handleFieldChange = (field, value) => {
  updateField(field, value);
  // La validación se ejecuta automáticamente
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Formulario de Registro**
Implementa un formulario de registro con validación en tiempo real.

### **Ejercicio 2: Gestión de Lista**
Crea un componente que gestione una lista de elementos con estado local.

### **Ejercicio 3: Formulario Multi-paso**
Desarrolla un formulario que se divida en múltiples pasos con validación.

---

## 🚀 Proyecto de la Clase

### **App de Gestión de Estado Local**

Crea una aplicación que demuestre:
- **Formularios complejos**: Con validación en tiempo real
- **Estado reactivo**: Que se actualice automáticamente
- **Validación personalizada**: Reglas específicas por campo
- **Historial de cambios**: Seguimiento de modificaciones

**Requisitos:**
1. Implementar formulario con múltiples campos
2. Crear validación en tiempo real
3. Usar el hook personalizado de formulario
4. Implementar historial de cambios
5. Crear interfaz de usuario intuitiva

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Estado local**: Gestión de datos en componentes individuales
- **useState y useEffect**: Hooks fundamentales para estado
- **Validación en tiempo real**: Verificación automática de campos
- **Patrón de elevación**: Compartir estado entre componentes

### **Habilidades Desarrolladas:**
- ✅ Usar useState para gestión básica de estado
- ✅ Implementar useEffect para efectos secundarios
- ✅ Crear hooks personalizados para formularios
- ✅ Implementar validación en tiempo real
- ✅ Gestionar estado complejo de manera eficiente

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Context API**, que te permitirá compartir estado entre componentes sin pasar props manualmente.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Módulo 3: Navegación y Routing](../midLevel_1/README.md)
- **➡️ Siguiente Clase**: [Context API](clase_2_context_api.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
