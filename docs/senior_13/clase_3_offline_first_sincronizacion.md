# 📱 Clase 3: Offline First y Sincronización

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar una arquitectura offline-first, manejar persistencia local de datos, configurar sincronización automática cuando hay conexión, y resolver conflictos de datos de manera elegante. Crearás una experiencia de usuario fluida que funcione perfectamente tanto online como offline.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** arquitectura offline-first
2. **Manejar** persistencia local con IndexedDB
3. **Configurar** sincronización automática de datos
4. **Resolver** conflictos de datos de manera inteligente
5. **Crear** colas de operaciones offline

---

## 📚 Contenido de la Clase

### **1. Arquitectura Offline-First**

#### **¿Qué es Offline-First?**
Offline-first es un enfoque de desarrollo donde la aplicación está diseñada para funcionar completamente offline, con sincronización como una característica adicional cuando hay conexión.

#### **Principios Fundamentales**
- **Datos locales primero**: Todos los datos se almacenan localmente
- **Sincronización opcional**: La conexión mejora la experiencia pero no es requerida
- **Resiliencia**: La app funciona independientemente del estado de la red
- **Consistencia eventual**: Los datos se sincronizan cuando es posible
- **Experiencia fluida**: Sin interrupciones por cambios de conectividad

#### **Arquitectura de Datos**
```javascript
// Arquitectura offline-first
const OfflineFirstArchitecture = {
  local: {
    storage: 'IndexedDB',
    cache: 'Cache API',
    queue: 'Operation Queue'
  },
  sync: {
    strategy: 'Background Sync',
    conflict: 'Last Write Wins / Merge',
    retry: 'Exponential Backoff'
  },
  ui: {
    indicators: 'Connection Status',
    feedback: 'Sync Progress',
    fallbacks: 'Offline Mode'
  }
};
```

### **2. Persistencia Local con IndexedDB**

#### **Configuración de IndexedDB**
```javascript
// db.js - Configuración de IndexedDB
class OfflineDatabase {
  constructor() {
    this.dbName = 'PWAOfflineDB';
    this.version = 1;
    this.db = null;
  }

  async init() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        // Store para usuarios
        if (!db.objectStoreNames.contains('users')) {
          const userStore = db.createObjectStore('users', { keyPath: 'id' });
          userStore.createIndex('email', 'email', { unique: true });
        }
        
        // Store para posts
        if (!db.objectStoreNames.contains('posts')) {
          const postStore = db.createObjectStore('posts', { keyPath: 'id' });
          postStore.createIndex('userId', 'userId', { unique: false });
          postStore.createIndex('createdAt', 'createdAt', { unique: false });
        }
        
        // Store para operaciones pendientes
        if (!db.objectStoreNames.contains('pendingOperations')) {
          const pendingStore = db.createObjectStore('pendingOperations', { 
            keyPath: 'id', 
            autoIncrement: true 
          });
          pendingStore.createIndex('type', 'type', { unique: false });
          pendingStore.createIndex('timestamp', 'timestamp', { unique: false });
        }
      };
    });
  }

  // Métodos CRUD genéricos
  async add(storeName, data) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    return store.add(data);
  }

  async get(storeName, key) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    return store.get(key);
  }

  async getAll(storeName) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    return store.getAll();
  }

  async update(storeName, data) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    return store.put(data);
  }

  async delete(storeName, key) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    return store.delete(key);
  }
}
```

