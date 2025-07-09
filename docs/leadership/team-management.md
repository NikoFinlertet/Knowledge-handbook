# Управление командой и процессами разработки

## Agile/Scrum

### Scrum Framework
```
Sprint Planning → Sprint → Daily Standup → Sprint Review → Retrospective
     ↓             ↓           ↓              ↓             ↓
   2-4 weeks    1-4 weeks   15 min        1-2 hours    1-2 hours
```

### Sprint Planning
```markdown
## Sprint Planning Template

### Sprint Goal
Четкая цель спринта, понятная всем участникам команды.

### Capacity Planning
- Команда: 5 разработчиков
- Доступные дни: 10 рабочих дней
- Velocity: 40 story points (прошлый спринт)
- Планируемый объем: 35-45 story points

### Backlog Items
1. **User Story #123** (8 SP)
   - Acceptance Criteria
   - Definition of Done
   - Dependencies
   
2. **Bug Fix #456** (3 SP)
   - Steps to reproduce
   - Expected behavior
   
### Sprint Backlog
- [ ] Task 1: API endpoint implementation (5 SP)
- [ ] Task 2: Frontend component (8 SP)
- [ ] Task 3: Database migration (3 SP)
```

## Оценка задач

### Planning Poker
```
Fibonacci: 1, 2, 3, 5, 8, 13, 21, 34, 55, 89
T-Shirt: XS, S, M, L, XL, XXL

Критерии оценки:
- Сложность реализации
- Объем работы
- Риски и неопределенность
- Зависимости
```

### Story Points Guide
```markdown
## Story Points Reference

### 1 Point - Trivial
- Исправление опечатки
- Изменение текста
- Простая конфигурация

### 3 Points - Simple
- Простая CRUD операция
- Добавление валидации
- Обновление зависимости

### 5 Points - Medium
- Новый API endpoint
- Сложная бизнес-логика
- Интеграция с внешним сервисом

### 8 Points - Complex
- Новая функциональность
- Архитектурные изменения
- Сложная интеграция

### 13+ Points - Very Complex
- Необходимо разбить на меньшие задачи
```

## Код-ревью

### Pull Request Template
```markdown
## Description
Краткое описание изменений

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added where necessary
- [ ] Documentation updated
- [ ] No console.log or debug code

## Screenshots (if applicable)
Скриншоты UI изменений

## Related Issues
Fixes #123
```

### Code Review Guidelines
```typescript
// Хорошо: понятное именование
const calculateTotalPrice = (items: CartItem[]): number => {
  return items.reduce((total, item) => total + item.price * item.quantity, 0);
};

// Плохо: неясное именование
const calc = (items: any[]): number => {
  return items.reduce((a, b) => a + b.p * b.q, 0);
};

// Хорошо: разделение ответственности
class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  async createUser(userData: CreateUserDto): Promise<User> {
    const user = await this.userRepository.create(userData);
    await this.emailService.sendWelcomeEmail(user.email);
    return user;
  }
}

// Плохо: смешанная ответственность
class UserService {
  async createUser(userData: any): Promise<any> {
    // Создание пользователя
    const user = { ...userData, id: generateId() };
    await database.save(user);
    
    // Отправка email (не должно быть здесь)
    const emailHtml = `<h1>Welcome ${user.name}!</h1>`;
    await sendEmail(user.email, 'Welcome', emailHtml);
    
    return user;
  }
}
```

### Review Checklist
```markdown
## Code Review Checklist

### Functionality
- [ ] Code works as expected
- [ ] Edge cases handled
- [ ] Error handling implemented
- [ ] Performance considerations

### Code Quality
- [ ] Clean and readable code
- [ ] Proper naming conventions
- [ ] No code duplication
- [ ] SOLID principles followed

### Security
- [ ] Input validation
- [ ] No hardcoded secrets
- [ ] SQL injection prevention
- [ ] XSS protection

### Testing
- [ ] Tests cover new functionality
- [ ] Tests are meaningful
- [ ] No brittle tests
- [ ] Mock dependencies properly

### Documentation
- [ ] Comments where necessary
- [ ] API documentation updated
- [ ] README updated if needed
```

