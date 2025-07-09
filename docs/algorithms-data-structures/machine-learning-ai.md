# Machine Learning и AI/LLM 🤖

## Содержание
1. [Основы Machine Learning](#основы-machine-learning)
2. [Нейронные сети](#нейронные-сети)
3. [Deep Learning](#deep-learning)
4. [Natural Language Processing](#natural-language-processing)
5. [Large Language Models](#large-language-models)
6. [Computer Vision](#computer-vision)
7. [Reinforcement Learning](#reinforcement-learning)

---

## Основы Machine Learning

### Supervised Learning
```python
# Линейная регрессия с нуля
import numpy as np
import matplotlib.pyplot as plt

class LinearRegression:
    """Линейная регрессия с градиентным спуском"""
    
    def __init__(self, learning_rate=0.01, iterations=1000):
        self.learning_rate = learning_rate
        self.iterations = iterations
        self.weights = None
        self.bias = None
        self.cost_history = []
    
    def fit(self, X, y):
        """Обучение модели"""
        # Инициализация параметров
        n_samples, n_features = X.shape
        self.weights = np.zeros(n_features)
        self.bias = 0
        
        # Градиентный спуск
        for i in range(self.iterations):
            # Предсказание: y_pred = X * w + b
            y_pred = np.dot(X, self.weights) + self.bias
            
            # Вычисление стоимости (MSE)
            cost = (1 / (2 * n_samples)) * np.sum((y_pred - y) ** 2)
            self.cost_history.append(cost)
            
            # Вычисление градиентов
            dw = (1 / n_samples) * np.dot(X.T, (y_pred - y))
            db = (1 / n_samples) * np.sum(y_pred - y)
            
            # Обновление параметров
            self.weights -= self.learning_rate * dw
            self.bias -= self.learning_rate * db
    
    def predict(self, X):
        """Предсказание"""
        return np.dot(X, self.weights) + self.bias
    
    def r2_score(self, y_true, y_pred):
        """Коэффициент детерминации R²"""
        ss_res = np.sum((y_true - y_pred) ** 2)
        ss_tot = np.sum((y_true - np.mean(y_true)) ** 2)
        return 1 - (ss_res / ss_tot)

# Пример использования
def demo_linear_regression():
    # Генерируем данные
    np.random.seed(42)
    X = np.random.randn(100, 1)
    y = 2 * X.squeeze() + 1 + np.random.randn(100) * 0.1
    
    # Обучение модели
    model = LinearRegression(learning_rate=0.01, iterations=1000)
    model.fit(X, y)
    
    # Предсказание
    y_pred = model.predict(X)
    r2 = model.r2_score(y, y_pred)
    
    print(f"Веса: {model.weights}")
    print(f"Смещение: {model.bias}")
    print(f"R² score: {r2}")
```

### Logistic Regression
```python
class LogisticRegression:
    """Логистическая регрессия для бинарной классификации"""
    
    def __init__(self, learning_rate=0.01, iterations=1000):
        self.learning_rate = learning_rate
        self.iterations = iterations
        self.weights = None
        self.bias = None
    
    def _sigmoid(self, z):
        """Сигмоидная функция активации"""
        # Избегаем переполнения
        z = np.clip(z, -500, 500)
        return 1 / (1 + np.exp(-z))
    
    def fit(self, X, y):
        """Обучение модели"""
        n_samples, n_features = X.shape
        self.weights = np.zeros(n_features)
        self.bias = 0
        
        for i in range(self.iterations):
            # Линейная модель: z = X * w + b
            z = np.dot(X, self.weights) + self.bias
            
            # Предсказание вероятности: p = sigmoid(z)
            y_pred = self._sigmoid(z)
            
            # Логистическая функция потерь
            cost = (-1 / n_samples) * np.sum(y * np.log(y_pred + 1e-8) + 
                                           (1 - y) * np.log(1 - y_pred + 1e-8))
            
            # Градиенты
            dw = (1 / n_samples) * np.dot(X.T, (y_pred - y))
            db = (1 / n_samples) * np.sum(y_pred - y)
            
            # Обновление параметров
            self.weights -= self.learning_rate * dw
            self.bias -= self.learning_rate * db
    
    def predict(self, X):
        """Предсказание классов"""
        z = np.dot(X, self.weights) + self.bias
        y_pred = self._sigmoid(z)
        return [1 if p > 0.5 else 0 for p in y_pred]
    
    def predict_proba(self, X):
        """Предсказание вероятностей"""
        z = np.dot(X, self.weights) + self.bias
        return self._sigmoid(z)
```

### Decision Trees
```python
class DecisionTree:
    """Простое дерево решений"""
    
    def __init__(self, max_depth=5, min_samples_split=2):
        self.max_depth = max_depth
        self.min_samples_split = min_samples_split
        self.tree = None
    
    def _gini_impurity(self, y):
        """Индекс Джини для измерения неопределенности"""
        classes = np.unique(y)
        impurity = 1.0
        
        for cls in classes:
            p = np.sum(y == cls) / len(y)
            impurity -= p ** 2
        
        return impurity
    
    def _information_gain(self, X_column, y, threshold):
        """Информационный выигрыш от разделения"""
        left_mask = X_column <= threshold
        right_mask = ~left_mask
        
        if np.sum(left_mask) == 0 or np.sum(right_mask) == 0:
            return 0
        
        n = len(y)
        left_weight = np.sum(left_mask) / n
        right_weight = np.sum(right_mask) / n
        
        # Взвешенная сумма примесей после разделения
        gain = (self._gini_impurity(y) - 
                left_weight * self._gini_impurity(y[left_mask]) - 
                right_weight * self._gini_impurity(y[right_mask]))
        
        return gain
    
    def _best_split(self, X, y):
        """Поиск лучшего разделения"""
        best_gain = 0
        best_feature = None
        best_threshold = None
        
        n_features = X.shape[1]
        
        for feature_idx in range(n_features):
            column = X[:, feature_idx]
            thresholds = np.unique(column)
            
            for threshold in thresholds:
                gain = self._information_gain(column, y, threshold)
                
                if gain > best_gain:
                    best_gain = gain
                    best_feature = feature_idx
                    best_threshold = threshold
        
        return best_feature, best_threshold
    
    def _build_tree(self, X, y, depth=0):
        """Рекурсивное построение дерева"""
        n_samples = len(y)
        
        # Условия остановки
        if (depth >= self.max_depth or 
            n_samples < self.min_samples_split or 
            len(np.unique(y)) == 1):
            
            # Листовой узел - возвращаем наиболее частый класс
            return np.bincount(y).argmax()
        
        # Поиск лучшего разделения
        feature, threshold = self._best_split(X, y)
        
        if feature is None:
            return np.bincount(y).argmax()
        
        # Разделение данных
        left_mask = X[:, feature] <= threshold
        right_mask = ~left_mask
        
        # Рекурсивное построение поддеревьев
        node = {
            'feature': feature,
            'threshold': threshold,
            'left': self._build_tree(X[left_mask], y[left_mask], depth + 1),
            'right': self._build_tree(X[right_mask], y[right_mask], depth + 1)
        }
        
        return node
    
    def fit(self, X, y):
        """Обучение дерева"""
        self.tree = self._build_tree(X, y)
    
    def _predict_sample(self, sample):
        """Предсказание для одного образца"""
        node = self.tree
        
        while isinstance(node, dict):
            if sample[node['feature']] <= node['threshold']:
                node = node['left']
            else:
                node = node['right']
        
        return node
    
    def predict(self, X):
        """Предсказание для массива образцов"""
        return [self._predict_sample(sample) for sample in X]
```

---

## Нейронные сети

### Персептрон
```python
class Perceptron:
    """Простой персептрон для бинарной классификации"""
    
    def __init__(self, learning_rate=0.01, epochs=1000):
        self.learning_rate = learning_rate
        self.epochs = epochs
        self.weights = None
        self.bias = None
    
    def _activation(self, z):
        """Ступенчатая функция активации"""
        return np.where(z >= 0, 1, 0)
    
    def fit(self, X, y):
        """Обучение персептрона"""
        n_samples, n_features = X.shape
        
        # Инициализация весов
        self.weights = np.random.normal(0, 0.01, n_features)
        self.bias = 0
        
        for epoch in range(self.epochs):
            for i in range(n_samples):
                # Прямое распространение
                z = np.dot(X[i], self.weights) + self.bias
                y_pred = self._activation(z)
                
                # Обновление весов (правило обучения персептрона)
                error = y[i] - y_pred
                self.weights += self.learning_rate * error * X[i]
                self.bias += self.learning_rate * error
    
    def predict(self, X):
        """Предсказание"""
        z = np.dot(X, self.weights) + self.bias
        return self._activation(z)
```

### Многослойный персептрон
```python
class MLP:
    """Многослойный персептрон с обратным распространением"""
    
    def __init__(self, layers, learning_rate=0.01, epochs=1000):
        self.layers = layers  # [input_size, hidden_size, ..., output_size]
        self.learning_rate = learning_rate
        self.epochs = epochs
        self.weights = []
        self.biases = []
        
        # Инициализация весов и смещений
        for i in range(len(layers) - 1):
            # Xavier инициализация
            w = np.random.randn(layers[i], layers[i+1]) * np.sqrt(2.0 / layers[i])
            b = np.zeros((1, layers[i+1]))
            self.weights.append(w)
            self.biases.append(b)
    
    def _sigmoid(self, z):
        """Сигмоидная функция активации"""
        return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
    
    def _sigmoid_derivative(self, z):
        """Производная сигмоидной функции"""
        return z * (1 - z)
    
    def _relu(self, z):
        """ReLU функция активации"""
        return np.maximum(0, z)
    
    def _relu_derivative(self, z):
        """Производная ReLU"""
        return np.where(z > 0, 1, 0)
    
    def forward(self, X):
        """Прямое распространение"""
        activations = [X]
        
        for i in range(len(self.weights)):
            z = np.dot(activations[-1], self.weights[i]) + self.biases[i]
            
            if i < len(self.weights) - 1:
                # Скрытые слои - ReLU
                activation = self._relu(z)
            else:
                # Выходной слой - сигмоида
                activation = self._sigmoid(z)
            
            activations.append(activation)
        
        return activations
    
    def backward(self, X, y, activations):
        """Обратное распространение"""
        m = X.shape[0]
        
        # Вычисляем градиенты для весов и смещений
        dW = []
        dB = []
        
        # Ошибка выходного слоя
        delta = activations[-1] - y
        
        for i in range(len(self.weights) - 1, -1, -1):
            # Градиенты для весов и смещений
            dw = (1/m) * np.dot(activations[i].T, delta)
            db = (1/m) * np.sum(delta, axis=0, keepdims=True)
            
            dW.insert(0, dw)
            dB.insert(0, db)
            
            if i > 0:
                # Распространение ошибки в предыдущий слой
                delta = np.dot(delta, self.weights[i].T)
                
                # Умножаем на производную функции активации
                if i > 0:  # Скрытые слои используют ReLU
                    delta *= self._relu_derivative(activations[i])
        
        return dW, dB
    
    def fit(self, X, y):
        """Обучение сети"""
        for epoch in range(self.epochs):
            # Прямое распространение
            activations = self.forward(X)
            
            # Обратное распространение
            dW, dB = self.backward(X, y, activations)
            
            # Обновление весов и смещений
            for i in range(len(self.weights)):
                self.weights[i] -= self.learning_rate * dW[i]
                self.biases[i] -= self.learning_rate * dB[i]
            
            # Вычисление потерь
            if epoch % 100 == 0:
                loss = np.mean((activations[-1] - y) ** 2)
                print(f"Epoch {epoch}, Loss: {loss:.4f}")
    
    def predict(self, X):
        """Предсказание"""
        activations = self.forward(X)
        return activations[-1]
```

---

## Deep Learning

### Convolutional Neural Network
```python
class ConvolutionalLayer:
    """Слой свертки для CNN"""
    
    def __init__(self, input_channels, output_channels, kernel_size, stride=1, padding=0):
        self.input_channels = input_channels
        self.output_channels = output_channels
        self.kernel_size = kernel_size
        self.stride = stride
        self.padding = padding
        
        # Инициализация весов фильтров
        self.weights = np.random.randn(
            output_channels, input_channels, kernel_size, kernel_size
        ) * 0.1
        self.bias = np.zeros(output_channels)
    
    def forward(self, X):
        """Прямое распространение свертки"""
        batch_size, channels, height, width = X.shape
        
        # Добавляем padding
        if self.padding > 0:
            X = np.pad(X, ((0, 0), (0, 0), (self.padding, self.padding), 
                          (self.padding, self.padding)), mode='constant')
        
        # Вычисляем размеры выходного тензора
        out_height = (height + 2 * self.padding - self.kernel_size) // self.stride + 1
        out_width = (width + 2 * self.padding - self.kernel_size) // self.stride + 1
        
        # Выходной тензор
        output = np.zeros((batch_size, self.output_channels, out_height, out_width))
        
        # Выполняем свертку
        for b in range(batch_size):
            for f in range(self.output_channels):
                for i in range(out_height):
                    for j in range(out_width):
                        h_start = i * self.stride
                        h_end = h_start + self.kernel_size
                        w_start = j * self.stride
                        w_end = w_start + self.kernel_size
                        
                        # Элементарная свертка
                        region = X[b, :, h_start:h_end, w_start:w_end]
                        output[b, f, i, j] = np.sum(region * self.weights[f]) + self.bias[f]
        
        return output
    
    def relu_activation(self, x):
        """ReLU активация"""
        return np.maximum(0, x)

class MaxPoolingLayer:
    """Слой максимального пулинга"""
    
    def __init__(self, pool_size=2, stride=2):
        self.pool_size = pool_size
        self.stride = stride
    
    def forward(self, X):
        """Прямое распространение пулинга"""
        batch_size, channels, height, width = X.shape
        
        out_height = (height - self.pool_size) // self.stride + 1
        out_width = (width - self.pool_size) // self.stride + 1
        
        output = np.zeros((batch_size, channels, out_height, out_width))
        
        for b in range(batch_size):
            for c in range(channels):
                for i in range(out_height):
                    for j in range(out_width):
                        h_start = i * self.stride
                        h_end = h_start + self.pool_size
                        w_start = j * self.stride
                        w_end = w_start + self.pool_size
                        
                        # Максимальное значение в окне
                        pool_region = X[b, c, h_start:h_end, w_start:w_end]
                        output[b, c, i, j] = np.max(pool_region)
        
        return output

class SimpleCNN:
    """Простая сверточная нейронная сеть"""
    
    def __init__(self):
        # Архитектура: Conv -> ReLU -> MaxPool -> Flatten -> Dense
        self.conv1 = ConvolutionalLayer(1, 32, 3, padding=1)
        self.pool1 = MaxPoolingLayer(2, 2)
        self.flatten_size = None
        self.dense_weights = None
        self.dense_bias = None
    
    def forward(self, X):
        """Прямое распространение через всю сеть"""
        # Сверточный слой
        conv_out = self.conv1.forward(X)
        relu_out = self.conv1.relu_activation(conv_out)
        
        # Пулинг
        pool_out = self.pool1.forward(relu_out)
        
        # Выпрямление (flatten)
        batch_size = pool_out.shape[0]
        flattened = pool_out.reshape(batch_size, -1)
        
        # Инициализация полносвязного слоя при первом запуске
        if self.dense_weights is None:
            self.flatten_size = flattened.shape[1]
            self.dense_weights = np.random.randn(self.flatten_size, 10) * 0.1
            self.dense_bias = np.zeros(10)
        
        # Полносвязный слой
        dense_out = np.dot(flattened, self.dense_weights) + self.dense_bias
        
        # Softmax для классификации
        exp_scores = np.exp(dense_out - np.max(dense_out, axis=1, keepdims=True))
        probabilities = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)
        
        return probabilities
```

---

## Natural Language Processing

### Bag of Words
```python
class BagOfWords:
    """Модель мешка слов"""
    
    def __init__(self, max_features=1000):
        self.max_features = max_features
        self.vocabulary = {}
        self.word_to_index = {}
        self.index_to_word = {}
    
    def fit(self, documents):
        """Построение словаря"""
        # Подсчет частоты слов
        word_counts = {}
        for doc in documents:
            words = doc.lower().split()
            for word in words:
                word_counts[word] = word_counts.get(word, 0) + 1
        
        # Выбираем наиболее частые слова
        sorted_words = sorted(word_counts.items(), key=lambda x: x[1], reverse=True)
        top_words = sorted_words[:self.max_features]
        
        # Создаем словарь
        for idx, (word, count) in enumerate(top_words):
            self.word_to_index[word] = idx
            self.index_to_word[idx] = word
            self.vocabulary[word] = count
    
    def transform(self, documents):
        """Преобразование документов в векторы"""
        vectors = []
        
        for doc in documents:
            vector = np.zeros(len(self.word_to_index))
            words = doc.lower().split()
            
            for word in words:
                if word in self.word_to_index:
                    vector[self.word_to_index[word]] += 1
            
            vectors.append(vector)
        
        return np.array(vectors)
    
    def fit_transform(self, documents):
        """Обучение и преобразование"""
        self.fit(documents)
        return self.transform(documents)

class TFIDFVectorizer:
    """TF-IDF векторизация"""
    
    def __init__(self, max_features=1000):
        self.max_features = max_features
        self.vocabulary = {}
        self.idf_values = {}
    
    def fit(self, documents):
        """Вычисление IDF значений"""
        # Подсчет частоты слов и документов
        word_doc_count = {}
        word_total_count = {}
        
        for doc in documents:
            words = set(doc.lower().split())  # Уникальные слова в документе
            for word in words:
                word_doc_count[word] = word_doc_count.get(word, 0) + 1
        
        # Подсчет общей частоты слов
        for doc in documents:
            words = doc.lower().split()
            for word in words:
                word_total_count[word] = word_total_count.get(word, 0) + 1
        
        # Выбираем наиболее частые слова
        sorted_words = sorted(word_total_count.items(), key=lambda x: x[1], reverse=True)
        top_words = sorted_words[:self.max_features]
        
        # Создаем словарь и вычисляем IDF
        for idx, (word, _) in enumerate(top_words):
            self.vocabulary[word] = idx
            # IDF = log(N / df), где N - общее число документов, df - число документов с данным словом
            self.idf_values[word] = np.log(len(documents) / word_doc_count[word])
    
    def transform(self, documents):
        """Преобразование в TF-IDF векторы"""
        vectors = []
        
        for doc in documents:
            vector = np.zeros(len(self.vocabulary))
            words = doc.lower().split()
            word_counts = {}
            
            # Подсчет частоты слов в документе (TF)
            for word in words:
                if word in self.vocabulary:
                    word_counts[word] = word_counts.get(word, 0) + 1
            
            # Вычисление TF-IDF
            for word, count in word_counts.items():
                if word in self.vocabulary:
                    tf = count / len(words)  # Term Frequency
                    idf = self.idf_values[word]  # Inverse Document Frequency
                    vector[self.vocabulary[word]] = tf * idf
            
            vectors.append(vector)
        
        return np.array(vectors)
    
    def fit_transform(self, documents):
        """Обучение и преобразование"""
        self.fit(documents)
        return self.transform(documents)
```

### Word Embeddings
```python
class Word2Vec:
    """Упрощенная реализация Word2Vec (Skip-gram)"""
    
    def __init__(self, vector_size=100, window=5, min_count=1, epochs=5):
        self.vector_size = vector_size
        self.window = window
        self.min_count = min_count
        self.epochs = epochs
        self.word_to_index = {}
        self.index_to_word = {}
        self.vocabulary = {}
        self.embeddings = None
    
    def build_vocabulary(self, sentences):
        """Построение словаря"""
        word_counts = {}
        
        for sentence in sentences:
            words = sentence.lower().split()
            for word in words:
                word_counts[word] = word_counts.get(word, 0) + 1
        
        # Фильтруем слова по частоте
        filtered_words = {word: count for word, count in word_counts.items() 
                         if count >= self.min_count}
        
        # Создаем индексы
        for idx, word in enumerate(filtered_words.keys()):
            self.word_to_index[word] = idx
            self.index_to_word[idx] = word
            self.vocabulary[word] = filtered_words[word]
    
    def generate_training_data(self, sentences):
        """Генерация обучающих пар (центральное слово, контекстное слово)"""
        training_data = []
        
        for sentence in sentences:
            words = sentence.lower().split()
            # Фильтруем слова, которые есть в словаре
            words = [word for word in words if word in self.word_to_index]
            
            for i, center_word in enumerate(words):
                # Определяем контекстное окно
                start = max(0, i - self.window)
                end = min(len(words), i + self.window + 1)
                
                for j in range(start, end):
                    if i != j:  # Пропускаем само центральное слово
                        context_word = words[j]
                        training_data.append((center_word, context_word))
        
        return training_data
    
    def train(self, sentences, learning_rate=0.01):
        """Обучение модели"""
        self.build_vocabulary(sentences)
        vocab_size = len(self.vocabulary)
        
        # Инициализация матриц весов
        # Матрица входных весов (центральные слова)
        self.embeddings = np.random.uniform(-0.5/self.vector_size, 0.5/self.vector_size,
                                          (vocab_size, self.vector_size))
        
        # Матрица выходных весов (контекстные слова)
        output_weights = np.random.uniform(-0.5/self.vector_size, 0.5/self.vector_size,
                                         (vocab_size, self.vector_size))
        
        training_data = self.generate_training_data(sentences)
        
        for epoch in range(self.epochs):
            loss = 0
            for center_word, context_word in training_data:
                center_idx = self.word_to_index[center_word]
                context_idx = self.word_to_index[context_word]
                
                # Прямое распространение
                h = self.embeddings[center_idx]  # Скрытый слой
                u = np.dot(output_weights, h)    # Выходной слой
                
                # Softmax
                exp_u = np.exp(u - np.max(u))
                y_pred = exp_u / np.sum(exp_u)
                
                # Целевой вектор (one-hot)
                y_true = np.zeros(vocab_size)
                y_true[context_idx] = 1
                
                # Потеря
                loss += -np.log(y_pred[context_idx] + 1e-10)
                
                # Обратное распространение
                error = y_pred - y_true
                
                # Обновление весов
                grad_output = np.outer(error, h)
                grad_hidden = np.dot(output_weights.T, error)
                
                output_weights -= learning_rate * grad_output
                self.embeddings[center_idx] -= learning_rate * grad_hidden
            
            if epoch % 1 == 0:
                print(f"Epoch {epoch}, Loss: {loss/len(training_data):.4f}")
    
    def get_vector(self, word):
        """Получение вектора слова"""
        if word in self.word_to_index:
            return self.embeddings[self.word_to_index[word]]
        return None
    
    def most_similar(self, word, topn=5):
        """Поиск наиболее похожих слов"""
        if word not in self.word_to_index:
            return []
        
        word_vector = self.get_vector(word)
        similarities = []
        
        for other_word in self.vocabulary:
            if other_word != word:
                other_vector = self.get_vector(other_word)
                # Косинусное сходство
                similarity = np.dot(word_vector, other_vector) / (
                    np.linalg.norm(word_vector) * np.linalg.norm(other_vector)
                )
                similarities.append((other_word, similarity))
        
        # Сортируем по убыванию сходства
        similarities.sort(key=lambda x: x[1], reverse=True)
        return similarities[:topn]
```

---

## Large Language Models

### Transformer Architecture
```python
class MultiHeadAttention:
    """Механизм множественного внимания"""
    
    def __init__(self, d_model, num_heads):
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        # Матрицы для Q, K, V
        self.W_q = np.random.randn(d_model, d_model) * 0.1
        self.W_k = np.random.randn(d_model, d_model) * 0.1
        self.W_v = np.random.randn(d_model, d_model) * 0.1
        self.W_o = np.random.randn(d_model, d_model) * 0.1
    
    def scaled_dot_product_attention(self, Q, K, V, mask=None):
        """Внимание с масштабированием"""
        # Вычисляем веса внимания
        scores = np.matmul(Q, K.transpose(-2, -1)) / np.sqrt(self.d_k)
        
        # Применяем маску (для предотвращения внимания к будущим токенам)
        if mask is not None:
            scores = np.where(mask == 0, -1e9, scores)
        
        # Softmax для нормализации весов
        attention_weights = self.softmax(scores)
        
        # Применяем веса к значениям
        output = np.matmul(attention_weights, V)
        
        return output, attention_weights
    
    def softmax(self, x):
        """Softmax функция"""
        exp_x = np.exp(x - np.max(x, axis=-1, keepdims=True))
        return exp_x / np.sum(exp_x, axis=-1, keepdims=True)
    
    def forward(self, query, key, value, mask=None):
        """Прямое распространение"""
        batch_size, seq_len, d_model = query.shape
        
        # Линейные преобразования
        Q = np.matmul(query, self.W_q)
        K = np.matmul(key, self.W_k)
        V = np.matmul(value, self.W_v)
        
        # Разбиваем на головы внимания
        Q = Q.reshape(batch_size, seq_len, self.num_heads, self.d_k).transpose(0, 2, 1, 3)
        K = K.reshape(batch_size, seq_len, self.num_heads, self.d_k).transpose(0, 2, 1, 3)
        V = V.reshape(batch_size, seq_len, self.num_heads, self.d_k).transpose(0, 2, 1, 3)
        
        # Применяем внимание
        attention_output, attention_weights = self.scaled_dot_product_attention(Q, K, V, mask)
        
        # Объединяем головы
        attention_output = attention_output.transpose(0, 2, 1, 3).reshape(
            batch_size, seq_len, d_model
        )
        
        # Финальная линейная трансформация
        output = np.matmul(attention_output, self.W_o)
        
        return output, attention_weights

class TransformerBlock:
    """Блок трансформера"""
    
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_ff = d_ff
        self.dropout = dropout
        
        # Слой множественного внимания
        self.multi_head_attention = MultiHeadAttention(d_model, num_heads)
        
        # Feed-forward сеть
        self.ff_w1 = np.random.randn(d_model, d_ff) * 0.1
        self.ff_b1 = np.zeros(d_ff)
        self.ff_w2 = np.random.randn(d_ff, d_model) * 0.1
        self.ff_b2 = np.zeros(d_model)
        
        # Параметры для нормализации слоев
        self.ln1_gamma = np.ones(d_model)
        self.ln1_beta = np.zeros(d_model)
        self.ln2_gamma = np.ones(d_model)
        self.ln2_beta = np.zeros(d_model)
    
    def layer_norm(self, x, gamma, beta, epsilon=1e-6):
        """Нормализация слоя"""
        mean = np.mean(x, axis=-1, keepdims=True)
        var = np.var(x, axis=-1, keepdims=True)
        normalized = (x - mean) / np.sqrt(var + epsilon)
        return gamma * normalized + beta
    
    def feed_forward(self, x):
        """Feed-forward сеть"""
        # Первый слой с ReLU
        h1 = np.maximum(0, np.matmul(x, self.ff_w1) + self.ff_b1)
        
        # Второй слой
        h2 = np.matmul(h1, self.ff_w2) + self.ff_b2
        
        return h2
    
    def forward(self, x, mask=None):
        """Прямое распространение через блок"""
        # Множественное внимание с остаточной связью
        attn_output, attn_weights = self.multi_head_attention.forward(x, x, x, mask)
        x = self.layer_norm(x + attn_output, self.ln1_gamma, self.ln1_beta)
        
        # Feed-forward с остаточной связью
        ff_output = self.feed_forward(x)
        x = self.layer_norm(x + ff_output, self.ln2_gamma, self.ln2_beta)
        
        return x, attn_weights

class SimpleTransformer:
    """Упрощенная модель трансформера"""
    
    def __init__(self, vocab_size, d_model=512, num_heads=8, num_layers=6, d_ff=2048, max_seq_len=512):
        self.vocab_size = vocab_size
        self.d_model = d_model
        self.num_heads = num_heads
        self.num_layers = num_layers
        self.d_ff = d_ff
        self.max_seq_len = max_seq_len
        
        # Эмбеддинги токенов
        self.token_embedding = np.random.randn(vocab_size, d_model) * 0.1
        
        # Позиционные эмбеддинги
        self.position_embedding = self.create_positional_encoding(max_seq_len, d_model)
        
        # Слои трансформера
        self.transformer_blocks = [
            TransformerBlock(d_model, num_heads, d_ff) for _ in range(num_layers)
        ]
        
        # Выходной слой
        self.output_projection = np.random.randn(d_model, vocab_size) * 0.1
    
    def create_positional_encoding(self, max_len, d_model):
        """Создание позиционных эмбеддингов"""
        pe = np.zeros((max_len, d_model))
        position = np.arange(0, max_len).reshape(-1, 1)
        
        div_term = np.exp(np.arange(0, d_model, 2) * -(np.log(10000.0) / d_model))
        
        pe[:, 0::2] = np.sin(position * div_term)
        pe[:, 1::2] = np.cos(position * div_term)
        
        return pe
    
    def forward(self, input_ids, mask=None):
        """Прямое распространение"""
        seq_len = input_ids.shape[1]
        
        # Получаем эмбеддинги токенов
        token_emb = self.token_embedding[input_ids]
        
        # Добавляем позиционные эмбеддинги
        pos_emb = self.position_embedding[:seq_len]
        x = token_emb + pos_emb
        
        # Проходим через слои трансформера
        attention_weights = []
        for transformer_block in self.transformer_blocks:
            x, attn_weights = transformer_block.forward(x, mask)
            attention_weights.append(attn_weights)
        
        # Выходной слой
        logits = np.matmul(x, self.output_projection)
        
        return logits, attention_weights
    
    def generate_text(self, input_ids, max_length=50, temperature=1.0):
        """Генерация текста"""
        generated = input_ids.copy()
        
        for _ in range(max_length):
            # Получаем предсказания для последнего токена
            logits, _ = self.forward(generated)
            next_token_logits = logits[0, -1, :] / temperature
            
            # Применяем softmax
            exp_logits = np.exp(next_token_logits - np.max(next_token_logits))
            probs = exp_logits / np.sum(exp_logits)
            
            # Сэмплируем следующий токен
            next_token = np.random.choice(len(probs), p=probs)
            
            # Добавляем к последовательности
            generated = np.concatenate([generated, [[next_token]]], axis=1)
        
        return generated
```

---

## Практические примеры

### Обработка текста с BERT-подобной архитектурой
```python
class TextClassifier:
    """Классификатор текста на основе трансформера"""
    
    def __init__(self, vocab_size, num_classes, d_model=256, num_heads=8, num_layers=4):
        self.vocab_size = vocab_size
        self.num_classes = num_classes
        
        # Трансформер для извлечения признаков
        self.transformer = SimpleTransformer(
            vocab_size=vocab_size,
            d_model=d_model,
            num_heads=num_heads,
            num_layers=num_layers
        )
        
        # Классификационная голова
        self.classifier = np.random.randn(d_model, num_classes) * 0.1
        self.classifier_bias = np.zeros(num_classes)
    
    def forward(self, input_ids, labels=None):
        """Прямое распространение"""
        # Получаем представления от трансформера
        hidden_states, attention_weights = self.transformer.forward(input_ids)
        
        # Используем [CLS] токен (первый токен) для классификации
        cls_hidden = hidden_states[:, 0, :]
        
        # Классификация
        logits = np.matmul(cls_hidden, self.classifier) + self.classifier_bias
        
        if labels is not None:
            # Вычисляем потери
            loss = self.cross_entropy_loss(logits, labels)
            return logits, loss
        
        return logits
    
    def cross_entropy_loss(self, logits, labels):
        """Функция потерь для классификации"""
        batch_size = logits.shape[0]
        
        # Softmax
        exp_logits = np.exp(logits - np.max(logits, axis=1, keepdims=True))
        probs = exp_logits / np.sum(exp_logits, axis=1, keepdims=True)
        
        # Перекрестная энтропия
        log_probs = np.log(probs + 1e-10)
        loss = -np.sum(log_probs[range(batch_size), labels]) / batch_size
        
        return loss
    
    def predict(self, input_ids):
        """Предсказание класса"""
        logits = self.forward(input_ids)
        return np.argmax(logits, axis=1)
```

## Связь с другими разделами

- **[Алгоритмы](./algorithms.md)** - Алгоритмы оптимизации для ML
- **[Структуры данных](./data-structures.md)** - Эффективные структуры для ML
- **[CS основы](./computer-science-fundamentals.md)** - Параллельные вычисления для ML
- **[Математические основы](./mathematics-foundations.md)** - Математика для ML

---

*Следующий раздел: [Математические основы →](./mathematics-foundations.md)* 