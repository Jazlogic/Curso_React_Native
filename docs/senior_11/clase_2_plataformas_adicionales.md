# Clase 2: Plataformas Adicionales

## Objetivos de la Clase
- Configurar React Native Windows
- Configurar React Native macOS
- Entender las diferencias entre plataformas desktop
- Implementar funcionalidades específicas de cada plataforma

## 1. React Native Windows

### Instalación y Configuración
```bash
# Instalar React Native Windows
npx react-native-windows-init --overwrite

# Instalar dependencias adicionales
npm install react-native-windows
npm install --save-dev @types/react-native-windows
```

### Configuración del Proyecto
```xml
<!-- windows/MyApp/MyApp.vcxproj -->
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <ReactNativeWindowsDir>$(MSBuildThisFileDirectory)..\node_modules\react-native-windows\</ReactNativeWindowsDir>
  </PropertyGroup>
  <Import Project="$(ReactNativeWindowsDir)\PropertySheets\External\Microsoft.ReactNative.UWP.CppApp.props" Condition="Exists('$(ReactNativeWindowsDir)\PropertySheets\External\Microsoft.ReactNative.UWP.CppApp.props')" />
  <Import Project="$(ReactNativeWindowsDir)\PropertySheets\External\Microsoft.ReactNative.UWP.CppApp.targets" Condition="Exists('$(ReactNativeWindowsDir)\PropertySheets\External\Microsoft.ReactNative.UWP.CppApp.targets')" />
</Project>
```

### Componentes Específicos de Windows
```typescript
// components/WindowsSpecific.tsx
import React from 'react';
import { View, Text, StyleSheet, Platform } from 'react-native';

export const WindowsSpecific: React.FC = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Windows App</Text>
      
      {Platform.OS === 'windows' && (
        <View style={styles.windowsFeatures}>
          <Text>Windows-specific features:</Text>
          <Text>• Windows 10/11 integration</Text>
          <Text>• Live Tiles support</Text>
          <Text>• Windows Ink support</Text>
          <Text>• Cortana integration</Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f0f0f0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  windowsFeatures: {
    backgroundColor: '#e3f2fd',
    padding: 16,
    borderRadius: 8,
  },
});
```

### APIs Específicas de Windows
```typescript
// utils/WindowsAPI.ts
import { Platform, NativeModules } from 'react-native';

export const WindowsAPI = {
  // Acceso a APIs nativas de Windows
  getWindowsVersion: async (): Promise<string> => {
    if (Platform.OS === 'windows') {
      try {
        const { WindowsInfo } = NativeModules;
        return await WindowsInfo.getVersion();
      } catch (error) {
        console.error('Error getting Windows version:', error);
        return 'Unknown';
      }
    }
    return 'Not Windows';
  },

  // Integración con Windows Ink
  enableInkSupport: () => {
    if (Platform.OS === 'windows') {
      // Implementar soporte para Windows Ink
      console.log('Windows Ink support enabled');
    }
  },

  // Live Tiles
  updateLiveTile: (tileData: any) => {
    if (Platform.OS === 'windows') {
      // Actualizar Live Tile
      console.log('Live Tile updated:', tileData);
    }
  },

  // Cortana Integration
  registerCortanaCommands: (commands: string[]) => {
    if (Platform.OS === 'windows') {
      // Registrar comandos de voz para Cortana
      console.log('Cortana commands registered:', commands);
    }
  },
};
```

## 2. React Native macOS

### Instalación y Configuración
```bash
# Instalar React Native macOS
npx react-native-macos-init

# Instalar dependencias adicionales
npm install react-native-macos
npm install --save-dev @types/react-native-macos
```

