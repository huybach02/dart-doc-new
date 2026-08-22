---
title: Primary constructor
description: >-
  Tạo lớp ngắn gọn bằng cách
  khai báo parameter và field trong header của lớp.
prevpage:
  url: /language/constructors
  title: Constructor
nextpage:
  url: /language/methods
  title: Method
---

:::version-note
Primary constructor yêu cầu [language version][] tối thiểu là 3.13.
:::

<a id="overview"></a>
## Tổng quan

Primary constructor cung cấp một cách ngắn gọn để khai báo các field của lớp
và constructor chính của lớp đó trên một dòng duy nhất.
Chúng giảm phần code lặp lại khi khai báo field, truyền parameter
và gán chúng trong phần thân constructor.
Cách viết tắt này thay đổi cách bạn viết khai báo,
nhưng không thay đổi hành vi runtime.

<a id="before-and-after"></a>
### Trước và sau

Hãy xem xét lớp truyền thống có hai field và một constructor sau:

<?code-excerpt "language/lib/primary_constructors/point.dart (point)"?>
```dart
// Cú pháp hiện tại.
class Point {
  int x;
  int y;

  Point(this.x, this.y);
}
```

Việc dùng primary constructor giúp cùng một lớp trở nên ngắn gọn hơn nhiều:

<?code-excerpt "language/lib/primary_constructors/point.dart (point-primary)" replace="/PointPrimary/Point/g"?>
```dart
// Sử dụng primary constructor.
class Point(var int x, var int y);
```

Khai báo một parameter trong primary constructor với `var` hoặc `final`
sẽ ngầm tạo ra một instance variable cho parameter đó.

Mọi constructor khác được khai báo bên trong phần thân lớp
được gọi là **in-body constructor**.

Để bảo đảm primary constructor thực thi trên mọi instance mới,
một class, mixin class hoặc enum có primary constructor
không thể có bất kỳ generative in-body constructor không redirect nào khác.

:::note
Nếu muốn khai báo constructor truyền thống trong
phần thân lớp ngắn gọn hơn mà không lặp lại tên lớp,
hãy xem [Cú pháp constructor ngắn gọn][Concise constructor syntax].
:::

[Concise constructor syntax]: /language/constructors#concise-constructor-syntax

<a id="field-declarations-in-parameters"></a>
## Khai báo field trong parameter

Các parameter trong primary constructor có modifier `var` hoặc `final`,
được gọi là **declaring parameter**, ngầm tạo ra một field.

Nếu bỏ modifier, parameter sẽ không tạo field.
Nó hoạt động giống hệt parameter trong constructor truyền thống.

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (declaring-parameters)"?>
```dart
// Khai báo cả hai field x và y.
class Point(var int x, var int y);

// Không khai báo field.
class User(String name);
```

Vì các modifier `final` và `var` trên parameter được dành riêng
cho declaring parameter trong primary constructor,
bạn không thể dùng chúng trên parameter trong các loại function khác.

Đối với extension type, primary constructor phải có đúng một parameter.
Parameter này luôn là declaring parameter, ngay cả khi bạn bỏ modifier.
Bạn có thể dùng modifier `final`, nhưng dùng `var` là một lỗi.

Mixin class chỉ có thể có primary constructor
không có parameter, phần thân hoặc initializer list.

<a id="primary-initializer-scope" aria-hidden="true"></a>

<a id="primary-constructor-scopes"></a>
## Scope của primary constructor

Khi dùng primary constructor,
các parameter bạn khai báo trong header của lớp
khả dụng trong những phần khác nhau của khai báo lớp.
Dart quản lý khả năng hiển thị của chúng bằng hai scope riêng biệt:

*   **Primary initializer scope**:
    Áp dụng cho initializer của field không phải `late` trong phần thân lớp và
    initializer list của primary constructor (sau `this :`).
    Trong scope này, một tên parameter như `x` tham chiếu
    trực tiếp đến constructor parameter.
