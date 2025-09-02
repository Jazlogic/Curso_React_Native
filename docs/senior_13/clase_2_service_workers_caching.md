# 📱 Clase 2: Service Workers y Caching

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar Service Workers para crear funcionalidad offline, configurar estrategias de caching avanzadas, manejar actualizaciones y sincronización en segundo plano. Dominarás las técnicas esenciales para hacer que tu PWA funcione sin conexión a internet.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** Service Workers desde cero
2. **Configurar** estrategias de caching apropiadas
3. **Manejar** funcionalidad offline y fallbacks
4. **Implementar** background sync y actualizaciones
5. **Optimizar** el rendimiento con caching inteligente

---

## 📚 Contenido de la Clase

### **1. Fundamentos de Service Workers**

#### **¿Qué es un Service Worker?**
Un Service Worker es un script que se ejecuta en segundo plano, separado de la página web, y actúa como un proxy entre la aplicación y la red.

#### **Características Principales**
- **Ejecución en segundo plano** independiente de la página
- **Control de red** y caching de recursos
- **Eventos de ciclo de vida** (install, activate, fetch)
- **Persistencia** entre sesiones del navegador
- **Comunicación** con la página principal via postMessage

#### **Ciclo de Vida del Service Worker**
```javascript
// sw.js - Service Worker básico
const CACHE_NAME = 'mi-app-v1';
const urlsToCache = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/manifest.json'
];

// Evento de instalación
self.addEventListener('install', (event) => {
  console.log('Service Worker: Instalando...');
  
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log('Service Worker: Cache abierto');
        return cache.addAll(urlsToCache);
      })
      .then(() => {
        // Forzar activación inmediata
        return self.skipWaiting();
      })
  );
});

// Evento de activación
self.addEventListener('activate', (event) => {
  console.log('Service Worker: Activando...');
  
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_NAME) {
            console.log('Service Worker: Eliminando cache antiguo:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    }).then(() => {
      // Tomar control de todas las páginas
      return self.clients.claim();
    })
  );
});

// Evento de fetch
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Devolver desde cache si está disponible
        if (response) {
          return response;
        }
        
        // Si no está en cache, hacer fetch de la red
        return fetch(event.request);
      })
  );
});
```

### **2. Estrategias de Caching**

#### **Cache First Strategy**
Ideal para recursos estáticos que raramente cambian.

```javascript
// Cache First - Para assets estáticos
self.addEventListener('fetch', (event) => {
  if (event.request.destination === 'image' || 
      event.request.destination === 'style' ||
      event.request.destination === 'script') {
    
    event.respondWith(
      caches.match(event.request)
        .then((response) => {
          if (response) {
            return response;
          }
          
          return fetch(event.request).then((response) => {
            // Verificar si la respuesta es válida
            if (!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }
            
            // Clonar la respuesta
            const responseToCache = response.clone();
            
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, responseToCache);
              });
            
            return response;
          });
        })
    );
  }
});
```

#### **Network First Strategy**
Ideal para datos dinámicos que deben estar actualizados.

```javascript
// Network First - Para datos dinámicos
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      fetch(event.request)
        .then((response) => {
          // Si la red responde, actualizar cache
          if (response.status === 200) {
            const responseToCache = response.clone();
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, responseToCache);
              });
          }
          return response;
        })
        .catch(() => {
          // Si la red falla, devolver desde cache
          return caches.match(event.request);
        })
    );
  }
});
```

#### **Stale While Revalidate Strategy**
Combina velocidad del cache con datos actualizados.

```javascript
// Stale While Revalidate
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((cachedResponse) => {
        // Hacer fetch en segundo plano para actualizar
        const fetchPromise = fetch(event.request).then((networkResponse) => {
          // Actualizar cache con nueva respuesta
          caches.open(CACHE_NAME)
            .then((cache) => {
              cache.put(event.request, networkResponse.clone());
            });
          return networkResponse;
        });
        
        // Devolver cache inmediatamente, o fetch si no hay cache
        return cachedResponse || fetchPromise;
      })
  );
});
```

#### **Network Only Strategy**
Para requests críticos que siempre deben ser frescos.

```javascript
// Network Only - Para autenticación y datos críticos
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/auth/') || 
      event.request.url.includes('/payment/')) {
    event.respondWith(fetch(event.request));
  }
});
```

### **3. Funcionalidad Offline y Fallbacks**

#### **Página Offline Personalizada**
```javascript
// Fallback para páginas offline
self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request)
        .catch(() => {
          return caches.match('/offline.html');
        })
    );
  }
});
```

