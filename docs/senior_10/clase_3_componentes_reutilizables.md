# Clase 3: Componentes Reutilizables

## Objetivos de la Clase
- Diseñar componentes reutilizables y escalables
- Implementar patrones de composición
- Crear sistemas de props flexibles
- Gestionar variantes y estados de componentes

## 1. Principios de Componentes Reutilizables

### Características Clave
- **Reutilización**: Usable en múltiples contextos
- **Flexibilidad**: Adaptable a diferentes necesidades
- **Consistencia**: Comportamiento predecible
- **Mantenibilidad**: Fácil de actualizar y modificar

### Beneficios
- **Reducción de código duplicado**
- **Consistencia en la UI**
- **Facilidad de mantenimiento**
- **Mejor experiencia de desarrollo**

## 2. Arquitectura de Componentes

### Estructura de Directorios
```
components/
├── base/           # Componentes base/fundamentales
│   ├── Button/
│   ├── Input/
│   ├── Text/
│   └── View/
├── composite/      # Componentes compuestos
│   ├── Card/
│   ├── Modal/
│   ├── Form/
│   └── List/
├── layout/         # Componentes de layout
│   ├── Container/
│   ├── Grid/
│   ├── Stack/
│   └── Flex/
└── domain/         # Componentes específicos del dominio
    ├── ProductCard/
    ├── UserProfile/
    └── OrderSummary/
```

### Convenciones de Nomenclatura
```typescript
// Componentes base: PascalCase
export const Button = () => {};

// Props interfaces: ComponentName + Props
interface ButtonProps {
  title: string;
  onPress: () => void;
}

// Variantes: camelCase
type ButtonVariant = 'primary' | 'secondary' | 'outline';

// Estados: camelCase
type ButtonState = 'default' | 'loading' | 'disabled';
```

## 3. Componente Base: Button

### Implementación Completa
```typescript
// components/base/Button/Button.tsx
import React from 'react';
import {
  TouchableOpacity,
  Text,
  ActivityIndicator,
  TouchableOpacityProps,
  ViewStyle,
  TextStyle,
} from 'react-native';
import { useTheme } from '../../../hooks/useTheme';
import { createButtonStyles } from './Button.styles';

export type ButtonVariant = 'primary' | 'secondary' | 'outline' | 'ghost';
export type ButtonSize = 'sm' | 'md' | 'lg';
export type ButtonState = 'default' | 'loading' | 'disabled';

interface ButtonProps extends Omit<TouchableOpacityProps, 'disabled'> {
  title: string;
  variant?: ButtonVariant;
  size?: ButtonSize;
  state?: ButtonState;
  loading?: boolean;
  disabled?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  fullWidth?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
}

export const Button: React.FC<ButtonProps> = ({
  title,
  variant = 'primary',
  size = 'md',
  state = 'default',
  loading = false,
  disabled = false,
  leftIcon,
  rightIcon,
  fullWidth = false,
  style,
  textStyle,
  onPress,
  ...props
}) => {
  const { theme } = useTheme();
  const styles = createButtonStyles(theme);
  
  const isDisabled = disabled || state === 'disabled' || loading;
  const isLoading = loading || state === 'loading';
  
  const getButtonStyle = (): ViewStyle[] => {
    const baseStyle = [styles.button];
    const variantStyle = styles[`button_${variant}`];
    const sizeStyle = styles[`button_${size}`];
    const stateStyle = isDisabled ? styles.button_disabled : {};
    const widthStyle = fullWidth ? styles.button_fullWidth : {};
    
    return [
      baseStyle,
      variantStyle,
      sizeStyle,
      stateStyle,
      widthStyle,
      style,
    ];
  };
  
  const getTextStyle = (): TextStyle[] => {
    const baseStyle = [styles.buttonText];
    const variantStyle = styles[`buttonText_${variant}`];
    const sizeStyle = styles[`buttonText_${size}`];
    const stateStyle = isDisabled ? styles.buttonText_disabled : {};
    
    return [
      baseStyle,
      variantStyle,
      sizeStyle,
      stateStyle,
      textStyle,
    ];
  };
  
  const handlePress = (event: any) => {
    if (!isDisabled && onPress) {
      onPress(event);
    }
  };
  
  return (
    <TouchableOpacity
      style={getButtonStyle()}
      onPress={handlePress}
      disabled={isDisabled}
      activeOpacity={isDisabled ? 1 : 0.7}
      {...props}
    >
      {isLoading ? (
        <ActivityIndicator
          size="small"
          color={variant === 'outline' || variant === 'ghost' 
            ? theme.colors.primary[500] 
            : theme.colors.neutral[50]
          }
        />
      ) : (
        <>
          {leftIcon && <>{leftIcon}</>}
          <Text style={getTextStyle()}>{title}</Text>
          {rightIcon && <>{rightIcon}</>}
        </>
      )}
    </TouchableOpacity>
  );
};
```

