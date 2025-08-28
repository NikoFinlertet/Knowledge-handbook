# Абстрактные классы

Абстрактный класс - это класс, который не может быть реализован
и используется как базовый класс для создания других классов.

Он содержит абстрактные методы (минимум один обязан быть), которые должны быть реализованы в классах-наследниках.
Абстрактные классы могут содержать и обычные методы, которые уже реализованы и могут быть использованы в классах-наследниках.

## Смысл абстрактных классов

Например, вот псевдо-код **ДО** абстрактного класса:

```cpp
class Tea{
    public function addTea() {
        print("Adding tea leaves");
        return;
    }

    public function addSugar() {
        print("Adding sugar");
        return;
    }

    public function addMilk() {
        print("Adding milk");
        return;
    }

    public function make() {
        addTeaLeaves();
        addSugar();
        addMilk();
        return;
    }
};

class Coffee{
    public function addCoffee() {
        print("Adding coffee beans");
        return;
    }

    public function addSugar() {
        print("Adding sugar");
        return;
    }

    public function addMilk() {
        print("Adding milk");
        return;
    }

    public function make() {
        addCoffeeBeans();
        addSugar();
        addMilk();
        return;
    }
};

//Call procedures
tea = new Tea()
tea.make()

coffee = new Coffee()
coffee.make()
```

В приведенных классах у нас есть общие методы: `addSugar()`, `addMilk()` и `make()`.

Нам надо убрать повторяющийся код - это мы может сделать вот так:

```cpp
abstract class Template{
    public function addSugar() {
        print("Adding sugar");
        return;
    }

    public function addMilk() {
        print("Adding milk");
        return;
    }

    public function make() {
        addSugar();
        addMilk();
        addPrimaryTopings();
        return;
    }

    abstract public function addPrimaryTopings();
}

class Tea extends Template{
    public function addPrimaryTopings() {
        print("Adding tea leaves");
        return;
    }
}

class Coffee extends Template{
    public function addPrimaryTopings() {
        print("Adding coffee beans");
        return;
    }
}

//Call procedures
tea = new Tea()
tea.make()

coffee = new Coffee()
coffee.make()
```
Мы убрали общие методы в абстрактный класс `Template`
и создали абстрактный(~простите за тавтологию~) метод `addPrimaryTopings()`,
чтобы разделить реализацию поведения этого метода в каждом классе.


Таким образом, абстрактные классы нужны только для снижения complexity(сложность) кода!


## Ссылки
- [SOLID](./README.md)
