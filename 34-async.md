---
title: Lập trình bất đồng bộ
description: Thông tin về cách viết code bất đồng bộ trong Dart.
shortTitle: Lập trình async
prevpage:
  url: /language/concurrency
  title: Concurrency
nextpage:
  url: /language/isolates
  title: Isolate
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

Các library Dart có rất nhiều function
trả về object [`Future`][] hoặc [`Stream`][].
Những function này là _bất đồng bộ_:
chúng trả về sau khi thiết lập
một thao tác có thể tốn thời gian
(chẳng hạn I/O),
mà không chờ thao tác đó hoàn tất.

Các keyword `async` và `await` hỗ trợ lập trình bất đồng bộ,
cho phép bạn viết code bất đồng bộ
trông tương tự code đồng bộ.


<a id="handling-futures"></a>
## Xử lý Future

Khi cần kết quả của một Future đã hoàn thành,
bạn có hai lựa chọn:

* Dùng `async` và `await`, như được mô tả ở đây và trong
  [hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await).
* Dùng API Future, như được mô tả trong
  [tài liệu `dart:async`](/libraries/dart-async#future).

Code dùng `async` và `await` là bất đồng bộ,
nhưng trông rất giống code đồng bộ.
Ví dụ, sau đây là đoạn code dùng `await`
để chờ kết quả của một function bất đồng bộ:

<?code-excerpt "misc/lib/language_tour/async.dart (await-look-up-version)"?>
```dart
await lookUpVersion();
```

Để dùng `await`, code phải nằm trong một function `async`—một
function được đánh dấu là `async`:

<?code-excerpt "misc/lib/language_tour/async.dart (checkVersion)" replace="/async|await/[!$&!]/g"?>
```dart
Future<void> checkVersion() [!async!] {
  var version = [!await!] lookUpVersion();
  // Làm điều gì đó với version
}
```

:::note
Mặc dù function `async` có thể thực hiện các thao tác tốn thời gian,
nó không chờ những thao tác đó.
Thay vào đó, function `async` chỉ thực thi
cho đến khi gặp biểu thức `await` đầu tiên.
Sau đó nó trả về một object `Future`,
và chỉ tiếp tục thực thi sau khi biểu thức `await` hoàn tất.
:::

Dùng `try`, `catch` và `finally` để xử lý lỗi và dọn dẹp
trong code sử dụng `await`:

<?code-excerpt "misc/lib/language_tour/async.dart (try-catch)"?>
```dart
try {
  version = await lookUpVersion();
} catch (e) {
  // Phản ứng khi không thể tra cứu version
}
```

Bạn có thể dùng `await` nhiều lần trong một function `async`.
Ví dụ, đoạn code sau chờ ba lần
để nhận kết quả của các function:

<?code-excerpt "misc/lib/language_tour/async.dart (repeated-await)"?>
```dart
var entrypoint = await findEntryPoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
```

Trong <code>await <em>expression</em></code>,
giá trị của <code><em>expression</em></code> thường là Future;
nếu không phải, giá trị đó sẽ tự động được bọc trong Future.
Object Future này biểu thị một lời hứa sẽ trả về object.
Giá trị của <code>await <em>expression</em></code> là object được trả về đó.
Biểu thức await làm quá trình thực thi tạm dừng cho đến khi object khả dụng.

**Nếu gặp lỗi compile-time khi dùng `await`,
hãy bảo đảm `await` nằm trong một function `async`.**
Ví dụ, để dùng `await` trong function `main()` của ứng dụng,
phần thân của `main()` phải được đánh dấu là `async`:

<?code-excerpt "misc/lib/language_tour/async.dart (main)" replace="/async|await/[!$&!]/g"?>
```dart
void main() [!async!] {
  checkVersion();
  print('In main: version is ${[!await!] lookUpVersion()}');
}
```

:::note
Ví dụ trước dùng một function `async` (`checkVersion()`)
mà không chờ kết quả—một cách làm có thể gây vấn đề
nếu code giả định function đã thực thi xong.
Để tránh vấn đề này,
hãy dùng [linter rule unawaited_futures][unawaited_futures linter rule].
:::

Để xem phần giới thiệu tương tác về cách dùng future, `async` và `await`,
xem [hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await).


<a id="declaring-async-functions"></a>
## Khai báo function async

Function `async` là function có phần thân được đánh dấu bằng
modifier `async`.

Thêm keyword `async` vào một function làm cho nó trả về Future.
Ví dụ, hãy xem function đồng bộ sau,
function này trả về String:

<?code-excerpt "misc/lib/language_tour/async.dart (sync-look-up-version)"?>
```dart
String lookUpVersion() => '1.0.0';
```

Nếu bạn thay đổi nó thành function `async`—ví dụ,
vì implementation trong tương lai sẽ tốn thời gian—thì
giá trị trả về là Future:

<?code-excerpt "misc/lib/language_tour/async.dart (async-look-up-version)"?>
```dart
Future<String> lookUpVersion() async => '1.0.0';
```

Lưu ý rằng phần thân function không cần dùng API Future.
Dart tạo object Future nếu cần.
Nếu function không trả về giá trị hữu ích,
hãy đặt return type của nó là `Future<void>`.

Để xem phần giới thiệu tương tác về cách dùng future, `async` và `await`,
xem [hướng dẫn lập trình bất đồng bộ](/libraries/async/async-await).

{% comment %}
VIỆC CẦN LÀM #1117: Chúng ta nên đề cập generalized void ở đâu nữa?
{% endcomment %}


<a id="handling-streams"></a>
## Xử lý Stream

Khi cần nhận giá trị từ Stream,
bạn có hai lựa chọn:

* Dùng `async` và một _vòng lặp for bất đồng bộ_ (`await for`).
* Dùng API Stream, như được mô tả trong
  [tài liệu `dart:async`](/libraries/dart-async#stream).

:::note
Trước khi dùng `await for`, hãy bảo đảm rằng nó làm code rõ ràng hơn và bạn
thực sự muốn chờ tất cả kết quả của stream. Ví dụ, thông thường bạn
**không nên** dùng `await for` cho listener sự kiện UI, vì framework UI
gửi các stream sự kiện vô tận.
:::

Vòng lặp for bất đồng bộ có dạng sau:

<?code-excerpt "misc/lib/language_tour/async.dart (await-for)"?>
```dart
await for (varOrType identifier in expression) {
  // Thực thi mỗi khi stream phát ra một giá trị.
}
```

Giá trị của <code><em>expression</em></code> phải có kiểu Stream.
Quá trình thực thi diễn ra như sau:

1. Chờ cho đến khi stream phát ra một giá trị.
2. Thực thi phần thân vòng lặp for,
   với biến được đặt thành giá trị vừa phát ra.
3. Lặp lại bước 1 và 2 cho đến khi stream đóng.

Để ngừng lắng nghe stream,
bạn có thể dùng statement `break` hoặc `return`,
thao tác này thoát khỏi vòng lặp for
và hủy đăng ký khỏi stream.

**Nếu gặp lỗi compile-time khi implement vòng lặp for bất đồng bộ,
hãy bảo đảm `await for` nằm trong một function `async`.**
Ví dụ, để dùng vòng lặp for bất đồng bộ trong function `main()` của ứng dụng,
phần thân của `main()` phải được đánh dấu là `async`:

<?code-excerpt "misc/lib/language_tour/async.dart (number-thinker)" replace="/async|await for/[!$&!]/g"?>
```dart
void main() [!async!] {
  // ...
  [!await for!] (final request in requestServer) {
    handleRequest(request);
  }
  // ...
}
```

Để biết thêm thông tin về khả năng hỗ trợ lập trình bất đồng bộ của Dart,
xem tài liệu library [`dart:async`](/libraries/dart-async).

[`Future`]: {{site.dart-api}}/dart-async/Future-class.html
[`Stream`]: {{site.dart-api}}/dart-async/Stream-class.html
[unawaited_futures linter rule]: /tools/linter-rules/unawaited_futures
