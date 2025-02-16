# Добавляйте к неэкспортируемым глобальным символам префикс _

Добавляйте к неэкспортируемым символам верхнего уровня "var" и "const" префикс "_", чтобы при
их использовании было ясно, что они являются глобальными символами.

Обоснование: Переменные и константы верхнего уровня имеют область действия пакета. Использование
общего имени позволяет легко случайно использовать неправильное значение в другом
файле.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

**Исключение**: В неэкспортированных значениях ошибок может использоваться префикс `err` без подчеркивания.
Смотрите [Наименование ошибки](error-name.md).
