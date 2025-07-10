# FAANG Подготовка

> **Топ-150 задач для успешного поступления в крупные технологические компании**
>
> Систематическая подготовка к Google, Meta, Amazon, Apple, Netflix

## 🎯 Стратегия подготовки FAANG

### Временные рамки
- **3-6 месяцев:** Интенсивная подготовка
- **6-12 месяцев:** Комфортная подготовка
- **12+ месяцев:** Подготовка с нуля

### Распределение времени
- **60% - Решение задач** (LeetCode, HackerRank)
- **25% - Изучение теории** (алгоритмы, структуры данных)
- **10% - Mock интервью** (Pramp, Interviewing.io)
- **5% - System Design** (для senior позиций)

---

## 📊 Анализ задач по компаниям

### Google (частота встречаемости)
1. **Графы и деревья** - 35%
2. **Dynamic Programming** - 25%
3. **Строки и массивы** - 20%
4. **Математические алгоритмы** - 15%
5. **Системное проектирование** - 5%

### Meta (Facebook)
1. **Строки и массивы** - 30%
2. **Графы** - 25%
3. **Dynamic Programming** - 20%
4. **Деревья** - 15%
5. **Хэширование** - 10%

### Amazon
1. **Массивы и строки** - 30%
2. **Деревья** - 25%
3. **Графы** - 20%
4. **Leadership Principles** - 15%
5. **System Design** - 10%

---

## 🏆 Топ-150 задач FAANG

### 📈 Уровень Easy (30 задач)

#### Массивы и строки
```python
# ==================== 1. TWO SUM ====================
# Компании: Google, Amazon, Facebook, Apple
# Частота: Очень высокая
# Паттерн: Hash Map

def two_sum(nums, target):
    """
    Найти индексы двух чисел, сумма которых равна target
    
    Временная сложность: O(n)
    Пространственная сложность: O(n)
    """
    num_map = {}
    
    for i, num in enumerate(nums):
        complement = target - num
        if complement in num_map:
            return [num_map[complement], i]
        num_map[num] = i
    
    return []

# ==================== 2. VALID PARENTHESES ====================
# Компании: Google, Amazon, Facebook
# Частота: Очень высокая
# Паттерн: Stack

def is_valid_parentheses(s):
    """
    Проверить корректность скобок
    
    Временная сложность: O(n)
    Пространственная сложность: O(n)
    """
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            stack.append(char)
    
    return len(stack) == 0

# ==================== 3. MERGE TWO SORTED LISTS ====================
# Компании: Google, Amazon, Facebook, Apple
# Частота: Высокая
# Паттерн: Two Pointers

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def merge_two_lists(l1, l2):
    """
    Слияние двух отсортированных списков
    
    Временная сложность: O(n + m)
    Пространственная сложность: O(1)
    """
    dummy = ListNode(0)
    current = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next
    
    # Добавляем оставшиеся элементы
    current.next = l1 or l2
    
    return dummy.next
```

### 🔥 Уровень Medium (70 задач)

#### Самые важные задачи Medium

```python
# ==================== 1. LONGEST SUBSTRING WITHOUT REPEATING ====================
# Компании: Google, Amazon, Facebook, Apple
# Частота: Очень высокая
# Паттерн: Sliding Window

def length_of_longest_substring(s):
    """
    Длина самой длинной подстроки без повторений
    
    Временная сложность: O(n)
    Пространственная сложность: O(min(m, n))
    """
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        
        char_set.add(s[right])
        max_length = max(max_length, right - left + 1)
    
    return max_length

# ==================== 2. GROUP ANAGRAMS ====================
# Компании: Google, Amazon, Facebook
# Частота: Высокая
# Паттерн: Hash Map

def group_anagrams(strs):
    """
    Группировка анаграмм
    
    Временная сложность: O(n * m log m)
    Пространственная сложность: O(n * m)
    """
    from collections import defaultdict
    
    anagram_groups = defaultdict(list)
    
    for s in strs:
        # Сортируем символы как ключ
        key = ''.join(sorted(s))
        anagram_groups[key].append(s)
    
    return list(anagram_groups.values())

# ==================== 3. MAXIMUM SUBARRAY ====================
# Компании: Google, Amazon, Apple
# Частота: Очень высокая
# Паттерн: Kadane's Algorithm

def max_subarray(nums):
    """
    Максимальная сумма подмассива (Kadane's Algorithm)
    
    Временная сложность: O(n)
    Пространственная сложность: O(1)
    """
    max_ending_here = max_so_far = nums[0]
    
    for i in range(1, len(nums)):
        # Либо продолжаем текущий подмассив, либо начинаем новый
        max_ending_here = max(nums[i], max_ending_here + nums[i])
        max_so_far = max(max_so_far, max_ending_here)
    
    return max_so_far
```

