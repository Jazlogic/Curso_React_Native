# Clase 1: Fundamentos de Desarrollo de Juegos

## 🎯 Objetivos de la Clase

- Comprender los fundamentos del desarrollo de juegos en React Native
- Configurar motores de juegos y herramientas necesarias
- Implementar sistemas básicos de física y colisiones
- Crear animaciones complejas con Reanimated
- Desarrollar un juego simple funcional

## 📋 Contenido

### 1. Introducción al Desarrollo de Juegos

#### ¿Qué es un Motor de Juegos?
Un motor de juegos es un framework que proporciona las herramientas necesarias para crear videojuegos, incluyendo:

- **Sistema de renderizado** para gráficos
- **Motor de física** para colisiones y movimiento
- **Sistema de audio** para efectos sonoros
- **Gestión de entrada** para controles
- **Sistema de animación** para movimientos

#### Motores de Juegos para React Native

**React Native Game Engine**
```javascript
import { GameEngine } from 'react-native-game-engine';

const Game = () => {
  const entities = {
    player: {
      position: [50, 50],
      renderer: <Player />,
      physics: {
        velocity: [0, 0],
        acceleration: [0, 0]
      }
    }
  };

  return (
    <GameEngine
      systems={[Physics, Renderer]}
      entities={entities}
    />
  );
};
```

**Skia para Gráficos Avanzados**
```javascript
import { Canvas, Circle, Group } from '@shopify/react-native-skia';

const GameCanvas = () => {
  return (
    <Canvas style={{ flex: 1 }}>
      <Group>
        <Circle cx={100} cy={100} r={50} color="red" />
        <Circle cx={200} cy={150} r={30} color="blue" />
      </Group>
    </Canvas>
  );
};
```

### 2. Configuración del Entorno

#### Instalación de Dependencias
```bash
# React Native Game Engine
npm install react-native-game-engine

# Skia para gráficos
npm install @shopify/react-native-skia

# Reanimated para animaciones
npm install react-native-reanimated

# Física
npm install matter-js
```

#### Configuración de Metro
```javascript
// metro.config.js
const { getDefaultConfig } = require('metro-config');

module.exports = (async () => {
  const {
    resolver: { sourceExts, assetExts },
  } = await getDefaultConfig();
  
  return {
    transformer: {
      babelTransformerPath: require.resolve('react-native-svg-transformer'),
    },
    resolver: {
      assetExts: assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...sourceExts, 'svg'],
    },
  };
})();
```

### 3. Sistemas de Física

#### Implementación Básica de Física
```javascript
import Matter from 'matter-js';

const Physics = (entities, { time }) => {
  let engine = entities.physics.engine;
  
  Matter.Engine.update(engine, time.delta);
  
  return entities;
};

// Configuración del motor de física
const setupPhysics = () => {
  const engine = Matter.Engine.create();
  const world = engine.world;
  
  // Gravedad
  engine.world.gravity.y = 0.5;
  
  return { engine, world };
};
```

#### Colisiones y Detección
```javascript
const CollisionSystem = (entities, { events, dispatch }) => {
  events.forEach(event => {
    if (event.type === 'collision') {
      const { bodyA, bodyB } = event;
      
      // Lógica de colisión
      if (bodyA.label === 'player' && bodyB.label === 'enemy') {
        dispatch({ type: 'game-over' });
      }
    }
  });
  
  return entities;
};
```

### 4. Animaciones con Reanimated

#### Animaciones de Movimiento
```javascript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
} from 'react-native-reanimated';

const AnimatedPlayer = ({ position }) => {
  const translateX = useSharedValue(position.x);
  const translateY = useSharedValue(position.y);
  
  const animatedStyle = useAnimatedStyle(() => {
    return {
      transform: [
        { translateX: translateX.value },
        { translateY: translateY.value }
      ]
    };
  });
  
  const moveTo = (x, y) => {
    translateX.value = withSpring(x);
    translateY.value = withSpring(y);
  };
  
  return (
    <Animated.View style={[styles.player, animatedStyle]} />
  );
};
```

#### Efectos Visuales
```javascript
const ParticleEffect = ({ position }) => {
  const opacity = useSharedValue(1);
  const scale = useSharedValue(1);
  
  useEffect(() => {
    opacity.value = withTiming(0, { duration: 1000 });
    scale.value = withTiming(2, { duration: 1000 });
  }, []);
  
  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ scale: scale.value }]
  }));
  
  return (
    <Animated.View style={[styles.particle, animatedStyle]} />
  );
};
```

