# ⚛️ React Patterns & Best Practices

## 📖 Содержание
- [Compound Components](#compound-components)
- [Render Props](#render-props) 
- [Higher-Order Components](#higher-order-components)
- [Custom Hooks](#custom-hooks)
- [State Management Patterns](#state-management-patterns)
- [Performance Patterns](#performance-patterns)

---

## 🧩 Compound Components

### Теория
Compound Components позволяют создавать компоненты, которые работают вместе для формирования единого функционала, предоставляя гибкий API.

### Практическая реализация

```tsx
// Modal.tsx - Compound Component
import React, { createContext, useContext, useState } from 'react';

// Контекст для передачи состояния между компонентами
interface ModalContextType {
  isOpen: boolean;
  open: () => void;
  close: () => void;
}

const ModalContext = createContext<ModalContextType | null>(null);

// Hook для использования контекста модального окна
const useModal = () => {
  const context = useContext(ModalContext);
  if (!context) {
    throw new Error('useModal должен использоваться внутри Modal');
  }
  return context;
};

// Основной компонент Modal
const Modal: React.FC<{ children: React.ReactNode }> & {
  Trigger: React.FC<{ children: React.ReactNode }>;
  Content: React.FC<{ children: React.ReactNode }>;
  Header: React.FC<{ children: React.ReactNode }>;
  Body: React.FC<{ children: React.ReactNode }>;
  Footer: React.FC<{ children: React.ReactNode }>;
  CloseButton: React.FC<{ children?: React.ReactNode }>;
} = ({ children }) => {
  const [isOpen, setIsOpen] = useState(false);
  
  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  
  return (
    <ModalContext.Provider value={{ isOpen, open, close }}>
      {children}
    </ModalContext.Provider>
  );
};

// Триггер для открытия модального окна
Modal.Trigger = ({ children }) => {
  const { open } = useModal();
  
  return (
    <div onClick={open} style={{ cursor: 'pointer' }}>
      {children}
    </div>
  );
};

// Контент модального окна
Modal.Content = ({ children }) => {
  const { isOpen, close } = useModal();
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={close}>
      <div 
        className="modal-content" 
        onClick={(e) => e.stopPropagation()}
      >
        {children}
      </div>
    </div>
  );
};

// Заголовок модального окна
Modal.Header = ({ children }) => (
  <div className="modal-header">
    {children}
  </div>
);

// Тело модального окна
Modal.Body = ({ children }) => (
  <div className="modal-body">
    {children}
  </div>
);

// Футер модального окна
Modal.Footer = ({ children }) => (
  <div className="modal-footer">
    {children}
  </div>
);

// Кнопка закрытия
Modal.CloseButton = ({ children = '×' }) => {
  const { close } = useModal();
  
  return (
    <button 
      className="modal-close-button" 
      onClick={close}
      aria-label="Закрыть модальное окно"
    >
      {children}
    </button>
  );
};

// Использование Compound Component
const App = () => {
  return (
    <div>
      <Modal>
        <Modal.Trigger>
          <button>Открыть модальное окно</button>
        </Modal.Trigger>
        
        <Modal.Content>
          <Modal.Header>
            <h2>Заголовок модального окна</h2>
            <Modal.CloseButton />
          </Modal.Header>
          
          <Modal.Body>
            <p>Содержимое модального окна...</p>
            <form>
              <input type="text" placeholder="Введите текст" />
            </form>
          </Modal.Body>
          
          <Modal.Footer>
            <button>Сохранить</button>
            <Modal.CloseButton>
              <button>Отмена</button>
            </Modal.CloseButton>
          </Modal.Footer>
        </Modal.Content>
      </Modal>
    </div>
  );
};

export default Modal;
```

---

## 🎭 Render Props

### Теория
Render Props - паттерн для передачи логики между компонентами через prop, который является функцией.

### Примеры реализации

```tsx
// MouseTracker.tsx - Компонент с Render Props
import React, { useState, useEffect } from 'react';

interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  children: (mouse: MousePosition) => React.ReactNode;
  // Альтернативный способ через render prop
  render?: (mouse: MousePosition) => React.ReactNode;
}

const MouseTracker: React.FC<MouseTrackerProps> = ({ children, render }) => {
  const [mousePosition, setMousePosition] = useState<MousePosition>({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (event: MouseEvent) => {
      setMousePosition({
        x: event.clientX,
        y: event.clientY
      });
    };
    
    document.addEventListener('mousemove', handleMouseMove);
    
    return () => {
      document.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);
  
  // Поддерживаем оба способа: children и render prop
  return <>{render ? render(mousePosition) : children(mousePosition)}</>;
};

// DataFetcher.tsx - Универсальный компонент для загрузки данных
interface DataFetcherProps<T> {
  url: string;
  children: (data: {
    data: T | null;
    loading: boolean;
    error: string | null;
    refetch: () => void;
  }) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Произошла ошибка');
    } finally {
      setLoading(false);
    }
  };
  
  useEffect(() => {
    fetchData();
  }, [url]);
  
  return <>{children({ data, loading, error, refetch: fetchData })}</>;
}

// Использование Render Props
const App = () => {
  return (
    <div>
      {/* Отслеживание мыши */}
      <MouseTracker>
        {({ x, y }) => (
          <div>
            <h2>Позиция мыши:</h2>
            <p>X: {x}, Y: {y}</p>
            <div 
              style={{
                position: 'absolute',
                left: x - 10,
                top: y - 10,
                width: 20,
                height: 20,
                backgroundColor: 'red',
                borderRadius: '50%',
                pointerEvents: 'none'
              }}
            />
          </div>
        )}
      </MouseTracker>
      
      {/* Загрузка данных */}
      <DataFetcher<User[]> url="/api/users">
        {({ data, loading, error, refetch }) => {
          if (loading) return <div>Загрузка...</div>;
          if (error) return <div>Ошибка: {error} <button onClick={refetch}>Повторить</button></div>;
          if (!data) return <div>Нет данных</div>;
          
          return (
            <div>
              <h2>Пользователи:</h2>
              <button onClick={refetch}>Обновить</button>
              <ul>
                {data.map(user => (
                  <li key={user.id}>{user.name} - {user.email}</li>
                ))}
              </ul>
            </div>
          );
        }}
      </DataFetcher>
    </div>
  );
};

// FormField.tsx - Render Props для полей формы
interface FormFieldProps {
  name: string;
  initialValue?: string;
  validate?: (value: string) => string | null;
  children: (fieldProps: {
    value: string;
    onChange: (value: string) => void;
    error: string | null;
    isValid: boolean;
    reset: () => void;
  }) => React.ReactNode;
}

const FormField: React.FC<FormFieldProps> = ({ 
  name, 
  initialValue = '', 
  validate,
  children 
}) => {
  const [value, setValue] = useState(initialValue);
  const [error, setError] = useState<string | null>(null);
  const [touched, setTouched] = useState(false);
  
  const handleChange = (newValue: string) => {
    setValue(newValue);
    setTouched(true);
    
    if (validate) {
      const validationError = validate(newValue);
      setError(validationError);
    }
  };
  
  const reset = () => {
    setValue(initialValue);
    setError(null);
    setTouched(false);
  };
  
  const isValid = !error && touched;
  
  return (
    <>
      {children({
        value,
        onChange: handleChange,
        error: touched ? error : null,
        isValid,
        reset
      })}
    </>
  );
};

// Использование FormField
const LoginForm = () => {
  const validateEmail = (email: string) => {
    if (!email) return 'Email обязателен';
    if (!/\S+@\S+\.\S+/.test(email)) return 'Некорректный email';
    return null;
  };
  
  const validatePassword = (password: string) => {
    if (!password) return 'Пароль обязателен';
    if (password.length < 6) return 'Пароль должен быть не менее 6 символов';
    return null;
  };
  
  return (
    <form>
      <FormField name="email" validate={validateEmail}>
        {({ value, onChange, error, isValid }) => (
          <div className="form-field">
            <label>Email:</label>
            <input
              type="email"
              value={value}
              onChange={(e) => onChange(e.target.value)}
              className={error ? 'error' : isValid ? 'valid' : ''}
            />
            {error && <span className="error-message">{error}</span>}
          </div>
        )}
      </FormField>
      
      <FormField name="password" validate={validatePassword}>
        {({ value, onChange, error, isValid, reset }) => (
          <div className="form-field">
            <label>Пароль:</label>
            <input
              type="password"
              value={value}
              onChange={(e) => onChange(e.target.value)}
              className={error ? 'error' : isValid ? 'valid' : ''}
            />
            {error && <span className="error-message">{error}</span>}
            <button type="button" onClick={reset}>Сбросить</button>
          </div>
        )}
      </FormField>
    </form>
  );
};
```

---

## 🎯 Custom Hooks

### Теория
Custom Hooks позволяют извлекать и переиспользовать логику состояния между компонентами.

### Практические примеры

```tsx
// useApi.ts - Hook для работы с API
import { useState, useEffect, useCallback } from 'react';

interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
}

function useApi<T>(url: string, options?: RequestInit): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Произошла ошибка');
      setData(null);
    } finally {
      setLoading(false);
    }
  }, [url, options]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// useLocalStorage.ts - Hook для работы с localStorage
import { useState, useEffect } from 'react';

function useLocalStorage<T>(
  key: string, 
  initialValue: T
): [T, (value: T | ((val: T) => T)) => void] {
  // Получаем значение из localStorage или используем initial
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Ошибка чтения localStorage для ключа "${key}":`, error);
      return initialValue;
    }
  });
  
  // Обновляем localStorage при изменении значения
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Ошибка записи в localStorage для ключа "${key}":`, error);
    }
  };
  
  return [storedValue, setValue];
}

// useDebounce.ts - Hook для debounce
import { useState, useEffect } from 'react';

function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// useForm.ts - Hook для управления формами
import { useState, useCallback } from 'react';

interface UseFormOptions<T> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit: (values: T) => Promise<void> | void;
}

interface UseFormResult<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  isSubmitting: boolean;
  handleChange: (name: keyof T, value: any) => void;
  handleSubmit: (e: React.FormEvent) => Promise<void>;
  reset: () => void;
  setFieldValue: (name: keyof T, value: any) => void;
  setFieldError: (name: keyof T, error: string) => void;
}

function useForm<T extends Record<string, any>>({
  initialValues,
  validate,
  onSubmit
}: UseFormOptions<T>): UseFormResult<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = useCallback((name: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Очищаем ошибку для поля при изменении
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: undefined }));
    }
  }, [errors]);
  
  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Валидация
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      // Если есть ошибки, не отправляем форму
      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }
    
    try {
      setIsSubmitting(true);
      await onSubmit(values);
    } catch (error) {
      console.error('Ошибка отправки формы:', error);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate, onSubmit]);
  
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setIsSubmitting(false);
  }, [initialValues]);
  
  const setFieldValue = useCallback((name: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
  }, []);
  
  const setFieldError = useCallback((name: keyof T, error: string) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  }, []);
  
  return {
    values,
    errors,
    isSubmitting,
    handleChange,
    handleSubmit,
    reset,
    setFieldValue,
    setFieldError
  };
}

// Использование custom hooks
const UserProfile = () => {
  // Загрузка данных пользователя
  const { data: user, loading, error, refetch } = useApi<User>('/api/user/profile');
  
  // Настройки в localStorage
  const [settings, setSettings] = useLocalStorage('userSettings', {
    theme: 'light',
    notifications: true
  });
  
  // Поиск с debounce
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  // Форма редактирования профиля
  const form = useForm({
    initialValues: {
      name: user?.name || '',
      email: user?.email || '',
      bio: user?.bio || ''
    },
    validate: (values) => {
      const errors: any = {};
      
      if (!values.name) {
        errors.name = 'Имя обязательно';
      }
      
      if (!values.email) {
        errors.email = 'Email обязателен';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Некорректный email';
      }
      
      return errors;
    },
    onSubmit: async (values) => {
      await fetch('/api/user/profile', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values)
      });
      
      refetch(); // Обновляем данные
    }
  });
  
  // Эффект для поиска
  useEffect(() => {
    if (debouncedSearchTerm) {
      // Выполняем поиск только после debounce
      console.log('Поиск:', debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error}</div>;
  
  return (
    <div>
      <h1>Профиль пользователя</h1>
      
      {/* Настройки */}
      <div>
        <label>
          <input
            type="checkbox"
            checked={settings.notifications}
            onChange={(e) => setSettings({
              ...settings,
              notifications: e.target.checked
            })}
          />
          Уведомления
        </label>
      </div>
      
      {/* Поиск */}
      <div>
        <input
          type="text"
          placeholder="Поиск..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
      </div>
      
      {/* Форма редактирования */}
      <form onSubmit={form.handleSubmit}>
        <div>
          <label>Имя:</label>
          <input
            type="text"
            value={form.values.name}
            onChange={(e) => form.handleChange('name', e.target.value)}
          />
          {form.errors.name && <span>{form.errors.name}</span>}
        </div>
        
        <div>
          <label>Email:</label>
          <input
            type="email"
            value={form.values.email}
            onChange={(e) => form.handleChange('email', e.target.value)}
          />
          {form.errors.email && <span>{form.errors.email}</span>}
        </div>
        
        <div>
          <label>О себе:</label>
          <textarea
            value={form.values.bio}
            onChange={(e) => form.handleChange('bio', e.target.value)}
          />
        </div>
        
        <button type="submit" disabled={form.isSubmitting}>
          {form.isSubmitting ? 'Сохранение...' : 'Сохранить'}
        </button>
        
        <button type="button" onClick={form.reset}>
          Сбросить
        </button>
      </form>
    </div>
  );
};
```

---

## ⚡ Performance Patterns

### Мемоизация компонентов

```tsx
// OptimizedComponents.tsx
import React, { memo, useMemo, useCallback, useState } from 'react';

// Мемоизированный компонент товара
interface ProductProps {
  product: {
    id: number;
    name: string;
    price: number;
    description: string;
  };
  onAddToCart: (id: number) => void;
  isInCart: boolean;
}

const Product = memo<ProductProps>(({ product, onAddToCart, isInCart }) => {
  console.log(`Рендер продукта: ${product.name}`); // Для отладки
  
  // Мемоизируем обработчик клика
  const handleAddToCart = useCallback(() => {
    onAddToCart(product.id);
  }, [product.id, onAddToCart]);
  
  // Мемоизируем вычисления
  const formattedPrice = useMemo(() => {
    return new Intl.NumberFormat('ru-RU', {
      style: 'currency',
      currency: 'RUB'
    }).format(product.price);
  }, [product.price]);
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>{product.description}</p>
      <p className="price">{formattedPrice}</p>
      <button 
        onClick={handleAddToCart}
        disabled={isInCart}
      >
        {isInCart ? 'В корзине' : 'В корзину'}
      </button>
    </div>
  );
});

// Оптимизированный список продуктов
interface ProductListProps {
  products: Product[];
  cartItems: number[];
  onAddToCart: (id: number) => void;
  searchTerm: string;
}

const ProductList = memo<ProductListProps>(({ 
  products, 
  cartItems, 
  onAddToCart, 
  searchTerm 
}) => {
  // Мемоизируем фильтрацию продуктов
  const filteredProducts = useMemo(() => {
    if (!searchTerm) return products;
    
    return products.filter(product =>
      product.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      product.description.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [products, searchTerm]);
  
  // Мемоизируем Set для быстрой проверки наличия в корзине
  const cartItemsSet = useMemo(() => {
    return new Set(cartItems);
  }, [cartItems]);
  
  return (
    <div className="product-list">
      {filteredProducts.map(product => (
        <Product
          key={product.id}
          product={product}
          onAddToCart={onAddToCart}
          isInCart={cartItemsSet.has(product.id)}
        />
      ))}
    </div>
  );
});

// Виртуализация для больших списков
import { FixedSizeList as List } from 'react-window';

interface VirtualizedListProps {
  items: any[];
  itemHeight: number;
  height: number;
}

const VirtualizedList: React.FC<VirtualizedListProps> = ({ 
  items, 
  itemHeight, 
  height 
}) => {
  const Row = useCallback(({ index, style }) => {
    const item = items[index];
    
    return (
      <div style={style} className="list-item">
        <Product
          product={item}
          onAddToCart={() => {}}
          isInCart={false}
        />
      </div>
    );
  }, [items]);
  
  return (
    <List
      height={height}
      itemCount={items.length}
      itemSize={itemHeight}
      width="100%"
    >
      {Row}
    </List>
  );
};

// Lazy loading компонентов
const LazyProductDetails = React.lazy(() => import('./ProductDetails'));
const LazyShoppingCart = React.lazy(() => import('./ShoppingCart'));

const App = () => {
  const [showDetails, setShowDetails] = useState(false);
  const [showCart, setShowCart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowDetails(true)}>
        Показать детали
      </button>
      
      <button onClick={() => setShowCart(true)}>
        Показать корзину
      </button>
      
      <Suspense fallback={<div>Загрузка деталей...</div>}>
        {showDetails && <LazyProductDetails />}
      </Suspense>
      
      <Suspense fallback={<div>Загрузка корзины...</div>}>
        {showCart && <LazyShoppingCart />}
      </Suspense>
    </div>
  );
};
```

---

## 📚 Связанные разделы

- [[javascript-fundamentals|JavaScript Fundamentals]] - Основы JavaScript для React
- [[frontend-advanced|Frontend Advanced]] - Продвинутые техники React
- [[../technical-skills/testing|Testing]] - Тестирование React компонентов
- [[../fundamentals/design-patterns|Design Patterns]] - Общие паттерны проектирования
- [[web-apis-browser|Web APIs]] - Браузерные API в React приложениях 