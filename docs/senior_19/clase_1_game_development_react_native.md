# Clase 1: Desarrollo de Juegos en React Native

## Objetivos de la Clase
- Comprender los fundamentos del desarrollo de juegos en React Native
- Aprender sobre las librerías y herramientas disponibles
- Implementar un juego básico con física y animaciones
- Entender las consideraciones de rendimiento para juegos

## Contenido de la Clase

### 1. Introducción al Desarrollo de Juegos en React Native

#### ¿Por qué React Native para Juegos?
- **Ventajas:**
  - Desarrollo multiplataforma
  - Reutilización de código
  - Ecosistema JavaScript/TypeScript
  - Integración con APIs nativas

- **Limitaciones:**
  - Rendimiento limitado para juegos complejos
  - Dependencia del JavaScript Bridge
  - Memoria limitada en dispositivos móviles

#### Tipos de Juegos Apropiados para React Native
- **Juegos 2D simples:** Puzzles, plataformas, arcade
- **Juegos de cartas:** Solitario, poker, blackjack
- **Juegos de estrategia:** Tower defense, city builders
- **Juegos casuales:** Match-3, endless runners

### 2. Librerías y Herramientas para Juegos

#### React Native Game Engine
```bash
npm install react-native-game-engine
```

#### React Native Skia (Para gráficos avanzados)
```bash
npm install @shopify/react-native-skia
```

#### React Native Reanimated (Para animaciones)
```bash
npm install react-native-reanimated
```

#### React Native Gesture Handler
```bash
npm install react-native-gesture-handler
```

### 3. Estructura de un Juego en React Native

#### Componentes Principales
```jsx
// GameEngine.js
import React from 'react';
import { GameEngine } from 'react-native-game-engine';
import { Physics } from './systems/Physics';
import { Renderer } from './systems/Renderer';
import { Input } from './systems/Input';

const Game = () => {
  const entities = {
    player: {
      position: [100, 100],
      size: [50, 50],
      color: 'blue',
      velocity: [0, 0],
      renderer: 'player'
    },
    enemy: {
      position: [200, 200],
      size: [30, 30],
      color: 'red',
      velocity: [1, 0],
      renderer: 'enemy'
    }
  };

  const systems = [Physics, Renderer, Input];

  return (
    <GameEngine
      entities={entities}
      systems={systems}
      style={{ flex: 1 }}
    />
  );
};

export default Game;
```

#### Sistema de Física
```jsx
// systems/Physics.js
const Physics = (entities, { time }) => {
  Object.keys(entities).forEach(key => {
    const entity = entities[key];
    if (entity.velocity) {
      entity.position[0] += entity.velocity[0] * time.delta;
      entity.position[1] += entity.velocity[1] * time.delta;
    }
  });
  return entities;
};

export default Physics;
```

#### Sistema de Renderizado
```jsx
// systems/Renderer.js
import React from 'react';
import { View, StyleSheet } from 'react-native';

const Renderer = (entities, { screen }) => {
  return Object.keys(entities).map(key => {
    const entity = entities[key];
    if (entity.renderer) {
      return (
        <View
          key={key}
          style={[
            styles.entity,
            {
              left: entity.position[0],
              top: entity.position[1],
              width: entity.size[0],
              height: entity.size[1],
              backgroundColor: entity.color
            }
          ]}
        />
      );
    }
    return null;
  });
};

const styles = StyleSheet.create({
  entity: {
    position: 'absolute'
  }
});

export default Renderer;
```

### 4. Implementación de un Juego Básico

#### Juego de Plataformas Simple
```jsx
// PlatformGame.js
import React, { useState, useEffect } from 'react';
import { View, StyleSheet, Dimensions, TouchableOpacity } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  runOnJS
} from 'react-native-reanimated';

const { width, height } = Dimensions.get('window');

const PlatformGame = () => {
  const [score, setScore] = useState(0);
  const [gameOver, setGameOver] = useState(false);
  
  const playerY = useSharedValue(height - 100);
  const playerX = useSharedValue(width / 2);
  const jumpHeight = 150;
  const gravity = 5;

  const jump = () => {
    if (playerY.value === height - 100) {
      playerY.value = withSpring(playerY.value - jumpHeight);
      setTimeout(() => {
        playerY.value = withSpring(height - 100);
      }, 500);
    }
  };

  const playerStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: playerX.value },
      { translateY: playerY.value }
    ]
  }));

  return (
    <View style={styles.container}>
      <View style={styles.gameArea}>
        <Animated.View style={[styles.player, playerStyle]} />
        <View style={styles.platform} />
      </View>
      
      <TouchableOpacity style={styles.jumpButton} onPress={jump}>
        <Text style={styles.jumpText}>SALTAR</Text>
      </TouchableOpacity>
      
      <Text style={styles.score}>Puntuación: {score}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#87CEEB'
  },
  gameArea: {
    flex: 1,
    position: 'relative'
  },
  player: {
    width: 50,
    height: 50,
    backgroundColor: 'blue',
    borderRadius: 25,
    position: 'absolute'
  },
  platform: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    height: 50,
    backgroundColor: 'green'
  },
  jumpButton: {
    position: 'absolute',
    bottom: 100,
    right: 20,
    backgroundColor: 'orange',
    padding: 20,
    borderRadius: 10
  },
  jumpText: {
    color: 'white',
    fontWeight: 'bold'
  },
  score: {
    position: 'absolute',
    top: 50,
    left: 20,
    fontSize: 20,
    fontWeight: 'bold',
    color: 'white'
  }
});

export default PlatformGame;
```

