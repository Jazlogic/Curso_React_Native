# Clase 5: Arquitectura y Patrones Compuestos

## Objetivos de la Clase
- Implementar arquitecturas limpias (Clean Architecture)
- Crear patrones compuestos combinando múltiples patrones
- Aplicar principios de arquitectura en React Native
- Refactorizar aplicaciones existentes con patrones

## Duración
**1.5 horas** (90 minutos)

## Contenido Teórico

### Clean Architecture

Clean Architecture es un patrón arquitectónico que **separa las preocupaciones** en capas, haciendo el código más mantenible, testeable e independiente de frameworks.

#### Principios de Clean Architecture

```typescript
// ✅ Estructura de Clean Architecture
// src/
// ├── domain/           # Reglas de negocio (entidades, casos de uso)
// ├── data/            # Implementación de repositorios y fuentes de datos
// ├── presentation/    # UI y controladores
// └── infrastructure/  # Frameworks, bases de datos, APIs externas

// 1. DOMAIN LAYER - Reglas de negocio puras
interface User {
  id: string;
  email: string;
  name: string;
  isActive: boolean;
}

interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// Caso de uso de negocio
class CreateUserUseCase {
  constructor(private userRepository: UserRepository) {}
  
  async execute(userData: Omit<User, 'id'>): Promise<User> {
    // Validaciones de negocio
    if (!userData.email.includes('@')) {
      throw new Error('Invalid email format');
    }
    
    if (userData.name.trim().length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    
    // Crear usuario con ID único
    const user: User = {
      ...userData,
      id: this.generateId(),
      isActive: true
    };
    
    // Guardar en repositorio
    await this.userRepository.save(user);
    
    return user;
  }
  
  private generateId(): string {
    return `user_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// 2. DATA LAYER - Implementación de repositorios
class ApiUserRepository implements UserRepository {
  constructor(private apiClient: ApiClient) {}
  
  async findById(id: string): Promise<User | null> {
    try {
      const response = await this.apiClient.get(`/users/${id}`);
      return response.data;
    } catch (error) {
      if (error.status === 404) {
        return null;
      }
      throw error;
    }
  }
  
  async save(user: User): Promise<void> {
    if (user.id) {
      await this.apiClient.put(`/users/${user.id}`, user);
    } else {
      await this.apiClient.post('/users', user);
    }
  }
  
  async delete(id: string): Promise<void> {
    await this.apiClient.delete(`/users/${id}`);
  }
}

// 3. PRESENTATION LAYER - UI y controladores
class UserController {
  constructor(private createUserUseCase: CreateUserUseCase) {}
  
  async createUser(userData: any): Promise<{ success: boolean; user?: User; error?: string }> {
    try {
      const user = await this.createUserUseCase.execute(userData);
      return { success: true, user };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }
}

// 4. INFRASTRUCTURE LAYER - Implementaciones concretas
class ApiClient {
  constructor(private baseUrl: string) {}
  
  async get(endpoint: string): Promise<any> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
  }
  
  async post(endpoint: string, data: any): Promise<any> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
  }
  
  async put(endpoint: string, data: any): Promise<any> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
  }
  
  async delete(endpoint: string): Promise<void> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'DELETE'
    });
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
  }
}

// 5. COMPOSICIÓN Y DEPENDENCIAS
class AppContainer {
  private apiClient: ApiClient;
  private userRepository: UserRepository;
  private createUserUseCase: CreateUserUseCase;
  private userController: UserController;
  
  constructor() {
    // Inicializar dependencias de abajo hacia arriba
    this.apiClient = new ApiClient('https://api.example.com');
    this.userRepository = new ApiUserRepository(this.apiClient);
    this.createUserUseCase = new CreateUserUseCase(this.userRepository);
    this.userController = new UserController(this.createUserUseCase);
  }
  
  getUserController(): UserController {
    return this.userController;
  }
}
```

#### Clean Architecture en React Native
```typescript
// Implementación de Clean Architecture en React Native

// DOMAIN LAYER
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  category: string;
}

interface ProductRepository {
  getAll(): Promise<Product[]>;
  getById(id: string): Promise<Product | null>;
  search(query: string): Promise<Product[]>;
  getByCategory(category: string): Promise<Product[]>;
}

