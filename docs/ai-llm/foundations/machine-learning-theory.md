# 📊 Теория машинного обучения

## 🧮 Математические основы

### Линейная алгебра
```
Ключевые концепции:
├── Векторы и операции над ними
├── Матрицы и матричные операции
├── Собственные значения и векторы
├── Сингулярное разложение (SVD)
├── Нормы векторов и матриц
└── Проекции и ортогонализация
```

#### Векторные операции
```python
import numpy as np

# Основные операции
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Скалярное произведение
dot_product = np.dot(a, b)  # или a @ b
print(f"Dot product: {dot_product}")

# Норма вектора
norm_a = np.linalg.norm(a)
print(f"Norm of a: {norm_a}")

# Косинус угла между векторами
cosine_similarity = np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
print(f"Cosine similarity: {cosine_similarity}")
```

#### Матричные операции
```python
# Создание матриц
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Матричное умножение
C = A @ B
print(f"Matrix multiplication:\n{C}")

# Собственные значения и векторы
eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"Eigenvalues: {eigenvalues}")
print(f"Eigenvectors:\n{eigenvectors}")

# Сингулярное разложение
U, s, Vt = np.linalg.svd(A)
print(f"SVD - U:\n{U}")
print(f"SVD - singular values: {s}")
print(f"SVD - Vt:\n{Vt}")
```

### Теория вероятностей
```
Фундаментальные понятия:
├── Вероятностные пространства
├── Случайные величины
├── Распределения вероятностей
├── Условная вероятность
├── Теорема Байеса
├── Центральная предельная теорема
└── Марковские цепи
```

#### Байесовская теория
```python
import scipy.stats as stats

# Теорема Байеса: P(A|B) = P(B|A) * P(A) / P(B)
def bayes_theorem(prior, likelihood, evidence):
    """
    Вычисление апостериорной вероятности
    """
    posterior = (likelihood * prior) / evidence
    return posterior

# Пример: диагностика заболевания
# P(болезнь) = 0.01 - априорная вероятность
# P(тест+|болезнь) = 0.95 - чувствительность теста
# P(тест+|здоров) = 0.05 - ложноположительные результаты

prior_disease = 0.01
prior_healthy = 0.99
likelihood_positive_given_disease = 0.95
likelihood_positive_given_healthy = 0.05

# Полная вероятность положительного теста
evidence = (likelihood_positive_given_disease * prior_disease + 
           likelihood_positive_given_healthy * prior_healthy)

# Вероятность болезни при положительном тесте
posterior = bayes_theorem(prior_disease, likelihood_positive_given_disease, evidence)
print(f"Вероятность болезни при положительном тесте: {posterior:.3f}")
```

### Статистический вывод
```python
# Методы оценки параметров
from scipy import stats
import matplotlib.pyplot as plt

# Генерация данных
np.random.seed(42)
true_mean = 5
true_std = 2
sample_size = 1000
data = np.random.normal(true_mean, true_std, sample_size)

# Максимальное правдоподобие (MLE)
mle_mean = np.mean(data)
mle_std = np.std(data, ddof=0)

# Байесовская оценка (с априорным распределением)
# Предполагаем нормальное априорное распределение для среднего
prior_mean = 0
prior_std = 10

# Апостериорное распределение для среднего
posterior_precision = 1/prior_std**2 + sample_size/true_std**2
posterior_mean = (prior_mean/prior_std**2 + np.sum(data)/true_std**2) / posterior_precision
posterior_std = np.sqrt(1/posterior_precision)

print(f"Истинные параметры: μ={true_mean}, σ={true_std}")
print(f"MLE оценки: μ={mle_mean:.3f}, σ={mle_std:.3f}")
print(f"Байесовская оценка: μ={posterior_mean:.3f}, σ={posterior_std:.3f}")
```

---

## 🎯 Типы обучения

### Supervised Learning (Обучение с учителем)
```
Задачи:
├── Классификация - предсказание категорий
├── Регрессия - предсказание непрерывных значений
├── Ранжирование - упорядочивание объектов
└── Structured prediction - предсказание структур
```

