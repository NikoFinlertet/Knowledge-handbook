# 🧠 Углубленные основы глубокого обучения

## 🔬 Математические основы глубокого обучения

### Теория аппроксимации универсальными функциями
```
Теорема о универсальной аппроксимации:
├── Сети с одним скрытым слоем могут аппроксимировать любую непрерывную функцию
├── Требования: достаточное количество нейронов
├── Практические ограничения: экспоненциальный рост сложности
└── Глубокие сети: более эффективная аппроксимация сложных функций
```

#### Математическое обоснование
```python
import numpy as np
import torch
import torch.nn as nn
import matplotlib.pyplot as plt

class UniversalApproximator(nn.Module):
    """Демонстрация универсальной аппроксимации"""
    def __init__(self, hidden_size=100):
        super().__init__()
        self.hidden = nn.Linear(1, hidden_size)
        self.output = nn.Linear(hidden_size, 1)
        self.activation = nn.ReLU()
    
    def forward(self, x):
        hidden = self.activation(self.hidden(x))
        return self.output(hidden)

# Целевая функция для аппроксимации
def target_function(x):
    return np.sin(2 * np.pi * x) * np.exp(-x)

# Генерация данных
x_train = np.linspace(0, 2, 1000).reshape(-1, 1)
y_train = target_function(x_train.flatten())

# Обучение аппроксиматора
model = UniversalApproximator(hidden_size=50)
criterion = nn.MSELoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)

X_tensor = torch.FloatTensor(x_train)
y_tensor = torch.FloatTensor(y_train.reshape(-1, 1))

# Процесс обучения
for epoch in range(1000):
    optimizer.zero_grad()
    outputs = model(X_tensor)
    loss = criterion(outputs, y_tensor)
    loss.backward()
    optimizer.step()
    
    if epoch % 200 == 0:
        print(f'Epoch {epoch}, Loss: {loss.item():.6f}')
```

### Теория информации в глубоком обучении
```
Информационно-теоретические принципы:
├── Энтропия и взаимная информация
├── Information Bottleneck принцип
├── Сжатие и предсказание
├── Каскадное представление информации
└── Критическое обучение (Critical Learning)
```

#### Информационный узкий проход
```python
class InformationBottleneck:
    """Реализация Information Bottleneck принципа"""
    
    def __init__(self, beta=1.0):
        self.beta = beta  # Баланс между сжатием и предсказанием
    
    def mutual_information(self, X, Y):
        """Вычисление взаимной информации I(X;Y)"""
        # Упрощенная версия через энтропию
        joint_entropy = self.entropy(torch.cat([X, Y], dim=1))
        marginal_x = self.entropy(X)
        marginal_y = self.entropy(Y)
        return marginal_x + marginal_y - joint_entropy
    
    def entropy(self, X):
        """Дифференциальная энтропия"""
        # Аппроксимация через гауссовскую энтропию
        cov_matrix = torch.cov(X.T)
        det = torch.det(cov_matrix + 1e-8 * torch.eye(X.shape[1]))
        return 0.5 * torch.log(det * (2 * np.pi * np.e))
    
    def ib_loss(self, X, T, Y):
        """
        Information Bottleneck Loss
        L = -I(T;Y) + β*I(X;T)
        где T - промежуточное представление
        """
        compression_term = self.mutual_information(X, T)
        prediction_term = self.mutual_information(T, Y)
        return -prediction_term + self.beta * compression_term

# Пример использования в нейронной сети
class IBNet(nn.Module):
    def __init__(self, input_dim, hidden_dim, output_dim, beta=1.0):
        super().__init__()
        self.encoder = nn.Linear(input_dim, hidden_dim)
        self.decoder = nn.Linear(hidden_dim, output_dim)
        self.ib = InformationBottleneck(beta)
        
    def forward(self, x):
        t = torch.relu(self.encoder(x))  # Промежуточное представление
        y = self.decoder(t)
        return y, t
    
    def compute_ib_loss(self, x, y_true):
        y_pred, t = self.forward(x)
        pred_loss = nn.MSELoss()(y_pred, y_true)
        ib_loss = self.ib.ib_loss(x, t, y_true)
        return pred_loss + 0.1 * ib_loss
```

