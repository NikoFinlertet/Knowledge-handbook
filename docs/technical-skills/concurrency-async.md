# Конкурентность и асинхронность

**Конкурентность** — это способность системы обрабатывать несколько задач одновременно, а **параллелизм** — это фактическое выполнение задач в одно и то же время на разных процессорах или ядрах.

## Go: Goroutines и Channels

### Goroutines

**Goroutine** — это легковесный поток выполнения в Go, управляемый runtime Go.

**Характеристики:**
- Очень малое потребление памяти (~2KB стартовый размер стека)
- Управляются Go runtime, а не операционной системой
- Могут выполняться на одном или нескольких OS-потоках (M:N модель)

**Основы использования:**
```go
// services/user-service/examples/goroutines.go
// Пример использования goroutines для параллельной обработки задач

package main

import (
    "fmt"  // Для вывода в консоль
    "time" // Для работы с временем
)

// ==================== WORKER FUNCTION ====================
// Функция-воркер, которая будет выполняться в отдельной goroutine
func worker(id int, done chan bool) {
    // Логирование начала работы воркера
    fmt.Printf("Worker %d starting\n", id)
    
    // ==================== ИМИТАЦИЯ РАБОТЫ ====================
    // Имитация полезной работы (например, обработка данных, сетевой запрос)
    time.Sleep(time.Second)  // Блокирует выполнение на 1 секунду
    
    // Логирование завершения работы
    fmt.Printf("Worker %d done\n", id)
    
    // ==================== СИГНАЛИЗАЦИЯ О ЗАВЕРШЕНИИ ====================
    // Отправляем сигнал в канал о том, что работа завершена
    done <- true  // Не блокирует, так как канал буферизован
}

// ==================== MAIN FUNCTION ====================
func main() {
    // ==================== СОЗДАНИЕ КАНАЛА ДЛЯ СИНХРОНИЗАЦИИ ====================
    // Создаем буферизованный канал для синхронизации с goroutines
    // Размер буфера = 3, что позволяет всем воркерам отправить сигнал без блокировки
    done := make(chan bool, 3)
    
    // ==================== ЗАПУСК GOROUTINES ====================
    // Запускаем 3 goroutines параллельно
    for i := 1; i <= 3; i++ {
        // Ключевое слово 'go' запускает функцию в отдельной goroutine
        go worker(i, done)  // Каждый воркер получает свой уникальный ID
    }
    
    // ==================== ОЖИДАНИЕ ЗАВЕРШЕНИЯ ВСЕХ GOROUTINES ====================
    // Ждем сигналов от всех воркеров о завершении работы
    for i := 1; i <= 3; i++ {
        <-done  // Блокируется до получения сигнала из канала
    }
    
    // ==================== ЗАВЕРШЕНИЕ ПРОГРАММЫ ====================
    fmt.Println("All workers completed")
}

// ==================== ОБЪЯСНЕНИЕ РАБОТЫ ====================
// 1. Создается буферизованный канал done с размером 3
// 2. Запускаются 3 goroutines, каждая выполняет функцию worker
// 3. Каждая goroutine работает независимо и параллельно
// 4. При завершении работы каждая goroutine отправляет true в канал done
// 5. Главная goroutine ждет 3 сигнала из канала done
// 6. После получения всех сигналов программа завершается

// ==================== ПРЕИМУЩЕСТВА ====================
// - Параллельное выполнение: все воркеры работают одновременно
// - Низкое потребление памяти: goroutines очень легковесные
// - Безопасная синхронизация: используются каналы вместо мьютексов
// - Простота: код читается и понимается легко
```

### Channels

**Channel** — это типизированный канал для безопасной передачи данных между goroutines.

**Типы каналов:**
```go
// Небуферизованный канал (синхронный)
ch := make(chan int)

// Буферизованный канал (асинхронный)
buffered := make(chan int, 10)

// Канал только для чтения
readOnly := make(<-chan int)

// Канал только для записи
writeOnly := make(chan<- int)
```

