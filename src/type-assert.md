# Обработать ошибки при подтверждении типа

Форма с единственным возвращаемым значением [утверждение типа] будет запаниковывать при вводе неправильного
типа. Поэтому всегда используйте идиому "запятая в порядке".

[утверждение типа]: https://go.dev/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->
