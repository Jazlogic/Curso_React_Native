# 📱 Clase 5: Instalación y App-like Experience

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a configurar Web App Manifest para instalación nativa, implementar install prompts personalizados, crear app shortcuts y funcionalidades rápidas, y optimizar la experiencia para que se sienta como una aplicación nativa. Dominarás las técnicas para hacer que tu PWA sea completamente instalable y funcional.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Configurar** Web App Manifest completo
2. **Implementar** install prompts personalizados
3. **Crear** app shortcuts y funcionalidades rápidas
4. **Optimizar** la experiencia app-like
5. **Testing** y debugging de PWA

---

## 📚 Contenido de la Clase

### **1. Web App Manifest Avanzado**

#### **Configuración Completa del Manifest**
```json
{
  "name": "Mi Aplicación PWA",
  "short_name": "MiApp",
  "description": "Una aplicación PWA increíble con todas las características",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "orientation": "portrait-primary",
  "scope": "/",
  "lang": "es",
  "dir": "ltr",
  "categories": ["productivity", "utilities", "business"],
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/desktop-1.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide",
      "label": "Vista principal de la aplicación"
    },
    {
      "src": "/screenshots/mobile-1.png",
      "sizes": "390x844",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Vista móvil de la aplicación"
    }
  ],
  "shortcuts": [
    {
      "name": "Nuevo Post",
      "short_name": "Nuevo",
      "description": "Crear un nuevo post",
      "url": "/new-post",
      "icons": [
        {
          "src": "/icons/shortcut-new.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Mis Favoritos",
      "short_name": "Favoritos",
      "description": "Ver posts favoritos",
      "url": "/favorites",
      "icons": [
        {
          "src": "/icons/shortcut-favorites.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Configuración",
      "short_name": "Config",
      "description": "Abrir configuración",
      "url": "/settings",
      "icons": [
        {
          "src": "/icons/shortcut-settings.png",
          "sizes": "96x96"
        }
      ]
    }
  ],
  "related_applications": [
    {
      "platform": "play",
      "url": "https://play.google.com/store/apps/details?id=com.miapp",
      "id": "com.miapp"
    },
    {
      "platform": "itunes",
      "url": "https://apps.apple.com/app/miapp/id123456789"
    }
  ],
  "prefer_related_applications": false,
  "edge_side_panel": {
    "preferred_width": 400
  },
  "launch_handler": {
    "client_mode": "navigate-existing"
  }
}
```

### **2. Install Prompts Personalizados**

