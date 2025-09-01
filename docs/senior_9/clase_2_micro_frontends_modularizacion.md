# Clase 2: Micro-Frontends y Modularización 🧩

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender** el concepto de micro-frontends y sus beneficios
2. **Implementar** arquitecturas modulares en React Native
3. **Diseñar** sistemas de federación de módulos
4. **Crear** comunicación efectiva entre componentes modulares
5. **Aplicar** patrones de lazy loading y code splitting

## ⏱️ Duración Estimada

**2.5 horas** - Teoría: 1.5h | Práctica: 1h

## 🔗 Navegación

- **📚 Módulo**: [Módulo 16: Arquitecturas Empresariales](../README.md)
- **⬅️ Anterior**: [Clase 1: Fundamentos de Arquitecturas Empresariales](clase_1_fundamentos_arquitecturas_empresariales.md)
- **➡️ Siguiente**: [Clase 3: Feature Flags y Configuración Dinámica](clase_3_feature_flags_configuracion_dinamica.md)
- **🏠 Inicio**: [Índice del Curso](../../../INDICE_COMPLETO.md)

---

## 🎯 Contenido Teórico

### 1. ¿Qué son los Micro-Frontends?

Los **micro-frontends** son una arquitectura donde una aplicación frontend se divide en múltiples aplicaciones más pequeñas e independientes, cada una desarrollada, desplegada y mantenida por equipos diferentes. En React Native, esto se traduce en:

- **Módulos independientes** que pueden desarrollarse por separado
- **Despliegues independientes** sin afectar otras partes de la app
- **Tecnologías heterogéneas** en diferentes módulos
- **Equipos autónomos** trabajando en paralelo

#### 1.1 Beneficios de los Micro-Frontends

```typescript
// ✅ BENEFICIOS:
// 1. Desarrollo paralelo por equipos
// 2. Despliegues independientes
// 3. Escalabilidad del equipo
// 4. Tecnologías heterogéneas
// 5. Mantenimiento simplificado
// 6. Testing aislado por módulo

// ❌ DESAFÍOS:
// 1. Complejidad de integración
// 2. Duplicación de dependencias
// 3. Gestión de estado compartido
// 4. Performance overhead
// 5. Testing de integración
```

### 2. Arquitectura de Micro-Frontends

#### 2.1 Patrón de Composición

```typescript
// core/ModuleRegistry.ts
interface IModule {
  id: string;
  name: string;
  version: string;
  entryPoint: string;
  dependencies: string[];
  metadata: Record<string, any>;
}

class ModuleRegistry {
  private modules: Map<string, IModule> = new Map();
  private loadedModules: Map<string, any> = new Map();
  
  registerModule(module: IModule): void {
    this.modules.set(module.id, module);
  }
  
  async loadModule(moduleId: string): Promise<any> {
    if (this.loadedModules.has(moduleId)) {
      return this.loadedModules.get(moduleId);
    }
    
    const module = this.modules.get(moduleId);
    if (!module) {
      throw new Error(`Module ${moduleId} not found`);
    }
    
    // Verificar dependencias
    await this.ensureDependencies(module.dependencies);
    
    // Cargar módulo dinámicamente
    const moduleInstance = await this.dynamicImport(module.entryPoint);
    this.loadedModules.set(moduleId, moduleInstance);
    
    return moduleInstance;
  }
  
  private async ensureDependencies(dependencies: string[]): Promise<void> {
    for (const depId of dependencies) {
      await this.loadModule(depId);
    }
  }
  
  private async dynamicImport(entryPoint: string): Promise<any> {
    // En React Native, esto podría ser un bundle separado
    // o un módulo que se carga desde el servidor
    return import(entryPoint);
  }
}

// Uso
const registry = new ModuleRegistry();

// Registrar módulos
registry.registerModule({
  id: 'user-management',
  name: 'User Management',
  version: '1.0.0',
  entryPoint: './modules/user-management',
  dependencies: ['core-auth'],
  metadata: { team: 'team-a', priority: 'high' }
});

registry.registerModule({
  id: 'core-auth',
  name: 'Core Authentication',
  version: '1.0.0',
  entryPoint: './modules/core-auth',
  dependencies: [],
  metadata: { team: 'core-team', priority: 'critical' }
});
```

