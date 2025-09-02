# 🌍 Clase 4: Formatos Regionales

## 📋 Descripción de la Clase

### **¿Qué Aprenderás?**

En esta clase aprenderás a implementar formatos regionales para fechas, monedas, números, unidades de medida, calendarios y zonas horarias, adaptando tu aplicación a las convenciones culturales de diferentes regiones del mundo.

### **🎯 Objetivos de Aprendizaje**

Al finalizar esta clase, serás capaz de:

1. **Formatear** fechas según convenciones regionales
2. **Configurar** formatos de moneda y números por región
3. **Implementar** unidades de medida y peso locales
4. **Manejar** calendarios y zonas horarias
5. **Adaptar** formatos según preferencias del usuario

### **⏱️ Duración Estimada**
- **Teoría**: 45 minutos
- **Práctica**: 75 minutos
- **Total**: 2 horas

---

## 📚 Contenido Teórico

### **1. Formateo de Fechas por Región**

#### **API de Internacionalización para Fechas**

```javascript
// utils/dateFormatter.js
class DateFormatter {
  constructor() {
    this.locales = {
      'en-US': 'en-US',
      'es-ES': 'es-ES',
      'fr-FR': 'fr-FR',
      'de-DE': 'de-DE',
      'ar-SA': 'ar-SA',
      'ja-JP': 'ja-JP',
      'zh-CN': 'zh-CN',
    };
  }
  
  formatDate(date, locale = 'en-US', options = {}) {
    const defaultOptions = {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.DateTimeFormat(locale, formatOptions).format(date);
    } catch (error) {
      console.error('Date formatting error:', error);
      return date.toLocaleDateString();
    }
  }
  
  formatTime(date, locale = 'en-US', options = {}) {
    const defaultOptions = {
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.DateTimeFormat(locale, formatOptions).format(date);
    } catch (error) {
      console.error('Time formatting error:', error);
      return date.toLocaleTimeString();
    }
  }
  
  formatDateTime(date, locale = 'en-US', options = {}) {
    const defaultOptions = {
      year: 'numeric',
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.DateTimeFormat(locale, formatOptions).format(date);
    } catch (error) {
      console.error('DateTime formatting error:', error);
      return date.toLocaleString();
    }
  }
  
  getRelativeTime(date, locale = 'en-US') {
    const now = new Date();
    const diffInSeconds = Math.floor((now - date) / 1000);
    
    const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });
    
    if (diffInSeconds < 60) {
      return rtf.format(-diffInSeconds, 'second');
    } else if (diffInSeconds < 3600) {
      return rtf.format(-Math.floor(diffInSeconds / 60), 'minute');
    } else if (diffInSeconds < 86400) {
      return rtf.format(-Math.floor(diffInSeconds / 3600), 'hour');
    } else if (diffInSeconds < 2592000) {
      return rtf.format(-Math.floor(diffInSeconds / 86400), 'day');
    } else if (diffInSeconds < 31536000) {
      return rtf.format(-Math.floor(diffInSeconds / 2592000), 'month');
    } else {
      return rtf.format(-Math.floor(diffInSeconds / 31536000), 'year');
    }
  }
}

export const dateFormatter = new DateFormatter();
```

#### **Ejemplos de Formateo de Fechas**

```javascript
// Ejemplos de uso
const date = new Date('2024-01-15T14:30:00Z');

// Formateo básico
console.log(dateFormatter.formatDate(date, 'en-US')); // "January 15, 2024"
console.log(dateFormatter.formatDate(date, 'es-ES')); // "15 de enero de 2024"
console.log(dateFormatter.formatDate(date, 'fr-FR')); // "15 janvier 2024"
console.log(dateFormatter.formatDate(date, 'de-DE')); // "15. Januar 2024"
console.log(dateFormatter.formatDate(date, 'ar-SA')); // "١٥ يناير ٢٠٢٤"

// Formateo con opciones personalizadas
console.log(dateFormatter.formatDate(date, 'en-US', {
  year: 'numeric',
  month: 'short',
  day: 'numeric',
  weekday: 'long'
})); // "Monday, Jan 15, 2024"

// Tiempo relativo
console.log(dateFormatter.getRelativeTime(date, 'en-US')); // "2 days ago"
console.log(dateFormatter.getRelativeTime(date, 'es-ES')); // "hace 2 días"
```

#### **Componente de Fecha Localizada**

