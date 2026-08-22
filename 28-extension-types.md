---
title: Extension types
description: Tìm hiểu cách viết interface chỉ có tính static cho một kiểu hiện có.
prevpage:
  url: /language/extension-methods
  title: Extension method
nextpage:
  url: /language/callable-objects
  title: Object có thể gọi
---

Extension type là một abstraction tại compile time “bọc”
một kiểu hiện có bằng một interface khác chỉ có tính static.
Chúng là thành phần quan trọng của [static JS interop][] vì
có thể dễ dàng sửa đổi interface của một kiểu hiện có
(điều thiết yếu đối với mọi dạng interop)
mà không phát sinh chi phí của một wrapper thực sự.

Extension type áp đặt kỷ luật lên tập hợp thao tác (hay interface)
khả dụng cho object của một kiểu nền,
được gọi là *representation type*.
Khi định nghĩa interface của extension type,
bạn có thể chọn tái sử dụng một số member của representation type,
bỏ những member khác, thay thế những member khác và thêm chức năng mới.

Ví dụ sau bọc kiểu `int` để tạo một extension type
chỉ cho phép các thao tác có ý nghĩa đối với số ID:

```dart
extension type IdNumber(int id) {
  // Bọc toán tử '<' của kiểu 'int':
  operator <(IdNumber other) => id < other.id;
  // Chẳng hạn, không khai báo toán tử '+',
  // vì phép cộng không có ý nghĩa đối với số ID.
}

void main() {
  // Khi không có sự kiểm soát của extension type,
  // 'int' để lộ số ID cho các thao tác không an toàn:
  int myUnsafeId = 42424242;
  myUnsafeId = myUnsafeId + 10; // Hoạt động, nhưng không nên được phép với ID.

  var safeId = IdNumber(42424242);
  safeId + 10; // Lỗi compile-time: Không có toán tử '+'.
  myUnsafeId = safeId; // Lỗi compile-time: Sai kiểu.
  myUnsafeId = safeId as int; // HỢP LỆ: Cast tại runtime sang representation type.
  safeId < IdNumber(42424241); // HỢP LỆ: Dùng toán tử '<' được bọc.
}
```

:::note
Extension type phục vụ cùng mục đích như **wrapper class**,
nhưng không yêu cầu tạo thêm object tại runtime,
điều có thể trở nên tốn kém khi bạn cần bọc nhiều object.
Vì extension type chỉ có tính static và bị loại bỏ khi biên dịch nên tại runtime,
về cơ bản chúng không gây chi phí.

[**Extension method**][ext] (cũng được gọi đơn giản là “extension”)
là một abstraction static tương tự extension type.
Tuy nhiên, extension method bổ sung chức năng *trực tiếp*
cho mọi instance của kiểu nền.
Extension type thì khác;
interface của extension type *chỉ* áp dụng cho biểu thức
có static type là extension type đó.
Theo mặc định, chúng khác với interface của kiểu nền.
:::

[static JS interop]: /go/next-gen-js-interop
[ext]: /language/extension-methods

<a id="syntax"></a>
## Cú pháp

<a id="declaration"></a>
### Khai báo

Định nghĩa một extension type mới bằng khai báo `extension type` và một tên,
theo sau là *khai báo representation type* trong dấu ngoặc đơn:

```dart
extension type E(int i) {
  // Định nghĩa tập hợp thao tác.
}
```

Khai báo representation type `(int i)` chỉ định rằng
kiểu nền của extension type `E` là `int`,
và tham chiếu đến *representation object* có tên `i`.
Khai báo này cũng đưa vào:

- Một getter ngầm định cho representation object
  có representation type làm return type: `int get i`.
- Một constructor ngầm định: `E(int i) : i = i`.

Representation getter cung cấp quyền truy cập vào
representation object được định kiểu là kiểu nền.
Getter nằm trong scope của phần thân extension type,
và bạn có thể truy cập nó bằng tên giống như mọi getter khác:

- Bên trong phần thân extension type bằng `i` (hoặc `this.i`).
- Bên ngoài bằng phép trích xuất property `e.i`
  (trong đó `e` có extension type làm static type).

Khai báo extension type cũng có thể bao gồm [type parameter][generics]
giống như lớp hoặc extension:

```dart
extension type E<T>(List<T> elements) {
  // ...
}
```

[generics]: /language/generics

<a id="constructors"></a>
### Constructor

Bạn có thể tùy chọn khai báo [constructor][constructors] trong phần thân extension type.
Bản thân khai báo representation là một constructor ngầm định,
vì vậy theo mặc định nó thay thế unnamed constructor của extension type.
Mọi generative constructor không redirect bổ sung đều phải
khởi tạo instance variable của representation object
bằng `this.i` trong initializer list hoặc formal parameter.

```dart
extension type E(int i) {
  E.n(this.i);
  E.m(int j, String foo) : i = j + foo.length;
}

void main() {
  E(4); // Unnamed constructor ngầm định.
  E.n(3); // Named constructor.
  E.m(5, "Hello!"); // Named constructor có thêm parameter.
}
```

Hoặc bạn có thể đặt tên cho constructor khai báo representation,
trong trường hợp đó sẽ có chỗ cho unnamed constructor trong phần thân:

```dart
extension type const E._(int it) {
  E(): this._(42);
  E.otherName(this.it);
}

void main2() {
  E();
  const E._(2);
  E.otherName(3);
}
```

Bạn cũng có thể ẩn hoàn toàn constructor, thay vì chỉ định nghĩa một constructor mới,
bằng cùng cú pháp constructor private `_` dành cho lớp. Ví dụ,
nếu bạn chỉ muốn client dựng `E` bằng `String`, mặc dù
kiểu nền là `int`:

```dart
extension type E._(int i) {
  E.fromString(String foo) : i = int.parse(foo);
}
```

Bạn cũng có thể khai báo forwarding generative constructor,
hoặc [factory constructor][factory]
(cũng có thể chuyển tiếp đến constructor của sub-extension type).

[constructors]: /language/constructors
[factory]: /language/constructors#factory-constructors

<a id="members"></a>
### Member

Khai báo member trong phần thân extension type để định nghĩa interface của nó
theo cùng cách bạn làm với member của lớp.
Member của extension type có thể là method, getter, setter hoặc operator
([instance variable][instance variables] không phải [`external`][] và [abstract member][abstract members]
không được phép):

```dart
extension type NumberE(int value) {
  // Operator:
  NumberE operator +(NumberE other) =>
      NumberE(value + other.value);
  // Getter:
  NumberE get myNum => this;
  // Method:
  bool isValid() => !value.isNegative;
}
```

