# 📚 Clase 3: Configuración del Entorno

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 2: Diferencias con React Web](clase_2_diferencias_react_web.md)
- **➡️ Siguiente**: [Clase 4: Primer Proyecto](clase_4_primer_proyecto.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Configurar completamente** el entorno de desarrollo para React Native
2. **Instalar y configurar** Node.js, npm/yarn y React Native CLI
3. **Configurar Android Studio** para desarrollo Android
4. **Configurar Xcode** para desarrollo iOS (solo macOS)
5. **Verificar la instalación** y resolver problemas comunes

---

## 📚 Contenido Teórico

### **¿Qué Necesitas para Desarrollar en React Native?**

Para desarrollar aplicaciones React Native, necesitas configurar varios componentes:

#### **Herramientas Básicas:**
- **Node.js**: Runtime de JavaScript
- **npm/yarn**: Gestores de paquetes
- **React Native CLI**: Herramienta de línea de comandos
- **Git**: Control de versiones

#### **Para Android:**
- **Android Studio**: IDE oficial para Android
- **Android SDK**: Kit de desarrollo de Android
- **Java Development Kit (JDK)**: Entorno de desarrollo Java

#### **Para iOS (solo macOS):**
- **Xcode**: IDE oficial para iOS
- **CocoaPods**: Gestor de dependencias para iOS
- **Simulador de iOS**: Para probar apps

---

## 💻 Implementación Práctica

### **1. Script de Verificación del Entorno**

```javascript:src/scripts/environmentCheck.js
// Script para verificar que el entorno esté correctamente configurado
import { execSync } from 'child_process';
import { platform } from 'os';

class EnvironmentChecker {
  constructor() {
    this.platform = platform();
    this.checks = [];
    this.errors = [];
    this.warnings = [];
  }

  // Verificar Node.js
  checkNodeJS() {
    try {
      const nodeVersion = execSync('node --version', { encoding: 'utf8' }).trim();
      const npmVersion = execSync('npm --version', { encoding: 'utf8' }).trim();
      
      this.checks.push({
        name: 'Node.js',
        status: '✅',
        version: nodeVersion,
        details: `npm: ${npmVersion}`
      });
      
      // Verificar versión mínima de Node.js
      const majorVersion = parseInt(nodeVersion.replace('v', '').split('.')[0]);
      if (majorVersion < 16) {
        this.warnings.push('Node.js 16+ es recomendado para React Native');
      }
    } catch (error) {
      this.errors.push('Node.js no está instalado o no está en el PATH');
    }
  }

  // Verificar React Native CLI
  checkReactNativeCLI() {
    try {
      const rnVersion = execSync('npx react-native --version', { encoding: 'utf8' }).trim();
      
      this.checks.push({
        name: 'React Native CLI',
        status: '✅',
        version: rnVersion,
        details: 'Instalado globalmente'
      });
    } catch (error) {
      try {
        // Verificar si está instalado localmente
        const localVersion = execSync('npx react-native --version', { encoding: 'utf8' }).trim();
        this.checks.push({
          name: 'React Native CLI',
          status: '⚠️',
          version: localVersion,
          details: 'Instalado localmente (recomendado global)'
        });
      } catch (localError) {
        this.errors.push('React Native CLI no está instalado');
      }
    }
  }

  // Verificar Android (solo en Windows/Linux)
  checkAndroid() {
    if (this.platform === 'win32' || this.platform === 'linux') {
      try {
        // Verificar ANDROID_HOME
        const androidHome = process.env.ANDROID_HOME;
        if (!androidHome) {
          this.errors.push('Variable ANDROID_HOME no está configurada');
          return;
        }

        // Verificar Android SDK
        const sdkVersion = execSync(`${androidHome}/tools/bin/sdkmanager --version`, { encoding: 'utf8' }).trim();
        
        this.checks.push({
          name: 'Android SDK',
          status: '✅',
          version: sdkVersion,
          details: `Path: ${androidHome}`
        });

        // Verificar emulador
        try {
          const emulatorList = execSync(`${androidHome}/emulator/emulator -list-avds`, { encoding: 'utf8' }).trim();
          if (emulatorList) {
            this.checks.push({
              name: 'Android Emulator',
              status: '✅',
              version: 'Disponible',
              details: `${emulatorList.split('\n').length} AVDs configurados`
            });
          } else {
            this.warnings.push('No hay emuladores Android configurados');
          }
        } catch (emulatorError) {
          this.warnings.push('Android Emulator no está configurado correctamente');
        }

      } catch (error) {
        this.errors.push('Android SDK no está configurado correctamente');
      }
    }
  }

  // Verificar iOS (solo en macOS)
  checkIOS() {
    if (this.platform === 'darwin') {
      try {
        // Verificar Xcode
        const xcodeVersion = execSync('xcodebuild -version', { encoding: 'utf8' }).trim();
        
        this.checks.push({
          name: 'Xcode',
          status: '✅',
          version: xcodeVersion.split('\n')[0],
          details: 'Instalado y configurado'
        });

        // Verificar CocoaPods
        try {
          const podVersion = execSync('pod --version', { encoding: 'utf8' }).trim();
          this.checks.push({
            name: 'CocoaPods',
            status: '✅',
            version: podVersion,
            details: 'Instalado'
          });
        } catch (podError) {
          this.warnings.push('CocoaPods no está instalado');
        }

        // Verificar simulador de iOS
        try {
          const simulatorList = execSync('xcrun simctl list devices', { encoding: 'utf8' }).trim();
          if (simulatorList.includes('iPhone')) {
            this.checks.push({
              name: 'iOS Simulator',
              status: '✅',
              version: 'Disponible',
              details: 'Simuladores iOS configurados'
            });
          } else {
            this.warnings.push('No hay simuladores iOS configurados');
          }
        } catch (simulatorError) {
          this.warnings.push('iOS Simulator no está configurado correctamente');
        }

      } catch (error) {
        this.errors.push('Xcode no está instalado o configurado');
      }
    }
  }

  // Verificar Git
  checkGit() {
    try {
      const gitVersion = execSync('git --version', { encoding: 'utf8' }).trim();
      
      this.checks.push({
        name: 'Git',
        status: '✅',
        version: gitVersion,
        details: 'Configurado'
      });
    } catch (error) {
      this.warnings.push('Git no está instalado');
    }
  }

  // Ejecutar todas las verificaciones
  runAllChecks() {
    console.log('🔍 Verificando entorno de desarrollo...\n');
    
    this.checkNodeJS();
    this.checkReactNativeCLI();
    this.checkAndroid();
    this.checkIOS();
    this.checkGit();
    
    this.displayResults();
  }

  // Mostrar resultados
  displayResults() {
    console.log('📊 RESULTADOS DE LA VERIFICACIÓN:\n');
    
    // Mostrar verificaciones exitosas
    this.checks.forEach(check => {
      console.log(`${check.status} ${check.name}: ${check.version}`);
      if (check.details) {
        console.log(`   ${check.details}`);
      }
    });
    
    // Mostrar advertencias
    if (this.warnings.length > 0) {
      console.log('\n⚠️  ADVERTENCIAS:');
      this.warnings.forEach(warning => {
        console.log(`   • ${warning}`);
      });
    }
    
    // Mostrar errores
    if (this.errors.length > 0) {
      console.log('\n❌ ERRORES:');
      this.errors.forEach(error => {
        console.log(`   • ${error}`);
      });
      console.log('\n🔧 Resuelve estos errores antes de continuar.');
    } else {
      console.log('\n🎉 ¡Tu entorno está listo para React Native!');
    }
    
    // Mostrar recomendaciones
    this.displayRecommendations();
  }

  // Mostrar recomendaciones
  displayRecommendations() {
    console.log('\n💡 RECOMENDACIONES:');
    
    if (this.platform === 'win32') {
      console.log('   • Windows: Considera usar WSL2 para mejor rendimiento');
      console.log('   • Instala Android Studio desde la página oficial');
    } else if (this.platform === 'darwin') {
      console.log('   • macOS: Instala Xcode desde la App Store');
      console.log('   • Ejecuta: sudo gem install cocoapods');
    } else if (this.platform === 'linux') {
      console.log('   • Linux: Instala Android Studio desde la página oficial');
      console.log('   • Configura variables de entorno en ~/.bashrc');
    }
    
    console.log('   • Mantén Node.js actualizado (versión LTS recomendada)');
    console.log('   • Usa npx para ejecutar React Native CLI');
  }
}

// Función para ejecutar la verificación
export const checkEnvironment = () => {
  const checker = new EnvironmentChecker();
  checker.runAllChecks();
  
  return {
    checks: checker.checks,
    warnings: checker.warnings,
    errors: checker.errors,
    isReady: checker.errors.length === 0
  };
};

// Ejecutar si se llama directamente
if (require.main === module) {
  checkEnvironment();
}
```

### **2. Script de Configuración Automática**

```bash:scripts/setup-environment.sh
#!/bin/bash

# Script de configuración automática del entorno React Native
# Compatible con macOS, Linux y Windows (WSL)

set -e

echo "🚀 Configurando entorno de desarrollo React Native..."

# Detectar sistema operativo
if [[ "$OSTYPE" == "darwin"* ]]; then
    OS="macos"
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
    OS="linux"
elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]]; then
    OS="windows"
else
    echo "❌ Sistema operativo no soportado: $OSTYPE"
    exit 1
fi

echo "📱 Sistema operativo detectado: $OS"

# Función para instalar Node.js
install_nodejs() {
    echo "📦 Verificando Node.js..."
    
    if command -v node &> /dev/null; then
        NODE_VERSION=$(node --version)
        echo "✅ Node.js ya está instalado: $NODE_VERSION"
    else
        echo "📥 Instalando Node.js..."
        
        if [[ "$OS" == "macos" ]]; then
            # macOS: usar Homebrew
            if command -v brew &> /dev/null; then
                brew install node
            else
                echo "📥 Instalando Homebrew..."
                /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
                brew install node
            fi
        elif [[ "$OS" == "linux" ]]; then
            # Linux: usar NodeSource
            curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
            sudo apt-get install -y nodejs
        elif [[ "$OS" == "windows" ]]; then
            echo "📥 Por favor, instala Node.js desde https://nodejs.org/"
            echo "   Selecciona la versión LTS y reinicia tu terminal"
            exit 1
        fi
    fi
    
    # Verificar npm
    if command -v npm &> /dev/null; then
        NPM_VERSION=$(npm --version)
        echo "✅ npm instalado: $NPM_VERSION"
    else
        echo "❌ npm no está disponible"
        exit 1
    fi
}

# Función para instalar React Native CLI
install_react_native_cli() {
    echo "📱 Verificando React Native CLI..."
    
    if npx react-native --version &> /dev/null; then
        RN_VERSION=$(npx react-native --version)
        echo "✅ React Native CLI disponible: $RN_VERSION"
    else
        echo "📥 Instalando React Native CLI globalmente..."
        npm install -g @react-native-community/cli
    fi
}

# Función para configurar Android (Linux y Windows)
setup_android() {
    if [[ "$OS" == "linux" ]] || [[ "$OS" == "windows" ]]; then
        echo "🤖 Configurando entorno Android..."
        
        # Verificar si Android Studio está instalado
        if [[ -d "$HOME/Android/Sdk" ]] || [[ -d "/usr/local/android-sdk" ]]; then
            echo "✅ Android SDK detectado"
        else
            echo "📥 Por favor, instala Android Studio desde:"
            echo "   https://developer.android.com/studio"
            echo "   Durante la instalación, asegúrate de instalar:"
            echo "   - Android SDK"
            echo "   - Android SDK Platform"
            echo "   - Android Virtual Device"
            echo "   - Performance (Intel HAXM)"
        fi
        
        # Configurar variables de entorno
        if [[ "$OS" == "linux" ]]; then
            echo "🔧 Configurando variables de entorno..."
            
            # Agregar al .bashrc
            if ! grep -q "ANDROID_HOME" ~/.bashrc; then
                echo "" >> ~/.bashrc
                echo "# React Native Android" >> ~/.bashrc
                echo "export ANDROID_HOME=\$HOME/Android/Sdk" >> ~/.bashrc
                echo "export PATH=\$PATH:\$ANDROID_HOME/emulator" >> ~/.bashrc
                echo "export PATH=\$PATH:\$ANDROID_HOME/tools" >> ~/.bashrc
                echo "export PATH=\$PATH:\$ANDROID_HOME/tools/bin" >> ~/.bashrc
                echo "export PATH=\$PATH:\$ANDROID_HOME/platform-tools" >> ~/.bashrc
            fi
            
            echo "✅ Variables de entorno configuradas en ~/.bashrc"
            echo "   Por favor, reinicia tu terminal o ejecuta: source ~/.bashrc"
        fi
    fi
}

# Función para configurar iOS (solo macOS)
setup_ios() {
    if [[ "$OS" == "macos" ]]; then
        echo "🍎 Configurando entorno iOS..."
        
        # Verificar Xcode
        if command -v xcodebuild &> /dev/null; then
            XCODE_VERSION=$(xcodebuild -version | head -n 1)
            echo "✅ Xcode instalado: $XCODE_VERSION"
        else
            echo "📥 Por favor, instala Xcode desde la App Store"
            echo "   Después de la instalación, ejecuta:"
            echo "   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer"
            exit 1
        fi
        
        # Verificar CocoaPods
        if command -v pod &> /dev/null; then
            POD_VERSION=$(pod --version)
            echo "✅ CocoaPods instalado: $POD_VERSION"
        else
            echo "📥 Instalando CocoaPods..."
            sudo gem install cocoapods
        fi
        
        # Configurar simulador
        echo "📱 Configurando simulador iOS..."
        sudo xcrun simctl list devices &> /dev/null
        echo "✅ Simulador iOS configurado"
    fi
}

# Función para verificar Java
check_java() {
    echo "☕ Verificando Java..."
    
    if command -v java &> /dev/null; then
        JAVA_VERSION=$(java -version 2>&1 | head -n 1)
        echo "✅ Java instalado: $JAVA_VERSION"
    else
        echo "📥 Por favor, instala Java Development Kit (JDK) 11 o superior"
        if [[ "$OS" == "macos" ]]; then
            echo "   macOS: brew install openjdk@11"
        elif [[ "$OS" == "linux" ]]; then
            echo "   Linux: sudo apt-get install openjdk-11-jdk"
        fi
    fi
}

# Función para crear proyecto de prueba
create_test_project() {
    echo "🧪 Creando proyecto de prueba..."
    
    if [[ -d "TestProject" ]]; then
        echo "⚠️  El directorio TestProject ya existe, saltando..."
        return
    fi
    
    echo "📱 Creando proyecto React Native de prueba..."
    npx react-native init TestProject --template react-native-template-typescript
    
    if [[ $? -eq 0 ]]; then
        echo "✅ Proyecto de prueba creado exitosamente"
        echo "📁 Para ejecutar el proyecto:"
        echo "   cd TestProject"
        if [[ "$OS" == "macos" ]]; then
            echo "   npx react-native run-ios"
        fi
        echo "   npx react-native run-android"
    else
        echo "❌ Error al crear el proyecto de prueba"
    fi
}

# Función principal
main() {
    echo "🎯 Iniciando configuración del entorno React Native..."
    
    install_nodejs
    install_react_native_cli
    check_java
    setup_android
    setup_ios
    
    echo ""
    echo "🎉 ¡Configuración completada!"
    echo ""
    echo "📋 Próximos pasos:"
    echo "   1. Reinicia tu terminal"
    echo "   2. Ejecuta: npx react-native doctor"
    echo "   3. Crea tu primer proyecto: npx react-native init MiApp"
    echo ""
    
    # Preguntar si crear proyecto de prueba
    read -p "¿Quieres crear un proyecto de prueba? (y/n): " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        create_test_project
    fi
}

# Ejecutar función principal
main "$@"
```

### **3. Configuración de Variables de Entorno**

```bash:scripts/setup-env-vars.sh
#!/bin/bash

# Script para configurar variables de entorno para React Native

echo "🔧 Configurando variables de entorno para React Native..."

# Detectar sistema operativo
if [[ "$OSTYPE" == "darwin"* ]]; then
    OS="macos"
    SHELL_RC="$HOME/.zshrc"
    if [[ ! -f "$SHELL_RC" ]]; then
        SHELL_RC="$HOME/.bash_profile"
    fi
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
    OS="linux"
    SHELL_RC="$HOME/.bashrc"
elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]]; then
    OS="windows"
    echo "⚠️  En Windows, configura las variables de entorno manualmente:"
    echo "   - ANDROID_HOME: C:\Users\TuUsuario\AppData\Local\Android\Sdk"
    echo "   - PATH: Agrega las siguientes rutas:"
    echo "     %ANDROID_HOME%\platform-tools"
    echo "     %ANDROID_HOME%\emulator"
    echo "     %ANDROID_HOME%\tools"
    echo "     %ANDROID_HOME%\tools\bin"
    exit 0
else
    echo "❌ Sistema operativo no soportado: $OSTYPE"
    exit 1
fi

echo "📱 Sistema operativo: $OS"
echo "📁 Archivo de configuración: $SHELL_RC"

# Función para agregar variable si no existe
add_env_var() {
    local var_name="$1"
    local var_value="$2"
    
    if ! grep -q "export $var_name=" "$SHELL_RC"; then
        echo "" >> "$SHELL_RC"
        echo "# React Native - $var_name" >> "$SHELL_RC"
        echo "export $var_name=\"$var_value\"" >> "$SHELL_RC"
        echo "✅ Variable $var_name agregada"
    else
        echo "⚠️  Variable $var_name ya existe"
    fi
}

# Función para agregar al PATH si no existe
add_to_path() {
    local path_value="$1"
    
    if ! grep -q "export PATH=\$PATH:$path_value" "$SHELL_RC"; then
        echo "" >> "$SHELL_RC"
        echo "# React Native - PATH additions" >> "$SHELL_RC"
        echo "export PATH=\$PATH:$path_value" >> "$SHELL_RC"
        echo "✅ Ruta agregada al PATH: $path_value"
    else
        echo "⚠️  Ruta ya existe en PATH: $path_value"
    fi
}

# Configurar variables para Android
if [[ "$OS" == "linux" ]] || [[ "$OS" == "macos" ]]; then
    echo ""
    echo "🤖 Configurando variables de Android..."
    
    # Buscar Android SDK
    ANDROID_PATHS=(
        "$HOME/Android/Sdk"
        "/usr/local/android-sdk"
        "/opt/android-sdk"
        "$HOME/Library/Android/sdk"
    )
    
    ANDROID_HOME=""
    for path in "${ANDROID_PATHS[@]}"; do
        if [[ -d "$path" ]]; then
            ANDROID_HOME="$path"
            break
        fi
    done
    
    if [[ -n "$ANDROID_HOME" ]]; then
        echo "✅ Android SDK encontrado en: $ANDROID_HOME"
        
        # Agregar ANDROID_HOME
        add_env_var "ANDROID_HOME" "$ANDROID_HOME"
        
        # Agregar rutas al PATH
        add_to_path "\$ANDROID_HOME/emulator"
        add_to_path "\$ANDROID_HOME/tools"
        add_to_path "\$ANDROID_HOME/tools/bin"
        add_to_path "\$ANDROID_HOME/platform-tools"
        
    else
        echo "❌ Android SDK no encontrado"
        echo "📥 Por favor, instala Android Studio y configura ANDROID_HOME manualmente"
    fi
fi

# Configurar variables para iOS (solo macOS)
if [[ "$OS" == "macos" ]]; then
    echo ""
    echo "🍎 Configurando variables de iOS..."
    
    # Verificar Xcode
    if command -v xcodebuild &> /dev/null; then
        echo "✅ Xcode detectado"
        
        # Agregar variables de iOS si no existen
        if ! grep -q "export IOS_DEPLOYMENT_TARGET" "$SHELL_RC"; then
            echo "" >> "$SHELL_RC"
            echo "# React Native - iOS" >> "$SHELL_RC"
            echo "export IOS_DEPLOYMENT_TARGET=12.4" >> "$SHELL_RC"
            echo "✅ Variable IOS_DEPLOYMENT_TARGET agregada"
        fi
    else
        echo "⚠️  Xcode no está instalado"
    fi
fi

# Configurar variables generales
echo ""
echo "⚙️  Configurando variables generales..."

# Agregar variables de React Native
if ! grep -q "export REACT_NATIVE_PACKAGER_HOSTNAME" "$SHELL_RC"; then
    echo "" >> "$SHELL_RC"
    echo "# React Native - General" >> "$SHELL_RC"
    echo "export REACT_NATIVE_PACKAGER_HOSTNAME=localhost" >> "$SHELL_RC"
    echo "✅ Variable REACT_NATIVE_PACKAGER_HOSTNAME agregada"
fi

# Agregar variables de Metro
if ! grep -q "export REACT_NATIVE_PACKAGER_RESET_CACHE" "$SHELL_RC"; then
    echo "" >> "$SHELL_RC"
    echo "export REACT_NATIVE_PACKAGER_RESET_CACHE=true" >> "$SHELL_RC"
    echo "✅ Variable REACT_NATIVE_PACKAGER_RESET_CACHE agregada"
fi

echo ""
echo "🎉 Variables de entorno configuradas en $SHELL_RC"
echo ""
echo "📋 Para aplicar los cambios:"
echo "   1. Reinicia tu terminal, o"
echo "   2. Ejecuta: source $SHELL_RC"
echo ""
echo "🔍 Para verificar la configuración:"
echo "   echo \$ANDROID_HOME"
echo "   echo \$PATH"
echo ""
echo "🧪 Para probar la configuración:"
echo "   npx react-native doctor"
```

---

## 🧪 Casos de Uso Prácticos

### **1. Verificación del Entorno en tu App**

```javascript:src/components/EnvironmentStatus.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  ScrollView,
  TouchableOpacity,
  StyleSheet,
  Alert
} from 'react-native';

const EnvironmentStatus = () => {
  const [environmentInfo, setEnvironmentInfo] = useState(null);
  const [isChecking, setIsChecking] = useState(false);

  const checkEnvironment = async () => {
    setIsChecking(true);
    
    try {
      // Simular verificación del entorno
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      const info = {
        nodeJS: {
          installed: true,
          version: 'v18.17.0',
          status: '✅'
        },
        reactNativeCLI: {
          installed: true,
          version: '11.3.7',
          status: '✅'
        },
        android: {
          installed: true,
          sdkVersion: '33.0.0',
          emulator: true,
          status: '✅'
        },
        ios: {
          installed: true,
          xcodeVersion: '14.3.1',
          simulator: true,
          status: '✅'
        }
      };
      
      setEnvironmentInfo(info);
    } catch (error) {
      Alert.alert('Error', 'Error al verificar el entorno');
    } finally {
      setIsChecking(false);
    }
  };

  useEffect(() => {
    checkEnvironment();
  }, []);

  const renderStatusItem = (title, info) => (
    <View style={styles.statusItem}>
      <Text style={styles.statusTitle}>{title}</Text>
      <View style={styles.statusDetails}>
        <Text style={styles.statusIcon}>{info.status}</Text>
        <View style={styles.statusInfo}>
          <Text style={styles.statusText}>
            {info.installed ? 'Instalado' : 'No instalado'}
          </Text>
          {info.version && (
            <Text style={styles.versionText}>v{info.version}</Text>
          )}
        </View>
      </View>
    </View>
  );

  if (!environmentInfo) {
    return (
      <View style={styles.loadingContainer}>
        <Text style={styles.loadingText}>
          {isChecking ? 'Verificando entorno...' : 'Presiona para verificar'}
        </Text>
        {!isChecking && (
          <TouchableOpacity style={styles.checkButton} onPress={checkEnvironment}>
            <Text style={styles.buttonText}>Verificar Entorno</Text>
          </TouchableOpacity>
        )}
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Estado del Entorno</Text>
      
      {renderStatusItem('Node.js', environmentInfo.nodeJS)}
      {renderStatusItem('React Native CLI', environmentInfo.reactNativeCLI)}
      {renderStatusItem('Android SDK', environmentInfo.android)}
      {renderStatusItem('iOS Development', environmentInfo.ios)}
      
      <View style={styles.summaryContainer}>
        <Text style={styles.summaryTitle}>Resumen</Text>
        <Text style={styles.summaryText}>
          Tu entorno está configurado correctamente para desarrollar aplicaciones React Native.
        </Text>
      </View>
      
      <TouchableOpacity style={styles.refreshButton} onPress={checkEnvironment}>
        <Text style={styles.buttonText}>🔄 Verificar Nuevamente</Text>
      </TouchableOpacity>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: '#f5f5f5'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333'
  },
  statusItem: {
    backgroundColor: 'white',
    padding: 16,
    borderRadius: 8,
    marginBottom: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  statusTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#333',
    marginBottom: 12
  },
  statusDetails: {
    flexDirection: 'row',
    alignItems: 'center'
  },
  statusIcon: {
    fontSize: 24,
    marginRight: 12
  },
  statusInfo: {
    flex: 1
  },
  statusText: {
    fontSize: 16,
    color: '#666',
    marginBottom: 4
  },
  versionText: {
    fontSize: 14,
    color: '#999'
  },
  summaryContainer: {
    backgroundColor: '#e8f5e8',
    padding: 16,
    borderRadius: 8,
    marginVertical: 20
  },
  summaryTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#2e7d32',
    marginBottom: 8
  },
  summaryText: {
    fontSize: 16,
    color: '#2e7d32',
    lineHeight: 24
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20
  },
  loadingText: {
    fontSize: 18,
    color: '#666',
    textAlign: 'center',
    marginBottom: 20
  },
  checkButton: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8
  },
  refreshButton: {
    backgroundColor: '#34C759',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 20
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600'
  }
});

export default EnvironmentStatus;
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Verificación Manual**
Verifica manualmente cada componente del entorno y documenta cualquier problema encontrado.

### **Ejercicio 2: Script de Configuración**
Crea un script personalizado para configurar tu entorno específico.

### **Ejercicio 3: Resolución de Problemas**
Identifica y resuelve al menos 3 problemas comunes de configuración.

### **Ejercicio 4: Documentación del Entorno**
Crea una guía paso a paso para configurar el entorno en tu sistema operativo.

---

## 🎯 Proyecto de la Clase

### **App de Verificación de Entorno**

Crea una aplicación que verifique y muestre el estado de tu entorno de desarrollo React Native.

**Requisitos:**
- Verificación automática de todas las herramientas
- Interfaz visual del estado del entorno
- Solución de problemas comunes
- Guía de configuración paso a paso
- Notificaciones de problemas detectados

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Native Environment Setup](https://reactnative.dev/docs/environment-setup)
- [Android Studio Setup](https://developer.android.com/studio)
- [Xcode Setup](https://developer.apple.com/xcode/)

### **Guías de Instalación:**
- [Node.js Installation](https://nodejs.org/en/download/)
- [React Native CLI](https://github.com/react-native-community/cli)
- [Metro Bundler](https://facebook.github.io/metro/)

### **Herramientas de Verificación:**
- [React Native Doctor](https://github.com/react-native-community/cli#doctor)
- [Environment Checker](https://github.com/react-native-community/cli#environment-info)

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Configuración completa** del entorno de desarrollo
2. **Instalación de herramientas** necesarias
3. **Configuración de Android** y iOS
4. **Verificación automática** del entorno
5. **Resolución de problemas** comunes

### **🚀 Próximos Pasos:**
- Crear tu primer proyecto React Native
- Aprender sobre Metro bundler
- Entender la estructura de un proyecto

### **💡 Conceptos Clave:**
- **Entorno de Desarrollo**: Conjunto de herramientas necesarias
- **Variables de Entorno**: Configuración del sistema
- **SDK**: Software Development Kit
- **CLI**: Command Line Interface
- **Bundler**: Empaquetador de código

---

**🎯 Objetivo**: Tener un entorno de desarrollo completamente configurado y funcional para React Native.

**💡 Consejo**: Dedica tiempo a configurar correctamente tu entorno. Una configuración adecuada te ahorrará muchos problemas en el futuro.
