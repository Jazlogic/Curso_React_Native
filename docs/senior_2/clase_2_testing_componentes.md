# 🧪 **Clase 2: Testing de Componentes** - React Native

## 📋 **Objetivos de la Clase**
- Aprender a testear componentes React Native
- Usar React Testing Library para testing de componentes
- Implementar mocks para dependencias externas
- Testing de props, eventos y estado de componentes

## ⏱️ **Duración**
**2 horas**

## 🔧 **Configuración de React Testing Library**

### Instalación Adicional
```bash
npm install --save-dev @testing-library/react-native @testing-library/jest-native
npm install --save-dev react-test-renderer
```

### Configuración de Jest para Componentes
```javascript
// jest.setup.js
import '@testing-library/jest-native/extend-expect';

// Mock de react-native
jest.mock('react-native', () => {
  const RN = jest.requireActual('react-native');
  
  return {
    ...RN,
    // Mock de componentes nativos
    Text: 'Text',
    View: 'View',
    TouchableOpacity: 'TouchableOpacity',
    TextInput: 'TextInput',
    ScrollView: 'ScrollView',
    FlatList: 'FlatList',
    Image: 'Image',
    // Mock de APIs nativas
    Alert: {
      alert: jest.fn(),
    },
    Platform: {
      OS: 'ios',
      select: jest.fn((obj) => obj.ios),
    },
  };
});

// Mock de react-native-vector-icons
jest.mock('react-native-vector-icons/MaterialIcons', () => 'Icon');
```

## 📚 **Contenido Teórico**

### 1. **Testing de Componentes Básicos**

#### **Componente Simple a Testear**
```typescript
// src/components/Button.tsx
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

interface ButtonProps {
  title: string;
  onPress: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
}

export const Button: React.FC<ButtonProps> = ({
  title,
  onPress,
  disabled = false,
  variant = 'primary'
}) => {
  return (
    <TouchableOpacity
      style={[
        styles.button,
        styles[variant],
        disabled && styles.disabled
      ]}
      onPress={onPress}
      disabled={disabled}
      testID="custom-button"
    >
      <Text style={[
        styles.text,
        styles[`${variant}Text`],
        disabled && styles.disabledText
      ]}>
        {title}
      </Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
    minWidth: 120,
  },
  primary: {
    backgroundColor: '#007AFF',
  },
  secondary: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#007AFF',
  },
  disabled: {
    backgroundColor: '#E5E5EA',
    borderColor: '#E5E5EA',
  },
  text: {
    fontSize: 16,
    fontWeight: '600',
  },
  primaryText: {
    color: '#FFFFFF',
  },
  secondaryText: {
    color: '#007AFF',
  },
  disabledText: {
    color: '#8E8E93',
  },
});
```

#### **Tests para el Componente Button**
```typescript
// src/components/__tests__/Button.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { Button } from '../Button';

describe('Button Component', () => {
  const mockOnPress = jest.fn();
  
  beforeEach(() => {
    mockOnPress.mockClear();
  });
  
  test('debe renderizar con título correcto', () => {
    const { getByText } = render(
      <Button title="Presionar" onPress={mockOnPress} />
    );
    
    expect(getByText('Presionar')).toBeTruthy();
  });
  
  test('debe llamar onPress cuando se presiona', () => {
    const { getByTestId } = render(
      <Button title="Presionar" onPress={mockOnPress} />
    );
    
    const button = getByTestId('custom-button');
    fireEvent.press(button);
    
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });
  
  test('debe aplicar estilos de variante primary por defecto', () => {
    const { getByTestId } = render(
      <Button title="Presionar" onPress={mockOnPress} />
    );
    
    const button = getByTestId('custom-button');
    expect(button.props.style).toContainEqual(
      expect.objectContaining({ backgroundColor: '#007AFF' })
    );
  });
  
  test('debe aplicar estilos de variante secondary', () => {
    const { getByTestId } = render(
      <Button 
        title="Presionar" 
        onPress={mockOnPress} 
        variant="secondary" 
      />
    );
    
    const button = getByTestId('custom-button');
    expect(button.props.style).toContainEqual(
      expect.objectContaining({ 
        backgroundColor: 'transparent',
        borderWidth: 1,
        borderColor: '#007AFF'
      })
    );
  });
  
  test('debe estar deshabilitado cuando disabled es true', () => {
    const { getByTestId } = render(
      <Button 
        title="Presionar" 
        onPress={mockOnPress} 
        disabled={true} 
      />
    );
    
    const button = getByTestId('custom-button');
    expect(button.props.disabled).toBe(true);
  });
  
  test('no debe llamar onPress cuando está deshabilitado', () => {
    const { getByTestId } = render(
      <Button 
        title="Presionar" 
        onPress={mockOnPress} 
        disabled={true} 
      />
    );
    
    const button = getByTestId('custom-button');
    fireEvent.press(button);
    
    expect(mockOnPress).not.toHaveBeenCalled();
  });
});
```