### Estilos del Button
```typescript
// components/base/Button/Button.styles.ts
import { StyleSheet, ViewStyle, TextStyle } from 'react-native';
import { Theme } from '../../../tokens/theme';

export const createButtonStyles = (theme: Theme) => StyleSheet.create({
  // Base styles
  button: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: theme.borders.radius.md,
    ...theme.shadows.sm,
  },
  buttonText: {
    fontWeight: theme.typography.fontWeight.medium,
    textAlign: 'center',
  },
  
  // Variants
  button_primary: {
    backgroundColor: theme.colors.primary[500],
  },
  button_secondary: {
    backgroundColor: theme.colors.secondary[500],
  },
  button_outline: {
    backgroundColor: 'transparent',
    borderWidth: theme.borders.width.thin,
    borderColor: theme.colors.primary[500],
  },
  button_ghost: {
    backgroundColor: 'transparent',
  },
  
  // Text variants
  buttonText_primary: {
    color: theme.colors.neutral[50],
  },
  buttonText_secondary: {
    color: theme.colors.neutral[50],
  },
  buttonText_outline: {
    color: theme.colors.primary[500],
  },
  buttonText_ghost: {
    color: theme.colors.primary[500],
  },
  
  // Sizes
  button_sm: {
    paddingHorizontal: theme.spacing.sm,
    paddingVertical: theme.spacing.xs,
    minHeight: 32,
  },
  button_md: {
    paddingHorizontal: theme.spacing.md,
    paddingVertical: theme.spacing.sm,
    minHeight: 40,
  },
  button_lg: {
    paddingHorizontal: theme.spacing.lg,
    paddingVertical: theme.spacing.md,
    minHeight: 48,
  },
  
  // Text sizes
  buttonText_sm: {
    fontSize: theme.typography.fontSize.sm,
  },
  buttonText_md: {
    fontSize: theme.typography.fontSize.base,
  },
  buttonText_lg: {
    fontSize: theme.typography.fontSize.lg,
  },
  
  // States
  button_disabled: {
    opacity: 0.5,
  },
  buttonText_disabled: {
    opacity: 0.7,
  },
  
  // Layout
  button_fullWidth: {
    width: '100%',
  },
});
```

### Índice del Componente
```typescript
// components/base/Button/index.ts
export { Button } from './Button';
export type { ButtonProps, ButtonVariant, ButtonSize, ButtonState } from './Button';
export { createButtonStyles } from './Button.styles';
```

## 4. Componente Compuesto: Card

### Implementación
```typescript
// components/composite/Card/Card.tsx
import React from 'react';
import { View, ViewProps, ViewStyle } from 'react-native';
import { useTheme } from '../../../hooks/useTheme';
import { createCardStyles } from './Card.styles';

export type CardVariant = 'default' | 'elevated' | 'outlined' | 'filled';
export type CardSize = 'sm' | 'md' | 'lg';

interface CardProps extends ViewProps {
  variant?: CardVariant;
  size?: CardSize;
  padding?: boolean;
  margin?: boolean;
  children: React.ReactNode;
  style?: ViewStyle;
}

export const Card: React.FC<CardProps> = ({
  variant = 'default',
  size = 'md',
  padding = true,
  margin = false,
  children,
  style,
  ...props
}) => {
  const { theme } = useTheme();
  const styles = createCardStyles(theme);
  
  const getCardStyle = (): ViewStyle[] => {
    const baseStyle = [styles.card];
    const variantStyle = styles[`card_${variant}`];
    const sizeStyle = styles[`card_${size}`];
    const paddingStyle = padding ? styles.card_padding : {};
    const marginStyle = margin ? styles.card_margin : {};
    
    return [
      baseStyle,
      variantStyle,
      sizeStyle,
      paddingStyle,
      marginStyle,
      style,
    ];
  };
  
  return (
    <View style={getCardStyle()} {...props}>
      {children}
    </View>
  );
};

// Subcomponentes
export const CardHeader: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme } = useTheme();
  const styles = createCardStyles(theme);
  
  return <View style={styles.cardHeader}>{children}</View>;
};

export const CardBody: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme } = useTheme();
  const styles = createCardStyles(theme);
  
  return <View style={styles.cardBody}>{children}</View>;
};

export const CardFooter: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme } = useTheme();
  const styles = createCardStyles(theme);
  
  return <View style={styles.cardFooter}>{children}</View>;
};
```

