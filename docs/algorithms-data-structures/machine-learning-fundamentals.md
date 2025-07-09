# Machine Learning Fundamentals

> **Основы машинного обучения для разработчиков**
>
> От математических принципов до практических алгоритмов

## 📊 Математические основы

### Линейная алгебра

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, confusion_matrix

# ==================== ВЕКТОРНЫЕ ОПЕРАЦИИ ====================
class LinearAlgebraML:
    """
    Основы линейной алгебры для ML
    """
    
    def vector_operations(self):
        """
        Базовые векторные операции
        
        Векторы - основа представления данных в ML
        """
        # Создание векторов
        a = np.array([1, 2, 3])
        b = np.array([4, 5, 6])
        
        # Скалярное произведение - мера сходства
        dot_product = np.dot(a, b)  # a·b = 1*4 + 2*5 + 3*6 = 32
        
        # Норма вектора - длина/величина
        norm_a = np.linalg.norm(a)  # ||a|| = √(1² + 2² + 3²) = √14
        
        # Косинусное сходство - угол между векторами
        cosine_similarity = dot_product / (np.linalg.norm(a) * np.linalg.norm(b))
        
        # Евклидово расстояние - расстояние между точками
        euclidean_distance = np.linalg.norm(a - b)
        
        return {
            'dot_product': dot_product,
            'norm_a': norm_a,
            'cosine_similarity': cosine_similarity,
            'euclidean_distance': euclidean_distance
        }
    
    def matrix_operations(self):
        """
        Матричные операции в ML
        
        Матрицы представляют датасеты и трансформации
        """
        # Матрица данных: строки = образцы, столбцы = признаки
        X = np.array([[1, 2], [3, 4], [5, 6]])  # 3 образца, 2 признака
        
        # Транспонирование - поворот матрицы
        X_T = X.T  # Теперь 2 строки, 3 столбца
        
        # Матричное умножение
        result = np.dot(X_T, X)  # (2x3) × (3x2) = (2x2)
        
        # Собственные векторы и значения
        eigenvalues, eigenvectors = np.linalg.eig(result)
        
        return {
            'original_shape': X.shape,
            'transposed_shape': X_T.shape,
            'matrix_product': result,
            'eigenvalues': eigenvalues,
            'eigenvectors': eigenvectors
        }

# ==================== СТАТИСТИКА И ВЕРОЯТНОСТЬ ====================
class StatisticsForML:
    """
    Статистические концепции для ML
    """
    
    def descriptive_statistics(self, data):
        """
        Описательная статистика
        
        Основные метрики для понимания данных
        """
        # Меры центральной тенденции
        mean = np.mean(data)           # Среднее - центр распределения
        median = np.median(data)       # Медиана - устойчива к выбросам
        mode = np.bincount(data).argmax()  # Мода - самое частое значение
        
        # Меры разброса
        variance = np.var(data)        # Дисперсия - разброс данных
        std_dev = np.std(data)         # Стандартное отклонение
        range_val = np.max(data) - np.min(data)  # Размах
        
        # Квантили
        q1 = np.percentile(data, 25)   # 1-й квартиль
        q3 = np.percentile(data, 75)   # 3-й квартиль
        iqr = q3 - q1                  # Межквартильный размах
        
        return {
            'mean': mean, 'median': median, 'mode': mode,
            'variance': variance, 'std_dev': std_dev, 'range': range_val,
            'q1': q1, 'q3': q3, 'iqr': iqr
        }
    
    def probability_distributions(self):
        """
        Основные вероятностные распределения
        """
        # Нормальное распределение - основа многих ML алгоритмов
        normal_data = np.random.normal(loc=0, scale=1, size=1000)
        
        # Биномиальное распределение - для классификации
        binomial_data = np.random.binomial(n=10, p=0.5, size=1000)
        
        # Равномерное распределение - для инициализации весов
        uniform_data = np.random.uniform(low=0, high=1, size=1000)
        
        return {
            'normal': normal_data,
            'binomial': binomial_data,
            'uniform': uniform_data
        }
    
    def hypothesis_testing(self, sample1, sample2):
        """
        Проверка гипотез
        
        Статистическая значимость различий
        """
        from scipy import stats
        
        # t-тест для сравнения средних
        t_stat, p_value = stats.ttest_ind(sample1, sample2)
        
        # Интерпретация результата
        alpha = 0.05  # Уровень значимости
        is_significant = p_value < alpha
        
        return {
            't_statistic': t_stat,
            'p_value': p_value,
            'is_significant': is_significant,
            'confidence_level': 1 - alpha
        }

