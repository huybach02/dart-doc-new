---
title: Creating streams in Dart
shortTitle: Tạo stream
description: Stream là một chuỗi kết quả; hãy tìm hiểu cách tạo stream của riêng bạn.
original-date: 2013-04-08
date: 2021-05-16
---

<style>
.comment {color:red;}
</style>

_Viết bởi Lasse Nielsen <br>
Tháng 4 năm 2013 (cập nhật tháng 5 năm 2021)_

Thư viện dart:async chứa hai kiểu
quan trọng đối với nhiều API Dart:
[Stream]({{site.dart-api}}/dart-async/Stream-class.html) và
[Future.]({{site.dart-api}}/dart-async/Future-class.html)
Trong khi Future biểu diễn kết quả của một phép tính đơn lẻ,
stream là một _chuỗi_ kết quả.
Bạn lắng nghe stream để được thông báo về kết quả
(cả dữ liệu và error)
cũng như khi stream đóng.
Bạn cũng có thể tạm dừng trong khi lắng nghe hoặc ngừng lắng nghe stream
trước khi nó hoàn thành.

Nhưng bài viết này không nói về việc _sử dụng_ stream.
Nó nói về cách tạo stream của riêng bạn.
Bạn có thể tạo stream theo một vài cách:

* Biến đổi stream hiện có.
* Tạo stream từ đầu bằng hàm `async*`.
* Tạo stream bằng `StreamController`.

Bài viết này trình bày code cho từng cách tiếp cận
và đưa ra các mẹo giúp bạn implement stream đúng cách.

Để được trợ giúp về cách sử dụng stream, hãy xem
[Lập trình bất đồng bộ: Stream](/libraries/async/using-streams).


<a id="transforming-an-existing-stream"></a>
## Biến đổi một stream hiện có

Trường hợp phổ biến khi tạo stream là bạn đã có một stream
và muốn tạo stream mới dựa trên event của stream ban đầu.
Ví dụ, bạn có thể có stream các byte và
muốn chuyển đổi nó thành stream các string bằng cách giải mã đầu vào theo UTF-8.
Cách tổng quát nhất là tạo một stream mới,
chờ event trên stream ban đầu rồi
phát ra event mới. Ví dụ:

<?code-excerpt "misc/lib/articles/creating-streams/line_stream_generator.dart (split-into-lines)"?>
```dart
/// Tách một stream các string liên tiếp thành các dòng.
///
/// String đầu vào được cung cấp theo các chunk nhỏ hơn thông qua
/// stream `source`.
Stream<String> lines(Stream<String> source) async* {
  // Lưu bất kỳ dòng chưa hoàn chỉnh nào từ chunk trước.
  var partial = '';
  // Chờ đến khi có chunk mới rồi xử lý nó.
  await for (final chunk in source) {
    var lines = chunk.split('\n');
    lines[0] = partial + lines[0]; // Thêm dòng chưa hoàn chỉnh vào đầu.
    partial = lines.removeLast(); // Xóa dòng chưa hoàn chỉnh mới.
    for (final line in lines) {
      yield line; // Thêm các dòng vào stream đầu ra.
    }
  }
  // Thêm dòng chưa hoàn chỉnh cuối cùng vào stream đầu ra, nếu có.
  if (partial.isNotEmpty) yield partial;
}
```

Với nhiều phép biến đổi phổ biến,
bạn có thể dùng các method biến đổi do `Stream` cung cấp
như `map()`, `where()`, `expand()` và `take()`.

Ví dụ, giả sử bạn có một stream, `counterStream`,
phát ra counter tăng dần mỗi giây.
Nó có thể được implement như sau:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (basic-usage)"?>
```dart
var counterStream = Stream<int>.periodic(
  const Duration(seconds: 1),
  (x) => x,
).take(15);
```