**Применение в проекте:**
```go
// services/user-service/internal/infrastructure/worker_pool.go
type WorkerPool struct {
    workers    int
    jobQueue   chan Job
    resultChan chan Result
    quit       chan bool
}

type Job struct {
    ID   int
    Data interface{}
}

type Result struct {
    JobID int
    Data  interface{}
    Error error
}

func NewWorkerPool(workers int, jobQueueSize int) *WorkerPool {
    return &WorkerPool{
        workers:    workers,
        jobQueue:   make(chan Job, jobQueueSize),
        resultChan: make(chan Result, jobQueueSize),
        quit:       make(chan bool),
    }
}

func (wp *WorkerPool) Start() {
    for i := 0; i < wp.workers; i++ {
        go wp.worker(i)
    }
}

func (wp *WorkerPool) worker(id int) {
    for {
        select {
        case job := <-wp.jobQueue:
            result := wp.processJob(job)
            wp.resultChan <- result
        case <-wp.quit:
            fmt.Printf("Worker %d stopping\n", id)
            return
        }
    }
}

func (wp *WorkerPool) processJob(job Job) Result {
    // Обработка задачи
    time.Sleep(100 * time.Millisecond) // Имитация работы
    
    return Result{
        JobID: job.ID,
        Data:  fmt.Sprintf("Processed job %d", job.ID),
        Error: nil,
    }
}

func (wp *WorkerPool) AddJob(job Job) {
    wp.jobQueue <- job
}

func (wp *WorkerPool) GetResult() Result {
    return <-wp.resultChan
}

func (wp *WorkerPool) Stop() {
    close(wp.quit)
}
```

### Паттерны синхронизации

#### 1. WaitGroup
```go
// services/user-service/examples/waitgroup.go
import (
    "fmt"
    "sync"
    "time"
)

func processUsers(users []string) {
    var wg sync.WaitGroup
    
    for _, user := range users {
        wg.Add(1) // Увеличиваем счетчик
        
        go func(userID string) {
            defer wg.Done() // Уменьшаем счетчик при завершении
            
            // Обработка пользователя
            fmt.Printf("Processing user: %s\n", userID)
            time.Sleep(100 * time.Millisecond)
        }(user)
    }
    
    wg.Wait() // Ждем завершения всех goroutines
    fmt.Println("All users processed")
}
```

#### 2. Mutex для защиты разделяемых ресурсов
```go
// services/user-service/internal/infrastructure/counter.go
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

#### 3. Select для мультиплексирования каналов
```go
// services/user-service/examples/select.go
func fanIn(input1, input2 <-chan string) <-chan string {
    output := make(chan string)
    
    go func() {
        defer close(output)
        for {
            select {
            case msg, ok := <-input1:
                if !ok {
                    input1 = nil
                } else {
                    output <- msg
                }
            case msg, ok := <-input2:
                if !ok {
                    input2 = nil
                } else {
                    output <- msg
                }
            }
            
            if input1 == nil && input2 == nil {
                break
            }
        }
    }()
    
    return output
}
```

### Context для управления lifecycle
```go
// services/user-service/internal/application/service.go
import (
    "context"
    "time"
)

func (s *UserService) ProcessUsersWithTimeout(ctx context.Context, users []User) error {
    // Создаем контекст с таймаутом
    timeoutCtx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    
    resultChan := make(chan error, 1)
    
    go func() {
        // Длительная операция
        err := s.processUsersInternal(users)
        resultChan <- err
    }()
    
    select {
    case err := <-resultChan:
        return err
    case <-timeoutCtx.Done():
        return timeoutCtx.Err() // Возвращаем ошибку таймаута
    }
}

func (s *UserService) processUsersInternal(users []User) error {
    for _, user := range users {
        // Проверяем, не отменена ли операция
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            // Продолжаем обработку
        }
        
        if err := s.processUser(user); err != nil {
            return err
        }
    }
    return nil
}
```

## Python: Async/Await и Multiprocessing

### Async/Await

**Async/Await** — это синтаксис для асинхронного программирования в Python, основанный на корутинах.

**Основы:**
```python
# services/order-service/examples/async_basics.py
import asyncio
import aiohttp
import time

