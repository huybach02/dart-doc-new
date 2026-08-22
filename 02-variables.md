---
title: Biến
description: Tìm hiểu về các biến trong Dart.
prevpage:
  url: /language
  title: Giới thiệu
nextpage:
  url: /language/operators
  title: Toán tử
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

Sau đây là một ví dụ về cách tạo và khởi tạo một biến:

<?code-excerpt "misc/lib/language_tour/variables.dart (var-decl)"?>
```dart
var name = 'Bob';
```

Biến lưu trữ các tham chiếu. Biến có tên `name` chứa một tham chiếu đến
một đối tượng `String` có giá trị là "Bob".

Kiểu của biến `name` được suy luận là `String`,
nhưng bạn có thể thay đổi kiểu đó bằng cách chỉ định rõ kiểu.
Nếu một đối tượng không bị giới hạn ở một kiểu duy nhất,
hãy chỉ định kiểu `Object` (hoặc `dynamic` nếu cần).

<?code-excerpt "misc/lib/language_tour/variables.dart (type-decl)"?>
```dart
Object name = 'Bob';
```

Một lựa chọn khác là khai báo tường minh kiểu vốn sẽ được suy luận:

<?code-excerpt "misc/lib/language_tour/variables.dart (static-types)"?>
```dart
String name = 'Bob';
```

