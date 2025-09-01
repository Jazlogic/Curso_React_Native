# Clase 3: Feature Flags y Configuración Dinámica 🚩

## 📋 Objetivos de la Clase

Al finalizar esta clase, serás capaz de:

1. **Comprender** el concepto de feature flags y sus casos de uso
2. **Implementar** sistemas de feature flags robustos y escalables
3. **Diseñar** configuraciones dinámicas para aplicaciones React Native
4. **Crear** sistemas de A/B testing y rollouts graduales
5. **Aplicar** patrones de configuración remota y local

## ⏱️ Duración Estimada

**2 horas** - Teoría: 1h | Práctica: 1h

## 🔗 Navegación

- **📚 Módulo**: [Módulo 16: Arquitecturas Empresariales](../README.md)
- **⬅️ Anterior**: [Clase 2: Micro-Frontends y Modularización](clase_2_micro_frontends_modularizacion.md)
- **➡️ Siguiente**: [Clase 4: Arquitecturas Multi-Tenant](clase_4_arquitecturas_multi_tenant.md)
- **🏠 Inicio**: [Índice del Curso](../../../INDICE_COMPLETO.md)

---

## 🎯 Contenido Teórico

### 1. ¿Qué son los Feature Flags?

Los **feature flags** (también conocidos como feature toggles o feature switches) son una técnica de desarrollo que permite activar o desactivar funcionalidades de una aplicación sin necesidad de hacer un nuevo despliegue. En React Native, esto permite:

- **Control granular** de funcionalidades por usuario, región o versión
- **Rollouts graduales** de nuevas características
- **A/B testing** para optimizar la experiencia del usuario
- **Rollback rápido** en caso de problemas
- **Desarrollo continuo** sin interrumpir la producción

#### 1.1 Tipos de Feature Flags

```typescript
// ✅ TIPOS DE FEATURE FLAGS:

// 1. Release Flags: Controlan funcionalidades completas
const RELEASE_FLAGS = {
  NEW_UI_DESIGN: 'new-ui-design',
  ADVANCED_SEARCH: 'advanced-search',
  DARK_MODE: 'dark-mode'
};

// 2. Experiment Flags: Para A/B testing
const EXPERIMENT_FLAGS = {
  BUTTON_COLOR: 'button-color-experiment',
  CHECKOUT_FLOW: 'checkout-flow-experiment',
  RECOMMENDATION_ALGO: 'recommendation-algo-experiment'
};

// 3. Permission Flags: Control de acceso por rol
const PERMISSION_FLAGS = {
  ADMIN_FEATURES: 'admin-features',
  PREMIUM_FEATURES: 'premium-features',
  BETA_FEATURES: 'beta-features'
};

// 4. Operational Flags: Control operacional
const OPERATIONAL_FLAGS = {
  MAINTENANCE_MODE: 'maintenance-mode',
  RATE_LIMITING: 'rate-limiting',
  CACHE_ENABLED: 'cache-enabled'
};
```

### 2. Arquitectura del Sistema de Feature Flags

#### 2.1 Core Feature Flag Service

