---
title: "Lập trình bất đồng bộ: future, async, await"
shortTitle: Future, async và await
description: Tìm hiểu và thực hành viết code bất đồng bộ trong DartPad!
---
<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt plaster="none"?>

Hướng dẫn này dạy bạn cách viết code bất đồng bộ bằng
future và các keyword `async`, `await`.
Với trình chỉnh sửa DartPad nhúng,
bạn có thể kiểm tra kiến thức bằng cách chạy code ví dụ
và hoàn thành bài tập.

Để tận dụng tối đa hướng dẫn, bạn nên có:

* Kiến thức về [cú pháp Dart cơ bản](/language).
* Một ít kinh nghiệm viết code bất đồng bộ trong ngôn ngữ khác.
* Đã bật các lint [`discarded_futures`][] và [`unawaited_futures`][].

[`discarded_futures`]: /tools/linter-rules/discarded_futures
[`unawaited_futures`]: /tools/linter-rules/unawaited_futures

Hướng dẫn này bao gồm:

* Cách và thời điểm dùng các keyword `async`, `await`.
* Cách việc dùng `async`, `await` ảnh hưởng đến thứ tự thực thi.
* Cách xử lý error từ lệnh gọi bất đồng bộ
  bằng biểu thức `try-catch` trong hàm `async`.

Thời gian ước tính để hoàn thành hướng dẫn: 40–60 phút.

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

<a id="why-asynchronous-code-matters"></a>
## Vì sao code bất đồng bộ quan trọng

Thao tác bất đồng bộ cho phép chương trình hoàn thành công việc
trong khi chờ thao tác khác kết thúc.
Đây là một số thao tác bất đồng bộ phổ biến:

* Lấy dữ liệu qua mạng.
* Ghi vào database.
* Đọc dữ liệu từ file.

Những phép tính bất đồng bộ như vậy thường cung cấp kết quả dưới dạng `Future`
hoặc, nếu kết quả có nhiều phần, dưới dạng `Stream`.
Những phép tính này đưa tính bất đồng bộ vào chương trình.
Để thích ứng với tính bất đồng bộ ban đầu đó,
các hàm Dart thông thường khác cũng cần trở thành bất đồng bộ.

Để tương tác với các kết quả bất đồng bộ,
bạn có thể dùng keyword `async` và `await`.
Phần lớn hàm bất đồng bộ chỉ là hàm Dart async
phụ thuộc, có thể ở sâu bên dưới,
vào một phép tính vốn dĩ bất đồng bộ.

<a id="example-incorrectly-using-an-asynchronous-function"></a>
### Ví dụ: Sử dụng sai một hàm bất đồng bộ

Ví dụ sau trình bày cách sai
để dùng hàm bất đồng bộ (`fetchUserOrder()`).
Sau đó bạn sẽ sửa ví dụ bằng `async` và `await`.
Trước khi chạy, hãy thử tìm vấn đề—
bạn nghĩ đầu ra sẽ là gì?

<?code-excerpt "async_await/bin/get_order_sync_bad.dart" remove="Fetching"?>
```dartpad
// Ví dụ này trình bày cách *không nên* viết code Dart bất đồng bộ.

String createOrderMessage() {
  var order = fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    // Hãy hình dung hàm này phức tạp và chậm hơn.
    Future.delayed(const Duration(seconds: 2), () => 'Large Latte');

void main() {
  print(createOrderMessage());
}
```

Đây là lý do ví dụ không in được value
mà `fetchUserOrder()` cuối cùng tạo ra:

* `fetchUserOrder()` là hàm bất đồng bộ mà sau một khoảng trì hoãn,
  cung cấp string mô tả đơn hàng của người dùng: một "Large Latte".
* Để lấy đơn hàng, `createOrderMessage()` phải gọi `fetchUserOrder()`
  và chờ nó hoàn tất. Vì `createOrderMessage()` *không* chờ
  `fetchUserOrder()` hoàn tất, `createOrderMessage()` không
  lấy được value string mà `fetchUserOrder()` cuối cùng cung cấp.
* Thay vào đó, `createOrderMessage()` nhận biểu diễn của công việc đang chờ
  được thực hiện: một future chưa hoàn thành.
  Bạn sẽ tìm hiểu thêm về future trong phần tiếp theo.
* Vì `createOrderMessage()` không lấy được value mô tả đơn hàng,
  ví dụ không in "Large Latte" ra console mà thay vào đó
  in "Your order is: Instance of '_Future\<String\>'".

Trong các phần tiếp theo, bạn sẽ tìm hiểu về future và cách làm việc với future
(bằng `async` và `await`)
để có thể viết code cần thiết giúp `fetchUserOrder()`
in value mong muốn ("Large Latte") ra console.

:::secondary Thuật ngữ chính
* **Thao tác đồng bộ**: Thao tác đồng bộ chặn thao tác khác
  thực thi cho đến khi nó hoàn tất.
