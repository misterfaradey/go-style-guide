# Функциональные параметры

Функциональные параметры - это шаблон, в котором вы объявляете непрозрачный тип "Option"
, который записывает информацию в некоторую внутреннюю структуру. Вы принимаете различное количество
этих параметров и действуете на основе полной информации, записанной параметрами во
внутренней структуре.

Используйте этот шаблон для необязательных аргументов в конструкторах и других общедоступных API
, которые, как вы предполагаете, потребуется расширить, особенно если у вас уже есть три или
более аргумента для этих функций.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Open(
  addr string,
  cache bool,
  logger *zap.Logger
) (*Connection, error) {
  // ...
}
```

</td><td>

```go
// package db

type Option interface {
  // ...
}

func WithCache(c bool) Option {
  // ...
}

func WithLogger(log *zap.Logger) Option {
  // ...
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  // ...
}
```

</td></tr>
<tr><td>

Параметры кэша и регистратора должны быть указаны всегда, даже если пользователь
хочет использовать параметры по умолчанию.

```go
db.Open(addr, db.DefaultCache, zap.NewNop())
db.Open(addr, db.DefaultCache, log)
db.Open(addr, false /* cache */, zap.NewNop())
db.Open(addr, false /* cache */, log)
```

</td><td>

Опции предоставляются только в случае необходимости.

```go
db.Open(addr)
db.Open(addr, db.WithLogger(log))
db.Open(addr, db.WithCache(false))
db.Open(
  addr,
  db.WithCache(false),
  db.WithLogger(log),
)
```

</td></tr>
</tbody></table>

Мы предлагаем способ реализации этого шаблона с помощью интерфейса `Option`,
который содержит неэкспортированный метод, записывающий параметры в неэкспортированную структуру `options`.

```go
type options struct {
  cache  bool
  logger *zap.Logger
}

type Option interface {
  apply(*options)
}

type cacheOption bool

func (c cacheOption) apply(opts *options) {
  opts.cache = bool(c)
}

func WithCache(c bool) Option {
  return cacheOption(c)
}

type loggerOption struct {
  Log *zap.Logger
}

func (l loggerOption) apply(opts *options) {
  opts.logger = l.Log
}

func WithLogger(log *zap.Logger) Option {
  return loggerOption{Log: log}
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    cache:  defaultCache,
    logger: zap.NewNop(),
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}
```

Обратите внимание, что существует способ реализации этого шаблона с помощью замыканий, но мы
считаем, что приведенный выше шаблон обеспечивает большую гибкость для авторов и
проще в отладке и тестировании для пользователей. В частности, это позволяет
сравнивать параметры друг с другом в тестах и макетах, а также при закрытии, когда это
невозможно. Кроме того, это позволяет options реализовывать другие интерфейсы, включая
`fmt.Stringer`, который позволяет создавать понятные пользователю строковые представления
параметров.

Смотрите также,

- [Самореферентные функции и дизайн опций]
- [Функциональные возможности для дружественных API-интерфейсов]

  [Самореферентные функции и разработка опций]:  https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Функциональные возможности для дружественных API-интерфейсов]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->
