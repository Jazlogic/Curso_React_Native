# Clase 4: Arquitectura de Sistemas de Componentes

## Objetivos de la Clase
- Diseñar arquitecturas escalables de sistemas de componentes
- Implementar patrones de composición avanzados
- Gestionar dependencias y versionado
- Crear sistemas de documentación y testing

## 1. Arquitectura de Sistemas de Componentes

### Principios Arquitectónicos
- **Modularidad**: Componentes independientes y cohesivos
- **Escalabilidad**: Fácil adición de nuevos componentes
- **Mantenibilidad**: Código limpio y bien estructurado
- **Reutilización**: Máxima reutilización de código

### Estructura de Directorios Avanzada
```
design-system/
├── packages/
│   ├── core/              # Tokens y utilidades base
│   │   ├── tokens/
│   │   ├── utils/
│   │   └── types/
│   ├── components/        # Componentes base
│   │   ├── Button/
│   │   ├── Input/
│   │   └── Text/
│   ├── layouts/           # Componentes de layout
│   │   ├── Container/
│   │   ├── Grid/
│   │   └── Stack/
│   ├── patterns/          # Patrones compuestos
│   │   ├── Form/
│   │   ├── Modal/
│   │   └── List/
│   └── themes/            # Temas y variaciones
│       ├── light/
│       ├── dark/
│       └── custom/
├── docs/                  # Documentación
│   ├── stories/
│   ├── examples/
│   └── guides/
├── tests/                 # Tests globales
│   ├── unit/
│   ├── integration/
│   └── e2e/
└── tools/                 # Herramientas de desarrollo
    ├── build/
    ├── lint/
    └── release/
```

## 2. Gestión de Dependencias

### Package.json del Sistema
```json
{
  "name": "@company/design-system",
  "version": "1.0.0",
  "description": "Design System for React Native",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc && npm run build:stories",
    "build:stories": "storybook build",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.{ts,tsx}",
    "lint:fix": "eslint src/**/*.{ts,tsx} --fix",
    "release": "npm run build && npm publish",
    "storybook": "storybook dev -p 6006"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-native": "^0.72.0",
    "@react-native-async-storage/async-storage": "^1.19.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "@types/react-native": "^0.72.0",
    "typescript": "^5.0.0",
    "jest": "^29.0.0",
    "@testing-library/react-native": "^12.0.0",
    "eslint": "^8.0.0",
    "@storybook/react-native": "^6.0.0"
  },
  "peerDependencies": {
    "react": ">=16.8.0",
    "react-native": ">=0.60.0"
  }
}
```

### Configuración de TypeScript
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2018",
    "lib": ["es2018"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": false,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "jsx": "react-jsx"
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.*",
    "**/*.stories.*"
  ]
}
```

## 3. Sistema de Tokens Avanzado

### Tokens con Metadatos
```typescript
// packages/core/tokens/types.ts
export interface TokenMetadata {
  name: string;
  description: string;
  category: string;
  version: string;
  deprecated?: boolean;
  replacement?: string;
}

export interface ColorToken {
  value: string;
  metadata: TokenMetadata;
  variants?: {
    light: string;
    dark: string;
  };
}

export interface TypographyToken {
  value: {
    fontSize: number;
    fontWeight: string;
    lineHeight: number;
    fontFamily: string;
  };
  metadata: TokenMetadata;
}

export interface SpacingToken {
  value: number;
  metadata: TokenMetadata;
  scale?: 'linear' | 'exponential';
}
```

### Generador de Tokens
```typescript
// packages/core/tokens/generator.ts
import { ColorToken, TypographyToken, SpacingToken } from './types';

export class TokenGenerator {
  private tokens: Map<string, any> = new Map();
  
  addColorToken(name: string, token: ColorToken) {
    this.tokens.set(`color.${name}`, token);
  }
  
  addTypographyToken(name: string, token: TypographyToken) {
    this.tokens.set(`typography.${name}`, token);
  }
  
  addSpacingToken(name: string, token: SpacingToken) {
    this.tokens.set(`spacing.${name}`, token);
  }
  
  generateTheme(themeName: string) {
    const theme: any = {};
    
    for (const [key, token] of this.tokens) {
      const keys = key.split('.');
      let current = theme;
      
      for (let i = 0; i < keys.length - 1; i++) {
        if (!current[keys[i]]) {
          current[keys[i]] = {};
        }
        current = current[keys[i]];
      }
      
      current[keys[keys.length - 1]] = token.value;
    }
    
    return theme;
  }
  
  exportToJSON() {
    const result: any = {};
    for (const [key, token] of this.tokens) {
      result[key] = token;
    }
    return JSON.stringify(result, null, 2);
  }
  
