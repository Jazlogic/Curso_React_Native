# ☁️ Clase 4: Servicios de Expo

## 🎯 Objetivos de la Clase
- Implementar servicios en la nube de Expo
- Configurar Expo Updates para actualizaciones OTA
- Implementar notificaciones push con Expo
- Utilizar servicios de autenticación y almacenamiento seguro

---

## 📚 Contenido Teórico

### 1. Expo Updates - Actualizaciones OTA

#### **¿Qué son las Actualizaciones OTA?**
**Over-The-Air (OTA)** permite actualizar tu aplicación sin pasar por las tiendas de aplicaciones:

- **Actualizaciones instantáneas**: Sin esperar aprobación de stores
- **Rollbacks automáticos**: Reversión si algo sale mal
- **Actualizaciones incrementales**: Solo se descargan los cambios
- **Control de versiones**: Gestión de diferentes versiones

#### **Configuración Básica**
```javascript
// app.json
{
  "expo": {
    "updates": {
      "enabled": true,
      "fallbackToCacheTimeout": 0,
      "url": "https://u.expo.dev/your-project-id"
    },
    "runtimeVersion": {
      "policy": "sdkVersion"
    }
  }
}
```

#### **Implementación de Updates**
```javascript
// services/updateService.js
import * as Updates from 'expo-updates';
import { Alert } from 'react-native';

export class UpdateService {
  // Verificar actualizaciones disponibles
  static async checkForUpdates() {
    try {
      const update = await Updates.checkForUpdateAsync();
      
      if (update.isAvailable) {
        console.log('Actualización disponible:', update);
        return update;
      }
      
      return null;
    } catch (error) {
      console.error('Error al verificar actualizaciones:', error);
      return null;
    }
  }

  // Descargar y aplicar actualización
  static async downloadAndApplyUpdate() {
    try {
      console.log('Descargando actualización...');
      
      const result = await Updates.fetchUpdateAsync();
      
      if (result.isNew) {
        console.log('Actualización descargada, aplicando...');
        
        Alert.alert(
          'Actualización Disponible',
          'Se ha descargado una nueva versión. La app se reiniciará para aplicar los cambios.',
          [
            {
              text: 'Reiniciar Ahora',
              onPress: () => Updates.reloadAsync(),
            },
            {
              text: 'Más Tarde',
              style: 'cancel',
            },
          ]
        );
      }
      
      return result;
    } catch (error) {
      console.error('Error al aplicar actualización:', error);
      throw error;
    }
  }

  // Configurar listener de eventos
  static setupUpdateListener() {
    Updates.addListener((event) => {
      if (event.type === Updates.UpdateEvent.UPDATE_AVAILABLE) {
        console.log('Nueva actualización disponible');
        this.downloadAndApplyUpdate();
      } else if (event.type === Updates.UpdateEvent.UPDATE_DOWNLOADED) {
        console.log('Actualización descargada');
      }
    });
  }

  // Obtener información de la versión actual
  static getCurrentVersion() {
    return {
      runtimeVersion: Updates.runtimeVersion,
      channel: Updates.channel,
      isEmbeddedLaunch: Updates.isEmbeddedLaunch,
    };
  }
}

// Uso en App.js
import { UpdateService } from './services/updateService';

export default function App() {
  useEffect(() => {
    // Configurar listener de actualizaciones
    UpdateService.setupUpdateListener();
    
    // Verificar actualizaciones al iniciar
    UpdateService.checkForUpdates();
  }, []);

  // ... resto del código
}
```

#### **Configuración Avanzada de Updates**
```javascript
// app.config.js
export default {
  expo: {
    name: "Mi App",
    slug: "mi-app",
    version: "1.0.0",
    orientation: "portrait",
    icon: "./assets/icon.png",
    userInterfaceStyle: "light",
    splash: {
      image: "./assets/splash.png",
      resizeMode: "contain",
      backgroundColor: "#ffffff"
    },
    updates: {
      enabled: true,
      fallbackToCacheTimeout: 0,
      url: "https://u.expo.dev/your-project-id"
    },
    runtimeVersion: {
      policy: "sdkVersion"
    },
    extra: {
      eas: {
        projectId: "your-project-id"
      }
    }
  }
};
```

### 2. Expo Notifications - Notificaciones Push

