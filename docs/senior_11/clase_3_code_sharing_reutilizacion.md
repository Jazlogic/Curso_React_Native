# Clase 3: Code Sharing y Reutilización

## Objetivos de la Clase
- Implementar estrategias efectivas de code sharing
- Crear hooks y utilidades multiplataforma
- Gestionar código específico vs código común
- Optimizar la reutilización de código

## 1. Estrategias de Code Sharing

### Arquitectura de Código Compartido
```
src/
├── shared/           # Código compartido entre todas las plataformas
│   ├── components/   # Componentes reutilizables
│   ├── hooks/        # Hooks personalizados
│   ├── utils/        # Utilidades comunes
│   ├── services/     # Servicios de API
│   ├── types/        # Tipos TypeScript
│   └── constants/    # Constantes
├── platforms/        # Código específico por plataforma
│   ├── web/
│   ├── ios/
│   ├── android/
│   ├── windows/
│   └── macos/
└── app/             # Lógica de la aplicación
    ├── screens/
    ├── navigation/
    └── store/
```

### Configuración de Resolución de Módulos
```javascript
// metro.config.js
const { getDefaultConfig } = require('metro-config');

module.exports = (async () => {
  const {
    resolver: { sourceExts, assetExts },
  } = await getDefaultConfig();
  
  return {
    resolver: {
      assetExts: assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...sourceExts, 'svg'],
      platforms: ['ios', 'android', 'native', 'web', 'windows', 'macos'],
    },
  };
})();
```

### Webpack Config para Code Sharing
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      'react-native$': 'react-native-web',
      '@shared': path.resolve(__dirname, 'src/shared'),
      '@platforms': path.resolve(__dirname, 'src/platforms'),
      '@app': path.resolve(__dirname, 'src/app'),
    },
    extensions: [
      '.web.js',
      '.web.ts',
      '.web.tsx',
      '.js',
      '.ts',
      '.tsx',
      '.json',
    ],
  },
};
```

## 2. Hooks Multiplataforma

### Hook de Plataforma
```typescript
// shared/hooks/usePlatform.ts
import { Platform } from 'react-native';
import { useMemo } from 'react';

export const usePlatform = () => {
  return useMemo(() => ({
    isWeb: Platform.OS === 'web',
    isIOS: Platform.OS === 'ios',
    isAndroid: Platform.OS === 'android',
    isWindows: Platform.OS === 'windows',
    isMacOS: Platform.OS === 'macos',
    isMobile: Platform.OS === 'ios' || Platform.OS === 'android',
    isDesktop: Platform.OS === 'windows' || Platform.OS === 'macos',
    platform: Platform.OS,
  }), []);
};
```

### Hook de Dimensiones Responsive
```typescript
// shared/hooks/useResponsive.ts
import { useState, useEffect } from 'react';
import { Dimensions } from 'react-native';

export const useResponsive = () => {
  const [dimensions, setDimensions] = useState(Dimensions.get('window'));

  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => {
      setDimensions(window);
    });

    return () => subscription?.remove();
  }, []);

  const { width, height } = dimensions;

  return {
    width,
    height,
    isMobile: width < 768,
    isTablet: width >= 768 && width < 1024,
    isDesktop: width >= 1024,
    isLandscape: width > height,
    isPortrait: height > width,
  };
};
```

### Hook de API Multiplataforma
```typescript
// shared/hooks/useAPI.ts
import { useState, useCallback } from 'react';
import { Platform } from 'react-native';

interface APIResponse<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