### Configuración del Proyecto
```ruby
# macos/Podfile
require_relative '../node_modules/react-native/scripts/react_native_pods'
require_relative '../node_modules/@react-native-community/cli-platform-ios/native_modules'

platform :macos, '10.14'
prepare_react_native_project!

target 'MyAppMacOS' do
  config = use_native_modules!

  use_react_native!(
    :path => config[:reactNativePath],
    :macos => true,
    :hermes_enabled => flags[:hermes_enabled],
    :fabric_enabled => flags[:fabric_enabled],
    :flipper_configuration => FlipperConfiguration.enabled,
    :app_clip_configuration => false
  )

  target 'MyAppMacOSTests' do
    inherit! :complete
  end

  post_install do |installer|
    react_native_post_install(installer)
  end
end
```

### Componentes Específicos de macOS
```typescript
// components/MacOSSpecific.tsx
import React from 'react';
import { View, Text, StyleSheet, Platform } from 'react-native';

export const MacOSSpecific: React.FC = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>macOS App</Text>
      
      {Platform.OS === 'macos' && (
        <View style={styles.macosFeatures}>
          <Text>macOS-specific features:</Text>
          <Text>• Touch Bar support</Text>
          <Text>• Menu Bar integration</Text>
          <Text>• Spotlight integration</Text>
          <Text>• Handoff support</Text>
          <Text>• Metal performance</Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  macosFeatures: {
    backgroundColor: '#e8f5e8',
    padding: 16,
    borderRadius: 8,
  },
});
```

### APIs Específicas de macOS
```typescript
// utils/MacOSAPI.ts
import { Platform, NativeModules } from 'react-native';

export const MacOSAPI = {
  // Acceso a APIs nativas de macOS
  getMacOSVersion: async (): Promise<string> => {
    if (Platform.OS === 'macos') {
      try {
        const { MacOSInfo } = NativeModules;
        return await MacOSInfo.getVersion();
      } catch (error) {
        console.error('Error getting macOS version:', error);
        return 'Unknown';
      }
    }
    return 'Not macOS';
  },

  // Touch Bar support
  updateTouchBar: (touchBarItems: any[]) => {
    if (Platform.OS === 'macos') {
      // Actualizar Touch Bar
      console.log('Touch Bar updated:', touchBarItems);
    }
  },

  // Menu Bar integration
  updateMenuBar: (menuItems: any[]) => {
    if (Platform.OS === 'macos') {
      // Actualizar Menu Bar
      console.log('Menu Bar updated:', menuItems);
    }
  },

  // Spotlight integration
  indexContent: (content: any) => {
    if (Platform.OS === 'macos') {
      // Indexar contenido para Spotlight
      console.log('Content indexed for Spotlight:', content);
    }
  },

  // Handoff support
  enableHandoff: (activityType: string, userInfo: any) => {
    if (Platform.OS === 'macos') {
      // Habilitar Handoff
      console.log('Handoff enabled:', activityType, userInfo);
    }
  },
};
```

## 3. Diferencias entre Plataformas Desktop

### Comparación de Características
```typescript
// utils/PlatformComparison.ts
import { Platform } from 'react-native';

export const PlatformFeatures = {
  // Características específicas por plataforma
  getPlatformFeatures: () => {
    const features = {
      windows: [
        'Live Tiles',
        'Windows Ink',
        'Cortana Integration',
        'Windows Hello',
        'Xbox Integration',
        'Windows Store',
      ],
      macos: [
        'Touch Bar',
        'Menu Bar',
        'Spotlight Integration',
        'Handoff',
        'Metal Performance',
        'Mac App Store',
      ],
      web: [
        'PWA Support',
        'Service Workers',
        'Web APIs',
        'SEO Optimization',
        'Browser Extensions',
        'WebAssembly',
      ],
    };

    return features[Platform.OS as keyof typeof features] || [];
  },

  // APIs disponibles por plataforma
  getAvailableAPIs: () => {
    const apis = {
      windows: [
        'Windows.Storage',
        'Windows.Networking',
        'Windows.Media',
        'Windows.Graphics',
        'Windows.UI',
      ],
      macos: [
        'AppKit',
        'Foundation',
        'Core Graphics',
        'Core Animation',
        'Metal',
      ],
      web: [
        'Web APIs',
        'Service Workers',
        'WebGL',
        'WebRTC',
        'WebAssembly',
      ],
    };

    return apis[Platform.OS as keyof typeof apis] || [];
  },
};
```

