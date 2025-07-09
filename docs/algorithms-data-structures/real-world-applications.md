# Реальные применения алгоритмов

> **Как алгоритмы и структуры данных используются в современной разработке**
>
> От веб-приложений до машинного обучения

## 🌐 Веб-разработка

### Кэширование (Hash Tables)

```python
# ==================== LRU CACHE ДЛЯ ВЕБ-ПРИЛОЖЕНИЯ ====================
# Кэширование результатов API запросов
from collections import OrderedDict
import time

class LRUCache:
    """
    LRU Cache для кэширования результатов API
    Используется в: Redis, Memcached, браузерах
    """
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = OrderedDict()
        self.timestamps = {}
    
    def get(self, key):
        """Получение из кэша - O(1)"""
        if key in self.cache:
            # Перемещаем в конец (most recently used)
            self.cache.move_to_end(key)
            return self.cache[key]
        return None
    
    def put(self, key, value, ttl=300):
        """Добавление в кэш - O(1)"""
        if key in self.cache:
            self.cache.move_to_end(key)
        else:
            if len(self.cache) >= self.capacity:
                # Удаляем LRU элемент
                oldest_key = next(iter(self.cache))
                del self.cache[oldest_key]
                del self.timestamps[oldest_key]
        
        self.cache[key] = value
        self.timestamps[key] = time.time() + ttl

# ==================== ПРИМЕНЕНИЕ В DJANGO/FLASK ====================
class APICache:
    """Кэш для API endpoints"""
    
    def __init__(self):
        self.cache = LRUCache(1000)
    
    def cached_api_call(self, endpoint, params):
        """Кэширование API вызовов"""
        cache_key = f"{endpoint}:{hash(str(params))}"
        
        # Проверяем кэш
        cached_result = self.cache.get(cache_key)
        if cached_result:
            return cached_result
        
        # Выполняем реальный API вызов
        result = self.make_api_call(endpoint, params)
        
        # Кэшируем результат
        self.cache.put(cache_key, result, ttl=300)
        return result
    
    def make_api_call(self, endpoint, params):
        """Имитация API вызова"""
        # Здесь был бы реальный HTTP запрос
        return f"Result for {endpoint} with {params}"

# ==================== RATE LIMITING ====================
class RateLimiter:
    """
    Rate limiting для API
    Алгоритм: Sliding Window
    """
    def __init__(self, max_requests=100, window_seconds=60):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = {}  # user_id -> [timestamps]
    
    def is_allowed(self, user_id):
        """Проверка rate limit - O(k) где k - количество запросов в окне"""
        now = time.time()
        
        if user_id not in self.requests:
            self.requests[user_id] = []
        
        # Убираем старые запросы
        user_requests = self.requests[user_id]
        cutoff_time = now - self.window_seconds
        
        # Фильтруем запросы в текущем окне
        self.requests[user_id] = [
            timestamp for timestamp in user_requests 
            if timestamp > cutoff_time
        ]
        
        # Проверяем лимит
        if len(self.requests[user_id]) >= self.max_requests:
            return False
        
        # Добавляем текущий запрос
        self.requests[user_id].append(now)
        return True

# Пример использования в Flask
"""
@app.route('/api/data')
def get_data():
    user_id = request.headers.get('User-ID')
    
    if not rate_limiter.is_allowed(user_id):
        return {'error': 'Rate limit exceeded'}, 429
    
    return api_cache.cached_api_call('/data', request.args)
"""
```

### Поиск и автокомплит (Trie)

