---
title: Dart's core libraries
shortTitle: Core library
description: Tìm hiểu về core library và API của Dart.
nextpage:
  url: /libraries/dart-core
  title: dart:core
---

<style>
  th:first-child {
    width: 80%;
  }
</style>

Dart có tập hợp core library phong phú, cung cấp những thành phần thiết yếu cho nhiều
task lập trình hằng ngày như
làm việc với collection object (`dart:collection`),
thực hiện tính toán (`dart:math`)
và encode/decode dữ liệu (`dart:convert`).
Các API bổ sung có trong
[những package thường dùng](/resources/useful-packages).

<a id="library-tour"></a>
## Tour về library

Các hướng dẫn sau trình bày cách dùng những tính năng chính của core library trong Dart.
Chúng chỉ cung cấp tổng quan và hoàn toàn không toàn diện.
Bất cứ khi nào cần thêm chi tiết về library hoặc member của nó,
hãy tham khảo [Dart API reference.][Dart API]

[dart:core](/libraries/dart-core)
: Kiểu dựng sẵn, collection và chức năng cốt lõi khác.
  Library này được tự động import vào mọi chương trình Dart.

[dart:async](/libraries/dart-async)
: Hỗ trợ lập trình bất đồng bộ với các lớp như Future và Stream.

[dart:math](/libraries/dart-math)
: Hằng số và function toán học, cùng một bộ sinh số ngẫu nhiên.

[dart:convert](/libraries/dart-convert)
: Encoder và decoder để chuyển đổi giữa các dạng biểu diễn dữ liệu khác nhau,
  bao gồm JSON và UTF-8.

[dart:io](/libraries/dart-io)
: I/O cho chương trình có thể dùng Dart VM,
  bao gồm ứng dụng Flutter, server và script dòng lệnh.

[dart:js_interop](/interop/js-interop)
: API dành cho interop với nền tảng web.
  Cùng với `package:web`, `dart:js_interop` thay thế `dart:html`.


Như đã đề cập, các trang này chỉ là tổng quan;
chúng chỉ đề cập một số library dart:* và
không đề cập library bên thứ ba.