Để xem nhanh các event, bạn có thể dùng code như sau:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (basic-for-each)"?>
```dart
counterStream.forEach(print); // In một integer mỗi giây, 15 lần.
```

Để biến đổi event của stream, bạn có thể gọi một method biến đổi
như `map()` trên stream trước khi lắng nghe.
Method trả về một stream mới.

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (use-map)"?>
```dart
// Nhân đôi integer trong mỗi event.
var doubleCounterStream = counterStream.map((int x) => x * 2);
doubleCounterStream.forEach(print);
```

Thay cho `map()`, bạn có thể sử dụng bất kỳ method biến đổi nào khác,
như sau:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (use-where)"?>
```dart
.where((int x) => x.isEven) // Chỉ giữ lại event integer chẵn.
.expand((x) => [x, x]) // Nhân đôi mỗi event.
.take(5) // Dừng sau năm event đầu tiên.
```

Thông thường, một method biến đổi là tất cả những gì bạn cần.
Tuy nhiên, nếu cần kiểm soát phép biến đổi nhiều hơn nữa,
bạn có thể chỉ định một
[StreamTransformer]({{site.dart-api}}/dart-async/StreamTransformer-class.html)
bằng method `transform()` của `Stream`.
Các thư viện nền tảng cung cấp stream transformer cho nhiều tác vụ phổ biến.
Ví dụ, code sau sử dụng các transformer `utf8.decoder` và `LineSplitter`
do thư viện dart:convert cung cấp.

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (use-transform)"?>
```dart
Stream<List<int>> content = File('someFile.txt').openRead();
List<String> lines = await content
    .transform(utf8.decoder)
    .transform(const LineSplitter())
    .toList();
```


<a id="creating-a-stream-from-scratch"></a>
## Tạo stream từ đầu

Một cách tạo stream mới là dùng hàm generator bất đồng bộ (`async*`).
Stream được tạo khi hàm được gọi,
và thân hàm bắt đầu chạy khi stream được lắng nghe.
Khi hàm trả về, stream đóng.
Cho đến khi hàm trả về, nó có thể phát event trên stream bằng
các câu lệnh `yield` hoặc `yield*`.

Đây là ví dụ cơ bản phát ra các số theo khoảng thời gian đều đặn:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (async-generator)" replace="/timedCounterGenerator/timedCounter/g"?>
```dart
Stream<int> timedCounter(Duration interval, [int? maxCount]) async* {
  int i = 0;
  while (true) {
    await Future.delayed(interval);
    yield i++;
    if (i == maxCount) break;
  }
}
```

{% comment %}
[ĐANG CHỜ: trình bày code sử dụng nó để chúng ta có ngữ cảnh cho
phần đề cập đến StreamSubscription?]
{% endcomment %}

Hàm này trả về một `Stream`.
Khi stream đó được lắng nghe, thân hàm bắt đầu chạy.
Nó liên tục trì hoãn trong khoảng thời gian được yêu cầu rồi yield số tiếp theo.
Nếu tham số `maxCount` bị bỏ qua, vòng lặp không có điều kiện dừng,
do đó stream phát ra những số ngày càng lớn mãi mãi—
hoặc cho đến khi listener hủy subscription.

Khi listener hủy
(bằng cách gọi `cancel()` trên object `StreamSubscription`
do method `listen()` trả về),
lần tiếp theo thân hàm đến câu lệnh `yield`,
`yield` sẽ hoạt động như câu lệnh `return`.
Mọi block `finally` bao quanh được thực thi,
và hàm thoát.
Nếu hàm cố gắng yield một value trước khi thoát,
việc đó thất bại và hoạt động như return.

Khi hàm cuối cùng thoát, future do method `cancel()` trả về sẽ hoàn thành.
Nếu hàm thoát với error, future hoàn thành với error đó;
nếu không, nó hoàn thành với `null`.

