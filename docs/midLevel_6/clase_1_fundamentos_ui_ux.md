# 🎨 Clase 1: Fundamentos de UI-UX

## 📍 Navegación
- **📚 Módulo**: [Módulo 9: UI-UX y Diseño de Interfaces](README.md)
- **⬅️ Anterior**: [Clase 5: Publicación y Distribución](../midLevel_5/clase_5_publicacion_distribucion.md)
- **➡️ Siguiente**: [Clase 2: Diseño Visual y Branding](clase_2_diseno_visual_branding.md)

---

## 🎯 Objetivos de la Clase

### **Al Finalizar esta Clase, Serás Capaz de:**
1. **Comprender** los principios fundamentales del diseño de interfaces
2. **Aplicar** las heurísticas de Nielsen en el diseño móvil
3. **Entender** la psicología del usuario y comportamiento humano
4. **Realizar** investigación de usuarios y análisis de necesidades
5. **Crear** wireframes y flujos de usuario efectivos
6. **Implementar** principios de usabilidad en React Native

---

## 🧠 ¿Qué es UI-UX?

### **UI (User Interface) - Interfaz de Usuario**
La **UI** se refiere a la presentación visual y elementos interactivos de una aplicación. Es lo que el usuario ve y toca directamente.

**Componentes de UI:**
- Botones, campos de texto, iconos
- Colores, tipografía, espaciado
- Layouts, navegación, formularios
- Animaciones y transiciones

### **UX (User Experience) - Experiencia de Usuario**
La **UX** abarca toda la experiencia del usuario al interactuar con la aplicación, incluyendo aspectos emocionales, psicológicos y prácticos.

**Elementos de UX:**
- Facilidad de uso y aprendizaje
- Eficiencia en la realización de tareas
- Satisfacción y disfrute del usuario
- Accesibilidad e inclusión

---

## 🎨 Principios Fundamentales de Diseño

### **1. Simplicidad (Keep It Simple)**
**¿Por qué es importante?**
- Reduce la carga cognitiva del usuario
- Acelera el proceso de aprendizaje
- Minimiza errores y confusión

**Cómo aplicarlo:**
```jsx
// ❌ Complicado - Múltiples opciones confusas
<View style={styles.complexContainer}>
  <Text>Configuración Avanzada</Text>
  <Text>Opciones de Personalización</Text>
  <Text>Parámetros del Sistema</Text>
  <Text>Configuración de Red</Text>
</View>

// ✅ Simple - Opciones claras y directas
<View style={styles.simpleContainer}>
  <TouchableOpacity style={styles.option}>
    <Text style={styles.optionText}>Perfil</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.option}>
    <Text style={styles.optionText}>Notificaciones</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.option}>
    <Text style={styles.optionText}>Privacidad</Text>
  </TouchableOpacity>
</View>
```

### **2. Consistencia (Consistency)**
**¿Por qué es importante?**
- Reduce el tiempo de aprendizaje
- Crea confianza en el usuario
- Mejora la eficiencia de uso

**Elementos que deben ser consistentes:**
- **Colores**: Paleta de colores unificada
- **Tipografía**: Jerarquía y estilos de texto
- **Espaciado**: Márgenes y padding uniformes
- **Interacciones**: Comportamiento de botones y gestos

**Ejemplo de sistema de colores consistente:**
```jsx
// Sistema de colores centralizado
const colors = {
  primary: '#007AFF',
  secondary: '#5856D6',
  success: '#34C759',
  warning: '#FF9500',
  error: '#FF3B30',
  background: '#FFFFFF',
  surface: '#F2F2F7',
  text: '#000000',
  textSecondary: '#8E8E93',
  border: '#C6C6C8'
};

// Uso consistente en toda la app
<View style={[styles.container, { backgroundColor: colors.background }]}>
  <Text style={[styles.title, { color: colors.text }]}>Título</Text>
  <TouchableOpacity style={[styles.button, { backgroundColor: colors.primary }]}>
    <Text style={[styles.buttonText, { color: colors.background }]}>Acción</Text>
  </TouchableOpacity>
</View>
```

