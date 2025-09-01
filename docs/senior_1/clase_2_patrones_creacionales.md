# Clase 2: Patrones Creacionales

## Objetivos de la Clase
- Implementar el patrón Singleton para servicios globales
- Crear factories para la generación de componentes y objetos
- Utilizar el patrón Builder para objetos complejos
- Aplicar Abstract Factory para familias de objetos relacionados

## Duración
**2 horas** (120 minutos)

## Contenido Teórico

### Patrón Singleton

El patrón Singleton garantiza que una clase tenga **una única instancia** y proporciona un punto de acceso global a ella.

#### Implementación Básica
```typescript
// ❌ Implementación incorrecta - No es thread-safe
class DatabaseConnection {
  private static instance: DatabaseConnection;
  
  private constructor() {}
  
  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
}

// ✅ Implementación correcta con lazy initialization
class DatabaseConnection {
  private static instance: DatabaseConnection | null = null;
  private static readonly lock = new Object();
  
  private constructor() {
    // Inicialización de la conexión
    console.log('Database connection initialized');
  }
  
  static getInstance(): DatabaseConnection {
    if (DatabaseConnection.instance === null) {
      synchronized(DatabaseConnection.lock, () => {
        if (DatabaseConnection.instance === null) {
          DatabaseConnection.instance = new DatabaseConnection();
        }
      });
    }
    return DatabaseConnection.instance;
  }
  
  query(sql: string): Promise<any> {
    console.log(`Executing query: ${sql}`);
    // Implementación de la consulta
    return Promise.resolve({ result: 'data' });
  }
  
  close(): void {
    console.log('Database connection closed');
  }
}

// Uso del Singleton
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();

console.log(db1 === db2); // true - Misma instancia
```

#### Singleton en React Native
```typescript
// Singleton para configuración de la app
class AppConfig {
  private static instance: AppConfig | null = null;
  
  private constructor() {}
  
  static getInstance(): AppConfig {
    if (!AppConfig.instance) {
      AppConfig.instance = new AppConfig();
    }
    return AppConfig.instance;
  }
  
  // Configuración de la aplicación
  readonly API_BASE_URL = 'https://api.miapp.com';
  readonly APP_VERSION = '1.0.0';
  readonly DEBUG_MODE = __DEV__;
  
  // Métodos de configuración
  setApiUrl(url: string): void {
    (this as any).API_BASE_URL = url;
  }
  
  getApiUrl(): string {
    return this.API_BASE_URL;
  }
}

// Hook personalizado para usar la configuración
const useAppConfig = () => {
  const config = useMemo(() => AppConfig.getInstance(), []);
  return config;
};

// Uso en componentes
const ApiService = () => {
  const config = useAppConfig();
  
  const fetchData = async () => {
    const response = await fetch(`${config.getApiUrl()}/users`);
    return response.json();
  };
  
  return { fetchData };
};
```

### Patrón Factory Method

El Factory Method define una interfaz para crear objetos, pero permite a las subclases decidir qué clase instanciar.

#### Implementación Básica
```typescript
// Interfaz para productos
interface IButton {
  render(): string;
  onClick(): void;
}

// Productos concretos
class AndroidButton implements IButton {
  render(): string {
    return 'Android Button';
  }
  
  onClick(): void {
    console.log('Android button clicked');
  }
}

class IOSButton implements IButton {
  render(): string {
    return 'iOS Button';
  }
  
  onClick(): void {
    console.log('iOS button clicked');
  }
}

// Creator abstracto
abstract class ButtonCreator {
  abstract createButton(): IButton;
  
  renderButton(): string {
    const button = this.createButton();
    return button.render();
  }
}

// Creators concretos
class AndroidButtonCreator extends ButtonCreator {
  createButton(): IButton {
    return new AndroidButton();
  }
}

class IOSButtonCreator extends ButtonCreator {
  createButton(): IButton {
    return new IOSButton();
  }
}

// Uso del Factory Method
const createButtonForPlatform = (platform: 'android' | 'ios'): IButton => {
  switch (platform) {
    case 'android':
      return new AndroidButtonCreator().createButton();
    case 'ios':
      return new IOSButtonCreator().createButton();
    default:
      throw new Error(`Unsupported platform: ${platform}`);
  }
};
```

