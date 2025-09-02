# 🎨 Clase 2: Diseño Visual y Branding

## 📍 Navegación
- **📚 Módulo**: [Módulo 9: UI-UX y Diseño de Interfaces](README.md)
- **⬅️ Anterior**: [Clase 1: Fundamentos de UI-UX](clase_1_fundamentos_ui_ux.md)
- **➡️ Siguiente**: [Clase 3: Layout y Composición](clase_3_layout_composicion.md)

---

## 🎯 Objetivos de la Clase

### **Al Finalizar esta Clase, Serás Capaz de:**
1. **Comprender** la psicología del color y su impacto en el usuario
2. **Diseñar** sistemas tipográficos profesionales y legibles
3. **Crear** iconografía coherente y significativa
4. **Desarrollar** sistemas de diseño escalables
5. **Implementar** estrategias de branding efectivas
6. **Aplicar** principios de diseño visual en React Native

---

## 🌈 Psicología del Color

### **¿Por qué es Importante el Color?**

El color afecta directamente:
- **Emociones** del usuario
- **Percepción** de la marca
- **Legibilidad** del contenido
- **Accesibilidad** de la interfaz
- **Comportamiento** del usuario

### **Teoría del Color**

#### **Colores Primarios**
- **Rojo**: Energía, pasión, urgencia
- **Azul**: Confianza, estabilidad, profesionalismo
- **Amarillo**: Optimismo, claridad, atención

#### **Colores Secundarios**
- **Verde**: Crecimiento, salud, naturaleza
- **Naranja**: Creatividad, entusiasmo, éxito
- **Púrpura**: Lujo, sabiduría, creatividad

### **Implementación en React Native**

#### **Sistema de Colores Profesional**
```jsx
const colorSystem = {
  // Colores primarios de marca
  primary: {
    50: '#E3F2FD',   // Azul muy claro
    100: '#BBDEFB',  // Azul claro
    200: '#90CAF9',  // Azul medio
    300: '#64B5F6',  // Azul
    400: '#42A5F5',  // Azul
    500: '#2196F3',  // Azul principal
    600: '#1E88E5',  // Azul oscuro
    700: '#1976D2',  // Azul muy oscuro
    800: '#1565C0',  // Azul extremadamente oscuro
    900: '#0D47A1',  // Azul más oscuro
  },
  
  // Colores semánticos
  semantic: {
    success: { light: '#81C784', main: '#4CAF50', dark: '#388E3C' },
    warning: { light: '#FFB74D', main: '#FF9800', dark: '#F57C00' },
    error: { light: '#E57373', main: '#F44336', dark: '#D32F2F' },
    info: { light: '#64B5F6', main: '#2196F3', dark: '#1976D2' }
  },
  
  // Colores de fondo y texto
  background: { primary: '#FFFFFF', secondary: '#F5F5F5', tertiary: '#EEEEEE' },
  text: { primary: '#212121', secondary: '#757575', disabled: '#BDBDBD', inverse: '#FFFFFF' },
  border: { light: '#E0E0E0', medium: '#BDBDBD', dark: '#9E9E9E' }
};

// Hook para usar colores con temas
const useThemeColors = () => {
  const [isDarkMode, setIsDarkMode] = useState(false);
  
  const colors = useMemo(() => ({
    ...colorSystem,
    background: {
      primary: isDarkMode ? '#121212' : colorSystem.background.primary,
      secondary: isDarkMode ? '#1E1E1E' : colorSystem.background.secondary,
      tertiary: isDarkMode ? '#2D2D2D' : colorSystem.background.tertiary,
    },
    text: {
      primary: isDarkMode ? '#FFFFFF' : colorSystem.text.primary,
      secondary: isDarkMode ? '#B0B0B0' : colorSystem.text.secondary,
      disabled: isDarkMode ? '#666666' : colorSystem.text.disabled,
    }
  }), [isDarkMode]);
  
  return { colors, isDarkMode, setIsDarkMode };
};
```

---

## 🔤 Sistemas Tipográficos

### **¿Por qué es Importante la Tipografía?**

La tipografía es fundamental para:
- **Legibilidad** del contenido
- **Jerarquía visual** de la información
- **Personalidad** de la marca
- **Consistencia** del diseño
- **Accesibilidad** del usuario

