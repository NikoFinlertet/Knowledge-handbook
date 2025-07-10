# 🧮 Продвинутая теория машинного обучения

## 🔬 Теория обучения (Learning Theory)

### PAC Learning Framework
```
Probably Approximately Correct (PAC) Learning:
├── Concept class C
├── Hypothesis space H  
├── Sample complexity m(ε, δ)
├── VC Dimension
├── Rademacher Complexity
└── Generalization bounds
```

#### Математические основы PAC Learning
```python
import numpy as np
import torch
import matplotlib.pyplot as plt
from scipy import stats

class PACLearningAnalysis:
    """Анализ PAC Learning свойств"""
    
    @staticmethod
    def vc_dimension_linear_classifier(d):
        """
        VC dimension для линейных классификаторов в R^d
        VC(H) = d + 1 для гиперплоскостей в R^d
        """
        return d + 1
    
    @staticmethod
    def sample_complexity_bound(vc_dim, epsilon, delta):
        """
        Sample complexity bound:
        m ≥ (8/ε) * [VC(H) * log(13/ε) + log(4/δ)]
        """
        term1 = vc_dim * np.log(13 / epsilon)
        term2 = np.log(4 / delta)
        sample_complexity = (8 / epsilon) * (term1 + term2)
        
        return int(np.ceil(sample_complexity))
    
    @staticmethod
    def rademacher_complexity_estimate(model, data_loader, num_samples=1000):
        """
        Эмпирическая оценка Rademacher complexity:
        R_m(H) = E_σ[sup_{h∈H} (1/m) Σ σ_i h(x_i)]
        """
        complexities = []
        
        for _ in range(num_samples):
            # Случайные Rademacher переменные
            sigma = torch.randint(0, 2, (len(data_loader.dataset),)) * 2 - 1
            
            batch_complexities = []
            idx = 0
            
            for batch_x, _ in data_loader:
                batch_size = batch_x.size(0)
                batch_sigma = sigma[idx:idx + batch_size].float()
                
                # Предсказания модели
                with torch.no_grad():
                    outputs = model(batch_x)
                    if outputs.dim() > 1:
                        outputs = outputs.mean(dim=1)  # Для многомерного выхода
                
                # Rademacher correlation
                correlation = torch.mean(batch_sigma * outputs)
                batch_complexities.append(correlation.item())
                
                idx += batch_size
            
            complexities.append(np.mean(batch_complexities))
        
        return np.mean(complexities), np.std(complexities)
    
    @staticmethod
    def generalization_bound(train_error, rademacher_complexity, m, delta=0.05):
        """
        Generalization bound через Rademacher complexity:
        R(h) ≤ R̂(h) + 2R_m(H) + sqrt(log(2/δ) / (2m))
        """
        confidence_term = np.sqrt(np.log(2 / delta) / (2 * m))
        bound = train_error + 2 * rademacher_complexity + confidence_term
        
        return bound

class BayesianLearningTheory:
    """Байесовская теория обучения"""
    
    @staticmethod
    def bayesian_posterior(prior_mean, prior_cov, likelihood_precision, data_mean, n_samples):
        """
        Байесовское обновление для гауссовского случая:
        p(θ|D) ∝ p(D|θ) * p(θ)
        """
        # Precision (обратная ковариация)
        prior_precision = torch.inverse(prior_cov)
        
        # Posterior precision
        posterior_precision = prior_precision + n_samples * likelihood_precision * torch.eye(prior_mean.size(0))
        
        # Posterior covariance
        posterior_cov = torch.inverse(posterior_precision)
        
        # Posterior mean
        posterior_mean = posterior_cov @ (prior_precision @ prior_mean + 
                                        n_samples * likelihood_precision * data_mean)
        
        return posterior_mean, posterior_cov
    
    @staticmethod
    def model_evidence(data, prior_params, likelihood_params):
        """
        Model evidence (marginal likelihood):
        p(D|M) = ∫ p(D|θ,M) p(θ|M) dθ
        """
        # Упрощенная версия для гауссовского случая
        log_evidence = 0
        
        # Prior contribution
        log_evidence += stats.multivariate_normal.logpdf(
            prior_params['mean'], 
            np.zeros_like(prior_params['mean']), 
            prior_params['cov']
        )
        
        # Likelihood contribution
        for x in data:
            log_evidence += stats.norm.logpdf(x, 
                                            likelihood_params['mean'], 
                                            likelihood_params['std'])
        
        return log_evidence
    
    @staticmethod
    def bayes_factor(evidence1, evidence2):
        """
        Bayes Factor для сравнения моделей:
        BF = p(D|M1) / p(D|M2)
        """
        return np.exp(evidence1 - evidence2)

# Информационная теория в ML
class InformationTheoreticLearning:
    """Информационно-теоретические аспекты обучения"""
    
    @staticmethod
    def mutual_information_neural_estimation(x, y, critic_network, num_samples=1000):
        """
        MINE: Mutual Information Neural Estimation
        I(X;Y) = sup_T E_P[T(x,y)] - log E_Q[exp(T(x,y))]
        где Q - product marginal
        """
        # Joint samples
        joint_samples = torch.cat([x, y], dim=1)
        
        # Marginal samples (shuffle y)
        y_shuffled = y[torch.randperm(y.size(0))]
        marginal_samples = torch.cat([x, y_shuffled], dim=1)
        
        # Critic scores
        joint_scores = critic_network(joint_samples)
        marginal_scores = critic_network(marginal_samples)
        
        # MINE estimator
        mi_estimate = torch.mean(joint_scores) - torch.log(torch.mean(torch.exp(marginal_scores)))
        
        return mi_estimate
    
    @staticmethod
    def entropy_estimation(samples, k=3):
        """
        Оценка энтропии через k-NN:
        H(X) ≈ ψ(N) - ψ(k) + log(c_d) + (d/N) Σ log(ρ_k(i))
        """
        n, d = samples.shape
        
        # k-NN distances
        from sklearn.neighbors import NearestNeighbors
        nbrs = NearestNeighbors(n_neighbors=k+1).fit(samples)
        distances, _ = nbrs.kneighbors(samples)
        
        # k-th nearest neighbor distances (excluding self)
        knn_distances = distances[:, k]
        
        # Volume constant for d-dimensional unit ball
        c_d = np.pi**(d/2) / np.math.gamma(d/2 + 1)
        
        # Digamma function
        from scipy.special import digamma
        
        # Entropy estimate
        entropy = digamma(n) - digamma(k) + np.log(c_d) + (d/n) * np.sum(np.log(knn_distances + 1e-10))
        
        return entropy
    
    @staticmethod
    def conditional_entropy(x, y, k=3):
        """
        Conditional entropy H(Y|X) через k-NN
        """
        joint_entropy = InformationTheoreticLearning.entropy_estimation(
            np.column_stack([x, y]), k=k
        )
        x_entropy = InformationTheoreticLearning.entropy_estimation(x, k=k)
        
        conditional_entropy = joint_entropy - x_entropy
        return conditional_entropy
    
    @staticmethod
    def information_bottleneck_objective(x, y, z, beta=1.0):
        """
        Information Bottleneck:
        L = I(Z;Y) - β * I(X;Z)
        """
        # Mutual informations (упрощенная оценка)
        i_z_y = np.corrcoef(z.flatten(), y.flatten())[0, 1]**2  # Squared correlation
        i_x_z = np.corrcoef(x.flatten(), z.flatten())[0, 1]**2
        
        objective = i_z_y - beta * i_x_z
        return objective, i_z_y, i_x_z

# Теория оптимизации для ML
class OptimizationTheory:
    """Теоретические аспекты оптимизации в ML"""
    
    @staticmethod
    def convergence_analysis_sgd(gradient_norms, learning_rates, lipschitz_constant):
        """
        Анализ сходимости SGD:
        Для сильно выпуклых функций: E[f(x_t) - f*] ≤ (1-μη)^t * [f(x_0) - f*]
        """
        convergence_rates = []
        
        for t, (grad_norm, lr) in enumerate(zip(gradient_norms, learning_rates)):
            # Условие сходимости: lr < 2/L
            if lr >= 2 / lipschitz_constant:
                print(f"Warning: Learning rate {lr} too large at step {t}")
            
            # Theoretical convergence rate (simplified)
            rate = (1 - lr * 0.1)**t  # Assuming strong convexity parameter μ = 0.1
            convergence_rates.append(rate)
        
        return convergence_rates
    
    @staticmethod
    def adam_convergence_bound(gradients, beta1=0.9, beta2=0.999, epsilon=1e-8):
        """
        Теоретические свойства Adam:
        Convergence bound для non-convex case
        """
        t = len(gradients)
        
        # Bias correction terms
        beta1_t = beta1**t
        beta2_t = beta2**t
        
        # Second moment bound
        v_max = max([torch.norm(g)**2 for g in gradients])
        
        # Theoretical bound (simplified)
        bound = np.sqrt(v_max) / (1 - beta1_t) * np.sqrt(1 - beta2_t)
        
        return bound
    
    @staticmethod
    def escape_from_saddle_points(hessian_eigenvalues, noise_scale):
        """
        Анализ избегания седловых точек:
        Noisy SGD может избежать седловых точек при достаточном шуме
        """
        min_eigenvalue = min(hessian_eigenvalues)
        
        # Критерий избегания седловой точки
        escape_condition = noise_scale > abs(min_eigenvalue)
        
        # Время escape (приблизительное)
        if escape_condition:
            escape_time = 1 / (noise_scale - abs(min_eigenvalue))
        else:
            escape_time = float('inf')
        
        return escape_condition, escape_time

# Curse of Dimensionality
class DimensionalityAnalysis:
    """Анализ проклятия размерности"""
    
    @staticmethod
    def volume_concentration(dimensions, num_samples=10000):
        """
        Концентрация объема в высоких размерностях:
        В d-мерном шаре большая часть объема сконцентрирована у поверхности
        """
        results = {}
        
        for d in dimensions:
            # Генерация точек в единичном шаре
            points = np.random.randn(num_samples, d)
            norms = np.linalg.norm(points, axis=1)
            
            # Нормализация к единичному шару
            points = points / norms[:, np.newaxis]
            
            # Расстояния от центра
            distances = np.linalg.norm(points, axis=1)
            
            # Доля объема в внешней оболочке (r > 0.9)
            outer_shell_fraction = np.mean(distances > 0.9)
            
            results[d] = {
                'mean_distance': np.mean(distances),
                'std_distance': np.std(distances),
                'outer_shell_fraction': outer_shell_fraction
            }
        
        return results
    
    @staticmethod
    def nearest_neighbor_distance_analysis(dimensions, n_points=1000):
        """
        Анализ расстояний до ближайших соседей в высоких размерностях
        """
        results = {}
        
        for d in dimensions:
            # Случайные точки в гиперкубе [0,1]^d
            points = np.random.uniform(0, 1, (n_points, d))
            
            # Вычисление расстояний до ближайших соседей
            from sklearn.neighbors import NearestNeighbors
            nbrs = NearestNeighbors(n_neighbors=2).fit(points)
            distances, _ = nbrs.kneighbors(points)
            
            # Расстояние до ближайшего соседа (исключая саму точку)
            nn_distances = distances[:, 1]
            
            results[d] = {
                'mean_nn_distance': np.mean(nn_distances),
                'std_nn_distance': np.std(nn_distances),
                'min_nn_distance': np.min(nn_distances),
                'max_nn_distance': np.max(nn_distances)
            }
        
        return results
    
    @staticmethod
    def intrinsic_dimensionality_mle(data, k=10):
        """
        Maximum Likelihood оценка intrinsic dimensionality:
        d̂ = (k-1) / Σ log(r_k / r_1)
        """
        from sklearn.neighbors import NearestNeighbors
        
        n, ambient_dim = data.shape
        
        # k nearest neighbors
        nbrs = NearestNeighbors(n_neighbors=k+1).fit(data)
        distances, _ = nbrs.kneighbors(data)
        
        # Исключаем саму точку (distance = 0)
        distances = distances[:, 1:]
        
        # MLE estimator
        ratios = distances[:, -1] / distances[:, 0]  # r_k / r_1
        log_ratios = np.log(ratios + 1e-10)
        
        intrinsic_dim = (k - 1) / np.mean(log_ratios)
        
        return intrinsic_dim

# Теория глубоких сетей
class DeepLearningTheory:
    """Теоретические аспекты глубокого обучения"""
    
    @staticmethod
    def neural_tangent_kernel(model, x1, x2):
        """
        Neural Tangent Kernel (NTK):
        K(x1, x2) = E[∂f(x1)/∂θ · ∂f(x2)/∂θ]
        """
        # Вычисление градиентов по параметрам
        def compute_gradients(x):
            output = model(x.unsqueeze(0))
            gradients = torch.autograd.grad(output.sum(), model.parameters(), 
                                          create_graph=True, retain_graph=True)
            return torch.cat([g.flatten() for g in gradients])
        
        grad1 = compute_gradients(x1)
        grad2 = compute_gradients(x2)
        
        # Скалярное произведение градиентов
        ntk_value = torch.dot(grad1, grad2)
        
        return ntk_value.item()
    
    @staticmethod
    def effective_dimension(model, data_loader, threshold=0.99):
        """
        Effective dimension нейронной сети:
        Количество собственных значений NTK, объясняющих threshold дисперсии
        """
        ntk_matrix = []
        data_points = []
        
        # Собираем данные
        for batch_x, _ in data_loader:
            for x in batch_x[:10]:  # Ограничиваем для вычислительной эффективности
                data_points.append(x)
            if len(data_points) >= 100:
                break
        
        # Вычисляем NTK матрицу
        n = len(data_points)
        ntk_matrix = torch.zeros(n, n)
        
        for i in range(n):
            for j in range(i, n):
                ntk_val = DeepLearningTheory.neural_tangent_kernel(
                    model, data_points[i], data_points[j]
                )
                ntk_matrix[i, j] = ntk_val
                ntk_matrix[j, i] = ntk_val
        
        # Собственные значения
        eigenvalues, _ = torch.linalg.eigh(ntk_matrix)
        eigenvalues = torch.sort(eigenvalues, descending=True)[0]
        
        # Эффективная размерность
        total_variance = eigenvalues.sum()
        cumulative_variance = torch.cumsum(eigenvalues, dim=0)
        effective_dim = torch.argmax((cumulative_variance / total_variance >= threshold).float()) + 1
        
        return effective_dim.item(), eigenvalues
    
    @staticmethod
    def lottery_ticket_analysis(model, sparsity_levels):
        """
        Анализ Lottery Ticket Hypothesis:
        Поиск подсетей, которые можно обучить изолированно
        """
        original_params = {}
        for name, param in model.named_parameters():
            original_params[name] = param.clone()
        
        results = {}
        
        for sparsity in sparsity_levels:
            # Создание маски
            mask = {}
            for name, param in model.named_parameters():
                # Magnitude-based pruning
                flat_param = param.abs().flatten()
                threshold = torch.quantile(flat_param, sparsity)
                mask[name] = (param.abs() > threshold).float()
            
            # Применение маски
            masked_params = 0
            total_params = 0
            
            for name, param in model.named_parameters():
                param.data *= mask[name]
                masked_params += mask[name].sum().item()
                total_params += param.numel()
            
            actual_sparsity = 1 - (masked_params / total_params)
            
            results[sparsity] = {
                'actual_sparsity': actual_sparsity,
                'remaining_params': masked_params,
                'mask': mask
            }
        
        # Восстановление оригинальных параметров
        for name, param in model.named_parameters():
            param.data = original_params[name]
        
        return results
    
    @staticmethod
    def generalization_gap_analysis(train_losses, val_losses, model_complexity):
        """
        Анализ generalization gap:
        Gap = Training Error - Validation Error
        """
        gap = np.array(val_losses) - np.array(train_losses)
        
        # Корреляция с complexity
        correlation = np.corrcoef(gap, model_complexity)[0, 1]
        
        # Тренд over time
        trend_coef = np.polyfit(range(len(gap)), gap, 1)[0]
        
        return {
            'mean_gap': np.mean(gap),
            'std_gap': np.std(gap),
            'max_gap': np.max(gap),
            'complexity_correlation': correlation,
            'trend_coefficient': trend_coef
        }

# Робастность и adversarial примеры
class RobustnessTheory:
    """Теория робастности моделей"""
    
    @staticmethod
    def lipschitz_constant_estimation(model, input_dim, num_samples=1000):
        """
        Оценка константы Липшица:
        L = max |f(x1) - f(x2)| / |x1 - x2|
        """
        lipschitz_estimates = []
        
        for _ in range(num_samples):
            # Случайные пары точек
            x1 = torch.randn(1, input_dim)
            x2 = torch.randn(1, input_dim)
            
            with torch.no_grad():
                y1 = model(x1)
                y2 = model(x2)
            
            # Расстояния
            input_dist = torch.norm(x1 - x2)
            output_dist = torch.norm(y1 - y2)
            
            if input_dist > 1e-6:  # Избегаем деления на 0
                lipschitz_est = (output_dist / input_dist).item()
                lipschitz_estimates.append(lipschitz_est)
        
        return max(lipschitz_estimates), np.mean(lipschitz_estimates)
    
    @staticmethod
    def adversarial_robustness_certificate(model, x, epsilon, norm='l2'):
        """
        Сертификат adversarial robustness:
        Гарантированная устойчивость в ε-окрестности
        """
        x.requires_grad_(True)
        output = model(x)
        
        # Градиент по входу
        grad = torch.autograd.grad(output.sum(), x, create_graph=True)[0]
        
        if norm == 'l2':
            # L2 norm robustness bound
            grad_norm = torch.norm(grad, p=2)
            certificate = epsilon * grad_norm
        elif norm == 'linf':
            # L∞ norm robustness bound
            grad_norm = torch.norm(grad, p=float('inf'))
            certificate = epsilon * grad_norm
        else:
            raise ValueError(f"Unsupported norm: {norm}")
        
        return certificate.item(), grad_norm.item()
    
    @staticmethod
    def certified_defense_analysis(model, test_loader, epsilon=0.1):
        """
        Анализ certified defense методов
        """
        certificates = []
        predictions = []
        
        for batch_x, batch_y in test_loader:
            for x, y in zip(batch_x, batch_y):
                cert, _ = RobustnessTheory.adversarial_robustness_certificate(
                    model, x.unsqueeze(0), epsilon
                )
                
                with torch.no_grad():
                    pred = model(x.unsqueeze(0)).argmax(dim=1)
                
                certificates.append(cert)
                predictions.append((pred == y).item())
        
        # Certified accuracy
        certified_correct = sum(p and c < 1.0 for p, c in zip(predictions, certificates))
        certified_accuracy = certified_correct / len(predictions)
        
        return {
            'certified_accuracy': certified_accuracy,
            'mean_certificate': np.mean(certificates),
            'certificate_distribution': certificates
        }
```

