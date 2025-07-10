# Теория чисел и криптография 🔐

> **Навигация**: [[calculus-optimization|← Математический анализ]] | [[mathematics-algorithms|Главная]] | [[statistics-data-analysis|Статистика →]]

## Содержание
1. [Модульная арифметика](#🔐-модульная-арифметика)
2. [Алгоритмы факторизации](#🔢-алгоритмы-факторизации)
3. [Криптографические протоколы](#🛡️-криптографические-протоколы)
4. [Практические применения](#🚀-практические-применения)

---

## 🔐 Модульная арифметика

**Математическая основа:** Сравнения по модулю, китайская теорема об остатках, функция Эйлера

```python
import math
import random

class NumberTheoryAlgorithms:
    """Алгоритмы теории чисел"""
    
    @staticmethod
    def gcd_extended(a, b):
        """
        Расширенный алгоритм Евклида
        Находит НОД(a,b) и коэффициенты x,y такие что ax + by = НОД(a,b)
        """
        if a == 0:
            return b, 0, 1
        
        gcd, x1, y1 = NumberTheoryAlgorithms.gcd_extended(b % a, a)
        x = y1 - (b // a) * x1
        y = x1
        
        return gcd, x, y
    
    @staticmethod
    def mod_inverse(a, m):
        """
        Модульное обращение: найти x такое что ax ≡ 1 (mod m)
        Существует только если НОД(a,m) = 1
        """
        gcd, x, _ = NumberTheoryAlgorithms.gcd_extended(a, m)
        
        if gcd != 1:
            raise ValueError(f"Модульное обращение не существует для {a} mod {m}")
        
        return (x % m + m) % m
    
    @staticmethod
    def fast_power_mod(base, exp, mod):
        """
        Быстрое возведение в степень по модулю
        base^exp mod mod за O(log exp)
        """
        result = 1
        base = base % mod
        
        while exp > 0:
            if exp % 2 == 1:
                result = (result * base) % mod
            
            exp = exp >> 1
            base = (base * base) % mod
        
        return result
    
    @staticmethod
    def chinese_remainder_theorem(remainders, moduli):
        """
        Китайская теорема об остатках
        Решает систему сравнений x ≡ r_i (mod m_i)
        """
        # Проверяем взаимную простоту модулей
        for i in range(len(moduli)):
            for j in range(i + 1, len(moduli)):
                if math.gcd(moduli[i], moduli[j]) != 1:
                    raise ValueError("Модули должны быть попарно взаимно простыми")
        
        total_mod = 1
        for mod in moduli:
            total_mod *= mod
        
        result = 0
        for i in range(len(remainders)):
            Mi = total_mod // moduli[i]
            yi = NumberTheoryAlgorithms.mod_inverse(Mi, moduli[i])
            result += remainders[i] * Mi * yi
        
        return result % total_mod
    
    @staticmethod
    def euler_totient(n):
        """
        Функция Эйлера φ(n) - количество чисел от 1 до n, взаимно простых с n
        """
        result = n
        p = 2
        
        while p * p <= n:
            if n % p == 0:
                while n % p == 0:
                    n //= p
                result -= result // p
            p += 1
        
        if n > 1:
            result -= result // n
        
        return result
```

---

## 🔢 Алгоритмы факторизации

```python
class FactorizationAlgorithms:
    """Алгоритмы факторизации чисел"""
    
    @staticmethod
    def trial_division(n):
        """Факторизация методом пробных делений"""
        factors = []
        d = 2
        
        while d * d <= n:
            while n % d == 0:
                factors.append(d)
                n //= d
            d += 1
        
        if n > 1:
            factors.append(n)
        
        return factors
    
    @staticmethod
    def pollard_rho(n, max_iterations=10000):
        """
        Алгоритм Полларда ρ для факторизации
        Основан на парадоксе дней рождения
        """
        if n % 2 == 0:
            return 2
        
        x = random.randint(2, n - 1)
        y = x
        c = random.randint(1, n - 1)
        d = 1
        
        def f(x):
            return (x * x + c) % n
        
        while d == 1:
            x = f(x)
            y = f(f(y))
            d = math.gcd(abs(x - y), n)
            
            max_iterations -= 1
            if max_iterations <= 0:
                return None
        
        return d if d != n else None
    
    @staticmethod
    def fermat_factorization(n):
        """
        Метод факторизации Ферма
        Представляет n = a² - b² = (a+b)(a-b)
        """
        if n % 2 == 0:
            return 2, n // 2
        
        a = math.ceil(math.sqrt(n))
        b_squared = a * a - n
        
        while b_squared < 0 or math.sqrt(b_squared) != int(math.sqrt(b_squared)):
            a += 1
            b_squared = a * a - n
        
        b = int(math.sqrt(b_squared))
        return a - b, a + b
```

---

## 🛡️ Криптографические протоколы

### RSA шифрование

```python
class RSACryptosystem:
    """Реализация криптосистемы RSA"""
    
    def __init__(self, key_size=1024):
        self.key_size = key_size
        self.public_key = None
        self.private_key = None
    
    def _is_prime(self, n, k=10):
        """Тест простоты Миллера-Рабина"""
        if n < 2:
            return False
        if n == 2 or n == 3:
            return True
        if n % 2 == 0:
            return False
        
        # Представляем n-1 в виде d * 2^r
        r = 0
        d = n - 1
        while d % 2 == 0:
            r += 1
            d //= 2
        
        for _ in range(k):
            a = random.randint(2, n - 2)
            x = pow(a, d, n)
            
            if x == 1 or x == n - 1:
                continue
            
            for _ in range(r - 1):
                x = pow(x, 2, n)
                if x == n - 1:
                    break
            else:
                return False
        
        return True
    
    def _generate_prime(self, bits):
        """Генерация случайного простого числа заданной битности"""
        while True:
            candidate = random.getrandbits(bits)
            candidate |= (1 << bits - 1) | 1  # Устанавливаем старший и младший биты
            
            if self._is_prime(candidate):
                return candidate
    
    def generate_keypair(self):
        """Генерация пары ключей RSA"""
        # Генерируем два больших простых числа
        p = self._generate_prime(self.key_size // 2)
        q = self._generate_prime(self.key_size // 2)
        
        # Вычисляем n = p * q
        n = p * q
        
        # Вычисляем функцию Эйлера φ(n) = (p-1)(q-1)
        phi_n = (p - 1) * (q - 1)
        
        # Выбираем открытую экспоненту e (обычно 65537)
        e = 65537
        
        # Убеждаемся что НОД(e, φ(n)) = 1
        if math.gcd(e, phi_n) != 1:
            e = 3
            while math.gcd(e, phi_n) != 1:
                e += 2
        
        # Вычисляем закрытую экспоненту d
        d = NumberTheoryAlgorithms.mod_inverse(e, phi_n)
        
        self.public_key = (n, e)
        self.private_key = (n, d)
        
        return self.public_key, self.private_key
    
    def encrypt(self, message, public_key):
        """Шифрование сообщения"""
        n, e = public_key
        
        if isinstance(message, str):
            # Преобразуем строку в число
            message_int = int.from_bytes(message.encode(), 'big')
        else:
            message_int = message
        
        if message_int >= n:
            raise ValueError("Сообщение слишком длинное для данного размера ключа")
        
        ciphertext = NumberTheoryAlgorithms.fast_power_mod(message_int, e, n)
        return ciphertext
    
    def decrypt(self, ciphertext, private_key):
        """Расшифровка сообщения"""
        n, d = private_key
        
        plaintext_int = NumberTheoryAlgorithms.fast_power_mod(ciphertext, d, n)
        
        # Преобразуем число обратно в строку
        try:
            byte_length = (plaintext_int.bit_length() + 7) // 8
            plaintext = plaintext_int.to_bytes(byte_length, 'big').decode()
            return plaintext
        except:
            return plaintext_int
    
    def sign(self, message, private_key):
        """Цифровая подпись"""
        n, d = private_key
        
        if isinstance(message, str):
            message_hash = hash(message) % n
        else:
            message_hash = message % n
        
        signature = NumberTheoryAlgorithms.fast_power_mod(message_hash, d, n)
        return signature
    
    def verify_signature(self, message, signature, public_key):
        """Проверка цифровой подписи"""
        n, e = public_key
        
        if isinstance(message, str):
            message_hash = hash(message) % n
        else:
            message_hash = message % n
        
        verified_hash = NumberTheoryAlgorithms.fast_power_mod(signature, e, n)
        return message_hash == verified_hash

# Diffie-Hellman обмен ключами
class DiffieHellmanKeyExchange:
    """Протокол обмена ключами Диффи-Хеллмана"""
    
    def __init__(self, p=None, g=None):
        if p is None:
            # Используем стандартное простое число
            self.p = 2147483647  # Простое число Мерсенна 2^31 - 1
        else:
            self.p = p
        
        if g is None:
            self.g = 2  # Примитивный корень
        else:
            self.g = g
    
    def generate_private_key(self):
        """Генерация приватного ключа"""
        return random.randint(1, self.p - 1)
    
    def generate_public_key(self, private_key):
        """Генерация публичного ключа"""
        return NumberTheoryAlgorithms.fast_power_mod(self.g, private_key, self.p)
    
    def generate_shared_secret(self, private_key, other_public_key):
        """Генерация общего секретного ключа"""
        return NumberTheoryAlgorithms.fast_power_mod(other_public_key, private_key, self.p)
```

---

## 🚀 Практические применения

### Хеширование с криптографической стойкостью

```python
class CryptographicHashFunction:
    """Упрощенная криптографическая хеш-функция"""
    
    def __init__(self, modulus=2**61 - 1):
        self.modulus = modulus
        self.multiplier = random.randint(2, modulus - 1)
    
    def hash(self, data):
        """Вычисление хеша"""
        if isinstance(data, str):
            data = data.encode()
        
        hash_value = 0
        for byte in data:
            hash_value = (hash_value * self.multiplier + byte) % self.modulus
        
        return hash_value
    
    def merkle_tree_root(self, data_blocks):
        """Построение дерева Меркла и вычисление корневого хеша"""
        if not data_blocks:
            return None
        
        # Хешируем все блоки данных
        hashes = [self.hash(block) for block in data_blocks]
        
        # Строим дерево снизу вверх
        while len(hashes) > 1:
            next_level = []
            
            for i in range(0, len(hashes), 2):
                if i + 1 < len(hashes):
                    # Хешируем пару хешей
                    combined = str(hashes[i]) + str(hashes[i + 1])
                    next_level.append(self.hash(combined))
                else:
                    # Нечетное количество - дублируем последний
                    combined = str(hashes[i]) + str(hashes[i])
                    next_level.append(self.hash(combined))
            
            hashes = next_level
        
        return hashes[0]

# Демонстрация использования
def demo_cryptography():
    """Демонстрация криптографических алгоритмов"""
    
    # RSA шифрование
    rsa = RSACryptosystem(key_size=512)  # Маленький размер для демонстрации
    public_key, private_key = rsa.generate_keypair()
    
    message = "Секретное сообщение"
    print(f"Исходное сообщение: {message}")
    
    # Шифрование
    ciphertext = rsa.encrypt(message, public_key)
    print(f"Зашифрованное: {ciphertext}")
    
    # Расшифровка
    decrypted = rsa.decrypt(ciphertext, private_key)
    print(f"Расшифрованное: {decrypted}")
    
    # Цифровая подпись
    signature = rsa.sign(message, private_key)
    is_valid = rsa.verify_signature(message, signature, public_key)
    print(f"Подпись действительна: {is_valid}")
    
    # Обмен ключами Диффи-Хеллмана
    dh = DiffieHellmanKeyExchange()
    
    # Алиса генерирует ключи
    alice_private = dh.generate_private_key()
    alice_public = dh.generate_public_key(alice_private)
    
    # Боб генерирует ключи
    bob_private = dh.generate_private_key()
    bob_public = dh.generate_public_key(bob_private)
    
    # Генерация общего секрета
    alice_shared_secret = dh.generate_shared_secret(alice_private, bob_public)
    bob_shared_secret = dh.generate_shared_secret(bob_private, alice_public)
    
    print(f"Общие секреты совпадают: {alice_shared_secret == bob_shared_secret}")
```

**Когда использовать методы теории чисел:**

- ✅ **RSA**: Шифрование данных, цифровые подписи
- ✅ **Диффи-Хеллман**: Безопасный обмен ключами
- ✅ **Модульная арифметика**: Хеширование, контрольные суммы
- ✅ **Факторизация**: Криптоанализ, проверка безопасности

---

## 🔗 Связанные темы

- [[probability-randomized-algorithms|Вероятностные алгоритмы]] - тесты простоты
- [[discrete-mathematics-algorithms|Дискретная математика]] - теория групп и полей  
- [[calculus-optimization|Математический анализ]] - оптимизация криптографических параметров
- [[practical-applications|Практические применения]] - безопасность систем

## 📚 Дополнительные ресурсы

- **Книги**: "Introduction to Modern Cryptography" - Katz & Lindell
- **Курсы**: Stanford CS255, MIT 6.857
- **Практика**: Cryptopals challenges, CryptoHack 