### **3. Jerarquía Visual (Visual Hierarchy)**
**¿Por qué es importante?**
- Guía la atención del usuario
- Organiza la información de forma lógica
- Facilita la comprensión del contenido

**Elementos de jerarquía:**
- **Tamaño**: Elementos más importantes son más grandes
- **Color**: Colores contrastantes para destacar
- **Espaciado**: Agrupación lógica de elementos
- **Tipografía**: Diferentes pesos y tamaños de fuente

**Ejemplo de jerarquía visual:**
```jsx
const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  // Jerarquía 1: Título principal
  mainTitle: {
    fontSize: 28,
    fontWeight: 'bold',
    color: colors.text,
    marginBottom: 16,
    textAlign: 'center',
  },
  // Jerarquía 2: Subtítulos
  subtitle: {
    fontSize: 20,
    fontWeight: '600',
    color: colors.text,
    marginBottom: 12,
  },
  // Jerarquía 3: Texto del cuerpo
  bodyText: {
    fontSize: 16,
    color: colors.textSecondary,
    lineHeight: 24,
    marginBottom: 8,
  },
  // Jerarquía 4: Texto secundario
  caption: {
    fontSize: 14,
    color: colors.textSecondary,
    fontStyle: 'italic',
  }
});
```

### **4. Feedback y Respuesta (Feedback)**
**¿Por qué es importante?**
- Confirma que la acción del usuario fue recibida
- Proporciona información sobre el estado del sistema
- Reduce la ansiedad y confusión del usuario

**Tipos de feedback:**
- **Visual**: Cambios de color, iconos, animaciones
- **Táctil**: Vibración (haptic feedback)
- **Auditivo**: Sonidos y notificaciones
- **Inmediato**: Respuesta instantánea a la interacción

**Ejemplo de feedback visual:**
```jsx
const [isPressed, setIsPressed] = useState(false);

<TouchableOpacity
  style={[
    styles.button,
    isPressed && styles.buttonPressed
  ]}
  onPressIn={() => setIsPressed(true)}
  onPressOut={() => setIsPressed(false)}
  onPress={handlePress}
>
  <Text style={styles.buttonText}>Presionar</Text>
</TouchableOpacity>

const styles = StyleSheet.create({
  button: {
    backgroundColor: colors.primary,
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  buttonPressed: {
    backgroundColor: colors.primary,
    opacity: 0.8,
    transform: [{ scale: 0.98 }],
    shadowOpacity: 0.2,
  }
});
```

---

## 🔍 Heurísticas de Nielsen

### **¿Qué son las Heurísticas de Nielsen?**
Son 10 principios de usabilidad desarrollados por Jakob Nielsen que sirven como guía para evaluar y mejorar la usabilidad de interfaces.

### **1. Visibilidad del Estado del Sistema**
**Principio**: El sistema siempre debe informar al usuario sobre lo que está pasando.

**Ejemplo en React Native:**
```jsx
const [isLoading, setIsLoading] = useState(false);
const [progress, setProgress] = useState(0);

// Mostrar estado de carga
{isLoading && (
  <View style={styles.loadingContainer}>
    <ActivityIndicator size="large" color={colors.primary} />
    <Text style={styles.loadingText}>Cargando...</Text>
    <ProgressBar progress={progress} color={colors.primary} />
  </View>
)}

// Mostrar estado de éxito/error
{!isLoading && (
  <View style={styles.statusContainer}>
    {success ? (
      <Text style={[styles.statusText, { color: colors.success }]}>
        ✅ Operación completada
      </Text>
    ) : (
      <Text style={[styles.statusText, { color: colors.error }]}>
        ❌ Error en la operación
      </Text>
    )}
  </View>
)}
```

### **2. Correspondencia entre el Sistema y el Mundo Real**
**Principio**: El sistema debe hablar el lenguaje del usuario, usando palabras, frases y conceptos familiares.

