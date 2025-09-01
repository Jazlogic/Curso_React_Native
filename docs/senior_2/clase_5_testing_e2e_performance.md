# 🧪 **Clase 5: Testing E2E y Performance** - React Native

## 📋 **Objetivos de la Clase**
- Aprender testing end-to-end con Detox
- Testing de performance y accesibilidad
- Implementar testing de navegación completa
- Testing de apps en dispositivos reales

## ⏱️ **Duración**
**1.5 horas**

## 🔧 **Configuración de Detox**

### Instalación de Dependencias
```bash
npm install --save-dev detox
npm install --save-dev @types/jest
```

### Configuración de Detox (detox.config.js)
```javascript
module.exports = {
  testRunner: 'jest',
  runnerConfig: 'e2e/config.json',
  configurations: {
    'ios.sim.debug': {
      type: 'ios.simulator',
      binaryPath: 'ios/build/Build/Products/Debug-iphonesimulator/YourApp.app',
      build: 'xcodebuild -workspace ios/YourApp.xcworkspace -scheme YourApp -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build',
      device: {
        type: 'iPhone 14'
      }
    },
    'android.emu.debug': {
      type: 'android.emulator',
      binaryPath: 'android/app/build/outputs/apk/debug/app-debug.apk',
      build: 'cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug',
      device: {
        avdName: 'Pixel_4_API_30'
      }
    }
  }
};
```

### Configuración de Jest para E2E (e2e/config.json)
```json
{
  "setupFilesAfterEnv": ["./init.js"],
  "testEnvironment": "node",
  "testRunner": "detox/runners/jest/stream",
  "testTimeout": 120000,
  "reporters": ["detox/runners/jest/streamReporter"],
  "verbose": true
}
```

## 📚 **Contenido Teórico**

### 1. **Testing E2E con Detox**

#### **Test de Flujo de Login Completo**
```typescript
// e2e/auth.e2e.ts
import { device, element, by, expect } from 'detox';

describe('Authentication Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('debe mostrar pantalla de login al abrir la app', async () => {
    await expect(element(by.text('Iniciar Sesión'))).toBeVisible();
    await expect(element(by.id('email-input'))).toBeVisible();
    await expect(element(by.id('password-input'))).toBeVisible();
    await expect(element(by.id('login-button'))).toBeVisible();
  });

  it('debe validar campos requeridos', async () => {
    const loginButton = element(by.id('login-button'));
    
    await loginButton.tap();
    
    await expect(element(by.text('El email es requerido'))).toBeVisible();
    await expect(element(by.text('La contraseña es requerida'))).toBeVisible();
  });

  it('debe mostrar error con credenciales inválidas', async () => {
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('invalid@email.com');
    await passwordInput.typeText('wrongpassword');
    await loginButton.tap();
    
    await expect(element(by.text('Credenciales inválidas'))).toBeVisible();
  });

  it('debe hacer login exitoso y navegar al home', async () => {
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    // Esperar a que se complete el login
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
    
    // Verificar que estamos en el home
    await expect(element(by.text('Dashboard'))).toBeVisible();
    await expect(element(by.id('user-profile'))).toBeVisible();
  });

  it('debe hacer logout y volver a la pantalla de login', async () => {
    // Primero hacer login
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
    
    // Hacer logout
    const profileButton = element(by.id('user-profile'));
    await profileButton.tap();
    
    const logoutButton = element(by.text('Cerrar Sesión'));
    await logoutButton.tap();
    
    // Verificar que volvimos al login
    await expect(element(by.text('Iniciar Sesión'))).toBeVisible();
    await expect(element(by.id('email-input'))).toBeVisible();
  });
});
```