```python
# ==================== TRIE ДЛЯ АВТОКОМПЛИТА ====================
class TrieNode:
    """Узел префиксного дерева"""
    def __init__(self):
        self.children = {}
        self.is_end_word = False
        self.frequency = 0  # Для ранжирования предложений

class AutocompleteService:
    """
    Сервис автокомплита
    Используется в: Google Search, IDE, e-commerce поиск
    """
    def __init__(self):
        self.root = TrieNode()
    
    def add_word(self, word, frequency=1):
        """Добавление слова в индекс - O(m) где m - длина слова"""
        node = self.root
        for char in word.lower():
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        
        node.is_end_word = True
        node.frequency += frequency
    
    def search_suggestions(self, prefix, max_results=10):
        """Поиск предложений по prefix - O(p + n) где p - длина prefix"""
        node = self.root
        
        # Находим узел с prefix
        for char in prefix.lower():
            if char not in node.children:
                return []
            node = node.children[char]
        
        # Собираем все слова с данным prefix
        suggestions = []
        self._collect_words(node, prefix, suggestions)
        
        # Сортируем по частоте
        suggestions.sort(key=lambda x: x[1], reverse=True)
        
        return [word for word, freq in suggestions[:max_results]]
    
    def _collect_words(self, node, prefix, suggestions):
        """Рекурсивный сбор слов"""
        if node.is_end_word:
            suggestions.append((prefix, node.frequency))
        
        for char, child_node in node.children.items():
            self._collect_words(child_node, prefix + char, suggestions)

# ==================== ИНТЕГРАЦИЯ С ELASTICSEARCH ====================
class SearchService:
    """Сервис поиска с поддержкой fuzzy matching"""
    
    def __init__(self):
        self.trie = AutocompleteService()
        self.load_popular_searches()
    
    def load_popular_searches(self):
        """Загрузка популярных поисковых запросов"""
        popular_searches = [
            ("python tutorial", 1000),
            ("javascript framework", 800),
            ("react components", 600),
            ("python web scraping", 500),
            ("javascript async", 400)
        ]
        
        for query, frequency in popular_searches:
            self.trie.add_word(query, frequency)
    
    def get_suggestions(self, query, max_suggestions=5):
        """Получение предложений для поиска"""
        return self.trie.search_suggestions(query, max_suggestions)
    
    def fuzzy_search(self, query, max_distance=2):
        """Поиск с учетом опечаток (edit distance)"""
        # Реализация алгоритма Левенштейна для fuzzy search
        # Используется в Elasticsearch, Solr
        pass

# Пример использования в React/Vue
"""
// Frontend код
const SearchComponent = () => {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  
  const handleInputChange = async (value) => {
    setQuery(value);
    
    if (value.length > 2) {
      const response = await fetch(`/api/suggestions?q=${value}`);
      const data = await response.json();
      setSuggestions(data.suggestions);
    }
  };
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => handleInputChange(e.target.value)}
        placeholder="Поиск..."
      />
      <ul>
        {suggestions.map(suggestion => (
          <li key={suggestion}>{suggestion}</li>
        ))}
      </ul>
    </div>
  );
};
"""
```

---

## 🗄️ Базы данных

### Индексы (B-деревья)

```python
# ==================== B-TREE INDEX СИМУЛЯЦИЯ ====================
class BTreeNode:
    """Узел B-дерева (упрощенная версия)"""
    def __init__(self, is_leaf=False):
        self.keys = []
        self.values = []
        self.children = []
        self.is_leaf = is_leaf
        self.max_keys = 3  # Для простоты демонстрации

class DatabaseIndex:
    """
    Симуляция индекса базы данных
    Используется в: MySQL, PostgreSQL, MongoDB
    """
    def __init__(self):
        self.root = BTreeNode(is_leaf=True)
    
    def search(self, key):
        """Поиск по индексу - O(log n)"""
        return self._search_node(self.root, key)
    
    def _search_node(self, node, key):
        """Рекурсивный поиск в узле"""
        i = 0
        while i < len(node.keys) and key > node.keys[i]:
            i += 1
        
        if i < len(node.keys) and key == node.keys[i]:
            return node.values[i]
        
        if node.is_leaf:
            return None
        
        return self._search_node(node.children[i], key)
    
    def insert(self, key, value):
        """Вставка в индекс - O(log n)"""
        # Упрощенная реализация
        # В реальности здесь сложная логика split/merge узлов
        pass

# ==================== QUERY OPTIMIZER ====================
class QueryOptimizer:
    """
    Оптимизатор запросов
    Использует статистику для выбора оптимального плана
    """
    def __init__(self):
        self.table_stats = {}
        self.index_stats = {}
    
    def analyze_query(self, sql_query):
        """Анализ SQL запроса"""
        # Парсинг WHERE условий
        conditions = self.parse_where_conditions(sql_query)
        
        # Выбор оптимального индекса
        best_index = self.choose_best_index(conditions)
        
        # Оценка стоимости выполнения
        cost = self.estimate_cost(conditions, best_index)
        
        return {
            'plan': 'index_scan' if best_index else 'table_scan',
            'index': best_index,
            'estimated_cost': cost
        }
    
    def parse_where_conditions(self, sql):
        """Парсинг WHERE условий"""
        # Упрощенная реализация
        # В реальности используются сложные парсеры
        return []
    
    def choose_best_index(self, conditions):
        """Выбор лучшего индекса"""
        # Анализ селективности индексов
        # Оценка кардинальности результата
        return None
    
    def estimate_cost(self, conditions, index):
        """Оценка стоимости выполнения"""
        # Модель стоимости на основе:
        # - I/O операций
        # - CPU время
        # - Память
        return 100
```

