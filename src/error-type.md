# Типы ошибок

Существует несколько вариантов объявления ошибок.
Прежде чем выбрать вариант, наиболее подходящий для вашего варианта использования, рассмотрите следующее.

- Должен ли вызывающий объект сопоставлять ошибку, чтобы он мог ее обработать?
  Если да, мы должны поддерживать ошибки [`.Is`] или [`ошибки.Как`] функции
  путем объявления переменной ошибки верхнего уровня или пользовательского типа.
- Является ли сообщение об ошибке статической строкой
  или динамической строкой, требующей контекстной информации?
  Для первого мы можем использовать [`errors.New`], но для второго мы должны
  использовать [`fmt.Errorf`] или пользовательский тип ошибки.
- Распространяем ли мы новую ошибку, возвращаемую нисходящей функцией?
  Если да, то смотрите [раздел, посвященный переносу ошибок](error-wrap.md).

[`errors.Is`]: https://pkg.go.dev/errors#Is
[`errors.As`]: https://pkg.go.dev/errors#As

| Error matching? | Error Message | Guidance                            |
|-----------------|---------------|-------------------------------------|
| No              | static        | [`errors.New`]                      |
| No              | dynamic       | [`fmt.Errorf`]                      |
| Yes             | static        | top-level `var` with [`errors.New`] |
| Yes             | dynamic       | пользовательский тип `error`        |

[`errors.New`]: https://pkg.go.dev/errors#New
[`fmt.Errorf`]: https://pkg.go.dev/fmt#Errorf

Например,
используйте [`errors.New`] для ошибки со статической строкой.
Экспортируйте эту ошибку в качестве переменной, чтобы поддерживать сопоставление с `errors.Is",
если вызывающей стороне необходимо сопоставить и обработать эту ошибку.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Для ошибки с динамической строкой
используйте [`fmt.Errorf`], если вызывающей стороне не нужно сопоставлять ее,
и пользовательскую "ошибку", если вызывающей стороне действительно нужно сопоставить ее.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  var notFound *NotFoundError
  if errors.As(err, &notFound) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Обратите внимание, что если вы экспортируете переменные или типы ошибок из пакета,
они станут частью общедоступного API пакета.