export const useAPI = <T>() => {
  const [state, setState] = useState<APIResponse<T>>({
    data: null,
    loading: false,
    error: null,
  });

  const fetchData = useCallback(async (url: string, options?: RequestInit) => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const response = await fetch(url, {
        ...options,
        headers: {
          'Content-Type': 'application/json',
          ...options?.headers,
        },
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({
        data: null,
        loading: false,
        error: error instanceof Error ? error.message : 'Unknown error',
      });
    }
  }, []);

  return {
    ...state,
    fetchData,
  };
};
```

### Hook de Storage Multiplataforma
```typescript
// shared/hooks/useStorage.ts
import { useState, useEffect, useCallback } from 'react';
import { Platform } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const useStorage = <T>(key: string, defaultValue: T) => {
  const [value, setValue] = useState<T>(defaultValue);
  const [loading, setLoading] = useState(true);

  const getStorage = useCallback(async (): Promise<T> => {
    if (Platform.OS === 'web') {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } else {
      const item = await AsyncStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    }
  }, [key, defaultValue]);

  const setStorage = useCallback(async (newValue: T) => {
    if (Platform.OS === 'web') {
      localStorage.setItem(key, JSON.stringify(newValue));
    } else {
      await AsyncStorage.setItem(key, JSON.stringify(newValue));
    }
    setValue(newValue);
  }, [key]);

  const removeStorage = useCallback(async () => {
    if (Platform.OS === 'web') {
      localStorage.removeItem(key);
    } else {
      await AsyncStorage.removeItem(key);
    }
    setValue(defaultValue);
  }, [key, defaultValue]);

  useEffect(() => {
    const loadValue = async () => {
      try {
        const storedValue = await getStorage();
        setValue(storedValue);
      } catch (error) {
        console.error('Error loading from storage:', error);
      } finally {
        setLoading(false);
      }
    };

    loadValue();
  }, [getStorage]);

  return {
    value,
    setValue: setStorage,
    removeValue: removeStorage,
    loading,
  };
};
```

## 3. Utilidades Multiplataforma

### Utilidades de Navegación
```typescript
// shared/utils/navigation.ts
import { Platform, Linking } from 'react-native';

export const NavigationUtils = {
  // Abrir URL en la plataforma apropiada
  openURL: async (url: string) => {
    try {
      if (Platform.OS === 'web') {
        window.open(url, '_blank');
      } else {
        const supported = await Linking.canOpenURL(url);
        if (supported) {
          await Linking.openURL(url);
        } else {
          console.error('Cannot open URL:', url);
        }
      }
    } catch (error) {
      console.error('Error opening URL:', error);
    }
  },

  // Compartir contenido
  share: async (content: string) => {
    if (Platform.OS === 'web') {
      if (navigator.share) {
        await navigator.share({ text: content });
      } else {
        // Fallback para navegadores que no soportan Web Share API
        navigator.clipboard.writeText(content);
        alert('Content copied to clipboard');
      }
    } else {
      const { share } = require('react-native');
      await share({ message: content });
    }
  },

  // Obtener información del dispositivo
  getDeviceInfo: () => {
    if (Platform.OS === 'web') {
      return {
        platform: 'web',
        userAgent: navigator.userAgent,
        language: navigator.language,
        online: navigator.onLine,
      };
    } else {
      return {
        platform: Platform.OS,
        version: Platform.Version,
      };
    }
  },
};
```

### Utilidades de Validación
```typescript
// shared/utils/validation.ts
export const ValidationUtils = {
  // Validar email
  isValidEmail: (email: string): boolean => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  },

  // Validar teléfono
  isValidPhone: (phone: string): boolean => {
    const phoneRegex = /^\+?[\d\s\-\(\)]+$/;
    return phoneRegex.test(phone) && phone.replace(/\D/g, '').length >= 10;
  },

  // Validar contraseña
  isValidPassword: (password: string): boolean => {
    // Al menos 8 caracteres, una mayúscula, una minúscula, un número
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d@$!%*?&]{8,}$/;
    return passwordRegex.test(password);
  },

  // Validar URL
  isValidURL: (url: string): boolean => {
    try {
      new URL(url);
      return true;
    } catch {
      return false;
    }
  },

  // Sanitizar input
  sanitizeInput: (input: string): string => {
    return input.trim().replace(/[<>]/g, '');
  },
};
```

### Utilidades de Formateo
```typescript
// shared/utils/formatting.ts
export const FormattingUtils = {
  // Formatear fecha
  formatDate: (date: Date, locale: string = 'en-US'): string => {
    return new Intl.DateTimeFormat(locale, {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
    }).format(date);
  },

  // Formatear moneda
  formatCurrency: (amount: number, currency: string = 'USD', locale: string = 'en-US'): string => {
    return new Intl.NumberFormat(locale, {
      style: 'currency',
      currency,
    }).format(amount);
  },

  // Formatear número
  formatNumber: (number: number, locale: string = 'en-US'): string => {
    return new Intl.NumberFormat(locale).format(number);
  },

  // Formatear porcentaje
  formatPercentage: (value: number, locale: string = 'en-US'): string => {
    return new Intl.NumberFormat(locale, {
      style: 'percent',
      minimumFractionDigits: 2,
    }).format(value / 100);
  },

  // Formatear texto (capitalizar, etc.)
  capitalize: (text: string): string => {
    return text.charAt(0).toUpperCase() + text.slice(1).toLowerCase();
  },

  // Truncar texto
  truncate: (text: string, maxLength: number): string => {
    if (text.length <= maxLength) return text;
    return text.slice(0, maxLength) + '...';
  },
};
```

## 4. Servicios Compartidos

### Servicio de API
```typescript
// shared/services/APIService.ts
import { Platform } from 'react-native';

