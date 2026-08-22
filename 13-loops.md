---
title: Loops
description: Tìm hiểu cách sử dụng vòng lặp để kiểm soát luồng code Dart.
prevpage:
  url: /language/pattern-types
  title: Các kiểu pattern
nextpage:
  url: /language/branches
  title: Rẽ nhánh
---

Trang này trình bày cách bạn có thể kiểm soát luồng code Dart bằng vòng lặp và
các câu lệnh hỗ trợ:

-   Vòng lặp `for`
-   Vòng lặp `while` và `do while`
-   `break` và `continue`

Bạn cũng có thể thao tác control flow trong Dart bằng:

- [Rẽ nhánh][], chẳng hạn như `if` và `switch`
- [Exception][], chẳng hạn như `try`, `catch` và `throw`

<a id="for-loops"></a>

## Vòng lặp for

Bạn có thể lặp bằng vòng lặp `for` tiêu chuẩn. Ví dụ:

<?code-excerpt "language/test/control_flow/loops_test.dart (for)"?>
```dart
var message = StringBuffer('Dart is fun');
for (var i = 0; i < 5; i++) {
  message.write('!');
}
```

Closure bên trong vòng lặp `for` của Dart capture _giá trị_ của index.
Điều này tránh một cạm bẫy phổ biến trong JavaScript. Ví dụ, hãy xem:

<?code-excerpt "language/test/control_flow/loops_test.dart (for-and-closures)"?>
```dart
var callbacks = [];
for (var i = 0; i < 2; i++) {
  callbacks.add(() => print(i));
}

for (final c in callbacks) {
  c();
}
```

Kết quả đầu ra là `0`, sau đó là `1`, đúng như mong đợi. Ngược lại, ví dụ này
sẽ in `2`, sau đó là `2` trong JavaScript.

Đôi khi bạn có thể không cần biết bộ đếm vòng lặp hiện tại
khi lặp qua một kiểu [`Iterable`][], chẳng hạn như `List` hoặc `Set`.
Trong trường hợp đó, hãy sử dụng vòng lặp `for-in` để code gọn hơn:

<?code-excerpt "language/lib/control_flow/loops.dart (collection)"?>
```dart
for (var candidate in candidates) {
  candidate.interview();
}
```

Trong vòng lặp ở ví dụ trên, `candidate` được
định nghĩa bên trong phần thân vòng lặp và
lần lượt được đặt để tham chiếu đến từng giá trị trong `candidates`.
`candidate` là một [biến][] cục bộ.
Việc gán lại `candidate` bên trong phần thân vòng lặp chỉ
thay đổi biến cục bộ của lần lặp đó và
không sửa đổi iterable `candidates` ban đầu.

Để xử lý các giá trị thu được từ iterable,
bạn cũng có thể sử dụng một [pattern][] trong vòng lặp `for-in`:

<?code-excerpt "language/lib/control_flow/loops.dart (collection-for-pattern)"?>
```dart
for (final Candidate(:name, :yearsExperience) in candidates) {
  print('$name has $yearsExperience of experience.');
}
```

:::tip
Để thực hành sử dụng `for-in`, hãy làm theo
[hướng dẫn về collection Iterable](/libraries/collections/iterables).
:::

Các lớp Iterable cũng có phương thức [forEach()][] như một lựa chọn khác:

<?code-excerpt "language/test/control_flow/loops_test.dart (for-each)"?>
```dart
var collection = [1, 2, 3];
collection.forEach(print); // 1 2 3
```

[biến]: /language/variables

<a id="while-and-do-while"></a>

## While và do-while

Vòng lặp `while` đánh giá điều kiện trước vòng lặp:

<?code-excerpt "language/lib/control_flow/loops.dart (while)"?>
```dart
while (!isDone()) {
  doSomething();
}
```

Vòng lặp `do`-`while` đánh giá điều kiện *sau* vòng lặp:

<?code-excerpt "language/lib/control_flow/loops.dart (do-while)"?>
```dart
do {
  printLine();
} while (!atEndOfPage());
```


<a id="break-and-continue"></a>

## Break và continue

Sử dụng `break` để dừng lặp:

<?code-excerpt "language/lib/control_flow/loops.dart (while-break)"?>
```dart
while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}
```

Sử dụng `continue` để chuyển sang lần lặp tiếp theo:

<?code-excerpt "language/lib/control_flow/loops.dart (for-continue)"?>
```dart
for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}
```

Nếu bạn đang sử dụng một [`Iterable`][] như list hoặc set,
cách viết ví dụ trên có thể khác:

<?code-excerpt "language/lib/control_flow/loops.dart (where)"?>
```dart
candidates
    .where((c) => c.yearsExperience >= 5)
    .forEach((c) => c.interview());
```

<a id="labels"></a>

## Label

Label là một identifier theo sau bởi dấu hai chấm (`labelName:`)
mà bạn có thể đặt trước một câu lệnh để tạo
_câu lệnh có label_. Vòng lặp và switch case thường được sử dụng làm
câu lệnh có label. Một câu lệnh có label có thể được tham chiếu sau đó
trong câu lệnh `break` hoặc `continue` như sau:

* `break labelName;`
  Chấm dứt việc thực thi câu lệnh có label.
  Điều này hữu ích để thoát khỏi một vòng lặp ngoài cụ thể khi bạn đang
  ở bên trong vòng lặp lồng nhau.

* `continue labelName;`
  Bỏ qua phần còn lại của lần lặp hiện tại trong
  vòng lặp có label và tiếp tục với lần lặp tiếp theo.

Label được sử dụng để quản lý control flow. Chúng thường được dùng với
vòng lặp và switch case, đồng thời cho phép bạn chỉ định câu lệnh cần
thoát hoặc tiếp tục, thay vì mặc định tác động đến
vòng lặp trong cùng.

<a id="labels-in-for-loop-using-break"></a>

### Label trong vòng lặp for sử dụng `break` {:.no_toc}

Đoạn code sau minh họa cách sử dụng label tên `outerLoop`
trong vòng lặp `for` với câu lệnh `break`:

<?code-excerpt "language/lib/control_flow/loops.dart (label-for-loop-break)"?>
```dart
outerLoop:
for (var i = 1; i <= 3; i++) {
  for (var j = 1; j <= 3; j++) {
    print('i = $i, j = $j');
    if (i == 2 && j == 2) {
      break outerLoop;
    }
  }
}
print('outerLoop exited');
```

Trong ví dụ trên, khi `i == 2` và `j == 2`, câu lệnh `break outerLoop;`
dừng cả vòng lặp trong lẫn ngoài. Kết quả đầu ra là:

```plaintext
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 2, j = 1
i = 2, j = 2
outerLoop exited
```

<a id="labels-in-for-loop-using-continue"></a>

### Label trong vòng lặp for sử dụng `continue` {:.no_toc}

Đoạn code sau minh họa cách sử dụng label tên `outerLoop`
trong vòng lặp `for` với câu lệnh `continue`:

<?code-excerpt "language/lib/control_flow/loops.dart (label-for-loop-continue)"?>
```dart
outerLoop:
for (var i = 1; i <= 3; i++) {
  for (var j = 1; j <= 3; j++) {
    if (i == 2 && j == 2) {
      continue outerLoop;
    }
    print('i = $i, j = $j');
  }
}
```

Trong ví dụ trên, khi `i == 2` và `j == 2`, `continue outerLoop;` bỏ qua
những lần lặp còn lại của `i = 2` và chuyển sang `i = 3`. Kết quả đầu ra là:

```plaintext
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 2, j = 1
i = 3, j = 1
i = 3, j = 2
i = 3, j = 3
```

<a id="labels-in-while-loop-using-break"></a>

### Label trong vòng lặp while sử dụng `break` {:.no_toc}

