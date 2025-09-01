# 📚 Clase 4: Primer Proyecto

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 3: Configuración del Entorno](clase_3_configuracion_entorno.md)
- **➡️ Siguiente**: [Clase 5: Metro Bundler](clase_5_metro_bundler.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Crear tu primer proyecto** React Native desde cero
2. **Entender la estructura** de un proyecto React Native
3. **Ejecutar la aplicación** en simulador/emulador
4. **Modificar el código** y ver cambios en tiempo real
5. **Navegar por los archivos** principales del proyecto

---

## 📚 Contenido Teórico

### **¿Qué es un Proyecto React Native?**

Un **proyecto React Native** es una aplicación móvil completa que incluye:

#### **Estructura del Proyecto:**
- **Código JavaScript/TypeScript**: Lógica de la aplicación
- **Configuración nativa**: Archivos específicos de iOS y Android
- **Dependencias**: Librerías y herramientas necesarias
- **Metro bundler**: Sistema de empaquetado y hot reloading

#### **Tipos de Proyectos:**
1. **Proyecto Vanilla**: React Native CLI puro
2. **Proyecto Expo**: Framework simplificado para desarrollo rápido
3. **Proyecto Template**: Plantillas predefinidas con funcionalidades

---

## 💻 Implementación Práctica

### **1. Creación del Primer Proyecto**

```bash:scripts/create-first-project.sh
#!/bin/bash

# Script para crear tu primer proyecto React Native

echo "🚀 Creando tu primer proyecto React Native..."

# Verificar que React Native CLI esté disponible
if ! command -v npx &> /dev/null; then
    echo "❌ npx no está disponible. Instala Node.js primero."
    exit 1
fi

# Verificar React Native CLI
if ! npx react-native --version &> /dev/null; then
    echo "📥 Instalando React Native CLI..."
    npm install -g @react-native-community/cli
fi

# Solicitar nombre del proyecto
read -p "📱 Nombre de tu proyecto (sin espacios): " PROJECT_NAME

if [[ -z "$PROJECT_NAME" ]]; then
    PROJECT_NAME="MiPrimeraApp"
    echo "⚠️  Usando nombre por defecto: $PROJECT_NAME"
fi

# Verificar que el nombre sea válido
if [[ ! "$PROJECT_NAME" =~ ^[a-zA-Z][a-zA-Z0-9]*$ ]]; then
    echo "❌ Nombre inválido. Solo letras y números, comenzando con letra."
    exit 1
fi

# Verificar que el directorio no exista
if [[ -d "$PROJECT_NAME" ]]; then
    echo "❌ El directorio $PROJECT_NAME ya existe. Elige otro nombre."
    exit 1
fi

echo "📁 Creando proyecto: $PROJECT_NAME"

# Crear el proyecto
echo "⏳ Esto puede tomar varios minutos..."
npx react-native init "$PROJECT_NAME" --template react-native-template-typescript

if [[ $? -eq 0 ]]; then
    echo "✅ Proyecto creado exitosamente!"
    
    # Navegar al proyecto
    cd "$PROJECT_NAME"
    
    # Mostrar estructura del proyecto
    echo ""
    echo "📋 Estructura del proyecto creado:"
    tree -L 3 -I 'node_modules' || ls -la
    
    # Mostrar comandos útiles
    echo ""
    echo "🎯 Comandos útiles:"
    echo "   📱 Ejecutar en Android: npx react-native run-android"
    echo "   🍎 Ejecutar en iOS: npx react-native run-ios"
    echo "   📦 Instalar dependencias: npm install"
    echo "   🧹 Limpiar cache: npx react-native start --reset-cache"
    echo "   📚 Ver documentación: npx react-native --help"
    
    # Preguntar si ejecutar el proyecto
    echo ""
    read -p "¿Quieres ejecutar el proyecto ahora? (y/n): " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        echo "🚀 Iniciando Metro bundler..."
        npx react-native start
    fi
    
else
    echo "❌ Error al crear el proyecto"
    exit 1
fi
```

### **2. Estructura de un Proyecto React Native**

```javascript:src/examples/ProjectStructure.js
// Ejemplo de la estructura típica de un proyecto React Native

/*
📁 MiPrimeraApp/
├── 📁 android/                 # Código nativo de Android
│   ├── 📁 app/                 # Aplicación principal
│   ├── 📁 gradle/              # Configuración de Gradle
│   ├── build.gradle            # Configuración del proyecto
│   └── settings.gradle         # Configuración de módulos
├── 📁 ios/                     # Código nativo de iOS
│   ├── 📁 MiPrimeraApp/        # Aplicación principal
│   ├── 📁 MiPrimeraApp.xcodeproj/  # Proyecto Xcode
│   └── 📁 MiPrimeraApp.xcworkspace/ # Workspace Xcode
├── 📁 src/                     # Código fuente de la aplicación
│   ├── 📁 components/          # Componentes reutilizables
│   ├── 📁 screens/             # Pantallas de la aplicación
│   ├── 📁 navigation/          # Configuración de navegación
│   ├── 📁 services/            # Servicios y APIs
│   ├── 📁 hooks/               # Hooks personalizados
│   ├── 📁 utils/               # Utilidades y helpers
│   └── 📁 types/               # Definiciones de TypeScript
├── 📁 __tests__/               # Tests de la aplicación
├── 📄 .eslintrc.js             # Configuración de ESLint
├── 📄 .prettierrc.js           # Configuración de Prettier
├── 📄 .watchmanconfig          # Configuración de Watchman
├── 📄 App.tsx                  # Componente principal
├── 📄 index.js                 # Punto de entrada
├── 📄 metro.config.js          # Configuración de Metro
├── 📄 package.json             # Dependencias y scripts
├── 📄 react-native.config.js   # Configuración de React Native
└── 📄 tsconfig.json            # Configuración de TypeScript
*/

// Componente principal de la aplicación
import React from 'react';
import {
  SafeAreaView,
  ScrollView,
  StatusBar,
  StyleSheet,
  Text,
  useColorScheme,
  View,
} from 'react-native';

import {
  Colors,
  DebugInstructions,
  Header,
  LearnMoreLinks,
  ReloadInstructions,
} from 'react-native/Libraries/NewAppScreen';

function Section({children, title}: {children: React.ReactNode; title: string}) {
  const isDarkMode = useColorScheme() === 'dark';
  return (
    <View style={styles.sectionContainer}>
      <Text
        style={[
          styles.sectionTitle,
          {
            color: isDarkMode ? Colors.white : Colors.black,
          },
        ]}>
        {title}
      </Text>
      <Text
        style={[
          styles.sectionDescription,
          {
            color: isDarkMode ? Colors.light : Colors.dark,
          },
        ]}>
        {children}
      </Text>
    </View>
  );
}

function App(): JSX.Element {
  const isDarkMode = useColorScheme() === 'dark';

  const backgroundStyle = {
    backgroundColor: isDarkMode ? Colors.darker : Colors.lighter,
  };

  return (
    <SafeAreaView style={backgroundStyle}>
      <StatusBar
        barStyle={isDarkMode ? 'light-content' : 'dark-content'}
        backgroundColor={backgroundStyle.backgroundColor}
      />
      <ScrollView
        contentInsetAdjustmentBehavior="automatic"
        style={backgroundStyle}>
        <Header />
        <View
          style={{
            backgroundColor: isDarkMode ? Colors.black : Colors.white,
          }}>
          <Section title="¡Bienvenido a React Native!">
            Esta es tu primera aplicación React Native. 
            Edita <Text style={styles.highlight}>App.tsx</Text> para cambiar esta pantalla.
          </Section>
          <Section title="Ver Cambios">
            <ReloadInstructions />
          </Section>
          <Section title="Debug">
            <DebugInstructions />
          </Section>
          <Section title="Aprende Más">
            Lee la documentación para descubrir qué hacer a continuación.
          </Section>
          <LearnMoreLinks />
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  sectionContainer: {
    marginTop: 32,
    paddingHorizontal: 24,
  },
  sectionTitle: {
    fontSize: 24,
    fontWeight: '600',
  },
  sectionDescription: {
    marginTop: 8,
    fontSize: 18,
    fontWeight: '400',
  },
  highlight: {
    fontWeight: '700',
  },
});

export default App;
```

### **3. Script de Configuración del Proyecto**

```javascript:src/scripts/setupProject.js
// Script para configurar un proyecto React Native recién creado
import { execSync } from 'child_process';
import { readFileSync, writeFileSync, existsSync } from 'fs';
import { join } from 'path';

class ProjectSetup {
  constructor(projectPath) {
    this.projectPath = projectPath;
    this.packageJsonPath = join(projectPath, 'package.json');
    this.eslintConfigPath = join(projectPath, '.eslintrc.js');
    this.prettierConfigPath = join(projectPath, '.prettierrc.js');
  }

  // Configurar ESLint
  setupESLint() {
    console.log('🔧 Configurando ESLint...');
    
    const eslintConfig = `module.exports = {
  root: true,
  extends: [
    '@react-native',
    '@react-native-community',
    'prettier',
  ],
  rules: {
    'prettier/prettier': 'error',
    'no-console': 'warn',
    'no-unused-vars': 'warn',
    'react-native/no-inline-styles': 'warn',
    'react-native/no-unused-styles': 'warn',
    'react-native/split-platform-components': 'warn',
  },
  plugins: ['prettier'],
};`;

    writeFileSync(this.eslintConfigPath, eslintConfig);
    console.log('✅ ESLint configurado');
  }

  // Configurar Prettier
  setupPrettier() {
    console.log('🎨 Configurando Prettier...');
    
    const prettierConfig = `module.exports = {
  arrowParens: 'avoid',
  bracketSameLine: true,
  bracketSpacing: false,
  singleQuote: true,
  trailingComma: 'all',
  tabWidth: 2,
  semi: true,
  printWidth: 80,
};`;

    writeFileSync(this.prettierConfigPath, prettierConfig);
    console.log('✅ Prettier configurado');
  }

  // Instalar dependencias de desarrollo
  installDevDependencies() {
    console.log('📦 Instalando dependencias de desarrollo...');
    
    const devDependencies = [
      '@types/react',
      '@types/react-native',
      '@typescript-eslint/eslint-plugin',
      '@typescript-eslint/parser',
      'eslint-config-prettier',
      'eslint-plugin-prettier',
      'prettier',
    ];

    try {
      execSync(`cd "${this.projectPath}" && npm install --save-dev ${devDependencies.join(' ')}`, { stdio: 'inherit' });
      console.log('✅ Dependencias de desarrollo instaladas');
    } catch (error) {
      console.error('❌ Error al instalar dependencias:', error.message);
    }
  }

  // Crear estructura de directorios
  createDirectoryStructure() {
    console.log('📁 Creando estructura de directorios...');
    
    const directories = [
      'src/components',
      'src/screens',
      'src/navigation',
      'src/services',
      'src/hooks',
      'src/utils',
      'src/types',
      'src/assets',
      'src/assets/images',
      'src/assets/icons',
      '__tests__',
    ];

    directories.forEach(dir => {
      const fullPath = join(this.projectPath, dir);
      if (!existsSync(fullPath)) {
        execSync(`mkdir -p "${fullPath}"`);
        console.log(`   📁 Creado: ${dir}`);
      }
    });

    console.log('✅ Estructura de directorios creada');
  }

  // Crear archivos de ejemplo
  createExampleFiles() {
    console.log('📄 Creando archivos de ejemplo...');
    
    // Componente de ejemplo
    const exampleComponent = `import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

interface ExampleComponentProps {
  title: string;
  description?: string;
}

export const ExampleComponent: React.FC<ExampleComponentProps> = ({
  title,
  description,
}) => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>{title}</Text>
      {description && <Text style={styles.description}>{description}</Text>}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
    margin: 8,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 8,
  },
  description: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
});

