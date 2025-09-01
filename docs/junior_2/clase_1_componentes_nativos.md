# 📚 Clase 1: Componentes Nativos Básicos

## 🧭 Navegación del Módulo
- **⬅️ Anterior**: [README del Módulo](README.md)
- **➡️ Siguiente**: [Clase 2: JSX y Estructura](clase_2_jsx_estructura.md)
- **🏠 [Volver al Inicio](../../README.md)**

---

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Comprender los componentes nativos** fundamentales de React Native
2. **Implementar View, Text e Image** en aplicaciones reales
3. **Crear layouts básicos** usando componentes nativos
4. **Entender las diferencias** entre componentes web y nativos
5. **Aplicar estilos básicos** a componentes nativos

---

## 📚 Contenido Teórico

### **¿Qué son los Componentes Nativos?**

Los **componentes nativos** en React Native son elementos que se renderizan directamente como componentes nativos del sistema operativo:

#### **Componentes Básicos:**
- **View**: Contenedor equivalente a `<div>` en HTML
- **Text**: Texto equivalente a `<p>`, `<span>`, `<h1>` en HTML
- **Image**: Imagen equivalente a `<img>` en HTML
- **ScrollView**: Contenedor con scroll
- **TouchableOpacity**: Botón táctil

#### **Características:**
- **Rendimiento nativo**: Usan componentes reales del sistema
- **Comportamiento nativo**: Se comportan como elementos nativos
- **Accesibilidad**: Incluyen características de accesibilidad nativas
- **Plataforma específica**: Pueden tener comportamientos diferentes por plataforma

---

## 💻 Implementación Práctica

### **1. Componente View - Contenedor Principal**

```javascript:src/components/BasicView.js
import React from 'react';
import { View, StyleSheet } from 'react-native';

const BasicView = () => {
  return (
    <View style={styles.container}>
      {/* View como contenedor principal */}
      <View style={styles.header}>
        <View style={styles.headerContent}>
          <View style={styles.logo} />
          <View style={styles.titleBar} />
        </View>
      </View>

      {/* View como contenedor de contenido */}
      <View style={styles.content}>
        <View style={styles.sidebar}>
          <View style={styles.menuItem} />
          <View style={styles.menuItem} />
          <View style={styles.menuItem} />
        </View>
        
        <View style={styles.mainContent}>
          <View style={styles.card}>
            <View style={styles.cardHeader} />
            <View style={styles.cardBody} />
            <View style={styles.cardFooter} />
          </View>
          
          <View style={styles.card}>
            <View style={styles.cardHeader} />
            <View style={styles.cardBody} />
            <View style={styles.cardFooter} />
          </View>
        </View>
      </View>

      {/* View como footer */}
      <View style={styles.footer}>
        <View style={styles.footerContent} />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    height: 80,
    backgroundColor: '#007AFF',
    paddingTop: 40,
    paddingHorizontal: 20,
  },
  headerContent: {
    flex: 1,
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
  },
  logo: {
    width: 40,
    height: 40,
    backgroundColor: 'white',
    borderRadius: 20,
  },
  titleBar: {
    width: 120,
    height: 20,
    backgroundColor: 'white',
    borderRadius: 10,
  },
  content: {
    flex: 1,
    flexDirection: 'row',
  },
  sidebar: {
    width: 80,
    backgroundColor: '#e0e0e0',
    padding: 10,
  },
  menuItem: {
    width: '100%',
    height: 40,
    backgroundColor: '#ccc',
    marginBottom: 10,
    borderRadius: 8,
  },
  mainContent: {
    flex: 1,
    padding: 20,
  },
  card: {
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 16,
    marginBottom: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 4,
  },
  cardHeader: {
    width: '60%',
    height: 16,
    backgroundColor: '#007AFF',
    borderRadius: 8,
    marginBottom: 12,
  },
  cardBody: {
    width: '100%',
    height: 60,
    backgroundColor: '#f0f0f0',
    borderRadius: 8,
    marginBottom: 12,
  },
  cardFooter: {
    width: '40%',
    height: 12,
    backgroundColor: '#ccc',
    borderRadius: 6,
  },
  footer: {
    height: 60,
    backgroundColor: '#333',
    padding: 20,
  },
  footerContent: {
    width: '100%',
    height: '100%',
    backgroundColor: '#555',
    borderRadius: 8,
  },
});

export default BasicView;
```

### **2. Componente Text - Manejo de Texto**