---

## 🏗️ Продвинутые архитектуры

### Attention механизмы и их варианты
```
Эволюция Attention:
├── Additive Attention (Bahdanau, 2014)
├── Multiplicative Attention (Luong, 2015)
├── Self-Attention (Transformer, 2017)
├── Multi-Head Attention
├── Sparse Attention (Local, Strided)
├── Linear Attention
└── Cross-Attention
```

#### Реализация различных типов Attention
```python
import math

class MultiHeadAttention(nn.Module):
    """Многоголовое внимание с различными вариантами"""
    
    def __init__(self, d_model, n_heads, attention_type='scaled_dot_product'):
        super().__init__()
        self.d_model = d_model
        self.n_heads = n_heads
        self.d_k = d_model // n_heads
        self.attention_type = attention_type
        
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
        
        if attention_type == 'additive':
            self.W_a = nn.Linear(2 * self.d_k, self.d_k)
            self.v_a = nn.Linear(self.d_k, 1)
    
    def scaled_dot_product_attention(self, Q, K, V, mask=None):
        """Классическое scaled dot-product attention"""
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        
        attn_weights = torch.softmax(scores, dim=-1)
        context = torch.matmul(attn_weights, V)
        return context, attn_weights
    
    def additive_attention(self, Q, K, V, mask=None):
        """Additive (Bahdanau) attention"""
        seq_len_q, seq_len_k = Q.size(-2), K.size(-2)
        
        # Расширение размерностей для broadcast
        Q_expanded = Q.unsqueeze(-2).expand(-1, -1, -1, seq_len_k, -1)
        K_expanded = K.unsqueeze(-3).expand(-1, -1, seq_len_q, -1, -1)
        
        # Конкатенация и проекция
        combined = torch.cat([Q_expanded, K_expanded], dim=-1)
        energy = torch.tanh(self.W_a(combined))
        scores = self.v_a(energy).squeeze(-1)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        
        attn_weights = torch.softmax(scores, dim=-1)
        context = torch.matmul(attn_weights, V)
        return context, attn_weights
    
    def sparse_attention(self, Q, K, V, window_size=4):
        """Локальное sparse attention"""
        seq_len = Q.size(-2)
        
        # Создание маски для локального внимания
        mask = torch.zeros(seq_len, seq_len)
        for i in range(seq_len):
            start = max(0, i - window_size // 2)
            end = min(seq_len, i + window_size // 2 + 1)
            mask[i, start:end] = 1
        
        return self.scaled_dot_product_attention(Q, K, V, mask)
    
    def forward(self, query, key, value, mask=None):
        batch_size = query.size(0)
        
        # Линейные проекции
        Q = self.W_q(query).view(batch_size, -1, self.n_heads, self.d_k).transpose(1, 2)
        K = self.W_k(key).view(batch_size, -1, self.n_heads, self.d_k).transpose(1, 2)
        V = self.W_v(value).view(batch_size, -1, self.n_heads, self.d_k).transpose(1, 2)
        
        # Выбор типа attention
        if self.attention_type == 'scaled_dot_product':
            attn_output, attn_weights = self.scaled_dot_product_attention(Q, K, V, mask)
        elif self.attention_type == 'additive':
            attn_output, attn_weights = self.additive_attention(Q, K, V, mask)
        elif self.attention_type == 'sparse':
            attn_output, attn_weights = self.sparse_attention(Q, K, V)
        
        # Конкатенация голов
        attn_output = attn_output.transpose(1, 2).contiguous().view(
            batch_size, -1, self.d_model)
        
        return self.W_o(attn_output), attn_weights
```

