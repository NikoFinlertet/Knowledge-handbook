# Структуры данных

> **Основные структуры данных для разработчиков**
>
> От массивов до графов с реализациями на Go и Python

## 📋 Обзор структур данных

### Классификация по использованию

```
📊 ЛИНЕЙНЫЕ СТРУКТУРЫ
├── Массивы (Arrays)
├── Связные списки (Linked Lists) 
├── Стеки (Stacks)
└── Очереди (Queues)

🌳 НЕЛИНЕЙНЫЕ СТРУКТУРЫ  
├── Деревья (Trees)
├── Графы (Graphs)
└── Хэш-таблицы (Hash Tables)
```

---

## 🔢 Массивы (Arrays)

### Характеристики
- **Доступ по индексу:** O(1)
- **Поиск:** O(n) 
- **Вставка в конец:** O(1) амортизированно
- **Удаление:** O(n)

### Реализация и применение

```go
// ==================== БАЗОВЫЕ ОПЕРАЦИИ С МАССИВАМИ ====================
package main

import "fmt"

// ==================== СТРУКТУРА ДИНАМИЧЕСКОГО МАССИВА ====================
type DynamicArray struct {
    data     []int  // Внутренний слайс для хранения данных
    size     int    // Текущее количество элементов
    capacity int    // Максимальная вместимость без реаллокации
}

// ==================== КОНСТРУКТОР ====================
func NewDynamicArray() *DynamicArray {
    return &DynamicArray{
        data:     make([]int, 0, 4), // Начальная емкость 4 элемента
        size:     0,
        capacity: 4,
    }
}

// ==================== ПОЛУЧЕНИЕ ЭЛЕМЕНТА ПО ИНДЕКСУ ====================
// Временная сложность: O(1) - константное время
// Пространственная сложность: O(1)
func (da *DynamicArray) Get(index int) (int, error) {
    // Проверка границ массива
    if index < 0 || index >= da.size {
        return 0, fmt.Errorf("index %d out of bounds [0, %d)", index, da.size)
    }
    
    // Прямой доступ к элементу по индексу - O(1)
    return da.data[index], nil
}

// ==================== ДОБАВЛЕНИЕ ЭЛЕМЕНТА В КОНЕЦ ====================
// Временная сложность: O(1) амортизированно, O(n) в худшем случае
// Пространственная сложность: O(1) обычно, O(n) при реаллокации
func (da *DynamicArray) Append(value int) {
    // Проверяем, нужно ли увеличить емкость
    if da.size >= da.capacity {
        da.resize()  // Увеличиваем емкость в 2 раза - O(n)
    }
    
    // Добавляем элемент в конец - O(1)
    da.data[da.size] = value
    da.size++
}

// ==================== ИЗМЕНЕНИЕ РАЗМЕРА МАССИВА ====================
// Временная сложность: O(n) - копирование всех элементов
// Пространственная сложность: O(n) - новый массив
func (da *DynamicArray) resize() {
    // Удваиваем емкость для амортизированной сложности O(1)
    newCapacity := da.capacity * 2
    newData := make([]int, da.size, newCapacity)
    
    // Копируем все существующие элементы - O(n)
    copy(newData, da.data)
    
    // Обновляем внутренние поля
    da.data = newData
    da.capacity = newCapacity
}

// ==================== ВСТАВКА ПО ИНДЕКСУ ====================
// Временная сложность: O(n) - сдвиг элементов
// Пространственная сложность: O(1)
func (da *DynamicArray) Insert(index, value int) error {
    // Проверка границ
    if index < 0 || index > da.size {
        return fmt.Errorf("index %d out of bounds [0, %d]", index, da.size)
    }
    
    // Увеличиваем размер если нужно
    if da.size >= da.capacity {
        da.resize()
    }
    
    // Сдвигаем элементы вправо - O(n) в худшем случае
    for i := da.size; i > index; i-- {
        da.data[i] = da.data[i-1]
    }
    
    // Вставляем новый элемент
    da.data[index] = value
    da.size++
    
    return nil
}

// ==================== УДАЛЕНИЕ ПО ИНДЕКСУ ====================
// Временная сложность: O(n) - сдвиг элементов
// Пространственная сложность: O(1)
func (da *DynamicArray) RemoveAt(index int) (int, error) {
    // Проверка границ
    if index < 0 || index >= da.size {
        return 0, fmt.Errorf("index %d out of bounds [0, %d)", index, da.size)
    }
    
    // Сохраняем удаляемый элемент
    removedValue := da.data[index]
    
    // Сдвигаем элементы влево - O(n) в худшем случае
    for i := index; i < da.size-1; i++ {
        da.data[i] = da.data[i+1]
    }
    
    da.size--
    return removedValue, nil
}

// ==================== ПОИСК ЭЛЕМЕНТА ====================
// Временная сложность: O(n) - линейный поиск
// Пространственная сложность: O(1)
func (da *DynamicArray) Find(value int) int {
    // Проходим по всем элементам
    for i := 0; i < da.size; i++ {
        if da.data[i] == value {
            return i  // Возвращаем индекс найденного элемента
        }
    }
    return -1  // Элемент не найден
}

// ==================== РАЗМЕР И ЕМКОСТЬ ====================
func (da *DynamicArray) Size() int     { return da.size }
func (da *DynamicArray) Capacity() int { return da.capacity }
func (da *DynamicArray) IsEmpty() bool { return da.size == 0 }

// ==================== ПРИМЕР ИСПОЛЬЗОВАНИЯ ====================
func exampleUsage() {
    arr := NewDynamicArray()
    
    // Добавление элементов - O(1) амортизированно
    arr.Append(10)
    arr.Append(20)
    arr.Append(30)
    
    // Доступ по индексу - O(1)
    value, _ := arr.Get(1)
    fmt.Printf("Element at index 1: %d\n", value)
    
    // Вставка - O(n)
    arr.Insert(1, 15)  // [10, 15, 20, 30]
    
    // Удаление - O(n)
    removed, _ := arr.RemoveAt(2)  // [10, 15, 30]
    fmt.Printf("Removed element: %d\n", removed)
    
    // Поиск - O(n)
    index := arr.Find(30)
    fmt.Printf("Index of 30: %d\n", index)
}
```

