# Криптография и информационная безопасность

## 🔐 Введение в криптографию

### Что такое криптография?
- **Определение** - наука о защите информации с помощью математических методов
- **Цели** - конфиденциальность, целостность, аутентичность, неотказуемость
- **Применение** - защита данных, цифровые подписи, аутентификация
- **Важность** - основа современной информационной безопасности

### Основные принципы
```
Принципы Кёрхгоффа:
├── Безопасность системы не должна зависеть от секретности алгоритма
├── Безопасность должна основываться только на секретности ключа
├── Алгоритм должен быть публично известен и проверен
├── Ключ должен быть легко изменяемым
└── Система должна быть практичной в использовании
```

## 🗝️ Основные понятия

### Классическая криптография
```
Базовые концепции:
├── Открытый текст (Plaintext)
├── Шифртекст (Ciphertext)
├── Ключ (Key)
├── Алгоритм шифрования (Encryption Algorithm)
├── Алгоритм дешифрования (Decryption Algorithm)
├── Криптоанализ (Cryptanalysis)
└── Криптостойкость (Cryptographic Strength)
```

### Типы криптографических атак
```
Модели атак:
├── Атака только на шифртекст
├── Атака с известным открытым текстом
├── Атака с выбранным открытым текстом
├── Атака с выбранным шифртекстом
├── Атака с адаптивно выбранным шифртекстом
├── Атака по времени
├── Атака по энергопотреблению
└── Атака по побочным каналам
```

## 🔑 Симметричная криптография

### Блочные шифры
```
Алгоритмы блочного шифрования:
├── DES (Data Encryption Standard)
├── 3DES (Triple DES)
├── AES (Advanced Encryption Standard)
├── Blowfish
├── Twofish
├── RC5/RC6
├── IDEA
└── Camellia
```

### Потоковые шифры
```
Алгоритмы потокового шифрования:
├── RC4
├── ChaCha20
├── Salsa20
├── A5/1, A5/2
├── E0 (Bluetooth)
├── Trivium
└── Grain
```

### Режимы работы блочных шифров
```
Режимы шифрования:
├── ECB (Electronic Codebook)
├── CBC (Cipher Block Chaining)
├── CFB (Cipher Feedback)
├── OFB (Output Feedback)
├── CTR (Counter)
├── GCM (Galois/Counter Mode)
├── CCM (Counter with CBC-MAC)
└── XTS (XEX-based tweaked-codebook)
```

## 🔐 Асимметричная криптография

### Алгоритмы с открытым ключом
```
Основные алгоритмы:
├── RSA (Rivest-Shamir-Adleman)
├── DSA (Digital Signature Algorithm)
├── DH (Diffie-Hellman)
├── ECDH (Elliptic Curve Diffie-Hellman)
├── ECDSA (Elliptic Curve DSA)
├── ElGamal
├── Paillier
└── Lattice-based (post-quantum)
```

### Эллиптические кривые
```
Криптография на эллиптических кривых:
├── Математические основы
├── Преимущества над RSA
├── Стандартные кривые (P-256, P-384, P-521)
├── Curve25519
├── Ed25519
├── Безопасность кривых
└── Реализация и производительность
```

### Математические основы
```
Теоретические основы:
├── Теория чисел
├── Модулярная арифметика
├── Дискретный логарифм
├── Факторизация больших чисел
├── Китайская теорема об остатках
├── Квадратичные вычеты
└── Эллиптические кривые
```

## 🏗️ Криптографические протоколы

### Протоколы согласования ключей
```
Key Agreement Protocols:
├── Diffie-Hellman
├── ECDH
├── Station-to-Station
├── MQV (Menezes-Qu-Vanstone)
├── ECMQV
├── SPEKE
└── J-PAKE
```

### Протоколы аутентификации
```
Authentication Protocols:
├── Challenge-Response
├── Kerberos
├── SAML
├── OAuth 2.0
├── OpenID Connect
├── RADIUS
├── LDAP
└── Multi-factor Authentication
```

### Протоколы безопасной связи
```
Secure Communication Protocols:
├── TLS/SSL (Transport Layer Security)
├── IPSec
├── SSH (Secure Shell)
├── S/MIME
├── PGP/GPG
├── Signal Protocol
├── OTR (Off-the-Record)
└── ZRTP
```