### **Principios de Tipografía**

#### **Jerarquía Tipográfica**
- **Títulos**: 24-32px - Información más importante
- **Subtítulos**: 18-20px - Información secundaria
- **Cuerpo**: 14-16px - Contenido principal
- **Caption**: 12-14px - Información auxiliar

#### **Espaciado y Line-height**
- **Line-height**: 1.4-1.6 para texto del cuerpo
- **Espaciado entre párrafos**: 1.5-2 veces el tamaño de fuente
- **Márgenes**: Consistentes y proporcionales

### **Implementación en React Native**

#### **Sistema Tipográfico Completo**
```jsx
const typography = {
  fontFamily: {
    primary: Platform.select({ ios: 'SF Pro Display', android: 'Roboto' }),
    secondary: Platform.select({ ios: 'SF Pro Text', android: 'Roboto' }),
    monospace: Platform.select({ ios: 'SF Mono', android: 'Roboto Mono' }),
  },
  
  fontSize: {
    xs: 12, sm: 14, base: 16, lg: 18, xl: 20,
    '2xl': 24, '3xl': 30, '4xl': 36, '5xl': 48,
  },
  
  fontWeight: {
    light: '300', normal: '400', medium: '500',
    semibold: '600', bold: '700', extrabold: '800',
  },
  
  lineHeight: { tight: 1.25, snug: 1.375, normal: 1.5, relaxed: 1.625, loose: 2 },
  letterSpacing: { tighter: -0.05, tight: -0.025, normal: 0, wide: 0.025, wider: 0.05, widest: 0.1 }
};

// Componente de texto tipográfico
const Typography = ({ variant = 'body', children, style, color, ...props }) => {
  const { colors } = useThemeColors();
  
  const getTypographyStyle = () => {
    const baseStyle = {
      fontFamily: typography.fontFamily.primary,
      color: color || colors.text.primary,
    };
    
    const variantStyles = {
      h1: {
        fontSize: typography.fontSize['4xl'],
        fontWeight: typography.fontWeight.bold,
        lineHeight: typography.fontSize['4xl'] * typography.lineHeight.tight,
        letterSpacing: typography.letterSpacing.tight,
      },
      h2: {
        fontSize: typography.fontSize['3xl'],
        fontWeight: typography.fontWeight.bold,
        lineHeight: typography.fontSize['3xl'] * typography.lineHeight.snug,
        letterSpacing: typography.letterSpacing.tight,
      },
      h3: {
        fontSize: typography.fontSize['2xl'],
        fontWeight: typography.fontWeight.semibold,
        lineHeight: typography.fontSize['2xl'] * typography.lineHeight.snug,
      },
      body: {
        fontSize: typography.fontSize.base,
        fontWeight: typography.fontWeight.normal,
        lineHeight: typography.fontSize.base * typography.lineHeight.relaxed,
      },
      bodyLarge: {
        fontSize: typography.fontSize.lg,
        fontWeight: typography.fontWeight.normal,
        lineHeight: typography.fontSize.lg * typography.lineHeight.relaxed,
      },
      caption: {
        fontSize: typography.fontSize.xs,
        fontWeight: typography.fontWeight.normal,
        lineHeight: typography.fontSize.xs * typography.lineHeight.normal,
        color: colors.text.secondary,
      },
      button: {
        fontSize: typography.fontSize.base,
        fontWeight: typography.fontWeight.medium,
        letterSpacing: typography.letterSpacing.wide,
      }
    };
    
    return {
      ...baseStyle,
      ...variantStyles[variant],
      ...style,
    };
  };
  
  return (
    <Text style={getTypographyStyle()} {...props}>
      {children}
    </Text>
  );
};
```

---

## 🎨 Iconografía y Símbolos

### **¿Por qué son Importantes los Iconos?**

Los iconos son esenciales para:
- **Comunicación visual** rápida
- **Ahorro de espacio** en la interfaz
- **Navegación intuitiva** del usuario
- **Consistencia visual** de la marca
- **Accesibilidad** para usuarios internacionales

### **Principios de Iconografía**

#### **1. Simplicidad**
- **Formas básicas** y reconocibles
- **Detalles mínimos** necesarios
- **Consistencia** en el estilo

#### **2. Reconocibilidad**
- **Convenciones** establecidas
- **Contexto** claro de uso
- **Testing** con usuarios