Theo [mặc định](#transparency), interface member của representation type
không phải interface member của extension type.
Để làm cho một member riêng lẻ của representation type khả dụng
trên extension type, bạn phải viết khai báo cho nó
trong định nghĩa extension type, giống `operator +` trong `NumberE`.
Bạn cũng có thể định nghĩa member mới không liên quan đến representation type,
như getter `i` và method `isValid`.

[`external`]: /language/functions#external
[instance variables]: /language/classes#instance-variables
[abstract members]: /language/methods#abstract-methods

<a id="implements"></a>
### Implements

Bạn có thể tùy chọn dùng mệnh đề `implements` để:

- Đưa vào một quan hệ subtype trên extension type, VÀ
- Thêm member của representation object vào interface của extension type.

Mệnh đề `implements` đưa vào một quan hệ [applicability][]
giống quan hệ giữa [extension method][ext] và kiểu `on` của nó.
Member áp dụng được cho supertype cũng áp dụng được cho
subtype, trừ khi subtype có một khai báo với cùng tên member.

Extension type chỉ có thể implement:

- **Representation type của chính nó**.
  Điều này làm cho tất cả member của representation type ngầm khả dụng
  cho extension type.

  ```dart
  extension type NumberI(int i)
    implements int{
    // 'NumberI' có thể gọi tất cả member của 'int',
    // cộng với mọi thứ khác nó khai báo ở đây.
  }
  ```

- **Một supertype của representation type**.
  Điều này làm cho member của supertype khả dụng,
  nhưng không nhất thiết là tất cả member của representation type.

  ```dart
  extension type Sequence<T>(List<T> _) implements Iterable<T> {
    // Các thao tác tốt hơn List.
  }

  extension type Id(int _id) implements Object {
    // Làm cho extension type trở thành non-nullable.
    static Id? tryParse(String source) => int.tryParse(source) as Id?;
  }
  ```

- **Một extension type khác** hợp lệ trên cùng representation type.
  Điều này cho phép bạn tái sử dụng thao tác trên nhiều extension type
  (tương tự đa kế thừa).

  ```dart
  extension type const Opt<T>._(({T value})? _) {
    const factory Opt(T value) = Val<T>;
    const factory Opt.none() = Non<T>;
  }
  extension type const Val<T>._(({T value}) _) implements Opt<T> {
    const Val(T value) : this._((value: value));
    T get value => _.value;
  }
  extension type const Non<T>._(Null _) implements Opt<Never> {
    const Non() : this._(null);
  }
  ```

Đọc mục [Sử dụng](#usage) để tìm hiểu thêm về tác động của `implements`
trong các tình huống khác nhau.

[applicability]: {{site.repo.dart.lang}}/blob/main/accepted/2.7/static-extension-methods/feature-specification.md#examples
[ext]: /language/extension-methods

<a id="redeclare"></a>
#### `@redeclare`

Việc khai báo một member của extension type
trùng tên với member của supertype *không* phải là quan hệ
override như giữa các lớp, mà là một *redeclaration*.
Khai báo member của extension type *thay thế hoàn toàn* mọi
member của supertype có cùng tên.
Không thể cung cấp một
implementation thay thế cho cùng một function.

Bạn có thể dùng annotation [`@redeclare`][] từ `package:meta` để
cho compiler biết bạn *có chủ ý* chọn
dùng cùng tên với member của supertype.
Sau đó analyzer sẽ cảnh báo nếu điều đó không thực sự đúng,
ví dụ nếu một trong các tên bị gõ sai.

<?code-excerpt "language/lib/extension_types/redeclare.dart"?>
```dart
import 'package:meta/meta.dart';

extension type MyString(String _) implements String {
  // Thay thế 'String.operator[]'.
  @redeclare
  int operator [](int index) => codeUnitAt(index);
}
```

Bạn cũng có thể bật lint [`annotate_redeclares`][]
để nhận cảnh báo nếu khai báo một method của extension type
ẩn member của superinterface mà *không* được annotate bằng `@redeclare`.

[`@redeclare`]: {{site.pub-api}}/meta/latest/meta/redeclare-constant.html
[`annotate_redeclares`]: /tools/linter-rules/annotate_redeclares

<a id="usage"></a>
## Sử dụng

Để dùng một extension type, hãy tạo instance giống như với một lớp:
bằng cách gọi constructor:

```dart
extension type NumberE(int value) {
  NumberE operator +(NumberE other) =>
      NumberE(value + other.value);

  NumberE get next => NumberE(value + 1);
  bool isValid() => !value.isNegative;
}

void testE() {
  var num = NumberE(1);
}
```

Sau đó, bạn có thể gọi member trên object giống như với object của lớp.

Extension type có hai use case cốt lõi đều hợp lệ như nhau,
nhưng khác biệt đáng kể:

1. Cung cấp interface *mở rộng* cho một kiểu hiện có.
2. Cung cấp interface *khác* cho một kiểu hiện có.

:::note
Trong mọi trường hợp, representation type của extension type không bao giờ là subtype của nó,
vì vậy representation type không thể được dùng thay thế cho nhau tại nơi
cần extension type.
:::

<a id="transparency" aria-hidden="true"></a>
<a id="1-provide-an-extended-interface-to-an-existing-type" aria-hidden="true"></a>

### 1. Cung cấp interface *mở rộng* cho một kiểu hiện có {: #provide-extended-interface}

Khi extension type [implement](#implements) representation type của nó,
bạn có thể xem nó là “transparent”, vì nó cho phép extension type
“nhìn thấy” kiểu nền.

Extension type transparent có thể gọi tất cả member của
representation type (không bị [redeclare](#redeclare)),
cộng với mọi auxiliary member mà nó định nghĩa.
Điều này tạo ra một interface mới, *mở rộng* cho kiểu hiện có.
Interface mới khả dụng cho các biểu thức
có static type là extension type.

Điều này có nghĩa bạn *có thể* gọi member của representation type
(không giống extension type [không transparent](#provide-different-interface)),
như sau:

```dart
extension type NumberT(int value)
  implements int {
  // Không khai báo tường minh bất kỳ member nào của 'int'.
  NumberT get i => this;
}

void main () {
  // Tất cả đều HỢP LỆ: Tính transparent cho phép gọi member `int` trên extension type:
  var v1 = NumberT(1); // kiểu của v1: NumberT
  int v2 = NumberT(2); // kiểu của v2: int
  var v3 = v1.i - v1;  // kiểu của v3: int
  var v4 = v2 + v1; // kiểu của v4: int
  var v5 = 2 + v1; // kiểu của v5: int
  // Lỗi: Interface của extension type không khả dụng cho representation type
  v2.i;
}
```

Bạn cũng có thể có extension type “gần như transparent”
bổ sung member mới và điều chỉnh member khác bằng cách redeclare một tên member nhất định
từ supertype.
Ví dụ, điều này cho phép bạn dùng kiểu chặt chẽ hơn cho một số parameter của method,
hoặc giá trị mặc định khác.

Một cách tiếp cận extension type gần như transparent khác là implement
một kiểu là supertype của representation type.
Ví dụ, nếu representation type là private nhưng supertype của nó
định nghĩa phần interface quan trọng đối với client.

<a id="2-provide-a-different-interface-to-an-existing-type" aria-hidden="true"></a>

### 2. Cung cấp interface *khác* cho một kiểu hiện có {:#provide-different-interface}

Một extension type không [transparent](#transparency)
(không [`implement`](#implements) representation type của nó)
được xử lý về mặt static như một kiểu hoàn toàn mới,
khác biệt với representation type.
Bạn không thể gán nó cho representation type,
và nó không để lộ member của representation type.

Ví dụ, hãy xem extension type `NumberE` mà chúng ta đã khai báo trong phần [Sử dụng](#usage):

```dart
void testE() {
  var num1 = NumberE(1);
  int num2 = NumberE(2); // Lỗi: Không thể gán 'NumberE' cho 'int'.

  num1.isValid(); // HỢP LỆ: Gọi extension member.
  num1.isNegative(); // Lỗi: 'NumberE' không định nghĩa member 'isNegative' của 'int'.

  var sum1 = num1 + num1; // HỢP LỆ: 'NumberE' định nghĩa '+'.
  var diff1 = num1 - num1; // Lỗi: 'NumberE' không định nghĩa member '-' của 'int'.
  var diff2 = num1.value - 2; // HỢP LỆ: Có thể truy cập representation object bằng tham chiếu.
  var sum2 = num1 + 2; // Lỗi: Không thể gán 'int' cho kiểu parameter 'NumberE'.

  List<NumberE> numbers = [
    NumberE(1),
    num1.next, // HỢP LỆ: Getter 'next' trả về kiểu 'NumberE'.
    1, // Lỗi: Không thể gán element 'int' cho kiểu list 'NumberE'.
  ];
}
```

Bạn có thể dùng extension type theo cách này để *thay thế* interface
của một kiểu hiện có. Điều này cho phép bạn mô hình hóa một interface
có ý nghĩa đối với các ràng buộc của kiểu mới
(như ví dụ `IdNumber` trong phần giới thiệu), đồng thời hưởng lợi từ
hiệu năng và sự tiện lợi của một kiểu đơn giản được định nghĩa sẵn như `int`.

Use case này gần với khả năng đóng gói hoàn toàn
của wrapper class nhất (nhưng trên thực tế chỉ là một
abstraction được bảo vệ [*phần nào*](#type-considerations)).

<a id="type-considerations"></a>
## Những điều cần cân nhắc về kiểu

Extension type là cấu trúc bọc tại compile time.
Tại runtime, hoàn toàn không còn dấu vết của extension type.
Mọi truy vấn kiểu hoặc thao tác runtime tương tự đều hoạt động trên representation type.

Điều này khiến extension type trở thành một abstraction *không an toàn*,
vì bạn luôn có thể tìm ra representation type tại runtime
và truy cập object nền.

Kiểm tra dynamic type (`e is T`), cast (`e as T`)
và các truy vấn kiểu runtime khác (như `switch (e) ...` hoặc `if (e case ...)`)
đều đánh giá thành representation object nền,
và kiểm tra kiểu dựa trên runtime type của object đó.
Điều này đúng khi static type của `e` là extension type,
và khi kiểm tra với extension type (`case MyExtensionType(): ... `).

```dart
void main() {
  var n = NumberE(1);

  // Runtime type của 'n' là representation type 'int'.
  if (n is int) print(n); // In ra 1.

  // Có thể dùng method của 'int' trên 'n' tại runtime.
  if (n case int x) print(x.toRadixString(10)); // In ra 1.
  switch (n) {
    case int(:var isEven): print("$n (${isEven ? "even" : "odd"})"); // In ra 1 (odd).
  }
}
```

Tương tự, trong ví dụ này, static type của giá trị được match
là static type của extension type:

```dart
void main() {
  int i = 2;
  if (i is NumberE) print("It is"); // In ra 'It is'.
  if (i case NumberE v) print("value: ${v.value}"); // In ra 'value: 2'.
  switch (i) {
    case NumberE(:var value): print("value: $value"); // In ra 'value: 2'.
  }
}
```

Khi `i` nhận static type `NumberE` dựa trên cast hoặc pattern match, `i`
vẫn tham chiếu đến cùng object, `v` tham chiếu đến cùng object như `i`, và
bản thân object vẫn không thay đổi. Chỉ static type thay đổi
(và do đó các method chúng ta có thể gọi trên object này thay đổi). Đặc biệt,
việc thay đổi kiểu không liên quan đến lời gọi constructor. Nếu muốn
thực thi constructor (ví dụ để thực hiện một dạng validation), bạn
cần viết lời gọi constructor tường minh (chẳng hạn `NumberE(i)`).

Điều quan trọng là phải nhận thức được hành vi này khi dùng extension type.
Luôn ghi nhớ rằng extension type tồn tại và có ý nghĩa tại compile time,
nhưng bị _xóa_ trong quá trình biên dịch.

Ví dụ, hãy xem một biểu thức `e` có static type là
extension type `E`. Giả sử representation type của `E` là `R`.
Runtime type của giá trị `e` khi đó là subtype của `R`.
Ngay cả bản thân kiểu cũng bị xóa;
`List<E>` hoàn toàn giống `List<R>` tại runtime.

Nói cách khác, wrapper class thực sự có thể đóng gói object được bọc,
trong khi extension type chỉ là một view tại compile time trên object được bọc.
Mặc dù wrapper thực sự an toàn hơn, đánh đổi lại là extension type
cho bạn lựa chọn tránh wrapper object, điều có thể
cải thiện đáng kể hiệu năng trong một số tình huống.