```typescript
// core/FeatureFlagService.ts
interface FeatureFlag {
  key: string;
  name: string;
  description: string;
  enabled: boolean;
  type: 'release' | 'experiment' | 'permission' | 'operational';
  rules: FeatureFlagRule[];
  metadata: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}

interface FeatureFlagRule {
  type: 'user' | 'percentage' | 'environment' | 'version' | 'custom';
  condition: any;
  value: any;
}

class FeatureFlagService {
  private flags: Map<string, FeatureFlag> = new Map();
  private providers: FeatureFlagProvider[] = [];
  private cache: Map<string, { value: boolean; expires: number }> = new Map();
  private cacheTTL = 5 * 60 * 1000; // 5 minutos
  
  constructor(providers: FeatureFlagProvider[] = []) {
    this.providers = providers;
  }
  
  async initialize(): Promise<void> {
    // Cargar flags desde todos los providers
    for (const provider of this.providers) {
      try {
        const flags = await provider.getFlags();
        flags.forEach(flag => this.flags.set(flag.key, flag));
      } catch (error) {
        console.error(`Failed to load flags from provider:`, error);
      }
    }
  }
  
  async isEnabled(flagKey: string, context: FeatureFlagContext): Promise<boolean> {
    // Verificar cache primero
    const cached = this.cache.get(flagKey);
    if (cached && cached.expires > Date.now()) {
      return cached.value;
    }
    
    // Obtener flag
    const flag = this.flags.get(flagKey);
    if (!flag) {
      return false; // Flag no encontrado, deshabilitado por defecto
    }
    
    // Evaluar reglas
    const isEnabled = await this.evaluateRules(flag.rules, context);
    
    // Guardar en cache
    this.cache.set(flagKey, {
      value: isEnabled,
      expires: Date.now() + this.cacheTTL
    });
    
    return isEnabled;
  }
  
  async getFlag(flagKey: string): Promise<FeatureFlag | null> {
    return this.flags.get(flagKey) || null;
  }
  
  async getAllFlags(): Promise<FeatureFlag[]> {
    return Array.from(this.flags.values());
  }
  
  async refreshFlags(): Promise<void> {
    this.cache.clear();
    await this.initialize();
  }
  
  private async evaluateRules(rules: FeatureFlagRule[], context: FeatureFlagContext): Promise<boolean> {
    if (rules.length === 0) {
      return true; // Sin reglas, habilitado por defecto
    }
    
    // Evaluar todas las reglas (AND lógico)
    for (const rule of rules) {
      if (!await this.evaluateRule(rule, context)) {
        return false;
      }
    }
    
    return true;
  }
  
  private async evaluateRule(rule: FeatureFlagRule, context: FeatureFlagContext): Promise<boolean> {
    switch (rule.type) {
      case 'user':
        return this.evaluateUserRule(rule, context);
      case 'percentage':
        return this.evaluatePercentageRule(rule, context);
      case 'environment':
        return this.evaluateEnvironmentRule(rule, context);
      case 'version':
        return this.evaluateVersionRule(rule, context);
      case 'custom':
        return this.evaluateCustomRule(rule, context);
      default:
        return false;
    }
  }
  
  private evaluateUserRule(rule: FeatureFlagRule, context: FeatureFlagContext): boolean {
    const { condition, value } = rule;
    
    switch (condition) {
      case 'user_id':
        return value.includes(context.userId);
      case 'user_role':
        return value.includes(context.userRole);
      case 'user_tier':
        return value.includes(context.userTier);
      default:
        return false;
    }
  }
  
  private evaluatePercentageRule(rule: FeatureFlagRule, context: FeatureFlagContext): boolean {
    const { condition, value } = rule;
    
    if (condition === 'random') {
      // Generar hash determinístico basado en userId
      const hash = this.hashString(context.userId || 'anonymous');
      const percentage = (hash % 100) + 1;
      return percentage <= value;
    }
    
    return false;
  }
  
  private evaluateEnvironmentRule(rule: FeatureFlagRule, context: FeatureFlagContext): boolean {
    const { condition, value } = rule;
    
    if (condition === 'environment') {
      return value.includes(context.environment);
    }
    
    return false;
  }
  
  private evaluateVersionRule(rule: FeatureFlagRule, context: FeatureFlagContext): boolean {
    const { condition, value } = rule;
    
    if (condition === 'app_version') {
      return this.compareVersions(context.appVersion, value);
    }
    
    return false;
  }
  
  private evaluateCustomRule(rule: FeatureFlagRule, context: FeatureFlagContext): boolean {
    // Implementar lógica personalizada
    if (typeof rule.condition === 'function') {
      return rule.condition(context, rule.value);
    }
    
    return false;
  }
  
  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convertir a 32-bit integer
    }
    return Math.abs(hash);
  }
  
  private compareVersions(version1: string, version2: string): boolean {
    const v1 = version1.split('.').map(Number);
    const v2 = version2.split('.').map(Number);
    
    for (let i = 0; i < Math.max(v1.length, v2.length); i++) {
      const num1 = v1[i] || 0;
      const num2 = v2[i] || 0;
      
      if (num1 > num2) return true;
      if (num1 < num2) return false;
    }
    
    return true;
  }
}
```

#### 2.2 Providers de Feature Flags

