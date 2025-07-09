# Продвинутый Frontend: React, Next.js, State Management

## React Hooks

### useState и useEffect
```tsx
// frontend/src/hooks/useUsers.ts
import { useState, useEffect } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

export const useUsers = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('/api/users');
        if (!response.ok) throw new Error('Failed to fetch users');
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  return { users, loading, error };
};
```

### useCallback и useMemo
```tsx
// frontend/src/components/UserList.tsx
import React, { useCallback, useMemo } from 'react';

interface Props {
  users: User[];
  onUserSelect: (user: User) => void;
  searchTerm: string;
}

export const UserList: React.FC<Props> = ({ users, onUserSelect, searchTerm }) => {
  // Мемоизация фильтрации
  const filteredUsers = useMemo(() => {
    return users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      user.email.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [users, searchTerm]);

  // Мемоизация callback
  const handleUserClick = useCallback((user: User) => {
    onUserSelect(user);
  }, [onUserSelect]);

  return (
    <div>
      {filteredUsers.map(user => (
        <div key={user.id} onClick={() => handleUserClick(user)}>
          {user.name} ({user.email})
        </div>
      ))}
    </div>
  );
};
```

### Custom Hooks
```tsx
// frontend/src/hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

export const useLocalStorage = <T>(key: string, initialValue: T) => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue] as const;
};
```

## Context API

### AuthContext
```tsx
// frontend/src/contexts/AuthContext.tsx
import React, { createContext, useContext, useReducer, useEffect } from 'react';

interface User {
  id: string;
  email: string;
  name: string;
}

interface AuthState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, loading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { ...state, user: action.payload, loading: false, error: null };
    case 'LOGIN_FAILURE':
      return { ...state, loading: false, error: action.payload };
    case 'LOGOUT':
      return { ...state, user: null, loading: false, error: null };
    default:
      return state;
  }
};

const AuthContext = createContext<{
  state: AuthState;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
} | null>(null);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    loading: false,
    error: null,
  });

  const login = async (email: string, password: string) => {
    dispatch({ type: 'LOGIN_START' });
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) throw new Error('Login failed');

      const { user, token } = await response.json();
      localStorage.setItem('token', token);
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: error.message });
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    dispatch({ type: 'LOGOUT' });
  };

  return (
    <AuthContext.Provider value={{ state, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

## Redux Toolkit

### Store Setup
```ts
// frontend/src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import authSlice from './slices/authSlice';
import usersSlice from './slices/usersSlice';

export const store = configureStore({
  reducer: {
    auth: authSlice,
    users: usersSlice,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST'],
      },
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Auth Slice
```ts
// frontend/src/store/slices/authSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

interface AuthState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

export const loginAsync = createAsyncThunk(
  'auth/login',
  async ({ email, password }: { email: string; password: string }) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) {
      throw new Error('Login failed');
    }

    return response.json();
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    loading: false,
    error: null,
  } as AuthState,
  reducers: {
    logout: (state) => {
      state.user = null;
      localStorage.removeItem('token');
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginAsync.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginAsync.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        localStorage.setItem('token', action.payload.token);
      })
      .addCase(loginAsync.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Login failed';
      });
  },
});

export const { logout, clearError } = authSlice.actions;
export default authSlice.reducer;
```

## Zustand (альтернатива Redux)

```ts
// frontend/src/store/useStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      (set, get) => ({
        user: null,
        loading: false,
        login: async (email: string, password: string) => {
          set({ loading: true });
          try {
            const response = await fetch('/api/auth/login', {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ email, password }),
            });

            if (!response.ok) throw new Error('Login failed');

            const { user, token } = await response.json();
            localStorage.setItem('token', token);
            set({ user, loading: false });
          } catch (error) {
            set({ loading: false });
            throw error;
          }
        },
        logout: () => {
          localStorage.removeItem('token');
          set({ user: null });
        },
      }),
      {
        name: 'auth-storage',
        partialize: (state) => ({ user: state.user }),
      }
    )
  )
);
```

## Next.js SSR/SSG

### Server-Side Rendering
```tsx
// pages/users/[id].tsx
import { GetServerSideProps } from 'next';
import { User } from '../../types';

interface Props {
  user: User;
}

export default function UserPage({ user }: Props) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

export const getServerSideProps: GetServerSideProps = async ({ params }) => {
  const { id } = params!;
  
  // Данные получаются на сервере при каждом запросе
  const response = await fetch(`${process.env.API_URL}/users/${id}`);
  const user = await response.json();

  return {
    props: {
      user,
    },
  };
};
```

### Static Site Generation
```tsx
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  id: string;
  title: string;
  content: string;
  slug: string;
}

interface Props {
  post: Post;
}

export default function BlogPost({ post }: Props) {
  return (
    <div>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </div>
  );
}

export const getStaticPaths: GetStaticPaths = async () => {
  const response = await fetch(`${process.env.API_URL}/posts`);
  const posts = await response.json();

  const paths = posts.map((post: Post) => ({
    params: { slug: post.slug },
  }));

  return {
    paths,
    fallback: 'blocking', // или false для 404
  };
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const { slug } = params!;
  
  const response = await fetch(`${process.env.API_URL}/posts/${slug}`);
  const post = await response.json();

  return {
    props: {
      post,
    },
    revalidate: 3600, // ISR - обновление каждый час
  };
};
```

### App Router (Next.js 13+)
```tsx
// app/users/[id]/page.tsx
interface Props {
  params: { id: string };
}

async function getUser(id: string) {
  const response = await fetch(`${process.env.API_URL}/users/${id}`, {
    cache: 'force-cache', // или 'no-store' для динамических данных
  });
  return response.json();
}

export default async function UserPage({ params }: Props) {
  const user = await getUser(params.id);

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## Оптимизация производительности

### Lazy Loading
```tsx
// frontend/src/App.tsx
import { lazy, Suspense } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Users = lazy(() => import('./pages/Users'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

export default function App() {
  return (
    <Router>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/users" element={<Users />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </Router>
  );
}
```

### React.memo и виртуализация
```tsx
// frontend/src/components/VirtualizedList.tsx
import React from 'react';
import { FixedSizeList as List } from 'react-window';

interface Item {
  id: string;
  name: string;
}

interface Props {
  items: Item[];
}

const Row = React.memo(({ index, style, data }: any) => (
  <div style={style}>
    <div>{data[index].name}</div>
  </div>
));

export const VirtualizedList: React.FC<Props> = ({ items }) => {
  return (
    <List
      height={400}
      itemCount={items.length}
      itemSize={50}
      itemData={items}
    >
      {Row}
    </List>
  );
};
```

## Material-UI

### Настройка темы
```tsx
// frontend/src/theme/index.ts
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
  typography: {
    fontFamily: 'Roboto, Arial, sans-serif',
    h1: {
      fontSize: '2rem',
      fontWeight: 600,
    },
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
        },
      },
    },
  },
});
```

### Кастомные компоненты
```tsx
// frontend/src/components/UserCard.tsx
import React from 'react';
import { Card, CardContent, Typography, Avatar, Box } from '@mui/material';
import { styled } from '@mui/material/styles';

