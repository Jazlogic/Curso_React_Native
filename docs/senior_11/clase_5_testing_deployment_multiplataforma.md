# Clase 5: Testing y Deployment Multiplataforma

## Objetivos de la Clase
- Implementar testing en múltiples plataformas
- Configurar CI/CD para builds multiplataforma
- Desplegar aplicaciones en diferentes stores y plataformas
- Monitorear y mantener aplicaciones multiplataforma

## 1. Testing Multiplataforma

### Setup de Testing
```typescript
// __tests__/setup.ts
import 'react-native-gesture-handler/jestSetup';

// Mock para diferentes plataformas
jest.mock('react-native/Libraries/Utilities/Platform', () => ({
  OS: 'web', // Cambiar según la plataforma a testear
  select: jest.fn((obj) => obj.web || obj.default),
}));

// Mock para APIs nativas
jest.mock('react-native', () => {
  const RN = jest.requireActual('react-native');
  
  return {
    ...RN,
    NativeModules: {
      ...RN.NativeModules,
      // Mocks específicos por plataforma
    },
  };
});

// Mock para web APIs
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});

// Mock para localStorage
Object.defineProperty(window, 'localStorage', {
  value: {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn(),
  },
});
```

### Test de Componentes Multiplataforma
```typescript
// __tests__/components/Button.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { Button } from '../../shared/components/Button';

describe('Button Component', () => {
  it('renders correctly on web', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Platform').OS)
      .mockReturnValue('web');

    const { getByText } = render(
      <Button title="Test Button" onPress={() => {}} />
    );

    expect(getByText('Test Button')).toBeTruthy();
  });

  it('renders correctly on mobile', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Platform').OS)
      .mockReturnValue('ios');

    const { getByText } = render(
      <Button title="Test Button" onPress={() => {}} />
    );

    expect(getByText('Test Button')).toBeTruthy();
  });

  it('handles press events', () => {
    const onPress = jest.fn();
    const { getByText } = render(
      <Button title="Test Button" onPress={onPress} />
    );

    fireEvent.press(getByText('Test Button'));
    expect(onPress).toHaveBeenCalledTimes(1);
  });

  it('applies correct styles for different variants', () => {
    const { getByText, rerender } = render(
      <Button title="Primary" variant="primary" onPress={() => {}} />
    );

    expect(getByText('Primary')).toBeTruthy();

    rerender(
      <Button title="Secondary" variant="secondary" onPress={() => {}} />
    );

    expect(getByText('Secondary')).toBeTruthy();
  });
});
```

### Test de Hooks Multiplataforma
```typescript
// __tests__/hooks/usePlatform.test.ts
import { renderHook } from '@testing-library/react-hooks';
import { usePlatform } from '../../shared/hooks/usePlatform';

describe('usePlatform Hook', () => {
  it('returns correct platform info for web', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Platform').OS)
      .mockReturnValue('web');

    const { result } = renderHook(() => usePlatform());

    expect(result.current.isWeb).toBe(true);
    expect(result.current.isMobile).toBe(false);
    expect(result.current.isDesktop).toBe(false);
    expect(result.current.platform).toBe('web');
  });

  it('returns correct platform info for iOS', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Platform').OS)
      .mockReturnValue('ios');

    const { result } = renderHook(() => usePlatform());

    expect(result.current.isWeb).toBe(false);
    expect(result.current.isMobile).toBe(true);
    expect(result.current.isDesktop).toBe(false);
    expect(result.current.platform).toBe('ios');
  });

  it('returns correct platform info for Windows', () => {
    jest.mocked(require('react-native/Libraries/Utilities/Platform').OS)
      .mockReturnValue('windows');

    const { result } = renderHook(() => usePlatform());

    expect(result.current.isWeb).toBe(false);
    expect(result.current.isMobile).toBe(false);
    expect(result.current.isDesktop).toBe(true);
    expect(result.current.platform).toBe('windows');
  });
});
```

