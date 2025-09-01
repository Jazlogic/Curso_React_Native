# 🎯 Clase 1: Fundamentos de Expo

## 🎯 Objetivos de la Clase
- Entender qué es Expo y sus ventajas para el desarrollo
- Comprender las limitaciones y cuándo usar Expo
- Configurar un proyecto Expo desde cero
- Explorar la estructura de un proyecto Expo

---

## 📚 Contenido Teórico

### 1. ¿Qué es Expo?

#### **Definición y Propósito**
**Expo** es una plataforma completa que simplifica el desarrollo de aplicaciones React Native:

- **Framework y SDK**: Conjunto de herramientas y APIs pre-construidas
- **Servicios en la nube**: Backend y herramientas de desarrollo
- **Workflow simplificado**: Desarrollo, testing y publicación más sencillos
- **Ecosistema unificado**: Herramientas integradas para todo el ciclo de desarrollo

#### **Ventajas de Expo:**
- **Desarrollo rápido**: Configuración mínima, inicio inmediato
- **APIs nativas**: Acceso fácil a funcionalidades del dispositivo
- **Herramientas integradas**: CLI, DevTools, testing en dispositivos
- **Servicios en la nube**: Notificaciones, analytics, actualizaciones OTA
- **Comunidad activa**: Soporte y recursos abundantes
- **Cross-platform**: Funciona en iOS y Android sin configuración adicional

#### **Limitaciones de Expo:**
- **Dependencia de servicios**: Algunas funcionalidades requieren servicios de Expo
- **Tamaño del bundle**: Puede ser más grande que React Native puro
- **Personalización nativa**: Limitada en el managed workflow
- **Versiones del SDK**: Dependencia de las actualizaciones de Expo

### 2. Tipos de Workflow en Expo

#### **Managed Workflow**
```bash
# Crear proyecto con managed workflow
npx create-expo-app MiApp
cd MiApp
npx expo start
```

**Características:**
- **Configuración automática**: Expo maneja toda la configuración
- **APIs pre-construidas**: SDK completo disponible
- **Fácil publicación**: Expo Go y EAS Build
- **Limitaciones**: No se puede modificar código nativo

#### **Bare Workflow**
```bash
# Crear proyecto con bare workflow
npx create-expo-app MiApp --template bare-minimum
cd MiApp
npx expo run:ios  # o run:android
```

**Características:**
- **Control total**: Acceso completo al código nativo
- **Flexibilidad**: Puedes usar cualquier librería nativa
- **Configuración manual**: Necesitas configurar Xcode/Android Studio
- **Mantenimiento**: Requiere más conocimiento técnico

### 3. Configuración del Entorno

#### **Instalación de Expo CLI**
```bash
# Instalación global (opcional)
npm install -g @expo/cli

# O usar npx (recomendado)
npx @expo/cli --version
```

#### **Requisitos del Sistema**
```bash
# Verificar Node.js (versión 16 o superior)
node --version

# Verificar npm
npm --version

# Verificar Git
git --version
```

#### **Instalación de Expo Go**
- **iOS**: App Store - "Expo Go"
- **Android**: Google Play Store - "Expo Go"

### 4. Creación del Primer Proyecto

#### **Paso 1: Crear el Proyecto**
```bash
# Crear nuevo proyecto Expo
npx create-expo-app MiPrimeraAppExpo

# Navegar al directorio
cd MiPrimeraAppExpo

# Ver estructura del proyecto
ls -la
```

#### **Paso 2: Estructura del Proyecto**
```
MiPrimeraAppExpo/
├── App.js                 # Punto de entrada principal
├── app.json              # Configuración de Expo
├── package.json          # Dependencias del proyecto
├── babel.config.js       # Configuración de Babel
├── assets/               # Recursos estáticos
│   ├── icon.png         # Icono de la app
│   ├── splash.png       # Pantalla de carga
│   └── adaptive-icon.png # Icono adaptativo
├── node_modules/         # Dependencias instaladas
└── .gitignore           # Archivos ignorados por Git
```

#### **Paso 3: Configuración Básica (app.json)**
```json
{
  "expo": {
    "name": "Mi Primera App Expo",
    "slug": "mi-primera-app-expo",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

### 5. Configuración del Entorno de Desarrollo

#### **Iniciar el Servidor de Desarrollo**
```bash
# Iniciar servidor de desarrollo
npx expo start

