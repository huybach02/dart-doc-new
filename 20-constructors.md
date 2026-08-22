---
title: Constructor
description: Mọi kiến thức về cách sử dụng constructor trong Dart.
prevpage:
  url: /language/classes
  title: Lớp
nextpage:
  url: /language/primary-constructors
  title: Primary constructor
---

Constructor là các function đặc biệt tạo ra instance của lớp.

Dart implement nhiều loại constructor.
Ngoại trừ default constructor,
các function này dùng cùng tên với lớp của chúng.

[Generative constructor][generative]
: Tạo instance mới và khởi tạo các instance variable.

[Default constructor][default]
: Được dùng để tạo instance mới khi chưa chỉ định constructor.
  Nó không nhận argument và không có tên.

[Named constructor][named]
: Làm rõ mục đích của constructor hoặc
  cho phép tạo nhiều constructor cho cùng một lớp.

[Constant constructor][constant]
: Tạo instance dưới dạng hằng số tại compile time.

[Factory constructor][factory]
: Tạo một instance mới của subtype hoặc
  trả về một instance có sẵn từ cache.

[Redirecting constructor][redirecting]
: Chuyển tiếp lời gọi đến một constructor khác trong cùng lớp.

[default]: #default-constructors
[generative]: #generative-constructors
[named]: #named-constructors
[constant]: #constant-constructors
[factory]: #factory-constructors
[redirecting]: #redirecting-constructors

<?code-excerpt path-base="misc/lib/language_tour/classes"?>

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

<a id="types-of-constructors"></a>
## Các loại constructor

<a id="generative-constructors"></a>
### Generative constructor

Để khởi tạo một lớp, hãy dùng generative constructor.

<?code-excerpt "point_alt.dart (idiomatic-constructor)" plaster="none"?>
```dart
class Point {
  // Các instance variable lưu tọa độ của điểm.
  double x;
  double y;

  // Generative constructor với initializing formal parameter:
  Point(this.x, this.y);
}
```

<a id="default-constructors"></a>
### Default constructor

Nếu bạn không khai báo constructor, Dart sẽ dùng default constructor.
Default constructor là generative constructor không có argument hoặc tên.

<a id="named-constructors"></a>
### Named constructor

Dùng named constructor để implement nhiều constructor cho một lớp
hoặc để cung cấp thêm sự rõ ràng:

<?code-excerpt "point.dart (named-constructor)" replace="/Point\.\S*/[!$&!]/g" plaster="none"?>
```dart
const double xOrigin = 0;
const double yOrigin = 0;

class Point {
  final double x;
  final double y;

  // Đặt các instance variable x và y
  // trước khi phần thân constructor chạy.
  Point(this.x, this.y);

  // Named constructor
  [!Point.origin()!] : x = xOrigin, y = yOrigin;
}
```

Subclass không kế thừa named constructor của superclass.
Để tạo subclass với một named constructor được định nghĩa trong superclass,
hãy implement constructor đó trong subclass.

<a id="constant-constructors"></a>
### Constant constructor

Nếu lớp của bạn tạo ra các object không thay đổi, hãy biến những
object này thành hằng số tại compile time.
Để biến object thành hằng số tại compile time, hãy định nghĩa constructor `const`
với tất cả instance variable được đặt là `final`.

<?code-excerpt "immutable_point.dart"?>
```dart
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

Constant constructor không phải lúc nào cũng tạo hằng số.
Chúng có thể được gọi trong một context không phải `const`.
Để tìm hiểu thêm, hãy xem mục [sử dụng constructor][using constructors].

<a id="redirecting-constructors"></a>
### Redirecting constructor

Một constructor có thể redirect đến constructor khác trong cùng lớp.
Redirecting constructor có phần thân rỗng.
Constructor dùng `this` thay cho tên lớp sau dấu hai chấm (`:`).

<?code-excerpt "point_redirecting.dart"?>
```dart
class Point {
  double x, y;

  // Constructor chính của lớp này.
  Point(this.x, this.y);