#### **Test de Navegación Completa**
```typescript
// e2e/navigation.e2e.ts
import { device, element, by, expect } from 'detox';

describe('Navigation Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
    
    // Hacer login primero
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
  });

  beforeEach(async () => {
    // Navegar al home antes de cada test
    const homeTab = element(by.id('home-tab'));
    await homeTab.tap();
  });

  it('debe navegar entre tabs correctamente', async () => {
    // Verificar tab home activo
    await expect(element(by.text('Dashboard'))).toBeVisible();
    
    // Navegar a productos
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    await expect(element(by.text('Productos'))).toBeVisible();
    
    // Navegar a perfil
    const profileTab = element(by.id('profile-tab'));
    await profileTab.tap();
    await expect(element(by.text('Mi Perfil'))).toBeVisible();
    
    // Volver a home
    await homeTab.tap();
    await expect(element(by.text('Dashboard'))).toBeVisible();
  });

  it('debe navegar a detalle de producto y volver', async () => {
    // Ir a productos
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    // Seleccionar primer producto
    const firstProduct = element(by.id('product-item-0'));
    await firstProduct.tap();
    
    // Verificar detalle del producto
    await expect(element(by.text('Detalle del Producto'))).toBeVisible();
    await expect(element(by.id('product-image'))).toBeVisible();
    await expect(element(by.id('add-to-cart-button'))).toBeVisible();
    
    // Volver a la lista
    const backButton = element(by.id('back-button'));
    await backButton.tap();
    
    await expect(element(by.text('Productos'))).toBeVisible();
  });

  it('debe navegar a carrito y completar compra', async () => {
    // Ir a productos y agregar al carrito
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    const firstProduct = element(by.id('product-item-0'));
    await firstProduct.tap();
    
    const addToCartButton = element(by.id('add-to-cart-button'));
    await addToCartButton.tap();
    
    // Verificar que se agregó al carrito
    await expect(element(by.text('Producto agregado al carrito'))).toBeVisible();
    
    // Ir al carrito
    const cartTab = element(by.id('cart-tab'));
    await cartTab.tap();
    
    await expect(element(by.text('Carrito de Compras'))).toBeVisible();
    await expect(element(by.id('checkout-button'))).toBeVisible();
    
    // Completar compra
    const checkoutButton = element(by.id('checkout-button'));
    await checkoutButton.tap();
    
    // Verificar pantalla de checkout
    await expect(element(by.text('Finalizar Compra'))).toBeVisible();
    await expect(element(by.id('payment-form'))).toBeVisible();
  });
});
```

### 2. **Testing de Performance**

#### **Test de Rendimiento de Lista**
```typescript
// e2e/performance.e2e.ts
import { device, element, by, expect } from 'detox';

describe('Performance Tests', () => {
  beforeAll(async () => {
    await device.launchApp();
    
    // Hacer login
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
  });

  it('debe cargar lista de productos rápidamente', async () => {
    const startTime = Date.now();
    
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    // Esperar a que se cargue la lista
    await waitFor(element(by.id('products-list'))).toBeVisible().withTimeout(3000);
    
    const endTime = Date.now();
    const loadTime = endTime - startTime;
    
    // Verificar que la carga fue rápida (menos de 3 segundos)
    expect(loadTime).toBeLessThan(3000);
    
    // Verificar que se muestran productos
    await expect(element(by.id('product-item-0'))).toBeVisible();
  });

  it('debe hacer scroll suave en lista larga', async () => {
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    const productsList = element(by.id('products-list'));
    
    // Hacer scroll hacia abajo
    await productsList.scrollTo('bottom');
    
    // Verificar que se cargaron más productos
    await waitFor(element(by.id('product-item-20'))).toBeVisible().withTimeout(2000);
    
    // Hacer scroll hacia arriba
    await productsList.scrollTo('top');
    
    // Verificar que volvimos al inicio
    await expect(element(by.id('product-item-0'))).toBeVisible();
  });

  it('debe manejar búsqueda en tiempo real', async () => {
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    const searchInput = element(by.id('search-input'));
    await searchInput.tap();
    
    const startTime = Date.now();
    
    // Escribir búsqueda
    await searchInput.typeText('laptop');
    
    // Esperar resultados
    await waitFor(element(by.id('search-results'))).toBeVisible().withTimeout(2000);
    
    const endTime = Date.now();
    const searchTime = endTime - startTime;
    
    // Verificar que la búsqueda fue rápida
    expect(searchTime).toBeLessThan(2000);
    
    // Verificar resultados
    await expect(element(by.text('Laptop Gaming'))).toBeVisible();
  });
});
```

