# Clase 4: Arquitecturas Multi-Tenant 🏢

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender** el concepto de arquitecturas multi-tenant y sus patrones
2. **Implementar** sistemas multi-tenant con aislamiento de datos
3. **Diseñar** configuraciones por cliente y personalización
4. **Crear** sistemas de routing y navegación multi-tenant
5. **Aplicar** patrones de seguridad y escalabilidad multi-tenant

## ⏱️ Duración Estimada

**2 horas** - Teoría: 1h | Práctica: 1h

## 🔗 Navegación

- **📚 Módulo**: [Módulo 16: Arquitecturas Empresariales](../README.md)
- **⬅️ Anterior**: [Clase 3: Feature Flags y Configuración Dinámica](clase_3_feature_flags_configuracion_dinamica.md)
- **➡️ Siguiente**: [Clase 5: Sistemas Event-Driven y Patrones Avanzados](clase_5_sistemas_event_driven_patrones_avanzados.md)
- **🏠 Inicio**: [Índice del Curso](../../../INDICE_COMPLETO.md)

---

## 🎯 Contenido Teórico

### 1. ¿Qué son las Arquitecturas Multi-Tenant?

Una **arquitectura multi-tenant** es un patrón de diseño donde una sola instancia de una aplicación sirve a múltiples clientes (tenants), cada uno con sus propios datos, configuraciones y personalizaciones. En React Native, esto permite:

- **Escalabilidad** al servir múltiples clientes con una sola aplicación
- **Aislamiento** de datos entre diferentes tenants
- **Personalización** por cliente sin duplicar código
- **Mantenimiento** centralizado de la aplicación
- **Costos reducidos** por cliente

#### 1.1 Patrones de Multi-Tenancy

```typescript
// ✅ PATRONES DE MULTI-TENANCY:

// 1. Database per Tenant: Base de datos separada por cliente
const DATABASE_PER_TENANT = {
  tenant1: 'tenant1_db',
  tenant2: 'tenant2_db',
  tenant3: 'tenant3_db'
};

// 2. Shared Database, Separate Schemas: Una BD con esquemas separados
const SHARED_DB_SCHEMAS = {
  tenant1: 'tenant1_schema',
  tenant2: 'tenant2_schema',
  tenant3: 'tenant3_schema'
};

// 3. Shared Database, Shared Schema: Una BD con discriminador de tenant
const SHARED_DB_SHARED_SCHEMA = {
  discriminator: 'tenant_id',
  data: 'all_tenants_data'
};

// 4. Hybrid Approach: Combinación de patrones según necesidades
const HYBRID_APPROACH = {
  critical_data: 'database_per_tenant',
  shared_data: 'shared_schema',
  analytics: 'shared_database'
};
```

### 2. Core Multi-Tenant Service

#### 2.1 Tenant Management Service

