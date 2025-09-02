# Clase 5: Casos de Uso Avanzados y Proyectos Integradores

## Objetivos de la Clase
- Implementar proyectos complejos que integren juegos, AR y sensores
- Desarrollar aplicaciones de nivel empresarial
- Aplicar todas las técnicas aprendidas en proyectos reales
- Crear portafolio de proyectos avanzados

## Contenido de la Clase

### 1. Proyecto 1: Aplicación de Fitness con AR

#### Descripción del Proyecto
Una aplicación de fitness que utiliza AR para guiar ejercicios y sensores para monitorear el rendimiento.

#### Estructura del Proyecto
```jsx
// FitnessARApp.js
import React, { useState, useEffect } from 'react';
import { View, StyleSheet, Text, TouchableOpacity, Alert } from 'react-native';
import { ARView, ARPlane, ARModel } from 'react-native-ar';
import { accelerometer, gyroscope } from 'react-native-sensors';
import { Vibration } from 'react-native';

const FitnessARApp = () => {
  const [currentExercise, setCurrentExercise] = useState(null);
  const [exerciseData, setExerciseData] = useState({
    reps: 0,
    sets: 0,
    calories: 0,
    heartRate: 0
  });
  const [isTracking, setIsTracking] = useState(false);
  const [arPlane, setArPlane] = useState(null);

  const exercises = [
    {
      id: 1,
      name: 'Push-ups',
      model: 'pushup_model.obj',
      targetReps: 10,
      instructions: 'Coloca las manos en el suelo y realiza flexiones'
    },
    {
      id: 2,
      name: 'Squats',
      model: 'squat_model.obj',
      targetReps: 15,
      instructions: 'Baja y sube manteniendo la espalda recta'
    },
    {
      id: 3,
      name: 'Plank',
      model: 'plank_model.obj',
      targetReps: 1,
      instructions: 'Mantén la posición de plancha durante 30 segundos'
    }
  ];

  useEffect(() => {
    if (isTracking && currentExercise) {
      startExerciseTracking();
    }
  }, [isTracking, currentExercise]);

  const startExerciseTracking = () => {
    const subscription = accelerometer.subscribe(({ x, y, z }) => {
      const magnitude = Math.sqrt(x * x + y * y + z * z);
      
      // Detectar repeticiones basado en el tipo de ejercicio
      if (currentExercise.name === 'Push-ups' && magnitude > 15) {
        setExerciseData(prev => ({
          ...prev,
          reps: prev.reps + 1
        }));
        Vibration.vibrate(100);
      }
    });

    return () => subscription.unsubscribe();
  };

  const onPlaneDetected = (plane) => {
    setArPlane(plane);
  };

  const startExercise = (exercise) => {
    setCurrentExercise(exercise);
    setIsTracking(true);
    setExerciseData({
      reps: 0,
      sets: 0,
      calories: 0,
      heartRate: 0
    });
  };

  const stopExercise = () => {
    setIsTracking(false);
    setCurrentExercise(null);
  };

  const completeSet = () => {
    if (exerciseData.reps >= currentExercise.targetReps) {
      setExerciseData(prev => ({
        ...prev,
        sets: prev.sets + 1,
        reps: 0
      }));
      Alert.alert('¡Set completado!', 'Bien hecho, descansa un momento');
    } else {
      Alert.alert('Continúa', `Te faltan ${currentExercise.targetReps - exerciseData.reps} repeticiones`);
    }
  };

  return (
    <View style={styles.container}>
      {!currentExercise ? (
        <View style={styles.exerciseSelection}>
          <Text style={styles.title}>Selecciona un Ejercicio</Text>
          {exercises.map((exercise) => (
            <TouchableOpacity
              key={exercise.id}
              style={styles.exerciseButton}
              onPress={() => startExercise(exercise)}
            >
              <Text style={styles.exerciseButtonText}>{exercise.name}</Text>
            </TouchableOpacity>
          ))}
        </View>
      ) : (
        <View style={styles.exerciseView}>
          <ARView style={styles.arView} onPlaneDetected={onPlaneDetected}>
            {arPlane && (
              <ARModel
                source={currentExercise.model}
                position={arPlane.position}
                rotation={[0, 0, 0]}
                scale={[0.5, 0.5, 0.5]}
              />
            )}
          </ARView>
          
          <View style={styles.exerciseInfo}>
            <Text style={styles.exerciseName}>{currentExercise.name}</Text>
            <Text style={styles.instructions}>{currentExercise.instructions}</Text>
            
            <View style={styles.statsContainer}>
              <View style={styles.stat}>
                <Text style={styles.statLabel}>Repeticiones</Text>
                <Text style={styles.statValue}>{exerciseData.reps}/{currentExercise.targetReps}</Text>
              </View>
              
              <View style={styles.stat}>
                <Text style={styles.statLabel}>Sets</Text>
                <Text style={styles.statValue}>{exerciseData.sets}</Text>
              </View>
            </View>
            
            <View style={styles.controls}>
              <TouchableOpacity style={styles.controlButton} onPress={completeSet}>
                <Text style={styles.controlButtonText}>Completar Set</Text>
              </TouchableOpacity>
              
              <TouchableOpacity style={styles.controlButton} onPress={stopExercise}>
                <Text style={styles.controlButtonText}>Finalizar</Text>
              </TouchableOpacity>
            </View>
          </View>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  exerciseSelection: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
    textAlign: 'center'
  },
  exerciseButton: {
    backgroundColor: '#4CAF50',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    minWidth: 200,
    alignItems: 'center'
  },
  exerciseButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  },
  exerciseView: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  exerciseInfo: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(255,255,255,0.95)',
    padding: 20,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20
  },
  exerciseName: {
    fontSize: 20,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 10
  },
  instructions: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 20,
    color: '#666'
  },
  statsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20
  },
  stat: {
    alignItems: 'center'
  },
  statLabel: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5
  },
  statValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333'
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-around'
  },
  controlButton: {
    backgroundColor: '#2196F3',
    padding: 15,
    borderRadius: 10,
    minWidth: 120,
    alignItems: 'center'
  },
  controlButtonText: {
    color: 'white',
    fontWeight: 'bold'
  }
});

export default FitnessARApp;
```

