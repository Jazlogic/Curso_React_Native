# Clase 3: Geolocalización y Mapas 🗺️

## Objetivos de la Clase
- Implementar geolocalización GPS en React Native
- Trabajar con coordenadas y cálculos de distancia
- Integrar mapas interactivos con react-native-maps
- Implementar geocodificación y búsqueda de direcciones
- Crear un sistema de tracking de ubicación

## Duración Estimada
**2 horas**

## Contenido Teórico

### 1. Geolocalización GPS

Para implementar geolocalización en React Native, usamos `@react-native-community/geolocation`:

```javascript
import Geolocation from '@react-native-community/geolocation';
import { PermissionsAndroid, Platform, Alert } from 'react-native';

class LocationManager {
  // Configuración por defecto para geolocalización
  static locationOptions = {
    enableHighAccuracy: true, // Alta precisión GPS
    timeout: 15000, // Timeout de 15 segundos
    maximumAge: 10000, // Cache de ubicación por 10 segundos
  };

  // Solicitar permiso de ubicación
  static async requestLocationPermission() {
    if (Platform.OS === 'android') {
      try {
        const granted = await PermissionsAndroid.request(
          PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
          {
            title: 'Permiso de Ubicación',
            message: 'La app necesita acceso a tu ubicación para mostrar servicios cercanos',
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

  // Obtener ubicación actual
  static getCurrentPosition() {
    return new Promise((resolve, reject) => {
      Geolocation.getCurrentPosition(
        (position) => {
          const location = {
            latitude: position.coords.latitude,
            longitude: position.coords.longitude,
            accuracy: position.coords.accuracy,
            altitude: position.coords.altitude,
            heading: position.coords.heading,
            speed: position.coords.speed,
            timestamp: position.timestamp,
          };
          resolve(location);
        },
        (error) => {
          reject(new Error(`Error de geolocalización: ${error.message}`));
        },
        this.locationOptions
      );
    });
  }

  // Observar cambios de ubicación
  static watchPosition(onLocationChange, onError) {
    return Geolocation.watchPosition(
      (position) => {
        const location = {
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          accuracy: position.coords.accuracy,
          altitude: position.coords.altitude,
          heading: position.coords.heading,
          speed: position.coords.speed,
          timestamp: position.timestamp,
        };
        onLocationChange(location);
      },
      onError,
      this.locationOptions
    );
  }

  // Detener observación de ubicación
  static clearWatch(watchId) {
    Geolocation.clearWatch(watchId);
  }

  // Calcular distancia entre dos puntos (fórmula de Haversine)
  static calculateDistance(lat1, lon1, lat2, lon2) {
    const R = 6371; // Radio de la Tierra en kilómetros
    const dLat = this.deg2rad(lat2 - lat1);
    const dLon = this.deg2rad(lon2 - lon1);
    
    const a = 
      Math.sin(dLat / 2) * Math.sin(dLat / 2) +
      Math.cos(this.deg2rad(lat1)) * Math.cos(this.deg2rad(lat2)) *
      Math.sin(dLon / 2) * Math.sin(dLon / 2);
    
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const distance = R * c; // Distancia en kilómetros
    
    return distance;
  }

  // Convertir grados a radianes
  static deg2rad(deg) {
    return deg * (Math.PI / 180);
  }

  // Obtener dirección desde coordenadas (geocodificación inversa)
  static async reverseGeocode(latitude, longitude) {
    try {
      const response = await fetch(
        `https://maps.googleapis.com/maps/api/geocode/json?latlng=${latitude},${longitude}&key=YOUR_API_KEY`
      );
      const data = await response.json();
      
      if (data.results && data.results.length > 0) {
        return data.results[0].formatted_address;
      }
      
      return 'Dirección no encontrada';
    } catch (error) {
      console.error('Error en geocodificación inversa:', error);
      return 'Error al obtener dirección';
    }
  }
}
```

### 2. Componente de Mapa Interactivo

```javascript
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator,
} from 'react-native';
import MapView, { Marker, Circle, Polyline } from 'react-native-maps';
import { LocationManager } from './LocationManager';