* **Hàm đồng bộ**: Hàm đồng bộ chỉ thực hiện thao tác đồng bộ.
* **Thao tác bất đồng bộ**: Sau khi bắt đầu, thao tác bất đồng bộ cho phép
  thao tác khác thực thi trước khi nó hoàn tất.
* **Hàm bất đồng bộ**: Hàm bất đồng bộ thực hiện ít nhất một
  thao tác bất đồng bộ và cũng có thể thực hiện thao tác _đồng bộ_.
:::


<a id="what-is-a-future"></a>
## Future là gì?

Một future (chữ "f" thường) là instance
của class [Future][] (chữ "F" hoa).
Future biểu diễn kết quả của thao tác bất đồng bộ
và có thể ở hai trạng thái: chưa hoàn thành hoặc đã hoàn thành.

:::note
_Uncompleted_ (chưa hoàn thành) là thuật ngữ Dart chỉ trạng thái của future
trước khi nó tạo ra value.
:::

<a id="uncompleted"></a>
### Chưa hoàn thành

Khi gọi hàm bất đồng bộ, nó trả về future chưa hoàn thành.
Future đó đang chờ thao tác bất đồng bộ của hàm
kết thúc hoặc throw error.

<a id="completed"></a>
### Đã hoàn thành

Nếu thao tác bất đồng bộ thành công,
future hoàn thành với value.
Nếu không, nó hoàn thành với error.

<a id="completing-with-a-value"></a>
#### Hoàn thành với value

Future kiểu `Future<T>` hoàn thành với value kiểu `T`.
Ví dụ, future kiểu `Future<String>` tạo ra value string.
Nếu future không tạo ra value có thể sử dụng,
kiểu của future là `Future<void>`.

<a id="completing-with-an-error"></a>
#### Hoàn thành với error

Nếu thao tác bất đồng bộ do hàm thực hiện thất bại vì bất kỳ lý do nào,
future hoàn thành với error.

<a id="example-introducing-futures"></a>
### Ví dụ: Giới thiệu future

Trong ví dụ sau, `fetchUserOrder()` trả về future
hoàn thành sau khi in ra console.
Vì không trả về value có thể sử dụng,
`fetchUserOrder()` có kiểu `Future<void>`.
Trước khi chạy ví dụ,
hãy thử dự đoán nội dung nào sẽ in trước:
"Large Latte" hay "Fetching user order...".

<?code-excerpt "async_await/bin/futures_intro.dart (no-error)"?>
```dartpad
Future<void> fetchUserOrder() {
  // Hãy hình dung hàm này đang lấy thông tin người dùng từ service hoặc database khác.
  return Future.delayed(const Duration(seconds: 2), () => print('Large Latte'));
}

void main() {
  fetchUserOrder();
  print('Fetching user order...');
}
```

Trong ví dụ trên,
dù `fetchUserOrder()` thực thi trước lệnh gọi `print()` ở dòng 8,
console hiển thị đầu ra dòng 8 ("Fetching user order...")
trước đầu ra từ `fetchUserOrder()` ("Large Latte").
Đó là vì `fetchUserOrder()` trì hoãn trước khi in "Large Latte".

<a id="example-completing-with-an-error"></a>
### Ví dụ: Hoàn thành với error

Chạy ví dụ sau để xem future hoàn thành với error như thế nào.
Một chút nữa bạn sẽ học cách xử lý error.

<?code-excerpt "async_await/bin/futures_intro.dart (error)" replace="/Error//g"?>
```dartpad
Future<void> fetchUserOrder() {
  // Hãy hình dung hàm này đang lấy thông tin người dùng nhưng gặp bug.
  return Future.delayed(
    const Duration(seconds: 2),
    () => throw Exception('Logout failed: user ID is invalid'),
  );
}

void main() {
  fetchUserOrder();
  print('Fetching user order...');
}
```

Trong ví dụ này, `fetchUserOrder()` hoàn thành
với error cho biết user ID không hợp lệ.

Bạn đã tìm hiểu future và cách chúng hoàn thành,
nhưng làm thế nào để dùng kết quả của hàm bất đồng bộ?
Trong phần tiếp theo, bạn sẽ học cách lấy kết quả
bằng keyword `async` và `await`.

:::secondary Ôn tập nhanh
* Instance [Future\<T\>][Future] tạo ra value kiểu `T`.
* Nếu future không tạo ra value có thể sử dụng,
  kiểu của future là `Future<void>`.
* Future có thể ở một trong hai trạng thái: chưa hoàn thành hoặc đã hoàn thành.
* Khi gọi hàm trả về future,
  hàm xếp hàng công việc cần làm và trả về future chưa hoàn thành.
* Khi thao tác của future kết thúc,
  future hoàn thành với value hoặc error.

**Thuật ngữ chính:**

* **Future**: class [Future][] của Dart.
* **future**: instance của class `Future` trong Dart.
:::

<a id="working-with-futures-async-and-await"></a>
## Làm việc với future: async và await

