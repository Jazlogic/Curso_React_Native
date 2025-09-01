# Clase 3: Testing E2E con Detox 🧪

## Objetivos de la Clase
- Configurar Detox para testing E2E en React Native
- Crear escenarios de testing para flujos completos de usuario
- Implementar testing de navegación y interacciones complejas
- Configurar testing automatizado en CI/CD

## Contenido Teórico

### 1. Configuración de Detox

#### Instalación y Configuración Básica
```bash
# Instalar Detox CLI globalmente
npm install -g detox-cli

# Inicializar Detox en el proyecto
detox init

# Instalar dependencias
npm install --save-dev detox
```

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
      build: 'xcodebuild -workspace ios/YourApp.xcworkspace -scheme YourApp -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build',
    },
    'ios.sim.release': {
      type: 'ios.simulator',
      device: 'iPhone 14',
      build: 'xcodebuild -workspace ios/YourApp.xcworkspace -scheme YourApp -configuration Release -sdk iphonesimulator -derivedDataPath ios/build',
    },
    'android.emu.debug': {
      type: 'android.emulator',
      device: 'Pixel_4_API_30',
      build: 'cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug && cd ..',
      binaryPath: 'android/app/build/outputs/apk/debug/app-debug.apk',
    },
    'android.emu.release': {
      type: 'android.emulator',
      device: 'Pixel_4_API_30',
      build: 'cd android && ./gradlew assembleRelease assembleAndroidTest -DtestBuildType=release && cd ..',
      binaryPath: 'android/app/build/outputs/apk/release/app-release.apk',
    },
  },
  artifacts: {
    plugins: {
      screenshot: {
        enabled: true,
        shouldTakeAutomaticSnapshots: true,
        keepOnlyFailedTestsArtifacts: true,
      },
      video: {
        enabled: true,
        keepOnlyFailedTestsArtifacts: true,
      },
      instruments: {
        enabled: false,
      },
      network: {
        enabled: true,
        keepOnlyFailedTestsArtifacts: true,
      },
    },
  },
};
```

#### Configuración de Jest para E2E
```json
// e2e/config.json
{
  "setupFilesAfterEnv": ["./init.js"],
  "testEnvironment": "node",
  "testRunner": "detox/runners/jest/runner",
  "testTimeout": 120000,
  "reporters": ["detox/runners/jest/reporter"],
  "verbose": true
}
```

#### Archivo de Inicialización
```javascript
// e2e/init.js
import { device, element, by, expect } from 'detox';

beforeAll(async () => {
  await device.launchApp();
});

beforeEach(async () => {
  await device.reloadReactNative();
});

afterAll(async () => {
  await device.terminateApp();
});
```

### 2. Testing de Flujos de Usuario

#### Flujo de Autenticación Completo
```javascript
// e2e/flows/authFlow.e2e.js
import { device, element, by, expect } from 'detox';

