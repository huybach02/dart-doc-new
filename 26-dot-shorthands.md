---
title: Dot shorthand
description: Tìm hiểu về cú pháp dot shorthand trong Dart.
prevpage:
  url: /language/enums
  title: Enum
nextpage:
  url: /language/extension-methods
  title: Extension method
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>

:::version-note
Dot shorthand yêu cầu [language version][] tối thiểu là 3.10.
:::

[language version]: /language/versioning

<a id="overview"></a>
## Tổng quan

Cú pháp dot shorthand `.foo` cho phép bạn viết code Dart ngắn gọn hơn
bằng cách bỏ kiểu khi compiler có thể suy luận kiểu đó từ context.
Nó cung cấp một lựa chọn gọn gàng thay cho việc viết đầy đủ
`ContextType.foo` khi truy cập giá trị enum,
member static hoặc constructor.

Về bản chất, dot shorthand cho phép một biểu thức
bắt đầu bằng một trong những dạng sau, rồi tùy chọn nối tiếp
các thao tác khác vào biểu thức đó:

* Identifier `.myValue`

* Constructor `.new()`

* Tạo hằng số `const .myValue()`

Sau đây là ví dụ nhanh về cách nó đơn giản hóa phép gán enum:

<?code-excerpt "language/lib/shorthands/intro.dart"?>
```dart
// Dùng cú pháp dot shorthand trên enum:
enum Status { none, running, stopped, paused }

Status currentStatus = .running; // Thay cho Status.running

// Dùng cú pháp dot shorthand trên method static:
int port = .parse('8080'); // Thay cho int.parse('8080')

// Dùng cú pháp dot shorthand trên constructor:
class Point {
  final int x, y;
  Point(this.x, this.y);
  Point.origin() : x = 0, y = 0;
}

Point origin = .origin(); // Thay cho Point.origin()
```

<a id="the-role-of-context-type"></a>
## Vai trò của context type

Dot shorthand dùng [context type][] để xác định member
mà compiler sẽ phân giải đến. Context type là kiểu mà Dart
mong đợi một biểu thức có được dựa trên vị trí của biểu thức đó.
Ví dụ, trong `Status currentStatus = .running`,
compiler biết một `Status` được mong đợi nên suy luận
`.running` có nghĩa là `Status.running`.

[context type]: /resources/glossary#context-type

<a id="lexical-structure-and-syntax"></a>
## Cấu trúc lexical và cú pháp

_Static member shorthand_ là một biểu thức
bắt đầu bằng dấu chấm (`.`).
Khi kiểu đã được biết từ context xung quanh,
cú pháp này cung cấp một cách ngắn gọn để truy cập
member static, constructor và giá trị enum.

<a id="enums"></a>
### Enum

Một use case chính và rất được khuyến nghị cho dot shorthand
là với enum, đặc biệt trong phép gán và statement switch,
nơi kiểu enum rất rõ ràng.

<?code-excerpt "language/lib/shorthands/enums.dart"?>
```dart
enum LogLevel { debug, info, warning, error }

/// Trả về mã màu cần dùng cho [level] log được chỉ định.
String colorCode(LogLevel level) {
  // Dùng cú pháp dot shorthand cho giá trị enum trong các case switch:
  return switch (level) {
    .debug => 'gray', // Thay cho LogLevel.debug
    .info => 'blue', // Thay cho LogLevel.info
    .warning => 'orange', // Thay cho LogLevel.warning
    .error => 'red', // Thay cho LogLevel.error
  };
}

// Ví dụ sử dụng:
String warnColor = colorCode(.warning); // Trả về 'orange'
```

<a id="named-constructors"></a>
### Named constructor

Dot shorthand hữu ích khi gọi named constructor
hoặc factory constructor. Cú pháp này cũng hoạt động khi cung cấp
type argument cho constructor của generic class.

<?code-excerpt "language/lib/shorthands/constructors.dart"?>
```dart
class Point {
  final double x, y;
  const Point(this.x, this.y);
  const Point.origin() : x = 0, y = 0; // Named constructor

  // Factory constructor
  factory Point.fromList(List<double> list) {
    return Point(list[0], list[1]);
  }
}

// Dùng cú pháp dot shorthand trên named constructor:
Point origin = .origin(); // Thay cho Point.origin()

// Dùng cú pháp dot shorthand trên factory constructor:
Point p1 = .fromList([1.0, 2.0]); // Thay cho Point.fromList([1.0, 2.0])

// Dùng cú pháp dot shorthand trên constructor của generic class:
List<int> intList = .filled(5, 0); // Thay cho List.filled(5, 0)
```