interface APIResponse<T> {
  data: T;
  status: number;
  message?: string;
}

class APIService {
  private baseURL: string;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<APIResponse<T>> {
    const url = `${this.baseURL}${endpoint}`;
    
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
      ...options,
    };

    // Agregar headers específicos de plataforma
    if (Platform.OS === 'web') {
      config.headers = {
        ...config.headers,
        'X-Platform': 'web',
      };
    } else {
      config.headers = {
        ...config.headers,
        'X-Platform': Platform.OS,
      };
    }

    try {
      const response = await fetch(url, config);
      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.message || 'API request failed');
      }

      return {
        data,
        status: response.status,
      };
    } catch (error) {
      throw new Error(error instanceof Error ? error.message : 'Unknown error');
    }
  }

  async get<T>(endpoint: string): Promise<APIResponse<T>> {
    return this.request<T>(endpoint, { method: 'GET' });
  }

  async post<T>(endpoint: string, data: any): Promise<APIResponse<T>> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  async put<T>(endpoint: string, data: any): Promise<APIResponse<T>> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  async delete<T>(endpoint: string): Promise<APIResponse<T>> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

export const apiService = new APIService(process.env.API_BASE_URL || 'https://api.example.com');
```

### Servicio de Autenticación
```typescript
// shared/services/AuthService.ts
import { Platform } from 'react-native';
import { apiService } from './APIService';

interface User {
  id: string;
  email: string;
  name: string;
}

interface AuthResponse {
  user: User;
  token: string;
}

class AuthService {
  private token: string | null = null;

  async login(email: string, password: string): Promise<AuthResponse> {
    const response = await apiService.post<AuthResponse>('/auth/login', {
      email,
      password,
    });

    this.token = response.data.token;
    this.storeToken(response.data.token);

    return response.data;
  }

  async register(email: string, password: string, name: string): Promise<AuthResponse> {
    const response = await apiService.post<AuthResponse>('/auth/register', {
      email,
      password,
      name,
    });

    this.token = response.data.token;
    this.storeToken(response.data.token);

    return response.data;
  }

  async logout(): Promise<void> {
    this.token = null;
    this.removeToken();
  }

  isAuthenticated(): boolean {
    return !!this.token;
  }

  getToken(): string | null {
    return this.token;
  }

  private storeToken(token: string): void {
    if (Platform.OS === 'web') {
      localStorage.setItem('auth_token', token);
    } else {
      // Usar AsyncStorage para móvil
      const AsyncStorage = require('@react-native-async-storage/async-storage').default;
      AsyncStorage.setItem('auth_token', token);
    }
  }

