# Алгоритмы и структуры данных для Senior разработчика

Для Senior разработчика важно понимать не только как использовать готовые структуры данных, но и когда и почему выбирать конкретный алгоритм. Этот материал фокусируется на практическом применении в реальных проектах.

## Основные структуры данных

### 1. Массивы и слайсы (Arrays & Slices)

```go
// Динамические массивы в Go
type DynamicArray[T any] struct {
    data []T
    size int
}

func NewDynamicArray[T any]() *DynamicArray[T] {
    return &DynamicArray[T]{
        data: make([]T, 0, 10),
    }
}

func (da *DynamicArray[T]) Append(item T) {
    da.data = append(da.data, item)
    da.size++
}

func (da *DynamicArray[T]) Get(index int) (T, error) {
    if index < 0 || index >= da.size {
        var zero T
        return zero, errors.New("index out of range")
    }
    return da.data[index], nil
}

// Применение в проекте - пагинация
func (r *UserRepository) GetUsersPage(page, pageSize int) ([]User, error) {
    offset := (page - 1) * pageSize
    limit := pageSize
    
    query := `SELECT * FROM users LIMIT $1 OFFSET $2`
    rows, err := r.db.Query(query, limit, offset)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    users := make([]User, 0, pageSize) // Заранее выделяем память
    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Email, &user.Name); err != nil {
            return nil, err
        }
        users = append(users, user)
    }
    
    return users, nil
}
```

### 2. Хэш-таблицы (Hash Tables)

```go
// Кэш с TTL на основе хэш-таблицы
type CacheItem struct {
    value     interface{}
    expiresAt time.Time
}

type TTLCache struct {
    mu    sync.RWMutex
    items map[string]*CacheItem
}

func NewTTLCache() *TTLCache {
    cache := &TTLCache{
        items: make(map[string]*CacheItem),
    }
    
    // Очистка истекших элементов
    go cache.cleanup()
    return cache
}

func (c *TTLCache) Set(key string, value interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.items[key] = &CacheItem{
        value:     value,
        expiresAt: time.Now().Add(ttl),
    }
}

func (c *TTLCache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    item, exists := c.items[key]
    if !exists || time.Now().After(item.expiresAt) {
        return nil, false
    }
    
    return item.value, true
}

// Применение в user service
type UserService struct {
    repo  UserRepository
    cache *TTLCache
}

func (s *UserService) GetUser(id string) (*User, error) {
    // Проверяем кэш
    if cached, found := s.cache.Get(id); found {
        return cached.(*User), nil
    }
    
    // Загружаем из базы
    user, err := s.repo.FindByID(id)
    if err != nil {
        return nil, err
    }
    
    // Кэшируем на 5 минут
    s.cache.Set(id, user, 5*time.Minute)
    return user, nil
}
```

### 3. Деревья (Trees)

```go
// Trie для автодополнения
type TrieNode struct {
    children map[rune]*TrieNode
    isEnd    bool
    value    string
}

type Trie struct {
    root *TrieNode
}

func NewTrie() *Trie {
    return &Trie{
        root: &TrieNode{
            children: make(map[rune]*TrieNode),
        },
    }
}

func (t *Trie) Insert(word string) {
    node := t.root
    for _, char := range word {
        if _, exists := node.children[char]; !exists {
            node.children[char] = &TrieNode{
                children: make(map[rune]*TrieNode),
            }
        }
        node = node.children[char]
    }
    node.isEnd = true
    node.value = word
}

func (t *Trie) Search(prefix string) []string {
    node := t.root
    for _, char := range prefix {
        if _, exists := node.children[char]; !exists {
            return []string{}
        }
        node = node.children[char]
    }
    
    var results []string
    t.dfs(node, &results)
    return results
}

func (t *Trie) dfs(node *TrieNode, results *[]string) {
    if node.isEnd {
        *results = append(*results, node.value)
    }
    
    for _, child := range node.children {
        t.dfs(child, results)
    }
}

// Применение в поиске пользователей
type UserSearchService struct {
    trie *Trie
}

func (s *UserSearchService) BuildIndex(users []User) {
    for _, user := range users {
        s.trie.Insert(strings.ToLower(user.Name))
        s.trie.Insert(strings.ToLower(user.Email))
    }
}

func (s *UserSearchService) Autocomplete(query string) []string {
    return s.trie.Search(strings.ToLower(query))
}
```

### 4. Очереди (Queues)