  // Ủy quyền cho constructor chính.
  Point.alongXAxis(double x) : this(x, 0);
}
```

<a id="factory-constructors"></a>
### Factory constructor

Khi gặp một trong hai trường hợp implement constructor sau,
hãy dùng keyword `factory`:

* Constructor không phải lúc nào cũng tạo instance mới của lớp đó.
  Mặc dù factory constructor không thể trả về `null`,
  nó có thể trả về:

  * một instance có sẵn từ cache thay vì tạo instance mới
  * một instance mới của subtype

* Bạn cần thực hiện công việc không đơn giản trước khi dựng một instance.
  Điều này có thể bao gồm kiểm tra argument hoặc thực hiện bất kỳ xử lý nào khác
  mà initializer list không thể đảm nhiệm.

:::tip
Bạn cũng có thể xử lý việc khởi tạo muộn một biến final
bằng [`late final`][late-final-ivar] (hãy cẩn thận!).
:::

Ví dụ sau bao gồm hai factory constructor.

* Factory constructor `Logger` trả về object từ cache.
* Factory constructor `Logger.fromJson` khởi tạo một biến final
  từ object JSON.

<?code-excerpt "logger.dart (constructors)"?>
```dart
class Logger {
  final String name;
  bool mute = false;

  // _cache là private trong library, nhờ
  // dấu _ ở trước tên của nó.
  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }

  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```

:::warning
Factory constructor không thể truy cập `this`.
:::

Hãy dùng factory constructor như bất kỳ constructor nào khác:

<?code-excerpt "logger.dart (logger)"?>
```dart
var logger = Logger('UI');
logger.log('Button clicked');

var logMap = {'name': 'UI'};
var loggerJson = Logger.fromJson(logMap);
```

<a id="redirecting-factory-constructors"></a>
### Redirecting factory constructor

Redirecting factory constructor chỉ định lời gọi đến constructor của một lớp khác
để dùng bất cứ khi nào ai đó gọi redirecting constructor.

```dart
factory Listenable.merge(List<Listenable> listenables) = _MergingListenable
```

Vì factory constructor thông thường cũng có thể tạo
và trả về instance của lớp khác,
redirecting factory có vẻ không cần thiết.
Tuy nhiên, redirecting factory có một số ưu điểm:

* Một abstract class có thể cung cấp constant constructor
  sử dụng constant constructor của một lớp khác.
* Redirecting factory constructor tránh việc các forwarder
  phải lặp lại formal parameter và giá trị mặc định của chúng.

<a id="constructor-tear-offs"></a>
### Constructor tear-off

Dart cho phép bạn cung cấp constructor làm parameter mà không gọi nó.
Được gọi là _tear-off_ (vì bạn _tách bỏ_ các dấu ngoặc đơn),
nó đóng vai trò là một closure gọi constructor với cùng các parameter.

Nếu tear-off là một constructor có cùng
signature và return type mà method chấp nhận,
bạn có thể dùng tear-off làm parameter hoặc biến.

Tear-off khác với lambda hoặc anonymous function.
Lambda đóng vai trò là wrapper cho constructor,
trong khi tear-off chính là constructor.

**Dùng tear-off**

```dart tag=good
// Dùng tear-off cho named constructor:
var strings = charCodes.map(String.fromCharCode);

// Dùng tear-off cho unnamed constructor:
var buffers = charCodes.map(StringBuffer.new);
```

**Không dùng lambda**

```dart tag=bad
// Thay cho lambda dành cho named constructor:
var strings = charCodes.map((code) => String.fromCharCode(code));

// Thay cho lambda dành cho unnamed constructor:
var buffers = charCodes.map((code) => StringBuffer(code));
```

Để xem thảo luận thêm, hãy xem video Decoding Flutter này về tear-off.

<YouTubeEmbed id="OmCaloD7sis" title="Dart Tear-offs | Decoding Flutter"></YouTubeEmbed>

<a id="concise-constructor-syntax"></a>
## Cú pháp constructor ngắn gọn

Trong Dart 3.13 trở lên, bạn có thể bỏ tên lớp
khi khai báo generative hoặc factory constructor bên trong phần thân lớp
bằng cách dùng trực tiếp modifier `new` hoặc `factory`:

-   `new` hoặc `new named`
-   `factory` hoặc `factory named`

Không giống named constructor truyền thống (chẳng hạn `Point.origin()`),
named constructor ngắn gọn không dùng dấu chấm
giữa keyword (modifier) và tên.

<?code-excerpt "concise_syntax.dart (concise-syntax)"?>
```dart
class Point {
  double x, y;

