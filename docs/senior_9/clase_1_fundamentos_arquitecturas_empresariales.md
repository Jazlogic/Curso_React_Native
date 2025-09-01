# Clase 1: Fundamentos de Arquitecturas Empresariales 🏢

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender** los principios fundamentales de arquitecturas empresariales
2. **Identificar** patrones de diseño arquitectónico comunes
3. **Aplicar** principios de escalabilidad y mantenibilidad
4. **Diseñar** arquitecturas modulares y desacopladas
5. **Evaluar** la calidad arquitectónica de aplicaciones existentes

## ⏱️ Duración Estimada

**2.5 horas** - Teoría: 1.5h | Práctica: 1h

## 🔗 Navegación

- **📚 Módulo**: [Módulo 16: Arquitecturas Empresariales](../README.md)
- **⬅️ Anterior**: [Clase 5: Testing Automatizado y CI/CD](../senior_8/clase_5_testing_automatizado_cicd.md)
- **➡️ Siguiente**: [Clase 2: Micro-Frontends y Modularización](clase_2_micro_frontends_modularizacion.md)
- **🏠 Inicio**: [Índice del Curso](../../../INDICE_COMPLETO.md)

---

## 🎯 Contenido Teórico

### 1. ¿Qué es una Arquitectura Empresarial?

Una **arquitectura empresarial** es la estructura organizativa y tecnológica que define cómo se construye, despliega y mantiene una aplicación a nivel empresarial. No se trata solo de código, sino de un sistema completo que considera:

- **Escalabilidad**: Capacidad de crecer sin degradar el rendimiento
- **Mantenibilidad**: Facilidad de modificar y actualizar el sistema
- **Robustez**: Resistencia a fallos y recuperación ante errores
- **Seguridad**: Protección de datos y acceso controlado
- **Performance**: Rendimiento óptimo bajo diferentes cargas

### 2. Principios Fundamentales

#### 2.1 Separación de Responsabilidades (Separation of Concerns)

```typescript
// ❌ MAL: Todo mezclado en un componente
const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const fetchUser = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/users/me');
      const userData = await response.json();
      setUser(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const updateUser = async (userData) => {
    // Lógica de actualización mezclada con UI
  };
  
  // UI mezclada con lógica de negocio
  return (
    <View>
      {loading ? <ActivityIndicator /> : <UserInfo user={user} />}
    </View>
  );
};

// ✅ BIEN: Responsabilidades separadas
// hooks/useUser.ts
export const useUser = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const fetchUser = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/users/me');
      const userData = await response.json();
      setUser(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return { user, loading, fetchUser };
};

// services/userService.ts
export const userService = {
  updateUser: async (userData) => {
    const response = await fetch('/api/users/me', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    return response.json();
  }
};

// components/UserProfile.tsx
const UserProfile = () => {
  const { user, loading, fetchUser } = useUser();
  
  // Solo se encarga de la UI
  return (
    <View>
      {loading ? <ActivityIndicator /> : <UserInfo user={user} />}
    </View>
  );
};
```

#### 2.2 Inversión de Dependencias (Dependency Inversion)

```typescript
// ❌ MAL: Dependencia directa de implementación
class UserRepository {
  async getUser(id: string) {
    return await fetch(`/api/users/${id}`).then(r => r.json());
  }
}

class UserService {
  private userRepo = new UserRepository(); // Dependencia concreta
  
  async getUser(id: string) {
    return await this.userRepo.getUser(id);
  }
}

// ✅ BIEN: Dependencia de abstracción
interface IUserRepository {
  getUser(id: string): Promise<User>;
}

class UserRepository implements IUserRepository {
  async getUser(id: string) {
    return await fetch(`/api/users/${id}`).then(r => r.json());
  }
}

class UserService {
  constructor(private userRepo: IUserRepository) {} // Dependencia de interfaz
  
  async getUser(id: string) {
    return await this.userRepo.getUser(id);
  }
}

// Inyección de dependencias
const userService = new UserService(new UserRepository());
```

