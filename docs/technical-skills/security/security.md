# Безопасность в FullStack разработке

## OWASP Top 10 (2021)

### 1. Broken Access Control
```javascript
// Плохо: прямое обращение к объекту без проверки
app.get('/users/:id', (req, res) => {
  const user = getUserById(req.params.id); // Любой может получить любого пользователя
  res.json(user);
});

// Хорошо: проверка доступа
app.get('/users/:id', authenticateToken, (req, res) => {
  const userId = req.params.id;
  
  // Проверяем, что пользователь может получить эти данные
  if (req.user.id !== userId && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  const user = getUserById(userId);
  res.json(user);
});
```

### 2. Cryptographic Failures
```javascript
// Плохо: слабое хеширование
const hash = crypto.createHash('md5').update(password).digest('hex');

// Хорошо: bcrypt с солью
const bcrypt = require('bcrypt');
const saltRounds = 12;
const hash = await bcrypt.hash(password, saltRounds);
```

### 3. Injection (SQL, NoSQL, XSS)
```javascript
// Плохо: SQL инъекция
app.post('/login', (req, res) => {
  const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;
  // Опасно!
});

// Хорошо: подготовленные запросы
app.post('/login', (req, res) => {
  const query = 'SELECT * FROM users WHERE email = ?';
  db.query(query, [req.body.email], (err, results) => {
    // Безопасно
  });
});
```

## JWT (JSON Web Tokens)

### Структура JWT
```
Header.Payload.Signature
```

### Реализация JWT
```javascript
// services/api-gateway/src/auth/jwt.js
const jwt = require('jsonwebtoken');

class JWTService {
  constructor() {
    this.secret = process.env.JWT_SECRET;
    this.refreshSecret = process.env.JWT_REFRESH_SECRET;
  }

  generateTokens(payload) {
    const accessToken = jwt.sign(
      payload,
      this.secret,
      { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
      payload,
      this.refreshSecret,
      { expiresIn: '7d' }
    );
    
    return { accessToken, refreshToken };
  }

  verifyToken(token) {
    try {
      return jwt.verify(token, this.secret);
    } catch (error) {
      throw new Error('Invalid token');
    }
  }

  refreshToken(refreshToken) {
    try {
      const payload = jwt.verify(refreshToken, this.refreshSecret);
      delete payload.iat;
      delete payload.exp;
      return this.generateTokens(payload);
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }
}

// Middleware для проверки токена
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }
  
  try {
    const user = jwtService.verifyToken(token);
    req.user = user;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
};
```

## OAuth2

### Authorization Code Flow
```
1. Client -> Authorization Server: GET /authorize
2. User authenticates
3. Authorization Server -> Client: Redirect with code
4. Client -> Authorization Server: POST /token (with code)
5. Authorization Server -> Client: Access token
6. Client -> Resource Server: API call with token
```

### OAuth2 Implementation
```javascript
// services/api-gateway/src/auth/oauth2.js
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
  try {
    let user = await User.findOne({ googleId: profile.id });
    
    if (!user) {
      user = await User.create({
        googleId: profile.id,
        name: profile.displayName,
        email: profile.emails[0].value
      });
    }
    
    return done(null, user);
  } catch (error) {
    return done(error, null);
  }
}));

// Routes
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    const tokens = jwtService.generateTokens({
      id: req.user.id,
      email: req.user.email
    });
    res.redirect(`/dashboard?token=${tokens.accessToken}`);
  }
);
```

## Rate Limiting

```javascript
// services/api-gateway/src/middleware/rateLimit.js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const limiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate-limit:'
  }),
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100, // 100 запросов на IP
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
});

// Строгий лимит для API аутентификации
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // 5 попыток за 15 минут
  skipSuccessfulRequests: true,
});

app.use('/api/', limiter);
app.use('/auth/', authLimiter);
```

## Content Security Policy (CSP)

```javascript
// services/api-gateway/src/middleware/security.js
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", "https://cdnjs.cloudflare.com"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
      frameAncestors: ["'none'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  crossOriginEmbedderPolicy: false,
}));
```

## Input Validation