### Test de Integración
```typescript
// __tests__/integration/App.test.tsx
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import App from '../../App';

describe('App Integration', () => {
  it('renders correctly on all platforms', async () => {
    const platforms = ['web', 'ios', 'android', 'windows', 'macos'];
    
    for (const platform of platforms) {
      jest.mocked(require('react-native/Libraries/Utilities/Platform').OS)
        .mockReturnValue(platform);

      const { getByText } = render(<App />);
      
      await waitFor(() => {
        expect(getByText('Multiplatform Counter App')).toBeTruthy();
      });
    }
  });

  it('handles counter functionality across platforms', async () => {
    const { getByText } = render(<App />);
    
    const incrementButton = getByText('+');
    const decrementButton = getByText('-');
    const counterText = getByText('Count: 0');

    expect(counterText).toBeTruthy();

    fireEvent.press(incrementButton);
    await waitFor(() => {
      expect(getByText('Count: 1')).toBeTruthy();
    });

    fireEvent.press(decrementButton);
    await waitFor(() => {
      expect(getByText('Count: 0')).toBeTruthy();
    });
  });
});
```

## 2. CI/CD Multiplataforma

### GitHub Actions Workflow
```yaml
# .github/workflows/multiplatform.yml
name: Multiplatform Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        platform: [web, ios, android, windows, macos]
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests for ${{ matrix.platform }}
        run: npm run test:${{ matrix.platform }}
        env:
          PLATFORM: ${{ matrix.platform }}

  build-web:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build web
        run: npm run build:web
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'

  build-mobile:
    runs-on: macos-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '11'
      
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
      
      - name: Build Android
        run: npm run build:android
      
      - name: Build iOS
        run: npm run build:ios
      
      - name: Upload to App Center
        uses: microsoft/appcenter-cli-action@v1
        with:
          app-secret: ${{ secrets.APP_CENTER_SECRET }}
          app: ${{ secrets.APP_CENTER_APP }}
          release-notes: 'Automated build from GitHub Actions'
          file: 'android/app/build/outputs/apk/release/app-release.apk'

  build-desktop:
    runs-on: windows-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build Windows
        run: npm run build:windows
      
      - name: Upload Windows build
        uses: actions/upload-artifact@v3
        with:
          name: windows-build
          path: windows/MyApp/bin/x64/Release/MyApp.exe

  build-macos:
    runs-on: macos-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build macOS
        run: npm run build:macos
      
      - name: Upload macOS build
        uses: actions/upload-artifact@v3
        with:
          name: macos-build
          path: macos/MyApp.app
```

### Scripts de Build
```json
{
  "scripts": {
    "test:web": "jest --testPathPattern=web --setupFilesAfterEnv=__tests__/setup-web.ts",
    "test:ios": "jest --testPathPattern=ios --setupFilesAfterEnv=__tests__/setup-ios.ts",
    "test:android": "jest --testPathPattern=android --setupFilesAfterEnv=__tests__/setup-android.ts",
    "test:windows": "jest --testPathPattern=windows --setupFilesAfterEnv=__tests__/setup-windows.ts",
    "test:macos": "jest --testPathPattern=macos --setupFilesAfterEnv=__tests__/setup-macos.ts",
    "test:all": "npm run test:web && npm run test:ios && npm run test:android && npm run test:windows && npm run test:macos",
    "build:web": "webpack --mode production",
    "build:ios": "react-native run-ios --configuration Release",
    "build:android": "cd android && ./gradlew assembleRelease",
    "build:windows": "react-native run-windows --release",
    "build:macos": "react-native run-macos --configuration Release",
    "build:all": "npm run build:web && npm run build:ios && npm run build:android && npm run build:windows && npm run build:macos"
  }
}
```

## 3. Deployment Multiplataforma

### Deployment Web
```typescript
// scripts/deploy-web.ts
import { execSync } from 'child_process';
import { writeFileSync } from 'fs';

const deployWeb = async () => {
  console.log('Building web application...');
  
  // Build the application
  execSync('npm run build:web', { stdio: 'inherit' });
  
  // Create deployment configuration
  const vercelConfig = {
    version: 2,
    builds: [
      {
        src: 'dist/**/*',
        use: '@vercel/static',
      },
    ],
    routes: [
      {
        src: '/(.*)',
        dest: '/index.html',
      },
    ],
  };
  
  writeFileSync('vercel.json', JSON.stringify(vercelConfig, null, 2));
  
  console.log('Web deployment configuration created');
};

deployWeb().catch(console.error);
```

