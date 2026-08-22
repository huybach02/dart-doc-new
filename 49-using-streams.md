---
title: "Asynchronous programming: Streams"
breadcrumb: Sử dụng stream
description: Tìm hiểu cách sử dụng stream single-subscription và broadcast.
---

:::secondary Mục đích là gì?
* Stream cung cấp một chuỗi dữ liệu bất đồng bộ.
* Chuỗi dữ liệu bao gồm event do người dùng tạo và dữ liệu đọc từ file.
* Bạn có thể xử lý stream bằng **await for** hoặc
  `listen()` từ Stream API.
* Stream cung cấp cách phản hồi error.
* Có hai loại stream: single-subscription và broadcast.
:::

<iframe width="560" height="315" src="https://www.youtube.com/embed/nQBpOIHE4eE?si=hM5ONj3PXHckEuCS" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Lập trình bất đồng bộ trong Dart được đặc trưng bởi các class
[Future][] và [Stream][].

Future biểu diễn một phép tính không hoàn thành ngay lập tức.
Trong khi hàm thông thường trả về kết quả, hàm bất đồng bộ
trả về Future mà cuối cùng sẽ chứa kết quả.
Future sẽ báo cho bạn khi kết quả sẵn sàng.

Stream là một chuỗi event bất đồng bộ.
Nó giống như một Iterable bất đồng bộ—thay vì nhận
event tiếp theo khi bạn yêu cầu, stream sẽ báo cho bạn rằng
có một event khi event đó sẵn sàng.

<a id="receiving-stream-events"></a>
## Nhận event của stream

Stream có thể được tạo theo nhiều cách, đây là chủ đề của một
bài viết khác, nhưng tất cả đều có thể được sử dụng theo cùng một cách: _vòng lặp
for bất đồng bộ_ (thường chỉ được gọi là **await for**)
lặp qua các event của stream giống như **vòng lặp for** lặp
qua một [Iterable][]. Ví dụ:

<?code-excerpt "misc/lib/tutorial/sum_stream.dart (sum-stream)" replace="/async|await for/[!$&!]/g"?>
```dart
Future<int> sumStream(Stream<int> stream) [!async!] {
  var sum = 0;
  [!await for!] (final value in stream) {
    sum += value;
  }
  return sum;
}
```

Code này chỉ đơn giản nhận từng event của một stream các event integer,
cộng chúng lại và trả về (một future của) tổng.
Khi thân vòng lặp kết thúc,
hàm tạm dừng cho đến khi event tiếp theo đến hoặc stream kết thúc.

Hàm được đánh dấu bằng keyword `async`, điều bắt buộc
khi sử dụng vòng lặp **await for**.

Ví dụ sau kiểm thử code trước đó bằng cách
tạo một stream integer đơn giản bằng hàm `async*`:

:::note
Trang này sử dụng DartPad nhúng để hiển thị các ví dụ có thể chạy.
{% render 'dartpads-embedded-troubleshooting.md' %}
:::

<?code-excerpt "misc/lib/tutorial/sum_stream.dart"?>
```dartpad
Future<int> sumStream(Stream<int> stream) async {
  var sum = 0;
  await for (final value in stream) {
    sum += value;
  }
  return sum;
}

Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    yield i;
  }
}

void main() async {
  var stream = countStream(10);
  var sum = await sumStream(stream);
  print(sum); // 55
}
```

:::note
Nhấp vào **Run** để xem kết quả trong **Console**.
:::

<a id="error-events"></a>
## Event lỗi

Stream kết thúc khi không còn event nào trong đó,
và code nhận event được thông báo về điều này giống như
khi được thông báo rằng một event mới đã đến.
Khi đọc event bằng vòng lặp **await for**,
vòng lặp dừng khi stream kết thúc.

Trong một số trường hợp, error xảy ra trước khi stream kết thúc;
có thể mạng gặp lỗi khi lấy file từ server từ xa,
hoặc code tạo event có bug,
nhưng ai đó cần biết về điều đó.

Stream cũng có thể truyền event lỗi giống như truyền event dữ liệu.
Phần lớn stream sẽ dừng sau error đầu tiên,
nhưng có thể có stream truyền nhiều hơn một error,
và stream truyền thêm dữ liệu sau một event lỗi.
Trong tài liệu này, chúng ta chỉ thảo luận stream truyền tối đa một error.