#### Factory Method en React Native
```typescript
// Factory para componentes de navegación según plataforma
interface INavigationComponent {
  render(): React.ReactElement;
}

class AndroidNavigation implements INavigationComponent {
  render(): React.ReactElement {
    return (
      <View style={styles.androidNav}>
        <TouchableOpacity style={styles.androidButton}>
          <Text>Android Navigation</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

class IOSNavigation implements INavigationComponent {
  render(): React.ReactElement {
    return (
      <View style={styles.iosNav}>
        <TouchableOpacity style={styles.iosButton}>
          <Text>iOS Navigation</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

// Factory para crear navegación según plataforma
class NavigationFactory {
  static createNavigation(): INavigationComponent {
    if (Platform.OS === 'android') {
      return new AndroidNavigation();
    } else {
      return new IOSNavigation();
    }
  }
}

// Componente que usa el factory
const PlatformSpecificNavigation = () => {
  const navigation = useMemo(() => NavigationFactory.createNavigation(), []);
  
  return navigation.render();
};
```

### Patrón Abstract Factory

El Abstract Factory proporciona una interfaz para crear **familias de objetos relacionados** sin especificar sus clases concretas.

#### Implementación Básica
```typescript
// Interfaces para familias de productos
interface IButton {
  render(): string;
}

interface ICheckbox {
  render(): string;
}

interface ITextField {
  render(): string;
}

// Familia Android
class AndroidButton implements IButton {
  render(): string {
    return 'Android Button';
  }
}

class AndroidCheckbox implements ICheckbox {
  render(): string {
    return 'Android Checkbox';
  }
}

class AndroidTextField implements ITextField {
  render(): string {
    return 'Android TextField';
  }
}

// Familia iOS
class IOSButton implements IButton {
  render(): string {
    return 'iOS Button';
  }
}

class IOSCheckbox implements ICheckbox {
  render(): string {
    return 'iOS Checkbox';
  }
}

class IOSTextField implements ITextField {
  render(): string {
    return 'iOS TextField';
  }
}

// Abstract Factory
interface IUIFactory {
  createButton(): IButton;
  createCheckbox(): ICheckbox;
  createTextField(): ITextField;
}

// Factories concretos
class AndroidUIFactory implements IUIFactory {
  createButton(): IButton {
    return new AndroidButton();
  }
  
  createCheckbox(): ICheckbox {
    return new AndroidCheckbox();
  }
  
  createTextField(): ITextField {
    return new AndroidTextField();
  }
}

class IOSUIFactory implements IUIFactory {
  createButton(): IButton {
    return new IOSButton();
  }
  
  createCheckbox(): ICheckbox {
    return new IOSCheckbox();
  }
  
  createTextField(): ITextField {
    return new IOSTextField();
  }
}

// Factory selector
class UIFactorySelector {
  static createFactory(platform: 'android' | 'ios'): IUIFactory {
    switch (platform) {
      case 'android':
        return new AndroidUIFactory();
      case 'ios':
        return new IOSUIFactory();
      default:
        throw new Error(`Unsupported platform: ${platform}`);
    }
  }
}
```

#### Abstract Factory en React Native
```typescript
// Factory para temas de la aplicación
interface ITheme {
  colors: {
    primary: string;
    secondary: string;
    background: string;
    text: string;
  };
  spacing: {
    small: number;
    medium: number;
    large: number;
  };
  typography: {
    fontSize: {
      small: number;
      medium: number;
      large: number;
    };
  };
}

class LightTheme implements ITheme {
  colors = {
    primary: '#007AFF',
    secondary: '#5856D6',
    background: '#FFFFFF',
    text: '#000000'
  };
  
  spacing = {
    small: 8,
    medium: 16,
    large: 24
  };
  
  typography = {
    fontSize: {
      small: 12,
      medium: 16,
      large: 20
    }
  };
}

class DarkTheme implements ITheme {
  colors = {
    primary: '#0A84FF',
    secondary: '#5E5CE6',
    background: '#000000',
    text: '#FFFFFF'
  };
  
  spacing = {
    small: 8,
    medium: 16,
    large: 24
  };
  
  typography = {
    fontSize: {
      small: 12,
      medium: 16,
      large: 20
    }
  };
}

// Factory para temas
class ThemeFactory {
  static createTheme(type: 'light' | 'dark'): ITheme {
    switch (type) {
      case 'light':
        return new LightTheme();
      case 'dark':
        return new DarkTheme();
      default:
        throw new Error(`Unsupported theme: ${type}`);
    }
  }
}

// Hook para usar el tema
const useTheme = (type: 'light' | 'dark') => {
  return useMemo(() => ThemeFactory.createTheme(type), [type]);
};

// Componente que usa el tema
const ThemedButton = ({ title, onPress }: { title: string; onPress: () => void }) => {
  const theme = useTheme('light'); // O detectar automáticamente
  
  return (
    <TouchableOpacity
      style={{
        backgroundColor: theme.colors.primary,
        padding: theme.spacing.medium,
        borderRadius: 8
      }}
      onPress={onPress}
    >
      <Text style={{
        color: theme.colors.background,
        fontSize: theme.typography.fontSize.medium
      }}>
        {title}
      </Text>
    </TouchableOpacity>
  );
};
```

