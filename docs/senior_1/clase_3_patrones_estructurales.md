# Clase 3: Patrones Estructurales

## Objetivos de la Clase
- Implementar el patrón Adapter para interfaces incompatibles
- Utilizar el patrón Decorator para funcionalidad dinámica
- Aplicar el patrón Facade para simplificar subsistemas complejos
- Crear estructuras jerárquicas con el patrón Composite

## Duración
**2 horas** (120 minutos)

## Contenido Teórico

### Patrón Adapter

El patrón Adapter permite que **interfaces incompatibles trabajen juntas**, actuando como un "traductor" entre dos sistemas.

#### Implementación Básica
```typescript
// ❌ Sin Adapter - Interfaces incompatibles
interface OldPaymentSystem {
  processPayment(amount: number, currency: string): boolean;
}

interface NewPaymentSystem {
  pay(amount: number, currency: string): Promise<PaymentResult>;
}

// Sistema antiguo
class LegacyPaymentSystem implements OldPaymentSystem {
  processPayment(amount: number, currency: string): boolean {
    console.log(`Processing ${amount} ${currency} with legacy system`);
    return Math.random() > 0.1; // 90% success rate
  }
}

// Sistema nuevo
class ModernPaymentSystem implements NewPaymentSystem {
  async pay(amount: number, currency: string): Promise<PaymentResult> {
    console.log(`Processing ${amount} ${currency} with modern system`);
    const success = Math.random() > 0.05; // 95% success rate
    
    return {
      success,
      transactionId: success ? `txn_${Date.now()}` : null,
      message: success ? 'Payment successful' : 'Payment failed'
    };
  }
}

// ✅ Con Adapter - Interfaz unificada
interface PaymentResult {
  success: boolean;
  transactionId: string | null;
  message: string;
}

// Adapter para el sistema antiguo
class LegacyPaymentAdapter implements NewPaymentSystem {
  constructor(private legacySystem: OldPaymentSystem) {}
  
  async pay(amount: number, currency: string): Promise<PaymentResult> {
    const success = this.legacySystem.processPayment(amount, currency);
    
    return {
      success,
      transactionId: success ? `legacy_${Date.now()}` : null,
      message: success ? 'Legacy payment successful' : 'Legacy payment failed'
    };
  }
}

// Uso del Adapter
const legacySystem = new LegacyPaymentSystem();
const modernSystem = new ModernPaymentSystem();
const legacyAdapter = new LegacyPaymentAdapter(legacySystem);

// Ahora ambos sistemas pueden usarse con la misma interfaz
const processPayments = async () => {
  const payments = [
    { system: modernSystem, amount: 100, currency: 'USD' },
    { system: legacyAdapter, amount: 50, currency: 'EUR' }
  ];
  
  for (const payment of payments) {
    const result = await payment.system.pay(payment.amount, payment.currency);
    console.log(`Payment result:`, result);
  }
};
```

#### Adapter en React Native
```typescript
// Adapter para diferentes sistemas de navegación
interface NavigationSystem {
  navigate(screen: string, params?: any): void;
  goBack(): void;
  push(screen: string, params?: any): void;
}

// React Navigation (sistema principal)
class ReactNavigationAdapter implements NavigationSystem {
  constructor(private navigation: any) {}
  
  navigate(screen: string, params?: any): void {
    this.navigation.navigate(screen, params);
  }
  
  goBack(): void {
    this.navigation.goBack();
  }
  
  push(screen: string, params?: any): void {
    this.navigation.push(screen, params);
  }
}

// Navegación nativa (sistema alternativo)
class NativeNavigationAdapter implements NavigationSystem {
  navigate(screen: string, params?: any): void {
    // Usar navegación nativa
    if (Platform.OS === 'ios') {
      // iOS navigation
    } else {
      // Android navigation
    }
  }
  
  goBack(): void {
    // Implementación nativa
  }
  
  push(screen: string, params?: any): void {
    // Implementación nativa
  }
}

// Hook para usar navegación unificada
const useUnifiedNavigation = () => {
  const navigation = useNavigation();
  
  return useMemo(() => {
    // Detectar qué sistema de navegación usar
    if (navigation) {
      return new ReactNavigationAdapter(navigation);
    } else {
      return new NativeNavigationAdapter();
    }
  }, [navigation]);
};

// Componente que usa navegación unificada
const NavigationButton = ({ screen, params, title }: {
  screen: string;
  params?: any;
  title: string;
}) => {
  const nav = useUnifiedNavigation();
  
  return (
    <TouchableOpacity onPress={() => nav.navigate(screen, params)}>
      <Text>{title}</Text>
    </TouchableOpacity>
  );
};
```