  // Unnamed generative constructor ngắn gọn.
  new(this.x, this.y);

  // Named generative constructor ngắn gọn.
  new origin() : x = 0, y = 0;

  // Tương đương với `factory Point.clone(Point other)`.
  factory clone(Point other) => Point(other.x, other.y);
}
```

Cú pháp này giảm độ dài dòng và giúp việc refactor tên lớp dễ dàng hơn.

:::note
Với cú pháp này, một method tên `factory` không có return type
(như `factory () {}`) được diễn giải là unnamed factory constructor.
:::

Bảng sau cho thấy cách ánh xạ cú pháp constructor truyền thống
sang cú pháp constructor ngắn gọn (cho một lớp tên `LongClassName`):

| Cú pháp Dart gốc                         | Cú pháp viết tắt            |
| ----------------------------------------- | --------------------------- |
| `LongClassName() {}`                      | `new() {}`                  |
| `LongClassName.name() {}`                 | `new name() {}`             |
| `const LongClassName();`                  | `const new();`              |
| `const LongClassName.name();`             | `const new name();`         |
| `LongClassName(): this.other();`          | `new(): this.other();`      |
| `LongClassName.name(): this();`           | `new name(): this();`       |
| `const LongClassName(): this.other();`    | `const new(): this.other();` |
| `const LongClassName.name(): this();`     | `const new name(): this();` |
| `factory LongClassName() { ... }`         | `factory() { ... }`         |
| `factory LongClassName.name() { ... }`    | `factory name() { ... }`    |
| `factory LongClassName() = D;`            | `factory() = D;`            |
| `factory LongClassName.name() = D;`       | `factory name() = D;`       |
| `const factory LongClassName() = D;`      | `const factory() = D;`      |
| `const factory LongClassName.name() = D;` | `const factory name() = D;`  |

Để có cú pháp còn ngắn gọn hơn,
nơi bạn định nghĩa field và constructor trên một dòng duy nhất,
xem [Primary constructor](/language/primary-constructors).

<a id="instance-variable-initialization"></a>
## Khởi tạo instance variable

Dart cung cấp một số cách để khởi tạo instance variable.
Bạn có thể gán giá trị trong khai báo,
dùng initializing formal parameter,
hoặc dùng initializer list.

<a id="initialize-instance-variables-in-the-declaration"></a>
### Khởi tạo instance variable trong khai báo

Khởi tạo các instance variable khi bạn khai báo biến.

<?code-excerpt "point_alt.dart (initialize-declaration)" plaster="none"?>
```dart
class PointA {
  double x = 1.0;
  double y = 2.0;

  // Default constructor ngầm định đặt các biến này thành (1.0,2.0)
  // PointA();

  @override
  String toString() {
    return 'PointA($x,$y)';
  }
}
```

<a id="use-initializing-formal-parameters"></a>
### Dùng initializing formal parameter

Để đơn giản hóa pattern phổ biến là gán argument của constructor
cho một instance variable, Dart có *initializing formal parameter*.

Trong khai báo constructor, hãy thêm `this.<propertyName>`
và bỏ phần thân. Keyword `this` tham chiếu đến instance hiện tại.

Khi có xung đột tên, hãy dùng `this`.
Nếu không, style Dart sẽ bỏ `this`.
Có một ngoại lệ đối với generative constructor, nơi
bạn phải đặt tiền tố `this` trước tên initializing formal parameter.

Như đã lưu ý trước đó trong hướng dẫn này, một số constructor
và một số phần của constructor không thể truy cập `this`. Chúng bao gồm:

* Factory constructor
* Vế phải của initializer list
* Argument cho constructor của superclass

Initializing formal parameter cũng cho phép bạn khởi tạo
instance variable non-nullable hoặc `final`.
Cả hai loại biến này đều yêu cầu được khởi tạo hoặc có giá trị mặc định.

<?code-excerpt "point_alt.dart (initialize-formal)" plaster="none"?>
```dart
class PointB {
  final double x;
  final double y;