```typescript
// core/TenantService.ts
interface Tenant {
  id: string;
  name: string;
  domain: string;
  subdomain: string;
  status: 'active' | 'inactive' | 'suspended';
  plan: 'basic' | 'premium' | 'enterprise';
  features: string[];
  configuration: TenantConfiguration;
  metadata: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}

interface TenantConfiguration {
  theme: ThemeConfig;
  branding: BrandingConfig;
  features: FeatureConfig;
  limits: LimitConfig;
  integrations: IntegrationConfig;
}

interface ThemeConfig {
  primaryColor: string;
  secondaryColor: string;
  fontFamily: string;
  logoUrl: string;
  customCSS?: string;
}

interface BrandingConfig {
  companyName: string;
  logo: string;
  favicon: string;
  colors: Record<string, string>;
  fonts: Record<string, string>;
}

class TenantService {
  private tenants: Map<string, Tenant> = new Map();
  private currentTenant: Tenant | null = null;
  private tenantResolver: TenantResolver;
  
  constructor(tenantResolver: TenantResolver) {
    this.tenantResolver = tenantResolver;
  }
  
  async initialize(): Promise<void> {
    // Resolver tenant actual basado en contexto
    const tenantId = await this.tenantResolver.resolveCurrentTenant();
    if (tenantId) {
      await this.setCurrentTenant(tenantId);
    }
  }
  
  async setCurrentTenant(tenantId: string): Promise<void> {
    const tenant = await this.getTenant(tenantId);
    if (!tenant) {
      throw new Error(`Tenant ${tenantId} not found`);
    }
    
    this.currentTenant = tenant;
    
    // Emitir evento de cambio de tenant
    this.emitTenantChanged(tenant);
  }
  
  getCurrentTenant(): Tenant | null {
    return this.currentTenant;
  }
  
  async getTenant(tenantId: string): Promise<Tenant | null> {
    // Verificar cache local
    if (this.tenants.has(tenantId)) {
      return this.tenants.get(tenantId)!;
    }
    
    // Cargar desde API
    try {
      const tenant = await this.fetchTenantFromAPI(tenantId);
      if (tenant) {
        this.tenants.set(tenantId, tenant);
      }
      return tenant;
    } catch (error) {
      console.error(`Failed to fetch tenant ${tenantId}:`, error);
      return null;
    }
  }
  
  async getAllTenants(): Promise<Tenant[]> {
    return Array.from(this.tenants.values());
  }
  
  async createTenant(tenantData: Partial<Tenant>): Promise<Tenant> {
    // Validar datos del tenant
    this.validateTenantData(tenantData);
    
    // Crear tenant en API
    const tenant = await this.createTenantInAPI(tenantData);
    
    // Agregar a cache local
    this.tenants.set(tenant.id, tenant);
    
    return tenant;
  }
  
  async updateTenant(tenantId: string, updates: Partial<Tenant>): Promise<Tenant> {
    const tenant = await this.getTenant(tenantId);
    if (!tenant) {
      throw new Error(`Tenant ${tenantId} not found`);
    }
    
    // Actualizar en API
    const updatedTenant = await this.updateTenantInAPI(tenantId, updates);
    
    // Actualizar cache local
    this.tenants.set(tenantId, updatedTenant);
    
    // Si es el tenant actual, actualizarlo
    if (this.currentTenant?.id === tenantId) {
      this.currentTenant = updatedTenant;
      this.emitTenantChanged(updatedTenant);
    }
    
    return updatedTenant;
  }
  
  private validateTenantData(tenantData: Partial<Tenant>): void {
    if (!tenantData.name || !tenantData.domain) {
      throw new Error('Tenant name and domain are required');
    }
    
    // Validar formato de dominio
    const domainRegex = /^[a-zA-Z0-9][a-zA-Z0-9-]{1,61}[a-zA-Z0-9]\.[a-zA-Z]{2,}$/;
    if (!domainRegex.test(tenantData.domain)) {
      throw new Error('Invalid domain format');
    }
  }
  
  private async fetchTenantFromAPI(tenantId: string): Promise<Tenant | null> {
    try {
      const response = await fetch(`/api/tenants/${tenantId}`, {
        headers: {
          'Authorization': `Bearer ${this.getAuthToken()}`,
          'Content-Type': 'application/json'
        }
      });
      
      if (!response.ok) {
        if (response.status === 404) {
          return null;
        }
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Failed to fetch tenant from API:', error);
      return null;
    }
  }
  
  private async createTenantInAPI(tenantData: Partial<Tenant>): Promise<Tenant> {
    const response = await fetch('/api/tenants', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.getAuthToken()}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(tenantData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    return await response.json();
  }
  
  private async updateTenantInAPI(tenantId: string, updates: Partial<Tenant>): Promise<Tenant> {
    const response = await fetch(`/api/tenants/${tenantId}`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${this.getAuthToken()}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(updates)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    return await response.json();
  }
  
  private emitTenantChanged(tenant: Tenant): void {
    // Emitir evento de cambio de tenant
    // Esto puede ser usado por otros servicios para reaccionar
    console.log(`Current tenant changed to: ${tenant.name}`);
  }
  
  private getAuthToken(): string {
    // Implementar lógica para obtener token de autenticación
    return 'your-auth-token';
  }
}
```

#### 2.2 Tenant Resolver

