---
title: Generics
description: Tìm hiểu về các kiểu generic trong Dart.
prevpage:
  url: /language/collections
  title: Collection
nextpage:
  url: /language/typedefs
  title: Typedef
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

Nếu xem tài liệu API dành cho kiểu mảng cơ bản
[`List`][], bạn sẽ thấy kiểu này thực ra là `List<E>`.
Ký pháp \<...\> đánh dấu List là một kiểu *generic* (hay kiểu *được tham số hóa*)—
một kiểu có các tham số kiểu hình thức. [Theo quy ước][], hầu hết biến kiểu
có tên gồm một chữ cái, chẳng hạn như E, T, S, K và V.

<a id="why-use-generics"></a>

## Tại sao nên sử dụng generic?

Generic thường cần thiết để bảo đảm type safety, nhưng chúng còn mang lại nhiều lợi ích
khác ngoài việc cho phép code của bạn chạy:

* Việc chỉ định đúng các kiểu generic tạo ra code được sinh tốt hơn.
* Bạn có thể sử dụng generic để giảm sự trùng lặp code.

Nếu bạn muốn một list chỉ chứa chuỗi, bạn có thể
khai báo nó là `List<String>` (đọc là "list của string"). Theo cách đó,
bạn, những lập trình viên cùng làm việc và các công cụ có thể phát hiện rằng việc gán một giá trị không phải chuỗi vào
list có thể là một lỗi. Sau đây là ví dụ:

```dart tag=fails-sa
var names = <String>[];
names.addAll(['Seth', 'Kathy', 'Lars']);
names.add(42); // Lỗi
```

Một lý do khác để sử dụng generic là giảm sự trùng lặp code.
Generic cho phép bạn dùng chung một interface và phần triển khai giữa
nhiều kiểu, đồng thời vẫn tận dụng được khả năng phân tích
tĩnh. Ví dụ, giả sử bạn tạo một interface để
cache một đối tượng:

<?code-excerpt "misc/lib/language_tour/generics/cache.dart (object-cache)"?>
```dart
abstract class ObjectCache {
  Object getByKey(String key);
  void setByKey(String key, Object value);
}
```

Bạn nhận ra rằng mình muốn có một phiên bản dành riêng cho chuỗi của interface này,
vì vậy bạn tạo một interface khác:

<?code-excerpt "misc/lib/language_tour/generics/cache.dart (string-cache)"?>
```dart
abstract class StringCache {
  String getByKey(String key);
  void setByKey(String key, String value);
}
```

Sau đó, bạn quyết định muốn có một phiên bản dành riêng cho số của
interface này... Bạn đã hiểu ý tưởng rồi đấy.

Kiểu generic giúp bạn không phải tạo tất cả những interface này.
Thay vào đó, bạn có thể tạo một interface duy nhất nhận một tham số kiểu:

<?code-excerpt "misc/lib/language_tour/generics/cache.dart (cache)"?>
```dart
abstract class Cache<T> {
  T getByKey(String key);
  void setByKey(String key, T value);
}
```

Trong đoạn code này, T là kiểu đại diện. Đây là một placeholder mà bạn có thể
hiểu là kiểu sẽ được một lập trình viên định nghĩa sau.


<a id="using-collection-literals"></a>

## Sử dụng collection literal

Literal của list, set và map có thể được tham số hóa. Literal được tham số hóa
giống hệt các literal bạn đã thấy, ngoại trừ việc bạn thêm
<code>&lt;<em>type</em>></code> (đối với list và set) hoặc
<code>&lt;<em>keyType</em>, <em>valueType</em>></code> (đối với map)
trước dấu ngoặc mở. Sau đây là ví dụ về cách sử dụng typed literal:

<?code-excerpt "misc/lib/language_tour/generics/misc.dart (collection-literals)"?>
```dart
var names = <String>['Seth', 'Kathy', 'Lars'];
var uniqueNames = <String>{'Seth', 'Kathy', 'Lars'};
var pages = <String, String>{
  'index.html': 'Homepage',
  'robots.txt': 'Hints for web robots',
  'humans.txt': 'We are people, not machines',
};
```


<a id="using-parameterized-types-with-constructors"></a>

## Sử dụng kiểu được tham số hóa với constructor

Để chỉ định một hoặc nhiều kiểu khi sử dụng constructor, hãy đặt các kiểu trong
dấu ngoặc góc (`<...>`) ngay sau tên lớp. Ví dụ:

<?code-excerpt "misc/test/language_tour/generics_test.dart (constructor-1)"?>
```dart
var nameSet = Set<String>.of(names);
```

Đoạn code sau tạo một `SplayTreeMap` có
key là số nguyên và value thuộc kiểu `View`:

<?code-excerpt "misc/test/language_tour/generics_test.dart (constructor-2)"?>
```dart
var views = SplayTreeMap<int, View>();
```


<a id="generic-collections-and-the-types-they-contain"></a>

## Collection generic và các kiểu mà chúng chứa

Các kiểu generic trong Dart được *reified*, nghĩa là chúng mang theo thông tin
về kiểu tại runtime. Ví dụ, bạn có thể kiểm tra kiểu của một
collection:

<?code-excerpt "misc/test/language_tour/generics_test.dart (generic-collections)"?>
```dart
var names = <String>[];
names.addAll(['Seth', 'Kathy', 'Lars']);
print(names is List<String>); // true
```

