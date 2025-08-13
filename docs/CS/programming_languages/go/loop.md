# Цикл

В Go только один вид цикла - `for`, который универсален и позволяет создавать в том числе while и do-while.

## Структура темы:
- [Синтаксис](##syntax)
- [Обьявление](##declaration)
- [Ссылки](##links)


## Синтаксис{##syntax}
```go
for инициализация; условие; итерация {
    // тело цикла
}

```

## Обьявление{##declaration}
```go

//Классический
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

//аналог while(true)
for {
	if (end_of_loop != nil) {
		break
	}
}

//аналог for-each
// Каждую итерацию - копия элементов из range
for index, value := range array {
	fmt.Println(index, value)
}

```

### Операторы
- break: прерывает цикл полностью
- continue: пропускает текущую итерацию и переходит к следующей

## Ссылки{##links}
- [Go](./README.md)
