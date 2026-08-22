---
title: dart:io
description: Tìm hiểu về các tính năng chính trong thư viện dart:io của Dart.
prevpage:
  url: /libraries/dart-convert
  title: dart:convert
nextpage:
  url: /interop/js-interop/
  title: dart:js_interop
---

<?code-excerpt plaster="none"?>

Thư viện [dart:io][] cung cấp các API để làm việc với
file, directory, process, socket, WebSocket, cũng như HTTP client và server.

:::important
Chỉ các [ứng dụng Flutter]({{site.flutter}}) không chạy trên web, script dòng lệnh
và server mới có thể import và sử dụng `dart:io`; ứng dụng web thì không.
:::

Nhìn chung, thư viện dart:io triển khai và khuyến khích sử dụng API bất đồng bộ.
Các method đồng bộ có thể dễ dàng chặn một ứng dụng, khiến ứng dụng
khó mở rộng. Vì vậy, phần lớn thao tác trả về kết quả thông qua object Future
hoặc Stream—một pattern phổ biến trên các nền tảng server hiện đại như
Node.js.

Một số ít method đồng bộ trong thư viện dart:io được đánh dấu rõ ràng
bằng hậu tố Sync trong tên method. Các method đồng bộ không được đề cập ở đây.

Để sử dụng thư viện dart:io, bạn phải import thư viện này:

<?code-excerpt "misc/test/library_tour/io_test.dart (import)"?>
```dart
import 'dart:io';
```

<a id="files-and-directories"></a>
## File và directory