```javascript
// components/LocalizedDate.js
import React from 'react';
import { Text, StyleSheet } from 'react-native';
import { useTranslation } from 'react-i18next';
import { dateFormatter } from '../utils/dateFormatter';

const LocalizedDate = ({ 
  date, 
  format = 'date',
  style,
  options = {} 
}) => {
  const { i18n } = useTranslation();
  const locale = i18n.language;
  
  const formatDate = () => {
    switch (format) {
      case 'date':
        return dateFormatter.formatDate(date, locale, options);
      case 'time':
        return dateFormatter.formatTime(date, locale, options);
      case 'datetime':
        return dateFormatter.formatDateTime(date, locale, options);
      case 'relative':
        return dateFormatter.getRelativeTime(date, locale);
      default:
        return dateFormatter.formatDate(date, locale, options);
    }
  };
  
  return (
    <Text style={style}>
      {formatDate()}
    </Text>
  );
};

export default LocalizedDate;
```

### **2. Formateo de Moneda y Números**

#### **Formateo de Moneda**

```javascript
// utils/currencyFormatter.js
class CurrencyFormatter {
  constructor() {
    this.currencies = {
      'USD': 'USD',
      'EUR': 'EUR',
      'GBP': 'GBP',
      'JPY': 'JPY',
      'CNY': 'CNY',
      'INR': 'INR',
      'BRL': 'BRL',
      'MXN': 'MXN',
      'CAD': 'CAD',
      'AUD': 'AUD',
    };
  }
  
  formatCurrency(amount, currency = 'USD', locale = 'en-US', options = {}) {
    const defaultOptions = {
      style: 'currency',
      currency: currency,
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.NumberFormat(locale, formatOptions).format(amount);
    } catch (error) {
      console.error('Currency formatting error:', error);
      return `${currency} ${amount.toFixed(2)}`;
    }
  }
  
  formatNumber(number, locale = 'en-US', options = {}) {
    const defaultOptions = {
      minimumFractionDigits: 0,
      maximumFractionDigits: 2,
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.NumberFormat(locale, formatOptions).format(number);
    } catch (error) {
      console.error('Number formatting error:', error);
      return number.toString();
    }
  }
  
  formatPercent(value, locale = 'en-US', options = {}) {
    const defaultOptions = {
      style: 'percent',
      minimumFractionDigits: 0,
      maximumFractionDigits: 2,
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.NumberFormat(locale, formatOptions).format(value / 100);
    } catch (error) {
      console.error('Percent formatting error:', error);
      return `${value}%`;
    }
  }
  
  formatCompactNumber(number, locale = 'en-US') {
    try {
      return new Intl.NumberFormat(locale, {
        notation: 'compact',
        compactDisplay: 'short'
      }).format(number);
    } catch (error) {
      console.error('Compact number formatting error:', error);
      return number.toString();
    }
  }
}

export const currencyFormatter = new CurrencyFormatter();
```

#### **Ejemplos de Formateo de Moneda**

```javascript
// Ejemplos de uso
const amount = 1234.56;

// Formateo de moneda
console.log(currencyFormatter.formatCurrency(amount, 'USD', 'en-US')); // "$1,234.56"
console.log(currencyFormatter.formatCurrency(amount, 'EUR', 'es-ES')); // "1.234,56 €"
console.log(currencyFormatter.formatCurrency(amount, 'JPY', 'ja-JP')); // "¥1,235"
console.log(currencyFormatter.formatCurrency(amount, 'INR', 'en-IN')); // "₹1,234.56"

// Formateo de números
console.log(currencyFormatter.formatNumber(amount, 'en-US')); // "1,234.56"
console.log(currencyFormatter.formatNumber(amount, 'es-ES')); // "1.234,56"
console.log(currencyFormatter.formatNumber(amount, 'fr-FR')); // "1 234,56"

// Formateo de porcentajes
console.log(currencyFormatter.formatPercent(0.1234, 'en-US')); // "12%"
console.log(currencyFormatter.formatPercent(0.1234, 'es-ES')); // "12 %"

// Números compactos
console.log(currencyFormatter.formatCompactNumber(1234, 'en-US')); // "1.2K"
console.log(currencyFormatter.formatCompactNumber(1234567, 'en-US')); // "1.2M"
```

#### **Componente de Moneda Localizada**

