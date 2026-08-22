---
title: dart:async
description: Tìm hiểu về các tính năng chính trong thư viện dart:async của Dart.
prevpage:
  url: /libraries/dart-core
  title: dart:core
nextpage:
  url: /libraries/dart-math
  title: dart:math
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt plaster="none"?>

Lập trình bất đồng bộ thường sử dụng các hàm callback, nhưng Dart
cung cấp những lựa chọn thay thế: object [Future][] và [Stream][]. Một
Future giống như lời hứa rằng kết quả sẽ được cung cấp vào một thời điểm trong
tương lai. Stream là cách nhận một chuỗi value, chẳng hạn như các event.
Future, Stream và nhiều thành phần khác nằm trong
thư viện dart:async ([tài liệu tham khảo API][dart:async]).

:::note
Không phải lúc nào bạn cũng cần sử dụng trực tiếp API Future hoặc Stream.
Ngôn ngữ Dart hỗ trợ code bất đồng bộ bằng
các keyword như `async` và `await`.
Hãy xem [hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await)
để biết chi tiết.
:::

Thư viện dart:async hoạt động trong cả ứng dụng web và ứng dụng dòng lệnh. Để
sử dụng, hãy import dart:async:

<?code-excerpt "misc/lib/library_tour/async/future.dart (import)"?>
```dart
import 'dart:async';
```

:::tip
Bạn không cần import dart:async để sử dụng API Future và
Stream vì dart:core export các class đó.
:::

<a id="future"></a>
## Future

Object Future xuất hiện trong khắp các thư viện Dart, thường là object
được một method bất đồng bộ trả về. Khi một future *hoàn thành*, value của nó
sẵn sàng để sử dụng.


<a id="using-await"></a>
### Sử dụng await

Trước khi trực tiếp sử dụng Future API, hãy cân nhắc sử dụng `await`.
Code sử dụng biểu thức `await` có thể dễ hiểu hơn
code sử dụng Future API.

Hãy xem xét hàm sau. Hàm sử dụng method `then()` của Future
để thực thi lần lượt ba hàm bất đồng bộ,
chờ từng hàm hoàn thành trước khi thực thi hàm tiếp theo.

<?code-excerpt "misc/lib/library_tour/async/future.dart (run-using-future)"?>
```dart
void runUsingFuture() {
  // ...
  findEntryPoint()
      .then((entryPoint) {
        return runExecutable(entryPoint, args);
      })
      .then(flushThenExit);
}
```

Code tương đương dùng biểu thức await
trông giống code đồng bộ hơn:

<?code-excerpt "misc/lib/library_tour/async/future.dart (run-using-async-await)"?>
```dart
Future<void> runUsingAsyncAwait() async {
  // ...
  var entryPoint = await findEntryPoint();
  var exitCode = await runExecutable(entryPoint, args);
  await flushThenExit(exitCode);
}
```

Một hàm `async` có thể bắt exception từ Future.
Ví dụ:

<?code-excerpt "misc/lib/library_tour/async/future.dart (catch)"?>
```dart
var entryPoint = await findEntryPoint();
try {
  var exitCode = await runExecutable(entryPoint, args);
  await flushThenExit(exitCode);
} catch (e) {
  // Xử lý lỗi...
}
```

:::important
Hàm async trả về Future. Nếu không muốn hàm của bạn trả về một
future, hãy sử dụng giải pháp khác. Ví dụ, bạn có thể gọi một hàm `async`
từ hàm của mình.
:::

Để biết thêm thông tin về cách sử dụng `await` và các tính năng liên quan của ngôn ngữ Dart,
hãy xem [hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await).


<a id="basic-usage"></a>
### Cách sử dụng cơ bản

Bạn có thể sử dụng `then()` để lên lịch cho code chạy khi future hoàn thành. Ví dụ,
[`Client.read()`][] trả về một Future vì HTTP request
có thể mất một lúc. Sử dụng `then()` cho phép bạn chạy một đoạn code khi Future đó
đã hoàn thành và value string được hứa hẹn đã sẵn sàng:

<?code-excerpt "misc/lib/library_tour/async/basic.dart (then)"?>
```dart
httpClient.read(url).then((String result) {
  print(result);
});
```

Sử dụng `catchError()` để xử lý mọi error hoặc exception mà một object Future
có thể throw.

<?code-excerpt "misc/lib/library_tour/async/basic.dart (catch-error)"?>
```dart
httpClient
    .read(url)
    .then((String result) {
      print(result);
    })
    .catchError((e) {
      // Xử lý hoặc bỏ qua lỗi.
    });
```