#### **Test de Memoria y Rendimiento**
```typescript
// e2e/memory.e2e.ts
import { device, element, by, expect } from 'detox';

describe('Memory and Performance Tests', () => {
  beforeAll(async () => {
    await device.launchApp();
    
    // Hacer login
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
  });

  it('debe mantener rendimiento después de navegación intensiva', async () => {
    const productsTab = element(by.id('products-tab'));
    const profileTab = element(by.id('profile-tab'));
    const homeTab = element(by.id('home-tab'));
    
    // Navegar múltiples veces entre tabs
    for (let i = 0; i < 10; i++) {
      await productsTab.tap();
      await waitFor(element(by.text('Productos'))).toBeVisible();
      
      await profileTab.tap();
      await waitFor(element(by.text('Mi Perfil'))).toBeVisible();
      
      await homeTab.tap();
      await waitFor(element(by.text('Dashboard'))).toBeVisible();
    }
    
    // Verificar que la app sigue funcionando correctamente
    await expect(element(by.text('Dashboard'))).toBeVisible();
    
    // Verificar que podemos interactuar con elementos
    const productsTab2 = element(by.id('products-tab'));
    await productsTab2.tap();
    await expect(element(by.text('Productos'))).toBeVisible();
  });

  it('debe manejar múltiples operaciones sin degradación', async () => {
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    // Agregar múltiples productos al carrito
    for (let i = 0; i < 5; i++) {
      const productItem = element(by.id(`product-item-${i}`));
      await productItem.tap();
      
      const addToCartButton = element(by.id('add-to-cart-button'));
      await addToCartButton.tap();
      
      // Esperar confirmación
      await waitFor(element(by.text('Producto agregado al carrito'))).toBeVisible();
      
      // Volver a la lista
      const backButton = element(by.id('back-button'));
      await backButton.tap();
    }
    
    // Verificar que el carrito tiene los productos
    const cartTab = element(by.id('cart-tab'));
    await cartTab.tap();
    
    await expect(element(by.text('Carrito de Compras'))).toBeVisible();
    await expect(element(by.id('cart-item-0'))).toBeVisible();
    await expect(element(by.id('cart-item-4'))).toBeVisible();
  });
});
```

### 3. **Testing de Accesibilidad**

#### **Test de Accesibilidad Básica**
```typescript
// e2e/accessibility.e2e.ts
import { device, element, by, expect } from 'detox';

describe('Accessibility Tests', () => {
  beforeAll(async () => {
    await device.launchApp();
    
    // Hacer login
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
  });

  it('debe tener labels accesibles en formularios', async () => {
    const profileTab = element(by.id('profile-tab'));
    await profileTab.tap();
    
    // Verificar que los campos tienen labels
    await expect(element(by.label('Nombre'))).toBeVisible();
    await expect(element(by.label('Email'))).toBeVisible();
    await expect(element(by.label('Teléfono'))).toBeVisible();
  });

  it('debe tener botones con texto descriptivo', async () => {
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    // Verificar que los botones tienen texto claro
    await expect(element(by.text('Agregar al Carrito'))).toBeVisible();
    await expect(element(by.text('Ver Detalles'))).toBeVisible();
    await expect(element(by.text('Filtrar'))).toBeVisible();
  });

  it('debe tener navegación accesible', async () => {
    // Verificar que los tabs tienen labels
    await expect(element(by.label('Inicio'))).toBeVisible();
    await expect(element(by.label('Productos'))).toBeVisible();
    await expect(element(by.label('Carrito'))).toBeVisible();
    await expect(element(by.label('Perfil'))).toBeVisible();
  });

  it('debe tener mensajes de error accesibles', async () => {
    const profileTab = element(by.id('profile-tab'));
    await profileTab.tap();
    
    const saveButton = element(by.id('save-profile-button'));
    await saveButton.tap();
    
    // Verificar que los errores son visibles y claros
    await expect(element(by.text('Por favor completa todos los campos requeridos'))).toBeVisible();
  });
});
```

### 4. **Testing de Casos Edge**

