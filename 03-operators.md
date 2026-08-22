---
title: Toán tử
description: Tìm hiểu về các toán tử được Dart hỗ trợ.
prevpage:
  url: /language/variables
  title: Biến
nextpage:
  url: /language/comments
  title: Chú thích
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

<a name="operators"></a>

Dart hỗ trợ các toán tử được trình bày trong bảng sau.
Bảng này trình bày tính kết hợp của toán tử
và [độ ưu tiên của toán tử](#operator-precedence-example) trong Dart, từ cao xuống thấp;
đây là một cách **xấp xỉ** các mối quan hệ giữa toán tử trong Dart.
Bạn có thể triển khai nhiều [toán tử dưới dạng member của lớp][].

| Mô tả                                   | Toán tử                                                                                            | Tính kết hợp |
|-----------------------------------------|----------------------------------------------------------------------------------------------------|--------------|
| unary postfix                           | *`expr`*`++`    *`expr`*`--`    `()`    `[]`    `?[]`    `.`    `?.`    `!`                        | Không có     |
| unary prefix                            | `-`*`expr`*    `!`*`expr`*    `~`*`expr`*    `++`*`expr`*    `--`*`expr`*      `await` *`expr`*    | Không có     |
| phép nhân                               | `*`    `/`    `%`    `~/`                                                                          | Trái         |
| phép cộng                               | `+`    `-`                                                                                         | Trái         |
| dịch bit                                | `<<`    `>>`    `>>>`                                                                              | Trái         |
| AND theo bit                            | `&`                                                                                                | Trái         |
| XOR theo bit                            | `^`                                                                                                | Trái         |
| OR theo bit                             | <code>&#124;</code>                                                                                | Trái         |
| quan hệ và kiểm tra kiểu                | `>=`    `>`    `<=`    `<`    `as`    `is`    `is!`                                                | Không có     |
| bằng nhau                               | `==`    `!=`                                                                                       | Không có     |
| AND logic                               | `&&`                                                                                               | Trái         |
| OR logic                                | <code>&#124;&#124;</code>                                                                          | Trái         |
| if-null                                 | `??`                                                                                               | Trái         |
| điều kiện                               | *`expr1`*    `?`    *`expr2`*    `:`    *`expr3`*                                                  | Phải         |
| cascade                                 | `..`    `?..`                                                                                      | Trái         |
| phép gán                                | `=`    `*=`    `/=`    `+=`    `-=`    `&=`    `^=`    *v.v.*                                      | Phải         |
| spread ([Xem ghi chú](#spread-operators)) | `...`    `...?`                                                                                  | Không có     |

{:.table .table-striped}

:::warning
Chỉ nên sử dụng bảng trên như một hướng dẫn hữu ích.
Khái niệm độ ưu tiên và tính kết hợp của toán tử
là cách diễn đạt xấp xỉ sự thật trong ngữ pháp của ngôn ngữ.
Bạn có thể tìm thấy hành vi chính thức của các mối quan hệ giữa toán tử trong Dart
ở phần ngữ pháp được định nghĩa trong [Đặc tả ngôn ngữ Dart][].
:::

Khi sử dụng toán tử, bạn tạo ra các biểu thức. Sau đây là một số ví dụ
về biểu thức toán tử:

<?code-excerpt "misc/test/language_tour/operators_test.dart (expressions)" replace="/,//g"?>
```dart
a++
a + b
a = b
a == b
c ? a : b
a is T
```

<a id="operator-precedence-example"></a>

## Ví dụ về độ ưu tiên của toán tử

Trong [bảng toán tử](#operators),
mỗi toán tử có độ ưu tiên cao hơn các toán tử ở những hàng
nằm sau nó. Ví dụ, toán tử nhân `%` có độ ưu tiên cao hơn
(và do đó thực thi trước) toán tử bằng `==`,
toán tử này lại có độ ưu tiên cao hơn toán tử AND logic `&&`. Độ
ưu tiên đó có nghĩa là hai đoạn code sau thực thi theo cùng một
cách:

<?code-excerpt "misc/test/language_tour/operators_test.dart (precedence)"?>
```dart
// Dấu ngoặc đơn cải thiện khả năng đọc.
if ((n % i == 0) && (d % i == 0)) {
  // ...
}

// Khó đọc hơn nhưng tương đương.
if (n % i == 0 && d % i == 0) {
  // ...
}
```

:::warning
Đối với các toán tử nhận hai toán hạng, toán hạng ngoài cùng bên trái quyết định
phương thức nào được sử dụng. Ví dụ, nếu bạn có một đối tượng `Vector` và
một đối tượng `Point`, thì `aVector + aPoint` sử dụng phép cộng (`+`) của `Vector`.
:::


<a id="arithmetic-operators"></a>

## Toán tử số học

Dart hỗ trợ các toán tử số học thông thường như trong bảng sau.

| Toán tử     | Ý nghĩa                                                                     |
|-------------|-----------------------------------------------------------------------------|
| `+`         | Cộng                                                                         |
| `-`         | Trừ                                                                          |
| `-`*`expr`* | Dấu trừ unary, còn gọi là phép phủ định (đảo dấu của biểu thức)              |
| `*`         | Nhân                                                                         |
| `/`         | Chia                                                                         |
| `~/`        | Chia và trả về kết quả là số nguyên                                          |
| `%`         | Lấy phần dư của phép chia số nguyên (modulo)                                 |

{:.table .table-striped}

Ví dụ:

<?code-excerpt "misc/test/language_tour/operators_test.dart (arithmetic)"?>
```dart
assert(2 + 3 == 5);
assert(2 - 3 == -1);
assert(2 * 3 == 6);
assert(5 / 2 == 2.5); // Kết quả là double
assert(5 ~/ 2 == 2); // Kết quả là int
assert(5 % 2 == 1); // Phần dư

assert('5/2 = ${5 ~/ 2} r ${5 % 2}' == '5/2 = 2 r 1');
```

Dart cũng hỗ trợ cả toán tử tăng và giảm dạng prefix lẫn postfix.

| Toán tử     | Ý nghĩa                                                         |
|-------------|-----------------------------------------------------------------|
| `++`*`var`* | *`var`*  `=`  *`var `*  `+ 1` (giá trị biểu thức là *`var`* ` + 1`) |
| *`var`*`++` | *`var`*  `=`  *`var `*  `+ 1` (giá trị biểu thức là *`var`*)       |
| `--`*`var`* | *`var`*  `=`  *`var `*  `- 1` (giá trị biểu thức là *`var`* ` - 1`) |
| *`var`*`--` | *`var`*  `=`  *`var `*  `- 1` (giá trị biểu thức là *`var`*)       |

{:.table .table-striped}

Ví dụ:

<?code-excerpt "misc/test/language_tour/operators_test.dart (increment-decrement)"?>
```dart
int a;
int b;

a = 0;
b = ++a; // Tăng a trước khi b nhận giá trị.
assert(a == b); // 1 == 1

a = 0;
b = a++; // Tăng a sau khi b nhận giá trị.
assert(a != b); // 1 != 0

a = 0;
b = --a; // Giảm a trước khi b nhận giá trị.
assert(a == b); // -1 == -1

a = 0;
b = a--; // Giảm a sau khi b nhận giá trị.
assert(a != b); // -1 != 0
```


<a id="equality-and-relational-operators"></a>

## Toán tử bằng nhau và quan hệ

Bảng sau liệt kê ý nghĩa của các toán tử bằng nhau và quan hệ.

| Toán tử | Ý nghĩa                                      |
|---------|----------------------------------------------|
| `==`    | Bằng; xem phần thảo luận bên dưới            |
| `!=`    | Không bằng                                   |
| `>`     | Lớn hơn                                      |
| `<`     | Nhỏ hơn                                      |
| `>=`    | Lớn hơn hoặc bằng                            |
| `<=`    | Nhỏ hơn hoặc bằng                            |

{:.table .table-striped}

Để kiểm tra xem hai đối tượng x và y có biểu diễn cùng một thứ hay không, hãy sử dụng
toán tử `==`. (Trong trường hợp hiếm gặp khi bạn cần biết liệu hai
đối tượng có chính xác là cùng một đối tượng hay không, hãy sử dụng hàm [identical()][]
thay thế.) Toán tử `==` hoạt động như sau:

1.  Nếu *x* hoặc *y* là null, trả về true nếu cả hai đều là null và false nếu chỉ
    một trong hai là null.

2.  Trả về kết quả của việc gọi phương thức `==` trên *x* với đối số *y*.
    (Đúng vậy, các toán tử như `==` là những phương thức được gọi
    trên toán hạng đầu tiên của chúng.
    Để biết chi tiết, hãy xem [Toán tử][].)

Sau đây là ví dụ sử dụng từng toán tử bằng nhau và
quan hệ:

<?code-excerpt "misc/test/language_tour/operators_test.dart (relational)"?>
```dart
assert(2 == 2);
assert(2 != 3);
assert(3 > 2);
assert(2 < 3);
assert(3 >= 3);
assert(2 <= 3);
```


<a id="type-test-operators"></a>

## Toán tử kiểm tra kiểu

Các toán tử `as`, `is` và `is!` rất tiện dụng để kiểm tra kiểu tại
runtime.

| Toán tử | Ý nghĩa                                                       |
|---------|---------------------------------------------------------------|
| `as`    | Ép kiểu (cũng được dùng để chỉ định [tiền tố library][])       |
| `is`    | True nếu đối tượng có kiểu được chỉ định                       |
| `is!`   | True nếu đối tượng không có kiểu được chỉ định                 |

{:.table .table-striped}

Kết quả của `obj is T` là true nếu `obj` implement interface
do `T` chỉ định. Ví dụ, `obj is Object?` luôn là true.

Chỉ sử dụng toán tử `as` để ép một đối tượng sang một kiểu cụ thể khi và chỉ khi
bạn chắc chắn đối tượng đó thuộc kiểu ấy. Ví dụ:

<?code-excerpt "misc/lib/language_tour/classes/employee.dart (emp-as-person)"?>
```dart
(employee as Person).firstName = 'Bob';
```

Nếu không chắc đối tượng thuộc kiểu `T`, hãy dùng `is T` để kiểm tra
kiểu trước khi sử dụng đối tượng.
<?code-excerpt "misc/lib/language_tour/classes/employee.dart (emp-is-person)"?>
```dart
if (employee is Person) {
  // Kiểm tra kiểu
  employee.firstName = 'Bob';
}
```

:::note
Hai đoạn code không tương đương. Nếu `employee` là null hoặc không phải `Person`,
ví dụ thứ nhất sẽ ném một exception; ví dụ thứ hai không làm gì cả.
:::

<a id="assignment-operators"></a>

## Toán tử gán

Như bạn đã thấy, bạn có thể gán giá trị bằng toán tử `=`.
Để chỉ gán nếu biến nhận phép gán là null,
hãy sử dụng toán tử `??=`.

<?code-excerpt "misc/test/language_tour/operators_test.dart (assignment)"?>
```dart
// Gán giá trị cho a
a = value;
// Gán giá trị cho b nếu b là null; nếu không, b vẫn giữ nguyên
b ??= value;
```

Các toán tử gán kết hợp như `+=` kết hợp
một phép toán với một phép gán.

|      |       |       |        |                      |
|------|-------|-------|--------|----------------------|
| `=`  | `*=`  | `%=`  | `>>>=` | `^=`                 |
| `+=` | `/=`  | `<<=` | `&=`   | <code>&#124;=</code> |
| `-=` | `~/=` | `>>=` |        |                      |

{:.table}

Sau đây là cách các toán tử gán kết hợp hoạt động:

|                           | Phép gán kết hợp     | Biểu thức tương đương |
|---------------------------|----------------------|-----------------------|
| **Đối với toán tử *op*:** | `a `  *`op`*`= b`    | `a = a ` *`op `* `b`  |
| **Ví dụ:**                | `a += b`             | `a = a + b`           |

{:.table}

Ví dụ sau sử dụng các toán tử gán và gán kết hợp:

<?code-excerpt "misc/test/language_tour/operators_test.dart (op-assign)"?>
```dart
var a = 2; // Gán bằng =
a *= 3; // Gán và nhân: a = a * 3
assert(a == 6);
```


<a id="logical-operators"></a>

## Toán tử logic

Bạn có thể đảo hoặc kết hợp các biểu thức boolean bằng các toán tử
logic.

| Toán tử                   | Ý nghĩa                                                                 |
|---------------------------|-------------------------------------------------------------------------|
| `!`*`expr`*               | đảo biểu thức theo sau (đổi false thành true và ngược lại)              |
| <code>&#124;&#124;</code> | OR logic                                                                |
| `&&`                      | AND logic                                                               |

{:.table .table-striped}

Sau đây là ví dụ sử dụng các toán tử logic:

<?code-excerpt "misc/lib/language_tour/operators.dart (op-logical)"?>
```dart
if (!done && (col == 0 || col == 3)) {
  // ...Thực hiện điều gì đó...
}
```


<a id="bitwise-and-shift-operators"></a>

## Toán tử bitwise và toán tử dịch

Bạn có thể thao tác từng bit riêng lẻ của các số trong Dart. Thông thường,
bạn sẽ sử dụng các toán tử bitwise và toán tử dịch này với số nguyên.

| Toán tử             | Ý nghĩa                                                |
|----------------------|--------------------------------------------------------|
| `&`                  | AND                                                    |
| <code>&#124;</code>  | OR                                                     |
| `^`                  | XOR                                                    |
| `~`*`expr`*          | Phép bù bitwise unary (0 thành 1; 1 thành 0)           |
| `<<`                 | Dịch trái                                              |
| `>>`                 | Dịch phải                                              |
| `>>>`                | Dịch phải không dấu                                    |

{:.table .table-striped}

:::note
Hành vi của các phép toán bitwise với toán hạng lớn hoặc âm
có thể khác nhau giữa các nền tảng.
Để tìm hiểu thêm, hãy xem
[Khác biệt giữa các nền tảng đối với phép toán bitwise][].
:::

Sau đây là ví dụ sử dụng các toán tử bitwise và toán tử dịch:

<?code-excerpt "misc/test/language_tour/operators_test.dart (op-bitwise)"?>
```dart
final value = 0x22;
final bitmask = 0x0f;

assert((value & bitmask) == 0x02); // AND
assert((value & ~bitmask) == 0x20); // AND NOT
assert((value | bitmask) == 0x2f); // OR
assert((value ^ bitmask) == 0x2d); // XOR

assert((value << 4) == 0x220); // Dịch trái
assert((value >> 4) == 0x02); // Dịch phải

// Ví dụ dịch phải cho kết quả hành vi khác trên web
// vì giá trị toán hạng thay đổi khi bị mask thành 32 bit:
assert((-value >> 4) == -0x03);

assert((value >>> 4) == 0x02); // Dịch phải không dấu
assert((-value >>> 4) > 0); // Dịch phải không dấu
```

:::version-note
Toán tử `>>>` (được gọi là _triple-shift_ hoặc _dịch không dấu_)
yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 2.14.
:::

[Khác biệt giữa các nền tảng đối với phép toán bitwise]: /resources/language/number-representation#bitwise-operations

<a id="conditional-expressions"></a>

## Biểu thức điều kiện

Dart có hai toán tử cho phép bạn đánh giá ngắn gọn các biểu thức
mà nếu không có chúng có thể cần đến câu lệnh [if-else][]:

_`condition`_ `?` _`expr1`_ `:` _`expr2`_
: Nếu _condition_ là true, đánh giá _expr1_ (và trả về giá trị của nó);
  nếu không, đánh giá và trả về giá trị của _expr2_.

_`expr1`_ `??` _`expr2`_
: Nếu _expr1_ khác null, trả về giá trị của nó;
  nếu không, đánh giá và trả về giá trị của _expr2_.

Khi cần gán một giá trị
dựa trên một biểu thức boolean,
hãy cân nhắc sử dụng toán tử điều kiện `?` và `:`.

<?code-excerpt "misc/lib/language_tour/operators.dart (if-then-else-operator)"?>
```dart
var visibility = isPublic ? 'public' : 'private';
```

Nếu biểu thức boolean kiểm tra null,
hãy cân nhắc sử dụng toán tử if-null `??`
(còn gọi là toán tử null-coalescing).

<?code-excerpt "misc/test/language_tour/operators_test.dart (if-null)"?>
```dart
String playerName(String? name) => name ?? 'Guest';
```

Ví dụ trên có thể được viết theo ít nhất hai cách khác,
nhưng không ngắn gọn bằng:

<?code-excerpt "misc/test/language_tour/operators_test.dart (if-null-alt)"?>
```dart
// Phiên bản dài hơn một chút sử dụng toán tử ?:.
String playerName(String? name) => name != null ? name : 'Guest';

// Phiên bản rất dài sử dụng câu lệnh if-else.
String playerName(String? name) {
  if (name != null) {
    return name;
  } else {
    return 'Guest';
  }
}
```

<a id="cascade-notation"></a>

## Ký pháp cascade

Cascade (`..`, `?..`) cho phép bạn thực hiện một chuỗi thao tác
trên cùng một đối tượng. Ngoài việc truy cập các member instance,
bạn cũng có thể gọi các phương thức instance trên chính đối tượng đó.
Điều này thường giúp bạn không phải tạo một biến tạm thời và
cho phép bạn viết code liền mạch hơn.

Hãy xem xét đoạn code sau:

<?code-excerpt "misc/lib/language_tour/cascades.dart (cascade)"?>
```dart
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;
```

Constructor `Paint()`
trả về một đối tượng `Paint`.
Đoạn code theo sau ký pháp cascade hoạt động
trên đối tượng này và bỏ qua mọi giá trị
có thể được trả về.

Ví dụ trên tương đương với đoạn code sau:

<?code-excerpt "misc/lib/language_tour/cascades.dart (cascade-expanded)"?>
```dart
var paint = Paint();
paint.color = Colors.black;
paint.strokeCap = StrokeCap.round;
paint.strokeWidth = 5.0;
```

Nếu đối tượng mà cascade thao tác trên đó có thể là null,
hãy sử dụng cascade _ngắt mạch khi null_ (`?..`) cho thao tác đầu tiên.
Bắt đầu bằng `?..` bảo đảm không thao tác cascade nào
được thử thực hiện trên đối tượng null đó.

<?code-excerpt "misc/test/language_tour/browser_test.dart (cascade-operator)"?>
```dart
document.querySelector('#confirm') // Lấy một đối tượng.
  ?..textContent =
      'Confirm' // Sử dụng các member của đối tượng.
  ..classList.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'))
  ..scrollIntoView();
```

Đoạn code trên tương đương với đoạn sau:

<?code-excerpt "misc/test/language_tour/browser_test.dart (cascade-operator-example-expanded)"?>
```dart
final button = document.querySelector('#confirm');
button?.textContent = 'Confirm';
button?.classList.add('important');
button?.onClick.listen((e) => window.alert('Confirmed!'));
button?.scrollIntoView();
```

Bạn cũng có thể lồng cascade. Ví dụ:

<?code-excerpt "misc/lib/language_tour/operators.dart (nested-cascades)"?>
```dart
final addressBook =
    (AddressBookBuilder()
          ..name = 'jenny'
          ..email = 'jenny@example.com'
          ..phone =
              (PhoneNumberBuilder()
                    ..number = '415-555-0100'
                    ..label = 'home')
                  .build())
        .build();
```

Hãy cẩn thận tạo cascade trên một hàm trả về
một đối tượng thực sự. Ví dụ, đoạn code sau thất bại:

<?code-excerpt "misc/lib/language_tour/operators.dart (cannot-cascade-on-void)" plaster="none"?>
```dart
var sb = StringBuffer();
sb.write('foo')
  ..write('bar'); // Lỗi: phương thức 'write' không được định nghĩa cho 'void'.
```

Lời gọi `sb.write()` trả về void,
và bạn không thể tạo cascade trên `void`.

:::note
Nói một cách chính xác, ký pháp "hai dấu chấm" dành cho cascade không phải là một toán tử.
Nó chỉ là một phần trong cú pháp Dart.
:::

<a id="spread-operators"></a>

## Toán tử spread

Toán tử spread đánh giá một biểu thức tạo ra một collection,
unpack các giá trị kết quả và chèn chúng vào một collection khác.

**Toán tử spread thực ra không phải là một biểu thức toán tử**.
Cú pháp `...`/`...?` là một phần của chính collection literal.
Vì vậy, bạn có thể tìm hiểu thêm về toán tử spread trên trang
[Collection](/language/collections#spread-operators).

Vì không phải là toán tử nên cú pháp này không có
bất kỳ "[độ ưu tiên của toán tử](#operators)" nào.
Trên thực tế, nó có "độ ưu tiên" thấp nhất &mdash;
bất kỳ loại biểu thức nào cũng hợp lệ làm target của spread, chẳng hạn như:

```dart
[...a + b]
```

<a id="other-operators"></a>

## Các toán tử khác

Bạn đã thấy hầu hết những toán tử còn lại trong các ví dụ khác:

| Toán tử | Tên                                | Ý nghĩa                                                                                                                                                                                                                                                                                  |
|---------|------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `()`    | Áp dụng hàm                        | Biểu diễn một lời gọi hàm                                                                                                                                                                                                                                                                |
| `[]`    | Truy cập subscript                 | Biểu diễn một lời gọi đến toán tử `[]` có thể override; ví dụ: `fooList[1]` truyền số nguyên `1` cho `fooList` để truy cập phần tử tại index `1`                                                                                                                                            |
| `?[]`   | Truy cập subscript có điều kiện    | Giống `[]`, nhưng toán hạng ngoài cùng bên trái có thể là null; ví dụ: `fooList?[1]` truyền số nguyên `1` cho `fooList` để truy cập phần tử tại index `1`, trừ khi `fooList` là null (trong trường hợp đó, biểu thức được đánh giá thành null)                                               |
| `.`     | Truy cập member                    | Tham chiếu đến một thuộc tính của biểu thức; ví dụ: `foo.bar` chọn thuộc tính `bar` từ biểu thức `foo`                                                                                                                                                                                     |
| `?.`    | Truy cập member có điều kiện       | Giống `.`, nhưng toán hạng ngoài cùng bên trái có thể là null; ví dụ: `foo?.bar` chọn thuộc tính `bar` từ biểu thức `foo`, trừ khi `foo` là null (trong trường hợp đó, giá trị của `foo?.bar` là null)                                                                                     |
| `!`     | Toán tử khẳng định not-null        | Ép một biểu thức thành kiểu non-nullable cơ sở của nó và ném exception runtime nếu phép ép thất bại; ví dụ: `foo!.bar` khẳng định `foo` khác null và chọn thuộc tính `bar`, trừ khi `foo` là null (trong trường hợp đó, một exception runtime được ném)                                      |

{:.table .table-striped}

Để biết thêm thông tin về các toán tử `.`, `?.` và `..`, hãy xem
[Lớp][].


[toán tử dưới dạng member của lớp]: /language/methods#operators
[Đặc tả ngôn ngữ Dart]: /resources/language/spec
[identical()]: {{site.dart-api}}/dart-core/identical.html
[Toán tử]: /language/methods#operators
[tiền tố library]: /language/libraries#specifying-a-library-prefix
[if-else]: /language/branches#if
[phiên bản ngôn ngữ]: /language/versioning
[Lớp]: /language/classes
