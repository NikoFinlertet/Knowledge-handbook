# Производная функции
Пусть функция F от n переменных:
$$
F(x_1,x_2,...,x_n)
$$

Тогда частная производная
$$ 
\frac{\partial F}{\partial x_i}
$$
показывает как изменится значение функции при изменении аргумента

## Aлгоритм вычисления частной производной в точке

$$
\begin{align}
\frac{\partial F(x)}{\partial x_i} = \frac{F(x + e_i*h) - F(x-e_i*h)}{2*h}\\
x\ -\ точка(вектор\ размерности\ n)\ в\ которой\ ищем\ производную \\
e_i\ -\ единичный\ вектор,\ направленный\ вдоль\ оси\ x_i\\
h\ -\ изменение\ аргумента\ функции
\end{align}
$$

### Пример на Python
```python
import numpy as np
from typing import Callable

H = 0.000_001

def f(x : np.array) -> float:
    """функция параболоида для теста"""
    return x[0] * x[0] + x[1] * x[1]

def der(func : Callable, x : np.array, e : np.array, h=H) -> float:
    """частная производная функции от n переменных в точке x"""
    delta = e * h
    return (func(x + delta) - func(x - delta)) / (2 * h)

if __name__ == '__main__':
    #проверяем экстремум (0,0) dF/dx == 0
    point = np.array([0.0,0.0])
    e = np.array([1.0,0.0])
    print(der(f, point, e))

    #проверяем точку (0,1) dF/dy == 2
    point = np.array([0.0, 1.0])
    e = np.array([0.0, 1.0])
    print(der(f, point, e))
```
