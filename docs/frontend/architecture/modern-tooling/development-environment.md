# 🖥️ Development Environment - Настройка среды разработки

> **Development Environment** - оптимизация рабочего окружения разработчика для максимальной продуктивности, включая настройку редактора, переменных окружения и инструментов отладки.

## 📋 Обзор раздела

Правильно настроенная среда разработки критически важна для продуктивности. Современные IDE предоставляют мощные возможности для автодополнения, рефакторинга, отладки и автоматизации рутинных задач.

## 🎯 VS Code - Оптимальная настройка

### Comprehensive VS Code Configuration

```json
// .vscode/settings.json - Полная конфигурация проекта
{
  // =============================================================================
  // ОСНОВНЫЕ НАСТРОЙКИ РЕДАКТОРА
  // =============================================================================
  "editor.fontSize": 14,
  "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
  "editor.fontLigatures": true,
  "editor.lineHeight": 1.5,
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.wordWrap": "on",
  "editor.minimap.enabled": true,
  "editor.minimap.maxColumn": 120,
  "editor.rulers": [80, 120],
  "editor.renderWhitespace": "boundary",
  "editor.trimAutoWhitespace": true,
  "editor.detectIndentation": false,
  
  // =============================================================================
  // АВТОСОХРАНЕНИЕ И ФОРМАТИРОВАНИЕ
  // =============================================================================
  "files.autoSave": "onFocusChange",
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.trimFinalNewlines": true,
  
  // Форматирование
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": false,
  
  // =============================================================================
  // ESLINT И АВТОФИКСЫ
  // =============================================================================
  "eslint.enable": true,
  "eslint.validate": [
    "javascript",
    "javascriptreact", 
    "typescript",
    "typescriptreact"
  ],
  "eslint.workingDirectories": ["src"],
  "eslint.codeActionsOnSave.mode": "problems",
  
  // Автоматические действия при сохранении
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true,
    "source.removeUnusedImports": true,
    "source.sortImports": true
  },
  
  // =============================================================================
  // TYPESCRIPT НАСТРОЙКИ
  // =============================================================================
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.suggest.includeCompletionsForImportStatements": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "typescript.inlayHints.enumMemberValues.enabled": true,
  "typescript.inlayHints.functionLikeReturnTypes.enabled": true,
  "typescript.inlayHints.parameterNames.enabled": "literals",
  "typescript.inlayHints.parameterTypes.enabled": true,
  "typescript.inlayHints.propertyDeclarationTypes.enabled": true,
  "typescript.inlayHints.variableTypes.enabled": false,
  
  // =============================================================================
  // JAVASCRIPT/REACT НАСТРОЙКИ
  // =============================================================================
  "javascript.suggest.autoImports": true,
  "javascript.updateImportsOnFileMove.enabled": "always",
  "react.experimental.refreshOnLinkChange": true,
  
  // =============================================================================
  // EMMET КОНФИГУРАЦИЯ
  // =============================================================================
  "emmet.includeLanguages": {
    "javascript": "javascriptreact",
    "typescript": "typescriptreact"
  },
  "emmet.triggerExpansionOnTab": true,
  "emmet.showSuggestionsAsSnippets": true,
  
  // =============================================================================
  // ФАЙЛОВЫЕ АССОЦИАЦИИ
  // =============================================================================
  "files.associations": {
    "*.css": "postcss",
    "*.env*": "properties",
    "*.mdx": "markdown",
    ".babelrc": "json",
    ".eslintrc": "json",
    ".prettierrc": "json"
  },
  
  // =============================================================================
  // ПОИСК И ИСКЛЮЧЕНИЯ
  // =============================================================================
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/build": true,
    "**/.git": true,
    "**/coverage": true,
    "**/.next": true,
    "**/.nuxt": true,
    "**/out": true
  },
  
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "**/node_modules": true,
    "**/dist": true,
    "**/build": true
  },
  
  "files.watcherExclude": {
    "**/.git/objects/**": true,
    "**/.git/subtree-cache/**": true,
    "**/node_modules/**": true,
    "**/dist/**": true,
    "**/build/**": true
  },
  
  // =============================================================================
  // GIT ИНТЕГРАЦИЯ
  // =============================================================================
  "git.autofetch": true,
  "git.confirmSync": false,
  "git.enableSmartCommit": true,
  "git.suggestSmartCommit": false,
  "scm.diffDecorations": "gutter",
  "diffEditor.ignoreTrimWhitespace": false,
  
  // =============================================================================
  // ТЕРМИНАЛ
  // =============================================================================
  "terminal.integrated.fontSize": 13,
  "terminal.integrated.fontFamily": "'JetBrains Mono', monospace",
  "terminal.integrated.shell.osx": "/bin/zsh",
  "terminal.integrated.copyOnSelection": true,
  "terminal.integrated.cursorBlinking": true,
  
  // =============================================================================
  // ОТЛАДКА
  // =============================================================================
  "debug.inlineValues": "auto",
  "debug.showBreakpointsInOverviewRuler": true,
  "debug.console.fontSize": 13,
  
  // =============================================================================
  // СПЕЦИФИЧНЫЕ НАСТРОЙКИ ЯЗЫКОВ
  // =============================================================================
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.suggest.insertMode": "replace"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.suggest.insertMode": "replace"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.quickSuggestions": {
      "strings": true
    }
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[scss]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.wordWrap": "on",
    "editor.quickSuggestions": false
  }
}
```

