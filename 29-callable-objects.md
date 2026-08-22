---
title: Object có thể gọi
description: Tìm hiểu cách tạo và sử dụng object có thể gọi trong Dart.
showToc: false
prevpage:
  url: /language/extension-types
  title: Extension type
nextpage:
  url: /language/class-modifiers
  title: Class modifier
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore: (stable|beta|dev)[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore: (stable|beta|dev)[^\n]+\n/$1\n/g; /. • (lib|test)\/\w+\.dart:\d+:\d+//g"?>

Để cho phép một instance của lớp Dart được gọi như một function,
hãy implement method `call()`.

Method `call()` cho phép instance của bất kỳ lớp nào định nghĩa nó mô phỏng một function.
Method này hỗ trợ cùng chức năng như [function][functions] thông thường,
chẳng hạn parameter và return type.

Trong ví dụ sau, lớp `WannabeFunction` định nghĩa một function `call()`
nhận ba string và nối chúng lại, phân tách mỗi string bằng một khoảng trắng,
rồi thêm dấu chấm than. Nhấp vào **Run** để thực thi code.

<?code-excerpt "misc/lib/language_tour/callable_objects.dart"?>
```dartpad
class WannabeFunction {
  String call(String a, String b, String c) => '$a $b $c!';
}

var wf = WannabeFunction();
var out = wf('Hi', 'there,', 'gang');

void main() => print(out);
```

[functions]: /language/functions
