# 🎨 HTML & CSS Fundamentals

## 📖 Содержание
- [Семантический HTML](#семантический-html)
- [CSS Flexbox](#css-flexbox)
- [CSS Grid](#css-grid)
- [CSS Positioning](#css-positioning)
- [Responsive Design](#responsive-design)
- [CSS Animations](#css-animations)
- [CSS Variables](#css-variables)
- [CSS Methodologies](#css-methodologies)

---

## 🏗️ Семантический HTML

### Теория
Семантический HTML использует HTML-элементы в соответствии с их назначением. Это улучшает доступность, SEO и читаемость кода.

### Практические примеры

```html
<!-- ❌ Неправильно: дивы без семантики -->
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
    <div class="nav-item">About</div>
  </div>
</div>

<!-- ✅ Правильно: семантические элементы -->
<header role="banner">
  <!-- header определяет шапку страницы или секции -->
  <nav role="navigation" aria-label="Main navigation">
    <!-- nav содержит навигационные ссылки -->
    <ul>
      <li><a href="/" aria-current="page">Home</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>
</header>

<main role="main">
  <!-- main содержит основной контент страницы -->
  <article>
    <!-- article - самостоятельная единица контента -->
    <header>
      <h1>Заголовок статьи</h1>
      <time datetime="2024-01-15">15 января 2024</time>
    </header>
    
    <section>
      <!-- section группирует связанный контент -->
      <h2>Подзаголовок</h2>
      <p>Содержание секции...</p>
    </section>
    
    <aside>
      <!-- aside содержит дополнительную информацию -->
      <h3>Дополнительная информация</h3>
      <p>Связанные ссылки...</p>
    </aside>
  </article>
</main>

<footer role="contentinfo">
  <!-- footer содержит информацию о документе -->
  <p>&copy; 2024 Моя компания</p>
</footer>
```

### Доступность (A11y)

```html
<!-- Формы с правильной доступностью -->
<form>
  <fieldset>
    <!-- fieldset группирует связанные поля -->
    <legend>Личная информация</legend>
    
    <label for="name">
      Имя *
      <input 
        type="text" 
        id="name" 
        name="name" 
        required 
        aria-describedby="name-help"
        autocomplete="given-name"
      >
    </label>
    <div id="name-help" class="help-text">
      Введите ваше полное имя
    </div>
    
    <label for="email">
      Email *
      <input 
        type="email" 
        id="email" 
        name="email" 
        required
        aria-describedby="email-error"
        autocomplete="email"
      >
    </label>
    <div id="email-error" class="error" aria-live="polite">
      <!-- aria-live уведомляет скрин-ридеры об изменениях -->
    </div>
  </fieldset>
  
  <button type="submit" aria-describedby="submit-help">
    Отправить
  </button>
  <div id="submit-help" class="help-text">
    Нажмите для отправки формы
  </div>
</form>
```

---

## 🔧 CSS Flexbox

### Теория
Flexbox предоставляет эффективный способ размещения, распределения и выравнивания элементов в контейнере, даже когда их размер неизвестен.

### Основные концепции

```css
/* Flex Container - родительский элемент */
.flex-container {
  display: flex; /* Включает flex контекст */
  
  /* Направление главной оси */
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  
  /* Перенос строк */
  flex-wrap: nowrap; /* nowrap | wrap | wrap-reverse */
  
  /* Короткая запись для direction + wrap */
  flex-flow: row nowrap;
  
  /* Выравнивание по главной оси */
  justify-content: flex-start; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
  
  /* Выравнивание по поперечной оси */
  align-items: stretch; /* stretch | flex-start | flex-end | center | baseline */
  
  /* Выравнивание строк при переносе */
  align-content: stretch; /* stretch | flex-start | flex-end | center | space-between | space-around */
}

/* Flex Items - дочерние элементы */
.flex-item {
  /* Порядок отображения */
  order: 0; /* целое число, по умолчанию 0 */
  
  /* Способность расширяться */
  flex-grow: 0; /* число >= 0, по умолчанию 0 */
  
  /* Способность сжиматься */
  flex-shrink: 1; /* число >= 0, по умолчанию 1 */
  
  /* Базовый размер */
  flex-basis: auto; /* auto | <length> | <percentage> */
  
  /* Короткая запись */
  flex: 0 1 auto; /* grow shrink basis */
  
  /* Индивидуальное выравнивание */
  align-self: auto; /* auto | flex-start | flex-end | center | baseline | stretch */
}
```

### Практические примеры

```css
/* Пример 1: Навигационное меню */
.navbar {
  display: flex;
  justify-content: space-between; /* Логотип слева, меню справа */
  align-items: center; /* Вертикальное выравнивание по центру */
  padding: 1rem 2rem;
  background: #fff;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.navbar__logo {
  font-size: 1.5rem;
  font-weight: bold;
}

.navbar__menu {
  display: flex;
  gap: 2rem; /* Расстояние между элементами меню */
  list-style: none;
  margin: 0;
  padding: 0;
}

/* Пример 2: Карточная сетка */
.card-grid {
  display: flex;
  flex-wrap: wrap; /* Разрешаем перенос на новую строку */
  gap: 1rem; /* Отступы между карточками */
  padding: 1rem;
}

.card {
  flex: 1 1 300px; /* Растем и сжимаемся, минимум 300px */
  max-width: 400px; /* Максимальная ширина карточки */
  
  /* Стили карточки */
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  padding: 1.5rem;
  transition: transform 0.2s ease;
}

.card:hover {
  transform: translateY(-2px); /* Небольшой подъем при наведении */
}

/* Пример 3: Центрирование контента */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  
  /* Flexbox для центрирования */
  display: flex;
  justify-content: center; /* Горизонтальное центрирование */
  align-items: center; /* Вертикальное центрирование */
}

.modal {
  background: white;
  border-radius: 8px;
  padding: 2rem;
  max-width: 90vw;
  max-height: 90vh;
  overflow: auto;
}

/* Пример 4: Sticky footer */
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.page__header {
  /* Заголовок фиксированного размера */
}

.page__main {
  flex: 1; /* Занимает все доступное пространство */
  padding: 2rem;
}

.page__footer {
  /* Футер прижат к низу */
  background: #f5f5f5;
  padding: 1rem;
  text-align: center;
}
```

---

## 🎯 CSS Grid

### Теория
CSS Grid - двумерная система макета, позволяющая создавать сложные макеты с строками и столбцами.

### Основные концепции

```css
/* Grid Container */
.grid-container {
  display: grid;
  
  /* Определение колонок */
  grid-template-columns: 200px 1fr 100px; /* 3 колонки: фиксированная, гибкая, фиксированная */
  /* grid-template-columns: repeat(3, 1fr); /* 3 равные колонки */
  /* grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* Адаптивные колонки */
  
  /* Определение строк */
  grid-template-rows: auto 1fr auto; /* Шапка, контент, футер */
  
  /* Зазоры */
  gap: 1rem; /* Или grid-gap в старых браузерах */
  column-gap: 2rem; /* Отдельно для колонок */
  row-gap: 1rem; /* Отдельно для строк */
  
  /* Именованные области */
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}

/* Grid Items */
.grid-item {
  /* Размещение по линиям */
  grid-column-start: 1;
  grid-column-end: 3;
  /* Короткая запись */
  grid-column: 1 / 3; /* От линии 1 до линии 3 */
  grid-column: 1 / span 2; /* От линии 1, занять 2 колонки */
  
  /* Размещение по строкам */
  grid-row: 2 / 4;
  
  /* Еще короче */
  grid-area: 2 / 1 / 4 / 3; /* row-start / column-start / row-end / column-end */
  
  /* Размещение по именованным областям */
  grid-area: header;
}
```

### Практические примеры

```css
/* Пример 1: Основной макет страницы */
.page-layout {
  display: grid;
  min-height: 100vh;
  
  /* Определяем колонки: sidebar, main, aside */
  grid-template-columns: 250px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  
  /* Именованные области для лучшей читаемости */
  grid-template-areas:
    "header  header  header"
    "sidebar main    aside"
    "footer  footer  footer";
  
  gap: 1rem;
}

.page-header { 
  grid-area: header; 
  background: #2c3e50;
  color: white;
  padding: 1rem;
}

.page-sidebar { 
  grid-area: sidebar; 
  background: #ecf0f1;
  padding: 1rem;
}

.page-main { 
  grid-area: main; 
  padding: 1rem;
}

.page-aside { 
  grid-area: aside; 
  background: #f8f9fa;
  padding: 1rem;
}

.page-footer { 
  grid-area: footer; 
  background: #34495e;
  color: white;
  padding: 1rem;
  text-align: center;
}

/* Адаптивность */
@media (max-width: 768px) {
  .page-layout {
    grid-template-columns: 1fr; /* Одна колонка на мобильных */
    grid-template-areas:
      "header"
      "main"
      "sidebar"
      "aside"
      "footer";
  }
}

/* Пример 2: Галерея изображений */
.image-gallery {
  display: grid;
  
  /* Автоматические колонки с минимальной шириной 250px */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  padding: 1rem;
}

.gallery-item {
  position: relative;
  aspect-ratio: 1; /* Квадратные элементы */
  overflow: hidden;
  border-radius: 8px;
  
  /* Некоторые элементы занимают больше места */
}

.gallery-item.featured {
  grid-column: span 2; /* Занимает 2 колонки */
  grid-row: span 2; /* Занимает 2 строки */
}

.gallery-item img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Изображение покрывает весь контейнер */
  transition: transform 0.3s ease;
}

.gallery-item:hover img {
  transform: scale(1.05); /* Увеличение при наведении */
}

/* Пример 3: Форма с Grid */
.form-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1rem;
  max-width: 600px;
  margin: 0 auto;
}

.form-field {
  display: flex;
  flex-direction: column;
}

.form-field.full-width {
  grid-column: 1 / -1; /* Занимает всю ширину */
}

.form-field label {
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-field input,
.form-field textarea {
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.form-field textarea {
  resize: vertical;
  min-height: 100px;
}

/* Адаптивная форма */
@media (max-width: 600px) {
  .form-grid {
    grid-template-columns: 1fr; /* Одна колонка на мобильных */
  }
  
  .form-field.full-width {
    grid-column: 1; /* Сбрасываем spanning */
  }
}
```

---

## 📱 Responsive Design

### Теория
Адаптивный дизайн обеспечивает оптимальное отображение сайта на всех устройствах через гибкие макеты, изображения и медиа-запросы.

### Mobile-First подход

```css
/* Базовые стили для мобильных устройств */
.container {
  width: 100%;
  padding: 1rem;
  margin: 0 auto;
}

.navigation {
  display: flex;
  flex-direction: column; /* Вертикальная навигация на мобильных */
}

.nav-item {
  padding: 0.75rem;
  border-bottom: 1px solid #eee;
}

/* Tablet styles */
@media (min-width: 768px) {
  .container {
    max-width: 750px;
    padding: 1.5rem;
  }
  
  .navigation {
    flex-direction: row; /* Горизонтальная навигация на планшетах */
  }
  
  .nav-item {
    border-bottom: none;
    border-right: 1px solid #eee;
  }
  
  .nav-item:last-child {
    border-right: none;
  }
}

/* Desktop styles */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    padding: 2rem;
  }
  
  /* Дополнительные стили для больших экранов */
}

/* Large desktop styles */
@media (min-width: 1200px) {
  .container {
    max-width: 1400px;
  }
}
```

### Адаптивные изображения

```css
/* Базовые адаптивные изображения */
img {
  max-width: 100%;
  height: auto; /* Сохраняет пропорции */
}

/* Адаптивные фоновые изображения */
.hero-banner {
  background-image: url('hero-mobile.jpg');
  background-size: cover;
  background-position: center;
  min-height: 50vh;
}

@media (min-width: 768px) {
  .hero-banner {
    background-image: url('hero-tablet.jpg');
    min-height: 60vh;
  }
}

@media (min-width: 1024px) {
  .hero-banner {
    background-image: url('hero-desktop.jpg');
    min-height: 80vh;
  }
}

/* CSS для HTML picture элемента */
.responsive-image {
  width: 100%;
  height: auto;
}
```

```html
<!-- HTML для адаптивных изображений -->
<picture>
  <source 
    media="(min-width: 1024px)" 
    srcset="hero-large.webp 1200w, hero-large@2x.webp 2400w"
    type="image/webp"
  >
  <source 
    media="(min-width: 768px)" 
    srcset="hero-medium.webp 800w, hero-medium@2x.webp 1600w"
    type="image/webp"
  >
  <source 
    srcset="hero-small.webp 400w, hero-small@2x.webp 800w"
    type="image/webp"
  >
  <img 
    src="hero-medium.jpg" 
    alt="Описание изображения"
    class="responsive-image"
    loading="lazy"
  >
</picture>
```

### Типографика

```css
/* Адаптивная типографика */
:root {
  /* Базовые размеры для мобильных */
  --font-size-h1: 2rem;
  --font-size-h2: 1.75rem;
  --font-size-h3: 1.5rem;
  --font-size-body: 1rem;
  --line-height: 1.6;
}

@media (min-width: 768px) {
  :root {
    --font-size-h1: 2.5rem;
    --font-size-h2: 2rem;
    --font-size-h3: 1.75rem;
    --font-size-body: 1.125rem;
  }
}

@media (min-width: 1024px) {
  :root {
    --font-size-h1: 3rem;
    --font-size-h2: 2.25rem;
    --font-size-h3: 2rem;
    --font-size-body: 1.125rem;
  }
}

h1 { font-size: var(--font-size-h1); }
h2 { font-size: var(--font-size-h2); }
h3 { font-size: var(--font-size-h3); }
body { 
  font-size: var(--font-size-body); 
  line-height: var(--line-height);
}

/* Fluid typography с clamp() */
h1 {
  font-size: clamp(2rem, 4vw, 3rem); /* мин, предпочтительный, макс */
}

p {
  font-size: clamp(1rem, 2.5vw, 1.25rem);
}
```

---

## 🎬 CSS Animations

### Теория
CSS анимации позволяют создавать плавные переходы и движения без JavaScript, улучшая пользовательский опыт.

### Transitions

```css
/* Базовые переходы */
.button {
  background-color: #3498db;
  color: white;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  /* Переход для всех изменяемых свойств */
  transition: all 0.3s ease;
  
  /* Или более специфично */
  transition: 
    background-color 0.3s ease,
    transform 0.3s ease,
    box-shadow 0.3s ease;
}

.button:hover {
  background-color: #2980b9;
  transform: translateY(-2px); /* Поднимаем кнопку */
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.button:active {
  transform: translateY(0); /* Возвращаем на место при клике */
  transition-duration: 0.1s; /* Быстрый переход для клика */
}

/* Переходы для форм */
.input-field {
  border: 2px solid #ddd;
  padding: 0.75rem;
  border-radius: 4px;
  outline: none;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.input-field:focus {
  border-color: #3498db;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
}

/* Переходы для навигации */
.nav-link {
  color: #333;
  text-decoration: none;
  position: relative;
  transition: color 0.3s ease;
}

.nav-link::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 0;
  height: 2px;
  background-color: #3498db;
  transition: width 0.3s ease;
}

.nav-link:hover {
  color: #3498db;
}

.nav-link:hover::after {
  width: 100%;
}
```

### Keyframe Animations

```css
/* Определение анимации */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* Применение анимаций */
.fade-in {
  animation: fadeInUp 0.6s ease-out;
}

.pulse-button {
  animation: pulse 2s infinite;
}

.loading-spinner {
  animation: spin 1s linear infinite;
}

/* Сложная анимация с несколькими этапами */
@keyframes complexAnimation {
  0% {
    opacity: 0;
    transform: translateX(-100px) rotate(-180deg);
  }
  25% {
    opacity: 0.5;
    transform: translateX(-50px) rotate(-90deg);
  }
  50% {
    opacity: 0.8;
    transform: translateX(0) rotate(0deg);
  }
  75% {
    opacity: 0.9;
    transform: translateX(10px) rotate(10deg);
  }
  100% {
    opacity: 1;
    transform: translateX(0) rotate(0deg);
  }
}

.complex-element {
  animation: complexAnimation 1.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

/* Анимация при скролле (с Intersection Observer) */
.scroll-reveal {
  opacity: 0;
  transform: translateY(50px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.scroll-reveal.revealed {
  opacity: 1;
  transform: translateY(0);
}

/* Hover эффекты для карточек */
.card {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.15);
}

/* Анимация загрузки */
@keyframes skeleton {
  0% {
    background-position: -200px 0;
  }
  100% {
    background-position: calc(200px + 100%) 0;
  }
}

.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200px 100%;
  animation: skeleton 1.5s infinite;
}
```

---

## 🎨 CSS Variables (Custom Properties)

### Теория
CSS переменные позволяют хранить значения и переиспользовать их, упрощая поддержку и создание тем.

### Основы использования

```css
/* Определение глобальных переменных */
:root {
  /* Цветовая палитра */
  --primary-color: #3498db;
  --primary-hover: #2980b9;
  --primary-light: #e3f2fd;
  
  --secondary-color: #95a5a6;
  --accent-color: #e74c3c;
  
  --text-color: #333;
  --text-muted: #666;
  --text-light: #999;
  
  --bg-color: #ffffff;
  --bg-alt: #f8f9fa;
  --border-color: #dee2e6;
  
  /* Размеры */
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  --border-radius: 4px;
  --border-radius-lg: 8px;
  
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.1);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
  --shadow-lg: 0 10px 25px rgba(0,0,0,0.15);
  
  /* Анимации */
  --transition-fast: 0.15s ease;
  --transition-base: 0.3s ease;
  --transition-slow: 0.5s ease;
}

/* Использование переменных */
.button {
  background-color: var(--primary-color);
  color: white;
  padding: var(--spacing-sm) var(--spacing-md);
  border: none;
  border-radius: var(--border-radius);
  font-size: var(--font-size-base);
  transition: background-color var(--transition-base);
  box-shadow: var(--shadow-sm);
}

.button:hover {
  background-color: var(--primary-hover);
  box-shadow: var(--shadow-md);
}

/* Переменные с fallback значениями */
.element {
  color: var(--custom-color, var(--text-color)); /* Если custom-color не определен, используем text-color */
  margin: var(--custom-margin, var(--spacing-md)); /* Fallback на spacing-md */
}
```

### Темизация

```css
/* Светлая тема (по умолчанию) */
:root {
  --theme-bg: #ffffff;
  --theme-text: #333333;
  --theme-border: #dee2e6;
  --theme-shadow: rgba(0,0,0,0.1);
}

/* Темная тема */
[data-theme="dark"] {
  --theme-bg: #1a1a1a;
  --theme-text: #e0e0e0;
  --theme-border: #404040;
  --theme-shadow: rgba(255,255,255,0.1);
  
  /* Инвертируем некоторые цвета */
  --primary-light: #1e3a8a;
}

/* Применение темы */
body {
  background-color: var(--theme-bg);
  color: var(--theme-text);
  border-color: var(--theme-border);
  transition: background-color var(--transition-base), color var(--transition-base);
}

.card {
  background-color: var(--theme-bg);
  border: 1px solid var(--theme-border);
  box-shadow: 0 2px 4px var(--theme-shadow);
}

/* Переключение темы через JavaScript */
/*
const toggleTheme = () => {
  const currentTheme = document.documentElement.getAttribute('data-theme');
  const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
  document.documentElement.setAttribute('data-theme', newTheme);
  localStorage.setItem('theme', newTheme);
};
*/
```

### Динамические переменные

```css
/* Переменные для компонентов */
.progress-bar {
  --progress-width: 0%; /* Будет изменяться через JavaScript */
  --progress-color: var(--primary-color);
  
  width: 100%;
  height: 8px;
  background-color: var(--bg-alt);
  border-radius: var(--border-radius);
  overflow: hidden;
}

.progress-bar::after {
  content: '';
  display: block;
  width: var(--progress-width);
  height: 100%;
  background-color: var(--progress-color);
  transition: width var(--transition-base);
}

/* Адаптивные переменные */
:root {
  --container-padding: var(--spacing-md);
}

@media (min-width: 768px) {
  :root {
    --container-padding: var(--spacing-lg);
  }
}

@media (min-width: 1024px) {
  :root {
    --container-padding: var(--spacing-xl);
  }
}

.container {
  padding: var(--container-padding);
}

/* Переменные для анимаций */
.animated-element {
  --animation-duration: 0.3s;
  --animation-delay: 0s;
  --animation-easing: ease-in-out;
  
  transition: 
    transform var(--animation-duration) var(--animation-easing) var(--animation-delay),
    opacity var(--animation-duration) var(--animation-easing) var(--animation-delay);
}

/* Изменение переменных через hover */
.interactive-card {
  --card-scale: 1;
  --card-shadow: var(--shadow-sm);
  
  transform: scale(var(--card-scale));
  box-shadow: var(--card-shadow);
  transition: transform var(--transition-base), box-shadow var(--transition-base);
}

.interactive-card:hover {
  --card-scale: 1.02;
  --card-shadow: var(--shadow-lg);
}
```

---

## 🏗️ CSS Methodologies

### BEM (Block Element Modifier)

```css
/* Блок - независимый компонент */
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 1rem;
}

/* Элементы - части блока */
.card__header {
  border-bottom: 1px solid #eee;
  padding-bottom: 1rem;
  margin-bottom: 1rem;
}

.card__title {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0;
}

.card__subtitle {
  color: #666;
  font-size: 0.875rem;
  margin: 0.25rem 0 0 0;
}

.card__body {
  line-height: 1.6;
}

.card__footer {
  border-top: 1px solid #eee;
  padding-top: 1rem;
  margin-top: 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card__actions {
  display: flex;
  gap: 0.5rem;
}

/* Модификаторы - вариации блока или элемента */
.card--featured {
  border: 2px solid #3498db;
  box-shadow: 0 4px 8px rgba(52, 152, 219, 0.2);
}

.card--large {
  padding: 2rem;
}

.card__title--large {
  font-size: 1.5rem;
}

.card__actions--centered {
  justify-content: center;
}

/* Состояния (можно комбинировать с модификаторами) */
.card.is-loading {
  opacity: 0.6;
  pointer-events: none;
}

.card.is-error {
  border-color: #e74c3c;
  background-color: #fdf2f2;
}
```

```html
<!-- HTML с BEM -->
<article class="card card--featured">
  <header class="card__header">
    <h2 class="card__title card__title--large">Заголовок карточки</h2>
    <p class="card__subtitle">Подзаголовок</p>
  </header>
  
  <div class="card__body">
    <p>Содержимое карточки...</p>
  </div>
  
  <footer class="card__footer">
    <div class="card__actions card__actions--centered">
      <button class="button button--primary">Действие</button>
      <button class="button button--secondary">Отмена</button>
    </div>
  </footer>
</article>
```

### Utility-First (как в Tailwind CSS)

```css
/* Утилитарные классы для типографики */
.text-xs { font-size: 0.75rem; }
.text-sm { font-size: 0.875rem; }
.text-base { font-size: 1rem; }
.text-lg { font-size: 1.125rem; }
.text-xl { font-size: 1.25rem; }

.font-light { font-weight: 300; }
.font-normal { font-weight: 400; }
.font-medium { font-weight: 500; }
.font-semibold { font-weight: 600; }
.font-bold { font-weight: 700; }

.text-left { text-align: left; }
.text-center { text-align: center; }
.text-right { text-align: right; }

/* Утилитарные классы для отступов */
.m-0 { margin: 0; }
.m-1 { margin: 0.25rem; }
.m-2 { margin: 0.5rem; }
.m-3 { margin: 0.75rem; }
.m-4 { margin: 1rem; }

.mt-0 { margin-top: 0; }
.mt-1 { margin-top: 0.25rem; }
.mt-2 { margin-top: 0.5rem; }
.mt-4 { margin-top: 1rem; }

.p-0 { padding: 0; }
.p-1 { padding: 0.25rem; }
.p-2 { padding: 0.5rem; }
.p-4 { padding: 1rem; }
.p-6 { padding: 1.5rem; }

/* Утилитарные классы для цветов */
.text-gray-900 { color: #1a202c; }
.text-gray-700 { color: #2d3748; }
.text-gray-500 { color: #718096; }
.text-blue-600 { color: #3182ce; }
.text-red-600 { color: #e53e3e; }

.bg-white { background-color: #ffffff; }
.bg-gray-100 { background-color: #f7fafc; }
.bg-blue-500 { background-color: #4299e1; }

/* Утилитарные классы для флексбокса */
.flex { display: flex; }
.inline-flex { display: inline-flex; }
.flex-col { flex-direction: column; }
.flex-row { flex-direction: row; }

.justify-start { justify-content: flex-start; }
.justify-center { justify-content: center; }
.justify-between { justify-content: space-between; }
.justify-end { justify-content: flex-end; }

.items-start { align-items: flex-start; }
.items-center { align-items: center; }
.items-end { align-items: flex-end; }

/* Утилитарные классы для позиционирования */
.relative { position: relative; }
.absolute { position: absolute; }
.fixed { position: fixed; }
.static { position: static; }

.top-0 { top: 0; }
.right-0 { right: 0; }
.bottom-0 { bottom: 0; }
.left-0 { left: 0; }

/* Адаптивные утилиты */
@media (min-width: 640px) {
  .sm\:flex { display: flex; }
  .sm\:hidden { display: none; }
  .sm\:text-lg { font-size: 1.125rem; }
}

@media (min-width: 768px) {
  .md\:flex { display: flex; }
  .md\:grid { display: grid; }
  .md\:text-xl { font-size: 1.25rem; }
}
```

---

## 📚 Связанные разделы

- [[../javascript-fundamentals|JavaScript Fundamentals]] - Основы JavaScript для интерактивности
- [[../react-ecosystem|React Ecosystem]] - Современная библиотека для UI
- [[frontend-advanced|Frontend Advanced]] - Продвинутые техники frontend разработки
- [[../technical-skills/testing|Testing]] - Тестирование frontend приложений
- [[../architecture/design-patterns|Design Patterns]] - Паттерны в frontend архитектуре
- [[../fundamentals/web-standards|Web Standards]] - Веб-стандарты и best practices 