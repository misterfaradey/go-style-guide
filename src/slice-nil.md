# nil - допустимый фрагмент

`nil` - допустимый фрагмент длиной 0. Это означает, что,

- Вы не должны возвращать фрагмент нулевой длины явно. Возвращайте `nil`
  вместо.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- Чтобы проверить, пуст ли фрагмент, всегда используйте `len(s) == 0`. Не проверяйте наличие
  `nil`.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- Нулевое значение (фрагмент, объявленный с помощью `var`) можно использовать немедленно без использования
  `make()`.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

Помните, что, хотя это допустимый фрагмент, нулевой фрагмент не эквивалентен
выделенному фрагменту длиной 0 - один из них равен нулю, а другой - нет, - и в разных
ситуациях (например, при сериализации) они могут обрабатываться по-разному.