```go
// Очередь с приоритетами для обработки задач
type PriorityItem struct {
    Value    interface{}
    Priority int
    Index    int
}

type PriorityQueue []*PriorityItem

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].Priority > pq[j].Priority
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].Index = i
    pq[j].Index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*PriorityItem)
    item.Index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    old[n-1] = nil
    item.Index = -1
    *pq = old[0 : n-1]
    return item
}

// Применение в обработке заказов
type OrderProcessor struct {
    queue *PriorityQueue
    mu    sync.Mutex
}

func NewOrderProcessor() *OrderProcessor {
    pq := make(PriorityQueue, 0)
    heap.Init(&pq)
    
    return &OrderProcessor{
        queue: &pq,
    }
}

func (op *OrderProcessor) AddOrder(order *Order) {
    op.mu.Lock()
    defer op.mu.Unlock()
    
    priority := op.calculatePriority(order)
    item := &PriorityItem{
        Value:    order,
        Priority: priority,
    }
    
    heap.Push(op.queue, item)
}

func (op *OrderProcessor) ProcessNext() *Order {
    op.mu.Lock()
    defer op.mu.Unlock()
    
    if op.queue.Len() == 0 {
        return nil
    }
    
    item := heap.Pop(op.queue).(*PriorityItem)
    return item.Value.(*Order)
}

func (op *OrderProcessor) calculatePriority(order *Order) int {
    priority := 0
    
    // VIP клиенты
    if order.IsVIP {
        priority += 100
    }
    
    // Большие заказы
    if order.Total > 1000 {
        priority += 50
    }
    
    // Срочные заказы
    if order.IsUrgent {
        priority += 75
    }
    
    return priority
}
```

## Основные алгоритмы

### 1. Поиск и сортировка

```go
// Бинарный поиск для отсортированных данных
func BinarySearch[T comparable](arr []T, target T, compare func(T, T) int) int {
    left, right := 0, len(arr)-1
    
    for left <= right {
        mid := left + (right-left)/2
        cmp := compare(arr[mid], target)
        
        if cmp == 0 {
            return mid
        } else if cmp < 0 {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    
    return -1
}

// Применение в поиске пользователей по возрасту
func (r *UserRepository) FindUsersByAgeRange(minAge, maxAge int) ([]User, error) {
    // Предполагаем, что пользователи отсортированы по возрасту
    users, err := r.GetAllUsersSortedByAge()
    if err != nil {
        return nil, err
    }
    
    startIdx := BinarySearch(users, User{Age: minAge}, func(a, b User) int {
        return a.Age - b.Age
    })
    
    endIdx := BinarySearch(users, User{Age: maxAge}, func(a, b User) int {
        return a.Age - b.Age
    })
    
    if startIdx == -1 || endIdx == -1 {
        return []User{}, nil
    }
    
    return users[startIdx:endIdx+1], nil
}
```

### 2. Алгоритмы графов

```go
// Граф для моделирования отношений пользователей
type UserGraph struct {
    adjacencyList map[string][]string
    mu            sync.RWMutex
}

func NewUserGraph() *UserGraph {
    return &UserGraph{
        adjacencyList: make(map[string][]string),
    }
}

func (g *UserGraph) AddFriendship(userID1, userID2 string) {
    g.mu.Lock()
    defer g.mu.Unlock()
    
    g.adjacencyList[userID1] = append(g.adjacencyList[userID1], userID2)
    g.adjacencyList[userID2] = append(g.adjacencyList[userID2], userID1)
}

// BFS для поиска кратчайшего пути между пользователями
func (g *UserGraph) FindShortestPath(start, end string) []string {
    g.mu.RLock()
    defer g.mu.RUnlock()
    
    if start == end {
        return []string{start}
    }
    
    visited := make(map[string]bool)
    queue := []string{start}
    parent := make(map[string]string)
    
    visited[start] = true
    
    for len(queue) > 0 {
        current := queue[0]
        queue = queue[1:]
        
        for _, neighbor := range g.adjacencyList[current] {
            if !visited[neighbor] {
                visited[neighbor] = true
                parent[neighbor] = current
                queue = append(queue, neighbor)
                
                if neighbor == end {
                    return g.buildPath(parent, start, end)
                }
            }
        }
    }
    
    return []string{} // Путь не найден
}

func (g *UserGraph) buildPath(parent map[string]string, start, end string) []string {
    path := []string{}
    current := end
    
    for current != start {
        path = append([]string{current}, path...)
        current = parent[current]
    }
    
    path = append([]string{start}, path...)
    return path
}

// Применение в рекомендациях друзей
type FriendRecommendationService struct {
    userGraph *UserGraph
}

func (s *FriendRecommendationService) RecommendFriends(userID string) []string {
    // Найти друзей друзей (расстояние 2)
    friends := s.userGraph.adjacencyList[userID]
    recommendations := make(map[string]int)
    
    for _, friend := range friends {
        friendsOfFriend := s.userGraph.adjacencyList[friend]
        for _, fof := range friendsOfFriend {
            if fof != userID && !contains(friends, fof) {
                recommendations[fof]++
            }
        }
    }
    
    // Сортировка по количеству общих друзей
    var result []string
    for userID, count := range recommendations {
        if count >= 2 { // Минимум 2 общих друга
            result = append(result, userID)
        }
    }
    
    return result
}
```