### 🚀 Уровень Hard (50 задач)

#### Топ Hard задачи

```python
# ==================== 1. MEDIAN OF TWO SORTED ARRAYS ====================
# Компании: Google, Apple, Facebook
# Частота: Высокая
# Паттерн: Binary Search

def find_median_sorted_arrays(nums1, nums2):
    """
    Медиана двух отсортированных массивов
    
    Временная сложность: O(log(min(m, n)))
    Пространственная сложность: O(1)
    """
    # Гарантируем, что nums1 меньше nums2
    if len(nums1) > len(nums2):
        nums1, nums2 = nums2, nums1
    
    m, n = len(nums1), len(nums2)
    left, right = 0, m
    
    while left <= right:
        partition_x = (left + right) // 2
        partition_y = (m + n + 1) // 2 - partition_x
        
        # Элементы слева от разделения
        max_left_x = float('-inf') if partition_x == 0 else nums1[partition_x - 1]
        max_left_y = float('-inf') if partition_y == 0 else nums2[partition_y - 1]
        
        # Элементы справа от разделения
        min_right_x = float('inf') if partition_x == m else nums1[partition_x]
        min_right_y = float('inf') if partition_y == n else nums2[partition_y]
        
        if max_left_x <= min_right_y and max_left_y <= min_right_x:
            # Найдено правильное разделение
            if (m + n) % 2 == 0:
                return (max(max_left_x, max_left_y) + min(min_right_x, min_right_y)) / 2
            else:
                return max(max_left_x, max_left_y)
        elif max_left_x > min_right_y:
            right = partition_x - 1
        else:
            left = partition_x + 1
    
    return -1

# ==================== 2. TRAPPING RAIN WATER ====================
# Компании: Google, Amazon, Facebook
# Частота: Высокая
# Паттерн: Two Pointers

def trap_rain_water(height):
    """
    Сбор дождевой воды
    
    Временная сложность: O(n)
    Пространственная сложность: O(1)
    """
    if not height:
        return 0
    
    left = 0
    right = len(height) - 1
    left_max = right_max = 0
    water_trapped = 0
    
    while left < right:
        if height[left] < height[right]:
            if height[left] >= left_max:
                left_max = height[left]
            else:
                water_trapped += left_max - height[left]
            left += 1
        else:
            if height[right] >= right_max:
                right_max = height[right]
            else:
                water_trapped += right_max - height[right]
            right -= 1
    
    return water_trapped
```

---

## 📈 План подготовки по неделям

### Недели 1-4: Основы (Easy задачи)
```
📊 Неделя 1: Массивы и строки
- Two Sum, Valid Parentheses, Merge Sorted Array
- Reverse String, First Unique Character
- Цель: 2-3 задачи в день

📊 Неделя 2: Связные списки
- Merge Two Sorted Lists, Remove Duplicates
- Reverse Linked List, Linked List Cycle
- Цель: 2-3 задачи в день

📊 Неделя 3: Стеки и очереди
- Valid Parentheses, Implement Stack using Queues
- Min Stack, Daily Temperatures
- Цель: 2-3 задачи в день

📊 Неделя 4: Деревья основы
- Maximum Depth, Same Tree, Invert Binary Tree
- Path Sum, Binary Tree Paths
- Цель: 2-3 задачи в день
```

### Недели 5-12: Средний уровень (Medium задачи)
```
📊 Неделя 5-6: Sliding Window + Two Pointers
- Longest Substring Without Repeating
- 3Sum, Container With Most Water
- Цель: 2 задачи в день

📊 Неделя 7-8: Графы и деревья
- Binary Tree Level Order Traversal
- Number of Islands, Clone Graph
- Цель: 2 задачи в день

📊 Неделя 9-10: Dynamic Programming
- Coin Change, House Robber
- Longest Common Subsequence
- Цель: 1-2 задачи в день

📊 Неделя 11-12: Backtracking
- Generate Parentheses, Permutations
- Subsets, Combination Sum
- Цель: 1-2 задачи в день
```