**Ejemplo:**
```jsx
// ❌ Términos técnicos confusos
<Text>Error 404: Resource not found</Text>
<Text>Invalid input parameters</Text>

// ✅ Términos familiares para el usuario
<Text>No se encontró la página solicitada</Text>
<Text>Por favor, verifica la información ingresada</Text>

// Iconos que representan acciones familiares
<View style={styles.actionButtons}>
  <TouchableOpacity style={styles.actionButton}>
    <Icon name="trash" size={24} color={colors.error} />
    <Text>Eliminar</Text>
  </TouchableOpacity>
  
  <TouchableOpacity style={styles.actionButton}>
    <Icon name="edit" size={24} color={colors.primary} />
    <Text>Editar</Text>
  </TouchableOpacity>
  
  <TouchableOpacity style={styles.actionButton}>
    <Icon name="share" size={24} color={colors.secondary} />
    <Text>Compartir</Text>
  </TouchableOpacity>
</View>
```

### **3. Control y Libertad del Usuario**
**Principio**: Los usuarios necesitan una "salida de emergencia" claramente marcada para salir de estados no deseados.

**Ejemplo:**
```jsx
const [showModal, setShowModal] = useState(false);

// Modal con salida clara
<Modal visible={showModal} animationType="slide">
  <View style={styles.modalContainer}>
    <View style={styles.modalHeader}>
      <Text style={styles.modalTitle}>Confirmar Acción</Text>
      <TouchableOpacity 
        onPress={() => setShowModal(false)}
        style={styles.closeButton}
      >
        <Icon name="close" size={24} color={colors.text} />
      </TouchableOpacity>
    </View>
    
    <View style={styles.modalContent}>
      <Text>¿Estás seguro de que quieres continuar?</Text>
    </View>
    
    <View style={styles.modalActions}>
      <TouchableOpacity 
        style={[styles.button, styles.buttonSecondary]}
        onPress={() => setShowModal(false)}
      >
        <Text>Cancelar</Text>
      </TouchableOpacity>
      
      <TouchableOpacity 
        style={[styles.button, styles.buttonPrimary]}
        onPress={handleConfirm}
      >
        <Text>Confirmar</Text>
      </TouchableOpacity>
    </View>
  </View>
</Modal>
```

### **4. Consistencia y Estándares**
**Principio**: Los usuarios no deben preguntarse si diferentes palabras, situaciones o acciones significan lo mismo.

**Ejemplo de consistencia en navegación:**
```jsx
// Navegación consistente en toda la app
const NavigationTabs = () => (
  <Tab.Navigator
    screenOptions={{
      tabBarActiveTintColor: colors.primary,
      tabBarInactiveTintColor: colors.textSecondary,
      tabBarStyle: styles.tabBar,
      headerStyle: styles.header,
      headerTintColor: colors.text,
    }}
  >
    <Tab.Screen 
      name="Home" 
      component={HomeScreen}
      options={{
        tabBarIcon: ({ color, size }) => (
          <Icon name="home" size={size} color={color} />
        ),
        title: 'Inicio'
      }}
    />
    
    <Tab.Screen 
      name="Profile" 
      component={ProfileScreen}
      options={{
        tabBarIcon: ({ color, size }) => (
          <Icon name="user" size={size} color={color} />
        ),
        title: 'Perfil'
      }}
    />
  </Tab.Navigator>
);
```

### **5. Prevención de Errores**
**Principio**: Es mejor prevenir que ocurran errores que mostrar mensajes de error.

**Ejemplo de validación preventiva:**
```jsx
const [email, setEmail] = useState('');
const [isValidEmail, setIsValidEmail] = useState(true);

const validateEmail = (text) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  const isValid = emailRegex.test(text);
  setIsValidEmail(isValid);
  return isValid;
};

const handleEmailChange = (text) => {
  setEmail(text);
  if (text.length > 0) {
    validateEmail(text);
  } else {
    setIsValidEmail(true); // No mostrar error si está vacío
  }
};

// Campo con validación en tiempo real
<View style={styles.inputContainer}>
  <Text style={styles.label}>Email</Text>
  <TextInput
    style={[
      styles.input,
      !isValidEmail && styles.inputError
    ]}
    value={email}
    onChangeText={handleEmailChange}
    placeholder="tu@email.com"
    keyboardType="email-address"
    autoCapitalize="none"
  />
  {!isValidEmail && (
    <Text style={styles.errorText}>
      Por favor, ingresa un email válido
    </Text>
  )}
</View>
```