# ==================== ОСНОВНЫЕ АЛГОРИТМЫ ML ====================
class SupervisedLearning:
    """
    Алгоритмы обучения с учителем
    """
    
    def __init__(self):
        self.models = {}
    
    def linear_regression_from_scratch(self, X, y):
        """
        Линейная регрессия с нуля
        
        Находит лучшую прямую линию через данные
        Формула: y = w₀ + w₁x₁ + w₂x₂ + ... + wₙxₙ
        """
        # Добавляем колонку единиц для свободного члена
        X_with_bias = np.column_stack([np.ones(len(X)), X])
        
        # Аналитическое решение: w = (X^T * X)^(-1) * X^T * y
        weights = np.linalg.inv(X_with_bias.T @ X_with_bias) @ X_with_bias.T @ y
        
        # Предсказание
        def predict(X_new):
            X_new_with_bias = np.column_stack([np.ones(len(X_new)), X_new])
            return X_new_with_bias @ weights
        
        return weights, predict
    
    def gradient_descent_regression(self, X, y, learning_rate=0.01, epochs=1000):
        """
        Линейная регрессия с градиентным спуском
        
        Итеративная оптимизация - основа глубокого обучения
        """
        # Инициализация весов
        m, n = X.shape
        weights = np.zeros(n + 1)  # +1 для bias
        X_with_bias = np.column_stack([np.ones(m), X])
        
        cost_history = []
        
        for epoch in range(epochs):
            # Прямой проход
            predictions = X_with_bias @ weights
            
            # Вычисление ошибки (MSE)
            cost = np.mean((predictions - y) ** 2)
            cost_history.append(cost)
            
            # Вычисление градиентов
            gradients = (2 / m) * X_with_bias.T @ (predictions - y)
            
            # Обновление весов
            weights -= learning_rate * gradients
        
        return weights, cost_history
    
    def logistic_regression_from_scratch(self, X, y, learning_rate=0.01, epochs=1000):
        """
        Логистическая регрессия с нуля
        
        Классификация с использованием сигмоидной функции
        """
        # Сигмоидная функция
        def sigmoid(z):
            # Защита от переполнения
            z = np.clip(z, -500, 500)
            return 1 / (1 + np.exp(-z))
        
        # Инициализация
        m, n = X.shape
        weights = np.zeros(n + 1)
        X_with_bias = np.column_stack([np.ones(m), X])
        
        cost_history = []
        
        for epoch in range(epochs):
            # Прямой проход
            linear_pred = X_with_bias @ weights
            predictions = sigmoid(linear_pred)
            
            # Логистическая функция потерь
            epsilon = 1e-15  # Избегаем log(0)
            predictions = np.clip(predictions, epsilon, 1 - epsilon)
            cost = -np.mean(y * np.log(predictions) + (1 - y) * np.log(1 - predictions))
            cost_history.append(cost)
            
            # Градиенты
            gradients = (1 / m) * X_with_bias.T @ (predictions - y)
            
            # Обновление весов
            weights -= learning_rate * gradients
        
        def predict_proba(X_new):
            X_new_with_bias = np.column_stack([np.ones(len(X_new)), X_new])
            return sigmoid(X_new_with_bias @ weights)
        
        def predict(X_new):
            return (predict_proba(X_new) >= 0.5).astype(int)
        
        return weights, predict, predict_proba, cost_history

