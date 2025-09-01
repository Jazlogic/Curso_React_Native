# Clase 4: Patrones de Comportamiento

## Objetivos de la Clase
- Implementar el patrón Observer para comunicación entre componentes
- Utilizar el patrón Strategy para algoritmos intercambiables
- Aplicar el patrón Command para encapsular operaciones
- Crear máquinas de estado con el patrón State

## Duración
**2 horas** (120 minutos)

## Contenido Teórico

### Patrón Observer

El patrón Observer define una **dependencia uno-a-muchos** entre objetos, donde cuando un objeto cambia su estado, todos sus dependientes son notificados automáticamente.

#### Implementación Básica
```typescript
// Interfaz para observadores
interface Observer {
  update(data: any): void;
}

// Interfaz para sujetos observables
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(data: any): void;
}

// Sujeto observable (publisher)
class NewsAgency implements Subject {
  private observers: Observer[] = [];
  private news: string[] = [];
  
  attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      return console.log('Observer already attached');
    }
    
    console.log('Observer attached');
    this.observers.push(observer);
  }
  
  detach(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer);
    if (observerIndex === -1) {
      return console.log('Observer not found');
    }
    
    this.observers.splice(observerIndex, 1);
    console.log('Observer detached');
  }
  
  notify(data: any): void {
    console.log('Notifying observers...');
    this.observers.forEach(observer => observer.update(data));
  }
  
  // Método de negocio que dispara notificaciones
  addNews(news: string): void {
    this.news.push(news);
    this.notify({ type: 'NEWS_ADDED', content: news, timestamp: new Date() });
  }
  
  getNews(): string[] {
    return [...this.news];
  }
}

// Observadores concretos (subscribers)
class NewsChannel implements Observer {
  constructor(private name: string) {}
  
  update(data: any): void {
    if (data.type === 'NEWS_ADDED') {
      console.log(`[${this.name}] Breaking news: ${data.content}`);
      console.log(`[${this.name}] Time: ${data.timestamp}`);
    }
  }
}

class NewsWebsite implements Observer {
  constructor(private url: string) {}
  
  update(data: any): void {
    if (data.type === 'NEWS_ADDED') {
      console.log(`[${this.url}] New article published: ${data.content}`);
      // Aquí se podría actualizar la base de datos o enviar notificaciones
    }
  }
}

class NewsApp implements Observer {
  constructor(private appName: string) {}
  
  update(data: any): void {
    if (data.type === 'NEWS_ADDED') {
      console.log(`[${this.appName}] Push notification: ${data.content}`);
      // Aquí se podría enviar una notificación push
    }
  }
}

// Uso del patrón Observer
const newsAgency = new NewsAgency();

const cnn = new NewsChannel('CNN');
const bbc = new NewsChannel('BBC');
const website = new NewsWebsite('https://news.com');
const app = new NewsApp('NewsApp');

// Suscribir observadores
newsAgency.attach(cnn);
newsAgency.attach(bbc);
newsAgency.attach(website);
newsAgency.attach(app);

// Agregar noticias (dispara notificaciones automáticamente)
newsAgency.addNews('Breaking: New technology breakthrough!');
newsAgency.addNews('Sports: Team wins championship!');

// Desuscribir un observador
newsAgency.detach(cnn);

// Agregar más noticias (CNN no recibirá esta notificación)
newsAgency.addNews('Weather: Sunny day expected!');
```

