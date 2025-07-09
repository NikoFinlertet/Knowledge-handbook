# Neural Networks & Deep Learning

> **Нейронные сети и глубокое обучение**
>
> От простого перцептрона до современных архитектур

## 🧠 Основы нейронных сетей

### Математические основы

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification, make_circles
from sklearn.preprocessing import StandardScaler

# ==================== БАЗОВЫЙ ПЕРЦЕПТРОН ====================
class Perceptron:
    """
    Простейшая нейронная сеть - перцептрон
    
    Может решать только линейно разделимые задачи
    """
    
    def __init__(self, learning_rate=0.1, max_epochs=100):
        self.learning_rate = learning_rate
        self.max_epochs = max_epochs
        self.weights = None
        self.bias = None
        self.errors = []
    
    def activation_function(self, x):
        """
        Ступенчатая функция активации
        
        Выход: 1 если x >= 0, иначе 0
        """
        return np.where(x >= 0, 1, 0)
    
    def fit(self, X, y):
        """
        Обучение перцептрона
        
        Правило обучения: w = w + α * (y - ŷ) * x
        """
        # Инициализация весов
        n_features = X.shape[1]
        self.weights = np.random.normal(0, 0.1, n_features)
        self.bias = 0
        
        for epoch in range(self.max_epochs):
            epoch_errors = 0
            
            for i in range(len(X)):
                # Прямое распространение
                linear_output = np.dot(X[i], self.weights) + self.bias
                prediction = self.activation_function(linear_output)
                
                # Вычисление ошибки
                error = y[i] - prediction
                
                # Обновление весов и bias
                if error != 0:
                    self.weights += self.learning_rate * error * X[i]
                    self.bias += self.learning_rate * error
                    epoch_errors += 1
            
            self.errors.append(epoch_errors)
            
            # Если нет ошибок, обучение завершено
            if epoch_errors == 0:
                break
    
    def predict(self, X):
        """Предсказание для новых данных"""
        linear_output = np.dot(X, self.weights) + self.bias
        return self.activation_function(linear_output)