```javascript:src/components/BasicText.js
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const BasicText = () => {
  return (
    <View style={styles.container}>
      {/* Text como título principal */}
      <Text style={styles.mainTitle}>
        Bienvenido a React Native
      </Text>

      {/* Text como subtítulo */}
      <Text style={styles.subtitle}>
        Aprende los fundamentos de los componentes nativos
      </Text>

      {/* Text como párrafo */}
      <Text style={styles.paragraph}>
        Los componentes de texto en React Native son fundamentales para mostrar 
        información al usuario. A diferencia de HTML, no necesitas envolver el 
        texto en elementos específicos como {'<p>'} o {'<span>'}.
      </Text>

      {/* Text con diferentes estilos */}
      <View style={styles.textExamples}>
        <Text style={styles.boldText}>
          Este texto está en negrita
        </Text>
        
        <Text style={styles.italicText}>
          Este texto está en cursiva
        </Text>
        
        <Text style={styles.underlinedText}>
          Este texto está subrayado
        </Text>
        
        <Text style={styles.coloredText}>
          Este texto tiene un color personalizado
        </Text>
      </View>

      {/* Text con diferentes tamaños */}
      <View style={styles.sizeExamples}>
        <Text style={styles.smallText}>Texto pequeño (12px)</Text>
        <Text style={styles.mediumText}>Texto mediano (16px)</Text>
        <Text style={styles.largeText}>Texto grande (20px)</Text>
        <Text style={styles.xlargeText}>Texto extra grande (24px)</Text>
      </View>

      {/* Text con diferentes alineaciones */}
      <View style={styles.alignmentExamples}>
        <Text style={styles.leftAligned}>Texto alineado a la izquierda</Text>
        <Text style={styles.centerAligned}>Texto centrado</Text>
        <Text style={styles.rightAligned}>Texto alineado a la derecha</Text>
      </View>

      {/* Text con diferentes pesos de fuente */}
      <View style={styles.weightExamples}>
        <Text style={styles.lightWeight}>Peso ligero</Text>
        <Text style={styles.normalWeight}>Peso normal</Text>
        <Text style={styles.mediumWeight}>Peso medio</Text>
        <Text style={styles.boldWeight}>Peso negrita</Text>
        <Text style={styles.heavyWeight}>Peso pesado</Text>
      </View>

      {/* Text con diferentes colores */}
      <View style={styles.colorExamples}>
        <Text style={styles.primaryColor}>Color primario</Text>
        <Text style={styles.secondaryColor}>Color secundario</Text>
        <Text style={styles.successColor}>Color de éxito</Text>
        <Text style={styles.warningColor}>Color de advertencia</Text>
        <Text style={styles.errorColor}>Color de error</Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  mainTitle: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#333',
    textAlign: 'center',
    marginBottom: 16,
  },
  subtitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#666',
    textAlign: 'center',
    marginBottom: 24,
  },
  paragraph: {
    fontSize: 16,
    lineHeight: 24,
    color: '#444',
    textAlign: 'justify',
    marginBottom: 24,
  },
  textExamples: {
    marginBottom: 24,
  },
  boldText: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 8,
  },
  italicText: {
    fontSize: 16,
    fontStyle: 'italic',
    color: '#333',
    marginBottom: 8,
  },
  underlinedText: {
    fontSize: 16,
    textDecorationLine: 'underline',
    color: '#333',
    marginBottom: 8,
  },
  coloredText: {
    fontSize: 16,
    color: '#007AFF',
    marginBottom: 8,
  },
  sizeExamples: {
    marginBottom: 24,
  },
  smallText: {
    fontSize: 12,
    color: '#666',
    marginBottom: 4,
  },
  mediumText: {
    fontSize: 16,
    color: '#333',
    marginBottom: 4,
  },
  largeText: {
    fontSize: 20,
    color: '#333',
    marginBottom: 4,
  },
  xlargeText: {
    fontSize: 24,
    color: '#333',
    marginBottom: 4,
  },
  alignmentExamples: {
    marginBottom: 24,
  },
  leftAligned: {
    fontSize: 16,
    textAlign: 'left',
    color: '#333',
    marginBottom: 8,
  },
  centerAligned: {
    fontSize: 16,
    textAlign: 'center',
    color: '#333',
    marginBottom: 8,
  },
  rightAligned: {
    fontSize: 16,
    textAlign: 'right',
    color: '#333',
    marginBottom: 8,
  },
  weightExamples: {
    marginBottom: 24,
  },
  lightWeight: {
    fontSize: 16,
    fontWeight: '300',
    color: '#333',
    marginBottom: 4,
  },
  normalWeight: {
    fontSize: 16,
    fontWeight: '400',
    color: '#333',
    marginBottom: 4,
  },
  mediumWeight: {
    fontSize: 16,
    fontWeight: '500',
    color: '#333',
    marginBottom: 4,
  },
  boldWeight: {
    fontSize: 16,
    fontWeight: '700',
    color: '#333',
    marginBottom: 4,
  },
  heavyWeight: {
    fontSize: 16,
    fontWeight: '900',
    color: '#333',
    marginBottom: 4,
  },
  colorExamples: {
    marginBottom: 24,
  },
  primaryColor: {
    fontSize: 16,
    color: '#007AFF',
    marginBottom: 4,
  },
  secondaryColor: {
    fontSize: 16,
    color: '#5856D6',
    marginBottom: 4,
  },
  successColor: {
    fontSize: 16,
    color: '#34C759',
    marginBottom: 4,
  },
  warningColor: {
    fontSize: 16,
    color: '#FF9500',
    marginBottom: 4,
  },
  errorColor: {
    fontSize: 16,
    color: '#FF3B30',
    marginBottom: 4,
  },
});

export default BasicText;
```

