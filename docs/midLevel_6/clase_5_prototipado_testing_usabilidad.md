# 🧪 Clase 5: Prototipado y Testing de Usabilidad

## 📍 Navegación
- **📚 Módulo**: [Módulo 9: UI-UX y Diseño de Interfaces](README.md)
- **⬅️ Anterior**: [Clase 4: Accesibilidad e Inclusión](clase_4_accesibilidad_inclusion.md)
- **➡️ Siguiente**: [Módulo 10: Patrones de Diseño](../senior_1/README.md)

---

## 🎯 Objetivos de la Clase

### **Al Finalizar esta Clase, Serás Capaz de:**
1. **Comprender** los tipos y fidelidades de prototipos
2. **Crear** prototipos interactivos con herramientas profesionales
3. **Diseñar** user flows efectivos y completos
4. **Implementar** testing de usabilidad sistemático
5. **Analizar** resultados de testing y métricas UX
6. **Aplicar** metodologías de prototipado en React Native

---

## 🎨 ¿Qué es el Prototipado?

### **¿Por qué es Importante el Prototipado?**

El prototipado es fundamental para:
- **Validar** ideas antes del desarrollo
- **Comunicar** conceptos de diseño
- **Reducir** costos de desarrollo
- **Iterar** rápidamente sobre soluciones
- **Testear** usabilidad tempranamente

### **Tipos de Prototipos**

#### **Por Fidelidad**

##### **Baja Fidelidad**
- **Wireframes** básicos
- **Bocetos** en papel
- **Contenido** placeholder
- **Navegación** básica

##### **Media Fidelidad**
- **Contenido** más realista
- **Navegación** funcional
- **Algunos** elementos visuales
- **Interacciones** básicas

##### **Alta Fidelidad**
- **Diseño visual** completo
- **Contenido** real o muy realista
- **Interacciones** avanzadas
- **Microinteracciones** incluidas

#### **Por Funcionalidad**

##### **Prototipos Estáticos**
- **Imágenes** de pantallas
- **Sin interacción** funcional
- **Para** comunicación visual

##### **Prototipos Interactivos**
- **Navegación** funcional
- **Interacciones** básicas
- **Simulación** de funcionalidades

##### **Prototipos Funcionales**
- **Código** real funcional
- **Datos** conectados
- **Funcionalidades** completas

---

## 🛠️ Herramientas de Prototipado

### **Herramientas Populares**

#### **Figma**
- **Colaboración** en tiempo real
- **Componentes** reutilizables
- **Prototipado** interactivo
- **Handoff** a desarrollo

#### **Adobe XD**
- **Integración** con Creative Suite
- **Prototipado** avanzado
- **Animaciones** y transiciones
- **Plugins** extensivos

#### **Sketch + InVision**
- **Diseño** vectorial profesional
- **Prototipado** con InVision
- **Comentarios** colaborativos
- **Especificaciones** de desarrollo

#### **Framer**
- **Prototipado** de alta fidelidad
- **Código** real React
- **Animaciones** avanzadas
- **Componentes** interactivos

### **Implementación en React Native**

#### **Prototipo de Baja Fidelidad**
```jsx
// Componente de wireframe básico
const WireframeBox = ({ 
  width = 100, 
  height = 50, 
  label,
  style,
  onPress 
}) => {
  const wireframeStyle = {
    width,
    height,
    borderWidth: 2,
    borderColor: '#CCCCCC',
    borderStyle: 'dashed',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5F5F5',
    ...style
  };
  
  const Wrapper = onPress ? TouchableOpacity : View;
  
  return (
    <Wrapper style={wireframeStyle} onPress={onPress}>
      {label && (
        <Text style={styles.wireframeLabel}>{label}</Text>
      )}
    </Wrapper>
  );
};

// Ejemplo de wireframe de pantalla
const LoginWireframe = () => {
  return (
    <View style={styles.wireframeContainer}>
      <WireframeBox 
        width={200} 
        height={80} 
        label="Logo" 
        style={{ marginBottom: 40 }}
      />
      
      <WireframeBox 
        width={300} 
        height={40} 
        label="Email Input" 
        style={{ marginBottom: 16 }}
      />
      
      <WireframeBox 
        width={300} 
        height={40} 
        label="Password Input" 
        style={{ marginBottom: 24 }}
      />
      
      <WireframeBox 
        width={300} 
        height={50} 
        label="Login Button" 
        onPress={() => console.log('Login pressed')}
        style={{ backgroundColor: '#E3F2FD' }}
      />
      
      <WireframeBox 
        width={150} 
        height={30} 
        label="Forgot Password" 
        style={{ marginTop: 16 }}
      />
    </View>
  );
};
```