Các keyword `async`, `await` cung cấp cách khai báo
để định nghĩa hàm bất đồng bộ và dùng kết quả của chúng.
Hãy nhớ hai hướng dẫn cơ bản khi dùng `async`, `await`:

* __Để định nghĩa hàm async, thêm `async` trước thân hàm:__
* __Keyword `await` chỉ hoạt động trong hàm `async`.__

Đây là ví dụ chuyển đổi `main()`
từ hàm đồng bộ thành bất đồng bộ.

Trước tiên, thêm keyword `async` trước thân hàm:

<?code-excerpt "async_await/bin/get_order_sync_bad.dart (main-sig)" replace="/main\(\)/$& async/g; /async/[!$&!]/g; /$/ ··· }/g"?>
```dart
void main() [!async!] { ··· }
```

Nếu hàm có return type được khai báo,
hãy cập nhật kiểu thành `Future<T>`,
trong đó `T` là kiểu value mà hàm trả về.
Nếu hàm không trả về rõ ràng một value,
return type là `Future<void>`:

<?code-excerpt "async_await/bin/get_order.dart (main-sig)" replace="/Future<\w+\W/[!$&!]/g;  /$/ ··· }/g"?>
```dart
[!Future<void>!] main() async { ··· }
```

Giờ bạn có hàm `async`,
bạn có thể dùng keyword `await` để chờ future hoàn thành:

<?code-excerpt "async_await/bin/get_order.dart (print-order)" replace="/await/[!$&!]/g"?>
```dart
print([!await!] createOrderMessage());
```

Như hai ví dụ sau trình bày, các keyword `async`, `await`
tạo ra code bất đồng bộ trông rất giống code đồng bộ.
Những khác biệt duy nhất được tô sáng trong ví dụ bất đồng bộ,
nằm bên phải ví dụ đồng bộ nếu cửa sổ đủ rộng.

<a id="example-synchronous-functions"></a>
#### Ví dụ: Hàm đồng bộ

<?code-excerpt "async_await/bin/get_order_sync_bad.dart (no-warning)" replace="/(\s+\/\/ )(Imagine.*? is )(.*)/$1$2$1$3/g"?>
```dart
String createOrderMessage() {
  var order = fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    // Hãy hình dung hàm này
    // phức tạp và chậm hơn.
    Future.delayed(const Duration(seconds: 2), () => 'Large Latte');

void main() {
  print('Fetching user order...');
  print(createOrderMessage());
}
```

```plaintext
Fetching user order...
Your order is: Instance of 'Future<String>'
```

Như hai ví dụ sau trình bày,
nó hoạt động giống code đồng bộ.

<a id="example-asynchronous-functions"></a>
#### Ví dụ: Hàm bất đồng bộ

<?code-excerpt "async_await/bin/get_order.dart" replace="/(\s+\/\/ )(Imagine.*? is )(.*)/$1$2$1$3/g; /async|await/[!$&!]/g; /(Future<\w+\W)( [^f])/[!$1!]$2/g; /4/2/g"?>
```dart
[!Future<String>!] createOrderMessage() [!async!] {
  var order = [!await!] fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    // Hãy hình dung hàm này
    // phức tạp và chậm hơn.
    Future.delayed(const Duration(seconds: 2), () => 'Large Latte');

[!Future<void>!] main() [!async!] {
  print('Fetching user order...');
  print([!await!] createOrderMessage());
}
```

```plaintext
Fetching user order...
Your order is: Large Latte
```

Ví dụ bất đồng bộ khác ở ba điểm:

* Return type của `createOrderMessage()` thay đổi
  từ `String` thành `Future<String>`.
* Keyword **`async`** xuất hiện trước thân hàm của
  `createOrderMessage()` và `main()`.
* Keyword **`await`** xuất hiện trước khi gọi các hàm bất đồng bộ
  `fetchUserOrder()` và `createOrderMessage()`.

:::secondary Thuật ngữ chính
* **async**: Bạn có thể dùng keyword `async` trước thân hàm để đánh dấu nó là
  bất đồng bộ.
* **Hàm async**: Hàm `async` là hàm được gắn nhãn bằng keyword `async`.
* **await**: Bạn có thể dùng keyword `await` để lấy kết quả hoàn thành của
  biểu thức bất đồng bộ. Keyword `await` chỉ hoạt động trong hàm `async`.
:::

<a id="execution-flow-with-async-and-await"></a>
### Luồng thực thi với async và await

Hàm `async` chạy đồng bộ cho đến keyword `await` đầu tiên.
Điều này nghĩa là trong thân hàm `async`,
mọi code đồng bộ trước keyword `await` đầu tiên thực thi ngay lập tức.

<a id="example-execution-within-async-functions"></a>
### Ví dụ: Thực thi bên trong hàm async

Chạy ví dụ sau để xem việc thực thi diễn ra
trong thân hàm `async` như thế nào.
Bạn nghĩ đầu ra sẽ là gì?

