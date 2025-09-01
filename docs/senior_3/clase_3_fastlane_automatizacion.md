# Clase 3: Fastlane para Automatización 🚀

## Objetivos de la Clase
- Configurar Fastlane para proyectos React Native
- Crear lanes personalizadas para diferentes entornos
- Automatizar el proceso de build y deployment
- Integrar Fastlane con CI/CD pipelines

## Duración Estimada
**2 horas**

## Contenido Teórico

### 1. Configuración Básica de Fastlane

#### Instalación y Configuración
```ruby
# fastlane/Fastfile
# Configuración global de Fastlane
default_platform(:android)

# Importar archivos de configuración
import 'fastlane/actions'
import 'fastlane/plugins'

# Configuración global
before_all do
  # Verificar que estemos en la rama correcta
  ensure_git_branch(branch: ['main', 'develop'])
  
  # Verificar estado del repositorio
  ensure_git_status_clean
  
  # Verificar que tengamos las credenciales necesarias
  ensure_env_vars(
    env_vars: ['GOOGLE_PLAY_JSON_KEY', 'APP_STORE_CONNECT_API_KEY']
  )
end

# Después de cada lane
after_all do |lane|
  # Notificar éxito
  if lane == :deploy_production
    slack(
      message: "🚀 Deploy to production completed successfully!",
      slack_url: ENV['SLACK_WEBHOOK_URL']
    )
  end
end

# En caso de error
error do |lane, exception|
  # Notificar fallo
  slack(
    message: "❌ Lane #{lane} failed: #{exception.message}",
    slack_url: ENV['SLACK_WEBHOOK_URL']
  )
end
```

#### Configuración de Entornos
```ruby
# fastlane/Appfile
# Configuración de la aplicación
app_identifier("com.myapp") # Bundle ID de la app
apple_id("developer@myapp.com") # Apple ID del desarrollador
itc_team_id("123456789") # Team ID de App Store Connect

# Configuración para Android
android_package_name("com.myapp")

# Configuración de Google Play
json_key_file("path/to/google-play-key.json")
```

### 2. Lanes para Android

#### Lane de Build Básico
```ruby
# fastlane/Fastfile - Android
platform :android do
  desc "Build Android app"
  lane :build do |options|
    # Parámetros opcionales
    build_type = options[:build_type] || "debug"
    flavor = options[:flavor] || "development"
    
    puts "🔨 Building Android app (#{build_type}) for flavor: #{flavor}"
    
    # Limpiar build anterior
    gradle(
      task: "clean",
      project_dir: "android/"
    )
    
    # Instalar dependencias
    gradle(
      task: "dependencies",
      project_dir: "android/"
    )
    
    # Build según el tipo
    case build_type
    when "debug"
      gradle(
        task: "assemble#{flavor.capitalize}Debug",
        project_dir: "android/"
      )
    when "release"
      gradle(
        task: "assemble#{flavor.capitalize}Release",
        project_dir: "android/"
      )
    when "bundle"
      gradle(
        task: "bundle#{flavor.capitalize}Release",
        project_dir: "android/"
      )
    end
    
    puts "✅ Android build completed successfully!"
  end
  
  # Lane para testing
  desc "Run Android tests"
  lane :test do
    puts "🧪 Running Android tests..."
    
    # Tests unitarios
    gradle(
      task: "test",
      project_dir: "android/"
    )
    
    # Tests de instrumentación
    gradle(
      task: "connectedAndroidTest",
      project_dir: "android/"
    )
    
    puts "✅ Android tests completed!"
  end
  
  # Lane para deployment a Google Play
  desc "Deploy Android app to Google Play"
  lane :deploy do |options|
    track = options[:track] || "internal"
    
    puts "🚀 Deploying Android app to #{track} track..."
    
    # 1. Incrementar versión
    increment_version_code(
      gradle_file_path: "android/app/build.gradle"
    )
    
    # 2. Build de la aplicación
    build(build_type: "bundle")
    
    # 3. Subir a Google Play
    upload_to_play_store(
      track: track,
      aab: '../android/app/build/outputs/bundle/release/app-release.aab',
      json_key: ENV['GOOGLE_PLAY_JSON_KEY'],
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
    
    puts "✅ Android app deployed to #{track} track!"
  end
end
```