### Продвинутые типы слоев
```python
class GatedLinearUnit(nn.Module):
    """Gated Linear Unit (GLU) - улучшенная активация"""
    def __init__(self, input_dim, output_dim):
        super().__init__()
        self.linear = nn.Linear(input_dim, 2 * output_dim)
        
    def forward(self, x):
        gated = self.linear(x)
        values, gates = gated.chunk(2, dim=-1)
        return values * torch.sigmoid(gates)

class SwiGLU(nn.Module):
    """SwiGLU activation (используется в PaLM, LLaMA)"""
    def __init__(self, input_dim, hidden_dim):
        super().__init__()
        self.w1 = nn.Linear(input_dim, hidden_dim, bias=False)
        self.w2 = nn.Linear(hidden_dim, input_dim, bias=False)
        self.w3 = nn.Linear(input_dim, hidden_dim, bias=False)
        
    def forward(self, x):
        return self.w2(torch.silu(self.w1(x)) * self.w3(x))

class RMSNorm(nn.Module):
    """Root Mean Square Layer Normalization"""
    def __init__(self, dim, eps=1e-8):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(dim))
        
    def forward(self, x):
        rms = torch.sqrt(torch.mean(x**2, dim=-1, keepdim=True) + self.eps)
        return x / rms * self.weight

class ReZero(nn.Module):
    """ReZero: улучшение сходимости глубоких сетей"""
    def __init__(self, layer):
        super().__init__()
        self.layer = layer
        self.alpha = nn.Parameter(torch.zeros(1))
        
    def forward(self, x):
        return x + self.alpha * self.layer(x)
```

---

## 🎯 Методы оптимизации

### Адаптивные оптимизаторы второго порядка
```python
class AdamW(torch.optim.Optimizer):
    """AdamW с правильной weight decay регуляризацией"""
    
    def __init__(self, params, lr=1e-3, betas=(0.9, 0.999), eps=1e-8, weight_decay=1e-2):
        defaults = dict(lr=lr, betas=betas, eps=eps, weight_decay=weight_decay)
        super().__init__(params, defaults)
    
    def step(self, closure=None):
        for group in self.param_groups:
            for p in group['params']:
                if p.grad is None:
                    continue
                
                # Weight decay (не на momentum!)
                p.data.mul_(1 - group['lr'] * group['weight_decay'])
                
                grad = p.grad.data
                state = self.state[p]
                
                if len(state) == 0:
                    state['step'] = 0
                    state['exp_avg'] = torch.zeros_like(p.data)
                    state['exp_avg_sq'] = torch.zeros_like(p.data)
                
                exp_avg, exp_avg_sq = state['exp_avg'], state['exp_avg_sq']
                beta1, beta2 = group['betas']
                state['step'] += 1
                
                # Обновление моментов
                exp_avg.mul_(beta1).add_(grad, alpha=1 - beta1)
                exp_avg_sq.mul_(beta2).addcmul_(grad, grad, value=1 - beta2)
                
                # Bias correction
                bias_correction1 = 1 - beta1 ** state['step']
                bias_correction2 = 1 - beta2 ** state['step']
                
                step_size = group['lr'] / bias_correction1
                bias_correction2_sqrt = math.sqrt(bias_correction2)
                
                # Обновление параметров
                p.data.addcdiv_(exp_avg, exp_avg_sq.sqrt() / bias_correction2_sqrt + group['eps'], 
                              value=-step_size)

class LAMB(torch.optim.Optimizer):
    """LAMB optimizer для больших batch sizes"""
    
    def __init__(self, params, lr=1e-3, betas=(0.9, 0.999), eps=1e-6, weight_decay=0.01):
        defaults = dict(lr=lr, betas=betas, eps=eps, weight_decay=weight_decay)
        super().__init__(params, defaults)
    
    def step(self, closure=None):
        for group in self.param_groups:
            for p in group['params']:
                if p.grad is None:
                    continue
                
                grad = p.grad.data
                state = self.state[p]
                
                if len(state) == 0:
                    state['step'] = 0
                    state['exp_avg'] = torch.zeros_like(p.data)
                    state['exp_avg_sq'] = torch.zeros_like(p.data)
                
                exp_avg, exp_avg_sq = state['exp_avg'], state['exp_avg_sq']
                beta1, beta2 = group['betas']
                state['step'] += 1
                
                # Momentum и RMSprop
                exp_avg.mul_(beta1).add_(grad, alpha=1 - beta1)
                exp_avg_sq.mul_(beta2).addcmul_(grad, grad, value=1 - beta2)
                
                # Bias correction
                bias_correction1 = 1 - beta1 ** state['step']
                bias_correction2 = 1 - beta2 ** state['step']
                
                # Обновление Adam
                adam_step = exp_avg / bias_correction1 / (exp_avg_sq / bias_correction2).sqrt().add_(group['eps'])
                
                # Weight decay
                adam_step.add_(p.data, alpha=group['weight_decay'])
                
                # Layer-wise адаптация
                adam_norm = adam_step.norm()
                weight_norm = p.data.norm()
                
                if weight_norm > 0 and adam_norm > 0:
                    trust_ratio = weight_norm / adam_norm
                    trust_ratio = min(trust_ratio, 10.0)  # Ограничение
                else:
                    trust_ratio = 1.0
                
                # Финальное обновление
                p.data.add_(adam_step, alpha=-group['lr'] * trust_ratio)
```