### Patrón Decorator

El patrón Decorator permite **añadir funcionalidad dinámicamente** a objetos sin alterar su estructura.

#### Implementación Básica
```typescript
// Interfaz base
interface Coffee {
  cost(): number;
  description(): string;
}

// Implementación básica
class SimpleCoffee implements Coffee {
  cost(): number {
    return 2.0;
  }
  
  description(): string {
    return 'Simple coffee';
  }
}

// Decorator base
abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}
  
  cost(): number {
    return this.coffee.cost();
  }
  
  description(): string {
    return this.coffee.description();
  }
}

// Decorators concretos
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.5;
  }
  
  description(): string {
    return this.coffee.description() + ', milk';
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.2;
  }
  
  description(): string {
    return this.coffee.description() + ', sugar';
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.7;
  }
  
  description(): string {
    return this.coffee.description() + ', whip';
  }
}

// Uso de decorators
const coffee = new SimpleCoffee();
console.log(`${coffee.description()}: $${coffee.cost()}`);

const coffeeWithMilk = new MilkDecorator(coffee);
console.log(`${coffeeWithMilk.description()}: $${coffeeWithMilk.cost()}`);

const coffeeWithMilkAndSugar = new SugarDecorator(coffeeWithMilk);
console.log(`${coffeeWithMilkAndSugar.description()}: $${coffeeWithMilkAndSugar.cost()}`);

const fancyCoffee = new WhipDecorator(new MilkDecorator(new SugarDecorator(coffee)));
console.log(`${fancyCoffee.description()}: $${fancyCoffee.cost()}`);
```

#### Decorator en React Native
```typescript
// Decorator para componentes con funcionalidad adicional
interface ComponentDecorator {
  (Component: React.ComponentType<any>): React.ComponentType<any>;
}

// Decorator para logging
const withLogging: ComponentDecorator = (Component) => {
  return (props) => {
    useEffect(() => {
      console.log(`Component ${Component.name} mounted with props:`, props);
      
      return () => {
        console.log(`Component ${Component.name} unmounted`);
      };
    }, []);
    
    return <Component {...props} />;
  };
};

// Decorator para analytics
const withAnalytics: ComponentDecorator = (Component) => {
  return (props) => {
    const trackEvent = useCallback((eventName: string, data?: any) => {
      // Implementar tracking de analytics
      console.log(`Analytics: ${eventName}`, data);
    }, []);
    
    return <Component {...props} trackEvent={trackEvent} />;
  };
};

// Decorator para error boundaries
const withErrorBoundary: ComponentDecorator = (Component) => {
  return (props) => {
    const [hasError, setHasError] = useState(false);
    
    if (hasError) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          <Text>Something went wrong</Text>
          <TouchableOpacity onPress={() => setHasError(false)}>
            <Text>Try again</Text>
          </TouchableOpacity>
        </View>
      );
    }
    
    return (
      <ErrorBoundary onError={() => setHasError(true)}>
        <Component {...props} />
      </ErrorBoundary>
    );
  };
};

// Componente base
const UserProfile = ({ user, trackEvent }: { user: User; trackEvent?: (event: string, data?: any) => void }) => {
  const handlePress = useCallback(() => {
    trackEvent?.('profile_viewed', { userId: user.id });
  }, [user.id, trackEvent]);
  
  return (
    <TouchableOpacity onPress={handlePress}>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
    </TouchableOpacity>
  );
};

// Aplicar decorators
const EnhancedUserProfile = withLogging(withAnalytics(withErrorBoundary(UserProfile)));

// Uso del componente decorado
const App = () => {
  const user = { id: '1', name: 'John Doe', email: 'john@example.com' };
  
  return <EnhancedUserProfile user={user} />;
};
```

### Patrón Facade

El patrón Facade proporciona una **interfaz simplificada** a un subsistema complejo, ocultando la complejidad interna.