### 2. Proyecto 2: Juego de Realidad Aumentada

#### Descripción del Proyecto
Un juego de estrategia en AR donde los jugadores construyen y defienden bases en el mundo real.

#### Implementación del Juego
```jsx
// ARStrategyGame.js
import React, { useState, useEffect, useRef } from 'react';
import { View, StyleSheet, Text, TouchableOpacity, Alert } from 'react-native';
import { ARView, ARPlane, ARBox, ARSphere } from 'react-native-ar';
import { Vibration } from 'react-native';

const ARStrategyGame = () => {
  const [gameState, setGameState] = useState('menu'); // menu, playing, paused
  const [playerResources, setPlayerResources] = useState({
    energy: 100,
    materials: 50,
    units: 0
  });
  const [buildings, setBuildings] = useState([]);
  const [enemies, setEnemies] = useState([]);
  const [selectedBuilding, setSelectedBuilding] = useState(null);
  const [arPlane, setArPlane] = useState(null);
  const gameLoop = useRef(null);

  const buildingTypes = [
    {
      id: 'generator',
      name: 'Generador',
      cost: { energy: 0, materials: 20 },
      model: 'generator.obj',
      production: { energy: 5, materials: 0 }
    },
    {
      id: 'factory',
      name: 'Fábrica',
      cost: { energy: 30, materials: 40 },
      model: 'factory.obj',
      production: { energy: 0, materials: 10 }
    },
    {
      id: 'barracks',
      name: 'Cuartel',
      cost: { energy: 50, materials: 30 },
      model: 'barracks.obj',
      production: { energy: 0, materials: 0, units: 1 }
    }
  ];

  useEffect(() => {
    if (gameState === 'playing') {
      startGameLoop();
    } else {
      stopGameLoop();
    }

    return () => stopGameLoop();
  }, [gameState]);

  const startGameLoop = () => {
    gameLoop.current = setInterval(() => {
      updateGame();
    }, 1000);
  };

  const stopGameLoop = () => {
    if (gameLoop.current) {
      clearInterval(gameLoop.current);
      gameLoop.current = null;
    }
  };

  const updateGame = () => {
    // Actualizar recursos basado en edificios
    setPlayerResources(prev => {
      let newResources = { ...prev };
      
      buildings.forEach(building => {
        const buildingType = buildingTypes.find(bt => bt.id === building.type);
        if (buildingType) {
          newResources.energy += buildingType.production.energy;
          newResources.materials += buildingType.production.materials;
          newResources.units += buildingType.production.units || 0;
        }
      });
      
      return newResources;
    });

    // Generar enemigos aleatoriamente
    if (Math.random() < 0.1) {
      spawnEnemy();
    }

    // Actualizar posición de enemigos
    setEnemies(prev => prev.map(enemy => ({
      ...enemy,
      position: [
        enemy.position[0] + enemy.velocity[0],
        enemy.position[1],
        enemy.position[2] + enemy.velocity[2]
      ]
    })));
  };

  const spawnEnemy = () => {
    const newEnemy = {
      id: Date.now(),
      position: [Math.random() * 2 - 1, 0, -2],
      velocity: [0, 0, 0.1],
      health: 100,
      type: 'basic'
    };
    setEnemies(prev => [...prev, newEnemy]);
  };

  const onPlaneDetected = (plane) => {
    setArPlane(plane);
  };

  const startGame = () => {
    setGameState('playing');
    setPlayerResources({ energy: 100, materials: 50, units: 0 });
    setBuildings([]);
    setEnemies([]);
  };

  const pauseGame = () => {
    setGameState('paused');
  };

  const resumeGame = () => {
    setGameState('playing');
  };

  const selectBuilding = (buildingType) => {
    if (canAfford(buildingType.cost)) {
      setSelectedBuilding(buildingType);
    } else {
      Alert.alert('Recursos insuficientes', 'No tienes suficientes recursos para construir este edificio');
    }
  };

  const canAfford = (cost) => {
    return playerResources.energy >= cost.energy && playerResources.materials >= cost.materials;
  };

  const placeBuilding = (position) => {
    if (selectedBuilding && arPlane) {
      const newBuilding = {
        id: Date.now(),
        type: selectedBuilding.id,
        position: position,
        health: 100
      };
      
      setBuildings(prev => [...prev, newBuilding]);
      setPlayerResources(prev => ({
        ...prev,
        energy: prev.energy - selectedBuilding.cost.energy,
        materials: prev.materials - selectedBuilding.cost.materials
      }));
      setSelectedBuilding(null);
      Vibration.vibrate(100);
    }
  };

  const attackEnemy = (enemyId) => {
    setEnemies(prev => prev.filter(enemy => enemy.id !== enemyId));
    Vibration.vibrate(200);
  };

  return (
    <View style={styles.container}>
      {gameState === 'menu' && (
        <View style={styles.menu}>
          <Text style={styles.title}>AR Strategy Game</Text>
          <TouchableOpacity style={styles.menuButton} onPress={startGame}>
            <Text style={styles.menuButtonText}>Iniciar Juego</Text>
          </TouchableOpacity>
        </View>
      )}

      {gameState === 'playing' && (
        <View style={styles.gameView}>
          <ARView style={styles.arView} onPlaneDetected={onPlaneDetected}>
            {arPlane && (
              <ARPlane
                position={arPlane.position}
                rotation={arPlane.rotation}
                scale={arPlane.scale}
                onPress={(event) => placeBuilding(event.nativeEvent.location)}
                opacity={0.3}
              />
            )}
            
            {buildings.map((building) => {
              const buildingType = buildingTypes.find(bt => bt.id === building.type);
              return (
                <ARBox
                  key={building.id}
                  position={building.position}
                  rotation={[0, 0, 0]}
                  scale={[0.2, 0.2, 0.2]}
                  color="blue"
                />
              );
            })}
            
            {enemies.map((enemy) => (
              <ARSphere
                key={enemy.id}
                position={enemy.position}
                rotation={[0, 0, 0]}
                scale={[0.1, 0.1, 0.1]}
                color="red"
                onPress={() => attackEnemy(enemy.id)}
              />
            ))}
          </ARView>
          
          <View style={styles.gameUI}>
            <View style={styles.resources}>
              <Text style={styles.resourceText}>Energía: {playerResources.energy}</Text>
              <Text style={styles.resourceText}>Materiales: {playerResources.materials}</Text>
              <Text style={styles.resourceText}>Unidades: {playerResources.units}</Text>
            </View>
            
            <View style={styles.buildings}>
              {buildingTypes.map((building) => (
                <TouchableOpacity
                  key={building.id}
                  style={[
                    styles.buildingButton,
                    !canAfford(building.cost) && styles.disabledButton
                  ]}
                  onPress={() => selectBuilding(building)}
                >
                  <Text style={styles.buildingButtonText}>{building.name}</Text>
                </TouchableOpacity>
              ))}
            </View>
            
            <TouchableOpacity style={styles.pauseButton} onPress={pauseGame}>
              <Text style={styles.pauseButtonText}>Pausa</Text>
            </TouchableOpacity>
          </View>
        </View>
      )}

      {gameState === 'paused' && (
        <View style={styles.pauseMenu}>
          <Text style={styles.title}>Juego Pausado</Text>
          <TouchableOpacity style={styles.menuButton} onPress={resumeGame}>
            <Text style={styles.menuButtonText}>Continuar</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.menuButton} onPress={() => setGameState('menu')}>
            <Text style={styles.menuButtonText}>Menú Principal</Text>
          </TouchableOpacity>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  menu: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 30,
    textAlign: 'center'
  },
  menuButton: {
    backgroundColor: '#4CAF50',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    minWidth: 200,
    alignItems: 'center'
  },
  menuButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  },
  gameView: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  gameUI: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(255,255,255,0.95)',
    padding: 20,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20
  },
  resources: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20
  },
  resourceText: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333'
  },
  buildings: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20
  },
  buildingButton: {
    backgroundColor: '#2196F3',
    padding: 10,
    borderRadius: 5,
    minWidth: 80,
    alignItems: 'center'
  },
  disabledButton: {
    backgroundColor: '#ccc'
  },
  buildingButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold'
  },
  pauseButton: {
    backgroundColor: '#f44336',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center'
  },
  pauseButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold'
  },
  pauseMenu: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: 'rgba(0,0,0,0.8)'
  }
});

export default ARStrategyGame;
```

