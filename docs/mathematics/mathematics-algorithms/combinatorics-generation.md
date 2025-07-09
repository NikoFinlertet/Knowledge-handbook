# Комбинаторика и генерация 🎯

> **Навигация**: [[statistics-data-analysis|← Статистика]] | [[mathematics-algorithms|Главная]]

## Содержание
1. [Комбинаторные алгоритмы](#🎯-комбинаторные-алгоритмы)
2. [Генерация комбинаций](#🔄-генерация-комбинаций)
3. [Практические применения](#🚀-практические-применения)

---

## 🎯 Комбинаторные алгоритмы

**Математическая основа:** Перестановки, размещения, сочетания

```python
import math
from itertools import permutations, combinations

class CombinatoricsAlgorithms:
    """Комбинаторные алгоритмы"""
    
    @staticmethod
    def factorial(n):
        """Факториал n!"""
        if n < 0:
            raise ValueError("Факториал определен только для неотрицательных чисел")
        if n == 0 or n == 1:
            return 1
        
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result
    
    @staticmethod
    def permutations_count(n, r=None):
        """Количество размещений P(n,r) = n!/(n-r)!"""
        if r is None:
            r = n
        if r > n or r < 0:
            return 0
        return CombinatoricsAlgorithms.factorial(n) // CombinatoricsAlgorithms.factorial(n - r)
    
    @staticmethod
    def combinations_count(n, r):
        """Количество сочетаний C(n,r) = n!/(r!(n-r)!)"""
        if r > n or r < 0:
            return 0
        if r == 0 or r == n:
            return 1
        
        # Оптимизация: C(n,r) = C(n,n-r)
        r = min(r, n - r)
        
        result = 1
        for i in range(r):
            result = result * (n - i) // (i + 1)
        return result
    
    @staticmethod
    def generate_permutations(arr):
        """Генерация всех перестановок массива"""
        def backtrack(current_perm, remaining):
            if not remaining:
                yield current_perm[:]
                return
            
            for i, item in enumerate(remaining):
                current_perm.append(item)
                yield from backtrack(current_perm, remaining[:i] + remaining[i+1:])
                current_perm.pop()
        
        yield from backtrack([], arr)
    
    @staticmethod
    def generate_combinations(arr, r):
        """Генерация всех сочетаний размера r"""
        def backtrack(current_comb, start_index):
            if len(current_comb) == r:
                yield current_comb[:]
                return
            
            for i in range(start_index, len(arr)):
                current_comb.append(arr[i])
                yield from backtrack(current_comb, i + 1)
                current_comb.pop()
        
        yield from backtrack([], 0)
    
    @staticmethod
    def next_permutation(arr):
        """
        Генерация следующей перестановки в лексикографическом порядке
        Алгоритм Нарайаны
        """
        arr = arr[:]  # Создаем копию
        
        # Шаг 1: Найти самый правый символ, который меньше следующего
        i = len(arr) - 2
        while i >= 0 and arr[i] >= arr[i + 1]:
            i -= 1
        
        if i == -1:
            return None  # Это последняя перестановка
        
        # Шаг 2: Найти потолок arr[i] справа от i
        j = len(arr) - 1
        while arr[j] <= arr[i]:
            j -= 1
        
        # Шаг 3: Поменять местами arr[i] и arr[j]
        arr[i], arr[j] = arr[j], arr[i]
        
        # Шаг 4: Развернуть суффикс начиная с i+1
        arr[i + 1:] = reversed(arr[i + 1:])
        
        return arr

class SubsetGeneration:
    """Генерация подмножеств"""
    
    @staticmethod
    def generate_all_subsets(arr):
        """Генерация всех подмножеств (включая пустое)"""
        n = len(arr)
        for i in range(2**n):
            subset = []
            for j in range(n):
                if i & (1 << j):
                    subset.append(arr[j])
            yield subset
    
    @staticmethod
    def generate_subsets_backtrack(arr):
        """Генерация подмножеств методом backtracking"""
        def backtrack(index, current_subset):
            if index == len(arr):
                yield current_subset[:]
                return
            
            # Не включаем текущий элемент
            yield from backtrack(index + 1, current_subset)
            
            # Включаем текущий элемент
            current_subset.append(arr[index])
            yield from backtrack(index + 1, current_subset)
            current_subset.pop()
        
        yield from backtrack(0, [])
    
    @staticmethod
    def k_subsets(arr, k):
        """Разбиение множества на k непустых подмножеств (числа Стирлинга)"""
        def stirling_partition(elements, k, current_partition):
            if not elements:
                if len(current_partition) == k:
                    yield [subset[:] for subset in current_partition]
                return
            
            element = elements[0]
            remaining = elements[1:]
            
            # Добавляем элемент в существующее подмножество
            for i, subset in enumerate(current_partition):
                subset.append(element)
                yield from stirling_partition(remaining, k, current_partition)
                subset.pop()
            
            # Создаем новое подмножество, если это возможно
            if len(current_partition) < k:
                current_partition.append([element])
                yield from stirling_partition(remaining, k, current_partition)
                current_partition.pop()
        
        yield from stirling_partition(arr, k, [])
```

---

## 🔄 Генерация комбинаций

```python
class AdvancedGeneration:
    """Продвинутые алгоритмы генерации"""
    
    @staticmethod
    def generate_binary_strings(n):
        """Генерация всех двоичных строк длины n"""
        for i in range(2**n):
            yield format(i, f'0{n}b')
    
    @staticmethod
    def generate_passwords(charset, length):
        """Генерация всех возможных паролей заданной длины"""
        def generate_recursive(current_password, remaining_length):
            if remaining_length == 0:
                yield current_password
                return
            
            for char in charset:
                yield from generate_recursive(current_password + char, remaining_length - 1)
        
        yield from generate_recursive("", length)
    
    @staticmethod
    def generate_partitions(n):
        """Генерация всех разбиений числа n"""
        def partition_helper(n, max_value=None):
            if max_value is None:
                max_value = n
            
            if n == 0:
                yield []
                return
            
            for i in range(min(max_value, n), 0, -1):
                for partition in partition_helper(n - i, i):
                    yield [i] + partition
        
        yield from partition_helper(n)
    
    @staticmethod
    def gray_code(n):
        """
        Генерация кода Грея для n битов
        Каждое следующее число отличается ровно в одном бите
        """
        if n == 0:
            return ['']
        
        smaller_gray = AdvancedGeneration.gray_code(n - 1)
        
        # Первая половина: добавляем '0' к началу
        first_half = ['0' + code for code in smaller_gray]
        
        # Вторая половина: добавляем '1' к началу в обратном порядке
        second_half = ['1' + code for code in reversed(smaller_gray)]
        
        return first_half + second_half
    
    @staticmethod
    def derangements(arr):
        """
        Генерация всех беспорядков (перестановки без неподвижных точек)
        D(n) = n! * Σ(k=0 to n) (-1)^k / k!
        """
        def is_derangement(perm, original):
            return all(perm[i] != original[i] for i in range(len(perm)))
        
        original_indices = list(range(len(arr)))
        
        for perm_indices in CombinatoricsAlgorithms.generate_permutations(original_indices):
            if is_derangement(perm_indices, original_indices):
                yield [arr[i] for i in perm_indices]

class CatalanNumbers:
    """Числа Каталана и связанные структуры"""
    
    @staticmethod
    def catalan_number(n):
        """n-е число Каталана: C_n = (2n)! / ((n+1)! * n!)"""
        if n <= 1:
            return 1
        
        return CombinatoricsAlgorithms.combinations_count(2*n, n) // (n + 1)
    
    @staticmethod
    def generate_dyck_words(n):
        """
        Генерация слов Дика длины 2n
        (правильные скобочные последовательности)
        """
        def generate_recursive(current, open_count, close_count):
            if len(current) == 2 * n:
                yield current
                return
            
            # Добавляем открывающую скобку
            if open_count < n:
                yield from generate_recursive(current + '(', open_count + 1, close_count)
            
            # Добавляем закрывающую скобку
            if close_count < open_count:
                yield from generate_recursive(current + ')', open_count, close_count + 1)
        
        yield from generate_recursive('', 0, 0)
    
    @staticmethod
    def generate_binary_trees(n):
        """
        Генерация всех полных двоичных деревьев с n внутренними узлами
        Количество = n-е число Каталана
        """
        if n == 0:
            yield None
            return
        
        for i in range(n):
            for left_tree in CatalanNumbers.generate_binary_trees(i):
                for right_tree in CatalanNumbers.generate_binary_trees(n - 1 - i):
                    yield {
                        'left': left_tree,
                        'right': right_tree,
                        'value': f'node_{n}_{i}'
                    }
```

---

## 🚀 Практические применения

```python
class PracticalApplications:
    """Практические применения комбинаторики"""
    
    @staticmethod
    def generate_test_cases(parameters):
        """
        Генерация тестовых случаев для параметризованного тестирования
        parameters: {'param1': [val1, val2], 'param2': [val3, val4]}
        """
        param_names = list(parameters.keys())
        param_values = list(parameters.values())
        
        def generate_combinations(index, current_combination):
            if index == len(param_names):
                yield dict(zip(param_names, current_combination))
                return
            
            for value in param_values[index]:
                yield from generate_combinations(index + 1, current_combination + [value])
        
        yield from generate_combinations(0, [])
    
    @staticmethod
    def load_balancing_configurations(servers, weights):
        """Генерация конфигураций для балансировки нагрузки"""
        total_servers = len(servers)
        
        # Генерируем все возможные распределения весов
        for weight_assignment in CombinatoricsAlgorithms.generate_permutations(weights):
            config = {}
            for i, server in enumerate(servers):
                config[server] = weight_assignment[i]
            yield config
    
    @staticmethod
    def feature_flag_combinations(features):
        """Генерация всех комбинаций feature flags"""
        for subset in SubsetGeneration.generate_all_subsets(features):
            config = {feature: feature in subset for feature in features}
            yield config
    
    @staticmethod
    def database_sharding_strategies(tables, shards):
        """Генерация стратегий шардирования базы данных"""
        # Простое распределение таблиц по шардам
        for assignment in CombinatoricsAlgorithms.generate_combinations(
            list(range(len(shards))), len(tables)
        ):
            strategy = {}
            for i, table in enumerate(tables):
                if i < len(assignment):
                    strategy[table] = shards[assignment[i]]
            if len(strategy) == len(tables):
                yield strategy

# Демонстрация использования
def demo_combinatorics():
    """Демонстрация комбинаторных алгоритмов"""
    
    # Тестирование комбинаций
    test_params = {
        'browser': ['chrome', 'firefox', 'safari'],
        'os': ['windows', 'mac', 'linux'],
        'device': ['desktop', 'mobile']
    }
    
    print("Тестовые конфигурации:")
    for i, config in enumerate(PracticalApplications.generate_test_cases(test_params)):
        if i < 5:  # Показываем первые 5
            print(f"  {config}")
    
    total_configs = CombinatoricsAlgorithms.combinations_count(3, 1) * \
                   CombinatoricsAlgorithms.combinations_count(3, 1) * \
                   CombinatoricsAlgorithms.combinations_count(2, 1)
    print(f"Всего конфигураций: {total_configs}")
    
    # Feature flags
    features = ['dark_mode', 'new_ui', 'analytics', 'beta_feature']
    print(f"\nКомбинации feature flags (всего {2**len(features)}):")
    
    for i, config in enumerate(PracticalApplications.feature_flag_combinations(features)):
        if i < 8:  # Показываем первые 8
            enabled_features = [f for f, enabled in config.items() if enabled]
            print(f"  Включены: {enabled_features}")
    
    # Числа Каталана
    print(f"\nЧисла Каталана:")
    for n in range(6):
        catalan_n = CatalanNumbers.catalan_number(n)
        print(f"  C_{n} = {catalan_n}")
    
    # Код Грея
    print(f"\n3-битный код Грея:")
    for code in AdvancedGeneration.gray_code(3):
        print(f"  {code}")
```

**Когда использовать комбинаторные алгоритмы:**

- ✅ **Генерация тестов**: Покрытие всех возможных комбинаций
- ✅ **Оптимизация**: Перебор конфигураций для поиска оптимальной
- ✅ **Криптография**: Генерация ключей, анализ стойкости
- ✅ **Игры**: Генерация уровней, головоломок

---

## 🔗 Связанные темы

- [[discrete-mathematics-algorithms|Дискретная математика]] - основы комбинаторики
- [[probability-randomized-algorithms|Вероятностные алгоритмы]] - случайная генерация
- [[graph-theory-structures|Теория графов]] - комбинаторная оптимизация
- [[number-theory-cryptography|Теория чисел]] - комбинаторная криптография

## 📚 Дополнительные ресурсы

- **Книги**: "Concrete Mathematics" - Knuth, Graham, Patashnik
- **Курсы**: MIT 6.042, Stanford CS103
- **Практика**: Project Euler, Codeforces combinatorics problems 