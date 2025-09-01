# Clase 1: Fundamentos de Patrones de Diseño

## Objetivos de la Clase
- Comprender qué son los patrones de diseño y por qué son importantes
- Aprender los principios SOLID como base para patrones efectivos
- Identificar cuándo y cómo aplicar patrones de diseño
- Analizar ejemplos de patrones en código existente

## Duración
**1.5 horas** (90 minutos)

## Contenido Teórico

### ¿Qué son los Patrones de Diseño?

Los patrones de diseño son **soluciones reutilizables** para problemas comunes que surgen durante el desarrollo de software. Son como "recetas" que han sido probadas y refinadas por la comunidad de desarrolladores.

```typescript
// ❌ Sin patrón - Código acoplado y difícil de mantener
class UserService {
  async getUser(id: string) {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    
    // Lógica de negocio mezclada con acceso a datos
    if (user.role === 'admin') {
      user.permissions = ['read', 'write', 'delete'];
    }
    
    return user;
  }
}

// ✅ Con patrón - Separación de responsabilidades
interface IUserRepository {
  findById(id: string): Promise<User>;
}

interface IUserBusinessLogic {
  enrichUserPermissions(user: User): User;
}

class UserService {
  constructor(
    private userRepo: IUserRepository,
    private businessLogic: IUserBusinessLogic
  ) {}
  
  async getUser(id: string): Promise<User> {
    const user = await this.userRepo.findById(id);
    return this.businessLogic.enrichUserPermissions(user);
  }
}
```

### Principios SOLID

Los principios SOLID son la base para crear patrones de diseño efectivos:

#### 1. **S**ingle Responsibility Principle (SRP)
```typescript
// ❌ Una clase con múltiples responsabilidades
class UserManager {
  async createUser(userData: UserData) { /* ... */ }
  async validateEmail(email: string) { /* ... */ }
  async sendWelcomeEmail(user: User) { /* ... */ }
  async saveToDatabase(user: User) { /* ... */ }
}

// ✅ Clases con responsabilidades únicas
class UserCreator {
  async createUser(userData: UserData): Promise<User> { /* ... */ }
}

class EmailValidator {
  async validateEmail(email: string): Promise<boolean> { /* ... */ }
}

class EmailService {
  async sendWelcomeEmail(user: User): Promise<void> { /* ... */ }
}

class UserRepository {
  async save(user: User): Promise<void> { /* ... */ }
}
```

#### 2. **O**pen/Closed Principle (OCP)
```typescript
// ❌ Cerrado para extensión, abierto para modificación
class PaymentProcessor {
  processPayment(payment: Payment) {
    if (payment.type === 'credit') {
      // Lógica específica para tarjeta de crédito
    } else if (payment.type === 'debit') {
      // Lógica específica para débito
    }
    // Cada nuevo tipo requiere modificar esta clase
  }
}

// ✅ Abierto para extensión, cerrado para modificación
interface IPaymentStrategy {
  process(payment: Payment): Promise<PaymentResult>;
}

class CreditCardStrategy implements IPaymentStrategy {
  async process(payment: Payment): Promise<PaymentResult> {
    // Lógica específica para tarjeta de crédito
  }
}

class DebitCardStrategy implements IPaymentStrategy {
  async process(payment: Payment): Promise<PaymentResult> {
    // Lógica específica para débito
  }
}

class PaymentProcessor {
  constructor(private strategies: Map<string, IPaymentStrategy>) {}
  
  processPayment(payment: Payment): Promise<PaymentResult> {
    const strategy = this.strategies.get(payment.type);
    if (!strategy) {
      throw new Error(`Unsupported payment type: ${payment.type}`);
    }
    return strategy.process(payment);
  }
}
```

#### 3. **L**iskov Substitution Principle (LSP)
```typescript
// ❌ Violación del LSP
class Bird {
  fly(): void {
    console.log('Flying...');
  }
}

class Penguin extends Bird {
  fly(): void {
    throw new Error("Penguins can't fly!");
  }
}

// ✅ Cumpliendo el LSP
interface IFlyable {
  fly(): void;
}

class Bird {
  // Comportamiento base para aves
}

class Sparrow extends Bird implements IFlyable {
  fly(): void {
    console.log('Flying...');
  }
}

class Penguin extends Bird {
  // Pingüinos no implementan IFlyable
  swim(): void {
    console.log('Swimming...');
  }
}
```

#### 4. **I**nterface Segregation Principle (ISP)
```typescript
// ❌ Interfaz monolítica
interface IWorker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// ✅ Interfaces específicas
interface IWorkable {
  work(): void;
}

interface IEatable {
  eat(): void;
}

interface ISleepable {
  sleep(): void;
}

class Human implements IWorkable, IEatable, ISleepable {
  work(): void { /* ... */ }
  eat(): void { /* ... */ }
  sleep(): void { /* ... */ }
}

class Robot implements IWorkable {
  work(): void { /* ... */ }
  // No necesita eat() ni sleep()
}
```

#### 5. **D**ependency Inversion Principle (DIP)
```typescript
// ❌ Dependencias concretas
class UserService {
  private userRepository = new MySQLUserRepository();
  
  async getUser(id: string) {
    return this.userRepository.findById(id);
  }
}

// ✅ Dependencias abstractas
interface IUserRepository {
  findById(id: string): Promise<User>;
}

class UserService {
  constructor(private userRepository: IUserRepository) {}
  
  async getUser(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}

// Inyección de dependencias
const mysqlRepo = new MySQLUserRepository();
const userService = new UserService(mysqlRepo);
```