#### Линейная регрессия
```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
import matplotlib.pyplot as plt

# Генерация данных
X = np.linspace(0, 1, 100).reshape(-1, 1)
y = 1.5 * X.ravel() + 0.2 * np.random.normal(size=X.shape[0])

# Простая линейная регрессия
lr = LinearRegression()
lr.fit(X, y)

# Полиномиальная регрессия
poly_features = PolynomialFeatures(degree=3)
X_poly = poly_features.fit_transform(X)
poly_lr = LinearRegression()
poly_lr.fit(X_poly, y)

# Математическая формула линейной регрессии:
# y = w₀ + w₁x₁ + w₂x₂ + ... + wₙxₙ
# или в векторной форме: y = w^T x + b

print(f"Коэффициенты: {lr.coef_}")
print(f"Intercept: {lr.intercept_}")
print(f"R² score: {lr.score(X, y):.3f}")
```

#### Логистическая регрессия
```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification

# Генерация данных для классификации
X, y = make_classification(n_samples=1000, n_features=2, n_redundant=0, 
                          n_informative=2, n_clusters_per_class=1, random_state=42)

# Логистическая регрессия
log_reg = LogisticRegression()
log_reg.fit(X, y)

# Математическая формула:
# p(y=1|x) = 1 / (1 + exp(-(w^T x + b)))
# log-odds: log(p/(1-p)) = w^T x + b

def sigmoid(z):
    """Сигмоидная функция"""
    return 1 / (1 + np.exp(-z))

# Предсказание вероятностей
probabilities = log_reg.predict_proba(X)
print(f"Точность: {log_reg.score(X, y):.3f}")
```

### Unsupervised Learning (Обучение без учителя)
```
Основные задачи:
├── Кластеризация - группировка похожих объектов
├── Понижение размерности - сжатие данных
├── Обнаружение аномалий - поиск выбросов
├── Ассоциативные правила - поиск закономерностей
└── Генеративное моделирование - создание новых данных
```

#### K-means кластеризация
```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# Генерация данных
X, _ = make_blobs(n_samples=300, centers=4, cluster_std=0.6, random_state=42)

# K-means алгоритм
kmeans = KMeans(n_clusters=4, random_state=42)
labels = kmeans.fit_predict(X)
centroids = kmeans.cluster_centers_

# Математический алгоритм K-means:
# 1. Инициализировать k центроидов случайно
# 2. Повторять до сходимости:
#    a) Назначить каждую точку ближайшему центроиду
#    b) Обновить центроиды как среднее точек в кластере

def kmeans_manual(X, k, max_iters=100):
    """Ручная реализация K-means"""
    # Инициализация центроидов
    centroids = X[np.random.choice(X.shape[0], k, replace=False)]
    
    for _ in range(max_iters):
        # Назначение точек кластерам
        distances = np.sqrt(((X - centroids[:, np.newaxis])**2).sum(axis=2))
        labels = np.argmin(distances, axis=0)
        
        # Обновление центроидов
        new_centroids = np.array([X[labels == i].mean(axis=0) for i in range(k)])
        
        # Проверка сходимости
        if np.allclose(centroids, new_centroids):
            break
        centroids = new_centroids
    
    return labels, centroids

manual_labels, manual_centroids = kmeans_manual(X, 4)
```

#### Principal Component Analysis (PCA)
```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_iris

# Загрузка данных
iris = load_iris()
X = iris.data
y = iris.target

# PCA
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)

# Математическая формула PCA:
# 1. Центрирование данных: X_centered = X - mean(X)
# 2. Вычисление ковариационной матрицы: C = (X_centered^T × X_centered) / (n-1)
# 3. Собственные векторы и значения: C × v = λ × v
# 4. Проекция: X_new = X_centered × V

print(f"Объясненная дисперсия: {pca.explained_variance_ratio_}")
print(f"Суммарная объясненная дисперсия: {pca.explained_variance_ratio_.sum():.3f}")

# Компоненты (главные направления)
print(f"Компоненты PCA:\n{pca.components_}")
```

### Reinforcement Learning (Обучение с подкреплением)
```
Ключевые концепции:
├── Агент, среда, действия, состояния
├── Функция вознаграждения
├── Политика (policy)
├── Функция ценности (value function)
├── Q-learning
└── Policy gradient methods
```

