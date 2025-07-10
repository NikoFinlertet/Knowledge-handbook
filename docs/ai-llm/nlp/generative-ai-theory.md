# 🎨 Углубленная теория генеративного ИИ

## 🔬 Математические основы генеративного моделирования

### Теория вероятностей в генеративных моделях
```
Фундаментальные концепции:
├── Плотность вероятности p(x)
├── Условная плотность p(x|z)
├── Апостериорная плотность p(z|x)
├── Маргинальная вероятность
├── Принцип максимального правдоподобия
└── Вариационный вывод (Variational Inference)
```

#### Вариационный автоэнкодер (VAE) - математические основы
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import numpy as np

class VAE(nn.Module):
    """Полная математическая реализация VAE"""
    
    def __init__(self, input_dim, latent_dim, hidden_dim=400):
        super().__init__()
        self.latent_dim = latent_dim
        
        # Encoder q(z|x)
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU()
        )
        
        # Параметры латентного распределения
        self.fc_mu = nn.Linear(hidden_dim, latent_dim)
        self.fc_logvar = nn.Linear(hidden_dim, latent_dim)
        
        # Decoder p(x|z)
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, input_dim),
            nn.Sigmoid()  # Для данных в [0,1]
        )
    
    def encode(self, x):
        """Энкодер: q(z|x)"""
        h = self.encoder(x)
        mu = self.fc_mu(h)
        logvar = self.fc_logvar(h)
        return mu, logvar
    
    def reparametrize(self, mu, logvar):
        """Reparametrization trick: z = μ + σ⊙ε, где ε ~ N(0,I)"""
        std = torch.exp(0.5 * logvar)
        eps = torch.randn_like(std)
        return mu + eps * std
    
    def decode(self, z):
        """Декодер: p(x|z)"""
        return self.decoder(z)
    
    def forward(self, x):
        mu, logvar = self.encode(x)
        z = self.reparametrize(mu, logvar)
        x_recon = self.decode(z)
        return x_recon, mu, logvar, z
    
    def loss_function(self, x_recon, x, mu, logvar):
        """
        VAE Loss = Reconstruction Loss + KL Divergence
        
        L = E_q(z|x)[log p(x|z)] - KL(q(z|x) || p(z))
        
        где KL(q(z|x) || p(z)) = KL(N(μ,σ²) || N(0,I))
                                = 0.5 * Σ(1 + log(σ²) - μ² - σ²)
        """
        # Reconstruction loss (Binary Cross Entropy)
        recon_loss = F.binary_cross_entropy(x_recon, x, reduction='sum')
        
        # KL divergence loss
        # KL(N(μ,σ²) || N(0,I)) = 0.5 * Σ(1 + log(σ²) - μ² - σ²)
        kl_loss = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())
        
        return recon_loss + kl_loss, recon_loss, kl_loss
    
    def sample(self, num_samples, device):
        """Генерация новых образцов из априорного распределения"""
        with torch.no_grad():
            z = torch.randn(num_samples, self.latent_dim, device=device)
            samples = self.decode(z)
            return samples

# Теоретический анализ VAE
class VAEAnalysis:
    """Анализ теоретических свойств VAE"""
    
    @staticmethod
    def evidence_lower_bound(x_recon, x, mu, logvar):
        """
        Evidence Lower Bound (ELBO):
        ELBO = E_q(z|x)[log p(x|z)] - KL(q(z|x) || p(z))
        
        log p(x) >= ELBO
        """
        log_likelihood = -F.binary_cross_entropy(x_recon, x, reduction='sum')
        kl_divergence = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())
        
        elbo = log_likelihood - kl_divergence
        return elbo, log_likelihood, kl_divergence
    
    @staticmethod
    def posterior_collapse_metric(mu, logvar):
        """
        Метрика для определения posterior collapse:
        AU (Active Units) - количество активных латентных измерений
        """
        # Вариация каждого латентного измерения по батчу
        mu_var = torch.var(mu, dim=0)
        
        # Активное измерение если его вариация больше порога
        threshold = 0.01
        active_units = (mu_var > threshold).sum().item()
        
        return active_units, mu_var
    
    @staticmethod
    def mutual_information_estimate(z, x_recon):
        """Оценка взаимной информации I(z; x)"""
        # Упрощенная оценка через корреляцию
        z_flat = z.view(z.size(0), -1)
        x_flat = x_recon.view(x_recon.size(0), -1)
        
        # Центрирование
        z_centered = z_flat - z_flat.mean(dim=0, keepdim=True)
        x_centered = x_flat - x_flat.mean(dim=0, keepdim=True)
        
        # Ковариационная матрица
        cov_zx = torch.mm(z_centered.t(), x_centered) / (z.size(0) - 1)
        
        # Приближение MI через спектральную норму
        mi_estimate = torch.norm(cov_zx, p='fro') / (z_flat.size(1) * x_flat.size(1))
        
        return mi_estimate