#### Implementación Básica
```typescript
// Subsistemas complejos
class AudioSystem {
  play(): void {
    console.log('Audio system playing');
  }
  
  stop(): void {
    console.log('Audio system stopped');
  }
  
  setVolume(level: number): void {
    console.log(`Audio volume set to ${level}`);
  }
}

class VideoSystem {
  play(): void {
    console.log('Video system playing');
  }
  
  stop(): void {
    console.log('Video system stopped');
  }
  
  setBrightness(level: number): void {
    console.log(`Video brightness set to ${level}`);
  }
}

class LightingSystem {
  setBrightness(level: number): void {
    console.log(`Lighting brightness set to ${level}`);
  }
  
  setColor(color: string): void {
    console.log(`Lighting color set to ${color}`);
  }
}

class PopcornMachine {
  turnOn(): void {
    console.log('Popcorn machine turned on');
  }
  
  turnOff(): void {
    console.log('Popcorn machine turned off');
  }
  
  pop(): void {
    console.log('Popcorn popping...');
  }
}

// ✅ Facade que simplifica el subsistema
class HomeTheaterFacade {
  private audio: AudioSystem;
  private video: VideoSystem;
  private lighting: LightingSystem;
  private popcorn: PopcornMachine;
  
  constructor() {
    this.audio = new AudioSystem();
    this.video = new VideoSystem();
    this.lighting = new LightingSystem();
    this.popcorn = new PopcornMachine();
  }
  
  watchMovie(): void {
    console.log('🎬 Setting up movie theater...');
    
    this.popcorn.turnOn();
    this.popcorn.pop();
    
    this.lighting.setBrightness(10);
    this.lighting.setColor('dim');
    
    this.audio.setVolume(75);
    this.video.setBrightness(80);
    
    this.video.play();
    this.audio.play();
    
    console.log('🎬 Movie theater ready!');
  }
  
  endMovie(): void {
    console.log('🏁 Ending movie...');
    
    this.audio.stop();
    this.video.stop();
    
    this.lighting.setBrightness(100);
    this.lighting.setColor('warm');
    
    this.popcorn.turnOff();
    
    console.log('🏁 Movie ended, theater reset');
  }
  
  listenToMusic(): void {
    console.log('🎵 Setting up music mode...');
    
    this.lighting.setBrightness(50);
    this.lighting.setColor('blue');
    
    this.audio.setVolume(60);
    this.audio.play();
    
    console.log('🎵 Music mode ready!');
  }
}

// Uso del Facade
const homeTheater = new HomeTheaterFacade();

// Operaciones simples que ocultan la complejidad
homeTheater.watchMovie();
// ... después de la película
homeTheater.endMovie();

// Cambiar a modo música
homeTheater.listenToMusic();
```