#### **Prototipo de Media Fidelidad**
```jsx
// Componente de prototipo con más detalle
const PrototypeScreen = ({ title, children, showHeader = true }) => {
  return (
    <View style={styles.prototypeScreen}>
      {showHeader && (
        <View style={styles.prototypeHeader}>
          <Text style={styles.prototypeTitle}>{title}</Text>
        </View>
      )}
      
      <View style={styles.prototypeContent}>
        {children}
      </View>
    </View>
  );
};

const LoginPrototype = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  return (
    <PrototypeScreen title="Iniciar Sesión">
      <View style={styles.logoContainer}>
        <View style={styles.logoPlaceholder}>
          <Text style={styles.logoText}>LOGO</Text>
        </View>
      </View>
      
      <View style={styles.formContainer}>
        <TextInput
          style={styles.prototypeInput}
          placeholder="Correo electrónico"
          value={email}
          onChangeText={setEmail}
          keyboardType="email-address"
        />
        
        <TextInput
          style={styles.prototypeInput}
          placeholder="Contraseña"
          value={password}
          onChangeText={setPassword}
          secureTextEntry
        />
        
        <TouchableOpacity 
          style={styles.prototypeButton}
          onPress={() => Alert.alert('Login', 'Funcionalidad de login')}
        >
          <Text style={styles.prototypeButtonText}>Iniciar Sesión</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.forgotPassword}>
          <Text style={styles.forgotPasswordText}>¿Olvidaste tu contraseña?</Text>
        </TouchableOpacity>
      </View>
    </PrototypeScreen>
  );
};
```

---

## 🗺️ User Flows y Journey Mapping

### **¿Qué son los User Flows?**

Los user flows son diagramas que muestran el camino que sigue un usuario para completar una tarea específica en la aplicación.

### **¿Por qué son Importantes?**

Los user flows son importantes para:
- **Identificar** puntos de fricción
- **Optimizar** la navegación
- **Reducir** pasos innecesarios
- **Mejorar** la conversión
- **Documentar** la experiencia

### **Tipos de User Flows**

#### **Task Flow**
- **Flujo** para una tarea específica
- **Enfoque** en la funcionalidad
- **Pasos** lineales y directos

#### **User Flow**
- **Flujo** considerando diferentes tipos de usuarios
- **Múltiples** caminos posibles
- **Decisiones** del usuario

#### **Wireflow**
- **Combinación** de wireframes y flujos
- **Representación visual** de pantallas
- **Conexiones** entre interfaces

### **Implementación en React Native**

#### **Componente de User Flow**
```jsx
// Componente para mapear user flows
const UserFlowStep = ({ 
  stepNumber, 
  title, 
  description, 
  screen,
  actions = [],
  isActive = false,
  onPress 
}) => {
  return (
    <TouchableOpacity 
      style={[
        styles.flowStep,
        isActive && styles.flowStepActive
      ]}
      onPress={onPress}
    >
      <View style={styles.stepNumber}>
        <Text style={styles.stepNumberText}>{stepNumber}</Text>
      </View>
      
      <View style={styles.stepContent}>
        <Text style={styles.stepTitle}>{title}</Text>
        <Text style={styles.stepDescription}>{description}</Text>
        
        {screen && (
          <View style={styles.stepScreen}>
            {screen}
          </View>
        )}
        
        {actions.length > 0 && (
          <View style={styles.stepActions}>
            {actions.map((action, index) => (
              <TouchableOpacity 
                key={index}
                style={styles.stepAction}
                onPress={action.onPress}
              >
                <Text style={styles.stepActionText}>{action.label}</Text>
              </TouchableOpacity>
            ))}
          </View>
        )}
      </View>
    </TouchableOpacity>
  );
};

// Ejemplo de user flow completo
const LoginUserFlow = () => {
  const [currentStep, setCurrentStep] = useState(1);
  
  const flowSteps = [
    {
      id: 1,
      title: 'Pantalla de Bienvenida',
      description: 'Usuario ve la pantalla inicial de la app',
      screen: <WireframeBox width={150} height={200} label="Welcome Screen" />,
      actions: [
        { label: 'Continuar', onPress: () => setCurrentStep(2) }
      ]
    },
    {
      id: 2,
      title: 'Formulario de Login',
      description: 'Usuario ingresa credenciales',
      screen: <WireframeBox width={150} height={200} label="Login Form" />,
      actions: [
        { label: 'Login', onPress: () => setCurrentStep(3) },
        { label: 'Registro', onPress: () => setCurrentStep(4) }
      ]
    },
    {
      id: 3,
      title: 'Dashboard Principal',
      description: 'Usuario accede a la pantalla principal',
      screen: <WireframeBox width={150} height={200} label="Dashboard" />,
      actions: []
    },
    {
      id: 4,
      title: 'Formulario de Registro',
      description: 'Usuario crea nueva cuenta',
      screen: <WireframeBox width={150} height={200} label="Register Form" />,
      actions: [
        { label: 'Crear Cuenta', onPress: () => setCurrentStep(3) }
      ]
    }
  ];
  
  return (
    <ScrollView style={styles.userFlowContainer}>
      <Text style={styles.flowTitle}>User Flow: Proceso de Login</Text>
      
      {flowSteps.map((step) => (
        <UserFlowStep
          key={step.id}
          stepNumber={step.id}
          title={step.title}
          description={step.description}
          screen={step.screen}
          actions={step.actions}
          isActive={currentStep === step.id}
          onPress={() => setCurrentStep(step.id)}
        />
      ))}
    </ScrollView>
  );
};
```

