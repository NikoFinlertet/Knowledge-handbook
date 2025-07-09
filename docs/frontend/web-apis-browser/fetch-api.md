# 🌐 Fetch API - HTTP-запросы и сетевое взаимодействие

> **Fetch API** - современный стандарт для выполнения HTTP-запросов в браузере, заменяющий устаревший XMLHttpRequest более элегантным Promise-based интерфейсом.

## 📋 Содержание раздела

- [Основы Fetch API](#-основы-fetch-api)
- [Обработка ошибок](#-обработка-ошибок)
- [Продвинутые паттерны](#-продвинутые-паттерны)
- [API-клиенты](#-api-клиенты)
- [Тестирование](#-тестирование)

---

## 🚀 Основы Fetch API

### Базовые HTTP-запросы

```javascript
// 🟢 GET запрос - загрузка данных
async function fetchUsers() {
  try {
    const response = await fetch('https://api.example.com/users');
    
    // Проверка успешности запроса
    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
    }
    
    const users = await response.json();
    console.log('Пользователи загружены:', users);
    return users;
  } catch (error) {
    console.error('Ошибка при загрузке пользователей:', error);
    throw error;
  }
}

// 🟡 POST запрос - создание ресурса
async function createUser(userData) {
  try {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${getAuthToken()}`
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || 'Ошибка создания пользователя');
    }
    
    return await response.json();
  } catch (error) {
    console.error('Ошибка при создании пользователя:', error);
    throw error;
  }
}

// 🟡 PUT запрос - обновление ресурса
async function updateUser(userId, userData) {
  const response = await fetch(`/api/users/${userId}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${getAuthToken()}`
    },
    body: JSON.stringify(userData)
  });
  
  if (!response.ok) {
    throw new Error(`Ошибка обновления: ${response.status}`);
  }
  
  return await response.json();
}

// 🔴 DELETE запрос - удаление ресурса
async function deleteUser(userId) {
  const response = await fetch(`/api/users/${userId}`, {
    method: 'DELETE',
    headers: {
      'Authorization': `Bearer ${getAuthToken()}`
    }
  });
  
  if (!response.ok) {
    throw new Error(`Ошибка удаления: ${response.status}`);
  }
  
  // DELETE может возвращать пустой ответ
  const contentType = response.headers.get('content-type');
  if (contentType && contentType.includes('application/json')) {
    return await response.json();
  }
  
  return { success: true };
}
```

### Работа с различными типами данных

```javascript
// 📁 Загрузка файлов
async function uploadFile(file, progressCallback) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('description', 'Загруженный файл');
  formData.append('category', 'documents');
  
  try {
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData // Не устанавливаем Content-Type для FormData
    });
    
    if (!response.ok) {
      throw new Error('Ошибка загрузки файла');
    }
    
    return await response.json();
  } catch (error) {
    console.error('Ошибка загрузки:', error);
    throw error;
  }
}

// 📊 Загрузка с отслеживанием прогресса
async function uploadFileWithProgress(file, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    const formData = new FormData();
    formData.append('file', file);
    
    // Отслеживание прогресса загрузки
    xhr.upload.addEventListener('progress', (event) => {
      if (event.lengthComputable) {
        const percentComplete = (event.loaded / event.total) * 100;
        onProgress(percentComplete);
      }
    });
    
    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });
    
    xhr.addEventListener('error', () => reject(new Error('Upload failed')));
    
    xhr.open('POST', '/api/upload');
    xhr.send(formData);
  });
}

// 🖼️ Загрузка изображений и blob данных
async function downloadImage(imageUrl) {
  try {
    const response = await fetch(imageUrl);
    
    if (!response.ok) {
      throw new Error(`Ошибка загрузки изображения: ${response.status}`);
    }
    
    const blob = await response.blob();
    
    // Создаем URL для blob
    const imageObjectURL = URL.createObjectURL(blob);
    
    return {
      blob,
      url: imageObjectURL,
      type: blob.type,
      size: blob.size
    };
  } catch (error) {
    console.error('Ошибка загрузки изображения:', error);
    throw error;
  }
}

// 📄 Работа с текстовыми данными
async function fetchTextData(url) {
  const response = await fetch(url);
  
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }
  
  // Различные способы получения текста
  const text = await response.text(); // Обычный текст
  // const json = await response.json(); // JSON
  // const formData = await response.formData(); // FormData
  // const arrayBuffer = await response.arrayBuffer(); // ArrayBuffer
  
  return text;
}
```

---

## ⚠️ Обработка ошибок

### Типы ошибок и их обработка

```javascript
// Кастомные типы ошибок для разных сценариев
class ApiError extends Error {
  constructor(message, status, endpoint, response) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.endpoint = endpoint;
    this.response = response;
  }
}

