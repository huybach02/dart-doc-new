---
title: Branches
description: Tìm hiểu cách sử dụng nhánh để kiểm soát luồng code Dart.
prevpage:
  url: /language/loops
  title: Vòng lặp
nextpage:
  url: /language/error-handling
  title: Xử lý lỗi
---

Trang này trình bày cách bạn có thể kiểm soát luồng code Dart bằng các nhánh:

- Câu lệnh và phần tử `if`
- Câu lệnh và phần tử `if-case`
- Câu lệnh và biểu thức `switch`

Bạn cũng có thể thao tác control flow trong Dart bằng:

- [Vòng lặp][], chẳng hạn như `for` và `while`
- [Exception][], chẳng hạn như `try`, `catch` và `throw`

<a id="if"></a>

## If

Dart hỗ trợ câu lệnh `if` với mệnh đề `else` tùy chọn.
Điều kiện trong dấu ngoặc đơn sau `if` phải là
một biểu thức được đánh giá thành [boolean][]:

<?code-excerpt "language/lib/control_flow/branches.dart (if-else)"?>
```dart
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

Để tìm hiểu cách sử dụng `if` trong ngữ cảnh biểu thức,
hãy xem [Biểu thức điều kiện][].

<a id="if-case"></a>

### If-case

Câu lệnh `if` trong Dart hỗ trợ mệnh đề `case` theo sau bởi một [pattern][]:

<?code-excerpt "language/lib/control_flow/branches.dart (if-case)"?>
```dart
if (pair case [int x, int y]) return Point(x, y);
```

Nếu pattern khớp với giá trị,
nhánh sẽ thực thi với mọi biến mà pattern định nghĩa nằm trong scope.

Trong ví dụ trên,
list pattern `[int x, int y]` khớp với giá trị `pair`,
vì vậy nhánh `return Point(x, y)` thực thi với các biến
do pattern định nghĩa là `x` và `y`.

Nếu không, control flow sẽ chuyển đến nhánh `else`
để thực thi nếu có:

<?code-excerpt "language/lib/control_flow/branches.dart (if-case-else)"?>
```dart
if (pair case [int x, int y]) {
  print('Was coordinate array $x,$y');
} else {
  throw FormatException('Invalid coordinates.');
}
```

Câu lệnh if-case cung cấp một cách để so khớp và
[destructure][] với _một_ pattern duy nhất.
Để kiểm tra một giá trị với _nhiều_ pattern, hãy sử dụng [switch](#switch).

:::version-note
Mệnh đề case trong câu lệnh if yêu cầu
[phiên bản ngôn ngữ][] tối thiểu là 3.0.
:::

<a id="switch"></a>
<a id="switch-statements"></a>

## Câu lệnh switch

Câu lệnh `switch` đánh giá một biểu thức giá trị với một chuỗi case.
Mỗi mệnh đề `case` là một [pattern][] để giá trị so khớp.
Bạn có thể sử dụng [bất kỳ loại pattern nào][] cho một case.

Khi giá trị khớp với pattern của một case, phần thân case sẽ thực thi.
Mệnh đề `case` không rỗng nhảy đến cuối switch sau khi hoàn tất.
Chúng không cần câu lệnh `break`.
Những cách hợp lệ khác để kết thúc một mệnh đề `case` không rỗng là câu lệnh
[`continue`][break], [`throw`][] hoặc [`return`][].

Sử dụng mệnh đề `default` hoặc [wildcard `_`][] để
thực thi code khi không mệnh đề `case` nào khớp:

<?code-excerpt "language/lib/control_flow/branches.dart (switch)"?>
```dart
var command = 'OPEN';
switch (command) {
  case 'CLOSED':
    executeClosed();
  case 'PENDING':
    executePending();
  case 'APPROVED':
    executeApproved();
  case 'DENIED':
    executeDenied();
  case 'OPEN':
    executeOpen();
  default:
    executeUnknown();
}
```

<a id="switch-share"></a>

Case rỗng fall through đến case tiếp theo, cho phép các case dùng chung phần thân.
Đối với một case rỗng không fall through,
hãy dùng [`break`][break] làm phần thân.
Đối với fall-through không tuần tự,
bạn có thể sử dụng [câu lệnh `continue`][break] và một label:

<?code-excerpt "language/lib/control_flow/branches.dart (switch-empty)"?>
```dart
switch (command) {
  case 'OPEN':
    executeOpen();
    continue newCase; // Tiếp tục thực thi tại label newCase.

  case 'DENIED': // Case rỗng fall through.
  case 'CLOSED':
    executeClosed(); // Chạy cho cả DENIED và CLOSED.

  newCase:
  case 'PENDING':
    executeNowClosed(); // Chạy cho cả OPEN và PENDING.
}
```

Bạn có thể sử dụng [pattern logical-or][] để cho phép các case dùng chung phần thân hoặc guard.
Để tìm hiểu thêm về pattern và mệnh đề case,
hãy xem tài liệu pattern về [Câu lệnh và biểu thức switch][].

[Câu lệnh và biểu thức switch]: /language/patterns#switch-statements-and-expressions

<a id="switch-expressions"></a>

### Biểu thức switch

Một _biểu thức_ `switch` tạo ra giá trị dựa trên phần thân biểu thức
của bất kỳ case nào khớp.
Bạn có thể sử dụng biểu thức switch ở bất cứ nơi nào Dart cho phép biểu thức,
_ngoại trừ_ ở đầu một câu lệnh biểu thức. Ví dụ:

```dart
var x = switch (y) { ... };

print(switch (x) { ... });