#### **Configuración de Notificaciones Push**
```javascript
// services/notificationService.js
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import Constants from 'expo-constants';

// Configurar comportamiento de notificaciones
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export class NotificationService {
  // Registrar dispositivo para notificaciones push
  static async registerForPushNotifications() {
    let token;

    if (Device.isDevice) {
      const { status: existingStatus } = await Notifications.getPermissionsAsync();
      let finalStatus = existingStatus;
      
      if (existingStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }
      
      if (finalStatus !== 'granted') {
        console.log('Permisos de notificación denegados');
        return null;
      }

      // Obtener token de Expo
      token = (await Notifications.getExpoPushTokenAsync({
        projectId: Constants.expoConfig.extra.eas.projectId,
      })).data;

      console.log('Token de notificación:', token);
    } else {
      console.log('Debe usar un dispositivo físico para notificaciones push');
    }

    // Configurar canal para Android
    if (Platform.OS === 'android') {
      await Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: '#FF231F7C',
      });
    }

    return token;
  }

  // Enviar notificación local
  static async scheduleLocalNotification(title, body, data = {}, trigger = null) {
    try {
      const notificationId = await Notifications.scheduleNotificationAsync({
        content: {
          title,
          body,
          data,
          sound: 'default',
        },
        trigger: trigger || { seconds: 2 },
      });

      console.log('Notificación local programada:', notificationId);
      return notificationId;
    } catch (error) {
      console.error('Error al programar notificación local:', error);
      throw error;
    }
  }

  // Enviar notificación push
  static async sendPushNotification(expoPushToken, title, body, data = {}) {
    const message = {
      to: expoPushToken,
      sound: 'default',
      title,
      body,
      data,
    };

    try {
      const response = await fetch('https://exp.host/--/api/v2/push/send', {
        method: 'POST',
        headers: {
          Accept: 'application/json',
          'Accept-encoding': 'gzip, deflate',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(message),
      });

      const result = await response.json();
      console.log('Notificación push enviada:', result);
      return result;
    } catch (error) {
      console.error('Error al enviar notificación push:', error);
      throw error;
    }
  }

  // Configurar listener de notificaciones
  static setupNotificationListeners() {
    // Listener para notificaciones recibidas
    const notificationListener = Notifications.addNotificationReceivedListener(notification => {
      console.log('Notificación recibida:', notification);
    });

    // Listener para respuestas a notificaciones
    const responseListener = Notifications.addNotificationResponseReceivedListener(response => {
      console.log('Respuesta a notificación:', response);
      
      // Manejar la respuesta (navegación, acciones, etc.)
      this.handleNotificationResponse(response);
    });

    return { notificationListener, responseListener };
  }

  // Manejar respuesta a notificación
  static handleNotificationResponse(response) {
    const { data } = response.notification.request.content;
    
    // Ejemplo: navegación basada en datos de la notificación
    if (data.screen) {
      // Navegar a la pantalla especificada
      console.log('Navegando a:', data.screen);
    }
  }

  // Cancelar notificación programada
  static async cancelNotification(notificationId) {
    try {
      await Notifications.cancelScheduledNotificationAsync(notificationId);
      console.log('Notificación cancelada:', notificationId);
    } catch (error) {
      console.error('Error al cancelar notificación:', error);
    }
  }

  // Obtener todas las notificaciones programadas
  static async getScheduledNotifications() {
    try {
      const notifications = await Notifications.getAllScheduledNotificationsAsync();
      console.log('Notificaciones programadas:', notifications);
      return notifications;
    } catch (error) {
      console.error('Error al obtener notificaciones:', error);
      return [];
    }
  }
}
```

