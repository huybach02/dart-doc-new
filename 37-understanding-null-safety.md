---
title: Tìm hiểu sâu về null safety
description: >-
    Phân tích chuyên sâu những thay đổi trong ngôn ngữ và thư viện Dart liên quan đến null safety.
---

_Viết bởi Bob Nystrom<br>
Tháng 7 năm 2020_

Null safety là thay đổi lớn nhất mà chúng tôi thực hiện với Dart kể từ khi thay thế
hệ thống kiểu tùy chọn unsound ban đầu bằng [một hệ thống kiểu static sound][strong]
trong Dart 2.0. Khi Dart ra mắt lần đầu, null safety ở compile time là một tính năng
hiếm gặp và cần phần giới thiệu dài. Ngày nay, Kotlin, Swift, Rust và các
ngôn ngữ khác đều có lời giải riêng cho vấn đề đã trở nên rất [quen
thuộc.][billion] Đây là một ví dụ:

[strong]: /language/type-system
[billion]: https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/

```dart
// Không có null safety:
bool isEmpty(String string) => string.length == 0;

void main() {
  isEmpty(null);
}
```

Nếu chạy chương trình Dart này không có null safety, nó sẽ throw exception
`NoSuchMethodError` tại lệnh gọi `.length`. Value `null` là một
instance của class `Null`, và `Null` không có getter "length". Lỗi runtime
thật tệ. Điều này đặc biệt đúng với một ngôn ngữ như Dart, được thiết kế
để chạy trên thiết bị của người dùng cuối. Nếu ứng dụng server thất bại, bạn thường có thể
khởi động lại trước khi có ai nhận ra. Nhưng khi ứng dụng Flutter crash trên điện thoại
của người dùng, họ sẽ không hài lòng. Khi người dùng không hài lòng, bạn cũng không hài lòng.

Developer thích các ngôn ngữ static-typed như Dart vì chúng cho phép
type checker tìm lỗi trong code ở compile time, thường ngay trong IDE.
Bạn tìm thấy bug càng sớm thì càng có thể sửa sớm. Khi nhà thiết kế ngôn ngữ
nói về "sửa lỗi null reference", họ muốn nói đến việc làm phong phú static type
checker để ngôn ngữ có thể phát hiện lỗi như nỗ lực gọi
`.length` trên một value có thể là `null` ở trên.

Không có một giải pháp đúng duy nhất cho vấn đề này. Rust và Kotlin đều có
cách tiếp cận riêng phù hợp với bối cảnh của các ngôn ngữ đó. Tài liệu này trình bày
toàn bộ chi tiết về lời giải của chúng tôi cho Dart. Nó bao gồm những thay đổi đối với
hệ thống kiểu static cùng một bộ thay đổi và tính năng ngôn ngữ mới khác,
không chỉ giúp bạn viết code null-safe mà hy vọng còn *thích thú* khi làm vậy.

Tài liệu này dài. Nếu bạn muốn nội dung ngắn hơn, chỉ trình bày những gì
cần biết để bắt đầu, hãy bắt đầu với [tổng quan][]. Khi đã
sẵn sàng tìm hiểu sâu hơn và có thời gian, hãy quay lại đây để hiểu
ngôn ngữ xử lý `null` *như thế nào*, *tại sao* chúng tôi thiết kế như vậy và
cách viết Dart hiện đại, null-safe, đúng idiom. (Tiết lộ trước: kết quả
gần với cách bạn viết Dart hiện nay đến bất ngờ.)

[overview]: /null-safety

Mỗi cách mà ngôn ngữ có thể dùng để xử lý lỗi null reference đều có
ưu và nhược điểm. Những nguyên tắc sau định hướng các lựa chọn của chúng tôi:

*   **Code phải an toàn theo mặc định.** Nếu bạn viết code Dart mới và không sử dụng
    bất kỳ tính năng không an toàn rõ ràng nào, code sẽ không bao giờ throw lỗi null reference ở
    runtime. Mọi lỗi null reference có thể xảy ra đều được bắt theo cách static. Nếu
    muốn trì hoãn một phần việc kiểm tra đó đến runtime để có độ linh hoạt cao hơn,
    bạn có thể làm vậy, nhưng phải chủ động chọn bằng cách sử dụng một tính năng hiển thị
    rõ ràng trong code.

    Nói cách khác, chúng tôi không đưa cho bạn áo phao rồi để bạn tự
    nhớ mặc nó mỗi lần xuống nước. Thay vào đó, chúng tôi
    đưa cho bạn một con thuyền không chìm. Bạn sẽ khô ráo trừ khi nhảy ra khỏi thuyền.

*   **Code null-safe phải dễ viết.** Phần lớn code Dart hiện có
    đúng về mặt dynamic và không throw lỗi null reference. Bạn thích chương trình
    Dart của mình theo cách hiện tại, và chúng tôi muốn bạn có thể tiếp tục
    viết code như vậy. Sự an toàn không nên đòi hỏi hy sinh tính dễ dùng,
    chịu phạt bởi type checker hoặc phải thay đổi đáng kể
    cách tư duy.

*   **Code null-safe tạo ra phải hoàn toàn sound.** "Soundness" trong
    bối cảnh kiểm tra static có ý nghĩa khác nhau với mỗi người. Với
    chúng tôi, trong bối cảnh null safety, điều đó nghĩa là nếu một biểu thức có
    kiểu static không cho phép `null`, thì không lần thực thi nào của biểu thức đó
    có thể đánh giá thành `null`. Ngôn ngữ cung cấp bảo đảm này
    chủ yếu qua kiểm tra static, nhưng cũng có thể liên quan đến một số kiểm tra runtime.
    (Tuy vậy, hãy nhớ nguyên tắc đầu tiên: bất kỳ nơi nào những kiểm tra runtime đó
    xảy ra đều do bạn lựa chọn.)

    Soundness quan trọng đối với niềm tin của người dùng. Một con thuyền *hầu như*
    luôn nổi không phải là con thuyền bạn hào hứng dùng để vượt biển. Nhưng nó cũng
    quan trọng với những hacker compiler dũng cảm của chúng tôi. Khi ngôn ngữ đưa ra bảo đảm
    chắc chắn về các thuộc tính ngữ nghĩa của chương trình, compiler có thể thực hiện
    tối ưu hóa với giả định các thuộc tính đó là đúng. Với `null`, điều này
    nghĩa là chúng tôi có thể tạo code nhỏ hơn bằng cách loại bỏ kiểm tra `null` không cần thiết,
    và code nhanh hơn do không phải xác minh receiver là non-`null` trước khi gọi method.

    Một điểm cần lưu ý: Chúng tôi chỉ bảo đảm soundness trong chương trình Dart hoàn toàn null
    safe. Dart hỗ trợ chương trình chứa hỗn hợp code null-safe mới hơn
    và code legacy cũ hơn. Trong các chương trình dùng nhiều version này, lỗi null reference
    vẫn có thể xảy ra. Trong chương trình dùng nhiều version, bạn nhận được toàn bộ lợi ích
    an toàn *static* trong các phần null-safe, nhưng không có soundness runtime đầy đủ
    cho đến khi toàn bộ ứng dụng null-safe.

Lưu ý rằng *loại bỏ* `null` không phải mục tiêu. `null` không có gì sai.
Ngược lại, khả năng biểu diễn *sự vắng mặt* của một value rất hữu ích.
Xây dựng hỗ trợ cho một value "vắng mặt" đặc biệt ngay trong ngôn ngữ
giúp việc xử lý sự vắng mặt linh hoạt và dễ dùng. Nó làm nền tảng cho optional
parameter, operator null-aware `?.` tiện lợi và khởi tạo mặc định. Không phải
`null` là xấu, mà chính việc `null` đi *đến nơi bạn không mong đợi*
mới gây ra vấn đề.

Do đó, với null safety, mục tiêu là trao cho bạn *quyền kiểm soát* và *hiểu biết* về
nơi `null` có thể chảy qua chương trình, cùng sự chắc chắn rằng nó không thể chảy
đến nơi gây crash.

<a id="nullability-in-the-type-system"></a>
## Khả năng nullable trong hệ thống kiểu

Null safety bắt đầu trong hệ thống kiểu static vì mọi thứ khác đều dựa trên
nó. Chương trình Dart của bạn có cả một vũ trụ kiểu: các kiểu primitive
như `int` và `String`, các kiểu collection như `List`, cùng tất cả class
và kiểu do bạn và các package bạn sử dụng định nghĩa. Trước null safety, hệ thống kiểu
static cho phép value `null` chảy vào biểu thức thuộc bất kỳ kiểu nào trong số đó.

Theo thuật ngữ lý thuyết kiểu, kiểu `Null` được xem là subtype của mọi kiểu:

<img src="/assets/img/null-safety/understanding-null-safety/hierarchy-before.png" alt="Hệ thống phân cấp trước Null Safety" width="335">

Tập hợp thao tác—getter, setter, method và
operator—được phép trên một số biểu thức được xác định bởi kiểu của nó. Nếu kiểu
là `List`, bạn có thể gọi `.add()` hoặc `[]` trên đó. Nếu là `int`, bạn có thể gọi `+`.
Nhưng value `null` không định nghĩa bất kỳ method nào trong số đó. Việc cho phép `null`
chảy vào biểu thức thuộc kiểu khác nghĩa là bất kỳ thao tác nào cũng có thể
thất bại. Đây thực sự là cốt lõi của lỗi null reference—mọi lỗi đều
bắt nguồn từ việc cố tra cứu một method hoặc property không tồn tại trên `null`.

