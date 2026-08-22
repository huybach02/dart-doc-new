---
title: Hàm
description: Mọi kiến thức về function trong Dart.
prevpage:
  url: /language/error-handling
  title: Xử lý lỗi
nextpage:
  url: /language/metadata
  title: Metadata
---

Dart là một ngôn ngữ hướng đối tượng thực thụ, vì vậy ngay cả function cũng là object
và có một kiểu, [Function.][Function API reference]
Điều này có nghĩa là function có thể được gán cho biến hoặc được truyền làm argument
cho các function khác. Bạn cũng có thể gọi một instance của lớp Dart như thể
nó là một function. Để biết chi tiết, xem [Object có thể gọi][Callable objects].

Sau đây là một ví dụ về cách implement một function:

<?code-excerpt "misc/lib/language_tour/functions.dart (function)"?>
```dart
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```

Mặc dù Effective Dart khuyến nghị
[type annotation cho API công khai][type annotations for public APIs],
function vẫn hoạt động nếu bạn bỏ qua các kiểu:

<?code-excerpt "misc/lib/language_tour/functions.dart (function-omitting-types)"?>
```dart
isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```

Đối với function chỉ chứa một biểu thức, bạn có thể dùng cú pháp viết tắt:

<?code-excerpt "misc/lib/language_tour/functions.dart (function-shorthand)"?>
```dart
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```

Cú pháp <code>=> <em>expr</em></code> là dạng viết tắt của
<code>{ return <em>expr</em>; }</code>. Ký hiệu `=>`
đôi khi được gọi là cú pháp _arrow_.

:::note
Chỉ _biểu thức_ mới có thể xuất hiện giữa arrow (`=>`) và dấu chấm phẩy (`;`).
Biểu thức được đánh giá thành giá trị.
Điều này có nghĩa là bạn không thể viết một statement ở nơi Dart mong đợi một giá trị.
Ví dụ,
bạn có thể dùng [biểu thức điều kiện][conditional expression] nhưng không thể dùng [câu lệnh if][if statement].
Trong ví dụ trước,
`_nobleGases[atomicNumber] != null;` trả về một giá trị boolean.
Sau đó function trả về một giá trị boolean
cho biết `atomicNumber` có nằm trong phạm vi khí hiếm hay không.
:::

<a id="parameters"></a>
## Parameter

Một function có thể có số lượng _required positional_ parameter tùy ý. Theo sau chúng
có thể là các _named_ parameter hoặc các _optional positional_ parameter
(nhưng không thể có cả hai).

:::note
Một số API—đáng chú ý là các constructor widget của [Flutter][]—chỉ dùng named
parameter, ngay cả với những parameter bắt buộc. Xem mục tiếp theo để biết
chi tiết.
:::

Bạn có thể dùng [dấu phẩy cuối][trailing commas] khi truyền argument cho một function
hoặc khi định nghĩa các parameter của function.

<a id="named-parameters"></a>
### Named parameter

Named parameter là không bắt buộc
trừ khi chúng được đánh dấu rõ ràng bằng `required`.

Khi định nghĩa một function, hãy dùng
<code>{<em>param1</em>, <em>param2</em>, …}</code>
để chỉ định các named parameter.
Nếu bạn không cung cấp giá trị mặc định
hoặc không đánh dấu một named parameter là `required`,
kiểu của chúng phải là nullable
vì giá trị mặc định của chúng sẽ là `null`:

<?code-excerpt "misc/lib/language_tour/functions.dart (specify-named-parameters)"?>
```dart
/// Đặt các flag [bold] và [hidden] ...
void enableFlags({bool? bold, bool? hidden}) {
  ...
}
```

Khi gọi một function,
bạn có thể chỉ định named argument bằng
<code><em>paramName</em>: <em>value</em></code>.
Ví dụ:

<?code-excerpt "misc/lib/language_tour/functions.dart (use-named-parameters)"?>
```dart
enableFlags(bold: true, hidden: false);
```

