---
title: Lớp
description: Tổng quan về lớp, các instance của lớp và các member của chúng.
prevpage:
  url: /language/error-handling
  title: Xử lý lỗi
nextpage:
  url: /language/constructors
  title: Constructor
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

Dart là một ngôn ngữ hướng đối tượng với các lớp và cơ chế kế thừa dựa trên mixin.
Mọi object đều là một instance của một lớp, và tất cả các lớp ngoại trừ `Null`
đều kế thừa từ [`Object`][].
*Kế thừa dựa trên mixin* có nghĩa là mặc dù mọi lớp
(ngoại trừ [lớp trên cùng][top-and-bottom], `Object?`)
chỉ có đúng một superclass, phần thân của một lớp có thể được tái sử dụng trong
nhiều hệ thống phân cấp lớp.
[Extension method][] là một cách để
thêm chức năng vào một lớp mà không thay đổi lớp đó hoặc tạo subclass.
[Class modifier][] cho phép bạn kiểm soát cách các library có thể tạo subtype của một lớp.


<a id="using-class-members"></a>
## Sử dụng các member của lớp

Object có các *member* bao gồm function và dữ liệu (tương ứng là *method* và
*instance variable*). Khi gọi một method, bạn *invoke* method đó
trên một object: method có quyền truy cập các function và dữ liệu của object đó.

Dùng dấu chấm (`.`) để tham chiếu đến một instance variable hoặc method:

<?code-excerpt "misc/test/language_tour/classes_test.dart (object-members)"?>
```dart
var p = Point(2, 2);

// Lấy giá trị của y.
assert(p.y == 2);

// Gọi distanceTo() trên p.
double distance = p.distanceTo(Point(4, 4));
```

Dùng `?.` thay cho `.` để tránh exception
khi toán hạng ngoài cùng bên trái là null:

<?code-excerpt "misc/test/language_tour/classes_test.dart (safe-member-access)"?>
```dart
// Nếu p khác null, đặt một biến bằng giá trị y của nó.
var a = p?.y;
```


<a id="using-constructors"></a>
## Sử dụng constructor

Bạn có thể tạo một object bằng *constructor*.
Tên constructor có thể là <code><em>ClassName</em></code> hoặc
<code><em>ClassName</em>.<em>identifier</em></code>. Ví dụ,
đoạn code sau tạo các object `Point` bằng các constructor
`Point()` và `Point.fromJson()`:

<?code-excerpt "misc/test/language_tour/classes_test.dart (object-creation)" replace="/ as .*?;/;/g"?>
```dart
var p1 = Point(2, 2);
var p2 = Point.fromJson({'x': 1, 'y': 2});
```

Đoạn code sau có tác dụng tương tự, nhưng
dùng keyword `new` không bắt buộc trước tên constructor:

<?code-excerpt "misc/test/language_tour/classes_test.dart (object-creation-new)" replace="/ as .*?;/;/g"?>
```dart
var p1 = new Point(2, 2);
var p2 = new Point.fromJson({'x': 1, 'y': 2});
```

Một số lớp cung cấp [constant constructor][].
Để tạo một hằng số tại compile time bằng constant constructor,
hãy đặt keyword `const` trước tên constructor:

<?code-excerpt "misc/test/language_tour/classes_test.dart (const)"?>
```dart
var p = const ImmutablePoint(2, 2);
```

Việc dựng hai hằng số compile-time giống hệt nhau sẽ tạo ra một
instance canonical duy nhất:

<?code-excerpt "misc/test/language_tour/classes_test.dart (identical)"?>
```dart
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);

assert(identical(a, b)); // Chúng là cùng một instance!
```

Trong một _constant context_, bạn có thể bỏ `const` trước constructor
hoặc literal. Ví dụ, hãy xem đoạn code tạo một const map sau:

<?code-excerpt "misc/test/language_tour/classes_test.dart (const-context-withconst)" replace="/pointAndLine1/pointAndLine/g"?>
```dart
// Có rất nhiều keyword const ở đây.
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```

Bạn có thể bỏ tất cả các lần dùng keyword `const` ngoại trừ lần đầu tiên:

