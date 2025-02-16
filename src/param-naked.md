# Избегайте открытых параметров

Именованные параметры в вызовах функций могут ухудшить читаемость. Добавляйте комментарии в стиле Си
(`/* ... */`) для имен параметров, если их значение неочевидно.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

А еще лучше, замените обычные типы `bool` пользовательскими типами для более удобочитаемого и
типобезопасного кода. В будущем для этого параметра будет разрешено не только два состояния (true/false).

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady Status = iota + 1
  StatusDone
  // // Возможно, в будущем у нас будет статус InProgress.
)

func printInfo(name string, region Region, status Status)
```