#### Facade en React Native
```typescript
// Facade para gestión de estado de la aplicación
class AppStateFacade {
  private asyncStorage: AsyncStorage;
  private reduxStore: any;
  private navigation: any;
  
  constructor() {
    this.asyncStorage = AsyncStorage;
    this.reduxStore = null; // Se inicializa después
    this.navigation = null; // Se inicializa después
  }
  
  // Inicialización
  initialize(store: any, navigation: any): void {
    this.reduxStore = store;
    this.navigation = navigation;
  }
  
  // Gestión de usuario
  async login(credentials: { email: string; password: string }): Promise<boolean> {
    try {
      // Lógica compleja de login
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });
      
      if (response.ok) {
        const { token, user } = await response.json();
        
        // Guardar en AsyncStorage
        await this.asyncStorage.setItem('authToken', token);
        await this.asyncStorage.setItem('userData', JSON.stringify(user));
        
        // Actualizar Redux
        this.reduxStore.dispatch({ type: 'LOGIN_SUCCESS', payload: { user, token } });
        
        // Navegar a la pantalla principal
        this.navigation.navigate('Main');
        
        return true;
      }
      
      return false;
    } catch (error) {
      console.error('Login error:', error);
      return false;
    }
  }
  
  async logout(): Promise<void> {
    try {
      // Limpiar AsyncStorage
      await this.asyncStorage.removeItem('authToken');
      await this.asyncStorage.removeItem('userData');
      
      // Limpiar Redux
      this.reduxStore.dispatch({ type: 'LOGOUT' });
      
      // Navegar a login
      this.navigation.navigate('Login');
    } catch (error) {
      console.error('Logout error:', error);
    }
  }
  
  // Gestión de datos
  async saveData(key: string, data: any): Promise<void> {
    try {
      await this.asyncStorage.setItem(key, JSON.stringify(data));
      this.reduxStore.dispatch({ type: 'DATA_SAVED', payload: { key, data } });
    } catch (error) {
      console.error('Save data error:', error);
    }
  }
  
  async loadData(key: string): Promise<any> {
    try {
      const data = await this.asyncStorage.getItem(key);
      return data ? JSON.parse(data) : null;
    } catch (error) {
      console.error('Load data error:', error);
      return null;
    }
  }
  
  // Gestión de navegación
  navigateToScreen(screen: string, params?: any): void {
    this.navigation.navigate(screen, params);
  }
  
  goBack(): void {
    this.navigation.goBack();
  }
}

// Hook para usar el Facade
const useAppState = () => {
  const [facade] = useState(() => new AppStateFacade());
  
  useEffect(() => {
    // Inicializar cuando estén disponibles las dependencias
    const initializeFacade = async () => {
      const store = getReduxStore(); // Obtener store de Redux
      const navigation = useNavigation(); // Obtener navegación
      
      if (store && navigation) {
        facade.initialize(store, navigation);
      }
    };
    
    initializeFacade();
  }, [facade]);
  
  return facade;
};

// Componente que usa el Facade
const LoginScreen = () => {
  const appState = useAppState();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleLogin = async () => {
    const success = await appState.login({ email, password });
    
    if (!success) {
      Alert.alert('Error', 'Login failed');
    }
  };
  
  return (
    <View>
      <TextInput
        value={email}
        onChangeText={setEmail}
        placeholder="Email"
      />
      <TextInput
        value={password}
        onChangeText={setPassword}
        placeholder="Password"
        secureTextEntry
      />
      <TouchableOpacity onPress={handleLogin}>
        <Text>Login</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### Patrón Composite

El patrón Composite permite tratar **objetos individuales y composiciones de objetos de manera uniforme**.

#### Implementación Básica
```typescript
// Interfaz común para objetos individuales y composiciones
interface FileSystemItem {
  name: string;
  size: number;
  display(indent: string): void;
}

// Objeto individual (Leaf)
class File implements FileSystemItem {
  constructor(public name: string, public size: number) {}
  
  display(indent: string): void {
    console.log(`${indent}📄 ${this.name} (${this.size} bytes)`);
  }
}

// Composición de objetos (Composite)
class Directory implements FileSystemItem {
  private children: FileSystemItem[] = [];
  
  constructor(public name: string) {}
  
  add(item: FileSystemItem): void {
    this.children.push(item);
  }
  
  remove(item: FileSystemItem): void {
    const index = this.children.indexOf(item);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  get size(): number {
    return this.children.reduce((total, child) => total + child.size, 0);
  }
  
  display(indent: string): void {
    console.log(`${indent}📁 ${this.name} (${this.size} bytes)`);
    
    this.children.forEach(child => {
      child.display(indent + '  ');
    });
  }
}

// Uso del Composite
const createFileSystem = (): Directory => {
  const root = new Directory('root');
  
  const documents = new Directory('documents');
  documents.add(new File('report.pdf', 1024));
  documents.add(new File('presentation.pptx', 2048));
  
  const pictures = new Directory('pictures');
  pictures.add(new File('vacation.jpg', 512));
  pictures.add(new File('family.jpg', 768));
  
  const downloads = new Directory('downloads');
  downloads.add(new File('app.zip', 4096));
  
  root.add(documents);
  root.add(pictures);
  root.add(downloads);
  
  return root;
};

const fileSystem = createFileSystem();
fileSystem.display('');

// Operaciones uniformes en objetos individuales y composiciones
const printSize = (item: FileSystemItem): void => {
  console.log(`${item.name}: ${item.size} bytes`);
};

printSize(fileSystem); // root: 8448 bytes
printSize(new File('test.txt', 100)); // test.txt: 100 bytes
```

#### Composite en React Native
```typescript
// Composite para componentes de UI
interface UIComponent {
  render(): React.ReactElement;
  addChild?(child: UIComponent): void;
  removeChild?(child: UIComponent): void;
}

// Componente individual
class Button implements UIComponent {
  constructor(private title: string, private onPress: () => void) {}
  
