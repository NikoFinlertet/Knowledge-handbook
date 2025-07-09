# 📍 Geolocation API - Определение местоположения

> **Geolocation API** - браузерный интерфейс для получения географических координат пользователя с учетом приватности и UX.

## 📋 Содержание

- [Основы геолокации](#-основы-геолокации)
- [Продвинутые паттерны](#-продвинутые-паттерны)
- [UX и приватность](#-ux-и-приватность)
- [Практические применения](#-практические-применения)

---

## 🌍 Основы геолокации

### Получение текущей позиции

```javascript
// Класс для работы с геолокацией
class GeolocationService {
  constructor() {
    this.defaultOptions = {
      enableHighAccuracy: true,
      timeout: 10000,
      maximumAge: 300000 // 5 минут
    };
  }
  
  // Проверка поддержки API
  isSupported() {
    return 'geolocation' in navigator;
  }
  
  // Получение текущей позиции
  async getCurrentPosition(options = {}) {
    if (!this.isSupported()) {
      throw new Error('Geolocation API не поддерживается');
    }
    
    const finalOptions = { ...this.defaultOptions, ...options };
    
    return new Promise((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(
        position => resolve(this.formatPosition(position)),
        error => reject(this.formatError(error)),
        finalOptions
      );
    });
  }
  
  // Отслеживание позиции
  watchPosition(callback, errorCallback, options = {}) {
    if (!this.isSupported()) {
      errorCallback(new Error('Geolocation API не поддерживается'));
      return null;
    }
    
    const finalOptions = { ...this.defaultOptions, ...options };
    
    return navigator.geolocation.watchPosition(
      position => callback(this.formatPosition(position)),
      error => errorCallback(this.formatError(error)),
      finalOptions
    );
  }
  
  // Остановка отслеживания
  clearWatch(watchId) {
    if (watchId) {
      navigator.geolocation.clearWatch(watchId);
    }
  }
  
  // Форматирование позиции
  formatPosition(position) {
    return {
      latitude: position.coords.latitude,
      longitude: position.coords.longitude,
      accuracy: position.coords.accuracy,
      altitude: position.coords.altitude,
      altitudeAccuracy: position.coords.altitudeAccuracy,
      heading: position.coords.heading,
      speed: position.coords.speed,
      timestamp: position.timestamp
    };
  }
  
  // Форматирование ошибок
  formatError(error) {
    const errorMessages = {
      1: 'Доступ к геолокации запрещен',
      2: 'Позиция недоступна',
      3: 'Время ожидания истекло'
    };
    
    return new Error(errorMessages[error.code] || 'Неизвестная ошибка геолокации');
  }
  
  // Расчет расстояния между точками (формула Haversine)
  calculateDistance(lat1, lon1, lat2, lon2) {
    const R = 6371; // Радиус Земли в км
    const dLat = this.degToRad(lat2 - lat1);
    const dLon = this.degToRad(lon2 - lon1);
    const a = 
      Math.sin(dLat/2) * Math.sin(dLat/2) +
      Math.cos(this.degToRad(lat1)) * Math.cos(this.degToRad(lat2)) * 
      Math.sin(dLon/2) * Math.sin(dLon/2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    return R * c;
  }
  
  degToRad(deg) {
    return deg * (Math.PI/180);
  }
}

// Использование геолокации
const geoService = new GeolocationService();

// Пример получения местоположения
async function getUserLocation() {
  try {
    const position = await geoService.getCurrentPosition({
      enableHighAccuracy: true,
      timeout: 5000
    });
    
    console.log('Координаты пользователя:', position);
    return position;
  } catch (error) {
    console.error('Ошибка получения координат:', error);
    throw error;
  }
}
```

---

## 🎯 Продвинутые паттерны

### Геозоны и уведомления

```javascript
// Система геозон
class GeofenceManager {
  constructor(geolocationService) {
    this.geoService = geolocationService;
    this.geofences = new Map();
    this.watchId = null;
    this.currentPosition = null;
  }
  
  // Добавление геозоны
  addGeofence(id, center, radius, callbacks) {
    this.geofences.set(id, {
      center: center,
      radius: radius,
      callbacks: callbacks,
      isInside: false
    });
  }
  
  // Начало мониторинга
  startMonitoring() {
    this.watchId = this.geoService.watchPosition(
      position => this.checkGeofences(position),
      error => console.error('Ошибка отслеживания:', error),
      { enableHighAccuracy: true }
    );
  }
  
  // Остановка мониторинга
  stopMonitoring() {
    if (this.watchId) {
      this.geoService.clearWatch(this.watchId);
      this.watchId = null;
    }
  }
  
  // Проверка геозон
  checkGeofences(position) {
    this.currentPosition = position;
    
    this.geofences.forEach((geofence, id) => {
      const distance = this.geoService.calculateDistance(
        position.latitude, position.longitude,
        geofence.center.lat, geofence.center.lng
      ) * 1000; // переводим в метры
      
      const isCurrentlyInside = distance <= geofence.radius;
      const wasInside = geofence.isInside;
      
      if (isCurrentlyInside && !wasInside) {
        // Вход в геозону
        geofence.callbacks.onEnter?.(id, position);
        geofence.isInside = true;
      } else if (!isCurrentlyInside && wasInside) {
        // Выход из геозоны
        geofence.callbacks.onExit?.(id, position);
        geofence.isInside = false;
      }
    });
  }
}

// Пример использования геозон
const geofenceManager = new GeofenceManager(geoService);

// Добавляем геозону вокруг офиса
geofenceManager.addGeofence('office', 
  { lat: 55.7558, lng: 37.6173 }, // Москва, Красная площадь
  100, // 100 метров
  {
    onEnter: (id, position) => {
      showNotification('Добро пожаловать в офис!');
      logEvent('entered_office', position);
    },
    onExit: (id, position) => {
      showNotification('До свидания!');
      logEvent('left_office', position);
    }
  }
);

geofenceManager.startMonitoring();
```

---

## 🔐 UX и приватность

### Запрос разрешений с объяснением

```javascript
// Менеджер разрешений для геолокации
class LocationPermissionManager {
  constructor() {
    this.permissionStatus = null;
  }
  
  // Проверка текущего статуса разрешения
  async checkPermissionStatus() {
    if ('permissions' in navigator) {
      try {
        const permission = await navigator.permissions.query({ name: 'geolocation' });
        this.permissionStatus = permission.state;
        return permission.state;
      } catch (error) {
        console.warn('Permissions API недоступен');
      }
    }
    
    return 'unknown';
  }
  
  // Запрос доступа с пояснением
  async requestLocationAccess(options = {}) {
    const {
      title = 'Доступ к местоположению',
      message = 'Для корректной работы приложения нужен доступ к вашему местоположению',
      showRationale = true
    } = options;
    
    const status = await this.checkPermissionStatus();
    
    if (status === 'granted') {
      return true;
    }
    
    if (status === 'denied') {
      this.showPermissionDeniedDialog();
      return false;
    }
    
    if (showRationale) {
      const userAgreed = await this.showLocationRationale(title, message);
      if (!userAgreed) {
        return false;
      }
    }
    
    try {
      // Пытаемся получить позицию для запроса разрешения
      await geoService.getCurrentPosition({ timeout: 1000 });
      return true;
    } catch (error) {
      if (error.message.includes('запрещен')) {
        this.showPermissionDeniedDialog();
      }
      return false;
    }
  }
  
  // Объяснение необходимости доступа
  showLocationRationale(title, message) {
    return new Promise(resolve => {
      const modal = document.createElement('div');
      modal.className = 'location-permission-modal';
      modal.innerHTML = `
        <div class=\"modal-content\">
          <h3>${title}</h3>
          <p>${message}</p>
          <div class=\"modal-actions\">
            <button id=\"allow-location\">Разрешить</button>
            <button id=\"deny-location\">Не сейчас</button>
          </div>
        </div>
      `;
      
      document.body.appendChild(modal);
      
      modal.querySelector('#allow-location').onclick = () => {
        document.body.removeChild(modal);
        resolve(true);
      };
      
      modal.querySelector('#deny-location').onclick = () => {
        document.body.removeChild(modal);
        resolve(false);
      };
    });
  }
  
  // Диалог для случая отказа
  showPermissionDeniedDialog() {
    const message = `
      Доступ к геолокации запрещен. 
      Для включения:
      1. Нажмите на иконку замка в адресной строке
      2. Разрешите доступ к местоположению
      3. Обновите страницу
    `;
    
    alert(message);
  }
}

// Использование менеджера разрешений
const permissionManager = new LocationPermissionManager();

async function initLocationFeatures() {
  const hasPermission = await permissionManager.requestLocationAccess({
    title: 'Найти ближайшие кафе',
    message: 'Мы покажем кафе рядом с вами для удобного заказа'
  });
  
  if (hasPermission) {
    const position = await getUserLocation();
    loadNearbyPlaces(position);
  } else {
    showFallbackLocationInput();
  }
}
```

---

## 🎯 Практические применения

### Поиск ближайших объектов

```javascript
// Сервис поиска ближайших мест
class NearbyPlacesService {
  constructor(geolocationService) {
    this.geoService = geolocationService;
  }
  
  // Поиск ближайших мест
  async findNearbyPlaces(category, radius = 1000) {
    try {
      const position = await this.geoService.getCurrentPosition();
      
      const response = await fetch('/api/places/nearby', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          latitude: position.latitude,
          longitude: position.longitude,
          category: category,
          radius: radius
        })
      });
      
      const places = await response.json();
      
      // Добавляем расстояние до каждого места
      return places.map(place => ({
        ...place,
        distance: this.geoService.calculateDistance(
          position.latitude, position.longitude,
          place.latitude, place.longitude
        )
      })).sort((a, b) => a.distance - b.distance);
      
    } catch (error) {
      console.error('Ошибка поиска мест:', error);
      throw error;
    }
  }
  
  // Построение маршрута
  async getDirections(destination) {
    const origin = await this.geoService.getCurrentPosition();
    
    const directionsUrl = `https://www.google.com/maps/dir/${origin.latitude},${origin.longitude}/${destination.latitude},${destination.longitude}`;
    
    return directionsUrl;
  }
}

// Отслеживание движения для фитнес-приложения
class ActivityTracker {
  constructor(geolocationService) {
    this.geoService = geolocationService;
    this.isTracking = false;
    this.positions = [];
    this.startTime = null;
  }
  
  startTracking() {
    this.isTracking = true;
    this.startTime = Date.now();
    this.positions = [];
    
    this.watchId = this.geoService.watchPosition(
      position => this.recordPosition(position),
      error => console.error('Ошибка трекинга:', error),
      { enableHighAccuracy: true, timeout: 30000 }
    );
  }
  
  stopTracking() {
    this.isTracking = false;
    if (this.watchId) {
      this.geoService.clearWatch(this.watchId);
    }
    
    return this.getActivitySummary();
  }
  
  recordPosition(position) {
    if (this.isTracking) {
      this.positions.push({
        ...position,
        timestamp: Date.now()
      });
    }
  }
  
  getActivitySummary() {
    if (this.positions.length < 2) {
      return { distance: 0, duration: 0, averageSpeed: 0 };
    }
    
    let totalDistance = 0;
    
    for (let i = 1; i < this.positions.length; i++) {
      const prev = this.positions[i - 1];
      const curr = this.positions[i];
      
      totalDistance += this.geoService.calculateDistance(
        prev.latitude, prev.longitude,
        curr.latitude, curr.longitude
      );
    }
    
    const duration = (Date.now() - this.startTime) / 1000 / 60; // в минутах
    const averageSpeed = duration > 0 ? (totalDistance / duration) * 60 : 0; // км/ч
    
    return {
      distance: Math.round(totalDistance * 100) / 100, // км
      duration: Math.round(duration), // минуты
      averageSpeed: Math.round(averageSpeed * 10) / 10, // км/ч
      positions: this.positions
    };
  }
}

// Пример использования
const nearbyPlaces = new NearbyPlacesService(geoService);
const activityTracker = new ActivityTracker(geoService);

// Поиск ближайших кафе
async function findNearestCafes() {
  try {
    const cafes = await nearbyPlaces.findNearbyPlaces('cafe', 500);
    displayPlaces(cafes);
  } catch (error) {
    showError('Не удалось найти кафе поблизости');
  }
}
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **Location Service** - Класс для получения координат с обработкой ошибок  
2. **Distance Calculator** - Расчет расстояний между точками
3. **Permission Manager** - UX для запроса разрешений

### 🟡 Средний уровень  
4. **Geofence System** - Система геозон с уведомлениями
5. **Nearby Places** - Поиск объектов поблизости
6. **Route Tracking** - Отслеживание маршрута

### 🔴 Продвинутый уровень
7. **Activity Tracker** - Фитнес-трекер с анализом активности
8. **Location History** - История перемещений с визуализацией  
9. **Smart Notifications** - Контекстные уведомления по местоположению

## 🔗 Связанные разделы

- **[[storage-apis|💾 Storage APIs]]** - Сохранение истории местоположений
- **[[service-workers-pwa#notifications|🚀 PWA Notifications]]** - Push-уведомления по геозонам
- **[[intersection-observer|👁️ Intersection Observer]]** - Оптимизация карт
- **[[../../technical-skills/security|🔒 Security]]** - Приватность геоданных

---

⏱️ **Время изучения**: 2-3 часа  
🎯 **Уровень**: Базовый - Средний  
📋 **Предварительные требования**: JavaScript ES6+, основы математики, UX принципы 