### **Implementación en React Native**

#### **Sistema de Iconos Profesional**
```jsx
import Icon from 'react-native-vector-icons/MaterialIcons';

// Sistema de iconos organizado por categorías
const iconSystem = {
  navigation: {
    home: 'home', back: 'arrow-back', forward: 'arrow-forward',
    close: 'close', menu: 'menu', search: 'search', filter: 'filter-list',
  },
  actions: {
    add: 'add', edit: 'edit', delete: 'delete', save: 'save',
    share: 'share', download: 'file-download', upload: 'file-upload',
  },
  states: {
    success: 'check-circle', error: 'error', warning: 'warning',
    info: 'info', loading: 'hourglass-empty', offline: 'wifi-off',
  },
  content: {
    image: 'image', video: 'video-library', audio: 'audiotrack',
    document: 'description', folder: 'folder', link: 'link',
  }
};

// Componente de icono profesional
const AppIcon = ({ name, size = 24, color, style, onPress, disabled = false }) => {
  const { colors } = useThemeColors();
  
  const iconColor = color || colors.text.primary;
  const iconStyle = {
    color: disabled ? colors.text.disabled : iconColor,
    opacity: disabled ? 0.6 : 1,
    ...style,
  };
  
  if (onPress) {
    return (
      <TouchableOpacity
        onPress={onPress}
        disabled={disabled}
        style={iconStyle}
        activeOpacity={0.7}
      >
        <Icon name={name} size={size} color={iconColor} />
      </TouchableOpacity>
    );
  }
  
  return <Icon name={name} size={size} color={iconColor} style={iconStyle} />;
};
```

---

## 🎯 Sistemas de Diseño

### **¿Qué es un Sistema de Diseño?**

Un sistema de diseño es una colección de componentes reutilizables, guiados por estándares claros, que pueden ser ensamblados para construir cualquier número de aplicaciones.

### **Componentes de un Sistema de Diseño**

#### **1. Tokens de Diseño**
- **Colores**: Paletas y variantes
- **Tipografía**: Familias, tamaños, pesos
- **Espaciado**: Márgenes, padding, gaps
- **Sombras**: Elevación y profundidad
- **Bordes**: Radios, grosores, colores

#### **2. Componentes Base**
- **Botones**: Primarios, secundarios, terciarios
- **Campos de texto**: Inputs, textareas, selectores
- **Navegación**: Tabs, breadcrumbs, paginación
- **Feedback**: Alertas, notificaciones, loaders

### **Implementación en React Native**

#### **Sistema de Espaciado**
```jsx
const spacing = {
  xs: 4, sm: 8, md: 16, lg: 24, xl: 32, '2xl': 48, '3xl': 64,
  
  button: { padding: 16, margin: 8, borderRadius: 8 },
  card: { padding: 16, margin: 8, borderRadius: 12 },
  input: { padding: 12, margin: 8, borderRadius: 8 },
  section: { padding: 24, margin: 16 }
};

const useSpacing = () => ({ spacing });
```

#### **Sistema de Sombras y Elevación**
```jsx
const shadows = {
  ios: {
    none: { shadowColor: 'transparent', shadowOffset: { width: 0, height: 0 }, shadowOpacity: 0, shadowRadius: 0 },
    sm: { shadowColor: '#000', shadowOffset: { width: 0, height: 1 }, shadowOpacity: 0.05, shadowRadius: 2 },
    md: { shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.1, shadowRadius: 4 },
    lg: { shadowColor: '#000', shadowOffset: { width: 0, height: 4 }, shadowOpacity: 0.15, shadowRadius: 8 },
  },
  android: {
    none: { elevation: 0 }, sm: { elevation: 1 }, md: { elevation: 2 },
    lg: { elevation: 4 }, xl: { elevation: 8 }
  }
};

const useShadows = () => {
  const isIOS = Platform.OS === 'ios';
  const getShadow = (level = 'none') => isIOS ? shadows.ios[level] : shadows.android[level];
  return { getShadow, isIOS };
};
```