#### Observer en React Native
```typescript
// Sistema de eventos personalizado para React Native
class EventEmitter {
  private events: Map<string, Function[]> = new Map();
  
  on(event: string, callback: Function): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(callback);
  }
  
  off(event: string, callback: Function): void {
    if (!this.events.has(event)) return;
    
    const callbacks = this.events.get(event)!;
    const index = callbacks.indexOf(callback);
    if (index > -1) {
      callbacks.splice(index, 1);
    }
  }
  
  emit(event: string, data?: any): void {
    if (!this.events.has(event)) return;
    
    this.events.get(event)!.forEach(callback => {
      try {
        callback(data);
      } catch (error) {
        console.error(`Error in event ${event}:`, error);
      }
    });
  }
  
  once(event: string, callback: Function): void {
    const onceCallback = (data: any) => {
      callback(data);
      this.off(event, onceCallback);
    };
    this.on(event, onceCallback);
  }
}

// Singleton para el sistema de eventos global
class GlobalEventBus extends EventEmitter {
  private static instance: GlobalEventBus | null = null;
  
  private constructor() {
    super();
  }
  
  static getInstance(): GlobalEventBus {
    if (!GlobalEventBus.instance) {
      GlobalEventBus.instance = new GlobalEventBus();
    }
    return GlobalEventBus.instance;
  }
}

// Hook personalizado para usar eventos
const useEventBus = () => {
  const eventBus = useMemo(() => GlobalEventBus.getInstance(), []);
  
  const subscribe = useCallback((event: string, callback: Function) => {
    eventBus.on(event, callback);
    
    // Cleanup automático cuando el componente se desmonta
    return () => eventBus.off(event, callback);
  }, [eventBus]);
  
  const emit = useCallback((event: string, data?: any) => {
    eventBus.emit(event, data);
  }, [eventBus]);
  
  return { subscribe, emit };
};

// Componente que emite eventos
const UserActions = () => {
  const { emit } = useEventBus();
  
  const handleLogin = useCallback(() => {
    emit('USER_LOGGED_IN', { userId: '123', timestamp: new Date() });
  }, [emit]);
  
  const handleLogout = useCallback(() => {
    emit('USER_LOGGED_OUT', { userId: '123', timestamp: new Date() });
  }, [emit]);
  
  const handleProfileUpdate = useCallback((profileData: any) => {
    emit('PROFILE_UPDATED', { userId: '123', data: profileData, timestamp: new Date() });
  }, [emit]);
  
  return (
    <View>
      <TouchableOpacity onPress={handleLogin}>
        <Text>Login</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={handleLogout}>
        <Text>Logout</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => handleProfileUpdate({ name: 'John' })}>
        <Text>Update Profile</Text>
      </TouchableOpacity>
    </View>
  );
};

// Componente que escucha eventos
const UserNotifications = () => {
  const { subscribe } = useEventBus();
  const [notifications, setNotifications] = useState<string[]>([]);
  
  useEffect(() => {
    const unsubscribeLogin = subscribe('USER_LOGGED_IN', (data: any) => {
      setNotifications(prev => [...prev, `User ${data.userId} logged in at ${data.timestamp}`]);
    });
    
    const unsubscribeLogout = subscribe('USER_LOGGED_OUT', (data: any) => {
      setNotifications(prev => [...prev, `User ${data.userId} logged out at ${data.timestamp}`]);
    });
    
    const unsubscribeProfile = subscribe('PROFILE_UPDATED', (data: any) => {
      setNotifications(prev => [...prev, `Profile updated for user ${data.userId}`]);
    });
    
    // Cleanup
    return () => {
      unsubscribeLogin();
      unsubscribeLogout();
      unsubscribeProfile();
    };
  }, [subscribe]);
  
  return (
    <View>
      <Text>Notifications:</Text>
      {notifications.map((notification, index) => (
        <Text key={index}>{notification}</Text>
      ))}
    </View>
  );
};
```

### Patrón Strategy

El patrón Strategy define una **familia de algoritmos**, encapsula cada uno y los hace intercambiables, permitiendo que el algoritmo varíe independientemente de los clientes que lo usan.

#### Implementación Básica
```typescript
// Interfaz para estrategias
interface PaymentStrategy {
  pay(amount: number): string;
}

// Estrategias concretas
class CreditCardPayment implements PaymentStrategy {
  constructor(private cardNumber: string, private cvv: string) {}
  
  pay(amount: number): string {
    console.log(`Processing $${amount} payment with credit card ending in ${this.cardNumber.slice(-4)}`);
    return `Credit card payment of $${amount} processed successfully`;
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}
  
  pay(amount: number): string {
    console.log(`Processing $${amount} payment with PayPal account ${this.email}`);
    return `PayPal payment of $${amount} processed successfully`;
  }
}

class BitcoinPayment implements PaymentStrategy {
  constructor(private walletAddress: string) {}
  
  pay(amount: number): string {
    console.log(`Processing $${amount} payment with Bitcoin wallet ${this.walletAddress}`);
    return `Bitcoin payment of $${amount} processed successfully`;
  }
}

class CashPayment implements PaymentStrategy {
  pay(amount: number): string {
    console.log(`Processing $${amount} payment with cash`);
    return `Cash payment of $${amount} received`;
  }
}

// Contexto que usa las estrategias
class ShoppingCart {
  private items: { name: string; price: number }[] = [];
  private paymentStrategy: PaymentStrategy | null = null;
  
  addItem(name: string, price: number): void {
    this.items.push({ name, price });
  }
  
  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.paymentStrategy = strategy;
  }
  
  checkout(): string {
    if (!this.paymentStrategy) {
      throw new Error('Payment strategy not set');
    }
    
    const total = this.items.reduce((sum, item) => sum + item.price, 0);
    const result = this.paymentStrategy.pay(total);
    
    // Limpiar carrito después del pago
    this.items = [];
    
    return result;
  }
  
  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
  
  getItems(): { name: string; price: number }[] {
    return [...this.items];
  }
}

// Uso del patrón Strategy
const cart = new ShoppingCart();

// Agregar items
cart.addItem('Laptop', 999);
cart.addItem('Mouse', 25);
cart.addItem('Keyboard', 75);

console.log('Total:', cart.getTotal()); // $1099

// Probar diferentes estrategias de pago
const creditCard = new CreditCardPayment('1234-5678-9012-3456', '123');
cart.setPaymentStrategy(creditCard);
console.log(cart.checkout());

// Agregar más items
cart.addItem('Monitor', 299);
cart.addItem('Headphones', 150);

const paypal = new PayPalPayment('user@example.com');
cart.setPaymentStrategy(paypal);
console.log(cart.checkout());

// Pago en efectivo
cart.addItem('USB Cable', 10);
cart.setPaymentStrategy(new CashPayment());
console.log(cart.checkout());
```