class GetProductsUseCase {
  constructor(private productRepository: ProductRepository) {}
  
  async execute(filters?: { category?: string; query?: string }): Promise<Product[]> {
    if (filters?.category) {
      return this.productRepository.getByCategory(filters.category);
    }
    
    if (filters?.query) {
      return this.productRepository.search(filters.query);
    }
    
    return this.productRepository.getAll();
  }
}

// DATA LAYER
class ApiProductRepository implements ProductRepository {
  constructor(private apiClient: ApiClient) {}
  
  async getAll(): Promise<Product[]> {
    const response = await this.apiClient.get('/products');
    return response.data;
  }
  
  async getById(id: string): Promise<Product | null> {
    try {
      const response = await this.apiClient.get(`/products/${id}`);
      return response.data;
    } catch (error) {
      if (error.status === 404) return null;
      throw error;
    }
  }
  
  async search(query: string): Promise<Product[]> {
    const response = await this.apiClient.get(`/products/search?q=${encodeURIComponent(query)}`);
    return response.data;
  }
  
  async getByCategory(category: string): Promise<Product[]> {
    const response = await this.apiClient.get(`/products/category/${encodeURIComponent(category)}`);
    return response.data;
  }
}

// PRESENTATION LAYER
const useProducts = (filters?: { category?: string; query?: string }) => {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const getProductsUseCase = useMemo(() => {
    const apiClient = new ApiClient('https://api.example.com');
    const productRepository = new ApiProductRepository(apiClient);
    return new GetProductsUseCase(productRepository);
  }, []);
  
  const fetchProducts = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await getProductsUseCase.execute(filters);
      setProducts(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [getProductsUseCase, filters]);
  
  useEffect(() => {
    fetchProducts();
  }, [fetchProducts]);
  
  return { products, loading, error, refetch: fetchProducts };
};

// Componente que usa el hook
const ProductList = ({ category, searchQuery }: { 
  category?: string; 
  searchQuery?: string 
}) => {
  const { products, loading, error, refetch } = useProducts({ 
    category, 
    query: searchQuery 
  });
  
  if (loading) {
    return <ActivityIndicator size="large" />;
  }
  
  if (error) {
    return (
      <View>
        <Text>Error: {error}</Text>
        <TouchableOpacity onPress={refetch}>
          <Text>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }
  
  return (
    <FlatList
      data={products}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <View>
          <Text>{item.name}</Text>
          <Text>${item.price}</Text>
          <Text>{item.description}</Text>
        </View>
      )}
    />
  );
};
```

### Patrones Compuestos

Los patrones compuestos combinan **múltiples patrones de diseño** para crear soluciones más robustas y flexibles.

#### Composite + Strategy + Observer

```typescript
// Sistema de notificaciones con patrones compuestos

// 1. COMPOSITE - Estructura jerárquica de notificaciones
interface NotificationComponent {
  send(): Promise<void>;
  add(child: NotificationComponent): void;
  remove(child: NotificationComponent): void;
  getChildren(): NotificationComponent[];
}

// Notificación individual (Leaf)
class SingleNotification implements NotificationComponent {
  constructor(
    private message: string,
    private strategy: NotificationStrategy,
    private observers: NotificationObserver[] = []
  ) {}
  
  async send(): Promise<void> {
    const result = await this.strategy.send(this.message);
    
    // Notificar a observadores
    this.observers.forEach(observer => observer.onNotificationSent(result));
  }
  
  add(child: NotificationComponent): void {
    // Las notificaciones individuales no pueden tener hijos
    throw new Error('Single notifications cannot have children');
  }
  
  remove(child: NotificationComponent): void {
    throw new Error('Single notifications cannot have children');
  }
  
  getChildren(): NotificationComponent[] {
    return [];
  }
}

// Grupo de notificaciones (Composite)
class NotificationGroup implements NotificationComponent {
  private children: NotificationComponent[] = [];
  
  async send(): Promise<void> {
    // Enviar todas las notificaciones en paralelo
    const promises = this.children.map(child => child.send());
    await Promise.all(promises);
  }
  