### Estilos del Card
```typescript
// components/composite/Card/Card.styles.ts
import { StyleSheet, ViewStyle } from 'react-native';
import { Theme } from '../../../tokens/theme';

export const createCardStyles = (theme: Theme) => StyleSheet.create({
  // Base card
  card: {
    borderRadius: theme.borders.radius.lg,
    backgroundColor: theme.colors.surface,
  },
  
  // Variants
  card_default: {
    ...theme.shadows.sm,
  },
  card_elevated: {
    ...theme.shadows.lg,
  },
  card_outlined: {
    borderWidth: theme.borders.width.thin,
    borderColor: theme.colors.neutral[200],
  },
  card_filled: {
    backgroundColor: theme.colors.neutral[100],
  },
  
  // Sizes
  card_sm: {
    borderRadius: theme.borders.radius.md,
  },
  card_md: {
    borderRadius: theme.borders.radius.lg,
  },
  card_lg: {
    borderRadius: theme.borders.radius.xl,
  },
  
  // Layout
  card_padding: {
    padding: theme.spacing.md,
  },
  card_margin: {
    margin: theme.spacing.sm,
  },
  
  // Subcomponents
  cardHeader: {
    marginBottom: theme.spacing.sm,
  },
  cardBody: {
    flex: 1,
  },
  cardFooter: {
    marginTop: theme.spacing.sm,
    paddingTop: theme.spacing.sm,
    borderTopWidth: theme.borders.width.thin,
    borderTopColor: theme.colors.neutral[200],
  },
});
```

## 5. Componente de Layout: Container

### Implementación
```typescript
// components/layout/Container/Container.tsx
import React from 'react';
import { View, ViewProps, ViewStyle } from 'react-native';
import { useTheme } from '../../../hooks/useTheme';
import { createContainerStyles } from './Container.styles';

export type ContainerVariant = 'default' | 'fluid' | 'centered';
export type ContainerPadding = 'none' | 'sm' | 'md' | 'lg';

interface ContainerProps extends ViewProps {
  variant?: ContainerVariant;
  padding?: ContainerPadding;
  maxWidth?: number;
  children: React.ReactNode;
  style?: ViewStyle;
}

export const Container: React.FC<ContainerProps> = ({
  variant = 'default',
  padding = 'md',
  maxWidth,
  children,
  style,
  ...props
}) => {
  const { theme } = useTheme();
  const styles = createContainerStyles(theme);
  
  const getContainerStyle = (): ViewStyle[] => {
    const baseStyle = [styles.container];
    const variantStyle = styles[`container_${variant}`];
    const paddingStyle = styles[`container_padding_${padding}`];
    const maxWidthStyle = maxWidth ? { maxWidth } : {};
    
    return [
      baseStyle,
      variantStyle,
      paddingStyle,
      maxWidthStyle,
      style,
    ];
  };
  
  return (
    <View style={getContainerStyle()} {...props}>
      {children}
    </View>
  );
};
```

## 6. Patrones de Composición

