# 🏗️ Build Tools - Инструменты сборки

> **Build Tools** - специализированные инструменты для обработки CSS, изображений, шрифтов и других ресурсов в процессе сборки проекта.

## 📋 Обзор раздела

Build Tools обрабатывают статические ресурсы, оптимизируют CSS, компилируют препроцессоры и подготавливают активы для продакшена. Современные инструменты сборки автоматизируют множество рутинных задач.

## 🎨 PostCSS - Современная обработка CSS

### PostCSS Configuration

```javascript
// postcss.config.js - Основная конфигурация
module.exports = {
  plugins: [
    // Autoprefixer - автоматические вендорные префиксы
    require('autoprefixer'),
    
    // PostCSS Preset Env - современный CSS
    require('postcss-preset-env')({
      stage: 1, // Уровень экспериментальных возможностей
      features: {
        'nesting-rules': true,
        'custom-media-queries': true,
        'custom-properties': true,
        'media-query-ranges': true
      },
      autoprefixer: {
        grid: 'autoplace'
      }
    }),
    
    // PurgeCSS - удаление неиспользуемого CSS
    require('@fullhuman/postcss-purgecss')({
      content: [
        './src/**/*.html',
        './src/**/*.jsx',
        './src/**/*.tsx',
        './src/**/*.js',
        './src/**/*.ts'
      ],
      defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
      safelist: {
        standard: [/^btn-/, /^modal-/, /^dropdown-/],
        deep: [/^data-theme/],
        greedy: [/^fa-/]
      }
    }),
    
    // CSSnano - минификация CSS
    process.env.NODE_ENV === 'production' && require('cssnano')({
      preset: ['default', {
        discardComments: {
          removeAll: true
        },
        normalizeWhitespace: false
      }]
    }),
    
    // PostCSS Import - поддержка @import
    require('postcss-import')({
      path: ['src/styles', 'node_modules']
    }),
    
    // PostCSS Nested - вложенность как в Sass
    require('postcss-nested'),
    
    // PostCSS Mixins - миксины
    require('postcss-mixins')({
      mixinsFiles: 'src/styles/mixins/*.pcss'
    }),
    
    // PostCSS Simple Vars - переменные
    require('postcss-simple-vars')({
      variables: require('./src/styles/variables.js')
    }),
    
    // PostCSS Functions - кастомные функции
    require('postcss-functions')({
      functions: {
        // Функция для rem конвертации
        rem: (value) => `${value / 16}rem`,
        
        // Функция для цветовых вариаций
        lighten: (color, amount) => {
          // Логика осветления цвета
          return color;
        }
      }
    })
  ].filter(Boolean)
};
```

### Продвинутые PostCSS плагины

```javascript
// postcss.config.advanced.js
const path = require('path');

module.exports = {
  plugins: [
    // Critical CSS - выделение критического CSS
    require('postcss-critical-split')({
      output: 'critical',
      modules: ['@namespace critical'],
      separator: '/* critical:end */'
    }),
    
    // Responsive breakpoints
    require('postcss-custom-media')({
      importFrom: 'src/styles/media-queries.css'
    }),
    
    // Design tokens integration
    require('postcss-design-tokens')({
      files: ['src/design-tokens/**/*.json']
    }),
    
    // SVG инлайнинг
    require('postcss-inline-svg')({
      paths: ['src/assets/icons']
    }),
    
    // Спрайты изображений
    require('postcss-sprites')({
      stylesheetPath: './dist/css',
      spritePath: './dist/images/',
      filterBy: (image) => {
        return image.url.indexOf('/sprite/') !== -1;
      },
      groupBy: (image) => {
        const groups = image.url.split('/');
        return groups[groups.length - 2];
      }
    }),
    
    // Оптимизация шрифтов
    require('postcss-font-display')({
      display: 'swap',
      replace: false
    }),
    
    // Анализ размера CSS
    require('postcss-size')(),
    
    // Сортировка CSS свойств
    require('postcss-sorting')({
      order: [
        'custom-properties',
        'dollar-variables',
        'declarations',
        'at-rules',
        'rules'
      ],
      'properties-order': 'alphabetical',
      'unspecified-properties-position': 'bottom'
    })
  ]
};
```

