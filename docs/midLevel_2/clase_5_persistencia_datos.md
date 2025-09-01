# 📚 Clase 5: Persistencia de Datos

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 4: Hooks Personalizados Avanzados](clase_4_hooks_personalizados_avanzados.md)
- **➡️ Siguiente**: [Módulo 5: Networking y APIs](../midLevel_3/README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender las diferentes opciones de persistencia en React Native
- Implementar AsyncStorage para almacenamiento clave-valor
- Dominar SQLite para bases de datos relacionales
- Crear sistemas de sincronización offline-first
- Implementar estrategias de backup y restauración

---

## 📚 Contenido Teórico

### **¿Qué es la Persistencia de Datos?**

La persistencia de datos es la capacidad de almacenar información localmente en el dispositivo, permitiendo que la aplicación funcione offline y mantenga el estado entre sesiones.

#### **Opciones de persistencia:**
- **AsyncStorage**: Almacenamiento clave-valor simple
- **SQLite**: Base de datos relacional completa
- **Realm**: Base de datos NoSQL orientada a objetos
- **MMKV**: Almacenamiento de alto rendimiento
- **WatermelonDB**: Base de datos reactiva y sincronizable

---

## 💻 Implementación Práctica

### **1. Servicio de Persistencia con AsyncStorage**

```javascript:src/services/StorageService.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { Platform } from 'react-native';

// Servicio de persistencia con AsyncStorage
class StorageService {
  constructor() {
    this.prefix = '@AppStorage:';
    this.encryptionEnabled = false;
    this.compressionEnabled = false;
    this.maxStorageSize = 50 * 1024 * 1024; // 50MB
  }
  
  // Generar clave completa
  generateKey(key) {
    return `${this.prefix}${key}`;
  }
  
  // Guardar datos
  async setItem(key, value, options = {}) {
    try {
      const fullKey = this.generateKey(key);
      const dataToStore = {
        value,
        timestamp: Date.now(),
        version: options.version || '1.0',
        metadata: options.metadata || {},
      };
      
      // Comprimir si está habilitado
      if (this.compressionEnabled) {
        dataToStore.value = await this.compressData(value);
      }
      
      // Encriptar si está habilitado
      if (this.encryptionEnabled) {
        dataToStore.value = await this.encryptData(dataToStore.value);
      }
      
      const jsonValue = JSON.stringify(dataToStore);
      
      // Verificar tamaño antes de guardar
      if (jsonValue.length > this.maxStorageSize) {
        throw new Error('Datos demasiado grandes para almacenar');
      }
      
      await AsyncStorage.setItem(fullKey, jsonValue);
      
      // Actualizar metadatos de almacenamiento
      await this.updateStorageMetadata(key, dataToStore);
      
      return true;
    } catch (error) {
      console.error('Error al guardar datos:', error);
      throw error;
    }
  }
  
  // Obtener datos
  async getItem(key, options = {}) {
    try {
      const fullKey = this.generateKey(key);
      const jsonValue = await AsyncStorage.getItem(fullKey);
      
      if (jsonValue === null) {
        return null;
      }
      
      const storedData = JSON.parse(jsonValue);
      
      // Verificar versión si es requerido
      if (options.requireVersion && storedData.version !== options.requireVersion) {
        console.warn(`Versión de datos no coincide: ${storedData.version} vs ${options.requireVersion}`);
        return null;
      }
      
      let value = storedData.value;
      
      // Desencriptar si está habilitado
      if (this.encryptionEnabled) {
        value = await this.decryptData(value);
      }
      
      // Descomprimir si está habilitado
      if (this.compressionEnabled) {
        value = await this.decompressData(value);
      }
      
      // Verificar expiración
      if (options.expirationTime && storedData.timestamp) {
        const now = Date.now();
        const expirationTime = storedData.timestamp + options.expirationTime;
        
        if (now > expirationTime) {
          await this.removeItem(key);
          return null;
        }
      }
      
      return value;
    } catch (error) {
      console.error('Error al obtener datos:', error);
      return null;
    }
  }
  
  // Remover datos
  async removeItem(key) {
    try {
      const fullKey = this.generateKey(key);
      await AsyncStorage.removeItem(fullKey);
      
      // Limpiar metadatos
      await this.removeStorageMetadata(key);
      
      return true;
    } catch (error) {
      console.error('Error al remover datos:', error);
      return false;
    }
  }
  
  // Obtener múltiples elementos
  async getMultipleItems(keys, options = {}) {
    try {
      const fullKeys = keys.map(key => this.generateKey(key));
      const results = await AsyncStorage.multiGet(fullKeys);
      
      const parsedResults = {};
      
      for (const [fullKey, jsonValue] of results) {
        if (jsonValue !== null) {
          const key = fullKey.replace(this.prefix, '');
          const value = await this.getItem(key, options);
          if (value !== null) {
            parsedResults[key] = value;
          }
        }
      }
      
      return parsedResults;
    } catch (error) {
      console.error('Error al obtener múltiples elementos:', error);
      return {};
    }
  }
  
  // Guardar múltiples elementos
  async setMultipleItems(items, options = {}) {
    try {
      const operations = [];
      
      for (const [key, value] of Object.entries(items)) {
        operations.push(this.setItem(key, value, options));
      }
      
      await Promise.all(operations);
      return true;
    } catch (error) {
      console.error('Error al guardar múltiples elementos:', error);
      return false;
    }
  }
  
  // Limpiar todos los datos
  async clear() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      const appKeys = keys.filter(key => key.startsWith(this.prefix));
      
      await AsyncStorage.multiRemove(appKeys);
      await this.clearStorageMetadata();
      
      return true;
    } catch (error) {
      console.error('Error al limpiar almacenamiento:', error);
      return false;
    }
  }
  
  // Obtener todas las claves
  async getAllKeys() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      return keys
        .filter(key => key.startsWith(this.prefix))
        .map(key => key.replace(this.prefix, ''));
    } catch (error) {
      console.error('Error al obtener claves:', error);
      return [];
    }
  }
  
  // Obtener información del almacenamiento
  async getStorageInfo() {
    try {
      const keys = await this.getAllKeys();
      let totalSize = 0;
      
      for (const key of keys) {
        const value = await this.getItem(key);
        if (value) {
          totalSize += JSON.stringify(value).length;
        }
      }
      
      return {
        keyCount: keys.length,
        totalSize,
        maxSize: this.maxStorageSize,
        availableSpace: this.maxStorageSize - totalSize,
        usagePercentage: (totalSize / this.maxStorageSize) * 100,
      };
    } catch (error) {
      console.error('Error al obtener información del almacenamiento:', error);
      return null;
    }
  }
  
  // Migrar datos a nueva versión
  async migrateData(fromVersion, toVersion, migrationFunction) {
    try {
      const keys = await this.getAllKeys();
      let migratedCount = 0;
      
      for (const key of keys) {
        const value = await this.getItem(key, { requireVersion: fromVersion });
        
        if (value !== null) {
          const migratedValue = await migrationFunction(value, key);
          await this.setItem(key, migratedValue, { version: toVersion });
          migratedCount++;
        }
      }
      
      return { success: true, migratedCount };
    } catch (error) {
      console.error('Error en migración de datos:', error);
      return { success: false, error: error.message };
    }
  }
  
  // Métodos auxiliares para metadatos
  async updateStorageMetadata(key, metadata) {
    try {
      const metadataKey = `metadata:${key}`;
      await AsyncStorage.setItem(this.generateKey(metadataKey), JSON.stringify(metadata));
    } catch (error) {
      console.error('Error al actualizar metadatos:', error);
    }
  }
  
  async removeStorageMetadata(key) {
    try {
      const metadataKey = `metadata:${key}`;
      await AsyncStorage.removeItem(this.generateKey(metadataKey));
    } catch (error) {
      console.error('Error al remover metadatos:', error);
    }
  }
  
  async clearStorageMetadata() {
    try {
      const keys = await this.getAllKeys();
      const metadataKeys = keys.filter(key => key.startsWith('metadata:'));
      
      for (const key of metadataKeys) {
        await this.removeItem(key);
      }
    } catch (error) {
      console.error('Error al limpiar metadatos:', error);
    }
  }
  
  // Métodos de compresión (simulados)
  async compressData(data) {
    // Implementar compresión real aquí
    return data;
  }
  
  async decompressData(data) {
    // Implementar descompresión real aquí
    return data;
  }
  
  // Métodos de encriptación (simulados)
  async encryptData(data) {
    // Implementar encriptación real aquí
    return data;
  }
  
  async decryptData(data) {
    // Implementar desencriptación real aquí
    return data;
  }
}

// Instancia singleton
const storageService = new StorageService();

export default storageService;
```

### **2. Servicio de Base de Datos SQLite**

```javascript:src/services/DatabaseService.js
import SQLite from 'react-native-sqlite-storage';

// Configuración de SQLite
SQLite.DEBUG(__DEV__);
SQLite.enablePromise(true);

// Servicio de base de datos SQLite
class DatabaseService {
  constructor() {
    this.database = null;
    this.isInitialized = false;
    this.version = 1;
    this.migrations = [];
  }
  
  // Inicializar base de datos
  async initialize() {
    try {
      if (this.isInitialized) {
        return this.database;
      }
      
      // Abrir base de datos
      this.database = await SQLite.openDatabase({
        name: 'AppDatabase.db',
        location: 'default',
        createFromLocation: '~AppDatabase.db',
      });
      
      // Configurar base de datos
      await this.database.executeSql('PRAGMA foreign_keys = ON');
      await this.database.executeSql('PRAGMA journal_mode = WAL');
      await this.database.executeSql('PRAGMA synchronous = NORMAL');
      
      // Ejecutar migraciones
      await this.runMigrations();
      
      this.isInitialized = true;
      console.log('Base de datos inicializada correctamente');
      
      return this.database;
    } catch (error) {
      console.error('Error al inicializar base de datos:', error);
      throw error;
    }
  }
  
  // Ejecutar migraciones
  async runMigrations() {
    try {
      // Crear tabla de versiones si no existe
      await this.executeSql(`
        CREATE TABLE IF NOT EXISTS schema_versions (
          version INTEGER PRIMARY KEY,
          applied_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )
      `);
      
      // Obtener versión actual
      const [result] = await this.executeSql(
        'SELECT MAX(version) as current_version FROM schema_versions'
      );
      
      const currentVersion = result.rows.item(0)?.current_version || 0;
      
      // Ejecutar migraciones pendientes
      for (const migration of this.migrations) {
        if (migration.version > currentVersion) {
          await this.executeSql(migration.sql);
          
          await this.executeSql(
            'INSERT INTO schema_versions (version) VALUES (?)',
            [migration.version]
          );
          
          console.log(`Migración ${migration.version} aplicada`);
        }
      }
    } catch (error) {
      console.error('Error al ejecutar migraciones:', error);
      throw error;
    }
  }
  
  // Ejecutar consulta SQL
  async executeSql(sql, params = []) {
    try {
      if (!this.isInitialized) {
        await this.initialize();
      }
      
      return await this.database.executeSql(sql, params);
    } catch (error) {
      console.error('Error en consulta SQL:', error);
      throw error;
    }
  }
  
  // Consulta SELECT
  async select(sql, params = []) {
    try {
      const [result] = await this.executeSql(sql, params);
      const rows = [];
      
      for (let i = 0; i < result.rows.length; i++) {
        rows.push(result.rows.item(i));
      }
      
      return rows;
    } catch (error) {
      console.error('Error en consulta SELECT:', error);
      return [];
    }
  }
  
  // Insertar registro
  async insert(table, data) {
    try {
      const columns = Object.keys(data);
      const placeholders = columns.map(() => '?').join(', ');
      const values = Object.values(data);
      
      const sql = `INSERT INTO ${table} (${columns.join(', ')}) VALUES (${placeholders})`;
      
      const [result] = await this.executeSql(sql, values);
      
      return {
        id: result.insertId,
        rowsAffected: result.rowsAffected,
      };
    } catch (error) {
      console.error('Error al insertar:', error);
      throw error;
    }
  }
  
  // Actualizar registro
  async update(table, data, where, whereParams = []) {
    try {
      const setClause = Object.keys(data).map(key => `${key} = ?`).join(', ');
      const values = [...Object.values(data), ...whereParams];
      
      const sql = `UPDATE ${table} SET ${setClause} WHERE ${where}`;
      
      const [result] = await this.executeSql(sql, values);
      
      return {
        rowsAffected: result.rowsAffected,
      };
    } catch (error) {
      console.error('Error al actualizar:', error);
      throw error;
    }
  }
  
  // Eliminar registro
  async delete(table, where, whereParams = []) {
    try {
      const sql = `DELETE FROM ${table} WHERE ${where}`;
      
      const [result] = await this.executeSql(sql, whereParams);
      
      return {
        rowsAffected: result.rowsAffected,
      };
    } catch (error) {
      console.error('Error al eliminar:', error);
      throw error;
    }
  }
  
  // Transacción
  async transaction(callback) {
    try {
      if (!this.isInitialized) {
        await this.initialize();
      }
      
      return await this.database.transaction(callback);
    } catch (error) {
      console.error('Error en transacción:', error);
      throw error;
    }
  }
  
  // Cerrar base de datos
  async close() {
    try {
      if (this.database && this.isInitialized) {
        await this.database.close();
        this.isInitialized = false;
        console.log('Base de datos cerrada');
      }
    } catch (error) {
      console.error('Error al cerrar base de datos:', error);
    }
  }
  
  // Agregar migración
  addMigration(version, sql) {
    this.migrations.push({ version, sql });
    this.migrations.sort((a, b) => a.version - b.version);
  }
  
  // Obtener información de la base de datos
  async getDatabaseInfo() {
    try {
      const tables = await this.select(`
        SELECT name FROM sqlite_master 
        WHERE type='table' AND name NOT LIKE 'sqlite_%'
      `);
      
      const tableInfo = {};
      
      for (const table of tables) {
        const columns = await this.select(`PRAGMA table_info(${table.name})`);
        const rowCount = await this.select(`SELECT COUNT(*) as count FROM ${table.name}`);
        
        tableInfo[table.name] = {
          columns: columns.length,
          rows: rowCount[0]?.count || 0,
        };
      }
      
      return {
        version: this.version,
        tables: tableInfo,
        totalTables: tables.length,
      };
    } catch (error) {
      console.error('Error al obtener información de la base de datos:', error);
      return null;
    }
  }
}

// Instancia singleton
const databaseService = new DatabaseService();

// Agregar migraciones de ejemplo
databaseService.addMigration(1, `
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`);

databaseService.addMigration(2, `
  CREATE TABLE IF NOT EXISTS user_profiles (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    avatar TEXT,
    bio TEXT,
    preferences TEXT,
    FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE
  )
`);

export default databaseService;
```

### **3. Hook para Gestión de Persistencia**

```javascript:src/hooks/usePersistence.js
import { useState, useCallback, useRef, useEffect } from 'react';
import storageService from '../services/StorageService';
import databaseService from '../services/DatabaseService';

// Hook para gestión de persistencia
const usePersistence = (options = {}) => {
  const {
    storageType = 'asyncStorage', // 'asyncStorage', 'sqlite', 'hybrid'
    key,
    defaultValue = null,
    autoSave = true,
    autoLoad = true,
    encryption = false,
    compression = false,
    expirationTime = null,
  } = options;
  
  // Estado local
  const [data, setData] = useState(defaultValue);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [lastSaved, setLastSaved] = useState(null);
  
  // Referencias
  const isInitialized = useRef(false);
  const saveTimeout = useRef(null);
  const loadPromise = useRef(null);
  
  // Cargar datos
  const loadData = useCallback(async () => {
    if (!key) return;
    
    try {
      setIsLoading(true);
      setError(null);
      
      let loadedData = null;
      
      switch (storageType) {
        case 'asyncStorage':
          loadedData = await storageService.getItem(key, {
            encryption,
            compression,
            expirationTime,
          });
          break;
          
        case 'sqlite':
          if (key.includes(':')) {
            const [table, id] = key.split(':');
            const rows = await databaseService.select(
              `SELECT * FROM ${table} WHERE id = ?`,
              [id]
            );
            loadedData = rows[0] || null;
          } else {
            const rows = await databaseService.select(
              `SELECT * FROM ${key}`
            );
            loadedData = rows;
          }
          break;
          
        case 'hybrid':
          // Intentar AsyncStorage primero, luego SQLite
          loadedData = await storageService.getItem(key, {
            encryption,
            compression,
            expirationTime,
          });
          
          if (loadedData === null) {
            // Intentar cargar desde SQLite
            const rows = await databaseService.select(
              `SELECT * FROM app_data WHERE key = ?`,
              [key]
            );
            loadedData = rows[0]?.value ? JSON.parse(rows[0].value) : null;
          }
          break;
      }
      
      if (loadedData !== null) {
        setData(loadedData);
        setLastSaved(new Date());
      }
      
      return loadedData;
    } catch (error) {
      console.error('Error al cargar datos:', error);
      setError(error.message);
      return null;
    } finally {
      setIsLoading(false);
    }
  }, [key, storageType, encryption, compression, expirationTime]);
  
  // Guardar datos
  const saveData = useCallback(async (newData = data) => {
    if (!key) return false;
    
    try {
      setError(null);
      
      let success = false;
      
      switch (storageType) {
        case 'asyncStorage':
          success = await storageService.setItem(key, newData, {
            encryption,
            compression,
            expirationTime,
          });
          break;
          
        case 'sqlite':
          if (key.includes(':')) {
            const [table, id] = key.split(':');
            const existing = await databaseService.select(
              `SELECT id FROM ${table} WHERE id = ?`,
              [id]
            );
            
            if (existing.length > 0) {
              await databaseService.update(table, newData, 'id = ?', [id]);
            } else {
              await databaseService.insert(table, { id, ...newData });
            }
            success = true;
          } else {
            // Guardar en tabla genérica
            const existing = await databaseService.select(
              'SELECT id FROM app_data WHERE key = ?',
              [key]
            );
            
            if (existing.length > 0) {
              await databaseService.update(
                'app_data',
                { value: JSON.stringify(newData), updated_at: new Date() },
                'key = ?',
                [key]
              );
            } else {
              await databaseService.insert('app_data', {
                key,
                value: JSON.stringify(newData),
                created_at: new Date(),
                updated_at: new Date(),
              });
            }
            success = true;
          }
          break;
          
        case 'hybrid':
          // Guardar en ambos lugares
          const asyncSuccess = await storageService.setItem(key, newData, {
            encryption,
            compression,
            expirationTime,
          });
          
          const sqliteSuccess = await databaseService.executeSql(`
            INSERT OR REPLACE INTO app_data (key, value, updated_at)
            VALUES (?, ?, ?)
          `, [key, JSON.stringify(newData), new Date()]);
          
          success = asyncSuccess && sqliteSuccess;
          break;
      }
      
      if (success) {
        setData(newData);
        setLastSaved(new Date());
      }
      
      return success;
    } catch (error) {
      console.error('Error al guardar datos:', error);
      setError(error.message);
      return false;
    }
  }, [key, storageType, data, encryption, compression, expirationTime]);
  
  // Guardar con debounce
  const saveWithDebounce = useCallback((newData, delay = 1000) => {
    if (saveTimeout.current) {
      clearTimeout(saveTimeout.current);
    }
    
    saveTimeout.current = setTimeout(() => {
      saveData(newData);
    }, delay);
  }, [saveData]);
  
  // Sincronizar datos
  const syncData = useCallback(async () => {
    if (storageType !== 'hybrid') return false;
    
    try {
      // Cargar desde AsyncStorage
      const asyncData = await storageService.getItem(key, {
        encryption,
        compression,
        expirationTime,
      });
      
      // Cargar desde SQLite
      const rows = await databaseService.select(
        'SELECT * FROM app_data WHERE key = ?',
        [key]
      );
      const sqliteData = rows[0]?.value ? JSON.parse(rows[0].value) : null;
      
      // Determinar datos más recientes
      let mostRecentData = null;
      let mostRecentTime = 0;
      
      if (asyncData && asyncData.timestamp) {
        mostRecentData = asyncData;
        mostRecentTime = asyncData.timestamp;
      }
      
      if (sqliteData && sqliteData.timestamp) {
        if (sqliteData.timestamp > mostRecentTime) {
          mostRecentData = sqliteData;
          mostRecentTime = sqliteData.timestamp;
        }
      }
      
      // Sincronizar si es necesario
      if (mostRecentData) {
        await saveData(mostRecentData);
        return true;
      }
      
      return false;
    } catch (error) {
      console.error('Error al sincronizar datos:', error);
      return false;
    }
  }, [key, storageType, encryption, compression, expirationTime, saveData]);
  
  // Limpiar datos
  const clearData = useCallback(async () => {
    if (!key) return false;
    
    try {
      let success = false;
      
      switch (storageType) {
        case 'asyncStorage':
          success = await storageService.removeItem(key);
          break;
          
        case 'sqlite':
          if (key.includes(':')) {
            const [table, id] = key.split(':');
            await databaseService.delete(table, 'id = ?', [id]);
            success = true;
          } else {
            await databaseService.delete('app_data', 'key = ?', [key]);
            success = true;
          }
          break;
          
        case 'hybrid':
          const asyncSuccess = await storageService.removeItem(key);
          const sqliteSuccess = await databaseService.delete(
            'app_data',
            'key = ?',
            [key]
          );
          success = asyncSuccess && sqliteSuccess;
          break;
      }
      
      if (success) {
        setData(defaultValue);
        setLastSaved(null);
      }
      
      return success;
    } catch (error) {
      console.error('Error al limpiar datos:', error);
      return false;
    }
  }, [key, storageType, defaultValue]);
  
  // Efecto de inicialización
  useEffect(() => {
    if (!isInitialized.current && autoLoad) {
      loadData();
      isInitialized.current = true;
    }
    
    return () => {
      if (saveTimeout.current) {
        clearTimeout(saveTimeout.current);
      }
    };
  }, [autoLoad, loadData]);
  
  // Efecto de auto-guardado
  useEffect(() => {
    if (autoSave && data !== defaultValue && lastSaved) {
      saveWithDebounce(data);
    }
  }, [data, autoSave, defaultValue, lastSaved, saveWithDebounce]);
  
  return {
    // Estado
    data,
    isLoading,
    error,
    lastSaved,
    
    // Funciones
    loadData,
    saveData,
    saveWithDebounce,
    syncData,
    clearData,
    
    // Utilidades
    isInitialized: isInitialized.current,
    hasData: data !== null && data !== defaultValue,
    needsSync: storageType === 'hybrid' && lastSaved === null,
  };
};

export default usePersistence;
```

---

## 🧪 Casos de Uso

### **Caso 1: Persistencia Simple**
```javascript
const { data, saveData, loadData } = usePersistence({
  key: 'user_preferences',
  defaultValue: { theme: 'light', language: 'es' }
});
```

### **Caso 2: Persistencia con SQLite**
```javascript
const { data, saveData } = usePersistence({
  storageType: 'sqlite',
  key: 'users:123',
  defaultValue: { name: '', email: '' }
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Caché**
Implementa un sistema de caché con expiración automática.

### **Ejercicio 2: Sincronización Offline**
Crea un sistema de sincronización offline-first con cola de operaciones.

### **Ejercicio 3: Backup y Restauración**
Desarrolla un sistema de backup y restauración de datos locales.

---

## 🚀 Proyecto de la Clase

### **App con Persistencia Completa**

Crea una aplicación que demuestre:
- **Múltiples tipos de almacenamiento**: AsyncStorage, SQLite, híbrido
- **Sincronización offline**: Con cola de operaciones pendientes
- **Migración de datos**: Entre versiones de la aplicación
- **Backup automático**: Con encriptación y compresión

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **AsyncStorage**: Almacenamiento clave-valor simple y eficiente
- **SQLite**: Base de datos relacional completa para datos complejos
- **Sincronización**: Estrategias para mantener datos consistentes
- **Migración**: Actualización de esquemas de datos

### **Habilidades Desarrolladas:**
- ✅ Implementar persistencia con AsyncStorage
- ✅ Configurar y usar SQLite en React Native
- ✅ Crear sistemas de sincronización offline
- ✅ Gestionar migraciones y versiones de datos

### **Próximos Pasos:**
En el siguiente módulo aprenderemos sobre **Networking y APIs**, que te permitirá conectar tu aplicación con servicios externos.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [Hooks Personalizados Avanzados](clase_4_hooks_personalizados_avanzados.md)
- **➡️ Siguiente Módulo**: [Módulo 5: Networking y APIs](../midLevel_3/README.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
