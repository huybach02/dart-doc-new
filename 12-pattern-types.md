---
title: Pattern types
description: Tài liệu tham khảo về các kiểu pattern trong Dart.
prevpage:
  url: /language/patterns
  title: Pattern
nextpage:
  url: /language/loops
  title: Vòng lặp
---

Trang này là tài liệu tham khảo về các loại pattern khác nhau.
Để xem tổng quan về cách pattern hoạt động, nơi bạn có thể sử dụng chúng trong Dart và các
trường hợp sử dụng phổ biến, hãy truy cập trang [Pattern][] chính.

<a id="pattern-precedence"></a>

#### Độ ưu tiên của pattern

Tương tự [độ ưu tiên của toán tử](/language/operators#operator-precedence-example),
việc đánh giá pattern tuân theo các quy tắc về độ ưu tiên.
Bạn có thể sử dụng [pattern có dấu ngoặc đơn](#parenthesized) để
đánh giá pattern có độ ưu tiên thấp hơn trước.

Tài liệu này liệt kê các kiểu pattern theo thứ tự độ ưu tiên tăng dần:

* Pattern [logical-or](#logical-or) có độ ưu tiên thấp hơn [logical-and](#logical-and),
pattern logical-and có độ ưu tiên thấp hơn pattern [quan hệ](#relational),
và cứ tiếp tục như vậy.

* Các pattern unary postfix ([cast](#cast), [null-check](#null-check)
và [null-assert](#null-assert)) có cùng mức độ ưu tiên.

* Những primary pattern còn lại có cùng độ ưu tiên cao nhất.
Pattern kiểu collection ([record](#record), [list](#list) và [map](#map))
cùng pattern [Object](#object) bao hàm dữ liệu khác,
vì vậy được đánh giá trước dưới dạng outer pattern.

<a id="logical-or"></a>

## Logical-or

`subpattern1 || subpattern2`

Pattern logical-or phân tách các subpattern bằng `||` và khớp nếu bất kỳ
nhánh nào khớp. Các nhánh được đánh giá từ trái sang phải. Khi một nhánh khớp,
các nhánh còn lại không được đánh giá.

<?code-excerpt "language/lib/patterns/pattern_types.dart (logical-or)"?>
```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false,
};
```

Subpattern trong pattern logical-or có thể bind biến, nhưng các nhánh phải
định nghĩa cùng một tập hợp biến, vì chỉ một nhánh được đánh giá khi
pattern khớp.

<a id="logical-and"></a>

## Logical-and

`subpattern1 && subpattern2`

Một cặp pattern phân tách bằng `&&` chỉ khớp nếu cả hai subpattern cùng khớp. Nếu
nhánh trái không khớp, nhánh phải không được đánh giá.

Subpattern trong pattern logical-and có thể bind biến, nhưng các biến trong
mỗi subpattern không được trùng nhau, vì tất cả chúng đều sẽ được bind nếu pattern
khớp:

<?code-excerpt "language/lib/patterns/pattern_types.dart (logical-and)"?>
```dart
switch ((1, 2)) {
  // Lỗi, cả hai subpattern đều cố bind 'b'.
  case (var a, var b) && (var b, var c): // ...
}
```

<a id="relational"></a>

## Quan hệ

`== expression`

`< expression`

Pattern quan hệ so sánh giá trị được so khớp với một hằng số đã cho bằng bất kỳ
toán tử bằng nhau hoặc quan hệ nào: `==`, `!=`, `<`, `>`, `<=` và `>=`.

Pattern khớp khi việc gọi toán tử thích hợp trên giá trị được so khớp
với hằng số làm đối số trả về `true`.

Pattern quan hệ hữu ích để so khớp trên các khoảng số, đặc biệt khi
kết hợp với [pattern logical-and](#logical-and):

<?code-excerpt "language/lib/patterns/pattern_types.dart (relational)"?>
```dart
String asciiCharType(int char) {
  const space = 32;
  const zero = 48;
  const nine = 57;

  return switch (char) {
    < space => 'control',
    == space => 'space',
    > space && < zero => 'punctuation',
    >= zero && <= nine => 'digit',
    _ => '',
  };
}
```

<a id="cast"></a>

## Cast

`foo as String`

Cast pattern cho phép bạn chèn một [phép ép kiểu][] vào giữa quá trình destructuring,
trước khi truyền giá trị cho một subpattern khác:

<?code-excerpt "language/lib/patterns/pattern_types.dart (cast)"?>
```dart
(num, Object) record = (1, 's');
var (i as int, s as String) = record;
```

Cast pattern sẽ [ném exception][] nếu giá trị không có kiểu đã nêu.
Giống [pattern null-assert](#null-assert), điều này cho phép bạn cưỡng chế khẳng định
kiểu mong đợi của một giá trị đã được destructure.

<a id="null-check"></a>

## Null-check

`subpattern?`

Pattern null-check trước tiên khớp nếu giá trị không phải null, sau đó so khớp inner
pattern với chính giá trị đó. Chúng cho phép bạn bind một biến có kiểu là
kiểu cơ sở non-nullable của giá trị nullable đang được so khớp.

Để coi giá trị `null` là trường hợp so khớp thất bại
mà không ném exception, hãy sử dụng pattern null-check.

<?code-excerpt "language/lib/patterns/pattern_types.dart (null-check)"?>
```dart
String? maybeString = 'nullable with base type String';
switch (maybeString) {
  case var s?:
  // Tại đây, 's' có kiểu String non-nullable.
}
```

Để khớp khi giá trị _là_ null, hãy sử dụng [constant pattern](#constant) `null`.

<a id="null-assert"></a>

## Null-assert

`subpattern!`

Pattern null-assert trước tiên khớp nếu đối tượng không phải null, sau đó khớp trên giá trị.
Chúng cho phép giá trị khác null đi qua nhưng sẽ [ném exception][] nếu giá trị được so khớp
là null.

Để bảo đảm giá trị `null` không bị âm thầm xem là trường hợp so khớp thất bại,
hãy sử dụng pattern null-assert trong khi so khớp:

<?code-excerpt "language/lib/patterns/pattern_types.dart (null-assert-match)"?>
```dart
List<String?> row = ['user', null];
switch (row) {
  case ['user', var name!]: // ...
  // Tại đây, 'name' là một chuỗi non-nullable.
}
```

Để loại bỏ giá trị `null` khỏi pattern khai báo biến,
hãy sử dụng pattern null-assert:

<?code-excerpt "language/lib/patterns/pattern_types.dart (null-assert-dec)"?>
```dart
(int?, int?) position = (2, 3);

var (x!, y!) = position;
```

Để khớp khi giá trị _là_ null, hãy sử dụng [constant pattern](#constant) `null`.

<a id="constant"></a>

## Constant

`123, null, 'string', math.pi, SomeClass.constant, const Thing(1, 2), const (1 + 2)`

Constant pattern khớp khi giá trị bằng hằng số:

<?code-excerpt "language/lib/patterns/pattern_types.dart (constant)"?>
```dart
switch (number) {
  // Khớp nếu 1 == number.
  case 1: // ...
}
```

Bạn có thể trực tiếp sử dụng literal đơn giản và tham chiếu đến hằng số được đặt tên làm constant pattern:

- Number literal (`123`, `45.56`)
- Boolean literal (`true`)
- String literal (`'string'`)
- Hằng số được đặt tên (`someConstant`, `math.pi`, `double.infinity`)
- Constructor hằng (`const Point(0, 0)`)
- Collection literal hằng (`const []`, `const {1, 2}`)

Biểu thức hằng phức tạp hơn phải được đặt trong dấu ngoặc đơn và có tiền tố
`const` (`const (1 + 2)`):

<?code-excerpt "language/lib/patterns/pattern_types.dart (complex-constant)"?>
```dart
// List pattern hoặc map pattern:
case [a, b]: // ...

// List literal hoặc map literal:
case const [a, b]: // ...
```

<a id="variable"></a>

## Variable

`var bar, String str, final int _`

Variable pattern bind biến mới với các giá trị đã được so khớp hoặc destructure.
Chúng thường xuất hiện như một phần của [destructuring pattern][destructure] để
capture một giá trị đã được destructure.

Các biến nằm trong scope ở vùng code chỉ có thể đi đến sau khi
pattern đã khớp.

<?code-excerpt "language/lib/patterns/pattern_types.dart (variable)"?>
```dart
switch ((1, 2)) {
  // 'var a' và 'var b' là các variable pattern lần lượt bind với 1 và 2.
  case (var a, var b): // ...
  // 'a' và 'b' nằm trong scope trong phần thân case.
}
```

Variable pattern _có kiểu_ chỉ khớp nếu giá trị được so khớp có kiểu đã khai báo,
và thất bại nếu không:

<?code-excerpt "language/lib/patterns/pattern_types.dart (variable-typed)"?>
```dart
switch ((1, 2)) {
  // Không khớp.
  case (int a, String b): // ...
}
```

Bạn có thể sử dụng [wildcard pattern](#wildcard) làm variable pattern.

<a id="identifier"></a>

## Identifier

`foo, _`

Identifier pattern hoạt động như một [constant pattern](#constant) hoặc như một
[variable pattern](#variable), tùy thuộc vào ngữ cảnh nơi chúng xuất hiện:

- Ngữ cảnh [khai báo][]: khai báo một biến mới với tên identifier:
  `var (a, b) = (1, 2);`
- Ngữ cảnh [phép gán][]: gán cho biến hiện có với tên identifier:
  `(a, b) = (3, 4);`
- Ngữ cảnh [so khớp][]: được xem là một constant pattern được đặt tên (trừ khi tên là `_`):
  <?code-excerpt "language/lib/patterns/pattern_types.dart (match-context)"?>
  ```dart
  const c = 1;
  switch (2) {
    case c:
      print('match $c');
    default:
      print('no match'); // In ra "no match".
  }
  ```
- Identifier [wildcard](#wildcard) trong bất kỳ ngữ cảnh nào: khớp với mọi giá trị và loại bỏ giá trị đó:
  `case [_, var y, _]: print('The middle element is $y');`

<a id="parenthesized"></a>

## Có dấu ngoặc đơn

`(subpattern)`

Giống biểu thức có dấu ngoặc đơn, dấu ngoặc đơn trong pattern cho phép bạn kiểm soát
[độ ưu tiên của pattern](#pattern-precedence) và chèn một pattern có độ ưu tiên thấp hơn
ở nơi yêu cầu pattern có độ ưu tiên cao hơn.

Ví dụ, hãy tưởng tượng các hằng số boolean `x`, `y` và `z`
lần lượt bằng `true`, `true` và `false`.
Mặc dù ví dụ sau giống việc đánh giá biểu thức boolean,
ví dụ này đang so khớp pattern.

<?code-excerpt "language/lib/patterns/pattern_types.dart (parens)"?>
```dart
// ...
x || y => 'matches true',
x || y && z => 'matches true',
x || (y && z) => 'matches true',
// `x || y && z` tương tự `x || (y && z)`.
(x || y) && z => 'matches nothing',
// ...
```

Dart bắt đầu so khớp pattern từ trái sang phải.

1. Pattern đầu tiên khớp `true` vì `x` khớp `true`.
1. Pattern thứ hai khớp `true` vì `x` khớp `true`.
1. Pattern thứ ba khớp `true` vì `x` khớp `true`.
1. Pattern thứ tư `(x || y) && z` không có kết quả khớp.

   * `x` khớp `true`, vì vậy Dart không thử so khớp `y`.
   * Mặc dù `(x || y)` khớp `true`, `z` không khớp `true`.
   * Do đó, pattern `(x || y) && z` không khớp `true`.
   * Subpattern `(x || y)` không khớp `false`,
     vì vậy Dart không thử so khớp `z`.
   * Do đó, pattern `(x || y) && z` không khớp `false`.
   * Kết luận, `(x || y) && z` không có kết quả khớp.

<a id="list"></a>

## List

`[subpattern1, subpattern2]`

List pattern khớp với những giá trị implement [`List`][], sau đó so khớp đệ quy
các subpattern với phần tử của list để destructure chúng theo vị trí:

<?code-excerpt "language/lib/patterns/switch.dart (list-pattern)"?>
```dart
const a = 'a';
const b = 'b';
switch (obj) {
  // List pattern [a, b] trước tiên khớp với obj nếu obj là list có hai field,
  // sau đó nếu các field khớp với constant subpattern 'a' và 'b'.
  case [a, b]:
    print('$a, $b');
}
```

List pattern yêu cầu số phần tử trong pattern phải khớp với toàn bộ
list. Tuy nhiên, bạn có thể sử dụng [rest element](#rest-element) làm placeholder để
đại diện cho số lượng phần tử bất kỳ trong list.

<a id="rest-element"></a>

### Rest element

List pattern có thể chứa _một_ rest element (`...`), cho phép so khớp list
có độ dài bất kỳ.

<?code-excerpt "language/lib/patterns/pattern_types.dart (rest)"?>
```dart
var [a, b, ..., c, d] = [1, 2, 3, 4, 5, 6, 7];
// In ra "1 2 6 7".
print('$a $b $c $d');
```

Rest element cũng có thể có một subpattern thu thập các phần tử không khớp
với những subpattern khác trong list vào một list mới:

<?code-excerpt "language/lib/patterns/pattern_types.dart (rest-sub)"?>
```dart
var [a, b, ...rest, c, d] = [1, 2, 3, 4, 5, 6, 7];
// In ra "1 2 [3, 4, 5] 6 7".
print('$a $b $rest $c $d');
```

<a id="map"></a>

## Map

`{"key": subpattern1, someConst: subpattern2}`

Map pattern khớp với những giá trị implement [`Map`][], sau đó so khớp đệ quy
các subpattern với key của map để destructure chúng.

Map pattern không yêu cầu pattern phải khớp với toàn bộ map. Map pattern
bỏ qua mọi key mà map chứa nhưng pattern không so khớp.
Việc cố so khớp một key không tồn tại trong map sẽ
ném [`StateError`][]:

<?code-excerpt "language/lib/patterns/pattern_types.dart (map-error)"?>
```dart
final {'foo': int? foo} = {};
```

<a id="record"></a>

## Record

`(subpattern1, subpattern2)`

`(x: subpattern1, y: subpattern2)`

Record pattern khớp với một đối tượng [record][] và destructure các field của nó.
Nếu giá trị không phải record có cùng [shape][] với pattern, việc so khớp
thất bại. Nếu đúng, các field subpattern được so khớp với những
field tương ứng trong record.

Record pattern yêu cầu pattern khớp với toàn bộ record. Để destructure
một record có field _được đặt tên_ bằng pattern, hãy đưa tên field vào pattern:

<?code-excerpt "language/lib/patterns/pattern_types.dart (record)"?>
```dart
var (myString: foo, myNumber: bar) = (myString: 'string', myNumber: 1);
```

Tên getter có thể được bỏ qua và suy luận từ [variable pattern](#variable)
hoặc [identifier pattern](#identifier) trong field subpattern. Những cặp
pattern sau đều tương đương:

<?code-excerpt "language/lib/patterns/pattern_types.dart (record-getter)"?>
```dart
// Record pattern với variable subpattern:
var (untyped: untyped, typed: int typed) = record;
var (:untyped, :int typed) = record;

switch (record) {
  case (untyped: var untyped, typed: int typed): // ...
  case (:var untyped, :int typed): // ...
}

// Record pattern với subpattern null-check và null-assert:
switch (record) {
  case (checked: var checked?, asserted: var asserted!): // ...
  case (:var checked?, :var asserted!): // ...
}

// Record pattern với cast subpattern:
var (untyped: untyped as int, typed: typed as String) = record;
var (:untyped as int, :typed as String) = record;
```

<a id="object"></a>

## Object

`SomeClass(x: subpattern1, y: subpattern2)`

Object pattern kiểm tra giá trị được so khớp với một kiểu được đặt tên đã cho để destructure
dữ liệu bằng getter trên các thuộc tính của đối tượng. Chúng bị [refute][]
nếu giá trị không có cùng kiểu.

<?code-excerpt "language/lib/patterns/pattern_types.dart (object)"?>
```dart
switch (shape) {
  // Khớp nếu shape thuộc kiểu Rect, sau đó khớp với các thuộc tính của Rect.
  case Rect(width: var w, height: var h): // ...
}
```

Tên getter có thể được bỏ qua và suy luận từ [variable pattern](#variable)
hoặc [identifier pattern](#identifier) trong field subpattern:

<?code-excerpt "language/lib/patterns/pattern_types.dart (object-getter)"?>
```dart
// Bind các biến mới x và y với giá trị thuộc tính x và y của Point.
var Point(:x, :y) = Point(1, 2);
```

Object pattern không yêu cầu pattern phải khớp với toàn bộ đối tượng.
Nếu một đối tượng có các field bổ sung mà pattern không destructure, nó vẫn có thể khớp.

<a id="wildcard"></a>

## Wildcard

`_`

Pattern có tên `_` là wildcard, hoặc là [variable pattern](#variable) hoặc
[identifier pattern](#identifier), không bind hoặc gán cho bất kỳ biến nào.

Nó hữu ích như một placeholder ở những nơi bạn cần một subpattern để
destructure các giá trị theo vị trí nằm sau:

<?code-excerpt "language/lib/patterns/pattern_types.dart (wildcard)"?>
```dart
var list = [1, 2, 3];
var [_, two, _] = list;
```

Tên wildcard có type annotation hữu ích khi bạn muốn kiểm tra kiểu của một giá trị
nhưng không bind giá trị với một tên:

<?code-excerpt "language/lib/patterns/pattern_types.dart (wildcard-typed)"?>
```dart
switch (record) {
  case (int _, String _):
    print('First field is int and second is String.');
}
```

[Pattern]: /language/patterns
[phép ép kiểu]: /language/operators#type-test-operators
[destructure]: /language/patterns#destructuring
[ném exception]: /language/error-handling#throw
[khai báo]: /language/patterns#variable-declaration
[phép gán]: /language/patterns#variable-assignment
[so khớp]: /language/patterns#matching
[`List`]: /language/collections#lists
[`Map`]: /language/collections#maps
[`StateError`]: {{site.dart-api}}/dart-core/StateError-class.html
[refute]: /resources/glossary#refutable-pattern
[record]: /language/records
[shape]: /language/records#record-types
[switch]: /language/branches#switch