## 🎯 Sass/SCSS Build Pipeline

### Sass Configuration

```javascript
// sass.config.js
const sass = require('sass');
const postcss = require('postcss');
const autoprefixer = require('autoprefixer');

module.exports = {
  // Sass опции
  sassOptions: {
    implementation: sass,
    sourceMap: true,
    outputStyle: 'expanded',
    
    // Пути для @import
    includePaths: [
      'node_modules',
      'src/styles',
      'src/styles/abstracts',
      'src/styles/components'
    ],
    
    // Кастомные функции Sass
    functions: {
      // Функция для breakpoints
      'breakpoint($name)': (name) => {
        const breakpoints = {
          'small': '640px',
          'medium': '768px',
          'large': '1024px',
          'xlarge': '1280px'
        };
        return new sass.types.String(breakpoints[name.getValue()]);
      },
      
      // Функция для z-index управления
      'z-index($layer)': (layer) => {
        const layers = {
          'below': -1,
          'base': 0,
          'content': 10,
          'overlay': 100,
          'modal': 1000,
          'tooltip': 10000
        };
        return new sass.types.Number(layers[layer.getValue()]);
      }
    }
  },
  
  // PostCSS обработка после Sass
  postProcess: async (css, sourceMap) => {
    const result = await postcss([
      autoprefixer,
      require('postcss-preset-env')()
    ]).process(css, {
      from: undefined,
      map: { prev: sourceMap, inline: false }
    });
    
    return {
      css: result.css,
      map: result.map
    };
  }
};
```

### SCSS Structure

```scss
// src/styles/main.scss - Главный файл стилей
// =============================================================================
// SETTINGS
// =============================================================================
@import 'abstracts/variables';
@import 'abstracts/functions';
@import 'abstracts/mixins';
@import 'abstracts/placeholders';

// =============================================================================
// TOOLS
// =============================================================================
@import 'vendors/normalize';
@import 'vendors/bootstrap-grid';

// =============================================================================
// GENERIC
// =============================================================================
@import 'base/reset';
@import 'base/typography';
@import 'base/helpers';

// =============================================================================
// ELEMENTS
// =============================================================================
@import 'elements/buttons';
@import 'elements/forms';
@import 'elements/tables';

// =============================================================================
// OBJECTS
// =============================================================================
@import 'objects/layout';
@import 'objects/container';
@import 'objects/grid';

// =============================================================================
// COMPONENTS
// =============================================================================
@import 'components/header';
@import 'components/navigation';
@import 'components/hero';
@import 'components/card';
@import 'components/modal';
@import 'components/footer';

// =============================================================================
// UTILITIES
// =============================================================================
@import 'utilities/spacing';
@import 'utilities/typography';
@import 'utilities/colors';
@import 'utilities/display';

// =============================================================================
// THEMES
// =============================================================================
@import 'themes/default';
@import 'themes/dark';
```

