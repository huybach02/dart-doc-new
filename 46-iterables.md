---
title: Collection Iterable
description: >-
  Hướng dẫn tương tác về cách sử dụng object Iterable như list và set.
showBreadcrumbs: false
---
<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt plaster="none"?>

Hướng dẫn này dạy bạn cách sử dụng các collection
implement class [Iterable][iterable class]—ví dụ
[List][list class] và [Set.][set class]
Iterable là khối xây dựng cơ bản cho
mọi loại ứng dụng Dart,
và có lẽ bạn đã sử dụng chúng
mà không nhận ra.
Hướng dẫn này giúp bạn tận dụng tối đa chúng.

Sử dụng các trình chỉnh sửa DartPad nhúng,
bạn có thể kiểm tra kiến thức bằng cách
chạy code ví dụ và hoàn thành bài tập.

Để tận dụng tối đa hướng dẫn,
bạn nên có kiến thức cơ bản về [cú pháp Dart](/language).

Hướng dẫn này bao gồm các nội dung sau:

* Cách đọc element của Iterable.
* Cách kiểm tra element của Iterable có thỏa mãn điều kiện hay không.
* Cách lọc nội dung của Iterable.
* Cách ánh xạ nội dung của Iterable thành value khác.

Thời gian ước tính để hoàn thành hướng dẫn: 60 phút.

:::note
Trang này sử dụng DartPad nhúng để hiển thị ví dụ và bài tập.
{% render 'dartpads-embedded-troubleshooting.md' %}
:::

Các bài tập trong hướng dẫn có snippet code được hoàn thành một phần.
Bạn có thể dùng DartPad để kiểm tra kiến thức bằng cách hoàn thành code và
nhấp nút **Run**.
**Không chỉnh sửa code kiểm thử trong hàm `main` hoặc bên dưới**.

Nếu cần trợ giúp, hãy mở rộng dropdown **Hint** hoặc **Solution**
sau mỗi bài tập.

<a id="what-are-collections"></a>
## Collection là gì?

Collection là object
biểu diễn một nhóm object, được gọi là _element_.
Iterable là một loại collection.

Collection có thể rỗng hoặc chứa nhiều element.
Tùy mục đích,
collection có thể có cấu trúc và implementation khác nhau.
Đây là một số kiểu collection phổ biến nhất:

* [List:][list class] Dùng để đọc element theo index.
* [Set:][set class] Dùng để chứa element chỉ có thể xuất hiện một lần.
* [Map:][map class] Dùng để đọc element bằng key.


<a id="what-is-an-iterable"></a>
## Iterable là gì?

`Iterable` là collection các element có thể được truy cập tuần tự.

Trong Dart, `Iterable` là abstract class,
nghĩa là bạn không thể trực tiếp instantiate nó.
Tuy nhiên, bạn có thể tạo `Iterable` mới bằng cách tạo `List` hoặc `Set` mới.

Cả `List` và `Set` đều là `Iterable`,
nên có cùng method và property như class `Iterable`.

`Map` sử dụng cấu trúc dữ liệu nội bộ khác nhau,
tùy theo implementation.
Ví dụ, [HashMap][hashmap class] dùng hash table
trong đó element (còn gọi là _value_) được lấy bằng key.
Element của `Map` cũng có thể được đọc dưới dạng object `Iterable`
bằng property `entries` hoặc `values` của map.

Ví dụ này trình bày `List` các `int`,
đồng thời cũng là `Iterable` các `int`:

<?code-excerpt "iterables/test/iterables_test.dart (iterable)"?>
```dart
Iterable<int> iterable = [1, 2, 3];
```

Điểm khác với `List` là với `Iterable`,
bạn không thể bảo đảm đọc element theo index sẽ hiệu quả.
Không giống `List`, `Iterable` không có operator `[]`.

Ví dụ, hãy xem code **không hợp lệ** sau:

<?code-excerpt "iterables/test/iterables_test.dart (iterable-elementat)" replace="/\.elementAt\(1\)/[![1]!]/g"?>
```dart tag=bad
Iterable<int> iterable = [1, 2, 3];
int value = iterable[![1]!];
```

Nếu đọc element bằng `[]`,
compiler cho biết operator `'[]'`
không được định nghĩa cho class `Iterable`,
nghĩa là bạn không thể dùng `[index]` trong trường hợp này.

Thay vào đó, bạn có thể đọc element bằng `elementAt()`,
method này có thể bước qua các item của iterable cho đến khi
đến item tại vị trí đó.

<?code-excerpt "iterables/test/iterables_test.dart (iterable-elementat)"?>
```dart
Iterable<int> iterable = [1, 2, 3];
int value = iterable.elementAt(1);
```

Tùy implementation của iterable và số lượng item,
`elementAt` có thể có độ phức tạp tuyến tính và tốn kém.
Nếu định truy cập lặp lại các item cụ thể, hãy cân nhắc
gọi `.toList()` trên iterable để chuyển đổi nó thành list một lần,
rồi dùng operator `[]`.

