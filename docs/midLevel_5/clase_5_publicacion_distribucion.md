# 📤 Clase 5: Publicación y Distribución

## 🎯 Objetivos de la Clase
- Publicar y distribuir aplicaciones Expo
- Configurar Expo Application Services (EAS)
- Generar builds nativos para iOS y Android
- Publicar en App Store y Google Play Store

---

## 📚 Contenido Teórico

### 1. Expo Application Services (EAS)

#### **¿Qué es EAS?**
**Expo Application Services (EAS)** es un conjunto de servicios en la nube que simplifica el proceso de build, testing y distribución de aplicaciones:

- **EAS Build**: Genera builds nativos en la nube
- **EAS Submit**: Envía apps a las tiendas automáticamente
- **EAS Update**: Gestiona actualizaciones OTA
- **EAS Analytics**: Proporciona métricas de uso

#### **Instalación y Configuración**
```bash
# Instalar EAS CLI
npm install -g @expo/eas-cli

# Verificar instalación
eas --version

# Iniciar sesión en Expo
eas login

# Configurar proyecto
eas build:configure
```

#### **Configuración del Proyecto**
```javascript
// eas.json
{
  "cli": {
    "version": ">= 3.13.3"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": true
      }
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

### 2. EAS Build - Generación de Builds

#### **Configuración de Builds**
```bash
# Configurar build para desarrollo
eas build --profile development --platform ios

# Configurar build para preview
eas build --profile preview --platform android

# Configurar build para producción
eas build --profile production --platform all
```

#### **Configuración Avanzada de Builds**
```javascript
// eas.json con configuración avanzada
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://dev-api.miapp.com"
      },
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "apk"
      }
    },
    "preview": {
      "distribution": "internal",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://staging-api.miapp.com"
      },
      "ios": {
        "simulator": true,
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "aab"
      }
    },
    "production": {
      "autoIncrement": true,
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.miapp.com"
      },
      "ios": {
        "resourceClass": "m-large",
        "credentialsSource": "remote"
      },
      "android": {
        "buildType": "aab",
        "credentialsSource": "remote"
      }
    }
  }
}
```

#### **Configuración de Credenciales**
```bash
# Configurar credenciales de iOS
eas credentials

# Configurar credenciales de Android
eas credentials

# Ver credenciales configuradas
eas credentials:list
```

#### **Builds Automatizados con GitHub Actions**
```yaml
# .github/workflows/eas-build.yml
name: EAS Build
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Setup EAS
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Build Android APK
        run: eas build --platform android --profile preview --non-interactive

      - name: Build iOS Simulator
        run: eas build --platform ios --profile preview --non-interactive

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-artifacts
          path: |
            *.apk
            *.ipa
```

### 3. EAS Submit - Envío a Tiendas

#### **Configuración de Submit**
```bash
# Configurar submit para iOS
eas submit --platform ios

# Configurar submit para Android
eas submit --platform android

# Submit automático después del build
eas build --auto-submit
```

#### **Configuración Avanzada de Submit**
```javascript
// eas.json con configuración de submit
{
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@email.com",
        "ascAppId": "your-app-store-connect-app-id",
        "appleTeamId": "your-apple-team-id"
      },
      "android": {
        "track": "production",
        "releaseStatus": "completed"
      }
    },
    "preview": {
      "ios": {
        "appleId": "your-apple-id@email.com",
        "ascAppId": "your-app-store-connect-app-id",
        "appleTeamId": "your-apple-team-id"
      },
      "android": {
        "track": "internal"
      }
    }
  }
}
```

#### **Submit Automatizado con CI/CD**
```yaml
# .github/workflows/eas-submit.yml
name: EAS Submit
on:
  workflow_run:
    workflows: ["EAS Build"]
    types:
      - completed

jobs:
  submit:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Setup EAS
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Submit to stores
        run: |
          if [ "${{ github.ref }}" = "refs/heads/main" ]; then
            eas submit --platform all --profile production --non-interactive
          else
            eas submit --platform all --profile preview --non-interactive
          fi