## Наставничество

### Onboarding Process
```markdown
## Developer Onboarding Plan

### Week 1: Environment Setup
- [ ] Development environment setup
- [ ] Access to repositories and tools
- [ ] Architecture overview presentation
- [ ] Meet the team (1:1 meetings)

### Week 2: First Tasks
- [ ] Simple bug fixes (1-2 story points)
- [ ] Code review participation
- [ ] Development process understanding
- [ ] First PR submission

### Week 3: Feature Development
- [ ] Small feature implementation
- [ ] Testing best practices
- [ ] Code review giving/receiving
- [ ] Team collaboration

### Week 4: Independence
- [ ] Medium complexity task
- [ ] Process documentation review
- [ ] Feedback session with mentor
- [ ] Goal setting for next month
```

### Mentoring Best Practices
```markdown
## Mentoring Guide

### Do's
✅ Ask guiding questions instead of giving direct answers
✅ Pair programming sessions
✅ Regular check-ins and feedback
✅ Share knowledge and resources
✅ Be patient and supportive

### Don'ts
❌ Micromanage or dictate solutions
❌ Assume knowledge level
❌ Skip explanation of "why"
❌ Give all answers immediately
❌ Forget to follow up

### Pair Programming Session Structure
1. **Problem Understanding** (5 min)
2. **Approach Discussion** (10 min)
3. **Implementation** (30 min)
4. **Review and Refactor** (10 min)
5. **Next Steps** (5 min)
```

## Презентация решений

### Technical Presentation Template
```markdown
## Technical Solution Presentation

### 1. Problem Statement (5 min)
- Current situation
- Business impact
- Technical challenges

### 2. Solution Overview (10 min)
- High-level approach
- Architecture diagram
- Key technologies

### 3. Implementation Details (15 min)
- Technical specifications
- Code examples
- Integration points

### 4. Benefits & Trade-offs (5 min)
- Advantages
- Limitations
- Alternative approaches

### 5. Timeline & Resources (5 min)
- Implementation phases
- Resource requirements
- Risk mitigation

### 6. Q&A (10 min)
```

### Architecture Decision Record (ADR)
```markdown
# ADR-001: Microservices Architecture

## Status
Accepted

## Context
Our monolithic application is becoming difficult to maintain and scale.
Team growth makes parallel development challenging.

## Decision
We will adopt microservices architecture with the following services:
- User Service (Go)
- Order Service (Python)
- API Gateway (Node.js)

## Consequences

### Positive
- Independent scaling
- Technology diversity
- Team autonomy
- Fault isolation

### Negative
- Increased complexity
- Network latency
- Data consistency challenges
- Monitoring overhead

## Implementation
- Phase 1: Extract User Service (4 weeks)
- Phase 2: Extract Order Service (6 weeks)
- Phase 3: Implement API Gateway (2 weeks)
```

## Коммуникация с бизнесом

### Business Requirements Translation
```markdown
## Business Requirement: "Users want faster checkout"

### Technical Analysis
1. **Current Performance**
   - Checkout takes 5-8 seconds
   - Database queries: 12 per checkout
   - External API calls: 3 per checkout

2. **Technical Solutions**
   - Database query optimization
   - Caching layer implementation
   - Async processing for non-critical operations

3. **Business Impact**
   - 50% faster checkout (target: 2-3 seconds)
   - Increased conversion rate
   - Better user experience

### Implementation Plan
- Week 1-2: Database optimization
- Week 3: Cache implementation
- Week 4: Async processing
- Week 5: Testing and monitoring
```

