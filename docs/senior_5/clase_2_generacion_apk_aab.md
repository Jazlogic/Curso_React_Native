# 🚀 Clase 2: Generación de APK/AAB

## 🎯 Objetivos de la Clase
- Configurar Gradle para builds de producción
- Generar APK de debug y release
- Crear AAB (Android App Bundle) para Google Play
- Implementar firma de código y gestión de keystores

---

## 📚 Contenido Teórico

### 1. Configuración de Gradle para Builds

#### **Estructura del archivo build.gradle**
```gradle
// android/app/build.gradle
apply plugin: "com.android.application"
apply plugin: "com.facebook.react"

android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        applicationId "com.miapp"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "1.0.0"
    }

    buildTypes {
        debug {
            debuggable true
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        
        release {
            debuggable false
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }

    signingConfigs {
        release {
            storeFile file('release-key.keystore')
            storePassword System.getenv('KEYSTORE_PASSWORD')
            keyAlias System.getenv('KEY_ALIAS')
            keyPassword System.getenv('KEY_PASSWORD')
        }
    }
}
```

#### **Configuración de versiones**
```gradle
// android/gradle.properties
# Project-wide Gradle settings
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# Android configuration
android.useAndroidX=true
android.enableJetifier=true

# Version configuration
VERSION_NAME=1.0.0
VERSION_CODE=1

# Build configuration
android.enableR8.fullMode=true
android.enableD8=true
```

### 2. Generación de APK de Debug

#### **Build de Debug**
```bash
# Generar APK de debug
cd android
./gradlew assembleDebug

# El APK se genera en:
# android/app/build/outputs/apk/debug/app-debug.apk
```

#### **Configuración de Debug**
```gradle
buildTypes {
    debug {
        debuggable true
        minifyEnabled false
        applicationIdSuffix ".debug"
        versionNameSuffix "-debug"
        
        // Configuración específica para debug
        buildConfigField "String", "API_BASE_URL", "\"https://dev-api.miapp.com\""
        buildConfigField "boolean", "ENABLE_LOGGING", "true"
    }
}
```

### 3. Generación de APK de Release

#### **Build de Release**
```bash
# Generar APK de release
cd android
./gradlew assembleRelease

# El APK se genera en:
# android/app/build/outputs/apk/release/app-release-unsigned.apk
```

#### **Configuración de Release**
```gradle
buildTypes {
    release {
        debuggable false
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.release
        
        // Configuración específica para release
        buildConfigField "String", "API_BASE_URL", "\"https://api.miapp.com\""
        buildConfigField "boolean", "ENABLE_LOGGING", "false"
    }
}
```

### 4. Creación de AAB (Android App Bundle)

#### **¿Qué es AAB?**
**AAB** (Android App Bundle) es el formato de publicación recomendado por Google para Google Play Store. Permite:
- **Tamaños de descarga optimizados** por dispositivo
- **Múltiples APKs** generados automáticamente
- **Mejor gestión** de recursos y idiomas

#### **Generación de AAB**
```bash
# Generar AAB
cd android
./gradlew bundleRelease

# El AAB se genera en:
# android/app/build/outputs/bundle/release/app-release.aab
```

#### **Configuración para AAB**
```gradle
android {
    bundle {
        language {
            enableSplit = true
        }
        density {
            enableSplit = true
        }
        abi {
            enableSplit = true
        }
    }
}
```

### 5. Firma de Código y Keystores

#### **Generación de Keystore**
```bash
# Generar keystore para release
keytool -genkey -v -keystore release-key.keystore -alias miapp-key -keyalg RSA -keysize 2048 -validity 10000
```

#### **Configuración de Firma**
```gradle
signingConfigs {
    release {
        storeFile file('release-key.keystore')
        storePassword System.getenv('KEYSTORE_PASSWORD')
        keyAlias System.getenv('KEY_ALIAS')
        keyPassword System.getenv('KEY_PASSWORD')
    }
}
```

#### **Variables de Entorno**
```bash
# .env o configuración del sistema
export KEYSTORE_PASSWORD="tu_password_seguro"
export KEY_ALIAS="miapp-key"
export KEY_PASSWORD="tu_key_password"
```

### 6. ProGuard y Optimización

#### **Configuración de ProGuard**
```proguard
# android/app/proguard-rules.pro

# Reglas para React Native
-keep class com.facebook.react.** { *; }
-keep class com.facebook.hermes.** { *; }
-keep class com.swmansion.reanimated.** { *; }

# Reglas para librerías específicas
-keep class com.google.android.gms.** { *; }
-keep class com.facebook.** { *; }

# Reglas para tu aplicación
-keep class com.miapp.** { *; }
```

#### **Optimizaciones de Build**
```gradle
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    dexOptions {
        javaMaxHeapSize "4g"
        preDexLibraries = false
    }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración Completa de Build
Configura un proyecto React Native con builds de debug y release:

**Requisitos:**
- Configuración de Gradle para debug y release
- Generación de APK y AAB
- Configuración de ProGuard
- Variables de entorno para keystores

**Implementación:**
```gradle
android {
    compileSdkVersion 33
    buildToolsVersion "33.0.0"
    
    defaultConfig {
        applicationId "com.tuapp"
        minSdkVersion 21
        targetSdkVersion 33
        versionCode 1
        versionName "1.0.0"
    }
    
    buildTypes {
        debug {
            debuggable true
            applicationIdSuffix ".debug"
        }
        
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
    
    signingConfigs {
        release {
            storeFile file('release-key.keystore')
            storePassword System.getenv('KEYSTORE_PASSWORD')
            keyAlias System.getenv('KEY_ALIAS')
            keyPassword System.getenv('KEY_PASSWORD')
        }
    }
}
```

### Ejercicio 2: Script de Build Automatizado
Crea un script que automatice el proceso de build:

**Requisitos:**
- Script para generar APK de debug
- Script para generar AAB de release
- Validación de keystores
- Upload automático a Google Play

**Implementación:**
```bash
#!/bin/bash
# build.sh

set -e

echo "🚀 Iniciando build de React Native..."

# Verificar variables de entorno
if [ -z "$KEYSTORE_PASSWORD" ] || [ -z "$KEY_ALIAS" ] || [ -z "$KEY_PASSWORD" ]; then
    echo "❌ Error: Variables de entorno de keystore no configuradas"
    exit 1
fi

# Instalar dependencias
echo "📦 Instalando dependencias..."
npm ci

# Build de debug
echo "🔧 Generando APK de debug..."
cd android
./gradlew assembleDebug
echo "✅ APK de debug generado en app/build/outputs/apk/debug/"

# Build de release
echo "🚀 Generando AAB de release..."
./gradlew bundleRelease
echo "✅ AAB de release generado en app/build/outputs/bundle/release/"

echo "🎉 Build completado exitosamente!"
```

---

## 🔍 Puntos Clave

1. **Gradle** es el sistema de build estándar para Android
2. **AAB** es el formato recomendado para Google Play Store
3. **Firma de código** es obligatoria para releases de producción
4. **ProGuard** optimiza y ofusca el código de release
5. **Variables de entorno** protegen información sensible de keystores

---

## 📖 Recursos Adicionales

- [Android App Bundle](https://developer.android.com/guide/app-bundle)
- [Android Code Signing](https://developer.android.com/studio/publish/app-signing)
- [ProGuard for Android](https://developer.android.com/studio/build/shrink-code)
- [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Configuración de Builds** y cómo configurar diferentes variantes de build, variables de entorno y optimización para producción.