```python
# ==================== РЕАЛИЗАЦИЯ НА PYTHON ====================

class DynamicArray:
    """
    Динамический массив с автоматическим изменением размера
    Аналог list в Python, но с явным контролем емкости
    """
    
    def __init__(self, initial_capacity=4):
        """
        Инициализация динамического массива
        
        Args:
            initial_capacity (int): Начальная емкость массива
        """
        self._data = [None] * initial_capacity  # Внутренний массив фиксированного размера
        self._size = 0                          # Текущее количество элементов
        self._capacity = initial_capacity       # Максимальная емкость
    
    # ==================== ОСНОВНЫЕ ОПЕРАЦИИ ====================
    
    def __getitem__(self, index):
        """
        Получение элемента по индексу - O(1)
        
        Args:
            index (int): Индекс элемента
            
        Returns:
            Элемент по указанному индексу
            
        Raises:
            IndexError: Если индекс вне границ массива
        """
        if not 0 <= index < self._size:
            raise IndexError(f"Index {index} out of range [0, {self._size})")
        
        return self._data[index]  # Прямой доступ - O(1)
    
    def __setitem__(self, index, value):
        """Установка значения по индексу - O(1)"""
        if not 0 <= index < self._size:
            raise IndexError(f"Index {index} out of range [0, {self._size})")
        
        self._data[index] = value
    
    def append(self, value):
        """
        Добавление элемента в конец массива
        
        Временная сложность: O(1) амортизированно
        """
        # Проверяем, нужно ли увеличить емкость
        if self._size >= self._capacity:
            self._resize()  # O(n) операция, но редкая
        
        # Добавляем элемент в конец - O(1)
        self._data[self._size] = value
        self._size += 1
    
    def _resize(self):
        """
        Увеличение емкости массива в 2 раза
        
        Временная сложность: O(n)
        Пространственная сложность: O(n)
        """
        # Создаем новый массив удвоенного размера
        old_data = self._data
        self._capacity *= 2
        self._data = [None] * self._capacity
        
        # Копируем все элементы в новый массив - O(n)
        for i in range(self._size):
            self._data[i] = old_data[i]
    
    def insert(self, index, value):
        """
        Вставка элемента по индексу
        
        Временная сложность: O(n) - нужно сдвинуть элементы
        """
        if not 0 <= index <= self._size:
            raise IndexError(f"Index {index} out of range [0, {self._size}]")
        
        # Увеличиваем емкость если нужно
        if self._size >= self._capacity:
            self._resize()
        
        # Сдвигаем элементы вправо - O(n) в худшем случае
        for i in range(self._size, index, -1):
            self._data[i] = self._data[i - 1]
        
        # Вставляем новый элемент
        self._data[index] = value
        self._size += 1
    
    def remove_at(self, index):
        """
        Удаление элемента по индексу
        
        Временная сложность: O(n) - нужно сдвинуть элементы
        """
        if not 0 <= index < self._size:
            raise IndexError(f"Index {index} out of range [0, {self._size})")
        
        # Сохраняем удаляемый элемент
        removed_value = self._data[index]
        
        # Сдвигаем элементы влево - O(n) в худшем случае  
        for i in range(index, self._size - 1):
            self._data[i] = self._data[i + 1]
        
        self._size -= 1
        return removed_value
    
    def find(self, value):
        """
        Поиск первого вхождения элемента
        
        Временная сложность: O(n) - линейный поиск
        """
        for i in range(self._size):
            if self._data[i] == value:
                return i
        return -1  # Не найден
    
    # ==================== ИНФОРМАЦИОННЫЕ МЕТОДЫ ====================
    
    def __len__(self):
        """Размер массива"""
        return self._size
    
    def capacity(self):
        """Текущая емкость массива"""
        return self._capacity
    
    def is_empty(self):
        """Проверка на пустоту"""
        return self._size == 0
    
    def __str__(self):
        """Строковое представление массива"""
        elements = [str(self._data[i]) for i in range(self._size)]
        return f"[{', '.join(elements)}]"

# ==================== ПРАКТИЧЕСКОЕ ПРИМЕНЕНИЕ ====================

def array_applications():
    """
    Примеры практического применения массивов
    """
    
    # 1. Кэш с ограниченным размером (LRU Cache)
    class SimpleCache:
        def __init__(self, max_size=100):
            self.data = DynamicArray()
            self.max_size = max_size
        
        def add_item(self, item):
            if len(self.data) >= self.max_size:
                self.data.remove_at(0)  # Удаляем самый старый элемент
            self.data.append(item)      # Добавляем новый элемент
    
    # 2. Буфер для потока данных
    class StreamBuffer:
        def __init__(self, buffer_size=1000):
            self.buffer = DynamicArray() 
            self.buffer_size = buffer_size
        
        def add_data(self, data_chunk):
            self.buffer.append(data_chunk)
            
            # Очищаем старые данные при переполнении
            while len(self.buffer) > self.buffer_size:
                self.buffer.remove_at(0)
        
        def get_recent_data(self, count=10):
            """Получить последние N элементов"""
            start_index = max(0, len(self.buffer) - count)
            recent_data = []
            
            for i in range(start_index, len(self.buffer)):
                recent_data.append(self.buffer[i])
            
            return recent_data
    
    # 3. Список пользователей онлайн
    class OnlineUsers:
        def __init__(self):
            self.users = DynamicArray()
        
        def user_connect(self, user_id):
            # Проверяем, не подключен ли уже пользователь
            if self.users.find(user_id) == -1:
                self.users.append(user_id)
        
        def user_disconnect(self, user_id):
            index = self.users.find(user_id)
            if index != -1:
                self.users.remove_at(index)
        
        def get_online_count(self):
            return len(self.users)
```