#### Q-Learning алгоритм
```python
import gym
import numpy as np

class QLearningAgent:
    def __init__(self, state_size, action_size, learning_rate=0.1, 
                 discount_factor=0.95, epsilon=1.0, epsilon_decay=0.995):
        self.state_size = state_size
        self.action_size = action_size
        self.lr = learning_rate
        self.gamma = discount_factor
        self.epsilon = epsilon
        self.epsilon_decay = epsilon_decay
        self.q_table = np.zeros((state_size, action_size))
    
    def choose_action(self, state):
        """Epsilon-greedy policy"""
        if np.random.random() < self.epsilon:
            return np.random.choice(self.action_size)
        return np.argmax(self.q_table[state])
    
    def learn(self, state, action, reward, next_state, done):
        """Q-learning update rule"""
        if done:
            target = reward
        else:
            target = reward + self.gamma * np.max(self.q_table[next_state])
        
        # Q(s,a) ← Q(s,a) + α[r + γ max Q(s',a') - Q(s,a)]
        self.q_table[state, action] += self.lr * (target - self.q_table[state, action])
        
        if self.epsilon > 0.01:
            self.epsilon *= self.epsilon_decay

# Математическая формула Q-learning:
# Q(s,a) ← Q(s,a) + α[r + γ max Q(s',a') - Q(s,a)]
# где:
# - α - learning rate
# - γ - discount factor
# - r - reward
# - s,a - текущее состояние и действие
# - s' - следующее состояние
```

---

## 📈 Функции потерь и оптимизация

### Функции потерь
```python
import torch
import torch.nn as nn

# Основные функции потерь

# 1. Mean Squared Error (для регрессии)
def mse_loss(y_true, y_pred):
    return np.mean((y_true - y_pred)**2)

# 2. Cross-entropy (для классификации)
def cross_entropy_loss(y_true, y_pred):
    # y_true - one-hot encoded, y_pred - probabilities
    return -np.sum(y_true * np.log(y_pred + 1e-15))

# 3. Binary Cross-entropy
def binary_cross_entropy(y_true, y_pred):
    return -np.mean(y_true * np.log(y_pred + 1e-15) + 
                   (1 - y_true) * np.log(1 - y_pred + 1e-15))

# 4. Hinge Loss (для SVM)
def hinge_loss(y_true, y_pred):
    return np.maximum(0, 1 - y_true * y_pred)

# PyTorch implementations
mse = nn.MSELoss()
ce = nn.CrossEntropyLoss()
bce = nn.BCELoss()
```

### Методы оптимизации
```python
# Градиентный спуск и его варианты

class SGD:
    def __init__(self, learning_rate=0.01):
        self.lr = learning_rate
    
    def update(self, params, gradients):
        """Стохастический градиентный спуск"""
        for param, grad in zip(params, gradients):
            param -= self.lr * grad

class Momentum:
    def __init__(self, learning_rate=0.01, momentum=0.9):
        self.lr = learning_rate
        self.momentum = momentum
        self.velocities = None
    
    def update(self, params, gradients):
        """SGD с моментом"""
        if self.velocities is None:
            self.velocities = [np.zeros_like(p) for p in params]
        
        for param, grad, velocity in zip(params, gradients, self.velocities):
            velocity[:] = self.momentum * velocity - self.lr * grad
            param += velocity

class Adam:
    def __init__(self, learning_rate=0.001, beta1=0.9, beta2=0.999, epsilon=1e-8):
        self.lr = learning_rate
        self.beta1 = beta1
        self.beta2 = beta2
        self.epsilon = epsilon
        self.m = None  # First moment
        self.v = None  # Second moment
        self.t = 0     # Time step
    
    def update(self, params, gradients):
        """Adam optimizer"""
        if self.m is None:
            self.m = [np.zeros_like(p) for p in params]
            self.v = [np.zeros_like(p) for p in params]
        
        self.t += 1
        
        for param, grad, m, v in zip(params, gradients, self.m, self.v):
            # Update biased first moment estimate
            m[:] = self.beta1 * m + (1 - self.beta1) * grad
            
            # Update biased second raw moment estimate
            v[:] = self.beta2 * v + (1 - self.beta2) * grad**2
            
            # Compute bias-corrected moments
            m_hat = m / (1 - self.beta1**self.t)
            v_hat = v / (1 - self.beta2**self.t)
            
            # Update parameters
            param -= self.lr * m_hat / (np.sqrt(v_hat) + self.epsilon)
```

