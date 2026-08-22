---
title: Class modifier
description: >-
  Các keyword modifier cho khai báo lớp nhằm kiểm soát quyền truy cập của library bên ngoài.
prevpage:
  url: /language/callable-objects
  title: Object có thể gọi
nextpage:
  url: /language/class-modifiers-for-apis
  title: Class modifier dành cho người duy trì API
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore: (stable|beta|dev)[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore: (stable|beta|dev)[^\n]+\n/$1\n/g; /. • (lib|test)\/\w+\.dart:\d+:\d+//g"?>

:::version-note
Các class modifier, ngoại trừ `abstract`, yêu cầu
[language version][] tối thiểu là 3.0.
:::

Class modifier kiểm soát cách một lớp hoặc mixin có thể được sử dụng, cả
[từ bên trong library của chính nó](#abstract) lẫn
từ bên ngoài library nơi nó được định nghĩa.

Các keyword modifier đứng trước khai báo class hoặc mixin.
Ví dụ, viết `abstract class` sẽ định nghĩa một abstract class.
Tập hợp đầy đủ các modifier có thể xuất hiện trước khai báo lớp bao gồm:

- `abstract`
- `base`
- `final`
- `interface`
- `sealed`
- [`mixin`][class, mixin, or mixin class]

Chỉ modifier `base` có thể xuất hiện trước khai báo mixin.
Các modifier không áp dụng cho những khai báo khác như
`enum`, `typedef`, `extension` hoặc `extension type`.

Khi quyết định có nên dùng class modifier hay không,
hãy cân nhắc mục đích sử dụng dự kiến của lớp và
những hành vi mà lớp cần có thể dựa vào.

:::tip
Nếu bạn đã quen thuộc với class modifier của Dart và chỉ muốn
xem phần khái quát hoặc ôn lại hành vi của chúng khi kết hợp,
hãy xem [Tham chiếu class modifier][Class modifier reference].

Nếu bạn duy trì một library,
hãy đọc trang [Class modifier dành cho người duy trì API][Class modifiers for API maintainers] để biết
cách xử lý những thay đổi này cho các library của bạn.
:::

[Class modifier reference]: /language/modifier-reference
[Class modifiers for API maintainers]: /language/class-modifiers-for-apis

<a id="no-modifier"></a>
## Không có modifier

Để cho phép không hạn chế việc dựng hoặc tạo subtype từ bất kỳ library nào,
hãy dùng khai báo `class` hoặc `mixin` không có modifier. Theo mặc định, bạn có thể:

- [Dựng][Construct] instance mới của một lớp.
- [Mở rộng][Extend] một lớp để tạo subtype mới.
- [Implement][] interface của một lớp hoặc mixin.
- [Mix in][mixin] một mixin hoặc mixin class.

<a id="abstract"></a>
## `abstract`

Để định nghĩa một lớp không yêu cầu implementation đầy đủ, cụ thể cho
toàn bộ interface của nó, hãy dùng modifier `abstract`.

Abstract class không thể được dựng từ bất kỳ library nào, dù là library của chính nó
hay library bên ngoài. Abstract class thường có [abstract method][abstract methods].

<?code-excerpt "language/lib/class_modifiers/ex1/a.dart"?>
```dart title="a.dart"
abstract class Vehicle {
  void moveForward(int meters);
}
```

<?code-excerpt "language/lib/class_modifiers/ex1/b.dart (abstract-usages)"?>
```dart title="b.dart"
import 'a.dart';

// Lỗi: Không thể khởi tạo `Vehicle` vì
// nó được đánh dấu là `abstract`.
Vehicle myVehicle = Vehicle();

// Có thể được mở rộng.
class Car extends Vehicle {
  int passengers = 4;

  @override
  void moveForward(int meters) {
    // ...
  }
}

// Có thể được implement.
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) {
    // ...
  }
}
```

Nếu muốn abstract class của bạn trông như có thể khởi tạo được,
hãy định nghĩa một [factory constructor][].

<a id="base"></a>
## `base`

Để bắt buộc kế thừa implementation của một lớp hoặc mixin,
hãy dùng modifier `base`.
Base class không cho phép implementation bên ngoài library của chính nó.
Điều này bảo đảm:

- Constructor của base class được gọi bất cứ khi nào
  một instance thuộc subtype của lớp được tạo.
- Tất cả private member đã implement đều tồn tại trong subtype.
- Một member mới được implement trong lớp `base` không làm hỏng subtype,
  vì tất cả subtype đều kế thừa member mới.
  - Điều này đúng trừ khi subtype đã khai báo một member cùng tên
    và có signature không tương thích.

Bạn phải đánh dấu mọi lớp implement hoặc mở rộng base class bằng
`base`, `final` hoặc `sealed`. Điều này ngăn library bên ngoài
phá vỡ các bảo đảm của base class.

<?code-excerpt "language/lib/class_modifiers/ex2/a.dart"?>
```dart title="a.dart"
base class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}
```

<?code-excerpt "language/lib/class_modifiers/ex2/b.dart"?>
```dart title="b.dart"
import 'a.dart';

// Có thể được dựng.
Vehicle myVehicle = Vehicle();

// Có thể được mở rộng.
base class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// LỖI: Không thể implement `Vehicle` trong một library khác vì
// nó được đánh dấu bằng `base`.
base class MockVehicle implements Vehicle {
  @override
  void moveForward() {
    // ...
  }
}
```

<a id="interface"></a>
## `interface`

Để định nghĩa một interface, hãy dùng modifier `interface`.
Library bên ngoài library định nghĩa chính interface có thể
implement interface nhưng không thể mở rộng nó.
Điều này bảo đảm:

- Khi một instance method của lớp gọi
  một instance method khác trên `this`, nó sẽ luôn
  gọi một implementation đã biết của method từ cùng library.
- Library khác không thể override những method mà các method của
  interface class có thể gọi sau này theo cách ngoài dự kiến.
  Điều này làm giảm [vấn đề fragile base class][fragile base class problem].

<?code-excerpt "language/lib/class_modifiers/ex3/a.dart"?>
```dart title="a.dart"
interface class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}
```

<?code-excerpt "language/lib/class_modifiers/ex3/b.dart"?>
```dart title="b.dart"
import 'a.dart';

// Có thể được dựng.
Vehicle myVehicle = Vehicle();

// LỖI: Không thể mở rộng `Vehicle` trong một library khác vì
// nó được đánh dấu bằng `interface`.
class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// Có thể được implement.
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) {
    // ...
  }
}
```

<a id="abstract-interface"></a>
### `abstract interface`

Cách dùng phổ biến nhất của modifier `interface` là định nghĩa một pure interface.
[Kết hợp](#combining-modifiers) các modifier `interface` và [`abstract`](#abstract)
để tạo một `abstract interface class`.

Giống lớp `interface`, library khác có thể implement nhưng không thể kế thừa
pure interface. Giống lớp `abstract`, pure interface có thể có abstract member.

<a id="final"></a>
## `final`

Để đóng hệ thống phân cấp kiểu, hãy dùng modifier `final`.
Điều này ngăn việc tạo subtype từ một lớp bên ngoài library hiện tại.
Không cho phép cả kế thừa lẫn implementation sẽ ngăn hoàn toàn việc tạo subtype.
Điều này bảo đảm:

- Bạn có thể thêm các thay đổi tăng dần vào API một cách an toàn.
- Bạn có thể gọi instance method khi biết chúng chưa bị ghi đè trong
  subclass của bên thứ ba.

Final class có thể được mở rộng hoặc implement trong cùng library.
Modifier `final` bao hàm tác dụng của `base`, vì vậy
mọi subclass cũng phải được đánh dấu là `base`, `final` hoặc `sealed`.

<?code-excerpt "language/lib/class_modifiers/ex4/a.dart"?>
```dart title="a.dart"
final class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}
```

<?code-excerpt "language/lib/class_modifiers/ex4/b.dart"?>
```dart title="b.dart"
import 'a.dart';

// Có thể được dựng.
Vehicle myVehicle = Vehicle();

// LỖI: Không thể mở rộng `Vehicle` trong library khác
// vì nó được đánh dấu `final`.
class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// LỖI: Không thể implement `Vehicle` trong một library khác vì
// nó được đánh dấu `final`.
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) {
    // ...
  }
}
```

<a id="sealed"></a>
## `sealed`

Để tạo một tập subtype đã biết, có thể liệt kê, hãy dùng modifier `sealed`.
Điều này cho phép bạn tạo một switch trên các subtype đó
được bảo đảm về mặt static là [_exhaustive_][exhaustive].

Modifier `sealed` ngăn một lớp bị mở rộng hoặc
implement bên ngoài library của chính nó. Sealed class ngầm là
[abstract](#abstract).

- Bản thân chúng không thể được dựng.
- Chúng có thể có [factory constructor](/language/constructors#factory-constructors).
- Chúng có thể định nghĩa constructor cho subclass sử dụng.

Tuy nhiên, subclass của sealed class không ngầm là abstract.

Compiler nhận biết mọi direct subtype có thể có
vì chúng chỉ có thể tồn tại trong cùng library.
Điều này cho phép compiler cảnh báo khi switch không
xử lý exhaustive tất cả subtype có thể có trong các case:

<?code-excerpt "language/lib/class_modifiers/ex5/sealed.dart"?>
```dart
sealed class Vehicle {}

class Car extends Vehicle {}

class Truck implements Vehicle {}

class Bicycle extends Vehicle {}

// LỖI: Không thể khởi tạo `Vehicle` vì
// nó được đánh dấu `sealed` và do đó ngầm là abstract.
Vehicle myVehicle = Vehicle();

// Subclass của sealed class có thể được khởi tạo trừ khi cũng bị hạn chế.
Vehicle myCar = Car();

extension VehicleSounds on Vehicle {
  String get sound {
    // LỖI: Switch không tính đến một cách exhaustive
    // tất cả object có thể có thuộc kiểu `Vehicle`.
    // Trong ví dụ này, một `Vehicle` có runtime type là `Bicycle`
    // sẽ không khớp với bất kỳ case nào.
    return switch (this) {
      Car() => 'vroom',
      Truck() => 'VROOOOMM',
    };
  }
}
```

Nếu không muốn [switch exhaustive][exhaustive],
hoặc muốn có thể thêm subtype sau này mà không phá vỡ API,
hãy dùng modifier [`final`](#final). Để xem phần so sánh sâu hơn,
đọc [`sealed` so với `final`](/language/class-modifiers-for-apis#sealed-versus-final).

<a id="combining-modifiers"></a>
## Kết hợp modifier

Bạn có thể kết hợp một số modifier để tạo các lớp hạn chế chồng lên nhau.
Một khai báo lớp có thể gồm, theo thứ tự:

1. (Không bắt buộc) `abstract`, mô tả liệu lớp có thể
   chứa abstract member hay không và ngăn việc khởi tạo.
2. (Không bắt buộc) Một trong `base`, `interface`, `final` hoặc `sealed`, mô tả
   các hạn chế đối với việc library khác tạo subtype của lớp.
3. (Không bắt buộc) `mixin`, mô tả liệu khai báo có thể được mix in hay không.
4. Chính keyword `class`.

Bạn không thể kết hợp một số modifier vì chúng
mâu thuẫn, dư thừa hoặc loại trừ lẫn nhau theo cách khác:

* `abstract` với `sealed`. Lớp [sealed](#sealed) ngầm là
  [abstract](#abstract).
* `interface`, `final` hoặc `sealed` với `mixin`. Các access modifier này
  ngăn việc [mix in][mixin].

Để được hướng dẫn thêm về cách kết hợp class modifier,
hãy xem [Tham chiếu class modifier][Class modifier reference].

[Class modifier reference]: /language/modifier-reference


[language version]: /language/versioning
[class, mixin, or mixin class]: /language/mixins#class-mixin-or-mixin-class
[mixin]: /language/mixins
[fragile base class problem]: https://en.wikipedia.org/wiki/Fragile_base_class
[`noSuchMethod`]: /language/extend#nosuchmethod
[construct]: /language/constructors
[extend]: /language/extend
[implement]: /language/classes#implicit-interfaces
[factory constructor]: /language/constructors#factory-constructors
[exhaustive]: /language/branches#exhaustiveness-checking
[abstract methods]: /language/methods#abstract-methods
[syntax specification]: {{site.repo.dart.lang}}/blob/main/accepted/3.0/class-modifiers/feature-specification.md#syntax
