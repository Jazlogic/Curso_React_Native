# 📱 Clase 4: Push Notifications Web

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar notificaciones push web, configurar VAPID keys para autenticación, crear notificaciones locales y programadas, y desarrollar estrategias de engagement para mantener a los usuarios conectados con tu PWA. Dominarás las técnicas para crear notificaciones efectivas y no intrusivas.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Implementar** notificaciones push web desde cero
2. **Configurar** VAPID keys y autenticación
3. **Crear** notificaciones locales y programadas
4. **Desarrollar** estrategias de engagement efectivas
5. **Manejar** permisos y preferencias de usuario

---

## 📚 Contenido de la Clase

### **1. Fundamentos de Push Notifications**

#### **¿Qué son las Push Notifications Web?**
Las notificaciones push web permiten que las aplicaciones web envíen mensajes a los usuarios incluso cuando la aplicación no está abierta, proporcionando una forma de re-enganchar usuarios y mantenerlos informados.

#### **Características Principales**
- **Entrega asíncrona**: Se envían sin que la app esté abierta
- **Persistencia**: Se muestran hasta que el usuario las interactúe
- **Personalización**: Contenido y acciones personalizables
- **Cross-platform**: Funcionan en múltiples dispositivos
- **Rich content**: Soporte para imágenes, acciones y sonidos

#### **Arquitectura de Push Notifications**
```javascript
// Arquitectura de Push Notifications
const PushNotificationArchitecture = {
  client: {
    subscription: 'PushSubscription',
    permission: 'Notification Permission',
    serviceWorker: 'Service Worker Registration'
  },
  server: {
    vapid: 'VAPID Keys',
    endpoint: 'Push Service Endpoint',
    payload: 'Notification Payload'
  },
  pushService: {
    fcm: 'Firebase Cloud Messaging',
    webpush: 'Web Push Protocol',
    delivery: 'Message Delivery'
  }
};
```

### **2. Configuración de VAPID Keys**

#### **¿Qué son las VAPID Keys?**
VAPID (Voluntary Application Server Identification) es un estándar que permite identificar el servidor que envía notificaciones push, proporcionando autenticación y autorización.

#### **Generación de VAPID Keys**
```javascript
// Generar VAPID keys
const webpush = require('web-push');

// Generar nuevas keys (solo una vez)
const vapidKeys = webpush.generateVAPIDKeys();

console.log('Public Key:', vapidKeys.publicKey);
console.log('Private Key:', vapidKeys.privateKey);

// Configurar VAPID details
webpush.setVapidDetails(
  'mailto:tu-email@ejemplo.com',
  vapidKeys.publicKey,
  vapidKeys.privateKey
);
```

#### **Configuración en el Cliente**
```javascript
// pushNotifications.js - Configuración del cliente
class PushNotificationManager {
  constructor() {
    this.vapidPublicKey = 'TU_VAPID_PUBLIC_KEY_AQUI';
    this.registration = null;
    this.subscription = null;
  }

  async init() {
    // Verificar soporte
    if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
      throw new Error('Push notifications no soportadas');
    }

    // Registrar Service Worker
    this.registration = await navigator.serviceWorker.register('/sw.js');
    
    // Obtener suscripción existente
    this.subscription = await this.registration.pushManager.getSubscription();
    
    return this.subscription;
  }

  // Solicitar permisos
  async requestPermission() {
    const permission = await Notification.requestPermission();
    
    if (permission === 'granted') {
      return true;
    } else if (permission === 'denied') {
      throw new Error('Permisos de notificación denegados');
    } else {
      throw new Error('Permisos de notificación no determinados');
    }
  }

  // Suscribirse a notificaciones push
  async subscribe() {
    if (!this.registration) {
      await this.init();
    }

    const permission = await this.requestPermission();
    
    if (permission) {
      this.subscription = await this.registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(this.vapidPublicKey)
      });

      // Enviar suscripción al servidor
      await this.sendSubscriptionToServer(this.subscription);
      
      return this.subscription;
    }
  }

  // Desuscribirse
  async unsubscribe() {
    if (this.subscription) {
      await this.subscription.unsubscribe();
      this.subscription = null;
      
      // Notificar al servidor
      await this.removeSubscriptionFromServer();
    }
  }

  // Convertir clave VAPID
  urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, '+')
      .replace(/_/g, '/');

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
  }

  // Enviar suscripción al servidor
  async sendSubscriptionToServer(subscription) {
    try {
      const response = await fetch('/api/subscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          subscription: subscription,
          userId: this.getUserId(),
          preferences: this.getUserPreferences()
        })
      });

      if (!response.ok) {
        throw new Error('Error enviando suscripción al servidor');
      }

      return await response.json();
    } catch (error) {
      console.error('Error enviando suscripción:', error);
      throw error;
    }
  }

  // Obtener ID de usuario
  getUserId() {
    // Implementar lógica para obtener ID de usuario
    return localStorage.getItem('userId') || 'anonymous';
  }

  // Obtener preferencias de usuario
  getUserPreferences() {
    return {
      categories: ['news', 'updates', 'reminders'],
      frequency: 'daily',
      quietHours: { start: '22:00', end: '08:00' }
    };
  }
}
```