### Рекомендуемые расширения

```json
// .vscode/extensions.json - Полный набор расширений
{
  "recommendations": [
    // =============================================================================
    // ОСНОВНЫЕ ИНСТРУМЕНТЫ РАЗРАБОТКИ
    // =============================================================================
    "esbenp.prettier-vscode",              // Форматирование кода
    "dbaeumer.vscode-eslint",              // ESLint интеграция
    "ms-vscode.vscode-typescript-next",    // Расширенный TypeScript
    "bradlc.vscode-tailwindcss",           // Tailwind CSS IntelliSense
    "ms-vscode.vscode-json",               // JSON поддержка
    
    // =============================================================================
    // REACT И FRONTEND
    // =============================================================================
    "formulahendry.auto-rename-tag",       // Автопереименование тегов
    "christian-kohler.npm-intellisense",   // npm IntelliSense
    "christian-kohler.path-intellisense",  // Автодополнение путей
    "jpoissonnier.vscode-styled-components", // Styled Components
    "ms-vscode.vscode-mdx",                // MDX поддержка
    "unifiedjs.vscode-mdx",                // Расширенная MDX поддержка
    
    // =============================================================================
    // CSS И СТИЛИЗАЦИЯ
    // =============================================================================
    "bradlc.vscode-tailwindcss",           // Tailwind CSS
    "csstools.postcss",                    // PostCSS поддержка
    "syler.sass-indented",                 // Sass/SCSS поддержка
    "pranaygp.vscode-css-peek",           // CSS Peek
    "zignd.html-css-class-completion",     // CSS классы автодополнение
    
    // =============================================================================
    // GIT И ВЕРСИОНИРОВАНИЕ
    // =============================================================================
    "eamodio.gitlens",                     // Расширенная Git интеграция
    "mhutchie.git-graph",                  // Git граф
    "donjayamanne.githistory",             // История Git
    "github.vscode-pull-request-github",   // GitHub Pull Requests
    
    // =============================================================================
    // УТИЛИТЫ И ПРОДУКТИВНОСТЬ
    // =============================================================================
    "gruntfuggly.todo-tree",              // TODO комментарии
    "streetsidesoftware.code-spell-checker", // Проверка орфографии
    "usernamehw.errorlens",               // Ошибки инлайн
    "ms-vscode.live-server",              // Live Server
    "ms-vsliveshare.vsliveshare",         // Live Share
    "wallabyjs.console-ninja",            // Расширенная отладка console
    
    // =============================================================================
    // ТЕСТИРОВАНИЕ
    // =============================================================================
    "orta.vscode-jest",                   // Jest поддержка
    "ms-playwright.playwright",           // Playwright тесты
    "hbenl.vscode-test-explorer",         // Test Explorer
    
    // =============================================================================
    // ДОКУМЕНТАЦИЯ И MARKDOWN
    // =============================================================================
    "yzhang.markdown-all-in-one",         // Markdown расширения
    "shd101wyy.markdown-preview-enhanced", // Улучшенный preview
    "bierner.markdown-mermaid",           // Mermaid диаграммы
    
    // =============================================================================
    // ДИЗАЙН И UX
    // =============================================================================
    "tomoki1207.pdf",                     // PDF просмотр
    "ms-vscode.hexeditor",                // Hex редактор
    "formulahendry.auto-close-tag",       // Автозакрытие тегов
    "redhat.vscode-yaml",                 // YAML поддержка
    
    // =============================================================================
    // DOCKER И DEVOPS
    // =============================================================================
    "ms-vscode-remote.remote-containers",  // Dev Containers
    "ms-azuretools.vscode-docker",        // Docker поддержка
    
    // =============================================================================
    // ТЕМЫ И ВНЕШНИЙ ВИД
    // =============================================================================
    "github.github-vscode-theme",         // GitHub темы
    "pkief.material-icon-theme",          // Material иконки
    "zhuangtongfa.material-theme",        // Material тема
    "dracula-theme.theme-dracula"         // Dracula тема
  ],
  
  "unwantedRecommendations": [
    "ms-vscode.vscode-typescript",        // Заменен на typescript-next
    "hookyqr.beautify"                    // Заменен на Prettier
  ]
}
```