### Deployment Móvil
```typescript
// scripts/deploy-mobile.ts
import { execSync } from 'child_process';

const deployMobile = async () => {
  console.log('Building mobile applications...');
  
  // Build iOS
  console.log('Building iOS...');
  execSync('npm run build:ios', { stdio: 'inherit' });
  
  // Build Android
  console.log('Building Android...');
  execSync('npm run build:android', { stdio: 'inherit' });
  
  // Upload to App Center
  console.log('Uploading to App Center...');
  execSync('appcenter distribute release --app MyApp/MyApp-iOS --file ios/build/MyApp.ipa', { stdio: 'inherit' });
  execSync('appcenter distribute release --app MyApp/MyApp-Android --file android/app/build/outputs/apk/release/app-release.apk', { stdio: 'inherit' });
  
  console.log('Mobile deployment completed');
};

deployMobile().catch(console.error);
```

### Deployment Desktop
```typescript
// scripts/deploy-desktop.ts
import { execSync } from 'child_process';

const deployDesktop = async () => {
  console.log('Building desktop applications...');
  
  // Build Windows
  console.log('Building Windows...');
  execSync('npm run build:windows', { stdio: 'inherit' });
  
  // Build macOS
  console.log('Building macOS...');
  execSync('npm run build:macos', { stdio: 'inherit' });
  
  // Create installers
  console.log('Creating installers...');
  execSync('npm run create-installers', { stdio: 'inherit' });
  
  console.log('Desktop deployment completed');
};

deployDesktop().catch(console.error);
```

## 4. Monitoreo y Analytics

### Configuración de Analytics
```typescript
// shared/services/AnalyticsService.ts
import { Platform } from 'react-native';

interface AnalyticsEvent {
  name: string;
  properties?: Record<string, any>;
  timestamp?: number;
}

class AnalyticsService {
  private platform: string;

  constructor() {
    this.platform = Platform.OS;
  }

  // Track event
  track(event: AnalyticsEvent) {
    const eventData = {
      ...event,
      platform: this.platform,
      timestamp: event.timestamp || Date.now(),
    };

    if (Platform.OS === 'web') {
      // Google Analytics 4
      if (typeof gtag !== 'undefined') {
        gtag('event', event.name, eventData.properties);
      }
    } else {
      // Firebase Analytics
      const { analytics } = require('@react-native-firebase/analytics');
      analytics().logEvent(event.name, eventData.properties);
    }
  }

  // Track screen view
  trackScreenView(screenName: string, screenClass?: string) {
    this.track({
      name: 'screen_view',
      properties: {
        screen_name: screenName,
        screen_class: screenClass || screenName,
      },
    });
  }

  // Track user action
  trackUserAction(action: string, properties?: Record<string, any>) {
    this.track({
      name: 'user_action',
      properties: {
        action,
        ...properties,
      },
    });
  }

  // Track error
  trackError(error: Error, context?: Record<string, any>) {
    this.track({
      name: 'error',
      properties: {
        error_message: error.message,
        error_stack: error.stack,
        ...context,
      },
    });
  }
}

export const analyticsService = new AnalyticsService();
```

### Configuración de Crashlytics
```typescript
// shared/services/CrashlyticsService.ts
import { Platform } from 'react-native';

class CrashlyticsService {
  // Log error
  logError(error: Error, context?: Record<string, any>) {
    if (Platform.OS === 'web') {
      // Sentry for web
      const Sentry = require('@sentry/react');
      Sentry.captureException(error, { extra: context });
    } else {
      // Firebase Crashlytics for mobile
      const crashlytics = require('@react-native-firebase/crashlytics').default();
      crashlytics().recordError(error);
      
      if (context) {
        Object.entries(context).forEach(([key, value]) => {
          crashlytics().setAttribute(key, String(value));
        });
      }
    }
  }

  // Set user ID
  setUserId(userId: string) {
    if (Platform.OS === 'web') {
      const Sentry = require('@sentry/react');
      Sentry.setUser({ id: userId });
    } else {
      const crashlytics = require('@react-native-firebase/crashlytics').default();
      crashlytics().setUserId(userId);
    }
  }

  // Set custom key
  setCustomKey(key: string, value: string) {
    if (Platform.OS === 'web') {
      const Sentry = require('@sentry/react');
      Sentry.setTag(key, value);
    } else {
      const crashlytics = require('@react-native-firebase/crashlytics').default();
      crashlytics().setAttribute(key, value);
    }
  }
}

export const crashlyticsService = new CrashlyticsService();
```