Một ví dụ hữu ích hơn là hàm chuyển đổi
một chuỗi future thành stream:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (stream-from-futures)"?>
```dart
Stream<T> streamFromFutures<T>(Iterable<Future<T>> futures) async* {
  for (final future in futures) {
    var result = await future;
    yield result;
  }
}
```

Hàm này yêu cầu iterable `futures` cung cấp future mới,
chờ future đó, phát ra value kết quả rồi lặp lại.
Nếu một future hoàn thành với error, stream hoàn thành với error đó.

Hiếm khi hàm `async*` xây dựng stream từ hư không.
Nó cần lấy dữ liệu từ đâu đó,
và thường nơi đó là một stream khác.
Trong một số trường hợp, như chuỗi future ở trên,
dữ liệu đến từ các nguồn event bất đồng bộ khác.
Tuy nhiên, trong nhiều trường hợp, hàm `async*` quá đơn giản để
dễ dàng xử lý nhiều nguồn dữ liệu.
Đó là lúc cần đến class `StreamController`.


<a id="using-a-streamcontroller"></a>
## Sử dụng StreamController

Nếu event của stream đến từ các phần khác nhau trong chương trình,
chứ không chỉ từ stream hoặc future mà hàm `async` có thể duyệt,
hãy sử dụng
[StreamController]({{site.dart-api}}/dart-async/StreamController-class.html)
để tạo và nạp dữ liệu cho stream.

`StreamController` cung cấp cho bạn một stream mới
và cách thêm event vào stream tại bất kỳ thời điểm và vị trí nào.
Stream có toàn bộ logic cần thiết để xử lý listener và việc tạm dừng.
Bạn trả về stream và giữ controller cho riêng mình.

Ví dụ sau
(từ [stream_controller_bad.dart][])
trình bày cách sử dụng `StreamController` cơ bản nhưng có lỗi
để implement hàm `timedCounter()` từ các ví dụ trước.
Code này tạo stream để trả về,
rồi đưa dữ liệu vào stream dựa trên event timer,
vốn không phải future cũng không phải event stream.

[stream_controller_bad.dart]: {{site.repo.this}}/blob/main/examples/misc/lib/articles/creating-streams/stream_controller_bad.dart

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller_bad.dart (flawed-stream)"?>
```dart tag=bad
// LƯU Ý: Implementation này CÓ LỖI!
// Nó khởi chạy trước khi có subscriber và không implement việc tạm dừng.
Stream<int> timedCounter(Duration interval, [int? maxCount]) {
  var controller = StreamController<int>();
  int counter = 0;
  void tick(Timer timer) {
    counter++;
    controller.add(counter); // Yêu cầu stream gửi value counter làm event.
    if (maxCount != null && counter >= maxCount) {
      timer.cancel();
      controller.close(); // Yêu cầu stream đóng và báo cho listener.
    }
  }

  Timer.periodic(interval, tick); // SAI: Khởi chạy trước khi có subscriber.
  return controller.stream;
}
```

Như trước, bạn có thể sử dụng stream do `timedCounter()` trả về như sau:
{% comment %}
**[ĐANG CHỜ: Chúng ta đã trình bày điều này trước đó chưa?]**
{% endcomment %}

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller_bad.dart (using-stream)"?>
```dart
var counterStream = timedCounter(const Duration(seconds: 1), 15);
counterStream.listen(print); // In một integer mỗi giây, 15 lần.
```

Implementation `timedCounter()` này có
một vài vấn đề:

* Nó bắt đầu tạo event trước khi có subscriber.
* Nó tiếp tục tạo event ngay cả khi subscriber yêu cầu tạm dừng.

Như các phần tiếp theo trình bày,
bạn có thể khắc phục cả hai vấn đề bằng cách chỉ định
các callback như `onListen` và `onPause`
khi tạo `StreamController`.


<a id="waiting-for-a-subscription"></a>
### Chờ một subscription