---

## 🧪 Testing de Usabilidad

### **¿Qué es el Testing de Usabilidad?**

El testing de usabilidad es el proceso de evaluar una aplicación probándola con usuarios reales para identificar problemas de usabilidad y oportunidades de mejora.

### **¿Por qué es Importante?**

El testing de usabilidad es importante para:
- **Identificar** problemas de usabilidad
- **Validar** decisiones de diseño
- **Mejorar** la experiencia del usuario
- **Reducir** costos de desarrollo
- **Aumentar** la satisfacción del usuario

### **Tipos de Testing de Usabilidad**

#### **Testing Moderado**
- **Facilitador** presente durante la sesión
- **Interacción** directa con el usuario
- **Clarificaciones** en tiempo real
- **Observación** detallada del comportamiento

#### **Testing No Moderado**
- **Usuario** completa tareas independientemente
- **Grabación** de pantalla y audio
- **Análisis** posterior de resultados
- **Escalabilidad** para más usuarios

#### **A/B Testing**
- **Comparación** entre dos versiones
- **Métricas** cuantitativas
- **Decisiones** basadas en datos
- **Optimización** continua

### **Implementación en React Native**

#### **Hook para Testing de Usabilidad**
```jsx
// Hook para tracking de usabilidad
const useUsabilityTracking = (screenName) => {
  const [sessionData, setSessionData] = useState({
    startTime: Date.now(),
    interactions: [],
    errors: [],
    completedTasks: [],
  });
  
  const trackInteraction = useCallback((interactionType, elementId, details = {}) => {
    const interaction = {
      timestamp: Date.now(),
      type: interactionType,
      elementId,
      screenName,
      ...details
    };
    
    setSessionData(prev => ({
      ...prev,
      interactions: [...prev.interactions, interaction]
    }));
    
    // Enviar a analytics (Firebase, etc.)
    analytics().logEvent('user_interaction', {
      screen_name: screenName,
      interaction_type: interactionType,
      element_id: elementId,
      ...details
    });
  }, [screenName]);
  
  const trackError = useCallback((errorType, errorMessage, context = {}) => {
    const error = {
      timestamp: Date.now(),
      type: errorType,
      message: errorMessage,
      screenName,
      ...context
    };
    
    setSessionData(prev => ({
      ...prev,
      errors: [...prev.errors, error]
    }));
    
    // Enviar error a analytics
    analytics().logEvent('usability_error', {
      screen_name: screenName,
      error_type: errorType,
      error_message: errorMessage,
      ...context
    });
  }, [screenName]);
  
  const trackTaskCompletion = useCallback((taskId, success, timeToComplete) => {
    const task = {
      taskId,
      success,
      timeToComplete,
      timestamp: Date.now(),
      screenName
    };
    
    setSessionData(prev => ({
      ...prev,
      completedTasks: [...prev.completedTasks, task]
    }));
    
    // Enviar completion a analytics
    analytics().logEvent('task_completion', {
      task_id: taskId,
      success,
      time_to_complete: timeToComplete,
      screen_name: screenName
    });
  }, [screenName]);
  
  const getSessionMetrics = () => {
    const sessionDuration = Date.now() - sessionData.startTime;
    const interactionCount = sessionData.interactions.length;
    const errorCount = sessionData.errors.length;
    const completedTasksCount = sessionData.completedTasks.filter(task => task.success).length;
    const totalTasks = sessionData.completedTasks.length;
    
    return {
      sessionDuration,
      interactionCount,
      errorCount,
      completedTasksCount,
      totalTasks,
      successRate: totalTasks > 0 ? (completedTasksCount / totalTasks) * 100 : 0,
      averageTimePerInteraction: interactionCount > 0 ? sessionDuration / interactionCount : 0
    };
  };
  
  return {
    trackInteraction,
    trackError,
    trackTaskCompletion,
    getSessionMetrics,
    sessionData
  };
};
```