#### Strategy en React Native
```typescript
// Estrategias para diferentes tipos de navegación
interface NavigationStrategy {
  navigate(screen: string, params?: any): void;
  getBackButton(): React.ReactElement;
}

// Estrategia para navegación estándar
class StandardNavigationStrategy implements NavigationStrategy {
  constructor(private navigation: any) {}
  
  navigate(screen: string, params?: any): void {
    this.navigation.navigate(screen, params);
  }
  
  getBackButton(): React.ReactElement {
    return (
      <TouchableOpacity onPress={() => this.navigation.goBack()}>
        <Text>← Back</Text>
      </TouchableOpacity>
    );
  }
}

// Estrategia para navegación con tabs
class TabNavigationStrategy implements NavigationStrategy {
  constructor(private navigation: any) {}
  
  navigate(screen: string, params?: any): void {
    this.navigation.navigate(screen, params);
  }
  
  getBackButton(): React.ReactElement {
    return (
      <TouchableOpacity onPress={() => this.navigation.navigate('Home')}>
        <Text>🏠 Home</Text>
      </TouchableOpacity>
    );
  }
}

// Estrategia para navegación modal
class ModalNavigationStrategy implements NavigationStrategy {
  constructor(private navigation: any) {}
  
  navigate(screen: string, params?: any): void {
    this.navigation.navigate(screen, params);
  }
  
  getBackButton(): React.ReactElement {
    return (
      <TouchableOpacity onPress={() => this.navigation.goBack()}>
        <Text>✕ Close</Text>
      </TouchableOpacity>
    );
  }
}

// Contexto que usa las estrategias
class NavigationContext {
  private strategy: NavigationStrategy;
  
  constructor(strategy: NavigationStrategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy: NavigationStrategy): void {
    this.strategy = strategy;
  }
  
  navigate(screen: string, params?: any): void {
    this.strategy.navigate(screen, params);
  }
  
  getBackButton(): React.ReactElement {
    return this.strategy.getBackButton();
  }
}

// Hook para usar navegación con estrategias
const useNavigationStrategy = (type: 'standard' | 'tab' | 'modal') => {
  const navigation = useNavigation();
  
  const context = useMemo(() => {
    let strategy: NavigationStrategy;
    
    switch (type) {
      case 'standard':
        strategy = new StandardNavigationStrategy(navigation);
        break;
      case 'tab':
        strategy = new TabNavigationStrategy(navigation);
        break;
      case 'modal':
        strategy = new ModalNavigationStrategy(navigation);
        break;
      default:
        strategy = new StandardNavigationStrategy(navigation);
    }
    
    return new NavigationContext(strategy);
  }, [navigation, type]);
  
  return context;
};

// Componente que usa diferentes estrategias de navegación
const ScreenWithNavigation = ({ navigationType }: { navigationType: 'standard' | 'tab' | 'modal' }) => {
  const navContext = useNavigationStrategy(navigationType);
  
  return (
    <View>
      <Text>Screen Content</Text>
      {navContext.getBackButton()}
      
      <TouchableOpacity onPress={() => navContext.navigate('Profile')}>
        <Text>Go to Profile</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### Patrón Command

El patrón Command encapsula una **solicitud como un objeto**, permitiendo parametrizar clientes con diferentes solicitudes, hacer cola de operaciones y soportar operaciones deshacer.

#### Implementación Básica
```typescript
// Interfaz para comandos
interface Command {
  execute(): void;
  undo(): void;
}