### Patrón Builder

El patrón Builder permite construir objetos complejos **paso a paso**, separando la construcción de su representación.

#### Implementación Básica
```typescript
// Producto complejo
class UserProfile {
  constructor(
    public name: string = '',
    public email: string = '',
    public age: number = 0,
    public bio: string = '',
    public avatar: string = '',
    public preferences: string[] = []
  ) {}
  
  toString(): string {
    return `UserProfile(name=${this.name}, email=${this.email}, age=${this.age})`;
  }
}

// Builder abstracto
interface IUserProfileBuilder {
  setName(name: string): IUserProfileBuilder;
  setEmail(email: string): IUserProfileBuilder;
  setAge(age: number): IUserProfileBuilder;
  setBio(bio: string): IUserProfileBuilder;
  setAvatar(avatar: string): IUserProfileBuilder;
  setPreferences(preferences: string[]): IUserProfileBuilder;
  build(): UserProfile;
}

// Builder concreto
class UserProfileBuilder implements IUserProfileBuilder {
  private userProfile: UserProfile = new UserProfile();
  
  setName(name: string): IUserProfileBuilder {
    this.userProfile.name = name;
    return this;
  }
  
  setEmail(email: string): IUserProfileBuilder {
    this.userProfile.email = email;
    return this;
  }
  
  setAge(age: number): IUserProfileBuilder {
    this.userProfile.age = age;
    return this;
  }
  
  setBio(bio: string): IUserProfileBuilder {
    this.userProfile.bio = bio;
    return this;
  }
  
  setAvatar(avatar: string): IUserProfileBuilder {
    this.userProfile.avatar = avatar;
    return this;
  }
  
  setPreferences(preferences: string[]): IUserProfileBuilder {
    this.userProfile.preferences = preferences;
    return this;
  }
  
  build(): UserProfile {
    // Validaciones antes de construir
    if (!this.userProfile.name) {
      throw new Error('Name is required');
    }
    if (!this.userProfile.email) {
      throw new Error('Email is required');
    }
    
    const result = this.userProfile;
    this.userProfile = new UserProfile(); // Reset para reutilización
    return result;
  }
}

// Director (opcional)
class UserProfileDirector {
  constructor(private builder: IUserProfileBuilder) {}
  
  createMinimalProfile(name: string, email: string): UserProfile {
    return this.builder
      .setName(name)
      .setEmail(email)
      .build();
  }
  
  createFullProfile(
    name: string,
    email: string,
    age: number,
    bio: string,
    avatar: string,
    preferences: string[]
  ): UserProfile {
    return this.builder
      .setName(name)
      .setEmail(email)
      .setAge(age)
      .setBio(bio)
      .setAvatar(avatar)
      .setPreferences(preferences)
      .build();
  }
}

// Uso del Builder
const builder = new UserProfileBuilder();
const director = new UserProfileDirector(builder);

// Construcción directa
const user1 = builder
  .setName('John Doe')
  .setEmail('john@example.com')
  .setAge(30)
  .build();

// Construcción con director
const user2 = director.createMinimalProfile('Jane Doe', 'jane@example.com');
const user3 = director.createFullProfile(
  'Bob Smith',
  'bob@example.com',
  25,
  'Software Developer',
  'avatar.jpg',
  ['coding', 'reading', 'music']
);
```

