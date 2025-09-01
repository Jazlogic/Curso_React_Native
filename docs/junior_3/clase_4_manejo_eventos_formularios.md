# 📚 Clase 4: Manejo de Eventos y Formularios

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 3: useRef y Referencias](clase_3_useRef_referencias.md)
- **➡️ Siguiente**: [Clase 5: Validación y Estado Complejo](clase_5_validacion_estado_complejo.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el manejo de eventos en React Native
- Dominar la creación y gestión de formularios
- Implementar validación en tiempo real
- Gestionar diferentes tipos de entrada del usuario
- Crear interfaces interactivas y responsivas

---

## 📚 Contenido Teórico

### **¿Qué son los Eventos en React Native?**

Los eventos son acciones que ocurren cuando el usuario interactúa con la aplicación. React Native proporciona un sistema de eventos unificado que funciona de manera similar en iOS y Android.

#### **Tipos de eventos principales:**
- **Touch Events**: onPress, onLongPress, onPressIn, onPressOut
- **Text Input Events**: onChangeText, onFocus, onBlur, onSubmitEditing
- **Scroll Events**: onScroll, onScrollBeginDrag, onScrollEndDrag
- **Layout Events**: onLayout, onContentSizeChange

---

## 💻 Implementación Práctica

### **1. Manejo Básico de Eventos**

```javascript:src/components/BasicEventHandling.js
import React, { useState } from 'react';
import { 
  View, 
  Text, 
  TouchableOpacity, 
  TextInput, 
  StyleSheet, 
  ScrollView,
  Alert 
} from 'react-native';

// Componente que demuestra manejo básico de eventos
const BasicEventHandling = () => {
  // Estado para diferentes tipos de eventos
  const [touchCount, setTouchCount] = useState(0);
  const [longPressCount, setLongPressCount] = useState(0);
  const [inputValue, setInputValue] = useState('');
  const [focusCount, setFocusCount] = useState(0);
  const [scrollPosition, setScrollPosition] = useState(0);
  const [lastEvent, setLastEvent] = useState('Ninguno');

  // Función para manejar toque simple
  const handlePress = () => {
    setTouchCount(prev => prev + 1);
    setLastEvent('Toque simple');
    console.log('Toque simple detectado');
  };

  // Función para manejar toque largo
  const handleLongPress = () => {
    setLongPressCount(prev => prev + 1);
    setLastEvent('Toque largo');
    Alert.alert('Toque Largo', '¡Has mantenido presionado!');
    console.log('Toque largo detectado');
  };

  // Función para manejar entrada de texto
  const handleTextChange = (text) => {
    setInputValue(text);
    setLastEvent('Cambio de texto');
    console.log('Texto cambiado:', text);
  };

  // Función para manejar focus del input
  const handleFocus = () => {
    setFocusCount(prev => prev + 1);
    setLastEvent('Input enfocado');
    console.log('Input enfocado');
  };

  // Función para manejar blur del input
  const handleBlur = () => {
    setLastEvent('Input desenfocado');
    console.log('Input desenfocado');
  };

  // Función para manejar envío del input
  const handleSubmit = () => {
    setLastEvent('Texto enviado');
    Alert.alert('Envío', `Texto enviado: ${inputValue}`);
    console.log('Texto enviado:', inputValue);
  };

  // Función para manejar scroll
  const handleScroll = (event) => {
    const position = event.nativeEvent.contentOffset.y;
    setScrollPosition(position);
    setLastEvent('Scroll');
  };

  // Función para resetear contadores
  const resetCounters = () => {
    setTouchCount(0);
    setLongPressCount(0);
    setFocusCount(0);
    setLastEvent('Contadores reseteados');
  };

  // Función para limpiar input
  const clearInput = () => {
    setInputValue('');
    setLastEvent('Input limpiado');
  };

  return (
    <ScrollView 
      style={styles.container}
      onScroll={handleScroll}
      scrollEventThrottle={16}
    >
      <View style={styles.header}>
        <Text style={styles.title}>Manejo de Eventos</Text>
        <Text style={styles.subtitle}>
          Demostración de diferentes tipos de eventos en React Native
        </Text>
      </View>

      {/* Sección de Eventos de Toque */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>👆 Eventos de Toque</Text>
        
        <View style={styles.eventRow}>
          <TouchableOpacity 
            style={styles.eventButton} 
            onPress={handlePress}
            activeOpacity={0.7}
          >
            <Text style={styles.buttonText}>Toque Simple</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={styles.eventButton} 
            onLongPress={handleLongPress}
            delayLongPress={1000}
            activeOpacity={0.7}
          >
            <Text style={styles.buttonText}>Toque Largo</Text>
          </TouchableOpacity>
        </View>
        
        <View style={styles.counterRow}>
          <Text style={styles.counterText}>
            Toques: {touchCount}
          </Text>
          <Text style={styles.counterText}>
            Toques Largos: {longPressCount}
          </Text>
        </View>
      </View>

      {/* Sección de Eventos de Input */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>⌨️ Eventos de Input</Text>
        
        <TextInput
          style={styles.input}
          placeholder="Escribe algo aquí..."
          value={inputValue}
          onChangeText={handleTextChange}
          onFocus={handleFocus}
          onBlur={handleBlur}
          onSubmitEditing={handleSubmit}
          returnKeyType="send"
          blurOnSubmit={false}
        />
        
        <View style={styles.inputControls}>
          <TouchableOpacity 
            style={styles.controlButton} 
            onPress={clearInput}
          >
            <Text style={styles.buttonText}>Limpiar</Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={styles.controlButton} 
            onPress={handleSubmit}
          >
            <Text style={styles.buttonText}>Enviar</Text>
          </TouchableOpacity>
        </View>
        
        <Text style={styles.counterText}>
          Veces enfocado: {focusCount}
        </Text>
      </View>

      {/* Sección de Eventos de Scroll */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>📜 Eventos de Scroll</Text>
        
        <Text style={styles.scrollInfo}>
          Posición de scroll: {scrollPosition.toFixed(0)}px
        </Text>
        
        <Text style={styles.scrollHint}>
          Desliza hacia arriba y abajo para ver el evento de scroll
        </Text>
      </View>

      {/* Sección de Información de Eventos */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>ℹ️ Información de Eventos</Text>
        
        <Text style={styles.eventInfo}>
          Último evento: {lastEvent}
        </Text>
        
        <TouchableOpacity 
          style={styles.resetButton} 
          onPress={resetCounters}
        >
          <Text style={styles.buttonText}>Resetear Contadores</Text>
        </TouchableOpacity>
      </View>

      {/* Espacio adicional para scroll */}
      <View style={styles.spacer} />
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
  section: {
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
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#2c3e50',
  },
  eventRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 15,
  },
  eventButton: {
    backgroundColor: '#3498db',
    padding: 15,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  counterRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  counterText: {
    fontSize: 16,
    color: '#7f8c8d',
    fontWeight: '500',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    marginBottom: 15,
    fontSize: 16,
  },
  inputControls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 15,
  },
  controlButton: {
    backgroundColor: '#27ae60',
    padding: 12,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  scrollInfo: {
    fontSize: 18,
    textAlign: 'center',
    color: '#3498db',
    fontWeight: 'bold',
    marginBottom: 10,
  },
  scrollHint: {
    fontSize: 14,
    textAlign: 'center',
    color: '#7f8c8d',
    fontStyle: 'italic',
  },
  eventInfo: {
    fontSize: 16,
    textAlign: 'center',
    color: '#e74c3c',
    fontWeight: '500',
    marginBottom: 15,
    padding: 10,
    backgroundColor: '#f8f9fa',
    borderRadius: 8,
  },
  resetButton: {
    backgroundColor: '#e74c3c',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  spacer: {
    height: 100,
  },
});

export default BasicEventHandling;
```

### **2. Formulario Completo con Validación**

```javascript:src/components/AdvancedFormExample.js
import React, { useState, useRef } from 'react';
import { 
  View, 
  Text, 
  TextInput, 
  TouchableOpacity, 
  StyleSheet, 
  ScrollView,
  Alert,
  KeyboardAvoidingView,
  Platform 
} from 'react-native';

// Componente que demuestra formulario avanzado con validación
const AdvancedFormExample = () => {
  // Referencias para inputs
  const nameInputRef = useRef(null);
  const emailInputRef = useRef(null);
  const phoneInputRef = useRef(null);
  const passwordInputRef = useRef(null);
  const confirmPasswordInputRef = useRef(null);

  // Estado del formulario
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    phone: '',
    password: '',
    confirmPassword: '',
    acceptTerms: false,
  });

  // Estado de validación
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Estado para mostrar/ocultar contraseñas
  const [showPassword, setShowPassword] = useState(false);
  const [showConfirmPassword, setShowConfirmPassword] = useState(false);

  // Función para actualizar campo
  const updateField = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    setTouched(prev => ({ ...prev, [field]: true }));
    
    // Validar campo inmediatamente
    validateField(field, value);
  };

  // Función para validar campo individual
  const validateField = (field, value) => {
    let error = '';

    switch (field) {
      case 'name':
        if (!value.trim()) {
          error = 'El nombre es requerido';
        } else if (value.trim().length < 2) {
          error = 'El nombre debe tener al menos 2 caracteres';
        }
        break;

      case 'email':
        if (!value.trim()) {
          error = 'El email es requerido';
        } else if (!/\S+@\S+\.\S+/.test(value)) {
          error = 'El email no es válido';
        }
        break;

      case 'phone':
        if (value && !/^[\+]?[0-9\s\-\(\)]{10,}$/.test(value)) {
          error = 'El teléfono no es válido';
        }
        break;

      case 'password':
        if (!value) {
          error = 'La contraseña es requerida';
        } else if (value.length < 8) {
          error = 'La contraseña debe tener al menos 8 caracteres';
        } else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          error = 'La contraseña debe contener mayúsculas, minúsculas y números';
        }
        break;

      case 'confirmPassword':
        if (!value) {
          error = 'Confirma tu contraseña';
        } else if (value !== formData.password) {
          error = 'Las contraseñas no coinciden';
        }
        break;

      default:
        break;
    }

    setErrors(prev => ({
      ...prev,
      [field]: error
    }));

    return error === '';
  };

  // Función para validar todo el formulario
  const validateForm = () => {
    const newErrors = {};
    let isValid = true;

    Object.keys(formData).forEach(field => {
      if (field !== 'acceptTerms') {
        const fieldValid = validateField(field, formData[field]);
        if (!fieldValid) {
          isValid = false;
        }
      }
    });

    if (!formData.acceptTerms) {
      newErrors.acceptTerms = 'Debes aceptar los términos y condiciones';
      isValid = false;
    }

    setErrors(newErrors);
    return isValid;
  };

  // Función para enviar formulario
  const handleSubmit = async () => {
    if (!validateForm()) {
      Alert.alert('Error', 'Por favor corrige los errores del formulario');
      return;
    }

    setIsSubmitting(true);

    try {
      // Simular envío a servidor
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      Alert.alert(
        'Éxito', 
        'Formulario enviado correctamente',
        [
          {
            text: 'OK',
            onPress: () => resetForm()
          }
        ]
      );
    } catch (error) {
      Alert.alert('Error', 'Hubo un problema al enviar el formulario');
    } finally {
      setIsSubmitting(false);
    }
  };

  // Función para resetear formulario
  const resetForm = () => {
    setFormData({
      name: '',
      email: '',
      phone: '',
      password: '',
      confirmPassword: '',
      acceptTerms: false,
    });
    setErrors({});
    setTouched({});
    setShowPassword(false);
    setShowConfirmPassword(false);
  };

  // Función para alternar términos
  const toggleTerms = () => {
    setFormData(prev => ({ ...prev, acceptTerms: !prev.acceptTerms }));
    if (errors.acceptTerms) {
      setErrors(prev => ({ ...prev, acceptTerms: '' }));
    }
  };

  // Función para obtener estilo del input
  const getInputStyle = (field) => {
    const baseStyle = styles.input;
    const errorStyle = errors[field] && touched[field] ? styles.inputError : null;
    const successStyle = !errors[field] && touched[field] && formData[field] ? styles.inputSuccess : null;
    
    return [baseStyle, errorStyle, successStyle];
  };

  // Función para obtener mensaje de error
  const getErrorMessage = (field) => {
    if (errors[field] && touched[field]) {
      return <Text style={styles.errorText}>{errors[field]}</Text>;
    }
    return null;
  };

  return (
    <KeyboardAvoidingView 
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <ScrollView style={styles.scrollView}>
        <View style={styles.header}>
          <Text style={styles.title}>Formulario Avanzado</Text>
          <Text style={styles.subtitle}>
            Con validación en tiempo real y manejo de errores
          </Text>
        </View>

        {/* Campo Nombre */}
        <View style={styles.fieldContainer}>
          <Text style={styles.label}>Nombre *</Text>
          <TextInput
            ref={nameInputRef}
            style={getInputStyle('name')}
            placeholder="Ingresa tu nombre completo"
            value={formData.name}
            onChangeText={(text) => updateField('name', text)}
            onBlur={() => validateField('name', formData.name)}
            returnKeyType="next"
            onSubmitEditing={() => emailInputRef.current?.focus()}
          />
          {getErrorMessage('name')}
        </View>

        {/* Campo Email */}
        <View style={styles.fieldContainer}>
          <Text style={styles.label}>Email *</Text>
          <TextInput
            ref={emailInputRef}
            style={getInputStyle('email')}
            placeholder="tu@email.com"
            value={formData.email}
            onChangeText={(text) => updateField('email', text)}
            onBlur={() => validateField('email', formData.email)}
            keyboardType="email-address"
            autoCapitalize="none"
            returnKeyType="next"
            onSubmitEditing={() => phoneInputRef.current?.focus()}
          />
          {getErrorMessage('email')}
        </View>

        {/* Campo Teléfono */}
        <View style={styles.fieldContainer}>
          <Text style={styles.label}>Teléfono</Text>
          <TextInput
            ref={phoneInputRef}
            style={getInputStyle('phone')}
            placeholder="+1 (555) 123-4567"
            value={formData.phone}
            onChangeText={(text) => updateField('phone', text)}
            onBlur={() => validateField('phone', formData.phone)}
            keyboardType="phone-pad"
            returnKeyType="next"
            onSubmitEditing={() => passwordInputRef.current?.focus()}
          />
          {getErrorMessage('phone')}
        </View>

        {/* Campo Contraseña */}
        <View style={styles.fieldContainer}>
          <Text style={styles.label}>Contraseña *</Text>
          <TextInput
            ref={passwordInputRef}
            style={getInputStyle('password')}
            placeholder="Mínimo 8 caracteres"
            value={formData.password}
            onChangeText={(text) => updateField('password', text)}
            onBlur={() => validateField('password', formData.password)}
            secureTextEntry={!showPassword}
            returnKeyType="next"
            onSubmitEditing={() => confirmPasswordInputRef.current?.focus()}
          />
          <TouchableOpacity 
            style={styles.showPasswordButton}
            onPress={() => setShowPassword(!showPassword)}
          >
            <Text style={styles.showPasswordText}>
              {showPassword ? 'Ocultar' : 'Mostrar'}
            </Text>
          </TouchableOpacity>
          {getErrorMessage('password')}
        </View>

        {/* Campo Confirmar Contraseña */}
        <View style={styles.fieldContainer}>
          <Text style={styles.label}>Confirmar Contraseña *</Text>
          <TextInput
            ref={confirmPasswordInputRef}
            style={getInputStyle('confirmPassword')}
            placeholder="Repite tu contraseña"
            value={formData.confirmPassword}
            onChangeText={(text) => updateField('confirmPassword', text)}
            onBlur={() => validateField('confirmPassword', formData.confirmPassword)}
            secureTextEntry={!showConfirmPassword}
            returnKeyType="done"
            onSubmitEditing={handleSubmit}
          />
          <TouchableOpacity 
            style={styles.showPasswordButton}
            onPress={() => setShowConfirmPassword(!showConfirmPassword)}
          >
            <Text style={styles.showPasswordText}>
              {showConfirmPassword ? 'Ocultar' : 'Mostrar'}
            </Text>
          </TouchableOpacity>
          {getErrorMessage('confirmPassword')}
        </View>

        {/* Términos y Condiciones */}
        <View style={styles.termsContainer}>
          <TouchableOpacity 
            style={styles.checkboxContainer}
            onPress={toggleTerms}
          >
            <View style={[
              styles.checkbox, 
              formData.acceptTerms && styles.checkboxChecked
            ]}>
              {formData.acceptTerms && <Text style={styles.checkmark}>✓</Text>}
            </View>
            <Text style={styles.termsText}>
              Acepto los términos y condiciones
            </Text>
          </TouchableOpacity>
          {errors.acceptTerms && (
            <Text style={styles.errorText}>{errors.acceptTerms}</Text>
          )}
        </View>

        {/* Botones de Acción */}
        <View style={styles.buttonContainer}>
          <TouchableOpacity 
            style={[styles.submitButton, isSubmitting && styles.disabledButton]}
            onPress={handleSubmit}
            disabled={isSubmitting}
          >
            <Text style={styles.buttonText}>
              {isSubmitting ? 'Enviando...' : 'Enviar Formulario'}
            </Text>
          </TouchableOpacity>
          
          <TouchableOpacity 
            style={styles.resetButton}
            onPress={resetForm}
          >
            <Text style={styles.resetButtonText}>Limpiar Formulario</Text>
          </TouchableOpacity>
        </View>

        {/* Resumen de Validación */}
        <View style={styles.validationSummary}>
          <Text style={styles.summaryTitle}>Resumen de Validación:</Text>
          <Text style={styles.summaryText}>
            Campos válidos: {Object.keys(formData).filter(field => 
              field === 'acceptTerms' ? formData[field] : !errors[field] && touched[field]
            ).length} / {Object.keys(formData).length}
          </Text>
        </View>
      </ScrollView>
    </KeyboardAvoidingView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  scrollView: {
    flex: 1,
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
  label: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 8,
    color: '#2c3e50',
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
  errorText: {
    color: '#e74c3c',
    fontSize: 14,
    marginTop: 5,
  },
  showPasswordButton: {
    position: 'absolute',
    right: 12,
    top: 40,
  },
  showPasswordText: {
    color: '#3498db',
    fontSize: 14,
    fontWeight: '500',
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
  disabledButton: {
    backgroundColor: '#bdc3c7',
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
  validationSummary: {
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
  summaryTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 8,
    color: '#2c3e50',
  },
  summaryText: {
    fontSize: 14,
    color: '#7f8c8d',
  },
});

export default AdvancedFormExample;
```

### **3. Hook Personalizado para Formularios**

```javascript:src/hooks/useForm.js
import { useState, useCallback, useRef, useEffect } from 'react';

// Hook personalizado para gestión avanzada de formularios
const useForm = (initialState = {}, validationRules = {}, options = {}) => {
  const {
    validateOnChange = true, // Validar al cambiar
    validateOnBlur = true,   // Validar al perder focus
    validateOnSubmit = true, // Validar al enviar
    debounceValidation = 300, // Delay para validación
  } = options;

  // Estado del formulario
  const [formData, setFormData] = useState(initialState);
  
  // Estado de validación
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isValid, setIsValid] = useState(false);
  
  // Referencias para control
  const validationTimeoutRef = useRef(null);
  const submitAttemptsRef = useRef(0);
  const lastSubmitTimeRef = useRef(0);

  // Función para actualizar campo
  const updateField = useCallback((field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    setTouched(prev => ({ ...prev, [field]: true }));
    
    // Validar campo si está habilitado
    if (validateOnChange) {
      if (debounceValidation > 0) {
        // Validación con debounce
        if (validationTimeoutRef.current) {
          clearTimeout(validationTimeoutRef.current);
        }
        
        validationTimeoutRef.current = setTimeout(() => {
          validateField(field, value);
        }, debounceValidation);
      } else {
        // Validación inmediata
        validateField(field, value);
      }
    }
  }, [validateOnChange, debounceValidation]);

  // Función para validar campo individual
  const validateField = useCallback((field, value) => {
    const rules = validationRules[field];
    if (!rules) return true;

    let error = '';

    // Validación requerida
    if (rules.required && (!value || value.trim() === '')) {
      error = rules.requiredMessage || 'Este campo es requerido';
    }

    // Validación de longitud mínima
    if (rules.minLength && value && value.length < rules.minLength) {
      error = rules.minLengthMessage || `Mínimo ${rules.minLength} caracteres`;
    }

    // Validación de longitud máxima
    if (rules.maxLength && value && value.length > rules.maxLength) {
      error = rules.maxLengthMessage || `Máximo ${rules.maxLength} caracteres`;
    }

    // Validación de patrón
    if (rules.pattern && value && !rules.pattern.test(value)) {
      error = rules.patternMessage || 'Formato inválido';
    }

    // Validación personalizada
    if (rules.custom && value) {
      const customError = rules.custom(value, formData);
      if (customError) {
        error = customError;
      }
    }

    // Validación asíncrona (si existe)
    if (rules.async && value && !error) {
      // Aquí se implementaría validación asíncrona
      // Por ahora solo simulamos
    }

    setErrors(prev => ({
      ...prev,
      [field]: error
    }));

    return error === '';
  }, [validationRules, formData]);

  // Función para validar todo el formulario
  const validateForm = useCallback(() => {
    const newErrors = {};
    let formValid = true;

    Object.keys(validationRules).forEach(field => {
      const fieldValid = validateField(field, formData[field]);
      if (!fieldValid) {
        formValid = false;
      }
    });

    setIsValid(formValid);
    return formValid;
  }, [validationRules, validateField, formData]);

  // Función para manejar blur de campo
  const handleBlur = useCallback((field) => {
    if (validateOnBlur) {
      validateField(field, formData[field]);
    }
  }, [validateOnBlur, validateField, formData]);

  // Función para enviar formulario
  const submitForm = useCallback(async (onSubmit) => {
    if (validateOnSubmit && !validateForm()) {
      return { success: false, errors };
    }

    // Prevenir múltiples envíos
    if (isSubmitting) {
      return { success: false, error: 'Formulario ya se está enviando' };
    }

    // Verificar límite de intentos
    const now = Date.now();
    if (submitAttemptsRef.current >= 3 && 
        now - lastSubmitTimeRef.current < 60000) {
      return { 
        success: false, 
        error: 'Demasiados intentos. Espere 1 minuto.' 
      };
    }

    setIsSubmitting(true);
    submitAttemptsRef.current += 1;
    lastSubmitTimeRef.current = now;

    try {
      const result = await onSubmit(formData);
      return { success: true, data: result };
    } catch (error) {
      return { success: false, error: error.message };
    } finally {
      setIsSubmitting(false);
    }
  }, [validateOnSubmit, validateForm, errors, isSubmitting, formData]);

  // Función para resetear formulario
  const resetForm = useCallback(() => {
    setFormData(initialState);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
    setIsValid(false);
    submitAttemptsRef.current = 0;
    lastSubmitTimeRef.current = 0;
  }, [initialState]);

  // Función para establecer valores
  const setFormValues = useCallback((values) => {
    setFormData(prev => ({ ...prev, ...values }));
  }, []);

  // Función para establecer errores
  const setFieldError = useCallback((field, error) => {
    setErrors(prev => ({ ...prev, [field]: error }));
  }, []);

  // Función para limpiar errores
  const clearErrors = useCallback((field = null) => {
    if (field) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[field];
        return newErrors;
      });
    } else {
      setErrors({});
    }
  }, []);

  // Función para obtener estadísticas
  const getFormStats = useCallback(() => ({
    isValid,
    isSubmitting,
    errorCount: Object.keys(errors).length,
    touchedCount: Object.keys(touched).length,
    submitAttempts: submitAttemptsRef.current,
    hasErrors: Object.keys(errors).length > 0,
    isDirty: Object.keys(touched).length > 0,
  }), [isValid, isSubmitting, errors, touched]);

  // Efecto para validación automática
  useEffect(() => {
    if (Object.keys(touched).length > 0) {
      validateForm();
    }
  }, [formData, touched, validateForm]);

  // Efecto de limpieza
  useEffect(() => {
    return () => {
      if (validationTimeoutRef.current) {
        clearTimeout(validationTimeoutRef.current);
      }
    };
  }, []);

  return {
    // Estado
    formData,
    errors,
    touched,
    isSubmitting,
    isValid,
    
    // Funciones
    updateField,
    validateField,
    validateForm,
    submitForm,
    resetForm,
    setFormValues,
    setFieldError,
    clearErrors,
    handleBlur,
    
    // Utilidades
    getFormStats,
  };
};

export default useForm;
```

---

## 🧪 Casos de Uso

### **Caso 1: Evento Simple**
```javascript
const handlePress = () => {
  console.log('Botón presionado');
};
```

### **Caso 2: Formulario Básico**
```javascript
const { formData, updateField, submitForm } = useForm(
  { name: '', email: '' },
  { name: { required: true } }
);
```

### **Caso 3: Validación en Tiempo Real**
```javascript
const { updateField, handleBlur, errors } = useForm(
  initialState,
  validationRules,
  { validateOnChange: true, validateOnBlur: true }
);
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Formulario de Contacto**
Crea un formulario de contacto con validación de email y teléfono.

### **Ejercicio 2: Sistema de Login**
Implementa un formulario de login con validación de contraseña.

### **Ejercicio 3: Formulario de Registro**
Desarrolla un formulario de registro completo con múltiples validaciones.

---

## 🚀 Proyecto de la Clase

### **App de Formularios Interactivos**

Crea una aplicación que demuestre:
- **Manejo de eventos**: Toques, inputs, scroll
- **Formularios complejos**: Múltiples campos y validaciones
- **Validación en tiempo real**: Feedback inmediato al usuario
- **Hooks personalizados**: Lógica reutilizable para formularios

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Eventos**: Sistema unificado para iOS y Android
- **Formularios**: Gestión de estado y validación
- **Validación**: Reglas y feedback en tiempo real
- **Hooks personalizados**: Lógica reutilizable para formularios

### **Habilidades Desarrolladas:**
- ✅ Manejar diferentes tipos de eventos
- ✅ Crear formularios complejos y validados
- ✅ Implementar validación en tiempo real
- ✅ Desarrollar hooks personalizados para formularios

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Validación y Estado Complejo**, que te permitirá implementar sistemas de validación avanzados y manejar estados complejos en formularios.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [useRef y Referencias](clase_3_useRef_referencias.md)
- **➡️ Siguiente Clase**: [Validación y Estado Complejo](clase_5_validacion_estado_complejo.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
