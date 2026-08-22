---
title: Concurrency in Dart
description: >-
  Dùng isolate để cho phép thực thi code song song trên nhiều lõi xử lý.
shortTitle: Concurrency
lastVerified: 2023-12-14
prevpage:
  url: /language/modifier-reference
  title: Tham chiếu class modifier
nextpage:
  url: /language/async
  title: Async
---

<?code-excerpt path-base="concurrency"?>

Trang này cung cấp tổng quan khái niệm cấp cao về cách lập trình concurrent hoạt động
trong Dart. Trang giải thích event loop, các tính năng ngôn ngữ async và isolate.
Để xem ví dụ code thực tế hơn về cách sử dụng concurrency trong Dart,
hãy đọc trang [Lập trình bất đồng bộ](/language/async) và
trang [Isolate](/language/isolates).

Lập trình concurrent trong Dart đề cập đến cả API bất đồng bộ như `Future`
và `Stream`, cùng *isolate*, cho phép bạn chuyển process sang các
lõi riêng biệt.

Mọi code Dart đều chạy trong isolate, bắt đầu trong main isolate mặc định,
và tùy chọn mở rộng sang bất kỳ isolate tiếp theo nào bạn
tạo tường minh. Khi spawn một isolate mới,
nó có bộ nhớ riêng biệt và event loop riêng.
Event loop là yếu tố giúp lập trình bất đồng bộ và
concurrent trở nên khả thi trong Dart.

<a id="event-loop"></a>
## Event loop

Mô hình runtime của Dart dựa trên event loop.
Event loop chịu trách nhiệm thực thi code của chương trình,
thu thập và xử lý sự kiện, cùng nhiều việc khác.

Khi ứng dụng chạy, tất cả sự kiện được thêm vào một queue,
gọi là *event queue*.
Sự kiện có thể là bất kỳ thứ gì, từ request vẽ lại UI,
thao tác nhấn và gõ phím của user, cho đến I/O từ đĩa.
Vì ứng dụng không thể dự đoán thứ tự xảy ra của sự kiện,
event loop xử lý từng sự kiện theo thứ tự chúng được đưa vào queue.

![Hình minh họa các sự kiện được đưa lần lượt vào event loop](/assets/img/language/concurrency/event-loop.png){:.diagram-wrap}

Cách event loop hoạt động tương tự đoạn code sau:

```dart
while (eventQueue.waitForEvent()) {
  eventQueue.processNextEvent();
}
```

Event loop trong ví dụ này là đồng bộ và chạy trên một thread duy nhất.
Tuy nhiên, phần lớn ứng dụng Dart cần làm nhiều việc cùng lúc.
Ví dụ, ứng dụng client có thể cần thực thi HTTP request,
đồng thời lắng nghe user nhấn button.
Để xử lý việc này, Dart cung cấp nhiều API async,
như [Future, Stream và async-await](/language/async).
Các API này được xây dựng quanh event loop này.

Ví dụ, hãy xem việc tạo network request:

```dart
http.get('https://example.com').then((response) {
  if (response.statusCode == 200) {
    print('Success!');
  }  
}
```

Khi đoạn code này đến event loop, nó lập tức gọi
mệnh đề đầu tiên `http.get` và trả về `Future`.
Nó cũng yêu cầu event loop giữ callback trong mệnh đề `then()`
cho đến khi HTTP request được giải quyết. Khi điều đó xảy ra,
event loop phải thực thi callback đó và truyền kết quả request làm argument.

![Hình minh họa sự kiện async được thêm vào event loop và giữ callback để thực thi sau](/assets/img/language/concurrency/async-event-loop.png){:.diagram-wrap}

Nhìn chung, đây cũng là cách event loop xử lý mọi
sự kiện bất đồng bộ khác trong Dart, chẳng hạn object [`Stream`][].

[`Stream`]: {{site.dart-api}}/dart-async/Stream-class.html

<a id="asynchronous-programming"></a>
## Lập trình bất đồng bộ

Mục này tóm tắt các loại và cú pháp lập trình bất đồng bộ khác nhau trong Dart.
Nếu đã quen thuộc với `Future`, `Stream` và async-await,
bạn có thể chuyển đến [mục isolate][isolates section].

[isolates section]: #isolates

<a id="futures"></a>
### Future