#### **Operaciones Offline**
```javascript
// offlineOperations.js - Manejo de operaciones offline
class OfflineOperations {
  constructor(database) {
    this.db = database;
    this.syncQueue = [];
    this.isOnline = navigator.onLine;
    this.setupEventListeners();
  }

  setupEventListeners() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.processSyncQueue();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }

  // Agregar operación a la cola
  async queueOperation(operation) {
    const pendingOp = {
      id: Date.now() + Math.random(),
      type: operation.type,
      data: operation.data,
      timestamp: Date.now(),
      retries: 0,
      maxRetries: 3
    };

    await this.db.add('pendingOperations', pendingOp);
    this.syncQueue.push(pendingOp);

    if (this.isOnline) {
      this.processSyncQueue();
    }
  }

  // Procesar cola de sincronización
  async processSyncQueue() {
    if (!this.isOnline || this.syncQueue.length === 0) {
      return;
    }

    const operations = [...this.syncQueue];
    this.syncQueue = [];

    for (const operation of operations) {
      try {
        await this.executeOperation(operation);
        await this.db.delete('pendingOperations', operation.id);
      } catch (error) {
        console.error('Error sincronizando operación:', error);
        await this.handleSyncError(operation, error);
      }
    }
  }

  // Ejecutar operación individual
  async executeOperation(operation) {
    switch (operation.type) {
      case 'CREATE_POST':
        return this.createPost(operation.data);
      case 'UPDATE_POST':
        return this.updatePost(operation.data);
      case 'DELETE_POST':
        return this.deletePost(operation.data.id);
      default:
        throw new Error(`Tipo de operación no soportado: ${operation.type}`);
    }
  }

  // Manejar errores de sincronización
  async handleSyncError(operation, error) {
    operation.retries++;
    
    if (operation.retries < operation.maxRetries) {
      // Reintentar con backoff exponencial
      const delay = Math.pow(2, operation.retries) * 1000;
      setTimeout(() => {
        this.syncQueue.push(operation);
        this.processSyncQueue();
      }, delay);
    } else {
      // Marcar como fallida permanentemente
      console.error('Operación fallida permanentemente:', operation);
      await this.db.delete('pendingOperations', operation.id);
    }
  }
}
```

### **3. Sincronización de Datos**

#### **Estrategias de Sincronización**
```javascript
// syncStrategies.js - Estrategias de sincronización
class SyncStrategies {
  constructor(database) {
    this.db = database;
  }

  // Last Write Wins - Última escritura gana
  async lastWriteWins(localData, remoteData) {
    const localTimestamp = new Date(localData.updatedAt).getTime();
    const remoteTimestamp = new Date(remoteData.updatedAt).getTime();
    
    return remoteTimestamp > localTimestamp ? remoteData : localData;
  }

  // Merge Strategy - Combinar cambios
  async mergeStrategy(localData, remoteData) {
    const merged = { ...localData };
    
    // Combinar campos no conflictivos
    Object.keys(remoteData).forEach(key => {
      if (key !== 'id' && key !== 'updatedAt') {
        if (localData[key] !== remoteData[key]) {
          // Resolver conflicto por prioridad o lógica de negocio
          merged[key] = this.resolveConflict(key, localData[key], remoteData[key]);
        }
      }
    });
    
    merged.updatedAt = new Date().toISOString();
    merged.conflictResolved = true;
    
    return merged;
  }

  // Resolver conflictos específicos
  resolveConflict(field, localValue, remoteValue) {
    switch (field) {
      case 'title':
        // Para títulos, preferir el más largo (más detallado)
        return localValue.length > remoteValue.length ? localValue : remoteValue;
      case 'content':
        // Para contenido, preferir el más reciente
        return remoteValue;
      case 'tags':
        // Para tags, combinar ambos
        const localTags = Array.isArray(localValue) ? localValue : [];
        const remoteTags = Array.isArray(remoteValue) ? remoteValue : [];
        return [...new Set([...localTags, ...remoteTags])];
      default:
        return remoteValue;
    }
  }

  // Sincronización bidireccional
  async bidirectionalSync() {
    try {
      // Obtener datos locales modificados
      const localChanges = await this.getLocalChanges();
      
      // Enviar cambios locales al servidor
      for (const change of localChanges) {
        await this.pushToServer(change);
      }
      
      // Obtener cambios del servidor
      const serverChanges = await this.pullFromServer();
      
      // Aplicar cambios del servidor
      for (const change of serverChanges) {
        await this.applyServerChange(change);
      }
      
    } catch (error) {
      console.error('Error en sincronización bidireccional:', error);
      throw error;
    }
  }

  // Obtener cambios locales
  async getLocalChanges() {
    const allData = await this.db.getAll('posts');
    return allData.filter(item => 
      item.localModified && !item.synced
    );
  }

  // Aplicar cambio del servidor
  async applyServerChange(serverData) {
    const localData = await this.db.get('posts', serverData.id);
    
    if (!localData) {
      // Nuevo dato del servidor
      await this.db.add('posts', {
        ...serverData,
        synced: true,
        localModified: false
      });
    } else if (!localData.localModified) {
      // Actualizar dato local sin conflictos
      await this.db.update('posts', {
        ...serverData,
        synced: true,
        localModified: false
      });
    } else {
      // Resolver conflicto
      const resolved = await this.mergeStrategy(localData, serverData);
      await this.db.update('posts', resolved);
    }
  }
}
```