  // Đặt các instance variable x và y
  // trước khi phần thân constructor chạy.
  PointB(this.x, this.y);

  // Initializing formal parameter cũng có thể là không bắt buộc.
  PointB.optional([this.x = 0.0, this.y = 0.0]);
}
```


Điều này cũng hoạt động với named variable.

<?code-excerpt "point_alt.dart (initialize-named)" plaster="none"?>
```dart
class PointC {
  double x; // phải được đặt trong constructor
  double y; // phải được đặt trong constructor

  // Generative constructor với initializing formal parameter
  // có giá trị mặc định
  PointC.named({this.x = 1.0, this.y = 1.0});

  @override
  String toString() {
    return 'PointC.named($x,$y)';
  }
}

// Constructor dùng named variable.
final pointC = PointC.named(x: 2.0, y: 2.0);
```

Mọi biến được đưa vào từ initializing formal parameter đều là
final và chỉ nằm trong scope của các biến được khởi tạo.

Để thực hiện logic mà bạn không thể biểu diễn trong initializer list,
hãy tạo một [factory constructor](#factory-constructors)
hoặc [method static][static method] với logic đó.
Sau đó, bạn có thể truyền các giá trị đã tính toán cho một constructor thông thường.

Bạn có thể đặt constructor parameter là nullable để tránh phải khởi tạo chúng.

<?code-excerpt "point_alt.dart (initialize-null)" plaster="none"?>
```dart
class PointD {
  double? x; // null nếu không được đặt trong constructor
  double? y; // null nếu không được đặt trong constructor

  // Generative constructor với initializing formal parameter
  PointD(this.x, this.y);

  @override
  String toString() {
    return 'PointD($x,$y)';
  }
}
```

<a id="private-named-parameters"></a>
### Private named parameter

:::version-note
Việc dùng private named parameter làm initializing formal
yêu cầu [language version][] tối thiểu là 3.12.
:::

Trong Dart, field bắt đầu bằng dấu gạch dưới là private đối với library của nó.
Để khởi tạo một field private bằng named parameter,
bạn có thể viết phần code gán thủ công lặp lại
trong initializer list:

<?code-excerpt "point_private_old.dart (initialize-private-named-before)" plaster="none"?>
```dart
class Point {
  final double _x;
  Point({required double x}) : _x = x;
}
```

Bạn cũng có thể khởi tạo field private trực tiếp
trong danh sách parameter của constructor.
Khi bạn thêm tiền tố `this._` cho named parameter,
compiler tự động loại bỏ dấu gạch dưới đối với bên gọi,
cho phép họ dùng một tên public gọn gàng:

<?code-excerpt "point_private_new.dart (initialize-private-named-after)" plaster="none"?>
```dart
class Point {
  final double _x;
  Point({required this._x});
}
```

Trong cả hai trường hợp,
bên gọi dùng tên public `x` tại call site:

<?code-excerpt "point_private_new.dart (initialize-private-named-usage)" plaster="none"?>
```dart
var p = Point(x: 1.0);
```

Giống [named parameter](/language/functions#named-parameters) thông thường, bạn có thể
đặt private named parameter thành optional hoặc required.
Bạn cũng có thể cung cấp giá trị mặc định tường minh.

Trong ví dụ sau, parameter `_x` là optional và mặc định là `null`.
Parameter `_y` cũng là optional nhưng có giá trị mặc định tường minh là `0.0`:

<?code-excerpt "point_alt.dart (initialize-private-named)" plaster="none"?>
```dart
class PointPrivate {
  final double? _x; // Field nullable
  final double _y; // Field non-nullable

  PointPrivate({this._x, this._y = 0.0});

  @override
  String toString() => 'PointPrivate($_x, $_y)';
}