```

---

## 🎭 Generative Adversarial Networks (GANs)

### Теория игр в GANs
```
Математическая формулировка:
├── Minimax игра: min_G max_D V(D,G)
├── Nash equilibrium
├── Wasserstein distance
├── f-divergences
└── Mode collapse и training instability
```

#### Продвинутая GAN архитектура
```python
class WassersteinGAN(nn.Module):
    """Wasserstein GAN с Gradient Penalty"""
    
    def __init__(self, latent_dim, img_shape):
        super().__init__()
        self.latent_dim = latent_dim
        self.img_shape = img_shape
        
        # Generator
        self.generator = self._make_generator()
        
        # Critic (не дискриминатор в WGAN)
        self.critic = self._make_critic()
    
    def _make_generator(self):
        return nn.Sequential(
            nn.Linear(self.latent_dim, 128),
            nn.LeakyReLU(0.2),
            nn.Linear(128, 256),
            nn.BatchNorm1d(256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 512),
            nn.BatchNorm1d(512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, np.prod(self.img_shape)),
            nn.Tanh()
        )
    
    def _make_critic(self):
        return nn.Sequential(
            nn.Linear(np.prod(self.img_shape), 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 128),
            nn.LeakyReLU(0.2),
            nn.Linear(128, 1)
        )
    
    def gradient_penalty(self, real_samples, fake_samples, device):
        """
        Gradient Penalty для WGAN-GP:
        GP = E[(||∇_x D(x)||_2 - 1)²]
        где x ~ uniform[real_samples, fake_samples]
        """
        batch_size = real_samples.size(0)
        
        # Случайная интерполяция
        alpha = torch.rand(batch_size, 1, device=device)
        alpha = alpha.expand_as(real_samples)
        
        interpolated = alpha * real_samples + (1 - alpha) * fake_samples
        interpolated.requires_grad_(True)
        
        # Критик для интерполированных образцов
        critic_interpolated = self.critic(interpolated)
        
        # Градиенты
        gradients = torch.autograd.grad(
            outputs=critic_interpolated,
            inputs=interpolated,
            grad_outputs=torch.ones_like(critic_interpolated),
            create_graph=True,
            retain_graph=True,
            only_inputs=True
        )[0]
        
        # Gradient penalty
        gradients_norm = torch.sqrt(torch.sum(gradients ** 2, dim=1) + 1e-12)
        gradient_penalty = torch.mean((gradients_norm - 1) ** 2)
        
        return gradient_penalty
    
    def wasserstein_loss(self, real_samples, fake_samples, device, lambda_gp=10):
        """
        Wasserstein Loss с Gradient Penalty:
        L_D = E[D(fake)] - E[D(real)] + λ * GP
        L_G = -E[D(fake)]
        """
        # Critic scores
        real_score = self.critic(real_samples)
        fake_score = self.critic(fake_samples)
        
        # Wasserstein distance
        wasserstein_d = torch.mean(fake_score) - torch.mean(real_score)
        
        # Gradient penalty
        gp = self.gradient_penalty(real_samples, fake_samples, device)
        
        # Critic loss
        critic_loss = wasserstein_d + lambda_gp * gp
        
        # Generator loss
        generator_loss = -torch.mean(fake_score)
        
        return critic_loss, generator_loss, wasserstein_d, gp

