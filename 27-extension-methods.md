---
title: Extension methods
description: Tìm hiểu cách bổ sung chức năng cho API hiện có.
prevpage:
  url: /language/dot-shorthands
  title: Dot shorthand
nextpage:
  url: /language/extension-types
  title: Extension type
---

Extension method bổ sung chức năng cho library hiện có.
Bạn có thể đang dùng extension method mà thậm chí không biết.
Ví dụ, khi bạn dùng tính năng hoàn thành code trong IDE,
nó đề xuất extension method cùng với method thông thường.

Nếu việc xem video giúp bạn học,
hãy xem phần tổng quan này về extension method.

<YouTubeEmbed id="D3j0OSfT9ZI" title="Dart extension methods"></YouTubeEmbed>

<a id="overview"></a>
## Tổng quan

Khi dùng API của người khác hoặc
khi implement một library được sử dụng rộng rãi,
việc thay đổi API thường không thực tế hoặc không thể thực hiện.
Nhưng bạn vẫn có thể muốn bổ sung một số chức năng.

Ví dụ, hãy xem đoạn code sau parse một string thành integer:

```dart
int.parse('42')
```

Sẽ rất tiện—ngắn hơn và dễ dùng với công cụ hơn—nếu
chức năng đó nằm trên `String`:

```dart
'42'.parseInt()
```

Để cho phép đoạn code đó,
bạn có thể import một library chứa extension của lớp `String`:

<?code-excerpt "extension_methods/lib/string_extensions/usage_simple_extension.dart (basic)" plaster="none"?>
```dart
import 'string_apis.dart';

void main() {
  print('42'.parseInt()); // Dùng một extension method.
}
```

Extension không chỉ có thể định nghĩa method,
mà còn định nghĩa các member khác như getter, setter và operator.
Ngoài ra, extension có thể có tên, điều này hữu ích nếu xảy ra xung đột API.
Sau đây là cách bạn có thể implement extension method `parseInt()`,
bằng một extension (tên là `NumberParsing`) hoạt động trên string:

<?code-excerpt "extension_methods/lib/string_extensions/string_apis.dart (parseInt)" plaster="none"?>
```dart title="lib/string_apis.dart"
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }
}
```

Mục tiếp theo mô tả cách _sử dụng_ extension method.
Sau đó là các mục về cách _implement_ extension method.


<a id="using-extension-methods"></a>
## Sử dụng extension method

Giống như mọi code Dart, extension method nằm trong library.
Bạn đã thấy cách dùng một extension method—chỉ cần
import library chứa nó và dùng nó như một method thông thường:

<?code-excerpt "extension_methods/lib/string_extensions/usage_simple_extension.dart (import-and-use)" plaster="none"?>
```dart
// Import một library chứa extension trên String.
import 'string_apis.dart';

void main() {
  print('42'.padLeft(5)); // Dùng method của String.
  print('42'.parseInt()); // Dùng extension method.
}
```

