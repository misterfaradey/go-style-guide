# Группировка и упорядочение функций

- Функции должны быть отсортированы в приблизительном порядке вызова.
- Функции в файле должны быть сгруппированы по получателю.

Поэтому экспортируемые функции должны отображаться в файле первыми, после
определений `struct`, `const`, `var`.

`new XYZ()`/`NewXYZ()` может появиться после определения типа, но перед
остальными методами в получателе.

Поскольку функции сгруппированы по получателю, простые служебные функции должны отображаться
ближе к концу файла.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>