```typescript
// core/TenantResolver.ts
interface TenantResolver {
  resolveCurrentTenant(): Promise<string | null>;
}

// Resolver basado en subdominio
class SubdomainTenantResolver implements TenantResolver {
  async resolveCurrentTenant(): Promise<string | null> {
    // En React Native, esto podría ser basado en la configuración de la app
    // o en el entorno de desarrollo
    const currentSubdomain = this.getCurrentSubdomain();
    
    if (currentSubdomain) {
      return this.subdomainToTenantId(currentSubdomain);
    }
    
    return null;
  }
  
  private getCurrentSubdomain(): string | null {
    // En React Native, esto podría venir de:
    // - Configuración de la app
    // - Variables de entorno
    // - API de configuración
    // - Deep linking
    
    // Ejemplo: obtener de configuración
    return Config.get('TENANT_SUBDOMAIN') || null;
  }
  
  private subdomainToTenantId(subdomain: string): string {
    // Mapear subdominio a ID de tenant
    const subdomainMap: Record<string, string> = {
      'company1': 'tenant-company1',
      'company2': 'tenant-company2',
      'demo': 'tenant-demo'
    };
    
    return subdomainMap[subdomain] || subdomain;
  }
}

// Resolver basado en configuración de usuario
class UserConfigTenantResolver implements TenantResolver {
  async resolveCurrentTenant(): Promise<string | null> {
    // Obtener tenant del usuario autenticado
    const user = await this.getCurrentUser();
    if (user && user.tenantId) {
      return user.tenantId;
    }
    
    return null;
  }
  
  private async getCurrentUser(): Promise<any> {
    // Implementar lógica para obtener usuario actual
    // Esto podría venir de AsyncStorage, Redux, Context, etc.
    return null;
  }
}

// Resolver híbrido
class HybridTenantResolver implements TenantResolver {
  private resolvers: TenantResolver[];
  
  constructor(resolvers: TenantResolver[]) {
    this.resolvers = resolvers;
  }
  
  async resolveCurrentTenant(): Promise<string | null> {
    // Intentar cada resolver en orden
    for (const resolver of this.resolvers) {
      try {
        const tenantId = await resolver.resolveCurrentTenant();
        if (tenantId) {
          return tenantId;
        }
      } catch (error) {
        console.error('Resolver failed:', error);
      }
    }
    
    return null;
  }
}
```

### 3. Aislamiento de Datos

#### 3.1 Tenant-Aware Repository

```typescript
// repositories/TenantAwareRepository.ts
abstract class TenantAwareRepository<T> {
  protected abstract tableName: string;
  protected tenantService: TenantService;
  
  constructor(tenantService: TenantService) {
    this.tenantService = tenantService;
  }
  
  protected getCurrentTenantId(): string {
    const currentTenant = this.tenantService.getCurrentTenant();
    if (!currentTenant) {
      throw new Error('No current tenant set');
    }
    return currentTenant.id;
  }
  
  protected addTenantFilter(query: any): any {
    const tenantId = this.getCurrentTenantId();
    
    return {
      ...query,
      where: {
        ...query.where,
        tenantId
      }
    };
  }
  
  async findById(id: string): Promise<T | null> {
    const query = this.addTenantFilter({
      where: { id }
    });
    
    // Implementar lógica de búsqueda
    return this.executeQuery(query);
  }
  
  async findOne(filters: any): Promise<T | null> {
    const query = this.addTenantFilter({
      where: filters
    });
    
    return this.executeQuery(query);
  }
  
  async find(filters?: any): Promise<T[]> {
    const query = this.addTenantFilter({
      where: filters || {}
    });
    
    return this.executeQuery(query);
  }
  
  async create(data: Partial<T>): Promise<T> {
    const tenantId = this.getCurrentTenantId();
    
    const entityData = {
      ...data,
      tenantId,
      createdAt: new Date(),
      updatedAt: new Date()
    };
    
    return this.executeCreate(entityData);
  }
  
  async update(id: string, data: Partial<T>): Promise<T> {
    const tenantId = this.getCurrentTenantId();
    
    const updateData = {
      ...data,
      tenantId,
      updatedAt: new Date()
    };
    
    return this.executeUpdate(id, updateData);
  }
  
  async delete(id: string): Promise<void> {
    const tenantId = this.getCurrentTenantId();
    
    // Verificar que el registro pertenece al tenant actual
    const existing = await this.findById(id);
    if (!existing) {
      throw new Error('Entity not found or access denied');
    }
    
    await this.executeDelete(id, tenantId);
  }
  
  protected abstract executeQuery(query: any): Promise<T | T[]>;
  protected abstract executeCreate(data: any): Promise<T>;
  protected abstract executeUpdate(id: string, data: any): Promise<T>;
  protected abstract executeDelete(id: string, tenantId: string): Promise<void>;
}

// Implementación concreta para usuarios
class TenantUserRepository extends TenantAwareRepository<User> {
  protected tableName = 'users';
  
  protected async executeQuery(query: any): Promise<User | User[]> {
    // Implementar lógica de consulta específica para usuarios
    // Esto podría ser con SQLite, Realm, o cualquier otra base de datos
    return [];
  }
  
  protected async executeCreate(data: any): Promise<User> {
    // Implementar creación de usuario
    return data as User;
  }
  
  protected async executeUpdate(id: string, data: any): Promise<User> {
    // Implementar actualización de usuario
    return data as User;
  }
  
  protected async executeDelete(id: string, tenantId: string): Promise<void> {
    // Implementar eliminación de usuario
  }
  
  // Métodos específicos para usuarios
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ email });
  }
  
  async findByRole(role: string): Promise<User[]> {
    return this.find({ role });
  }
}
```