#### **Fallback para Imágenes**
```javascript
// Fallback para imágenes
self.addEventListener('fetch', (event) => {
  if (event.request.destination === 'image') {
    event.respondWith(
      caches.match(event.request)
        .then((response) => {
          if (response) {
            return response;
          }
          
          return fetch(event.request)
            .catch(() => {
              // Devolver imagen placeholder
              return caches.match('/images/placeholder.png');
            });
        })
    );
  }
});
```

#### **Fallback para API**
```javascript
// Fallback para API calls
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      fetch(event.request)
        .then((response) => {
          if (response.status === 200) {
            // Guardar respuesta exitosa en cache
            const responseToCache = response.clone();
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, responseToCache);
              });
          }
          return response;
        })
        .catch(() => {
          // Devolver datos del cache o respuesta por defecto
          return caches.match(event.request)
            .then((cachedResponse) => {
              if (cachedResponse) {
                return cachedResponse;
              }
              
              // Respuesta por defecto para offline
              return new Response(
                JSON.stringify({ 
                  error: 'Sin conexión', 
                  offline: true 
                }),
                {
                  status: 200,
                  headers: { 'Content-Type': 'application/json' }
                }
              );
            });
        })
    );
  }
});
```

### **4. Background Sync y Actualizaciones**

#### **Background Sync**
```javascript
// Background Sync para operaciones offline
self.addEventListener('sync', (event) => {
  if (event.tag === 'background-sync') {
    event.waitUntil(doBackgroundSync());
  }
});

async function doBackgroundSync() {
  try {
    // Obtener datos pendientes de IndexedDB
    const pendingData = await getPendingData();
    
    // Enviar datos a servidor
    for (const data of pendingData) {
      await fetch('/api/sync', {
        method: 'POST',
        body: JSON.stringify(data),
        headers: {
          'Content-Type': 'application/json'
        }
      });
      
      // Marcar como sincronizado
      await markAsSynced(data.id);
    }
  } catch (error) {
    console.error('Error en background sync:', error);
  }
}
```

#### **Push Notifications**
```javascript
// Manejo de push notifications
self.addEventListener('push', (event) => {
  const options = {
    body: event.data ? event.data.text() : 'Nueva notificación',
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    vibrate: [100, 50, 100],
    data: {
      dateOfArrival: Date.now(),
      primaryKey: 1
    },
    actions: [
      {
        action: 'explore',
        title: 'Ver detalles',
        icon: '/icons/checkmark.png'
      },
      {
        action: 'close',
        title: 'Cerrar',
        icon: '/icons/xmark.png'
      }
    ]
  };
  
  event.waitUntil(
    self.registration.showNotification('Mi PWA', options)
  );
});

// Manejo de clicks en notificaciones
self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  
  if (event.action === 'explore') {
    // Abrir la app
    event.waitUntil(
      clients.openWindow('/')
    );
  }
});
```

#### **Actualizaciones Automáticas**
```javascript
// Detección de actualizaciones
self.addEventListener('message', (event) => {
  if (event.data && event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});

// Notificar a la página sobre actualizaciones
self.addEventListener('controllerchange', () => {
  // Recargar la página para usar el nuevo Service Worker
  window.location.reload();
});
```

### **5. Optimización de Rendimiento**

#### **Cache Inteligente**
```javascript
// Cache inteligente con versionado
const CACHE_VERSION = 'v1.0.0';
const CACHE_NAME = `mi-app-${CACHE_VERSION}`;

// Estrategia de cache con TTL
const CACHE_TTL = {
  static: 7 * 24 * 60 * 60 * 1000, // 7 días
  dynamic: 24 * 60 * 60 * 1000,    // 1 día
  api: 5 * 60 * 1000               // 5 minutos
};

self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  const cacheKey = getCacheKey(event.request);
  
  event.respondWith(
    caches.match(cacheKey)
      .then((cachedResponse) => {
        if (cachedResponse) {
          const cacheTime = cachedResponse.headers.get('sw-cache-time');
          const now = Date.now();
          
          if (cacheTime && (now - parseInt(cacheTime)) < CACHE_TTL.static) {
            return cachedResponse;
          }
        }
        
        return fetch(event.request)
          .then((response) => {
            if (response.status === 200) {
              const responseToCache = response.clone();
              responseToCache.headers.set('sw-cache-time', Date.now().toString());
              
              caches.open(CACHE_NAME)
                .then((cache) => {
                  cache.put(cacheKey, responseToCache);
                });
            }
            return response;
          });
      })
  );
});
```

#### **Precaching de Recursos Críticos**
```javascript
// Precaching de recursos críticos
const CRITICAL_RESOURCES = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/manifest.json',
  '/offline.html'
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        return cache.addAll(CRITICAL_RESOURCES);
      })
      .then(() => {
        return self.skipWaiting();
      })
  );
});
```

