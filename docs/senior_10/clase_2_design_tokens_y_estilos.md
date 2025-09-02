# Clase 2: Design Tokens y Estilos

## Objetivos de la Clase
- Comprender qué son los Design Tokens
- Implementar un sistema de tokens en React Native
- Crear estilos consistentes y escalables
- Gestionar temas y variaciones de diseño

## 1. ¿Qué son los Design Tokens?

### Concepto
Los Design Tokens son valores atómicos que definen las propiedades visuales de un diseño:
- Colores
- Tipografías
- Espaciados
- Bordes
- Sombras
- Animaciones

### Beneficios
- **Consistencia**: Valores unificados en toda la aplicación
- **Escalabilidad**: Fácil mantenimiento y actualización
- **Flexibilidad**: Soporte para múltiples temas
- **Colaboración**: Bridge entre diseño y desarrollo

## 2. Estructura de Design Tokens

### Organización Jerárquica
```typescript
// tokens/colors.ts
export const colors = {
  // Base colors
  primary: {
    50: '#f0f9ff',
    100: '#e0f2fe',
    500: '#0ea5e9',
    900: '#0c4a6e',
  },
  secondary: {
    50: '#fdf4ff',
    100: '#fae8ff',
    500: '#a855f7',
    900: '#581c87',
  },
  // Semantic colors
  semantic: {
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
    info: '#3b82f6',
  },
  // Neutral colors
  neutral: {
    50: '#f9fafb',
    100: '#f3f4f6',
    500: '#6b7280',
    900: '#111827',
  },
} as const;

// tokens/typography.ts
export const typography = {
  fontFamily: {
    primary: 'System',
    secondary: 'System',
    mono: 'Courier New',
  },
  fontSize: {
    xs: 12,
    sm: 14,
    base: 16,
    lg: 18,
    xl: 20,
    '2xl': 24,
    '3xl': 30,
    '4xl': 36,
  },
  fontWeight: {
    normal: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
  },
  lineHeight: {
    tight: 1.25,
    normal: 1.5,
    relaxed: 1.75,
  },
} as const;

// tokens/spacing.ts
export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  '2xl': 48,
  '3xl': 64,
} as const;

// tokens/borders.ts
export const borders = {
  radius: {
    none: 0,
    sm: 4,
    md: 8,
    lg: 12,
    xl: 16,
    full: 9999,
  },
  width: {
    none: 0,
    thin: 1,
    medium: 2,
    thick: 4,
  },
} as const;

// tokens/shadows.ts
export const shadows = {
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
    elevation: 1,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 2,
  },
  lg: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.15,
    shadowRadius: 8,
    elevation: 4,
  },
} as const;
```

## 3. Implementación en React Native

### Sistema de Tokens
```typescript
// tokens/index.ts
export * from './colors';
export * from './typography';
export * from './spacing';
export * from './borders';
export * from './shadows';

// tokens/theme.ts
import { colors, typography, spacing, borders, shadows } from './index';

export const lightTheme = {
  colors: {
    ...colors,
    background: colors.neutral[50],
    surface: colors.neutral[100],
    text: colors.neutral[900],
    textSecondary: colors.neutral[500],
  },
  typography,
  spacing,
  borders,
  shadows,
} as const;

export const darkTheme = {
  colors: {
    ...colors,
    background: colors.neutral[900],
    surface: colors.neutral[800],
    text: colors.neutral[50],
    textSecondary: colors.neutral[400],
  },
  typography,
  spacing,
  borders,
  shadows,
} as const;

export type Theme = typeof lightTheme;
```

### Hook de Tema
```typescript
// hooks/useTheme.ts
import { useContext, createContext } from 'react';
import { lightTheme, darkTheme, Theme } from '../tokens/theme';

interface ThemeContextType {
  theme: Theme;
  isDark: boolean;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// components/ThemeProvider.tsx
import React, { useState, ReactNode } from 'react';
import { ThemeContext } from '../hooks/useTheme';
import { lightTheme, darkTheme } from '../tokens/theme';

interface ThemeProviderProps {
  children: ReactNode;
  initialTheme?: 'light' | 'dark';
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({
  children,
  initialTheme = 'light',
}) => {
  const [isDark, setIsDark] = useState(initialTheme === 'dark');
  
  const theme = isDark ? darkTheme : lightTheme;
  
  const toggleTheme = () => setIsDark(!isDark);
  
  return (
    <ThemeContext.Provider value={{ theme, isDark, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

## 4. Creación de Estilos

### Estilos Base
```typescript
// styles/base.ts
import { StyleSheet } from 'react-native';
import { Theme } from '../tokens/theme';

