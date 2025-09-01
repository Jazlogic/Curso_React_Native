# Clase 2: GitHub Actions para React Native 🚀

## Objetivos de la Clase
- Configurar workflows de GitHub Actions para React Native
- Implementar builds automatizados para Android e iOS
- Optimizar el proceso de CI/CD con caching y paralelización
- Crear workflows avanzados con testing y deployment

## Duración Estimada
**2 horas**

## Contenido Teórico

### 1. Configuración Básica de GitHub Actions

#### Estructura de Workflows
```yaml
# .github/workflows/react-native-ci.yml
name: React Native CI/CD

# Triggers del workflow
on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main, develop ]
  workflow_dispatch:  # Ejecución manual
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
        - development
        - staging
        - production

# Variables globales del workflow
env:
  NODE_VERSION: '18'
  JAVA_VERSION: '11'
  RUBY_VERSION: '3.0'
  ANDROID_SDK_VERSION: '33'
  XCODE_VERSION: '14.2'

# Jobs del workflow
jobs:
  # Job 1: Tests y Linting
  test:
    name: Test and Lint
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Para git history completo
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install && cd ..
          
      - name: Run ESLint
        run: npm run lint
        
      - name: Run TypeScript check
        run: npm run type-check
        
      - name: Run tests
        run: npm test -- --coverage
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
```

#### Workflow de Build para Android
```yaml
# .github/workflows/android-build.yml
  build-android:
    name: Build Android
    runs-on: ubuntu-latest
    needs: test  # Solo ejecutar si pasan los tests
    
    strategy:
      matrix:
        build-type: [debug, release]
        include:
          - build-type: debug
            gradle-task: assembleDebug
            artifact-name: android-debug
          - build-type: release
            gradle-task: assembleRelease
            artifact-name: android-release
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: ${{ env.JAVA_VERSION }}
          
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
        with:
          sdk-platform: ${{ env.ANDROID_SDK_VERSION }}
          sdk-build-tools: ${{ env.ANDROID_SDK_VERSION }}
          
      - name: Cache Gradle
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-
            
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build Android APK
        run: |
          cd android
          chmod +x gradlew
          ./gradlew clean ${{ matrix.gradle-task }}
          
      - name: Upload Android build
        uses: actions/upload-artifact@v3
        with:
          name: ${{ matrix.artifact-name }}
          path: |
            android/app/build/outputs/apk/*/
            android/app/build/outputs/bundle/*/
          retention-days: 30
          
      - name: Build size report
        run: |
          echo "📱 Android Build Size Report" >> $GITHUB_STEP_SUMMARY
          echo "Build Type: ${{ matrix.build-type }}" >> $GITHUB_STEP_SUMMARY
          echo "APK Size:" >> $GITHUB_STEP_SUMMARY
          ls -lh android/app/build/outputs/apk/*/ >> $GITHUB_STEP_SUMMARY
```

#### Workflow de Build para iOS
```yaml
# .github/workflows/ios-build.yml
  build-ios:
    name: Build iOS
    runs-on: macos-latest
    needs: test
    
    strategy:
      matrix:
        build-type: [debug, release]
        include:
          - build-type: debug
            configuration: Debug
            scheme: MyApp
          - build-type: release
            configuration: Release
            scheme: MyApp
            
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: ${{ env.RUBY_VERSION }}
          bundler-cache: true
          
      - name: Install Fastlane
        run: |
          gem install fastlane
          gem install cocoapods
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install && cd ..
          
      - name: Cache CocoaPods
        uses: actions/cache@v3
        with:
          path: ios/Pods
          key: ${{ runner.os }}-pods-${{ hashFiles('ios/Podfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-pods-
            
      - name: Setup Xcode
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: ${{ env.XCODE_VERSION }}
          
      - name: Build iOS app
        run: |
          cd ios
          fastlane build configuration:${{ matrix.configuration }} scheme:${{ matrix.scheme }}
          
      - name: Upload iOS build
        uses: actions/upload-artifact@v3
        with:
          name: ios-${{ matrix.build-type }}
          path: |
            ios/build/
            ios/MyApp.ipa
          retention-days: 30
          
      - name: Build size report
        run: |
          echo "🍎 iOS Build Size Report" >> $GITHUB_STEP_SUMMARY
          echo "Build Type: ${{ matrix.build-type }}" >> $GITHUB_STEP_SUMMARY
          echo "Configuration: ${{ matrix.configuration }}" >> $GITHUB_STEP_SUMMARY
          echo "Build Size:" >> $GITHUB_STEP_SUMMARY
          ls -lh ios/build/ >> $GITHUB_STEP_SUMMARY
```

