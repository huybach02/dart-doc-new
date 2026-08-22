---
title: The Dart type system
shortTitle: Hệ thống kiểu
description: Lý do và cách viết code Dart sound.
prevpage:
  url: /language/typedefs
  title: Typedef
nextpage:
  url: /language/patterns
  title: Pattern
---
<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /([A-Z]\w*)\d\b/$1/g; /\b(main)\d\b/$1/g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt path-base="type_system"?>

Ngôn ngữ Dart là type-safe: nó sử dụng kết hợp kiểm tra kiểu tĩnh
và [kiểm tra runtime](#runtime-checks) để
bảo đảm giá trị của một biến luôn khớp với kiểu tĩnh của biến,
đôi khi được gọi là sound typing.
Mặc dù _kiểu_ là bắt buộc, _type annotation_ là tùy chọn
nhờ [suy luận kiểu](#type-inference).

Một lợi ích của kiểm tra kiểu tĩnh là khả năng tìm lỗi
tại thời điểm biên dịch bằng [static analyzer][analysis] của Dart.

Bạn có thể sửa hầu hết lỗi phân tích tĩnh bằng cách thêm type annotation vào các lớp
generic. Các lớp generic phổ biến nhất là các kiểu collection
`List<T>` và `Map<K,V>`.

Ví dụ, trong đoạn code sau, hàm `printInts()` in một list số nguyên,
còn `main()` tạo một list và truyền nó vào `printInts()`.

<?code-excerpt "lib/strong_analysis.dart (opening-example)" replace="/list(?=\))/[!$&!]/g"?>
```dart tag=fails-sa
void printInts(List<int> a) => print(a);

void main() {
  final list = [];
  list.add(1);
  list.add('2');
  printInts([!list!]);
}
```

Đoạn code trên tạo ra lỗi kiểu tại `list` (được làm nổi bật
ở trên) trong lời gọi `printInts(list)`:

<?code-excerpt "analyzer-results-stable.txt" retain="/strong_analysis.*List.*argument_type_not_assignable/" replace="/-(.*?):(.*?):(.*?)-/-/g; /. • (lib|test)\/\w+\.dart:\d+:\d+//g"?>
```plaintext
error - The argument type 'List<dynamic>' can't be assigned to the parameter type 'List<int>'. - argument_type_not_assignable
```

Lỗi này làm nổi bật một phép ép kiểu ngầm không sound từ `List<dynamic>` sang `List<int>`.
Biến `list` có kiểu tĩnh `List<dynamic>`. Nguyên nhân là
khai báo khởi tạo `var list = []` không cung cấp đủ thông tin
để analyzer suy luận một đối số kiểu cụ thể hơn `dynamic`.
Hàm `printInts()` yêu cầu một tham số thuộc kiểu `List<int>`,
dẫn đến kiểu không khớp.

Khi thêm type annotation (`<int>`) lúc tạo list
(được làm nổi bật bên dưới), analyzer báo rằng
không thể gán đối số chuỗi cho tham số `int`.
Việc bỏ dấu nháy trong `list.add('2')` tạo ra code
vượt qua phân tích tĩnh và chạy mà không có error hoặc warning.

<?code-excerpt "test/strong_test.dart (opening-example)" replace="/<int.(?=\[)|2/[!$&!]/g"?>
```dart tag=passes-sa
void printInts(List<int> a) => print(a);

void main() {
  final list = [!<int>!][];
  list.add(1);
  list.add([!2!]);
  printInts(list);
}
```

[Thử trong DartPad]({{site.dartpad}}/?id=25074a51a00c71b4b000f33b688dedd0).

<a id="what-is-soundness"></a>

## Soundness là gì?

*Soundness* liên quan đến việc bảo đảm chương trình không thể rơi vào một số
trạng thái không hợp lệ. Một *hệ thống kiểu* sound có nghĩa là bạn không bao giờ có thể rơi vào
trạng thái mà một biểu thức được đánh giá thành giá trị không khớp
với kiểu tĩnh của biểu thức. Ví dụ, nếu kiểu tĩnh của một biểu thức
là `String`, thì tại runtime, bạn được bảo đảm chỉ nhận được một chuỗi
khi đánh giá biểu thức đó.

Hệ thống kiểu của Dart, giống hệ thống kiểu trong Java và C#, là sound. Nó
thực thi soundness bằng cách kết hợp kiểm tra tĩnh
(lỗi tại thời điểm biên dịch) với kiểm tra runtime. Ví dụ, việc gán một `String`
cho `int` là lỗi tại thời điểm biên dịch. Ép một đối tượng thành `String` bằng
`as String` sẽ thất bại với lỗi runtime nếu đối tượng không phải `String`.


<a id="the-benefits-of-soundness"></a>

## Lợi ích của soundness

Một hệ thống kiểu sound mang lại nhiều lợi ích:

* Phát hiện lỗi liên quan đến kiểu tại thời điểm biên dịch.<br>
  Hệ thống kiểu sound buộc code phải rõ ràng về kiểu,
  vì vậy những lỗi liên quan đến kiểu có thể khó tìm tại runtime
  sẽ được phát hiện tại thời điểm biên dịch.

* Code dễ đọc hơn.<br>
  Code dễ đọc hơn vì bạn có thể tin rằng một giá trị thực sự có
  kiểu đã chỉ định. Trong Dart sound, kiểu không thể nói dối.

* Code dễ bảo trì hơn.<br>
  Với hệ thống kiểu sound, khi bạn thay đổi một phần code,
  hệ thống kiểu có thể cảnh báo về những phần code khác
  vừa bị hỏng.

* Biên dịch ahead-of-time (AOT) tốt hơn.<br>
  Mặc dù có thể biên dịch AOT mà không cần kiểu, code được sinh ra
  sẽ kém hiệu quả hơn nhiều.


<a id="tips-for-passing-static-analysis"></a>

## Mẹo để vượt qua phân tích tĩnh

Hầu hết quy tắc dành cho kiểu tĩnh đều dễ hiểu.
Sau đây là một số quy tắc ít rõ ràng hơn:

* Sử dụng kiểu trả về sound khi override phương thức.
* Sử dụng kiểu tham số sound khi override phương thức.
* Không sử dụng list dynamic như một list có kiểu.

Hãy xem chi tiết các quy tắc này qua những ví dụ sử dụng hệ thống phân cấp
kiểu sau:

<img src="/assets/img/language/type-hierarchy.png" class="diagram-wrap" alt="hệ thống phân cấp động vật trong đó supertype là Animal và các subtype là Alligator, Cat cùng HoneyBadger; Cat có các subtype Lion và MaineCoon">

<a name="use-proper-return-types"></a>

### Sử dụng kiểu trả về sound khi override phương thức

Kiểu trả về của một phương thức trong subclass phải cùng kiểu hoặc là
subtype của kiểu trả về từ phương thức trong superclass.
Hãy xem phương thức getter trong lớp `Animal`:

<?code-excerpt "lib/animal.dart (Animal)" replace="/Animal get.*/[!$&!]/g"?>
```dart
class Animal {
  void chase(Animal a) {
     ...
  }
  [!Animal get parent => ...!]
}
```

Phương thức getter `parent` trả về một `Animal`. Trong subclass `HoneyBadger`,
bạn có thể thay kiểu trả về của getter bằng `HoneyBadger`
(hoặc bất kỳ subtype nào khác của `Animal`), nhưng không được dùng một kiểu không liên quan.

<?code-excerpt "lib/animal.dart (HoneyBadger)" replace="/(\w+)(?= get)/[!$&!]/g"?>
```dart tag=passes-sa
class HoneyBadger extends Animal {
  @override
  void chase(Animal a) {
     ...
  }

  @override
  [!HoneyBadger!] get parent => ...
}
```

<?code-excerpt "lib/animal.dart (HoneyBadger)" replace="/HoneyBadger get/[!Root!] get/g"?>
```dart tag=fails-sa
class HoneyBadger extends Animal {
  @override
  void chase(Animal a) {
     ...
  }

  @override
  [!Root!] get parent => ...
}
```

<a name="use-proper-param-types"></a>

### Sử dụng kiểu tham số sound khi override phương thức

Tham số của một phương thức được override phải có cùng kiểu
hoặc là supertype của tham số tương ứng trong superclass.
Không "thắt chặt" kiểu tham số bằng cách thay thế kiểu đó bằng
subtype của tham số ban đầu.

:::note
Nếu có lý do hợp lệ để sử dụng subtype, bạn có thể dùng
[keyword `covariant`](/language/type-system#covariant-keyword).
:::

Hãy xem phương thức `chase(Animal)` của lớp `Animal`:

<?code-excerpt "lib/animal.dart (Animal)" replace="/void chase.*/[!$&!]/g"?>
```dart
class Animal {
  [!void chase(Animal a) {!]
     ...
  }
  Animal get parent => ...
}
```

Phương thức `chase()` nhận một `Animal`. `HoneyBadger` đuổi theo bất cứ thứ gì.
Override phương thức `chase()` để nhận bất cứ thứ gì (`Object`) là hợp lệ.

<?code-excerpt "lib/animal.dart (chase-Object)" replace="/Object/[!$&!]/g"?>
```dart tag=passes-sa
class HoneyBadger extends Animal {
  @override
  void chase([!Object!] a) {
     ...
  }

  @override
  Animal get parent => ...
}
```

Đoạn code sau thắt chặt tham số của phương thức `chase()`
từ `Animal` thành `Mouse`, một subclass của `Animal`.

<?code-excerpt "lib/incorrect_animal.dart (chase-mouse)" replace="/Mouse/[!$&!]/g"?>
```dart tag=fails-sa
class [!Mouse!] extends Animal {
   ...
}

class Cat extends Animal {
  @override
  void chase([!Mouse!] a) {
     ...
  }
}
```

Đoạn code này không type-safe vì khi đó bạn có thể định nghĩa
một con mèo rồi sai nó đuổi theo một con cá sấu:

<?code-excerpt "lib/incorrect_animal.dart (would-not-be-type-safe)" replace="/Alligator/[!$&!]/g"?>
```dart
Animal a = Cat();
a.chase([!Alligator!]()); // Không type-safe và cũng không an toàn cho mèo.
```

<a id="dont-use-a-dynamic-list-as-a-typed-list"></a>

### Không sử dụng list dynamic như một list có kiểu

List `dynamic` phù hợp khi bạn muốn có một list chứa
nhiều loại đối tượng khác nhau. Tuy nhiên, bạn không thể sử dụng
list `dynamic` như một list có kiểu.

Quy tắc này cũng áp dụng cho các instance của kiểu generic.

Đoạn code sau tạo một list `dynamic` gồm `Dog` rồi gán nó cho
một list thuộc kiểu `Cat`, thao tác này tạo ra lỗi trong quá trình phân tích tĩnh.

<?code-excerpt "lib/incorrect_animal.dart (invalid-dynamic-list)" replace="/(<dynamic\x3E)(.*?)Error/[!$1!]$2Error/g"?>
```dart tag=fails-sa
void main() {
  List<Cat> foo = [!<dynamic>!][Dog()]; // Lỗi
  List<dynamic> bar = <dynamic>[Dog(), Cat()]; // OK
}
```

<a id="runtime-checks"></a>

## Kiểm tra runtime

Kiểm tra runtime xử lý các vấn đề type safety
không thể được phát hiện tại thời điểm biên dịch.

Ví dụ, đoạn code sau ném một exception tại runtime
vì việc ép một list chó thành list mèo là lỗi:

<?code-excerpt "test/strong_test.dart (runtime-checks)" replace="/animals as[^;]*/[!$&!]/g"?>
```dart tag=runtime-fail
void main() {
  List<Animal> animals = <Dog>[Dog()];
  List<Cat> cats = [!animals as List<Cat>!];
}
```

<a id="implicit-downcasts-from-dynamic"></a>

### Phép downcast ngầm từ `dynamic`

Biểu thức có kiểu tĩnh `dynamic` có thể được
ép kiểu ngầm thành một kiểu cụ thể hơn.
Nếu kiểu thực tế không khớp, phép ép sẽ ném lỗi tại runtime.
Hãy xem phương thức `assumeString` sau:

<?code-excerpt "lib/strong_analysis.dart (downcast-check)" replace="/string = object/[!$&!]/g"?>
```dart tag=passes-sa
int assumeString(dynamic object) {
  String [!string = object!]; // Kiểm tra tại runtime rằng `object` là `String`.
  return string.length;
}
```

Trong ví dụ này, nếu `object` là `String`, phép ép thành công.
Nếu nó không phải subtype của `String`, chẳng hạn như `int`,
một `TypeError` sẽ được ném:

<?code-excerpt "lib/strong_analysis.dart (fail-downcast-check)" replace="/1/[!$&!]/g"?>
```dart tag=runtime-fail
final length = assumeString([!1!]);
```

:::tip
Để ngăn phép downcast ngầm từ `dynamic` và tránh vấn đề này,
hãy cân nhắc bật chế độ _strict casts_ của analyzer.

```yaml title="analysis_options.yaml" highlightLines=3
analyzer:
  language:
    strict-casts: true
```

Để tìm hiểu thêm về cách tùy chỉnh hành vi của analyzer,
hãy xem [Tùy chỉnh phân tích tĩnh](/tools/analysis).
:::

<a id="type-inference"></a>

## Suy luận kiểu

Analyzer có thể suy luận kiểu cho field, phương thức, biến cục bộ
và hầu hết đối số kiểu generic.
Khi analyzer không có đủ thông tin để suy luận
một kiểu cụ thể, nó sử dụng kiểu `dynamic`.

Sau đây là ví dụ về cách suy luận kiểu hoạt động với generic.
Trong ví dụ này, biến tên `arguments` chứa một map
ghép các key dạng chuỗi với value thuộc nhiều kiểu khác nhau.

Nếu định kiểu tường minh cho biến, bạn có thể viết như sau:

<?code-excerpt "lib/strong_analysis.dart (type-inference-1-orig)" replace="/Map<String, Object\?\x3E/[!$&!]/g"?>
```dart
[!Map<String, Object?>!] arguments = {'argA': 'hello', 'argB': 42};
```

Hoặc bạn có thể sử dụng `var` hay `final` và để Dart suy luận kiểu:

<?code-excerpt "lib/strong_analysis.dart (type-inference-1)" replace="/var/[!$&!]/g"?>
```dart
[!var!] arguments = {'argA': 'hello', 'argB': 42}; // Map<String, Object>
```

Map literal suy luận kiểu của nó từ các entry,
sau đó biến suy luận kiểu từ kiểu của map literal.
Trong map này, các key đều là chuỗi nhưng value có
kiểu khác nhau (`String` và `int`, với upper bound là `Object`).
Vì vậy, map literal có kiểu `Map<String, Object>`,
và biến `arguments` cũng vậy.


<a id="field-and-method-inference"></a>

### Suy luận field và phương thức

Một field hoặc phương thức không có kiểu được chỉ định và override
field hoặc phương thức từ superclass sẽ kế thừa kiểu của
phương thức hoặc field trong superclass.

Một field không có kiểu được khai báo hoặc kế thừa nhưng được khai báo
với giá trị ban đầu sẽ nhận kiểu suy luận dựa trên giá trị ban đầu.

<a id="static-field-inference"></a>

### Suy luận field static

Field và biến static được suy luận kiểu từ
initializer của chúng. Lưu ý rằng quá trình suy luận thất bại nếu gặp một chu trình
(nghĩa là việc suy luận kiểu của biến phụ thuộc vào việc biết
kiểu của chính biến đó).

<a id="local-variable-inference"></a>

### Suy luận biến cục bộ

Kiểu biến cục bộ được suy luận từ initializer nếu có.
Các phép gán sau đó không được xem xét.
Điều này có thể khiến một kiểu quá cụ thể được suy luận.
Nếu vậy, bạn có thể thêm type annotation.

<?code-excerpt "lib/strong_analysis.dart (local-var-type-inference-error)"?>
```dart tag=fails-sa
var x = 3; // x được suy luận là int.
x = 4.0;
```

<?code-excerpt "lib/strong_analysis.dart (local-var-type-inference-ok)"?>
```dart tag=passes-sa
num y = 3; // num có thể là double hoặc int.
y = 4.0;
```

<a id="type-argument-inference"></a>

### Suy luận đối số kiểu

Đối số kiểu của lời gọi constructor và lời gọi
[phương thức generic](/language/generics#using-generic-methods)
được suy luận dựa trên sự kết hợp giữa thông tin đi xuống từ ngữ cảnh
xuất hiện và thông tin đi lên từ các đối số của constructor
hoặc phương thức generic. Nếu quá trình suy luận không làm điều bạn muốn hoặc mong đợi,
bạn luôn có thể chỉ định tường minh đối số kiểu.

<?code-excerpt "lib/strong_analysis.dart (type-arg-inference)"?>
```dart tag=passes-sa
// Được suy luận như khi bạn viết <int>[].
List<int> listOfInt = [];

// Được suy luận như khi bạn viết <double>[3.0].
var listOfDouble = [3.0];

// Được suy luận là Iterable<int>.
var ints = listOfDouble.map((x) => x.toInt());
```

Trong ví dụ cuối, `x` được suy luận là `double` bằng thông tin đi xuống.
Kiểu trả về của closure được suy luận là `int` bằng thông tin đi lên.
Dart sử dụng kiểu trả về này làm thông tin đi lên khi suy luận đối số kiểu
của phương thức `map()`: `<int>`.

<a id="inference-using-bounds"></a>

#### Suy luận bằng bound

:::version-note
Suy luận bằng bound yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 3.7.0.
:::

Với tính năng suy luận bằng bound,
thuật toán suy luận kiểu của Dart tạo các ràng buộc bằng cách
kết hợp những ràng buộc hiện có với các bound kiểu đã khai báo,
thay vì chỉ dùng các phép xấp xỉ best-effort.

Điều này đặc biệt quan trọng với kiểu [F-bound][],
trong đó suy luận bằng bound suy luận chính xác rằng, trong ví dụ bên dưới,
`X` có thể được bind với `B`.
Nếu không có tính năng này, đối số kiểu phải được chỉ định tường minh: `f<B>(C())`:

<?code-excerpt "lib/strong_analysis.dart (inference-using-bounds)"?>
```dart
class A<X extends A<X>> {}

class B extends A<B> {}

class C extends B {}

void f<X extends A<X>>(X x) {}

void main() {
  f(B()); // OK.

  // OK. Nếu không suy luận bằng bound, quá trình suy luận dựa trên các phép xấp xỉ
  // best-effort sẽ thất bại sau khi phát hiện `C` không phải subtype của `A<C>`.
  f(C());

  f<B>(C()); // OK.
}
```

Sau đây là ví dụ thực tế hơn sử dụng những kiểu thông dụng trong Dart như `int` hoặc `num`:

<?code-excerpt "lib/bounded/instantiate_to_bound.dart (inference-using-bounds-2)"?>
```dart
X max<X extends Comparable<X>>(X x1, X x2) => x1.compareTo(x2) > 0 ? x1 : x2;

void main() {
  // Được suy luận là `max<num>(3, 7)` khi có tính năng này; thất bại nếu không có.
  max(3, 7);
}
```

Với suy luận bằng bound, Dart có thể *phân rã* đối số kiểu,
trích xuất thông tin kiểu từ bound của tham số kiểu generic.
Điều này cho phép những hàm như `f` trong ví dụ sau bảo toàn cả
kiểu iterable cụ thể (`List` hoặc `Set`) *và* kiểu phần tử.
Trước khi có suy luận bằng bound, điều này không thể thực hiện được
mà không làm mất type safety hoặc thông tin kiểu cụ thể.

```dart
(X, Y) f<X extends Iterable<Y>, Y>(X x) => (x, x.first);

void main() {
  var (myList, myInt) = f([1]);
  myInt.whatever; // Lỗi tại thời điểm biên dịch, `myInt` có kiểu `int`.

  var (mySet, myString) = f({'Hello!'});
  mySet.union({}); // Hoạt động, `mySet` có kiểu `Set<String>`.
}
```

Nếu không có suy luận bằng bound, `myInt` sẽ có kiểu `dynamic`.
Thuật toán suy luận trước đây sẽ không phát hiện biểu thức không chính xác
`myInt.whatever` tại thời điểm biên dịch mà thay vào đó sẽ ném lỗi tại runtime.
Ngược lại, `mySet.union({})` sẽ là lỗi tại thời điểm biên dịch
nếu không có suy luận bằng bound, vì thuật toán trước đó không thể
bảo toàn thông tin rằng `mySet` là một `Set`.

Để biết thêm thông tin về thuật toán suy luận bằng bound,
hãy đọc [tài liệu thiết kế][].


[F-bound]: /language/generics/#f-bounds
[tài liệu thiết kế]: {{site.repo.dart.lang}}/blob/main/accepted/3.7/inference-using-bounds/design-document.md#motivating-example

<a id="substituting-types"></a>

## Thay thế kiểu

Khi override một phương thức, bạn thay thế một thành phần thuộc một kiểu (trong
phương thức cũ) bằng một thành phần có thể mang kiểu mới (trong phương thức mới).
Tương tự, khi truyền một đối số vào hàm,
bạn thay thế một thành phần mang một kiểu (tham số
có kiểu được khai báo) bằng một thành phần mang kiểu khác
(đối số thực tế). Khi nào bạn có thể thay thế một thành phần
mang một kiểu bằng thành phần mang subtype hoặc supertype?

Khi thay thế kiểu, việc suy nghĩ theo *consumer*
và *producer* sẽ hữu ích. Consumer hấp thụ một kiểu còn producer tạo ra một kiểu.

**Bạn có thể thay kiểu của consumer bằng một supertype và thay kiểu của producer
bằng một subtype.**

Hãy xem các ví dụ về phép gán kiểu đơn giản và phép gán với
kiểu generic.

<a id="simple-type-assignment"></a>

### Phép gán kiểu đơn giản

Khi gán đối tượng cho đối tượng, khi nào bạn có thể thay một kiểu bằng
kiểu khác? Câu trả lời phụ thuộc vào việc đối tượng là consumer
hay producer.

Hãy xem hệ thống phân cấp kiểu sau:

<img src="/assets/img/language/type-hierarchy.png" class="diagram-wrap" alt="hệ thống phân cấp động vật trong đó supertype là Animal và các subtype là Alligator, Cat cùng HoneyBadger; Cat có các subtype Lion và MaineCoon">

Hãy xem phép gán đơn giản sau, trong đó `Cat c` là một _consumer_
và `Cat()` là một _producer_:

<?code-excerpt "lib/strong_analysis.dart (Cat-Cat-ok)"?>
```dart
Cat c = Cat();
```

Ở vị trí consumer, thay một thành phần tiêu thụ một
kiểu cụ thể (`Cat`) bằng thành phần tiêu thụ bất cứ thứ gì (`Animal`) là an toàn,
vì vậy được phép thay `Cat c` bằng `Animal c`, do `Animal` là
supertype của `Cat`.

<?code-excerpt "lib/strong_analysis.dart (Animal-Cat-ok)"?>
```dart tag=passes-sa
Animal c = Cat();
```

Nhưng việc thay `Cat c` bằng `MaineCoon c` phá vỡ type safety,
vì superclass có thể cung cấp một kiểu `Cat` với hành vi khác,
chẳng hạn như `Lion`:

<?code-excerpt "lib/strong_analysis.dart (MaineCoon-Cat-err)"?>
```dart tag=fails-sa
MaineCoon c = Cat();
```

Ở vị trí producer, việc thay một thành phần tạo ra một
kiểu (`Cat`) bằng một kiểu cụ thể hơn (`MaineCoon`) là an toàn. Vì vậy,
đoạn sau được phép:

<?code-excerpt "lib/strong_analysis.dart (Cat-MaineCoon-ok)"?>
```dart tag=passes-sa
Cat c = MaineCoon();
```

<a id="generic-type-assignment"></a>

### Phép gán kiểu generic

Các quy tắc có giống với kiểu generic không? Có. Hãy xét hệ thống phân cấp
của các list động vật—một `List` của `Cat` là subtype của `List`
của `Animal`, đồng thời là supertype của `List` của `MaineCoon`:

<img src="/assets/img/language/type-hierarchy-generics.png" class="diagram-wrap" alt="List<Animal> -> List<Cat> -> List<MaineCoon>">

Trong ví dụ sau,
bạn có thể gán một list `MaineCoon` cho `myCats`
vì `List<MaineCoon>` là subtype của `List<Cat>`:

<?code-excerpt "lib/strong_analysis.dart (generic-type-assignment-MaineCoon)" replace="/<MaineCoon/<[!MaineCoon!]/g"?>
```dart tag=passes-sa
List<[!MaineCoon!]> myMaineCoons = ...
List<Cat> myCats = myMaineCoons;
```

Nếu đi theo hướng ngược lại thì sao?
Bạn có thể gán một list `Animal` cho `List<Cat>` không?

<?code-excerpt "lib/strong_analysis.dart (generic-type-assignment-Animal)" replace="/<Animal/<[!Animal!]/g"?>
```dart tag=fails-sa
List<[!Animal!]> myAnimals = ...
List<Cat> myCats = myAnimals;
```

Phép gán này không vượt qua phân tích tĩnh
vì nó tạo ra một phép downcast ngầm,
điều không được phép từ các kiểu không phải `dynamic` như `Animal`.

Để loại code này vượt qua phân tích tĩnh,
bạn có thể sử dụng một phép ép tường minh.

<?code-excerpt "lib/strong_analysis.dart (generic-type-assignment-implied-cast)" replace="/as.*(?=;)/[!$&!]/g"?>
```dart
List<Animal> myAnimals = ...
List<Cat> myCats = myAnimals [!as List<Cat>!];
```

Tuy nhiên, phép ép tường minh vẫn có thể thất bại tại runtime,
tùy thuộc vào kiểu thực tế của list đang được ép (`myAnimals`).

<a id="methods"></a>

### Phương thức

Khi override một phương thức, các quy tắc producer và consumer vẫn được áp dụng.
Ví dụ:

<img src="/assets/img/language/consumer-producer-methods.png" class="diagram-wrap" alt="lớp Animal cho thấy phương thức chase là consumer và getter parent là producer">

Đối với consumer (chẳng hạn như phương thức `chase(Animal)`), bạn có thể thay
kiểu tham số bằng supertype. Đối với producer (chẳng hạn như
phương thức getter `parent`), bạn có thể thay kiểu trả về bằng
subtype.

Để biết thêm thông tin, hãy xem
[Sử dụng kiểu trả về sound khi override phương thức](#use-proper-return-types)
và [Sử dụng kiểu tham số sound khi override phương thức](#use-proper-param-types).

<a id="covariant-keyword" aria-hidden="true"></a>

#### Tham số covariant

Một số pattern code (hiếm khi được sử dụng) dựa vào việc thắt chặt một kiểu
bằng cách override kiểu của tham số bằng subtype, điều này không hợp lệ.
Trong trường hợp này, bạn có thể sử dụng keyword `covariant` để
cho analyzer biết bạn đang cố ý làm vậy.
Điều này loại bỏ lỗi tĩnh và thay vào đó kiểm tra kiểu đối số không hợp lệ
tại runtime.

Phần sau cho thấy cách bạn có thể sử dụng `covariant`:

<?code-excerpt "lib/covariant.dart" replace="/covariant/[!$&!]/g"?>
```dart tag=passes-sa
class Animal {
  void chase(Animal x) {
     ...
  }
}

class Mouse extends Animal {
   ...
}

class Cat extends Animal {
  @override
  void chase([!covariant!] Mouse x) {
     ...
  }
}
```

Mặc dù ví dụ này minh họa việc sử dụng `covariant` trong subtype,
keyword `covariant` có thể được đặt trong phương thức của superclass
hoặc subclass.
Thông thường, phương thức của superclass là vị trí tốt nhất để đặt keyword này.
Keyword `covariant` áp dụng cho một tham số duy nhất và
cũng được hỗ trợ trên setter và field.

<a id="other-resources"></a>

## Các tài nguyên khác

Những tài nguyên sau cung cấp thêm thông tin về Dart sound:

* [Khắc phục lỗi type promotion](/tools/non-promotion-reasons) -
  Hiểu và tìm hiểu cách khắc phục lỗi type promotion.
* [Sound null safety](/null-safety) -
  Tìm hiểu cách viết code với sound null safety.
* [Tùy chỉnh phân tích tĩnh][analysis] -
  Cách thiết lập và tùy chỉnh analyzer cùng linter
  bằng file analysis options.


[analysis]: /tools/analysis
[phiên bản ngôn ngữ]: /language/versioning
[null safety]: /null-safety
