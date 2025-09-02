# Clase 4: Responsive Design Multiplataforma

## Objetivos de la Clase
- Implementar breakpoints y media queries
- Crear layouts adaptativos para diferentes pantallas
- Desarrollar componentes responsive y flexibles
- Optimizar la experiencia de usuario en múltiples dispositivos

## 1. Breakpoints y Media Queries

### Sistema de Breakpoints
```typescript
// shared/constants/breakpoints.ts
export const BREAKPOINTS = {
  xs: 0,
  sm: 576,
  md: 768,
  lg: 992,
  xl: 1200,
  xxl: 1400,
} as const;

export type Breakpoint = keyof typeof BREAKPOINTS;

export const getBreakpoint = (width: number): Breakpoint => {
  if (width >= BREAKPOINTS.xxl) return 'xxl';
  if (width >= BREAKPOINTS.xl) return 'xl';
  if (width >= BREAKPOINTS.lg) return 'lg';
  if (width >= BREAKPOINTS.md) return 'md';
  if (width >= BREAKPOINTS.sm) return 'sm';
  return 'xs';
};
```

### Hook de Breakpoints
```typescript
// shared/hooks/useBreakpoints.ts
import { useState, useEffect } from 'react';
import { Dimensions } from 'react-native';
import { BREAKPOINTS, getBreakpoint, Breakpoint } from '../constants/breakpoints';

export const useBreakpoints = () => {
  const [breakpoint, setBreakpoint] = useState<Breakpoint>('xs');
  const [dimensions, setDimensions] = useState(Dimensions.get('window'));

  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => {
      setDimensions(window);
      setBreakpoint(getBreakpoint(window.width));
    });

    return () => subscription?.remove();
  }, []);

  const { width, height } = dimensions;

  return {
    breakpoint,
    width,
    height,
    isXs: breakpoint === 'xs',
    isSm: breakpoint === 'sm',
    isMd: breakpoint === 'md',
    isLg: breakpoint === 'lg',
    isXl: breakpoint === 'xl',
    isXxl: breakpoint === 'xxl',
    isMobile: width < BREAKPOINTS.md,
    isTablet: width >= BREAKPOINTS.md && width < BREAKPOINTS.lg,
    isDesktop: width >= BREAKPOINTS.lg,
    isLandscape: width > height,
    isPortrait: height > width,
  };
};
```

### Media Queries para Web
```typescript
// shared/utils/mediaQueries.ts
import { Platform } from 'react-native';

export const MediaQueries = {
  // Crear media query para web
  createMediaQuery: (breakpoint: string) => {
    if (Platform.OS === 'web') {
      return `@media (min-width: ${breakpoint}px)`;
    }
    return '';
  },

  // Verificar si el breakpoint está activo
  isBreakpointActive: (width: number, breakpoint: string) => {
    return width >= parseInt(breakpoint);
  },

  // Obtener estilos específicos para breakpoint
  getResponsiveStyles: (styles: any, breakpoint: string) => {
    if (Platform.OS === 'web') {
      return {
        [`@media (min-width: ${breakpoint}px)`]: styles,
      };
    }
    return styles;
  },
};
```

## 2. Layouts Adaptativos

### Container Responsive
```typescript
// shared/components/Container.tsx
import React from 'react';
import { View, StyleSheet, ViewStyle } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';
import { BREAKPOINTS } from '../constants/breakpoints';

interface ContainerProps {
  children: React.ReactNode;
  fluid?: boolean;
  maxWidth?: number;
  padding?: number | { horizontal: number; vertical: number };
  style?: ViewStyle;
}

export const Container: React.FC<ContainerProps> = ({
  children,
  fluid = false,
  maxWidth,
  padding = 16,
  style,
}) => {
  const { width, isMobile, isTablet, isDesktop } = useBreakpoints();

  const getContainerStyle = (): ViewStyle => {
    const baseStyle: ViewStyle = {
      flex: 1,
    };

    if (!fluid) {
      baseStyle.maxWidth = maxWidth || getMaxWidth();
      baseStyle.alignSelf = 'center';
    }

    if (typeof padding === 'number') {
      baseStyle.paddingHorizontal = padding;
      baseStyle.paddingVertical = padding;
    } else {
      baseStyle.paddingHorizontal = padding.horizontal;
      baseStyle.paddingVertical = padding.vertical;
    }

    return baseStyle;
  };

  const getMaxWidth = (): number => {
    if (isMobile) return width;
    if (isTablet) return BREAKPOINTS.lg;
    if (isDesktop) return BREAKPOINTS.xl;
    return BREAKPOINTS.xxl;
  };

  return (
    <View style={[styles.container, getContainerStyle(), style]}>
      {children}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    width: '100%',
  },
});
```