return switch (x) { ... };
```

Nếu muốn sử dụng switch ở đầu một câu lệnh biểu thức,
hãy dùng [câu lệnh switch](#switch-statements).

Biểu thức switch cho phép bạn viết lại một _câu lệnh_ switch như sau:

<?code-excerpt "language/lib/control_flow/branches.dart (switch-stmt)"?>
```dart
// Trong đó slash, star, comma, semicolon, v.v. là các biến hằng...
switch (charCode) {
  case slash || star || plus || minus: // Pattern logical-or
    token = operator(charCode);
  case comma || semicolon: // Pattern logical-or
    token = punctuation(charCode);
  case >= digit0 && <= digit9: // Pattern quan hệ và logical-and
    token = number();
  default:
    throw FormatException('Invalid');
}
```

Thành một _biểu thức_ như sau:

<?code-excerpt "language/lib/control_flow/branches.dart (switch-exp)"?>
```dart
token = switch (charCode) {
  slash || star || plus || minus => operator(charCode),
  comma || semicolon => punctuation(charCode),
  >= digit0 && <= digit9 => number(),
  _ => throw FormatException('Invalid'),
};
```

Cú pháp của biểu thức `switch` khác với cú pháp câu lệnh `switch`:

- Case _không_ bắt đầu bằng keyword `case`.
- Phần thân case là một biểu thức duy nhất thay vì một chuỗi câu lệnh.
- Mỗi case phải có phần thân; không có fallthrough ngầm đối với case rỗng.
- Pattern của case được phân tách với phần thân bằng `=>` thay vì `:`.
- Các case được phân tách bằng `,` (và cho phép dấu `,` cuối tùy chọn).
- Case mặc định _chỉ_ có thể dùng `_`, thay vì cho phép cả `default` lẫn `_`.

:::version-note
Biểu thức switch yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 3.0.
:::

<a id="exhaustiveness-checking"></a>

### Kiểm tra tính đầy đủ

Kiểm tra tính đầy đủ là một tính năng báo cáo
lỗi tại thời điểm biên dịch nếu một giá trị có thể đi vào switch nhưng
không khớp với bất kỳ case nào.

<?code-excerpt "language/lib/control_flow/branches.dart (exh-bool)"?>
```dart
// Switch không đầy đủ trên bool?, thiếu case để khớp khả năng null:
switch (nullableBool) {
  case true:
    print('yes');
  case false:
    print('no');
}
```

Case mặc định (`default` hoặc `_`) bao quát mọi giá trị khả dĩ
có thể đi qua switch.
Điều này làm cho switch trên bất kỳ kiểu nào cũng đầy đủ.

[Enum][enum] và [kiểu sealed][sealed] đặc biệt hữu ích cho
switch vì ngay cả khi không có case mặc định,
các giá trị khả dĩ của chúng vẫn được biết và có thể liệt kê đầy đủ.
Sử dụng [modifier `sealed`][sealed] trên một lớp để bật
kiểm tra tính đầy đủ khi switch trên các subtype của lớp đó:

<?code-excerpt "language/lib/patterns/algebraic_datatypes.dart (algebraic-datatypes)"?>
```dart
sealed class Shape {}

class Square implements Shape {
  final double length;
  Square(this.length);
}

class Circle implements Shape {
  final double radius;
  Circle(this.radius);
}

double calculateArea(Shape shape) => switch (shape) {
  Square(length: var l) => l * l,
  Circle(radius: var r) => math.pi * r * r,
};
```

Nếu ai đó thêm một subclass mới của `Shape`,
biểu thức `switch` này sẽ không đầy đủ.
Kiểm tra tính đầy đủ sẽ thông báo cho bạn về subtype còn thiếu.
Điều này cho phép bạn sử dụng Dart phần nào theo
[phong cách kiểu dữ liệu đại số hàm](https://en.wikipedia.org/wiki/Algebraic_data_type).

<a id="when"></a>
<a id="guard-clause"></a>

## Mệnh đề guard

Để đặt một mệnh đề guard tùy chọn sau mệnh đề `case`, hãy sử dụng keyword `when`.
Mệnh đề guard có thể theo sau `if case`,
cũng như cả câu lệnh lẫn biểu thức `switch`.

```dart
// Câu lệnh switch:
switch (something) {
  case somePattern when some || boolean || expression:
    //             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Mệnh đề guard.
    body;
}

// Biểu thức switch:
var value = switch (something) {
  somePattern when some || boolean || expression => body,
  //               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Mệnh đề guard.
};

// Câu lệnh if-case:
if (something case somePattern when some || boolean || expression) {
  //                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Mệnh đề guard.
  body;
}
```

Guard đánh giá một biểu thức boolean bất kỳ _sau_ khi so khớp.
Điều này cho phép bạn thêm các ràng buộc khác về
việc phần thân case có nên thực thi hay không.
Khi mệnh đề guard được đánh giá thành false,
quá trình thực thi chuyển sang case tiếp theo thay vì
thoát khỏi toàn bộ switch.

[phiên bản ngôn ngữ]: /language/versioning
[Vòng lặp]: /language/loops
[Exception]: /language/error-handling
[Biểu thức điều kiện]: /language/operators#conditional-expressions
[boolean]: /language/built-in-types#booleans
[pattern]: /language/patterns
[enum]: /language/enums
[`throw`]: /language/error-handling#throw
[`return`]: /language/functions#return-values
[wildcard `_`]: /language/pattern-types#wildcard
[break]: /language/loops#break-and-continue
[sealed]: /language/class-modifiers#sealed
[bất kỳ loại pattern nào]: /language/pattern-types
[destructure]: /language/patterns#destructuring
[phần về switch]: /language/patterns#switch-statements-and-expressions
[pattern logical-or]: /language/patterns#or-pattern-switch