void testPrivate() {
  var p = PointPrivate(x: 1.0, y: 2.0);
  print(p);
}
```

<a id="constraints"></a>
#### Ràng buộc

* **Không xung đột:** Cả tên private lẫn tên public được tạo ra
  đều không được trùng với bất kỳ tên parameter nào khác trong cùng constructor.
* **Chỉ initializing formal:** Nhìn chung, named parameter trong Dart
  không thể là private. Khả năng này là một ngoại lệ chỉ áp dụng
  cho named parameter là initializing formal (`this._field`).
  Bạn không thể dùng identifier private cho named parameter thông thường.
* **Tên public hợp lệ:** Tên private phải ánh xạ đến một identifier public hợp lệ.
  Ví dụ, `this._` hoặc `this._2x` không hợp lệ
  vì chúng không có phần tương ứng public hợp lệ.

<a id="usage-in-initializer-lists"></a>
#### Sử dụng trong initializer list

Trong initializer list của constructor,
hãy tham chiếu parameter bằng tên private của nó:

<?code-excerpt "point_alt.dart (initialize-private-named-assert)" plaster="none"?>
```dart
class PointPrivateAssert {
  final double _x;

  PointPrivateAssert({required this._x}) : assert(_x >= 0);
}
```

<a id="interaction-with-super-parameters"></a>
#### Tương tác với super parameter

Khi mở rộng một lớp dùng private named parameter,
subclass dùng tên public cho [super parameter][super parameters].

Trong ví dụ sau, lớp `Tool` định nghĩa field private `_price`.
Mặc dù field là private,
named parameter tương ứng của nó là public (`price`, không phải `_price`).
Để chuyển tiếp giá trị,
subclass `Hammer` dùng identifier public `price`:

<?code-excerpt "point_alt.dart (initialize-private-named-super)" plaster="none"?>
```dart
class Tool {
  final int _price;
  Tool({required this._price});
}

class Hammer extends Tool {
  // Chuyển tiếp đến argument public 'price'
  Hammer({required super.price});
}
```

[super parameters]: /resources/glossary#super-parameter

<a id="use-an-initializer-list"></a>
### Dùng initializer list

Trước khi phần thân constructor chạy, bạn có thể khởi tạo instance variable.
Phân tách các initializer bằng dấu phẩy.

<?code-excerpt "point_alt.dart (initializer-list)"?>
```dart
// Initializer list đặt các instance variable trước khi
// phần thân constructor chạy.
Point.fromJson(Map<String, double> json) : x = json['x']!, y = json['y']! {
  print('In Point.fromJson(): ($x, $y)');
}
```

:::warning
Vế phải của initializer list không thể truy cập `this`.
:::

Để validate input trong quá trình phát triển,
dùng `assert` trong initializer list.

<?code-excerpt "point_alt.dart (initializer-list-with-assert)" replace="/assert\(.*?\)/[!$&!]/g"?>
```dart
Point.withAssert(this.x, this.y) : [!assert(x >= 0)!] {
  print('In Point.withAssert(): ($x, $y)');
}
```

Initializer list giúp thiết lập các field `final`.

Ví dụ sau khởi tạo ba field `final` trong initializer list.
Để thực thi code, nhấp vào **Run**.

<?code-excerpt "point_with_distance_field.dart"?>
```dartpad
import 'dart:math';

class Point {
  final double x;
  final double y;
  final double distanceFromOrigin;

  Point(double x, double y)
    : x = x,
      y = y,
      distanceFromOrigin = sqrt(x * x + y * y);
}

void main() {
  var p = Point(2, 3);
  print(p.distanceFromOrigin);
}
```

<a id="constructor-inheritance"></a>
## Kế thừa constructor

_Subclass_, hay lớp con, không kế thừa *constructor*
từ _superclass_, hay lớp cha trực tiếp của chúng.
Nếu một lớp không khai báo constructor, nó chỉ có thể dùng
[default constructor](#default-constructors).

Một lớp có thể kế thừa _parameter_ của superclass.
Chúng được gọi là [super parameter](#super-parameters)

Constructor hoạt động theo cách phần nào tương tự
cách bạn gọi một chuỗi method static.
Mỗi subclass có thể gọi constructor của superclass để khởi tạo một instance,
giống như subclass có thể gọi method static của superclass.
Quá trình này không “kế thừa” phần thân hoặc signature của constructor.

<a id="non-default-superclass-constructors"></a>
### Constructor không mặc định của superclass

Dart thực thi các constructor theo thứ tự sau:

1. [initializer list](#use-an-initializer-list)
1. constructor không tên, không có argument của superclass
1. constructor không có argument của lớp chính

Nếu superclass không có constructor không tên, không có argument,
hãy gọi một trong các constructor trong superclass.
Trước phần thân constructor (nếu có),
hãy chỉ định constructor của superclass sau dấu hai chấm (`:`).

Trong ví dụ sau,
constructor của lớp `Employee` gọi named constructor
của superclass `Person`. Để thực thi code sau, nhấp vào **Run**.

<?code-excerpt "employee.dart (super)" plaster="none"?>
```dartpad
class Person {
  String? firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person không có default constructor;
  // bạn phải gọi super.fromJson().
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}