export const createBaseStyles = (theme: Theme) => StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: theme.colors.background,
  },
  row: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  column: {
    flexDirection: 'column',
  },
  center: {
    justifyContent: 'center',
    alignItems: 'center',
  },
  spaceBetween: {
    justifyContent: 'space-between',
  },
  spaceAround: {
    justifyContent: 'space-around',
  },
  spaceEvenly: {
    justifyContent: 'space-evenly',
  },
  fullWidth: {
    width: '100%',
  },
  fullHeight: {
    height: '100%',
  },
});
```

### Estilos de Componentes
```typescript
// styles/components.ts
import { StyleSheet } from 'react-native';
import { Theme } from '../tokens/theme';

export const createComponentStyles = (theme: Theme) => StyleSheet.create({
  button: {
    paddingHorizontal: theme.spacing.md,
    paddingVertical: theme.spacing.sm,
    borderRadius: theme.borders.radius.md,
    backgroundColor: theme.colors.primary[500],
    ...theme.shadows.sm,
  },
  buttonText: {
    color: theme.colors.neutral[50],
    fontSize: theme.typography.fontSize.base,
    fontWeight: theme.typography.fontWeight.medium,
    textAlign: 'center',
  },
  input: {
    borderWidth: theme.borders.width.thin,
    borderColor: theme.colors.neutral[300],
    borderRadius: theme.borders.radius.md,
    paddingHorizontal: theme.spacing.md,
    paddingVertical: theme.spacing.sm,
    fontSize: theme.typography.fontSize.base,
    color: theme.colors.text,
    backgroundColor: theme.colors.surface,
  },
  card: {
    backgroundColor: theme.colors.surface,
    borderRadius: theme.borders.radius.lg,
    padding: theme.spacing.md,
    ...theme.shadows.md,
  },
  text: {
    color: theme.colors.text,
    fontSize: theme.typography.fontSize.base,
    lineHeight: theme.typography.fontSize.base * theme.typography.lineHeight.normal,
  },
  textSecondary: {
    color: theme.colors.textSecondary,
    fontSize: theme.typography.fontSize.sm,
  },
  heading: {
    color: theme.colors.text,
    fontSize: theme.typography.fontSize['2xl'],
    fontWeight: theme.typography.fontWeight.bold,
    lineHeight: theme.typography.fontSize['2xl'] * theme.typography.lineHeight.tight,
  },
});
```

## 5. Componentes con Estilos

### Button Component
```typescript
// components/Button.tsx
import React from 'react';
import { TouchableOpacity, Text, TouchableOpacityProps } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import { createComponentStyles } from '../styles/components';

interface ButtonProps extends TouchableOpacityProps {
  title: string;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
}

export const Button: React.FC<ButtonProps> = ({
  title,
  variant = 'primary',
  size = 'md',
  style,
  ...props
}) => {
  const { theme } = useTheme();
  const styles = createComponentStyles(theme);
  
  const getButtonStyle = () => {
    const baseStyle = [styles.button];
    
    switch (variant) {
      case 'secondary':
        return [...baseStyle, { backgroundColor: theme.colors.secondary[500] }];
      case 'outline':
        return [
          ...baseStyle,
          {
            backgroundColor: 'transparent',
            borderWidth: theme.borders.width.thin,
            borderColor: theme.colors.primary[500],
          },
        ];
      default:
        return baseStyle;
    }
  };
  
  const getTextStyle = () => {
    const baseStyle = [styles.buttonText];
    
    if (variant === 'outline') {
      return [...baseStyle, { color: theme.colors.primary[500] }];
    }
    
    return baseStyle;
  };
  
  return (
    <TouchableOpacity style={[getButtonStyle(), style]} {...props}>
      <Text style={getTextStyle()}>{title}</Text>
    </TouchableOpacity>
  );
};
```

### Input Component
```typescript
// components/Input.tsx
import React from 'react';
import { TextInput, TextInputProps, View, Text } from 'react-native';
import { useTheme } from '../hooks/useTheme';
import { createComponentStyles } from '../styles/components';