  add(child: NotificationComponent): void {
    this.children.push(child);
  }
  
  remove(child: NotificationComponent): void {
    const index = this.children.indexOf(child);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  getChildren(): NotificationComponent[] {
    return [...this.children];
  }
}

// 2. STRATEGY - Diferentes estrategias de envío
interface NotificationStrategy {
  send(message: string): Promise<NotificationResult>;
}

class EmailNotificationStrategy implements NotificationStrategy {
  async send(message: string): Promise<NotificationResult> {
    // Simular envío de email
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    return {
      success: true,
      method: 'email',
      timestamp: new Date(),
      message: `Email sent: ${message}`
    };
  }
}

class PushNotificationStrategy implements NotificationStrategy {
  async send(message: string): Promise<NotificationResult> {
    // Simular envío de push notification
    await new Promise(resolve => setTimeout(resolve, 500));
    
    return {
      success: true,
      method: 'push',
      timestamp: new Date(),
      message: `Push notification sent: ${message}`
    };
  }
}

class SMSNotificationStrategy implements NotificationStrategy {
  async send(message: string): Promise<NotificationResult> {
    // Simular envío de SMS
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    return {
      success: true,
      method: 'sms',
      timestamp: new Date(),
      message: `SMS sent: ${message}`
    };
  }
}

// 3. OBSERVER - Seguimiento de notificaciones
interface NotificationObserver {
  onNotificationSent(result: NotificationResult): void;
}

class NotificationLogger implements NotificationObserver {
  onNotificationSent(result: NotificationResult): void {
    console.log(`[${result.timestamp.toISOString()}] ${result.method}: ${result.message}`);
  }
}

class NotificationAnalytics implements NotificationObserver {
  onNotificationSent(result: NotificationResult): void {
    // Enviar métricas a servicio de analytics
    console.log(`Analytics: ${result.method} notification sent at ${result.timestamp}`);
  }
}

// 4. FACTORY - Creación de notificaciones
class NotificationFactory {
  static createEmailNotification(message: string, observers: NotificationObserver[] = []): SingleNotification {
    return new SingleNotification(message, new EmailNotificationStrategy(), observers);
  }
  
  static createPushNotification(message: string, observers: NotificationObserver[] = []): SingleNotification {
    return new SingleNotification(message, new PushNotificationStrategy(), observers);
  }
  
  static createSMSNotification(message: string, observers: NotificationObserver[] = []): SingleNotification {
    return new SingleNotification(message, new SMSNotificationStrategy(), observers);
  }
  
  static createMultiChannelNotification(message: string, observers: NotificationObserver[] = []): NotificationGroup {
    const group = new NotificationGroup();
    
    group.add(this.createEmailNotification(message, observers));
    group.add(this.createPushNotification(message, observers));
    group.add(this.createSMSNotification(message, observers));
    
    return group;
  }
}

// 5. USO DEL SISTEMA COMPUESTO
class NotificationService {
  private observers: NotificationObserver[] = [];
  
  addObserver(observer: NotificationObserver): void {
    this.observers.push(observer);
  }
  
  removeObserver(observer: NotificationObserver): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  
  async sendNotification(message: string, type: 'email' | 'push' | 'sms' | 'multi'): Promise<void> {
    let notification: NotificationComponent;
    
    switch (type) {
      case 'email':
        notification = NotificationFactory.createEmailNotification(message, this.observers);
        break;
      case 'push':
        notification = NotificationFactory.createPushNotification(message, this.observers);
        break;
      case 'sms':
        notification = NotificationFactory.createSMSNotification(message, this.observers);
        break;
      case 'multi':
        notification = NotificationFactory.createMultiChannelNotification(message, this.observers);
        break;
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
    
    await notification.send();
  }
}

// Uso del sistema compuesto
const notificationService = new NotificationService();

// Agregar observadores
notificationService.addObserver(new NotificationLogger());
notificationService.addObserver(new NotificationAnalytics());

// Enviar diferentes tipos de notificaciones
await notificationService.sendNotification('Welcome to our app!', 'email');
await notificationService.sendNotification('New message received', 'push');
await notificationService.sendNotification('Order confirmed', 'sms');
await notificationService.sendNotification('System maintenance', 'multi');
```

#### Repository + Factory + Strategy + Observer

```typescript
// Sistema de caché inteligente con patrones compuestos

// 1. REPOSITORY - Interfaz para acceso a datos
interface DataRepository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<void>;
  delete(id: string): Promise<void>;
}

// 2. STRATEGY - Diferentes estrategias de caché
interface CacheStrategy {
  get(key: string): Promise<any>;
  set(key: string, value: any, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
  clear(): Promise<void>;
}

class MemoryCacheStrategy implements CacheStrategy {
  private cache = new Map<string, { value: any; expires: number }>();
  