#### 2.3 Principio de Responsabilidad Única (Single Responsibility)

```typescript
// ❌ MAL: Múltiples responsabilidades
class UserManager {
  async createUser(userData: UserData) {
    // Validación
    if (!userData.email || !userData.password) {
      throw new Error('Email and password are required');
    }
    
    // Encriptación
    const hashedPassword = await bcrypt.hash(userData.password, 10);
    
    // Persistencia
    const user = await this.userRepository.create({
      ...userData,
      password: hashedPassword
    });
    
    // Notificación
    await this.emailService.sendWelcomeEmail(user.email);
    
    // Logging
    console.log(`User created: ${user.id}`);
    
    return user;
  }
}

// ✅ BIEN: Responsabilidades separadas
class UserValidator {
  validate(userData: UserData): ValidationResult {
    if (!userData.email || !userData.password) {
      return { isValid: false, errors: ['Email and password are required'] };
    }
    return { isValid: true, errors: [] };
  }
}

class PasswordHasher {
  async hash(password: string): Promise<string> {
    return await bcrypt.hash(password, 10);
  }
}

class UserManager {
  constructor(
    private validator: UserValidator,
    private hasher: PasswordHasher,
    private repository: UserRepository,
    private emailService: EmailService,
    private logger: Logger
  ) {}
  
  async createUser(userData: UserData) {
    // Validación
    const validation = this.validator.validate(userData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Encriptación
    const hashedPassword = await this.hasher.hash(userData.password);
    
    // Persistencia
    const user = await this.repository.create({
      ...userData,
      password: hashedPassword
    });
    
    // Notificación
    await this.emailService.sendWelcomeEmail(user.email);
    
    // Logging
    this.logger.info(`User created: ${user.id}`);
    
    return user;
  }
}
```

### 3. Patrones de Diseño Arquitectónico

#### 3.1 Patrón Repository

```typescript
// interfaces/IUserRepository.ts
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(userData: CreateUserData): Promise<User>;
  update(id: string, userData: UpdateUserData): Promise<User>;
  delete(id: string): Promise<void>;
  findAll(filters?: UserFilters): Promise<User[]>;
}

// repositories/UserRepository.ts
class UserRepository implements IUserRepository {
  constructor(private db: Database) {}
  
  async findById(id: string): Promise<User | null> {
    const user = await this.db.users.findOne({ where: { id } });
    return user ? this.mapToUser(user) : null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    const user = await this.db.users.findOne({ where: { email } });
    return user ? this.mapToUser(user) : null;
  }
  
  async create(userData: CreateUserData): Promise<User> {
    const user = await this.db.users.create(userData);
    return this.mapToUser(user);
  }
  
  async update(id: string, userData: UpdateUserData): Promise<User> {
    const user = await this.db.users.update(userData, { where: { id } });
    return this.mapToUser(user);
  }
  
  async delete(id: string): Promise<void> {
    await this.db.users.destroy({ where: { id } });
  }
  
  async findAll(filters?: UserFilters): Promise<User[]> {
    const users = await this.db.users.findAll({ where: filters });
    return users.map(user => this.mapToUser(user));
  }
  
  private mapToUser(dbUser: any): User {
    return {
      id: dbUser.id,
      email: dbUser.email,
      name: dbUser.name,
      createdAt: dbUser.createdAt,
      updatedAt: dbUser.updatedAt
    };
  }
}
```

#### 3.2 Patrón Factory

```typescript
// interfaces/IUserService.ts
interface IUserService {
  createUser(userData: CreateUserData): Promise<User>;
  getUser(id: string): Promise<User>;
}

// factories/UserServiceFactory.ts
class UserServiceFactory {
  static create(config: ServiceConfig): IUserService {
    switch (config.environment) {
      case 'development':
        return new MockUserService();
      case 'production':
        return new RealUserService(
          new UserRepository(new Database(config.database)),
          new EmailService(config.email),
          new Logger(config.logging)
        );
      default:
        throw new Error(`Unknown environment: ${config.environment}`);
    }
  }
}

// Uso
const userService = UserServiceFactory.create({
  environment: 'production',
  database: { host: 'localhost', port: 5432 },
  email: { apiKey: 'key' },
  logging: { level: 'info' }
});
```

