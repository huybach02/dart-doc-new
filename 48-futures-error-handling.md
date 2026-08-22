---
title: Future và xử lý lỗi
description: >-
  Mọi điều bạn muốn biết về cách xử lý error và exception
  khi viết code bất đồng bộ. Và còn nhiều hơn thế nữa.
---

Ngôn ngữ Dart có
[hỗ trợ bất đồng bộ](/language/async) native,
giúp code Dart bất đồng bộ dễ đọc và dễ viết hơn nhiều.
Tuy nhiên, một số code—đặc biệt là code cũ—vẫn có thể sử dụng
[các method của Future][Future class]
như `then()`, `catchError()` và `whenComplete()`.

Trang này có thể giúp bạn tránh một số cạm bẫy phổ biến
khi sử dụng các method Future đó.

:::warning
Bạn không cần trang này nếu code của bạn sử dụng
hỗ trợ bất đồng bộ của ngôn ngữ:
`async`, `await` và xử lý lỗi bằng try-catch.
Để biết thêm thông tin, hãy xem
[hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await).
:::


<a id="the-future-api-and-callbacks"></a>
## Future API và callback

Các hàm sử dụng Future API đăng ký callback để xử lý
value (hoặc error) hoàn thành một Future. Ví dụ:

<?code-excerpt "futures/lib/simple.dart (then-catch)"?>
```dart
myFunc().then(processValue).catchError(handleError);
```

Các callback đã đăng ký được kích hoạt theo những quy tắc sau: callback của `then()`
được kích hoạt nếu nó được gọi trên một Future hoàn thành với value;
callback của `catchError()` được kích hoạt nếu nó được gọi trên một Future hoàn thành
với error.

Trong ví dụ trên, nếu Future của `myFunc()` hoàn thành với value,
callback của `then()` sẽ được kích hoạt. Nếu không có error mới nào được tạo ra trong `then()`,
callback của `catchError()` sẽ không được kích hoạt. Mặt khác, nếu `myFunc()`
hoàn thành với error, callback của `then()` không được kích hoạt và
callback của `catchError()` được kích hoạt.

<a id="examples-of-using-then-with-catcherror"></a>
## Ví dụ sử dụng then() với catchError()

Các lệnh gọi `then()` và `catchError()` được nối chuỗi là một pattern phổ biến khi
làm việc với Future và có thể được xem là tương đương gần đúng với
block try-catch.

Một vài phần tiếp theo đưa ra ví dụ về pattern này.

<a id="catcherror-as-a-comprehensive-error-handler"></a>
### catchError() như một trình xử lý lỗi toàn diện

Ví dụ sau xử lý việc throw exception từ bên trong callback của `then()`
và minh họa tính linh hoạt của `catchError()` với vai trò trình xử lý lỗi:

<?code-excerpt "futures/lib/simple.dart (comprehensive-errors)" replace="/ellipsis\(\);/.../g;"?>
```dart
myFunc()
    .then((value) {
      doSomethingWith(value);
      ...
      throw Exception('Some arbitrary error');
    })
    .catchError(handleError);
```

Nếu Future của `myFunc()` hoàn thành với value, callback của `then()` sẽ được kích hoạt. Nếu
code trong callback của `then()` throw (như trong ví dụ trên),
Future của `then()` hoàn thành với error. Error đó được
`catchError()` xử lý.

Nếu Future của `myFunc()` hoàn thành với error, Future của `then()` hoàn thành
với error đó. Error cũng được `catchError()` xử lý.

Bất kể error bắt nguồn từ bên trong `myFunc()` hay bên trong
`then()`, `catchError()` đều xử lý thành công.

<a id="error-handling-within-then"></a>
### Xử lý lỗi bên trong then()

Để xử lý lỗi chi tiết hơn, bạn có thể đăng ký callback thứ hai (`onError`)
bên trong `then()` để xử lý những Future hoàn thành với error. Đây là
signature của `then()`:

<?code-excerpt "futures/lib/simple.dart (future-then)"?>
```dart
Future<R> then<R>(FutureOr<R> Function(T value) onValue, {Function? onError});
```

Chỉ đăng ký callback onError tùy chọn nếu bạn muốn phân biệt
giữa error được chuyển tiếp _đến_ `then()` và error được tạo ra _bên trong_
`then()`:

<?code-excerpt "futures/lib/simple.dart (throws-then-catch)"?>
```dart
asyncErrorFunction()
    .then(
      successCallback,
      onError: (e) {
        handleError(e); // Error ban đầu.
        anotherAsyncErrorFunction(); // Ôi, error mới.
      },
    )
    .catchError(handleError); // Error từ bên trong then() đã được xử lý.
```

Trong ví dụ trên, error của Future từ `asyncErrorFunction()` được xử lý bằng
callback `onError`; `anotherAsyncErrorFunction()` khiến Future của `then()`
hoàn thành với error; error này được `catchError()` xử lý.

Nhìn chung, không nên triển khai hai chiến lược xử lý lỗi khác nhau:
chỉ đăng ký callback thứ hai nếu có lý do thuyết phục
để bắt error bên trong `then()`.

<a id="errors-in-the-middle-of-a-long-chain"></a>
### Error ở giữa một chuỗi dài

Việc có một chuỗi liên tiếp các lệnh gọi `then()` và bắt error
được tạo ra từ bất kỳ phần nào trong chuỗi bằng `catchError()` là điều phổ biến:

<?code-excerpt "futures/lib/long_chain.dart"?>
```dart
Future<String> one() => Future.value('from one');
Future<String> two() => Future.error('error from two');
Future<String> three() => Future.value('from three');
Future<String> four() => Future.value('from four');

void main() {
  one() // Future hoàn thành với "from one".
      .then((_) => two()) // Future hoàn thành với error của two().
      .then((_) => three()) // Future hoàn thành với error của two().
      .then((_) => four()) // Future hoàn thành với error của two().
      .then((value) => value.length) // Future hoàn thành với error của two().
      .catchError((e) {
        print('Got error: $e'); // Cuối cùng, callback được kích hoạt.
        return 42; // Future hoàn thành với 42.
      })
      .then((value) {
        print('The value is $value');
      });
}

// Output của chương trình này:
//   Got error: error from two
//   The value is 42
```

Trong code trên, Future của `one()` hoàn thành với value, nhưng Future của `two()`
hoàn thành với error. Khi `then()` được gọi trên một Future
hoàn thành với error, callback của `then()` không được kích hoạt. Thay vào đó,
Future của `then()` hoàn thành với error của receiver. Trong ví dụ,
điều này có nghĩa là sau khi `two()` được gọi, Future do mỗi
`then()` tiếp theo trả về đều hoàn thành với error của `two()`. Error đó cuối cùng
được xử lý bên trong `catchError()`.

<a id="handling-specific-errors"></a>
### Xử lý các error cụ thể

Nếu chúng ta muốn bắt một error cụ thể thì sao? Hoặc bắt nhiều error thì sao?

`catchError()` nhận một named argument tùy chọn, `test`,
cho phép chúng ta truy vấn loại error được throw.

<?code-excerpt "futures/lib/simple.dart (future-catch-error)"?>
```dart
Future<T> catchError(Function onError, {bool Function(Object error)? test});
```

Hãy xem xét `handleAuthResponse(params)`, một hàm xác thực người dùng
dựa trên các param được cung cấp và chuyển hướng người dùng đến URL thích hợp.
Với workflow phức tạp, `handleAuthResponse()` có thể tạo ra nhiều
error và exception khác nhau, và bạn nên xử lý chúng theo cách khác nhau. Sau đây là
cách bạn có thể sử dụng `test` để làm điều đó:

<?code-excerpt "futures/lib/simple.dart (auth-response)" replace="/ellipsis\(\)/.../g;"?>
```dart
void main() {
  handleAuthResponse(const {'username': 'dash', 'age': 3})
      .then((_) => ...)
      .catchError(handleFormatException, test: (e) => e is FormatException)
      .catchError(
        handleAuthorizationException,
        test: (e) => e is AuthorizationException,
      );
}
```

<a id="async-try-catch-finally-using-whencomplete"></a>
## try-catch-finally bất đồng bộ bằng whenComplete()

Nếu `then().catchError()` phản ánh try-catch, `whenComplete()` tương đương
với 'finally'. Callback được đăng ký bên trong `whenComplete()` được
gọi khi receiver của `whenComplete()` hoàn thành, dù nó hoàn thành với
value hay error:

<?code-excerpt "futures/lib/simple.dart (connect-server)"?>
```dart
final server = connectToServer();
server
    .post(myUrl, fields: const {'name': 'Dash', 'profession': 'mascot'})
    .then(handleResponse)
    .catchError(handleError)
    .whenComplete(server.close);
```

Chúng ta muốn gọi `server.close` bất kể `server.post()` tạo ra
response hợp lệ hay error. Chúng ta đảm bảo điều này xảy ra bằng cách đặt nó bên trong
`whenComplete()`.

<a id="completing-the-future-returned-by-whencomplete"></a>
### Hoàn thành Future do whenComplete() trả về

Nếu không có error nào được phát ra từ bên trong `whenComplete()`, Future của nó hoàn thành
theo cùng cách với Future mà `whenComplete()` được gọi trên đó. Điều này
dễ hiểu nhất qua các ví dụ.

Trong code dưới đây, Future của `then()` hoàn thành với error, do đó
Future của `whenComplete()` cũng hoàn thành với error đó.

<?code-excerpt "futures/lib/when_complete.dart (with-error)" replace="/withErrorMain/main/g; "?>
```dart
void main() {
  asyncErrorFunction()
      // Future hoàn thành với error:
      .then((_) => print("Won't reach here"))
      // Future hoàn thành với cùng error:
      .whenComplete(() => print('Reaches here'))
      // Future hoàn thành với cùng error:
      .then((_) => print("Won't reach here"))
      // Error được xử lý ở đây:
      .catchError(handleError);
}
```

Trong code dưới đây, Future của `then()` hoàn thành với error, hiện được
`catchError()` xử lý. Vì Future của `catchError()` hoàn thành với
`someObject`, Future của `whenComplete()` hoàn thành với cùng object đó.

<?code-excerpt "futures/lib/when_complete.dart (with-object)" replace="/ellipsis\(\)/.../g; /withObjectMain/main/g; "?>
```dart
void main() {
  asyncErrorFunction()
      // Future hoàn thành với error:
      .then((_) => ...)
      .catchError((e) {
        handleError(e);
        printErrorMessage();
        return someObject; // Future hoàn thành với someObject
      })
      .whenComplete(() => print('Done!')); // Future hoàn thành với someObject
}
```

<a id="errors-originating-within-whencomplete"></a>
### Error bắt nguồn từ bên trong whenComplete()

Nếu callback của `whenComplete()` throw error, Future của `whenComplete()`
hoàn thành với error đó:

<?code-excerpt "futures/lib/when_complete.dart (when-complete-error)" replace="/whenCompleteError/main/g; "?>
```dart
void main() {
  asyncErrorFunction()
      // Future hoàn thành với value:
      .catchError(handleError)
      // Future hoàn thành với error:
      .whenComplete(() => throw Exception('New error'))
      // Error được xử lý:
      .catchError(handleError);
}
```


<a id="potential-problem-failing-to-register-error-handlers-early"></a>
## Vấn đề tiềm ẩn: không đăng ký sớm trình xử lý lỗi

Điều tối quan trọng là trình xử lý lỗi phải được cài đặt trước khi Future hoàn thành:
điều này tránh trường hợp Future hoàn thành với error nhưng trình xử lý lỗi
chưa được đính kèm và error vô tình lan truyền. Hãy xem xét
code này:

<?code-excerpt "futures/lib/early_error_handlers.dart (bad)" replace="/ellipsis\(\)/.../g; /mainBad/main/g;"?>
```dart
void main() {
  Future<Object> future = asyncErrorFunction();

  // SAI: Quá muộn để xử lý exception của asyncErrorFunction().
  Future.delayed(const Duration(milliseconds: 500), () {
    future.then(...).catchError(...);
  });
}
```

Trong code trên, `catchError()` không được đăng ký cho đến nửa giây sau khi
`asyncErrorFunction()` được gọi, và error không được xử lý.

Vấn đề biến mất nếu `asyncErrorFunction()` được gọi bên trong
callback `Future.delayed()`:

<?code-excerpt "futures/lib/early_error_handlers.dart (good)" replace="/ellipsis\(\)/.../g; /mainGood/main/g;"?>
```dart
void main() {
  Future.delayed(const Duration(milliseconds: 500), () {
    asyncErrorFunction()
        .then(...)
        .catchError(...); // Chúng ta đến được đây.
  });
}
```