`Future` biểu diễn kết quả của một thao tác bất đồng bộ mà cuối cùng
sẽ hoàn thành với một giá trị hoặc lỗi.

Trong code mẫu này, return type `Future<String>` biểu diễn một
lời hứa cuối cùng sẽ cung cấp giá trị `String` (hoặc lỗi).

<?code-excerpt "lib/future_syntax.dart (read-async)"?>
```dart
Future<String> _readFileAsync(String filename) {
  final file = File(filename);

  // .readAsString() trả về Future.
  // .then() đăng ký callback để thực thi khi `readAsString` được giải quyết.
  return file.readAsString().then((contents) {
    return contents.trim();
  });
}
```

<a id="the-async-await-syntax"></a>
### Cú pháp async-await

Các keyword `async` và `await` cung cấp một cách khai báo để định nghĩa
function bất đồng bộ và sử dụng kết quả của chúng.

Sau đây là ví dụ về code đồng bộ
bị block trong khi chờ file I/O:

<?code-excerpt "lib/sync_number_of_keys.dart (blocking)"?>
```dart
const String filename = 'with_keys.json';

void main() {
  // Đọc một số dữ liệu.
  final fileData = _readFileSync();
  final jsonData = jsonDecode(fileData);

  // Dùng dữ liệu đó.
  print('Number of JSON keys: ${jsonData.length}');
}

String _readFileSync() {
  final file = File(filename);
  final contents = file.readAsStringSync();
  return contents.trim();
}
```

Sau đây là code tương tự, nhưng có những thay đổi (được highlight) để làm nó bất đồng bộ:

<?code-excerpt "lib/async_number_of_keys.dart (non-blocking)" replace="/async|await|readAsString\(\)/[!$&!]/g; /Future<\w+\W/[!$&!]/g;"?>
```dart
const String filename = 'with_keys.json';

void main() [!async!] {
  // Đọc một số dữ liệu.
  final fileData = [!await!] _readFileAsync();
  final jsonData = jsonDecode(fileData);

  // Dùng dữ liệu đó.
  print('Number of JSON keys: ${jsonData.length}');
}

[!Future<String>!] _readFileAsync() [!async!] {
  final file = File(filename);
  final contents = [!await!] file.[!readAsString()!];
  return contents.trim();
}
```

Function `main()` dùng keyword `await` trước `_readFileAsync()`
để cho code Dart khác (chẳng hạn event handler) dùng CPU trong khi code native
(file I/O) thực thi. Dùng `await` cũng có tác dụng chuyển
`Future<String>` được `_readFileAsync()` trả về thành `String`. Do đó,
biến `contents` có kiểu ngầm định `String`.

:::note
Keyword `await` chỉ hoạt động trong function có `async` trước
phần thân function.
:::

Như hình sau cho thấy, code Dart tạm dừng trong khi `readAsString()`
thực thi code không phải Dart, trong Dart runtime hoặc hệ điều hành.
Sau khi `readAsString()` trả về giá trị, quá trình thực thi code Dart tiếp tục.

![Hình dạng flowchart cho thấy code ứng dụng thực thi từ lúc bắt đầu đến khi thoát, xen giữa là thời gian chờ I/O native](/assets/img/language/concurrency/basics-await.png){:.diagram-wrap}

<a id="streams"></a>
### Stream

Dart cũng hỗ trợ code bất đồng bộ dưới dạng stream. Stream
cung cấp giá trị trong tương lai và lặp lại theo thời gian. Một lời hứa cung cấp
chuỗi giá trị `int` theo thời gian có kiểu `Stream<int>`.

Trong ví dụ sau, stream được tạo bằng `Stream.periodic`
liên tục phát ra một giá trị `int` mới mỗi giây.

<?code-excerpt "lib/stream_syntax.dart"?>
```dart
Stream<int> stream = Stream.periodic(const Duration(seconds: 1), (i) => i * i);
```

<a id="await-for-and-yield"></a>
#### await-for và yield

Await-for là một loại vòng lặp for thực thi mỗi iteration tiếp theo của
vòng lặp khi giá trị mới được cung cấp. Nói cách khác, nó được dùng để “lặp qua”
stream. Trong ví dụ này, function `sumStream` sẽ phát ra giá trị mới
khi stream được cung cấp làm argument phát ra giá trị mới.
Keyword `yield` được dùng thay cho `return` trong function
trả về stream giá trị.