class StyleGAN_Components:
    """Компоненты StyleGAN архитектуры"""
    
    @staticmethod
    def adaptive_instance_norm(content_feat, style_feat):
        """
        Adaptive Instance Normalization:
        AdaIN(x, y) = y_s * ((x - μ(x)) / σ(x)) + y_b
        
        где y_s, y_b получаются из style кода
        """
        size = content_feat.size()
        style_mean = content_feat.view(size[0], size[1], -1).mean(dim=2).view(size[0], size[1], 1, 1)
        style_std = content_feat.view(size[0], size[1], -1).std(dim=2).view(size[0], size[1], 1, 1)
        
        normalized_feat = (content_feat - style_mean.expand(size)) / style_std.expand(size)
        
        # Style parameters from style_feat
        style_A = style_feat[:, :size[1]]  # scale
        style_B = style_feat[:, size[1]:]  # bias
        
        return normalized_feat * style_A.unsqueeze(2).unsqueeze(3) + style_B.unsqueeze(2).unsqueeze(3)
    
    @staticmethod
    def style_modulation(w, style_dim):
        """
        Style modulation в StyleGAN:
        s = A(w) где A - affine transformation
        """
        # Mapping network результат w -> style parameters
        # Это упрощенная версия
        style_scale = torch.ones(w.size(0), style_dim, device=w.device)
        style_bias = torch.zeros(w.size(0), style_dim, device=w.device)
        
        return style_scale, style_bias
    
    @staticmethod
    def progressive_growing_loss(real_imgs, fake_imgs, alpha, low_res_real, low_res_fake):
        """
        Progressive Growing loss:
        L = (1-α)*L_low + α*L_high
        """
        # High resolution loss
        high_res_loss = F.mse_loss(fake_imgs, real_imgs)
        
        # Low resolution loss
        low_res_loss = F.mse_loss(low_res_fake, low_res_real)
        
        # Weighted combination
        total_loss = (1 - alpha) * low_res_loss + alpha * high_res_loss
        
        return total_loss

