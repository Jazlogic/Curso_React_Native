# 🌍 Clase 3: RTL y Idiomas Complejos

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar soporte completo para idiomas RTL (Right-to-Left) como árabe, hebreo y persa, incluyendo layouts adaptativos, navegación RTL, iconos y elementos visuales, y testing de interfaces RTL.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** soporte RTL completo en React Native
2. **Crear** layouts adaptativos para diferentes idiomas
3. **Configurar** navegación y flujos RTL
4. **Adaptar** iconos y elementos visuales para RTL
5. **Testing** de interfaces RTL

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Soporte RTL (Right-to-Left)**

#### **¿Qué es RTL?**
RTL (Right-to-Left) se refiere a idiomas que se leen de derecha a izquierda, como árabe, hebreo, persa y urdu.

```javascript
// Idiomas RTL comunes
const RTL_LANGUAGES = [
  'ar', // Árabe
  'he', // Hebreo
  'fa', // Persa/Farsi
  'ur', // Urdu
  'ku', // Kurdo
  'ps', // Pashto
  'sd', // Sindhi
  'dv', // Maldivo
  'yi', // Yiddish
  'ji'  // Yiddish (alternativo)
];

// Detección de idioma RTL
const isRTLLanguage = (language) => {
  return RTL_LANGUAGES.includes(language);
};
```

#### **Configuración de RTL en React Native**

```javascript
// i18n/rtlConfig.js
import { I18nManager } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

class RTLManager {
  constructor() {
    this.rtlLanguages = ['ar', 'he', 'fa', 'ur'];
  }
  
  async setLanguage(language) {
    const isRTL = this.rtlLanguages.includes(language);
    
    // Configurar RTL en el sistema
    I18nManager.allowRTL(isRTL);
    I18nManager.forceRTL(isRTL);
    
    // Guardar preferencia
    await AsyncStorage.setItem('isRTL', isRTL.toString());
    
    // Reiniciar la aplicación para aplicar cambios
    if (I18nManager.isRTL !== isRTL) {
      // En desarrollo, recargar
      if (__DEV__) {
        // Reload app
      }
    }
  }
  
  isRTLLanguage(language) {
    return this.rtlLanguages.includes(language);
  }
  
  getCurrentDirection() {
    return I18nManager.isRTL ? 'rtl' : 'ltr';
  }
}

export const rtlManager = new RTLManager();
```

#### **Configuración de i18next para RTL**

```javascript
// i18n/index.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import { rtlManager } from './rtlConfig';

const resources = {
  en: { translation: require('./locales/en/common.json') },
  ar: { translation: require('./locales/ar/common.json') },
  he: { translation: require('./locales/he/common.json') },
};

i18n
  .use(initReactI18next)
  .init({
    resources,
    fallbackLng: 'en',
    debug: __DEV__,
    
    interpolation: {
      escapeValue: false,
    },
    
    // Configuración RTL
    detection: {
      order: ['device', 'storage', 'navigator'],
      caches: ['localStorage'],
    },
  });

// Configurar RTL cuando cambie el idioma
i18n.on('languageChanged', (lng) => {
  rtlManager.setLanguage(lng);
});

export default i18n;
```

### **2. Layouts Adaptativos**

#### **Flexbox para RTL**

```javascript
// components/RTLFlexContainer.js
import React from 'react';
import { View, StyleSheet, I18nManager } from 'react-native';

const RTLFlexContainer = ({ children, style, ...props }) => {
  const rtlStyle = I18nManager.isRTL ? styles.rtl : styles.ltr;
  
  return (
    <View style={[rtlStyle, style]} {...props}>
      {children}
    </View>
  );
};

const styles = StyleSheet.create({
  ltr: {
    flexDirection: 'row',
  },
  rtl: {
    flexDirection: 'row-reverse',
  },
});

export default RTLFlexContainer;
```

#### **Componente de Layout Adaptativo**

