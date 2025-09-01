# Clase 5: Sistemas Event-Driven y Patrones Avanzados 🚀

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender** los principios de arquitecturas event-driven
2. **Implementar** sistemas de mensajería y eventos
3. **Aplicar** patrones CQRS y Event Sourcing
4. **Crear** sistemas de procesamiento de eventos asíncrono
5. **Diseñar** arquitecturas escalables basadas en eventos

## ⏱️ Duración Estimada

**2 horas** - Teoría: 1h | Práctica: 1h

## 🔗 Navegación

- **📚 Módulo**: [Módulo 16: Arquitecturas Empresariales](../README.md)
- **⬅️ Anterior**: [Clase 4: Arquitecturas Multi-Tenant](clase_4_arquitecturas_multi_tenant.md)
- **🏠 Inicio**: [Índice del Curso](../../../INDICE_COMPLETO.md)

---

## 🎯 Contenido Teórico

### 1. ¿Qué son las Arquitecturas Event-Driven?

Una **arquitectura event-driven** es un patrón de diseño donde los componentes se comunican a través de eventos, permitiendo un desacoplamiento completo entre productores y consumidores. En React Native, esto facilita:

- **Desacoplamiento** entre diferentes partes de la aplicación
- **Escalabilidad** al procesar eventos de forma asíncrona
- **Mantenibilidad** al separar responsabilidades
- **Testabilidad** al aislar componentes
- **Extensibilidad** al agregar nuevos listeners sin modificar código existente

#### 1.1 Beneficios de las Arquitecturas Event-Driven

```typescript
// ✅ BENEFICIOS:

// 1. Desacoplamiento: Los componentes no conocen entre sí
// 2. Escalabilidad: Procesamiento paralelo de eventos
// 3. Mantenibilidad: Cambios aislados sin afectar otros componentes
// 4. Testabilidad: Testing unitario de componentes aislados
// 5. Extensibilidad: Nuevas funcionalidades sin modificar código existente
// 6. Resiliencia: Fallos aislados no afectan todo el sistema

// ❌ DESAFÍOS:

// 1. Complejidad: Mayor complejidad en el flujo de datos
// 2. Debugging: Dificultad para rastrear el flujo de eventos
// 3. Consistencia: Garantizar consistencia en sistemas distribuidos
// 4. Testing: Testing de integración más complejo
// 5. Performance: Overhead en el procesamiento de eventos
```

### 2. Core Event System

#### 2.1 Event Bus Avanzado

