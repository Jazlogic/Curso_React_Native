# 🧪 **Clase 1: Fundamentos de Testing** - React Native

## 📋 **Objetivos de la Clase**
- Comprender los fundamentos del testing en React Native
- Configurar Jest y React Testing Library
- Escribir primeros tests unitarios
- Entender conceptos básicos: test, describe, expect, matchers

## ⏱️ **Duración**
**1.5 horas**

## 🔧 **Configuración Inicial**

### Instalación de Dependencias
```bash
npm install --save-dev jest @testing-library/react-native @testing-library/jest-native
npm install --save-dev @babel/preset-env @babel/preset-react
```

### Configuración de Jest (jest.config.js)
```javascript
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  transformIgnorePatterns: [
    'node_modules/(?!(react-native|@react-native|@react-navigation)/)'
  ],
  testEnvironment: 'jsdom',
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.{js,jsx,ts,tsx}'
  ]
};
```

### Configuración de Jest Setup (jest.setup.js)
```javascript
import '@testing-library/jest-native/extend-expect';

// Mock de AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);

// Mock de react-native-gesture-handler
jest.mock('react-native-gesture-handler', () => {});

// Mock de react-native-reanimated
jest.mock('react-native-reanimated', () => {
  const Reanimated = require('react-native-reanimated/mock');
  Reanimated.default.call = () => {};
  return Reanimated;
});
```

## 📚 **Contenido Teórico**

### 1. **Conceptos Básicos de Testing**

#### **¿Qué es Testing?**
El testing es el proceso de verificar que nuestro código funciona correctamente y cumple con los requisitos esperados.

```typescript
// Ejemplo básico de test
describe('Calculadora', () => {
  test('debe sumar dos números correctamente', () => {
    // Arrange (Preparar)
    const a = 2;
    const b = 3;
    
    // Act (Actuar)
    const resultado = a + b;
    
    // Assert (Verificar)
    expect(resultado).toBe(5);
  });
});
```

#### **Estructura de un Test**
```typescript
describe('Grupo de Tests', () => {
  // beforeEach se ejecuta antes de cada test
  beforeEach(() => {
    // Configuración común
  });
  
  // afterEach se ejecuta después de cada test
  afterEach(() => {
    // Limpieza común
  });
  
  test('descripción del test', () => {
    // Cuerpo del test
  });
});
```

### 2. **Matchers de Jest**

#### **Matchers Básicos**
```typescript
describe('Matchers Básicos', () => {
  test('toBe - igualdad estricta', () => {
    expect(2 + 2).toBe(4);
    expect('hola').toBe('hola');
  });
  
  test('toEqual - igualdad profunda', () => {
    const objeto1 = { nombre: 'Juan', edad: 25 };
    const objeto2 = { nombre: 'Juan', edad: 25 };
    
    expect(objeto1).toEqual(objeto2);
    expect(objeto1).not.toBe(objeto2); // Referencias diferentes
  });
  
  test('toBeTruthy/toBeFalsy', () => {
    expect(true).toBeTruthy();
    expect(false).toBeFalsy();
    expect(0).toBeFalsy();
    expect('').toBeFalsy();
    expect(null).toBeFalsy();
    expect(undefined).toBeFalsy();
  });
});
```

#### **Matchers de Arrays y Objetos**
```typescript
describe('Matchers de Arrays y Objetos', () => {
  test('toContain - verificar elementos en array', () => {
    const frutas = ['manzana', 'banana', 'naranja'];
    
    expect(frutas).toContain('banana');
    expect(frutas).not.toContain('uva');
  });
  
  test('toHaveLength - verificar longitud', () => {
    const array = [1, 2, 3, 4, 5];
    
    expect(array).toHaveLength(5);
    expect(array).not.toHaveLength(3);
  });
  
  test('toHaveProperty - verificar propiedades de objeto', () => {
    const usuario = {
      id: 1,
      nombre: 'Ana',
      email: 'ana@email.com'
    };
    
    expect(usuario).toHaveProperty('nombre');
    expect(usuario).toHaveProperty('email', 'ana@email.com');
  });
});
```

### 3. **Testing de Funciones Simples**

#### **Función de Utilidad**
```typescript
// src/utils/mathUtils.ts
export const sumar = (a: number, b: number): number => {
  return a + b;
};

export const multiplicar = (a: number, b: number): number => {
  return a * b;
};

export const dividir = (a: number, b: number): number => {
  if (b === 0) {
    throw new Error('No se puede dividir por cero');
  }
  return a / b;
};

export const esPar = (numero: number): boolean => {
  return numero % 2 === 0;
};
```