### 2. Workflows Avanzados

#### Workflow de Testing Avanzado
```yaml
# .github/workflows/advanced-testing.yml
  advanced-testing:
    name: Advanced Testing
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit
        
      - name: Run integration tests
        run: npm run test:integration
        
      - name: Run E2E tests
        run: npm run test:e2e
        
      - name: Run performance tests
        run: npm run test:performance
        
      - name: Generate test report
        run: |
          npm run test:report
          
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: |
            test-results/
            coverage/
            performance-results/
          retention-days: 90
          
      - name: Comment PR with test results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const coverage = JSON.parse(fs.readFileSync('coverage/coverage-summary.json', 'utf8'));
            
            const totalCoverage = coverage.total.lines.pct;
            const emoji = totalCoverage >= 80 ? '🟢' : totalCoverage >= 60 ? '🟡' : '🔴';
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Test Results ${emoji}\n\n` +
                     `- **Coverage**: ${totalCoverage}%\n` +
                     `- **Unit Tests**: ✅ Passed\n` +
                     `- **Integration Tests**: ✅ Passed\n` +
                     `- **E2E Tests**: ✅ Passed\n` +
                     `- **Performance Tests**: ✅ Passed`
            });
```

#### Workflow de Deployment Automático
```yaml
# .github/workflows/auto-deploy.yml
  auto-deploy:
    name: Auto Deploy
    runs-on: ubuntu-latest
    needs: [test, build-android, build-ios]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    environment: production  # Requiere aprobación manual
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Download Android build
        uses: actions/download-artifact@v3
        with:
          name: android-release
          
      - name: Download iOS build
        uses: actions/download-artifact@v3
        with:
          name: ios-release
          
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: ${{ env.RUBY_VERSION }}
          
      - name: Install Fastlane
        run: gem install fastlane
        
      - name: Deploy to Google Play
        run: |
          cd android
          fastlane deploy_to_play_store
        env:
          GOOGLE_PLAY_JSON_KEY: ${{ secrets.GOOGLE_PLAY_JSON_KEY }}
          
      - name: Deploy to App Store
        run: |
          cd ios
          fastlane deploy_to_app_store
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref_name }}
          release_name: Release ${{ github.ref_name }}
          body: |
            ## What's Changed
            
            ### 🚀 New Features
            - Feature 1
            - Feature 2
            
            ### 🐛 Bug Fixes
            - Bug fix 1
            - Bug fix 2
            
            ### 📱 Builds
            - Android APK: [Download](link-to-android)
            - iOS IPA: [Download](link-to-ios)
            
          draft: false
          prerelease: false
          
      - name: Notify team
        run: |
          echo "🚀 Deployment completed successfully!"
          echo "Version: ${{ github.ref_name }}"
          echo "Environment: Production"
        # Aquí podrías integrar con Slack, Discord, etc.