<a id="non-nullable-and-nullable-types"></a>
### Kiểu non-nullable và nullable

Null safety loại bỏ vấn đề tận gốc bằng cách thay đổi hệ thống phân cấp kiểu.
Kiểu `Null` vẫn tồn tại, nhưng không còn là subtype của mọi kiểu.
Thay vào đó, hệ thống phân cấp kiểu trông như sau:

<img src="/assets/img/null-safety/understanding-null-safety/hierarchy-after.png" alt="Hệ thống phân cấp sau Null Safety" width="344">

Vì `Null` không còn là subtype, không kiểu nào ngoài class `Null` đặc biệt
cho phép value `null`. Chúng tôi đã làm mọi kiểu trở thành *non-nullable theo mặc định*. Nếu bạn
có biến kiểu `String`, nó sẽ luôn chứa *một string*. Vậy là
chúng ta đã sửa mọi lỗi null reference.

Nếu cho rằng `null` hoàn toàn không hữu ích, chúng tôi có thể dừng ở đây. Nhưng `null`
hữu ích nên vẫn cần một cách xử lý nó. Optional parameter là một
trường hợp minh họa tốt. Hãy xem code Dart null-safe này:

```dart
// Sử dụng null safety:
void makeCoffee(String coffee, [String? dairy]) {
  if (dairy != null) {
    print('$coffee with $dairy');
  } else {
    print('Black $coffee');
  }
}
```

Ở đây, chúng ta muốn cho phép tham số `dairy` nhận bất kỳ string nào hoặc value
`null`, nhưng không nhận gì khác. Để biểu diễn điều đó, chúng ta gán cho `dairy` một *kiểu nullable* bằng cách
đặt `?` ở cuối kiểu cơ sở `String`. Bên trong,
về cơ bản đây là việc định nghĩa một [union][] của kiểu cơ sở và kiểu `Null`.
Vì vậy, `String?` sẽ là dạng viết tắt của `String|Null` nếu Dart có
kiểu union đầy đủ tính năng.

[union]: https://en.wikipedia.org/wiki/Union_type

<a id="using-nullable-types"></a>
### Sử dụng kiểu nullable

Nếu có một biểu thức với kiểu nullable, bạn có thể làm gì với kết quả?
Vì nguyên tắc của chúng tôi là an toàn theo mặc định, câu trả lời là không nhiều. Chúng tôi không thể cho phép bạn
gọi các method của kiểu cơ sở trên đó vì chúng có thể thất bại nếu value
là `null`:

```dart
// Null safety unsound giả định:
void bad(String? maybeString) {
  print(maybeString.length);
}

void main() {
  bad(null);
}
```

Code này sẽ crash nếu chúng tôi cho phép bạn chạy. Các method và property duy nhất mà chúng tôi có thể
an toàn cho phép bạn truy cập là những thành phần được cả kiểu cơ sở và class `Null`
định nghĩa. Đó chỉ là `toString()`, `==` và `hashCode`. Vì vậy, bạn có thể dùng
kiểu nullable làm key của map, lưu chúng trong set, so sánh với value khác,
và dùng trong nội suy string, nhưng chỉ có vậy.

Chúng tương tác với kiểu non-nullable như thế nào? Việc truyền một kiểu
*non*-nullable vào nơi mong đợi kiểu nullable luôn an toàn. Nếu một hàm
chấp nhận `String?`, truyền `String` được phép vì không gây ra vấn đề.
Chúng tôi mô hình hóa điều này bằng cách biến mọi kiểu nullable thành supertype của
kiểu cơ sở. Bạn cũng có thể an toàn truyền `null` vào nơi mong đợi kiểu nullable, nên
`Null` cũng là subtype của mọi kiểu nullable:

<img src="/assets/img/null-safety/understanding-null-safety/nullable-hierarchy.png" alt="Nullable" width="235">

Nhưng đi theo hướng ngược lại và truyền kiểu nullable vào nơi mong đợi
kiểu non-nullable cơ sở là không an toàn. Code mong đợi `String` có thể
gọi các method String trên value. Nếu bạn truyền `String?`, `null` có thể
chảy vào và gây lỗi:

```dart
// Null safety unsound giả định:
void requireStringNotNull(String definitelyString) {
  print(definitelyString.length);
}

void main() {
  String? maybeString = null; // Hoặc không!
  requireStringNotNull(maybeString);
}
```

Chương trình này không an toàn và chúng ta không nên cho phép nó. Tuy nhiên, Dart luôn có
một thứ gọi là *implicit downcast*. Chẳng hạn, nếu bạn truyền một value kiểu
`Object` vào hàm mong đợi `String`, type checker cho phép:

```dart
// Không có null safety:
void requireStringNotObject(String definitelyString) {
  print(definitelyString.length);
}

void main() {
  Object maybeString = 'it is';
  requireStringNotObject(maybeString);
}
```

Để duy trì soundness, compiler ngầm chèn cast `as String` trên
đối số cho `requireStringNotObject()`. Cast đó có thể thất bại và throw
exception ở runtime, nhưng tại compile time, Dart nói điều này ổn. Vì
kiểu non-nullable được mô hình hóa là subtype của kiểu nullable, implicit downcast
sẽ cho phép bạn truyền `String?` vào nơi mong đợi `String`. Cho phép điều đó
sẽ vi phạm mục tiêu an toàn theo mặc định. Vì vậy, với null safety, chúng tôi
loại bỏ hoàn toàn implicit downcast.

Điều này khiến lệnh gọi `requireStringNotNull()` tạo compile error, đúng như
bạn mong muốn. Nhưng nó cũng có nghĩa là *mọi* implicit downcast trở thành compile
error, bao gồm lệnh gọi `requireStringNotObject()`. Bạn sẽ phải tự thêm
explicit downcast:

```dart
// Sử dụng null safety:
void requireStringNotObject(String definitelyString) {
  print(definitelyString.length);
}

void main() {
  Object maybeString = 'it is';
  requireStringNotObject(maybeString as String);
}
```

Chúng tôi cho rằng đây là thay đổi tốt về tổng thể. Ấn tượng của chúng tôi là phần lớn người dùng chưa bao giờ
thích implicit downcast. Đặc biệt, có thể trước đây bạn từng gặp rắc rối với điều này:

```dart
// Không có null safety:
List<int> filterEvens(List<int> ints) {
  return ints.where((n) => n.isEven);
}
```

Bạn phát hiện bug không? Method `.where()` là lazy, nên nó trả về `Iterable`, không phải
`List`. Chương trình này compile nhưng sau đó throw exception ở runtime khi
cố cast `Iterable` đó thành kiểu `List` mà `filterEvens` khai báo sẽ
trả về. Khi implicit downcast bị loại bỏ, đây trở thành compile error.

Chúng ta đang nói đến đâu rồi? Đúng, như thể chúng ta lấy vũ trụ kiểu trong
chương trình và chia thành hai nửa:

<img src="/assets/img/null-safety/understanding-null-safety/bifurcate.png" alt="Kiểu Nullable và Non-Nullable" width="668">

Có một vùng kiểu non-nullable. Các kiểu đó cho phép bạn truy cập mọi
method thú vị, nhưng tuyệt đối không bao giờ chứa `null`. Và có một
họ song song gồm tất cả kiểu nullable tương ứng. Chúng cho phép `null`,
nhưng bạn không thể làm nhiều với chúng. Chúng tôi cho value chảy từ phía non-nullable
sang phía nullable vì điều đó an toàn, nhưng không cho đi theo hướng ngược lại.

Điều đó khiến kiểu nullable dường như về cơ bản vô dụng. Chúng không có method và
bạn không thể thoát khỏi chúng. Đừng lo, chúng tôi có cả một bộ tính năng để
giúp bạn chuyển value từ nửa nullable sang phía bên kia mà chúng ta sẽ sớm tìm hiểu.

<a id="top-and-bottom"></a>
### Top và bottom

Phần này hơi mang tính chuyên môn sâu. Bạn hầu như có thể bỏ qua, ngoại trừ hai
bullet ở cuối, trừ khi bạn thích kiến thức về hệ thống kiểu. Hãy hình dung tất cả
kiểu trong chương trình với các cạnh giữa những kiểu là subtype và supertype
của nhau. Nếu vẽ ra như các sơ đồ trong tài liệu này, nó sẽ
tạo thành một đồ thị có hướng khổng lồ với supertype như `Object` gần đỉnh và class
lá như kiểu của riêng bạn gần đáy.

Nếu đồ thị có hướng đó hội tụ tại đỉnh, nơi có một kiểu duy nhất
là supertype (trực tiếp hoặc gián tiếp), kiểu đó được gọi là *top
type*. Tương tự, nếu có một kiểu kỳ lạ ở đáy là subtype của
mọi kiểu, bạn có một *bottom type*. (Trong trường hợp này, đồ thị có hướng là một
[lattice.][lattice])

