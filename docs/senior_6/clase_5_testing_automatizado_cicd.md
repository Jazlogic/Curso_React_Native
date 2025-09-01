# Clase 5: Testing Automatizado y CI/CD 🔄

## Objetivos de la Clase
- Configurar pipelines de testing automatizado
- Implementar testing continuo en CI/CD
- Crear reportes y métricas de testing
- Automatizar testing en múltiples entornos

## Contenido Teórico

### 1. Pipelines de Testing Automatizado

#### Configuración de GitHub Actions
```yaml
# .github/workflows/testing-pipeline.yml
name: Testing Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  schedule:
    - cron: '0 2 * * *' # Ejecutar diariamente a las 2 AM

env:
  NODE_VERSION: '18'
  RN_VERSION: '0.72.0'

jobs:
  # Job de testing unitario
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit
        
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: unit-tests
          name: codecov-umbrella
          
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: unit-test-results
          path: coverage/
          retention-days: 30

  # Job de testing de integración
  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Start mock server
        run: npm run test:server:start
        
      - name: Run integration tests
        run: npm run test:integration
        
      - name: Stop mock server
        run: npm run test:server:stop
        
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: integration-test-results
          path: test-results/
          retention-days: 30

  # Job de testing E2E en iOS
  e2e-tests-ios:
    name: E2E Tests - iOS
    runs-on: macos-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Setup iOS Simulator
        uses: reactivecircus/ios-simulator-action@v2
        with:
          simulator: 'iPhone 14'
          ios: '16.2'
          
      - name: Build and test iOS
        run: |
          cd ios
          pod install
          cd ..
          npm run test:e2e:ios
          
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: e2e-ios-results
          path: e2e/artifacts/
          retention-days: 30

  # Job de testing E2E en Android
  e2e-tests-android:
    name: E2E Tests - Android
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
        
      - name: Setup Android Emulator
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 30
          target: google_apis
          arch: x86_64
          profile: Nexus 6
          
      - name: Build and test Android
        run: npm run test:e2e:android
        
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: e2e-android-results
          path: e2e/artifacts/
          retention-days: 30

  # Job de testing de rendimiento
  performance-tests:
    name: Performance Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run performance tests
        run: npm run test:performance
        
      - name: Generate performance report
        run: npm run test:performance:report
        
      - name: Upload performance results
        uses: actions/upload-artifact@v3
        with:
          name: performance-results
          path: performance-reports/
          retention-days: 30

  # Job de testing de seguridad
  security-tests:
    name: Security Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run security audit
        run: npm audit --audit-level moderate
        
      - name: Run dependency check
        run: npm run test:security:dependencies
        
      - name: Run SAST scan
        run: npm run test:security:sast
        
      - name: Upload security results
        uses: actions/upload-artifact@v3
        with:
          name: security-results
          path: security-reports/
          retention-days: 30

  # Job de reportes y métricas
  test-reporting:
    name: Test Reporting
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests, e2e-tests-ios, e2e-tests-android, performance-tests, security-tests]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Download all test results
        uses: actions/download-artifact@v3
        with:
          path: test-results/
          
      - name: Generate comprehensive report
        run: npm run test:report:generate
        
      - name: Upload comprehensive report
        uses: actions/upload-artifact@v3
        with:
          name: comprehensive-test-report
          path: test-reports/
          retention-days: 90
          
      - name: Send notification
        if: always()
        run: npm run test:notify
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
          TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
```

#### Configuración de Jest para CI/CD
```javascript
// jest.config.ci.js
module.exports = {
  ...require('./jest.config.js'),
  
  // Configuración específica para CI/CD
  ci: true,
  
  // Configuración de reportes
  reporters: [
    'default',
    [
      'jest-junit',
      {
        outputDirectory: 'test-results',
        outputName: 'junit.xml',
        classNameTemplate: '{classname}',
        titleTemplate: '{title}',
        ancestorSeparator: ' › ',
        usePathForSuiteName: true,
      },
    ],
    [
      'jest-html-reporters',
      {
        publicPath: 'test-results',
        filename: 'test-report.html',
        expand: true,
        hideIcon: true,
        testCommand: 'npm test',
      },
    ],
  ],
  
  // Configuración de cobertura para CI
  collectCoverage: true,
  coverageReporters: ['text', 'lcov', 'html', 'json'],
  coverageDirectory: 'coverage',
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  
  // Configuración de timeouts para CI
  testTimeout: 30000,
  
  // Configuración de workers para CI
  maxWorkers: 2,
  
  // Configuración de verbose para CI
  verbose: true,
  
  // Configuración de cache para CI
  cache: false,
  
  // Configuración de clearMocks para CI
  clearMocks: true,
  resetMocks: true,
  restoreMocks: true,
};
```

