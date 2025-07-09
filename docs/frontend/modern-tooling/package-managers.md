# 📦 Package Managers - Управление зависимостями

> **Package Managers** - инструменты для управления зависимостями JavaScript проектов, включая установку, обновление и версионирование пакетов.

## 📋 Обзор раздела

Package Managers автоматизируют управление зависимостями в JavaScript проектах. Современные менеджеры пакетов обеспечивают детерминированные установки, оптимизацию дискового пространства и безопасность.

## 📄 NPM - Стандартный менеджер

### NPM Scripts Configuration

```json
{
  "name": "my-frontend-app",
  "version": "1.0.0",
  "scripts": {
    // Разработка
    "dev": "vite",
    "start": "vite --host",
    "preview": "vite preview",
    
    // Сборка
    "build": "vite build",
    "build:analyze": "vite build && npx webpack-bundle-analyzer dist/static/js/*.js",
    "build:prod": "NODE_ENV=production vite build",
    
    // Тестирование
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:e2e": "cypress run",
    "test:e2e:open": "cypress open",
    "test:ci": "jest --ci --coverage --watchAll=false",
    
    // Качество кода
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint src --ext .js,.jsx,.ts,.tsx --fix",
    "lint:staged": "lint-staged",
    "format": "prettier --write \"src/**/*.{js,jsx,ts,tsx,json,css,scss,md}\"",
    "format:check": "prettier --check \"src/**/*.{js,jsx,ts,tsx,json,css,scss,md}\"",
    "type-check": "tsc --noEmit",
    
    // Git hooks
    "pre-commit": "lint-staged",
    "prepare": "husky install",
    
    // Инструменты разработки
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook",
    "clean": "rimraf dist node_modules/.cache",
    
    // Анализ и деплой
    "analyze": "npx webpack-bundle-analyzer dist/static/js/*.js",
    "serve": "serve -s build",
    "deploy": "npm run build && npm run deploy:surge",
    "deploy:surge": "surge --project ./dist --domain my-app.surge.sh",
    "deploy:netlify": "netlify deploy --prod --dir=dist",
    
    // Обслуживание
    "deps:check": "npm outdated",
    "deps:update": "npm update",
    "deps:audit": "npm audit",
    "deps:fix": "npm audit fix"
  },
  
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.8.1"
  },
  
  "devDependencies": {
    "@types/react": "^18.0.27",
    "@types/react-dom": "^18.0.10",
    "@vitejs/plugin-react": "^3.1.0",
    "vite": "^4.1.0"
  },
  
  "engines": {
    "node": ">=16.0.0",
    "npm": ">=8.0.0"
  },
  
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead",
    "not ie 11"
  ]
}
```

### NPM Configuration

```bash
# .npmrc - конфигурация npm
# Реестр пакетов
registry=https://registry.npmjs.org/

# Кеширование
cache=/Users/username/.npm
cache-min=3600

# Производительность
progress=true
audit-level=moderate
fund=false

# Безопасность
audit=true
audit-level=moderate

# Приватные пакеты
@mycompany:registry=https://npm.mycompany.com/
//npm.mycompany.com/:_authToken=${NPM_TOKEN}

# Настройки сборки
scripts-prepend-node-path=true
```

## 🧶 Yarn - Быстрый и надежный

### Yarn Workspaces (Monorepo)

```json
// package.json - корневой файл монорепо
{
  "name": "my-monorepo",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ],
  
  "scripts": {
    // Команды для всех workspace
    "build": "yarn workspaces foreach run build",
    "test": "yarn workspaces foreach run test",
    "lint": "yarn workspaces foreach run lint",
    "clean": "yarn workspaces foreach run clean",
    
    // Выборочные команды
    "build:ui": "yarn workspace @company/ui build",
    "test:web": "yarn workspace @company/web test",
    "dev:all": "yarn workspaces foreach --parallel run dev",
    
    // Управление зависимостями
    "add:ui": "yarn workspace @company/ui add",
    "remove:web": "yarn workspace @company/web remove"
  },
  
  "devDependencies": {
    "lerna": "^6.4.1",
    "typescript": "^4.9.5"
  },
  
  "packageManager": "yarn@3.4.1"
}

// packages/ui/package.json
{
  "name": "@company/ui",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  
  "scripts": {
    "build": "rollup -c",
    "dev": "rollup -c --watch",
    "test": "jest",
    "lint": "eslint src"
  },
  
  "dependencies": {
    "react": "^18.2.0"
  },
  
  "peerDependencies": {
    "react": ">=16.8.0"
  },
  
  "files": [
    "dist"
  ]
}

// apps/web/package.json
{
  "name": "@company/web",
  "version": "1.0.0",
  "private": true,
  
  "dependencies": {
    "@company/ui": "workspace:*",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "jest"
  }
}
```

### Yarn Configuration

```yaml
# .yarnrc.yml - Yarn 3+ конфигурация
compressionLevel: mixed
enableGlobalCache: false
enableTelemetry: false

# PnP (Plug'n'Play) настройки
nodeLinker: pnp
pnpEnableInlining: false
pnpFallbackMode: none

# Workspace настройки
enableImmutableInstalls: true
enableInlineBuilds: false

# Настройки производительности
httpTimeout: 60000
networkConcurrency: 16

# Plugins
plugins:
  - path: .yarn/plugins/@yarnpkg/plugin-interactive-tools.cjs
    spec: "@yarnpkg/plugin-interactive-tools"
  - path: .yarn/plugins/@yarnpkg/plugin-workspace-tools.cjs
    spec: "@yarnpkg/plugin-workspace-tools"

# Версии Node.js
packageExtensions:
  react@*:
    peerDependencies:
      react-dom: "*"
```