### Планировщики скорости обучения
```python
class CosineAnnealingWarmup(torch.optim.lr_scheduler._LRScheduler):
    """Косинусный отжиг с warm-up периодом"""
    
    def __init__(self, optimizer, warmup_epochs, max_epochs, eta_min=0):
        self.warmup_epochs = warmup_epochs
        self.max_epochs = max_epochs
        self.eta_min = eta_min
        super().__init__(optimizer)
    
    def get_lr(self):
        if self.last_epoch < self.warmup_epochs:
            # Линейный warm-up
            return [base_lr * self.last_epoch / self.warmup_epochs for base_lr in self.base_lrs]
        else:
            # Косинусный отжиг
            progress = (self.last_epoch - self.warmup_epochs) / (self.max_epochs - self.warmup_epochs)
            return [self.eta_min + (base_lr - self.eta_min) * 
                   (1 + math.cos(math.pi * progress)) / 2 for base_lr in self.base_lrs]

class OneCycleLR(torch.optim.lr_scheduler._LRScheduler):
    """One Cycle Learning Rate Policy"""
    
    def __init__(self, optimizer, max_lr, total_steps, pct_start=0.3, anneal_strategy='cos'):
        self.max_lr = max_lr if isinstance(max_lr, list) else [max_lr] * len(optimizer.param_groups)
        self.total_steps = total_steps
        self.pct_start = pct_start
        self.anneal_strategy = anneal_strategy
        super().__init__(optimizer)
    
    def get_lr(self):
        lrs = []
        step_num = self.last_epoch
        
        for base_lr, max_lr in zip(self.base_lrs, self.max_lr):
            if step_num <= self.pct_start * self.total_steps:
                # Фаза роста
                pct = step_num / (self.pct_start * self.total_steps)
                lr = base_lr + (max_lr - base_lr) * pct
            else:
                # Фаза спада
                pct = (step_num - self.pct_start * self.total_steps) / ((1 - self.pct_start) * self.total_steps)
                if self.anneal_strategy == 'cos':
                    lr = base_lr + (max_lr - base_lr) * (1 + math.cos(math.pi * pct)) / 2
                else:  # linear
                    lr = max_lr - (max_lr - base_lr) * pct
            
            lrs.append(lr)
        
        return lrs
```

---

## 🛡️ Продвинутая регуляризация