  exportToCSS() {
    let css = ':root {\n';
    for (const [key, token] of this.tokens) {
      const cssVar = `--${key.replace('.', '-')}`;
      css += `  ${cssVar}: ${token.value};\n`;
    }
    css += '}';
    return css;
  }
}
```

## 4. Sistema de Componentes con Composición

### Component Registry
```typescript
// packages/core/registry/ComponentRegistry.ts
import { ComponentType } from 'react';

export interface ComponentMetadata {
  name: string;
  version: string;
  category: string;
  description: string;
  props: Record<string, any>;
  examples: any[];
  dependencies: string[];
}

export class ComponentRegistry {
  private components: Map<string, ComponentType> = new Map();
  private metadata: Map<string, ComponentMetadata> = new Map();
  
  register<T extends ComponentType>(
    name: string,
    component: T,
    metadata: ComponentMetadata
  ) {
    this.components.set(name, component);
    this.metadata.set(name, metadata);
  }
  
  get(name: string): ComponentType | undefined {
    return this.components.get(name);
  }
  
  getMetadata(name: string): ComponentMetadata | undefined {
    return this.metadata.get(name);
  }
  
  getAllComponents(): string[] {
    return Array.from(this.components.keys());
  }
  
  getComponentsByCategory(category: string): string[] {
    return Array.from(this.metadata.entries())
      .filter(([_, metadata]) => metadata.category === category)
      .map(([name, _]) => name);
  }
}
```

### Component Factory
```typescript
// packages/core/factory/ComponentFactory.ts
import { ComponentType, createElement } from 'react';
import { ComponentRegistry } from '../registry/ComponentRegistry';

export class ComponentFactory {
  constructor(private registry: ComponentRegistry) {}
  
  createComponent(
    name: string,
    props: any = {},
    children?: any
  ): React.ReactElement | null {
    const Component = this.registry.get(name);
    
    if (!Component) {
      console.warn(`Component ${name} not found in registry`);
      return null;
    }
    
    return createElement(Component, props, children);
  }
  
  createFromConfig(config: {
    type: string;
    props?: any;
    children?: any[];
  }): React.ReactElement | null {
    const { type, props = {}, children = [] } = config;
    
    const childElements = children.map((child, index) => {
      if (typeof child === 'string') {
        return child;
      }
      
      if (typeof child === 'object' && child.type) {
        return this.createFromConfig(child);
      }
      
      return null;
    }).filter(Boolean);
    
    return this.createComponent(type, props, childElements);
  }
}
```

## 5. Sistema de Temas Avanzado

### Theme Manager
```typescript
// packages/themes/ThemeManager.ts
import { Theme } from '../core/tokens/theme';
import { TokenGenerator } from '../core/tokens/generator';

export class ThemeManager {
  private themes: Map<string, Theme> = new Map();
  private currentTheme: string = 'light';
  private listeners: Set<(theme: Theme) => void> = new Set();
  
  constructor(private tokenGenerator: TokenGenerator) {
    this.initializeDefaultThemes();
  }
  
  private initializeDefaultThemes() {
    // Light theme
    const lightTheme = this.tokenGenerator.generateTheme('light');
    this.themes.set('light', lightTheme);
    
    // Dark theme
    const darkTheme = this.tokenGenerator.generateTheme('dark');
    this.themes.set('dark', darkTheme);
  }
  
  addTheme(name: string, theme: Theme) {
    this.themes.set(name, theme);
  }
  
  setTheme(name: string) {
    if (this.themes.has(name)) {
      this.currentTheme = name;
      this.notifyListeners();
    }
  }
  
  getCurrentTheme(): Theme {
    return this.themes.get(this.currentTheme) || this.themes.get('light')!;
  }
  
  getTheme(name: string): Theme | undefined {
    return this.themes.get(name);
  }
  
  subscribe(listener: (theme: Theme) => void) {
    this.listeners.add(listener);
    
    return () => {
      this.listeners.delete(listener);
    };
  }
  
  private notifyListeners() {
    const theme = this.getCurrentTheme();
    this.listeners.forEach(listener => listener(theme));
  }
}
```

### Theme Provider Avanzado
```typescript
// packages/themes/ThemeProvider.tsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import { ThemeManager } from './ThemeManager';
import { Theme } from '../core/tokens/theme';