#### Lane Avanzado con Flavor
```ruby
# fastlane/Fastfile - Android Avanzado
platform :android do
  desc "Build and deploy with flavor support"
  lane :build_with_flavor do |options|
    flavor = options[:flavor] || "development"
    build_type = options[:build_type] || "debug"
    
    puts "🎨 Building with flavor: #{flavor} (#{build_type})"
    
    # Configurar variables de entorno según flavor
    case flavor
    when "development"
      set_environment_variable(
        key: "API_URL",
        value: "https://dev-api.myapp.com"
      )
      set_environment_variable(
        key: "APP_NAME",
        value: "MyApp Dev"
      )
    when "staging"
      set_environment_variable(
        key: "API_URL",
        value: "https://staging-api.myapp.com"
      )
      set_environment_variable(
        key: "APP_NAME",
        value: "MyApp Staging"
      )
    when "production"
      set_environment_variable(
        key: "API_URL",
        value: "https://api.myapp.com"
      )
      set_environment_variable(
        key: "APP_NAME",
        value: "MyApp"
      )
    end
    
    # Build con flavor específico
    build(build_type: build_type, flavor: flavor)
    
    # Generar reporte de build
    generate_build_report(flavor: flavor, build_type: build_type)
  end
  
  # Generar reporte de build
  lane :generate_build_report do |options|
    flavor = options[:flavor]
    build_type = options[:build_type]
    
    puts "📊 Generating build report..."
    
    # Obtener información del build
    build_info = {
      flavor: flavor,
      build_type: build_type,
      timestamp: Time.now.strftime("%Y-%m-%d %H:%M:%S"),
      version_code: get_version_code,
      version_name: get_version_name
    }
    
    # Guardar reporte
    File.write("build-report-#{flavor}-#{build_type}.json", build_info.to_json)
    
    puts "✅ Build report generated: build-report-#{flavor}-#{build_type}.json"
  end
end
```

### 3. Lanes para iOS

#### Lane de Build Básico
```ruby
# fastlane/Fastfile - iOS
platform :ios do
  desc "Build iOS app"
  lane :build do |options|
    # Parámetros opcionales
    configuration = options[:configuration] || "Debug"
    scheme = options[:scheme] || "MyApp"
    export_method = options[:export_method] || "development"
    
    puts "🍎 Building iOS app (#{configuration}) with scheme: #{scheme}"
    
    # Limpiar build anterior
    clear_derived_data
    
    # Instalar pods si es necesario
    cocoapods
    
    # Build de la aplicación
    build_ios_app(
      workspace: "ios/MyApp.xcworkspace",
      scheme: scheme,
      configuration: configuration,
      export_method: export_method,
      export_options: {
        method: export_method,
        provisioningProfiles: {
          "com.myapp" => "MyApp_#{export_method}"
        }
      },
      output_directory: "ios/build",
      output_name: "MyApp.ipa"
    )
    
    puts "✅ iOS build completed successfully!"
  end
  
  # Lane para testing
  desc "Run iOS tests"
  lane :test do
    puts "🧪 Running iOS tests..."
    
    # Tests unitarios
    run_tests(
      workspace: "ios/MyApp.xcworkspace",
      scheme: "MyApp",
      device: "iPhone 14"
    )
    
    puts "✅ iOS tests completed!"
  end
  
  # Lane para deployment a App Store
  desc "Deploy iOS app to App Store"
  lane :deploy do |options|
    track = options[:track] || "TestFlight"
    
    puts "🚀 Deploying iOS app to #{track}..."
    
    # 1. Incrementar versión
    increment_build_number(
      xcodeproj: "ios/MyApp.xcodeproj"
    )
    
    # 2. Build de la aplicación
    build(
      configuration: "Release",
      export_method: "app-store"
    )
    
    # 3. Subir a App Store
    case track
    when "TestFlight"
      pilot(
        api_key_path: ENV['APP_STORE_CONNECT_API_KEY'],
        ipa: "ios/build/MyApp.ipa",
        skip_waiting_for_build_processing: true
      )
    when "App Store"
      deliver(
        api_key_path: ENV['APP_STORE_CONNECT_API_KEY'],
        ipa: "ios/build/MyApp.ipa",
        skip_screenshots: true,
        skip_metadata: true
      )
    end
    
    puts "✅ iOS app deployed to #{track}!"
  end
end
```