### Componente de Comparación
```typescript
// components/PlatformComparison.tsx
import React from 'react';
import { View, Text, StyleSheet, ScrollView } from 'react-native';
import { PlatformFeatures } from '../utils/PlatformComparison';

export const PlatformComparison: React.FC = () => {
  const features = PlatformFeatures.getPlatformFeatures();
  const apis = PlatformFeatures.getAvailableAPIs();

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Platform Features</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Available Features:</Text>
        {features.map((feature, index) => (
          <Text key={index} style={styles.featureItem}>
            • {feature}
          </Text>
        ))}
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Available APIs:</Text>
        {apis.map((api, index) => (
          <Text key={index} style={styles.apiItem}>
            • {api}
          </Text>
        ))}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f9f9f9',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  section: {
    marginBottom: 20,
    backgroundColor: 'white',
    padding: 16,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  featureItem: {
    fontSize: 14,
    marginBottom: 5,
    color: '#666',
  },
  apiItem: {
    fontSize: 14,
    marginBottom: 5,
    color: '#666',
    fontFamily: 'monospace',
  },
});
```

## 4. Testing Multiplataforma

### Setup de Testing
```typescript
// __tests__/setup.ts
import 'react-native-gesture-handler/jestSetup';

// Mock para diferentes plataformas
jest.mock('react-native/Libraries/Utilities/Platform', () => ({
  OS: 'windows', // Cambiar según la plataforma a testear
  select: jest.fn((obj) => obj.windows || obj.default),
}));

// Mock para APIs nativas
jest.mock('react-native', () => {
  const RN = jest.requireActual('react-native');
  
  return {
    ...RN,
    NativeModules: {
      ...RN.NativeModules,
      WindowsInfo: {
        getVersion: jest.fn(() => Promise.resolve('10.0.19041')),
      },
      MacOSInfo: {
        getVersion: jest.fn(() => Promise.resolve('11.0')),
      },
    },
  };
});
```

### Test de Plataforma Específica
```typescript
// __tests__/PlatformSpecific.test.tsx
import React from 'react';
import { render } from '@testing-library/react-native';
import { Platform } from 'react-native';
import { WindowsSpecific } from '../components/WindowsSpecific';
import { MacOSSpecific } from '../components/MacOSSpecific';

describe('Platform Specific Components', () => {
  it('renders Windows-specific content on Windows', () => {
    Platform.OS = 'windows';
    const { getByText } = render(<WindowsSpecific />);
    
    expect(getByText('Windows App')).toBeTruthy();
    expect(getByText('Windows-specific features:')).toBeTruthy();
  });

  it('renders macOS-specific content on macOS', () => {
    Platform.OS = 'macos';
    const { getByText } = render(<MacOSSpecific />);
    
    expect(getByText('macOS App')).toBeTruthy();
    expect(getByText('macOS-specific features:')).toBeTruthy();
  });
});
```

### Test de APIs Nativas
```typescript
// __tests__/NativeAPIs.test.tsx
import { WindowsAPI, MacOSAPI } from '../utils/PlatformAPI';

describe('Native APIs', () => {
  it('gets Windows version', async () => {
    const version = await WindowsAPI.getWindowsVersion();
    expect(version).toBe('10.0.19041');
  });

  it('gets macOS version', async () => {
    const version = await MacOSAPI.getMacOSVersion();
    expect(version).toBe('11.0');
  });

  it('handles non-Windows platform', async () => {
    const version = await WindowsAPI.getWindowsVersion();
    expect(version).toBe('Not Windows');
  });
});
```

## 5. Configuración de Build