Pattern `then().catchError()` là phiên bản bất đồng bộ của
`try`-`catch`.

:::important
Hãy đảm bảo gọi `catchError()` trên kết quả của `then()`—không phải trên
kết quả của `Future` ban đầu. Nếu không,
`catchError()` chỉ có thể xử lý lỗi từ phép tính của Future ban đầu,
chứ không xử lý lỗi từ handler được `then()` đăng ký.
:::

[`Client.read()`]: {{site.pub-api}}/http/latest/http/Client/read.html

<a id="chaining-multiple-asynchronous-methods"></a>
### Nối chuỗi nhiều method bất đồng bộ

Method `then()` trả về một Future, cung cấp một cách hữu ích để chạy
nhiều hàm bất đồng bộ theo thứ tự nhất định.
Nếu callback được đăng ký với `then()` trả về một Future,
`then()` sẽ trả về một Future hoàn thành
với cùng kết quả như Future mà callback trả về.
Nếu callback trả về value thuộc bất kỳ kiểu nào khác,
`then()` tạo một Future mới hoàn thành với value đó.

<?code-excerpt "misc/lib/library_tour/async/future.dart (then-chain)"?>
```dart
Future result = costlyQuery(url);
result
    .then((value) => expensiveWork(value))
    .then((_) => lengthyComputation())
    .then((_) => print('Done!'))
    .catchError((exception) {
      /* Xử lý exception... */
    });
```

Trong ví dụ trên, các method chạy theo thứ tự sau:

1.  `costlyQuery()`
2.  `expensiveWork()`
3.  `lengthyComputation()`

Đây là cùng đoạn code đó được viết bằng await:

<?code-excerpt "misc/lib/library_tour/async/future.dart (then-chain-as-await)"?>
```dart
try {
  final value = await costlyQuery(url);
  await expensiveWork(value);
  await lengthyComputation();
  print('Done!');
} catch (e) {
  /* Xử lý exception... */
}
```


<a id="waiting-for-multiple-futures"></a>
### Chờ nhiều future

Đôi khi thuật toán của bạn cần gọi nhiều hàm bất đồng bộ và
chờ tất cả hoàn thành trước khi tiếp tục. Sử dụng method static [Future.wait()][]
để quản lý nhiều Future và chờ chúng hoàn thành:

<?code-excerpt "misc/lib/library_tour/async/future.dart (wait)" replace="/elideBody;/\/* ... *\//g"?>
```dart
Future<void> deleteLotsOfFiles() async =>  ...
Future<void> copyLotsOfFiles() async =>  ...
Future<void> checksumLotsOfOtherFiles() async =>  ...

await Future.wait([
  deleteLotsOfFiles(),
  copyLotsOfFiles(),
  checksumLotsOfOtherFiles(),
]);
print('Done with all the long steps!');
```

`Future.wait()` trả về một future hoàn thành khi tất cả future được cung cấp
đã hoàn thành. Nó hoàn thành với kết quả của chúng,
hoặc với một error nếu bất kỳ future nào được cung cấp thất bại.

<a id="handling-errors-for-multiple-futures"></a>
### Xử lý lỗi cho nhiều future

Bạn cũng có thể chờ các thao tác song song trên:

* Một [iterable][iterable-futures] các future
* Một [record][record-futures] có các future làm positional field

Các extension này trả về một `Future` chứa
các value kết quả của tất cả future được cung cấp.
Không giống `Future.wait`, chúng cũng cho phép bạn xử lý lỗi.

Nếu bất kỳ future nào trong collection hoàn thành với lỗi,
`wait` hoàn thành với một [`ParallelWaitError`][].
Điều này cho phép caller xử lý từng lỗi riêng lẻ và
giải phóng các kết quả thành công nếu cần.

Khi bạn _không_ cần các value kết quả từ từng future,
hãy sử dụng `wait` trên một _iterable_ các future:

```dart
Future<int> delete() async => ...;
Future<String> copy() async => ...;
Future<bool> errorResult() async => ...;

void main() async {
  try {
    // Chờ từng future trong một list, trả về một list các future:
    var results = await [delete(), copy(), errorResult()].wait;
  } on ParallelWaitError<List<bool?>, List<AsyncError?>> catch (e) {
    print(e.values[0]);    // In ra future thành công
    print(e.values[1]);    // In ra future thành công
    print(e.values[2]);    // In ra null khi kết quả là một error

    print(e.errors[0]);    // In ra null khi kết quả thành công
    print(e.errors[1]);    // In ra null khi kết quả thành công
    print(e.errors[2]);    // In ra error
  }
}
```