async def fetch_user(session, user_id):
    """Асинхронная функция для получения пользователя"""
    url = f"http://user-service:8081/api/v1/users/{user_id}"
    
    async with session.get(url) as response:
        if response.status == 200:
            return await response.json()
        else:
            raise Exception(f"Failed to fetch user {user_id}")

async def fetch_multiple_users(user_ids):
    """Получение нескольких пользователей параллельно"""
    async with aiohttp.ClientSession() as session:
        # Создаем задачи для параллельного выполнения
        tasks = [fetch_user(session, user_id) for user_id in user_ids]
        
        # Ждем завершения всех задач
        users = await asyncio.gather(*tasks, return_exceptions=True)
        
        return users

# Использование
async def main():
    user_ids = ["1", "2", "3", "4", "5"]
    
    start_time = time.time()
    users = await fetch_multiple_users(user_ids)
    end_time = time.time()
    
    print(f"Fetched {len(users)} users in {end_time - start_time:.2f} seconds")
    
    for i, user in enumerate(users):
        if isinstance(user, Exception):
            print(f"Error fetching user {user_ids[i]}: {user}")
        else:
            print(f"User: {user.get('name', 'Unknown')}")

# Запуск
if __name__ == "__main__":
    asyncio.run(main())
```

### Применение в проекте
```python
# services/order-service/async_service.py
import asyncio
import aiohttp
from typing import List, Dict, Any
from dataclasses import dataclass

@dataclass
class AsyncOrderService:
    def __init__(self, user_service_url: str):
        self.user_service_url = user_service_url
        self.session = None
    
    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            await self.session.close()
    
    async def get_user(self, user_id: str) -> Dict[str, Any]:
        """Получить пользователя асинхронно"""
        url = f"{self.user_service_url}/api/v1/users/{user_id}"
        
        async with self.session.get(url) as response:
            if response.status == 200:
                return await response.json()
            else:
                raise Exception(f"User {user_id} not found")
    
    async def validate_order_items(self, items: List[Dict]) -> List[Dict]:
        """Валидация товаров в заказе"""
        tasks = []
        
        for item in items:
            task = self.validate_product(item['product_id'])
            tasks.append(task)
        
        # Параллельная валидация всех товаров
        validation_results = await asyncio.gather(*tasks, return_exceptions=True)
        
        validated_items = []
        for i, result in enumerate(validation_results):
            if isinstance(result, Exception):
                raise Exception(f"Invalid product {items[i]['product_id']}: {result}")
            
            validated_items.append({
                **items[i],
                'product_info': result
            })
        
        return validated_items
    
    async def validate_product(self, product_id: str) -> Dict[str, Any]:
        """Валидация конкретного товара"""
        # Имитация обращения к внешнему сервису
        await asyncio.sleep(0.1)  # Имитация сетевой задержки
        
        # Моковые данные для примера
        return {
            'id': product_id,
            'name': f'Product {product_id}',
            'price': 100.0,
            'available': True
        }
    
    async def create_order_async(self, user_id: str, items: List[Dict]) -> Dict[str, Any]:
        """Создание заказа с асинхронной валидацией"""
        # Параллельно получаем пользователя и валидируем товары
        user_task = self.get_user(user_id)
        items_task = self.validate_order_items(items)
        
        try:
            user, validated_items = await asyncio.gather(user_task, items_task)
        except Exception as e:
            raise Exception(f"Order validation failed: {e}")
        
        # Создаем заказ
        order = {
            'id': f"order_{int(time.time())}",
            'user_id': user_id,
            'user_name': user['name'],
            'items': validated_items,
            'total_amount': sum(item['price'] * item['quantity'] for item in validated_items),
            'status': 'pending'
        }
        
        return order
```

### Async Context Managers
```python
# services/order-service/infrastructure/async_database.py
import asyncpg
from typing import Optional, List, Dict, Any