```typescript
// providers/FeatureFlagProvider.ts
interface FeatureFlagProvider {
  getFlags(): Promise<FeatureFlag[]>;
  updateFlag(flag: FeatureFlag): Promise<void>;
  deleteFlag(flagKey: string): Promise<void>;
}

// providers/LocalFeatureFlagProvider.ts
class LocalFeatureFlagProvider implements FeatureFlagProvider {
  private flags: FeatureFlag[] = [];
  private storage: AsyncStorage;
  
  constructor(storage: AsyncStorage) {
    this.storage = storage;
    this.loadFlagsFromStorage();
  }
  
  async getFlags(): Promise<FeatureFlag[]> {
    return this.flags;
  }
  
  async updateFlag(flag: FeatureFlag): Promise<void> {
    const index = this.flags.findIndex(f => f.key === flag.key);
    if (index >= 0) {
      this.flags[index] = { ...flag, updatedAt: new Date() };
    } else {
      this.flags.push({ ...flag, createdAt: new Date(), updatedAt: new Date() });
    }
    
    await this.saveFlagsToStorage();
  }
  
  async deleteFlag(flagKey: string): Promise<void> {
    this.flags = this.flags.filter(f => f.key !== flagKey);
    await this.saveFlagsToStorage();
  }
  
  private async loadFlagsFromStorage(): Promise<void> {
    try {
      const stored = await this.storage.getItem('feature_flags');
      if (stored) {
        this.flags = JSON.parse(stored);
      }
    } catch (error) {
      console.error('Failed to load flags from storage:', error);
    }
  }
  
  private async saveFlagsToStorage(): Promise<void> {
    try {
      await this.storage.setItem('feature_flags', JSON.stringify(this.flags));
    } catch (error) {
      console.error('Failed to save flags to storage:', error);
    }
  }
}

// providers/RemoteFeatureFlagProvider.ts
class RemoteFeatureFlagProvider implements FeatureFlagProvider {
  private apiUrl: string;
  private apiKey: string;
  private cache: Map<string, { flags: FeatureFlag[]; expires: number }> = new Map();
  private cacheTTL = 10 * 60 * 1000; // 10 minutos
  
  constructor(apiUrl: string, apiKey: string) {
    this.apiUrl = apiUrl;
    this.apiKey = apiKey;
  }
  
  async getFlags(): Promise<FeatureFlag[]> {
    // Verificar cache
    const cached = this.cache.get('flags');
    if (cached && cached.expires > Date.now()) {
      return cached.flags;
    }
    
    try {
      const response = await fetch(`${this.apiUrl}/flags`, {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        }
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const flags = await response.json();
      
      // Guardar en cache
      this.cache.set('flags', {
        flags,
        expires: Date.now() + this.cacheTTL
      });
      
      return flags;
    } catch (error) {
      console.error('Failed to fetch flags from remote:', error);
      
      // Retornar flags del cache si están disponibles (aunque expirados)
      const cached = this.cache.get('flags');
      if (cached) {
        return cached.flags;
      }
      
      return [];
    }
  }
  
  async updateFlag(flag: FeatureFlag): Promise<void> {
    try {
      const response = await fetch(`${this.apiUrl}/flags/${flag.key}`, {
        method: 'PUT',
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(flag)
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      // Limpiar cache
      this.cache.delete('flags');
    } catch (error) {
      console.error('Failed to update flag:', error);
      throw error;
    }
  }
  
  async deleteFlag(flagKey: string): Promise<void> {
    try {
      const response = await fetch(`${this.apiUrl}/flags/${flagKey}`, {
        method: 'DELETE',
        headers: {
          'Authorization': `Bearer ${this.apiKey}`
        }
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      // Limpiar cache
      this.cache.delete('flags');
    } catch (error) {
      console.error('Failed to delete flag:', error);
      throw error;
    }
  }
}
```

### 3. Hook de React para Feature Flags

```typescript
// hooks/useFeatureFlag.ts
interface UseFeatureFlagReturn {
  isEnabled: boolean;
  loading: boolean;
  error: string | null;
  flag: FeatureFlag | null;
  refresh: () => Promise<void>;
}

export const useFeatureFlag = (
  flagKey: string,
  context: FeatureFlagContext
): UseFeatureFlagReturn => {
  const [isEnabled, setIsEnabled] = useState(false);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [flag, setFlag] = useState<FeatureFlag | null>(null);
  
  const featureFlagService = useFeatureFlagService();
  
  const checkFlag = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const [enabled, flagData] = await Promise.all([
        featureFlagService.isEnabled(flagKey, context),
        featureFlagService.getFlag(flagKey)
      ]);
      
      setIsEnabled(enabled);
      setFlag(flagData);
    } catch (err) {
      setError(err.message);
      setIsEnabled(false);
    } finally {
      setLoading(false);
    }
  }, [flagKey, context, featureFlagService]);
  
  useEffect(() => {
    checkFlag();
  }, [checkFlag]);
  
  const refresh = useCallback(async () => {
    await checkFlag();
  }, [checkFlag]);
  
  return {
    isEnabled,
    loading,
    error,
    flag,
    refresh
  };
};

// hooks/useFeatureFlags.ts
export const useFeatureFlags = (
  flagKeys: string[],
  context: FeatureFlagContext
): Record<string, UseFeatureFlagReturn> => {
  const results: Record<string, UseFeatureFlagReturn> = {};
  
  flagKeys.forEach(key => {
    results[key] = useFeatureFlag(key, context);
  });
  
  return results;
};

// hooks/useFeatureFlagService.ts
export const useFeatureFlagService = (): FeatureFlagService => {
  const context = useContext(FeatureFlagContext);
  if (!context) {
    throw new Error('useFeatureFlagService must be used within FeatureFlagProvider');
  }
  return context.service;
};
```

