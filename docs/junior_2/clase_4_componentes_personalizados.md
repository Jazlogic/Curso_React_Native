# 📚 Clase 4: Componentes Personalizados

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 3: Flexbox y Layout](clase_3_flexbox_layout.md)
- **➡️ Siguiente**: [Módulo 3: Navegación y Routing](../midLevel_1/README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender cómo crear componentes personalizados reutilizables
- Aprender a pasar props y manejar eventos entre componentes
- Dominar la composición de componentes y patrones de diseño
- Crear bibliotecas de componentes personalizados
- Entender las mejores prácticas para componentes reutilizables

---

## 📚 Contenido Teórico

### **¿Qué son los Componentes Personalizados?**

Los componentes personalizados son funciones o clases que encapsulan lógica y UI reutilizable. En React Native, estos componentes te permiten crear interfaces modulares, mantenibles y escalables.

#### **Características principales:**
- **Reutilizables**: Se pueden usar en múltiples partes de la aplicación
- **Modulares**: Cada componente tiene una responsabilidad específica
- **Mantenibles**: Cambios en un componente se reflejan en toda la app
- **Testeables**: Se pueden probar de manera independiente
- **Componibles**: Se pueden combinar para crear interfaces complejas

### **Tipos de Componentes:**

#### **1. Componentes Funcionales:**
- Usan funciones de JavaScript
- Más simples y ligeros
- Recomendados para la mayoría de casos
- Usan hooks para estado y efectos

#### **2. Componentes de Clase:**
- Usan clases de ES6
- Más complejos pero más potentes
- Útiles para componentes con lógica compleja
- Usan métodos de ciclo de vida

### **Patrones de Diseño de Componentes:**

1. **Presentational Components**: Solo muestran UI, no tienen lógica
2. **Container Components**: Manejan estado y lógica de negocio
3. **Compound Components**: Componentes que trabajan juntos
4. **Higher-Order Components**: Envuelven otros componentes
5. **Render Props**: Pasan funciones como props para renderizar contenido

---

## 💻 Implementación Práctica

### **1. Componente de Tarjeta Personalizada**

```javascript:src/components/CustomCard.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Componente de tarjeta personalizada y reutilizable
const CustomCard = ({ 
  // Props del componente con valores por defecto
  title = 'Sin título',           // Título de la tarjeta
  subtitle = '',                  // Subtítulo opcional
  description = '',               // Descripción del contenido
  image = null,                   // Imagen opcional
  onPress = null,                 // Función que se ejecuta al presionar
  onLongPress = null,             // Función que se ejecuta al presionar largo
  variant = 'default',            // Variante visual de la tarjeta
  size = 'medium',                // Tamaño de la tarjeta
  disabled = false,               // Si la tarjeta está deshabilitada
  children,                       // Contenido personalizado
  style,                          // Estilos adicionales
  ...props                        // Otras props que se pasan al View
}) => {
  // Función para obtener estilos según la variante
  const getVariantStyles = () => {
    switch (variant) {
      case 'primary':
        return {
          backgroundColor: '#007bff',
          borderColor: '#0056b3',
        };
      case 'success':
        return {
          backgroundColor: '#28a745',
          borderColor: '#1e7e34',
        };
      case 'warning':
        return {
          backgroundColor: '#ffc107',
          borderColor: '#e0a800',
        };
      case 'danger':
        return {
          backgroundColor: '#dc3545',
          borderColor: '#c82333',
        };
      case 'info':
        return {
          backgroundColor: '#17a2b8',
          borderColor: '#117a8b',
        };
      default:
        return {
          backgroundColor: '#ffffff',
          borderColor: '#dee2e6',
        };
    }
  };
  
  // Función para obtener estilos según el tamaño
  const getSizeStyles = () => {
    switch (size) {
      case 'small':
        return {
          padding: 12,
          borderRadius: 8,
          minHeight: 80,
        };
      case 'large':
        return {
          padding: 24,
          borderRadius: 16,
          minHeight: 160,
        };
      default: // medium
        return {
          padding: 16,
          borderRadius: 12,
          minHeight: 120,
        };
    }
  };
  
  // Función para obtener colores de texto según la variante
  const getTextColors = () => {
    const isLightVariant = ['warning', 'info'].includes(variant);
    return {
      title: isLightVariant ? '#212529' : '#ffffff',
      subtitle: isLightVariant ? '#6c757d' : '#e9ecef',
      description: isLightVariant ? '#495057' : '#dee2e6',
    };
  };
  
  // Obtener estilos calculados
  const variantStyles = getVariantStyles();
  const sizeStyles = getSizeStyles();
  const textColors = getTextColors();
  
  // Determinar si la tarjeta es presionable
  const isPressable = onPress || onLongPress;
  
  // Componente base (View o TouchableOpacity)
  const CardContainer = isPressable ? TouchableOpacity : View;
  
  // Props del contenedor
  const containerProps = isPressable ? {
    onPress: disabled ? null : onPress,
    onLongPress: disabled ? null : onLongPress,
    activeOpacity: 0.7,
    disabled,
  } : {};
  
  return (
    <CardContainer
      style={[
        styles.container,
        variantStyles,
        sizeStyles,
        disabled && styles.disabled,
        style,
      ]}
      {...containerProps}
      {...props}
    >
      {/* Imagen de la tarjeta (si existe) */}
      {image && (
        <View style={styles.imageContainer}>
          {typeof image === 'string' ? (
            <Image source={{ uri: image }} style={styles.image} />
          ) : (
            image
          )}
        </View>
      )}
      
      {/* Contenido de la tarjeta */}
      <View style={styles.content}>
        {/* Título de la tarjeta */}
        {title && (
          <Text style={[
            styles.title,
            { color: textColors.title }
          ]}>
            {title}
          </Text>
        )}
        
        {/* Subtítulo de la tarjeta */}
        {subtitle && (
          <Text style={[
            styles.subtitle,
            { color: textColors.subtitle }
          ]}>
            {subtitle}
          </Text>
        )}
        
        {/* Descripción de la tarjeta */}
        {description && (
          <Text style={[
            styles.description,
            { color: textColors.description }
          ]}>
            {description}
          </Text>
        )}
        
        {/* Contenido personalizado */}
        {children}
      </View>
      
      {/* Indicador de estado (si está deshabilitada) */}
      {disabled && (
        <View style={styles.disabledOverlay}>
          <Text style={styles.disabledText}>No disponible</Text>
        </View>
      )}
    </CardContainer>
  );
};

// Estilos del componente
const styles = StyleSheet.create({
  // Contenedor principal de la tarjeta
  container: {
    borderWidth: 1,
    elevation: 2, // Sombra en Android
    shadowColor: '#000', // Color de sombra en iOS
    shadowOffset: { width: 0, height: 2 }, // Offset de la sombra
    shadowOpacity: 0.1, // Opacidad de la sombra
    shadowRadius: 4, // Radio de la sombra
    marginVertical: 8, // Margen vertical entre tarjetas
    marginHorizontal: 16, // Margen horizontal desde los bordes
  },
  
  // Contenedor de la imagen
  imageContainer: {
    marginBottom: 12, // Espacio entre imagen y contenido
    alignItems: 'center', // Centra la imagen horizontalmente
  },
  
  // Estilo de la imagen
  image: {
    width: '100%', // Ocupa todo el ancho disponible
    height: 120, // Altura fija de la imagen
    borderRadius: 8, // Bordes redondeados
    resizeMode: 'cover', // Modo de redimensionamiento
  },
  
  // Contenido de la tarjeta
  content: {
    flex: 1, // Ocupa todo el espacio disponible
  },
  
  // Título de la tarjeta
  title: {
    fontSize: 18, // Tamaño de fuente del título
    fontWeight: 'bold', // Peso de la fuente
    marginBottom: 8, // Espacio después del título
    lineHeight: 24, // Altura de línea para mejor legibilidad
  },
  
  // Subtítulo de la tarjeta
  subtitle: {
    fontSize: 14, // Tamaño de fuente del subtítulo
    fontWeight: '500', // Peso de la fuente
    marginBottom: 6, // Espacio después del subtítulo
    lineHeight: 20, // Altura de línea
  },
  
  // Descripción de la tarjeta
  description: {
    fontSize: 14, // Tamaño de fuente de la descripción
    lineHeight: 20, // Altura de línea
    marginBottom: 8, // Espacio después de la descripción
  },
  
  // Estado deshabilitado
  disabled: {
    opacity: 0.6, // Reduce la opacidad
    backgroundColor: '#f8f9fa', // Color de fondo deshabilitado
    borderColor: '#dee2e6', // Color de borde deshabilitado
  },
  
  // Overlay para estado deshabilitado
  disabledOverlay: {
    position: 'absolute', // Posicionamiento absoluto
    top: 0, // Desde la parte superior
    left: 0, // Desde la izquierda
    right: 0, // Hasta la derecha
    bottom: 0, // Hasta la parte inferior
    backgroundColor: 'rgba(0, 0, 0, 0.1)', // Fondo semi-transparente
    justifyContent: 'center', // Centra verticalmente
    alignItems: 'center', // Centra horizontalmente
    borderRadius: 12, // Bordes redondeados
  },
  
  // Texto del overlay deshabilitado
  disabledText: {
    color: '#6c757d', // Color del texto
    fontSize: 12, // Tamaño de fuente
    fontWeight: '500', // Peso de la fuente
  },
});

// Propiedades del componente para documentación
CustomCard.propTypes = {
  title: PropTypes.string,
  subtitle: PropTypes.string,
  description: PropTypes.string,
  image: PropTypes.oneOfType([PropTypes.string, PropTypes.element]),
  onPress: PropTypes.func,
  onLongPress: PropTypes.func,
  variant: PropTypes.oneOf(['default', 'primary', 'success', 'warning', 'danger', 'info']),
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  disabled: PropTypes.bool,
  children: PropTypes.node,
  style: PropTypes.oneOfType([PropTypes.object, PropTypes.array]),
};

// Valores por defecto del componente
CustomCard.defaultProps = {
  title: 'Sin título',
  subtitle: '',
  description: '',
  image: null,
  onPress: null,
  onLongPress: null,
  variant: 'default',
  size: 'medium',
  disabled: false,
  children: null,
  style: {},
};

export default CustomCard;
```

### **2. Componente de Botón Personalizado**

```javascript:src/components/CustomButton.js
import React from 'react';
import { TouchableOpacity, Text, ActivityIndicator, StyleSheet } from 'react-native';

// Componente de botón personalizado y reutilizable
const CustomButton = ({
  // Props del componente
  title = 'Botón',                // Texto del botón
  onPress = () => {},             // Función que se ejecuta al presionar
  onLongPress = null,             // Función que se ejecuta al presionar largo
  variant = 'primary',            // Variante visual del botón
  size = 'medium',                // Tamaño del botón
  disabled = false,               // Si el botón está deshabilitado
  loading = false,                // Si el botón está cargando
  fullWidth = false,              // Si el botón ocupa todo el ancho
  icon = null,                    // Icono opcional
  iconPosition = 'left',          // Posición del icono
  style,                          // Estilos adicionales
  textStyle,                      // Estilos del texto
  ...props                        // Otras props que se pasan al TouchableOpacity
}) => {
  // Función para obtener estilos según la variante
  const getVariantStyles = () => {
    switch (variant) {
      case 'primary':
        return {
          backgroundColor: '#007bff',
          borderColor: '#0056b3',
        };
      case 'secondary':
        return {
          backgroundColor: '#6c757d',
          borderColor: '#545b62',
        };
      case 'success':
        return {
          backgroundColor: '#28a745',
          borderColor: '#1e7e34',
        };
      case 'warning':
        return {
          backgroundColor: '#ffc107',
          borderColor: '#e0a800',
        };
      case 'danger':
        return {
          backgroundColor: '#dc3545',
          borderColor: '#c82333',
        };
      case 'outline':
        return {
          backgroundColor: 'transparent',
          borderColor: '#007bff',
        };
      default:
        return {
          backgroundColor: '#007bff',
          borderColor: '#0056b3',
        };
    }
  };
  
  // Función para obtener estilos según el tamaño
  const getSizeStyles = () => {
    switch (size) {
      case 'small':
        return {
          paddingVertical: 8,
          paddingHorizontal: 16,
          borderRadius: 6,
          minHeight: 36,
        };
      case 'large':
        return {
          paddingVertical: 16,
          paddingHorizontal: 32,
          borderRadius: 12,
          minHeight: 56,
        };
      default: // medium
        return {
          paddingVertical: 12,
          paddingHorizontal: 24,
          borderRadius: 8,
          minHeight: 48,
        };
    }
  };
  
  // Función para obtener colores según la variante
  const getColors = () => {
    const isOutline = variant === 'outline';
    const isLight = ['warning'].includes(variant);
    
    return {
      backgroundColor: isOutline ? 'transparent' : getVariantStyles().backgroundColor,
      borderColor: getVariantStyles().borderColor,
      textColor: isOutline ? '#007bff' : (isLight ? '#212529' : '#ffffff'),
    };
  };
  
  // Obtener estilos calculados
  const variantStyles = getVariantStyles();
  const sizeStyles = getSizeStyles();
  const colors = getColors();
  
  // Determinar si el botón está realmente deshabilitado
  const isDisabled = disabled || loading;
  
  // Función para renderizar el icono
  const renderIcon = () => {
    if (!icon) return null;
    
    const iconStyle = [
      styles.icon,
      iconPosition === 'right' && styles.iconRight,
      { color: colors.textColor },
    ];
    
    return (
      <View style={iconStyle}>
        {typeof icon === 'string' ? (
          <Text style={[styles.iconText, { color: colors.textColor }]}>
            {icon}
          </Text>
        ) : (
          icon
        )}
      </View>
    );
  };
  
  // Función para renderizar el contenido del botón
  const renderContent = () => {
    if (loading) {
      return (
        <ActivityIndicator 
          color={colors.textColor} 
          size="small" 
        />
      );
    }
    
    return (
      <>
        {/* Icono izquierdo */}
        {icon && iconPosition === 'left' && renderIcon()}
        
        {/* Título del botón */}
        <Text style={[
          styles.title,
          { color: colors.textColor },
          textStyle,
        ]}>
          {title}
        </Text>
        
        {/* Icono derecho */}
        {icon && iconPosition === 'right' && renderIcon()}
      </>
    );
  };
  
  return (
    <TouchableOpacity
      style={[
        styles.container,
        variantStyles,
        sizeStyles,
        fullWidth && styles.fullWidth,
        isDisabled && styles.disabled,
        style,
      ]}
      onPress={isDisabled ? null : onPress}
      onLongPress={isDisabled ? null : onLongPress}
      activeOpacity={0.7}
      disabled={isDisabled}
      {...props}
    >
      {renderContent()}
    </TouchableOpacity>
  );
};

// Estilos del componente
const styles = StyleSheet.create({
  // Contenedor principal del botón
  container: {
    borderWidth: 2, // Grosor del borde
    justifyContent: 'center', // Centra el contenido verticalmente
    alignItems: 'center', // Centra el contenido horizontalmente
    flexDirection: 'row', // Organiza elementos horizontalmente
    elevation: 2, // Sombra en Android
    shadowColor: '#000', // Color de sombra en iOS
    shadowOffset: { width: 0, height: 2 }, // Offset de la sombra
    shadowOpacity: 0.1, // Opacidad de la sombra
    shadowRadius: 4, // Radio de la sombra
  },
  
  // Botón de ancho completo
  fullWidth: {
    width: '100%', // Ocupa todo el ancho disponible
  },
  
  // Título del botón
  title: {
    fontSize: 16, // Tamaño de fuente
    fontWeight: '600', // Peso de la fuente
    textAlign: 'center', // Centra el texto
  },
  
  // Icono del botón
  icon: {
    marginRight: 8, // Espacio a la derecha del icono
    justifyContent: 'center', // Centra el icono verticalmente
    alignItems: 'center', // Centra el icono horizontalmente
  },
  
  // Icono en posición derecha
  iconRight: {
    marginRight: 0, // Sin margen derecho
    marginLeft: 8, // Margen izquierdo en su lugar
  },
  
  // Texto del icono (para emojis o caracteres especiales)
  iconText: {
    fontSize: 18, // Tamaño de fuente del icono
  },
  
  // Estado deshabilitado
  disabled: {
    opacity: 0.6, // Reduce la opacidad
    backgroundColor: '#e9ecef', // Color de fondo deshabilitado
    borderColor: '#dee2e6', // Color de borde deshabilitado
  },
});

export default CustomButton;
```

### **3. Hook Personalizado para Componentes**

```javascript:src/hooks/useComponentState.js
import { useState, useCallback, useRef, useEffect } from 'react';

// Hook personalizado para manejar el estado de componentes
const useComponentState = (initialState = {}) => {
  // Estado principal del componente
  const [state, setState] = useState(initialState);
  
  // Referencia al estado anterior para comparaciones
  const previousState = useRef(initialState);
  
  // Referencia al estado inicial para reset
  const initialStateRef = useRef(initialState);
  
  // Función para actualizar el estado
  const updateState = useCallback((updates) => {
    setState(prevState => {
      // Guardamos el estado anterior
      previousState.current = prevState;
      
      // Aplicamos las actualizaciones
      if (typeof updates === 'function') {
        return updates(prevState);
      }
      
      return { ...prevState, ...updates };
    });
  }, []);
  
  // Función para actualizar una propiedad específica
  const updateProperty = useCallback((key, value) => {
    updateState(prevState => ({
      ...prevState,
      [key]: value,
    }));
  }, [updateState]);
  
  // Función para resetear el estado
  const resetState = useCallback(() => {
    setState(initialStateRef.current);
    previousState.current = initialStateRef.current;
  }, []);
  
  // Función para resetear propiedades específicas
  const resetProperties = useCallback((properties) => {
    setState(prevState => {
      const resetState = { ...prevState };
      
      properties.forEach(prop => {
        if (initialStateRef.current.hasOwnProperty(prop)) {
          resetState[prop] = initialStateRef.current[prop];
        }
      });
      
      return resetState;
    });
  }, []);
  
  // Función para obtener el valor anterior de una propiedad
  const getPreviousValue = useCallback((key) => {
    return previousState.current[key];
  }, []);
  
  // Función para verificar si una propiedad ha cambiado
  const hasPropertyChanged = useCallback((key) => {
    return state[key] !== previousState.current[key];
  }, [state]);
  
  // Función para obtener todas las propiedades que han cambiado
  const getChangedProperties = useCallback(() => {
    const changed = {};
    
    Object.keys(state).forEach(key => {
      if (state[key] !== previousState.current[key]) {
        changed[key] = {
          previous: previousState.current[key],
          current: state[key],
        };
      }
    });
    
    return changed;
  }, [state]);
  
  // Función para validar el estado
  const validateState = useCallback((validators) => {
    const errors = {};
    
    Object.entries(validators).forEach(([key, validator]) => {
      if (validator && typeof validator === 'function') {
        const result = validator(state[key], state);
        if (result !== true) {
          errors[key] = result;
        }
      }
    });
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors,
    };
  }, [state]);
  
  // Función para crear un setter específico
  const createSetter = useCallback((key) => {
    return (value) => updateProperty(key, value);
  }, [updateProperty]);
  
  // Función para crear un toggle para booleanos
  const createToggle = useCallback((key) => {
    return () => updateProperty(key, !state[key]);
  }, [updateProperty, state]);
  
  // Función para incrementar/decrementar números
  const createIncrementer = useCallback((key, step = 1) => {
    return (increment = true) => {
      updateProperty(key, state[key] + (increment ? step : -step));
    };
  }, [updateProperty, state]);
  
  // Efecto para actualizar la referencia del estado inicial
  useEffect(() => {
    initialStateRef.current = initialState;
  }, [initialState]);
  
  // Efecto para limpiar referencias cuando el componente se desmonta
  useEffect(() => {
    return () => {
      previousState.current = null;
      initialStateRef.current = null;
    };
  }, []);
  
  return {
    // Estado
    state,
    previousState: previousState.current,
    
    // Funciones de actualización
    updateState,
    updateProperty,
    resetState,
    resetProperties,
    
    // Funciones de utilidad
    getPreviousValue,
    hasPropertyChanged,
    getChangedProperties,
    validateState,
    
    // Funciones de conveniencia
    createSetter,
    createToggle,
    createIncrementer,
    
    // Estado inicial
    initialState: initialStateRef.current,
  };
};

export default useComponentState;
```

### **4. Utilidades para Componentes**

```javascript:src/utils/componentUtils.js
// Utilidades para trabajar con componentes en React Native

// Función para crear un componente con props por defecto
export const createComponentWithDefaults = (Component, defaultProps) => {
  const WrappedComponent = (props) => {
    const mergedProps = { ...defaultProps, ...props };
    return <Component {...mergedProps} />;
  };
  
  // Copiar propiedades estáticas
  WrappedComponent.displayName = `withDefaults(${Component.displayName || Component.name})`;
  WrappedComponent.propTypes = Component.propTypes;
  WrappedComponent.defaultProps = Component.defaultProps;
  
  return WrappedComponent;
};

// Función para crear un componente con validación
export const createComponentWithValidation = (Component, validators) => {
  const WrappedComponent = (props) => {
    // Validar props
    const validationErrors = {};
    
    Object.entries(validators).forEach(([key, validator]) => {
      if (validator && typeof validator === 'function') {
        const result = validator(props[key], props);
        if (result !== true) {
          validationErrors[key] = result;
        }
      }
    });
    
    // Si hay errores de validación, mostrar error o usar valores por defecto
    if (Object.keys(validationErrors).length > 0) {
      console.warn('Validation errors:', validationErrors);
      
      // Aplicar valores por defecto para props inválidas
      const validatedProps = { ...props };
      Object.entries(validationErrors).forEach(([key, error]) => {
        if (Component.defaultProps && Component.defaultProps[key] !== undefined) {
          validatedProps[key] = Component.defaultProps[key];
        }
      });
      
      return <Component {...validatedProps} />;
    }
    
    return <Component {...props} />;
  };
  
  WrappedComponent.displayName = `withValidation(${Component.displayName || Component.name})`;
  WrappedComponent.propTypes = Component.propTypes;
  WrappedComponent.defaultProps = Component.defaultProps;
  
  return WrappedComponent;
};

// Función para crear un componente con memoización
export const createMemoizedComponent = (Component, shouldUpdate) => {
  const MemoizedComponent = React.memo(Component, shouldUpdate);
  
  MemoizedComponent.displayName = `memoized(${Component.displayName || Component.name})`;
  MemoizedComponent.propTypes = Component.propTypes;
  MemoizedComponent.defaultProps = Component.defaultProps;
  
  return MemoizedComponent;
};

// Función para crear un componente con logging
export const createComponentWithLogging = (Component, logLevel = 'info') => {
  const LoggedComponent = (props) => {
    // Log de props al montar
    useEffect(() => {
      if (logLevel === 'debug' || logLevel === 'info') {
        console.log(`[${Component.displayName || Component.name}] Mounted with props:`, props);
      }
    }, []);
    
    // Log de props al actualizar
    useEffect(() => {
      if (logLevel === 'debug') {
        console.log(`[${Component.displayName || Component.name}] Updated with props:`, props);
      }
    });
    
    // Log al desmontar
    useEffect(() => {
      return () => {
        if (logLevel === 'debug' || logLevel === 'info') {
          console.log(`[${Component.displayName || Component.name}] Unmounted`);
        }
      };
    }, []);
    
    return <Component {...props} />;
  };
  
  LoggedComponent.displayName = `withLogging(${Component.displayName || Component.name})`;
  LoggedComponent.propTypes = Component.propTypes;
  LoggedComponent.defaultProps = Component.defaultProps;
  
  return LoggedComponent;
};

// Función para crear un componente con error boundary
export const createComponentWithErrorBoundary = (Component, fallback) => {
  class ErrorBoundary extends React.Component {
    constructor(props) {
      super(props);
      this.state = { hasError: false, error: null };
    }
    
    static getDerivedStateFromError(error) {
      return { hasError: true, error };
    }
    
    componentDidCatch(error, errorInfo) {
      console.error('Component error:', error, errorInfo);
    }
    
    render() {
      if (this.state.hasError) {
        if (fallback) {
          return typeof fallback === 'function' 
            ? fallback(this.state.error, this.props)
            : fallback;
        }
        
        return (
          <View style={styles.errorContainer}>
            <Text style={styles.errorText}>Algo salió mal</Text>
            <TouchableOpacity
              style={styles.retryButton}
              onPress={() => this.setState({ hasError: false, error: null })}
            >
              <Text style={styles.retryButtonText}>Reintentar</Text>
            </TouchableOpacity>
          </View>
        );
      }
      
      return <Component {...this.props} />;
    }
  }
  
  ErrorBoundary.displayName = `withErrorBoundary(${Component.displayName || Component.name})`;
  ErrorBoundary.propTypes = Component.propTypes;
  ErrorBoundary.defaultProps = Component.defaultProps;
  
  return ErrorBoundary;
};

// Función para crear un componente con loading state
export const createComponentWithLoading = (Component, loadingComponent) => {
  const LoadingComponent = ({ loading, ...props }) => {
    if (loading) {
      return loadingComponent || (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#007bff" />
          <Text style={styles.loadingText}>Cargando...</Text>
        </View>
      );
    }
    
    return <Component {...props} />;
  };
  
  LoadingComponent.displayName = `withLoading(${Component.displayName || Component.name})`;
  LoadingComponent.propTypes = {
    ...Component.propTypes,
    loading: PropTypes.bool,
  };
  LoadingComponent.defaultProps = {
    ...Component.defaultProps,
    loading: false,
  };
  
  return LoadingComponent;
};

// Estilos para componentes de utilidad
const styles = StyleSheet.create({
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f8f9fa',
  },
  
  errorText: {
    fontSize: 16,
    color: '#dc3545',
    marginBottom: 20,
    textAlign: 'center',
  },
  
  retryButton: {
    backgroundColor: '#007bff',
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
  },
  
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f8f9fa',
  },
  
  loadingText: {
    fontSize: 16,
    color: '#6c757d',
    marginTop: 16,
  },
});
```

---

## 🧪 Casos de Uso

### **Caso 1: Componente de Lista Personalizada**
```javascript
const CustomList = ({ 
  items, 
  renderItem, 
  emptyMessage = 'No hay elementos',
  loading = false,
  onRefresh = null,
  refreshing = false 
}) => {
  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007bff" />
        <Text style={styles.loadingText}>Cargando...</Text>
      </View>
    );
  }
  
  if (items.length === 0) {
    return (
      <View style={styles.emptyContainer}>
        <Text style={styles.emptyText}>{emptyMessage}</Text>
      </View>
    );
  }
  
  return (
    <FlatList
      data={items}
      renderItem={renderItem}
      keyExtractor={(item, index) => item.id || index.toString()}
      onRefresh={onRefresh}
      refreshing={refreshing}
      showsVerticalScrollIndicator={false}
    />
  );
};
```

### **Caso 2: Componente de Formulario Personalizado**
```javascript
const CustomForm = ({ 
  fields, 
  onSubmit, 
  submitText = 'Enviar',
  loading = false,
  initialValues = {},
  validation = {}
}) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  
  const handleSubmit = () => {
    // Validar campos
    const newErrors = {};
    Object.keys(validation).forEach(field => {
      const value = values[field];
      const validator = validation[field];
      
      if (validator && !validator(value, values)) {
        newErrors[field] = `Campo ${field} es requerido`;
      }
    });
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    onSubmit(values);
  };
  
  return (
    <View style={styles.formContainer}>
      {fields.map(field => (
        <View key={field.name} style={styles.fieldContainer}>
          <Text style={styles.fieldLabel}>{field.label}</Text>
          <TextInput
            style={[styles.fieldInput, errors[field.name] && styles.fieldInputError]}
            value={values[field.name] || ''}
            onChangeText={(text) => setValues(prev => ({ ...prev, [field.name]: text }))}
            placeholder={field.placeholder}
            secureTextEntry={field.secure}
          />
          {errors[field.name] && (
            <Text style={styles.fieldError}>{errors[field.name]}</Text>
          )}
        </View>
      ))}
      
      <CustomButton
        title={submitText}
        onPress={handleSubmit}
        loading={loading}
        fullWidth
        style={styles.submitButton}
      />
    </View>
  );
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear un Componente de Modal**
Crea un componente modal personalizado con:
- Overlay de fondo
- Contenido personalizable
- Animaciones de entrada/salida
- Botones de acción configurables

### **Ejercicio 2: Componente de Navegación por Tabs**
Implementa un componente de navegación por pestañas con:
- Múltiples pestañas
- Indicador de pestaña activa
- Contenido personalizable por pestaña
- Animaciones de transición

### **Ejercicio 3: Componente de Búsqueda**
Crea un componente de búsqueda con:
- Campo de entrada
- Filtros configurables
- Resultados en tiempo real
- Historial de búsquedas

---

## 🚀 Proyecto de la Clase

### **Biblioteca de Componentes Personalizados**

Crea una biblioteca de componentes reutilizables:
- **Componentes de UI**: Botones, tarjetas, inputs, modales
- **Componentes de Layout**: Contenedores, grids, listas
- **Componentes de Navegación**: Tabs, breadcrumbs, paginación
- **Componentes de Datos**: Gráficos, tablas, formularios

**Requisitos:**
1. Crear al menos 10 componentes personalizados
2. Implementar props configurables para cada componente
3. Crear documentación con ejemplos de uso
4. Implementar sistema de temas y variantes
5. Crear tests para cada componente

**Estructura sugerida:**
```
src/
├── components/
│   ├── ui/
│   │   ├── Button/
│   │   ├── Card/
│   │   ├── Input/
│   │   └── Modal/
│   ├── layout/
│   │   ├── Container/
│   │   ├── Grid/
│   │   └── List/
│   └── navigation/
│       ├── Tabs/
│       └── Pagination/
├── hooks/
│   └── useComponentState.js
├── utils/
│   └── componentUtils.js
└── docs/
    └── components.md
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Native Components](https://reactnative.dev/docs/components-and-apis)
- [React Components](https://es.reactjs.org/docs/components-and-props.html)

### **Artículos Recomendados:**
- "Cómo crear componentes reutilizables en React Native"
- "Patrones de diseño para componentes móviles"
- "Mejores prácticas para bibliotecas de componentes"

### **Herramientas:**
- [Storybook](https://storybook.js.org/) - Para documentar componentes
- [React DevTools](https://github.com/facebook/react-devtools) - Para debugging
- [React Native Elements](https://reactnativeelements.com/) - Biblioteca de componentes

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Componentes personalizados**: Funciones o clases que encapsulan UI reutilizable
- **Props**: Datos que se pasan entre componentes
- **Composición**: Combinar componentes para crear interfaces complejas
- **Patrones de diseño**: Estructuras comunes para organizar componentes
- **Reutilización**: Crear componentes que se pueden usar en múltiples lugares

### **Habilidades Desarrolladas:**
- ✅ Crear componentes personalizados reutilizables
- ✅ Manejar props y eventos entre componentes
- ✅ Implementar patrones de diseño de componentes
- ✅ Crear hooks personalizados para componentes
- ✅ Construir bibliotecas de componentes

### **Próximos Pasos:**
En el siguiente módulo aprenderemos sobre **Navegación y Routing**, que te permitirá crear aplicaciones con múltiples pantallas y navegación entre ellas.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Flexbox y Layout](clase_3_flexbox_layout.md)
- **➡️ Siguiente Módulo**: [Módulo 3: Navegación y Routing](../midLevel_1/README.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
