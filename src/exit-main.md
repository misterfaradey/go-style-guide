# Завершите работу в главном меню

Для немедленного завершения работы в программах Go используйте [`os.Exit`] или [`log.Fatal*`]. (Паника
- не лучший способ выхода из программ, пожалуйста, [не паникуйте](panic.md).)
- 
  [`os.Exit`]: https://pkg.go.dev/os#Exit
  [`log.Fatal*`]: https://pkg.go.dev/log#Fatal

Вызовите одну из `os.Exit` или `log.Fatal*` **только в `main()`**. Все остальные
функции должны возвращать ошибки, сигнализирующие о сбое.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func main() {
  body := readFile(path)
  fmt.Println(body)
}

func readFile(path string) string {
  f, err := os.Open(path)
  if err != nil {
    log.Fatal(err)
  }

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  return string(b)
}
```

</td><td>

```go
func main() {
  body, err := readFile(path)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(body)
}

func readFile(path string) (string, error) {
  f, err := os.Open(path)
  if err != nil {
    return "", err
  }

  b, err := io.ReadAll(f)
  if err != nil {
    return "", err
  }

  return string(b), nil
}
```

</td></tr>
</tbody></table>

Обоснование: Программы с несколькими функциями, которые завершают работу, создают несколько проблем:

- Неочевидный поток управления: любая функция может завершить работу программы, поэтому становится
  трудно разобраться в потоке управления.
- Сложно протестировать: функция, которая завершает работу программы, также завершит тест
  , вызывающий ее. Это затрудняет тестирование функции и создает риск
  пропуска других тестов, которые еще не были запущены с помощью "go test".
- Пропущенная очистка: когда функция выходит из программы, она пропускает вызовы функций
  помещается в очередь с инструкциями `отложить`. Это увеличивает риск пропуска важных
  задач по очистке.
