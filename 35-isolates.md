---
title: Isolates
description: Thông tin về cách viết isolate trong Dart.
shortTitle: Isolate
lastVerified: 2024-01-04
prevpage:
  url: /language/async
  title: Hỗ trợ bất đồng bộ
nextpage:
  url: /null-safety
  title: Sound null safety
---

<?code-excerpt path-base="concurrency"?>

Trang này thảo luận một số ví dụ sử dụng API `Isolate` để implement
isolate.

Bạn nên dùng isolate bất cứ khi nào ứng dụng xử lý các phép tính
đủ lớn để tạm thời block những phép tính khác.
Ví dụ phổ biến nhất là trong ứng dụng [Flutter][], khi bạn
cần thực hiện phép tính lớn mà nếu không có isolate có thể khiến
UI không phản hồi.

:::flutter-note
Flutter web không hỗ trợ nhiều isolate
Xem thêm: [Concurrency trên web](https://dart.dev/language/concurrency#concurrency-on-the-web)
:::

Không có quy tắc nào về thời điểm bạn _bắt buộc_ phải dùng isolate,
nhưng sau đây là một số tình huống khác mà chúng có thể hữu ích:

- Parse và decode JSON blob cực lớn.
- Xử lý và nén ảnh, audio và video.
- Chuyển đổi file audio và video.
- Thực hiện tìm kiếm và lọc phức tạp trên list lớn hoặc bên trong
  filesystem.
- Thực hiện I/O, chẳng hạn giao tiếp với database.
- Xử lý số lượng network request lớn.

[Flutter]: {{site.flutter-docs}}/perf/isolates

<a id="implementing-a-simple-worker-isolate"></a>
## Implement worker isolate đơn giản

Các ví dụ này implement main isolate
spawn một worker isolate đơn giản.
[`Isolate.run()`][] đơn giản hóa các bước phía sau việc
thiết lập và quản lý worker isolate:

1. Spawn (khởi động và tạo) một isolate.
2. Chạy một function trên isolate được spawn.
3. Thu thập kết quả.
4. Trả kết quả về main isolate.
5. Chấm dứt isolate sau khi công việc hoàn tất.
6. Kiểm tra, thu thập rồi throw exception và error trở lại main isolate.

[`Isolate.run()`]: {{site.dart-api}}/dart-isolate/Isolate/run.html

:::flutter-note
Nếu đang dùng Flutter, bạn có thể dùng [function `compute` của Flutter][Flutter's `compute` function]
thay cho `Isolate.run()`.
:::

[Flutter's `compute` function]: {{site.flutter-api}}/flutter/foundation/compute.html

<a id="running-an-existing-method-in-a-new-isolate"></a>
### Chạy một method hiện có trong isolate mới

1. Gọi `run()` để spawn một isolate mới (một [background worker][]),
   trực tiếp trong [main isolate][] trong khi `main()` chờ kết quả:

<?code-excerpt "lib/simple_worker_isolate.dart (main)"?>
```dart
const String filename = 'with_keys.json';

void main() async {
  // Đọc một số dữ liệu.
  final jsonData = await Isolate.run(_readAndParseJson);

  // Dùng dữ liệu đó.
  print('Number of JSON keys: ${jsonData.length}');
}
```

2. Truyền cho worker isolate function bạn muốn nó thực thi
   làm argument đầu tiên. Trong ví dụ này, đó là function hiện có `_readAndParseJson()`:

<?code-excerpt "lib/simple_worker_isolate.dart (spawned)"?>
```dart
Future<Map<String, dynamic>> _readAndParseJson() async {
  final fileData = await File(filename).readAsString();
  final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
  return jsonData;
}
```

3. `Isolate.run()` nhận kết quả do `_readAndParseJson()` trả về
   và gửi giá trị trở lại main isolate,
   đồng thời tắt worker isolate.

4. Worker isolate *chuyển giao* bộ nhớ chứa kết quả
   cho main isolate. Nó *không sao chép* dữ liệu.
   Worker isolate thực hiện một lượt xác minh để bảo đảm
   các object được phép chuyển giao.

`_readAndParseJson()` là một function bất đồng bộ hiện có,
cũng có thể dễ dàng chạy trực tiếp trong main isolate.
Việc dùng `Isolate.run()` để chạy nó thay thế sẽ hỗ trợ concurrency.
Worker isolate abstract hóa hoàn toàn các phép tính
của `_readAndParseJson()`. Nó có thể hoàn tất mà không block main isolate.

Kết quả của `Isolate.run()` luôn là Future,
vì code trong main isolate tiếp tục chạy.
Việc phép tính mà worker isolate thực thi là đồng bộ hay bất đồng bộ
không ảnh hưởng đến main isolate,
vì trong cả hai trường hợp nó đều chạy concurrent.

Để xem chương trình hoàn chỉnh, hãy xem ví dụ [send_and_receive.dart][].

[send_and_receive.dart]: {{site.repo.dart.samples}}/blob/main/isolates/bin/send_and_receive.dart
[background worker]: /language/concurrency#background-workers
[main isolate]: /language/concurrency#the-main-isolate

<a id="sending-closures-with-isolates"></a>
### Gửi closure bằng isolate

Bạn cũng có thể tạo worker isolate đơn giản bằng `run()` và dùng
function literal hoặc closure trực tiếp trong main isolate.

<?code-excerpt "lib/simple_isolate_closure.dart (worker)"?>
```dart
const String filename = 'with_keys.json';

void main() async {
  // Đọc một số dữ liệu.
  final jsonData = await Isolate.run(() async {
    final fileData = await File(filename).readAsString();
    final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
    return jsonData;
  });

  // Dùng dữ liệu đó.
  print('Number of JSON keys: ${jsonData.length}');
}
```

Ví dụ này đạt được kết quả giống ví dụ trước.
Một isolate mới được spawn, tính toán một thứ gì đó rồi gửi kết quả trở lại.

Tuy nhiên, giờ isolate gửi một [closure][].
Closure ít bị hạn chế hơn function có tên điển hình,
cả về cách hoạt động lẫn cách được viết trong code.
Trong ví dụ này, `Isolate.run()` thực thi phần trông giống code local
theo cách concurrent. Theo nghĩa đó, bạn có thể hình dung `run()` hoạt động như
một operator control flow cho “chạy song song”.

[closure]: /language/functions#anonymous-functions

<a id="sending-multiple-messages-between-isolates-with-ports"></a>
## Gửi nhiều message giữa các isolate bằng port

Isolate tồn tại ngắn hạn thuận tiện để sử dụng,
nhưng phát sinh overhead hiệu năng để spawn isolate mới
và sao chép object từ isolate này sang isolate khác.
Nếu code phụ thuộc vào việc liên tục chạy cùng một phép tính
bằng `Isolate.run`, bạn có thể cải thiện hiệu năng bằng cách tạo
isolate tồn tại lâu, không thoát ngay lập tức.

Để làm vậy, bạn có thể dùng một số API isolate cấp thấp mà
`Isolate.run` abstract hóa:

* [`Isolate.spawn()`][] và [`Isolate.exit()`][]
* [`ReceivePort`][] và [`SendPort`][]
* [Method `SendPort.send()`][`SendPort.send()` method]


Mục này trình bày các bước cần thiết để thiết lập
giao tiếp hai chiều giữa isolate mới được spawn
và [main isolate][].
Ví dụ đầu tiên, [Port cơ bản](#basic-ports-example), giới thiệu process
ở mức khái quát. Ví dụ thứ hai, [Port vững chắc](#robust-ports-example),
dần bổ sung thêm chức năng thực tế cho ví dụ đầu tiên.

[`Isolate.exit()`]: {{site.dart-api}}/dart-isolate/Isolate/exit.html
[`Isolate.spawn()`]: {{site.dart-api}}/dart-isolate/Isolate/spawn.html
[`ReceivePort`]: {{site.dart-api}}/dart-isolate/ReceivePort-class.html
[`SendPort`]: {{site.dart-api}}/dart-isolate/SendPort-class.html
[`SendPort.send()` method]: {{site.dart-api}}/dart-isolate/SendPort/send.html
[main isolate]: /language/concurrency#isolates


<a id="receiveport-and-sendport"></a>
### `ReceivePort` và `SendPort`

Việc thiết lập giao tiếp tồn tại lâu giữa các isolate cần
hai lớp (ngoài `Isolate`): `ReceivePort` và `SendPort`.
Những port này là cách duy nhất để isolate giao tiếp với nhau.

`ReceivePort` là object xử lý message được gửi từ isolate khác.
Những message đó được gửi qua `SendPort`.

:::note
Một object `SendPort` được liên kết với đúng một `ReceivePort`,
nhưng một `ReceivePort` có thể có nhiều `SendPort`.
Khi tạo `ReceivePort`, nó tự tạo một `SendPort` cho chính mình.
Bạn có thể tạo thêm `SendPort`
có khả năng gửi message đến một `ReceivePort` hiện có.
:::

Port hoạt động tương tự object [`Stream`][]
(thực tế, receive port implement `Stream`!)
Bạn có thể hình dung `SendPort` và `ReceivePort` lần lượt giống
`StreamController` và listener của Stream.
`SendPort` giống `StreamController` vì bạn “thêm” message vào nó
bằng [method `SendPort.send()`][`SendPort.send()` method], và những message đó được listener xử lý,
trong trường hợp này là `ReceivePort`. Sau đó `ReceivePort` xử lý message nhận được
bằng cách truyền chúng làm argument cho callback bạn cung cấp.

<a id="setting-up-ports"></a>
#### Thiết lập port

Isolate mới được spawn chỉ có thông tin nó nhận qua lời gọi
`Isolate.spawn`. Nếu cần main isolate tiếp tục giao tiếp
với isolate được spawn sau thời điểm tạo ban đầu, bạn phải thiết lập một
kênh giao tiếp nơi isolate được spawn có thể gửi message đến
main isolate. Isolate chỉ có thể giao tiếp bằng cách truyền message.
Chúng không thể “nhìn thấy” bên trong bộ nhớ của nhau,
đây là nguồn gốc của tên gọi “isolate”.

Để thiết lập giao tiếp hai chiều này, trước tiên hãy tạo một [`ReceivePort`][]
trong main isolate, sau đó truyền [`SendPort`][] của nó làm argument cho
isolate mới khi spawn bằng `Isolate.spawn`.
Sau đó isolate mới tạo `ReceivePort` riêng và gửi _`SendPort` của nó_
trở lại qua `SendPort` được main isolate truyền cho nó.
Main isolate nhận `SendPort` này và
giờ cả hai phía đều có một kênh mở để gửi và nhận message.

:::note
Các sơ đồ trong mục này mang tính khái quát và nhằm truyền đạt
_khái niệm_ dùng port cho isolate. Implementation thực tế cần
thêm một chút code mà bạn sẽ tìm thấy
[ở phần sau của trang này](#basic-ports-example).
:::

![Hình minh họa quá trình thiết lập port giữa các isolate](/assets/img/language/concurrency/ports-setup.png){:.diagram-wrap}

1. Tạo một `ReceivePort` trong main isolate. `SendPort` được tự động tạo
   dưới dạng property trên `ReceivePort`.
2. Spawn worker isolate bằng `Isolate.spawn()`.
3. Truyền tham chiếu đến `ReceivePort.sendPort` làm message đầu tiên cho
   worker isolate.
4. Tạo một `ReceivePort` mới khác trong worker isolate.
5. Truyền tham chiếu đến `ReceivePort.sendPort` của worker isolate làm
   message đầu tiên gửi _trở lại_ main isolate.

Cùng với việc tạo port và thiết lập giao tiếp, bạn cũng cần
cho port biết phải làm gì khi nhận được message. Điều này được thực hiện bằng
method `listen` trên mỗi `ReceivePort` tương ứng.

![Hình minh họa quá trình truyền message qua port giữa các isolate](/assets/img/language/concurrency/ports-passing-messages.png){:.diagram-wrap}

1. Gửi message qua tham chiếu của main isolate đến `SendPort`
   của worker isolate.
2. Nhận và xử lý message thông qua listener trên `ReceivePort`
   của worker isolate. Đây là nơi phép tính bạn muốn chuyển khỏi
   main isolate được thực thi.
3. Gửi message trả về qua tham chiếu của worker isolate đến `SendPort`
   của main isolate.
4. Nhận message thông qua listener trên `ReceivePort` của main isolate.

<a id="basic-ports-example"></a>
### Ví dụ port cơ bản

Ví dụ này minh họa cách thiết lập một worker isolate tồn tại lâu
có giao tiếp hai chiều với main isolate.
Code sử dụng ví dụ gửi văn bản JSON đến isolate mới,
nơi JSON sẽ được parse và decode,
trước khi được gửi trở lại main isolate.

:::warning
Ví dụ này nhằm dạy _mức tối thiểu_ cần thiết để
spawn một isolate mới có thể gửi và nhận nhiều message theo thời gian.

Nó không bao gồm những chức năng quan trọng được kỳ vọng
trong phần mềm production như xử lý lỗi, đóng port
và sắp xếp thứ tự message.

[Ví dụ port vững chắc][robust ports example] trong mục tiếp theo đề cập những chức năng này và
thảo luận một số vấn đề có thể phát sinh khi thiếu chúng.
:::

[robust ports example]: #robust-ports-example

<a id="step-1-define-the-worker-class"></a>
#### Bước 1: Định nghĩa lớp worker

Trước tiên, hãy tạo một lớp cho background worker isolate.
Lớp này chứa mọi chức năng bạn cần để:

- Spawn một isolate.
- Gửi message đến isolate đó.
- Yêu cầu isolate decode một số JSON.
- Gửi JSON đã decode trở lại main isolate.

Lớp expose hai public method: một method spawn worker
isolate và một method xử lý việc gửi message đến worker isolate đó.

Các mục còn lại trong ví dụ này sẽ lần lượt cho bạn thấy
cách điền nội dung cho các method của lớp.

<?code-excerpt "lib/basic_ports_example/start.dart (worker)"?>
```dart
class Worker {
  Future<void> spawn() async {
    // TODO: Thêm chức năng spawn worker isolate.
  }

  void _handleResponsesFromIsolate(dynamic message) {
    // TODO: Xử lý message được gửi lại từ worker isolate.
  }

  static void _startRemoteIsolate(SendPort port) {
    // TODO: Định nghĩa code phải được thực thi trên worker isolate.
  }

  Future<void> parseJson(String message) async {
    // TODO: Định nghĩa public method có thể
    // được dùng để gửi message đến worker isolate.
  }
}
```

<a id="step-2-spawn-a-worker-isolate"></a>
#### Bước 2: Spawn worker isolate

Method `Worker.spawn` là nơi bạn nhóm code để tạo
worker isolate và bảo đảm nó có thể nhận cũng như gửi message.

- Trước tiên, tạo một `ReceivePort`. Điều này cho phép main isolate nhận
  message được gửi từ worker isolate mới spawn.
- Tiếp theo, thêm listener vào receive port để xử lý message worker isolate
  sẽ gửi lại. Callback được truyền cho
  listener là `_handleResponsesFromIsolate`, sẽ được đề cập
  trong [bước 4](#step-4-handle-messages-on-the-main-isolate).
- Cuối cùng, spawn worker isolate bằng `Isolate.spawn`. Method này nhận hai
  argument: function sẽ được thực thi trên worker isolate
  (được đề cập trong [bước 3](#step-3-execute-code-on-the-worker-isolate))
  và property `sendPort` của receive port.

<?code-excerpt "lib/basic_ports_example/complete.dart (spawn)"?>
```dart
Future<void> spawn() async {
  final receivePort = ReceivePort();
  receivePort.listen(_handleResponsesFromIsolate);
  await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
}
```

Argument `receivePort.sendPort` sẽ được truyền cho
callback (`_startRemoteIsolate`) làm argument khi callback được gọi trên
worker isolate. Đây là bước đầu tiên để bảo đảm worker isolate có
cách gửi message trở lại main isolate.

<a id="step-3-execute-code-on-the-worker-isolate"></a>
#### Bước 3: Thực thi code trên worker isolate

Trong bước này, bạn định nghĩa method `_startRemoteIsolate` được gửi đến
worker isolate để thực thi khi isolate spawn. Method này giống method “main”
của worker isolate.

- Trước tiên, tạo một `ReceivePort` mới khác. Port này nhận
  message trong tương lai từ main isolate.
- Tiếp theo, gửi `SendPort` của port đó trở lại main isolate.
- Cuối cùng, thêm listener vào `ReceivePort` mới. Listener này xử lý
  message mà main isolate gửi đến worker isolate.

<?code-excerpt "lib/basic_ports_example/complete.dart (start-remote-isolate)"?>
```dart
static void _startRemoteIsolate(SendPort port) {
  final receivePort = ReceivePort();
  port.send(receivePort.sendPort);

  receivePort.listen((dynamic message) async {
    if (message is String) {
      final transformed = jsonDecode(message);
      port.send(transformed);
    }
  });
}
```

Listener trên `ReceivePort` của worker decode JSON được main isolate truyền vào,
rồi gửi JSON đã decode trở lại main isolate.

Listener này là entrypoint cho message được gửi từ main isolate đến
worker isolate. **Đây là cơ hội duy nhất để bạn cho worker isolate biết code nào
phải thực thi trong tương lai.**

<a id="step-4-handle-messages-on-the-main-isolate"></a>
#### Bước 4: Xử lý message trên main isolate

Cuối cùng, bạn cần cho main isolate biết cách xử lý message được gửi từ
worker isolate trở lại main isolate. Để làm vậy, bạn cần điền nội dung cho
method `_handleResponsesFromIsolate`. Hãy nhớ rằng method này được truyền cho
method `receivePort.listen`, như được mô tả
trong [bước 2](#step-2-spawn-a-worker-isolate):

<?code-excerpt "lib/basic_ports_example/complete.dart (spawn)"?>
```dart
Future<void> spawn() async {
  final receivePort = ReceivePort();
  receivePort.listen(_handleResponsesFromIsolate);
  await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
}
```

Cũng hãy nhớ rằng bạn đã gửi `SendPort` trở lại main isolate
trong [bước 3](#step-3-execute-code-on-the-worker-isolate). Method này xử lý việc
nhận `SendPort` đó, cũng như xử lý message trong tương lai (sẽ là
JSON đã decode).

- Trước tiên, kiểm tra message có phải `SendPort` không. Nếu đúng, gán port đó cho
  property `_sendPort` của lớp để có thể dùng gửi message sau này.
- Tiếp theo, kiểm tra message có kiểu `Map<String, dynamic>` không, đây là kiểu
  dự kiến của JSON đã decode. Nếu đúng, xử lý message đó bằng logic riêng của
  ứng dụng. Trong ví dụ này, message được in ra.

<?code-excerpt "lib/basic_ports_example/complete.dart (handle-responses)"?>
```dart
void _handleResponsesFromIsolate(dynamic message) {
  if (message is SendPort) {
    _sendPort = message;
    _isolateReady.complete();
  } else if (message is Map<String, dynamic>) {
    print(message);
  }
}
```

<a id="step-5-add-a-completer-to-ensure-your-isolate-is-set-up"></a>
#### Bước 5: Thêm completer để bảo đảm isolate đã được thiết lập

Để hoàn thiện lớp, hãy định nghĩa public method tên `parseJson`, chịu
trách nhiệm gửi message đến worker isolate. Nó cũng cần bảo đảm
message có thể được gửi trước khi isolate được thiết lập hoàn chỉnh.
Để xử lý việc này, hãy dùng [`Completer`][].

- Trước tiên, thêm một property cấp lớp có tên `_isolateReady` thuộc kiểu `Completer`.
- Tiếp theo, thêm lời gọi `complete()` trên completer trong
  method `_handleResponsesFromIsolate` (được tạo trong [bước 4](#step-4-handle-messages-on-the-main-isolate)) nếu message là
  `SendPort`.
- Cuối cùng, trong method `parseJson`, thêm `await _isolateReady.future` trước
  `_sendPort.send`. Điều này bảo đảm không message nào có thể được gửi đến
  worker isolate cho đến khi nó được spawn _và_ đã gửi `SendPort` trở lại
  main isolate.

<?code-excerpt "lib/basic_ports_example/complete.dart (parse-json)"?>
```dart
Future<void> parseJson(String message) async {
  await _isolateReady.future;
  _sendPort.send(message);
}
```

<a id="complete-example"></a>
#### Ví dụ hoàn chỉnh

<details>
  <summary>Mở rộng để xem ví dụ hoàn chỉnh</summary>

  <?code-excerpt "lib/basic_ports_example/complete.dart"?>
  ```dart
  import 'dart:async';
  import 'dart:convert';
  import 'dart:isolate';
  
  void main() async {
    final worker = Worker();
    await worker.spawn();
    await worker.parseJson('{"key":"value"}');
  }
  
  class Worker {
    late SendPort _sendPort;
    final Completer<void> _isolateReady = Completer.sync();
  
    Future<void> spawn() async {
      final receivePort = ReceivePort();
      receivePort.listen(_handleResponsesFromIsolate);
      await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
    }
  
    void _handleResponsesFromIsolate(dynamic message) {
      if (message is SendPort) {
        _sendPort = message;
        _isolateReady.complete();
      } else if (message is Map<String, dynamic>) {
        print(message);
      }
    }
  
    static void _startRemoteIsolate(SendPort port) {
      final receivePort = ReceivePort();
      port.send(receivePort.sendPort);
  
      receivePort.listen((dynamic message) async {
        if (message is String) {
          final transformed = jsonDecode(message);
          port.send(transformed);
        }
      });
    }
  
    Future<void> parseJson(String message) async {
      await _isolateReady.future;
      _sendPort.send(message);
    }
  
  }
  ```

</details>

<a id="robust-ports-example"></a>
### Ví dụ port vững chắc

[Ví dụ trước][previous example] giải thích các building block cơ bản cần để thiết lập
isolate tồn tại lâu có giao tiếp hai chiều. Như đã đề cập, ví dụ đó thiếu
một số tính năng quan trọng, chẳng hạn xử lý lỗi, khả năng đóng
port khi không còn được dùng và sự không nhất quán về thứ tự message
trong một số tình huống.

Ví dụ này mở rộng thông tin trong ví dụ đầu tiên bằng cách tạo
worker isolate tồn tại lâu có thêm những tính năng này cùng nhiều tính năng khác,
và tuân theo pattern thiết kế tốt hơn. Mặc dù code này có điểm tương đồng với ví dụ đầu,
nó không phải phần mở rộng của ví dụ đó.

:::note
Ví dụ này giả định rằng bạn đã quen với
việc thiết lập giao tiếp giữa các isolate bằng `Isolate.spawn` và port,
nội dung đã được trình bày trong [ví dụ trước][previous example].
:::

<a id="step-1-define-the-worker-class-1"></a>
#### Bước 1: Định nghĩa lớp worker

Trước tiên, hãy tạo một lớp cho background worker isolate. Lớp này chứa
mọi chức năng bạn cần để:

- Spawn một isolate.
- Gửi message đến isolate đó.
- Yêu cầu isolate decode một số JSON.
- Gửi JSON đã decode trở lại main isolate.

Lớp expose ba public method: một method tạo worker
isolate, một method xử lý việc gửi message đến worker isolate đó và một method
có thể tắt port khi chúng không còn được dùng.

<?code-excerpt "lib/robust_ports_example/start.dart (worker)"?>
```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  Future<Object?> parseJson(String message) async {
    // TODO: Bảo đảm port vẫn đang mở.
    _commands.send(message);
  }

  static Future<Worker> spawn() async {
    // TODO: Thêm chức năng tạo object Worker mới có
    // kết nối đến isolate được spawn.
    throw UnimplementedError();
  }

  Worker._(this._responses, this._commands) {
    // TODO: Khởi tạo listener receive port của main isolate.
  }

  void _handleResponsesFromIsolate(dynamic message) {
    // TODO: Xử lý message được gửi lại từ worker isolate.
  }

  static void _handleCommandsToIsolate(ReceivePort rp, SendPort sp) async {
    // TODO: Xử lý message được gửi lại từ worker isolate.
  }

  static void _startRemoteIsolate(SendPort sp) {
    // TODO: Khởi tạo port của worker isolate.
  }
}
```

:::note
Trong ví dụ này, instance `SendPort` và `ReceivePort`
tuân theo quy ước đặt tên best practice, trong đó chúng được đặt tên theo quan hệ với
main isolate. Message được gửi qua `SendPort` từ main isolate
đến worker isolate được gọi là _command_, còn message được gửi trở lại
main isolate được gọi là _response_.
:::

<a id="step-2-create-a-rawreceiveport-in-the-workerspawn-method"></a>
#### Bước 2: Tạo `RawReceivePort` trong method `Worker.spawn`

Trước khi spawn isolate, bạn cần tạo một [`RawReceivePort`][], đây là
`ReceivePort` cấp thấp hơn. Dùng `RawReceivePort` là pattern được ưu tiên
vì nó cho phép bạn tách logic khởi động isolate khỏi logic
xử lý việc truyền message trên isolate.

Trong method `Worker.spawn`:

- Trước tiên, tạo `RawReceivePort`. `ReceivePort` này chỉ chịu trách nhiệm
  nhận message ban đầu từ worker isolate, message đó sẽ là
  `SendPort`.
- Tiếp theo, tạo `Completer` cho biết khi nào isolate sẵn sàng
  nhận message. Khi hoàn thành, nó sẽ trả về một record chứa
  `ReceivePort` và `SendPort`.
- Tiếp theo, định nghĩa property `RawReceivePort.handler`. Property này là
  một `Function?` hoạt động giống `ReceivePort.listener`. Function được gọi
  khi port này nhận message.
- Bên trong function handler, gọi `connection.complete()`. Method này nhận
  một [record][] chứa `ReceivePort` và `SendPort` làm argument.
  `SendPort` là message ban đầu được gửi từ worker isolate và trong
  bước tiếp theo sẽ được gán cho `SendPort` cấp lớp tên `_commands`.
- Sau đó, tạo một `ReceivePort` mới bằng
  constructor `ReceivePort.fromRawReceivePort` và truyền
  `initPort` vào.

<?code-excerpt "lib/robust_ports_example/spawn_1.dart (worker-spawn)" plaster="none"?>
```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  static Future<Worker> spawn() async {
    // Tạo receive port và thêm initial message handler của nó.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
        ReceivePort.fromRawReceivePort(initPort),
        commandPort,
      ));
    };
  }
}
```

Bằng cách tạo `RawReceivePort` trước rồi mới tạo `ReceivePort`, bạn có thể
thêm callback mới vào `ReceivePort.listen` sau này. Ngược lại, nếu bạn
tạo ngay `ReceivePort`, bạn chỉ có thể thêm
một `listener`, vì `ReceivePort` implement [`Stream`][], thay vì
[`BroadcastStream`][].

Về thực chất, điều này cho phép bạn tách logic khởi động isolate khỏi
logic xử lý việc nhận message sau khi thiết lập giao tiếp
hoàn tất. Lợi ích này sẽ rõ ràng hơn khi logic trong các method khác
phát triển.

<a id="step-3-spawn-a-worker-isolate-with-isolate-spawn"></a>
#### Bước 3: Spawn worker isolate bằng `Isolate.spawn`

Bước này tiếp tục điền nội dung cho method `Worker.spawn`. Bạn sẽ thêm code
cần thiết để spawn isolate và trả về instance `Worker` từ lớp này.
Trong ví dụ này, lời gọi `Isolate.spawn` được bọc trong
[block `try`/`catch`][`try`/`catch` block], bảo đảm rằng nếu isolate không khởi động được,
`initPort` sẽ đóng và object `Worker` sẽ không được tạo.

- Trước tiên, thử spawn worker isolate trong block `try`/`catch`. Nếu việc spawn
  worker isolate thất bại, đóng receive port được tạo trong
  bước trước. Method được truyền cho `Isolate.spawn` sẽ được trình bày ở bước sau.
- Tiếp theo, await `connection.future`, rồi destructure send port và
  receive port từ record mà nó trả về.
- Cuối cùng, trả về instance `Worker` bằng cách gọi private constructor của nó
  và truyền vào các port từ completer đó.

<?code-excerpt "lib/robust_ports_example/spawn_2.dart (worker-spawn)" plaster="none"?>
```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  static Future<Worker> spawn() async {
    // Tạo receive port và thêm initial message handler của nó.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
        ReceivePort.fromRawReceivePort(initPort),
        commandPort,
      ));
    };
    // Spawn isolate.
    try {
      await Isolate.spawn(_startRemoteIsolate, (initPort.sendPort));
    } on Object {
      initPort.close();
      rethrow;
    }

    final (ReceivePort receivePort, SendPort sendPort) =
        await connection.future;

    return Worker._(receivePort, sendPort);
  }
}
```

Lưu ý rằng trong ví dụ này (so với [ví dụ trước][previous example]), `Worker.spawn`
hoạt động như một static constructor bất đồng bộ cho lớp này và là cách duy nhất để
tạo instance `Worker`. Điều này đơn giản hóa API, giúp code
tạo instance `Worker` gọn hơn.

<a id="step-4-complete-the-isolate-setup-process"></a>
#### Bước 4: Hoàn tất process thiết lập isolate

Trong bước này, bạn sẽ hoàn tất process thiết lập isolate cơ bản. Nội dung này gần như
hoàn toàn tương ứng với [ví dụ trước][previous example] và không có khái niệm mới.
Có một thay đổi nhỏ là code được chia thành nhiều method hơn, đây là
practice thiết kế giúp bạn sẵn sàng bổ sung thêm chức năng
trong phần còn lại của ví dụ.
Để xem hướng dẫn chuyên sâu về process thiết lập isolate cơ bản, hãy xem
[ví dụ port cơ bản](#basic-ports-example).

Trước tiên, tạo private constructor được trả về từ method `Worker.spawn`.
Trong phần thân constructor, thêm listener vào receive port được main isolate dùng,
và truyền một method chưa được định nghĩa tên `_handleResponsesFromIsolate`
cho listener đó.

<?code-excerpt "lib/robust_ports_example/step_4.dart (constructor)" plaster="none"?>
```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  Worker._(this._responses, this._commands) {
    _responses.listen(_handleResponsesFromIsolate);
  }
}
```

Tiếp theo, thêm code vào `_startRemoteIsolate` chịu trách nhiệm khởi tạo
các port trên worker isolate. [Hãy nhớ](#step-3-spawn-a-worker-isolate-with-isolate-spawn) rằng
method này được truyền cho `Isolate.spawn` trong method `Worker.spawn`, và nó sẽ
nhận `SendPort` của main isolate làm argument.

- Tạo một `ReceivePort` mới.
- Gửi `SendPort` của port đó trở lại main isolate.
- Gọi một method mới tên `_handleCommandsToIsolate` và truyền cả
  `ReceivePort` mới lẫn `SendPort` từ main isolate làm argument.

<?code-excerpt "lib/robust_ports_example/step_4.dart (start-isolate)"?>
```dart
static void _startRemoteIsolate(SendPort sendPort) {
  final receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
  _handleCommandsToIsolate(receivePort, sendPort);
}
```

Tiếp theo, thêm method `_handleCommandsToIsolate`, chịu trách nhiệm
nhận message từ main isolate, decode JSON trên worker isolate,
và gửi JSON đã decode trở lại dưới dạng response.

- Trước tiên, khai báo listener trên `ReceivePort` của worker isolate.
- Bên trong callback được thêm vào listener, thử decode JSON được truyền
  từ main isolate bên trong [block `try`/`catch`][`try`/`catch` block]. Nếu decode
  thành công, gửi JSON đã decode trở lại main isolate.
- Nếu có lỗi, gửi lại một [`RemoteError`][].

<?code-excerpt "lib/robust_ports_example/step_4.dart (handle-commands)"?>
```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
    try {
      final jsonData = jsonDecode(message as String);
      sendPort.send(jsonData);
    } catch (e) {
      sendPort.send(RemoteError(e.toString(), ''));
    }
  });
}
```

Tiếp theo, thêm code cho method `_handleResponsesFromIsolate`.

- Trước tiên, kiểm tra message có phải `RemoteError` không; nếu đúng, bạn
  nên `throw` lỗi đó.
- Nếu không, in message. Trong các bước sau, bạn sẽ cập nhật code này để
  trả về message thay vì in chúng.

<?code-excerpt "lib/robust_ports_example/step_4.dart (handle-response)"?>
```dart
void _handleResponsesFromIsolate(dynamic message) {
  if (message is RemoteError) {
    throw message;
  } else {
    print(message);
  }
}
```

Cuối cùng, thêm method `parseJson`, đây là public method cho phép
code bên ngoài gửi JSON đến worker isolate để được decode.

<?code-excerpt "lib/robust_ports_example/step_4.dart (parse-json)"?>
```dart
Future<Object?> parseJson(String message) async {
  _commands.send(message);
}
```

Bạn sẽ cập nhật method này trong bước tiếp theo.

<a id="step-5-handle-multiple-messages-at-the-same-time"></a>
#### Bước 5: Xử lý nhiều message cùng lúc

Hiện tại, nếu bạn nhanh chóng gửi message đến worker isolate, isolate sẽ
gửi response JSON đã decode theo _thứ tự chúng hoàn tất_, thay vì
thứ tự chúng được gửi. Bạn không có cách nào xác định response nào
tương ứng với message nào.

Trong bước này, bạn sẽ khắc phục vấn đề bằng cách cung cấp id cho mỗi message và
dùng object `Completer` để bảo đảm rằng khi code bên ngoài gọi `parseJson`,
response được trả về cho bên gọi đó là response chính xác.

Trước tiên, thêm hai property cấp lớp vào `Worker`:

- `Map<int, Completer<Object?>> _activeRequests`
- `int _idCounter`

<?code-excerpt "lib/robust_ports_example/step_5_add_completers.dart (vars)"?>
```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;
  // ···
}
```

Map `_activeRequests` liên kết message được gửi đến worker isolate
với một `Completer`. Key dùng trong `_activeRequests` được lấy
từ `_idCounter`, biến này sẽ tăng khi có thêm message được gửi.

Tiếp theo, cập nhật method `parseJson` để tạo completer trước khi nó gửi
message đến worker isolate.

- Trước tiên, tạo một `Completer`.
- Tiếp theo, tăng `_idCounter` để mỗi `Completer` được liên kết với một
  số duy nhất.
- Thêm entry vào map `_activeRequests`, trong đó key là giá trị hiện tại
  của `_idCounter` và completer là value.
- Gửi message đến worker isolate cùng với id. Vì bạn chỉ có thể
  gửi một giá trị qua `SendPort`, hãy bọc id và message trong một
  [record][].
- Cuối cùng, trả về future của completer, future này cuối cùng sẽ chứa
  response từ worker isolate.

<?code-excerpt "lib/robust_ports_example/step_5_add_completers.dart (parse-json)"?>
```dart
Future<Object?> parseJson(String message) async {
  final completer = Completer<Object?>.sync();
  final id = _idCounter++;
  _activeRequests[id] = completer;
  _commands.send((id, message));
  return await completer.future;
}
```

Bạn cũng cần cập nhật `_handleResponsesFromIsolate`
và `_handleCommandsToIsolate` để xử lý hệ thống này.

Trong `_handleCommandsToIsolate`, bạn cần tính đến việc `message` là
record có hai giá trị thay vì chỉ văn bản JSON. Hãy làm vậy bằng cách destructure
các giá trị từ `message`.

Sau đó, sau khi decode JSON, cập nhật lời gọi `sendPort.send` để truyền cả
id lẫn JSON đã decode trở lại main isolate, một lần nữa dùng record.

<?code-excerpt "lib/robust_ports_example/step_5_add_completers.dart (handle-commands)"?>
```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
    final (int id, String jsonText) = message as (int, String); // Mới
    try {
      final jsonData = jsonDecode(jsonText);
      sendPort.send((id, jsonData)); // Đã cập nhật
    } catch (e) {
      sendPort.send((id, RemoteError(e.toString(), '')));
    }
  });
}
```

Cuối cùng, cập nhật `_handleResponsesFromIsolate`.

- Trước tiên, một lần nữa destructure id và response từ argument message.
- Sau đó, xóa completer tương ứng với request này khỏi
  map `_activeRequests`.
- Cuối cùng, thay vì throw error hoặc in JSON đã decode, hãy hoàn tất
  completer và truyền response vào. Khi việc này hoàn tất, response sẽ
  được trả về cho code đã gọi `parseJson` trên main isolate.

<?code-excerpt "lib/robust_ports_example/step_5_add_completers.dart (handle-response)"?>
```dart
void _handleResponsesFromIsolate(dynamic message) {
  final (int id, Object? response) = message as (int, Object?); // Mới
  final completer = _activeRequests.remove(id)!; // Mới

  if (response is RemoteError) {
    completer.completeError(response); // Đã cập nhật
  } else {
    completer.complete(response); // Đã cập nhật
  }
}
```

<a id="step-6-add-functionality-to-close-the-ports"></a>
#### Bước 6: Thêm chức năng đóng port

Khi isolate không còn được code sử dụng, bạn nên đóng
port trên main isolate và worker isolate.

- Trước tiên, thêm một boolean cấp lớp theo dõi liệu port đã đóng hay chưa.
- Sau đó, thêm method `Worker.close`. Bên trong method này:
  - Cập nhật `_closed` thành true.
  - Gửi message cuối cùng đến worker isolate.
    Message này là một `String` có nội dung “shutdown”,
    nhưng nó có thể là bất kỳ object nào bạn muốn.
    Bạn sẽ dùng nó trong đoạn code tiếp theo.
- Cuối cùng, kiểm tra `_activeRequests` có rỗng không. Nếu có, hãy đóng
  `ReceivePort` của main isolate có tên `_responses`.

<?code-excerpt "lib/robust_ports_example/step_6_close_ports.dart (close)"?>
```dart
class Worker {
  bool _closed = false;
  // ···
  void close() {
    if (!_closed) {
      _closed = true;
      _commands.send('shutdown');
      if (_activeRequests.isEmpty) _responses.close();
      print('--- port closed --- ');
    }
  }
}
```

- Tiếp theo, bạn cần xử lý message “shutdown” trong worker isolate. Thêm
  code sau vào method `_handleCommandsToIsolate`. Code này sẽ kiểm tra
  message có phải `String` mang nội dung “shutdown” hay không. Nếu đúng, nó sẽ đóng
  `ReceivePort` của worker isolate rồi return.

<?code-excerpt "lib/robust_ports_example/step_6_close_ports.dart (handle-commands)"?>
```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
    // Block if mới.
    if (message == 'shutdown') {
      receivePort.close();
      return;
    }
    final (int id, String jsonText) = message as (int, String);
    try {
      final jsonData = jsonDecode(jsonText);
      sendPort.send((id, jsonData));
    } catch (e) {
      sendPort.send((id, RemoteError(e.toString(), '')));
    }
  });
}
```

- Cuối cùng, bạn nên thêm code để kiểm tra port đã đóng hay chưa trước khi thử
  gửi message. Thêm một dòng vào method `Worker.parseJson`.

<?code-excerpt "lib/robust_ports_example/step_6_close_ports.dart (parse-json)"?>
```dart
Future<Object?> parseJson(String message) async {
  if (_closed) throw StateError('Closed'); // Mới
  final completer = Completer<Object?>.sync();
  final id = _idCounter++;
  _activeRequests[id] = completer;
  _commands.send((id, message));
  return await completer.future;
}
```

<a id="complete-example-1"></a>
#### Ví dụ hoàn chỉnh

<details>
  <summary>Mở rộng tại đây để xem ví dụ đầy đủ</summary>

<?code-excerpt "lib/robust_ports_example/complete.dart"?>
```dart
import 'dart:async';
import 'dart:convert';
import 'dart:isolate';

void main() async {
  final worker = await Worker.spawn();
  print(await worker.parseJson('{"key":"value"}'));
  print(await worker.parseJson('"banana"'));
  print(await worker.parseJson('[true, false, null, 1, "string"]'));
  print(
    await Future.wait([worker.parseJson('"yes"'), worker.parseJson('"no"')]),
  );
  worker.close();
}

class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;
  bool _closed = false;

  Future<Object?> parseJson(String message) async {
    if (_closed) throw StateError('Closed');
    final completer = Completer<Object?>.sync();
    final id = _idCounter++;
    _activeRequests[id] = completer;
    _commands.send((id, message));
    return await completer.future;
  }

  static Future<Worker> spawn() async {
    // Tạo receive port và thêm initial message handler của nó.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
        ReceivePort.fromRawReceivePort(initPort),
        commandPort,
      ));
    };

    // Spawn isolate.
    try {
      await Isolate.spawn(_startRemoteIsolate, (initPort.sendPort));
    } on Object {
      initPort.close();
      rethrow;
    }

    final (ReceivePort receivePort, SendPort sendPort) =
        await connection.future;

    return Worker._(receivePort, sendPort);
  }

  Worker._(this._responses, this._commands) {
    _responses.listen(_handleResponsesFromIsolate);
  }

  void _handleResponsesFromIsolate(dynamic message) {
    final (int id, Object? response) = message as (int, Object?);
    final completer = _activeRequests.remove(id)!;

    if (response is RemoteError) {
      completer.completeError(response);
    } else {
      completer.complete(response);
    }

    if (_closed && _activeRequests.isEmpty) _responses.close();
  }

  static void _handleCommandsToIsolate(
    ReceivePort receivePort,
    SendPort sendPort,
  ) {
    receivePort.listen((message) {
      if (message == 'shutdown') {
        receivePort.close();
        return;
      }
      final (int id, String jsonText) = message as (int, String);
      try {
        final jsonData = jsonDecode(jsonText);
        sendPort.send((id, jsonData));
      } catch (e) {
        sendPort.send((id, RemoteError(e.toString(), '')));
      }
    });
  }

  static void _startRemoteIsolate(SendPort sendPort) {
    final receivePort = ReceivePort();
    sendPort.send(receivePort.sendPort);
    _handleCommandsToIsolate(receivePort, sendPort);
  }

  void close() {
    if (!_closed) {
      _closed = true;
      _commands.send('shutdown');
      if (_activeRequests.isEmpty) _responses.close();
      print('--- port closed --- ');
    }
  }
}
```

</details>

[`Isolate.exit()`]: {{site.dart-api}}/dart-isolate/Isolate/exit.html
[`Isolate.spawn()`]: {{site.dart-api}}/dart-isolate/Isolate/spawn.html
[`ReceivePort`]: {{site.dart-api}}/dart-isolate/ReceivePort-class.html
[`SendPort`]: {{site.dart-api}}/dart-isolate/SendPort-class.html
[`SendPort.send()` method]: {{site.dart-api}}/dart-isolate/SendPort/send.html
[main isolate]: /language/concurrency#isolates
[`Stream`]: {{site.dart-api}}/dart-async/Stream-class.html
[`BroadcastStream`]: {{site.dart-api}}/dart-async/BroadcastStream-class.html
[`Completer`]: {{site.dart-api}}/dart-async/Completer-class.html
[`RawReceivePort`]: {{site.dart-api}}/dart-isolate/RawReceivePort-class.html
[record]: /language/records
[previous example]: #basic-ports-example
[`try`/`catch` block]: /language/error-handling#catch
[`RemoteError`]: {{site.dart-api}}/dart-isolate/RemoteError-class.html
