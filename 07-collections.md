---
title: Collection
description: Tóm tắt về các loại collection khác nhau trong Dart.
prevpage:
  url: /language/records
  title: Record
nextpage:
  url: /language/generics
  title: Generic
---

Dart hỗ trợ sẵn các [collection][] list, set và map.
Để tìm hiểu thêm về cách cấu hình những kiểu mà collection chứa,
hãy xem [Generic][].

[Generic]: /language/generics
[collection]: /libraries/dart-core#collections

<a id="lists"></a>

## List

Có lẽ collection phổ biến nhất trong gần như mọi ngôn ngữ lập trình
là *array*, tức một nhóm đối tượng có thứ tự. Trong Dart, array là
đối tượng [`List`][], vì vậy hầu hết mọi người chỉ gọi chúng là *list*.

List literal trong Dart được biểu thị bằng danh sách các
phần tử phân tách bởi dấu phẩy và đặt trong dấu ngoặc vuông (`[]`). Mỗi phần tử
thường là một biểu thức. Sau đây là một list Dart đơn giản:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (list-literal)"?>
```dart
var list = [1, 2, 3];
```

:::note
Dart suy luận rằng `list` có kiểu `List<int>`. Nếu bạn cố thêm đối tượng không phải số nguyên
vào list này, analyzer hoặc runtime sẽ phát sinh lỗi. Để biết thêm
thông tin, hãy đọc về [suy luận kiểu][].
:::

<a id="trailing-comma"></a>
Bạn có thể thêm dấu phẩy sau item cuối cùng trong một collection literal Dart.
_Dấu phẩy cuối_ này không ảnh hưởng đến collection,
nhưng có thể giúp ngăn lỗi sao chép-dán.

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (trailing-commas)"?>
```dart
var list = ['Car', 'Boat', 'Plane',];
```

List sử dụng index bắt đầu từ 0, trong đó 0 là index của phần tử đầu tiên
và `list.length - 1` là index của phần tử cuối cùng.
Bạn có thể lấy độ dài của list bằng thuộc tính `.length`
và truy cập các phần tử bằng toán tử subscript (`[]`):

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (list-indexing)"?>
```dart
var list = [1, 2, 3];
assert(list.length == 3);
assert(list[1] == 2);

list[1] = 1;
assert(list[1] == 1);
```

Để tạo một list là hằng số tại thời điểm biên dịch,
hãy thêm `const` trước list literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (const-list)"?>
```dart
var constantList = const [1, 2, 3];
// constantList[1] = 1; // Dòng này sẽ gây ra lỗi.
```

