---
title: Records
description: Tóm tắt về cấu trúc dữ liệu record trong Dart.
prevpage:
  url: /language/built-in-types
  title: Các kiểu dựng sẵn
nextpage:
  url: /language/collections
  title: Collection
---

:::version-note
  Record yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 3.0.
:::

Record là một kiểu tổng hợp ẩn danh, immutable. Giống những [kiểu collection][] khác,
record cho phép bạn gộp nhiều đối tượng vào một đối tượng duy nhất. Không giống các kiểu
collection khác, record có kích thước cố định, không đồng nhất và có kiểu.

Record là những giá trị thực; bạn có thể lưu trữ chúng trong biến,
lồng chúng, truyền chúng vào và trả chúng về từ hàm,
cũng như lưu chúng trong những cấu trúc dữ liệu như list, map và set.

<a id="record-syntax"></a>

## Cú pháp record

_Biểu thức record_ là danh sách các field được đặt tên hoặc theo vị trí, phân tách bằng dấu phẩy
và đặt trong dấu ngoặc đơn:

<?code-excerpt "language/test/records_test.dart (record-syntax)"?>
```dart
var record = ('first', a: 2, b: true, 'last');
```

_Record type annotation_ là danh sách các kiểu phân tách bằng dấu phẩy và đặt trong dấu ngoặc đơn.
Bạn có thể sử dụng record type annotation để định nghĩa kiểu trả về và kiểu tham số.
Ví dụ, các phần `(int, int)` trong đoạn sau là record type annotation:

<?code-excerpt "language/test/records_test.dart (record-type-annotation)"?>
```dart
(int, int) swap((int, int) record) {
  var (a, b) = record;
  return (b, a);
}
```

Các field trong biểu thức record và type annotation phản ánh cách
[tham số và đối số][] hoạt động trong hàm.
Field theo vị trí được đặt trực tiếp bên trong dấu ngoặc đơn:

<?code-excerpt "language/test/records_test.dart (record-type-declaration)"?>
```dart
// Record type annotation trong một khai báo biến:
(String, int) record;

// Khởi tạo biến bằng một biểu thức record:
record = ('A string', 123);
```

Trong record type annotation, field được đặt tên nằm trong một phần
được bao bởi dấu ngoặc nhọn gồm các cặp kiểu-và-tên, sau tất cả field theo vị trí. Trong biểu thức
record, tên nằm trước từng giá trị field và theo sau bởi dấu hai chấm:

<?code-excerpt "language/test/records_test.dart (record-type-named-declaration)"?>
```dart
// Record type annotation trong một khai báo biến:
({int a, bool b}) record;

// Khởi tạo biến bằng một biểu thức record:
record = (a: 123, b: true);
```