#### **Componente de Testing A/B**
```jsx
// Componente para A/B testing
const ABTestComponent = ({ 
  testId,
  variantA,
  variantB,
  onVariantShown,
  children 
}) => {
  const [variant, setVariant] = useState(null);
  
  useEffect(() => {
    // Determinar qué variante mostrar (50/50 split)
    const userVariant = Math.random() < 0.5 ? 'A' : 'B';
    setVariant(userVariant);
    
    // Trackear qué variante se mostró
    if (onVariantShown) {
      onVariantShown(userVariant);
    }
    
    // Enviar a analytics
    analytics().logEvent('ab_test_variant_shown', {
      test_id: testId,
      variant: userVariant,
      user_id: getUserId(),
      timestamp: Date.now()
    });
  }, [testId, onVariantShown]);
  
  const trackConversion = (conversionType) => {
    analytics().logEvent('ab_test_conversion', {
      test_id: testId,
      variant,
      conversion_type: conversionType,
      user_id: getUserId(),
      timestamp: Date.now()
    });
  };
  
  if (!variant) {
    return <ActivityIndicator size="small" />;
  }
  
  return (
    <View style={styles.abTestContainer}>
      {variant === 'A' ? variantA : variantB}
      
      {/* Componente hijo puede usar trackConversion */}
      {children && React.cloneElement(children, { trackConversion })}
    </View>
  );
};

// Ejemplo de A/B testing
const ButtonABTest = () => {
  return (
    <ABTestComponent
      testId="login_button_test"
      variantA={
        <TouchableOpacity style={styles.buttonVariantA}>
          <Text style={styles.buttonTextA}>Iniciar Sesión</Text>
        </TouchableOpacity>
      }
      variantB={
        <TouchableOpacity style={styles.buttonVariantB}>
          <Text style={styles.buttonTextB}>Entrar</Text>
        </TouchableOpacity>
      }
      onVariantShown={(variant) => {
        console.log(`Mostrando variante: ${variant}`);
      }}
    >
      <ConversionTracker />
    </ABTestComponent>
  );
};
```

---

## 📊 Métricas y Análisis

### **Métricas de Usabilidad**

#### **Métricas Cuantitativas**
- **Tiempo de tarea** - Tiempo para completar una tarea
- **Tasa de éxito** - Porcentaje de tareas completadas exitosamente
- **Tasa de error** - Número de errores por sesión
- **Eficiencia** - Tareas completadas por unidad de tiempo

#### **Métricas Cualitativas**
- **Satisfacción** del usuario
- **Facilidad** de uso percibida
- **Confianza** en la aplicación
- **Intención** de uso futuro

#### **Métricas de Engagement**
- **Tiempo** en la aplicación
- **Frecuencia** de uso
- **Retención** de usuarios
- **Conversión** de objetivos

### **Implementación en React Native**

#### **Sistema de Métricas Completo**
```jsx
// Hook para métricas de usabilidad
const useUsabilityMetrics = () => {
  const [metrics, setMetrics] = useState({
    sessionStart: Date.now(),
    screenViews: [],
    interactions: [],
    errors: [],
    tasks: []
  });
  
  const trackScreenView = useCallback((screenName) => {
    const screenView = {
      screenName,
      timestamp: Date.now(),
      duration: 0
    };
    
    setMetrics(prev => {
      // Calcular duración de la pantalla anterior
      const lastScreen = prev.screenViews[prev.screenViews.length - 1];
      if (lastScreen) {
        lastScreen.duration = Date.now() - lastScreen.timestamp;
      }
      
      return {
        ...prev,
        screenViews: [...prev.screenViews, screenView]
      };
    });
    
    // Enviar a analytics
    analytics().logEvent('screen_view', {
      screen_name: screenName,
      timestamp: Date.now()
    });
  }, []);
  
  const trackUserInteraction = useCallback((action, element, details = {}) => {
    const interaction = {
      action,
      element,
      timestamp: Date.now(),
      ...details
    };
    
    setMetrics(prev => ({
      ...prev,
      interactions: [...prev.interactions, interaction]
    }));
    
    // Enviar a analytics
    analytics().logEvent('user_interaction', {
      action,
      element,
      ...details
    });
  }, []);
  
  const trackTask = useCallback((taskId, status, duration) => {
    const task = {
      taskId,
      status, // 'completed', 'abandoned', 'failed'
      duration,
      timestamp: Date.now()
    };
    
    setMetrics(prev => ({
      ...prev,
      tasks: [...prev.tasks, task]
    }));
    
    // Enviar a analytics
    analytics().logEvent('task_completion', {
      task_id: taskId,
      status,
      duration
    });
  }, []);
  
  const getSessionMetrics = () => {
    const sessionDuration = Date.now() - metrics.sessionStart;
    const totalScreenViews = metrics.screenViews.length;
    const totalInteractions = metrics.interactions.length;
    const totalErrors = metrics.errors.length;
    const completedTasks = metrics.tasks.filter(task => task.status === 'completed').length;
    const totalTasks = metrics.tasks.length;
    
    return {
      sessionDuration,
      totalScreenViews,
      totalInteractions,
      totalErrors,
      completedTasks,
      totalTasks,
      successRate: totalTasks > 0 ? (completedTasks / totalTasks) * 100 : 0,
      averageScreenTime: totalScreenViews > 0 ? 
        metrics.screenViews.reduce((acc, screen) => acc + screen.duration, 0) / totalScreenViews : 0,
      interactionsPerMinute: sessionDuration > 0 ? 
        (totalInteractions / (sessionDuration / 60000)) : 0
    };
  };
  
  return {
    trackScreenView,
    trackUserInteraction,
    trackTask,
    getSessionMetrics,
    metrics
  };
};
```