#### **Test de Manejo de Errores**
```typescript
// e2e/error-handling.e2e.ts
import { device, element, by, expect } from 'detox';

describe('Error Handling Tests', () => {
  beforeAll(async () => {
    await device.launchApp();
    
    // Hacer login
    const emailInput = element(by.id('email-input'));
    const passwordInput = element(by.id('password-input'));
    const loginButton = element(by.id('login-button'));
    
    await emailInput.typeText('user@email.com');
    await passwordInput.typeText('password123');
    await loginButton.tap();
    
    await waitFor(element(by.text('Bienvenido'))).toBeVisible().withTimeout(5000);
  });

  it('debe manejar pérdida de conexión', async () => {
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    // Simular pérdida de conexión (esto requeriría configuración especial)
    // await device.setNetworkState('offline');
    
    // Intentar cargar productos
    await waitFor(element(by.text('Sin conexión a internet'))).toBeVisible().withTimeout(5000);
    
    // Verificar botón de reintentar
    const retryButton = element(by.text('Reintentar'));
    await retryButton.tap();
    
    // Restaurar conexión
    // await device.setNetworkState('online');
    
    // Verificar que se cargaron los productos
    await waitFor(element(by.id('products-list'))).toBeVisible().withTimeout(5000);
  });

  it('debe manejar datos corruptos', async () => {
    const profileTab = element(by.id('profile-tab'));
    await profileTab.tap();
    
    // Simular datos corruptos (esto requeriría configuración especial del backend)
    // await device.sendToHome();
    // await device.launchApp();
    
    // Verificar que se muestra mensaje de error apropiado
    await expect(element(by.text('Error al cargar datos del perfil'))).toBeVisible();
    
    // Verificar botón de reintentar
    const retryButton = element(by.text('Reintentar'));
    await retryButton.tap();
    
    // Verificar que se cargó correctamente
    await waitFor(element(by.text('Mi Perfil'))).toBeVisible();
  });

  it('debe manejar timeouts de API', async () => {
    const productsTab = element(by.id('products-tab'));
    await productsTab.tap();
    
    // Simular timeout (esto requeriría configuración especial)
    // await device.setNetworkLatency(10000);
    
    // Intentar cargar productos
    await waitFor(element(by.text('La solicitud tardó demasiado'))).toBeVisible().withTimeout(15000);
    
    // Verificar botón de reintentar
    const retryButton = element(by.text('Reintentar'));
    await retryButton.tap();
    
    // Restaurar latencia normal
    // await device.setNetworkLatency(0);
    
    // Verificar que se cargaron los productos
    await waitFor(element(by.id('products-list'))).toBeVisible().withTimeout(5000);
  });
});
```

## 🧪 **Ejercicios Prácticos**

### **Ejercicio 1: Test E2E de Flujo de Registro**
Crea tests para un flujo completo de registro de usuario:

```typescript
// Flujo a testear
export const RegistrationFlow: React.FC = () => {
  // Implementa:
  // - Formulario de registro
  // - Validación de datos
  // - Confirmación de email
  // - Activación de cuenta
};

// Escribe tests que cubran:
// - Flujo completo de registro
// - Validaciones de formulario
// - Confirmación de email
// - Activación de cuenta
```

### **Ejercicio 2: Test de Performance de Imágenes**
Crea tests para verificar el rendimiento de carga de imágenes:

```typescript
// Componente a testear
export const ImageGallery: React.FC = () => {
  // Implementa:
  // - Galería de imágenes
  // - Lazy loading
  // - Cache de imágenes
  // - Optimización de tamaño
};

// Escribe tests que cubran:
// - Tiempo de carga de imágenes
// - Lazy loading
// - Cache de imágenes
// - Optimización de memoria
```

## 📝 **Resumen de la Clase**

### **Conceptos Clave Aprendidos**
1. **Testing E2E**: Verificación de flujos completos de usuario
2. **Detox**: Framework para testing E2E en React Native
3. **Testing de Performance**: Verificación de rendimiento y velocidad
4. **Testing de Accesibilidad**: Verificación de usabilidad para todos los usuarios
5. **Testing de Casos Edge**: Verificación de manejo de errores y situaciones límite
6. **Testing en Dispositivos Reales**: Verificación en entorno real

### **Próximos Pasos**
- Estrategias de testing y mejores prácticas
- Cobertura de tests y CI/CD
- Testing de aplicaciones complejas

## 🔗 **Navegación**
- [⬅️ Clase Anterior](clase_4_testing_integracion_apis.md) - Testing de Integración y APIs
- [🏠 Inicio](../../README.md)
- [📚 Índice Completo](../../INDICE_COMPLETO.md)
- [➡️ Siguiente Módulo](../senior_3/README.md) - State Management Avanzado

---

**💡 Tip**: Usa Detox para testing E2E real en dispositivos. Esto te permite detectar problemas que solo aparecen en el entorno real y no en simuladores.