#### 3.2 Tenant Context Provider

```typescript
// context/TenantContext.tsx
interface TenantContextValue {
  currentTenant: Tenant | null;
  setCurrentTenant: (tenantId: string) => Promise<void>;
  isLoading: boolean;
  error: string | null;
}

const TenantContext = createContext<TenantContextValue | null>(null);

export const TenantProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [currentTenant, setCurrentTenantState] = useState<Tenant | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const tenantService = useMemo(() => {
    const resolver = new HybridTenantResolver([
      new SubdomainTenantResolver(),
      new UserConfigTenantResolver()
    ]);
    return new TenantService(resolver);
  }, []);
  
  const setCurrentTenant = useCallback(async (tenantId: string) => {
    try {
      setIsLoading(true);
      setError(null);
      
      await tenantService.setCurrentTenant(tenantId);
      const tenant = tenantService.getCurrentTenant();
      setCurrentTenantState(tenant);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  }, [tenantService]);
  
  useEffect(() => {
    const initializeTenant = async () => {
      try {
        setIsLoading(true);
        await tenantService.initialize();
        const tenant = tenantService.getCurrentTenant();
        setCurrentTenantState(tenant);
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };
    
    initializeTenant();
  }, [tenantService]);
  
  const value: TenantContextValue = {
    currentTenant,
    setCurrentTenant,
    isLoading,
    error
  };
  
  return (
    <TenantContext.Provider value={value}>
      {children}
    </TenantContext.Provider>
  );
};

export const useTenant = (): TenantContextValue => {
  const context = useContext(TenantContext);
  if (!context) {
    throw new Error('useTenant must be used within TenantProvider');
  }
  return context;
};
```

### 4. Configuración por Tenant

#### 4.1 Tenant Configuration Service