describe('Authentication Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  describe('Login Flow', () => {
    it('debe mostrar pantalla de login inicialmente', async () => {
      // Verificar que estamos en la pantalla de login
      await expect(element(by.text('Login'))).toBeVisible();
      await expect(element(by.id('email-input'))).toBeVisible();
      await expect(element(by.id('password-input'))).toBeVisible();
      await expect(element(by.id('login-button'))).toBeVisible();
    });

    it('debe validar campos vacíos', async () => {
      // Intentar hacer login sin datos
      await element(by.id('login-button')).tap();
      
      // Verificar que aparece mensaje de error
      await expect(element(by.text('Please fill in all fields'))).toBeVisible();
    });

    it('debe validar formato de email', async () => {
      // Ingresar email inválido
      await element(by.id('email-input')).typeText('invalid-email');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Verificar mensaje de error
      await expect(element(by.text('Please enter a valid email'))).toBeVisible();
    });

    it('debe hacer login exitoso', async () => {
      // Ingresar credenciales válidas
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Verificar que se navega a la pantalla principal
      await expect(element(by.text('Welcome'))).toBeVisible();
      await expect(element(by.text('Home'))).toBeVisible();
    });

    it('debe manejar credenciales inválidas', async () => {
      // Ingresar credenciales incorrectas
      await element(by.id('email-input')).typeText('wrong@example.com');
      await element(by.id('password-input')).typeText('wrongpassword');
      await element(by.id('login-button')).tap();
      
      // Verificar mensaje de error
      await expect(element(by.text('Invalid credentials'))).toBeVisible();
    });
  });

  describe('Registration Flow', () => {
    it('debe navegar a pantalla de registro', async () => {
      // Tocar en "Create Account"
      await element(by.text('Create Account')).tap();
      
      // Verificar que estamos en la pantalla de registro
      await expect(element(by.text('Register'))).toBeVisible();
      await expect(element(by.id('name-input'))).toBeVisible();
      await expect(element(by.id('email-input'))).toBeVisible();
      await expect(element(by.id('password-input'))).toBeVisible();
      await expect(element(by.id('confirm-password-input'))).toBeVisible();
    });

    it('debe validar contraseñas coincidentes', async () => {
      // Navegar a registro
      await element(by.text('Create Account')).tap();
      
      // Llenar formulario con contraseñas diferentes
      await element(by.id('name-input')).typeText('John Doe');
      await element(by.id('email-input')).typeText('john@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('confirm-password-input')).typeText('password456');
      await element(by.id('register-button')).tap();
      
      // Verificar mensaje de error
      await expect(element(by.text('Passwords do not match'))).toBeVisible();
    });

    it('debe registrar usuario exitosamente', async () => {
      // Navegar a registro
      await element(by.text('Create Account')).tap();
      
      // Llenar formulario correctamente
      await element(by.id('name-input')).typeText('John Doe');
      await element(by.id('email-input')).typeText('john@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('confirm-password-input')).typeText('password123');
      await element(by.id('register-button')).tap();
      
      // Verificar que se navega a la pantalla principal
      await expect(element(by.text('Welcome John Doe'))).toBeVisible();
    });
  });

  describe('Logout Flow', () => {
    it('debe hacer logout exitosamente', async () => {
      // Primero hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Verificar que estamos logueados
      await expect(element(by.text('Welcome'))).toBeVisible();
      
      // Ir a configuración
      await element(by.text('Settings')).tap();
      
      // Hacer logout
      await element(by.id('logout-button')).tap();
      
      // Verificar que volvimos a la pantalla de login
      await expect(element(by.text('Login'))).toBeVisible();
    });
  });
});
```

#### Flujo de Navegación Principal
```javascript
// e2e/flows/navigationFlow.e2e.js
import { device, element, by, expect } from 'detox';

