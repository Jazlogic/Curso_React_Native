# 📱 Clase 4: Optimización de Imágenes

## 🎯 Objetivos de la Clase

### **Al Finalizar Esta Clase Serás Capaz de:**
1. **Implementar** lazy loading de imágenes en listas y grids
2. **Configurar** sistemas de caching para imágenes
3. **Optimizar** el tamaño y formato de imágenes
4. **Implementar** placeholders y loading states
5. **Crear** sistemas de pre-carga inteligente

---

## 📚 Contenido Teórico

### **1. Lazy Loading de Imágenes**

#### **A. Lazy Loading Básico**
```javascript
const LazyImage = ({ uri, width, height, placeholder }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isVisible, setIsVisible] = useState(false);
  
  const onLoad = useCallback(() => {
    setIsLoaded(true);
  }, []);
  
  const onError = useCallback(() => {
    console.error('Error loading image:', uri);
  }, [uri]);
  
  return (
    <View style={{ width, height }}>
      {!isVisible && (
        <View style={[styles.placeholder, { width, height }]}>
          {placeholder}
        </View>
      )}
      
      {isVisible && (
        <Image
          source={{ uri }}
          style={[styles.image, { width, height }]}
          onLoad={onLoad}
          onError={onError}
          fadeDuration={300}
          loadingIndicatorSource={placeholder}
        />
      )}
    </View>
  );
};
```

#### **B. Lazy Loading con Intersection Observer**
```javascript
const LazyImageWithObserver = ({ uri, width, height, placeholder }) => {
  const [isVisible, setIsVisible] = useState(false);
  const [isLoaded, setIsLoaded] = useState(false);
  const imageRef = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.unobserve(entry.target);
        }
      },
      {
        threshold: 0.1,
        rootMargin: '50px',
      }
    );
    
    if (imageRef.current) {
      observer.observe(imageRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <View ref={imageRef} style={{ width, height }}>
      {!isVisible && (
        <View style={[styles.placeholder, { width, height }]}>
          {placeholder}
        </View>
      )}
      
      {isVisible && (
        <Image
          source={{ uri }}
          style={[styles.image, { width, height }]}
          onLoad={() => setIsLoaded(true)}
          fadeDuration={300}
        />
      )}
    </View>
  );
};
```

### **2. Sistemas de Caching**

#### **A. Caching Básico con AsyncStorage**
```javascript
const ImageCache = {
  async get(key) {
    try {
      const cached = await AsyncStorage.getItem(`image_${key}`);
      if (cached) {
        const { uri, timestamp, expiresIn } = JSON.parse(cached);
        if (Date.now() - timestamp < expiresIn) {
          return uri;
        }
      }
      return null;
    } catch (error) {
      console.error('Error getting cached image:', error);
      return null;
    }
  },
  
  async set(key, uri, expiresIn = 24 * 60 * 60 * 1000) { // 24 horas
    try {
      const cacheData = {
        uri,
        timestamp: Date.now(),
        expiresIn,
      };
      await AsyncStorage.setItem(`image_${key}`, JSON.stringify(cacheData));
    } catch (error) {
      console.error('Error setting cached image:', error);
    }
  },
  
  async clear() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      const imageKeys = keys.filter(key => key.startsWith('image_'));
      await AsyncStorage.multiRemove(imageKeys);
    } catch (error) {
      console.error('Error clearing image cache:', error);
    }
  }
};

const CachedImage = ({ uri, width, height, placeholder }) => {
  const [cachedUri, setCachedUri] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    const loadCachedImage = async () => {
      const cached = await ImageCache.get(uri);
      if (cached) {
        setCachedUri(cached);
        setIsLoading(false);
      } else {
        setIsLoading(false);
      }
    };
    
    loadCachedImage();
  }, [uri]);
  
  const onLoad = useCallback(async () => {
    await ImageCache.set(uri, uri);
  }, [uri]);
  
  if (isLoading) {
    return (
      <View style={[styles.placeholder, { width, height }]}>
        {placeholder}
      </View>
    );
  }
  
  return (
    <Image
      source={{ uri: cachedUri || uri }}
      style={[styles.image, { width, height }]}
      onLoad={onLoad}
      fadeDuration={300}
    />
  );
};
```