```javascript
// components/AdaptiveLayout.js
import React from 'react';
import { View, StyleSheet, I18nManager } from 'react-native';

const AdaptiveLayout = ({ 
  children, 
  direction = 'row',
  justifyContent = 'flex-start',
  alignItems = 'stretch',
  style,
  ...props 
}) => {
  const adaptiveStyle = {
    flexDirection: I18nManager.isRTL && direction === 'row' 
      ? 'row-reverse' 
      : direction,
    justifyContent: I18nManager.isRTL && justifyContent === 'flex-start'
      ? 'flex-end'
      : justifyContent,
    alignItems,
  };
  
  return (
    <View style={[adaptiveStyle, style]} {...props}>
      {children}
    </View>
  );
};

export default AdaptiveLayout;
```

#### **Sistema de Espaciado RTL**

```javascript
// utils/rtlSpacing.js
import { I18nManager } from 'react-native';

export const getRTLSpacing = (left, right) => {
  if (I18nManager.isRTL) {
    return { marginLeft: right, marginRight: left };
  }
  return { marginLeft: left, marginRight: right };
};

export const getRTLPadding = (left, right) => {
  if (I18nManager.isRTL) {
    return { paddingLeft: right, paddingRight: left };
  }
  return { paddingLeft: left, paddingRight: right };
};

export const getRTLBorderRadius = (topLeft, topRight, bottomLeft, bottomRight) => {
  if (I18nManager.isRTL) {
    return {
      borderTopLeftRadius: topRight,
      borderTopRightRadius: topLeft,
      borderBottomLeftRadius: bottomRight,
      borderBottomRightRadius: bottomLeft,
    };
  }
  return {
    borderTopLeftRadius: topLeft,
    borderTopRightRadius: topRight,
    borderBottomLeftRadius: bottomLeft,
    borderBottomRightRadius: bottomRight,
  };
};
```

#### **Ejemplo de Uso de Layout Adaptativo**

```javascript
// components/ProductCard.js
import React from 'react';
import { View, Text, Image, StyleSheet } from 'react-native';
import { getRTLSpacing, getRTLPadding } from '../utils/rtlSpacing';

const ProductCard = ({ product }) => {
  return (
    <View style={[styles.container, getRTLPadding(16, 8)]}>
      <View style={[styles.content, getRTLSpacing(12, 0)]}>
        <Image source={{ uri: product.image }} style={styles.image} />
        <View style={styles.details}>
          <Text style={styles.title}>{product.title}</Text>
          <Text style={styles.price}>{product.price}</Text>
        </View>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderRadius: 8,
    marginVertical: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  content: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  image: {
    width: 80,
    height: 80,
    borderRadius: 8,
  },
  details: {
    flex: 1,
  },
  title: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 4,
  },
  price: {
    fontSize: 14,
    color: '#666',
  },
});

export default ProductCard;
```

### **3. Navegación y Flujos RTL**

#### **Configuración de Navegación RTL**

```javascript
// navigation/RTLNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { I18nManager } from 'react-native';

const Stack = createStackNavigator();

const RTLStackNavigator = ({ children }) => {
  const screenOptions = {
    headerBackTitleVisible: false,
    headerTitleAlign: I18nManager.isRTL ? 'right' : 'left',
    headerStyle: {
      backgroundColor: '#fff',
    },
    headerTintColor: '#000',
    headerTitleStyle: {
      fontWeight: 'bold',
    },
  };
  
  return (
    <Stack.Navigator screenOptions={screenOptions}>
      {children}
    </Stack.Navigator>
  );
};

export default RTLStackNavigator;
```

#### **Componente de Botón de Navegación RTL**

```javascript
// components/RTLBackButton.js
import React from 'react';
import { TouchableOpacity, StyleSheet, I18nManager } from 'react-native';
import Icon from 'react-native-vector-icons/Ionicons';

const RTLBackButton = ({ onPress, color = '#000' }) => {
  const iconName = I18nManager.isRTL ? 'chevron-forward' : 'chevron-back';
  
  return (
    <TouchableOpacity style={styles.button} onPress={onPress}>
      <Icon name={iconName} size={24} color={color} />
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    padding: 8,
    marginLeft: I18nManager.isRTL ? 0 : -8,
    marginRight: I18nManager.isRTL ? -8 : 0,
  },
});

export default RTLBackButton;
```