// Comandos concretos
class LightOnCommand implements Command {
  constructor(private light: Light) {}
  
  execute(): void {
    this.light.turnOn();
  }
  
  undo(): void {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}
  
  execute(): void {
    this.light.turnOff();
  }
  
  undo(): void {
    this.light.turnOn();
  }
}

class LightDimCommand implements Command {
  constructor(private light: Light, private level: number) {}
  
  execute(): void {
    this.light.setBrightness(this.level);
  }
  
  undo(): void {
    this.light.setBrightness(100); // Volver al brillo original
  }
}

// Receptor
class Light {
  private isOn: boolean = false;
  private brightness: number = 100;
  
  turnOn(): void {
    this.isOn = true;
    console.log('Light is ON');
  }
  
  turnOff(): void {
    this.isOn = false;
    console.log('Light is OFF');
  }
  
  setBrightness(level: number): void {
    this.brightness = Math.max(0, Math.min(100, level));
    console.log(`Light brightness set to ${this.brightness}%`);
  }
  
  getStatus(): string {
    return `Light is ${this.isOn ? 'ON' : 'OFF'} at ${this.brightness}% brightness`;
  }
}

// Invocador (Remote Control)
class RemoteControl {
  private onCommands: Command[] = [];
  private offCommands: Command[] = [];
  private undoCommand: Command | null = null;
  
  setCommand(slot: number, onCommand: Command, offCommand: Command): void {
    this.onCommands[slot] = onCommand;
    this.offCommands[slot] = offCommand;
  }
  
  onButtonPressed(slot: number): void {
    if (this.onCommands[slot]) {
      this.onCommands[slot].execute();
      this.undoCommand = this.onCommands[slot];
    }
  }
  
  offButtonPressed(slot: number): void {
    if (this.offCommands[slot]) {
      this.offCommands[slot].execute();
      this.undoCommand = this.offCommands[slot];
    }
  }
  
  undoButtonPressed(): void {
    if (this.undoCommand) {
      this.undoCommand.undo();
      this.undoCommand = null;
    }
  }
}

// Uso del patrón Command
const livingRoomLight = new Light();
const kitchenLight = new Light();

const livingRoomLightOn = new LightOnCommand(livingRoomLight);
const livingRoomLightOff = new LightOffCommand(livingRoomLight);
const livingRoomLightDim = new LightDimCommand(livingRoomLight, 50);

const kitchenLightOn = new LightOnCommand(kitchenLight);
const kitchenLightOff = new LightOffCommand(kitchenLight);

const remote = new RemoteControl();

// Configurar comandos
remote.setCommand(0, livingRoomLightOn, livingRoomLightOff);
remote.setCommand(1, kitchenLightOn, kitchenLightOff);

// Usar el control remoto
console.log('--- Living Room Light Control ---');
remote.onButtonPressed(0);  // Encender luz de sala
remote.undoButtonPressed(); // Deshacer (apagar)

console.log('--- Kitchen Light Control ---');
remote.onButtonPressed(1);  // Encender luz de cocina

console.log('--- Dimming Living Room Light ---');
livingRoomLightDim.execute(); // Ajustar brillo
livingRoomLightDim.undo();   // Deshacer (brillo completo)

console.log('--- Status ---');
console.log(livingRoomLight.getStatus());
console.log(kitchenLight.getStatus());
```

#### Command en React Native
```typescript
// Comandos para acciones de usuario
interface UserActionCommand {
  execute(): Promise<void>;
  undo(): Promise<void>;
  canUndo(): boolean;
}

// Comando para agregar item al carrito
class AddToCartCommand implements UserActionCommand {
  constructor(
    private item: { id: string; name: string; price: number },
    private cartService: CartService
  ) {}
  
  async execute(): Promise<void> {
    await this.cartService.addItem(this.item);
  }
  
  async undo(): Promise<void> {
    await this.cartService.removeItem(this.item.id);
  }
  
  canUndo(): boolean {
    return true;
  }
}

// Comando para actualizar perfil
class UpdateProfileCommand implements UserActionCommand {
  private previousData: any;
  
  constructor(
    private userId: string,
    private newData: any,
    private profileService: ProfileService
  ) {}
  