```typescript
// services/TenantConfigurationService.ts
class TenantConfigurationService {
  private tenantService: TenantService;
  private cache: Map<string, TenantConfiguration> = new Map();
  private cacheTTL = 30 * 60 * 1000; // 30 minutos
  
  constructor(tenantService: TenantService) {
    this.tenantService = tenantService;
  }
  
  async getConfiguration(): Promise<TenantConfiguration> {
    const currentTenant = this.tenantService.getCurrentTenant();
    if (!currentTenant) {
      throw new Error('No current tenant set');
    }
    
    // Verificar cache
    const cached = this.cache.get(currentTenant.id);
    if (cached && this.isCacheValid(currentTenant.id)) {
      return cached;
    }
    
    // Cargar configuración
    const config = await this.loadConfiguration(currentTenant.id);
    
    // Guardar en cache
    this.cache.set(currentTenant.id, {
      ...config,
      _cachedAt: Date.now()
    });
    
    return config;
  }
  
  async getTheme(): Promise<ThemeConfig> {
    const config = await this.getConfiguration();
    return config.theme;
  }
  
  async getBranding(): Promise<BrandingConfig> {
    const config = await this.getConfiguration();
    return config.branding;
  }
  
  async getFeatures(): Promise<FeatureConfig> {
    const config = await this.getConfiguration();
    return config.features;
  }
  
  async isFeatureEnabled(featureKey: string): Promise<boolean> {
    const features = await this.getFeatures();
    return features[featureKey] === true;
  }
  
  async getLimit(limitKey: string): Promise<number> {
    const config = await this.getConfiguration();
    return config.limits[limitKey] || 0;
  }
  
  private async loadConfiguration(tenantId: string): Promise<TenantConfiguration> {
    try {
      const response = await fetch(`/api/tenants/${tenantId}/configuration`, {
        headers: {
          'Authorization': `Bearer ${this.getAuthToken()}`,
          'Content-Type': 'application/json'
        }
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Failed to load tenant configuration:', error);
      
      // Retornar configuración por defecto
      return this.getDefaultConfiguration();
    }
  }
  
  private getDefaultConfiguration(): TenantConfiguration {
    return {
      theme: {
        primaryColor: '#007AFF',
        secondaryColor: '#5856D6',
        fontFamily: 'System',
        logoUrl: '',
        customCSS: ''
      },
      branding: {
        companyName: 'Default Company',
        logo: '',
        favicon: '',
        colors: {},
        fonts: {}
      },
      features: {
        darkMode: false,
        advancedSearch: false,
        analytics: false
      },
      limits: {
        maxUsers: 10,
        maxStorage: 1024 * 1024 * 100, // 100MB
        maxProjects: 5
      },
      integrations: {
        slack: false,
        email: true,
        sms: false
      }
    };
  }
  
  private isCacheValid(tenantId: string): boolean {
    const cached = this.cache.get(tenantId);
    if (!cached || !cached._cachedAt) {
      return false;
    }
    
    return Date.now() - cached._cachedAt < this.cacheTTL;
  }
  
  private getAuthToken(): string {
    // Implementar lógica para obtener token de autenticación
    return 'your-auth-token';
  }
}

// Hook para configuración del tenant
export const useTenantConfiguration = () => {
  const [configuration, setConfiguration] = useState<TenantConfiguration | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const { currentTenant } = useTenant();
  const configService = useMemo(() => {
    if (!currentTenant) return null;
    return new TenantConfigurationService(tenantService);
  }, [currentTenant]);
  
  useEffect(() => {
    const loadConfiguration = async () => {
      if (!configService) return;
      
      try {
        setLoading(true);
        setError(null);
        
        const config = await configService.getConfiguration();
        setConfiguration(config);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    loadConfiguration();
  }, [configService]);
  
  return {
    configuration,
    loading,
    error,
    refresh: loadConfiguration
  };
};
```

### 5. Componentes Multi-Tenant

#### 5.1 Tenant-Aware Components