### Кэширование запросов

```python
# ==================== QUERY CACHE ====================
import hashlib
import json

class QueryCache:
    """
    Кэш запросов к базе данных
    Используется в: Redis, Memcached, Application Layer
    """
    def __init__(self, redis_client):
        self.redis = redis_client
        self.default_ttl = 300  # 5 минут
    
    def get_cached_query(self, sql, params):
        """Получение результата из кэша"""
        cache_key = self._generate_cache_key(sql, params)
        
        cached_result = self.redis.get(cache_key)
        if cached_result:
            return json.loads(cached_result)
        
        return None
    
    def cache_query_result(self, sql, params, result, ttl=None):
        """Кэширование результата запроса"""
        cache_key = self._generate_cache_key(sql, params)
        ttl = ttl or self.default_ttl
        
        serialized_result = json.dumps(result, default=str)
        self.redis.setex(cache_key, ttl, serialized_result)
    
    def _generate_cache_key(self, sql, params):
        """Генерация ключа кэша"""
        query_signature = f"{sql}:{json.dumps(params, sort_keys=True)}"
        return hashlib.md5(query_signature.encode()).hexdigest()
    
    def invalidate_cache(self, pattern):
        """Инвалидация кэша по паттерну"""
        keys = self.redis.keys(f"*{pattern}*")
        if keys:
            self.redis.delete(*keys)

# ==================== DATABASE WRAPPER ====================
class CachedDatabase:
    """Обертка для базы данных с кэшированием"""
    
    def __init__(self, db_connection, query_cache):
        self.db = db_connection
        self.cache = query_cache
    
    def execute_query(self, sql, params=None, use_cache=True):
        """Выполнение запроса с кэшированием"""
        params = params or {}
        
        # Проверяем кэш для SELECT запросов
        if use_cache and sql.strip().upper().startswith('SELECT'):
            cached_result = self.cache.get_cached_query(sql, params)
            if cached_result:
                return cached_result
        
        # Выполняем запрос к базе
        result = self._execute_raw_query(sql, params)
        
        # Кэшируем результат SELECT запросов
        if use_cache and sql.strip().upper().startswith('SELECT'):
            self.cache.cache_query_result(sql, params, result)
        
        # Инвалидируем кэш для модифицирующих запросов
        if self._is_modifying_query(sql):
            table_name = self._extract_table_name(sql)
            self.cache.invalidate_cache(table_name)
        
        return result
    
    def _execute_raw_query(self, sql, params):
        """Выполнение запроса к базе"""
        # Здесь выполняется реальный SQL запрос
        pass
    
    def _is_modifying_query(self, sql):
        """Проверка на модифицирующий запрос"""
        modifying_keywords = ['INSERT', 'UPDATE', 'DELETE', 'CREATE', 'DROP', 'ALTER']
        return any(sql.strip().upper().startswith(keyword) for keyword in modifying_keywords)
    
    def _extract_table_name(self, sql):
        """Извлечение имени таблицы из SQL"""
        # Упрощенная реализация
        # В реальности нужен полноценный SQL парсер
        return 'table'
```

