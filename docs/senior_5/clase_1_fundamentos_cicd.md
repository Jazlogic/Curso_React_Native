# 🚀 Clase 1: Fundamentos de CI/CD

## 🎯 Objetivos de la Clase
- Entender qué es CI/CD y por qué es importante para React Native
- Conocer las herramientas y plataformas de CI/CD disponibles
- Configurar pipelines automatizados básicos
- Integrar CI/CD con repositorios Git

---

## 📚 Contenido Teórico

### 1. ¿Qué es CI/CD?

**CI/CD** (Integración Continua/Despliegue Continuo) es una metodología que automatiza el proceso de desarrollo, testing y despliegue de software.

#### **CI - Integración Continua**
- **Automatización de builds** en cada commit/push
- **Testing automático** antes de la integración
- **Detección temprana** de problemas
- **Calidad del código** consistente

#### **CD - Despliegue Continuo**
- **Automatización del despliegue** a diferentes entornos
- **Rollbacks automáticos** en caso de fallos
- **Despliegue a producción** con aprobación manual
- **Monitoreo continuo** del estado de la aplicación

### 2. ¿Por qué CI/CD es importante para React Native?

#### **Desafíos Específicos de React Native:**
- **Doble plataforma**: iOS y Android requieren builds separados
- **Dependencias nativas**: CocoaPods, Gradle, etc.
- **Testing multiplataforma**: Simuladores y emuladores
- **Code signing**: Certificados y keystores para producción
- **Despliegue**: App Store y Google Play Store

#### **Beneficios del CI/CD:**
- **Consistencia**: Builds idénticos en cada entorno
- **Velocidad**: Automatización reduce tiempo de despliegue
- **Calidad**: Testing automático en cada cambio
- **Seguridad**: Proceso controlado y auditado
- **Escalabilidad**: Manejo de múltiples desarrolladores

### 3. Herramientas y Plataformas de CI/CD

#### **Plataformas en la Nube:**
- **GitHub Actions**: Integrado con GitHub, gratuito para repos públicos
- **GitLab CI**: Integrado con GitLab, muy potente
- **CircleCI**: Especializado en CI/CD, excelente para móvil
- **Travis CI**: Clásico, bueno para proyectos open source
- **Azure DevOps**: Solución completa de Microsoft

#### **Servidores Self-Hosted:**
- **Jenkins**: Open source, muy flexible
- **TeamCity**: De JetBrains, excelente para equipos grandes
- **Bamboo**: De Atlassian, integrado con Jira

#### **Herramientas de Automatización:**
- **Fastlane**: Automatización de builds y despliegue móvil
- **Gradle**: Sistema de build para Android
- **CocoaPods**: Gestor de dependencias para iOS
- **Xcode**: Builds de iOS
- **Android Studio**: Builds de Android

### 4. Arquitectura de un Pipeline CI/CD

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Source Code   │───▶│   Build Stage   │───▶│  Test Stage    │
│   (Git Push)    │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                       │
                                ▼                       ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │  Package Stage  │───▶│ Deploy Stage   │
                       │                 │    │                 │
                       └─────────────────┘    └─────────────────┘
```

#### **Etapas del Pipeline:**

1. **Source Stage**: Trigger del pipeline (push, PR, tag)
2. **Build Stage**: Compilación del código fuente
3. **Test Stage**: Ejecución de tests automatizados
4. **Package Stage**: Creación de artefactos (APK, AAB, IPA)
5. **Deploy Stage**: Despliegue a entornos de testing/producción

### 5. Configuración Básica de CI/CD

#### **GitHub Actions - Workflow Básico**
```yaml
# .github/workflows/ci-cd.yml
name: React Native CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-android:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
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
          distribution: 'zulu'
          java-version: '11'
      
      - name: Build Android APK
        run: |
          cd android
          ./gradlew assembleDebug
      
      - name: Upload APK
        uses: actions/upload-artifact@v3
        with:
          name: app-debug
          path: android/app/build/outputs/apk/debug/
```

#### **GitLab CI - Pipeline Básico**
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - package

variables:
  NODE_VERSION: "18"

build_android:
  stage: build
  image: node:18
  before_script:
    - npm ci
  script:
    - cd android
    - ./gradlew assembleDebug
  artifacts:
    paths:
      - android/app/build/outputs/apk/debug/
    expire_in: 1 week
  only:
    - main
    - develop
```

### 6. Integración con Repositorios Git

#### **Estrategias de Branching:**
- **GitFlow**: Branches para features, develop, release, main
- **GitHub Flow**: Branches para features, PR directo a main
- **Trunk-Based Development**: Desarrollo directo en main

#### **Triggers del Pipeline:**
- **Push a main**: Build y deploy automático
- **Pull Request**: Build y test automático
- **Tags**: Release builds
- **Manual**: Trigger manual para builds especiales

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración Básica de GitHub Actions
Configura un workflow básico para tu proyecto React Native:

**Requisitos:**
- Trigger en push a main y develop
- Setup de Node.js y Java
- Instalación de dependencias
- Build básico de Android

**Implementación:**
```yaml
name: React Native Basic CI

on:
  push:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '11'
      - run: cd android && ./gradlew assembleDebug
```

### Ejercicio 2: Pipeline de Testing Básico
Implementa un pipeline que incluya testing:

**Requisitos:**
- Linting del código
- Testing unitario con Jest
- Testing de TypeScript
- Reportes de cobertura

**Implementación:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm run test:coverage
      - uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

---

## 🔍 Puntos Clave

1. **CI/CD es esencial** para proyectos React Native profesionales
2. **Automatización** reduce errores humanos y acelera el desarrollo
3. **Testing automático** asegura calidad en cada cambio
4. **Plataformas en la nube** son más fáciles de configurar inicialmente
5. **Integración con Git** es fundamental para el flujo de trabajo

---

## 📖 Recursos Adicionales

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [CircleCI Documentation](https://circleci.com/docs/)
- [Fastlane Documentation](https://docs.fastlane.tools/)

---

## ➡️ Siguiente Clase
En la siguiente clase aprenderemos sobre **Generación de APK/AAB** y cómo configurar Gradle para builds de producción, incluyendo firma de código y keystores.