---

## 🧠 Psicología del Usuario

### **Principios Psicológicos en el Diseño**

#### **1. Ley de Hick-Hyman (Tiempo de Decisión)**
**Principio**: El tiempo que toma tomar una decisión aumenta con el número de opciones disponibles.

**Aplicación en React Native:**
```jsx
// ❌ Demasiadas opciones - Confunde al usuario
<View style={styles.menuContainer}>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Opción 1</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Opción 2</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Opción 3</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Opción 4</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Opción 5</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Opción 6</Text>
  </TouchableOpacity>
</View>

// ✅ Opciones agrupadas y organizadas
<View style={styles.menuContainer}>
  <Text style={styles.menuSection}>Acciones Principales</Text>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Crear Nuevo</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Buscar</Text>
  </TouchableOpacity>
  
  <Text style={styles.menuSection}>Configuración</Text>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Perfil</Text>
  </TouchableOpacity>
  <TouchableOpacity style={styles.menuItem}>
    <Text>Preferencias</Text>
  </TouchableOpacity>
</View>
```

#### **2. Ley de Fitts (Tiempo de Movimiento)**
**Principio**: El tiempo para alcanzar un objetivo es proporcional a la distancia y tamaño del objetivo.

**Aplicación:**
```jsx
// Botones grandes y fáciles de alcanzar
const styles = StyleSheet.create({
  // Botón principal - Grande y accesible
  primaryButton: {
    backgroundColor: colors.primary,
    paddingVertical: 16,
    paddingHorizontal: 32,
    borderRadius: 12,
    minHeight: 56, // Tamaño mínimo para facilitar el toque
    alignItems: 'center',
    justifyContent: 'center',
    marginVertical: 8,
  },
  
  // Botones de acción flotantes - Fácil acceso
  fabButton: {
    position: 'absolute',
    bottom: 24,
    right: 24,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: colors.primary,
    alignItems: 'center',
    justifyContent: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.3,
    shadowRadius: 8,
    elevation: 8,
  }
});
```

#### **3. Principio de Proximidad**
**Principio**: Los elementos relacionados deben estar agrupados visualmente.

**Ejemplo:**
```jsx
// Agrupación lógica de elementos relacionados
<View style={styles.formSection}>
  <Text style={styles.sectionTitle}>Información Personal</Text>
  
  <View style={styles.inputGroup}>
    <Text style={styles.label}>Nombre</Text>
    <TextInput style={styles.input} placeholder="Tu nombre" />
  </View>
  
  <View style={styles.inputGroup}>
    <Text style={styles.label}>Apellido</Text>
    <TextInput style={styles.input} placeholder="Tu apellido" />
  </View>
  
  <View style={styles.inputGroup}>
    <Text style={styles.label}>Email</Text>
    <TextInput style={styles.input} placeholder="tu@email.com" />
  </View>
</View>

<View style={styles.formSection}>
  <Text style={styles.sectionTitle}>Dirección</Text>
  
  <View style={styles.inputGroup}>
    <Text style={styles.label}>Calle</Text>
    <TextInput style={styles.input} placeholder="Nombre de la calle" />
  </View>
  
  <View style={styles.inputGroup}>
    <Text style={styles.label}>Ciudad</Text>
    <TextInput style={styles.input} placeholder="Nombre de la ciudad" />
  </View>
</View>
```

---

## 🔬 Investigación de Usuarios

### **Métodos de Investigación**

#### **1. Entrevistas con Usuarios**
**Objetivo**: Entender necesidades, motivaciones y comportamientos.

**Preguntas clave:**
- ¿Qué problemas intentas resolver con esta app?
- ¿Cómo usas actualmente aplicaciones similares?
- ¿Qué te frustra de las apps existentes?
- ¿Qué funcionalidades consideras esenciales?