# ==================== МНОГОСЛОЙНЫЙ ПЕРЦЕПТРОН ====================
class MLP:
    """
    Многослойный перцептрон с обратным распространением ошибки
    
    Может решать нелинейные задачи
    """
    
    def __init__(self, layer_sizes, learning_rate=0.1, max_epochs=1000):
        self.layer_sizes = layer_sizes
        self.learning_rate = learning_rate
        self.max_epochs = max_epochs
        self.weights = []
        self.biases = []
        self.initialize_weights()
    
    def initialize_weights(self):
        """
        Инициализация весов методом Xavier
        
        Помогает избежать проблем с затуханием/взрывом градиентов
        """
        for i in range(len(self.layer_sizes) - 1):
            # Xavier инициализация
            limit = np.sqrt(6 / (self.layer_sizes[i] + self.layer_sizes[i + 1]))
            weight = np.random.uniform(-limit, limit, 
                                     (self.layer_sizes[i], self.layer_sizes[i + 1]))
            bias = np.zeros(self.layer_sizes[i + 1])
            
            self.weights.append(weight)
            self.biases.append(bias)
    
    def sigmoid(self, x):
        """
        Сигмоидная функция активации
        
        f(x) = 1 / (1 + e^(-x))
        Выход: (0, 1)
        """
        # Защита от переполнения
        x = np.clip(x, -500, 500)
        return 1 / (1 + np.exp(-x))
    
    def sigmoid_derivative(self, x):
        """
        Производная сигмоидной функции
        
        f'(x) = f(x) * (1 - f(x))
        """
        return x * (1 - x)
    
    def relu(self, x):
        """
        ReLU функция активации
        
        f(x) = max(0, x)
        Решает проблему затухания градиентов
        """
        return np.maximum(0, x)
    
    def relu_derivative(self, x):
        """
        Производная ReLU
        
        f'(x) = 1 если x > 0, иначе 0
        """
        return np.where(x > 0, 1, 0)
    
    def forward_propagation(self, X):
        """
        Прямое распространение
        
        Вычисляет выход сети для входных данных
        """
        activations = [X]
        
        for i in range(len(self.weights)):
            # Линейная комбинация
            z = np.dot(activations[-1], self.weights[i]) + self.biases[i]
            
            # Функция активации
            if i == len(self.weights) - 1:  # Выходной слой
                a = self.sigmoid(z)
            else:  # Скрытые слои
                a = self.relu(z)
            
            activations.append(a)
        
        return activations
    
    def backward_propagation(self, X, y, activations):
        """
        Обратное распространение ошибки
        
        Вычисляет градиенты для обновления весов
        """
        m = X.shape[0]  # Количество примеров
        
        # Инициализация градиентов
        weight_gradients = [np.zeros_like(w) for w in self.weights]
        bias_gradients = [np.zeros_like(b) for b in self.biases]
        
        # Ошибка выходного слоя
        output_error = activations[-1] - y
        
        # Обратное распространение
        errors = [output_error]
        
        for i in range(len(self.weights) - 1, -1, -1):
            # Градиент весов
            weight_gradients[i] = np.dot(activations[i].T, errors[-1]) / m
            
            # Градиент bias
            bias_gradients[i] = np.mean(errors[-1], axis=0)
            
            # Ошибка предыдущего слоя
            if i > 0:
                if i == len(self.weights) - 1:  # От выходного слоя
                    delta = errors[-1] * self.sigmoid_derivative(activations[i + 1])
                else:  # От скрытого слоя
                    delta = errors[-1] * self.relu_derivative(activations[i + 1])
                
                error = np.dot(delta, self.weights[i].T)
                errors.append(error)
        
        return weight_gradients, bias_gradients
    
    def fit(self, X, y, validation_data=None):
        """
        Обучение нейронной сети
        """
        # Преобразование y в правильный формат
        if len(y.shape) == 1:
            y = y.reshape(-1, 1)
        
        train_losses = []
        val_losses = []
        
        for epoch in range(self.max_epochs):
            # Прямое распространение
            activations = self.forward_propagation(X)
            
            # Обратное распространение
            weight_gradients, bias_gradients = self.backward_propagation(X, y, activations)
            
            # Обновление весов
            for i in range(len(self.weights)):
                self.weights[i] -= self.learning_rate * weight_gradients[i]
                self.biases[i] -= self.learning_rate * bias_gradients[i]
            
            # Вычисление потерь
            train_loss = self.compute_loss(y, activations[-1])
            train_losses.append(train_loss)
            
            # Валидационные потери
            if validation_data:
                X_val, y_val = validation_data
                val_predictions = self.predict(X_val)
                val_loss = self.compute_loss(y_val.reshape(-1, 1), val_predictions)
                val_losses.append(val_loss)
            
            # Логирование
            if epoch % 100 == 0:
                print(f"Epoch {epoch}, Train Loss: {train_loss:.4f}")
        
        return train_losses, val_losses
    
    def compute_loss(self, y_true, y_pred):
        """
        Бинарная кросс-энтропия
        
        Loss = -1/m * Σ(y*log(ŷ) + (1-y)*log(1-ŷ))
        """
        # Избегаем log(0)
        epsilon = 1e-15
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        
        return -np.mean(y_true * np.log(y_pred) + (1 - y_true) * np.log(1 - y_pred))
    
    def predict(self, X):
        """Предсказание для новых данных"""
        activations = self.forward_propagation(X)
        return activations[-1]

# ==================== ФУНКЦИИ АКТИВАЦИИ ====================
class ActivationFunctions:
    """
    Различные функции активации и их свойства
    """
    
    @staticmethod
    def sigmoid(x):
        """
        Сигмоидная функция
        
        Плюсы: гладкая, дифференцируемая
        Минусы: проблема затухания градиентов
        """
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
    
    @staticmethod
    def tanh(x):
        """
        Гиперболический тангенс
        
        Плюсы: выход в диапазоне [-1, 1], симметрична
        Минусы: все еще проблема затухания градиентов
        """
        return np.tanh(x)
    
    @staticmethod
    def relu(x):
        """
        Rectified Linear Unit
        
        Плюсы: простота, решает проблему затухания градиентов
        Минусы: может "умереть" (dead ReLU problem)
        """
        return np.maximum(0, x)
    
    @staticmethod
    def leaky_relu(x, alpha=0.01):
        """
        Leaky ReLU
        
        Решает проблему "мертвых" нейронов
        """
        return np.where(x > 0, x, alpha * x)
    
    @staticmethod
    def elu(x, alpha=1.0):
        """
        Exponential Linear Unit
        
        Плюсы: гладкая, отрицательные значения
        """
        return np.where(x > 0, x, alpha * (np.exp(x) - 1))
    
    @staticmethod
    def swish(x):
        """
        Swish функция (Self-Gated)
        
        f(x) = x * sigmoid(x)
        Показывает хорошие результаты в глубоких сетях
        """
        return x * ActivationFunctions.sigmoid(x)
    
    @staticmethod
    def softmax(x):
        """
        Softmax для многоклассовой классификации
        
        Преобразует логиты в вероятности
        """
        exp_x = np.exp(x - np.max(x, axis=1, keepdims=True))
        return exp_x / np.sum(exp_x, axis=1, keepdims=True)

