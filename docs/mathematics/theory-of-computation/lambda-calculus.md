# Лямбда-исчисление

> **Формальная система для описания функций**  
> Основа функционального программирования и теории типов

[[Теория вычислений]] → **Лямбда-исчисление**

---

## 🎯 Ключевые концепции

### λ Основы лямбда-исчисления
- **Лямбда-абстракция** - определение функции `λx.M`
- **Применение** - вызов функции `M N`
- **Переменные** - свободные и связанные
- **Редукции** - вычислительные правила

### 🔗 Связи с другими темами
- **[[Теория типов]]** - типизированное лямбда-исчисление
- **[[Функциональное программирование]]** - практическая реализация идей
- **[[Формальные языки и грамматики]]** - синтаксическая структура

---

## 📝 Синтаксис и семантика

### Базовый синтаксис

```python
class LambdaExpression:
    """Представление лямбда-выражений"""
    
    def __init__(self, expr_type, **kwargs):
        self.type = expr_type
        
        if expr_type == 'variable':
            self.name = kwargs['name']
        elif expr_type == 'abstraction':
            self.variable = kwargs['variable']
            self.body = kwargs['body']
        elif expr_type == 'application':
            self.function = kwargs['function']
            self.argument = kwargs['argument']
    
    def __str__(self):
        """Строковое представление выражения"""
        if self.type == 'variable':
            return self.name
        elif self.type == 'abstraction':
            return f"λ{self.variable}.{self.body}"
        elif self.type == 'application':
            func_str = str(self.function)
            arg_str = str(self.argument)
            
            # Добавляем скобки при необходимости
            if self.function.type == 'abstraction':
                func_str = f"({func_str})"
            if self.argument.type == 'application':
                arg_str = f"({arg_str})"
                
            return f"{func_str} {arg_str}"
    
    def free_variables(self):
        """Множество свободных переменных"""
        if self.type == 'variable':
            return {self.name}
        elif self.type == 'abstraction':
            body_free = self.body.free_variables()
            return body_free - {self.variable}
        elif self.type == 'application':
            return self.function.free_variables() | self.argument.free_variables()
    
    def substitute(self, var, replacement):
        """Подстановка переменной"""
        if self.type == 'variable':
            return replacement if self.name == var else self
        elif self.type == 'abstraction':
            if self.variable == var:
                # Связанная переменная заслоняет подстановку
                return self
            elif self.variable in replacement.free_variables():
                # Избегаем захвата переменных - переименование
                new_var = self.fresh_variable(replacement.free_variables() | {var})
                renamed_body = self.body.substitute(self.variable, 
                                                  LambdaExpression('variable', name=new_var))
                new_body = renamed_body.substitute(var, replacement)
                return LambdaExpression('abstraction', variable=new_var, body=new_body)
            else:
                new_body = self.body.substitute(var, replacement)
                return LambdaExpression('abstraction', variable=self.variable, body=new_body)
        elif self.type == 'application':
            new_function = self.function.substitute(var, replacement)
            new_argument = self.argument.substitute(var, replacement)
            return LambdaExpression('application', function=new_function, argument=new_argument)
    
    def fresh_variable(self, used_vars, base='x'):
        """Генерация свежей переменной"""
        counter = 0
        candidate = base
        while candidate in used_vars:
            counter += 1
            candidate = f"{base}{counter}"
        return candidate

# Конструкторы для удобства
def Var(name):
    return LambdaExpression('variable', name=name)

def Lam(var, body):
    return LambdaExpression('abstraction', variable=var, body=body)

def App(func, arg):
    return LambdaExpression('application', function=func, argument=arg)

# Примеры базовых выражений
identity = Lam('x', Var('x'))  # λx.x
constant = Lam('x', Lam('y', Var('x')))  # λx.λy.x
application_example = App(identity, Var('a'))  # (λx.x) a

print(f"Тождественная функция: {identity}")
print(f"Константная функция: {constant}")
print(f"Применение: {application_example}")
print(f"Свободные переменные в применении: {application_example.free_variables()}")
```

### Вычислительные правила