<?code-excerpt "iterables/test/iterables_test.dart (iterable-to-list)"?>
```dart
final items = veryLargeIterable().toList();

final tenthItem = items[9];
final hundredthItem = items[99];
final thousandthItem = items[999];
final lastItem = items.last;
```

Tiếp tục phần sau để tìm hiểu thêm về
cách truy cập element của `Iterable`.

<a id="reading-elements"></a>
## Đọc element

Bạn có thể đọc tuần tự element của iterable
bằng vòng lặp `for-in`.

<a id="example-using-a-for-in-loop"></a>
### Ví dụ: Sử dụng vòng lặp for-in

Ví dụ sau trình bày cách đọc element bằng vòng lặp `for-in`.

<?code-excerpt "iterables/test/iterables_test.dart (for-in)"?>
```dartpad
void main() {
  const iterable = ['Salad', 'Popcorn', 'Toast'];
  for (final element in iterable) {
    print(element);
  }
}
```

:::note Chi tiết
Ở phía sau, vòng lặp `for-in` dùng một _iterator._
Tuy nhiên, bạn hiếm khi thấy [Iterator API][iterator class] được dùng trực tiếp
vì `for-in` dễ đọc, dễ hiểu hơn
và ít gây lỗi hơn.
:::

:::secondary Thuật ngữ chính
* **Iterable**: Class [Iterable][iterable class] của Dart.
* **Iterator**: Object được `for-in` dùng để đọc element từ
  object `Iterable`.
* **Vòng lặp `for-in`**: Cách dễ dàng để đọc tuần tự element từ
  `Iterable`.
:::

<a id="example-using-first-and-last"></a>
### Ví dụ: Sử dụng first và last

Trong một số trường hợp, bạn chỉ muốn truy cập element đầu tiên hoặc cuối cùng
của `Iterable`.

Với class `Iterable`, bạn không thể truy cập trực tiếp element,
nên không thể gọi `iterable[0]` để truy cập element đầu tiên.
Thay vào đó, bạn có thể dùng `first`,
để lấy element đầu tiên.

Ngoài ra, với class `Iterable`,
bạn không thể dùng operator `[]` để truy cập element cuối,
nhưng có thể dùng property `last`.

:::warning
Vì truy cập element cuối của `Iterable` đòi hỏi
bước qua mọi element khác,
**`last` có thể chậm.**
Dùng `first` hoặc `last` trên **`Iterable` rỗng**
dẫn đến [StateError.][StateError class]
:::

<?code-excerpt "iterables/test/iterables_test.dart (first-last)"?>
```dartpad
void main() {
  Iterable<String> iterable = const ['Salad', 'Popcorn', 'Toast'];
  print('The first element is ${iterable.first}');
  print('The last element is ${iterable.last}');
}
```

Trong ví dụ này, bạn đã thấy cách dùng `first` và `last` để
lấy element đầu và cuối của `Iterable`.
Bạn cũng có thể tìm element đầu tiên thỏa mãn một điều kiện.
Phần tiếp theo trình bày cách làm bằng method `firstWhere()`.

<a id="example-using-firstwhere"></a>
### Ví dụ: Sử dụng firstWhere()

Bạn đã thấy có thể truy cập tuần tự element của `Iterable`
và dễ dàng lấy element đầu hoặc cuối.

Giờ bạn sẽ học cách dùng `firstWhere()` để tìm element đầu tiên
thỏa mãn điều kiện nhất định.
Method này yêu cầu bạn truyền một _predicate_,
là hàm trả về true nếu
đầu vào thỏa mãn điều kiện nhất định.

<?code-excerpt "iterables/test/iterables_test.dart (firstwhere)"?>
```dart
String element = iterable.firstWhere((element) => element.length > 5);
```

Ví dụ, nếu muốn tìm `String` đầu tiên có
hơn 5 ký tự,
bạn phải truyền predicate trả về true khi
kích thước element lớn hơn 5.

Chạy ví dụ sau để xem `firstWhere()` hoạt động.
Bạn có nghĩ tất cả hàm sẽ cho cùng kết quả không?

<?code-excerpt "iterables/test/iterables_test.dart (first-where-long)"?>
```dartpad
bool predicate(String item) {
  return item.length > 5;
}

void main() {
  const items = ['Salad', 'Popcorn', 'Toast', 'Lasagne'];

  // Bạn có thể tìm bằng biểu thức đơn giản:
  var foundItem1 = items.firstWhere((item) => item.length > 5);
  print(foundItem1);

  // Hoặc thử dùng function block:
  var foundItem2 = items.firstWhere((item) {
    return item.length > 5;
  });
  print(foundItem2);

  // Hoặc thậm chí truyền function reference:
  var foundItem3 = items.firstWhere(predicate);
  print(foundItem3);

  // Bạn cũng có thể dùng hàm `orElse` nếu không tìm thấy value!
  var foundItem4 = items.firstWhere(
    (item) => item.length > 10,
    orElse: () => 'None!',
  );
  print(foundItem4);
}
```

