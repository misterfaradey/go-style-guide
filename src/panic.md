# Не паникуйте

Код, запущенный в рабочей среде, должен избегать паники. Паника является основным источником
[каскадных сбоев]. При возникновении ошибки функция должна вернуть сообщение об ошибке и
позволить вызывающей стороне решить, как с этим справиться.

[каскадные сбои]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func run(args []string) {
  if len(args) == 0 {
    panic("an argument is required")
  }
  // ...
}

func main() {
  run(os.Args[1:])
}
```

</td><td>

```go
func run(args []string) error {
  if len(args) == 0 {
    return errors.New("an argument is required")
  }
  // ...
  return nil
}

func main() {
  if err := run(os.Args[1:]); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

</td></tr>
</tbody></table>

Panic/recovery не является стратегией обработки ошибок. Программа должна паниковать только тогда, когда
происходит что-то непоправимое, например, нулевое разыменование. Исключением из этого правила является
инициализация программы: сбои при запуске программы, которые должны привести к прерыванию работы
программы, могут вызвать панику.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Даже в тестах отдавайте предпочтение `t.Fatal` или `t.FailNow`, а не `panics`, чтобы убедиться, что
тест помечен как неудачный.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>
