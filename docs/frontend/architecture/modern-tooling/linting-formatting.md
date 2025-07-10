# 🔍 Linting & Formatting - Автоматизация качества кода

> **Linting & Formatting** - инструменты для автоматической проверки качества кода, обеспечения единообразного стиля и предотвращения ошибок на этапе разработки.

## 📋 Обзор раздела

Современная разработка требует автоматизации проверки качества кода. Линтеры находят потенциальные ошибки и проблемы, форматтеры обеспечивают единообразный стиль, а Git hooks автоматизируют эти процессы.

## 🔍 ESLint - Статический анализ кода

### Полная конфигурация ESLint

```javascript
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
    jest: true
  },
  
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'plugin:import/recommended',
    'plugin:import/typescript',
    'prettier' // Должен быть последним
  ],
  
  parser: '@typescript-eslint/parser',
  
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: './tsconfig.json'
  },
  
  plugins: [
    'react',
    'react-hooks',
    '@typescript-eslint',
    'jsx-a11y',
    'import',
    'testing-library'
  ],
  
  rules: {
    // TypeScript
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-empty-function': 'off',
    '@typescript-eslint/no-non-null-assertion': 'warn',
    '@typescript-eslint/prefer-optional-chain': 'error',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    
    // React
    'react/react-in-jsx-scope': 'off', // Не нужно в React 17+
    'react/prop-types': 'off', // Используем TypeScript
    'react/display-name': 'off',
    'react/no-unescaped-entities': 'off',
    'react/jsx-key': 'error',
    'react/jsx-no-duplicate-props': 'error',
    'react/jsx-no-target-blank': 'error',
    'react/jsx-curly-brace-presence': ['error', {
      props: 'never',
      children: 'never'
    }],
    
    // React Hooks
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    
    // Import/Export
    'import/order': [
      'error',
      {
        groups: [
          'builtin',
          'external',
          'internal',
          'parent',
          'sibling',
          'index'
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true
        }
      }
    ],
    'import/no-unresolved': 'error',
    'import/no-cycle': 'error',
    'import/no-self-import': 'error',
    'import/no-useless-path-segments': 'error',
    
    // Общие правила
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'prefer-const': 'error',
    'no-var': 'error',
    'no-unused-expressions': 'error',
    'no-duplicate-imports': 'error',
    'no-multiple-empty-lines': ['error', { max: 1 }],
    'eol-last': ['error', 'always'],
    'comma-dangle': ['error', 'es5'],
    
    // Безопасность
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-new-func': 'error',
    'no-script-url': 'error'
  },
  
  settings: {
    react: {
      version: 'detect'
    },
    'import/resolver': {
      typescript: {
        alwaysTryTypes: true,
        project: './tsconfig.json'
      }
    }
  },
  
  overrides: [
    // Правила для тестов
    {
      files: ['**/__tests__/**/*', '**/*.test.*', '**/*.spec.*'],
      extends: ['plugin:testing-library/react'],
      rules: {
        'testing-library/no-debugging-utils': 'warn',
        'testing-library/no-manual-cleanup': 'error',
        'testing-library/prefer-screen-queries': 'error',
        'testing-library/no-wait-for-multiple-assertions': 'error'
      }
    },
    
    // Правила для Storybook
    {
      files: ['*.stories.*'],
      rules: {
        'import/no-anonymous-default-export': 'off',
        'react-hooks/rules-of-hooks': 'off'
      }
    },
    
    // Правила для конфигурационных файлов
    {
      files: ['*.config.js', '*.config.ts'],
      env: {
        node: true
      },
      rules: {
        '@typescript-eslint/no-var-requires': 'off'
      }
    }
  ]
};
```

### ESLint для специфичных случаев

```javascript
// .eslintrc.performance.js - для производительности
module.exports = {
  extends: ['./.eslintrc.js'],
  
  plugins: ['react-perf'],
  
  rules: {
    // React performance
    'react-perf/jsx-no-new-object-as-prop': 'warn',
    'react-perf/jsx-no-new-array-as-prop': 'warn',
    'react-perf/jsx-no-new-function-as-prop': 'warn',
    'react-perf/jsx-no-jsx-as-prop': 'warn',
    
    // General performance
    'no-await-in-loop': 'error',
    'prefer-promise-reject-errors': 'error'
  }
};
```

## ✨ Prettier - Форматирование кода

### Конфигурация Prettier

