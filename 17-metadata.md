---
title: Metadata
description: Metadata và annotation trong Dart.
prevpage:
  url: /language/functions
  title: Hàm
nextpage:
  url: /language/libraries
  title: Library và import
---


Sử dụng metadata để cung cấp thêm thông tin tĩnh về code.
Một metadata annotation bắt đầu bằng ký tự `@`, theo sau bởi
tham chiếu đến một hằng số tại thời điểm biên dịch (chẳng hạn `deprecated`) hoặc
lời gọi đến một constructor hằng.

Metadata có thể được gắn vào hầu hết cấu trúc chương trình Dart bằng cách
thêm annotation trước phần khai báo hoặc directive của cấu trúc.

<a id="built-in-annotations"></a>

## Annotation dựng sẵn

Các annotation sau khả dụng cho mọi code Dart:

[`@Deprecated`][]
: Đánh dấu một khai báo là deprecated,
  cho biết nên chuyển khỏi khai báo đó,
  kèm thông báo giải thích phần thay thế và ngày có thể bị loại bỏ.

  Ngoài annotation `@Deprecated` chung,
  bạn có thể sử dụng các annotation cụ thể để deprecate một số
  cách sử dụng nhất định của khai báo:

  * [`@Deprecated.extend()`][]: Việc mở rộng lớp bị deprecate.
  * [`@Deprecated.implement()`][]: Việc implement lớp hoặc
    mixin bị deprecate.
  * [`@Deprecated.subclass()`][]: Việc tạo subclass (mở rộng hoặc
    implement) lớp hoặc mixin bị deprecate.
  * [`@Deprecated.mixin()`][]: Việc mix in lớp bị deprecate.
  * [`@Deprecated.instantiate()`][]: Việc tạo instance của lớp bị deprecate.
  * [`@Deprecated.optional()`][]: Việc bỏ qua đối số cho
    tham số bị deprecate.

  Sau đây là ví dụ sử dụng annotation `@Deprecated`:

  <?code-excerpt "misc/lib/language_tour/metadata/television.dart (deprecated)"?>
  ```dart highlightLines=3
  class Television {
    /// Hãy dùng [turnOn] để bật nguồn thay thế.
    @Deprecated('Use turnOn instead')
    void activate() {
      turnOn();
    }

    /// Bật nguồn TV.
    void turnOn() {
      // ···
    }
    // ···
  }
  ```

[`@deprecated`][]
: Đánh dấu một khai báo là deprecated cho đến một bản phát hành tương lai không được chỉ định.
  Nên sử dụng `@Deprecated` và [cung cấp thông báo deprecation][].

[`@override`][]
: Đánh dấu một member instance là phần override hoặc triển khai của
  một member cùng tên từ lớp cha hoặc interface.
  Để xem ví dụ sử dụng `@override`, hãy xem [Mở rộng một lớp][].

[`@pragma`][]
: Cung cấp hướng dẫn hoặc gợi ý cụ thể về một khai báo cho
  các công cụ Dart, chẳng hạn compiler hoặc analyzer.

[Dart analyzer][] cung cấp phản hồi dưới dạng diagnostic nếu
cần annotation `@override` và khi sử dụng
member được annotate bằng `@deprecated` hoặc `@Deprecated`.

[`@Deprecated`]: {{site.dart-api}}/dart-core/Deprecated-class.html
[`@deprecated`]: {{site.dart-api}}/dart-core/deprecated-constant.html
[`@override`]: {{site.dart-api}}/dart-core/override-constant.html
[`@pragma`]: {{site.dart-api}}/dart-core/pragma-class.html
[cung cấp thông báo deprecation]: /tools/linter-rules/provide_deprecation_message
[Mở rộng một lớp]: /language/extend
[Dart analyzer]: /tools/analysis
[`@Deprecated.extend()`]: {{site.dart-api}}/dart-core/Deprecated/Deprecated.extend.html
[`@Deprecated.implement()`]: {{site.dart-api}}/dart-core/Deprecated/Deprecated.implement.html
[`@Deprecated.subclass()`]: {{site.dart-api}}/dart-core/Deprecated/Deprecated.subclass.html
[`@Deprecated.mixin()`]: {{site.dart-api}}/dart-core/Deprecated/Deprecated.mixin.html
[`@Deprecated.instantiate()`]: {{site.dart-api}}/dart-core/Deprecated/Deprecated.instantiate.html
[`@Deprecated.optional()`]: {{site.dart-api}}/dart-core/Deprecated/Deprecated.optional.html