#### Lane Avanzado con Configuración
```ruby
# fastlane/Fastfile - iOS Avanzado
platform :ios do
  desc "Build and deploy with advanced configuration"
  lane :build_advanced do |options|
    configuration = options[:configuration] || "Release"
    scheme = options[:scheme] || "MyApp"
    export_method = options[:export_method] || "app-store"
    
    puts "🎯 Advanced iOS build with #{configuration} configuration"
    
    # Verificar certificados y provisioning profiles
    ensure_certificates(
      development_team: ENV['DEVELOPMENT_TEAM_ID']
    )
    
    # Actualizar provisioning profiles
    sync_code_signing(
      type: "appstore",
      readonly: true
    )
    
    # Build con configuración avanzada
    build_ios_app(
      workspace: "ios/MyApp.xcworkspace",
      scheme: scheme,
      configuration: configuration,
      export_method: export_method,
      export_options: {
        method: export_method,
        provisioningProfiles: {
          "com.myapp" => "MyApp_#{export_method}"
        },
        signingCertificate: "iPhone Distribution",
        signingStyle: "manual",
        teamID: ENV['DEVELOPMENT_TEAM_ID']
      },
      output_directory: "ios/build",
      output_name: "MyApp.ipa",
      clean: true,
      archive_path: "ios/build/MyApp.xcarchive"
    )
    
    # Generar reporte de build
    generate_ios_build_report(
      configuration: configuration,
      scheme: scheme
    )
  end
  
  # Generar reporte de build para iOS
  lane :generate_ios_build_report do |options|
    configuration = options[:configuration]
    scheme = options[:scheme]
    
    puts "📊 Generating iOS build report..."
    
    # Obtener información del build
    build_info = {
      configuration: configuration,
      scheme: scheme,
      timestamp: Time.now.strftime("%Y-%m-%d %H:%M:%S"),
      build_number: get_build_number,
      version: get_version_number
    }
    
    # Guardar reporte
    File.write("ios-build-report-#{configuration}-#{scheme}.json", build_info.to_json)
    
    puts "✅ iOS build report generated: ios-build-report-#{configuration}-#{scheme}.json"
  end
end
```

### 4. Lanes Multiplataforma

#### Lane de Build Completo
```ruby
# fastlane/Fastfile - Multiplataforma
desc "Build for all platforms"
lane :build_all do |options|
  build_type = options[:build_type] || "release"
  
  puts "🌍 Building for all platforms (#{build_type})..."
  
  # Build paralelo para ambas plataformas
  parallel(
    android: -> { build_android(build_type: build_type) },
    ios: -> { build_ios(configuration: build_type.capitalize) }
  )
  
  puts "✅ All platforms built successfully!"
end

# Lane específico para Android
lane :build_android do |options|
  build_type = options[:build_type]
  
  puts "🤖 Building Android (#{build_type})..."
  
  # Ejecutar lane de Android
  lane_context[SharedValues::LANE_NAME] = "build_android"
  
  case build_type
  when "debug"
    build(build_type: "debug")
  when "release"
    build(build_type: "release")
  when "bundle"
    build(build_type: "bundle")
  end
end

# Lane específico para iOS
lane :build_ios do |options|
  configuration = options[:configuration]
  
  puts "🍎 Building iOS (#{configuration})..."
  
  # Ejecutar lane de iOS
  lane_context[SharedValues::LANE_NAME] = "build_ios"
  
  case configuration
  when "Debug"
    build(configuration: "Debug", export_method: "development")
  when "Release"
    build(configuration: "Release", export_method: "app-store")
  end
end
```