### Stochastic Depth и DropPath
```python
class StochasticDepth(nn.Module):
    """Stochastic Depth для ResNet и DenseNet"""
    
    def __init__(self, p=0.5, mode='row'):
        super().__init__()
        self.p = p
        self.mode = mode
    
    def forward(self, x, residual):
        if not self.training:
            return x + residual
        
        batch_size = x.size(0)
        
        if self.mode == 'row':
            # Случайно пропускаем целые образцы
            mask = torch.rand(batch_size, 1, 1, 1, device=x.device) > self.p
        else:
            # Bernoulli для каждого элемента
            mask = torch.rand_like(x) > self.p
        
        return x + residual * mask.float() / (1 - self.p)

class DropPath(nn.Module):
    """Drop paths (Stochastic Depth) для Vision Transformers"""
    
    def __init__(self, drop_prob=0.0):
        super().__init__()
        self.drop_prob = drop_prob
        
    def forward(self, x):
        if not self.training or self.drop_prob == 0.0:
            return x
        
        keep_prob = 1 - self.drop_prob
        shape = (x.shape[0],) + (1,) * (x.ndim - 1)
        random_tensor = keep_prob + torch.rand(shape, dtype=x.dtype, device=x.device)
        random_tensor.floor_()
        
        return x.div(keep_prob) * random_tensor

class SpectrualNormalization(nn.Module):
    """Spectral Normalization для стабилизации обучения GANs"""
    
    def __init__(self, layer, n_power_iterations=1):
        super().__init__()
        self.layer = layer
        self.n_power_iterations = n_power_iterations
        
        # Инициализация u и v векторов
        weight = layer.weight
        height = weight.size(0)
        width = weight.view(height, -1).size(1)
        
        self.register_buffer('u', torch.randn(height))
        self.register_buffer('v', torch.randn(width))
        
        self.u.data = F.normalize(self.u.data)
        self.v.data = F.normalize(self.v.data)
    
    def power_iteration(self, weight):
        """Power iteration для вычисления спектральной нормы"""
        weight_mat = weight.view(weight.size(0), -1)
        
        for _ in range(self.n_power_iterations):
            # v = normalize(W^T * u)
            self.v.data = F.normalize(torch.mv(weight_mat.t(), self.u.data))
            # u = normalize(W * v)
            self.u.data = F.normalize(torch.mv(weight_mat, self.v.data))
        
        # σ = u^T * W * v
        sigma = torch.dot(self.u.data, torch.mv(weight_mat, self.v.data))
        return sigma
    
    def forward(self, *args, **kwargs):
        if self.training:
            sigma = self.power_iteration(self.layer.weight)
            # Нормализация весов
            self.layer.weight.data = self.layer.weight.data / sigma
        
        return self.layer(*args, **kwargs)
```

### Продвинутые техники нормализации
```python
class LayerScale(nn.Module):
    """Layer Scale для улучшения обучения глубоких Transformers"""
    
    def __init__(self, dim, init_values=1e-5):
        super().__init__()
        self.gamma = nn.Parameter(init_values * torch.ones(dim))
        
    def forward(self, x):
        return self.gamma * x

class GEGLU(nn.Module):
    """GEGLU activation из GLU variants improve Transformer"""
    
    def __init__(self, input_dim, output_dim):
        super().__init__()
        self.proj = nn.Linear(input_dim, output_dim * 2)
        
    def forward(self, x):
        x_proj = self.proj(x)
        x1, x2 = x_proj.chunk(2, dim=-1)
        return x1 * F.gelu(x2)

class AdaptiveLayerNorm(nn.Module):
    """Adaptive Layer Normalization с обучаемыми параметрами"""
    
    def __init__(self, num_features, condition_dim):
        super().__init__()
        self.num_features = num_features
        self.ln = nn.LayerNorm(num_features, affine=False)
        
        # Условные параметры
        self.scale_net = nn.Linear(condition_dim, num_features)
        self.shift_net = nn.Linear(condition_dim, num_features)
        
    def forward(self, x, condition):
        normalized = self.ln(x)
        
        scale = self.scale_net(condition)
        shift = self.shift_net(condition)
        
        return normalized * (1 + scale) + shift
```