Để xem tổng quan tất cả library mà Dart hỗ trợ trên những nền tảng khác nhau,
hãy xem danh sách [Library đa nền tảng](#multi-platform-libraries),
[Library nền tảng Native](#native-platform-libraries) và
[Library nền tảng Web](#web-platform-libraries) bên dưới.

Những nơi khác để tìm thông tin library là
[website pub.dev]({{site.pub}}) và
[hướng dẫn library dành cho web developer][webdev libraries].
Bạn có thể tìm tài liệu API cho tất cả library dart:* trong
[Dart API reference][Dart API] hoặc nếu đang dùng Flutter,
[Flutter API reference][api-flutter].

Để tìm hiểu thêm về ngôn ngữ Dart,
hãy xem [tài liệu và ví dụ ngôn ngữ](/language).

[Dart API]: {{site.dart-api}}
[webdev libraries]: /web/libraries
[api-flutter]: {{site.flutter-api}}

<a id="multi-platform-libraries"></a>
## Library đa nền tảng

Bảng sau liệt kê các core library Dart hoạt động trên mọi
[nền tảng Dart](/overview#platform).

| Library                                       | Ghi chú                         |
|-----------------------------------------------|-------------------------------|
| [`dart:core`][dart-core]<br>Kiểu dựng sẵn, collection và chức năng cốt lõi khác cho mọi chương trình Dart. | |
| [`dart:async`][dart-async], [`package:async`][package-async]<br>Hỗ trợ lập trình bất đồng bộ với các lớp như `Future` và `Stream`.<br>`package:async` cung cấp tiện ích bổ sung quanh các kiểu `Future` và `Stream`. | |
| [`dart:collection`][dart-collection], [`package:collection`][package-collection]<br>Các lớp và tiện ích bổ sung cho khả năng hỗ trợ collection trong `dart:core`.<br>`package:collection` cung cấp thêm implementation collection và function để làm việc trên cũng như với collection. | |
| [`dart:convert`][dart-convert], [`package:convert`][package-convert]<br>Encoder và decoder để chuyển đổi giữa những dạng biểu diễn dữ liệu khác nhau, bao gồm JSON và UTF-8.<br>`package:convert` cung cấp thêm encoder và decoder. ||
| [`dart:developer`][dart-developer]<br>Tương tác với công cụ developer như debugger và inspector. | Chỉ [Native JIT][jit] và [compiler JavaScript development][development JavaScript compiler] |
| [`dart:math`][dart-math]<br>Hằng số và function toán học, cùng bộ sinh số ngẫu nhiên. | |
| [`dart:typed_data`][dart-typed_data], [`package:typed_data`][package-typed_data]<br>List xử lý hiệu quả dữ liệu có kích thước cố định (ví dụ integer 8 byte không dấu) và kiểu số SIMD.<br>`package:typed_data` cung cấp thêm lớp và function làm việc với typed data. | |

{:.table .table-striped}

<a id="native-platform-libraries"></a>
## Library nền tảng Native

Bảng sau liệt kê các core library Dart hoạt động trên
[nền tảng Dart native](/overview#native-platform) (code được compile AOT và JIT).

| Library                                       | Ghi chú                         |
|-----------------------------------------------|-------------------------------|
| [`dart:ffi`][dart-ffi], [`package:ffi`][package-ffi]<br>Foreign function interface cho phép code Dart dùng API C native.<br>`package:ffi` chứa tiện ích, bao gồm hỗ trợ chuyển đổi giữa string Dart và string C. | |
| [`dart:io`][dart-io], [`package:io`][package-io]<br>Hỗ trợ file, socket, HTTP và I/O khác cho ứng dụng không phải web.<br>`package:io` cung cấp chức năng bao gồm hỗ trợ màu ANSI, sao chép file và exit code tiêu chuẩn. | |
| [`dart:isolate`][dart-isolate]<br>Lập trình concurrent bằng isolate: các worker độc lập tương tự thread. | |
| [`dart:mirrors`][dart-mirrors]<br>Reflection cơ bản có hỗ trợ introspection và lời gọi dynamic. | Thử nghiệm<br>Chỉ [Native JIT][jit] (_không phải_&nbsp;Flutter) |

{:.table .table-striped}

<a id="web-platform-libraries"></a>
## Library nền tảng Web

Bảng sau liệt kê các core library Dart hoạt động trên
[nền tảng Dart web](/overview#web-platform) (code được compile thành JavaScript).
Các công cụ mới nhất được khuyến nghị được **in đậm**, còn công cụ cũ được *in nghiêng*
(xem [Khả năng tương tác JavaScript][JavaScript interoperability] để biết thêm thông tin).

| Library                                       | Ghi chú                         |
|-----------------------------------------------|-------------------------------|
| [**`package:web`**][pkg-web] <br>Binding API trình duyệt nhẹ được xây dựng quanh JS interop | Thay thế tất cả library web `dart:*`. Đọc [hướng dẫn migration][html-web]. |
| [**`dart:js_interop`**][js-interop] <br>Interop với JavaScript và API trình duyệt. | Thay thế `package:js`. |
| [**`dart:js_interop_unsafe`**][js-interop-unsafe] <br>Method tiện ích để thao tác dynamic với object JavaScript. | Thay thế `dart:js_util`. |
| [*`dart:html`*][dart-html] *(cũ)* <br>HTML element và resource khác cho ứng dụng web. | Dùng `package:web` thay thế. |
| [*`dart:indexed_db`*][dart-indexed_db] *(cũ)* <br>Kho key-value phía client có hỗ trợ index. | Dùng `package:web` thay thế. |
| [*`dart:js`*][dart-js], [*`dart:js_util`*][dart-js-util], [*`package:js`*][package-js] *(cũ)* <br>Primitive cấp thấp và annotation cấp cao hơn cho JS interop. | Dùng `dart:js_interop` hoặc `dart:js_interop_unsafe` thay thế. |
| [*`dart:svg`*][dart-svg] *(cũ)* <br>Scalable Vector Graphics. | Dùng `package:web` thay thế. |
| [*`dart:web_audio`*][dart-web_audio] *(cũ)* <br>Lập trình audio độ trung thực cao trong trình duyệt. | Dùng `package:web` thay thế. |
| [*`dart:web_gl`*][dart-web_gl] *(cũ)* <br>Lập trình 3D trong trình duyệt. | Dùng `package:web` thay thế. |

{:.table .table-striped}


<!---
Library đa nền tảng
-->
[dart-core]: {{site.dart-api}}/dart-core/dart-core-library.html
[dart-async]: {{site.dart-api}}/dart-async/dart-async-library.html
[package-async]: {{site.pub-pkg}}/async
[dart-collection]: {{site.dart-api}}/dart-collection/dart-collection-library.html
[package-collection]: {{site.pub-pkg}}/collection
[dart-convert]: {{site.dart-api}}/dart-convert/dart-convert-library.html
[package-convert]: {{site.pub-pkg}}/convert
[dart-developer]: {{site.dart-api}}/dart-developer/dart-developer-library.html
[dart-math]: {{site.dart-api}}/dart-math/dart-math-library.html
[dart-typed_data]: {{site.dart-api}}/dart-typed_data/dart-typed_data-library.html
[package-typed_data]: {{site.pub-pkg}}/typed_data

<!---
Library nền tảng Native
-->
[dart-ffi]: {{site.dart-api}}/dart-ffi/dart-ffi-library.html
[package-ffi]: {{site.pub-pkg}}/ffi
[dart-io]: {{site.dart-api}}/dart-io/dart-io-library.html
[package-io]: {{site.pub-pkg}}/io
[dart-isolate]: {{site.dart-api}}/dart-isolate/dart-isolate-library.html
[dart-mirrors]: {{site.dart-api}}/dart-mirrors/dart-mirrors-library.html

<!---
Library nền tảng Web
-->
[pkg-web]: {{site.pub-pkg}}/web
[js-interop]: {{site.dart-api}}/dart-js_interop/dart-js_interop-library.html
[js-interop-unsafe]: {{site.dart-api}}/dart-js_interop_unsafe/dart-js_interop_unsafe-library.html
[dart-html]: {{site.dart-api}}/dart-html/dart-html-library.html
[dart-indexed_db]: {{site.dart-api}}/dart-indexed_db/dart-indexed_db-library.html
[dart-js]: {{site.dart-api}}/dart-js/dart-js-library.html
[package-js]: {{site.pub-pkg}}/js
[dart-js_util]: {{site.dart-api}}/dart-js_util/dart-js_util-library.html
[dart-svg]: {{site.dart-api}}/dart-svg/dart-svg-library.html
[dart-web_audio]: {{site.dart-api}}/dart-web_audio/dart-web_audio-library.html
[dart-web_gl]: {{site.dart-api}}/dart-web_gl/dart-web_gl-library.html

<!---
Khác
-->
[development JavaScript compiler]: /tools/webdev#serve
[jit]: /overview#native-platform
[JavaScript interoperability]: /interop/js-interop
[html-web]: /interop/js-interop/package-web