<?code-excerpt "async_await/bin/async_example.dart" remove="/\/\/ print/"?>
```dartpad
Future<void> printOrderMessage() async {
  print('Awaiting user order...');
  var order = await fetchUserOrder();
  print('Your order is: $order');
}

Future<String> fetchUserOrder() {
  // Hãy hình dung hàm này phức tạp và chậm hơn.
  return Future.delayed(const Duration(seconds: 4), () => 'Large Latte');
}

void main() async {
  countSeconds(4);
  await printOrderMessage();
}

// Bạn có thể bỏ qua hàm này—nó ở đây để trực quan hóa thời gian trì hoãn trong ví dụ.
void countSeconds(int s) {
  for (var i = 1; i <= s; i++) {
    Future.delayed(Duration(seconds: i), () => print(i));
  }
}
```

Sau khi chạy code trong ví dụ trên, hãy thử đảo dòng 2 và 3:

<?code-excerpt "async_await/bin/async_example.dart (swap-stmts)" replace="/\/\/ (print)/$1/g"?>
```dart
var order = await fetchUserOrder();
print('Awaiting user order...');
```

Lưu ý thời điểm đầu ra thay đổi vì `print('Awaiting user order')`
giờ xuất hiện sau keyword `await` đầu tiên trong `printOrderMessage()`.

<a id="exercise-practice-using-async-and-await"></a>
### Bài tập: Thực hành sử dụng async và await

Bài tập sau là unit test đang thất bại
chứa các snippet code được hoàn thành một phần.
Nhiệm vụ của bạn là hoàn thành bài tập bằng cách viết code để test pass.
Bạn không cần implement `main()`.

Để mô phỏng thao tác bất đồng bộ, hãy gọi các hàm sau
được cung cấp cho bạn:

| Hàm               | Type signature                   | Mô tả                                       |
|--------------------|----------------------------------|---------------------------------------------|
| fetchRole()        | `Future<String> fetchRole()`     | Lấy mô tả ngắn về vai trò của người dùng.   |
| fetchLoginAmount() | `Future<int> fetchLoginAmount()` | Lấy số lần người dùng đã đăng nhập.          |

{:.table .table-striped}

<a id="part-1-reportuserrole"></a>
#### Phần 1: `reportUserRole()`

Thêm code vào hàm `reportUserRole()` để nó thực hiện các việc sau:

* Trả về future hoàn thành với string sau:
  `"User role: <user role>"`
  * Lưu ý: Bạn phải dùng value thực tế do `fetchRole()` trả về;
    sao chép và dán return value mẫu sẽ không làm test pass.
  * Return value mẫu: `"User role: tester"`
* Lấy vai trò người dùng bằng cách gọi hàm `fetchRole()` được cung cấp.

<a id="part-2-reportlogins"></a>
#### Phần 2: `reportLogins()`

Implement hàm `async` `reportLogins()` để nó thực hiện các việc sau:

* Trả về string `"Total number of logins: <# of logins>"`.
  * Lưu ý: Bạn phải dùng value thực tế do `fetchLoginAmount()` trả về;
    sao chép và dán return value mẫu sẽ không làm test pass.
  * Return value mẫu từ `reportLogins()`: `"Total number of logins: 57"`
* Lấy số lần đăng nhập bằng cách gọi hàm `fetchLoginAmount()` được cung cấp.

