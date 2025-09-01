# 📚 Clase 5: Validación y Estado Complejo

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 4: Manejo de Eventos y Formularios](clase_4_manejo_eventos_formularios.md)
- **➡️ Siguiente**: [Módulo 4: Navegación y Routing](../midLevel_1/README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Implementar sistemas de validación avanzados
- Gestionar estados complejos en formularios
- Crear validaciones personalizadas y asíncronas
- Implementar manejo de errores robusto
- Crear sistemas de validación reutilizables

---

## 📚 Contenido Teórico

### **¿Qué es la Validación Avanzada?**

La validación avanzada va más allá de simples verificaciones de campos requeridos. Incluye validaciones personalizadas, asíncronas, dependientes entre campos y manejo de errores complejos.

#### **Tipos de validación avanzada:**
- **Validación personalizada**: Lógica específica del negocio
- **Validación asíncrona**: Verificación contra servidores
- **Validación dependiente**: Campos que dependen de otros
- **Validación condicional**: Reglas que cambian según el contexto

---

## 💻 Implementación Práctica

### **1. Sistema de Validación Avanzado**

```javascript:src/components/AdvancedValidationExample.js
import React, { useState, useRef, useEffect } from 'react';
import { 
  View, 
  Text, 
  TextInput, 
  TouchableOpacity, 
  StyleSheet, 
  ScrollView,
  Alert,
  ActivityIndicator 
} from 'react-native';

// Componente que demuestra validación avanzada
const AdvancedValidationExample = () => {
  // Referencias para inputs
  const usernameRef = useRef(null);
  const emailRef = useRef(null);
  const passwordRef = useRef(null);
  const confirmPasswordRef = useRef(null);
  const ageRef = useRef(null);
  const websiteRef = useRef(null);

  // Estado del formulario
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    age: '',
    website: '',
    acceptTerms: false,
    newsletter: false,
  });

  // Estado de validación
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isValidating, setIsValidating] = useState({});
  const [validationHistory, setValidationHistory] = useState({});

  // Estado para validaciones asíncronas
  const [asyncValidationQueue, setAsyncValidationQueue] = useState([]);
  const [isProcessingQueue, setIsProcessingQueue] = useState(false);

  // Reglas de validación avanzadas
  const validationRules = {
    username: {
      required: true,
      minLength: 3,
      maxLength: 20,
      pattern: /^[a-zA-Z0-9_]+$/,
      custom: (value) => {
        if (value.includes('admin') || value.includes('root')) {
          return 'El nombre de usuario no puede contener palabras reservadas';
        }
        return null;
      },
      async: async (value) => {
        // Simular verificación de disponibilidad
        await new Promise(resolve => setTimeout(resolve, 1000));
        const takenUsernames = ['john', 'jane', 'test'];
        if (takenUsernames.includes(value.toLowerCase())) {
          return 'Este nombre de usuario ya está en uso';
        }
        return null;
      }
    },
    email: {
      required: true,
      pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      async: async (value) => {
        // Simular verificación de dominio
        await new Promise(resolve => setTimeout(resolve, 800));
        const blockedDomains = ['tempmail.com', 'spam.com'];
        const domain = value.split('@')[1];
        if (blockedDomains.includes(domain)) {
          return 'Este dominio de email no está permitido';
        }
        return null;
      }
    },
    password: {
      required: true,
      minLength: 8,
      custom: (value) => {
        const hasUpperCase = /[A-Z]/.test(value);
        const hasLowerCase = /[a-z]/.test(value);
        const hasNumbers = /\d/.test(value);
        const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(value);
        
        const errors = [];
        if (!hasUpperCase) errors.push('mayúsculas');
        if (!hasLowerCase) errors.push('minúsculas');
        if (!hasNumbers) errors.push('números');
        if (!hasSpecialChar) errors.push('caracteres especiales');
        
        if (errors.length > 0) {
          return `Debe contener: ${errors.join(', ')}`;
        }
        return null;
      }
    },
    confirmPassword: {
      required: true,
      custom: (value) => {
        if (value !== formData.password) {
          return 'Las contraseñas no coinciden';
        }
        return null;
      }
    },
    age: {
      custom: (value) => {
        if (value && (isNaN(value) || value < 13 || value > 120)) {
          return 'La edad debe estar entre 13 y 120 años';
        }
        return null;
      }
    },
    website: {
      pattern: /^https?:\/\/.+/,
      custom: (value) => {
        if (value && !value.startsWith('http')) {
          return 'La URL debe comenzar con http:// o https://';
        }
        return null;
      }
    }
  };

  // Función para actualizar campo
  const updateField = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    setTouched(prev => ({ ...prev, [field]: true }));
    
    // Limpiar error previo
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' }));
    }
    
    // Validar campo
    validateField(field, value);
  };

  // Función para validar campo individual
  const validateField = async (field, value) => {
    const rules = validationRules[field];
    if (!rules) return true;

    setIsValidating(prev => ({ ...prev, [field]: true }));
    
    let error = '';
    let validationType = '';

    try {
      // Validación requerida
      if (rules.required && (!value || value.trim() === '')) {
        error = 'Este campo es requerido';
        validationType = 'required';
      }
      // Validación de longitud mínima
      else if (rules.minLength && value && value.length < rules.minLength) {
        error = `Mínimo ${rules.minLength} caracteres`;
        validationType = 'length';
      }
      // Validación de longitud máxima
      else if (rules.maxLength && value && value.length > rules.maxLength) {
        error = `Máximo ${rules.maxLength} caracteres`;
        validationType = 'length';
      }
      // Validación de patrón
      else if (rules.pattern && value && !rules.pattern.test(value)) {
        error = 'Formato inválido';
        validationType = 'pattern';
      }
      // Validación personalizada
      else if (rules.custom && value) {
        error = rules.custom(value);
        validationType = 'custom';
      }
      // Validación asíncrona
      else if (rules.async && value && !error) {
        try {
          error = await rules.async(value);
          validationType = 'async';
        } catch (asyncError) {
          error = 'Error en validación asíncrona';
          validationType = 'async_error';
        }
      }
    } catch (validationError) {
      error = 'Error en validación';
      validationType = 'error';
    }

    // Actualizar errores
    setErrors(prev => ({
      ...prev,
      [field]: error
    }));

    // Actualizar historial de validación
    setValidationHistory(prev => ({
      ...prev,
      [field]: {
        timestamp: Date.now(),
        value,
        error,
        type: validationType,
        success: !error
      }
    }));

    setIsValidating(prev => ({ ...prev, [field]: false }));
    
    return error === '';
  };

  // Función para validar todo el formulario
  const validateForm = async () => {
    const fields = Object.keys(validationRules);
    const validationPromises = fields.map(field => 
      validateField(field, formData[field])
    );
    
    const results = await Promise.all(validationPromises);
    const isValid = results.every(result => result === true);
    
    return isValid;
  };

  // Función para enviar formulario
  const handleSubmit = async () => {
    const isValid = await validateForm();
    
    if (!isValid) {
      Alert.alert('Error', 'Por favor corrige los errores del formulario');
      return;
    }

    if (!formData.acceptTerms) {
      Alert.alert('Error', 'Debes aceptar los términos y condiciones');
      return;
    }

    Alert.alert('Éxito', 'Formulario validado correctamente');
  };

  // Función para resetear formulario
  const resetForm = () => {
    setFormData({
      username: '',
      email: '',
      password: '',
      confirmPassword: '',
      age: '',
      website: '',
      acceptTerms: false,
      newsletter: false,
    });
    setErrors({});
    setTouched({});
    setValidationHistory({});
  };

  // Función para obtener estilo del input
  const getInputStyle = (field) => {
    const baseStyle = styles.input;
    const errorStyle = errors[field] && touched[field] ? styles.inputError : null;
    const successStyle = !errors[field] && touched[field] && formData[field] ? styles.inputSuccess : null;
    const validatingStyle = isValidating[field] ? styles.inputValidating : null;
    
    return [baseStyle, errorStyle, successStyle, validatingStyle];
  };

  // Función para obtener mensaje de error
  const getErrorMessage = (field) => {
    if (errors[field] && touched[field]) {
      return <Text style={styles.errorText}>{errors[field]}</Text>;
    }
    return null;
  };

  // Función para obtener indicador de validación
  const getValidationIndicator = (field) => {
    if (isValidating[field]) {
      return <ActivityIndicator size="small" color="#3498db" style={styles.validationIndicator} />;
    }
    
    if (errors[field] && touched[field]) {
      return <Text style={styles.errorIcon}>❌</Text>;
    }
    
    if (!errors[field] && touched[field] && formData[field]) {
      return <Text style={styles.successIcon}>✅</Text>;
    }
    
    return null;
  };

  // Función para obtener estadísticas de validación
  const getValidationStats = () => {
    const totalFields = Object.keys(validationRules).length;
    const validFields = Object.keys(validationRules).filter(field => 
      !errors[field] && touched[field] && formData[field]
    ).length;
    const errorFields = Object.keys(errors).filter(field => errors[field]).length;
    const pendingFields = totalFields - validFields - errorFields;

    return { totalFields, validFields, errorFields, pendingFields };
  };

  const stats = getValidationStats();

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Validación Avanzada</Text>
        <Text style={styles.subtitle}>
          Sistema completo de validación con reglas personalizadas y asíncronas
        </Text>
      </View>

      {/* Estadísticas de Validación */}
      <View style={styles.statsContainer}>
        <Text style={styles.statsTitle}>Estado de Validación</Text>
        <View style={styles.statsRow}>
          <View style={styles.statItem}>
            <Text style={styles.statNumber}>{stats.validFields}</Text>
            <Text style={styles.statLabel}>Válidos</Text>
          </View>
          <View style={styles.statItem}>
            <Text style={styles.statNumber}>{stats.errorFields}</Text>
            <Text style={styles.statLabel}>Con Errores</Text>
          </View>
          <View style={styles.statItem}>
            <Text style={styles.statNumber}>{stats.pendingFields}</Text>
            <Text style={styles.statLabel}>Pendientes</Text>
          </View>
        </View>
      </View>

      {/* Campo Username */}
      <View style={styles.fieldContainer}>
        <View style={styles.fieldHeader}>
          <Text style={styles.label}>Nombre de Usuario *</Text>
          {getValidationIndicator('username')}
        </View>
        <TextInput
          ref={usernameRef}
          style={getInputStyle('username')}
          placeholder="Mínimo 3 caracteres, solo letras, números y _"
          value={formData.username}
          onChangeText={(text) => updateField('username', text)}
          onBlur={() => validateField('username', formData.username)}
          returnKeyType="next"
          onSubmitEditing={() => emailRef.current?.focus()}
        />
        {getErrorMessage('username')}
        <Text style={styles.fieldHint}>
          El nombre de usuario se verificará contra la base de datos
        </Text>
      </View>

      {/* Campo Email */}
      <View style={styles.fieldContainer}>
        <View style={styles.fieldHeader}>
          <Text style={styles.label}>Email *</Text>
          {getValidationIndicator('email')}
        </View>
        <TextInput
          ref={emailRef}
          style={getInputStyle('email')}
          placeholder="tu@email.com"
          value={formData.email}
          onChangeText={(text) => updateField('email', text)}
          onBlur={() => validateField('email', formData.email)}
          keyboardType="email-address"
          autoCapitalize="none"
          returnKeyType="next"
          onSubmitEditing={() => passwordRef.current?.focus()}
        />
        {getErrorMessage('email')}
        <Text style={styles.fieldHint}>
          Se verificará que el dominio del email esté permitido
        </Text>
      </View>

      {/* Campo Password */}
      <View style={styles.fieldContainer}>
        <View style={styles.fieldHeader}>
          <Text style={styles.label}>Contraseña *</Text>
          {getValidationIndicator('password')}
        </View>
        <TextInput
          ref={passwordRef}
          style={getInputStyle('password')}
          placeholder="Mínimo 8 caracteres con mayúsculas, minúsculas, números y símbolos"
          value={formData.password}
          onChangeText={(text) => updateField('password', text)}
          onBlur={() => validateField('password', formData.password)}
          secureTextEntry={true}
          returnKeyType="next"
          onSubmitEditing={() => confirmPasswordRef.current?.focus()}
        />
        {getErrorMessage('password')}
        <Text style={styles.fieldHint}>
          Debe contener mayúsculas, minúsculas, números y caracteres especiales
        </Text>
      </View>

      {/* Campo Confirmar Password */}
      <View style={styles.fieldContainer}>
        <View style={styles.fieldHeader}>
          <Text style={styles.label}>Confirmar Contraseña *</Text>
          {getValidationIndicator('confirmPassword')}
        </View>
        <TextInput
          ref={confirmPasswordRef}
          style={getInputStyle('confirmPassword')}
          placeholder="Repite tu contraseña"
          value={formData.confirmPassword}
          onChangeText={(text) => updateField('confirmPassword', text)}
          onBlur={() => validateField('confirmPassword', formData.confirmPassword)}
          secureTextEntry={true}
          returnKeyType="next"
          onSubmitEditing={() => ageRef.current?.focus()}
        />
        {getErrorMessage('confirmPassword')}
      </View>

      {/* Campo Edad */}
      <View style={styles.fieldContainer}>
        <View style={styles.fieldHeader}>
          <Text style={styles.label}>Edad</Text>
          {getValidationIndicator('age')}
        </View>
        <TextInput
          ref={ageRef}
          style={getInputStyle('age')}
          placeholder="Entre 13 y 120 años"
          value={formData.age}
          onChangeText={(text) => updateField('age', text)}
          onBlur={() => validateField('age', formData.age)}
          keyboardType="numeric"
          returnKeyType="next"
          onSubmitEditing={() => websiteRef.current?.focus()}
        />
        {getErrorMessage('age')}
      </View>

      {/* Campo Website */}
      <View style={styles.fieldContainer}>
        <View style={styles.fieldHeader}>
          <Text style={styles.label}>Sitio Web</Text>
          {getValidationIndicator('website')}
        </View>
        <TextInput
          ref={websiteRef}
          style={getInputStyle('website')}
          placeholder="https://tusitio.com"
          value={formData.website}
          onChangeText={(text) => updateField('website', text)}
          onBlur={() => validateField('website', formData.website)}
          autoCapitalize="none"
          returnKeyType="done"
        />
        {getErrorMessage('website')}
        <Text style={styles.fieldHint}>
          Debe comenzar con http:// o https://
        </Text>
      </View>

      {/* Términos y Condiciones */}
      <View style={styles.termsContainer}>
        <TouchableOpacity 
          style={styles.checkboxContainer}
          onPress={() => setFormData(prev => ({ ...prev, acceptTerms: !prev.acceptTerms }))}
        >
          <View style={[
            styles.checkbox, 
            formData.acceptTerms && styles.checkboxChecked
          ]}>
            {formData.acceptTerms && <Text style={styles.checkmark}>✓</Text>}
          </View>
          <Text style={styles.termsText}>
            Acepto los términos y condiciones *
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={styles.checkboxContainer}
          onPress={() => setFormData(prev => ({ ...prev, newsletter: !prev.newsletter }))}
        >
          <View style={[
            styles.checkbox, 
            formData.newsletter && styles.checkboxChecked
          ]}>
            {formData.newsletter && <Text style={styles.checkmark}>✓</Text>}
          </View>
          <Text style={styles.termsText}>
            Suscribirme al boletín de noticias
          </Text>
        </TouchableOpacity>
      </View>

      {/* Botones de Acción */}
      <View style={styles.buttonContainer}>
        <TouchableOpacity 
          style={styles.submitButton}
          onPress={handleSubmit}
        >
          <Text style={styles.buttonText}>Validar y Enviar</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={styles.resetButton}
          onPress={resetForm}
        >
          <Text style={styles.resetButtonText}>Limpiar Formulario</Text>
        </TouchableOpacity>
      </View>

      {/* Historial de Validación */}
      <View style={styles.historyContainer}>
        <Text style={styles.historyTitle}>Historial de Validación</Text>
        {Object.entries(validationHistory).map(([field, history]) => (
          <View key={field} style={styles.historyItem}>
            <Text style={styles.historyField}>{field}:</Text>
            <Text style={[
              styles.historyStatus,
              history.success ? styles.historySuccess : styles.historyError
            ]}>
              {history.success ? '✅ Válido' : `❌ ${history.error}`}
            </Text>
            <Text style={styles.historyTime}>
              {new Date(history.timestamp).toLocaleTimeString()}
            </Text>
          </View>
        ))}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    padding: 20,
    backgroundColor: 'white',
    marginBottom: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 10,
    color: '#2c3e50',
  },
  subtitle: {
    fontSize: 16,
    textAlign: 'center',
    color: '#7f8c8d',
  },
  statsContainer: {
    backgroundColor: 'white',
    marginHorizontal: 20,
    marginBottom: 20,
    padding: 20,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  statsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#2c3e50',
    textAlign: 'center',
  },
  statsRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  statItem: {
    alignItems: 'center',
  },
  statNumber: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#3498db',
  },
  statLabel: {
    fontSize: 12,
    color: '#7f8c8d',
    marginTop: 5,
  },
  fieldContainer: {
    backgroundColor: 'white',
    marginHorizontal: 20,
    marginBottom: 20,
    padding: 20,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  fieldHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8,
  },
  label: {
    fontSize: 16,
    fontWeight: '600',
    color: '#2c3e50',
    flex: 1,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    backgroundColor: '#f8f9fa',
  },
  inputError: {
    borderColor: '#e74c3c',
    backgroundColor: '#fdf2f2',
  },
  inputSuccess: {
    borderColor: '#27ae60',
    backgroundColor: '#f0f9f0',
  },
  inputValidating: {
    borderColor: '#3498db',
    backgroundColor: '#f0f8ff',
  },
  errorText: {
    color: '#e74c3c',
    fontSize: 14,
    marginTop: 5,
  },
  fieldHint: {
    fontSize: 12,
    color: '#7f8c8d',
    marginTop: 5,
    fontStyle: 'italic',
  },
  validationIndicator: {
    marginLeft: 10,
  },
  errorIcon: {
    fontSize: 16,
    marginLeft: 10,
  },
  successIcon: {
    fontSize: 16,
    marginLeft: 10,
  },
  termsContainer: {
    backgroundColor: 'white',
    marginHorizontal: 20,
    marginBottom: 20,
    padding: 20,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  checkboxContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 15,
  },
  checkbox: {
    width: 24,
    height: 24,
    borderWidth: 2,
    borderColor: '#ddd',
    borderRadius: 4,
    marginRight: 12,
    alignItems: 'center',
    justifyContent: 'center',
  },
  checkboxChecked: {
    backgroundColor: '#3498db',
    borderColor: '#3498db',
  },
  checkmark: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  termsText: {
    fontSize: 16,
    color: '#2c3e50',
    flex: 1,
  },
  buttonContainer: {
    marginHorizontal: 20,
    marginBottom: 20,
  },
  submitButton: {
    backgroundColor: '#3498db',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 15,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  resetButton: {
    backgroundColor: 'transparent',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#e74c3c',
  },
  resetButtonText: {
    color: '#e74c3c',
    fontSize: 16,
    fontWeight: '600',
  },
  historyContainer: {
    backgroundColor: 'white',
    marginHorizontal: 20,
    marginBottom: 20,
    padding: 20,
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  historyTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#2c3e50',
  },
  historyItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#ecf0f1',
  },
  historyField: {
    fontSize: 14,
    fontWeight: '500',
    color: '#2c3e50',
    flex: 1,
  },
  historyStatus: {
    fontSize: 12,
    flex: 2,
    textAlign: 'center',
  },
  historySuccess: {
    color: '#27ae60',
  },
  historyError: {
    color: '#e74c3c',
  },
  historyTime: {
    fontSize: 10,
    color: '#7f8c8d',
    flex: 1,
    textAlign: 'right',
  },
});

export default AdvancedValidationExample;
```

### **2. Hook de Validación Avanzada**

```javascript:src/hooks/useAdvancedValidation.js
import { useState, useCallback, useRef, useEffect } from 'react';

// Hook personalizado para validación avanzada
const useAdvancedValidation = (validationRules = {}, options = {}) => {
  const {
    validateOnChange = true,
    validateOnBlur = true,
    validateOnSubmit = true,
    debounceTime = 300,
    maxAsyncValidations = 5,
    retryAttempts = 3,
    retryDelay = 1000,
  } = options;

  // Estado de validación
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isValidating, setIsValidating] = useState({});
  const [validationHistory, setValidationHistory] = useState({});
  const [asyncValidationQueue, setAsyncValidationQueue] = useState([]);
  const [isProcessingQueue, setIsProcessingQueue] = useState(false);

  // Referencias para control
  const validationTimeoutsRef = useRef({});
  const retryCountersRef = useRef({});
  const lastValidationTimeRef = useRef({});

  // Función para validar campo individual
  const validateField = useCallback(async (field, value, context = {}) => {
    const rules = validationRules[field];
    if (!rules) return { isValid: true, error: null };

    setIsValidating(prev => ({ ...prev, [field]: true }));

    try {
      let error = null;
      let validationType = 'sync';
      let validationDetails = {};

      // Validación requerida
      if (rules.required && (!value || value.trim() === '')) {
        error = rules.requiredMessage || 'Este campo es requerido';
        validationType = 'required';
      }
      // Validación de longitud mínima
      else if (rules.minLength && value && value.length < rules.minLength) {
        error = rules.minLengthMessage || `Mínimo ${rules.minLength} caracteres`;
        validationType = 'length';
        validationDetails = { minLength: rules.minLength, actual: value.length };
      }
      // Validación de longitud máxima
      else if (rules.maxLength && value && value.length > rules.maxLength) {
        error = rules.maxLengthMessage || `Máximo ${rules.maxLength} caracteres`;
        validationType = 'length';
        validationDetails = { maxLength: rules.maxLength, actual: value.length };
      }
      // Validación de patrón
      else if (rules.pattern && value && !rules.pattern.test(value)) {
        error = rules.patternMessage || 'Formato inválido';
        validationType = 'pattern';
        validationDetails = { pattern: rules.pattern.toString() };
      }
      // Validación de rango numérico
      else if (rules.range && value) {
        const numValue = parseFloat(value);
        if (isNaN(numValue) || numValue < rules.range.min || numValue > rules.range.max) {
          error = rules.rangeMessage || `Debe estar entre ${rules.range.min} y ${rules.range.max}`;
          validationType = 'range';
          validationDetails = { min: rules.range.min, max: rules.range.max, actual: numValue };
        }
      }
      // Validación personalizada
      else if (rules.custom && value) {
        try {
          error = rules.custom(value, context);
          validationType = 'custom';
        } catch (customError) {
          error = 'Error en validación personalizada';
          validationType = 'custom_error';
          validationDetails = { originalError: customError.message };
        }
      }
      // Validación dependiente
      else if (rules.dependent && value) {
        const dependentField = rules.dependent.field;
        const dependentValue = context[dependentField];
        if (rules.dependent.condition && !rules.dependent.condition(value, dependentValue)) {
          error = rules.dependent.message || 'Validación dependiente falló';
          validationType = 'dependent';
          validationDetails = { dependentField, dependentValue };
        }
      }
      // Validación asíncrona
      else if (rules.async && value && !error) {
        try {
          error = await rules.async(value, context);
          validationType = 'async';
        } catch (asyncError) {
          error = 'Error en validación asíncrona';
          validationType = 'async_error';
          validationDetails = { originalError: asyncError.message };
        }
      }

      // Actualizar historial de validación
      const validationResult = {
        timestamp: Date.now(),
        value,
        error,
        type: validationType,
        details: validationDetails,
        success: !error,
        retryCount: retryCountersRef.current[field] || 0,
      };

      setValidationHistory(prev => ({
        ...prev,
        [field]: validationResult,
      }));

      // Actualizar errores
      setErrors(prev => ({
        ...prev,
        [field]: error
      }));

      // Actualizar contador de reintentos
      if (error && validationType === 'async_error') {
        retryCountersRef.current[field] = (retryCountersRef.current[field] || 0) + 1;
      } else {
        retryCountersRef.current[field] = 0;
      }

      setIsValidating(prev => ({ ...prev, [field]: false }));
      
      return { isValid: !error, error, type: validationType, details: validationDetails };

    } catch (validationError) {
      const error = 'Error inesperado en validación';
      const validationResult = {
        timestamp: Date.now(),
        value,
        error,
        type: 'unexpected_error',
        details: { originalError: validationError.message },
        success: false,
        retryCount: retryCountersRef.current[field] || 0,
      };

      setValidationHistory(prev => ({
        ...prev,
        [field]: validationResult,
      }));

      setErrors(prev => ({
        ...prev,
        [field]: error
      }));

      setIsValidating(prev => ({ ...prev, [field]: false }));
      
      return { isValid: false, error, type: 'unexpected_error' };
    }
  }, [validationRules]);

  // Función para validar campo con debounce
  const validateFieldWithDebounce = useCallback((field, value, context = {}) => {
    // Limpiar timeout previo
    if (validationTimeoutsRef.current[field]) {
      clearTimeout(validationTimeoutsRef.current[field]);
    }

    // Crear nuevo timeout
    validationTimeoutsRef.current[field] = setTimeout(() => {
      validateField(field, value, context);
    }, debounceTime);
  }, [validateField, debounceTime]);

  // Función para validar todo el formulario
  const validateForm = useCallback(async (formData) => {
    const fields = Object.keys(validationRules);
    const validationPromises = fields.map(field => 
      validateField(field, formData[field], formData)
    );
    
    const results = await Promise.all(validationPromises);
    const isValid = results.every(result => result.isValid);
    
    return { isValid, results };
  }, [validationRules, validateField]);

  // Función para validar campos dependientes
  const validateDependentFields = useCallback(async (field, value, formData) => {
    const dependentFields = Object.keys(validationRules).filter(fieldName => {
      const rules = validationRules[fieldName];
      return rules.dependent && rules.dependent.field === field;
    });

    if (dependentFields.length > 0) {
      const validationPromises = dependentFields.map(dependentField => 
        validateField(dependentField, formData[dependentField], formData)
      );
      
      await Promise.all(validationPromises);
    }
  }, [validationRules, validateField]);

  // Función para manejar cambio de campo
  const handleFieldChange = useCallback((field, value, formData) => {
    if (validateOnChange) {
      validateFieldWithDebounce(field, value, formData);
    }
  }, [validateOnChange, validateFieldWithDebounce]);

  // Función para manejar blur de campo
  const handleFieldBlur = useCallback(async (field, value, formData) => {
    if (validateOnBlur) {
      await validateField(field, value, formData);
      await validateDependentFields(field, value, formData);
    }
  }, [validateOnBlur, validateField, validateDependentFields]);

  // Función para agregar a cola de validación asíncrona
  const addToAsyncQueue = useCallback((field, value, context) => {
    setAsyncValidationQueue(prev => {
      const newQueue = [...prev];
      const existingIndex = newQueue.findIndex(item => item.field === field);
      
      if (existingIndex >= 0) {
        newQueue[existingIndex] = { field, value, context, timestamp: Date.now() };
      } else {
        newQueue.push({ field, value, context, timestamp: Date.now() });
      }
      
      // Mantener solo los últimos maxAsyncValidations
      return newQueue.slice(-maxAsyncValidations);
    });
  }, [maxAsyncValidations]);

  // Función para procesar cola de validación
  const processAsyncQueue = useCallback(async () => {
    if (isProcessingQueue || asyncValidationQueue.length === 0) return;

    setIsProcessingQueue(true);

    try {
      const currentItem = asyncValidationQueue[0];
      setAsyncValidationQueue(prev => prev.slice(1));

      await validateField(currentItem.field, currentItem.value, currentItem.context);
    } catch (error) {
      console.error('Error procesando cola de validación:', error);
    } finally {
      setIsProcessingQueue(false);
    }
  }, [isProcessingQueue, asyncValidationQueue, validateField]);

  // Efecto para procesar cola cuando cambia
  useEffect(() => {
    if (asyncValidationQueue.length > 0 && !isProcessingQueue) {
      processAsyncQueue();
    }
  }, [asyncValidationQueue, isProcessingQueue, processAsyncQueue]);

  // Función para obtener estadísticas de validación
  const getValidationStats = useCallback(() => {
    const totalFields = Object.keys(validationRules).length;
    const validFields = Object.keys(validationRules).filter(field => 
      !errors[field] && touched[field]
    ).length;
    const errorFields = Object.keys(errors).filter(field => errors[field]).length;
    const pendingFields = totalFields - validFields - errorFields;
    const asyncValidations = Object.keys(isValidating).filter(field => isValidating[field]).length;

    return {
      totalFields,
      validFields,
      errorFields,
      pendingFields,
      asyncValidations,
      queueLength: asyncValidationQueue.length,
      isProcessingQueue,
    };
  }, [validationRules, errors, touched, isValidating, asyncValidationQueue, isProcessingQueue]);

  // Función para limpiar validaciones
  const clearValidation = useCallback((field = null) => {
    if (field) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[field];
        return newErrors;
      });
      setValidationHistory(prev => {
        const newHistory = { ...prev };
        delete newHistory[field];
        return newHistory;
      });
      retryCountersRef.current[field] = 0;
    } else {
      setErrors({});
      setValidationHistory({});
      retryCountersRef.current = {};
    }
  }, []);

  // Función para resetear validación
  const resetValidation = useCallback(() => {
    setErrors({});
    setTouched({});
    setIsValidating({});
    setValidationHistory({});
    setAsyncValidationQueue([]);
    setIsProcessingQueue(false);
    retryCountersRef.current = {};
    Object.values(validationTimeoutsRef.current).forEach(timeout => {
      if (timeout) clearTimeout(timeout);
    });
    validationTimeoutsRef.current = {};
  }, []);

  // Efecto de limpieza
  useEffect(() => {
    return () => {
      Object.values(validationTimeoutsRef.current).forEach(timeout => {
        if (timeout) clearTimeout(timeout);
      });
    };
  }, []);

  return {
    // Estado
    errors,
    touched,
    isValidating,
    validationHistory,
    asyncValidationQueue,
    isProcessingQueue,
    
    // Funciones de validación
    validateField,
    validateFieldWithDebounce,
    validateForm,
    validateDependentFields,
    
    // Manejadores de eventos
    handleFieldChange,
    handleFieldBlur,
    
    // Control de cola asíncrona
    addToAsyncQueue,
    processAsyncQueue,
    
    // Utilidades
    getValidationStats,
    clearValidation,
    resetValidation,
    
    // Configuración
    options: {
      validateOnChange,
      validateOnBlur,
      validateOnSubmit,
      debounceTime,
      maxAsyncValidations,
      retryAttempts,
      retryDelay,
    },
  };
};

export default useAdvancedValidation;
```

---

## 🧪 Casos de Uso

### **Caso 1: Validación Personalizada**
```javascript
const rules = {
  username: {
    custom: (value) => {
      if (value.includes('admin')) {
        return 'No se permiten nombres reservados';
      }
      return null;
    }
  }
};
```

### **Caso 2: Validación Asíncrona**
```javascript
const rules = {
  email: {
    async: async (value) => {
      const response = await checkEmailAvailability(value);
      return response.available ? null : 'Email ya en uso';
    }
  }
};
```

### **Caso 3: Validación Dependiente**
```javascript
const rules = {
  confirmPassword: {
    dependent: {
      field: 'password',
      condition: (value, password) => value === password,
      message: 'Las contraseñas no coinciden'
    }
  }
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Validación de Formulario de Registro**
Crea un formulario con validaciones personalizadas para nombre, email y contraseña.

### **Ejercicio 2: Validación Asíncrona de Usuario**
Implementa validación asíncrona para verificar disponibilidad de nombre de usuario.

### **Ejercicio 3: Sistema de Validación Complejo**
Desarrolla un sistema que maneje validaciones dependientes y condicionales.

---

## 🚀 Proyecto de la Clase

### **App de Validación Avanzada**

Crea una aplicación que demuestre:
- **Validaciones personalizadas**: Reglas específicas del negocio
- **Validaciones asíncronas**: Verificación contra servidores
- **Validaciones dependientes**: Campos que dependen de otros
- **Sistema robusto**: Manejo de errores y reintentos

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Validación avanzada**: Reglas personalizadas y asíncronas
- **Estados complejos**: Gestión de múltiples tipos de validación
- **Validación dependiente**: Campos que se validan entre sí
- **Sistema robusto**: Manejo de errores y reintentos

### **Habilidades Desarrolladas:**
- ✅ Implementar validaciones personalizadas y asíncronas
- ✅ Gestionar estados complejos en formularios
- ✅ Crear sistemas de validación dependiente
- ✅ Desarrollar hooks de validación avanzada

### **Próximos Pasos:**
En el siguiente módulo aprenderemos sobre **Navegación y Routing**, que te permitirá crear aplicaciones con múltiples pantallas y navegación compleja.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Manejo de Eventos y Formularios](clase_4_manejo_eventos_formularios.md)
- **➡️ Siguiente Módulo**: [Navegación y Routing](../midLevel_1/README.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
