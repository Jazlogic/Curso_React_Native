# 📚 Módulo 4: Navegación y Routing

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [Módulo 3: Estado y Props](../junior_3/README.md)
- **➡️ Siguiente**: [Módulo 5: Gestión de Estado](../midLevel_2/README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Descripción del Módulo

El **Módulo 4: Navegación y Routing** es fundamental para crear aplicaciones React Native con múltiples pantallas y navegación fluida entre ellas. Aprenderás a implementar diferentes tipos de navegación, desde la básica hasta sistemas completamente personalizados.

### **¿Qué aprenderás?**

- **Navegación básica**: Conceptos fundamentales y configuración inicial
- **Stack Navigator**: Navegación con pilas de pantallas y transiciones
- **Tab Navigator**: Navegación por pestañas con múltiples stacks
- **Drawer Navigator**: Menús laterales deslizables
- **Navegación personalizada**: Sistemas únicos con transiciones y gestos

---

## 📚 Clases del Módulo

### **Clase 1: [Navegación Básica](clase_1_navegacion_basica.md)**
- **Objetivo**: Comprender los conceptos fundamentales de navegación en React Native
- **Contenido**: 
  - Configuración de React Navigation
  - Navegación básica entre pantallas
  - Estructura de navegación simple
  - Diferencias entre navegación web y móvil
- **Duración**: 2-3 horas
- **Prerrequisitos**: Módulo 2 completado

### **Clase 2: [Stack Navigator Avanzado](clase_2_stack_navigator.md)**
- **Objetivo**: Dominar las opciones avanzadas del Stack Navigator
- **Contenido**:
  - Transiciones y animaciones personalizadas
  - Navegación con múltiples stacks
  - Navegación anidada y compleja
  - Optimización de rendimiento
- **Duración**: 3-4 horas
- **Prerrequisitos**: Clase 1 completada

### **Clase 3: [Tab Navigator](clase_3_tab_navigator.md)**
- **Objetivo**: Comprender el funcionamiento del Tab Navigator en React Native
- **Contenido**:
  - Navegación por pestañas con múltiples stacks
  - Personalización de tabs y iconos
  - Navegación anidada con tabs y stacks
  - Experiencias de usuario fluidas
- **Duración**: 3-4 horas
- **Prerrequisitos**: Clase 2 completada

### **Clase 4: [Drawer Navigator](clase_4_drawer_navigator.md)**
- **Objetivo**: Comprender el funcionamiento del Drawer Navigator en React Native
- **Contenido**:
  - Menús laterales deslizables
  - Personalización del drawer y opciones
  - Navegación anidada con drawer y otros navegadores
  - Experiencias de usuario intuitivas
- **Duración**: 3-4 horas
- **Prerrequisitos**: Clase 3 completada

### **Clase 5: [Navegación Personalizada](clase_5_navegacion_personalizada.md)**
- **Objetivo**: Crear navegación completamente personalizada
- **Contenido**:
  - Transiciones y animaciones únicas
  - Navegadores personalizados
  - Gestos e interacciones avanzadas
  - Experiencias de navegación únicas
- **Duración**: 4-5 horas
- **Prerrequisitos**: Clase 4 completada

---

## 🚀 Proyecto Integrador del Módulo

### **App de Navegación Completa**

Crea una aplicación que demuestre todos los tipos de navegación aprendidos:

#### **Requisitos del Proyecto:**
1. **Navegación básica**: Implementar navegación entre pantallas principales
2. **Stack Navigator**: Crear flujos de navegación con transiciones
3. **Tab Navigator**: Implementar navegación por pestañas
4. **Drawer Navigator**: Agregar menú lateral deslizable
5. **Navegación personalizada**: Crear transiciones únicas

#### **Estructura Sugerida:**
```
src/
├── navigation/
│   ├── AppNavigator.js          # Navegador principal
│   ├── AuthNavigator.js         # Navegador de autenticación
│   ├── MainNavigator.js         # Navegador principal con tabs
│   ├── TabNavigator.js          # Navegador de pestañas
│   ├── DrawerNavigator.js       # Navegador de drawer
│   └── CustomNavigator.js       # Navegador personalizado
├── screens/
│   ├── auth/
│   │   ├── LoginScreen.js
│   │   └── RegisterScreen.js
│   ├── main/
│   │   ├── HomeScreen.js
│   │   ├── ProfileScreen.js
│   │   ├── SettingsScreen.js
│   │   └── NotificationsScreen.js
│   └── custom/
│       ├── CustomTransitionScreen.js
│       └── GestureScreen.js
├── hooks/
│   ├── useNavigation.js
│   ├── useTabNavigation.js
│   ├── useDrawerNavigation.js
│   └── useCustomNavigation.js
└── utils/
    ├── navigationUtils.js
    ├── tabNavigatorUtils.js
    └── drawerNavigatorUtils.js
```

#### **Funcionalidades Clave:**
- **Autenticación**: Flujo de login/registro con Stack Navigator
- **Navegación principal**: Tabs para secciones principales
- **Menú lateral**: Drawer para acceso a todas las funcionalidades
- **Transiciones personalizadas**: Animaciones únicas entre pantallas
- **Gestos avanzados**: Navegación por gestos táctiles

---

## 🎯 Objetivos de Aprendizaje

### **Al finalizar este módulo serás capaz de:**

✅ **Configurar React Navigation** en proyectos React Native
✅ **Implementar Stack Navigator** con transiciones personalizadas
✅ **Crear Tab Navigator** con navegación anidada
✅ **Desarrollar Drawer Navigator** con menús laterales
✅ **Crear navegación personalizada** con transiciones únicas
✅ **Gestionar navegación anidada** entre diferentes tipos de navegadores
✅ **Optimizar el rendimiento** de la navegación
✅ **Implementar gestos personalizados** para navegación
✅ **Crear hooks personalizados** para manejo de navegación
✅ **Desarrollar aplicaciones complejas** con múltiples flujos de navegación

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Navigation](https://reactnavigation.org/)
- [Stack Navigator](https://reactnavigation.org/docs/stack-navigator)
- [Tab Navigator](https://reactnavigation.org/docs/bottom-tab-navigator)
- [Drawer Navigator](https://reactnavigation.org/docs/drawer-navigator)

### **Artículos Recomendados:**
- "Guía completa de React Navigation en React Native"
- "Mejores prácticas para navegación en apps móviles"
- "Cómo optimizar el rendimiento de la navegación"

### **Herramientas Útiles:**
- [React Navigation DevTools](https://github.com/react-navigation/devtools)
- [Flipper](https://fbflipper.com/) para debugging de navegación

---

## 🔗 Enlaces de Navegación

### **Navegación del Módulo:**
- **Clase 1**: [Navegación Básica](clase_1_navegacion_basica.md)
- **Clase 2**: [Stack Navigator Avanzado](clase_2_stack_navigator.md)
- **Clase 3**: [Tab Navigator](clase_3_tab_navigator.md)
- **Clase 4**: [Drawer Navigator](clase_4_drawer_navigator.md)
- **Clase 5**: [Navegación Personalizada](clase_5_navegacion_personalizada.md)

### **Navegación entre Módulos:**
- **⬅️ Módulo Anterior**: [Módulo 2: Componentes Básicos](../junior_2/README.md)
- **➡️ Siguiente Módulo**: [Módulo 4: Estado y Gestión de Datos](../midLevel_2/README.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 📝 Resumen del Módulo

### **Conceptos Clave Aprendidos:**
- **Sistemas de navegación**: Diferentes tipos y sus casos de uso
- **Navegación anidada**: Combinación de múltiples navegadores
- **Transiciones personalizadas**: Animaciones únicas entre pantallas
- **Gestos táctiles**: Interacciones avanzadas para navegación
- **Performance**: Optimización de la navegación en React Native

### **Habilidades Desarrolladas:**
- ✅ Configuración completa de React Navigation
- ✅ Implementación de todos los tipos de navegadores
- ✅ Creación de navegación personalizada
- ✅ Gestión de navegación anidada
- ✅ Optimización de rendimiento

### **Próximos Pasos:**
En el siguiente módulo aprenderemos sobre **Estado y Gestión de Datos**, donde aplicarás los conocimientos de navegación para crear aplicaciones con estado complejo y persistencia de datos.

---

## 🎉 ¡Felicidades!

Has completado exitosamente el **Módulo 3: Navegación y Routing**. Ahora tienes las habilidades necesarias para crear aplicaciones React Native con navegación profesional y experiencias de usuario fluidas.

**Continúa tu aprendizaje con el siguiente módulo para convertirte en un desarrollador React Native experto! 🚀**