*   **Primary parameter scope**:
    Áp dụng cho block phần thân của primary constructor (bên trong `{ ... }`).
    Trong scope này, tên của declaring parameter
    tham chiếu đến _instance variable được tạo ra_ (field),
    trong khi tên của non-declaring parameter vẫn
    tham chiếu đến constructor parameter.

Primary initializer scope giúp các parameter trong header
khả dụng trực tiếp để khởi tạo các field không phải `late`,
loại bỏ nhu cầu dùng một initializer list riêng.
Nó hoạt động giống như khi bạn khởi tạo
biến trong initializer list của constructor truyền thống:

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (initializer-scope)"?>
```dart
class DeltaPoint(final int x, int delta) {
  // Truy cập trực tiếp các parameter 'x' và 'delta'!
  final int y = x + delta;
}
```

Primary parameter scope bảo đảm rằng
mọi cập nhật đối với instance variable đều
được phản ánh chính xác trong phần thân constructor,
trong khi initializer vẫn có quyền truy cập các parameter ban đầu.
Ví dụ sau cho thấy cùng tên `x`
được phân giải khác nhau trong mỗi scope như thế nào:

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (scoping-shadowing)"?>
```dart
class ScopingDemo(var String x, String suffix) {
  // Trong initializer của field không phải late, 'x' tham chiếu đến parameter 'x'.
  final String fieldAtDeclaration = x;
  final String fieldInInitializer;

  // Trong initializer list, 'x' tham chiếu đến parameter 'x'.
  this : fieldInInitializer = x {
    // Bên trong phần thân, 'x' tham chiếu đến instance variable được tạo ra,
    // vì vậy việc gán cho nó sẽ cập nhật field.
    x = x.toUpperCase();
    // 'suffix' không tạo field, nên nó vẫn tham chiếu đến parameter.
    print('$x$suffix');
  }
}
```

Hành vi nhất quán này giúp việc refactor giữa
constructor truyền thống và primary constructor đơn giản, an toàn hơn.

<a id="add-constructor-bodies"></a>
## Thêm phần thân constructor

Để validate input hoặc thực hiện quá trình khởi tạo phức tạp,
bạn có thể thêm phần thân cho primary constructor bên trong định nghĩa lớp.
Phần thân này dùng keyword `this` theo sau là một block:

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (constructor-bodies)" replace="/PointWithBody/Point/g"?>
```dart
class Point(var int x, var int y) {
  this : assert(x >= 0 && y >= 0) {
    print('Point initialized at ($x, $y)');
  }
}
```

Block có thể chỉ định initializer list sau `this` và/hoặc phần thân function.
Để chỉ cung cấp initializer list,
kết thúc nó bằng dấu chấm phẩy, chẳng hạn `this : assert(x >= 0);`.
Bạn cũng có thể annotate block bằng metadata,
chẳng hạn `@metadata this;`.

<a id="initialize-private-fields"></a>
## Khởi tạo field private

Để khởi tạo một field private bằng named parameter,
bạn có thể viết phần code gán thủ công lặp lại trong constructor truyền thống:

<?code-excerpt "language/lib/primary_constructors/private_named_parameters.dart (private-named-old)" replace="/UserOld/User/g"?>
```dart
// Biến thể không dùng private named parameter.
class User({required String name}) {
  String _name = name;
}
```

Với primary constructor và tính năng private named parameter,
bạn có thể khai báo field private trực tiếp trong header của constructor.
Khi dùng tên private (có dấu gạch dưới ở đầu) cho named parameter,
compiler tự động làm cho tên parameter trở thành public đối với bên gọi
bằng cách loại bỏ dấu gạch dưới:

<?code-excerpt "language/lib/primary_constructors/private_named_parameters.dart (private-named-new)" replace="/UserNew/User/g"?>
```dart
// Biến thể dùng private named parameter.
class User({required var String _name});
```

