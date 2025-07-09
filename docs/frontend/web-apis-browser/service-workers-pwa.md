# 🚀 Service Workers & PWA - Прогрессивные веб-приложения

> **Service Workers** - мощный инструмент для создания прогрессивных веб-приложений с офлайн-функциональностью, фоновой синхронизацией и push-уведомлениями.

## 📋 Содержание

- [Основы Service Workers](#-основы-service-workers)
- [Стратегии кэширования](#-стратегии-кэширования)
- [PWA и Web App Manifest](#-pwa-и-web-app-manifest)
- [Push-уведомления](#-push-уведомления)

---

## ⚙️ Основы Service Workers

### Регистрация и жизненный цикл

```javascript
// Менеджер Service Worker
class ServiceWorkerManager {
  constructor() {
    this.registration = null;
    this.isSupported = 'serviceWorker' in navigator;
  }
  
  // Регистрация Service Worker
  async register(scriptURL, options = {}) {
    if (!this.isSupported) {
      throw new Error('Service Workers не поддерживаются');
    }
    
    try {
      this.registration = await navigator.serviceWorker.register(scriptURL, {
        scope: '/',
        ...options
      });
      
      console.log('Service Worker зарегистрирован:', this.registration.scope);
      
      // Слушаем обновления
      this.registration.addEventListener('updatefound', () => {
        this.handleUpdate();
      });
      
      // Проверяем статус
      this.checkServiceWorkerState();
      
      return this.registration;
    } catch (error) {
      console.error('Ошибка регистрации Service Worker:', error);
      throw error;
    }
  }
  
  // Обработка обновлений
  handleUpdate() {
    const newWorker = this.registration.installing;
    
    newWorker.addEventListener('statechange', () => {
      if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
        // Новая версия доступна
        this.showUpdateAvailable();
      }
    });
  }
  
  showUpdateAvailable() {
    if (confirm('Доступна новая версия приложения. Обновить?')) {
      window.location.reload();
    }
  }
  
  // Проверка состояния Service Worker
  checkServiceWorkerState() {
    if (this.registration.waiting) {
      // SW ожидает активации
      this.showUpdateAvailable();
    }
    
    if (this.registration.active) {
      console.log('Service Worker активен');
    }
  }
  
  // Отправка сообщения в Service Worker
  async sendMessage(message) {
    if (!this.registration || !this.registration.active) {
      throw new Error('Service Worker не активен');
    }
    
    return new Promise((resolve, reject) => {
      const messageChannel = new MessageChannel();
      
      messageChannel.port1.onmessage = (event) => {
        if (event.data.error) {
          reject(new Error(event.data.error));
        } else {
          resolve(event.data);
        }
      };
      
      this.registration.active.postMessage(message, [messageChannel.port2]);
    });
  }
  
  // Отписка от Service Worker
  async unregister() {
    if (this.registration) {
      const result = await this.registration.unregister();
      console.log('Service Worker отписан:', result);
      return result;
    }
    return false;
  }
}

// Глобальный менеджер
const swManager = new ServiceWorkerManager();

// Инициализация Service Worker
async function initServiceWorker() {
  try {
    await swManager.register('/service-worker.js');
    
    // Слушаем сообщения от Service Worker
    navigator.serviceWorker.addEventListener('message', (event) => {
      console.log('Сообщение от SW:', event.data);
    });
    
  } catch (error) {
    console.error('Ошибка инициализации SW:', error);
  }
}

// Service Worker script (service-worker.js)
const swCode = `
const CACHE_NAME = 'app-v1.0.0';
const STATIC_CACHE = 'static-v1';
const DYNAMIC_CACHE = 'dynamic-v1';

// Ресурсы для предварительного кэширования
const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/styles/main.css',
  '/scripts/app.js',
  '/manifest.json',
  '/images/icon-192.png',
  '/images/icon-512.png'
];

// Установка Service Worker
self.addEventListener('install', (event) => {
  console.log('SW: Installing...');
  
  event.waitUntil(
    caches.open(STATIC_CACHE)
      .then(cache => {
        console.log('SW: Caching static assets');
        return cache.addAll(STATIC_ASSETS);
      })
      .then(() => {
        // Принудительная активация нового SW
        return self.skipWaiting();
      })
  );
});

// Активация Service Worker
self.addEventListener('activate', (event) => {
  console.log('SW: Activating...');
  
  event.waitUntil(
    Promise.all([
      // Очистка старых кэшей
      cleanupOldCaches(),
      // Захват контроля над всеми клиентами
      self.clients.claim()
    ])
  );
});

// Очистка устаревших кэшей
async function cleanupOldCaches() {
  const cacheNames = await caches.keys();
  const oldCaches = cacheNames.filter(name => 
    name !== STATIC_CACHE && name !== DYNAMIC_CACHE
  );
  
  return Promise.all(
    oldCaches.map(name => caches.delete(name))
  );
}

// Обработка запросов
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Игнорируем запросы к другим доменам и chrome-extension
  if (url.origin !== location.origin || url.protocol === 'chrome-extension:') {
    return;
  }
  
  // Выбираем стратегию кэширования
  if (STATIC_ASSETS.includes(url.pathname)) {
    // Cache First для статических ресурсов
    event.respondWith(cacheFirst(request));
  } else if (url.pathname.startsWith('/api/')) {
    // Network First для API
    event.respondWith(networkFirst(request));
  } else if (request.destination === 'image') {
    // Stale While Revalidate для изображений
    event.respondWith(staleWhileRevalidate(request));
  } else {
    // Network First по умолчанию
    event.respondWith(networkFirst(request));
  }
});
`;

// Сохранение Service Worker в файл (для примера)
function createServiceWorkerFile() {
  const blob = new Blob([swCode], { type: 'application/javascript' });
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = 'service-worker.js';
  a.click();
  
  URL.revokeObjectURL(url);
}
```

---

## 📦 Стратегии кэширования

### Cache Strategies Implementation

```javascript
// Стратегии кэширования в Service Worker
const cacheStrategiesCode = `
// 1. Cache First - сначала кэш, потом сеть
async function cacheFirst(request) {
  try {
    const cachedResponse = await caches.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    const networkResponse = await fetch(request);
    
    if (networkResponse.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  } catch (error) {
    // Возвращаем fallback страницу для HTML запросов
    if (request.destination === 'document') {
      return caches.match('/offline.html');
    }
    throw error;
  }
}

// 2. Network First - сначала сеть, потом кэш
async function networkFirst(request) {
  try {
    const networkResponse = await fetch(request);
    
    if (networkResponse.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  } catch (error) {
    const cachedResponse = await caches.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    // Fallback для HTML страниц
    if (request.destination === 'document') {
      return caches.match('/offline.html');
    }
    
    throw error;
  }
}

// 3. Stale While Revalidate - возвращаем кэш, обновляем в фоне
async function staleWhileRevalidate(request) {
  const cachedResponse = caches.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    if (response.ok) {
      const cache = caches.open(DYNAMIC_CACHE);
      cache.then(c => c.put(request, response.clone()));
    }
    return response;
  }).catch(() => {
    // Игнорируем ошибки сети для SWR
  });
  
  return (await cachedResponse) || fetchPromise;
}

// 4. Network Only - только сеть
async function networkOnly(request) {
  return fetch(request);
}

// 5. Cache Only - только кэш
async function cacheOnly(request) {
  return caches.match(request);
}

// Управление кэшем
class CacheManager {
  constructor() {
    this.maxCacheSize = 50; // Максимум файлов в динамическом кэше
  }
  
  // Очистка кэша по размеру
  async trimCache(cacheName, maxItems) {
    const cache = await caches.open(cacheName);
    const keys = await cache.keys();
    
    if (keys.length > maxItems) {
      const keysToDelete = keys.slice(0, keys.length - maxItems);
      await Promise.all(
        keysToDelete.map(key => cache.delete(key))
      );
    }
  }
  
  // Очистка по времени
  async cleanExpiredCache(cacheName, maxAge) {
    const cache = await caches.open(cacheName);
    const keys = await cache.keys();
    const now = Date.now();
    
    for (const request of keys) {
      const response = await cache.match(request);
      const dateHeader = response.headers.get('date');
      
      if (dateHeader) {
        const responseTime = new Date(dateHeader).getTime();
        if (now - responseTime > maxAge) {
          await cache.delete(request);
        }
      }
    }
  }
  
  // Предварительная загрузка ресурсов
  async preloadResources(urls) {
    const cache = await caches.open(STATIC_CACHE);
    
    const preloadPromises = urls.map(async url => {
      try {
        const response = await fetch(url);
        if (response.ok) {
          await cache.put(url, response);
        }
      } catch (error) {
        console.warn('Ошибка предзагрузки:', url, error);
      }
    });
    
    await Promise.all(preloadPromises);
  }
}

const cacheManager = new CacheManager();

// Регулярная очистка кэша
setInterval(() => {
  cacheManager.trimCache(DYNAMIC_CACHE, 50);
  cacheManager.cleanExpiredCache(DYNAMIC_CACHE, 7 * 24 * 60 * 60 * 1000); // 7 дней
}, 60 * 60 * 1000); // каждый час
`;
```

---

## 📱 PWA и Web App Manifest

### Progressive Web App Configuration

```javascript
// Менеджер PWA
class PWAManager {
  constructor() {
    this.deferredPrompt = null;
    this.isInstalled = false;
    this.initPWA();
  }
  
  initPWA() {
    // Слушаем событие beforeinstallprompt
    window.addEventListener('beforeinstallprompt', (event) => {
      event.preventDefault();
      this.deferredPrompt = event;
      this.showInstallButton();
    });
    
    // Проверяем, установлено ли приложение
    window.addEventListener('appinstalled', () => {
      this.isInstalled = true;
      this.hideInstallButton();
      console.log('PWA установлено!');
    });
    
    // Проверяем запуск из установленного PWA
    if (window.matchMedia('(display-mode: standalone)').matches) {
      this.isInstalled = true;
      console.log('Запущено как PWA');
    }
  }
  
  // Показ кнопки установки
  showInstallButton() {
    const installButton = document.getElementById('install-button');
    if (installButton) {
      installButton.style.display = 'block';
      installButton.addEventListener('click', () => this.promptInstall());
    }
  }
  
  hideInstallButton() {
    const installButton = document.getElementById('install-button');
    if (installButton) {
      installButton.style.display = 'none';
    }
  }
  
  // Запрос установки
  async promptInstall() {
    if (!this.deferredPrompt) {
      return false;
    }
    
    this.deferredPrompt.prompt();
    const result = await this.deferredPrompt.userChoice;
    
    console.log('Результат установки:', result.outcome);
    
    this.deferredPrompt = null;
    this.hideInstallButton();
    
    return result.outcome === 'accepted';
  }
  
  // Проверка критериев PWA
  checkPWARequirements() {
    const requirements = {
      serviceWorker: 'serviceWorker' in navigator,
      manifest: !!document.querySelector('link[rel="manifest"]'),
      https: location.protocol === 'https:' || location.hostname === 'localhost'
    };
    
    const allMet = Object.values(requirements).every(Boolean);
    
    console.log('PWA Requirements:', requirements);
    return { requirements, allMet };
  }
}

// Web App Manifest
const manifestConfig = {
  name: "My Progressive Web App",
  short_name: "MyPWA",
  description: "Описание моего прогрессивного веб-приложения",
  start_url: "/",
  display: "standalone",
  orientation: "portrait",
  theme_color: "#000000",
  background_color: "#ffffff",
  categories: ["productivity", "utilities"],
  screenshots: [
    {
      src: "/screenshots/desktop.png",
      sizes: "1280x720",
      type: "image/png",
      form_factor: "wide"
    },
    {
      src: "/screenshots/mobile.png", 
      sizes: "540x720",
      type: "image/png",
      form_factor: "narrow"
    }
  ],
  icons: [
    {
      src: "/images/icon-72.png",
      sizes: "72x72",
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-96.png",
      sizes: "96x96", 
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-128.png",
      sizes: "128x128",
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-144.png",
      sizes: "144x144",
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-152.png",
      sizes: "152x152",
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-192.png",
      sizes: "192x192",
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-384.png",
      sizes: "384x384",
      type: "image/png",
      purpose: "maskable any"
    },
    {
      src: "/images/icon-512.png",
      sizes: "512x512",
      type: "image/png",
      purpose: "maskable any"
    }
  ]
};

// Создание файла манифеста
function createManifestFile() {
  const manifestBlob = new Blob([JSON.stringify(manifestConfig, null, 2)], {
    type: 'application/json'
  });
  
  const manifestUrl = URL.createObjectURL(manifestBlob);
  
  const link = document.createElement('link');
  link.rel = 'manifest';
  link.href = manifestUrl;
  document.head.appendChild(link);
  
  return manifestUrl;
}

// Инициализация PWA
const pwaManager = new PWAManager();
```

---

## 🔔 Push-уведомления

### Notifications и Background Sync

```javascript
// Менеджер push-уведомлений
class PushNotificationManager {
  constructor() {
    this.registration = null;
    this.subscription = null;
    this.vapidPublicKey = 'YOUR_VAPID_PUBLIC_KEY'; // Замените на ваш ключ
  }
  
  // Инициализация с Service Worker регистрацией
  init(registration) {
    this.registration = registration;
  }
  
  // Проверка поддержки уведомлений
  isSupported() {
    return 'Notification' in window && 'serviceWorker' in navigator;
  }
  
  // Запрос разрешения на уведомления
  async requestPermission() {
    if (!this.isSupported()) {
      throw new Error('Push уведомления не поддерживаются');
    }
    
    const permission = await Notification.requestPermission();
    console.log('Разрешение на уведомления:', permission);
    
    return permission === 'granted';
  }
  
  // Подписка на push-уведомления
  async subscribe() {
    if (!this.registration) {
      throw new Error('Service Worker не зарегистрирован');
    }
    
    const hasPermission = await this.requestPermission();
    if (!hasPermission) {
      throw new Error('Нет разрешения на уведомления');
    }
    
    try {
      this.subscription = await this.registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(this.vapidPublicKey)
      });
      
      console.log('Подписка создана:', this.subscription);
      
      // Отправляем подписку на сервер
      await this.sendSubscriptionToServer(this.subscription);
      
      return this.subscription;
    } catch (error) {
      console.error('Ошибка подписки:', error);
      throw error;
    }
  }
  
  // Отписка от push-уведомлений
  async unsubscribe() {
    if (this.subscription) {
      await this.subscription.unsubscribe();
      await this.removeSubscriptionFromServer(this.subscription);
      this.subscription = null;
      console.log('Отписка выполнена');
    }
  }
  
  // Получение текущей подписки
  async getSubscription() {
    if (!this.registration) {
      return null;
    }
    
    this.subscription = await this.registration.pushManager.getSubscription();
    return this.subscription;
  }
  
  // Отправка подписки на сервер
  async sendSubscriptionToServer(subscription) {
    try {
      await fetch('/api/push/subscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          subscription: subscription,
          userAgent: navigator.userAgent
        })
      });
    } catch (error) {
      console.error('Ошибка отправки подписки на сервер:', error);
    }
  }
  
  // Удаление подписки с сервера
  async removeSubscriptionFromServer(subscription) {
    try {
      await fetch('/api/push/unsubscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ subscription })
      });
    } catch (error) {
      console.error('Ошибка удаления подписки с сервера:', error);
    }
  }
  
  // Показ локального уведомления
  async showNotification(title, options = {}) {
    const defaultOptions = {
      body: '',
      icon: '/images/icon-192.png',
      badge: '/images/badge-72.png',
      vibrate: [200, 100, 200],
      data: {},
      actions: [],
      requireInteraction: false,
      silent: false
    };
    
    const finalOptions = { ...defaultOptions, ...options };
    
    if (this.registration) {
      return this.registration.showNotification(title, finalOptions);
    } else {
      return new Notification(title, finalOptions);
    }
  }
  
  // Конвертация VAPID ключа
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
}

// Background Sync в Service Worker
const backgroundSyncCode = `
// Обработка push-уведомлений в SW
self.addEventListener('push', (event) => {
  console.log('Push сообщение получено');
  
  let notificationData = {
    title: 'Новое уведомление',
    body: 'У вас есть новое сообщение',
    icon: '/images/icon-192.png',
    badge: '/images/badge-72.png',
    tag: 'default',
    data: {}
  };
  
  if (event.data) {
    try {
      notificationData = { ...notificationData, ...event.data.json() };
    } catch (error) {
      console.error('Ошибка парсинга push данных:', error);
    }
  }
  
  event.waitUntil(
    self.registration.showNotification(notificationData.title, notificationData)
  );
});

// Обработка кликов по уведомлениям
self.addEventListener('notificationclick', (event) => {
  console.log('Клик по уведомлению:', event.notification);
  
  event.notification.close();
  
  const clickAction = event.action || 'default';
  const notificationData = event.notification.data || {};
  
  event.waitUntil(
    handleNotificationClick(clickAction, notificationData)
  );
});

async function handleNotificationClick(action, data) {
  const clients = await self.clients.matchAll({ type: 'window' });
  
  switch (action) {
    case 'open':
    case 'default':
      const urlToOpen = data.url || '/';
      
      // Ищем существующую вкладку
      const existingClient = clients.find(client => 
        client.url.includes(urlToOpen)
      );
      
      if (existingClient) {
        await existingClient.focus();
      } else {
        await self.clients.openWindow(urlToOpen);
      }
      break;
      
    case 'close':
      // Просто закрываем уведомление
      break;
      
    case 'reply':
      // Обработка быстрого ответа
      await handleQuickReply(data);
      break;
  }
}

// Background Sync для отложенных действий
self.addEventListener('sync', (event) => {
  console.log('Background sync:', event.tag);
  
  if (event.tag === 'background-sync') {
    event.waitUntil(syncData());
  }
});

async function syncData() {
  try {
    // Получаем данные из IndexedDB для синхронизации
    const pendingData = await getPendingDataFromDB();
    
    for (const item of pendingData) {
      try {
        await syncItem(item);
        await markAsSynced(item.id);
      } catch (error) {
        console.error('Ошибка синхронизации элемента:', error);
      }
    }
  } catch (error) {
    console.error('Ошибка background sync:', error);
  }
}
`;

// Глобальный менеджер уведомлений
const pushManager = new PushNotificationManager();

// Инициализация push-уведомлений
async function initPushNotifications() {
  try {
    const registration = await swManager.register('/service-worker.js');
    pushManager.init(registration);
    
    // Проверяем существующую подписку
    const existingSubscription = await pushManager.getSubscription();
    
    if (!existingSubscription) {
      console.log('Нет активной подписки на push-уведомления');
    } else {
      console.log('Активная подписка найдена');
    }
    
  } catch (error) {
    console.error('Ошибка инициализации push-уведомлений:', error);
  }
}

// Пример использования уведомлений
async function subscribeToNotifications() {
  try {
    await pushManager.subscribe();
    
    // Показываем приветственное уведомление
    await pushManager.showNotification('Уведомления включены!', {
      body: 'Теперь вы будете получать важные обновления',
      actions: [
        { action: 'settings', title: 'Настройки' },
        { action: 'close', title: 'Закрыть' }
      ]
    });
    
  } catch (error) {
    console.error('Ошибка подписки на уведомления:', error);
  }
}
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **SW Registration** - Регистрация и базовое кэширование
2. **Offline Page** - Страница для офлайн-режима
3. **PWA Manifest** - Настройка манифеста приложения

### 🟡 Средний уровень
4. **Cache Strategies** - Различные стратегии кэширования
5. **Push Notifications** - Система push-уведомлений
6. **Background Sync** - Фоновая синхронизация данных

### 🔴 Продвинутый уровень
7. **Advanced PWA** - Полнофункциональное PWA
8. **Workbox Integration** - Интеграция с Workbox
9. **Performance Monitoring** - Мониторинг производительности

## 🔗 Связанные разделы

- **[[web-workers|⚡ Web Workers]]** - Фоновые вычисления
- **[[storage-apis|💾 Storage APIs]]** - Офлайн-хранение данных
- **[[fetch-api|🌐 Fetch API]]** - Сетевые запросы в SW
- **[[../../fundamentals/development-principles|⚡ Performance]]** - Оптимизация загрузки

---

⏱️ **Время изучения**: 5-6 часов  
🎯 **Уровень**: Средний - Продвинутый  
📋 **Предварительные требования**: JavaScript ES6+, Web APIs, HTTP, базы данных 