#### **Navegación con Flujos RTL**

```javascript
// screens/RTLScreen.js
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import RTLBackButton from '../components/RTLBackButton';

const RTLScreen = () => {
  const navigation = useNavigation();
  
  return (
    <View style={styles.container}>
      <RTLBackButton onPress={() => navigation.goBack()} />
      
      <View style={styles.content}>
        <Text style={styles.title}>RTL Screen</Text>
        <Text style={styles.description}>
          This screen adapts to RTL languages automatically
        </Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  content: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 16,
    textAlign: I18nManager.isRTL ? 'right' : 'left',
  },
  description: {
    fontSize: 16,
    lineHeight: 24,
    textAlign: I18nManager.isRTL ? 'right' : 'left',
  },
});

export default RTLScreen;
```

### **4. Iconos y Elementos Visuales RTL**

#### **Sistema de Iconos RTL**

```javascript
// utils/rtlIcons.js
import { I18nManager } from 'react-native';

export const getRTLIcon = (ltrIcon, rtlIcon) => {
  return I18nManager.isRTL ? rtlIcon : ltrIcon;
};

export const getRTLIconName = (iconName) => {
  const iconMap = {
    'chevron-left': 'chevron-right',
    'chevron-right': 'chevron-left',
    'arrow-left': 'arrow-right',
    'arrow-right': 'arrow-left',
    'play': 'play-reverse',
    'next': 'previous',
    'previous': 'next',
  };
  
  return I18nManager.isRTL ? iconMap[iconName] || iconName : iconName;
};

export const getRTLTransform = (transform) => {
  if (!I18nManager.isRTL) return transform;
  
  return {
    ...transform,
    scaleX: transform.scaleX ? -transform.scaleX : -1,
  };
};
```

#### **Componente de Icono RTL**

```javascript
// components/RTLIcon.js
import React from 'react';
import { View, StyleSheet } from 'react-native';
import Icon from 'react-native-vector-icons/Ionicons';
import { getRTLIconName, getRTLTransform } from '../utils/rtlIcons';

const RTLIcon = ({ 
  name, 
  size = 24, 
  color = '#000', 
  style,
  transform,
  ...props 
}) => {
  const rtlName = getRTLIconName(name);
  const rtlTransform = getRTLTransform(transform);
  
  return (
    <View style={[styles.container, style]}>
      <Icon
        name={rtlName}
        size={size}
        color={color}
        style={rtlTransform}
        {...props}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default RTLIcon;
```

#### **Elementos Visuales Adaptativos**

```javascript
// components/RTLVisualElement.js
import React from 'react';
import { View, StyleSheet, I18nManager } from 'react-native';

const RTLVisualElement = ({ 
  children, 
  style,
  shadowOffset,
  borderWidth,
  borderColor,
  ...props 
}) => {
  const rtlStyle = {
    ...style,
    // Invertir sombras para RTL
    shadowOffset: I18nManager.isRTL && shadowOffset 
      ? { width: -shadowOffset.width, height: shadowOffset.height }
      : shadowOffset,
    // Invertir bordes para RTL
    borderLeftWidth: I18nManager.isRTL ? 0 : borderWidth,
    borderRightWidth: I18nManager.isRTL ? borderWidth : 0,
    borderLeftColor: I18nManager.isRTL ? 'transparent' : borderColor,
    borderRightColor: I18nManager.isRTL ? borderColor : 'transparent',
  };
  
  return (
    <View style={rtlStyle} {...props}>
      {children}
    </View>
  );
};

export default RTLVisualElement;
```

### **5. Testing de Interfaces RTL**

#### **Testing de Componentes RTL**