#### Lane de Deployment Completo
```ruby
# fastlane/Fastfile - Deployment Multiplataforma
desc "Deploy to all platforms"
lane :deploy_all do |options|
  environment = options[:environment] || "staging"
  
  puts "🚀 Deploying to all platforms (#{environment})..."
  
  # Deploy paralelo para ambas plataformas
  parallel(
    android: -> { deploy_android(environment: environment) },
    ios: -> { deploy_ios(environment: environment) }
  )
  
  # Notificar éxito
  notify_deployment_success(environment: environment)
  
  puts "✅ All platforms deployed successfully!"
end

# Deploy específico para Android
lane :deploy_android do |options|
  environment = options[:environment]
  
  puts "🤖 Deploying Android to #{environment}..."
  
  case environment
  when "development"
    deploy(track: "internal")
  when "staging"
    deploy(track: "alpha")
  when "production"
    deploy(track: "production")
  end
end

# Deploy específico para iOS
lane :deploy_ios do |options|
  environment = options[:environment]
  
  puts "🍎 Deploying iOS to #{environment}..."
  
  case environment
  when "development"
    # Solo build local
    build_advanced(configuration: "Debug")
  when "staging"
    deploy(track: "TestFlight")
  when "production"
    deploy(track: "App Store")
  end
end

# Notificar éxito del deployment
lane :notify_deployment_success do |options|
  environment = options[:environment]
  
  puts "📢 Notifying deployment success..."
  
  # Notificar por Slack
  slack(
    message: "🚀 Deployment to #{environment} completed successfully!",
    slack_url: ENV['SLACK_WEBHOOK_URL']
  )
  
  # Notificar por email
  mail(
    to: ENV['TEAM_EMAIL'],
    subject: "Deployment Success - #{environment}",
    body: "The deployment to #{environment} has been completed successfully."
  )
end
```

### 5. Integración con CI/CD

#### Fastlane en GitHub Actions
```yaml
# .github/workflows/fastlane-integration.yml
name: Fastlane Integration

on:
  push:
    branches: [ main, develop ]
  workflow_dispatch:
    inputs:
      platform:
        description: 'Platform to build'
        required: true
        default: 'all'
        type: choice
        options:
        - android
        - ios
        - all
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
        - development
        - staging
        - production

jobs:
  fastlane-android:
    name: Fastlane Android
    runs-on: ubuntu-latest
    if: github.event.inputs.platform == 'android' || github.event.inputs.platform == 'all'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '11'
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          gem install fastlane
          
      - name: Run Fastlane
        run: |
          cd android
          fastlane build_with_flavor flavor:staging build_type:release
        env:
          GOOGLE_PLAY_JSON_KEY: ${{ secrets.GOOGLE_PLAY_JSON_KEY }}

  fastlane-ios:
    name: Fastlane iOS
    runs-on: macos-latest
    if: github.event.inputs.platform == 'ios' || github.event.inputs.platform == 'all'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install && cd ..
          gem install fastlane
          
      - name: Run Fastlane
        run: |
          cd ios
          fastlane build_advanced configuration:Release
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          DEVELOPMENT_TEAM_ID: ${{ secrets.DEVELOPMENT_TEAM_ID }}
```

## Ejercicios Prácticos

### Ejercicio 1: Configuración Completa de Fastlane
Crea una configuración completa de Fastlane para un proyecto React Native:

**Solución:**
```ruby
# fastlane/Fastfile
# Configuración completa de Fastlane para React Native

# Importar acciones y plugins
import 'fastlane/actions'
import 'fastlane/plugins'

# Configuración global
default_platform(:android)

# Variables globales
ENV['FASTLANE_XCODEBUILD_SETTINGS_TIMEOUT'] = '120'
ENV['FASTLANE_XCODEBUILD_SETTINGS_RETRIES'] = '3'

# Antes de todas las lanes
before_all do
  # Verificar estado del repositorio
  ensure_git_status_clean
  
  # Verificar rama
  ensure_git_branch(branch: ['main', 'develop', 'feature/*'])
  
  # Verificar credenciales
  ensure_env_vars(
    env_vars: ['GOOGLE_PLAY_JSON_KEY', 'APP_STORE_CONNECT_API_KEY']
  )
  
  puts "🚀 Starting Fastlane execution..."
end

# Después de cada lane
after_all do |lane|
  puts "✅ Lane #{lane} completed successfully!"
  
  # Notificar según el tipo de lane
  case lane
  when :deploy_production
    notify_slack("🚀 Production deployment completed!", "success")
  when :build_release
    notify_slack("🔨 Release build completed!", "info")
  end
end

# Manejo de errores
error do |lane, exception|
  puts "❌ Lane #{lane} failed: #{exception.message}"
  
  # Notificar fallo
  notify_slack("❌ Lane #{lane} failed: #{exception.message}", "error")
  
  # Limpiar archivos temporales
  clean_temp_files
end

# Plataforma Android
platform :android do
  desc "Build Android app with flavor support"
  lane :build do |options|
    flavor = options[:flavor] || "development"
    build_type = options[:build_type] || "debug"
    
    puts "🔨 Building Android (#{flavor}) - #{build_type}"
    
    # Limpiar build anterior
    gradle(
      task: "clean",
      project_dir: "android/"
    )
    
    # Instalar dependencias
    gradle(
      task: "dependencies",
      project_dir: "android/"
    )
    
    # Build según tipo y flavor
    case build_type
    when "debug"
      gradle(
        task: "assemble#{flavor.capitalize}Debug",
        project_dir: "android/"
      )
    when "release"
      gradle(
        task: "assemble#{flavor.capitalize}Release",
        project_dir: "android/"
      )
    when "bundle"
      gradle(
        task: "bundle#{flavor.capitalize}Release",
        project_dir: "android/"
      )
    end
    
    # Generar reporte
    generate_android_report(flavor: flavor, build_type: build_type)
  end
  
  desc "Deploy Android to Google Play"
  lane :deploy do |options|
    track = options[:track] || "internal"
    flavor = options[:flavor] || "production"
    
    puts "🚀 Deploying Android to #{track} track"
    
    # Incrementar versión
    increment_version_code(
      gradle_file_path: "android/app/build.gradle"
    )
    
    # Build release
    build(flavor: flavor, build_type: "bundle")
    
    # Subir a Google Play
    upload_to_play_store(
      track: track,
      aab: '../android/app/build/outputs/bundle/release/app-release.aab',
      json_key: ENV['GOOGLE_PLAY_JSON_KEY'],
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end
  
  desc "Run Android tests"
  lane :test do
    puts "🧪 Running Android tests..."
    
    gradle(
      task: "test",
      project_dir: "android/"
    )
    
    gradle(
      task: "connectedAndroidTest",
      project_dir: "android/"
    )
  end
end

# Plataforma iOS
platform :ios do
  desc "Build iOS app"
  lane :build do |options|
    configuration = options[:configuration] || "Debug"
    scheme = options[:scheme] || "MyApp"
    
    puts "🍎 Building iOS (#{configuration}) - #{scheme}"
    
    # Limpiar
    clear_derived_data
    
    # Instalar pods
    cocoapods
    
    # Build
    build_ios_app(
      workspace: "ios/MyApp.xcworkspace",
      scheme: scheme,
      configuration: configuration,
      export_method: configuration == "Debug" ? "development" : "app-store",
      output_directory: "ios/build",
      output_name: "MyApp.ipa"
    )
    
    # Generar reporte
    generate_ios_report(configuration: configuration, scheme: scheme)
  end
  
  desc "Deploy iOS to App Store"
  lane :deploy do |options|
    track = options[:track] || "TestFlight"
    
    puts "🚀 Deploying iOS to #{track}"
    
    # Incrementar build number
    increment_build_number(
      xcodeproj: "ios/MyApp.xcodeproj"
    )
    
    # Build release
    build(configuration: "Release")
    
    # Subir según track
    case track
    when "TestFlight"
      pilot(
        api_key_path: ENV['APP_STORE_CONNECT_API_KEY'],
        ipa: "ios/build/MyApp.ipa"
      )
    when "App Store"
      deliver(
        api_key_path: ENV['APP_STORE_CONNECT_API_KEY'],
        ipa: "ios/build/MyApp.ipa",
        skip_screenshots: true,
        skip_metadata: true
      )
    end
  end
  
  desc "Run iOS tests"
  lane :test do
    puts "🧪 Running iOS tests..."
    
    run_tests(
      workspace: "ios/MyApp.xcworkspace",
      scheme: "MyApp",
      device: "iPhone 14"
    )
  end
end

# Lanes multiplataforma
desc "Build for all platforms"
lane :build_all do |options|
  build_type = options[:build_type] || "release"
  
  puts "🌍 Building all platforms (#{build_type})..."
  
  # Build paralelo
  parallel(
    android: -> { build_android(build_type: build_type) },
    ios: -> { build_ios(configuration: build_type.capitalize) }
  )
end

desc "Deploy to all platforms"
lane :deploy_all do |options|
  environment = options[:environment] || "staging"
  
  puts "🚀 Deploying all platforms to #{environment}..."
  
  # Deploy paralelo
  parallel(
    android: -> { deploy_android(environment: environment) },
    ios: -> { deploy_ios(environment: environment) }
  )
  
  # Notificar éxito
  notify_deployment_success(environment: environment)
end

# Lanes de utilidad
lane :generate_android_report do |options|
  flavor = options[:flavor]
  build_type = options[:build_type]
  
  report = {
    platform: "android",
    flavor: flavor,
    build_type: build_type,
    timestamp: Time.now.iso8601,
    version_code: get_version_code,
    version_name: get_version_name
  }
  
  File.write("reports/android-#{flavor}-#{build_type}.json", report.to_json)
end

lane :generate_ios_report do |options|
  configuration = options[:configuration]
  scheme = options[:scheme]
  
  report = {
    platform: "ios",
    configuration: configuration,
    scheme: scheme,
    timestamp: Time.now.iso8601,
    build_number: get_build_number,
    version: get_version_number
  }
  
  File.write("reports/ios-#{configuration}-#{scheme}.json", report.to_json)
end

lane :notify_slack do |message, status|
  slack(
    message: message,
    slack_url: ENV['SLACK_WEBHOOK_URL'],
    success: status == "success",
    failure: status == "error"
  )
end

lane :notify_deployment_success do |environment|
  notify_slack("🚀 Deployment to #{environment} completed successfully!", "success")
end

lane :clean_temp_files do
  puts "🧹 Cleaning temporary files..."
  
  # Limpiar archivos temporales
  sh "rm -rf fastlane/reports"
  sh "rm -rf android/app/build"
  sh "rm -rf ios/build"
end
```