---

## 🛠️ Implementación Práctica

### **1. Service Worker Avanzado**

#### **Configuración Completa**
```javascript
// sw.js - Service Worker completo
const CACHE_VERSION = 'v1.0.0';
const CACHE_NAME = `mi-pwa-${CACHE_VERSION}`;
const STATIC_CACHE = `static-${CACHE_VERSION}`;
const DYNAMIC_CACHE = `dynamic-${CACHE_VERSION}`;

// Recursos estáticos para precaching
const STATIC_ASSETS = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/manifest.json',
  '/offline.html',
  '/icons/icon-192x192.png',
  '/icons/icon-512x512.png'
];

// Instalación
self.addEventListener('install', (event) => {
  console.log('SW: Instalando...');
  
  event.waitUntil(
    caches.open(STATIC_CACHE)
      .then((cache) => {
        console.log('SW: Precaching recursos estáticos');
        return cache.addAll(STATIC_ASSETS);
      })
      .then(() => {
        console.log('SW: Instalación completada');
        return self.skipWaiting();
      })
      .catch((error) => {
        console.error('SW: Error en instalación:', error);
      })
  );
});

// Activación
self.addEventListener('activate', (event) => {
  console.log('SW: Activando...');
  
  event.waitUntil(
    caches.keys()
      .then((cacheNames) => {
        return Promise.all(
          cacheNames.map((cacheName) => {
            if (cacheName !== STATIC_CACHE && cacheName !== DYNAMIC_CACHE) {
              console.log('SW: Eliminando cache antiguo:', cacheName);
              return caches.delete(cacheName);
            }
          })
        );
      })
      .then(() => {
        console.log('SW: Activación completada');
        return self.clients.claim();
      })
  );
});

// Fetch con estrategias múltiples
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Estrategia según el tipo de recurso
  if (request.destination === 'document') {
    event.respondWith(networkFirst(request));
  } else if (request.destination === 'image') {
    event.respondWith(cacheFirst(request));
  } else if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request));
  } else {
    event.respondWith(staleWhileRevalidate(request));
  }
});

// Estrategias de caching
async function cacheFirst(request) {
  const cachedResponse = await caches.match(request);
  if (cachedResponse) {
    return cachedResponse;
  }
  
  try {
    const networkResponse = await fetch(request);
    if (networkResponse.status === 200) {
      const cache = await caches.open(STATIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    return networkResponse;
  } catch (error) {
    // Fallback para imágenes
    if (request.destination === 'image') {
      return caches.match('/images/placeholder.png');
    }
    throw error;
  }
}

async function networkFirst(request) {
  try {
    const networkResponse = await fetch(request);
    if (networkResponse.status === 200) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    return networkResponse;
  } catch (error) {
    const cachedResponse = await caches.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    // Fallback para páginas
    if (request.mode === 'navigate') {
      return caches.match('/offline.html');
    }
    
    throw error;
  }
}

async function staleWhileRevalidate(request) {
  const cachedResponse = await caches.match(request);
  
  const fetchPromise = fetch(request).then((networkResponse) => {
    if (networkResponse.status === 200) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    return networkResponse;
  });
  
  return cachedResponse || fetchPromise;
}
```

### **2. Integración con React Native Web**

#### **Hook para Service Worker**
```javascript
// useServiceWorker.js
import { useState, useEffect } from 'react';

export const useServiceWorker = () => {
  const [swRegistration, setSwRegistration] = useState(null);
  const [updateAvailable, setUpdateAvailable] = useState(false);
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    // Registrar Service Worker
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/sw.js')
        .then((registration) => {
          setSwRegistration(registration);
          
          // Escuchar actualizaciones
          registration.addEventListener('updatefound', () => {
            const newWorker = registration.installing;
            newWorker.addEventListener('statechange', () => {
              if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                setUpdateAvailable(true);
              }
            });
          });
        })
        .catch((error) => {
          console.error('Error registrando SW:', error);
        });
    }

    // Escuchar cambios de conectividad
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  const updateServiceWorker = () => {
    if (swRegistration && swRegistration.waiting) {
      swRegistration.waiting.postMessage({ type: 'SKIP_WAITING' });
      window.location.reload();
    }
  };

  return {
    swRegistration,
    updateAvailable,
    isOnline,
    updateServiceWorker
  };
};
```

