# 🚀 Clase 4: Despliegue Automatizado

## 🎯 Objetivos de la Clase
- Configurar Fastlane para automatización de despliegue
- Automatizar despliegue a Google Play Store
- Integrar con App Store Connect para iOS
- Implementar rollbacks y versiones de emergencia

---

## 📚 Contenido Teórico

### 1. Introducción a Fastlane

#### **¿Qué es Fastlane?**
**Fastlane** es una herramienta de automatización para iOS y Android que simplifica el proceso de build, testing y despliegue de aplicaciones móviles.

#### **Ventajas de Fastlane:**
- **Automatización completa** del proceso de despliegue
- **Integración nativa** con Xcode y Android Studio
- **Soporte multiplataforma** (iOS y Android)
- **Extensible** con plugins y scripts personalizados
- **Integración CI/CD** con GitHub Actions, GitLab CI, etc.

#### **Instalación de Fastlane**
```bash
# Instalación con RubyGems
gem install fastlane

# O con Bundler (recomendado)
bundle init
bundle add fastlane
bundle exec fastlane --version
```

### 2. Configuración de Fastlane para Android

#### **Inicialización del Proyecto**
```bash
# En el directorio raíz del proyecto
fastlane init
```

#### **Estructura de Archivos Fastlane**
```
fastlane/
├── Appfile          # Configuración de la app
├── Fastfile         # Definición de lanes
├── Pluginfile       # Plugins de Fastlane
└── actions/         # Acciones personalizadas
```

#### **Appfile - Configuración de la Aplicación**
```ruby
# fastlane/Appfile
json_key_file("path/to/api-key.json") # Path to the json secret file - Follow https://docs.fastlane.tools/actions/supply/#setup to get one
package_name("com.miapp") # e.g. com.yourcompany.yourapp
```

#### **Fastfile - Definición de Lanes**
```ruby
# fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Build and deploy to Google Play Store"
  lane :deploy do
    # Build de la aplicación
    gradle(
      task: "clean bundleRelease",
      project_dir: "android/"
    )
    
    # Subir a Google Play Store
    upload_to_play_store(
      track: 'production',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab'
    )
  end
  
  desc "Build and deploy to internal testing"
  lane :deploy_internal do
    gradle(
      task: "clean bundleRelease",
      project_dir: "android/"
    )
    
    upload_to_play_store(
      track: 'internal',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab'
    )
  end
  
  desc "Build and deploy to beta testing"
  lane :deploy_beta do
    gradle(
      task: "clean bundleRelease",
      project_dir: "android/"
    )
    
    upload_to_play_store(
      track: 'beta',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab'
    )
  end
end
```

### 3. Configuración de Google Play Console

#### **Configuración de API Key**
1. **Ir a Google Play Console**
2. **Configuración > API access**
3. **Crear nuevo proyecto de servicio**
4. **Descargar archivo JSON de credenciales**

#### **Configuración de Permisos**
```json
{
  "type": "service_account",
  "project_id": "tu-proyecto",
  "private_key_id": "key-id",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "tu-app@tu-proyecto.iam.gserviceaccount.com",
  "client_id": "client-id",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/tu-app%40tu-proyecto.iam.gserviceaccount.com"
}
```

#### **Configuración en Fastfile**
```ruby
lane :deploy do
  # Configurar credenciales
  gradle(
    task: "clean bundleRelease",
    project_dir: "android/"
  )
  
  # Subir a Google Play Store
  upload_to_play_store(
    track: 'production',
    aab: '../android/app/build/outputs/bundle/release/app-release.aab',
    json_key: 'path/to/api-key.json',
    release_status: 'completed'
  )
end
```

### 4. Configuración de Fastlane para iOS

#### **Fastfile para iOS**
```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Build and deploy to App Store"
  lane :deploy do
    # Build de la aplicación
    build_ios_app(
      scheme: "MyApp",
      export_method: "app-store",
      configuration: "Release"
    )
    
    # Subir a App Store Connect
    upload_to_app_store(
      skip_metadata: true,
      skip_screenshots: true,
      skip_binary_upload: false
    )
  end
  
  desc "Build and deploy to TestFlight"
  lane :deploy_testflight do
    build_ios_app(
      scheme: "MyApp",
      export_method: "app-store",
      configuration: "Release"
    )
    
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
  end
  
  desc "Build and deploy to development"
  lane :deploy_development do
    build_ios_app(
      scheme: "MyApp",
      export_method: "development",
      configuration: "Debug"
    )
    
    # Instalar en simulador o dispositivo
    install_on_device
  end
end
```

### 5. Lanes Avanzadas y Automatización

#### **Lane de Build Completo**
```ruby
desc "Build completo de la aplicación"
lane :build do
  # Verificar estado del repositorio
  ensure_git_status_clean
  
  # Incrementar versión
  increment_version_code(
    xcodeproj: "ios/MyApp.xcodeproj"
  )
  
  # Build de Android
  gradle(
    task: "clean bundleRelease",
    project_dir: "android/"
  )
  
  # Build de iOS
  build_ios_app(
    scheme: "MyApp",
    export_method: "app-store",
    configuration: "Release"
  )
  
  # Notificar éxito
  slack(
    message: "Build completado exitosamente! 🎉",
    success: true
  )
end
```

