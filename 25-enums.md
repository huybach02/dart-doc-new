---
title: Enumerated types
description: Tìm hiểu về kiểu enum trong Dart.
shortTitle: Enum
prevpage:
  url: /language/mixins
  title: Mixin
nextpage:
  url: /language/dot-shorthands
  title: Dot shorthand
---

Kiểu liệt kê, thường được gọi là _enumeration_ hoặc _enum_,
là một loại lớp đặc biệt dùng để biểu diễn
một số lượng cố định các giá trị hằng.

:::note
Mọi enum tự động mở rộng lớp [`Enum`][].
Chúng cũng là sealed,
nghĩa là không thể được tạo subclass, implement, mix in
hoặc được khởi tạo tường minh theo cách khác.

Abstract class và mixin có thể implement hoặc mở rộng `Enum` một cách tường minh,
nhưng trừ khi sau đó chúng được implement bởi hoặc được mix vào một khai báo enum,
không object nào thực sự có thể implement kiểu của lớp hoặc mixin đó.
:::

<a id="declaring-simple-enums"></a>
## Khai báo enum đơn giản

Để khai báo một kiểu liệt kê đơn giản,
dùng keyword `enum` và
liệt kê các giá trị bạn muốn đưa vào enum:

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (enum)"?>
```dart
enum Color { red, green, blue }
```

:::tip
Bạn cũng có thể dùng [dấu phẩy cuối][trailing commas] khi khai báo kiểu liệt kê
để giúp ngăn lỗi sao chép và dán.
:::

<a id="declaring-enhanced-enums"></a>
## Khai báo enhanced enum

:::version-note
Enhanced enum yêu cầu [language version][] tối thiểu là 2.17.
:::

Dart cũng cho phép khai báo enum khai báo các lớp
có field, method và const constructor,
được giới hạn ở một số lượng cố định các instance hằng đã biết.

Để khai báo enhanced enum,
hãy dùng cú pháp tương tự [lớp][classes] thông thường,
nhưng kèm một số yêu cầu bổ sung:

* Instance variable phải là `final`,
  bao gồm cả những biến được [mixin][mixins] thêm vào.
* Tất cả [generative constructor][generative constructors] phải là constant.
* [Factory constructor][Factory constructors] chỉ có thể trả về
  một trong các instance enum cố định đã biết.
* Không thể mở rộng bất kỳ lớp nào khác vì [`Enum`] được tự động mở rộng.
* Không được override `index`, `hashCode` hoặc toán tử bằng `==`.
* Không thể khai báo member tên `values` trong enum,
  vì nó sẽ xung đột với getter static `values` được tự động tạo.
* Tất cả instance của enum phải được khai báo
  ở đầu khai báo, và phải khai báo ít nhất một instance.

Instance method trong enhanced enum có thể dùng `this` để
tham chiếu đến giá trị enum hiện tại.

Sau đây là ví dụ khai báo một enhanced enum
có nhiều instance, instance variable,
getter và một interface được implement:

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (enhanced)"?>
```dart
enum Vehicle implements Comparable<Vehicle> {
  car(tires: 4, passengers: 5, carbonPerKilometer: 400),
  bus(tires: 6, passengers: 50, carbonPerKilometer: 800),
  bicycle(tires: 2, passengers: 1, carbonPerKilometer: 0);

  const Vehicle({
    required this.tires,
    required this.passengers,
    required this.carbonPerKilometer,
  });

  final int tires;
  final int passengers;
  final int carbonPerKilometer;

  int get carbonFootprint => (carbonPerKilometer / passengers).round();

  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
```

:::tip
Trong Dart 3.13 trở lên,
bạn có thể khai báo enhanced enum ngắn gọn hơn nữa
bằng [primary constructor][primary constructors]:

<?code-excerpt "language/lib/primary_constructors/enum.dart (enhanced-primary)"?>
```dart
enum Vehicle(
  final int tires,
  final int passengers,
  final int carbonPerKilometer,
) implements Comparable<Vehicle> {
  car(4, 5, 400),
  bus(6, 50, 800),
  bicycle(2, 1, 0);

  int get carbonFootprint => (carbonPerKilometer / passengers).round();

  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
```
:::

[language version]: /language/versioning
[primary constructors]: /language/primary-constructors

<a id="using-enums"></a>
## Sử dụng enum

Truy cập các giá trị liệt kê giống như
bất kỳ [biến static][static variable] nào khác:

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (access)"?>
```dart
final favoriteColor = Color.blue;
if (favoriteColor == Color.blue) {
  print('Your favorite color is blue!');
}
```

Mỗi giá trị trong enum có một getter `index`,
trả về vị trí bắt đầu từ 0 của giá trị trong khai báo enum.
Ví dụ, giá trị đầu tiên có index 0,
và giá trị thứ hai có index 1.

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (index)"?>
```dart
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

Để lấy list gồm tất cả giá trị liệt kê,
dùng hằng `values` của enum.

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (values)"?>
```dart
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);
```

Bạn có thể dùng enum trong [statement switch][switch statements], và
bạn sẽ nhận cảnh báo nếu không xử lý tất cả giá trị của enum:

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (switch)"?>
```dart
var aColor = Color.blue;

switch (aColor) {
  case Color.red:
    print('Red as roses!');
  case Color.green:
    print('Green as grass!');
  default: // Nếu không có dòng này, bạn sẽ thấy CẢNH BÁO.
    print(aColor); // 'Color.blue'
}
```

Nếu cần truy cập tên của một giá trị liệt kê,
chẳng hạn `'blue'` từ `Color.blue`,
hãy dùng property `.name`:

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (name)"?>
```dart
print(Color.blue.name); // 'blue'
```

Bạn có thể truy cập member của một giá trị enum
giống như trên object thông thường:

<?code-excerpt "misc/lib/language_tour/classes/enum.dart (method-call)"?>
```dart
print(Vehicle.car.carbonFootprint);
```

[`Enum`]: {{site.dart-api}}/dart-core/Enum-class.html
[trailing commas]: /language/collections#lists
[classes]: /language/classes
[mixins]: /language/mixins
[generative constructors]: /language/constructors#constant-constructors
[Factory constructors]: /language/constructors#factory-constructors
[static variable]: /language/classes#class-variables-and-methods
[switch statements]: /language/branches#switch