<?code-excerpt "lib/await_for_syntax.dart"?>
```dart
Stream<int> sumStream(Stream<int> stream) async* {
  var sum = 0;
  await for (final value in stream) {
    yield sum += value;
  }
}
```

Nếu muốn tìm hiểu thêm về cách dùng `async`, `await`, `Stream` và
`Future`, hãy xem [hướng dẫn lập trình bất đồng bộ][asynchronous programming tutorial].

[asynchronous programming tutorial]: /libraries/async/async-await

<a id="isolates"></a>
## Isolate

Ngoài [API bất đồng bộ](#asynchronous-programming), Dart hỗ trợ concurrency thông qua isolate.
Phần lớn thiết bị hiện đại có CPU đa lõi.
Để tận dụng nhiều lõi, developer đôi khi dùng
thread dùng chung bộ nhớ chạy concurrent. Tuy nhiên, concurrency với trạng thái dùng chung
[dễ xảy ra lỗi](https://en.wikipedia.org/wiki/Race_condition#In_software) và có thể
dẫn đến code phức tạp.

Thay vì thread, mọi code Dart đều chạy bên trong isolate.
Với isolate, code Dart có thể thực hiện nhiều task độc lập cùng lúc,
dùng thêm lõi xử lý nếu có.
Isolate giống thread hoặc process, nhưng mỗi isolate có bộ nhớ riêng
và một thread duy nhất chạy event loop.

Mỗi isolate có các field global riêng,
bảo đảm không trạng thái nào trong một isolate có thể được truy cập
từ isolate khác. Isolate chỉ có thể giao tiếp với nhau thông qua việc truyền message.
Không có trạng thái dùng chung giữa các isolate nghĩa là những vấn đề phức tạp của concurrency như
[mutex hoặc lock](https://en.wikipedia.org/wiki/Lock_(computer_science))
và [data race](https://en.wikipedia.org/wiki/Race_condition#Data_race)
sẽ không xảy ra trong Dart. Dù vậy, isolate không ngăn hoàn toàn race condition.
Để biết thêm thông tin về mô hình concurrency này, hãy đọc về
[mô hình Actor](https://en.wikipedia.org/wiki/Actor_model).

:::note Ghi chú về nền tảng
Chỉ [nền tảng Dart Native][Dart Native platform] implement isolate.
Để tìm hiểu thêm về nền tảng Dart Web,
xem mục [Concurrency trên web](#concurrency-on-the-web).
:::

[Dart Native platform]: /overview#platform

<a id="the-main-isolate"></a>
### Main isolate

Trong hầu hết trường hợp, bạn hoàn toàn không cần nghĩ về isolate. Chương trình Dart mặc định chạy
trong main isolate. Đây là thread nơi chương trình bắt đầu chạy
và thực thi, như minh họa trong hình sau:

![Hình minh họa main isolate chạy `main()`, phản hồi sự kiện rồi thoát](/assets/img/language/concurrency/basics-main-isolate.png){:.diagram-wrap}

Ngay cả chương trình chỉ có một isolate cũng có thể thực thi trơn tru. Trước khi tiếp tục đến
dòng code tiếp theo, những ứng dụng này dùng [async-await][] để chờ thao tác bất đồng bộ
hoàn tất. Một ứng dụng hoạt động tốt sẽ khởi động nhanh, đến event loop sớm nhất
có thể. Sau đó ứng dụng phản hồi nhanh từng sự kiện trong queue, dùng
thao tác bất đồng bộ khi cần.

[async-await]: /libraries/async/async-await

<a id="the-isolate-life-cycle"></a>
### Vòng đời isolate

Như hình sau cho thấy,
mỗi isolate bắt đầu bằng việc chạy một số code Dart,
chẳng hạn function `main()`.
Code Dart này có thể đăng ký một số event listener—ví dụ để
phản hồi input của user hoặc file I/O.
Khi function ban đầu của isolate trả về,
isolate vẫn tồn tại nếu cần xử lý sự kiện.
Sau khi xử lý sự kiện, isolate thoát.

![Hình tổng quát hơn cho thấy mọi isolate chạy một số code, tùy chọn phản hồi sự kiện rồi thoát](/assets/img/language/concurrency/basics-isolate.png){:.diagram-wrap}

<a id="event-handling"></a>
### Xử lý sự kiện

Trong ứng dụng client, event queue của main isolate có thể chứa request vẽ lại
và thông báo về thao tác nhấn cùng các sự kiện UI khác. Ví dụ, hình sau
minh họa một sự kiện vẽ lại, theo sau là sự kiện nhấn, rồi hai sự kiện vẽ lại.
Event loop lấy sự kiện từ queue theo thứ tự vào trước, ra trước.

![Hình minh họa các sự kiện được đưa lần lượt vào event loop](/assets/img/language/concurrency/event-loop.png){:.diagram-wrap}

Việc xử lý sự kiện diễn ra trên main isolate sau khi `main()` thoát. Trong
hình sau, sau khi `main()` thoát, main isolate xử lý sự kiện
vẽ lại đầu tiên. Sau đó, main isolate xử lý sự kiện nhấn, rồi đến một
sự kiện vẽ lại.

Nếu một thao tác đồng bộ tốn quá nhiều thời gian xử lý, ứng dụng có thể
không phản hồi. Trong hình sau, code xử lý thao tác nhấn mất quá nhiều thời gian nên
các sự kiện tiếp theo được xử lý quá muộn. Ứng dụng có thể trông như bị đóng băng và mọi
animation mà nó thực hiện có thể bị giật.

![Hình minh họa một handler thao tác nhấn có thời gian thực thi quá lâu](/assets/img/language/concurrency/event-jank.png){:.diagram-wrap}

Trong ứng dụng client, kết quả của thao tác đồng bộ quá dài thường là
[animation UI bị giật, không mượt][jank]. Tệ hơn, UI có thể hoàn toàn
không phản hồi.

[jank]: {{site.flutter-docs}}/perf/rendering-performance

<a id="background-workers"></a>
### Worker chạy nền

Nếu UI của ứng dụng không phản hồi do một phép tính tốn thời gian—
ví dụ [parse file JSON lớn][json]—hãy cân nhắc chuyển
phép tính đó sang worker isolate, thường được gọi là _background worker_.
Một trường hợp phổ biến, được minh họa trong hình sau, là spawn một worker
isolate đơn giản thực hiện phép tính rồi thoát. Worker isolate trả về
kết quả trong một message khi nó thoát.

[json]: {{site.flutter-docs}}/cookbook/networking/background-parsing

![Hình minh họa main isolate và một worker isolate đơn giản](/assets/img/language/concurrency/isolate-bg-worker.png){:.diagram-wrap}

Worker isolate có thể thực hiện I/O
(chẳng hạn đọc và ghi file), đặt timer và nhiều việc khác. Nó có
bộ nhớ riêng và không chia sẻ trạng thái với main isolate. Worker isolate có thể
block mà không ảnh hưởng đến isolate khác.

<a id="using-isolates"></a>
### Sử dụng isolate

Có hai cách làm việc với isolate trong Dart, tùy use case:

* Dùng [`Isolate.run()`][] để thực hiện một phép tính duy nhất trên thread riêng.
* Dùng [`Isolate.spawn()`][] để tạo isolate xử lý nhiều
  message theo thời gian hoặc một background worker. Để biết thêm thông tin về cách làm việc
  với isolate tồn tại lâu, hãy đọc trang [Isolate](/language/isolates).

Trong phần lớn trường hợp, `Isolate.run` là API được khuyến nghị
để chạy process trong background.

<a id="isolaterun"></a>
#### `Isolate.run()`

Method static `Isolate.run()` yêu cầu một argument: callback sẽ được
chạy trên isolate mới được spawn.

<?code-excerpt "lib/isolate_run_syntax.dart (slow)"?>
```dart
int slowFib(int n) => n <= 1 ? 1 : slowFib(n - 1) + slowFib(n - 2);

// Tính toán mà không block isolate hiện tại.
void fib40() async {
  var result = await Isolate.run(() => slowFib(40));
  print('Fib(40) = $result');
}
```

<a id="performance-and-isolate-groups"></a>
### Hiệu năng và isolate group

Khi một isolate gọi [`Isolate.spawn()`][], hai isolate có cùng
code executable và nằm trong cùng một _isolate group_. Isolate group cho phép
các tối ưu hiệu năng như chia sẻ code; isolate mới lập tức chạy
code thuộc sở hữu của isolate group. Ngoài ra, `Isolate.exit()` chỉ hoạt động khi
các isolate nằm trong cùng isolate group.

Trong một số trường hợp đặc biệt, bạn có thể cần dùng [`Isolate.spawnUri()`][], method này
thiết lập isolate mới bằng bản sao code tại URI được chỉ định.
Tuy nhiên, `spawnUri()` chậm hơn nhiều so với `spawn()` và isolate mới không
nằm trong isolate group của isolate đã spawn nó. Một hệ quả hiệu năng khác là việc truyền message
chậm hơn khi các isolate nằm trong những group khác nhau.

[`Isolate.spawnUri()`]: {{site.dart-api}}/dart-isolate/Isolate/spawnUri.html

<a id="limitations-of-isolates"></a>
### Hạn chế của isolate

<a id="isolates-arent-threads"></a>
#### Isolate không phải thread

Nếu bạn đến với Dart từ một ngôn ngữ có multithreading, việc kỳ vọng
isolate hoạt động giống thread là hợp lý, nhưng thực tế không phải vậy. Mỗi isolate
có trạng thái riêng, bảo đảm không trạng thái nào trong một isolate có thể
được truy cập từ isolate khác. Do đó, isolate bị giới hạn bởi
quyền truy cập vào bộ nhớ của chính nó.

Ví dụ, nếu bạn có một ứng dụng với
biến global mutable, biến đó sẽ là một biến riêng
trong isolate được spawn. Nếu mutate biến đó trong isolate được spawn,
biến trong main isolate vẫn không bị ảnh hưởng. Đây là cách isolate được
thiết kế để hoạt động và là điều quan trọng cần ghi nhớ khi cân nhắc
sử dụng isolate.

<a id="message-types"></a>
#### Kiểu message

Message được gửi qua [`SendPort`][]
có thể là gần như mọi kiểu object Dart, nhưng có một vài ngoại lệ:

- Object có resource native, chẳng hạn [`Socket`][].
- [`ReceivePort`][]
- [`DynamicLibrary`][]
- [`Finalizable`][]
- [`Finalizer`][]
- [`NativeFinalizer`][]
- [`Pointer`][]
- [`UserTag`][]
- Instance của lớp được đánh dấu bằng `@pragma('vm:isolate-unsendable')`

Ngoài những ngoại lệ đó, mọi object đều có thể được gửi.
Xem tài liệu [`SendPort.send`][] để biết thêm thông tin.

Lưu ý rằng `Isolate.spawn()` và `Isolate.exit()` abstract hóa object `SendPort`,
vì vậy chúng chịu cùng các hạn chế.

[`SendPort.send`]: {{site.dart-api}}/dart-isolate/SendPort/send.html
[`Socket`]: {{site.dart-api}}/dart-io/Socket-class.html
[`DynamicLibrary`]: {{site.dart-api}}/dart-ffi/DynamicLibrary-class.html
[`Finalizable`]: {{site.dart-api}}/dart-ffi/Finalizable-class.html
[`Finalizer`]: {{site.dart-api}}/dart-core/Finalizer-class.html
[`NativeFinalizer`]: {{site.dart-api}}/dart-ffi/NativeFinalizer-class.html
[`Pointer`]: {{site.dart-api}}/dart-ffi/Pointer-class.html
[`UserTag`]: {{site.dart-api}}/dart-developer/UserTag-class.html

<a id="synchronous-blocking-communication-between-isolates"></a>
#### Giao tiếp blocking đồng bộ giữa các isolate

Có giới hạn về số lượng isolate có thể chạy song song.
Giới hạn này không ảnh hưởng đến giao tiếp *bất đồng bộ* tiêu chuẩn giữa các isolate
thông qua message trong Dart. Bạn có thể có hàng trăm isolate chạy concurrent
và tiếp tục tiến triển. Isolate được schedule trên CPU theo kiểu round-robin
và thường xuyên nhường lượt cho nhau.

Isolate chỉ có thể giao tiếp *đồng bộ* bên ngoài Dart thuần túy,
bằng cách dùng code C thông qua [FFI].
Việc cố giao tiếp đồng bộ giữa các isolate
bằng cách blocking đồng bộ trong lời gọi FFI
có thể dẫn đến deadlock nếu số isolate vượt quá giới hạn,
trừ khi được xử lý đặc biệt cẩn thận.
Giới hạn không được hardcode thành một con số cụ thể,
mà được tính dựa trên kích thước heap của Dart VM khả dụng cho ứng dụng Dart.

Để tránh tình huống này, code C thực hiện blocking đồng bộ
cần rời isolate hiện tại trước khi thực hiện thao tác blocking
và vào lại isolate trước khi trả về Dart từ lời gọi FFI.
Đọc về [`Dart_EnterIsolate`] và [`Dart_ExitIsolate`] để tìm hiểu thêm.

[FFI]: /interop/c-interop
[`Dart_EnterIsolate`]: {{site.repo.dart.sdk}}/blob/c9a8bbd8d6024e419b5e5f26b5131285eb19cc93/runtime/include/dart_api.h#L1254
[`Dart_ExitIsolate`]: {{site.repo.dart.sdk}}/blob/c9a8bbd8d6024e419b5e5f26b5131285eb19cc93/runtime/include/dart_api.h#L1455

<a id="web"></a>
<a id="concurrency-on-the-web"></a>
## Concurrency trên web

Mọi ứng dụng Dart có thể dùng `async-await`, `Future` và `Stream`
cho các phép tính interleaved không blocking. Tuy nhiên, [nền tảng Dart web][Dart web platform]
không hỗ trợ isolate. Ứng dụng Dart web có thể dùng [web worker][web workers] để chạy script
trong background thread tương tự isolate. Tuy nhiên, chức năng và
khả năng của web worker phần nào khác với isolate.

Ví dụ, khi web worker gửi dữ liệu giữa các thread, chúng sao chép dữ liệu
qua lại. Tuy nhiên, sao chép dữ liệu có thể rất chậm, đặc biệt với message lớn.
Isolate cũng làm vậy, nhưng còn cung cấp API có thể _chuyển giao_
bộ nhớ chứa message hiệu quả hơn.

Việc tạo web worker và isolate cũng khác nhau. Bạn chỉ có thể tạo web worker
bằng cách khai báo entrypoint chương trình riêng và biên dịch riêng. Khởi động
web worker tương tự dùng `Isolate.spawnUri` để khởi động isolate. Bạn cũng có thể
khởi động isolate bằng `Isolate.spawn`, cách này cần ít resource hơn
vì nó
[tái sử dụng một số code và dữ liệu giống nhau](#performance-and-isolate-groups)
như isolate thực hiện spawn. Web worker không có API tương đương.

[Dart web platform]: /overview#platform
[web workers]: https://developer.mozilla.org/docs/Web/API/Web_Workers_API/Using_web_workers


<a id="additional-resources"></a>
## Tài nguyên bổ sung

- Nếu đang dùng nhiều isolate, hãy cân nhắc
  [`IsolateNameServer`][]
  trong Flutter hoặc
  [`package:isolate_name_server`][] cung cấp
  chức năng tương tự cho ứng dụng Dart không phải Flutter.
- Đọc thêm về [mô hình Actor][Actor model], nền tảng của isolate trong Dart.
- Tài liệu bổ sung về API `Isolate`:
    - [`Isolate.exit()`][]
    - [`Isolate.spawn()`][]
    - [`ReceivePort`][]
    - [`SendPort`][]

[`IsolateNameServer`]: {{site.flutter-api}}/flutter/dart-ui/IsolateNameServer-class.html
[`package:isolate_name_server`]: {{site.pub-pkg}}/isolate_name_server
[Actor model]: https://en.wikipedia.org/wiki/Actor_model
[`Isolate.run()`]: {{site.dart-api}}/dart-isolate/Isolate/run.html
[`Isolate.exit()`]: {{site.dart-api}}/dart-isolate/Isolate/exit.html
[`Isolate.spawn()`]: {{site.dart-api}}/dart-isolate/Isolate/spawn.html
[`ReceivePort`]: {{site.dart-api}}/dart-isolate/ReceivePort-class.html
[`SendPort`]: {{site.dart-api}}/dart-isolate/SendPort-class.html