```typescript
// components/TenantAwareComponent.tsx
interface TenantAwareComponentProps {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

export const TenantAwareComponent: React.FC<TenantAwareComponentProps> = ({
  children,
  fallback = <ActivityIndicator />
}) => {
  const { currentTenant, isLoading, error } = useTenant();
  
  if (isLoading) {
    return <>{fallback}</>;
  }
  
  if (error) {
    return <Text>Error: {error}</Text>;
  }
  
  if (!currentTenant) {
    return <Text>No tenant selected</Text>;
  }
  
  return <>{children}</>;
};

// Componente con tema del tenant
export const ThemedComponent: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { configuration, loading } = useTenantConfiguration();
  
  if (loading || !configuration) {
    return <ActivityIndicator />;
  }
  
  const { theme } = configuration;
  
  return (
    <View style={[styles.container, { backgroundColor: theme.primaryColor }]}>
      <Text style={[styles.text, { fontFamily: theme.fontFamily }]}>
        {children}
      </Text>
    </View>
  );
};

// Componente con branding del tenant
export const BrandedHeader: React.FC = () => {
  const { configuration, loading } = useTenantConfiguration();
  
  if (loading || !configuration) {
    return <View style={styles.header}><ActivityIndicator /></View>;
  }
  
  const { branding } = configuration;
  
  return (
    <View style={styles.header}>
      {branding.logo && (
        <Image source={{ uri: branding.logo }} style={styles.logo} />
      )}
      <Text style={styles.companyName}>{branding.companyName}</Text>
    </View>
  );
};

// Componente con límites del tenant
export const TenantLimitChecker: React.FC<{
  limitKey: string;
  currentValue: number;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}> = ({ limitKey, currentValue, children, fallback }) => {
  const { configuration, loading } = useTenantConfiguration();
  
  if (loading || !configuration) {
    return <ActivityIndicator />;
  }
  
  const limit = configuration.limits[limitKey] || 0;
  
  if (currentValue >= limit) {
    return <>{fallback || <Text>Limit reached for {limitKey}</Text>}</>;
  }
  
  return <>{children}</>;
};

// Uso en componentes
const UserManagement = () => {
  return (
    <TenantAwareComponent>
      <ThemedComponent>
        <BrandedHeader />
        
        <TenantLimitChecker limitKey="maxUsers" currentValue={5}>
          <UserList />
        </TenantLimitChecker>
        
        <TenantLimitChecker limitKey="maxUsers" currentValue={5} fallback={
          <Text>Upgrade your plan to add more users</Text>
        }>
          <AddUserButton />
        </TenantLimitChecker>
      </ThemedComponent>
    </TenantAwareComponent>
  );
};
```

### 6. Routing Multi-Tenant

#### 6.1 Tenant-Aware Navigation

```typescript
// navigation/TenantNavigation.tsx
class TenantNavigationService {
  private tenantService: TenantService;
  private navigation: any;
  
  constructor(tenantService: TenantService, navigation: any) {
    this.tenantService = tenantService;
    this.navigation = navigation;
  }
  
  navigateToTenantSpecificRoute(routeName: string, params?: any): void {
    const currentTenant = this.tenantService.getCurrentTenant();
    if (!currentTenant) {
      throw new Error('No current tenant set');
    }
    
    // Agregar información del tenant a los parámetros
    const tenantParams = {
      ...params,
      tenantId: currentTenant.id,
      tenantName: currentTenant.name
    };
    
    this.navigation.navigate(routeName, tenantParams);
  }
  
  getTenantSpecificRouteName(baseRoute: string): string {
    const currentTenant = this.tenantService.getCurrentTenant();
    if (!currentTenant) {
      return baseRoute;
    }
    
    return `${currentTenant.id}_${baseRoute}`;
  }
  
  canAccessRoute(routeName: string): boolean {
    const currentTenant = this.tenantService.getCurrentTenant();
    if (!currentTenant) {
      return false;
    }
    
    // Verificar si el tenant tiene acceso a esta ruta
    const routeAccess = this.getRouteAccessForTenant(currentTenant.id);
    return routeAccess.includes(routeName);
  }
  
  private getRouteAccessForTenant(tenantId: string): string[] {
    // Implementar lógica para obtener acceso a rutas por tenant
    // Esto podría venir de la configuración del tenant
    const routeAccessMap: Record<string, string[]> = {
      'tenant-basic': ['home', 'profile', 'settings'],
      'tenant-premium': ['home', 'profile', 'settings', 'analytics', 'reports'],
      'tenant-enterprise': ['home', 'profile', 'settings', 'analytics', 'reports', 'admin', 'billing']
    };
    
    return routeAccessMap[tenantId] || ['home'];
  }
}

// Hook para navegación multi-tenant
export const useTenantNavigation = () => {
  const navigation = useNavigation();
  const { currentTenant } = useTenant();
  
  const navigationService = useMemo(() => {
    if (!currentTenant) return null;
    return new TenantNavigationService(tenantService, navigation);
  }, [currentTenant, navigation]);
  
  const navigateToTenantRoute = useCallback((routeName: string, params?: any) => {
    if (!navigationService) {
      console.error('No tenant navigation service available');
      return;
    }
    
    navigationService.navigateToTenantSpecificRoute(routeName, params);
  }, [navigationService]);
  
  const canAccessRoute = useCallback((routeName: string): boolean => {
    if (!navigationService) return false;
    return navigationService.canAccessRoute(routeName);
  }, [navigationService]);
  
  return {
    navigateToTenantRoute,
    canAccessRoute,
    currentTenant
  };
};

// Uso en componentes
const TenantDashboard = () => {
  const { navigateToTenantRoute, canAccessRoute, currentTenant } = useTenantNavigation();
  
  if (!currentTenant) {
    return <Text>No tenant selected</Text>;
  }
  
  return (
    <View>
      <Text>Dashboard for {currentTenant.name}</Text>
      
      {canAccessRoute('profile') && (
        <TouchableOpacity onPress={() => navigateToTenantRoute('profile')}>
          <Text>Go to Profile</Text>
        </TouchableOpacity>
      )}
      
      {canAccessRoute('analytics') && (
        <TouchableOpacity onPress={() => navigateToTenantRoute('analytics')}>
          <Text>View Analytics</Text>
        </TouchableOpacity>
      )}
      
      {canAccessRoute('admin') && (
        <TouchableOpacity onPress={() => navigateToTenantRoute('admin')}>
          <Text>Admin Panel</Text>
        </TouchableOpacity>
      )}
    </View>
  );
};
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Implementar Sistema Multi-Tenant Básico

**Objetivo**: Crear un sistema básico de multi-tenancy con aislamiento de datos.

**Requisitos**:
1. **Implementar TenantService** para gestión de tenants
2. **Crear TenantAwareRepository** para aislamiento de datos
3. **Implementar TenantContext** para React
4. **Crear componentes** que usen información del tenant

**Código Base**:
```typescript
// Implementar desde cero
interface Tenant {
  id: string;
  name: string;
  configuration: TenantConfiguration;
}

