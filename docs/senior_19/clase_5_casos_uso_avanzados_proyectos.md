# Clase 5: Casos de Uso Avanzados y Proyectos

## 🎯 Objetivos de la Clase

- Desarrollar juegos multiplayer en tiempo real
- Crear experiencias de compra en AR
- Implementar aplicaciones educativas con AR
- Integrar analytics para juegos
- Implementar estrategias de monetización

## 📋 Contenido

### 1. Juegos Multiplayer

#### Configuración de WebSocket
```javascript
import io from 'socket.io-client';

const MultiplayerGame = () => {
  const [socket, setSocket] = useState(null);
  const [players, setPlayers] = useState([]);
  const [gameState, setGameState] = useState({});
  
  useEffect(() => {
    const newSocket = io('ws://your-game-server.com');
    setSocket(newSocket);
    
    newSocket.on('connect', () => {
      console.log('Conectado al servidor');
    });
    
    newSocket.on('playerJoined', (player) => {
      setPlayers(prev => [...prev, player]);
    });
    
    newSocket.on('playerLeft', (playerId) => {
      setPlayers(prev => prev.filter(p => p.id !== playerId));
    });
    
    newSocket.on('gameStateUpdate', (state) => {
      setGameState(state);
    });
    
    return () => newSocket.close();
  }, []);
  
  const sendPlayerAction = (action) => {
    if (socket) {
      socket.emit('playerAction', action);
    }
  };
  
  return (
    <View style={styles.gameContainer}>
      {players.map(player => (
        <PlayerComponent
          key={player.id}
          player={player}
          onAction={sendPlayerAction}
        />
      ))}
    </View>
  );
};
```

### 2. AR Shopping Experiences

#### Visualización de Productos
```javascript
const ARProductViewer = () => {
  const [selectedProduct, setSelectedProduct] = useState(null);
  const [arObject, setArObject] = useState(null);
  
  const loadProductModel = async (product) => {
    try {
      const model = await load3DModel(product.modelUrl);
      setArObject(model);
      setSelectedProduct(product);
    } catch (error) {
      console.error('Error loading product model:', error);
    }
  };
  
  const placeProduct = (position) => {
    if (arObject && selectedProduct) {
      const placedProduct = {
        id: Date.now(),
        product: selectedProduct,
        position: position,
        scale: selectedProduct.scale || [1, 1, 1]
      };
      
      savePlacedProduct(placedProduct);
    }
  };
  
  return (
    <View style={styles.arContainer}>
      <ARScene onTap={placeProduct}>
        {arObject && (
          <Model3D
            model={arObject}
            position={[0, 0, -0.5]}
            scale={selectedProduct.scale}
          />
        )}
      </ARScene>
      
      <ProductCatalog
        products={products}
        onSelect={loadProductModel}
        selected={selectedProduct}
      />
    </View>
  );
};
```

### 3. Educational AR Apps

#### Anatomía Humana
```javascript
const AnatomyAR = () => {
  const [selectedSystem, setSelectedSystem] = useState('skeletal');
  const [currentLayer, setCurrentLayer] = useState(0);
  
  const anatomySystems = {
    skeletal: {
      layers: ['bones', 'joints', 'muscles'],
      models: ['skeleton.usdz', 'joints.usdz', 'muscles.usdz']
    },
    cardiovascular: {
      layers: ['heart', 'arteries', 'veins'],
      models: ['heart.usdz', 'arteries.usdz', 'veins.usdz']
    }
  };
  
  const currentSystem = anatomySystems[selectedSystem];
  
  return (
    <View style={styles.anatomyContainer}>
      <ARScene>
        {currentSystem.models.map((model, index) => (
          <Model3D
            key={index}
            model={model}
            position={[0, 0, -0.5]}
            opacity={index <= currentLayer ? 1 : 0.3}
          />
        ))}
      </ARScene>
      
      <SystemSelector
        systems={Object.keys(anatomySystems)}
        selected={selectedSystem}
        onSelect={setSelectedSystem}
      />
    </View>
  );
};
```

### 4. Gaming Analytics

#### Event Tracking
```javascript
import { Analytics } from 'react-native-analytics';

const GameAnalytics = () => {
  const [gameSession, setGameSession] = useState({
    startTime: Date.now(),
    level: 1,
    score: 0,
    deaths: 0
  });
  
  const trackEvent = (eventName, properties = {}) => {
    Analytics.track(eventName, {
      ...properties,
      sessionId: gameSession.id,
      timestamp: Date.now()
    });
  };
  
  const trackLevelComplete = (level, score, time) => {
    trackEvent('level_complete', {
      level: level,
      score: score,
      completionTime: time
    });
  };
  
  return { trackEvent, trackLevelComplete };
};
```

### 5. Monetization Strategies

#### In-App Purchases
```javascript
import { InAppPurchase } from 'react-native-iap';

const MonetizationSystem = () => {
  const [products, setProducts] = useState([]);
  
  useEffect(() => {
    const initializeIAP = async () => {
      try {
        await InAppPurchase.initConnection();
        
        const productIds = [
          'com.yourapp.coins_100',
          'com.yourapp.premium'
        ];
        
        const availableProducts = await InAppPurchase.getProducts(productIds);
        setProducts(availableProducts);
      } catch (error) {
        console.error('Error initializing IAP:', error);
      }
    };
    
    initializeIAP();
  }, []);
  
  const purchaseProduct = async (productId) => {
    try {
      const purchase = await InAppPurchase.requestPurchase(productId);
      
      if (purchase) {
        processPurchase(purchase);
      }
    } catch (error) {
      console.error('Error purchasing product:', error);
    }
  };
  
  return { products, purchaseProduct };
};
```

### 6. Proyecto Integrador: AR Gaming Experience

#### Implementación del Juego
```javascript
const ARGamingExperience = () => {
  const [gameState, setGameState] = useState('menu');
  const [player, setPlayer] = useState(null);
  const [enemies, setEnemies] = useState([]);
  const [score, setScore] = useState(0);
  
  const startGame = () => {
    setGameState('playing');
    initializeGame();
  };
  
  const initializeGame = () => {
    const newPlayer = createPlayer();
    setPlayer(newPlayer);
    
    const newEnemies = generateEnemies(5);
    setEnemies(newEnemies);
  };
  
  return (
    <View style={styles.gameContainer}>
      {gameState === 'menu' && (
        <MenuScreen onStart={startGame} />
      )}
      
      {gameState === 'playing' && (
        <GameScreen
          player={player}
          enemies={enemies}
          score={score}
        />
      )}
    </View>
  );
};
```

## 🎮 Ejercicios Prácticos

### Ejercicio 1: Juego Multiplayer
Implementa un juego simple con múltiples jugadores.

### Ejercicio 2: AR Shopping
Crea una experiencia de compra en realidad aumentada.

### Ejercicio 3: Educational AR
Desarrolla una aplicación educativa con AR.

## 📚 Recursos Adicionales

- [Socket.IO](https://socket.io/)
- [React Native IAP](https://github.com/dooboolab/react-native-iap)
- [AdMob](https://developers.google.com/admob)
- [Game Analytics](https://gameanalytics.com/)

## ✅ Checklist de la Clase

- [ ] Implementar juego multiplayer
- [ ] Crear experiencia de compra AR
- [ ] Desarrollar app educativa AR
- [ ] Integrar analytics de juegos
- [ ] Implementar monetización
- [ ] Completar el proyecto integrador

---

**¡Felicidades!** Has completado el Módulo 28: Gaming y Realidad Aumentada. Ahora tienes las habilidades para crear experiencias inmersivas de juegos y realidad aumentada con React Native.