  private removeToken(): void {
    if (Platform.OS === 'web') {
      localStorage.removeItem('auth_token');
    } else {
      const AsyncStorage = require('@react-native-async-storage/async-storage').default;
      AsyncStorage.removeItem('auth_token');
    }
  }
}

export const authService = new AuthService();
```

## 5. Componentes Compartidos

### Componente Base
```typescript
// shared/components/BaseComponent.tsx
import React from 'react';
import { View, StyleSheet, ViewStyle } from 'react-native';
import { usePlatform } from '../hooks/usePlatform';

interface BaseComponentProps {
  children: React.ReactNode;
  style?: ViewStyle;
  webStyle?: ViewStyle;
  mobileStyle?: ViewStyle;
  desktopStyle?: ViewStyle;
}

export const BaseComponent: React.FC<BaseComponentProps> = ({
  children,
  style,
  webStyle,
  mobileStyle,
  desktopStyle,
}) => {
  const { isWeb, isMobile, isDesktop } = usePlatform();

  const getPlatformStyle = (): ViewStyle => {
    if (isWeb && webStyle) return webStyle;
    if (isMobile && mobileStyle) return mobileStyle;
    if (isDesktop && desktopStyle) return desktopStyle;
    return {};
  };

  return (
    <View style={[styles.base, style, getPlatformStyle()]}>
      {children}
    </View>
  );
};

const styles = StyleSheet.create({
  base: {
    flex: 1,
  },
});
```

### Componente de Botón Multiplataforma
```typescript
// shared/components/Button.tsx
import React from 'react';
import { TouchableOpacity, Text, StyleSheet, ViewStyle, TextStyle } from 'react-native';
import { usePlatform } from '../hooks/usePlatform';

interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
}

export const Button: React.FC<ButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  style,
  textStyle,
}) => {
  const { isWeb } = usePlatform();

  const getButtonStyle = (): ViewStyle => {
    const baseStyle = [styles.button, styles[`button_${size}`]];
    
    if (variant === 'primary') {
      baseStyle.push(styles.button_primary);
    } else if (variant === 'secondary') {
      baseStyle.push(styles.button_secondary);
    } else if (variant === 'outline') {
      baseStyle.push(styles.button_outline);
    }

    if (disabled) {
      baseStyle.push(styles.button_disabled);
    }

    if (isWeb) {
      baseStyle.push(styles.button_web);
    }

    return baseStyle;
  };

  const getTextStyle = (): TextStyle => {
    const baseStyle = [styles.text, styles[`text_${size}`]];
    
    if (variant === 'primary') {
      baseStyle.push(styles.text_primary);
    } else if (variant === 'secondary') {
      baseStyle.push(styles.text_secondary);
    } else if (variant === 'outline') {
      baseStyle.push(styles.text_outline);
    }

    if (disabled) {
      baseStyle.push(styles.text_disabled);
    }

    return baseStyle;
  };

  return (
    <TouchableOpacity
      style={[getButtonStyle(), style]}
      onPress={onPress}
      disabled={disabled}
      activeOpacity={disabled ? 1 : 0.7}
    >
      <Text style={[getTextStyle(), textStyle]}>{title}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  button_web: {
    cursor: 'pointer',
    userSelect: 'none',
  },
  button_small: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    minHeight: 32,
  },
  button_medium: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    minHeight: 40,
  },
  button_large: {
    paddingHorizontal: 20,
    paddingVertical: 12,
    minHeight: 48,
  },
  button_primary: {
    backgroundColor: '#007AFF',
  },
  button_secondary: {
    backgroundColor: '#6C757D',
  },
  button_outline: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#007AFF',
  },
  button_disabled: {
    opacity: 0.5,
  },
  text: {
    fontWeight: '600',
    textAlign: 'center',
  },
  text_small: {
    fontSize: 14,
  },
  text_medium: {
    fontSize: 16,
  },
  text_large: {
    fontSize: 18,
  },
  text_primary: {
    color: '#FFFFFF',
  },
  text_secondary: {
    color: '#FFFFFF',
  },
  text_outline: {
    color: '#007AFF',
  },
  text_disabled: {
    opacity: 0.7,
  },
});
```

## 6. Testing de Código Compartido

### Test de Hooks
```typescript
// __tests__/shared/hooks/usePlatform.test.ts
import { renderHook } from '@testing-library/react-hooks';
import { usePlatform } from '../../../shared/hooks/usePlatform';