### **3. Notificaciones Locales y Programadas**

#### **Notificaciones Locales**
```javascript
// notificacionesLocales.js - Notificaciones locales
class LocalNotifications {
  constructor() {
    this.permission = null;
    this.checkPermission();
  }

  async checkPermission() {
    this.permission = Notification.permission;
    return this.permission;
  }

  // Crear notificación local
  async createNotification(title, options = {}) {
    if (this.permission !== 'granted') {
      const permission = await Notification.requestPermission();
      if (permission !== 'granted') {
        throw new Error('Permisos de notificación no concedidos');
      }
    }

    const notification = new Notification(title, {
      body: options.body || '',
      icon: options.icon || '/icons/icon-192x192.png',
      badge: options.badge || '/icons/badge-72x72.png',
      image: options.image || null,
      tag: options.tag || 'default',
      requireInteraction: options.requireInteraction || false,
      silent: options.silent || false,
      vibrate: options.vibrate || [100, 50, 100],
      data: options.data || {},
      actions: options.actions || [],
      timestamp: Date.now()
    });

    // Manejar clicks
    notification.onclick = (event) => {
      event.preventDefault();
      this.handleNotificationClick(notification, event);
    };

    // Auto-cerrar después de 5 segundos (si no requiere interacción)
    if (!options.requireInteraction) {
      setTimeout(() => {
        notification.close();
      }, 5000);
    }

    return notification;
  }

  // Manejar click en notificación
  handleNotificationClick(notification, event) {
    // Abrir la aplicación
    window.focus();
    
    // Navegar a URL específica si está en los datos
    if (notification.data && notification.data.url) {
      window.location.href = notification.data.url;
    }

    // Cerrar la notificación
    notification.close();
  }

  // Notificación con acciones
  async createActionableNotification(title, body, actions) {
    return this.createNotification(title, {
      body,
      requireInteraction: true,
      actions: actions.map(action => ({
        action: action.id,
        title: action.title,
        icon: action.icon
      })),
      data: { actions: actions }
    });
  }

  // Notificación con imagen
  async createImageNotification(title, body, imageUrl) {
    return this.createNotification(title, {
      body,
      image: imageUrl,
      requireInteraction: true
    });
  }
}
```