### 3. Proyecto 3: Aplicación de Navegación AR

#### Descripción del Proyecto
Una aplicación de navegación que utiliza AR para mostrar direcciones en el mundo real.

#### Implementación de Navegación AR
```jsx
// ARNavigationApp.js
import React, { useState, useEffect, useRef } from 'react';
import { View, StyleSheet, Text, TouchableOpacity, Alert } from 'react-native';
import { ARView, ARPlane, ARBox } from 'react-native-ar';
import Geolocation from '@react-native-community/geolocation';
import { Vibration } from 'react-native';

const ARNavigationApp = () => {
  const [currentLocation, setCurrentLocation] = useState(null);
  const [destination, setDestination] = useState(null);
  const [navigationActive, setNavigationActive] = useState(false);
  const [distance, setDistance] = useState(0);
  const [direction, setDirection] = useState(0);
  const [arPlane, setArPlane] = useState(null);
  const navigationInterval = useRef(null);

  const destinations = [
    {
      id: 1,
      name: 'Estación de Metro',
      coordinates: { latitude: 40.4168, longitude: -3.7038 },
      distance: 500
    },
    {
      id: 2,
      name: 'Centro Comercial',
      coordinates: { latitude: 40.4200, longitude: -3.7100 },
      distance: 800
    },
    {
      id: 3,
      name: 'Parque Central',
      coordinates: { latitude: 40.4100, longitude: -3.7000 },
      distance: 1200
    }
  ];

  useEffect(() => {
    getCurrentLocation();
  }, []);

  useEffect(() => {
    if (navigationActive && currentLocation && destination) {
      startNavigation();
    } else {
      stopNavigation();
    }

    return () => stopNavigation();
  }, [navigationActive, currentLocation, destination]);

  const getCurrentLocation = () => {
    Geolocation.getCurrentPosition(
      (position) => {
        setCurrentLocation({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude
        });
      },
      (error) => {
        Alert.alert('Error', 'No se pudo obtener la ubicación actual');
        console.log(error);
      },
      { enableHighAccuracy: true, timeout: 15000, maximumAge: 10000 }
    );
  };

  const startNavigation = () => {
    navigationInterval.current = setInterval(() => {
      updateNavigation();
    }, 1000);
  };

  const stopNavigation = () => {
    if (navigationInterval.current) {
      clearInterval(navigationInterval.current);
      navigationInterval.current = null;
    }
  };

  const updateNavigation = () => {
    if (currentLocation && destination) {
      const newDistance = calculateDistance(currentLocation, destination.coordinates);
      const newDirection = calculateDirection(currentLocation, destination.coordinates);
      
      setDistance(newDistance);
      setDirection(newDirection);
      
      // Vibración cuando se está cerca del destino
      if (newDistance < 50) {
        Vibration.vibrate(500);
        Alert.alert('¡Has llegado!', `Has llegado a ${destination.name}`);
        setNavigationActive(false);
      }
    }
  };

  const calculateDistance = (from, to) => {
    const R = 6371e3; // Radio de la Tierra en metros
    const φ1 = from.latitude * Math.PI / 180;
    const φ2 = to.latitude * Math.PI / 180;
    const Δφ = (to.latitude - from.latitude) * Math.PI / 180;
    const Δλ = (to.longitude - from.longitude) * Math.PI / 180;

    const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
              Math.cos(φ1) * Math.cos(φ2) *
              Math.sin(Δλ/2) * Math.sin(Δλ/2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

    return R * c;
  };

  const calculateDirection = (from, to) => {
    const φ1 = from.latitude * Math.PI / 180;
    const φ2 = to.latitude * Math.PI / 180;
    const Δλ = (to.longitude - from.longitude) * Math.PI / 180;

    const y = Math.sin(Δλ) * Math.cos(φ2);
    const x = Math.cos(φ1) * Math.sin(φ2) - Math.sin(φ1) * Math.cos(φ2) * Math.cos(Δλ);

    return Math.atan2(y, x) * 180 / Math.PI;
  };

  const onPlaneDetected = (plane) => {
    setArPlane(plane);
  };

  const startNavigationTo = (dest) => {
    setDestination(dest);
    setNavigationActive(true);
  };

  const stopNavigationTo = () => {
    setNavigationActive(false);
    setDestination(null);
  };

  const getDirectionText = (angle) => {
    const directions = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
    const index = Math.round(angle / 45) % 8;
    return directions[index];
  };

  return (
    <View style={styles.container}>
      {!navigationActive ? (
        <View style={styles.destinationSelection}>
          <Text style={styles.title}>Selecciona un Destino</Text>
          {destinations.map((dest) => (
            <TouchableOpacity
              key={dest.id}
              style={styles.destinationButton}
              onPress={() => startNavigationTo(dest)}
            >
              <Text style={styles.destinationButtonText}>{dest.name}</Text>
              <Text style={styles.destinationDistance}>{dest.distance}m</Text>
            </TouchableOpacity>
          ))}
        </View>
      ) : (
        <View style={styles.navigationView}>
          <ARView style={styles.arView} onPlaneDetected={onPlaneDetected}>
            {arPlane && (
              <ARPlane
                position={arPlane.position}
                rotation={arPlane.rotation}
                scale={arPlane.scale}
                opacity={0.3}
              />
            )}
            
            {arPlane && (
              <ARBox
                position={[
                  arPlane.position[0] + Math.sin(direction * Math.PI / 180) * 0.5,
                  arPlane.position[1],
                  arPlane.position[2] + Math.cos(direction * Math.PI / 180) * 0.5
                ]}
                rotation={[0, direction * Math.PI / 180, 0]}
                scale={[0.1, 0.1, 0.1]}
                color="green"
              />
            )}
          </ARView>
          
          <View style={styles.navigationUI}>
            <Text style={styles.destinationName}>{destination.name}</Text>
            <Text style={styles.distanceText}>{distance.toFixed(0)}m</Text>
            <Text style={styles.directionText}>{getDirectionText(direction)}</Text>
            
            <TouchableOpacity style={styles.stopButton} onPress={stopNavigationTo}>
              <Text style={styles.stopButtonText}>Detener Navegación</Text>
            </TouchableOpacity>
          </View>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  destinationSelection: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
    textAlign: 'center'
  },
  destinationButton: {
    backgroundColor: '#4CAF50',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    minWidth: 250,
    alignItems: 'center'
  },
  destinationButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 5
  },
  destinationDistance: {
    color: 'white',
    fontSize: 14
  },
  navigationView: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  navigationUI: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(255,255,255,0.95)',
    padding: 20,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
    alignItems: 'center'
  },
  destinationName: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'center'
  },
  distanceText: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#4CAF50',
    marginBottom: 5
  },
  directionText: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#2196F3',
    marginBottom: 20
  },
  stopButton: {
    backgroundColor: '#f44336',
    padding: 15,
    borderRadius: 10,
    minWidth: 200,
    alignItems: 'center'
  },
  stopButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold'
  }
});

export default ARNavigationApp;
```

