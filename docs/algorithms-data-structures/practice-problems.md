# Практические задачи

> **Сборник задач для тренировки алгоритмического мышления**
>
> От базовых до продвинутых с детальными решениями и объяснениями

## 🎯 Как использовать этот раздел

### Подход к решению задач
1. **Прочитать условие** - понять что требуется
2. **Проанализировать примеры** - найти закономерности
3. **Выбрать подход** - определить паттерн решения
4. **Оценить сложность** - время и память
5. **Написать код** - реализовать решение
6. **Протестировать** - проверить на примерах

---

## 📊 Задачи по уровням сложности

### 🟢 Легкие задачи (Easy)

#### Задача 1: Палиндром
```python
# ==================== ЗАДАЧА: ПРОВЕРКА ПАЛИНДРОМА ====================
# Проверить, является ли строка палиндромом (читается одинаково слева направо и справа налево)
# Учитывать только буквы и цифры, игнорировать регистр

def is_palindrome(s):
    """
    Проверка палиндрома
    
    Временная сложность: O(n)
    Пространственная сложность: O(1)
    
    Паттерн: Two Pointers
    """
    # Приводим к нижнему регистру и оставляем только буквы/цифры
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    
    # Два указателя с концов строки
    left, right = 0, len(cleaned) - 1
    
    while left < right:
        if cleaned[left] != cleaned[right]:
            return False
        left += 1
        right -= 1
    
    return True

# ==================== ТЕСТИРОВАНИЕ ====================
def test_palindrome():
    assert is_palindrome("A man, a plan, a canal: Panama") == True
    assert is_palindrome("race a car") == False
    assert is_palindrome("") == True
    assert is_palindrome("Madam") == True
    print("✅ Все тесты пройдены!")

# ==================== АЛЬТЕРНАТИВНОЕ РЕШЕНИЕ ====================
def is_palindrome_recursive(s):
    """
    Рекурсивное решение для палиндрома
    """
    def helper(left, right):
        # Пропускаем не буквенно-цифровые символы
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        
        # Базовый случай
        if left >= right:
            return True
        
        # Проверяем текущие символы
        if s[left].lower() != s[right].lower():
            return False
        
        return helper(left + 1, right - 1)
    
    return helper(0, len(s) - 1)
```

#### Задача 2: Анаграммы
```python
# ==================== ЗАДАЧА: ВАЛИДНЫЕ АНАГРАММЫ ====================
# Проверить, являются ли две строки анаграммами

def is_anagram(s, t):
    """
    Проверка анаграмм
    
    Временная сложность: O(n)
    Пространственная сложность: O(1) - фиксированный размер алфавита
    
    Паттерн: Hash Map (Counter)
    """
    if len(s) != len(t):
        return False
    
    # Подсчитываем частоту символов
    char_count = {}
    
    # Увеличиваем счетчик для первой строки
    for char in s:
        char_count[char] = char_count.get(char, 0) + 1
    
    # Уменьшаем счетчик для второй строки
    for char in t:
        if char not in char_count:
            return False
        char_count[char] -= 1
        if char_count[char] == 0:
            del char_count[char]
    
    return len(char_count) == 0

# ==================== РЕШЕНИЕ ЧЕРЕЗ СОРТИРОВКУ ====================
def is_anagram_sort(s, t):
    """
    Решение через сортировку
    
    Временная сложность: O(n log n)
    Пространственная сложность: O(1)
    """
    return sorted(s) == sorted(t)

# ==================== ТЕСТИРОВАНИЕ ====================
def test_anagram():
    assert is_anagram("anagram", "nagaram") == True
    assert is_anagram("rat", "car") == False
    assert is_anagram("listen", "silent") == True
    print("✅ Анаграммы: все тесты пройдены!")
```

### 🟡 Средние задачи (Medium)