### 4. Componentes con Feature Flags

```typescript
// components/FeatureFlagWrapper.tsx
interface FeatureFlagWrapperProps {
  flagKey: string;
  context: FeatureFlagContext;
  fallback?: React.ReactNode;
  children: React.ReactNode;
}

export const FeatureFlagWrapper: React.FC<FeatureFlagWrapperProps> = ({
  flagKey,
  context,
  fallback = null,
  children
}) => {
  const { isEnabled, loading, error } = useFeatureFlag(flagKey, context);
  
  if (loading) {
    return <ActivityIndicator />;
  }
  
  if (error) {
    console.error(`Feature flag error for ${flagKey}:`, error);
    return fallback;
  }
  
  return isEnabled ? <>{children}</> : <>{fallback}</>;
};

// components/ConditionalFeature.tsx
interface ConditionalFeatureProps {
  flagKey: string;
  context: FeatureFlagContext;
  enabledContent: React.ReactNode;
  disabledContent?: React.ReactNode;
  loadingContent?: React.ReactNode;
}

export const ConditionalFeature: React.FC<ConditionalFeatureProps> = ({
  flagKey,
  context,
  enabledContent,
  disabledContent = null,
  loadingContent = <ActivityIndicator />
}) => {
  const { isEnabled, loading } = useFeatureFlag(flagKey, context);
  
  if (loading) {
    return <>{loadingContent}</>;
  }
  
  return isEnabled ? <>{enabledContent}</> : <>{disabledContent}</>;
};

// Uso en componentes
const UserDashboard = () => {
  const context = useFeatureFlagContext();
  
  return (
    <View>
      <Text>User Dashboard</Text>
      
      {/* Feature flag simple */}
      <FeatureFlagWrapper flagKey="new-ui-design" context={context}>
        <NewUIDesign />
      </FeatureFlagWrapper>
      
      {/* Feature flag con contenido condicional */}
      <ConditionalFeature
        flagKey="advanced-search"
        context={context}
        enabledContent={<AdvancedSearch />}
        disabledContent={<BasicSearch />}
      />
      
      {/* Feature flag con fallback */}
      <FeatureFlagWrapper 
        flagKey="dark-mode" 
        context={context}
        fallback={<Text>Dark mode coming soon!</Text>}
      >
        <DarkModeToggle />
      </FeatureFlagWrapper>
    </View>
  );
};
```

### 5. Sistema de A/B Testing