### 2. **Testing de Componentes con Estado**

#### **Componente con Estado Interno**
```typescript
// src/components/Counter.tsx
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

export const Counter: React.FC = () => {
  const [count, setCount] = useState(0);
  
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(0);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Contador</Text>
      <Text style={styles.count} testID="count-display">{count}</Text>
      
      <View style={styles.buttonContainer}>
        <TouchableOpacity 
          style={styles.button} 
          onPress={decrement}
          testID="decrement-button"
        >
          <Text style={styles.buttonText}>-</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={styles.button} 
          onPress={reset}
          testID="reset-button"
        >
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
        
        <TouchableOpacity 
          style={styles.button} 
          onPress={increment}
          testID="increment-button"
        >
          <Text style={styles.buttonText}>+</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  count: {
    fontSize: 48,
    fontWeight: 'bold',
    color: '#007AFF',
    marginBottom: 30,
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 10,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    minWidth: 60,
    alignItems: 'center',
  },
  buttonText: {
    color: '#FFFFFF',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

#### **Tests para el Componente Counter**
```typescript
// src/components/__tests__/Counter.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { Counter } from '../Counter';

describe('Counter Component', () => {
  test('debe renderizar con contador en 0', () => {
    const { getByTestId } = render(<Counter />);
    
    const countDisplay = getByTestId('count-display');
    expect(countDisplay.props.children).toBe(0);
  });
  
  test('debe incrementar contador al presionar botón +', () => {
    const { getByTestId } = render(<Counter />);
    
    const incrementButton = getByTestId('increment-button');
    const countDisplay = getByTestId('count-display');
    
    fireEvent.press(incrementButton);
    expect(countDisplay.props.children).toBe(1);
    
    fireEvent.press(incrementButton);
    expect(countDisplay.props.children).toBe(2);
  });
  
  test('debe decrementar contador al presionar botón -', () => {
    const { getByTestId } = render(<Counter />);
    
    const incrementButton = getByTestId('increment-button');
    const decrementButton = getByTestId('decrement-button');
    const countDisplay = getByTestId('count-display');
    
    // Incrementar primero
    fireEvent.press(incrementButton);
    fireEvent.press(incrementButton);
    expect(countDisplay.props.children).toBe(2);
    
    // Decrementar
    fireEvent.press(decrementButton);
    expect(countDisplay.props.children).toBe(1);
    
    fireEvent.press(decrementButton);
    expect(countDisplay.props.children).toBe(0);
  });
  
  test('debe resetear contador al presionar botón Reset', () => {
    const { getByTestId } = render(<Counter />);
    
    const incrementButton = getByTestId('increment-button');
    const resetButton = getByTestId('reset-button');
    const countDisplay = getByTestId('count-display');
    
    // Incrementar varias veces
    fireEvent.press(incrementButton);
    fireEvent.press(incrementButton);
    fireEvent.press(incrementButton);
    expect(countDisplay.props.children).toBe(3);
    
    // Resetear
    fireEvent.press(resetButton);
    expect(countDisplay.props.children).toBe(0);
  });
  
  test('debe permitir contador negativo', () => {
    const { getByTestId } = render(<Counter />);
    
    const decrementButton = getByTestId('decrement-button');
    const countDisplay = getByTestId('count-display');
    
    fireEvent.press(decrementButton);
    expect(countDisplay.props.children).toBe(-1);
    
    fireEvent.press(decrementButton);
    expect(countDisplay.props.children).toBe(-2);
  });
});
```

### 3. **Testing de Componentes con Props Complejas**

#### **Componente de Lista de Usuarios**
```typescript
// src/components/UserList.tsx
import React from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet } from 'react-native';