#### **Sistema de Install Prompts**
```javascript
// installPrompt.js - Sistema de install prompts
class InstallPromptManager {
  constructor() {
    this.deferredPrompt = null;
    this.isInstalled = false;
    this.installPromptShown = false;
    this.setupEventListeners();
  }

  setupEventListeners() {
    // Escuchar evento beforeinstallprompt
    window.addEventListener('beforeinstallprompt', (e) => {
      e.preventDefault();
      this.deferredPrompt = e;
      this.showInstallPrompt();
    });

    // Escuchar evento appinstalled
    window.addEventListener('appinstalled', () => {
      this.isInstalled = true;
      this.hideInstallPrompt();
      this.trackInstallation();
    });

    // Detectar si ya está instalada
    this.checkIfInstalled();
  }

  // Verificar si la app ya está instalada
  checkIfInstalled() {
    // Verificar display mode
    if (window.matchMedia('(display-mode: standalone)').matches) {
      this.isInstalled = true;
      return;
    }

    // Verificar si se ejecuta desde la pantalla de inicio
    if (window.navigator.standalone === true) {
      this.isInstalled = true;
      return;
    }

    // Verificar referrer
    if (document.referrer.includes('android-app://')) {
      this.isInstalled = true;
      return;
    }
  }

  // Mostrar prompt de instalación
  showInstallPrompt() {
    if (this.isInstalled || this.installPromptShown) {
      return;
    }

    this.installPromptShown = true;
    this.createInstallPrompt();
  }

  // Crear prompt de instalación personalizado
  createInstallPrompt() {
    const prompt = document.createElement('div');
    prompt.className = 'install-prompt';
    prompt.innerHTML = `
      <div class="install-prompt-content">
        <div class="install-prompt-icon">
          <img src="/icons/icon-192x192.png" alt="App Icon">
        </div>
        <div class="install-prompt-text">
          <h3>Instalar Mi App</h3>
          <p>Instala nuestra app para una mejor experiencia</p>
          <ul class="install-benefits">
            <li>✓ Acceso rápido desde la pantalla de inicio</li>
            <li>✓ Funciona offline</li>
            <li>✓ Notificaciones push</li>
            <li>✓ Experiencia nativa</li>
          </ul>
        </div>
        <div class="install-prompt-actions">
          <button class="install-prompt-dismiss">Ahora no</button>
          <button class="install-prompt-install">Instalar</button>
        </div>
      </div>
    `;

    document.body.appendChild(prompt);

    // Manejar clicks
    prompt.querySelector('.install-prompt-install').onclick = () => {
      this.installApp();
    };

    prompt.querySelector('.install-prompt-dismiss').onclick = () => {
      this.dismissInstallPrompt();
    };

    // Auto-ocultar después de 10 segundos
    setTimeout(() => {
      if (document.body.contains(prompt)) {
        this.dismissInstallPrompt();
      }
    }, 10000);
  }

  // Instalar la app
  async installApp() {
    if (!this.deferredPrompt) {
      return;
    }

    try {
      // Mostrar el prompt nativo
      this.deferredPrompt.prompt();
      
      // Esperar la respuesta del usuario
      const { outcome } = await this.deferredPrompt.userChoice;
      
      if (outcome === 'accepted') {
        console.log('Usuario aceptó la instalación');
        this.trackInstallation('accepted');
      } else {
        console.log('Usuario rechazó la instalación');
        this.trackInstallation('dismissed');
      }
      
      this.deferredPrompt = null;
      this.hideInstallPrompt();
    } catch (error) {
      console.error('Error durante la instalación:', error);
    }
  }

  // Ocultar prompt de instalación
  dismissInstallPrompt() {
    const prompt = document.querySelector('.install-prompt');
    if (prompt) {
      prompt.remove();
    }
    this.installPromptShown = false;
  }

  // Ocultar prompt (después de instalación)
  hideInstallPrompt() {
    this.dismissInstallPrompt();
  }

  // Rastrear instalación
  trackInstallation(outcome = 'installed') {
    // Enviar analytics
    if (typeof gtag !== 'undefined') {
      gtag('event', 'pwa_install', {
        event_category: 'PWA',
        event_label: outcome,
        value: 1
      });
    }

    // Enviar a servidor
    fetch('/api/analytics/install', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        outcome,
        timestamp: Date.now(),
        userAgent: navigator.userAgent
      })
    }).catch(error => {
      console.error('Error enviando analytics de instalación:', error);
    });
  }
}
```

### **3. App Shortcuts y Funcionalidades Rápidas**

