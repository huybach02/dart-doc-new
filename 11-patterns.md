---
title: Patterns
description: Tóm tắt về pattern trong Dart.
prevpage:
  url: /language/type-system
  title: Hệ thống kiểu
nextpage:
  url: /language/pattern-types
  title: Các kiểu pattern
---

:::version-note
Pattern yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 3.0.
:::

Pattern là một nhóm cú pháp trong ngôn ngữ Dart,
giống như câu lệnh và biểu thức.
Một pattern biểu diễn shape của một tập hợp giá trị mà
nó có thể so khớp với các giá trị thực tế.

Trang này mô tả:

- Pattern làm được gì.
- Pattern được phép xuất hiện ở đâu trong code Dart.
- Các trường hợp sử dụng phổ biến của pattern.

Để tìm hiểu về các loại pattern khác nhau,
hãy truy cập trang [các kiểu pattern][types].

<a id="what-patterns-do"></a>

## Pattern làm được gì

Nhìn chung, một pattern có thể **so khớp** một giá trị, **destructure** một giá trị hoặc cả hai,
tùy thuộc vào ngữ cảnh và shape của pattern.

Trước tiên, _pattern matching_ cho phép bạn kiểm tra một giá trị đã cho:

- Có một shape nhất định hay không.
- Có phải một hằng số nhất định hay không.
- Có bằng một giá trị khác hay không.
- Có một kiểu nhất định hay không.

Sau đó, _pattern destructuring_ cung cấp cú pháp khai báo thuận tiện để
chia giá trị đó thành các phần cấu thành. Trong quá trình này, cùng một pattern cũng có thể cho phép bạn
bind biến với một số hoặc tất cả những phần đó.

<a id="matching"></a>

### So khớp

Pattern luôn kiểm tra với một giá trị để xác định liệu giá trị có dạng
bạn mong đợi hay không. Nói cách khác, bạn đang kiểm tra liệu giá trị có _khớp_ với pattern hay không.

Điều gì tạo nên một kết quả khớp phụ thuộc vào [loại pattern][types] bạn đang sử dụng.
Ví dụ, constant pattern khớp nếu giá trị bằng hằng số của pattern:

<?code-excerpt "language/lib/patterns/switch.dart (constant-pattern)"?>
```dart
switch (number) {
  // Constant pattern khớp nếu 1 == number.
  case 1:
    print('one');
}
```

Nhiều pattern sử dụng subpattern, đôi khi tương ứng được gọi là pattern _outer_ và _inner_.
Pattern so khớp đệ quy trên các subpattern.
Ví dụ, từng field riêng lẻ của bất kỳ pattern [kiểu collection][] nào có thể là
[variable pattern][variable] hoặc [constant pattern][constant]:

<?code-excerpt "language/lib/patterns/switch.dart (list-pattern)"?>
```dart
const a = 'a';
const b = 'b';
switch (obj) {
  // List pattern [a, b] trước tiên khớp với obj nếu obj là list có hai field,
  // sau đó nếu các field khớp với constant subpattern 'a' và 'b'.
  case [a, b]:
    print('$a, $b');
}
```

Để bỏ qua các phần trong giá trị được so khớp, bạn có thể sử dụng [wildcard pattern][]
làm placeholder. Đối với list pattern, bạn có thể sử dụng [rest element][].

<a id="destructuring"></a>

### Destructuring

Khi một đối tượng và pattern khớp nhau, pattern có thể truy cập dữ liệu của đối tượng
và trích xuất dữ liệu theo từng phần. Nói cách khác, pattern _destructure_ đối tượng:

<?code-excerpt "language/lib/patterns/destructuring.dart (list-pattern)"?>
```dart
var numList = [1, 2, 3];
// List pattern [a, b, c] destructure ba phần tử từ numList...
var [a, b, c] = numList;
// ...và gán chúng cho các biến mới.
print(a + b + c);
```

Bạn có thể lồng [bất kỳ loại pattern nào][types] bên trong một destructuring pattern.
Ví dụ, case pattern này so khớp và destructure một list hai phần tử
có phần tử đầu tiên là `'a'` hoặc `'b'`:

<?code-excerpt "language/lib/patterns/destructuring.dart (nested-pattern)"?>
```dart
switch (list) {
  case ['a' || 'b', var c]:
    print(c);
}
```

<a id="places-patterns-can-appear"></a>