## 🔧 Environment Variables Management

### Dotenv Configuration

```bash
# .env - Базовые переменные окружения
# =============================================================================
# APPLICATION SETTINGS
# =============================================================================
NODE_ENV=development
PORT=3000
HOST=localhost

# =============================================================================
# API CONFIGURATION
# =============================================================================
REACT_APP_API_URL=http://localhost:8000/api
REACT_APP_API_VERSION=v1
REACT_APP_API_TIMEOUT=10000

# =============================================================================
# AUTHENTICATION
# =============================================================================
REACT_APP_AUTH_DOMAIN=dev-auth.example.com
REACT_APP_AUTH_CLIENT_ID=your_auth_client_id
REACT_APP_JWT_SECRET=your_jwt_secret_key

# =============================================================================
# THIRD PARTY SERVICES
# =============================================================================
REACT_APP_GOOGLE_ANALYTICS_ID=GA-XXXXXXXXX
REACT_APP_STRIPE_PUBLIC_KEY=pk_test_xxxxxxxxx
REACT_APP_SENTRY_DSN=https://your-sentry-dsn
REACT_APP_FIREBASE_API_KEY=your_firebase_api_key
REACT_APP_FIREBASE_PROJECT_ID=your_firebase_project

# =============================================================================
# FEATURE FLAGS
# =============================================================================
REACT_APP_FEATURE_ANALYTICS=true
REACT_APP_FEATURE_DARK_MODE=true
REACT_APP_FEATURE_EXPERIMENTAL=false

# =============================================================================
# DEVELOPMENT TOOLS
# =============================================================================
GENERATE_SOURCEMAP=true
FAST_REFRESH=true
BROWSER=chrome
DANGEROUSLY_DISABLE_HOST_CHECK=false

# =============================================================================
# BUILD CONFIGURATION
# =============================================================================
BUILD_PATH=dist
PUBLIC_URL=/
INLINE_RUNTIME_CHUNK=false
IMAGE_INLINE_SIZE_LIMIT=8192
```

```bash
# .env.local - Локальные переопределения (не в git)
# Персональные настройки разработчика
REACT_APP_DEBUG=true
REACT_APP_MOCK_API=true
REACT_APP_LOG_LEVEL=debug

# Локальные ключи API
REACT_APP_OPENAI_API_KEY=your_local_openai_key
REACT_APP_LOCAL_DB_URL=postgresql://localhost:5432/myapp_dev
```