### Недели 13-20: Продвинутый уровень (Hard задачи)
```
📊 Неделя 13-16: Сложные алгоритмы
- Median of Two Sorted Arrays
- Trapping Rain Water, Sliding Window Maximum
- Цель: 1 задача в день

📊 Неделя 17-20: Подготовка к интервью
- Mock интервью 2-3 раза в неделю
- Повторение сложных тем
- Цель: Довести до автоматизма
```

---

## 🎯 Специфика по компаниям

### Google
```python
# ==================== ТИПИЧНАЯ ЗАДАЧА GOOGLE ====================
# Поиск в 2D матрице

def search_matrix(matrix, target):
    """
    Поиск в отсортированной 2D матрице
    
    Каждая строка отсортирована слева направо
    Первый элемент каждой строки больше последнего элемента предыдущей
    """
    if not matrix or not matrix[0]:
        return False
    
    rows, cols = len(matrix), len(matrix[0])
    left, right = 0, rows * cols - 1
    
    while left <= right:
        mid = (left + right) // 2
        mid_value = matrix[mid // cols][mid % cols]
        
        if mid_value == target:
            return True
        elif mid_value < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return False
```

### Meta (Facebook)
```python
# ==================== ТИПИЧНАЯ ЗАДАЧА META ====================
# Валидация дерева поиска

def is_valid_bst(root):
    """
    Проверка корректности бинарного дерева поиска
    """
    def validate(node, min_val, max_val):
        if not node:
            return True
        
        if node.val <= min_val or node.val >= max_val:
            return False
        
        return (validate(node.left, min_val, node.val) and 
                validate(node.right, node.val, max_val))
    
    return validate(root, float('-inf'), float('inf'))
```

### Amazon
```python
# ==================== ТИПИЧНАЯ ЗАДАЧА AMAZON ====================
# Поиск в повернутом массиве

def search_rotated_array(nums, target):
    """
    Поиск в повернутом отсортированном массиве
    
    Временная сложность: O(log n)
    Пространственная сложность: O(1)
    """
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if nums[mid] == target:
            return mid
        
        # Левая половина отсортирована
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        # Правая половина отсортирована
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1
```

---

## 🏅 Mock Interview Checklist

### Подготовка к интервью
- [ ] **Понимание задачи** - задавать уточняющие вопросы
- [ ] **Примеры** - разобрать входные/выходные данные
- [ ] **Подход** - объяснить стратегию решения
- [ ] **Сложность** - оценить время и память
- [ ] **Реализация** - написать чистый код
- [ ] **Тестирование** - проверить на примерах
- [ ] **Оптимизация** - улучшить решение

### Коммуникация
- [ ] Думать вслух
- [ ] Объяснять каждый шаг
- [ ] Задавать вопросы
- [ ] Учитывать обратную связь

### Типичные ошибки
- ❌ Сразу начинать писать код
- ❌ Не учитывать краевые случаи
- ❌ Неправильная оценка сложности
- ❌ Молчание во время решения
- ❌ Паника при застревании

---

## 📚 Дополнительные ресурсы

### Книги
- "Cracking the Coding Interview" - Gayle McDowell
- "Elements of Programming Interviews" - Aziz, Lee, Prakash
- "Programming Interviews Exposed" - Mongan, Kindler

### Онлайн платформы
- **LeetCode** - основная платформа
- **HackerRank** - дополнительная практика
- **CodeSignal** - симуляция интервью
- **Pramp** - mock интервью

### YouTube каналы
- **Tushar Roy** - детальные объяснения
- **Back To Back SWE** - паттерны решений
- **Kevin Naughton Jr.** - live coding

---

## 📊 Отслеживание прогресса

### Метрики успеха
```python
# Трекер прогресса
progress_tracker = {
    'easy_solved': 0,      # Цель: 50+
    'medium_solved': 0,    # Цель: 100+
    'hard_solved': 0,      # Цель: 30+
    'mock_interviews': 0,  # Цель: 20+
    'companies_applied': 0, # Цель: 5+
    'average_solve_time': 0, # Цель: <45 мин для medium
}
```

### Еженедельные цели
- **Easy:** 10-15 задач в неделю
- **Medium:** 7-10 задач в неделю  
- **Hard:** 2-5 задач в неделю
- **Mock:** 1-2 интервью в неделю

---

*Подготовка к FAANG - это марафон, а не спринт. Постоянство важнее скорости* 🏃‍♂️💪 