<a id="potential-problem-accidentally-mixing-synchronous-and-asynchronous-errors"></a>
## Vấn đề tiềm ẩn: vô tình trộn lẫn error đồng bộ và bất đồng bộ

Các hàm trả về Future hầu như luôn phải phát error của chúng trong
future. Vì chúng ta không muốn caller của các hàm đó phải
triển khai nhiều kịch bản xử lý lỗi, chúng ta muốn ngăn mọi error đồng bộ
rò rỉ ra ngoài. Hãy xem xét code này:

<?code-excerpt "futures/bin/mixing_errors_problematic.dart (parse)"?>
```dart
Future<int> parseAndRead(Map<String, dynamic> data) {
  final filename = obtainFilename(data); // Có thể throw.
  final file = File(filename);
  return file.readAsString().then((contents) {
    return parseFileData(contents); // Có thể throw.
  });
}
```

Hai hàm trong code đó có khả năng throw theo cách đồng bộ:
`obtainFilename()` và `parseFileData()`. Vì `parseFileData()` thực thi
bên trong callback `then()`, error của nó không rò rỉ ra khỏi hàm.
Thay vào đó, Future của `then()` hoàn thành với error của `parseFileData()`, error đó
cuối cùng hoàn thành Future của `parseAndRead()`, và error có thể được
`catchError()` xử lý thành công.

Nhưng `obtainFilename()` không được gọi bên trong callback `then()`; nếu _nó_
throw, một error đồng bộ sẽ lan truyền:

<?code-excerpt "futures/bin/mixing_errors_problematic.dart (main)"?>
```dart
void main() {
  parseAndRead(data).catchError((e) {
    print('Inside catchError');
    print(e);
    return -1;
  });
}

// Output của chương trình:
//   Unhandled exception:
//   <error from obtainFilename>
//   ...
```

Vì việc sử dụng `catchError()` không bắt được error, client của
`parseAndRead()` sẽ phải triển khai một chiến lược xử lý lỗi riêng cho
error này.

<a id="solution-using-futuresync-to-wrap-your-code"></a>
### Giải pháp: Sử dụng Future.sync() để bọc code của bạn

Một pattern phổ biến để đảm bảo không có error đồng bộ nào vô tình
được throw từ một hàm là bọc thân hàm bên trong một callback `Future.sync()`
mới:

<?code-excerpt "futures/bin/mixing_errors_solution.dart (parse)"?>
```dart
Future<int> parseAndRead(Map<String, dynamic> data) {
  return Future.sync(() {
    final filename = obtainFilename(data); // Có thể throw.
    final file = File(filename);
    return file.readAsString().then((contents) {
      return parseFileData(contents); // Có thể throw.
    });
  });
}
```

Nếu callback trả về một value không phải Future, Future của `Future.sync()` hoàn thành
với value đó. Nếu callback throw (như trong ví dụ
trên), Future hoàn thành với error. Nếu chính callback trả về một
Future, value hoặc error của Future đó sẽ hoàn thành Future của `Future.sync()`.

Với code được bọc trong `Future.sync()`, `catchError()` có thể xử lý mọi error:

<?code-excerpt "futures/bin/mixing_errors_solution.dart (main)"?>
```dart
void main() {
  parseAndRead(data).catchError((e) {
    print('Inside catchError');
    print(e);
    return -1;
  });
}

// Output của chương trình:
//   Inside catchError
//   <error from obtainFilename>
```

`Future.sync()` giúp code của bạn chống chịu được exception không được bắt. Nếu
hàm của bạn chứa rất nhiều code, có khả năng bạn đang làm
điều gì đó nguy hiểm mà không nhận ra:

<?code-excerpt "futures/bin/mixing_errors_problematic.dart (fragile)" replace="/ellipsis\(\);/.../g;"?>
```dart
Future fragileFunc() {
  return Future.sync(() {
    final x = someFunc(); // Bất ngờ throw trong một số trường hợp hiếm gặp.
    var y = 10 / x; // x không được bằng 0.
    ...
  });
}
```

`Future.sync()` không chỉ cho phép bạn xử lý những error mà bạn biết có thể xảy ra,
mà còn ngăn error *vô tình* rò rỉ ra khỏi hàm.


<a id="more-information"></a>
## Thông tin thêm

Xem [tài liệu tham khảo API của Future][Future class]
để biết thêm thông tin về Future.

[Future class]: {{site.dart-api}}/dart-async/Future-class.html