### 3. Динамическое программирование

```go
// Кэширование с мемоизацией для дорогих вычислений
type MemoizedCalculator struct {
    cache map[string]interface{}
    mu    sync.RWMutex
}

func NewMemoizedCalculator() *MemoizedCalculator {
    return &MemoizedCalculator{
        cache: make(map[string]interface{}),
    }
}

func (c *MemoizedCalculator) Calculate(key string, fn func() interface{}) interface{} {
    c.mu.RLock()
    if cached, exists := c.cache[key]; exists {
        c.mu.RUnlock()
        return cached
    }
    c.mu.RUnlock()
    
    c.mu.Lock()
    defer c.mu.Unlock()
    
    // Двойная проверка
    if cached, exists := c.cache[key]; exists {
        return cached
    }
    
    result := fn()
    c.cache[key] = result
    return result
}

// Применение в вычислении статистики пользователей
type UserStatisticsService struct {
    calculator *MemoizedCalculator
    userRepo   UserRepository
}

func (s *UserStatisticsService) GetUserEngagementScore(userID string) float64 {
    key := fmt.Sprintf("engagement:%s", userID)
    
    result := s.calculator.Calculate(key, func() interface{} {
        return s.calculateEngagementScore(userID)
    })
    
    return result.(float64)
}

func (s *UserStatisticsService) calculateEngagementScore(userID string) float64 {
    // Дорогое вычисление
    user, _ := s.userRepo.FindByID(userID)
    
    score := 0.0
    score += float64(user.LoginCount) * 0.1
    score += float64(user.PostCount) * 0.3
    score += float64(user.CommentCount) * 0.2
    // ... другие метрики
    
    return score
}
```

## Алгоритмы для конкретных задач

### 1. Rate Limiting (Алгоритм скользящего окна)

```go
type SlidingWindowRateLimiter struct {
    windowSize time.Duration
    maxRequests int
    requests   map[string][]time.Time
    mu         sync.RWMutex
}

func NewSlidingWindowRateLimiter(windowSize time.Duration, maxRequests int) *SlidingWindowRateLimiter {
    return &SlidingWindowRateLimiter{
        windowSize:  windowSize,
        maxRequests: maxRequests,
        requests:    make(map[string][]time.Time),
    }
}

func (rl *SlidingWindowRateLimiter) Allow(clientID string) bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()
    
    now := time.Now()
    windowStart := now.Add(-rl.windowSize)
    
    // Получаем существующие запросы
    clientRequests := rl.requests[clientID]
    
    // Удаляем устаревшие запросы
    validRequests := make([]time.Time, 0)
    for _, reqTime := range clientRequests {
        if reqTime.After(windowStart) {
            validRequests = append(validRequests, reqTime)
        }
    }
    
    // Проверяем лимит
    if len(validRequests) >= rl.maxRequests {
        return false
    }
    
    // Добавляем новый запрос
    validRequests = append(validRequests, now)
    rl.requests[clientID] = validRequests
    
    return true
}
```

### 2. Consistent Hashing (для распределения данных)