### 5. Sistema de Entidades

#### Definición de Entidades
```javascript
const createPlayer = (x, y) => ({
  position: [x, y],
  size: [50, 50],
  renderer: <Player />,
  physics: {
    body: Matter.Bodies.rectangle(x, y, 50, 50),
    velocity: [0, 0]
  },
  health: 100,
  score: 0
});

const createEnemy = (x, y) => ({
  position: [x, y],
  size: [40, 40],
  renderer: <Enemy />,
  physics: {
    body: Matter.Bodies.rectangle(x, y, 40, 40),
    velocity: [0, 1]
  },
  health: 50
});
```

#### Sistema de Renderizado
```javascript
const Renderer = (entities, { screen, time }) => {
  return Object.keys(entities).map(key => {
    const entity = entities[key];
    
    if (!entity.renderer) return null;
    
    return (
      <View
        key={key}
        style={{
          position: 'absolute',
          left: entity.position[0],
          top: entity.position[1],
          width: entity.size[0],
          height: entity.size[1]
        }}
      >
        {entity.renderer}
      </View>
    );
  });
};
```

### 6. Gestión de Estado del Juego

#### Game State Management
```javascript
const useGameState = () => {
  const [gameState, setGameState] = useState({
    score: 0,
    level: 1,
    lives: 3,
    isGameOver: false,
    isPaused: false
  });
  
  const updateScore = (points) => {
    setGameState(prev => ({
      ...prev,
      score: prev.score + points
    }));
  };
  
  const gameOver = () => {
    setGameState(prev => ({
      ...prev,
      isGameOver: true
    }));
  };
  
  return { gameState, updateScore, gameOver };
};
```

### 7. Proyecto Práctico: Juego de Plataformas

#### Estructura del Proyecto
```
src/
├── components/
│   ├── Player.js
│   ├── Enemy.js
│   ├── Platform.js
│   └── GameCanvas.js
├── systems/
│   ├── Physics.js
│   ├── Renderer.js
│   └── Collision.js
├── entities/
│   ├── createPlayer.js
│   ├── createEnemy.js
│   └── createPlatform.js
└── Game.js
```

#### Implementación del Juego
```javascript
import { GameEngine } from 'react-native-game-engine';
import { Physics, Renderer, Collision } from './systems';
import { createPlayer, createEnemy, createPlatform } from './entities';

const PlatformGame = () => {
  const [entities, setEntities] = useState({
    player: createPlayer(50, 300),
    enemy1: createEnemy(200, 250),
    enemy2: createEnemy(350, 200),
    platform1: createPlatform(0, 400, 400, 20),
    platform2: createPlatform(200, 300, 200, 20)
  });
  
  const onEvent = (e) => {
    if (e.type === 'game-over') {
      // Lógica de game over
    }
  };
  
  return (
    <GameEngine
      systems={[Physics, Renderer, Collision]}
      entities={entities}
      onEvent={onEvent}
    />
  );
};
```

## 🎮 Ejercicios Prácticos

### Ejercicio 1: Juego de Pelota
Crea un juego simple donde una pelota rebota en los bordes de la pantalla.

### Ejercicio 2: Sistema de Colisiones
Implementa detección de colisiones entre múltiples objetos.

### Ejercicio 3: Animaciones de Partículas
Crea efectos de partículas cuando ocurren colisiones.

## 📚 Recursos Adicionales

- [React Native Game Engine Documentation](https://github.com/bberak/react-native-game-engine)
- [Skia Documentation](https://shopify.github.io/react-native-skia/)
- [Matter.js Physics Engine](https://brm.io/matter-js/)
- [Game Development Patterns](https://gameprogrammingpatterns.com/)

## ✅ Checklist de la Clase

- [ ] Configurar el entorno de desarrollo de juegos
- [ ] Implementar sistema básico de física
- [ ] Crear animaciones con Reanimated
- [ ] Desarrollar sistema de colisiones
- [ ] Completar el juego de plataformas
- [ ] Entender los conceptos de entidades y sistemas

---

**Siguiente Clase**: Realidad Aumentada con ARKit y ARCore