### Compound Components
```typescript
// components/composite/Modal/Modal.tsx
import React, { createContext, useContext } from 'react';
import { Modal as RNModal, View, ViewProps } from 'react-native';
import { useTheme } from '../../../hooks/useTheme';
import { createModalStyles } from './Modal.styles';

interface ModalContextType {
  isOpen: boolean;
  onClose: () => void;
}

const ModalContext = createContext<ModalContextType | undefined>(undefined);

const useModalContext = () => {
  const context = useContext(ModalContext);
  if (!context) {
    throw new Error('Modal components must be used within Modal');
  }
  return context;
};

interface ModalProps extends ViewProps {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
}

export const Modal: React.FC<ModalProps> = ({ isOpen, onClose, children, ...props }) => {
  const { theme } = useTheme();
  const styles = createModalStyles(theme);
  
  return (
    <ModalContext.Provider value={{ isOpen, onClose }}>
      <RNModal
        visible={isOpen}
        transparent
        animationType="fade"
        onRequestClose={onClose}
      >
        <View style={styles.overlay}>
          <View style={[styles.modal, props.style]} {...props}>
            {children}
          </View>
        </View>
      </RNModal>
    </ModalContext.Provider>
  );
};

// Subcomponentes
export const ModalHeader: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme } = useTheme();
  const styles = createModalStyles(theme);
  
  return <View style={styles.modalHeader}>{children}</View>;
};

export const ModalBody: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme } = useTheme();
  const styles = createModalStyles(theme);
  
  return <View style={styles.modalBody}>{children}</View>;
};

export const ModalFooter: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { theme } = useTheme();
  const styles = createModalStyles(theme);
  
  return <View style={styles.modalFooter}>{children}</View>;
};

export const ModalCloseButton: React.FC<{ onPress?: () => void }> = ({ onPress }) => {
  const { onClose } = useModalContext();
  const { theme } = useTheme();
  const styles = createModalStyles(theme);
  
  return (
    <Button
      variant="ghost"
      size="sm"
      onPress={onPress || onClose}
      style={styles.closeButton}
    >
      ×
    </Button>
  );
};
```

### Render Props Pattern
```typescript
// components/base/DataList/DataList.tsx
import React from 'react';
import { FlatList, FlatListProps, View } from 'react-native';

interface DataListProps<T> extends Omit<FlatListProps<T>, 'renderItem'> {
  data: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  emptyComponent?: React.ReactNode;
  loadingComponent?: React.ReactNode;
  isLoading?: boolean;
}

export const DataList = <T,>({
  data,
  renderItem,
  emptyComponent,
  loadingComponent,
  isLoading = false,
  ...props
}: DataListProps<T>) => {
  if (isLoading) {
    return <View>{loadingComponent}</View>;
  }
  
  if (data.length === 0) {
    return <View>{emptyComponent}</View>;
  }
  
  return (
    <FlatList
      data={data}
      renderItem={({ item, index }) => renderItem(item, index)}
      {...props}
    />
  );
};
```

## 7. Hooks Personalizados para Componentes

### useComponentState
```typescript
// hooks/useComponentState.ts
import { useState, useCallback } from 'react';

export type ComponentState = 'default' | 'loading' | 'success' | 'error';

interface UseComponentStateReturn {
  state: ComponentState;
  isLoading: boolean;
  isSuccess: boolean;
  isError: boolean;
  setLoading: () => void;
  setSuccess: () => void;
  setError: () => void;
  setDefault: () => void;
  reset: () => void;
}

export const useComponentState = (initialState: ComponentState = 'default'): UseComponentStateReturn => {
  const [state, setState] = useState<ComponentState>(initialState);
  
  const setLoading = useCallback(() => setState('loading'), []);
  const setSuccess = useCallback(() => setState('success'), []);
  const setError = useCallback(() => setState('error'), []);
  const setDefault = useCallback(() => setState('default'), []);
  const reset = useCallback(() => setState(initialState), [initialState]);
  
  return {
    state,
    isLoading: state === 'loading',
    isSuccess: state === 'success',
    isError: state === 'error',
    setLoading,
    setSuccess,
    setError,
    setDefault,
    reset,
  };
};
```

### useComponentSize
```typescript
// hooks/useComponentSize.ts
import { useState, useCallback } from 'react';
import { LayoutChangeEvent } from 'react-native';

interface ComponentSize {
  width: number;
  height: number;
}

export const useComponentSize = () => {
  const [size, setSize] = useState<ComponentSize>({ width: 0, height: 0 });
  
  const onLayout = useCallback((event: LayoutChangeEvent) => {
    const { width, height } = event.nativeEvent.layout;
    setSize({ width, height });
  }, []);
  
  return { size, onLayout };
};
```