class NetworkError extends Error {
  constructor(message, endpoint) {
    super(message);
    this.name = 'NetworkError';
    this.endpoint = endpoint;
  }
}

class TimeoutError extends Error {
  constructor(message, timeout) {
    super(message);
    this.name = 'TimeoutError';
    this.timeout = timeout;
  }
}

// Универсальная функция для обработки fetch запросов
async function safeFetch(url, options = {}) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), options.timeout || 10000);
  
  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal
    });
    
    clearTimeout(timeoutId);
    
    // Проверяем статус ответа
    if (!response.ok) {
      let errorMessage = `HTTP ${response.status}: ${response.statusText}`;
      let errorData = null;
      
      // Пытаемся получить дополнительную информацию об ошибке
      const contentType = response.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        try {
          errorData = await response.json();
          errorMessage = errorData.message || errorData.error || errorMessage;
        } catch (e) {
          // Игнорируем ошибки парсинга
        }
      }
      
      throw new ApiError(errorMessage, response.status, url, errorData);
    }
    
    return response;
  } catch (error) {
    clearTimeout(timeoutId);
    
    if (error.name === 'AbortError') {
      throw new TimeoutError(`Запрос превысил время ожидания`, options.timeout || 10000);
    }
    
    if (error instanceof ApiError) {
      throw error;
    }
    
    // Сетевые ошибки (нет интернета, DNS и т.д.)
    throw new NetworkError(`Сетевая ошибка: ${error.message}`, url);
  }
}

// Retry механизм с экспоненциальным backoff
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  const { retries = 0, delay = 1000, ...fetchOptions } = options;
  
  try {
    return await safeFetch(url, fetchOptions);
  } catch (error) {
    if (retries < maxRetries && shouldRetry(error)) {
      console.warn(`Повторная попытка ${retries + 1}/${maxRetries} через ${delay}ms`);
      
      await new Promise(resolve => setTimeout(resolve, delay));
      
      return fetchWithRetry(url, {
        ...options,
        retries: retries + 1,
        delay: delay * 2 // Экспоненциальный backoff
      }, maxRetries);
    }
    
    throw error;
  }
}

// Определяем, стоит ли повторять запрос
function shouldRetry(error) {
  // Повторяем только для определенных типов ошибок
  if (error instanceof NetworkError) return true;
  if (error instanceof TimeoutError) return true;
  if (error instanceof ApiError && error.status >= 500) return true;
  
  return false;
}

// Пример использования с обработкой ошибок
async function loadUserData(userId) {
  try {
    const response = await fetchWithRetry(`/api/users/${userId}`, {
      timeout: 5000,
      headers: {
        'Authorization': `Bearer ${getAuthToken()}`
      }
    });
    
    return await response.json();
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.status) {
        case 401:
          // Неавторизован - перенаправляем на логин
          redirectToLogin();
          break;
        case 403:
          // Нет доступа
          showErrorMessage('У вас нет доступа к этому ресурсу');
          break;
        case 404:
          // Не найден
          showErrorMessage('Пользователь не найден');
          break;
        default:
          // Серверная ошибка
          showErrorMessage('Произошла ошибка на сервере');
      }
    } else if (error instanceof NetworkError) {
      showErrorMessage('Проверьте подключение к интернету');
    } else if (error instanceof TimeoutError) {
      showErrorMessage('Запрос занял слишком много времени');
    } else {
      showErrorMessage('Произошла неизвестная ошибка');
    }
    
    throw error;
  }
}
```

---

## 🎯 Продвинутые паттерны

### Паралельные запросы и управление конкурентностью

```javascript
// Параллельные запросы с Promise.all
async function loadDashboardData() {
  try {
    const [users, posts, comments, analytics] = await Promise.all([
      fetchWithRetry('/api/users'),
      fetchWithRetry('/api/posts'),
      fetchWithRetry('/api/comments'),
      fetchWithRetry('/api/analytics')
    ]);
    
    return {
      users: await users.json(),
      posts: await posts.json(),
      comments: await comments.json(),
      analytics: await analytics.json()
    };
  } catch (error) {
    console.error('Ошибка загрузки данных дашборда:', error);
    throw error;
  }
}