```javascript
// .prettierrc.js
module.exports = {
  // Основные настройки
  printWidth: 100,
  tabWidth: 2,
  useTabs: false,
  semi: true,
  singleQuote: true,
  quoteProps: 'as-needed',
  
  // JSX настройки
  jsxSingleQuote: true,
  jsxBracketSameLine: false,
  
  // Trailing commas
  trailingComma: 'es5',
  
  // Пробелы
  bracketSpacing: true,
  arrowParens: 'avoid',
  
  // Переносы строк
  endOfLine: 'lf',
  
  // Встроенная поддержка
  embeddedLanguageFormatting: 'auto',
  
  // Переопределения для конкретных файлов
  overrides: [
    {
      files: '*.json',
      options: {
        printWidth: 200,
        tabWidth: 2
      }
    },
    {
      files: '*.md',
      options: {
        proseWrap: 'always',
        printWidth: 80
      }
    },
    {
      files: '*.yml',
      options: {
        tabWidth: 2,
        singleQuote: false
      }
    },
    {
      files: '*.css',
      options: {
        singleQuote: false
      }
    }
  ]
};
```

```bash
# .prettierignore
# Сгенерированные файлы
dist/
build/
coverage/
.next/
.nuxt/

# Зависимости
node_modules/
vendor/

# Конфигурация
package-lock.json
yarn.lock
pnpm-lock.yaml

# Файлы сборки
*.min.js
*.bundle.js
*.chunk.js

# Документация
CHANGELOG.md
*.generated.*

# IDE
.vscode/
.idea/
```

## 🪝 Husky & Lint-staged - Git Hooks автоматизация

### Конфигурация Husky

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,css,scss,md}": [
      "prettier --write"
    ],
    "*.{js,jsx,ts,tsx,json,css,scss,md}": [
      "git add"
    ]
  },
  
  "scripts": {
    "prepare": "husky install",
    "pre-commit": "lint-staged",
    "commit-msg": "commitlint --edit $1"
  }
}
```

```bash
#!/bin/sh
# .husky/pre-commit
. "$(dirname "$0")/_/husky.sh"

# Запуск lint-staged
npx lint-staged

# Проверка типов TypeScript
npm run type-check

# Запуск тестов для измененных файлов
npm run test:staged
```

```bash
#!/bin/sh
# .husky/pre-push
. "$(dirname "$0")/_/husky.sh"

# Полная проверка перед push
npm run lint
npm run type-check
npm run test:ci
npm run build
```

### CommitLint Configuration

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'build',    // Изменения сборки
        'chore',    // Рутинные задачи
        'ci',       // CI изменения
        'docs',     // Документация
        'feat',     // Новая функциональность
        'fix',      // Исправления
        'perf',     // Оптимизация производительности
        'refactor', // Рефакторинг
        'revert',   // Отмена коммита
        'style',    // Форматирование
        'test'      // Тесты
      ]
    ],
    'subject-max-length': [2, 'always', 100],
    'subject-case': [
      2, 
      'never', 
      ['sentence-case', 'start-case', 'pascal-case', 'upper-case']
    ],
    'body-leading-blank': [2, 'always'],
    'footer-leading-blank': [2, 'always']
  }
};
```

## 🔧 Интеграция с VS Code

### VS Code Settings

```json
// .vscode/settings.json
{
  // ESLint
  "eslint.enable": true,
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "eslint.workingDirectories": ["src"],
  "eslint.codeActionsOnSave.mode": "problems",
  
  // Prettier
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": true,
  
  // Автоматические действия при сохранении
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true,
    "source.removeUnusedImports": true
  },
  
  // TypeScript
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  
  // Файловые ассоциации
  "files.associations": {
    "*.css": "postcss"
  },
  
  // Исключения из поиска
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/build": true,
    "**/.git": true,
    "**/coverage": true
  }
}
```

### Рекомендуемые расширения

```json
// .vscode/extensions.json
{
  "recommendations": [
    // Основные инструменты
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "ms-vscode.vscode-typescript-next",
    
    // React/JavaScript
    "formulahendry.auto-rename-tag",
    "christian-kohler.npm-intellisense",
    "christian-kohler.path-intellisense",
    
    // CSS/Styling
    "bradlc.vscode-tailwindcss",
    "jpoissonnier.vscode-styled-components",
    
    // Утилиты
    "gruntfuggly.todo-tree",
    "streetsidesoftware.code-spell-checker",
    "ms-vscode.vscode-json",
    "redhat.vscode-yaml"
  ]
}
```