## 8. Testing de Componentes

### Test del Button
```typescript
// components/base/Button/__tests__/Button.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { Button } from '../Button';

describe('Button', () => {
  it('renders correctly', () => {
    const { getByText } = render(<Button title="Test Button" />);
    expect(getByText('Test Button')).toBeTruthy();
  });
  
  it('calls onPress when pressed', () => {
    const onPress = jest.fn();
    const { getByText } = render(
      <Button title="Test Button" onPress={onPress} />
    );
    
    fireEvent.press(getByText('Test Button'));
    expect(onPress).toHaveBeenCalledTimes(1);
  });
  
  it('does not call onPress when disabled', () => {
    const onPress = jest.fn();
    const { getByText } = render(
      <Button title="Test Button" onPress={onPress} disabled />
    );
    
    fireEvent.press(getByText('Test Button'));
    expect(onPress).not.toHaveBeenCalled();
  });
  
  it('shows loading state', () => {
    const { getByTestId } = render(
      <Button title="Test Button" loading />
    );
    
    expect(getByTestId('activity-indicator')).toBeTruthy();
  });
});
```

## 9. Documentación de Componentes

### Storybook Stories
```typescript
// components/base/Button/Button.stories.tsx
import React from 'react';
import { Story, Meta } from '@storybook/react-native';
import { Button, ButtonProps } from './Button';

export default {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['primary', 'secondary', 'outline', 'ghost'],
    },
    size: {
      control: { type: 'select' },
      options: ['sm', 'md', 'lg'],
    },
    state: {
      control: { type: 'select' },
      options: ['default', 'loading', 'disabled'],
    },
  },
} as Meta;

const Template: Story<ButtonProps> = (args) => <Button {...args} />;

export const Primary = Template.bind({});
Primary.args = {
  title: 'Primary Button',
  variant: 'primary',
};

export const Secondary = Template.bind({});
Secondary.args = {
  title: 'Secondary Button',
  variant: 'secondary',
};

export const Outline = Template.bind({});
Outline.args = {
  title: 'Outline Button',
  variant: 'outline',
};

export const Loading = Template.bind({});
Loading.args = {
  title: 'Loading Button',
  loading: true,
};

export const Disabled = Template.bind({});
Disabled.args = {
  title: 'Disabled Button',
  disabled: true,
};
```

## 10. Mejores Prácticas

### Diseño de Props
- **Props opcionales**: Usar valores por defecto sensatos
- **Tipos específicos**: Evitar `any` y usar tipos específicos
- **Props compuestas**: Agrupar props relacionadas en objetos
- **Forwarding de props**: Usar `...props` para flexibilidad

### Performance
- **Memoización**: Usar `React.memo` para componentes pesados
- **Lazy loading**: Cargar componentes solo cuando sea necesario
- **Optimización de re-renders**: Evitar recrear objetos en cada render

### Accesibilidad
- **Labels**: Proporcionar labels descriptivos
- **Estados**: Indicar estados claramente
- **Navegación**: Soporte para navegación por teclado
- **Contraste**: Asegurar contraste adecuado

## Conclusión

Los componentes reutilizables son la base de un sistema de diseño escalable. Permiten:

- **Consistencia** en toda la aplicación
- **Eficiencia** en el desarrollo
- **Mantenibilidad** del código
- **Escalabilidad** del sistema

En la siguiente clase exploraremos la arquitectura de sistemas de componentes y la gestión de estado global.

## Tarea

1. Crear al menos 5 componentes base reutilizables
2. Implementar un componente compuesto usando el patrón de composición
3. Crear hooks personalizados para gestión de estado de componentes
4. Escribir tests para todos los componentes creados
5. Documentar los componentes usando Storybook

## Enlaces Útiles

- [React Native Components](https://reactnative.dev/docs/components-and-apis)
- [React Component Patterns](https://reactpatterns.com/)
- [Storybook for React Native](https://storybook.js.org/docs/react-native/get-started/introduction)
- [Testing Library React Native](https://callstack.github.io/react-native-testing-library/)