Theo nguyên tắc, stream nên chờ subscriber trước khi bắt đầu công việc.
Hàm `async*` tự động làm điều này,
nhưng khi sử dụng `StreamController`,
bạn có toàn quyền kiểm soát và có thể thêm event ngay cả khi không nên.
Khi stream không có subscriber,
`StreamController` của nó buffer event,
điều có thể dẫn đến rò rỉ bộ nhớ
nếu stream không bao giờ có subscriber.

Hãy thử thay code sử dụng stream bằng đoạn sau:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller_bad.dart (pre-subscribe-problem)"?>
```dart
void listenAfterDelay() async {
  var counterStream = timedCounter(const Duration(seconds: 1), 15);
  await Future.delayed(const Duration(seconds: 5));

  // Sau 5 giây, thêm một listener.
  await for (final n in counterStream) {
    print(n); // In một integer mỗi giây, 15 lần.
  }
}
```

Khi code này chạy,
không có gì được in trong 5 giây đầu tiên,
mặc dù stream đang làm việc.
Sau đó listener được thêm,
và khoảng 5 event đầu tiên được in cùng lúc
vì chúng đã được `StreamController` buffer.

Để được thông báo về subscription, hãy chỉ định đối số
`onListen` khi tạo `StreamController`.
Callback `onListen` được gọi
khi stream có subscriber đầu tiên.
Nếu bạn chỉ định callback `onCancel`,
nó được gọi khi controller mất subscriber cuối cùng.
Trong ví dụ trên,
`Timer.periodic()`
nên được chuyển vào handler `onListen`,
như trình bày trong phần tiếp theo.


<a id="honoring-the-pause-state"></a>
### Tôn trọng trạng thái tạm dừng

Tránh tạo event khi listener đã yêu cầu tạm dừng.
Hàm `async*` tự động tạm dừng tại câu lệnh `yield`
khi stream subscription bị tạm dừng.
Ngược lại, `StreamController` buffer event trong lúc tạm dừng.
Nếu code cung cấp event không tôn trọng việc tạm dừng,
kích thước buffer có thể tăng vô hạn.
Ngoài ra, nếu listener ngừng lắng nghe ngay sau khi tạm dừng,
công sức tạo buffer sẽ bị lãng phí.

Để xem điều gì xảy ra khi không hỗ trợ tạm dừng,
hãy thử thay code sử dụng stream bằng đoạn sau:

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller_bad.dart (pause-problem)"?>
```dart
void listenWithPause() {
  var counterStream = timedCounter(const Duration(seconds: 1), 15);
  late StreamSubscription<int> subscription;

  subscription = counterStream.listen((int counter) {
    print(counter); // In một integer mỗi giây.
    if (counter == 5) {
      // Sau 5 tick, tạm dừng năm giây rồi tiếp tục.
      subscription.pause(Future.delayed(const Duration(seconds: 5)));
    }
  });
}
```

Khi năm giây tạm dừng kết thúc,
các event được phát trong thời gian đó đều được nhận cùng lúc.
Điều này xảy ra vì source của stream không tôn trọng việc tạm dừng
và tiếp tục thêm event vào stream.
Vì vậy stream buffer các event,
rồi làm rỗng buffer khi stream hết tạm dừng.

Phiên bản `timedCounter()` sau
(từ [stream_controller.dart][])
implement việc tạm dừng bằng các callback
`onListen`, `onPause`, `onResume` và `onCancel`
trên `StreamController`.

[stream_controller.dart]: {{site.repo.this}}/blob/main/examples/misc/lib/articles/creating-streams/stream_controller.dart

