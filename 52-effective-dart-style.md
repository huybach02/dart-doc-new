---
title: "Effective Dart: Style"
breadcrumb: Style
description: Quy tắc format và đặt tên để code nhất quán, dễ đọc.
nextpage:
  url: /effective-dart/documentation
  title: Documentation
prevpage:
  url: /effective-dart
  title: Tổng quan
---
<?code-excerpt plaster="none"?>
<?code-excerpt path-base="misc/lib/effective_dart"?>

Một phần quan trọng đến bất ngờ của code tốt là style tốt. Cách đặt tên,
sắp xếp và format nhất quán giúp code *giống nhau* có *vẻ ngoài* giống nhau.
Nó tận dụng phần cứng so khớp pattern mạnh mẽ mà phần lớn chúng ta có trong
hệ thị giác. Nếu dùng style nhất quán trên toàn hệ sinh thái Dart,
tất cả chúng ta sẽ dễ học hỏi và đóng góp vào code của nhau hơn.

<a id="identifiers"></a>
## Identifier

Identifier có ba kiểu trong Dart.

* Tên `UpperCamelCase` viết hoa chữ cái đầu của mỗi từ, bao gồm
  từ đầu tiên.

* Tên `lowerCamelCase` viết hoa chữ cái đầu của mỗi từ, *ngoại trừ*
  từ đầu luôn viết thường, ngay cả khi đó là acronym.

* Tên `lowercase_with_underscores` chỉ dùng chữ thường,
  kể cả acronym, và phân tách các từ bằng `_`.

<a id="do-name-types-using-uppercamelcase"></a>
### DO đặt tên type bằng `UpperCamelCase`

{% render 'linter-rule-mention.md', rules:'camel_case_types' %}

Class, enum type, typedef và type parameter nên viết hoa chữ cái đầu
của mỗi từ (bao gồm từ đầu tiên) và không dùng ký tự phân cách.

<?code-excerpt "style_good.dart (type-names)"?>
```dart tag=good
class SliderMenu {
   ...
}

class HttpRequest {
   ...
}

typedef Predicate<T> = bool Function(T value);
```

Điều này thậm chí bao gồm class dự định dùng trong metadata annotation.

<?code-excerpt "style_good.dart (annotation-type-names)"?>
```dart tag=good
class Foo {
  const Foo([Object? arg]);
}

@Foo(anArg)
class A {
   ...
}

@Foo()
class B {
   ...
}
```

Nếu constructor của annotation class không nhận parameter, bạn có thể muốn
tạo constant `lowerCamelCase` riêng cho nó.

<?code-excerpt "style_good.dart (annotation-const)"?>
```dart tag=good
const foo = Foo();

@foo
class C {
   ...
}
```

<a id="do-name-extensions-using-uppercamelcase"></a>
### DO đặt tên extension bằng `UpperCamelCase`

{% render 'linter-rule-mention.md', rules:'camel_case_extensions' %}

Giống type, [extension][] nên viết hoa chữ cái đầu của mỗi từ
(bao gồm từ đầu tiên)
và không dùng ký tự phân cách.

<?code-excerpt "style_good.dart (extension-names)"?>
```dart tag=good
extension MyFancyList<T> on List<T> {
   ...
}

extension SmartIterable<T> on Iterable<T> {
   ...
}
```

[extensions]: /language/extension-methods

<a id="do-name-libraries-and-source-files-using-lowercase_with_underscores"></a>
### DO đặt tên package, directory và source file bằng `lowercase_with_underscores` {:#do-name-packages-and-file-system-entities-using-lowercase-with-underscores}

{% render 'linter-rule-mention.md', rules:'file_names, package_names' %}

Một số file system không phân biệt chữ hoa chữ thường, nên nhiều project yêu cầu filename
hoàn toàn viết thường. Dùng ký tự phân cách giúp tên vẫn dễ đọc
ở dạng đó. Dùng dấu gạch dưới làm ký tự phân cách bảo đảm tên vẫn
là identifier Dart hợp lệ, điều có thể hữu ích nếu sau này ngôn ngữ hỗ trợ
symbolic import.

