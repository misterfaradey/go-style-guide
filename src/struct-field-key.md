# Используйте имена полей для инициализации структур

При инициализации структур почти всегда следует указывать имена полей.
Теперь это поддерживается [`go vet`].

[`go vet`]: https://pkg.go.dev/cmd/vet

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Исключение: Имена полей *могут* быть опущены в тестовых таблицах, если имеется 3 или
менее полей.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```