#### **2. Observación de Usuarios**
**Objetivo**: Ver cómo los usuarios interactúan realmente con la aplicación.

**Aspectos a observar:**
- Tiempo para completar tareas
- Errores y confusión
- Comportamientos inesperados
- Expresiones faciales y lenguaje corporal

#### **3. Encuestas y Cuestionarios**
**Objetivo**: Recopilar datos cuantitativos de una muestra más grande.

**Tipos de preguntas:**
- Escalas de satisfacción (1-5)
- Preguntas de opción múltiple
- Preguntas abiertas para feedback cualitativo

### **Implementación en React Native**

#### **Sistema de Analytics para Investigación**
```jsx
import analytics from '@react-native-firebase/analytics';

// Tracking de eventos de usuario
const trackUserAction = async (action, screen, details = {}) => {
  try {
    await analytics.logEvent('user_action', {
      action,
      screen,
      timestamp: new Date().toISOString(),
      ...details
    });
  } catch (error) {
    console.log('Error tracking event:', error);
  }
};

// Tracking de tiempo en pantalla
const trackScreenTime = async (screenName, timeSpent) => {
  try {
    await analytics.logEvent('screen_time', {
      screen: screenName,
      time_spent: timeSpent,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    console.log('Error tracking screen time:', error);
  }
};

// Uso en componentes
useEffect(() => {
  const startTime = Date.now();
  
  return () => {
    const timeSpent = Date.now() - startTime;
    trackScreenTime('HomeScreen', timeSpent);
  };
}, []);

// Tracking de interacciones
<TouchableOpacity
  onPress={() => {
    trackUserAction('button_press', 'HomeScreen', { button: 'create_post' });
    handleCreatePost();
  }}
>
  <Text>Crear Post</Text>
</TouchableOpacity>
```

---

## ✏️ Wireframing y Flujos de Usuario

### **¿Qué son los Wireframes?**
Los wireframes son representaciones visuales simples de la estructura y funcionalidad de una interfaz, sin elementos de diseño visual.

### **Tipos de Wireframes**

#### **1. Wireframes de Baja Fidelidad**
- Bocetos a mano o digitales simples
- Enfoque en estructura y flujo
- Rápido de crear y modificar

#### **2. Wireframes de Media Fidelidad**
- Más detallados, creados con herramientas digitales
- Incluyen elementos de navegación y contenido
- Balance entre velocidad y detalle

#### **3. Wireframes de Alta Fidelidad**
- Muy detallados, casi prototipos
- Incluyen tipografía, colores y espaciado
- Requieren más tiempo pero son más claros

### **Creación de Wireframes en React Native**

#### **Componente de Wireframe Básico**
```jsx
const WireframeBox = ({ width, height, label, style }) => (
  <View style={[styles.wireframeBox, { width, height }, style]}>
    <Text style={styles.wireframeLabel}>{label}</Text>
  </View>
);

const styles = StyleSheet.create({
  wireframeBox: {
    backgroundColor: '#E0E0E0',
    borderWidth: 1,
    borderColor: '#999',
    borderRadius: 4,
    alignItems: 'center',
    justifyContent: 'center',
    margin: 4,
  },
  wireframeLabel: {
    color: '#666',
    fontSize: 12,
    textAlign: 'center',
  }
});

// Uso para crear wireframes rápidos
const HomeScreenWireframe = () => (
  <View style={styles.container}>
    {/* Header */}
    <WireframeBox width="100%" height={60} label="Header" />
    
    {/* Search Bar */}
    <WireframeBox width="90%" height={50} label="Search Bar" />
    
    {/* Content Grid */}
    <View style={styles.contentGrid}>
      <WireframeBox width="48%" height={120} label="Card 1" />
      <WireframeBox width="48%" height={120} label="Card 2" />
      <WireframeBox width="48%" height={120} label="Card 3" />
      <WireframeBox width="48%" height={120} label="Card 4" />
    </View>
    
    {/* Bottom Navigation */}
    <WireframeBox width="100%" height={80} label="Bottom Nav" />
  </View>
);
```

### **Flujos de Usuario**