```python
class LambdaCalculus:
    """Система вычислений в лямбда-исчислении"""
    
    def __init__(self):
        self.reduction_steps = []
    
    def alpha_conversion(self, expr, old_var, new_var):
        """α-конверсия: переименование связанных переменных"""
        if expr.type == 'abstraction' and expr.variable == old_var:
            new_body = expr.body.substitute(old_var, Var(new_var))
            return LambdaExpression('abstraction', variable=new_var, body=new_body)
        return expr
    
    def beta_reduction(self, expr, trace=False):
        """β-редукция: применение функции к аргументу"""
        if trace:
            self.reduction_steps = []
            
        def reduce_step(e):
            if trace:
                self.reduction_steps.append(str(e))
            
            if e.type == 'application':
                if e.function.type == 'abstraction':
                    # Основная β-редукция: (λx.M) N → M[x := N]
                    result = e.function.body.substitute(e.function.variable, e.argument)
                    if trace:
                        self.reduction_steps.append(f"→ {result}")
                    return reduce_step(result) if result != e else result
                else:
                    # Редуцируем функцию и аргумент
                    new_function = reduce_step(e.function)
                    new_argument = reduce_step(e.argument)
                    if new_function != e.function or new_argument != e.argument:
                        return reduce_step(LambdaExpression('application', 
                                                         function=new_function, 
                                                         argument=new_argument))
            elif e.type == 'abstraction':
                new_body = reduce_step(e.body)
                if new_body != e.body:
                    return LambdaExpression('abstraction', variable=e.variable, body=new_body)
            
            return e
        
        return reduce_step(expr)
    
    def eta_conversion(self, expr):
        """η-конверсия: λx.(f x) ↔ f (если x не свободна в f)"""
        if expr.type == 'abstraction':
            body = expr.body
            if (body.type == 'application' and 
                body.argument.type == 'variable' and
                body.argument.name == expr.variable and
                expr.variable not in body.function.free_variables()):
                return body.function
        return expr
    
    def normalize(self, expr, max_steps=100):
        """Нормализация выражения"""
        current = expr
        steps = 0
        
        while steps < max_steps:
            reduced = self.beta_reduction(current)
            if str(reduced) == str(current):
                break
            current = reduced
            steps += 1
        
        if steps >= max_steps:
            return current, "Возможное зацикливание"
        
        return current, f"Нормализовано за {steps} шагов"

# Демонстрация редукций
calc = LambdaCalculus()

# Пример: (λx.x) y → y
expr1 = App(Lam('x', Var('x')), Var('y'))
result1, info1 = calc.normalize(expr1)
print(f"\nПример 1: {expr1} → {result1}")

# Пример: (λx.λy.x) a b → a
expr2 = App(App(Lam('x', Lam('y', Var('x'))), Var('a')), Var('b'))
result2, info2 = calc.normalize(expr2)
print(f"Пример 2: {expr2} → {result2}")

# Трассировка редукции
result_traced = calc.beta_reduction(expr2, trace=True)
print(f"Трассировка: {' → '.join(calc.reduction_steps)}")
```

---

## 🔢 Кодирование данных

### Числа Чёрча