```typescript
// core/AdvancedEventBus.ts
interface Event {
  id: string;
  type: string;
  payload: any;
  metadata: EventMetadata;
  timestamp: Date;
  source: string;
  correlationId?: string;
  causationId?: string;
}

interface EventMetadata {
  version: string;
  schema: string;
  priority: 'low' | 'normal' | 'high' | 'critical';
  retryCount: number;
  maxRetries: number;
  ttl?: number;
}

interface EventHandler {
  id: string;
  eventType: string;
  handler: (event: Event) => Promise<void>;
  priority: number;
  retryPolicy: RetryPolicy;
  errorHandler?: (error: Error, event: Event) => Promise<void>;
}

interface RetryPolicy {
  maxRetries: number;
  backoffStrategy: 'linear' | 'exponential' | 'fixed';
  initialDelay: number;
  maxDelay: number;
}

class AdvancedEventBus {
  private handlers: Map<string, EventHandler[]> = new Map();
  private eventHistory: Event[] = [];
  private maxHistorySize = 10000;
  private processingEvents: Set<string> = new Set();
  private deadLetterQueue: Event[] = [];
  private metrics: EventMetrics = {
    totalEvents: 0,
    processedEvents: 0,
    failedEvents: 0,
    averageProcessingTime: 0
  };
  
  async publish(event: Event): Promise<void> {
    try {
      // Validar evento
      this.validateEvent(event);
      
      // Agregar a historial
      this.addToHistory(event);
      
      // Obtener handlers para este tipo de evento
      const handlers = this.handlers.get(event.type) || [];
      
      if (handlers.length === 0) {
        console.warn(`No handlers found for event type: ${event.type}`);
        return;
      }
      
      // Ordenar handlers por prioridad
      const sortedHandlers = handlers.sort((a, b) => b.priority - a.priority);
      
      // Procesar eventos en paralelo
      const processingPromises = sortedHandlers.map(handler => 
        this.processEventWithHandler(event, handler)
      );
      
      await Promise.allSettled(processingPromises);
      
      // Actualizar métricas
      this.updateMetrics(event, 'success');
      
    } catch (error) {
      console.error('Failed to publish event:', error);
      this.updateMetrics(event, 'error');
      throw error;
    }
  }
  
  subscribe(
    eventType: string,
    handler: (event: Event) => Promise<void>,
    options: Partial<EventHandler> = {}
  ): string {
    const handlerId = this.generateHandlerId();
    
    const eventHandler: EventHandler = {
      id: handlerId,
      eventType,
      handler,
      priority: options.priority || 0,
      retryPolicy: options.retryPolicy || {
        maxRetries: 3,
        backoffStrategy: 'exponential',
        initialDelay: 1000,
        maxDelay: 30000
      },
      errorHandler: options.errorHandler
    };
    
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    
    this.handlers.get(eventType)!.push(eventHandler);
    
    return handlerId;
  }
  
  unsubscribe(handlerId: string): boolean {
    for (const [eventType, handlers] of this.handlers.entries()) {
      const index = handlers.findIndex(h => h.id === handlerId);
      if (index >= 0) {
        handlers.splice(index, 1);
        
        // Si no hay más handlers para este tipo, limpiar la entrada
        if (handlers.length === 0) {
          this.handlers.delete(eventType);
        }
        
        return true;
      }
    }
    
    return false;
  }
  
  async replayEvents(eventType: string, fromTimestamp: Date): Promise<void> {
    const events = this.eventHistory.filter(
      event => event.type === eventType && event.timestamp >= fromTimestamp
    );
    
    console.log(`Replaying ${events.length} events of type ${eventType}`);
    
    for (const event of events) {
      await this.publish(event);
    }
  }
  
  getMetrics(): EventMetrics {
    return { ...this.metrics };
  }
  
  getDeadLetterQueue(): Event[] {
    return [...this.deadLetterQueue];
  }
  
  private async processEventWithHandler(event: Event, handler: EventHandler): Promise<void> {
    const startTime = Date.now();
    
    try {
      // Verificar si el evento ya está siendo procesado
      if (this.processingEvents.has(event.id)) {
        console.warn(`Event ${event.id} is already being processed`);
        return;
      }
      
      this.processingEvents.add(event.id);
      
      // Procesar evento con política de reintentos
      await this.executeWithRetry(event, handler);
      
      const processingTime = Date.now() - startTime;
      this.updateProcessingTime(processingTime);
      
    } catch (error) {
      console.error(`Failed to process event ${event.id} with handler ${handler.id}:`, error);
      
      // Ejecutar error handler si está disponible
      if (handler.errorHandler) {
        try {
          await handler.errorHandler(error, event);
        } catch (errorHandlerError) {
          console.error('Error handler failed:', errorHandlerError);
        }
      }
      
      // Mover a dead letter queue si se agotaron los reintentos
      if (event.metadata.retryCount >= handler.retryPolicy.maxRetries) {
        this.moveToDeadLetterQueue(event);
      }
      
      throw error;
    } finally {
      this.processingEvents.delete(event.id);
    }
  }
  
  private async executeWithRetry(event: Event, handler: EventHandler): Promise<void> {
    let lastError: Error;
    
    for (let attempt = 0; attempt <= handler.retryPolicy.maxRetries; attempt++) {
      try {
        await handler.handler(event);
        return; // Éxito, salir del loop
      } catch (error) {
        lastError = error as Error;
        
        if (attempt === handler.retryPolicy.maxRetries) {
          break; // Último intento falló
        }
        
        // Calcular delay para el siguiente intento
        const delay = this.calculateBackoffDelay(attempt, handler.retryPolicy);
        
        console.log(`Retry attempt ${attempt + 1} for event ${event.id} in ${delay}ms`);
        
        await this.sleep(delay);
      }
    }
    
    // Todos los intentos fallaron
    throw lastError!;
  }
  
  private calculateBackoffDelay(attempt: number, retryPolicy: RetryPolicy): number {
    let delay: number;
    
    switch (retryPolicy.backoffStrategy) {
      case 'linear':
        delay = retryPolicy.initialDelay * (attempt + 1);
        break;
      case 'exponential':
        delay = retryPolicy.initialDelay * Math.pow(2, attempt);
        break;
      case 'fixed':
        delay = retryPolicy.initialDelay;
        break;
      default:
        delay = retryPolicy.initialDelay;
    }
    
    return Math.min(delay, retryPolicy.maxDelay);
  }
  
  private validateEvent(event: Event): void {
    if (!event.id || !event.type || !event.payload) {
      throw new Error('Invalid event: missing required fields');
    }
    
    if (event.metadata.retryCount > event.metadata.maxRetries) {
      throw new Error('Invalid event: retry count exceeds max retries');
    }
  }
  
  private addToHistory(event: Event): void {
    this.eventHistory.push(event);
    
    if (this.eventHistory.length > this.maxHistorySize) {
      this.eventHistory.shift();
    }
  }
  
  private moveToDeadLetterQueue(event: Event): void {
    this.deadLetterQueue.push(event);
    console.warn(`Event ${event.id} moved to dead letter queue`);
  }
  
  private updateMetrics(event: Event, status: 'success' | 'error'): void {
    this.metrics.totalEvents++;
    
    if (status === 'success') {
      this.metrics.processedEvents++;
    } else {
      this.metrics.failedEvents++;
    }
  }
  
  private updateProcessingTime(processingTime: number): void {
    const currentAvg = this.metrics.averageProcessingTime;
    const totalProcessed = this.metrics.processedEvents;
    
    this.metrics.averageProcessingTime = 
      (currentAvg * (totalProcessed - 1) + processingTime) / totalProcessed;
  }
  
  private generateHandlerId(): string {
    return `handler_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

