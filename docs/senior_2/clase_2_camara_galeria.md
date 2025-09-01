# Clase 2: Cámara y Galería 📸

## Objetivos de la Clase
- Implementar acceso a la cámara del dispositivo
- Acceder a la galería de fotos y videos
- Capturar fotos y grabar videos
- Editar y procesar imágenes
- Crear componentes reutilizables para multimedia

## Duración Estimada
**2 horas**

## Contenido Teórico

### 1. Acceso a la Cámara

Para acceder a la cámara en React Native, necesitamos usar librerías nativas como `react-native-image-picker` o `react-native-camera`.

```javascript
import { launchCamera, launchImageLibrary } from 'react-native-image-picker';
import { PermissionsAndroid, Platform, Alert } from 'react-native';

class CameraManager {
  // Configuración por defecto para la cámara
  static cameraOptions = {
    mediaType: 'photo', // 'photo', 'video', o 'mixed'
    quality: 0.8, // Calidad de 0 a 1
    includeBase64: false, // Incluir datos base64
    saveToPhotos: true, // Guardar en galería
    cameraType: 'back', // 'back' o 'front'
    maxWidth: 1920, // Ancho máximo
    maxHeight: 1080, // Alto máximo
  };

  // Configuración para galería
  static galleryOptions = {
    mediaType: 'mixed',
    quality: 0.8,
    includeBase64: false,
    selectionLimit: 10, // Máximo de archivos a seleccionar
  };

  // Solicitar permiso de cámara
  static async requestCameraPermission() {
    if (Platform.OS === 'android') {
      try {
        const granted = await PermissionsAndroid.request(
          PermissionsAndroid.PERMISSIONS.CAMERA,
          {
            title: 'Permiso de Cámara',
            message: 'La app necesita acceso a la cámara',
            buttonNeutral: 'Preguntar más tarde',
            buttonNegative: 'Cancelar',
            buttonPositive: 'OK',
          }
        );
        return granted === PermissionsAndroid.RESULTS.GRANTED;
      } catch (err) {
        console.warn('Error al solicitar permiso:', err);
        return false;
      }
    }
    return true; // iOS maneja permisos automáticamente
  }

  // Abrir cámara para tomar foto
  static async takePhoto(options = {}) {
    const hasPermission = await this.requestCameraPermission();
    if (!hasPermission) {
      Alert.alert('Permiso Denegado', 'No se puede acceder a la cámara');
      return null;
    }

    try {
      const result = await launchCamera({
        ...this.cameraOptions,
        ...options,
      });

      if (result.didCancel) {
        console.log('Usuario canceló la captura');
        return null;
      }

      if (result.errorCode) {
        throw new Error(result.errorMessage);
      }

      return result.assets?.[0] || null;
    } catch (error) {
      console.error('Error al tomar foto:', error);
      Alert.alert('Error', 'No se pudo tomar la foto');
      return null;
    }
  }

  // Abrir galería para seleccionar archivos
  static async selectFromGallery(options = {}) {
    try {
      const result = await launchImageLibrary({
        ...this.galleryOptions,
        ...options,
      });

      if (result.didCancel) {
        console.log('Usuario canceló la selección');
        return null;
      }

      if (result.errorCode) {
        throw new Error(result.errorMessage);
      }

      return result.assets || [];
    } catch (error) {
      console.error('Error al seleccionar de galería:', error);
      Alert.alert('Error', 'No se pudo acceder a la galería');
      return null;
    }
  }
}
```

### 2. Componente de Cámara Avanzado