---

## 📊 Статистическая теория обучения

### Concentration Inequalities
```python
class ConcentrationInequalities:
    """Неравенства концентрации для анализа обучения"""
    
    @staticmethod
    def hoeffding_bound(n, a, b, delta):
        """
        Неравенство Хёффдинга:
        P(|X̄ - E[X]| ≥ t) ≤ 2exp(-2nt²/(b-a)²)
        """
        # Bound на отклонение для заданной probability
        t = np.sqrt(-np.log(delta/2) * (b - a)**2 / (2 * n))
        return t
    
    @staticmethod
    def mcdiarmid_bound(n, c_values, delta):
        """
        Неравенство МакДиармида:
        P(|f(X) - E[f(X)]| ≥ t) ≤ 2exp(-2t²/Σc_i²)
        """
        sum_c_squared = sum(c**2 for c in c_values)
        t = np.sqrt(-np.log(delta/2) * sum_c_squared / 2)
        return t
    
    @staticmethod
    def bennett_inequality(variance, bound, n, delta):
        """
        Неравенство Беннетта (для ограниченных случайных величин):
        Более точное чем Хёффдинга при малой дисперсии
        """
        def h(u):
            return (1 + u) * np.log(1 + u) - u
        
        # Решаем уравнение для t
        from scipy.optimize import fsolve
        
        def equation(t):
            return variance * h(bound * t / variance) - t**2 / n + np.log(delta / 2)
        
        try:
            t = fsolve(equation, 0.1)[0]
            return max(0, t)
        except:
            return float('inf')

# Множественное тестирование
class MultipleTestingCorrection:
    """Коррекция множественного тестирования в ML"""
    
    @staticmethod
    def bonferroni_correction(p_values, alpha=0.05):
        """
        Поправка Бонферрони:
        Adjusted α = α / m
        """
        m = len(p_values)
        adjusted_alpha = alpha / m
        significant = [p <= adjusted_alpha for p in p_values]
        
        return significant, adjusted_alpha
    
    @staticmethod
    def benjamini_hochberg_fdr(p_values, alpha=0.05):
        """
        Контроль False Discovery Rate (Benjamini-Hochberg):
        Находим наибольший k: P(k) ≤ (k/m) * α
        """
        m = len(p_values)
        sorted_indices = np.argsort(p_values)
        sorted_p_values = np.array(p_values)[sorted_indices]
        
        # BH критерий
        bh_thresholds = [(i + 1) / m * alpha for i in range(m)]
        
        # Находим наибольший индекс удовлетворяющий критерию
        significant_idx = -1
        for i in range(m - 1, -1, -1):
            if sorted_p_values[i] <= bh_thresholds[i]:
                significant_idx = i
                break
        
        # Отмечаем значимые гипотезы
        significant = [False] * m
        if significant_idx >= 0:
            for i in range(significant_idx + 1):
                original_idx = sorted_indices[i]
                significant[original_idx] = True
        
        return significant, significant_idx
    
    @staticmethod
    def storey_fdr_estimation(p_values, lambda_threshold=0.5):
        """
        Оценка Storey для истинного FDR:
        π̂₀ = #{p_i > λ} / [(1-λ)m]
        """
        m = len(p_values)
        num_above_threshold = sum(1 for p in p_values if p > lambda_threshold)
        
        pi_0_estimate = num_above_threshold / ((1 - lambda_threshold) * m)
        pi_0_estimate = min(pi_0_estimate, 1.0)  # Ограничиваем сверху
        
        return pi_0_estimate
```