### 4. Proyecto 4: Aplicación de Educación AR

#### Descripción del Proyecto
Una aplicación educativa que utiliza AR para enseñar conceptos de anatomía, química o historia.

#### Implementación de Educación AR
```jsx
// AREducationApp.js
import React, { useState, useEffect } from 'react';
import { View, StyleSheet, Text, TouchableOpacity, Alert } from 'react-native';
import { ARView, ARPlane, ARModel, ARText } from 'react-native-ar';
import { Vibration } from 'react-native';

const AREducationApp = () => {
  const [currentSubject, setCurrentSubject] = useState(null);
  const [currentLesson, setCurrentLesson] = useState(null);
  const [lessonProgress, setLessonProgress] = useState(0);
  const [arPlane, setArPlane] = useState(null);
  const [interactiveElements, setInteractiveElements] = useState([]);

  const subjects = [
    {
      id: 'anatomy',
      name: 'Anatomía',
      lessons: [
        {
          id: 'heart',
          name: 'Sistema Cardiovascular',
          model: 'heart.obj',
          description: 'Aprende sobre el corazón y el sistema circulatorio',
          interactivePoints: [
            { id: 'atrium', name: 'Aurícula', position: [0, 0.1, 0], description: 'Recibe sangre del cuerpo' },
            { id: 'ventricle', name: 'Ventrículo', position: [0, -0.1, 0], description: 'Bombea sangre al cuerpo' }
          ]
        },
        {
          id: 'brain',
          name: 'Sistema Nervioso',
          model: 'brain.obj',
          description: 'Explora el cerebro y sus funciones',
          interactivePoints: [
            { id: 'cerebrum', name: 'Cerebro', position: [0, 0.2, 0], description: 'Controla el pensamiento y movimiento' },
            { id: 'cerebellum', name: 'Cerebelo', position: [0, -0.2, 0], description: 'Controla el equilibrio y coordinación' }
          ]
        }
      ]
    },
    {
      id: 'chemistry',
      name: 'Química',
      lessons: [
        {
          id: 'molecule',
          name: 'Estructura Molecular',
          model: 'water_molecule.obj',
          description: 'Aprende sobre la estructura del agua',
          interactivePoints: [
            { id: 'oxygen', name: 'Oxígeno', position: [0, 0, 0], description: 'Átomo de oxígeno' },
            { id: 'hydrogen1', name: 'Hidrógeno 1', position: [0.1, 0, 0], description: 'Primer átomo de hidrógeno' },
            { id: 'hydrogen2', name: 'Hidrógeno 2', position: [-0.1, 0, 0], description: 'Segundo átomo de hidrógeno' }
          ]
        }
      ]
    }
  ];

  const onPlaneDetected = (plane) => {
    setArPlane(plane);
  };

  const selectSubject = (subject) => {
    setCurrentSubject(subject);
    setCurrentLesson(null);
    setLessonProgress(0);
  };

  const selectLesson = (lesson) => {
    setCurrentLesson(lesson);
    setLessonProgress(0);
    setInteractiveElements(lesson.interactivePoints);
  };

  const onInteractivePointPress = (point) => {
    Alert.alert(point.name, point.description);
    Vibration.vibrate(100);
    
    // Actualizar progreso de la lección
    setLessonProgress(prev => prev + 1);
    
    // Marcar punto como visitado
    setInteractiveElements(prev => 
      prev.map(element => 
        element.id === point.id 
          ? { ...element, visited: true }
          : element
      )
    );
  };

  const completeLesson = () => {
    Alert.alert('¡Lección Completada!', 'Has completado esta lección exitosamente');
    setCurrentLesson(null);
    setLessonProgress(0);
    setInteractiveElements([]);
  };

  const goBack = () => {
    if (currentLesson) {
      setCurrentLesson(null);
      setLessonProgress(0);
      setInteractiveElements([]);
    } else if (currentSubject) {
      setCurrentSubject(null);
    }
  };

  return (
    <View style={styles.container}>
      {!currentSubject ? (
        <View style={styles.subjectSelection}>
          <Text style={styles.title}>Selecciona una Materia</Text>
          {subjects.map((subject) => (
            <TouchableOpacity
              key={subject.id}
              style={styles.subjectButton}
              onPress={() => selectSubject(subject)}
            >
              <Text style={styles.subjectButtonText}>{subject.name}</Text>
            </TouchableOpacity>
          ))}
        </View>
      ) : !currentLesson ? (
        <View style={styles.lessonSelection}>
          <Text style={styles.title}>{currentSubject.name}</Text>
          {currentSubject.lessons.map((lesson) => (
            <TouchableOpacity
              key={lesson.id}
              style={styles.lessonButton}
              onPress={() => selectLesson(lesson)}
            >
              <Text style={styles.lessonButtonText}>{lesson.name}</Text>
              <Text style={styles.lessonDescription}>{lesson.description}</Text>
            </TouchableOpacity>
          ))}
          <TouchableOpacity style={styles.backButton} onPress={goBack}>
            <Text style={styles.backButtonText}>Volver</Text>
          </TouchableOpacity>
        </View>
      ) : (
        <View style={styles.lessonView}>
          <ARView style={styles.arView} onPlaneDetected={onPlaneDetected}>
            {arPlane && (
              <ARPlane
                position={arPlane.position}
                rotation={arPlane.rotation}
                scale={arPlane.scale}
                opacity={0.3}
              />
            )}
            
            {arPlane && (
              <ARModel
                source={currentLesson.model}
                position={arPlane.position}
                rotation={[0, 0, 0]}
                scale={[0.5, 0.5, 0.5]}
              />
            )}
            
            {interactiveElements.map((element) => (
              <ARBox
                key={element.id}
                position={[
                  arPlane.position[0] + element.position[0],
                  arPlane.position[1] + element.position[1],
                  arPlane.position[2] + element.position[2]
                ]}
                rotation={[0, 0, 0]}
                scale={[0.05, 0.05, 0.05]}
                color={element.visited ? 'green' : 'red'}
                onPress={() => onInteractivePointPress(element)}
              />
            ))}
          </ARView>
          
          <View style={styles.lessonUI}>
            <Text style={styles.lessonName}>{currentLesson.name}</Text>
            <Text style={styles.progressText}>
              Progreso: {lessonProgress}/{interactiveElements.length}
            </Text>
            
            <View style={styles.controls}>
              <TouchableOpacity style={styles.controlButton} onPress={goBack}>
                <Text style={styles.controlButtonText}>Volver</Text>
              </TouchableOpacity>
              
              {lessonProgress >= interactiveElements.length && (
                <TouchableOpacity style={styles.controlButton} onPress={completeLesson}>
                  <Text style={styles.controlButtonText}>Completar</Text>
                </TouchableOpacity>
              )}
            </View>
          </View>
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  subjectSelection: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
    textAlign: 'center'
  },
  subjectButton: {
    backgroundColor: '#4CAF50',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    minWidth: 200,
    alignItems: 'center'
  },
  subjectButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold'
  },
  lessonSelection: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20
  },
  lessonButton: {
    backgroundColor: '#2196F3',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    minWidth: 250,
    alignItems: 'center'
  },
  lessonButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 5
  },
  lessonDescription: {
    color: 'white',
    fontSize: 14,
    textAlign: 'center'
  },
  backButton: {
    backgroundColor: '#f44336',
    padding: 15,
    borderRadius: 10,
    marginTop: 20,
    minWidth: 150,
    alignItems: 'center'
  },
  backButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold'
  },
  lessonView: {
    flex: 1
  },
  arView: {
    flex: 1
  },
  lessonUI: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(255,255,255,0.95)',
    padding: 20,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
    alignItems: 'center'
  },
  lessonName: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
    textAlign: 'center'
  },
  progressText: {
    fontSize: 16,
    color: '#666',
    marginBottom: 20
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    width: '100%'
  },
  controlButton: {
    backgroundColor: '#4CAF50',
    padding: 15,
    borderRadius: 10,
    minWidth: 120,
    alignItems: 'center'
  },
  controlButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold'
  }
});

export default AREducationApp;
```

