# 🎨 Clase 3: Layout y Composición

## 📍 Navegación
- **📚 Módulo**: [Módulo 9: UI-UX y Diseño de Interfaces](README.md)
- **⬅️ Anterior**: [Clase 2: Diseño Visual y Branding](clase_2_diseno_visual_branding.md)
- **➡️ Siguiente**: [Clase 4: Accesibilidad e Inclusión](clase_4_accesibilidad_inclusion.md)

---

## 🎯 Objetivos de la Clase

### **Al Finalizar esta Clase, Serás Capaz de:**
1. **Comprender** los principios fundamentales de layout y composición
2. **Diseñar** sistemas de grid profesionales y escalables
3. **Crear** layouts responsive para diferentes dispositivos
4. **Implementar** patrones de composición efectivos
5. **Optimizar** el espaciado y la jerarquía visual
6. **Aplicar** principios de layout en React Native

---

## 📐 Principios de Layout y Composición

### **¿Por qué es Importante el Layout?**

El layout es fundamental para:
- **Organizar** la información de forma lógica
- **Guiar** la atención del usuario
- **Mejorar** la legibilidad del contenido
- **Crear** jerarquía visual clara
- **Optimizar** el uso del espacio disponible

### **Principios Fundamentales**

#### **1. Proximidad**
- **Elementos relacionados** deben estar agrupados
- **Espaciado consistente** entre grupos
- **Separación visual** clara entre secciones

#### **2. Alineación**
- **Grid invisible** que conecta elementos
- **Bordes y ejes** alineados consistentemente
- **Relaciones visuales** claras entre elementos

#### **3. Repetición**
- **Patrones consistentes** en toda la interfaz
- **Elementos visuales** que se repiten
- **Coherencia** en el diseño

#### **4. Contraste**
- **Diferencias claras** entre elementos
- **Jerarquía visual** establecida
- **Foco** en elementos importantes

---

## 🔲 Sistemas de Grid

### **¿Qué es un Sistema de Grid?**

Un sistema de grid es una estructura de columnas y filas que ayuda a organizar el contenido de manera consistente y profesional.

### **Tipos de Grid**

#### **Grid de 12 Columnas**
- **Flexibilidad** para diferentes layouts
- **Divisibilidad** por 2, 3, 4, 6
- **Estándar** en diseño web y móvil

#### **Grid de 8 Columnas**
- **Simplicidad** para layouts básicos
- **Divisibilidad** por 2 y 4
- **Ideal** para aplicaciones móviles

### **Implementación en React Native**

#### **Sistema de Grid Profesional**
```jsx
const gridSystem = {
  breakpoints: {
    xs: 320, sm: 576, md: 768, lg: 1024, xl: 1200,
  },
  
  columns: {
    xs: 4, sm: 6, md: 8, lg: 12, xl: 12,
  },
  
  gutters: {
    xs: 8, sm: 12, md: 16, lg: 24, xl: 32,
  },
  
  margins: {
    xs: 16, sm: 24, md: 32, lg: 48, xl: 64,
  }
};

const useGrid = () => {
  const [screenWidth, setScreenWidth] = useState(Dimensions.get('window').width);
  
  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => {
      setScreenWidth(window.width);
    });
    
    return () => subscription?.remove();
  }, []);
  
  const getBreakpoint = () => {
    const { width } = { width: screenWidth };
    
    if (width >= 1200) return 'xl';
    if (width >= 1024) return 'lg';
    if (width >= 768) return 'md';
    if (width >= 576) return 'sm';
    return 'xs';
  };
  
  const getCurrentGrid = () => {
    const breakpoint = getBreakpoint();
    return {
      columns: gridSystem.columns[breakpoint],
      gutters: gridSystem.gutters[breakpoint],
      margins: gridSystem.margins[breakpoint],
      breakpoint
    };
  };
  
  return { getCurrentGrid, screenWidth };
};
```

