# 💾 Storage APIs - Локальное хранение данных

> **Storage APIs** - браузерные интерфейсы для сохранения данных на стороне клиента: localStorage, sessionStorage, IndexedDB и другие технологии хранения.

## 📋 Содержание

- [Web Storage API](#-web-storage-api)
- [IndexedDB](#-indexeddb)
- [Управление данными](#-управление-данными)
- [Практические паттерны](#-практические-паттерны)

---

## 🗄️ Web Storage API

### localStorage и sessionStorage

```javascript
// Класс для работы с localStorage
class StorageManager {
  constructor(storageType = 'localStorage') {
    this.storage = window[storageType];
  }
  
  // Сохранение с автоматической сериализацией и TTL
  set(key, value, expirationHours = null) {
    try {
      const item = {
        value: value,
        timestamp: Date.now(),
        expiration: expirationHours ? Date.now() + (expirationHours * 60 * 60 * 1000) : null
      };
      
      this.storage.setItem(key, JSON.stringify(item));
      return true;
    } catch (error) {
      console.error('Ошибка сохранения в storage:', error);
      return false;
    }
  }
  
  // Получение с проверкой TTL
  get(key, defaultValue = null) {
    try {
      const item = this.storage.getItem(key);
      if (!item) return defaultValue;
      
      const parsed = JSON.parse(item);
      
      // Проверяем не истек ли срок
      if (parsed.expiration && Date.now() > parsed.expiration) {
        this.remove(key);
        return defaultValue;
      }
      
      return parsed.value;
    } catch (error) {
      console.error('Ошибка чтения из storage:', error);
      return defaultValue;
    }
  }
  
  // Удаление
  remove(key) {
    this.storage.removeItem(key);
  }
  
  // Очистка всех данных
  clear() {
    this.storage.clear();
  }
  
  // Получение всех ключей
  keys() {
    return Object.keys(this.storage);
  }
  
  // Проверка существования
  has(key) {
    return this.storage.getItem(key) !== null;
  }
  
  // Получение размера хранилища
  size() {
    return this.storage.length;
  }
}

// Инициализация
const localStorage = new StorageManager('localStorage');
const sessionStorage = new StorageManager('sessionStorage');

// Корзина покупок с localStorage
class ShoppingCart {
  constructor() {
    this.cartKey = 'shopping_cart';
    this.storage = localStorage;
  }
  
  addItem(product, quantity = 1) {
    const cart = this.getCart();
    const existingItem = cart.find(item => item.id === product.id);
    
    if (existingItem) {
      existingItem.quantity += quantity;
    } else {
      cart.push({ ...product, quantity });
    }
    
    this.saveCart(cart);
    this.triggerCartUpdate();
  }
  
  getCart() {
    return this.storage.get(this.cartKey, []);
  }
  
  saveCart(cart) {
    this.storage.set(this.cartKey, cart);
  }
  
  getTotalPrice() {
    return this.getCart().reduce((total, item) => 
      total + (item.price * item.quantity), 0
    );
  }
  
  triggerCartUpdate() {
    window.dispatchEvent(new CustomEvent('cartUpdated', {
      detail: { cart: this.getCart(), totalPrice: this.getTotalPrice() }
    }));
  }
}
```

---

## 🗃️ IndexedDB

### Современная работа с IndexedDB

```javascript
// Класс для работы с IndexedDB
class IndexedDBManager {
  constructor(dbName, version = 1) {
    this.dbName = dbName;
    this.version = version;
    this.db = null;
  }
  
  // Инициализация базы данных
  async init(stores = []) {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        stores.forEach(store => {
          if (!db.objectStoreNames.contains(store.name)) {
            const objectStore = db.createObjectStore(store.name, store.options);
            
            // Создаем индексы
            if (store.indexes) {
              store.indexes.forEach(index => {
                objectStore.createIndex(index.name, index.keyPath, index.options);
              });
            }
          }
        });
      };
    });
  }
  
  // Добавление записи
  async add(storeName, data) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.add(data);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Получение записи по ключу
  async get(storeName, key) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.get(key);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Получение всех записей
  async getAll(storeName) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.getAll();
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Поиск с использованием курсора
  async query(storeName, indexName, query) {
    const transaction = this.db.transaction([storeName], 'readonly');
    const store = transaction.objectStore(storeName);
    const index = store.index(indexName);
    
    const results = [];
    
    return new Promise((resolve, reject) => {
      const request = index.openCursor(IDBKeyRange.only(query));
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        if (cursor) {
          results.push(cursor.value);
          cursor.continue();
        } else {
          resolve(results);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Обновление записи
  async update(storeName, data) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.put(data);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Удаление записи
  async delete(storeName, key) {
    const transaction = this.db.transaction([storeName], 'readwrite');
    const store = transaction.objectStore(storeName);
    
    return new Promise((resolve, reject) => {
      const request = store.delete(key);
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}

// Инициализация IndexedDB для приложения
const appDB = new IndexedDBManager('MyApp', 1);

// Настройка структуры базы данных
const stores = [
  {
    name: 'users',
    options: { keyPath: 'id', autoIncrement: true },
    indexes: [
      { name: 'email', keyPath: 'email', options: { unique: true } },
      { name: 'name', keyPath: 'name', options: { unique: false } }
    ]
  },
  {
    name: 'posts',
    options: { keyPath: 'id', autoIncrement: true },
    indexes: [
      { name: 'userId', keyPath: 'userId', options: { unique: false } },
      { name: 'createdAt', keyPath: 'createdAt', options: { unique: false } }
    ]
  }
];

// Инициализация базы данных
appDB.init(stores).then(() => {
  console.log('IndexedDB инициализирована');
}).catch(error => {
  console.error('Ошибка инициализации IndexedDB:', error);
});

// Примеры использования
async function saveUser(user) {
  try {
    const id = await appDB.add('users', user);
    console.log('Пользователь сохранен с ID:', id);
    return id;
  } catch (error) {
    console.error('Ошибка сохранения пользователя:', error);
  }
}

async function findUsersByName(name) {
  try {
    const users = await appDB.query('users', 'name', name);
    return users;
  } catch (error) {
    console.error('Ошибка поиска пользователей:', error);
    return [];
  }
}
```

---

## 🔄 Управление данными

### Синхронизация и кэширование

```javascript
// Менеджер кэша для API данных
class DataManager {
  constructor() {
    this.cache = new Map();
    this.storage = localStorage;
    this.syncQueue = [];
  }
  
  // Загрузка данных с кэшированием
  async loadData(endpoint, useCache = true, cacheTTL = 3600) {
    const cacheKey = `api_${endpoint}`;
    
    if (useCache) {
      // Проверяем localStorage кэш
      const cached = this.storage.get(cacheKey);
      if (cached) {
        console.log('Данные из localStorage кэша');
        return cached;
      }
      
      // Проверяем memory кэш
      if (this.cache.has(cacheKey)) {
        const memCached = this.cache.get(cacheKey);
        if (Date.now() < memCached.expires) {
          console.log('Данные из памяти');
          return memCached.data;
        }
      }
    }
    
    try {
      const response = await fetch(endpoint);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      
      const data = await response.json();
      
      // Сохраняем в оба кэша
      if (useCache) {
        this.storage.set(cacheKey, data, cacheTTL / 3600); // TTL в часах
        this.cache.set(cacheKey, {
          data: data,
          expires: Date.now() + (cacheTTL * 1000)
        });
      }
      
      return data;
    } catch (error) {
      console.error('Ошибка загрузки данных:', error);
      
      // Пытаемся вернуть устаревший кэш
      const staleCache = this.storage.get(cacheKey);
      if (staleCache) {
        console.warn('Возвращаем устаревший кэш');
        return staleCache;
      }
      
      throw error;
    }
  }
  
  // Сохранение данных для офлайн синхронизации
  saveForSync(action, data) {
    const syncItem = {
      id: Date.now() + Math.random(),
      action: action,
      data: data,
      timestamp: Date.now(),
      synced: false
    };
    
    this.syncQueue.push(syncItem);
    this.storage.set('sync_queue', this.syncQueue);
    
    // Пытаемся синхронизировать если онлайн
    if (navigator.onLine) {
      this.syncData();
    }
  }
  
  // Синхронизация данных с сервером
  async syncData() {
    const queue = this.storage.get('sync_queue', []);
    const unsyncedItems = queue.filter(item => !item.synced);
    
    for (const item of unsyncedItems) {
      try {
        await this.syncItem(item);
        item.synced = true;
      } catch (error) {
        console.error('Ошибка синхронизации:', error);
      }
    }
    
    // Обновляем очередь
    this.storage.set('sync_queue', queue);
    
    // Удаляем синхронизированные элементы старше суток
    const oneDayAgo = Date.now() - (24 * 60 * 60 * 1000);
    const cleanQueue = queue.filter(item => 
      !item.synced || item.timestamp > oneDayAgo
    );
    
    this.storage.set('sync_queue', cleanQueue);
  }
  
  async syncItem(item) {
    const { action, data } = item;
    
    switch (action) {
      case 'CREATE_USER':
        await fetch('/api/users', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(data)
        });
        break;
        
      case 'UPDATE_USER':
        await fetch(`/api/users/${data.id}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(data)
        });
        break;
        
      default:
        console.warn('Неизвестное действие синхронизации:', action);
    }
  }
}

