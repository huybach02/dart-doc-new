---
title: Typedef
description: Tìm hiểu về bí danh kiểu trong Dart.
showToc: false
prevpage:
  url: /language/generics
  title: Generic
nextpage:
  url: /language/type-system
  title: Hệ thống kiểu
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

Bí danh kiểu—thường được gọi là _typedef_ vì
nó được khai báo bằng keyword `typedef`—là
một cách ngắn gọn để tham chiếu đến một kiểu.
Sau đây là ví dụ về cách khai báo và sử dụng một bí danh kiểu có tên `IntList`:

<?code-excerpt "misc/lib/language_tour/typedefs/misc.dart (int-list)"?>
```dart
typedef IntList = List<int>;
IntList il = [1, 2, 3];
```

Bí danh kiểu có thể có các tham số kiểu:

<?code-excerpt "misc/lib/language_tour/typedefs/misc.dart (list-mapper)"?>
```dart
typedef ListMapper<X> = Map<X, List<X>>;
Map<String, List<String>> m1 = {}; // Dài dòng.
ListMapper<String> m2 = {}; // Tương tự nhưng ngắn gọn và rõ ràng hơn.
```

:::version-note
Trước phiên bản 2.13, typedef bị giới hạn ở các kiểu hàm.
Việc sử dụng typedef mới yêu cầu [phiên bản ngôn ngữ][] tối thiểu là 2.13.
:::

Trong hầu hết trường hợp, chúng tôi khuyến nghị sử dụng [kiểu hàm inline][] thay cho typedef
đối với hàm.
Tuy nhiên, typedef cho hàm vẫn có thể hữu ích:

<?code-excerpt "misc/lib/language_tour/typedefs/misc.dart (compare)"?>
```dart
typedef Compare<T> = int Function(T a, T b);

int sort(int a, int b) => a - b;

void main() {
  assert(sort is Compare<int>); // True!
}
```

[phiên bản ngôn ngữ]: /language/versioning
[kiểu hàm inline]: /effective-dart/design#prefer-inline-function-types-over-typedefs