### **4. Resolución de Conflictos**

#### **Detección de Conflictos**
```javascript
// conflictResolution.js - Resolución de conflictos
class ConflictResolution {
  constructor(database) {
    this.db = database;
  }

  // Detectar conflictos
  async detectConflicts(localData, remoteData) {
    const conflicts = [];
    
    Object.keys(remoteData).forEach(key => {
      if (key !== 'id' && key !== 'updatedAt') {
        if (localData[key] !== remoteData[key]) {
          conflicts.push({
            field: key,
            localValue: localData[key],
            remoteValue: remoteData[key],
            localTimestamp: localData.updatedAt,
            remoteTimestamp: remoteData.updatedAt
          });
        }
      }
    });
    
    return conflicts;
  }

  // Resolver conflictos automáticamente
  async autoResolveConflicts(conflicts, strategy = 'lastWriteWins') {
    const resolved = {};
    
    for (const conflict of conflicts) {
      switch (strategy) {
        case 'lastWriteWins':
          resolved[conflict.field] = this.lastWriteWins(conflict);
          break;
        case 'localWins':
          resolved[conflict.field] = conflict.localValue;
          break;
        case 'remoteWins':
          resolved[conflict.field] = conflict.remoteValue;
          break;
        case 'merge':
          resolved[conflict.field] = this.mergeValues(conflict);
          break;
        default:
          resolved[conflict.field] = conflict.remoteValue;
      }
    }
    
    return resolved;
  }

  // Última escritura gana
  lastWriteWins(conflict) {
    const localTime = new Date(conflict.localTimestamp).getTime();
    const remoteTime = new Date(conflict.remoteTimestamp).getTime();
    
    return remoteTime > localTime ? conflict.remoteValue : conflict.localValue;
  }

  // Combinar valores
  mergeValues(conflict) {
    const { field, localValue, remoteValue } = conflict;
    
    switch (field) {
      case 'tags':
        const localTags = Array.isArray(localValue) ? localValue : [];
        const remoteTags = Array.isArray(remoteValue) ? remoteValue : [];
        return [...new Set([...localTags, ...remoteTags])];
      
      case 'content':
        // Para contenido, combinar con separador
        return `${localValue}\n\n---\n\n${remoteValue}`;
      
      case 'title':
        // Para títulos, usar el más descriptivo
        return localValue.length > remoteValue.length ? localValue : remoteValue;
      
      default:
        return remoteValue;
    }
  }

  // Resolución manual de conflictos
  async manualConflictResolution(conflicts) {
    return new Promise((resolve) => {
      // Mostrar UI para resolución manual
      const conflictUI = this.createConflictUI(conflicts);
      document.body.appendChild(conflictUI);
      
      // Escuchar resolución del usuario
      conflictUI.addEventListener('resolve', (event) => {
        document.body.removeChild(conflictUI);
        resolve(event.detail.resolution);
      });
    });
  }

  // Crear UI para resolución de conflictos
  createConflictUI(conflicts) {
    const container = document.createElement('div');
    container.className = 'conflict-resolution-modal';
    
    conflicts.forEach(conflict => {
      const conflictElement = document.createElement('div');
      conflictElement.className = 'conflict-item';
      
      conflictElement.innerHTML = `
        <h3>Conflicto en: ${conflict.field}</h3>
        <div class="conflict-options">
          <label>
            <input type="radio" name="${conflict.field}" value="local">
            Local: ${conflict.localValue}
          </label>
          <label>
            <input type="radio" name="${conflict.field}" value="remote">
            Remoto: ${conflict.remoteValue}
          </label>
          <label>
            <input type="radio" name="${conflict.field}" value="merge">
            Combinar
          </label>
        </div>
      `;
      
      container.appendChild(conflictElement);
    });
    
    const resolveButton = document.createElement('button');
    resolveButton.textContent = 'Resolver Conflictos';
    resolveButton.addEventListener('click', () => {
      const resolution = this.collectResolution(conflicts);
      container.dispatchEvent(new CustomEvent('resolve', { detail: { resolution } }));
    });
    
    container.appendChild(resolveButton);
    return container;
  }
}
```