#### **Sistema de Shortcuts**
```javascript
// appShortcuts.js - Sistema de shortcuts
class AppShortcuts {
  constructor() {
    this.shortcuts = [];
    this.loadShortcuts();
  }

  // Cargar shortcuts desde manifest
  loadShortcuts() {
    // Los shortcuts se definen en el manifest.json
    // Aquí manejamos la lógica de navegación
    this.setupShortcutHandlers();
  }

  // Configurar manejadores de shortcuts
  setupShortcutHandlers() {
    // Escuchar eventos de shortcuts
    window.addEventListener('shortcut', (event) => {
      this.handleShortcut(event.detail);
    });

    // Manejar navegación desde shortcuts
    this.handleShortcutNavigation();
  }

  // Manejar navegación desde shortcuts
  handleShortcutNavigation() {
    const urlParams = new URLSearchParams(window.location.search);
    const shortcut = urlParams.get('shortcut');
    
    if (shortcut) {
      this.handleShortcut(shortcut);
    }
  }

  // Manejar shortcut específico
  handleShortcut(shortcutName) {
    switch (shortcutName) {
      case 'new-post':
        this.navigateToNewPost();
        break;
      case 'favorites':
        this.navigateToFavorites();
        break;
      case 'settings':
        this.navigateToSettings();
        break;
      default:
        console.log('Shortcut no reconocido:', shortcutName);
    }
  }

  // Navegar a nuevo post
  navigateToNewPost() {
    // Verificar autenticación
    if (!this.isAuthenticated()) {
      this.showLoginPrompt();
      return;
    }

    // Navegar a la página de nuevo post
    window.location.href = '/new-post';
  }

  // Navegar a favoritos
  navigateToFavorites() {
    window.location.href = '/favorites';
  }

  // Navegar a configuración
  navigateToSettings() {
    window.location.href = '/settings';
  }

  // Verificar autenticación
  isAuthenticated() {
    return localStorage.getItem('authToken') !== null;
  }

  // Mostrar prompt de login
  showLoginPrompt() {
    const modal = document.createElement('div');
    modal.className = 'login-prompt-modal';
    modal.innerHTML = `
      <div class="login-prompt-content">
        <h3>Iniciar Sesión</h3>
        <p>Necesitas iniciar sesión para crear un nuevo post</p>
        <div class="login-prompt-actions">
          <button class="login-prompt-cancel">Cancelar</button>
          <button class="login-prompt-login">Iniciar Sesión</button>
        </div>
      </div>
    `;

    document.body.appendChild(modal);

    // Manejar clicks
    modal.querySelector('.login-prompt-login').onclick = () => {
      window.location.href = '/login';
    };

    modal.querySelector('.login-prompt-cancel').onclick = () => {
      modal.remove();
    };
  }

  // Crear shortcut dinámico
  createDynamicShortcut(name, url, icon) {
    // Esto requeriría actualizar el manifest dinámicamente
    // Por ahora, manejamos la lógica de navegación
    this.shortcuts.push({ name, url, icon });
  }
}
```

### **4. Optimización de Experiencia App-like**

