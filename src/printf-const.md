# Строки форматирования вне Printf

Если вы объявляете строки форматирования для функций в стиле Printf вне строкового
литерала, сделайте их значениями const.

Это помогает программе go vet выполнять статический анализ строки форматирования.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>
