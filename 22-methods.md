---
title: Methods
description: Tìm hiểu về phương thức trong Dart.
prevpage:
  url: /language/primary-constructors
  title: Primary constructor
nextpage:
  url: /language/extend
  title: Mở rộng một lớp
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>

Phương thức là những hàm cung cấp hành vi cho một đối tượng.

<a id="instance-methods"></a>

## Phương thức instance

Phương thức instance trên đối tượng có thể truy cập biến instance và `this`.
Phương thức `distanceTo()` trong ví dụ sau là một phương thức
instance:

<?code-excerpt "misc/lib/language_tour/classes/point.dart (class-with-distance-to)" plaster="none"?>
```dart
import 'dart:math';

class Point {
  final double x;
  final double y;

  // Đặt các biến instance x và y
  // trước khi phần thân constructor chạy.
  Point(this.x, this.y);

  double distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }

}
```

<a id="operators"></a>

## Toán tử

Hầu hết toán tử là phương thức instance có tên đặc biệt.
Dart cho phép bạn định nghĩa toán tử với các tên sau:

|       |      |      |      |       |      |
|-------|------|------|------|-------|------|
| `<`   | `>`  | `<=` | `>=` | `==`  | `~`  |
| `-`   | `+`  | `/`  | `~/` | `*`   | `%`  |
| `\|`  | `ˆ`  | `&`  | `<<` | `>>>` | `>>` |
| `[]=` | `[]` |      |      |       |      |

{:.table}

:::note
Có thể bạn đã nhận thấy một số [toán tử][], chẳng hạn `!=`, không nằm trong
danh sách tên. Những toán tử này không phải phương thức instance.
Hành vi của chúng được tích hợp sẵn trong Dart.
:::

{%- comment %}
  Ghi chú nội bộ từ https://github.com/dart-lang/site-www/pull/2691#discussion_r506184100:
  - `??`, `&&` và `||` bị loại trừ vì chúng là các toán tử lazy/short-circuit
  - `!` có lẽ bị loại trừ vì lý do lịch sử
{% endcomment %}

Để khai báo một toán tử, hãy sử dụng identifier dựng sẵn
`operator`, sau đó là toán tử bạn đang định nghĩa.
Ví dụ sau định nghĩa phép cộng (`+`), phép trừ (`-`)
và tính bằng nhau (`==`) của vector:

<?code-excerpt "misc/lib/language_tour/classes/vector.dart"?>
```dart
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  @override
  bool operator ==(Object other) =>
      other is Vector && x == other.x && y == other.y;

  @override
  int get hashCode => Object.hash(x, y);
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```


<a id="getters-and-setters"></a>

## Getter và setter

Getter và setter là những phương thức đặc biệt cung cấp quyền truy cập đọc và ghi
đối với thuộc tính của đối tượng. Hãy nhớ rằng mỗi biến instance có
một getter ngầm định và thêm một setter nếu phù hợp. Bạn có thể tạo
thuộc tính bổ sung bằng cách triển khai getter và setter, sử dụng
các keyword `get` và `set`:

<?code-excerpt "misc/lib/language_tour/classes/rectangle.dart"?>
```dart highlightLines=8-12
/// Một hình chữ nhật trong hệ tọa độ màn hình,
/// trong đó gốc tọa độ `(0, 0)` nằm ở góc trên bên trái.
class Rectangle {
  double left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Định nghĩa hai thuộc tính được tính toán: right và bottom.
  double get right => left + width;
  set right(double value) => left = value - width;
  double get bottom => top + height;
  set bottom(double value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

Với getter và setter, bạn có thể bắt đầu bằng biến instance, sau đó
bao bọc chúng bằng phương thức mà không cần thay đổi code client.

:::note
Các toán tử như tăng (`++`) hoạt động theo cách mong đợi, dù
getter có được định nghĩa tường minh hay không. Để tránh mọi side effect
ngoài dự kiến, toán tử gọi getter chính xác một lần và lưu giá trị
vào biến tạm.
:::

<a id="abstract-methods"></a>

## Phương thức abstract

Phương thức instance, getter và setter có thể là abstract, định nghĩa một
interface nhưng để phần triển khai cho các lớp khác.
Phương thức abstract chỉ có thể tồn tại trong [lớp abstract][] hoặc [mixin][].

Để biến một phương thức thành abstract, hãy sử dụng dấu chấm phẩy (`;`) thay cho phần thân phương thức:

<?code-excerpt "misc/lib/language_tour/classes/doer.dart"?>
```dart
abstract class Doer {
  // Định nghĩa các biến instance và phương thức...

  void doSomething(); // Định nghĩa một phương thức abstract.
}

class EffectiveDoer extends Doer {
  void doSomething() {
    // Cung cấp phần triển khai, vì vậy phương thức ở đây không phải abstract...
  }
}
```

[toán tử]: /language/operators
[lớp abstract]: /language/class-modifiers#abstract
[mixin]: /language/mixins