```plaintext tag=good
my_package
└─ lib
   └─ file_system.dart
   └─ slider_menu.dart
```

```plaintext tag=bad
mypackage
└─ lib
   └─ file-system.dart
   └─ SliderMenu.dart
```


<a id="do-name-import-prefixes-using-lowercase_with_underscores"></a>
### DO đặt tên import prefix bằng `lowercase_with_underscores`

{% render 'linter-rule-mention.md', rules:'library_prefixes' %}

<?code-excerpt "style_lib_good.dart (import-as)" replace="/(package):examples\/effective_dart\/foo.dart[^']*/$1:angular_components\/angular_components.dart/g; /(package):examples\/effective_dart\/bar.dart[^']*/$1:js\/js.dart/g"?>
```dart tag=good
import 'dart:math' as math;

import 'package:angular_components/angular_components.dart' as angular_components;
import 'package:js/js.dart' as js;
```

<?code-excerpt "style_lib_good.dart (import-as)" replace="/(package):examples\/effective_dart\/foo.dart[^']*/$1:angular_components\/angular_components.dart/g; /as angular_components/as angularComponents/g; /(package):examples\/effective_dart\/bar.dart[^']*/$1:js\/js.dart/g; / math/ Math/g;/as js/as JS/g"?>
```dart tag=bad
import 'dart:math' as Math;

import 'package:angular_components/angular_components.dart' as angularComponents;
import 'package:js/js.dart' as JS;
```


<a id="do-name-other-identifiers-using-lowercamelcase"></a>
### DO đặt tên identifier khác bằng `lowerCamelCase`

{% render 'linter-rule-mention.md', rules:'non_constant_identifier_names' %}

Class member, định nghĩa top-level, variable, parameter và named
parameter nên viết hoa chữ cái đầu của mỗi từ *ngoại trừ* từ đầu tiên,
và không dùng ký tự phân cách.

<?code-excerpt "style_good.dart (misc-names)"?>
```dart tag=good
var count = 3;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
```


<a id="prefer-using-lowercamelcase-for-constant-names"></a>
### PREFER dùng `lowerCamelCase` cho tên constant

{% render 'linter-rule-mention.md', rules:'constant_identifier_names' %}

Trong code mới, dùng `lowerCamelCase` cho biến constant, bao gồm value enum.

<?code-excerpt "style_good.dart (const-names)"?>
```dart tag=good
const pi = 3.14;
const defaultTimeout = 1000;
final urlScheme = RegExp('^([a-z]+):');

class Dice {
  static final numberGenerator = Random();
}
```

<?code-excerpt "style_bad.dart (const-names)"?>
```dart tag=bad
const PI = 3.14;
const DefaultTimeout = 1000;
final URL_SCHEME = RegExp('^([a-z]+):');

class Dice {
  static final NUMBER_GENERATOR = Random();
}
```

Bạn có thể dùng `SCREAMING_CAPS` để nhất quán với code hiện có,
như trong các trường hợp sau:

* Khi thêm code vào file hoặc library đã dùng `SCREAMING_CAPS`.
* Khi tạo code Dart song song với code Java—ví dụ,
  trong enum type được tạo từ [protobuf.][protobufs.]

:::note
Ban đầu chúng tôi dùng style `SCREAMING_CAPS` của Java cho constant. Chúng tôi
thay đổi vì một vài lý do:

* `SCREAMING_CAPS` trông không đẹp trong nhiều trường hợp,
  đặc biệt là value enum cho những thứ như màu CSS.
* Constant thường được đổi thành biến final non-const,
  điều sẽ buộc phải đổi tên.
* Property `values` được định nghĩa trên enum type là const và viết thường.

:::

[protobufs.]: {{site.pub-pkg}}/protobuf


<a id="do-capitalize-acronyms-and-abbreviations-longer-than-two-letters-like-words"></a>
### DO viết hoa acronym và abbreviation dài hơn hai chữ cái như từ thông thường