#### Задача 3: Поиск в повернутом массиве
```python
# ==================== ЗАДАЧА: ПОИСК В ПОВЕРНУТОМ МАССИВЕ ====================
# Найти элемент в отсортированном массиве, который был повернут

def search_rotated_array(nums, target):
    """
    Поиск в повернутом отсортированном массиве
    
    Временная сложность: O(log n)
    Пространственная сложность: O(1)
    
    Паттерн: Modified Binary Search
    """
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            return mid
        
        # Определяем, какая половина отсортирована
        if nums[left] <= nums[mid]:
            # Левая половина отсортирована
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:
            # Правая половина отсортирована
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1

# ==================== ПОИСК МИНИМУМА В ПОВЕРНУТОМ МАССИВЕ ====================
def find_min_rotated(nums):
    """
    Поиск минимального элемента в повернутом массиве
    
    Временная сложность: O(log n)
    Пространственная сложность: O(1)
    """
    left, right = 0, len(nums) - 1
    
    while left < right:
        mid = (left + right) // 2
        
        if nums[mid] > nums[right]:
            # Минимум в правой половине
            left = mid + 1
        else:
            # Минимум в левой половине или mid - минимум
            right = mid
    
    return nums[left]

# ==================== ТЕСТИРОВАНИЕ ====================
def test_rotated_search():
    assert search_rotated_array([4,5,6,7,0,1,2], 0) == 4
    assert search_rotated_array([4,5,6,7,0,1,2], 3) == -1
    assert find_min_rotated([3,4,5,1,2]) == 1
    assert find_min_rotated([4,5,6,7,0,1,2]) == 0
    print("✅ Поиск в повернутом массиве: все тесты пройдены!")
```

#### Задача 4: Группировка анаграмм
```python
# ==================== ЗАДАЧА: ГРУППИРОВКА АНАГРАММ ====================
# Сгруппировать анаграммы из массива строк

def group_anagrams(strs):
    """
    Группировка анаграмм
    
    Временная сложность: O(n * m log m), где n - количество строк, m - средняя длина строки
    Пространственная сложность: O(n * m)
    
    Паттерн: Hash Map с сортировкой как ключ
    """
    from collections import defaultdict
    
    anagram_groups = defaultdict(list)
    
    for s in strs:
        # Сортированная строка как ключ
        key = ''.join(sorted(s))
        anagram_groups[key].append(s)
    
    return list(anagram_groups.values())

# ==================== АЛЬТЕРНАТИВНОЕ РЕШЕНИЕ ====================
def group_anagrams_count(strs):
    """
    Группировка анаграмм через подсчет символов
    
    Временная сложность: O(n * m)
    Пространственная сложность: O(n * m)
    """
    from collections import defaultdict
    
    anagram_groups = defaultdict(list)
    
    for s in strs:
        # Создаем ключ из частот символов
        char_count = [0] * 26
        for char in s:
            char_count[ord(char) - ord('a')] += 1
        
        # Преобразуем в кортеж для использования как ключ
        key = tuple(char_count)
        anagram_groups[key].append(s)
    
    return list(anagram_groups.values())

# ==================== ТЕСТИРОВАНИЕ ====================
def test_group_anagrams():
    result = group_anagrams(["eat","tea","tan","ate","nat","bat"])
    expected = [["eat","tea","ate"], ["tan","nat"], ["bat"]]
    
    # Сортируем для корректного сравнения
    result_sorted = [sorted(group) for group in result]
    expected_sorted = [sorted(group) for group in expected]
    
    assert sorted(result_sorted) == sorted(expected_sorted)
    print("✅ Группировка анаграмм: все тесты пройдены!")
```

### 🔴 Сложные задачи (Hard)