Khi đọc stream bằng **await for**, error được throw bởi
câu lệnh vòng lặp. Điều này cũng kết thúc vòng lặp. Bạn có thể bắt
error bằng **try-catch**. Ví dụ sau throw một
error khi iterator của vòng lặp bằng 4:

<?code-excerpt "misc/lib/tutorial/sum_stream_with_catch.dart"?>
```dartpad
Future<int> sumStream(Stream<int> stream) async {
  var sum = 0;
  try {
    await for (final value in stream) {
      sum += value;
    }
  } catch (e) {
    return -1;
  }
  return sum;
}

Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    if (i == 4) {
      throw Exception('Intentional exception');
    } else {
      yield i;
    }
  }
}

void main() async {
  var stream = countStream(10);
  var sum = await sumStream(stream);
  print(sum); // -1
}
```

:::note
Nhấp vào **Run** để xem kết quả trong **Console**.
:::


<a id="working-with-streams"></a>
## Làm việc với stream

Class Stream chứa một số method hỗ trợ có thể thực hiện
các thao tác phổ biến trên stream cho bạn,
tương tự các method trên [Iterable.][Iterable]
Ví dụ, bạn có thể tìm integer dương cuối cùng trong stream bằng
`lastWhere()` từ Stream API.

<?code-excerpt "misc/lib/tutorial/misc.dart (last-positive)"?>
```dart
Future<int> lastPositive(Stream<int> stream) =>
    stream.lastWhere((x) => x >= 0);
```

## Hai loại stream {:#two-kinds-of-streams}

Có hai loại stream.

### Stream single-subscription {:#single-subscription-streams}

Loại stream phổ biến nhất chứa một chuỗi event là các phần
của một tổng thể lớn hơn.
Event cần được truyền theo đúng thứ tự và không
bỏ sót bất kỳ event nào.
Đây là loại stream bạn nhận được khi đọc file hoặc nhận
web request.

Stream như vậy chỉ có thể được lắng nghe một lần.
Việc lắng nghe lại sau đó có thể đồng nghĩa với bỏ lỡ các event ban đầu,
khiến phần còn lại của stream không còn ý nghĩa.
Khi bạn bắt đầu lắng nghe,
dữ liệu sẽ được lấy và cung cấp theo từng chunk.

### Stream broadcast {:#broadcast-streams}

Loại stream còn lại dành cho các message riêng lẻ có thể
được xử lý từng message một. Ví dụ, loại stream này có thể được sử dụng cho
event chuột trong trình duyệt.

Bạn có thể bắt đầu lắng nghe stream như vậy bất cứ lúc nào,
và nhận các event được phát ra trong khi bạn lắng nghe.
Nhiều listener có thể lắng nghe cùng lúc,
và bạn có thể lắng nghe lại sau khi hủy một
subscription trước đó.

## Các method xử lý stream {:#process-stream-methods}

Các method sau trên [Stream\<T>][Stream] xử lý stream và trả về một
kết quả:

<?code-excerpt "misc/lib/tutorial/stream_interface.dart (main-stream-members)" remove="/^\s*Stream/"?>
```dart
Future<T> get first;
Future<bool> get isEmpty;
Future<T> get last;
Future<int> get length;
Future<T> get single;
Future<bool> any(bool Function(T element) test);
Future<bool> contains(Object? needle);
Future<E> drain<E>([E? futureValue]);
Future<T> elementAt(int index);
Future<bool> every(bool Function(T element) test);
Future<T> firstWhere(bool Function(T element) test, {T Function()? orElse});
Future<S> fold<S>(S initialValue, S Function(S previous, T element) combine);
Future forEach(void Function(T element) action);
Future<String> join([String separator = '']);
Future<T> lastWhere(bool Function(T element) test, {T Function()? orElse});
Future pipe(StreamConsumer<T> streamConsumer);
Future<T> reduce(T Function(T previous, T element) combine);
Future<T> singleWhere(bool Function(T element) test, {T Function()? orElse});
Future<List<T>> toList();
Future<Set<T>> toSet();
```

Tất cả hàm này, ngoại trừ `drain()` và `pipe()`,
tương ứng với một hàm tương tự trên [Iterable.][Iterable]
Mỗi hàm đều có thể được viết dễ dàng bằng một hàm `async`
với vòng lặp **await for** (hoặc chỉ cần sử dụng một trong các method khác).
Ví dụ, một số implementation có thể là:

<?code-excerpt "misc/lib/tutorial/misc.dart (mock-stream-method-implementations)"?>
```dart
Future<bool> contains(Object? needle) async {
  await for (final event in this) {
    if (event == needle) return true;
  }
  return false;
}

Future forEach(void Function(T element) action) async {
  await for (final event in this) {
    action(event);
  }
}

Future<List<T>> toList() async {
  final result = <T>[];
  await forEach(result.add);
  return result;
}

Future<String> join([String separator = '']) async =>
    (await toList()).join(separator);
```

(Các implementation thực tế phức tạp hơn một chút,
nhưng chủ yếu vì lý do lịch sử.)

## Các method sửa đổi stream {:#modify-stream-methods}

Các method sau trên `Stream` trả về một stream mới dựa trên
stream ban đầu.
Mỗi method chờ đến khi có đối tượng lắng nghe stream mới rồi mới
lắng nghe stream ban đầu.

<?code-excerpt "misc/lib/tutorial/stream_interface.dart (main-stream-members)" remove="/async\w+|distinct|transform/" retain="/^\s*Stream/"?>
```dart
Stream<R> cast<R>();
Stream<S> expand<S>(Iterable<S> Function(T element) convert);
Stream<S> map<S>(S Function(T event) convert);
Stream<T> skip(int count);
Stream<T> skipWhile(bool Function(T element) test);
Stream<T> take(int count);
Stream<T> takeWhile(bool Function(T element) test);
Stream<T> where(bool Function(T event) test);
```

Các method trên tương ứng với các method tương tự trên [Iterable][],
biến đổi một iterable thành iterable khác.
Tất cả đều có thể được viết dễ dàng bằng hàm `async`
với vòng lặp **await for**.

<?code-excerpt "misc/lib/tutorial/stream_interface.dart (main-stream-members)" remove="/transform/" retain="/async\w+|distinct/"?>
```dart
Stream<E> asyncExpand<E>(Stream<E>? Function(T event) convert);
Stream<E> asyncMap<E>(FutureOr<E> Function(T event) convert);
Stream<T> distinct([bool Function(T previous, T next)? equals]);
```

Các hàm `asyncExpand()` và `asyncMap()` tương tự
`expand()` và `map()`,
nhưng cho phép đối số hàm của chúng là một hàm bất đồng bộ.
Hàm `distinct()` không tồn tại trên `Iterable`, nhưng hoàn toàn có thể có.

<?code-excerpt "misc/lib/tutorial/stream_interface.dart (special-stream-members)"?>
```dart
Stream<T> handleError(Function onError, {bool Function(dynamic error)? test});
Stream<T> timeout(
  Duration timeLimit, {
  void Function(EventSink<T> sink)? onTimeout,
});
Stream<S> transform<S>(StreamTransformer<T, S> streamTransformer);
```

Ba hàm cuối chuyên biệt hơn.
Chúng liên quan đến xử lý lỗi mà vòng lặp **await for**
không thể quản lý trực tiếp; error đầu tiên gặp phải sẽ
kết thúc vòng lặp và stream subscription của nó, không có
cơ chế phục hồi tích hợp sẵn.

Code sau minh họa cách sử dụng `handleError()`
để lọc error khỏi stream trước khi nó được
vòng lặp **await for** sử dụng.

<?code-excerpt "misc/lib/tutorial/misc.dart (map-log-errors)" plaster="none"?>
```dart highlightLines=5
Stream<S> mapLogErrors<S, T>(
  Stream<T> stream,
  S Function(T event) convert,
) async* {
  var streamWithoutErrors = stream.handleError((e) => log(e));

  await for (final event in streamWithoutErrors) {
    yield convert(event);
  }
}
```

Trong ví dụ trước, vòng lặp **await for** không bao giờ
được quay lại nếu stream không phát ra event nào.
Để tránh điều này, hãy sử dụng hàm `timeout()` để tạo
một stream mới. `timeout()` cho phép bạn đặt
giới hạn thời gian và tiếp tục phát event trên stream
được trả về.

Code sau sửa đổi ví dụ trước.
Nó thêm timeout hai giây và tạo ra một
error phù hợp nếu không có event nào xảy ra trong hai giây trở lên.

<?code-excerpt "misc/lib/tutorial/misc.dart (stream-timeout)"?>
```dart highlightLines=6-12
Stream<S> mapLogErrors<S, T>(
  Stream<T> stream,
  S Function(T event) convert,
) async* {
  var streamWithoutErrors = stream.handleError((e) => log(e));
  var streamWithTimeout = streamWithoutErrors.timeout(
    const Duration(seconds: 2),
    onTimeout: (eventSink) {
      eventSink.addError('Timed out after 2 seconds');
      eventSink.close();
    },
  );

  await for (final event in streamWithTimeout) {
    yield convert(event);
  }
}
```