:::note
Ngược lại, generic trong Java sử dụng *erasure*, nghĩa là các
tham số kiểu generic bị loại bỏ tại runtime. Trong Java, bạn có thể kiểm tra liệu
một đối tượng có phải List hay không, nhưng không thể kiểm tra liệu nó có phải `List<String>` hay không.
:::


<a id="restricting-the-parameterized-type"></a>

## Giới hạn kiểu được tham số hóa

Khi triển khai một kiểu generic,
bạn có thể muốn giới hạn các kiểu có thể được cung cấp làm đối số,
để đối số phải là subtype của một kiểu cụ thể.
Giới hạn này được gọi là bound.
Bạn có thể thực hiện điều này bằng `extends`.

Một trường hợp sử dụng phổ biến là bảo đảm một kiểu là non-nullable
bằng cách biến nó thành subtype của `Object`
(thay vì kiểu mặc định [`Object?`][top-and-bottom]).

<?code-excerpt "misc/lib/language_tour/generics/misc.dart (non-nullable)"?>
```dart
class Foo<T extends Object> {
  // Bất kỳ kiểu nào được cung cấp cho T của Foo đều phải là non-nullable.
}
```

Bạn có thể sử dụng `extends` với các kiểu khác ngoài `Object`.
Sau đây là ví dụ về việc mở rộng `SomeBaseClass`,
để có thể gọi các member của `SomeBaseClass` trên những đối tượng thuộc kiểu `T`:

<?code-excerpt "misc/lib/language_tour/generics/base_class.dart (generic)" replace="/extends SomeBaseClass(?=. \{)/[!$&!]/g"?>
```dart
class Foo<T [!extends SomeBaseClass!]> {
  // Phần triển khai được đặt tại đây...
  String toString() => "Instance of 'Foo<$T>'";
}

class Extender extends SomeBaseClass {
  ...
}
```

Bạn hoàn toàn có thể sử dụng `SomeBaseClass` hoặc bất kỳ subtype nào của nó làm đối số generic:

<?code-excerpt "misc/test/language_tour/generics_test.dart (SomeBaseClass-ok)" replace="/Foo.\w+./[!$&!]/g"?>
```dart
var someBaseClassFoo = [!Foo<SomeBaseClass>!]();
var extenderFoo = [!Foo<Extender>!]();
```

Việc không chỉ định đối số generic cũng hợp lệ:

<?code-excerpt "misc/test/language_tour/generics_test.dart (no-generic-arg-ok)" replace="/expect\((.*?).toString\(\), .(.*?).\);/print($1); \/\/ $2/g"?>
```dart
var foo = Foo();
print(foo); // Instance of 'Foo<SomeBaseClass>'
```

Việc chỉ định bất kỳ kiểu nào không phải `SomeBaseClass` sẽ gây ra lỗi:

```dart tag=fails-sa
var foo = [!Foo<Object>!]();
```

<a id="f-bounds"></a>

### Giới hạn tham số kiểu tự tham chiếu (F-bound)

Khi sử dụng bound để giới hạn các kiểu tham số, bạn có thể tham chiếu bound
trở lại chính tham số kiểu đó. Điều này tạo ra một ràng buộc tự tham chiếu,
hay F-bound. Ví dụ:

<?code-excerpt "misc/test/language_tour/generics_test.dart (f-bound)"?>
```dart
abstract interface class Comparable<T> {
  int compareTo(T o);
}

int compareAndOffset<T extends Comparable<T>>(T t1, T t2) =>
    t1.compareTo(t2) + 1;

class A implements Comparable<A> {
  @override
  int compareTo(A other) => /*...phần triển khai...*/ 0;
}

int useIt = compareAndOffset(A(), A());
```

F-bound `T extends Comparable<T>` có nghĩa là `T` phải có khả năng so sánh với chính nó.
Do đó, `A` chỉ có thể được so sánh với các instance khác cùng kiểu.

<a id="using-generic-methods"></a>

## Sử dụng phương thức generic

Phương thức và hàm cũng cho phép các đối số kiểu:

<!-- {{site.dartpad}}/a02c53b001977efa4d803109900f21bb -->
<!-- https://gist.github.com/a02c53b001977efa4d803109900f21bb -->
<?code-excerpt "misc/test/language_tour/generics_test.dart (method)" replace="/<T.(?=\()|T/[!$&!]/g"?>
```dart
[!T!] first[!<T>!](List<[!T!]> ts) {
  // Thực hiện một số công việc ban đầu hoặc kiểm tra lỗi, sau đó...
  [!T!] tmp = ts[0];
  // Thực hiện thêm một số thao tác kiểm tra hoặc xử lý...
  return tmp;
}
```

Ở đây, tham số kiểu generic trên `first` (`<T>`)
cho phép bạn sử dụng đối số kiểu `T` ở một số vị trí:

* Trong kiểu trả về của hàm (`T`).
* Trong kiểu của một đối số (`List<T>`).
* Trong kiểu của một biến cục bộ (`T tmp`).

[`List`]: {{site.dart-api}}/dart-core/List-class.html
[Theo quy ước]: /effective-dart/design#do-follow-existing-mnemonic-conventions-when-naming-type-parameters
[top-and-bottom]: /null-safety/understanding-null-safety#top-and-bottom