### Grid System
```typescript
// shared/components/Grid.tsx
import React from 'react';
import { View, StyleSheet, ViewStyle } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';

interface GridProps {
  children: React.ReactNode;
  columns?: number | { xs?: number; sm?: number; md?: number; lg?: number; xl?: number };
  gap?: number;
  style?: ViewStyle;
}

export const Grid: React.FC<GridProps> = ({
  children,
  columns = 1,
  gap = 16,
  style,
}) => {
  const { isXs, isSm, isMd, isLg, isXl } = useBreakpoints();

  const getColumns = (): number => {
    if (typeof columns === 'number') {
      return columns;
    }

    if (isXl && columns.xl) return columns.xl;
    if (isLg && columns.lg) return columns.lg;
    if (isMd && columns.md) return columns.md;
    if (isSm && columns.sm) return columns.sm;
    if (isXs && columns.xs) return columns.xs;

    return 1;
  };

  const getGridStyle = (): ViewStyle => {
    const cols = getColumns();
    return {
      flexDirection: 'row',
      flexWrap: 'wrap',
      marginHorizontal: -gap / 2,
    };
  };

  const getItemStyle = (): ViewStyle => {
    const cols = getColumns();
    return {
      width: `${100 / cols}%`,
      paddingHorizontal: gap / 2,
      marginBottom: gap,
    };
  };

  return (
    <View style={[styles.grid, getGridStyle(), style]}>
      {React.Children.map(children, (child, index) => (
        <View key={index} style={getItemStyle()}>
          {child}
        </View>
      ))}
    </View>
  );
};

const styles = StyleSheet.create({
  grid: {
    flex: 1,
  },
});
```

