---
title: Chú thích
description: Các loại chú thích khác nhau trong Dart.
prevpage:
  url: /language/operators
  title: Toán tử
nextpage:
  url: /language/built-in-types
  title: Các kiểu dựng sẵn
---

Dart hỗ trợ chú thích một dòng, chú thích nhiều dòng và
documentation comment.


<a id="single-line-comments"></a>

## Chú thích một dòng

Chú thích một dòng bắt đầu bằng `//`. Mọi nội dung nằm giữa `//` và
cuối dòng đều bị trình biên dịch Dart bỏ qua.

<?code-excerpt "misc/lib/language_tour/comments.dart (single-line-comments)"?>
```dart
void main() {
  // TODO: tái cấu trúc thành AbstractLlamaGreetingFactory?
  print('Welcome to my Llama farm!');
}
```

<a id="multi-line-comments"></a>

## Chú thích nhiều dòng

Chú thích nhiều dòng bắt đầu bằng `/*` và kết thúc bằng `*/`. Mọi nội dung
nằm giữa `/*` và `*/` đều bị trình biên dịch Dart bỏ qua (trừ khi
chú thích đó là documentation comment; xem phần tiếp theo). Các chú thích nhiều dòng
có thể lồng nhau.

<?code-excerpt "misc/lib/language_tour/comments.dart (multi-line-comments)"?>
```dart
void main() {
  /*
   * Việc này tốn rất nhiều công sức. Hãy cân nhắc nuôi gà.

  Llama larry = Llama();
  larry.feed();
  larry.exercise();
  larry.clean();
   */
}
```

<a id="documentation-comments"></a>

## Documentation comment

Documentation comment là chú thích nhiều dòng hoặc một dòng bắt đầu
bằng `///` hoặc `/**`. Việc sử dụng `///` trên các dòng liên tiếp có
hiệu quả tương tự một doc comment nhiều dòng.

Bên trong một documentation comment, analyzer bỏ qua toàn bộ văn bản
trừ khi văn bản được đặt trong dấu ngoặc vuông. Bằng cách sử dụng dấu ngoặc vuông, bạn có thể tham chiếu đến
lớp, phương thức, field, biến top-level, hàm và
tham số. Các tên trong dấu ngoặc vuông được phân giải trong lexical scope của
phần tử chương trình đang được lập tài liệu.

Sau đây là ví dụ về các documentation comment có tham chiếu đến những
lớp và đối số khác:

<?code-excerpt "misc/lib/language_tour/comments.dart (doc-comments)"?>
```dart
/// Một loài lạc đà Nam Mỹ đã được thuần hóa (Lama glama).
///
/// Các nền văn hóa Andes đã sử dụng lạc đà không bướu làm thịt và làm động vật
/// thồ hàng từ thời tiền Tây Ban Nha.
///
/// Cũng giống như mọi loài động vật khác, lạc đà không bướu cần ăn,
/// vì vậy đừng quên [feed] cho chúng một ít [Food].
class Llama {
  String? name;

  /// Cho lạc đà không bướu của bạn ăn [food].
  ///
  /// Một con lạc đà không bướu thông thường ăn một kiện cỏ khô mỗi tuần.
  void feed(Food food) {
    // ...
  }

  /// Cho lạc đà không bướu của bạn vận động bằng một [activity] trong
  /// [timeLimit] phút.
  void exercise(Activity activity, int timeLimit) {
    // ...
  }
}
```

Trong tài liệu được tạo cho lớp, `[feed]` trở thành một liên kết
đến tài liệu của phương thức `feed`,
và `[Food]` trở thành một liên kết đến tài liệu của lớp `Food`.

Để phân tích code Dart và tạo tài liệu HTML, bạn có thể sử dụng
công cụ tạo tài liệu của Dart, [`dart doc`](/tools/dart-doc).
Để xem ví dụ về tài liệu đã được tạo, hãy xem
[tài liệu API Dart.]({{site.dart-api}})
Để nhận lời khuyên về cách cấu trúc chú thích, hãy xem
[Effective Dart: Tài liệu.](/effective-dart/documentation)
