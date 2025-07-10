# Байесовская статистика 🔮

> [[practical-applications|← Практические применения]] | [[README|Оглавление]] | [[advanced-methods|Продвинутые методы →]]

## 🎯 Что изучим

- **Теорема Байеса** - обновление убеждений на основе новых данных
- **Байесовский вывод** - альтернатива классической статистике
- **Практические применения** - рекомендательные системы, spam-фильтры

---

## 1. Основы байесовского подхода

### 🧠 Теорема Байеса

```python
class BayesianAnalyzer:
    def bayes_theorem(self, prior, likelihood, evidence):
        """P(H|E) = P(E|H) * P(H) / P(E)"""
        if evidence == 0:
            return 0
        return (likelihood * prior) / evidence
    
    def spam_filter(self, email_words, spam_prior=0.3):
        """Байесовский спам-фильтр"""
        word_probs = {
            'free': {'spam': 0.8, 'ham': 0.1},
            'money': {'spam': 0.7, 'ham': 0.05},
            'meeting': {'spam': 0.1, 'ham': 0.6},
            'project': {'spam': 0.2, 'ham': 0.7}
        }
        
        likelihood_spam = 1.0
        likelihood_ham = 1.0
        
        for word in email_words:
            if word in word_probs:
                likelihood_spam *= word_probs[word]['spam']
                likelihood_ham *= word_probs[word]['ham']
        
        # Evidence (нормализация)
        evidence = likelihood_spam * spam_prior + likelihood_ham * (1 - spam_prior)
        
        # Апостериорные вероятности
        posterior_spam = self.bayes_theorem(spam_prior, likelihood_spam, evidence)
        
        return {
            'spam_probability': posterior_spam,
            'classification': 'SPAM' if posterior_spam > 0.5 else 'HAM',
            'confidence': max(posterior_spam, 1 - posterior_spam)
        }

# Пример использования
bayesian = BayesianAnalyzer()

email1 = ['free', 'money']
email2 = ['meeting', 'project']

result1 = bayesian.spam_filter(email1)
result2 = bayesian.spam_filter(email2)

print(f"Email 1: {result1['classification']} (p={result1['spam_probability']:.3f})")
print(f"Email 2: {result2['classification']} (p={result2['spam_probability']:.3f})")
```

---

## 2. Байесовское обновление

```python
class BayesianUpdater:
    def __init__(self, initial_belief=0.5):
        self.belief = initial_belief
        self.history = []
    
    def update_with_evidence(self, evidence_supports, likelihood=0.8):
        """Обновление убеждения новыми данными"""
        prior = self.belief
        
        if evidence_supports:
            # Evidence поддерживает гипотезу
            posterior = (likelihood * prior) / (likelihood * prior + (1 - likelihood) * (1 - prior))
        else:
            # Evidence противоречит гипотезе
            posterior = ((1 - likelihood) * prior) / ((1 - likelihood) * prior + likelihood * (1 - prior))
        
        self.history.append({
            'prior': prior,
            'evidence': evidence_supports,
            'posterior': posterior,
            'change': posterior - prior
        })
        
        self.belief = posterior
        return posterior
    
    def get_confidence(self):
        """Уверенность в текущем убеждении"""
        return abs(self.belief - 0.5) * 2

# Пример: Байесовский A/B тест
class BayesianABTest:
    def __init__(self):
        self.updater = BayesianUpdater(0.5)  # 50% - B лучше A
        self.data = {'a': [], 'b': []}
    
    def add_conversion(self, variant, converted):
        """Добавление результата конверсии"""
        self.data[variant].append(converted)
        
        # Обновляем убеждение если есть данные для обеих групп
        if len(self.data['a']) > 0 and len(self.data['b']) > 0:
            rate_a = sum(self.data['a']) / len(self.data['a'])
            rate_b = sum(self.data['b']) / len(self.data['b'])
            
            # Простая эвристика: B лучше если rate_b > rate_a
            evidence_b_better = rate_b > rate_a
            self.updater.update_with_evidence(evidence_b_better)
        
        return {
            'belief_b_better': self.updater.belief,
            'confidence': self.updater.get_confidence(),
            'recommendation': self.get_recommendation()
        }
    
    def get_recommendation(self):
        """Рекомендация на основе убеждения"""
        belief = self.updater.belief
        confidence = self.updater.get_confidence()
        
        if belief > 0.8 and confidence > 0.6:
            return "ВНЕДРИТЬ B: Сильные доказательства"
        elif belief > 0.7:
            return "ВЕРОЯТНО B: Умеренные доказательства"
        elif belief < 0.3 and confidence > 0.6:
            return "ОСТАТЬСЯ С A: B хуже"
        else:
            return "ПРОДОЛЖИТЬ ТЕСТ: Недостаточно данных"

# Симуляция A/B теста
import random
random.seed(42)

ab_test = BayesianABTest()

print("Байесовский A/B тест:")
for i in range(20):
    variant = 'a' if i % 2 == 0 else 'b'
    # B немного лучше (6% vs 5% конверсия)
    conversion_rate = 0.05 if variant == 'a' else 0.06
    converted = random.random() < conversion_rate
    
    result = ab_test.add_conversion(variant, converted)
    
    if i % 4 == 3:  # Выводим каждые 4 итерации
        print(f"Шаг {i+1}: Убеждение B лучше = {result['belief_b_better']:.3f}")
        print(f"         Рекомендация: {result['recommendation']}")

print(f"\nФинальное убеждение: {ab_test.updater.belief:.3f}")
print(f"Уверенность: {ab_test.updater.get_confidence():.3f}")
```