  async execute(): Promise<void> {
    // Guardar datos anteriores para poder deshacer
    this.previousData = await this.profileService.getProfile(this.userId);
    await this.profileService.updateProfile(this.userId, this.newData);
  }
  
  async undo(): Promise<void> {
    if (this.previousData) {
      await this.profileService.updateProfile(this.userId, this.previousData);
    }
  }
  
  canUndo(): boolean {
    return !!this.previousData;
  }
}

// Comando para enviar mensaje
class SendMessageCommand implements UserActionCommand {
  constructor(
    private message: { text: string; recipientId: string },
    private messageService: MessageService
  ) {}
  
  async execute(): Promise<void> {
    await this.messageService.sendMessage(this.message);
  }
  
  async undo(): Promise<void> {
    // No se puede deshacer el envío de un mensaje
    throw new Error('Cannot undo sent message');
  }
  
  canUndo(): boolean {
    return false;
  }
}

// Invocador que maneja comandos
class CommandInvoker {
  private commandHistory: UserActionCommand[] = [];
  private maxHistorySize: number = 10;
  
  async executeCommand(command: UserActionCommand): Promise<void> {
    try {
      await command.execute();
      
      // Solo agregar a la historia si se puede deshacer
      if (command.canUndo()) {
        this.commandHistory.push(command);
        
        // Mantener tamaño máximo de historia
        if (this.commandHistory.length > this.maxHistorySize) {
          this.commandHistory.shift();
        }
      }
    } catch (error) {
      console.error('Command execution failed:', error);
      throw error;
    }
  }
  
  async undoLastCommand(): Promise<void> {
    if (this.commandHistory.length === 0) {
      throw new Error('No commands to undo');
    }
    
    const lastCommand = this.commandHistory.pop()!;
    await lastCommand.undo();
  }
  
  canUndo(): boolean {
    return this.commandHistory.length > 0;
  }
  
  getHistorySize(): number {
    return this.commandHistory.length;
  }
}

// Hook para usar comandos
const useCommandInvoker = () => {
  const [invoker] = useState(() => new CommandInvoker());
  
  const executeCommand = useCallback(async (command: UserActionCommand) => {
    await invoker.executeCommand(command);
  }, [invoker]);
  
  const undoLastCommand = useCallback(async () => {
    await invoker.undoLastCommand();
  }, [invoker]);
  
  const canUndo = useCallback(() => {
    return invoker.canUndo();
  }, [invoker]);
  
  return { executeCommand, undoLastCommand, canUndo };
};

// Componente que usa comandos
const UserActionsWithCommands = () => {
  const { executeCommand, undoLastCommand, canUndo } = useCommandInvoker();
  const [loading, setLoading] = useState(false);
  
  const handleAddToCart = useCallback(async (item: any) => {
    setLoading(true);
    try {
      const command = new AddToCartCommand(item, cartService);
      await executeCommand(command);
      Alert.alert('Success', 'Item added to cart');
    } catch (error) {
      Alert.alert('Error', 'Failed to add item to cart');
    } finally {
      setLoading(false);
    }
  }, [executeCommand]);
  
  const handleUndo = useCallback(async () => {
    try {
      await undoLastCommand();
      Alert.alert('Success', 'Action undone');
    } catch (error) {
      Alert.alert('Error', 'Failed to undo action');
    }
  }, [undoLastCommand]);
  
  return (
    <View>
      <TouchableOpacity 
        onPress={() => handleAddToCart({ id: '1', name: 'Product', price: 99 })}
        disabled={loading}
      >
        <Text>{loading ? 'Adding...' : 'Add to Cart'}</Text>
      </TouchableOpacity>
      
      {canUndo() && (
        <TouchableOpacity onPress={handleUndo}>
          <Text>Undo Last Action</Text>
        </TouchableOpacity>
      )}
    </View>
  );
};
```

### Patrón State

El patrón State permite que un objeto **cambie su comportamiento cuando cambia su estado interno**, pareciendo que el objeto cambia de clase.

#### Implementación Básica
```typescript
// Interfaz para estados
interface VendingMachineState {
  insertCoin(): void;
  ejectCoin(): void;
  turnCrank(): void;
  dispense(): void;
}

// Estado: Sin moneda
class NoCoinState implements VendingMachineState {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log('Coin inserted');
    this.machine.setState(this.machine.getHasCoinState());
  }
  
  ejectCoin(): void {
    console.log('No coin to eject');
  }
  
  turnCrank(): void {
    console.log('Please insert a coin first');
  }
  
  dispense(): void {
    console.log('Please insert a coin first');
  }
}