#### **Componentes Grid**
```jsx
const GridContainer = ({ children, fluid = false, style }) => {
  const { getCurrentGrid } = useGrid();
  const { margins } = getCurrentGrid();
  
  const containerStyle = {
    paddingHorizontal: fluid ? 0 : margins,
    ...style
  };
  
  return (
    <View style={containerStyle}>
      {children}
    </View>
  );
};

const Row = ({ children, style, alignItems = 'flex-start', justifyContent = 'flex-start' }) => {
  const { getCurrentGrid } = useGrid();
  const { gutters } = getCurrentGrid();
  
  const rowStyle = {
    flexDirection: 'row',
    flexWrap: 'wrap',
    marginHorizontal: -gutters / 2,
    alignItems,
    justifyContent,
    ...style
  };
  
  return (
    <View style={rowStyle}>
      {children}
    </View>
  );
};

const Col = ({ children, size, xs, sm, md, lg, xl, style, ...props }) => {
  const { getCurrentGrid } = useGrid();
  const { columns, gutters } = getCurrentGrid();
  
  const getColumnSize = () => {
    const breakpoint = getCurrentGrid().breakpoint;
    
    if (breakpoint === 'xl' && xl !== undefined) return xl;
    if (breakpoint === 'lg' && lg !== undefined) return lg;
    if (breakpoint === 'md' && md !== undefined) return md;
    if (breakpoint === 'sm' && sm !== undefined) return sm;
    if (breakpoint === 'xs' && xs !== undefined) return xs;
    
    return size || 12;
  };
  
  const columnSize = getColumnSize();
  const widthPercentage = (columnSize / columns) * 100;
  
  const colStyle = {
    width: `${widthPercentage}%`,
    paddingHorizontal: gutters / 2,
    ...style
  };
  
  return (
    <View style={colStyle} {...props}>
      {children}
    </View>
  );
};
```

---

## 📱 Layouts Responsive

### **¿Qué es el Diseño Responsive?**

El diseño responsive es la capacidad de una interfaz de adaptarse a diferentes tamaños de pantalla y orientaciones, manteniendo la usabilidad y la estética.

### **Estrategias de Diseño Responsive**

#### **Mobile-First**
- **Diseño base** para dispositivos móviles
- **Progresión** hacia pantallas más grandes
- **Optimización** para el caso de uso más común

#### **Breakpoints Inteligentes**
- **Basados en contenido** no en dispositivos específicos
- **Adaptación fluida** entre breakpoints
- **Consideración** del contexto de uso

### **Implementación en React Native**

#### **Hook de Responsive Design**
```jsx
const useResponsive = () => {
  const [dimensions, setDimensions] = useState({
    width: Dimensions.get('window').width,
    height: Dimensions.get('window').height,
    scale: Dimensions.get('window').scale,
    fontScale: Dimensions.get('window').fontScale,
  });
  
  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => {
      setDimensions({
        width: window.width,
        height: window.height,
        scale: window.scale,
        fontScale: window.fontScale,
      });
    });
    
    return () => subscription?.remove();
  }, []);
  
  const isPortrait = dimensions.height > dimensions.width;
  const isLandscape = dimensions.width > dimensions.height;
  
  const getBreakpoint = () => {
    const { width } = dimensions;
    
    if (width >= 1200) return 'xl';
    if (width >= 1024) return 'lg';
    if (width >= 768) return 'md';
    if (width >= 576) return 'sm';
    return 'xs';
  };
  
  const isMobile = () => getBreakpoint() === 'xs' || getBreakpoint() === 'sm';
  const isTablet = () => getBreakpoint() === 'md';
  const isDesktop = () => getBreakpoint() === 'lg' || getBreakpoint() === 'xl';
  
  return {
    dimensions,
    isPortrait,
    isLandscape,
    getBreakpoint,
    isMobile,
    isTablet,
    isDesktop,
  };
};
```