export default ExampleComponent;
`;

    const componentPath = join(this.projectPath, 'src/components/ExampleComponent.tsx');
    writeFileSync(componentPath, exampleComponent);
    console.log('   📄 Creado: src/components/ExampleComponent.tsx');

    // Hook de ejemplo
    const exampleHook = `import { useState, useEffect } from 'react';

export const useExample = (initialValue: string = '') => {
  const [value, setValue] = useState(initialValue);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    // Lógica del hook aquí
    console.log('Hook example initialized with:', initialValue);
  }, [initialValue]);

  const updateValue = (newValue: string) => {
    setIsLoading(true);
    // Simular operación asíncrona
    setTimeout(() => {
      setValue(newValue);
      setIsLoading(false);
    }, 500);
  };

  return {
    value,
    isLoading,
    updateValue,
  };
};
`;

    const hookPath = join(this.projectPath, 'src/hooks/useExample.ts');
    writeFileSync(hookPath, exampleHook);
    console.log('   📄 Creado: src/hooks/useExample.ts');

    // Tipo de ejemplo
    const exampleTypes = `// Tipos básicos para la aplicación

export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

export interface AppConfig {
  theme: 'light' | 'dark';
  language: 'es' | 'en';
  notifications: boolean;
}

export type ApiResponse<T> = {
  data: T;
  success: boolean;
  message?: string;
  error?: string;
};
`;

    const typesPath = join(this.projectPath, 'src/types/index.ts');
    writeFileSync(typesPath, exampleTypes);
    console.log('   📄 Creado: src/types/index.ts');
  }

  // Configurar scripts de package.json
  updatePackageScripts() {
    console.log('📝 Actualizando scripts de package.json...');
    
    try {
      const packageJson = JSON.parse(readFileSync(this.packageJsonPath, 'utf8'));
      
      packageJson.scripts = {
        ...packageJson.scripts,
        'lint': 'eslint . --ext .js,.jsx,.ts,.tsx',
        'lint:fix': 'eslint . --ext .js,.jsx,.ts,.tsx --fix',
        'format': 'prettier --write "**/*.{js,jsx,ts,tsx,json,md}"',
        'type-check': 'tsc --noEmit',
        'test': 'jest',
        'test:watch': 'jest --watch',
        'clean': 'npx react-native start --reset-cache',
        'android:clean': 'cd android && ./gradlew clean && cd ..',
        'ios:clean': 'cd ios && xcodebuild clean && cd ..',
      };

      writeFileSync(this.packageJsonPath, JSON.stringify(packageJson, null, 2));
      console.log('✅ Scripts de package.json actualizados');
    } catch (error) {
      console.error('❌ Error al actualizar package.json:', error.message);
    }
  }

  // Ejecutar configuración completa
  runSetup() {
    console.log('🚀 Iniciando configuración del proyecto...\n');
    
    this.setupESLint();
    this.setupPrettier();
    this.installDevDependencies();
    this.createDirectoryStructure();
    this.createExampleFiles();
    this.updatePackageScripts();
    
    console.log('\n🎉 ¡Configuración del proyecto completada!');
    console.log('\n📋 Próximos pasos:');
    console.log('   1. cd ' + this.projectPath);
    console.log('   2. npm run lint');
    console.log('   3. npm run format');
    console.log('   4. npx react-native run-android (o run-ios)');
  }
}

// Función para ejecutar la configuración
export const setupProject = (projectPath) => {
  const setup = new ProjectSetup(projectPath);
  setup.runSetup();
};

// Ejecutar si se llama directamente
if (require.main === module) {
  const projectPath = process.argv[2];
  if (!projectPath) {
    console.error('❌ Uso: node setupProject.js <ruta-del-proyecto>');
    process.exit(1);
  }
  setupProject(projectPath);
}
```

