# Clase 4: Notificaciones Push y Locales 🔔

## Objetivos de la Clase
- Implementar notificaciones push con Firebase Cloud Messaging
- Crear notificaciones locales programadas
- Manejar badges y contadores de notificaciones
- Implementar deep linking para navegación
- Crear un sistema de gestión de notificaciones

## Duración Estimada
**2 horas**

## Contenido Teórico

### 1. Notificaciones Push con Firebase

Para implementar notificaciones push, necesitamos configurar Firebase Cloud Messaging (FCM):

```javascript
import messaging from '@react-native-firebase/messaging';
import { Platform, Alert, PermissionsAndroid } from 'react-native';

class PushNotificationManager {
  // Solicitar permiso para notificaciones push
  static async requestPermission() {
    if (Platform.OS === 'ios') {
      const authStatus = await messaging().requestPermission();
      const enabled =
        authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
        authStatus === messaging.AuthorizationStatus.PROVISIONAL;

      if (enabled) {
        console.log('Permiso de notificaciones push concedido');
        return true;
      } else {
        console.log('Permiso de notificaciones push denegado');
        return false;
      }
    }
    
    // Android no requiere permiso explícito para notificaciones push
    return true;
  }

  // Obtener token FCM del dispositivo
  static async getFCMToken() {
    try {
      const token = await messaging().getToken();
      console.log('Token FCM:', token);
      return token;
    } catch (error) {
      console.error('Error al obtener token FCM:', error);
      return null;
    }
  }

  // Suscribirse a un tema específico
  static async subscribeToTopic(topic) {
    try {
      await messaging().subscribeToTopic(topic);
      console.log(`Suscrito al tema: ${topic}`);
      return true;
    } catch (error) {
      console.error(`Error al suscribirse al tema ${topic}:`, error);
      return false;
    }
  }

  // Desuscribirse de un tema
  static async unsubscribeFromTopic(topic) {
    try {
      await messaging().unsubscribeFromTopic(topic);
      console.log(`Desuscrito del tema: ${topic}`);
      return true;
    } catch (error) {
      console.error(`Error al desuscribirse del tema ${topic}:`, error);
      return false;
    }
  }

  // Configurar manejadores de notificaciones
  static setupNotificationHandlers() {
    // Notificación recibida cuando la app está en primer plano
    const unsubscribeForeground = messaging().onMessage(async (remoteMessage) => {
      console.log('Notificación recibida en primer plano:', remoteMessage);
      
      // Mostrar notificación local personalizada
      this.showLocalNotification({
        title: remoteMessage.notification?.title || 'Nueva notificación',
        body: remoteMessage.notification?.body || 'Tienes un nuevo mensaje',
        data: remoteMessage.data,
      });
    });

    // Notificación abierta desde estado cerrado
    const unsubscribeBackground = messaging().onNotificationOpenedApp((remoteMessage) => {
      console.log('App abierta desde notificación en segundo plano:', remoteMessage);
      this.handleNotificationNavigation(remoteMessage);
    });

    // Verificar si la app fue abierta desde una notificación
    messaging()
      .getInitialNotification()
      .then((remoteMessage) => {
        if (remoteMessage) {
          console.log('App abierta desde notificación inicial:', remoteMessage);
          this.handleNotificationNavigation(remoteMessage);
        }
      });

    // Retornar función para limpiar suscripciones
    return () => {
      unsubscribeForeground();
      unsubscribeBackground();
    };
  }

  // Manejar navegación desde notificación
  static handleNotificationNavigation(remoteMessage) {
    const { data } = remoteMessage;
    
    if (data?.screen) {
      // Navegar a la pantalla específica
      // navigation.navigate(data.screen, data.params);
      console.log(`Navegando a: ${data.screen}`);
    }
  }

  // Enviar token al servidor
  static async sendTokenToServer(token, userId) {
    try {
      const response = await fetch('https://tu-api.com/fcm-tokens', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          userId,
          token,
          platform: Platform.OS,
        }),
      });
      
      if (response.ok) {
        console.log('Token enviado al servidor correctamente');
        return true;
      } else {
        console.error('Error al enviar token al servidor');
        return false;
      }
    } catch (error) {
      console.error('Error de red al enviar token:', error);
      return false;
    }
  }
}
```