```typescript
// core/ABTestingService.ts
interface ABTest {
  id: string;
  name: string;
  description: string;
  variants: ABTestVariant[];
  trafficAllocation: number; // Porcentaje de usuarios en el test
  startDate: Date;
  endDate?: Date;
  status: 'draft' | 'running' | 'paused' | 'completed';
}

interface ABTestVariant {
  id: string;
  name: string;
  weight: number; // Porcentaje del tráfico para esta variante
  config: Record<string, any>;
}

class ABTestingService {
  private tests: Map<string, ABTest> = new Map();
  private featureFlagService: FeatureFlagService;
  
  constructor(featureFlagService: FeatureFlagService) {
    this.featureFlagService = featureFlagService;
  }
  
  async createTest(test: ABTest): Promise<void> {
    // Crear feature flags para cada variante
    for (const variant of test.variants) {
      const flagKey = `ab_test_${test.id}_${variant.id}`;
      
      await this.featureFlagService.updateFlag({
        key: flagKey,
        name: `${test.name} - ${variant.name}`,
        description: test.description,
        enabled: test.status === 'running',
        type: 'experiment',
        rules: [
          {
            type: 'percentage',
            condition: 'random',
            value: (test.trafficAllocation * variant.weight) / 100
          }
        ],
        metadata: {
          abTestId: test.id,
          variantId: variant.id,
          variantConfig: variant.config
        },
        createdAt: new Date(),
        updatedAt: new Date()
      });
    }
    
    this.tests.set(test.id, test);
  }
  
  async getVariant(testId: string, userId: string): Promise<ABTestVariant | null> {
    const test = this.tests.get(testId);
    if (!test || test.status !== 'running') {
      return null;
    }
    
    // Determinar variante basada en userId y pesos
    const hash = this.hashString(userId + testId);
    const randomValue = (hash % 100) + 1;
    
    let cumulativeWeight = 0;
    for (const variant of test.variants) {
      cumulativeWeight += variant.weight;
      if (randomValue <= cumulativeWeight) {
        return variant;
      }
    }
    
    return test.variants[0]; // Fallback
  }
  
  async trackEvent(testId: string, variantId: string, eventName: string, userId: string): Promise<void> {
    // Implementar tracking de eventos para análisis
    console.log(`AB Test Event: ${testId}/${variantId} - ${eventName} - User: ${userId}`);
  }
  
  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
}

// Uso del A/B testing
const abTestingService = new ABTestingService(featureFlagService);

// Crear test A/B
await abTestingService.createTest({
  id: 'button-color-test',
  name: 'Button Color Test',
  description: 'Testing different button colors for conversion',
  variants: [
    { id: 'control', name: 'Control (Blue)', weight: 50, config: { color: '#007AFF' } },
    { id: 'variant-a', name: 'Variant A (Green)', weight: 25, config: { color: '#34C759' } },
    { id: 'variant-b', name: 'Variant B (Red)', weight: 25, config: { color: '#FF3B30' } }
  ],
  trafficAllocation: 20, // 20% de usuarios en el test
  startDate: new Date(),
  status: 'running'
});

// En el componente
const ButtonColorTest = () => {
  const context = useFeatureFlagContext();
  const { isEnabled: isInTest } = useFeatureFlag('ab_test_button-color-test_control', context);
  
  if (isInTest) {
    // Usuario está en el test, mostrar variante
    const variant = await abTestingService.getVariant('button-color-test', userId);
    return (
      <TouchableOpacity 
        style={{ backgroundColor: variant.config.color }}
        onPress={() => {
          abTestingService.trackEvent('button-color-test', variant.id, 'button_clicked', userId);
        }}
      >
        <Text>Click Me</Text>
      </TouchableOpacity>
    );
  }
  
  // Usuario no está en el test, mostrar botón normal
  return (
    <TouchableOpacity style={{ backgroundColor: '#007AFF' }}>
      <Text>Click Me</Text>
    </TouchableOpacity>
  );
};
```

### 6. Configuración Dinámica

