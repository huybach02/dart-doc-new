---
title: Tham chiếu class modifier
description: >-
  Các tổ hợp class modifier được phép và không được phép.
prevpage:
  url: /language/class-modifiers
  title: Class modifier
nextpage:
  url: /language/concurrency
  title: Concurrency trong Dart
---

Trang này chứa thông tin tham chiếu về
[class modifier](/language/class-modifiers).

<a id="valid-combinations"></a>
## Các tổ hợp hợp lệ

Các tổ hợp class modifier hợp lệ và khả năng tương ứng của chúng là:

| Khai báo                     | [Khởi tạo][Construct]? | [Mở rộng][Extend]? | [Implement][]? | [Mix in][]? | [Exhaustive][]? |
|-----------------------------|----------------|-------------|----------------|-------------|-----------------|
| `class`                     | **Có**         | **Có**      | **Có**         | Không       | Không           |
| `base class`                | **Có**         | **Có**      | Không          | Không       | Không           |
| `interface class`           | **Có**         | Không       | **Có**         | Không       | Không           |
| `final class`               | **Có**         | Không       | Không          | Không       | Không           |
| `sealed class`              | Không          | Không       | Không          | Không       | **Có**          |
| `abstract class`            | Không          | **Có**      | **Có**         | Không       | Không           |
| `abstract base class`       | Không          | **Có**      | Không          | Không       | Không           |
| `abstract interface class`  | Không          | Không       | **Có**         | Không       | Không           |
| `abstract final class`      | Không          | Không       | Không          | Không       | Không           |
| `mixin class`               | **Có**         | **Có**      | **Có**         | **Có**      | Không           |
| `base mixin class`          | **Có**         | **Có**      | Không          | **Có**      | Không           |
| `abstract mixin class`      | Không          | **Có**      | **Có**         | **Có**      | Không           |
| `abstract base mixin class` | Không          | **Có**      | Không          | **Có**      | Không           |
| `mixin`                     | Không          | Không       | **Có**         | **Có**      | Không           |
| `base mixin`                | Không          | Không       | Không          | **Có**      | Không           |

{: .table .table-striped .nowrap}

[Construct]: /language/classes#using-constructors
[Extend]: /language/extend
[Implement]: /language/classes#implicit-interfaces
[Mix in]: /language/mixins
[Exhaustive]: /language/branches#exhaustiveness-checking

<a id="invalid-combinations"></a>
## Các tổ hợp không hợp lệ

Một số [tổ hợp][combinations] modifier không được phép:

| Tổ hợp                                        | Lý do                                                                                                                                                                                |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `base`, `interface` và `final`                | Tất cả đều kiểm soát cùng hai khả năng (`extend` và `implement`), vì vậy loại trừ lẫn nhau.                                                                                           |
| `sealed` và `abstract`                        | Cả hai đều không thể được khởi tạo, vì vậy dùng cùng nhau là dư thừa.                                                                                                                 |
| `sealed` với `base`, `interface` hoặc `final` | Kiểu `sealed` vốn đã không thể được mix in, mở rộng hoặc implement từ library khác, vì vậy việc kết hợp với các modifier được liệt kê là dư thừa.                                     |
| `mixin` và `abstract`                         | Cả hai đều không thể được khởi tạo, vì vậy dùng cùng nhau là dư thừa.                                                                                                                 |
| `mixin` và `interface`, `final` hoặc `sealed` | Khai báo `mixin` hoặc `mixin class` được thiết kế để được mix in, trong khi các modifier được liệt kê ngăn cản điều đó.                                                               |
| `enum` và bất kỳ modifier nào                 | Khai báo `enum` không thể được mở rộng, implement hoặc mix in và luôn có thể được khởi tạo, vì vậy không modifier nào áp dụng cho khai báo `enum`.                                    |
| `extension type` và bất kỳ modifier nào       | Khai báo `extension type` không thể được mở rộng hoặc mix in và chỉ có thể được implement bởi khai báo `extension type` khác.                                                         |

{: .table .table-striped .nowrap}

[combinations]: /language/class-modifiers#combining-modifiers
