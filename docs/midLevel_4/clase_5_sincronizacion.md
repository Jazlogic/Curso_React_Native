# 📚 Clase 5: Sincronización de Datos

## 🧭 Navegación del Módulo

- **⬅️ Anterior**: [Clase 4: Migraciones y Versionado](clase_4_migraciones.md)
- **➡️ Siguiente**: [Clase 6: Backup y Restauración](clase_6_backup_restauracion.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

- Comprender los conceptos de sincronización de datos
- Implementar sincronización offline/online
- Manejar conflictos de sincronización
- Crear estrategias de sincronización eficientes
- Implementar sincronización en tiempo real
- Testing de sincronización

---

## 📖 Contenido Teórico

### 1. ¿Qué es la Sincronización de Datos?

La sincronización de datos es el proceso de mantener la consistencia de datos entre múltiples fuentes, como:

- **Dispositivos móviles** (smartphones, tablets)
- **Servidores remotos** (APIs, bases de datos en la nube)
- **Aplicaciones web** (interfaces de administración)
- **Bases de datos locales** (almacenamiento offline)

**Tipos de sincronización:**
- **Unidireccional**: Los datos fluyen en una sola dirección
- **Bidireccional**: Los datos se sincronizan en ambas direcciones
- **En tiempo real**: Sincronización inmediata cuando hay cambios
- **Periódica**: Sincronización programada a intervalos regulares
- **Manual**: Sincronización iniciada por el usuario

### 2. ¿Por qué es Importante?

**Sin sincronización:**
- Los usuarios pierden datos al cambiar de dispositivo
- Inconsistencias entre diferentes fuentes de datos
- Experiencia de usuario fragmentada
- Pérdida de productividad en entornos offline

**Con sincronización:**
- Datos consistentes en todos los dispositivos
- Experiencia offline/online fluida
- Colaboración en tiempo real
- Resiliencia ante fallos de red

---

## 🛠️ Implementación Práctica

### 1. Servicio de Sincronización Básico

Vamos a crear un servicio completo de sincronización:

```javascript
// services/syncService.js
import NetInfo from '@react-native-community/netinfo';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Constantes para el sistema de sincronización
const SYNC_STATUS_KEY = 'sync_status';
const PENDING_CHANGES_KEY = 'pending_changes';
const LAST_SYNC_KEY = 'last_sync_timestamp';
const SYNC_INTERVAL = 5 * 60 * 1000; // 5 minutos

// Estados de sincronización
export const SYNC_STATUS = {
  IDLE: 'idle',
  SYNCING: 'syncing',
  SUCCESS: 'success',
  ERROR: 'error',
  OFFLINE: 'offline'
};

// Tipos de operaciones
export const OPERATION_TYPES = {
  CREATE: 'CREATE',
  UPDATE: 'UPDATE',
  DELETE: 'DELETE'
};

// Clase principal del servicio de sincronización
export class SyncService {
  // Propiedades de la clase
  isOnline = false;
  isSyncing = false;
  syncInterval = null;
  listeners = new Set();

  // Constructor de la clase
  constructor() {
    // Inicializamos el estado de conexión
    this.initializeNetworkListener();
    
    // Inicializamos el intervalo de sincronización
    this.startSyncInterval();
  }

  // Método para inicializar el listener de red
  initializeNetworkListener() {
    try {
      // Suscribimos a cambios en el estado de la red
      NetInfo.addEventListener(state => {
        const wasOnline = this.isOnline;
        this.isOnline = state.isConnected && state.isInternetReachable;
        
        console.log(`Network status changed: ${this.isOnline ? 'Online' : 'Offline'}`);
        
        // Si cambiamos de offline a online, intentamos sincronizar
        if (!wasOnline && this.isOnline) {
          console.log('Coming back online, attempting sync...');
          this.syncData();
        }
        
        // Notificamos a los listeners del cambio de estado
        this.notifyListeners({
          type: 'NETWORK_STATUS_CHANGED',
          isOnline: this.isOnline
        });
      });
      
      console.log('Network listener initialized successfully');
    } catch (error) {
      console.error('Error initializing network listener:', error);
    }
  }

  // Método para iniciar el intervalo de sincronización
  startSyncInterval() {
    try {
      // Limpiamos el intervalo existente si hay uno
      if (this.syncInterval) {
        clearInterval(this.syncInterval);
      }
      
      // Iniciamos un nuevo intervalo
      this.syncInterval = setInterval(() => {
        if (this.isOnline && !this.isSyncing) {
          console.log('Periodic sync triggered');
          this.syncData();
        }
      }, SYNC_INTERVAL);
      
      console.log('Sync interval started successfully');
    } catch (error) {
      console.error('Error starting sync interval:', error);
    }
  }

  // Método para detener el intervalo de sincronización
  stopSyncInterval() {
    try {
      if (this.syncInterval) {
        clearInterval(this.syncInterval);
        this.syncInterval = null;
        console.log('Sync interval stopped successfully');
      }
    } catch (error) {
      console.error('Error stopping sync interval:', error);
    }
  }

  // Método principal para sincronizar datos
  async syncData() {
    try {
      // Verificamos si ya estamos sincronizando
      if (this.isSyncing) {
        console.log('Sync already in progress, skipping...');
        return;
      }
      
      // Verificamos si estamos online
      if (!this.isOnline) {
        console.log('Device is offline, skipping sync...');
        this.updateSyncStatus(SYNC_STATUS.OFFLINE);
        return;
      }
      
      // Marcamos que estamos sincronizando
      this.isSyncing = true;
      this.updateSyncStatus(SYNC_STATUS.SYNCING);
      
      console.log('Starting data synchronization...');
      
      // Obtenemos los cambios pendientes
      const pendingChanges = await this.getPendingChanges();
      
      if (pendingChanges.length === 0) {
        console.log('No pending changes to sync');
        this.updateSyncStatus(SYNC_STATUS.SUCCESS);
        return;
      }
      
      console.log(`Found ${pendingChanges.length} pending changes to sync`);
      
      // Procesamos cada cambio pendiente
      const results = await this.processPendingChanges(pendingChanges);
      
      // Limpiamos los cambios procesados exitosamente
      await this.cleanupProcessedChanges(results.successful);
      
      // Actualizamos el estado de sincronización
      if (results.failed.length > 0) {
        console.warn(`${results.failed.length} changes failed to sync`);
        this.updateSyncStatus(SYNC_STATUS.ERROR);
      } else {
        console.log('All changes synced successfully');
        this.updateSyncStatus(SYNC_STATUS.SUCCESS);
      }
      
      // Actualizamos el timestamp de la última sincronización
      await this.updateLastSyncTimestamp();
      
      // Notificamos a los listeners del resultado
      this.notifyListeners({
        type: 'SYNC_COMPLETED',
        results,
        timestamp: new Date().toISOString()
      });
      
    } catch (error) {
      console.error('Error during data synchronization:', error);
      this.updateSyncStatus(SYNC_STATUS.ERROR);
      
      // Notificamos a los listeners del error
      this.notifyListeners({
        type: 'SYNC_ERROR',
        error: error.message,
        timestamp: new Date().toISOString()
      });
    } finally {
      // Marcamos que ya no estamos sincronizando
      this.isSyncing = false;
    }
  }

  // Método para obtener cambios pendientes
  async getPendingChanges() {
    try {
      const pendingChangesData = await AsyncStorage.getItem(PENDING_CHANGES_KEY);
      
      if (!pendingChangesData) {
        return [];
      }
      
      const pendingChanges = JSON.parse(pendingChangesData);
      
      // Filtramos cambios que no han expirado
      const now = Date.now();
      const validChanges = pendingChanges.filter(change => {
        // Los cambios expiran después de 24 horas
        const changeAge = now - change.timestamp;
        const maxAge = 24 * 60 * 60 * 1000; // 24 horas
        
        if (changeAge > maxAge) {
          console.log(`Removing expired change: ${change.id}`);
          return false;
        }
        
        return true;
      });
      
      // Si hay cambios expirados, los eliminamos
      if (validChanges.length !== pendingChanges.length) {
        await AsyncStorage.setItem(PENDING_CHANGES_KEY, JSON.stringify(validChanges));
      }
      
      return validChanges;
    } catch (error) {
      console.error('Error getting pending changes:', error);
      return [];
    }
  }

  // Método para procesar cambios pendientes
  async processPendingChanges(pendingChanges) {
    const results = {
      successful: [],
      failed: []
    };
    
    try {
      // Procesamos cada cambio secuencialmente para evitar conflictos
      for (const change of pendingChanges) {
        try {
          console.log(`Processing change: ${change.id} (${change.type})`);
          
          // Procesamos el cambio según su tipo
          const result = await this.processChange(change);
          
          if (result.success) {
            results.successful.push({
              changeId: change.id,
              result: result.data
            });
            console.log(`Change ${change.id} processed successfully`);
          } else {
            results.failed.push({
              changeId: change.id,
              error: result.error,
              retryCount: change.retryCount || 0
            });
            console.log(`Change ${change.id} failed: ${result.error}`);
          }
        } catch (error) {
          console.error(`Error processing change ${change.id}:`, error);
          results.failed.push({
            changeId: change.id,
            error: error.message,
            retryCount: change.retryCount || 0
          });
        }
      }
      
      return results;
    } catch (error) {
      console.error('Error processing pending changes:', error);
      throw error;
    }
  }

  // Método para procesar un cambio individual
  async processChange(change) {
    try {
      // Simulamos el procesamiento según el tipo de operación
      switch (change.type) {
        case OPERATION_TYPES.CREATE:
          return await this.processCreateOperation(change);
        case OPERATION_TYPES.UPDATE:
          return await this.processUpdateOperation(change);
        case OPERATION_TYPES.DELETE:
          return await this.processDeleteOperation(change);
        default:
          throw new Error(`Unknown operation type: ${change.type}`);
      }
    } catch (error) {
      console.error(`Error processing change ${change.id}:`, error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para procesar operación de creación
  async processCreateOperation(change) {
    try {
      // Aquí implementarías la lógica real para crear en el servidor
      // Por ahora simulamos una llamada a API
      console.log(`Creating ${change.entityType} with data:`, change.data);
      
      // Simulamos delay de red
      await this.simulateNetworkDelay();
      
      // Simulamos respuesta exitosa del servidor
      const serverResponse = {
        id: Date.now(),
        ...change.data,
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString()
      };
      
      return {
        success: true,
        data: serverResponse
      };
    } catch (error) {
      console.error('Error in create operation:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para procesar operación de actualización
  async processUpdateOperation(change) {
    try {
      console.log(`Updating ${change.entityType} ${change.entityId} with data:`, change.data);
      
      // Simulamos delay de red
      await this.simulateNetworkDelay();
      
      // Simulamos respuesta exitosa del servidor
      const serverResponse = {
        id: change.entityId,
        ...change.data,
        updatedAt: new Date().toISOString()
      };
      
      return {
        success: true,
        data: serverResponse
      };
    } catch (error) {
      console.error('Error in update operation:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para procesar operación de eliminación
  async processDeleteOperation(change) {
    try {
      console.log(`Deleting ${change.entityType} ${change.entityId}`);
      
      // Simulamos delay de red
      await this.simulateNetworkDelay();
      
      // Simulamos respuesta exitosa del servidor
      return {
        success: true,
        data: { deleted: true, entityId: change.entityId }
      };
    } catch (error) {
      console.error('Error in delete operation:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para simular delay de red (solo para testing)
  async simulateNetworkDelay() {
    return new Promise(resolve => {
      setTimeout(resolve, Math.random() * 1000 + 500); // 500-1500ms
    });
  }

  // Método para limpiar cambios procesados exitosamente
  async cleanupProcessedChanges(successfulChanges) {
    try {
      if (successfulChanges.length === 0) {
        return;
      }
      
      // Obtenemos los cambios pendientes actuales
      const pendingChanges = await this.getPendingChanges();
      
      // Filtramos los cambios exitosos
      const remainingChanges = pendingChanges.filter(change => {
        return !successfulChanges.some(successful => successful.changeId === change.id);
      });
      
      // Guardamos los cambios restantes
      await AsyncStorage.setItem(PENDING_CHANGES_KEY, JSON.stringify(remainingChanges));
      
      console.log(`Cleaned up ${successfulChanges.length} successful changes`);
    } catch (error) {
      console.error('Error cleaning up processed changes:', error);
    }
  }

  // Método para agregar un cambio pendiente
  async addPendingChange(change) {
    try {
      // Generamos un ID único para el cambio
      const changeWithId = {
        id: `change_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
        timestamp: Date.now(),
        retryCount: 0,
        ...change
      };
      
      // Obtenemos los cambios pendientes existentes
      const existingChanges = await this.getPendingChanges();
      
      // Agregamos el nuevo cambio
      const updatedChanges = [...existingChanges, changeWithId];
      
      // Guardamos los cambios actualizados
      await AsyncStorage.setItem(PENDING_CHANGES_KEY, JSON.stringify(updatedChanges));
      
      console.log(`Added pending change: ${changeWithId.id} (${change.type})`);
      
      // Notificamos a los listeners del nuevo cambio
      this.notifyListeners({
        type: 'PENDING_CHANGE_ADDED',
        change: changeWithId
      });
      
      // Si estamos online, intentamos sincronizar inmediatamente
      if (this.isOnline && !this.isSyncing) {
        console.log('Triggering immediate sync for new change');
        this.syncData();
      }
      
      return changeWithId.id;
    } catch (error) {
      console.error('Error adding pending change:', error);
      throw error;
    }
  }

  // Método para actualizar el estado de sincronización
  async updateSyncStatus(status) {
    try {
      const syncStatus = {
        status,
        timestamp: new Date().toISOString(),
        isOnline: this.isOnline
      };
      
      await AsyncStorage.setItem(SYNC_STATUS_KEY, JSON.stringify(syncStatus));
      
      // Notificamos a los listeners del cambio de estado
      this.notifyListeners({
        type: 'SYNC_STATUS_CHANGED',
        status: syncStatus
      });
      
      console.log(`Sync status updated to: ${status}`);
    } catch (error) {
      console.error('Error updating sync status:', error);
    }
  }

  // Método para obtener el estado actual de sincronización
  async getSyncStatus() {
    try {
      const syncStatusData = await AsyncStorage.getItem(SYNC_STATUS_KEY);
      
      if (!syncStatusData) {
        return {
          status: SYNC_STATUS.IDLE,
          timestamp: null,
          isOnline: this.isOnline
        };
      }
      
      return JSON.parse(syncStatusData);
    } catch (error) {
      console.error('Error getting sync status:', error);
      return {
        status: SYNC_STATUS.ERROR,
        timestamp: null,
        isOnline: this.isOnline
      };
    }
  }

  // Método para actualizar el timestamp de la última sincronización
  async updateLastSyncTimestamp() {
    try {
      const timestamp = new Date().toISOString();
      await AsyncStorage.setItem(LAST_SYNC_KEY, timestamp);
      console.log(`Last sync timestamp updated: ${timestamp}`);
    } catch (error) {
      console.error('Error updating last sync timestamp:', error);
    }
  }

  // Método para obtener el timestamp de la última sincronización
  async getLastSyncTimestamp() {
    try {
      return await AsyncStorage.getItem(LAST_SYNC_KEY);
    } catch (error) {
      console.error('Error getting last sync timestamp:', error);
      return null;
    }
  }

  // Método para forzar la sincronización
  async forceSync() {
    try {
      console.log('Force sync requested');
      
      if (this.isSyncing) {
        console.log('Sync already in progress, cannot force sync');
        return false;
      }
      
      await this.syncData();
      return true;
    } catch (error) {
      console.error('Error during force sync:', error);
      return false;
    }
  }

  // Método para agregar un listener
  addListener(listener) {
    this.listeners.add(listener);
    
    // Retornamos una función para remover el listener
    return () => {
      this.listeners.delete(listener);
    };
  }

  // Método para notificar a todos los listeners
  notifyListeners(event) {
    this.listeners.forEach(listener => {
      try {
        listener(event);
      } catch (error) {
        console.error('Error in sync listener:', error);
      }
    });
  }

  // Método para limpiar recursos
  cleanup() {
    try {
      this.stopSyncInterval();
      this.listeners.clear();
      console.log('Sync service cleaned up successfully');
    } catch (error) {
      console.error('Error cleaning up sync service:', error);
    }
  }
}