```typescript
// core/ConfigurationService.ts
interface Configuration {
  key: string;
  value: any;
  type: 'string' | 'number' | 'boolean' | 'object' | 'array';
  description: string;
  defaultValue: any;
  validation?: (value: any) => boolean;
  metadata: Record<string, any>;
}

class ConfigurationService {
  private configs: Map<string, Configuration> = new Map();
  private values: Map<string, any> = new Map();
  private listeners: Map<string, Function[]> = new Map();
  
  async setConfig(key: string, value: any): Promise<void> {
    const config = this.configs.get(key);
    if (!config) {
      throw new Error(`Configuration ${key} not found`);
    }
    
    // Validar valor
    if (config.validation && !config.validation(value)) {
      throw new Error(`Invalid value for configuration ${key}`);
    }
    
    // Actualizar valor
    this.values.set(key, value);
    
    // Notificar listeners
    this.notifyListeners(key, value);
  }
  
  getConfig(key: string): any {
    return this.values.get(key) ?? this.configs.get(key)?.defaultValue;
  }
  
  hasConfig(key: string): boolean {
    return this.configs.has(key);
  }
  
  watchConfig(key: string, listener: Function): () => void {
    if (!this.listeners.has(key)) {
      this.listeners.set(key, []);
    }
    
    this.listeners.get(key)!.push(listener);
    
    return () => {
      const listeners = this.listeners.get(key) || [];
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  }
  
  private notifyListeners(key: string, value: any): void {
    const listeners = this.listeners.get(key) || [];
    listeners.forEach(listener => {
      try {
        listener(value);
      } catch (error) {
        console.error(`Error in config listener for ${key}:`, error);
      }
    });
  }
}

// Hook para configuración
export const useConfiguration = (key: string): [any, (value: any) => Promise<void>] => {
  const [value, setValue] = useState<any>(null);
  const configService = useConfigurationService();
  
  useEffect(() => {
    const currentValue = configService.getConfig(key);
    setValue(currentValue);
    
    const unsubscribe = configService.watchConfig(key, (newValue: any) => {
      setValue(newValue);
    });
    
    return unsubscribe;
  }, [key, configService]);
  
  const updateValue = useCallback(async (newValue: any) => {
    await configService.setConfig(key, newValue);
  }, [key, configService]);
  
  return [value, updateValue];
};

// Uso en componentes
const DynamicButton = () => {
  const [buttonColor, setButtonColor] = useConfiguration('button_color');
  const [buttonText, setButtonText] = useConfiguration('button_text');
  
  return (
    <TouchableOpacity 
      style={{ backgroundColor: buttonColor || '#007AFF' }}
      onPress={() => {
        // Cambiar configuración dinámicamente
        setButtonColor('#FF3B30');
        setButtonText('Clicked!');
      }}
    >
      <Text style={{ color: 'white' }}>{buttonText || 'Click Me'}</Text>
    </TouchableOpacity>
  );
};
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Implementar Sistema de Feature Flags

**Objetivo**: Crear un sistema completo de feature flags con providers locales y remotos.

**Requisitos**:
1. **Implementar FeatureFlagService** con evaluación de reglas
2. **Crear providers** para almacenamiento local y remoto
3. **Implementar hook useFeatureFlag** para React
4. **Crear componentes** que usen feature flags

**Código Base**:
```typescript
// Implementar desde cero
interface FeatureFlag {
  key: string;
  enabled: boolean;
  rules: FeatureFlagRule[];
}

class FeatureFlagService {
  // Implementar servicio completo
}

const useFeatureFlag = (flagKey: string) => {
  // Implementar hook
};
```

### Ejercicio 2: Sistema de A/B Testing

**Objetivo**: Crear un sistema de A/B testing que permita experimentos controlados.

**Requisitos**:
1. **Implementar ABTestingService** para gestión de tests
2. **Sistema de asignación** de variantes por usuario
3. **Tracking de eventos** para análisis
4. **Integración** con feature flags

**Código Base**:
```typescript
// Implementar desde cero
interface ABTest {
  id: string;
  variants: ABTestVariant[];
  trafficAllocation: number;
}

class ABTestingService {
  // Implementar servicio de A/B testing
}
```

### Ejercicio 3: Configuración Dinámica

**Objetivo**: Implementar un sistema de configuración que permita cambios en tiempo real.

**Requisitos**:
1. **ConfigurationService** para gestión de configuraciones
2. **Sistema de validación** para valores de configuración
3. **Hook useConfiguration** para React
4. **Componentes** que reaccionen a cambios de configuración

---

## 📚 Recursos Adicionales

### Documentación
- [Feature Flags Best Practices](https://featureflags.io/)
- [LaunchDarkly Documentation](https://docs.launchdarkly.com/)
- [Split.io Documentation](https://help.split.io/)

### Herramientas
- [LaunchDarkly](https://launchdarkly.com/)
- [Split.io](https://split.io/)
- [Optimizely](https://www.optimizely.com/)

### Próximos Pasos
- **Clase 4**: Arquitecturas Multi-Tenant
- **Clase 5**: Sistemas Event-Driven y Patrones Avanzados

---

## ✅ Checklist de Completado

- [ ] Comprendí el concepto de feature flags y sus casos de uso
- [ ] Implementé sistemas de feature flags robustos y escalables
- [ ] Diseñé configuraciones dinámicas para aplicaciones React Native
- [ ] Creé sistemas de A/B testing y rollouts graduales
- [ ] Apliqué patrones de configuración remota y local
- [ ] Implementé FeatureFlagService con evaluación de reglas
- [ ] Creé providers para almacenamiento local y remoto
- [ ] Implementé hook useFeatureFlag para React
- [ ] Creé componentes que usen feature flags
- [ ] Implementé sistema de A/B testing con variantes
- [ ] Creé sistema de configuración dinámica
- [ ] Completé los ejercicios prácticos de feature flags

---

**🎯 Próximo Objetivo**: En la siguiente clase aprenderemos sobre **Arquitecturas Multi-Tenant**, implementando sistemas que permitan servir múltiples clientes con configuraciones y datos aislados.