#### **Tests para Funciones de Utilidad**
```typescript
// src/utils/__tests__/mathUtils.test.ts
import { sumar, multiplicar, dividir, esPar } from '../mathUtils';

describe('Math Utils', () => {
  describe('sumar', () => {
    test('debe sumar dos números positivos', () => {
      expect(sumar(2, 3)).toBe(5);
      expect(sumar(0, 5)).toBe(5);
      expect(sumar(-1, 1)).toBe(0);
    });
    
    test('debe sumar números decimales', () => {
      expect(sumar(1.5, 2.5)).toBe(4);
      expect(sumar(0.1, 0.2)).toBeCloseTo(0.3);
    });
  });
  
  describe('multiplicar', () => {
    test('debe multiplicar dos números', () => {
      expect(multiplicar(2, 3)).toBe(6);
      expect(multiplicar(0, 5)).toBe(0);
      expect(multiplicar(-2, 3)).toBe(-6);
    });
  });
  
  describe('dividir', () => {
    test('debe dividir dos números correctamente', () => {
      expect(dividir(6, 2)).toBe(3);
      expect(dividir(5, 2)).toBe(2.5);
    });
    
    test('debe lanzar error al dividir por cero', () => {
      expect(() => dividir(5, 0)).toThrow('No se puede dividir por cero');
      expect(() => dividir(5, 0)).toThrow(Error);
    });
  });
  
  describe('esPar', () => {
    test('debe identificar números pares e impares', () => {
      expect(esPar(2)).toBe(true);
      expect(esPar(4)).toBe(true);
      expect(esPar(1)).toBe(false);
      expect(esPar(3)).toBe(false);
      expect(esPar(0)).toBe(true);
    });
  });
});
```

### 4. **Testing de Async Functions**

#### **Función Async de Ejemplo**
```typescript
// src/services/userService.ts
export interface Usuario {
  id: number;
  nombre: string;
  email: string;
}

export class UserService {
  async obtenerUsuario(id: number): Promise<Usuario> {
    // Simular llamada a API
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (id > 0) {
          resolve({
            id,
            nombre: `Usuario ${id}`,
            email: `usuario${id}@email.com`
          });
        } else {
          reject(new Error('ID de usuario inválido'));
        }
      }, 100);
    });
  }
  
  async obtenerUsuarios(): Promise<Usuario[]> {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve([
          { id: 1, nombre: 'Ana', email: 'ana@email.com' },
          { id: 2, nombre: 'Juan', email: 'juan@email.com' },
          { id: 3, nombre: 'María', email: 'maria@email.com' }
        ]);
      }, 100);
    });
  }
}
```

#### **Tests para Funciones Async**
```typescript
// src/services/__tests__/userService.test.ts
import { UserService, Usuario } from '../userService';

describe('UserService', () => {
  let userService: UserService;
  
  beforeEach(() => {
    userService = new UserService();
  });
  
  describe('obtenerUsuario', () => {
    test('debe obtener usuario por ID válido', async () => {
      const usuario = await userService.obtenerUsuario(1);
      
      expect(usuario).toEqual({
        id: 1,
        nombre: 'Usuario 1',
        email: 'usuario1@email.com'
      });
    });
    
    test('debe lanzar error para ID inválido', async () => {
      await expect(userService.obtenerUsuario(0)).rejects.toThrow('ID de usuario inválido');
      await expect(userService.obtenerUsuario(-1)).rejects.toThrow('ID de usuario inválido');
    });
  });
  
  describe('obtenerUsuarios', () => {
    test('debe obtener lista de usuarios', async () => {
      const usuarios = await userService.obtenerUsuarios();
      
      expect(usuarios).toHaveLength(3);
      expect(usuarios[0]).toHaveProperty('id', 1);
      expect(usuarios[0]).toHaveProperty('nombre', 'Ana');
      expect(usuarios[0]).toHaveProperty('email', 'ana@email.com');
    });
    
    test('debe retornar array de usuarios con estructura correcta', async () => {
      const usuarios = await userService.obtenerUsuarios();
      
      usuarios.forEach(usuario => {
        expect(usuario).toHaveProperty('id');
        expect(usuario).toHaveProperty('nombre');
        expect(usuario).toHaveProperty('email');
        expect(typeof usuario.id).toBe('number');
        expect(typeof usuario.nombre).toBe('string');
        expect(typeof usuario.email).toBe('string');
      });
    });
  });
});
```