## 🛡️ Хеширование и цифровые подписи

### Криптографические хеш-функции
```
Hash Functions:
├── MD5 (устарел)
├── SHA-1 (устарел)
├── SHA-2 (SHA-256, SHA-384, SHA-512)
├── SHA-3 (Keccak)
├── BLAKE2
├── BLAKE3
├── Whirlpool
└── RIPEMD-160
```

### Цифровые подписи
```
Digital Signature Algorithms:
├── RSA-PSS
├── DSA
├── ECDSA
├── EdDSA
├── Schnorr signatures
├── BLS signatures
├── Ring signatures
└── Blind signatures
```

### Коды аутентификации сообщений
```
Message Authentication Codes:
├── HMAC (Hash-based MAC)
├── CMAC (Cipher-based MAC)
├── GMAC (Galois MAC)
├── Poly1305
├── SipHash
├── BLAKE2b MAC
└── KMAC
```

## 🔒 Управление ключами

### Жизненный цикл ключей
```
Key Lifecycle:
├── Генерация ключей
├── Распределение ключей
├── Хранение ключей
├── Использование ключей
├── Архивирование ключей
├── Восстановление ключей
├── Обновление ключей
└── Уничтожение ключей
```

### Инфраструктура открытых ключей (PKI)
```
PKI Components:
├── Certificate Authority (CA)
├── Registration Authority (RA)
├── Certificate Repository
├── Certificate Revocation List (CRL)
├── Online Certificate Status Protocol (OCSP)
├── Hardware Security Modules (HSM)
├── Key Escrow
└── Time Stamping Authority (TSA)
```

### Стандарты и форматы
```
Standards and Formats:
├── X.509 certificates
├── PKCS (Public Key Cryptography Standards)
├── ASN.1 (Abstract Syntax Notation)
├── PEM (Privacy-Enhanced Mail)
├── DER (Distinguished Encoding Rules)
├── JWK (JSON Web Key)
├── JWS (JSON Web Signature)
└── JWE (JSON Web Encryption)
```

## 🌐 Безопасность сетей

### Сетевые угрозы
```
Network Threats:
├── Перехват трафика (Sniffing)
├── Подмена адресов (Spoofing)
├── Атака "человек посередине" (MITM)
├── DNS-отравление
├── DDoS атаки
├── Сканирование портов
├── SQL-инъекции
└── Cross-Site Scripting (XSS)
```

### Защита сетевого трафика
```
Network Security:
├── Firewalls
├── Intrusion Detection Systems (IDS)
├── Intrusion Prevention Systems (IPS)
├── Virtual Private Networks (VPN)
├── Network Access Control (NAC)
├── Deep Packet Inspection (DPI)
├── Security Information and Event Management (SIEM)
└── Zero Trust Architecture
```

### Веб-безопасность
```
Web Security:
├── HTTPS/TLS
├── HTTP Security Headers
├── Cross-Origin Resource Sharing (CORS)
├── Content Security Policy (CSP)
├── Web Application Firewalls (WAF)
├── Rate Limiting
├── Input Validation
└── Secure Coding Practices
```

## 💾 Безопасность данных

### Защита данных в покое
```
Data at Rest Protection:
├── Full Disk Encryption (FDE)
├── File-level encryption
├── Database encryption
├── Backup encryption
├── Cloud storage encryption
├── Key management
├── Access controls
└── Data masking
```

### Защита данных в движении
```
Data in Transit Protection:
├── TLS/SSL
├── IPSec
├── SSH tunneling
├── VPN
├── End-to-end encryption
├── Certificate pinning
├── Perfect Forward Secrecy
└── Secure protocols
```

### Классификация и маркировка данных
```
Data Classification:
├── Публичные данные
├── Внутренние данные
├── Конфиденциальные данные
├── Строго конфиденциальные данные
├── Персональные данные
├── Коммерческая тайна
├── Государственная тайна
└── Медицинские данные
```

## 🔐 Современные угрозы и защита

### Квантовая криптография

**Принцип неопределенности Гейзенберга:**
$$\Delta x \cdot \Delta p \geq \frac{\hbar}{2}$$