interface Usuario {
  id: number;
  nombre: string;
  email: string;
  activo: boolean;
}

interface UserListProps {
  usuarios: Usuario[];
  onUserPress: (usuario: Usuario) => void;
  onUserDelete: (id: number) => void;
  isLoading?: boolean;
  emptyMessage?: string;
}

export const UserList: React.FC<UserListProps> = ({
  usuarios,
  onUserPress,
  onUserDelete,
  isLoading = false,
  emptyMessage = 'No hay usuarios disponibles'
}) => {
  const renderUserItem = ({ item }: { item: Usuario }) => (
    <TouchableOpacity
      style={[styles.userItem, !item.activo && styles.inactiveUser]}
      onPress={() => onUserPress(item)}
      testID={`user-item-${item.id}`}
    >
      <View style={styles.userInfo}>
        <Text style={styles.userName}>{item.nombre}</Text>
        <Text style={styles.userEmail}>{item.email}</Text>
        <Text style={[
          styles.userStatus,
          item.activo ? styles.activeStatus : styles.inactiveStatus
        ]}>
          {item.activo ? 'Activo' : 'Inactivo'}
        </Text>
      </View>
      
      <TouchableOpacity
        style={styles.deleteButton}
        onPress={() => onUserDelete(item.id)}
        testID={`delete-button-${item.id}`}
      >
        <Text style={styles.deleteButtonText}>Eliminar</Text>
      </TouchableOpacity>
    </TouchableOpacity>
  );
  
  if (isLoading) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.loadingText}>Cargando usuarios...</Text>
      </View>
    );
  }
  
  if (usuarios.length === 0) {
    return (
      <View style={styles.centerContainer}>
        <Text style={styles.emptyText}>{emptyMessage}</Text>
      </View>
    );
  }
  
  return (
    <FlatList
      data={usuarios}
      renderItem={renderUserItem}
      keyExtractor={(item) => item.id.toString()}
      testID="user-list"
      contentContainerStyle={styles.listContainer}
    />
  );
};

const styles = StyleSheet.create({
  listContainer: {
    padding: 16,
  },
  userItem: {
    flexDirection: 'row',
    backgroundColor: '#FFFFFF',
    padding: 16,
    marginBottom: 8,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#E5E5EA',
    alignItems: 'center',
  },
  inactiveUser: {
    opacity: 0.6,
  },
  userInfo: {
    flex: 1,
  },
  userName: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 4,
  },
  userEmail: {
    fontSize: 14,
    color: '#666666',
    marginBottom: 4,
  },
  userStatus: {
    fontSize: 12,
    fontWeight: '500',
  },
  activeStatus: {
    color: '#34C759',
  },
  inactiveStatus: {
    color: '#FF3B30',
  },
  deleteButton: {
    backgroundColor: '#FF3B30',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 6,
  },
  deleteButtonText: {
    color: '#FFFFFF',
    fontSize: 12,
    fontWeight: '600',
  },
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  loadingText: {
    fontSize: 16,
    color: '#666666',
  },
  emptyText: {
    fontSize: 16,
    color: '#666666',
    textAlign: 'center',
  },
});
```

#### **Tests para el Componente UserList**
```typescript
// src/components/__tests__/UserList.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { UserList } from '../UserList';

const mockUsuarios = [
  {
    id: 1,
    nombre: 'Ana García',
    email: 'ana@email.com',
    activo: true
  },
  {
    id: 2,
    nombre: 'Juan López',
    email: 'juan@email.com',
    activo: false
  },
  {
    id: 3,
    nombre: 'María Rodríguez',
    email: 'maria@email.com',
    activo: true
  }
];