// Promise.allSettled для обработки частичных неудач
async function loadOptionalData() {
  const requests = [
    fetchWithRetry('/api/users'),
    fetchWithRetry('/api/posts'),
    fetchWithRetry('/api/settings'), // Может упасть, но это не критично
    fetchWithRetry('/api/notifications') // Тоже не критично
  ];
  
  const results = await Promise.allSettled(requests);
  
  const data = {};
  const errors = [];
  
  for (let i = 0; i < results.length; i++) {
    const result = results[i];
    const keys = ['users', 'posts', 'settings', 'notifications'];
    
    if (result.status === 'fulfilled') {
      data[keys[i]] = await result.value.json();
    } else {
      errors.push({ key: keys[i], error: result.reason });
    }
  }
  
  if (errors.length > 0) {
    console.warn('Некоторые данные не удалось загрузить:', errors);
  }
  
  return data;
}

// Ограничение конкурентности запросов
class ConcurrencyManager {
  constructor(maxConcurrent = 5) {
    this.maxConcurrent = maxConcurrent;
    this.running = 0;
    this.queue = [];
  }
  
  async fetch(url, options = {}) {
    return new Promise((resolve, reject) => {
      this.queue.push({ url, options, resolve, reject });
      this.processQueue();
    });
  }
  
  async processQueue() {
    if (this.running >= this.maxConcurrent || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { url, options, resolve, reject } = this.queue.shift();
    
    try {
      const response = await safeFetch(url, options);
      resolve(response);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.processQueue(); // Обрабатываем следующий в очереди
    }
  }
}

// Использование менеджера конкурентности
const concurrencyManager = new ConcurrencyManager(3);

async function loadManyResources(urls) {
  const promises = urls.map(url => 
    concurrencyManager.fetch(url).then(response => response.json())
  );
  
  return Promise.all(promises);
}
```

### Кэширование и оптимизация

```javascript
// Простой HTTP кэш с TTL
class HttpCache {
  constructor(defaultTTL = 300000) { // 5 минут по умолчанию
    this.cache = new Map();
    this.defaultTTL = defaultTTL;
  }
  
  // Создание ключа кэша из URL и опций
  createCacheKey(url, options = {}) {
    const { method = 'GET', body, headers } = options;
    return `${method}:${url}:${JSON.stringify({ body, headers })}`;
  }
  
  // Получение из кэша
  get(url, options = {}) {
    const key = this.createCacheKey(url, options);
    const cached = this.cache.get(key);
    
    if (!cached) return null;
    
    // Проверяем не истек ли TTL
    if (Date.now() > cached.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    return cached.data;
  }
  
  // Сохранение в кэш
  set(url, options, data, ttl) {
    const key = this.createCacheKey(url, options);
    const expiresAt = Date.now() + (ttl || this.defaultTTL);
    
    this.cache.set(key, {
      data: data,
      expiresAt: expiresAt,
      createdAt: Date.now()
    });
  }
  
  // Очистка кэша
  clear() {
    this.cache.clear();
  }
  
  // Очистка истекших записей
  cleanup() {
    const now = Date.now();
    for (const [key, value] of this.cache.entries()) {
      if (now > value.expiresAt) {
        this.cache.delete(key);
      }
    }
  }
}

// Fetch с кэшированием
const httpCache = new HttpCache();

async function cachedFetch(url, options = {}) {
  const { useCache = true, cacheTTL, ...fetchOptions } = options;
  
  // Проверяем кэш только для GET запросов
  if (useCache && (!fetchOptions.method || fetchOptions.method === 'GET')) {
    const cached = httpCache.get(url, fetchOptions);
    if (cached) {
      console.log('Данные из кэша:', url);
      return Promise.resolve(new Response(JSON.stringify(cached)));
    }
  }
  
  try {
    const response = await safeFetch(url, fetchOptions);
    
    // Кэшируем только успешные GET запросы
    if (useCache && response.ok && (!fetchOptions.method || fetchOptions.method === 'GET')) {
      const data = await response.json();
      httpCache.set(url, fetchOptions, data, cacheTTL);
      
      // Возвращаем новый Response с данными
      return new Response(JSON.stringify(data));
    }
    
    return response;
  } catch (error) {
    console.error('Ошибка в cachedFetch:', error);
    throw error;
  }
}

// Очистка кэша каждые 10 минут
setInterval(() => {
  httpCache.cleanup();
}, 600000);
```

---

## 🏗️ API-клиенты

### Базовый API клиент

```javascript
class ApiClient {
  constructor(config = {}) {
    this.baseUrl = config.baseUrl || '';
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...config.defaultHeaders
    };
    this.defaultTimeout = config.timeout || 10000;
    this.cache = new HttpCache(config.cacheTTL);
    this.interceptors = {
      request: [],
      response: []
    };
  }
  