### **3. Componente Image - Manejo de Imágenes**

```javascript:src/components/BasicImage.js
import React, { useState } from 'react';
import { View, Image, Text, StyleSheet, TouchableOpacity } from 'react-native';

const BasicImage = () => {
  const [currentImage, setCurrentImage] = useState(0);

  // Array de imágenes de ejemplo
  const images = [
    {
      uri: 'https://picsum.photos/300/200?random=1',
      title: 'Imagen 1',
      description: 'Descripción de la imagen 1'
    },
    {
      uri: 'https://picsum.photos/300/200?random=2',
      title: 'Imagen 2',
      description: 'Descripción de la imagen 2'
    },
    {
      uri: 'https://picsum.photos/300/200?random=3',
      title: 'Imagen 3',
      description: 'Descripción de la imagen 3'
    },
    {
      uri: 'https://picsum.photos/300/200?random=4',
      title: 'Imagen 4',
      description: 'Descripción de la imagen 4'
    }
  ];

  const nextImage = () => {
    setCurrentImage((prev) => (prev + 1) % images.length);
  };

  const previousImage = () => {
    setCurrentImage((prev) => (prev - 1 + images.length) % images.length);
  };

  return (
    <View style={styles.container}>
      {/* Título de la sección */}
      <Text style={styles.title}>Componente Image</Text>
      
      {/* Contenedor de la imagen principal */}
      <View style={styles.imageContainer}>
        <Image
          source={{ uri: images[currentImage].uri }}
          style={styles.mainImage}
          resizeMode="cover"
          // Propiedades adicionales de Image
          fadeDuration={300}
          loadingIndicatorSource={{ uri: 'https://picsum.photos/50/50?random=loading' }}
        />
        
        {/* Overlay con información de la imagen */}
        <View style={styles.imageOverlay}>
          <Text style={styles.imageTitle}>{images[currentImage].title}</Text>
          <Text style={styles.imageDescription}>{images[currentImage].description}</Text>
        </View>
      </View>

      {/* Controles de navegación */}
      <View style={styles.controls}>
        <TouchableOpacity style={styles.controlButton} onPress={previousImage}>
          <Text style={styles.controlButtonText}>⬅️ Anterior</Text>
        </TouchableOpacity>
        
        <Text style={styles.imageCounter}>
          {currentImage + 1} de {images.length}
        </Text>
        
        <TouchableOpacity style={styles.controlButton} onPress={nextImage}>
          <Text style={styles.controlButtonText}>Siguiente ➡️</Text>
        </TouchableOpacity>
      </View>

      {/* Galería de miniaturas */}
      <View style={styles.thumbnailContainer}>
        <Text style={styles.thumbnailTitle}>Galería de Imágenes</Text>
        <View style={styles.thumbnailGrid}>
          {images.map((image, index) => (
            <TouchableOpacity
              key={index}
              style={[
                styles.thumbnail,
                index === currentImage && styles.activeThumbnail
              ]}
              onPress={() => setCurrentImage(index)}
            >
              <Image
                source={{ uri: image.uri }}
                style={styles.thumbnailImage}
                resizeMode="cover"
              />
            </TouchableOpacity>
          ))}
        </View>
      </View>

      {/* Ejemplos de diferentes resizeMode */}
      <View style={styles.resizeModeExamples}>
        <Text style={styles.resizeModeTitle}>Diferentes Modos de Redimensionamiento</Text>
        
        <View style={styles.resizeModeRow}>
          <View style={styles.resizeModeItem}>
            <Text style={styles.resizeModeLabel}>cover</Text>
            <Image
              source={{ uri: 'https://picsum.photos/100/100?random=cover' }}
              style={styles.resizeModeImage}
              resizeMode="cover"
            />
          </View>
          
          <View style={styles.resizeModeItem}>
            <Text style={styles.resizeModeLabel}>contain</Text>
            <Image
              source={{ uri: 'https://picsum.photos/100/100?random=contain' }}
              style={styles.resizeModeImage}
              resizeMode="contain"
            />
          </View>
          
          <View style={styles.resizeModeItem}>
            <Text style={styles.resizeModeLabel}>stretch</Text>
            <Image
              source={{ uri: 'https://picsum.photos/100/100?random=stretch' }}
              style={styles.resizeModeImage}
              resizeMode="stretch"
            />
          </View>
          
          <View style={styles.resizeModeItem}>
            <Text style={styles.resizeModeLabel}>center</Text>
            <Image
              source={{ uri: 'https://picsum.photos/100/100?random=center' }}
              style={styles.resizeModeImage}
              resizeMode="center"
            />
          </View>
        </View>
      </View>

      {/* Ejemplos de imágenes con diferentes estilos */}
      <View style={styles.styleExamples}>
        <Text style={styles.styleExamplesTitle}>Ejemplos de Estilos</Text>
        
        <View style={styles.styleRow}>
          <Image
            source={{ uri: 'https://picsum.photos/80/80?random=rounded' }}
            style={styles.roundedImage}
          />
          <Text style={styles.styleLabel}>Imagen redondeada</Text>
        </View>
        
        <View style={styles.styleRow}>
          <Image
            source={{ uri: 'https://picsum.photos/80/80?random=circle' }}
            style={styles.circularImage}
          />
          <Text style={styles.styleLabel}>Imagen circular</Text>
        </View>
        
        <View style={styles.styleRow}>
          <Image
            source={{ uri: 'https://picsum.photos/80/80?random=shadow' }}
            style={styles.shadowImage}
          />
          <Text style={styles.styleLabel}>Imagen con sombra</Text>
        </View>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    textAlign: 'center',
    marginBottom: 20,
  },
  imageContainer: {
    position: 'relative',
    marginBottom: 20,
  },
  mainImage: {
    width: '100%',
    height: 250,
    borderRadius: 12,
  },
  imageOverlay: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(0, 0, 0, 0.7)',
    padding: 16,
    borderBottomLeftRadius: 12,
    borderBottomRightRadius: 12,
  },
  imageTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 4,
  },
  imageDescription: {
    fontSize: 14,
    color: 'rgba(255, 255, 255, 0.8)',
  },
  controls: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    marginBottom: 20,
  },
  controlButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  controlButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  imageCounter: {
    fontSize: 16,
    color: '#666',
    fontWeight: '600',
  },
  thumbnailContainer: {
    marginBottom: 20,
  },
  thumbnailTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#333',
    marginBottom: 12,
  },
  thumbnailGrid: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  thumbnail: {
    width: 70,
    height: 70,
    borderRadius: 8,
    overflow: 'hidden',
    borderWidth: 2,
    borderColor: 'transparent',
  },
  activeThumbnail: {
    borderColor: '#007AFF',
  },
  thumbnailImage: {
    width: '100%',
    height: '100%',
  },
  resizeModeExamples: {
    marginBottom: 20,
  },
  resizeModeTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#333',
    marginBottom: 12,
  },
  resizeModeRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  resizeModeItem: {
    alignItems: 'center',
  },
  resizeModeLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 8,
    textAlign: 'center',
  },
  resizeModeImage: {
    width: 80,
    height: 80,
    borderRadius: 8,
    backgroundColor: '#e0e0e0',
  },
  styleExamples: {
    marginBottom: 20,
  },
  styleExamplesTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#333',
    marginBottom: 12,
  },
  styleRow: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 16,
  },
  roundedImage: {
    width: 80,
    height: 80,
    borderRadius: 12,
    marginRight: 16,
  },
  circularImage: {
    width: 80,
    height: 80,
    borderRadius: 40,
    marginRight: 16,
  },
  shadowImage: {
    width: 80,
    height: 80,
    borderRadius: 8,
    marginRight: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.3,
    shadowRadius: 8,
    elevation: 8,
  },
  styleLabel: {
    fontSize: 16,
    color: '#333',
  },
});

export default BasicImage;
```