  async get(key: string): Promise<any> {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() > item.expires) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  async set(key: string, value: any, ttl: number = 300000): Promise<void> {
    this.cache.set(key, {
      value,
      expires: Date.now() + ttl
    });
  }
  
  async delete(key: string): Promise<void> {
    this.cache.delete(key);
  }
  
  async clear(): Promise<void> {
    this.cache.clear();
  }
}

class AsyncStorageCacheStrategy implements CacheStrategy {
  async get(key: string): Promise<any> {
    try {
      const item = await AsyncStorage.getItem(key);
      if (!item) return null;
      
      const { value, expires } = JSON.parse(item);
      
      if (Date.now() > expires) {
        await this.delete(key);
        return null;
      }
      
      return value;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }
  
  async set(key: string, value: any, ttl: number = 300000): Promise<void> {
    try {
      const item = {
        value,
        expires: Date.now() + ttl
      };
      
      await AsyncStorage.setItem(key, JSON.stringify(item));
    } catch (error) {
      console.error('Cache set error:', error);
    }
  }
  
  async delete(key: string): Promise<void> {
    try {
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Cache delete error:', error);
    }
  }
  
  async clear(): Promise<void> {
    try {
      const keys = await AsyncStorage.getAllKeys();
      await AsyncStorage.multiRemove(keys);
    } catch (error) {
      console.error('Cache clear error:', error);
    }
  }
}

// 3. FACTORY - Creación de estrategias de caché
class CacheStrategyFactory {
  static createStrategy(type: 'memory' | 'asyncStorage'): CacheStrategy {
    switch (type) {
      case 'memory':
        return new MemoryCacheStrategy();
      case 'asyncStorage':
        return new AsyncStorageCacheStrategy();
      default:
        throw new Error(`Unknown cache strategy: ${type}`);
    }
  }
}

// 4. OBSERVER - Monitoreo de operaciones de caché
interface CacheObserver {
  onCacheHit(key: string): void;
  onCacheMiss(key: string): void;
  onCacheSet(key: string): void;
  onCacheDelete(key: string): void;
}

class CacheMetrics implements CacheObserver {
  private metrics = {
    hits: 0,
    misses: 0,
    sets: 0,
    deletes: 0
  };
  
  onCacheHit(key: string): void {
    this.metrics.hits++;
    console.log(`Cache hit for key: ${key}`);
  }
  
  onCacheMiss(key: string): void {
    this.metrics.misses++;
    console.log(`Cache miss for key: ${key}`);
  }
  
  onCacheSet(key: string): void {
    this.metrics.sets++;
    console.log(`Cache set for key: ${key}`);
  }
  
  onCacheDelete(key: string): void {
    this.metrics.deletes++;
    console.log(`Cache delete for key: ${key}`);
  }
  
  getMetrics() {
    return { ...this.metrics };
  }
  
  getHitRate(): number {
    const total = this.metrics.hits + this.metrics.misses;
    return total > 0 ? this.metrics.hits / total : 0;
  }
}

// 5. REPOSITORIO CON CACHÉ INTELIGENTE
class CachedRepository<T> implements DataRepository<T> {
  constructor(
    private repository: DataRepository<T>,
    private cacheStrategy: CacheStrategy,
    private observers: CacheObserver[] = []
  ) {}
  
  private notifyObservers(event: keyof CacheObserver, key: string): void {
    this.observers.forEach(observer => {
      const method = observer[event];
      if (typeof method === 'function') {
        method.call(observer, key);
      }
    });
  }
  