class TenantService {
  // Implementar servicio de tenants
}

class TenantAwareRepository<T> {
  // Implementar repositorio con aislamiento
}
```

### Ejercicio 2: Configuración por Tenant

**Objetivo**: Implementar sistema de configuración personalizable por tenant.

**Requisitos**:
1. **TenantConfigurationService** para gestión de configuraciones
2. **Sistema de temas** personalizables por tenant
3. **Feature flags** por tenant
4. **Límites y restricciones** configurables

### Ejercicio 3: Navegación Multi-Tenant

**Objetivo**: Crear sistema de navegación que respete permisos del tenant.

**Requisitos**:
1. **TenantNavigationService** para rutas específicas
2. **Control de acceso** basado en plan del tenant
3. **Rutas dinámicas** según configuración
4. **Middleware de autorización** por tenant

---

## 📚 Recursos Adicionales

### Documentación
- [Multi-Tenant Architecture Patterns](https://martinfowler.com/articles/microservices.html)
- [SaaS Multi-Tenancy](https://en.wikipedia.org/wiki/Multitenancy)
- [Database Design for Multi-Tenant Applications](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-design-patterns-multi-tenancy-saas-applications)

### Herramientas
- [PostgreSQL Row Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [MySQL Multi-Tenancy](https://dev.mysql.com/doc/refman/8.0/en/)
- [SQLite with Tenant Isolation](https://www.sqlite.org/)

### Próximos Pasos
- **Clase 5**: Sistemas Event-Driven y Patrones Avanzados

---

## ✅ Checklist de Completado

- [ ] Comprendí el concepto de arquitecturas multi-tenant y sus patrones
- [ ] Implementé sistemas multi-tenant con aislamiento de datos
- [ ] Diseñé configuraciones por cliente y personalización
- [ ] Creé sistemas de routing y navegación multi-tenant
- [ ] Apliqué patrones de seguridad y escalabilidad multi-tenant
- [ ] Implementé TenantService para gestión de tenants
- [ ] Creé TenantAwareRepository para aislamiento de datos
- [ ] Implementé TenantContext para React
- [ ] Creé TenantConfigurationService para configuraciones
- [ ] Implementé TenantNavigationService para navegación
- [ ] Creé componentes multi-tenant con temas y branding
- [ ] Completé los ejercicios prácticos de multi-tenancy

---

**🎯 Próximo Objetivo**: En la siguiente clase aprenderemos sobre **Sistemas Event-Driven y Patrones Avanzados**, implementando arquitecturas basadas en eventos para aplicaciones escalables y desacopladas.