---

## 🔗 Связные списки (Linked Lists)

### Характеристики
- **Доступ по индексу:** O(n)
- **Поиск:** O(n)
- **Вставка в начало:** O(1)
- **Удаление из начала:** O(1)

### Реализация односвязного списка

```go
// ==================== УЗЕЛ СПИСКА ====================
type ListNode struct {
    Val  int       // Значение узла
    Next *ListNode // Указатель на следующий узел
}

// ==================== ОДНОСВЯЗНЫЙ СПИСОК ====================
type LinkedList struct {
    head *ListNode // Указатель на первый узел
    size int       // Текущий размер списка
}

// ==================== КОНСТРУКТОР ====================
func NewLinkedList() *LinkedList {
    return &LinkedList{
        head: nil, // Пустой список
        size: 0,
    }
}

// ==================== ДОБАВЛЕНИЕ В НАЧАЛО ====================
// Временная сложность: O(1) - константное время
// Пространственная сложность: O(1)
func (ll *LinkedList) PrependNode(val int) {
    // Создаем новый узел
    newNode := &ListNode{
        Val:  val,
        Next: ll.head, // Новый узел указывает на старый head
    }
    
    // Обновляем head на новый узел
    ll.head = newNode
    ll.size++
}

// ==================== ДОБАВЛЕНИЕ В КОНЕЦ ====================
// Временная сложность: O(n) - нужно дойти до конца
// Пространственная сложность: O(1)
func (ll *LinkedList) AppendNode(val int) {
    newNode := &ListNode{Val: val, Next: nil}
    
    // Если список пустой - новый узел становится head
    if ll.head == nil {
        ll.head = newNode
        ll.size++
        return
    }
    
    // Идем до конца списка - O(n)
    current := ll.head
    for current.Next != nil {
        current = current.Next
    }
    
    // Привязываем новый узел к концу
    current.Next = newNode
    ll.size++
}

// ==================== ПОИСК ЭЛЕМЕНТА ====================
// Временная сложность: O(n) - линейный поиск
// Пространственная сложность: O(1)
func (ll *LinkedList) Find(val int) *ListNode {
    current := ll.head
    
    // Проходим по всем узлам пока не найдем или не дойдем до конца
    for current != nil {
        if current.Val == val {
            return current // Найден!
        }
        current = current.Next
    }
    
    return nil // Не найден
}

// ==================== УДАЛЕНИЕ ПО ЗНАЧЕНИЮ ====================
// Временная сложность: O(n) - поиск + удаление
// Пространственная сложность: O(1)
func (ll *LinkedList) Remove(val int) bool {
    // Случай 1: Список пустой
    if ll.head == nil {
        return false
    }
    
    // Случай 2: Удаляем head
    if ll.head.Val == val {
        ll.head = ll.head.Next // Сдвигаем head на следующий узел
        ll.size--
        return true
    }
    
    // Случай 3: Ищем узел для удаления
    current := ll.head
    for current.Next != nil {
        if current.Next.Val == val {
            // Найден! Исключаем из цепочки
            current.Next = current.Next.Next
            ll.size--
            return true
        }
        current = current.Next
    }
    
    return false // Не найден
}

// ==================== ПОЛУЧЕНИЕ ПО ИНДЕКСУ ====================
// Временная сложность: O(n) - нужно пройти n узлов
// Пространственная сложность: O(1)
func (ll *LinkedList) GetByIndex(index int) (*ListNode, error) {
    // Проверка границ
    if index < 0 || index >= ll.size {
        return nil, fmt.Errorf("index %d out of bounds", index)
    }
    
    // Проходим index узлов
    current := ll.head
    for i := 0; i < index; i++ {
        current = current.Next
    }
    
    return current, nil
}

// ==================== ПРЕОБРАЗОВАНИЕ В СЛАЙС ====================
// Временная сложность: O(n)
// Пространственная сложность: O(n)
func (ll *LinkedList) ToSlice() []int {
    result := make([]int, 0, ll.size)
    current := ll.head
    
    for current != nil {
        result = append(result, current.Val)
        current = current.Next
    }
    
    return result
}

// ==================== ИНФОРМАЦИОННЫЕ МЕТОДЫ ====================
func (ll *LinkedList) Size() int     { return ll.size }
func (ll *LinkedList) IsEmpty() bool { return ll.head == nil }

// ==================== РЕВЕРС СПИСКА ====================
// Временная сложность: O(n)
// Пространственная сложность: O(1)
func (ll *LinkedList) Reverse() {
    var prev *ListNode = nil
    current := ll.head
    
    // Меняем направление всех указателей
    for current != nil {
        nextTemp := current.Next  // Сохраняем следующий узел
        current.Next = prev       // Разворачиваем указатель
        prev = current           // Сдвигаем prev
        current = nextTemp       // Сдвигаем current
    }
    
    ll.head = prev // Обновляем head
}
```

