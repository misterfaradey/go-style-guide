# Инициализация ссылок на структуры

Используйте `&T{}` вместо `new(T)` при инициализации ссылок на структуры, чтобы это
соответствовало инициализации структуры.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// несоответствующий
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>