#### 2.2 Sistema de Routing Modular

```typescript
// core/ModularRouter.ts
interface RouteConfig {
  path: string;
  moduleId: string;
  componentName: string;
  params?: Record<string, any>;
  guards?: string[];
}

class ModularRouter {
  private routes: RouteConfig[] = [];
  private moduleRegistry: ModuleRegistry;
  
  constructor(moduleRegistry: ModuleRegistry) {
    this.moduleRegistry = moduleRegistry;
  }
  
  addRoute(route: RouteConfig): void {
    this.routes.push(route);
  }
  
  async navigateTo(path: string, params?: Record<string, any>): Promise<void> {
    const route = this.routes.find(r => this.matchPath(r.path, path));
    if (!route) {
      throw new Error(`Route not found: ${path}`);
    }
    
    // Verificar guards
    if (route.guards) {
      await this.checkGuards(route.guards);
    }
    
    // Cargar módulo si no está cargado
    const module = await this.moduleRegistry.loadModule(route.moduleId);
    
    // Navegar al componente
    await this.navigateToComponent(route.componentName, params);
  }
  
  private matchPath(routePath: string, currentPath: string): boolean {
    // Implementar lógica de matching de rutas
    return routePath === currentPath;
  }
  
  private async checkGuards(guards: string[]): Promise<void> {
    for (const guard of guards) {
      const guardInstance = await this.moduleRegistry.loadModule(guard);
      if (!guardInstance.canActivate()) {
        throw new Error(`Guard ${guard} blocked navigation`);
      }
    }
  }
  
  private async navigateToComponent(componentName: string, params?: Record<string, any>): Promise<void> {
    // Implementar navegación al componente
    console.log(`Navigating to ${componentName} with params:`, params);
  }
}

// Configuración de rutas
const router = new ModularRouter(registry);

router.addRoute({
  path: '/users',
  moduleId: 'user-management',
  componentName: 'UserList',
  guards: ['auth-guard']
});

router.addRoute({
  path: '/users/:id',
  moduleId: 'user-management',
  componentName: 'UserDetail',
  guards: ['auth-guard', 'user-permission-guard']
});
```

### 3. Implementación de Módulos

#### 3.1 Estructura de un Módulo

```typescript
// modules/user-management/index.ts
export interface UserManagementModule {
  components: {
    UserList: React.ComponentType<any>;
    UserDetail: React.ComponentType<any>;
    UserForm: React.ComponentType<any>;
  };
  services: {
    userService: IUserService;
    userValidationService: IUserValidationService;
  };
  hooks: {
    useUsers: () => UseUsersReturn;
    useUser: (id: string) => UseUserReturn;
  };
  constants: {
    USER_ROLES: Record<string, string>;
    VALIDATION_RULES: Record<string, any>;
  };
}

// modules/user-management/components/UserList.tsx
export const UserList: React.FC<UserListProps> = ({ onUserSelect }) => {
  const { users, loading, error, fetchUsers } = useUsers();
  
  useEffect(() => {
    fetchUsers();
  }, []);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <FlatList
      data={users}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <TouchableOpacity onPress={() => onUserSelect(item)}>
          <UserCard user={item} />
        </TouchableOpacity>
      )}
    />
  );
};

// modules/user-management/services/UserService.ts
export class UserService implements IUserService {
  async getUsers(): Promise<User[]> {
    const response = await fetch('/api/users');
    return response.json();
  }
  
  async getUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
  
  async createUser(userData: CreateUserData): Promise<User> {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    return response.json();
  }
}

// modules/user-management/hooks/useUsers.ts
export const useUsers = (): UseUsersReturn => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const fetchUsers = async () => {
    setLoading(true);
    try {
      const data = await userService.getUsers();
      setUsers(data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  return { users, loading, error, fetchUsers };
};

// modules/user-management/index.ts
export const UserManagementModule: UserManagementModule = {
  components: {
    UserList,
    UserDetail,
    UserForm
  },
  services: {
    userService: new UserService(),
    userValidationService: new UserValidationService()
  },
  hooks: {
    useUsers,
    useUser
  },
  constants: {
    USER_ROLES: {
      ADMIN: 'admin',
      USER: 'user',
      MODERATOR: 'moderator'
    },
    VALIDATION_RULES: {
      email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      password: /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$/
    }
  }
};
```