### Flex Layout
```typescript
// shared/components/Flex.tsx
import React from 'react';
import { View, StyleSheet, ViewStyle } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';

interface FlexProps {
  children: React.ReactNode;
  direction?: 'row' | 'column' | 'row-reverse' | 'column-reverse';
  wrap?: 'wrap' | 'nowrap' | 'wrap-reverse';
  justify?: 'flex-start' | 'flex-end' | 'center' | 'space-between' | 'space-around' | 'space-evenly';
  align?: 'flex-start' | 'flex-end' | 'center' | 'stretch' | 'baseline';
  gap?: number;
  responsive?: {
    direction?: { xs?: string; sm?: string; md?: string; lg?: string; xl?: string };
    wrap?: { xs?: string; sm?: string; md?: string; lg?: string; xl?: string };
    justify?: { xs?: string; sm?: string; md?: string; lg?: string; xl?: string };
    align?: { xs?: string; sm?: string; md?: string; lg?: string; xl?: string };
  };
  style?: ViewStyle;
}

export const Flex: React.FC<FlexProps> = ({
  children,
  direction = 'row',
  wrap = 'nowrap',
  justify = 'flex-start',
  align = 'flex-start',
  gap = 0,
  responsive,
  style,
}) => {
  const { isXs, isSm, isMd, isLg, isXl } = useBreakpoints();

  const getResponsiveValue = (prop: any, breakpoint: string) => {
    if (!responsive || !responsive[prop]) return null;
    return responsive[prop][breakpoint];
  };

  const getFlexStyle = (): ViewStyle => {
    let flexDirection = direction;
    let flexWrap = wrap;
    let justifyContent = justify;
    let alignItems = align;

    if (responsive) {
      if (isXl && getResponsiveValue('direction', 'xl')) {
        flexDirection = getResponsiveValue('direction', 'xl') as any;
      } else if (isLg && getResponsiveValue('direction', 'lg')) {
        flexDirection = getResponsiveValue('direction', 'lg') as any;
      } else if (isMd && getResponsiveValue('direction', 'md')) {
        flexDirection = getResponsiveValue('direction', 'md') as any;
      } else if (isSm && getResponsiveValue('direction', 'sm')) {
        flexDirection = getResponsiveValue('direction', 'sm') as any;
      } else if (isXs && getResponsiveValue('direction', 'xs')) {
        flexDirection = getResponsiveValue('direction', 'xs') as any;
      }

      if (isXl && getResponsiveValue('wrap', 'xl')) {
        flexWrap = getResponsiveValue('wrap', 'xl') as any;
      } else if (isLg && getResponsiveValue('wrap', 'lg')) {
        flexWrap = getResponsiveValue('wrap', 'lg') as any;
      } else if (isMd && getResponsiveValue('wrap', 'md')) {
        flexWrap = getResponsiveValue('wrap', 'md') as any;
      } else if (isSm && getResponsiveValue('wrap', 'sm')) {
        flexWrap = getResponsiveValue('wrap', 'sm') as any;
      } else if (isXs && getResponsiveValue('wrap', 'xs')) {
        flexWrap = getResponsiveValue('wrap', 'xs') as any;
      }

      if (isXl && getResponsiveValue('justify', 'xl')) {
        justifyContent = getResponsiveValue('justify', 'xl') as any;
      } else if (isLg && getResponsiveValue('justify', 'lg')) {
        justifyContent = getResponsiveValue('justify', 'lg') as any;
      } else if (isMd && getResponsiveValue('justify', 'md')) {
        justifyContent = getResponsiveValue('justify', 'md') as any;
      } else if (isSm && getResponsiveValue('justify', 'sm')) {
        justifyContent = getResponsiveValue('justify', 'sm') as any;
      } else if (isXs && getResponsiveValue('justify', 'xs')) {
        justifyContent = getResponsiveValue('justify', 'xs') as any;
      }

      if (isXl && getResponsiveValue('align', 'xl')) {
        alignItems = getResponsiveValue('align', 'xl') as any;
      } else if (isLg && getResponsiveValue('align', 'lg')) {
        alignItems = getResponsiveValue('align', 'lg') as any;
      } else if (isMd && getResponsiveValue('align', 'md')) {
        alignItems = getResponsiveValue('align', 'md') as any;
      } else if (isSm && getResponsiveValue('align', 'sm')) {
        alignItems = getResponsiveValue('align', 'sm') as any;
      } else if (isXs && getResponsiveValue('align', 'xs')) {
        alignItems = getResponsiveValue('align', 'xs') as any;
      }
    }

    return {
      flexDirection,
      flexWrap,
      justifyContent,
      alignItems,
      gap,
    };
  };

  return (
    <View style={[styles.flex, getFlexStyle(), style]}>
      {children}
    </View>
  );
};

const styles = StyleSheet.create({
  flex: {
    flex: 1,
  },
});
```

## 3. Componentes Responsive