### 2. Notificaciones Locales

Las notificaciones locales se muestran incluso cuando la app está cerrada:

```javascript
import PushNotification from 'react-native-push-notification';
import { Platform } from 'react-native';

class LocalNotificationManager {
  // Configurar el canal de notificaciones (Android)
  static configure() {
    PushNotification.configure({
      // Configuración básica
      onRegister: function (token) {
        console.log('TOKEN:', token);
      },
      
      onNotification: function (notification) {
        console.log('NOTIFICATION:', notification);
        // Manejar la notificación local
        this.handleLocalNotification(notification);
      },
      
      // Permisos
      permissions: {
        alert: true,
        badge: true,
        sound: true,
      },
      
      // Pop initial notification
      popInitialNotification: true,
      
      // Request permissions on iOS
      requestPermissions: Platform.OS === 'ios',
    });

    // Crear canal para Android
    if (Platform.OS === 'android') {
      PushNotification.createChannel(
        {
          channelId: 'default-channel',
          channelName: 'Canal por defecto',
          channelDescription: 'Canal para notificaciones generales',
          playSound: true,
          soundName: 'default',
          importance: 4, // IMPORTANCE_HIGH
          vibrate: true,
        },
        (created) => console.log(`Canal creado: ${created}`)
      );
    }
  }

  // Mostrar notificación local inmediata
  static showNotification(title, message, data = {}) {
    PushNotification.localNotification({
      channelId: 'default-channel',
      title: title,
      message: message,
      data: data,
      playSound: true,
      soundName: 'default',
      importance: 'high',
      priority: 'high',
      vibrate: true,
      vibration: 300,
      autoCancel: true,
      largeIcon: 'ic_launcher',
      smallIcon: 'ic_notification',
      bigText: message,
      subText: 'Toca para ver más',
      color: '#2196F3',
      number: 10, // Badge count
    });
  }

  // Programar notificación local
  static scheduleNotification(title, message, date, data = {}) {
    PushNotification.localNotificationSchedule({
      channelId: 'default-channel',
      title: title,
      message: message,
      date: date,
      data: data,
      playSound: true,
      soundName: 'default',
      importance: 'high',
      priority: 'high',
      vibrate: true,
      vibration: 300,
      autoCancel: true,
      largeIcon: 'ic_launcher',
      smallIcon: 'ic_notification',
      bigText: message,
      subText: 'Notificación programada',
      color: '#4CAF50',
      repeatType: 'day', // 'week', 'day', 'hour', 'minute'
    });
  }

  // Cancelar todas las notificaciones programadas
  static cancelAllNotifications() {
    PushNotification.cancelAllLocalNotifications();
  }

  // Cancelar notificación específica por ID
  static cancelNotification(notificationId) {
    PushNotification.cancelLocalNotification(notificationId);
  }

  // Obtener notificaciones programadas
  static getScheduledNotifications() {
    return new Promise((resolve) => {
      PushNotification.getScheduledLocalNotifications((notifications) => {
        resolve(notifications);
      });
    });
  }

  // Manejar notificación local
  static handleLocalNotification(notification) {
    const { data } = notification;
    
    if (data?.action) {
      switch (data.action) {
        case 'open_screen':
          // Navegar a pantalla específica
          console.log(`Abriendo pantalla: ${data.screen}`);
          break;
        case 'show_alert':
          // Mostrar alerta
          Alert.alert('Notificación', data.message);
          break;
        default:
          console.log('Acción de notificación no reconocida:', data.action);
      }
    }
  }
}
```

### 3. Sistema de Badges y Contadores