```javascript
// components/LocalizedCurrency.js
import React from 'react';
import { Text, StyleSheet } from 'react-native';
import { useTranslation } from 'react-i18next';
import { currencyFormatter } from '../utils/currencyFormatter';

const LocalizedCurrency = ({ 
  amount, 
  currency = 'USD',
  style,
  options = {} 
}) => {
  const { i18n } = useTranslation();
  const locale = i18n.language;
  
  const formatCurrency = () => {
    return currencyFormatter.formatCurrency(amount, currency, locale, options);
  };
  
  return (
    <Text style={style}>
      {formatCurrency()}
    </Text>
  );
};

export default LocalizedCurrency;
```

### **3. Unidades de Medida y Peso**

#### **Formateo de Unidades**

```javascript
// utils/unitFormatter.js
class UnitFormatter {
  constructor() {
    this.units = {
      // Longitud
      'meter': 'meter',
      'kilometer': 'kilometer',
      'centimeter': 'centimeter',
      'inch': 'inch',
      'foot': 'foot',
      'yard': 'yard',
      'mile': 'mile',
      
      // Peso
      'gram': 'gram',
      'kilogram': 'kilogram',
      'pound': 'pound',
      'ounce': 'ounce',
      
      // Temperatura
      'celsius': 'celsius',
      'fahrenheit': 'fahrenheit',
      'kelvin': 'kelvin',
      
      // Volumen
      'liter': 'liter',
      'milliliter': 'milliliter',
      'gallon': 'gallon',
      'quart': 'quart',
      'pint': 'pint',
    };
  }
  
  formatUnit(value, unit, locale = 'en-US', options = {}) {
    const defaultOptions = {
      style: 'unit',
      unit: unit,
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.NumberFormat(locale, formatOptions).format(value);
    } catch (error) {
      console.error('Unit formatting error:', error);
      return `${value} ${unit}`;
    }
  }
  
  formatTemperature(value, unit = 'celsius', locale = 'en-US') {
    try {
      return new Intl.NumberFormat(locale, {
        style: 'unit',
        unit: unit,
        unitDisplay: 'short'
      }).format(value);
    } catch (error) {
      console.error('Temperature formatting error:', error);
      return `${value}°${unit === 'celsius' ? 'C' : 'F'}`;
    }
  }
  
  convertTemperature(value, fromUnit, toUnit) {
    if (fromUnit === toUnit) return value;
    
    // Convertir a Celsius primero
    let celsius;
    if (fromUnit === 'fahrenheit') {
      celsius = (value - 32) * 5 / 9;
    } else if (fromUnit === 'kelvin') {
      celsius = value - 273.15;
    } else {
      celsius = value;
    }
    
    // Convertir a unidad destino
    if (toUnit === 'fahrenheit') {
      return celsius * 9 / 5 + 32;
    } else if (toUnit === 'kelvin') {
      return celsius + 273.15;
    } else {
      return celsius;
    }
  }
  
  convertLength(value, fromUnit, toUnit) {
    const conversions = {
      'meter': 1,
      'kilometer': 1000,
      'centimeter': 0.01,
      'inch': 0.0254,
      'foot': 0.3048,
      'yard': 0.9144,
      'mile': 1609.34,
    };
    
    const meters = value * conversions[fromUnit];
    return meters / conversions[toUnit];
  }
  
  convertWeight(value, fromUnit, toUnit) {
    const conversions = {
      'gram': 1,
      'kilogram': 1000,
      'pound': 453.592,
      'ounce': 28.3495,
    };
    
    const grams = value * conversions[fromUnit];
    return grams / conversions[toUnit];
  }
}

export const unitFormatter = new UnitFormatter();
```

#### **Ejemplos de Formateo de Unidades**

```javascript
// Ejemplos de uso
const value = 25.5;

// Formateo de unidades
console.log(unitFormatter.formatUnit(value, 'kilometer', 'en-US')); // "25.5 km"
console.log(unitFormatter.formatUnit(value, 'kilometer', 'es-ES')); // "25,5 km"
console.log(unitFormatter.formatUnit(value, 'kilogram', 'en-US')); // "25.5 kg"
console.log(unitFormatter.formatUnit(value, 'kilogram', 'es-ES')); // "25,5 kg"

// Formateo de temperatura
console.log(unitFormatter.formatTemperature(value, 'celsius', 'en-US')); // "25.5°C"
console.log(unitFormatter.formatTemperature(value, 'fahrenheit', 'en-US')); // "25.5°F"

// Conversión de unidades
console.log(unitFormatter.convertTemperature(25, 'celsius', 'fahrenheit')); // 77
console.log(unitFormatter.convertLength(1000, 'meter', 'kilometer')); // 1
console.log(unitFormatter.convertWeight(1000, 'gram', 'kilogram')); // 1
```