```python
class ChurchNumerals:
    """Кодирование натуральных чисел в лямбда-исчислении"""
    
    def __init__(self):
        self.calc = LambdaCalculus()
    
    def church_numeral(self, n):
        """Создание числа Чёрча для n"""
        # n = λf.λx.f^n(x) = λf.λx.f(f(...f(x)...))
        
        def apply_n_times(n, f_var, x_var):
            if n == 0:
                return Var(x_var)
            else:
                inner = apply_n_times(n - 1, f_var, x_var)
                return App(Var(f_var), inner)
        
        body = apply_n_times(n, 'f', 'x')
        return Lam('f', Lam('x', body))
    
    def church_add(self):
        """Функция сложения для чисел Чёрча"""
        # add = λm.λn.λf.λx.m f (n f x)
        return Lam('m', 
                  Lam('n', 
                     Lam('f', 
                        Lam('x', 
                           App(App(Var('m'), Var('f')),
                              App(App(Var('n'), Var('f')), Var('x')))))))
    
    def church_multiply(self):
        """Функция умножения для чисел Чёрча"""
        # mult = λm.λn.λf.m (n f)
        return Lam('m',
                  Lam('n',
                     Lam('f',
                        App(Var('m'), 
                           App(Var('n'), Var('f'))))))
    
    def church_successor(self):
        """Функция следующего числа"""
        # succ = λn.λf.λx.f (n f x)
        return Lam('n',
                  Lam('f',
                     Lam('x',
                        App(Var('f'),
                           App(App(Var('n'), Var('f')), Var('x'))))))
    
    def demonstrate_arithmetic(self):
        """Демонстрация арифметики Чёрча"""
        
        # Создаем числа
        zero = self.church_numeral(0)   # λf.λx.x
        one = self.church_numeral(1)    # λf.λx.f x
        two = self.church_numeral(2)    # λf.λx.f (f x)
        three = self.church_numeral(3)  # λf.λx.f (f (f x))
        
        # Операции
        add = self.church_add()
        mult = self.church_multiply()
        succ = self.church_successor()
        
        print("=== Числа Чёрча ===")
        print(f"0 = {zero}")
        print(f"1 = {one}")
        print(f"2 = {two}")
        print(f"3 = {three}")
        
        # Демонстрация сложения: 1 + 2 = 3
        addition_expr = App(App(add, one), two)
        sum_result, _ = self.calc.normalize(addition_expr, max_steps=50)
        print(f"\n1 + 2 = {sum_result}")
        
        # Демонстрация следующего числа: succ 2 = 3
        succ_expr = App(succ, two)
        succ_result, _ = self.calc.normalize(succ_expr, max_steps=50)
        print(f"succ 2 = {succ_result}")
        
        return {
            'zero': zero,
            'one': one,
            'two': two,
            'three': three,
            'add': add,
            'mult': mult,
            'succ': succ
        }

# Демонстрация чисел Чёрча
church = ChurchNumerals()
church_numbers = church.demonstrate_arithmetic()
```

### Булевы значения

```python
class ChurchBooleans:
    """Кодирование булевых значений и условных выражений"""
    
    def __init__(self):
        self.calc = LambdaCalculus()
    
    def church_true(self):
        """Истина = λx.λy.x (выбирает первый аргумент)"""
        return Lam('x', Lam('y', Var('x')))
    
    def church_false(self):
        """Ложь = λx.λy.y (выбирает второй аргумент)"""
        return Lam('x', Lam('y', Var('y')))
    
    def church_if(self):
        """Условное выражение = λp.λx.λy.p x y"""
        return Lam('p', Lam('x', Lam('y', App(App(Var('p'), Var('x')), Var('y')))))
    
    def church_and(self):
        """Логическое И = λp.λq.p q p"""
        return Lam('p', Lam('q', App(App(Var('p'), Var('q')), Var('p'))))
    
    def church_or(self):
        """Логическое ИЛИ = λp.λq.p p q"""
        return Lam('p', Lam('q', App(App(Var('p'), Var('p')), Var('q'))))
    
    def church_not(self):
        """Логическое НЕ = λp.λx.λy.p y x"""
        return Lam('p', Lam('x', Lam('y', App(App(Var('p'), Var('y')), Var('x')))))
    
    def demonstrate_boolean_logic(self):
        """Демонстрация булевой логики"""
        
        true_val = self.church_true()
        false_val = self.church_false()
        if_expr = self.church_if()
        and_op = self.church_and()
        or_op = self.church_or()
        not_op = self.church_not()
        
        print("\n=== Булевы значения Чёрча ===")
        print(f"TRUE = {true_val}")
        print(f"FALSE = {false_val}")
        
        # Демонстрация IF: IF TRUE a b = a
        if_true_expr = App(App(App(if_expr, true_val), Var('a')), Var('b'))
        if_true_result, _ = self.calc.normalize(if_true_expr)
        print(f"\nIF TRUE a b = {if_true_result}")
        
        # Демонстрация AND: TRUE AND FALSE = FALSE
        and_expr = App(App(and_op, true_val), false_val)
        and_result, _ = self.calc.normalize(and_expr)
        print(f"TRUE AND FALSE = {and_result}")
        
        # Демонстрация NOT: NOT TRUE = FALSE
        not_expr = App(not_op, true_val)
        not_result, _ = self.calc.normalize(not_expr)
        print(f"NOT TRUE = {not_result}")
        
        return {
            'true': true_val,
            'false': false_val,
            'if': if_expr,
            'and': and_op,
            'or': or_op,
            'not': not_op
        }

# Демонстрация булевой логики
booleans = ChurchBooleans()
boolean_ops = booleans.demonstrate_boolean_logic()
```

