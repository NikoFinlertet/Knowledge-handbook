# ⚡ Web Workers - Фоновые вычисления

> **Web Workers** - API для выполнения JavaScript-кода в фоновых потоках, обеспечивающего отзывчивость UI при тяжелых вычислениях.

## 📋 Содержание

- [Основы Web Workers](#-основы-web-workers)
- [Dedicated Workers](#-dedicated-workers)
- [Shared Workers](#-shared-workers)
- [Практические применения](#-практические-применения)

---

## 🔧 Основы Web Workers

### Создание и коммуникация

```javascript
// Класс для управления Web Workers
class WorkerManager {
  constructor() {
    this.workers = new Map();
    this.messageId = 0;
  }
  
  // Создание Dedicated Worker
  createWorker(workerScript, workerName = 'default') {
    try {
      const worker = new Worker(workerScript);
      const workerId = workerName + '_' + Date.now();
      
      const workerWrapper = {
        worker: worker,
        name: workerName,
        script: workerScript,
        pendingMessages: new Map(),
        isTerminated: false
      };
      
      // Обработка сообщений от worker'а
      worker.onmessage = (event) => {
        this.handleWorkerMessage(workerId, event);
      };
      
      worker.onerror = (error) => {
        console.error(`Ошибка в worker ${workerName}:`, error);
      };
      
      this.workers.set(workerId, workerWrapper);
      return workerId;
      
    } catch (error) {
      console.error('Ошибка создания worker:', error);
      return null;
    }
  }
  
  // Отправка сообщения в worker
  async sendMessage(workerId, data, transferableObjects = []) {
    const workerWrapper = this.workers.get(workerId);
    if (!workerWrapper || workerWrapper.isTerminated) {
      throw new Error('Worker не найден или завершен');
    }
    
    const messageId = ++this.messageId;
    const message = {
      id: messageId,
      data: data,
      timestamp: Date.now()
    };
    
    return new Promise((resolve, reject) => {
      // Сохраняем callback для ответа
      workerWrapper.pendingMessages.set(messageId, { resolve, reject });
      
      // Отправляем сообщение
      try {
        workerWrapper.worker.postMessage(message, transferableObjects);
      } catch (error) {
        workerWrapper.pendingMessages.delete(messageId);
        reject(error);
      }
      
      // Таймаут для сообщения
      setTimeout(() => {
        if (workerWrapper.pendingMessages.has(messageId)) {
          workerWrapper.pendingMessages.delete(messageId);
          reject(new Error('Таймаут ожидания ответа от worker'));
        }
      }, 30000);
    });
  }
  
  // Обработка сообщений от worker'а
  handleWorkerMessage(workerId, event) {
    const workerWrapper = this.workers.get(workerId);
    if (!workerWrapper) return;
    
    const { id, data, error } = event.data;
    
    if (id && workerWrapper.pendingMessages.has(id)) {
      const { resolve, reject } = workerWrapper.pendingMessages.get(id);
      workerWrapper.pendingMessages.delete(id);
      
      if (error) {
        reject(new Error(error));
      } else {
        resolve(data);
      }
    }
  }
  
  // Завершение worker'а
  terminateWorker(workerId) {
    const workerWrapper = this.workers.get(workerId);
    if (workerWrapper) {
      workerWrapper.worker.terminate();
      workerWrapper.isTerminated = true;
      
      // Отклоняем все ожидающие сообщения
      workerWrapper.pendingMessages.forEach(({ reject }) => {
        reject(new Error('Worker был завершен'));
      });
      
      this.workers.delete(workerId);
    }
  }
  
  // Завершение всех worker'ов
  terminateAll() {
    this.workers.forEach((_, workerId) => {
      this.terminateWorker(workerId);
    });
  }
}

// Создание inline worker без отдельного файла
class InlineWorkerBuilder {
  static create(workerFunction) {
    const workerCode = `
      ${workerFunction.toString()}
      
      self.onmessage = function(event) {
        const { id, data } = event.data;
        
        try {
          const result = (${workerFunction.name})(data);
          
          // Если результат Promise
          if (result && typeof result.then === 'function') {
            result
              .then(data => {
                self.postMessage({ id, data });
              })
              .catch(error => {
                self.postMessage({ id, error: error.message });
              });
          } else {
            self.postMessage({ id, data: result });
          }
        } catch (error) {
          self.postMessage({ id, error: error.message });
        }
      };
    `;
    
    const blob = new Blob([workerCode], { type: 'application/javascript' });
    return URL.createObjectURL(blob);
  }
}

// Глобальный менеджер
const workerManager = new WorkerManager();

// Пример функции для выполнения в worker'е
function heavyCalculation(data) {
  const { numbers, operation } = data;
  
  switch (operation) {
    case 'fibonacci':
      return numbers.map(n => fibonacci(n));
    case 'isPrime':
      return numbers.map(n => isPrime(n));
    case 'factorial':
      return numbers.map(n => factorial(n));
    default:
      throw new Error('Неизвестная операция');
  }
  
  function fibonacci(n) {
    if (n <= 1) return n;
    let a = 0, b = 1;
    for (let i = 2; i <= n; i++) {
      [a, b] = [b, a + b];
    }
    return b;
  }
  
  function isPrime(n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 === 0 || n % 3 === 0) return false;
    
    for (let i = 5; i * i <= n; i += 6) {
      if (n % i === 0 || n % (i + 2) === 0) return false;
    }
    return true;
  }
  
  function factorial(n) {
    if (n <= 1) return 1;
    let result = 1;
    for (let i = 2; i <= n; i++) {
      result *= i;
    }
    return result;
  }
}

// Использование inline worker
async function performHeavyCalculation() {
  const workerUrl = InlineWorkerBuilder.create(heavyCalculation);
  const workerId = workerManager.createWorker(workerUrl);
  
  try {
    const result = await workerManager.sendMessage(workerId, {
      numbers: [40, 41, 42, 43, 44],
      operation: 'fibonacci'
    });
    
    console.log('Результат вычислений:', result);
    return result;
  } finally {
    workerManager.terminateWorker(workerId);
    URL.revokeObjectURL(workerUrl);
  }
}
```

---

## 👥 Dedicated Workers

### Обработка изображений

```javascript
// Worker для обработки изображений
class ImageProcessingWorker {
  constructor() {
    this.workerId = null;
    this.initWorker();
  }
  
  initWorker() {
    const workerCode = `
      self.onmessage = function(event) {
        const { id, operation, imageData, options } = event.data;
        
        try {
          let result;
          
          switch (operation) {
            case 'grayscale':
              result = applyGrayscale(imageData);
              break;
            case 'blur':
              result = applyBlur(imageData, options.radius);
              break;
            case 'brightness':
              result = adjustBrightness(imageData, options.value);
              break;
            case 'contrast':
              result = adjustContrast(imageData, options.value);
              break;
            default:
              throw new Error('Неизвестная операция: ' + operation);
          }
          
          self.postMessage({ id, result }, [result.data.buffer]);
        } catch (error) {
          self.postMessage({ id, error: error.message });
        }
      };
      
      function applyGrayscale(imageData) {
        const data = new Uint8ClampedArray(imageData.data);
        
        for (let i = 0; i < data.length; i += 4) {
          const gray = data[i] * 0.299 + data[i + 1] * 0.587 + data[i + 2] * 0.114;
          data[i] = gray;     // R
          data[i + 1] = gray; // G
          data[i + 2] = gray; // B
          // Alpha остается без изменений
        }
        
        return new ImageData(data, imageData.width, imageData.height);
      }
      
      function applyBlur(imageData, radius) {
        const data = new Uint8ClampedArray(imageData.data);
        const width = imageData.width;
        const height = imageData.height;
        
        // Простое box blur
        for (let y = 0; y < height; y++) {
          for (let x = 0; x < width; x++) {
            let r = 0, g = 0, b = 0, count = 0;
            
            for (let dy = -radius; dy <= radius; dy++) {
              for (let dx = -radius; dx <= radius; dx++) {
                const nx = x + dx;
                const ny = y + dy;
                
                if (nx >= 0 && nx < width && ny >= 0 && ny < height) {
                  const index = (ny * width + nx) * 4;
                  r += data[index];
                  g += data[index + 1];
                  b += data[index + 2];
                  count++;
                }
              }
            }
            
            const index = (y * width + x) * 4;
            data[index] = r / count;
            data[index + 1] = g / count;
            data[index + 2] = b / count;
          }
        }
        
        return new ImageData(data, width, height);
      }
      
      function adjustBrightness(imageData, value) {
        const data = new Uint8ClampedArray(imageData.data);
        
        for (let i = 0; i < data.length; i += 4) {
          data[i] = Math.min(255, Math.max(0, data[i] + value));     // R
          data[i + 1] = Math.min(255, Math.max(0, data[i + 1] + value)); // G
          data[i + 2] = Math.min(255, Math.max(0, data[i + 2] + value)); // B
        }
        
        return new ImageData(data, imageData.width, imageData.height);
      }
      
      function adjustContrast(imageData, value) {
        const data = new Uint8ClampedArray(imageData.data);
        const factor = (259 * (value + 255)) / (255 * (259 - value));
        
        for (let i = 0; i < data.length; i += 4) {
          data[i] = Math.min(255, Math.max(0, factor * (data[i] - 128) + 128));
          data[i + 1] = Math.min(255, Math.max(0, factor * (data[i + 1] - 128) + 128));
          data[i + 2] = Math.min(255, Math.max(0, factor * (data[i + 2] - 128) + 128));
        }
        
        return new ImageData(data, imageData.width, imageData.height);
      }
    `;
    
    const blob = new Blob([workerCode], { type: 'application/javascript' });
    const workerUrl = URL.createObjectURL(blob);
    
    this.workerId = workerManager.createWorker(workerUrl);
    URL.revokeObjectURL(workerUrl);
  }
  
  async processImage(canvas, operation, options = {}) {
    const ctx = canvas.getContext('2d');
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    
    try {
      const result = await workerManager.sendMessage(this.workerId, {
        operation: operation,
        imageData: imageData,
        options: options
      }, [imageData.data.buffer]);
      
      // Применяем результат к canvas
      ctx.putImageData(result, 0, 0);
      return result;
      
    } catch (error) {
      console.error('Ошибка обработки изображения:', error);
      throw error;
    }
  }
  
  destroy() {
    if (this.workerId) {
      workerManager.terminateWorker(this.workerId);
      this.workerId = null;
    }
  }
}

// Пример использования обработки изображений
const imageProcessor = new ImageProcessingWorker();

async function applyImageFilters() {
  const canvas = document.getElementById('imageCanvas');
  
  try {
    // Применяем фильтры последовательно
    await imageProcessor.processImage(canvas, 'brightness', { value: 20 });
    await imageProcessor.processImage(canvas, 'contrast', { value: 30 });
    await imageProcessor.processImage(canvas, 'blur', { radius: 2 });
    
    console.log('Фильтры применены');
  } catch (error) {
    console.error('Ошибка применения фильтров:', error);
  }
}
```

---

## 🔄 Shared Workers

### Общее состояние между вкладками

```javascript
// Создание Shared Worker
class SharedStateManager {
  constructor() {
    this.worker = null;
    this.callbacks = new Map();
    this.messageId = 0;
    this.initSharedWorker();
  }
  
  initSharedWorker() {
    const workerCode = `
      // Shared Worker code
      const connections = new Set();
      const sharedState = new Map();
      
      self.onconnect = function(event) {
        const port = event.ports[0];
        connections.add(port);
        
        port.onmessage = function(event) {
          const { id, action, key, value } = event.data;
          
          try {
            let result;
            
            switch (action) {
              case 'set':
                sharedState.set(key, value);
                // Уведомляем все подключения об изменении
                broadcastToOthers(port, { action: 'stateChanged', key, value });
                result = true;
                break;
                
              case 'get':
                result = sharedState.get(key);
                break;
                
              case 'delete':
                result = sharedState.delete(key);
                broadcastToOthers(port, { action: 'stateDeleted', key });
                break;
                
              case 'getAll':
                result = Object.fromEntries(sharedState);
                break;
                
              case 'clear':
                sharedState.clear();
                broadcastToOthers(port, { action: 'stateCleared' });
                result = true;
                break;
                
              default:
                throw new Error('Неизвестное действие: ' + action);
            }
            
            port.postMessage({ id, result });
          } catch (error) {
            port.postMessage({ id, error: error.message });
          }
        };
        
        port.onclose = function() {
          connections.delete(port);
        };
        
        // Отправляем текущее состояние новому подключению
        port.postMessage({
          action: 'initialState',
          state: Object.fromEntries(sharedState)
        });
      };
      
      function broadcastToOthers(sender, message) {
        connections.forEach(port => {
          if (port !== sender) {
            port.postMessage(message);
          }
        });
      }
    `;
    
    try {
      const blob = new Blob([workerCode], { type: 'application/javascript' });
      const workerUrl = URL.createObjectURL(blob);
      
      this.worker = new SharedWorker(workerUrl);
      this.worker.port.start();
      
      // Обработка сообщений от Shared Worker
      this.worker.port.onmessage = (event) => {
        this.handleMessage(event.data);
      };
      
      URL.revokeObjectURL(workerUrl);
    } catch (error) {
      console.error('Ошибка создания Shared Worker:', error);
    }
  }
  
  handleMessage(data) {
    const { id, result, error, action, key, value, state } = data;
    
    if (id && this.callbacks.has(id)) {
      // Ответ на запрос
      const { resolve, reject } = this.callbacks.get(id);
      this.callbacks.delete(id);
      
      if (error) {
        reject(new Error(error));
      } else {
        resolve(result);
      }
    } else if (action) {
      // Уведомление об изменении состояния
      this.handleStateChange(action, key, value, state);
    }
  }
  
  handleStateChange(action, key, value, state) {
    switch (action) {
      case 'initialState':
        this.onInitialState?.(state);
        break;
      case 'stateChanged':
        this.onStateChanged?.(key, value);
        break;
      case 'stateDeleted':
        this.onStateDeleted?.(key);
        break;
      case 'stateCleared':
        this.onStateCleared?.();
        break;
    }
  }
  
  async sendCommand(action, key, value) {
    if (!this.worker) {
      throw new Error('Shared Worker не инициализирован');
    }
    
    const id = ++this.messageId;
    
    return new Promise((resolve, reject) => {
      this.callbacks.set(id, { resolve, reject });
      
      this.worker.port.postMessage({
        id: id,
        action: action,
        key: key,
        value: value
      });
      
      // Таймаут
      setTimeout(() => {
        if (this.callbacks.has(id)) {
          this.callbacks.delete(id);
          reject(new Error('Таймаут команды'));
        }
      }, 5000);
    });
  }
  
  // Методы для работы с состоянием
  async set(key, value) {
    return this.sendCommand('set', key, value);
  }
  
  async get(key) {
    return this.sendCommand('get', key);
  }
  
  async delete(key) {
    return this.sendCommand('delete', key);
  }
  
  async getAll() {
    return this.sendCommand('getAll');
  }
  
  async clear() {
    return this.sendCommand('clear');
  }
}

// Глобальный менеджер состояния
const sharedState = new SharedStateManager();

// Обработчики изменений состояния
sharedState.onStateChanged = (key, value) => {
  console.log(`Состояние изменено: ${key} = ${value}`);
  
  // Обновляем UI
  const element = document.querySelector(`[data-state-key="${key}"]`);
  if (element) {
    element.textContent = value;
  }
};

sharedState.onStateDeleted = (key) => {
  console.log(`Состояние удалено: ${key}`);
};

// Пример использования
async function updateSharedCounter() {
  try {
    const currentCount = await sharedState.get('counter') || 0;
    await sharedState.set('counter', currentCount + 1);
  } catch (error) {
    console.error('Ошибка обновления счетчика:', error);
  }
}
```

---

## 🎯 Практические применения

### Обработка больших массивов данных

```javascript
// Worker для обработки больших датасетов
class DataProcessingWorker {
  constructor() {
    this.workerId = null;
    this.initWorker();
  }
  
  initWorker() {
    const workerCode = `
      self.onmessage = function(event) {
        const { id, operation, data, options } = event.data;
        
        try {
          let result;
          
          switch (operation) {
            case 'filter':
              result = filterData(data, options.predicate);
              break;
            case 'map':
              result = mapData(data, options.mapper);
              break;
            case 'reduce':
              result = reduceData(data, options.reducer, options.initialValue);
              break;
            case 'sort':
              result = sortData(data, options.compareFn);
              break;
            case 'aggregate':
              result = aggregateData(data, options.groupBy, options.aggregators);
              break;
            case 'search':
              result = searchData(data, options.query, options.fields);
              break;
            default:
              throw new Error('Неизвестная операция: ' + operation);
          }
          
          self.postMessage({ id, result });
        } catch (error) {
          self.postMessage({ id, error: error.message });
        }
      };
      
      function filterData(data, predicateStr) {
        const predicate = new Function('item', 'index', 'array', 'return ' + predicateStr);
        return data.filter(predicate);
      }
      
      function mapData(data, mapperStr) {
        const mapper = new Function('item', 'index', 'array', 'return ' + mapperStr);
        return data.map(mapper);
      }
      
      function reduceData(data, reducerStr, initialValue) {
        const reducer = new Function('acc', 'item', 'index', 'array', 'return ' + reducerStr);
        return data.reduce(reducer, initialValue);
      }
      
      function sortData(data, compareFnStr) {
        if (compareFnStr) {
          const compareFn = new Function('a', 'b', 'return ' + compareFnStr);
          return [...data].sort(compareFn);
        }
        return [...data].sort();
      }
      
      function aggregateData(data, groupByField, aggregators) {
        const groups = {};
        
        data.forEach(item => {
          const key = item[groupByField];
          if (!groups[key]) {
            groups[key] = [];
          }
          groups[key].push(item);
        });
        
        const result = {};
        
        Object.keys(groups).forEach(key => {
          const group = groups[key];
          result[key] = {};
          
          aggregators.forEach(agg => {
            switch (agg.type) {
              case 'count':
                result[key][agg.name] = group.length;
                break;
              case 'sum':
                result[key][agg.name] = group.reduce((sum, item) => sum + (item[agg.field] || 0), 0);
                break;
              case 'avg':
                const sum = group.reduce((s, item) => s + (item[agg.field] || 0), 0);
                result[key][agg.name] = sum / group.length;
                break;
              case 'min':
                result[key][agg.name] = Math.min(...group.map(item => item[agg.field] || 0));
                break;
              case 'max':
                result[key][agg.name] = Math.max(...group.map(item => item[agg.field] || 0));
                break;
            }
          });
        });
        
        return result;
      }
      
      function searchData(data, query, fields) {
        const lowerQuery = query.toLowerCase();
        
        return data.filter(item => {
          return fields.some(field => {
            const value = item[field];
            return value && value.toString().toLowerCase().includes(lowerQuery);
          });
        });
      }
    `;
    
    const blob = new Blob([workerCode], { type: 'application/javascript' });
    const workerUrl = URL.createObjectURL(blob);
    
    this.workerId = workerManager.createWorker(workerUrl);
    URL.revokeObjectURL(workerUrl);
  }
  
  async processData(operation, data, options = {}) {
    try {
      return await workerManager.sendMessage(this.workerId, {
        operation: operation,
        data: data,
        options: options
      });
    } catch (error) {
      console.error('Ошибка обработки данных:', error);
      throw error;
    }
  }
  
  destroy() {
    if (this.workerId) {
      workerManager.terminateWorker(this.workerId);
      this.workerId = null;
    }
  }
}

// Пример использования с большим массивом данных
async function processLargeDataset() {
  const dataProcessor = new DataProcessingWorker();
  
  // Генерируем большой массив тестовых данных
  const largeDataset = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    name: `User ${i}`,
    age: Math.floor(Math.random() * 80) + 18,
    city: ['Moscow', 'SPB', 'Kazan', 'Ekb'][Math.floor(Math.random() * 4)],
    salary: Math.floor(Math.random() * 100000) + 30000
  }));
  
  console.time('Data processing');
  
  try {
    // Фильтруем пользователей старше 30
    const adults = await dataProcessor.processData('filter', largeDataset, {
      predicate: 'item.age > 30'
    });
    
    console.log('Взрослых пользователей:', adults.length);
    
    // Группируем по городам с агрегацией
    const cityStats = await dataProcessor.processData('aggregate', largeDataset, {
      groupBy: 'city',
      aggregators: [
        { type: 'count', name: 'total' },
        { type: 'avg', name: 'avgAge', field: 'age' },
        { type: 'avg', name: 'avgSalary', field: 'salary' },
        { type: 'max', name: 'maxSalary', field: 'salary' }
      ]
    });
    
    console.log('Статистика по городам:', cityStats);
    
    // Поиск пользователей
    const searchResults = await dataProcessor.processData('search', largeDataset, {
      query: 'User 123',
      fields: ['name']
    });
    
    console.log('Результаты поиска:', searchResults.length);
    
  } finally {
    console.timeEnd('Data processing');
    dataProcessor.destroy();
  }
}
```

### Криптографические операции

```javascript
// Worker для криптографических операций
class CryptoWorker {
  constructor() {
    this.workerId = null;
    this.initWorker();
  }
  
  initWorker() {
    const workerCode = `
      // Импортируем crypto API в worker
      importScripts('https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js');
      
      self.onmessage = function(event) {
        const { id, operation, data, options } = event.data;
        
        try {
          let result;
          
          switch (operation) {
            case 'hash':
              result = hashData(data, options.algorithm);
              break;
            case 'encrypt':
              result = encryptData(data, options.key, options.algorithm);
              break;
            case 'decrypt':
              result = decryptData(data, options.key, options.algorithm);
              break;
            case 'generateKeyPair':
              result = generateKeyPair(options.algorithm, options.keySize);
              break;
            case 'pbkdf2':
              result = deriveKey(data, options.salt, options.iterations);
              break;
            default:
              throw new Error('Неизвестная операция: ' + operation);
          }
          
          self.postMessage({ id, result });
        } catch (error) {
          self.postMessage({ id, error: error.message });
        }
      };
      
      function hashData(data, algorithm = 'SHA256') {
        const hash = CryptoJS[algorithm](data);
        return hash.toString(CryptoJS.enc.Hex);
      }
      
      function encryptData(data, key, algorithm = 'AES') {
        const encrypted = CryptoJS[algorithm].encrypt(data, key);
        return encrypted.toString();
      }
      
      function decryptData(encryptedData, key, algorithm = 'AES') {
        const decrypted = CryptoJS[algorithm].decrypt(encryptedData, key);
        return decrypted.toString(CryptoJS.enc.Utf8);
      }
      
      function deriveKey(password, salt, iterations = 10000) {
        const key = CryptoJS.PBKDF2(password, salt, {
          keySize: 256/32,
          iterations: iterations
        });
        return key.toString(CryptoJS.enc.Hex);
      }
      
      function generateKeyPair(algorithm, keySize) {
        // Упрощенная генерация ключей (в реальности используйте WebCrypto API)
        const key1 = CryptoJS.lib.WordArray.random(keySize / 8);
        const key2 = CryptoJS.lib.WordArray.random(keySize / 8);
        
        return {
          publicKey: key1.toString(CryptoJS.enc.Hex),
          privateKey: key2.toString(CryptoJS.enc.Hex)
        };
      }
    `;
    
    const blob = new Blob([workerCode], { type: 'application/javascript' });
    const workerUrl = URL.createObjectURL(blob);
    
    this.workerId = workerManager.createWorker(workerUrl);
    URL.revokeObjectURL(workerUrl);
  }
  
  async hash(data, algorithm = 'SHA256') {
    return this.performCryptoOperation('hash', data, { algorithm });
  }
  
  async encrypt(data, key, algorithm = 'AES') {
    return this.performCryptoOperation('encrypt', data, { key, algorithm });
  }
  
  async decrypt(encryptedData, key, algorithm = 'AES') {
    return this.performCryptoOperation('decrypt', encryptedData, { key, algorithm });
  }
  
  async deriveKey(password, salt, iterations = 10000) {
    return this.performCryptoOperation('pbkdf2', password, { salt, iterations });
  }
  
  async performCryptoOperation(operation, data, options) {
    try {
      return await workerManager.sendMessage(this.workerId, {
        operation: operation,
        data: data,
        options: options
      });
    } catch (error) {
      console.error('Ошибка криптографической операции:', error);
      throw error;
    }
  }
  
  destroy() {
    if (this.workerId) {
      workerManager.terminateWorker(this.workerId);
      this.workerId = null;
    }
  }
}

// Пример использования криптографии
const cryptoWorker = new CryptoWorker();

async function encryptUserData() {
  const userData = JSON.stringify({
    username: 'john_doe',
    email: 'john@example.com',
    preferences: { theme: 'dark', lang: 'ru' }
  });
  
  const password = 'user-password-123';
  const salt = 'random-salt-string';
  
  try {
    // Генерируем ключ из пароля
    const derivedKey = await cryptoWorker.deriveKey(password, salt);
    console.log('Ключ сгенерирован');
    
    // Шифруем данные
    const encrypted = await cryptoWorker.encrypt(userData, derivedKey);
    console.log('Данные зашифрованы:', encrypted);
    
    // Расшифровываем для проверки
    const decrypted = await cryptoWorker.decrypt(encrypted, derivedKey);
    console.log('Данные расшифрованы:', JSON.parse(decrypted));
    
  } catch (error) {
    console.error('Ошибка криптографии:', error);
  }
}
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **Basic Worker** - Простой worker для математических вычислений
2. **Image Processing** - Фильтры для изображений в worker'е
3. **Data Filtering** - Фильтрация больших массивов

### 🟡 Средний уровень
4. **Shared State** - Общее состояние между вкладками через Shared Worker
5. **File Processing** - Обработка файлов в background
6. **Search Engine** - Поиск по большим объемам данных

### 🔴 Продвинутый уровень
7. **Crypto Operations** - Криптографические операции
8. **Machine Learning** - ML вычисления в worker'ах
9. **Real-time Analytics** - Анализ данных в реальном времени

## 🔗 Связанные разделы

- **[[service-workers-pwa|🚀 Service Workers]]** - Кэширование и PWA
- **[[intersection-observer|👁️ Intersection Observer]]** - Оптимизация с worker'ами
- **[[storage-apis|💾 Storage APIs]]** - Общее состояние
- **[[../../technical-skills/security|🔒 Security]]** - Безопасность в worker'ах

---

⏱️ **Время изучения**: 4-5 часов  
🎯 **Уровень**: Средний - Продвинутый  
📋 **Предварительные требования**: JavaScript ES6+, асинхронное программирование, основы криптографии 