#### 3.3 Patrón Observer

```typescript
// interfaces/IEventEmitter.ts
interface IEventEmitter {
  on(event: string, listener: Function): void;
  off(event: string, listener: Function): void;
  emit(event: string, data?: any): void;
}

// core/EventEmitter.ts
class EventEmitter implements IEventEmitter {
  private events: Map<string, Function[]> = new Map();
  
  on(event: string, listener: Function): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(listener);
  }
  
  off(event: string, listener: Function): void {
    const listeners = this.events.get(event);
    if (listeners) {
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    }
  }
  
  emit(event: string, data?: any): void {
    const listeners = this.events.get(event);
    if (listeners) {
      listeners.forEach(listener => listener(data));
    }
  }
}

// services/UserService.ts
class UserService extends EventEmitter {
  async createUser(userData: CreateUserData): Promise<User> {
    const user = await this.userRepository.create(userData);
    
    // Emitir evento de usuario creado
    this.emit('user:created', user);
    
    return user;
  }
}

// Uso
const userService = new UserService();

userService.on('user:created', (user: User) => {
  console.log(`New user created: ${user.email}`);
  // Enviar email de bienvenida
  emailService.sendWelcomeEmail(user.email);
});

userService.on('user:created', (user: User) => {
  // Registrar en analytics
  analytics.track('user_created', { userId: user.id });
});
```

### 4. Principios de Escalabilidad

#### 4.1 Escalabilidad Horizontal vs Vertical

```typescript
// ❌ MAL: Escalabilidad vertical (monolito)
class MonolithicApp {
  private users: User[] = [];
  private products: Product[] = [];
  private orders: Order[] = [];
  
  // Todo en una sola clase - difícil de escalar
  async handleUserRequest() { /* ... */ }
  async handleProductRequest() { /* ... */ }
  async handleOrderRequest() { /* ... */ }
}

// ✅ BIEN: Escalabilidad horizontal (módulos separados)
// services/UserService.ts
class UserService {
  async handleRequest() { /* ... */ }
}

// services/ProductService.ts
class ProductService {
  async handleRequest() { /* ... */ }
}

// services/OrderService.ts
class OrderService {
  async handleRequest() { /* ... */ }
}

// Cada servicio puede ejecutarse en instancias separadas
```

#### 4.2 Caching y Memoria Compartida

```typescript
// interfaces/ICache.ts
interface ICache {
  get(key: string): Promise<any>;
  set(key: string, value: any, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
  clear(): Promise<void>;
}

// services/CacheService.ts
class CacheService implements ICache {
  private cache = new Map<string, { value: any; expires: number }>();
  
  async get(key: string): Promise<any> {
    const item = this.cache.get(key);
    if (item && item.expires > Date.now()) {
      return item.value;
    }
    if (item) {
      this.cache.delete(key);
    }
    return null;
  }
  
  async set(key: string, value: any, ttl: number = 3600000): Promise<void> {
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

// services/UserService.ts
class UserService {
  constructor(
    private userRepository: IUserRepository,
    private cache: ICache
  ) {}
  
  async getUser(id: string): Promise<User> {
    // Intentar obtener del cache primero
    const cachedUser = await this.cache.get(`user:${id}`);
    if (cachedUser) {
      return cachedUser;
    }
    
    // Si no está en cache, obtener de la base de datos
    const user = await this.userRepository.findById(id);
    if (user) {
      // Guardar en cache por 1 hora
      await this.cache.set(`user:${id}`, user, 3600000);
    }
    
    return user;
  }
}
```