## Những nơi pattern có thể xuất hiện

Bạn có thể sử dụng pattern ở một số vị trí trong ngôn ngữ Dart:

<a id="pattern-uses"></a>

- [Khai báo](#variable-declaration) và [phép gán](#variable-assignment) biến cục bộ
- [Vòng lặp for và for-in][for]
- [if-case][if] và [switch-case][switch]
- Control flow trong [collection literal][]

Phần này mô tả các trường hợp sử dụng phổ biến để so khớp và destructure bằng pattern.

<a id="variable-declaration"></a>

### Khai báo biến

Bạn có thể sử dụng _khai báo biến pattern_ ở bất cứ nơi nào Dart cho phép khai báo biến cục bộ.
Pattern so khớp với giá trị ở bên phải khai báo.
Sau khi khớp, pattern destructure giá trị và bind nó với các biến cục bộ mới:

<?code-excerpt "language/lib/patterns/destructuring.dart (variable-declaration)"?>
```dart
// Khai báo các biến mới a, b và c.
var (a, [b, c]) = ('str', [1, 2]);
```

Khai báo biến pattern phải bắt đầu bằng `var` hoặc `final`, theo sau
bởi một pattern.

<a id="variable-assignment"></a>

### Phép gán biến

_Pattern gán biến_ nằm ở bên trái một phép gán.
Trước tiên, nó destructure đối tượng khớp. Sau đó, nó gán các giá trị cho
các biến _hiện có_ thay vì bind biến mới.

Sử dụng pattern gán biến để hoán đổi giá trị của hai biến mà không cần
khai báo biến tạm thứ ba:

<?code-excerpt "language/lib/patterns/destructuring.dart (variable-assignment)"?>
```dart
var (a, b) = ('left', 'right');
(b, a) = (a, b); // Hoán đổi.
print('$a $b'); // In ra "right left".
```

<a id="switch-statements-and-expressions"></a>

### Câu lệnh và biểu thức switch

Mỗi mệnh đề case chứa một pattern. Điều này áp dụng cho [câu lệnh switch][switch]
và [biểu thức][], cũng như [câu lệnh if-case][if].
Bạn có thể sử dụng [bất kỳ loại pattern nào][types] trong một case.

_Case pattern_ là [refutable][].
Chúng cho phép control flow:
- So khớp và destructure đối tượng đang được switch.
- Tiếp tục thực thi nếu đối tượng không khớp.

Những giá trị mà pattern destructure trong case sẽ trở thành biến cục bộ.
Scope của chúng chỉ nằm trong phần thân case đó.

<?code-excerpt "language/lib/patterns/switch.dart (switch-statement)"?>
```dart
switch (obj) {
  // Khớp nếu 1 == obj.
  case 1:
    print('one');

  // Khớp nếu giá trị của obj nằm giữa
  // các giá trị hằng 'first' và 'last'.
  case >= first && <= last:
    print('in range');

  // Khớp nếu obj là record có hai field,
  // sau đó gán các field cho 'a' và 'b'.
  case (var a, var b):
    print('a = $a, b = $b');

  default:
}
```

<a id="or-pattern-switch"></a>

[Pattern logical-or][logical-or] hữu ích khi để nhiều case dùng chung
phần thân trong biểu thức hoặc câu lệnh switch:

<?code-excerpt "language/lib/patterns/switch.dart (or-share-body)"?>
```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false,
};
```

Câu lệnh switch có thể cho phép nhiều case dùng chung phần thân
[mà không sử dụng pattern logical-or][share], nhưng pattern này vẫn
hữu ích riêng trong việc cho phép nhiều case dùng chung một [guard][]:

<?code-excerpt "language/lib/patterns/switch.dart (or-share-guard)"?>
```dart
switch (shape) {
  case Square(size: var s) || Circle(size: var s) when s > 0:
    print('Non-empty symmetric shape');
}
```

[Mệnh đề guard][guard] đánh giá một điều kiện bất kỳ như một phần của case mà không
thoát khỏi switch nếu điều kiện là false
(điều sẽ xảy ra nếu dùng câu lệnh `if` trong phần thân case).

<?code-excerpt "language/lib/control_flow/branches.dart (guard)"?>
```dart
switch (pair) {
  case (int a, int b):
    if (a > b) print('First element greater');
  // Nếu false, không in gì và thoát khỏi switch.
  case (int a, int b) when a > b:
    // Nếu false, không in gì nhưng chuyển sang case tiếp theo.
    print('First element greater');
  case (int a, int b):
    print('First element not greater');
}
```

<a id="for-and-for-in-loops"></a>

### Vòng lặp for và for-in

Bạn có thể sử dụng pattern trong [vòng lặp for và for-in][for] để lặp qua và destructure
các giá trị trong collection.

Ví dụ này sử dụng [object destructuring][object] trong vòng lặp for-in để destructure
các đối tượng [`MapEntry`][] mà lời gọi `<Map>.entries` trả về:

<?code-excerpt "language/lib/patterns/for_in.dart (for-in-pattern)"?>
```dart
Map<String, int> hist = {'a': 23, 'b': 100};

for (var MapEntry(key: key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
```

Object pattern kiểm tra rằng `hist.entries` có kiểu được đặt tên `MapEntry`,
sau đó đệ quy vào các subpattern field được đặt tên `key` và `value`.
Trong mỗi lần lặp, nó gọi getter `key` và getter `value` trên `MapEntry`,
rồi lần lượt bind kết quả với các biến cục bộ `key` và `count`.

Bind kết quả của lời gọi getter với một biến cùng tên là trường hợp sử dụng phổ biến,
vì vậy object pattern cũng có thể suy luận tên getter từ
[variable subpattern][variable]. Điều này cho phép bạn đơn giản hóa variable pattern
từ dạng dư thừa như `key: key` thành chỉ `:key`:

<?code-excerpt "language/lib/patterns/for_in.dart (for-in-short)"?>
```dart
for (var MapEntry(:key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
```

<a id="use-cases-for-patterns"></a>

## Các trường hợp sử dụng pattern

[Phần trước](#places-patterns-can-appear)
mô tả _cách_ pattern phù hợp với các cấu trúc code Dart khác.
Bạn đã thấy một số trường hợp sử dụng thú vị làm ví dụ, chẳng hạn như [hoán đổi](#variable-assignment)
giá trị của hai biến hoặc
[destructure cặp key-value](#for-and-for-in-loops)
trong map. Phần này mô tả thêm nhiều trường hợp sử dụng, trả lời:

- _Khi nào và tại sao_ bạn có thể muốn sử dụng pattern.
- Chúng giải quyết những loại vấn đề nào.
- Chúng phù hợp nhất với những idiom nào.

<a id="destructuring-multiple-returns"></a>

### Destructure nhiều giá trị trả về

Record cho phép tổng hợp và [trả về nhiều giá trị][] từ một
lời gọi hàm duy nhất. Pattern bổ sung khả năng destructure các field của record
trực tiếp vào biến cục bộ, inline với lời gọi hàm.

Thay vì khai báo riêng từng biến cục bộ mới cho mỗi field record
như sau:

<?code-excerpt "language/lib/patterns/destructuring.dart (destructure-multiple-returns-1)"?>
```dart
var info = userInfo(json);
var name = info.$1;
var age = info.$2;
```

Bạn có thể destructure các field của record do một hàm trả về vào biến cục bộ
bằng [khai báo biến](#variable-declaration) hoặc
[pattern gán](#variable-assignment), với một [record pattern][record]
làm subpattern:

<?code-excerpt "language/lib/patterns/destructuring.dart (destructure-multiple-returns-2)"?>
```dart
var (name, age) = userInfo(json);
```

Để destructure một record có field được đặt tên bằng pattern:

<?code-excerpt "language/lib/patterns/destructuring.dart (destructure-multiple-returns-3)"?>
```dart
final (:name, :age) =
    getData(); // Ví dụ, trả về (name: 'doug', age: 25);
```

<a id="destructuring-class-instances"></a>

### Destructure instance lớp

[Object pattern][object] so khớp với các kiểu đối tượng được đặt tên, cho phép
bạn destructure dữ liệu bằng những getter mà lớp của đối tượng đã cung cấp.

Để destructure một instance của lớp, hãy sử dụng kiểu được đặt tên,
theo sau bởi các thuộc tính cần destructure đặt trong dấu ngoặc đơn:

<?code-excerpt "language/lib/patterns/destructuring.dart (destructure-class-instances)"?>
```dart
final Foo myFoo = Foo(one: 'one', two: 2);
var Foo(:one, :two) = myFoo;
print('one $one, two $two');
```

<a id="algebraic-data-types"></a>

### Kiểu dữ liệu đại số

Object destructuring và switch case rất phù hợp để viết
code theo phong cách [kiểu dữ liệu đại số][].
Sử dụng phương pháp này khi:
- Bạn có một họ các kiểu liên quan.
- Bạn có một thao tác cần hành vi cụ thể cho từng kiểu.
- Bạn muốn nhóm hành vi đó ở một nơi thay vì phân tán nó trên tất cả
các định nghĩa kiểu khác nhau.

Thay vì triển khai thao tác dưới dạng phương thức instance cho mọi kiểu,
hãy giữ các biến thể của thao tác trong một hàm duy nhất switch trên các subtype:

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

<a id="validating-incoming-json"></a>

### Xác thực JSON nhận vào

[Map][] và [list][] pattern rất phù hợp để destructure các cặp key-value trong
dữ liệu đã deserialize, chẳng hạn như dữ liệu được phân tích từ JSON:

<?code-excerpt "language/lib/patterns/json.dart (json-1)"?>
```dart
var data = {
  'user': ['Lily', 13],
};
var {'user': [name, age]} = data;
```

Nếu bạn biết dữ liệu JSON có cấu trúc như mong đợi,
ví dụ trên là thực tế.
Nhưng dữ liệu thường đến từ nguồn bên ngoài, chẳng hạn qua mạng.
Bạn cần xác thực trước để xác nhận cấu trúc của dữ liệu.

Nếu không có pattern, việc xác thực sẽ dài dòng:

<?code-excerpt "language/lib/patterns/json.dart (json-2)"?>
```dart
if (data is Map<String, Object?> &&
    data.length == 1 &&
    data.containsKey('user')) {
  var user = data['user'];
  if (user is List<Object> &&
      user.length == 2 &&
      user[0] is String &&
      user[1] is int) {
    var name = user[0] as String;
    var age = user[1] as int;
    print('User $name is $age years old.');
  }
}
```

Một [case pattern](#switch-statements-and-expressions) duy nhất
có thể thực hiện cùng một thao tác xác thực.
Case đơn hoạt động tốt nhất dưới dạng câu lệnh [if-case][if].
Pattern cung cấp phương pháp khai báo rõ ràng hơn và ít dài dòng hơn nhiều
để xác thực JSON:

<?code-excerpt "language/lib/patterns/json.dart (json-3)"?>
```dart
if (data case {'user': [String name, int age]}) {
  print('User $name is $age years old.');
}
```

Case pattern này đồng thời xác thực rằng:

- `json` là map vì trước tiên nó phải khớp với [map pattern][map] outer để tiếp tục.
  - Và vì là map, nó cũng xác nhận `json` không phải null.
- `json` chứa key `user`.
- Key `user` ghép với một list gồm hai giá trị.
- Kiểu của các giá trị list là `String` và `int`.
- Các biến cục bộ mới dùng để giữ giá trị là `name` và `age`.


[phiên bản ngôn ngữ]: /language/versioning
[types]: /language/pattern-types
[kiểu collection]: /language/collections
[wildcard pattern]: /language/pattern-types#wildcard
[rest element]: /language/pattern-types#rest-element
[null-check pattern]: /language/pattern-types#null-check
[for]: /language/loops#for-loops
[if]: /language/branches#if-case
[switch]: /language/branches#switch-statements
[biểu thức]: /language/branches#switch-expressions
[collection literal]: /language/collections#control-flow-operators
[null-assert pattern]: /language/pattern-types#null-assert
[record]: /language/pattern-types#record
[trả về nhiều giá trị]: /language/records#multiple-returns
[refutable]: /resources/glossary#refutable-pattern
[constant]: /language/pattern-types#constant
[list]: /language/pattern-types#list
[map]: /language/pattern-types#map
[variable]: /language/pattern-types#variable
[logical-or]: /language/pattern-types#logical-or
[share]: /language/branches#switch-share
[guard]: /language/branches#guard-clause
[relational]: /language/pattern-types#relational
[check]: /language/pattern-types#null-check
[assert]: /language/pattern-types#null-assert
[object]: /language/pattern-types#object
[`MapEntry`]: {{site.dart-api}}/dart-core/MapEntry-class.html
[kiểu dữ liệu đại số]: https://en.wikipedia.org/wiki/Algebraic_data_type