describe('Navigation Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
    // Hacer login para acceder a la navegación principal
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
    // Hacer login nuevamente
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
  });

  describe('Tab Navigation', () => {
    it('debe navegar entre tabs correctamente', async () => {
      // Verificar que estamos en Home
      await expect(element(by.text('Home'))).toBeVisible();
      await expect(element(by.text('Welcome to the app'))).toBeVisible();
      
      // Navegar a Users
      await element(by.text('Users')).tap();
      await expect(element(by.text('Users List'))).toBeVisible();
      
      // Navegar a Profile
      await element(by.text('Profile')).tap();
      await expect(element(by.text('User Profile'))).toBeVisible();
      
      // Navegar a Settings
      await element(by.text('Settings')).tap();
      await expect(element(by.text('App Settings'))).toBeVisible();
      
      // Volver a Home
      await element(by.text('Home')).tap();
      await expect(element(by.text('Welcome to the app'))).toBeVisible();
    });

    it('debe mantener estado entre tabs', async () => {
      // Ir a Users y crear un usuario
      await element(by.text('Users')).tap();
      await element(by.id('add-user-button')).tap();
      await element(by.id('user-name-input')).typeText('Test User');
      await element(by.id('user-email-input')).typeText('testuser@example.com');
      await element(by.id('save-user-button')).tap();
      
      // Verificar que el usuario se creó
      await expect(element(by.text('Test User'))).toBeVisible();
      
      // Ir a otro tab y volver
      await element(by.text('Profile')).tap();
      await element(by.text('Users')).tap();
      
      // Verificar que el usuario sigue ahí
      await expect(element(by.text('Test User'))).toBeVisible();
    });
  });

  describe('Stack Navigation', () => {
    it('debe navegar a detalles de usuario', async () => {
      // Ir a Users
      await element(by.text('Users')).tap();
      
      // Tocar en un usuario
      await element(by.text('John Doe')).tap();
      
      // Verificar que estamos en detalles
      await expect(element(by.text('User Details'))).toBeVisible();
      await expect(element(by.text('John Doe'))).toBeVisible();
      await expect(element(by.text('john@example.com'))).toBeVisible();
    });

    it('debe volver atrás correctamente', async () => {
      // Ir a Users
      await element(by.text('Users')).tap();
      
      // Ir a detalles
      await element(by.text('John Doe')).tap();
      await expect(element(by.text('User Details'))).toBeVisible();
      
      // Volver atrás
      await element(by.id('back-button')).tap();
      
      // Verificar que estamos en la lista
      await expect(element(by.text('Users List'))).toBeVisible();
    });

    it('debe navegar a edición de usuario', async () => {
      // Ir a Users
      await element(by.text('Users')).tap();
      
      // Ir a detalles
      await element(by.text('John Doe')).tap();
      
      // Editar usuario
      await element(by.id('edit-button')).tap();
      await expect(element(by.text('Edit User'))).toBeVisible();
      
      // Cambiar nombre
      await element(by.id('edit-name-input')).clearText();
      await element(by.id('edit-name-input')).typeText('John Updated');
      
      // Guardar cambios
      await element(by.id('save-button')).tap();
      
      // Verificar que se guardó
      await expect(element(by.text('User updated successfully'))).toBeVisible();
    });
  });

  describe('Deep Linking', () => {
    it('debe abrir app en pantalla específica', async () => {
      // Simular deep link a perfil de usuario
      await device.openURL('myapp://user/123/profile');
      
      // Verificar que se abre en el perfil correcto
      await expect(element(by.text('User Profile'))).toBeVisible();
      await expect(element(by.text('User ID: 123'))).toBeVisible();
    });

    it('debe manejar deep links inválidos', async () => {
      // Simular deep link inválido
      await device.openURL('myapp://invalid/route');
      
      // Verificar que se maneja correctamente
      await expect(element(by.text('Page not found'))).toBeVisible();
    });
  });
});
```

### 3. Testing de Interacciones Complejas

#### Testing de Formularios Dinámicos
```javascript
// e2e/flows/formInteractions.e2e.js
import { device, element, by, expect } from 'detox';