### 5. Métricas de Calidad Arquitectónica

#### 5.1 Acoplamiento (Coupling)

```typescript
// ❌ ALTO ACOPLAMIENTO: Dependencias directas
class UserController {
  private userService = new UserService();
  private emailService = new EmailService();
  private logger = new Logger();
  
  async createUser(req: Request, res: Response) {
    // Depende directamente de implementaciones concretas
  }
}

// ✅ BAJO ACOPLAMIENTO: Dependencias inyectadas
class UserController {
  constructor(
    private userService: IUserService,
    private emailService: IEmailService,
    private logger: ILogger
  ) {}
  
  async createUser(req: Request, res: Response) {
    // Depende de interfaces, no de implementaciones
  }
}
```

#### 5.2 Cohesión (Cohesion)

```typescript
// ❌ BAJA COHESIÓN: Responsabilidades mezcladas
class UserManager {
  // Gestión de usuarios
  async createUser() { /* ... */ }
  async updateUser() { /* ... */ }
  
  // Gestión de emails (responsabilidad diferente)
  async sendEmail() { /* ... */ }
  async validateEmail() { /* ... */ }
  
  // Gestión de archivos (responsabilidad diferente)
  async uploadAvatar() { /* ... */ }
  async deleteAvatar() { /* ... */ }
}

// ✅ ALTA COHESIÓN: Responsabilidades relacionadas
class UserManager {
  // Solo gestión de usuarios
  async createUser() { /* ... */ }
  async updateUser() { /* ... */ }
  async deleteUser() { /* ... */ }
  async getUser() { /* ... */ }
}

class EmailManager {
  // Solo gestión de emails
  async sendEmail() { /* ... */ }
  async validateEmail() { /* ... */ }
  async scheduleEmail() { /* ... */ }
}

class FileManager {
  // Solo gestión de archivos
  async uploadFile() { /* ... */ }
  async deleteFile() { /* ... */ }
  async getFile() { /* ... */ }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Refactorización de Arquitectura

**Objetivo**: Refactorizar un componente monolítico aplicando principios de arquitectura empresarial.

**Código Inicial**:
```typescript
// Componente monolítico que necesita refactorización
const UserDashboard = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const fetchUsers = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const createUser = async (userData) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      const newUser = await response.json();
      setUsers([...users, newUser]);
    } catch (err) {
      setError(err.message);
    }
  };
  
  const deleteUser = async (userId) => {
    try {
      await fetch(`/api/users/${userId}`, { method: 'DELETE' });
      setUsers(users.filter(user => user.id !== userId));
    } catch (err) {
      setError(err.message);
    }
  };
  
  useEffect(() => {
    fetchUsers();
  }, []);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <View>
      <Text>Users Dashboard</Text>
      {users.map(user => (
        <View key={user.id}>
          <Text>{user.name}</Text>
          <Text>{user.email}</Text>
          <Button title="Delete" onPress={() => deleteUser(user.id)} />
        </View>
      ))}
    </View>
  );
};
```

**Tareas**:
1. **Separar responsabilidades** en hooks, servicios y componentes
2. **Implementar patrón Repository** para la gestión de usuarios
3. **Aplicar inyección de dependencias** para los servicios
4. **Crear interfaces** para definir contratos claros
5. **Implementar manejo de errores** centralizado

**Solución Esperada**:
```typescript
// interfaces/IUserRepository.ts
interface IUserRepository {
  findAll(): Promise<User[]>;
  create(userData: CreateUserData): Promise<User>;
  delete(id: string): Promise<void>;
}

// repositories/UserRepository.ts
class UserRepository implements IUserRepository {
  async findAll(): Promise<User[]> {
    const response = await fetch('/api/users');
    return response.json();
  }
  
  async create(userData: CreateUserData): Promise<User> {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    return response.json();
  }
  
  async delete(id: string): Promise<void> {
    await fetch(`/api/users/${id}`, { method: 'DELETE' });
  }
}