interface EventMetrics {
  totalEvents: number;
  processedEvents: number;
  failedEvents: number;
  averageProcessingTime: number;
}
```

#### 2.2 Event Store

```typescript
// core/EventStore.ts
interface EventStore {
  appendEvents(streamId: string, events: Event[]): Promise<void>;
  getEvents(streamId: string, fromVersion?: number): Promise<Event[]>;
  getEventStreams(): Promise<string[]>;
  getStreamVersion(streamId: string): Promise<number>;
}

class InMemoryEventStore implements EventStore {
  private streams: Map<string, Event[]> = new Map();
  private streamVersions: Map<string, number> = new Map();
  
  async appendEvents(streamId: string, events: Event[]): Promise<void> {
    if (!this.streams.has(streamId)) {
      this.streams.set(streamId, []);
      this.streamVersions.set(streamId, 0);
    }
    
    const stream = this.streams.get(streamId)!;
    const currentVersion = this.streamVersions.get(streamId)!;
    
    // Verificar versión esperada
    if (events.length > 0 && events[0].metadata.version !== currentVersion.toString()) {
      throw new Error(`Concurrency conflict: expected version ${currentVersion}, got ${events[0].metadata.version}`);
    }
    
    // Agregar eventos al stream
    stream.push(...events);
    
    // Actualizar versión del stream
    this.streamVersions.set(streamId, currentVersion + events.length);
  }
  
  async getEvents(streamId: string, fromVersion: number = 0): Promise<Event[]> {
    const stream = this.streams.get(streamId);
    if (!stream) {
      return [];
    }
    
    return stream.slice(fromVersion);
  }
  
  async getEventStreams(): Promise<string[]> {
    return Array.from(this.streams.keys());
  }
  
  async getStreamVersion(streamId: string): Promise<number> {
    return this.streamVersions.get(streamId) || 0;
  }
}

// Event Store con persistencia
class PersistentEventStore implements EventStore {
  private storage: AsyncStorage;
  private inMemoryStore: InMemoryEventStore;
  
  constructor(storage: AsyncStorage) {
    this.storage = storage;
    this.inMemoryStore = new InMemoryEventStore();
    this.loadFromStorage();
  }
  
  async appendEvents(streamId: string, events: Event[]): Promise<void> {
    await this.inMemoryStore.appendEvents(streamId, events);
    await this.saveToStorage();
  }
  
  async getEvents(streamId: string, fromVersion: number = 0): Promise<Event[]> {
    return this.inMemoryStore.getEvents(streamId, fromVersion);
  }
  
  async getEventStreams(): Promise<string[]> {
    return this.inMemoryStore.getEventStreams();
  }
  
  async getStreamVersion(streamId: string): Promise<number> {
    return this.inMemoryStore.getStreamVersion(streamId);
  }
  
  private async loadFromStorage(): Promise<void> {
    try {
      const stored = await this.storage.getItem('event_store');
      if (stored) {
        const data = JSON.parse(stored);
        // Restaurar estado desde almacenamiento
        // Implementar lógica de restauración
      }
    } catch (error) {
      console.error('Failed to load event store from storage:', error);
    }
  }
  
  private async saveToStorage(): Promise<void> {
    try {
      const data = {
        streams: Array.from(this.inMemoryStore['streams'].entries()),
        versions: Array.from(this.inMemoryStore['streamVersions'].entries())
      };
      
      await this.storage.setItem('event_store', JSON.stringify(data));
    } catch (error) {
      console.error('Failed to save event store to storage:', error);
    }
  }
}
```

### 3. Patrón CQRS (Command Query Responsibility Segregation)

#### 3.1 Command and Query Handlers

```typescript
// core/CQRS/Command.ts
interface Command {
  id: string;
  type: string;
  payload: any;
  metadata: CommandMetadata;
  timestamp: Date;
}

interface CommandMetadata {
  userId: string;
  tenantId: string;
  correlationId: string;
  causationId?: string;
}

