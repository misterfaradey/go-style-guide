# Допустимы мьютексы с нулевым значением

Допустимо нулевое значение `sync.Mutex` и `sync.RWMutex`, поэтому вам почти
никогда не понадобится указатель на мьютекс.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

Если вы используете структуру по указателю, то мьютекс должен быть полем без указателя на
него. Не вставляйте мьютекс в структуру, даже если структура не экспортируется.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type SMap struct {
  sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

<tr><td>

Поле `Mutex`, а также методы "Lock" и "Unlock" непреднамеренно являются частью
экспортируемого API `SMap`.

</td><td>

Мьютекс и его методы - это детали реализации SMap, скрытые от
вызывающих его объектов.

</td></tr>
</tbody></table>