interface ThemeContextType {
  theme: Theme;
  themeName: string;
  setTheme: (name: string) => void;
  availableThemes: string[];
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

interface ThemeProviderProps {
  children: React.ReactNode;
  themeManager: ThemeManager;
  initialTheme?: string;
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({
  children,
  themeManager,
  initialTheme = 'light',
}) => {
  const [theme, setTheme] = useState(themeManager.getCurrentTheme());
  const [themeName, setThemeName] = useState(initialTheme);
  
  useEffect(() => {
    themeManager.setTheme(initialTheme);
    
    const unsubscribe = themeManager.subscribe((newTheme) => {
      setTheme(newTheme);
      setThemeName(themeManager.currentTheme);
    });
    
    return unsubscribe;
  }, [themeManager, initialTheme]);
  
  const handleSetTheme = (name: string) => {
    themeManager.setTheme(name);
  };
  
  const availableThemes = Array.from(themeManager.themes.keys());
  
  return (
    <ThemeContext.Provider
      value={{
        theme,
        themeName,
        setTheme: handleSetTheme,
        availableThemes,
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
};
```

## 6. Sistema de Documentación

### Documentación Automática
```typescript
// tools/docs/DocumentationGenerator.ts
import { ComponentRegistry } from '../packages/core/registry/ComponentRegistry';
import { TokenGenerator } from '../packages/core/tokens/generator';

export class DocumentationGenerator {
  constructor(
    private componentRegistry: ComponentRegistry,
    private tokenGenerator: TokenGenerator
  ) {}
  
  generateComponentDocs(componentName: string): string {
    const metadata = this.componentRegistry.getMetadata(componentName);
    if (!metadata) {
      return `# ${componentName}\n\nComponent not found.`;
    }
    
    return `# ${metadata.name}

## Description
${metadata.description}

## Version
${metadata.version}

## Props
${this.generatePropsTable(metadata.props)}

## Examples
${this.generateExamples(metadata.examples)}

## Dependencies
${metadata.dependencies.map(dep => `- ${dep}`).join('\n')}
`;
  }
  
  generateTokensDocs(): string {
    const tokens = this.tokenGenerator.exportToJSON();
    const parsedTokens = JSON.parse(tokens);
    
    let docs = '# Design Tokens\n\n';
    
    for (const [category, tokens] of Object.entries(parsedTokens)) {
      docs += `## ${category}\n\n`;
      
      for (const [name, token] of Object.entries(tokens as any)) {
        docs += `### ${name}\n`;
        docs += `**Value:** ${token.value}\n\n`;
        if (token.metadata?.description) {
          docs += `**Description:** ${token.metadata.description}\n\n`;
        }
      }
    }
    
    return docs;
  }
  
  private generatePropsTable(props: Record<string, any>): string {
    let table = '| Prop | Type | Default | Description |\n';
    table += '|------|------|---------|-------------|\n';
    
    for (const [prop, config] of Object.entries(props)) {
      table += `| ${prop} | ${config.type} | ${config.default || '-'} | ${config.description || '-'} |\n`;
    }
    
    return table;
  }
  
  private generateExamples(examples: any[]): string {
    return examples.map((example, index) => {
      return `### Example ${index + 1}\n\n\`\`\`tsx\n${example.code}\n\`\`\``;
    }).join('\n\n');
  }
}
```

## 7. Sistema de Testing

### Test Suite Global
```typescript
// tests/setup/TestSetup.ts
import { configure } from '@testing-library/react-native';
import { ThemeProvider } from '../packages/themes/ThemeProvider';
import { ThemeManager } from '../packages/themes/ThemeManager';
import { TokenGenerator } from '../packages/core/tokens/generator';

// Configurar testing library
configure({
  asyncUtilTimeout: 5000,
});

// Setup global test utilities
export const createTestThemeManager = () => {
  const tokenGenerator = new TokenGenerator();
  // Agregar tokens de prueba
  tokenGenerator.addColorToken('primary', {
    value: '#007AFF',
    metadata: {
      name: 'primary',
      description: 'Primary brand color',
      category: 'color',
      version: '1.0.0',
    },
  });
  
  return new ThemeManager(tokenGenerator);
};

export const renderWithTheme = (component: React.ReactElement) => {
  const themeManager = createTestThemeManager();
  
  return render(
    <ThemeProvider themeManager={themeManager}>
      {component}
    </ThemeProvider>
  );
};
```

### Test de Integración
```typescript
// tests/integration/ComponentIntegration.test.tsx
import React from 'react';
import { renderWithTheme } from '../setup/TestSetup';
import { Button } from '../../packages/components/Button';
import { Card } from '../../packages/patterns/Card';

describe('Component Integration', () => {
  it('should render Button inside Card correctly', () => {
    const { getByText } = renderWithTheme(
      <Card>
        <Button title="Test Button" />
      </Card>
    );
    
    expect(getByText('Test Button')).toBeTruthy();
  });
  
  it('should handle theme changes across components', () => {
    const { getByText, rerender } = renderWithTheme(
      <Button title="Test Button" />
    );
    
    // Cambiar tema y re-renderizar
    rerender(
      <Button title="Test Button" />
    );
    
    expect(getByText('Test Button')).toBeTruthy();
  });
});
```

## 8. Sistema de Build y Release

### Build Script
```typescript
// tools/build/BuildScript.ts
import { execSync } from 'child_process';
import { writeFileSync, mkdirSync } from 'fs';
import { join } from 'path';

export class BuildScript {
  private outputDir = 'dist';
  
  async build() {
    console.log('Building design system...');
    
    // Limpiar directorio de salida
    this.cleanOutput();
    
    // Compilar TypeScript
    this.compileTypeScript();
    
    // Generar documentación
    this.generateDocumentation();
    
    // Generar archivos de distribución
    this.generateDistributionFiles();
    
    console.log('Build completed successfully!');
  }
  
  private cleanOutput() {
    execSync(`rm -rf ${this.outputDir}`);
    mkdirSync(this.outputDir, { recursive: true });
  }
  
  private compileTypeScript() {
    execSync('tsc', { stdio: 'inherit' });
  }
  
  private generateDocumentation() {
    // Generar documentación de componentes
    const componentDocs = this.generateComponentDocumentation();
    writeFileSync(
      join(this.outputDir, 'docs', 'components.md'),
      componentDocs
    );
    
    // Generar documentación de tokens
    const tokenDocs = this.generateTokenDocumentation();
    writeFileSync(
      join(this.outputDir, 'docs', 'tokens.md'),
      tokenDocs
    );
  }
  
  private generateDistributionFiles() {
    // Generar archivo de índice principal
    const indexContent = this.generateIndexFile();
    writeFileSync(join(this.outputDir, 'index.js'), indexContent);
    
    // Generar archivos de tipos
    const typesContent = this.generateTypesFile();
    writeFileSync(join(this.outputDir, 'index.d.ts'), typesContent);
  }
  
  private generateComponentDocumentation(): string {
    // Implementar generación de documentación
    return '# Components Documentation\n\n';
  }
  
  private generateTokenDocumentation(): string {
    // Implementar generación de documentación de tokens
    return '# Tokens Documentation\n\n';
  }
  
  private generateIndexFile(): string {
    return `
export * from './components';
export * from './tokens';
export * from './themes';
export * from './utils';
`;
  }
  
  private generateTypesFile(): string {
    return `
export * from './components';
export * from './tokens';
export * from './themes';
export * from './utils';
`;
  }
}
```

## 9. Monorepo con Lerna

### Configuración de Lerna
```json
// lerna.json
{
  "version": "1.0.0",
  "npmClient": "npm",
  "command": {
    "publish": {
      "conventionalCommits": true,
      "message": "chore(release): publish"
    },
    "bootstrap": {
      "hoist": true
    }
  },
  "packages": [
    "packages/*"
  ]
}
```

### Workspace Configuration
```json
// package.json (root)
{
  "name": "design-system-monorepo",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "bootstrap": "lerna bootstrap",
    "build": "lerna run build",
    "test": "lerna run test",
    "lint": "lerna run lint",
    "release": "lerna publish"
  },
  "devDependencies": {
    "lerna": "^6.0.0"
  }
}
```

## 10. Mejores Prácticas

### Organización
- **Separación de responsabilidades**: Cada paquete tiene una responsabilidad específica
- **Dependencias claras**: Evitar dependencias circulares
- **Versionado semántico**: Seguir SemVer para releases
- **Documentación actualizada**: Mantener documentación sincronizada

### Performance
- **Tree shaking**: Estructura que permita tree shaking
- **Lazy loading**: Cargar componentes solo cuando sea necesario
- **Memoización**: Usar memoización estratégicamente
- **Bundle analysis**: Analizar el tamaño del bundle

### Testing
- **Cobertura completa**: Tests para todos los componentes
- **Tests de integración**: Verificar funcionamiento conjunto
- **Tests visuales**: Verificar renderizado correcto
- **Tests de accesibilidad**: Verificar cumplimiento de estándares

## Conclusión

Una arquitectura sólida de sistemas de componentes permite:

- **Escalabilidad** del sistema
- **Mantenibilidad** del código
- **Reutilización** máxima
- **Colaboración** efectiva entre equipos

En la siguiente clase exploraremos la implementación de patrones avanzados y la optimización de performance.

## Tarea

1. Implementar un sistema completo de componentes con la arquitectura propuesta
2. Crear un sistema de tokens con metadatos
3. Implementar un ThemeManager avanzado
4. Configurar un sistema de build y release
5. Crear documentación automática para todos los componentes

## Enlaces Útiles

- [Lerna](https://lerna.js.org/)
- [Monorepo Best Practices](https://monorepo.tools/)
- [Design System Architecture](https://bradfrost.com/blog/post/atomic-web-design/)
- [Component Library Best Practices](https://storybook.js.org/docs/react/writing-stories/introduction)
