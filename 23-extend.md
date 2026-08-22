---
title: Extend a class
description: Tìm hiểu cách tạo subclass từ superclass.
prevpage:
  url: /language/methods
  title: Phương thức
nextpage:
  url: /language/mixins
  title: Mixin
---

Dùng `extends` để tạo subclass và dùng `super` để tham chiếu đến
superclass:

<?code-excerpt "misc/lib/language_tour/classes/extends.dart (smart-tv)" replace="/extends|super/[!$&!]/g"?>
```dart
class Television {
  void turnOn() {
    _illuminateDisplay();
    _activateIrSensor();
  }
  // ···
}

class SmartTelevision [!extends!] Television {
  void turnOn() {
    [!super!].turnOn();
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
  // ···
}
```

Để xem một cách sử dụng khác của `extends`, hãy đọc phần thảo luận về
[kiểu được parameter hóa][parameterized types] trên trang Generic.

<a id="overriding-members"></a>
## Override member

Subclass có thể override instance method (bao gồm [toán tử][operators]),
getter và setter.
Bạn có thể dùng annotation `@override` để cho biết rằng bạn
cố ý override một member:

<?code-excerpt "misc/lib/language_tour/metadata/television.dart (override)" replace="/@override/[!$&!]/g"?>
```dart
class Television {
  // ···
  set contrast(int value) {
    // ···
  }
}

class SmartTelevision extends Television {
  [!@override!]
  set contrast(num value) {
    // ···
  }
  // ···
}
```

Khai báo method override phải khớp với
method (hoặc các method) mà nó override theo một số phương diện:

* Return type phải cùng kiểu với (hoặc là subtype của)
  return type của method bị override.
* Kiểu parameter phải cùng kiểu với (hoặc là supertype của)
  kiểu parameter của method bị override.
  Trong ví dụ trước, setter `contrast` của `SmartTelevision`
  thay đổi kiểu parameter từ `int` thành một supertype là `num`.
* Nếu method bị override chấp nhận _n_ positional parameter,
  thì method override cũng phải chấp nhận _n_ positional parameter.
* Một [generic method][] không thể override method non-generic,
  và method non-generic không thể override generic method.

Đôi khi bạn có thể muốn thu hẹp kiểu của
parameter của method hoặc instance variable.
Điều này vi phạm các quy tắc thông thường và
tương tự downcast ở chỗ nó có thể gây lỗi kiểu tại runtime.
Dù vậy, vẫn có thể thu hẹp kiểu
nếu code có thể bảo đảm rằng lỗi kiểu sẽ không xảy ra.
Trong trường hợp này, bạn có thể dùng
[keyword `covariant`](/language/type-system#covariant-keyword)
trong khai báo parameter.
Để biết chi tiết, xem
[đặc tả ngôn ngữ Dart][Dart language specification].

:::warning
Nếu override `==`, bạn cũng nên override getter `hashCode` của Object.
Để xem ví dụ override `==` và `hashCode`, hãy đọc
[Implement map key](/libraries/dart-core#implementing-map-keys).
:::

<a id="nosuchmethod"></a>
## noSuchMethod()

Để phát hiện hoặc phản ứng bất cứ khi nào code cố sử dụng một method hoặc
instance variable không tồn tại, bạn có thể override `noSuchMethod()`:

<?code-excerpt "misc/lib/language_tour/classes/no_such_method.dart (no-such-method-impl)" replace="/noSuchMethod(?!,)/[!$&!]/g"?>
```dart
class A {
  // Trừ khi bạn override noSuchMethod, việc sử dụng một
  // member không tồn tại sẽ dẫn đến NoSuchMethodError.
  @override
  void [!noSuchMethod!](Invocation invocation) {
    print(
      'You tried to use a non-existent member: '
      '${invocation.memberName}',
    );
  }
}
```

Bạn **không thể gọi** một method chưa được implement trừ khi
**một** trong các điều sau là đúng:

* Receiver có static type là `dynamic`.

* Receiver có một static type
định nghĩa method chưa được implement (abstract được chấp nhận),
và dynamic type của receiver có một implementation của `noSuchMethod()`
khác với implementation trong lớp `Object`.

Để biết thêm thông tin, xem
[đặc tả không chính thức về chuyển tiếp noSuchMethod.]({{site.repo.dart.lang}}/blob/main/archive/feature-specifications/nosuchmethod-forwarding.md)

[parameterized types]: /language/generics#restricting-the-parameterized-type
[operators]: /language/methods#operators
[generic method]: /language/generics#using-generic-methods
[Dart language specification]: /resources/language/spec