void main() {
  var employee = Employee.fromJson({});
  print(employee);
  // In ra:
  // in Person
  // in Employee
  // Instance of 'Employee'
}
```

Vì Dart đánh giá các argument cho constructor của superclass *trước khi*
gọi constructor, một argument có thể là biểu thức như một
lời gọi function.

<?code-excerpt "employee.dart (method-then-constructor)"?>
```dart
class Employee extends Person {
  Employee() : super.fromJson(fetchDefaultData());
  // ···
}
```

:::warning
Argument cho constructor của superclass không thể truy cập `this`.
Ví dụ, argument có thể gọi method *static*
nhưng không thể gọi method *instance*.
:::

<a id="super-parameters"></a>
### Super parameter

Để tránh truyền từng parameter vào lời gọi super của constructor,
hãy dùng super-initializer parameter để chuyển tiếp parameter
đến constructor được chỉ định hoặc default constructor của superclass.
Bạn không thể dùng tính năng này với
[redirecting constructor](#redirecting-constructors).
Super-initializer parameter có cú pháp và ngữ nghĩa giống
[initializing formal parameter](#use-initializing-formal-parameters).

:::version-note
Việc dùng super-initializer parameter
yêu cầu [language version][] tối thiểu là 2.17.
Nếu đang dùng language version cũ hơn,
bạn phải truyền thủ công tất cả parameter của super constructor.
:::

Nếu lời gọi super-constructor bao gồm positional argument,
super-initializer parameter không thể là positional.

<?code-excerpt "super_initializer_positional_parameters.dart (positional)" plaster="none"?>
```dart
class Vector2d {
  final double x;
  final double y;

  Vector2d(this.x, this.y);
}

class Vector3d extends Vector2d {
  final double z;

  // Chuyển tiếp các parameter x và y đến default super constructor giống như:
  // Vector3d(final double x, final double y, this.z) : super(x, y);
  Vector3d(super.x, super.y, this.z);
}
```

Để minh họa thêm, hãy xem xét ví dụ sau.

```dart
  // Nếu bạn gọi super constructor (`super(0)`) với bất kỳ
  // positional argument nào, việc dùng super parameter (`super.x`)
  // sẽ dẫn đến lỗi.
  Vector3d.xAxisError(super.x): z = 0, super(0); // KHÔNG HỢP LỆ
```

Named constructor này cố đặt giá trị `x` hai lần:
một lần trong super constructor và một lần dưới dạng
positional super parameter.
Vì cả hai đều xử lý positional parameter `x`, điều này dẫn đến lỗi.

Khi super constructor có named argument, bạn có thể chia chúng
giữa named super parameter (`super.y` trong ví dụ tiếp theo)
và named argument cho lời gọi super constructor
(`super.named(x: 0)`).

<?code-excerpt "super_initializer_named_parameters.dart (named)" plaster="none"?>
```dart
class Vector2d {
  // ...
  Vector2d.named({required this.x, required this.y});
}

class Vector3d extends Vector2d {
  final double z;

  // Chuyển tiếp parameter y đến named super constructor giống như:
  // Vector3d.yzPlane({required double y, required this.z})
  //       : super.named(x: 0, y: y);
  Vector3d.yzPlane({required super.y, required this.z}) : super.named(x: 0);
}
```

[language version]: /language/versioning
[using constructors]: /language/classes#using-constructors
[late-final-ivar]: /effective-dart/design#avoid-public-late-final-fields-without-initializers
[static method]: /language/classes#static-methods