#### 3.2 Lazy Loading de Módulos

```typescript
// core/LazyModuleLoader.ts
class LazyModuleLoader {
  private moduleRegistry: ModuleRegistry;
  private loadingModules: Set<string> = new Set();
  private moduleCache: Map<string, any> = new Map();
  
  constructor(moduleRegistry: ModuleRegistry) {
    this.moduleRegistry = moduleRegistry;
  }
  
  async loadModuleLazy(moduleId: string): Promise<any> {
    // Verificar si ya está en cache
    if (this.moduleCache.has(moduleId)) {
      return this.moduleCache.get(moduleId);
    }
    
    // Verificar si ya se está cargando
    if (this.loadingModules.has(moduleId)) {
      // Esperar a que termine de cargar
      return this.waitForModule(moduleId);
    }
    
    // Marcar como cargando
    this.loadingModules.add(moduleId);
    
    try {
      // Cargar módulo
      const module = await this.moduleRegistry.loadModule(moduleId);
      
      // Guardar en cache
      this.moduleCache.set(moduleId, module);
      
      return module;
    } finally {
      // Remover de la lista de cargando
      this.loadingModules.delete(moduleId);
    }
  }
  
  private async waitForModule(moduleId: string): Promise<any> {
    return new Promise((resolve) => {
      const checkModule = () => {
        if (this.moduleCache.has(moduleId)) {
          resolve(this.moduleCache.get(moduleId));
        } else {
          setTimeout(checkModule, 100);
        }
      };
      checkModule();
    });
  }
  
  preloadModule(moduleId: string): void {
    // Precargar módulo en background
    this.loadModuleLazy(moduleId).catch(console.error);
  }
  
  clearCache(): void {
    this.moduleCache.clear();
  }
}

// Uso con React.lazy
const LazyUserManagement = React.lazy(() => 
  lazyModuleLoader.loadModuleLazy('user-management')
    .then(module => ({ default: module.components.UserList }))
);

const UserManagementWrapper = () => (
  <Suspense fallback={<ActivityIndicator />}>
    <LazyUserManagement />
  </Suspense>
);
```

### 4. Comunicación Entre Módulos

#### 4.1 Event Bus para Comunicación

```typescript
// core/ModuleEventBus.ts
interface ModuleEvent {
  type: string;
  payload: any;
  source: string;
  timestamp: number;
  id: string;
}

class ModuleEventBus {
  private listeners: Map<string, Function[]> = new Map();
  private eventHistory: ModuleEvent[] = [];
  private maxHistorySize = 1000;
  
  emit(eventType: string, payload: any, source: string): void {
    const event: ModuleEvent = {
      type: eventType,
      payload,
      source,
      timestamp: Date.now(),
      id: this.generateEventId()
    };
    
    // Agregar a historial
    this.eventHistory.push(event);
    if (this.eventHistory.length > this.maxHistorySize) {
      this.eventHistory.shift();
    }
    
    // Notificar listeners
    const listeners = this.listeners.get(eventType) || [];
    listeners.forEach(listener => {
      try {
        listener(event);
      } catch (error) {
        console.error(`Error in event listener for ${eventType}:`, error);
      }
    });
  }
  
  on(eventType: string, listener: Function): () => void {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, []);
    }
    
    this.listeners.get(eventType)!.push(listener);
    
    // Retornar función para desuscribirse
    return () => {
      const listeners = this.listeners.get(eventType) || [];
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  }
  
  off(eventType: string, listener: Function): void {
    const listeners = this.listeners.get(eventType) || [];
    const index = listeners.indexOf(listener);
    if (index > -1) {
      listeners.splice(index, 1);
    }
  }
  
  getEventHistory(eventType?: string): ModuleEvent[] {
    if (eventType) {
      return this.eventHistory.filter(event => event.type === eventType);
    }
    return [...this.eventHistory];
  }
  
  private generateEventId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Uso en módulos
const eventBus = new ModuleEventBus();

// Módulo A emite evento
eventBus.emit('user:created', { userId: '123', email: 'user@example.com' }, 'user-management');

// Módulo B escucha evento
const unsubscribe = eventBus.on('user:created', (event) => {
  console.log(`User created in ${event.source}:`, event.payload);
  // Realizar acciones basadas en el evento
  notificationService.sendWelcomeEmail(event.payload.email);
  analyticsService.trackUserCreation(event.payload.userId);
});

// Desuscribirse cuando no se necesite
unsubscribe();
```