**Квантовые состояния (кубиты):**
$$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$$
где $|\alpha|^2 + |\beta|^2 = 1$.

**Квантовая запутанность (состояния Белла):**
$$|\Phi^+\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$$
$$|\Phi^-\rangle = \frac{1}{\sqrt{2}}(|00\rangle - |11\rangle)$$
$$|\Psi^+\rangle = \frac{1}{\sqrt{2}}(|01\rangle + |10\rangle)$$
$$|\Psi^-\rangle = \frac{1}{\sqrt{2}}(|01\rangle - |10\rangle)$$

**Протокол BB84:**
- Алиса генерирует случайную последовательность битов: $b_i \in \{0,1\}$
- Выбирает случайные базисы: $\theta_i \in \{+, \times\}$
- Кодирует кубиты: $|0\rangle_+, |1\rangle_+, |0\rangle_\times, |1\rangle_\times$
- Боб измеряет в случайных базисах и получает результат $b'_i$

**Вероятность ошибки при перехвате:**
$$P_{error} = \sin^2\left(\frac{\pi}{8}\right) = \frac{2-\sqrt{2}}{4} \approx 0.146$$

**Протокол E91:**
Использует EPR-пары в состоянии:
$$|\Psi^-\rangle = \frac{1}{\sqrt{2}}(|01\rangle - |10\rangle)$$

**Неравенство Белла (CHSH):**
$$|E(a,b) - E(a,b') + E(a',b) + E(a',b')| \leq 2$$

Для квантовых систем может нарушаться до:
$$S_{max} = 2\sqrt{2} \approx 2.828$$

**Квантовая телепортация состояния:**
$$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$$

После измерения и классической связи восстанавливается исходное состояние.

**Квантовая стойкость к атакам:**
- **Intercept-and-resend**: обнаруживается с вероятностью $1-(3/4)^n$
- **Beam-splitting**: ограничена принципом неопределенности
- **Photon-number-splitting**: требует квантовые коды коррекции ошибок

```
Quantum Cryptography Applications:
├── Квантовое распределение ключей (QKD)
│   ├── BB84 протокол
│   ├── E91 протокол  
│   ├── SARG04 протокол
│   └── Continuous Variable QKD
├── Квантовая запутанность
│   ├── EPR пары
│   ├── Состояния Белла
│   └── Многочастичная запутанность
├── Квантовые протоколы
│   ├── Квантовая телепортация
│   ├── Квантовая секретная связь
│   └── Квантовые подписи
├── Квантовые сети
│   ├── Квантовые повторители  
│   ├── Квантовый интернет
│   └── Distributed QKD
├── Квантовая безопасность
│   ├── Анализ атак на QKD
│   ├── Device-independent QKD
│   └── Measurement-device-independent QKD
└── Практические реализации
    ├── Волоконно-оптические системы
    ├── Спутниковые QKD системы
    └── Интегрированные фотонные чипы
```

### Постквантовая криптография

**Основная угроза: алгоритм Шора**
Время работы квантового алгоритма для факторизации:
$$T_{quantum} = O((\log N)^3)$$

против классического:
$$T_{classic} = O(e^{(\log N)^{1/3}(\log \log N)^{2/3}})$$

**Lattice-based криптография:**

**Задача Learning With Errors (LWE):**
Дан вектор $\vec{s} \in \mathbb{Z}_q^n$ и пары $(a_i, b_i)$ где:
$$b_i = \langle \vec{a_i}, \vec{s} \rangle + e_i \pmod{q}$$
где $e_i$ — малая ошибка. Найти $\vec{s}$.

**Ring-LWE (RLWE):**
В кольце полиномов $R_q = \mathbb{Z}_q[x]/(x^n + 1)$:
$$b(x) = a(x) \cdot s(x) + e(x) \pmod{q}$$

**Shortest Vector Problem (SVP):**
Найти кратчайший ненулевой вектор в решетке $\Lambda$:
$$\lambda_1(\Lambda) = \min_{\vec{v} \in \Lambda, \vec{v} \neq 0} ||\vec{v}||$$

**Code-based криптография:**

**Syndrome Decoding Problem:**
Дана матрица $H \in \mathbb{F}_2^{m \times n}$ и вектор $\vec{s} \in \mathbb{F}_2^m$.
Найти $\vec{e}$ веса $\leq t$ такой, что:
$$H \vec{e}^T = \vec{s}^T$$