```

### 4. EAS Update - Gestión de Actualizaciones

#### **Configuración de Updates**
```javascript
// app.json
{
  "expo": {
    "updates": {
      "enabled": true,
      "fallbackToCacheTimeout": 0,
      "url": "https://u.expo.dev/your-project-id"
    },
    "runtimeVersion": {
      "policy": "sdkVersion"
    },
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

#### **Publicación de Updates**
```bash
# Publicar update para canal de desarrollo
eas update --branch development --message "Fix de bug crítico"

# Publicar update para canal de producción
eas update --branch production --message "Nueva funcionalidad"

# Publicar update con mensaje de rollback
eas update --branch production --message "Rollback a versión estable"
```

#### **Gestión de Canales de Update**
```bash
# Crear nuevo canal
eas channel:create production

# Ver canales existentes
eas channel:list

# Ver updates en un canal
eas update:list --branch production

# Rollback a update específico
eas update:rollback --branch production --target 12345678-1234-1234-1234-123456789012
```

### 5. Publicación en Tiendas

#### **App Store Connect (iOS)**
```bash
# Configurar credenciales de iOS
eas credentials

# Build para App Store
eas build --platform ios --profile production

# Submit a App Store
eas submit --platform ios --profile production
```

#### **Google Play Console (Android)**
```bash
# Configurar credenciales de Android
eas credentials

# Build para Google Play
eas build --platform android --profile production

# Submit a Google Play
eas submit --platform android --profile production
```

#### **Configuración de Metadatos**
```javascript
// app.json con metadatos para tiendas
{
  "expo": {
    "name": "Mi Aplicación",
    "slug": "mi-aplicacion",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.tuempresa.miapp",
      "buildNumber": "1",
      "infoPlist": {
        "NSCameraUsageDescription": "Esta app necesita acceso a la cámara para tomar fotos",
        "NSLocationWhenInUseUsageDescription": "Esta app necesita acceso a la ubicación para mostrar contenido relevante"
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.tuempresa.miapp",
      "versionCode": 1,
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION"
      ]
    }
  }
}
```

### 6. Monitoreo y Analytics

#### **EAS Analytics**
```javascript
// Configuración de analytics
import * as Analytics from 'expo-analytics';

// Eventos personalizados
Analytics.logEvent('app_opened', {
  timestamp: new Date().toISOString(),
  user_id: 'user123',
  app_version: '1.0.0'
});

// Eventos de conversión
Analytics.logEvent('purchase_completed', {
  value: 29.99,
  currency: 'USD',
  product_id: 'premium_subscription'
});
```

#### **Configuración de Crashlytics**
```javascript
// app.json
{
  "expo": {
    "plugins": [
      [
        "expo-build-properties",
        {
          "ios": {
            "useFrameworks": "static"
          }
        }
      ]
    ]
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración Completa de EAS
Configura EAS para tu proyecto:

**Requisitos:**
- Configurar EAS CLI y credenciales
- Crear perfiles de build para desarrollo, preview y producción
- Configurar builds automatizados con GitHub Actions
- Implementar submit automático a tiendas

**Implementación:**
```bash
# 1. Instalar y configurar EAS
npm install -g @expo/eas-cli
eas login
eas build:configure

# 2. Configurar credenciales
eas credentials

# 3. Crear workflows de GitHub Actions
# Crear .github/workflows/eas-build.yml y eas-submit.yml
```

### Ejercicio 2: Pipeline de Build y Deploy
Implementa un pipeline completo de CI/CD:

**Requisitos:**
- Build automático en push a main/develop
- Testing automático antes del build
- Submit automático a tiendas en main
- Notificaciones de estado del build

**Implementación:**
```yaml
# .github/workflows/complete-pipeline.yml
name: Complete Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - run: eas build --platform all --profile preview

  submit:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - run: eas submit --platform all --profile production
```

### Ejercicio 3: Sistema de Updates OTA
Implementa un sistema completo de actualizaciones:

**Requisitos:**
- Configurar canales de update (development, staging, production)
- Implementar verificación automática de updates
- Crear interfaz para gestión manual de updates
- Implementar rollback automático en caso de error

**Implementación:**
```javascript
// services/updateManager.js
import * as Updates from 'expo-updates';
import { Alert } from 'react-native';

export class UpdateManager {
  static async checkForUpdates() {
    try {
      const update = await Updates.checkForUpdateAsync();
      return update;
    } catch (error) {
      console.error('Error checking for updates:', error);
      return null;
    }
  }

  static async applyUpdate() {
    try {
      const result = await Updates.fetchUpdateAsync();
      if (result.isNew) {
        Alert.alert(
          'Update Available',
          'A new version is available. The app will restart to apply changes.',
          [
            {
              text: 'Restart Now',
              onPress: () => Updates.reloadAsync(),
            },
            {
              text: 'Later',
              style: 'cancel',
            },
          ]
        );
      }
      return result;
    } catch (error) {
      console.error('Error applying update:', error);
      throw error;
    }
  }

  static async rollbackToPreviousVersion() {
    try {
      // Implementar lógica de rollback
      console.log('Rolling back to previous version...');
      await Updates.reloadAsync();
    } catch (error) {
      console.error('Error rolling back:', error);
      throw error;
    }
  }
}
```

---

## 🔍 Puntos Clave

1. **EAS** simplifica todo el proceso de build, testing y distribución
2. **EAS Build** genera builds nativos en la nube sin configuración local
3. **EAS Submit** automatiza el envío a las tiendas de aplicaciones
4. **EAS Update** gestiona actualizaciones OTA de forma eficiente
5. **GitHub Actions** permite automatizar todo el pipeline de CI/CD
6. **Los perfiles de build** permiten configuraciones específicas por ambiente
7. **Las credenciales** se gestionan de forma segura en la nube

---

## 📖 Recursos Adicionales

- [EAS Documentation](https://docs.expo.dev/eas/)
- [EAS Build Documentation](https://docs.expo.dev/build/introduction/)
- [EAS Submit Documentation](https://docs.expo.dev/submit/introduction/)
- [EAS Update Documentation](https://docs.expo.dev/eas-update/)

---

## 🎉 ¡Clase Completada!

Has completado exitosamente la **Clase 5: Publicación y Distribución** 🚀

**Lo que has aprendido:**
- ✅ Configuración completa de EAS
- ✅ Generación de builds nativos en la nube
- ✅ Submit automático a tiendas de aplicaciones
- ✅ Gestión de actualizaciones OTA
- ✅ Automatización completa del pipeline

**Próximos pasos:**
- Implementar EAS en tu proyecto
- Configurar builds automatizados
- Publicar tu primera aplicación en las tiendas

¡Excelente trabajo! 🎯

---

## 🎉 ¡Módulo Completado!

Has completado exitosamente el **Módulo 8: Expo y Desarrollo Rápido** 🚀

**Lo que has aprendido en todo el módulo:**
- ✅ Fundamentos de Expo y configuración del entorno
- ✅ SDK de Expo y APIs nativas
- ✅ Herramientas de desarrollo y debugging
- ✅ Servicios en la nube de Expo
- ✅ Publicación y distribución de aplicaciones

**Próximos pasos:**
- Implementar todos los conocimientos en proyectos reales
- Explorar funcionalidades avanzadas de Expo
- Continuar con el siguiente módulo del curso

¡Felicidades por completar el módulo de Expo! 🎯

---

## 🔗 Navegación del Módulo

### **Clase Anterior**: [Clase 4: Servicios de Expo](clase_4_servicios_expo.md)
### **Volver al Módulo**: [Módulo 8: Expo y Desarrollo Rápido](README.md)
### **Módulo Siguiente**: [Módulo 9: Patrones de Diseño](../senior_1/README.md)
### **Volver al Inicio**: [Índice Completo](../INDICE_COMPLETO.md)