---

## 🔬 Продвинутые техники обучения

### Curriculum Learning и Self-Paced Learning
```python
class CurriculumScheduler:
    """Планировщик для Curriculum Learning"""
    
    def __init__(self, dataset, difficulty_fn, initial_pace=0.1):
        self.dataset = dataset
        self.difficulty_fn = difficulty_fn
        self.pace = initial_pace
        self.current_subset_size = int(len(dataset) * initial_pace)
        
        # Сортировка по сложности
        difficulties = [difficulty_fn(sample) for sample in dataset]
        self.sorted_indices = sorted(range(len(dataset)), key=lambda i: difficulties[i])
    
    def get_current_subset(self):
        """Получить текущее подмножество для обучения"""
        current_indices = self.sorted_indices[:self.current_subset_size]
        return torch.utils.data.Subset(self.dataset, current_indices)
    
    def update_pace(self, epoch, total_epochs):
        """Обновление темпа обучения"""
        # Линейное увеличение сложности
        progress = epoch / total_epochs
        self.pace = 0.1 + 0.9 * progress
        self.current_subset_size = min(len(self.dataset), 
                                     int(len(self.dataset) * self.pace))

class SelfPacedLearning:
    """Self-Paced Learning implementation"""
    
    def __init__(self, initial_lambda=1.0, lambda_increase_rate=1.1):
        self.lam = initial_lambda
        self.lambda_increase_rate = lambda_increase_rate
        
    def compute_weights(self, losses):
        """Вычисление весов для образцов"""
        weights = torch.zeros_like(losses)
        weights[losses < self.lam] = 1.0
        return weights
    
    def spl_loss(self, losses, weights):
        """Self-paced learning loss"""
        return torch.sum(weights * losses) - self.lam * torch.sum(weights)
    
    def update_lambda(self):
        """Обновление параметра λ"""
        self.lam *= self.lambda_increase_rate

# Пример использования в тренировочном цикле
def train_with_curriculum(model, dataset, curriculum_scheduler, epochs=100):
    optimizer = torch.optim.Adam(model.parameters())
    
    for epoch in range(epochs):
        # Получение текущего подмножества данных
        current_dataset = curriculum_scheduler.get_current_subset()
        dataloader = torch.utils.data.DataLoader(current_dataset, batch_size=32)
        
        for batch in dataloader:
            # Стандартное обучение на текущем подмножестве
            optimizer.zero_grad()
            loss = model.compute_loss(batch)
            loss.backward()
            optimizer.step()
        
        # Обновление темпа
        curriculum_scheduler.update_pace(epoch, epochs)
```

