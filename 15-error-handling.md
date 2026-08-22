---
title: Error handling
description: Tìm hiểu cách xử lý error và exception trong Dart.
prevpage:
  url: /language/branches
  title: Rẽ nhánh
nextpage:
  url: /language/functions
  title: Hàm
---

<a id="exceptions"></a>

## Exception

Code Dart của bạn có thể ném và bắt exception. Exception là error
cho biết điều gì đó ngoài dự kiến đã xảy ra. Nếu exception không được
bắt, [isolate][] đã phát sinh exception sẽ bị đình chỉ,
và thông thường isolate cùng chương trình của nó sẽ bị chấm dứt.

Khác với Java, tất cả exception trong Dart đều là unchecked exception.
Phương thức không khai báo những exception mà chúng có thể ném và bạn không
bắt buộc phải bắt bất kỳ exception nào.

Dart cung cấp các kiểu [`Exception`][] và [`Error`][],
cùng nhiều subtype được định nghĩa sẵn. Tất nhiên, bạn có thể
định nghĩa exception riêng. Tuy nhiên, chương trình Dart có thể ném bất kỳ
đối tượng khác null nào—không chỉ đối tượng Exception và Error—làm exception.

<a id="throw"></a>

### Throw

Sau đây là ví dụ về việc ném, hay *phát sinh*, một exception:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (throw-FormatException)"?>
```dart
throw FormatException('Expected at least 1 section');
```

Bạn cũng có thể ném đối tượng bất kỳ:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (out-of-llamas)"?>
```dart
throw 'Out of llamas!';
```

:::note
Code đạt chất lượng production thường ném các kiểu
implement [`Error`][] hoặc [`Exception`][].
:::

Vì việc ném exception là một biểu thức, bạn có thể ném exception
trong câu lệnh =\>, cũng như ở bất cứ nơi nào khác cho phép biểu thức:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (throw-is-an-expression)"?>
```dart
void distanceTo(Point other) => throw UnimplementedError();
```


<a id="catch"></a>

### Catch

Việc bắt, hay thu giữ, một exception sẽ ngăn exception
lan truyền (trừ khi bạn rethrow exception).
Bắt exception cho bạn cơ hội xử lý nó:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (try)"?>
```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  buyMoreLlamas();
}
```

Để xử lý code có thể ném nhiều loại exception, bạn có thể
chỉ định nhiều mệnh đề catch. Mệnh đề catch đầu tiên khớp với
kiểu của đối tượng được ném sẽ xử lý exception. Nếu mệnh đề catch không
chỉ định kiểu, mệnh đề đó có thể xử lý bất kỳ kiểu đối tượng nào được ném:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (try-catch)"?>
```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // Một exception cụ thể
  buyMoreLlamas();
} on Exception catch (e) {
  // Bất kỳ đối tượng nào khác là exception
  print('Unknown exception: $e');
} catch (e) {
  // Không chỉ định kiểu, xử lý tất cả
  print('Something really unknown: $e');
}
```

Như đoạn code trên cho thấy, bạn có thể sử dụng `on`, `catch` hoặc cả hai.
Sử dụng `on` khi cần chỉ định kiểu exception. Sử dụng `catch` khi
trình xử lý exception cần đối tượng exception.

Bạn có thể chỉ định một hoặc hai tham số cho `catch()`.
Tham số đầu tiên là exception được ném,
và tham số thứ hai là stack trace (một đối tượng [`StackTrace`][]).

<?code-excerpt "misc/lib/language_tour/exceptions.dart (try-catch-2)" replace="/\(e.*?\)/[!$&!]/g"?>
```dart
try {
  // ···
} on Exception catch [!(e)!] {
  print('Exception details:\n $e');
} catch [!(e, s)!] {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```

Để xử lý một phần exception
trong khi vẫn cho phép nó lan truyền,
hãy sử dụng keyword `rethrow`.