#### **¿Qué son los Flujos de Usuario?**
Los flujos de usuario son diagramas que muestran cómo un usuario navega a través de una aplicación para completar una tarea específica.

#### **Ejemplo de Flujo de Registro**
```jsx
// Componente para visualizar flujos de usuario
const UserFlowStep = ({ step, title, description, isActive, isCompleted }) => (
  <View style={styles.flowStep}>
    <View style={[
      styles.stepCircle,
      isActive && styles.stepActive,
      isCompleted && styles.stepCompleted
    ]}>
      <Text style={styles.stepNumber}>{step}</Text>
    </View>
    
    <View style={styles.stepContent}>
      <Text style={styles.stepTitle}>{title}</Text>
      <Text style={styles.stepDescription}>{description}</Text>
    </View>
    
    {!isCompleted && <View style={styles.stepConnector} />}
  </View>
);

const RegistrationFlow = () => {
  const [currentStep, setCurrentStep] = useState(1);
  
  return (
    <View style={styles.flowContainer}>
      <UserFlowStep
        step={1}
        title="Información Básica"
        description="Nombre, email y contraseña"
        isActive={currentStep === 1}
        isCompleted={currentStep > 1}
      />
      
      <UserFlowStep
        step={2}
        title="Verificación"
        description="Confirmar email y teléfono"
        isActive={currentStep === 2}
        isCompleted={currentStep > 2}
      />
      
      <UserFlowStep
        step={3}
        title="Perfil"
        description="Foto y preferencias"
        isActive={currentStep === 3}
        isCompleted={currentStep > 3}
      />
      
      <UserFlowStep
        step={4}
        title="Completado"
        description="¡Bienvenido a la app!"
        isActive={currentStep === 4}
        isCompleted={currentStep === 4}
      />
    </View>
  );
};
```

---

## 🧪 Testing de Usabilidad

### **Métodos de Testing**

#### **1. Testing de Usabilidad Moderado**
- **Facilitador**: Guía al usuario a través de tareas
- **Observación**: Ve y escucha las reacciones del usuario
- **Feedback**: Pregunta sobre la experiencia

#### **2. Testing de Usabilidad No Moderado**
- **Usuarios**: Completan tareas de forma independiente
- **Herramientas**: Software de grabación y análisis
- **Escalabilidad**: Puede incluir muchos usuarios

#### **3. A/B Testing**
- **Comparación**: Dos versiones de la misma funcionalidad
- **Métricas**: Medición de rendimiento objetivo
- **Decisiones**: Basadas en datos, no en opiniones

### **Implementación de Testing en React Native**

#### **Sistema de Testing de Usabilidad**
```jsx
import { InteractionManager } from 'react-native';

// Hook para tracking de interacciones
const useUsabilityTracking = (screenName) => {
  const [interactions, setInteractions] = useState([]);
  const [startTime, setStartTime] = useState(Date.now());
  
  const trackInteraction = (action, element, details = {}) => {
    const interaction = {
      action,
      element,
      timestamp: Date.now(),
      timeFromStart: Date.now() - startTime,
      ...details
    };
    
    setInteractions(prev => [...prev, interaction]);
    
    // Enviar a analytics
    trackUserAction(action, screenName, details);
  };
  
  const getSessionMetrics = () => {
    const totalTime = Date.now() - startTime;
    const interactionCount = interactions.length;
    
    return {
      totalTime,
      interactionCount,
      interactions
    };
  };
  
  return {
    trackInteraction,
    getSessionMetrics,
    interactions
  };
};

// Uso en componentes
const HomeScreen = () => {
  const { trackInteraction, getSessionMetrics } = useUsabilityTracking('HomeScreen');
  
  const handleButtonPress = (buttonName) => {
    trackInteraction('button_press', buttonName, {
      button_type: 'primary',
      screen_position: 'center'
    });
    
    // Lógica del botón
  };
  
  const handleSwipe = (direction) => {
    trackInteraction('swipe_gesture', 'content_area', {
      direction,
      gesture_type: 'swipe'
    });
  };
  
  // Enviar métricas al finalizar la sesión
  useEffect(() => {
    return () => {
      const metrics = getSessionMetrics();
      console.log('Session Metrics:', metrics);
    };
  }, []);
  
  return (
    <View style={styles.container}>
      <TouchableOpacity
        onPress={() => handleButtonPress('create_post')}
        style={styles.button}
      >
        <Text>Crear Post</Text>
      </TouchableOpacity>
    </View>
  );
};
```