interface CommandHandler<T extends Command> {
  handle(command: T): Promise<void>;
}

// Comando para crear usuario
class CreateUserCommand implements Command {
  id: string;
  type = 'CreateUser';
  payload: {
    email: string;
    name: string;
    role: string;
  };
  metadata: CommandMetadata;
  timestamp: Date;
  
  constructor(payload: CreateUserCommand['payload'], metadata: CommandMetadata) {
    this.id = this.generateId();
    this.payload = payload;
    this.metadata = metadata;
    this.timestamp = new Date();
  }
  
  private generateId(): string {
    return `cmd_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Handler para crear usuario
class CreateUserCommandHandler implements CommandHandler<CreateUserCommand> {
  private eventBus: AdvancedEventBus;
  private userRepository: UserRepository;
  
  constructor(eventBus: AdvancedEventBus, userRepository: UserRepository) {
    this.eventBus = eventBus;
    this.userRepository = userRepository;
  }
  
  async handle(command: CreateUserCommand): Promise<void> {
    try {
      // Validar comando
      this.validateCommand(command);
      
      // Crear usuario
      const user = await this.userRepository.create({
        email: command.payload.email,
        name: command.payload.name,
        role: command.payload.role,
        tenantId: command.metadata.tenantId
      });
      
      // Publicar evento de usuario creado
      const userCreatedEvent: Event = {
        id: this.generateEventId(),
        type: 'UserCreated',
        payload: {
          userId: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
          tenantId: user.tenantId
        },
        metadata: {
          version: '1.0',
          schema: 'UserCreated',
          priority: 'normal',
          retryCount: 0,
          maxRetries: 3
        },
        timestamp: new Date(),
        source: 'CreateUserCommandHandler',
        correlationId: command.metadata.correlationId,
        causationId: command.id
      };
      
      await this.eventBus.publish(userCreatedEvent);
      
    } catch (error) {
      console.error('Failed to handle CreateUserCommand:', error);
      
      // Publicar evento de error
      const commandFailedEvent: Event = {
        id: this.generateEventId(),
        type: 'CommandFailed',
        payload: {
          commandId: command.id,
          commandType: command.type,
          error: error.message,
          userId: command.metadata.userId
        },
        metadata: {
          version: '1.0',
          schema: 'CommandFailed',
          priority: 'high',
          retryCount: 0,
          maxRetries: 3
        },
        timestamp: new Date(),
        source: 'CreateUserCommandHandler',
        correlationId: command.metadata.correlationId,
        causationId: command.id
      };
      
      await this.eventBus.publish(commandFailedEvent);
      
      throw error;
    }
  }
  
  private validateCommand(command: CreateUserCommand): void {
    if (!command.payload.email || !command.payload.name) {
      throw new Error('Email and name are required');
    }
    
    if (!command.metadata.userId || !command.metadata.tenantId) {
      throw new Error('User ID and tenant ID are required');
    }
  }
  
  private generateEventId(): string {
    return `evt_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Query para obtener usuarios
interface GetUsersQuery {
  tenantId: string;
  filters?: {
    role?: string;
    search?: string;
    limit?: number;
    offset?: number;
  };
}

interface GetUsersQueryResult {
  users: User[];
  total: number;
  hasMore: boolean;
}

// Handler para consultar usuarios
class GetUsersQueryHandler {
  private userRepository: UserRepository;
  
  constructor(userRepository: UserRepository) {
    this.userRepository = userRepository;
  }
  
  async handle(query: GetUsersQuery): Promise<GetUsersQueryResult> {
    const { tenantId, filters = {} } = query;
    
    // Construir filtros
    const whereClause: any = { tenantId };
    
    if (filters.role) {
      whereClause.role = filters.role;
    }
    
    if (filters.search) {
      whereClause.name = { $like: `%${filters.search}%` };
    }
    
    // Obtener usuarios
    const users = await this.userRepository.find(whereClause, {
      limit: filters.limit || 20,
      offset: filters.offset || 0,
      orderBy: { createdAt: 'DESC' }
    });
    
    // Obtener total
    const total = await this.userRepository.count(whereClause);
    
    return {
      users,
      total,
      hasMore: (filters.offset || 0) + (filters.limit || 20) < total
    };
  }
}
```

#### 3.2 Command Bus

```typescript
// core/CQRS/CommandBus.ts
interface CommandBus {
  execute<T extends Command>(command: T): Promise<void>;
  registerHandler<T extends Command>(commandType: string, handler: CommandHandler<T>): void;
}

class InMemoryCommandBus implements CommandBus {
  private handlers: Map<string, CommandHandler<any>> = new Map();
  
  async execute<T extends Command>(command: T): Promise<void> {
    const handler = this.handlers.get(command.type);
    
    if (!handler) {
      throw new Error(`No handler registered for command type: ${command.type}`);
    }
    
    await handler.handle(command);
  }
  
  registerHandler<T extends Command>(commandType: string, handler: CommandHandler<T>): void {
    this.handlers.set(commandType, handler);
  }
}

// Query Bus
interface QueryBus {
  execute<T>(query: any): Promise<T>;
  registerHandler<T>(queryType: string, handler: (query: any) => Promise<T>): void;
}

class InMemoryQueryBus implements QueryBus {
  private handlers: Map<string, (query: any) => Promise<any>> = new Map();
  
  async execute<T>(query: any): Promise<T> {
    const handler = this.handlers.get(query.constructor.name);
    
    if (!handler) {
      throw new Error(`No handler registered for query type: ${query.constructor.name}`);
    }
    
    return await handler(query);
  }
  
  registerHandler<T>(queryType: string, handler: (query: any) => Promise<T>): void {
    this.handlers.set(queryType, handler);
  }
}
```

### 4. Event Sourcing

#### 4.1 Event Sourced Aggregate

```typescript
// core/EventSourcing/EventSourcedAggregate.ts
abstract class EventSourcedAggregate {
  protected id: string;
  protected version: number = 0;
  protected uncommittedEvents: Event[] = [];
  
  constructor(id: string) {
    this.id = id;
  }
  
  get aggregateId(): string {
    return this.id;
  }
  
  get currentVersion(): number {
    return this.version;
  }
  
  get hasUncommittedEvents(): boolean {
    return this.uncommittedEvents.length > 0;
  }
  
  getUncommittedEvents(): Event[] {
    return [...this.uncommittedEvents];
  }
  
  markEventsAsCommitted(): void {
    this.uncommittedEvents = [];
  }
  
  loadFromHistory(events: Event[]): void {
    events.forEach(event => {
      this.applyEvent(event);
      this.version++;
    });
  }
  
  protected applyEvent(event: Event): void {
    const methodName = `on${event.type}`;
    const method = (this as any)[methodName];
    
    if (typeof method === 'function') {
      method.call(this, event);
    }
  }
  
  protected raiseEvent(event: Event): void {
    event.metadata.version = this.version.toString();
    this.uncommittedEvents.push(event);
    this.applyEvent(event);
  }
}

// Implementación concreta: User Aggregate
class UserAggregate extends EventSourcedAggregate {
  private email: string = '';
  private name: string = '';
  private role: string = '';
  private status: 'active' | 'inactive' = 'active';
  private tenantId: string = '';
  
  constructor(id: string) {
    super(id);
  }
  
  static create(id: string, email: string, name: string, role: string, tenantId: string): UserAggregate {
    const user = new UserAggregate(id);
    user.raiseEvent({
      id: user.generateEventId(),
      type: 'UserCreated',
      payload: { email, name, role, tenantId },
      metadata: {
        version: '1.0',
        schema: 'UserCreated',
        priority: 'normal',
        retryCount: 0,
        maxRetries: 3
      },
      timestamp: new Date(),
      source: 'UserAggregate'
    });
    
    return user;
  }
  
  updateProfile(name: string, role: string): void {
    if (this.status !== 'active') {
      throw new Error('Cannot update inactive user');
    }
    
    this.raiseEvent({
      id: this.generateEventId(),
      type: 'UserProfileUpdated',
      payload: { name, role },
      metadata: {
        version: '1.0',
        schema: 'UserProfileUpdated',
        priority: 'normal',
        retryCount: 0,
        maxRetries: 3
      },
      timestamp: new Date(),
      source: 'UserAggregate'
    });
  }
  
  deactivate(): void {
    if (this.status === 'inactive') {
      return; // Ya está inactivo
    }
    
    this.raiseEvent({
      id: this.generateEventId(),
      type: 'UserDeactivated',
      payload: { reason: 'User requested deactivation' },
      metadata: {
        version: '1.0',
        schema: 'UserDeactivated',
        priority: 'normal',
        retryCount: 0,
        maxRetries: 3
      },
      timestamp: new Date(),
      source: 'UserAggregate'
    });
  }
  
  // Event handlers
  private onUserCreated(event: Event): void {
    this.email = event.payload.email;
    this.name = event.payload.name;
    this.role = event.payload.role;
    this.tenantId = event.payload.tenantId;
    this.status = 'active';
  }
  
  private onUserProfileUpdated(event: Event): void {
    this.name = event.payload.name;
    this.role = event.payload.role;
  }
  
  private onUserDeactivated(event: Event): void {
    this.status = 'inactive';
  }
  
  private generateEventId(): string {
    return `evt_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  // Getters
  get userEmail(): string { return this.email; }
  get userName(): string { return this.name; }
  get userRole(): string { return this.role; }
  get userStatus(): string { return this.status; }
  get userTenantId(): string { return this.tenantId; }
}
```

#### 4.2 Event Store Repository

```typescript
// core/EventSourcing/EventStoreRepository.ts
abstract class EventStoreRepository<T extends EventSourcedAggregate> {
  protected eventStore: EventStore;
  
  constructor(eventStore: EventStore) {
    this.eventStore = eventStore;
  }
  
  async save(aggregate: T): Promise<void> {
    if (!aggregate.hasUncommittedEvents) {
      return; // No hay eventos para guardar
    }
    
    const events = aggregate.getUncommittedEvents();
    const streamId = this.getStreamId(aggregate);
    
    try {
      await this.eventStore.appendEvents(streamId, events);
      aggregate.markEventsAsCommitted();
    } catch (error) {
      console.error('Failed to save aggregate events:', error);
      throw error;
    }
  }
  
  async getById(id: string): Promise<T | null> {
    const streamId = this.getStreamId({ aggregateId: id } as T);
    const events = await this.eventStore.getEvents(streamId);
    
    if (events.length === 0) {
      return null;
    }
    
    const aggregate = this.createAggregate(id);
    aggregate.loadFromHistory(events);
    
    return aggregate;
  }
  
  protected abstract getStreamId(aggregate: T | { aggregateId: string }): string;
  protected abstract createAggregate(id: string): T;
}

// Implementación para User Aggregate
class UserEventStoreRepository extends EventStoreRepository<UserAggregate> {
  protected getStreamId(aggregate: UserAggregate | { aggregateId: string }): string {
    return `user-${aggregate.aggregateId}`;
  }
  
  protected createAggregate(id: string): UserAggregate {
    return new UserAggregate(id);
  }
}
```

### 5. Procesamiento de Eventos Asíncrono

#### 5.1 Event Processor

```typescript
// core/EventProcessing/EventProcessor.ts
interface EventProcessor {
  processEvent(event: Event): Promise<void>;
  getProcessorName(): string;
  getSupportedEventTypes(): string[];
}

class UserEventProcessor implements EventProcessor {
  private userRepository: UserRepository;
  private emailService: EmailService;
  private analyticsService: AnalyticsService;
  
  constructor(
    userRepository: UserRepository,
    emailService: EmailService,
    analyticsService: AnalyticsService
  ) {
    this.userRepository = userRepository;
    this.emailService = emailService;
    this.analyticsService = analyticsService;
  }
  
  getProcessorName(): string {
    return 'UserEventProcessor';
  }
  
  getSupportedEventTypes(): string[] {
    return ['UserCreated', 'UserProfileUpdated', 'UserDeactivated'];
  }
  
  async processEvent(event: Event): Promise<void> {
    switch (event.type) {
      case 'UserCreated':
        await this.handleUserCreated(event);
        break;
      case 'UserProfileUpdated':
        await this.handleUserProfileUpdated(event);
        break;
      case 'UserDeactivated':
        await this.handleUserDeactivated(event);
        break;
      default:
        console.warn(`Unsupported event type: ${event.type}`);
    }
  }
  
  private async handleUserCreated(event: Event): Promise<void> {
    try {
      // Actualizar vista de lectura
      await this.userRepository.create({
        id: event.payload.userId,
        email: event.payload.email,
        name: event.payload.name,
        role: event.payload.role,
        tenantId: event.payload.tenantId,
        status: 'active',
        createdAt: event.timestamp,
        updatedAt: event.timestamp
      });
      
      // Enviar email de bienvenida
      await this.emailService.sendWelcomeEmail(event.payload.email, event.payload.name);
      
      // Registrar en analytics
      await this.analyticsService.trackUserCreation({
        userId: event.payload.userId,
        tenantId: event.payload.tenantId,
        timestamp: event.timestamp
      });
      
    } catch (error) {
      console.error('Failed to handle UserCreated event:', error);
      throw error;
    }
  }
  
  private async handleUserProfileUpdated(event: Event): Promise<void> {
    try {
      // Actualizar vista de lectura
      await this.userRepository.update(event.payload.userId, {
        name: event.payload.name,
        role: event.payload.role,
        updatedAt: event.timestamp
      });
      
      // Registrar en analytics
      await this.analyticsService.trackProfileUpdate({
        userId: event.payload.userId,
        timestamp: event.timestamp
      });
      
    } catch (error) {
      console.error('Failed to handle UserProfileUpdated event:', error);
      throw error;
    }
  }
  
  private async handleUserDeactivated(event: Event): Promise<void> {
    try {
      // Actualizar vista de lectura
      await this.userRepository.update(event.payload.userId, {
        status: 'inactive',
        updatedAt: event.timestamp
      });
      
      // Registrar en analytics
      await this.analyticsService.trackUserDeactivation({
        userId: event.payload.userId,
        timestamp: event.timestamp
      });
      
    } catch (error) {
      console.error('Failed to handle UserDeactivated event:', error);
      throw error;
    }
  }
}

// Event Processing Pipeline
class EventProcessingPipeline {
  private eventBus: AdvancedEventBus;
  private processors: EventProcessor[] = [];
  private isProcessing: boolean = false;
  
  constructor(eventBus: AdvancedEventBus) {
    this.eventBus = eventBus;
  }
  
  addProcessor(processor: EventProcessor): void {
    this.processors.push(processor);
  }
  
  async start(): Promise<void> {
    if (this.isProcessing) {
      return;
    }
    
    this.isProcessing = true;
    
    // Suscribirse a todos los tipos de eventos soportados
    const supportedEventTypes = this.getAllSupportedEventTypes();
    
    for (const eventType of supportedEventTypes) {
      this.eventBus.subscribe(eventType, async (event: Event) => {
        await this.processEvent(event);
      });
    }
    
    console.log('Event processing pipeline started');
  }
  
  async stop(): Promise<void> {
    this.isProcessing = false;
    console.log('Event processing pipeline stopped');
  }
  
  private async processEvent(event: Event): Promise<void> {
    const relevantProcessors = this.processors.filter(processor =>
      processor.getSupportedEventTypes().includes(event.type)
    );
    
    if (relevantProcessors.length === 0) {
      console.warn(`No processors found for event type: ${event.type}`);
      return;
    }
    
    // Procesar evento con todos los processors relevantes
    const processingPromises = relevantProcessors.map(processor =>
      this.processEventWithProcessor(event, processor)
    );
    
    await Promise.allSettled(processingPromises);
  }
  
  private async processEventWithProcessor(event: Event, processor: EventProcessor): Promise<void> {
    try {
      const startTime = Date.now();
      
      await processor.processEvent(event);
      
      const processingTime = Date.now() - startTime;
      console.log(`Event ${event.type} processed by ${processor.getProcessorName()} in ${processingTime}ms`);
      
    } catch (error) {
      console.error(`Failed to process event ${event.type} with ${processor.getProcessorName()}:`, error);
      throw error;
    }
  }
  
  private getAllSupportedEventTypes(): string[] {
    const allTypes = new Set<string>();
    
    this.processors.forEach(processor => {
      processor.getSupportedEventTypes().forEach(eventType => {
        allTypes.add(eventType);
      });
    });
    
    return Array.from(allTypes);
  }
}
```

### 6. Integración con React Native

#### 6.1 Event-Driven Components

```typescript
// components/EventDrivenComponent.tsx
export const EventDrivenUserManager: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const commandBus = useCommandBus();
  const queryBus = useQueryBus();
  const eventBus = useEventBus();
  
  useEffect(() => {
    loadUsers();
    
    // Suscribirse a eventos de usuario
    const unsubscribe = eventBus.subscribe('UserCreated', handleUserCreated);
    
    return unsubscribe;
  }, []);
  
  const loadUsers = async () => {
    try {
      setLoading(true);
      setError(null);
      
      const query = new GetUsersQuery('tenant-1', { limit: 50 });
      const result = await queryBus.execute(query);
      
      setUsers(result.users);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const handleUserCreated = async (event: Event) => {
    // Agregar nuevo usuario a la lista
    const newUser: User = {
      id: event.payload.userId,
      email: event.payload.email,
      name: event.payload.name,
      role: event.payload.role,
      status: 'active',
      tenantId: event.payload.tenantId,
      createdAt: event.timestamp,
      updatedAt: event.timestamp
    };
    
    setUsers(prevUsers => [newUser, ...prevUsers]);
  };
  
  const createUser = async (userData: { email: string; name: string; role: string }) => {
    try {
      const command = new CreateUserCommand(userData, {
        userId: 'current-user-id',
        tenantId: 'tenant-1',
        correlationId: 'correlation-id'
      });
      
      await commandBus.execute(command);
      
      // El evento se manejará automáticamente por el listener
      
    } catch (err) {
      setError(err.message);
    }
  };
  
  const updateUser = async (userId: string, updates: { name: string; role: string }) => {
    try {
      const command = new UpdateUserProfileCommand(userId, updates, {
        userId: 'current-user-id',
        tenantId: 'tenant-1',
        correlationId: 'correlation-id'
      });
      
      await commandBus.execute(command);
      
    } catch (err) {
      setError(err.message);
    }
  };
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <View>
      <Text>User Management</Text>
      
      <UserForm onSubmit={createUser} />
      
      <FlatList
        data={users}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <UserCard 
            user={item} 
            onUpdate={(updates) => updateUser(item.id, updates)}
          />
        )}
      />
    </View>
  );
};

// Hooks para CQRS
export const useCommandBus = (): CommandBus => {
  const context = useContext(CommandBusContext);
  if (!context) {
    throw new Error('useCommandBus must be used within CommandBusProvider');
  }
  return context;
};

export const useQueryBus = (): QueryBus => {
  const context = useContext(QueryBusContext);
  if (!context) {
    throw new Error('useQueryBus must be used within QueryBusProvider');
  }
  return context;
};

export const useEventBus = (): AdvancedEventBus => {
  const context = useContext(EventBusContext);
  if (!context) {
    throw new Error('useEventBus must be used within EventBusProvider');
  }
  return context;
};
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Implementar Sistema Event-Driven Básico

**Objetivo**: Crear un sistema básico de eventos con publicación y suscripción.

**Requisitos**:
1. **Implementar AdvancedEventBus** con manejo de errores y reintentos
2. **Crear sistema de eventos** para gestión de usuarios
3. **Implementar handlers** para diferentes tipos de eventos
4. **Crear componentes** que usen el sistema de eventos

**Código Base**:
```typescript
// Implementar desde cero
interface Event {
  id: string;
  type: string;
  payload: any;
  timestamp: Date;
}

class AdvancedEventBus {
  // Implementar bus de eventos avanzado
}

// Eventos de usuario
const UserEvents = {
  UserCreated: 'UserCreated',
  UserUpdated: 'UserUpdated',
  UserDeleted: 'UserDeleted'
};
```

### Ejercicio 2: Implementar CQRS

**Objetivo**: Crear un sistema CQRS completo con comandos y consultas separados.

**Requisitos**:
1. **CommandBus y QueryBus** para separación de responsabilidades
2. **Command handlers** para operaciones de escritura
3. **Query handlers** para operaciones de lectura
4. **Integración** con el sistema de eventos

### Ejercicio 3: Event Sourcing

**Objetivo**: Implementar un sistema de Event Sourcing para un agregado.

**Requisitos**:
1. **EventSourcedAggregate** para gestión de estado
2. **Event Store** para persistencia de eventos
3. **Event Processors** para proyecciones
4. **Reconstrucción** de estado desde eventos

---

## 📚 Recursos Adicionales

### Documentación
- [Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)
- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)

### Herramientas
- [EventStore](https://eventstore.com/)
- [Apache Kafka](https://kafka.apache.org/)
- [RabbitMQ](https://www.rabbitmq.com/)

### Próximos Pasos
- **Fin del Módulo 16**: Arquitecturas Empresariales
- **Fin del Curso**: React Native Completo

---

## ✅ Checklist de Completado

- [ ] Comprendí los principios de arquitecturas event-driven
- [ ] Implementé sistemas de mensajería y eventos
- [ ] Apliqué patrones CQRS y Event Sourcing
- [ ] Creé sistemas de procesamiento de eventos asíncrono
- [ ] Diseñé arquitecturas escalables basadas en eventos
- [ ] Implementé AdvancedEventBus con manejo de errores
- [ ] Creé sistema de Event Store para persistencia
- [ ] Implementé CommandBus y QueryBus para CQRS
- [ ] Creé EventSourcedAggregate para gestión de estado
- [ ] Implementé Event Processing Pipeline
- [ ] Integré sistema event-driven con React Native
- [ ] Completé los ejercicios prácticos de event-driven

---

**🎯 ¡Felicidades!**: Has completado exitosamente el **Módulo 16: Arquitecturas Empresariales** y con él todo el curso de React Native. Ahora tienes conocimientos avanzados en arquitecturas empresariales, micro-frontends, feature flags, multi-tenancy y sistemas event-driven.

**🏆 Resumen del Módulo 16**:
- **Clase 1**: Fundamentos de Arquitecturas Empresariales
- **Clase 2**: Micro-Frontends y Modularización  
- **Clase 3**: Feature Flags y Configuración Dinámica
- **Clase 4**: Arquitecturas Multi-Tenant
- **Clase 5**: Sistemas Event-Driven y Patrones Avanzados

**🚀 Próximos Pasos Recomendados**:
1. **Practicar** implementando los ejercicios de cada clase
2. **Construir** aplicaciones usando los patrones aprendidos
3. **Explorar** frameworks y herramientas adicionales
4. **Contribuir** a proyectos open source
5. **Mentorear** a otros desarrolladores

**📚 Recursos para Continuar Aprendiendo**:
- Patrones de diseño avanzados
- Arquitecturas de microservicios
- DevOps y CI/CD
- Testing de arquitectura
- Performance y optimización
- Seguridad en aplicaciones móviles

¡Has completado un curso completo y avanzado de React Native! 🎉