```dartpad theme="dark"
// Phần 1
// Gọi hàm async fetchRole() được cung cấp
// để trả về vai trò người dùng.
Future<String> reportUserRole() async {
  // TODO: Implement hàm reportUserRole ở đây.
}

// Phần 2
// TODO: Implement hàm reportLogins ở đây.
// Gọi hàm async fetchLoginAmount() được cung cấp
// để trả về số lần người dùng đã đăng nhập.
reportLogins() {}

// Các hàm sau được cung cấp cho bạn để mô phỏng
// những thao tác bất đồng bộ có thể mất một lúc.

Future<String> fetchRole() => Future.delayed(_halfSecond, () => _role);
Future<int> fetchLoginAmount() => Future.delayed(_halfSecond, () => _logins);

// Code sau được dùng để kiểm thử và cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.

void main() async {
  print('Testing...');
  List<String> messages = [];
  const passed = 'PASSED';
  const testFailedMessage = 'Test failed for the function:';
  const typoMessage = 'Test failed! Check for typos in your return value';
  try {
    messages
      ..add(_makeReadable(
          testLabel: 'Part 1',
          testResult: await _asyncEquals(
            expected: 'User role: administrator',
            actual: await reportUserRole(),
            typoKeyword: _role,
          ),
          readableErrors: {
            typoMessage: typoMessage,
            'null':
                'Test failed! Did you forget to implement or return from reportUserRole?',
            'User role: Instance of \'Future<String>\'':
                '$testFailedMessage reportUserRole. Did you use the await keyword?',
            'User role: Instance of \'_Future<String>\'':
                '$testFailedMessage reportUserRole. Did you use the await keyword?',
            'User role:':
                '$testFailedMessage reportUserRole. Did you return a user role?',
            'User role: ':
                '$testFailedMessage reportUserRole. Did you return a user role?',
            'User role: tester':
                '$testFailedMessage reportUserRole. Did you invoke fetchRole to fetch the user\'s role?',
          }))
      ..add(_makeReadable(
          testLabel: 'Part 2',
          testResult: await _asyncEquals(
            expected: 'Total number of logins: 42',
            actual: await reportLogins(),
            typoKeyword: _logins.toString(),
          ),
          readableErrors: {
            typoMessage: typoMessage,
            'null':
                'Test failed! Did you forget to implement or return from reportLogins?',
            'Total number of logins: Instance of \'Future<int>\'':
                '$testFailedMessage reportLogins. Did you use the await keyword?',
            'Total number of logins: Instance of \'_Future<int>\'':
                '$testFailedMessage reportLogins. Did you use the await keyword?',
            'Total number of logins: ':
                '$testFailedMessage reportLogins. Did you return the number of logins?',
            'Total number of logins:':
                '$testFailedMessage reportLogins. Did you return the number of logins?',
            'Total number of logins: 57':
                '$testFailedMessage reportLogins. Did you invoke fetchLoginAmount to fetch the number of user logins?',
          }))
      ..removeWhere((m) => m.contains(passed))
      ..toList();

    if (messages.isEmpty) {
      print('Success. All tests passed!');
    } else {
      messages.forEach(print);
    }
  } on UnimplementedError {
    print(
        'Test failed! Did you forget to implement or return from reportUserRole?');
  } catch (e) {
    print('Tried to run solution, but received an exception: $e');
  }
}

const _role = 'administrator';
const _logins = 42;
const _halfSecond = Duration(milliseconds: 500);

// Trình hỗ trợ test.
String _makeReadable({
  required String testResult,
  required Map<String, String> readableErrors,
  required String testLabel,
}) {
  if (readableErrors.containsKey(testResult)) {
    var readable = readableErrors[testResult];
    return '$testLabel $readable';
  } else {
    return '$testLabel $testResult';
  }
}

// Assertion dùng trong test.
Future<String> _asyncEquals({
  required String expected,
  required dynamic actual,
  required String typoKeyword,
}) async {
  var strActual = actual is String ? actual : actual.toString();
  try {
    if (expected == actual) {
      return 'PASSED';
    } else if (strActual.contains(typoKeyword)) {
      return 'Test failed! Check for typos in your return value';
    } else {
      return strActual;
    }
  } catch (e) {
    return e.toString();
  }
}
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập async-await.">Gợi ý</summary>

  Bạn có nhớ thêm keyword `async` vào hàm `reportUserRole` không?

  Bạn có nhớ dùng keyword `await` trước khi gọi `fetchRole()` không?

  Hãy nhớ: `reportUserRole` cần trả về `Future`.

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập async-await.">Lời giải</summary>

  ```dart
  Future<String> reportUserRole() async {
    final username = await fetchRole();
    return 'User role: $username';
  }
  
  Future<String> reportLogins() async {
    final logins = await fetchLoginAmount();
    return 'Total number of logins: $logins';
  }
  ```

</details>

<a id="handling-errors"></a>
## Xử lý error

Để xử lý error trong hàm `async`, hãy dùng try-catch:

<?code-excerpt "async_await/bin/try_catch.dart (try-catch)" remove="print(order)"?>
```dart
try {
  print('Awaiting user order...');
  var order = await fetchUserOrder();
} catch (err) {
  print('Caught error: $err');
}
```

Trong hàm `async`, bạn có thể viết
[mệnh đề try-catch](/language/error-handling#catch)
giống như trong code đồng bộ.

<a id="example-async-and-await-with-try-catch"></a>
### Ví dụ: async và await với try-catch

Chạy ví dụ sau để xem cách xử lý error
từ hàm bất đồng bộ.
Bạn nghĩ đầu ra sẽ là gì?

<?code-excerpt "async_await/bin/try_catch.dart"?>
```dartpad
Future<void> printOrderMessage() async {
  try {
    print('Awaiting user order...');
    var order = await fetchUserOrder();
    print(order);
  } catch (err) {
    print('Caught error: $err');
  }
}

Future<String> fetchUserOrder() {
  // Hãy hình dung hàm này phức tạp hơn.
  var str = Future.delayed(
    const Duration(seconds: 4),
    () => throw 'Cannot locate user order',
  );
  return str;
}