// Estado: Con moneda
class HasCoinState implements VendingMachineState {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log('Coin already inserted');
  }
  
  ejectCoin(): void {
    console.log('Coin ejected');
    this.machine.setState(this.machine.getNoCoinState());
  }
  
  turnCrank(): void {
    console.log('Crank turned, dispensing...');
    this.machine.setState(this.machine.getSoldState());
  }
  
  dispense(): void {
    console.log('Please turn the crank');
  }
}

// Estado: Vendiendo
class SoldState implements VendingMachineState {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log('Please wait, dispensing...');
  }
  
  ejectCoin(): void {
    console.log('Sorry, already turned the crank');
  }
  
  turnCrank(): void {
    console.log('Turning twice doesn\'t get you another gumball');
  }
  
  dispense(): void {
    this.machine.releaseBall();
    
    if (this.machine.getCount() > 0) {
      this.machine.setState(this.machine.getNoCoinState());
    } else {
      console.log('Out of gumballs');
      this.machine.setState(this.machine.getSoldOutState());
    }
  }
}

// Estado: Sin stock
class SoldOutState implements VendingMachineState {
  constructor(private machine: VendingMachine) {}
  
  insertCoin(): void {
    console.log('Machine is sold out');
  }
  
  ejectCoin(): void {
    console.log('No coin to eject');
  }
  
  turnCrank(): void {
    console.log('Machine is sold out');
  }
  
  dispense(): void {
    console.log('Machine is sold out');
  }
}

// Contexto principal
class VendingMachine {
  private noCoinState: VendingMachineState;
  private hasCoinState: VendingMachineState;
  private soldState: VendingMachineState;
  private soldOutState: VendingMachineState;
  
  private currentState: VendingMachineState;
  private count: number = 0;
  
  constructor(numberGumballs: number) {
    this.noCoinState = new NoCoinState(this);
    this.hasCoinState = new HasCoinState(this);
    this.soldState = new SoldState(this);
    this.soldOutState = new SoldOutState(this);
    
    this.count = numberGumballs;
    
    if (numberGumballs > 0) {
      this.currentState = this.noCoinState;
    } else {
      this.currentState = this.soldOutState;
    }
  }
  
  insertCoin(): void {
    this.currentState.insertCoin();
  }
  
  ejectCoin(): void {
    this.currentState.ejectCoin();
  }
  
  turnCrank(): void {
    this.currentState.turnCrank();
  }
  
  dispense(): void {
    this.currentState.dispense();
  }
  
  setState(state: VendingMachineState): void {
    this.currentState = state;
  }
  
  releaseBall(): void {
    console.log('A gumball comes rolling out the slot...');
    if (this.count !== 0) {
      this.count--;
    }
  }
  
  // Getters para estados
  getNoCoinState(): VendingMachineState { return this.noCoinState; }
  getHasCoinState(): VendingMachineState { return this.hasCoinState; }
  getSoldState(): VendingMachineState { return this.soldState; }
  getSoldOutState(): VendingMachineState { return this.soldOutState; }
  
  getCount(): number { return this.count; }
  
  getCurrentState(): string {
    if (this.currentState === this.noCoinState) return 'No Coin';
    if (this.currentState === this.hasCoinState) return 'Has Coin';
    if (this.currentState === this.soldState) return 'Sold';
    if (this.currentState === this.soldOutState) return 'Sold Out';
    return 'Unknown';
  }
}

// Uso del patrón State
const machine = new VendingMachine(5);

console.log('Machine state:', machine.getCurrentState());

machine.insertCoin();
console.log('Machine state:', machine.getCurrentState());

machine.turnCrank();
console.log('Machine state:', machine.getCurrentState());

machine.insertCoin();
machine.turnCrank();
machine.insertCoin();
machine.turnCrank();
machine.insertCoin();
machine.turnCrank();
machine.insertCoin();
machine.turnCrank();

console.log('Machine state:', machine.getCurrentState());
console.log('Gumballs remaining:', machine.getCount());
```

#### State en React Native
```typescript
// Estados para un formulario de login
interface FormState {
  handleInputChange(field: string, value: string): void;
  handleSubmit(): void;
  handleReset(): void;
  render(): React.ReactElement;
}

// Estado: Formulario vacío
class EmptyFormState implements FormState {
  constructor(private form: LoginForm) {}
  
  handleInputChange(field: string, value: string): void {
    this.form.updateField(field, value);
    
    // Cambiar a estado de llenado si hay algún valor
    if (value.trim()) {
      this.form.setState(new FillingFormState(this.form));
    }
  }
  
