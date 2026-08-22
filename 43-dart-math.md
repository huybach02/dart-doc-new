---
title: dart:math
description: Tìm hiểu về các tính năng chính trong thư viện dart:math của Dart.
prevpage:
  url: /libraries/dart-async
  title: dart:async
nextpage:
  url: /libraries/dart-convert
  title: dart:convert
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt plaster="none"?>

Thư viện dart:math ([tài liệu tham khảo API][dart:math])
cung cấp các chức năng phổ biến như sine và cosine,
giá trị lớn nhất và nhỏ nhất, cùng các hằng số như *pi* và *e*. Phần lớn
chức năng trong thư viện Math được triển khai dưới dạng các hàm top-level.

Để sử dụng thư viện này trong ứng dụng, hãy import dart:math.

<?code-excerpt "misc/test/library_tour/math_test.dart (import)"?>
```dart
import 'dart:math';
```


<a id="trigonometry"></a>
## Lượng giác

Thư viện Math cung cấp các hàm lượng giác cơ bản:

<?code-excerpt "misc/test/library_tour/math_test.dart (trig)"?>
```dart
// Cosine
assert(cos(pi) == -1.0);

// Sine
var degrees = 30;
var radians = degrees * (pi / 180);
// radians hiện là 0.52359.
var sinOf30degrees = sin(radians);
// sin 30° = 0.5
assert((sinOf30degrees - 0.5).abs() < 0.01);
```

:::note
Các hàm này sử dụng radian, không phải độ!
:::


<a id="maximum-and-minimum"></a>
## Giá trị lớn nhất và nhỏ nhất

Thư viện Math cung cấp các method `max()` và `min()`:

<?code-excerpt "misc/test/library_tour/math_test.dart (min-max)"?>
```dart
assert(max(1, 1000) == 1000);
assert(min(1, -1000) == -1000);
```


<a id="math-constants"></a>
## Các hằng số toán học

Hãy tìm các hằng số bạn yêu thích—*pi*, *e* và nhiều hằng số khác—trong thư viện Math:

<?code-excerpt "misc/test/library_tour/math_test.dart (constants)"?>
```dart
// Xem thư viện Math để biết thêm các hằng số khác.
print(e); // 2.718281828459045
print(pi); // 3.141592653589793
print(sqrt2); // 1.4142135623730951
```


<a id="random-numbers"></a>
## Số ngẫu nhiên

Tạo các số ngẫu nhiên bằng class [Random][]. Bạn có thể
tùy chọn cung cấp một seed cho constructor của Random.

<?code-excerpt "misc/test/library_tour/math_test.dart (random)"?>
```dart
var random = Random();
random.nextDouble(); // Trong khoảng từ 0.0 đến 1.0: [0, 1)
random.nextInt(10); // Trong khoảng từ 0 đến 9.
```

Bạn thậm chí có thể tạo các giá trị boolean ngẫu nhiên:

<?code-excerpt "misc/test/library_tour/math_test.dart (random-bool)"?>
```dart
var random = Random();
random.nextBool(); // true hoặc false
```

:::warning
Triển khai mặc định của `Random` cung cấp một stream các bit giả ngẫu nhiên
không phù hợp cho mục đích mật mã.
Để tạo một trình sinh số ngẫu nhiên an toàn về mặt mật mã,
hãy sử dụng constructor [`Random.secure()`][].
:::

<a id="more-information"></a>
## Thông tin thêm

Tham khảo [tài liệu API của Math][dart:math] để xem danh sách đầy đủ các method.
Đồng thời xem tài liệu tham khảo API cho [num,][num] [int,][int] và [double.][double]

[Random]: {{site.dart-api}}/dart-math/Random-class.html
[`Random.secure()`]: {{site.dart-api}}/dart-math/Random/Random.secure.html
[dart:math]: {{site.dart-api}}/dart-math/dart-math-library.html
[double]: {{site.dart-api}}/dart-core/double-class.html
[int]: {{site.dart-api}}/dart-core/int-class.html
[num]: {{site.dart-api}}/dart-core/num-class.html