#### **B. Caching Avanzado con React Native Fast Image**
```javascript
import FastImage from 'react-native-fast-image';

const FastCachedImage = ({ uri, width, height, priority = 'normal' }) => {
  return (
    <FastImage
      source={{ uri, priority: FastImage.priority[priority.toUpperCase()] }}
      style={{ width, height }}
      resizeMode={FastImage.resizeMode.cover}
    />
  );
};

// Configuración global de cache
FastImage.preload([
  { uri: 'https://example.com/image1.jpg' },
  { uri: 'https://example.com/image2.jpg' },
]);

// Limpiar cache
FastImage.clearMemoryCache();
FastImage.clearDiskCache();
```

### **3. Optimización de Tamaño y Formato**

#### **A. Redimensionamiento de Imágenes**
```javascript
const OptimizedImage = ({ uri, width, height, quality = 0.8 }) => {
  const [optimizedUri, setOptimizedUri] = useState(uri);
  
  useEffect(() => {
    const optimizeImage = async () => {
      try {
        // Simular optimización de imagen
        const response = await fetch(uri);
        const blob = await response.blob();
        
        // Crear canvas para redimensionar
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        
        canvas.width = width;
        canvas.height = height;
        
        // Dibujar imagen redimensionada
        const img = new Image();
        img.onload = () => {
          ctx.drawImage(img, 0, 0, width, height);
          
          // Convertir a blob con calidad optimizada
          canvas.toBlob(
            (blob) => {
              const optimizedUrl = URL.createObjectURL(blob);
              setOptimizedUri(optimizedUrl);
            },
            'image/jpeg',
            quality
          );
        };
        img.src = URL.createObjectURL(blob);
      } catch (error) {
        console.error('Error optimizing image:', error);
        setOptimizedUri(uri); // Fallback a imagen original
      }
    };
    
    optimizeImage();
  }, [uri, width, height, quality]);
  
  return (
    <Image
      source={{ uri: optimizedUri }}
      style={{ width, height }}
      resizeMode="cover"
      fadeDuration={0}
    />
  );
};
```

#### **B. Formato WebP y Compresión**
```javascript
const WebPImage = ({ uri, width, height }) => {
  const [webPUri, setWebPUri] = useState(null);
  
  useEffect(() => {
    const convertToWebP = async () => {
      try {
        // Verificar si el dispositivo soporta WebP
        const supportsWebP = await Image.resolveAssetSource(
          require('./test.webp')
        );
        
        if (supportsWebP) {
          // Convertir URI a WebP
          const webPUrl = uri.replace(/\.(jpg|jpeg|png)$/i, '.webp');
          setWebPUri(webPUrl);
        } else {
          setWebPUri(uri); // Fallback a formato original
        }
      } catch (error) {
        console.error('Error converting to WebP:', error);
        setWebPUri(uri);
      }
    };
    
    convertToWebP();
  }, [uri]);
  
  return (
    <Image
      source={{ uri: webPUri || uri }}
      style={{ width, height }}
      resizeMode="cover"
    />
  );
};
```

### **4. Placeholders y Loading States**

