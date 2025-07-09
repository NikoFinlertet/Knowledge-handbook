# 🎯 Bundlers - Современные сборщики

> **Bundlers** - инструменты для объединения и оптимизации JavaScript модулей, стилей и ресурсов в готовые для продакшена пакеты.

## 📋 Обзор раздела

Bundlers решают проблему модульности в браузере, объединяя множество файлов в оптимизированные бандлы. Современные сборщики также обеспечивают hot reload, code splitting, оптимизацию изображений и множество других возможностей.

## ⚡ Webpack - Мощный и гибкий

### Базовая конфигурация Webpack

```javascript
// webpack.config.js - Базовая конфигурация Webpack
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');

const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  // Точка входа
  entry: {
    main: './src/index.js',
    vendor: './src/vendor.js' // Отдельный бандл для библиотек
  },

  // Выходные файлы
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isProduction 
      ? '[name].[contenthash].js'  // Хеширование для кеширования
      : '[name].js',
    clean: true, // Очищаем dist при каждой сборке
    publicPath: '/' // Базовый путь для ресурсов
  },

  // Режим разработки/продакшена
  mode: isProduction ? 'production' : 'development',
  
  // Source maps для отладки
  devtool: isProduction ? 'source-map' : 'eval-source-map',

  // Настройки dev сервера
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    port: 3000,
    hot: true, // Hot Module Replacement
    historyApiFallback: true, // SPA routing
    open: true // Автоматически открывать браузер
  },

  // Лоадеры для обработки файлов
  module: {
    rules: [
      // JavaScript/TypeScript
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                targets: 'defaults',
                useBuiltIns: 'usage',
                corejs: 3
              }],
              '@babel/preset-react',
              '@babel/preset-typescript'
            ],
            plugins: [
              '@babel/plugin-proposal-class-properties',
              '@babel/plugin-transform-runtime'
            ]
          }
        }
      },

      // CSS/SCSS
      {
        test: /\.(css|scss|sass)$/,
        use: [
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                auto: true, // Автоматические CSS модули для .module.css
                localIdentName: isProduction 
                  ? '[hash:base64:5]' 
                  : '[name]__[local]--[hash:base64:5]'
              },
              sourceMap: true
            }
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  'autoprefixer',
                  'postcss-preset-env'
                ]
              }
            }
          },
          'sass-loader'
        ]
      },

      // Изображения
      {
        test: /\.(png|jpg|jpeg|gif|svg)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024 // Инлайн файлы меньше 8KB
          }
        },
        generator: {
          filename: 'images/[name].[hash][ext]'
        }
      },

      // Шрифты
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash][ext]'
        }
      }
    ]
  },

  // Плагины
  plugins: [
    // Генерация HTML файла
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: isProduction
    }),

    // Извлечение CSS в отдельные файлы (только для продакшена)
    ...(isProduction ? [
      new MiniCssExtractPlugin({
        filename: 'css/[name].[contenthash].css'
      })
    ] : [])
  ],

  // Оптимизация
  optimization: {
    minimize: isProduction,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: isProduction // Удаляем console.log в продакшене
          }
        }
      }),
      new CssMinimizerPlugin()
    ],

    // Разделение кода (code splitting)
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Отдельный чанк для vendor библиотек
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10
        },
        // Общий код между чанками
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          priority: 5,
          reuseExistingChunk: true
        }
      }
    },

    // Runtime chunk для лучшего кеширования
    runtimeChunk: 'single'
  },

  // Разрешение модулей
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@assets': path.resolve(__dirname, 'src/assets')
    }
  }
};
```

## 🚀 Vite - Быстрый и современный

### Конфигурация Vite

```javascript
// vite.config.js - Современная альтернатива Webpack
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig(({ command, mode }) => {
  const isProduction = mode === 'production';

  return {
    // Плагины
    plugins: [
      react({
        // Поддержка React Fast Refresh
        fastRefresh: true,
        
        // Babel конфигурация
        babel: {
          plugins: [
            // Эмоциональные стили
            ['@emotion/babel-plugin', { autoLabel: 'dev-only' }]
          ]
        }
      })
    ],

    // Настройки сервера разработки
    server: {
      port: 3000,
      host: true,
      open: true,
      
      // Прокси для API
      proxy: {
        '/api': {
          target: 'http://localhost:8000',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        }
      }
    },

    // Настройки сборки
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      sourcemap: isProduction ? 'hidden' : true,
      
      // Минификация
      minify: isProduction ? 'terser' : false,
      terserOptions: {
        compress: {
          drop_console: isProduction,
          drop_debugger: isProduction
        }
      },

      // Rollup опции (Vite использует Rollup под капотом)
      rollupOptions: {
        input: {
          main: resolve(__dirname, 'index.html'),
          admin: resolve(__dirname, 'admin.html') // Множественные точки входа
        },
        
        output: {
          // Разделение чанков
          manualChunks: {
            vendor: ['react', 'react-dom'],
            router: ['react-router-dom'],
            ui: ['@material-ui/core', '@material-ui/icons']
          }
        }
      },

      // Размер предупреждений
      chunkSizeWarningLimit: 1000
    },

    // Разрешение модулей
    resolve: {
      alias: {
        '@': resolve(__dirname, 'src'),
        '@components': resolve(__dirname, 'src/components'),
        '@hooks': resolve(__dirname, 'src/hooks'),
        '@utils': resolve(__dirname, 'src/utils'),
        '@assets': resolve(__dirname, 'src/assets'),
        '@styles': resolve(__dirname, 'src/styles')
      }
    },

    // CSS настройки
    css: {
      modules: {
        localsConvention: 'camelCase',
        generateScopedName: isProduction 
          ? '[hash:base64:5]'
          : '[name]__[local]--[hash:base64:5]'
      },
      
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@styles/variables.scss";`
        }
      }
    },

    // Переменные окружения
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
      __BUILD_TIME__: JSON.stringify(new Date().toISOString())
    },

    // Оптимизация зависимостей
    optimizeDeps: {
      include: ['react', 'react-dom'],
      exclude: ['some-es6-module']
    }
  };
});
```

## 📦 Rollup - Для библиотек

### Конфигурация Rollup

```javascript
// rollup.config.js - Для библиотек
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from '@rollup/plugin-typescript';
import { terser } from 'rollup-plugin-terser';
import peerDepsExternal from 'rollup-plugin-peer-deps-external';
import postcss from 'rollup-plugin-postcss';