#### **Componente de Botón con Sistema de Diseño**
```jsx
const Button = ({ variant = 'primary', size = 'medium', children, onPress, disabled = false }) => {
  const { colors } = useThemeColors();
  const { spacing } = useSpacing();
  
  const getButtonStyle = () => {
    const baseStyle = {
      borderRadius: spacing.button.borderRadius,
      alignItems: 'center',
      justifyContent: 'center',
      flexDirection: 'row',
    };
    
    const variantStyles = {
      primary: { backgroundColor: colors.primary[500], borderColor: colors.primary[500] },
      secondary: { backgroundColor: 'transparent', borderColor: colors.primary[500], borderWidth: 1 },
      success: { backgroundColor: colors.semantic.success.main, borderColor: colors.semantic.success.main },
      warning: { backgroundColor: colors.semantic.warning.main, borderColor: colors.semantic.warning.main },
      error: { backgroundColor: colors.semantic.error.main, borderColor: colors.semantic.error.main }
    };
    
    const sizeStyles = {
      small: { paddingVertical: 8, paddingHorizontal: 16, minHeight: 36 },
      medium: { paddingVertical: 12, paddingHorizontal: 24, minHeight: 44 },
      large: { paddingVertical: 16, paddingHorizontal: 32, minHeight: 56 }
    };
    
    const stateStyles = disabled ? {
      backgroundColor: colors.text.disabled,
      borderColor: colors.text.disabled,
      opacity: 0.6,
    } : {};
    
    return {
      ...baseStyle,
      ...variantStyles[variant],
      ...sizeStyles[size],
      ...stateStyles,
    };
  };
  
  const getTextStyle = () => {
    const baseTextStyle = { fontWeight: '600', textAlign: 'center' };
    
    const variantTextStyles = {
      primary: { color: colors.text.inverse },
      secondary: { color: colors.primary[500] },
      success: { color: colors.text.inverse },
      warning: { color: colors.text.inverse },
      error: { color: colors.text.inverse },
    };
    
    const sizeTextStyles = {
      small: { fontSize: 14 }, medium: { fontSize: 16 }, large: { fontSize: 18 },
    };
    
    return {
      ...baseTextStyle,
      ...variantTextStyles[variant],
      ...sizeTextStyles[size],
    };
  };
  
  return (
    <TouchableOpacity
      style={getButtonStyle()}
      onPress={onPress}
      disabled={disabled}
      activeOpacity={0.8}
    >
      <Text style={getTextStyle()}>{children}</Text>
    </TouchableOpacity>
  );
};
```

---

## 🏷️ Branding y Identidad Visual

### **¿Qué es el Branding?**

El branding es el proceso de crear una identidad visual única y memorable para una marca, que incluya:
- **Logo** y símbolos
- **Paleta de colores** corporativa
- **Tipografía** de marca
- **Tono de voz** y personalidad
- **Valores** y promesas

### **Elementos del Branding**

#### **1. Logo y Símbolos**
- **Logo principal**: Versión completa de la marca
- **Símbolo**: Elemento gráfico reconocible
- **Logotipo**: Versión solo texto
- **Favicon**: Versión simplificada para apps

#### **2. Paleta de Colores Corporativa**
- **Color primario**: Color principal de la marca
- **Color secundario**: Color de apoyo
- **Colores acento**: Colores para destacar elementos
- **Colores neutros**: Colores de fondo y texto

### **Implementación en React Native**

#### **Sistema de Branding Completo**
```jsx
const branding = {
  brand: {
    name: 'FitFlow',
    tagline: 'Transforma tu vida, un movimiento a la vez',
    description: 'App de fitness personalizada para todos los niveles',
  },
  
  brandColors: {
    primary: '#FF6B35',      // Naranja energético
    secondary: '#4ECDC4',    // Turquesa fresco
    accent: '#45B7D1',       // Azul confianza
    success: '#96CEB4',      // Verde salud
    warning: '#FFEAA7',      // Amarillo atención
    error: '#DDA0A0',        // Rojo suave
  },
  
  brandTypography: {
    display: { fontFamily: 'Poppins-Bold', fontSize: 48, lineHeight: 56, letterSpacing: -0.02 },
    headline: { fontFamily: 'Poppins-SemiBold', fontSize: 32, lineHeight: 40, letterSpacing: -0.01 },
    title: { fontFamily: 'Poppins-Medium', fontSize: 24, lineHeight: 32, letterSpacing: 0 },
    body: { fontFamily: 'Inter-Regular', fontSize: 16, lineHeight: 24, letterSpacing: 0.01 },
    caption: { fontFamily: 'Inter-Medium', fontSize: 14, lineHeight: 20, letterSpacing: 0.02 }
  },
  
  brandIcons: {
    primary: 'fitness-center', secondary: 'directions-run', accent: 'self-improvement',
    success: 'check-circle', warning: 'warning', error: 'error',
  }
};

const useBranding = () => ({ branding });
```