### Meta-Learning и MAML
```python
class MAML(nn.Module):
    """Model-Agnostic Meta-Learning"""
    
    def __init__(self, model, inner_lr=0.01, meta_lr=0.001):
        super().__init__()
        self.model = model
        self.inner_lr = inner_lr
        self.meta_optimizer = torch.optim.Adam(self.model.parameters(), lr=meta_lr)
        
    def inner_update(self, task_data, task_labels):
        """Внутреннее обновление для конкретной задачи"""
        # Копируем параметры
        fast_weights = [p.clone() for p in self.model.parameters()]
        
        # Градиентный шаг для задачи
        outputs = self.model(task_data)
        loss = F.cross_entropy(outputs, task_labels)
        
        # Вычисляем градиенты
        grads = torch.autograd.grad(loss, self.model.parameters(), create_graph=True)
        
        # Обновляем веса
        fast_weights = [w - self.inner_lr * g for w, g in zip(fast_weights, grads)]
        
        return fast_weights
    
    def meta_update(self, meta_batch):
        """Мета-обновление параметров"""
        meta_loss = 0.0
        
        for task in meta_batch:
            support_x, support_y, query_x, query_y = task
            
            # Внутреннее обновление
            fast_weights = self.inner_update(support_x, support_y)
            
            # Вычисление потерь на query set с обновленными весами
            query_outputs = self.forward_with_params(query_x, fast_weights)
            meta_loss += F.cross_entropy(query_outputs, query_y)
        
        # Мета-градиентный шаг
        meta_loss /= len(meta_batch)
        self.meta_optimizer.zero_grad()
        meta_loss.backward()
        self.meta_optimizer.step()
        
        return meta_loss.item()
    
    def forward_with_params(self, x, params):
        """Forward pass с кастомными параметрами"""
        # Реализация зависит от архитектуры модели
        # Пример для простой сети
        for i, (layer, param) in enumerate(zip(self.model.modules(), params)):
            if isinstance(layer, nn.Linear):
                x = F.linear(x, param)
                if i < len(params) - 1:  # Не последний слой
                    x = F.relu(x)
        return x

class PrototypicalNetwork(nn.Module):
    """Prototypical Networks для few-shot learning"""
    
    def __init__(self, encoder):
        super().__init__()
        self.encoder = encoder
        
    def compute_prototypes(self, support_set, support_labels, n_ways):
        """Вычисление прототипов для каждого класса"""
        embeddings = self.encoder(support_set)
        prototypes = []
        
        for class_idx in range(n_ways):
            class_mask = (support_labels == class_idx)
            class_embeddings = embeddings[class_mask]
            prototype = class_embeddings.mean(dim=0)
            prototypes.append(prototype)
        
        return torch.stack(prototypes)
    
    def forward(self, support_set, support_labels, query_set, n_ways):
        # Вычисление прототипов
        prototypes = self.compute_prototypes(support_set, support_labels, n_ways)
        
        # Эмбеддинги для query set
        query_embeddings = self.encoder(query_set)
        
        # Вычисление расстояний до прототипов
        distances = torch.cdist(query_embeddings, prototypes)
        
        # Логиты как отрицательные расстояния
        logits = -distances
        
        return logits
```

---

## 📊 Анализ и интерпретация моделей

