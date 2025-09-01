# 🧪 **Clase 4: Testing de Integración y APIs** - React Native

## 📋 **Objetivos de la Clase**
- Aprender testing de integración entre componentes
- Testing de APIs y servicios externos
- Implementar mocks para servicios de red
- Testing de flujos completos de usuario

## ⏱️ **Duración**
**2 horas**

## 🔧 **Configuración para Testing de Integración**

### Instalación de Dependencias
```bash
npm install --save-dev msw msw-node
npm install --save-dev @testing-library/react-native
```

### Configuración de MSW (Mock Service Worker)
```javascript
// src/test/setup.ts
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## 📚 **Contenido Teórico**

### 1. **Testing de Integración de Componentes**

#### **Componente de Gestión de Usuarios**
```typescript
// src/components/UserManagement.tsx
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { UserList } from './UserList';
import { UserForm } from './UserForm';
import { UserService } from '../services/UserService';

interface Usuario {
  id: number;
  nombre: string;
  email: string;
  activo: boolean;
}

export const UserManagement: React.FC = () => {
  const [usuarios, setUsuarios] = useState<Usuario[]>([]);
  const [loading, setLoading] = useState(true);
  const [showForm, setShowForm] = useState(false);
  const [editingUser, setEditingUser] = useState<Usuario | null>(null);
  
  const userService = new UserService();
  
  useEffect(() => {
    loadUsers();
  }, []);
  
  const loadUsers = async () => {
    try {
      setLoading(true);
      const data = await userService.getUsers();
      setUsuarios(data);
    } catch (error) {
      Alert.alert('Error', 'No se pudieron cargar los usuarios');
    } finally {
      setLoading(false);
    }
  };
  
  const handleUserPress = (usuario: Usuario) => {
    setEditingUser(usuario);
    setShowForm(true);
  };
  
  const handleUserDelete = async (id: number) => {
    try {
      await userService.deleteUser(id);
      setUsuarios(prev => prev.filter(u => u.id !== id));
      Alert.alert('Éxito', 'Usuario eliminado correctamente');
    } catch (error) {
      Alert.alert('Error', 'No se pudo eliminar el usuario');
    }
  };
  
  const handleFormSubmit = async (userData: Omit<Usuario, 'id'>) => {
    try {
      if (editingUser) {
        const updatedUser = await userService.updateUser(editingUser.id, userData);
        setUsuarios(prev => 
          prev.map(u => u.id === editingUser.id ? updatedUser : u)
        );
        Alert.alert('Éxito', 'Usuario actualizado correctamente');
      } else {
        const newUser = await userService.createUser(userData);
        setUsuarios(prev => [...prev, newUser]);
        Alert.alert('Éxito', 'Usuario creado correctamente');
      }
      
      setShowForm(false);
      setEditingUser(null);
    } catch (error) {
      Alert.alert('Error', 'No se pudo guardar el usuario');
    }
  };
  
  const handleFormCancel = () => {
    setShowForm(false);
    setEditingUser(null);
  };
  
  if (loading) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.loadingText}>Cargando usuarios...</Text>
      </View>
    );
  }
  
  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Gestión de Usuarios</Text>
        <TouchableOpacity
          style={styles.addButton}
          onPress={() => setShowForm(true)}
          testID="add-user-button"
        >
          <Text style={styles.addButtonText}>+ Agregar Usuario</Text>
        </TouchableOpacity>
      </View>
      
      {showForm ? (
        <UserForm
          user={editingUser}
          onSubmit={handleFormSubmit}
          onCancel={handleFormCancel}
        />
      ) : (
        <UserList
          usuarios={usuarios}
          onUserPress={handleUserPress}
          onUserDelete={handleUserDelete}
        />
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 20,
  },
  title: { fontSize: 24, fontWeight: 'bold' },
  addButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 8,
  },
  addButtonText: { color: '#FFFFFF', fontWeight: '600' },
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: { fontSize: 16, color: '#666666' },
});
```

#### **Tests de Integración**
```typescript
// src/components/__tests__/UserManagement.integration.test.tsx
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { UserManagement } from '../UserManagement';
import { UserService } from '../../services/UserService';