#### **Notificaciones Programadas**
```javascript
// notificacionesProgramadas.js - Notificaciones programadas
class ScheduledNotifications {
  constructor() {
    this.scheduledNotifications = new Map();
    this.loadScheduledNotifications();
  }

  // Programar notificación
  scheduleNotification(title, options, triggerTime) {
    const notificationId = this.generateId();
    
    const scheduledNotification = {
      id: notificationId,
      title,
      options,
      triggerTime,
      scheduled: true
    };

    // Calcular delay
    const delay = triggerTime.getTime() - Date.now();
    
    if (delay > 0) {
      const timeoutId = setTimeout(() => {
        this.showScheduledNotification(scheduledNotification);
        this.scheduledNotifications.delete(notificationId);
        this.saveScheduledNotifications();
      }, delay);

      scheduledNotification.timeoutId = timeoutId;
      this.scheduledNotifications.set(notificationId, scheduledNotification);
      this.saveScheduledNotifications();
    }

    return notificationId;
  }

  // Mostrar notificación programada
  async showScheduledNotification(scheduledNotification) {
    const localNotifications = new LocalNotifications();
    
    try {
      await localNotifications.createNotification(
        scheduledNotification.title,
        scheduledNotification.options
      );
    } catch (error) {
      console.error('Error mostrando notificación programada:', error);
    }
  }

  // Cancelar notificación programada
  cancelScheduledNotification(notificationId) {
    const scheduledNotification = this.scheduledNotifications.get(notificationId);
    
    if (scheduledNotification) {
      clearTimeout(scheduledNotification.timeoutId);
      this.scheduledNotifications.delete(notificationId);
      this.saveScheduledNotifications();
      return true;
    }
    
    return false;
  }

  // Programar notificación recurrente
  scheduleRecurringNotification(title, options, interval) {
    const scheduleNext = () => {
      const nextTime = new Date(Date.now() + interval);
      const notificationId = this.scheduleNotification(title, options, nextTime);
      
      // Programar la siguiente
      setTimeout(() => {
        scheduleNext();
      }, interval);
    };

    // Programar la primera
    scheduleNext();
  }

  // Cargar notificaciones programadas desde localStorage
  loadScheduledNotifications() {
    try {
      const saved = localStorage.getItem('scheduledNotifications');
      if (saved) {
        const notifications = JSON.parse(saved);
        
        notifications.forEach(notification => {
          const triggerTime = new Date(notification.triggerTime);
          const delay = triggerTime.getTime() - Date.now();
          
          if (delay > 0) {
            const timeoutId = setTimeout(() => {
              this.showScheduledNotification(notification);
            }, delay);
            
            notification.timeoutId = timeoutId;
            this.scheduledNotifications.set(notification.id, notification);
          }
        });
      }
    } catch (error) {
      console.error('Error cargando notificaciones programadas:', error);
    }
  }

  // Guardar notificaciones programadas en localStorage
  saveScheduledNotifications() {
    const notifications = Array.from(this.scheduledNotifications.values())
      .map(notification => ({
        ...notification,
        timeoutId: undefined // No guardar timeoutId
      }));
    
    localStorage.setItem('scheduledNotifications', JSON.stringify(notifications));
  }

  // Generar ID único
  generateId() {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
}
```

### **4. Estrategias de Engagement**