### **5. Colas de Operaciones**

#### **Sistema de Colas**
```javascript
// operationQueue.js - Sistema de colas de operaciones
class OperationQueue {
  constructor(database) {
    this.db = database;
    this.queue = [];
    this.isProcessing = false;
    this.maxRetries = 3;
    this.retryDelay = 1000; // 1 segundo
  }

  // Agregar operación a la cola
  async enqueue(operation) {
    const queuedOperation = {
      id: this.generateId(),
      type: operation.type,
      data: operation.data,
      timestamp: Date.now(),
      retries: 0,
      status: 'pending',
      priority: operation.priority || 'normal'
    };

    await this.db.add('operationQueue', queuedOperation);
    this.queue.push(queuedOperation);
    
    // Procesar cola si no está procesando
    if (!this.isProcessing) {
      this.processQueue();
    }
  }

  // Procesar cola de operaciones
  async processQueue() {
    if (this.isProcessing || this.queue.length === 0) {
      return;
    }

    this.isProcessing = true;

    // Ordenar por prioridad
    this.queue.sort((a, b) => {
      const priorityOrder = { high: 3, normal: 2, low: 1 };
      return priorityOrder[b.priority] - priorityOrder[a.priority];
    });

    while (this.queue.length > 0) {
      const operation = this.queue.shift();
      
      try {
        await this.executeOperation(operation);
        await this.db.delete('operationQueue', operation.id);
      } catch (error) {
        await this.handleOperationError(operation, error);
      }
    }

    this.isProcessing = false;
  }

  // Ejecutar operación individual
  async executeOperation(operation) {
    switch (operation.type) {
      case 'CREATE':
        return this.createOperation(operation.data);
      case 'UPDATE':
        return this.updateOperation(operation.data);
      case 'DELETE':
        return this.deleteOperation(operation.data);
      case 'SYNC':
        return this.syncOperation(operation.data);
      default:
        throw new Error(`Tipo de operación no soportado: ${operation.type}`);
    }
  }

  // Manejar errores de operación
  async handleOperationError(operation, error) {
    operation.retries++;
    operation.lastError = error.message;

    if (operation.retries < this.maxRetries) {
      // Reintentar con backoff exponencial
      const delay = this.retryDelay * Math.pow(2, operation.retries - 1);
      
      setTimeout(() => {
        this.queue.push(operation);
        this.processQueue();
      }, delay);
    } else {
      // Marcar como fallida
      operation.status = 'failed';
      await this.db.update('operationQueue', operation);
      console.error('Operación fallida permanentemente:', operation);
    }
  }

  // Generar ID único
  generateId() {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }

  // Obtener estado de la cola
  getQueueStatus() {
    return {
      total: this.queue.length,
      pending: this.queue.filter(op => op.status === 'pending').length,
      processing: this.isProcessing,
      failed: this.queue.filter(op => op.status === 'failed').length
    };
  }
}
```

---

## 🛠️ Implementación Práctica

### **1. Hook de Offline-First**