## 5. Mantenimiento Multiplataforma

### Script de Actualización
```typescript
// scripts/update-dependencies.ts
import { execSync } from 'child_process';
import { readFileSync, writeFileSync } from 'fs';

const updateDependencies = async () => {
  console.log('Updating dependencies...');
  
  // Update npm dependencies
  execSync('npm update', { stdio: 'inherit' });
  
  // Update React Native
  execSync('npx react-native upgrade', { stdio: 'inherit' });
  
  // Update platform-specific dependencies
  if (process.platform === 'win32') {
    execSync('npm run update:windows', { stdio: 'inherit' });
  } else if (process.platform === 'darwin') {
    execSync('npm run update:macos', { stdio: 'inherit' });
  }
  
  // Update web dependencies
  execSync('npm run update:web', { stdio: 'inherit' });
  
  console.log('Dependencies updated successfully');
};

updateDependencies().catch(console.error);
```

### Script de Limpieza
```typescript
// scripts/clean.ts
import { execSync } from 'child_process';
import { rmSync, existsSync } from 'fs';

const clean = async () => {
  console.log('Cleaning build artifacts...');
  
  // Clean React Native
  execSync('npx react-native clean', { stdio: 'inherit' });
  
  // Clean web build
  if (existsSync('dist')) {
    rmSync('dist', { recursive: true, force: true });
  }
  
  // Clean platform-specific builds
  if (existsSync('ios/build')) {
    rmSync('ios/build', { recursive: true, force: true });
  }
  
  if (existsSync('android/app/build')) {
    rmSync('android/app/build', { recursive: true, force: true });
  }
  
  if (existsSync('windows/MyApp/bin')) {
    rmSync('windows/MyApp/bin', { recursive: true, force: true });
  }
  
  if (existsSync('macos/MyApp/build')) {
    rmSync('macos/MyApp/build', { recursive: true, force: true });
  }
  
  console.log('Clean completed');
};

clean().catch(console.error);
```

## 6. Mejores Prácticas

### Testing
- **Cobertura completa**: Probar en todas las plataformas
- **Tests de integración**: Verificar funcionamiento conjunto
- **Tests de regresión**: Detectar problemas en actualizaciones
- **Tests de performance**: Verificar rendimiento en cada plataforma

### Deployment
- **Automatización**: Usar CI/CD para todos los deployments
- **Versionado**: Mantener versiones consistentes
- **Rollback**: Planificar estrategias de rollback
- **Monitoreo**: Implementar monitoreo en todas las plataformas

### Mantenimiento
- **Actualizaciones regulares**: Mantener dependencias actualizadas
- **Documentación**: Documentar cambios y decisiones
- **Comunicación**: Comunicar cambios a todos los equipos
- **Testing**: Probar cambios en todas las plataformas

## Conclusión

El testing y deployment multiplataforma requiere:

- **Testing comprehensivo** en todas las plataformas
- **CI/CD automatizado** para builds y deployments
- **Monitoreo y analytics** en todas las plataformas
- **Mantenimiento regular** y actualizaciones

## Tarea

1. Configurar testing para todas las plataformas
2. Implementar CI/CD con GitHub Actions
3. Configurar deployment automático
4. Implementar monitoreo y analytics
5. Crear scripts de mantenimiento

## Enlaces Útiles

- [GitHub Actions](https://docs.github.com/en/actions)
- [App Center](https://appcenter.ms/)
- [Vercel](https://vercel.com/)
- [Firebase](https://firebase.google.com/)
- [Sentry](https://sentry.io/)