<a id="unnamed-constructors"></a>
### Unnamed constructor

Dot shorthand `.new` cung cấp một cách ngắn gọn để gọi
unnamed constructor của một lớp. Cách này hữu ích
để gán field hoặc biến khi kiểu đã được
khai báo tường minh.

Cú pháp này đặc biệt hiệu quả trong việc làm gọn
initializer lặp lại của field trong lớp.
Như ví dụ “sau” sử dụng dot shorthand dưới đây cho thấy,
nó có thể được dùng cho constructor cả có và không có argument.
Nó cũng suy luận mọi generic type argument
từ context.

**Không dùng dot shorthand:**

<?code-excerpt "language/lib/shorthands/unnamed_constructors.dart (unnamed-before)"?>
```dart
class _PageState extends State<Page> {
  late final AnimationController _animationController = AnimationController(
    vsync: this,
  );
  final ScrollController _scrollController = ScrollController();

  final GlobalKey<ScaffoldMessengerState> scaffoldKey =
      GlobalKey<ScaffoldMessengerState>();

  Map<String, Map<String, bool>> properties = <String, Map<String, bool>>{};
  // ...
}
```

**Dùng dot shorthand:**

<?code-excerpt "language/lib/shorthands/unnamed_constructors.dart (unnamed-after)" replace="/_PageStateAfter/_PageState/g;"?>
```dart
// Dùng cú pháp dot shorthand để gọi unnamed constructor:
class _PageState extends State<Page> {
  late final AnimationController _animationController = .new(vsync: this);
  final ScrollController _scrollController = .new();
  final GlobalKey<ScaffoldMessengerState> scaffoldKey = .new();
  Map<String, Map<String, bool>> properties = .new();
  // ...
}
```

<a id="static-members"></a>
### Member static

Bạn có thể dùng cú pháp dot shorthand để gọi method static hoặc
truy cập field/getter static. Compiler suy luận
lớp đích từ context type của biểu thức.

<?code-excerpt "language/lib/shorthands/static_members.dart"?>
```dart
// Dùng cú pháp dot shorthand để gọi method static:
int httpPort = .parse('80'); // Thay cho int.parse('80')

// Dùng cú pháp dot shorthand để truy cập field hoặc getter static:
BigInt bigIntZero = .zero; // Thay cho BigInt.zero
```

<a id="constant-expressions"></a>
### Biểu thức hằng

Bạn có thể dùng dot shorthand trong constant context
nếu member đang được truy cập là hằng số tại compile time.
Điều này thường gặp với giá trị enum và khi gọi constructor `const`.

<?code-excerpt "language/lib/shorthands/consts.dart"?>
```dart
enum Status { none, running, stopped, paused }

class Point {
  final double x, y;
  const Point(this.x, this.y);
  const Point.origin() : x = 0.0, y = 0.0;
}

// Dùng cú pháp dot shorthand cho giá trị enum:
const Status defaultStatus = .running; // Thay cho Status.running

// Dùng cú pháp dot shorthand để gọi const named constructor:
const Point myOrigin = .origin(); // Thay cho Point.origin()

// Dùng cú pháp dot shorthand trong const collection literal:
const List<Point> keyPoints = [.origin(), .new(1.0, 1.0)];
// Thay cho [Point.origin(), Point(1.0, 1.0)]
```

<a id="rules-and-limitations"></a>
## Quy tắc và hạn chế

Dot shorthand dựa vào một context type rõ ràng, dẫn đến
một số quy tắc và hạn chế cụ thể mà bạn nên biết.

<a id="clear-context-type-required-in-chains"></a>
### Cần context type rõ ràng trong chain

Mặc dù bạn có thể nối tiếp các thao tác như lời gọi method hoặc
truy cập property vào dot shorthand, toàn bộ biểu thức
được validate dựa trên context type.

Trước tiên compiler dùng context để xác định dot shorthand
được phân giải thành gì. Mọi thao tác tiếp theo trong
chain phải trả về một giá trị khớp với cùng context type ban đầu đó.

