# Python Backend Ecosystem: Полное руководство

## 🐍 Python Fundamentals

### Python для Backend
```
Преимущества Python:
├── Читаемость кода: простой и понятный синтаксис
├── Богатая экосистема: множество библиотек
├── Быстрая разработка: высокая продуктивность
├── Многопарадигменность: ООП, функциональное программирование
├── Кросс-платформенность: работа на разных ОС
└── Сообщество: активное сообщество разработчиков

Основные концепции:
├── Duck typing: утиная типизация
├── LEGB rule: область видимости
├── Decorators: декораторы
├── Context managers: менеджеры контекста
├── Generators: генераторы
├── Asyncio: асинхронное программирование
└── GIL: Global Interpreter Lock
```

### Управление зависимостями
```
Инструменты:
├── pip: стандартный менеджер пакетов
├── pipenv: Pipfile + virtualenv
├── poetry: современный менеджер зависимостей
├── conda: научные пакеты
├── virtualenv: виртуальные окружения
└── pyenv: управление версиями Python

Poetry пример:
```toml
[tool.poetry]
name = "my-backend"
version = "0.1.0"
description = "Backend API"

[tool.poetry.dependencies]
python = "^3.11"
fastapi = "^0.104.1"
uvicorn = "^0.24.0"
sqlalchemy = "^2.0.23"
pydantic = "^2.5.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.3"
black = "^23.11.0"
flake8 = "^6.1.0"
mypy = "^1.7.1"
```

## 🌐 Web Frameworks

### FastAPI
```
Современный фреймворк:
├── Automatic API docs: автоматическая документация
├── Type hints: подсказки типов
├── Async support: асинхронная поддержка
├── Pydantic integration: валидация данных
├── High performance: высокая производительность
├── OpenAPI standard: стандарт OpenAPI
└── Dependency injection: внедрение зависимостей

Основные компоненты:
├── Path operations: операции путей
├── Request/Response models: модели запросов/ответов
├── Dependency injection: внедрение зависимостей
├── Middleware: промежуточные обработчики
├── Background tasks: фоновые задачи
├── WebSocket support: поддержка WebSocket
└── Testing: встроенное тестирование

Пример приложения:
```python
from fastapi import FastAPI, Depends, HTTPException
from pydantic import BaseModel
from sqlalchemy.orm import Session
from typing import List

app = FastAPI(title="User API", version="1.0.0")

class UserBase(BaseModel):
    email: str
    name: str

class UserCreate(UserBase):
    password: str

class User(UserBase):
    id: int
    is_active: bool
    
    class Config:
        orm_mode = True

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.post("/users/", response_model=User)
async def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return create_user_db(db=db, user=user)

@app.get("/users/", response_model=List[User])
async def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = get_users(db, skip=skip, limit=limit)
    return users

@app.get("/users/{user_id}", response_model=User)
async def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = get_user(db, user_id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user
```

### Django
```
Полнофункциональный фреймворк:
├── ORM: объектно-реляционное отображение
├── Admin panel: панель администрирования
├── Authentication: система аутентификации
├── Forms: обработка форм
├── Templates: система шаблонов
├── Middleware: промежуточные обработчики
├── Security: встроенная безопасность
└── I18n: интернационализация

Архитектура MTV:
├── Model: модель данных
├── Template: шаблон представления
├── View: представление (контроллер)
├── URL dispatcher: диспетчер URL
├── Settings: настройки приложения
└── Apps: приложения Django

Django REST Framework:
├── Serializers: сериализаторы
├── ViewSets: наборы представлений
├── Routers: маршрутизаторы
├── Permissions: разрешения
├── Authentication: аутентификация
├── Throttling: ограничение скорости
└── Pagination: пагинация

Пример API:
```python
from rest_framework import viewsets, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from .models import User
from .serializers import UserSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]
    
    @action(detail=True, methods=['post'])
    def set_password(self, request, pk=None):
        user = self.get_object()
        password = request.data.get('password')
        user.set_password(password)
        user.save()
        return Response({'status': 'password set'})
    
    @action(detail=False)
    def recent_users(self, request):
        recent = User.objects.filter(
            date_joined__gte=timezone.now() - timedelta(days=7)
        )
        serializer = self.get_serializer(recent, many=True)
        return Response(serializer.data)
