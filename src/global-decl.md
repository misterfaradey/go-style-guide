# Объявления переменных верхнего уровня

На верхнем уровне используйте стандартное ключевое слово `var`. Не указывайте тип,
если только он не совпадает с типом выражения.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Поскольку F уже указывает, что возвращает строку, 
// нам не нужно снова указывать тип.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Укажите тип, если тип выражения не совсем соответствует требуемому типу.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F возвращает объект типа MyError, но нам нужна ошибка.
```
