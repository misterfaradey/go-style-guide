# Тестовые таблицы

Табличные тесты с [субтестами] могут быть полезным шаблоном для написания тестов
, позволяющим избежать дублирования кода, когда основная логика тестирования повторяется.

Если тестируемую систему необходимо протестировать в нескольких условиях, когда
определенные части входных и выходных данных изменяются, следует
использовать тест на основе таблиц, чтобы уменьшить избыточность и улучшить читаемость.

  [subtests]: https://go.dev/blog/subtests

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

Тестовые таблицы упрощают добавление контекста к сообщениям об ошибках, сокращают дублирование
логики и добавляют новые тестовые примеры.

Мы придерживаемся соглашения о том, что фрагмент структур называется "тестами"
, а каждый тестовый пример - `tt`. Кроме того, мы рекомендуем пояснять входные и выходные
значения для каждого тестового примера с помощью префиксов  `give` и `want`.

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

## Избегайте излишней сложности табличных тестов

Табличные тесты могут быть сложными для чтения и сопровождения, если субтесты содержат условные
утверждения или другую логику ветвления. Табличные тесты не следует использовать всякий
раз, когда внутри субтестов должна быть сложная или условная логика (т.е. сложная логика внутри цикла `for`).

Большие и сложные табличные тесты ухудшают читаемость и удобство сопровождения, поскольку у средств чтения тестов могут
возникнуть трудности с отладкой возникающих ошибок.

Подобные табличные тесты следует разделить либо на несколько тестовых таблиц, либо на несколько
отдельных функций "Test...".

Вот некоторые идеалы, к которым следует стремиться::

* Сосредоточьтесь на самой узкой единице поведения
* Сведите к минимуму "глубину тестирования" и избегайте условных утверждений (см. ниже)
* Убедитесь, что все поля таблицы используются во всех тестах
* Убедитесь, что вся логика тестирования выполняется для всех вариантов таблиц

В этом контексте "глубина тестирования" означает "в рамках данного теста количество
последовательных утверждений, для выполнения которых требуются предыдущие утверждения" (аналогично
цикломатической сложности).
Наличие "более мелких" тестов означает, что между
утверждениями меньше взаимосвязей и, что более важно, что эти утверждения с меньшей вероятностью
будут условными по умолчанию.

В частности, табличные тесты могут стать запутанными и трудными для чтения, если они используют несколько
путей ветвления (например, `shouldError`, `expectCall` и т.д.), используют множество инструкций `if` для
конкретных условных ожиданий (например, `shouldCallFoo`) или размещают функции внутри
таблицы (например, `setupMocks func(*FooMock)`).

Однако при тестировании поведения, которое
изменяется только в зависимости от измененных входных данных, может оказаться предпочтительнее сгруппировать похожие случаи
в табличный тест, чтобы лучше проиллюстрировать, как меняется поведение во всех входных данных,
а не разбивать сопоставимые в остальном блоки на отдельные тесты
и усложняет их сравнение и противопоставление.

Если текст теста короткий и понятный,
допустимо иметь единый путь ветвления для случаев успеха и неудачи
с полем таблицы, например "shouldErr", для указания ожидаемых ошибок.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func TestComplicatedTable(t *testing.T) {
  tests := []struct {
    give          string
    want          string
    wantErr       error
    shouldCallX   bool
    shouldCallY   bool
    giveXResponse string
    giveXErr      error
    giveYResponse string
    giveYErr      error
  }{
    // ...
  }

  for _, tt := range tests {
    t.Run(tt.give, func(t *testing.T) {
      // setup mocks
      ctrl := gomock.NewController(t)
      xMock := xmock.NewMockX(ctrl)
      if tt.shouldCallX {
        xMock.EXPECT().Call().Return(
          tt.giveXResponse, tt.giveXErr,
        )
      }
      yMock := ymock.NewMockY(ctrl)
      if tt.shouldCallY {
        yMock.EXPECT().Call().Return(
          tt.giveYResponse, tt.giveYErr,
        )
      }

      got, err := DoComplexThing(tt.give, xMock, yMock)

      // verify results
      if tt.wantErr != nil {
        require.EqualError(t, err, tt.wantErr)
        return
      }
      require.NoError(t, err)
      assert.Equal(t, want, got)
    })
  }
}
```

</td><td>

```go
func TestShouldCallX(t *testing.T) {
  // setup mocks
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)
  xMock.EXPECT().Call().Return("XResponse", nil)

  yMock := ymock.NewMockY(ctrl)

  got, err := DoComplexThing("inputX", xMock, yMock)

  require.NoError(t, err)
  assert.Equal(t, "want", got)
}

func TestShouldCallYAndFail(t *testing.T) {
  // setup mocks
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)

  yMock := ymock.NewMockY(ctrl)
  yMock.EXPECT().Call().Return("YResponse", nil)

  _, err := DoComplexThing("inputY", xMock, yMock)
  assert.EqualError(t, err, "Y failed")
}
```
</td></tr>
</tbody></table>

Такая сложность затрудняет изменение, понимание и доказательство
правильности теста.

Хотя строгих рекомендаций не существует,
при выборе между табличными тестами и отдельными тестами
для нескольких входов/выходов в систему всегда следует учитывать удобство чтения и сопровождения.

## Параллельные тесты

Параллельные тесты, такие как некоторые специализированные циклы (например, те, которые запускают
goroutines или фиксируют ссылки как часть тела цикла),
должны обеспечивать явное назначение переменных цикла в пределах области действия цикла, чтобы
гарантировать, что они содержат ожидаемые значения.

```go
tests := []struct{
  give string
  // ...
}{
  // ...
}

for _, tt := range tests {
  tt := tt // for t.Parallel
  t.Run(tt.give, func(t *testing.T) {
    t.Parallel()
    // ...
  })
}
```

В приведенном выше примере мы должны объявить переменную `tt`
, область действия которой ограничена итерацией цикла, из-за использования `t.t` приведена функция `Parallel()`.
Если мы этого не сделаем, большинство или все тесты получат неожиданное значение для
`tt` или значение, которое меняется по мере их выполнения.

<!-- TODO: Explain how to use _test packages. -->