---

## 🚀 Системы обработки данных

### MapReduce для больших данных

```python
# ==================== MAPREDUCE FRAMEWORK ====================
from multiprocessing import Pool
from collections import defaultdict
import json

class MapReduceFramework:
    """
    Упрощенная реализация MapReduce
    Используется в: Hadoop, Apache Spark
    """
    def __init__(self, num_workers=4):
        self.num_workers = num_workers
    
    def run_job(self, data, mapper, reducer):
        """Выполнение MapReduce job"""
        # Фаза Map
        mapped_data = self._map_phase(data, mapper)
        
        # Фаза Shuffle & Sort
        shuffled_data = self._shuffle_phase(mapped_data)
        
        # Фаза Reduce
        result = self._reduce_phase(shuffled_data, reducer)
        
        return result
    
    def _map_phase(self, data, mapper):
        """Map фаза - параллельная обработка"""
        with Pool(self.num_workers) as pool:
            chunk_size = len(data) // self.num_workers
            chunks = [data[i:i + chunk_size] for i in range(0, len(data), chunk_size)]
            
            mapped_results = pool.map(mapper, chunks)
        
        # Объединяем результаты
        all_mapped = []
        for result in mapped_results:
            all_mapped.extend(result)
        
        return all_mapped
    
    def _shuffle_phase(self, mapped_data):
        """Shuffle фаза - группировка по ключам"""
        shuffled = defaultdict(list)
        
        for key, value in mapped_data:
            shuffled[key].append(value)
        
        return shuffled
    
    def _reduce_phase(self, shuffled_data, reducer):
        """Reduce фаза - агрегация"""
        results = {}
        
        for key, values in shuffled_data.items():
            results[key] = reducer(key, values)
        
        return results

# ==================== ПРИМЕР: ПОДСЧЕТ СЛОВ ====================
def word_count_mapper(text_chunk):
    """Mapper для подсчета слов"""
    result = []
    for line in text_chunk:
        words = line.strip().split()
        for word in words:
            result.append((word.lower(), 1))
    return result

def word_count_reducer(key, values):
    """Reducer для подсчета слов"""
    return sum(values)

# ==================== АНАЛИЗ ЛОГОВ ====================
class LogAnalyzer:
    """
    Анализатор логов сервера
    Обрабатывает терабайты логов распределенно
    """
    def __init__(self):
        self.mapreduce = MapReduceFramework(num_workers=8)
    
    def analyze_access_logs(self, log_files):
        """Анализ логов доступа"""
        # Загружаем логи
        logs = self._load_logs(log_files)
        
        # Подсчет запросов по IP
        ip_counts = self.mapreduce.run_job(
            logs, 
            self._ip_mapper, 
            self._count_reducer
        )
        
        # Подсчет ошибок по времени
        error_timeline = self.mapreduce.run_job(
            logs,
            self._error_mapper,
            self._count_reducer
        )
        
        return {
            'top_ips': self._get_top_n(ip_counts, 10),
            'error_timeline': error_timeline
        }
    
    def _load_logs(self, log_files):
        """Загрузка логов"""
        logs = []
        for file_path in log_files:
            with open(file_path, 'r') as f:
                logs.extend(f.readlines())
        return logs
    
    def _ip_mapper(self, log_chunk):
        """Mapper для извлечения IP адресов"""
        result = []
        for line in log_chunk:
            # Парсинг Apache/Nginx лога
            parts = line.split()
            if len(parts) > 0:
                ip = parts[0]
                result.append((ip, 1))
        return result
    
    def _error_mapper(self, log_chunk):
        """Mapper для анализа ошибок"""
        result = []
        for line in log_chunk:
            parts = line.split()
            if len(parts) > 8:
                status_code = parts[8]
                timestamp = parts[3][1:12]  # Извлекаем час
                
                if status_code.startswith('4') or status_code.startswith('5'):
                    result.append((timestamp, 1))
        return result
    
    def _count_reducer(self, key, values):
        """Reducer для подсчета"""
        return sum(values)
    
    def _get_top_n(self, data, n):
        """Получение топ N элементов"""
        return sorted(data.items(), key=lambda x: x[1], reverse=True)[:n]

# Пример использования
"""
analyzer = LogAnalyzer()
log_files = ['/var/log/nginx/access.log.1', '/var/log/nginx/access.log.2']
results = analyzer.analyze_access_logs(log_files)

print("Топ IP адресов:")
for ip, count in results['top_ips']:
    print(f"{ip}: {count} запросов")
"""
```