### 5. **Mocks y Stubs**

#### **Mock de Dependencias Externas**
```typescript
// src/services/__tests__/userService.test.ts
import { UserService } from '../userService';

// Mock de fetch global
global.fetch = jest.fn();

describe('UserService con Mock de Fetch', () => {
  let userService: UserService;
  
  beforeEach(() => {
    userService = new UserService();
    // Limpiar todos los mocks antes de cada test
    jest.clearAllMocks();
  });
  
  test('debe hacer llamada a API correcta', async () => {
    // Configurar mock de fetch
    const mockResponse = {
      id: 1,
      nombre: 'Ana',
      email: 'ana@email.com'
    };
    
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockResponse
    });
    
    // Llamar al método que usa fetch
    const resultado = await userService.obtenerUsuarioDesdeAPI(1);
    
    // Verificar que fetch fue llamado con los parámetros correctos
    expect(global.fetch).toHaveBeenCalledWith('https://api.ejemplo.com/usuarios/1');
    expect(global.fetch).toHaveBeenCalledTimes(1);
    expect(resultado).toEqual(mockResponse);
  });
  
  test('debe manejar errores de API', async () => {
    // Mock de fetch que falla
    (global.fetch as jest.Mock).mockRejectedValueOnce(new Error('Error de red'));
    
    await expect(userService.obtenerUsuarioDesdeAPI(1)).rejects.toThrow('Error de red');
  });
});
```

## 🧪 **Ejercicios Prácticos**

### **Ejercicio 1: Testing de Validaciones**
Crea tests para una función de validación de email:

```typescript
// Función a testear
export const validarEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Escribe tests que cubran:
// - Emails válidos
// - Emails inválidos
// - Casos edge (string vacío, null, undefined)
```

### **Ejercicio 2: Testing de Formateo de Fechas**
Crea tests para una función de formateo de fechas:

```typescript
// Función a testear
export const formatearFecha = (fecha: Date, formato: 'corta' | 'larga'): string => {
  if (formato === 'corta') {
    return fecha.toLocaleDateString('es-ES');
  } else {
    return fecha.toLocaleDateString('es-ES', {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  }
};

// Escribe tests que cubran:
// - Formato corto
// - Formato largo
// - Diferentes fechas
// - Manejo de errores
```

### **Ejercicio 3: Testing de Lógica de Negocio**
Crea tests para una función de cálculo de descuentos:

```typescript
// Función a testear
export const calcularDescuento = (
  precio: number, 
  porcentajeDescuento: number, 
  esClientePremium: boolean
): number => {
  if (precio <= 0 || porcentajeDescuento < 0 || porcentajeDescuento > 100) {
    throw new Error('Parámetros inválidos');
  }
  
  let descuento = precio * (porcentajeDescuento / 100);
  
  if (esClientePremium) {
    descuento *= 1.1; // 10% extra para clientes premium
  }
  
  return Math.round(descuento * 100) / 100; // Redondear a 2 decimales
};

// Escribe tests que cubran:
// - Cálculo básico de descuento
// - Descuento premium
// - Validación de parámetros
// - Casos edge
```

## 📝 **Resumen de la Clase**

### **Conceptos Clave Aprendidos**
1. **Testing Unitario**: Verificación de funciones individuales
2. **Jest**: Framework de testing con matchers poderosos
3. **Estructura de Tests**: describe, test, beforeEach, afterEach
4. **Matchers**: toBe, toEqual, toContain, toHaveProperty
5. **Async Testing**: Manejo de promesas y funciones asíncronas
6. **Mocks**: Simulación de dependencias externas

### **Próximos Pasos**
- Testing de componentes React Native
- Testing de hooks personalizados
- Testing de integración y APIs

## 🔗 **Navegación**
- [⬅️ Clase Anterior](../senior_1/clase_5_arquitectura_patrones_compuestos.md) - Arquitectura y Patrones Compuestos
- [🏠 Inicio](../../README.md)
- [📚 Índice Completo](../../INDICE_COMPLETO.md)
- [➡️ Siguiente Clase](clase_2_testing_componentes.md) - Testing de Componentes

---

**💡 Tip**: Escribe tests mientras desarrollas tu código, no después. Esto te ayudará a pensar mejor en el diseño y detectar problemas temprano.