### Двусвязный список

```python
class DoublyLinkedNode:
    """Узел двусвязного списка"""
    def __init__(self, val=0):
        self.val = val
        self.next = None  # Указатель на следующий узел
        self.prev = None  # Указатель на предыдущий узел

class DoublyLinkedList:
    """
    Двусвязный список с указателями на начало и конец
    Позволяет эффективно добавлять/удалять с обоих концов
    """
    
    def __init__(self):
        # Создаем sentinel узлы для упрощения операций
        self.head = DoublyLinkedNode()  # Фиктивный head
        self.tail = DoublyLinkedNode()  # Фиктивный tail
        
        # Связываем sentinel узлы
        self.head.next = self.tail
        self.tail.prev = self.head
        
        self.size = 0
    
    # ==================== ДОБАВЛЕНИЕ В НАЧАЛО ====================
    # Временная сложность: O(1)
    def prepend(self, val):
        """Добавление элемента в начало списка"""
        new_node = DoublyLinkedNode(val)
        
        # Вставляем между head и первым реальным узлом
        new_node.next = self.head.next
        new_node.prev = self.head
        
        # Обновляем связи соседних узлов
        self.head.next.prev = new_node
        self.head.next = new_node
        
        self.size += 1
    
    # ==================== ДОБАВЛЕНИЕ В КОНЕЦ ====================
    # Временная сложность: O(1) - благодаря указателю на tail!
    def append(self, val):
        """Добавление элемента в конец списка"""
        new_node = DoublyLinkedNode(val)
        
        # Вставляем между последним реальным узлом и tail
        new_node.prev = self.tail.prev
        new_node.next = self.tail
        
        # Обновляем связи соседних узлов
        self.tail.prev.next = new_node
        self.tail.prev = new_node
        
        self.size += 1
    
    # ==================== УДАЛЕНИЕ УЗЛА ====================
    # Временная сложность: O(1) - если есть указатель на узел
    def remove_node(self, node):
        """Удаление конкретного узла"""
        if node == self.head or node == self.tail:
            return  # Нельзя удалить sentinel узлы
        
        # Переконфигурируем связи
        node.prev.next = node.next
        node.next.prev = node.prev
        
        self.size -= 1
    
    # ==================== ПОИСК И УДАЛЕНИЕ ПО ЗНАЧЕНИЮ ====================
    # Временная сложность: O(n)
    def remove_value(self, val):
        """Удаление первого узла с заданным значением"""
        current = self.head.next  # Начинаем с первого реального узла
        
        while current != self.tail:
            if current.val == val:
                self.remove_node(current)
                return True
            current = current.next
        
        return False
    
    # ==================== ВЫВОД СПИСКА ====================
    def to_list(self):
        """Преобразование в обычный список Python"""
        result = []
        current = self.head.next
        
        while current != self.tail:
            result.append(current.val)
            current = current.next
        
        return result
    
    def __len__(self):
        return self.size
    
    def is_empty(self):
        return self.size == 0

# ==================== ПРАКТИЧЕСКОЕ ПРИМЕНЕНИЕ ====================

class LRUCache:
    """
    LRU Cache с использованием хэшмапы + двусвязного списка
    Временная сложность всех операций: O(1)
    """
    
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}  # key -> node mapping
        
        # Двусвязный список для отслеживания порядка использования
        self.dll = DoublyLinkedList()
    
    def get(self, key):
        """Получение значения по ключу - O(1)"""
        if key in self.cache:
            node = self.cache[key]
            # Перемещаем в начало (most recently used)
            self.dll.remove_node(node)
            self.dll.prepend(node.val)
            return node.val
        return -1
    
    def put(self, key, value):
        """Добавление пары ключ-значение - O(1)"""
        if key in self.cache:
            # Обновляем существующий ключ
            node = self.cache[key]
            node.val = value
            self.dll.remove_node(node)
            self.dll.prepend(value)
        else:
            # Добавляем новый ключ
            if len(self.dll) >= self.capacity:
                # Удаляем least recently used (последний в списке)
                # ... реализация удаления LRU элемента
                pass
            
            self.dll.prepend(value)
            # Обновляем хэшмапу
            # ... связывание ключа с узлом
```