Trong ví dụ này, bạn có thể thấy ba cách viết predicate:

* **Dạng biểu thức:**
  Code kiểm thử có một dòng dùng cú pháp mũi tên (`=>`).
* **Dạng block:**
  Code kiểm thử có nhiều dòng giữa dấu ngoặc và một câu lệnh return.
* **Dạng hàm:**
  Code kiểm thử nằm trong hàm bên ngoài được truyền vào
  method `firstWhere()` làm parameter.

Không có cách đúng hay sai.
Hãy dùng cách phù hợp nhất với bạn
và làm code dễ đọc, dễ hiểu hơn.

Ví dụ cuối gọi `firstWhere()` với
optional named parameter `orElse`,
cung cấp lựa chọn thay thế khi không tìm thấy element.
Trong trường hợp này, text `'None!'` được trả về vì
không element nào thỏa mãn điều kiện được cung cấp.

:::note
Nếu không element nào thỏa mãn test predicate và
parameter `orElse` không được cung cấp,
`firstWhere()` sẽ throw [StateError.][StateError class]
:::

:::secondary Ôn tập nhanh
* Element của `Iterable` phải được truy cập tuần tự.
* Cách dễ nhất để lặp qua mọi element là
  dùng vòng lặp `for-in`.
* Bạn có thể dùng getter `first` và `last` để lấy
  element đầu và cuối.
* Bạn cũng có thể tìm element đầu tiên
  thỏa mãn điều kiện bằng `firstWhere()`.
* Bạn có thể viết test predicate dưới dạng biểu thức, block hoặc hàm.

  **Thuật ngữ chính:**
* **Predicate:**
  Hàm trả về `true` khi một điều kiện nhất định được thỏa mãn.
:::

<a id="exercise-practice-writing-a-test-predicate"></a>
### Bài tập: Thực hành viết test predicate

Bài tập sau là unit test đang thất bại,
chứa snippet code được hoàn thành một phần.
Nhiệm vụ của bạn là hoàn thành bài tập bằng cách viết code để các test pass.
Bạn không cần implement `main()`.

Bài tập giới thiệu `singleWhere()`.
Method này hoạt động tương tự `firstWhere()`,
nhưng trong trường hợp này nó chỉ mong đợi một element của `Iterable`
thỏa mãn predicate.
Nếu nhiều hơn một hoặc không có element nào trong `Iterable`
thỏa mãn điều kiện predicate,
method sẽ throw exception [StateError][StateError class].

:::warning
`singleWhere()` bước qua toàn bộ `Iterable` đến element cuối,
có thể gây vấn đề nếu `Iterable` vô hạn hoặc
chứa collection element lớn.
:::

Mục tiêu là implement predicate cho `singleWhere()` thỏa mãn
các điều kiện sau:

* Element chứa ký tự `'a'`.
* Element bắt đầu bằng ký tự `'M'`.

Mọi element trong dữ liệu kiểm thử đều là [string][String class];
bạn có thể xem tài liệu class để được trợ giúp.