### 2. Testing Automatizado con Scripts

#### Scripts de Testing
```json
// package.json scripts
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest --config jest.config.ci.js --testPathPattern=__tests__ --testPathIgnorePatterns=e2e|integration",
    "test:integration": "jest --config jest.config.ci.js --testPathPattern=integration",
    "test:e2e": "detox test --configuration ios.sim.debug",
    "test:e2e:ios": "detox test --configuration ios.sim.debug --artifacts-location e2e/artifacts/ios",
    "test:e2e:android": "detox test --configuration android.emu.debug --artifacts-location e2e/artifacts/android",
    "test:performance": "jest --config jest.config.ci.js --testPathPattern=performance",
    "test:performance:report": "node scripts/generate-performance-report.js",
    "test:security": "npm run test:security:dependencies && npm run test:security:sast",
    "test:security:dependencies": "npm audit --audit-level moderate",
    "test:security:sast": "node scripts/security-scan.js",
    "test:report:generate": "node scripts/generate-test-report.js",
    "test:notify": "node scripts/send-test-notification.js",
    "test:server:start": "node scripts/start-mock-server.js",
    "test:server:stop": "node scripts/stop-mock-server.js",
    "test:coverage": "jest --coverage --coverageReporters=text --coverageReporters=lcov --coverageReporters=html",
    "test:watch": "jest --watch",
    "test:debug": "jest --detectOpenHandles --forceExit",
    "test:ci": "npm run test:unit && npm run test:integration && npm run test:performance && npm run test:security"
  }
}
```