<a id="analyzer-supported-annotations"></a>

## Annotation được analyzer hỗ trợ

Ngoài việc hỗ trợ và phân tích [annotation dựng sẵn][],
[Dart analyzer][] còn hỗ trợ và cung cấp diagnostic bổ sung cho
nhiều annotation từ [`package:meta`][].
Một số annotation thường dùng mà package cung cấp bao gồm:

[`@visibleForTesting`][]
: Đánh dấu một member của package là public chỉ để
  có thể truy cập member từ các test của package.
  Analyzer ẩn member khỏi gợi ý tự động hoàn thành
  và cảnh báo nếu nó được sử dụng từ package khác.

[`@awaitNotRequired`][]
: Đánh dấu các biến có kiểu `Future` hoặc hàm trả về `Future`
  là không yêu cầu caller phải await `Future`.
  Điều này ngăn analyzer cảnh báo caller không await `Future`
  do lint [`discarded_futures`][] hoặc [`unawaited_futures`][].

Để tìm hiểu thêm về những annotation này và các annotation khác do package cung cấp,
điều chúng biểu thị, chức năng chúng bật và cách sử dụng,
hãy xem [tài liệu API `package:meta/meta.dart`][meta-api].

[annotation dựng sẵn]: #built-in-annotations
[Dart analyzer]: /tools/analysis
[`@visibleForTesting`]: {{site.pub-api}}/meta/latest/meta/visibleForTesting-constant.html
[`@awaitNotRequired`]: {{site.pub-api}}/meta/latest/meta/awaitNotRequired-constant.html
[`discarded_futures`]: /tools/linter-rules/discarded_futures
[`unawaited_futures`]: /tools/linter-rules/unawaited_futures
[meta-api]: {{site.pub-api}}/meta/latest/meta/meta-library.html

<a id="custom-annotations"></a>

## Annotation tùy chỉnh

Bạn có thể định nghĩa metadata annotation riêng. Sau đây là ví dụ
định nghĩa một annotation `@Todo` nhận hai đối số:

<?code-excerpt "misc/lib/language_tour/metadata/todo.dart (definition)"?>
```dart
class Todo {
  final String who;
  final String what;

  const Todo(this.who, this.what);
}
```

Và sau đây là ví dụ sử dụng annotation `@Todo` đó:

<?code-excerpt "misc/lib/language_tour/metadata/misc.dart (usage)"?>
```dart highlightLines=1
@Todo('Dash', 'Implement this function')
void doSomething() {
  print('Do something');
}
```

<a id="specifying-supported-targets"></a>

### Chỉ định target được hỗ trợ {:.no_toc}

Để cho biết loại cấu trúc ngôn ngữ
nên được annotate bằng annotation của bạn,
hãy sử dụng annotation [`@Target`][] từ [`package:meta`][].

Ví dụ, nếu bạn muốn annotation `@Todo` trước đó
chỉ được phép dùng trên hàm và phương thức,
bạn sẽ thêm annotation sau:

<?code-excerpt "misc/lib/language_tour/metadata/todo.dart (target-kinds)"?>
```dart highlightLines=3
import 'package:meta/meta_meta.dart';

@Target({TargetKind.function, TargetKind.method})
class Todo {
  // ···
}
```

Với cấu hình này, analyzer sẽ cảnh báo nếu `Todo` được sử dụng làm
annotation trên bất kỳ khai báo nào ngoài hàm top-level hoặc phương thức.

[`@Target`]: {{site.pub-api}}/meta/latest/meta_meta/Target-class.html
[`package:meta`]: {{site.pub-pkg}}/meta