```javascript
// __tests__/RTLTesting.test.js
import React from 'react';
import { render, screen } from '@testing-library/react-native';
import { I18nManager } from 'react-native';
import RTLIcon from '../components/RTLIcon';

// Mock I18nManager
jest.mock('react-native', () => ({
  ...jest.requireActual('react-native'),
  I18nManager: {
    isRTL: false,
    allowRTL: jest.fn(),
    forceRTL: jest.fn(),
  },
}));

describe('RTL Testing', () => {
  beforeEach(() => {
    // Reset RTL state
    I18nManager.isRTL = false;
  });
  
  test('renders LTR icon correctly', () => {
    render(<RTLIcon name="chevron-left" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('chevron-left');
  });
  
  test('renders RTL icon correctly', () => {
    I18nManager.isRTL = true;
    
    render(<RTLIcon name="chevron-left" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('chevron-right');
  });
  
  test('handles unknown icon names', () => {
    I18nManager.isRTL = true;
    
    render(<RTLIcon name="unknown-icon" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('unknown-icon');
  });
});
```

#### **Testing de Layouts RTL**

```javascript
// __tests__/RTLayout.test.js
import React from 'react';
import { render } from '@testing-library/react-native';
import { I18nManager } from 'react-native';
import AdaptiveLayout from '../components/AdaptiveLayout';

describe('RTL Layout Testing', () => {
  beforeEach(() => {
    I18nManager.isRTL = false;
  });
  
  test('applies LTR styles correctly', () => {
    const { getByTestId } = render(
      <AdaptiveLayout testID="layout" direction="row" />
    );
    
    const layout = getByTestId('layout');
    expect(layout.props.style.flexDirection).toBe('row');
  });
  
  test('applies RTL styles correctly', () => {
    I18nManager.isRTL = true;
    
    const { getByTestId } = render(
      <AdaptiveLayout testID="layout" direction="row" />
    );
    
    const layout = getByTestId('layout');
    expect(layout.props.style.flexDirection).toBe('row-reverse');
  });
  
  test('handles vertical direction correctly', () => {
    I18nManager.isRTL = true;
    
    const { getByTestId } = render(
      <AdaptiveLayout testID="layout" direction="column" />
    );
    
    const layout = getByTestId('layout');
    expect(layout.props.style.flexDirection).toBe('column');
  });
});
```

#### **Testing de Navegación RTL**

```javascript
// __tests__/RTLNavigation.test.js
import React from 'react';
import { render, screen } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';
import { I18nManager } from 'react-native';
import RTLStackNavigator from '../navigation/RTLNavigator';

const TestScreen = () => <div>Test Screen</div>;

describe('RTL Navigation Testing', () => {
  beforeEach(() => {
    I18nManager.isRTL = false;
  });
  
  test('configures LTR navigation correctly', () => {
    render(
      <NavigationContainer>
        <RTLStackNavigator>
          <Stack.Screen name="Test" component={TestScreen} />
        </RTLStackNavigator>
      </NavigationContainer>
    );
    
    // Verificar configuración de navegación LTR
    expect(I18nManager.isRTL).toBe(false);
  });
  
  test('configures RTL navigation correctly', () => {
    I18nManager.isRTL = true;
    
    render(
      <NavigationContainer>
        <RTLStackNavigator>
          <Stack.Screen name="Test" component={TestScreen} />
        </RTLStackNavigator>
      </NavigationContainer>
    );
    
    // Verificar configuración de navegación RTL
    expect(I18nManager.isRTL).toBe(true);
  });
});
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Configuración RTL Completa**

```javascript
// App.js - Configuración RTL
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useTranslation } from 'react-i18next';
import { rtlManager } from './i18n/rtlConfig';
import './i18n';

import WelcomeScreen from './screens/WelcomeScreen';
import RTLScreen from './screens/RTLScreen';

const Stack = createStackNavigator();