#### **Componente de Header con Branding**
```jsx
const BrandHeader = ({ title, subtitle, showLogo = true }) => {
  const { branding } = useBranding();
  const { colors } = useThemeColors();
  
  return (
    <View style={styles.header}>
      {showLogo && (
        <View style={styles.logoContainer}>
          <Icon 
            name={branding.brandIcons.primary} 
            size={32} 
            color={branding.brandColors.primary} 
          />
          <Text style={[styles.brandName, branding.brandTypography.title]}>
            {branding.brand.name}
          </Text>
        </View>
      )}
      
      <View style={styles.headerContent}>
        <Text style={[styles.title, branding.brandTypography.headline]}>
          {title}
        </Text>
        {subtitle && (
          <Text style={[styles.subtitle, branding.brandTypography.body]}>
            {subtitle}
          </Text>
        )}
      </View>
    </View>
  );
};
```

---

## 📱 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Colores**
**Objetivo**: Crear un sistema de colores completo para una app de tu elección.

**Requisitos:**
- Paleta de 5 colores principales con variantes
- Colores semánticos (éxito, error, advertencia, información)
- Colores de fondo y texto
- Implementación en React Native

**Entregable**: Archivo de colores con implementación y ejemplo de uso.

### **Ejercicio 2: Sistema Tipográfico**
**Objetivo**: Diseñar un sistema tipográfico profesional.

**Requisitos:**
- 3 variantes de títulos (H1, H2, H3)
- 2 variantes de texto del cuerpo
- Texto de botón y caption
- Implementación con componentes reutilizables

**Entregable**: Sistema tipográfico completo con ejemplos de uso.

### **Ejercicio 3: Sistema de Iconos**
**Objetivo**: Crear un sistema de iconos organizado y consistente.

**Requisitos:**
- Categorización por funcionalidad
- Iconos para navegación, acciones y estados
- Componente reutilizable de icono
- Implementación con react-native-vector-icons

**Entregable**: Sistema de iconos organizado con documentación.

---

## 🎯 Resumen de la Clase

### **✅ Lo que has aprendido:**
1. **Psicología del color** - Impacto emocional y comportamental
2. **Sistemas tipográficos** - Jerarquía, legibilidad y consistencia
3. **Iconografía profesional** - Simplicidad, reconocibilidad y escalabilidad
4. **Sistemas de diseño** - Tokens, componentes y patrones
5. **Branding efectivo** - Identidad visual y personalidad de marca
6. **Implementación práctica** - Código React Native reutilizable

### **🚀 Próximos pasos:**
- **Práctica**: Completa los ejercicios de esta clase
- **Experimentación**: Crea tu propio sistema de colores y tipografía
- **Documentación**: Desarrolla guías de estilo para tu proyecto
- **Siguiente clase**: Layout y Composición

---

## 🔗 Recursos Adicionales

### **📚 Lectura Recomendada**
- **"Color: A Natural History of the Palette"** - Victoria Finlay
- **"Thinking with Type"** - Ellen Lupton
- **"Logo Design Love"** - David Airey

### **🌐 Recursos Online**
- **Adobe Color**: [color.adobe.com](https://color.adobe.com/)
- **Google Fonts**: [fonts.google.com](https://fonts.google.com/)
- **IconFinder**: [iconfinder.com](https://www.iconfinder.com/)

---

**🎨 ¡Excelente! Has completado la clase de Diseño Visual y Branding.**

Ahora tienes un conocimiento sólido de cómo crear sistemas visuales profesionales y coherentes. En la siguiente clase aprenderemos sobre **Layout y Composición**, donde aplicaremos estos principios visuales para crear interfaces bien estructuradas y atractivas.

**¡Continúa con la siguiente clase para dominar la composición visual!** 🚀

### **Clase Siguiente**: [Clase 3: Layout y Composición](clase_3_layout_composicion.md)