// Exportamos una instancia singleton del servicio
export default new SyncService();
```

### 2. Servicio de Sincronización de Entidades Específicas

```javascript
// services/entitySyncService.js
import SyncService, { OPERATION_TYPES } from './syncService';

// Clase para sincronizar entidades específicas
export class EntitySyncService {
  constructor(entityType) {
    this.entityType = entityType;
  }

  // Método para crear una entidad
  async createEntity(data) {
    try {
      console.log(`Creating ${this.entityType} entity:`, data);
      
      // Guardamos localmente primero
      const localEntity = await this.saveLocally(data);
      
      // Agregamos a la cola de sincronización
      const changeId = await SyncService.addPendingChange({
        type: OPERATION_TYPES.CREATE,
        entityType: this.entityType,
        data: localEntity,
        localId: localEntity.id
      });
      
      console.log(`Entity ${this.entityType} queued for sync with ID: ${changeId}`);
      
      return {
        success: true,
        entity: localEntity,
        changeId
      };
    } catch (error) {
      console.error(`Error creating ${this.entityType} entity:`, error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para actualizar una entidad
  async updateEntity(entityId, updates) {
    try {
      console.log(`Updating ${this.entityType} entity ${entityId}:`, updates);
      
      // Actualizamos localmente primero
      const updatedEntity = await this.updateLocally(entityId, updates);
      
      // Agregamos a la cola de sincronización
      const changeId = await SyncService.addPendingChange({
        type: OPERATION_TYPES.UPDATE,
        entityType: this.entityType,
        entityId,
        data: updates,
        localId: entityId
      });
      
      console.log(`Entity ${this.entityType} update queued for sync with ID: ${changeId}`);
      
      return {
        success: true,
        entity: updatedEntity,
        changeId
      };
    } catch (error) {
      console.error(`Error updating ${this.entityType} entity:`, error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para eliminar una entidad
  async deleteEntity(entityId) {
    try {
      console.log(`Deleting ${this.entityType} entity ${entityId}`);
      
      // Eliminamos localmente primero
      await this.deleteLocally(entityId);
      
      // Agregamos a la cola de sincronización
      const changeId = await SyncService.addPendingChange({
        type: OPERATION_TYPES.DELETE,
        entityType: this.entityType,
        entityId,
        localId: entityId
      });
      
      console.log(`Entity ${this.entityType} deletion queued for sync with ID: ${changeId}`);
      
      return {
        success: true,
        changeId
      };
    } catch (error) {
      console.error(`Error deleting ${this.entityType} entity:`, error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para sincronizar desde el servidor
  async syncFromServer() {
    try {
      console.log(`Syncing ${this.entityType} entities from server...`);
      
      // Aquí implementarías la lógica para obtener datos del servidor
      // Por ahora simulamos la respuesta
      const serverEntities = await this.fetchFromServer();
      
      // Sincronizamos con los datos locales
      const syncResult = await this.mergeWithLocalData(serverEntities);
      
      console.log(`Sync from server completed: ${syncResult.updated} updated, ${syncResult.created} created`);
      
      return {
        success: true,
        result: syncResult
      };
    } catch (error) {
      console.error(`Error syncing ${this.entityType} from server:`, error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Método para obtener datos del servidor (simulado)
  async fetchFromServer() {
    // Simulamos una llamada a API
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    // Retornamos datos simulados
    return [
      {
        id: 'server_1',
        name: 'Entity from Server 1',
        description: 'This entity was created on the server',
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString()
      },
      {
        id: 'server_2',
        name: 'Entity from Server 2',
        description: 'Another server entity',
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString()
      }
    ];
  }

  // Método para fusionar datos del servidor con datos locales
  async mergeWithLocalData(serverEntities) {
    try {
      let updated = 0;
      let created = 0;
      
      for (const serverEntity of serverEntities) {
        // Verificamos si la entidad ya existe localmente
        const localEntity = await this.getLocalEntity(serverEntity.id);
        
        if (localEntity) {
          // Actualizamos la entidad local si es más reciente
          if (new Date(serverEntity.updatedAt) > new Date(localEntity.updatedAt)) {
            await this.updateLocally(serverEntity.id, serverEntity);
            updated++;
          }
        } else {
          // Creamos la entidad local si no existe
          await this.saveLocally(serverEntity);
          created++;
        }
      }
      
      return { updated, created };
    } catch (error) {
      console.error('Error merging server data with local data:', error);
      throw error;
    }
  }

  // Métodos auxiliares para operaciones locales (implementar según tu storage)
  async saveLocally(data) {
    // Implementar según tu sistema de almacenamiento local
    const entity = {
      id: data.id || `local_${Date.now()}`,
      ...data,
      createdAt: data.createdAt || new Date().toISOString(),
      updatedAt: new Date().toISOString()
    };
    
    // Aquí guardarías en AsyncStorage, SQLite, o Realm
    console.log(`Saved ${this.entityType} locally:`, entity);
    
    return entity;
  }

  async updateLocally(entityId, updates) {
    // Implementar según tu sistema de almacenamiento local
    const entity = {
      id: entityId,
      ...updates,
      updatedAt: new Date().toISOString()
    };
    
    console.log(`Updated ${this.entityType} locally:`, entity);
    
    return entity;
  }

  async deleteLocally(entityId) {
    // Implementar según tu sistema de almacenamiento local
    console.log(`Deleted ${this.entityType} locally: ${entityId}`);
  }

  async getLocalEntity(entityId) {
    // Implementar según tu sistema de almacenamiento local
    // Retornar null si no existe
    return null;
  }
}

// Servicios específicos para diferentes entidades
export const userSyncService = new EntitySyncService('User');
export const productSyncService = new EntitySyncService('Product');
export const orderSyncService = new EntitySyncService('Order');
```

### 3. Hook para Usar la Sincronización

```javascript
// hooks/useSync.js
import { useState, useEffect, useCallback } from 'react';
import SyncService, { SYNC_STATUS } from '../services/syncService';

export const useSync = () => {
  const [syncState, setSyncState] = useState({
    status: SYNC_STATUS.IDLE,
    isOnline: false,
    isSyncing: false,
    lastSync: null,
    pendingChanges: 0,
    error: null
  });

  useEffect(() => {
    // Inicializamos el estado
    initializeSyncState();
    
    // Suscribimos a eventos de sincronización
    const unsubscribe = SyncService.addListener(handleSyncEvent);
    
    // Cleanup al desmontar
    return () => {
      unsubscribe();
    };
  }, []);

  const initializeSyncState = async () => {
    try {
      const [status, lastSync, pendingChanges] = await Promise.all([
        SyncService.getSyncStatus(),
        SyncService.getLastSyncTimestamp(),
        SyncService.getPendingChanges()
      ]);
      
      setSyncState(prev => ({
        ...prev,
        status: status.status,
        isOnline: status.isOnline,
        lastSync: lastSync,
        pendingChanges: pendingChanges.length
      }));
    } catch (error) {
      console.error('Error initializing sync state:', error);
    }
  };

  const handleSyncEvent = (event) => {
    switch (event.type) {
      case 'SYNC_STATUS_CHANGED':
        setSyncState(prev => ({
          ...prev,
          status: event.status.status,
          isOnline: event.status.isOnline
        }));
        break;
        
      case 'SYNC_COMPLETED':
        setSyncState(prev => ({
          ...prev,
          isSyncing: false,
          lastSync: event.timestamp
        }));
        // Recargamos el estado
        initializeSyncState();
        break;
        
      case 'SYNC_ERROR':
        setSyncState(prev => ({
          ...prev,
          isSyncing: false,
          error: event.error
        }));
        break;
        
      case 'PENDING_CHANGE_ADDED':
        setSyncState(prev => ({
          ...prev,
          pendingChanges: prev.pendingChanges + 1
        }));
        break;
        
      case 'NETWORK_STATUS_CHANGED':
        setSyncState(prev => ({
          ...prev,
          isOnline: event.isOnline
        }));
        break;
    }
  };

  const forceSync = useCallback(async () => {
    try {
      setSyncState(prev => ({ ...prev, isSyncing: true, error: null }));
      
      const success = await SyncService.forceSync();
      
      if (!success) {
        setSyncState(prev => ({
          ...prev,
          isSyncing: false,
          error: 'Failed to start sync'
        }));
      }
    } catch (error) {
      setSyncState(prev => ({
        ...prev,
        isSyncing: false,
        error: error.message
      }));
    }
  }, []);

  const getSyncStatusText = useCallback(() => {
    switch (syncState.status) {
      case SYNC_STATUS.IDLE:
        return 'Idle';
      case SYNC_STATUS.SYNCING:
        return 'Syncing...';
      case SYNC_STATUS.SUCCESS:
        return 'Last sync successful';
      case SYNC_STATUS.ERROR:
        return 'Last sync failed';
      case SYNC_STATUS.OFFLINE:
        return 'Offline';
      default:
        return 'Unknown';
    }
  }, [syncState.status]);

  const getSyncStatusColor = useCallback(() => {
    switch (syncState.status) {
      case SYNC_STATUS.SUCCESS:
        return '#4caf50';
      case SYNC_STATUS.ERROR:
        return '#f44336';
      case SYNC_STATUS.SYNCING:
        return '#2196f3';
      case SYNC_STATUS.OFFLINE:
        return '#ff9800';
      default:
        return '#9e9e9e';
    }
  }, [syncState.status]);

  return {
    ...syncState,
    forceSync,
    getSyncStatusText,
    getSyncStatusColor
  };
};
```

---

## 💡 Casos de Uso Prácticos

### 1. Componente de Estado de Sincronización

```javascript
// components/SyncStatus.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet, ActivityIndicator } from 'react-native';
import { useSync } from '../hooks/useSync';

const SyncStatus = () => {
  const {
    status,
    isOnline,
    isSyncing,
    lastSync,
    pendingChanges,
    error,
    forceSync,
    getSyncStatusText,
    getSyncStatusColor
  } = useSync();

  const formatLastSync = (timestamp) => {
    if (!timestamp) return 'Never';
    
    const date = new Date(timestamp);
    const now = new Date();
    const diffMs = now - date;
    const diffMins = Math.floor(diffMs / 60000);
    
    if (diffMins < 1) return 'Just now';
    if (diffMins < 60) return `${diffMins} minutes ago`;
    
    const diffHours = Math.floor(diffMins / 60);
    if (diffHours < 24) return `${diffHours} hours ago`;
    
    const diffDays = Math.floor(diffHours / 24);
    return `${diffDays} days ago`;
  };

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Sync Status</Text>
        <View style={[styles.statusIndicator, { backgroundColor: getSyncStatusColor() }]} />
      </View>
      
      <View style={styles.content}>
        <View style={styles.row}>
          <Text style={styles.label}>Status:</Text>
          <Text style={styles.value}>{getSyncStatusText()}</Text>
        </View>
        
        <View style={styles.row}>
          <Text style={styles.label}>Connection:</Text>
          <Text style={[styles.value, { color: isOnline ? '#4caf50' : '#f44336' }]}>
            {isOnline ? 'Online' : 'Offline'}
          </Text>
        </View>
        
        <View style={styles.row}>
          <Text style={styles.label}>Last Sync:</Text>
          <Text style={styles.value}>{formatLastSync(lastSync)}</Text>
        </View>
        
        <View style={styles.row}>
          <Text style={styles.label}>Pending Changes:</Text>
          <Text style={styles.value}>{pendingChanges}</Text>
        </View>
        
        {error && (
          <View style={styles.errorContainer}>
            <Text style={styles.errorText}>Error: {error}</Text>
          </View>
        )}
      </View>
      
      <View style={styles.actions}>
        <TouchableOpacity
          style={[styles.syncButton, isSyncing && styles.syncButtonDisabled]}
          onPress={forceSync}
          disabled={isSyncing || !isOnline}
        >
          {isSyncing ? (
            <ActivityIndicator size="small" color="white" />
          ) : (
            <Text style={styles.syncButtonText}>Sync Now</Text>
          )}
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 16,
    margin: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 16,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    flex: 1,
  },
  statusIndicator: {
    width: 12,
    height: 12,
    borderRadius: 6,
  },
  content: {
    marginBottom: 16,
  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0',
  },
  label: {
    fontSize: 14,
    color: '#666',
    fontWeight: '500',
  },
  value: {
    fontSize: 14,
    fontWeight: '600',
  },
  errorContainer: {
    backgroundColor: '#ffebee',
    padding: 12,
    borderRadius: 8,
    marginTop: 12,
  },
  errorText: {
    color: '#c62828',
    fontSize: 14,
  },
  actions: {
    alignItems: 'center',
  },
  syncButton: {
    backgroundColor: '#2196f3',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
    minWidth: 120,
    alignItems: 'center',
  },
  syncButtonDisabled: {
    backgroundColor: '#ccc',
  },
  syncButtonText: {
    color: 'white',
    fontWeight: '600',
    fontSize: 16,
  },
});

export default SyncStatus;
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Sincronización Básico**
1. Implementa el servicio de sincronización básico
2. Crea el sistema de cola de cambios pendientes
3. Implementa sincronización automática y manual
4. Prueba la sincronización con datos simulados

### **Ejercicio 2: Sincronización de Entidades**
1. Implementa el servicio de sincronización de entidades
2. Crea servicios para usuarios, productos y pedidos
3. Implementa operaciones CRUD con sincronización
4. Prueba la sincronización bidireccional

### **Ejercicio 3: Manejo de Conflictos**
1. Implementa detección de conflictos de sincronización
2. Crea estrategias de resolución de conflictos
3. Implementa versionado de entidades
4. Prueba escenarios de conflicto

### **Ejercicio 4: Sincronización en Tiempo Real**
1. Implementa WebSockets para sincronización en tiempo real
2. Crea notificaciones push para cambios
3. Implementa sincronización selectiva
4. Prueba la latencia de sincronización

### **Ejercicio 5: Testing de Sincronización**
1. Crea tests unitarios para el servicio de sincronización
2. Implementa tests de integración para sincronización
3. Prueba escenarios offline/online
4. Crea tests de rendimiento para sincronización

---

## 🎯 Proyecto de la Clase

### **App de Notas con Sincronización**

Crea una aplicación que permita:
1. **Crear y editar notas** localmente
2. **Sincronización automática** con el servidor
3. **Funcionamiento offline** completo
4. **Resolución de conflictos** automática
5. **Sincronización en tiempo real** entre dispositivos

**Estructura del Proyecto:**
```
src/
├── services/
│   ├── syncService.js
│   ├── entitySyncService.js
│   └── noteService.js
├── hooks/
│   └── useSync.js
├── components/
│   ├── SyncStatus.js
│   ├── NoteList.js
│   └── NoteEditor.js
├── screens/
│   ├── NotesScreen.js
│   └── SyncScreen.js
└── utils/
    └── conflictResolver.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [NetInfo](https://github.com/react-native-community/react-native-netinfo)
- [AsyncStorage](https://react-native-async-storage.github.io/async-storage/)

### **Mejores Prácticas:**
- **Offline-first**: Diseña para funcionar sin conexión
- **Conflict Resolution**: Implementa estrategias claras de resolución
- **Error Handling**: Maneja errores de red graciosamente
- **Performance**: Optimiza la sincronización para grandes volúmenes

### **Estrategias de Sincronización:**
- **Pull-based**: El cliente solicita datos del servidor
- **Push-based**: El servidor envía datos al cliente
- **Hybrid**: Combinación de ambas estrategias
- **Event-driven**: Sincronización basada en eventos

---

## 🔍 Resumen de la Clase

### **Conceptos Clave:**
- ✅ La sincronización es esencial para apps multi-dispositivo
- ✅ Implementa estrategias offline-first para mejor UX
- ✅ Maneja conflictos de sincronización de forma robusta
- ✅ Usa colas de cambios pendientes para operaciones offline
- ✅ Monitorea el estado de la red para sincronización automática

### **Próximos Pasos:**
- 🔄 Completar todos los ejercicios de esta clase
- 🔄 Implementar la app de notas con sincronización
- 🔄 Practicar con diferentes escenarios de red
- 🔄 Prepararse para la siguiente clase sobre backup y restauración

---

## 🚀 Siguiente Clase

En la **Clase 6: Backup y Restauración**, aprenderás:
- Cómo crear backups automáticos de datos
- Implementación de restauración de datos
- Estrategias de backup incremental
- Cifrado de backups
- Testing de backup y restauración

---

**¡Excelente trabajo con la sincronización! 🎉**

**Recuerda**: La sincronización es la base de las apps modernas. Diseña siempre pensando en offline-first y maneja los conflictos de forma elegante.