describe('UserList Component', () => {
  const mockOnUserPress = jest.fn();
  const mockOnUserDelete = jest.fn();
  
  beforeEach(() => {
    mockOnUserPress.mockClear();
    mockOnUserDelete.mockClear();
  });
  
  test('debe renderizar lista de usuarios', () => {
    const { getByTestId, getAllByTestId } = render(
      <UserList
        usuarios={mockUsuarios}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
      />
    );
    
    const userList = getByTestId('user-list');
    const userItems = getAllByTestId(/user-item-/);
    
    expect(userList).toBeTruthy();
    expect(userItems).toHaveLength(3);
  });
  
  test('debe mostrar información correcta de cada usuario', () => {
    const { getByText } = render(
      <UserList
        usuarios={mockUsuarios}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
      />
    );
    
    expect(getByText('Ana García')).toBeTruthy();
    expect(getByText('ana@email.com')).toBeTruthy();
    expect(getByText('Activo')).toBeTruthy();
    
    expect(getByText('Juan López')).toBeTruthy();
    expect(getByText('juan@email.com')).toBeTruthy();
    expect(getByText('Inactivo')).toBeTruthy();
  });
  
  test('debe llamar onUserPress al presionar usuario', () => {
    const { getByTestId } = render(
      <UserList
        usuarios={mockUsuarios}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
      />
    );
    
    const userItem = getByTestId('user-item-1');
    fireEvent.press(userItem);
    
    expect(mockOnUserPress).toHaveBeenCalledWith(mockUsuarios[0]);
  });
  
  test('debe llamar onUserDelete al presionar botón eliminar', () => {
    const { getByTestId } = render(
      <UserList
        usuarios={mockUsuarios}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
      />
    );
    
    const deleteButton = getByTestId('delete-button-2');
    fireEvent.press(deleteButton);
    
    expect(mockOnUserDelete).toHaveBeenCalledWith(2);
  });
  
  test('debe mostrar mensaje de carga cuando isLoading es true', () => {
    const { getByText, queryByTestId } = render(
      <UserList
        usuarios={mockUsuarios}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
        isLoading={true}
      />
    );
    
    expect(getByText('Cargando usuarios...')).toBeTruthy();
    expect(queryByTestId('user-list')).toBeNull();
  });
  
  test('debe mostrar mensaje vacío cuando no hay usuarios', () => {
    const { getByText, queryByTestId } = render(
      <UserList
        usuarios={[]}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
      />
    );
    
    expect(getByText('No hay usuarios disponibles')).toBeTruthy();
    expect(queryByTestId('user-list')).toBeNull();
  });
  
  test('debe mostrar mensaje personalizado cuando no hay usuarios', () => {
    const { getByText } = render(
      <UserList
        usuarios={[]}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
        emptyMessage="No se encontraron usuarios"
      />
    );
    
    expect(getByText('No se encontraron usuarios')).toBeTruthy();
  });
  
  test('debe aplicar estilos diferentes para usuarios inactivos', () => {
    const { getByTestId } = render(
      <UserList
        usuarios={mockUsuarios}
        onUserPress={mockOnUserPress}
        onUserDelete={mockOnUserDelete}
      />
    );
    
    const activeUser = getByTestId('user-item-1');
    const inactiveUser = getByTestId('user-item-2');
    
    // Verificar que el usuario inactivo tiene opacidad reducida
    expect(inactiveUser.props.style).toContainEqual(
      expect.objectContaining({ opacity: 0.6 })
    );
    
    // Verificar que el usuario activo no tiene opacidad reducida
    expect(activeUser.props.style).not.toContainEqual(
      expect.objectContaining({ opacity: 0.6 })
    );
  });
});
```

### 4. **Testing de Componentes con Navegación**

#### **Mock de Navegación**
```typescript
// src/components/__tests__/__mocks__/navigation.ts
export const mockNavigation = {
  navigate: jest.fn(),
  goBack: jest.fn(),
  push: jest.fn(),
  pop: jest.fn(),
  reset: jest.fn(),
  setOptions: jest.fn(),
  addListener: jest.fn(),
  removeListener: jest.fn(),
  isFocused: jest.fn(() => true),
  canGoBack: jest.fn(() => true),
  getParent: jest.fn(),
  getState: jest.fn(),
  dispatch: jest.fn(),
};