const StyledCard = styled(Card)(({ theme }) => ({
  maxWidth: 345,
  margin: theme.spacing(2),
  '&:hover': {
    boxShadow: theme.shadows[8],
  },
}));

interface Props {
  user: {
    id: string;
    name: string;
    email: string;
    avatar?: string;
  };
}

export const UserCard: React.FC<Props> = ({ user }) => {
  return (
    <StyledCard>
      <CardContent>
        <Box display="flex" alignItems="center" mb={2}>
          <Avatar src={user.avatar} alt={user.name} />
          <Box ml={2}>
            <Typography variant="h6">{user.name}</Typography>
            <Typography variant="body2" color="text.secondary">
              {user.email}
            </Typography>
          </Box>
        </Box>
      </CardContent>
    </StyledCard>
  );
};
```

## Ant Design

### Настройка темы
```tsx
// frontend/src/App.tsx
import { ConfigProvider } from 'antd';
import { theme } from './theme';

const App = () => {
  return (
    <ConfigProvider
      theme={{
        token: {
          colorPrimary: '#1890ff',
          borderRadius: 8,
        },
        components: {
          Button: {
            colorPrimary: '#00b96b',
          },
        },
      }}
    >
      <MyApp />
    </ConfigProvider>
  );
};
```

### Форма с валидацией
```tsx
// frontend/src/components/UserForm.tsx
import React from 'react';
import { Form, Input, Button, message } from 'antd';

interface FormValues {
  name: string;
  email: string;
  password: string;
}

export const UserForm: React.FC = () => {
  const [form] = Form.useForm();

  const onFinish = async (values: FormValues) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values),
      });

      if (!response.ok) throw new Error('Failed to create user');
      
      message.success('User created successfully');
      form.resetFields();
    } catch (error) {
      message.error('Failed to create user');
    }
  };

  return (
    <Form form={form} onFinish={onFinish} layout="vertical">
      <Form.Item
        label="Name"
        name="name"
        rules={[{ required: true, message: 'Please input your name!' }]}
      >
        <Input />
      </Form.Item>

      <Form.Item
        label="Email"
        name="email"
        rules={[
          { required: true, message: 'Please input your email!' },
          { type: 'email', message: 'Please enter a valid email!' },
        ]}
      >
        <Input />
      </Form.Item>

      <Form.Item
        label="Password"
        name="password"
        rules={[{ required: true, min: 6, message: 'Password must be at least 6 characters!' }]}
      >
        <Input.Password />
      </Form.Item>

      <Form.Item>
        <Button type="primary" htmlType="submit">
          Create User
        </Button>
      </Form.Item>
    </Form>
  );
};
```

## Лучшие практики

### Performance
1. **Используйте React.memo** для дорогих компонентов
2. **Lazy loading** для code splitting
3. **useMemo/useCallback** для оптимизации
4. **Виртуализация** для больших списков

### State Management
1. **Выбирайте правильный инструмент**: Context API для простых случаев, Redux/Zustand для сложных
2. **Нормализуйте состояние** для сложных данных
3. **Используйте селекторы** для производительности

### SEO и доступность
1. **SSR/SSG** для SEO
2. **Semantic HTML** и ARIA атрибуты
3. **Keyboard navigation** поддержка
4. **Meta tags** и Open Graph

## Заключение

Современный React предоставляет мощные инструменты для создания производительных и масштабируемых приложений. Выбор технологий зависит от требований проекта.

## См. также
- [Тестирование](./testing.md)
- [CI/CD и DevOps](./cicd-devops.md)
- [Мониторинг](./deployment-monitoring.md) 