#### Script de Generación de Reportes
```javascript
// scripts/generate-test-report.js
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');

class TestReportGenerator {
  constructor() {
    this.reportData = {
      timestamp: new Date().toISOString(),
      summary: {},
      details: {},
      coverage: {},
      performance: {},
      security: {},
    };
  }

  // Generar reporte completo
  async generateReport() {
    try {
      console.log('🔍 Generating comprehensive test report...');
      
      // Recopilar datos de diferentes fuentes
      await this.collectTestResults();
      await this.collectCoverageData();
      await this.collectPerformanceData();
      await this.collectSecurityData();
      
      // Generar resumen
      this.generateSummary();
      
      // Crear reporte HTML
      this.generateHTMLReport();
      
      // Crear reporte JSON
      this.generateJSONReport();
      
      // Crear reporte Markdown
      this.generateMarkdownReport();
      
      console.log('✅ Test report generated successfully!');
      
    } catch (error) {
      console.error('❌ Error generating test report:', error);
      process.exit(1);
    }
  }

  // Recopilar resultados de tests
  async collectTestResults() {
    console.log('📊 Collecting test results...');
    
    const testResultsPath = path.join(process.cwd(), 'test-results');
    
    if (fs.existsSync(testResultsPath)) {
      const files = fs.readdirSync(testResultsPath);
      
      for (const file of files) {
        if (file.endsWith('.xml')) {
          const filePath = path.join(testResultsPath, file);
          const content = fs.readFileSync(filePath, 'utf8');
          
          // Parsear resultados de Jest
          const results = this.parseJestResults(content);
          this.reportData.details[file] = results;
        }
      }
    }
  }

  // Recopilar datos de cobertura
  async collectCoverageData() {
    console.log('📈 Collecting coverage data...');
    
    const coveragePath = path.join(process.cwd(), 'coverage');
    
    if (fs.existsSync(coveragePath)) {
      const lcovPath = path.join(coveragePath, 'lcov.info');
      
      if (fs.existsSync(lcovPath)) {
        const lcovContent = fs.readFileSync(lcovPath, 'utf8');
        this.reportData.coverage = this.parseLcovData(lcovContent);
      }
      
      const summaryPath = path.join(coveragePath, 'coverage-summary.json');
      
      if (fs.existsSync(summaryPath)) {
        const summaryContent = fs.readFileSync(summaryPath, 'utf8');
        this.reportData.coverage.summary = JSON.parse(summaryContent);
      }
    }
  }

  // Recopilar datos de rendimiento
  async collectPerformanceData() {
    console.log('⚡ Collecting performance data...');
    
    const performancePath = path.join(process.cwd(), 'performance-reports');
    
    if (fs.existsSync(performancePath)) {
      const files = fs.readdirSync(performancePath);
      
      for (const file of files) {
        if (file.endsWith('.json')) {
          const filePath = path.join(performancePath, file);
          const content = fs.readFileSync(filePath, 'utf8');
          
          try {
            const data = JSON.parse(content);
            this.reportData.performance[file] = data;
          } catch (error) {
            console.warn(`Warning: Could not parse performance file ${file}`);
          }
        }
      }
    }
  }

  // Recopilar datos de seguridad
  async collectSecurityData() {
    console.log('🔒 Collecting security data...');
    
    const securityPath = path.join(process.cwd(), 'security-reports');
    
    if (fs.existsSync(securityPath)) {
      const files = fs.readdirSync(securityPath);
      
      for (const file of files) {
        if (file.endsWith('.json')) {
          const filePath = path.join(securityPath, file);
          const content = fs.readFileSync(filePath, 'utf8');
          
          try {
            const data = JSON.parse(content);
            this.reportData.security[file] = data;
          } catch (error) {
            console.warn(`Warning: Could not parse security file ${file}`);
          }
        }
      }
    }
  }

  // Generar resumen
  generateSummary() {
    console.log('📋 Generating summary...');
    
    // Resumen de tests
    let totalTests = 0;
    let passedTests = 0;
    let failedTests = 0;
    let skippedTests = 0;
    
    Object.values(this.reportData.details).forEach(result => {
      totalTests += result.total || 0;
      passedTests += result.passed || 0;
      failedTests += result.failed || 0;
      skippedTests += result.skipped || 0;
    });
    
    this.reportData.summary = {
      totalTests,
      passedTests,
      failedTests,
      skippedTests,
      successRate: totalTests > 0 ? ((passedTests / totalTests) * 100).toFixed(2) : 0,
      coverage: this.reportData.coverage.summary?.total?.statements?.pct || 0,
      performance: this.calculatePerformanceScore(),
      security: this.calculateSecurityScore(),
    };
  }

  // Calcular score de rendimiento
  calculatePerformanceScore() {
    if (!this.reportData.performance || Object.keys(this.reportData.performance).length === 0) {
      return 0;
    }
    
    let totalScore = 0;
    let count = 0;
    
    Object.values(this.reportData.performance).forEach(data => {
      if (data.score !== undefined) {
        totalScore += data.score;
        count++;
      }
    });
    
    return count > 0 ? (totalScore / count).toFixed(2) : 0;
  }

  // Calcular score de seguridad
  calculateSecurityScore() {
    if (!this.reportData.security || Object.keys(this.reportData.security).length === 0) {
      return 0;
    }
    
    let totalScore = 0;
    let count = 0;
    
    Object.values(this.reportData.security).forEach(data => {
      if (data.score !== undefined) {
        totalScore += data.score;
        count++;
      }
    });
    
    return count > 0 ? (totalScore / count).toFixed(2) : 0;
  }

  // Generar reporte HTML
  generateHTMLReport() {
    console.log('🌐 Generating HTML report...');
    
    const htmlTemplate = this.getHTMLTemplate();
    const reportPath = path.join(process.cwd(), 'test-reports', 'index.html');
    
    // Crear directorio si no existe
    const dir = path.dirname(reportPath);
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir, { recursive: true });
    }
    
    fs.writeFileSync(reportPath, htmlTemplate);
    console.log(`📄 HTML report saved to: ${reportPath}`);
  }

  // Generar reporte JSON
  generateJSONReport() {
    console.log('📄 Generating JSON report...');
    
    const reportPath = path.join(process.cwd(), 'test-reports', 'report.json');
    
    // Crear directorio si no existe
    const dir = path.dirname(reportPath);
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir, { recursive: true });
    }
    
    fs.writeFileSync(reportPath, JSON.stringify(this.reportData, null, 2));
    console.log(`📄 JSON report saved to: ${reportPath}`);
  }

  // Generar reporte Markdown
  generateMarkdownReport() {
    console.log('📝 Generating Markdown report...');
    
    const markdownContent = this.getMarkdownTemplate();
    const reportPath = path.join(process.cwd(), 'test-reports', 'README.md');
    
    // Crear directorio si no existe
    const dir = path.dirname(reportPath);
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir, { recursive: true });
    }
    
    fs.writeFileSync(reportPath, markdownContent);
    console.log(`📄 Markdown report saved to: ${reportPath}`);
  }

  // Template HTML
  getHTMLTemplate() {
    const { summary } = this.reportData;
    
    return `
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Test Report - ${new Date().toLocaleDateString()}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background: #f5f5f5; padding: 20px; border-radius: 8px; margin-bottom: 20px; }
        .summary { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-bottom: 20px; }
        .metric { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); text-align: center; }
        .metric-value { font-size: 2em; font-weight: bold; color: #007AFF; }
        .metric-label { color: #666; margin-top: 10px; }
        .success { color: #28a745; }
        .warning { color: #ffc107; }
        .danger { color: #dc3545; }
    </style>
</head>
<body>
    <div class="header">
        <h1>🧪 Test Report</h1>
        <p>Generated on: ${new Date().toLocaleString()}</p>
    </div>
    
    <div class="summary">
        <div class="metric">
            <div class="metric-value ${summary.successRate >= 90 ? 'success' : summary.successRate >= 70 ? 'warning' : 'danger'}">${summary.successRate}%</div>
            <div class="metric-label">Success Rate</div>
        </div>
        <div class="metric">
            <div class="metric-value">${summary.totalTests}</div>
            <div class="metric-label">Total Tests</div>
        </div>
        <div class="metric">
            <div class="metric-value">${summary.passedTests}</div>
            <div class="metric-label">Passed</div>
        </div>
        <div class="metric">
            <div class="metric-value">${summary.failedTests}</div>
            <div class="metric-label">Failed</div>
        </div>
        <div class="metric">
            <div class="metric-value">${summary.coverage}%</div>
            <div class="metric-label">Coverage</div>
        </div>
        <div class="metric">
            <div class="metric-value">${summary.performance}</div>
            <div class="metric-label">Performance Score</div>
        </div>
        <div class="metric">
            <div class="metric-value">${summary.security}</div>
            <div class="metric-label">Security Score</div>
        </div>
    </div>
    
    <h2>📊 Detailed Results</h2>
    <pre>${JSON.stringify(this.reportData, null, 2)}</pre>
</body>
</html>`;
  }

  // Template Markdown
  getMarkdownTemplate() {
    const { summary, timestamp } = this.reportData;
    
    return `# 🧪 Test Report

**Generated:** ${new Date(timestamp).toLocaleString()}

## 📋 Summary

| Metric | Value |
|--------|-------|
| **Success Rate** | ${summary.successRate}% |
| **Total Tests** | ${summary.totalTests} |
| **Passed** | ${summary.passedTests} |
| **Failed** | ${summary.failedTests} |
| **Skipped** | ${summary.skippedTests} |
| **Coverage** | ${summary.coverage}% |
| **Performance Score** | ${summary.performance} |
| **Security Score** | ${summary.security} |

## 📊 Test Results

### Unit Tests
- **Status**: ${summary.totalTests > 0 ? '✅ Completed' : '❌ No tests found'}
- **Coverage**: ${summary.coverage}%

### Performance Tests
- **Score**: ${summary.performance}
- **Status**: ${summary.performance >= 80 ? '✅ Good' : summary.performance >= 60 ? '⚠️ Needs improvement' : '❌ Poor'}

### Security Tests
- **Score**: ${summary.security}
- **Status**: ${summary.security >= 90 ? '✅ Secure' : summary.security >= 70 ? '⚠️ Needs attention' : '❌ Vulnerable'}

## 🔍 Recommendations

${this.generateRecommendations()}

---
*Report generated automatically by Test Report Generator*
`;
  }

  // Generar recomendaciones
  generateRecommendations() {
    const recommendations = [];
    
    if (this.reportData.summary.successRate < 90) {
      recommendations.push('- 🔴 **High Priority**: Fix failing tests to improve success rate');
    }
    
    if (this.reportData.summary.coverage < 80) {
      recommendations.push('- 🟡 **Medium Priority**: Increase test coverage to at least 80%');
    }
    
    if (this.reportData.summary.performance < 80) {
      recommendations.push('- 🟡 **Medium Priority**: Optimize performance based on test results');
    }
    
    if (this.reportData.summary.security < 90) {
      recommendations.push('- 🔴 **High Priority**: Address security vulnerabilities immediately');
    }
    
    if (recommendations.length === 0) {
      recommendations.push('- ✅ **All Good**: Tests are passing and quality metrics are within acceptable ranges');
    }
    
    return recommendations.join('\n');
  }

  // Parsear resultados de Jest
  parseJestResults(xmlContent) {
    // Implementación básica de parsing XML
    // En un entorno real, usaría una librería como xml2js
    const totalMatch = xmlContent.match(/testsuites.*tests="(\d+)"/);
    const failuresMatch = xmlContent.match(/failures="(\d+)"/);
    const skippedMatch = xmlContent.match(/skipped="(\d+)"/);
    
    return {
      total: totalMatch ? parseInt(totalMatch[1]) : 0,
      failed: failuresMatch ? parseInt(failuresMatch[1]) : 0,
      skipped: skippedMatch ? parseInt(skippedMatch[1]) : 0,
      passed: totalMatch ? parseInt(totalMatch[1]) - (failuresMatch ? parseInt(failuresMatch[1]) : 0) - (skippedMatch ? parseInt(skippedMatch[1]) : 0) : 0,
    };
  }

  // Parsear datos de LCOV
  parseLcovData(lcovContent) {
    // Implementación básica de parsing LCOV
    const lines = lcovContent.split('\n');
    let totalLines = 0;
    let coveredLines = 0;
    
    for (const line of lines) {
      if (line.startsWith('LF:')) {
        totalLines += parseInt(line.split(':')[1]);
      } else if (line.startsWith('LH:')) {
        coveredLines += parseInt(line.split(':')[1]);
      }
    }
    
    return {
      totalLines,
      coveredLines,
      coverage: totalLines > 0 ? ((coveredLines / totalLines) * 100).toFixed(2) : 0,
    };
  }
}

// Ejecutar generador
if (require.main === module) {
  const generator = new TestReportGenerator();
  generator.generateReport();
}

module.exports = TestReportGenerator;
```

### 3. Notificaciones y Alertas

#### Script de Notificaciones
```javascript
// scripts/send-test-notification.js
const https = require('https');
const fs = require('fs');
const path = require('path');

class TestNotificationSender {
  constructor() {
    this.reportPath = path.join(process.cwd(), 'test-reports', 'report.json');
    this.reportData = null;
  }

  // Enviar notificaciones
  async sendNotifications() {
    try {
      console.log('📢 Sending test notifications...');
      
      // Cargar reporte
      await this.loadReport();
      
      if (!this.reportData) {
        console.log('⚠️ No report data found, skipping notifications');
        return;
      }
      
      // Enviar notificación a Slack
      if (process.env.SLACK_WEBHOOK) {
        await this.sendSlackNotification();
      }
      
      // Enviar notificación a Teams
      if (process.env.TEAMS_WEBHOOK) {
        await this.sendTeamsNotification();
      }
      
      // Enviar notificación por email (si está configurado)
      if (process.env.EMAIL_SMTP_HOST) {
        await this.sendEmailNotification();
      }
      
      console.log('✅ Notifications sent successfully!');
      
    } catch (error) {
      console.error('❌ Error sending notifications:', error);
    }
  }

  // Cargar reporte
  async loadReport() {
    if (fs.existsSync(this.reportPath)) {
      const content = fs.readFileSync(this.reportPath, 'utf8');
      this.reportData = JSON.parse(content);
    }
  }

  // Enviar notificación a Slack
  async sendSlackNotification() {
    const { summary } = this.reportData;
    
    const message = {
      text: '🧪 Test Results Summary',
      blocks: [
        {
          type: 'header',
          text: {
            type: 'plain_text',
            text: '🧪 Test Results Summary',
            emoji: true,
          },
        },
        {
          type: 'section',
          fields: [
            {
              type: 'mrkdwn',
              text: `*Success Rate:*\n${summary.successRate}%`,
            },
            {
              type: 'mrkdwn',
              text: `*Total Tests:*\n${summary.totalTests}`,
            },
            {
              type: 'mrkdwn',
              text: `*Coverage:*\n${summary.coverage}%`,
            },
            {
              type: 'mrkdwn',
              text: `*Performance:*\n${summary.performance}`,
            },
          ],
        },
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: this.getSlackStatusText(summary),
          },
        },
        {
          type: 'actions',
          elements: [
            {
              type: 'button',
              text: {
                type: 'plain_text',
                text: 'View Full Report',
                emoji: true,
              },
              url: `${process.env.GITHUB_SERVER_URL || 'https://github.com'}/${process.env.GITHUB_REPOSITORY}/actions/runs/${process.env.GITHUB_RUN_ID}`,
            },
          ],
        },
      ],
    };
    
    await this.sendWebhook(process.env.SLACK_WEBHOOK, message);
  }

  // Enviar notificación a Teams
  async sendTeamsNotification() {
    const { summary } = this.reportData;
    
    const message = {
      '@type': 'MessageCard',
      '@context': 'http://schema.org/extensions',
      summary: 'Test Results Summary',
      title: '🧪 Test Results Summary',
      text: this.getTeamsStatusText(summary),
      themeColor: this.getTeamsThemeColor(summary),
      sections: [
        {
          title: '📊 Test Metrics',
          facts: [
            {
              name: 'Success Rate',
              value: `${summary.successRate}%`,
            },
            {
              name: 'Total Tests',
              value: summary.totalTests.toString(),
            },
            {
              name: 'Coverage',
              value: `${summary.coverage}%`,
            },
            {
              name: 'Performance Score',
              value: summary.performance.toString(),
            },
            {
              name: 'Security Score',
              value: summary.security.toString(),
            },
          ],
        },
      ],
      potentialAction: [
        {
          '@type': 'OpenUri',
          name: 'View Full Report',
          targets: [
            {
              os: 'default',
              uri: `${process.env.GITHUB_SERVER_URL || 'https://github.com'}/${process.env.GITHUB_REPOSITORY}/actions/runs/${process.env.GITHUB_RUN_ID}`,
            },
          ],
        },
      ],
    };
    
    await this.sendWebhook(process.env.TEAMS_WEBHOOK, message);
  }

  // Enviar notificación por email
  async sendEmailNotification() {
    // Implementación de envío por email
    // Requiere configuración de SMTP
    console.log('📧 Email notifications not implemented yet');
  }

  // Enviar webhook
  async sendWebhook(url, data) {
    return new Promise((resolve, reject) => {
      const postData = JSON.stringify(data);
      
      const options = {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Content-Length': Buffer.byteLength(postData),
        },
      };
      
      const req = https.request(url, options, (res) => {
        let responseData = '';
        
        res.on('data', (chunk) => {
          responseData += chunk;
        });
        
        res.on('end', () => {
          if (res.statusCode >= 200 && res.statusCode < 300) {
            resolve(responseData);
          } else {
            reject(new Error(`HTTP ${res.statusCode}: ${responseData}`));
          }
        });
      });
      
      req.on('error', (error) => {
        reject(error);
      });
      
      req.write(postData);
      req.end();
    });
  }

  // Obtener texto de estado para Slack
  getSlackStatusText(summary) {
    if (summary.successRate >= 90 && summary.coverage >= 80) {
      return '🎉 *All tests are passing and quality metrics are excellent!*';
    } else if (summary.successRate >= 70 && summary.coverage >= 60) {
      return '⚠️ *Tests are mostly passing but some improvements are needed.*';
    } else {
      return '🚨 *Critical issues detected! Immediate attention required.*';
    }
  }

  // Obtener texto de estado para Teams
  getTeamsStatusText(summary) {
    if (summary.successRate >= 90 && summary.coverage >= 80) {
      return '🎉 All tests are passing and quality metrics are excellent!';
    } else if (summary.successRate >= 70 && summary.coverage >= 60) {
      return '⚠️ Tests are mostly passing but some improvements are needed.';
    } else {
      return '🚨 Critical issues detected! Immediate attention required.';
    }
  }

  // Obtener color de tema para Teams
  getTeamsThemeColor(summary) {
    if (summary.successRate >= 90 && summary.coverage >= 80) {
      return '00FF00'; // Verde
    } else if (summary.successRate >= 70 && summary.coverage >= 60) {
      return 'FFA500'; // Naranja
    } else {
      return 'FF0000'; // Rojo
    }
  }
}

// Ejecutar notificador
if (require.main === module) {
  const notifier = new TestNotificationSender();
  notifier.sendNotifications();
}

module.exports = TestNotificationSender;
```

## Ejercicios Prácticos

### Ejercicio 1: Pipeline de Testing Completo
Crea un pipeline de testing completo para una aplicación React Native:

```yaml
# Pipeline a implementar
name: Complete Testing Pipeline

on:
  push:
    branches: [ main, develop, feature/* ]
  pull_request:
    branches: [ main, develop ]

jobs:
  # 1. Linting y formateo
  # 2. Testing unitario
  # 3. Testing de integración
  # 4. Testing E2E en iOS
  # 5. Testing E2E en Android
  # 6. Testing de rendimiento
  # 7. Testing de seguridad
  # 8. Generación de reportes
  # 9. Notificaciones
```

**Requisitos del pipeline:**
- Ejecución paralela cuando sea posible
- Caché de dependencias
- Artefactos de testing
- Reportes automáticos
- Notificaciones inteligentes

### Ejercicio 2: Sistema de Métricas de Testing
Crea un sistema completo de métricas y reportes:

```javascript
// Sistema a implementar
export class TestingMetricsSystem {
  constructor() {
    this.metrics = {};
  }

  // Métricas de ejecución
  trackTestExecution() { /* implementar */ }
  
  // Métricas de cobertura
  trackCoverage() { /* implementar */ }
  
  // Métricas de rendimiento
  trackPerformance() { /* implementar */ }
  
  // Métricas de calidad
  trackQuality() { /* implementar */ }
  
  // Generar dashboard
  generateDashboard() { /* implementar */ }
}
```

**Requisitos del sistema:**
- Tracking automático de métricas
- Dashboard en tiempo real
- Alertas configurables
- Histórico de métricas
- Exportación de datos

### Ejercicio 3: Testing en Múltiples Entornos
Crea un sistema de testing para diferentes entornos:

```javascript
// Sistema a implementar
describe('Multi-Environment Testing', () => {
  it('debe ejecutar tests en desarrollo', async () => {
    // 1. Configurar entorno de desarrollo
    // 2. Ejecutar tests unitarios
    // 3. Ejecutar tests de integración
    // 4. Verificar resultados
  });

  it('debe ejecutar tests en staging', async () => {
    // 1. Configurar entorno de staging
    // 2. Ejecutar tests E2E
    // 3. Ejecutar tests de rendimiento
    // 4. Verificar resultados
  });

  it('debe ejecutar tests en producción', async () => {
    // 1. Configurar entorno de producción
    // 2. Ejecutar tests de humo
    // 3. Ejecutar tests de seguridad
    // 4. Verificar resultados
  });
});
```

**Requisitos del testing:**
- Configuración automática de entornos
- Tests específicos por entorno
- Validación de configuración
- Rollback automático en fallos
- Monitoreo de salud del sistema

## Resumen de la Clase

En esta clase hemos aprendido:

1. **Pipelines de Testing**: Configuración completa de CI/CD con GitHub Actions
2. **Testing Automatizado**: Scripts y herramientas para automatización
3. **Reportes y Métricas**: Generación automática de reportes y dashboards
4. **Notificaciones**: Sistema de alertas y notificaciones inteligentes

## Navegación
- **Anterior**: [Clase 4: Testing de Rendimiento](clase_4_testing_rendimiento.md)
- **Siguiente**: [README del Módulo](../README.md)
- **Inicio**: [Índice del Curso](../../INDICE_COMPLETO.md)