```

### Flask
```
Микрофреймворк:
├── Minimalist: минималистичный
├── Flexible: гибкий
├── Extensible: расширяемый
├── Blueprints: модули
├── Jinja2: шаблонизатор
├── Werkzeug: WSGI утилиты
└── Configuration: конфигурация

Flask-RESTful пример:
```python
from flask import Flask
from flask_restful import Api, Resource, reqparse
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
db = SQLAlchemy(app)
api = Api(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

class UserResource(Resource):
    def get(self, user_id):
        user = User.query.get_or_404(user_id)
        return {'id': user.id, 'name': user.name, 'email': user.email}
    
    def put(self, user_id):
        parser = reqparse.RequestParser()
        parser.add_argument('name', type=str, required=True)
        parser.add_argument('email', type=str, required=True)
        args = parser.parse_args()
        
        user = User.query.get_or_404(user_id)
        user.name = args['name']
        user.email = args['email']
        db.session.commit()
        
        return {'id': user.id, 'name': user.name, 'email': user.email}

api.add_resource(UserResource, '/users/<int:user_id>')
```

## 🗄️ Database Integration

### SQLAlchemy
```
ORM для Python:
├── Core: низкоуровневый SQL
├── ORM: объектно-реляционное отображение
├── Connection pooling: пул соединений
├── Migrations: миграции (с Alembic)
├── Multiple databases: множественные БД
└── Async support: асинхронная поддержка

Модели:
```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    email = Column(String(120), unique=True, nullable=False)
    name = Column(String(80), nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(String(2000))
    author_id = Column(Integer, ForeignKey('users.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    author = relationship("User", back_populates="posts")

# Запросы
def get_user_posts(db: Session, user_id: int):
    return db.query(Post).filter(Post.author_id == user_id).all()

def create_user(db: Session, user: UserCreate):
    db_user = User(email=user.email, name=user.name)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

### Async SQLAlchemy
```
Асинхронная версия:
├── asyncio support: поддержка asyncio
├── Async sessions: асинхронные сессии
├── Async queries: асинхронные запросы
├── Connection pooling: пул соединений
└── Performance: производительность

Пример:
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy import select

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_user(db: AsyncSession, user_id: int):
    result = await db.execute(select(User).where(User.id == user_id))
    return result.scalar_one_or_none()

async def get_users(db: AsyncSession, skip: int = 0, limit: int = 100):
    result = await db.execute(select(User).offset(skip).limit(limit))
    return result.scalars().all()
```

### MongoDB с Motor
```
Асинхронный MongoDB:
├── Motor: асинхронный драйвер
├── Document database: документная БД
├── Flexible schema: гибкая схема
├── Aggregation: агрегация
├── GridFS: файловая система
└── Replica sets: наборы реплик

Пример:
```python
from motor.motor_asyncio import AsyncIOMotorClient
from bson import ObjectId
from typing import List, Optional

class MongoManager:
    def __init__(self, connection_string: str):
        self.client = AsyncIOMotorClient(connection_string)
        self.db = self.client.myapp
        self.users = self.db.users
    
    async def create_user(self, user_data: dict) -> str:
        result = await self.users.insert_one(user_data)
        return str(result.inserted_id)
    
    async def get_user(self, user_id: str) -> Optional[dict]:
        user = await self.users.find_one({"_id": ObjectId(user_id)})
        if user:
            user["_id"] = str(user["_id"])
        return user
    
    async def get_users(self, skip: int = 0, limit: int = 100) -> List[dict]:
        cursor = self.users.find({}).skip(skip).limit(limit)
        users = await cursor.to_list(length=limit)
        for user in users:
            user["_id"] = str(user["_id"])
        return users
```

## 🔐 Authentication & Security

### JWT с PyJWT
```
JSON Web Tokens:
├── Token generation: генерация токенов
├── Token verification: проверка токенов
├── Claims: утверждения
├── Expiration: истечение срока
├── Refresh tokens: токены обновления
└── Security: безопасность

Реализация:
```python
import jwt
from datetime import datetime, timedelta
from passlib.context import CryptContext
from typing import Optional

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        email: str = payload.get("sub")
        if email is None:
            raise HTTPException(status_code=401, detail="Invalid token")
        return email
    except jwt.PyJWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### OAuth2 с Authlib
```
OAuth2 провайдер:
├── Authorization server: сервер авторизации
├── Resource server: сервер ресурсов
├── Client credentials: учетные данные клиента
├── Authorization code: код авторизации
├── Implicit grant: неявное предоставление
└── Password grant: предоставление пароля

FastAPI + OAuth2:
```python
from authlib.integrations.starlette_client import OAuth
from starlette.middleware.sessions import SessionMiddleware

oauth = OAuth()
oauth.register(
    name='google',
    client_id='your-client-id',
    client_secret='your-client-secret',
    server_metadata_url='https://accounts.google.com/.well-known/openid_configuration',
    client_kwargs={'scope': 'openid email profile'}
)

app.add_middleware(SessionMiddleware, secret_key="your-secret-key")

@app.get('/auth/google')
async def google_auth(request: Request):
    redirect_uri = request.url_for('google_callback')
    return await oauth.google.authorize_redirect(request, redirect_uri)

@app.get('/auth/google/callback')
async def google_callback(request: Request):
    token = await oauth.google.authorize_access_token(request)
    user_info = token.get('userinfo')
    # Обработка информации о пользователе
    return {"user": user_info}
```

## 🔄 Async Programming

### Asyncio Fundamentals
```
Асинхронное программирование:
├── Event loop: цикл событий
├── Coroutines: корутины
├── Tasks: задачи
├── Futures: будущие значения
├── Async/await: синтаксис
└── Concurrency: конкурентность

Основные концепции:
├── async def: определение корутины
├── await: ожидание результата
├── asyncio.create_task(): создание задачи
├── asyncio.gather(): группировка задач
├── asyncio.run(): запуск цикла событий
└── asyncio.sleep(): асинхронная пауза
```

### Async HTTP клиенты
```
aiohttp:
├── Async HTTP client: асинхронный HTTP клиент
├── Async HTTP server: асинхронный HTTP сервер
├── WebSocket support: поддержка WebSocket
├── Middleware: промежуточные обработчики
├── Session management: управление сессиями
└── Connection pooling: пул соединений

Пример:
```python
import aiohttp
import asyncio
from typing import List, Dict

async def fetch_user(session: aiohttp.ClientSession, user_id: int) -> Dict:
    async with session.get(f'https://api.example.com/users/{user_id}') as response:
        return await response.json()

async def fetch_multiple_users(user_ids: List[int]) -> List[Dict]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_user(session, user_id) for user_id in user_ids]
        return await asyncio.gather(*tasks)

# Использование
users = asyncio.run(fetch_multiple_users([1, 2, 3, 4, 5]))
```

httpx:
```python
import httpx
import asyncio

async def async_request_example():
    async with httpx.AsyncClient() as client:
        response = await client.get('https://api.example.com/data')
        return response.json()

# Поддерживает sync и async API
def sync_request_example():
    with httpx.Client() as client:
        response = client.get('https://api.example.com/data')
        return response.json()
```

## 🧪 Testing

### Pytest
```
Фреймворк тестирования:
├── Test discovery: обнаружение тестов
├── Fixtures: фикстуры
├── Parametrization: параметризация
├── Mocking: мокирование
├── Coverage: покрытие кода
├── Plugins: плагины
└── Async testing: асинхронное тестирование

Пример тестов:
```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from main import app, get_db
from database import Base

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture(scope="session")
def db():
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client(db):
    def override_get_db():
        try:
            yield db
        finally:
            db.close()
    
    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as c:
        yield c

def test_create_user(client):
    response = client.post("/users/", json={"name": "Test User", "email": "test@example.com"})
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Test User"
    assert data["email"] == "test@example.com"

def test_get_user(client):
    # Создаем пользователя
    create_response = client.post("/users/", json={"name": "Test User", "email": "test@example.com"})
    user_id = create_response.json()["id"]
    
    # Получаем пользователя
    response = client.get(f"/users/{user_id}")
    assert response.status_code == 200
    data = response.json()
    assert data["name"] == "Test User"
```

### Async Testing
```
pytest-asyncio:
├── Async fixtures: асинхронные фикстуры
├── Async tests: асинхронные тесты
├── Event loop: цикл событий
├── Async mocking: асинхронное мокирование
└── Database testing: тестирование БД

Пример:
```python
import pytest
import asyncio
from httpx import AsyncClient
from main import app

@pytest.mark.asyncio
async def test_async_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/users/")
        assert response.status_code == 200

@pytest.fixture
async def async_client():
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.mark.asyncio
async def test_with_fixture(async_client):
    response = await async_client.post("/users/", json={"name": "Test", "email": "test@example.com"})
    assert response.status_code == 201
```

## 📊 API Documentation

### OpenAPI с FastAPI
```
Автоматическая документация:
├── Swagger UI: интерактивная документация
├── ReDoc: альтернативная документация
├── OpenAPI schema: схема API
├── Response models: модели ответов
├── Request models: модели запросов
└── Examples: примеры

Расширенная документация:
```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional

app = FastAPI(
    title="My API",
    description="This is a very fancy project, with auto docs for the API and everything",
    version="2.5.0",
    contact={
        "name": "Deadpoolio the Amazing",
        "url": "http://x-force.example.com/contact/",
        "email": "dp@x-force.example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
)

class UserCreate(BaseModel):
    name: str = Field(..., description="The name of the user", example="John Doe")
    email: str = Field(..., description="The email of the user", example="john@example.com")
    age: Optional[int] = Field(None, description="The age of the user", example=25)

@app.post("/users/", response_model=User, tags=["users"])
async def create_user(
    user: UserCreate,
    db: Session = Depends(get_db)
):
    """
    Create a new user with all the information:
    
    - **name**: required string
    - **email**: required string, must be valid email
    - **age**: optional integer
    """
    return create_user_db(db=db, user=user)
```

## 🚀 Performance Optimization

### Caching
```
Redis caching:
├── Cache-aside pattern: паттерн кеширования
├── TTL: время жизни
├── Cache invalidation: инвалидация кеша
├── Distributed caching: распределенное кеширование
└── Cache warming: прогрев кеша

Реализация:
```python
import redis
import json
import asyncio
from typing import Optional, Any

class CacheManager:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url, decode_responses=True)
    
    async def get(self, key: str) -> Optional[Any]:
        try:
            value = self.redis.get(key)
            return json.loads(value) if value else None
        except Exception:
            return None
    
    async def set(self, key: str, value: Any, ttl: int = 3600):
        try:
            self.redis.setex(key, ttl, json.dumps(value))
        except Exception:
            pass
    
    async def delete(self, key: str):
        try:
            self.redis.delete(key)
        except Exception:
            pass

# Использование в FastAPI
cache = CacheManager("redis://localhost:6379")

@app.get("/users/{user_id}")
async def get_user(user_id: int, db: Session = Depends(get_db)):
    cache_key = f"user:{user_id}"
    
    # Проверяем кеш
    cached_user = await cache.get(cache_key)
    if cached_user:
        return cached_user
    
    # Получаем из БД
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Кешируем результат
    user_data = {"id": user.id, "name": user.name, "email": user.email}
    await cache.set(cache_key, user_data, ttl=1800)
    
    return user_data
```

### Database Optimization
```
Оптимизация запросов:
├── Eager loading: жадная загрузка
├── Lazy loading: ленивая загрузка
├── Connection pooling: пул соединений
├── Query optimization: оптимизация запросов
├── Indexing: индексация
└── Batch operations: пакетные операции

SQLAlchemy оптимизация:
```python
from sqlalchemy.orm import joinedload, selectinload
from sqlalchemy import func

# Жадная загрузка связей
def get_users_with_posts(db: Session):
    return db.query(User).options(joinedload(User.posts)).all()

# Пакетные операции
def create_multiple_users(db: Session, users: List[UserCreate]):
    db_users = [User(**user.dict()) for user in users]
    db.add_all(db_users)
    db.commit()
    return db_users

# Агрегация
def get_user_stats(db: Session):
    return db.query(
        func.count(User.id).label('total_users'),
        func.count(User.id).filter(User.is_active == True).label('active_users'),
        func.avg(User.age).label('average_age')
    ).first()

# Оптимизированная пагинация
def get_users_paginated(db: Session, page: int, per_page: int):
    offset = (page - 1) * per_page
    return db.query(User).offset(offset).limit(per_page).all()
```

## 🔧 Background Tasks

### Celery
```
Распределенная очередь задач:
├── Task queue: очередь задач
├── Workers: воркеры
├── Broker: брокер сообщений
├── Result backend: хранилище результатов
├── Scheduling: планирование задач
├── Monitoring: мониторинг
└── Retry logic: логика повторных попыток

Конфигурация:
```python
from celery import Celery
from kombu import Queue

app = Celery('myapp')
app.config_from_object('celeryconfig')

# celeryconfig.py
broker_url = 'redis://localhost:6379/0'
result_backend = 'redis://localhost:6379/0'
task_serializer = 'json'
accept_content = ['json']
result_serializer = 'json'
timezone = 'UTC'
enable_utc = True

task_routes = {
    'myapp.tasks.send_email': {'queue': 'email'},
    'myapp.tasks.process_data': {'queue': 'data'},
}

task_default_queue = 'default'
task_queues = (
    Queue('default', routing_key='default'),
    Queue('email', routing_key='email'),
    Queue('data', routing_key='data'),
)

# Задачи
@app.task(bind=True, max_retries=3)
def send_email(self, email_data):
    try:
        # Логика отправки email
        send_email_logic(email_data)
        return f"Email sent to {email_data['to']}"
    except Exception as exc:
        self.retry(countdown=60, exc=exc)

@app.task
def process_large_dataset(dataset_id):
    # Обработка больших данных
    dataset = get_dataset(dataset_id)
    result = process_data(dataset)
    return result

# Использование в FastAPI
@app.post("/send-email/")
async def send_email_endpoint(email_data: EmailData):
    task = send_email.delay(email_data.dict())
    return {"task_id": task.id, "status": "queued"}

@app.get("/task/{task_id}")
async def get_task_status(task_id: str):
    task = send_email.AsyncResult(task_id)
    return {"task_id": task_id, "status": task.status, "result": task.result}
```

### Background Tasks в FastAPI
```python
from fastapi import BackgroundTasks

def write_log(message: str):
    with open("log.txt", "a") as log:
        log.write(f"{message}\n")

@app.post("/send-notification/")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, f"Notification sent to {email}")
    background_tasks.add_task(send_email_notification, email)
    return {"message": "Notification sent in the background"}