# ==================== МЕТРИКИ И ОЦЕНКА ====================
class ModelEvaluation:
    """
    Метрики для оценки качества моделей
    """
    
    def regression_metrics(self, y_true, y_pred):
        """
        Метрики для регрессии
        """
        # Средняя абсолютная ошибка
        mae = np.mean(np.abs(y_true - y_pred))
        
        # Средняя квадратичная ошибка
        mse = np.mean((y_true - y_pred) ** 2)
        
        # Корень из средней квадратичной ошибки
        rmse = np.sqrt(mse)
        
        # Коэффициент детерминации R²
        ss_res = np.sum((y_true - y_pred) ** 2)
        ss_tot = np.sum((y_true - np.mean(y_true)) ** 2)
        r2 = 1 - (ss_res / ss_tot)
        
        return {
            'MAE': mae,
            'MSE': mse,
            'RMSE': rmse,
            'R²': r2
        }
    
    def classification_metrics(self, y_true, y_pred):
        """
        Метрики для классификации
        """
        # Матрица ошибок
        cm = confusion_matrix(y_true, y_pred)
        
        # Основные метрики
        accuracy = accuracy_score(y_true, y_pred)
        
        # Для бинарной классификации
        if len(np.unique(y_true)) == 2:
            tn, fp, fn, tp = cm.ravel()
            
            precision = tp / (tp + fp) if (tp + fp) > 0 else 0
            recall = tp / (tp + fn) if (tp + fn) > 0 else 0
            specificity = tn / (tn + fp) if (tn + fp) > 0 else 0
            
            f1_score = 2 * (precision * recall) / (precision + recall) if (precision + recall) > 0 else 0
            
            return {
                'accuracy': accuracy,
                'precision': precision,
                'recall': recall,
                'specificity': specificity,
                'f1_score': f1_score,
                'confusion_matrix': cm
            }
        
        return {
            'accuracy': accuracy,
            'confusion_matrix': cm
        }
    
    def cross_validation(self, X, y, model_class, k_folds=5):
        """
        K-кратная кросс-валидация
        
        Более надежная оценка качества модели
        """
        from sklearn.model_selection import KFold
        
        kf = KFold(n_splits=k_folds, shuffle=True, random_state=42)
        fold_scores = []
        
        for train_idx, val_idx in kf.split(X):
            # Разделение данных
            X_train, X_val = X[train_idx], X[val_idx]
            y_train, y_val = y[train_idx], y[val_idx]
            
            # Обучение модели
            model = model_class()
            model.fit(X_train, y_train)
            
            # Оценка на валидационной выборке
            y_pred = model.predict(X_val)
            score = accuracy_score(y_val, y_pred)
            fold_scores.append(score)
        
        return {
            'mean_score': np.mean(fold_scores),
            'std_score': np.std(fold_scores),
            'fold_scores': fold_scores
        }

# ==================== ПРАКТИЧЕСКИЙ ПРИМЕР ====================
def ml_pipeline_example():
    """
    Полный пример ML пайплайна
    """
    # Генерация данных
    X, y = make_classification(n_samples=1000, n_features=2, n_redundant=0, 
                              n_informative=2, n_clusters_per_class=1, random_state=42)
    
    # Разделение на обучающую и тестовую выборки
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # Создание и обучение модели
    sl = SupervisedLearning()
    weights, predict, predict_proba, cost_history = sl.logistic_regression_from_scratch(
        X_train, y_train, learning_rate=0.1, epochs=1000
    )
    
    # Предсказания
    y_pred = predict(X_test)
    y_pred_proba = predict_proba(X_test)
    
    # Оценка качества
    evaluator = ModelEvaluation()
    metrics = evaluator.classification_metrics(y_test, y_pred)
    
    print("Результаты логистической регрессии:")
    print(f"Точность: {metrics['accuracy']:.3f}")
    print(f"Precision: {metrics['precision']:.3f}")
    print(f"Recall: {metrics['recall']:.3f}")
    print(f"F1-score: {metrics['f1_score']:.3f}")
    
    # Визуализация границы решения
    def plot_decision_boundary(X, y, predict_func):
        """
        Визуализация границы решения
        """
        h = 0.02
        x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
        y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
        
        xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                            np.arange(y_min, y_max, h))
        
        mesh_points = np.c_[xx.ravel(), yy.ravel()]
        Z = predict_func(mesh_points)
        Z = Z.reshape(xx.shape)
        
        plt.figure(figsize=(10, 8))
        plt.contourf(xx, yy, Z, alpha=0.8, cmap=plt.cm.RdYlBu)
        scatter = plt.scatter(X[:, 0], X[:, 1], c=y, cmap=plt.cm.RdYlBu, edgecolors='black')
        plt.colorbar(scatter)
        plt.title('Граница решения логистической регрессии')
        plt.xlabel('Признак 1')
        plt.ylabel('Признак 2')
        plt.show()
    
    # Построение графика
    plot_decision_boundary(X_test, y_test, predict)
    
    return {
        'weights': weights,
        'metrics': metrics,
        'cost_history': cost_history
    }