**McEliece криптосистема:**
- Открытый ключ: $G' = SGP$ где $G$ — порождающая матрица кода Гоппы
- Шифрование: $\vec{c} = \vec{m}G' + \vec{e}$
- Расшифрование через алгоритм декодирования

**Multivariate криптография:**

**MQ Problem:**
Система многочленов над $\mathbb{F}_q$:
$$\begin{cases}
f_1(x_1, \ldots, x_n) = 0 \\
\vdots \\
f_m(x_1, \ldots, x_n) = 0
\end{cases}$$

**Oil and Vinegar scheme:**
Разделение переменных на "oil" и "vinegar":
$$f_k = \sum_{i \in V, j \in V} \alpha_{ij}^{(k)} x_i x_j + \sum_{i \in V, j \in O} \beta_{ij}^{(k)} x_i x_j + \text{linear terms}$$

**Hash-based подписи:**

**Lamport подпись:**
Для бита $b_i$ сообщения используется:
$$sk_i^{(b_i)}, \quad pk_i^{(j)} = H(sk_i^{(j)})$$

**Merkle Tree подписи:**
Корень дерева: $root = H(H(leaf_1 || leaf_2) || H(leaf_3 || leaf_4))$

**XMSS (eXtended Merkle Signature Scheme):**
$$WOTS^+ = \text{Winternitz One-Time Signature}$$
с параметром Winternitz $w$:
$$l_1 = \lceil \frac{n}{\log_2 w} \rceil, \quad l_2 = \lfloor \frac{\log_2 l_1(w-1))}{\log_2 w} \rfloor + 1$$

**Isogeny-based криптография:**

**Supersingular Isogeny Diffie-Hellman (SIDH):**
Эллиптические кривые над $\mathbb{F}_{p^2}$ где $p = 2^a 3^b - 1$.

**Изогения степени $\ell$:**
$$\phi: E_1 \rightarrow E_2$$
$$\deg(\phi) = \ell$$

**j-инвариант:**
$$j(E) = 1728 \frac{4a^3}{4a^3 + 27b^2}$$

для кривой $y^2 = x^3 + ax + b$.

**Symmetric криптография (устойчивая к квантовым атакам):**

**Алгоритм Гровера:**
Ускорение поиска с $O(2^n)$ до $O(2^{n/2})$.

**Увеличение размеров ключей:**
- AES-128 → AES-256
- SHA-256 → SHA-512  
- Требования к стойкости удваиваются

```
Post-Quantum Cryptography Standards:
├── Lattice-based cryptography
│   ├── CRYSTALS-Kyber (шифрование)
│   ├── CRYSTALS-Dilithium (подписи)
│   ├── FALCON (подписи)
│   └── NTRU (шифрование)
├── Code-based cryptography
│   ├── Classic McEliece
│   ├── BIKE (Bit Flipping Key Encapsulation)
│   └── HQC (Hamming Quasi-Cyclic)
├── Multivariate cryptography
│   ├── Rainbow (подписи)
│   ├── GeMSS (подписи) 
│   └── LUOV (подписи)
├── Hash-based signatures
│   ├── XMSS (eXtended Merkle)
│   ├── LMS (Leighton-Micali)
│   └── SPHINCS+ (stateless)
├── Isogeny-based cryptography
│   ├── SIKE (supersingular isogeny)
│   └── CSIDH (commutative SIDH)
├── Symmetric key увеличение
│   ├── AES-256, ChaCha20
│   ├── SHA-3, BLAKE2/3
│   └── Authenticated encryption
├── NIST стандартизация
│   ├── Раунд 3 финалисты (2022)
│   ├── Дополнительные алгоритмы
│   └── Криптоанализ и безопасность
└── Стратегии миграции
    ├── Hybrid классические + пост-квантовые
    ├── Crypto-agility архитектуры
    └── Производительность и размеры ключей
```

### Блокчейн и криптовалюты
```
Blockchain Security:
├── Proof of Work
├── Proof of Stake
├── Digital signatures
├── Merkle trees
├── Hash functions
├── Smart contract security
├── Wallet security
└── Consensus mechanisms
```