describe('usePlatform', () => {
  it('returns correct platform info for web', () => {
    jest.mock('react-native/Libraries/Utilities/Platform', () => ({
      OS: 'web',
    }));

    const { result } = renderHook(() => usePlatform());

    expect(result.current.isWeb).toBe(true);
    expect(result.current.isMobile).toBe(false);
    expect(result.current.isDesktop).toBe(false);
    expect(result.current.platform).toBe('web');
  });

  it('returns correct platform info for mobile', () => {
    jest.mock('react-native/Libraries/Utilities/Platform', () => ({
      OS: 'ios',
    }));

    const { result } = renderHook(() => usePlatform());

    expect(result.current.isWeb).toBe(false);
    expect(result.current.isMobile).toBe(true);
    expect(result.current.isDesktop).toBe(false);
    expect(result.current.platform).toBe('ios');
  });
});
```

### Test de Utilidades
```typescript
// __tests__/shared/utils/validation.test.ts
import { ValidationUtils } from '../../../shared/utils/validation';

describe('ValidationUtils', () => {
  describe('isValidEmail', () => {
    it('validates correct email addresses', () => {
      expect(ValidationUtils.isValidEmail('test@example.com')).toBe(true);
      expect(ValidationUtils.isValidEmail('user.name@domain.co.uk')).toBe(true);
    });

    it('rejects invalid email addresses', () => {
      expect(ValidationUtils.isValidEmail('invalid-email')).toBe(false);
      expect(ValidationUtils.isValidEmail('test@')).toBe(false);
      expect(ValidationUtils.isValidEmail('@example.com')).toBe(false);
    });
  });

  describe('isValidPassword', () => {
    it('validates strong passwords', () => {
      expect(ValidationUtils.isValidPassword('Password123')).toBe(true);
      expect(ValidationUtils.isValidPassword('MyStr0ng!Pass')).toBe(true);
    });

    it('rejects weak passwords', () => {
      expect(ValidationUtils.isValidPassword('password')).toBe(false);
      expect(ValidationUtils.isValidPassword('12345678')).toBe(false);
      expect(ValidationUtils.isValidPassword('Password')).toBe(false);
    });
  });
});
```

## 7. Mejores Prácticas

### Organización de Código
- **Separar por responsabilidad**: Lógica de negocio vs UI
- **Usar TypeScript**: Para mejor type safety
- **Documentar APIs**: Especialmente para código compartido
- **Testing**: Probar código compartido en todas las plataformas

### Optimización
- **Lazy loading**: Cargar código específico solo cuando sea necesario
- **Tree shaking**: Eliminar código no utilizado
- **Bundle analysis**: Analizar el tamaño del bundle
- **Performance**: Optimizar para cada plataforma

## Conclusión

El code sharing efectivo requiere:

- **Arquitectura clara** con separación de responsabilidades
- **Hooks y utilidades** bien diseñados
- **Testing comprehensivo** en todas las plataformas
- **Optimización** del bundle y performance

En la siguiente clase exploraremos el diseño responsive multiplataforma.

## Tarea

1. Implementar hooks multiplataforma personalizados
2. Crear utilidades compartidas para validación y formateo
3. Desarrollar servicios de API compartidos
4. Crear componentes base reutilizables
5. Configurar testing para código compartido

## Enlaces Útiles

- [React Hooks](https://reactjs.org/docs/hooks-intro.html)
- [TypeScript](https://www.typescriptlang.org/)
- [Jest Testing](https://jestjs.io/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