---

## 🤖 Machine Learning

### Алгоритмы рекомендаций

```python
# ==================== COLLABORATIVE FILTERING ====================
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

class RecommendationSystem:
    """
    Система рекомендаций
    Используется в: Netflix, Spotify, Amazon, YouTube
    """
    def __init__(self):
        self.user_item_matrix = None
        self.user_similarity = None
        self.item_similarity = None
    
    def fit(self, user_item_matrix):
        """Обучение модели"""
        self.user_item_matrix = user_item_matrix
        
        # Вычисляем similarity матрицы
        self.user_similarity = cosine_similarity(user_item_matrix)
        self.item_similarity = cosine_similarity(user_item_matrix.T)
    
    def recommend_items(self, user_id, n_recommendations=10):
        """Рекомендации для пользователя"""
        # Collaborative Filtering на основе пользователей
        user_scores = self._calculate_user_based_scores(user_id)
        
        # Фильтруем уже просмотренные
        user_items = self.user_item_matrix[user_id]
        unseen_items = np.where(user_items == 0)[0]
        
        # Сортируем по убыванию score
        recommendations = sorted(
            [(item, user_scores[item]) for item in unseen_items],
            key=lambda x: x[1],
            reverse=True
        )
        
        return recommendations[:n_recommendations]
    
    def _calculate_user_based_scores(self, user_id):
        """Вычисление scores на основе похожих пользователей"""
        similar_users = self.user_similarity[user_id]
        
        # Взвешенная сумма ratings от похожих пользователей
        scores = np.zeros(self.user_item_matrix.shape[1])
        
        for item_id in range(len(scores)):
            numerator = 0
            denominator = 0
            
            for other_user_id in range(len(similar_users)):
                if other_user_id != user_id:
                    similarity = similar_users[other_user_id]
                    rating = self.user_item_matrix[other_user_id, item_id]
                    
                    if rating > 0:  # Пользователь оценил этот item
                        numerator += similarity * rating
                        denominator += abs(similarity)
            
            scores[item_id] = numerator / denominator if denominator > 0 else 0
        
        return scores

# ==================== CONTENT-BASED FILTERING ====================
from sklearn.feature_extraction.text import TfidfVectorizer

class ContentBasedRecommender:
    """
    Рекомендации на основе контента
    Используется для рекомендации статей, фильмов, музыки
    """
    def __init__(self):
        self.tfidf = TfidfVectorizer(max_features=10000, stop_words='english')
        self.item_features = None
        self.item_similarity = None
    
    def fit(self, items_data):
        """Обучение на описаниях items"""
        # items_data = [{'id': 1, 'title': '...', 'description': '...', 'genre': '...'}]
        
        # Создаем текстовое представление каждого item
        item_texts = []
        for item in items_data:
            text = f"{item['title']} {item['description']} {item['genre']}"
            item_texts.append(text)
        
        # Векторизуем тексты
        self.item_features = self.tfidf.fit_transform(item_texts)
        
        # Вычисляем similarity между items
        self.item_similarity = cosine_similarity(self.item_features)
    
    def recommend_similar_items(self, item_id, n_recommendations=10):
        """Рекомендации похожих items"""
        # Получаем similarity scores для данного item
        similarity_scores = self.item_similarity[item_id]
        
        # Сортируем по убыванию similarity
        similar_items = sorted(
            enumerate(similarity_scores),
            key=lambda x: x[1],
            reverse=True
        )[1:]  # Исключаем сам item
        
        return similar_items[:n_recommendations]

# ==================== HYBRID RECOMMENDER ====================
class HybridRecommender:
    """
    Гибридная система рекомендаций
    Комбинирует collaborative и content-based подходы
    """
    def __init__(self, collaborative_weight=0.7, content_weight=0.3):
        self.collaborative = RecommendationSystem()
        self.content_based = ContentBasedRecommender()
        self.collaborative_weight = collaborative_weight
        self.content_weight = content_weight
    
    def fit(self, user_item_matrix, items_data):
        """Обучение обеих моделей"""
        self.collaborative.fit(user_item_matrix)
        self.content_based.fit(items_data)
    
    def recommend(self, user_id, n_recommendations=10):
        """Гибридные рекомендации"""
        # Получаем рекомендации от обеих систем
        collab_recs = self.collaborative.recommend_items(user_id, n_recommendations * 2)
        
        # Комбинируем scores
        hybrid_scores = {}
        
        for item_id, collab_score in collab_recs:
            # Получаем content-based score
            content_score = self._get_content_score(user_id, item_id)
            
            # Взвешенная комбинация
            hybrid_score = (
                self.collaborative_weight * collab_score +
                self.content_weight * content_score
            )
            
            hybrid_scores[item_id] = hybrid_score
        
        # Сортируем по гибридному score
        recommendations = sorted(
            hybrid_scores.items(),
            key=lambda x: x[1],
            reverse=True
        )
        
        return recommendations[:n_recommendations]
    
    def _get_content_score(self, user_id, item_id):
        """Получение content-based score"""
        # Упрощенная реализация
        # В реальности анализируем профиль пользователя
        return 0.5
```