### **4. Calendarios y Zonas Horarias**

#### **Manejo de Zonas Horarias**

```javascript
// utils/timezoneFormatter.js
class TimezoneFormatter {
  constructor() {
    this.timezones = {
      'UTC': 'UTC',
      'America/New_York': 'America/New_York',
      'America/Los_Angeles': 'America/Los_Angeles',
      'Europe/London': 'Europe/London',
      'Europe/Paris': 'Europe/Paris',
      'Asia/Tokyo': 'Asia/Tokyo',
      'Asia/Shanghai': 'Asia/Shanghai',
      'Australia/Sydney': 'Australia/Sydney',
    };
  }
  
  formatInTimezone(date, timezone, locale = 'en-US', options = {}) {
    const defaultOptions = {
      timeZone: timezone,
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
    };
    
    const formatOptions = { ...defaultOptions, ...options };
    
    try {
      return new Intl.DateTimeFormat(locale, formatOptions).format(date);
    } catch (error) {
      console.error('Timezone formatting error:', error);
      return date.toLocaleString();
    }
  }
  
  getTimezoneOffset(timezone) {
    try {
      const now = new Date();
      const utc = new Date(now.getTime() + (now.getTimezoneOffset() * 60000));
      const target = new Date(utc.toLocaleString('en-US', { timeZone: timezone }));
      return (target.getTime() - utc.getTime()) / 60000;
    } catch (error) {
      console.error('Timezone offset error:', error);
      return 0;
    }
  }
  
  convertTimezone(date, fromTimezone, toTimezone) {
    try {
      const utc = new Date(date.getTime() + (date.getTimezoneOffset() * 60000));
      const target = new Date(utc.toLocaleString('en-US', { timeZone: toTimezone }));
      return target;
    } catch (error) {
      console.error('Timezone conversion error:', error);
      return date;
    }
  }
  
  getAvailableTimezones() {
    return Object.keys(this.timezones);
  }
  
  getTimezoneInfo(timezone) {
    try {
      const now = new Date();
      const offset = this.getTimezoneOffset(timezone);
      const formatted = this.formatInTimezone(now, timezone, 'en-US', {
        timeZoneName: 'long'
      });
      
      return {
        timezone,
        offset,
        formatted,
        isDST: this.isDST(now, timezone)
      };
    } catch (error) {
      console.error('Timezone info error:', error);
      return null;
    }
  }
  
  isDST(date, timezone) {
    try {
      const jan = new Date(date.getFullYear(), 0, 1);
      const jul = new Date(date.getFullYear(), 6, 1);
      
      const janOffset = this.getTimezoneOffset(timezone);
      const julOffset = this.getTimezoneOffset(timezone);
      
      return janOffset !== julOffset;
    } catch (error) {
      console.error('DST check error:', error);
      return false;
    }
  }
}

export const timezoneFormatter = new TimezoneFormatter();
```

#### **Componente de Zona Horaria**

```javascript
// components/TimezoneDisplay.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useTranslation } from 'react-i18next';
import { timezoneFormatter } from '../utils/timezoneFormatter';

const TimezoneDisplay = ({ 
  timezone, 
  style,
  showOffset = true,
  showDST = true 
}) => {
  const { i18n } = useTranslation();
  const [currentTime, setCurrentTime] = useState(new Date());
  const [timezoneInfo, setTimezoneInfo] = useState(null);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCurrentTime(new Date());
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  useEffect(() => {
    if (timezone) {
      const info = timezoneFormatter.getTimezoneInfo(timezone);
      setTimezoneInfo(info);
    }
  }, [timezone]);
  
  const formatTime = () => {
    if (!timezone) return currentTime.toLocaleTimeString();
    
    return timezoneFormatter.formatInTimezone(
      currentTime, 
      timezone, 
      i18n.language,
      {
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        timeZoneName: 'short'
      }
    );
  };
  
  const formatDate = () => {
    if (!timezone) return currentTime.toLocaleDateString();
    
    return timezoneFormatter.formatInTimezone(
      currentTime, 
      timezone, 
      i18n.language,
      {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      }
    );
  };
  
  return (
    <View style={[styles.container, style]}>
      <Text style={styles.time}>{formatTime()}</Text>
      <Text style={styles.date}>{formatDate()}</Text>
      
      {showOffset && timezoneInfo && (
        <Text style={styles.offset}>
          UTC{timezoneInfo.offset >= 0 ? '+' : ''}{timezoneInfo.offset / 60}
        </Text>
      )}
      
      {showDST && timezoneInfo && timezoneInfo.isDST && (
        <Text style={styles.dst}>DST</Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
  },
  time: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  date: {
    fontSize: 16,
    color: '#666',
    marginTop: 4,
  },
  offset: {
    fontSize: 12,
    color: '#999',
    marginTop: 2,
  },
  dst: {
    fontSize: 12,
    color: '#007AFF',
    marginTop: 2,
  },
});

export default TimezoneDisplay;
```