// Инициализация менеджера данных
const dataManager = new DataManager();

// Автоматическая синхронизация при восстановлении соединения
window.addEventListener('online', () => {
  console.log('Соединение восстановлено, синхронизируем данные');
  dataManager.syncData();
});
```

---

## 🎯 Практические паттерны

### Управление настройками пользователя

```javascript
// Менеджер пользовательских настроек
class SettingsManager {
  constructor() {
    this.storage = localStorage;
    this.defaultSettings = {
      theme: 'light',
      language: 'ru',
      notifications: true,
      autoSave: true,
      cacheEnabled: true
    };
  }
  
  // Получение настройки
  get(key) {
    const settings = this.storage.get('user_settings', this.defaultSettings);
    return settings[key] !== undefined ? settings[key] : this.defaultSettings[key];
  }
  
  // Установка настройки
  set(key, value) {
    const settings = this.storage.get('user_settings', this.defaultSettings);
    settings[key] = value;
    this.storage.set('user_settings', settings);
    
    // Уведомляем об изменении
    window.dispatchEvent(new CustomEvent('settingsChanged', {
      detail: { key, value, settings }
    }));
  }
  
  // Сброс к defaults
  reset() {
    this.storage.set('user_settings', this.defaultSettings);
    window.dispatchEvent(new CustomEvent('settingsReset'));
  }
  