#### Задача 5: Медиана двух отсортированных массивов
```python
# ==================== ЗАДАЧА: МЕДИАНА ДВУХ ОТСОРТИРОВАННЫХ МАССИВОВ ====================
# Найти медиану двух отсортированных массивов за O(log(min(m,n)))

def find_median_sorted_arrays(nums1, nums2):
    """
    Медиана двух отсортированных массивов
    
    Временная сложность: O(log(min(m, n)))
    Пространственная сложность: O(1)
    
    Паттерн: Binary Search on Answer
    """
    # Гарантируем, что nums1 меньше nums2
    if len(nums1) > len(nums2):
        nums1, nums2 = nums2, nums1
    
    m, n = len(nums1), len(nums2)
    left, right = 0, m
    
    while left <= right:
        # Разделяем nums1 и nums2
        partition_x = (left + right) // 2
        partition_y = (m + n + 1) // 2 - partition_x
        
        # Находим граничные элементы
        max_left_x = float('-inf') if partition_x == 0 else nums1[partition_x - 1]
        min_right_x = float('inf') if partition_x == m else nums1[partition_x]
        
        max_left_y = float('-inf') if partition_y == 0 else nums2[partition_y - 1]
        min_right_y = float('inf') if partition_y == n else nums2[partition_y]
        
        # Проверяем правильность разделения
        if max_left_x <= min_right_y and max_left_y <= min_right_x:
            # Нашли правильное разделение
            if (m + n) % 2 == 0:
                return (max(max_left_x, max_left_y) + min(min_right_x, min_right_y)) / 2
            else:
                return max(max_left_x, max_left_y)
        elif max_left_x > min_right_y:
            # Слишком много элементов из nums1
            right = partition_x - 1
        else:
            # Слишком мало элементов из nums1
            left = partition_x + 1
    
    return -1

# ==================== ПРОСТОЕ РЕШЕНИЕ O(m + n) ====================
def find_median_simple(nums1, nums2):
    """
    Простое решение через слияние
    
    Временная сложность: O(m + n)
    Пространственная сложность: O(m + n)
    """
    merged = []
    i = j = 0
    
    # Сливаем два массива
    while i < len(nums1) and j < len(nums2):
        if nums1[i] <= nums2[j]:
            merged.append(nums1[i])
            i += 1
        else:
            merged.append(nums2[j])
            j += 1
    
    # Добавляем оставшиеся элементы
    merged.extend(nums1[i:])
    merged.extend(nums2[j:])
    
    n = len(merged)
    if n % 2 == 0:
        return (merged[n // 2 - 1] + merged[n // 2]) / 2
    else:
        return merged[n // 2]

# ==================== ТЕСТИРОВАНИЕ ====================
def test_median():
    assert find_median_sorted_arrays([1, 3], [2]) == 2.0
    assert find_median_sorted_arrays([1, 2], [3, 4]) == 2.5
    assert find_median_sorted_arrays([0, 0], [0, 0]) == 0.0
    print("✅ Медиана двух массивов: все тесты пройдены!")
```

---

## 🧩 Задачи по паттернам

### Two Pointers Pattern

#### Задача 6: Container With Most Water
```python
# ==================== ЗАДАЧА: КОНТЕЙНЕР С НАИБОЛЬШИМ КОЛИЧЕСТВОМ ВОДЫ ====================
# Найти две линии, которые вместе с осью x образуют контейнер с наибольшим объемом воды

def max_area(height):
    """
    Контейнер с максимальной площадью
    
    Временная сложность: O(n)
    Пространственная сложность: O(1)
    
    Паттерн: Two Pointers
    """
    left, right = 0, len(height) - 1
    max_water = 0
    
    while left < right:
        # Вычисляем текущую площадь
        width = right - left
        current_height = min(height[left], height[right])
        current_area = width * current_height
        
        max_water = max(max_water, current_area)
        
        # Двигаем указатель с меньшей высотой
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water

# ==================== ТЕСТИРОВАНИЕ ====================
def test_max_area():
    assert max_area([1,8,6,2,5,4,8,3,7]) == 49
    assert max_area([1,1]) == 1
    assert max_area([4,3,2,1,4]) == 16
    print("✅ Container With Most Water: все тесты пройдены!")
```

### Sliding Window Pattern

#### Задача 7: Minimum Window Substring
```python
# ==================== ЗАДАЧА: МИНИМАЛЬНАЯ ПОДСТРОКА С ОКНОМ ====================
# Найти минимальную подстроку в s, которая содержит все символы из t

def min_window(s, t):
    """
    Минимальная подстрока-окно
    
    Временная сложность: O(|s| + |t|)
    Пространственная сложность: O(|s| + |t|)
    
    Паттерн: Sliding Window
    """
    if not s or not t:
        return ""
    
    # Подсчитываем частоты символов в t
    dict_t = {}
    for char in t:
        dict_t[char] = dict_t.get(char, 0) + 1
    
    required = len(dict_t)  # Количество уникальных символов в t
    
    # Скользящее окно
    left = right = 0
    formed = 0  # Количество символов с нужной частотой
    
    window_counts = {}
    
    # Результат: (длина окна, левый индекс, правый индекс)
    ans = float("inf"), None, None
    
    while right < len(s):
        # Добавляем символ справа в окно
        character = s[right]
        window_counts[character] = window_counts.get(character, 0) + 1
        
        # Если частота текущего символа достигла нужной
        if character in dict_t and window_counts[character] == dict_t[character]:
            formed += 1
        
        # Пытаемся сжать окно, пока оно валидное
        while left <= right and formed == required:
            character = s[left]
            
            # Сохраняем самое маленькое окно
            if right - left + 1 < ans[0]:
                ans = (right - left + 1, left, right)
            
            # Символ слева больше не входит в окно
            window_counts[character] -= 1
            if character in dict_t and window_counts[character] < dict_t[character]:
                formed -= 1
            
            left += 1
        
        right += 1
    
    return "" if ans[0] == float("inf") else s[ans[1]:ans[2] + 1]

# ==================== ТЕСТИРОВАНИЕ ====================
def test_min_window():
    assert min_window("ADOBECODEBANC", "ABC") == "BANC"
    assert min_window("a", "a") == "a"
    assert min_window("a", "aa") == ""
    print("✅ Minimum Window Substring: все тесты пройдены!")
```