<a id="default-parameters"></a>
Để định nghĩa một giá trị mặc định khác `null` cho named parameter,
dùng `=` để chỉ định giá trị mặc định.
Giá trị được chỉ định phải là một hằng số tại compile time.
Ví dụ:

<?code-excerpt "misc/lib/language_tour/functions.dart (named-parameter-default-values)"?>
```dart
/// Đặt các flag [bold] và [hidden] ...
void enableFlags({bool bold = false, bool hidden = false}) {
  ...
}

// bold sẽ là true; hidden sẽ là false.
enableFlags(bold: true);
```

Nếu thay vào đó bạn muốn một named parameter là bắt buộc,
yêu cầu bên gọi phải cung cấp giá trị cho parameter,
hãy annotate chúng bằng `required`:

<?code-excerpt "misc/lib/language_tour/functions.dart (required-named-parameters)" replace="/required/[!$&!]/g"?>
```dart
const Scrollbar({super.key, [!required!] Widget child});
```

Nếu ai đó cố tạo một `Scrollbar`
mà không chỉ định argument `child`,
analyzer sẽ báo một vấn đề.

:::note
Một parameter được đánh dấu `required` vẫn có thể là nullable:

<?code-excerpt "misc/lib/language_tour/functions.dart (required-named-parameters-nullable)" replace="/Widget\?/[!$&!]/g; /ScrollbarTwo/Scrollbar/g;"?>
```dart
const Scrollbar({super.key, required [!Widget?!] child});
```

:::

Bạn có thể muốn đặt positional argument trước,
nhưng Dart không yêu cầu điều đó.
Dart cho phép đặt named argument ở bất kỳ đâu trong
danh sách argument khi điều này phù hợp với API của bạn:

<?code-excerpt "misc/lib/language_tour/functions.dart (named-arguments-anywhere)"?>
```dart
repeat(times: 2, () {
  ...
});
```

<a id="optional-positional-parameters"></a>
### Optional positional parameter

Việc đặt một tập các parameter của function trong `[]`
sẽ đánh dấu chúng là optional positional parameter.
Nếu bạn không cung cấp giá trị mặc định,
kiểu của chúng phải là nullable
vì giá trị mặc định của chúng sẽ là `null`:

<?code-excerpt "misc/test/language_tour/functions_test.dart (optional-positional-parameters)"?>
```dart
String say(String from, String msg, [String? device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```

Sau đây là ví dụ gọi function này
mà không có parameter không bắt buộc:

<?code-excerpt "misc/test/language_tour/functions_test.dart (call-without-optional-param)"?>
```dart
assert(say('Bob', 'Howdy') == 'Bob says Howdy');
```

Và sau đây là ví dụ gọi function này với parameter thứ ba:

<?code-excerpt "misc/test/language_tour/functions_test.dart (call-with-optional-param)"?>
```dart
assert(
  say('Bob', 'Howdy', 'smoke signal') ==
      'Bob says Howdy with a smoke signal',
);
```

Để định nghĩa một giá trị mặc định khác `null` cho optional positional parameter,
dùng `=` để chỉ định giá trị mặc định.
Giá trị được chỉ định phải là một hằng số tại compile time.
Ví dụ:

<?code-excerpt "misc/test/language_tour/functions_test.dart (optional-positional-param-default)"?>
```dart
String say(String from, String msg, [String device = 'carrier pigeon']) {
  var result = '$from says $msg with a $device';
  return result;
}

assert(say('Bob', 'Howdy') == 'Bob says Howdy with a carrier pigeon');
```

<a id="the-main-function" aria-hidden="true"></a>

<a id="parameter-modifiers"></a>
### Modifier của parameter

Trong Dart 3.13 trở lên,
bạn không thể dùng các modifier như `final` hoặc `var` cho parameter function thông thường.
Các keyword này giờ được dành riêng cho
[primary constructor][] để khai báo instance field.

Nếu muốn thực thi tính bất biến cho parameter của function,
hãy dùng các lint như [`parameter_assignments`][] thay cho
keyword `final` trong signature.