Trong cả hai trường hợp, bên gọi dùng tên public `name`
tại call site: `User(name: 'John Doe')`.

<a id="empty-bodies"></a>
## Phần thân rỗng

Phần thân rỗng của class, mixin class, extension hoặc extension type (`{}`)
có thể được thay bằng dấu chấm phẩy (`;`).
Mặc dù điều này đúng nói chung với các khai báo này,
nó đặc biệt hữu ích khi dùng primary constructor để
giữ toàn bộ khai báo trên một dòng duy nhất.

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (empty-bodies)" replace="/EmptyBodyPoint/Point/g"?>
```dart
class Point(var int x, var int y);
```

<a id="constant-primary-constructors"></a>
## Constant primary constructor

Giống như constructor truyền thống,
primary constructor có thể là constant
nếu lớp và các field của nó cho phép.
Để khai báo một constant primary constructor,
đặt modifier `const` trước tên lớp trong header của lớp:

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (const-constructor)"?>
```dart
class const ConstPoint(final int x, final int y) {
  final int z;
  // Constant primary constructor có thể có initializer list,
  // nhưng không thể có body block.
  this : z = x + y;
}
```

Constant primary constructor có các ràng buộc quan trọng sau:

*   **Không có body block**:
    Có phần thân `{ ... }` là lỗi compile-time, ngay cả khi nó rỗng.
    Constant primary constructor chỉ có thể dùng
    initializer list theo sau bởi dấu chấm phẩy.
*   **Field final được khởi tạo chắc chắn**:
    Giống mọi lớp có generative const constructor,
    mọi instance variable phải là `final`, không thể là `late`,
    và phải được khởi tạo chắc chắn bởi declaring parameter,
    field initializer hoặc initializer list của primary constructor.
*   **Initializer [có khả năng là hằng số][potentially constant]**:
    Biểu thức khởi tạo cho mỗi instance variable
    phải có khả năng là hằng số.
    Điều này bao gồm field initializer và
    biểu thức trong initializer list của primary constructor.

<a id="named-primary-constructors"></a>
## Named primary constructor

Bạn cũng có thể khai báo primary constructor dưới dạng
**named constructor** bằng cách thêm dấu chấm (`.`) và
một tên sau tên lớp trong header của lớp:

<?code-excerpt "language/lib/primary_constructors/point.dart (point-named)" replace="/PointNamed/Point/g"?>
```dart
// Một named primary constructor.
class Point.custom(var int x, var int y);
```

Một pattern phổ biến là định nghĩa một
primary constructor private (chẳng hạn `Point._`) nhằm
hạn chế việc khởi tạo trực tiếp và
buộc bên gọi dùng factory method hoặc constructor khác:

<?code-excerpt "language/lib/primary_constructors/point.dart (point-private)" replace="/PointPrivate/Point/g"?>
```dart
// Một named primary constructor private.
class Point._(var int x, var int y);
```

<a id="super-parameters"></a>
## Super parameter

Super parameter hoạt động giống như trong constructor truyền thống,
cho phép bạn chuyển tiếp parameter đến constructor của superclass:

<?code-excerpt "language/lib/primary_constructors/super_parameters.dart (super-parameters)"?>
```dart
class Person(final String name, final int age);

class Employee(super.name, super.age, final String role) extends Person;
```

Điều này giảm code lặp lại trong các cấu trúc lớp phân cấp,
loại bỏ nhu cầu viết initializer list theo cách thủ công
hoặc lặp lại khai báo parameter.

<a id="enum-primary-constructors"></a>
## Primary constructor của enum

Bạn có thể dùng primary constructor để khai báo
[enhanced enum][] ngắn gọn hơn nhiều.

Bằng cách dùng primary constructor,
bạn có thể định nghĩa field của enum và constructor của nó trên một dòng,
loại bỏ phần code lặp lại thường có khi
khai báo field, parameter và khởi tạo chúng:

<?code-excerpt "language/lib/primary_constructors/primary_constructors.dart (enums)"?>
```dart
enum Color(final String hex) {
  red('#FF0000'),
  green('#00FF00'),
  blue('#0000FF');
}
```

Primary constructor trong enum **ngầm là constant**.
Mặc dù bạn có thể tùy chọn viết modifier `const` trước tên enum
(như trong `enum const Color`), nó là dư thừa và có thể được bỏ qua.

[enhanced enums]: /language/enums#declaring-enhanced-enums

<a id="constraints-and-breaking-changes"></a>
## Ràng buộc và breaking change

Hãy ghi nhớ các ràng buộc và lỗi tiềm ẩn sau
khi dùng primary constructor:

*   **Declaring parameter không thể là `late` hoặc `external`**:
    Các modifier `late` và `external` không được phép
    trên parameter trong header của primary constructor.
    Để dùng các modifier này, hãy khai báo field trong phần thân lớp như thường lệ.
*   **Xung đột tên**:
    Khai báo một parameter trong primary constructor
    có cùng tên với method hoặc field khác trong phần thân lớp
    sẽ dẫn đến lỗi compile-time.
*   **Không gán cho parameter của primary constructor**:
    Parameter của primary constructor là read-only
    trong primary initializer scope.
    Việc gán cho chúng (chẳng hạn bằng `x = 5` hoặc `x++`) trong field initializer hoặc
    initializer list của primary constructor là lỗi compile-time.
*   **Khởi tạo hai lần**:
    Bạn không thể khởi tạo một instance variable đồng thời trong
    khai báo của nó và trong initializer list của primary constructor
    (hoặc dưới dạng initializing formal parameter), ngay cả khi field là mutable.
    Làm vậy sẽ dẫn đến lỗi compile-time.
*   **Ràng buộc của phần body**:
    *   Phần body của primary constructor (block `this`) không thể
        dùng các modifier `async`, `async*` hoặc `sync*`, và
        không thể dùng cú pháp phần thân biểu thức arrow (`=>`).
    *   Bạn không thể viết block `this` nếu header của lớp
        không khai báo primary constructor.
    *   Một lớp chỉ có thể có tối đa một phần body của primary constructor.
*   **Primary constructor của mixin class phải đơn giản**:
    Mixin class chỉ có thể khai báo primary constructor nếu nó
    không có parameter, initializer list và phần thân.
*   **Covariant parameter**:
    Bạn chỉ có thể dùng modifier `covariant` trên
    parameter của primary constructor nếu đó là
    declaring parameter mutable (dùng modifier `var`).
    Dùng `covariant` trên parameter `final` hoặc non-declaring là
    lỗi compile-time vì chúng không tạo setter.

:::warning
**Breaking change và trường hợp biên quan trọng:**

*   **Hạn chế `final` và `var` trong parameter của function thông thường**:
    Với primary constructor,
    việc dùng `final` hoặc `var` trên formal parameter trong function thông thường
    trở thành lỗi compile-time.
    Chúng được dành riêng cho
    declaring parameter trong primary constructor.
    Lưu ý rằng các lint `avoid_final_parameters` và
    `var_with_no_type_annotation` chỉ hoạt động với
    [language version][] 3.12 trở xuống.
    Để bắt buộc parameter bất biến như một lựa chọn style trong Dart 3.13 trở lên,
    hãy dùng linter rule [`parameter_assignments`][].
*   **Trường hợp biên của method `factory`**:
    Nếu bạn có một method tên `factory` không có return type
    (ví dụ `factory() {}`),
    compiler sẽ parse nó như một factory constructor
    sau khi primary constructor được phát hành.
    Hãy bảo đảm những method như vậy có return type tường minh
    để tránh xung đột này.
:::

[language version]: /language/versioning
[`parameter_assignments`]: /tools/linter-rules/parameter_assignments
[potentially constant]: /resources/glossary#potentially-constant
