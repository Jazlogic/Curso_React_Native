# 📚 Clase 6: Backup y Restauración

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 5: Sincronización](clase_5_sincronizacion.md)
- **➡️ Siguiente**: [README del Módulo](README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Implementar sistemas de backup automático** para AsyncStorage, SQLite y Realm
2. **Crear estrategias de restauración** con validación de integridad de datos
3. **Implementar cifrado** para backups sensibles
4. **Gestionar versiones de backup** y limpieza automática
5. **Crear sistemas de testing** para validar backups y restauraciones
6. **Implementar backup en la nube** con sincronización

---

## 📚 Contenido Teórico

### **¿Qué es Backup y Restauración?**

El **backup** es el proceso de crear copias de seguridad de los datos de la aplicación, mientras que la **restauración** es el proceso de recuperar esos datos cuando sea necesario.

### **Tipos de Backup:**

1. **Backup Completo**: Copia todos los datos
2. **Backup Incremental**: Solo copia cambios desde el último backup
3. **Backup Diferencial**: Copia cambios desde el backup completo inicial

### **Estrategias de Backup:**

- **Backup Automático**: Se ejecuta en intervalos regulares
- **Backup Manual**: Se ejecuta cuando el usuario lo solicita
- **Backup en Eventos**: Se ejecuta antes de actualizaciones importantes

---

## 💻 Implementación Práctica

### **1. Servicio de Backup para AsyncStorage**

```javascript:src/services/AsyncStorageBackupService.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import CryptoJS from 'crypto-js';

class AsyncStorageBackupService {
  constructor(encryptionKey = null) {
    this.encryptionKey = encryptionKey;
    this.backupPrefix = '@backup_';
    this.maxBackups = 10; // Mantener solo los últimos 10 backups
  }

  // Crear backup con timestamp
  async createBackup(description = '') {
    try {
      // Obtener todas las claves de AsyncStorage
      const keys = await AsyncStorage.getAllKeys();
      
      // Obtener todos los valores
      const data = {};
      for (const key of keys) {
        if (!key.startsWith(this.backupPrefix)) { // Excluir otros backups
          const value = await AsyncStorage.getItem(key);
          data[key] = value;
        }
      }

      // Crear objeto de backup con metadata
      const backup = {
        timestamp: Date.now(),
        description,
        data,
        version: '1.0',
        checksum: this.calculateChecksum(data)
      };

      // Cifrar si hay clave de encriptación
      const backupKey = `${this.backupPrefix}${backup.timestamp}`;
      const backupData = this.encryptionKey 
        ? this.encryptBackup(backup)
        : JSON.stringify(backup);

      // Guardar backup
      await AsyncStorage.setItem(backupKey, backupData);
      
      // Limpiar backups antiguos
      await this.cleanOldBackups();
      
      return {
        success: true,
        backupId: backup.timestamp,
        size: JSON.stringify(backup).length
      };
    } catch (error) {
      console.error('Error creating backup:', error);
      return { success: false, error: error.message };
    }
  }

  // Restaurar desde backup
  async restoreBackup(backupId) {
    try {
      const backupKey = `${this.backupPrefix}${backupId}`;
      const backupData = await AsyncStorage.getItem(backupKey);
      
      if (!backupData) {
        throw new Error('Backup not found');
      }

      // Descifrar si está encriptado
      let backup;
      if (this.encryptionKey) {
        backup = this.decryptBackup(backupData);
      } else {
        backup = JSON.parse(backupData);
      }

      // Validar checksum
      if (!this.validateBackup(backup)) {
        throw new Error('Backup corrupted or invalid');
      }

      // Crear backup del estado actual antes de restaurar
      await this.createBackup('Auto-backup before restore');

      // Restaurar datos
      for (const [key, value] of Object.entries(backup.data)) {
        await AsyncStorage.setItem(key, value);
      }

      return {
        success: true,
        restoredItems: Object.keys(backup.data).length,
        backupDate: new Date(backup.timestamp).toLocaleString()
      };
    } catch (error) {
      console.error('Error restoring backup:', error);
      return { success: false, error: error.message };
    }
  }

  // Listar todos los backups disponibles
  async listBackups() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      const backupKeys = keys.filter(key => key.startsWith(this.backupPrefix));
      
      const backups = [];
      for (const key of backupKeys) {
        const backupData = await AsyncStorage.getItem(key);
        let backup;
        
        if (this.encryptionKey) {
          backup = this.decryptBackup(backupData);
        } else {
          backup = JSON.parse(backupData);
        }

        backups.push({
          id: backup.timestamp,
          date: new Date(backup.timestamp).toLocaleString(),
          description: backup.description,
          size: JSON.stringify(backup).length,
          version: backup.version
        });
      }

      // Ordenar por fecha (más reciente primero)
      return backups.sort((a, b) => b.id - a.id);
    } catch (error) {
      console.error('Error listing backups:', error);
      return [];
    }
  }

  // Eliminar backup específico
  async deleteBackup(backupId) {
    try {
      const backupKey = `${this.backupPrefix}${backupId}`;
      await AsyncStorage.removeItem(backupKey);
      return { success: true };
    } catch (error) {
      console.error('Error deleting backup:', error);
      return { success: false, error: error.message };
    }
  }

  // Limpiar backups antiguos
  async cleanOldBackups() {
    try {
      const backups = await this.listBackups();
      
      if (backups.length > this.maxBackups) {
        const backupsToDelete = backups.slice(this.maxBackups);
        
        for (const backup of backupsToDelete) {
          await this.deleteBackup(backup.id);
        }
        
        return {
          success: true,
          deletedCount: backupsToDelete.length
        };
      }
      
      return { success: true, deletedCount: 0 };
    } catch (error) {
      console.error('Error cleaning old backups:', error);
      return { success: false, error: error.message };
    }
  }

  // Cifrar backup
  encryptBackup(backup) {
    const jsonString = JSON.stringify(backup);
    return CryptoJS.AES.encrypt(jsonString, this.encryptionKey).toString();
  }

  // Descifrar backup
  decryptBackup(encryptedData) {
    const bytes = CryptoJS.AES.decrypt(encryptedData, this.encryptionKey);
    return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
  }

  // Calcular checksum para validación
  calculateChecksum(data) {
    const jsonString = JSON.stringify(data);
    return CryptoJS.SHA256(jsonString).toString();
  }

  // Validar integridad del backup
  validateBackup(backup) {
    if (!backup.data || !backup.timestamp || !backup.checksum) {
      return false;
    }
    
    const calculatedChecksum = this.calculateChecksum(backup.data);
    return calculatedChecksum === backup.checksum;
  }
}

export default AsyncStorageBackupService;
```

### **2. Servicio de Backup para SQLite**

```javascript:src/services/SQLiteBackupService.js
import SQLite from 'react-native-sqlite-storage';
import RNFS from 'react-native-fs';
import CryptoJS from 'crypto-js';

class SQLiteBackupService {
  constructor(databasePath, encryptionKey = null) {
    this.databasePath = databasePath;
    this.encryptionKey = encryptionKey;
    this.backupDir = `${RNFS.DocumentDirectoryPath}/backups`;
    this.maxBackups = 10;
  }

  // Crear backup de la base de datos
  async createBackup(description = '') {
    try {
      // Crear directorio de backups si no existe
      await this.ensureBackupDirectory();
      
      // Generar nombre del archivo de backup
      const timestamp = Date.now();
      const backupFileName = `backup_${timestamp}.db`;
      const backupPath = `${this.backupDir}/${backupFileName}`;
      
      // Copiar archivo de base de datos
      await RNFS.copyFile(this.databasePath, backupPath);
      
      // Crear archivo de metadata
      const metadata = {
        timestamp,
        description,
        originalPath: this.databasePath,
        backupPath,
        version: '1.0',
        fileSize: await RNFS.stat(backupPath).then(stat => stat.size),
        checksum: await this.calculateFileChecksum(backupPath)
      };
      
      const metadataPath = `${this.backupDir}/metadata_${timestamp}.json`;
      const metadataContent = this.encryptionKey 
        ? this.encryptMetadata(metadata)
        : JSON.stringify(metadata);
      
      await RNFS.writeFile(metadataPath, metadataContent, 'utf8');
      
      // Limpiar backups antiguos
      await this.cleanOldBackups();
      
      return {
        success: true,
        backupId: timestamp,
        backupPath,
        metadataPath,
        size: metadata.fileSize
      };
    } catch (error) {
      console.error('Error creating SQLite backup:', error);
      return { success: false, error: error.message };
    }
  }

  // Restaurar desde backup
  async restoreBackup(backupId) {
    try {
      const metadataPath = `${this.backupDir}/metadata_${backupId}.json`;
      const backupPath = `${this.backupDir}/backup_${backupId}.db`;
      
      // Verificar que existe el backup
      if (!(await RNFS.exists(backupPath))) {
        throw new Error('Backup file not found');
      }
      
      // Leer metadata
      const metadataContent = await RNFS.readFile(metadataPath, 'utf8');
      let metadata;
      
      if (this.encryptionKey) {
        metadata = this.decryptMetadata(metadataContent);
      } else {
        metadata = JSON.parse(metadataContent);
      }
      
      // Validar checksum
      const currentChecksum = await this.calculateFileChecksum(backupPath);
      if (currentChecksum !== metadata.checksum) {
        throw new Error('Backup file corrupted');
      }
      
      // Crear backup del estado actual
      await this.createBackup('Auto-backup before restore');
      
      // Cerrar conexiones activas a la base de datos
      await SQLite.close();
      
      // Restaurar archivo
      await RNFS.copyFile(backupPath, this.databasePath);
      
      return {
        success: true,
        restoredFrom: new Date(metadata.timestamp).toLocaleString(),
        fileSize: metadata.fileSize
      };
    } catch (error) {
      console.error('Error restoring SQLite backup:', error);
      return { success: false, error: error.message };
    }
  }

  // Listar backups disponibles
  async listBackups() {
    try {
      await this.ensureBackupDirectory();
      
      const files = await RNFS.readDir(this.backupDir);
      const metadataFiles = files.filter(file => 
        file.name.startsWith('metadata_') && file.name.endsWith('.json')
      );
      
      const backups = [];
      for (const file of metadataFiles) {
        try {
          const content = await RNFS.readFile(file.path, 'utf8');
          let metadata;
          
          if (this.encryptionKey) {
            metadata = this.decryptMetadata(content);
          } else {
            metadata = JSON.parse(content);
          }
          
          backups.push({
            id: metadata.timestamp,
            date: new Date(metadata.timestamp).toLocaleString(),
            description: metadata.description,
            size: metadata.fileSize,
            version: metadata.version
          });
        } catch (error) {
          console.warn(`Error reading metadata file ${file.name}:`, error);
        }
      }
      
      return backups.sort((a, b) => b.id - a.id);
    } catch (error) {
      console.error('Error listing SQLite backups:', error);
      return [];
    }
  }

  // Eliminar backup específico
  async deleteBackup(backupId) {
    try {
      const backupPath = `${this.backupDir}/backup_${backupId}.db`;
      const metadataPath = `${this.backupDir}/metadata_${backupId}.json`;
      
      if (await RNFS.exists(backupPath)) {
        await RNFS.unlink(backupPath);
      }
      
      if (await RNFS.exists(metadataPath)) {
        await RNFS.unlink(metadataPath);
      }
      
      return { success: true };
    } catch (error) {
      console.error('Error deleting SQLite backup:', error);
      return { success: false, error: error.message };
    }
  }

  // Limpiar backups antiguos
  async cleanOldBackups() {
    try {
      const backups = await this.listBackups();
      
      if (backups.length > this.maxBackups) {
        const backupsToDelete = backups.slice(this.maxBackups);
        
        for (const backup of backupsToDelete) {
          await this.deleteBackup(backup.id);
        }
        
        return {
          success: true,
          deletedCount: backupsToDelete.length
        };
      }
      
      return { success: true, deletedCount: 0 };
    } catch (error) {
      console.error('Error cleaning old SQLite backups:', error);
      return { success: false, error: error.message };
    }
  }

  // Asegurar que existe el directorio de backups
  async ensureBackupDirectory() {
    const exists = await RNFS.exists(this.backupDir);
    if (!exists) {
      await RNFS.mkdir(this.backupDir);
    }
  }

  // Calcular checksum del archivo
  async calculateFileChecksum(filePath) {
    const content = await RNFS.readFile(filePath, 'base64');
    return CryptoJS.SHA256(content).toString();
  }

  // Cifrar metadata
  encryptMetadata(metadata) {
    const jsonString = JSON.stringify(metadata);
    return CryptoJS.AES.encrypt(jsonString, this.encryptionKey).toString();
  }

  // Descifrar metadata
  decryptMetadata(encryptedData) {
    const bytes = CryptoJS.AES.decrypt(encryptedData, this.encryptionKey);
    return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
  }
}

export default SQLiteBackupService;
```

### **3. Hook Personalizado para Backup**

```javascript:src/hooks/useBackup.js
import { useState, useCallback } from 'react';
import AsyncStorageBackupService from '../services/AsyncStorageBackupService';
import SQLiteBackupService from '../services/SQLiteBackupService';

export const useBackup = (storageType = 'asyncStorage', options = {}) => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [backups, setBackups] = useState([]);

  // Inicializar servicio según el tipo
  const getBackupService = useCallback(() => {
    if (storageType === 'sqlite') {
      return new SQLiteBackupService(options.databasePath, options.encryptionKey);
    }
    return new AsyncStorageBackupService(options.encryptionKey);
  }, [storageType, options]);

  // Crear backup
  const createBackup = useCallback(async (description = '') => {
    setIsLoading(true);
    setError(null);
    
    try {
      const service = getBackupService();
      const result = await service.createBackup(description);
      
      if (result.success) {
        // Actualizar lista de backups
        await loadBackups();
        return result;
      } else {
        throw new Error(result.error);
      }
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    } finally {
      setIsLoading(false);
    }
  }, [getBackupService]);

  // Restaurar backup
  const restoreBackup = useCallback(async (backupId) => {
    setIsLoading(true);
    setError(null);
    
    try {
      const service = getBackupService();
      const result = await service.restoreBackup(backupId);
      
      if (result.success) {
        return result;
      } else {
        throw new Error(result.error);
      }
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    } finally {
      setIsLoading(false);
    }
  }, [getBackupService]);

  // Cargar lista de backups
  const loadBackups = useCallback(async () => {
    try {
      const service = getBackupService();
      const backupList = await service.listBackups();
      setBackups(backupList);
      return backupList;
    } catch (err) {
      setError(err.message);
      return [];
    }
  }, [getBackupService]);

  // Eliminar backup
  const deleteBackup = useCallback(async (backupId) => {
    try {
      const service = getBackupService();
      const result = await service.deleteBackup(backupId);
      
      if (result.success) {
        // Actualizar lista
        await loadBackups();
        return result;
      } else {
        throw new Error(result.error);
      }
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    }
  }, [getBackupService, loadBackups]);

  // Limpiar backups antiguos
  const cleanOldBackups = useCallback(async () => {
    try {
      const service = getBackupService();
      const result = await service.cleanOldBackups();
      
      if (result.success) {
        // Actualizar lista
        await loadBackups();
        return result;
      } else {
        throw new Error(result.error);
      }
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    }
  }, [getBackupService, loadBackups]);

  return {
    // Estado
    isLoading,
    error,
    backups,
    
    // Acciones
    createBackup,
    restoreBackup,
    loadBackups,
    deleteBackup,
    cleanOldBackups,
    
    // Utilidades
    refreshBackups: loadBackups
  };
};
```

### **4. Componente de Gestión de Backups**

```javascript:src/components/BackupManager.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  FlatList,
  Alert,
  StyleSheet,
  ActivityIndicator
} from 'react-native';
import { useBackup } from '../hooks/useBackup';

const BackupManager = ({ storageType, options, onBackupCreated, onBackupRestored }) => {
  const [description, setDescription] = useState('');
  const [selectedBackup, setSelectedBackup] = useState(null);
  
  const {
    isLoading,
    error,
    backups,
    createBackup,
    restoreBackup,
    loadBackups,
    deleteBackup,
    cleanOldBackups
  } = useBackup(storageType, options);

  // Cargar backups al montar el componente
  useEffect(() => {
    loadBackups();
  }, [loadBackups]);

  // Crear nuevo backup
  const handleCreateBackup = async () => {
    const result = await createBackup(description);
    
    if (result.success) {
      Alert.alert(
        'Backup Creado',
        `Backup creado exitosamente. ID: ${result.backupId}`,
        [{ text: 'OK' }]
      );
      
      setDescription('');
      onBackupCreated?.(result);
    } else {
      Alert.alert('Error', `Error al crear backup: ${result.error}`);
    }
  };

  // Restaurar backup
  const handleRestoreBackup = async (backupId) => {
    Alert.alert(
      'Confirmar Restauración',
      '¿Estás seguro de que quieres restaurar este backup? Se creará un backup automático del estado actual.',
      [
        { text: 'Cancelar', style: 'cancel' },
        {
          text: 'Restaurar',
          style: 'destructive',
          onPress: async () => {
            const result = await restoreBackup(backupId);
            
            if (result.success) {
              Alert.alert(
                'Restauración Exitosa',
                `Backup restaurado desde: ${result.restoredFrom}`,
                [{ text: 'OK' }]
              );
              
              onBackupRestored?.(result);
            } else {
              Alert.alert('Error', `Error al restaurar: ${result.error}`);
            }
          }
        }
      ]
    );
  };

  // Eliminar backup
  const handleDeleteBackup = async (backupId) => {
    Alert.alert(
      'Confirmar Eliminación',
      '¿Estás seguro de que quieres eliminar este backup?',
      [
        { text: 'Cancelar', style: 'cancel' },
        {
          text: 'Eliminar',
          style: 'destructive',
          onPress: async () => {
            const result = await deleteBackup(backupId);
            
            if (result.success) {
              Alert.alert('Backup Eliminado', 'Backup eliminado exitosamente');
            } else {
              Alert.alert('Error', `Error al eliminar: ${result.error}`);
            }
          }
        }
      ]
    );
  };

  // Limpiar backups antiguos
  const handleCleanOldBackups = async () => {
    Alert.alert(
      'Limpiar Backups Antiguos',
      '¿Estás seguro de que quieres eliminar los backups antiguos?',
      [
        { text: 'Cancelar', style: 'cancel' },
        {
          text: 'Limpiar',
          style: 'destructive',
          onPress: async () => {
            const result = await cleanOldBackups();
            
            if (result.success) {
              Alert.alert(
                'Limpieza Completada',
                `${result.deletedCount} backups antiguos eliminados`
              );
            } else {
              Alert.alert('Error', `Error al limpiar: ${result.error}`);
            }
          }
        }
      ]
    );
  };

  // Renderizar item de backup
  const renderBackupItem = ({ item }) => (
    <View style={styles.backupItem}>
      <View style={styles.backupInfo}>
        <Text style={styles.backupDate}>{item.date}</Text>
        <Text style={styles.backupDescription}>{item.description}</Text>
        <Text style={styles.backupSize}>Tamaño: {item.size} bytes</Text>
        <Text style={styles.backupVersion}>Versión: {item.version}</Text>
      </View>
      
      <View style={styles.backupActions}>
        <TouchableOpacity
          style={[styles.actionButton, styles.restoreButton]}
          onPress={() => handleRestoreBackup(item.id)}
        >
          <Text style={styles.buttonText}>Restaurar</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.actionButton, styles.deleteButton]}
          onPress={() => handleDeleteBackup(item.id)}
        >
          <Text style={styles.buttonText}>Eliminar</Text>
        </TouchableOpacity>
      </View>
    </View>
  );

  if (isLoading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Procesando...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Gestor de Backups</Text>
      
      {/* Crear nuevo backup */}
      <View style={styles.createBackupSection}>
        <Text style={styles.sectionTitle}>Crear Nuevo Backup</Text>
        
        <TouchableOpacity
          style={styles.createButton}
          onPress={handleCreateBackup}
          disabled={isLoading}
        >
          <Text style={styles.buttonText}>Crear Backup</Text>
        </TouchableOpacity>
      </View>

      {/* Lista de backups */}
      <View style={styles.backupsSection}>
        <View style={styles.sectionHeader}>
          <Text style={styles.sectionTitle}>Backups Disponibles</Text>
          <TouchableOpacity
            style={styles.cleanButton}
            onPress={handleCleanOldBackups}
          >
            <Text style={styles.buttonText}>Limpiar Antiguos</Text>
          </TouchableOpacity>
        </View>
        
        {backups.length === 0 ? (
          <Text style={styles.noBackupsText}>No hay backups disponibles</Text>
        ) : (
          <FlatList
            data={backups}
            renderItem={renderBackupItem}
            keyExtractor={(item) => item.id.toString()}
            style={styles.backupsList}
          />
        )}
      </View>

      {/* Mostrar errores */}
      {error && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>Error: {error}</Text>
        </View>
      )}
    </View>
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
  createBackupSection: {
    backgroundColor: 'white',
    padding: 16,
    borderRadius: 8,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  backupsSection: {
    flex: 1,
    backgroundColor: 'white',
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3
  },
  sectionHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#333'
  },
  createButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 6,
    alignItems: 'center'
  },
  cleanButton: {
    backgroundColor: '#FF9500',
    padding: 8,
    borderRadius: 6
  },
  buttonText: {
    color: 'white',
    fontWeight: '600'
  },
  backupsList: {
    flex: 1
  },
  backupItem: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  backupInfo: {
    marginBottom: 12
  },
  backupDate: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
    marginBottom: 4
  },
  backupDescription: {
    fontSize: 14,
    color: '#666',
    marginBottom: 4
  },
  backupSize: {
    fontSize: 12,
    color: '#999'
  },
  backupVersion: {
    fontSize: 12,
    color: '#999'
  },
  backupActions: {
    flexDirection: 'row',
    gap: 8
  },
  actionButton: {
    flex: 1,
    padding: 8,
    borderRadius: 6,
    alignItems: 'center'
  },
  restoreButton: {
    backgroundColor: '#34C759'
  },
  deleteButton: {
    backgroundColor: '#FF3B30'
  },
  noBackupsText: {
    textAlign: 'center',
    padding: 20,
    color: '#999',
    fontStyle: 'italic'
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  loadingText: {
    marginTop: 12,
    fontSize: 16,
    color: '#666'
  },
  errorContainer: {
    backgroundColor: '#FFE5E5',
    padding: 12,
    borderRadius: 6,
    marginTop: 16
  },
  errorText: {
    color: '#D70015',
    textAlign: 'center'
  }
});

export default BackupManager;
```

---

## 🧪 Casos de Uso Prácticos

### **1. Backup Automático en App de Notas**

```javascript:src/screens/NotesScreen.js
import React, { useEffect } from 'react';
import { useBackup } from '../hooks/useBackup';

const NotesScreen = () => {
  const { createBackup } = useBackup('asyncStorage', {
    encryptionKey: 'my-secret-key'
  });

  // Crear backup automático cada 24 horas
  useEffect(() => {
    const backupInterval = setInterval(async () => {
      await createBackup('Backup automático diario');
    }, 24 * 60 * 60 * 1000); // 24 horas

    return () => clearInterval(backupInterval);
  }, [createBackup]);

  // Crear backup antes de actualizaciones importantes
  const handleImportantUpdate = async () => {
    await createBackup('Backup antes de actualización importante');
    // Proceder con la actualización
  };

  return (
    // ... resto del componente
  );
};
```

### **2. Backup en App de Base de Datos**

```javascript:src/screens/DatabaseScreen.js
import React from 'react';
import { useBackup } from '../hooks/useBackup';
import BackupManager from '../components/BackupManager';

const DatabaseScreen = () => {
  const { createBackup } = useBackup('sqlite', {
    databasePath: '/path/to/database.db',
    encryptionKey: 'database-secret-key'
  });

  // Crear backup antes de operaciones críticas
  const handleCriticalOperation = async () => {
    await createBackup('Backup antes de operación crítica');
    // Realizar operación crítica
  };

  return (
    <View style={{ flex: 1 }}>
      <BackupManager
        storageType="sqlite"
        options={{
          databasePath: '/path/to/database.db',
          encryptionKey: 'database-secret-key'
        }}
        onBackupCreated={(result) => {
          console.log('Backup creado:', result);
        }}
        onBackupRestored={(result) => {
          console.log('Backup restaurado:', result);
        }}
      />
    </View>
  );
};
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Implementar Backup Automático**
Crea un sistema que haga backup automático cada vez que se guarden más de 10 elementos en AsyncStorage.

### **Ejercicio 2: Validación de Integridad**
Implementa un sistema que valide la integridad de los backups antes de permitir la restauración.

### **Ejercicio 3: Backup en la Nube**
Crea un sistema que sincronice backups locales con almacenamiento en la nube (Firebase, AWS S3).

### **Ejercicio 4: Compresión de Backups**
Implementa compresión de archivos para reducir el tamaño de los backups de SQLite.

---

## 🎯 Proyecto de la Clase

### **App de Gestión de Backups**

Crea una aplicación que permita:
- Crear backups manuales y automáticos
- Restaurar desde cualquier backup
- Gestionar versiones de backup
- Cifrar backups sensibles
- Validar integridad de datos
- Limpiar backups antiguos automáticamente

**Requisitos:**
- Soporte para AsyncStorage y SQLite
- Interfaz intuitiva para gestión
- Notificaciones de backup exitoso/fallido
- Estadísticas de uso de almacenamiento
- Exportar/importar backups

---

## 📚 Recursos Adicionales

### **Documentación:**
- [React Native File System](https://github.com/itinance/react-native-fs)
- [CryptoJS](https://github.com/brix/crypto-js)
- [SQLite Backup](https://www.sqlite.org/backup.html)

### **Artículos:**
- [Best Practices for Mobile App Backup](https://developer.android.com/guide/topics/data/autobackup)
- [iOS App Backup Guidelines](https://developer.apple.com/icloud/)

### **Herramientas:**
- [Flipper](https://fbflipper.com/) - Para debugging de almacenamiento
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Sistemas de backup** para diferentes tipos de almacenamiento
2. **Estrategias de restauración** con validación de integridad
3. **Cifrado de backups** para datos sensibles
4. **Gestión de versiones** y limpieza automática
5. **Testing y validación** de backups
6. **Interfaces de usuario** para gestión de backups

### **🚀 Próximos Pasos:**
- Implementar backup en la nube
- Crear sistemas de compresión
- Implementar backup incremental
- Crear dashboards de monitoreo

### **💡 Conceptos Clave:**
- **Backup**: Copia de seguridad de datos
- **Restauración**: Recuperación de datos desde backup
- **Checksum**: Validación de integridad
- **Cifrado**: Protección de datos sensibles
- **Versionado**: Control de versiones de backup

---

**🎯 Objetivo**: Dominar la creación y gestión de sistemas de backup robustos para aplicaciones React Native.

**💡 Consejo**: Siempre prueba tus sistemas de backup en un entorno seguro antes de usarlos en producción.
