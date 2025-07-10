# 🚀 JavaScript Fundamentals

## 📖 Содержание
- [ES6+ Синтаксис](#es6-синтаксис)
- [Асинхронное программирование](#асинхронное-программирование)
- [Работа с DOM](#работа-с-dom)
- [Модули](#модули)
- [Функциональное программирование](#функциональное-программирование)

---

## 🔄 ES6+ Синтаксис

### Переменные и константы

```javascript
// ❌ Избегайте var - имеет function scope
var oldWay = 'проблематично';

// ✅ Используйте let для переменных - block scope
let mutableValue = 'можно изменить';
mutableValue = 'изменено';

// ✅ Используйте const для констант - block scope
const immutableValue = 'нельзя переназначить';
const user = { name: 'John' }; // Объект const можно мутировать
user.name = 'Jane'; // ✅ Это работает
// user = {}; // ❌ Это вызовет ошибку
```

### Деструктуризация

```javascript
// Деструктуризация объектов
const user = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  settings: {
    theme: 'dark',
    notifications: true
  }
};

// Простая деструктуризация
const { name, email } = user;

// С переименованием
const { name: userName, email: userEmail } = user;

// С значениями по умолчанию
const { name, age = 25 } = user;

// Вложенная деструктуризация
const { settings: { theme, notifications } } = user;

// Деструктуризация массивов
const colors = ['red', 'green', 'blue', 'yellow'];
const [primary, secondary, ...rest] = colors;
// primary = 'red', secondary = 'green', rest = ['blue', 'yellow']

// Пропуск элементов
const [first, , third] = colors;
// first = 'red', third = 'blue'

// В параметрах функции
function displayUser({ name, email, age = 'unknown' }) {
  console.log(`${name} (${email}) - возраст: ${age}`);
}

displayUser(user);
```

### Arrow Functions

```javascript
// Традиционная функция
function traditionalFunction(x, y) {
  return x + y;
}

// Arrow function - краткая запись
const arrowFunction = (x, y) => x + y;

// С одним параметром скобки не нужны
const square = x => x * x;

// Без параметров нужны пустые скобки
const greet = () => 'Привет!';

// С блоком кода нужен explicit return
const complexFunction = (x, y) => {
  const sum = x + y;
  return sum * 2;
};

// Arrow functions не имеют собственного this
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    // ✅ Arrow function наследует this из класса
    setTimeout(() => {
      this.count++;
      console.log(this.count);
    }, 1000);
  }
  
  incrementWrong() {
    // ❌ Обычная function имеет свой this
    setTimeout(function() {
      this.count++; // this здесь undefined или window
    }, 1000);
  }
}
```

### Template Literals

```javascript
const name = 'John';
const age = 30;

// ❌ Конкатенация строк
const oldWay = 'Привет, меня зовут ' + name + ' и мне ' + age + ' лет';

// ✅ Template literals
const newWay = `Привет, меня зовут ${name} и мне ${age} лет`;

// Многострочные строки
const html = `
  <div class="user-card">
    <h2>${name}</h2>
    <p>Возраст: ${age}</p>
    <p>Статус: ${age >= 18 ? 'Взрослый' : 'Несовершеннолетний'}</p>
  </div>
`;

// Вызов функций в шаблонах
const formatDate = date => date.toLocaleDateString('ru-RU');
const today = new Date();
const message = `Сегодня ${formatDate(today)}`;
```

---

## ⚡ Асинхронное программирование

### Promises

```javascript
// Создание Promise
const fetchUserData = (userId) => {
  return new Promise((resolve, reject) => {
    // Симуляция API запроса
    setTimeout(() => {
      if (userId > 0) {
        resolve({
          id: userId,
          name: 'John Doe',
          email: 'john@example.com'
        });
      } else {
        reject(new Error('Invalid user ID'));
      }
    }, 1000);
  });
};

// Использование Promise
fetchUserData(1)
  .then(user => {
    console.log('Пользователь получен:', user);
    return user.email; // Возвращаем email для следующего .then
  })
  .then(email => {
    console.log('Email пользователя:', email);
  })
  .catch(error => {
    console.error('Ошибка:', error.message);
  })
  .finally(() => {
    console.log('Запрос завершен');
  });

// Promise.all - ждет выполнения всех промисов
const fetchMultipleUsers = async () => {
  try {
    const users = await Promise.all([
      fetchUserData(1),
      fetchUserData(2),
      fetchUserData(3)
    ]);
    console.log('Все пользователи:', users);
  } catch (error) {
    console.error('Один из запросов упал:', error);
  }
};

// Promise.allSettled - ждет завершения всех, независимо от результата
const fetchUsersWithErrors = async () => {
  const results = await Promise.allSettled([
    fetchUserData(1),
    fetchUserData(-1), // Вызовет ошибку
    fetchUserData(2)
  ]);
  
  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      console.log(`Пользователь ${index + 1}:`, result.value);
    } else {
      console.error(`Ошибка ${index + 1}:`, result.reason.message);
    }
  });
};
```

### Async/Await

```javascript
// Функция с async/await
async function fetchAndDisplayUser(userId) {
  try {
    // await ждет выполнения Promise
    const user = await fetchUserData(userId);
    console.log('Пользователь:', user);
    
    // Можно использовать результат дальше
    const profileData = await fetchUserProfile(user.id);
    console.log('Профиль:', profileData);
    
    return { user, profile: profileData };
  } catch (error) {
    console.error('Произошла ошибка:', error.message);
    throw error; // Перебрасываем ошибку выше
  }
}

// Обработка ошибок в async функциях
async function handleUserData() {
  try {
    const result = await fetchAndDisplayUser(1);
    updateUI(result);
  } catch (error) {
    showErrorMessage(error.message);
  }
}

// Параллельные запросы с async/await
async function fetchUsersInParallel() {
  // ❌ Последовательное выполнение - медленно
  const user1 = await fetchUserData(1);
  const user2 = await fetchUserData(2);
  
  // ✅ Параллельное выполнение - быстрее
  const [user1Fast, user2Fast] = await Promise.all([
    fetchUserData(1),
    fetchUserData(2)
  ]);
  
  return [user1Fast, user2Fast];
}

// Работа с реальными API
async function fetchRealData() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    
    // Проверяем успешность запроса
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const users = await response.json();
    console.log('Пользователи из API:', users);
    
    return users;
  } catch (error) {
    if (error instanceof TypeError) {
      console.error('Сетевая ошибка:', error.message);
    } else {
      console.error('Ошибка API:', error.message);
    }
    throw error;
  }
}
```

---

## 🌐 Работа с DOM

### Современные способы выбора элементов

```javascript
// Современные селекторы
const element = document.querySelector('.my-class'); // Первый элемент
const elements = document.querySelectorAll('.my-class'); // Все элементы (NodeList)

// Преобразование NodeList в массив для использования методов массива
const elementsArray = [...elements]; // Spread оператор
const elementsArray2 = Array.from(elements); // Array.from

// Работа с данными элементов
elementsArray.forEach(el => {
  console.log(el.textContent);
});

// Фильтрация элементов
const visibleElements = elementsArray.filter(el => 
  el.style.display !== 'none'
);
```

### Создание и изменение элементов

```javascript
// Создание элементов
function createUserCard(user) {
  // Создаем элементы
  const card = document.createElement('div');
  const title = document.createElement('h3');
  const email = document.createElement('p');
  const button = document.createElement('button');
  
  // Добавляем классы и атрибуты
  card.className = 'user-card';
  card.dataset.userId = user.id; // data-user-id
  
  // Устанавливаем содержимое
  title.textContent = user.name;
  email.textContent = user.email;
  button.textContent = 'Подробнее';
  
  // Собираем структуру
  card.appendChild(title);
  card.appendChild(email);
  card.appendChild(button);
  
  // Добавляем обработчик события
  button.addEventListener('click', () => {
    showUserDetails(user.id);
  });
  
  return card;
}

// Вставка в DOM
function displayUsers(users) {
  const container = document.querySelector('#users-container');
  
  // Очищаем контейнер
  container.innerHTML = '';
  
  // Создаем и добавляем карточки
  users.forEach(user => {
    const card = createUserCard(user);
    container.appendChild(card);
  });
}

// Работа с классами
function toggleTheme() {
  const body = document.body;
  
  // Современные методы работы с классами
  body.classList.toggle('dark-theme');
  
  // Другие методы
  // body.classList.add('dark-theme');
  // body.classList.remove('dark-theme');
  // body.classList.contains('dark-theme');
}
```

### События

```javascript
// Современная обработка событий
class FormHandler {
  constructor(formSelector) {
    this.form = document.querySelector(formSelector);
    this.init();
  }
  
  init() {
    // Обработка отправки формы
    this.form.addEventListener('submit', this.handleSubmit.bind(this));
    
    // Обработка изменений инпутов
    this.form.addEventListener('input', this.handleInput.bind(this));
    
    // Делегирование событий для динамических элементов
    this.form.addEventListener('click', this.handleClick.bind(this));
  }
  
  handleSubmit(event) {
    event.preventDefault(); // Отменяем стандартное поведение
    
    const formData = new FormData(this.form);
    const data = Object.fromEntries(formData); // Преобразуем в объект
    
    this.submitData(data);
  }
  
  handleInput(event) {
    const { target } = event;
    
    // Валидация в реальном времени
    if (target.type === 'email') {
      this.validateEmail(target);
    }
  }
  
  handleClick(event) {
    // Делегирование событий
    if (event.target.matches('.remove-button')) {
      this.removeItem(event.target.closest('.item'));
    }
  }
  
  validateEmail(input) {
    const isValid = input.value.includes('@');
    input.classList.toggle('invalid', !isValid);
  }
  
  async submitData(data) {
    try {
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data)
      });
      
      if (response.ok) {
        this.showSuccess();
      } else {
        throw new Error('Ошибка отправки');
      }
    } catch (error) {
      this.showError(error.message);
    }
  }
  
  showSuccess() {
    this.showMessage('Данные успешно отправлены!', 'success');
  }
  
  showError(message) {
    this.showMessage(message, 'error');
  }
  
  showMessage(text, type) {
    const message = document.createElement('div');
    message.className = `message ${type}`;
    message.textContent = text;
    
    this.form.appendChild(message);
    
    // Автоматическое удаление через 3 секунды
    setTimeout(() => {
      message.remove();
    }, 3000);
  }
}

// Использование
document.addEventListener('DOMContentLoaded', () => {
  new FormHandler('#user-form');
});
```

---

## 📦 Модули

### ES6 Modules

```javascript
// utils.js - экспорт функций
export const formatDate = (date) => {
  return date.toLocaleDateString('ru-RU');
};

export const debounce = (func, wait) => {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
};

// Экспорт по умолчанию
export default class ApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }
  
  async get(endpoint) {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    return response.json();
  }
  
  async post(endpoint, data) {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

// main.js - импорт модулей
import ApiClient from './utils.js'; // Импорт по умолчанию
import { formatDate, debounce } from './utils.js'; // Именованный импорт

// Импорт с переименованием
import { formatDate as formatDateRU } from './utils.js';

// Импорт всего модуля
import * as Utils from './utils.js';

// Использование
const api = new ApiClient('https://api.example.com');
const formattedDate = formatDate(new Date());
const debouncedSearch = debounce(searchFunction, 300);
```

---

## 🎯 Функциональное программирование

### Методы массивов

```javascript
const users = [
  { id: 1, name: 'John', age: 25, department: 'IT', salary: 50000 },
  { id: 2, name: 'Jane', age: 30, department: 'HR', salary: 45000 },
  { id: 3, name: 'Bob', age: 35, department: 'IT', salary: 60000 },
  { id: 4, name: 'Alice', age: 28, department: 'Marketing', salary: 48000 }
];

// map - преобразование элементов
const userNames = users.map(user => user.name);
const userSummaries = users.map(user => ({
  id: user.id,
  summary: `${user.name} (${user.age} лет) - ${user.department}`
}));

// filter - фильтрация элементов
const itUsers = users.filter(user => user.department === 'IT');
const youngUsers = users.filter(user => user.age < 30);
const highEarners = users.filter(user => user.salary > 48000);

// find - поиск первого элемента
const johnUser = users.find(user => user.name === 'John');
const firstItUser = users.find(user => user.department === 'IT');

// reduce - агрегация данных
const totalSalary = users.reduce((sum, user) => sum + user.salary, 0);
const averageSalary = totalSalary / users.length;

// Группировка по департаментам
const usersByDepartment = users.reduce((groups, user) => {
  const dept = user.department;
  if (!groups[dept]) {
    groups[dept] = [];
  }
  groups[dept].push(user);
  return groups;
}, {});

// Комплексная обработка данных
const departmentStats = users
  .filter(user => user.age >= 25) // Только взрослые сотрудники
  .reduce((stats, user) => {
    const dept = user.department;
    if (!stats[dept]) {
      stats[dept] = {
        count: 0,
        totalSalary: 0,
        employees: []
      };
    }
    stats[dept].count += 1;
    stats[dept].totalSalary += user.salary;
    stats[dept].employees.push(user.name);
    return stats;
  }, {});

// Добавляем средние зарплаты
Object.keys(departmentStats).forEach(dept => {
  const stat = departmentStats[dept];
  stat.averageSalary = Math.round(stat.totalSalary / stat.count);
});

console.log(departmentStats);
```

### Высшие функции и композиция

```javascript
// Высшие функции - функции, которые принимают или возвращают другие функции
const createValidator = (rule) => (value) => rule(value);

const isRequired = createValidator(value => 
  value !== null && value !== undefined && value !== ''
);

const isEmail = createValidator(value => 
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
);

const minLength = (min) => createValidator(value => 
  value && value.length >= min
);

// Композиция функций
const compose = (...functions) => (value) =>
  functions.reduceRight((acc, fn) => fn(acc), value);

const pipe = (...functions) => (value) =>
  functions.reduce((acc, fn) => fn(acc), value);

// Пример использования композиции
const processUserData = pipe(
  (user) => ({ ...user, name: user.name.trim() }),
  (user) => ({ ...user, email: user.email.toLowerCase() }),
  (user) => ({ ...user, createdAt: new Date() })
);

const processedUser = processUserData({
  name: '  John Doe  ',
  email: 'JOHN@EXAMPLE.COM'
});

// Каррирование
const multiply = (a) => (b) => a * b;
const double = multiply(2);
const triple = multiply(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Практический пример: система валидации
class FormValidator {
  constructor() {
    this.rules = new Map();
  }
  
  addRule(fieldName, ...validators) {
    this.rules.set(fieldName, validators);
    return this; // Для цепочки методов
  }
  
  validate(data) {
    const errors = {};
    
    for (const [fieldName, validators] of this.rules) {
      const value = data[fieldName];
      const fieldErrors = [];
      
      for (const validator of validators) {
        if (!validator(value)) {
          fieldErrors.push(validator.message || 'Некорректное значение');
        }
      }
      
      if (fieldErrors.length > 0) {
        errors[fieldName] = fieldErrors;
      }
    }
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }
}

// Использование валидатора
const userValidator = new FormValidator()
  .addRule('name', isRequired, minLength(2))
  .addRule('email', isRequired, isEmail);

const validationResult = userValidator.validate({
  name: 'John',
  email: 'john@example.com'
});

console.log(validationResult); // { isValid: true, errors: {} }
```

---

## 📚 Связанные разделы

- [[html-css-fundamentals|HTML & CSS Fundamentals]] - Основы веб-технологий
- [[../react-ecosystem|React Ecosystem]] - Современная библиотека для UI
- [[frontend-advanced|Frontend Advanced]] - Продвинутые техники frontend
- [[../technical-skills/testing|Testing]] - Тестирование JavaScript кода
- [[../fundamentals/algorithms-basics|Algorithms]] - Алгоритмы на JavaScript 