```scss
// src/styles/abstracts/_mixins.scss - Полезные миксины
// Responsive breakpoints
@mixin respond-to($breakpoint) {
  @if $breakpoint == small {
    @media (min-width: 640px) { @content; }
  }
  @if $breakpoint == medium {
    @media (min-width: 768px) { @content; }
  }
  @if $breakpoint == large {
    @media (min-width: 1024px) { @content; }
  }
  @if $breakpoint == xlarge {
    @media (min-width: 1280px) { @content; }
  }
}

// Flexbox utilities
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin flex-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

// Typography scale
@mixin font-size($size, $line-height: null) {
  font-size: $size;
  @if $line-height {
    line-height: $line-height;
  }
}

// Button styles
@mixin button-variant($bg, $color: white, $border: $bg) {
  background-color: $bg;
  color: $color;
  border: 1px solid $border;
  
  &:hover {
    background-color: darken($bg, 10%);
    border-color: darken($border, 10%);
  }
  
  &:focus {
    box-shadow: 0 0 0 3px rgba($bg, 0.3);
  }
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
}

// Card component
@mixin card($padding: 1rem, $border-radius: 8px, $shadow: true) {
  padding: $padding;
  border-radius: $border-radius;
  background: white;
  
  @if $shadow {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }
}

// Visually hidden
@mixin visually-hidden {
  position: absolute !important;
  width: 1px !important;
  height: 1px !important;
  padding: 0 !important;
  margin: -1px !important;
  overflow: hidden !important;
  clip: rect(0, 0, 0, 0) !important;
  white-space: nowrap !important;
  border: 0 !important;
}
```

## 🖼️ Asset Processing

### Image Optimization

```javascript
// imagemin.config.js
const imagemin = require('imagemin');
const imageminJpegtran = require('imagemin-jpegtran');
const imageminPngquant = require('imagemin-pngquant');
const imageminSvgo = require('imagemin-svgo');
const imageminWebp = require('imagemin-webp');

module.exports = {
  // Базовая оптимизация
  plugins: [
    // JPEG оптимизация
    imageminJpegtran({
      progressive: true
    }),
    
    // PNG оптимизация
    imageminPngquant({
      quality: [0.6, 0.8]
    }),
    
    // SVG оптимизация
    imageminSvgo({
      plugins: [
        { removeViewBox: false },
        { removeDimensions: true },
        { cleanupIDs: true }
      ]
    })
  ],
  
  // WebP генерация
  webp: {
    plugins: [
      imageminWebp({
        quality: 75,
        method: 6
      })
    ]
  },
  
  // Responsive images
  responsive: {
    sizes: [400, 800, 1200, 1600],
    quality: 80,
    progressive: true,
    withoutEnlargement: true
  }
};
```

### Font Processing

```javascript
// fonts.config.js
const fontmin = require('fontmin');

module.exports = {
  // Субсеты шрифтов
  subsets: {
    latin: 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789',
    cyrillic: 'АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯабвгдеёжзийклмнопрстуфхцчшщъыьэюя',
    numbers: '0123456789',
    punctuation: '.,!?;:""()[]{}@#$%^&*-_=+|\\/<>~`'
  },
  
  // Форматы вывода
  formats: ['woff2', 'woff', 'ttf'],
  
  // Минификация
  minify: true,
  
  // CSS генерация
  generateCSS: true,
  fontDisplay: 'swap'
};
```

## 🔧 Webpack Asset Processing

### Asset Loaders Configuration

```javascript
// webpack.assets.config.js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      // CSS/PostCSS обработка
      {
        test: /\.css$/,
        use: [
          process.env.NODE_ENV === 'production'
            ? MiniCssExtractPlugin.loader
            : 'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                auto: true,
                localIdentName: '[name]__[local]--[hash:base64:5]'
              },
              sourceMap: true,
              importLoaders: 1
            }
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                config: path.resolve(__dirname, 'postcss.config.js')
              }
            }
          }
        ]
      },
      
      // SCSS обработка
      {
        test: /\.scss$/,
        use: [
          process.env.NODE_ENV === 'production'
            ? MiniCssExtractPlugin.loader
            : 'style-loader',
          'css-loader',
          'postcss-loader',
          {
            loader: 'sass-loader',
            options: {
              implementation: require('sass'),
              sassOptions: {
                includePaths: ['src/styles']
              }
            }
          }
        ]
      },
      
      // Изображения с оптимизацией
      {
        test: /\.(png|jpg|jpeg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              name: 'images/[name].[hash:8].[ext]'
            }
          },
          {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: {
                progressive: true,
                quality: 80
              },
              optipng: {
                enabled: false
              },
              pngquant: {
                quality: [0.6, 0.8],
                speed: 4
              },
              gifsicle: {
                interlaced: false
              }
            }
          }
        ]
      },
      
      // SVG как компоненты React
      {
        test: /\.svg$/,
        issuer: /\.[jt]sx?$/,
        use: [
          {
            loader: '@svgr/webpack',
            options: {
              icon: true,
              typescript: true,
              dimensions: false
            }
          }
        ]
      },
      
      // Шрифты
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: {
          loader: 'file-loader',
          options: {
            name: 'fonts/[name].[hash:8].[ext]'
          }
        }
      }
    ]
  },
  
  plugins: [
    // Извлечение CSS
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
      chunkFilename: 'css/[name].[contenthash:8].chunk.css'
    })
  ]
};
```

## 🎯 Advanced Build Optimizations

### Critical CSS Extraction

```javascript
// critical-css.config.js
const critical = require('critical');
const glob = require('glob');