#### **useOfflineFirst Hook**
```javascript
// useOfflineFirst.js
import { useState, useEffect, useCallback } from 'react';
import { OfflineDatabase } from './db';
import { OfflineOperations } from './offlineOperations';
import { SyncStrategies } from './syncStrategies';

export const useOfflineFirst = () => {
  const [db, setDb] = useState(null);
  const [operations, setOperations] = useState(null);
  const [sync, setSync] = useState(null);
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  const [syncStatus, setSyncStatus] = useState('idle');

  useEffect(() => {
    const initOfflineFirst = async () => {
      try {
        const database = new OfflineDatabase();
        await database.init();
        
        const offlineOps = new OfflineOperations(database);
        const syncStrategies = new SyncStrategies(database);
        
        setDb(database);
        setOperations(offlineOps);
        setSync(syncStrategies);
      } catch (error) {
        console.error('Error inicializando offline-first:', error);
      }
    };

    initOfflineFirst();

    // Escuchar cambios de conectividad
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  // Sincronizar datos
  const syncData = useCallback(async () => {
    if (!sync || !isOnline) return;
    
    setSyncStatus('syncing');
    try {
      await sync.bidirectionalSync();
      setSyncStatus('success');
    } catch (error) {
      setSyncStatus('error');
      console.error('Error sincronizando:', error);
    }
  }, [sync, isOnline]);

  // Agregar dato offline
  const addOfflineData = useCallback(async (storeName, data) => {
    if (!db || !operations) return;
    
    // Agregar localmente
    await db.add(storeName, {
      ...data,
      localModified: true,
      synced: false,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    });
    
    // Agregar a cola de operaciones
    await operations.queueOperation({
      type: 'CREATE',
      data: { storeName, data }
    });
  }, [db, operations]);

  // Actualizar dato offline
  const updateOfflineData = useCallback(async (storeName, id, updates) => {
    if (!db || !operations) return;
    
    const existing = await db.get(storeName, id);
    if (!existing) return;
    
    const updated = {
      ...existing,
      ...updates,
      localModified: true,
      synced: false,
      updatedAt: new Date().toISOString()
    };
    
    await db.update(storeName, updated);
    
    await operations.queueOperation({
      type: 'UPDATE',
      data: { storeName, id, updates }
    });
  }, [db, operations]);

  return {
    db,
    operations,
    sync,
    isOnline,
    syncStatus,
    syncData,
    addOfflineData,
    updateOfflineData
  };
};
```

### **2. Componente de Sincronización**

#### **SyncStatus Component**
```javascript
// SyncStatus.js
import React, { useEffect, useState } from 'react';
import { useOfflineFirst } from './useOfflineFirst';

const SyncStatus = () => {
  const { isOnline, syncStatus, syncData } = useOfflineFirst();
  const [lastSync, setLastSync] = useState(null);

  useEffect(() => {
    if (syncStatus === 'success') {
      setLastSync(new Date());
    }
  }, [syncStatus]);

  const handleSync = () => {
    syncData();
  };

  const getStatusIcon = () => {
    if (!isOnline) return '🔴';
    if (syncStatus === 'syncing') return '🔄';
    if (syncStatus === 'success') return '🟢';
    if (syncStatus === 'error') return '⚠️';
    return '🟡';
  };

  const getStatusText = () => {
    if (!isOnline) return 'Sin conexión';
    if (syncStatus === 'syncing') return 'Sincronizando...';
    if (syncStatus === 'success') return 'Sincronizado';
    if (syncStatus === 'error') return 'Error de sincronización';
    return 'Pendiente de sincronización';
  };

  return (
    <div className="sync-status">
      <div className="sync-indicator">
        <span className="status-icon">{getStatusIcon()}</span>
        <span className="status-text">{getStatusText()}</span>
      </div>
      
      {lastSync && (
        <div className="last-sync">
          Última sincronización: {lastSync.toLocaleTimeString()}
        </div>
      )}
      
      {isOnline && syncStatus !== 'syncing' && (
        <button onClick={handleSync} className="sync-button">
          Sincronizar
        </button>
      )}
    </div>
  );
};

export default SyncStatus;
```

