---
title: Các kiểu dựng sẵn
description: Thông tin về các kiểu được Dart hỗ trợ.
prevpage:
  url: /language/comments
  title: Chú thích
nextpage:
  url: /language/records
  title: Record
---

Ngôn ngữ Dart hỗ trợ đặc biệt những kiểu sau:

- [Số](#numbers) (`int`, `double`)
- [Chuỗi](#strings) (`String`)
- [Boolean](#booleans) (`bool`)
- [Record][] (`(value1, value2)`)
- [Hàm][] (`Function`)
- [List][] (`List`, còn được gọi là *array*)
- [Set][] (`Set`)
- [Map][] (`Map`)
- [Rune](#runes-and-grapheme-clusters) (`Runes`; thường được thay thế bằng API `characters`)
- [Symbol](#symbols) (`Symbol`)
- Giá trị `null` (`Null`)

Sự hỗ trợ này bao gồm khả năng tạo đối tượng bằng literal.
Ví dụ, `'this is a string'` là một string literal
và `true` là một boolean literal.

Vì mọi biến trong Dart đều tham chiếu đến một đối tượng—một instance của
*lớp*—nên thông thường bạn có thể sử dụng *constructor* để khởi tạo biến. Một số
kiểu dựng sẵn có constructor riêng. Ví dụ, bạn có thể
sử dụng constructor `Map()` để tạo một map.

Một số kiểu khác cũng có vai trò đặc biệt trong ngôn ngữ Dart:

* `Object`: Superclass của mọi lớp Dart ngoại trừ `Null`.
* `Enum`: Superclass của mọi enum.
* `Future` và `Stream`: Được sử dụng trong [lập trình bất đồng bộ][].
* `Iterable`: Được sử dụng trong [vòng lặp for-in][iteration] và
  trong [hàm generator][] đồng bộ.
* `Never`: Cho biết một biểu thức không bao giờ có thể
  đánh giá xong một cách thành công.
  Thường được dùng nhất cho các hàm luôn ném exception.
* `dynamic`: Cho biết bạn muốn tắt kiểm tra tĩnh.
  Thông thường, bạn nên sử dụng `Object` hoặc `Object?` thay thế.
* `void`: Cho biết một giá trị không bao giờ được sử dụng.
  Thường được dùng làm kiểu trả về.

Các lớp `Object`, `Object?`, `Null` và `Never`
có vai trò đặc biệt trong hệ thống phân cấp lớp.
Hãy tìm hiểu những vai trò này trong [Tìm hiểu null safety][].

{% comment %}
Nếu chúng ta quyết định trình bày thêm về `dynamic`,
đây là một ví dụ hay minh họa tác dụng của dynamic:
  dynamic a = 2;
  String b = a; // Không có vấn đề gì! Cho đến runtime, khi bạn gặp một lỗi không được bắt.

  Object c = 2;
  String d = c;  // Có vấn đề!
{% endcomment %}


<a id="numbers"></a>

## Số

Số trong Dart có hai dạng:

[`int`][]

:   Các giá trị số nguyên không lớn hơn 64 bit,
    [tùy thuộc vào nền tảng][dart-numbers].
    Trên nền tảng native, giá trị có thể nằm trong khoảng
    -2<sup>63</sup> đến 2<sup>63</sup> - 1.
    Trên web, giá trị số nguyên được biểu diễn dưới dạng số JavaScript
    (giá trị dấu phẩy động 64 bit không có phần thập phân)
    và có thể nằm trong khoảng -2<sup>53</sup> đến 2<sup>53</sup> - 1.

[`double`][]

:   Số dấu phẩy động 64 bit (độ chính xác kép), theo quy định của
    tiêu chuẩn IEEE 754.

Cả `int` và `double` đều là subtype của [`num`][].
Kiểu `num` bao gồm các toán tử cơ bản như +, -, / và \*,
đồng thời cũng là nơi bạn tìm thấy `abs()`, `ceil()`
và `floor()`, cùng nhiều phương thức khác.
(Các toán tử bitwise, chẳng hạn như \>\>, được định nghĩa trong lớp `int`.)
Nếu `num` và các subtype của nó không có thứ bạn đang tìm kiếm, thư viện
[`dart:math`][] có thể có.

Số nguyên là số không có dấu thập phân. Sau đây là một số ví dụ về
việc định nghĩa integer literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (integer-literals)"?>
```dart
var x = 1;
var hex = 0xDEADBEEF;
```

Nếu một số có phần thập phân, đó là double. Sau đây là một số ví dụ
về việc định nghĩa double literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (double-literals)"?>
```dart
var y = 1.1;
var exponents = 1.42e5;
```

Bạn cũng có thể khai báo một biến là `num`. Nếu làm như vậy, biến đó
có thể mang cả giá trị số nguyên lẫn double.

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (declare-num)"?>
```dart
num x = 1; // x có thể mang cả giá trị int lẫn double
x += 2.5;
```

Integer literal được tự động chuyển đổi thành double khi cần:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (int-to-double)"?>
```dart
double z = 1; // Tương đương với double z = 1.0.
```

Sau đây là cách chuyển chuỗi thành số hoặc ngược lại:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (number-conversion)"?>
```dart
// String -> int
var one = int.parse('1');
assert(one == 1);

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');
```

Kiểu `int` định nghĩa các toán tử truyền thống gồm dịch bit (`<<`, `>>`, `>>>`),
phép bù (`~`), AND (`&`), OR (`|`) và XOR (`^`),
rất hữu ích để thao tác và mask các flag trong bit field.
Ví dụ:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (bit-shifting)"?>
```dart
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 | 4) == 7); // 0011 | 0100 == 0111
assert((3 & 4) == 0); // 0011 & 0100 == 0000
```

Để xem thêm ví dụ, hãy xem phần
[toán tử bitwise và toán tử dịch][].

Number literal là hằng số tại thời điểm biên dịch.
Nhiều biểu thức số học cũng là hằng số tại thời điểm biên dịch,
miễn là các toán hạng của chúng là
hằng số tại thời điểm biên dịch được đánh giá thành số.

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (const-num)"?>
```dart
const msPerSecond = 1000;
const secondsUntilRetry = 5;
const msUntilRetry = secondsUntilRetry * msPerSecond;
```

Để biết thêm thông tin, hãy xem [Số trong Dart][dart-numbers].

<a id="digit-separators"></a>

Bạn có thể sử dụng một hoặc nhiều dấu gạch dưới (`_`) làm dấu phân cách chữ số
để các number literal dài dễ đọc hơn.
Nhiều dấu phân cách chữ số cho phép phân nhóm ở cấp độ cao hơn.

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (digit-separators)"?>
```dart
var n1 = 1_000_000;
var n2 = 0.000_000_000_01;
var n3 = 0x00_14_22_01_23_45; // Địa chỉ MAC
var n4 = 555_123_4567; // Số điện thoại Hoa Kỳ
var n5 = 100__000_000__000_000; // một trăm triệu triệu!
```

:::version-note
Việc sử dụng dấu phân cách chữ số yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 3.6.
:::

<a id="strings"></a>

## Chuỗi

Một chuỗi Dart (đối tượng `String`) chứa một chuỗi các code unit UTF-16.
Bạn có thể sử dụng
dấu nháy đơn hoặc dấu nháy kép để tạo chuỗi:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (quoting)"?>
```dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

<a id="string-interpolation"></a>

Bạn có thể đặt giá trị của một biểu thức bên trong chuỗi bằng
`${`*`expression`*`}`. Nếu biểu thức là một identifier, bạn có thể bỏ
`{}`. Để lấy chuỗi tương ứng với một đối tượng, Dart gọi
phương thức `toString()` của đối tượng đó.

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (string-interpolation)"?>
```dart
var s = 'string interpolation';

assert(
  'Dart has $s, which is very handy.' ==
      'Dart has string interpolation, '
          'which is very handy.',
);
assert(
  'That deserves all caps. '
          '${s.toUpperCase()} is very handy!' ==
      'That deserves all caps. '
          'STRING INTERPOLATION is very handy!',
);
```

:::note
Toán tử `==` kiểm tra xem hai đối tượng có tương đương hay không.
Hai chuỗi tương đương nếu chúng chứa
cùng một chuỗi code unit.
:::

Bạn có thể nối chuỗi bằng các string literal liền kề hoặc toán tử `+`:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (adjacent-string-literals)"?>
```dart
var s1 =
    'String '
    'concatenation'
    " works even over line breaks.";
assert(
  s1 ==
      'String concatenation works even over '
          'line breaks.',
);

var s2 = 'The + operator ' + 'works, as well.';
assert(s2 == 'The + operator works, as well.');
```

Để tạo chuỗi nhiều dòng, hãy sử dụng ba dấu nháy
đơn hoặc kép:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (triple-quotes)"?>
```dart
var s1 = '''
You can create
multi-line strings like this one.
''';

var s2 = """This is also a
multi-line string.""";
```

Bạn có thể tạo chuỗi "raw" bằng cách đặt `r` ở trước chuỗi:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (raw-strings)"?>
```dart
var s = r'In a raw string, not even \n gets special treatment.';
```

Hãy xem [Rune và grapheme cluster](#runes-and-grapheme-clusters) để biết chi tiết về cách
biểu diễn ký tự Unicode trong chuỗi.

String literal là hằng số tại thời điểm biên dịch,
miễn là mọi biểu thức được nội suy đều là hằng số tại thời điểm biên dịch
được đánh giá thành null hoặc một giá trị số, chuỗi hay boolean.

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (string-literals)"?>
```dart
// Những giá trị này dùng được trong chuỗi const.
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';

// Những giá trị này KHÔNG dùng được trong chuỗi const.
var aNum = 0;
var aBool = true;
var aString = 'a string';
const aConstList = [1, 2, 3];

const validConstString = '$aConstNum $aConstBool $aConstString';
// const invalidConstString = '$aNum $aBool $aString $aConstList';
```

Để biết thêm thông tin về cách sử dụng chuỗi, hãy xem
[Chuỗi và biểu thức chính quy](/libraries/dart-core#strings-and-regular-expressions).


<a id="booleans"></a>

## Boolean

Để biểu diễn giá trị boolean, Dart có một kiểu tên là `bool`. Chỉ có hai
đối tượng thuộc kiểu bool: các boolean literal `true` và `false`,
cả hai đều là hằng số tại thời điểm biên dịch.

Type safety của Dart có nghĩa là bạn không thể sử dụng code như
<code>if (<em>nonBooleanValue</em>)</code> hoặc
<code>assert (<em>nonBooleanValue</em>)</code>.
Thay vào đó, hãy kiểm tra tường minh các giá trị như sau:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (no-truthy)"?>
```dart
// Kiểm tra chuỗi rỗng.
var fullName = '';
assert(fullName.isEmpty);

// Kiểm tra số 0.
var hitPoints = 0;
assert(hitPoints == 0);

// Kiểm tra null.
var unicorn = null;
assert(unicorn == null);

// Kiểm tra NaN.
var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);
```

<a id="runes-and-grapheme-clusters"></a>

## Rune và grapheme cluster

Trong Dart, [rune][] cung cấp các code point Unicode của một chuỗi.
Bạn có thể sử dụng [package characters][]
để xem hoặc thao tác những ký tự mà người dùng cảm nhận,
còn được gọi là
[grapheme cluster Unicode (mở rộng).][grapheme cluster]

Unicode định nghĩa một giá trị số duy nhất cho mỗi chữ cái, chữ số
và ký hiệu được sử dụng trong mọi hệ thống chữ viết trên thế giới.
Vì chuỗi Dart là một chuỗi các code unit UTF-16,
việc biểu diễn code point Unicode trong chuỗi đòi hỏi
cú pháp đặc biệt.
Cách thông thường để biểu diễn một code point Unicode là
`\uXXXX`, trong đó XXXX là một giá trị thập lục phân gồm 4 chữ số.
Ví dụ, ký tự trái tim (♥) là `\u2665`.
Để chỉ định nhiều hơn hoặc ít hơn 4 chữ số hex,
hãy đặt giá trị trong dấu ngoặc nhọn.
Ví dụ, emoji cười (😆) là `\u{1f606}`.

Nếu cần đọc hoặc ghi từng ký tự Unicode riêng lẻ,
hãy sử dụng getter `characters` được package characters
định nghĩa trên String.
Đối tượng [`Characters`][] được trả về biểu diễn chuỗi dưới dạng
một chuỗi các grapheme cluster.
Sau đây là ví dụ sử dụng API characters:

<?code-excerpt "misc/lib/language_tour/characters.dart"?>
```dart
import 'package:characters/characters.dart';

void main() {
  var hi = 'Hi 🇩🇰';
  print(hi);
  print('The end of the string: ${hi.substring(hi.length - 1)}');
  print('The last character: ${hi.characters.last}');
}
```

Kết quả đầu ra, tùy thuộc vào môi trường của bạn, trông gần giống như sau:

```console
$ dart run bin/main.dart
Hi 🇩🇰
The end of the string: ???
The last character: 🇩🇰
```

Để biết chi tiết về cách sử dụng package characters nhằm thao tác chuỗi,
hãy xem [ví dụ][ví dụ characters] và [tài liệu tham khảo API][API characters]
của package characters.

<a id="symbols"></a>

## Symbol

Một đối tượng [`Symbol`][]
biểu diễn một toán tử hoặc identifier được khai báo trong chương trình Dart. Có thể
bạn sẽ không bao giờ cần sử dụng symbol, nhưng chúng vô cùng hữu ích cho những API
tham chiếu đến identifier bằng tên, bởi vì quá trình minify thay đổi tên identifier
nhưng không thay đổi symbol của identifier.

Để lấy symbol cho một identifier, hãy sử dụng symbol literal, tức là chỉ cần
`#` theo sau bởi identifier:

```plaintext
#radix
#bar
```

{% comment %}
Code từ excerpt sau không thực sự là nội dung được hiển thị trên trang

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (symbols)"?>
```dart
void main() {
  print(Function.apply(int.parse, ['11']));
  print(Function.apply(int.parse, ['11'], {#radix: 16}));
}
```
{% endcomment %}

Symbol literal là hằng số tại thời điểm biên dịch.



[Record]: /language/records
[Hàm]: /language/functions#function-types
[List]: /language/collections#lists
[Set]: /language/collections#sets
[Map]: /language/collections#maps
[lập trình bất đồng bộ]: /language/async
[iteration]: /libraries/dart-core#iteration
[hàm generator]: /language/functions#generators
[Tìm hiểu null safety]: /null-safety/understanding-null-safety#top-and-bottom
[`int`]: {{site.dart-api}}/dart-core/int-class.html
[`double`]: {{site.dart-api}}/dart-core/double-class.html
[`num`]: {{site.dart-api}}/dart-core/num-class.html
[`dart:math`]: {{site.dart-api}}/dart-math/dart-math-library.html
[toán tử bitwise và toán tử dịch]: /language/operators#bitwise-and-shift-operators
[dart-numbers]: /resources/language/number-representation
[rune]: {{site.dart-api}}/dart-core/Runes-class.html
[package characters]: {{site.pub-pkg}}/characters
[grapheme cluster]: https://unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries
[`Characters`]: {{site.pub-api}}/characters/latest/characters/Characters-class.html
[API characters]: {{site.pub-api}}/characters
[ví dụ characters]: {{site.pub-pkg}}/characters/example
[`Symbol`]: {{site.dart-api}}/dart-core/Symbol-class.html
[phiên bản ngôn ngữ]: /language/versioning