## ⚡ PNPM - Эффективный и быстрый

### PNPM Configuration

```yaml
# .pnpmrc - конфигурация PNPM
# Хранилище пакетов
store-dir=~/.pnpm-store
shared-workspace-lockfile=true
link-workspace-packages=true
prefer-workspace-packages=true

# Настройки кеширования
cache-dir=~/.pnpm-cache
state-dir=~/.pnpm-state

# Строгий режим
strict-peer-dependencies=true
auto-install-peers=true
resolve-peers-from-workspace-root=true

# Производительность
network-concurrency=16
child-concurrency=5
fetch-retries=2
fetch-retry-factor=10
fetch-retry-mintimeout=10000
fetch-retry-maxtimeout=60000

# Hoisting настройки
hoist=true
hoist-pattern[]=*eslint*
hoist-pattern[]=*prettier*
shamefully-hoist=false

# Настройки реестра
registry=https://registry.npmjs.org/
@mycompany:registry=https://npm.mycompany.com/

# Игнорирование
ignore-pnpmfile=false
ignore-scripts=false
```

### PNPM Workspace

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
  - 'tools/*'
  - '!**/test/**'
```

```json
// package.json - корневой для PNPM workspace
{
  "name": "monorepo",
  "private": true,
  
  "scripts": {
    "build": "pnpm -r run build",
    "test": "pnpm -r run test",
    "lint": "pnpm -r run lint",
    "dev": "pnpm -r --parallel run dev",
    
    // Фильтрация по workspace
    "build:ui": "pnpm --filter @company/ui build",
    "test:changed": "pnpm --filter '[HEAD^]' test",
    "build:deps": "pnpm --filter ...@company/web build"
  },
  
  "devDependencies": {
    "typescript": "workspace:*",
    "@types/node": "^18.14.2"
  }
}
```

## 🔄 Сравнение Package Managers

### Производительность и возможности

| Критерий | NPM | Yarn | PNPM |
|----------|-----|------|------|
| **Скорость установки** | 🟡 Средняя | 🟢 Быстрая | 🟢 Очень быстрая |
| **Дисковое пространство** | 🔴 Дублирование | 🟡 Оптимизация | 🟢 Экономия 50-70% |
| **Детерминизм** | 🟡 package-lock | 🟢 yarn.lock | 🟢 pnpm-lock |
| **Workspaces** | 🟡 Базовые | 🟢 Продвинутые | 🟢 Продвинутые |
| **PnP поддержка** | ❌ Нет | 🟢 Да | ❌ Нет |
| **Безопасность** | 🟡 Аудит | 🟢 Расширенный | 🟢 Строгий |

### Рекомендации по выбору

**NPM - выбирайте если:**
- Простые проекты
- Стандартная среда
- Минимальная настройка
- CI/CD совместимость

**Yarn - выбирайте если:**
- Монорепо структура
- Нужен PnP режим
- Команда любит инновации
- Сложные workspace

**PNPM - выбирайте если:**
- Экономия места критична
- Много проектов
- Быстрота важна
- Строгость зависимостей

## 🛠️ Лучшие практики

### Управление зависимостями

```bash
# Проверка устаревших пакетов
npm outdated
yarn outdated
pnpm outdated

# Обновление зависимостей
npm update
yarn upgrade
pnpm update

# Интерактивное обновление
npx npm-check-updates -i
yarn upgrade-interactive
pnpm update -i

# Аудит безопасности
npm audit
yarn audit
pnpm audit

# Исправление уязвимостей
npm audit fix
yarn audit fix
pnpm audit fix
```

### Lock файлы

```bash
# Создание lock файла
npm install # package-lock.json
yarn install # yarn.lock
pnpm install # pnpm-lock.yaml

# Установка точно по lock файлу
npm ci
yarn install --frozen-lockfile
pnpm install --frozen-lockfile
```

### Кеширование в CI

```yaml
# GitHub Actions - npm
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# GitHub Actions - yarn
- name: Get yarn cache directory path
  id: yarn-cache-dir-path
  run: echo "dir=$(yarn cache dir)" >> $GITHUB_OUTPUT
- uses: actions/cache@v3
  with:
    path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
    key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}

# GitHub Actions - pnpm
- name: Setup pnpm
  uses: pnpm/action-setup@v2
  with:
    version: 7
- name: Cache pnpm modules
  uses: actions/cache@v3
  with:
    path: ~/.pnpm-store
    key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **NPM Scripts** - Настроить полный набор скриптов для проекта
2. **Lock Files** - Понять различия между lock файлами
3. **Dependencies Types** - Правильно разделить deps/devDeps/peerDeps

### 🟡 Средний уровень
4. **Yarn Workspaces** - Создать монорепо с shared библиотеками
5. **PNPM Setup** - Мигрировать проект на PNPM
6. **Private Registry** - Настроить приватный npm registry

### 🔴 Продвинутый уровень
7. **Custom Registry** - Развернуть собственный npm registry
8. **Monorepo CI/CD** - Настроить CI для монорепо с affected commands
9. **Package Publishing** - Автоматизировать публикацию пакетов

## 🔗 Связанные разделы

- **[[bundlers|Bundlers]]** - Интеграция с системами сборки
- **[[frontend-cicd|Frontend CI/CD]]** - Автоматизация package management
- **[[development-environment|Development Environment]]** - Настройка среды разработки
- **[[../../infrastructure/docker|Docker]]** - Контейнеризация с package managers

---

⏱️ **Время изучения**: 8-15 часов  
🎯 **Уровень**: Базовый - Средний  
📋 **Предварительные требования**: Node.js, командная строка 