### Hàm transform() {:#transform-function}

Hàm `transform()` không chỉ dành cho xử lý lỗi;
nó là một dạng "map" tổng quát hơn cho stream.
Map thông thường yêu cầu một value cho mỗi event đến.
Tuy nhiên, đặc biệt với stream I/O,
có thể cần nhiều event đầu vào để tạo một event đầu ra.
Một [StreamTransformer][] có thể xử lý việc đó.
Ví dụ, decoder như [Utf8Decoder][] là transformer.
Transformer chỉ yêu cầu một hàm, [bind()][], có thể được
triển khai dễ dàng bằng một hàm `async`.

### Đọc và giải mã file {:#reading-decoding-file}

Code sau đọc một file và chạy hai phép transform trên stream.
Đầu tiên, nó chuyển đổi dữ liệu từ UTF8 rồi chạy dữ liệu qua
một [LineSplitter.][LineSplitter]
Tất cả dòng được in ra, ngoại trừ những dòng bắt đầu bằng dấu thăng, `#`.

<?code-excerpt "misc/bin/cat_no_hash.dart"?>
```dart
import 'dart:convert';
import 'dart:io';

void main(List<String> args) async {
  var file = File(args[0]);
  var lines = utf8.decoder
      .bind(file.openRead())
      .transform(const LineSplitter());
  await for (final line in lines) {
    if (!line.startsWith('#')) print(line);
  }
}
```

## Method listen() {:#listen-method}

Method cuối cùng trên Stream là `listen()`. Đây là method "cấp thấp"—
tất cả hàm stream khác đều được định nghĩa dựa trên `listen()`.

<?code-excerpt "misc/lib/tutorial/stream_interface.dart (listen)"?>
```dart
StreamSubscription<T> listen(
  void Function(T event)? onData, {
  Function? onError,
  void Function()? onDone,
  bool? cancelOnError,
});
```

Để tạo một kiểu `Stream` mới, bạn chỉ cần extend class `Stream`
và implement method `listen()`—tất cả method khác
trên `Stream` đều gọi `listen()` để hoạt động.

Method `listen()` cho phép bạn bắt đầu lắng nghe một stream.
Cho đến khi bạn làm vậy,
stream là một object bất hoạt mô tả những event bạn muốn thấy.
Khi bạn lắng nghe,
một object [StreamSubscription][] được trả về, biểu diễn
stream đang hoạt động và tạo ra event.
Điều này tương tự như việc `Iterable` chỉ là một collection các object,
nhưng iterator mới là thành phần thực sự thực hiện phép lặp.

Stream subscription cho phép bạn tạm dừng subscription,
tiếp tục sau khi tạm dừng
và hủy hoàn toàn.
Bạn có thể đặt callback để được gọi cho từng event dữ liệu hoặc
event lỗi, cũng như khi stream đóng.

<a id="other-resources"></a>
## Tài nguyên khác

Đọc các tài liệu sau để biết thêm chi tiết về cách sử dụng stream
và lập trình bất đồng bộ trong Dart.

* [Tạo Stream trong Dart](/libraries/async/creating-streams),
  một bài viết về cách tạo stream của riêng bạn
* [Future và xử lý lỗi](/libraries/async/futures-error-handling),
  một bài viết giải thích cách xử lý lỗi bằng Future API
* [Lập trình bất đồng bộ](/language/async),
  trình bày chuyên sâu về hỗ trợ bất đồng bộ của ngôn ngữ Dart
* [Tài liệu tham khảo Stream API]({{site.dart-api}}/dart-async/Stream-class.html)

[bind()]: {{site.dart-api}}/dart-async/StreamTransformer/bind.html
[LineSplitter]: {{site.dart-api}}/dart-convert/LineSplitter-class.html
[Future]: {{site.dart-api}}/dart-async/Future-class.html
[Iterable]: {{site.dart-api}}/dart-core/Iterable-class.html
[Stream]: {{site.dart-api}}/dart-async/Stream-class.html
[StreamSubscription]: {{site.dart-api}}/dart-async/StreamSubscription-class.html
[StreamTransformer]: {{site.dart-api}}/dart-async/StreamTransformer-class.html
[Utf8Decoder]: {{site.dart-api}}/dart-convert/Utf8Decoder-class.html
