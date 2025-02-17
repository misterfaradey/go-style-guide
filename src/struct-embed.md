# # Встраивание в структуры

Встроенные типы должны располагаться в верхней части списка полей
структуры, и должна быть пустая строка, отделяющая встроенные поля от обычных.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

Внедрение должно приносить ощутимую пользу, например, добавлять или расширять
функциональность семантически приемлемым способом. Это должно происходить с нулевым
негативным воздействием на пользователя (см. также: [Избегайте внедрения типов в общедоступные структуры](embed-public.md)).

Исключение: Мьютексы не должны встраиваться даже в неэкспортируемые типы. Смотрите также: [Допустимы мьютексы с нулевым значением](mutex-zero-value.md).

Встраивание ** не должно быть**:

- чисто косметическим или ориентированным на удобство.
- Усложняет создание или использование внешних типов.
- Влияет на нулевые значения внешних типов. Если внешний тип имеет полезное нулевое значение, он
  после внедрения внутреннего типа все равно должно быть полезное нулевое значение.
- Отображение несвязанных функций или полей из внешнего типа в качестве побочного эффекта
  внедрения внутреннего типа.
- Отображение неэкспортированных типов.
- Изменение семантики копирования внешних типов.
- Измените API внешнего типа или семантику типа.
- Внедрите неканоническую форму внутреннего типа.
- Предоставьте подробную информацию о реализации внешнего типа.
- Разрешите пользователям наблюдать за внутренними элементами типа или управлять ими.
- Измените общее поведение внутренних функций, обернув их таким образом, чтобы это
  могло приятно удивить пользователей.

Проще говоря, внедряйте сознательно и преднамеренно. Хороший тест на лакмусовую бумажку заключается в следующем: "будут
ли все эти экспортированные внутренние методы/поля добавлены непосредственно во внешний тип".;
если ответ "какой-то" или "нет", не вводите внутренний тип - используйте
вместо него поле.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type A struct {
    // Ошибка: функции A.Lock() и A.Unlock()
    // доступны в настоящее время, не предоставляют
    // функциональных преимуществ и позволяют
    // пользователям управлять подробностями о
    // внутреннем устройстве A.
    sync.Mutex
}
```

</td><td>

```go
type countingWriteCloser struct {
    // Хорошо: функция Write() предусмотрена на этом
    // внешнем уровне для определенной
    // цели и делегирует работу
    // функции Write() внутреннего типа.
    io.WriteCloser

    count int
}

func (w *countingWriteCloser) Write(bs []byte) (int, error) {
    w.count += len(bs)
    return w.WriteCloser.Write(bs)
}
```

</td></tr>
<tr><td>

```go
type Book struct {
    // Bad: указатель изменяет полезность нулевого значения
    io.ReadWriter

    // other fields
}

// later

var b Book
b.Read(...)  // panic: nil pointer
b.String()   // panic: nil pointer
b.Write(...) // panic: nil pointer
```

</td><td>

```go
type Book struct {
    // Good: имеет полезное нулевое значение
    bytes.Buffer

    // other fields
}

// later

var b Book
b.Read(...)  // ok
b.String()   // ok
b.Write(...) // ok
```

</td></tr>
<tr><td>

```go
type Client struct {
    sync.Mutex
    sync.WaitGroup
    bytes.Buffer
    url.URL
}
```

</td><td>

```go
type Client struct {
    mtx sync.Mutex
    wg  sync.WaitGroup
    buf bytes.Buffer
    url url.URL
}
```

</td></tr>
</tbody></table>