# ==================== РЕГУЛЯРИЗАЦИЯ В НЕЙРОННЫХ СЕТЯХ ====================
class Regularization:
    """
    Методы регуляризации для нейронных сетей
    """
    
    @staticmethod
    def dropout(X, dropout_rate=0.5, training=True):
        """
        Dropout регуляризация
        
        Случайно "выключает" нейроны во время обучения
        Предотвращает переобучение
        """
        if not training:
            return X
        
        mask = np.random.binomial(1, 1 - dropout_rate, X.shape)
        return X * mask / (1 - dropout_rate)
    
    @staticmethod
    def batch_normalization(X, gamma=1, beta=0, eps=1e-8):
        """
        Batch Normalization
        
        Нормализует входы каждого слоя
        Ускоряет обучение и стабилизирует процесс
        """
        mean = np.mean(X, axis=0)
        var = np.var(X, axis=0)
        
        X_normalized = (X - mean) / np.sqrt(var + eps)
        return gamma * X_normalized + beta, mean, var
    
    @staticmethod
    def layer_normalization(X, gamma=1, beta=0, eps=1e-8):
        """
        Layer Normalization
        
        Нормализует по признакам (не по батчу)
        Полезно для RNN и трансформеров
        """
        mean = np.mean(X, axis=1, keepdims=True)
        var = np.var(X, axis=1, keepdims=True)
        
        X_normalized = (X - mean) / np.sqrt(var + eps)
        return gamma * X_normalized + beta

# ==================== ОПТИМИЗАТОРЫ ====================
class Optimizers:
    """
    Различные алгоритмы оптимизации
    """
    
    class SGD:
        """
        Стохастический градиентный спуск
        """
        def __init__(self, learning_rate=0.01, momentum=0.0):
            self.learning_rate = learning_rate
            self.momentum = momentum
            self.velocities = {}
        
        def update(self, params, gradients, param_name):
            if param_name not in self.velocities:
                self.velocities[param_name] = np.zeros_like(params)
            
            # Обновление скорости с моментом
            self.velocities[param_name] = (self.momentum * self.velocities[param_name] + 
                                         self.learning_rate * gradients)
            
            # Обновление параметров
            return params - self.velocities[param_name]
    
    class Adam:
        """
        Adaptive Moment Estimation
        
        Комбинирует идеи momentum и RMSprop
        """
        def __init__(self, learning_rate=0.001, beta1=0.9, beta2=0.999, epsilon=1e-8):
            self.learning_rate = learning_rate
            self.beta1 = beta1
            self.beta2 = beta2
            self.epsilon = epsilon
            self.m = {}  # Первый момент
            self.v = {}  # Второй момент
            self.t = 0   # Временной шаг
        
        def update(self, params, gradients, param_name):
            self.t += 1
            
            if param_name not in self.m:
                self.m[param_name] = np.zeros_like(params)
                self.v[param_name] = np.zeros_like(params)
            
            # Обновление моментов
            self.m[param_name] = self.beta1 * self.m[param_name] + (1 - self.beta1) * gradients
            self.v[param_name] = self.beta2 * self.v[param_name] + (1 - self.beta2) * gradients**2
            
            # Коррекция смещения
            m_corrected = self.m[param_name] / (1 - self.beta1**self.t)
            v_corrected = self.v[param_name] / (1 - self.beta2**self.t)
            
            # Обновление параметров
            return params - self.learning_rate * m_corrected / (np.sqrt(v_corrected) + self.epsilon)
    
    class RMSprop:
        """
        Root Mean Square Propagation
        
        Адаптивная скорость обучения
        """
        def __init__(self, learning_rate=0.001, decay_rate=0.9, epsilon=1e-8):
            self.learning_rate = learning_rate
            self.decay_rate = decay_rate
            self.epsilon = epsilon
            self.cache = {}
        
        def update(self, params, gradients, param_name):
            if param_name not in self.cache:
                self.cache[param_name] = np.zeros_like(params)
            
            # Обновление кэша
            self.cache[param_name] = (self.decay_rate * self.cache[param_name] + 
                                    (1 - self.decay_rate) * gradients**2)
            
            # Обновление параметров
            return params - self.learning_rate * gradients / (np.sqrt(self.cache[param_name]) + self.epsilon)