## Function main() {:#main}

Mọi ứng dụng phải có một function `main()` ở top level, đóng vai trò là
entrypoint của ứng dụng. Function `main()` trả về `void` và có một
parameter `List<String>` không bắt buộc dành cho các argument.

Sau đây là một function `main()` đơn giản:

<?code-excerpt "misc/test/samples_test.dart (hello-world)"?>
```dart
void main() {
  print('Hello, World!');
}
```

Sau đây là ví dụ về function `main()` cho một ứng dụng dòng lệnh
nhận các argument:

<?code-excerpt "misc/test/language_tour/functions_test.dart (main-args)"?>
```dart title="args.dart"
// Chạy ứng dụng như sau: dart run args.dart 1 test
void main(List<String> arguments) {
  print(arguments);

  assert(arguments.length == 2);
  assert(int.parse(arguments[0]) == 1);
  assert(arguments[1] == 'test');
}
```

Bạn có thể dùng [library args]({{site.pub-pkg}}/args) để
định nghĩa và parse các argument dòng lệnh.

<a id="functions-as-first-class-objects"></a>
## Function dưới dạng first-class object

Bạn có thể truyền một function làm parameter cho một function khác. Ví dụ:

<?code-excerpt "misc/lib/language_tour/functions.dart (function-as-param)"?>
```dart
void printElement(int element) {
  print(element);
}

var list = [1, 2, 3];

// Truyền printElement làm parameter.
list.forEach(printElement);
```

Bạn cũng có thể gán một function cho một biến, chẳng hạn như:

<?code-excerpt "misc/test/language_tour/functions_test.dart (function-as-var)"?>
```dart
var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
assert(loudify('hello') == '!!! HELLO !!!');
```

Ví dụ này dùng một anonymous function.
Mục tiếp theo trình bày thêm về chúng.

<a id="function-types"></a>
## Kiểu function

Bạn có thể chỉ định kiểu của một function, được gọi là _function type_.
Function type được lấy từ phần header của một khai báo function bằng cách
thay tên function bằng keyword `Function`.
Ngoài ra, bạn được phép bỏ tên của positional parameter, nhưng
không thể bỏ tên của named parameter. Ví dụ:

<?code-excerpt "misc/lib/language_tour/functions.dart (function-type)"?>
```dart
void greet(String name, {String greeting = 'Hello'}) =>
    print('$greeting $name!');

// Lưu `greet` vào một biến và gọi nó.
void Function(String, {String greeting}) g = greet;
g('Dash', greeting: 'Howdy');
```

:::note
Trong Dart, function là first-class object,
nghĩa là chúng có thể được gán cho biến,
được truyền làm argument và được trả về từ các function khác.

Bạn có thể dùng khai báo [`typedef`][] để đặt tên rõ ràng cho function type,
điều này có thể hữu ích cho tính rõ ràng và khả năng tái sử dụng.
:::

[`typedef`]: /language/typedefs

<a id="anonymous-functions"></a>
## Anonymous function

Mặc dù bạn đặt tên cho hầu hết function, chẳng hạn như `main()` hoặc `printElement()`,
bạn cũng có thể tạo function không có tên.
Các function này được gọi là _anonymous function_, _lambda_ hoặc _closure_.

Một anonymous function tương tự một named function vì nó có:

- Không hoặc nhiều parameter, phân tách bằng dấu phẩy
- Type annotation không bắt buộc nằm giữa các dấu ngoặc đơn.

Khối code sau chứa phần thân của function:

```dart
([[Type] param1[, ...]]) {
  codeBlock;
}
```

Ví dụ sau định nghĩa một anonymous function
với parameter không có kiểu là `item`.
Anonymous function truyền nó cho function `map`.
Function `map`, được gọi cho từng item trong list,
chuyển mỗi string thành chữ hoa.
Sau đó, anonymous function được truyền cho `forEach`
in từng string đã chuyển đổi cùng với độ dài của nó.