export const mockRoute = {
  key: 'test-route',
  name: 'TestScreen',
  params: {},
};
```

#### **Componente con Navegación**
```typescript
// src/components/UserCard.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';

interface Usuario {
  id: number;
  nombre: string;
  email: string;
  rol: string;
}

interface UserCardProps {
  usuario: Usuario;
  onEdit?: () => void;
}

export const UserCard: React.FC<UserCardProps> = ({ usuario, onEdit }) => {
  const navigation = useNavigation();
  
  const handleUserPress = () => {
    navigation.navigate('UserDetail', { userId: usuario.id });
  };
  
  const handleEditPress = () => {
    if (onEdit) {
      onEdit();
    } else {
      navigation.navigate('UserEdit', { userId: usuario.id });
    }
  };
  
  return (
    <View style={styles.container}>
      <TouchableOpacity 
        style={styles.userInfo}
        onPress={handleUserPress}
        testID={`user-card-${usuario.id}`}
      >
        <Text style={styles.userName}>{usuario.nombre}</Text>
        <Text style={styles.userEmail}>{usuario.email}</Text>
        <Text style={styles.userRole}>{usuario.rol}</Text>
      </TouchableOpacity>
      
      <TouchableOpacity
        style={styles.editButton}
        onPress={handleEditPress}
        testID={`edit-button-${usuario.id}`}
      >
        <Text style={styles.editButtonText}>Editar</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    backgroundColor: '#FFFFFF',
    padding: 16,
    marginBottom: 8,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#E5E5EA',
    alignItems: 'center',
  },
  userInfo: {
    flex: 1,
  },
  userName: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 4,
  },
  userEmail: {
    fontSize: 14,
    color: '#666666',
    marginBottom: 4,
  },
  userRole: {
    fontSize: 12,
    color: '#007AFF',
    fontWeight: '500',
  },
  editButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 6,
  },
  editButtonText: {
    color: '#FFFFFF',
    fontSize: 12,
    fontWeight: '600',
  },
});
```

#### **Tests para Componente con Navegación**
```typescript
// src/components/__tests__/UserCard.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { UserCard } from '../UserCard';

// Mock de react-navigation
jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({
    navigate: jest.fn(),
    goBack: jest.fn(),
    push: jest.fn(),
    pop: jest.fn(),
    reset: jest.fn(),
    setOptions: jest.fn(),
    addListener: jest.fn(),
    removeListener: jest.fn(),
    isFocused: jest.fn(() => true),
    canGoBack: jest.fn(() => true),
    getParent: jest.fn(),
    getState: jest.fn(),
    dispatch: jest.fn(),
  }),
}));

const mockUsuario = {
  id: 1,
  nombre: 'Ana García',
  email: 'ana@email.com',
  rol: 'Administrador'
};