---

## 🧪 Casos de Uso Prácticos

### **1. Crear y Configurar un Proyecto**

```bash
# 1. Crear el proyecto
npx react-native init MiPrimeraApp --template react-native-template-typescript

# 2. Navegar al proyecto
cd MiPrimeraApp

# 3. Configurar el proyecto
node setupProject.js .

# 4. Ejecutar la aplicación
npx react-native run-android
# o
npx react-native run-ios
```

### **2. Modificar la Aplicación**

```javascript:src/screens/HomeScreen.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  SafeAreaView,
} from 'react-native';
import { ExampleComponent } from '../components/ExampleComponent';
import { useExample } from '../hooks/useExample';

const HomeScreen = () => {
  const [counter, setCounter] = useState(0);
  const { value, isLoading, updateValue } = useExample('Valor inicial');

  const incrementCounter = () => {
    setCounter(prev => prev + 1);
  };

  const resetCounter = () => {
    setCounter(0);
  };

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.content}>
        <Text style={styles.title}>¡Mi Primera App React Native!</Text>
        
        <ExampleComponent 
          title="Contador Interactivo"
          description="Toca los botones para interactuar con la aplicación"
        />
        
        <View style={styles.counterContainer}>
          <Text style={styles.counterText}>Contador: {counter}</Text>
          
          <View style={styles.buttonContainer}>
            <TouchableOpacity style={styles.button} onPress={incrementCounter}>
              <Text style={styles.buttonText}>➕ Incrementar</Text>
            </TouchableOpacity>
            
            <TouchableOpacity style={styles.resetButton} onPress={resetCounter}>
              <Text style={styles.buttonText}>🔄 Reiniciar</Text>
            </TouchableOpacity>
          </View>
        </View>
        
        <ExampleComponent 
          title="Hook Personalizado"
          description={`Valor del hook: ${value} ${isLoading ? '(cargando...)' : ''}`}
        />
        
        <TouchableOpacity 
          style={styles.updateButton} 
          onPress={() => updateValue('Nuevo valor ' + Date.now())}
        >
          <Text style={styles.buttonText}>🔄 Actualizar Hook</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f0f0f0',
  },
  content: {
    flex: 1,
    padding: 20,
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    textAlign: 'center',
    marginBottom: 30,
  },
  counterContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    marginVertical: 20,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 4,
  },
  counterText: {
    fontSize: 20,
    fontWeight: '600',
    color: '#333',
    marginBottom: 20,
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 12,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#FF3B30',
    padding: 12,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  updateButton: {
    backgroundColor: '#34C759',
    padding: 16,
    borderRadius: 8,
    marginTop: 20,
    minWidth: 200,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
});

export default HomeScreen;
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Crear Proyecto Básico**
Crea un proyecto React Native y modifica el texto de bienvenida.

### **Ejercicio 2: Agregar Funcionalidad**
Implementa un contador simple con botones de incrementar y decrementar.

### **Ejercicio 3: Personalizar Estilos**
Cambia los colores, fuentes y layout de la aplicación por defecto.

### **Ejercicio 4: Crear Componente Personalizado**
Crea un componente de tarjeta personalizable y úsalo en la aplicación.

---

## 🎯 Proyecto de la Clase

### **App de Bienvenida Personalizada**

Crea una aplicación que muestre información personalizada y tenga funcionalidades interactivas básicas.

**Requisitos:**
- Pantalla de bienvenida personalizada
- Componente de contador interactivo
- Hook personalizado funcional
- Estilos atractivos y responsive
- Navegación entre pantallas básica

---

## 📚 Recursos Adicionales

### **Documentación:**
- [React Native Getting Started](https://reactnative.dev/docs/getting-started)
- [Project Structure](https://reactnative.dev/docs/project-structure)
- [TypeScript Template](https://github.com/react-native-community/cli#typescript-template)

### **Plantillas:**
- [React Native Template TypeScript](https://github.com/react-native-community/cli-templates)
- [Expo Template](https://docs.expo.dev/guides/typescript/)

### **Herramientas:**
- [React Native CLI](https://github.com/react-native-community/cli)
- [Metro Bundler](https://facebook.github.io/metro/)

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Crear proyectos** React Native desde cero
2. **Estructura de archivos** y directorios
3. **Configuración automática** del proyecto
4. **Modificación del código** y hot reloading
5. **Ejecución en simuladores** y emuladores

### **🚀 Próximos Pasos:**
- Entender Metro bundler
- Aprender sobre componentes nativos
- Crear interfaces más complejas

### **💡 Conceptos Clave:**
- **Proyecto**: Estructura completa de la aplicación
- **CLI**: Herramienta de línea de comandos
- **Template**: Plantilla predefinida para el proyecto
- **Hot Reloading**: Recarga automática de cambios
- **Simulador/Emulador**: Entorno de prueba virtual

---

**🎯 Objetivo**: Crear y ejecutar tu primera aplicación React Native completamente funcional.

**💡 Consejo**: Experimenta con los cambios en tiempo real. El hot reloading te permitirá ver los resultados inmediatamente.