#### **Sistema de Optimización**
```javascript
// appLikeExperience.js - Optimización de experiencia
class AppLikeExperience {
  constructor() {
    this.isStandalone = window.matchMedia('(display-mode: standalone)').matches;
    this.setupAppLikeFeatures();
  }

  // Configurar características app-like
  setupAppLikeFeatures() {
    this.hideBrowserUI();
    this.setupStatusBar();
    this.setupSplashScreen();
    this.setupNavigation();
    this.setupGestures();
  }

  // Ocultar UI del navegador
  hideBrowserUI() {
    if (this.isStandalone) {
      // Ocultar elementos del navegador
      document.documentElement.classList.add('standalone-mode');
      
      // Ajustar viewport para status bar
      this.adjustViewportForStatusBar();
    }
  }

  // Ajustar viewport para status bar
  adjustViewportForStatusBar() {
    const metaViewport = document.querySelector('meta[name="viewport"]');
    if (metaViewport) {
      metaViewport.content = 'width=device-width, initial-scale=1.0, viewport-fit=cover';
    }
  }

  // Configurar status bar
  setupStatusBar() {
    if (this.isStandalone) {
      // Crear status bar personalizada
      const statusBar = document.createElement('div');
      statusBar.className = 'status-bar';
      statusBar.innerHTML = `
        <div class="status-bar-content">
          <div class="status-bar-left">
            <span class="status-bar-time">${new Date().toLocaleTimeString()}</span>
          </div>
          <div class="status-bar-center">
            <span class="status-bar-title">Mi App</span>
          </div>
          <div class="status-bar-right">
            <span class="status-bar-battery">100%</span>
            <span class="status-bar-signal">📶</span>
          </div>
        </div>
      `;

      document.body.insertBefore(statusBar, document.body.firstChild);

      // Actualizar tiempo cada minuto
      setInterval(() => {
        const timeElement = statusBar.querySelector('.status-bar-time');
        if (timeElement) {
          timeElement.textContent = new Date().toLocaleTimeString();
        }
      }, 60000);
    }
  }

  // Configurar splash screen
  setupSplashScreen() {
    if (this.isStandalone) {
      const splashScreen = document.createElement('div');
      splashScreen.className = 'splash-screen';
      splashScreen.innerHTML = `
        <div class="splash-screen-content">
          <img src="/icons/icon-512x512.png" alt="App Icon" class="splash-screen-icon">
          <h1 class="splash-screen-title">Mi App</h1>
          <div class="splash-screen-loader">
            <div class="loader"></div>
          </div>
        </div>
      `;

      document.body.appendChild(splashScreen);

      // Ocultar splash screen cuando la app esté lista
      window.addEventListener('load', () => {
        setTimeout(() => {
          splashScreen.classList.add('fade-out');
          setTimeout(() => {
            splashScreen.remove();
          }, 500);
        }, 1000);
      });
    }
  }

  // Configurar navegación
  setupNavigation() {
    // Crear barra de navegación personalizada
    const navBar = document.createElement('nav');
    navBar.className = 'app-nav-bar';
    navBar.innerHTML = `
      <div class="nav-bar-content">
        <button class="nav-back-button" onclick="history.back()">
          ←
        </button>
        <h1 class="nav-title">Mi App</h1>
        <button class="nav-menu-button" onclick="this.toggleMenu()">
          ☰
        </button>
      </div>
    `;

    document.body.insertBefore(navBar, document.body.firstChild);

    // Actualizar título de navegación
    this.updateNavTitle();
  }

  // Actualizar título de navegación
  updateNavTitle() {
    const navTitle = document.querySelector('.nav-title');
    if (navTitle) {
      const pageTitle = document.title;
      navTitle.textContent = pageTitle;
    }
  }

  // Configurar gestos
  setupGestures() {
    let startY = 0;
    let startX = 0;

    // Swipe para volver atrás
    document.addEventListener('touchstart', (e) => {
      startY = e.touches[0].clientY;
      startX = e.touches[0].clientX;
    });

    document.addEventListener('touchend', (e) => {
      const endY = e.changedTouches[0].clientY;
      const endX = e.changedTouches[0].clientX;
      const diffY = startY - endY;
      const diffX = startX - endX;

      // Swipe horizontal para navegación
      if (Math.abs(diffX) > Math.abs(diffY) && Math.abs(diffX) > 100) {
        if (diffX > 0) {
          // Swipe izquierda - ir adelante
          history.forward();
        } else {
          // Swipe derecha - ir atrás
          history.back();
        }
      }
    });
  }
}
```

### **5. Testing y Debugging de PWA**