  // Добавление интерцепторов
  addRequestInterceptor(interceptor) {
    this.interceptors.request.push(interceptor);
  }
  
  addResponseInterceptor(interceptor) {
    this.interceptors.response.push(interceptor);
  }
  
  // Применение интерцепторов запроса
  async applyRequestInterceptors(url, options) {
    let modifiedUrl = url;
    let modifiedOptions = { ...options };
    
    for (const interceptor of this.interceptors.request) {
      const result = await interceptor(modifiedUrl, modifiedOptions);
      if (result) {
        modifiedUrl = result.url || modifiedUrl;
        modifiedOptions = result.options || modifiedOptions;
      }
    }
    
    return { url: modifiedUrl, options: modifiedOptions };
  }
  
  // Применение интерцепторов ответа
  async applyResponseInterceptors(response) {
    let modifiedResponse = response;
    
    for (const interceptor of this.interceptors.response) {
      const result = await interceptor(modifiedResponse);
      if (result) {
        modifiedResponse = result;
      }
    }
    
    return modifiedResponse;
  }
  
  // Базовый метод для запросов
  async request(endpoint, options = {}) {
    const url = `${this.baseUrl}${endpoint}`;
    const config = {
      timeout: this.defaultTimeout,
      headers: { ...this.defaultHeaders, ...options.headers },
      ...options
    };
    
    // Применяем интерцепторы запроса
    const { url: finalUrl, options: finalOptions } = await this.applyRequestInterceptors(url, config);
    
    try {
      let response = await fetchWithRetry(finalUrl, finalOptions);
      
      // Применяем интерцепторы ответа
      response = await this.applyResponseInterceptors(response);
      
      return response;
    } catch (error) {
      console.error(`API Error [${endpoint}]:`, error);
      throw error;
    }
  }
  
  // HTTP методы
  async get(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }
  
  async post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
  
  async put(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
  
  async patch(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PATCH',
      body: JSON.stringify(data)
    });
  }
  
  async delete(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'DELETE' });
  }
}

// Специализированный клиент для пользователей
class UserApiClient extends ApiClient {
  constructor(config) {
    super({
      baseUrl: '/api/users',
      ...config
    });
    
    // Добавляем интерцептор для автоматического добавления токена
    this.addRequestInterceptor(async (url, options) => {
      const token = getAuthToken();
      if (token) {
        options.headers = {
          ...options.headers,
          'Authorization': `Bearer ${token}`
        };
      }
      return { url, options };
    });
    
    // Интерцептор для обработки 401 ошибок
    this.addResponseInterceptor(async (response) => {
      if (response.status === 401) {
        // Токен истек, обновляем его
        await refreshAuthToken();
        // Можно повторить запрос с новым токеном
      }
      return response;
    });
  }
  
  async getUsers(page = 1, limit = 20) {
    const response = await this.get(`?page=${page}&limit=${limit}`);
    return response.json();
  }
  
  async getUser(id) {
    const response = await this.get(`/${id}`);
    return response.json();
  }
  
  async createUser(userData) {
    const response = await this.post('', userData);
    return response.json();
  }
  
  async updateUser(id, userData) {
    const response = await this.put(`/${id}`, userData);
    return response.json();
  }
  
  async deleteUser(id) {
    const response = await this.delete(`/${id}`);
    return response.ok;
  }
  
  async searchUsers(query) {
    const response = await this.get(`/search?q=${encodeURIComponent(query)}`);
    return response.json();
  }
}

// Использование API клиента
const userApi = new UserApiClient();

// Примеры использования
async function loadUsersPage() {
  try {
    const users = await userApi.getUsers(1, 10);
    displayUsers(users);
  } catch (error) {
    showErrorMessage('Не удалось загрузить пользователей');
  }
}
```

---

## 🧪 Тестирование

### Mock-сервер для разработки

```javascript
// Простой mock-сервер для тестирования
class MockApiServer {
  constructor() {
    this.routes = new Map();
    this.delay = 500; // Имитация задержки сети
  }
  
  // Регистрация маршрута
  mock(method, path, handler) {
    const key = `${method.toUpperCase()}:${path}`;
    this.routes.set(key, handler);
  }
  
  // Обработка запроса
  async handleRequest(url, options = {}) {
    const method = options.method || 'GET';
    const path = new URL(url, 'http://localhost').pathname;
    const key = `${method}:${path}`;
    
    const handler = this.routes.get(key);
    if (!handler) {
      return new Response(JSON.stringify({ error: 'Not Found' }), {
        status: 404,
        headers: { 'Content-Type': 'application/json' }
      });
    }
    
    // Имитируем задержку сети
    await new Promise(resolve => setTimeout(resolve, this.delay));
    
    try {
      const result = await handler(url, options);
      
      if (result instanceof Response) {
        return result;
      }
      
      return new Response(JSON.stringify(result), {
        status: 200,
        headers: { 'Content-Type': 'application/json' }
      });
    } catch (error) {
      return new Response(JSON.stringify({ error: error.message }), {
        status: 500,
        headers: { 'Content-Type': 'application/json' }
      });
    }
  }
}

