# 🚀 Clase 3: Configuración de Builds

## 🎯 Objetivos de la Clase
- Configurar diferentes variantes de build (flavors)
- Implementar variables de entorno y configuración
- Optimizar builds para producción
- Manejar dependencias y assets de manera eficiente

---

## 📚 Contenido Teórico

### 1. Variantes de Build (Product Flavors)

#### **¿Qué son los Product Flavors?**
Los **Product Flavors** permiten crear diferentes versiones de tu aplicación desde el mismo código fuente, con configuraciones específicas para cada variante.

#### **Configuración Básica de Flavors**
```gradle
// android/app/build.gradle
android {
    flavorDimensions "environment", "version"
    
    productFlavors {
        // Flavors de entorno
        development {
            dimension "environment"
            applicationIdSuffix ".dev"
            versionNameSuffix "-dev"
            buildConfigField "String", "API_BASE_URL", "\"https://dev-api.miapp.com\""
            buildConfigField "boolean", "ENABLE_LOGGING", "true"
            buildConfigField "boolean", "ENABLE_ANALYTICS", "false"
        }
        
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            versionNameSuffix "-staging"
            buildConfigField "String", "API_BASE_URL", "\"https://staging-api.miapp.com\""
            buildConfigField "boolean", "ENABLE_LOGGING", "true"
            buildConfigField "boolean", "ENABLE_ANALYTICS", "true"
        }
        
        production {
            dimension "environment"
            buildConfigField "String", "API_BASE_URL", "\"https://api.miapp.com\""
            buildConfigField "boolean", "ENABLE_LOGGING", "false"
            buildConfigField "boolean", "ENABLE_ANALYTICS", "true"
        }
        
        // Flavors de versión
        free {
            dimension "version"
            applicationIdSuffix ".free"
            buildConfigField "boolean", "PREMIUM_FEATURES", "false"
        }
        
        premium {
            dimension "version"
            applicationIdSuffix ".premium"
            buildConfigField "boolean", "PREMIUM_FEATURES", "true"
        }
    }
}
```

#### **Build Types Combinados con Flavors**
```gradle
buildTypes {
    debug {
        debuggable true
        minifyEnabled false
        applicationIdSuffix ".debug"
    }
    
    release {
        debuggable false
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.release
    }
    
    // Build type personalizado
    beta {
        debuggable false
        minifyEnabled true
        applicationIdSuffix ".beta"
        versionNameSuffix "-beta"
        signingConfig signingConfigs.beta
    }
}
```

### 2. Variables de Entorno y Configuración

#### **Archivo de Configuración**
```javascript
// config/environment.js
const ENV = {
  development: {
    API_BASE_URL: 'https://dev-api.miapp.com',
    ENABLE_LOGGING: true,
    ENABLE_ANALYTICS: false,
    SENTRY_DSN: 'https://dev-sentry.miapp.com',
    GOOGLE_MAPS_API_KEY: 'dev-google-maps-key'
  },
  staging: {
    API_BASE_URL: 'https://staging-api.miapp.com',
    ENABLE_LOGGING: true,
    ENABLE_ANALYTICS: true,
    SENTRY_DSN: 'https://staging-sentry.miapp.com',
    GOOGLE_MAPS_API_KEY: 'staging-google-maps-key'
  },
  production: {
    API_BASE_URL: 'https://api.miapp.com',
    ENABLE_LOGGING: false,
    ENABLE_ANALYTICS: true,
    SENTRY_DSN: 'https://sentry.miapp.com',
    GOOGLE_MAPS_API_KEY: 'prod-google-maps-key'
  }
};

export default ENV[__DEV__ ? 'development' : 'production'];
```

#### **Configuración por Build Type**
```gradle
buildTypes {
    debug {
        buildConfigField "String", "BUILD_TYPE", "\"debug\""
        buildConfigField "int", "TIMEOUT_SECONDS", "30"
    }
    
    release {
        buildConfigField "String", "BUILD_TYPE", "\"release\""
        buildConfigField "int", "TIMEOUT_SECONDS", "15"
    }
    
    beta {
        buildConfigField "String", "BUILD_TYPE", "\"beta\""
        buildConfigField "int", "TIMEOUT_SECONDS", "20"
    }
}
```

### 3. Optimización de Builds para Producción