```javascript
import PushNotification from 'react-native-push-notification';
import { Platform } from 'react-native';

class BadgeManager {
  // Establecer número de badge
  static setBadgeNumber(number) {
    if (Platform.OS === 'ios') {
      PushNotification.setApplicationIconBadgeNumber(number);
    } else {
      // Android maneja badges automáticamente
      console.log(`Badge establecido en: ${number}`);
    }
  }

  // Obtener número de badge actual
  static getBadgeNumber() {
    if (Platform.OS === 'ios') {
      return PushNotification.getApplicationIconBadgeNumber();
    }
    return 0; // Android no expone esta información
  }

  // Incrementar badge
  static incrementBadge() {
    const currentBadge = this.getBadgeNumber();
    this.setBadgeNumber(currentBadge + 1);
  }

  // Decrementar badge
  static decrementBadge() {
    const currentBadge = this.getBadgeNumber();
    if (currentBadge > 0) {
      this.setBadgeNumber(currentBadge - 1);
    }
  }

  // Limpiar badge
  static clearBadge() {
    this.setBadgeNumber(0);
  }

  // Actualizar badge basado en notificaciones no leídas
  static async updateBadgeFromUnreadCount() {
    try {
      // Obtener notificaciones no leídas del servidor
      const response = await fetch('https://tu-api.com/notifications/unread-count');
      const { count } = await response.json();
      
      this.setBadgeNumber(count);
      return count;
    } catch (error) {
      console.error('Error al actualizar badge:', error);
      return 0;
    }
  }
}
```

### 4. Deep Linking y Navegación

```javascript
import { Linking } from 'react-native';
import { navigationRef } from './NavigationRef';

class DeepLinkManager {
  // Configurar deep linking
  static configure() {
    // Manejar enlaces cuando la app está abierta
    const handleDeepLink = (url) => {
      this.handleDeepLink(url);
    };

    // Escuchar enlaces entrantes
    Linking.addEventListener('url', handleDeepLink);

    // Verificar si la app fue abierta desde un enlace
    Linking.getInitialURL().then((url) => {
      if (url) {
        this.handleDeepLink(url);
      }
    });

    // Retornar función de limpieza
    return () => {
      Linking.removeAllListeners('url');
    };
  }

  // Manejar deep link
  static handleDeepLink(url) {
    console.log('Deep link recibido:', url);
    
    try {
      const parsedUrl = new URL(url);
      const { hostname, pathname, searchParams } = parsedUrl;
      
      // Estructura esperada: myapp://screen/params
      const pathSegments = pathname.split('/').filter(Boolean);
      
      if (pathSegments.length > 0) {
        const screen = pathSegments[0];
        const params = {};
        
        // Convertir parámetros de búsqueda a objeto
        searchParams.forEach((value, key) => {
          params[key] = value;
        });
        
        // Navegar a la pantalla
        this.navigateToScreen(screen, params);
      }
    } catch (error) {
      console.error('Error al procesar deep link:', error);
    }
  }

  // Navegar a pantalla específica
  static navigateToScreen(screen, params = {}) {
    if (navigationRef.current) {
      try {
        navigationRef.current.navigate(screen, params);
        console.log(`Navegando a ${screen} con params:`, params);
      } catch (error) {
        console.error(`Error al navegar a ${screen}:`, error);
      }
    } else {
      console.warn('Navigation ref no está disponible');
    }
  }

  // Crear deep link
  static createDeepLink(screen, params = {}) {
    const baseUrl = 'myapp://';
    const queryString = Object.keys(params)
      .map(key => `${key}=${encodeURIComponent(params[key])}`)
      .join('&');
    
    const url = `${baseUrl}${screen}${queryString ? '?' + queryString : ''}`;
    return url;
  }

  // Abrir enlace externo
  static async openExternalLink(url) {
    try {
      const supported = await Linking.canOpenURL(url);
      
      if (supported) {
        await Linking.openURL(url);
      } else {
        console.log(`No se puede abrir el enlace: ${url}`);
      }
    } catch (error) {
      console.error('Error al abrir enlace externo:', error);
    }
  }

  // Compartir contenido
  static async shareContent(title, message, url) {
    try {
      const shareUrl = url || this.createDeepLink('share', { title, message });
      
      await Linking.openURL(
        `mailto:?subject=${encodeURIComponent(title)}&body=${encodeURIComponent(message + '\n\n' + shareUrl)}`
      );
    } catch (error) {
      console.error('Error al compartir contenido:', error);
    }
  }
}
```