<?code-excerpt "misc/test/language_tour/classes_test.dart (const-context-noconst)" replace="/pointAndLine2/pointAndLine/g"?>
```dart
// Chỉ một const, thiết lập constant context.
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

Nếu một constant constructor nằm ngoài constant context
và được gọi mà không có `const`,
nó sẽ tạo một **object không phải hằng số**:

<?code-excerpt "misc/test/language_tour/classes_test.dart (nonconst-const-constructor)"?>
```dart
var a = const ImmutablePoint(1, 1); // Tạo một hằng số
var b = ImmutablePoint(1, 1); // KHÔNG tạo hằng số

assert(!identical(a, b)); // KHÔNG phải cùng một instance!
```


<a id="getting-an-objects-type"></a>
## Lấy kiểu của một object

Để lấy kiểu của một object tại runtime,
bạn có thể dùng property `runtimeType` của `Object`,
property này trả về một object [`Type`][].

<?code-excerpt "misc/test/language_tour/classes_test.dart (runtime-type)"?>
```dart
print('The type of a is ${a.runtimeType}');
```

:::warning
Hãy dùng một [toán tử kiểm tra kiểu][type test operator] thay cho `runtimeType`
để kiểm tra kiểu của object.
Trong môi trường production, phép kiểm tra `object is Type` ổn định hơn
phép kiểm tra `object.runtimeType == Type`.
:::

Cho đến đây, bạn đã thấy cách _sử dụng_ các lớp.
Phần còn lại của mục này trình bày cách _implement_ các lớp.


<a id="instance-variables"></a>
## Instance variable

Sau đây là cách khai báo các instance variable:

<?code-excerpt "misc/lib/language_tour/classes/point_with_main.dart (class)"?>
```dart
class Point {
  double? x; // Khai báo instance variable x, ban đầu là null.
  double? y; // Khai báo y, ban đầu là null.
  double z = 0; // Khai báo z, ban đầu là 0.
}
```

Một instance variable chưa được khởi tạo, được khai báo với
[kiểu nullable][], có giá trị `null`.
Các instance variable non-nullable [phải được khởi tạo][] ngay khi khai báo.

Mọi instance variable đều tạo ra một method *getter* ngầm định.
Các instance variable không phải `final` và
các instance variable `late final` không có initializer cũng tạo ra
một method *setter* ngầm định. Để biết chi tiết,
xem [Getter và setter][Getters and setters].

<?code-excerpt "misc/lib/language_tour/classes/point_with_main.dart (class-main)" replace="/(double .*?;).*/$1/g" plaster="none"?>
```dart
class Point {
  double? x; // Khai báo instance variable x, ban đầu là null.
  double? y; // Khai báo y, ban đầu là null.
}

void main() {
  var point = Point();
  point.x = 4; // Dùng method setter cho x.
  assert(point.x == 4); // Dùng method getter cho x.
  assert(point.y == null); // Giá trị mặc định là null.
}
```

Việc khởi tạo một instance variable không phải `late` tại nơi nó được khai báo
sẽ đặt giá trị khi instance được tạo,
trước khi constructor và initializer list của nó thực thi.
Do đó, biểu thức khởi tạo (sau dấu `=`)
của một instance variable không phải `late` không thể truy cập `this`.

<?code-excerpt "misc/lib/language_tour/classes/point_this.dart"?>
```dart
double initialX = 1.5;

class Point {
  // HỢP LỆ, có thể truy cập các khai báo không phụ thuộc vào `this`:
  double? x = initialX;

  // LỖI, không thể truy cập `this` trong initializer không phải `late`:
  double? y = this.x;

  // HỢP LỆ, có thể truy cập `this` trong initializer `late`:
  late double? z = this.x;

  // HỢP LỆ, `this.x` và `this.y` là các khai báo parameter, không phải biểu thức:
  Point(this.x, this.y);
}
```

Instance variable có thể là `final`,
trong trường hợp đó chúng phải được đặt đúng một lần.
Hãy khởi tạo các instance variable `final`, không phải `late`
tại nơi khai báo,
bằng một constructor parameter, hoặc
bằng [initializer list][] của constructor:

<?code-excerpt "misc/lib/effective_dart/usage_good.dart (field-init-at-decl)"?>
```dart
class ProfileMark {
  final String name;
  final DateTime start = DateTime.now();