  handleSubmit(): void {
    // No hacer nada en estado vacío
  }
  
  handleReset(): void {
    // No hacer nada en estado vacío
  }
  
  render(): React.ReactElement {
    return (
      <View>
        <Text>Please fill in the form</Text>
        <TextInput
          placeholder="Email"
          onChangeText={(value) => this.handleInputChange('email', value)}
        />
        <TextInput
          placeholder="Password"
          secureTextEntry
          onChangeText={(value) => this.handleInputChange('password', value)}
        />
        <TouchableOpacity disabled>
          <Text>Submit (disabled)</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

// Estado: Formulario siendo llenado
class FillingFormState implements FormState {
  constructor(private form: LoginForm) {}
  
  handleInputChange(field: string, value: string): void {
    this.form.updateField(field, value);
    
    // Cambiar a estado válido si todos los campos están llenos
    if (this.form.isFormValid()) {
      this.form.setState(new ValidFormState(this.form));
    }
    
    // Volver a estado vacío si se borran todos los campos
    if (this.form.isFormEmpty()) {
      this.form.setState(new EmptyFormState(this.form));
    }
  }
  
  handleSubmit(): void {
    // No hacer nada hasta que el formulario sea válido
  }
  
  handleReset(): void {
    this.form.resetForm();
    this.form.setState(new EmptyFormState(this.form));
  }
  
  render(): React.ReactElement {
    return (
      <View>
        <Text>Filling form...</Text>
        <TextInput
          placeholder="Email"
          value={this.form.getFieldValue('email')}
          onChangeText={(value) => this.handleInputChange('email', value)}
        />
        <TextInput
          placeholder="Password"
          value={this.form.getFieldValue('password')}
          secureTextEntry
          onChangeText={(value) => this.handleInputChange('password', value)}
        />
        <TouchableOpacity disabled>
          <Text>Submit (disabled)</Text>
        </TouchableOpacity>
        <TouchableOpacity onPress={() => this.handleReset()}>
          <Text>Reset</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

// Estado: Formulario válido
class ValidFormState implements FormState {
  constructor(private form: LoginForm) {}
  
  handleInputChange(field: string, value: string): void {
    this.form.updateField(field, value);
    
    // Cambiar a estado de llenado si se borra algún campo
    if (!this.form.isFormValid()) {
      this.form.setState(new FillingFormState(this.form));
    }
  }
  
  handleSubmit(): void {
    this.form.setState(new SubmittingFormState(this.form));
    this.form.submitForm();
  }
  
  handleReset(): void {
    this.form.resetForm();
    this.form.setState(new EmptyFormState(this.form));
  }
  
  render(): React.ReactElement {
    return (
      <View>
        <Text>Form is ready!</Text>
        <TextInput
          placeholder="Email"
          value={this.form.getFieldValue('email')}
          onChangeText={(value) => this.handleInputChange('email', value)}
        />
        <TextInput
          placeholder="Password"
          value={this.form.getFieldValue('password')}
          secureTextEntry
          onChangeText={(value) => this.handleInputChange('password', value)}
        />
        <TouchableOpacity onPress={() => this.handleSubmit()}>
          <Text>Submit</Text>
        </TouchableOpacity>
        <TouchableOpacity onPress={() => this.handleReset()}>
          <Text>Reset</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

// Estado: Enviando formulario
class SubmittingFormState implements FormState {
  constructor(private form: LoginForm) {}
  
  handleInputChange(field: string, value: string): void {
    // No permitir cambios durante el envío
  }
  
  handleSubmit(): void {
    // No permitir múltiples envíos
  }
  
  handleReset(): void {
    // No permitir reset durante el envío
  }
  
  render(): React.ReactElement {
    return (
      <View>
        <Text>Submitting...</Text>
        <ActivityIndicator size="large" />
        <TextInput
          placeholder="Email"
          value={this.form.getFieldValue('email')}
          editable={false}
        />
        <TextInput
          placeholder="Password"
          value={this.form.getFieldValue('password')}
          secureTextEntry
          editable={false}
        />
        <TouchableOpacity disabled>
          <Text>Submitting...</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

// Contexto principal del formulario
class LoginForm {
  private currentState: FormState;
  private fields: Map<string, string> = new Map();
  
  constructor() {
    this.currentState = new EmptyFormState(this);
  }
  
  setState(state: FormState): void {
    this.currentState = state;
  }
  
  updateField(field: string, value: string): void {
    this.fields.set(field, value);
  }
  
  getFieldValue(field: string): string {
    return this.fields.get(field) || '';
  }
  
  isFormValid(): boolean {
    const email = this.fields.get('email') || '';
    const password = this.fields.get('password') || '';
    
    return email.trim().length > 0 && password.trim().length > 0;
  }
  
  isFormEmpty(): boolean {
    return Array.from(this.fields.values()).every(value => !value.trim());
  }
  
  resetForm(): void {
    this.fields.clear();
  }
  
  async submitForm(): Promise<void> {
    try {
      // Simular envío
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      // Cambiar a estado de éxito o error según el resultado
      if (Math.random() > 0.5) {
        this.setState(new SuccessFormState(this));
      } else {
        this.setState(new ErrorFormState(this));
      }
    } catch (error) {
      this.setState(new ErrorFormState(this));
    }
  }
  
  // Métodos delegados al estado actual
  handleInputChange(field: string, value: string): void {
    this.currentState.handleInputChange(field, value);
  }
  
  handleSubmit(): void {
    this.currentState.handleSubmit();
  }
  
  handleReset(): void {
    this.currentState.handleReset();
  }
  
  render(): React.ReactElement {
    return this.currentState.render();
  }
}

// Hook para usar el formulario con estados
const useLoginForm = () => {
  const [form] = useState(() => new LoginForm());
  
  const handleInputChange = useCallback((field: string, value: string) => {
    form.handleInputChange(field, value);
  }, [form]);
  
  const handleSubmit = useCallback(() => {
    form.handleSubmit();
  }, [form]);
  
  const handleReset = useCallback(() => {
    form.handleReset();
  }, [form]);
  
  return { form, handleInputChange, handleSubmit, handleReset };
};

// Componente que usa el formulario con estados
const LoginFormWithStates = () => {
  const { form, handleInputChange, handleSubmit, handleReset } = useLoginForm();
  
  // Forzar re-render cuando cambie el estado
  const [, forceUpdate] = useReducer(x => x + 1, 0);
  
  useEffect(() => {
    // Re-renderizar cuando cambie el estado
    const interval = setInterval(forceUpdate, 100);
    return () => clearInterval(interval);
  }, []);
  
  return (
    <View>
      {form.render()}
    </View>
  );
};
```

## Ejercicios Prácticos

### Ejercicio 1: Implementación de Observer
Crea un sistema de notificaciones push con Observer:

```typescript
// Implementa NotificationSystem con:
// - Diferentes tipos de notificaciones (email, push, SMS)
// - Suscripción/desuscripción de usuarios
// - Notificación automática cuando cambie el estado
// - Hook personalizado para usar el sistema
```

### Ejercicio 2: Strategy para Validación
Crea estrategias de validación para formularios:

```typescript
// Implementa ValidationStrategy para:
// - Validación de email
// - Validación de contraseña
// - Validación de teléfono
// - Validación de fecha
// - Hook para usar diferentes estrategias
```

### Ejercicio 3: Command para Historial
Crea un sistema de comandos con historial:

```typescript
// Implementa CommandHistory con:
// - Ejecución de comandos
- Deshacer/rehacer operaciones
// - Límite de historial
// - Comandos compuestos
// - Persistencia de historial
```

## Resumen de la Clase

### Conceptos Clave Aprendidos:
- ✅ **Observer** permite comunicación desacoplada entre objetos
- ✅ **Strategy** encapsula algoritmos intercambiables
- ✅ **Command** encapsula operaciones como objetos
- ✅ **State** cambia comportamiento según estado interno
- ✅ **Patrones de comportamiento** mejoran la flexibilidad del código

### Próximos Pasos:
- Implementar arquitectura y patrones compuestos en la siguiente clase
- Aplicar patrones de comportamiento en componentes React Native
- Practicar la refactorización de código existente

### Recursos Adicionales:
- [Behavioral Design Patterns](https://refactoring.guru/design-patterns/behavioral-patterns)
- [React Native State Management](https://reactnative.dev/docs/state)
- [TypeScript Design Patterns](https://www.typescriptlang.org/docs/)

---

## Navegación
- **Anterior**: [Clase 3: Patrones Estructurales](clase_3_patrones_estructurales.md)
- **Siguiente**: [Clase 5: Arquitectura y Patrones Compuestos](clase_5_arquitectura_patrones_compuestos.md)
- **README del Módulo**: [Módulo 8: Patrones de Diseño](README.md)
- **Inicio**: [Índice Completo](../../INDICE_COMPLETO.md)