### 5. Componente de Gestión de Notificaciones

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  FlatList,
  Alert,
  Switch,
} from 'react-native';
import { PushNotificationManager } from './PushNotificationManager';
import { LocalNotificationManager } from './LocalNotificationManager';
import { BadgeManager } from './BadgeManager';

const NotificationSettings = () => {
  const [pushEnabled, setPushEnabled] = useState(false);
  const [localEnabled, setLocalEnabled] = useState(false);
  const [badgeCount, setBadgeCount] = useState(0);
  const [scheduledNotifications, setScheduledNotifications] = useState([]);

  useEffect(() => {
    initializeNotifications();
    loadScheduledNotifications();
  }, []);

  // Inicializar notificaciones
  const initializeNotifications = async () => {
    const pushPermission = await PushNotificationManager.requestPermission();
    setPushEnabled(pushPermission);
    
    if (pushPermission) {
      const token = await PushNotificationManager.getFCMToken();
      if (token) {
        // Enviar token al servidor
        await PushNotificationManager.sendTokenToServer(token, 'user123');
      }
    }
  };

  // Cargar notificaciones programadas
  const loadScheduledNotifications = async () => {
    const notifications = await LocalNotificationManager.getScheduledNotifications();
    setScheduledNotifications(notifications);
  };

  // Cambiar estado de notificaciones push
  const togglePushNotifications = async (value) => {
    if (value) {
      const permission = await PushNotificationManager.requestPermission();
      setPushEnabled(permission);
    } else {
      setPushEnabled(false);
      // Aquí podrías desuscribir al usuario del servidor
    }
  };

  // Cambiar estado de notificaciones locales
  const toggleLocalNotifications = (value) => {
    setLocalEnabled(value);
    if (!value) {
      LocalNotificationManager.cancelAllNotifications();
      setScheduledNotifications([]);
    }
  };

  // Programar notificación de prueba
  const scheduleTestNotification = () => {
    const futureDate = new Date(Date.now() + 10000); // 10 segundos en el futuro
    
    LocalNotificationManager.scheduleNotification(
      'Notificación de Prueba',
      'Esta es una notificación programada para probar el sistema',
      futureDate,
      { action: 'show_alert', message: '¡Notificación de prueba exitosa!' }
    );
    
    Alert.alert('Éxito', 'Notificación programada para 10 segundos');
    loadScheduledNotifications();
  };

  // Mostrar notificación inmediata
  const showTestNotification = () => {
    LocalNotificationManager.showNotification(
      'Notificación Inmediata',
      'Esta es una notificación de prueba',
      { action: 'show_alert', message: '¡Notificación inmediata!' }
    );
  };

  // Cambiar badge
  const changeBadge = (increment) => {
    if (increment) {
      BadgeManager.incrementBadge();
    } else {
      BadgeManager.decrementBadge();
    }
    setBadgeCount(BadgeManager.getBadgeNumber());
  };

  // Limpiar badge
  const clearBadge = () => {
    BadgeManager.clearBadge();
    setBadgeCount(0);
  };

  // Renderizar notificación programada
  const renderScheduledNotification = ({ item }) => (
    <View style={styles.notificationItem}>
      <Text style={styles.notificationTitle}>{item.title}</Text>
      <Text style={styles.notificationMessage}>{item.message}</Text>
      <Text style={styles.notificationDate}>
        Programada para: {new Date(item.date).toLocaleString()}
      </Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Configuración de Notificaciones</Text>
      
      {/* Notificaciones Push */}
      <View style={styles.settingRow}>
        <Text style={styles.settingLabel}>Notificaciones Push</Text>
        <Switch
          value={pushEnabled}
          onValueChange={togglePushNotifications}
        />
      </View>
      
      {/* Notificaciones Locales */}
      <View style={styles.settingRow}>
        <Text style={styles.settingLabel}>Notificaciones Locales</Text>
        <Switch
          value={localEnabled}
          onValueChange={toggleLocalNotifications}
        />
      </View>
      
      {/* Contador de Badge */}
      <View style={styles.badgeSection}>
        <Text style={styles.badgeLabel}>Contador de Badge: {badgeCount}</Text>
        <View style={styles.badgeControls}>
          <TouchableOpacity
            style={styles.badgeButton}
            onPress={() => changeBadge(true)}
          >
            <Text style={styles.badgeButtonText}>+</Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.badgeButton}
            onPress={() => changeBadge(false)}
          >
            <Text style={styles.badgeButtonText}>-</Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.clearBadgeButton}
            onPress={clearBadge}
          >
            <Text style={styles.clearBadgeButtonText}>Limpiar</Text>
          </TouchableOpacity>
        </View>
      </View>
      
      {/* Botones de prueba */}
      <View style={styles.testButtons}>
        <TouchableOpacity
          style={styles.testButton}
          onPress={showTestNotification}
        >
          <Text style={styles.testButtonText}>🔔 Notificación Inmediata</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.testButton}
          onPress={scheduleTestNotification}
        >
          <Text style={styles.testButtonText}>⏰ Programar Notificación</Text>
        </TouchableOpacity>
      </View>
      
      {/* Lista de notificaciones programadas */}
      {scheduledNotifications.length > 0 && (
        <View style={styles.scheduledSection}>
          <Text style={styles.sectionTitle}>Notificaciones Programadas</Text>
          <FlatList
            data={scheduledNotifications}
            renderItem={renderScheduledNotification}
            keyExtractor={(item, index) => index.toString()}
            style={styles.notificationsList}
          />
        </View>
      )}
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
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  settingRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  settingLabel: {
    fontSize: 16,
    color: '#333',
  },
  badgeSection: {
    marginVertical: 20,
    padding: 15,
    backgroundColor: 'white',
    borderRadius: 8,
  },
  badgeLabel: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 15,
    textAlign: 'center',
  },
  badgeControls: {
    flexDirection: 'row',
    justifyContent: 'center',
    gap: 10,
  },
  badgeButton: {
    backgroundColor: '#2196F3',
    width: 40,
    height: 40,
    borderRadius: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  badgeButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  clearBadgeButton: {
    backgroundColor: '#f44336',
    paddingHorizontal: 15,
    paddingVertical: 10,
    borderRadius: 6,
  },
  clearBadgeButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: '600',
  },
  testButtons: {
    gap: 15,
    marginVertical: 20,
  },
  testButton: {
    backgroundColor: '#4CAF50',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  testButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  scheduledSection: {
    marginTop: 20,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  notificationsList: {
    maxHeight: 200,
  },
  notificationItem: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
  },
  notificationTitle: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
    marginBottom: 5,
  },
  notificationMessage: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
  notificationDate: {
    fontSize: 12,
    color: '#999',
  },
});