#### **Sistema de Testing**
```javascript
// pwaTesting.js - Testing de PWA
class PWATesting {
  constructor() {
    this.tests = [];
    this.results = {};
  }

  // Ejecutar todos los tests
  async runAllTests() {
    console.log('🧪 Iniciando tests de PWA...');
    
    const testResults = {
      manifest: await this.testManifest(),
      serviceWorker: await this.testServiceWorker(),
      installability: await this.testInstallability(),
      offline: await this.testOfflineFunctionality(),
      performance: await this.testPerformance(),
      accessibility: await this.testAccessibility()
    };

    this.results = testResults;
    this.displayResults(testResults);
    
    return testResults;
  }

  // Test del manifest
  async testManifest() {
    const results = {
      hasManifest: false,
      validManifest: false,
      hasIcons: false,
      hasShortcuts: false,
      errors: []
    };

    try {
      const manifestLink = document.querySelector('link[rel="manifest"]');
      if (manifestLink) {
        results.hasManifest = true;
        
        const response = await fetch(manifestLink.href);
        const manifest = await response.json();
        
        // Validar campos requeridos
        const requiredFields = ['name', 'short_name', 'start_url', 'display', 'icons'];
        const missingFields = requiredFields.filter(field => !manifest[field]);
        
        if (missingFields.length === 0) {
          results.validManifest = true;
        } else {
          results.errors.push(`Campos faltantes: ${missingFields.join(', ')}`);
        }
        
        // Validar iconos
        if (manifest.icons && manifest.icons.length > 0) {
          results.hasIcons = true;
        } else {
          results.errors.push('No se encontraron iconos');
        }
        
        // Validar shortcuts
        if (manifest.shortcuts && manifest.shortcuts.length > 0) {
          results.hasShortcuts = true;
        }
      } else {
        results.errors.push('No se encontró el manifest');
      }
    } catch (error) {
      results.errors.push(`Error cargando manifest: ${error.message}`);
    }

    return results;
  }

  // Test del Service Worker
  async testServiceWorker() {
    const results = {
      hasServiceWorker: false,
      isRegistered: false,
      isActive: false,
      errors: []
    };

    try {
      if ('serviceWorker' in navigator) {
        results.hasServiceWorker = true;
        
        const registration = await navigator.serviceWorker.getRegistration();
        if (registration) {
          results.isRegistered = true;
          
          if (registration.active) {
            results.isActive = true;
          } else {
            results.errors.push('Service Worker no está activo');
          }
        } else {
          results.errors.push('Service Worker no está registrado');
        }
      } else {
        results.errors.push('Service Worker no soportado');
      }
    } catch (error) {
      results.errors.push(`Error verificando Service Worker: ${error.message}`);
    }

    return results;
  }

  // Test de instalabilidad
  async testInstallability() {
    const results = {
      isInstallable: false,
      hasInstallPrompt: false,
      isStandalone: false,
      errors: []
    };

    try {
      // Verificar si es instalable
      if (window.matchMedia('(display-mode: standalone)').matches) {
        results.isStandalone = true;
        results.isInstallable = true;
      } else {
        // Verificar criterios de instalabilidad
        const manifestLink = document.querySelector('link[rel="manifest"]');
        if (manifestLink) {
          const response = await fetch(manifestLink.href);
          const manifest = await response.json();
          
          if (manifest.name && manifest.short_name && manifest.icons && manifest.start_url) {
            results.isInstallable = true;
          } else {
            results.errors.push('Manifest no cumple criterios de instalabilidad');
          }
        }
      }
      
      // Verificar si hay install prompt disponible
      if (window.deferredPrompt) {
        results.hasInstallPrompt = true;
      }
    } catch (error) {
      results.errors.push(`Error verificando instalabilidad: ${error.message}`);
    }

    return results;
  }

  // Test de funcionalidad offline
  async testOfflineFunctionality() {
    const results = {
      worksOffline: false,
      hasOfflinePage: false,
      hasCacheStrategy: false,
      errors: []
    };

    try {
      // Simular offline
      const originalOnline = navigator.onLine;
      Object.defineProperty(navigator, 'onLine', {
        writable: true,
        value: false
      });

      // Disparar evento offline
      window.dispatchEvent(new Event('offline'));

      // Intentar cargar la página
      const response = await fetch(window.location.href);
      if (response.ok) {
        results.worksOffline = true;
      }

      // Restaurar estado online
      Object.defineProperty(navigator, 'onLine', {
        writable: true,
        value: originalOnline
      });

      // Verificar página offline
      const offlineResponse = await fetch('/offline.html');
      if (offlineResponse.ok) {
        results.hasOfflinePage = true;
      }

    } catch (error) {
      results.errors.push(`Error verificando funcionalidad offline: ${error.message}`);
    }

    return results;
  }

  // Test de performance
  async testPerformance() {
    const results = {
      loadTime: 0,
      isFast: false,
      hasLazyLoading: false,
      errors: []
    };

    try {
      // Medir tiempo de carga
      const loadTime = performance.timing.loadEventEnd - performance.timing.navigationStart;
      results.loadTime = loadTime;
      
      if (loadTime < 3000) {
        results.isFast = true;
      }

      // Verificar lazy loading
      const lazyImages = document.querySelectorAll('img[loading="lazy"]');
      if (lazyImages.length > 0) {
        results.hasLazyLoading = true;
      }

    } catch (error) {
      results.errors.push(`Error verificando performance: ${error.message}`);
    }

    return results;
  }

  // Test de accesibilidad
  async testAccessibility() {
    const results = {
      hasAltText: false,
      hasHeadings: false,
      hasFocusManagement: false,
      errors: []
    };

    try {
      // Verificar alt text en imágenes
      const images = document.querySelectorAll('img');
      const imagesWithAlt = document.querySelectorAll('img[alt]');
      if (images.length === imagesWithAlt.length) {
        results.hasAltText = true;
      } else {
        results.errors.push('Algunas imágenes no tienen alt text');
      }

      // Verificar estructura de headings
      const headings = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
      if (headings.length > 0) {
        results.hasHeadings = true;
      } else {
        results.errors.push('No se encontraron headings');
      }

      // Verificar manejo de focus
      const focusableElements = document.querySelectorAll('button, input, select, textarea, a[href]');
      if (focusableElements.length > 0) {
        results.hasFocusManagement = true;
      }

    } catch (error) {
      results.errors.push(`Error verificando accesibilidad: ${error.message}`);
    }

    return results;
  }

  // Mostrar resultados
  displayResults(results) {
    console.log('📊 Resultados de tests de PWA:');
    
    Object.entries(results).forEach(([testName, result]) => {
      console.log(`\n${testName.toUpperCase()}:`);
      console.log(`✅ Éxitos: ${Object.values(result).filter(v => v === true).length}`);
      console.log(`❌ Errores: ${result.errors ? result.errors.length : 0}`);
      
      if (result.errors && result.errors.length > 0) {
        result.errors.forEach(error => {
          console.log(`  - ${error}`);
        });
      }
    });
  }
}
```