---

## 3. Байесовские сети

```python
class SimpleBayesNet:
    def __init__(self):
        self.nodes = {}
        self.dependencies = {}
    
    def add_node(self, name, states, probs):
        """Добавление узла"""
        self.nodes[name] = {'states': states, 'probs': probs}
    
    def add_dependency(self, child, parent, conditional_table):
        """Добавление зависимости"""
        if child not in self.dependencies:
            self.dependencies[child] = {}
        self.dependencies[child][parent] = conditional_table
    
    def predict(self, evidence, target):
        """Простое предсказание"""
        if target not in self.nodes:
            return {}
        
        results = {}
        for state in self.nodes[target]['states']:
            prob = self._calc_probability(target, state, evidence)
            results[state] = prob
        
        # Нормализация
        total = sum(results.values())
        if total > 0:
            results = {k: v/total for k, v in results.items()}
        
        return results
    
    def _calc_probability(self, var, state, evidence):
        """Вычисление вероятности состояния"""
        base_prob = self.nodes[var]['probs'].get(state, 0.1)
        
        # Учитываем зависимости
        for parent in self.dependencies.get(var, {}):
            if parent in evidence:
                parent_state = evidence[parent]
                conditional_table = self.dependencies[var][parent]
                if parent_state in conditional_table:
                    base_prob *= conditional_table[parent_state].get(state, 0.1)
        
        return base_prob

# Пример: сеть для e-commerce
ecommerce_net = SimpleBayesNet()

# Узлы
ecommerce_net.add_node('device', ['mobile', 'desktop'], 
                      {'mobile': 0.6, 'desktop': 0.4})

ecommerce_net.add_node('time', ['day', 'night'], 
                      {'day': 0.7, 'night': 0.3})

ecommerce_net.add_node('engagement', ['low', 'high'], 
                      {'low': 0.6, 'high': 0.4})

ecommerce_net.add_node('purchase', ['yes', 'no'], 
                      {'yes': 0.1, 'no': 0.9})

# Зависимости
ecommerce_net.add_dependency('engagement', 'device', {
    'mobile': {'low': 0.7, 'high': 0.3},
    'desktop': {'low': 0.4, 'high': 0.6}
})

ecommerce_net.add_dependency('engagement', 'time', {
    'day': {'low': 0.5, 'high': 0.5},
    'night': {'low': 0.8, 'high': 0.2}
})

ecommerce_net.add_dependency('purchase', 'engagement', {
    'low': {'yes': 0.05, 'no': 0.95},
    'high': {'yes': 0.25, 'no': 0.75}
})

# Предсказания
desktop_day = ecommerce_net.predict(
    evidence={'device': 'desktop', 'time': 'day'}, 
    target='purchase'
)

mobile_night = ecommerce_net.predict(
    evidence={'device': 'mobile', 'time': 'night'}, 
    target='purchase'
)

print("Байесовская сеть e-commerce:")
print(f"Покупка (десктоп днем): {desktop_day}")
print(f"Покупка (мобайл ночью): {mobile_night}")
```

---

## 📚 Практические применения

### 🎯 Рекомендательные системы
- Персонализация контента
- Collaborative filtering
- Content-based рекомендации

### 🛡️ Системы безопасности  
- Обнаружение аномалий
- Fraud detection
- Spam фильтрация

### 📊 Бизнес-аналитика
- A/B тестирование
- Customer segmentation
- Прогнозирование churn

---

## 🔗 Связанные темы

- [[probability-theory|Основы теории вероятностей]] - математическая база
- [[statistical-tests|Статистические тесты]] - классические методы
- [[practical-applications|Практические применения]] - реальные кейсы

---

## 💡 Ключевые преимущества

- **Инкрементальное обучение** - модели обновляются с новыми данными
- **Учёт неопределённости** - явное моделирование uncertainty  
- **Интерпретируемость** - понятная логика решений
- **Работа с малыми выборками** - эффективен даже при ограниченных данных

> 🔮 **Байесовский подход**: "Все модели неправильные, но некоторые обновляемые!" 