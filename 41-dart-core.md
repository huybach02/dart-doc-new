---
title: dart:core
description: Tìm hiểu về các tính năng chính trong thư viện dart:core của Dart.
prevpage:
  url: /libraries
  title: Các thư viện cốt lõi
nextpage:
  url: /libraries/dart-async
  title: dart:async
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt plaster="none"?>

Thư viện dart:core ([tài liệu tham khảo API][dart:core])
cung cấp một tập hợp nhỏ nhưng thiết yếu các chức năng tích hợp sẵn.
Thư viện này được tự động import vào mọi chương trình Dart.


<a id="printing-to-the-console"></a>
## In ra console

Method top-level `print()` nhận một đối số duy nhất (bất kỳ Object nào)
và hiển thị giá trị string của object đó (do `toString()` trả về)
trong console.

<?code-excerpt "misc/test/library_tour/core_test.dart (print)"?>
```dart
print(anObject);
print('I drink $tea.');
```

Để biết thêm thông tin về string cơ bản và `toString()`, hãy xem
[String](/language/built-in-types#strings) trong phần giới thiệu ngôn ngữ.


<a id="numbers"></a>
## Số

Thư viện dart:core định nghĩa các class num, int và double; các class này
có một số tiện ích cơ bản để làm việc với số.

Bạn có thể chuyển đổi một string thành integer hoặc double bằng các method `parse()`
tương ứng của int và double:

<?code-excerpt "misc/test/library_tour/core_test.dart (int-double-parse)"?>
```dart
assert(int.parse('42') == 42);
assert(int.parse('0x42') == 66);
assert(double.parse('0.50') == 0.5);
```

Hoặc sử dụng method parse() của num, method này tạo một integer nếu có thể,
nếu không thì tạo một double:

<?code-excerpt "misc/test/library_tour/core_test.dart (num-parse)"?>
```dart
assert(num.parse('42') is int);
assert(num.parse('0x42') is int);
assert(num.parse('0.50') is double);
```

Để chỉ định cơ số của một integer, hãy thêm tham số `radix`:

<?code-excerpt "misc/test/library_tour/core_test.dart (radix)"?>
```dart
assert(int.parse('42', radix: 16) == 66);
```

Sử dụng method `toString()` để chuyển đổi một
int hoặc double thành string. Để chỉ định số chữ số ở bên phải
dấu thập phân, hãy sử dụng [toStringAsFixed().][toStringAsFixed()] Để chỉ định
số chữ số có nghĩa trong string, hãy sử dụng
[toStringAsPrecision():][toStringAsPrecision()]

<?code-excerpt "misc/test/library_tour/core_test.dart (to-string)"?>
```dart
// Chuyển đổi một int thành string.
assert(42.toString() == '42');

// Chuyển đổi một double thành string.
assert(123.456.toString() == '123.456');

// Chỉ định số chữ số sau dấu thập phân.
assert(123.456.toStringAsFixed(2) == '123.46');

// Chỉ định số chữ số có nghĩa.
assert(123.456.toStringAsPrecision(2) == '1.2e+2');
assert(double.parse('1.2e+2') == 120.0);
```

Để biết thêm thông tin, hãy xem tài liệu API cho
[int,][int] [double,][double] và [num.][num] Đồng thời xem
[phần dart:math](/libraries/dart-math)

<a id="strings-and-regular-expressions"></a>
## String và regular expression

Một string trong Dart là một chuỗi bất biến gồm các code unit UTF-16.
Phần giới thiệu ngôn ngữ có thêm thông tin về
[string](/language/built-in-types#strings).
Bạn có thể sử dụng regular expression (các object RegExp)
để tìm kiếm trong string và
thay thế các phần của string.

Class String định nghĩa các method như `split()`, `contains()`,
`startsWith()`, `endsWith()` và nhiều method khác.

<a id="searching-inside-a-string"></a>
### Tìm kiếm bên trong string

Bạn có thể tìm các vị trí cụ thể trong một string, cũng như kiểm tra
xem string bắt đầu hoặc kết thúc bằng một pattern cụ thể hay không. Ví dụ:

<?code-excerpt "misc/test/library_tour/core_test.dart (contains-etc)"?>
```dart
// Kiểm tra xem một string có chứa string khác hay không.
assert('Never odd or even'.contains('odd'));

// Một string có bắt đầu bằng string khác không?
assert('Never odd or even'.startsWith('Never'));

// Một string có kết thúc bằng string khác không?
assert('Never odd or even'.endsWith('even'));

// Tìm vị trí của một string bên trong một string.
assert('Never odd or even'.indexOf('odd') == 6);
```

<a id="extracting-data-from-a-string"></a>
### Trích xuất dữ liệu từ string

Bạn có thể lấy từng ký tự riêng lẻ từ một string dưới dạng String hoặc int,
tương ứng. Nói chính xác hơn, bạn thực sự nhận được từng code unit UTF-16;
những ký tự có mã cao như ký hiệu khóa Sol
('\\u{1D11E}') gồm hai code unit mỗi ký tự.

Bạn cũng có thể trích xuất một substring hoặc tách một string thành danh sách
các substring:

<?code-excerpt "misc/test/library_tour/core_test.dart (substring-etc)"?>
```dart
// Lấy một substring.
assert('Never odd or even'.substring(6, 9) == 'odd');

// Tách một string bằng một string pattern.
var parts = 'progressive web apps'.split(' ');
assert(parts.length == 3);
assert(parts[0] == 'progressive');

// Lấy một code unit UTF-16 (dưới dạng string) theo index.
assert('Never odd or even'[0] == 'N');

// Sử dụng split() với tham số là string rỗng để lấy
// danh sách tất cả ký tự (dưới dạng String); hữu ích khi
// lặp.
for (final char in 'hello'.split('')) {
  print(char);
}

// Lấy tất cả code unit UTF-16 trong string.
var codeUnitList = 'Never odd or even'.codeUnits.toList();
assert(codeUnitList[0] == 78);
```

:::note
Trong nhiều trường hợp, bạn muốn làm việc với
các cụm tự vị Unicode (Unicode grapheme cluster)
thay vì những code unit thuần túy.
Đây là các ký tự theo cách người dùng nhận biết
(ví dụ, "🇬🇧" là một ký tự theo cảm nhận của
người dùng nhưng gồm nhiều code unit UTF-16).
Để làm việc này, nhóm Dart cung cấp
[`package characters`.]({{site.pub-pkg}}/characters)
:::

<a id="converting-to-uppercase-or-lowercase"></a>
### Chuyển đổi thành chữ hoa hoặc chữ thường

Bạn có thể dễ dàng chuyển đổi string thành biến thể chữ hoa và chữ thường:

<?code-excerpt "misc/test/library_tour/core_test.dart (case-conversions)"?>
```dart
// Chuyển đổi thành chữ hoa.
assert('web apps'.toUpperCase() == 'WEB APPS');

// Chuyển đổi thành chữ thường.
assert('WEB APPS'.toLowerCase() == 'web apps');
```

:::note
Các method này không hoạt động với mọi ngôn ngữ. Ví dụ, chữ *I* không dấu chấm
trong bảng chữ cái Thổ Nhĩ Kỳ bị chuyển đổi không chính xác.
:::


<a id="trimming-and-empty-strings"></a>
### Cắt khoảng trắng và string rỗng

Loại bỏ toàn bộ khoảng trắng ở đầu và cuối bằng `trim()`. Để kiểm tra
xem một string có rỗng (độ dài bằng không) hay không, hãy sử dụng `isEmpty`.

<?code-excerpt "misc/test/library_tour/core_test.dart (trim-etc)"?>
```dart
// Cắt khoảng trắng của một string.
assert('  hello  '.trim() == 'hello');

// Kiểm tra xem một string có rỗng hay không.
assert(''.isEmpty);

// String chỉ chứa khoảng trắng không phải là rỗng.
assert('  '.isNotEmpty);
```

<a id="replacing-part-of-a-string"></a>
### Thay thế một phần của string

String là các object bất biến, nghĩa là bạn có thể tạo nhưng không thể
thay đổi chúng. Nếu xem kỹ [tài liệu tham khảo API của String,][String]
bạn sẽ nhận thấy rằng
không method nào thực sự thay đổi trạng thái của một String. Ví dụ,
method `replaceAll()` trả về một String mới mà không thay đổi
String ban đầu:

<?code-excerpt "misc/test/library_tour/core_test.dart (replace)"?>
```dart
var greetingTemplate = 'Hello, NAME!';
var greeting = greetingTemplate.replaceAll(RegExp('NAME'), 'Bob');

// greetingTemplate không thay đổi.
assert(greeting != greetingTemplate);
```

<a id="building-a-string"></a>
### Xây dựng một string

Để tạo một string bằng chương trình, bạn có thể sử dụng StringBuffer. Một
StringBuffer không tạo object String mới cho đến khi `toString()` được
gọi. Method `writeAll()` có tham số thứ hai tùy chọn cho phép
bạn chỉ định một ký tự phân cách—trong trường hợp này là một khoảng trắng.

<?code-excerpt "misc/test/library_tour/core_test.dart (string-buffer)"?>
```dart
var sb = StringBuffer();
sb
  ..write('Use a StringBuffer for ')
  ..writeAll(['efficient', 'string', 'creation'], ' ')
  ..write('.');

var fullString = sb.toString();

assert(fullString == 'Use a StringBuffer for efficient string creation.');
```

<a id="regular-expressions"></a>
### Regular expression

Class RegExp cung cấp các khả năng tương tự regular expression của JavaScript.
Sử dụng regular expression để tìm kiếm hiệu quả và so khớp pattern
trong string.

<?code-excerpt "misc/test/library_tour/core_test.dart (regexp)"?>
```dart
// Đây là regular expression cho một hoặc nhiều chữ số.
var digitSequence = RegExp(r'\d+');

var lettersOnly = 'llamas live fifteen to twenty years';
var someDigits = 'llamas live 15 to 20 years';

// contains() có thể sử dụng một regular expression.
assert(!lettersOnly.contains(digitSequence));
assert(someDigits.contains(digitSequence));

// Thay thế mọi kết quả khớp bằng một string khác.
var exedOut = someDigits.replaceAll(digitSequence, 'XX');
assert(exedOut == 'llamas live XX to XX years');
```

Bạn cũng có thể làm việc trực tiếp với class RegExp. Class Match
cung cấp quyền truy cập vào một kết quả khớp regular expression.

<?code-excerpt "misc/test/library_tour/core_test.dart (match)"?>
```dart
var digitSequence = RegExp(r'\d+');
var someDigits = 'llamas live 15 to 20 years';

// Kiểm tra xem regular expression có kết quả khớp trong một string hay không.
assert(digitSequence.hasMatch(someDigits));

// Lặp qua tất cả kết quả khớp.
for (final match in digitSequence.allMatches(someDigits)) {
  print(match.group(0)); // 15, sau đó là 20
}
```

<a id="more-information"></a>
### Thông tin thêm

Tham khảo [tài liệu API của String][String] để xem danh sách đầy đủ
các method. Đồng thời xem tài liệu API cho [StringBuffer,][StringBuffer]
[Pattern,][Pattern] [RegExp,][RegExp] và [Match.][Match]

<a id="collections"></a>
## Collection

Dart đi kèm một API collection cốt lõi, bao gồm các class cho
list, set và map.

:::tip
Để thực hành sử dụng những API dùng được cho cả list và set,
hãy làm theo [hướng dẫn về collection Iterable](/libraries/collections/iterables).
:::

<a id="lists"></a>
### List

Như phần giới thiệu ngôn ngữ trình bày, bạn có thể sử dụng literal để tạo và
khởi tạo [list](/language/collections#lists). Ngoài ra, hãy sử dụng một trong các constructor của List.
Class List cũng định nghĩa một số method để thêm item vào và xóa item khỏi list.

<?code-excerpt "misc/test/library_tour/core_test.dart (list)"?>
```dart
// Tạo một list string rỗng.
var grains = <String>[];
assert(grains.isEmpty);

// Tạo list bằng list literal.
var fruits = ['apples', 'oranges'];

// Thêm vào list.
fruits.add('kiwis');

// Thêm nhiều item vào list.
fruits.addAll(['grapes', 'bananas']);

// Lấy độ dài list.
assert(fruits.length == 5);

// Xóa một item.
var appleIndex = fruits.indexOf('apples');
fruits.removeAt(appleIndex);
assert(fruits.length == 4);

// Xóa tất cả element khỏi list.
fruits.clear();
assert(fruits.isEmpty);

// Bạn cũng có thể tạo một List bằng một trong các constructor.
var vegetables = List.filled(99, 'broccoli');
assert(vegetables.every((v) => v == 'broccoli'));
```

Sử dụng `indexOf()` để tìm index của một object trong list:

<?code-excerpt "misc/test/library_tour/core_test.dart (index-of)"?>
```dart
var fruits = ['apples', 'oranges'];

// Truy cập một item trong list theo index.
assert(fruits[0] == 'apples');

// Tìm một item trong list.
assert(fruits.indexOf('apples') == 0);
```

Sắp xếp list bằng method `sort()`. Bạn có thể cung cấp một hàm sắp xếp
để so sánh hai object. Hàm sắp xếp này phải trả về \<
0 khi *nhỏ hơn*, 0 khi *bằng nhau* và \> 0 khi *lớn hơn*. Ví dụ sau
sử dụng `compareTo()`, được định nghĩa bởi
[Comparable][] và được String triển khai.

<?code-excerpt "misc/test/library_tour/core_test.dart (compare-to)"?>
```dart
var fruits = ['bananas', 'apples', 'oranges'];

// Sắp xếp một list.
fruits.sort((a, b) => a.compareTo(b));
assert(fruits[0] == 'apples');
```

List là kiểu được tham số hóa
([generic](/language/generics)),
vì vậy bạn có thể chỉ định kiểu mà list
phải chứa:

<?code-excerpt "misc/test/library_tour/core_test.dart (list-of-string)"?>
```dart
// List này chỉ được chứa string.
var fruits = <String>[];

fruits.add('apples');
var fruit = fruits[0];
assert(fruit is String);
```

<?code-excerpt "misc/lib/library_tour/core/collections.dart (list-of-string)"?>
```dart tag=fails-sa
fruits.add(5); // Lỗi: Không thể gán 'int' cho 'String'
```

:::note
Trong nhiều trường hợp, bạn không
cần chỉ định rõ các kiểu generic,
vì Dart sẽ
[suy luận](/language/type-system#type-inference)
chúng cho bạn.
Một list như `['Dash', 'Dart']` được hiểu
là một `List<String>` (đọc là: list các string).

Tuy nhiên, có những lúc bạn _nên_ chỉ định
kiểu generic. Chẳng hạn, khi Dart không có
thông tin nào để suy luận: `[]` có thể là list của bất kỳ
tổ hợp đối tượng nào.
Đó thường không phải điều bạn muốn, vì vậy bạn viết `<String>[]`,
`<Person>[]` hoặc dạng tương tự.
:::

Tham khảo [tài liệu API của List][List] để xem danh sách đầy đủ các method.

<a id="sets"></a>
### Set

Set trong Dart là một collection không có thứ tự gồm các item duy nhất. Vì set
không có thứ tự, bạn không thể lấy các item của set theo index (vị trí).

<?code-excerpt "misc/test/library_tour/core_test.dart (set)"?>
```dart
// Tạo một set string rỗng.
var ingredients = <String>{};

// Thêm các item mới vào set.
ingredients.addAll(['gold', 'titanium', 'xenon']);
assert(ingredients.length == 3);

// Thêm một item trùng lặp không có tác dụng.
ingredients.add('gold');
assert(ingredients.length == 3);

// Xóa một item khỏi set.
ingredients.remove('gold');
assert(ingredients.length == 2);

// Bạn cũng có thể tạo set bằng
// một trong các constructor.
var atomicNumbers = Set.from([79, 22, 54]);
```

Sử dụng `contains()` và `containsAll()` để kiểm tra xem một hoặc nhiều
object có nằm trong set hay không:

<?code-excerpt "misc/test/library_tour/core_test.dart (contains)"?>
```dart
var ingredients = Set<String>();
ingredients.addAll(['gold', 'titanium', 'xenon']);

// Kiểm tra xem một item có trong set hay không.
assert(ingredients.contains('titanium'));

// Kiểm tra xem tất cả item có trong set hay không.
assert(ingredients.containsAll(['titanium', 'xenon']));
```

Giao (intersection) là một set có các item cùng nằm trong hai set khác.

<?code-excerpt "misc/test/library_tour/core_test.dart (intersection)"?>
```dart
var ingredients = Set<String>();
ingredients.addAll(['gold', 'titanium', 'xenon']);

// Tạo giao của hai set.
var nobleGases = Set.from(['xenon', 'argon']);
var intersection = ingredients.intersection(nobleGases);
assert(intersection.length == 1);
assert(intersection.contains('xenon'));
```

Tham khảo [tài liệu API của Set][Set] để xem danh sách đầy đủ các method.

<a id="maps"></a>
### Map

Map, thường được gọi là *dictionary* hoặc *hash*, là một collection không có
thứ tự gồm các cặp key-value. Map liên kết một key với một value để
dễ truy xuất. Khác với JavaScript, object Dart không phải là map.

Bạn có thể khai báo map bằng cú pháp literal ngắn gọn, hoặc có thể sử dụng
constructor truyền thống:

<?code-excerpt "misc/test/library_tour/core_test.dart (map)"?>
```dart
// Map thường sử dụng string làm key.
var hawaiianBeaches = {
  'Oahu': ['Waikiki', 'Kailua', 'Waimanalo'],
  'Big Island': ['Wailea Bay', 'Pololu Beach'],
  'Kauai': ['Hanalei', 'Poipu'],
};

// Có thể xây dựng map từ constructor.
var searchTerms = Map();

// Map là kiểu được tham số hóa; bạn có thể chỉ định
// kiểu của key và value.
var nobleGases = Map<int, String>();
```

Bạn thêm, lấy và đặt các item của map bằng cú pháp dấu ngoặc vuông. Sử dụng `remove()`
để xóa một key và value của nó khỏi map.

<?code-excerpt "misc/test/library_tour/core_test.dart (remove)"?>
```dart
var nobleGases = {54: 'xenon'};

// Truy xuất một value bằng key.
assert(nobleGases[54] == 'xenon');

// Kiểm tra xem map có chứa một key hay không.
assert(nobleGases.containsKey(54));

// Xóa một key và value của nó.
nobleGases.remove(54);
assert(!nobleGases.containsKey(54));
```

Bạn có thể truy xuất tất cả value hoặc tất cả key từ một map:

<?code-excerpt "misc/test/library_tour/core_test.dart (keys)"?>
```dart
var hawaiianBeaches = {
  'Oahu': ['Waikiki', 'Kailua', 'Waimanalo'],
  'Big Island': ['Wailea Bay', 'Pololu Beach'],
  'Kauai': ['Hanalei', 'Poipu'],
};

// Lấy tất cả key dưới dạng một collection không có thứ tự
// (một Iterable).
var keys = hawaiianBeaches.keys;

assert(keys.length == 3);
assert(keys.contains('Oahu'));

// Lấy tất cả value dưới dạng một collection không có thứ tự
// (một Iterable các List).
var values = hawaiianBeaches.values;
assert(values.length == 3);
assert(values.any((v) => v.contains('Waikiki')));
```

Để kiểm tra xem map có chứa một key hay không, hãy sử dụng `containsKey()`. Vì các
value của map có thể là null, bạn không thể chỉ dựa vào việc lấy value của
key rồi kiểm tra null để xác định key có tồn tại hay không.

<?code-excerpt "misc/test/library_tour/core_test.dart (contains-key)"?>
```dart
var hawaiianBeaches = {
  'Oahu': ['Waikiki', 'Kailua', 'Waimanalo'],
  'Big Island': ['Wailea Bay', 'Pololu Beach'],
  'Kauai': ['Hanalei', 'Poipu'],
};

assert(hawaiianBeaches.containsKey('Oahu'));
assert(!hawaiianBeaches.containsKey('Florida'));
```

Sử dụng method `putIfAbsent()` khi bạn muốn gán một value cho một key
khi và chỉ khi key đó chưa tồn tại trong map. Bạn phải cung cấp
một hàm trả về value.

<?code-excerpt "misc/test/library_tour/core_test.dart (put-if-absent)"?>
```dart
var teamAssignments = <String, String>{};
teamAssignments.putIfAbsent('Catcher', () => pickToughestKid());
assert(teamAssignments['Catcher'] != null);
```

Tham khảo [tài liệu API của Map][Map] để xem danh sách đầy đủ các method.

<a id="common-collection-methods"></a>
### Các method collection phổ biến

List, Set và Map cùng chia sẻ những chức năng phổ biến có trong nhiều collection.
Một số chức năng chung này được định nghĩa bởi class Iterable,
mà List và Set triển khai.

:::note
  Mặc dù Map không triển khai Iterable, bạn có thể lấy các Iterable từ nó bằng
  các property `keys` và `values` của Map.
:::

Sử dụng `isEmpty` hoặc `isNotEmpty` để kiểm tra xem list, set hoặc map có item hay không:

<?code-excerpt "misc/test/library_tour/core_test.dart (is-empty)"?>
```dart
var coffees = <String>[];
var teas = ['green', 'black', 'chamomile', 'earl grey'];
assert(coffees.isEmpty);
assert(teas.isNotEmpty);
```

Để áp dụng một hàm cho từng item trong list, set hoặc map, bạn có thể sử dụng
`forEach()`:

<?code-excerpt "misc/test/library_tour/core_test.dart (list-for-each)"?>
```dart
var teas = ['green', 'black', 'chamomile', 'earl grey'];

teas.forEach((tea) => print('I drink $tea'));
```

Khi gọi `forEach()` trên map, hàm của bạn phải nhận hai
đối số (key và value):

<?code-excerpt "misc/test/library_tour/core_test.dart (map-for-each)"?>
```dart
hawaiianBeaches.forEach((k, v) {
  print('I want to visit $k and swim at $v');
  // I want to visit Oahu and swim at
  // [Waikiki, Kailua, Waimanalo], v.v.
});
```

Iterable cung cấp method `map()`, cho bạn tất cả kết quả trong
một object duy nhất:

<?code-excerpt "misc/test/library_tour/core_test.dart (list-map)"?>
```dart
var teas = ['green', 'black', 'chamomile', 'earl grey'];

var loudTeas = teas.map((tea) => tea.toUpperCase());
loudTeas.forEach(print);
```

:::note
Object do `map()` trả về là một Iterable được *đánh giá lazy*: hàm của bạn
không được gọi cho đến khi bạn yêu cầu một item từ object được trả về.
:::

Để buộc hàm của bạn được gọi ngay lập tức trên từng item, hãy sử dụng
`map().toList()` hoặc `map().toSet()`:

<?code-excerpt "misc/test/library_tour/core_test.dart (to-list)"?>
```dart
var loudTeas = teas.map((tea) => tea.toUpperCase()).toList();
```

Sử dụng method `where()` của Iterable để lấy tất cả item khớp với một
điều kiện. Sử dụng các method `any()` và `every()` của Iterable để kiểm tra xem
một số hoặc tất cả item có khớp với điều kiện hay không.
{% comment %}
ĐANG CHỜ: Thay đổi ví dụ theo đề xuất của floitsch để dùng (có thể là)
các thành phố thay cho isDecaffeinated.
{% endcomment %}


<?code-excerpt "misc/test/library_tour/core_test.dart (where-etc)"?>
```dart
var teas = ['green', 'black', 'chamomile', 'earl grey'];

// Chamomile không chứa caffeine.
bool isDecaffeinated(String teaName) => teaName == 'chamomile';

// Sử dụng where() để chỉ tìm các item trả về true
// từ hàm được cung cấp.
var decaffeinatedTeas = teas.where((tea) => isDecaffeinated(tea));
// hoặc teas.where(isDecaffeinated)

// Sử dụng any() để kiểm tra xem ít nhất một item trong
// collection có thỏa mãn điều kiện hay không.
assert(teas.any(isDecaffeinated));

// Sử dụng every() để kiểm tra xem tất cả item trong
// collection có thỏa mãn điều kiện hay không.
assert(!teas.every(isDecaffeinated));
```

Để xem danh sách đầy đủ các method, hãy tham khảo [tài liệu API của Iterable,][Iterable]
cũng như tài liệu cho [List,][List] [Set,][Set] và [Map.][Map]


<a id="uris"></a>
## URI

[Class Uri][Uri] cung cấp
các hàm mã hóa và giải mã string để sử dụng trong URI (mà bạn có thể
biết đến với tên *URL*). Các hàm này xử lý những ký tự đặc biệt đối với
URI, chẳng hạn như `&` và `=`. Class Uri cũng parse và cung cấp
các component của URI—host, port, scheme, v.v.
{% comment %}
{ĐANG CHỜ: trình bày
các constructor: Uri.http, Uri.https, Uri.file, theo đề xuất của floitsch}
{% endcomment %}

<a id="encoding-and-decoding-fully-qualified-uris"></a>
### Mã hóa và giải mã URI đầy đủ

Để mã hóa và giải mã các ký tự *ngoại trừ* những ký tự có ý nghĩa đặc biệt trong
URI (chẳng hạn như `/`, `:`, `&`, `#`), hãy sử dụng các method `encodeFull()` và
`decodeFull()`. Các method này phù hợp để mã hóa hoặc giải mã
một URI đầy đủ, đồng thời giữ nguyên các ký tự URI đặc biệt.

<?code-excerpt "misc/test/library_tour/core_test.dart (encode-full)"?>
```dart
var uri = 'https://example.org/api?foo=some message';

var encoded = Uri.encodeFull(uri);
assert(encoded == 'https://example.org/api?foo=some%20message');

var decoded = Uri.decodeFull(encoded);
assert(uri == decoded);
```

Lưu ý rằng chỉ khoảng trắng giữa `some` và `message` được mã hóa.

<a id="encoding-and-decoding-uri-components"></a>
### Mã hóa và giải mã các component URI

Để mã hóa và giải mã tất cả ký tự trong một string có ý nghĩa đặc biệt
trong URI, bao gồm nhưng không giới hạn ở `/`, `&` và `:`, hãy sử dụng
các method `encodeComponent()` và `decodeComponent()`.

<?code-excerpt "misc/test/library_tour/core_test.dart (encode-component)"?>
```dart
var uri = 'https://example.org/api?foo=some message';

var encoded = Uri.encodeComponent(uri);
assert(
  encoded == 'https%3A%2F%2Fexample.org%2Fapi%3Ffoo%3Dsome%20message',
);

var decoded = Uri.decodeComponent(encoded);
assert(uri == decoded);
```

Lưu ý rằng mọi ký tự đặc biệt đều được mã hóa. Ví dụ, `/` được
mã hóa thành `%2F`.

<a id="parsing-uris"></a>
### Parse URI

Nếu có một object Uri hoặc một string URI, bạn có thể lấy các phần của nó bằng
những field của Uri như `path`. Để tạo một Uri từ string, hãy sử dụng
method static `parse()`:

<?code-excerpt "misc/test/library_tour/core_test.dart (uri-parse)"?>
```dart
var uri = Uri.parse('https://example.org:8080/foo/bar#frag');

assert(uri.scheme == 'https');
assert(uri.host == 'example.org');
assert(uri.path == '/foo/bar');
assert(uri.fragment == 'frag');
assert(uri.origin == 'https://example.org:8080');
```

Xem [tài liệu API của Uri][Uri] để biết thêm các component URI mà bạn có thể lấy.

<a id="building-uris"></a>
### Xây dựng URI

Bạn có thể xây dựng URI từ các phần riêng lẻ bằng constructor `Uri()`:

<?code-excerpt "misc/test/library_tour/core_test.dart (uri)"?>
```dart
var uri = Uri(
  scheme: 'https',
  host: 'example.org',
  path: '/foo/bar',
  fragment: 'frag',
  queryParameters: {'lang': 'dart'},
);
assert(uri.toString() == 'https://example.org/foo/bar?lang=dart#frag');
```

Nếu bạn không cần chỉ định fragment,
để tạo URI với scheme http hoặc https,
bạn có thể sử dụng các factory constructor [`Uri.http`][] hoặc [`Uri.https`][]:

<?code-excerpt "misc/test/library_tour/core_test.dart (uri-http)"?>
```dart
var httpUri = Uri.http('example.org', '/foo/bar', {'lang': 'dart'});
var httpsUri = Uri.https('example.org', '/foo/bar', {'lang': 'dart'});

assert(httpUri.toString() == 'http://example.org/foo/bar?lang=dart');
assert(httpsUri.toString() == 'https://example.org/foo/bar?lang=dart');
```

[`Uri.http`]: {{site.dart-api}}/dart-core/Uri/Uri.http.html
[`Uri.https`]: {{site.dart-api}}/dart-core/Uri/Uri.https.html

<a id="dates-and-times"></a>
## Ngày và giờ

Object DateTime là một thời điểm. Múi giờ là UTC hoặc
múi giờ địa phương.

Bạn có thể tạo object DateTime bằng một số constructor và method:

<?code-excerpt "misc/test/library_tour/core_test.dart (date-time)"?>
```dart
// Lấy ngày và giờ hiện tại.
var now = DateTime.now();

// Tạo một DateTime mới với múi giờ địa phương.
var y2k = DateTime(2000); // Ngày 1 tháng 1 năm 2000

// Chỉ định tháng và ngày.
y2k = DateTime(2000, 1, 2); // Ngày 2 tháng 1 năm 2000

// Chỉ định ngày dưới dạng giờ UTC.
y2k = DateTime.utc(2000); // 1/1/2000, UTC

// Chỉ định ngày và giờ bằng mili giây kể từ Unix epoch.
y2k = DateTime.fromMillisecondsSinceEpoch(946684800000, isUtc: true);

// Parse một ngày ISO 8601 trong múi giờ UTC.
y2k = DateTime.parse('2000-01-01T00:00:00Z');

// Tạo một DateTime mới từ DateTime có sẵn, chỉ điều chỉnh một số property:
var sameTimeLastYear = now.copyWith(year: now.year - 1);
```

:::warning
Các thao tác `DateTime` có thể cho kết quả ngoài mong đợi liên quan đến
giờ tiết kiệm ánh sáng ban ngày và những điều chỉnh thời gian không tiêu chuẩn khác.  
:::

Property `millisecondsSinceEpoch` của một ngày trả về số
mili giây kể từ "Unix epoch"—ngày 1 tháng 1 năm 1970, UTC:

<?code-excerpt "misc/test/library_tour/core_test.dart (milliseconds-since-epoch)"?>
```dart
// 1/1/2000, UTC
var y2k = DateTime.utc(2000);
assert(y2k.millisecondsSinceEpoch == 946684800000);

// 1/1/1970, UTC
var unixEpoch = DateTime.utc(1970);
assert(unixEpoch.millisecondsSinceEpoch == 0);
```

Sử dụng class Duration để tính khoảng chênh lệch giữa hai ngày và
dịch chuyển một ngày về phía trước hoặc phía sau:

<?code-excerpt "misc/test/library_tour/core_test.dart (duration)"?>
```dart
var y2k = DateTime.utc(2000);

// Cộng một năm.
var y2001 = y2k.add(const Duration(days: 366));
assert(y2001.year == 2001);

// Trừ 30 ngày.
var december2000 = y2001.subtract(const Duration(days: 30));
assert(december2000.year == 2000);
assert(december2000.month == 12);

// Tính khoảng chênh lệch giữa hai ngày.
// Trả về một object Duration.
var duration = y2001.difference(y2k);
assert(duration.inDays == 366); // y2k là năm nhuận.
```

:::warning
Sử dụng `Duration` để dịch chuyển một `DateTime` theo ngày có thể gây vấn đề do
sự thay đổi đồng hồ (chẳng hạn khi chuyển sang giờ tiết kiệm ánh sáng ban ngày).
Hãy sử dụng ngày UTC nếu bạn phải dịch chuyển theo ngày.
:::

Để xem danh sách đầy đủ các method,
hãy tham khảo tài liệu API cho [DateTime][] và [Duration.][Duration]


<a id="utility-classes"></a>
## Các class tiện ích

Thư viện core chứa nhiều class tiện ích, hữu ích cho việc sắp xếp,
ánh xạ value và lặp.

<a id="comparing-objects"></a>
### So sánh object

Triển khai interface [Comparable][]
để cho biết một object có thể được so sánh với object khác,
thường phục vụ việc sắp xếp. Method `compareTo()` trả về \< 0 khi
*nhỏ hơn*, 0 khi *bằng nhau* và \> 0 khi *lớn hơn*.

<?code-excerpt "misc/lib/library_tour/core/comparable.dart"?>
```dart
class Line implements Comparable<Line> {
  final int length;
  const Line(this.length);

  @override
  int compareTo(Line other) => length - other.length;
}

void main() {
  var short = const Line(1);
  var long = const Line(100);
  assert(short.compareTo(long) < 0);
}
```

<a id="implementing-map-keys"></a>
### Triển khai key của map

Mỗi object trong Dart tự động cung cấp một mã hash dạng integer, do đó
có thể được sử dụng làm key trong map. Tuy nhiên, bạn có thể override getter
`hashCode` để tạo một mã hash tùy chỉnh. Nếu làm vậy, bạn cũng có thể
muốn override operator `==`. Các object bằng nhau (qua
`==`) phải có mã hash giống hệt nhau. Mã hash không nhất thiết phải
duy nhất, nhưng nên được phân bố tốt.

:::tip
Để triển khai getter `hashCode` một cách nhất quán và dễ dàng,
hãy cân nhắc sử dụng các method hash static do class `Object` cung cấp.

Để tạo một mã hash duy nhất cho nhiều property của một object,
bạn có thể sử dụng [`Object.hash()`][].
Để tạo mã hash cho một collection,
bạn có thể sử dụng [`Object.hashAll()`][] (nếu thứ tự element quan trọng)
hoặc [`Object.hashAllUnordered()`][].
:::

[`Object.hash()`]: {{site.dart-api}}/dart-core/Object/hash.html
[`Object.hashAll()`]: {{site.dart-api}}/dart-core/Object/hashAll.html
[`Object.hashAllUnordered()`]: {{site.dart-api}}/dart-core/Object/hashAllUnordered.html

{% comment %}
Lưu ý: Có sự bất đồng về việc có nên đưa identical() vào implementation của ==
hay không. Nó có thể cải thiện tốc độ, ít nhất khi bạn cần
so sánh nhiều field. Họ không tự động thực hiện identical() vì theo
quy ước, NaN != NaN.
{% endcomment %}

<?code-excerpt "misc/lib/library_tour/core/hash_code.dart"?>
```dart
class Person {
  final String firstName, lastName;

  Person(this.firstName, this.lastName);

  // Override hashCode bằng các method hash static
  // do class `Object` cung cấp.
  @override
  int get hashCode => Object.hash(firstName, lastName);

  // Thông thường, bạn nên triển khai operator `==` nếu
  // override `hashCode`.
  @override
  bool operator ==(Object other) {
    return other is Person &&
        other.firstName == firstName &&
        other.lastName == lastName;
  }
}

void main() {
  var p1 = Person('Bob', 'Smith');
  var p2 = Person('Bob', 'Smith');
  var p3 = 'not a person';
  assert(p1.hashCode == p2.hashCode);
  assert(p1 == p2);
  assert(p1 != p3);
}
```

<a id="iteration"></a>
### Phép lặp

Các class [Iterable][] và [Iterator][]
hỗ trợ truy cập tuần tự vào một collection các value.
Để thực hành sử dụng những collection này,
hãy làm theo [hướng dẫn về collection Iterable](/libraries/collections/iterables).

Nếu bạn tạo một class có thể cung cấp Iterator để sử dụng trong vòng lặp for-in,
hãy extend (nếu có thể) hoặc implement Iterable.
Implement Iterator để định nghĩa khả năng lặp thực tế.

<?code-excerpt "misc/lib/library_tour/core/iterator.dart (structure)"?>
```dart
class Process {
  // Biểu diễn một process...
}

class ProcessIterator implements Iterator<Process> {
  @override
  Process get current => ...
  @override
  bool moveNext() => ...
}

// Một class giả định cho phép bạn lặp qua tất cả
// process. Extend một subclass của [Iterable].
class Processes extends IterableBase<Process> {
  @override
  final Iterator<Process> iterator = ProcessIterator();
}

void main() {
  // Có thể sử dụng object Iterable với for-in.
  for (final process in Processes()) {
    // Thực hiện điều gì đó với process.
  }
}
```

<a id="exceptions"></a>
## Exception

Thư viện core của Dart định nghĩa nhiều exception và error phổ biến.
Exception được xem là những điều kiện bạn có thể dự liệu và
bắt. Error là những điều kiện bạn không mong đợi hoặc dự liệu.

Hai error phổ biến nhất là:

[NoSuchMethodError][]
: Được throw khi object nhận (có thể là `null`) không
  implement một method.

[ArgumentError][]
: Có thể được throw bởi một method gặp phải đối số ngoài mong đợi.

Throw một exception dành riêng cho ứng dụng là cách phổ biến để cho biết
đã xảy ra lỗi. Bạn có thể định nghĩa exception tùy chỉnh bằng cách
implement interface Exception:

<?code-excerpt "misc/lib/library_tour/core/exception.dart"?>
```dart
class FooException implements Exception {
  final String? msg;

  const FooException([this.msg]);

  @override
  String toString() => msg ?? 'FooException';
}
```

Để biết thêm thông tin, hãy xem
[Exception](/language/error-handling#exceptions)
(trong phần giới thiệu ngôn ngữ) và [tài liệu tham khảo API của Exception.][Exception]

<a id="weak-references-and-finalizers"></a>
## Weak reference và finalizer

Dart là một ngôn ngữ [được thu gom rác][],
nghĩa là bất kỳ object Dart nào
không được tham chiếu
đều có thể bị garbage collector loại bỏ.
Hành vi mặc định này có thể không phù hợp trong
một số trường hợp liên quan đến tài nguyên native hoặc
khi không thể sửa đổi object đích.

Một [WeakReference][]
lưu trữ tham chiếu đến object đích
mà không ảnh hưởng đến cách object đó
được garbage collector thu gom.
Một lựa chọn khác là sử dụng [Expando][]
để thêm property vào một object.

Có thể sử dụng [Finalizer][] để thực thi một hàm callback
sau khi một object không còn được tham chiếu.
Tuy nhiên, callback này không được đảm bảo sẽ thực thi.

[NativeFinalizer][]
cung cấp những bảo đảm mạnh hơn
khi tương tác với code native bằng [dart:ffi][];
callback của nó được gọi ít nhất một lần
sau khi object không còn được tham chiếu.
Ngoài ra, nó có thể được sử dụng để đóng các tài nguyên native
như kết nối database hoặc file đang mở.

Để đảm bảo một object không bị
garbage collector thu gom và finalize quá sớm,
các class có thể implement interface [Finalizable][].
Khi một biến local là Finalizable,
nó sẽ không bị garbage collector thu gom
cho đến khi block code nơi nó được khai báo đã kết thúc.

:::version-note
Hỗ trợ weak reference và finalizer đã được thêm trong Dart 2.17.
:::


[ArgumentError]: {{site.dart-api}}/dart-core/ArgumentError-class.html
[Comparable]: {{site.dart-api}}/dart-core/Comparable-class.html
[DateTime]: {{site.dart-api}}/dart-core/DateTime-class.html
[Duration]: {{site.dart-api}}/dart-core/Duration-class.html
[Exception]: {{site.dart-api}}/dart-core/Exception-class.html
[Expando]: {{site.dart-api}}/dart-core/Expando-class.html
[Finalizable]: {{site.dart-api}}/dart-ffi/Finalizable-class.html
[Finalizer]: {{site.dart-api}}/dart-core/Finalizer-class.html
[Iterable]: {{site.dart-api}}/dart-core/Iterable-class.html
[Iterator]: {{site.dart-api}}/dart-core/Iterator-class.html
[List]: {{site.dart-api}}/dart-core/List-class.html
[Map]: {{site.dart-api}}/dart-core/Map-class.html
[Match]: {{site.dart-api}}/dart-core/Match-class.html
[NativeFinalizer]: {{site.dart-api}}/dart-ffi/NativeFinalizer-class.html
[NoSuchMethodError]: {{site.dart-api}}/dart-core/NoSuchMethodError-class.html
[Pattern]: {{site.dart-api}}/dart-core/Pattern-class.html
[RegExp]: {{site.dart-api}}/dart-core/RegExp-class.html
[Set]: {{site.dart-api}}/dart-core/Set-class.html
[StringBuffer]: {{site.dart-api}}/dart-core/StringBuffer-class.html
[String]: {{site.dart-api}}/dart-core/String-class.html
[Uri]: {{site.dart-api}}/dart-core/Uri-class.html
[WeakReference]: {{site.dart-api}}/dart-core/WeakReference-class.html
[dart:core]: {{site.dart-api}}/dart-core/dart-core-library.html
[dart:ffi]: /interop/c-interop
[double]: {{site.dart-api}}/dart-core/double-class.html
[garbage-collected]: {{site.flutter-blog}}/flutter-dont-fear-the-garbage-collector
[int]: {{site.dart-api}}/dart-core/int-class.html
[num]: {{site.dart-api}}/dart-core/num-class.html
[toStringAsFixed()]: {{site.dart-api}}/dart-core/num/toStringAsFixed.html
[toStringAsPrecision()]: {{site.dart-api}}/dart-core/num/toStringAsPrecision.html