## 🎯 Продвинутые практики

### Кастомные ESLint правила

```javascript
// eslint-rules/no-hardcoded-colors.js
module.exports = {
  meta: {
    type: 'problem',
    docs: {
      description: 'Disallow hardcoded color values',
      category: 'Best Practices'
    },
    schema: []
  },
  
  create(context) {
    const colorRegex = /#[0-9a-fA-F]{3,6}|rgb\(|rgba\(|hsl\(|hsla\(/;
    
    return {
      Literal(node) {
        if (typeof node.value === 'string' && colorRegex.test(node.value)) {
          context.report({
            node,
            message: 'Avoid hardcoded colors. Use design tokens instead.'
          });
        }
      }
    };
  }
};
```

### Conditional Linting

```javascript
// .eslintrc.conditional.js
const isProduction = process.env.NODE_ENV === 'production';
const isCI = process.env.CI === 'true';

module.exports = {
  extends: ['./.eslintrc.js'],
  
  rules: {
    // Строже в продакшене
    'no-console': isProduction ? 'error' : 'warn',
    'no-debugger': isProduction ? 'error' : 'warn',
    'no-unused-vars': isProduction ? 'error' : 'warn',
    
    // Дополнительные правила в CI
    ...(isCI && {
      'prefer-const': 'error',
      'no-var': 'error',
      '@typescript-eslint/no-explicit-any': 'error'
    })
  }
};
```

### Staged Testing

```javascript
// scripts/test-staged.js
const { execSync } = require('child_process');

function getChangedFiles() {
  const changedFiles = execSync('git diff --cached --name-only --diff-filter=ACM')
    .toString()
    .trim()
    .split('\n')
    .filter(file => file.match(/\.(js|jsx|ts|tsx)$/));
    
  return changedFiles;
}

function runTestsForChangedFiles() {
  const changedFiles = getChangedFiles();
  
  if (changedFiles.length === 0) {
    console.log('No relevant files changed');
    return;
  }
  
  // Находим связанные тестовые файлы
  const testFiles = changedFiles
    .map(file => file.replace(/\.(js|jsx|ts|tsx)$/, '.test.$1'))
    .filter(file => {
      try {
        require.resolve(`./${file}`);
        return true;
      } catch {
        return false;
      }
    });
    
  if (testFiles.length > 0) {
    execSync(`jest ${testFiles.join(' ')}`, { stdio: 'inherit' });
  }
}

runTestsForChangedFiles();
```

## 📊 Качественные метрики

### Настройка отчетности

```javascript
// .eslintrc.reporting.js
module.exports = {
  extends: ['./.eslintrc.js'],
  
  // Форматтеры для различных нужд
  // npm run lint -- --format=json > eslint-report.json
  // npm run lint -- --format=html > eslint-report.html
  // npm run lint -- --format=junit > eslint-junit.xml
};
```

### SonarQube интеграция

```yaml
# sonar-project.properties
sonar.projectKey=my-frontend-project
sonar.sources=src
sonar.tests=src
sonar.test.inclusions=**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx
sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.eslint.reportPaths=eslint-report.json
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **ESLint Setup** - Настроить ESLint для React TypeScript проекта
2. **Prettier Integration** - Интегрировать Prettier с ESLint
3. **Basic Git Hooks** - Настроить pre-commit hook с lint-staged

### 🟡 Средний уровень
4. **Custom Rules** - Создать кастомное ESLint правило
5. **Monorepo Linting** - Настроить линтинг для монорепо
6. **Performance Linting** - Добавить правила для оптимизации производительности

### 🔴 Продвинутый уровень
7. **ESLint Plugin** - Разработать собственный ESLint плагин
8. **Advanced Automation** - Настроить сложную автоматизацию с условной логикой
9. **Team Standards** - Создать корпоративный конфиг для команды

## 🔗 Связанные разделы

- **[[development-environment|Development Environment]]** - Настройка VS Code и IDE
- **[[frontend-cicd|Frontend CI/CD]]** - Автоматизация в CI/CD пайплайнах  
- **[[bundlers|Bundlers]]** - Интеграция с системами сборки
- **[[../testing-frontend|Frontend Testing]]** - Линтинг тестового кода

---

⏱️ **Время изучения**: 10-18 часов  
🎯 **Уровень**: Базовый - Продвинутый  
📋 **Предварительные требования**: JavaScript, Git basics 