// hooks/useUsers.ts
export const useUsers = (userRepository: IUserRepository) => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const fetchUsers = async () => {
    setLoading(true);
    try {
      const data = await userRepository.findAll();
      setUsers(data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const createUser = async (userData: CreateUserData) => {
    try {
      const newUser = await userRepository.create(userData);
      setUsers([...users, newUser]);
      setError(null);
    } catch (err) {
      setError(err.message);
    }
  };
  
  const deleteUser = async (userId: string) => {
    try {
      await userRepository.delete(userId);
      setUsers(users.filter(user => user.id !== userId));
      setError(null);
    } catch (err) {
      setError(err.message);
    }
  };
  
  return {
    users,
    loading,
    error,
    fetchUsers,
    createUser,
    deleteUser
  };
};

// components/UserDashboard.tsx
const UserDashboard = () => {
  const userRepository = new UserRepository();
  const { users, loading, error, fetchUsers, createUser, deleteUser } = useUsers(userRepository);
  
  useEffect(() => {
    fetchUsers();
  }, []);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <View>
      <Text>Users Dashboard</Text>
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user} 
          onDelete={() => deleteUser(user.id)} 
        />
      ))}
    </View>
  );
};
```

### Ejercicio 2: Implementación de Patrón Observer

**Objetivo**: Crear un sistema de eventos para notificaciones en tiempo real.

**Requisitos**:
1. **Crear un EventEmitter** que permita suscribirse a eventos
2. **Implementar un sistema de notificaciones** que use el patrón Observer
3. **Crear diferentes tipos de listeners** para diferentes eventos
4. **Implementar un sistema de logging** que registre todos los eventos

**Código Base**:
```typescript
// Implementar desde cero
interface IEventEmitter {
  on(event: string, listener: Function): void;
  off(event: string, listener: Function): void;
  emit(event: string, data?: any): void;
}

class NotificationService {
  // Implementar sistema de notificaciones
}

class LoggingService {
  // Implementar sistema de logging
}
```

### Ejercicio 3: Evaluación de Calidad Arquitectónica

**Objetivo**: Analizar y evaluar la calidad arquitectónica de un código existente.

**Tareas**:
1. **Identificar problemas** de acoplamiento y cohesión
2. **Proponer mejoras** aplicando principios de arquitectura
3. **Crear diagramas** de dependencias
4. **Implementar refactorizaciones** sugeridas

---

## 📚 Recursos Adicionales

### Documentación
- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Design Patterns](https://refactoring.guru/design-patterns)

### Herramientas
- [ArchUnit](https://www.archunit.org/) - Testing de arquitectura
- [Dependency Cruiser](https://github.com/sverweij/dependency-cruiser) - Análisis de dependencias
- [PlantUML](https://plantuml.com/) - Diagramas de arquitectura

### Próximos Pasos
- **Clase 2**: Micro-Frontends y Modularización
- **Clase 3**: Feature Flags y Configuración Dinámica
- **Clase 4**: Arquitecturas Multi-Tenant
- **Clase 5**: Sistemas Event-Driven y Patrones Avanzados

---

## ✅ Checklist de Completado

- [ ] Comprendí los principios fundamentales de arquitecturas empresariales
- [ ] Apliqué patrones de diseño arquitectónico (Repository, Factory, Observer)
- [ ] Implementé principios de escalabilidad horizontal y vertical
- [ ] Refactoricé código aplicando separación de responsabilidades
- [ ] Evalué la calidad arquitectónica de aplicaciones existentes
- [ ] Completé los ejercicios prácticos de refactorización
- [ ] Implementé el patrón Observer para notificaciones
- [ ] Analicé y mejoré la calidad arquitectónica de código

---

**🎯 Próximo Objetivo**: En la siguiente clase aprenderemos sobre **Micro-Frontends y Modularización**, aplicando los principios arquitectónicos que acabamos de estudiar para crear aplicaciones modulares y escalables.