:::note
Trang này tuân theo
[khuyến nghị trong hướng dẫn về style](/effective-dart/design#types),
đó là sử dụng `var` thay cho type annotation đối với các biến cục bộ.
:::

<a id="null-safety"></a>

## Null safety

Ngôn ngữ Dart thực thi sound null safety.

Null safety ngăn chặn lỗi phát sinh do vô tình truy cập
các biến được đặt thành `null`. Lỗi này được gọi là lỗi null dereference.
Lỗi null dereference xảy ra khi bạn truy cập một thuộc tính hoặc gọi một phương thức
trên một biểu thức được đánh giá thành `null`.
Ngoại lệ của quy tắc này là khi `null` hỗ trợ thuộc tính hoặc phương thức đó,
chẳng hạn như `toString()` hoặc `hashCode`. Với null safety, trình biên dịch Dart
phát hiện những lỗi tiềm ẩn này tại thời điểm biên dịch.

Ví dụ, giả sử bạn muốn tìm giá trị tuyệt đối của một biến `int` tên là `i`.
Nếu `i` là `null`, việc gọi `i.abs()` sẽ gây ra lỗi null dereference.
Trong các ngôn ngữ khác, thao tác này có thể dẫn đến lỗi runtime.
Trình biên dịch của Dart ngăn lỗi này bằng cách không cho phép các thao tác đó.

Null safety đưa ra ba thay đổi chính:

1.  Khi chỉ định kiểu cho một biến, tham số hoặc thành phần liên quan khác,
    bạn có thể kiểm soát việc kiểu đó có cho phép `null` hay không.
    Để cho phép null, hãy thêm dấu `?` vào cuối phần khai báo kiểu.

    ```dart
    String? name  // Kiểu nullable. Có thể là `null` hoặc chuỗi.

    String name   // Kiểu non-nullable. Không thể là `null` nhưng có thể là chuỗi.
    ```

2.  Bạn phải khởi tạo các biến trước khi sử dụng chúng.
    Các biến nullable mặc định có giá trị `null`, vì vậy chúng được khởi tạo theo mặc định.
    Dart không đặt giá trị ban đầu cho các kiểu non-nullable.
    Dart buộc bạn phải đặt một giá trị ban đầu.
    Dart không cho phép bạn quan sát một biến chưa được khởi tạo.
    Điều này ngăn bạn truy cập các thuộc tính hoặc gọi các phương thức
    khi kiểu của receiver có thể là `null`
    nhưng `null` không hỗ trợ phương thức hoặc thuộc tính đang được sử dụng.

3.  Bạn không thể truy cập thuộc tính hoặc gọi phương thức trên một biểu thức có
    kiểu nullable. Ngoại lệ tương tự được áp dụng khi đó là một thuộc tính hoặc phương thức mà `null` hỗ trợ, chẳng hạn như `hashCode` hoặc `toString()`.

Sound null safety chuyển các **lỗi runtime** tiềm ẩn
thành các lỗi phân tích tại **thời điểm chỉnh sửa**.
Null safety đánh dấu một biến non-null khi biến đó thuộc một trong các trường hợp:

* Chưa được khởi tạo bằng một giá trị khác null.
* Được gán một giá trị `null`.

Việc kiểm tra này cho phép bạn sửa các lỗi đó _trước khi_ triển khai ứng dụng.

<a id="default-value"></a>

## Giá trị mặc định

Các biến chưa được khởi tạo có kiểu nullable
sẽ có giá trị ban đầu là `null`.
Ngay cả các biến có kiểu số ban đầu cũng là null,
bởi vì số—giống như mọi thứ khác trong Dart—đều là đối tượng.

<?code-excerpt "misc/test/language_tour/variables_test.dart (var-null-init)"?>
```dart
int? lineCount;
assert(lineCount == null);
```

:::note
Code production bỏ qua lời gọi `assert()`. Trong quá trình phát triển, ngược lại,
<code>assert(<em>condition</em>)</code> sẽ ném một exception nếu
_condition_ là false. Để biết chi tiết, hãy xem [Assert][].
:::

Với null safety, bạn phải khởi tạo giá trị
của các biến non-nullable trước khi sử dụng chúng:

<?code-excerpt "misc/lib/language_tour/variables.dart (var-ns-init)"?>
```dart
int lineCount = 0;
```

Bạn không phải khởi tạo một biến cục bộ ngay tại nơi khai báo,
nhưng bạn cần gán giá trị cho biến trước khi sử dụng.
Ví dụ, đoạn code sau hợp lệ vì
Dart có thể phát hiện rằng `lineCount` khác null vào thời điểm
nó được truyền vào `print()`:

<?code-excerpt "misc/lib/language_tour/variables.dart (var-ns-flow)"?>
```dart
int lineCount;

if (weLikeToCount) {
  lineCount = countLines();
} else {
  lineCount = 0;
}

print(lineCount);
```

Các biến top-level và biến lớp được khởi tạo lazy;
code khởi tạo sẽ chạy
vào lần đầu tiên biến được sử dụng.


<a id="late-variables"></a>

## Biến late

Modifier `late` có hai trường hợp sử dụng:

* Khai báo một biến non-nullable được khởi tạo sau khi khai báo.
* Khởi tạo một biến theo cách lazy.

Thông thường, quá trình phân tích control flow của Dart có thể phát hiện khi một biến non-nullable
được đặt thành một giá trị khác null trước khi được sử dụng,
nhưng đôi khi quá trình phân tích không làm được điều đó.
Hai trường hợp phổ biến là biến top-level và biến instance:
Dart thường không thể xác định liệu chúng đã được đặt hay chưa,
vì vậy Dart không cố xác định.

Nếu bạn chắc chắn rằng một biến đã được đặt trước khi sử dụng,
nhưng Dart không đồng ý,
bạn có thể sửa lỗi bằng cách đánh dấu biến là `late`:

<?code-excerpt "misc/lib/language_tour/variables.dart (var-late-top-level)" replace="/late/[!$&!]/g"?>
```dart
[!late!] String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```

:::warning Lưu ý
Nếu bạn không khởi tạo một biến `late`,
một lỗi runtime sẽ xảy ra khi biến được sử dụng.
:::

Khi bạn đánh dấu một biến là `late` nhưng khởi tạo biến ngay tại nơi khai báo,
initializer sẽ chạy vào lần đầu tiên biến được sử dụng.
Cách khởi tạo lazy này hữu ích trong một số trường hợp:

* Có thể không cần đến biến đó,
  đồng thời việc khởi tạo biến lại tốn kém.
* Bạn đang khởi tạo một biến instance
  và initializer của biến cần truy cập `this`.

Trong ví dụ sau,
nếu biến `temperature` không bao giờ được sử dụng,
thì hàm `readThermometer()` tốn kém sẽ không bao giờ được gọi:

<?code-excerpt "misc/lib/language_tour/variables.dart (var-late-lazy)" replace="/late/[!$&!]/g"?>
```dart
// Đây là lời gọi readThermometer() duy nhất của chương trình.
[!late!] String temperature = readThermometer(); // Được khởi tạo lazy.
```


<a id="final-and-const"></a>

## Final và const

Nếu không bao giờ có ý định thay đổi một biến, hãy sử dụng `final` hoặc `const`,
thay cho `var` hoặc dùng kèm một kiểu. Một biến final chỉ có thể được đặt
một lần; một biến const là hằng số tại thời điểm biên dịch. (Các biến const
ngầm định là final.)

:::note
[Các biến instance][] có thể là `final` nhưng không thể là `const`.
:::

Sau đây là ví dụ về cách tạo và đặt một biến `final`:

<?code-excerpt "misc/lib/language_tour/variables.dart (final)"?>
```dart
final name = 'Bob'; // Không có type annotation
final String nickname = 'Bobby';
```

Bạn không thể thay đổi giá trị của một biến `final`:

<?code-excerpt "misc/lib/language_tour/variables.dart (cant-assign-to-final)"?>
```dart tag=fails-sa
name = 'Alice'; // Lỗi: một biến final chỉ có thể được đặt một lần.
```

Sử dụng `const` cho các biến mà bạn muốn là **hằng số tại thời điểm biên dịch**. Nếu
biến const nằm ở cấp lớp, hãy đánh dấu nó là `static const`.
Tại nơi khai báo biến, hãy đặt giá trị thành một hằng số tại thời điểm biên dịch,
chẳng hạn như một literal số hoặc chuỗi, một biến const
hoặc kết quả của một phép toán số học trên các số không đổi:

<?code-excerpt "misc/lib/language_tour/variables.dart (const)"?>
```dart
const bar = 1000000; // Đơn vị áp suất (dynes/cm2)
const double atm = 1.01325 * bar; // Áp suất khí quyển tiêu chuẩn
```

Keyword `const` không chỉ dùng để khai báo các biến hằng.
Bạn cũng có thể dùng nó để tạo các _giá trị_ hằng,
cũng như để khai báo các constructor _tạo ra_ giá trị hằng.
Bất kỳ biến nào cũng có thể mang một giá trị hằng.

<?code-excerpt "misc/lib/language_tour/variables.dart (const-vs-final)"?>
```dart
var foo = const [];
final bar = const [];
const baz = []; // Tương đương với `const []`
```

Bạn có thể bỏ `const` khỏi biểu thức khởi tạo của một khai báo `const`,
như trường hợp `baz` ở trên. Để biết chi tiết, hãy xem [KHÔNG sử dụng const dư thừa][].

Bạn có thể thay đổi tham chiếu của một biến không phải final, không phải const,
ngay cả khi trước đó biến mang một giá trị `const`:

<?code-excerpt "misc/lib/language_tour/variables.dart (reassign-to-non-final)"?>
```dart
foo = [1, 2, 3]; // Trước đó là const []
```

Bạn không thể thay đổi giá trị của một biến `const`:

<?code-excerpt "misc/lib/language_tour/variables.dart (cant-assign-to-const)"?>
```dart tag=fails-sa
baz = [42]; // Lỗi: không thể gán giá trị cho các biến hằng.
```

Bạn có thể định nghĩa các hằng số sử dụng
[kiểm tra kiểu và ép kiểu][] (`is` và `as`),
[`if` trong collection][],
và [các toán tử spread][] (`...` và `...?`):

<?code-excerpt "misc/lib/language_tour/variables.dart (const-dart-25)"?>
```dart
const Object i = 3; // Trong đó i là một Object const có giá trị int...
const list = [i as int]; // Sử dụng phép ép kiểu.
const map = {if (i is int) i: 'int'}; // Sử dụng is và collection if.
const set = {if (list is List<int>) ...list}; // ...và một phép spread.
```

:::note
Mặc dù không thể sửa đổi một đối tượng `final`,
nhưng các field của đối tượng có thể thay đổi. 
Ngược lại, một đối tượng `const` và các field của nó
không thể thay đổi: chúng là _immutable_ (bất biến).
:::

Để biết thêm thông tin về cách sử dụng `const` nhằm tạo các giá trị hằng, hãy xem
[List][], [Map][] và [Lớp][].


<a id="wildcard-variables"></a>

## Biến wildcard

:::version-note
Biến wildcard yêu cầu
[phiên bản ngôn ngữ][] tối thiểu là 3.7.
:::

Một biến wildcard có tên `_` khai báo một biến cục bộ hoặc tham số
không binding; về cơ bản, đó là một placeholder.
Initializer, nếu có, vẫn được thực thi nhưng không thể truy cập giá trị.
Nhiều khai báo có tên `_` có thể tồn tại trong cùng một namespace mà không gây lỗi xung đột.

Các khai báo top-level hoặc các member mà tính riêng tư của library có thể bị ảnh hưởng
không phải là cách sử dụng hợp lệ của biến wildcard.
Các khai báo cục bộ trong một block scope, chẳng hạn như những ví dụ sau,
có thể khai báo một wildcard:

* Khai báo biến cục bộ.
  ```dart
  main() {
    var _ = 1;
    int _ = 2;
  }
  ```

* Khai báo biến trong vòng lặp `for`.
  ```dart
  for (var _ in list) {}
  ```

* Các tham số của mệnh đề `catch`.
  ```dart
  try {
    throw '!';
  } catch (_) {
    print('oops');
  }
  ```

* Các tham số kiểu generic và tham số kiểu hàm.
  ```dart
  class T<_> {}
  void genericFunction<_>() {}

  takeGenericCallback(<_>() => true);
  ```

* Các tham số hàm.
  ```dart
  Foo(_, this._, super._, void _()) {}

  list.where((_) => true);

  void f(void g(int _, bool _)) {}

  typedef T = void Function(String _, String _);
  ```

:::tip
Bật lint [`unnecessary_underscores`][] để xác định nơi một biến wildcard không binding `_`
có thể thay thế quy ước trước đây là sử dụng nhiều dấu gạch dưới có binding
(`__`, `___`, v.v.) nhằm tránh xung đột tên.
:::

[Assert]: /language/error-handling#assert
[Các biến instance]: /language/classes#instance-variables
[KHÔNG sử dụng const dư thừa]: /effective-dart/usage#dont-use-const-redundantly
[kiểm tra kiểu và ép kiểu]: /language/operators#type-test-operators
[`if` trong collection]: /language/collections#control-flow-operators
[các toán tử spread]: /language/collections#spread-operators
[List]: /language/collections#lists
[Map]: /language/collections#maps
[Lớp]: /language/classes
[phiên bản ngôn ngữ]: /language/versioning
[`unnecessary_underscores`]: /tools/linter-rules/unnecessary_underscores