// Настройка mock-сервера
const mockServer = new MockApiServer();

// Mock данные
const mockUsers = [
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
];

// Регистрация mock маршрутов
mockServer.mock('GET', '/api/users', () => mockUsers);

mockServer.mock('GET', '/api/users/1', () => mockUsers[0]);

mockServer.mock('POST', '/api/users', (url, options) => {
  const userData = JSON.parse(options.body);
  const newUser = { id: Date.now(), ...userData };
  mockUsers.push(newUser);
  return newUser;
});

// Переопределение fetch для использования mock-сервера
const originalFetch = window.fetch;

window.fetch = async function(url, options = {}) {
  // Используем mock только для /api маршрутов
  if (typeof url === 'string' && url.includes('/api/')) {
    return mockServer.handleRequest(url, options);
  }
  
  // Для остальных запросов используем оригинальный fetch
  return originalFetch(url, options);
};

// Функция для отключения mock-сервера
function disableMockServer() {
  window.fetch = originalFetch;
}
```

### Тестирование с Jest

```javascript
// __tests__/api-client.test.js
import { UserApiClient } from '../src/api/user-api-client';

// Mock fetch
global.fetch = jest.fn();

describe('UserApiClient', () => {
  let userApi;
  
  beforeEach(() => {
    userApi = new UserApiClient({ baseUrl: '/api/users' });
    fetch.mockClear();
  });
  
  describe('getUsers', () => {
    it('должен загружать пользователей', async () => {
      const mockUsers = [
        { id: 1, name: 'John' },
        { id: 2, name: 'Jane' }
      ];
      
      fetch.mockResolvedValueOnce({
        ok: true,
        json: async () => mockUsers
      });
      
      const users = await userApi.getUsers();
      
      expect(fetch).toHaveBeenCalledWith('/api/users?page=1&limit=20', {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' },
        timeout: 10000
      });
      
      expect(users).toEqual(mockUsers);
    });
    
    it('должен обрабатывать ошибки API', async () => {
      fetch.mockResolvedValueOnce({
        ok: false,
        status: 500,
        statusText: 'Internal Server Error'
      });
      
      await expect(userApi.getUsers()).rejects.toThrow('HTTP 500');
    });
  });
  
  describe('createUser', () => {
    it('должен создавать пользователя', async () => {
      const userData = { name: 'New User', email: 'new@example.com' };
      const createdUser = { id: 3, ...userData };
      
      fetch.mockResolvedValueOnce({
        ok: true,
        json: async () => createdUser
      });
      
      const result = await userApi.createUser(userData);
      
      expect(fetch).toHaveBeenCalledWith('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
        timeout: 10000
      });
      
      expect(result).toEqual(createdUser);
    });
  });
});
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **Простой HTTP клиент** - Создать класс для выполнения GET, POST, PUT, DELETE запросов
2. **Обработка ошибок** - Добавить централизованную обработку HTTP ошибок
3. **Loading состояния** - Реализовать индикаторы загрузки для запросов

### 🟡 Средний уровень
4. **Retry механизм** - Добавить автоматическое повторение неудачных запросов
5. **Кэширование** - Реализовать HTTP кэш с TTL
6. **Интерцепторы** - Создать систему интерцепторов для запросов и ответов

### 🔴 Продвинутый уровень
7. **Конкурентность** - Ограничить количество одновременных запросов
8. **Mock система** - Создать полноценную систему для мокирования API
9. **Полный API клиент** - Разработать production-ready API клиент с метриками

## 🔗 Связанные разделы

- **[[storage-apis|💾 Storage APIs]]** - Кэширование API данных локально
- **[[service-workers-pwa#offline-sync|🚀 Service Workers]]** - Офлайн синхронизация запросов
- **[[../testing-frontend|🧪 Frontend Testing]]** - Тестирование HTTP запросов
- **[[../../infrastructure/linux-deployment/monitoring-logging|📊 Monitoring]]** - Мониторинг API производительности

---

⏱️ **Время изучения**: 4-6 часов  
🎯 **Уровень**: Базовый - Средний  
📋 **Предварительные требования**: JavaScript ES6+, Promises, async/await 