Khi bạn _cần_ các value kết quả riêng lẻ từ từng future,
hãy sử dụng `wait` trên một _record_ các future.
Cách này có thêm lợi ích là các future có thể thuộc những kiểu khác nhau:

```dart
Future<int> delete() async => ...;
Future<String> copy() async => ...;
Future<bool> errorResult() async => ...;

void main() async {
  try {
    // Chờ từng future trong một record.
    // Trả về một record các future mà bạn có thể destructure.
    final (deleteInt, copyString, errorBool) =
        await (delete(), copy(), errorResult()).wait;

    // Làm điều gì đó với các kết quả...
  } on ParallelWaitError<
    (int?, String?, bool?),
    (AsyncError?, AsyncError?, AsyncError?)
  > catch (e) {
    // ...
  }
}
```

[iterable-futures]: {{site.dart-api}}/dart-async/FutureIterable/wait.html
[record-futures]: {{site.dart-api}}/dart-async/FutureRecord2/wait.html

<a id="stream"></a>
## Stream

Object Stream xuất hiện trong khắp các API Dart, biểu diễn chuỗi
dữ liệu. Ví dụ, event HTML như thao tác nhấp nút được truyền bằng
stream. Bạn cũng có thể đọc file dưới dạng stream.


<a id="using-an-asynchronous-for-loop"></a>
### Sử dụng vòng lặp for bất đồng bộ

Đôi khi bạn có thể sử dụng vòng lặp for bất đồng bộ (`await for`)
thay vì sử dụng Stream API.

Hãy xem xét hàm sau.
Hàm sử dụng method `listen()` của Stream
để subscribe vào một danh sách file,
truyền vào một function literal tìm kiếm trong từng file hoặc directory.

<?code-excerpt "misc/lib/library_tour/async/stream.dart (listen)" replace="/listen/[!$&!]/g"?>
```dart
void main(List<String> arguments) {
  // ...
  FileSystemEntity.isDirectory(searchPath).then((isDir) {
    if (isDir) {
      final startingDir = Directory(searchPath);
      startingDir.list().[!listen!]((entity) {
        if (entity is File) {
          searchFile(entity, searchTerms);
        }
      });
    } else {
      searchFile(File(searchPath), searchTerms);
    }
  });
}
```

Code tương đương dùng biểu thức await,
bao gồm một vòng lặp for bất đồng bộ (`await for`),
trông giống code đồng bộ hơn:

<?code-excerpt "misc/lib/library_tour/async/stream.dart (await-for)" replace="/await for/[!$&!]/g"?>
```dart
void main(List<String> arguments) async {
  // ...
  if (await FileSystemEntity.isDirectory(searchPath)) {
    final startingDir = Directory(searchPath);
    [!await for!] (final entity in startingDir.list()) {
      if (entity is File) {
        searchFile(entity, searchTerms);
      }
    }
  } else {
    searchFile(File(searchPath), searchTerms);
  }
}
```

:::important
Trước khi sử dụng `await for`, hãy đảm bảo nó làm code rõ ràng hơn và bạn
thực sự muốn chờ tất cả kết quả của stream. Ví dụ, thông thường bạn
**không** nên sử dụng `await for` cho DOM event listener vì DOM
gửi ra stream event vô tận. Nếu bạn sử dụng `await for` để lần lượt đăng ký hai
DOM event listener, loại event thứ hai sẽ không bao giờ được xử lý.
:::

Để biết thêm thông tin về cách sử dụng `await` và các
tính năng liên quan của ngôn ngữ Dart, hãy xem
[hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await).


<a id="listening-for-stream-data"></a>
### Lắng nghe dữ liệu stream

Để nhận từng value khi nó đến, hãy sử dụng `await for` hoặc
subscribe vào stream bằng method `listen()`:

<?code-excerpt "misc/lib/library_tour/async/stream_web.dart (listen)" replace="/listen/[!$&!]/g"?>
```dart
// Thêm một event handler vào button.
submitButton.onClick.[!listen!]((e) {
  // Khi button được nhấp, code này sẽ chạy.
  submitData();
});
```

Trong ví dụ này, property `onClick` là một object `Stream` do
button submit cung cấp.