describe('UserCard Component', () => {
  const mockOnEdit = jest.fn();
  
  beforeEach(() => {
    mockOnEdit.mockClear();
  });
  
  test('debe renderizar información del usuario', () => {
    const { getByText } = render(
      <UserCard usuario={mockUsuario} />
    );
    
    expect(getByText('Ana García')).toBeTruthy();
    expect(getByText('ana@email.com')).toBeTruthy();
    expect(getByText('Administrador')).toBeTruthy();
  });
  
  test('debe navegar a UserDetail al presionar información del usuario', () => {
    const mockNavigate = jest.fn();
    jest.spyOn(require('@react-navigation/native'), 'useNavigation')
      .mockReturnValue({ navigate: mockNavigate });
    
    const { getByTestId } = render(
      <UserCard usuario={mockUsuario} />
    );
    
    const userCard = getByTestId('user-card-1');
    fireEvent.press(userCard);
    
    expect(mockNavigate).toHaveBeenCalledWith('UserDetail', { userId: 1 });
  });
  
  test('debe llamar onEdit cuando se proporciona', () => {
    const { getByTestId } = render(
      <UserCard usuario={mockUsuario} onEdit={mockOnEdit} />
    );
    
    const editButton = getByTestId('edit-button-1');
    fireEvent.press(editButton);
    
    expect(mockOnEdit).toHaveBeenCalledTimes(1);
  });
  
  test('debe navegar a UserEdit cuando no se proporciona onEdit', () => {
    const mockNavigate = jest.fn();
    jest.spyOn(require('@react-navigation/native'), 'useNavigation')
      .mockReturnValue({ navigate: mockNavigate });
    
    const { getByTestId } = render(
      <UserCard usuario={mockUsuario} />
    );
    
    const editButton = getByTestId('edit-button-1');
    fireEvent.press(editButton);
    
    expect(mockNavigate).toHaveBeenCalledWith('UserEdit', { userId: 1 });
  });
});
```

## 🧪 **Ejercicios Prácticos**

### **Ejercicio 1: Testing de Formulario**
Crea tests para un componente de formulario de login:

```typescript
// Componente a testear
export const LoginForm: React.FC<{
  onSubmit: (email: string, password: string) => void;
  isLoading?: boolean;
}> = ({ onSubmit, isLoading = false }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = () => {
    if (email && password) {
      onSubmit(email, password);
    }
  };
  
  return (
    <View>
      <TextInput
        value={email}
        onChangeText={setEmail}
        placeholder="Email"
        testID="email-input"
      />
      <TextInput
        value={password}
        onChangeText={setPassword}
        placeholder="Contraseña"
        secureTextEntry
        testID="password-input"
      />
      <Button
        title={isLoading ? "Cargando..." : "Iniciar Sesión"}
        onPress={handleSubmit}
        disabled={isLoading || !email || !password}
        testID="submit-button"
      />
    </View>
  );
};

// Escribe tests que cubran:
// - Renderizado inicial
// - Cambio de valores en inputs
// - Validación de formulario
// - Estado de loading
// - Llamada a onSubmit
```

### **Ejercicio 2: Testing de Componente con Async Operations**
Crea tests para un componente que hace llamadas async:

```typescript
// Componente a testear
export const ProductList: React.FC = () => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchProducts();
  }, []);
  
  const fetchProducts = async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/products');
      const data = await response.json();
      setProducts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <Text>Cargando...</Text>;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <FlatList
      data={products}
      renderItem={({ item }) => <ProductItem product={item} />}
      keyExtractor={item => item.id}
    />
  );
};

// Escribe tests que cubran:
// - Estado de loading
// - Estado de error
// - Renderizado de productos
// - Mock de fetch
```

## 📝 **Resumen de la Clase**

### **Conceptos Clave Aprendidos**
1. **React Testing Library**: Herramienta para testing de componentes
2. **Testing de Props**: Verificación de propiedades y comportamiento
3. **Testing de Eventos**: Simulación de interacciones del usuario
4. **Testing de Estado**: Verificación de cambios de estado
5. **Mocks de Navegación**: Simulación de react-navigation
6. **Testing de Componentes Complejos**: Listas, formularios, etc.

### **Próximos Pasos**
- Testing de hooks personalizados
- Testing de lógica de negocio
- Testing de integración y APIs

## 🔗 **Navegación**
- [⬅️ Clase Anterior](clase_1_fundamentos_testing.md) - Fundamentos de Testing
- [🏠 Inicio](../../README.md)
- [📚 Índice Completo](../../INDICE_COMPLETO.md)
- [➡️ Siguiente Clase](clase_3_testing_hooks_logica.md) - Testing de Hooks y Lógica

---

**💡 Tip**: Usa `testID` para identificar elementos en tus tests. Esto hace que los tests sean más robustos y menos propensos a romperse por cambios en el UI.