<?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (better-stream)"?>
```dart
Stream<int> timedCounter(Duration interval, [int? maxCount]) {
  late StreamController<int> controller;
  Timer? timer;
  int counter = 0;

  void tick(_) {
    counter++;
    controller.add(counter); // Yêu cầu stream gửi value counter làm event.
    if (counter == maxCount) {
      timer?.cancel();
      controller.close(); // Yêu cầu stream đóng và báo cho listener.
    }
  }

  void startTimer() {
    timer = Timer.periodic(interval, tick);
  }

  void stopTimer() {
    timer?.cancel();
    timer = null;
  }

  controller = StreamController<int>(
    onListen: startTimer,
    onPause: stopTimer,
    onResume: startTimer,
    onCancel: stopTimer,
  );

  return controller.stream;
}
```

Chạy code này với hàm `listenWithPause()` ở trên.
Bạn sẽ thấy nó ngừng đếm khi tạm dừng
và tiếp tục một cách trơn tru sau đó.

Bạn phải sử dụng tất cả listener—`onListen`,
`onCancel`, `onPause` và `onResume`—để
được thông báo về thay đổi trạng thái tạm dừng.
Lý do là nếu trạng thái subscription và tạm dừng
cùng thay đổi một lúc,
chỉ callback `onListen` hoặc `onCancel` được gọi.


<a id="final-hints"></a>
## Những gợi ý cuối cùng

Khi tạo stream mà không sử dụng hàm async*,
hãy ghi nhớ những mẹo sau:

* Cẩn thận khi sử dụng controller đồng bộ—ví dụ,
  controller được tạo bằng `StreamController(sync: true)`.
  Khi bạn gửi event trên controller đồng bộ không bị tạm dừng
  (ví dụ, bằng các method `add()`, `addError()` hoặc `close()` do
  [EventSink]({{site.dart-api}}/dart-async/EventSink-class.html) định nghĩa),
  event được gửi ngay lập tức đến tất cả listener trên stream.
  Listener của `Stream` tuyệt đối không được gọi cho đến khi
  code thêm listener đã trả về hoàn toàn,
  và sử dụng controller đồng bộ sai thời điểm có thể
  phá vỡ lời hứa này và khiến code tốt thất bại.
  Tránh sử dụng controller đồng bộ.

* Nếu bạn sử dụng `StreamController`,
  callback `onListen` được gọi trước khi
  lệnh gọi `listen` trả về `StreamSubscription`.
  Đừng để callback `onListen` phụ thuộc vào việc
  subscription đã tồn tại.
  Ví dụ, trong code sau,
  một event `onListen` được kích hoạt
  (và `handler` được gọi)
  trước khi biến `subscription`
  có value hợp lệ.

  <?code-excerpt "misc/lib/articles/creating-streams/stream_controller.dart (stream-listen-hint)"?>
  ```dart
  subscription = stream.listen(handler);
  ```

* Các callback `onListen`, `onPause`, `onResume` và `onCancel`
  do `StreamController` định nghĩa được
  stream gọi khi trạng thái listener của stream thay đổi,
  nhưng không bao giờ trong lúc phát event
  hoặc trong lệnh gọi một handler thay đổi trạng thái khác.
  Trong những trường hợp đó, callback thay đổi trạng thái bị trì hoãn cho đến khi
  callback trước hoàn thành.

* Đừng cố tự implement interface `Stream`.
  Rất dễ xử lý sai một cách tinh vi sự tương tác giữa event, callback,
  cũng như việc thêm và xóa listener.
  Luôn sử dụng stream hiện có, có thể từ `StreamController`,
  để implement lệnh gọi `listen` của stream mới.

* Mặc dù có thể tạo các class extend `Stream` với
  nhiều chức năng hơn bằng cách extend class `Stream`,
  implement method `listen` và thêm chức năng ở trên,
  nhưng nhìn chung điều đó không được khuyến nghị vì
  nó đưa vào một kiểu mới mà người dùng phải cân nhắc.
  Thay vì một class _là_ `Stream` (và hơn thế nữa),
  bạn thường có thể tạo một class _có_ `Stream` (và hơn thế nữa).

{% comment %}
Các test cho bài viết này nằm tại /src/tests/site/articles/creating-streams.
{% endcomment %}