  // Экспорт настроек
  export() {
    return this.storage.get('user_settings', this.defaultSettings);
  }
  
  // Импорт настроек
  import(settings) {
    const merged = { ...this.defaultSettings, ...settings };
    this.storage.set('user_settings', merged);
    window.dispatchEvent(new CustomEvent('settingsImported'));
  }
}

// Использование менеджера настроек
const settings = new SettingsManager();

// Применение темы при загрузке
document.addEventListener('DOMContentLoaded', () => {
  const theme = settings.get('theme');
  document.body.className = theme === 'dark' ? 'dark-theme' : 'light-theme';
});

// Слушаем изменения настроек
window.addEventListener('settingsChanged', (event) => {
  const { key, value } = event.detail;
  
  if (key === 'theme') {
    document.body.className = value === 'dark' ? 'dark-theme' : 'light-theme';
  }
});
```

### Офлайн-первый подход

```javascript
// Стратегия "офлайн-первый" для данных
class OfflineFirstManager {
  constructor() {
    this.storage = localStorage;
    this.isOnline = navigator.onLine;
    
    // Слушаем изменения состояния сети
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.syncWhenOnline();
    });
    
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }
  
  // Получение данных (сначала локально, потом с сервера)
  async getData(key, endpoint) {
    // Всегда сначала возвращаем локальные данные
    const localData = this.storage.get(key);
    
    if (!this.isOnline) {
      console.log('Офлайн режим: возвращаем локальные данные');
      return localData || [];
    }
    
    try {
      // Загружаем свежие данные с сервера
      const response = await fetch(endpoint);
      if (response.ok) {
        const serverData = await response.json();
        
        // Обновляем локальные данные
        this.storage.set(key, serverData);
        
        return serverData;
      }
    } catch (error) {
      console.warn('Ошибка загрузки с сервера, используем локальные данные');
    }
    
    return localData || [];
  }
  
  // Сохранение данных (локально + в очередь синхронизации)
  async saveData(key, data, endpoint) {
    // Сначала сохраняем локально
    this.storage.set(key, data);
    
    if (this.isOnline) {
      try {
        // Пытаемся отправить на сервер
        await fetch(endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(data)
        });
        
        console.log('Данные синхронизированы с сервером');
      } catch (error) {
        // Добавляем в очередь синхронизации
        this.addToSyncQueue(key, data, endpoint);
      }
    } else {
      // Добавляем в очередь синхронизации
      this.addToSyncQueue(key, data, endpoint);
    }
  }
  
  addToSyncQueue(key, data, endpoint) {
    const queue = this.storage.get('sync_queue', []);
    queue.push({ key, data, endpoint, timestamp: Date.now() });
    this.storage.set('sync_queue', queue);
  }
  
  async syncWhenOnline() {
    const queue = this.storage.get('sync_queue', []);
    
    for (const item of queue) {
      try {
        await fetch(item.endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(item.data)
        });
        
        console.log('Синхронизирован элемент:', item.key);
      } catch (error) {
        console.error('Ошибка синхронизации:', error);
        break; // Останавливаемся на первой ошибке
      }
    }
    
    // Очищаем успешно синхронизированную очередь
    this.storage.remove('sync_queue');
  }
}