#### **Sistema de Engagement**
```javascript
// engagementStrategies.js - Estrategias de engagement
class EngagementStrategies {
  constructor() {
    this.userBehavior = this.loadUserBehavior();
    this.notificationHistory = this.loadNotificationHistory();
  }

  // Analizar comportamiento del usuario
  analyzeUserBehavior() {
    const now = Date.now();
    const lastActivity = this.userBehavior.lastActivity || now;
    const timeSinceLastActivity = now - lastActivity;
    
    return {
      isActive: timeSinceLastActivity < 24 * 60 * 60 * 1000, // 24 horas
      engagementLevel: this.calculateEngagementLevel(),
      preferredTimes: this.getPreferredTimes(),
      responseRate: this.calculateResponseRate()
    };
  }

  // Calcular nivel de engagement
  calculateEngagementLevel() {
    const recentNotifications = this.notificationHistory
      .filter(n => Date.now() - n.timestamp < 7 * 24 * 60 * 60 * 1000) // 7 días
      .length;
    
    const recentClicks = this.notificationHistory
      .filter(n => n.clicked && Date.now() - n.timestamp < 7 * 24 * 60 * 60 * 1000)
      .length;
    
    const clickRate = recentNotifications > 0 ? recentClicks / recentNotifications : 0;
    
    if (clickRate > 0.3) return 'high';
    if (clickRate > 0.1) return 'medium';
    return 'low';
  }

  // Obtener horarios preferidos
  getPreferredTimes() {
    const clickTimes = this.notificationHistory
      .filter(n => n.clicked)
      .map(n => new Date(n.timestamp).getHours());
    
    // Calcular horas más activas
    const hourCounts = {};
    clickTimes.forEach(hour => {
      hourCounts[hour] = (hourCounts[hour] || 0) + 1;
    });
    
    return Object.keys(hourCounts)
      .sort((a, b) => hourCounts[b] - hourCounts[a])
      .slice(0, 3)
      .map(hour => parseInt(hour));
  }

  // Calcular tasa de respuesta
  calculateResponseRate() {
    const total = this.notificationHistory.length;
    const clicked = this.notificationHistory.filter(n => n.clicked).length;
    
    return total > 0 ? clicked / total : 0;
  }

  // Determinar si enviar notificación
  shouldSendNotification(type, content) {
    const behavior = this.analyzeUserBehavior();
    
    // No enviar si el usuario está muy activo
    if (behavior.isActive && behavior.engagementLevel === 'high') {
      return false;
    }
    
    // No enviar si la tasa de respuesta es muy baja
    if (behavior.responseRate < 0.05) {
      return false;
    }
    
    // Verificar horarios preferidos
    const currentHour = new Date().getHours();
    if (!behavior.preferredTimes.includes(currentHour)) {
      return false;
    }
    
    // Verificar frecuencia
    if (this.isTooFrequent(type)) {
      return false;
    }
    
    return true;
  }

  // Verificar si es muy frecuente
  isTooFrequent(type) {
    const recent = this.notificationHistory
      .filter(n => n.type === type && Date.now() - n.timestamp < 60 * 60 * 1000) // 1 hora
      .length;
    
    return recent >= 3; // Máximo 3 por hora por tipo
  }

  // Registrar notificación enviada
  recordNotificationSent(notification) {
    this.notificationHistory.push({
      id: notification.id,
      type: notification.type,
      timestamp: Date.now(),
      clicked: false,
      dismissed: false
    });
    
    this.saveNotificationHistory();
  }

  // Registrar click en notificación
  recordNotificationClick(notificationId) {
    const notification = this.notificationHistory.find(n => n.id === notificationId);
    if (notification) {
      notification.clicked = true;
      this.saveNotificationHistory();
    }
  }

  // Cargar comportamiento del usuario
  loadUserBehavior() {
    try {
      const saved = localStorage.getItem('userBehavior');
      return saved ? JSON.parse(saved) : { lastActivity: Date.now() };
    } catch (error) {
      return { lastActivity: Date.now() };
    }
  }

  // Guardar comportamiento del usuario
  saveUserBehavior() {
    localStorage.setItem('userBehavior', JSON.stringify(this.userBehavior));
  }

  // Cargar historial de notificaciones
  loadNotificationHistory() {
    try {
      const saved = localStorage.getItem('notificationHistory');
      return saved ? JSON.parse(saved) : [];
    } catch (error) {
      return [];
    }
  }

  // Guardar historial de notificaciones
  saveNotificationHistory() {
    // Mantener solo los últimos 30 días
    const thirtyDaysAgo = Date.now() - 30 * 24 * 60 * 60 * 1000;
    this.notificationHistory = this.notificationHistory
      .filter(n => n.timestamp > thirtyDaysAgo);
    
    localStorage.setItem('notificationHistory', JSON.stringify(this.notificationHistory));
  }
}
```

### **5. Manejo de Permisos y Preferencias**