  async findById(id: string): Promise<T | null> {
    const cacheKey = `entity_${id}`;
    
    // Intentar obtener del caché
    const cached = await this.cacheStrategy.get(cacheKey);
    if (cached) {
      this.notifyObservers('onCacheHit', cacheKey);
      return cached;
    }
  
    this.notifyObservers('onCacheMiss', cacheKey);
    
    // Obtener del repositorio
    const entity = await this.repository.findById(id);
    
    if (entity) {
      // Guardar en caché
      await this.cacheStrategy.set(cacheKey, entity, 300000); // 5 minutos
      this.notifyObservers('onCacheSet', cacheKey);
    }
    
    return entity;
  }
  
  async findAll(): Promise<T[]> {
    const cacheKey = 'entities_all';
    
    // Intentar obtener del caché
    const cached = await this.cacheStrategy.get(cacheKey);
    if (cached) {
      this.notifyObservers('onCacheHit', cacheKey);
      return cached;
    }
    
    this.notifyObservers('onCacheMiss', cacheKey);
    
    // Obtener del repositorio
    const entities = await this.repository.findAll();
    
    // Guardar en caché
    await this.cacheStrategy.set(cacheKey, entities, 600000); // 10 minutos
    this.notifyObservers('onCacheSet', cacheKey);
    
    return entities;
  }
  
  async save(entity: T): Promise<void> {
    await this.repository.save(entity);
    
    // Invalidar caché relacionado
    const id = (entity as any).id;
    if (id) {
      await this.cacheStrategy.delete(`entity_${id}`);
      this.notifyObservers('onCacheDelete', `entity_${id}`);
    }
    
    // Invalidar caché de lista
    await this.cacheStrategy.delete('entities_all');
    this.notifyObservers('onCacheDelete', 'entities_all');
  }
  
  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
    
    // Invalidar caché relacionado
    await this.cacheStrategy.delete(`entity_${id}`);
    this.notifyObservers('onCacheDelete', `entity_${id}`);
    
    // Invalidar caché de lista
    await this.cacheStrategy.delete('entities_all');
    this.notifyObservers('onCacheDelete', 'entities_all');
  }
}

// 6. USO DEL SISTEMA COMPUESTO
class UserService {
  constructor(private userRepository: DataRepository<User>) {}
  
  async getUser(id: string): Promise<User | null> {
    return this.userRepository.findById(id);
  }
  
  async getAllUsers(): Promise<User[]> {
    return this.userRepository.findAll();
  }
  
  async createUser(userData: Omit<User, 'id'>): Promise<User> {
    const user: User = {
      ...userData,
      id: this.generateId()
    };
    
    await this.userRepository.save(user);
    return user;
  }
  
  async updateUser(id: string, userData: Partial<User>): Promise<void> {
    const existingUser = await this.userRepository.findById(id);
    if (!existingUser) {
      throw new Error('User not found');
    }
    
    const updatedUser = { ...existingUser, ...userData };
    await this.userRepository.save(updatedUser);
  }
  
  async deleteUser(id: string): Promise<void> {
    await this.userRepository.delete(id);
  }
  