```dartpad theme="dark"
// Implement predicate của singleWhere
// với các điều kiện sau
// * Element chứa ký tự `'a'`
// * Element bắt đầu bằng ký tự `'M'`
String singleWhere(Iterable<String> items) {
  return items.singleWhere(TODO('Implement the outlined predicate.'));
}

// Code sau được dùng để cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.
void main() {
  const items = [
    'Salad',
    'Popcorn',
    'Milk',
    'Toast',
    'Sugar',
    'Mozzarella',
    'Tomato',
    'Egg',
    'Water',
  ];

  try {
    final str = singleWhere(items);
    if (str == 'Mozzarella') {
      print('Success. All tests passed!');
    } else {
      print(
        'Tried calling singleWhere, but received $str instead of '
        'the expected value \'Mozzarella\'',
      );
    }
  } on StateError catch (stateError) {
    print(
      'Tried calling singleWhere, but received a StateError: ${stateError.message}. '
      'singleWhere will fail if 0 or many elements match the predicate.',
    );
  } on UnimplementedError {
    print(
      'Tried running `singleWhere`, but received an error. '
      'Did you implement the function?',
    );
  } catch (e) {
    print('Tried calling singleWhere, but received an exception: $e');
  }
}
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập predicate.">Gợi ý</summary>

  Lời giải của bạn có thể dùng các method `contains` và `startsWith`
  từ class `String`.

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập predicate.">Lời giải</summary>

  ```dart
  String singleWhere(Iterable<String> items) {
    return items.singleWhere(
            (element) => element.startsWith('M') && element.contains('a'));
  }
  ```

</details>

<a id="checking-conditions"></a>
## Kiểm tra điều kiện

Khi làm việc với `Iterable`, đôi khi bạn cần xác minh
mọi element của collection thỏa mãn điều kiện nào đó.

Bạn có thể muốn viết lời giải bằng vòng lặp `for-in` như sau:

<?code-excerpt "iterables/test/iterables_test.dart (every-bad)"?>
```dart tag=bad
for (final item in items) {
  if (item.length < 5) {
    return false;
  }
}
return true;
```

Tuy nhiên, bạn có thể làm điều tương tự bằng method `every()`:

<?code-excerpt "iterables/test/iterables_test.dart (every-good)"?>
```dart
return items.every((item) => item.length >= 5);
```

Sử dụng method `every()` tạo ra code
dễ đọc, gọn và ít gây lỗi hơn.

<a id="example-using-any-and-every"></a>
### Ví dụ: Sử dụng any() và every()

Class `Iterable` cung cấp hai method mà
bạn có thể dùng để xác minh điều kiện:

* `any()`: Trả về true nếu ít nhất một element thỏa mãn điều kiện.
* `every()`: Trả về true nếu mọi element thỏa mãn điều kiện.

Chạy bài tập này để xem chúng hoạt động.

<?code-excerpt "iterables/test/iterables_test.dart (any-every)"?>
```dartpad
void main() {
  const items = ['Salad', 'Popcorn', 'Toast'];

  if (items.any((item) => item.contains('a'))) {
    print('At least one item contains "a"');
  }

  if (items.every((item) => item.length >= 5)) {
    print('All items have length >= 5');
  }
}
```

Trong ví dụ, `any()` xác minh rằng
ít nhất một element chứa ký tự `a`,
và `every()` xác minh mọi element
có độ dài bằng hoặc lớn hơn 5.

Sau khi chạy code, hãy thử thay đổi predicate của `any()` để
nó trả về false:

<?code-excerpt "iterables/test/iterables_test.dart (any-false)"?>
```dart
if (items.any((item) => item.contains('Z'))) {
  print('At least one item contains "Z"');
} else {
  print('No item contains "Z"');
}
```

Bạn cũng có thể dùng `any()` để xác minh không element nào của `Iterable`
thỏa mãn điều kiện nhất định.


<a id="exercise-verify-that-an-iterable-satisfies-a-condition"></a>
### Bài tập: Xác minh Iterable thỏa mãn điều kiện

Bài tập sau giúp thực hành dùng các method
`any()` và `every()` được mô tả trong ví dụ trước.
Trong trường hợp này, bạn làm việc với một nhóm người dùng,
được biểu diễn bằng object `User` có member field `age`.

Dùng `any()` và `every()` để implement hai hàm:

* Phần 1: Implement `anyUserUnder18()`.
  * Trả về `true` nếu ít nhất một người dùng từ 17 tuổi trở xuống.
* Phần 2: Implement `everyUserOver13()`.
  * Trả về `true` nếu mọi người dùng từ 14 tuổi trở lên.

```dartpad
bool anyUserUnder18(Iterable<User> users) {
  // TODO: Implement hàm anyUserUnder18.
}

bool everyUserOver13(Iterable<User> users) {
  // TODO: Implement hàm everyUserOver13.
}

class User {
  final String name;
  final int age;

  User(
    this.name,
    this.age,
  );
}