## Recursos Adicionales

### Documentación
- [React Native AR](https://github.com/react-native-ar/react-native-ar)
- [React Native Sensors](https://github.com/react-native-sensors/react-native-sensors)
- [React Native Reanimated](https://docs.swmansion.com/react-native-reanimated/)

### Herramientas
- [Blender](https://www.blender.org/) - Para crear modelos 3D
- [Unity](https://unity.com/) - Para desarrollo de AR avanzado
- [Flipper](https://fbflipper.com/) - Para debugging

### Tutoriales
- [Building AR Apps with React Native](https://blog.logrocket.com/ar-development-react-native/)
- [React Native Game Development](https://reactnative.dev/docs/game-development)

## Ejercicios Prácticos

### Ejercicio 1: Aplicación de Fitness AR
Crear una aplicación de fitness que utilice AR para guiar ejercicios.

### Ejercicio 2: Juego de Estrategia AR
Desarrollar un juego de estrategia en AR.

### Ejercicio 3: Aplicación de Navegación AR
Implementar una aplicación de navegación con AR.

### Ejercicio 4: Aplicación Educativa AR
Crear una aplicación educativa que utilice AR para enseñar conceptos.

## Evaluación

### Criterios de Evaluación
- **Funcionalidad (40%):** La aplicación funciona correctamente
- **Innovación (30%):** Uso creativo de AR y sensores
- **Código (20%):** Estructura y calidad del código
- **Presentación (10%):** Demo y documentación

### Entregables
- Código fuente de la aplicación
- Demo de la aplicación funcionando
- Documentación del proyecto
- Presentación del proyecto

---

**Duración estimada:** 3 horas  
**Dificultad:** Avanzada  
**Prerrequisitos:** Conocimientos sólidos de React Native, AR, sensores y optimización