---

## 🛠️ Implementación Práctica

### **1. Hook de Instalación**

#### **useInstallation Hook**
```javascript
// useInstallation.js
import { useState, useEffect, useCallback } from 'react';

export const useInstallation = () => {
  const [isInstallable, setIsInstallable] = useState(false);
  const [isInstalled, setIsInstalled] = useState(false);
  const [installPrompt, setInstallPrompt] = useState(null);

  useEffect(() => {
    // Verificar si ya está instalada
    const checkInstalled = () => {
      const isStandalone = window.matchMedia('(display-mode: standalone)').matches;
      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
      const isInStandaloneMode = ('standalone' in window.navigator) && window.navigator.standalone;
      
      return isStandalone || (isIOS && isInStandaloneMode);
    };

    setIsInstalled(checkInstalled());

    // Escuchar evento beforeinstallprompt
    const handleBeforeInstallPrompt = (e) => {
      e.preventDefault();
      setInstallPrompt(e);
      setIsInstallable(true);
    };

    // Escuchar evento appinstalled
    const handleAppInstalled = () => {
      setIsInstalled(true);
      setIsInstallable(false);
      setInstallPrompt(null);
    };

    window.addEventListener('beforeinstallprompt', handleBeforeInstallPrompt);
    window.addEventListener('appinstalled', handleAppInstalled);

    return () => {
      window.removeEventListener('beforeinstallprompt', handleBeforeInstallPrompt);
      window.removeEventListener('appinstalled', handleAppInstalled);
    };
  }, []);

  const install = useCallback(async () => {
    if (!installPrompt) return false;

    try {
      const result = await installPrompt.prompt();
      const { outcome } = await installPrompt.userChoice;
      
      setInstallPrompt(null);
      setIsInstallable(false);
      
      return outcome === 'accepted';
    } catch (error) {
      console.error('Error durante la instalación:', error);
      return false;
    }
  }, [installPrompt]);

  return {
    isInstallable,
    isInstalled,
    install
  };
};
```

### **2. Componente de Instalación**

#### **InstallButton Component**
```javascript
// InstallButton.js
import React from 'react';
import { useInstallation } from './useInstallation';

const InstallButton = ({ className = '', children }) => {
  const { isInstallable, isInstalled, install } = useInstallation();

  if (isInstalled) {
    return (
      <div className={`install-status installed ${className}`}>
        ✅ Aplicación instalada
      </div>
    );
  }

  if (!isInstallable) {
    return null;
  }

  return (
    <button 
      onClick={install}
      className={`install-button ${className}`}
    >
      {children || '📱 Instalar App'}
    </button>
  );
};

export default InstallButton;
```

### **3. Testing de PWA**