class AsyncDatabasePool:
    def __init__(self, connection_string: str, min_size: int = 5, max_size: int = 20):
        self.connection_string = connection_string
        self.min_size = min_size
        self.max_size = max_size
        self.pool: Optional[asyncpg.Pool] = None
    
    async def __aenter__(self):
        self.pool = await asyncpg.create_pool(
            self.connection_string,
            min_size=self.min_size,
            max_size=self.max_size
        )
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.pool:
            await self.pool.close()
    
    async def execute_query(self, query: str, *args) -> List[Dict[str, Any]]:
        """Выполнить SQL запрос"""
        async with self.pool.acquire() as connection:
            rows = await connection.fetch(query, *args)
            return [dict(row) for row in rows]
    
    async def execute_transaction(self, queries: List[tuple]) -> bool:
        """Выполнить транзакцию с несколькими запросами"""
        async with self.pool.acquire() as connection:
            async with connection.transaction():
                try:
                    for query, args in queries:
                        await connection.execute(query, *args)
                    return True
                except Exception as e:
                    # Транзакция автоматически откатится
                    raise e
```

### Multiprocessing

**Multiprocessing** — это модуль для создания процессов для CPU-интенсивных задач.

```python
# services/order-service/examples/multiprocessing_example.py
import multiprocessing as mp
import time
from typing import List

def cpu_intensive_task(data_chunk: List[int]) -> List[int]:
    """CPU-интенсивная задача"""
    result = []
    for num in data_chunk:
        # Имитация сложных вычислений
        factorial = 1
        for i in range(1, num + 1):
            factorial *= i
        result.append(factorial % 1000000)  # Берем остаток для управляемости размера
    return result

def process_data_parallel(data: List[int], num_processes: int = None) -> List[int]:
    """Обработка данных с использованием multiprocessing"""
    if num_processes is None:
        num_processes = mp.cpu_count()
    
    # Разделяем данные на чанки
    chunk_size = len(data) // num_processes
    chunks = [data[i:i + chunk_size] for i in range(0, len(data), chunk_size)]
    
    # Создаем пул процессов
    with mp.Pool(processes=num_processes) as pool:
        results = pool.map(cpu_intensive_task, chunks)
    
    # Объединяем результаты
    final_result = []
    for result_chunk in results:
        final_result.extend(result_chunk)
    
    return final_result

def compare_performance():
    """Сравнение производительности последовательной и параллельной обработки"""
    data = list(range(1, 1001))  # Числа от 1 до 1000
    
    # Последовательная обработка
    start_time = time.time()
    sequential_result = cpu_intensive_task(data)
    sequential_time = time.time() - start_time
    
    # Параллельная обработка
    start_time = time.time()
    parallel_result = process_data_parallel(data)
    parallel_time = time.time() - start_time
    
    print(f"Sequential processing: {sequential_time:.2f} seconds")
    print(f"Parallel processing: {parallel_time:.2f} seconds")
    print(f"Speedup: {sequential_time / parallel_time:.2f}x")
    
    # Проверяем, что результаты одинаковые
    assert sequential_result == parallel_result
    print("Results are identical ✓")

if __name__ == "__main__":
    compare_performance()
```

### Комбинирование async и multiprocessing
```python
# services/order-service/hybrid_processing.py
import asyncio
import multiprocessing as mp
from concurrent.futures import ProcessPoolExecutor
from typing import List, Dict, Any