describe('Form Interactions', () => {
  beforeAll(async () => {
    await device.launchApp();
    // Login
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
    // Login nuevamente
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();
  });

  describe('Dynamic Form Validation', () => {
    it('debe validar formulario en tiempo real', async () => {
      // Ir a crear usuario
      await element(by.text('Users')).tap();
      await element(by.id('add-user-button')).tap();
      
      // Verificar validación de email
      await element(by.id('user-email-input')).typeText('invalid-email');
      await element(by.id('user-email-input')).tap();
      
      // Verificar mensaje de error
      await expect(element(by.text('Please enter a valid email'))).toBeVisible();
      
      // Corregir email
      await element(by.id('user-email-input')).clearText();
      await element(by.id('user-email-input')).typeText('valid@example.com');
      
      // Verificar que el error desaparece
      await expect(element(by.text('Please enter a valid email'))).not.toBeVisible();
    });

    it('debe mostrar/ocultar campos condicionales', async () => {
      // Ir a crear usuario
      await element(by.text('Users')).tap();
      await element(by.id('add-user-button')).tap();
      
      // Seleccionar tipo de usuario "Admin"
      await element(by.id('user-type-selector')).tap();
      await element(by.text('Admin')).tap();
      
      // Verificar que aparece campo de permisos
      await expect(element(by.id('permissions-input'))).toBeVisible();
      
      // Cambiar a tipo "User"
      await element(by.id('user-type-selector')).tap();
      await element(by.text('User')).tap();
      
      // Verificar que desaparece campo de permisos
      await expect(element(by.id('permissions-input'))).not.toBeVisible();
    });
  });

  describe('File Upload', () => {
    it('debe permitir subir imagen de perfil', async () => {
      // Ir a perfil
      await element(by.text('Profile')).tap();
      
      // Tocar en imagen de perfil
      await element(by.id('profile-image')).tap();
      
      // Seleccionar opción de cámara
      await element(by.text('Take Photo')).tap();
      
      // Simular captura de foto
      await device.takeScreenshot('profile-photo-capture');
      
      // Verificar que se actualiza la imagen
      await expect(element(by.id('profile-image-updated'))).toBeVisible();
    });

    it('debe validar tipo y tamaño de archivo', async () => {
      // Ir a perfil
      await element(by.text('Profile')).tap();
      
      // Intentar subir archivo grande
      await element(by.id('profile-image')).tap();
      await element(by.text('Choose from Library')).tap();
      
      // Simular selección de archivo grande
      // (En un test real, esto requeriría mock del selector de archivos)
      
      // Verificar mensaje de error
      await expect(element(by.text('File size too large'))).toBeVisible();
    });
  });

  describe('Search and Filter', () => {
    it('debe filtrar usuarios en tiempo real', async () => {
      // Ir a Users
      await element(by.text('Users')).tap();
      
      // Buscar usuario específico
      await element(by.id('search-input')).typeText('John');
      
      // Verificar que se filtran los resultados
      await expect(element(by.text('John Doe'))).toBeVisible();
      await expect(element(by.text('Jane Smith'))).not.toBeVisible();
      
      // Limpiar búsqueda
      await element(by.id('search-input')).clearText();
      
      // Verificar que se muestran todos los usuarios
      await expect(element(by.text('John Doe'))).toBeVisible();
      await expect(element(by.text('Jane Smith'))).toBeVisible();
    });

    it('debe aplicar filtros múltiples', async () => {
      // Ir a Users
      await element(by.text('Users')).tap();
      
      // Abrir filtros
      await element(by.id('filters-button')).tap();
      
      // Filtrar por rol
      await element(by.id('role-filter')).tap();
      await element(by.text('Admin')).tap();
      
      // Filtrar por estado
      await element(by.id('status-filter')).tap();
      await element(by.text('Active')).tap();
      
      // Aplicar filtros
      await element(by.id('apply-filters-button')).tap();
      
      // Verificar resultados filtrados
      await expect(element(by.text('Admin Users'))).toBeVisible();
      await expect(element(by.text('User Users'))).not.toBeVisible();
    });
  });
});
```

### 4. Testing de Rendimiento y Estabilidad

#### Testing de Rendimiento
```javascript
// e2e/flows/performance.e2e.js
import { device, element, by, expect } from 'detox';