```

## 📊 Monitoring & Logging

### Structured Logging
```python
import structlog
import logging
from datetime import datetime

# Конфигурация structlog
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.processors.JSONRenderer()
    ],
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    wrapper_class=structlog.stdlib.BoundLogger,
    cache_logger_on_first_use=True,
)

logger = structlog.get_logger()

# Middleware для логирования
@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    start_time = datetime.utcnow()
    
    logger.info(
        "Request started",
        method=request.method,
        url=str(request.url),
        user_agent=request.headers.get("user-agent")
    )
    
    response = await call_next(request)
    
    process_time = (datetime.utcnow() - start_time).total_seconds()
    
    logger.info(
        "Request completed",
        method=request.method,
        url=str(request.url),
        status_code=response.status_code,
        process_time=process_time
    )
    
    return response
```

### Prometheus Metrics
```python
from prometheus_client import Counter, Histogram, generate_latest
from prometheus_client.openmetrics.exposition import CONTENT_TYPE_LATEST

# Метрики
REQUEST_COUNT = Counter('requests_total', 'Total requests', ['method', 'endpoint'])
REQUEST_DURATION = Histogram('request_duration_seconds', 'Request duration')

@app.middleware("http")
async def prometheus_middleware(request: Request, call_next):
    start_time = time.time()
    
    response = await call_next(request)
    
    REQUEST_COUNT.labels(method=request.method, endpoint=request.url.path).inc()
    REQUEST_DURATION.observe(time.time() - start_time)
    
    return response