---

## 📱 Ejercicios Prácticos

### **Ejercicio 1: Análisis de Usabilidad**
**Objetivo**: Analizar una aplicación existente usando las heurísticas de Nielsen.

**Pasos:**
1. Descarga una aplicación móvil popular
2. Navega por todas las pantallas principales
3. Identifica violaciones de las heurísticas de Nielsen
4. Documenta cada problema encontrado
5. Propón soluciones específicas

**Entregable**: Reporte de análisis con al menos 5 problemas identificados y sus soluciones.

### **Ejercicio 2: Creación de Wireframes**
**Objetivo**: Crear wireframes para una aplicación de gestión de tareas.

**Requisitos:**
- Pantalla de lista de tareas
- Pantalla de creación de tarea
- Pantalla de detalle de tarea
- Pantalla de configuración

**Entregable**: Wireframes digitales o bocetos a mano escaneados.

### **Ejercicio 3: Flujo de Usuario**
**Objetivo**: Diseñar el flujo completo para crear y completar una tarea.

**Pasos:**
1. Identifica todos los pasos necesarios
2. Crea un diagrama de flujo
3. Considera casos de error y excepciones
4. Optimiza el flujo para minimizar pasos

**Entregable**: Diagrama de flujo con máximo 6 pasos principales.

---

## 🎯 Resumen de la Clase

### **✅ Lo que has aprendido:**
1. **Fundamentos de UI-UX** - Diferencias entre interfaz y experiencia
2. **Principios de diseño** - Simplicidad, consistencia, jerarquía, feedback
3. **Heurísticas de Nielsen** - 10 principios para evaluar usabilidad
4. **Psicología del usuario** - Leyes de Hick-Hyman, Fitts y proximidad
5. **Investigación de usuarios** - Métodos para entender necesidades
6. **Wireframing** - Creación de prototipos de baja fidelidad
7. **Flujos de usuario** - Diseño de experiencias completas
8. **Testing de usabilidad** - Evaluación de la experiencia del usuario

### **🚀 Próximos pasos:**
- **Práctica**: Completa los ejercicios de esta clase
- **Investigación**: Analiza aplicaciones que uses regularmente
- **Documentación**: Crea tu primer wireframe
- **Siguiente clase**: Diseño Visual y Branding

---

## 🔗 Recursos Adicionales

### **📚 Lectura Recomendada**
- **"Don't Make Me Think"** - Steve Krug
- **"The Design of Everyday Things"** - Don Norman
- **"Universal Principles of Design"** - William Lidwell

### **🌐 Recursos Online**
- **Nielsen Norman Group**: [nngroup.com](https://www.nngroup.com/)
- **UX Planet**: [uxplanet.org](https://uxplanet.org/)
- **Smashing Magazine**: [smashingmagazine.com](https://www.smashingmagazine.com/)

### **🛠️ Herramientas**
- **Figma**: [figma.com](https://www.figma.com/)
- **Balsamiq**: [balsamiq.com](https://balsamiq.com/)
- **InVision**: [invisionapp.com](https://www.invisionapp.com/)

---

**🎨 ¡Felicidades! Has completado la primera clase de fundamentos de UI-UX.**

Ahora tienes una base sólida en los principios de diseño de interfaces y experiencia de usuario. En la siguiente clase aprenderemos sobre **Diseño Visual y Branding**, donde aplicaremos estos fundamentos para crear interfaces visualmente atractivas y efectivas.

**¡Continúa con la siguiente clase para expandir tus habilidades de diseño!** 🚀

### **Clase Siguiente**: [Clase 2: Diseño Visual y Branding](clase_2_diseno_visual_branding.md)