<?code-excerpt "misc/test/language_tour/functions_test.dart (anonymous-function)"?>
```dart
const list = ['apples', 'bananas', 'oranges'];

var uppercaseList = list.map((item) {
  return item.toUpperCase();
}).toList();
// Chuyển thành list sau khi ánh xạ

for (var item in uppercaseList) {
  print('$item: ${item.length}');
}
```

Nhấp vào **Run** để thực thi code.

<?code-excerpt "misc/test/language_tour/functions_test.dart (anonymous-function-main)"?>
```dartpad
void main() {
  const list = ['apples', 'bananas', 'oranges'];

  var uppercaseList = list.map((item) {
    return item.toUpperCase();
  }).toList();
  // Chuyển thành list sau khi ánh xạ

  for (var item in uppercaseList) {
    print('$item: ${item.length}');
  }
}
```

Nếu function chỉ chứa một biểu thức hoặc statement return duy nhất,
bạn có thể rút gọn nó bằng ký hiệu arrow.
Dán dòng sau vào DartPad rồi nhấp **Run**
để xác minh rằng nó tương đương về chức năng.

<?code-excerpt "misc/test/language_tour/functions_test.dart (anon-func)"?>
```dart
var uppercaseList = list.map((item) => item.toUpperCase()).toList();
uppercaseList.forEach((item) => print('$item: ${item.length}'));
```

<a id="lexical-scope"></a>
## Lexical scope

Dart xác định scope của biến dựa trên bố cục code.
Một ngôn ngữ lập trình có tính năng này được gọi là ngôn ngữ có lexical scope.
Bạn có thể “lần theo các dấu ngoặc nhọn ra phía ngoài” để xem một biến có nằm trong scope hay không.

**Ví dụ:** Một chuỗi function lồng nhau với các biến ở mỗi cấp scope:

<?code-excerpt "misc/test/language_tour/functions_test.dart (nested-functions)"?>
```dart
bool topLevel = true;

void main() {
  var insideMain = true;

  void myFunction() {
    var insideFunction = true;

    void nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

Method `nestedFunction()` có thể dùng biến từ mọi cấp,
cho đến tận top level.

<a id="lexical-closures"></a>
## Lexical closure

Một object function có thể truy cập các biến trong lexical scope của nó
khi function nằm ngoài scope đó được gọi là _closure_.

Function có thể đóng trên các biến được định nghĩa trong những scope bao quanh. Trong
ví dụ sau, `makeAdder()` capture biến `addBy`. Dù function
được trả về đi đến đâu, nó vẫn ghi nhớ `addBy`.

<?code-excerpt "misc/test/language_tour/functions_test.dart (function-closure)"?>
```dart
/// Trả về một function cộng [addBy] vào
/// argument của function.
Function makeAdder(int addBy) {
  return (int i) => addBy + i;
}