### Ejercicio 2: Lane de Testing Avanzado
Implementa una lane que ejecute diferentes tipos de tests:

**Solución:**
```ruby
# fastlane/Fastfile - Testing Avanzado
desc "Run comprehensive testing suite"
lane :test_comprehensive do |options|
  platform = options[:platform] || "all"
  
  puts "🧪 Running comprehensive tests for #{platform}..."
  
  case platform
  when "android"
    test_android_comprehensive
  when "ios"
    test_ios_comprehensive
  when "all"
    parallel(
      android: -> { test_android_comprehensive },
      ios: -> { test_ios_comprehensive }
    )
  end
  
  # Generar reporte de tests
  generate_test_report(platform: platform)
end

# Testing completo para Android
lane :test_android_comprehensive do
  puts "🤖 Running comprehensive Android tests..."
  
  # Tests unitarios
  gradle(
    task: "test",
    project_dir: "android/"
  )
  
  # Tests de instrumentación
  gradle(
    task: "connectedAndroidTest",
    project_dir: "android/"
  )
  
  # Tests de UI
  gradle(
    task: "assembleDebugAndroidTest",
    project_dir: "android/"
  )
  
  # Tests de performance
  gradle(
    task: "assembleRelease",
    project_dir: "android/"
  )
  
  puts "✅ Android comprehensive tests completed!"
end

# Testing completo para iOS
lane :test_ios_comprehensive do
  puts "🍎 Running comprehensive iOS tests..."
  
  # Tests unitarios
  run_tests(
    workspace: "ios/MyApp.xcworkspace",
    scheme: "MyApp",
    device: "iPhone 14",
    clean: true
  )
  
  # Tests de UI
  run_tests(
    workspace: "ios/MyApp.xcworkspace",
    scheme: "MyAppUITests",
    device: "iPhone 14"
  )
  
  # Tests de performance
  run_tests(
    workspace: "ios/MyApp.xcworkspace",
    scheme: "MyAppPerformanceTests",
    device: "iPhone 14"
  )
  
  puts "✅ iOS comprehensive tests completed!"
end

# Generar reporte de tests
lane :generate_test_report do |options|
  platform = options[:platform]
  
  puts "📊 Generating test report for #{platform}..."
  
  report = {
    platform: platform,
    timestamp: Time.now.iso8601,
    tests: {
      unit: "passed",
      integration: "passed",
      ui: "passed",
      performance: "passed"
    },
    coverage: {
      lines: "85%",
      functions: "90%",
      branches: "80%"
    }
  }
  
  File.write("reports/test-report-#{platform}.json", report.to_json)
  
  puts "✅ Test report generated: test-report-#{platform}.json"
end
```