#### 4.2 Estado Compartido Entre Módulos

```typescript
// core/SharedState.ts
interface StateChange {
  key: string;
  oldValue: any;
  newValue: any;
  timestamp: number;
  source: string;
}

class SharedState {
  private state: Map<string, any> = new Map();
  private listeners: Map<string, Function[]> = new Map();
  private changeHistory: StateChange[] = [];
  private maxHistorySize = 500;
  
  set(key: string, value: any, source: string): void {
    const oldValue = this.state.get(key);
    
    // Solo actualizar si el valor cambió
    if (oldValue !== value) {
      this.state.set(key, value);
      
      // Registrar cambio
      const change: StateChange = {
        key,
        oldValue,
        newValue: value,
        timestamp: Date.now(),
        source
      };
      
      this.changeHistory.push(change);
      if (this.changeHistory.length > this.maxHistorySize) {
        this.changeHistory.shift();
      }
      
      // Notificar listeners
      this.notifyListeners(key, change);
    }
  }
  
  get(key: string): any {
    return this.state.get(key);
  }
  
  has(key: string): boolean {
    return this.state.has(key);
  }
  
  delete(key: string, source: string): boolean {
    if (this.state.has(key)) {
      const oldValue = this.state.get(key);
      this.state.delete(key);
      
      // Registrar cambio
      const change: StateChange = {
        key,
        oldValue,
        newValue: undefined,
        timestamp: Date.now(),
        source
      };
      
      this.changeHistory.push(change);
      this.notifyListeners(key, change);
      
      return true;
    }
    return false;
  }
  
  watch(key: string, listener: Function): () => void {
    if (!this.listeners.has(key)) {
      this.listeners.set(key, []);
    }
    
    this.listeners.get(key)!.push(listener);
    
    // Retornar función para desuscribirse
    return () => {
      const listeners = this.listeners.get(key) || [];
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  }
  
  private notifyListeners(key: string, change: StateChange): void {
    const listeners = this.listeners.get(key) || [];
    listeners.forEach(listener => {
      try {
        listener(change);
      } catch (error) {
        console.error(`Error in state listener for ${key}:`, error);
      }
    });
  }
  
  getChangeHistory(key?: string): StateChange[] {
    if (key) {
      return this.changeHistory.filter(change => change.key === key);
    }
    return [...this.changeHistory];
  }
  
  clear(): void {
    this.state.clear();
    this.listeners.clear();
    this.changeHistory.length = 0;
  }
}

// Uso en módulos
const sharedState = new SharedState();

// Módulo A establece estado
sharedState.set('currentUser', { id: '123', name: 'John' }, 'auth-module');

// Módulo B observa cambios
const unsubscribe = sharedState.watch('currentUser', (change) => {
  console.log(`User changed from ${change.oldValue?.name} to ${change.newValue?.name}`);
  // Actualizar UI o realizar acciones
});

// Módulo C lee estado
const currentUser = sharedState.get('currentUser');
console.log('Current user:', currentUser);
```