## 🛠️ Инструменты и технологии

### Криптографические библиотеки
```
Cryptographic Libraries:
├── OpenSSL
├── Libsodium
├── Botan
├── Crypto++
├── cryptography (Python)
├── JCA (Java)
├── CNG (Windows)
└── CommonCrypto (macOS)
```

### Инструменты анализа безопасности
```
Security Analysis Tools:
├── Nmap (network scanning)
├── Wireshark (protocol analysis)
├── Metasploit (penetration testing)
├── Burp Suite (web application security)
├── OWASP ZAP
├── John the Ripper (password cracking)
├── Hashcat (password recovery)
└── Nessus (vulnerability scanning)
```

### Аппаратные средства безопасности
```
Hardware Security:
├── Hardware Security Modules (HSM)
├── Trusted Platform Modules (TPM)
├── Smart cards
├── USB security tokens
├── Secure enclaves
├── Hardware random number generators
├── Cryptographic accelerators
└── Secure boot
```

## 📊 Стандарты и сертификация

### Международные стандарты
```
Standards Organizations:
├── ISO/IEC 27001 (Information Security Management)
├── NIST (National Institute of Standards and Technology)
├── FIPS (Federal Information Processing Standards)
├── Common Criteria
├── IETF (Internet Engineering Task Force)
├── IEEE (Institute of Electrical and Electronics Engineers)
├── ITU-T (International Telecommunication Union)
└── RFC (Request for Comments)
```

### Соответствие требованиям
```
Compliance Frameworks:
├── GDPR (General Data Protection Regulation)
├── HIPAA (Health Insurance Portability and Accountability Act)
├── PCI DSS (Payment Card Industry Data Security Standard)
├── SOX (Sarbanes-Oxley Act)
├── FISMA (Federal Information Security Management Act)
├── SOC 2 (Service Organization Control 2)
├── ISO 27001/27002
└── COBIT (Control Objectives for Information and Related Technologies)
```

## 🎯 Практические применения

### Кибербезопасность предприятий
```
Enterprise Security:
├── Security Operations Center (SOC)
├── Incident Response
├── Threat Intelligence
├── Vulnerability Management
├── Identity and Access Management (IAM)
├── Privileged Access Management (PAM)
├── Data Loss Prevention (DLP)
└── Security Awareness Training
```

### Мобильная безопасность
```
Mobile Security:
├── App sandboxing
├── Device encryption
├── Mobile Device Management (MDM)
├── Application security
├── Secure communication
├── Biometric authentication
├── Mobile malware protection
└── BYOD (Bring Your Own Device) policies
```

### Облачная безопасность
```
Cloud Security:
├── Shared responsibility model
├── Cloud Access Security Broker (CASB)
├── Cloud Security Posture Management (CSPM)
├── Container security
├── Serverless security
├── Multi-cloud security
├── Cloud encryption
└── Cloud compliance
```

## 🔮 Будущее криптографии

### Новые направления
```
Emerging Areas:
├── Homomorphic encryption
├── Functional encryption
├── Secure multi-party computation
├── Zero-knowledge proofs
├── Differential privacy
├── Attribute-based encryption
├── Searchable encryption
└── Lightweight cryptography
```

### Исследовательские тренды
```
Research Trends:
├── Quantum-safe cryptography
├── AI/ML in cybersecurity
├── Behavioral analytics
├── Cyber threat intelligence
├── Automated security response
├── Privacy-preserving technologies
├── Decentralized security
└── Cyber-physical systems security
```

## 🎯 Практические рекомендации

### Для изучения
```
Learning Path:
1. Математические основы
2. Классическая криптография
3. Современные алгоритмы
4. Протоколы безопасности
5. Практическая криптография
6. Анализ безопасности
```

### Лабораторные работы
```
Hands-on Projects:
├── Реализация шифров
├── Анализ протоколов
├── Тестирование на проникновение
├── Создание PKI
├── Анализ малware
├── Разработка secure applications
└── Incident response simulation
```

---

**Криптография и безопасность** - это **щит и меч** в мире цифровых технологий. Понимание этих принципов **критически важно** для защиты информации в **современном взаимосвязанном мире**! 🔐🛡️ 