---

## 🧪 Casos de Uso Prácticos

### **1. Crear un Layout Básico**

```javascript:src/screens/BasicLayoutScreen.js
import React from 'react';
import { SafeAreaView, ScrollView } from 'react-native';
import BasicView from '../components/BasicView';
import BasicText from '../components/BasicText';
import BasicImage from '../components/BasicImage';

const BasicLayoutScreen = () => {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <ScrollView>
        <BasicView />
        <BasicText />
        <BasicImage />
      </ScrollView>
    </SafeAreaView>
  );
};

export default BasicLayoutScreen;
```

### **2. Hook para Gestión de Imágenes**

```javascript:src/hooks/useImageGallery.js
import { useState, useCallback } from 'react';

export const useImageGallery = (images = []) => {
  const [currentIndex, setCurrentIndex] = useState(0);
  const [isLoading, setIsLoading] = useState(false);

  const nextImage = useCallback(() => {
    setCurrentIndex((prev) => (prev + 1) % images.length);
  }, [images.length]);

  const previousImage = useCallback(() => {
    setCurrentIndex((prev) => (prev - 1 + images.length) % images.length);
  }, [images.length]);

  const goToImage = useCallback((index) => {
    if (index >= 0 && index < images.length) {
      setCurrentIndex(index);
    }
  }, [images.length]);

  const currentImage = images[currentIndex] || null;

  return {
    currentIndex,
    currentImage,
    isLoading,
    setIsLoading,
    nextImage,
    previousImage,
    goToImage,
    totalImages: images.length,
    hasNext: currentIndex < images.length - 1,
    hasPrevious: currentIndex > 0,
  };
};
```

