# Начинайте перечисления с единицы

Стандартный способ введения перечислений в Go - это объявить пользовательский тип
и группу `const` с помощью `iota`. Поскольку переменные по умолчанию имеют значение 0,
обычно вы должны начинать перечисления с ненулевого значения.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

Есть случаи, когда использование нулевого значения имеет смысл, например, когда
нулевое значение является желательным поведением по умолчанию.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->