### **5. Adaptación de Formatos**

#### **Sistema de Preferencias Regionales**

```javascript
// utils/regionalPreferences.js
class RegionalPreferences {
  constructor() {
    this.preferences = {
      'en-US': {
        dateFormat: 'MM/DD/YYYY',
        timeFormat: '12h',
        currency: 'USD',
        temperature: 'fahrenheit',
        weight: 'pound',
        length: 'foot',
        numberFormat: '1,234.56',
      },
      'es-ES': {
        dateFormat: 'DD/MM/YYYY',
        timeFormat: '24h',
        currency: 'EUR',
        temperature: 'celsius',
        weight: 'kilogram',
        length: 'meter',
        numberFormat: '1.234,56',
      },
      'fr-FR': {
        dateFormat: 'DD/MM/YYYY',
        timeFormat: '24h',
        currency: 'EUR',
        temperature: 'celsius',
        weight: 'kilogram',
        length: 'meter',
        numberFormat: '1 234,56',
      },
      'de-DE': {
        dateFormat: 'DD.MM.YYYY',
        timeFormat: '24h',
        currency: 'EUR',
        temperature: 'celsius',
        weight: 'kilogram',
        length: 'meter',
        numberFormat: '1.234,56',
      },
    };
  }
  
  getPreferences(locale) {
    return this.preferences[locale] || this.preferences['en-US'];
  }
  
  updatePreferences(locale, newPreferences) {
    if (this.preferences[locale]) {
      this.preferences[locale] = { ...this.preferences[locale], ...newPreferences };
    }
  }
  
  getDateFormat(locale) {
    return this.getPreferences(locale).dateFormat;
  }
  
  getTimeFormat(locale) {
    return this.getPreferences(locale).timeFormat;
  }
  
  getCurrency(locale) {
    return this.getPreferences(locale).currency;
  }
  
  getTemperatureUnit(locale) {
    return this.getPreferences(locale).temperature;
  }
  
  getWeightUnit(locale) {
    return this.getPreferences(locale).weight;
  }
  
  getLengthUnit(locale) {
    return this.getPreferences(locale).length;
  }
  
  getNumberFormat(locale) {
    return this.getPreferences(locale).numberFormat;
  }
}

export const regionalPreferences = new RegionalPreferences();
```

#### **Componente de Configuración Regional**

```javascript
// components/RegionalSettings.js
import React, { useState, useEffect } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  TouchableOpacity, 
  ScrollView 
} from 'react-native';
import { useTranslation } from 'react-i18next';
import { regionalPreferences } from '../utils/regionalPreferences';

const RegionalSettings = () => {
  const { i18n } = useTranslation();
  const [preferences, setPreferences] = useState({});
  
  useEffect(() => {
    const currentPreferences = regionalPreferences.getPreferences(i18n.language);
    setPreferences(currentPreferences);
  }, [i18n.language]);
  
  const updatePreference = (key, value) => {
    const newPreferences = { ...preferences, [key]: value };
    setPreferences(newPreferences);
    regionalPreferences.updatePreferences(i18n.language, newPreferences);
  };
  
  const renderOption = (title, key, options) => (
    <View style={styles.optionContainer}>
      <Text style={styles.optionTitle}>{title}</Text>
      <View style={styles.optionsRow}>
        {options.map((option) => (
          <TouchableOpacity
            key={option.value}
            style={[
              styles.optionButton,
              preferences[key] === option.value && styles.selectedOption
            ]}
            onPress={() => updatePreference(key, option.value)}
          >
            <Text style={[
              styles.optionText,
              preferences[key] === option.value && styles.selectedOptionText
            ]}>
              {option.label}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
    </View>
  );
  
  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Regional Settings</Text>
      
      {renderOption('Date Format', 'dateFormat', [
        { value: 'MM/DD/YYYY', label: 'MM/DD/YYYY' },
        { value: 'DD/MM/YYYY', label: 'DD/MM/YYYY' },
        { value: 'YYYY-MM-DD', label: 'YYYY-MM-DD' },
      ])}
      
      {renderOption('Time Format', 'timeFormat', [
        { value: '12h', label: '12 Hour' },
        { value: '24h', label: '24 Hour' },
      ])}
      
      {renderOption('Currency', 'currency', [
        { value: 'USD', label: 'USD' },
        { value: 'EUR', label: 'EUR' },
        { value: 'GBP', label: 'GBP' },
        { value: 'JPY', label: 'JPY' },
      ])}
      
      {renderOption('Temperature', 'temperature', [
        { value: 'celsius', label: 'Celsius' },
        { value: 'fahrenheit', label: 'Fahrenheit' },
      ])}
      
      {renderOption('Weight', 'weight', [
        { value: 'kilogram', label: 'Kilogram' },
        { value: 'pound', label: 'Pound' },
      ])}
      
      {renderOption('Length', 'length', [
        { value: 'meter', label: 'Meter' },
        { value: 'foot', label: 'Foot' },
      ])}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    margin: 20,
  },
  optionContainer: {
    backgroundColor: '#fff',
    margin: 20,
    padding: 20,
    borderRadius: 12,
  },
  optionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  optionsRow: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    gap: 12,
  },
  optionButton: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 20,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  selectedOption: {
    backgroundColor: '#007AFF',
    borderColor: '#007AFF',
  },
  optionText: {
    fontSize: 14,
    color: '#333',
  },
  selectedOptionText: {
    color: '#fff',
  },
});

export default RegionalSettings;
```