### Text Responsive
```typescript
// shared/components/ResponsiveText.tsx
import React from 'react';
import { Text, StyleSheet, TextStyle } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';

interface ResponsiveTextProps {
  children: React.ReactNode;
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | 'xxl';
  weight?: 'normal' | 'medium' | 'semibold' | 'bold';
  color?: string;
  align?: 'left' | 'center' | 'right' | 'justify';
  responsive?: {
    size?: { xs?: string; sm?: string; md?: string; lg?: string; xl?: string };
    weight?: { xs?: string; sm?: string; md?: string; lg?: string; xl?: string };
  };
  style?: TextStyle;
}

export const ResponsiveText: React.FC<ResponsiveTextProps> = ({
  children,
  size = 'md',
  weight = 'normal',
  color = '#000',
  align = 'left',
  responsive,
  style,
}) => {
  const { isXs, isSm, isMd, isLg, isXl } = useBreakpoints();

  const getResponsiveValue = (prop: any, breakpoint: string) => {
    if (!responsive || !responsive[prop]) return null;
    return responsive[prop][breakpoint];
  };

  const getTextStyle = (): TextStyle => {
    let fontSize = getFontSize(size);
    let fontWeight = getFontWeight(weight);

    if (responsive) {
      if (isXl && getResponsiveValue('size', 'xl')) {
        fontSize = getFontSize(getResponsiveValue('size', 'xl'));
      } else if (isLg && getResponsiveValue('size', 'lg')) {
        fontSize = getFontSize(getResponsiveValue('size', 'lg'));
      } else if (isMd && getResponsiveValue('size', 'md')) {
        fontSize = getFontSize(getResponsiveValue('size', 'md'));
      } else if (isSm && getResponsiveValue('size', 'sm')) {
        fontSize = getFontSize(getResponsiveValue('size', 'sm'));
      } else if (isXs && getResponsiveValue('size', 'xs')) {
        fontSize = getFontSize(getResponsiveValue('size', 'xs'));
      }

      if (isXl && getResponsiveValue('weight', 'xl')) {
        fontWeight = getFontWeight(getResponsiveValue('weight', 'xl'));
      } else if (isLg && getResponsiveValue('weight', 'lg')) {
        fontWeight = getFontWeight(getResponsiveValue('weight', 'lg'));
      } else if (isMd && getResponsiveValue('weight', 'md')) {
        fontWeight = getFontWeight(getResponsiveValue('weight', 'md'));
      } else if (isSm && getResponsiveValue('weight', 'sm')) {
        fontWeight = getFontWeight(getResponsiveValue('weight', 'sm'));
      } else if (isXs && getResponsiveValue('weight', 'xs')) {
        fontWeight = getFontWeight(getResponsiveValue('weight', 'xs'));
      }
    }

    return {
      fontSize,
      fontWeight,
      color,
      textAlign: align,
    };
  };

  const getFontSize = (size: string): number => {
    const sizes = {
      xs: 12,
      sm: 14,
      md: 16,
      lg: 18,
      xl: 20,
      xxl: 24,
    };
    return sizes[size as keyof typeof sizes] || sizes.md;
  };

  const getFontWeight = (weight: string): string => {
    const weights = {
      normal: '400',
      medium: '500',
      semibold: '600',
      bold: '700',
    };
    return weights[weight as keyof typeof weights] || weights.normal;
  };

  return (
    <Text style={[styles.text, getTextStyle(), style]}>
      {children}
    </Text>
  );
};

const styles = StyleSheet.create({
  text: {
    lineHeight: 1.5,
  },
});
```

