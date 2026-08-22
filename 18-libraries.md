---
title: Library và import
shortTitle: Library
description: Hướng dẫn import và triển khai library.
prevpage:
  url: /language/metadata
  title: Metadata
nextpage:
  url: /language/classes
  title: Lớp
---

Các directive `import` và `library` có thể giúp bạn tạo một codebase
modular và có thể chia sẻ. Library không chỉ cung cấp API mà còn
là một đơn vị riêng tư: identifier bắt đầu bằng dấu gạch dưới (`_`)
chỉ hiển thị bên trong library. *Mọi file Dart (cùng các part của nó) đều là một
[library][]*, ngay cả khi không sử dụng directive [`library`](#library-directive).

Library có thể được phân phối bằng [package](/tools/pub/packages).

Dart sử dụng dấu gạch dưới thay cho keyword access modifier
như `public`, `protected` hoặc `private`.
Mặc dù keyword access modifier trong những ngôn ngữ khác
cung cấp khả năng kiểm soát chi tiết hơn,
việc Dart sử dụng dấu gạch dưới và quyền riêng tư dựa trên library
cung cấp một cơ chế cấu hình đơn giản,
giúp cho phép triển khai hiệu quả [truy cập dynamic][]
và cải thiện tree shaking (loại bỏ dead code).

[library]: /resources/glossary#library
[truy cập dynamic]: /effective-dart/design#avoid-using-dynamic-unless-you-want-to-disable-static-checking

<a id="using-libraries"></a>

## Sử dụng library

Sử dụng `import` để chỉ định cách namespace từ một library được dùng trong
scope của một library khác.

Ví dụ, ứng dụng web Dart thường sử dụng library [`dart:js_interop`][],
có thể được import như sau:

<?code-excerpt "misc/test/language_tour/browser_test.dart (dart-js-interop-import)"?>
```dart
import 'dart:js_interop';
```

Đối số bắt buộc duy nhất của `import` là một URI chỉ định
library.
Đối với library dựng sẵn, URI có scheme `dart:` đặc biệt.
Đối với library khác, bạn có thể sử dụng đường dẫn hệ thống file hoặc scheme
`package:`. Scheme `package:` chỉ định các library do package
manager như công cụ pub cung cấp. Ví dụ:

<?code-excerpt "misc/test/language_tour/browser_test.dart (package-import)"?>
```dart
import 'package:test/test.dart';
```

:::note
*URI* là viết tắt của uniform resource identifier.
*URL* (uniform resource locator) là một loại URI phổ biến.
:::

<a id="specifying-a-library-prefix"></a>

### Chỉ định tiền tố library

Nếu bạn import hai library có identifier xung đột, bạn có thể
chỉ định tiền tố cho một hoặc cả hai library. Ví dụ, nếu library1
và library2 đều có lớp Element, bạn có thể có code như sau:

<?code-excerpt "misc/lib/language_tour/libraries/import_as.dart" replace="/(lib\d)\.dart/package:$1\/$&/g"?>
```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

// Sử dụng Element từ lib1.
Element element1 = Element();

// Sử dụng Element từ lib2.
lib2.Element element2 = lib2.Element();
```

Tiền tố import có tên [wildcard][] `_` là non-binding,
nhưng sẽ cung cấp quyền truy cập vào các extension không private trong library đó.

[wildcard]: /language/variables#wildcard-variables

<a id="importing-only-part-of-a-library"></a>

### Chỉ import một phần của library

Nếu chỉ muốn sử dụng một phần library, bạn có thể import có chọn lọc
library đó. Ví dụ:

<?code-excerpt "misc/lib/language_tour/libraries/show_hide.dart (imports)" replace="/(lib\d)\.dart/package:$1\/$&/g"?>
```dart
// Chỉ import foo.
import 'package:lib1/lib1.dart' show foo;

// Import tất cả tên NGOẠI TRỪ foo.
import 'package:lib2/lib2.dart' hide foo;
```

<a id="lazily-loading-a-library"></a>

#### Lazy load một library

*Deferred loading* (còn gọi là *lazy loading*)
cho phép ứng dụng web load một library theo yêu cầu,
nếu và khi cần library đó.
Sử dụng deferred loading khi bạn muốn đáp ứng một hoặc nhiều nhu cầu sau.

* Giảm thời gian khởi động ban đầu của ứng dụng web.
* Thực hiện A/B testing—ví dụ, thử
  các phần triển khai thay thế của một thuật toán.
* Load chức năng hiếm khi dùng, chẳng hạn như màn hình và dialog tùy chọn.

Điều đó không có nghĩa Dart load tất cả component deferred tại thời điểm khởi động.
Ứng dụng web có thể tải component deferred qua web khi cần.

Công cụ `dart` không hỗ trợ deferred loading cho target ngoài web.
Nếu bạn đang build ứng dụng Flutter,
hãy tham khảo phần triển khai deferred loading trong hướng dẫn Flutter về
[component deferred][flutter-deferred].

[flutter-deferred]: {{site.flutter-docs}}/perf/deferred-components

Để lazy load một library, trước tiên hãy import nó bằng `deferred as`.

<?code-excerpt "misc/lib/language_tour/libraries/greeter.dart (import)" replace="/hello\.dart/package:greetings\/$&/g"?>
```dart
import 'package:greetings/hello.dart' deferred as hello;
```

Khi cần library, hãy gọi
`loadLibrary()` bằng identifier của library.

<?code-excerpt "misc/lib/language_tour/libraries/greeter.dart (load-library)"?>
```dart
Future<void> greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

Trong đoạn code trên,
keyword `await` tạm dừng thực thi cho đến khi library được load.
Để biết thêm thông tin về `async` và `await`,
hãy xem [lập trình bất đồng bộ](/language/async).

Bạn có thể gọi `loadLibrary()` nhiều lần trên một library mà không gặp vấn đề.
Library chỉ được load một lần.

Hãy ghi nhớ những điều sau khi sử dụng deferred loading:

* Các hằng số của library deferred không phải hằng số trong file import.
  Hãy nhớ rằng những hằng số này không tồn tại cho đến khi library deferred được load.
* Bạn không thể sử dụng kiểu từ library deferred trong file import.
  Thay vào đó, hãy cân nhắc chuyển các kiểu interface sang một library được import bởi
  cả library deferred lẫn file import.
* Dart ngầm chèn `loadLibrary()` vào namespace mà bạn định nghĩa
  bằng <code>deferred as <em>namespace</em></code>.
  Hàm `loadLibrary()` trả về
  một [`Future`](/libraries/dart-async#future).

<a id="library-directive"></a>

### Directive `library`

Để chỉ định [doc comment][] hoặc [metadata annotation][] cấp library,
hãy gắn chúng vào khai báo `library` ở đầu file.

<?code-excerpt "misc/lib/effective_dart/docs_good.dart (library-doc)"?>
```dart
/// Một library test thực sự tuyệt vời.
@TestOn('browser')
library;
```

<a id="implementing-libraries"></a>

## Triển khai library

Hãy xem
[Tạo package](/tools/pub/create-packages)
để nhận lời khuyên về cách triển khai package, bao gồm:

* Cách tổ chức source code library.
* Cách sử dụng directive `export`.
* Khi nào nên sử dụng directive `part`.
* Cách sử dụng import và export có điều kiện để triển khai
  một library hỗ trợ nhiều nền tảng.

[`dart:js_interop`]: {{site.dart-api}}/dart-js_interop/dart-js_interop-library.html
[doc comment]: /effective-dart/documentation#consider-writing-a-library-level-doc-comment
[metadata annotation]: /language/metadata