@app.get("/metrics")
async def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

## 🐳 Containerization

### Docker
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Устанавливаем зависимости системы
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Копируем файлы зависимостей
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копируем код приложения
COPY . .

# Создаем пользователя
RUN useradd --create-home --shell /bin/bash app
USER app

# Экспонируем порт
EXPOSE 8000

# Команда запуска
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose
```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  worker:
    build: .
    command: celery -A main.celery worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

volumes:
  postgres_data:
```

## 🌍 Deployment

### Gunicorn + Nginx
```python
# gunicorn.conf.py
bind = "0.0.0.0:8000"
workers = 4
worker_class = "uvicorn.workers.UvicornWorker"
worker_connections = 1000
max_requests = 1000
max_requests_jitter = 100
timeout = 30
keepalive = 2
preload_app = True
```

```nginx
# nginx.conf
upstream app {
    server app:8000;
}

server {
    listen 80;
    server_name yourdomain.com;
    
    location / {
        proxy_pass http://app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /static/ {
        alias /app/static/;
    }
    
    location /media/ {
        alias /app/media/;
    }
}
```

### Kubernetes
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: python-api
  template:
    metadata:
      labels:
        app: python-api
    spec:
      containers:
      - name: api
        image: python-api:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: python-api-service
spec:
  selector:
    app: python-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