```go
type ConsistentHash struct {
    ring     map[uint32]string
    sortedHashes []uint32
    virtualNodes int
    mu          sync.RWMutex
}

func NewConsistentHash(virtualNodes int) *ConsistentHash {
    return &ConsistentHash{
        ring:         make(map[uint32]string),
        virtualNodes: virtualNodes,
    }
}

func (ch *ConsistentHash) AddNode(node string) {
    ch.mu.Lock()
    defer ch.mu.Unlock()
    
    for i := 0; i < ch.virtualNodes; i++ {
        virtualKey := fmt.Sprintf("%s:%d", node, i)
        hash := ch.hash(virtualKey)
        ch.ring[hash] = node
        ch.sortedHashes = append(ch.sortedHashes, hash)
    }
    
    sort.Slice(ch.sortedHashes, func(i, j int) bool {
        return ch.sortedHashes[i] < ch.sortedHashes[j]
    })
}

func (ch *ConsistentHash) GetNode(key string) string {
    ch.mu.RLock()
    defer ch.mu.RUnlock()
    
    if len(ch.ring) == 0 {
        return ""
    }
    
    hash := ch.hash(key)
    
    // Найти первый узел с хэшем >= hash
    idx := sort.Search(len(ch.sortedHashes), func(i int) bool {
        return ch.sortedHashes[i] >= hash
    })
    
    if idx == len(ch.sortedHashes) {
        idx = 0
    }
    
    return ch.ring[ch.sortedHashes[idx]]
}

func (ch *ConsistentHash) hash(key string) uint32 {
    h := fnv.New32a()
    h.Write([]byte(key))
    return h.Sum32()
}

// Применение в микросервисах для шардинга
type UserShardingService struct {
    consistentHash *ConsistentHash
    shards         map[string]UserRepository
}

func NewUserShardingService() *UserShardingService {
    ch := NewConsistentHash(150) // 150 виртуальных узлов
    
    // Добавляем шарды
    ch.AddNode("shard1")
    ch.AddNode("shard2")
    ch.AddNode("shard3")
    
    return &UserShardingService{
        consistentHash: ch,
        shards: map[string]UserRepository{
            "shard1": NewUserRepository("db1"),
            "shard2": NewUserRepository("db2"),
            "shard3": NewUserRepository("db3"),
        },
    }
}

func (s *UserShardingService) GetUser(userID string) (*User, error) {
    shardName := s.consistentHash.GetNode(userID)
    shard := s.shards[shardName]
    return shard.FindByID(userID)
}
```

## Сложность алгоритмов

### Big O Notation в практических задачах

```go
// O(1) - постоянное время
func GetUserFromCache(cache map[string]*User, userID string) *User {
    return cache[userID]
}

// O(log n) - логарифмическое время
func BinarySearchUser(users []User, targetID string) *User {
    // Реализация бинарного поиска
    return nil
}

// O(n) - линейное время
func FindUserByEmail(users []User, email string) *User {
    for _, user := range users {
        if user.Email == email {
            return &user
        }
    }
    return nil
}

// O(n log n) - логарифмически-линейное время
func SortUsersByAge(users []User) {
    sort.Slice(users, func(i, j int) bool {
        return users[i].Age < users[j].Age
    })
}

// O(n²) - квадратичное время (избегать!)
func FindDuplicateUsers(users []User) []User {
    var duplicates []User
    for i := 0; i < len(users); i++ {
        for j := i + 1; j < len(users); j++ {
            if users[i].Email == users[j].Email {
                duplicates = append(duplicates, users[i])
            }
        }
    }
    return duplicates
}
```

## Оптимизация производительности

### 1. Выбор правильной структуры данных

```go
// Для поиска - используйте map вместо slice
type UserService struct {
    // ❌ Медленный поиск O(n)
    users []User
    
    // ✅ Быстрый поиск O(1)
    usersByID    map[string]*User
    usersByEmail map[string]*User
}

func (s *UserService) FindUserByEmail(email string) (*User, error) {
    // O(1) вместо O(n)
    if user, exists := s.usersByEmail[email]; exists {
        return user, nil
    }
    return nil, errors.New("user not found")
}
```

### 2. Кэширование вычислений

```go
type ProductRecommendationService struct {
    cache map[string][]Product
    mu    sync.RWMutex
}

func (s *ProductRecommendationService) GetRecommendations(userID string) []Product {
    s.mu.RLock()
    if cached, exists := s.cache[userID]; exists {
        s.mu.RUnlock()
        return cached
    }
    s.mu.RUnlock()
    
    // Дорогое вычисление
    recommendations := s.calculateRecommendations(userID)
    
    s.mu.Lock()
    s.cache[userID] = recommendations
    s.mu.Unlock()
    
    return recommendations
}
```

## 🔗 Связанные темы

### 🏗️ Основы
- **[Development Principles](development-principles.md)** - DRY, KISS для алгоритмов
- **[Functional Programming](functional-programming.md)** - Функциональные алгоритмы

### 🏛️ Архитектура
- **[Microservices Architecture](../architecture/microservices-architecture.md)** - Алгоритмы в распределенных системах
- **[Clean Architecture](clean-architecture.md)** - Структуры данных в архитектуре

### ⚙️ Технические навыки
- **[Databases](../technical-skills/databases.md)** - Алгоритмы индексации
- **[Concurrency & Async](../technical-skills/concurrency-async.md)** - Конкурентные алгоритмы
- **[Senior Technical Mastery](../technical-skills/senior-technical-mastery.md)** - Продвинутые алгоритмы

---

**Путь обучения**: Algorithms & Data Structures → [Databases](../technical-skills/databases.md) → [Concurrency](../technical-skills/concurrency-async.md)  
**Сложность**: ⭐⭐⭐⭐ (4/5)  
**Время изучения**: 4-6 недель  
**Практика**: Реализация алгоритмов в текущих проектах, оптимизация узких мест 