---

## 🔄 Обобщение и регуляризация

### Bias-Variance Trade-off
```
Компоненты ошибки модели:
├── Bias (смещение) - систематическая ошибка
├── Variance (дисперсия) - чувствительность к данным
├── Irreducible error - неустранимый шум
└── Total error = Bias² + Variance + Noise
```

```python
# Демонстрация bias-variance trade-off
from sklearn.ensemble import BaggingRegressor
from sklearn.tree import DecisionTreeRegressor
from sklearn.metrics import mean_squared_error

def bias_variance_decomposition(X_train, y_train, X_test, y_test, n_trials=100):
    """
    Разложение ошибки на bias и variance
    """
    predictions = []
    
    for _ in range(n_trials):
        # Создание bootstrap выборки
        indices = np.random.choice(len(X_train), len(X_train), replace=True)
        X_boot = X_train[indices]
        y_boot = y_train[indices]
        
        # Обучение модели
        model = DecisionTreeRegressor(max_depth=10)
        model.fit(X_boot, y_boot)
        
        # Предсказание
        y_pred = model.predict(X_test)
        predictions.append(y_pred)
    
    predictions = np.array(predictions)
    
    # Вычисление bias и variance
    mean_prediction = np.mean(predictions, axis=0)
    bias_squared = np.mean((mean_prediction - y_test)**2)
    variance = np.mean(np.var(predictions, axis=0))
    
    return bias_squared, variance

# Пример использования
X = np.random.normal(0, 1, (1000, 5))
y = X[:, 0] + 0.5 * X[:, 1] + np.random.normal(0, 0.1, 1000)

X_train, X_test = X[:800], X[800:]
y_train, y_test = y[:800], y[800:]

bias_sq, variance = bias_variance_decomposition(X_train, y_train, X_test, y_test)
print(f"Bias²: {bias_sq:.4f}")
print(f"Variance: {variance:.4f}")
```

### Техники регуляризации
```python
# L1 и L2 регуляризация
from sklearn.linear_model import Ridge, Lasso, ElasticNet

# Ridge regression (L2 regularization)
# Loss = MSE + α * Σ(w²)
ridge = Ridge(alpha=1.0)

# Lasso regression (L1 regularization)
# Loss = MSE + α * Σ|w|
lasso = Lasso(alpha=1.0)

# Elastic Net (комбинация L1 и L2)
# Loss = MSE + α₁ * Σ|w| + α₂ * Σ(w²)
elastic_net = ElasticNet(alpha=1.0, l1_ratio=0.5)

# Dropout (для нейронных сетей)
class Dropout:
    def __init__(self, drop_rate=0.5):
        self.drop_rate = drop_rate
        self.mask = None
    
    def forward(self, x, training=True):
        if training:
            self.mask = np.random.binomial(1, 1-self.drop_rate, x.shape) / (1-self.drop_rate)
            return x * self.mask
        return x
    
    def backward(self, grad_output):
        return grad_output * self.mask

# Batch Normalization
class BatchNorm:
    def __init__(self, num_features, eps=1e-5, momentum=0.1):
        self.num_features = num_features
        self.eps = eps
        self.momentum = momentum
        
        # Learnable parameters
        self.gamma = np.ones(num_features)
        self.beta = np.zeros(num_features)
        
        # Running statistics
        self.running_mean = np.zeros(num_features)
        self.running_var = np.ones(num_features)
    
    def forward(self, x, training=True):
        if training:
            batch_mean = np.mean(x, axis=0)
            batch_var = np.var(x, axis=0)
            
            # Update running statistics
            self.running_mean = (1 - self.momentum) * self.running_mean + self.momentum * batch_mean
            self.running_var = (1 - self.momentum) * self.running_var + self.momentum * batch_var
            
            # Normalize
            x_norm = (x - batch_mean) / np.sqrt(batch_var + self.eps)
        else:
            x_norm = (x - self.running_mean) / np.sqrt(self.running_var + self.eps)
        
        # Scale and shift
        return self.gamma * x_norm + self.beta
```