jest.mock('../../services/UserService');
const MockedUserService = UserService as jest.MockedClass<typeof UserService>;

jest.mock('react-native', () => {
  const RN = jest.requireActual('react-native');
  return {
    ...RN,
    Alert: { alert: jest.fn() },
  };
});

describe('UserManagement Integration Tests', () => {
  let mockUserService: jest.Mocked<UserService>;
  
  const mockUsers = [
    { id: 1, nombre: 'Ana García', email: 'ana@email.com', activo: true },
    { id: 2, nombre: 'Juan López', email: 'juan@email.com', activo: false }
  ];
  
  beforeEach(() => {
    mockUserService = {
      getUsers: jest.fn(),
      createUser: jest.fn(),
      updateUser: jest.fn(),
      deleteUser: jest.fn(),
    } as any;
    
    MockedUserService.mockImplementation(() => mockUserService);
  });
  
  test('debe cargar y mostrar usuarios al inicializar', async () => {
    mockUserService.getUsers.mockResolvedValue(mockUsers);
    
    const { getByText } = render(<UserManagement />);
    
    expect(getByText('Cargando usuarios...')).toBeTruthy();
    
    await waitFor(() => {
      expect(getByText('Ana García')).toBeTruthy();
      expect(getByText('Juan López')).toBeTruthy();
    });
    
    expect(mockUserService.getUsers).toHaveBeenCalledTimes(1);
  });
  
  test('debe crear nuevo usuario exitosamente', async () => {
    mockUserService.getUsers.mockResolvedValue(mockUsers);
    const newUser = { id: 3, nombre: 'María', email: 'maria@email.com', activo: true };
    mockUserService.createUser.mockResolvedValue(newUser);
    
    const { getByTestId, getByText, getByPlaceholderText } = render(<UserManagement />);
    
    await waitFor(() => {
      expect(getByText('Ana García')).toBeTruthy();
    });
    
    const addButton = getByTestId('add-user-button');
    fireEvent.press(addButton);
    
    const nameInput = getByPlaceholderText('Nombre');
    const emailInput = getByPlaceholderText('Email');
    
    fireEvent.changeText(nameInput, 'María');
    fireEvent.changeText(emailInput, 'maria@email.com');
    
    const submitButton = getByText('Guardar');
    fireEvent.press(submitButton);
    
    await waitFor(() => {
      expect(mockUserService.createUser).toHaveBeenCalledWith({
        nombre: 'María',
        email: 'maria@email.com',
        activo: true
      });
    });
    
    await waitFor(() => {
      expect(getByText('María')).toBeTruthy();
    });
  });
  
  test('debe eliminar usuario exitosamente', async () => {
    mockUserService.getUsers.mockResolvedValue(mockUsers);
    mockUserService.deleteUser.mockResolvedValue(undefined);
    
    const { getByText } = render(<UserManagement />);
    
    await waitFor(() => {
      expect(getByText('Ana García')).toBeTruthy();
      expect(getByText('Juan López')).toBeTruthy();
    });
    
    const deleteButton = getByText('Eliminar');
    fireEvent.press(deleteButton);
    
    await waitFor(() => {
      expect(mockUserService.deleteUser).toHaveBeenCalledWith(1);
    });
    
    await waitFor(() => {
      expect(queryByText('Ana García')).toBeFalsy();
      expect(getByText('Juan López')).toBeTruthy();
    });
  });
});
```

### 2. **Testing de APIs con MSW**

#### **Servicio de Usuarios**
```typescript
// src/services/UserService.ts
export interface Usuario {
  id: number;
  nombre: string;
  email: string;
  activo: boolean;
}

export class UserService {
  private baseUrl = '/api/users';
  