---

## 🛠️ Implementación Práctica

### **Ejercicio 1: Dashboard de Formatos Regionales**

```javascript
// screens/RegionalDashboard.js
import React, { useState, useEffect } from 'react';
import { 
  View, 
  Text, 
  StyleSheet, 
  ScrollView,
  TouchableOpacity 
} from 'react-native';
import { useTranslation } from 'react-i18next';
import { dateFormatter } from '../utils/dateFormatter';
import { currencyFormatter } from '../utils/currencyFormatter';
import { unitFormatter } from '../utils/unitFormatter';
import { timezoneFormatter } from '../utils/timezoneFormatter';

const RegionalDashboard = () => {
  const { i18n } = useTranslation();
  const [currentTime, setCurrentTime] = useState(new Date());
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCurrentTime(new Date());
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  const locale = i18n.language;
  
  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Regional Formats</Text>
      
      {/* Fechas */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Dates</Text>
        <Text style={styles.example}>
          Date: {dateFormatter.formatDate(currentTime, locale)}
        </Text>
        <Text style={styles.example}>
          Time: {dateFormatter.formatTime(currentTime, locale)}
        </Text>
        <Text style={styles.example}>
          DateTime: {dateFormatter.formatDateTime(currentTime, locale)}
        </Text>
        <Text style={styles.example}>
          Relative: {dateFormatter.getRelativeTime(currentTime, locale)}
        </Text>
      </View>
      
      {/* Monedas */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Currency</Text>
        <Text style={styles.example}>
          USD: {currencyFormatter.formatCurrency(1234.56, 'USD', locale)}
        </Text>
        <Text style={styles.example}>
          EUR: {currencyFormatter.formatCurrency(1234.56, 'EUR', locale)}
        </Text>
        <Text style={styles.example}>
          Number: {currencyFormatter.formatNumber(1234.56, locale)}
        </Text>
        <Text style={styles.example}>
          Percent: {currencyFormatter.formatPercent(12.34, locale)}
        </Text>
        <Text style={styles.example}>
          Compact: {currencyFormatter.formatCompactNumber(1234567, locale)}
        </Text>
      </View>
      
      {/* Unidades */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Units</Text>
        <Text style={styles.example}>
          Length: {unitFormatter.formatUnit(25.5, 'kilometer', locale)}
        </Text>
        <Text style={styles.example}>
          Weight: {unitFormatter.formatUnit(25.5, 'kilogram', locale)}
        </Text>
        <Text style={styles.example}>
          Temperature: {unitFormatter.formatTemperature(25, 'celsius', locale)}
        </Text>
      </View>
      
      {/* Zonas Horarias */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Timezones</Text>
        {['UTC', 'America/New_York', 'Europe/London', 'Asia/Tokyo'].map(timezone => (
          <Text key={timezone} style={styles.example}>
            {timezone}: {timezoneFormatter.formatInTimezone(currentTime, timezone, locale)}
          </Text>
        ))}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    margin: 20,
  },
  section: {
    backgroundColor: '#fff',
    margin: 20,
    padding: 20,
    borderRadius: 12,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 16,
  },
  example: {
    fontSize: 16,
    marginBottom: 8,
    color: '#333',
  },
});

export default RegionalDashboard;
```