#### Builder en React Native
```typescript
// Builder para estilos de componentes
class StyleBuilder {
  private styles: any = {};
  
  backgroundColor(color: string): StyleBuilder {
    this.styles.backgroundColor = color;
    return this;
  }
  
  padding(value: number): StyleBuilder {
    this.styles.padding = value;
    return this;
  }
  
  margin(value: number): StyleBuilder {
    this.styles.margin = value;
    return this;
  }
  
  borderRadius(value: number): StyleBuilder {
    this.styles.borderRadius = value;
    return this;
  }
  
  shadow(): StyleBuilder {
    this.styles.shadowColor = '#000';
    this.styles.shadowOffset = { width: 0, height: 2 };
    this.styles.shadowOpacity = 0.25;
    this.styles.shadowRadius = 3.84;
    this.styles.elevation = 5;
    return this;
  }
  
  build(): any {
    const result = { ...this.styles };
    this.styles = {};
    return result;
  }
}

// Uso del StyleBuilder
const useStyles = () => {
  const primaryButtonStyle = useMemo(() => 
    new StyleBuilder()
      .backgroundColor('#007AFF')
      .padding(16)
      .borderRadius(8)
      .shadow()
      .build(),
    []
  );
  
  const secondaryButtonStyle = useMemo(() => 
    new StyleBuilder()
      .backgroundColor('#5856D6')
      .padding(12)
      .borderRadius(6)
      .build(),
    []
  );
  
  return { primaryButtonStyle, secondaryButtonStyle };
};

// Componente que usa los estilos
const StyledButton = ({ title, onPress, variant = 'primary' }: {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
}) => {
  const styles = useStyles();
  
  const buttonStyle = variant === 'primary' 
    ? styles.primaryButtonStyle 
    : styles.secondaryButtonStyle;
  
  return (
    <TouchableOpacity style={buttonStyle} onPress={onPress}>
      <Text style={{ color: 'white', textAlign: 'center' }}>
        {title}
      </Text>
    </TouchableOpacity>
  );
};
```

## Ejercicios Prácticos

### Ejercicio 1: Implementación de Singleton
Crea un Singleton para un servicio de notificaciones:

```typescript
// Implementa NotificationService como Singleton con:
// - Método para mostrar notificaciones
// - Método para configurar el tipo de notificación
// - Método para limpiar notificaciones
// - Hook personalizado para usar el servicio
```

### Ejercicio 2: Factory para Componentes
Crea un Factory para generar diferentes tipos de botones:

```typescript
// Implementa ButtonFactory que pueda crear:
// - Botones primarios
// - Botones secundarios
// - Botones de texto
// - Botones con iconos
// Cada tipo debe tener estilos y comportamientos diferentes
```

### Ejercicio 3: Builder para Formularios
Crea un Builder para construir formularios dinámicamente:

```typescript
// Implementa FormBuilder que permita:
// - Agregar campos de texto
// - Agregar campos de selección
// - Agregar validaciones
// - Configurar el layout
// - Generar el formulario final
```

## Resumen de la Clase

### Conceptos Clave Aprendidos:
- ✅ **Singleton** garantiza una única instancia global
- ✅ **Factory Method** delega la creación a subclases
- ✅ **Abstract Factory** crea familias de objetos relacionados
- ✅ **Builder** construye objetos complejos paso a paso
- ✅ **Patrones creacionales** mejoran la flexibilidad del código

### Próximos Pasos:
- Implementar patrones estructurales en la siguiente clase
- Aplicar patrones creacionales en componentes React Native
- Practicar la refactorización de código existente

### Recursos Adicionales:
- [Design Patterns in TypeScript](https://refactoring.guru/design-patterns/typescript)
- [React Native Design Patterns](https://reactnative.dev/docs/performance)
- [TypeScript Design Patterns](https://www.typescriptlang.org/docs/)

---

## Navegación
- **Anterior**: [Clase 1: Fundamentos de Patrones de Diseño](clase_1_fundamentos_patrones.md)
- **Siguiente**: [Clase 3: Patrones Estructurales](clase_3_patrones_estructurales.md)
- **README del Módulo**: [Módulo 8: Patrones de Diseño](README.md)
- **Inicio**: [Índice Completo](../../INDICE_COMPLETO.md)