async function generateCriticalCSS() {
  const pages = [
    { url: 'http://localhost:3000/', name: 'home' },
    { url: 'http://localhost:3000/products', name: 'products' },
    { url: 'http://localhost:3000/about', name: 'about' }
  ];
  
  for (const page of pages) {
    await critical.generate({
      inline: false,
      base: 'dist/',
      src: page.url,
      dest: `critical-${page.name}.css`,
      width: 1300,
      height: 900,
      minify: true,
      extract: true,
      ignore: {
        atrule: ['@font-face'],
        rule: [/\.non-critical/],
        decl: (node, value) => {
          return /url\(/.test(value);
        }
      }
    });
  }
}

module.exports = { generateCriticalCSS };
```

### Design Tokens Integration

```javascript
// design-tokens.config.js
const StyleDictionary = require('style-dictionary');

StyleDictionary.extend({
  source: ['tokens/**/*.json'],
  platforms: {
    scss: {
      transformGroup: 'scss',
      buildPath: 'src/styles/generated/',
      files: [
        {
          destination: '_tokens.scss',
          format: 'scss/variables'
        }
      ]
    },
    css: {
      transformGroup: 'css',
      buildPath: 'src/styles/generated/',
      files: [
        {
          destination: 'tokens.css',
          format: 'css/variables'
        }
      ]
    },
    js: {
      transformGroup: 'js',
      buildPath: 'src/tokens/',
      files: [
        {
          destination: 'tokens.js',
          format: 'javascript/es6'
        }
      ]
    }
  }
}).buildAllPlatforms();
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **PostCSS Setup** - Настроить PostCSS с автопрефиксером и preset-env
2. **Sass Integration** - Интегрировать Sass в процесс сборки
3. **Image Optimization** - Настроить оптимизацию изображений

### 🟡 Средний уровень
4. **Critical CSS** - Извлечь критический CSS для ускорения загрузки
5. **Design Tokens** - Настроить систему дизайн-токенов
6. **Advanced PostCSS** - Использовать продвинутые PostCSS плагины

### 🔴 Продвинутый уровень
7. **Custom PostCSS Plugin** - Написать собственный PostCSS плагин
8. **Asset Pipeline** - Создать полную систему обработки ресурсов
9. **Performance Budget** - Настроить мониторинг размера ресурсов

## 🔗 Связанные разделы

- **[[bundlers|Bundlers]]** - Интеграция с системами сборки
- **[[package-managers|Package Managers]]** - Управление зависимостями build tools
- **[[development-environment|Development Environment]]** - Настройка среды разработки
- **[[../css-layout-mastery|CSS Layout]]** - Современные CSS техники

---

⏱️ **Время изучения**: 12-20 часов  
🎯 **Уровень**: Средний - Продвинутый  
📋 **Предварительные требования**: CSS, Sass/SCSS, Node.js 