Thư viện I/O cho phép các ứng dụng dòng lệnh đọc và ghi file cũng như
duyệt các directory. Bạn có hai lựa chọn để đọc nội dung của một
file: đọc tất cả cùng lúc hoặc đọc theo stream. Việc đọc toàn bộ file cùng lúc đòi hỏi
đủ bộ nhớ để lưu tất cả nội dung của file. Nếu file rất
lớn hoặc bạn muốn xử lý file trong khi đọc, bạn nên sử dụng
Stream, như mô tả trong phần
[Đọc nội dung file theo stream](#streaming-file-contents).

<a id="reading-a-file-as-text"></a>
### Đọc file dưới dạng văn bản

Khi đọc một file văn bản được mã hóa bằng UTF-8, bạn có thể đọc toàn bộ
nội dung file bằng `readAsString()`. Khi từng dòng riêng lẻ
đóng vai trò quan trọng, bạn có thể sử dụng `readAsLines()`. Trong cả hai trường hợp, một object Future
được trả về và cung cấp nội dung file dưới dạng một hoặc nhiều
string.

<?code-excerpt "misc/test/library_tour/io_test.dart (read-as-string)" replace="/\btest_data\///g"?>
```dart
void main() async {
  var config = File('config.txt');

  // Đưa toàn bộ file vào một string duy nhất.
  var stringContents = await config.readAsString();
  print('The file is ${stringContents.length} characters long.');

  // Đưa mỗi dòng của file vào một string riêng.
  var lines = await config.readAsLines();
  print('The file is ${lines.length} lines long.');
}
```


<a id="reading-a-file-as-binary"></a>
### Đọc file dưới dạng nhị phân

Đoạn code sau đọc toàn bộ file dưới dạng các byte vào một danh sách các int.
Lệnh gọi `readAsBytes()` trả về một Future, cung cấp kết quả
khi kết quả sẵn sàng.

<?code-excerpt "misc/test/library_tour/io_test.dart (read-as-bytes)" replace="/\btest_data\///g"?>
```dart
void main() async {
  var config = File('config.txt');

  var contents = await config.readAsBytes();
  print('The file is ${contents.length} bytes long.');
}
```

<a id="handling-errors"></a>
### Xử lý lỗi

Để bắt lỗi nhằm tránh chúng dẫn đến các exception không được bắt, bạn có thể
đăng ký một handler `catchError` trên Future,
hoặc (trong một hàm `async`) sử dụng try-catch:

<?code-excerpt "misc/test/library_tour/io_test.dart (try-catch)" replace="/does-not-exist/config/g"?>
```dart
void main() async {
  var config = File('config.txt');
  try {
    var contents = await config.readAsString();
    print(contents);
  } catch (e) {
    print(e);
  }
}
```

<a id="streaming-file-contents"></a>
### Đọc nội dung file theo stream

Sử dụng Stream để đọc từng phần nhỏ của file.
Bạn có thể sử dụng [Stream API](/libraries/dart-async#stream)
hoặc `await for`, một phần trong
[hỗ trợ bất đồng bộ](/language/async) của Dart.

<?code-excerpt "misc/test/library_tour/io_test.dart (read-from-stream)" replace="/_?test_\w*\/?//g"?>
```dart
import 'dart:io';
import 'dart:convert';

void main() async {
  var config = File('config.txt');
  Stream<List<int>> inputStream = config.openRead();

  var lines = utf8.decoder.bind(inputStream).transform(const LineSplitter());
  try {
    await for (final line in lines) {
      print('Got ${line.length} characters from stream');
    }
    print('file is now closed');
  } catch (e) {
    print(e);
  }
}
```

<a id="writing-file-contents"></a>
### Ghi nội dung file

Bạn có thể sử dụng [IOSink][] để
ghi dữ liệu vào một file. Sử dụng method `openWrite()` của File để lấy một IOSink
mà bạn có thể ghi vào. Chế độ mặc định, `FileMode.write`, ghi đè hoàn toàn
dữ liệu hiện có trong file.

<?code-excerpt "misc/test/library_tour/io_test.dart (write-file)" replace="/\btest_data\///g"?>
```dart
var logFile = File('log.txt');
var sink = logFile.openWrite();
sink.write('FILE ACCESSED ${DateTime.now()}\n');
await sink.flush();
await sink.close();
```

Để thêm dữ liệu vào cuối file, hãy sử dụng tham số `mode` tùy chọn để
chỉ định `FileMode.append`:

<?code-excerpt "misc/test/library_tour/io_test.dart (append)" replace="/_?test_\w*\/?//g"?>
```dart
var sink = logFile.openWrite(mode: FileMode.append);
```

Để ghi dữ liệu nhị phân, hãy sử dụng `add(List<int> data)`.


<a id="listing-files-in-a-directory"></a>
### Liệt kê các file trong một directory

Việc tìm tất cả file và subdirectory trong một directory là một thao tác
bất đồng bộ. Method `list()` trả về một Stream phát ra một object
khi gặp một file hoặc directory.

<?code-excerpt "misc/test/library_tour/io_test.dart (list-dir)" replace="/\btest_data\b/tmp/g"?>
```dart
void main() async {
  var dir = Directory('tmp');

  try {
    var dirList = dir.list();
    await for (final FileSystemEntity f in dirList) {
      if (f is File) {
        print('Found file ${f.path}');
      } else if (f is Directory) {
        print('Found dir ${f.path}');
      }
    }
  } catch (e) {
    print(e.toString());
  }
}
```


<a id="other-common-functionality"></a>
### Các chức năng phổ biến khác

Các class File và Directory chứa các chức năng khác, bao gồm nhưng không
giới hạn ở:

- Tạo một file hoặc directory: `create()` trong File và Directory
- Xóa một file hoặc directory: `delete()` trong File và Directory
- Lấy độ dài của một file: `length()` trong File
- Truy cập ngẫu nhiên vào một file: `open()` trong File

Tham khảo tài liệu API cho [File][] và [Directory][] để xem danh sách
đầy đủ các method.


<a id="http-clients-and-servers"></a>
## HTTP client và server

Thư viện dart:io cung cấp các class mà ứng dụng dòng lệnh có thể sử dụng để
truy cập tài nguyên HTTP, cũng như để chạy HTTP server.

<a id="http-server"></a>
### HTTP server

Class [HttpServer][]
cung cấp chức năng cấp thấp để xây dựng web server. Bạn có thể
khớp các request handler, thiết lập header, stream dữ liệu và nhiều chức năng khác.

Web server mẫu sau trả về thông tin văn bản đơn giản.
Server này lắng nghe tại cổng 8888 và địa chỉ 127.0.0.1 (localhost),
phản hồi các request cho path `/dart`. Với bất kỳ path nào khác,
response là mã trạng thái 404 (không tìm thấy trang).

<?code-excerpt "misc/lib/library_tour/io/http_server.dart (process-requests)" replace="/Future<\w+\W/void/g; /\b_//g"?>
```dart
void main() async {
  final requests = await HttpServer.bind('localhost', 8888);
  await for (final request in requests) {
    processRequest(request);
  }
}

void processRequest(HttpRequest request) {
  print('Got request for ${request.uri.path}');
  final response = request.response;
  if (request.uri.path == '/dart') {
    response
      ..headers.contentType = ContentType('text', 'plain')
      ..write('Hello from the server');
  } else {
    response.statusCode = HttpStatus.notFound;
  }
  response.close();
}
```

<a id="http-client"></a>
### HTTP client

Bạn nên tránh sử dụng trực tiếp `dart:io` để tạo các HTTP request.
Class [HttpClient][] trong `dart:io` phụ thuộc vào nền tảng
và bị gắn với một implementation duy nhất.
Thay vào đó, hãy sử dụng một thư viện cấp cao hơn như
[`package:http`]({{site.pub-pkg}}/http).

Hướng dẫn [Lấy dữ liệu từ internet][]
giải thích cách tạo HTTP request
bằng `package:http`.

<a id="more-information"></a>
## Thông tin thêm

Trang này đã trình bày cách sử dụng các tính năng chính của thư viện [dart:io][].
Ngoài các API được thảo luận trong phần này, thư viện dart:io còn
cung cấp API cho [process,][Process] [socket,][Socket] và
[web socket.][WebSocket]
Để biết thêm thông tin về phát triển ứng dụng phía server và ứng dụng dòng lệnh, hãy xem
[tổng quan về Dart phía server.](/server)


[dart:io]: {{site.dart-api}}/dart-io/dart-io-library.html
[Directory]: {{site.dart-api}}/dart-io/Directory-class.html
[Fetch data from the internet]: /server/fetch-data
[File]: {{site.dart-api}}/dart-io/File-class.html
[HttpClient]: {{site.dart-api}}/dart-io/HttpClient-class.html
[HttpRequest]: {{site.dart-api}}/dart-html/HttpRequest-class.html
[HttpServer]: {{site.dart-api}}/dart-io/HttpServer-class.html
[IOSink]: {{site.dart-api}}/dart-io/IOSink-class.html
[Process]: {{site.dart-api}}/dart-io/Process-class.html
[Socket]: {{site.dart-api}}/dart-io/Socket-class.html
[WebSocket]: {{site.dart-api}}/dart-io/WebSocket-class.html