```bash
# .env.development - Среда разработки
NODE_ENV=development
REACT_APP_API_URL=http://localhost:8000/api
REACT_APP_ENABLE_REDUX_DEVTOOLS=true
REACT_APP_SHOW_PERFORMANCE_METRICS=true

# .env.staging - Staging среда
NODE_ENV=production
REACT_APP_API_URL=https://staging-api.example.com/api
REACT_APP_ENABLE_REDUX_DEVTOOLS=false
REACT_APP_SENTRY_ENVIRONMENT=staging

# .env.production - Продакшен
NODE_ENV=production
REACT_APP_API_URL=https://api.example.com/api
REACT_APP_ENABLE_REDUX_DEVTOOLS=false
REACT_APP_SENTRY_ENVIRONMENT=production
```

### Environment Variables Validation

```javascript
// src/config/env.js - Валидация переменных окружения
const requiredEnvVars = [
  'REACT_APP_API_URL',
  'REACT_APP_AUTH_DOMAIN'
];

const optionalEnvVars = [
  'REACT_APP_GOOGLE_ANALYTICS_ID',
  'REACT_APP_SENTRY_DSN'
];

// Проверка обязательных переменных
function validateRequiredEnvVars() {
  const missing = requiredEnvVars.filter(
    envVar => !process.env[envVar]
  );
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Типизированная конфигурация
export const config = {
  // API настройки
  api: {
    url: process.env.REACT_APP_API_URL,
    version: process.env.REACT_APP_API_VERSION || 'v1',
    timeout: parseInt(process.env.REACT_APP_API_TIMEOUT || '10000', 10)
  },
  
  // Аутентификация
  auth: {
    domain: process.env.REACT_APP_AUTH_DOMAIN,
    clientId: process.env.REACT_APP_AUTH_CLIENT_ID
  },
  
  // Feature flags
  features: {
    analytics: process.env.REACT_APP_FEATURE_ANALYTICS === 'true',
    darkMode: process.env.REACT_APP_FEATURE_DARK_MODE === 'true',
    experimental: process.env.REACT_APP_FEATURE_EXPERIMENTAL === 'true'
  },
  
  // Внешние сервисы
  services: {
    googleAnalytics: process.env.REACT_APP_GOOGLE_ANALYTICS_ID,
    sentry: process.env.REACT_APP_SENTRY_DSN,
    stripe: process.env.REACT_APP_STRIPE_PUBLIC_KEY
  },
  
  // Настройки разработки
  development: {
    debug: process.env.REACT_APP_DEBUG === 'true',
    mockApi: process.env.REACT_APP_MOCK_API === 'true',
    logLevel: process.env.REACT_APP_LOG_LEVEL || 'info'
  }
};

// Инициализация с валидацией
validateRequiredEnvVars();
```

## 🚀 Debug Configuration

### VS Code Debug Setup

```json
// .vscode/launch.json - Конфигурация отладки
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug React App",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["start"],
      "port": 3000,
      "protocol": "inspector",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "env": {
        "NODE_ENV": "development",
        "BROWSER": "none"
      }
    },
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": [
        "--runInBand",
        "--no-cache",
        "--no-coverage",
        "--watchAll=false"
      ],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "env": {
        "CI": "true"
      }
    },
    {
      "name": "Debug Current Jest Test",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": [
        "${relativeFile}",
        "--runInBand",
        "--no-cache",
        "--no-coverage",
        "--watchAll=false"
      ],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal"
    },
    {
      "name": "Attach to Chrome",
      "type": "chrome",
      "request": "attach",
      "port": 9222,
      "webRoot": "${workspaceFolder}/src",
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/*"
      }
    }
  ]
}
```

### Task Automation

```json
// .vscode/tasks.json - Автоматизация задач
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "npm: start",
      "type": "npm",
      "script": "start",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      },
      "problemMatcher": []
    },
    {
      "label": "npm: build",
      "type": "npm",
      "script": "build",
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "npm: test",
      "type": "npm",
      "script": "test",
      "group": "test",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Lint and Fix",
      "type": "shell",
      "command": "npm",
      "args": ["run", "lint:fix"],
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Type Check",
      "type": "shell",
      "command": "npm",
      "args": ["run", "type-check"],
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Clean Install",
      "type": "shell",
      "command": "rm",
      "args": ["-rf", "node_modules", "package-lock.json"],
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    }
  ]
}
```