### Структуры данных

```python
class ChurchDataStructures:
    """Кодирование структур данных в лямбда-исчислении"""
    
    def __init__(self):
        self.calc = LambdaCalculus()
    
    def church_pair(self):
        """Пара = λx.λy.λf.f x y"""
        return Lam('x', Lam('y', Lam('f', App(App(Var('f'), Var('x')), Var('y')))))
    
    def church_first(self):
        """Первый элемент пары = λp.p (λx.λy.x)"""
        true_val = Lam('x', Lam('y', Var('x')))  # TRUE
        return Lam('p', App(Var('p'), true_val))
    
    def church_second(self):
        """Второй элемент пары = λp.p (λx.λy.y)"""
        false_val = Lam('x', Lam('y', Var('y')))  # FALSE
        return Lam('p', App(Var('p'), false_val))
    
    def church_list_nil(self):
        """Пустой список = λx.λy.y (FALSE)"""
        return Lam('x', Lam('y', Var('y')))
    
    def church_list_cons(self):
        """Добавление в начало списка = λh.λt.λf.f h t"""
        return Lam('h', Lam('t', Lam('f', App(App(Var('f'), Var('h')), Var('t')))))
    
    def church_list_head(self):
        """Голова списка = λl.l (λh.λt.h)"""
        get_head = Lam('h', Lam('t', Var('h')))
        return Lam('l', App(Var('l'), get_head))
    
    def church_list_tail(self):
        """Хвост списка = λl.l (λh.λt.t)"""
        get_tail = Lam('h', Lam('t', Var('t')))
        return Lam('l', App(Var('l'), get_tail))
    
    def church_list_isnil(self):
        """Проверка на пустоту = λl.l (λh.λt.FALSE) TRUE"""
        false_val = Lam('x', Lam('y', Var('y')))
        true_val = Lam('x', Lam('y', Var('x')))
        always_false = Lam('h', Lam('t', false_val))
        return Lam('l', App(App(Var('l'), always_false), true_val))
    
    def demonstrate_data_structures(self):
        """Демонстрация структур данных"""
        
        pair_fn = self.church_pair()
        first_fn = self.church_first()
        second_fn = self.church_second()
        
        nil = self.church_list_nil()
        cons = self.church_list_cons()
        head = self.church_list_head()
        tail = self.church_list_tail()
        isnil = self.church_list_isnil()
        
        print("\n=== Структуры данных Чёрча ===")
        
        # Демонстрация пар
        pair_expr = App(App(pair_fn, Var('a')), Var('b'))
        first_expr = App(first_fn, pair_expr)
        first_result, _ = self.calc.normalize(first_expr)
        print(f"FIRST (PAIR a b) = {first_result}")
        
        # Демонстрация списков: [a, b] = CONS a (CONS b NIL)
        list_b = App(App(cons, Var('b')), nil)
        list_ab = App(App(cons, Var('a')), list_b)
        
        head_expr = App(head, list_ab)
        head_result, _ = self.calc.normalize(head_expr)
        print(f"HEAD [a, b] = {head_result}")
        
        tail_expr = App(tail, list_ab)
        tail_result, _ = self.calc.normalize(tail_expr, max_steps=20)
        print(f"TAIL [a, b] = {tail_result}")
        
        # Проверка пустоты
        isnil_nil_expr = App(isnil, nil)
        isnil_nil_result, _ = self.calc.normalize(isnil_nil_expr)
        print(f"ISNIL [] = {isnil_nil_result}")
        
        isnil_list_expr = App(isnil, list_ab)
        isnil_list_result, _ = self.calc.normalize(isnil_list_expr)
        print(f"ISNIL [a, b] = {isnil_list_result}")

# Демонстрация структур данных
data_structures = ChurchDataStructures()
data_structures.demonstrate_data_structures()
```

---

## 🔄 Рекурсия и неподвижные точки

### Комбинатор неподвижной точки