```

### 3. Optimizaciones y Caching

#### Caching Inteligente
```yaml
# .github/workflows/optimized-cache.yml
  optimized-cache:
    name: Optimized Cache
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Cache Node modules
        uses: actions/cache@v3
        id: node-cache
        with:
          path: |
            node_modules
            */*/node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
            
      - name: Cache Android build
        uses: actions/cache@v3
        id: android-cache
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
            android/.gradle
            android/app/build
          key: ${{ runner.os }}-android-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-android-
            
      - name: Cache iOS build
        uses: actions/cache@v3
        id: ios-cache
        with:
          path: |
            ios/Pods
            ios/build
            ~/Library/Developer/Xcode/DerivedData
          key: ${{ runner.os }}-ios-${{ hashFiles('ios/Podfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-ios-
            
      - name: Cache Yarn
        uses: actions/cache@v3
        id: yarn-cache
        with:
          path: |
            ~/.cache/yarn
            node_modules
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-
```

#### Paralelización de Jobs
```yaml
# .github/workflows/parallel-jobs.yml
  parallel-jobs:
    name: Parallel Jobs
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        platform: [android, ios]
        build-type: [debug, release]
        exclude:
          - platform: ios
            build-type: debug  # Excluir debug para iOS en CI
            
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup ${{ matrix.platform }}
        uses: ./.github/actions/setup-${{ matrix.platform }}
        with:
          build-type: ${{ matrix.build-type }}
          
      - name: Build ${{ matrix.platform }}
        run: |
          echo "Building ${{ matrix.platform }} (${{ matrix.build-type }})"
          npm run build:${{ matrix.platform }}:${{ matrix.build-type }}
          
      - name: Upload ${{ matrix.platform }} build
        uses: actions/upload-artifact@v3
        with:
          name: ${{ matrix.platform }}-${{ matrix.build-type }}
          path: build/${{ matrix.platform }}/
```

### 4. Monitoreo y Notificaciones

#### Workflow de Monitoreo
```yaml
# .github/workflows/monitoring.yml
  monitoring:
    name: Monitoring
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run security audit
        run: npm audit --audit-level=moderate
        
      - name: Check bundle size
        run: npm run analyze-bundle
        
      - name: Performance audit
        run: npm run lighthouse
        
      - name: Dependency check
        run: npm run check-dependencies
        
      - name: Send notifications
        if: always()
        run: |
          if [ "${{ job.status }}" == "success" ]; then
            echo "✅ All checks passed"
            # Notificar éxito
          else
            echo "❌ Some checks failed"
            # Notificar fallo
          fi
```

## Ejercicios Prácticos

### Ejercicio 1: Workflow Completo de CI/CD
Crea un workflow completo que incluya testing, building y deployment:

**Solución:**
```yaml
# .github/workflows/complete-cicd.yml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  release:
    types: [published]

env:
  NODE_VERSION: '18'
  JAVA_VERSION: '11'
  RUBY_VERSION: '3.0'

jobs:
  # 1. Quality Checks
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run ESLint
        run: npm run lint
        
      - name: Run Prettier check
        run: npm run format:check
        
      - name: Run TypeScript check
        run: npm run type-check
        
      - name: Run security audit
        run: npm audit --audit-level=moderate
        
      - name: Check bundle size
        run: npm run analyze-bundle

  # 2. Testing
  test:
    name: Testing
    runs-on: ubuntu-latest
    needs: quality
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit -- --coverage
        
      - name: Run integration tests
        run: npm run test:integration
        
      - name: Run E2E tests
        run: npm run test:e2e
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  # 3. Build Android
  build-android:
    name: Build Android
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: ${{ env.JAVA_VERSION }}
          
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
        with:
          sdk-platform: '33'
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build Android
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleRelease
          
      - name: Upload Android build
        uses: actions/upload-artifact@v3
        with:
          name: android-release
          path: android/app/build/outputs/

  # 4. Build iOS
  build-ios:
    name: Build iOS
    runs-on: macos-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: ${{ env.RUBY_VERSION }}
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install && cd ..
          
      - name: Build iOS
        run: |
          cd ios
          xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -destination generic/platform=iOS -archivePath MyApp.xcarchive archive
          
      - name: Upload iOS build
        uses: actions/upload-artifact@v3
        with:
          name: ios-release
          path: ios/

  # 5. Deploy
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: [build-android, build-ios]
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download builds
        uses: actions/download-artifact@v3
        with:
          name: android-release
          path: android-build/
          
      - name: Download iOS build
        uses: actions/download-artifact@v3
        with:
          name: ios-release
          path: ios-build/
          
      - name: Create release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref_name }}
          release_name: Release ${{ github.ref_name }}
          body: |
            ## 🚀 New Release
            
            ### 📱 Builds
            - Android APK: Available in artifacts
            - iOS IPA: Available in artifacts
            
            ### 🔄 Changes
            - Automated build and deployment
            - Quality checks passed
            - All tests green
            
          draft: false
          prerelease: false
```

### Ejercicio 2: Workflow de Testing Avanzado
Implementa un workflow que ejecute diferentes tipos de tests en paralelo:

**Solución:**
```yaml
# .github/workflows/advanced-testing.yml
name: Advanced Testing

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  # Tests unitarios
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
          flags: unit-tests

  # Tests de integración
  integration-tests:
    name: Integration Tests
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
        
      - name: Start mock server
        run: npm run start:mock &
        
      - name: Wait for server
        run: sleep 10
        
      - name: Run integration tests
        run: npm run test:integration
        
      - name: Stop mock server
        run: pkill -f "start:mock"

  # Tests E2E
  e2e-tests:
    name: E2E Tests
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
        
      - name: Run E2E tests
        run: npm run test:e2e
        
      - name: Upload E2E results
        uses: actions/upload-artifact@v3
        with:
          name: e2e-results
          path: e2e-results/

  # Tests de performance
  performance-tests:
    name: Performance Tests
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
        
      - name: Run performance tests
        run: npm run test:performance
        
      - name: Generate performance report
        run: npm run generate:performance-report
        
      - name: Upload performance results
        uses: actions/upload-artifact@v3
        with:
          name: performance-results
          path: performance-results/

  # Tests de seguridad
  security-tests:
    name: Security Tests
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
        
      - name: Run security audit
        run: npm audit --audit-level=moderate
        
      - name: Run dependency check
        run: npm run check:dependencies
        
      - name: Run SAST scan
        uses: github/codeql-action/init@v2
        with:
          languages: javascript
          
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2

  # Resumen de tests
  test-summary:
    name: Test Summary
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests, e2e-tests, performance-tests, security-tests]
    if: always()
    
    steps:
      - name: Generate test summary
        run: |
          echo "## 📊 Test Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Test Type | Status |" >> $GITHUB_STEP_SUMMARY
          echo "|-----------|--------|" >> $GITHUB_STEP_SUMMARY
          echo "| Unit Tests | ${{ needs.unit-tests.result == 'success' && '✅' || '❌' }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Integration Tests | ${{ needs.integration-tests.result == 'success' && '✅' || '❌' }} |" >> $GITHUB_STEP_SUMMARY
          echo "| E2E Tests | ${{ needs.e2e-tests.result == 'success' && '✅' || '❌' }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Performance Tests | ${{ needs.performance-tests.result == 'success' && '✅' || '❌' }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Security Tests | ${{ needs.security-tests.result == 'success' && '✅' || '❌' }} |" >> $GITHUB_STEP_SUMMARY
```

### Ejercicio 3: Sistema de Notificaciones
Implementa un sistema que notifique el estado de los workflows:

**Solución:**
```yaml
# .github/workflows/notifications.yml
name: Notifications

on:
  workflow_run:
    workflows: ["Complete CI/CD Pipeline"]
    types: [completed]

jobs:
  notify:
    name: Send Notifications
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion != 'skipped' }}
    
    steps:
      - name: Check workflow status
        id: check-status
        run: |
          if [ "${{ github.event.workflow_run.conclusion }}" == "success" ]; then
            echo "status=success" >> $GITHUB_OUTPUT
            echo "emoji=✅" >> $GITHUB_OUTPUT
            echo "message=Workflow completed successfully!" >> $GITHUB_OUTPUT
          else
            echo "status=failure" >> $GITHUB_OUTPUT
            echo "emoji=❌" >> $GITHUB_OUTPUT
            echo "message=Workflow failed!" >> $GITHUB_OUTPUT
          fi
          
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ steps.check-status.outputs.status }}
          text: |
            ${{ steps.check-status.outputs.emoji }} ${{ steps.check-status.outputs.message }}
            
            **Workflow**: ${{ github.event.workflow_run.name }}
            **Branch**: ${{ github.event.workflow_run.head_branch }}
            **Commit**: ${{ github.event.workflow_run.head_sha }}
            **Duration**: ${{ github.event.workflow_run.duration }}
            
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          
      - name: Notify Discord
        uses: sarisia/actions-status-discord@v1
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK_URL }}
          status: ${{ steps.check-status.outputs.status }}
          title: "GitHub Actions - ${{ github.event.workflow_run.name }}"
          description: |
            ${{ steps.check-status.outputs.message }}
            
            **Branch**: ${{ github.event.workflow_run.head_branch }}
            **Duration**: ${{ github.event.workflow_run.duration }}
            
      - name: Send email notification
        if: steps.check-status.outputs.status == 'failure'
        run: |
          echo "Sending failure notification email..."
          # Aquí implementarías el envío de email
          # usando servicios como SendGrid, AWS SES, etc.
```

## Resumen de la Clase

En esta clase hemos cubierto:

1. **Configuración básica** de GitHub Actions para React Native
2. **Workflows avanzados** con testing, building y deployment
3. **Optimizaciones** con caching y paralelización
4. **Monitoreo y notificaciones** del estado de los workflows
5. **Ejercicios prácticos** que implementan pipelines completos

## Próximos Pasos

En la siguiente clase aprenderemos a usar **Fastlane** para automatizar el proceso de build y deployment, incluyendo configuración de lanes personalizadas.

## Navegación
- **Clase Anterior**: [Clase 1: Fundamentos de CI/CD](clase_1_fundamentos_cicd.md)
- **Clase Siguiente**: [Clase 3: Fastlane para Automatización](clase_3_fastlane_automatizacion.md)
- **Volver al Módulo**: [README del Módulo 12](README.md)
- **Volver al Índice**: [Índice Completo](../../INDICE_COMPLETO.md)