---

## 📚 Стеки (Stacks)

### Характеристики LIFO (Last In, First Out)
- **Push (добавление):** O(1)
- **Pop (извлечение):** O(1)
- **Peek (просмотр вершины):** O(1)

### Реализация стека

```go
// ==================== СТЕК НА ОСНОВЕ СЛАЙСА ====================
type Stack struct {
    items []int // Внутреннее хранилище
}

func NewStack() *Stack {
    return &Stack{
        items: make([]int, 0),
    }
}

// ==================== PUSH - ДОБАВЛЕНИЕ НА ВЕРШИНУ ====================
// Временная сложность: O(1) амортизированно
func (s *Stack) Push(item int) {
    s.items = append(s.items, item)  // Добавляем в конец слайса
}

// ==================== POP - ИЗВЛЕЧЕНИЕ С ВЕРШИНЫ ====================
// Временная сложность: O(1)
func (s *Stack) Pop() (int, error) {
    if len(s.items) == 0 {
        return 0, fmt.Errorf("stack is empty")
    }
    
    // Извлекаем последний элемент
    index := len(s.items) - 1
    item := s.items[index]
    s.items = s.items[:index]  // Обрезаем слайс
    
    return item, nil
}

// ==================== PEEK - ПРОСМОТР ВЕРШИНЫ ====================
// Временная сложность: O(1)
func (s *Stack) Peek() (int, error) {
    if len(s.items) == 0 {
        return 0, fmt.Errorf("stack is empty")
    }
    
    return s.items[len(s.items)-1], nil  // Возвращаем без удаления
}

func (s *Stack) IsEmpty() bool { return len(s.items) == 0 }
func (s *Stack) Size() int     { return len(s.items) }

// ==================== ПРАКТИЧЕСКИЕ ПРИМЕНЕНИЯ СТЕКА ====================

// 1. ПРОВЕРКА СБАЛАНСИРОВАННОСТИ СКОБОК
func isValidParentheses(s string) bool {
    stack := NewStack()
    mapping := map[rune]rune{
        ')': '(',
        '}': '{',
        ']': '[',
    }
    
    for _, char := range s {
        if char == '(' || char == '{' || char == '[' {
            stack.Push(int(char))  // Открывающая скобка
        } else if char == ')' || char == '}' || char == ']' {
            if stack.IsEmpty() {
                return false  // Нет соответствующей открывающей скобки
            }
            
            top, _ := stack.Pop()
            if rune(top) != mapping[char] {
                return false  // Неправильная пара скобок
            }
        }
    }
    
    return stack.IsEmpty()  // Все скобки должны быть сбалансированы
}

// 2. ВЫЧИСЛЕНИЕ ВЫРАЖЕНИЙ В ОБРАТНОЙ ПОЛЬСКОЙ ЗАПИСИ
func evalRPN(tokens []string) int {
    stack := NewStack()
    
    for _, token := range tokens {
        switch token {
        case "+":
            b, _ := stack.Pop()
            a, _ := stack.Pop()
            stack.Push(a + b)
        case "-":
            b, _ := stack.Pop()
            a, _ := stack.Pop()
            stack.Push(a - b)
        case "*":
            b, _ := stack.Pop()
            a, _ := stack.Pop()
            stack.Push(a * b)
        case "/":
            b, _ := stack.Pop()
            a, _ := stack.Pop()
            stack.Push(a / b)
        default:
            // Число - преобразуем и кладем в стек
            num, _ := strconv.Atoi(token)
            stack.Push(num)
        }
    }
    
    result, _ := stack.Pop()
    return result
}

// 3. ОБХОД ДЕРЕВА В ГЛУБИНУ БЕЗ РЕКУРСИИ
func dfsIterative(root *TreeNode) []int {
    if root == nil {
        return []int{}
    }
    
    var result []int
    stack := []*TreeNode{root}  // Стек узлов для обхода
    
    for len(stack) > 0 {
        // Извлекаем узел из стека
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        
        result = append(result, node.Val)
        
        // Добавляем детей в стек (правого первым для left-first обхода)
        if node.Right != nil {
            stack = append(stack, node.Right)
        }
        if node.Left != nil {
            stack = append(stack, node.Left)
        }
    }
    
    return result
}

// 4. ФУНКЦИЯ ОТМЕНЫ (UNDO) В РЕДАКТОРЕ
type Editor struct {
    content string
    history *Stack  // Стек предыдущих состояний
}

func NewEditor() *Editor {
    return &Editor{
        content: "",
        history: NewStack(),
    }
}

func (e *Editor) Type(text string) {
    // Сохраняем текущее состояние перед изменением
    e.history.Push(len(e.content))  // Сохраняем позицию для undo
    e.content += text
}

func (e *Editor) Undo() {
    if !e.history.IsEmpty() {
        position, _ := e.history.Pop()
        e.content = e.content[:position]  // Обрезаем до предыдущего состояния
    }
}
```