# ==================== СВЕРТОЧНЫЕ НЕЙРОННЫЕ СЕТИ ====================
class ConvolutionalLayer:
    """
    Простая реализация сверточного слоя
    """
    
    def __init__(self, num_filters, filter_size, stride=1, padding=0):
        self.num_filters = num_filters
        self.filter_size = filter_size
        self.stride = stride
        self.padding = padding
        
        # Инициализация фильтров
        self.filters = np.random.randn(num_filters, filter_size, filter_size) * 0.1
        self.biases = np.zeros(num_filters)
    
    def convolution_2d(self, image, kernel):
        """
        2D свертка
        
        Основная операция CNN
        """
        # Размеры
        img_height, img_width = image.shape
        kernel_height, kernel_width = kernel.shape
        
        # Размер выходного изображения
        out_height = (img_height - kernel_height) // self.stride + 1
        out_width = (img_width - kernel_width) // self.stride + 1
        
        # Выходное изображение
        output = np.zeros((out_height, out_width))
        
        # Свертка
        for i in range(0, out_height * self.stride, self.stride):
            for j in range(0, out_width * self.stride, self.stride):
                # Извлечение окна
                window = image[i:i+kernel_height, j:j+kernel_width]
                
                # Элементное произведение и сумма
                output[i//self.stride, j//self.stride] = np.sum(window * kernel)
        
        return output
    
    def forward(self, input_image):
        """
        Прямое распространение через сверточный слой
        """
        batch_size, height, width = input_image.shape
        
        # Размер выходного изображения
        out_height = (height - self.filter_size) // self.stride + 1
        out_width = (width - self.filter_size) // self.stride + 1
        
        # Выходные feature maps
        output = np.zeros((batch_size, self.num_filters, out_height, out_width))
        
        for b in range(batch_size):
            for f in range(self.num_filters):
                # Свертка с фильтром
                conv_result = self.convolution_2d(input_image[b], self.filters[f])
                output[b, f] = conv_result + self.biases[f]
        
        return output

class PoolingLayer:
    """
    Слой субдискретизации (pooling)
    """
    
    def __init__(self, pool_size=2, stride=2, mode='max'):
        self.pool_size = pool_size
        self.stride = stride
        self.mode = mode
    
    def forward(self, input_tensor):
        """
        Прямое распространение через pooling слой
        """
        batch_size, channels, height, width = input_tensor.shape
        
        # Размер выходного тензора
        out_height = (height - self.pool_size) // self.stride + 1
        out_width = (width - self.pool_size) // self.stride + 1
        
        output = np.zeros((batch_size, channels, out_height, out_width))
        
        for b in range(batch_size):
            for c in range(channels):
                for i in range(out_height):
                    for j in range(out_width):
                        # Определение окна
                        start_i = i * self.stride
                        start_j = j * self.stride
                        end_i = start_i + self.pool_size
                        end_j = start_j + self.pool_size
                        
                        # Извлечение окна
                        window = input_tensor[b, c, start_i:end_i, start_j:end_j]
                        
                        # Pooling операция
                        if self.mode == 'max':
                            output[b, c, i, j] = np.max(window)
                        elif self.mode == 'avg':
                            output[b, c, i, j] = np.mean(window)
        
        return output

# ==================== РЕКУРРЕНТНЫЕ НЕЙРОННЫЕ СЕТИ ====================
class SimpleRNN:
    """
    Простая рекуррентная нейронная сеть
    """
    
    def __init__(self, input_size, hidden_size, output_size):
        self.input_size = input_size
        self.hidden_size = hidden_size
        self.output_size = output_size
        
        # Инициализация весов
        self.Wxh = np.random.randn(input_size, hidden_size) * 0.1
        self.Whh = np.random.randn(hidden_size, hidden_size) * 0.1
        self.Why = np.random.randn(hidden_size, output_size) * 0.1
        
        # Bias
        self.bh = np.zeros((1, hidden_size))
        self.by = np.zeros((1, output_size))
    
    def forward(self, inputs):
        """
        Прямое распространение через RNN
        
        inputs: последовательность входных данных
        """
        sequence_length = len(inputs)
        
        # Инициализация скрытого состояния
        h = np.zeros((1, self.hidden_size))
        
        # Хранение состояний
        hidden_states = []
        outputs = []
        
        for t in range(sequence_length):
            # Текущий вход
            x = inputs[t].reshape(1, -1)
            
            # Обновление скрытого состояния
            h = np.tanh(np.dot(x, self.Wxh) + np.dot(h, self.Whh) + self.bh)
            
            # Вычисление выхода
            y = np.dot(h, self.Why) + self.by
            
            hidden_states.append(h.copy())
            outputs.append(y.copy())
        
        return outputs, hidden_states

class LSTM:
    """
    Long Short-Term Memory
    
    Решает проблему затухания градиентов в RNN
    """
    
    def __init__(self, input_size, hidden_size):
        self.input_size = input_size
        self.hidden_size = hidden_size
        
        # Веса для forget gate
        self.Wf = np.random.randn(input_size + hidden_size, hidden_size) * 0.1
        self.bf = np.zeros((1, hidden_size))
        
        # Веса для input gate
        self.Wi = np.random.randn(input_size + hidden_size, hidden_size) * 0.1
        self.bi = np.zeros((1, hidden_size))
        
        # Веса для candidate values
        self.Wc = np.random.randn(input_size + hidden_size, hidden_size) * 0.1
        self.bc = np.zeros((1, hidden_size))
        
        # Веса для output gate
        self.Wo = np.random.randn(input_size + hidden_size, hidden_size) * 0.1
        self.bo = np.zeros((1, hidden_size))
    
    def sigmoid(self, x):
        """Сигмоидная функция"""
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
    
    def forward_step(self, x, h_prev, c_prev):
        """
        Один шаг LSTM
        
        x: текущий вход
        h_prev: предыдущее скрытое состояние
        c_prev: предыдущее состояние ячейки
        """
        # Объединение входа и предыдущего скрытого состояния
        combined = np.concatenate([x, h_prev], axis=1)
        
        # Forget gate: решает, что забыть из состояния ячейки
        f_gate = self.sigmoid(np.dot(combined, self.Wf) + self.bf)
        
        # Input gate: решает, какую информацию сохранить
        i_gate = self.sigmoid(np.dot(combined, self.Wi) + self.bi)
        
        # Candidate values: новая информация для сохранения
        c_candidate = np.tanh(np.dot(combined, self.Wc) + self.bc)
        
        # Обновление состояния ячейки
        c_new = f_gate * c_prev + i_gate * c_candidate
        
        # Output gate: решает, что вывести
        o_gate = self.sigmoid(np.dot(combined, self.Wo) + self.bo)
        
        # Новое скрытое состояние
        h_new = o_gate * np.tanh(c_new)
        
        return h_new, c_new
    
    def forward(self, inputs):
        """
        Прямое распространение через LSTM
        """
        sequence_length = len(inputs)
        
        # Инициализация состояний
        h = np.zeros((1, self.hidden_size))
        c = np.zeros((1, self.hidden_size))
        
        outputs = []
        
        for t in range(sequence_length):
            x = inputs[t].reshape(1, -1)
            h, c = self.forward_step(x, h, c)
            outputs.append(h.copy())
        
        return outputs

# ==================== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ====================
def neural_network_examples():
    """
    Практические примеры использования нейронных сетей
    """
    
    # Пример 1: Классификация с MLP
    print("=== MLP для классификации ===")
    
    # Создаем нелинейно разделимые данные
    X, y = make_circles(n_samples=1000, noise=0.1, factor=0.3, random_state=42)
    
    # Стандартизация данных
    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X)
    
    # Разделение на обучающую и тестовую выборки
    from sklearn.model_selection import train_test_split
    X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, random_state=42)
    
    # Создание и обучение MLP
    mlp = MLP(layer_sizes=[2, 10, 8, 1], learning_rate=0.1, max_epochs=1000)
    train_losses, _ = mlp.fit(X_train, y_train)
    
    # Предсказания
    predictions = mlp.predict(X_test)
    binary_predictions = (predictions > 0.5).astype(int)
    
    # Оценка качества
    accuracy = np.mean(binary_predictions.flatten() == y_test)
    print(f"Точность MLP: {accuracy:.3f}")
    
    # Пример 2: Сравнение функций активации
    print("\n=== Сравнение функций активации ===")
    
    x = np.linspace(-5, 5, 100)
    activations = ActivationFunctions()
    
    functions = {
        'Sigmoid': activations.sigmoid(x),
        'Tanh': activations.tanh(x),
        'ReLU': activations.relu(x),
        'Leaky ReLU': activations.leaky_relu(x),
        'Swish': activations.swish(x)
    }
    
    plt.figure(figsize=(12, 8))
    for name, values in functions.items():
        plt.plot(x, values, label=name, linewidth=2)
    
    plt.xlabel('x')
    plt.ylabel('f(x)')
    plt.title('Сравнение функций активации')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()
    
    # Пример 3: Демонстрация переобучения
    print("\n=== Анализ переобучения ===")
    
    # Малый датасет для демонстрации переобучения
    X_small, y_small = make_classification(n_samples=100, n_features=2, n_redundant=0, 
                                          n_informative=2, n_clusters_per_class=1, random_state=42)
    
    X_small_scaled = StandardScaler().fit_transform(X_small)
    
    # Большая сеть на малых данных
    large_mlp = MLP(layer_sizes=[2, 50, 50, 1], learning_rate=0.01, max_epochs=2000)
    
    # Обучение с валидацией
    X_train_small, X_val_small, y_train_small, y_val_small = train_test_split(
        X_small_scaled, y_small, test_size=0.3, random_state=42
    )
    
    train_losses, val_losses = large_mlp.fit(X_train_small, y_train_small, 
                                           validation_data=(X_val_small, y_val_small))
    
    # Построение кривых обучения
    plt.figure(figsize=(10, 6))
    plt.plot(train_losses, label='Training Loss', color='blue')
    plt.plot(val_losses, label='Validation Loss', color='red')
    plt.xlabel('Epochs')
    plt.ylabel('Loss')
    plt.title('Кривые обучения: признаки переобучения')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()
    
    return {
        'mlp_accuracy': accuracy,
        'train_losses': train_losses,
        'val_losses': val_losses
    }

