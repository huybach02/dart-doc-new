---
title: dart:convert
description: Tìm hiểu về các tính năng chính trong thư viện dart:convert của Dart.
prevpage:
  url: /libraries/dart-math
  title: dart:math
nextpage:
  url: /libraries/dart-io
  title: dart:io
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g"?>
<?code-excerpt plaster="none"?>

Thư viện dart:convert ([tài liệu tham khảo API][dart:convert])
có các converter cho JSON và UTF-8, đồng thời hỗ trợ việc tạo
thêm các converter. [JSON][] là một định dạng văn bản đơn giản để biểu diễn
các object và collection có cấu trúc. [UTF-8][] là một kiểu mã hóa có độ rộng biến đổi phổ biến,
có thể biểu diễn mọi ký tự trong bộ ký tự Unicode.

Để sử dụng thư viện này, hãy import dart:convert.

<?code-excerpt "misc/test/library_tour/convert_test.dart (import)"?>
```dart
import 'dart:convert';
```


<a id="decoding-and-encoding-json"></a>
## Giải mã và mã hóa JSON

Giải mã một string được mã hóa JSON thành object Dart bằng `jsonDecode()`:

<?code-excerpt "misc/test/library_tour/convert_test.dart (json-decode)"?>
```dart
// LƯU Ý: Hãy nhớ sử dụng dấu ngoặc kép ("),
// không phải dấu ngoặc đơn ('), bên trong string JSON.
// String này là JSON, không phải Dart.
var jsonString = '''
  [
    {"score": 40},
    {"score": 80}
  ]
''';

var scores = jsonDecode(jsonString);
assert(scores is List);

var firstScore = scores[0];
assert(firstScore is Map);
assert(firstScore['score'] == 40);
```

Mã hóa một object Dart được hỗ trợ thành string có định dạng JSON bằng
`jsonEncode()`:

<?code-excerpt "misc/test/library_tour/convert_test.dart (json-encode)"?>
```dart
var scores = [
  {'score': 40},
  {'score': 80},
  {'score': 100, 'overtime': true, 'special_guest': null},
];

var jsonText = jsonEncode(scores);
assert(
  jsonText ==
      '[{"score":40},{"score":80},'
          '{"score":100,"overtime":true,'
          '"special_guest":null}]',
);
```

Chỉ các object thuộc kiểu int, double, String, bool, null, List hoặc Map (với
key là string) mới có thể được mã hóa trực tiếp thành JSON. Các object List và Map được
mã hóa đệ quy.

Bạn có hai lựa chọn để mã hóa những object không thể được mã hóa trực tiếp.
Lựa chọn thứ nhất là gọi `jsonEncode()` với đối số thứ hai: một
hàm trả về một object có thể được mã hóa trực tiếp. Lựa chọn thứ hai
là bỏ qua đối số thứ hai; trong trường hợp đó, encoder sẽ gọi
method `toJson()` của object.

Để xem thêm ví dụ và các liên kết đến những package liên quan đến JSON, hãy xem
[Sử dụng JSON](/libraries/serialization/json).


<a id="decoding-and-encoding-utf-8-characters"></a>
## Giải mã và mã hóa các ký tự UTF-8

Sử dụng `utf8.decode()` để giải mã các byte được mã hóa UTF8 thành một string Dart:

<?code-excerpt "misc/test/library_tour/convert_test.dart (utf8-decode)" replace="/ \/\/line-br.*//g"?>
```dart
List<int> utf8Bytes = [
  0xc3, 0x8e, 0xc3, 0xb1, 0xc5, 0xa3, 0xc3, 0xa9,
  0x72, 0xc3, 0xb1, 0xc3, 0xa5, 0xc5, 0xa3, 0xc3,
  0xae, 0xc3, 0xb6, 0xc3, 0xb1, 0xc3, 0xa5, 0xc4,
  0xbc, 0xc3, 0xae, 0xc5, 0xbe, 0xc3, 0xa5, 0xc5,
  0xa3, 0xc3, 0xae, 0xe1, 0xbb, 0x9d, 0xc3, 0xb1,
];

var funnyWord = utf8.decode(utf8Bytes);

assert(funnyWord == 'Îñţérñåţîöñåļîžåţîờñ');
```

Để chuyển đổi một stream ký tự UTF-8 thành một string Dart, hãy chỉ định
`utf8.decoder` cho method `transform()` của Stream:

<?code-excerpt "misc/test/library_tour/io_test.dart (utf8-decoder)" replace="/utf8.decoder/[!$&!]/g"?>
```dart
var lines = [!utf8.decoder!].bind(inputStream).transform(const LineSplitter());
try {
  await for (final line in lines) {
    print('Got ${line.length} characters from stream');
  }
  print('file is now closed');
} catch (e) {
  print(e);
}
```

Sử dụng `utf8.encode()` để mã hóa một string Dart thành danh sách các byte
được mã hóa UTF8:

<?code-excerpt "misc/test/library_tour/convert_test.dart (utf8-encode)" replace="/ \/\/line-br.*//g"?>
```dart
Uint8List encoded = utf8.encode('Îñţérñåţîöñåļîžåţîờñ');

assert(encoded.length == utf8Bytes.length);
for (int i = 0; i < encoded.length; i++) {
  assert(encoded[i] == utf8Bytes[i]);
}
```


<a id="other-functionality"></a>
## Chức năng khác

Thư viện dart:convert cũng có các converter cho ASCII và ISO-8859-1
(Latin1). Để biết chi tiết, hãy xem [tài liệu tham khảo API cho thư viện dart:convert.][dart:convert]

[JSON]: https://www.json.org/
[UTF-8]: https://en.wikipedia.org/wiki/UTF-8
[dart:convert]: {{site.dart-api}}/dart-convert/dart-convert-library.html