#### **Uso en Componentes**
```javascript
// components/NotificationManager.js
import React, { useEffect, useState } from 'react';
import { View, Text, TouchableOpacity, Alert } from 'react-native';
import { NotificationService } from '../services/notificationService';

export default function NotificationManager() {
  const [expoPushToken, setExpoPushToken] = useState('');
  const [isRegistered, setIsRegistered] = useState(false);

  useEffect(() => {
    // Configurar listeners de notificación
    const listeners = NotificationService.setupNotificationListeners();

    // Registrar para notificaciones push
    registerForNotifications();

    return () => {
      // Limpiar listeners
      if (listeners.notificationListener) {
        Notifications.removeNotificationSubscription(listeners.notificationListener);
      }
      if (listeners.responseListener) {
        Notifications.removeNotificationSubscription(listeners.responseListener);
      }
    };
  }, []);

  const registerForNotifications = async () => {
    try {
      const token = await NotificationService.registerForPushNotifications();
      if (token) {
        setExpoPushToken(token);
        setIsRegistered(true);
        Alert.alert('Éxito', 'Dispositivo registrado para notificaciones');
      }
    } catch (error) {
      Alert.alert('Error', 'No se pudo registrar para notificaciones');
    }
  };

  const sendTestNotification = async () => {
    try {
      await NotificationService.scheduleLocalNotification(
        'Notificación de Prueba',
        'Esta es una notificación local de prueba',
        { type: 'test' }
      );
      Alert.alert('Éxito', 'Notificación local enviada');
    } catch (error) {
      Alert.alert('Error', 'No se pudo enviar la notificación');
    }
  };

  const sendPushNotification = async () => {
    if (!expoPushToken) {
      Alert.alert('Error', 'No hay token de notificación');
      return;
    }

    try {
      await NotificationService.sendPushNotification(
        expoPushToken,
        'Notificación Push',
        'Esta es una notificación push de prueba',
        { type: 'push_test' }
      );
      Alert.alert('Éxito', 'Notificación push enviada');
    } catch (error) {
      Alert.alert('Error', 'No se pudo enviar la notificación push');
    }
  };

  return (
    <View style={{ flex: 1, padding: 20, justifyContent: 'center' }}>
      <Text style={{ fontSize: 20, textAlign: 'center', marginBottom: 30 }}>
        Gestor de Notificaciones
      </Text>

      <TouchableOpacity
        style={{
          backgroundColor: '#007AFF',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={registerForNotifications}
      >
        <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
          {isRegistered ? 'Re-registrar' : 'Registrar para Notificaciones'}
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={{
          backgroundColor: '#34C759',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={sendTestNotification}
      >
        <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
          Enviar Notificación Local
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={{
          backgroundColor: '#FF9500',
          padding: 15,
          borderRadius: 10,
          marginBottom: 20,
        }}
        onPress={sendPushNotification}
        disabled={!isRegistered}
      >
        <Text style={{ color: 'white', textAlign: 'center', fontSize: 16 }}>
          Enviar Notificación Push
        </Text>
      </TouchableOpacity>

      {expoPushToken && (
        <View style={{ marginTop: 20 }}>
          <Text style={{ fontSize: 14, marginBottom: 10 }}>
            Token de Notificación:
          </Text>
          <Text style={{ fontSize: 12, color: '#666' }} numberOfLines={3}>
            {expoPushToken}
          </Text>
        </View>
      )}
    </View>
  );
}
```

### 3. Expo AuthSession - Autenticación OAuth