---

## 🌳 Задачи на деревья

#### Задача 8: Serialize and Deserialize Binary Tree
```python
# ==================== ЗАДАЧА: СЕРИАЛИЗАЦИЯ И ДЕСЕРИАЛИЗАЦИЯ ДЕРЕВА ====================
# Сериализовать и десериализовать бинарное дерево

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Codec:
    def serialize(self, root):
        """
        Сериализация дерева в строку
        
        Временная сложность: O(n)
        Пространственная сложность: O(n)
        
        Паттерн: DFS Preorder
        """
        def dfs(node):
            if not node:
                return "null"
            
            # Preorder: корень -> левое -> правое
            return str(node.val) + "," + dfs(node.left) + "," + dfs(node.right)
        
        return dfs(root)
    
    def deserialize(self, data):
        """
        Десериализация строки в дерево
        
        Временная сложность: O(n)
        Пространственная сложность: O(n)
        """
        def dfs():
            val = next(values)
            if val == "null":
                return None
            
            node = TreeNode(int(val))
            node.left = dfs()
            node.right = dfs()
            return node
        
        values = iter(data.split(","))
        return dfs()

# ==================== АЛЬТЕРНАТИВНОЕ РЕШЕНИЕ ЧЕРЕЗ BFS ====================
class CodecBFS:
    def serialize(self, root):
        """Сериализация через BFS (Level Order)"""
        if not root:
            return ""
        
        result = []
        queue = [root]
        
        while queue:
            node = queue.pop(0)
            if node:
                result.append(str(node.val))
                queue.append(node.left)
                queue.append(node.right)
            else:
                result.append("null")
        
        return ",".join(result)
    
    def deserialize(self, data):
        """Десериализация через BFS"""
        if not data:
            return None
        
        values = data.split(",")
        root = TreeNode(int(values[0]))
        queue = [root]
        i = 1
        
        while queue and i < len(values):
            node = queue.pop(0)
            
            if values[i] != "null":
                node.left = TreeNode(int(values[i]))
                queue.append(node.left)
            i += 1
            
            if i < len(values) and values[i] != "null":
                node.right = TreeNode(int(values[i]))
                queue.append(node.right)
            i += 1
        
        return root

# ==================== ТЕСТИРОВАНИЕ ====================
def test_serialize():
    # Создаем дерево: [1, 2, 3, null, null, 4, 5]
    root = TreeNode(1)
    root.left = TreeNode(2)
    root.right = TreeNode(3)
    root.right.left = TreeNode(4)
    root.right.right = TreeNode(5)
    
    codec = Codec()
    serialized = codec.serialize(root)
    deserialized = codec.deserialize(serialized)
    
    # Проверяем, что дерево восстановилось корректно
    assert deserialized.val == 1
    assert deserialized.left.val == 2
    assert deserialized.right.val == 3
    print("✅ Сериализация дерева: все тесты пройдены!")
```

---

## 🎯 Система тестирования

### Автоматический тестировщик
```python
# ==================== СИСТЕМА ТЕСТИРОВАНИЯ ====================

class TestRunner:
    def __init__(self):
        self.passed = 0
        self.failed = 0
        self.test_results = []
    
    def run_test(self, test_name, test_func):
        """Запуск отдельного теста"""
        try:
            test_func()
            self.passed += 1
            self.test_results.append(f"✅ {test_name}: PASSED")
            print(f"✅ {test_name}: PASSED")
        except Exception as e:
            self.failed += 1
            self.test_results.append(f"❌ {test_name}: FAILED - {str(e)}")
            print(f"❌ {test_name}: FAILED - {str(e)}")
    
    def run_all_tests(self):
        """Запуск всех тестов"""
        print("🚀 Запуск всех тестов...\n")
        
        # Список всех тестов
        tests = [
            ("Palindrome", test_palindrome),
            ("Anagram", test_anagram),
            ("Rotated Search", test_rotated_search),
            ("Group Anagrams", test_group_anagrams),
            ("Median Arrays", test_median),
            ("Max Area", test_max_area),
            ("Min Window", test_min_window),
            ("Serialize Tree", test_serialize),
        ]
        
        for test_name, test_func in tests:
            self.run_test(test_name, test_func)
        
        print(f"\n📊 Результаты:")
        print(f"✅ Пройдено: {self.passed}")
        print(f"❌ Провалено: {self.failed}")
        print(f"📈 Процент успеха: {(self.passed / (self.passed + self.failed)) * 100:.1f}%")

# ==================== ЗАПУСК ТЕСТОВ ====================
if __name__ == "__main__":
    runner = TestRunner()
    runner.run_all_tests()
```