const packageJson = require('./package.json');

export default {
  input: 'src/index.ts',
  
  output: [
    // CommonJS для Node.js
    {
      file: packageJson.main,
      format: 'cjs',
      sourcemap: true
    },
    
    // ES Modules для современных bundlers
    {
      file: packageJson.module,
      format: 'esm',
      sourcemap: true
    },
    
    // UMD для браузеров
    {
      file: packageJson.browser,
      format: 'umd',
      name: 'MyLibrary',
      sourcemap: true,
      globals: {
        react: 'React',
        'react-dom': 'ReactDOM'
      }
    }
  ],

  plugins: [
    // Исключаем peer dependencies
    peerDepsExternal(),
    
    // Разрешение модулей
    resolve({
      browser: true
    }),
    
    // CommonJS поддержка
    commonjs(),
    
    // TypeScript
    typescript({
      tsconfig: './tsconfig.json',
      declaration: true,
      declarationDir: 'dist/types'
    }),
    
    // CSS обработка
    postcss({
      extract: true,
      minimize: true
    }),
    
    // Минификация для production
    terser()
  ],

  // Внешние зависимости
  external: ['react', 'react-dom']
};
```

## 🔧 Сравнение Bundlers

### Выбор инструмента

| Критерий | Webpack | Vite | Rollup |
|----------|---------|------|--------|
| **Скорость dev** | 🟡 Средняя | 🟢 Очень быстрая | 🟡 Средняя |
| **Экосистема** | 🟢 Огромная | 🟡 Растущая | 🟡 Специализированная |
| **Конфигурация** | 🔴 Сложная | 🟢 Простая | 🟢 Простая |
| **Bundle размер** | 🟡 Хороший | 🟢 Отличный | 🟢 Отличный |
| **Поддержка legacy** | 🟢 Отличная | 🟡 Ограниченная | 🟡 Требует настройки |
| **Библиотеки** | 🟡 Возможно | 🟡 Возможно | 🟢 Идеально |

### Рекомендации по выбору

**Используйте Webpack если:**
- Большой legacy проект
- Нужна максимальная гибкость
- Сложные требования к сборке
- Много кастомных лоадеров

**Используйте Vite если:**
- Новый проект
- Нужна скорость разработки
- Modern JS (ES6+)
- React/Vue приложение

**Используйте Rollup если:**
- Создаете библиотеку
- Нужен минимальный bundle
- ES модули приоритет
- Простая структура проекта

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **Настройка Webpack** - Создать базовую конфигурацию для React проекта
2. **Vite Setup** - Мигрировать простой проект с Webpack на Vite
3. **Bundle Analysis** - Проанализировать размер бандла с webpack-bundle-analyzer

### 🟡 Средний уровень
4. **Code Splitting** - Настроить динамические импорты и lazy loading
5. **Multi-entry** - Создать конфигурацию с множественными точками входа
6. **Library Build** - Настроить Rollup для создания npm библиотеки

### 🔴 Продвинутый уровень
7. **Custom Plugin** - Написать кастомный Webpack плагин
8. **Module Federation** - Настроить микрофронтенды с Webpack 5
9. **Performance Optimization** - Оптимизировать время сборки и размер бандла

## 🔗 Связанные разделы

- **[[build-tools|Build Tools]]** - PostCSS и другие инструменты сборки
- **[[development-environment|Development Environment]]** - Настройка среды разработки
- **[[frontend-cicd|Frontend CI/CD]]** - Автоматизация сборки в CI/CD
- **[[../testing-frontend|Frontend Testing]]** - Интеграция тестов с bundlers

---

⏱️ **Время изучения**: 15-25 часов  
🎯 **Уровень**: Средний - Продвинутый  
📋 **Предварительные требования**: JavaScript ES6+, Node.js, npm 