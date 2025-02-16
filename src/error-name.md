# Присвоение имен ошибкам

Для значений ошибок, сохраненных в виде глобальных переменных,
используйте префикс "Err" или "ошибка" в зависимости от того, были ли они экспортированы.
Это руководство заменяет [Префикс неэкспортированных глобальных переменных на _](global-name.md).

```go
var (
  // Следующие две ошибки экспортированы,
  // чтобы пользователи этого пакета могли сопоставить их
  // с errors.Is.

  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  // Эта ошибка не экспортируется, потому что
  // мы не хотим делать ее частью нашего общедоступного API.
  // Мы все еще можем использовать ее внутри пакета
  // с помощью errors.Is.

  errNotFound = errors.New("not found")
)
```

Для пользовательских типов ошибок используйте вместо них суффикс `Error`.

```go
// Аналогично, эта ошибка экспортируется,
//чтобы пользователи этого пакета могли сопоставить ее
// с errors.As.

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

// И эта ошибка не экспортируется, потому что
// мы не хотим делать ее частью общедоступного API.
// Мы все еще можем использовать ее внутри пакета
// с помощью errors.As.

type resolveError struct {
  Path string
}

func (e *resolveError) Error() string {
  return fmt.Sprintf("resolve %q", e.Path)
}
```