# ==================== АРХИТЕКТУРНЫЕ ПАТТЕРНЫ ====================
deep_learning_architectures = {
    "Feedforward Networks": {
        "Применение": "Классификация, регрессия",
        "Особенности": "Простая архитектура, нет циклов",
        "Примеры": "MLP, Deep Neural Networks"
    },
    
    "Convolutional Networks": {
        "Применение": "Компьютерное зрение, обработка изображений",
        "Особенности": "Локальные связи, разделение весов",
        "Примеры": "LeNet, AlexNet, VGG, ResNet"
    },
    
    "Recurrent Networks": {
        "Применение": "Обработка последовательностей, NLP",
        "Особенности": "Память, рекуррентные связи",
        "Примеры": "RNN, LSTM, GRU"
    },
    
    "Attention Mechanisms": {
        "Применение": "Машинный перевод, понимание текста",
        "Особенности": "Фокусировка на важных частях",
        "Примеры": "Transformer, BERT, GPT"
    },
    
    "Generative Models": {
        "Применение": "Генерация данных, стиль-трансфер",
        "Особенности": "Обучение распределений",
        "Примеры": "GAN, VAE, Flow-based models"
    }
}

# ==================== ПРАКТИЧЕСКИЕ СОВЕТЫ ====================
deep_learning_best_practices = {
    "Подготовка данных": [
        "Стандартизация/нормализация входных данных",
        "Аугментация данных для увеличения разнообразия",
        "Правильное разделение на train/val/test",
        "Обработка несбалансированных классов"
    ],
    
    "Архитектура сети": [
        "Начинать с простой архитектуры",
        "Добавлять сложность постепенно",
        "Использовать пропуски соединений (skip connections)",
        "Правильно выбирать функции активации"
    ],
    
    "Обучение": [
        "Использовать современные оптимизаторы (Adam, AdamW)",
        "Применять техники регуляризации (dropout, batch norm)",
        "Мониторить градиенты и веса",
        "Использовать early stopping"
    ],
    
    "Отладка": [
        "Проверять на переобучение",
        "Анализировать кривые обучения",
        "Визуализировать активации",
        "Тестировать на простых данных"
    ]
}
```

---

*Нейронные сети - это попытка имитировать обучение мозга, но с математической точностью* 🧠⚡ 