---

## 🎯 Каузальность в машинном обучении

### Causal Inference
```python
class CausalInference:
    """Каузальный вывод в машинном обучении"""
    
    @staticmethod
    def average_treatment_effect(y, treatment, x_confounders):
        """
        Average Treatment Effect (ATE):
        ATE = E[Y(1) - Y(0)]
        """
        from sklearn.linear_model import LinearRegression
        
        # Propensity score (вероятность получить treatment)
        propensity_model = LinearRegression()
        propensity_model.fit(x_confounders, treatment)
        propensity_scores = propensity_model.predict_proba(x_confounders)[:, 1]
        
        # Inverse Propensity Weighting (IPW)
        treated_mask = treatment == 1
        control_mask = treatment == 0
        
        # ATE через IPW
        ate_treated = np.mean(y[treated_mask] / propensity_scores[treated_mask])
        ate_control = np.mean(y[control_mask] / (1 - propensity_scores[control_mask]))
        
        ate = ate_treated - ate_control
        
        return ate, propensity_scores
    
    @staticmethod
    def backdoor_adjustment(y, treatment, confounders, confounder_values):
        """
        Backdoor adjustment для каузального эффекта:
        P(Y|do(T=t)) = Σ_c P(Y|T=t,C=c) P(C=c)
        """
        from sklearn.preprocessing import LabelEncoder
        from collections import Counter
        
        # Кодирование категориальных переменных
        unique_confounders = np.unique(confounders, axis=0)
        confounder_probs = {}
        
        # Вычисление P(C=c)
        for conf in unique_confounders:
            count = np.sum(np.all(confounders == conf, axis=1))
            prob = count / len(confounders)
            confounder_probs[tuple(conf)] = prob
        
        # Adjustment formula
        adjusted_effect = 0
        
        for conf_val, prob in confounder_probs.items():
            # P(Y|T=1,C=c)
            mask_treated = (treatment == 1) & np.all(confounders == conf_val, axis=1)
            if np.any(mask_treated):
                y_treated_given_c = np.mean(y[mask_treated])
            else:
                y_treated_given_c = 0
            
            # P(Y|T=0,C=c)
            mask_control = (treatment == 0) & np.all(confounders == conf_val, axis=1)
            if np.any(mask_control):
                y_control_given_c = np.mean(y[mask_control])
            else:
                y_control_given_c = 0
            
            # Weighted contribution
            adjusted_effect += (y_treated_given_c - y_control_given_c) * prob
        
        return adjusted_effect
    
    @staticmethod
    def instrumental_variable_analysis(y, treatment, instrument, controls=None):
        """
        Instrumental Variable (IV) анализ:
        Использование инструментальной переменной для каузального вывода
        """
        from sklearn.linear_model import LinearRegression
        
        # Two-Stage Least Squares (2SLS)
        
        # First stage: Treatment ~ Instrument + Controls
        if controls is not None:
            first_stage_x = np.column_stack([instrument.reshape(-1, 1), controls])
        else:
            first_stage_x = instrument.reshape(-1, 1)
        
        first_stage = LinearRegression()
        first_stage.fit(first_stage_x, treatment)
        treatment_predicted = first_stage.predict(first_stage_x)
        
        # Second stage: Y ~ Predicted_Treatment + Controls
        if controls is not None:
            second_stage_x = np.column_stack([treatment_predicted.reshape(-1, 1), controls])
        else:
            second_stage_x = treatment_predicted.reshape(-1, 1)
        
        second_stage = LinearRegression()
        second_stage.fit(second_stage_x, y)
        
        # IV estimate (коэффициент при treatment)
        iv_estimate = second_stage.coef_[0]
        
        # First stage F-statistic (проверка силы инструмента)
        residuals_first = treatment - treatment_predicted
        mse_first = np.mean(residuals_first**2)
        
        # Simplified F-statistic
        f_stat = np.var(treatment_predicted) / mse_first if mse_first > 0 else 0
        
        return iv_estimate, f_stat, first_stage, second_stage

class CausalGraphs:
    """Работа с каузальными графами"""
    
    @staticmethod
    def d_separation_test(graph, x, y, conditioning_set):
        """
        Тест d-separation в каузальном графе:
        Определяет условную независимость переменных
        """
        # Упрощенная реализация для демонстрации
        # В реальности используйте специализированные библиотеки (causalgraphicalmodels, networkx)
        
        def has_path_blocked(path, conditioning_set):
            """Проверяет заблокирован ли путь conditioning set"""
            for i in range(1, len(path) - 1):
                node = path[i]
                prev_node = path[i-1]
                next_node = path[i+1]
                
                # Collider pattern: prev -> node <- next
                if graph.get(prev_node, {}).get(node) and graph.get(next_node, {}).get(node):
                    if node not in conditioning_set:
                        return True  # Blocked by collider
                
                # Chain/fork: prev -> node -> next or prev <- node -> next
                else:
                    if node in conditioning_set:
                        return True  # Blocked by conditioning
            
            return False
        
        # Найти все пути между x и y (упрощенно)
        def find_paths(start, end, path=[]):
            path = path + [start]
            if start == end:
                return [path]
            
            paths = []
            if start in graph:
                for node in graph[start]:
                    if node not in path:  # Избегаем циклов
                        newpaths = find_paths(node, end, path)
                        paths.extend(newpaths)
            return paths
        
        paths = find_paths(x, y)
        
        # Проверяем все пути
        all_paths_blocked = True
        for path in paths:
            if not has_path_blocked(path, conditioning_set):
                all_paths_blocked = False
                break
        
        return all_paths_blocked  # True если x ⊥ y | conditioning_set
    
    @staticmethod
    def identify_confounders(graph, treatment, outcome):
        """
        Идентификация конфаундеров по каузальному графу
        """
        confounders = set()
        
        # Конфаундер - переменная, влияющая на treatment и outcome
        for node in graph:
            influences_treatment = treatment in graph.get(node, {})
            influences_outcome = outcome in graph.get(node, {})
            
            if influences_treatment and influences_outcome:
                confounders.add(node)
        
        return list(confounders)
```

---

## 📚 Связанные материалы

### Математические основы
- [[machine-learning-theory|Основы теории ML]] - базовые концепции
- [[deep-learning-foundations|Глубокое обучение]] - нейронные сети
- [[neural-networks-architecture|Архитектуры]] - практические реализации

### Специализированные области
- [[generative-ai-theory|Генеративные модели]] - теория генерации
- [[ai-ethics-safety|Этика и безопасность]] - робастность моделей
- [[prompt-engineering|Prompt Engineering]] - каузальные рассуждения

---

🎓 **Ключевая мысль:** Продвинутая теория ML предоставляет математические инструменты для понимания фундаментальных ограничений и возможностей алгоритмов обучения! 🔬✨ 