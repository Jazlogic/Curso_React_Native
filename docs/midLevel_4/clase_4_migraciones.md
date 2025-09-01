# 📚 Clase 4: Migraciones y Versionado

## 🧭 Navegación del Módulo

- **⬅️ Anterior**: [Clase 3: Realm - Base de Datos NoSQL](clase_3_realm.md)
- **➡️ Siguiente**: [Clase 5: Sincronización de Datos](clase_5_sincronizacion.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

- Comprender qué son las migraciones y por qué son necesarias
- Implementar migraciones automáticas en AsyncStorage
- Manejar versionado de esquemas en SQLite
- Implementar migraciones complejas en Realm
- Crear estrategias de migración robustas
- Testing de migraciones

---

## 📖 Contenido Teórico

### 1. ¿Qué son las Migraciones?

Las migraciones son procesos que permiten actualizar la estructura de la base de datos cuando cambia el esquema de la aplicación. Son necesarias para:

- **Cambios en la estructura** de datos (nuevos campos, tablas)
- **Modificaciones en tipos** de datos existentes
- **Reestructuración** de relaciones entre entidades
- **Limpieza** de datos obsoletos
- **Optimización** de índices y consultas

**Tipos de migraciones:**
- **Automáticas**: Se ejecutan sin intervención del usuario
- **Manuales**: Requieren confirmación o intervención del usuario
- **Incrementales**: Se ejecutan paso a paso por versión
- **Rollback**: Permiten revertir a versiones anteriores

### 2. ¿Por qué son Importantes?

**Sin migraciones:**
- Los usuarios pierden datos al actualizar la app
- Errores de compatibilidad entre versiones
- Imposibilidad de agregar nuevas funcionalidades
- Pérdida de confianza del usuario

**Con migraciones:**
- Los datos se preservan entre versiones
- Actualizaciones transparentes para el usuario
- Evolución continua de la aplicación
- Experiencia de usuario consistente

---

## 🛠️ Implementación Práctica

### 1. Migraciones en AsyncStorage

Vamos a crear un sistema de migración robusto para AsyncStorage:

```javascript
// database/migrations/asyncStorageMigrations.js
import AsyncStorage from '@react-native-async-storage/async-storage';

// Constantes para el sistema de migración
const MIGRATION_VERSION_KEY = 'app_migration_version';
const CURRENT_APP_VERSION = 3; // Versión actual de la aplicación

// Clase principal para manejar migraciones
export class AsyncStorageMigrationService {
  // Propiedad para almacenar la versión actual de la base de datos
  currentVersion = 0;

  // Método para inicializar el servicio de migración
  async initialize() {
    try {
      // Obtenemos la versión actual de la base de datos
      const storedVersion = await AsyncStorage.getItem(MIGRATION_VERSION_KEY);
      this.currentVersion = storedVersion ? parseInt(storedVersion, 10) : 0;
      
      console.log(`Current database version: ${this.currentVersion}`);
      console.log(`Target app version: ${CURRENT_APP_VERSION}`);
      
      // Verificamos si necesitamos migrar
      if (this.currentVersion < CURRENT_APP_VERSION) {
        await this.performMigration();
      } else {
        console.log('Database is up to date');
      }
    } catch (error) {
      console.error('Error initializing migration service:', error);
      throw error;
    }
  }

  // Método principal para ejecutar migraciones
  async performMigration() {
    try {
      console.log('Starting database migration...');
      
      // Ejecutamos migraciones paso a paso
      for (let version = this.currentVersion + 1; version <= CURRENT_APP_VERSION; version++) {
        console.log(`Migrating to version ${version}...`);
        
        // Ejecutamos la migración específica para esta versión
        await this.migrateToVersion(version);
        
        // Actualizamos la versión en la base de datos
        await AsyncStorage.setItem(MIGRATION_VERSION_KEY, version.toString());
        this.currentVersion = version;
        
        console.log(`Successfully migrated to version ${version}`);
      }
      
      console.log('Database migration completed successfully');
    } catch (error) {
      console.error('Migration failed:', error);
      throw error;
    }
  }

  // Método para ejecutar migración a una versión específica
  async migrateToVersion(targetVersion) {
    try {
      switch (targetVersion) {
        case 1:
          await this.migrateToV1();
          break;
        case 2:
          await this.migrateToV2();
          break;
        case 3:
          await this.migrateToV3();
          break;
        default:
          console.warn(`No migration defined for version ${targetVersion}`);
      }
    } catch (error) {
      console.error(`Migration to version ${targetVersion} failed:`, error);
      throw error;
    }
  }

  // Migración a la versión 1: Estructura básica de usuario
  async migrateToV1() {
    console.log('Executing migration to version 1...');
    
    try {
      // Verificamos si ya existe la estructura de usuario
      const existingUser = await AsyncStorage.getItem('userData');
      
      if (!existingUser) {
        // Creamos la estructura inicial de usuario
        const defaultUser = {
          id: Date.now(),
          name: 'Usuario',
          email: '',
          preferences: {
            theme: 'light',
            language: 'es',
            notifications: true
          },
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString()
        };
        
        // Guardamos el usuario por defecto
        await AsyncStorage.setItem('userData', JSON.stringify(defaultUser));
        console.log('Created default user structure');
      }
      
      // Verificamos si existe la estructura de configuración
      const existingConfig = await AsyncStorage.getItem('appConfig');
      
      if (!existingConfig) {
        // Creamos la configuración inicial de la aplicación
        const defaultConfig = {
          firstLaunch: true,
          tutorialCompleted: false,
          lastUpdateCheck: new Date().toISOString(),
          features: {
            darkMode: true,
            offlineMode: true,
            analytics: false
          }
        };
        
        await AsyncStorage.setItem('appConfig', JSON.stringify(defaultConfig));
        console.log('Created default app configuration');
      }
      
      console.log('Migration to version 1 completed successfully');
    } catch (error) {
      console.error('Migration to version 1 failed:', error);
      throw error;
    }
  }

  // Migración a la versión 2: Sistema de categorías y productos
  async migrateToV2() {
    console.log('Executing migration to version 2...');
    
    try {
      // Verificamos si existe la estructura de categorías
      const existingCategories = await AsyncStorage.getItem('categories');
      
      if (!existingCategories) {
        // Creamos categorías por defecto
        const defaultCategories = [
          {
            id: 1,
            name: 'Electrónicos',
            description: 'Productos electrónicos y tecnología',
            isActive: true,
            createdAt: new Date().toISOString()
          },
          {
            id: 2,
            name: 'Ropa',
            description: 'Vestimenta y accesorios',
            isActive: true,
            createdAt: new Date().toISOString()
          },
          {
            id: 3,
            name: 'Hogar',
            description: 'Artículos para el hogar',
            isActive: true,
            createdAt: new Date().toISOString()
          }
        ];
        
        await AsyncStorage.setItem('categories', JSON.stringify(defaultCategories));
        console.log('Created default categories');
      }
      
      // Verificamos si existe la estructura de productos
      const existingProducts = await AsyncStorage.getItem('products');
      
      if (!existingProducts) {
        // Creamos productos de ejemplo
        const defaultProducts = [
          {
            id: 1,
            name: 'Smartphone XYZ',
            description: 'Teléfono inteligente de última generación',
            price: 299.99,
            categoryId: 1,
            isAvailable: true,
            stockQuantity: 10,
            createdAt: new Date().toISOString()
          },
          {
            id: 2,
            name: 'Camiseta Básica',
            description: 'Camiseta de algodón 100%',
            price: 19.99,
            categoryId: 2,
            isAvailable: true,
            stockQuantity: 50,
            createdAt: new Date().toISOString()
          }
        ];
        
        await AsyncStorage.setItem('products', JSON.stringify(defaultProducts));
        console.log('Created default products');
      }
      
      // Migramos datos de usuario existentes si es necesario
      await this.migrateUserDataToV2();
      
      console.log('Migration to version 2 completed successfully');
    } catch (error) {
      console.error('Migration to version 2 failed:', error);
      throw error;
    }
  }

  // Migración a la versión 3: Sistema de pedidos y historial
  async migrateToV3() {
    console.log('Executing migration to version 3...');
    
    try {
      // Verificamos si existe la estructura de pedidos
      const existingOrders = await AsyncStorage.getItem('orders');
      
      if (!existingOrders) {
        // Creamos estructura inicial de pedidos
        const defaultOrders = [];
        await AsyncStorage.setItem('orders', JSON.stringify(defaultOrders));
        console.log('Created orders structure');
      }
      
      // Verificamos si existe la estructura de historial
      const existingHistory = await AsyncStorage.getItem('userHistory');
      
      if (!existingHistory) {
        // Creamos historial inicial del usuario
        const defaultHistory = {
          lastViewedProducts: [],
          searchHistory: [],
          favoriteProducts: [],
          recentSearches: []
        };
        
        await AsyncStorage.setItem('userHistory', JSON.stringify(defaultHistory));
        console.log('Created user history structure');
      }
      
      // Migramos preferencias de usuario existentes
      await this.migrateUserPreferencesToV3();
      
      // Limpiamos datos obsoletos
      await this.cleanupObsoleteData();
      
      console.log('Migration to version 3 completed successfully');
    } catch (error) {
      console.error('Migration to version 3 failed:', error);
      throw error;
    }
  }

  // Método para migrar datos de usuario a la versión 2
  async migrateUserDataToV2() {
    try {
      const existingUser = await AsyncStorage.getItem('userData');
      
      if (existingUser) {
        const user = JSON.parse(existingUser);
        
        // Agregamos nuevos campos si no existen
        if (!user.hasOwnProperty('profile')) {
          user.profile = {
            avatar: null,
            phone: '',
            address: '',
            birthDate: null
          };
        }
        
        if (!user.hasOwnProperty('settings')) {
          user.settings = {
            privacy: {
              shareData: false,
              showProfile: true,
              allowNotifications: true
            },
            accessibility: {
              fontSize: 'medium',
              highContrast: false,
              reduceMotion: false
            }
          };
        }
        
        // Actualizamos la fecha de modificación
        user.updatedAt = new Date().toISOString();
        
        // Guardamos el usuario actualizado
        await AsyncStorage.setItem('userData', JSON.stringify(user));
        console.log('User data migrated to version 2');
      }
    } catch (error) {
      console.error('Error migrating user data to version 2:', error);
      throw error;
    }
  }

  // Método para migrar preferencias de usuario a la versión 3
  async migrateUserPreferencesToV3() {
    try {
      const existingUser = await AsyncStorage.getItem('userData');
      
      if (existingUser) {
        const user = JSON.parse(existingUser);
        
        // Agregamos nuevas preferencias si no existen
        if (!user.hasOwnProperty('preferences')) {
          user.preferences = {
            theme: 'light',
            language: 'es',
            notifications: true,
            sound: true,
            vibration: true
          };
        }
        
        // Agregamos preferencias de contenido
        if (!user.preferences.hasOwnProperty('content')) {
          user.preferences.content = {
            showAdultContent: false,
            filterExplicitContent: true,
            preferredCategories: [],
            blockedCategories: []
          };
        }
        
        // Agregamos preferencias de rendimiento
        if (!user.preferences.hasOwnProperty('performance')) {
          user.preferences.performance = {
            autoPlayVideos: false,
            preloadImages: true,
            cacheSize: 'medium',
            backgroundSync: true
          };
        }
        
        // Actualizamos la fecha de modificación
        user.updatedAt = new Date().toISOString();
        
        // Guardamos el usuario actualizado
        await AsyncStorage.setItem('userData', JSON.stringify(user));
        console.log('User preferences migrated to version 3');
      }
    } catch (error) {
      console.error('Error migrating user preferences to version 3:', error);
      throw error;
    }
  }

  // Método para limpiar datos obsoletos
  async cleanupObsoleteData() {
    try {
      // Obtenemos todas las claves almacenadas
      const allKeys = await AsyncStorage.getAllKeys();
      
      // Lista de claves obsoletas que ya no se usan
      const obsoleteKeys = [
        'oldUserData',
        'tempData',
        'debugInfo',
        'testData'
      ];
      
      // Eliminamos claves obsoletas si existen
      const keysToRemove = obsoleteKeys.filter(key => allKeys.includes(key));
      
      if (keysToRemove.length > 0) {
        await AsyncStorage.multiRemove(keysToRemove);
        console.log(`Cleaned up ${keysToRemove.length} obsolete keys`);
      }
    } catch (error) {
      console.error('Error cleaning up obsolete data:', error);
      // No lanzamos error aquí porque la limpieza no es crítica
    }
  }

  // Método para obtener la versión actual de la base de datos
  async getCurrentVersion() {
    try {
      const version = await AsyncStorage.getItem(MIGRATION_VERSION_KEY);
      return version ? parseInt(version, 10) : 0;
    } catch (error) {
      console.error('Error getting current version:', error);
      return 0;
    }
  }

  // Método para forzar una migración a una versión específica
  async forceMigrationToVersion(targetVersion) {
    try {
      console.log(`Forcing migration to version ${targetVersion}...`);
      
      if (targetVersion < 1 || targetVersion > CURRENT_APP_VERSION) {
        throw new Error(`Invalid target version: ${targetVersion}`);
      }
      
      // Ejecutamos la migración
      await this.migrateToVersion(targetVersion);
      
      // Actualizamos la versión
      await AsyncStorage.setItem(MIGRATION_VERSION_KEY, targetVersion.toString());
      this.currentVersion = targetVersion;
      
      console.log(`Successfully forced migration to version ${targetVersion}`);
    } catch (error) {
      console.error(`Forced migration to version ${targetVersion} failed:`, error);
      throw error;
    }
  }

  // Método para verificar si la base de datos está actualizada
  async isDatabaseUpToDate() {
    try {
      const currentVersion = await this.getCurrentVersion();
      return currentVersion >= CURRENT_APP_VERSION;
    } catch (error) {
      console.error('Error checking if database is up to date:', error);
      return false;
    }
  }

  // Método para obtener información de migración
  async getMigrationInfo() {
    try {
      const currentVersion = await this.getCurrentVersion();
      
      return {
        currentVersion,
        targetVersion: CURRENT_APP_VERSION,
        needsMigration: currentVersion < CURRENT_APP_VERSION,
        isUpToDate: currentVersion >= CURRENT_APP_VERSION,
        pendingVersions: currentVersion < CURRENT_APP_VERSION 
          ? Array.from({ length: CURRENT_APP_VERSION - currentVersion }, (_, i) => currentVersion + i + 1)
          : []
      };
    } catch (error) {
      console.error('Error getting migration info:', error);
      return null;
    }
  }
}

// Exportamos una instancia singleton del servicio
export default new AsyncStorageMigrationService();
```

### 2. Migraciones en SQLite

```javascript
// database/migrations/sqliteMigrations.js
import SQLite from 'react-native-sqlite-storage';

export class SQLiteMigrationService {
  constructor(db) {
    this.db = db;
    this.currentVersion = 0;
  }

  // Método para inicializar el sistema de migraciones
  async initialize() {
    try {
      // Creamos la tabla de versiones si no existe
      await this.createVersionTable();
      
      // Obtenemos la versión actual
      await this.getCurrentVersion();
      
      // Ejecutamos migraciones si es necesario
      await this.performMigrations();
      
      console.log('SQLite migration service initialized successfully');
    } catch (error) {
      console.error('Error initializing SQLite migration service:', error);
      throw error;
    }
  }

  // Método para crear la tabla de versiones
  async createVersionTable() {
    try {
      const createVersionTableQuery = `
        CREATE TABLE IF NOT EXISTS schema_versions (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          version INTEGER NOT NULL,
          applied_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          description TEXT
        );
      `;
      
      await this.db.executeSql(createVersionTableQuery);
      console.log('Version table created or already exists');
    } catch (error) {
      console.error('Error creating version table:', error);
      throw error;
    }
  }

  // Método para obtener la versión actual
  async getCurrentVersion() {
    try {
      const query = 'SELECT MAX(version) as current_version FROM schema_versions';
      const [results] = await this.db.executeSql(query);
      
      if (results.rows.length > 0) {
        this.currentVersion = results.rows.item(0).current_version || 0;
      } else {
        this.currentVersion = 0;
      }
      
      console.log(`Current SQLite schema version: ${this.currentVersion}`);
    } catch (error) {
      console.error('Error getting current version:', error);
      this.currentVersion = 0;
    }
  }

  // Método para ejecutar migraciones
  async performMigrations() {
    try {
      const targetVersion = 3; // Versión objetivo del esquema
      
      if (this.currentVersion < targetVersion) {
        console.log('Starting SQLite schema migrations...');
        
        for (let version = this.currentVersion + 1; version <= targetVersion; version++) {
          await this.migrateToVersion(version);
        }
        
        console.log('SQLite schema migrations completed successfully');
      } else {
        console.log('SQLite schema is up to date');
      }
    } catch (error) {
      console.error('Error performing SQLite migrations:', error);
      throw error;
    }
  }

  // Método para migrar a una versión específica
  async migrateToVersion(targetVersion) {
    try {
      console.log(`Migrating SQLite schema to version ${targetVersion}...`);
      
      switch (targetVersion) {
        case 1:
          await this.migrateToV1();
          break;
        case 2:
          await this.migrateToV2();
          break;
        case 3:
          await this.migrateToV3();
          break;
        default:
          console.warn(`No SQLite migration defined for version ${targetVersion}`);
      }
      
      // Registramos la migración
      await this.recordMigration(targetVersion);
      this.currentVersion = targetVersion;
      
      console.log(`Successfully migrated SQLite schema to version ${targetVersion}`);
    } catch (error) {
      console.error(`SQLite migration to version ${targetVersion} failed:`, error);
      throw error;
    }
  }

  // Migración a la versión 1: Tablas básicas
  async migrateToV1() {
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
      
      // Tabla de categorías
      const createCategoriesTable = `
        CREATE TABLE IF NOT EXISTS categories (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          description TEXT,
          is_active BOOLEAN DEFAULT 1,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
      `;
      
      await this.db.executeSql(createUsersTable);
      await this.db.executeSql(createCategoriesTable);
      
      console.log('SQLite schema version 1 created successfully');
    } catch (error) {
      console.error('Error creating SQLite schema version 1:', error);
      throw error;
    }
  }

  // Migración a la versión 2: Tabla de productos
  async migrateToV2() {
    try {
      const createProductsTable = `
        CREATE TABLE IF NOT EXISTS products (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          description TEXT,
          price REAL NOT NULL,
          category_id INTEGER,
          stock_quantity INTEGER DEFAULT 0,
          is_available BOOLEAN DEFAULT 1,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          FOREIGN KEY (category_id) REFERENCES categories (id)
        );
      `;
      
      await this.db.executeSql(createProductsTable);
      
      // Creamos índices para mejorar el rendimiento
      const createIndexes = [
        'CREATE INDEX IF NOT EXISTS idx_products_category ON products(category_id)',
        'CREATE INDEX IF NOT EXISTS idx_products_name ON products(name)',
        'CREATE INDEX IF NOT EXISTS idx_products_price ON products(price)'
      ];
      
      for (const indexQuery of createIndexes) {
        await this.db.executeSql(indexQuery);
      }
      
      console.log('SQLite schema version 2 created successfully');
    } catch (error) {
      console.error('Error creating SQLite schema version 2:', error);
      throw error;
    }
  }

  // Migración a la versión 3: Tabla de pedidos
  async migrateToV3() {
    try {
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
      
      // Tabla de items de pedido
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
      
      await this.db.executeSql(createOrdersTable);
      await this.db.executeSql(createOrderItemsTable);
      
      // Creamos índices adicionales
      const createIndexes = [
        'CREATE INDEX IF NOT EXISTS idx_orders_user ON orders(user_id)',
        'CREATE INDEX IF NOT EXISTS idx_orders_status ON orders(status)',
        'CREATE INDEX IF NOT EXISTS idx_order_items_order ON order_items(order_id)',
        'CREATE INDEX IF NOT EXISTS idx_order_items_product ON order_items(product_id)'
      ];
      
      for (const indexQuery of createIndexes) {
        await this.db.executeSql(indexQuery);
      }
      
      console.log('SQLite schema version 3 created successfully');
    } catch (error) {
      console.error('Error creating SQLite schema version 3:', error);
      throw error;
    }
  }

  // Método para registrar una migración
  async recordMigration(version) {
    try {
      const query = `
        INSERT INTO schema_versions (version, description) 
        VALUES (?, ?)
      `;
      
      const description = `Migration to schema version ${version}`;
      await this.db.executeSql(query, [version, description]);
      
      console.log(`Migration to version ${version} recorded`);
    } catch (error) {
      console.error('Error recording migration:', error);
      throw error;
    }
  }

  // Método para obtener información de migración
  async getMigrationInfo() {
    try {
      const query = `
        SELECT version, applied_at, description 
        FROM schema_versions 
        ORDER BY version DESC
      `;
      
      const [results] = await this.db.executeSql(query);
      
      const migrations = [];
      for (let i = 0; i < results.rows.length; i++) {
        migrations.push(results.rows.item(i));
      }
      
      return {
        currentVersion: this.currentVersion,
        migrations,
        isUpToDate: this.currentVersion >= 3
      };
    } catch (error) {
      console.error('Error getting migration info:', error);
      return null;
    }
  }
}
```

### 3. Migraciones en Realm

```javascript
// database/migrations/realmMigrations.js
import Realm from 'realm';

// Configuración de migración para Realm
export const createRealmConfig = (schemaVersion) => {
  return {
    schema: [
      // Aquí irían todos los esquemas actualizados
    ],
    schemaVersion,
    migration: (oldRealm, newRealm) => {
      console.log(`Migrating Realm from version ${oldRealm.schemaVersion} to ${newRealm.schemaVersion}`);
      
      // Ejecutamos migraciones específicas por versión
      if (oldRealm.schemaVersion < 1) {
        migrateToV1(oldRealm, newRealm);
      }
      
      if (oldRealm.schemaVersion < 2) {
        migrateToV2(oldRealm, newRealm);
      }
      
      if (oldRealm.schemaVersion < 3) {
        migrateToV3(oldRealm, newRealm);
      }
      
      console.log('Realm migration completed successfully');
    }
  };
};

// Migración a la versión 1
function migrateToV1(oldRealm, newRealm) {
  console.log('Executing Realm migration to version 1...');
  
  try {
    // Si no existe la colección de usuarios, la creamos
    if (!oldRealm.objects('User')) {
      console.log('Creating User collection...');
      // Realm creará automáticamente la colección
    }
    
    // Si no existe la colección de categorías, la creamos
    if (!oldRealm.objects('Category')) {
      console.log('Creating Category collection...');
      // Realm creará automáticamente la colección
    }
    
    console.log('Realm migration to version 1 completed');
  } catch (error) {
    console.error('Realm migration to version 1 failed:', error);
    throw error;
  }
}

// Migración a la versión 2
function migrateToV2(oldRealm, newRealm) {
  console.log('Executing Realm migration to version 2...');
  
  try {
    // Agregamos nuevos campos a usuarios existentes
    const users = newRealm.objects('User');
    
    for (const user of users) {
      // Agregamos campo de preferencias si no existe
      if (!user.hasOwnProperty('preferences')) {
        user.preferences = {
          theme: 'light',
          language: 'es',
          notifications: true
        };
      }
      
      // Agregamos campo de perfil si no existe
      if (!user.hasOwnProperty('profile')) {
        user.profile = {
          avatar: null,
          phone: '',
          address: '',
          birthDate: null
        };
      }
    }
    
    // Agregamos nuevos campos a categorías existentes
    const categories = newRealm.objects('Category');
    
    for (const category of categories) {
      // Agregamos campo de imagen si no existe
      if (!category.hasOwnProperty('imageUrl')) {
        category.imageUrl = null;
      }
      
      // Agregamos campo de estado activo si no existe
      if (!category.hasOwnProperty('isActive')) {
        category.isActive = true;
      }
    }
    
    console.log('Realm migration to version 2 completed');
  } catch (error) {
    console.error('Realm migration to version 2 failed:', error);
    throw error;
  }
}

// Migración a la versión 3
function migrateToV3(oldRealm, newRealm) {
  console.log('Executing Realm migration to version 3...');
  
  try {
    // Agregamos nuevos campos a productos existentes
    const products = newRealm.objects('Product');
    
    for (const product of products) {
      // Agregamos campo de etiquetas si no existe
      if (!product.hasOwnProperty('tags')) {
        product.tags = [];
      }
      
      // Agregamos campo de especificaciones si no existe
      if (!product.hasOwnProperty('specifications')) {
        product.specifications = {};
      }
      
      // Agregamos campo de calificación si no existe
      if (!product.hasOwnProperty('rating')) {
        product.rating = null;
      }
      
      // Agregamos campo de número de reseñas si no existe
      if (!product.hasOwnProperty('reviewCount')) {
        product.reviewCount = 0;
      }
    }
    
    // Agregamos nuevos campos a usuarios existentes
    const users = newRealm.objects('User');
    
    for (const user of users) {
      // Agregamos campo de historial si no existe
      if (!user.hasOwnProperty('history')) {
        user.history = {
          lastViewedProducts: [],
          searchHistory: [],
          favoriteProducts: [],
          recentSearches: []
        };
      }
      
      // Agregamos campo de configuración avanzada si no existe
      if (!user.hasOwnProperty('advancedSettings')) {
        user.advancedSettings = {
          dataSync: true,
          analytics: false,
          crashReporting: true,
          performanceMonitoring: true
        };
      }
    }
    
    console.log('Realm migration to version 3 completed');
  } catch (error) {
    console.error('Realm migration to version 3 failed:', error);
    throw error;
  }
}
```

---

## 💡 Casos de Uso Prácticos

### 1. Hook para Manejar Migraciones

```javascript
// hooks/useMigrations.js
import { useState, useEffect } from 'react';
import AsyncStorageMigrationService from '../database/migrations/asyncStorageMigrations';

export const useMigrations = () => {
  const [migrationState, setMigrationState] = useState({
    isInitialized: false,
    isMigrating: false,
    currentVersion: 0,
    targetVersion: 0,
    needsMigration: false,
    error: null
  });

  useEffect(() => {
    initializeMigrations();
  }, []);

  const initializeMigrations = async () => {
    try {
      setMigrationState(prev => ({ ...prev, isMigrating: true }));
      
      // Inicializamos el servicio de migración
      await AsyncStorageMigrationService.initialize();
      
      // Obtenemos información de migración
      const migrationInfo = await AsyncStorageMigrationService.getMigrationInfo();
      
      setMigrationState({
        isInitialized: true,
        isMigrating: false,
        currentVersion: migrationInfo.currentVersion,
        targetVersion: migrationInfo.targetVersion,
        needsMigration: migrationInfo.needsMigration,
        error: null
      });
    } catch (error) {
      setMigrationState(prev => ({
        ...prev,
        isInitialized: true,
        isMigrating: false,
        error: error.message
      }));
    }
  };

  const forceMigration = async (targetVersion) => {
    try {
      setMigrationState(prev => ({ ...prev, isMigrating: true }));
      
      await AsyncStorageMigrationService.forceMigrationToVersion(targetVersion);
      
      // Recargamos la información de migración
      const migrationInfo = await AsyncStorageMigrationService.getMigrationInfo();
      
      setMigrationState(prev => ({
        ...prev,
        isMigrating: false,
        currentVersion: migrationInfo.currentVersion,
        needsMigration: migrationInfo.needsMigration
      }));
    } catch (error) {
      setMigrationState(prev => ({
        ...prev,
        isMigrating: false,
        error: error.message
      }));
    }
  };

  return {
    ...migrationState,
    forceMigration
  };
};
```

### 2. Componente de Estado de Migración

```javascript
// components/MigrationStatus.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useMigrations } from '../hooks/useMigrations';

const MigrationStatus = () => {
  const {
    isInitialized,
    isMigrating,
    currentVersion,
    targetVersion,
    needsMigration,
    error,
    forceMigration
  } = useMigrations();

  const handleForceMigration = () => {
    Alert.alert(
      'Forzar Migración',
      `¿Estás seguro de que quieres migrar a la versión ${targetVersion}? Esta acción no se puede deshacer.`,
      [
        { text: 'Cancelar', style: 'cancel' },
        {
          text: 'Migrar',
          style: 'destructive',
          onPress: () => forceMigration(targetVersion)
        }
      ]
    );
  };

  if (!isInitialized) {
    return (
      <View style={styles.container}>
        <Text>Inicializando migraciones...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.errorContainer}>
        <Text style={styles.errorText}>Error: {error}</Text>
        <TouchableOpacity
          style={styles.retryButton}
          onPress={initializeMigrations}
        >
          <Text style={styles.buttonText}>Reintentar</Text>
        </TouchableOpacity>
      </View>
    );
  }

  if (isMigrating) {
    return (
      <View style={styles.container}>
        <Text>Migrando base de datos...</Text>
        <Text>Versión actual: {currentVersion}</Text>
        <Text>Versión objetivo: {targetVersion}</Text>
      </View>
    );
  }

  if (needsMigration) {
    return (
      <View style={styles.warningContainer}>
        <Text style={styles.warningText}>
          La base de datos necesita ser actualizada
        </Text>
        <Text>Versión actual: {currentVersion}</Text>
        <Text>Versión disponible: {targetVersion}</Text>
        <TouchableOpacity
          style={styles.migrateButton}
          onPress={handleForceMigration}
        >
          <Text style={styles.buttonText}>Actualizar Ahora</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.successContainer}>
      <Text style={styles.successText}>
        Base de datos actualizada ✓
      </Text>
      <Text>Versión actual: {currentVersion}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
    margin: 16,
  },
  errorContainer: {
    padding: 16,
    backgroundColor: '#ffebee',
    borderRadius: 8,
    margin: 16,
  },
  warningContainer: {
    padding: 16,
    backgroundColor: '#fff3e0',
    borderRadius: 8,
    margin: 16,
  },
  successContainer: {
    padding: 16,
    backgroundColor: '#e8f5e8',
    borderRadius: 8,
    margin: 16,
  },
  errorText: {
    color: '#c62828',
    fontWeight: 'bold',
    marginBottom: 8,
  },
  warningText: {
    color: '#ef6c00',
    fontWeight: 'bold',
    marginBottom: 8,
  },
  successText: {
    color: '#2e7d32',
    fontWeight: 'bold',
    marginBottom: 8,
  },
  migrateButton: {
    backgroundColor: '#ff9800',
    padding: 12,
    borderRadius: 6,
    marginTop: 12,
    alignItems: 'center',
  },
  retryButton: {
    backgroundColor: '#f44336',
    padding: 12,
    borderRadius: 6,
    marginTop: 12,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
  },
});

export default MigrationStatus;
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Migraciones Básico**
1. Implementa el servicio de migraciones para AsyncStorage
2. Crea migraciones para 3 versiones diferentes
3. Prueba la migración automática al inicializar la app
4. Implementa manejo de errores robusto

### **Ejercicio 2: Migraciones en SQLite**
1. Implementa el servicio de migraciones para SQLite
2. Crea tablas con diferentes versiones del esquema
3. Agrega índices y relaciones en migraciones posteriores
4. Prueba la integridad referencial después de las migraciones

### **Ejercicio 3: Migraciones en Realm**
1. Implementa migraciones para Realm con diferentes versiones
2. Agrega nuevos campos a esquemas existentes
3. Maneja cambios en tipos de datos
4. Prueba la compatibilidad hacia atrás

### **Ejercicio 4: Testing de Migraciones**
1. Crea tests unitarios para cada migración
2. Prueba escenarios de migración fallida
3. Implementa rollback de migraciones
4. Crea tests de integración para el sistema completo

### **Ejercicio 5: Estrategias Avanzadas**
1. Implementa migraciones condicionales
2. Crea migraciones que dependan de datos existentes
3. Implementa migraciones en lote para grandes volúmenes
4. Crea un sistema de logging detallado de migraciones

---

## 🎯 Proyecto de la Clase

### **Sistema de Migraciones Completo**

Crea una aplicación que demuestre:
1. **Migraciones automáticas** en AsyncStorage
2. **Versionado de esquemas** en SQLite
3. **Migraciones de objetos** en Realm
4. **Interfaz de usuario** para monitorear migraciones
5. **Testing completo** del sistema de migraciones

**Estructura del Proyecto:**
```
src/
├── database/
│   ├── migrations/
│   │   ├── asyncStorageMigrations.js
│   │   ├── sqliteMigrations.js
│   │   └── realmMigrations.js
│   ├── sqlite.js
│   ├── realm.js
│   └── schemas.js
├── hooks/
│   └── useMigrations.js
├── components/
│   └── MigrationStatus.js
├── services/
│   └── migrationService.js
└── screens/
    └── MigrationScreen.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [Realm Migrations](https://docs.mongodb.com/realm/sdk/javascript/advanced-guides/migrations/)
- [SQLite Schema Changes](https://www.sqlite.org/lang_altertable.html)

### **Mejores Prácticas:**
- **Planificación**: Diseña migraciones antes de implementarlas
- **Testing**: Prueba migraciones con datos reales
- **Backup**: Siempre haz backup antes de migrar
- **Rollback**: Planifica cómo revertir migraciones fallidas

### **Estrategias de Migración:**
- **Forward-only**: Migraciones que solo van hacia adelante
- **Rollback**: Migraciones que se pueden revertir
- **Blue-green**: Migración sin tiempo de inactividad
- **Canary**: Migración gradual a un subconjunto de usuarios

---

## 🔍 Resumen de la Clase

### **Conceptos Clave:**
- ✅ Las migraciones son esenciales para preservar datos entre versiones
- ✅ AsyncStorage requiere migraciones manuales con lógica de negocio
- ✅ SQLite permite migraciones automáticas con DDL
- ✅ Realm tiene un sistema de migraciones integrado y reactivo
- ✅ Siempre planifica y prueba las migraciones antes de implementarlas

### **Próximos Pasos:**
- 🔄 Completar todos los ejercicios de esta clase
- 🔄 Implementar el sistema completo de migraciones
- 🔄 Practicar con diferentes escenarios de migración
- 🔄 Prepararse para la siguiente clase sobre sincronización

---

## 🚀 Siguiente Clase

En la **Clase 5: Sincronización de Datos**, aprenderás:
- Cómo sincronizar datos entre dispositivos
- Implementación de sincronización offline/online
- Manejo de conflictos de sincronización
- Estrategias de sincronización eficientes
- Testing de sincronización

---

**¡Excelente trabajo con las migraciones! 🎉**

**Recuerda**: Las migraciones son críticas para la evolución de tu aplicación. Planifícalas bien y siempre haz testing exhaustivo.