### Image Responsive
```typescript
// shared/components/ResponsiveImage.tsx
import React from 'react';
import { Image, StyleSheet, ImageStyle, View } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';

interface ResponsiveImageProps {
  source: { uri: string } | number;
  alt?: string;
  width?: number | string;
  height?: number | string;
  aspectRatio?: number;
  responsive?: {
    width?: { xs?: number | string; sm?: number | string; md?: number | string; lg?: number | string; xl?: number | string };
    height?: { xs?: number | string; sm?: number | string; md?: number | string; lg?: number | string; xl?: number | string };
  };
  style?: ImageStyle;
}

export const ResponsiveImage: React.FC<ResponsiveImageProps> = ({
  source,
  alt,
  width,
  height,
  aspectRatio,
  responsive,
  style,
}) => {
  const { isXs, isSm, isMd, isLg, isXl } = useBreakpoints();

  const getResponsiveValue = (prop: any, breakpoint: string) => {
    if (!responsive || !responsive[prop]) return null;
    return responsive[prop][breakpoint];
  };

  const getImageStyle = (): ImageStyle => {
    let imageWidth = width;
    let imageHeight = height;

    if (responsive) {
      if (isXl && getResponsiveValue('width', 'xl')) {
        imageWidth = getResponsiveValue('width', 'xl');
      } else if (isLg && getResponsiveValue('width', 'lg')) {
        imageWidth = getResponsiveValue('width', 'lg');
      } else if (isMd && getResponsiveValue('width', 'md')) {
        imageWidth = getResponsiveValue('width', 'md');
      } else if (isSm && getResponsiveValue('width', 'sm')) {
        imageWidth = getResponsiveValue('width', 'sm');
      } else if (isXs && getResponsiveValue('width', 'xs')) {
        imageWidth = getResponsiveValue('width', 'xs');
      }

      if (isXl && getResponsiveValue('height', 'xl')) {
        imageHeight = getResponsiveValue('height', 'xl');
      } else if (isLg && getResponsiveValue('height', 'lg')) {
        imageHeight = getResponsiveValue('height', 'lg');
      } else if (isMd && getResponsiveValue('height', 'md')) {
        imageHeight = getResponsiveValue('height', 'md');
      } else if (isSm && getResponsiveValue('height', 'sm')) {
        imageHeight = getResponsiveValue('height', 'sm');
      } else if (isXs && getResponsiveValue('height', 'xs')) {
        imageHeight = getResponsiveValue('height', 'xs');
      }
    }

    return {
      width: imageWidth,
      height: imageHeight,
      aspectRatio,
    };
  };

  return (
    <View style={styles.container}>
      <Image
        source={source}
        style={[styles.image, getImageStyle(), style]}
        accessibilityLabel={alt}
        resizeMode="contain"
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    overflow: 'hidden',
  },
  image: {
    width: '100%',
    height: '100%',
  },
});
```

## 4. Testing Responsive

### Test de Breakpoints
```typescript
// __tests__/shared/hooks/useBreakpoints.test.ts
import { renderHook } from '@testing-library/react-hooks';
import { useBreakpoints } from '../../../shared/hooks/useBreakpoints';

// Mock Dimensions
jest.mock('react-native/Libraries/Utilities/Dimensions', () => ({
  get: jest.fn(() => ({ width: 375, height: 667 })),
  addEventListener: jest.fn(() => ({ remove: jest.fn() })),
}));

describe('useBreakpoints', () => {
  it('returns correct breakpoint for mobile', () => {
    const { result } = renderHook(() => useBreakpoints());

    expect(result.current.breakpoint).toBe('xs');
    expect(result.current.isMobile).toBe(true);
    expect(result.current.isTablet).toBe(false);
    expect(result.current.isDesktop).toBe(false);
  });

  it('returns correct breakpoint for tablet', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Dimensions').get)
      .mockReturnValue({ width: 768, height: 1024 });

    const { result } = renderHook(() => useBreakpoints());

    expect(result.current.breakpoint).toBe('md');
    expect(result.current.isMobile).toBe(false);
    expect(result.current.isTablet).toBe(true);
    expect(result.current.isDesktop).toBe(false);
  });

  it('returns correct breakpoint for desktop', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Dimensions').get)
      .mockReturnValue({ width: 1200, height: 800 });

    const { result } = renderHook(() => useBreakpoints());

    expect(result.current.breakpoint).toBe('xl');
    expect(result.current.isMobile).toBe(false);
    expect(result.current.isTablet).toBe(false);
    expect(result.current.isDesktop).toBe(true);
  });
});
```