export default NotificationSettings;
```

## Ejercicios Prácticos

### Ejercicio 1: Sistema de Notificaciones Completo
Crea un sistema que maneje notificaciones push, locales y badges de forma integrada.

### Ejercicio 2: Notificaciones Programadas
Implementa un sistema de recordatorios y notificaciones programadas por fecha y hora.

### Ejercicio 3: Deep Linking Avanzado
Crea un sistema de deep linking que permita navegar a pantallas específicas desde notificaciones.

## Resumen de la Clase

Hemos aprendido:
1. **Notificaciones Push**: Configuración con Firebase Cloud Messaging
2. **Notificaciones Locales**: Programación y manejo de eventos
3. **Sistema de Badges**: Contadores y gestión de notificaciones
4. **Deep Linking**: Navegación desde enlaces externos
5. **Gestión Integrada**: Sistema completo de notificaciones

## Navegación
- **Anterior**: [Clase 3: Geolocalización y Mapas](clase_3_geolocalizacion_mapas.md)
- **Siguiente**: [Clase 5: Sensores y Hardware Avanzado](clase_5_sensores_hardware_avanzado.md)
- **Inicio**: [Índice del Módulo](../senior_2/README.md)

## Próxima Clase
En la siguiente clase aprenderemos sobre **Sensores y Hardware Avanzado**, incluyendo acelerómetro, giroscopio, brújula, vibración y linterna.
