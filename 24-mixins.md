---
title: Mixin
description: Tìm hiểu cách bổ sung tính năng cho một lớp trong Dart.
prevpage:
  url: /language/extend
  title: Mở rộng một lớp
nextpage:
  url: /language/enums
  title: Enum
---

<?code-excerpt replace="/ *\/\/\s+ignore_for_file:[^\n]+\n//g; /(^|\n) *\/\/\s+ignore:[^\n]+\n/$1/g; /(\n[^\n]+) *\/\/\s+ignore:[^\n]+\n/$1\n/g; / *\/\/\s+ignore:[^\n]+//g; /([A-Z]\w*)\d\b/$1/g"?>

Mixin là một cách định nghĩa code có thể được tái sử dụng trong nhiều hệ thống phân cấp lớp.
Chúng được thiết kế để cung cấp hàng loạt implementation cho member.

Để dùng mixin, hãy dùng keyword `with`, theo sau là một hoặc nhiều tên
mixin. Ví dụ sau minh họa hai lớp sử dụng mixin (hay là subclass của
mixin):

<?code-excerpt "misc/lib/language_tour/classes/orchestra.dart (musician-and-maestro)" replace="/(with.*) \{/[!$1!] {/g"?>
```dart
class Musician extends Performer [!with Musical!] {
  // ···
}

class Maestro extends Person [!with Musical, Aggressive, Demented!] {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

Để định nghĩa một mixin, hãy dùng khai báo `mixin`.
Trong trường hợp hiếm hoi cần định nghĩa cả mixin _lẫn_ lớp, bạn có thể dùng
[khai báo `mixin class`](#class-mixin-or-mixin-class).

Mixin và mixin class không thể có mệnh đề `extends`,
và không được khai báo bất kỳ generative constructor nào.

Ví dụ:

<?code-excerpt "misc/lib/language_tour/classes/orchestra.dart (musical)"?>
```dart
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

<a id="specify-members-a-mixin-can-call-on-itself"></a>
## Chỉ định các member mà mixin có thể tự gọi

Đôi khi một mixin phụ thuộc vào khả năng gọi method hoặc truy cập field,
nhưng không thể tự định nghĩa các member đó (vì mixin không thể dùng constructor
parameter để khởi tạo field riêng của chúng).

Các mục sau đề cập những chiến lược khác nhau nhằm bảo đảm mọi subclass
của mixin đều định nghĩa các member mà hành vi của mixin phụ thuộc vào.

<a id="define-abstract-members-in-the-mixin"></a>
### Định nghĩa abstract member trong mixin

Khai báo abstract method trong mixin buộc mọi kiểu sử dụng
mixin phải định nghĩa abstract method mà hành vi của mixin phụ thuộc vào.

```dart
mixin Musician {
  void playInstrument(String instrumentName); // Abstract method.

  void playPiano() {
    playInstrument('Piano');
  }
  void playFlute() {
    playInstrument('Flute');
  }
}

class Virtuoso with Musician {

  @override
  void playInstrument(String instrumentName) { // Subclass phải định nghĩa.
    print('Plays the $instrumentName beautifully');
  }
}
```

<a id="access-state-in-the-mixins-subclass"></a>
#### Truy cập trạng thái trong subclass của mixin

Khai báo abstract member cũng cho phép bạn truy cập trạng thái trên subclass
của mixin bằng cách gọi getter được định nghĩa là abstract trên mixin:

```dart
/// Có thể áp dụng cho mọi kiểu có property [name] và cung cấp
/// implementation của [hashCode] và toán tử `==` dựa trên property đó.
mixin NameIdentity {
  String get name;

  @override
  int get hashCode => name.hashCode;

  @override
  bool operator ==(other) => other is NameIdentity && name == other.name;
}

class Person with NameIdentity {
  final String name;

  Person(this.name);
}
```

<a id="implement-an-interface"></a>
### Implement một interface

Tương tự việc khai báo mixin là abstract, việc đặt mệnh đề `implements` trên
mixin nhưng không thực sự implement interface cũng sẽ bảo đảm mọi dependency
member đều được định nghĩa cho mixin.

```dart
abstract interface class Tuner {
  void tuneInstrument();
}

mixin Guitarist implements Tuner {
  void playSong() {
    tuneInstrument();

    print('Strums guitar majestically.');
  }
}

class PunkRocker with Guitarist {

  @override
  void tuneInstrument() {
    print("Don't bother, being out of tune is punk rock.");
  }
}
```

<a id="use-the-on-clause-to-declare-a-superclass"></a>
### Dùng mệnh đề `on` để khai báo superclass

Mệnh đề `on` tồn tại để định nghĩa kiểu mà các lời gọi `super` được phân giải dựa vào.
Vì vậy, bạn chỉ nên dùng nó nếu cần có lời gọi `super` bên trong mixin.

Mệnh đề `on` buộc mọi lớp dùng mixin cũng phải là subclass
của kiểu trong mệnh đề `on`.
Nếu mixin phụ thuộc vào các member trong superclass,
điều này bảo đảm những member đó khả dụng tại nơi mixin được dùng:

```dart
class Musician {
  musicianMethod() {
    print('Playing music!');
  }
}

mixin MusicalPerformer [!on Musician!] {
  performerMethod() {
    print('Performing music!');
    super.musicianMethod();
  }
}

class SingerDancer extends Musician with MusicalPerformer { }

main() {
  SingerDancer().performerMethod();
}
```

Trong ví dụ này, chỉ các lớp mở rộng hoặc implement lớp `Musician`
mới có thể dùng mixin `MusicalPerformer`. Vì `SingerDancer` mở rộng `Musician`,
`SingerDancer` có thể mix in `MusicalPerformer`.

<a id="class-mixin-or-mixin-class"></a>
## `class`, `mixin` hay `mixin class`?

:::version-note
Khai báo `mixin class` yêu cầu [language version][] tối thiểu là 3.0.
:::

Khai báo `mixin` định nghĩa một mixin. Khai báo `class` định nghĩa một [lớp][class].
Khai báo `mixin class` định nghĩa một lớp có thể được dùng vừa như lớp thông thường,
vừa như mixin, với cùng tên và cùng kiểu.

```dart
mixin class Musician {
  // ...
}

class Novice with Musician { // Dùng Musician như một mixin
  // ...
}

class Novice extends Musician { // Dùng Musician như một lớp
  // ...
}
```

Mọi hạn chế áp dụng cho lớp hoặc mixin cũng áp dụng cho mixin class:

- Mixin không thể có mệnh đề `extends` hoặc `with`, vì vậy `mixin class` cũng không thể.
- Lớp không thể có mệnh đề `on`, vì vậy `mixin class` cũng không thể.

[language version]: /language/versioning
[class]: /language/classes
[class modifiers]: /language/class-modifiers