#### **Configuración de OAuth**
```javascript
// services/authService.js
import * as AuthSession from 'expo-auth-session';
import * as WebBrowser from 'expo-web-browser';
import { makeRedirectUri } from 'expo-auth-session';

// Configurar WebBrowser para OAuth
WebBrowser.maybeCompleteAuthSession();

export class AuthService {
  // Configuración de Google OAuth
  static googleConfig = {
    clientId: 'your-google-client-id.apps.googleusercontent.com',
    scopes: ['openid', 'profile', 'email'],
    redirectUri: makeRedirectUri({
      scheme: 'your-app-scheme'
    }),
  };

  // Configuración de Facebook OAuth
  static facebookConfig = {
    clientId: 'your-facebook-app-id',
    scopes: ['public_profile', 'email'],
    redirectUri: makeRedirectUri({
      scheme: 'your-app-scheme'
    }),
  };

  // Autenticación con Google
  static async signInWithGoogle() {
    try {
      const request = new AuthSession.AuthRequest({
        clientId: this.googleConfig.clientId,
        scopes: this.googleConfig.scopes,
        redirectUri: this.googleConfig.redirectUri,
        responseType: AuthSession.ResponseType.Code,
        additionalParameters: {},
      });

      const result = await request.promptAsync({
        authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
        useProxy: true,
      });

      if (result.type === 'success') {
        // Intercambiar código por token
        const tokenResult = await this.exchangeCodeForToken(result.params.code, 'google');
        return tokenResult;
      }

      return null;
    } catch (error) {
      console.error('Error en autenticación con Google:', error);
      throw error;
    }
  }

  // Autenticación con Facebook
  static async signInWithFacebook() {
    try {
      const request = new AuthSession.AuthRequest({
        clientId: this.facebookConfig.clientId,
        scopes: this.facebookConfig.scopes,
        redirectUri: this.facebookConfig.redirectUri,
        responseType: AuthSession.ResponseType.Code,
        additionalParameters: {},
      });

      const result = await request.promptAsync({
        authorizationEndpoint: 'https://www.facebook.com/v12.0/dialog/oauth',
        useProxy: true,
      });

      if (result.type === 'success') {
        // Intercambiar código por token
        const tokenResult = await this.exchangeCodeForToken(result.params.code, 'facebook');
        return tokenResult;
      }

      return null;
    } catch (error) {
      console.error('Error en autenticación con Facebook:', error);
      throw error;
    }
  }

  // Intercambiar código por token
  static async exchangeCodeForToken(code, provider) {
    try {
      let tokenEndpoint;
      let clientSecret;

      if (provider === 'google') {
        tokenEndpoint = 'https://oauth2.googleapis.com/token';
        clientSecret = 'your-google-client-secret';
      } else if (provider === 'facebook') {
        tokenEndpoint = 'https://graph.facebook.com/v12.0/oauth/access_token';
        clientSecret = 'your-facebook-app-secret';
      }

      const response = await fetch(tokenEndpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: new URLSearchParams({
          client_id: provider === 'google' ? this.googleConfig.clientId : this.facebookConfig.clientId,
          client_secret: clientSecret,
          code,
          grant_type: 'authorization_code',
          redirect_uri: provider === 'google' ? this.googleConfig.redirectUri : this.facebookConfig.redirectUri,
        }),
      });

      const tokenData = await response.json();
      console.log('Token obtenido:', tokenData);

      // Obtener información del usuario
      const userInfo = await this.getUserInfo(tokenData.access_token, provider);
      
      return {
        accessToken: tokenData.access_token,
        userInfo,
        provider,
      };
    } catch (error) {
      console.error('Error al intercambiar código por token:', error);
      throw error;
    }
  }

  // Obtener información del usuario
  static async getUserInfo(accessToken, provider) {
    try {
      let userInfoEndpoint;

      if (provider === 'google') {
        userInfoEndpoint = 'https://www.googleapis.com/oauth2/v2/userinfo';
      } else if (provider === 'facebook') {
        userInfoEndpoint = `https://graph.facebook.com/me?fields=id,name,email&access_token=${accessToken}`;
      }

      const response = await fetch(userInfoEndpoint, {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      });

      const userInfo = await response.json();
      console.log('Información del usuario:', userInfo);

      return userInfo;
    } catch (error) {
      console.error('Error al obtener información del usuario:', error);
      throw error;
    }
  }

  // Cerrar sesión
  static async signOut() {
    try {
      // Limpiar tokens almacenados
      await SecureStore.deleteItemAsync('accessToken');
      await SecureStore.deleteItemAsync('userInfo');
      
      console.log('Sesión cerrada exitosamente');
      return true;
    } catch (error) {
      console.error('Error al cerrar sesión:', error);
      return false;
    }
  }
}
```

### 4. Expo SecureStore - Almacenamiento Seguro

#### **Configuración y Uso**
```javascript
// services/secureStoreService.js
import * as SecureStore from 'expo-secure-store';

export class SecureStoreService {
  // Guardar datos de forma segura
  static async saveItem(key, value) {
    try {
      await SecureStore.setItemAsync(key, JSON.stringify(value));
      console.log('Dato guardado de forma segura:', key);
      return true;
    } catch (error) {
      console.error('Error al guardar dato seguro:', error);
      return false;
    }
  }

  // Obtener datos de forma segura
  static async getItem(key) {
    try {
      const value = await SecureStore.getItemAsync(key);
      if (value) {
        return JSON.parse(value);
      }
      return null;
    } catch (error) {
      console.error('Error al obtener dato seguro:', error);
      return null;
    }
  }

  // Eliminar dato seguro
  static async deleteItem(key) {
    try {
      await SecureStore.deleteItemAsync(key);
      console.log('Dato eliminado:', key);
      return true;
    } catch (error) {
      console.error('Error al eliminar dato seguro:', error);
      return false;
    }
  }