#### **PWATestSuite Component**
```javascript
// PWATestSuite.js
import React, { useState } from 'react';
import { PWATesting } from './pwaTesting';

const PWATestSuite = () => {
  const [isRunning, setIsRunning] = useState(false);
  const [results, setResults] = useState(null);

  const runTests = async () => {
    setIsRunning(true);
    try {
      const testing = new PWATesting();
      const testResults = await testing.runAllTests();
      setResults(testResults);
    } catch (error) {
      console.error('Error ejecutando tests:', error);
    } finally {
      setIsRunning(false);
    }
  };

  const getScore = () => {
    if (!results) return 0;
    
    const totalTests = Object.keys(results).length;
    const passedTests = Object.values(results).filter(result => 
      result.errors && result.errors.length === 0
    ).length;
    
    return Math.round((passedTests / totalTests) * 100);
  };

  return (
    <div className="pwa-test-suite">
      <h2>🧪 Test Suite de PWA</h2>
      
      <button 
        onClick={runTests}
        disabled={isRunning}
        className="run-tests-button"
      >
        {isRunning ? 'Ejecutando tests...' : 'Ejecutar Tests'}
      </button>

      {results && (
        <div className="test-results">
          <div className="test-score">
            <h3>Puntuación: {getScore()}%</h3>
          </div>
          
          {Object.entries(results).map(([testName, result]) => (
            <div key={testName} className="test-result">
              <h4>{testName}</h4>
              <div className="test-status">
                {result.errors && result.errors.length === 0 ? '✅' : '❌'}
              </div>
              {result.errors && result.errors.length > 0 && (
                <ul className="test-errors">
                  {result.errors.map((error, index) => (
                    <li key={index}>{error}</li>
                  ))}
                </ul>
              )}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default PWATestSuite;
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Manifest Completo**
Crea un Web App Manifest que incluya:
- Todos los iconos necesarios
- Shortcuts funcionales
- Screenshots para app stores
- Configuración de display modes

### **Ejercicio 2: Install Prompts**
Implementa un sistema de install prompts que:
- Muestre beneficios de la instalación
- Maneje diferentes estados de instalación
- Proporcione analytics de instalación
- Personalice la experiencia según el usuario

### **Ejercicio 3: App-like Experience**
Optimiza la experiencia para que se sienta nativa:
- Oculte elementos del navegador
- Implemente gestos de navegación
- Cree splash screen personalizado
- Maneje status bar y navegación

---

## 📖 Recursos Adicionales

### **Documentación**
- [Web App Manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest)
- [PWA Installability](https://web.dev/installable/)
- [App Shortcuts](https://web.dev/app-shortcuts/)

### **Herramientas**
- [PWA Builder](https://www.pwabuilder.com/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [PWA Testing Tools](https://web.dev/pwa-testing/)

### **Ejemplos**
- [PWA Examples](https://github.com/GoogleChrome/samples/tree/gh-pages/pwa)
- [Install Prompts](https://github.com/GoogleChrome/samples/tree/gh-pages/pwa/installable)

---

## 🚀 Conclusión del Módulo

¡Felicidades! Has completado el **Módulo 22: PWA y Funcionalidades Web Avanzadas**. Ahora tienes todos los conocimientos necesarios para crear Progressive Web Apps completas y funcionales.

### **Lo que has aprendido:**
- ✅ Fundamentos de PWA y sus características
- ✅ Service Workers y estrategias de caching
- ✅ Arquitectura offline-first y sincronización
- ✅ Push notifications y engagement
- ✅ Instalación y experiencia app-like

### **Próximos pasos:**
- Implementa tu primera PWA completa
- Experimenta con diferentes estrategias de caching
- Optimiza la experiencia offline
- Desarrolla estrategias de engagement efectivas

---

**💡 Consejo**: Las PWA representan el futuro del desarrollo web. Continúa experimentando y mejorando tus aplicaciones para ofrecer la mejor experiencia posible a tus usuarios.

**🎯 Objetivo**: Al final de este módulo, eres capaz de crear PWA completas que rivalicen con aplicaciones nativas en funcionalidad y experiencia de usuario.