#### **A. Placeholder Inteligente**
```javascript
const SmartPlaceholder = ({ width, height, text, color = '#f0f0f0' }) => {
  return (
    <View style={[styles.placeholder, { width, height, backgroundColor: color }]}>
      <Text style={styles.placeholderText}>{text}</Text>
    </View>
  );
};

const ProgressiveImage = ({ uri, width, height, placeholder, lowResUri }) => {
  const [currentUri, setCurrentUri] = useState(lowResUri || placeholder);
  const [isHighResLoaded, setIsHighResLoaded] = useState(false);
  
  const onHighResLoad = useCallback(() => {
    setIsHighResLoaded(true);
    setCurrentUri(uri);
  }, [uri]);
  
  return (
    <View style={{ width, height }}>
      {/* Imagen de baja resolución como placeholder */}
      {lowResUri && !isHighResLoaded && (
        <Image
          source={{ uri: lowResUri }}
          style={[styles.image, { width, height }]}
          blurRadius={5}
        />
      )}
      
      {/* Placeholder estático */}
      {!lowResUri && !isHighResLoaded && (
        <SmartPlaceholder
          width={width}
          height={height}
          text="Cargando..."
        />
      )}
      
      {/* Imagen de alta resolución */}
      <Image
        source={{ uri }}
        style={[
          styles.image,
          { width, height },
          isHighResLoaded && styles.highResImage
        ]}
        onLoad={onHighResLoad}
        fadeDuration={300}
      />
    </View>
  );
};
```

#### **B. Skeleton Loading**
```javascript
const ImageSkeleton = ({ width, height }) => {
  const [isAnimating, setIsAnimating] = useState(true);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setIsAnimating(prev => !prev);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return (
    <View style={[styles.skeleton, { width, height }]}>
      <View
        style={[
          styles.skeletonShimmer,
          {
            opacity: isAnimating ? 0.3 : 0.7,
          },
        ]}
      />
    </View>
  );
};

const SkeletonImage = ({ uri, width, height }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <View style={{ width, height }}>
      {!isLoaded && (
        <ImageSkeleton width={width} height={height} />
      )}
      
      <Image
        source={{ uri }}
        style={[
          styles.image,
          { width, height },
          !isLoaded && styles.hiddenImage
        ]}
        onLoad={() => setIsLoaded(true)}
        fadeDuration={300}
      />
    </View>
  );
};
```

### **5. Pre-carga Inteligente**

#### **A. Pre-carga de Imágenes Visibles**
```javascript
const ImagePreloader = ({ images, onPreloadComplete }) => {
  const [preloadedImages, setPreloadedImages] = useState(new Set());
  
  const preloadImage = useCallback(async (uri) => {
    try {
      await Image.prefetch(uri);
      setPreloadedImages(prev => new Set([...prev, uri]));
    } catch (error) {
      console.error('Error preloading image:', uri, error);
    }
  }, []);
  
  const preloadAll = useCallback(async () => {
    const preloadPromises = images.map(uri => preloadImage(uri));
    await Promise.all(preloadPromises);
    onPreloadComplete?.(Array.from(preloadedImages));
  }, [images, preloadImage, onPreloadComplete]);
  
  useEffect(() => {
    preloadAll();
  }, [preloadAll]);
  
  return null; // Componente invisible
};

const PreloadedImageList = ({ images }) => {
  const [isPreloading, setIsPreloading] = useState(true);
  
  const handlePreloadComplete = useCallback((preloadedUris) => {
    setIsPreloading(false);
    console.log(`Preloaded ${preloadedUris.length} images`);
  }, []);
  
  return (
    <View>
      {isPreloading && (
        <View style={styles.preloadIndicator}>
          <ActivityIndicator size="large" />
          <Text>Precargando imágenes...</Text>
        </View>
      )}
      
      <ImagePreloader
        images={images}
        onPreloadComplete={handlePreloadComplete}
      />
      
      {!isPreloading && (
        <FlatList
          data={images}
          renderItem={({ item }) => (
            <CachedImage
              uri={item}
              width={200}
              height={200}
            />
          )}
          keyExtractor={(item) => item}
          numColumns={2}
        />
      )}
    </View>
  );
};
```