```javascript
// services/api-gateway/src/validation/schemas.js
const Joi = require('joi');

const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])')).required(),
  name: Joi.string().min(2).max(50).required(),
});

const validateInput = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    if (error) {
      return res.status(400).json({ error: error.details[0].message });
    }
    next();
  };
};

app.post('/users', validateInput(createUserSchema), createUser);
```

## HTTPS и TLS

```javascript
// services/api-gateway/src/server.js
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('path/to/private-key.pem'),
  cert: fs.readFileSync('path/to/certificate.pem'),
  // Настройки для усиления безопасности
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384',
  honorCipherOrder: true,
  secureProtocol: 'TLSv1_2_method'
};

const server = https.createServer(options, app);

// Принуждение к HTTPS
app.use((req, res, next) => {
  if (req.header('x-forwarded-proto') !== 'https') {
    res.redirect(`https://${req.header('host')}${req.url}`);
  } else {
    next();
  }
});
```

## Secrets Management

```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  database-password: "super-secret-password"
  jwt-secret: "very-long-random-string"
  api-key: "external-service-key"
```

```javascript
// Environment-based secrets
const config = {
  jwt: {
    secret: process.env.JWT_SECRET || (() => {
      throw new Error('JWT_SECRET is required');
    })(),
    refreshSecret: process.env.JWT_REFRESH_SECRET,
  },
  database: {
    url: process.env.DATABASE_URL,
    password: process.env.DATABASE_PASSWORD,
  },
  redis: {
    url: process.env.REDIS_URL,
  },
  google: {
    clientId: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  },
};
```

## Аудит и логирование

```javascript
// services/api-gateway/src/middleware/audit.js
const auditLogger = require('../utils/auditLogger');

const auditMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    auditLogger.log({
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration,
      userId: req.user?.id,
      userAgent: req.get('User-Agent'),
      ip: req.ip,
      timestamp: new Date().toISOString(),
    });
  });
  
  next();
};

// Логирование чувствительных операций
const logSensitiveOperation = (operation, userId, details) => {
  auditLogger.log({
    type: 'SENSITIVE_OPERATION',
    operation,
    userId,
    details,
    timestamp: new Date().toISOString(),
  }, 'warn');
};
```

## Docker Security

```dockerfile
# Dockerfile с безопасными настройками
FROM node:18-alpine

# Создаем непривилегированного пользователя
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Устанавливаем зависимости как root
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Копируем код и меняем владельца
COPY --chown=nextjs:nodejs . .

# Переключаемся на непривилегированного пользователя
USER nextjs

# Открываем порт
EXPOSE 3000

# Запускаем приложение
CMD ["npm", "start"]
```

## Kubernetes Security

```yaml
# k8s/security-policies.yaml
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1001
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "250m"
```

## Лучшие практики безопасности

### Authentication & Authorization
1. **Используйте сильные пароли** и двухфакторную аутентификацию
2. **Реализуйте принцип минимальных привилегий**
3. **Регулярно обновляйте токены**
4. **Логируйте все попытки аутентификации**

### Data Protection
1. **Шифруйте данные** в покое и при передаче
2. **Используйте HTTPS** для всех соединений
3. **Не храните чувствительные данные** в логах
4. **Регулярно создавайте бэкапы**

### Infrastructure
1. **Обновляйте зависимости** и исправляйте уязвимости
2. **Используйте сканеры безопасности** в CI/CD
3. **Настройте мониторинг** подозрительной активности
4. **Ограничивайте сетевой доступ**

### Development
1. **Валидируйте все входные данные**
2. **Используйте параметризованные запросы**
3. **Не храните секреты в коде**
4. **Проводите код-ревью** с фокусом на безопасность

## Заключение

Безопасность - это не разовое действие, а непрерывный процесс:
- **Регулярные аудиты** и пентесты
- **Обновление знаний** о новых угрозах
- **Автоматизация** проверок безопасности
- **Культура безопасности** в команде

## См. также
- [CI/CD и DevOps](./cicd-devops.md)
- [Тестирование](./testing.md)
- [Мониторинг](./deployment-monitoring.md) 