// Использование офлайн-менеджера
const offlineManager = new OfflineFirstManager();

// Пример работы с задачами
async function loadTasks() {
  const tasks = await offlineManager.getData('tasks', '/api/tasks');
  displayTasks(tasks);
}

async function saveTask(task) {
  await offlineManager.saveData('tasks', task, '/api/tasks');
  showNotification('Задача сохранена');
}
```

## 🎯 Практические задания

### 🟢 Базовый уровень  
1. **StorageManager** - Создать класс для работы с localStorage с TTL
2. **Shopping Cart** - Реализовать корзину покупок с сохранением
3. **User Settings** - Система настроек пользователя

### 🟡 Средний уровень
4. **IndexedDB wrapper** - Создать удобную обертку для IndexedDB
5. **Data Sync** - Синхронизация данных между вкладками
6. **Offline Cache** - Кэширование данных для офлайн режима

### 🔴 Продвинутый уровень
7. **Conflict Resolution** - Разрешение конфликтов при синхронизации
8. **Storage Optimization** - Автоматическая очистка устаревших данных
9. **Cross-tab Communication** - Обмен данными между вкладками

## 🔗 Связанные разделы

- **[[fetch-api|🌐 Fetch API]]** - Кэширование API запросов
- **[[service-workers-pwa|🚀 Service Workers]]** - Расширенные стратегии кэширования
- **[[web-workers|⚡ Web Workers]]** - Обработка больших объемов данных
- **[[../../fundamentals/clean-architecture|🏗️ Clean Architecture]]** - Архитектура слоя данных

---

⏱️ **Время изучения**: 3-4 часа  
🎯 **Уровень**: Базовый - Средний  
📋 **Предварительные требования**: JavaScript ES6+, JSON, базы данных 