interface InputProps extends TextInputProps {
  label?: string;
  error?: string;
  helperText?: string;
}

export const Input: React.FC<InputProps> = ({
  label,
  error,
  helperText,
  style,
  ...props
}) => {
  const { theme } = useTheme();
  const styles = createComponentStyles(theme);
  
  const getInputStyle = () => {
    const baseStyle = [styles.input];
    
    if (error) {
      return [
        ...baseStyle,
        {
          borderColor: theme.colors.semantic.error,
          borderWidth: theme.borders.width.medium,
        },
      ];
    }
    
    return baseStyle;
  };
  
  return (
    <View style={{ marginBottom: theme.spacing.md }}>
      {label && (
        <Text style={[styles.text, { marginBottom: theme.spacing.xs }]}>
          {label}
        </Text>
      )}
      <TextInput
        style={[getInputStyle(), style]}
        placeholderTextColor={theme.colors.textSecondary}
        {...props}
      />
      {error && (
        <Text style={[styles.textSecondary, { color: theme.colors.semantic.error }]}>
          {error}
        </Text>
      )}
      {helperText && !error && (
        <Text style={styles.textSecondary}>{helperText}</Text>
      )}
    </View>
  );
};
```

## 6. Gestión de Temas

### Persistencia de Tema
```typescript
// utils/themeStorage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

const THEME_KEY = '@app_theme';

export const saveTheme = async (theme: 'light' | 'dark') => {
  try {
    await AsyncStorage.setItem(THEME_KEY, theme);
  } catch (error) {
    console.error('Error saving theme:', error);
  }
};

export const loadTheme = async (): Promise<'light' | 'dark'> => {
  try {
    const theme = await AsyncStorage.getItem(THEME_KEY);
    return theme === 'dark' ? 'dark' : 'light';
  } catch (error) {
    console.error('Error loading theme:', error);
    return 'light';
  }
};
```

### ThemeProvider Mejorado
```typescript
// components/ThemeProvider.tsx
import React, { useState, useEffect, ReactNode } from 'react';
import { ThemeContext } from '../hooks/useTheme';
import { lightTheme, darkTheme } from '../tokens/theme';
import { saveTheme, loadTheme } from '../utils/themeStorage';

interface ThemeProviderProps {
  children: ReactNode;
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [isDark, setIsDark] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    const initializeTheme = async () => {
      const savedTheme = await loadTheme();
      setIsDark(savedTheme === 'dark');
      setIsLoading(false);
    };
    