const InteractiveMap = () => {
  const [currentLocation, setCurrentLocation] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [watchId, setWatchId] = useState(null);
  const [routePoints, setRoutePoints] = useState([]);
  const mapRef = useRef(null);

  // Obtener ubicación inicial
  useEffect(() => {
    initializeLocation();
    return () => {
      if (watchId) {
        LocationManager.clearWatch(watchId);
      }
    };
  }, []);

  // Inicializar ubicación
  const initializeLocation = async () => {
    try {
      const hasPermission = await LocationManager.requestLocationPermission();
      if (!hasPermission) {
        Alert.alert('Permiso Denegado', 'No se puede acceder a la ubicación');
        setIsLoading(false);
        return;
      }

      const location = await LocationManager.getCurrentPosition();
      setCurrentLocation(location);
      
      // Iniciar seguimiento de ubicación
      const newWatchId = LocationManager.watchPosition(
        (newLocation) => {
          setCurrentLocation(newLocation);
          setRoutePoints(prev => [...prev, newLocation]);
        },
        (error) => {
          console.error('Error en seguimiento de ubicación:', error);
        }
      );
      
      setWatchId(newWatchId);
    } catch (error) {
      console.error('Error al inicializar ubicación:', error);
      Alert.alert('Error', 'No se pudo obtener la ubicación');
    } finally {
      setIsLoading(false);
    }
  };

  // Centrar mapa en ubicación actual
  const centerOnCurrentLocation = () => {
    if (currentLocation && mapRef.current) {
      mapRef.current.animateToRegion({
        latitude: currentLocation.latitude,
        longitude: currentLocation.longitude,
        latitudeDelta: 0.01,
        longitudeDelta: 0.01,
      });
    }
  };

  // Limpiar ruta
  const clearRoute = () => {
    setRoutePoints([]);
  };

  // Calcular distancia total de la ruta
  const calculateTotalDistance = () => {
    if (routePoints.length < 2) return 0;
    
    let totalDistance = 0;
    for (let i = 1; i < routePoints.length; i++) {
      const prev = routePoints[i - 1];
      const curr = routePoints[i];
      totalDistance += LocationManager.calculateDistance(
        prev.latitude,
        prev.longitude,
        curr.latitude,
        curr.longitude
      );
    }
    
    return totalDistance.toFixed(2);
  };

  if (isLoading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#2196F3" />
        <Text style={styles.loadingText}>Obteniendo ubicación...</Text>
      </View>
    );
  }

  if (!currentLocation) {
    return (
      <View style={styles.errorContainer}>
        <Text style={styles.errorText}>No se pudo obtener la ubicación</Text>
        <TouchableOpacity style={styles.retryButton} onPress={initializeLocation}>
          <Text style={styles.retryButtonText}>🔄 Reintentar</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <MapView
        ref={mapRef}
        style={styles.map}
        initialRegion={{
          latitude: currentLocation.latitude,
          longitude: currentLocation.longitude,
          latitudeDelta: 0.01,
          longitudeDelta: 0.01,
        }}
        showsUserLocation={true}
        showsMyLocationButton={false}
        showsCompass={true}
        showsScale={true}
      >
        {/* Marcador de ubicación actual */}
        <Marker
          coordinate={{
            latitude: currentLocation.latitude,
            longitude: currentLocation.longitude,
          }}
          title="Tu ubicación"
          description={`Precisión: ${currentLocation.accuracy}m`}
          pinColor="blue"
        />

        {/* Círculo de precisión */}
        <Circle
          center={{
            latitude: currentLocation.latitude,
            longitude: currentLocation.longitude,
          }}
          radius={currentLocation.accuracy}
          fillColor="rgba(0, 150, 255, 0.2)"
          strokeColor="rgba(0, 150, 255, 0.5)"
          strokeWidth={2}
        />

        {/* Ruta trazada */}
        {routePoints.length > 1 && (
          <Polyline
            coordinates={routePoints.map(point => ({
              latitude: point.latitude,
              longitude: point.longitude,
            }))}
            strokeColor="#FF6B6B"
            strokeWidth={3}
            lineDashPattern={[5, 5]}
          />
        )}
      </MapView>

      {/* Panel de controles */}
      <View style={styles.controls}>
        <TouchableOpacity
          style={styles.controlButton}
          onPress={centerOnCurrentLocation}
        >
          <Text style={styles.controlButtonText}>📍 Centrar</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.controlButton}
          onPress={clearRoute}
        >
          <Text style={styles.controlButtonText}>🗑️ Limpiar Ruta</Text>
        </TouchableOpacity>
      </View>

      {/* Información de la ruta */}
      {routePoints.length > 1 && (
        <View style={styles.routeInfo}>
          <Text style={styles.routeInfoText}>
            📍 Puntos: {routePoints.length}
          </Text>
          <Text style={styles.routeInfoText}>
            🚶 Distancia: {calculateTotalDistance()} km
          </Text>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  errorText: {
    fontSize: 16,
    color: '#f44336',
    marginBottom: 20,
    textAlign: 'center',
  },
  retryButton: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  controls: {
    flexDirection: 'row',
    padding: 20,
    gap: 10,
    backgroundColor: 'white',
  },
  controlButton: {
    flex: 1,
    backgroundColor: '#4CAF50',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  controlButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: '600',
  },
  routeInfo: {
    backgroundColor: 'white',
    padding: 15,
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
  },
  routeInfoText: {
    fontSize: 14,
    color: '#333',
    marginBottom: 5,
  },
});

export default InteractiveMap;
```

### 3. Sistema de Búsqueda y Geocodificación

```javascript
import { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  FlatList,
  Alert,
} from 'react-native';

class GeocodingService {
  // Buscar direcciones (geocodificación)
  static async searchAddress(query) {
    try {
      const response = await fetch(
        `https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURIComponent(query)}&key=YOUR_API_KEY`
      );
      const data = await response.json();
      
      if (data.results && data.results.length > 0) {
        return data.results.map(result => ({
          address: result.formatted_address,
          location: result.geometry.location,
          placeId: result.place_id,
        }));
      }
      
      return [];
    } catch (error) {
      console.error('Error en búsqueda de direcciones:', error);
      throw new Error('Error al buscar direcciones');
    }
  }

  // Obtener detalles de un lugar
  static async getPlaceDetails(placeId) {
    try {
      const response = await fetch(
        `https://maps.googleapis.com/maps/api/place/details/json?place_id=${placeId}&key=YOUR_API_KEY`
      );
      const data = await response.json();
      
      if (data.result) {
        return {
          name: data.result.name,
          address: data.result.formatted_address,
          phone: data.result.formatted_phone_number,
          website: data.result.website,
          rating: data.result.rating,
          types: data.result.types,
        };
      }
      
      return null;
    } catch (error) {
      console.error('Error al obtener detalles del lugar:', error);
      throw new Error('Error al obtener detalles');
    }
  }
}

const AddressSearch = ({ onLocationSelect }) => {
  const [searchQuery, setSearchQuery] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [isSearching, setIsSearching] = useState(false);

  // Buscar direcciones
  const handleSearch = async () => {
    if (!searchQuery.trim()) return;

    setIsSearching(true);
    try {
      const results = await GeocodingService.searchAddress(searchQuery);
      setSearchResults(results);
    } catch (error) {
      Alert.alert('Error', error.message);
    } finally {
      setIsSearching(false);
    }
  };

  // Seleccionar ubicación
  const handleLocationSelect = (location) => {
    onLocationSelect(location);
    setSearchResults([]);
    setSearchQuery('');
  };

  // Renderizar resultado de búsqueda
  const renderSearchResult = ({ item }) => (
    <TouchableOpacity
      style={styles.searchResult}
      onPress={() => handleLocationSelect(item)}
    >
      <Text style={styles.resultAddress}>{item.address}</Text>
      <Text style={styles.resultCoordinates}>
        {item.location.lat.toFixed(6)}, {item.location.lng.toFixed(6)}
      </Text>
    </TouchableOpacity>
  );

  return (
    <View style={styles.container}>
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Buscar dirección..."
          value={searchQuery}
          onChangeText={setSearchQuery}
          onSubmitEditing={handleSearch}
        />
        <TouchableOpacity
          style={styles.searchButton}
          onPress={handleSearch}
          disabled={isSearching}
        >
          <Text style={styles.searchButtonText}>
            {isSearching ? '🔍' : '🔍'}
          </Text>
        </TouchableOpacity>
      </View>

      {searchResults.length > 0 && (
        <FlatList
          data={searchResults}
          renderItem={renderSearchResult}
          keyExtractor={(item) => item.placeId}
          style={styles.resultsList}
        />
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: 'white',
    padding: 20,
  },
  searchContainer: {
    flexDirection: 'row',
    gap: 10,
  },
  searchInput: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 15,
    fontSize: 16,
  },
  searchButton: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
    minWidth: 50,
  },
  searchButtonText: {
    color: 'white',
    fontSize: 18,
  },
  resultsList: {
    marginTop: 10,
  },
  searchResult: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  resultAddress: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
    marginBottom: 5,
  },
  resultCoordinates: {
    fontSize: 14,
    color: '#666',
  },
});