### 5. Patrones de Integración

#### 5.1 Patrón Adapter para Módulos Externos

```typescript
// adapters/ExternalModuleAdapter.ts
interface ExternalModuleInterface {
  initialize(config: any): Promise<void>;
  render(container: any, props: any): Promise<void>;
  destroy(): Promise<void>;
  getMetadata(): Record<string, any>;
}

class ExternalModuleAdapter implements ExternalModuleInterface {
  private externalModule: any;
  private config: any;
  
  constructor(externalModule: any) {
    this.externalModule = externalModule;
  }
  
  async initialize(config: any): Promise<void> {
    this.config = config;
    
    // Adaptar configuración al formato esperado por el módulo externo
    const adaptedConfig = this.adaptConfig(config);
    
    // Inicializar módulo externo
    if (this.externalModule.init) {
      await this.externalModule.init(adaptedConfig);
    }
  }
  
  async render(container: any, props: any): Promise<void> {
    // Adaptar props al formato esperado
    const adaptedProps = this.adaptProps(props);
    
    // Renderizar módulo externo
    if (this.externalModule.render) {
      await this.externalModule.render(container, adaptedProps);
    }
  }
  
  async destroy(): Promise<void> {
    // Limpiar módulo externo
    if (this.externalModule.cleanup) {
      await this.externalModule.cleanup();
    }
  }
  
  getMetadata(): Record<string, any> {
    // Obtener metadatos del módulo externo
    return this.externalModule.metadata || {};
  }
  
  private adaptConfig(config: any): any {
    // Implementar lógica de adaptación de configuración
    return {
      ...config,
      // Adaptaciones específicas
    };
  }
  
  private adaptProps(props: any): any {
    // Implementar lógica de adaptación de props
    return {
      ...props,
      // Adaptaciones específicas
    };
  }
}

// Uso
const externalModule = await loadExternalModule('legacy-module');
const adapter = new ExternalModuleAdapter(externalModule);

await adapter.initialize({ apiUrl: '/api' });
await adapter.render(containerElement, { userId: '123' });
```

#### 5.2 Patrón Facade para Simplificar APIs

```typescript
// facades/ModuleFacade.ts
class ModuleFacade {
  private moduleRegistry: ModuleRegistry;
  private eventBus: ModuleEventBus;
  private sharedState: SharedState;
  
  constructor(
    moduleRegistry: ModuleRegistry,
    eventBus: ModuleEventBus,
    sharedState: SharedState
  ) {
    this.moduleRegistry = moduleRegistry;
    this.eventBus = eventBus;
    this.sharedState = sharedState;
  }
  
  // API simplificada para cargar módulos
  async loadModule(moduleId: string): Promise<any> {
    try {
      return await this.moduleRegistry.loadModule(moduleId);
    } catch (error) {
      console.error(`Failed to load module ${moduleId}:`, error);
      throw error;
    }
  }
  
  // API simplificada para comunicación
  emit(eventType: string, payload: any, source: string): void {
    this.eventBus.emit(eventType, payload, source);
  }
  
  on(eventType: string, listener: Function): () => void {
    return this.eventBus.on(eventType, listener);
  }
  
  // API simplificada para estado compartido
  setState(key: string, value: any, source: string): void {
    this.sharedState.set(key, value, source);
  }
  
  getState(key: string): any {
    return this.sharedState.get(key);
  }
  
  watchState(key: string, listener: Function): () => void {
    return this.sharedState.watch(key, listener);
  }
  
  // Métodos de utilidad
  async preloadModules(moduleIds: string[]): Promise<void> {
    const promises = moduleIds.map(id => 
      this.moduleRegistry.loadModule(id).catch(console.error)
    );
    await Promise.all(promises);
  }
  
  getLoadedModules(): string[] {
    // Implementar lógica para obtener módulos cargados
    return [];
  }
  
  clearCache(): void {
    // Implementar lógica para limpiar cache
  }
}

// Uso simplificado
const facade = new ModuleFacade(moduleRegistry, eventBus, sharedState);

// Cargar módulo
const userModule = await facade.loadModule('user-management');

// Comunicación
facade.emit('user:selected', { userId: '123' }, 'current-module');

// Estado compartido
facade.setState('selectedUserId', '123', 'current-module');
const selectedUserId = facade.getState('selectedUserId');
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Crear un Sistema de Módulos

**Objetivo**: Implementar un sistema completo de módulos con registro, carga y comunicación.

**Requisitos**:
1. **Crear ModuleRegistry** que permita registrar y cargar módulos
2. **Implementar sistema de dependencias** entre módulos
3. **Crear módulo de ejemplo** (ej: calculadora) con componentes y servicios
4. **Implementar comunicación** entre módulos usando EventBus

**Código Base**:
```typescript
// Implementar desde cero
interface IModule {
  id: string;
  name: string;
  version: string;
  entryPoint: string;
  dependencies: string[];
}