    initializeTheme();
  }, []);
  
  const toggleTheme = async () => {
    const newTheme = !isDark;
    setIsDark(newTheme);
    await saveTheme(newTheme ? 'dark' : 'light');
  };
  
  const theme = isDark ? darkTheme : lightTheme;
  
  if (isLoading) {
    return null; // o un loading spinner
  }
  
  return (
    <ThemeContext.Provider value={{ theme, isDark, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

## 7. Herramientas y Utilidades

### Generador de Estilos
```typescript
// utils/styleGenerator.ts
import { StyleSheet, ViewStyle, TextStyle } from 'react-native';
import { Theme } from '../tokens/theme';

export const createStyles = <T extends Record<string, ViewStyle | TextStyle>>(
  styles: (theme: Theme) => T
) => (theme: Theme) => StyleSheet.create(styles(theme));

// Uso
export const homeStyles = createStyles((theme) => ({
  container: {
    flex: 1,
    backgroundColor: theme.colors.background,
    padding: theme.spacing.md,
  },
  title: {
    fontSize: theme.typography.fontSize['2xl'],
    fontWeight: theme.typography.fontWeight.bold,
    color: theme.colors.text,
    marginBottom: theme.spacing.lg,
  },
}));
```

### Validación de Tokens
```typescript
// utils/tokenValidator.ts
import { colors, typography, spacing } from '../tokens';

export const validateTokens = () => {
  const errors: string[] = [];
  
  // Validar colores
  Object.entries(colors).forEach(([category, values]) => {
    if (typeof values !== 'object') {
      errors.push(`Color category ${category} is not an object`);
    }
  });
  
  // Validar tipografía
  if (typeof typography.fontSize !== 'object') {
    errors.push('Typography fontSize is not an object');
  }
  
  // Validar espaciado
  if (typeof spacing !== 'object') {
    errors.push('Spacing is not an object');
  }
  
  if (errors.length > 0) {
    console.error('Token validation errors:', errors);
    return false;
  }
  
  return true;
};
```

## 8. Mejores Prácticas

### Organización
- **Separación por categorías**: Colores, tipografía, espaciado, etc.
- **Nomenclatura consistente**: Usar nombres descriptivos y escalables
- **Documentación**: Comentar el propósito de cada token
- **Versionado**: Mantener compatibilidad hacia atrás

### Performance
- **Memoización**: Usar `useMemo` para estilos complejos
- **Lazy loading**: Cargar temas solo cuando sea necesario
- **Optimización**: Evitar recrear estilos en cada render

### Accesibilidad
- **Contraste**: Asegurar ratios de contraste adecuados
- **Tamaños**: Usar tamaños mínimos para texto y elementos táctiles
- **Colores**: No depender solo del color para transmitir información

## 9. Ejercicios Prácticos

### Ejercicio 1: Crear un Sistema de Tokens
```typescript
// Crear tokens para una aplicación de e-commerce
const ecommerceTokens = {
  colors: {
    brand: {
      primary: '#ff6b35',
      secondary: '#004e89',
    },
    status: {
      success: '#28a745',
      warning: '#ffc107',
      error: '#dc3545',
    },
  },
  // ... otros tokens
};
```

### Ejercicio 2: Implementar Tema Oscuro
```typescript
// Crear variaciones para tema oscuro
const darkTheme = {
  colors: {
    ...baseColors,
    background: '#1a1a1a',
    surface: '#2d2d2d',
    text: '#ffffff',
  },
  // ... otros tokens
};
```

### Ejercicio 3: Componente con Múltiples Variantes
```typescript
// Crear un componente Card con múltiples variantes
interface CardProps {
  variant: 'default' | 'elevated' | 'outlined';
  size: 'sm' | 'md' | 'lg';
  // ... otras props
}
```

## 10. Recursos y Herramientas

### Herramientas de Diseño
- **Figma**: Para crear y gestionar design tokens
- **Adobe XD**: Alternativa para diseño de tokens
- **Sketch**: Plugin para exportar tokens

### Librerías de Tokens
- **Style Dictionary**: Transformar tokens a múltiples formatos
- **Theo**: Generador de tokens de Salesforce
- **Design Tokens W3C**: Estándar para tokens

### Herramientas de Desarrollo
- **React Native Debugger**: Para inspeccionar estilos
- **Flipper**: Para debugging de temas
- **Storybook**: Para documentar componentes

## Conclusión

Los Design Tokens son fundamentales para crear sistemas de diseño escalables y mantenibles. Permiten:

- **Consistencia visual** en toda la aplicación
- **Facilidad de mantenimiento** y actualización
- **Colaboración efectiva** entre diseño y desarrollo
- **Soporte para múltiples temas** y variaciones

En la siguiente clase exploraremos la creación de componentes reutilizables y la arquitectura de sistemas de componentes.

## Tarea

1. Crear un sistema completo de Design Tokens para tu aplicación
2. Implementar un ThemeProvider con soporte para tema claro y oscuro
3. Crear al menos 3 componentes que utilicen el sistema de tokens
4. Implementar persistencia del tema seleccionado
5. Documentar todos los tokens creados

## Enlaces Útiles

- [Design Tokens W3C](https://design-tokens.github.io/community-group/)
- [Style Dictionary](https://amzn.github.io/style-dictionary/)
- [React Native Theming](https://reactnative.dev/docs/appearance)
- [Figma Design Tokens](https://www.figma.com/design-systems/design-tokens/)