### Categorías de Patrones de Diseño

#### 1. **Patrones Creacionales**
- **Singleton**: Garantiza una única instancia de una clase
- **Factory**: Crea objetos sin especificar su clase exacta
- **Builder**: Construye objetos complejos paso a paso

#### 2. **Patrones Estructurales**
- **Adapter**: Permite que interfaces incompatibles trabajen juntas
- **Decorator**: Añade funcionalidad dinámicamente
- **Facade**: Proporciona una interfaz simplificada a un subsistema complejo

#### 3. **Patrones de Comportamiento**
- **Observer**: Notifica a objetos cuando cambia el estado
- **Strategy**: Define una familia de algoritmos intercambiables
- **Command**: Encapsula una solicitud como un objeto

### Cuándo Usar Patrones de Diseño

#### ✅ **Usar patrones cuando:**
- El código se repite en múltiples lugares
- Hay cambios frecuentes en los requisitos
- Necesitas hacer el código más testeable
- Quieres mejorar la legibilidad y mantenibilidad

#### ❌ **NO usar patrones cuando:**
- El problema es simple y directo
- Agregas complejidad innecesaria
- No entiendes completamente el patrón
- Solo quieres "parecer profesional"

### Ejemplo Práctico: Análisis de Código

Vamos a analizar un componente React Native y aplicar principios SOLID:

```typescript
// ❌ Componente monolítico
const UserProfile = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  if (!user) return <Text>Usuario no encontrado</Text>;
  
  return (
    <View>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
      <Text>{user.role}</Text>
    </View>
  );
};
```

**Problemas identificados:**
1. **SRP**: El componente maneja fetching, estado, y renderizado
2. **OCP**: Difícil de extender sin modificar
3. **DIP**: Depende directamente de `fetch`

**Refactorización aplicando patrones:**

```typescript
// ✅ Separación de responsabilidades con patrones

// Hook personalizado para datos (Patrón Custom Hook)
const useUser = (userId: string) => {
  const [state, setState] = useState<{
    user: User | null;
    loading: boolean;
    error: string | null;
  }>({
    user: null,
    loading: false,
    error: null
  });
  
  useEffect(() => {
    const fetchUser = async () => {
      setState(prev => ({ ...prev, loading: true }));
      try {
        const userData = await userService.getUser(userId);
        setState({ user: userData, loading: false, error: null });
      } catch (err) {
        setState({ user: null, loading: false, error: err.message });
      }
    };
    
    fetchUser();
  }, [userId]);
  
  return state;
};

// Componente de presentación (Patrón Presentational Component)
const UserProfileView = ({ user }: { user: User }) => (
  <View>
    <Text>{user.name}</Text>
    <Text>{user.email}</Text>
    <Text>{user.role}</Text>
  </View>
);

// Componente contenedor (Patrón Container Component)
const UserProfile = ({ userId }: { userId: string }) => {
  const { user, loading, error } = useUser(userId);
  
  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  if (!user) return <Text>Usuario no encontrado</Text>;
  
  return <UserProfileView user={user} />;
};
```

## Ejercicios Prácticos

### Ejercicio 1: Análisis de Principios SOLID
Analiza el siguiente código e identifica qué principios SOLID se violan:

```typescript
class DataManager {
  async fetchData() {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  }
  
  async saveData(data: any) {
    const response = await fetch('/api/data', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    return response.json();
  }
  
  validateData(data: any) {
    // Validación compleja
    return data && data.id && data.name;
  }
  
  formatData(data: any) {
    // Formateo de datos
    return {
      ...data,
      createdAt: new Date().toISOString()
    };
  }
}
```

**Tareas:**
1. Identifica qué principios SOLID se violan
2. Refactoriza el código aplicando los principios
3. Explica cómo cada cambio mejora el código

### Ejercicio 2: Identificación de Patrones
Revisa tu código React Native existente y:

1. **Identifica** 3-5 lugares donde podrías aplicar patrones
2. **Documenta** qué patrón sería apropiado y por qué
3. **Implementa** al menos uno de los patrones identificados

### Ejercicio 3: Creación de Interfaces
Crea interfaces para un sistema de notificaciones que cumpla con ISP:

```typescript
// Define interfaces para:
// - Notificaciones push
// - Notificaciones por email
// - Notificaciones por SMS
// - Sistema de logging
```

## Resumen de la Clase

### Conceptos Clave Aprendidos:
- ✅ **Patrones de diseño** son soluciones reutilizables para problemas comunes
- ✅ **Principios SOLID** son la base para patrones efectivos
- ✅ **Separación de responsabilidades** mejora la mantenibilidad
- ✅ **Interfaces específicas** son mejores que interfaces monolíticas
- ✅ **Dependencias abstractas** hacen el código más testeable

### Próximos Pasos:
- Implementar patrones creacionales en la siguiente clase
- Practicar la refactorización de código existente
- Aplicar principios SOLID en componentes React Native

### Recursos Adicionales:
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350884)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
- [React Design Patterns](https://reactpatterns.com/)

---

## Navegación
- **Anterior**: [Módulo 7: Testing y Debugging](../midLevel_4/README.md)
- **Siguiente**: [Clase 2: Patrones Creacionales](clase_2_patrones_creacionales.md)
- **README del Módulo**: [Módulo 8: Patrones de Diseño](README.md)
- **Inicio**: [Índice Completo](../../INDICE_COMPLETO.md)