---

## 📱 Ejercicios Prácticos

### **Ejercicio 1: Prototipo Completo**
**Objetivo**: Crear un prototipo completo de una funcionalidad específica.

**Requisitos:**
- Wireframes de baja fidelidad
- Prototipo de media fidelidad funcional
- User flow documentado
- Testing con al menos 3 usuarios

**Entregable**: Prototipo funcional con documentación de testing.

### **Ejercicio 2: A/B Testing Implementation**
**Objetivo**: Implementar un sistema de A/B testing para una funcionalidad.

**Requisitos:**
- Dos variantes de la misma funcionalidad
- Sistema de tracking de conversiones
- Análisis de resultados
- Recomendaciones basadas en datos

**Entregable**: Sistema de A/B testing con análisis de resultados.

### **Ejercicio 3: Métricas de Usabilidad**
**Objetivo**: Implementar un sistema completo de métricas de usabilidad.

**Requisitos:**
- Tracking de interacciones del usuario
- Métricas de tiempo y eficiencia
- Dashboard de métricas en tiempo real
- Reportes de usabilidad

**Entregable**: Sistema de métricas con dashboard y reportes.

---

## 🎯 Resumen de la Clase

### **✅ Lo que has aprendido:**
1. **Prototipado** - Tipos, fidelidades y herramientas
2. **User flows** - Mapeo de experiencias y journey mapping
3. **Testing de usabilidad** - Métodos moderados y no moderados
4. **A/B testing** - Implementación y análisis de resultados
5. **Métricas de usabilidad** - Tracking y análisis de comportamiento
6. **Implementación práctica** - Código React Native para testing

### **🚀 Próximos pasos:**
- **Práctica**: Completa los ejercicios de esta clase
- **Implementación**: Crea prototipos para tus proyectos
- **Testing**: Realiza testing de usabilidad con usuarios reales
- **Siguiente módulo**: Patrones de Diseño (Senior Level)

---

## 🔗 Recursos Adicionales

### **📚 Lectura Recomendada**
- **"Don't Make Me Think"** - Steve Krug
- **"Rocket Surgery Made Easy"** - Steve Krug
- **"Observing the User Experience"** - Kuniavsky, Stahl & Moed

### **🌐 Recursos Online**
- **Figma**: [figma.com](https://www.figma.com/)
- **UserTesting**: [usertesting.com](https://www.usertesting.com/)
- **Hotjar**: [hotjar.com](https://www.hotjar.com/)

### **🛠️ Herramientas**
- **Figma**: Prototipado y diseño
- **Adobe XD**: Prototipado interactivo
- **InVision**: Prototipado y colaboración
- **Framer**: Prototipos de alta fidelidad

---

**🧪 ¡Excelente! Has completado el módulo de UI-UX y Diseño de Interfaces.**

Ahora tienes un conocimiento completo y sólido sobre todos los aspectos fundamentales del diseño de interfaces para aplicaciones móviles. Has aprendido desde los fundamentos básicos hasta técnicas avanzadas de prototipado y testing.

**¡Estás listo para el siguiente nivel: Patrones de Diseño Avanzados!** 🚀

### **Siguiente Módulo**: [Módulo 10: Patrones de Diseño](../senior_1/README.md)