## 🔒 Security Best Practices

### Input Validation
```python
from pydantic import BaseModel, validator, Field
from typing import Optional
import re

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., regex=r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
    age: Optional[int] = Field(None, ge=0, le=150)
    
    @validator('name')
    def validate_name(cls, v):
        if not re.match(r'^[a-zA-Z\s]+$', v):
            raise ValueError('Name must contain only letters and spaces')
        return v.strip()
    
    @validator('email')
    def validate_email(cls, v):
        # Дополнительная валидация email
        if '@' not in v:
            raise ValueError('Invalid email format')
        return v.lower()

# Middleware для валидации
@app.middleware("http")
async def validation_middleware(request: Request, call_next):
    # Проверка размера запроса
    if request.headers.get("content-length"):
        content_length = int(request.headers["content-length"])
        if content_length > 1024 * 1024:  # 1MB limit
            return JSONResponse(
                status_code=413,
                content={"error": "Request too large"}
            )
    
    response = await call_next(request)
    return response
```

### SQL Injection Prevention
```python
# Правильно - используем ORM
def get_user_by_email(db: Session, email: str):
    return db.query(User).filter(User.email == email).first()

# Правильно - параметризованные запросы
def get_user_by_email_raw(db: Session, email: str):
    result = db.execute(
        text("SELECT * FROM users WHERE email = :email"),
        {"email": email}
    )
    return result.fetchone()

# НЕПРАВИЛЬНО - уязвимо к SQL injection
def get_user_by_email_vulnerable(db: Session, email: str):
    query = f"SELECT * FROM users WHERE email = '{email}'"
    result = db.execute(text(query))
    return result.fetchone()
```