---

## 📊 Оценка моделей

### Метрики классификации
```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score, 
                           f1_score, roc_auc_score, confusion_matrix)

def evaluate_classification(y_true, y_pred, y_prob=None):
    """Полная оценка классификационной модели"""
    
    # Основные метрики
    accuracy = accuracy_score(y_true, y_pred)
    precision = precision_score(y_true, y_pred, average='weighted')
    recall = recall_score(y_true, y_pred, average='weighted')
    f1 = f1_score(y_true, y_pred, average='weighted')
    
    print(f"Accuracy: {accuracy:.3f}")
    print(f"Precision: {precision:.3f}")
    print(f"Recall: {recall:.3f}")
    print(f"F1-score: {f1:.3f}")
    
    # AUC-ROC (если есть вероятности)
    if y_prob is not None:
        auc = roc_auc_score(y_true, y_prob)
        print(f"AUC-ROC: {auc:.3f}")
    
    # Матрица ошибок
    cm = confusion_matrix(y_true, y_pred)
    print(f"Confusion Matrix:\n{cm}")
    
    return {
        'accuracy': accuracy,
        'precision': precision,
        'recall': recall,
        'f1': f1,
        'confusion_matrix': cm
    }

# Математические формулы:
# Precision = TP / (TP + FP)
# Recall = TP / (TP + FN)
# F1-score = 2 * (Precision * Recall) / (Precision + Recall)
# Accuracy = (TP + TN) / (TP + TN + FP + FN)
```

### Метрики регрессии
```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

def evaluate_regression(y_true, y_pred):
    """Полная оценка регрессионной модели"""
    
    # Основные метрики
    mse = mean_squared_error(y_true, y_pred)
    rmse = np.sqrt(mse)
    mae = mean_absolute_error(y_true, y_pred)
    r2 = r2_score(y_true, y_pred)
    
    # Дополнительные метрики
    mape = np.mean(np.abs((y_true - y_pred) / y_true)) * 100
    
    print(f"MSE: {mse:.3f}")
    print(f"RMSE: {rmse:.3f}")
    print(f"MAE: {mae:.3f}")
    print(f"R²: {r2:.3f}")
    print(f"MAPE: {mape:.2f}%")
    
    return {
        'mse': mse,
        'rmse': rmse,
        'mae': mae,
        'r2': r2,
        'mape': mape
    }

# Математические формулы:
# MSE = (1/n) * Σ(y_true - y_pred)²
# RMSE = √MSE
# MAE = (1/n) * Σ|y_true - y_pred|
# R² = 1 - SS_res / SS_tot
# MAPE = (100/n) * Σ|y_true - y_pred|/|y_true|
```

### Cross-Validation
```python
from sklearn.model_selection import cross_val_score, StratifiedKFold, TimeSeriesSplit

def comprehensive_cross_validation(model, X, y, cv_type='kfold', n_splits=5):
    """Комплексная кросс-валидация"""
    
    if cv_type == 'stratified':
        cv = StratifiedKFold(n_splits=n_splits, shuffle=True, random_state=42)
    elif cv_type == 'timeseries':
        cv = TimeSeriesSplit(n_splits=n_splits)
    else:  # standard k-fold
        cv = KFold(n_splits=n_splits, shuffle=True, random_state=42)
    
    # Различные метрики
    scoring = ['accuracy', 'precision_weighted', 'recall_weighted', 'f1_weighted']
    
    results = {}
    for metric in scoring:
        scores = cross_val_score(model, X, y, cv=cv, scoring=metric)
        results[metric] = {
            'scores': scores,
            'mean': scores.mean(),
            'std': scores.std()
        }
        print(f"{metric}: {scores.mean():.3f} (+/- {scores.std() * 2:.3f})")
    
    return results
```

---

## 🎯 Специальные темы