# O con alias
npm start
```

#### **Opciones de Inicio**
```bash
# Iniciar con configuración específica
npx expo start --tunnel        # Usar túnel para desarrollo remoto
npx expo start --localhost     # Solo localhost
npx expo start --lan          # Red local
npx expo start --offline      # Modo offline
```

#### **Comandos Útiles**
```bash
# Ver información del proyecto
npx expo info

# Instalar dependencias
npx expo install

# Previsualizar en web
npx expo start --web

# Ejecutar en simulador/emulador
npx expo run:ios
npx expo run:android
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Configuración Básica
Configura un proyecto Expo desde cero:

**Requisitos:**
- Crear proyecto con managed workflow
- Configurar app.json personalizado
- Iniciar servidor de desarrollo
- Conectar con Expo Go

**Implementación:**
```bash
# 1. Crear proyecto
npx create-expo-app MiAppExpo --template blank

# 2. Navegar al directorio
cd MiAppExpo

# 3. Personalizar app.json
# Editar nombre, slug, versión, etc.

# 4. Iniciar desarrollo
npx expo start

# 5. Escanear QR con Expo Go
```

### Ejercicio 2: Exploración de la Estructura
Explora y modifica la estructura del proyecto:

**Requisitos:**
- Crear directorios para componentes, servicios, etc.
- Modificar App.js para mostrar información personalizada
- Agregar assets personalizados
- Configurar navegación básica

**Implementación:**
```javascript
// App.js modificado
import React from 'react';
import { StyleSheet, Text, View, Image } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Image 
        source={require('./assets/icon.png')} 
        style={styles.logo} 
      />
      <Text style={styles.title}>¡Mi Primera App con Expo!</Text>
      <Text style={styles.subtitle}>Desarrollada con React Native + Expo</Text>
      <Text style={styles.version}>Versión 1.0.0</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  logo: {
    width: 100,
    height: 100,
    marginBottom: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'center',
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginBottom: 20,
    textAlign: 'center',
  },
  version: {
    fontSize: 14,
    color: '#999',
  },
});
```

### Ejercicio 3: Configuración de Entorno
Configura diferentes entornos de desarrollo:

**Requisitos:**
- Configurar variables de entorno
- Crear scripts de desarrollo
- Configurar diferentes configuraciones por entorno

**Implementación:**
```bash
# 1. Instalar dotenv
npm install dotenv

# 2. Crear archivos de configuración
touch .env.development
touch .env.production

# 3. Configurar variables
echo "API_URL=https://dev-api.miapp.com" > .env.development
echo "API_URL=https://api.miapp.com" > .env.production

# 4. Crear scripts en package.json
```

```json
{
  "scripts": {
    "start": "expo start",
    "start:dev": "expo start --dev-client",
    "start:prod": "expo start --no-dev",
    "build:android": "eas build --platform android",
    "build:ios": "eas build --platform ios"
  }
}
```

---

## 🔍 Puntos Clave

1. **Expo simplifica** el desarrollo de React Native con herramientas integradas
2. **Managed workflow** es ideal para principiantes y prototipos rápidos
3. **Bare workflow** ofrece control total pero requiere más configuración
4. **Expo CLI** es la herramienta principal para gestionar proyectos
5. **app.json** contiene toda la configuración del proyecto
6. **Expo Go** permite probar apps en dispositivos reales sin compilación

---

## 📖 Recursos Adicionales

- [Expo Documentation](https://docs.expo.dev/)
- [Expo CLI Reference](https://docs.expo.dev/workflow/expo-cli/)
- [Expo Configuration](https://docs.expo.dev/versions/latest/config/app/)
- [Expo Workflows](https://docs.expo.dev/introduction/managed-vs-bare/)

---

## 🎉 ¡Clase Completada!

Has completado exitosamente la **Clase 1: Fundamentos de Expo** 🚀

**Lo que has aprendido:**
- ✅ Qué es Expo y sus ventajas
- ✅ Tipos de workflow disponibles
- ✅ Configuración del entorno
- ✅ Creación de proyectos
- ✅ Estructura básica de un proyecto Expo

**Próximos pasos:**
- Practicar con los ejercicios
- Explorar más opciones de configuración
- Continuar con la siguiente clase sobre Expo SDK y APIs

¡Excelente trabajo! 🎯

---

## 🔗 Navegación del Módulo

### **Clase Siguiente**: [Clase 2: Expo SDK y APIs](clase_2_expo_sdk_apis.md)
### **Volver al Módulo**: [Módulo 8: Expo y Desarrollo Rápido](README.md)
### **Volver al Inicio**: [Índice Completo](../INDICE_COMPLETO.md)