---

## 📊 Аналитика и метрики

### Потоковая обработка данных

```python
# ==================== STREAM PROCESSING ====================
import heapq
from collections import deque
import time

class StreamProcessor:
    """
    Обработчик потоковых данных
    Используется в: Apache Kafka, Apache Storm, AWS Kinesis
    """
    def __init__(self, window_size=60):
        self.window_size = window_size  # секунды
        self.data_buffer = deque()
        self.metrics = {}
    
    def process_event(self, event):
        """Обработка события в реальном времени"""
        current_time = time.time()
        
        # Добавляем событие в буфер
        self.data_buffer.append({
            'event': event,
            'timestamp': current_time
        })
        
        # Очищаем старые события
        self._clean_old_events(current_time)
        
        # Обновляем метрики
        self._update_metrics()
    
    def _clean_old_events(self, current_time):
        """Удаление событий вне временного окна"""
        cutoff_time = current_time - self.window_size
        
        while self.data_buffer and self.data_buffer[0]['timestamp'] < cutoff_time:
            self.data_buffer.popleft()
    
    def _update_metrics(self):
        """Обновление метрик в реальном времени"""
        if not self.data_buffer:
            return
        
        # Подсчет событий по типам
        event_counts = {}
        total_events = 0
        
        for item in self.data_buffer:
            event_type = item['event'].get('type', 'unknown')
            event_counts[event_type] = event_counts.get(event_type, 0) + 1
            total_events += 1
        
        # Обновляем метрики
        self.metrics.update({
            'total_events': total_events,
            'events_per_type': event_counts,
            'events_per_second': total_events / self.window_size
        })
    
    def get_metrics(self):
        """Получение текущих метрик"""
        return self.metrics

# ==================== REAL-TIME ANALYTICS ====================
class RealTimeAnalytics:
    """
    Аналитика в реальном времени
    Для дашбордов, мониторинга, alerting
    """
    def __init__(self):
        self.processors = {}
        self.alerts = []
    
    def add_stream(self, stream_name, processor):
        """Добавление потока данных"""
        self.processors[stream_name] = processor
    
    def process_log_entry(self, log_entry):
        """Обработка записи лога"""
        # Парсинг лога
        parsed_log = self._parse_log(log_entry)
        
        # Обработка в соответствующих процессорах
        for stream_name, processor in self.processors.items():
            if self._should_process(stream_name, parsed_log):
                processor.process_event(parsed_log)
                
                # Проверка на алерты
                self._check_alerts(stream_name, processor)
    
    def _parse_log(self, log_entry):
        """Парсинг лога"""
        # Упрощенный парсинг
        parts = log_entry.split()
        
        return {
            'type': 'http_request',
            'ip': parts[0] if len(parts) > 0 else 'unknown',
            'status_code': parts[8] if len(parts) > 8 else '200',
            'response_time': float(parts[10]) if len(parts) > 10 else 0,
            'timestamp': time.time()
        }
    
    def _should_process(self, stream_name, event):
        """Определение, должен ли процессор обрабатывать событие"""
        # Фильтрация по типу события
        if stream_name == 'error_stream':
            return event.get('status_code', '200').startswith(('4', '5'))
        elif stream_name == 'performance_stream':
            return event.get('response_time', 0) > 0
        
        return True
    
    def _check_alerts(self, stream_name, processor):
        """Проверка условий для алертов"""
        metrics = processor.get_metrics()
        
        # Алерт на высокий RPS
        if metrics.get('events_per_second', 0) > 1000:
            self.alerts.append({
                'type': 'high_rps',
                'stream': stream_name,
                'value': metrics['events_per_second'],
                'timestamp': time.time()
            })
        
        # Алерт на высокий процент ошибок
        if stream_name == 'error_stream':
            error_rate = metrics.get('events_per_second', 0)
            if error_rate > 10:  # Более 10 ошибок в секунду
                self.alerts.append({
                    'type': 'high_error_rate',
                    'stream': stream_name,
                    'value': error_rate,
                    'timestamp': time.time()
                })
    
    def get_dashboard_data(self):
        """Данные для дашборда"""
        dashboard = {}
        
        for stream_name, processor in self.processors.items():
            dashboard[stream_name] = processor.get_metrics()
        
        dashboard['alerts'] = self.alerts[-10:]  # Последние 10 алертов
        
        return dashboard

# Пример использования
"""
analytics = RealTimeAnalytics()

# Добавляем потоки
analytics.add_stream('error_stream', StreamProcessor(window_size=60))
analytics.add_stream('performance_stream', StreamProcessor(window_size=300))

# Обрабатываем логи в реальном времени
log_entry = "192.168.1.1 - - [25/Dec/2023:10:00:00 +0000] GET /api/users 500 1234 0.5"
analytics.process_log_entry(log_entry)

# Получаем данные для дашборда
dashboard_data = analytics.get_dashboard_data()
"""
```