#### **Componente de Layout Adaptativo**
```jsx
const ResponsiveLayout = ({ mobileLayout, tabletLayout, desktopLayout, children }) => {
  const { isMobile, isTablet, isDesktop } = useResponsive();
  
  if (isMobile && mobileLayout) {
    return mobileLayout;
  }
  
  if (isTablet && tabletLayout) {
    return tabletLayout;
  }
  
  if (isDesktop && desktopLayout) {
    return desktopLayout;
  }
  
  return children;
};

const AdaptiveExample = () => {
  const { isMobile, isTablet, isDesktop } = useResponsive();
  
  return (
    <ResponsiveLayout
      mobileLayout={
        <View style={styles.mobileContainer}>
          <Text style={styles.title}>Vista Móvil</Text>
          <View style={styles.mobileCard}>
            <Text>Contenido optimizado para móvil</Text>
          </View>
        </View>
      }
      tabletLayout={
        <View style={styles.tabletContainer}>
          <Text style={styles.title}>Vista Tablet</Text>
          <Row>
            <Col size={6}>
              <View style={styles.tabletCard}>
                <Text>Columna izquierda</Text>
              </View>
            </Col>
            <Col size={6}>
              <View style={styles.tabletCard}>
                <Text>Columna derecha</Text>
              </View>
            </Col>
          </Row>
        </View>
      }
      desktopLayout={
        <View style={styles.desktopContainer}>
          <Text style={styles.title}>Vista Desktop</Text>
          <Row>
            <Col size={3}>
              <View style={styles.desktopCard}>
                <Text>Sidebar</Text>
              </View>
            </Col>
            <Col size={6}>
              <View style={styles.desktopCard}>
                <Text>Contenido principal</Text>
              </View>
            </Col>
            <Col size={3}>
              <View style={styles.desktopCard}>
                <Text>Sidebar derecho</Text>
              </View>
            </Col>
          </Row>
        </View>
      }
    >
      <View style={styles.defaultContainer}>
        <Text>Layout por defecto</Text>
      </View>
    </ResponsiveLayout>
  );
};
```

---

## 🎨 Patrones de Composición

### **¿Qué son los Patrones de Composición?**

Los patrones de composición son estructuras visuales probadas que ayudan a organizar el contenido de manera efectiva y atractiva.

### **Patrones Comunes**

#### **Layout de Tarjetas (Card Layout)**
- **Información agrupada** en contenedores visuales
- **Jerarquía clara** de contenido
- **Fácil escaneo** de información

#### **Layout de Lista (List Layout)**
- **Información secuencial** organizada verticalmente
- **Navegación fácil** entre elementos
- **Eficiente** uso del espacio vertical

#### **Layout de Grid (Grid Layout)**
- **Contenido visual** organizado en matriz
- **Comparación fácil** entre elementos
- **Uso eficiente** del espacio horizontal

### **Implementación en React Native**

#### **Componente de Layout de Tarjetas**
```jsx
const CardLayout = ({ items, columns = 2, spacing = 16, renderItem, style }) => {
  const { isMobile, isTablet } = useResponsive();
  
  const getColumns = () => {
    if (isMobile) return 1;
    if (isTablet) return Math.min(columns, 2);
    return columns;
  };
  
  const currentColumns = getColumns();
  const itemWidth = `${100 / currentColumns}%`;
  
  return (
    <View style={[styles.cardLayout, style]}>
      {items.map((item, index) => (
        <View
          key={index}
          style={[
            styles.cardItem,
            {
              width: itemWidth,
              paddingHorizontal: spacing / 2,
              paddingBottom: spacing,
            }
          ]}
        >
          {renderItem(item, index)}
        </View>
      ))}
    </View>
  );
};

const CardLayoutExample = () => {
  const items = [
    { id: 1, title: 'Tarjeta 1', content: 'Contenido de la primera tarjeta' },
    { id: 2, title: 'Tarjeta 2', content: 'Contenido de la segunda tarjeta' },
    { id: 3, title: 'Tarjeta 3', content: 'Contenido de la tercera tarjeta' },
    { id: 4, title: 'Tarjeta 4', content: 'Contenido de la cuarta tarjeta' },
  ];
  
  const renderCard = (item) => (
    <View style={styles.card}>
      <Text style={styles.cardTitle}>{item.title}</Text>
      <Text style={styles.cardContent}>{item.content}</Text>
    </View>
  );
  
  return (
    <CardLayout
      items={items}
      columns={2}
      spacing={16}
      renderItem={renderCard}
    />
  );
};
```