# Анализ режимов коллапса
class ModeCollapseAnalysis:
    """Анализ mode collapse в GANs"""
    
    @staticmethod
    def inception_score(images, model, num_splits=10):
        """
        Inception Score:
        IS = exp(E_x[KL(p(y|x) || p(y))])
        """
        # Предсказания Inception модели
        with torch.no_grad():
            preds = model(images)
            probs = F.softmax(preds, dim=1)
        
        # Разделение на chunks
        split_scores = []
        for i in range(num_splits):
            part = probs[i * len(probs) // num_splits: (i + 1) * len(probs) // num_splits]
            
            # p(y|x)
            py_x = part
            # p(y) = E_x[p(y|x)]
            py = py_x.mean(dim=0, keepdim=True)
            
            # KL divergence
            kl_div = torch.sum(py_x * (torch.log(py_x) - torch.log(py)), dim=1)
            split_scores.append(torch.exp(torch.mean(kl_div)))
        
        return torch.mean(torch.stack(split_scores)), torch.std(torch.stack(split_scores))
    
    @staticmethod
    def frechet_inception_distance(real_features, fake_features):
        """
        Fréchet Inception Distance:
        FID = ||μ_r - μ_f||² + Tr(Σ_r + Σ_f - 2√(Σ_r * Σ_f))
        """
        # Статистики реальных изображений
        mu_real = torch.mean(real_features, dim=0)
        sigma_real = torch.cov(real_features.T)
        
        # Статистики сгенерированных изображений
        mu_fake = torch.mean(fake_features, dim=0)
        sigma_fake = torch.cov(fake_features.T)
        
        # FID computation
        diff = mu_real - mu_fake
        sqrt_sigma = torch.sqrt(sigma_real @ sigma_fake)
        
        fid = torch.sum(diff ** 2) + torch.trace(sigma_real + sigma_fake - 2 * sqrt_sigma)
        return fid.item()
```

---

## 🌊 Flow-based модели

### Normalizing Flows
```python
class CouplingLayer(nn.Module):
    """Affine Coupling Layer для Normalizing Flows"""
    
    def __init__(self, dim, hidden_dim, mask):
        super().__init__()
        self.mask = mask
        
        # Сети для scale и translation
        self.scale_net = nn.Sequential(
            nn.Linear(dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, dim),
            nn.Tanh()  # Ограничиваем scale
        )
        
        self.translate_net = nn.Sequential(
            nn.Linear(dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, dim)
        )
    
    def forward(self, x, reverse=False):
        """
        Forward: y = x * exp(s(x_masked)) + t(x_masked)
        Reverse: x = (y - t(y_masked)) * exp(-s(y_masked))
        """
        if not reverse:
            # Forward direction
            x_masked = x * self.mask
            s = self.scale_net(x_masked)
            t = self.translate_net(x_masked)
            
            # Применение трансформации только к немаскированным элементам
            y = x * self.mask + (1 - self.mask) * (x * torch.exp(s) + t)
            
            # Log determinant Jacobian
            log_det_j = torch.sum((1 - self.mask) * s, dim=1)
            
            return y, log_det_j
        else:
            # Reverse direction
            y_masked = x * self.mask
            s = self.scale_net(y_masked)
            t = self.translate_net(y_masked)
            
            x = x * self.mask + (1 - self.mask) * ((x - t) * torch.exp(-s))
            
            # Log determinant Jacobian (обратное)
            log_det_j = -torch.sum((1 - self.mask) * s, dim=1)
            
            return x, log_det_j

class RealNVP(nn.Module):
    """Real-valued Non-Volume Preserving (RealNVP) Flow"""
    
    def __init__(self, dim, num_layers, hidden_dim):
        super().__init__()
        self.dim = dim
        
        # Чередующиеся маски
        masks = []
        for i in range(num_layers):
            mask = torch.zeros(dim)
            mask[i % 2::2] = 1  # Чередующиеся элементы
            masks.append(mask)
        
        self.layers = nn.ModuleList([
            CouplingLayer(dim, hidden_dim, masks[i])
            for i in range(num_layers)
        ])
    
    def forward(self, x, reverse=False):
        log_det_j_total = 0
        
        if not reverse:
            # x -> z (data to noise)
            for layer in self.layers:
                x, log_det_j = layer(x, reverse=False)
                log_det_j_total += log_det_j
        else:
            # z -> x (noise to data)
            for layer in reversed(self.layers):
                x, log_det_j = layer(x, reverse=True)
                log_det_j_total += log_det_j
        
        return x, log_det_j_total
    
    def log_likelihood(self, x):
        """
        Log-likelihood под моделью:
        log p(x) = log p(z) + log|det(J)|
        где z = f(x) и J - Jacobian трансформации
        """
        z, log_det_j = self.forward(x, reverse=False)
        
        # Prior log-likelihood (стандартная гауссовская)
        log_prior = -0.5 * torch.sum(z**2, dim=1) - 0.5 * self.dim * torch.log(2 * torch.tensor(np.pi))
        
        # Total log-likelihood
        log_likelihood = log_prior + log_det_j
        
        return log_likelihood, z
    
    def sample(self, num_samples, device):
        """Генерация образцов"""
        # Сэмплирование из prior
        z = torch.randn(num_samples, self.dim, device=device)
        
        # Трансформация в data space
        x, _ = self.forward(z, reverse=True)
        
        return x

class NeuralSplineFlow(nn.Module):
    """Neural Spline Flow - более выразительные трансформации"""
    
    def __init__(self, dim, num_bins=10, tail_bound=3.0):
        super().__init__()
        self.num_bins = num_bins
        self.tail_bound = tail_bound
        
        # Параметры сплайна
        self.unnormalized_widths = nn.Parameter(torch.randn(dim, num_bins))
        self.unnormalized_heights = nn.Parameter(torch.randn(dim, num_bins))
        self.unnormalized_derivatives = nn.Parameter(torch.randn(dim, num_bins + 1))
    
    def rational_quadratic_spline(self, inputs, unnormalized_widths, unnormalized_heights, unnormalized_derivatives):
        """
        Rational Quadratic Spline transformation
        Более гибкая альтернатива affine coupling
        """
        # Нормализация параметров
        widths = F.softmax(unnormalized_widths, dim=-1) * 2 * self.tail_bound
        heights = F.softmax(unnormalized_heights, dim=-1) * 2 * self.tail_bound
        derivatives = F.softplus(unnormalized_derivatives) + 1e-6
        
        # Вычисление knots
        cumwidths = torch.cumsum(widths, dim=-1)
        cumwidths = F.pad(cumwidths, (1, 0), value=0.0) - self.tail_bound
        
        cumheights = torch.cumsum(heights, dim=-1)
        cumheights = F.pad(cumheights, (1, 0), value=0.0) - self.tail_bound
        
        # Поиск интервала для каждого input
        bin_idx = torch.searchsorted(cumwidths[..., 1:], inputs)
        bin_idx = torch.clamp(bin_idx, 0, self.num_bins - 1)
        
        # Параметры для выбранного bin
        input_left_cumwidths = cumwidths.gather(-1, bin_idx)
        input_right_cumwidths = cumwidths.gather(-1, bin_idx + 1)
        
        input_left_cumheights = cumheights.gather(-1, bin_idx)
        input_right_cumheights = cumheights.gather(-1, bin_idx + 1)
        
        input_left_derivatives = derivatives.gather(-1, bin_idx)
        input_right_derivatives = derivatives.gather(-1, bin_idx + 1)
        
        # Рациональная квадратичная функция
        theta = (inputs - input_left_cumwidths) / (input_right_cumwidths - input_left_cumwidths)
        
        outputs = input_left_cumheights + (input_right_cumheights - input_left_cumheights) * (
            input_left_derivatives * theta +
            input_right_derivatives * theta * (1 - theta) +
            theta * (1 - theta)
        ) / (
            input_left_derivatives + input_right_derivatives - 2 + 
            (input_right_derivatives - input_left_derivatives) * theta
        )
        
        # Log determinant Jacobian
        log_det_j = torch.log(
            (input_right_cumheights - input_left_cumheights) *
            (input_left_derivatives + input_right_derivatives - 2 + 
             (input_right_derivatives - input_left_derivatives) * theta) ** 2 /
            ((input_right_cumwidths - input_left_cumwidths) *
             (input_left_derivatives * (1 - theta) + input_right_derivatives * theta))
        )
        
        return outputs, log_det_j
```

---

## 🔮 Автогрегрессивные модели

### PixelCNN и WaveNet архитектуры
```python
class MaskedConv2d(nn.Module):
    """Masked Convolution для PixelCNN"""
    
    def __init__(self, mask_type, in_channels, out_channels, kernel_size, padding):
        super().__init__()
        self.mask_type = mask_type
        
        self.conv = nn.Conv2d(in_channels, out_channels, kernel_size, padding=padding)
        
        # Создание маски
        self.register_buffer('mask', self.conv.weight.data.clone())
        _, _, kH, kW = self.conv.weight.size()
        self.mask.fill_(1)
        
        # Маскирование будущих пикселей
        self.mask[:, :, kH // 2, kW // 2 + (mask_type == 'B'):] = 0
        self.mask[:, :, kH // 2 + 1:] = 0
        
    def forward(self, x):
        self.conv.weight.data *= self.mask
        return self.conv(x)

class GatedResidualBlock(nn.Module):
    """Gated Residual Block для WaveNet/PixelCNN"""
    
    def __init__(self, channels, kernel_size, dilation=1):
        super().__init__()
        
        self.conv_filter = nn.Conv1d(channels, channels, kernel_size, 
                                   dilation=dilation, padding=dilation * (kernel_size - 1) // 2)
        self.conv_gate = nn.Conv1d(channels, channels, kernel_size,
                                 dilation=dilation, padding=dilation * (kernel_size - 1) // 2)
        
        self.conv_out = nn.Conv1d(channels, channels, 1)
        self.conv_skip = nn.Conv1d(channels, channels, 1)
        
    def forward(self, x):
        """
        Gated activation: tanh(W_f * x) ⊙ σ(W_g * x)
        где ⊙ - element-wise multiplication
        """
        residual = x
        
        # Gated activation
        filter_out = torch.tanh(self.conv_filter(x))
        gate_out = torch.sigmoid(self.conv_gate(x))
        gated = filter_out * gate_out
        
        # Output and skip connections
        out = self.conv_out(gated)
        skip = self.conv_skip(gated)
        
        return (out + residual), skip

class WaveNet(nn.Module):
    """WaveNet для генерации аудио"""
    
    def __init__(self, num_layers=30, num_blocks=3, residual_channels=512, 
                 gate_channels=512, skip_channels=256, end_channels=256, output_classes=256):
        super().__init__()
        
        self.num_layers = num_layers
        self.num_blocks = num_blocks
        
        # Входная конволюция
        self.start_conv = nn.Conv1d(1, residual_channels, 1)
        
        # Residual blocks с увеличивающимся dilation
        self.residual_blocks = nn.ModuleList()
        self.dilations = []
        
        for block in range(num_blocks):
            for layer in range(num_layers):
                dilation = 2 ** layer
                self.dilations.append(dilation)
                self.residual_blocks.append(
                    GatedResidualBlock(residual_channels, kernel_size=2, dilation=dilation)
                )
        
        # Выходные слои
        self.end_conv_1 = nn.Conv1d(skip_channels, end_channels, 1)
        self.end_conv_2 = nn.Conv1d(end_channels, output_classes, 1)
        
    def forward(self, x):
        """
        x: [batch_size, 1, length] - raw audio waveform
        """
        x = self.start_conv(x)
        
        skip_connections = []
        
        # Residual blocks
        for block in self.residual_blocks:
            x, skip = block(x)
            skip_connections.append(skip)
        
        # Суммирование skip connections
        out = torch.stack(skip_connections, dim=0).sum(dim=0)
        
        # Финальные конволюции
        out = F.relu(out)
        out = self.end_conv_1(out)
        out = F.relu(out)
        out = self.end_conv_2(out)
        
        return out
    
    def generate(self, length, temperature=1.0, device='cpu'):
        """Автогрегрессивная генерация"""
        # Начинаем с тишины
        generated = torch.zeros(1, 1, 1, device=device)
        
        with torch.no_grad():
            for i in range(length):
                # Предсказание следующего сэмпла
                logits = self.forward(generated)
                
                # Последний временной шаг
                next_sample_logits = logits[:, :, -1] / temperature
                
                # Сэмплирование
                next_sample = torch.multinomial(F.softmax(next_sample_logits, dim=1), 1)
                next_sample = (next_sample.float() - 127.5) / 127.5  # Нормализация
                
                # Добавление к последовательности
                generated = torch.cat([generated, next_sample.unsqueeze(-1)], dim=2)
        
        return generated.squeeze().cpu().numpy()

# Современные автогрегрессивные методы
class GPTBlock(nn.Module):
    """Transformer block для автогрегрессивного моделирования"""
    
    def __init__(self, d_model, n_heads, d_ff, dropout=0.1):
        super().__init__()
        
        self.attention = nn.MultiheadAttention(d_model, n_heads, dropout=dropout, batch_first=True)
        self.feed_forward = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Linear(d_ff, d_model),
            nn.Dropout(dropout)
        )
        
        self.ln1 = nn.LayerNorm(d_model)
        self.ln2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)
        
    def forward(self, x, mask=None):
        # Self-attention с causal mask
        attn_out, _ = self.attention(x, x, x, attn_mask=mask, need_weights=False)
        x = self.ln1(x + self.dropout(attn_out))
        
        # Feed-forward
        ff_out = self.feed_forward(x)
        x = self.ln2(x + ff_out)
        
        return x
    
    @staticmethod
    def create_causal_mask(seq_len, device):
        """Создание causal mask для автогрегрессии"""
        mask = torch.triu(torch.ones(seq_len, seq_len, device=device), diagonal=1)
        mask = mask.masked_fill(mask == 1, float('-inf'))
        return mask
```

---

## 📊 Оценка генеративных моделей

### Метрики качества
```python
class GenerativeMetrics:
    """Комплексная оценка генеративных моделей"""
    
    @staticmethod
    def perplexity(model, data_loader, device):
        """
        Perplexity для автогрегрессивных моделей:
        PPL = exp(-1/N * Σ log p(x_i))
        """
        model.eval()
        total_loss = 0
        total_tokens = 0
        
        with torch.no_grad():
            for batch in data_loader:
                inputs, targets = batch
                inputs, targets = inputs.to(device), targets.to(device)
                
                outputs = model(inputs)
                loss = F.cross_entropy(outputs.view(-1, outputs.size(-1)), 
                                     targets.view(-1), reduction='sum')
                
                total_loss += loss.item()
                total_tokens += targets.numel()
        
        avg_loss = total_loss / total_tokens
        perplexity = torch.exp(torch.tensor(avg_loss))
        
        return perplexity.item()
    
    @staticmethod
    def coverage_and_mmr(real_features, generated_features, k=5):
        """
        Coverage и MMD (Maximum Mean Discrepancy):
        - Coverage: доля реальных образцов, близких к сгенерированным
        - MMD: расстояние между распределениями в feature space
        """
        # Coverage
        real_to_gen_dists = torch.cdist(real_features, generated_features)
        min_dists, _ = torch.min(real_to_gen_dists, dim=1)
        threshold = torch.quantile(min_dists, 0.05)  # 5% quantile
        coverage = (min_dists < threshold).float().mean()
        
        # MMD с RBF ядром
        def rbf_kernel(x, y, sigma=1.0):
            pairwise_dists = torch.cdist(x, y) ** 2
            return torch.exp(-pairwise_dists / (2 * sigma ** 2))
        
        # Kernel matrices
        K_real_real = rbf_kernel(real_features, real_features)
        K_gen_gen = rbf_kernel(generated_features, generated_features)
        K_real_gen = rbf_kernel(real_features, generated_features)
        
        # MMD^2 estimator
        mmd_squared = (K_real_real.mean() + K_gen_gen.mean() - 2 * K_real_gen.mean())
        
        return coverage.item(), torch.sqrt(torch.clamp(mmd_squared, min=0)).item()
    
    @staticmethod
    def precision_recall_f1(real_features, generated_features, k=3):
        """
        Precision, Recall и F1 для генеративных моделей
        На основе k-NN в feature space
        """
        # k-NN для precision (сгенерированные -> реальные)
        gen_to_real_dists = torch.cdist(generated_features, real_features)
        knn_gen_to_real = torch.topk(gen_to_real_dists, k, dim=1, largest=False)[1]
        
        # k-NN для recall (реальные -> сгенерированные)
        real_to_gen_dists = torch.cdist(real_features, generated_features)
        knn_real_to_gen = torch.topk(real_to_gen_dists, k, dim=1, largest=False)[1]
        
        # Precision: доля сгенерированных, близких к реальным
        precision = len(torch.unique(knn_gen_to_real)) / len(generated_features)
        
        # Recall: доля реальных, близких к сгенерированным
        recall = len(torch.unique(knn_real_to_gen)) / len(real_features)
        
        # F1 score
        f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0
        
        return precision, recall, f1
    
    @staticmethod
    def likelihood_ratio_test(model, real_data, generated_data):
        """
        Likelihood Ratio Test для оценки качества модели
        """
        model.eval()
        
        with torch.no_grad():
            # Log-likelihood реальных данных
            real_ll = model.log_likelihood(real_data).mean()
            
            # Log-likelihood сгенерированных данных
            gen_ll = model.log_likelihood(generated_data).mean()
            
            # Отношение правдоподобий
            lr = torch.exp(real_ll - gen_ll)
        
        return lr.item(), real_ll.item(), gen_ll.item()

# Визуализация и анализ
class GenerativeAnalysis:
    """Инструменты для анализа генеративных моделей"""
    
    @staticmethod
    def latent_space_interpolation(model, z1, z2, num_steps=10):
        """Интерполяция в латентном пространстве"""
        alphas = torch.linspace(0, 1, num_steps)
        interpolated_samples = []
        
        with torch.no_grad():
            for alpha in alphas:
                z_interp = (1 - alpha) * z1 + alpha * z2
                sample = model.decode(z_interp)
                interpolated_samples.append(sample)
        
        return torch.stack(interpolated_samples)
    
    @staticmethod
    def attribute_manipulation(model, z, direction, steps=[-3, -2, -1, 0, 1, 2, 3]):
        """Манипуляция атрибутами в латентном пространстве"""
        manipulated_samples = []
        
        with torch.no_grad():
            for step in steps:
                z_modified = z + step * direction
                sample = model.decode(z_modified)
                manipulated_samples.append(sample)
        
        return torch.stack(manipulated_samples)
    
    @staticmethod
    def principal_components_analysis(latent_codes):
        """PCA для анализа латентного пространства"""
        # Центрирование
        centered = latent_codes - latent_codes.mean(dim=0, keepdim=True)
        
        # SVD
        U, S, V = torch.svd(centered)
        
        # Главные компоненты
        principal_components = V
        explained_variance = (S ** 2) / (latent_codes.size(0) - 1)
        explained_variance_ratio = explained_variance / explained_variance.sum()
        
        return principal_components, explained_variance_ratio
```

---

## 📚 Связанные материалы

### Теоретические основы
- [[deep-learning-foundations|Основы глубокого обучения]] - математические основы
- [[machine-learning-theory|Теория машинного обучения]] - вероятностные методы
- [[neural-networks-architecture|Архитектуры нейронных сетей]] - специализированные архитектуры

### Практические применения
- [[large-language-models|Большие языковые модели]] - генеративные модели для текста
- [[computer-vision|Компьютерное зрение]] - генерация изображений
- [[ai-in-production|AI в продакшене]] - развертывание генеративных моделей

---

🎨 **Ключевая идея:** Генеративный ИИ объединяет глубокую математическую теорию с творческими возможностями, открывая новые горизонты в создании контента и понимании данных! 🚀✨ 