# ==================== FEATURE ENGINEERING ====================
class FeatureEngineering:
    """
    Техники работы с признаками
    """
    
    def feature_scaling(self, X):
        """
        Масштабирование признаков
        
        Важно для алгоритмов, чувствительных к масштабу
        """
        # Стандартизация (Z-score): μ=0, σ=1
        standardized = (X - np.mean(X, axis=0)) / np.std(X, axis=0)
        
        # Нормализация (Min-Max): [0, 1]
        normalized = (X - np.min(X, axis=0)) / (np.max(X, axis=0) - np.min(X, axis=0))
        
        return {
            'standardized': standardized,
            'normalized': normalized
        }
    
    def polynomial_features(self, X, degree=2):
        """
        Создание полиномиальных признаков
        
        Увеличивает выразительность линейных моделей
        """
        from sklearn.preprocessing import PolynomialFeatures
        
        poly = PolynomialFeatures(degree=degree, include_bias=False)
        X_poly = poly.fit_transform(X)
        
        return X_poly, poly.get_feature_names_out()
    
    def handle_missing_values(self, X):
        """
        Обработка пропущенных значений
        """
        # Стратегии заполнения
        strategies = {
            'mean': np.nanmean(X, axis=0),
            'median': np.nanmedian(X, axis=0),
            'mode': np.array([np.bincount(col[~np.isnan(col)]).argmax() 
                             for col in X.T])
        }
        
        return strategies
```

---

## 🎯 Основные принципы ML

### Bias-Variance Tradeoff

```python
# ==================== BIAS-VARIANCE РАЗЛОЖЕНИЕ ====================
class BiasVarianceAnalysis:
    """
    Анализ компромисса между смещением и дисперсией
    """
    
    def simulate_bias_variance(self, n_experiments=100):
        """
        Симуляция bias-variance tradeoff
        
        Bias: ошибка из-за упрощающих предположений
        Variance: ошибка из-за чувствительности к тренировочным данным
        """
        from sklearn.ensemble import RandomForestRegressor
        from sklearn.linear_model import LinearRegression
        from sklearn.tree import DecisionTreeRegressor
        
        # Генерируем истинную функцию
        def true_function(x):
            return 1.5 * x + 0.3 * x**2 + 0.1 * x**3
        
        # Результаты экспериментов
        results = {
            'linear': [],
            'tree_depth_3': [],
            'tree_depth_10': [],
            'random_forest': []
        }
        
        # Проводим эксперименты
        for _ in range(n_experiments):
            # Генерируем данные с шумом
            X = np.random.uniform(-2, 2, 100).reshape(-1, 1)
            y = true_function(X.ravel()) + np.random.normal(0, 0.2, 100)
            
            # Тестовые данные
            X_test = np.linspace(-2, 2, 50).reshape(-1, 1)
            y_test_true = true_function(X_test.ravel())
            
            # Обучаем разные модели
            models = {
                'linear': LinearRegression(),
                'tree_depth_3': DecisionTreeRegressor(max_depth=3),
                'tree_depth_10': DecisionTreeRegressor(max_depth=10),
                'random_forest': RandomForestRegressor(n_estimators=100, max_depth=5)
            }
            
            for name, model in models.items():
                model.fit(X, y)
                y_pred = model.predict(X_test)
                results[name].append(y_pred)
        
        # Анализ результатов
        analysis = {}
        X_test = np.linspace(-2, 2, 50).reshape(-1, 1)
        y_test_true = true_function(X_test.ravel())
        
        for name, predictions in results.items():
            predictions = np.array(predictions)
            
            # Среднее предсказание
            mean_pred = np.mean(predictions, axis=0)
            
            # Bias² = (среднее предсказание - истинное значение)²
            bias_squared = np.mean((mean_pred - y_test_true)**2)
            
            # Variance = среднее от (предсказание - среднее предсказание)²
            variance = np.mean(np.var(predictions, axis=0))
            
            # Irreducible error (шум)
            noise = 0.2**2
            
            # Total error = Bias² + Variance + Noise
            total_error = bias_squared + variance + noise
            
            analysis[name] = {
                'bias_squared': bias_squared,
                'variance': variance,
                'noise': noise,
                'total_error': total_error
            }
        
        return analysis