void main() async {
  await printOrderMessage();
}
```

<a id="exercise-practice-handling-errors"></a>
### Bài tập: Thực hành xử lý error

Bài tập sau giúp thực hành xử lý error với code bất đồng bộ
bằng cách tiếp cận ở phần trước. Để mô phỏng thao tác bất đồng bộ,
code sẽ gọi hàm sau được cung cấp cho bạn:

| Hàm                | Type signature                      | Mô tả                                                         |
|---------------------|-------------------------------------|---------------------------------------------------------------|
| fetchNewUsername()  | `Future<String> fetchNewUsername()` | Trả về username mới mà bạn có thể dùng để thay username cũ.   |

{:.table .table-striped}

Dùng `async` và `await` để implement hàm `changeUsername()` bất đồng bộ
thực hiện các việc sau:

* Gọi hàm bất đồng bộ `fetchNewUsername()` được cung cấp
  và trả về kết quả.
  * Return value mẫu từ `changeUsername()`: `"jane_smith_92"`
* Bắt mọi error xảy ra và trả về value string của error.
  * Bạn có thể dùng method
    [toString()]({{site.dart-api}}/dart-core/ArgumentError/toString.html)
    để string hóa cả
    [Exception]({{site.dart-api}}/dart-core/Exception-class.html)
    và
    [Error.]({{site.dart-api}}/dart-core/Error-class.html)

```dartpad theme="dark"
// TODO: Implement changeUsername ở đây.
changeUsername() {}

// Hàm sau được cung cấp để mô phỏng
// thao tác bất đồng bộ có thể mất một lúc và
// có khả năng throw exception.

Future<String> fetchNewUsername() =>
    Future.delayed(const Duration(milliseconds: 500), () => throw UserError());

class UserError implements Exception {
  @override
  String toString() => 'New username is invalid';
}

// Code sau được dùng để kiểm thử và cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.

void main() async {
  final List<String> messages = [];
  const typoMessage = 'Test failed! Check for typos in your return value';

  print('Testing...');
  try {
    messages
      ..add(_makeReadable(
          testLabel: '',
          testResult: await _asyncDidCatchException(changeUsername),
          readableErrors: {
            typoMessage: typoMessage,
            _noCatch:
                'Did you remember to call fetchNewUsername within a try/catch block?',
          }))
      ..add(_makeReadable(
          testLabel: '',
          testResult: await _asyncErrorEquals(changeUsername),
          readableErrors: {
            typoMessage: typoMessage,
            _noCatch:
                'Did you remember to call fetchNewUsername within a try/catch block?',
          }))
      ..removeWhere((m) => m.contains(_passed))
      ..toList();

    if (messages.isEmpty) {
      print('Success. All tests passed!');
    } else {
      messages.forEach(print);
    }
  } catch (e) {
    print('Tried to run solution, but received an exception: $e');
  }
}

// Trình hỗ trợ test.
String _makeReadable({
  required String testResult,
  required Map<String, String> readableErrors,
  required String testLabel,
}) {
  if (readableErrors.containsKey(testResult)) {
    final readable = readableErrors[testResult];
    return '$testLabel $readable';
  } else {
    return '$testLabel $testResult';
  }
}

Future<String> _asyncErrorEquals(Function fn) async {
  final result = await fn();
  if (result == UserError().toString()) {
    return _passed;
  } else {
    return 'Test failed! Did you stringify and return the caught error?';
  }
}

Future<String> _asyncDidCatchException(Function fn) async {
  var caught = true;
  try {
    await fn();
  } on UserError catch (_) {
    caught = false;
  }

  if (caught == false) {
    return _noCatch;
  } else {
    return _passed;
  }
}

