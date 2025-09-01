# 📚 Clase 3: Realm - Base de Datos NoSQL

## 🧭 Navegación del Módulo

- **⬅️ Anterior**: [Clase 2: SQLite - Base de Datos Relacional](clase_2_sqlite.md)
- **➡️ Siguiente**: [Clase 4: Migraciones y Versionado](clase_4_migraciones.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

- Comprender qué es Realm y cuándo usarlo
- Configurar Realm en React Native
- Definir esquemas y modelos de datos
- Implementar operaciones CRUD con objetos
- Manejar relaciones entre objetos
- Implementar migraciones y versionado

---

## 📖 Contenido Teórico

### 1. ¿Qué es Realm?

Realm es una base de datos NoSQL moderna y rápida diseñada específicamente para aplicaciones móviles. Es ideal para:

- **Datos complejos** (objetos anidados, arrays)
- **Relaciones entre objetos** (referencias, listas)
- **Consultas en tiempo real** (observadores reactivos)
- **Sincronización** (Realm Sync con backend)
- **Alto rendimiento** (optimizado para móviles)

**Características principales:**
- **NoSQL**: No requiere esquemas SQL tradicionales
- **Orientado a objetos**: Los datos se mapean directamente a objetos JavaScript
- **Reactivo**: Cambios automáticos en la UI cuando cambian los datos
- **Transaccional**: ACID compliance completo
- **Multiplataforma**: iOS, Android, React Native, Node.js

### 2. Instalación y Configuración

#### **Instalación del Paquete:**
```bash
npm install realm
# o
yarn add realm
```

#### **Configuración de iOS (Podfile):**
```ruby
# ios/Podfile
pod 'RealmJS', :path => '../node_modules/realm'
```

#### **Configuración de Android (android/app/build.gradle):**
```gradle
android {
    // ... otras configuraciones
    
    packagingOptions {
        pickFirst '**/libc++_shared.so'
        pickFirst '**/libjsc.so'
    }
}
```

---

## 🛠️ Implementación Práctica

### 1. Definición de Esquemas

Vamos a crear esquemas para una aplicación de e-commerce:

```javascript
// database/schemas.js
import Realm from 'realm';

// Esquema para categorías de productos
export const CategorySchema = {
  name: 'Category',                    // Nombre del esquema (debe ser único)
  properties: {                        // Propiedades del objeto
    id: 'int',                         // ID único (entero)
    name: 'string',                    // Nombre de la categoría (texto)
    description: 'string?',            // Descripción opcional (texto, puede ser null)
    imageUrl: 'string?',               // URL de imagen opcional
    isActive: 'bool',                  // Estado activo/inactivo (booleano)
    createdAt: 'date',                 // Fecha de creación (fecha)
    updatedAt: 'date'                  // Fecha de última actualización
  },
  primaryKey: 'id'                     // Campo que actúa como clave primaria
};

// Esquema para productos
export const ProductSchema = {
  name: 'Product',                     // Nombre del esquema
  properties: {
    id: 'int',                         // ID único
    name: 'string',                    // Nombre del producto
    description: 'string?',            // Descripción opcional
    price: 'double',                   // Precio (número decimal)
    originalPrice: 'double?',          // Precio original (para descuentos)
    categoryId: 'int?',                // ID de la categoría (referencia)
    images: 'string[]',                // Array de URLs de imágenes
    tags: 'string[]',                  // Array de etiquetas
    specifications: 'string{}',        // Objeto de especificaciones (clave-valor)
    isAvailable: 'bool',               // Disponibilidad del producto
    stockQuantity: 'int',              // Cantidad en stock
    rating: 'double?',                 // Calificación promedio
    reviewCount: 'int',                // Número de reseñas
    createdAt: 'date',                 // Fecha de creación
    updatedAt: 'date'                  // Fecha de actualización
  },
  primaryKey: 'id'                     // Clave primaria
};

// Esquema para usuarios
export const UserSchema = {
  name: 'User',                        // Nombre del esquema
  properties: {
    id: 'int',                         // ID único
    email: 'string',                   // Email del usuario
    username: 'string',                // Nombre de usuario
    firstName: 'string',               // Nombre
    lastName: 'string',                // Apellido
    phone: 'string?',                  // Teléfono opcional
    avatar: 'string?',                 // URL del avatar
    isVerified: 'bool',                // Usuario verificado
    preferences: 'string{}',           // Preferencias del usuario
    createdAt: 'date',                 // Fecha de creación
    updatedAt: 'date'                  // Fecha de actualización
  },
  primaryKey: 'id'                     // Clave primaria
};

// Esquema para pedidos
export const OrderSchema = {
  name: 'Order',                       // Nombre del esquema
  properties: {
    id: 'int',                         // ID único
    userId: 'int',                     // ID del usuario que hizo el pedido
    items: 'OrderItem[]',              // Lista de items del pedido (relación)
    status: 'string',                  // Estado del pedido
    subtotal: 'double',                // Subtotal antes de impuestos
    tax: 'double',                     // Impuestos
    shipping: 'double',                // Costo de envío
    total: 'double',                   // Total del pedido
    shippingAddress: 'string{}',       // Dirección de envío
    billingAddress: 'string{}',        // Dirección de facturación
    paymentMethod: 'string',           // Método de pago
    notes: 'string?',                  // Notas del pedido
    createdAt: 'date',                 // Fecha de creación
    updatedAt: 'date'                  // Fecha de actualización
  },
  primaryKey: 'id'                     // Clave primaria
};

// Esquema para items de pedido
export const OrderItemSchema = {
  name: 'OrderItem',                   // Nombre del esquema
  properties: {
    id: 'int',                         // ID único
    orderId: 'int',                    // ID del pedido padre
    productId: 'int',                  // ID del producto
    productName: 'string',             // Nombre del producto (copia para historial)
    quantity: 'int',                   // Cantidad ordenada
    unitPrice: 'double',               // Precio unitario
    totalPrice: 'double',              // Precio total del item
    createdAt: 'date'                  // Fecha de creación
  },
  primaryKey: 'id'                     // Clave primaria
};

// Esquema para reseñas de productos
export const ReviewSchema = {
  name: 'Review',                      // Nombre del esquema
  properties: {
    id: 'int',                         // ID único
    productId: 'int',                  // ID del producto
    userId: 'int',                     // ID del usuario
    rating: 'int',                     // Calificación (1-5)
    title: 'string?',                  // Título de la reseña
    comment: 'string?',                // Comentario de la reseña
    images: 'string[]',                // Imágenes de la reseña
    isVerified: 'bool',                // Reseña verificada
    helpfulCount: 'int',               // Número de "me gusta"
    createdAt: 'date',                 // Fecha de creación
    updatedAt: 'date'                  // Fecha de actualización
  },
  primaryKey: 'id'                     // Clave primaria
};
```

### 2. Configuración de la Base de Datos

```javascript
// database/realm.js
import Realm from 'realm';
import {
  CategorySchema,
  ProductSchema,
  UserSchema,
  OrderSchema,
  OrderItemSchema,
  ReviewSchema
} from './schemas';

// Configuración de la base de datos Realm
const realmConfig = {
  schema: [
    CategorySchema,        // Incluimos todos los esquemas
    ProductSchema,
    UserSchema,
    OrderSchema,
    OrderItemSchema,
    ReviewSchema
  ],
  schemaVersion: 1,        // Versión actual del esquema
  migration: (oldRealm, newRealm) => {
    // Función de migración (se ejecuta cuando cambia la versión)
    if (oldRealm.schemaVersion < 1) {
      // Lógica de migración para la versión 1
      console.log('Migrating to schema version 1');
    }
  }
};

// Función para abrir la base de datos
export const openRealm = async () => {
  try {
    // Abrimos la base de datos con la configuración especificada
    const realm = await Realm.open(realmConfig);
    console.log('Realm database opened successfully');
    return realm;
  } catch (error) {
    console.error('Error opening Realm database:', error);
    throw error;
  }
};

// Función para cerrar la base de datos
export const closeRealm = (realm) => {
  if (realm && !realm.isClosed) {
    realm.close();
    console.log('Realm database closed successfully');
  }
};

// Función para eliminar la base de datos (útil para testing)
export const deleteRealm = async () => {
  try {
    const realmPath = Realm.defaultPath;
    await Realm.deleteFile({ path: realmPath });
    console.log('Realm database deleted successfully');
  } catch (error) {
    console.error('Error deleting Realm database:', error);
    throw error;
  }
};
```

### 3. Servicios de Base de Datos

#### **Servicio de Categorías:**

```javascript
// services/categoryService.js
import Realm from 'realm';
import { openRealm } from '../database/realm';

export class CategoryService {
  // Propiedad para almacenar la instancia de Realm
  realm = null;

  // Método para inicializar el servicio
  async initialize() {
    try {
      this.realm = await openRealm();
    } catch (error) {
      console.error('Failed to initialize CategoryService:', error);
      throw error;
    }
  }

  // Método para crear una nueva categoría
  async createCategory(categoryData) {
    try {
      // Verificamos que la base de datos esté abierta
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      let category;
      
      // Iniciamos una transacción de escritura
      this.realm.write(() => {
        // Creamos el objeto de categoría dentro de la transacción
        category = this.realm.create('Category', {
          id: Date.now(),                    // Generamos un ID único usando timestamp
          name: categoryData.name,           // Nombre de la categoría
          description: categoryData.description || null, // Descripción opcional
          imageUrl: categoryData.imageUrl || null,       // URL de imagen opcional
          isActive: categoryData.isActive !== false,     // Por defecto activa
          createdAt: new Date(),             // Fecha de creación actual
          updatedAt: new Date()              // Fecha de actualización actual
        });
      });

      console.log('Category created successfully:', category.name);
      return category;
    } catch (error) {
      console.error('Error creating category:', error);
      throw error;
    }
  }

  // Método para obtener todas las categorías
  async getAllCategories() {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      // Obtenemos todos los objetos de categoría y los convertimos a array
      const categories = this.realm.objects('Category');
      
      // Filtramos solo las categorías activas y las ordenamos por nombre
      const activeCategories = categories
        .filtered('isActive == true')        // Filtro por estado activo
        .sorted('name');                     // Ordenamiento por nombre

      // Convertimos el resultado a un array normal de JavaScript
      return Array.from(activeCategories);
    } catch (error) {
      console.error('Error getting categories:', error);
      throw error;
    }
  }

  // Método para obtener una categoría por ID
  async getCategoryById(id) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      // Buscamos la categoría por su ID primario
      const category = this.realm.objectForPrimaryKey('Category', id);
      
      if (!category) {
        return null; // Retornamos null si no se encuentra
      }

      return category;
    } catch (error) {
      console.error('Error getting category by ID:', error);
      throw error;
    }
  }

  // Método para actualizar una categoría
  async updateCategory(id, updates) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      // Buscamos la categoría existente
      const category = this.realm.objectForPrimaryKey('Category', id);
      
      if (!category) {
        throw new Error(`Category with ID ${id} not found`);
      }

      let updatedCategory;
      
      // Iniciamos una transacción de escritura
      this.realm.write(() => {
        // Actualizamos solo los campos permitidos
        if (updates.name !== undefined) {
          category.name = updates.name;
        }
        if (updates.description !== undefined) {
          category.description = updates.description;
        }
        if (updates.imageUrl !== undefined) {
          category.imageUrl = updates.imageUrl;
        }
        if (updates.isActive !== undefined) {
          category.isActive = updates.isActive;
        }
        
        // Actualizamos la fecha de modificación
        category.updatedAt = new Date();
        
        updatedCategory = category;
      });

      console.log('Category updated successfully:', updatedCategory.name);
      return updatedCategory;
    } catch (error) {
      console.error('Error updating category:', error);
      throw error;
    }
  }

  // Método para eliminar una categoría
  async deleteCategory(id) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      // Buscamos la categoría existente
      const category = this.realm.objectForPrimaryKey('Category', id);
      
      if (!category) {
        throw new Error(`Category with ID ${id} not found`);
      }

      // Iniciamos una transacción de escritura
      this.realm.write(() => {
        // Eliminamos la categoría de la base de datos
        this.realm.delete(category);
      });

      console.log('Category deleted successfully');
      return true;
    } catch (error) {
      console.error('Error deleting category:', error);
      throw error;
    }
  }

  // Método para buscar categorías por nombre
  async searchCategories(searchTerm) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      // Obtenemos todas las categorías
      const categories = this.realm.objects('Category');
      
      // Filtramos por término de búsqueda (búsqueda insensible a mayúsculas)
      const searchResults = categories
        .filtered('name CONTAINS[c] $0', searchTerm) // [c] = case insensitive
        .sorted('name');

      // Convertimos el resultado a un array normal
      return Array.from(searchResults);
    } catch (error) {
      console.error('Error searching categories:', error);
      throw error;
    }
  }

  // Método para obtener categorías con paginación
  async getCategoriesPaginated(page = 1, limit = 10) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      // Obtenemos todas las categorías activas
      const allCategories = this.realm.objects('Category')
        .filtered('isActive == true')
        .sorted('name');

      // Calculamos el total y la paginación
      const total = allCategories.length;
      const startIndex = (page - 1) * limit;
      const endIndex = startIndex + limit;
      
      // Obtenemos solo las categorías de la página actual
      const paginatedCategories = allCategories.slice(startIndex, endIndex);

      return {
        categories: Array.from(paginatedCategories),
        pagination: {
          page,
          limit,
          total,
          totalPages: Math.ceil(total / limit),
          hasNext: page * limit < total,
          hasPrev: page > 1
        }
      };
    } catch (error) {
      console.error('Error getting paginated categories:', error);
      throw error;
    }
  }

  // Método para cerrar el servicio
  close() {
    if (this.realm && !this.realm.isClosed) {
      this.realm.close();
      this.realm = null;
    }
  }
}

// Exportamos una instancia singleton del servicio
export default new CategoryService();
```

#### **Servicio de Productos:**

```javascript
// services/productService.js
import Realm from 'realm';
import { openRealm } from '../database/realm';

export class ProductService {
  realm = null;

  async initialize() {
    try {
      this.realm = await openRealm();
    } catch (error) {
      console.error('Failed to initialize ProductService:', error);
      throw error;
    }
  }

  // Método para crear un nuevo producto
  async createProduct(productData) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      let product;
      
      this.realm.write(() => {
        product = this.realm.create('Product', {
          id: Date.now(),
          name: productData.name,
          description: productData.description || null,
          price: productData.price,
          originalPrice: productData.originalPrice || productData.price,
          categoryId: productData.categoryId || null,
          images: productData.images || [],
          tags: productData.tags || [],
          specifications: productData.specifications || {},
          isAvailable: productData.isAvailable !== false,
          stockQuantity: productData.stockQuantity || 0,
          rating: productData.rating || null,
          reviewCount: productData.reviewCount || 0,
          createdAt: new Date(),
          updatedAt: new Date()
        });
      });

      console.log('Product created successfully:', product.name);
      return product;
    } catch (error) {
      console.error('Error creating product:', error);
      throw error;
    }
  }

  // Método para obtener productos con filtros avanzados
  async getProducts(filters = {}) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      let products = this.realm.objects('Product');

      // Aplicamos filtros si están especificados
      if (filters.categoryId) {
        products = products.filtered('categoryId == $0', filters.categoryId);
      }

      if (filters.isAvailable !== undefined) {
        products = products.filtered('isAvailable == $0', filters.isAvailable);
      }

      if (filters.minPrice !== undefined) {
        products = products.filtered('price >= $0', filters.minPrice);
      }

      if (filters.maxPrice !== undefined) {
        products = products.filtered('price <= $0', filters.maxPrice);
      }

      if (filters.searchTerm) {
        products = products.filtered('name CONTAINS[c] $0 OR description CONTAINS[c] $0', 
          filters.searchTerm, filters.searchTerm);
      }

      // Aplicamos ordenamiento
      if (filters.sortBy) {
        const sortOrder = filters.sortOrder === 'desc' ? true : false;
        products = products.sorted(filters.sortBy, sortOrder);
      } else {
        // Ordenamiento por defecto
        products = products.sorted('createdAt', true);
      }

      return Array.from(products);
    } catch (error) {
      console.error('Error getting products:', error);
      throw error;
    }
  }

  // Método para obtener un producto por ID
  async getProductById(id) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      const product = this.realm.objectForPrimaryKey('Product', id);
      return product || null;
    } catch (error) {
      console.error('Error getting product by ID:', error);
      throw error;
    }
  }

  // Método para actualizar un producto
  async updateProduct(id, updates) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      const product = this.realm.objectForPrimaryKey('Product', id);
      
      if (!product) {
        throw new Error(`Product with ID ${id} not found`);
      }

      let updatedProduct;
      
      this.realm.write(() => {
        // Actualizamos solo los campos permitidos
        const allowedFields = [
          'name', 'description', 'price', 'originalPrice', 'categoryId',
          'images', 'tags', 'specifications', 'isAvailable', 'stockQuantity'
        ];

        allowedFields.forEach(field => {
          if (updates[field] !== undefined) {
            product[field] = updates[field];
          }
        });
        
        product.updatedAt = new Date();
        updatedProduct = product;
      });

      console.log('Product updated successfully:', updatedProduct.name);
      return updatedProduct;
    } catch (error) {
      console.error('Error updating product:', error);
      throw error;
    }
  }

  // Método para eliminar un producto
  async deleteProduct(id) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      const product = this.realm.objectForPrimaryKey('Product', id);
      
      if (!product) {
        throw new Error(`Product with ID ${id} not found`);
      }

      this.realm.write(() => {
        this.realm.delete(product);
      });

      console.log('Product deleted successfully');
      return true;
    } catch (error) {
      console.error('Error deleting product:', error);
      throw error;
    }
  }

  // Método para obtener productos por categoría
  async getProductsByCategory(categoryId) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      const products = this.realm.objects('Product')
        .filtered('categoryId == $0 AND isAvailable == true', categoryId)
        .sorted('name');

      return Array.from(products);
    } catch (error) {
      console.error('Error getting products by category:', error);
      throw error;
    }
  }

  // Método para buscar productos por texto
  async searchProducts(searchTerm) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      const products = this.realm.objects('Product')
        .filtered('name CONTAINS[c] $0 OR description CONTAINS[c] $0 OR tags CONTAINS $0', 
          searchTerm, searchTerm, searchTerm)
        .filtered('isAvailable == true')
        .sorted('name');

      return Array.from(products);
    } catch (error) {
      console.error('Error searching products:', error);
      throw error;
    }
  }

  // Método para actualizar stock de un producto
  async updateStock(productId, quantity) {
    try {
      if (!this.realm || this.realm.isClosed) {
        throw new Error('Realm database is not open');
      }

      const product = this.realm.objectForPrimaryKey('Product', productId);
      
      if (!product) {
        throw new Error(`Product with ID ${productId} not found`);
      }

      let updatedProduct;
      
      this.realm.write(() => {
        product.stockQuantity = Math.max(0, product.stockQuantity + quantity);
        product.isAvailable = product.stockQuantity > 0;
        product.updatedAt = new Date();
        updatedProduct = product;
      });

      console.log(`Stock updated for product ${updatedProduct.name}: ${updatedProduct.stockQuantity}`);
      return updatedProduct;
    } catch (error) {
      console.error('Error updating stock:', error);
      throw error;
    }
  }

  close() {
    if (this.realm && !this.realm.isClosed) {
      this.realm.close();
      this.realm = null;
    }
  }
}

export default new ProductService();
```

---

## 💡 Casos de Uso Prácticos

### 1. Hook para Usar Realm

```javascript
// hooks/useRealm.js
import { useState, useEffect, useCallback } from 'react';
import { openRealm, closeRealm } from '../database/realm';

export const useRealm = () => {
  const [realm, setRealm] = useState(null);
  const [isInitialized, setIsInitialized] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const initRealm = async () => {
      try {
        const realmInstance = await openRealm();
        setRealm(realmInstance);
        setIsInitialized(true);
      } catch (err) {
        setError(err);
        console.error('Failed to initialize Realm:', err);
      }
    };

    initRealm();

    // Cleanup al desmontar el componente
    return () => {
      if (realm && !realm.isClosed) {
        closeRealm(realm);
      }
    };
  }, []);

  const executeWrite = useCallback(async (writeFunction) => {
    if (!realm || realm.isClosed) {
      throw new Error('Realm is not initialized');
    }

    try {
      let result;
      realm.write(() => {
        result = writeFunction(realm);
      });
      return result;
    } catch (err) {
      console.error('Write operation failed:', err);
      throw err;
    }
  }, [realm]);

  const executeQuery = useCallback(async (queryFunction) => {
    if (!realm || realm.isClosed) {
      throw new Error('Realm is not initialized');
    }

    try {
      return queryFunction(realm);
    } catch (err) {
      console.error('Query operation failed:', err);
      throw err;
    }
  }, [realm]);

  return {
    realm,
    isInitialized,
    error,
    executeWrite,
    executeQuery
  };
};
```

### 2. Componente de Lista de Productos

```javascript
// components/ProductList.js
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { useRealm } from '../hooks/useRealm';
import ProductService from '../services/productService';

const ProductList = ({ categoryId, onProductPress }) => {
  const { isInitialized, error } = useRealm();
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (isInitialized) {
      loadProducts();
    }
  }, [isInitialized, categoryId]);

  const loadProducts = async () => {
    try {
      setLoading(true);
      const filters = categoryId ? { categoryId } : {};
      const productList = await ProductService.getProducts(filters);
      setProducts(productList);
    } catch (err) {
      console.error('Failed to load products:', err);
    } finally {
      setLoading(false);
    }
  };

  const renderProduct = ({ item }) => (
    <TouchableOpacity
      style={styles.productItem}
      onPress={() => onProductPress(item)}
    >
      <Text style={styles.productName}>{item.name}</Text>
      <Text style={styles.productPrice}>${item.price}</Text>
      <Text style={styles.productStock}>
        Stock: {item.stockQuantity}
      </Text>
    </TouchableOpacity>
  );

  if (error) {
    return (
      <View style={styles.errorContainer}>
        <Text style={styles.errorText}>
          Error loading products: {error.message}
        </Text>
      </View>
    );
  }

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <Text>Loading products...</Text>
      </View>
    );
  }

  return (
    <FlatList
      data={products}
      renderItem={renderProduct}
      keyExtractor={(item) => item.id.toString()}
      style={styles.container}
      showsVerticalScrollIndicator={false}
    />
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  productItem: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
    backgroundColor: 'white',
  },
  productName: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 4,
  },
  productPrice: {
    fontSize: 14,
    color: '#007AFF',
    marginBottom: 2,
  },
  productStock: {
    fontSize: 12,
    color: '#666',
  },
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  errorText: {
    color: 'red',
    textAlign: 'center',
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default ProductList;
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Configuración Básica**
1. Instala Realm en tu proyecto React Native
2. Configura iOS y Android según la documentación
3. Crea los esquemas básicos para usuarios y productos
4. Prueba la inicialización de la base de datos

### **Ejercicio 2: Operaciones CRUD Básicas**
1. Implementa el servicio de categorías completo
2. Crea métodos para crear, leer, actualizar y eliminar
3. Prueba todas las operaciones con datos de ejemplo
4. Implementa manejo de errores robusto

### **Ejercicio 3: Consultas Avanzadas**
1. Implementa filtros múltiples en productos
2. Crea búsqueda por texto con múltiples campos
3. Implementa ordenamiento por diferentes criterios
4. Agrega paginación a las consultas

### **Ejercicio 4: Relaciones entre Objetos**
1. Crea esquemas para pedidos y items
2. Implementa relaciones uno a muchos
3. Crea consultas que incluyan objetos relacionados
4. Prueba la integridad referencial

### **Ejercicio 5: Observadores Reactivos**
1. Implementa listeners para cambios en tiempo real
2. Crea un componente que se actualice automáticamente
3. Maneja la suscripción y desuscripción correctamente
4. Optimiza el rendimiento con useMemo y useCallback

---

## 🎯 Proyecto de la Clase

### **E-commerce App con Realm**

Crea una aplicación que permita:
1. **Gestionar categorías** de productos
2. **Administrar productos** con imágenes y especificaciones
3. **Sistema de pedidos** con items relacionados
4. **Búsqueda avanzada** con filtros múltiples
5. **Actualizaciones en tiempo real** usando observadores

**Estructura del Proyecto:**
```
src/
├── database/
│   ├── schemas.js         # Definición de esquemas
│   └── realm.js           # Configuración de Realm
├── services/
│   ├── categoryService.js # Servicio de categorías
│   └── productService.js  # Servicio de productos
├── hooks/
│   └── useRealm.js        # Hook para usar Realm
├── components/
│   ├── CategoryList.js    # Lista de categorías
│   ├── ProductList.js     # Lista de productos
│   └── ProductForm.js     # Formulario de productos
└── screens/
    ├── CategoriesScreen.js # Pantalla de categorías
    ├── ProductsScreen.js   # Pantalla de productos
    └── ProductDetailScreen.js # Detalle de producto
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [Realm React Native](https://docs.mongodb.com/realm/sdk/react-native/)
- [Realm JavaScript](https://docs.mongodb.com/realm/sdk/javascript/)

### **Mejores Prácticas:**
- **Transacciones**: Siempre usa write() para modificaciones
- **Observadores**: Maneja correctamente las suscripciones
- **Esquemas**: Planifica bien la estructura de datos
- **Migraciones**: Implementa versionado de esquemas

### **Alternativas:**
- **SQLite**: Para datos relacionales complejos
- **WatermelonDB**: Para aplicaciones con sincronización
- **PouchDB**: Para sincronización con CouchDB

---

## 🔍 Resumen de la Clase

### **Conceptos Clave:**
- ✅ Realm es ideal para datos complejos y relaciones
- ✅ Los esquemas definen la estructura de los datos
- ✅ Las transacciones garantizan consistencia
- ✅ Los observadores permiten actualizaciones en tiempo real
- ✅ Realm es muy eficiente para aplicaciones móviles

### **Próximos Pasos:**
- 🔄 Completar todos los ejercicios de esta clase
- 🔄 Implementar la app de e-commerce
- 🔄 Practicar con relaciones complejas
- 🔄 Prepararse para la siguiente clase sobre migraciones

---

## 🚀 Siguiente Clase

En la **Clase 4: Migraciones y Versionado**, aprenderás:
- Cómo manejar cambios en esquemas de base de datos
- Implementación de migraciones automáticas
- Versionado de esquemas y datos
- Estrategias de migración para diferentes escenarios
- Testing de migraciones

---

**¡Excelente trabajo con Realm! 🎉**

**Recuerda**: Realm es poderoso para datos complejos. Domina los esquemas y relaciones antes de avanzar a migraciones.