# ==================== РЕГУЛЯРИЗАЦИЯ ====================
class Regularization:
    """
    Техники регуляризации для предотвращения переобучения
    """
    
    def ridge_regression(self, X, y, alpha=1.0):
        """
        Ridge регрессия (L2 регуляризация)
        
        Добавляет штраф за большие веса: Loss + α * ||w||²
        """
        # Добавляем bias
        X_with_bias = np.column_stack([np.ones(len(X)), X])
        
        # Аналитическое решение: w = (X^T*X + αI)^(-1) * X^T * y
        identity = np.eye(X_with_bias.shape[1])
        identity[0, 0] = 0  # Не регуляризуем bias
        
        weights = np.linalg.inv(X_with_bias.T @ X_with_bias + alpha * identity) @ X_with_bias.T @ y
        
        return weights
    
    def lasso_regression(self, X, y, alpha=1.0, max_iter=1000):
        """
        Lasso регрессия (L1 регуляризация)
        
        Добавляет штраф за количество признаков: Loss + α * ||w||₁
        Выполняет автоматический отбор признаков
        """
        # Координатный спуск для Lasso
        X_with_bias = np.column_stack([np.ones(len(X)), X])
        weights = np.zeros(X_with_bias.shape[1])
        
        for _ in range(max_iter):
            old_weights = weights.copy()
            
            for j in range(len(weights)):
                # Вычисляем остатки без j-го признака
                residuals = y - X_with_bias @ weights + weights[j] * X_with_bias[:, j]
                
                # Обновляем j-й вес
                rho = X_with_bias[:, j] @ residuals
                
                if j == 0:  # Bias не регуляризуем
                    weights[j] = rho / (X_with_bias[:, j] @ X_with_bias[:, j])
                else:
                    # Soft thresholding
                    z = X_with_bias[:, j] @ X_with_bias[:, j]
                    if rho > alpha:
                        weights[j] = (rho - alpha) / z
                    elif rho < -alpha:
                        weights[j] = (rho + alpha) / z
                    else:
                        weights[j] = 0
            
            # Проверка сходимости
            if np.allclose(weights, old_weights, rtol=1e-6):
                break
        
        return weights
    
    def elastic_net(self, X, y, alpha=1.0, l1_ratio=0.5):
        """
        Elastic Net: комбинация L1 и L2 регуляризации
        
        Loss + α * (l1_ratio * ||w||₁ + (1-l1_ratio) * ||w||²)
        """
        from sklearn.linear_model import ElasticNet
        
        model = ElasticNet(alpha=alpha, l1_ratio=l1_ratio)
        model.fit(X, y)
        
        return model.coef_, model.intercept_
