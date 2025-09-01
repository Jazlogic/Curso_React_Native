# Clase 4: Testing Automatizado en CI/CD 🧪

## Objetivos de la Clase
- Implementar testing automatizado en pipelines de CI/CD
- Configurar Jest, Detox y testing de integración
- Crear reportes de testing automatizados
- Integrar testing con workflows de CI/CD

## Duración Estimada
**2 horas**

## Contenido Teórico

### 1. Testing en CI/CD

#### Estrategia de Testing
```javascript
// testing/testing-strategy.js
class TestingStrategy {
  constructor() {
    this.testLevels = {
      unit: 'Componentes y funciones individuales',
      integration: 'Interacción entre componentes',
      e2e: 'Flujos completos de usuario',
      performance: 'Rendimiento y métricas',
      security: 'Vulnerabilidades y seguridad'
    };
  }
  
  // Determinar qué tests ejecutar según el contexto
  determineTestScope(context) {
    const { branch, changes, environment } = context;
    
    if (branch === 'main') {
      return ['unit', 'integration', 'e2e', 'performance', 'security'];
    } else if (branch === 'develop') {
      return ['unit', 'integration', 'e2e'];
    } else {
      return ['unit', 'integration'];
    }
  }
}
```

### 2. Jest para Testing Unitario

#### Configuración de Jest
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  transformIgnorePatterns: [
    'node_modules/(?!(react-native|@react-native|@react-navigation)/)'
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{js,jsx,ts,tsx}'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{js,jsx,ts,tsx}'
  ]
};
```

#### Tests Unitarios
```javascript
// src/components/Button/__tests__/Button.test.js
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import Button from '../Button';

describe('Button Component', () => {
  it('renders correctly with title', () => {
    const { getByText } = render(<Button title="Press Me" />);
    expect(getByText('Press Me')).toBeTruthy();
  });
  
  it('calls onPress when pressed', () => {
    const mockOnPress = jest.fn();
    const { getByText } = render(
      <Button title="Press Me" onPress={mockOnPress} />
    );
    
    fireEvent.press(getByText('Press Me'));
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });
  
  it('applies custom styles', () => {
    const customStyle = { backgroundColor: 'red' };
    const { getByTestId } = render(
      <Button title="Press Me" style={customStyle} testID="custom-button" />
    );
    
    const button = getByTestId('custom-button');
    expect(button.props.style).toContainEqual(customStyle);
  });
});
```

### 3. Detox para Testing E2E

#### Configuración de Detox
```javascript
// .detoxrc.js
module.exports = {
  testRunner: 'jest',
  runnerConfig: 'e2e/config.json',
  configurations: {
    'ios.sim.debug': {
      type: 'ios.simulator',
      device: 'iPhone 14',
      build: 'xcode',
      binaryPath: 'ios/build/Build/Products/Debug-iphonesimulator/MyApp.app'
    },
    'android.emu.debug': {
      type: 'android.emulator',
      device: 'Pixel_4_API_30',
      build: 'gradle',
      binaryPath: 'android/app/build/outputs/apk/debug/app-debug.apk'
    }
  }
};
```

#### Tests E2E
```javascript
// e2e/firstTest.e2e.js
describe('App Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });
  
  beforeEach(async () => {
    await device.reloadReactNative();
  });
  
  it('should navigate through main screens', async () => {
    // Verificar pantalla inicial
    await expect(element(by.text('Welcome'))).toBeVisible();
    
    // Navegar a segunda pantalla
    await element(by.id('next-button')).tap();
    await expect(element(by.text('Second Screen'))).toBeVisible();
    
    // Volver atrás
    await element(by.id('back-button')).tap();
    await expect(element(by.text('Welcome'))).toBeVisible();
  });
  
  it('should handle user input', async () => {
    const input = element(by.id('text-input'));
    const submitButton = element(by.id('submit-button'));
    
    await input.typeText('Hello World');
    await submitButton.tap();
    
    await expect(element(by.text('Hello World'))).toBeVisible();
  });
});
```

### 4. Testing de Integración

#### Tests de API
```javascript
// tests/integration/api.test.js
import { apiClient } from '../../src/services/api';

describe('API Integration Tests', () => {
  it('should fetch user data successfully', async () => {
    const userData = await apiClient.getUser(1);
    
    expect(userData).toHaveProperty('id');
    expect(userData).toHaveProperty('name');
    expect(userData).toHaveProperty('email');
  });
  
  it('should handle API errors gracefully', async () => {
    try {
      await apiClient.getUser(999999);
      fail('Should have thrown an error');
    } catch (error) {
      expect(error.status).toBe(404);
      expect(error.message).toContain('User not found');
    }
  });
});
```

### 5. Testing en CI/CD

#### Workflow de Testing
```yaml
# .github/workflows/testing.yml
name: Automated Testing

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit -- --coverage --watchAll=false
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  e2e-tests:
    name: E2E Tests
    runs-on: macos-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install && cd ..
          
      - name: Build for testing
        run: |
          cd ios
          xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 14' -derivedDataPath build
          
      - name: Run E2E tests
        run: npm run test:e2e:ios
        
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: e2e-results
          path: e2e/artifacts/
