# Clase 1: Fetch API Básica

## Navegación
- **Anterior**: [Módulo 5: Gestión de Estado Avanzada](./../midLevel_2/README.md)
- **Siguiente**: [Clase 2: Axios y Middleware](./clase_2_axios_middleware.md)
- **Módulo**: [Módulo 6: APIs y Networking](./README.md)
- **Índice**: [Índice Completo](./../INDICE_COMPLETO.md)
- **Navegación Rápida**: [Navegación Rápida](./../NAVEGACION_RAPIDA.md)

## 🎯 Objetivos de la Clase
- Comprender los fundamentos de Fetch API en React Native
- Implementar peticiones HTTP básicas (GET, POST, PUT, DELETE)
- Manejar respuestas y errores de manera efectiva
- Crear servicios reutilizables para APIs
- Implementar manejo de estados de carga y error

## 📚 Contenido Teórico

### 1. ¿Qué es Fetch API?

**Fetch API** es la interfaz nativa de JavaScript para realizar peticiones HTTP. Es una alternativa moderna a XMLHttpRequest que proporciona una API más limpia y basada en Promesas.

#### Características Principales:
- **Nativo**: No requiere librerías externas
- **Basado en Promesas**: Facilita el manejo asíncrono
- **Streaming**: Soporte para respuestas en streaming
- **CORS**: Manejo integrado de Cross-Origin Resource Sharing

### 2. Estructura Básica de una Petición Fetch

```javascript
// Estructura básica de fetch
fetch(url, options)
  .then(response => {
    // Procesar la respuesta
    return response.json();
  })
  .then(data => {
    // Usar los datos
    console.log(data);
  })
  .catch(error => {
    // Manejar errores
    console.error('Error:', error);
  });
```

#### Explicación Línea por Línea:
1. **`fetch(url, options)`**: Función que inicia la petición HTTP
   - `url`: La URL del endpoint de la API
   - `options`: Objeto con configuración (método, headers, body, etc.)

2. **`.then(response => {...})`**: Primera promesa que se resuelve con la respuesta HTTP
   - `response`: Objeto Response con información de la petición

3. **`return response.json()`**: Convierte la respuesta a JSON y retorna una nueva promesa

4. **`.then(data => {...})`**: Segunda promesa que se resuelve con los datos parseados
   - `data`: Los datos de la API ya convertidos a JavaScript

5. **`.catch(error => {...})`**: Captura cualquier error en la cadena de promesas

### 3. Métodos HTTP Básicos

#### GET - Obtener Datos
```javascript
// GET básico
const getUsers = async () => {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const users = await response.json();
    return users;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
};
```

#### POST - Crear Datos
```javascript
// POST con datos
const createUser = async (userData) => {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const newUser = await response.json();
    return newUser;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};
```

#### PUT - Actualizar Datos
```javascript
// PUT para actualizar
const updateUser = async (userId, userData) => {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const updatedUser = await response.json();
    return updatedUser;
  } catch (error) {
    console.error('Error updating user:', error);
    throw error;
  }
};
```

#### DELETE - Eliminar Datos
```javascript
// DELETE para eliminar
const deleteUser = async (userId) => {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      method: 'DELETE',
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return true; // Usuario eliminado exitosamente
  } catch (error) {
    console.error('Error deleting user:', error);
    throw error;
  }
};
```

### 4. Manejo de Headers y Opciones

#### Headers Básicos
```javascript
// Headers comunes en APIs
const headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json',
  'Authorization': `Bearer ${token}`,
  'User-Agent': 'MyApp/1.0',
};
```

#### Opciones Completas de Fetch
```javascript
const fetchOptions = {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`,
  },
  body: JSON.stringify(data),
  mode: 'cors', // cors, no-cors, same-origin
  cache: 'no-cache', // default, no-cache, reload, force-cache
  credentials: 'include', // omit, same-origin, include
  redirect: 'follow', // manual, follow, error
  referrerPolicy: 'no-referrer', // no-referrer, client
};
```

### 5. Manejo de Respuestas

#### Verificación de Estado HTTP
```javascript
const handleResponse = async (response) => {
  // Verificar si la respuesta es exitosa (200-299)
  if (response.ok) {
    return await response.json();
  }
  
  // Manejar diferentes códigos de error
  switch (response.status) {
    case 400:
      throw new Error('Bad Request - Datos inválidos');
    case 401:
      throw new Error('Unauthorized - No autorizado');
    case 403:
      throw new Error('Forbidden - Acceso prohibido');
    case 404:
      throw new Error('Not Found - Recurso no encontrado');
    case 500:
      throw new Error('Internal Server Error - Error del servidor');
    default:
      throw new Error(`HTTP error! status: ${response.status}`);
  }
};
```

#### Manejo de Diferentes Tipos de Respuesta
```javascript
const parseResponse = async (response) => {
  const contentType = response.headers.get('content-type');
  
  if (contentType && contentType.includes('application/json')) {
    return await response.json();
  }
  
  if (contentType && contentType.includes('text/')) {
    return await response.text();
  }
  
  if (contentType && contentType.includes('image/')) {
    return await response.blob();
  }
  
  // Respuesta por defecto
  return await response.text();
};
```

### 6. Servicios Reutilizables

#### Clase Base para APIs
```javascript
// services/ApiService.js
class ApiService {
  constructor(baseURL, defaultHeaders = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...defaultHeaders,
    };
  }
  
  // Método genérico para peticiones
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        ...this.defaultHeaders,
        ...options.headers,
      },
      ...options,
    };
    
    try {
      const response = await fetch(url, config);
      return await this.handleResponse(response);
    } catch (error) {
      console.error(`API Error: ${error.message}`);
      throw error;
    }
  }
  
  // Métodos HTTP específicos
  async get(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }
  
  async post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data),
    });
  }
  
  async put(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }
  
  async delete(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'DELETE' });
  }
  
  // Manejo de respuestas
  async handleResponse(response) {
    if (response.ok) {
      const contentType = response.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        return await response.json();
      }
      return await response.text();
    }
    
    throw new Error(`HTTP error! status: ${response.status}`);
  }
}