### Stakeholder Communication
```markdown
## Weekly Status Report Template

### Completed This Week
- ✅ User authentication API
- ✅ Order processing optimization
- ✅ Security vulnerabilities fixed

### In Progress
- 🔄 Payment integration (75% complete)
- 🔄 Mobile app UI updates (40% complete)

### Planned Next Week
- 📋 Shopping cart persistence
- 📋 Email notification system
- 📋 Performance testing

### Blockers & Risks
- ⚠️ Third-party API rate limits
- ⚠️ Database migration scheduling

### Metrics
- Deployment frequency: 3 times this week
- Lead time: 2.5 days average
- Bug reports: 2 (down from 5 last week)
```

## Git Workflow

### Feature Branch Strategy
```bash
# Создание feature branch
git checkout -b feature/user-authentication
git push -u origin feature/user-authentication

# Работа с коммитами
git add .
git commit -m "feat: add user login endpoint

- Implement JWT authentication
- Add input validation
- Include rate limiting
- Update API documentation

Closes #123"

# Pull Request workflow
git push origin feature/user-authentication
# Create PR in GitHub
# Code review process
# Merge to main
```

### Commit Message Convention
```
<type>(<scope>): <subject>

<body>

<footer>

Types:
- feat: новая функциональность
- fix: исправление бага
- docs: обновление документации
- style: форматирование кода
- refactor: рефакторинг
- test: добавление тестов
- chore: обновление зависимостей

Example:
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh to improve user experience
and reduce login frequency.

- Add refresh endpoint
- Update client-side token handling
- Add tests for refresh flow

Closes #456
Breaking Change: Auth header format changed
```

## Командные инструменты

### Daily Standup Format
```markdown
## Daily Standup (15 min max)

### What I did yesterday
- Completed user API endpoints
- Fixed 2 bugs in order service

### What I'm doing today  
- Working on payment integration
- Code review for frontend team

### Blockers
- Waiting for API documentation from payments team
- Need help with Docker configuration
```

### Retrospective Techniques

#### Start/Stop/Continue
```markdown
## Sprint Retrospective

### Start (What should we start doing?)
- Daily code reviews
- Automated testing in CI
- Architecture discussions

### Stop (What should we stop doing?)
- Long meetings without agenda
- Manual deployment process
- Skipping documentation

### Continue (What should we keep doing?)
- Pair programming sessions
- Regular team communication
- Knowledge sharing sessions
```

#### 4L's Retrospective
```markdown
## 4L's Retrospective

### Liked
- Great team collaboration
- Successful feature delivery
- Good code quality

### Learned
- New testing strategies
- Better Git workflow
- Performance optimization techniques

### Lacked
- Clear requirements
- Sufficient testing time
- Better error monitoring

### Longed For
- Automated deployment
- Better development tools
- More predictable sprint planning
```

## Лучшие практики

### Team Dynamics
1. **Психологическая безопасность** - создавайте среду для открытого общения
2. **Коллективная ответственность** - вся команда отвечает за качество
3. **Непрерывное обучение** - регулярные знания sharing сессии
4. **Автономия команды** - делегирование принятия решений

### Process Improvement
1. **Измеряйте метрики** - lead time, deployment frequency, MTTR
2. **Экспериментируйте** - пробуйте новые подходы
3. **Ретроспективы** - регулярно анализируйте и улучшайте процессы
4. **Автоматизация** - устраняйте ручную работу

### Knowledge Management
1. **Документация** - поддерживайте актуальную документацию
2. **Code review** - обучающий процесс для всей команды
3. **Architecture decisions** - документируйте важные решения
4. **Onboarding** - структурированный процесс для новых участников

## Заключение

Эффективное управление командой включает:
- **Четкие процессы** разработки
- **Качественные инструменты** коммуникации
- **Культуру обучения** и взаимопомощи
- **Фокус на ценности** для бизнеса

## См. также
- [CI/CD и DevOps](./cicd-devops.md)
- [Тестирование](./testing.md)
- [Event-Driven архитектура](./event-driven-fullstack.md) 