## 🔧 Browser DevTools Setup

### React DevTools Configuration

```javascript
// src/utils/devtools.js - Настройка инструментов разработки
// Настройка React DevTools
if (process.env.NODE_ENV === 'development') {
  // Профилировщик React
  if (typeof window !== 'undefined') {
    window.__REACT_DEVTOOLS_GLOBAL_HOOK__ = 
      window.__REACT_DEVTOOLS_GLOBAL_HOOK__ || {};
    
    // Кастомные названия компонентов
    window.__REACT_DEVTOOLS_GLOBAL_HOOK__.settings = {
      hideConsoleLogsInStrictMode: false,
      breakOnConsoleErrors: false
    };
  }
  
  // Redux DevTools
  window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ = 
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || 
    require('redux').compose;
}

// Performance monitoring
export const measurePerformance = (name, fn) => {
  if (process.env.NODE_ENV === 'development') {
    performance.mark(`${name}-start`);
    const result = fn();
    performance.mark(`${name}-end`);
    performance.measure(name, `${name}-start`, `${name}-end`);
    
    const measure = performance.getEntriesByName(name)[0];
    console.log(`${name}: ${measure.duration.toFixed(2)}ms`);
    
    return result;
  }
  return fn();
};
```

### Chrome DevTools Snippets

```javascript
// Chrome DevTools Snippets для React отладки

// 1. Найти React компонент по DOM элементу
function findReactComponent(dom) {
  const key = Object.keys(dom).find(key => 
    key.startsWith('__reactInternalInstance') || 
    key.startsWith('__reactFiber')
  );
  
  if (key) {
    const fiberNode = dom[key];
    return fiberNode && fiberNode.return && fiberNode.return.stateNode;
  }
  return null;
}

// 2. Получить props любого компонента
function getComponentProps(selector) {
  const element = document.querySelector(selector);
  const component = findReactComponent(element);
  return component ? component.props : null;
}

// 3. Логгирование всех re-renders
function logReRenders() {
  const originalRender = React.Component.prototype.render;
  React.Component.prototype.render = function(...args) {
    console.log(`Re-render: ${this.constructor.name}`);
    return originalRender.apply(this, args);
  };
}

// 4. Анализ производительности рендеринга
function analyzeRenderPerformance() {
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if (entry.name.includes('React')) {
        console.log(`${entry.name}: ${entry.duration.toFixed(2)}ms`);
      }
    }
  });
  
  observer.observe({ entryTypes: ['measure'] });
}
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **VS Code Setup** - Настроить VS Code с полным набором расширений
2. **Environment Variables** - Создать систему переменных окружения
3. **Debug Configuration** - Настроить отладку React приложения

### 🟡 Средний уровень
4. **Custom Snippets** - Создать кастомные сниппеты для VS Code
5. **Workspace Settings** - Настроить workspace для команды
6. **Performance Monitoring** - Внедрить мониторинг производительности

### 🔴 Продвинутый уровень
7. **VS Code Extension** - Разработать собственное расширение
8. **Development Scripts** - Создать автоматизированные скрипты разработки
9. **Team Workspace** - Настроить полную среду для команды

## 🔗 Связанные разделы

- **[[linting-formatting|Linting & Formatting]]** - Автоматизация качества кода
- **[[bundlers|Bundlers]]** - Интеграция с системами сборки
- **[[frontend-cicd|Frontend CI/CD]]** - Настройка CI/CD окружения
- **[[../testing-frontend|Frontend Testing]]** - Тестирование и отладка

---

⏱️ **Время изучения**: 8-15 часов  
🎯 **Уровень**: Базовый - Средний  
📋 **Предварительные требования**: VS Code, Node.js основы 