const _passed = 'PASSED';
const _noCatch = 'NO_CATCH';
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập xử lý error.">Gợi ý</summary>

  Implement `changeUsername` để trả về string từ `fetchNewUsername`, hoặc
  nếu thất bại, trả về value string của bất kỳ error nào xảy ra.

  Hãy nhớ: Bạn có thể dùng [câu lệnh try-catch](/language/error-handling#catch)
  để bắt và xử lý error.

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập xử lý error.">Lời giải</summary>

  ```dart
  Future<String> changeUsername() async {
    try {
      return await fetchNewUsername();
    } catch (err) {
      return err.toString();
    }
  }
  ```

</details>

<a id="exercise-putting-it-all-together"></a>
## Bài tập: Kết hợp tất cả

Đã đến lúc thực hành những gì bạn học được trong bài tập cuối.
Để mô phỏng thao tác bất đồng bộ, bài tập cung cấp các hàm bất đồng bộ
`fetchUsername()` và `logoutUser()`:

| Hàm             | Type signature                   | Mô tả                                                                      |
|------------------|----------------------------------|----------------------------------------------------------------------------|
| fetchUsername()  | `Future<String> fetchUsername()` | Trả về tên gắn với người dùng hiện tại.                                    |
| logoutUser()     | `Future<String> logoutUser()`    | Đăng xuất người dùng hiện tại và trả về username đã đăng xuất.              |

{:.table .table-striped}

Viết nội dung sau:

<a id="part-1-addhello"></a>
#### Phần 1: `addHello()`

* Viết hàm `addHello()` nhận một argument `String` duy nhất.
* `addHello()` trả về argument `String` với `'Hello '` ở trước.<br>
  Ví dụ: `addHello('Jon')` trả về `'Hello Jon'`.

<a id="part-2-greetuser"></a>
#### Phần 2: `greetUser()`

* Viết hàm `greetUser()` không nhận argument.
* Để lấy username, `greetUser()` gọi hàm bất đồng bộ
  `fetchUsername()` được cung cấp.
* `greetUser()` tạo lời chào cho người dùng bằng cách gọi `addHello()`,
  truyền username vào và trả về kết quả.<br>
  Ví dụ: Nếu `fetchUsername()` trả về `'Jenny'`,
  `greetUser()` trả về `'Hello Jenny'`.

<a id="part-3-saygoodbye"></a>
#### Phần 3: `sayGoodbye()`

* Viết hàm `sayGoodbye()` thực hiện:
  * Không nhận argument.
  * Bắt mọi error.
  * Gọi hàm bất đồng bộ `logoutUser()` được cung cấp.
* Nếu `logoutUser()` thất bại, `sayGoodbye()` trả về string bất kỳ bạn muốn.
* Nếu `logoutUser()` thành công, `sayGoodbye()` trả về string
  `'<result> Thanks, see you next time'`, trong đó `<result>` là
  value string do `logoutUser()` trả về.

```dartpad theme="dark"
// Phần 1
addHello(String user) {}

// Phần 2
// Gọi hàm async fetchUsername() được cung cấp
// để trả về username.
greetUser() {}

// Phần 3
// Gọi hàm async logoutUser() được cung cấp
// để đăng xuất người dùng.
sayGoodbye() {}

// Các hàm sau được cung cấp để bạn dùng trong lời giải.

Future<String> fetchUsername() => Future.delayed(_halfSecond, () => 'Jean');

Future<String> logoutUser() => Future.delayed(_halfSecond, _failOnce);

// Code sau được dùng để kiểm thử và cung cấp phản hồi về lời giải của bạn.
// Không cần đọc hoặc sửa đổi nó.

void main() async {
  const didNotImplement =
      'Test failed! Did you forget to implement or return from';

  final List<String> messages = [];

  print('Testing...');
  try {
    messages
      ..add(_makeReadable(
          testLabel: 'Part 1',
          testResult: await _asyncEquals(
              expected: 'Hello Jerry',
              actual: addHello('Jerry'),
              typoKeyword: 'Jerry'),
          readableErrors: {
            _typoMessage: _typoMessage,
            'null': '$didNotImplement addHello?',
            'Hello Instance of \'Future<String>\'':
                'Looks like you forgot to use the \'await\' keyword!',
            'Hello Instance of \'_Future<String>\'':
                'Looks like you forgot to use the \'await\' keyword!',
          }))
      ..add(_makeReadable(
          testLabel: 'Part 2',
          testResult: await _asyncEquals(
              expected: 'Hello Jean',
              actual: await greetUser(),
              typoKeyword: 'Jean'),
          readableErrors: {
            _typoMessage: _typoMessage,
            'null': '$didNotImplement greetUser?',
            'HelloJean':
                'Looks like you forgot the space between \'Hello\' and \'Jean\'',
            'Hello Instance of \'Future<String>\'':
                'Looks like you forgot to use the \'await\' keyword!',
            'Hello Instance of \'_Future<String>\'':
                'Looks like you forgot to use the \'await\' keyword!',
            '{Closure: (String) => dynamic from Function \'addHello\': static.(await fetchUsername())}':
                'Did you place the \'\$\' character correctly?',
            '{Closure \'addHello\'(await fetchUsername())}':
                'Did you place the \'\$\' character correctly?',
          }))
      ..add(_makeReadable(
          testLabel: 'Part 3',
          testResult: await _asyncDidCatchException(sayGoodbye),
          readableErrors: {
            _typoMessage:
                '$_typoMessage. Did you add the text \'Thanks, see you next time\'?',
            'null': '$didNotImplement sayGoodbye?',
            _noCatch:
                'Did you remember to call logoutUser within a try/catch block?',
            'Instance of \'Future<String>\' Thanks, see you next time':
                'Did you remember to use the \'await\' keyword in the sayGoodbye function?',
            'Instance of \'_Future<String>\' Thanks, see you next time':
                'Did you remember to use the \'await\' keyword in the sayGoodbye function?',
          }))
      ..add(_makeReadable(
          testLabel: 'Part 3',
          testResult: await _asyncEquals(
              expected: 'Success! Thanks, see you next time',
              actual: await sayGoodbye(),
              typoKeyword: 'Success'),
          readableErrors: {
            _typoMessage:
                '$_typoMessage. Did you add the text \'Thanks, see you next time\'?',
            'null': '$didNotImplement sayGoodbye?',
            _noCatch:
                'Did you remember to call logoutUser within a try/catch block?',
            'Instance of \'Future<String>\' Thanks, see you next time':
                'Did you remember to use the \'await\' keyword in the sayGoodbye function?',
            'Instance of \'_Future<String>\' Thanks, see you next time':
                'Did you remember to use the \'await\' keyword in the sayGoodbye function?',
            'Instance of \'_Exception\'':
                'CAUGHT Did you remember to return a string?',
          }))
      ..removeWhere((m) => m.contains(_passed))
      ..toList();

    if (messages.isEmpty) {
      print('Success. All tests passed!');
    } else {
      messages.forEach(print);
    }
  } catch (e) {
    print('Tried to run solution, but received an exception: $e');
  }
}

// Trình hỗ trợ test.
String _makeReadable({
  required String testResult,
  required Map<String, String> readableErrors,
  required String testLabel,
}) {
  String? readable;
  if (readableErrors.containsKey(testResult)) {
    readable = readableErrors[testResult];
    return '$testLabel $readable';
  } else if ((testResult != _passed) && (testResult.length < 18)) {
    readable = _typoMessage;
    return '$testLabel $readable';
  } else {
    return '$testLabel $testResult';
  }
}

Future<String> _asyncEquals({
  required String expected,
  required dynamic actual,
  required String typoKeyword,
}) async {
  final strActual = actual is String ? actual : actual.toString();
  try {
    if (expected == actual) {
      return _passed;
    } else if (strActual.contains(typoKeyword)) {
      return _typoMessage;
    } else {
      return strActual;
    }
  } catch (e) {
    return e.toString();
  }
}

Future<String> _asyncDidCatchException(Function fn) async {
  var caught = true;
  try {
    await fn();
  } on Exception catch (_) {
    caught = false;
  }

  if (caught == true) {
    return _passed;
  } else {
    return _noCatch;
  }
}

const _typoMessage = 'Test failed! Check for typos in your return value';
const _passed = 'PASSED';
const _noCatch = 'NO_CATCH';
const _halfSecond = Duration(milliseconds: 500);

String _failOnce() {
  if (_logoutSucceeds) {
    return 'Success!';
  } else {
    _logoutSucceeds = true;
    throw Exception('Logout failed');
  }
}

bool _logoutSucceeds = false;
```

<details>
  <summary title="Mở rộng để xem gợi ý cho bài tập 'Kết hợp tất cả'.">Gợi ý</summary>

  Các hàm `greetUser` và `sayGoodbye` phải bất đồng bộ,
  còn `addHello` phải là hàm đồng bộ thông thường.

  Hãy nhớ: Bạn có thể dùng [câu lệnh try-catch](/language/error-handling#catch)
  để bắt và xử lý error.

</details>

<details>
  <summary title="Mở rộng để xem lời giải của bài tập 'Kết hợp tất cả'.">Lời giải</summary>

  ```dart
  String addHello(String user) => 'Hello $user';
  
  Future<String> greetUser() async {
    final username = await fetchUsername();
    return addHello(username);
  }
  
  Future<String> sayGoodbye() async {
    try {
      final result = await logoutUser();
      return '$result Thanks, see you next time';
    } catch (e) {
      return 'Failed to logout user: $e';
    }
  }
  ```

</details>

<a id="which-lints-work-for-futures"></a>
## Những lint nào hoạt động cho future?

Để bắt lỗi phổ biến phát sinh khi làm việc với async và future,
hãy [bật](/tools/analysis#individual-rules) các lint sau:

* [`discarded_futures`][]
* [`unawaited_futures`][]

[`discarded_futures`]: /tools/linter-rules/discarded_futures
[`unawaited_futures`]: /tools/linter-rules/unawaited_futures

<a id="whats-next"></a>
## Tiếp theo là gì?

Chúc mừng, bạn đã hoàn thành hướng dẫn! Nếu muốn tìm hiểu thêm, đây là
một số gợi ý cho bước tiếp theo:

- Thử nghiệm với [DartPad]({{site.dartpad}}).
- Tìm hiểu thêm về future và code bất đồng bộ trong Dart:
  - [Hướng dẫn về Stream](/libraries/async/using-streams):
    Tìm hiểu cách làm việc với chuỗi event bất đồng bộ.
  - [Concurrency trong Dart](/language/concurrency):
    Hiểu và học cách implement concurrency trong Dart.
  - [Lập trình bất đồng bộ](/language/async):
    Tìm hiểu sâu hỗ trợ ngôn ngữ và thư viện của Dart cho code bất đồng bộ.
  - [Video Dart từ Google][Dart videos]:
    Xem một hoặc nhiều video về code bất đồng bộ.
- Tải [Dart SDK](/get-dart)!

[Dart videos]: {{site.yt.playlist}}PLjxrf2q8roU0Net_g1NT5_vOO3s_FR02J
[Future]: {{site.dart-api}}/dart-async/Future-class.html
[style guide]: /effective-dart/style
[documentation guide]: /effective-dart/documentation
[usage guide]: /effective-dart/usage
[design guide]: /effective-dart/design