### 5. Optimización de Rendimiento para Juegos

#### Técnicas de Optimización
- **Use Native Driver:** Para animaciones que no afectan el layout
- **Memoización:** Usar `React.memo` para componentes que no cambian
- **Lazy Loading:** Cargar assets solo cuando se necesiten
- **Object Pooling:** Reutilizar objetos para evitar garbage collection

#### Ejemplo de Optimización
```jsx
// OptimizedGameComponent.js
import React, { memo, useCallback } from 'react';
import { View, StyleSheet } from 'react-native';

const GameEntity = memo(({ position, size, color }) => {
  const style = useCallback(() => ({
    position: 'absolute',
    left: position[0],
    top: position[1],
    width: size[0],
    height: size[1],
    backgroundColor: color
  }), [position, size, color]);

  return <View style={style()} />;
});

export default GameEntity;
```

### 6. Integración con APIs Nativas

#### Vibración para Feedback
```jsx
import { Vibration } from 'react-native';

const triggerVibration = () => {
  Vibration.vibrate(100); // Vibra por 100ms
};
```

#### Sonidos y Música
```jsx
import Sound from 'react-native-sound';

const playSound = () => {
  const sound = new Sound('jump.mp3', Sound.MAIN_BUNDLE, (error) => {
    if (error) {
      console.log('Error loading sound:', error);
    } else {
      sound.play();
    }
  });
};
```

### 7. Consideraciones de Diseño

#### Responsive Design para Juegos
- Adaptar el tamaño de elementos según la pantalla
- Mantener proporciones consistentes
- Considerar diferentes orientaciones

#### Accesibilidad
- Controles táctiles claros
- Feedback visual y háptico
- Opciones de configuración

### 8. Próximos Pasos

#### Para la Próxima Clase
- Implementar colisiones
- Agregar enemigos y obstáculos
- Sistema de puntuación
- Menús y navegación

#### Proyecto Práctico
- Crear un juego completo
- Implementar diferentes niveles
- Agregar efectos de sonido
- Optimizar el rendimiento

## Recursos Adicionales

### Documentación
- [React Native Game Engine](https://github.com/bberak/react-native-game-engine)
- [React Native Skia](https://shopify.github.io/react-native-skia/)
- [React Native Reanimated](https://docs.swmansion.com/react-native-reanimated/)

### Tutoriales
- [Building a Game with React Native](https://blog.logrocket.com/building-game-react-native/)
- [React Native Game Development](https://reactnative.dev/docs/game-development)

### Herramientas
- [Flipper](https://fbflipper.com/) - Para debugging
- [React Native Performance](https://reactnative.dev/docs/performance)

## Ejercicios Prácticos

### Ejercicio 1: Juego Básico
Crear un juego simple donde el jugador evite obstáculos que caen desde arriba.

### Ejercicio 2: Optimización
Implementar técnicas de optimización en el juego creado.

### Ejercicio 3: Integración
Agregar vibración y sonidos al juego.

## Evaluación

### Criterios de Evaluación
- **Funcionalidad (40%):** El juego funciona correctamente
- **Rendimiento (30%):** Optimización implementada
- **Código (20%):** Estructura y calidad del código
- **Creatividad (10%):** Elementos únicos del juego

### Entregables
- Código fuente del juego
- Documentación de optimizaciones
- Demo del juego funcionando
- Reflexión sobre el proceso de desarrollo

---

**Duración estimada:** 2 horas  
**Dificultad:** Intermedia  
**Prerrequisitos:** Conocimientos sólidos de React Native, animaciones y hooks