Tên của các field được đặt tên trong một kiểu record là một phần của
[định nghĩa kiểu của record](#record-types), hay _shape_ của record.
Hai record có các field được đặt tên bằng những tên
khác nhau sẽ có kiểu khác nhau:

<?code-excerpt "language/test/records_test.dart (record-type-mismatched-names)"?>
```dart
({int a, int b}) recordAB = (a: 1, b: 2);
({int x, int y}) recordXY = (x: 3, y: 4);

// Lỗi biên dịch! Các record này không có cùng kiểu.
// recordAB = recordXY;
```

Trong record type annotation, bạn cũng có thể đặt tên cho các field *theo vị trí*, nhưng
những tên này chỉ dùng cho mục đích lập tài liệu và không ảnh hưởng đến kiểu của record:

<?code-excerpt "language/test/records_test.dart (record-type-matched-names)"?>
```dart
(int a, int b) recordAB = (1, 2);
(int x, int y) recordXY = (3, 4);

recordAB = recordXY; // OK.
```

Điều này tương tự cách các tham số theo vị trí
trong một [khai báo hàm hoặc function typedef][function-type] có thể mang tên,
nhưng những tên đó không ảnh hưởng đến signature của hàm.

Để biết thêm thông tin và ví dụ, hãy xem
[Kiểu record](#record-types) và [Tính bằng nhau của record](#record-equality).

<a id="record-fields"></a>

## Field của record

Field của record có thể được truy cập thông qua các getter dựng sẵn. Record là immutable,
vì vậy field không có setter.

Field được đặt tên cung cấp getter cùng tên. Field theo vị trí cung cấp getter
có tên `$<position>`, bỏ qua các field được đặt tên:

<?code-excerpt "language/test/records_test.dart (record-getters)"?>
```dart
var record = ('first', a: 2, b: true, 'last');

print(record.$1); // In ra 'first'
print(record.a); // In ra 2
print(record.b); // In ra true
print(record.$2); // In ra 'last'
```

Để tinh giản hơn nữa việc truy cập field của record,
hãy xem trang về [Pattern][pattern].

<a id="record-types"></a>

## Kiểu record

Không có khai báo kiểu dành cho từng kiểu record riêng lẻ.
Record được định kiểu theo cấu trúc dựa trên kiểu của các field.
_Shape_ của record (tập hợp field, kiểu của các field
và tên của chúng nếu có) xác định duy nhất kiểu của record.

Mỗi field trong record có kiểu riêng. Kiểu field có thể khác nhau trong cùng một
record. Hệ thống kiểu nhận biết kiểu của từng field ở bất cứ nơi nào field được truy cập
từ record:

<?code-excerpt "language/test/records_test.dart (record-getters-two)"?>
```dart
(num, Object) pair = (42, 'a');

var first = pair.$1; // Kiểu static `num`, kiểu runtime `int`.
var second = pair.$2; // Kiểu static `Object`, kiểu runtime `String`.
```

Hãy xét hai library không liên quan cùng tạo record có cùng một tập hợp field.
Hệ thống kiểu hiểu rằng những record đó có cùng kiểu dù các
library không phụ thuộc vào nhau.

:::tip
Mặc dù không thể khai báo một kiểu duy nhất cho một shape record,
bạn có thể tạo bí danh kiểu để tăng khả năng đọc và tái sử dụng.
Để tìm hiểu thời điểm và cách thực hiện,
hãy xem [Record và typedef](#records-and-typedefs).
:::

<a id="record-equality"></a>

## Tính bằng nhau của record

Hai record bằng nhau nếu chúng có cùng _shape_ (tập hợp field)
và các field tương ứng có cùng giá trị.
Vì _thứ tự_ field được đặt tên không phải một phần trong shape của record, thứ tự của các
field được đặt tên không ảnh hưởng đến tính bằng nhau.

Ví dụ:

<?code-excerpt "language/test/records_test.dart (record-shape)"?>
```dart
(int x, int y, int z) point = (1, 2, 3);
(int r, int g, int b) color = (1, 2, 3);

print(point == color); // In ra 'true'.
```

<?code-excerpt "language/test/records_test.dart (record-shape-mismatch)"?>
```dart
({int x, int y, int z}) point = (x: 1, y: 2, z: 3);
({int r, int g, int b}) color = (r: 1, g: 2, b: 3);

print(point == color); // In ra 'false'. Lint: bằng nhau trên các kiểu không liên quan.
```

Record tự động định nghĩa các phương thức `hashCode` và `==` dựa trên cấu trúc
các field của chúng.

<a id="multiple-returns"></a>

## Trả về nhiều giá trị

Record cho phép hàm trả về nhiều giá trị được gộp lại.
Để lấy các giá trị record từ giá trị trả về,
hãy [destructure][] các giá trị vào biến cục bộ bằng [pattern matching][pattern].

<?code-excerpt "language/test/records_test.dart (record-multiple-returns)"?>
```dart
// Trả về nhiều giá trị trong một record:
(String name, int age) userInfo(Map<String, dynamic> json) {
  return (json['name'] as String, json['age'] as int);
}

final json = <String, dynamic>{'name': 'Dash', 'age': 10, 'color': 'blue'};

// Destructure bằng record pattern có các field theo vị trí:
var (name, age) = userInfo(json);

/* Tương đương với:
  var info = userInfo(json);
  var name = info.$1;
  var age  = info.$2;
*/
```

Bạn cũng có thể destructure một record bằng [các field được đặt tên](#record-fields),
sử dụng cú pháp dấu hai chấm `:`; bạn có thể đọc thêm về cú pháp này trên trang
[Các kiểu pattern][]:

<?code-excerpt "language/test/records_test.dart (record-name-destructure)"?>
```dart
({String name, int age}) userInfo(Map<String, dynamic> json)
// ···
// Destructure bằng record pattern có các field được đặt tên:
final (:name, :age) = userInfo(json);
```

Bạn có thể trả về nhiều giá trị từ một hàm mà không dùng record,
nhưng những phương pháp khác đều có nhược điểm.
Ví dụ, việc tạo một lớp dài dòng hơn nhiều, còn sử dụng những kiểu collection
khác như `List` hoặc `Map` sẽ làm mất type safety.

:::note
Đặc điểm trả về nhiều giá trị và có kiểu không đồng nhất của record cho phép
thực thi song song các future thuộc những kiểu khác nhau; bạn có thể đọc thêm trong
[tài liệu `dart:async`][].
:::

<a id="records-as-simple-data-structures"></a>

## Record dưới dạng cấu trúc dữ liệu đơn giản

Record chỉ lưu giữ dữ liệu. Khi đó là tất cả những gì bạn cần,
record có sẵn ngay lập tức và dễ sử dụng
mà không cần khai báo bất kỳ lớp mới nào.
Đối với một danh sách đơn giản gồm các tuple dữ liệu có cùng shape,
một *list record* là cách biểu diễn trực tiếp nhất.

Ví dụ, hãy xem danh sách "định nghĩa button" này:

```dart
final buttons = [
  (
    label: "Button I",
    icon: const Icon(Icons.upload_file),
    onPressed: () => print("Action -> Button I"),
  ),
  (
    label: "Button II",
    icon: const Icon(Icons.info),
    onPressed: () => print("Action -> Button II"),
  )
];
```

Đoạn code này có thể được viết trực tiếp mà không cần bất kỳ khai báo bổ sung nào.

<a id="records-and-typedefs"></a>

### Record và typedef

Bạn có thể chọn sử dụng [typedef][] để đặt tên cho chính kiểu record
và sử dụng tên đó thay vì viết đầy đủ kiểu record.
Phương pháp này cho phép bạn nêu rõ một số field có thể là null (`?`),
ngay cả khi không entry hiện tại nào trong list mang giá trị null.

```dart
typedef ButtonItem = ({String label, Icon icon, void Function()? onPressed});
final List<ButtonItem> buttons = [
  // ...
];
```

Vì kiểu record là kiểu cấu trúc, việc đặt một tên như `ButtonItem`
chỉ đưa vào một bí danh giúp tham chiếu kiểu cấu trúc dễ dàng hơn:
`({String label, Icon icon, void Function()? onPressed})`.

Việc để toàn bộ code tham chiếu đến một kiểu record bằng bí danh giúp bạn dễ dàng
thay đổi phần triển khai của record sau này mà không cần cập nhật mọi tham chiếu.

Code có thể làm việc với các định nghĩa button đã cho theo cách tương tự
như với những instance lớp đơn giản:

```dart
List<Container> widget = [
  for (var button in buttons)
    Container(
      margin: const EdgeInsets.all(4.0),
      child: OutlinedButton.icon(
        onPressed: button.onPressed,
        icon: button.icon,
        label: Text(button.label),
      ),
    ),
];
```

Bạn thậm chí có thể quyết định đổi kiểu record thành kiểu lớp sau này để thêm phương thức:

```dart
class ButtonItem {
  final String label;
  final Icon icon;
  final void Function()? onPressed;
  ButtonItem({required this.label, required this.icon, this.onPressed});
  bool get hasOnPressed => onPressed != null;
}
```

Hoặc thành một [extension type][]:

```dart
extension type ButtonItem._(({String label, Icon icon, void Function()? onPressed}) _) {
  String get label => _.label;
  Icon get icon => _.icon;
  void Function()? get onPressed => _.onPressed;
  ButtonItem({required String label, required Icon icon, void Function()? onPressed})
      : this._((label: label, icon: icon, onPressed: onPressed));
  bool get hasOnPressed => _.onPressed != null;
}
```

Sau đó, hãy tạo danh sách định nghĩa button bằng các constructor của kiểu đó:

```dart
final List<ButtonItem> buttons =  [
  ButtonItem(
    label: "Button I",
    icon: const Icon(Icons.upload_file),
    onPressed: () => print("Action -> Button I"),
  ),
  ButtonItem(
    label: "Button II",
    icon: const Icon(Icons.info),
    onPressed: () => print("Action -> Button II"),
  )
];
```

Một lần nữa, tất cả đều được thực hiện mà không cần thay đổi code sử dụng list đó.

Việc thay đổi bất kỳ kiểu nào đòi hỏi code sử dụng kiểu phải hết sức cẩn thận
để không đưa ra các giả định. Đối với code dùng bí danh kiểu làm tham chiếu,
bí danh đó không cung cấp sự bảo vệ hoặc bảo đảm nào rằng giá trị được đặt bí danh là một record.
Extension type cũng cung cấp rất ít sự bảo vệ.
Chỉ lớp mới có thể cung cấp đầy đủ tính trừu tượng và đóng gói.

[phiên bản ngôn ngữ]: /language/versioning
[kiểu collection]: /language/collections
[pattern]: /language/patterns#destructuring-multiple-returns
[tài liệu `dart:async`]: /libraries/dart-async#handling-errors-for-multiple-futures
[tham số và đối số]: /language/functions#parameters
[function-type]: /language/functions#function-types
[destructure]: /language/patterns#destructuring
[Các kiểu pattern]: /language/pattern-types#record
[typedef]: /language/typedefs
[extension type]: /language/extension-types