Đó thường là tất cả những gì bạn cần biết để dùng extension method.
Khi viết code, bạn cũng có thể cần biết
extension method phụ thuộc vào static type (thay vì `dynamic`) như thế nào và
cách giải quyết [xung đột API](#api-conflicts).

<a id="static-types-and-dynamic"></a>
### Static type và dynamic

Bạn không thể gọi extension method trên biến có kiểu `dynamic`.
Ví dụ, đoạn code sau dẫn đến exception tại runtime:

<?code-excerpt "extension_methods/lib/string_extensions/usage_simple_extension.dart (dynamic)" plaster="none" replace="/  \/\/ print/print/g"?>
```dart
dynamic d = '2';
print(d.parseInt()); // Exception runtime: NoSuchMethodError
```

Extension method _có_ hoạt động với type inference của Dart.
Đoạn code sau hợp lệ vì
biến `v` được suy luận có kiểu `String`:

<?code-excerpt "extension_methods/lib/string_extensions/usage_simple_extension.dart (var)"?>
```dart
var v = '2';
print(v.parseInt()); // Output: 2
```

Lý do `dynamic` không hoạt động là vì
extension method được phân giải dựa trên static type của receiver.
Vì extension method được phân giải tĩnh,
chúng nhanh như khi gọi một function static.

Để biết thêm thông tin về static type và `dynamic`, xem
[Hệ thống kiểu của Dart](/language/type-system).

<a id="api-conflicts"></a>
### Xung đột API

Nếu một extension member xung đột với
interface hoặc một extension member khác,
bạn có một số lựa chọn.

Một lựa chọn là thay đổi cách import extension xung đột,
dùng `show` hoặc `hide` để giới hạn API được hiển thị:

<?code-excerpt "extension_methods/lib/string_extensions/usage_import.dart (hide-conflicts)" plaster="none"?>
```dart
// Định nghĩa extension method parseInt() của String.
import 'string_apis.dart';

// Cũng định nghĩa parseInt(), nhưng việc ẩn NumberParsing2
// sẽ ẩn extension method đó.
import 'string_apis_2.dart' hide NumberParsing2;

void main() {
  // Dùng parseInt() được định nghĩa trong 'string_apis.dart'.
  print('42'.parseInt());
}
```

Một lựa chọn khác là áp dụng extension một cách tường minh,
tạo ra code trông như thể extension là một wrapper class:

<?code-excerpt "extension_methods/lib/string_extensions/usage_explicit.dart (conflicts-explicit)" plaster="none"?>
```dart
// Cả hai library đều định nghĩa extension trên String chứa parseInt(),
// và các extension có tên khác nhau.
import 'string_apis.dart'; // Chứa extension NumberParsing.
import 'string_apis_2.dart'; // Chứa extension NumberParsing2.

void main() {
  // print('42'.parseInt()); // Không hoạt động.
  print(NumberParsing('42').parseInt());
  print(NumberParsing2('42').parseInt());
}
```

Nếu cả hai extension có cùng tên,
bạn có thể cần import bằng prefix:

<?code-excerpt "extension_methods/lib/string_extensions/usage_prefix.dart"?>
```dart
// Cả hai library đều định nghĩa extension tên NumberParsing
// chứa extension method parseInt(). Một extension NumberParsing
// (trong 'string_apis_3.dart') cũng định nghĩa parseNum().
import 'string_apis.dart';
import 'string_apis_3.dart' as rad;

void main() {
  // print('42'.parseInt()); // Không hoạt động.

  // Dùng extension ParseNumbers từ string_apis.dart.
  print(NumberParsing('42').parseInt());

  // Dùng extension ParseNumbers từ string_apis_3.dart.
  print(rad.NumberParsing('42').parseInt());

  // Chỉ string_apis_3.dart có parseNum().
  print('42'.parseNum());
}
```

Như ví dụ cho thấy,
bạn có thể gọi extension method ngầm định ngay cả khi import bằng prefix.
Lần duy nhất bạn cần dùng prefix là
để tránh xung đột tên khi gọi extension tường minh.


<a id="implementing-extension-methods"></a>
## Implement extension method

Dùng cú pháp sau để tạo một extension:

```plaintext
extension <extension name>? on <type> { // <extension-name> là không bắt buộc
  (<member definition>)* // Có thể cung cấp một hoặc nhiều <member definition>.
}
```

Ví dụ, sau đây là cách bạn có thể implement extension trên lớp `String`:

<?code-excerpt "extension_methods/lib/string_extensions/string_apis.dart"?>
```dart title="lib/string_apis.dart"
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }

  double parseDouble() {
    return double.parse(this);
  }

}
```

Member của extension có thể là method, getter, setter hoặc operator.
Extension cũng có thể có field static và helper method static.
Để truy cập member static bên ngoài khai báo extension,
hãy gọi chúng thông qua tên khai báo giống như [biến và method của lớp][class variables and methods].

[class variables and methods]: /language/classes#class-variables-and-methods

<a id="unnamed-extensions"></a>
### Extension không tên

Khi khai báo extension, bạn có thể bỏ tên.
Extension không tên chỉ hiển thị
trong library nơi chúng được khai báo.
Vì không có tên,
chúng không thể được áp dụng tường minh
để giải quyết [xung đột API](#api-conflicts).

<?code-excerpt "extension_methods/lib/string_extensions/string_apis_unnamed.dart (unnamed)"?>
```dart
extension on String {
  bool get isBlank => trim().isEmpty;
}
```

:::note
Bạn chỉ có thể gọi member static của extension không tên
bên trong khai báo extension đó.
:::

<a id="implementing-generic-extensions"></a>
## Implement generic extension

Extension có thể có generic type parameter.
Ví dụ, sau đây là một đoạn code mở rộng kiểu dựng sẵn `List<T>`
bằng một getter, một operator và một method:

<?code-excerpt "extension_methods/lib/fancylist.dart (generic)"?>
```dart
extension MyFancyList<T> on List<T> {
  int get doubleLength => length * 2;
  List<T> operator -() => reversed.toList();
  List<List<T>> split(int at) => [sublist(0, at), sublist(at)];
}
```

Kiểu `T` được bind dựa trên static type của list mà
các method được gọi trên đó.
{% comment %}
VIỆC CẦN LÀM (https://github.com/dart-lang/site-www/issues/2171):
Bổ sung thêm thông tin về generic extension.
Ví dụ, trong đoạn code sau, `T` là `PENDING` vì PENDING:

[PENDING: ví dụ]

[PENDING: Giải thích vì sao điều này quan trọng trong cách sử dụng thông thường.]
{% endcomment %}

<a id="resources"></a>
## Tài nguyên

Để biết thêm thông tin về extension method, xem các tài liệu sau:

* [Bài viết: Kiến thức cơ bản về Dart Extension Method][article]
* [Đặc tả tính năng][specification]
* [Ví dụ extension method][sample]

[specification]: {{site.repo.dart.lang}}/blob/main/accepted/2.7/static-extension-methods/feature-specification.md#dart-static-extension-methods-design
[article]: /blog/dart-extension-method-fundamentals
[sample]: {{site.repo.dart.samples}}/tree/main/extension_methods