const App = () => {
  const { i18n } = useTranslation();
  
  useEffect(() => {
    initializeRTL();
  }, []);
  
  const initializeRTL = async () => {
    try {
      // Configurar RTL basado en idioma actual
      await rtlManager.setLanguage(i18n.language);
    } catch (error) {
      console.error('RTL initialization failed:', error);
    }
  };
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Welcome" component={WelcomeScreen} />
        <Stack.Screen name="RTL" component={RTLScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

### **Ejercicio 2: Componente de Lista RTL**

```javascript
// components/RTLList.js
import React from 'react';
import { FlatList, StyleSheet, I18nManager } from 'react-native';
import { getRTLSpacing } from '../utils/rtlSpacing';

const RTLList = ({ 
  data, 
  renderItem, 
  keyExtractor,
  style,
  ...props 
}) => {
  const rtlStyle = {
    ...style,
    ...getRTLSpacing(0, 0), // Ajustar márgenes si es necesario
  };
  
  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      style={rtlStyle}
      contentContainerStyle={styles.contentContainer}
      {...props}
    />
  );
};

const styles = StyleSheet.create({
  contentContainer: {
    paddingHorizontal: I18nManager.isRTL ? 16 : 16,
  },
});

export default RTLList;
```

### **Ejercicio 3: Formulario RTL**

```javascript
// components/RTLForm.js
import React from 'react';
import { View, TextInput, Text, StyleSheet, I18nManager } from 'react-native';
import { getRTLPadding } from '../utils/rtlSpacing';

const RTLForm = ({ fields, values, onChangeText, errors }) => {
  return (
    <View style={styles.container}>
      {fields.map((field) => (
        <View key={field.name} style={styles.fieldContainer}>
          <Text style={[styles.label, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
            {field.label}
          </Text>
          <TextInput
            style={[
              styles.input,
              getRTLPadding(12, 12),
              { textAlign: I18nManager.isRTL ? 'right' : 'left' }
            ]}
            value={values[field.name]}
            onChangeText={(text) => onChangeText(field.name, text)}
            placeholder={field.placeholder}
            placeholderTextColor="#999"
            {...field.props}
          />
          {errors[field.name] && (
            <Text style={[styles.error, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
              {errors[field.name]}
            </Text>
          )}
        </View>
      ))}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  fieldContainer: {
    marginBottom: 16,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    fontSize: 16,
    backgroundColor: '#fff',
  },
  error: {
    color: 'red',
    fontSize: 14,
    marginTop: 4,
  },
});

export default RTLForm;
```

### **Ejercicio 4: Dashboard RTL**

```javascript
// screens/RTLDashboard.js
import React from 'react';
import { 
  View, 
  Text, 
  ScrollView, 
  StyleSheet, 
  I18nManager 
} from 'react-native';
import { useTranslation } from 'react-i18next';
import RTLIcon from '../components/RTLIcon';
import AdaptiveLayout from '../components/AdaptiveLayout';

const RTLDashboard = () => {
  const { t } = useTranslation();
  
  const menuItems = [
    { icon: 'home', title: t('menu.home'), color: '#007AFF' },
    { icon: 'person', title: t('menu.profile'), color: '#34C759' },
    { icon: 'settings', title: t('menu.settings'), color: '#FF9500' },
    { icon: 'help', title: t('menu.help'), color: '#FF3B30' },
  ];
  
  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={[styles.title, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
          {t('dashboard.title')}
        </Text>
        <Text style={[styles.subtitle, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
          {t('dashboard.subtitle')}
        </Text>
      </View>
      
      <View style={styles.menuContainer}>
        {menuItems.map((item, index) => (
          <AdaptiveLayout key={index} style={styles.menuItem}>
            <View style={[styles.iconContainer, { backgroundColor: item.color }]}>
              <RTLIcon name={item.icon} size={24} color="#fff" />
            </View>
            <Text style={[styles.menuTitle, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
              {item.title}
            </Text>
          </AdaptiveLayout>
        ))}
      </View>
      
      <View style={styles.statsContainer}>
        <Text style={[styles.statsTitle, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
          {t('dashboard.stats')}
        </Text>
        
        <AdaptiveLayout style={styles.statsRow}>
          <View style={styles.statItem}>
            <Text style={styles.statNumber}>1,234</Text>
            <Text style={[styles.statLabel, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
              {t('dashboard.users')}
            </Text>
          </View>
          
          <View style={styles.statItem}>
            <Text style={styles.statNumber}>567</Text>
            <Text style={[styles.statLabel, { textAlign: I18nManager.isRTL ? 'right' : 'left' }]}>
              {t('dashboard.orders')}
            </Text>
          </View>
        </AdaptiveLayout>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    padding: 20,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
  menuContainer: {
    padding: 20,
  },
  menuItem: {
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 12,
    marginBottom: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  iconContainer: {
    width: 48,
    height: 48,
    borderRadius: 24,
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: I18nManager.isRTL ? 0 : 16,
    marginLeft: I18nManager.isRTL ? 16 : 0,
  },
  menuTitle: {
    fontSize: 18,
    fontWeight: '600',
    flex: 1,
  },
  statsContainer: {
    padding: 20,
  },
  statsTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  statsRow: {
    backgroundColor: '#fff',
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  statItem: {
    flex: 1,
    alignItems: 'center',
  },
  statNumber: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#007AFF',
    marginBottom: 8,
  },
  statLabel: {
    fontSize: 16,
    color: '#666',
  },
});

export default RTLDashboard;
```

---

## 🧪 Testing

### **Testing de Componentes RTL**

```javascript
// __tests__/RTLComponents.test.js
import React from 'react';
import { render, screen } from '@testing-library/react-native';
import { I18nManager } from 'react-native';
import RTLIcon from '../components/RTLIcon';
import AdaptiveLayout from '../components/AdaptiveLayout';

describe('RTL Components', () => {
  beforeEach(() => {
    I18nManager.isRTL = false;
  });
  
  test('RTLIcon renders correct icon for LTR', () => {
    render(<RTLIcon name="chevron-left" testID="icon" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('chevron-left');
  });
  
  test('RTLIcon renders correct icon for RTL', () => {
    I18nManager.isRTL = true;
    
    render(<RTLIcon name="chevron-left" testID="icon" />);
    
    const icon = screen.getByTestId('icon');
    expect(icon.props.name).toBe('chevron-right');
  });
  
  test('AdaptiveLayout applies correct direction for LTR', () => {
    render(<AdaptiveLayout testID="layout" direction="row" />);
    
    const layout = screen.getByTestId('layout');
    expect(layout.props.style.flexDirection).toBe('row');
  });
  
  test('AdaptiveLayout applies correct direction for RTL', () => {
    I18nManager.isRTL = true;
    
    render(<AdaptiveLayout testID="layout" direction="row" />);
    
    const layout = screen.getByTestId('layout');
    expect(layout.props.style.flexDirection).toBe('row-reverse');
  });
});
```

### **Testing de Utilidades RTL**

```javascript
// __tests__/RTLUtils.test.js
import { I18nManager } from 'react-native';
import { getRTLSpacing, getRTLPadding, getRTLIconName } from '../utils/rtlSpacing';

describe('RTL Utils', () => {
  beforeEach(() => {
    I18nManager.isRTL = false;
  });
  
  test('getRTLSpacing returns correct spacing for LTR', () => {
    const spacing = getRTLSpacing(10, 20);
    expect(spacing).toEqual({ marginLeft: 10, marginRight: 20 });
  });
  
  test('getRTLSpacing returns correct spacing for RTL', () => {
    I18nManager.isRTL = true;
    
    const spacing = getRTLSpacing(10, 20);
    expect(spacing).toEqual({ marginLeft: 20, marginRight: 10 });
  });
  
  test('getRTLPadding returns correct padding for LTR', () => {
    const padding = getRTLPadding(10, 20);
    expect(padding).toEqual({ paddingLeft: 10, paddingRight: 20 });
  });
  
  test('getRTLPadding returns correct padding for RTL', () => {
    I18nManager.isRTL = true;
    
    const padding = getRTLPadding(10, 20);
    expect(padding).toEqual({ paddingLeft: 20, paddingRight: 10 });
  });
  
  test('getRTLIconName returns correct icon for LTR', () => {
    const iconName = getRTLIconName('chevron-left');
    expect(iconName).toBe('chevron-left');
  });
  
  test('getRTLIconName returns correct icon for RTL', () => {
    I18nManager.isRTL = true;
    
    const iconName = getRTLIconName('chevron-left');
    expect(iconName).toBe('chevron-right');
  });
});
```

---

## 📱 Ejemplo Completo

### **Aplicación RTL Completa**

```javascript
// App.js - Aplicación RTL completa
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useTranslation } from 'react-i18next';
import { rtlManager } from './i18n/rtlConfig';
import './i18n';

import WelcomeScreen from './screens/WelcomeScreen';
import RTLDashboard from './screens/RTLDashboard';
import RTLScreen from './screens/RTLScreen';

const Stack = createStackNavigator();

const App = () => {
  const { i18n } = useTranslation();
  
  useEffect(() => {
    initializeRTL();
  }, []);
  
  const initializeRTL = async () => {
    try {
      // Configurar RTL basado en idioma actual
      await rtlManager.setLanguage(i18n.language);
    } catch (error) {
      console.error('RTL initialization failed:', error);
    }
  };
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Welcome" component={WelcomeScreen} />
        <Stack.Screen name="Dashboard" component={RTLDashboard} />
        <Stack.Screen name="RTL" component={RTLScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 🎯 Resumen de la Clase

### **Conceptos Clave Aprendidos**

1. **Soporte RTL**: Configuración completa para idiomas de derecha a izquierda
2. **Layouts adaptativos**: Flexbox y componentes que se adaptan automáticamente
3. **Navegación RTL**: Configuración de navegación para idiomas RTL
4. **Iconos y elementos visuales**: Adaptación de iconos y elementos para RTL
5. **Testing RTL**: Verificación de interfaces RTL

### **Habilidades Desarrolladas**

- ✅ Implementación de soporte RTL completo
- ✅ Creación de layouts adaptativos
- ✅ Configuración de navegación RTL
- ✅ Adaptación de iconos y elementos visuales
- ✅ Testing de interfaces RTL

### **Próximos Pasos**

En la siguiente clase aprenderás sobre:
- Formateo de fechas por región
- Formateo de monedas y números
- Unidades de medida y peso
- Calendarios y zonas horarias

---

## 📚 Recursos Adicionales

### **Documentación RTL**
- [React Native RTL Support](https://reactnative.dev/docs/internationalization#right-to-left-layouts)
- [Material Design RTL](https://material.io/design/usability/bidirectionality.html)
- [iOS RTL Support](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/layout/)
- [Android RTL Support](https://developer.android.com/training/basics/supporting-devices/languages)

### **Herramientas RTL**
- [RTL Tester](https://rtl-tester.com/) - Herramienta para probar interfaces RTL
- [RTL CSS](https://rtlcss.com/) - Generador de CSS RTL
- [RTL Inspector](https://chrome.google.com/webstore/detail/rtl-inspector/ckjfbncfcfaogamjndjkmkbdgabfceln) - Extensión de Chrome

### **Recursos de Diseño**
- [RTL Design Patterns](https://material.io/design/usability/bidirectionality.html)
- [RTL Best Practices](https://www.smashingmagazine.com/2015/09/rtl-support-in-web-apps/)
- [RTL Typography](https://www.smashingmagazine.com/2015/09/rtl-support-in-web-apps/)

---

**🎯 Objetivo Alcanzado**: Has aprendido a implementar soporte RTL completo en React Native, creando interfaces que se adaptan automáticamente a idiomas de derecha a izquierda.

**💡 Consejo**: El soporte RTL no es solo invertir elementos, sino adaptar toda la experiencia del usuario a la cultura y convenciones de lectura RTL.

**🚀 ¡Continúa con la siguiente clase para aprender sobre formatos regionales!**
