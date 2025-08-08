# Интерфейсы
Понимание когда и как реализовывать


Интерфейс - абстрактный тип, в котором методы обьявляются **публично**
и тоже **абстрактно**.
При этом тело метода не определенно, кроме имени и параметров.

Интерфейс может быть реализован только при **полном** соответствии методов.
> это сделано, в свою очередь, для безопасности и читаемости кода,
> а именно - снижение complexity(сложность) программного кода и его архитектуры!

Но тогда вопрос - **конкретно для чего** нужен интерфейс?

## Смысл интерфейсов

Приведу такой псевдо-код:

```cpp
class LogToFile {
  public function write(msg string) {
    //Write log in file...
  }
}

class LogToDB {
  public function write(msg string) {
    //Write log in DataBase...
  }

class Controller{
  public function _constructor(loggerFunc LogToFile) {
    this.logger = loggerFunc
  }
}
```

Для того, чтобы записать логи в базу данных - придется менять жестко закодированную ссылку в `_constructor(loggerFunc LogToFile)` на `_constructor(loggerFunc LogToDB)`.

А теперь представьте, что у нас очень много самых различных классов,
тогда, если надо будет их изменить, то менять придется все классы вручную.

Но если мы воспользуемся интерфейсом, то тогда:
```cpp

interface Logger {
    public function write(msg string);
}


class LogToFile {
  public function write(msg string) {
    //Write log in file...
  }
}

class LogToDB {
  public function write(msg string) {
    //Write log in DataBase...
  }

class Controller{
  public function _constructor(loggerFunc Logger) {
    this.logger = loggerFunc
  }
}


//Call procedures
controller1 = Controller(new LogToFile)
controller1.write("Writing in file")

controller2 = Controller(new LogToDB)
controller2.write("Writing in data base")
```

Теперь, благодаря интерфейсу, нам не нужно вручную менять код конструктора!
Достаточно лишь передать нужную функцию в конструктор.

## Ссылки