#### **Sistema de Permisos**
```javascript
// permissionManager.js - Manejo de permisos
class PermissionManager {
  constructor() {
    this.permissions = {
      notifications: 'default',
      push: false
    };
    this.preferences = this.loadPreferences();
  }

  // Verificar permisos
  async checkPermissions() {
    this.permissions.notifications = Notification.permission;
    this.permissions.push = await this.checkPushPermission();
    
    return this.permissions;
  }

  // Verificar permiso de push
  async checkPushPermission() {
    if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
      return false;
    }

    try {
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.getSubscription();
      return !!subscription;
    } catch (error) {
      return false;
    }
  }

  // Solicitar permisos de manera contextual
  async requestPermissions(context) {
    const results = {
      notifications: false,
      push: false
    };

    // Solicitar permiso de notificaciones
    if (this.permissions.notifications === 'default') {
      const notificationPermission = await this.requestNotificationPermission(context);
      results.notifications = notificationPermission === 'granted';
    }

    // Solicitar permiso de push si las notificaciones están permitidas
    if (results.notifications && !this.permissions.push) {
      results.push = await this.requestPushPermission();
    }

    return results;
  }

  // Solicitar permiso de notificaciones con contexto
  async requestNotificationPermission(context) {
    // Mostrar explicación contextual
    const explanation = this.getPermissionExplanation(context);
    
    if (explanation) {
      const userConfirmed = await this.showPermissionDialog(explanation);
      if (!userConfirmed) {
        return 'denied';
      }
    }

    return await Notification.requestPermission();
  }

  // Obtener explicación contextual
  getPermissionExplanation(context) {
    const explanations = {
      'welcome': {
        title: 'Mantente informado',
        message: 'Te enviaremos notificaciones sobre actualizaciones importantes y contenido nuevo.',
        benefits: ['Nunca te pierdas contenido importante', 'Recibe recordatorios personalizados']
      },
      'feature': {
        title: 'Notificaciones de la función',
        message: 'Esta función funciona mejor con notificaciones habilitadas.',
        benefits: ['Mejor experiencia de usuario', 'Funcionalidad completa']
      },
      'engagement': {
        title: 'Mantente conectado',
        message: 'Recibe notificaciones cuando tengas mensajes o actualizaciones.',
        benefits: ['Respuestas más rápidas', 'Mejor comunicación']
      }
    };

    return explanations[context] || null;
  }

  // Mostrar diálogo de permisos
  async showPermissionDialog(explanation) {
    return new Promise((resolve) => {
      const dialog = document.createElement('div');
      dialog.className = 'permission-dialog';
      dialog.innerHTML = `
        <div class="permission-content">
          <h3>${explanation.title}</h3>
          <p>${explanation.message}</p>
          <ul>
            ${explanation.benefits.map(benefit => `<li>${benefit}</li>`).join('')}
          </ul>
          <div class="permission-actions">
            <button class="permission-deny">No, gracias</button>
            <button class="permission-allow">Permitir</button>
          </div>
        </div>
      `;

      document.body.appendChild(dialog);

      // Manejar clicks
      dialog.querySelector('.permission-allow').onclick = () => {
        document.body.removeChild(dialog);
        resolve(true);
      };

      dialog.querySelector('.permission-deny').onclick = () => {
        document.body.removeChild(dialog);
        resolve(false);
      };
    });
  }

  // Solicitar permiso de push
  async requestPushPermission() {
    try {
      const pushManager = new PushNotificationManager();
      await pushManager.subscribe();
      return true;
    } catch (error) {
      console.error('Error solicitando permiso de push:', error);
      return false;
    }
  }

  // Cargar preferencias
  loadPreferences() {
    try {
      const saved = localStorage.getItem('notificationPreferences');
      return saved ? JSON.parse(saved) : {
        categories: ['news', 'updates'],
        frequency: 'daily',
        quietHours: { start: '22:00', end: '08:00' },
        sound: true,
        vibration: true
      };
    } catch (error) {
      return {
        categories: ['news', 'updates'],
        frequency: 'daily',
        quietHours: { start: '22:00', end: '08:00' },
        sound: true,
        vibration: true
      };
    }
  }

  // Guardar preferencias
  savePreferences(preferences) {
    this.preferences = { ...this.preferences, ...preferences };
    localStorage.setItem('notificationPreferences', JSON.stringify(this.preferences));
  }
}
```

---

## 🛠️ Implementación Práctica

### **1. Hook de Push Notifications**