<?code-excerpt "misc/test/language_tour/exceptions_test.dart (rethrow)" replace="/rethrow;/[!$&!]/g"?>
```dart
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++); // Lỗi runtime
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    [!rethrow;!] // Cho phép caller nhìn thấy exception.
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```


<a id="finally"></a>

### Finally

Để bảo đảm một đoạn code chạy dù exception có được ném hay không, hãy sử dụng
mệnh đề `finally`. Nếu không có mệnh đề `catch` nào khớp với exception,
exception sẽ được lan truyền sau khi mệnh đề `finally` chạy:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (finally)"?>
```dart
try {
  breedMoreLlamas();
} finally {
  // Luôn dọn dẹp, ngay cả khi exception được ném.
  cleanLlamaStalls();
}
```

Mệnh đề `finally` chạy sau mọi mệnh đề `catch` khớp:

<?code-excerpt "misc/lib/language_tour/exceptions.dart (try-catch-finally)"?>
```dart
try {
  breedMoreLlamas();
} catch (e) {
  print('Error: $e'); // Xử lý exception trước.
} finally {
  cleanLlamaStalls(); // Sau đó dọn dẹp.
}
```

Để tìm hiểu thêm, hãy xem
[tài liệu exception của thư viện cốt lõi](/libraries/dart-core#exceptions).

<a id="assert"></a>

## Assert

Trong quá trình phát triển, hãy sử dụng câu lệnh assert—
`assert(<condition>, <optionalMessage>);`—để
làm gián đoạn quá trình thực thi bình thường nếu một điều kiện boolean là false.

<?code-excerpt "misc/test/language_tour/control_flow_test.dart (assert)"?>
```dart
// Bảo đảm biến có giá trị khác null.
assert(text != null);

// Bảo đảm giá trị nhỏ hơn 100.
assert(number < 100);

// Bảo đảm đây là URL https.
assert(urlString.startsWith('https'));
```

Để đính kèm thông báo vào một assertion,
hãy thêm một chuỗi làm đối số thứ hai của `assert`
(có thể kèm [dấu phẩy cuối][]):

<?code-excerpt "misc/test/language_tour/control_flow_test.dart (assert-with-message)"?>
```dart
assert(
  urlString.startsWith('https'),
  'URL ($urlString) should start with "https".',
);
```

Đối số đầu tiên của `assert` có thể là bất kỳ biểu thức nào
được phân giải thành giá trị boolean. Nếu giá trị biểu thức
là true, assertion thành công và quá trình thực thi
tiếp tục. Nếu là false, assertion thất bại và một exception
([`AssertionError`][]) được ném.

Chính xác thì assertion hoạt động khi nào?
Điều đó phụ thuộc vào công cụ và framework bạn sử dụng:

* Flutter bật assertion trong [debug mode][Flutter debug mode].
* Các công cụ chỉ dành cho phát triển như [`webdev serve`][]
  thường bật assertion theo mặc định.
* Một số công cụ như [`dart run`][] và [`dart compile js`][]
  hỗ trợ assertion thông qua flag dòng lệnh: `--enable-asserts`.

Trong code production, assertion bị bỏ qua
và các đối số của `assert` không được đánh giá.

[dấu phẩy cuối]: /language/collections#trailing-comma
[`AssertionError`]: {{site.dart-api}}/dart-core/AssertionError-class.html
[Flutter debug mode]: {{site.flutter-docs}}/testing/build-modes#debug
[`webdev serve`]: /tools/webdev#serve
[`dart run`]: /tools/dart-run
[`dart compile js`]: /tools/dart-compile#js

[isolate]: /language/concurrency#isolates
[`Error`]: {{site.dart-api}}/dart-core/Error-class.html
[`Exception`]: {{site.dart-api}}/dart-core/Exception-class.html
[`StackTrace`]: {{site.dart-api}}/dart-core/StackTrace-class.html