#### **Componente de Layout de Lista**
```jsx
const ListLayout = ({ items, renderItem, separator, style, ...props }) => {
  const renderSeparator = () => {
    if (separator) return separator;
    
    return (
      <View style={styles.defaultSeparator} />
    );
  };
  
  return (
    <View style={[styles.listLayout, style]} {...props}>
      {items.map((item, index) => (
        <View key={index}>
          {renderItem(item, index)}
          {index < items.length - 1 && renderSeparator()}
        </View>
      ))}
    </View>
  );
};

const ListLayoutExample = () => {
  const items = [
    { id: 1, title: 'Elemento 1', subtitle: 'Descripción del elemento 1' },
    { id: 2, title: 'Elemento 2', subtitle: 'Descripción del elemento 2' },
    { id: 3, title: 'Elemento 3', subtitle: 'Descripción del elemento 3' },
  ];
  
  const renderListItem = (item) => (
    <View style={styles.listItem}>
      <Text style={styles.listItemTitle}>{item.title}</Text>
      <Text style={styles.listItemSubtitle}>{item.subtitle}</Text>
    </View>
  );
  
  return (
    <ListLayout
      items={items}
      renderItem={renderListItem}
      separator={<View style={styles.customSeparator} />}
    />
  );
};
```

---

## 📏 Espaciado y Jerarquía Visual

### **¿Por qué es Importante el Espaciado?**

El espaciado es crucial para:
- **Respiración visual** de la interfaz
- **Agrupación lógica** de elementos
- **Jerarquía visual** clara
- **Legibilidad** del contenido
- **Experiencia** del usuario

### **Sistema de Espaciado**

#### **Escala de Espaciado**
- **Base de 8px** para consistencia
- **Multiplicadores** para diferentes niveles
- **Aplicación consistente** en toda la interfaz

#### **Tipos de Espaciado**
- **Padding**: Espacio interno de elementos
- **Margin**: Espacio entre elementos
- **Gap**: Espacio en layouts flexbox/grid

### **Implementación en React Native**

#### **Sistema de Espaciado Avanzado**
```jsx
const spacingSystem = {
  base: 8,
  
  scale: {
    0: 0, 1: 8, 2: 16, 3: 24, 4: 32,
    5: 40, 6: 48, 7: 56, 8: 64, 9: 72, 10: 80,
  },
  
  components: {
    button: {
      padding: { vertical: 16, horizontal: 24 },
      margin: { vertical: 8, horizontal: 0 },
    },
    card: {
      padding: 16,
      margin: { vertical: 8, horizontal: 0 },
    },
    input: {
      padding: { vertical: 12, horizontal: 16 },
      margin: { vertical: 8, horizontal: 0 },
    },
    section: {
      padding: { vertical: 32, horizontal: 0 },
      margin: { vertical: 16, horizontal: 0 },
    },
    container: {
      padding: { vertical: 0, horizontal: 16 },
      margin: { vertical: 0, horizontal: 0 },
    }
  },
  
  responsive: {
    xs: { container: 16, section: 24, card: 12 },
    sm: { container: 24, section: 32, card: 16 },
    md: { container: 32, section: 40, card: 20 },
    lg: { container: 48, section: 56, card: 24 },
    xl: { container: 64, section: 72, card: 32 },
  }
};

const useSpacing = () => {
  const { getBreakpoint } = useGrid();
  const breakpoint = getBreakpoint();
  
  const getSpacing = (type, size) => {
    if (type === 'scale') {
      return spacingSystem.scale[size];
    }
    
    if (type === 'components') {
      return spacingSystem.components[size];
    }
    
    if (type === 'responsive') {
      return spacingSystem.responsive[breakpoint][size];
    }
    
    return spacingSystem.base;
  };
  
  return { getSpacing, spacingSystem };
};
```