#### **B. Pre-carga Condicional**
```javascript
const ConditionalPreloader = ({ images, shouldPreload, priority = 'low' }) => {
  const [preloadedImages, setPreloadedImages] = useState(new Set());
  
  useEffect(() => {
    if (!shouldPreload) return;
    
    const preloadImages = async () => {
      const preloadPromises = images.map(async (uri) => {
        try {
          if (priority === 'high') {
            await Image.prefetch(uri);
          } else {
            // Pre-carga en background para prioridad baja
            Image.prefetch(uri).catch(() => {});
          }
          setPreloadedImages(prev => new Set([...prev, uri]));
        } catch (error) {
          console.error('Error preloading image:', uri, error);
        }
      });
      
      if (priority === 'high') {
        await Promise.all(preloadPromises);
      }
    };
    
    preloadImages();
  }, [images, shouldPreload, priority]);
  
  return null;
};

const SmartImageGallery = ({ images, isVisible }) => {
  return (
    <View>
      <ConditionalPreloader
        images={images}
        shouldPreload={isVisible}
        priority="high"
      />
      
      <FlatList
        data={images}
        renderItem={({ item }) => (
          <CachedImage
            uri={item}
            width={150}
            height={150}
          />
        )}
        keyExtractor={(item) => item}
        numColumns={3}
      />
    </View>
  );
};
```

---

## 💻 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Caching Completo**
Implementa un sistema de caching de imágenes con AsyncStorage.

```javascript
const ImageCacheSystem = () => {
  // Implementa:
  // - Cache en memoria y disco
  // - Expiración automática
  // - Limpieza de cache
  // - Métricas de cache
};
```

**Tarea**: Crea un sistema que cachee 100 imágenes con expiración.

### **Ejercicio 2: Lazy Loading en Lista**
Implementa lazy loading de imágenes en una lista de productos.

```javascript
const LazyImageList = ({ products }) => {
  // Implementa:
  // - Lazy loading con Intersection Observer
  // - Placeholders inteligentes
  // - Pre-carga de imágenes próximas
  // - Fallbacks para errores
};
```

**Tarea**: Crea una lista de 50 productos con lazy loading.

### **Ejercicio 3: Optimización de Imágenes**
Implementa un sistema de optimización automática de imágenes.

```javascript
const ImageOptimizer = ({ images, targetSize, quality }) => {
  // Implementa:
  // - Redimensionamiento automático
  // - Compresión de calidad
  // - Conversión a WebP
  // - Batch processing
};
```

**Tarea**: Optimiza 20 imágenes de diferentes tamaños.

---

## 🎯 Proyecto Integrador

### **Galería de Imágenes Ultra-Optimizada**

Crea una galería que demuestre todas las técnicas de optimización:

#### **Funcionalidades Requeridas:**
1. **Lazy Loading**: De imágenes según visibilidad
2. **Sistema de Cache**: En memoria y disco
3. **Optimización Automática**: Redimensionamiento y compresión
4. **Placeholders Inteligentes**: Skeleton loading y placeholders
5. **Pre-carga Condicional**: Según prioridad y visibilidad

#### **Requisitos Técnicos:**
- **Lazy Loading**: Con Intersection Observer
- **Cache Inteligente**: Con expiración automática
- **Optimización**: Redimensionamiento y compresión
- **Placeholders**: Skeleton loading y placeholders
- **Pre-carga**: Condicional y por prioridad

---

## 📚 Recursos Adicionales

### **Documentación Oficial:**
- [React Native Image](https://reactnative.dev/docs/image)
- [Fast Image](https://github.com/DylanVann/react-native-fast-image)
- [Image Caching](https://reactnative.dev/docs/image#cache)

### **Artículos Recomendados:**
- "Image Optimization in React Native"
- "Lazy Loading Images Best Practices"
- "Image Caching Strategies"

---

## 🎓 Próximos Pasos

### **En la Siguiente Clase Aprenderás:**
- **Optimización de Navegación**: Lazy loading de pantallas y transiciones

---

**🎯 Objetivo de la Clase**: Dominar la optimización de imágenes en React Native para crear aplicaciones con carga rápida y uso eficiente de memoria.

**💡 Consejo**: Las imágenes son el recurso más pesado en aplicaciones móviles. Optimizarlas correctamente puede mejorar dramáticamente el performance de tu app.