#### **Componente de Estado de Conexión**
```javascript
// ConnectionStatus.js
import React from 'react';
import { useServiceWorker } from './useServiceWorker';

const ConnectionStatus = () => {
  const { isOnline, updateAvailable, updateServiceWorker } = useServiceWorker();

  if (!isOnline) {
    return (
      <div className="connection-status offline">
        <span>🔴 Sin conexión - Modo offline</span>
      </div>
    );
  }

  if (updateAvailable) {
    return (
      <div className="connection-status update">
        <span>🔄 Actualización disponible</span>
        <button onClick={updateServiceWorker}>
          Actualizar
        </button>
      </div>
    );
  }

  return (
    <div className="connection-status online">
      <span>🟢 Conectado</span>
    </div>
  );
};

export default ConnectionStatus;
```

### **3. Testing de Service Workers**

#### **Testing con Jest**
```javascript
// sw.test.js
import { registerSW, unregisterSW } from './sw';

// Mock del Service Worker API
const mockServiceWorker = {
  register: jest.fn(),
  ready: Promise.resolve(),
  controller: null
};

Object.defineProperty(navigator, 'serviceWorker', {
  value: mockServiceWorker,
  writable: true
});

describe('Service Worker', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('debe registrar Service Worker correctamente', async () => {
    mockServiceWorker.register.mockResolvedValue({
      installing: null,
      waiting: null,
      active: null
    });

    await registerSW();
    
    expect(mockServiceWorker.register).toHaveBeenCalledWith('/sw.js');
  });

  test('debe manejar errores de registro', async () => {
    mockServiceWorker.register.mockRejectedValue(new Error('Registration failed'));
    
    await expect(registerSW()).rejects.toThrow('Registration failed');
  });
});
```

#### **Testing de Caching**
```javascript
// cache.test.js
import { cacheFirst, networkFirst } from './sw';

// Mock de caches
const mockCache = {
  match: jest.fn(),
  put: jest.fn(),
  addAll: jest.fn()
};

global.caches = {
  open: jest.fn().mockResolvedValue(mockCache),
  match: jest.fn()
};

global.fetch = jest.fn();

describe('Caching Strategies', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('cacheFirst debe devolver cache si está disponible', async () => {
    const mockRequest = new Request('/test');
    const mockResponse = new Response('cached data');
    
    mockCache.match.mockResolvedValue(mockResponse);
    
    const result = await cacheFirst(mockRequest);
    
    expect(result).toBe(mockResponse);
    expect(mockCache.match).toHaveBeenCalledWith(mockRequest);
  });

  test('networkFirst debe hacer fetch si no hay cache', async () => {
    const mockRequest = new Request('/api/test');
    const mockResponse = new Response('network data');
    
    mockCache.match.mockResolvedValue(null);
    global.fetch.mockResolvedValue(mockResponse);
    
    const result = await networkFirst(mockRequest);
    
    expect(result).toBe(mockResponse);
    expect(global.fetch).toHaveBeenCalledWith(mockRequest);
  });
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Service Worker Básico**
Implementa un Service Worker que:
- Precache recursos estáticos
- Implemente cache-first para imágenes
- Implemente network-first para API calls
- Maneje fallbacks para offline

### **Ejercicio 2: Estrategias de Caching**
Crea diferentes estrategias de caching para:
- Recursos estáticos (CSS, JS, imágenes)
- Datos dinámicos (API calls)
- Páginas HTML
- Assets de terceros

### **Ejercicio 3: Background Sync**
Implementa background sync para:
- Envío de formularios offline
- Sincronización de datos
- Notificaciones push
- Actualizaciones automáticas

---

## 📖 Recursos Adicionales

### **Documentación**
- [Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
- [Background Sync](https://developer.mozilla.org/en-US/docs/Web/API/Background_Sync_API)

### **Herramientas**
- [Workbox](https://developers.google.com/web/tools/workbox)
- [Service Worker Toolbox](https://github.com/GoogleChrome/sw-toolbox)
- [PWA Builder](https://www.pwabuilder.com/)

### **Ejemplos**
- [Service Worker Examples](https://github.com/GoogleChrome/samples/tree/gh-pages/service-worker)
- [PWA Examples](https://github.com/GoogleChrome/samples/tree/gh-pages/pwa)

---

## 🚀 Próximos Pasos

En la siguiente clase aprenderás sobre **Offline First y Sincronización**, donde implementarás persistencia local de datos, estrategias de sincronización y resolución de conflictos para crear una experiencia offline completa.

---

**💡 Consejo**: Los Service Workers son la base de las PWA. Domina las estrategias de caching y funcionalidad offline para crear aplicaciones verdaderamente progresivas.

**🎯 Objetivo**: Al final de esta clase, tendrás un Service Worker completamente funcional que maneje caching, offline y actualizaciones de manera inteligente.