class ModuleRegistry {
  // Implementar registro y carga de módulos
}

class ModuleEventBus {
  // Implementar sistema de eventos
}

// Módulo de ejemplo: Calculadora
const CalculatorModule = {
  // Implementar módulo completo
};
```

### Ejercicio 2: Sistema de Routing Modular

**Objetivo**: Crear un sistema de navegación que soporte módulos dinámicos.

**Requisitos**:
1. **Implementar ModularRouter** con configuración de rutas
2. **Soporte para guards** de navegación
3. **Lazy loading** de módulos según la ruta
4. **Navegación programática** entre módulos

**Código Base**:
```typescript
// Implementar desde cero
interface RouteConfig {
  path: string;
  moduleId: string;
  componentName: string;
  guards?: string[];
}

class ModularRouter {
  // Implementar routing modular
}

// Configurar rutas
const routes = [
  { path: '/calculator', moduleId: 'calculator', componentName: 'Calculator' },
  { path: '/users', moduleId: 'user-management', componentName: 'UserList' }
];
```

### Ejercicio 3: Integración de Módulos Externos

**Objetivo**: Crear adaptadores para integrar módulos con diferentes APIs.

**Requisitos**:
1. **Implementar ExternalModuleAdapter** para módulos externos
2. **Crear ModuleFacade** que simplifique la API
3. **Integrar módulo legacy** con el sistema modular
4. **Manejar errores** y fallbacks

---

## 📚 Recursos Adicionales

### Documentación
- [Micro-Frontends.org](https://micro-frontends.org/)
- [Module Federation](https://webpack.js.org/concepts/module-federation/)
- [React Lazy Loading](https://reactjs.org/docs/code-splitting.html)

### Herramientas
- [Webpack Module Federation](https://webpack.js.org/concepts/module-federation/)
- [Single-SPA](https://single-spa.js.org/)
- [React.lazy](https://reactjs.org/docs/code-splitting.html#reactlazy)

### Próximos Pasos
- **Clase 3**: Feature Flags y Configuración Dinámica
- **Clase 4**: Arquitecturas Multi-Tenant
- **Clase 5**: Sistemas Event-Driven y Patrones Avanzados

---

## ✅ Checklist de Completado

- [ ] Comprendí el concepto de micro-frontends y sus beneficios
- [ ] Implementé arquitecturas modulares en React Native
- [ ] Diseñé sistemas de federación de módulos
- [ ] Creé comunicación efectiva entre componentes modulares
- [ ] Apliqué patrones de lazy loading y code splitting
- [ ] Implementé ModuleRegistry para gestión de módulos
- [ ] Creé sistema de routing modular con guards
- [ ] Implementé comunicación entre módulos con EventBus
- [ ] Apliqué patrones de integración (Adapter, Facade)
- [ ] Completé los ejercicios prácticos de modularización

---

**🎯 Próximo Objetivo**: En la siguiente clase aprenderemos sobre **Feature Flags y Configuración Dinámica**, implementando sistemas que permitan controlar funcionalidades de la aplicación en tiempo real sin necesidad de nuevos despliegues.