  render(): React.ReactElement {
    return (
      <TouchableOpacity onPress={this.onPress}>
        <Text>{this.title}</Text>
      </TouchableOpacity>
    );
  }
}

// Componente compuesto
class Form implements UIComponent {
  private children: UIComponent[] = [];
  
  addChild(child: UIComponent): void {
    this.children.push(child);
  }
  
  removeChild(child: UIComponent): void {
    const index = this.children.indexOf(child);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  render(): React.ReactElement {
    return (
      <View style={styles.form}>
        {this.children.map((child, index) => (
          <View key={index} style={styles.formItem}>
            {child.render()}
          </View>
        ))}
      </View>
    );
  }
}

// Componente compuesto para layout
class Layout implements UIComponent {
  private children: UIComponent[] = [];
  private direction: 'row' | 'column';
  
  constructor(direction: 'row' | 'column' = 'column') {
    this.direction = direction;
  }
  
  addChild(child: UIComponent): void {
    this.children.push(child);
  }
  
  removeChild(child: UIComponent): void {
    const index = this.children.indexOf(child);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  render(): React.ReactElement {
    return (
      <View style={[
        styles.layout,
        { flexDirection: this.direction }
      ]}>
        {this.children.map((child, index) => (
          <View key={index}>
            {child.render()}
          </View>
        ))}
      </View>
    );
  }
}

// Uso del Composite
const createLoginForm = (): Form => {
  const form = new Form();
  
  const emailInput = new TextInput({ placeholder: 'Email' });
  const passwordInput = new TextInput({ placeholder: 'Password', secureTextEntry: true });
  const loginButton = new Button('Login', () => console.log('Login pressed'));
  
  const inputLayout = new Layout('column');
  inputLayout.addChild(emailInput);
  inputLayout.addChild(passwordInput);
  
  form.addChild(inputLayout);
  form.addChild(loginButton);
  
  return form;
};

// Componente que usa el Composite
const LoginFormComponent = () => {
  const form = useMemo(() => createLoginForm(), []);
  
  return form.render();
};
```

## Ejercicios Prácticos

### Ejercicio 1: Implementación de Adapter
Crea un Adapter para diferentes sistemas de almacenamiento:

```typescript
// Implementa StorageAdapter que unifique:
// - AsyncStorage (React Native)
// - LocalStorage (Web)
// - SecureStore (React Native)
// Todos deben implementar la misma interfaz
```

### Ejercicio 2: Decorator para Componentes
Crea decorators para funcionalidad común:

```typescript
// Implementa decorators para:
// - withLoading (mostrar spinner)
// - withRetry (reintentar operaciones)
// - withCaching (cachear datos)
// - withValidation (validar props)
```

### Ejercicio 3: Facade para API
Crea un Facade para simplificar llamadas a API:

```typescript
// Implementa ApiFacade que simplifique:
// - Autenticación automática
// - Manejo de errores
// - Cache de respuestas
// - Retry automático
// - Logging de requests
```

## Resumen de la Clase

### Conceptos Clave Aprendidos:
- ✅ **Adapter** permite que interfaces incompatibles trabajen juntas
- ✅ **Decorator** añade funcionalidad dinámicamente
- ✅ **Facade** simplifica subsistemas complejos
- ✅ **Composite** trata objetos individuales y composiciones uniformemente
- ✅ **Patrones estructurales** mejoran la organización del código

### Próximos Pasos:
- Implementar patrones de comportamiento en la siguiente clase
- Aplicar patrones estructurales en componentes React Native
- Practicar la refactorización de código existente

### Recursos Adicionales:
- [Structural Design Patterns](https://refactoring.guru/design-patterns/structural-patterns)
- [React Native Component Patterns](https://reactnative.dev/docs/components-and-apis)
- [TypeScript Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html)

---

## Navegación
- **Anterior**: [Clase 2: Patrones Creacionales](clase_2_patrones_creacionales.md)
- **Siguiente**: [Clase 4: Patrones de Comportamiento](clase_4_patrones_comportamiento.md)
- **README del Módulo**: [Módulo 8: Patrones de Diseño](README.md)
- **Inicio**: [Índice Completo](../../INDICE_COMPLETO.md)