### Test de Componentes Responsive
```typescript
// __tests__/shared/components/ResponsiveText.test.tsx
import React from 'react';
import { render } from '@testing-library/react-native';
import { ResponsiveText } from '../../../shared/components/ResponsiveText';

// Mock useBreakpoints
jest.mock('../../../shared/hooks/useBreakpoints', () => ({
  useBreakpoints: () => ({
    isXs: true,
    isSm: false,
    isMd: false,
    isLg: false,
    isXl: false,
  }),
}));

describe('ResponsiveText', () => {
  it('renders with default props', () => {
    const { getByText } = render(
      <ResponsiveText>Test text</ResponsiveText>
    );

    expect(getByText('Test text')).toBeTruthy();
  });

  it('applies responsive size', () => {
    const { getByText } = render(
      <ResponsiveText
        size="lg"
        responsive={{
          size: { xs: 'sm', sm: 'md', md: 'lg' }
        }}
      >
        Responsive text
      </ResponsiveText>
    );

    const text = getByText('Responsive text');
    expect(text).toBeTruthy();
  });
});
```

## 5. Optimización de Performance

### Lazy Loading de Componentes
```typescript
// shared/components/LazyResponsiveComponent.tsx
import React, { lazy, Suspense } from 'react';
import { View, ActivityIndicator } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';

const DesktopComponent = lazy(() => import('./DesktopComponent'));
const MobileComponent = lazy(() => import('./MobileComponent'));

export const LazyResponsiveComponent: React.FC = () => {
  const { isDesktop } = useBreakpoints();

  return (
    <Suspense fallback={
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" />
      </View>
    }>
      {isDesktop ? <DesktopComponent /> : <MobileComponent />}
    </Suspense>
  );
};
```

### Memoización de Componentes Responsive
```typescript
// shared/components/MemoizedResponsiveComponent.tsx
import React, { memo, useMemo } from 'react';
import { View, StyleSheet } from 'react-native';
import { useBreakpoints } from '../hooks/useBreakpoints';

interface MemoizedResponsiveComponentProps {
  data: any[];
  onItemPress: (item: any) => void;
}

export const MemoizedResponsiveComponent = memo<MemoizedResponsiveComponentProps>(
  ({ data, onItemPress }) => {
    const { isMobile, isTablet, isDesktop } = useBreakpoints();

    const getItemStyle = useMemo(() => {
      if (isMobile) {
        return { width: '100%', marginBottom: 8 };
      } else if (isTablet) {
        return { width: '50%', marginBottom: 12 };
      } else {
        return { width: '33.33%', marginBottom: 16 };
      }
    }, [isMobile, isTablet, isDesktop]);

    const getContainerStyle = useMemo(() => {
      return {
        flexDirection: isMobile ? 'column' : 'row',
        flexWrap: isMobile ? 'nowrap' : 'wrap',
      };
    }, [isMobile]);

    return (
      <View style={[styles.container, getContainerStyle]}>
        {data.map((item, index) => (
          <View key={index} style={[styles.item, getItemStyle]}>
            {/* Render item content */}
          </View>
        ))}
      </View>
    );
  }
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  item: {
    padding: 16,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
  },
});
```

## 6. Mejores Prácticas

### Diseño Responsive
- **Mobile First**: Diseñar primero para móvil
- **Breakpoints**: Usar breakpoints consistentes
- **Testing**: Probar en múltiples dispositivos
- **Performance**: Optimizar para cada tamaño de pantalla

### Consideraciones de UX
- **Touch Targets**: Tamaños apropiados para touch
- **Legibilidad**: Texto legible en todos los tamaños
- **Navegación**: Patrones de navegación apropiados
- **Accesibilidad**: Asegurar accesibilidad en todos los dispositivos

## Conclusión

El diseño responsive multiplataforma requiere:

- **Sistema de breakpoints** bien definido
- **Componentes flexibles** que se adapten a diferentes tamaños
- **Testing comprehensivo** en múltiples dispositivos
- **Optimización de performance** para cada plataforma

En la siguiente clase exploraremos testing y deployment multiplataforma.

## Tarea

1. Implementar sistema de breakpoints personalizado
2. Crear componentes responsive reutilizables
3. Desarrollar layouts adaptativos
4. Configurar testing para diseño responsive
5. Optimizar performance de componentes responsive

## Enlaces Útiles

- [Responsive Design](https://web.dev/responsive-web-design-basics/)
- [CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Media Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries)
