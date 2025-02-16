# Скопируйте срезы и карты на границах

Срезы и карты содержат указатели на базовые данные, поэтому будьте осторожны со сценариями
, когда их необходимо скопировать.

## Получение срезов и карт

Имейте в виду, что пользователи могут изменять карту или срез, полученные вами в качестве аргумента
, если вы сохраните ссылку на них.

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// Теперь мы можем изменять trips[0], не затрагивая d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

## Возвращаем фрагменты и карты

Аналогичным образом, будьте осторожны с пользовательскими изменениями карт или фрагментов, раскрывающими внутреннее
состояние.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// // Snapshot возвращает текущую статистику.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// моментальный снимок больше не защищен мьютексом, поэтому любой
// доступ к моментальному снимку зависит от скачков данных.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is теперь является копией.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>