---

## 📚 Полезные паттерны для решения

### Checklist перед решением задачи
```python
# ==================== ЧЕКЛИСТ РЕШЕНИЯ ЗАДАЧ ====================

def problem_solving_checklist():
    """
    Систематический подход к решению задач
    """
    checklist = [
        "1. 📖 Понять условие задачи",
        "2. 🔍 Разобрать примеры входных/выходных данных",
        "3. 🤔 Определить паттерн решения",
        "4. ⏰ Оценить временную сложность",
        "5. 💾 Оценить пространственную сложность",
        "6. 🛠️ Написать базовое решение",
        "7. 🧪 Протестировать на примерах",
        "8. 🔧 Оптимизировать если нужно",
        "9. 🎯 Учесть краевые случаи",
        "10. ✅ Финальное тестирование"
    ]
    
    for item in checklist:
        print(item)
    
    return checklist

# ==================== ТИПИЧНЫЕ ОШИБКИ И КАК ИХ ИЗБЕЖАТЬ ====================
def common_mistakes():
    """
    Частые ошибки при решении задач
    """
    mistakes = {
        "Off-by-one errors": "Внимательно с границами циклов и индексами",
        "Null pointer exceptions": "Всегда проверяйте на None/null",
        "Integer overflow": "Учитывайте переполнение для больших чисел",
        "Wrong complexity analysis": "Правильно считайте вложенные циклы",
        "Не учитывать edge cases": "Пустые массивы, одиночные элементы",
        "Модификация во время итерации": "Не изменяйте структуру во время обхода",
        "Неправильная инициализация": "Проверяйте начальные значения переменных"
    }
    
    for mistake, solution in mistakes.items():
        print(f"❌ {mistake}: {solution}")
    
    return mistakes
```

---

## 🏆 Прогресс-трекер

### Отслеживание решенных задач
```python
# ==================== ТРЕКЕР ПРОГРЕССА ====================

class ProgressTracker:
    def __init__(self):
        self.solved_problems = {
            'easy': [],
            'medium': [],
            'hard': []
        }
        self.patterns_mastered = set()
        self.time_spent = 0
    
    def mark_solved(self, problem_name, difficulty, pattern, time_minutes):
        """Отметить задачу как решенную"""
        self.solved_problems[difficulty].append({
            'name': problem_name,
            'pattern': pattern,
            'time': time_minutes
        })
        self.patterns_mastered.add(pattern)
        self.time_spent += time_minutes
    
    def get_stats(self):
        """Получить статистику"""
        total_solved = sum(len(problems) for problems in self.solved_problems.values())
        
        stats = {
            'total_solved': total_solved,
            'easy_solved': len(self.solved_problems['easy']),
            'medium_solved': len(self.solved_problems['medium']),
            'hard_solved': len(self.solved_problems['hard']),
            'patterns_mastered': len(self.patterns_mastered),
            'total_time_hours': self.time_spent / 60
        }
        
        return stats
    
    def print_progress(self):
        """Вывести прогресс"""
        stats = self.get_stats()
        
        print("📊 Ваш прогресс:")
        print(f"✅ Всего решено: {stats['total_solved']}")
        print(f"🟢 Easy: {stats['easy_solved']}")
        print(f"🟡 Medium: {stats['medium_solved']}")
        print(f"🔴 Hard: {stats['hard_solved']}")
        print(f"🧩 Освоено паттернов: {stats['patterns_mastered']}")
        print(f"⏰ Время изучения: {stats['total_time_hours']:.1f} часов")

# Пример использования
tracker = ProgressTracker()
tracker.mark_solved("Two Sum", "easy", "Hash Map", 15)
tracker.mark_solved("Longest Substring", "medium", "Sliding Window", 45)
tracker.print_progress()
```

---

*Практика — лучший учитель алгоритмов. Решайте задачи каждый день!* 💪🧠 