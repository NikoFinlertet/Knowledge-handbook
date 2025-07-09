# 🎭 React Testing

## 📋 Навигация
- [Testing Library Basics](#testing-library-basics)
- [Component Testing](#component-testing)
- [Hook Testing](#hook-testing)
- [Context Testing](#context-testing)
- [Best Practices](#best-practices)

---

## 🧪 Testing Library Basics

### Button Component

```jsx
// Button.jsx
import React from 'react';
import PropTypes from 'prop-types';

export const Button = ({ 
  children, 
  variant = 'primary', 
  disabled = false,
  loading = false,
  onClick,
  ...props 
}) => {
  const handleClick = (e) => {
    if (!disabled && !loading && onClick) {
      onClick(e);
    }
  };

  const className = `btn btn--${variant} ${disabled ? 'btn--disabled' : ''} ${loading ? 'btn--loading' : ''}`;

  return (
    <button
      className={className}
      onClick={handleClick}
      disabled={disabled || loading}
      aria-busy={loading}
      {...props}
    >
      {loading ? (
        <>
          <span className="btn__spinner" aria-hidden="true" />
          Loading...
        </>
      ) : (
        children
      )}
    </button>
  );
};

Button.propTypes = {
  children: PropTypes.node.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
  disabled: PropTypes.bool,
  loading: PropTypes.bool,
  onClick: PropTypes.func
};
```

---

## 🎯 Component Testing

### Button Tests

```jsx
// Button.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    
    const button = screen.getByRole('button', { name: /click me/i });
    expect(button).toBeInTheDocument();
  });

  test('applies correct CSS classes', () => {
    render(<Button variant="secondary" size="large">Test</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('btn', 'btn--secondary');
  });

  test('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('does not call onClick when disabled', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick} disabled>Click me</Button>);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(handleClick).not.toHaveBeenCalled();
    expect(button).toBeDisabled();
  });

  test('shows loading state', () => {
    render(<Button loading>Submit</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Loading...');
    expect(button).toHaveAttribute('aria-busy', 'true');
    expect(button).toBeDisabled();
  });

  test('matches snapshot', () => {
    const { container } = render(<Button variant="primary">Snapshot test</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

---

## 🪝 Hook Testing

### Custom Hook

```jsx
// useCounter.js
import { useState, useCallback } from 'react';

export const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  const setCountValue = useCallback((value) => {
    if (typeof value === 'number') {
      setCount(value);
    }
  }, []);

  return {
    count,
    increment,
    decrement,
    reset,
    setCount: setCountValue
  };
};
```

### Hook Tests

```jsx
// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter Hook', () => {
  test('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  test('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  test('should increment counter', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  test('should reset counter to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });

  test('should set custom value', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.setCount(25);
    });
    
    expect(result.current.count).toBe(25);
  });

  test('should ignore non-number values in setCount', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.setCount('invalid');
    });
    
    expect(result.current.count).toBe(5);
  });
});
```

---

## 🎨 Context Testing

### Theme Context

```jsx
// ThemeContext.jsx
import React, { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// ThemedButton.jsx
import React from 'react';
import { useTheme } from './ThemeContext';

export const ThemedButton = ({ children, ...props }) => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      className={`btn btn--${theme}`}
      onClick={toggleTheme}
      {...props}
    >
      {children} (Current theme: {theme})
    </button>
  );
};
```

### Context Tests

```jsx
// ThemedButton.test.jsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ThemeProvider } from './ThemeContext';
import { ThemedButton } from './ThemedButton';

const renderWithTheme = (component) => {
  return render(
    <ThemeProvider>
      {component}
    </ThemeProvider>
  );
};

describe('ThemedButton', () => {
  test('renders with light theme by default', () => {
    renderWithTheme(<ThemedButton>Toggle Theme</ThemedButton>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('btn--light');
    expect(button).toHaveTextContent('Toggle Theme (Current theme: light)');
  });

  test('toggles theme when clicked', async () => {
    const user = userEvent.setup();
    renderWithTheme(<ThemedButton>Toggle Theme</ThemedButton>);
    
    const button = screen.getByRole('button');
    
    await user.click(button);
    
    expect(button).toHaveClass('btn--dark');
    expect(button).toHaveTextContent('Toggle Theme (Current theme: dark)');
    
    await user.click(button);
    
    expect(button).toHaveClass('btn--light');
  });

  test('throws error when used outside ThemeProvider', () => {
    const consoleSpy = jest.spyOn(console, 'error').mockImplementation(() => {});
    
    expect(() => {
      render(<ThemedButton>Test</ThemedButton>);
    }).toThrow('useTheme must be used within ThemeProvider');
    
    consoleSpy.mockRestore();
  });
});
```

---

## 📊 Best Practices

### 🎯 Принципы React тестирования

1. **Тестируйте поведение, а не реализацию**
2. **Используйте semantic queries**
3. **Тестируйте пользовательские сценарии**
4. **Имитируйте реальное взаимодействие**

### 🛠️ Testing Utilities

```jsx
// testUtils.js
import React from 'react';
import { render } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { ThemeProvider } from './ThemeContext';

export const AllTheProviders = ({ children }) => {
  return (
    <BrowserRouter>
      <ThemeProvider>
        {children}
      </ThemeProvider>
    </BrowserRouter>
  );
};

export const renderWithProviders = (ui, options) => {
  return render(ui, { wrapper: AllTheProviders, ...options });
};

export const createMockUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  role: 'user',
  ...overrides
});
```

### 🔍 Queries Priority

```jsx
// ✅ Хорошо - доступные queries
test('should find elements accessibly', () => {
  render(<LoginForm />);
  
  // 1. getByRole - лучший выбор
  const submitButton = screen.getByRole('button', { name: /submit/i });
  
  // 2. getByLabelText - для форм
  const emailInput = screen.getByLabelText(/email/i);
  
  // 3. getByPlaceholderText
  const searchInput = screen.getByPlaceholderText(/search/i);
  
  // 4. getByText - для контента
  const heading = screen.getByText(/welcome/i);
});

// ❌ Плохо - queries по реализации
test('should avoid implementation details', () => {
  render(<LoginForm />);
  
  // Избегайте: getByTestId, getByClassName
  const button = screen.getByTestId('submit-button'); // только в крайнем случае
});
```

---

## 🔗 Связанные разделы

- [[unit-testing|Unit Testing]] - Модульное тестирование
- [[integration-testing|Integration Testing]] - Интеграционное тестирование
- [[testing-patterns|Testing Patterns]] - Паттерны тестирования
- [[e2e-testing|E2E Testing]] - End-to-End тестирование
- [[../react-patterns/component-architecture|Component Architecture]] - Архитектура компонентов 