---
title: Introduction to Dart
shortTitle: Kiến thức cơ bản về Dart
breadcrumb: Ngôn ngữ
description: Giới thiệu ngắn gọn về các chương trình Dart và những khái niệm quan trọng.
nextpage:
  url: /language/variables
  title: Biến
---

Trang này giới thiệu ngắn gọn về ngôn ngữ Dart
thông qua các ví dụ về những tính năng chính của ngôn ngữ. 

Để tìm hiểu thêm về ngôn ngữ Dart,
hãy truy cập các trang chuyên đề chuyên sâu riêng lẻ
được liệt kê trong mục **Ngôn ngữ** ở menu bên trái.

Để tìm hiểu về các thư viện cốt lõi của Dart,
hãy xem [tài liệu về thư viện cốt lõi](/libraries).
Bạn cũng có thể xem [Dart cheatsheet](/resources/dart-cheatsheet)
để có phần giới thiệu mang tính tương tác hơn.


<a id="hello-world"></a>

## Hello World

Mọi ứng dụng đều cần hàm top-level `main()`, nơi quá trình thực thi bắt đầu.
Các hàm không trả về một giá trị một cách tường minh có kiểu trả về `void`.
Để hiển thị văn bản trên console, bạn có thể sử dụng hàm top-level `print()`:

<?code-excerpt "misc/test/samples_test.dart (hello-world)"?>
```dart
void main() {
  print('Hello, World!');
}
```

Đọc thêm về [hàm `main()`][] trong Dart,
bao gồm các tham số tùy chọn dành cho đối số dòng lệnh.

[hàm `main()`]: /language/functions#the-main-function

<a id="variables"></a>

## Biến

Ngay cả trong code Dart [type-safe](/language/type-system),
bạn vẫn có thể khai báo hầu hết các biến bằng `var` mà không cần chỉ định tường minh kiểu của chúng.
Nhờ suy luận kiểu, kiểu của các biến này được xác định bởi giá trị ban đầu:


<?code-excerpt "misc/test/samples_test.dart (var)"?>
```dart
var name = 'Voyager I';
var year = 1977;
var antennaDiameter = 3.7;
var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune'];
var image = {
  'tags': ['saturn'],
  'url': '//path/to/saturn.jpg',
};
```

[Đọc thêm](/language/variables) về các biến trong Dart,
bao gồm giá trị mặc định, các keyword `final` và `const`, cũng như kiểu tĩnh.


<a id="control-flow-statements"></a>

## Các câu lệnh control flow

Dart hỗ trợ các câu lệnh control flow thông thường:

<?code-excerpt "misc/test/samples_test.dart (control-flow)"?>
```dart
if (year >= 2001) {
  print('21st century');
} else if (year >= 1901) {
  print('20th century');
}

for (final object in flybyObjects) {
  print(object);
}

for (int month = 1; month <= 12; month++) {
  print(month);
}

while (year < 2016) {
  year += 1;
}
```