### **Ejercicio 2: Componente de Formato Inteligente**

```javascript
// components/SmartFormatter.js
import React from 'react';
import { Text, StyleSheet } from 'react-native';
import { useTranslation } from 'react-i18next';
import { dateFormatter } from '../utils/dateFormatter';
import { currencyFormatter } from '../utils/currencyFormatter';
import { unitFormatter } from '../utils/unitFormatter';

const SmartFormatter = ({ 
  value, 
  type, 
  options = {},
  style 
}) => {
  const { i18n } = useTranslation();
  const locale = i18n.language;
  
  const formatValue = () => {
    switch (type) {
      case 'date':
        return dateFormatter.formatDate(value, locale, options);
      case 'time':
        return dateFormatter.formatTime(value, locale, options);
      case 'datetime':
        return dateFormatter.formatDateTime(value, locale, options);
      case 'relative':
        return dateFormatter.getRelativeTime(value, locale);
      case 'currency':
        return currencyFormatter.formatCurrency(value, options.currency, locale, options);
      case 'number':
        return currencyFormatter.formatNumber(value, locale, options);
      case 'percent':
        return currencyFormatter.formatPercent(value, locale, options);
      case 'compact':
        return currencyFormatter.formatCompactNumber(value, locale);
      case 'unit':
        return unitFormatter.formatUnit(value, options.unit, locale, options);
      case 'temperature':
        return unitFormatter.formatTemperature(value, options.unit, locale);
      default:
        return value.toString();
    }
  };
  
  return (
    <Text style={style}>
      {formatValue()}
    </Text>
  );
};

export default SmartFormatter;
```

### **Ejercicio 3: Hook de Formateo Regional**

```javascript
// hooks/useRegionalFormatting.js
import { useMemo } from 'react';
import { useTranslation } from 'react-i18next';
import { dateFormatter } from '../utils/dateFormatter';
import { currencyFormatter } from '../utils/currencyFormatter';
import { unitFormatter } from '../utils/unitFormatter';
import { timezoneFormatter } from '../utils/timezoneFormatter';

export const useRegionalFormatting = () => {
  const { i18n } = useTranslation();
  const locale = i18n.language;
  
  const formatters = useMemo(() => ({
    // Formateo de fechas
    formatDate: (date, options = {}) => 
      dateFormatter.formatDate(date, locale, options),
    
    formatTime: (date, options = {}) => 
      dateFormatter.formatTime(date, locale, options),
    
    formatDateTime: (date, options = {}) => 
      dateFormatter.formatDateTime(date, locale, options),
    
    getRelativeTime: (date) => 
      dateFormatter.getRelativeTime(date, locale),
    
    // Formateo de moneda
    formatCurrency: (amount, currency = 'USD', options = {}) => 
      currencyFormatter.formatCurrency(amount, currency, locale, options),
    
    formatNumber: (number, options = {}) => 
      currencyFormatter.formatNumber(number, locale, options),
    
    formatPercent: (value, options = {}) => 
      currencyFormatter.formatPercent(value, locale, options),
    
    formatCompactNumber: (number) => 
      currencyFormatter.formatCompactNumber(number, locale),
    
    // Formateo de unidades
    formatUnit: (value, unit, options = {}) => 
      unitFormatter.formatUnit(value, unit, locale, options),
    
    formatTemperature: (value, unit = 'celsius') => 
      unitFormatter.formatTemperature(value, unit, locale),
    
    // Formateo de zona horaria
    formatInTimezone: (date, timezone, options = {}) => 
      timezoneFormatter.formatInTimezone(date, timezone, locale, options),
    
    // Conversiones
    convertTemperature: (value, fromUnit, toUnit) => 
      unitFormatter.convertTemperature(value, fromUnit, toUnit),
    
    convertLength: (value, fromUnit, toUnit) => 
      unitFormatter.convertLength(value, fromUnit, toUnit),
    
    convertWeight: (value, fromUnit, toUnit) => 
      unitFormatter.convertWeight(value, fromUnit, toUnit),
  }), [locale]);
  
  return formatters;
};
```

---

## 🧪 Testing

### **Testing de Formateo Regional**