Acronym viết hoa có thể khó đọc,
và nhiều acronym liền nhau có thể dẫn đến tên mơ hồ.
Ví dụ, với identifier `HTTPSFTP`,
người đọc không thể biết nó chỉ `HTTPS` `FTP` hay `HTTP` `SFTP`.
Để tránh điều này,
hãy viết hoa phần lớn acronym và abbreviation như từ thông thường.
Identifier này sẽ là `HttpsFtp` nếu chỉ trường hợp trước
hoặc `HttpSftp` cho trường hợp sau.

Abbreviation và acronym hai chữ cái là ngoại lệ.
Nếu cả hai chữ cái được viết hoa trong tiếng Anh,
cả hai nên tiếp tục viết hoa khi dùng trong identifier.
Nếu không, hãy viết hoa như một từ.

```dart tag=good
// Dài hơn hai chữ cái, nên luôn như một từ:
Http // "hypertext transfer protocol"
Nasa // "national aeronautics and space administration"
Uri // "uniform resource identifier"
Esq // "esquire"
Ave // "avenue"

// Hai chữ cái, viết hoa trong tiếng Anh, nên viết hoa trong identifier:
ID // "identifier"
TV // "television"
UI // "user interface"

// Hai chữ cái, không viết hoa trong tiếng Anh, nên như một từ trong identifier:
Mr // "mister"
St // "street"
Rd // "road"
```

```dart tag=bad
HTTP // "hypertext transfer protocol"
NASA // "national aeronautics and space administration"
URI // "uniform resource identifier"
esq // "esquire"
ave // "avenue"

Id // "identifier"
Tv // "television"
Ui // "user interface"

MR // "mister"
ST // "street"
RD // "road"
```

Khi bất kỳ dạng abbreviation nào nằm ở đầu
identifier `lowerCamelCase`, abbreviation phải hoàn toàn viết thường:

```dart
var httpConnection = connect();
var tvSet = Television();
var mrRogers = 'hello, neighbor';
```

<a id="prefer-using-_-__-etc-for-unused-callback-parameters" aria-hidden="true"></a>

<a id="prefer-using-wildcards-for-unused-callback-parameters"></a>
### PREFER dùng wildcard cho callback parameter không dùng

Đôi khi type signature của hàm callback yêu cầu parameter,
nhưng implementation callback không _dùng_ parameter đó.
Trong trường hợp này, cách đúng idiom là đặt tên parameter không dùng là `_`,
khai báo một [biến wildcard][wildcards] non-binding.

<?code-excerpt "style_good.dart (unused-callback-param)"?>
```dart tag=good
futureOfVoid.then((_) {
  print('Operation complete.');
});
```

Vì biến wildcard non-binding,
bạn có thể đặt tên nhiều parameter không dùng là `_`.

<?code-excerpt "style_good.dart (unused-callback-params-multiple)"?>
```dart tag=good
.onError((_, _) {
  print('Operation failed.');
});
```

Guideline này chỉ dành cho hàm vừa *anonymous vừa local*.
Những hàm này thường được dùng ngay trong context làm rõ
parameter không dùng biểu diễn gì.
Ngược lại, hàm top-level và khai báo method không có context đó,
nên parameter của chúng phải được đặt tên để mục đích của từng parameter rõ ràng,
ngay cả khi không dùng.

:::version-note
Khai báo [biến wildcard][wildcards] non-binding yêu cầu
[language version][] tối thiểu 3.7.

Trong language version cũ hơn, dùng thêm dấu gạch dưới để
tránh xung đột tên, như `__` và `___`.
Để bắt buộc không dùng chúng và đơn giản hóa migration sang wildcard sau này,
hãy bật lint [`no_wildcard_variable_uses`][].

Để hỗ trợ migration từ quy ước này sang biến wildcard,
hãy bật lint [`unnecessary_underscores`][].
:::

[wildcards]: /language/variables#wildcard-variables
[language version]: /language/versioning
[`no_wildcard_variable_uses`]: /tools/linter-rules/no_wildcard_variable_uses
[`unnecessary_underscores`]: /tools/linter-rules/unnecessary_underscores