### Ensemble Methods
```python
from sklearn.ensemble import RandomForestClassifier, VotingClassifier, BaggingClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.svm import SVC
from sklearn.linear_model import LogisticRegression

# Bagging
bagging = BaggingClassifier(
    base_estimator=DecisionTreeClassifier(),
    n_estimators=100,
    random_state=42
)

# Random Forest (specialized bagging)
rf = RandomForestClassifier(n_estimators=100, random_state=42)

# Voting Classifier
voting_clf = VotingClassifier(
    estimators=[
        ('lr', LogisticRegression()),
        ('svm', SVC(probability=True)),
        ('rf', RandomForestClassifier())
    ],
    voting='soft'  # использует вероятности
)

# AdaBoost (Boosting)
from sklearn.ensemble import AdaBoostClassifier
ada_boost = AdaBoostClassifier(
    base_estimator=DecisionTreeClassifier(max_depth=1),
    n_estimators=100,
    learning_rate=1.0
)

# Gradient Boosting
from sklearn.ensemble import GradientBoostingClassifier
gb_clf = GradientBoostingClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3
)
```

### Feature Engineering
```python
from sklearn.preprocessing import (StandardScaler, MinMaxScaler, RobustScaler,
                                 PolynomialFeatures, OneHotEncoder)
from sklearn.feature_selection import SelectKBest, f_classif, RFE

class FeatureEngineer:
    def __init__(self):
        self.scalers = {}
        self.encoders = {}
        self.selectors = {}
    
    def scale_features(self, X, method='standard'):
        """Масштабирование признаков"""
        if method == 'standard':
            scaler = StandardScaler()
        elif method == 'minmax':
            scaler = MinMaxScaler()
        elif method == 'robust':
            scaler = RobustScaler()
        
        X_scaled = scaler.fit_transform(X)
        self.scalers[method] = scaler
        return X_scaled
    
    def create_polynomial_features(self, X, degree=2):
        """Создание полиномиальных признаков"""
        poly = PolynomialFeatures(degree=degree, include_bias=False)
        X_poly = poly.fit_transform(X)
        self.poly_transformer = poly
        return X_poly
    
    def encode_categorical(self, X_cat):
        """Кодирование категориальных признаков"""
        encoder = OneHotEncoder(sparse=False, handle_unknown='ignore')
        X_encoded = encoder.fit_transform(X_cat)
        self.categorical_encoder = encoder
        return X_encoded
    
    def select_features(self, X, y, method='univariate', k=10):
        """Отбор признаков"""
        if method == 'univariate':
            selector = SelectKBest(score_func=f_classif, k=k)
        elif method == 'rfe':
            estimator = RandomForestClassifier()
            selector = RFE(estimator, n_features_to_select=k)
        
        X_selected = selector.fit_transform(X, y)
        self.feature_selector = selector
        return X_selected

# Пример комплексной предобработки
def preprocess_pipeline(X_numeric, X_categorical, y):
    """Полный пайплайн предобработки"""
    fe = FeatureEngineer()
    
    # Масштабирование численных признаков
    X_num_scaled = fe.scale_features(X_numeric, 'standard')
    
    # Создание полиномиальных признаков
    X_num_poly = fe.create_polynomial_features(X_num_scaled, degree=2)
    
    # Кодирование категориальных признаков
    X_cat_encoded = fe.encode_categorical(X_categorical)
    
    # Объединение признаков
    X_combined = np.hstack([X_num_poly, X_cat_encoded])
    
    # Отбор лучших признаков
    X_final = fe.select_features(X_combined, y, method='univariate', k=20)
    
    return X_final, fe
```

---

## 📚 Дополнительные ресурсы

### Связанные темы
- [[deep-learning-foundations|Основы глубокого обучения]] - нейронные сети и глубокое обучение
- [[neural-networks-architecture|Архитектура нейронных сетей]] - современные архитектуры
- [[ai-fundamentals|Основы AI]] - общие принципы искусственного интеллекта
- [[../mathematics/programming-math|Математика для программистов]] - математические основы

### Практические инструменты
- **Scikit-learn** - основная библиотека ML для Python
- **Pandas** - анализ и обработка данных
- **NumPy** - численные вычисления
- **Matplotlib/Seaborn** - визуализация данных
- **Jupyter Notebooks** - интерактивная разработка

---

💡 **Совет:** Машинное обучение требует solid понимания математики. Начните с линейной алгебры и статистики, затем изучайте алгоритмы на практических примерах. Всегда помните о важности качественных данных и правильной оценки моделей! 