#### **usePushNotifications Hook**
```javascript
// usePushNotifications.js
import { useState, useEffect, useCallback } from 'react';
import { PushNotificationManager } from './pushNotifications';
import { LocalNotifications } from './notificacionesLocales';
import { ScheduledNotifications } from './notificacionesProgramadas';
import { EngagementStrategies } from './engagementStrategies';
import { PermissionManager } from './permissionManager';

export const usePushNotifications = () => {
  const [pushManager, setPushManager] = useState(null);
  const [localNotifications, setLocalNotifications] = useState(null);
  const [scheduledNotifications, setScheduledNotifications] = useState(null);
  const [engagement, setEngagement] = useState(null);
  const [permissionManager, setPermissionManager] = useState(null);
  const [permissions, setPermissions] = useState({
    notifications: 'default',
    push: false
  });
  const [isSubscribed, setIsSubscribed] = useState(false);

  useEffect(() => {
    const initPushNotifications = async () => {
      try {
        const pushMgr = new PushNotificationManager();
        const localNotif = new LocalNotifications();
        const scheduledNotif = new ScheduledNotifications();
        const engagementStrat = new EngagementStrategies();
        const permMgr = new PermissionManager();

        await pushMgr.init();
        
        setPushManager(pushMgr);
        setLocalNotifications(localNotif);
        setScheduledNotifications(scheduledNotif);
        setEngagement(engagementStrat);
        setPermissionManager(permMgr);

        // Verificar permisos
        const currentPermissions = await permMgr.checkPermissions();
        setPermissions(currentPermissions);
        setIsSubscribed(!!pushMgr.subscription);
      } catch (error) {
        console.error('Error inicializando push notifications:', error);
      }
    };

    initPushNotifications();
  }, []);

  // Suscribirse a notificaciones push
  const subscribe = useCallback(async (context = 'welcome') => {
    if (!pushManager || !permissionManager) return false;

    try {
      // Solicitar permisos
      const permissionResults = await permissionManager.requestPermissions(context);
      
      if (permissionResults.push) {
        await pushManager.subscribe();
        setIsSubscribed(true);
        setPermissions(await permissionManager.checkPermissions());
        return true;
      }
      
      return false;
    } catch (error) {
      console.error('Error suscribiéndose:', error);
      return false;
    }
  }, [pushManager, permissionManager]);

  // Desuscribirse
  const unsubscribe = useCallback(async () => {
    if (!pushManager) return false;

    try {
      await pushManager.unsubscribe();
      setIsSubscribed(false);
      setPermissions(await permissionManager.checkPermissions());
      return true;
    } catch (error) {
      console.error('Error desuscribiéndose:', error);
      return false;
    }
  }, [pushManager, permissionManager]);

  // Enviar notificación local
  const sendLocalNotification = useCallback(async (title, options) => {
    if (!localNotifications) return false;

    try {
      await localNotifications.createNotification(title, options);
      return true;
    } catch (error) {
      console.error('Error enviando notificación local:', error);
      return false;
    }
  }, [localNotifications]);

  // Programar notificación
  const scheduleNotification = useCallback((title, options, triggerTime) => {
    if (!scheduledNotifications) return null;

    return scheduledNotifications.scheduleNotification(title, options, triggerTime);
  }, [scheduledNotifications]);

  // Verificar si debe enviar notificación
  const shouldSendNotification = useCallback((type, content) => {
    if (!engagement) return true;

    return engagement.shouldSendNotification(type, content);
  }, [engagement]);

  return {
    permissions,
    isSubscribed,
    subscribe,
    unsubscribe,
    sendLocalNotification,
    scheduleNotification,
    shouldSendNotification
  };
};
```

### **2. Componente de Configuración de Notificaciones**