export default ApiService;
```

#### Uso del Servicio
```javascript
// services/userService.js
import ApiService from './ApiService';

class UserService extends ApiService {
  constructor() {
    super('https://jsonplaceholder.typicode.com');
  }
  
  // Obtener todos los usuarios
  async getUsers() {
    return this.get('/users');
  }
  
  // Obtener usuario por ID
  async getUserById(id) {
    return this.get(`/users/${id}`);
  }
  
  // Crear nuevo usuario
  async createUser(userData) {
    return this.post('/users', userData);
  }
  
  // Actualizar usuario
  async updateUser(id, userData) {
    return this.put(`/users/${id}`, userData);
  }
  
  // Eliminar usuario
  async deleteUser(id) {
    return this.delete(`/users/${id}`);
  }
}

export default new UserService();
```

### 7. Hook Personalizado para APIs

#### Hook useApi
```javascript
// hooks/useApi.js
import { useState, useCallback } from 'react';

export const useApi = (apiFunction) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await apiFunction(...args);
      setData(result);
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, [apiFunction]);

  return { data, loading, error, execute };
};
```

#### Uso del Hook
```javascript
// Componente usando el hook
import React from 'react';
import { View, Text, Button, ActivityIndicator } from 'react-native';
import { useApi } from '../hooks/useApi';
import userService from '../services/userService';

const UserList = () => {
  const { data: users, loading, error, execute: fetchUsers } = useApi(userService.getUsers);

  React.useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  if (loading) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" color="#0000ff" />
        <Text>Cargando usuarios...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text style={{ color: 'red' }}>Error: {error}</Text>
        <Button title="Reintentar" onPress={fetchUsers} />
      </View>
    );
  }

  return (
    <View style={{ flex: 1, padding: 20 }}>
      <Text style={{ fontSize: 24, fontWeight: 'bold', marginBottom: 20 }}>
        Lista de Usuarios
      </Text>
      {users?.map(user => (
        <View key={user.id} style={{ marginBottom: 10, padding: 10, backgroundColor: '#f0f0f0' }}>
          <Text style={{ fontWeight: 'bold' }}>{user.name}</Text>
          <Text>{user.email}</Text>
        </View>
      ))}
    </View>
  );
};

export default UserList;
```

## 🧪 Ejercicios Prácticos

### Ejercicio 1: API de Posts
Crea un servicio para consumir la API de posts de JSONPlaceholder:

```javascript
// Implementa estos métodos:
// - getPosts()
// - getPostById(id)
// - createPost(postData)
// - updatePost(id, postData)
// - deletePost(id)
```

### Ejercicio 2: Hook Personalizado para Posts
Crea un hook `usePosts` que maneje el estado de los posts:

```javascript
// El hook debe incluir:
// - Estado de posts
// - Estado de carga
// - Estado de error
// - Función para cargar posts
// - Función para crear posts
```

### Ejercicio 3: Componente de Posts
Crea un componente que muestre una lista de posts con funcionalidad CRUD:

```javascript
// El componente debe:
// - Mostrar lista de posts
// - Permitir crear nuevos posts
// - Permitir editar posts existentes
// - Permitir eliminar posts
// - Manejar estados de carga y error
```

## 📝 Resumen de la Clase

### **Conceptos Clave Aprendidos:**
1. **Fetch API**: API nativa para peticiones HTTP en JavaScript
2. **Métodos HTTP**: GET, POST, PUT, DELETE para operaciones CRUD
3. **Manejo de Respuestas**: Verificación de estado HTTP y parsing de datos
4. **Servicios Reutilizables**: Clases para organizar llamadas a APIs
5. **Hooks Personalizados**: Gestión de estado para operaciones de API

### **Habilidades Desarrolladas:**
- Implementar peticiones HTTP básicas
- Manejar errores y respuestas de APIs
- Crear servicios organizados y reutilizables
- Implementar hooks personalizados para APIs
- Gestionar estados de carga y error en componentes

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Axios y Middleware**, que nos permitirá:
- Simplificar las peticiones HTTP
- Implementar interceptores para requests y responses
- Manejar autenticación de manera más elegante
- Implementar middleware personalizado para logging y analytics

---

**💡 Consejo**: Practica creando diferentes servicios para APIs públicas como JSONPlaceholder, Pokemon API, o Rick and Morty API. Esto te ayudará a familiarizarte con diferentes estructuras de respuesta y patrones de API.