Nếu bạn chỉ quan tâm đến một event, bạn có thể lấy event đó bằng một property như
`first`, `last` hoặc `single`. Để kiểm tra event trước khi xử lý,
hãy sử dụng một method như `firstWhere()`, `lastWhere()` hoặc `singleWhere()`.
{% comment %}
{ĐANG CHỜ: ví dụ}
{% endcomment %}

Nếu bạn quan tâm đến một tập con các event, bạn có thể sử dụng các method như
`skip()`, `skipWhile()`, `take()`, `takeWhile()` và `where()`.
{% comment %}
{ĐANG CHỜ: ví dụ}
{% endcomment %}


<a id="transforming-stream-data"></a>
### Biến đổi dữ liệu stream

Thông thường, bạn cần thay đổi định dạng dữ liệu của stream trước khi có thể
sử dụng nó. Sử dụng method `transform()` để tạo một stream với
kiểu dữ liệu khác:

<?code-excerpt "misc/lib/library_tour/async/stream.dart (transform)"?>
```dart
var lines = inputStream
    .transform(utf8.decoder)
    .transform(const LineSplitter());
```

Ví dụ này sử dụng hai transformer. Đầu tiên, nó sử dụng utf8.decoder để
biến đổi stream các integer thành stream các string. Sau đó, nó sử dụng
LineSplitter để biến đổi stream các string thành stream gồm
các dòng riêng biệt. Những transformer này thuộc thư viện dart:convert (xem
[phần dart:convert](/libraries/dart-convert)).
{% comment %}
  ĐANG CHỜ: thêm onDone và onError. (Xem "Đọc nội dung file theo stream".)
{% endcomment %}


<a id="handling-errors-and-completion"></a>
### Xử lý lỗi và sự hoàn thành

Cách bạn chỉ định code xử lý lỗi và sự hoàn thành
phụ thuộc vào việc bạn sử dụng vòng lặp for bất đồng bộ (`await for`)
hay Stream API.

Nếu sử dụng vòng lặp for bất đồng bộ,
hãy dùng try-catch để xử lý lỗi.
Code thực thi sau khi stream đóng
được đặt sau vòng lặp for bất đồng bộ.

<?code-excerpt "misc/lib/library_tour/async/stream.dart (read-file-await-for)" replace="/try|catch/[!$&!]/g"?>
```dart
Future<void> readFileAwaitFor() async {
  var config = File('config.txt');
  Stream<List<int>> inputStream = config.openRead();

  var lines = inputStream
      .transform(utf8.decoder)
      .transform(const LineSplitter());
  [!try!] {
    await for (final line in lines) {
      print('Got ${line.length} characters from stream');
    }
    print('file is now closed');
  } [!catch!] (e) {
    print(e);
  }
}
```

Nếu sử dụng Stream API,
hãy xử lý lỗi bằng cách đăng ký một listener `onError`.
Chạy code sau khi stream đóng bằng cách đăng ký
một listener `onDone`.

<?code-excerpt "misc/lib/library_tour/async/stream.dart (on-done)" replace="/onDone|onError/[!$&!]/g"?>
```dart
var config = File('config.txt');
Stream<List<int>> inputStream = config.openRead();

inputStream
    .transform(utf8.decoder)
    .transform(const LineSplitter())
    .listen(
      (String line) {
        print('Got ${line.length} characters from stream');
      },
      [!onDone!]: () {
        print('file is now closed');
      },
      [!onError!]: (e) {
        print(e);
      },
    );
```


<a id="more-information"></a>
## Thông tin thêm

Để xem một số ví dụ về cách sử dụng Future và Stream trong ứng dụng dòng lệnh,
hãy xem [tài liệu dart:io][].
Đồng thời xem các bài viết và hướng dẫn sau:

- [Lập trình bất đồng bộ: future, async, await](/libraries/async/async-await)
- [Future và xử lý lỗi](/libraries/async/futures-error-handling)
- [Lập trình bất đồng bộ: stream](/libraries/async/using-streams)
- [Tạo stream trong Dart](/libraries/async/creating-streams)
- [Lập trình bất đồng bộ trong Dart: Isolate và event loop](/language/concurrency)


[Future.wait()]: {{site.dart-api}}/dart-async/Future/wait.html
[Future]: {{site.dart-api}}/dart-async/Future-class.html
[`ParallelWaitError`]: {{site.dart-api}}/dart-async/ParallelWaitError-class.html
[Stream]: {{site.dart-api}}/dart-async/Stream-class.html
[dart:async]: {{site.dart-api}}/dart-async/dart-async-library.html
[dart:io documentation]: /libraries/dart-io