Đọc thêm về các câu lệnh control flow trong Dart,
bao gồm [`break` và `continue`](/language/loops),
[`switch` và `case`](/language/branches),
cũng như [`assert`](/language/error-handling#assert).


<a id="functions"></a>

## Hàm

[Chúng tôi khuyến nghị](/effective-dart/design#types)
chỉ định kiểu cho từng đối số và giá trị trả về của hàm:

<?code-excerpt "misc/test/samples_test.dart (functions)"?>
```dart
int fibonacci(int n) {
  if (n == 0 || n == 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

var result = fibonacci(20);
```

Cú pháp viết tắt `=>` (_mũi tên_) rất tiện dụng cho các hàm
chứa một câu lệnh duy nhất.
Cú pháp này đặc biệt hữu ích khi truyền các hàm ẩn danh làm đối số:

<?code-excerpt "misc/test/samples_test.dart (arrow)"?>
```dart
flybyObjects.where((name) => name.contains('turn')).forEach(print);
```

Ngoài việc minh họa một hàm ẩn danh (đối số của `where()`),
đoạn code này còn cho thấy bạn có thể dùng một hàm làm đối số:
hàm top-level `print()` là một đối số của `forEach()`.

[Đọc thêm](/language/functions) về các hàm trong Dart,
bao gồm tham số tùy chọn, giá trị tham số mặc định và lexical scope.


<a id="comments"></a>

## Chú thích

Chú thích trong Dart thường bắt đầu bằng `//`.

```dart
// Đây là một chú thích một dòng thông thường.

/// Đây là một documentation comment, dùng để tạo tài liệu cho library,
/// class và các member của chúng. Những công cụ như IDE và dartdoc xử lý
/// doc comment theo cách đặc biệt.

/* Các chú thích như thế này cũng được hỗ trợ. */
```

[Đọc thêm](/language/comments) về chú thích trong Dart,
bao gồm cách hoạt động của công cụ tạo tài liệu.


<a id="imports"></a>

## Import

Để truy cập các API được định nghĩa trong những library khác, hãy sử dụng `import`.

<?code-excerpt "misc/test/samples_test.dart (import)" plaster="none"?>
```dart
// Import các thư viện cốt lõi
import 'dart:math';

// Import các thư viện từ package bên ngoài
import 'package:test/test.dart';

// Import file
import 'path/to/my_other_file.dart';
```

[Đọc thêm](/language/libraries)
về library và khả năng hiển thị trong Dart,
bao gồm tiền tố library, `show` và `hide`,
cũng như lazy loading thông qua keyword `deferred`.


<a id="classes"></a>

## Lớp

Sau đây là ví dụ về một lớp có ba thuộc tính, hai constructor
và một phương thức. Một trong các thuộc tính không thể được đặt trực tiếp, vì vậy nó
được định nghĩa bằng một phương thức getter (thay vì một biến). Phương thức này
sử dụng nội suy chuỗi để in các biểu diễn dạng chuỗi của biến bên trong
các string literal.

<?code-excerpt "misc/lib/samples/spacecraft.dart (class)"?>
```dart
class Spacecraft {
  String name;
  DateTime? launchDate;

  // Thuộc tính non-final chỉ đọc
  int? get launchYear => launchDate?.year;

  // Constructor, với cú pháp rút gọn để gán cho các member.
  Spacecraft(this.name, this.launchDate) {
    // Code khởi tạo được đặt tại đây.
  }

  // Named constructor chuyển tiếp đến constructor mặc định.
  Spacecraft.unlaunched(String name) : this(name, null);

  // Phương thức.
  void describe() {
    print('Spacecraft: $name');
    // Type promotion không hoạt động trên getter.
    var launchDate = this.launchDate;
    if (launchDate != null) {
      int years = DateTime.now().difference(launchDate).inDays ~/ 365;
      print('Launched: $launchYear ($years years ago)');
    } else {
      print('Unlaunched');
    }
  }
}
```

[Đọc thêm](/language/built-in-types#strings) về chuỗi,
bao gồm nội suy chuỗi, literal, biểu thức và phương thức `toString()`.

Bạn có thể sử dụng lớp `Spacecraft` như sau:

<?code-excerpt "misc/test/samples_test.dart (use-class)" plaster="none"?>
```dart
var voyager = Spacecraft('Voyager I', DateTime(1977, 9, 5));
voyager.describe();

var voyager3 = Spacecraft.unlaunched('Voyager III');
voyager3.describe();
```

[Đọc thêm](/language/classes) về các lớp trong Dart,
bao gồm initializer list, `new` và `const` tùy chọn, redirecting constructor,
constructor `factory`, getter, setter và nhiều nội dung khác.


<a id="enums"></a>

## Enum

Enum là một cách liệt kê một tập hợp giá trị hoặc instance được định nghĩa trước,
đồng thời bảo đảm không thể có bất kỳ instance nào khác của kiểu đó.

Sau đây là ví dụ về một `enum` đơn giản định nghĩa
một danh sách đơn giản gồm các kiểu hành tinh được định nghĩa trước:

<?code-excerpt "misc/lib/samples/spacecraft.dart (simple-enum)"?>
```dart
enum PlanetType { terrestrial, gas, ice }
```

Sau đây là ví dụ về một khai báo enhanced enum
cho một lớp mô tả các hành tinh,
với một tập hợp các instance hằng đã được định nghĩa,
cụ thể là những hành tinh trong chính hệ Mặt Trời của chúng ta.

<?code-excerpt "misc/lib/samples/spacecraft.dart (enhanced-enum)"?>
```dart
/// Enum liệt kê các hành tinh khác nhau trong hệ Mặt Trời của chúng ta
/// và một số thuộc tính của chúng.
enum Planet {
  mercury(planetType: PlanetType.terrestrial, moons: 0, hasRings: false),
  venus(planetType: PlanetType.terrestrial, moons: 0, hasRings: false),
  // ···
  uranus(planetType: PlanetType.ice, moons: 27, hasRings: true),
  neptune(planetType: PlanetType.ice, moons: 14, hasRings: true);

  /// Một generative constructor hằng
  const Planet({
    required this.planetType,
    required this.moons,
    required this.hasRings,
  });

  /// Tất cả các biến instance đều là final
  final PlanetType planetType;
  final int moons;
  final bool hasRings;

  /// Enhanced enum hỗ trợ getter và các phương thức khác
  bool get isGiant =>
      planetType == PlanetType.gas || planetType == PlanetType.ice;
}
```

Bạn có thể sử dụng enum `Planet` như sau:

<?code-excerpt "misc/test/samples_test.dart (use-enum)" plaster="none"?>
```dart
final yourPlanet = Planet.earth;

if (!yourPlanet.isGiant) {
  print('Your planet is not a "giant planet".');
}
```

Khi trình biên dịch có thể suy luận kiểu enum từ ngữ cảnh,
bạn có thể sử dụng cú pháp dot shorthand ngắn gọn hơn để truy cập các giá trị enum.
Thay vì viết đầy đủ `EnumName.value`,
bạn chỉ cần viết `.value`. Điều này có thể giúp code gọn gàng
và dễ đọc hơn.


Ví dụ, khi khai báo một biến có kiểu tường minh là `Planet`,
bạn có thể bỏ tên enum vì
kiểu `Planet` đã được xác lập:


```dart
// Thay cho cú pháp đầy đủ, tường minh:
Planet myPlanet = Planet.venus;

// Bạn có thể sử dụng dot shorthand:
Planet myPlanet = .venus;
```

Dot shorthand không chỉ giới hạn ở các khai báo biến.
Cú pháp này cũng có thể được sử dụng trong những ngữ cảnh như đối số hàm
và các case của switch, nơi kiểu enum đã rõ ràng với trình biên dịch.

[Đọc thêm](/language/enums) về enum trong Dart,
bao gồm các yêu cầu đối với enhanced enum, các thuộc tính được tự động đưa vào,
cách truy cập tên giá trị enum, hỗ trợ câu lệnh switch và nhiều nội dung khác.
[Đọc thêm](/language/dot-shorthands) về cú pháp dot shorthand.


<a id="inheritance"></a>

## Kế thừa

Dart có tính kế thừa đơn.

<?code-excerpt "misc/lib/samples/spacecraft.dart (extends)"?>
```dart
class Orbiter extends Spacecraft {
  double altitude;

  Orbiter(super.name, DateTime super.launchDate, this.altitude);
}
```

[Đọc thêm](/language/extend)
về việc mở rộng các lớp, annotation `@override` tùy chọn và nhiều nội dung khác.


<a id="mixins"></a>

## Mixin

Mixin là một cách tái sử dụng code trong nhiều hệ thống phân cấp lớp. Sau đây là
một khai báo mixin:

<?code-excerpt "misc/lib/samples/spacecraft.dart (mixin)"?>
```dart
mixin Piloted {
  int astronauts = 1;

  void describeCrew() {
    print('Number of astronauts: $astronauts');
  }
}
```

Để bổ sung các khả năng của một mixin vào một lớp, chỉ cần mở rộng lớp bằng mixin đó.

<?code-excerpt "misc/lib/samples/spacecraft.dart (mixin-use)" replace="/with/[!$&!]/g"?>
```dart
class PilotedCraft extends Spacecraft [!with!] Piloted {
  // ···
}
```

`PilotedCraft` giờ đây có field `astronauts` cũng như phương thức `describeCrew()`.

[Đọc thêm](/language/mixins) về mixin.


<a id="interfaces-and-abstract-classes"></a>

## Interface và lớp abstract

Mọi lớp đều ngầm định nghĩa một interface.
Do đó, bạn có thể implement bất kỳ lớp nào.

<?code-excerpt "misc/lib/samples/spacecraft.dart (implements)"?>
```dart
class MockSpaceship implements Spacecraft {
  // ···
}
```

Đọc thêm về [interface ngầm định](/language/classes#implicit-interfaces), hoặc
về [keyword `interface`](/language/class-modifiers#interface) tường minh.

Bạn có thể tạo một lớp abstract
để một lớp concrete mở rộng (hoặc implement).
Các lớp abstract có thể chứa các phương thức abstract (có phần thân rỗng).

<?code-excerpt "misc/lib/samples/spacecraft.dart (abstract)" replace="/abstract/[!$&!]/g"?>
```dart
[!abstract!] class Describable {
  void describe();

  void describeWithEmphasis() {
    print('=========');
    describe();
    print('=========');
  }
}
```

Bất kỳ lớp nào mở rộng `Describable` đều có phương thức `describeWithEmphasis()`,
phương thức này gọi phần triển khai `describe()` của lớp mở rộng.

[Đọc thêm](/language/class-modifiers#abstract)
về các lớp và phương thức abstract.


<a id="async"></a>

## Bất đồng bộ

Tránh callback hell và làm cho code dễ đọc hơn nhiều bằng cách
sử dụng `async` và `await`.

<?code-excerpt "misc/test/samples_test.dart (async)" replace="/async/[!$&!]/g"?>
```dart
const oneSecond = Duration(seconds: 1);
// ···
Future<void> printWithDelay(String message) [!async!] {
  await Future.delayed(oneSecond);
  print(message);
}
```

Phương thức trên tương đương với:

<?code-excerpt "misc/test/samples_test.dart (future-then)"?>
```dart
Future<void> printWithDelay(String message) {
  return Future.delayed(oneSecond).then((_) {
    print(message);
  });
}
```

Như ví dụ tiếp theo cho thấy, `async` và `await` giúp code bất đồng bộ
trở nên dễ đọc.

<?code-excerpt "misc/test/samples_test.dart (await)"?>
```dart
Future<void> createDescriptions(Iterable<String> objects) async {
  for (final object in objects) {
    try {
      var file = File('$object.txt');
      if (await file.exists()) {
        var modified = await file.lastModified();
        print(
          'File for $object already exists. It was modified on $modified.',
        );
        continue;
      }
      await file.create();
      await file.writeAsString('Start describing $object in this file.');
    } on IOException catch (e) {
      print('Cannot create description for $object: $e');
    }
  }
}
```

Bạn cũng có thể sử dụng `async*`, cú pháp này cung cấp một cách rõ ràng, dễ đọc để tạo stream.

<?code-excerpt "misc/test/samples_test.dart (async-star)"?>
```dart
Stream<String> report(Spacecraft craft, Iterable<String> objects) async* {
  for (final object in objects) {
    await Future.delayed(oneSecond);
    yield '${craft.name} flies by $object';
  }
}
```

[Đọc thêm](/language/async) về
hỗ trợ bất đồng bộ, bao gồm các hàm `async`, `Future`, `Stream`
và vòng lặp bất đồng bộ (`await for`).


<a id="exceptions"></a>

## Exception

Để phát sinh một exception, hãy sử dụng `throw`:

<?code-excerpt "misc/test/samples_test.dart (throw)"?>
```dart
if (astronauts == 0) {
  throw StateError('No astronauts.');
}
```

Để bắt một exception, hãy sử dụng câu lệnh `try` với `on` hoặc `catch` (hoặc cả hai):

<?code-excerpt "misc/test/samples_test.dart (try)" replace="/on.*e\)/[!$&!]/g"?>
```dart
Future<void> describeFlybyObjects(List<String> flybyObjects) async {
  try {
    for (final object in flybyObjects) {
      var description = await File('$object.txt').readAsString();
      print(description);
    }
  } [!on IOException catch (e)!] {
    print('Could not describe object: $e');
  } finally {
    flybyObjects.clear();
  }
}
```

Lưu ý rằng đoạn code trên là bất đồng bộ;
`try` hoạt động với cả code đồng bộ và bất đồng bộ trong một hàm `async`.

[Đọc thêm](/language/error-handling#exceptions) về exception,
bao gồm stack trace, `rethrow`
và sự khác biệt giữa `Error` với `Exception`.


<a id="important-concepts"></a>

## Các khái niệm quan trọng

Khi tiếp tục tìm hiểu ngôn ngữ Dart,
hãy ghi nhớ những dữ kiện và khái niệm sau:

-   Mọi thứ bạn có thể đặt vào một biến đều là một *đối tượng*, và mọi
    đối tượng đều là một instance của một *lớp*. Ngay cả số, hàm và
    `null` cũng là đối tượng.
    Ngoại trừ `null` (nếu bạn bật [sound null safety][ns]),
    tất cả đối tượng đều kế thừa từ lớp [`Object`][].

    :::version-note
    [Null safety][ns] được giới thiệu trong Dart 2.12.
    Việc sử dụng null safety yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 2.12.
    :::

-   Mặc dù Dart là ngôn ngữ strongly typed, type annotation là tùy chọn
    vì Dart có thể suy luận kiểu. Trong `var number = 101`, `number`
    được suy luận có kiểu `int`.

-   Biến không thể chứa `null` trừ khi bạn cho phép điều đó.
    Bạn có thể làm cho một biến trở thành nullable bằng cách
    đặt dấu hỏi (`?`) ở cuối kiểu của biến.
    Ví dụ, một biến có kiểu `int?` có thể là số nguyên
    hoặc có thể là `null`.
    Nếu bạn _biết_ một biểu thức không bao giờ được đánh giá thành `null`
    nhưng Dart không đồng ý,
    bạn có thể thêm `!` để khẳng định rằng biểu thức đó không phải null
    (và ném một exception nếu nó là null).
    Ví dụ: `int x = nullableButNotNullInt!`

-   Khi bạn muốn nói rõ rằng
    mọi kiểu đều được phép, hãy sử dụng kiểu `Object?`
    (nếu bạn đã bật null safety), `Object`,
    hoặc—nếu bạn phải trì hoãn việc kiểm tra kiểu đến runtime—
    [kiểu đặc biệt `dynamic`][ObjectVsDynamic].

-   Dart hỗ trợ các kiểu generic, chẳng hạn như `List<int>` (một danh sách số nguyên)
    hoặc `List<Object>` (một danh sách các đối tượng thuộc bất kỳ kiểu nào).

-   Dart hỗ trợ các hàm top-level (chẳng hạn như `main()`), cũng như
    những hàm gắn với một lớp hoặc đối tượng (tương ứng là các *phương thức static*
    và *phương thức instance*). Bạn cũng có thể tạo hàm bên trong
    hàm (các *hàm lồng nhau* hoặc *hàm cục bộ*).

-   Tương tự, Dart hỗ trợ các *biến* top-level, cũng như những biến
    gắn với một lớp hoặc đối tượng (biến static và biến instance). Biến instance
    đôi khi còn được gọi là *field* hoặc *property*.

-   Không giống Java, Dart không có các keyword `public`, `protected`
    và `private`. Nếu một identifier bắt đầu bằng dấu gạch dưới (`_`), identifier đó
    là private đối với library của nó. Để biết chi tiết, hãy xem
    [Library và import][].

-   *Identifier* có thể bắt đầu bằng một chữ cái hoặc dấu gạch dưới (`_`), theo sau là
    tổ hợp bất kỳ của những ký tự đó và chữ số.

-   Dart có cả *biểu thức* (có giá trị runtime) và
    *câu lệnh* (không có giá trị).
    Ví dụ, [biểu thức điều kiện][]
    `condition ? expr1 : expr2` có giá trị là `expr1` hoặc `expr2`.
    Hãy so sánh với một [câu lệnh if-else][], câu lệnh này không có giá trị.
    Một câu lệnh thường chứa một hoặc nhiều biểu thức,
    nhưng một biểu thức không thể trực tiếp chứa một câu lệnh.

-   Các công cụ Dart có thể báo cáo hai loại vấn đề: _warning_ và _error_.
    Warning chỉ là dấu hiệu cho thấy code của bạn có thể không hoạt động, nhưng
    chúng không ngăn chương trình thực thi. Error có thể xảy ra tại
    thời điểm biên dịch hoặc runtime. Lỗi tại thời điểm biên dịch ngăn code
    thực thi hoàn toàn; lỗi runtime dẫn đến việc một
    [exception][] được ném trong khi code thực thi.


<a id="additional-resources"></a>

## Tài nguyên bổ sung

Bạn có thể tìm thấy thêm tài liệu và ví dụ code trong
[tài liệu về thư viện cốt lõi](/libraries/dart-core)
và [tài liệu tham khảo API Dart]({{site.dart-api}}).
Code trên trang web này tuân theo các quy ước trong
[hướng dẫn về style của Dart](/effective-dart/style).

[Đặc tả ngôn ngữ Dart]: /resources/language/spec
[Chú thích]: /language/comments
[các kiểu dựng sẵn]: /language/built-in-types
[Chuỗi]: /language/built-in-types#strings
[Hàm main()]: /language/functions#the-main-function
[ns]: /null-safety
[`Object`]: {{site.dart-api}}/dart-core/Object-class.html
[phiên bản ngôn ngữ]: /language/versioning
[ObjectVsDynamic]: /effective-dart/design#avoid-using-dynamic-unless-you-want-to-disable-static-checking
[Library và import]: /language/libraries
[biểu thức điều kiện]: /language/operators#conditional-expressions
[câu lệnh if-else]: /language/branches#if
[exception]: /language/error-handling#exceptions