#### **Configuración de ProGuard Avanzada**
```proguard
# android/app/proguard-rules.pro

# Reglas para React Native
-keep class com.facebook.react.** { *; }
-keep class com.facebook.hermes.** { *; }
-keep class com.swmansion.reanimated.** { *; }

# Reglas para librerías específicas
-keep class com.google.android.gms.** { *; }
-keep class com.facebook.** { *; }
-keep class io.realm.** { *; }

# Reglas para tu aplicación
-keep class com.miapp.** { *; }

# Optimizaciones adicionales
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification

# Reglas para recursos
-keep class **.R$* {
    public static <fields>;
}
```

#### **Configuración de R8 (Reemplazo de ProGuard)**
```gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            
            // Configuración de R8
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    
    // Configuración de R8
    buildFeatures {
        buildConfig true
    }
}
```

### 4. Manejo de Dependencias y Assets

#### **Configuración de Dependencias por Flavor**
```gradle
dependencies {
    // Dependencias comunes
    implementation "com.facebook.react:react-native:+"
    implementation "com.google.android.material:material:1.5.0"
    
    // Dependencias específicas por flavor
    developmentImplementation "com.squareup.leakcanary:leakcanary-android:2.9.1"
    developmentImplementation "com.facebook.stetho:stetho:1.6.0"
    
    stagingImplementation "com.squareup.leakcanary:leakcanary-android:2.9.1"
    
    // Dependencias específicas por build type
    debugImplementation "com.facebook.stetho:stetho:1.6.0"
    debugImplementation "com.facebook.stetho:stetho-okhttp3:1.6.0"
}
```

#### **Configuración de Assets por Flavor**
```gradle
android {
    sourceSets {
        development {
            res.srcDirs = ['src/development/res', 'src/main/res']
            assets.srcDirs = ['src/development/assets', 'src/main/assets']
        }
        
        staging {
            res.srcDirs = ['src/staging/res', 'src/main/res']
            assets.srcDirs = ['src/staging/assets', 'src/main/assets']
        }
        
        production {
            res.srcDirs = ['src/main/res']
            assets.srcDirs = ['src/main/assets']
        }
    }
}
```

### 5. Configuración de Signing por Flavor

#### **Múltiples Keystores**
```gradle
signingConfigs {
    development {
        storeFile file('dev-keystore.jks')
        storePassword System.getenv('DEV_KEYSTORE_PASSWORD')
        keyAlias System.getenv('DEV_KEY_ALIAS')
        keyPassword System.getenv('DEV_KEY_PASSWORD')
    }
    
    staging {
        storeFile file('staging-keystore.jks')
        storePassword System.getenv('STAGING_KEYSTORE_PASSWORD')
        keyAlias System.getenv('STAGING_KEY_ALIAS')
        keyPassword System.getenv('STAGING_KEY_PASSWORD')
    }
    
    release {
        storeFile file('release-keystore.jks')
        storePassword System.getenv('RELEASE_KEYSTORE_PASSWORD')
        keyAlias System.getenv('RELEASE_KEY_ALIAS')
        keyPassword System.getenv('RELEASE_KEY_PASSWORD')
    }
}
```

#### **Asignación de Signing por Flavor**
```gradle
productFlavors {
    development {
        signingConfig signingConfigs.development
    }
    
    staging {
        signingConfig signingConfigs.staging
    }
    
    production {
        signingConfig signingConfigs.release
    }
}
```

### 6. Scripts de Build Personalizados