```python
class FixedPointCombinators:
    """Комбинаторы неподвижной точки для рекурсии"""
    
    def __init__(self):
        self.calc = LambdaCalculus()
    
    def y_combinator(self):
        """Y-комбинатор = λf.(λx.f (x x)) (λx.f (x x))"""
        # Это самоприменяющийся комбинатор для неподвижной точки
        inner = Lam('x', App(Var('f'), App(Var('x'), Var('x'))))
        return Lam('f', App(inner, inner))
    
    def demonstrate_factorial(self):
        """Демонстрация факториала через Y-комбинатор"""
        
        y_comb = self.y_combinator()
        
        # Шаблон факториала: λfact.λn.IF (ISZERO n) 1 (MULT n (fact (PRED n)))
        # Упрощенная версия для демонстрации концепции
        
        # Предположим, что у нас есть примитивы ISZERO, MULT, PRED, 1
        # fact_template = λfact.λn.IF (ISZERO n) 1 (MULT n (fact (PRED n)))
        fact_template = Lam('fact', 
                           Lam('n', 
                              Var('IF_THEN_ELSE')))  # Упрощенно
        
        # Факториал = Y fact_template
        factorial = App(y_comb, fact_template)
        
        print("\n=== Y-комбинатор и рекурсия ===")
        print(f"Y = {y_comb}")
        print(f"FACTORIAL = Y FACT_TEMPLATE")
        print("Где FACT_TEMPLATE = λfact.λn.IF (ISZERO n) 1 (MULT n (fact (PRED n)))")
        
        # В реальной реализации это позволило бы вычислять факториал
        # factorial 3 → 6
        print("\nY-комбинатор позволяет определять рекурсивные функции")
        print("без явной рекурсии в самом лямбда-исчислении")
        
        return factorial
    
    def omega_combinator(self):
        """Ω-комбинатор = (λx.x x)(λx.x x) - расходящееся выражение"""
        self_app = Lam('x', App(Var('x'), Var('x')))
        return App(self_app, self_app)
    
    def demonstrate_divergence(self):
        """Демонстрация расходящегося вычисления"""
        
        omega = self.omega_combinator()
        
        print(f"\n=== Ω-комбинатор ===")
        print(f"Ω = {omega}")
        print("Это выражение не имеет нормальной формы:")
        print("(λx.x x)(λx.x x) → (λx.x x)(λx.x x) → ...")
        print("Демонстрирует, что не все выражения нормализуются")
        
        return omega

# Демонстрация неподвижных точек
fixed_points = FixedPointCombinators()
factorial = fixed_points.demonstrate_factorial()
omega = fixed_points.demonstrate_divergence()
```

---

## 🎯 Типизированное лямбда-исчисление

### Простые типы

