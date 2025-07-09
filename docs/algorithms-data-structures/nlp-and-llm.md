# Natural Language Processing & Large Language Models

> **Обработка естественного языка и большие языковые модели**
>
> От токенизации до трансформеров и современных LLM

## 🔤 Основы NLP

### Предобработка текста

```python
import re
import numpy as np
from collections import Counter, defaultdict
import math
from typing import List, Dict, Tuple, Optional

# ==================== ТОКЕНИЗАЦИЯ ====================
class TextPreprocessor:
    """
    Базовая предобработка текста
    """
    
    def __init__(self):
        self.stop_words = {
            'english': {'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for', 'of', 'with', 'by', 'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has', 'had', 'do', 'does', 'did', 'will', 'would', 'could', 'should', 'may', 'might', 'must', 'can', 'this', 'that', 'these', 'those', 'i', 'you', 'he', 'she', 'it', 'we', 'they', 'me', 'him', 'her', 'us', 'them'},
            'russian': {'и', 'в', 'на', 'с', 'по', 'к', 'от', 'для', 'за', 'при', 'не', 'что', 'как', 'его', 'ее', 'их', 'был', 'была', 'было', 'были', 'есть', 'быть', 'или', 'но', 'да', 'нет', 'это', 'то', 'так', 'вот', 'еще', 'уже', 'тут', 'где', 'когда', 'кто', 'что', 'чем', 'про', 'под', 'над', 'без', 'через', 'между', 'среди', 'около', 'после', 'перед', 'во', 'со', 'из', 'до', 'же', 'ли', 'бы', 'только', 'даже', 'тоже', 'также', 'если', 'чтобы', 'потому', 'поэтому', 'хотя', 'несмотря'}
        }
    
    def tokenize(self, text: str) -> List[str]:
        """
        Токенизация текста
        
        Разбивает текст на отдельные токены (слова, знаки препинания)
        """
        # Приведение к нижнему регистру
        text = text.lower()
        
        # Удаление HTML тегов
        text = re.sub(r'<[^>]+>', '', text)
        
        # Удаление URL
        text = re.sub(r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\\(\\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+', '', text)
        
        # Удаление email
        text = re.sub(r'\S+@\S+', '', text)
        
        # Токенизация по словам и знакам препинания
        tokens = re.findall(r'\b\w+\b|[^\w\s]', text)
        
        return tokens
    
    def remove_stopwords(self, tokens: List[str], language: str = 'english') -> List[str]:
        """
        Удаление стоп-слов
        
        Убирает часто встречающиеся но малоинформативные слова
        """
        stop_words = self.stop_words.get(language, set())
        return [token for token in tokens if token not in stop_words]
    
    def stem_words(self, tokens: List[str]) -> List[str]:
        """
        Стемминг - приведение слов к корню
        
        Упрощенный алгоритм стемминга
        """
        # Простые правила стемминга для английского
        stemming_rules = [
            (r'ing$', ''),
            (r'ed$', ''),
            (r'er$', ''),
            (r'est$', ''),
            (r's$', ''),
            (r'ly$', ''),
            (r'tion$', ''),
            (r'ness$', ''),
            (r'ment$', ''),
        ]
        
        stemmed = []
        for token in tokens:
            for pattern, replacement in stemming_rules:
                if re.search(pattern, token) and len(token) > 3:
                    token = re.sub(pattern, replacement, token)
                    break
            stemmed.append(token)
        
        return stemmed
    
    def clean_text(self, text: str, language: str = 'english', 
                   remove_stops: bool = True, stem: bool = True) -> List[str]:
        """
        Полная предобработка текста
        """
        # Токенизация
        tokens = self.tokenize(text)
        
        # Удаление стоп-слов
        if remove_stops:
            tokens = self.remove_stopwords(tokens, language)
        
        # Стемминг
        if stem:
            tokens = self.stem_words(tokens)
        
        # Удаление коротких токенов
        tokens = [token for token in tokens if len(token) > 2]
        
        return tokens

# ==================== N-ГРАММЫ ====================
class NGramLanguageModel:
    """
    Языковая модель на основе n-грамм
    
    Предсказывает следующее слово на основе предыдущих n-1 слов
    """
    
    def __init__(self, n: int = 3):
        self.n = n
        self.ngrams = defaultdict(Counter)
        self.vocab = set()
    
    def train(self, texts: List[str]):
        """
        Обучение на корпусе текстов
        """
        preprocessor = TextPreprocessor()
        
        for text in texts:
            tokens = preprocessor.clean_text(text)
            self.vocab.update(tokens)
            
            # Добавляем специальные токены для начала и конца
            padded_tokens = ['<START>'] * (self.n - 1) + tokens + ['<END>']
            
            # Создаем n-граммы
            for i in range(len(padded_tokens) - self.n + 1):
                ngram = tuple(padded_tokens[i:i + self.n])
                context = ngram[:-1]
                next_word = ngram[-1]
                
                self.ngrams[context][next_word] += 1
    
    def get_probability(self, context: Tuple[str, ...], next_word: str) -> float:
        """
        Вероятность следующего слова по контексту
        
        P(w_n | w_{n-1}, w_{n-2}, ..., w_1) = Count(w_1...w_n) / Count(w_1...w_{n-1})
        """
        if context not in self.ngrams:
            return 0.0
        
        context_count = sum(self.ngrams[context].values())
        word_count = self.ngrams[context][next_word]
        
        return word_count / context_count if context_count > 0 else 0.0
    
    def get_probability_with_smoothing(self, context: Tuple[str, ...], next_word: str) -> float:
        """
        Вероятность с лаплассовым сглаживанием
        
        Добавляет 1 к каждому счетчику для обработки неизвестных слов
        """
        if context not in self.ngrams:
            return 1.0 / (len(self.vocab) + 1)
        
        context_count = sum(self.ngrams[context].values())
        word_count = self.ngrams[context][next_word]
        vocab_size = len(self.vocab)
        
        return (word_count + 1) / (context_count + vocab_size)
    
    def generate_text(self, context: Tuple[str, ...], max_length: int = 50) -> str:
        """
        Генерация текста на основе модели
        """
        generated = list(context)
        
        for _ in range(max_length):
            # Получаем распределение вероятностей для следующего слова
            if context in self.ngrams:
                candidates = list(self.ngrams[context].keys())
                probabilities = [self.get_probability_with_smoothing(context, word) 
                               for word in candidates]
                
                # Выбираем слово случайно согласно распределению
                if probabilities:
                    next_word = np.random.choice(candidates, p=probabilities)
                    
                    if next_word == '<END>':
                        break
                    
                    generated.append(next_word)
                    
                    # Обновляем контекст
                    context = tuple(generated[-(self.n-1):])
                else:
                    break
            else:
                break
        
        return ' '.join(generated[self.n-1:])  # Убираем START токены
    
    def calculate_perplexity(self, test_texts: List[str]) -> float:
        """
        Вычисление перплексии - метрики качества языковой модели
        
        Perplexity = 2^(-log_2(P(text)))
        Чем меньше перплексия, тем лучше модель
        """
        preprocessor = TextPreprocessor()
        total_log_prob = 0.0
        total_words = 0
        
        for text in test_texts:
            tokens = preprocessor.clean_text(text)
            padded_tokens = ['<START>'] * (self.n - 1) + tokens + ['<END>']
            
            for i in range(len(padded_tokens) - self.n + 1):
                ngram = tuple(padded_tokens[i:i + self.n])
                context = ngram[:-1]
                next_word = ngram[-1]
                
                prob = self.get_probability_with_smoothing(context, next_word)
                if prob > 0:
                    total_log_prob += math.log2(prob)
                    total_words += 1
        
        avg_log_prob = total_log_prob / total_words if total_words > 0 else 0
        perplexity = 2 ** (-avg_log_prob)
        
        return perplexity

# ==================== TF-IDF ====================
class TFIDFVectorizer:
    """
    Term Frequency - Inverse Document Frequency
    
    Преобразует документы в векторы на основе важности слов
    """
    
    def __init__(self, max_features: Optional[int] = None):
        self.max_features = max_features
        self.vocabulary = {}
        self.idf_values = {}
        self.preprocessor = TextPreprocessor()
    
    def fit(self, documents: List[str]):
        """
        Обучение на корпусе документов
        """
        # Предобработка всех документов
        processed_docs = [self.preprocessor.clean_text(doc) for doc in documents]
        
        # Подсчет частоты документов для каждого слова
        doc_frequencies = Counter()
        all_words = set()
        
        for doc_tokens in processed_docs:
            unique_words = set(doc_tokens)
            for word in unique_words:
                doc_frequencies[word] += 1
                all_words.add(word)
        
        # Создание словаря
        if self.max_features:
            # Берем самые частые слова
            most_common = doc_frequencies.most_common(self.max_features)
            self.vocabulary = {word: idx for idx, (word, _) in enumerate(most_common)}
        else:
            self.vocabulary = {word: idx for idx, word in enumerate(all_words)}
        
        # Вычисление IDF
        total_docs = len(documents)
        for word in self.vocabulary:
            df = doc_frequencies[word]
            # IDF = log(N / df) где N - общее количество документов
            self.idf_values[word] = math.log(total_docs / df)
    
    def transform(self, documents: List[str]) -> np.ndarray:
        """
        Преобразование документов в TF-IDF векторы
        """
        tfidf_matrix = np.zeros((len(documents), len(self.vocabulary)))
        
        for doc_idx, document in enumerate(documents):
            tokens = self.preprocessor.clean_text(document)
            
            # Подсчет TF (term frequency)
            tf_counter = Counter(tokens)
            total_terms = len(tokens)
            
            for word, count in tf_counter.items():
                if word in self.vocabulary:
                    word_idx = self.vocabulary[word]
                    
                    # TF = count / total_terms_in_doc
                    tf = count / total_terms
                    
                    # IDF для этого слова
                    idf = self.idf_values[word]
                    
                    # TF-IDF = TF * IDF
                    tfidf_matrix[doc_idx, word_idx] = tf * idf
        
        return tfidf_matrix
    
    def fit_transform(self, documents: List[str]) -> np.ndarray:
        """
        Обучение и преобразование в одном методе
        """
        self.fit(documents)
        return self.transform(documents)

# ==================== WORD2VEC (УПРОЩЕННАЯ ВЕРСИЯ) ====================
class Word2Vec:
    """
    Упрощенная реализация Word2Vec
    
    Обучает векторные представления слов
    """
    
    def __init__(self, vector_size: int = 100, window: int = 5, 
                 min_count: int = 1, learning_rate: float = 0.025):
        self.vector_size = vector_size
        self.window = window
        self.min_count = min_count
        self.learning_rate = learning_rate
        self.vocab = {}
        self.word_vectors = {}
        self.preprocessor = TextPreprocessor()
    
    def build_vocab(self, documents: List[str]):
        """
        Построение словаря
        """
        word_counts = Counter()
        
        for document in documents:
            tokens = self.preprocessor.clean_text(document)
            word_counts.update(tokens)
        
        # Фильтруем слова по минимальной частоте
        filtered_words = {word: count for word, count in word_counts.items() 
                         if count >= self.min_count}
        
        # Создаем словарь
        self.vocab = {word: idx for idx, word in enumerate(filtered_words.keys())}
        
        # Инициализация векторов слов
        vocab_size = len(self.vocab)
        for word in self.vocab:
            self.word_vectors[word] = np.random.uniform(-0.5, 0.5, self.vector_size)
    
    def generate_training_data(self, documents: List[str]) -> List[Tuple[str, str]]:
        """
        Генерация обучающих пар (центральное слово, контекстное слово)
        """
        training_data = []
        
        for document in documents:
            tokens = self.preprocessor.clean_text(document)
            # Фильтруем только слова из словаря
            tokens = [token for token in tokens if token in self.vocab]
            
            for i, target_word in enumerate(tokens):
                # Определяем окно контекста
                start = max(0, i - self.window)
                end = min(len(tokens), i + self.window + 1)
                
                for j in range(start, end):
                    if i != j:  # Не берем само слово
                        context_word = tokens[j]
                        training_data.append((target_word, context_word))
        
        return training_data
    
    def sigmoid(self, x: float) -> float:
        """Сигмоидная функция"""
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
    
    def train(self, documents: List[str], epochs: int = 5):
        """
        Обучение Word2Vec (упрощенная версия Skip-gram)
        """
        self.build_vocab(documents)
        training_data = self.generate_training_data(documents)
        
        for epoch in range(epochs):
            total_loss = 0
            
            for target_word, context_word in training_data:
                # Векторы слов
                target_vector = self.word_vectors[target_word]
                context_vector = self.word_vectors[context_word]
                
                # Прямое распространение
                dot_product = np.dot(target_vector, context_vector)
                prediction = self.sigmoid(dot_product)
                
                # Ошибка (цель = 1 для положительного примера)
                error = 1 - prediction
                total_loss += -math.log(prediction + 1e-10)
                
                # Обновление векторов
                gradient = error * self.learning_rate
                self.word_vectors[target_word] += gradient * context_vector
                self.word_vectors[context_word] += gradient * target_vector
            
            if epoch % 1 == 0:
                print(f"Epoch {epoch + 1}/{epochs}, Loss: {total_loss:.4f}")
    
    def get_similar_words(self, word: str, top_k: int = 5) -> List[Tuple[str, float]]:
        """
        Поиск похожих слов по косинусному сходству
        """
        if word not in self.word_vectors:
            return []
        
        word_vector = self.word_vectors[word]
        similarities = []
        
        for other_word, other_vector in self.word_vectors.items():
            if other_word != word:
                # Косинусное сходство
                cosine_sim = np.dot(word_vector, other_vector) / (
                    np.linalg.norm(word_vector) * np.linalg.norm(other_vector)
                )
                similarities.append((other_word, cosine_sim))
        
        # Сортировка по убыванию сходства
        similarities.sort(key=lambda x: x[1], reverse=True)
        return similarities[:top_k]

# ==================== ATTENTION MECHANISM ====================
class AttentionMechanism:
    """
    Базовый механизм внимания
    
    Позволяет модели фокусироваться на разных частях входной последовательности
    """
    
    def __init__(self, hidden_size: int):
        self.hidden_size = hidden_size
        # Веса для вычисления attention scores
        self.W_attention = np.random.randn(hidden_size, hidden_size) * 0.1
        self.v_attention = np.random.randn(hidden_size, 1) * 0.1
    
    def compute_attention_scores(self, hidden_states: np.ndarray) -> np.ndarray:
        """
        Вычисление attention scores
        
        hidden_states: [seq_len, hidden_size]
        """
        # Вычисляем энергию для каждого скрытого состояния
        energy = np.tanh(np.dot(hidden_states, self.W_attention))
        
        # Вычисляем attention scores
        scores = np.dot(energy, self.v_attention).flatten()
        
        return scores
    
    def apply_attention(self, hidden_states: np.ndarray, 
                       attention_scores: np.ndarray) -> np.ndarray:
        """
        Применение attention к скрытым состояниям
        """
        # Softmax для нормализации scores
        attention_weights = self.softmax(attention_scores)
        
        # Взвешенная сумма скрытых состояний
        context_vector = np.dot(attention_weights, hidden_states)
        
        return context_vector, attention_weights
    
    def softmax(self, x: np.ndarray) -> np.ndarray:
        """Softmax функция"""
        exp_x = np.exp(x - np.max(x))
        return exp_x / np.sum(exp_x)

# ==================== TRANSFORMER (УПРОЩЕННАЯ ВЕРСИЯ) ====================
class MultiHeadAttention:
    """
    Multi-Head Attention из архитектуры Transformer
    """
    
    def __init__(self, d_model: int, n_heads: int):
        self.d_model = d_model
        self.n_heads = n_heads
        self.d_k = d_model // n_heads
        
        # Веса для Query, Key, Value
        self.W_q = np.random.randn(d_model, d_model) * 0.1
        self.W_k = np.random.randn(d_model, d_model) * 0.1
        self.W_v = np.random.randn(d_model, d_model) * 0.1
        self.W_o = np.random.randn(d_model, d_model) * 0.1
    
    def scaled_dot_product_attention(self, Q: np.ndarray, K: np.ndarray, 
                                   V: np.ndarray, mask: Optional[np.ndarray] = None) -> np.ndarray:
        """
        Scaled Dot-Product Attention
        
        Attention(Q,K,V) = softmax(QK^T / √d_k)V
        """
        # Вычисляем attention scores
        scores = np.dot(Q, K.T) / np.sqrt(self.d_k)
        
        # Применяем маску (если есть)
        if mask is not None:
            scores = np.where(mask == 0, -np.inf, scores)
        
        # Softmax
        attention_weights = self.softmax(scores)
        
        # Применяем attention к values
        output = np.dot(attention_weights, V)
        
        return output, attention_weights
    
    def forward(self, x: np.ndarray, mask: Optional[np.ndarray] = None) -> np.ndarray:
        """
        Прямое распространение через Multi-Head Attention
        """
        batch_size, seq_len, d_model = x.shape
        
        # Вычисляем Q, K, V
        Q = np.dot(x, self.W_q)
        K = np.dot(x, self.W_k)
        V = np.dot(x, self.W_v)
        
        # Разделяем на heads
        Q = Q.reshape(batch_size, seq_len, self.n_heads, self.d_k).transpose(0, 2, 1, 3)
        K = K.reshape(batch_size, seq_len, self.n_heads, self.d_k).transpose(0, 2, 1, 3)
        V = V.reshape(batch_size, seq_len, self.n_heads, self.d_k).transpose(0, 2, 1, 3)
        
        # Применяем attention для каждого head
        attention_outputs = []
        for i in range(self.n_heads):
            head_output, _ = self.scaled_dot_product_attention(
                Q[:, i], K[:, i], V[:, i], mask
            )
            attention_outputs.append(head_output)
        
        # Concatenate heads
        concat_output = np.concatenate(attention_outputs, axis=-1)
        
        # Финальная линейная трансформация
        output = np.dot(concat_output, self.W_o)
        
        return output
    
    def softmax(self, x: np.ndarray) -> np.ndarray:
        """Softmax функция"""
        exp_x = np.exp(x - np.max(x, axis=-1, keepdims=True))
        return exp_x / np.sum(exp_x, axis=-1, keepdims=True)

# ==================== BERT-ПОДОБНАЯ МОДЕЛЬ ====================
class SimpleBERT:
    """
    Упрощенная версия BERT для понимания принципов
    """
    
    def __init__(self, vocab_size: int, hidden_size: int, max_seq_len: int, n_heads: int = 8):
        self.vocab_size = vocab_size
        self.hidden_size = hidden_size
        self.max_seq_len = max_seq_len
        self.n_heads = n_heads
        
        # Embedding слои
        self.token_embeddings = np.random.randn(vocab_size, hidden_size) * 0.1
        self.position_embeddings = np.random.randn(max_seq_len, hidden_size) * 0.1
        
        # Multi-Head Attention
        self.attention = MultiHeadAttention(hidden_size, n_heads)
        
        # Feed-Forward Network
        self.ff_W1 = np.random.randn(hidden_size, hidden_size * 4) * 0.1
        self.ff_W2 = np.random.randn(hidden_size * 4, hidden_size) * 0.1
        self.ff_b1 = np.zeros(hidden_size * 4)
        self.ff_b2 = np.zeros(hidden_size)
    
    def get_embeddings(self, token_ids: np.ndarray) -> np.ndarray:
        """
        Получение embeddings для токенов
        """
        seq_len = len(token_ids)
        
        # Token embeddings
        token_emb = self.token_embeddings[token_ids]
        
        # Position embeddings
        position_emb = self.position_embeddings[:seq_len]
        
        # Сумма embeddings
        embeddings = token_emb + position_emb
        
        return embeddings
    
    def feed_forward(self, x: np.ndarray) -> np.ndarray:
        """
        Feed-Forward Network
        """
        # Первый линейный слой + ReLU
        hidden = np.dot(x, self.ff_W1) + self.ff_b1
        hidden = np.maximum(0, hidden)  # ReLU
        
        # Второй линейный слой
        output = np.dot(hidden, self.ff_W2) + self.ff_b2
        
        return output
    
    def forward(self, token_ids: np.ndarray) -> np.ndarray:
        """
        Прямое распространение через BERT
        """
        # Получаем embeddings
        embeddings = self.get_embeddings(token_ids)
        embeddings = embeddings.reshape(1, len(token_ids), self.hidden_size)
        
        # Multi-Head Attention
        attention_output = self.attention.forward(embeddings)
        
        # Residual connection + Layer Norm (упрощенно)
        attention_output = attention_output + embeddings
        
        # Feed-Forward Network
        ff_output = self.feed_forward(attention_output)
        
        # Residual connection + Layer Norm (упрощенно)
        output = ff_output + attention_output
        
        return output.squeeze(0)

# ==================== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ====================
def nlp_examples():
    """
    Практические примеры NLP техник
    """
    
    # Пример 1: N-gram языковая модель
    print("=== N-gram языковая модель ===")
    
    # Обучающие тексты
    training_texts = [
        "The quick brown fox jumps over the lazy dog",
        "A quick brown dog runs in the park",
        "The lazy cat sleeps on the couch",
        "Dogs and cats are pets",
        "The brown fox runs quickly"
    ]
    
    # Создание и обучение модели
    ngram_model = NGramLanguageModel(n=3)
    ngram_model.train(training_texts)
    
    # Генерация текста
    context = ('quick', 'brown')
    generated = ngram_model.generate_text(context, max_length=10)
    print(f"Контекст: {context}")
    print(f"Сгенерированный текст: {generated}")
    
    # Вычисление перплексии
    test_texts = ["The quick brown fox runs"]
    perplexity = ngram_model.calculate_perplexity(test_texts)
    print(f"Перплексия: {perplexity:.2f}")
    
    # Пример 2: TF-IDF
    print("\n=== TF-IDF векторизация ===")
    
    documents = [
        "The cat sat on the mat",
        "The dog ran in the park",
        "Cats and dogs are pets",
        "I love my pet cat",
        "The park has many dogs"
    ]
    
    tfidf = TFIDFVectorizer(max_features=10)
    tfidf_matrix = tfidf.fit_transform(documents)
    
    print(f"Словарь: {list(tfidf.vocabulary.keys())}")
    print(f"Форма TF-IDF матрицы: {tfidf_matrix.shape}")
    print(f"Первый документ (TF-IDF): {tfidf_matrix[0]}")
    
    # Пример 3: Word2Vec
    print("\n=== Word2Vec обучение ===")
    
    w2v = Word2Vec(vector_size=50, window=3, min_count=1, learning_rate=0.025)
    w2v.train(documents, epochs=10)
    
    # Поиск похожих слов
    if 'cat' in w2v.word_vectors:
        similar_words = w2v.get_similar_words('cat', top_k=3)
        print(f"Слова, похожие на 'cat': {similar_words}")
    
    # Пример 4: Attention механизм
    print("\n=== Attention механизм ===")
    
    # Создаем случайные скрытые состояния
    hidden_states = np.random.randn(5, 8)  # 5 временных шагов, 8 признаков
    
    attention = AttentionMechanism(hidden_size=8)
    scores = attention.compute_attention_scores(hidden_states)
    context, weights = attention.apply_attention(hidden_states, scores)
    
    print(f"Attention веса: {weights}")
    print(f"Контекстный вектор: {context}")
    
    return {
        'ngram_perplexity': perplexity,
        'tfidf_matrix': tfidf_matrix,
        'word_vectors': w2v.word_vectors,
        'attention_weights': weights
    }

# ==================== LLM КОНЦЕПЦИИ ====================
class LLMConcepts:
    """
    Ключевые концепции больших языковых моделей
    """
    
    @staticmethod
    def tokenization_strategies():
        """
        Стратегии токенизации для LLM
        """
        return {
            "Word-level": {
                "Описание": "Токенизация по словам",
                "Плюсы": "Простота понимания",
                "Минусы": "Большой словарь, проблемы с OOV"
            },
            
            "Subword (BPE)": {
                "Описание": "Byte-Pair Encoding",
                "Плюсы": "Баланс между размером словаря и покрытием",
                "Минусы": "Сложность реализации"
            },
            
            "Character-level": {
                "Описание": "Токенизация по символам",
                "Плюсы": "Нет проблем с OOV",
                "Минусы": "Длинные последовательности"
            },
            
            "SentencePiece": {
                "Описание": "Универсальная токенизация",
                "Плюсы": "Работает с любыми языками",
                "Минусы": "Требует обучения"
            }
        }
    
    @staticmethod
    def training_paradigms():
        """
        Парадигмы обучения LLM
        """
        return {
            "Autoregressive": {
                "Описание": "Предсказание следующего токена",
                "Примеры": "GPT, PaLM, LLaMA",
                "Особенности": "Однонаправленная генерация"
            },
            
            "Masked Language Modeling": {
                "Описание": "Восстановление замаскированных токенов",
                "Примеры": "BERT, RoBERTa, DeBERTa",
                "Особенности": "Двунаправленный контекст"
            },
            
            "Encoder-Decoder": {
                "Описание": "Последовательность в последовательность",
                "Примеры": "T5, BART, mT5",
                "Особенности": "Универсальность задач"
            },
            
            "Instruction Following": {
                "Описание": "Следование инструкциям",
                "Примеры": "InstructGPT, ChatGPT, Claude",
                "Особенности": "Alignment с человеческими предпочтениями"
            }
        }
    
    @staticmethod
    def scaling_laws():
        """
        Законы масштабирования для LLM
        """
        return {
            "Kaplan et al.": {
                "Формула": "Performance ∝ N^α (где N - параметры)",
                "Вывод": "Больше параметров = лучше качество",
                "Ограничения": "Не учитывает качество данных"
            },
            
            "Chinchilla": {
                "Формула": "Optimal: Parameters ∝ Data^0.5",
                "Вывод": "Важен баланс параметров и данных",
                "Практика": "Меньше параметров, больше данных"
            },
            
            "Emergent Abilities": {
                "Описание": "Способности появляются при определенном размере",
                "Примеры": "In-context learning, chain-of-thought",
                "Порог": "Обычно >100B параметров"
            }
        }

# ==================== ПРАКТИЧЕСКИЕ СОВЕТЫ ====================
nlp_best_practices = {
    "Предобработка текста": [
        "Нормализация Unicode",
        "Обработка пунктуации",
        "Токенизация с учетом домена",
        "Обработка специальных символов"
    ],
    
    "Работа с данными": [
        "Балансировка классов",
        "Аугментация данных",
        "Валидация на разных доменах",
        "Обработка длинных текстов"
    ],
    
    "Выбор модели": [
        "Учитывать размер данных",
        "Баланс качества и скорости",
        "Требования к интерпретируемости",
        "Ограничения по ресурсам"
    ],
    
    "Оценка качества": [
        "Множественные метрики",
        "Анализ ошибок",
        "Тестирование на краевых случаях",
        "Человеческая оценка"
    ]
}

# ==================== СОВРЕМЕННЫЕ АРХИТЕКТУРЫ ====================
modern_architectures = {
    "Transformer": {
        "Год": 2017,
        "Ключевая идея": "Attention is All You Need",
        "Преимущества": "Параллелизация, длинные зависимости",
        "Применения": "Машинный перевод, понимание текста"
    },
    
    "BERT": {
        "Год": 2018,
        "Ключевая идея": "Bidirectional Encoder Representations",
        "Преимущества": "Понимание контекста",
        "Применения": "Классификация, NER, Q&A"
    },
    
    "GPT": {
        "Год": 2018-2023,
        "Ключевая идея": "Generative Pre-training",
        "Преимущества": "Генерация текста, few-shot learning",
        "Применения": "Чат-боты, генерация кода, творчество"
    },
    
    "T5": {
        "Год": 2019,
        "Ключевая идея": "Text-to-Text Transfer Transformer",
        "Преимущества": "Универсальность задач",
        "Применения": "Суммаризация, перевод, Q&A"
    }
}
```

---

*NLP превратилось из инженерии признаков в архитектуру моделей* 🤖📝 