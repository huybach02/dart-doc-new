---
title: Effective Dart
description: >-
  Các best practice để xây dựng thư viện Dart nhất quán,
  dễ bảo trì và hiệu quả.
nextpage:
  url: /effective-dart/style
  title: Style
---

Trong vài năm qua, chúng tôi đã viết rất nhiều code Dart và học được nhiều điều
về những gì hoạt động tốt và không tốt. Chúng tôi chia sẻ điều này để bạn cũng có thể
viết code nhất quán, mạnh mẽ và nhanh. Có hai chủ đề bao quát:

 1. **Hãy nhất quán.** Với những điều như format và cách viết hoa,
    tranh luận cách nào tốt hơn mang tính chủ quan và không thể giải quyết dứt điểm.
    Điều chúng ta biết là tính *nhất quán* hữu ích một cách khách quan.

    Nếu hai đoạn code trông khác nhau, nguyên nhân phải là chúng *thực sự*
    khác nhau theo một cách có ý nghĩa. Khi một đoạn code nổi bật và thu hút
    ánh mắt, nó phải làm vậy vì lý do hữu ích.

 2. **Hãy ngắn gọn.** Dart được thiết kế để quen thuộc nên kế thừa nhiều
    câu lệnh và biểu thức giống C, Java, JavaScript cùng các ngôn ngữ khác.
    Nhưng chúng tôi tạo Dart vì vẫn còn nhiều không gian để cải thiện những gì
    các ngôn ngữ đó cung cấp. Chúng tôi thêm nhiều tính năng, từ nội suy string đến
    initializing formal, để giúp bạn biểu đạt chủ đích đơn giản và
    dễ dàng hơn.

    Nếu có nhiều cách diễn đạt điều gì đó, nói chung bạn nên chọn cách
    súc tích nhất. Điều này không có nghĩa bạn nên [code golf][] đến mức
    nhồi cả chương trình vào một dòng. Mục tiêu là code *tiết kiệm*,
    không phải *dày đặc*.

[code golf]: https://en.wikipedia.org/wiki/Code_golf

<a id="the-guides"></a>
## Các hướng dẫn

Chúng tôi chia các guideline thành một vài trang riêng để dễ tiếp thu:

  * **[Hướng dẫn Style][Style Guide]** &ndash; Trang này định nghĩa quy tắc bố trí và
    tổ chức code, hoặc ít nhất những phần
    mà [`dart format`][] không xử lý cho bạn.
    Hướng dẫn style cũng chỉ định cách format identifier:
    `camelCase`, `using_underscores`, v.v.

  * **[Hướng dẫn Documentation][Documentation Guide]** &ndash; Trang này cho bạn biết mọi điều cần
    biết về nội dung bên trong comment, cả doc comment lẫn
    code comment thông thường hàng ngày.

  * **[Hướng dẫn Usage][Usage Guide]** &ndash; Trang này dạy bạn cách tận dụng tốt nhất
    tính năng ngôn ngữ để implement hành vi. Nếu nội dung nằm trong câu lệnh hoặc
    biểu thức, nó được đề cập ở đây.

  * **[Hướng dẫn Design][Design Guide]** &ndash; Đây là hướng dẫn mềm dẻo nhất nhưng có
    phạm vi rộng nhất. Nó trình bày những gì chúng tôi học được về việc thiết kế
    API thư viện nhất quán, dễ dùng. Nếu nội dung nằm trong type signature hoặc
    khai báo, trang này sẽ đề cập.

Để xem liên kết đến mọi guideline, hãy xem
[bản tóm tắt](#summary-of-all-rules).

[`dart format`]: /tools/dart-format
[style guide]: /effective-dart/style
[documentation guide]: /effective-dart/documentation
[usage guide]: /effective-dart/usage
[design guide]: /effective-dart/design

<a id="how-to-read-the-guides"></a>
## Cách đọc các hướng dẫn

Mỗi hướng dẫn được chia thành một vài phần. Các phần chứa danh sách guideline.
Mỗi guideline bắt đầu bằng một trong những từ sau:

* Guideline **DO** mô tả những practice luôn phải tuân theo. Hầu như
  không bao giờ có lý do hợp lệ để đi lệch khỏi chúng.

* Guideline **DON'T** là điều ngược lại: những việc hầu như không bao giờ là ý tưởng
  hay. Hy vọng chúng tôi không có nhiều guideline loại này như các ngôn ngữ khác vì
  có ít gánh nặng lịch sử hơn.

* Guideline **PREFER** là những practice bạn *nên* tuân theo. Tuy nhiên, có thể
  có hoàn cảnh mà làm khác đi là hợp lý. Chỉ cần bảo đảm bạn
  hiểu đầy đủ hệ quả của việc bỏ qua guideline khi làm vậy.

* Guideline **AVOID** là đối ngẫu của "prefer": những việc bạn không nên làm nhưng
  có thể có lý do chính đáng trong những dịp hiếm hoi.

* Guideline **CONSIDER** là những practice bạn có thể muốn hoặc không muốn
  tuân theo, tùy hoàn cảnh, tiền lệ và sở thích riêng.

Một số guideline mô tả một **ngoại lệ** nơi quy tắc *không* áp dụng. Khi
được liệt kê, các ngoại lệ có thể không đầy đủ—bạn vẫn có thể cần dùng
phán đoán của mình trong trường hợp khác.

Nghe như cảnh sát sẽ phá cửa nếu bạn không buộc dây giày đúng cách.
Mọi việc không tệ đến vậy. Phần lớn guideline ở đây là lẽ thường và tất cả chúng ta
đều là người biết lý lẽ. Mục tiêu, như luôn luôn, là code đẹp, dễ đọc và
dễ bảo trì.

Dart analyzer cung cấp linter
để giúp bạn viết code tốt, nhất quán,
tuân theo những guideline này và khác.
Nếu có một hoặc nhiều [linter rule][lints]
có thể giúp bạn tuân theo guideline,
guideline sẽ liên kết đến các rule đó.
Các liên kết dùng định dạng sau:

{% render 'linter-rule-mention.md', rules:'unnecessary_getters_setters' %}

Để tìm hiểu cách dùng linter,
hãy xem [Bật linter rule][]
và danh sách [linter rule][lints].

[Enabling linter rules]: /tools/analysis#enabling-linter-rules
[lints]: /tools/linter-rules

<a id="glossary"></a>
## Bảng thuật ngữ

Để giữ guideline ngắn gọn, chúng tôi dùng một vài thuật ngữ viết tắt để chỉ các
cấu trúc Dart khác nhau.

* **Library member** là top-level field, getter, setter hoặc function.
  Về cơ bản là bất kỳ thứ gì ở top level không phải type.

* **Class member** là constructor, field, getter, setter, function hoặc
  operator được khai báo bên trong class. Class member có thể là instance hoặc static,
  abstract hoặc concrete.

* **Member** là library member hoặc class member.

* **Variable**, khi được dùng theo nghĩa chung, chỉ biến top-level,
  parameter và biến local. Nó không bao gồm static hoặc instance field.

* **Type** là bất kỳ khai báo kiểu có tên nào: class, typedef hoặc enum.

* **Property** là biến top-level, getter (bên trong class hoặc ở top level,
  instance hoặc static), setter (tương tự) hoặc field (instance hoặc static).
  Nói gần đúng là mọi cấu trúc có tên "giống field".

<a id="summary-of-all-rules"></a>
## Tóm tắt tất cả quy tắc

{% render 'effective-dart-toc.md' %}