  private generateId(): string {
    return `user_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Implementación del repositorio base
class ApiUserRepository implements DataRepository<User> {
  constructor(private apiClient: ApiClient) {}
  
  async findById(id: string): Promise<User | null> {
    try {
      const response = await this.apiClient.get(`/users/${id}`);
      return response.data;
    } catch (error) {
      if (error.status === 404) return null;
      throw error;
    }
  }
  
  async findAll(): Promise<User[]> {
    const response = await this.apiClient.get('/users');
    return response.data;
  }
  
  async save(user: User): Promise<void> {
    if (user.id) {
      await this.apiClient.put(`/users/${user.id}`, user);
    } else {
      await this.apiClient.post('/users', user);
    }
  }
  
  async delete(id: string): Promise<void> {
    await this.apiClient.delete(`/users/${id}`);
  }
}

// Composición final
const createUserService = (): UserService => {
  // Crear estrategia de caché
  const cacheStrategy = CacheStrategyFactory.createStrategy('memory');
  
  // Crear repositorio base
  const apiClient = new ApiClient('https://api.example.com');
  const baseRepository = new ApiUserRepository(apiClient);
  
  // Crear repositorio con caché
  const cachedRepository = new CachedRepository(baseRepository, cacheStrategy);
  
  // Agregar observadores
  const metrics = new CacheMetrics();
  cachedRepository.observers.push(metrics);
  
  // Crear servicio
  return new UserService(cachedRepository);
};

// Hook para usar el servicio
const useUserService = () => {
  const [userService] = useState(() => createUserService());
  
  return userService;
};

// Componente que usa el servicio
const UserManagement = () => {
  const userService = useUserService();
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  
  const loadUsers = useCallback(async () => {
    setLoading(true);
    try {
      const result = await userService.getAllUsers();
      setUsers(result);
    } catch (error) {
      console.error('Error loading users:', error);
    } finally {
      setLoading(false);
    }
  }, [userService]);
  
  useEffect(() => {
    loadUsers();
  }, [loadUsers]);
  
  const handleCreateUser = useCallback(async () => {
    try {
      const newUser = await userService.createUser({
        name: 'New User',
        email: 'newuser@example.com',
        isActive: true
      });
      
      setUsers(prev => [...prev, newUser]);
    } catch (error) {
      console.error('Error creating user:', error);
    }
  }, [userService]);
  
  if (loading) {
    return <ActivityIndicator size="large" />;
  }
  
  return (
    <View>
      <TouchableOpacity onPress={handleCreateUser}>
        <Text>Create New User</Text>
      </TouchableOpacity>
      
      <FlatList
        data={users}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View>
            <Text>{item.name}</Text>
            <Text>{item.email}</Text>
            <Text>{item.isActive ? 'Active' : 'Inactive'}</Text>
          </View>
        )}
      />
    </View>
  );
};
```

## Ejercicios Prácticos

### Ejercicio 1: Implementación de Clean Architecture
Crea una aplicación de gestión de tareas con Clean Architecture:

```typescript
// Implementa:
// - Domain layer con entidades Task y casos de uso
// - Data layer con repositorios
// - Presentation layer con hooks personalizados
// - Infrastructure layer con AsyncStorage
// - Composición de dependencias
```

### Ejercicio 2: Patrón Compuesto Personalizado
Combina múltiples patrones para crear un sistema de logging:

```typescript
// Implementa:
// - Strategy para diferentes tipos de log (console, file, remote)
// - Observer para notificar eventos de log
// - Factory para crear estrategias
// - Decorator para añadir funcionalidad
// - Hook personalizado para usar el sistema
```

### Ejercicio 3: Refactorización de Aplicación
Refactoriza una aplicación existente aplicando patrones:

```typescript
// Identifica:
// - Componentes monolíticos para separar
// - Lógica de negocio para extraer
// - Dependencias para invertir
// - Patrones apropiados para aplicar
// - Estructura de carpetas para reorganizar
```

## Resumen de la Clase

### Conceptos Clave Aprendidos:
- ✅ **Clean Architecture** separa preocupaciones en capas
- ✅ **Patrones compuestos** combinan múltiples patrones
- ✅ **Separación de responsabilidades** mejora la mantenibilidad
- ✅ **Inversión de dependencias** hace el código más testeable
- ✅ **Arquitectura limpia** facilita el escalado y mantenimiento

### Próximos Pasos:
- Aplicar Clean Architecture en proyectos React Native
- Combinar patrones de diseño en soluciones complejas
- Refactorizar código existente con patrones

### Recursos Adicionales:
- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [React Native Architecture Patterns](https://reactnative.dev/docs/architecture-overview)
- [TypeScript Design Patterns](https://www.typescriptlang.org/docs/)

---

## Navegación
- **Anterior**: [Clase 4: Patrones de Comportamiento](clase_4_patrones_comportamiento.md)
- **Siguiente**: [Módulo 9: Arquitectura Avanzada](../senior_2/README.md)
- **README del Módulo**: [Módulo 8: Patrones de Diseño](README.md)
- **Inicio**: [Índice Completo](../../INDICE_COMPLETO.md)