  ProfileMark(this.name);
  ProfileMark.unnamed() : name = '';
}
```

Nếu cần gán giá trị cho một instance variable `final`
sau khi phần thân constructor bắt đầu, bạn có thể dùng một trong các cách sau:

* Tính toán giá trị trong một [factory constructor][],
  rồi truyền nó cho một generative constructor có nhiệm vụ
  khởi tạo instance variable `final`.
* Dùng `late final`, nhưng [_hãy cẩn thận:_][late-final-ivar]
  một `late final` không có initializer sẽ thêm setter vào API.

<a id="implicit-interfaces"></a>
## Interface ngầm định

Mỗi lớp ngầm định nghĩa một interface chứa tất cả instance member
của lớp và của mọi interface mà nó implement. Nếu muốn
tạo lớp A hỗ trợ API của lớp B mà không kế thừa implementation của B,
lớp A nên implement interface B.

Một lớp implement một hoặc nhiều interface bằng cách khai báo chúng trong
mệnh đề `implements`, rồi cung cấp các API mà những interface đó yêu cầu.
Ví dụ:

<?code-excerpt "misc/lib/language_tour/classes/impostor.dart"?>
```dart
// Một người. Interface ngầm định chứa greet().
class Person {
  // Có trong interface, nhưng chỉ hiển thị trong library này.
  final String _name;

  // Không có trong interface, vì đây là constructor.
  Person(this._name);

  // Có trong interface.
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// Một implementation của interface Person.
class Impostor implements Person {
  String get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}

String greetBob(Person person) => person.greet('Bob');

void main() {
  print(greetBob(Person('Kathy')));
  print(greetBob(Impostor()));
}
```

Sau đây là ví dụ chỉ định rằng một lớp implement nhiều interface:

<?code-excerpt "misc/lib/language_tour/classes/misc.dart (point-interfaces)"?>
```dart
class Point implements Comparable, Location {
  ...
}
```


<a id="class-variables-and-methods"></a>
## Biến và method của lớp

Dùng keyword `static` để implement các biến và method dùng chung cho toàn lớp.

<a id="static-variables"></a>
### Biến static

Biến static (biến của lớp) hữu ích cho trạng thái và
hằng số dùng chung cho toàn lớp:

<?code-excerpt "misc/lib/language_tour/classes/misc.dart (static-field)"?>
```dart
class Queue {
  static const initialCapacity = 16;
  // ···
}

void main() {
  assert(Queue.initialCapacity == 16);
}
```

Biến static không được khởi tạo cho đến khi chúng được sử dụng.

:::note
Trang này tuân theo [khuyến nghị của style guide](/effective-dart/style#identifiers)
là ưu tiên `lowerCamelCase` cho tên hằng số.
:::

<a id="static-methods"></a>
### Method static

Method static (method của lớp) không thao tác trên một instance,
vì vậy không có quyền truy cập `this`.
Tuy nhiên, chúng có quyền truy cập các biến static.
Như ví dụ sau cho thấy,
bạn gọi method static trực tiếp trên một lớp:

<?code-excerpt "misc/lib/language_tour/classes/point_with_distance_method.dart"?>
```dart
import 'dart:math';

class Point {
  double x, y;
  Point(this.x, this.y);

  static double distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```

:::note
Hãy cân nhắc dùng top-level function thay cho method static cho
các tiện ích và chức năng phổ biến hoặc được sử dụng rộng rãi.
:::

Bạn có thể dùng method static làm hằng số tại compile time. Ví dụ,
bạn có thể truyền một method static làm parameter cho constant constructor.


[`Object`]: {{site.dart-api}}/dart-core/Object-class.html
[top-and-bottom]: /null-safety/understanding-null-safety#top-and-bottom
[Extension method]: /language/extension-methods
[Class modifier]: /language/class-modifiers
[constant constructor]: /language/constructors#constant-constructors
[`Type`]: {{site.dart-api}}/dart-core/Type-class.html
[type test operator]: /language/operators#type-test-operators
[Getters and setters]: /language/methods#getters-and-setters
[initializer list]: /language/constructors#use-an-initializer-list
[factory constructor]: /language/constructors#factory-constructors
[late-final-ivar]: /effective-dart/design#avoid-public-late-final-fields-without-initializers
[nullable type]: /null-safety/understanding-null-safety#using-nullable-types
[must be initialized]: /null-safety/understanding-null-safety#uninitialized-variables