### Gradient-based методы интерпретации
```python
class GradCAM:
    """Gradient-weighted Class Activation Mapping"""
    
    def __init__(self, model, target_layer):
        self.model = model
        self.target_layer = target_layer
        self.gradients = None
        self.activations = None
        
        # Регистрация hooks
        self.target_layer.register_forward_hook(self.save_activation)
        self.target_layer.register_backward_hook(self.save_gradient)
    
    def save_activation(self, module, input, output):
        self.activations = output
    
    def save_gradient(self, module, grad_input, grad_output):
        self.gradients = grad_output[0]
    
    def generate_cam(self, input_image, class_idx):
        """Генерация CAM карты"""
        # Forward pass
        output = self.model(input_image)
        
        # Backward pass для целевого класса
        self.model.zero_grad()
        class_score = output[0, class_idx]
        class_score.backward()
        
        # Вычисление весов
        weights = torch.mean(self.gradients, dim=[2, 3])
        
        # Взвешенная комбинация активаций
        cam = torch.zeros(self.activations.shape[2:])
        for i, w in enumerate(weights[0]):
            cam += w * self.activations[0, i]
        
        # ReLU и нормализация
        cam = F.relu(cam)
        cam = (cam - cam.min()) / (cam.max() - cam.min())
        
        return cam

class IntegratedGradients:
    """Integrated Gradients для объяснения предсказаний"""
    
    def __init__(self, model):
        self.model = model
        
    def compute_gradients(self, inputs, target_class):
        """Вычисление градиентов"""
        inputs.requires_grad_(True)
        outputs = self.model(inputs)
        
        # Градиенты по целевому классу
        target_score = outputs[0, target_class]
        gradients = torch.autograd.grad(target_score, inputs)[0]
        
        return gradients
    
    def integrated_gradients(self, input_image, baseline=None, target_class=0, steps=50):
        """Вычисление Integrated Gradients"""
        if baseline is None:
            baseline = torch.zeros_like(input_image)
        
        # Генерация пути интерполяции
        alphas = torch.linspace(0, 1, steps)
        integrated_grads = torch.zeros_like(input_image)
        
        for alpha in alphas:
            # Интерполированное изображение
            interpolated = baseline + alpha * (input_image - baseline)
            
            # Градиенты
            grads = self.compute_gradients(interpolated, target_class)
            integrated_grads += grads
        
        # Среднее и умножение на разность
        integrated_grads /= steps
        integrated_grads *= (input_image - baseline)
        
        return integrated_grads

class SaliencyMaps:
    """Различные методы saliency maps"""
    
    @staticmethod
    def vanilla_gradient(model, input_image, target_class):
        """Простые градиенты"""
        input_image.requires_grad_(True)
        output = model(input_image)
        target_score = output[0, target_class]
        saliency = torch.autograd.grad(target_score, input_image)[0]
        return saliency.abs()
    
    @staticmethod
    def smooth_grad(model, input_image, target_class, noise_level=0.1, n_samples=50):
        """SmoothGrad - усреднение по зашумленным входам"""
        saliency_sum = torch.zeros_like(input_image)
        
        for _ in range(n_samples):
            # Добавление шума
            noise = torch.randn_like(input_image) * noise_level
            noisy_input = input_image + noise
            
            # Градиенты
            saliency = SaliencyMaps.vanilla_gradient(model, noisy_input, target_class)
            saliency_sum += saliency
        
        return saliency_sum / n_samples
    
    @staticmethod
    def guided_backprop(model, input_image, target_class):
        """Guided Backpropagation"""
        def relu_hook(module, grad_in, grad_out):
            # Модифицированный ReLU для guided backprop
            return (torch.clamp(grad_in[0], min=0),)
        
        # Регистрация hooks для всех ReLU слоев
        handles = []
        for module in model.modules():
            if isinstance(module, nn.ReLU):
                handles.append(module.register_backward_hook(relu_hook))
        
        # Вычисление градиентов
        saliency = SaliencyMaps.vanilla_gradient(model, input_image, target_class)
        
        # Удаление hooks
        for handle in handles:
            handle.remove()
        
        return saliency
```

---

## 📚 Связанные темы и ресурсы

### Связанные разделы
- [[neural-networks-architecture|Архитектуры нейронных сетей]] - конкретные архитектуры
- [[machine-learning-theory|Теория машинного обучения]] - математические основы
- [[large-language-models|Большие языковые модели]] - применение в NLP
- [[computer-vision|Компьютерное зрение]] - применение в CV
- [[ai-in-production|AI в продакшене]] - развертывание глубоких моделей

### Дополнительные ресурсы
```
Книги и курсы:
├── Deep Learning (Goodfellow, Bengio, Courville)
├── Pattern Recognition and Machine Learning (Bishop)
├── CS231n: Convolutional Neural Networks (Stanford)
├── CS224n: Natural Language Processing (Stanford)
├── Fast.ai Practical Deep Learning
└── Deep Learning Specialization (Coursera)

Исследовательские конференции:
├── NeurIPS (Neural Information Processing Systems)
├── ICML (International Conference on Machine Learning)
├── ICLR (International Conference on Learning Representations)
├── CVPR (Computer Vision and Pattern Recognition)
└── EMNLP/ACL (Natural Language Processing)
```

### Практические инструменты
```python
# Основные фреймворки
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import DataLoader

# Специализированные библиотеки
import transformers
import timm  # PyTorch Image Models
import torchvision
import albumentations  # Аугментации

# Визуализация и анализ
import wandb
import tensorboard
import captum  # Интерпретация моделей

# Оптимизация и эксперименты
import optuna
import ray.tune
import hydra
```

---

💡 **Ключевая идея:** Глубокое обучение - это не просто "большие нейронные сети", а сложная область со своими математическими основами, продвинутыми техниками и современными архитектурами. Понимание этих концепций критически важно для создания эффективных AI систем! 🧠🚀 