<?code-excerpt "language/lib/shorthands/chain.dart (chain)"?>
```dart
// .fromCharCode(72) phân giải thành String "H",
// sau đó instance method .toLowerCase() được gọi trên String đó.
String lowerH = .fromCharCode(72).toLowerCase();
// Thay cho String.fromCharCode(72).toLowerCase()

print(lowerH); // Output: h
```

<a id="asymmetric-equality-checks"></a>
### Phép kiểm tra bằng nhau bất đối xứng

Các toán tử `==` và `!=` có một quy tắc đặc biệt cho dot shorthand.
Khi cú pháp dot shorthand được dùng trực tiếp ở vế phải
của phép kiểm tra bằng nhau, Dart dùng static type của
vế trái để xác định lớp hoặc enum cho shorthand.

Ví dụ, trong biểu thức như `myColor == .green`,
kiểu của biến `myColor` được dùng làm context.
Điều này có nghĩa compiler diễn giải `.green` là `Color.green`.

<?code-excerpt "language/lib/shorthands/equality.dart"?>
```dart
enum Color { red, green, blue }

// Dùng cú pháp dot shorthand cho biểu thức bằng nhau:
void allowedExamples() {
  Color myColor = Color.red;
  bool condition = true;

  // HỢP LỆ: `myColor` là một `Color`, nên `.green` được suy luận là `Color.green`.
  if (myColor == .green) {
    print('The color is green.');
  }

  // HỢP LỆ: Cũng hoạt động với `!=`.
  if (myColor != .blue) {
    print('The color is not blue.');
  }

  // HỢP LỆ: Context cho toán tử ba ngôi là biến `inferredColor`
  // đang được gán, biến này có kiểu `Color`.
  Color inferredColor = condition ? .green : .blue;
  print('Inferred color is $inferredColor');
}
```

Dot shorthand phải nằm ở vế phải của toán tử `==`
hoặc `!=`. Việc so sánh với một biểu thức phức tạp hơn,
chẳng hạn biểu thức điều kiện, cũng không được phép.

<?code-excerpt "language/lib/shorthands/equality_with_errors.dart"?>
```dart tag=fails-sa
enum Color { red, green, blue }

void notAllowedExamples() {
  Color myColor = Color.red;
  bool condition = true;

  // LỖI: Shorthand phải nằm ở vế phải của `==`.
  // Toán tử `==` của Dart không đối xứng đối với tính năng này.
  if (.red == myColor) {
    print('This will not compile.');
  }

  // LỖI: Vế phải là một biểu thức phức tạp (biểu thức điều kiện),
  // đây không phải target hợp lệ cho shorthand trong phép so sánh.
  if (myColor == (condition ? .green : .blue)) {
    print('This will not compile.');
  }

  // LỖI: Type context bị mất khi cast `myColor` thành `Object`.
  // Compiler không còn biết `.green` phải tham chiếu đến `Color.green`.
  if ((myColor as Object) == .green) {
    print('This will not compile.');
  }
}
```

<a id="expression-statements-cant-start-with"></a>
### Expression statement không thể bắt đầu bằng `.`

Để tránh sự mơ hồ tiềm ẩn khi parse trong tương lai,
expression statement không được phép bắt đầu bằng
token `.`.

<?code-excerpt "language/lib/shorthands/expression_with_errors.dart"?>
```dart tag=fails-sa
class Logger {
  static void log(String message) {
    print(message);
  }
}

void main() {
  // LỖI: Expression statement không thể bắt đầu bằng `.`.
  // Compiler không có type context (như một phép gán biến)
  // để suy luận rằng `.log` phải tham chiếu đến `Logger.log`.
  .log('Hello');
}
```

<a id="limited-handling-of-union-types"></a>
### Hỗ trợ hạn chế đối với union type

Mặc dù có cách xử lý đặc biệt cho kiểu nullable (`T?`) và
`FutureOr<T>`, mức hỗ trợ vẫn có giới hạn.

*  Đối với kiểu nullable (`T?`), bạn có thể truy cập member static
   của `T`, nhưng không thể truy cập member static của `Null`.

*  Đối với `FutureOr<T>`, bạn có thể truy cập member static của `T`
  (chủ yếu để hỗ trợ giá trị trả về của function `async`), nhưng
  không thể truy cập member static của chính lớp `Future`.
