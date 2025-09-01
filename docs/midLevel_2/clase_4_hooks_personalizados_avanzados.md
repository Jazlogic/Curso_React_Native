# 📚 Clase 4: Hooks Personalizados Avanzados

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 3: Redux](clase_3_redux.md)
- **➡️ Siguiente**: [Clase 5: Persistencia de Datos](clase_5_persistencia_datos.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Crear hooks personalizados complejos y reutilizables
- Implementar hooks para gestión de formularios avanzados
- Dominar hooks para manejo de APIs y estado asíncrono
- Crear hooks para validación y manejo de errores
- Implementar hooks para optimización de performance

---

## 📚 Contenido Teórico

### **¿Qué son los Hooks Personalizados Avanzados?**

Los hooks personalizados avanzados son funciones que encapsulan lógica compleja y reutilizable, permitiendo compartir estado y comportamiento entre componentes de manera eficiente.

#### **Características principales:**
- **Reutilización**: Lógica compartida entre múltiples componentes
- **Composición**: Combinar múltiples hooks en uno solo
- **Abstracción**: Ocultar complejidad del componente
- **Testing**: Fácil de probar de manera aislada
- **Performance**: Optimización de re-renderizados

---

## 💻 Implementación Práctica

### **1. Hook para Gestión de Formularios Avanzados**

```javascript:src/hooks/useAdvancedForm.js
import { useState, useCallback, useRef, useEffect } from 'react';

// Hook para gestión avanzada de formularios
const useAdvancedForm = (initialState = {}, validationRules = {}) => {
  // Estado del formulario
  const [formData, setFormData] = useState(initialState);
  
  // Estado de validación
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  // Referencias para control
  const formRef = useRef(null);
  const submitAttempts = useRef(0);
  const lastSubmitTime = useRef(null);
  
  // Función para actualizar campo
  const updateField = useCallback((field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    setTouched(prev => ({ ...prev, [field]: true }));
    
    // Validar campo inmediatamente
    if (validationRules[field]) {
      const error = validateField(field, value);
      setErrors(prev => ({
        ...prev,
        [field]: error
      }));
    }
  }, [validationRules]);
  
  // Función para validar campo
  const validateField = useCallback((field, value) => {
    const rules = validationRules[field];
    if (!rules) return null;
    
    // Validaciones básicas
    if (rules.required && (!value || value.trim() === '')) {
      return rules.requiredMessage || 'Este campo es requerido';
    }
    
    if (rules.minLength && value && value.length < rules.minLength) {
      return rules.minLengthMessage || `Mínimo ${rules.minLength} caracteres`;
    }
    
    if (rules.pattern && value && !rules.pattern.test(value)) {
      return rules.patternMessage || 'Formato inválido';
    }
    
    // Validación personalizada
    if (rules.custom) {
      return rules.custom(value, formData);
    }
    
    return null;
  }, [validationRules, formData]);
  
  // Función para validar todo el formulario
  const validateForm = useCallback(() => {
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
    return isValid;
  }, [validationRules, formData, validateField]);
  
  // Función para enviar formulario
  const submitForm = useCallback(async (onSubmit) => {
    if (!validateForm()) {
      return { success: false, errors };
    }
    
    // Prevenir múltiples envíos
    if (isSubmitting) {
      return { success: false, error: 'Formulario ya se está enviando' };
    }
    
    // Verificar límite de intentos
    const now = Date.now();
    if (submitAttempts.current >= 3 && 
        now - lastSubmitTime.current < 60000) {
      return { 
        success: false, 
        error: 'Demasiados intentos. Espere 1 minuto.' 
      };
    }
    
    setIsSubmitting(true);
    submitAttempts.current += 1;
    lastSubmitTime.current = now;
    
    try {
      const result = await onSubmit(formData);
      return { success: true, data: result };
    } catch (error) {
      return { success: false, error: error.message };
    } finally {
      setIsSubmitting(false);
    }
  }, [validateForm, isSubmitting, formData, errors]);
  
  // Función para resetear formulario
  const resetForm = useCallback(() => {
    setFormData(initialState);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
    submitAttempts.current = 0;
    lastSubmitTime.current = null;
  }, [initialState]);
  
  // Función para establecer valores
  const setFormValues = useCallback((values) => {
    setFormData(prev => ({ ...prev, ...values }));
  }, []);
  
  // Función para obtener estadísticas
  const getFormStats = useCallback(() => ({
    isValid: Object.keys(errors).length === 0,
    isDirty: Object.keys(touched).length > 0,
    errorCount: Object.keys(errors).length,
    touchedCount: Object.keys(touched).length,
    submitAttempts: submitAttempts.current,
  }), [errors, touched]);
  
  // Efecto para validación automática
  useEffect(() => {
    if (Object.keys(touched).length > 0) {
      validateForm();
    }
  }, [formData, touched, validateForm]);
  
  return {
    // Estado
    formData,
    errors,
    touched,
    isSubmitting,
    
    // Funciones
    updateField,
    validateField,
    validateForm,
    submitForm,
    resetForm,
    setFormValues,
    getFormStats,
    
    // Referencias
    formRef,
  };
};

export default useAdvancedForm;
```

### **2. Hook para Gestión de APIs**

```javascript:src/hooks/useApi.js
import { useState, useCallback, useRef, useEffect } from 'react';

// Hook para gestión de APIs
const useApi = (apiFunction, options = {}) => {
  // Estado de la API
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  // Opciones del hook
  const {
    autoExecute = false,
    initialData = null,
    onSuccess,
    onError,
    retryCount = 3,
    retryDelay = 1000,
    cacheTime = 5 * 60 * 1000, // 5 minutos
  } = options;
  
  // Referencias para control
  const abortController = useRef(null);
  const retryTimeout = useRef(null);
  const cache = useRef(new Map());
  const lastFetchTime = useRef(0);
  
  // Función para ejecutar la API
  const execute = useCallback(async (params = {}) => {
    // Cancelar ejecución anterior
    if (abortController.current) {
      abortController.current.abort();
    }
    
    // Crear nuevo controlador de aborto
    abortController.current = new AbortController();
    
    // Verificar caché
    const cacheKey = JSON.stringify(params);
    const cachedData = cache.current.get(cacheKey);
    
    if (cachedData && Date.now() - lastFetchTime.current < cacheTime) {
      setData(cachedData);
      return { success: true, data: cachedData, fromCache: true };
    }
    
    setIsLoading(true);
    setError(null);
    
    try {
      const result = await apiFunction(params, abortController.current.signal);
      
      // Guardar en caché
      cache.current.set(cacheKey, result);
      lastFetchTime.current = Date.now();
      
      setData(result);
      setIsLoading(false);
      
      if (onSuccess) {
        onSuccess(result);
      }
      
      return { success: true, data: result };
    } catch (error) {
      if (error.name === 'AbortError') {
        return { success: false, error: 'Operación cancelada' };
      }
      
      setError(error.message);
      setIsLoading(false);
      
      if (onError) {
        onError(error);
      }
      
      return { success: false, error: error.message };
    }
  }, [apiFunction, cacheTime, onSuccess, onError]);
  
  // Función para reintentar
  const retry = useCallback(async (params = {}) => {
    let attempts = 0;
    
    const attempt = async () => {
      attempts++;
      const result = await execute(params);
      
      if (!result.success && attempts < retryCount) {
        retryTimeout.current = setTimeout(attempt, retryDelay * attempts);
      }
      
      return result;
    };
    
    return attempt();
  }, [execute, retryCount, retryDelay]);
  
  // Función para cancelar
  const cancel = useCallback(() => {
    if (abortController.current) {
      abortController.current.abort();
    }
    
    if (retryTimeout.current) {
      clearTimeout(retryTimeout.current);
    }
    
    setIsLoading(false);
  }, []);
  
  // Función para limpiar caché
  const clearCache = useCallback(() => {
    cache.current.clear();
    lastFetchTime.current = 0;
  }, []);
  
  // Función para actualizar datos
  const updateData = useCallback((updater) => {
    setData(prev => {
      if (typeof updater === 'function') {
        return updater(prev);
      }
      return updater;
    });
  }, []);
  
  // Efecto para ejecución automática
  useEffect(() => {
    if (autoExecute) {
      execute();
    }
    
    return () => {
      cancel();
    };
  }, [autoExecute, execute, cancel]);
  
  return {
    // Estado
    data,
    error,
    isLoading,
    
    // Funciones
    execute,
    retry,
    cancel,
    clearCache,
    updateData,
    
    // Utilidades
    isSuccess: !error && data !== null,
    isError: error !== null,
    isEmpty: data === null || (Array.isArray(data) && data.length === 0),
  };
};

export default useApi;
```

### **3. Hook para Validación Avanzada**

```javascript:src/hooks/useValidation.js
import { useState, useCallback, useRef, useEffect } from 'react';

// Hook para validación avanzada
const useValidation = (validationSchema = {}) => {
  // Estado de validación
  const [errors, setErrors] = useState({});
  const [warnings, setWarnings] = useState({});
  const [isValidating, setIsValidating] = useState(false);
  
  // Referencias para control
  const validationQueue = useRef([]);
  const validationTimeout = useRef(null);
  const lastValidationTime = useRef(0);
  
  // Función para validar campo
  const validateField = useCallback(async (field, value, context = {}) => {
    const rules = validationSchema[field];
    if (!rules) return null;
    
    const fieldErrors = [];
    const fieldWarnings = [];
    
    // Validaciones síncronas
    if (rules.required && (!value || value.trim() === '')) {
      fieldErrors.push(rules.requiredMessage || 'Campo requerido');
    }
    
    if (rules.minLength && value && value.length < rules.minLength) {
      fieldErrors.push(rules.minLengthMessage || `Mínimo ${rules.minLength} caracteres`);
    }
    
    if (rules.maxLength && value && value.length > rules.maxLength) {
      fieldErrors.push(rules.maxLengthMessage || `Máximo ${rules.maxLength} caracteres`);
    }
    
    if (rules.pattern && value && !rules.pattern.test(value)) {
      fieldErrors.push(rules.patternMessage || 'Formato inválido');
    }
    
    if (rules.custom && value) {
      try {
        const customResult = await rules.custom(value, context);
        if (customResult && typeof customResult === 'string') {
          fieldErrors.push(customResult);
        } else if (customResult && customResult.warning) {
          fieldWarnings.push(customResult.warning);
        }
      } catch (error) {
        fieldErrors.push('Error de validación personalizada');
      }
    }
    
    // Validaciones asíncronas
    if (rules.async && value) {
      try {
        const asyncResult = await rules.async(value, context);
        if (asyncResult && typeof asyncResult === 'string') {
          fieldErrors.push(asyncResult);
        }
      } catch (error) {
        fieldErrors.push('Error de validación asíncrona');
      }
    }
    
    return {
      errors: fieldErrors,
      warnings: fieldWarnings,
      isValid: fieldErrors.length === 0,
    };
  }, [validationSchema]);
  
  // Función para validar múltiples campos
  const validateFields = useCallback(async (fields, values, context = {}) => {
    setIsValidating(true);
    
    const validationPromises = fields.map(async (field) => {
      const result = await validateField(field, values[field], context);
      return { field, ...result };
    });
    
    const results = await Promise.all(validationPromises);
    
    const newErrors = {};
    const newWarnings = {};
    
    results.forEach(({ field, errors: fieldErrors, warnings: fieldWarnings }) => {
      if (fieldErrors.length > 0) {
        newErrors[field] = fieldErrors;
      }
      if (fieldWarnings.length > 0) {
        newWarnings[field] = fieldWarnings;
      }
    });
    
    setErrors(newErrors);
    setWarnings(newWarnings);
    setIsValidating(false);
    
    return {
      errors: newErrors,
      warnings: newWarnings,
      isValid: Object.keys(newErrors).length === 0,
    };
  }, [validateField]);
  
  // Función para validar formulario completo
  const validateForm = useCallback(async (values, context = {}) => {
    const fields = Object.keys(validationSchema);
    return await validateFields(fields, values, context);
  }, [validationSchema, validateFields]);
  
  // Función para validación con debounce
  const validateWithDebounce = useCallback((field, value, context = {}, delay = 300) => {
    // Limpiar timeout anterior
    if (validationTimeout.current) {
      clearTimeout(validationTimeout.current);
    }
    
    // Agregar a cola de validación
    validationQueue.current.push({ field, value, context });
    
    // Ejecutar validación después del delay
    validationTimeout.current = setTimeout(async () => {
      const queue = [...validationQueue.current];
      validationQueue.current = [];
      
      for (const item of queue) {
        const result = await validateField(item.field, item.value, item.context);
        
        setErrors(prev => ({
          ...prev,
          [item.field]: result.errors
        }));
        
        setWarnings(prev => ({
          ...prev,
          [item.field]: result.warnings
        }));
      }
    }, delay);
  }, [validateField]);
  
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
  
  // Función para limpiar warnings
  const clearWarnings = useCallback((field = null) => {
    if (field) {
      setWarnings(prev => {
        const newWarnings = { ...prev };
        delete newWarnings[field];
        return newWarnings;
      });
    } else {
      setWarnings({});
    }
  }, []);
  
  // Función para obtener estadísticas
  const getValidationStats = useCallback(() => ({
    errorCount: Object.keys(errors).length,
    warningCount: Object.keys(warnings).length,
    totalErrors: Object.values(errors).flat().length,
    totalWarnings: Object.values(warnings).flat().length,
    isValid: Object.keys(errors).length === 0,
    hasWarnings: Object.keys(warnings).length > 0,
  }), [errors, warnings]);
  
  // Efecto de limpieza
  useEffect(() => {
    return () => {
      if (validationTimeout.current) {
        clearTimeout(validationTimeout.current);
      }
    };
  }, []);
  
  return {
    // Estado
    errors,
    warnings,
    isValidating,
    
    // Funciones
    validateField,
    validateFields,
    validateForm,
    validateWithDebounce,
    clearErrors,
    clearWarnings,
    getValidationStats,
    
    // Utilidades
    hasErrors: Object.keys(errors).length > 0,
    hasWarnings: Object.keys(warnings).length > 0,
    isFormValid: Object.keys(errors).length === 0,
  };
};

export default useValidation;
```

---

## 🧪 Casos de Uso

### **Caso 1: Hook de Formulario Avanzado**
```javascript
const { formData, errors, submitForm, updateField } = useAdvancedForm(
  { name: '', email: '' },
  {
    name: { required: true, minLength: 2 },
    email: { required: true, pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ }
  }
);
```

### **Caso 2: Hook de API**
```javascript
const { data, error, isLoading, execute } = useApi(
  fetchUserData,
  { autoExecute: true, retryCount: 3 }
);
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Hook de Autenticación**
Crea un hook personalizado para gestionar autenticación completa.

### **Ejercicio 2: Hook de Notificaciones**
Implementa un hook para manejo de notificaciones push y locales.

### **Ejercicio 3: Hook de Geolocalización**
Desarrolla un hook para gestión de ubicación del usuario.

---

## 🚀 Proyecto de la Clase

### **App con Hooks Avanzados**

Crea una aplicación que demuestre:
- **Formularios complejos**: Con validación en tiempo real
- **Gestión de APIs**: Con manejo de errores y reintentos
- **Validación avanzada**: Con reglas personalizadas y asíncronas
- **Performance**: Con hooks optimizados

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Hooks personalizados**: Lógica reutilizable y composable
- **Validación avanzada**: Reglas síncronas y asíncronas
- **Gestión de APIs**: Con manejo de estado y errores
- **Performance**: Optimización de re-renderizados

### **Habilidades Desarrolladas:**
- ✅ Crear hooks personalizados complejos
- ✅ Implementar validación avanzada
- ✅ Gestionar APIs de manera eficiente
- ✅ Optimizar performance de componentes

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Persistencia de Datos**, que te permitirá almacenar información localmente en React Native.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Redux](clase_3_redux.md)
- **➡️ Siguiente Clase**: [Persistencia de Datos](clase_5_persistencia_datos.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