---

## 📝 Ejercicios Prácticos

### **Ejercicio 1: Layout Básico**
Crea un layout simple usando solo componentes View con diferentes colores y tamaños.

### **Ejercicio 2: Texto Estilizado**
Implementa diferentes estilos de texto y crea una tarjeta de información personal.

### **Ejercicio 3: Galería de Imágenes**
Crea una galería simple con navegación entre imágenes usando los componentes aprendidos.

### **Ejercicio 4: Componente Compuesto**
Combina View, Text e Image para crear un componente de tarjeta de producto.

---

## 🎯 Proyecto de la Clase

### **App de Componentes Básicos**

Crea una aplicación que demuestre el uso de todos los componentes nativos básicos.

**Requisitos:**
- Layout con múltiples secciones usando View
- Diferentes estilos de texto con Text
- Galería de imágenes con Image
- Navegación entre secciones
- Estilos atractivos y responsive

---

## 📚 Recursos Adicionales

### **Documentación:**
- [React Native Components](https://reactnative.dev/docs/components-and-apis)
- [View Component](https://reactnative.dev/docs/view)
- [Text Component](https://reactnative.dev/docs/text)
- [Image Component](https://reactnative.dev/docs/image)

### **Artículos:**
- [Understanding React Native Components](https://medium.com/@reactnative/understanding-react-native-components-4c5b5b5b5b5b)
- [Styling in React Native](https://medium.com/@reactnative/styling-in-react-native-5b5b5b5b5b5b)

---

## 📋 Resumen de la Clase

### **✅ Lo Que Aprendiste:**
1. **Componentes nativos fundamentales** de React Native
2. **Implementación de View** como contenedor principal
3. **Manejo de texto** con diferentes estilos y propiedades
4. **Gestión de imágenes** con múltiples opciones de configuración
5. **Creación de layouts básicos** combinando componentes

### **🚀 Próximos Pasos:**
- Aprender JSX y estructura de componentes
- Entender Flexbox para layouts
- Crear componentes personalizados

### **💡 Conceptos Clave:**
- **View**: Contenedor principal para layouts
- **Text**: Componente para mostrar texto
- **Image**: Componente para mostrar imágenes
- **StyleSheet**: Sistema de estilos de React Native
- **Props**: Propiedades de los componentes

---

**🎯 Objetivo**: Dominar los componentes nativos básicos para crear interfaces simples y funcionales.

**💡 Consejo**: Practica creando diferentes layouts usando solo estos componentes básicos antes de avanzar a componentes más complejos.
