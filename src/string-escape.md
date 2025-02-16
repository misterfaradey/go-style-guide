# Используйте необработанные строковые литералы, чтобы избежать экранирования

Go поддерживает [необработанные строковые литералы](https://go.dev/ref/spec#raw_string_lit),
которые могут занимать несколько строк и включать кавычки. Используйте их, чтобы избежать
ручного экранирования строк, которые намного сложнее читать.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>