### Scripts de Build
```json
{
  "scripts": {
    "build:windows": "react-native run-windows --release",
    "build:macos": "react-native run-macos --release",
    "build:web": "webpack --mode production",
    "build:all": "npm run build:web && npm run build:windows && npm run build:macos",
    "test:windows": "jest --testPathPattern=windows",
    "test:macos": "jest --testPathPattern=macos",
    "test:all": "jest --testPathPattern=windows && jest --testPathPattern=macos"
  }
}
```

### Configuración de CI/CD
```yaml
# .github/workflows/build.yml
name: Multiplatform Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-web:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm install
      - run: npm run build:web

  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm install
      - run: npm run build:windows

  build-macos:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm install
      - run: npm run build:macos
```

## 6. Optimizaciones Específicas

### Performance para Desktop
```typescript
// utils/DesktopOptimizations.ts
import { Platform } from 'react-native';

export const DesktopOptimizations = {
  // Optimizaciones específicas para desktop
  enableDesktopFeatures: () => {
    if (Platform.OS === 'windows' || Platform.OS === 'macos') {
      // Habilitar características específicas de desktop
      console.log('Desktop features enabled');
      
      // Optimizar para pantallas grandes
      // Habilitar atajos de teclado
      // Optimizar para mouse/trackpad
    }
  },

  // Gestión de memoria para desktop
  optimizeMemoryUsage: () => {
    if (Platform.OS === 'windows' || Platform.OS === 'macos') {
      // Implementar optimizaciones de memoria
      console.log('Memory optimization enabled');
    }
  },

  // Optimización de GPU
  enableGPUAcceleration: () => {
    if (Platform.OS === 'windows') {
      // Habilitar DirectX
      console.log('DirectX acceleration enabled');
    } else if (Platform.OS === 'macos') {
      // Habilitar Metal
      console.log('Metal acceleration enabled');
    }
  },
};
```

### Responsive Design para Desktop
```typescript
// components/DesktopLayout.tsx
import React from 'react';
import { View, StyleSheet, Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

export const DesktopLayout: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const isDesktop = width >= 1024;
  const isTablet = width >= 768 && width < 1024;

  return (
    <View style={[
      styles.container,
      isDesktop && styles.desktopContainer,
      isTablet && styles.tabletContainer,
    ]}>
      {children}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  desktopContainer: {
    maxWidth: 1200,
    alignSelf: 'center',
    paddingHorizontal: 40,
  },
  tabletContainer: {
    paddingHorizontal: 20,
  },
});
```

## 7. Mejores Prácticas

### Desarrollo Multiplataforma
- **Testing**: Probar en todas las plataformas objetivo
- **Performance**: Optimizar para cada plataforma
- **UX**: Adaptar la experiencia a cada plataforma
- **APIs**: Usar APIs nativas cuando sea necesario

### Consideraciones de Desktop
- **Atajos de teclado**: Implementar atajos estándar
- **Menús**: Usar menús nativos de la plataforma
- **Ventanas**: Gestionar ventanas apropiadamente
- **Integración**: Integrar con el sistema operativo

## Conclusión

Las plataformas desktop (Windows y macOS) ofrecen:

- **APIs nativas** específicas de cada plataforma
- **Características únicas** como Touch Bar, Live Tiles
- **Mejor performance** en hardware desktop
- **Integración** con el sistema operativo

En la siguiente clase exploraremos estrategias de code sharing y reutilización.

## Tarea

1. Configurar React Native Windows en un proyecto
2. Configurar React Native macOS en un proyecto
3. Crear componentes específicos para cada plataforma
4. Implementar APIs nativas específicas
5. Configurar testing para plataformas desktop

## Enlaces Útiles

- [React Native Windows](https://github.com/microsoft/react-native-windows)
- [React Native macOS](https://github.com/microsoft/react-native-macos)
- [Windows App Development](https://docs.microsoft.com/en-us/windows/apps/)
- [macOS App Development](https://developer.apple.com/macos/)