  async getUsers(): Promise<Usuario[]> {
    const response = await fetch(this.baseUrl);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
  
  async createUser(userData: Omit<Usuario, 'id'>): Promise<Usuario> {
    const response = await fetch(this.baseUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || `HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
  
  async updateUser(id: number, userData: Partial<Usuario>): Promise<Usuario> {
    const response = await fetch(`${this.baseUrl}/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || `HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
  
  async deleteUser(id: number): Promise<void> {
    const response = await fetch(`${this.baseUrl}/${id}`, { method: 'DELETE' });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
  }
}
```

#### **Tests de API con MSW**
```typescript
// src/services/__tests__/UserService.api.test.ts
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { UserService } from '../UserService';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, nombre: 'Ana García', email: 'ana@email.com', activo: true },
        { id: 2, nombre: 'Juan López', email: 'juan@email.com', activo: false }
      ])
    );
  }),
  
  rest.post('/api/users', async (req, res, ctx) => {
    const body = await req.json();
    
    if (!body.nombre || !body.email) {
      return res(
        ctx.status(400),
        ctx.json({ message: 'Nombre y email son requeridos' })
      );
    }
    
    return res(
      ctx.status(201),
      ctx.json({ id: 3, ...body })
    );
  }),
  
  rest.put('/api/users/:id', async (req, res, ctx) => {
    const { id } = req.params;
    const body = await req.json();
    
    return res(
      ctx.json({ id: parseInt(id), ...body })
    );
  }),
  
  rest.delete('/api/users/:id', (req, res, ctx) => {
    return res(ctx.status(204));
  })
);

describe('UserService API Tests', () => {
  let userService: UserService;
  
  beforeAll(() => server.listen());
  afterEach(() => server.resetHandlers());
  afterAll(() => server.close());
  
  beforeEach(() => {
    userService = new UserService();
  });
  
  test('debe obtener lista de usuarios exitosamente', async () => {
    const users = await userService.getUsers();
    
    expect(users).toHaveLength(2);
    expect(users[0]).toEqual({
      id: 1,
      nombre: 'Ana García',
      email: 'ana@email.com',
      activo: true
    });
  });
  
  test('debe crear usuario exitosamente', async () => {
    const userData = {
      nombre: 'María Rodríguez',
      email: 'maria@email.com',
      activo: true
    };
    
    const newUser = await userService.createUser(userData);
    
    expect(newUser).toEqual({
      id: 3,
      ...userData
    });
  });
  
  test('debe validar datos requeridos', async () => {
    const invalidData = {
      nombre: '',
      email: '',
      activo: true
    };
    
    await expect(userService.createUser(invalidData)).rejects.toThrow(
      'Nombre y email son requeridos'
    );
  });
  
  test('debe actualizar usuario exitosamente', async () => {
    const updateData = { nombre: 'Ana García López' };
    
    const updatedUser = await userService.updateUser(1, updateData);
    
    expect(updatedUser).toEqual({
      id: 1,
      nombre: 'Ana García López',
      email: 'ana@email.com',
      activo: true
    });
  });
  
  test('debe eliminar usuario exitosamente', async () => {
    await expect(userService.deleteUser(1)).resolves.not.toThrow();
  });
});
```

### 3. **Testing de Flujos Completos**

#### **Test de Flujo de Creación de Usuario**
```typescript
// src/components/__tests__/UserManagement.flow.test.tsx
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { UserManagement } from '../UserManagement';
import { UserService } from '../../services/UserService';

jest.mock('../../services/UserService');
const MockedUserService = UserService as jest.MockedClass<typeof UserService>;

describe('UserManagement User Flow Tests', () => {
  let mockUserService: jest.Mocked<UserService>;
  
  beforeEach(() => {
    mockUserService = {
      getUsers: jest.fn(),
      createUser: jest.fn(),
      updateUser: jest.fn(),
      deleteUser: jest.fn(),
    } as any;
    
    MockedUserService.mockImplementation(() => mockUserService);
  });
  
  test('flujo completo de creación de usuario', async () => {
    mockUserService.getUsers.mockResolvedValue([]);
    
    const { getByText, getByTestId, getByPlaceholderText } = render(
      <UserManagement />
    );
    
    await waitFor(() => {
      expect(getByText('No hay usuarios disponibles')).toBeTruthy();
    });
    
    const addButton = getByTestId('add-user-button');
    fireEvent.press(addButton);
    
    expect(getByText('Crear Usuario')).toBeTruthy();
    
    const nameInput = getByPlaceholderText('Nombre');
    const emailInput = getByPlaceholderText('Email');
    
    fireEvent.changeText(nameInput, 'Nuevo Usuario');
    fireEvent.changeText(emailInput, 'nuevo@email.com');
    
    const submitButton = getByText('Guardar');
    
    const newUser = {
      id: 1,
      nombre: 'Nuevo Usuario',
      email: 'nuevo@email.com',
      activo: true
    };
    mockUserService.createUser.mockResolvedValue(newUser);
    
    fireEvent.press(submitButton);
    
    await waitFor(() => {
      expect(mockUserService.createUser).toHaveBeenCalledWith({
        nombre: 'Nuevo Usuario',
        email: 'nuevo@email.com',
        activo: true
      });
    });
    
    await waitFor(() => {
      expect(getByText('Nuevo Usuario')).toBeTruthy();
    });
  });
  
  test('flujo completo de edición de usuario', async () => {
    const existingUsers = [
      { id: 1, nombre: 'Usuario Original', email: 'original@email.com', activo: true }
    ];
    
    mockUserService.getUsers.mockResolvedValue(existingUsers);
    
    const { getByText, getByPlaceholderText } = render(<UserManagement />);
    
    await waitFor(() => {
      expect(getByText('Usuario Original')).toBeTruthy();
    });
    
    const userItem = getByText('Usuario Original');
    fireEvent.press(userItem);
    
    expect(getByText('Editar Usuario')).toBeTruthy();
    
    const nameInput = getByPlaceholderText('Nombre');
    fireEvent.changeText(nameInput, 'Usuario Modificado');
    
    const submitButton = getByText('Guardar');
    
    const updatedUser = {
      id: 1,
      nombre: 'Usuario Modificado',
      email: 'original@email.com',
      activo: true
    };
    mockUserService.updateUser.mockResolvedValue(updatedUser);
    
    fireEvent.press(submitButton);
    
    await waitFor(() => {
      expect(mockUserService.updateUser).toHaveBeenCalledWith(1, {
        nombre: 'Usuario Modificado',
        email: 'original@email.com',
        activo: true
      });
    });
    
    await waitFor(() => {
      expect(getByText('Usuario Modificado')).toBeTruthy();
    });
  });
});
```

## 🧪 **Ejercicios Prácticos**

### **Ejercicio 1: Testing de Flujo de Autenticación**
Crea tests para un flujo completo de login/logout:

```typescript
// Componente a testear
export const AuthFlow: React.FC = () => {
  // Implementa:
  // - Formulario de login
  // - Validación de credenciales
  // - Redirección post-login
  // - Logout y limpieza de estado
};

// Escribe tests que cubran:
// - Flujo completo de login exitoso
// - Manejo de credenciales inválidas
// - Persistencia de sesión
// - Flujo de logout
```

### **Ejercicio 2: Testing de Flujo de Checkout**
Crea tests para un flujo de compra:

```typescript
// Componente a testear
export const CheckoutFlow: React.FC = () => {
  // Implementa:
  // - Selección de productos
  // - Validación de inventario
  // - Proceso de pago
  // - Confirmación de orden
};

// Escribe tests que cubran:
// - Flujo completo de compra
// - Validaciones de inventario
// - Manejo de errores de pago
// - Confirmación de orden
```

## 📝 **Resumen de la Clase**

### **Conceptos Clave Aprendidos**
1. **Testing de Integración**: Verificación de interacciones entre componentes
2. **MSW (Mock Service Worker)**: Simulación de APIs reales
3. **Testing de Flujos**: Verificación de casos de uso completos
4. **Testing de APIs**: Verificación de servicios y endpoints
5. **Mocks de Servicios**: Simulación de dependencias externas
6. **Testing de Estados**: Verificación de cambios de estado complejos

### **Próximos Pasos**
- Testing e2e y performance
- Estrategias de testing y mejores prácticas
- Cobertura de tests y CI/CD

## 🔗 **Navegación**
- [⬅️ Clase Anterior](clase_3_testing_hooks_logica.md) - Testing de Hooks y Lógica
- [🏠 Inicio](../../README.md)
- [📚 Índice Completo](../../INDICE_COMPLETO.md)
- [➡️ Siguiente Clase](clase_5_testing_e2e_performance.md) - Testing E2E y Performance

---

**💡 Tip**: Usa MSW para simular APIs reales en tus tests. Esto te permite testear integraciones sin depender de servicios externos y te da control total sobre las respuestas.