```

## Ejercicios Prácticos

### Ejercicio 1: Suite de Testing Completa
Implementa una suite de testing que cubra todos los niveles:

**Solución:**
```javascript
// tests/test-suite.js
import { runUnitTests } from './unit/runUnitTests';
import { runIntegrationTests } from './integration/runIntegrationTests';
import { runE2ETests } from './e2e/runE2ETests';
import { generateTestReport } from './utils/generateTestReport';

class TestSuite {
  constructor() {
    this.results = {
      unit: { passed: 0, failed: 0, total: 0 },
      integration: { passed: 0, failed: 0, total: 0 },
      e2e: { passed: 0, failed: 0, total: 0 }
    };
  }
  
  async runAllTests() {
    console.log('🧪 Starting comprehensive test suite...');
    
    try {
      // Tests unitarios
      this.results.unit = await runUnitTests();
      
      // Tests de integración
      this.results.integration = await runIntegrationTests();
      
      // Tests E2E
      this.results.e2e = await runE2ETests();
      
      // Generar reporte
      await generateTestReport(this.results);
      
      console.log('✅ All tests completed successfully!');
      return this.results;
      
    } catch (error) {
      console.error('❌ Test suite failed:', error);
      throw error;
    }
  }
  
  getSummary() {
    const total = {
      passed: this.results.unit.passed + this.results.integration.passed + this.results.e2e.passed,
      failed: this.results.unit.failed + this.results.integration.failed + this.results.e2e.failed,
      total: this.results.unit.total + this.results.integration.total + this.results.e2e.total
    };
    
    return {
      ...this.results,
      total,
      successRate: ((total.passed / total.total) * 100).toFixed(2) + '%'
    };
  }
}

export default TestSuite;
```

### Ejercicio 2: Testing de Performance
Implementa tests de performance automatizados:

**Solución:**
```javascript
// tests/performance/performance-tests.js
import { PerformanceMonitor } from '../../src/utils/PerformanceMonitor';

describe('Performance Tests', () => {
  let performanceMonitor;
  
  beforeEach(() => {
    performanceMonitor = new PerformanceMonitor();
  });
  
  it('should render main screen within 100ms', async () => {
    const startTime = performance.now();
    
    // Renderizar pantalla principal
    await renderMainScreen();
    
    const renderTime = performance.now() - startTime;
    expect(renderTime).toBeLessThan(100);
  });
  
  it('should handle large lists efficiently', async () => {
    const largeList = Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      title: `Item ${i}`,
      description: `Description for item ${i}`
    }));
    
    const startTime = performance.now();
    
    // Renderizar lista
    await renderList(largeList);
    
    const renderTime = performance.now() - startTime;
    expect(renderTime).toBeLessThan(500);
  });
  
  it('should maintain 60fps during animations', async () => {
    const fps = await performanceMonitor.measureFPS(() => {
      // Ejecutar animación
      runAnimation();
    });
    
    expect(fps).toBeGreaterThan(55);
  });
});
```

### Ejercicio 3: Testing de Seguridad
Implementa tests de seguridad automatizados:

**Solución:**
```javascript
// tests/security/security-tests.js
import { SecurityScanner } from '../../src/utils/SecurityScanner';

describe('Security Tests', () => {
  let securityScanner;
  
  beforeEach(() => {
    securityScanner = new SecurityScanner();
  });
  
  it('should not expose sensitive data in logs', async () => {
    const logs = await securityScanner.captureLogs();
    
    const sensitivePatterns = [
      /password/i,
      /token/i,
      /api_key/i,
      /secret/i
    ];
    
    sensitivePatterns.forEach(pattern => {
      expect(logs).not.toMatch(pattern);
    });
  });
  
  it('should validate input sanitization', async () => {
    const maliciousInputs = [
      '<script>alert("xss")</script>',
      'javascript:alert("xss")',
      'data:text/html,<script>alert("xss")</script>'
    ];
    
    maliciousInputs.forEach(input => {
      const sanitized = securityScanner.sanitizeInput(input);
      expect(sanitized).not.toContain('<script>');
      expect(sanitized).not.toContain('javascript:');
    });
  });
  
  it('should enforce HTTPS for API calls', async () => {
    const apiCalls = await securityScanner.captureAPICalls();
    
    apiCalls.forEach(call => {
      expect(call.url).toMatch(/^https:\/\//);
    });
  });
});
```

## Resumen de la Clase

En esta clase hemos cubierto:

1. **Estrategia de testing** en CI/CD
2. **Jest** para testing unitario
3. **Detox** para testing E2E
4. **Testing de integración** y API
5. **Testing automatizado** en workflows de CI/CD

## Próximos Pasos

En la siguiente clase aprenderemos sobre **Despliegue y Monitoreo**, incluyendo App Store Connect, Google Play Console y monitoreo de crashes.

## Navegación
- **Clase Anterior**: [Clase 3: Fastlane para Automatización](clase_3_fastlane_automatizacion.md)
- **Clase Siguiente**: [Clase 5: Despliegue y Monitoreo](clase_5_despliegue_monitoreo.md)
- **Volver al Módulo**: [README del Módulo 12](README.md)
- **Volver al Índice**: [Índice Completo](../../INDICE_COMPLETO.md)