// Code sau được dùng để cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.
void main() {
  final users = [
    User('Alice', 21),
    User('Bob', 17),
    User('Claire', 52),
    User('David', 14),
  ];

  try {
    final out = anyUserUnder18(users);
    if (!out) {
      print('Looks like `anyUserUnder18` is wrong. Keep trying!');
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `anyUserUnder18`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print('Tried running `anyUserUnder18`, but received an exception: $e');
    return;
  }

  try {
    // Chỉ có một người dùng trên 18 tuổi, nên phải là false
    final out = anyUserUnder18([User('Alice', 21)]);
    if (out) {
      print(
          'Looks like `anyUserUnder18` is wrong. What if all users are over 18?');
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `anyUserUnder18`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running `anyUserUnder18([User("Alice", 21)])`, '
      'but received an exception: $e',
    );
    return;
  }

  try {
    final out = everyUserOver13(users);
    if (!out) {
      print(
        'Looks like `everyUserOver13` is wrong. '
        'There are no users under 13!',
      );
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `everyUserOver13`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running `everyUserOver13`, '
      'but received an exception: $e',
    );
    return;
  }

  try {
    final out = everyUserOver13([User('Dan', 12)]);
    if (out) {
      print(
        'Looks like `everyUserOver13` is wrong. '
        'There is at least one user under 13!',
      );
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `everyUserOver13`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running `everyUserOver13([User(\'Dan\', 12)])`, '
      'but received an exception: $e',
    );
    return;
  }

  print('Success. All tests passed!');
}
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập lọc có điều kiện.">Gợi ý</summary>

  Hãy nhớ dùng các method `any` và `every` từ class `Iterable`.
  Để được trợ giúp và xem ví dụ dùng các method này, hãy tham khảo
  [phần thảo luận trước đó](#example-using-any-and-every).

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập lọc có điều kiện.">Lời giải</summary>

  ```dart
  bool anyUserUnder18(Iterable<User> users) {
    return users.any((user) => user.age < 18);
  }
  
  bool everyUserOver13(Iterable<User> users) {
    return users.every((user) => user.age > 13);
  }
  ```

</details>

:::secondary Ôn tập nhanh
* Mặc dù có thể dùng vòng lặp `for-in` để kiểm tra điều kiện,
  có những cách tốt hơn để làm điều đó.
* Method `any()` cho phép bạn kiểm tra xem
  có element nào thỏa mãn điều kiện hay không.
* Method `every()` cho phép bạn xác minh
  mọi element đều thỏa mãn điều kiện.
:::

<a id="filtering"></a>
## Lọc

Các phần trước trình bày method như `firstWhere()` hoặc
`singleWhere()` có thể giúp tìm một element
thỏa mãn predicate nhất định.

Nhưng nếu bạn muốn tìm tất cả element
thỏa mãn điều kiện nhất định thì sao?
Bạn có thể làm điều đó bằng method `where()`.

<?code-excerpt "iterables/test/iterables_test.dart (where)"?>
```dart
var evenNumbers = numbers.where((number) => number.isEven);
```

Trong ví dụ này,
`numbers` chứa `Iterable` với nhiều value `int`, và
`where()` tìm tất cả số chẵn.

Đầu ra của `where()` là một `Iterable` khác,
và bạn có thể dùng nó như vậy để lặp hoặc
áp dụng các method `Iterable` khác.
Trong ví dụ tiếp theo, đầu ra của `where()`
được dùng trực tiếp bên trong vòng lặp `for-in`.

<?code-excerpt "iterables/test/iterables_test.dart (where-for)"?>
```dart
var evenNumbers = numbers.where((number) => number.isEven);
for (final number in evenNumbers) {
  print('$number is even');
}
```

<a id="example-using-where"></a>
### Ví dụ: Sử dụng where()

Chạy ví dụ này để xem cách dùng `where()` cùng các method khác
như `any()`.

<?code-excerpt "iterables/test/iterables_test.dart (numbers-where)"?>
```dartpad
void main() {
  var evenNumbers = const [1, -2, 3, 42].where((number) => number.isEven);

  for (final number in evenNumbers) {
    print('$number is even.');
  }

  if (evenNumbers.any((number) => number.isNegative)) {
    print('evenNumbers contains negative numbers.');
  }

  // Nếu không element nào thỏa mãn predicate, đầu ra sẽ rỗng.
  var largeNumbers = evenNumbers.where((number) => number > 1000);
  if (largeNumbers.isEmpty) {
    print('largeNumbers is empty!');
  }
}
```

Trong ví dụ này, `where()` được dùng để tìm mọi số chẵn, sau đó
`any()` được dùng để kiểm tra kết quả có chứa số âm không.

Sau đó trong ví dụ, `where()` lại được dùng để
tìm mọi số lớn hơn 1000.
Vì không có số nào, kết quả là `Iterable` rỗng.

:::note
Nếu không element nào thỏa mãn predicate trong `where()`,
method trả về `Iterable` rỗng.
Không giống `singleWhere()` hoặc `firstWhere()`,
`where()` không throw exception [StateError][StateError class].
:::

<a id="example-using-takewhile"></a>
### Ví dụ: Sử dụng takeWhile

Các method `takeWhile()` và `skipWhile()` cũng có thể
giúp bạn lọc element khỏi `Iterable`.

Chạy ví dụ để xem `takeWhile()` và `skipWhile()` có thể
chia `Iterable` chứa các số như thế nào.

<?code-excerpt "iterables/test/iterables_test.dart (take-while-long)"?>
```dartpad
void main() {
  const numbers = [1, 3, -2, 0, 4, 5];

  var numbersUntilZero = numbers.takeWhile((number) => number != 0);
  print('Numbers until 0: $numbersUntilZero');

  var numbersStartingAtZero = numbers.skipWhile((number) => number != 0);
  print('Numbers starting at 0: $numbersStartingAtZero');
}
```

Trong ví dụ này, `takeWhile()` trả về `Iterable`
chứa mọi element trước element
thỏa mãn predicate.
Mặt khác, `skipWhile()` trả về `Iterable`
chứa mọi element sau và bao gồm element đầu tiên
_không_ thỏa mãn predicate.

Sau khi chạy ví dụ,
hãy thay đổi `takeWhile()` để lấy element cho đến khi
gặp số âm đầu tiên.

<?code-excerpt "iterables/test/iterables_test.dart (takewhile)"?>
```dart
var numbersUntilNegative = numbers.takeWhile(
  (number) => !number.isNegative,
);
```

Lưu ý điều kiện `number.isNegative` được phủ định bằng `!`.

<a id="exercise-filtering-elements-from-a-list"></a>
### Bài tập: Lọc element khỏi list

Bài tập sau giúp thực hành dùng method `where()` với
class `User` từ bài tập trước.

Dùng `where()` để implement hai hàm:

* Phần 1: Implement `filterOutUnder21()`.
  * Trả về `Iterable` chứa mọi người dùng từ 21 tuổi trở lên.
* Phần 2: Implement `findShortNamed()`.
  * Trả về `Iterable` chứa mọi người dùng có
    tên dài từ 3 ký tự trở xuống.

```dartpad theme="dark"
Iterable<User> filterOutUnder21(Iterable<User> users) {
  // TODO: Implement hàm filterOutUnder21.
}

Iterable<User> findShortNamed(Iterable<User> users) {
  // TODO: Implement hàm findShortNamed.
}

class User {
  final String name;
  final int age;

  User(
    this.name,
    this.age,
  );
}

// Code sau được dùng để cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.
void main() {
  final users = [
    User('Alice', 21),
    User('Bob', 17),
    User('Claire', 52),
    User('Dan', 12),
  ];

  try {
    final out = filterOutUnder21(users);
    if (out.any((user) => user.age < 21) || out.length != 2) {
      print(
        'Looks like `filterOutUnder21` is wrong, there are '
        'exactly two users with age under 21. Keep trying!',
      );
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `filterOutUnder21`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running `filterOutUnder21`, '
      'but received an exception: ${e.runtimeType}',
    );
    return;
  }

  try {
    final out = findShortNamed(users);
    if (out.any((user) => user.name.length > 3) || out.length != 2) {
      print(
        'Looks like `findShortNamed` is wrong, there are '
        'exactly two users with a three letter name. Keep trying!',
      );
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `findShortNamed`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running `findShortNamed`, '
      'but received an exception: ${e.runtimeType}',
    );
    return;
  }

  print('Success. All tests passed!');
}
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập lọc element.">Gợi ý</summary>

  Hãy nhớ tận dụng method `where` từ class `Iterable`.
  Để được trợ giúp và xem ví dụ dùng `where`, hãy tham khảo
  [phần thảo luận trước đó](#example-using-where).

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập lọc element.">Lời giải</summary>

  ```dart
  Iterable<User> filterOutUnder21(Iterable<User> users) {
    return users.where((user) => user.age >= 21);
  }
  
  Iterable<User> findShortNamed(Iterable<User> users) {
    return users.where((user) => user.name.length <= 3);
  }
  ```

</details>

:::secondary Ôn tập nhanh
* Lọc element của `Iterable` bằng `where()`.
* Đầu ra của `where()` là một `Iterable` khác.
* Dùng `takeWhile()` và `skipWhile()` để lấy element cho đến trước hoặc sau
  khi điều kiện được đáp ứng.
* Đầu ra của các method này có thể là `Iterable` rỗng.
:::

<a id="mapping"></a>
## Ánh xạ

Ánh xạ `Iterable` bằng method `map()` cho phép bạn
áp dụng một hàm trên từng element,
thay mỗi element bằng một element mới.

<?code-excerpt "iterables/test/iterables_test.dart (map-int)"?>
```dart
Iterable<int> output = numbers.map((number) => number * 10);
```

Trong ví dụ này, mỗi element của `Iterable` numbers được nhân với 10.

Bạn cũng có thể dùng `map()`
để biến đổi element thành object khác—ví dụ,
chuyển mọi `int` thành `String`,
như trong ví dụ sau:

<?code-excerpt "iterables/test/iterables_test.dart (map-string)"?>
```dart
Iterable<String> output = numbers.map((number) => number.toString());
```

:::note
`map()` trả về `Iterable` _lazy_, nghĩa là hàm được cung cấp
chỉ được gọi khi element được lặp.
:::

<a id="example-using-map-to-change-elements"></a>
### Ví dụ: Sử dụng map để thay đổi element

Chạy ví dụ để xem cách dùng `map()` nhằm
nhân mọi element của `Iterable` với 2.
Bạn nghĩ đầu ra sẽ là gì?

<?code-excerpt "iterables/test/iterables_test.dart (numbers-by-two)"?>
```dartpad
void main() {
  var numbersByTwo = const [1, -2, 3, 42].map((number) => number * 2);
  print('Numbers: $numbersByTwo');
}
```


<a id="exercise-mapping-to-a-different-type"></a>
### Bài tập: Ánh xạ sang kiểu khác

Trong ví dụ trước, bạn nhân element của `Iterable` với 2.
Cả đầu vào và đầu ra của thao tác đó đều là `Iterable` các `int`.

Trong bài tập này, code nhận một `Iterable` các `User`,
và bạn cần trả về `Iterable`
chứa string gồm tên và tuổi của từng người dùng.

Mỗi string trong `Iterable` phải theo định dạng:
`'{name} is {age}'`—ví dụ `'Alice is 21'`.

```dartpad theme="dark"
Iterable<String> getNameAndAges(Iterable<User> users) {
  // TODO: Implement hàm getNameAndAges.
}

class User {
  final String name;
  final int age;

  User(
    this.name,
    this.age,
  );
}

// Code sau được dùng để cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.
void main() {
  final users = [
    User('Alice', 21),
    User('Bob', 17),
    User('Claire', 52),
  ];

  try {
    final out = getNameAndAges(users).toList();
    if (!_listEquals(out, ['Alice is 21', 'Bob is 17', 'Claire is 52'])) {
      print(
        'Looks like `getNameAndAges` is wrong. Keep trying! '
        'The output was: $out',
      );
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `getNameAndAges`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print('Tried running the function, but received an exception: $e');
    return;
  }

  print('Success. All tests passed!');
}

bool _listEquals<T>(List<T>? a, List<T>? b) {
  if (a == null) return b == null;
  if (b == null || a.length != b.length) return false;
  for (var index = 0; index < a.length; index += 1) {
    if (a[index] != b[index]) return false;
  }
  return true;
}
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập ánh xạ element.">Gợi ý</summary>

  Hãy nhớ tận dụng method `map` từ class `Iterable`.
  Để được trợ giúp và xem ví dụ dùng `map`, hãy tham khảo
  [phần thảo luận trước đó](#example-using-map-to-change-elements).

  Để nối nhiều value thành một string, hãy cân nhắc
  sử dụng [nội suy string](/language/built-in-types#string-interpolation).

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập ánh xạ element.">Lời giải</summary>

  ```dart
  Iterable<String> getNameAndAges(Iterable<User> users) {
    return users.map((user) => '${user.name} is ${user.age}');
  }
  ```

</details>

:::secondary Ôn tập nhanh
* `map()` áp dụng một hàm cho mọi element của `Iterable`.
* Đầu ra của `map()` là một `Iterable` khác.
* Hàm không được đánh giá cho đến khi `Iterable` được lặp.
:::

<a id="exercise-putting-it-all-together"></a>
## Bài tập: Kết hợp tất cả

Đã đến lúc thực hành những gì bạn học được trong một bài tập cuối.

Bài tập cung cấp class `EmailAddress`,
có constructor nhận một string.
Một hàm khác được cung cấp là `isValidEmailAddress()`,
kiểm tra địa chỉ email có hợp lệ hay không.

| Constructor/hàm       | Type signature                           | Mô tả                                                    |
|-----------------------|------------------------------------------|----------------------------------------------------------|
| EmailAddress()        | `EmailAddress(String address)`           | Tạo `EmailAddress` cho địa chỉ được chỉ định.             |
| isValidEmailAddress() | `bool isValidEmailAddress(EmailAddress)` | Trả về `true` nếu `EmailAddress` được cung cấp hợp lệ.     |

{:.table .table-striped}

Viết code sau:

Phần 1: Implement `parseEmailAddresses()`.
- Viết hàm `parseEmailAddresses()`,
  nhận `Iterable<String>` chứa các địa chỉ email
  và trả về `Iterable<EmailAddress>`.
- Dùng method `map()` để ánh xạ từ `String` sang `EmailAddress`.
- Tạo object `EmailAddress` bằng
  constructor `EmailAddress(String)`.

Phần 2: Implement `anyInvalidEmailAddress()`.
- Viết hàm `anyInvalidEmailAddress()`,
  nhận `Iterable<EmailAddress>` và
  trả về `true` nếu có `EmailAddress` nào trong `Iterable` không hợp lệ.
- Dùng method `any()` cùng
  hàm `isValidEmailAddress()` được cung cấp.

Phần 3: Implement `validEmailAddresses()`.
- Viết hàm `validEmailAddresses()`,
  nhận `Iterable<EmailAddress>` và
  trả về `Iterable<EmailAddress>` khác chỉ chứa địa chỉ hợp lệ.
- Dùng method `where()` để lọc `Iterable<EmailAddress>`.
- Dùng hàm `isValidEmailAddress()` được cung cấp để đánh giá
  một `EmailAddress` có hợp lệ không.

```dartpad theme="dark"
Iterable<EmailAddress> parseEmailAddresses(Iterable<String> strings) {
  // TODO: Implement hàm parseEmailAddresses.
}

bool anyInvalidEmailAddress(Iterable<EmailAddress> emails) {
  // TODO: Implement hàm anyInvalidEmailAddress.
}

Iterable<EmailAddress> validEmailAddresses(Iterable<EmailAddress> emails) {
  // TODO: Implement hàm validEmailAddresses.
}

class EmailAddress {
  final String address;

  EmailAddress(this.address);

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is EmailAddress && address == other.address;

  @override
  int get hashCode => address.hashCode;

  @override
  String toString() => 'EmailAddress{address: $address}';
}

// Code sau được dùng để cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.
void main() {
  const input = [
    'ali@gmail.com',
    'bobgmail.com',
    'cal@gmail.com',
  ];

  const correctInput = ['dash@gmail.com', 'sparky@gmail.com'];

  bool _listEquals<T>(List<T>? a, List<T>? b) {
    if (a == null) return b == null;
    if (b == null || a.length != b.length) return false;
    for (var index = 0; index < a.length; index += 1) {
      if (a[index] != b[index]) return false;
    }
    return true;
  }

  final Iterable<EmailAddress> emails;
  final Iterable<EmailAddress> correctEmails;
  try {
    emails = parseEmailAddresses(input);
    correctEmails = parseEmailAddresses(correctInput);
    if (emails.isEmpty) {
      print(
        'Tried running `parseEmailAddresses`, but received an empty list.',
      );
      return;
    }
    if (!_listEquals(emails.toList(), [
      EmailAddress('ali@gmail.com'),
      EmailAddress('bobgmail.com'),
      EmailAddress('cal@gmail.com'),
    ])) {
      print('Looks like `parseEmailAddresses` is wrong. Keep trying!');
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `parseEmailAddresses`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running `parseEmailAddresses`, '
      'but received an exception: $e',
    );
    return;
  }

  try {
    final out = anyInvalidEmailAddress(emails);
    if (!out) {
      print(
        'Looks like `anyInvalidEmailAddress` is wrong. Keep trying! '
        'The result should be false with at least one invalid address.',
      );
      return;
    }
    final falseOut = anyInvalidEmailAddress(correctEmails);
    if (falseOut) {
      print(
        'Looks like `anyInvalidEmailAddress` is wrong. Keep trying! '
        'The result should be false with all valid addresses.',
      );
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `anyInvalidEmailAddress`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
        'Tried running `anyInvalidEmailAddress`, but received an exception: $e');
    return;
  }

  try {
    final valid = validEmailAddresses(emails);
    if (emails.isEmpty) {
      print('Tried running `validEmailAddresses`, but received an empty list.');
      return;
    }
    if (!_listEquals(valid.toList(), [
      EmailAddress('ali@gmail.com'),
      EmailAddress('cal@gmail.com'),
    ])) {
      print('Looks like `validEmailAddresses` is wrong. Keep trying!');
      return;
    }
  } on UnimplementedError {
    print(
      'Tried running `validEmailAddresses`, but received an error. '
      'Did you implement the function?',
    );
    return;
  } catch (e) {
    print(
      'Tried running the `validEmailAddresses`, '
      'but received an exception: $e',
    );
    return;
  }

  print('Success. All tests passed!');
}

bool isValidEmailAddress(EmailAddress email) {
  return email.address.contains('@');
}
```

<details>
  <summary title="Mở rộng để xem lời giải của bài tập 'Kết hợp tất cả'.">Lời giải</summary>

  ```dart
  Iterable<EmailAddress> parseEmailAddresses(Iterable<String> strings) {
    return strings.map((s) => EmailAddress(s));
  }
  
  bool anyInvalidEmailAddress(Iterable<EmailAddress> emails) {
    return emails.any((email) => !isValidEmailAddress(email));
  }
  
  Iterable<EmailAddress> validEmailAddresses(Iterable<EmailAddress> emails) {
    return emails.where((email) => isValidEmailAddress(email));
  }
  ```

</details>

<a id="whats-next"></a>
## Tiếp theo là gì

Chúc mừng, bạn đã hoàn thành hướng dẫn!
Nếu muốn tìm hiểu thêm,
đây là một số gợi ý cho bước tiếp theo:

* Thử nghiệm với [DartPad.]({{site.dartpad}})
* Đọc [tài liệu tham khảo API của Iterable][iterable class]
  để tìm hiểu các method không được hướng dẫn này đề cập.

[hashmap class]: {{site.dart-api}}/dart-collection/HashMap-class.html
[iterable class]: {{site.dart-api}}/dart-core/Iterable-class.html
[iterator class]: {{site.dart-api}}/dart-core/Iterator-class.html
[list class]: {{site.dart-api}}/dart-core/List-class.html
[map class]: {{site.dart-api}}/dart-core/Map-class.html
[set class]: {{site.dart-api}}/dart-core/Set-class.html
[StateError class]: {{site.dart-api}}/dart-core/StateError-class.html
[String class]: {{site.dart-api}}/dart-core/String-class.html