---

## 🌟 Итоговые рекомендации

### Когда использовать разные алгоритмы

```python
# ==================== GUIDE ПО ВЫБОРУ АЛГОРИТМОВ ====================

algorithm_guide = {
    "Веб-разработка": {
        "Кэширование": ["LRU Cache", "Hash Tables"],
        "Автокомплит": ["Trie", "Suffix Array"],
        "Rate Limiting": ["Sliding Window", "Token Bucket"],
        "Поиск": ["Elasticsearch", "Inverted Index"]
    },
    
    "Базы данных": {
        "Индексирование": ["B-Trees", "Hash Index"],
        "Запросы": ["Query Optimization", "Join Algorithms"],
        "Кэширование": ["Redis", "Memcached"]
    },
    
    "Машинное обучение": {
        "Рекомендации": ["Collaborative Filtering", "Content-Based"],
        "Поиск": ["Vector Search", "Approximate Nearest Neighbor"],
        "Обработка текста": ["TF-IDF", "Word Embeddings"]
    },
    
    "Аналитика": {
        "Большие данные": ["MapReduce", "Spark"],
        "Потоки": ["Stream Processing", "Time Series"],
        "Мониторинг": ["Sliding Window", "Exponential Smoothing"]
    }
}

def choose_algorithm(domain, problem_type):
    """Выбор алгоритма для конкретной задачи"""
    if domain in algorithm_guide:
        return algorithm_guide[domain].get(problem_type, ["Generic Solution"])
    return ["Research needed"]

# Пример использования
print("Для веб-разработки и автокомплита:")
print(choose_algorithm("Веб-разработка", "Автокомплит"))
```

---

*Алгоритмы в реальном мире — это не просто код, а решения бизнес-задач* 🚀💼 