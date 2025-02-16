# Проверка соответствия интерфейса

При необходимости проверьте соответствие интерфейса во время компиляции. Это включает в себя:

- Экспортированные типы, которые требуются для реализации определенных интерфейсов в рамках
  их контракта API
- Экспортированные или неэкспортированные типы, которые являются частью коллекции типов
  реализующих один и тот же интерфейс
- Другие случаи, когда нарушение интерфейса может привести к нарушению работы пользователей

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

Оператор `var _ http.Handler = (*Handler)(nil)` не удастся скомпилировать, если
`*Handler` когда-либо перестанет соответствовать интерфейсу `http.Handler`.

В правой части присваивания должно быть нулевое значение указанного
типа. Это значение равно "nil" для типов указателей (например, "*Handler"), срезов и отображений, а
также пустой struct для структурных типов.

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```
