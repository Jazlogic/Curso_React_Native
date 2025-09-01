# 📚 Clase 1: AsyncStorage - Almacenamiento Clave-Valor

## 🧭 Navegación del Módulo

- **⬅️ Anterior**: [README del Módulo](README.md)
- **➡️ Siguiente**: [Clase 2: SQLite - Base de Datos Relacional](clase_2_sqlite.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

- Comprender qué es AsyncStorage y cuándo usarlo
- Implementar operaciones básicas de almacenamiento (CRUD)
- Crear un servicio de almacenamiento reutilizable
- Manejar errores y casos edge
- Optimizar el rendimiento del almacenamiento

---

## 📖 Contenido Teórico

### 1. ¿Qué es AsyncStorage?

AsyncStorage es una solución de almacenamiento local para React Native que permite guardar datos en formato clave-valor de manera asíncrona. Es ideal para:

- **Configuraciones de usuario** (tema, idioma, preferencias)
- **Datos de sesión** (tokens, información del usuario)
- **Cache de datos** (respuestas de API, imágenes)
- **Configuraciones de la app** (primer uso, tutoriales)

**Características principales:**
- **Asíncrono**: No bloquea el hilo principal
- **Persistente**: Los datos sobreviven a reinicios de la app
- **Simple**: API fácil de usar con métodos básicos
- **Universal**: Funciona en iOS y Android
- **Límites**: Tiene restricciones de tamaño en iOS

### 2. Instalación y Configuración

#### **Instalación del Paquete:**
```bash
npm install @react-native-async-storage/async-storage
# o
yarn add @react-native-async-storage/async-storage
```

#### **Importación en el Código:**
```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';
```

---

## 🛠️ Implementación Práctica

### 1. Servicio Básico de Almacenamiento

Vamos a crear un servicio completo que maneje todas las operaciones de AsyncStorage:

```javascript
// utils/storage.js
import AsyncStorage from '@react-native-async-storage/async-storage';

// Exportamos un objeto con métodos para manejar el almacenamiento
export const storage = {
  // Método para guardar datos
  async setItem(key, value) {
    try {
      // JSON.stringify convierte cualquier valor a string
      // Esto es necesario porque AsyncStorage solo acepta strings
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      // Capturamos cualquier error que pueda ocurrir
      console.error('Error saving data:', error);
      // Podríamos lanzar el error para manejarlo en el componente
      throw error;
    }
  },

  // Método para obtener datos
  async getItem(key) {
    try {
      // Obtenemos el valor almacenado
      const value = await AsyncStorage.getItem(key);
      
      // Si no hay valor, retornamos null
      if (value === null) {
        return null;
      }
      
      // Parseamos el JSON de vuelta a su tipo original
      return JSON.parse(value);
    } catch (error) {
      console.error('Error reading data:', error);
      return null;
    }
  },

  // Método para eliminar un elemento específico
  async removeItem(key) {
    try {
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Error removing data:', error);
      throw error;
    }
  },

  // Método para limpiar todo el almacenamiento
  async clear() {
    try {
      await AsyncStorage.clear();
    } catch (error) {
      console.error('Error clearing data:', error);
      throw error;
    }
  },

  // Método para obtener todas las claves
  async getAllKeys() {
    try {
      return await AsyncStorage.getAllKeys();
    } catch (error) {
      console.error('Error getting keys:', error);
      return [];
    }
  },

  // Método para obtener múltiples valores
  async multiGet(keys) {
    try {
      const keyValuePairs = await AsyncStorage.multiGet(keys);
      // Convertimos el array de pares [key, value] a un objeto
      return keyValuePairs.reduce((result, [key, value]) => {
        result[key] = value ? JSON.parse(value) : null;
        return result;
      }, {});
    } catch (error) {
      console.error('Error getting multiple items:', error);
      return {};
    }
  },

  // Método para guardar múltiples valores
  async multiSet(keyValuePairs) {
    try {
      // Convertimos el objeto a pares [key, value] con JSON.stringify
      const pairs = Object.entries(keyValuePairs).map(([key, value]) => [
        key,
        JSON.stringify(value)
      ]);
      await AsyncStorage.multiSet(pairs);
    } catch (error) {
      console.error('Error setting multiple items:', error);
      throw error;
    }
  }
};
```

### 2. Explicación Línea por Línea del Servicio

#### **Importación:**
```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';
```
- **Explicación**: Importamos la librería AsyncStorage que nos permite acceder al almacenamiento nativo del dispositivo
- **¿Por qué es necesario?**: Sin esta importación, no podemos usar las funciones de almacenamiento

#### **Estructura del Objeto:**
```javascript
export const storage = {
  // métodos aquí...
};
```
- **Explicación**: Creamos un objeto que contiene todos nuestros métodos de almacenamiento
- **¿Por qué exportamos?**: Para poder importarlo en otros archivos y reutilizarlo

#### **Método setItem:**
```javascript
async setItem(key, value) {
  try {
    await AsyncStorage.setItem(key, JSON.stringify(value));
  } catch (error) {
    console.error('Error saving data:', error);
    throw error;
  }
}
```
- **`async`**: Hace que la función sea asíncrona, permitiendo usar `await`
- **`key`**: La clave (string) bajo la cual se guardará el valor
- **`value`**: El valor que queremos guardar (puede ser cualquier tipo)
- **`JSON.stringify(value)`**: Convierte el valor a string (AsyncStorage solo acepta strings)
- **`try-catch`**: Captura errores que puedan ocurrir durante el guardado
- **`throw error`**: Re-lanza el error para que el componente que lo llamó pueda manejarlo

#### **Método getItem:**
```javascript
async getItem(key) {
  try {
    const value = await AsyncStorage.getItem(key);
    
    if (value === null) {
      return null;
    }
    
    return JSON.parse(value);
  } catch (error) {
    console.error('Error reading data:', error);
    return null;
  }
}
```
- **`await AsyncStorage.getItem(key)`**: Obtiene el valor almacenado bajo la clave especificada
- **`value === null`**: Verifica si no hay valor almacenado
- **`JSON.parse(value)`**: Convierte el string de vuelta a su tipo original
- **`return null`**: Retorna null en caso de error o si no hay valor

#### **Método removeItem:**
```javascript
async removeItem(key) {
  try {
    await AsyncStorage.removeItem(key);
  } catch (error) {
    console.error('Error removing data:', error);
    throw error;
  }
}
```
- **`AsyncStorage.removeItem(key)`**: Elimina el elemento almacenado bajo la clave especificada
- **Manejo de errores**: Si falla la eliminación, lanzamos el error

#### **Método clear:**
```javascript
async clear() {
  try {
    await AsyncStorage.clear();
  } catch (error) {
    console.error('Error clearing data:', error);
    throw error;
  }
}
```
- **`AsyncStorage.clear()`**: Elimina TODOS los datos almacenados
- **⚠️ Precaución**: Este método es destructivo, usar con cuidado

#### **Método getAllKeys:**
```javascript
async getAllKeys() {
  try {
    return await AsyncStorage.getAllKeys();
  } catch (error) {
    console.error('Error getting keys:', error);
    return [];
  }
}
```
- **`AsyncStorage.getAllKeys()`**: Obtiene un array con todas las claves almacenadas
- **Retorno en error**: Si falla, retornamos un array vacío

#### **Método multiGet:**
```javascript
async multiGet(keys) {
  try {
    const keyValuePairs = await AsyncStorage.multiGet(keys);
    return keyValuePairs.reduce((result, [key, value]) => {
      result[key] = value ? JSON.parse(value) : null;
      return result;
    }, {});
  } catch (error) {
    console.error('Error getting multiple items:', error);
    return {};
  }
}
```
- **`AsyncStorage.multiGet(keys)`**: Obtiene múltiples valores en una sola operación (más eficiente)
- **`reduce`**: Convierte el array de pares [key, value] a un objeto
- **`value ? JSON.parse(value) : null`**: Si hay valor, lo parsea; si no, asigna null

#### **Método multiSet:**
```javascript
async multiSet(keyValuePairs) {
  try {
    const pairs = Object.entries(keyValuePairs).map(([key, value]) => [
      key,
      JSON.stringify(value)
    ]);
    await AsyncStorage.multiSet(pairs);
  } catch (error) {
    console.error('Error setting multiple items:', error);
    throw error;
  }
}
```
- **`Object.entries(keyValuePairs)`**: Convierte el objeto en un array de pares [key, value]
- **`map`**: Convierte cada valor a string usando JSON.stringify
- **`AsyncStorage.multiSet(pairs)`**: Guarda múltiples valores en una sola operación

---

## 💡 Casos de Uso Prácticos

### 1. Configuración de Usuario

```javascript
// services/configService.js
import { storage } from '../utils/storage';

export const configService = {
  // Guardar tema de la aplicación
  async saveTheme(theme) {
    // theme puede ser 'light', 'dark', o 'auto'
    await storage.setItem('theme', theme);
  },

  // Obtener tema guardado
  async getTheme() {
    // Si no hay tema guardado, retornamos 'light' por defecto
    return await storage.getItem('theme') || 'light';
  },

  // Guardar idioma
  async saveLanguage(language) {
    await storage.setItem('language', language);
  },

  // Obtener idioma
  async getLanguage() {
    return await storage.getItem('language') || 'es';
  },

  // Guardar preferencias completas del usuario
  async saveUserPreferences(preferences) {
    // preferences es un objeto con múltiples configuraciones
    await storage.setItem('userPreferences', preferences);
  },

  // Obtener preferencias del usuario
  async getUserPreferences() {
    return await storage.getItem('userPreferences') || {};
  }
};
```

### 2. Gestión de Sesión

```javascript
// services/sessionService.js
import { storage } from '../utils/storage';

export const sessionService = {
  // Guardar token de autenticación
  async saveAuthToken(token) {
    await storage.setItem('authToken', token);
  },

  // Obtener token de autenticación
  async getAuthToken() {
    return await storage.getItem('authToken');
  },

  // Eliminar token (logout)
  async clearAuthToken() {
    await storage.removeItem('authToken');
  },

  // Verificar si el usuario está autenticado
  async isAuthenticated() {
    const token = await this.getAuthToken();
    return token !== null;
  },

  // Guardar información del usuario
  async saveUserInfo(userInfo) {
    await storage.setItem('userInfo', userInfo);
  },

  // Obtener información del usuario
  async getUserInfo() {
    return await storage.getItem('userInfo');
  }
};
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Implementación Básica**
1. Crea el archivo `utils/storage.js` con el servicio completo
2. Implementa los métodos `setItem`, `getItem`, `removeItem` y `clear`
3. Prueba cada método con diferentes tipos de datos
4. Maneja los errores apropiadamente

### **Ejercicio 2: Servicio de Configuración**
1. Crea `services/configService.js` usando el storage service
2. Implementa métodos para guardar/obtener tema, idioma y preferencias
3. Crea un componente que use este servicio
4. Prueba la persistencia de datos

### **Ejercicio 3: Gestión de Sesión**
1. Crea `services/sessionService.js` para manejar autenticación
2. Implementa login/logout simulado
3. Crea un componente que muestre el estado de autenticación
4. Prueba la persistencia del token

### **Ejercicio 4: Cache de Datos**
1. Crea un servicio de cache que use AsyncStorage
2. Implementa TTL (Time To Live) para los datos
3. Crea métodos para limpiar cache expirado
4. Prueba con datos simulados de API

### **Ejercicio 5: Manejo de Errores Avanzado**
1. Implementa retry logic para operaciones fallidas
2. Crea un sistema de logging para errores de storage
3. Implementa fallback a valores por defecto
4. Prueba con diferentes escenarios de error

---

## 🎯 Proyecto de la Clase

### **App de Configuración Personalizable**

Crea una aplicación que permita al usuario:
1. **Cambiar tema** (claro/oscuro)
2. **Seleccionar idioma** (español/inglés)
3. **Configurar notificaciones** (on/off, horarios)
4. **Personalizar colores** de la interfaz
5. **Guardar preferencias** de accesibilidad

**Estructura del Proyecto:**
```
src/
├── utils/
│   └── storage.js          # Servicio de almacenamiento
├── services/
│   ├── configService.js    # Servicio de configuración
│   └── sessionService.js   # Servicio de sesión
├── components/
│   ├── ThemeToggle.js      # Selector de tema
│   ├── LanguageSelector.js # Selector de idioma
│   └── SettingsForm.js     # Formulario de configuración
└── screens/
    └── SettingsScreen.js   # Pantalla de configuración
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [AsyncStorage Documentation](https://react-native-async-storage.github.io/async-storage/)
- [React Native Storage Guide](https://reactnative.dev/docs/asyncstorage)

### **Mejores Prácticas:**
- **Tamaño de datos**: Evita guardar archivos grandes
- **Frecuencia de escritura**: No escribas en cada render
- **Manejo de errores**: Siempre implementa try-catch
- **Validación**: Verifica los datos antes de guardarlos

### **Alternativas:**
- **MMKV**: Para datos que requieren alto rendimiento
- **WatermelonDB**: Para bases de datos complejas
- **Realm**: Para datos estructurados y relaciones

---

## 🔍 Resumen de la Clase

### **Conceptos Clave:**
- ✅ AsyncStorage es ideal para datos simples clave-valor
- ✅ Siempre usar try-catch para manejar errores
- ✅ JSON.stringify/parse para convertir datos
- ✅ Los métodos son asíncronos (async/await)
- ✅ Crear servicios reutilizables para el almacenamiento

### **Próximos Pasos:**
- 🔄 Completar todos los ejercicios de esta clase
- 🔄 Implementar el proyecto de configuración
- 🔄 Practicar con diferentes tipos de datos
- 🔄 Prepararse para la siguiente clase sobre SQLite

---

## 🚀 Siguiente Clase

En la **Clase 2: SQLite - Base de Datos Relacional**, aprenderás:
- Cómo configurar SQLite en React Native
- Creación y gestión de tablas
- Operaciones CRUD complejas
- Relaciones entre tablas
- Migraciones y versionado de base de datos

---

**¡Excelente trabajo en esta primera clase! 🎉**

**Recuerda**: AsyncStorage es la base del almacenamiento local. Domina estos conceptos antes de avanzar a bases de datos más complejas.