```javascript
import React, { useState, useRef, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Image,
  Alert,
  ActivityIndicator,
} from 'react-native';
import { CameraManager } from './CameraManager';

const AdvancedCamera = () => {
  const [capturedImage, setCapturedImage] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [cameraMode, setCameraMode] = useState('photo'); // 'photo' o 'video'

  // Tomar foto
  const handleTakePhoto = async () => {
    setIsLoading(true);
    try {
      const photo = await CameraManager.takePhoto({
        mediaType: cameraMode,
        quality: 0.9,
        saveToPhotos: true,
      });

      if (photo) {
        setCapturedImage(photo);
        Alert.alert('Éxito', 'Foto capturada correctamente');
      }
    } catch (error) {
      console.error('Error al tomar foto:', error);
    } finally {
      setIsLoading(false);
    }
  };

  // Seleccionar de galería
  const handleSelectFromGallery = async () => {
    setIsLoading(true);
    try {
      const assets = await CameraManager.selectFromGallery({
        mediaType: cameraMode,
        selectionLimit: 1,
      });

      if (assets && assets.length > 0) {
        setCapturedImage(assets[0]);
        Alert.alert('Éxito', 'Archivo seleccionado correctamente');
      }
    } catch (error) {
      console.error('Error al seleccionar:', error);
    } finally {
      setIsLoading(false);
    }
  };

  // Cambiar modo de cámara
  const toggleCameraMode = () => {
    setCameraMode(prev => prev === 'photo' ? 'video' : 'photo');
  };

  // Limpiar imagen capturada
  const clearImage = () => {
    setCapturedImage(null);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Cámara Avanzada</Text>
      
      {/* Selector de modo */}
      <View style={styles.modeSelector}>
        <TouchableOpacity
          style={[
            styles.modeButton,
            cameraMode === 'photo' && styles.modeButtonActive,
          ]}
          onPress={() => setCameraMode('photo')}
        >
          <Text style={[
            styles.modeButtonText,
            cameraMode === 'photo' && styles.modeButtonTextActive,
          ]}>
            📸 Foto
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[
            styles.modeButton,
            cameraMode === 'video' && styles.modeButtonActive,
          ]}
          onPress={() => setCameraMode('video')}
        >
          <Text style={[
            styles.modeButtonText,
            cameraMode === 'video' && styles.modeButtonTextActive,
          ]}>
            🎥 Video
          </Text>
        </TouchableOpacity>
      </View>

      {/* Imagen capturada */}
      {capturedImage && (
        <View style={styles.imageContainer}>
          <Image
            source={{ uri: capturedImage.uri }}
            style={styles.capturedImage}
            resizeMode="contain"
          />
          <Text style={styles.imageInfo}>
            Tamaño: {capturedImage.fileSize} bytes
          </Text>
          <TouchableOpacity style={styles.clearButton} onPress={clearImage}>
            <Text style={styles.clearButtonText}>🗑️ Limpiar</Text>
          </TouchableOpacity>
        </View>
      )}

      {/* Controles de cámara */}
      <View style={styles.controls}>
        <TouchableOpacity
          style={styles.captureButton}
          onPress={handleTakePhoto}
          disabled={isLoading}
        >
          {isLoading ? (
            <ActivityIndicator color="white" />
          ) : (
            <Text style={styles.captureButtonText}>
              {cameraMode === 'photo' ? '📸 Tomar Foto' : '🎥 Grabar Video'}
            </Text>
          )}
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.galleryButton}
          onPress={handleSelectFromGallery}
          disabled={isLoading}
        >
          <Text style={styles.galleryButtonText}>🖼️ Galería</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333',
  },
  modeSelector: {
    flexDirection: 'row',
    marginBottom: 20,
    gap: 10,
  },
  modeButton: {
    flex: 1,
    padding: 15,
    borderRadius: 8,
    backgroundColor: '#e0e0e0',
    alignItems: 'center',
  },
  modeButtonActive: {
    backgroundColor: '#2196F3',
  },
  modeButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#666',
  },
  modeButtonTextActive: {
    color: 'white',
  },
  imageContainer: {
    alignItems: 'center',
    marginBottom: 20,
  },
  capturedImage: {
    width: 300,
    height: 300,
    borderRadius: 8,
    marginBottom: 10,
  },
  imageInfo: {
    fontSize: 14,
    color: '#666',
    marginBottom: 10,
  },
  clearButton: {
    backgroundColor: '#f44336',
    padding: 10,
    borderRadius: 6,
  },
  clearButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: '600',
  },
  controls: {
    gap: 15,
  },
  captureButton: {
    backgroundColor: '#4CAF50',
    padding: 20,
    borderRadius: 8,
    alignItems: 'center',
  },
  captureButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  galleryButton: {
    backgroundColor: '#FF9800',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  galleryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
});

export default AdvancedCamera;
```

### 3. Procesamiento de Imágenes

```javascript
import { Image } from 'react-native';
import { manipulateAsync, SaveFormat } from 'expo-image-manipulator';

class ImageProcessor {
  // Redimensionar imagen
  static async resizeImage(uri, width, height) {
    try {
      const result = await manipulateAsync(
        uri,
        [{ resize: { width, height } }],
        { compress: 0.8, format: SaveFormat.JPEG }
      );
      return result.uri;
    } catch (error) {
      console.error('Error al redimensionar imagen:', error);
      return uri;
    }
  }

  // Comprimir imagen
  static async compressImage(uri, quality = 0.8) {
    try {
      const result = await manipulateAsync(
        uri,
        [],
        { compress: quality, format: SaveFormat.JPEG }
      );
      return result.uri;
    } catch (error) {
      console.error('Error al comprimir imagen:', error);
      return uri;
    }
  }

  // Aplicar filtros
  static async applyFilter(uri, filter) {
    try {
      const manipulations = [];
      
      switch (filter) {
        case 'grayscale':
          manipulations.push({ grayscale: 1 });
          break;
        case 'sepia':
          manipulations.push({ sepia: 0.8 });
          break;
        case 'blur':
          manipulations.push({ blur: 2 });
          break;
        case 'sharpen':
          manipulations.push({ sharpen: 1 });
          break;
      }

      const result = await manipulateAsync(uri, manipulations, {
        compress: 0.9,
        format: SaveFormat.JPEG,
      });
      
      return result.uri;
    } catch (error) {
      console.error('Error al aplicar filtro:', error);
      return uri;
    }
  }
}
```

## Ejercicios Prácticos

### Ejercicio 1: App de Cámara Completa
Crea una aplicación completa que incluya cámara, galería y edición de imágenes.

### Ejercicio 2: Selector de Múltiples Imágenes
Implementa un selector que permita elegir múltiples imágenes de la galería.

### Ejercicio 3: Editor de Imágenes
Crea un editor básico con filtros, recorte y ajustes de brillo/contraste.

## Resumen de la Clase

Hemos aprendido:
1. **Acceso a Cámara**: Permisos y configuración
2. **Galería**: Selección de archivos multimedia
3. **Captura**: Fotos y videos con opciones personalizables
4. **Procesamiento**: Edición y manipulación de imágenes
5. **Componentes**: Interfaces reutilizables para multimedia

## Navegación
- **Anterior**: [Clase 1: Fundamentos de APIs Nativas](clase_1_fundamentos_apis_nativas.md)
- **Siguiente**: [Clase 3: Geolocalización y Mapas](clase_3_geolocalizacion_mapas.md)
- **Inicio**: [Índice del Módulo](../senior_2/README.md)

## Próxima Clase
En la siguiente clase aprenderemos sobre **Geolocalización y Mapas**, incluyendo GPS, coordenadas y mapas interactivos.
