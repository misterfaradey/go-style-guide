# Используйте `time` для управления временем

Время - это сложная штука. Часто делаются следующие неверные предположения о времени
.

1. В сутках 24 часа
2. В часе 60 минут
3. В неделе 7 дней
4. В году 365 дней
5. [И многое другое](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

Например, *1* означает, что добавление 24 часов к моменту времени не всегда
приводит к появлению нового календарного дня.

Поэтому всегда используйте пакет [`"time"`] при работе со временем, поскольку он
помогает справиться с этими неверными предположениями более безопасным и точным способом.

[`"time"`]: https://pkg.go.dev/time

## Используйте `time.Time` для значений моментов времени

Используйте [`time.Time`] при работе с моментами времени и методы
"time.Время" при сравнении, сложении или вычитании времени.

[`time.Time`]: https://pkg.go.dev/time#Time

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

</td><td>

```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

</td></tr>
</tbody></table>

## Используйте `time.Продолжительность` для обозначения периодов времени

Используйте [`time.Длительность"], когда речь идет о периодах времени.

[`time.Продолжительность`]: https://pkg.go.dev/time#Duration

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // это были секунды или миллисекунды?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

Возвращаясь к примеру добавления 24 часов к моменту времени, метод, который мы
используем для добавления времени, зависит от цели. Если мы хотим получить то же время суток, но на
следующий календарный день, мы должны использовать [`Time.AddDate`]. Однако, если мы хотим
, чтобы момент времени гарантированно был на 24 часа позже предыдущего, мы должны
использовать [`Time.Add`].

  [`Time.AddDate`]: https://pkg.go.dev/time#Time.AddDate
  [`Time.Add`]: https://pkg.go.dev/time#Time.Add

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

## Используйте `time.Time` и `time.Duration` во внешних системах

По возможности используйте `time.Duration` и `time.Time` при взаимодействии с внешними системами
. Например:

- Флаги командной строки: [`flag`] поддерживает "time".Длительность" с помощью
  [`time.ParseDuration`]
- JSON: [`encoding/json`] поддерживает кодировку `time.Time` в виде строки [RFC 3339]
  с помощью метода [`UnmarshalJSON`]
- SQL: [`database/sql`] поддерживает преобразование столбцов `DATETIME` или `TIMESTAMP`
  в `time.Time` и обратно, если базовый драйвер поддерживает это
- YAML: [`gopkg.in/yaml.v2`] поддерживает `time.Time` в виде строки [RFC 3339], и
  `time.Duration` через [`time.ParseDuration`].

  [`flag`]: https://pkg.go.dev/flag
  [`time.ParseDuration`]: https://pkg.go.dev/time#ParseDuration
  [`encoding/json`]: https://pkg.go.dev/encoding/json
  [RFC 3339]: https://tools.ietf.org/html/rfc3339
  [`UnmarshalJSON` method]: https://pkg.go.dev/time#Time.UnmarshalJSON
  [`database/sql`]: https://pkg.go.dev/database/sql
  [`gopkg.in/yaml.v2`]: https://pkg.go.dev/gopkg.in/yaml.v2

Если использовать `time.Duration` невозможно в этих взаимодействиях, используйте
`int` или `float64` и укажите единицу измерения в названии поля.

Например, поскольку `encoding/json` не поддерживает `time.Duration`, единица
измерения включена в название поля.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

Когда использовать `time.Time` невозможно.При таких взаимодействиях, если
не согласован альтернативный вариант, используйте "string" и форматируйте временные метки, как определено в
[RFC 3339]. Этот формат по умолчанию используется [`Time.UnmarshalText`] и
доступен для использования в `Time.Format` и `time.Parse` с помощью [`time.RFC3339`].

[`Time.UnmarshalText`]: https://pkg.go.dev/time#Time.UnmarshalText
[`time.RFC3339`]: https://pkg.go.dev/time#RFC3339

Хотя на практике это, как правило, не является проблемой, имейте в виду, что
пакет `"time"` не поддерживает синтаксический анализ временных меток с использованием високосных секунд
([8728]), а также не учитывает високосные секунды в расчетах ([15190]). Если
вы сравниваете два момента времени, разница не будет включать в себя високосные
секунды, которые могли произойти между этими двумя моментами.

[8728]: https://github.com/golang/go/issues/8728
[15190]: https://github.com/golang/go/issues/15190