#### **NotificationSettings Component**
```javascript
// NotificationSettings.js
import React, { useState, useEffect } from 'react';
import { usePushNotifications } from './usePushNotifications';

const NotificationSettings = () => {
  const {
    permissions,
    isSubscribed,
    subscribe,
    unsubscribe
  } = usePushNotifications();

  const [preferences, setPreferences] = useState({
    categories: ['news', 'updates'],
    frequency: 'daily',
    quietHours: { start: '22:00', end: '08:00' },
    sound: true,
    vibration: true
  });

  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    // Cargar preferencias desde localStorage
    const saved = localStorage.getItem('notificationPreferences');
    if (saved) {
      setPreferences(JSON.parse(saved));
    }
  }, []);

  const handleSubscribe = async () => {
    setIsLoading(true);
    try {
      const success = await subscribe('feature');
      if (success) {
        alert('¡Notificaciones habilitadas!');
      } else {
        alert('No se pudieron habilitar las notificaciones');
      }
    } catch (error) {
      alert('Error habilitando notificaciones');
    } finally {
      setIsLoading(false);
    }
  };

  const handleUnsubscribe = async () => {
    setIsLoading(true);
    try {
      const success = await unsubscribe();
      if (success) {
        alert('Notificaciones deshabilitadas');
      }
    } catch (error) {
      alert('Error deshabilitando notificaciones');
    } finally {
      setIsLoading(false);
    }
  };

  const handlePreferenceChange = (key, value) => {
    const newPreferences = { ...preferences, [key]: value };
    setPreferences(newPreferences);
    localStorage.setItem('notificationPreferences', JSON.stringify(newPreferences));
  };

  const handleCategoryToggle = (category) => {
    const newCategories = preferences.categories.includes(category)
      ? preferences.categories.filter(c => c !== category)
      : [...preferences.categories, category];
    
    handlePreferenceChange('categories', newCategories);
  };

  return (
    <div className="notification-settings">
      <h2>Configuración de Notificaciones</h2>
      
      {/* Estado de permisos */}
      <div className="permission-status">
        <h3>Estado de Permisos</h3>
        <div className="status-item">
          <span>Notificaciones:</span>
          <span className={`status ${permissions.notifications}`}>
            {permissions.notifications}
          </span>
        </div>
        <div className="status-item">
          <span>Push:</span>
          <span className={`status ${permissions.push ? 'granted' : 'denied'}`}>
            {permissions.push ? 'Habilitado' : 'Deshabilitado'}
          </span>
        </div>
      </div>

      {/* Suscripción */}
      <div className="subscription-section">
        <h3>Suscripción</h3>
        {isSubscribed ? (
          <button 
            onClick={handleUnsubscribe}
            disabled={isLoading}
            className="unsubscribe-button"
          >
            {isLoading ? 'Deshabilitando...' : 'Deshabilitar Notificaciones'}
          </button>
        ) : (
          <button 
            onClick={handleSubscribe}
            disabled={isLoading}
            className="subscribe-button"
          >
            {isLoading ? 'Habilitando...' : 'Habilitar Notificaciones'}
          </button>
        )}
      </div>

      {/* Preferencias */}
      <div className="preferences-section">
        <h3>Preferencias</h3>
        
        {/* Categorías */}
        <div className="preference-group">
          <label>Categorías:</label>
          <div className="category-options">
            {['news', 'updates', 'reminders', 'promotions'].map(category => (
              <label key={category} className="category-option">
                <input
                  type="checkbox"
                  checked={preferences.categories.includes(category)}
                  onChange={() => handleCategoryToggle(category)}
                />
                {category}
              </label>
            ))}
          </div>
        </div>

        {/* Frecuencia */}
        <div className="preference-group">
          <label>Frecuencia:</label>
          <select
            value={preferences.frequency}
            onChange={(e) => handlePreferenceChange('frequency', e.target.value)}
          >
            <option value="immediate">Inmediata</option>
            <option value="daily">Diaria</option>
            <option value="weekly">Semanal</option>
          </select>
        </div>

        {/* Horarios silenciosos */}
        <div className="preference-group">
          <label>Horarios Silenciosos:</label>
          <div className="quiet-hours">
            <input
              type="time"
              value={preferences.quietHours.start}
              onChange={(e) => handlePreferenceChange('quietHours', {
                ...preferences.quietHours,
                start: e.target.value
              })}
            />
            <span>a</span>
            <input
              type="time"
              value={preferences.quietHours.end}
              onChange={(e) => handlePreferenceChange('quietHours', {
                ...preferences.quietHours,
                end: e.target.value
              })}
            />
          </div>
        </div>

        {/* Sonido y vibración */}
        <div className="preference-group">
          <label>
            <input
              type="checkbox"
              checked={preferences.sound}
              onChange={(e) => handlePreferenceChange('sound', e.target.checked)}
            />
            Sonido
          </label>
        </div>

        <div className="preference-group">
          <label>
            <input
              type="checkbox"
              checked={preferences.vibration}
              onChange={(e) => handlePreferenceChange('vibration', e.target.checked)}
            />
            Vibración
          </label>
        </div>
      </div>
    </div>
  );
};

export default NotificationSettings;
```