### **3. Testing de Offline-First**

#### **Testing de IndexedDB**
```javascript
// offlineFirst.test.js
import { OfflineDatabase } from './db';
import { OfflineOperations } from './offlineOperations';

// Mock de IndexedDB
const mockIndexedDB = {
  open: jest.fn(),
  deleteDatabase: jest.fn()
};

global.indexedDB = mockIndexedDB;

describe('Offline-First Architecture', () => {
  let db;
  let operations;

  beforeEach(async () => {
    // Mock de base de datos
    const mockDB = {
      transaction: jest.fn(),
      objectStoreNames: { contains: jest.fn() },
      createObjectStore: jest.fn()
    };

    mockIndexedDB.open.mockReturnValue({
      onsuccess: null,
      onerror: null,
      onupgradeneeded: null,
      result: mockDB
    });

    db = new OfflineDatabase();
    await db.init();
    
    operations = new OfflineOperations(db);
  });

  test('debe inicializar base de datos correctamente', async () => {
    expect(db).toBeDefined();
    expect(db.db).toBeDefined();
  });

  test('debe agregar operación a la cola offline', async () => {
    const operation = {
      type: 'CREATE_POST',
      data: { title: 'Test Post', content: 'Test Content' }
    };

    await operations.queueOperation(operation);
    
    expect(operations.syncQueue).toHaveLength(1);
    expect(operations.syncQueue[0].type).toBe('CREATE_POST');
  });

  test('debe procesar cola cuando hay conexión', async () => {
    // Mock de fetch
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ success: true })
    });

    const operation = {
      type: 'CREATE_POST',
      data: { title: 'Test Post', content: 'Test Content' }
    };

    await operations.queueOperation(operation);
    operations.isOnline = true;
    await operations.processSyncQueue();

    expect(operations.syncQueue).toHaveLength(0);
  });
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Base de Datos Offline**
Implementa una base de datos IndexedDB que:
- Almacene datos de usuarios y posts
- Maneje operaciones CRUD offline
- Implemente índices para búsquedas eficientes
- Maneje versionado de esquemas

### **Ejercicio 2: Sincronización Bidireccional**
Crea un sistema de sincronización que:
- Detecte conflictos automáticamente
- Implemente múltiples estrategias de resolución
- Maneje reintentos con backoff exponencial
- Proporcione feedback visual del estado

### **Ejercicio 3: Cola de Operaciones**
Implementa una cola de operaciones que:
- Priorice operaciones por importancia
- Maneje reintentos automáticos
- Proporcione estado de progreso
- Persista operaciones entre sesiones

---

## 📖 Recursos Adicionales

### **Documentación**
- [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Background Sync](https://developer.mozilla.org/en-US/docs/Web/API/Background_Sync_API)
- [Offline Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage)

### **Librerías**
- [Dexie.js](https://dexie.org/) - Wrapper para IndexedDB
- [LocalForage](https://localforage.github.io/localForage/) - Storage simplificado
- [PouchDB](https://pouchdb.com/) - Base de datos offline con sync

### **Ejemplos**
- [Offline-First Examples](https://github.com/offlinefirst)
- [PWA Offline Examples](https://github.com/GoogleChrome/samples/tree/gh-pages/pwa)

---

## 🚀 Próximos Pasos

En la siguiente clase aprenderás sobre **Push Notifications Web**, donde implementarás notificaciones push, configuración de VAPID keys, y estrategias de engagement para mantener a los usuarios conectados con tu PWA.

---

**💡 Consejo**: Offline-first no es solo una característica técnica, es una filosofía de diseño. Prioriza la experiencia del usuario offline y usa la conectividad para mejorar, no para habilitar funcionalidades.

**🎯 Objetivo**: Al final de esta clase, tendrás una aplicación que funciona perfectamente offline con sincronización inteligente y resolución de conflictos automática.