#### **Componente de Espaciado Consistente**
```jsx
const Spacer = ({ size = 1, axis = 'vertical', style }) => {
  const { getSpacing } = useSpacing();
  const spacingValue = getSpacing('scale', size);
  
  const spacerStyle = {
    [axis === 'horizontal' ? 'width' : 'height']: spacingValue,
    ...style
  };
  
  return <View style={spacerStyle} />;
};

const SpacedContainer = ({ children, spacing = 2, direction = 'vertical', style }) => {
  const { getSpacing } = useSpacing();
  const spacingValue = getSpacing('scale', spacing);
  
  const containerStyle = {
    [direction === 'horizontal' ? 'paddingHorizontal' : 'paddingVertical']: spacingValue,
    ...style
  };
  
  return (
    <View style={containerStyle}>
      {children}
    </View>
  );
};

const SpacingExample = () => {
  return (
    <SpacedContainer spacing={3}>
      <Text style={styles.title}>Título Principal</Text>
      
      <Spacer size={2} />
      
      <Text style={styles.subtitle}>Subtítulo</Text>
      
      <Spacer size={1} />
      
      <Text style={styles.body}>
        Este es el contenido del cuerpo con espaciado consistente.
      </Text>
      
      <Spacer size={3} />
      
      <Button title="Acción Principal" />
    </SpacedContainer>
  );
};
```

---

## 📱 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Grid Personalizado**
**Objetivo**: Crear un sistema de grid personalizado para una app específica.

**Requisitos:**
- Grid de 8 columnas para móvil, 12 para tablet
- Breakpoints personalizados según necesidades
- Componentes GridContainer, Row y Col
- Ejemplo de uso con diferentes layouts

**Entregable**: Sistema de grid completo con documentación y ejemplos.

### **Ejercicio 2: Layout Responsive**
**Objetivo**: Diseñar un layout que se adapte a diferentes tamaños de pantalla.

**Requisitos:**
- Layout móvil optimizado para una mano
- Layout tablet con sidebar
- Layout desktop con múltiples columnas
- Hook useResponsive personalizado

**Entregable**: Layout responsive completo con diferentes breakpoints.

### **Ejercicio 3: Patrón de Composición**
**Objetivo**: Implementar un patrón de composición específico.

**Requisitos:**
- Layout de tarjetas con diferentes tamaños
- Layout de lista con separadores personalizados
- Layout de grid con espaciado consistente
- Componentes reutilizables

**Entregable**: Patrón de composición implementado con ejemplos de uso.

---

## 🎯 Resumen de la Clase

### **✅ Lo que has aprendido:**
1. **Principios de layout** - Proximidad, alineación, repetición, contraste
2. **Sistemas de grid** - Estructuras de columnas y filas profesionales
3. **Layouts responsive** - Adaptación a diferentes tamaños de pantalla
4. **Patrones de composición** - Estructuras visuales probadas y efectivas
5. **Espaciado y jerarquía** - Sistemas consistentes para organización visual
6. **Implementación práctica** - Código React Native reutilizable

### **🚀 Próximos pasos:**
- **Práctica**: Completa los ejercicios de esta clase
- **Experimentación**: Crea layouts para diferentes tipos de aplicaciones
- **Documentación**: Desarrolla guías de layout para tu proyecto
- **Siguiente clase**: Accesibilidad e Inclusión

---

## 🔗 Recursos Adicionales

### **📚 Lectura Recomendada**
- **"Grid Systems in Graphic Design"** - Josef Müller-Brockmann
- **"Layout Essentials"** - Beth Tondreau
- **"Responsive Web Design"** - Ethan Marcotte

### **🌐 Recursos Online**
- **CSS Grid**: [css-tricks.com/snippets/css/complete-guide-grid/](https://css-tricks.com/snippets/css/complete-guide-grid/)
- **Flexbox**: [css-tricks.com/snippets/css/a-guide-to-flexbox/](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- **Material Design Layout**: [material.io/design/layout/understanding-layout.html](https://material.io/design/layout/understanding-layout.html)

---

**🎨 ¡Excelente! Has completado la clase de Layout y Composición.**

Ahora tienes un conocimiento sólido de cómo crear layouts profesionales y responsive. En la siguiente clase aprenderemos sobre **Accesibilidad e Inclusión**, donde aplicaremos estos principios de layout para crear interfaces accesibles para todos los usuarios.

**¡Continúa con la siguiente clase para hacer el diseño más inclusivo!** 🚀

### **Clase Siguiente**: [Clase 4: Accesibilidad e Inclusión](clase_4_accesibilidad_inclusion.md)
