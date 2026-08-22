---
title: Class modifiers for API maintainers
description: >-
 Cách dùng các class modifier được thêm vào Dart 3.0
 để làm API của package vững chắc và dễ bảo trì hơn.
prevpage:
  url: /language/class-modifiers
  title: Class modifier
nextpage:
  url: /language/modifier-reference
  title: Tham chiếu class modifier
---

Dart 3.0 bổ sung một số [modifier mới][class modifiers]
mà bạn có thể đặt trên khai báo class và [mixin][].
Nếu là tác giả của một library package,
những modifier này cho bạn nhiều quyền kiểm soát hơn đối với những gì user được phép làm
với các kiểu mà package export.
Điều này có thể giúp package của bạn dễ phát triển hơn,
và giúp bạn dễ biết liệu một thay đổi trong code có thể làm hỏng code của user hay không.

[class modifiers]: /language/class-modifiers
[mixin]: /language/mixins

Dart 3.0 cũng bao gồm một [breaking change](/resources/dart-3-migration#mixin)
liên quan đến việc dùng lớp làm mixin.
Thay đổi này có thể không làm hỏng lớp *của bạn*,
nhưng có thể làm hỏng code của *user* sử dụng lớp đó.

Hướng dẫn này sẽ trình bày những thay đổi đó
để bạn biết cách dùng modifier mới,
và cách chúng ảnh hưởng đến user của library.

<a id="the-mixin-modifier-on-classes"></a>
## Modifier `mixin` trên lớp

Modifier quan trọng nhất cần lưu ý là `mixin`.
[Language version][Language versions] trước Dart 3.0 cho phép mọi lớp được dùng làm mixin
trong mệnh đề `with` của lớp khác, _TRỪ KHI_ lớp đó:

*   Khai báo bất kỳ constructor không phải factory nào.
*   Mở rộng bất kỳ lớp nào khác `Object`.

Điều này khiến bạn dễ vô tình làm hỏng code của người khác
bằng cách thêm constructor hoặc mệnh đề `extends` vào một lớp
mà không nhận ra người khác đang dùng nó trong mệnh đề `with`.

Dart 3.0 không còn mặc định cho phép dùng lớp làm mixin.
Thay vào đó, bạn phải chủ động chọn hành vi đó bằng cách khai báo `mixin class`:

```dart
mixin class Both {}

class UseAsMixin with Both {}
class UseAsSuperclass extends Both {}
```

Nếu cập nhật package lên Dart 3.0 mà không thay đổi bất kỳ code nào,
bạn có thể không thấy lỗi.
Nhưng bạn có thể vô tình làm hỏng code của user package
nếu họ đang dùng các lớp của bạn làm mixin.

[Language versions]: /language/versioning

<a id="migrating-classes-as-mixins"></a>
### Di chuyển các lớp được dùng làm mixin

Nếu lớp có constructor không phải factory, mệnh đề `extends`
hoặc mệnh đề `with`, thì nó vốn đã không thể được dùng làm mixin.
Hành vi sẽ không thay đổi với Dart 3.0;
không có gì phải lo lắng và bạn cũng không cần làm gì.

Trong thực tế, mô tả này áp dụng cho khoảng 90% lớp hiện có.
Đối với những lớp còn lại có thể được dùng làm mixin,
bạn phải quyết định mình muốn hỗ trợ điều gì.

Sau đây là một số câu hỏi giúp bạn quyết định. Câu đầu tiên mang tính thực tế:

*   **Bạn có muốn mạo hiểm làm hỏng code của bất kỳ user nào không?** Nếu câu trả lời chắc chắn là “không”,
    hãy đặt `mixin` trước mọi lớp
    [có thể được dùng làm mixin](#the-mixin-modifier-on-classes).
    Điều này bảo toàn chính xác hành vi hiện tại của API.

Mặt khác, nếu muốn tận dụng cơ hội này để
xem xét lại những khả năng mà API cung cấp,
bạn có thể muốn *không* biến nó thành `mixin class`.
Hãy cân nhắc hai câu hỏi thiết kế sau:

*   **Bạn có muốn user có thể trực tiếp dựng instance của nó không?**
    Nói cách khác, lớp có chủ ý không phải abstract không?

*   **Bạn có *muốn* mọi người có thể dùng khai báo làm mixin không?**
    Nói cách khác, bạn có muốn họ có thể dùng nó trong mệnh đề `with` không?

Nếu câu trả lời cho cả hai là “có”, hãy biến nó thành mixin class. Nếu câu trả lời cho
câu thứ hai là “không”, chỉ cần giữ nó là class. Nếu câu trả lời cho câu đầu tiên
là “không” và câu thứ hai là “có”, hãy đổi nó từ khai báo class thành
mixin.

Hai lựa chọn cuối—giữ nó là class hoặc biến nó thành pure mixin—
là các breaking change đối với API. Bạn nên tăng major version của package
nếu thực hiện việc này.

<a id="other-opt-in-modifiers"></a>
## Các modifier chủ động chọn khác

Xử lý class dưới dạng mixin là thay đổi quan trọng duy nhất trong Dart 3.0
ảnh hưởng đến API của package. Khi đã xử lý đến đây,
bạn có thể dừng nếu không muốn thực hiện thay đổi nào khác
đối với những gì package cho phép user làm.

Lưu ý rằng nếu tiếp tục và dùng bất kỳ modifier nào được mô tả bên dưới,
đó có khả năng là breaking change đối với API của package và đòi hỏi
tăng major version.

<a id="the-interface-modifier"></a>
## Modifier `interface`

Dart không có cú pháp riêng để khai báo pure interface.
Thay vào đó, bạn khai báo một abstract class tình cờ chỉ chứa
abstract method.
Khi user thấy lớp đó trong API của package,
họ có thể không biết liệu nó chứa code mà họ có thể tái sử dụng bằng cách mở rộng lớp,
hay thay vào đó nó được dùng như một interface.

Bạn có thể làm rõ điều đó bằng cách đặt modifier
[`interface`](/language/class-modifiers#interface) trên lớp.
Điều này cho phép lớp được dùng trong mệnh đề `implements`,
nhưng ngăn nó được dùng trong `extends`.

Ngay cả khi lớp *có* method không phải abstract,
bạn vẫn có thể muốn ngăn user mở rộng nó.
Kế thừa là một trong những kiểu coupling mạnh nhất trong phần mềm,
vì nó cho phép tái sử dụng code.
Nhưng coupling đó cũng [nguy hiểm và mong manh][dangerous and fragile].
Khi kế thừa vượt qua ranh giới package,
việc phát triển superclass mà không làm hỏng subclass có thể rất khó.

[dangerous and fragile]: https://en.wikipedia.org/wiki/Fragile_base_class

Đánh dấu lớp là `interface` cho phép user dựng nó (trừ khi nó [cũng được đánh dấu
`abstract`](/language/class-modifiers#abstract-interface))
và implement interface của lớp,
nhưng ngăn họ tái sử dụng bất kỳ code nào của lớp.

Khi lớp được đánh dấu `interface`, hạn chế này có thể được bỏ qua bên trong
library nơi lớp được khai báo.
Bên trong library, bạn được tự do mở rộng nó vì tất cả đều là code của bạn
và có lẽ bạn biết mình đang làm gì.
Hạn chế áp dụng cho package khác,
và thậm chí cả library khác trong chính package của bạn.

<a id="the-base-modifier"></a>
## Modifier `base`

Modifier [`base`](/language/class-modifiers#base)
phần nào đối lập với `interface`.
Nó cho phép bạn dùng lớp trong mệnh đề `extends`,
hoặc dùng mixin hay mixin class trong mệnh đề `with`.
Nhưng nó không cho phép code bên ngoài library của lớp
dùng lớp hoặc mixin trong mệnh đề `implements`.

Điều này bảo đảm rằng mọi object là instance
của interface lớp hoặc mixin đều kế thừa implementation thực tế của bạn.
Đặc biệt, điều đó có nghĩa mọi instance sẽ bao gồm
tất cả private member mà lớp hoặc mixin khai báo.
Điều này có thể giúp ngăn lỗi runtime có thể xảy ra nếu không có nó.

Hãy xem library sau:

```dart title="a.dart"
class A {
  void _privateMethod() {
    print('I inherited from A');
  }
}

void callPrivateMethod(A a) {
  a._privateMethod();
}
```

Đoạn code này tự nó có vẻ ổn,
nhưng không có gì ngăn user tạo một library khác như sau:

```dart title="b.dart"
import 'a.dart';

class B implements A {
  // Không có implementation của _privateMethod()!
}

main() {
  callPrivateMethod(B()); // Exception runtime!
}
```

Thêm modifier `base` vào lớp có thể giúp ngăn các lỗi runtime này.
Giống như với `interface`, bạn có thể bỏ qua hạn chế này
trong cùng library nơi base class hoặc mixin được khai báo.
Khi đó subclass trong cùng library
sẽ được nhắc implement private method.
Nhưng lưu ý rằng mục tiếp theo *có* áp dụng:

<a id="base-transitivity"></a>
### Tính bắc cầu của `base`

Mục tiêu của việc đánh dấu lớp là `base` là bảo đảm
mọi instance của kiểu đó thực sự kế thừa từ nó.
Để duy trì điều này, hạn chế base có tính “lan truyền”.
Mọi subtype của kiểu được đánh dấu `base`—*trực tiếp hoặc gián tiếp*—
cũng phải ngăn việc được implement.
Điều đó có nghĩa subtype phải được đánh dấu `base`
(hoặc `final` hay `sealed`, sẽ được trình bày tiếp theo).

Vì vậy, việc áp dụng `base` cho một kiểu đòi hỏi sự cẩn trọng.
Nó không chỉ ảnh hưởng đến những gì user có thể làm với lớp hoặc mixin của bạn,
mà còn ảnh hưởng đến những khả năng mà subclass *của họ* có thể cung cấp.
Một khi bạn đặt `base` trên một kiểu, toàn bộ hệ thống phân cấp bên dưới nó
bị cấm được implement.

Điều đó nghe có vẻ nghiêm ngặt, nhưng đây là cách hầu hết ngôn ngữ lập trình khác
luôn hoạt động.
Phần lớn không có interface ngầm định,
vì vậy khi khai báo class trong Java, C# hoặc ngôn ngữ khác,
về cơ bản bạn có cùng một ràng buộc.

<a id="the-final-modifier"></a>
## Modifier `final`

Nếu muốn tất cả hạn chế của cả `interface` và `base`,
bạn có thể đánh dấu class hoặc mixin class là [`final`](/language/class-modifiers#final).
Điều này ngăn bất kỳ ai bên ngoài library tạo
bất kỳ loại subtype nào của nó:
không được dùng nó trong mệnh đề `implements`, `extends`, `with` hoặc `on`.

Đây là mức hạn chế cao nhất đối với user của lớp.
Tất cả những gì họ có thể làm là dựng nó (trừ khi nó được đánh dấu `abstract`).
Đổi lại, bạn có ít hạn chế nhất với tư cách người duy trì lớp.
Bạn có thể thêm method mới, biến constructor thành factory constructor, v.v.
mà không lo làm hỏng code của user downstream.

<a id="the-sealed-modifer" aria-hidden="true"></a>

<a id="the-sealed-modifier"></a>
## Modifier `sealed`

Modifier cuối cùng, [`sealed`](/language/class-modifiers#sealed), là một trường hợp đặc biệt.
Nó tồn tại chủ yếu để hỗ trợ [kiểm tra exhaustiveness][exhaustiveness checking] trong pattern matching.
Nếu switch có case cho mọi direct subtype của một kiểu được đánh dấu `sealed`,
compiler sẽ biết switch là exhaustive.

[exhaustiveness checking]: /language/branches#exhaustiveness-checking

<?code-excerpt "language/lib/class_modifiers/sealed_exhaustiveness.dart"?>
```dart title="amigos.dart"
sealed class Amigo {}

class Lucky extends Amigo {}

class Dusty extends Amigo {}

class Ned extends Amigo {}

String lastName(Amigo amigo) => switch (amigo) {
  Lucky _ => 'Day',
  Dusty _ => 'Bottoms',
  Ned _ => 'Nederlander',
};
```

Switch này có một case cho mỗi subtype của `Amigo`.
Compiler biết rằng mọi instance của `Amigo` phải là instance của một
trong các subtype đó, vì vậy nó biết switch an toàn và exhaustive, đồng thời không
yêu cầu case default cuối cùng.

Để điều này hợp lý, compiler áp đặt hai hạn chế:

1.  Bản thân sealed class không thể được dựng trực tiếp.
    Nếu không, bạn có thể có một instance của `Amigo` không phải
    instance của *bất kỳ* subtype nào.
    Vì vậy mọi class `sealed` cũng ngầm là `abstract`.

2.  Mọi direct subtype của sealed type phải nằm trong cùng library
    nơi sealed type được khai báo.
    Bằng cách này, compiler có thể tìm thấy tất cả. Nó biết không có
    subtype ẩn nào khác không khớp với bất kỳ case nào.

Hạn chế thứ hai tương tự `final`.
Giống `final`, nó có nghĩa là lớp được đánh dấu `sealed` không thể trực tiếp
được mở rộng, implement hoặc mix in bên ngoài library nơi nó được khai báo.
Nhưng không giống `base` và `final`, không có hạn chế *bắc cầu*:

```dart title="amigo.dart"
sealed class Amigo {}
class Lucky extends Amigo {}
class Dusty extends Amigo {}
class Ned extends Amigo {}
```

```dart title="other.dart"
// Đây là lỗi:
class Bad extends Amigo {}

// Nhưng cả hai khai báo này đều hợp lệ:
class OtherLucky extends Lucky {}
class OtherDusty implements Dusty {}
```

Dĩ nhiên, nếu bạn *muốn* subtype của sealed type
cũng bị hạn chế, bạn có thể làm điều đó bằng cách đánh dấu chúng
với `interface`, `base`, `final` hoặc `sealed`.

<a id="sealed-versus-final"></a>
### `sealed` so với `final`

Nếu có một lớp mà bạn không muốn user có thể trực tiếp tạo subtype,
khi nào bạn nên dùng `sealed` thay vì `final`?
Có một vài quy tắc đơn giản:

*   Nếu muốn user có thể trực tiếp dựng instance của lớp,
    lớp đó *không thể* dùng `sealed` vì sealed type ngầm là abstract.

*   Nếu lớp không có subtype trong library của bạn, việc dùng
    `sealed` không có ý nghĩa vì bạn không nhận được lợi ích từ kiểm tra exhaustiveness.

Trong các trường hợp khác, nếu lớp có một số subtype do bạn định nghĩa,
thì `sealed` có khả năng là lựa chọn bạn muốn.
Nếu user thấy lớp có một vài subtype, việc có thể
xử lý riêng từng subtype dưới dạng case của switch
và để compiler biết toàn bộ kiểu đã được bao phủ sẽ rất tiện.

Dùng `sealed` đồng nghĩa nếu sau này bạn thêm một subtype khác vào library,
đó sẽ là breaking change đối với API.
Khi subtype mới xuất hiện,
tất cả switch hiện có trở thành non-exhaustive
vì chúng không xử lý kiểu mới.
Điều này hoàn toàn giống việc thêm một giá trị mới vào enum.

Các lỗi compile của switch non-exhaustive này *hữu ích* cho user
vì chúng thu hút sự chú ý của user đến những nơi trong code
mà họ cần xử lý kiểu mới.

Nhưng điều đó cũng có nghĩa bất cứ khi nào bạn thêm subtype mới, đây là breaking change.
Nếu muốn tự do thêm subtype mới theo cách không gây breaking change,
tốt hơn hết hãy đánh dấu supertype bằng `final` thay vì `sealed`.
Điều đó có nghĩa khi user switch trên một giá trị của supertype đó,
ngay cả khi họ có case cho tất cả subtype,
compiler vẫn buộc họ thêm một case default khác.
Case default đó sẽ là phần được thực thi nếu bạn thêm subtype sau này.

<a id="summary"></a>
## Tổng kết

Với tư cách người thiết kế API,
những modifier mới này cho bạn quyền kiểm soát cách user làm việc với code,
và ngược lại là cách bạn có thể phát triển code mà không làm hỏng code của họ.

Nhưng các lựa chọn này mang theo sự phức tạp:
giờ bạn có nhiều quyết định hơn phải đưa ra với tư cách người thiết kế API.
Ngoài ra, vì các tính năng này còn mới,
chúng ta vẫn chưa biết best practice sẽ là gì.
Hệ sinh thái của mỗi ngôn ngữ đều khác nhau và có nhu cầu khác nhau.

May mắn là bạn không cần tìm ra tất cả ngay lập tức.
Các giá trị mặc định được chọn có chủ ý để ngay cả khi bạn không làm gì,
các lớp phần lớn vẫn có những khả năng như trước phiên bản 3.0.
Nếu chỉ muốn giữ nguyên API,
hãy đặt `mixin` trên những lớp vốn đã hỗ trợ điều đó, và bạn đã hoàn tất.

Theo thời gian, khi hiểu rõ nơi cần quyền kiểm soát chi tiết hơn,
bạn có thể cân nhắc áp dụng một số modifier khác:

*   Dùng `interface` để ngăn user tái sử dụng code của lớp
    trong khi vẫn cho phép họ implement lại interface của lớp.

*   Dùng `base` để yêu cầu user tái sử dụng code của lớp
    và bảo đảm mọi instance thuộc kiểu của lớp là instance
    của chính lớp đó hoặc subclass.

*   Dùng `final` để ngăn hoàn toàn việc mở rộng lớp.

*   Dùng `sealed` để chủ động bật kiểm tra exhaustiveness trên một họ subtype.

Khi làm vậy, hãy tăng major version lúc publish package,
vì tất cả modifier này đều kéo theo những hạn chế là breaking change.