class HybridOrderProcessor:
    def __init__(self, max_workers: int = None):
        self.max_workers = max_workers or mp.cpu_count()
        self.executor = ProcessPoolExecutor(max_workers=self.max_workers)
    
    async def process_orders_hybrid(self, orders: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """Гибридная обработка: I/O операции через async, CPU-интенсивные через multiprocessing"""
        
        # 1. Асинхронно получаем дополнительные данные
        enriched_orders = await self.enrich_orders_async(orders)
        
        # 2. CPU-интенсивные вычисления через multiprocessing
        loop = asyncio.get_event_loop()
        processed_orders = await loop.run_in_executor(
            self.executor,
            self.calculate_complex_metrics,
            enriched_orders
        )
        
        # 3. Асинхронно сохраняем результаты
        await self.save_orders_async(processed_orders)
        
        return processed_orders
    
    async def enrich_orders_async(self, orders: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """Асинхронное обогащение данных заказов"""
        tasks = [self.enrich_single_order(order) for order in orders]
        return await asyncio.gather(*tasks)
    
    async def enrich_single_order(self, order: Dict[str, Any]) -> Dict[str, Any]:
        """Обогащение одного заказа"""
        # Имитация асинхронных I/O операций
        await asyncio.sleep(0.1)
        
        order['enriched_data'] = {
            'customer_segment': 'premium',
            'shipping_cost': 10.0,
            'tax_rate': 0.1
        }
        return order
    
    def calculate_complex_metrics(self, orders: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """CPU-интенсивные вычисления (выполняется в отдельном процессе)"""
        for order in orders:
            # Сложные вычисления
            total_amount = order.get('total_amount', 0)
            tax_rate = order['enriched_data']['tax_rate']
            shipping_cost = order['enriched_data']['shipping_cost']
            
            # Имитация сложных расчетов
            final_amount = total_amount * (1 + tax_rate) + shipping_cost
            discount = self.calculate_complex_discount(total_amount)
            
            order['final_amount'] = final_amount - discount
            order['discount'] = discount
        
        return orders
    
    def calculate_complex_discount(self, amount: float) -> float:
        """Сложный расчет скидки"""
        # Имитация CPU-интенсивных вычислений
        discount = 0
        for i in range(10000):
            discount += (amount * 0.001) / (i + 1)
        return min(discount, amount * 0.2)  # Максимум 20% скидки
    
    async def save_orders_async(self, orders: List[Dict[str, Any]]):
        """Асинхронное сохранение заказов"""
        tasks = [self.save_single_order(order) for order in orders]
        await asyncio.gather(*tasks)
    
    async def save_single_order(self, order: Dict[str, Any]):
        """Сохранение одного заказа"""
        # Имитация асинхронного сохранения в БД
        await asyncio.sleep(0.05)
        print(f"Saved order {order.get('id')} with final amount ${order.get('final_amount', 0):.2f}")
    
    def close(self):
        """Закрытие executor"""
        self.executor.shutdown(wait=True)
```

## Лучшие практики

### Go Goroutines
1. **Не создавайте слишком много goroutines** - используйте worker pools
2. **Всегда обрабатывайте завершение** - используйте context.Context
3. **Избегайте гонок данных** - используйте channels или mutex
4. **Закрывайте channels** - отправитель должен закрывать канал
5. **Используйте select с default** - для неблокирующих операций

### Python Async/Await
1. **Используйте async/await для I/O операций** - не для CPU-интенсивных задач
2. **Управляйте сессиями** - используйте context managers
3. **Группируйте операции** - используйте asyncio.gather()
4. **Обрабатывайте исключения** - используйте return_exceptions=True
5. **Пул соединений** - для базы данных используйте connection pooling

### Выбор подхода

**Используйте Goroutines когда:**
- Нужна простота и производительность
- Много конкурентных задач
- Системное программирование
- Сетевые серверы

**Используйте Async/Await когда:**
- Много I/O операций
- Веб-разработка
- API клиенты
- Интеграции с внешними сервисами

**Используйте Multiprocessing когда:**
- CPU-интенсивные задачи
- Параллельные вычисления
- Обработка больших данных
- Научные вычисления

## Заключение

Правильный выбор модели конкурентности зависит от характера задач:

- **Go goroutines** - лучший выбор для конкурентных сетевых приложений
- **Python async/await** - идеально для I/O-интенсивных веб-приложений  
- **Python multiprocessing** - необходим для CPU-интенсивных вычислений

В нашем проекте используется комбинация подходов: Go goroutines в User Service для обработки HTTP запросов, Python async/await в Order Service для работы с внешними API.

## См. также
- [Микросервисная архитектура](./microservices-architecture.md)
- [Базы данных](./databases.md)
- [Тестирование](./testing.md) 