[lattice]: https://en.wikipedia.org/wiki/Lattice_(order)

Hệ thống kiểu có top type và bottom type rất tiện lợi vì các thao tác
cấp kiểu như least upper bound (mà suy luận kiểu dùng để
xác định kiểu của biểu thức điều kiện dựa trên kiểu của hai nhánh)
luôn có thể tạo ra một kiểu. Trước null safety, `Object` là top type của Dart
và `Null` là bottom type.

Vì `Object` giờ là non-nullable, nó không còn là top type. `Null` không phải
subtype của nó. Dart không có top type *có tên*. Nếu cần top type, bạn cần
`Object?`. Tương tự, `Null` không còn là bottom type. Nếu đúng như vậy, mọi thứ
vẫn sẽ nullable. Thay vào đó, chúng tôi thêm một bottom type mới tên `Never`:

<img src="/assets/img/null-safety/understanding-null-safety/top-and-bottom.png" alt="Top và Bottom" width="360">

Trong thực tế, điều này có nghĩa:

*   Nếu muốn biểu thị rằng bạn cho phép value thuộc bất kỳ kiểu nào, hãy dùng `Object?`
    thay cho `Object`. Thực tế, việc dùng `Object` trở nên khá hiếm
    vì kiểu đó nghĩa là "có thể là mọi value ngoại trừ một value kỳ lạ bị
    cấm là `null`".