#### **Lane de Deploy con Validaciones**
```ruby
desc "Deploy con validaciones"
lane :deploy_safe do
  # Verificar tests
  run_tests(
    scheme: "MyApp",
    device: "iPhone 14"
  )
  
  # Verificar linting
  sh("cd android && ./gradlew lint")
  
  # Build de la aplicación
  build
  
  # Deploy a staging
  deploy_staging
  
  # Esperar confirmación manual
  prompt(
    text: "¿Proceder con deploy a producción?",
    boolean: true
  )
  
  # Deploy a producción
  deploy_production
end
```

### 6. Rollbacks y Versiones de Emergencia

#### **Lane de Rollback**
```ruby
desc "Rollback a versión anterior"
lane :rollback do
  # Obtener versión anterior
  previous_version = get_version_number(
    xcodeproj: "ios/MyApp.xcodeproj"
  )
  
  # Revertir a commit anterior
  sh("git revert HEAD --no-edit")
  
  # Build de la aplicación
  build
  
  # Deploy de emergencia
  deploy_emergency
  
  # Notificar rollback
  slack(
    message: "Rollback completado a versión #{previous_version}",
    success: false
  )
end
```

#### **Lane de Deploy de Emergencia**
```ruby
desc "Deploy de emergencia"
lane :deploy_emergency do
  # Build rápido sin tests
  build_ios_app(
    scheme: "MyApp",
    export_method: "app-store",
    configuration: "Release",
    skip_profile_detection: true
  )
  
  # Deploy inmediato
  upload_to_app_store(
    skip_metadata: true,
    skip_screenshots: true,
    skip_binary_upload: false,
    force: true
  )
  
  # Notificar equipo
  slack(
    message: "🚨 Deploy de emergencia completado",
    success: false
  )
end
```

### 7. Integración con CI/CD

#### **GitHub Actions con Fastlane**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Stores

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          bundler-cache: true
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '11'
      
      - name: Install dependencies
        run: |
          npm ci
          cd android && ./gradlew clean
      
      - name: Deploy to Google Play
        run: bundle exec fastlane deploy
        env:
          SUPPLY_JSON_KEY: ${{ secrets.GOOGLE_PLAY_API_KEY }}
```

#### **GitLab CI con Fastlane**
```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

deploy-android:
  stage: deploy
  image: ruby:3.0
  before_script:
    - gem install bundler
    - bundle install
  script:
    - bundle exec fastlane deploy
  only:
    - tags
  environment:
    name: production
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración Básica de Fastlane
Configura Fastlane para tu proyecto React Native:

**Requisitos:**
- Configuración para Android e iOS
- Lanes para build, test y deploy
- Integración con Google Play Store
- Configuración de credenciales

**Implementación:**
```ruby
# fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Build de la aplicación"
  lane :build do
    gradle(
      task: "clean bundleRelease",
      project_dir: "android/"
    )
  end
  
  desc "Deploy a Google Play Store"
  lane :deploy do
    build
    upload_to_play_store(
      track: 'production',
      aab: '../android/app/build/outputs/bundle/release/app-release.aab'
    )
  end
end

platform :ios do
  desc "Build de la aplicación iOS"
  lane :build do
    build_ios_app(
      scheme: "MyApp",
      export_method: "app-store"
    )
  end
  
  desc "Deploy a App Store"
  lane :deploy do
    build
    upload_to_app_store
  end
end
```

### Ejercicio 2: Pipeline de Deploy Automatizado
Crea un pipeline completo de deploy:

**Requisitos:**
- Validaciones automáticas antes del deploy
- Deploy a múltiples tracks (internal, beta, production)
- Notificaciones de estado
- Rollback automático en caso de fallo

**Implementación:**
```ruby
desc "Pipeline completo de deploy"
lane :deploy_pipeline do
  # Validaciones
  ensure_git_status_clean
  run_tests
  
  # Build
  build
  
  # Deploy a internal testing
  deploy_internal
  
  # Esperar feedback
  prompt(text: "¿Proceder con deploy a beta?")
  
  # Deploy a beta
  deploy_beta
  
  # Esperar confirmación final
  prompt(text: "¿Proceder con deploy a producción?")
  
  # Deploy a producción
  deploy_production
  
  # Notificar éxito
  notify_success
rescue => ex
  # Rollback automático
  rollback
  notify_failure(ex)
end
```

---

## 🔍 Puntos Clave

1. **Fastlane** automatiza completamente el proceso de deploy
2. **Google Play Console API** requiere configuración de credenciales
3. **Lanes** organizan y automatizan diferentes flujos de trabajo
4. **Rollbacks** son esenciales para manejar problemas en producción
5. **Integración CI/CD** permite deploy automático en cada release

---

## 📖 Recursos Adicionales

- [Fastlane Documentation](https://docs.fastlane.tools/)
- [Google Play Console API](https://developers.google.com/android-publisher)
- [App Store Connect API](https://developer.apple.com/app-store-connect/api/)
- [Fastlane Plugins](https://docs.fastlane.tools/plugins/available-plugins/)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Monitoreo y Métricas** y cómo implementar sistemas de monitoreo para builds y despliegues, incluyendo métricas de rendimiento y alertas automáticas.
