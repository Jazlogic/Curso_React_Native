# 📚 Clase 2: SQLite - Base de Datos Relacional

## 🧭 Navegación del Módulo

- **⬅️ Anterior**: [Clase 1: AsyncStorage - Almacenamiento Clave-Valor](clase_1_async_storage.md)
- **➡️ Siguiente**: [Clase 3: Realm - Base de Datos NoSQL](clase_3_realm.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

- Comprender qué es SQLite y cuándo usarlo
- Configurar SQLite en React Native
- Crear y gestionar tablas de base de datos
- Implementar operaciones CRUD complejas
- Manejar relaciones entre tablas
- Implementar migraciones y versionado

---

## 📖 Contenido Teórico

### 1. ¿Qué es SQLite?

SQLite es una base de datos relacional ligera y autónoma que se ejecuta dentro de la aplicación. Es ideal para:

- **Datos estructurados** (usuarios, productos, pedidos)
- **Relaciones complejas** (uno a muchos, muchos a muchos)
- **Consultas SQL avanzadas** (JOINs, GROUP BY, subconsultas)
- **Transacciones** (operaciones atómicas)
- **Datos que requieren integridad referencial**

**Características principales:**
- **Sin servidor**: No requiere proceso separado
- **Transaccional**: ACID compliance completo
- **SQL estándar**: Sintaxis familiar para desarrolladores
- **Eficiente**: Optimizado para dispositivos móviles
- **Portable**: Un solo archivo de base de datos

### 2. Instalación y Configuración

#### **Instalación del Paquete:**
```bash
npm install react-native-sqlite-storage
# o
yarn add react-native-sqlite-storage
```

#### **Configuración de iOS (Podfile):**
```ruby
# ios/Podfile
pod 'react-native-sqlite-storage', :path => '../node_modules/react-native-sqlite-storage'
```

#### **Configuración de Android (android/app/src/main/java/.../MainApplication.java):**
```java
// Agregar en el método getPackages()
packages.add(new SQLitePluginPackage());
```

---

## 🛠️ Implementación Práctica

### 1. Clase Helper para Base de Datos

Vamos a crear una clase completa que maneje todas las operaciones de SQLite:

```javascript
// database/sqlite.js
import SQLite from 'react-native-sqlite-storage';

// Habilitamos el modo debug para desarrollo
SQLite.DEBUG(true);

// Habilitamos el uso de promesas
SQLite.enablePromise(true);

// Configuración de la base de datos
const database_name = "AppDatabase.db";
const database_version = "1.0";
const database_displayname = "SQLite React Native Database";
const database_size = 200000; // 200MB

export default class DatabaseHelper {
  // Propiedad para almacenar la instancia de la base de datos
  db = null;

  // Método para inicializar la base de datos
  async initDB() {
    // Si ya tenemos una instancia, la retornamos
    if (this.db) {
      return this.db;
    }
    
    try {
      // Abrimos la base de datos con la configuración especificada
      this.db = await SQLite.openDatabase({
        name: database_name,           // Nombre del archivo de base de datos
        version: database_version,     // Versión actual
        displayName: database_displayname, // Nombre para mostrar
        size: database_size,           // Tamaño máximo en bytes
      });
      
      // Creamos las tablas necesarias
      await this.createTables();
      
      console.log('Database initialized successfully');
      return this.db;
    } catch (error) {
      console.error('Error initializing database:', error);
      throw error;
    }
  }

  // Método para crear las tablas de la base de datos
  async createTables() {
    try {
      // Tabla de usuarios
      const createUsersTable = `
        CREATE TABLE IF NOT EXISTS users (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          email TEXT UNIQUE NOT NULL,
          password_hash TEXT NOT NULL,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
      `;
      
      // Tabla de productos
      const createProductsTable = `
        CREATE TABLE IF NOT EXISTS products (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          description TEXT,
          price REAL NOT NULL,
          category_id INTEGER,
          stock_quantity INTEGER DEFAULT 0,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          FOREIGN KEY (category_id) REFERENCES categories (id)
        );
      `;
      
      // Tabla de categorías
      const createCategoriesTable = `
        CREATE TABLE IF NOT EXISTS categories (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          description TEXT,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
      `;
      
      // Tabla de pedidos
      const createOrdersTable = `
        CREATE TABLE IF NOT EXISTS orders (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          user_id INTEGER NOT NULL,
          total_amount REAL NOT NULL,
          status TEXT DEFAULT 'pending',
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          FOREIGN KEY (user_id) REFERENCES users (id)
        );
      `;
      
      // Tabla de items de pedido (relación muchos a muchos)
      const createOrderItemsTable = `
        CREATE TABLE IF NOT EXISTS order_items (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          order_id INTEGER NOT NULL,
          product_id INTEGER NOT NULL,
          quantity INTEGER NOT NULL,
          unit_price REAL NOT NULL,
          total_price REAL NOT NULL,
          FOREIGN KEY (order_id) REFERENCES orders (id),
          FOREIGN KEY (product_id) REFERENCES products (id)
        );
      `;
      
      // Ejecutamos todas las consultas de creación
      await this.db.executeSql(createUsersTable);
      await this.db.executeSql(createCategoriesTable);
      await this.db.executeSql(createProductsTable);
      await this.db.executeSql(createOrdersTable);
      await this.db.executeSql(createOrderItemsTable);
      
      console.log('Tables created successfully');
    } catch (error) {
      console.error('Error creating tables:', error);
      throw error;
    }
  }

  // Método para insertar un usuario
  async insertUser(user) {
    try {
      // Consulta SQL parametrizada para evitar SQL injection
      const query = `
        INSERT INTO users (name, email, password_hash) 
        VALUES (?, ?, ?)
      `;
      
      // Parámetros en el orden correcto
      const params = [user.name, user.email, user.passwordHash];
      
      // Ejecutamos la consulta
      const [result] = await this.db.executeSql(query, params);
      
      console.log('User inserted successfully, ID:', result.insertId);
      return result.insertId;
    } catch (error) {
      console.error('Error inserting user:', error);
      throw error;
    }
  }

  // Método para obtener todos los usuarios
  async getUsers() {
    try {
      const query = 'SELECT * FROM users ORDER BY created_at DESC';
      const [results] = await this.db.executeSql(query);
      
      // Convertimos los resultados a un array de objetos
      const users = [];
      for (let i = 0; i < results.rows.length; i++) {
        users.push(results.rows.item(i));
      }
      
      return users;
    } catch (error) {
      console.error('Error getting users:', error);
      throw error;
    }
  }

  // Método para obtener un usuario por ID
  async getUserById(id) {
    try {
      const query = 'SELECT * FROM users WHERE id = ?';
      const [results] = await this.db.executeSql(query, [id]);
      
      if (results.rows.length > 0) {
        return results.rows.item(0);
      }
      
      return null;
    } catch (error) {
      console.error('Error getting user by ID:', error);
      throw error;
    }
  }

  // Método para actualizar un usuario
  async updateUser(id, updates) {
    try {
      // Construimos la consulta dinámicamente basada en los campos a actualizar
      const fields = Object.keys(updates);
      const setClause = fields.map(field => `${field} = ?`).join(', ');
      
      const query = `
        UPDATE users 
        SET ${setClause}, updated_at = CURRENT_TIMESTAMP
        WHERE id = ?
      `;
      
      // Agregamos el ID al final de los parámetros
      const params = [...Object.values(updates), id];
      
      const [result] = await this.db.executeSql(query, params);
      
      console.log('User updated successfully, rows affected:', result.rowsAffected);
      return result.rowsAffected > 0;
    } catch (error) {
      console.error('Error updating user:', error);
      throw error;
    }
  }

  // Método para eliminar un usuario
  async deleteUser(id) {
    try {
      const query = 'DELETE FROM users WHERE id = ?';
      const [result] = await this.db.executeSql(query, [id]);
      
      console.log('User deleted successfully, rows affected:', result.rowsAffected);
      return result.rowsAffected > 0;
    } catch (error) {
      console.error('Error deleting user:', error);
      throw error;
    }
  }

  // Método para buscar usuarios por nombre o email
  async searchUsers(searchTerm) {
    try {
      const query = `
        SELECT * FROM users 
        WHERE name LIKE ? OR email LIKE ?
        ORDER BY name ASC
      `;
      
      // Agregamos % para búsqueda parcial
      const searchPattern = `%${searchTerm}%`;
      const params = [searchPattern, searchPattern];
      
      const [results] = await this.db.executeSql(query, params);
      
      const users = [];
      for (let i = 0; i < results.rows.length; i++) {
        users.push(results.rows.item(i));
      }
      
      return users;
    } catch (error) {
      console.error('Error searching users:', error);
      throw error;
    }
  }

  // Método para obtener usuarios con paginación
  async getUsersPaginated(page = 1, limit = 10) {
    try {
      const offset = (page - 1) * limit;
      
      const query = `
        SELECT * FROM users 
        ORDER BY created_at DESC 
        LIMIT ? OFFSET ?
      `;
      
      const params = [limit, offset];
      const [results] = await this.db.executeSql(query, params);
      
      const users = [];
      for (let i = 0; i < results.rows.length; i++) {
        users.push(results.rows.item(i));
      }
      
      // También obtenemos el total de usuarios para la paginación
      const countQuery = 'SELECT COUNT(*) as total FROM users';
      const [countResults] = await this.db.executeSql(countQuery);
      const total = countResults.rows.item(0).total;
      
      return {
        users,
        pagination: {
          page,
          limit,
          total,
          totalPages: Math.ceil(total / limit),
          hasNext: page * limit < total,
          hasPrev: page > 1
        }
      };
    } catch (error) {
      console.error('Error getting paginated users:', error);
      throw error;
    }
  }

  // Método para cerrar la base de datos
  async closeDB() {
    try {
      if (this.db) {
        await this.db.close();
        this.db = null;
        console.log('Database closed successfully');
      }
    } catch (error) {
      console.error('Error closing database:', error);
      throw error;
    }
  }

  // Método para eliminar la base de datos (útil para testing)
  async deleteDB() {
    try {
      if (this.db) {
        await this.db.close();
        this.db = null;
      }
      
      await SQLite.deleteDatabase({
        name: database_name,
        location: 'default'
      });
      
      console.log('Database deleted successfully');
    } catch (error) {
      console.error('Error deleting database:', error);
      throw error;
    }
  }
}
```

### 2. Explicación Línea por Línea de la Clase

#### **Importación y Configuración:**
```javascript
import SQLite from 'react-native-sqlite-storage';

SQLite.DEBUG(true);
SQLite.enablePromise(true);
```
- **`import SQLite`**: Importamos la librería SQLite para React Native
- **`SQLite.DEBUG(true)`**: Habilita logs de debug para desarrollo
- **`SQLite.enablePromise(true)`**: Permite usar async/await en lugar de callbacks

#### **Configuración de la Base de Datos:**
```javascript
const database_name = "AppDatabase.db";
const database_version = "1.0";
const database_displayname = "SQLite React Native Database";
const database_size = 200000; // 200MB
```
- **`database_name`**: Nombre del archivo de base de datos en el dispositivo
- **`database_version`**: Versión para manejar migraciones
- **`database_displayname`**: Nombre descriptivo para debugging
- **`database_size`**: Tamaño máximo en bytes (200MB)

#### **Propiedad de la Clase:**
```javascript
export default class DatabaseHelper {
  db = null;
```
- **`export default`**: Exporta la clase como exportación principal
- **`db = null`**: Propiedad para almacenar la instancia de la base de datos

#### **Método initDB:**
```javascript
async initDB() {
  if (this.db) {
    return this.db;
  }
  
  try {
    this.db = await SQLite.openDatabase({
      name: database_name,
      version: database_version,
      displayName: database_displayname,
      size: database_size,
    });
    
    await this.createTables();
    return this.db;
  } catch (error) {
    console.error('Error initializing database:', error);
    throw error;
  }
}
```
- **`if (this.db)`**: Verifica si ya tenemos una instancia (patrón singleton)
- **`SQLite.openDatabase()`**: Abre o crea la base de datos
- **`await this.createTables()`**: Crea las tablas necesarias
- **`throw error`**: Re-lanza el error para manejo en el componente

#### **Método createTables:**
```javascript
async createTables() {
  try {
    const createUsersTable = `
      CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        password_hash TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
      );
    `;
    
    await this.db.executeSql(createUsersTable);
  } catch (error) {
    console.error('Error creating tables:', error);
    throw error;
  }
}
```
- **`CREATE TABLE IF NOT EXISTS`**: Crea la tabla solo si no existe
- **`INTEGER PRIMARY KEY AUTOINCREMENT`**: ID auto-incremental único
- **`TEXT NOT NULL`**: Campo de texto obligatorio
- **`TEXT UNIQUE`**: Campo de texto único (no duplicados)
- **`DATETIME DEFAULT CURRENT_TIMESTAMP`**: Fecha automática de creación

#### **Método insertUser:**
```javascript
async insertUser(user) {
  try {
    const query = `
      INSERT INTO users (name, email, password_hash) 
      VALUES (?, ?, ?)
    `;
    
    const params = [user.name, user.email, user.passwordHash];
    const [result] = await this.db.executeSql(query, params);
    
    return result.insertId;
  } catch (error) {
    console.error('Error inserting user:', error);
    throw error;
  }
}
```
- **`INSERT INTO users (...)`**: Consulta SQL para insertar datos
- **`VALUES (?, ?, ?)`**: Placeholders para parámetros (previene SQL injection)
- **`const [result]`**: Destructuring del resultado (executeSql retorna un array)
- **`result.insertId`**: ID del registro insertado

#### **Método getUsers:**
```javascript
async getUsers() {
  try {
    const query = 'SELECT * FROM users ORDER BY created_at DESC';
    const [results] = await this.db.executeSql(query);
    
    const users = [];
    for (let i = 0; i < results.rows.length; i++) {
      users.push(results.rows.item(i));
    }
    
    return users;
  } catch (error) {
    console.error('Error getting users:', error);
    throw error;
  }
}
```
- **`SELECT * FROM users`**: Consulta para obtener todos los usuarios
- **`ORDER BY created_at DESC`**: Ordena por fecha de creación descendente
- **`results.rows.length`**: Número de filas retornadas
- **`results.rows.item(i)`**: Obtiene la fila en el índice especificado

#### **Método updateUser:**
```javascript
async updateUser(id, updates) {
  try {
    const fields = Object.keys(updates);
    const setClause = fields.map(field => `${field} = ?`).join(', ');
    
    const query = `
      UPDATE users 
      SET ${setClause}, updated_at = CURRENT_TIMESTAMP
      WHERE id = ?
    `;
    
    const params = [...Object.values(updates), id];
    const [result] = await this.db.executeSql(query, params);
    
    return result.rowsAffected > 0;
  } catch (error) {
    console.error('Error updating user:', error);
    throw error;
  }
}
```
- **`Object.keys(updates)`**: Obtiene las claves del objeto de actualizaciones
- **`fields.map(...).join(', ')`**: Construye la cláusula SET dinámicamente
- **`Object.values(updates)`**: Obtiene los valores de las actualizaciones
- **`result.rowsAffected`**: Número de filas afectadas por la actualización

---

## 💡 Casos de Uso Prácticos

### 1. Servicio de Usuarios

```javascript
// services/userService.js
import DatabaseHelper from '../database/sqlite';

class UserService {
  constructor() {
    this.dbHelper = new DatabaseHelper();
  }

  async initialize() {
    await this.dbHelper.initDB();
  }

  async createUser(userData) {
    // Hash de la contraseña (en producción usar bcrypt)
    const passwordHash = Buffer.from(userData.password).toString('base64');
    
    const user = {
      name: userData.name,
      email: userData.email,
      passwordHash
    };
    
    const userId = await this.dbHelper.insertUser(user);
    return this.getUserById(userId);
  }

  async authenticateUser(email, password) {
    const users = await this.dbHelper.searchUsers(email);
    const user = users.find(u => u.email === email);
    
    if (user) {
      const passwordHash = Buffer.from(password).toString('base64');
      if (user.password_hash === passwordHash) {
        return user;
      }
    }
    
    return null;
  }

  async updateUserProfile(userId, profileData) {
    const allowedFields = ['name', 'email'];
    const updates = {};
    
    // Solo permitimos actualizar campos específicos
    allowedFields.forEach(field => {
      if (profileData[field] !== undefined) {
        updates[field] = profileData[field];
      }
    });
    
    if (Object.keys(updates).length > 0) {
      return await this.dbHelper.updateUser(userId, updates);
    }
    
    return false;
  }

  async deleteUser(userId) {
    return await this.dbHelper.deleteUser(userId);
  }

  async getUserById(userId) {
    return await this.dbHelper.getUserById(userId);
  }

  async searchUsers(searchTerm) {
    return await this.dbHelper.searchUsers(searchTerm);
  }

  async getUsersPaginated(page, limit) {
    return await this.dbHelper.getUsersPaginated(page, limit);
  }
}

export default new UserService();
```

### 2. Hook para Usar la Base de Datos

```javascript
// hooks/useDatabase.js
import { useState, useEffect, useCallback } from 'react';
import DatabaseHelper from '../database/sqlite';

export const useDatabase = () => {
  const [dbHelper, setDbHelper] = useState(null);
  const [isInitialized, setIsInitialized] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const initDatabase = async () => {
      try {
        const helper = new DatabaseHelper();
        await helper.initDB();
        setDbHelper(helper);
        setIsInitialized(true);
      } catch (err) {
        setError(err);
        console.error('Failed to initialize database:', err);
      }
    };

    initDatabase();

    // Cleanup al desmontar el componente
    return () => {
      if (dbHelper) {
        dbHelper.closeDB();
      }
    };
  }, []);

  const executeQuery = useCallback(async (query, params = []) => {
    if (!dbHelper) {
      throw new Error('Database not initialized');
    }

    try {
      const [results] = await dbHelper.db.executeSql(query, params);
      return results;
    } catch (err) {
      console.error('Query execution failed:', err);
      throw err;
    }
  }, [dbHelper]);

  return {
    dbHelper,
    isInitialized,
    error,
    executeQuery
  };
};
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Configuración Básica**
1. Instala `react-native-sqlite-storage` en tu proyecto
2. Configura iOS y Android según la documentación
3. Crea la clase `DatabaseHelper` básica
4. Prueba la inicialización de la base de datos

### **Ejercicio 2: Operaciones CRUD de Usuarios**
1. Implementa todos los métodos CRUD para usuarios
2. Crea un formulario para agregar/editar usuarios
3. Implementa validación de datos antes de insertar
4. Prueba todas las operaciones

### **Ejercicio 3: Búsqueda y Filtrado**
1. Implementa búsqueda por nombre y email
2. Agrega filtros por fecha de creación
3. Implementa ordenamiento por diferentes campos
4. Crea una interfaz de búsqueda

### **Ejercicio 4: Paginación**
1. Implementa paginación en la lista de usuarios
2. Crea controles de navegación (anterior/siguiente)
3. Muestra información de paginación
4. Optimiza las consultas para grandes volúmenes

### **Ejercicio 5: Relaciones entre Tablas**
1. Crea las tablas de productos y categorías
2. Implementa relaciones uno a muchos
3. Crea consultas con JOINs
4. Prueba la integridad referencial

---

## 🎯 Proyecto de la Clase

### **Sistema de Gestión de Usuarios**

Crea una aplicación que permita:
1. **Registrar usuarios** con validación
2. **Autenticar usuarios** (login/logout)
3. **Gestionar perfiles** (editar información)
4. **Buscar usuarios** con filtros avanzados
5. **Administrar usuarios** (CRUD completo)

**Estructura del Proyecto:**
```
src/
├── database/
│   └── sqlite.js          # Clase helper de base de datos
├── services/
│   └── userService.js     # Servicio de usuarios
├── hooks/
│   └── useDatabase.js     # Hook para base de datos
├── components/
│   ├── UserForm.js        # Formulario de usuario
│   ├── UserList.js        # Lista de usuarios
│   └── UserSearch.js      # Búsqueda de usuarios
└── screens/
    ├── LoginScreen.js     # Pantalla de login
    ├── RegisterScreen.js  # Pantalla de registro
    └── UsersScreen.js     # Pantalla de gestión de usuarios
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage)
- [SQLite Documentation](https://www.sqlite.org/docs.html)

### **Mejores Prácticas:**
- **Transacciones**: Usa transacciones para operaciones múltiples
- **Índices**: Crea índices para campos de búsqueda frecuente
- **Prepared Statements**: Siempre usa parámetros para evitar SQL injection
- **Manejo de Errores**: Implementa manejo robusto de errores

### **Alternativas:**
- **WatermelonDB**: Para aplicaciones con datos complejos
- **Realm**: Para bases de datos NoSQL
- **PouchDB**: Para sincronización con CouchDB

---

## 🔍 Resumen de la Clase

### **Conceptos Clave:**
- ✅ SQLite es ideal para datos estructurados y relaciones
- ✅ Siempre usar parámetros para prevenir SQL injection
- ✅ Implementar manejo de errores robusto
- ✅ Usar transacciones para operaciones múltiples
- ✅ Crear índices para optimizar consultas

### **Próximos Pasos:**
- 🔄 Completar todos los ejercicios de esta clase
- 🔄 Implementar el sistema de gestión de usuarios
- 🔄 Practicar con consultas SQL complejas
- 🔄 Prepararse para la siguiente clase sobre Realm

---

## 🚀 Siguiente Clase

En la **Clase 3: Realm - Base de Datos NoSQL**, aprenderás:
- Cómo configurar Realm en React Native
- Definición de esquemas y modelos
- Operaciones CRUD con objetos
- Relaciones y consultas complejas
- Migraciones y versionado de esquemas

---

**¡Excelente trabajo con SQLite! 🎉**

**Recuerda**: SQLite es poderoso para datos relacionales. Domina las consultas SQL antes de avanzar a bases de datos NoSQL.