```

---

## 📚 Ключевые концепции

### Обучение и валидация

```python
# ==================== СТРАТЕГИИ ВАЛИДАЦИИ ====================
class ValidationStrategies:
    """
    Различные стратегии валидации моделей
    """
    
    def holdout_validation(self, X, y, test_size=0.2):
        """
        Простое разделение на train/test
        """
        return train_test_split(X, y, test_size=test_size, random_state=42)
    
    def k_fold_cv(self, X, y, k=5):
        """
        K-кратная кросс-валидация
        """
        from sklearn.model_selection import KFold
        
        kf = KFold(n_splits=k, shuffle=True, random_state=42)
        return [(train_idx, val_idx) for train_idx, val_idx in kf.split(X)]
    
    def stratified_k_fold(self, X, y, k=5):
        """
        Стратифицированная кросс-валидация
        
        Сохраняет пропорции классов в каждой fold
        """
        from sklearn.model_selection import StratifiedKFold
        
        skf = StratifiedKFold(n_splits=k, shuffle=True, random_state=42)
        return [(train_idx, val_idx) for train_idx, val_idx in skf.split(X, y)]
    
    def time_series_split(self, X, y, n_splits=5):
        """
        Валидация для временных рядов
        
        Не нарушает временную структуру данных
        """
        from sklearn.model_selection import TimeSeriesSplit
        
        tss = TimeSeriesSplit(n_splits=n_splits)
        return [(train_idx, val_idx) for train_idx, val_idx in tss.split(X)]

# ==================== ENSEMBLE METHODS ====================
class EnsembleMethods:
    """
    Методы ансамблирования
    """
    
    def voting_classifier(self, X, y, X_test):
        """
        Голосующий классификатор
        
        Объединяет предсказания разных моделей
        """
        from sklearn.ensemble import VotingClassifier
        from sklearn.linear_model import LogisticRegression
        from sklearn.tree import DecisionTreeClassifier
        from sklearn.svm import SVC
        
        # Создаем базовые классификаторы
        clf1 = LogisticRegression(random_state=42)
        clf2 = DecisionTreeClassifier(random_state=42)
        clf3 = SVC(probability=True, random_state=42)
        
        # Голосующий классификатор
        voting_clf = VotingClassifier(
            estimators=[('lr', clf1), ('dt', clf2), ('svc', clf3)],
            voting='soft'  # Использует вероятности
        )
        
        voting_clf.fit(X, y)
        return voting_clf.predict(X_test)
    
    def bagging_example(self, X, y):
        """
        Bagging (Bootstrap Aggregating)
        
        Обучает модели на разных подвыборках
        """
        from sklearn.ensemble import BaggingClassifier
        from sklearn.tree import DecisionTreeClassifier
        
        bagging = BaggingClassifier(
            base_estimator=DecisionTreeClassifier(),
            n_estimators=100,
            random_state=42
        )
        
        bagging.fit(X, y)
        return bagging
    
    def boosting_example(self, X, y):
        """
        Boosting
        
        Последовательно обучает модели, исправляя ошибки предыдущих
        """
        from sklearn.ensemble import AdaBoostClassifier
        from sklearn.tree import DecisionTreeClassifier
        
        ada_boost = AdaBoostClassifier(
            base_estimator=DecisionTreeClassifier(max_depth=1),
            n_estimators=100,
            random_state=42
        )
        
        ada_boost.fit(X, y)
        return ada_boost

# ==================== ПРАКТИЧЕСКИЕ СОВЕТЫ ====================
ml_best_practices = {
    "Подготовка данных": [
        "Всегда исследуйте данные перед обучением",
        "Обрабатывайте пропущенные значения",
        "Масштабируйте признаки для алгоритмов на расстояниях",
        "Проверяйте на выбросы и аномалии"
    ],
    
    "Выбор модели": [
        "Начинайте с простых моделей",
        "Используйте кросс-валидацию для оценки",
        "Сравнивайте несколько алгоритмов",
        "Учитывайте интерпретируемость"
    ],
    
    "Избежание переобучения": [
        "Больше данных > сложные модели",
        "Используйте регуляризацию",
        "Раннее остановка при обучении",
        "Валидация на отдельной выборке"
    ],
    
    "Оценка качества": [
        "Используйте несколько метрик",
        "Анализируйте матрицу ошибок",
        "Проверяйте на разных подгруппах данных",
        "Тестируйте на реальных данных"
    ]
}
```

---

*Машинное обучение - это искусство извлечения закономерностей из данных* 🤖📊 