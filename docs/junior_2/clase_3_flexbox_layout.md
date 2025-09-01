# 📚 Clase 3: Flexbox y Layout

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Clase 2: JSX y Estructura](clase_2_jsx_estructura.md)
- **➡️ Siguiente**: [Clase 4: Componentes Personalizados](clase_4_componentes_personalizados.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase
- Comprender el sistema de layout Flexbox en React Native
- Aprender a crear layouts responsivos y adaptables
- Dominar las propiedades principales de Flexbox
- Crear interfaces complejas con posicionamiento preciso
- Entender la diferencia entre Flexbox y otros sistemas de layout

---

## 📚 Contenido Teórico

### **¿Qué es Flexbox?**

Flexbox (Flexible Box Layout) es un modelo de layout unidimensional que permite distribuir elementos de manera flexible y responsiva. En React Native, Flexbox es el sistema de layout principal y funciona de manera similar al CSS web, pero con algunas diferencias importantes.

#### **Características principales:**
- **Unidimensional**: Organiza elementos en una sola dirección (fila o columna)
- **Flexible**: Los elementos pueden crecer, encogerse y reorganizarse
- **Responsivo**: Se adapta automáticamente a diferentes tamaños de pantalla
- **Intuitivo**: Sintaxis clara y fácil de entender

### **Propiedades principales de Flexbox:**

#### **1. Propiedades del contenedor (flexDirection, justifyContent, alignItems):**
- **flexDirection**: Define la dirección principal de los elementos
- **justifyContent**: Alinea elementos en la dirección principal
- **alignItems**: Alinea elementos en la dirección transversal
- **flexWrap**: Permite que los elementos se envuelvan a la siguiente línea

#### **2. Propiedades de los elementos hijos (flex, alignSelf):**
- **flex**: Define cómo crece, encoge y se comporta el elemento
- **alignSelf**: Alinea un elemento específico de manera diferente a sus hermanos

### **Diferencias con CSS Web:**

1. **flexDirection por defecto**: En React Native es `column`, en CSS es `row`
2. **flex por defecto**: En React Native es `0`, en CSS es `1`
3. **Unidades**: React Native no usa unidades como `px`, `em`, etc.
4. **Valores**: Algunos valores como `flex-start` se escriben como `flex-start`

---

## 💻 Implementación Práctica

### **1. Layout Básico con Flexbox**

```javascript:src/components/BasicFlexboxLayout.js
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const BasicFlexboxLayout = () => {
  return (
    // Contenedor principal con flex: 1 para ocupar toda la pantalla
    <View style={styles.container}>
      {/* Header fijo en la parte superior */}
      <View style={styles.header}>
        <Text style={styles.headerText}>Header Fijo</Text>
      </View>
      
      {/* Contenido principal que se expande */}
      <View style={styles.content}>
        {/* Primera fila de elementos */}
        <View style={styles.row}>
          <View style={styles.box}>
            <Text style={styles.boxText}>Caja 1</Text>
          </View>
          <View style={styles.box}>
            <Text style={styles.boxText}>Caja 2</Text>
          </View>
        </View>
        
        {/* Segunda fila de elementos */}
        <View style={styles.row}>
          <View style={styles.box}>
            <Text style={styles.boxText}>Caja 3</Text>
          </View>
          <View style={styles.box}>
            <Text style={styles.boxText}>Caja 4</Text>
          </View>
        </View>
        
        {/* Elemento centrado */}
        <View style={styles.centeredBox}>
          <Text style={styles.centeredText}>Centrado</Text>
        </View>
      </View>
      
      {/* Footer fijo en la parte inferior */}
      <View style={styles.footer}>
        <Text style={styles.footerText}>Footer Fijo</Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  // Contenedor principal: ocupa toda la pantalla
  container: {
    flex: 1, // flex: 1 hace que ocupe todo el espacio disponible
    backgroundColor: '#f5f5f5',
  },
  
  // Header: altura fija, centrado horizontalmente
  header: {
    height: 60, // Altura fija de 60 unidades
    backgroundColor: '#007bff',
    justifyContent: 'center', // Centra verticalmente el contenido
    alignItems: 'center', // Centra horizontalmente el contenido
    elevation: 4, // Sombra en Android
    shadowColor: '#000', // Color de sombra en iOS
    shadowOffset: { width: 0, height: 2 }, // Offset de la sombra
    shadowOpacity: 0.25, // Opacidad de la sombra
    shadowRadius: 4, // Radio de la sombra
  },
  
  headerText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  
  // Contenido principal: se expande para llenar el espacio disponible
  content: {
    flex: 1, // flex: 1 hace que ocupe todo el espacio restante
    padding: 20, // Padding interno de 20 unidades
    justifyContent: 'space-between', // Distribuye el espacio entre elementos
  },
  
  // Fila de elementos: organiza elementos horizontalmente
  row: {
    flexDirection: 'row', // flexDirection: 'row' organiza elementos horizontalmente
    justifyContent: 'space-between', // Distribuye el espacio entre elementos
    marginBottom: 20, // Margen inferior de 20 unidades
  },
  
  // Caja individual: tamaño fijo, centrada
  box: {
    width: 150, // Ancho fijo de 150 unidades
    height: 100, // Alto fijo de 100 unidades
    backgroundColor: '#28a745',
    justifyContent: 'center', // Centra verticalmente el texto
    alignItems: 'center', // Centra horizontalmente el texto
    borderRadius: 10, // Bordes redondeados
  },
  
  boxText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  
  // Caja centrada: ocupa todo el ancho disponible
  centeredBox: {
    backgroundColor: '#ffc107',
    padding: 20, // Padding interno de 20 unidades
    borderRadius: 10, // Bordes redondeados
    alignItems: 'center', // Centra horizontalmente el contenido
  },
  
  centeredText: {
    color: '#333',
    fontSize: 18,
    fontWeight: 'bold',
  },
  
  // Footer: altura fija, centrado horizontalmente
  footer: {
    height: 50, // Altura fija de 50 unidades
    backgroundColor: '#6c757d',
    justifyContent: 'center', // Centra verticalmente el contenido
    alignItems: 'center', // Centra horizontalmente el contenido
  },
  
  footerText: {
    color: 'white',
    fontSize: 14,
  },
});

export default BasicFlexboxLayout;
```

### **2. Layout Avanzado con Flexbox**

```javascript:src/components/AdvancedFlexboxLayout.js
import React, { useState } from 'react';
import { 
  View, 
  Text, 
  TouchableOpacity, 
  ScrollView, 
  StyleSheet 
} from 'react-native';

const AdvancedFlexboxLayout = () => {
  // Estado para controlar la vista actual
  const [currentView, setCurrentView] = useState('grid');
  
  // Datos de ejemplo para las tarjetas
  const cards = [
    { id: 1, title: 'Tarjeta 1', color: '#e74c3c' },
    { id: 2, title: 'Tarjeta 2', color: '#3498db' },
    { id: 3, title: 'Tarjeta 3', color: '#2ecc71' },
    { id: 4, title: 'Tarjeta 4', color: '#f39c12' },
    { id: 5, title: 'Tarjeta 5', color: '#9b59b6' },
    { id: 6, title: 'Tarjeta 6', color: '#1abc9c' },
  ];
  
  // Función para renderizar vista de cuadrícula
  const renderGridView = () => (
    <ScrollView style={styles.scrollContainer}>
      {/* Primera fila de tarjetas */}
      <View style={styles.gridRow}>
        {cards.slice(0, 2).map(card => (
          <TouchableOpacity
            key={card.id}
            style={[styles.gridCard, { backgroundColor: card.color }]}
          >
            <Text style={styles.cardTitle}>{card.title}</Text>
          </TouchableOpacity>
        ))}
      </View>
      
      {/* Segunda fila de tarjetas */}
      <View style={styles.gridRow}>
        {cards.slice(2, 4).map(card => (
          <TouchableOpacity
            key={card.id}
            style={[styles.gridCard, { backgroundColor: card.color }]}
          >
            <Text style={styles.cardTitle}>{card.title}</Text>
          </TouchableOpacity>
        ))}
      </View>
      
      {/* Tercera fila de tarjetas */}
      <View style={styles.gridRow}>
        {cards.slice(4, 6).map(card => (
          <TouchableOpacity
            key={card.id}
            style={[styles.gridCard, { backgroundColor: card.color }]}
          >
            <Text style={styles.cardTitle}>{card.title}</Text>
          </TouchableOpacity>
        ))}
      </View>
    </ScrollView>
  );
  
  // Función para renderizar vista de lista
  const renderListView = () => (
    <ScrollView style={styles.scrollContainer}>
      {cards.map(card => (
        <View key={card.id} style={styles.listCard}>
          <View style={[styles.listCardColor, { backgroundColor: card.color }]} />
          <View style={styles.listCardContent}>
            <Text style={styles.listCardTitle}>{card.title}</Text>
            <Text style={styles.listCardSubtitle}>
              Descripción de la tarjeta {card.id}
            </Text>
          </View>
          <TouchableOpacity style={styles.listCardButton}>
            <Text style={styles.listCardButtonText}>→</Text>
          </TouchableOpacity>
        </View>
      ))}
    </ScrollView>
  );
  
  // Función para renderizar vista de mosaico
  const renderMosaicView = () => (
    <ScrollView style={styles.scrollContainer}>
      {/* Primera fila: tarjeta grande */}
      <View style={styles.mosaicRow}>
        <TouchableOpacity
          style={[styles.mosaicCardLarge, { backgroundColor: cards[0].color }]}
        >
          <Text style={styles.mosaicCardTitle}>{cards[0].title}</Text>
        </TouchableOpacity>
      </View>
      
      {/* Segunda fila: dos tarjetas medianas */}
      <View style={styles.mosaicRow}>
        <TouchableOpacity
          style={[styles.mosaicCardMedium, { backgroundColor: cards[1].color }]}
        >
          <Text style={styles.mosaicCardTitle}>{cards[1].title}</Text>
        </TouchableOpacity>
        <TouchableOpacity
          style={[styles.mosaicCardMedium, { backgroundColor: cards[2].color }]}
        >
          <Text style={styles.mosaicCardTitle}>{cards[2].title}</Text>
        </TouchableOpacity>
      </View>
      
      {/* Tercera fila: tres tarjetas pequeñas */}
      <View style={styles.mosaicRow}>
        {cards.slice(3, 6).map(card => (
          <TouchableOpacity
            key={card.id}
            style={[styles.mosaicCardSmall, { backgroundColor: card.color }]}
          >
            <Text style={styles.mosaicCardTitle}>{card.title}</Text>
          </TouchableOpacity>
        ))}
      </View>
    </ScrollView>
  );
  
  return (
    <View style={styles.container}>
      {/* Header con navegación */}
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Layout Avanzado</Text>
        <View style={styles.navigation}>
          {['grid', 'list', 'mosaic'].map(view => (
            <TouchableOpacity
              key={view}
              style={[
                styles.navButton,
                currentView === view && styles.navButtonActive
              ]}
              onPress={() => setCurrentView(view)}
            >
              <Text style={[
                styles.navButtonText,
                currentView === view && styles.navButtonTextActive
              ]}>
                {view === 'grid' ? 'Cuadrícula' : 
                 view === 'list' ? 'Lista' : 'Mosaico'}
              </Text>
            </TouchableOpacity>
          ))}
        </View>
      </View>
      
      {/* Contenido dinámico */}
      <View style={styles.content}>
        {currentView === 'grid' && renderGridView()}
        {currentView === 'list' && renderListView()}
        {currentView === 'mosaic' && renderMosaicView()}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  // Contenedor principal
  container: {
    flex: 1,
    backgroundColor: '#f8f9fa',
  },
  
  // Header con navegación
  header: {
    backgroundColor: '#343a40',
    paddingTop: 40, // Para evitar el notch en iOS
    paddingBottom: 20,
    paddingHorizontal: 20,
  },
  
  headerTitle: {
    color: 'white',
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  
  // Navegación entre vistas
  navigation: {
    flexDirection: 'row', // Organiza botones horizontalmente
    justifyContent: 'space-between', // Distribuye el espacio entre botones
  },
  
  navButton: {
    flex: 1, // Cada botón ocupa el mismo espacio
    paddingVertical: 12,
    paddingHorizontal: 16,
    marginHorizontal: 5,
    borderRadius: 8,
    backgroundColor: '#495057',
    alignItems: 'center', // Centra el texto horizontalmente
  },
  
  navButtonActive: {
    backgroundColor: '#007bff',
  },
  
  navButtonText: {
    color: '#adb5bd',
    fontWeight: '500',
  },
  
  navButtonTextActive: {
    color: 'white',
  },
  
  // Contenido principal
  content: {
    flex: 1, // Ocupa todo el espacio restante
  },
  
  // Contenedor de scroll
  scrollContainer: {
    flex: 1,
    padding: 20,
  },
  
  // Estilos para vista de cuadrícula
  gridRow: {
    flexDirection: 'row', // Organiza tarjetas horizontalmente
    justifyContent: 'space-between', // Distribuye el espacio entre tarjetas
    marginBottom: 20,
  },
  
  gridCard: {
    width: '48%', // Cada tarjeta ocupa el 48% del ancho disponible
    height: 120,
    justifyContent: 'center', // Centra el texto verticalmente
    alignItems: 'center', // Centra el texto horizontalmente
    borderRadius: 12,
    elevation: 3,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  
  cardTitle: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  
  // Estilos para vista de lista
  listCard: {
    flexDirection: 'row', // Organiza elementos horizontalmente
    backgroundColor: 'white',
    marginBottom: 15,
    borderRadius: 12,
    padding: 15,
    alignItems: 'center', // Centra elementos verticalmente
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
  },
  
  listCardColor: {
    width: 20,
    height: 20,
    borderRadius: 10,
    marginRight: 15,
  },
  
  listCardContent: {
    flex: 1, // Ocupa todo el espacio disponible
  },
  
  listCardTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 4,
  },
  
  listCardSubtitle: {
    fontSize: 14,
    color: '#666',
  },
  
  listCardButton: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: '#007bff',
    justifyContent: 'center',
    alignItems: 'center',
  },
  
  listCardButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  
  // Estilos para vista de mosaico
  mosaicRow: {
    flexDirection: 'row', // Organiza tarjetas horizontalmente
    justifyContent: 'space-between', // Distribuye el espacio entre tarjetas
    marginBottom: 20,
  },
  
  mosaicCardLarge: {
    width: '100%', // Ocupa todo el ancho disponible
    height: 200,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 12,
    elevation: 3,
  },
  
  mosaicCardMedium: {
    width: '48%', // Cada tarjeta ocupa el 48% del ancho disponible
    height: 150,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 12,
    elevation: 3,
  },
  
  mosaicCardSmall: {
    width: '31%', // Cada tarjeta ocupa el 31% del ancho disponible
    height: 100,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 12,
    elevation: 3,
  },
  
  mosaicCardTitle: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
    textAlign: 'center',
  },
});

export default AdvancedFlexboxLayout;
```

### **3. Hook Personalizado para Layouts**

```javascript:src/hooks/useLayoutManager.js
import { useState, useCallback, useMemo } from 'react';
import { Dimensions } from 'react-native';

// Hook personalizado para manejar layouts responsivos
const useLayoutManager = (breakpoints = {}) => {
  // Obtener dimensiones de la pantalla
  const screenDimensions = useMemo(() => {
    const { width, height } = Dimensions.get('window');
    return { width, height };
  }, []);
  
  // Estado para el layout actual
  const [currentLayout, setCurrentLayout] = useState('default');
  
  // Estado para la orientación
  const [orientation, setOrientation] = useState(
    screenDimensions.width > screenDimensions.height ? 'landscape' : 'portrait'
  );
  
  // Breakpoints por defecto
  const defaultBreakpoints = {
    small: 480,
    medium: 768,
    large: 1024,
    xlarge: 1200,
    ...breakpoints
  };
  
  // Función para determinar el tamaño de pantalla
  const getScreenSize = useCallback(() => {
    const { width } = screenDimensions;
    
    if (width < defaultBreakpoints.small) return 'xs';
    if (width < defaultBreakpoints.medium) return 'small';
    if (width < defaultBreakpoints.large) return 'medium';
    if (width < defaultBreakpoints.xlarge) return 'large';
    return 'xlarge';
  }, [screenDimensions, defaultBreakpoints]);
  
  // Función para obtener el layout apropiado
  const getLayout = useCallback((layouts) => {
    const screenSize = getScreenSize();
    const currentOrientation = orientation;
    
    // Buscar el layout más apropiado
    const layoutKey = `${screenSize}_${currentOrientation}`;
    
    if (layouts[layoutKey]) {
      return layouts[layoutKey];
    }
    
    // Fallback al layout por defecto
    if (layouts[screenSize]) {
      return layouts[screenSize];
    }
    
    // Fallback al layout base
    return layouts.default || layouts.medium || {};
  }, [getScreenSize, orientation]);
  
  // Función para calcular dimensiones responsivas
  const getResponsiveDimensions = useCallback((dimensions) => {
    const screenSize = getScreenSize();
    const { width, height } = screenDimensions;
    
    const responsiveDimensions = {};
    
    Object.entries(dimensions).forEach(([key, value]) => {
      if (typeof value === 'object' && value.responsive) {
        // Valor responsivo basado en breakpoints
        responsiveDimensions[key] = value[screenSize] || value.default;
      } else if (typeof value === 'number' && value > 0 && value <= 1) {
        // Valor relativo (porcentaje de pantalla)
        responsiveDimensions[key] = value * (key.includes('Width') ? width : height);
      } else {
        // Valor fijo
        responsiveDimensions[key] = value;
      }
    });
    
    return responsiveDimensions;
  }, [getScreenSize, screenDimensions]);
  
  // Función para crear estilos responsivos
  const createResponsiveStyles = useCallback((styleDefinitions) => {
    const screenSize = getScreenSize();
    const responsiveStyles = {};
    
    Object.entries(styleDefinitions).forEach(([styleName, styleValue]) => {
      if (typeof styleValue === 'object' && styleValue.responsive) {
        // Estilo responsivo
        responsiveStyles[styleName] = styleValue[screenSize] || styleValue.default;
      } else {
        // Estilo fijo
        responsiveStyles[styleName] = styleValue;
      }
    });
    
    return responsiveStyles;
  }, [getScreenSize]);
  
  // Función para actualizar orientación
  const updateOrientation = useCallback((newOrientation) => {
    setOrientation(newOrientation);
  }, []);
  
  // Función para cambiar layout manualmente
  const setLayout = useCallback((layoutName) => {
    setCurrentLayout(layoutName);
  }, []);
  
  // Valores calculados
  const screenSize = getScreenSize();
  const isLandscape = orientation === 'landscape';
  const isPortrait = orientation === 'portrait';
  const isSmallScreen = screenSize === 'xs' || screenSize === 'small';
  const isLargeScreen = screenSize === 'large' || screenSize === 'xlarge';
  
  return {
    // Estado
    currentLayout,
    orientation,
    screenSize,
    screenDimensions,
    
    // Banderas de estado
    isLandscape,
    isPortrait,
    isSmallScreen,
    isLargeScreen,
    
    // Funciones
    getLayout,
    getResponsiveDimensions,
    createResponsiveStyles,
    updateOrientation,
    setLayout,
    
    // Breakpoints
    breakpoints: defaultBreakpoints,
  };
};

export default useLayoutManager;
```

### **4. Utilidades para Layout**

```javascript:src/utils/layoutUtils.js
// Utilidades para trabajar con layouts en React Native

// Función para calcular posiciones absolutas
export const calculateAbsolutePosition = (containerDimensions, elementDimensions, position) => {
  const { width: containerWidth, height: containerHeight } = containerDimensions;
  const { width: elementWidth, height: elementHeight } = elementDimensions;
  
  let left, top;
  
  switch (position.horizontal) {
    case 'left':
      left = position.margin || 0;
      break;
    case 'center':
      left = (containerWidth - elementWidth) / 2;
      break;
    case 'right':
      left = containerWidth - elementWidth - (position.margin || 0);
      break;
    default:
      left = position.margin || 0;
  }
  
  switch (position.vertical) {
    case 'top':
      top = position.margin || 0;
      break;
    case 'center':
      top = (containerHeight - elementHeight) / 2;
      break;
    case 'bottom':
      top = containerHeight - elementHeight - (position.margin || 0);
      break;
    default:
      top = position.margin || 0;
  }
  
  return { left, top };
};

// Función para crear grid de elementos
export const createGrid = (items, columns, spacing = 0) => {
  const rows = [];
  let currentRow = [];
  
  items.forEach((item, index) => {
    currentRow.push(item);
    
    if (currentRow.length === columns || index === items.length - 1) {
      rows.push([...currentRow]);
      currentRow = [];
    }
  });
  
  return rows.map((row, rowIndex) => ({
    row,
    rowIndex,
    items: row.map((item, colIndex) => ({
      item,
      colIndex,
      rowIndex,
      position: {
        x: colIndex * (100 / columns),
        y: rowIndex * (100 / rows.length),
      }
    }))
  }));
};

// Función para calcular dimensiones responsivas
export const getResponsiveDimensions = (baseDimensions, screenSize, breakpoints) => {
  const multipliers = {
    xs: 0.8,    // Pantallas muy pequeñas
    small: 0.9, // Pantallas pequeñas
    medium: 1,  // Pantallas medianas (base)
    large: 1.1, // Pantallas grandes
    xlarge: 1.2, // Pantallas muy grandes
  };
  
  const multiplier = multipliers[screenSize] || 1;
  
  return Object.entries(baseDimensions).reduce((acc, [key, value]) => {
    if (typeof value === 'number') {
      acc[key] = Math.round(value * multiplier);
    } else {
      acc[key] = value;
    }
    return acc;
  }, {});
};

// Función para crear estilos de sombra
export const createShadow = (elevation, color = '#000', opacity = 0.25) => {
  return {
    elevation, // Android
    shadowColor: color, // iOS
    shadowOffset: { width: 0, height: elevation / 2 }, // iOS
    shadowOpacity: opacity, // iOS
    shadowRadius: elevation / 2, // iOS
  };
};

// Función para crear estilos de borde
export const createBorder = (width = 1, color = '#ccc', style = 'solid') => {
  return {
    borderWidth: width,
    borderColor: color,
    borderStyle: style,
  };
};

// Función para crear estilos de padding responsivo
export const createResponsivePadding = (basePadding, screenSize) => {
  const multipliers = {
    xs: 0.7,
    small: 0.85,
    medium: 1,
    large: 1.15,
    xlarge: 1.3,
  };
  
  const multiplier = multipliers[screenSize] || 1;
  
  if (typeof basePadding === 'number') {
    return Math.round(basePadding * multiplier);
  }
  
  if (typeof basePadding === 'object') {
    return {
      top: Math.round((basePadding.top || 0) * multiplier),
      right: Math.round((basePadding.right || 0) * multiplier),
      bottom: Math.round((basePadding.bottom || 0) * multiplier),
      left: Math.round((basePadding.left || 0) * multiplier),
    };
  }
  
  return basePadding;
};

// Función para crear estilos de margen responsivo
export const createResponsiveMargin = (baseMargin, screenSize) => {
  const multipliers = {
    xs: 0.7,
    small: 0.85,
    medium: 1,
    large: 1.15,
    xlarge: 1.3,
  };
  
  const multiplier = multipliers[screenSize] || 1;
  
  if (typeof baseMargin === 'number') {
    return Math.round(baseMargin * multiplier);
  }
  
  if (typeof baseMargin === 'object') {
    return {
      top: Math.round((baseMargin.top || 0) * multiplier),
      right: Math.round((baseMargin.right || 0) * multiplier),
      bottom: Math.round((baseMargin.bottom || 0) * multiplier),
      left: Math.round((baseMargin.left || 0) * multiplier),
    };
  }
  
  return baseMargin;
};

// Función para validar layout
export const validateLayout = (layout) => {
  const errors = [];
  
  if (!layout.container) {
    errors.push('El layout debe tener un contenedor definido');
  }
  
  if (!layout.children || layout.children.length === 0) {
    errors.push('El layout debe tener al menos un elemento hijo');
  }
  
  // Verificar que los elementos hijos tengan posiciones válidas
  layout.children?.forEach((child, index) => {
    if (!child.position) {
      errors.push(`El elemento hijo ${index} debe tener una posición definida`);
    }
    
    if (typeof child.position.x !== 'number' || typeof child.position.y !== 'number') {
      errors.push(`El elemento hijo ${index} debe tener coordenadas numéricas`);
    }
  });
  
  return {
    isValid: errors.length === 0,
    errors,
  };
};
```

---

## 🧪 Casos de Uso

### **Caso 1: Layout de Dashboard**
```javascript
const DashboardLayout = () => {
  return (
    <View style={styles.container}>
      {/* Header fijo */}
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Dashboard</Text>
      </View>
      
      {/* Contenido principal con scroll */}
      <ScrollView style={styles.content}>
        {/* Primera fila: métricas */}
        <View style={styles.metricsRow}>
          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>1,234</Text>
            <Text style={styles.metricLabel}>Usuarios</Text>
          </View>
          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>567</Text>
            <Text style={styles.metricLabel}>Ventas</Text>
          </View>
        </View>
        
        {/* Segunda fila: gráficos */}
        <View style={styles.chartsRow}>
          <View style={styles.chartCard}>
            <Text style={styles.chartTitle}>Gráfico 1</Text>
          </View>
          <View style={styles.chartCard}>
            <Text style={styles.chartTitle}>Gráfico 2</Text>
          </View>
        </View>
      </ScrollView>
    </View>
  );
};
```

### **Caso 2: Layout de Formulario**
```javascript
const FormLayout = () => {
  return (
    <View style={styles.container}>
      {/* Header del formulario */}
      <View style={styles.formHeader}>
        <Text style={styles.formTitle}>Nuevo Usuario</Text>
      </View>
      
      {/* Contenido del formulario */}
      <ScrollView style={styles.formContent}>
        {/* Campo de nombre */}
        <View style={styles.inputGroup}>
          <Text style={styles.inputLabel}>Nombre</Text>
          <TextInput style={styles.textInput} placeholder="Ingresa tu nombre" />
        </View>
        
        {/* Campo de email */}
        <View style={styles.inputGroup}>
          <Text style={styles.inputLabel}>Email</Text>
          <TextInput style={styles.textInput} placeholder="Ingresa tu email" />
        </View>
        
        {/* Botones de acción */}
        <View style={styles.buttonGroup}>
          <TouchableOpacity style={styles.cancelButton}>
            <Text style={styles.cancelButtonText}>Cancelar</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.submitButton}>
            <Text style={styles.submitButtonText}>Guardar</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </View>
  );
};
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Crear un Layout de Galería**
Crea un layout que muestre imágenes en una cuadrícula adaptativa:
- 2 columnas en pantallas pequeñas
- 3 columnas en pantallas medianas
- 4 columnas en pantallas grandes

### **Ejercicio 2: Layout de Perfil de Usuario**
Crea un layout de perfil con:
- Header con foto de perfil y nombre
- Sección de información personal
- Sección de estadísticas
- Sección de acciones

### **Ejercicio 3: Layout de Navegación**
Implementa un layout con:
- Navegación inferior fija
- Contenido principal que se adapta
- Header que cambia según la vista

---

## 🚀 Proyecto de la Clase

### **App de Layout Responsivo**

Crea una aplicación que demuestre diferentes layouts:
- **Layout de cuadrícula**: Para mostrar productos
- **Layout de lista**: Para mostrar información detallada
- **Layout de mosaico**: Para mostrar contenido multimedia
- **Layout adaptativo**: Que cambie según el tamaño de pantalla

**Requisitos:**
1. Usar Flexbox para todos los layouts
2. Implementar layouts responsivos
3. Crear transiciones suaves entre layouts
4. Usar el hook personalizado de layout
5. Implementar orientación landscape/portrait

**Estructura sugerida:**
```
src/
├── components/
│   ├── ResponsiveLayout.js
│   ├── GridLayout.js
│   ├── ListLayout.js
│   └── MosaicLayout.js
├── hooks/
│   └── useLayoutManager.js
├── utils/
│   └── layoutUtils.js
└── screens/
    └── LayoutDemoScreen.js
```

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Native Layout](https://reactnative.dev/docs/layout-props)
- [Flexbox en React Native](https://reactnative.dev/docs/flexbox)

### **Artículos Recomendados:**
- "Flexbox en React Native: Guía completa"
- "Cómo crear layouts responsivos en React Native"
- "Mejores prácticas para layouts móviles"

### **Herramientas:**
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)
- [Flipper](https://fbflipper.com/) - Herramienta de debugging
- [Reactotron](https://github.com/infinitered/reactotron)

---

## 📝 Resumen de la Clase

### **Conceptos Clave:**
- **Flexbox**: Sistema de layout unidimensional para organizar elementos
- **Layout responsivo**: Interfaces que se adaptan a diferentes tamaños de pantalla
- **Propiedades Flexbox**: flexDirection, justifyContent, alignItems, flex
- **Posicionamiento**: Absoluto, relativo y flexbox
- **Breakpoints**: Puntos de cambio para layouts responsivos

### **Habilidades Desarrolladas:**
- ✅ Crear layouts básicos con Flexbox
- ✅ Implementar layouts responsivos
- ✅ Usar propiedades de Flexbox avanzadas
- ✅ Crear hooks personalizados para layout
- ✅ Manejar diferentes orientaciones de pantalla

### **Próximos Pasos:**
En la siguiente clase aprenderemos sobre **Componentes Personalizados**, que te permitirá crear componentes reutilizables y organizar mejor tu código.

---

## 🔗 Enlaces de Navegación

- **⬅️ Clase Anterior**: [JSX y Estructura](clase_2_jsx_estructura.md)
- **➡️ Siguiente Clase**: [Componentes Personalizados](clase_4_componentes_personalizados.md)
- **📚 [README del Módulo](README.md)**
- **🏠 [Volver al Inicio](../../README.md)**