*   Trong trường hợp hiếm hoi cần bottom type, hãy dùng
    `Never` thay cho `Null`.
    Điều này đặc biệt hữu ích để biểu thị một hàm không bao giờ trả về nhằm
    [hỗ trợ phân tích reachability](#never-for-unreachable-code).
    Nếu không biết mình có cần bottom type hay không, có lẽ bạn không cần.

<a id="ensuring-correctness"></a>
## Đảm bảo tính đúng đắn

Chúng tôi đã chia vũ trụ kiểu thành hai nửa nullable và non-nullable. Để
duy trì soundness và nguyên tắc rằng bạn không bao giờ có thể gặp lỗi null reference
ở runtime trừ khi yêu cầu điều đó, chúng tôi cần bảo đảm `null` không bao giờ
xuất hiện trong bất kỳ kiểu nào ở phía non-nullable.

Việc loại bỏ implicit downcast và loại `Null` khỏi vai trò bottom type bao phủ
mọi vị trí chính mà kiểu chảy qua chương trình qua assignment và
từ argument vào parameter trong lệnh gọi hàm. Các vị trí chính còn lại
nơi `null` có thể lẻn vào là khi biến mới xuất hiện và khi
bạn rời khỏi hàm. Vì vậy có thêm một số compile error:

<a id="invalid-returns"></a>
### Return không hợp lệ

Nếu một hàm có return type non-nullable, mọi path qua
hàm phải đến được câu lệnh `return` trả về value. Trước null
safety, Dart khá dễ dãi với việc thiếu return. Ví dụ:

```dart
// Không có null safety:
String missingReturn() {
  // Không có return.
}
```

Nếu phân tích code này, bạn nhận được một *gợi ý* nhẹ rằng *có lẽ* bạn quên return,
nhưng nếu không thì cũng không có gì nghiêm trọng. Đó là vì nếu việc thực thi đến cuối
thân hàm, Dart ngầm trả về `null`. Vì mọi kiểu đều nullable,
*về mặt kỹ thuật* hàm này an toàn, dù có lẽ không phải điều bạn muốn.

Với kiểu non-nullable sound, chương trình này hoàn toàn sai và không an toàn. Trong
null safety, bạn nhận compile error nếu hàm có return type non-nullable
không trả về value một cách đáng tin cậy. "Đáng tin cậy" nghĩa là ngôn ngữ
phân tích mọi path control flow qua hàm. Miễn là tất cả đều
trả về thứ gì đó, yêu cầu được thỏa mãn. Phân tích khá thông minh, nên ngay cả hàm này
cũng hợp lệ:

```dart
// Sử dụng null safety:
String alwaysReturns(int n) {
  if (n == 0) {
    return 'zero';
  } else if (n < 0) {
    throw ArgumentError('Negative values not allowed.');
  } else {
    if (n > 1000) {
      return 'big';
    } else {
      return n.toString();
    }
  }
}
```

Chúng ta sẽ đi sâu hơn vào flow analysis mới trong phần tiếp theo.

<a id="uninitialized-variables"></a>
### Biến chưa được khởi tạo

Khi khai báo biến mà không cung cấp initializer rõ ràng, Dart
mặc định khởi tạo biến bằng `null`. Điều đó tiện lợi nhưng rõ ràng
hoàn toàn không an toàn nếu kiểu của biến là non-nullable. Vì vậy, chúng ta phải siết chặt
quy tắc cho biến non-nullable:

*   **Khai báo biến top-level và static field phải có
    initializer.** Vì chúng có thể được truy cập và gán từ bất kỳ đâu trong
    chương trình, compiler không thể bảo đảm biến đã
    được cung cấp value trước khi sử dụng. Lựa chọn an toàn duy nhất là yêu cầu
    chính khai báo có biểu thức khởi tạo tạo ra
    value đúng kiểu:

    ```dart
    // Sử dụng null safety:
    int topLevel = 0;

    class SomeClass {
      static int staticField = 0;
    }
    ```

*   **Instance field phải có initializer tại khai báo, sử dụng một
    initializing formal hoặc được khởi tạo trong initialization
    list của constructor.** Có khá nhiều thuật ngữ. Đây là ví dụ:

    ```dart
    // Sử dụng null safety:
    class SomeClass {
      int atDeclaration = 0;
      int initializingFormal;
      int initializationList;

      SomeClass(this.initializingFormal)
          : initializationList = 0;
    }
    ```

    Nói cách khác, miễn là field có value trước khi bạn đến
    thân constructor thì mọi thứ ổn.

*   Biến local là trường hợp linh hoạt nhất. Một biến local non-nullable
    *không* cần có initializer. Code này hoàn toàn hợp lệ:

    ```dart
    // Sử dụng null safety:
    int tracingFibonacci(int n) {
      int result;
      if (n < 2) {
        result = n;
      } else {
        result = tracingFibonacci(n - 2) + tracingFibonacci(n - 1);
      }

      print(result);
      return result;
    }
    ```

    Quy tắc duy nhất là **biến local phải được *gán chắc chắn*
    trước khi sử dụng.** Ở đây chúng ta cũng dựa vào flow analysis mới đã nhắc đến.
    Miễn là mọi path đến nơi sử dụng biến đều khởi tạo nó
    trước, việc sử dụng là hợp lệ.

*   **Optional parameter phải có default value.** Nếu bạn không truyền
    argument cho optional positional hoặc named parameter, ngôn ngữ
    điền default value. Nếu bạn không chỉ định default value,
    default value _mặc định_ là `null`, và điều đó không hợp lệ nếu kiểu của parameter
    là non-nullable.

    Vì vậy, nếu muốn parameter là optional, bạn cần biến nó thành
    nullable hoặc chỉ định một default value non-`null` hợp lệ.

Những hạn chế này nghe có vẻ nặng nề, nhưng trong thực tế không quá tệ. Chúng
rất giống các hạn chế hiện có quanh biến `final`, và có lẽ
bạn đã làm việc với chúng nhiều năm mà hầu như không nhận ra. Ngoài ra,
hãy nhớ chúng chỉ áp dụng cho biến *non-nullable*. Bạn luôn có thể làm
kiểu nullable rồi nhận khởi tạo mặc định thành `null`.

Dù vậy, các quy tắc vẫn gây ra trở ngại. May thay, chúng tôi có một bộ
tính năng ngôn ngữ mới để làm trơn tru các pattern phổ biến nhất nơi những giới hạn
mới này làm bạn chậm lại. Nhưng trước tiên, đã đến lúc nói về flow analysis.

<a id="flow-analysis"></a>
## Flow analysis

[Control flow analysis][] đã tồn tại trong compiler nhiều năm. Nó phần lớn
ẩn khỏi người dùng và được dùng khi tối ưu hóa compiler, nhưng một số ngôn ngữ mới
đã bắt đầu sử dụng các kỹ thuật tương tự cho tính năng ngôn ngữ nhìn thấy được.
Dart đã có một chút flow analysis dưới dạng *type promotion*:

```dart
// Có (hoặc không có) null safety:
bool isEmptyList(Object object) {
  if (object is List) {
    return object.isEmpty; // <-- Hợp lệ!
  } else {
    return false;
  }
}
```

[control flow analysis]: https://en.wikipedia.org/wiki/Control_flow_analysis

Hãy lưu ý rằng trên dòng được đánh dấu, chúng ta có thể gọi `isEmpty` trên `object`. Method đó
được định nghĩa trên `List`, không phải `Object`. Điều này hoạt động vì type checker xem xét
mọi biểu thức `is` và các path control flow trong chương trình. Nếu thân của
một cấu trúc control flow chỉ thực thi khi một biểu thức `is` nhất định
trên biến là true, thì bên trong thân đó, kiểu của biến được "promote"
thành kiểu được kiểm tra.

Trong ví dụ này, nhánh then của câu lệnh `if` chỉ chạy khi
`object` thực sự chứa một list. Do đó, Dart promote `object` thành kiểu
`List` thay vì kiểu được khai báo là `Object`. Đây là tính năng tiện lợi, nhưng
khá hạn chế. Trước null safety, chương trình giống hệt về chức năng sau
không hoạt động:

```dart
// Không có null safety:
bool isEmptyList(Object object) {
  if (object is! List) return false;
  return object.isEmpty; // <-- Lỗi!
}
```

Một lần nữa, bạn chỉ có thể đến lệnh gọi `.isEmpty` khi `object` chứa list, nên
chương trình đúng về mặt dynamic. Nhưng quy tắc type promotion không đủ thông minh
để thấy rằng câu lệnh `return` nghĩa là chỉ có thể đến câu lệnh thứ hai khi
`object` là list.

Với null safety, chúng tôi đã lấy phân tích hạn chế này và làm nó [mạnh mẽ hơn
nhiều theo một số cách.][flow analysis]

[flow analysis]: {{site.repo.dart.lang}}/blob/main/resources/type-system/flow-analysis.md

<a id="reachability-analysis"></a>
### Phân tích reachability

Trước hết, chúng tôi đã sửa [khiếu nại lâu năm][18921] rằng type promotion
không thông minh với early return và các path code không thể đến được khác. Khi phân tích
một hàm, giờ nó tính đến `return`, `break`, `throw` và mọi cách khác
mà việc thực thi có thể kết thúc sớm trong hàm. Với null safety, hàm này:

[18921]: {{site.repo.dart.sdk}}/issues/18921

```dart
// Sử dụng null safety:
bool isEmptyList(Object object) {
  if (object is! List) return false;
  return object.isEmpty;
}
```

Giờ hoàn toàn hợp lệ. Vì câu lệnh `if` sẽ thoát hàm khi
`object` *không phải* `List`, Dart promote `object` thành `List` ở câu lệnh
thứ hai. Đây là cải tiến rất hay giúp ích nhiều cho code Dart, kể cả
code không liên quan đến nullability.

<a id="never-for-unreachable-code"></a>
### Never cho code không thể đến được

Bạn cũng có thể *lập trình* phân tích reachability này. Bottom type mới `Never`
không có value. (Loại value nào đồng thời là `String`, `bool` và
`int`?) Vậy một biểu thức có kiểu `Never` nghĩa là gì? Nó nghĩa là
biểu thức đó không bao giờ có thể hoàn tất việc đánh giá thành công. Nó phải throw
exception, abort hoặc bằng cách khác bảo đảm code xung quanh đang chờ
kết quả của biểu thức không bao giờ chạy.

Thực tế, theo ngôn ngữ, kiểu static của biểu thức `throw` là
`Never`. Kiểu `Never` được khai báo trong các thư viện core và bạn có thể dùng nó
làm type annotation. Có thể bạn có hàm hỗ trợ để dễ
throw một loại exception nhất định:

```dart
// Sử dụng null safety:
Never wrongType(String type, Object value) {
  throw ArgumentError('Expected $type, but was ${value.runtimeType}.');
}
```

Bạn có thể sử dụng như sau:

```dart
// Sử dụng null safety:
class Point {
  final int x, y;

  Point(this.x, this.y);

  Point operator +(Object other) {
    if (other is int) return Point(x + other, y + other);
    if (other is! Point) wrongType('int | Point', other);

    print('Adding two Point instances together: $this + $other');
    return Point(x + other.x, y + other.y);
  }

  // toString, hashCode và các implementation khác...
}
```

Chương trình này được phân tích không có lỗi. Lưu ý dòng cuối của method `+`
truy cập `.x` và `.y` trên `other`. Nó đã được promote thành `Point` dù
hàm không có `return` hoặc `throw`. Control flow analysis
biết kiểu được khai báo của `wrongType()` là `Never`, nghĩa là
nhánh then của câu lệnh `if` *phải* abort bằng cách nào đó. Vì chỉ có thể đến
câu lệnh cuối khi `other` là `Point`, Dart promote nó.

Nói cách khác, sử dụng `Never` trong API của bạn cho phép mở rộng
phân tích reachability của Dart.

<a id="definite-assignment-analysis"></a>
### Phân tích definite assignment

Tôi đã đề cập ngắn gọn điều này với biến local. Dart cần bảo đảm một
biến local non-nullable luôn được khởi tạo trước khi đọc. Chúng tôi dùng
*phân tích definite assignment* để linh hoạt nhất có thể. Ngôn ngữ
phân tích từng thân hàm và theo dõi assignment cho biến local
và parameter qua mọi path control flow. Miễn là biến
được gán trên mọi path đi đến một nơi sử dụng biến, biến được
xem là đã khởi tạo. Điều này cho phép bạn khai báo biến không có initializer rồi
khởi tạo sau đó bằng control flow phức tạp, kể cả khi biến
có kiểu non-nullable.

Chúng tôi cũng dùng phân tích definite assignment để làm biến *final* linh hoạt hơn.
Trước null safety, có thể khó dùng `final` cho biến local
nếu bạn cần khởi tạo chúng theo cách thú vị:

```dart
// Sử dụng null safety:
int tracingFibonacci(int n) {
  final int result;
  if (n < 2) {
    result = n;
  } else {
    result = tracingFibonacci(n - 2) + tracingFibonacci(n - 1);
  }

  print(result);
  return result;
}
```

Trước đây đây là lỗi vì biến `result` là `final` nhưng không có
initializer. Với flow analysis thông minh hơn trong null safety, chương trình này
hợp lệ. Phân tích có thể biết `result` chắc chắn được khởi tạo chính xác một lần
trên mọi path control flow, nên các ràng buộc để đánh dấu biến `final`
được thỏa mãn.

<a id="type-promotion-on-null-checks"></a>
### Type promotion khi kiểm tra null

Flow analysis thông minh hơn giúp ích nhiều code Dart, kể cả code không liên quan đến
nullability. Nhưng không phải ngẫu nhiên chúng tôi thực hiện các thay đổi này lúc này. Chúng tôi
đã phân chia kiểu thành tập nullable và non-nullable. Nếu bạn có một value
thuộc kiểu nullable, bạn không thể thực sự *làm* điều gì hữu ích với nó. Trong trường hợp
value *là* `null`, hạn chế đó là tốt. Nó ngăn bạn
gây crash.

Nhưng nếu value không phải `null`, sẽ tốt nếu có thể chuyển nó sang
phía non-nullable để gọi method. Flow analysis là một trong những
cách chính để làm điều này cho biến local và parameter
(và private final field, kể từ Dart 3.2). Chúng tôi đã mở rộng type
promotion để cũng xem xét biểu thức `== null` và `!= null`.

Nếu bạn kiểm tra một biến local có kiểu nullable để xem nó có khác `null` hay không,
sau đó Dart promote biến thành kiểu non-nullable cơ sở:

```dart
// Sử dụng null safety:
String makeCommand(String executable, [List<String>? arguments]) {
  var result = executable;
  if (arguments != null) {
    result += ' ' + arguments.join(' ');
  }
  return result;
}
```

Ở đây, `arguments` có kiểu nullable. Thông thường, điều đó cấm bạn gọi
`.join()` trên nó. Nhưng vì chúng ta bảo vệ lệnh gọi đó trong câu lệnh `if`
kiểm tra để bảo đảm value không phải `null`, Dart promote nó từ `List<String>?`
thành `List<String>` và cho phép gọi method trên đó hoặc truyền nó vào hàm
mong đợi list non-nullable.

Điều này nghe có vẻ khá nhỏ, nhưng promotion dựa trên flow khi kiểm tra null
là thứ giúp phần lớn code Dart hiện có hoạt động với null safety. Phần lớn code Dart
*đúng* về mặt dynamic và tránh throw lỗi null reference bằng cách
kiểm tra `null` trước khi gọi method. Flow analysis mới về kiểm tra null
chuyển tính đúng đắn *dynamic* đó thành tính đúng đắn *static* có thể chứng minh.

Tất nhiên, nó cũng hoạt động với phân tích thông minh hơn mà chúng tôi thực hiện cho reachability. Hàm
trên cũng có thể được viết như sau:

```dart
// Sử dụng null safety:
String makeCommand(String executable, [List<String>? arguments]) {
  var result = executable;
  if (arguments == null) return result;
  return result + ' ' + arguments.join(' ');
}
```

Ngôn ngữ cũng thông minh hơn về loại biểu thức gây ra promotion. Biểu thức
`== null` hoặc `!= null` rõ ràng tất nhiên hoạt động. Nhưng explicit cast bằng `as`,
assignment hoặc postfix operator `!`
(sẽ được trình bày [sau](#not-null-assertion-operator)) cũng gây ra
promotion. Mục tiêu chung là nếu code đúng về mặt dynamic và có thể
hợp lý xác định điều đó theo cách static, phân tích phải đủ thông minh để làm vậy.

Lưu ý rằng ban đầu type promotion chỉ hoạt động trên biến local,
và giờ cũng hoạt động trên private final field kể từ Dart 3.2.
Để biết thêm về cách làm việc với biến non-local,
hãy xem [Làm việc với field nullable](#working-with-nullable-fields).

<a id="unnecessary-code-warnings"></a>
### Cảnh báo code không cần thiết

Có phân tích reachability thông minh hơn và biết `null` có thể chảy qua
chương trình ở đâu giúp bảo đảm bạn *thêm* code để xử lý `null`. Nhưng chúng ta cũng
có thể dùng cùng phân tích để phát hiện code bạn *không* cần. Trước null safety,
nếu bạn viết như sau:

```dart
// Sử dụng null safety:
String checkList(List<Object> list) {
  if (list?.isEmpty ?? false) {
    return 'Got nothing';
  }
  return 'Got something';
}
```

Dart không có cách biết operator null-aware `?.` đó có hữu ích hay không.
Theo những gì nó biết, bạn có thể truyền `null` vào hàm. Nhưng trong Dart null-safe,
nếu bạn đã annotate hàm bằng kiểu `List` giờ là non-nullable, thì
nó biết `list` không bao giờ là `null`. Điều đó ngụ ý `?.` sẽ không bao giờ
làm điều hữu ích và bạn có thể cũng như nên chỉ dùng `.`.

Để giúp đơn giản hóa code, chúng tôi đã thêm cảnh báo cho code không cần thiết như
vậy vì phân tích static giờ đủ chính xác để phát hiện. Sử dụng
operator null-aware hoặc thậm chí phép kiểm tra như `== null` hay `!= null` trên
kiểu non-nullable sẽ được báo cáo là cảnh báo.

Và tất nhiên, điều này cũng hoạt động với type promotion thành non-nullable. Khi một
biến đã được promote thành kiểu non-nullable, bạn nhận cảnh báo nếu
kiểm tra lại `null` một cách dư thừa:

```dart
// Sử dụng null safety:
String checkList(List<Object>? list) {
  if (list == null) return 'No list';
  if (list?.isEmpty ?? false) {
    return 'Empty list';
  }
  return 'Got something';
}
```

Bạn nhận cảnh báo trên `?.` ở đây vì tại thời điểm nó thực thi, chúng ta
đã biết `list` không thể là `null`. Mục tiêu của các cảnh báo này không chỉ là
dọn dẹp code vô nghĩa. Bằng cách loại bỏ những phép kiểm tra `null` *không cần thiết*, chúng ta bảo đảm
những phép kiểm tra có ý nghĩa còn lại nổi bật. Chúng tôi muốn bạn có thể nhìn
vào code và *thấy* `null` có thể chảy ở đâu.

<a id="working-with-nullable-types"></a>
## Làm việc với kiểu nullable

Giờ chúng ta đã dồn `null` vào tập các kiểu nullable. Với flow analysis,
chúng ta có thể an toàn cho một số value non-`null` vượt rào sang phía non-nullable
nơi có thể sử dụng chúng. Đó là bước tiến lớn, nhưng nếu dừng ở đây,
hệ thống tạo ra vẫn hạn chế đến khó chịu. Flow analysis chỉ giúp với
biến local, parameter và private final field.

Để cố lấy lại càng nhiều càng tốt sự linh hoạt Dart có trước null
safety—và vượt xa hơn ở một số nơi—chúng tôi có một vài
tính năng mới khác.

<a id="smarter-null-aware-methods"></a>
### Các method null-aware thông minh hơn

Operator null-aware `?.` của Dart có từ lâu trước null safety. Ngữ nghĩa runtime
quy định rằng nếu receiver là `null`, việc truy cập property ở
phía phải bị bỏ qua và biểu thức đánh giá thành `null`:

```dart
// Không có null safety:
String notAString = null;
print(notAString?.length);
```

Thay vì throw exception, code này in "null". Operator null-aware là
công cụ hữu ích để khiến kiểu nullable dùng được trong Dart. Dù không thể cho bạn
gọi method trên kiểu nullable, chúng tôi có thể và thực sự cho phép dùng operator null-aware
trên chúng. Phiên bản sau null safety của chương trình là:

```dart
// Sử dụng null safety:
String? notAString = null;
print(notAString?.length);
```

Nó hoạt động giống hệt phiên bản trước.

Tuy nhiên, nếu từng dùng operator null-aware trong Dart, có lẽ bạn đã
gặp sự phiền toái khi dùng chúng trong chuỗi method. Giả sử bạn muốn
xem độ dài của một string có thể vắng mặt có phải số chẵn không:

```dart
// Sử dụng null safety:
String? notAString = null;
print(notAString?.length.isEven);
```

Dù chương trình dùng `?.`, nó vẫn throw exception ở runtime.
Vấn đề là receiver của biểu thức `.isEven` là kết quả của toàn bộ
biểu thức `notAString?.length` bên trái. Biểu thức đó đánh giá thành
`null`, nên ta gặp lỗi null reference khi cố gọi `.isEven`. Nếu từng
dùng `?.` trong Dart, có lẽ bạn đã khó khăn học được rằng phải áp dụng
operator null-aware cho *mọi* property hoặc method trong chuỗi sau lần đầu dùng nó:

```dart
String? notAString = null;
print(notAString?.length?.isEven);
```

Điều này phiền toái và tệ hơn nữa là che khuất thông tin quan trọng. Hãy xem:

```dart
// Sử dụng null safety:
showGizmo(Thing? thing) {
  print(thing?.doohickey?.gizmo);
}
```

Một câu hỏi: Getter `doohickey` trên `Thing` có thể trả về `null` không?
Có vẻ là *có thể* vì bạn dùng `?.` trên kết quả. Nhưng cũng có thể
`?.` thứ hai chỉ ở đó để xử lý trường hợp `thing` là
`null`, không phải kết quả của `doohickey`. Bạn không thể biết.

Để giải quyết, chúng tôi mượn ý tưởng thông minh từ thiết kế cùng tính năng của C#.
Khi dùng operator null-aware trong chuỗi method, nếu receiver đánh giá
thành `null`, thì *toàn bộ phần còn lại của chuỗi method bị short-circuit và
bỏ qua*. Nghĩa là nếu `doohickey` có return type non-nullable, bạn
có thể và nên viết:

```dart
// Sử dụng null safety:
void showGizmo(Thing? thing) {
  print(thing?.doohickey.gizmo);
}
```

Thực tế, bạn nhận cảnh báo code không cần thiết trên `?.` thứ hai nếu không
làm vậy. Nếu thấy code như:

```dart
// Sử dụng null safety:
void showGizmo(Thing? thing) {
  print(thing?.doohickey?.gizmo);
}
```

Thì bạn biết chắc điều đó nghĩa là chính `doohickey` có return type
nullable. Mỗi `?.` tương ứng với một path *riêng biệt* có thể khiến `null` chảy
vào chuỗi method. Điều này làm operator null-aware trong chuỗi method vừa
ngắn gọn vừa chính xác hơn.

Nhân tiện, chúng tôi thêm một vài operator null-aware khác:

```dart
// Sử dụng null safety:

// Cascade null-aware:
receiver?..method();

// Operator index null-aware:
receiver?[index];
```

Không có operator gọi hàm null-aware, nhưng bạn có thể viết:

```dart
// Được phép có hoặc không có null safety:
function?.call(arg1, arg2);
```

<a id="null-assertion-operator" aria-hidden="true"></a>
<a id="non-null-assertion-operator" aria-hidden="true"></a>

<a id="not-null-assertion-operator"></a>
### Operator not-null assertion

Điều tuyệt vời khi dùng flow analysis để chuyển biến nullable sang phía
non-nullable là việc đó an toàn có thể chứng minh. Bạn được gọi
method trên biến trước đây nullable mà không mất đi tính an toàn hay
hiệu năng của kiểu non-nullable.

Nhưng nhiều cách dùng kiểu nullable hợp lệ không thể được *chứng minh* là an toàn theo cách
làm phân tích static hài lòng. Ví dụ:

```dart
// Sử dụng null safety, không đúng:
class HttpResponse {
  final int code;
  final String? error;

  HttpResponse.ok()
      : code = 200,
        error = null;
  HttpResponse.notFound()
      : code = 404,
        error = 'Not found';

  @override
  String toString() {
    if (code == 200) return 'OK';
    return 'ERROR $code ${error.toUpperCase()}';
  }
}
```

Nếu thử chạy, bạn nhận compile error ở lệnh gọi `toUpperCase()`.
Field `error` nullable vì nó không có value trong response thành công.
Qua kiểm tra class, chúng ta thấy không bao giờ truy cập message `error`
khi nó là `null`. Nhưng điều đó đòi hỏi hiểu mối quan hệ
giữa value của `code` và nullability của `error`. Type checker
không thấy được kết nối đó.

Nói cách khác, người bảo trì code *biết* error sẽ không phải
`null` tại nơi sử dụng và cần cách assert điều đó. Thông thường,
bạn assert kiểu bằng cast `as`, và ở đây cũng có thể làm vậy:

```dart
// Sử dụng null safety:
String toString() {
  if (code == 200) return 'OK';
  return 'ERROR $code ${(error as String).toUpperCase()}';
}
```

Cast `error` thành kiểu `String` non-nullable sẽ throw exception runtime
nếu cast thất bại. Nếu không, nó cho ta string non-nullable để gọi method.

"Cast bỏ nullability" xuất hiện đủ thường xuyên để chúng tôi có cú pháp viết tắt mới.
Dấu chấm than postfix (`!`) lấy biểu thức bên trái và cast nó thành
kiểu non-nullable cơ sở. Vì vậy hàm trên tương đương:

```dart
// Sử dụng null safety:
String toString() {
  if (code == 200) return 'OK';
  return 'ERROR $code ${error!.toUpperCase()}';
}
```

"Bang operator" một ký tự này đặc biệt tiện khi kiểu cơ sở dài dòng.
Sẽ rất khó chịu nếu phải viết `as
Map<TransactionProviderFactory, List<Set<ResponseFilter>>>` chỉ để cast bỏ một
dấu `?` khỏi kiểu.

Tất nhiên, như mọi cast, dùng `!` làm mất một phần an toàn static. Cast
phải được kiểm tra ở runtime để giữ soundness và có thể thất bại rồi throw
exception. Nhưng bạn kiểm soát nơi chèn những cast này và luôn có thể
thấy chúng khi xem code.

<a id="late-variables"></a>
### Biến late

Nơi phổ biến nhất mà type checker không thể chứng minh tính an toàn của code là
quanh biến top-level và field. Đây là một ví dụ:

```dart
// Sử dụng null safety, không đúng:
class Coffee {
  String _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature + ' coffee';
}

void main() {
  var coffee = Coffee();
  coffee.heat();
  coffee.serve();
}
```

Ở đây, method `heat()` được gọi trước `serve()`. Điều đó nghĩa là `_temperature`
sẽ được khởi tạo thành value non-null trước khi sử dụng. Nhưng phân tích static
không thể xác định điều đó một cách khả thi. (Có thể làm được với ví dụ đơn giản
như thế này, nhưng trường hợp tổng quát cố theo dõi trạng thái của từng
instance của class là bài toán không thể xử lý được.)

Vì type checker không thể phân tích cách dùng field và biến top-level,
nó có quy tắc thận trọng rằng field non-nullable phải được khởi tạo
tại khai báo (hoặc trong initialization list của constructor đối với
instance field). Vì vậy Dart báo compile error trên class này.

Bạn có thể sửa lỗi bằng cách biến field thành nullable rồi
dùng operator not-null assertion tại các nơi sử dụng:

```dart
// Sử dụng null safety:
class Coffee {
  String? _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature! + ' coffee';
}
```

Code này hoạt động tốt. Nhưng nó gửi tín hiệu gây nhầm lẫn cho người bảo trì class.
Bằng cách đánh dấu `_temperature` nullable, bạn ngụ ý `null` là value hữu ích,
có ý nghĩa cho field đó. Nhưng đó không phải chủ đích. Field `_temperature`
không bao giờ nên được *quan sát* ở trạng thái `null`.

Để xử lý pattern phổ biến về trạng thái được khởi tạo trễ, chúng tôi thêm
modifier mới `late`. Bạn có thể dùng như sau:

```dart
// Sử dụng null safety:
class Coffee {
  late String _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature + ' coffee';
}
```

Lưu ý field `_temperature` có kiểu non-nullable nhưng chưa được khởi tạo.
Ngoài ra không có not-null assertion rõ ràng khi sử dụng nó.
Có một vài mô hình bạn có thể áp dụng cho ngữ nghĩa của `late`,
nhưng tôi nghĩ như sau: Modifier `late` nghĩa là
"thực thi các ràng buộc của biến này ở runtime thay vì compile time".
Gần như từ "late" mô tả *thời điểm* nó thực thi các bảo đảm của biến.

Trong trường hợp này, vì field không được khởi tạo chắc chắn, mỗi lần
đọc field, một kiểm tra runtime được chèn để bảo đảm nó đã được gán
value. Nếu chưa, exception được throw. Việc cho biến kiểu
`String` nghĩa là "bạn không bao giờ nên thấy tôi với value không phải string", còn
modifier `late` nghĩa là "hãy xác minh điều đó ở runtime".

Theo một số khía cạnh, modifier `late` "kỳ diệu" hơn dùng `?` vì bất kỳ
lần sử dụng field nào cũng có thể thất bại và không có gì hiển thị trong văn bản tại
nơi sử dụng. Nhưng bạn *phải* viết `late` tại khai báo để có
hành vi này, và chúng tôi tin rằng nhìn thấy modifier ở đó đủ rõ ràng
để có thể bảo trì.

Đổi lại, bạn có an toàn static tốt hơn dùng kiểu nullable. Vì kiểu
của field giờ non-nullable, việc cố gán `null` hoặc một `String`
nullable cho field là *compile* error. Modifier `late` cho phép bạn *trì hoãn*
khởi tạo nhưng vẫn cấm đối xử với nó như biến nullable.

<a id="lazy-initialization"></a>
### Khởi tạo lazy

Modifier `late` còn có một số năng lực đặc biệt khác. Có vẻ nghịch lý,
nhưng bạn có thể dùng `late` trên field có initializer:

```dart
// Sử dụng null safety:
class Weather {
  late int _temperature = _readThermometer();
}
```

Khi làm vậy, initializer trở thành *lazy*. Thay vì chạy ngay khi
instance được tạo, nó bị trì hoãn và chạy lazy lần đầu field
được truy cập. Nói cách khác, nó hoạt động chính xác như initializer trên
biến top-level hoặc static field. Điều này hữu ích khi biểu thức khởi tạo
tốn kém và có thể không cần đến.

Chạy initializer theo cách lazy mang lại thêm lợi ích khi dùng `late` trên
instance field. Thông thường initializer của instance field không thể truy cập `this`
vì bạn chưa có quyền truy cập object mới cho đến khi mọi initializer của field
hoàn tất. Nhưng với field `late`, điều đó không còn đúng, nên bạn *có thể*
truy cập `this`, gọi method hoặc truy cập field trên instance.

<a id="late-final-variables"></a>
### Biến late final

Bạn cũng có thể kết hợp `late` với `final`:

```dart
// Sử dụng null safety:
class Coffee {
  late final String _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  String serve() => _temperature + ' coffee';
}
```

Không giống field `final` thông thường, bạn không phải khởi tạo field trong
khai báo hoặc initialization list của constructor. Bạn có thể gán nó
sau ở runtime. Nhưng chỉ có thể gán *một lần*, và điều đó được kiểm tra
ở runtime. Nếu cố gán nhiều hơn một lần—như gọi cả
`heat()` và `chill()` ở đây—assignment thứ hai throw exception.
Đây là cách tuyệt vời để mô hình hóa trạng thái cuối cùng được khởi tạo và
sau đó bất biến.

Nói cách khác, modifier `late` mới kết hợp với các modifier biến khác của Dart
bao phủ phần lớn phạm vi tính năng của `lateinit` trong Kotlin và
`lazy` trong Swift. Bạn thậm chí có thể dùng nó trên biến local nếu muốn một chút
đánh giá lazy cục bộ.

<a id="required-named-parameters"></a>
### Named parameter bắt buộc

Để bảo đảm bạn không bao giờ thấy parameter `null` với kiểu non-nullable,
type checker yêu cầu mọi optional parameter có kiểu nullable hoặc
default value. Nếu bạn muốn named parameter có kiểu non-nullable
và không có default value thì sao? Điều đó ngụ ý bạn muốn yêu cầu caller
*luôn luôn* truyền nó. Nói cách khác, bạn muốn parameter *named*
nhưng không optional.

Tôi hình dung các loại parameter Dart bằng bảng này:

```plaintext
             mandatory    optional
            +------------+------------+
positional  | f(int x)   | f([int x]) |
            +------------+------------+
named       | ???        | f({int x}) |
            +------------+------------+
```

Vì lý do không rõ ràng, Dart từ lâu hỗ trợ ba góc của bảng nhưng
để trống tổ hợp named+mandatory. Với null safety, chúng tôi điền vào đó.
Bạn khai báo named parameter bắt buộc bằng cách đặt `required` trước
parameter:

```dart
// Sử dụng null safety:
function({int? a, required int? b, int? c, required int? d}) {}
```

Ở đây, mọi parameter phải được truyền theo tên. Parameter `a` và `c`
optional và có thể bỏ qua. Parameter `b` và `d` là bắt buộc và phải
được truyền. Lưu ý tính bắt buộc độc lập với nullability. Bạn có thể có
named parameter bắt buộc thuộc kiểu nullable và named parameter optional thuộc
kiểu non-nullable (nếu chúng có default value).

Đây là một tính năng khác mà tôi cho rằng giúp Dart tốt hơn bất kể
null safety. Nó đơn giản khiến ngôn ngữ có cảm giác hoàn thiện hơn.

<a id="abstract-fields"></a>
### Abstract field

Một trong những tính năng hay của Dart là
nó tuân thủ thứ gọi là [nguyên tắc truy cập đồng nhất][].
Theo cách nói thông thường, điều đó nghĩa là
field không thể phân biệt với getter và setter.
Việc một "property" trong class Dart nào đó được tính toán hay lưu trữ
là chi tiết implementation.
Vì vậy, khi định nghĩa interface bằng abstract class,
việc dùng khai báo field là phổ biến:

[uniform access principle]: https://en.wikipedia.org/wiki/Uniform_access_principle

```dart
abstract class Cup {
  Beverage contents;
}
```

Chủ đích là người dùng chỉ implement class đó chứ không extend nó.
Cú pháp field chỉ là cách viết ngắn hơn cho một cặp getter/setter:

```dart
abstract class Cup {
  Beverage get contents;
  set contents(Beverage);
}
```

Nhưng Dart không *biết* class này sẽ không bao giờ được dùng làm kiểu concrete.
Nó xem khai báo `contents` là field thật.
Và đáng tiếc, field đó non-nullable và không có initializer,
nên bạn nhận compile error.

Một cách sửa là dùng khai báo abstract getter/setter rõ ràng
như ví dụ thứ hai.
Nhưng cách đó hơi dài dòng,
vì vậy với null safety,
chúng tôi cũng thêm hỗ trợ khai báo abstract field rõ ràng:

```dart
abstract class Cup {
  abstract Beverage contents;
}
```

Nó hoạt động chính xác như ví dụ thứ hai.
Nó chỉ khai báo abstract getter và setter với tên và kiểu cho trước.

<a id="working-with-nullable-fields"></a>
### Làm việc với field nullable

Những tính năng mới này bao phủ nhiều pattern phổ biến và khiến việc làm việc với `null`
khá nhẹ nhàng trong phần lớn trường hợp. Nhưng dù vậy, kinh nghiệm của chúng tôi là field
nullable vẫn có thể khó xử lý. Trong trường hợp có thể biến field thành `late` và
non-nullable thì rất tốt. Nhưng trong nhiều trường hợp bạn cần *kiểm tra* xem
field có value hay không, và điều đó đòi hỏi biến nó thành nullable để quan sát
`null`.

Field nullable vừa private vừa final có thể type promote
(trừ [một số lý do cụ thể](/tools/non-promotion-reasons)).
Nếu vì lý do nào đó không thể làm field private và final,
bạn vẫn cần giải pháp thay thế.

Ví dụ, bạn có thể mong đợi code này hoạt động:

```dart
// Sử dụng null safety, không đúng:
class Coffee {
  String? _temperature;

  void heat() { _temperature = 'hot'; }
  void chill() { _temperature = 'iced'; }

  void checkTemp() {
    if (_temperature != null) {
      print('Ready to serve ' + _temperature + '!');
    }
  }

  String serve() => _temperature! + ' coffee';
}
```

Bên trong `checkTemp()`, chúng ta kiểm tra xem `_temperature` có là `null` không. Nếu không,
chúng ta truy cập nó và cuối cùng gọi `+` trên đó. Đáng tiếc, điều này không được phép.

Type promotion dựa trên flow chỉ có thể áp dụng cho field *vừa private vừa final*.
Nếu không, phân tích static không thể *chứng minh* value của field không
thay đổi giữa thời điểm kiểm tra `null` và thời điểm sử dụng.
(Hãy cân nhắc trường hợp bất thường khi chính field có thể bị override bằng
getter trong subclass trả về `null` ở lần gọi thứ hai.)

Vì chúng tôi quan tâm đến soundness, field public và/hoặc non-final không được promote,
và method trên không compile. Điều này khá phiền.
Trong trường hợp đơn giản như ở đây, cách tốt nhất là đặt `!` tại nơi dùng field.
Có vẻ dư thừa, nhưng đó ít nhiều là cách Dart hoạt động hiện nay.

Một pattern hữu ích khác là sao chép field vào biến local trước rồi
dùng biến đó:

```dart
// Sử dụng null safety:
void checkTemp() {
  var temperature = _temperature;
  if (temperature != null) {
    print('Ready to serve ' + temperature + '!');
  }
}
```

Vì type promotion áp dụng cho biến local, giờ code hoạt động tốt. Nếu cần
*thay đổi* value, hãy nhớ lưu lại vào field chứ không chỉ biến local.

Để biết thêm thông tin về cách xử lý những vấn đề type promotion này và khác,
hãy xem [Sửa lỗi type promotion](/tools/non-promotion-reasons).

<a id="nullability-and-generics"></a>
### Nullability và generic

Giống phần lớn ngôn ngữ static-typed hiện đại, Dart có generic class và
generic method. Chúng tương tác với nullability theo một số cách có vẻ
trái trực giác nhưng hợp lý khi bạn suy nghĩ kỹ hệ quả. Đầu tiên,
"kiểu này có nullable không?" không còn là câu hỏi yes/no đơn giản.
Hãy xem:

```dart
// Sử dụng null safety:
class Box<T> {
  final T object;
  Box(this.object);
}

void main() {
  Box<String>('a string');
  Box<int?>(null);
}
```

Trong định nghĩa `Box`, `T` là kiểu nullable hay non-nullable? Như
bạn thấy, nó có thể được instantiate bằng cả hai loại. Câu trả lời là `T` là
*kiểu có khả năng nullable*. Bên trong thân generic class hoặc method,
kiểu có khả năng nullable chịu mọi hạn chế của cả kiểu nullable
*và* non-nullable.

Điều trước nghĩa là bạn không thể gọi method nào trên nó ngoại trừ số ít method được định nghĩa trên
Object. Điều sau nghĩa là phải khởi tạo mọi field hoặc biến thuộc
kiểu đó trước khi dùng. Điều này có thể khiến type parameter khá khó xử lý.

Trong thực tế, một vài pattern xuất hiện. Trong class giống collection nơi type
parameter có thể được instantiate bằng bất kỳ kiểu nào, bạn chỉ cần xử lý
các hạn chế. Trong phần lớn trường hợp như ví dụ này, điều đó nghĩa là bảo đảm bạn
có quyền truy cập một value thuộc kiểu của type argument mỗi khi cần làm việc
với nó. May thay, class giống collection hiếm khi gọi method trên element.

Ở những nơi không có quyền truy cập value, bạn có thể làm nơi dùng
type parameter thành nullable:

```dart
// Sử dụng null safety:
class Box<T> {
  T? object;
  Box.empty();
  Box.full(this.object);
}
```

Lưu ý dấu `?` trên khai báo `object`. Giờ field có kiểu nullable rõ ràng,
nên có thể để nó chưa khởi tạo.

Khi làm kiểu type parameter nullable như `T?` ở đây, bạn có thể cần
cast bỏ nullability. Cách đúng là dùng explicit cast `as
T`, *không phải* operator `!`:

```dart
// Sử dụng null safety:
class Box<T> {
  T? object;
  Box.empty();
  Box.full(this.object);

  T unbox() => object as T;
}
```

Operator `!` *luôn* throw nếu value là `null`. Nhưng nếu type
parameter được instantiate bằng kiểu nullable, `null` là value hoàn toàn
hợp lệ cho `T`:

```dart
// Sử dụng null safety:
void main() {
  var box = Box<int?>.full(null);
  print(box.unbox());
}
```

Chương trình này phải chạy không lỗi. Dùng `as T` đạt được điều đó. Dùng
`!` sẽ throw exception.

Các generic type khác có bound hạn chế loại type argument
có thể áp dụng:

```dart
// Sử dụng null safety:
class Interval<T extends num> {
  T min, max;

  Interval(this.min, this.max);

  bool get isEmpty => max <= min;
}
```

Nếu bound non-nullable thì type parameter cũng non-nullable. Điều này
nghĩa là bạn chịu hạn chế của kiểu non-nullable—không thể để field
và biến chưa khởi tạo. Class ví dụ ở đây phải có constructor
khởi tạo các field.

Đổi lại hạn chế đó, bạn có thể gọi bất kỳ method nào trên value thuộc kiểu
type parameter được khai báo trên bound. Tuy nhiên, có bound non-nullable
ngăn *người dùng* generic class instantiate nó bằng type argument
nullable. Đó có lẽ là giới hạn hợp lý với phần lớn class.

Bạn cũng có thể dùng *bound* nullable:

```dart
// Sử dụng null safety:
class Interval<T extends num?> {
  T min, max;

  Interval(this.min, this.max);

  bool get isEmpty {
    var localMin = min;
    var localMax = max;

    // Không có min hoặc max nghĩa là interval không có điểm kết thúc.
    if (localMin == null || localMax == null) return false;
    return localMax <= localMin;
  }
}
```

Điều này nghĩa là trong thân class, bạn có sự linh hoạt để đối xử với
type parameter như nullable, nhưng cũng chịu các giới hạn của nullability.
Bạn không thể gọi bất kỳ thứ gì trên biến thuộc kiểu đó
trừ khi xử lý nullability trước. Trong ví dụ này,
chúng ta sao chép field vào biến local và kiểm tra các biến local đó với `null`
để flow analysis promote chúng thành kiểu non-nullable trước khi dùng `<=`.

Lưu ý bound nullable không ngăn người dùng instantiate class bằng kiểu
non-nullable. Bound nullable nghĩa là type argument *có thể* nullable,
không phải *bắt buộc*. (Thực tế, bound mặc định trên type parameter nếu
bạn không viết mệnh đề `extends` là bound nullable `Object?`.) Không có
cách *yêu cầu* type argument nullable. Nếu muốn nơi dùng type
parameter chắc chắn nullable và được ngầm khởi tạo thành `null`,
bạn có thể dùng `T?` bên trong thân class.

<a id="core-library-changes"></a>
## Thay đổi thư viện core

Có một vài điều chỉnh khác đây đó trong ngôn ngữ, nhưng chúng không đáng kể.
Chẳng hạn kiểu mặc định của `catch` không có mệnh đề `on` giờ là
`Object` thay cho `dynamic`. Phân tích fallthrough trong câu lệnh switch dùng
flow analysis mới.

Những thay đổi còn lại thực sự quan trọng với bạn nằm trong các thư viện core.
Trước khi bắt đầu Cuộc phiêu lưu Null Safety Vĩ đại, chúng tôi lo rằng
không có cách làm các thư viện core null-safe mà không phá vỡ nghiêm trọng
mọi thứ. Kết quả không tệ đến vậy. *Có* một vài thay đổi đáng kể,
nhưng phần lớn quá trình migration diễn ra suôn sẻ. Hầu hết thư viện core
không chấp nhận `null` và chuyển tự nhiên sang kiểu non-nullable, hoặc có chấp nhận và
tiếp nhận nó nhẹ nhàng bằng kiểu nullable.

Tuy nhiên có một vài góc quan trọng:

<a id="the-map-index-operator-is-nullable"></a>
### Operator index của Map là nullable

Đây không thực sự là thay đổi mà là điều cần biết. Operator index `[]` trên
class Map trả về `null` nếu key không tồn tại. Điều này ngụ ý
return type của operator đó phải nullable: `V?` thay cho `V`.

Chúng tôi có thể thay đổi method đó để throw exception khi key không
tồn tại rồi cho nó return type non-nullable dễ dùng hơn. Nhưng code
dùng operator index và kiểm tra `null` để xem key có vắng mặt
rất phổ biến—khoảng một nửa mọi trường hợp sử dụng theo phân tích của chúng tôi. Phá vỡ toàn bộ
code đó sẽ khiến hệ sinh thái Dart bùng cháy.

Thay vào đó, hành vi runtime giữ nguyên và vì thế return type buộc
phải nullable. Điều này nghĩa là nói chung bạn không thể dùng ngay kết quả
tra cứu map:

```dart
// Sử dụng null safety, không đúng:
var map = {'key': 'value'};
print(map['key'].length); // Lỗi.
```

Bạn nhận compile error khi cố gọi `.length` trên string nullable.
Trong trường hợp bạn *biết* key tồn tại, có thể chỉ cho type checker bằng `!`:

```dart
// Sử dụng null safety:
var map = {'key': 'value'};
print(map['key']!.length); // Hợp lệ.
```

Chúng tôi đã cân nhắc thêm method khác vào Map để làm việc này: tra cứu
key, throw nếu không tìm thấy, hoặc trả về value non-nullable. Nhưng gọi nó là gì?
Không tên nào ngắn hơn `!` một ký tự, và không tên method nào rõ ràng hơn
việc thấy `!` với ngữ nghĩa tích hợp ngay tại nơi gọi. Vì vậy, cách đúng idiom
để truy cập element chắc chắn có trong map là dùng `[]!`. Bạn sẽ quen với nó.

<a id="no-unnamed-list-constructor"></a>
### Không có unnamed constructor của List

Unnamed constructor trên `List` tạo list mới với kích thước cho trước nhưng
không khởi tạo bất kỳ element nào. Điều này sẽ tạo một lỗ hổng rất lớn trong
bảo đảm soundness nếu bạn tạo list thuộc kiểu non-nullable rồi truy cập element.

Để tránh điều đó, chúng tôi đã loại bỏ hoàn toàn constructor. Gọi
`List()` trong code null-safe là lỗi, ngay cả với kiểu nullable. Nghe có vẻ đáng sợ,
nhưng trong thực tế phần lớn code tạo list bằng list literal, `List.filled()`,
`List.generate()` hoặc là kết quả biến đổi collection khác. Với
trường hợp hiếm khi muốn tạo list rỗng thuộc kiểu nào đó, chúng tôi thêm
constructor `List.empty()` mới.

Pattern tạo list hoàn toàn chưa khởi tạo luôn có cảm giác không phù hợp
trong Dart, và giờ càng như vậy. Nếu có code bị hỏng bởi điều này,
bạn luôn có thể sửa bằng một trong nhiều cách khác để tạo list.

<a id="cannot-set-a-larger-length-on-non-nullable-lists"></a>
### Không thể đặt length lớn hơn trên list non-nullable

Điều này ít được biết đến, nhưng getter `length` trên `List` cũng có
*setter* tương ứng. Bạn có thể đặt length thành value ngắn hơn để cắt ngắn list.
Và cũng có thể đặt thành length *dài hơn* để đệm list bằng các element
chưa khởi tạo.

Nếu làm vậy với list thuộc kiểu non-nullable, bạn sẽ vi phạm
soundness khi sau đó truy cập các element chưa được ghi. Để ngăn điều đó,
setter `length` sẽ throw exception runtime nếu (và chỉ nếu) list có
element type non-nullable *và* bạn đặt length *dài hơn*. Việc cắt ngắn
list thuộc mọi kiểu vẫn hợp lệ, và bạn có thể tăng list thuộc kiểu nullable.

Điều này có hệ quả quan trọng nếu bạn định nghĩa kiểu list riêng extend
`ListBase` hoặc áp dụng `ListMixin`. Cả hai kiểu cung cấp một
implementation của `insert()` mà trước đây tạo chỗ cho element được chèn bằng cách
đặt length. Điều đó sẽ thất bại với null safety, nên thay vào đó chúng tôi thay đổi
implementation `insert()` trong `ListMixin` (được `ListBase` chia sẻ) để
gọi `add()`. Class list tùy chỉnh của bạn nên cung cấp định nghĩa `add()`
nếu muốn dùng method `insert()` được kế thừa đó.

<a id="cannot-access-iteratorcurrent-before-or-after-iteration"></a>
### Không thể truy cập Iterator.current trước hoặc sau phép lặp

Class `Iterator` là class "con trỏ" mutable dùng để duyệt element
của kiểu implement `Iterable`. Bạn được kỳ vọng gọi `moveNext()`
trước khi truy cập bất kỳ element nào để tiến tới element đầu tiên. Khi method đó
trả về `false`, bạn đã đến cuối và không còn element nào.

Trước đây `current` trả về `null` nếu gọi nó trước lần gọi
`moveNext()` đầu tiên hoặc sau khi phép lặp hoàn tất. Với null
safety, điều đó sẽ yêu cầu return type của `current` là `E?` chứ không phải `E`.
Đến lượt nó, điều đó nghĩa là mỗi lần truy cập element sẽ cần kiểm tra `null` runtime.

Những kiểm tra đó vô ích vì hầu như không ai truy cập element hiện tại
theo cách sai như vậy. Thay vào đó, chúng tôi làm kiểu của `current` là
`E`. Vì *có thể* không có value thuộc kiểu đó trước hoặc sau khi
lặp, chúng tôi để hành vi của iterator là undefined nếu bạn gọi nó khi
không được phép. Phần lớn implementation của `Iterator` throw `StateError`.

<a id="summary"></a>
## Tóm tắt

Đó là chuyến tham quan rất chi tiết qua toàn bộ thay đổi ngôn ngữ và thư viện
quanh null safety. Có rất nhiều nội dung, nhưng đây là thay đổi ngôn ngữ khá lớn.
Quan trọng hơn, chúng tôi muốn đạt đến nơi Dart vẫn tạo cảm giác gắn kết và
dễ dùng. Điều đó đòi hỏi thay đổi không chỉ hệ thống kiểu mà còn một số
tính năng hỗ trợ sử dụng xung quanh. Chúng tôi không muốn null safety có cảm giác
được gắn thêm một cách khiên cưỡng.

Những điểm cốt lõi cần ghi nhớ là:

*   Kiểu non-nullable theo mặc định và được làm nullable bằng cách thêm `?`.

*   Optional parameter phải nullable hoặc có default value. Bạn có thể dùng
    `required` để làm named parameter không optional. Biến top-level
    non-nullable và static field phải có initializer. Instance field
    non-nullable phải được khởi tạo trước khi thân constructor bắt đầu.

*   Chuỗi method sau operator null-aware short-circuit nếu receiver là
    `null`. Có các operator cascade (`?..`) và index (`?[]`) null-aware mới.
    Operator not-null assertion "bang" postfix (`!`) cast operand
    nullable thành kiểu non-nullable cơ sở.

*   Flow analysis cho phép bạn an toàn biến biến local và parameter nullable
    (và private final field, kể từ Dart 3.2)
    thành dạng non-nullable có thể sử dụng. Flow analysis mới cũng có quy tắc thông minh hơn
    cho type promotion, thiếu return, code không thể đến và khởi tạo biến.

*   Modifier `late` cho phép bạn dùng kiểu non-nullable và `final` ở những nơi
    nếu không thì không thể, đổi lại phải kiểm tra runtime. Nó cũng
    cung cấp field được khởi tạo lazy.

*   Class `List` được thay đổi để ngăn element chưa khởi tạo.

Cuối cùng, khi đã tiếp thu tất cả điều đó và đưa code vào thế giới null
safety, bạn có chương trình sound mà compiler có thể tối ưu, và mọi
nơi có thể xảy ra error runtime đều hiển thị trong code. Chúng tôi hy vọng bạn
cảm thấy điều đó xứng đáng với công sức bỏ ra.