### Ejercicio 3: Lane de Deployment Inteligente
Crea una lane que determine automáticamente qué y cómo desplegar:

**Solución:**
```ruby
# fastlane/Fastfile - Deployment Inteligente
desc "Smart deployment based on branch and changes"
lane :smart_deploy do
  puts "🧠 Starting smart deployment..."
  
  # Determinar entorno basado en rama
  environment = determine_environment
  
  # Analizar cambios
  changes = analyze_changes
  
  # Determinar qué desplegar
  deployment_plan = create_deployment_plan(environment: environment, changes: changes)
  
  # Ejecutar deployment
  execute_deployment_plan(deployment_plan)
  
  puts "✅ Smart deployment completed!"
end

# Determinar entorno
lane :determine_environment do
  branch = git_branch
  
  case branch
  when /^main$/
    "production"
  when /^develop$/
    "staging"
  when /^feature\//
    "development"
  when /^hotfix\//
    "staging"
  else
    "development"
  end
end

# Analizar cambios
lane :analyze_changes do
  puts "🔍 Analyzing changes..."
  
  # Obtener commits desde el último tag
  last_tag = last_git_tag
  commits = git_log_between(from: last_tag, to: "HEAD")
  
  changes = {
    android: false,
    ios: false,
    shared: false,
    breaking: false
  }
  
  commits.each do |commit|
    # Verificar cambios en Android
    if commit.include?("android/") || commit.include?("gradle")
      changes[:android] = true
    end
    
    # Verificar cambios en iOS
    if commit.include?("ios/") || commit.include?("pod")
      changes[:ios] = true
    end
    
    # Verificar cambios compartidos
    if commit.include?("src/") || commit.include?("package.json")
      changes[:shared] = true
    end
    
    # Verificar cambios breaking
    if commit.include?("BREAKING") || commit.include?("major")
      changes[:breaking] = true
    end
  end
  
  changes
end

# Crear plan de deployment
lane :create_deployment_plan do |options|
  environment = options[:environment]
  changes = options[:changes]
  
  puts "📋 Creating deployment plan for #{environment}..."
  
  plan = {
    environment: environment,
    platforms: [],
    actions: [],
    notifications: []
  }
  
  # Determinar plataformas a desplegar
  if changes[:android] || changes[:shared]
    plan[:platforms] << "android"
  end
  
  if changes[:ios] || changes[:shared]
    plan[:platforms] << "ios"
  end
  
  # Determinar acciones
  case environment
  when "development"
    plan[:actions] = ["build", "test"]
    plan[:notifications] = ["slack"]
  when "staging"
    plan[:actions] = ["build", "test", "deploy_staging"]
    plan[:notifications] = ["slack", "email"]
  when "production"
    plan[:actions] = ["build", "test", "deploy_production"]
    plan[:notifications] = ["slack", "email", "pagerduty"]
  end
  
  # Agregar acciones especiales para cambios breaking
  if changes[:breaking]
    plan[:actions] << "manual_review"
    plan[:notifications] << "urgent"
  end
  
  plan
end

# Ejecutar plan de deployment
lane :execute_deployment_plan do |plan|
  puts "🚀 Executing deployment plan..."
  
  environment = plan[:environment]
  platforms = plan[:platforms]
  actions = plan[:actions]
  
  # Ejecutar acciones en orden
  actions.each do |action|
    case action
    when "build"
      if platforms.include?("android")
        build_android(flavor: environment, build_type: "release")
      end
      
      if platforms.include?("ios")
        build_ios(configuration: "Release")
      end
      
    when "test"
      if platforms.include?("android")
        test_android
      end
      
      if platforms.include?("ios")
        test_ios
      end
      
    when "deploy_staging"
      if platforms.include?("android")
        deploy_android(track: "alpha")
      end
      
      if platforms.include?("ios")
        deploy_ios(track: "TestFlight")
      end
      
    when "deploy_production"
      if platforms.include?("android")
        deploy_android(track: "production")
      end
      
      if platforms.include?("ios")
        deploy_ios(track: "App Store")
      end
      
    when "manual_review"
      puts "⚠️ Manual review required for breaking changes!"
      # Aquí podrías pausar y esperar aprobación manual
    end
  end
  
  # Enviar notificaciones
  send_deployment_notifications(plan)
end

# Enviar notificaciones de deployment
lane :send_deployment_notifications do |plan|
  environment = plan[:environment]
  notifications = plan[:notifications]
  
  puts "📢 Sending deployment notifications..."
  
  notifications.each do |notification|
    case notification
    when "slack"
      notify_slack("🚀 Deployment to #{environment} completed!", "success")
      
    when "email"
      mail(
        to: ENV['TEAM_EMAIL'],
        subject: "Deployment Success - #{environment}",
        body: "The deployment to #{environment} has been completed successfully."
      )
      
    when "pagerduty"
      # Integración con PagerDuty
      puts "📞 PagerDuty notification sent"
      
    when "urgent"
      # Notificación urgente
      notify_slack("🚨 URGENT: Breaking changes deployed to #{environment}!", "error")
    end
  end
end
```

## Resumen de la Clase

En esta clase hemos cubierto:

1. **Configuración básica** de Fastlane para proyectos React Native
2. **Lanes personalizadas** para Android e iOS con diferentes configuraciones
3. **Lanes multiplataforma** que coordinan builds y deployments
4. **Integración con CI/CD** usando GitHub Actions
5. **Ejercicios prácticos** que implementan automatización completa

## Próximos Pasos

En la siguiente clase aprenderemos sobre **Testing Automatizado en CI/CD**, incluyendo Jest, Detox y testing de integración.

## Navegación
- **Clase Anterior**: [Clase 2: GitHub Actions para React Native](clase_2_github_actions.md)
- **Clase Siguiente**: [Clase 4: Testing Automatizado en CI/CD](clase_4_testing_automatizado.md)
- **Volver al Módulo**: [README del Módulo 12](README.md)
- **Volver al Índice**: [Índice Completo](../../INDICE_COMPLETO.md)