#### **Script de Build Completo**
```bash
#!/bin/bash
# build-all.sh

set -e

echo "🚀 Iniciando build completo de React Native..."

# Colores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Función para imprimir con color
print_status() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Verificar variables de entorno
check_env_vars() {
    local required_vars=("DEV_KEYSTORE_PASSWORD" "STAGING_KEYSTORE_PASSWORD" "RELEASE_KEYSTORE_PASSWORD")
    
    for var in "${required_vars[@]}"; do
        if [ -z "${!var}" ]; then
            print_error "Variable de entorno $var no está configurada"
            exit 1
        fi
    done
    
    print_status "Variables de entorno verificadas"
}

# Limpiar builds anteriores
clean_builds() {
    print_status "Limpiando builds anteriores..."
    cd android
    ./gradlew clean
    cd ..
}

# Build de development
build_development() {
    print_status "Generando build de development..."
    cd android
    ./gradlew assembleDevelopmentDebug
    ./gradlew bundleDevelopmentRelease
    cd ..
    print_status "Build de development completado"
}

# Build de staging
build_staging() {
    print_status "Generando build de staging..."
    cd android
    ./gradlew assembleStagingRelease
    ./gradlew bundleStagingRelease
    cd ..
    print_status "Build de staging completado"
}

# Build de production
build_production() {
    print_status "Generando build de production..."
    cd android
    ./gradlew assembleProductionRelease
    ./gradlew bundleProductionRelease
    cd ..
    print_status "Build de production completado"
}

# Función principal
main() {
    check_env_vars
    clean_builds
    
    case "${1:-all}" in
        "dev"|"development")
            build_development
            ;;
        "staging")
            build_staging
            ;;
        "prod"|"production")
            build_production
            ;;
        "all")
            build_development
            build_staging
            build_production
            ;;
        *)
            print_error "Uso: $0 [dev|staging|prod|all]"
            exit 1
            ;;
    esac
    
    print_status "🎉 Build completado exitosamente!"
}

# Ejecutar función principal
main "$@"
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración de Flavors Completa
Configura un proyecto con múltiples flavors y build types:

**Requisitos:**
- 3 flavors: development, staging, production
- 3 build types: debug, beta, release
- Configuración específica por flavor
- Assets y recursos diferenciados

**Implementación:**
```gradle
android {
    flavorDimensions "environment"
    
    productFlavors {
        development {
            dimension "environment"
            applicationIdSuffix ".dev"
            buildConfigField "String", "ENVIRONMENT", "\"development\""
        }
        
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            buildConfigField "String", "ENVIRONMENT", "\"staging\""
        }
        
        production {
            dimension "environment"
            buildConfigField "String", "ENVIRONMENT", "\"production\""
        }
    }
    
    buildTypes {
        debug {
            debuggable true
            applicationIdSuffix ".debug"
        }
        
        beta {
            debuggable false
            minifyEnabled true
            applicationIdSuffix ".beta"
        }
        
        release {
            debuggable false
            minifyEnabled true
            signingConfig signingConfigs.release
        }
    }
}
```

### Ejercicio 2: Script de Build Inteligente
Crea un script que detecte automáticamente el branch y genere el build correspondiente:

**Requisitos:**
- Detección automática del branch actual
- Build automático según el branch
- Validación de variables de entorno
- Notificaciones de estado

**Implementación:**
```bash
#!/bin/bash

# Obtener branch actual
CURRENT_BRANCH=$(git branch --show-current)

echo "🌿 Branch actual: $CURRENT_BRANCH"

# Determinar flavor según branch
case $CURRENT_BRANCH in
    "develop"|"development")
        FLAVOR="development"
        BUILD_TYPE="debug"
        ;;
    "staging"|"staging")
        FLAVOR="staging"
        BUILD_TYPE="beta"
        ;;
    "main"|"master"|"production")
        FLAVOR="production"
        BUILD_TYPE="release"
        ;;
    *)
        echo "❌ Branch no reconocido: $CURRENT_BRANCH"
        exit 1
        ;;
esac

echo "🚀 Generando build: $FLAVOR - $BUILD_TYPE"

# Ejecutar build
cd android
./gradlew "assemble${FLAVOR^}${BUILD_TYPE^}"
cd ..

echo "✅ Build completado: $FLAVOR - $BUILD_TYPE"
```

---

## 🔍 Puntos Clave

1. **Product Flavors** permiten crear múltiples versiones de la misma app
2. **Build Types** definen configuraciones de debug, beta y release
3. **Variables de entorno** deben protegerse y no committearse
4. **ProGuard/R8** optimiza y ofusca el código de producción
5. **Scripts personalizados** automatizan el proceso de build

---

## 📖 Recursos Adicionales

- [Android Product Flavors](https://developer.android.com/studio/build/build-variants)
- [Gradle Build Variants](https://developer.android.com/studio/build/build-variants)
- [ProGuard Configuration](https://www.guardsquare.com/manual/configuration/usage)
- [Android Build System](https://developer.android.com/studio/build)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Despliegue Automatizado** y cómo configurar Fastlane para automatizar el proceso de despliegue a Google Play Store y App Store Connect.