void main() {
  // Tạo một function cộng thêm 2.
  var add2 = makeAdder(2);

  // Tạo một function cộng thêm 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

<a id="tear-offs"></a>
## Tear-off

Khi bạn tham chiếu đến một function, method hoặc named constructor mà không có dấu ngoặc đơn,
Dart tạo ra một _tear-off_. Đây là một closure nhận cùng
các parameter như function và gọi function nền khi bạn gọi nó.
Nếu code của bạn cần một closure gọi một named function với cùng
các parameter mà closure chấp nhận, đừng bọc lời gọi trong lambda.
Hãy dùng tear-off.

<?code-excerpt "misc/lib/language_tour/tear_offs.dart (variables)" ?>
```dart
var charCodes = [68, 97, 114, 116];
var buffer = StringBuffer();
```

<?code-excerpt "misc/lib/language_tour/tear_offs.dart (good-example)" ?>
```dart tag=good
// Function tear-off
charCodes.forEach(print);

// Method tear-off
charCodes.forEach(buffer.write);
```

<?code-excerpt "misc/lib/language_tour/tear_offs.dart (bad-example)" ?>
```dart tag=bad
// Function lambda
charCodes.forEach((code) {
  print(code);
});

// Method lambda
charCodes.forEach((code) {
  buffer.write(code);
});
```

<a id="testing-functions-for-equality"></a>
## Kiểm tra tính bằng nhau của function

Sau đây là ví dụ kiểm tra tính bằng nhau của top-level function, method static và
instance method:

<?code-excerpt "misc/lib/language_tour/function_equality.dart"?>
```dart
void foo() {} // Một top-level function

class A {
  static void bar() {} // Một method static
  void baz() {} // Một instance method
}

void main() {
  Function x;

  // So sánh các top-level function.
  x = foo;
  assert(foo == x);

  // So sánh các method static.
  x = A.bar;
  assert(A.bar == x);

  // So sánh các instance method.
  var v = A(); // Instance #1 của A
  var w = A(); // Instance #2 của A
  var y = w;
  x = w.baz;

  // Các closure này tham chiếu cùng một instance (#2),
  // nên chúng bằng nhau.
  assert(y.baz == x);

  // Các closure này tham chiếu đến các instance khác nhau,
  // nên chúng không bằng nhau.
  assert(v.baz != w.baz);
}
```

<a id="return-values"></a>
## Giá trị trả về

Mọi function đều trả về một giá trị. Nếu không chỉ định giá trị trả về,
statement `return null;` được ngầm thêm vào phần thân function.

<?code-excerpt "misc/test/language_tour/functions_test.dart (implicit-return-null)"?>
```dart
foo() {}

assert(foo() == null);
```

Để trả về nhiều giá trị trong một function, hãy tổng hợp các giá trị trong một [record][].

```dart
(String, int) foo() {
  return ('something', 42);
}
```

[record]: /language/records#multiple-returns

<a id="getters-and-setters"></a>
## Getter và setter

Mọi lần truy cập property (top-level, static hoặc instance) đều là một lời gọi
getter hoặc setter. Một biến ngầm tạo ra getter
và nếu nó mutable thì tạo thêm setter. Đây là lý do khi truy cập một property,
thực chất bạn đang gọi một function nhỏ ở phía sau. Việc đọc một
property gọi function getter, còn ghi vào đó gọi function setter,
ngay cả trong trường hợp property được khai báo là một biến.

Tuy nhiên, bạn cũng có thể khai báo rõ ràng getter và setter bằng
các keyword tương ứng là `get` và `set`.
Điều này cho phép tính toán giá trị của property khi nó được đọc hoặc ghi.

Mục đích của việc dùng getter và setter là tạo ra sự
tách biệt rõ ràng giữa client (code dùng property) và
provider (lớp hoặc library định nghĩa property đó). Client yêu cầu hoặc
đặt một giá trị mà không cần biết giá trị đó được lưu trong một
biến đơn giản hay được tính ngay tại thời điểm đó. Điều này trao cho provider
quyền tự do thay đổi cách property hoạt động.

Ví dụ, vì giá trị của property có thể không được lưu ở bất kỳ đâu,
nó có thể được tính lại mỗi khi getter được gọi. Một ví dụ khác
là khi một giá trị được lưu trong biến private và quyền truy cập public
chỉ được cho phép bằng cách gọi getter hoặc setter.

Ví dụ sau minh họa điều này,
trong đó getter và setter `secret` cung cấp
quyền truy cập gián tiếp đến biến private `_secret`, đồng thời
thực hiện các thao tác riêng trên giá trị được gán và truy xuất.

<?code-excerpt "language/lib/functions/getters_setters.dart"?>
```dart highlightLines=3,7-10,14-20,24,33
// Định nghĩa biến `_secret` là private đối với library vì
// identifier của nó bắt đầu bằng dấu gạch dưới (`_`).
String _secret = 'Hello';

// Một top-level getter public
// cung cấp quyền đọc [_secret].
String get secret {
  print('Getter was used!');
  return _secret.toUpperCase();
}

// Một top-level setter public
// cung cấp quyền ghi vào [_secret].
set secret(String newMessage) {
  print('Setter was used!');
  if (newMessage.isNotEmpty) {
    _secret = newMessage;
    print('New secret: "$newMessage"');
  }
}

void main() {
  // Việc đọc giá trị sẽ gọi getter.
  print('Current message: $secret');

  /*
  Output:
  Getter was used!
  Current message: HELLO
  */

  // Việc gán giá trị sẽ gọi setter.
  secret = 'Dart is fun';

  // Đọc lại sẽ gọi getter để hiển thị giá trị mới được tính toán
  print('New message: $secret');

  /*
  Output:
  Setter was used! New secret: "Dart is fun"
  Getter was used!
  New message: DART IS FUN
  */
}
```

<a id="generators"></a>
## Generator

Khi cần tạo một chuỗi giá trị theo cách lazy,
hãy cân nhắc dùng một _generator function_.
Dart có hỗ trợ tích hợp cho hai loại generator function:

- Generator **đồng bộ**: Trả về một object [`Iterable`].
- Generator **bất đồng bộ**: Trả về một object [`Stream`].

Để implement một generator function **đồng bộ**,
đánh dấu phần thân function là `sync*`,
và dùng các statement `yield` để cung cấp giá trị:

<?code-excerpt "misc/test/language_tour/async_test.dart (sync-generator)"?>
```dart
Iterable<int> naturalsTo(int n) sync* {
  int k = 0;
  while (k < n) yield k++;
}
```

Để implement một generator function **bất đồng bộ**,
đánh dấu phần thân function là `async*`,
và dùng các statement `yield` để cung cấp giá trị:

<?code-excerpt "misc/test/language_tour/async_test.dart (async-generator)"?>
```dart
Stream<int> asynchronousNaturalsTo(int n) async* {
  int k = 0;
  while (k < n) yield k++;
}
```

Nếu generator của bạn là đệ quy,
bạn có thể cải thiện hiệu năng bằng cách dùng `yield*`:

<?code-excerpt "misc/test/language_tour/async_test.dart (recursive-generator)"?>
```dart
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    yield* naturalsDownFrom(n - 1);
  }
}
```

[`Iterable`]: {{site.dart-api}}/dart-core/Iterable-class.html
[`Stream`]: {{site.dart-api}}/dart-async/Stream-class.html

## External function {:#external}

External function là function có phần thân được implement tách biệt khỏi phần
khai báo. Thêm keyword `external` trước một khai báo function như sau:

```dart
external void someFunc(int i);
```

Implementation của external function có thể đến từ một library Dart khác,
hoặc phổ biến hơn là từ một ngôn ngữ khác. Trong ngữ cảnh interop, `external`
đưa vào thông tin kiểu cho các function hoặc giá trị ngoại lai,
giúp chúng có thể được sử dụng trong Dart. Implementation và cách sử dụng
phụ thuộc rất nhiều vào nền tảng, vì vậy hãy xem tài liệu interop, chẳng hạn về
[C][] hoặc [JavaScript][], để tìm hiểu thêm.

External function có thể là top-level function, [instance method][],
getter hoặc setter, hay [constructor không redirect][non-redirecting constructors].
Một [instance variable][] cũng có thể là `external`,
điều này tương đương với một external getter và (nếu biến
không phải `final`) một external setter.

[instance methods]: /language/methods#instance-methods
[non-redirecting constructors]: /language/constructors#redirecting-constructors
[instance variable]: /language/classes#instance-variables
[C]: /interop/c-interop
[JavaScript]: /interop/js-interop
[Function API reference]: {{site.dart-api}}/dart-core/Function-class.html
[Callable objects]: /language/callable-objects
[type annotations for public APIs]: /effective-dart/design#do-type-annotate-fields-and-top-level-variables-if-the-type-isnt-obvious
[if statement]: /language/branches#if
[conditional expression]: /language/operators#conditional-expressions
[Flutter]: {{site.flutter}}
[trailing commas]: /language/collections#lists
[primary constructors]: /language/primary-constructors
[`parameter_assignments`]: /tools/linter-rules/parameter_assignments