  // Verificar si existe una clave
  static async hasItem(key) {
    try {
      const value = await SecureStore.getItemAsync(key);
      return value !== null;
    } catch (error) {
      console.error('Error al verificar dato seguro:', error);
      return false;
    }
  }

  // Guardar credenciales de usuario
  static async saveUserCredentials(userId, credentials) {
    const key = `user_credentials_${userId}`;
    return await this.saveItem(key, credentials);
  }

  // Obtener credenciales de usuario
  static async getUserCredentials(userId) {
    const key = `user_credentials_${userId}`;
    return await this.getItem(key);
  }

  // Guardar token de autenticación
  static async saveAuthToken(token) {
    return await this.saveItem('auth_token', token);
  }

  // Obtener token de autenticación
  static async getAuthToken() {
    return await this.getItem('auth_token');
  }

  // Limpiar todos los datos seguros
  static async clearAll() {
    try {
      // Obtener todas las claves (esto requiere implementación personalizada)
      const commonKeys = [
        'auth_token',
        'user_credentials',
        'refresh_token',
        'session_data'
      ];

      for (const key of commonKeys) {
        await this.deleteItem(key);
      }

      console.log('Todos los datos seguros han sido eliminados');
      return true;
    } catch (error) {
      console.error('Error al limpiar datos seguros:', error);
      return false;
    }
  }
}
```

---

## 🛠️ Ejercicios Prácticos

### Ejercicio 1: Sistema de Actualizaciones OTA
Implementa un sistema completo de actualizaciones OTA:

**Requisitos:**
- Verificación automática de actualizaciones
- Descarga y aplicación de actualizaciones
- Rollback automático en caso de error
- Interfaz de usuario para gestionar actualizaciones

### Ejercicio 2: Sistema de Notificaciones Push
Crea un sistema completo de notificaciones:

**Requisitos:**
- Registro de dispositivos para notificaciones push
- Envío de notificaciones locales y remotas
- Manejo de respuestas del usuario
- Configuración de canales de notificación

### Ejercicio 3: Sistema de Autenticación OAuth
Implementa autenticación con múltiples proveedores:

**Requisitos:**
- Autenticación con Google y Facebook
- Manejo seguro de tokens
- Persistencia de sesiones
- Logout y renovación de tokens

---

## 🔍 Puntos Clave

1. **Expo Updates** permite actualizaciones OTA sin pasar por las tiendas
2. **Expo Notifications** maneja notificaciones locales y push de forma unificada
3. **Expo AuthSession** simplifica la implementación de OAuth
4. **Expo SecureStore** proporciona almacenamiento seguro para datos sensibles
5. **Los servicios de Expo** se integran perfectamente con el SDK
6. **La configuración** varía según el proveedor de OAuth
7. **El manejo de errores** es crucial para la robustez de los servicios

---

## 📖 Recursos Adicionales

- [Expo Updates Documentation](https://docs.expo.dev/versions/latest/sdk/updates/)
- [Expo Notifications Documentation](https://docs.expo.dev/versions/latest/sdk/notifications/)
- [Expo AuthSession Documentation](https://docs.expo.dev/versions/latest/sdk/auth-session/)
- [Expo SecureStore Documentation](https://docs.expo.dev/versions/latest/sdk/securestore/)

---

## 🎉 ¡Clase Completada!

Has completado exitosamente la **Clase 4: Servicios de Expo** 🚀

**Lo que has aprendido:**
- ✅ Implementación de actualizaciones OTA
- ✅ Sistema de notificaciones push completo
- ✅ Autenticación OAuth con múltiples proveedores
- ✅ Almacenamiento seguro de datos sensibles
- ✅ Integración de servicios en la nube

**Próximos pasos:**
- Implementar los servicios en tu aplicación
- Configurar proveedores OAuth
- Continuar con la siguiente clase sobre publicación y distribución

¡Excelente trabajo! 🎯

---

## 🔗 Navegación del Módulo

### **Clase Anterior**: [Clase 3: Herramientas de Desarrollo](clase_3_herramientas_desarrollo.md)
### **Clase Siguiente**: [Clase 5: Publicación y Distribución](clase_5_publicacion_distribucion.md)
### **Volver al Módulo**: [Módulo 8: Expo y Desarrollo Rápido](README.md)
### **Volver al Inicio**: [Índice Completo](../INDICE_COMPLETO.md)