Đoạn code sau minh họa cách sử dụng label tên `outerLoop` trong
vòng lặp `while` với câu lệnh `break`:

<?code-excerpt "language/lib/control_flow/loops.dart (label-while-loop-break)"?>
```dart
var i = 1;

outerLoop:
while (i <= 3) {
  var j = 1;
  while (j <= 3) {
    print('i = $i, j = $j');
    if (i == 2 && j == 2) {
      break outerLoop;
    }
    j++;
  }
  i++;
}
print('outerLoop exited');
```

Trong ví dụ trên, chương trình thoát khỏi cả vòng lặp `while` trong lẫn ngoài
khi `i == 2` và `j == 2`. Kết quả đầu ra là:

```plaintext
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 2, j = 1
i = 2, j = 2
outerLoop exited
```

<a id="labels-in-while-loop-using-continue"></a>

### Label trong vòng lặp while sử dụng `continue` {:.no_toc}

Đoạn code sau minh họa cách sử dụng label tên `outerLoop` trong
vòng lặp `while` với câu lệnh `continue`:

<?code-excerpt "language/lib/control_flow/loops.dart (label-while-loop-continue)"?>
```dart
var i = 1;

outerLoop:
while (i <= 3) {
  var j = 1;
  while (j <= 3) {
    if (i == 2 && j == 2) {
      i++;
      continue outerLoop;
    }
    print('i = $i, j = $j');
    j++;
  }
  i++;
}
```

Trong ví dụ trên, lần lặp với `i = 2` và `j = 2` bị bỏ qua, rồi vòng lặp chuyển
thẳng sang `i = 3`. Kết quả đầu ra là:

```plaintext
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 2, j = 1
i = 3, j = 1
i = 3, j = 2
i = 3, j = 3
```

<a id="labels-in-do-while-loop-using-break"></a>

### Label trong vòng lặp do-while sử dụng `break` {:.no_toc}

Đoạn code sau minh họa cách sử dụng label tên `outerLoop` trong
vòng lặp `do while` với câu lệnh `break`:

<?code-excerpt "language/lib/control_flow/loops.dart (label-do-while-loop-break)"?>
```dart
var i = 1;

outerLoop:
do {
  var j = 1;
  do {
    print('i = $i, j = $j');
    if (i == 2 && j == 2) {
      break outerLoop;
    }
    j++;
  } while (j <= 3);
  i++;
} while (i <= 3);

print('outerLoop exited');
```

Trong ví dụ trên, chương trình thoát khỏi cả vòng lặp trong lẫn ngoài khi `i == 2` và
`j == 2`. Kết quả đầu ra là:

```plaintext
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 2, j = 1
i = 2, j = 2
outerLoop exited
```

<a id="labels-in-do-while-loop-using-continue"></a>

### Label trong vòng lặp do-while sử dụng `continue` {:.no_toc}

Đoạn code sau minh họa cách sử dụng label tên `outerLoop` trong
vòng lặp `do while` với câu lệnh `continue`:

<?code-excerpt "language/lib/control_flow/loops.dart (label-do-while-loop-continue)"?>
```dart
var i = 1;

outerLoop:
do {
  var j = 1;
  do {
    if (i == 2 && j == 2) {
      i++;
      continue outerLoop;
    }
    print('i = $i, j = $j');
    j++;
  } while (j <= 3);
  i++;
} while (i <= 3);
```

Trong ví dụ trên, vòng lặp bỏ qua `i = 2` và `j = 2`, rồi chuyển thẳng sang `i = 3`.
Kết quả đầu ra là:

```plaintext
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
i = 2, j = 1
i = 3, j = 1
i = 3, j = 2
i = 3, j = 3
```

[Exception]: /language/error-handling
[Rẽ nhánh]: /language/branches
[iteration]: /libraries/dart-core#iteration
[forEach()]: {{site.dart-api}}/dart-core/Iterable/forEach.html
[`Iterable`]: {{site.dart-api}}/dart-core/Iterable-class.html
[pattern]: /language/patterns
