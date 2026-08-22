---
title: Sound null safety
breadcrumb: Null safety
description: Giới thiệu về null safety trong Dart.
---

Ngôn ngữ Dart thực thi sound null safety,
khiến việc vô tình truy cập member trên giá trị `null` trở thành bất khả thi.

Trong Dart, kiểu mặc định là non-nullable.
Biến có kiểu non-nullable phải được khởi tạo
và chỉ có thể được gán giá trị khác null.

Dart analyzer và compiler phát hiện việc sử dụng không an toàn
các giá trị có khả năng là `null` ngay khi chỉnh sửa,
biến những gì vốn là **lỗi runtime** trong ngôn ngữ khác
thành **lỗi phân tích** mà bạn có thể sửa trước khi deploy.

<a id="creating-variables" aria-hidden="true"></a>
<a id="introduction-through-examples" aria-hidden="true"></a>

<a id="examples"></a>
## Ví dụ

Không biến nào trong đoạn code sau có thể là `null`:

```dart
// Không giá trị nào trong số này có thể là null.
var i = 42; // Được suy luận là int.
String name = getFileName();
final b = Foo();
```

Để cho biết một biến có thể có giá trị `null`,
hãy thêm `?` vào khai báo kiểu:

```dart
int? aNullableInt = null;
```

- Để xem ví dụ tương tác, hãy thử [Dart cheatsheet][].
- Để tìm hiểu thêm về null safety, hãy xem
  [Tìm hiểu null safety][Understanding null safety].

[Dart cheatsheet]: /resources/dart-cheatsheet
[Understanding null safety]: /null-safety/understanding-null-safety

<a id="null-safety-principles"></a>
## Các nguyên tắc null safety

Null safety trong Dart được xây dựng trên hai nguyên tắc thiết kế cốt lõi:

**Mặc định là non-nullable**
: Trừ khi bạn nói rõ với Dart rằng biến có thể là null,
  biến đó được xem là non-nullable.
  Giá trị mặc định này được chọn sau khi nghiên cứu cho thấy
  non-null cho đến nay là lựa chọn phổ biến nhất trong API.

**Sound hoàn toàn**
: Nếu hệ thống kiểu xác định rằng
  một biến hoặc biểu thức có kiểu non-nullable,
  nó được bảo đảm không bao giờ có thể đánh giá thành `null` tại runtime.

Kết hợp lại, các nguyên tắc này tạo ra
ít bug hơn, binary nhỏ hơn và thực thi nhanh hơn.

<a id="dart-3-and-null-safety" aria-hidden="true"></a>
<a id="enable-null-safety" aria-hidden="true"></a>
<a id="where-to-learn-more" aria-hidden="true"></a>

## Tài nguyên migration lịch sử {:#migrate}

Dart đã thực thi sound null safety kể từ Dart 3, phát hành tháng 5 năm 2023.
Nếu bạn vẫn cần migrate ứng dụng hoặc package sang null safety,
hãy xem tài liệu được lưu trữ trong repository
[`dart-community/migrate-to-null-safety`][].

[`dart-community/migrate-to-null-safety`]: https://github.com/dart-community/migrate-to-null-safety#migrate-to-dart-null-safety