### **3. Testing de Push Notifications**

#### **Testing de Notificaciones**
```javascript
// pushNotifications.test.js
import { PushNotificationManager } from './pushNotifications';
import { LocalNotifications } from './notificacionesLocales';

// Mock de APIs
const mockNotification = {
  close: jest.fn(),
  onclick: null
};

global.Notification = jest.fn().mockImplementation(() => mockNotification);
global.Notification.requestPermission = jest.fn();
global.Notification.permission = 'default';

const mockServiceWorker = {
  register: jest.fn(),
  ready: Promise.resolve({
    pushManager: {
      getSubscription: jest.fn(),
      subscribe: jest.fn()
    }
  })
};

Object.defineProperty(navigator, 'serviceWorker', {
  value: mockServiceWorker,
  writable: true
});

describe('Push Notifications', () => {
  let pushManager;
  let localNotifications;

  beforeEach(() => {
    jest.clearAllMocks();
    pushManager = new PushNotificationManager();
    localNotifications = new LocalNotifications();
  });

  test('debe inicializar correctamente', async () => {
    await pushManager.init();
    expect(pushManager.registration).toBeDefined();
  });

  test('debe solicitar permisos correctamente', async () => {
    global.Notification.requestPermission.mockResolvedValue('granted');
    
    const result = await pushManager.requestPermission();
    expect(result).toBe(true);
    expect(global.Notification.requestPermission).toHaveBeenCalled();
  });

  test('debe crear notificación local', async () => {
    global.Notification.requestPermission.mockResolvedValue('granted');
    
    const notification = await localNotifications.createNotification(
      'Test Title',
      { body: 'Test Body' }
    );
    
    expect(global.Notification).toHaveBeenCalledWith('Test Title', {
      body: 'Test Body'
    });
  });

  test('debe manejar permisos denegados', async () => {
    global.Notification.requestPermission.mockResolvedValue('denied');
    
    await expect(pushManager.requestPermission()).rejects.toThrow(
      'Permisos de notificación denegados'
    );
  });
});
```

---

## 🎯 Ejercicios Prácticos

### **Ejercicio 1: Sistema de Notificaciones Básico**
Implementa un sistema de notificaciones que:
- Solicite permisos de manera contextual
- Envíe notificaciones locales
- Maneje clicks en notificaciones
- Proporcione configuración de preferencias

### **Ejercicio 2: Notificaciones Programadas**
Crea un sistema de notificaciones programadas que:
- Programe notificaciones para fechas futuras
- Maneje notificaciones recurrentes
- Persista notificaciones entre sesiones
- Permita cancelar notificaciones programadas

### **Ejercicio 3: Estrategias de Engagement**
Implementa estrategias de engagement que:
- Analicen el comportamiento del usuario
- Determinen el mejor momento para enviar notificaciones
- Personalicen el contenido según las preferencias
- Muestren métricas de engagement

---

## 📖 Recursos Adicionales

### **Documentación**
- [Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)
- [Notifications API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
- [VAPID](https://tools.ietf.org/html/rfc8292)

### **Herramientas**
- [Web Push Protocol](https://tools.ietf.org/html/rfc8030)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [OneSignal](https://onesignal.com/) - Servicio de notificaciones

### **Ejemplos**
- [Web Push Examples](https://github.com/GoogleChrome/samples/tree/gh-pages/push-messaging)
- [Notification Examples](https://github.com/GoogleChrome/samples/tree/gh-pages/notifications)

---

## 🚀 Próximos Pasos

En la siguiente clase aprenderás sobre **Instalación y App-like Experience**, donde implementarás Web App Manifest, install prompts, y optimizaciones para crear una experiencia similar a aplicaciones nativas.

---

**💡 Consejo**: Las notificaciones push son poderosas pero deben usarse con responsabilidad. Siempre respeta las preferencias del usuario y proporciona valor real en cada notificación.

**🎯 Objetivo**: Al final de esta clase, tendrás un sistema completo de notificaciones push que mejore el engagement y mantenga a los usuarios conectados con tu PWA.