describe('Performance Testing', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  describe('App Launch Performance', () => {
    it('debe abrir app en menos de 3 segundos', async () => {
      const startTime = Date.now();
      
      // Esperar a que la app esté lista
      await expect(element(by.text('Login'))).toBeVisible();
      
      const loadTime = Date.now() - startTime;
      expect(loadTime).toBeLessThan(3000);
    });

    it('debe cargar lista de usuarios eficientemente', async () => {
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Ir a Users
      await element(by.text('Users')).tap();
      
      const startTime = Date.now();
      
      // Esperar a que se cargue la lista
      await expect(element(by.text('Users List'))).toBeVisible();
      
      const loadTime = Date.now() - startTime;
      expect(loadTime).toBeLessThan(2000);
    });
  });

  describe('Memory Usage', () => {
    it('debe mantener uso de memoria estable', async () => {
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Navegar entre pantallas múltiples veces
      for (let i = 0; i < 10; i++) {
        await element(by.text('Users')).tap();
        await element(by.text('Profile')).tap();
        await element(by.text('Settings')).tap();
        await element(by.text('Home')).tap();
      }
      
      // Verificar que la app sigue funcionando
      await expect(element(by.text('Welcome to the app'))).toBeVisible();
    });
  });

  describe('Network Performance', () => {
    it('debe manejar conexiones lentas', async () => {
      // Simular conexión lenta (requiere configuración de proxy)
      await device.setURLBlacklist(['.*']);
      
      // Hacer login
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      
      // Verificar que se muestra loading
      await expect(element(by.text('Loading...'))).toBeVisible();
      
      // Restaurar conexión normal
      await device.setURLBlacklist([]);
      
      // Verificar que se completa el login
      await expect(element(by.text('Welcome'))).toBeVisible();
    });
  });
});
```

## Ejercicios Prácticos

### Ejercicio 1: Flujo de E-commerce Completo
Crea tests E2E para un flujo de compra completo:

```javascript
// Flujo a implementar
describe('E-commerce Flow', () => {
  it('debe completar proceso de compra completo', async () => {
    // 1. Login
    // 2. Navegar a productos
    // 3. Agregar productos al carrito
    // 4. Ir al carrito
    // 5. Proceder al checkout
    // 6. Llenar información de envío
    // 7. Seleccionar método de pago
    // 8. Confirmar orden
    // 9. Verificar confirmación
  });
});
```

**Requisitos del testing:**
- Test de navegación entre pantallas
- Test de interacciones con formularios
- Test de validaciones en tiempo real
- Test de manejo de errores
- Test de confirmaciones y mensajes

### Ejercicio 2: Testing de Chat en Tiempo Real
Crea tests E2E para una funcionalidad de chat:

```javascript
// Funcionalidad a implementar
describe('Real-time Chat', () => {
  it('debe enviar y recibir mensajes en tiempo real', async () => {
    // 1. Abrir chat
    // 2. Enviar mensaje
    // 3. Verificar que aparece
    // 4. Simular respuesta del otro usuario
    // 5. Verificar que se recibe
    // 6. Test de notificaciones
  });
});
```

**Requisitos del testing:**
- Test de envío de mensajes
- Test de recepción en tiempo real
- Test de notificaciones push
- Test de historial de mensajes
- Test de manejo de conexión

### Ejercicio 3: Testing de Offline/Online
Crea tests E2E para funcionalidad offline:

```javascript
// Funcionalidad a implementar
describe('Offline Functionality', () => {
  it('debe funcionar correctamente sin conexión', async () => {
    // 1. Simular modo offline
    // 2. Verificar que se muestra indicador offline
    // 3. Intentar acciones que requieren conexión
    // 4. Verificar que se guardan para sincronización
    // 5. Restaurar conexión
    // 6. Verificar sincronización
  });
});
```

**Requisitos del testing:**
- Test de detección de estado offline
- Test de funcionalidad offline
- Test de sincronización al volver online
- Test de manejo de conflictos
- Test de indicadores de estado

## Resumen de la Clase

En esta clase hemos aprendido:

1. **Configuración de Detox**: Setup completo para testing E2E en iOS y Android
2. **Testing de Flujos**: Testing de flujos completos de usuario y navegación
3. **Testing de Interacciones**: Testing de formularios, archivos y búsquedas
4. **Testing de Rendimiento**: Testing de velocidad, memoria y red

## Navegación
- **Anterior**: [Clase 2: Testing de Integración](clase_2_testing_integracion.md)
- **Siguiente**: [Clase 4: Testing de Rendimiento](clase_4_testing_rendimiento.md)
- **Inicio**: [Índice del Curso](../../INDICE_COMPLETO.md)