## 📚 Learning Path

### Beginner
```
Основы:
├── Python fundamentals: основы Python
├── FastAPI basics: основы FastAPI
├── Database integration: интеграция с БД
├── Basic authentication: базовая аутентификация
├── Testing: основы тестирования
└── Deployment: базовое развертывание
```

### Intermediate
```
Средний уровень:
├── Advanced FastAPI: продвинутый FastAPI
├── Async programming: асинхронное программирование
├── Caching: кеширование
├── Background tasks: фоновые задачи
├── Monitoring: мониторинг
├── Security: безопасность
└── Docker: контейнеризация
```

### Advanced
```
Продвинутый уровень:
├── Microservices: микросервисы
├── Performance optimization: оптимизация производительности
├── Advanced security: продвинутая безопасность
├── Kubernetes: оркестрация
├── System design: проектирование систем
├── DevOps: культура разработки
└── Distributed systems: распределенные системы
```

## 🛠️ Tools & Libraries

### Development Tools
```
IDE и редакторы:
├── PyCharm: полнофункциональная IDE
├── VS Code: легковесный редактор
├── Vim/Neovim: консольные редакторы
└── Jupyter: интерактивные блокноты

Линтеры и форматеры:
├── Black: автоматическое форматирование
├── isort: сортировка импортов
├── flake8: проверка стиля кода
├── mypy: статическая проверка типов
└── bandit: сканирование безопасности

Отладка:
├── pdb: встроенный отладчик
├── ipdb: улучшенный отладчик
├── pytest-xdist: параллельное тестирование
└── coverage: покрытие кода
```

### Production Tools
```
Мониторинг:
├── Sentry: отслеживание ошибок
├── New Relic: мониторинг производительности
├── Datadog: мониторинг инфраструктуры
└── Prometheus: метрики

Развертывание:
├── Heroku: простое развертывание
├── AWS: облачная платформа
├── Google Cloud: облачная платформа
├── DigitalOcean: простые VPS
└── Kubernetes: оркестрация контейнеров
```

Данное руководство представляет comprehensive обзор экосистемы Python для backend разработки, покрывая все аспекты от базовых концепций до продвинутых техник развертывания и мониторинга. 