```javascript
// __tests__/regionalFormatting.test.js
import { dateFormatter } from '../utils/dateFormatter';
import { currencyFormatter } from '../utils/currencyFormatter';
import { unitFormatter } from '../utils/unitFormatter';

describe('Regional Formatting', () => {
  const testDate = new Date('2024-01-15T14:30:00Z');
  
  test('formats dates correctly for different locales', () => {
    const enDate = dateFormatter.formatDate(testDate, 'en-US');
    const esDate = dateFormatter.formatDate(testDate, 'es-ES');
    const frDate = dateFormatter.formatDate(testDate, 'fr-FR');
    
    expect(enDate).toContain('January');
    expect(esDate).toContain('enero');
    expect(frDate).toContain('janvier');
  });
  
  test('formats currency correctly for different locales', () => {
    const amount = 1234.56;
    
    const usd = currencyFormatter.formatCurrency(amount, 'USD', 'en-US');
    const eur = currencyFormatter.formatCurrency(amount, 'EUR', 'es-ES');
    
    expect(usd).toContain('$');
    expect(eur).toContain('€');
  });
  
  test('formats units correctly for different locales', () => {
    const value = 25.5;
    
    const enUnit = unitFormatter.formatUnit(value, 'kilometer', 'en-US');
    const esUnit = unitFormatter.formatUnit(value, 'kilometer', 'es-ES');
    
    expect(enUnit).toContain('km');
    expect(esUnit).toContain('km');
  });
  
  test('converts units correctly', () => {
    const celsius = 25;
    const fahrenheit = unitFormatter.convertTemperature(celsius, 'celsius', 'fahrenheit');
    
    expect(fahrenheit).toBeCloseTo(77, 1);
  });
});
```

---

## 📱 Ejemplo Completo

### **Aplicación de Formatos Regionales**

```javascript
// App.js - Aplicación con formatos regionales
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useTranslation } from 'react-i18next';
import './i18n';

import WelcomeScreen from './screens/WelcomeScreen';
import RegionalDashboard from './screens/RegionalDashboard';
import RegionalSettings from './components/RegionalSettings';

const Stack = createStackNavigator();

const App = () => {
  const { i18n } = useTranslation();
  
  useEffect(() => {
    // Configurar formatos regionales basados en idioma
    console.log('Current locale:', i18n.language);
  }, [i18n.language]);
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Welcome" component={WelcomeScreen} />
        <Stack.Screen name="Regional" component={RegionalDashboard} />
        <Stack.Screen name="Settings" component={RegionalSettings} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 🎯 Resumen de la Clase

### **Conceptos Clave Aprendidos**

1. **Formateo de fechas**: Adaptación a convenciones regionales
2. **Formateo de moneda**: Soporte para múltiples monedas y formatos
3. **Unidades de medida**: Conversión y formateo de unidades
4. **Zonas horarias**: Manejo de tiempo global
5. **Preferencias regionales**: Configuración personalizada

### **Habilidades Desarrolladas**

- ✅ Formateo de fechas según región
- ✅ Configuración de formatos de moneda
- ✅ Implementación de unidades de medida
- ✅ Manejo de calendarios y zonas horarias
- ✅ Adaptación de formatos según preferencias

### **Próximos Pasos**

En la siguiente clase aprenderás sobre:
- Testing de aplicaciones multilingües
- Validación de traducciones
- Performance de aplicaciones i18n
- Debugging de problemas de localización

---

## 📚 Recursos Adicionales

### **APIs de Internacionalización**
- [Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat)
- [Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat)
- [Intl.RelativeTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat)

### **Herramientas de Formateo**
- [Moment.js](https://momentjs.com/) - Biblioteca de fechas
- [date-fns](https://date-fns.org/) - Biblioteca de fechas moderna
- [numeral.js](https://numeraljs.com/) - Biblioteca de formateo de números

### **Recursos de Localización**
- [CLDR](https://cldr.unicode.org/) - Common Locale Data Repository
- [ICU](http://site.icu-project.org/) - International Components for Unicode
- [i18n Best Practices](https://www.i18next.com/overview/best-practices)

---

**🎯 Objetivo Alcanzado**: Has aprendido a implementar formatos regionales completos, adaptando tu aplicación a las convenciones culturales de diferentes regiones del mundo.

**💡 Consejo**: Los formatos regionales no son solo estéticos, sino que reflejan las expectativas culturales de los usuarios. Es fundamental para la aceptación de tu aplicación en mercados internacionales.

**🚀 ¡Continúa con la siguiente clase para aprender sobre testing y optimización de aplicaciones multilingües!**