---

## 🚌 Очереди (Queues)

### Характеристики FIFO (First In, First Out)
- **Enqueue (добавление):** O(1)
- **Dequeue (извлечение):** O(1)
- **Front (просмотр первого):** O(1)

### Реализация очереди

```python
from collections import deque

class Queue:
    """
    Очередь на основе deque для эффективных операций с обоих концов
    """
    
    def __init__(self):
        self._items = deque()  # Двусторонняя очередь для O(1) операций
    
    # ==================== ENQUEUE - ДОБАВЛЕНИЕ В КОНЕЦ ====================
    # Временная сложность: O(1)
    def enqueue(self, item):
        """Добавление элемента в конец очереди"""
        self._items.append(item)  # Добавляем в правый конец
    
    # ==================== DEQUEUE - ИЗВЛЕЧЕНИЕ ИЗ НАЧАЛА ====================
    # Временная сложность: O(1)
    def dequeue(self):
        """Извлечение элемента из начала очереди"""
        if self.is_empty():
            raise IndexError("dequeue from empty queue")
        
        return self._items.popleft()  # Извлекаем из левого конца
    
    # ==================== FRONT - ПРОСМОТР ПЕРВОГО ЭЛЕМЕНТА ====================
    # Временная сложность: O(1)
    def front(self):
        """Просмотр первого элемента без извлечения"""
        if self.is_empty():
            raise IndexError("front from empty queue")
        
        return self._items[0]  # Первый элемент
    
    def is_empty(self):
        """Проверка на пустоту"""
        return len(self._items) == 0
    
    def size(self):
        """Размер очереди"""
        return len(self._items)
    
    def __str__(self):
        """Строковое представление очереди"""
        return f"Queue({list(self._items)})"

# ==================== ПРАКТИЧЕСКИЕ ПРИМЕНЕНИЯ ОЧЕРЕДИ ====================

# 1. ОБХОД ДЕРЕВА В ШИРИНУ (BFS)
def bfs_tree_traversal(root):
    """
    Обход бинарного дерева в ширину
    Временная сложность: O(n), где n - количество узлов
    """
    if not root:
        return []
    
    result = []
    queue = Queue()
    queue.enqueue(root)
    
    while not queue.is_empty():
        # Извлекаем узел из начала очереди
        node = queue.dequeue()
        result.append(node.val)
        
        # Добавляем детей в конец очереди
        if node.left:
            queue.enqueue(node.left)
        if node.right:
            queue.enqueue(node.right)
    
    return result

# 2. СИСТЕМА ОБРАБОТКИ ЗАДАЧ
class TaskProcessor:
    """
    Система обработки задач в порядке поступления
    """
    
    def __init__(self):
        self.task_queue = Queue()
        self.processing = False
    
    def add_task(self, task):
        """Добавление задачи в очередь"""
        print(f"Adding task: {task}")
        self.task_queue.enqueue(task)
    
    def process_next_task(self):
        """Обработка следующей задачи из очереди"""
        if self.task_queue.is_empty():
            print("No tasks to process")
            return None
        
        task = self.task_queue.dequeue()
        print(f"Processing task: {task}")
        
        # Имитация обработки задачи
        # В реальности здесь была бы полезная работа
        return task
    
    def process_all_tasks(self):
        """Обработка всех задач в очереди"""
        while not self.task_queue.is_empty():
            self.process_next_task()

# 3. БУФЕР ДЛЯ ПОТОКОВЫХ ДАННЫХ
class StreamBuffer:
    """
    Буфер для обработки потоковых данных с ограниченным размером
    """
    
    def __init__(self, max_size=1000):
        self.buffer = Queue()
        self.max_size = max_size
    
    def add_data(self, data):
        """Добавление данных в буфер"""
        # Если буфер переполнен, удаляем старые данные
        if self.buffer.size() >= self.max_size:
            self.buffer.dequeue()  # Удаляем самые старые данные
        
        self.buffer.enqueue(data)  # Добавляем новые данные
    
    def get_batch(self, batch_size=10):
        """Получение пакета данных для обработки"""
        batch = []
        
        for _ in range(min(batch_size, self.buffer.size())):
            if not self.buffer.is_empty():
                batch.append(self.buffer.dequeue())
        
        return batch

# 4. ПЛАНИРОВЩИК ROUND-ROBIN
class RoundRobinScheduler:
    """
    Планировщик задач по алгоритму Round Robin
    """
    
    def __init__(self, time_quantum=2):
        self.ready_queue = Queue()
        self.time_quantum = time_quantum
    
    def add_process(self, process_id, burst_time):
        """Добавление процесса в очередь готовности"""
        process = {'id': process_id, 'remaining_time': burst_time}
        self.ready_queue.enqueue(process)
    
    def schedule(self):
        """Выполнение планирования Round Robin"""
        while not self.ready_queue.is_empty():
            # Извлекаем процесс из начала очереди
            current_process = self.ready_queue.dequeue()
            
            # Выполняем процесс в течение кванта времени
            execution_time = min(self.time_quantum, current_process['remaining_time'])
            current_process['remaining_time'] -= execution_time
            
            print(f"Process {current_process['id']} executed for {execution_time} units")
            
            # Если процесс не завершен, возвращаем в конец очереди
            if current_process['remaining_time'] > 0:
                self.ready_queue.enqueue(current_process)
            else:
                print(f"Process {current_process['id']} completed")

# ==================== ПРИОРИТЕТНАЯ ОЧЕРЕДЬ ====================
import heapq

class PriorityQueue:
    """
    Приоритетная очередь на основе кучи
    Элементы извлекаются в порядке приоритета, а не поступления
    """
    
    def __init__(self):
        self._items = []
        self._index = 0  # Для разрешения конфликтов при одинаковом приоритете
    
    def enqueue(self, item, priority):
        """
        Добавление элемента с приоритетом
        Меньшее число = более высокий приоритет
        """
        # Используем отрицательный приоритет для max-heap поведения
        heapq.heappush(self._items, (priority, self._index, item))
        self._index += 1
    
    def dequeue(self):
        """Извлечение элемента с наивысшим приоритетом"""
        if self.is_empty():
            raise IndexError("dequeue from empty priority queue")
        
        priority, index, item = heapq.heappop(self._items)
        return item
    
    def is_empty(self):
        return len(self._items) == 0

# Пример использования приоритетной очереди
def hospital_emergency_system():
    """Система сортировки пациентов в больнице по приоритету"""
    emergency_queue = PriorityQueue()
    
    # Добавляем пациентов (меньший номер = более высокий приоритет)
    emergency_queue.enqueue("Пациент с сердечным приступом", 1)
    emergency_queue.enqueue("Пациент с переломом", 3)
    emergency_queue.enqueue("Пациент с простудой", 5)
    emergency_queue.enqueue("Пациент с инсультом", 1)
    
    # Обрабатываем пациентов в порядке приоритета
    while not emergency_queue.is_empty():
        patient = emergency_queue.dequeue()
        print(f"Обрабатываем: {patient}")
```

