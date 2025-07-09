# 👁️ Intersection Observer - Отслеживание видимости элементов

> **Intersection Observer** - API для эффективного отслеживания пересечения элементов с viewport или другими элементами.

## 📋 Содержание

- [Основы API](#-основы-api)
- [Ленивая загрузка](#-ленивая-загрузка)
- [Анимации при скролле](#-анимации-при-скролле)
- [Мониторинг производительности](#-мониторинг-производительности)

---

## 🔍 Основы API

### Базовое использование

```javascript
// Класс для управления Intersection Observer
class IntersectionManager {
  constructor() {
    this.observers = new Map();
  }
  
  // Создание observer с настройками
  createObserver(callback, options = {}) {
    const defaultOptions = {
      root: null, // viewport
      rootMargin: '0px',
      threshold: 0.1
    };
    
    const finalOptions = { ...defaultOptions, ...options };
    
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        callback(entry, observer);
      });
    }, finalOptions);
    
    const observerId = Date.now() + Math.random();
    this.observers.set(observerId, observer);
    
    return { observer, observerId };
  }
  
  // Наблюдение за элементом
  observe(element, callback, options = {}) {
    const { observer, observerId } = this.createObserver(callback, options);
    observer.observe(element);
    
    // Возвращаем функцию для отмены наблюдения
    return () => {
      observer.unobserve(element);
      observer.disconnect();
      this.observers.delete(observerId);
    };
  }
  
  // Очистка всех observer'ов
  cleanup() {
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
  }
}

// Глобальный менеджер
const intersectionManager = new IntersectionManager();

// Простой пример использования
function setupBasicObserver() {
  const targets = document.querySelectorAll('.observe-me');
  
  targets.forEach(target => {
    intersectionManager.observe(target, (entry) => {
      if (entry.isIntersecting) {
        console.log('Элемент виден:', entry.target);
        entry.target.classList.add('visible');
      } else {
        entry.target.classList.remove('visible');
      }
    }, {
      threshold: 0.5 // 50% элемента должно быть видно
    });
  });
}
```

---

## 🖼️ Ленивая загрузка

### Изображения и медиа

```javascript
// Класс для ленивой загрузки изображений
class LazyImageLoader {
  constructor() {
    this.loadedImages = new Set();
    this.failedImages = new Set();
    this.initObserver();
  }
  
  initObserver() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.loadImage(entry.target);
        }
      });
    }, {
      rootMargin: '50px' // Загружаем за 50px до появления
    });
  }
  
  // Регистрация изображений для ленивой загрузки
  observe(images) {
    if (typeof images === 'string') {
      images = document.querySelectorAll(images);
    }
    
    images.forEach(img => {
      if (img.dataset.src && !this.loadedImages.has(img)) {
        this.observer.observe(img);
        img.classList.add('lazy-loading');
      }
    });
  }
  
  // Загрузка конкретного изображения
  async loadImage(img) {
    if (this.loadedImages.has(img) || this.failedImages.has(img)) {
      return;
    }
    
    const src = img.dataset.src;
    if (!src) return;
    
    try {
      // Предзагружаем изображение
      await this.preloadImage(src);
      
      // Устанавливаем src и добавляем анимацию появления
      img.src = src;
      img.classList.remove('lazy-loading');
      img.classList.add('lazy-loaded');
      
      this.loadedImages.add(img);
      this.observer.unobserve(img);
      
      // Опционально загружаем версию высокого качества
      if (img.dataset.srcHd) {
        setTimeout(() => this.loadHighQuality(img), 100);
      }
      
    } catch (error) {
      console.error('Ошибка загрузки изображения:', error);
      this.failedImages.add(img);
      img.classList.add('lazy-error');
      this.observer.unobserve(img);
    }
  }
  
  // Предзагрузка изображения
  preloadImage(src) {
    return new Promise((resolve, reject) => {
      const img = new Image();
      img.onload = resolve;
      img.onerror = reject;
      img.src = src;
    });
  }
  
  // Загрузка версии высокого качества
  async loadHighQuality(img) {
    try {
      const hdSrc = img.dataset.srcHd;
      await this.preloadImage(hdSrc);
      
      img.src = hdSrc;
      img.classList.add('hd-loaded');
    } catch (error) {
      console.warn('Не удалось загрузить HD версию:', error);
    }
  }
}

// Ленивая загрузка видео
class LazyVideoLoader {
  constructor() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.loadVideo(entry.target);
        }
      });
    }, {
      rootMargin: '100px'
    });
  }
  
  observe(videos) {
    if (typeof videos === 'string') {
      videos = document.querySelectorAll(videos);
    }
    
    videos.forEach(video => this.observer.observe(video));
  }
  
  loadVideo(video) {
    if (video.dataset.src) {
      video.src = video.dataset.src;
      video.load();
      
      // Автовоспроизведение для muted видео
      if (video.muted) {
        video.play().catch(console.warn);
      }
      
      this.observer.unobserve(video);
    }
  }
}

// Использование ленивой загрузки
const imageLoader = new LazyImageLoader();
const videoLoader = new LazyVideoLoader();

// Инициализация после загрузки DOM
document.addEventListener('DOMContentLoaded', () => {
  imageLoader.observe('img[data-src]');
  videoLoader.observe('video[data-src]');
});
```

---

## 🎬 Анимации при скролле

### Reveal-анимации и параллакс

```javascript
// Класс для анимаций при скролле
class ScrollAnimations {
  constructor() {
    this.animatedElements = new Set();
    this.initObserver();
  }
  
  initObserver() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        this.handleAnimation(entry);
      });
    }, {
      threshold: [0, 0.25, 0.5, 0.75, 1],
      rootMargin: '-10px'
    });
  }
  
  // Регистрация элементов для анимации
  observe(elements, animationType = 'fadeIn') {
    if (typeof elements === 'string') {
      elements = document.querySelectorAll(elements);
    }
    
    elements.forEach(element => {
      element.dataset.animation = animationType;
      element.classList.add('scroll-animate');
      this.observer.observe(element);
    });
  }
  
  handleAnimation(entry) {
    const element = entry.target;
    const animationType = element.dataset.animation;
    const progress = entry.intersectionRatio;
    
    if (entry.isIntersecting) {
      if (!this.animatedElements.has(element)) {
        this.triggerAnimation(element, animationType, progress);
        this.animatedElements.add(element);
      }
    } else {
      // Для повторяющихся анимаций
      if (element.dataset.repeat === 'true') {
        this.animatedElements.delete(element);
        element.classList.remove('animated');
      }
    }
  }
  
  triggerAnimation(element, type, progress) {
    element.classList.add('animated', `animate-${type}`);
    
    switch (type) {
      case 'fadeIn':
        this.fadeInAnimation(element, progress);
        break;
      case 'slideUp':
        this.slideUpAnimation(element, progress);
        break;
      case 'parallax':
        this.parallaxAnimation(element, progress);
        break;
      case 'counter':
        this.counterAnimation(element);
        break;
    }
  }
  
  fadeInAnimation(element, progress) {
    element.style.opacity = progress;
    element.style.transform = `translateY(${(1 - progress) * 30}px)`;
  }
  
  slideUpAnimation(element, progress) {
    element.style.transform = `translateY(${(1 - progress) * 100}px)`;
    element.style.opacity = progress;
  }
  
  parallaxAnimation(element, progress) {
    const speed = parseFloat(element.dataset.speed) || 0.5;
    const yPos = progress * speed * 100;
    element.style.transform = `translateY(${yPos}px)`;
  }
  
  counterAnimation(element) {
    const target = parseInt(element.dataset.target) || 0;
    const duration = parseInt(element.dataset.duration) || 2000;
    const start = Date.now();
    
    const animate = () => {
      const elapsed = Date.now() - start;
      const progress = Math.min(elapsed / duration, 1);
      
      // Easing function
      const easeProgress = progress < 0.5 
        ? 2 * progress * progress 
        : -1 + (4 - 2 * progress) * progress;
      
      const current = Math.floor(target * easeProgress);
      element.textContent = current.toLocaleString();
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    };
    
    animate();
  }
}

// Прогресс-бар чтения
class ReadingProgress {
  constructor() {
    this.progressBar = this.createProgressBar();
    this.initObserver();
  }
  
  createProgressBar() {
    const bar = document.createElement('div');
    bar.className = 'reading-progress';
    bar.style.cssText = `
      position: fixed;
      top: 0;
      left: 0;
      width: 0%;
      height: 3px;
      background: #007bff;
      z-index: 9999;
      transition: width 0.1s ease;
    `;
    document.body.appendChild(bar);
    return bar;
  }
  
  initObserver() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.updateProgress(entry);
        }
      });
    }, {
      threshold: Array.from({length: 101}, (_, i) => i / 100)
    });
  }
  
  observeContent(contentSelector = 'article, .content') {
    const content = document.querySelector(contentSelector);
    if (content) {
      this.observer.observe(content);
    }
  }
  
  updateProgress(entry) {
    const progress = entry.intersectionRatio * 100;
    this.progressBar.style.width = `${progress}%`;
  }
}

// Использование анимаций
const scrollAnimations = new ScrollAnimations();
const readingProgress = new ReadingProgress();

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  scrollAnimations.observe('.fade-in', 'fadeIn');
  scrollAnimations.observe('.slide-up', 'slideUp');
  scrollAnimations.observe('.counter', 'counter');
  scrollAnimations.observe('.parallax', 'parallax');
  
  readingProgress.observeContent();
});
```

---

## 📊 Мониторинг производительности

### Аналитика видимости и взаимодействий

```javascript
// Класс для аналитики видимости
class VisibilityAnalytics {
  constructor() {
    this.viewData = new Map();
    this.initObserver();
  }
  
  initObserver() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        this.trackVisibility(entry);
      });
    }, {
      threshold: [0.1, 0.5, 0.9]
    });
  }
  
  // Отслеживание элементов
  track(elements, category = 'content') {
    if (typeof elements === 'string') {
      elements = document.querySelectorAll(elements);
    }
    
    elements.forEach(element => {
      const elementId = element.id || `element_${Date.now()}_${Math.random()}`;
      element.dataset.trackId = elementId;
      element.dataset.category = category;
      
      this.viewData.set(elementId, {
        element: element,
        category: category,
        firstView: null,
        lastView: null,
        totalViewTime: 0,
        maxVisibility: 0,
        viewCount: 0
      });
      
      this.observer.observe(element);
    });
  }
  
  trackVisibility(entry) {
    const elementId = entry.target.dataset.trackId;
    const data = this.viewData.get(elementId);
    
    if (!data) return;
    
    const now = Date.now();
    
    if (entry.isIntersecting) {
      // Элемент стал виден
      if (!data.firstView) {
        data.firstView = now;
      }
      data.lastView = now;
      data.viewCount++;
      data.maxVisibility = Math.max(data.maxVisibility, entry.intersectionRatio);
      
      // Начинаем отсчет времени просмотра
      data.viewStart = now;
    } else {
      // Элемент скрылся
      if (data.viewStart) {
        data.totalViewTime += now - data.viewStart;
        data.viewStart = null;
      }
    }
    
    // Отправляем событие аналитики
    this.sendAnalyticsEvent(elementId, entry);
  }
  
  sendAnalyticsEvent(elementId, entry) {
    const data = this.viewData.get(elementId);
    
    // Пример отправки в Google Analytics
    if (typeof gtag !== 'undefined') {
      gtag('event', 'element_view', {
        element_id: elementId,
        category: data.category,
        visibility_ratio: entry.intersectionRatio,
        view_count: data.viewCount
      });
    }
    
    // Пример отправки в собственную аналитику
    this.sendToCustomAnalytics({
      event: 'element_visibility',
      elementId: elementId,
      category: data.category,
      isVisible: entry.isIntersecting,
      visibilityRatio: entry.intersectionRatio,
      timestamp: Date.now()
    });
  }
  
  async sendToCustomAnalytics(eventData) {
    try {
      await fetch('/api/analytics', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(eventData)
      });
    } catch (error) {
      console.warn('Ошибка отправки аналитики:', error);
    }
  }
  
  // Получение отчета по видимости
  getVisibilityReport() {
    const report = [];
    
    this.viewData.forEach((data, elementId) => {
      const element = data.element;
      
      report.push({
        elementId: elementId,
        category: data.category,
        tagName: element.tagName,
        className: element.className,
        firstView: data.firstView,
        lastView: data.lastView,
        totalViewTime: data.totalViewTime,
        maxVisibility: data.maxVisibility,
        viewCount: data.viewCount,
        averageViewTime: data.viewCount > 0 ? data.totalViewTime / data.viewCount : 0
      });
    });
    
    return report.sort((a, b) => b.totalViewTime - a.totalViewTime);
  }
}

// Бесконечная прокрутка
class InfiniteScroll {
  constructor(container, loadMore) {
    this.container = container;
    this.loadMore = loadMore;
    this.isLoading = false;
    this.page = 1;
    this.initObserver();
    this.createSentinel();
  }
  
  initObserver() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && !this.isLoading) {
          this.loadMoreContent();
        }
      });
    }, {
      rootMargin: '100px'
    });
  }
  
  createSentinel() {
    this.sentinel = document.createElement('div');
    this.sentinel.className = 'infinite-scroll-sentinel';
    this.sentinel.style.height = '1px';
    this.container.appendChild(this.sentinel);
    this.observer.observe(this.sentinel);
  }
  
  async loadMoreContent() {
    this.isLoading = true;
    this.showLoader();
    
    try {
      const newContent = await this.loadMore(this.page);
      
      if (newContent && newContent.length > 0) {
        this.appendContent(newContent);
        this.page++;
      } else {
        // Больше контента нет
        this.observer.unobserve(this.sentinel);
        this.sentinel.textContent = 'Больше контента нет';
      }
    } catch (error) {
      console.error('Ошибка загрузки контента:', error);
      this.showError();
    } finally {
      this.isLoading = false;
      this.hideLoader();
    }
  }
  
  appendContent(content) {
    content.forEach(item => {
      const element = this.createContentElement(item);
      this.container.insertBefore(element, this.sentinel);
    });
  }
  
  createContentElement(item) {
    // Переопределить в наследнике
    const div = document.createElement('div');
    div.textContent = JSON.stringify(item);
    return div;
  }
  
  showLoader() {
    this.sentinel.innerHTML = '<div class="loader">Загрузка...</div>';
  }
  
  hideLoader() {
    this.sentinel.innerHTML = '';
  }
  
  showError() {
    this.sentinel.innerHTML = '<div class="error">Ошибка загрузки</div>';
  }
}

// Использование
const analytics = new VisibilityAnalytics();

// Отслеживаем важные блоки
analytics.track('.hero-section', 'hero');
analytics.track('.product-card', 'product');
analytics.track('.cta-button', 'conversion');

// Бесконечная прокрутка для списка постов
const postsContainer = document.querySelector('.posts-container');
const infiniteScroll = new InfiniteScroll(postsContainer, async (page) => {
  const response = await fetch(`/api/posts?page=${page}`);
  return response.json();
});
```

## 🎯 Практические задания

### 🟢 Базовый уровень
1. **Lazy Loading** - Ленивая загрузка изображений
2. **Scroll Animations** - Простые анимации при скролле  
3. **Visibility Tracker** - Отслеживание видимости элементов

### 🟡 Средний уровень
4. **Infinite Scroll** - Бесконечная прокрутка с пагинацией
5. **Progress Indicators** - Индикаторы прогресса чтения
6. **Performance Monitor** - Мониторинг производительности

### 🔴 Продвинутый уровень
7. **Advanced Analytics** - Подробная аналитика взаимодействий
8. **Smart Preloading** - Умная предзагрузка контента
9. **Virtual Scrolling** - Виртуализация для больших списков

## 🔗 Связанные разделы

- **[[fetch-api|🌐 Fetch API]]** - Загрузка контента для бесконечной прокрутки
- **[[web-workers|⚡ Web Workers]]** - Обработка больших списков
- **[[service-workers-pwa|🚀 Service Workers]]** - Кэширование медиа-контента
- **[[../../fundamentals/development-principles|⚡ Performance]]** - Оптимизация производительности

---

⏱️ **Время изучения**: 3-4 часа  
🎯 **Уровень**: Средний  
📋 **Предварительные требования**: JavaScript ES6+, DOM API, CSS анимации 