Để biết thêm thông tin về list, hãy tham khảo phần List trong
[tài liệu `dart:core`](/libraries/dart-core#lists).

[`List`]: {{site.dart-api}}/dart-core/List-class.html
[suy luận kiểu]: /language/type-system#type-inference

<a id="sets"></a>

## Set

Set trong Dart là một collection không có thứ tự gồm các phần tử duy nhất.
Khả năng hỗ trợ set của Dart được cung cấp bởi set literal và kiểu
[`Set`][].

Sau đây là một set Dart đơn giản được tạo bằng set literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (set-literal)"?>
```dart
var halogens = {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};
```

:::note
Dart suy luận rằng `halogens` có kiểu `Set<String>`. Nếu bạn cố thêm
phần tử sai kiểu vào set, analyzer hoặc runtime sẽ phát sinh lỗi. Để biết
thêm thông tin, hãy đọc về
[suy luận kiểu.](/language/type-system#type-inference)
:::

Để tạo một set rỗng, hãy sử dụng `{}` với một đối số kiểu đặt phía trước,
hoặc gán `{}` cho một biến có kiểu `Set`:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (set-vs-map)"?>
```dart
var names = <String>{};
// Set<String> names = {}; // Cách này cũng hoạt động.
// var names = {}; // Tạo map, không phải set.
```

:::note Set hay map?
Cú pháp của map literal tương tự cú pháp của set
literal. Vì map literal xuất hiện trước nên `{}` mặc định mang kiểu `Map`. Nếu
bạn quên type annotation trên `{}` hoặc trên biến nhận phép gán, thì
Dart sẽ tạo một đối tượng thuộc kiểu `Map<dynamic, dynamic>`.
:::

Thêm item vào một set hiện có bằng phương thức `add()` hoặc `addAll()`:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (set-add-items)"?>
```dart
var elements = <String>{};
elements.add('fluorine');
elements.addAll(halogens);
```

Sử dụng `.length` để lấy số lượng item trong set:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (set-length)"?>
```dart
var elements = <String>{};
elements.add('fluorine');
elements.addAll(halogens);
assert(elements.length == 5);
```

Để tạo một set là hằng số tại thời điểm biên dịch,
hãy thêm `const` trước set literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (const-set)"?>
```dart
final constantSet = const {
  'fluorine',
  'chlorine',
  'bromine',
  'iodine',
  'astatine',
};
// constantSet.add('helium'); // Dòng này sẽ gây ra lỗi.
```

Để biết thêm thông tin về set, hãy tham khảo phần Set trong
[tài liệu `dart:core`](/libraries/dart-core#sets).

[`Set`]: {{site.dart-api}}/dart-core/Set-class.html

<a id="maps"></a>

## Map

Trong map, mỗi phần tử là một cặp key-value. Mỗi key trong
một cặp được liên kết với một value, và cả key lẫn value
đều có thể là bất kỳ kiểu đối tượng nào. Mỗi key chỉ có thể xuất hiện một lần,
mặc dù cùng một value có thể được liên kết với nhiều
key khác nhau. Khả năng hỗ trợ map của Dart được cung cấp bởi
map literal và kiểu [`Map`][].

Sau đây là một vài map Dart đơn giản được tạo bằng map literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (map-literal)"?>
```dart
var gifts = {
  // Key:    Value
  'first': 'partridge',
  'second': 'turtledoves',
  'fifth': 'golden rings',
};

var nobleGases = {2: 'helium', 10: 'neon', 18: 'argon'};
```

:::note
Dart suy luận rằng `gifts` có kiểu `Map<String, String>` và `nobleGases`
có kiểu `Map<int, String>`. Nếu bạn cố thêm value sai kiểu vào
một trong hai map, analyzer hoặc runtime sẽ phát sinh lỗi. Để biết thêm thông tin,
hãy đọc về [suy luận kiểu][].
:::

Bạn có thể tạo các đối tượng tương tự bằng constructor Map:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (map-constructor)"?>
```dart
var gifts = Map<String, String>();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';
gifts['fifth'] = 'golden rings';

var nobleGases = Map<int, String>();
nobleGases[2] = 'helium';
nobleGases[10] = 'neon';
nobleGases[18] = 'argon';
```

:::note
Nếu đến từ một ngôn ngữ như C# hoặc Java, bạn có thể mong đợi thấy `new Map()`
thay vì chỉ `Map()`. Trong Dart, keyword `new` là tùy chọn.
Để biết chi tiết, hãy xem [Sử dụng constructor][].
:::

Thêm một cặp key-value mới vào map hiện có
bằng toán tử gán subscript (`[]=`):

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (map-add-item)"?>
```dart
var gifts = {'first': 'partridge'};
gifts['fourth'] = 'calling birds'; // Thêm một cặp key-value
```

Lấy một value từ map bằng toán tử subscript (`[]`):

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (map-retrieve-item)"?>
```dart
var gifts = {'first': 'partridge'};
assert(gifts['first'] == 'partridge');
```

Nếu tìm một key không có trong map, bạn sẽ nhận lại `null`:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (map-missing-key)"?>
```dart
var gifts = {'first': 'partridge'};
assert(gifts['fifth'] == null);
```

Sử dụng `.length` để lấy số cặp key-value trong map:

<?code-excerpt "misc/test/language_tour/built_in_types_test.dart (map-length)"?>
```dart
var gifts = {'first': 'partridge'};
gifts['fourth'] = 'calling birds';
assert(gifts.length == 2);
```

Để tạo một map là hằng số tại thời điểm biên dịch,
hãy thêm `const` trước map literal:

<?code-excerpt "misc/lib/language_tour/built_in_types.dart (const-map)"?>
```dart
final constantMap = const {2: 'helium', 10: 'neon', 18: 'argon'};

// constantMap[2] = 'Helium'; // Dòng này sẽ gây ra lỗi.
```

Để biết thêm thông tin về map, hãy tham khảo phần Map trong
[tài liệu `dart:core`](/libraries/dart-core#maps).

[Sử dụng constructor]: /language/classes#using-constructors
[`Map`]: {{site.dart-api}}/dart-core/Map-class.html
[suy luận kiểu]: /language/type-system#type-inference

<a id="collection-elements"></a>

## Phần tử collection

Collection literal chứa một chuỗi phần tử. Trong
runtime, từng phần tử được đánh giá, tạo ra không hoặc nhiều
giá trị, sau đó các giá trị này được chèn vào collection kết quả.
Những phần tử này thuộc hai nhóm chính: phần tử leaf
và phần tử control flow.

*   Phần tử leaf: Chèn một item riêng lẻ vào
    collection literal.

    *   Phần tử biểu thức: Đánh giá một
        biểu thức duy nhất và chèn giá trị kết quả
        vào collection.

    *   Phần tử entry của map: Đánh giá một cặp biểu thức key và value
        rồi chèn entry kết quả vào
        collection.

*   Phần tử control flow: Thêm có điều kiện hoặc lặp
    không hoặc nhiều giá trị vào collection bao quanh.

    *   Phần tử null-aware: Đánh giá một biểu thức và nếu
        kết quả không phải `null`, chèn giá trị vào
        collection bao quanh.

    *   Phần tử spread: Lặp qua một chuỗi đã cho
        (biểu thức collection) và chèn tất cả
        giá trị kết quả vào collection bao quanh.

    *   Phần tử spread null-aware: Tương tự
        phần tử spread, nhưng cho phép collection là
        `null` và không chèn gì nếu collection là null.

    *   Phần tử if: Đánh giá có điều kiện một phần tử bên trong
        dựa trên biểu thức điều kiện đã cho, đồng thời
        tùy chọn đánh giá một phần tử `else` khác nếu
        điều kiện là false.

    *   Phần tử for: Lặp và liên tục đánh giá
        một phần tử bên trong đã cho, chèn không hoặc nhiều
        giá trị kết quả.

Để tìm hiểu thêm về phần tử collection, hãy xem các
phần sau.

<a id="control-flow-operators" aria-hidden="true"></a>
<a id="spread-operators" aria-hidden="true"></a>
<a id="collection-operators" aria-hidden="true"></a>

<a id="expression-element"></a>

### Phần tử biểu thức

Phần tử biểu thức đánh giá một biểu thức duy nhất
và chèn giá trị kết quả vào collection. Biểu thức
này có thể bao gồm nhiều cấu trúc như
literal, biến, toán tử, lời gọi hàm và
lời gọi constructor.

Phần tử biểu thức có cú pháp sau trong collection:

```dart
<expression>
```

<a id="map-entry-element"></a>

### Phần tử entry của map

Phần tử entry của map đánh giá một cặp biểu thức key và value
rồi chèn entry kết quả vào
collection. Cả key lẫn value trong cặp này đều có thể
là biểu thức.

Phần tử entry của map có cú pháp sau trong collection:

```dart
<key_expression>: <value_expression>
```

<a id="null-aware-element"></a>

### Phần tử null-aware

Phần tử null-aware đánh giá một biểu thức và nếu
kết quả không phải `null`, chèn giá trị vào collection
bao quanh.

:::version-note
Phần tử collection null-aware yêu cầu
[phiên bản ngôn ngữ][] tối thiểu là 3.8.
:::

Phần tử null-aware có cú pháp sau trong một
phần tử biểu thức:

```dart
?<expression>
```

Phần tử null-aware có cú pháp sau trong một
phần tử entry của map:

```dart
// key là một phần tử null-aware
?<key_expression>: <value_expression>
```

```dart
// value là một phần tử null-aware
<key_expression>: ?<value_expression>
```

```dart
// key và value là các phần tử null-aware
?<key_expression>: ?<value_expression>
```

Trong ví dụ sau, kết quả của
phần tử null-aware `?absentValue` không được thêm vào list có tên
`items` vì `absentValue` là `null`:

<?code-excerpt "misc/test/language_tour/collections/null_aware_element_a.dart (code-sample)"?>
```dart
int? absentValue = null;
int? presentValue = 3;
var items = [
  1,
  ?absentValue,
  ?presentValue,
  absentValue,
  5,
]; // [1, 3, null, 5]
```

Ví dụ sau minh họa nhiều cách khác nhau để
bạn sử dụng phần tử null-aware bên trong
phần tử entry của map:

<?code-excerpt "misc/test/language_tour/collections/null_aware_element_b.dart (code-sample)"?>
```dart
String? presentKey = 'Apple';
String? absentKey = null;

int? presentValue = 3;
int? absentValue = null;

var itemsA = {presentKey: absentValue}; // {Apple: null}
var itemsB = {presentKey: ?absentValue}; // {}

var itemsC = {absentKey: presentValue}; // {null: 3}
var itemsD = {?absentKey: presentValue}; // {}

var itemsE = {absentKey: absentValue}; // {null: null}
var itemsF = {?absentKey: ?absentValue}; // {}
```

[phiên bản ngôn ngữ]: /language/versioning

<a id="spread-element"></a>

### Phần tử spread

Phần tử spread lặp qua một chuỗi đã cho và
chèn tất cả giá trị kết quả vào collection
bao quanh.

Phần tử spread có cú pháp sau trong collection.
Biểu thức chuỗi có thể biểu diễn bất kỳ biểu thức nào
được đánh giá thành một đối tượng implement `Iterable`:

```dart
...<sequence_expression>
```

Trong ví dụ sau, các phần tử trong list có tên `a`
được thêm vào list có tên `items`.

<?code-excerpt "misc/test/language_tour/collections/spread_operator_in_collection_a.dart (code_sample)"?>
```dart
var a = [1, 2, null, 4];
var items = [0, ...a, 5]; // [0, 1, 2, null, 4, 5]
```

Nếu bạn spread một biểu thức có thể được đánh giá thành
`null` và muốn bỏ qua `null` (không chèn
phần tử nào), hãy sử dụng [phần tử spread null-aware][].

Để tìm hiểu thêm về toán tử spread, hãy xem
[Toán tử spread][].

[Toán tử spread]: /language/operators/#spread-operators
[phần tử spread null-aware]: #null-spread-element

<a id="null-spread-element"></a>

### Phần tử spread null-aware

Phần tử spread null-aware tương tự
phần tử spread, nhưng cho phép collection là `null` và
không chèn gì nếu collection là null.

Phần tử spread null-aware có cú pháp sau trong
collection:

```dart
...?<sequence_expression>
```

Trong ví dụ sau, list có tên `a` bị bỏ qua
vì nó là null, nhưng các phần tử trong list có tên `b`
được thêm vào list có tên `items`. Lưu ý rằng nếu bản thân
collection không phải `null` nhưng chứa các phần tử
là `null`, những phần tử `null` đó vẫn được thêm vào kết quả.

```dart
List<int>? a = null;
var b = [1, null, 3];
var items = [0, ...?a, ...?b, 4]; // [0, 1, null, 3, 4]
```

Do null safety, bạn không thể thực hiện
thao tác spread (`...`) trên một giá trị có thể là null.
Ví dụ sau tạo ra lỗi tại thời điểm biên dịch vì tham số
`extraOptions` là nullable và toán tử spread được sử dụng
trên `extraOptions` không phải null-aware.

<?code-excerpt "misc/test/language_tour/collections/null_spread_operator_in_collection_b.dart (code_sample)"?>
```dart tag=fails-sa
List<String> buildCommandLine(
  String executable,
  List<String> options, [
  List<String>? extraOptions,
]) {
  return [
    executable,
    ...options,
    ...extraOptions, // <-- Lỗi
  ];
}

// Cách sử dụng:
//   buildCommandLine('dart', ['run', 'my_script.dart'], null);
// Kết quả:
//   Lỗi tại thời điểm biên dịch
```

Nếu muốn spread một collection nullable, hãy sử dụng
phần tử spread null-aware. Ví dụ sau hợp lệ
vì toán tử spread null-aware được sử dụng trên `extraOptions`.

<?code-excerpt "misc/test/language_tour/collections/null_spread_operator_in_collection_a.dart (code_sample)"?>
```dart
List<String> buildCommandLine(
  String executable,
  List<String> options, [
  List<String>? extraOptions,
]) {
  return [
    executable,
    ...options,
    ...?extraOptions, // <-- Bây giờ đã hợp lệ.
  ];
}

// Cách sử dụng:
//   buildCommandLine('dart', ['run', 'my_script.dart'], null);
// Kết quả:
//   [dart, run, my_script.dart]
```

Để tìm hiểu thêm về toán tử spread null-aware, hãy xem
[Toán tử spread][].

[Toán tử spread]: /language/operators/#spread-operators

<a id="if-element"></a>

### Phần tử if

Phần tử `if` đánh giá có điều kiện một phần tử bên trong
dựa trên biểu thức điều kiện đã cho, đồng thời tùy chọn
đánh giá một phần tử `else` khác nếu điều kiện là false.

Phần tử `if` có một số biến thể cú pháp:

```dart
// Nếu biểu thức bool là true, hãy đưa kết quả vào.
if (<bool_expression>) <result>
```

```dart
// Nếu biểu thức khớp với pattern, hãy đưa kết quả vào.
if (<expression> case <pattern>) <result>
```

```dart
// Nếu thao tác cho kết quả true, hãy đưa kết quả đầu tiên vào;
// nếu không, hãy đưa kết quả thứ hai vào.
if (<bool_expression>) <result> else <result>
```

```dart
// Nếu thao tác cho kết quả true, hãy đưa kết quả đầu tiên vào;
// nếu không, hãy đưa kết quả thứ hai vào.
if (<expression> case <pattern>) <result> else <result>
```

Các ví dụ sau minh họa nhiều cách khác nhau để
bạn sử dụng phần tử `if` bên trong collection với
một biểu thức boolean:

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_e.dart (code_sample)"?>
```dart
var includeItem = true;
var items = [0, if (includeItem) 1, 2, 3]; // [0, 1, 2, 3]
```

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_f.dart (code_sample)"?>
```dart
var includeItem = true;
var items = [0, if (!includeItem) 1, 2, 3]; // [0, 2, 3]
```

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_g.dart (code_sample)"?>
```dart
var name = 'apple';
var items = [0, if (name == 'orange') 1 else 10, 2, 3]; // [0, 10, 2, 3]
```

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_h.dart (code_sample)"?>
```dart
var name = 'apple';
var items = [
  0,
  if (name == 'kiwi') 1 else if (name == 'pear') 10,
  2,
  3,
]; // [0, 2, 3]
```

Các ví dụ sau minh họa nhiều cách khác nhau để
bạn sử dụng phần tử `if` có phần `case` bên trong
collection:

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_a.dart (code_sample)"?>
```dart
Object data = 123;
var typeInfo = [
  if (data case int i) 'Data is an integer: $i',
  if (data case String s) 'Data is a string: $s',
  if (data case bool b) 'Data is a boolean: $b',
  if (data case double d) 'Data is a double: $d',
]; // [Data is an integer: 123]
```

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_d.dart (code_sample)"?>
```dart
var word = 'hello';
var items = [
  1,
  if (word case String(length: var wordLength)) wordLength,
  3,
]; // [1, 5, 3]
```

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_b.dart (code_sample)"?>
```dart
var orderDetails = ['Apples', 12, ''];
var summary = [
  'Product: ${orderDetails[0]}',
  if (orderDetails case [_, int qty, _]) 'Quantity: $qty',
  if (orderDetails case [_, _, ''])
    'Delivery: Not Started'
  else
    'Delivery: In Progress',
]; // [Product: Apples, Quantity: 12, Delivery: Not Started]
```

Bạn có thể kết hợp các thao tác `if` khác nhau với
phần `else if`. Ví dụ:

<?code-excerpt "misc/test/language_tour/collections/if_case_operator_in_collection_c.dart (code_sample)"?>
```dart
var a = 'apple';
var b = 'orange';
var c = 'mango';
var items = [
  0,
  if (a == 'apple') 1 else if (a case 'mango') 10,
  if (b case 'pear') 2 else if (b == 'mango') 20,
  if (c case 'apple') 3 else if (c case 'mango') 30,
  4,
]; // [0, 1, 30, 4]
```

Để tìm hiểu thêm về điều kiện `if`, hãy xem
[câu lệnh `if`][]. Để tìm hiểu thêm về điều kiện `if-case`,
hãy xem [câu lệnh `if-case`][].

[câu lệnh `if`]: /language/branches#if
[câu lệnh `if-case`]: /language/branches#if-case

<a id="for-element"></a>

### Phần tử for

Phần tử `for` lặp và liên tục đánh giá một
phần tử bên trong đã cho, chèn không hoặc nhiều giá trị kết quả.

Phần tử `for` có cú pháp sau trong collection:

```dart
for (<expression> in <collection>) <result>
```

```dart
for (<initialization_clause>; <condition_clause>; <increment_clause>) <result>
```

Các ví dụ sau minh họa nhiều cách khác nhau để
bạn sử dụng phần tử `for` bên trong collection:

<?code-excerpt "misc/test/language_tour/collections/for_loop_in_collection_a.dart (code_sample)"?>
```dart
var numbers = [2, 3, 4];
var items = [1, for (var n in numbers) n * n, 7]; // [1, 4, 9, 16, 7]
```

<?code-excerpt "misc/test/language_tour/collections/for_loop_in_collection_b.dart (code_sample)"?>
```dart
var items = [1, for (var x = 5; x > 2; x--) x, 7]; // [1, 5, 4, 3, 7]
```

<?code-excerpt "misc/test/language_tour/collections/for_loop_in_collection_c.dart (code_sample)"?>
```dart
var items = [1, for (var x = 2; x < 4; x++) x, 7]; // [1, 2, 3, 7]
```

Để tìm hiểu thêm về vòng lặp `for`, hãy xem
[vòng lặp `for`][].

[vòng lặp `for`]: /language/loops/#for-loops

<a id="nesting-elements"></a>

### Lồng các phần tử control flow

Bạn có thể lồng các phần tử control flow vào nhau. Đây
là một lựa chọn thay thế mạnh mẽ cho list comprehension trong các
ngôn ngữ khác.

Trong ví dụ sau, chỉ các số chẵn trong
`numbers` được đưa vào `items`.

<?code-excerpt "misc/test/language_tour/collections/nesting_in_collection_a.dart (code_sample)"?>
```dart
var numbers = [1, 2, 3, 4, 5, 6, 7];
var items = [
  0,
  for (var n in numbers)
    if (n.isEven) n,
  8,
]; // [0, 2, 4, 6, 8]
```

Việc sử dụng spread trên một collection literal ngay bên trong
phần tử `if` hoặc `for` là cách viết phổ biến và đúng thành ngữ. Ví dụ:

<?code-excerpt "misc/test/language_tour/collections/nesting_in_collection_c.dart (code_sample)"?>
```dart
var items = [
  if (condition) oneThing(),
  if (condition) ...[multiple(), things()],
]; // [oneThing, [multiple_a, multiple_b], things]
```

Bạn có thể lồng mọi loại phần tử sâu tùy ý.
Trong ví dụ sau, các phần tử `if`, `for` và spread
được lồng vào nhau trong một collection:

<?code-excerpt "misc/test/language_tour/collections/nesting_in_collection_b.dart (code_sample)"?>
```dart
var nestItems = true;
var ys = [1, 2, 3, 4];
var items = [
  if (nestItems) ...[
    for (var x = 0; x < 3; x++)
      for (var y in ys)
        if (x < y) x + y * 10,
  ],
]; // [10, 20, 30, 40, 21, 31, 41, 32, 42]
```