export default AddressSearch;
```

## Ejercicios Prácticos

### Ejercicio 1: App de Tracking GPS
Crea una aplicación que registre la ruta del usuario y muestre estadísticas de distancia y tiempo.

### Ejercicio 2: Buscador de Lugares Cercanos
Implementa un sistema que encuentre lugares cercanos a la ubicación actual del usuario.

### Ejercicio 3: Navegador con Instrucciones
Crea un navegador básico que muestre instrucciones paso a paso entre dos puntos.

## Resumen de la Clase

Hemos aprendido:
1. **Geolocalización GPS**: Permisos y obtención de ubicación
2. **Mapas Interactivos**: Integración con react-native-maps
3. **Cálculos de Distancia**: Fórmula de Haversine
4. **Geocodificación**: Búsqueda de direcciones y coordenadas
5. **Tracking de Ubicación**: Seguimiento en tiempo real

## Navegación
- **Anterior**: [Clase 2: Cámara y Galería](clase_2_camara_galeria.md)
- **Siguiente**: [Clase 4: Notificaciones Push y Locales](clase_4_notificaciones_push_locales.md)
- **Inicio**: [Índice del Módulo](../senior_2/README.md)

## Próxima Clase
En la siguiente clase aprenderemos sobre **Notificaciones Push y Locales**, incluyendo notificaciones push, locales, badges y deep linking.