```python
class SimpleTypes:
    """Система простых типов для лямбда-исчисления"""
    
    def __init__(self):
        self.type_variables = set()
        self.type_counter = 0
    
    def fresh_type_variable(self):
        """Генерация новой типовой переменной"""
        self.type_counter += 1
        var_name = f"τ{self.type_counter}"
        self.type_variables.add(var_name)
        return var_name
    
    def parse_type(self, type_str):
        """Парсинг типового выражения"""
        # Упрощенный парсер для демонстрации
        if '→' in type_str:
            parts = type_str.split('→', 1)
            return {
                'kind': 'function',
                'domain': self.parse_type(parts[0].strip()),
                'codomain': self.parse_type(parts[1].strip())
            }
        else:
            return {'kind': 'atomic', 'name': type_str.strip()}
    
    def type_to_string(self, type_expr):
        """Преобразование типа в строку"""
        if type_expr['kind'] == 'atomic':
            return type_expr['name']
        elif type_expr['kind'] == 'function':
            domain_str = self.type_to_string(type_expr['domain'])
            codomain_str = self.type_to_string(type_expr['codomain'])
            
            # Добавляем скобки для ассоциативности влево
            if type_expr['domain']['kind'] == 'function':
                domain_str = f"({domain_str})"
            
            return f"{domain_str} → {codomain_str}"
    
    def type_check(self, expr, context=None):
        """Проверка типов выражения"""
        if context is None:
            context = {}
        
        if expr.type == 'variable':
            if expr.name in context:
                return context[expr.name]
            else:
                # Свободная переменная - создаем типовую переменную
                fresh_type = self.fresh_type_variable()
                context[expr.name] = {'kind': 'atomic', 'name': fresh_type}
                return context[expr.name]
        
        elif expr.type == 'abstraction':
            # λx.M : τ₁ → τ₂, где x : τ₁ и M : τ₂
            param_type = {'kind': 'atomic', 'name': self.fresh_type_variable()}
            new_context = context.copy()
            new_context[expr.variable] = param_type
            
            body_type = self.type_check(expr.body, new_context)
            
            return {
                'kind': 'function',
                'domain': param_type,
                'codomain': body_type
            }
        
        elif expr.type == 'application':
            # M N : τ₂, где M : τ₁ → τ₂ и N : τ₁
            func_type = self.type_check(expr.function, context)
            arg_type = self.type_check(expr.argument, context)
            
            if func_type['kind'] != 'function':
                raise TypeError(f"Ожидался функциональный тип, получен {self.type_to_string(func_type)}")
            
            # Проверяем совместимость типов (упрощенно)
            if not self.types_compatible(func_type['domain'], arg_type):
                raise TypeError(f"Несовместимые типы: {self.type_to_string(func_type['domain'])} и {self.type_to_string(arg_type)}")
            
            return func_type['codomain']
    
    def types_compatible(self, type1, type2):
        """Проверка совместимости типов (упрощенная унификация)"""
        # В реальной системе здесь была бы полная унификация
        return self.type_to_string(type1) == self.type_to_string(type2)
    
    def demonstrate_typing(self):
        """Демонстрация типизации"""
        
        print("\n=== Простая система типов ===")
        
        # Примеры типизированных выражений
        examples = [
            (Lam('x', Var('x')), "Тождественная функция"),
            (Lam('x', Lam('y', Var('x'))), "Константная функция"),
            (Lam('f', Lam('x', App(Var('f'), Var('x')))), "Применение функции"),
        ]
        
        for expr, description in examples:
            try:
                expr_type = self.type_check(expr)
                type_str = self.type_to_string(expr_type)
                print(f"{description}: {expr} : {type_str}")
            except TypeError as e:
                print(f"{description}: Ошибка типизации - {e}")
        
        # Пример ошибки типизации
        try:
            # Попытка применить переменную к себе: x x
            bad_expr = App(Var('x'), Var('x'))
            bad_type = self.type_check(bad_expr)
            print(f"Самоприменение: {bad_expr} : {self.type_to_string(bad_type)}")
        except TypeError as e:
            print(f"Самоприменение: Ошибка типизации - {e}")

# Демонстрация типизации
types = SimpleTypes()
types.demonstrate_typing()
```

---

## 🔗 Связанные темы

### 📚 Теоретические основы
- **[[Теория типов]]** - расширенные системы типов
- **[[Теория категорий]]** - математические основы
- **[[Формальные языки и грамматики]]** - синтаксическая структура

### 🛠️ Практические применения
- **[[Функциональное программирование]]** - Haskell, ML, Lisp
- **[[Компиляторы]]** - анализ и оптимизация функциональных языков
- **[[Формальная верификация]]** - доказательство свойств программ

### 🧮 Расширения
- **Полиморфизм** - System F и высшие порядки
- **Зависимые типы** - Martin-Löf типы
- **Линейные типы** - управление ресурсами

---

## 🎓 Практические задания

### 📝 Начальный уровень
1. **Основы синтаксиса**
   - [ ] Определить свободные и связанные переменные
   - [ ] Выполнить α-конверсию
   - [ ] Провести β-редукцию вручную

2. **Кодирование данных**
   - [ ] Реализовать числа Чёрча 0-5
   - [ ] Определить операции сложения и умножения
   - [ ] Кодировать булевы значения и логические операции

### 🔧 Средний уровень
3. **Структуры данных**
   - [ ] Реализовать списки и операции над ними
   - [ ] Определить пары и проекции
   - [ ] Кодировать натуральные числа с предикатами

4. **Рекурсия**
   - [ ] Понять Y-комбинатор
   - [ ] Определить факториал через неподвижную точку
   - [ ] Исследовать расходящиеся вычисления

### 🚀 Продвинутый уровень
5. **Типизация**
   - [ ] Изучить простую типизацию
   - [ ] Понять полиморфные типы
   - [ ] Исследовать зависимые типы

---

*"Лямбда-исчисление доказывает, что вся математика может быть выражена через функции - элегантная основа современного программирования"* λ✨ 