---

## 📊 Сравнение структур данных

| Операция | Array | Linked List | Stack | Queue |
|----------|-------|-------------|-------|-------|
| Доступ по индексу | O(1) | O(n) | O(n) | O(n) |
| Поиск | O(n) | O(n) | O(n) | O(n) |
| Вставка в начало | O(n) | O(1) | N/A | N/A |
| Вставка в конец | O(1)* | O(n)** | O(1) | O(1) |
| Удаление из начала | O(n) | O(1) | N/A | O(1) |
| Удаление из конца | O(1) | O(n)** | O(1) | N/A |
| Память | O(n) | O(n) | O(n) | O(n) |

*\* Амортизированно*  
*\*\* O(1) для двусвязного списка с указателем на tail*

---

## 🎯 Выбор структуры данных

### Когда использовать массивы
- ✅ Нужен быстрый доступ по индексу
- ✅ Часто читаем данные
- ✅ Размер данных примерно известен
- ❌ Частые вставки/удаления в середине

### Когда использовать связные списки  
- ✅ Частые вставки/удаления в начале
- ✅ Размер данных сильно меняется
- ✅ Неизвестен максимальный размер
- ❌ Нужен быстрый доступ по индексу

### Когда использовать стеки
- ✅ Обратная польская запись
- ✅ Проверка сбалансированности скобок
- ✅ DFS обход
- ✅ Функция Undo/Redo

### Когда использовать очереди
- ✅ BFS обход
- ✅ Обработка задач по FIFO
- ✅ Планирование процессов
- ✅ Буферизация данных

---

*Выбор правильной структуры данных — это 50% успеха алгоритма* 🎯 