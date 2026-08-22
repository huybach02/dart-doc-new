---
title: Quản lý phiên bản ngôn ngữ
description: >-
  Tìm hiểu cách quản lý phiên bản ngôn ngữ trong Dart hỗ trợ sự phát triển của ngôn ngữ
  và cách cấu hình phiên bản ngôn ngữ cho project cùng các library của bạn.
---

<a id="overview"></a>
## Tổng quan

Một Dart SDK có thể đồng thời hỗ trợ
nhiều phiên bản của ngôn ngữ Dart.
Compiler xác định code đang nhắm đến phiên bản nào
và diễn giải code theo phiên bản đó.

Quản lý phiên bản ngôn ngữ trở nên quan trọng trong những dịp hiếm hoi khi Dart
đưa vào một tính năng không tương thích như [null safety][].
Khi Dart đưa vào breaking change, code từng
compile được có thể không còn compile.
Quản lý phiên bản ngôn ngữ cho phép bạn đặt phiên bản ngôn ngữ cho từng library
để duy trì tính tương thích.

Trong trường hợp null safety, Dart SDK từ 2.12 đến 2.19 cho phép bạn
_lựa chọn_ cập nhật code để dùng null safety.
Dart dùng quản lý phiên bản ngôn ngữ để cho phép code không null-safe chạy
cùng với code null-safe.
Quyết định này hỗ trợ migration từ code không null-safe sang code null-safe.
Để xem lại ví dụ về cách ứng dụng hoặc package có thể migrate sang một
phiên bản ngôn ngữ mới có tính năng không tương thích, hãy xem bài viết
[Migration null safety dần dần cho project Dart lớn][Gradual null safety migration for large Dart projects].

Mỗi package có phiên bản ngôn ngữ mặc định bằng **cận dưới**
của [SDK constraint][] trong file `pubspec.yaml`.

**Ví dụ:** Entry sau trong file `pubspec.yaml`
cho biết package này mặc định dùng phiên bản ngôn ngữ Dart 2.18.

```yaml
environment:
  sdk: '>=2.18.0 <3.0.0'
```

Để tìm hiểu cách thức và lý do team Dart phát triển phương pháp quản lý phiên bản này,
hãy xem [đặc tả quản lý phiên bản ngôn ngữ][language versioning specification].

[null safety]: /null-safety
[Gradual null safety migration for large Dart projects]: /blog/gradual-null-safety-migration-for-large-dart-projects
[SDK constraint]: /tools/pub/pubspec#sdk-constraints
[language versioning specification]: {{site.repo.dart.lang}}/blob/main/accepted/2.8/language-versioning/feature-specification.md#dart-language-versioning

<a id="language-version-numbers"></a>
## Số phiên bản ngôn ngữ

Dart định dạng phiên bản ngôn ngữ dưới dạng hai số phân tách bằng dấu chấm.
Chúng được đọc là số phiên bản major và số phiên bản minor.
Số phiên bản minor có thể đưa vào breaking change.

Bản phát hành Dart có thể thêm số patch vào phiên bản ngôn ngữ.
Patch không nên thay đổi ngôn ngữ ngoại trừ bug fix.
Để minh họa: Dart 2.18.3 là bản phát hành mới nhất của
phiên bản ngôn ngữ Dart 2.18 SDK.

Mỗi bản phát hành Dart SDK hỗ trợ phiên bản ngôn ngữ của chính nó và
mọi phiên bản trước trong cùng số phiên bản major.
Điều đó có nghĩa Dart SDK 2.18.3 hỗ trợ phiên bản ngôn ngữ
từ 2.0 đến hết 2.18, nhưng không hỗ trợ Dart 1.x.

Việc suy ra phiên bản ngôn ngữ từ phiên bản SDK kéo theo những điều sau:

* Bất cứ khi nào một minor version của SDK được phát hành, một phiên bản ngôn ngữ mới xuất hiện.
  Trong thực tế, nhiều phiên bản ngôn ngữ này hoạt động rất giống
  phiên bản trước và hoàn toàn tương thích với nhau.
  Ví dụ: Ngôn ngữ Dart 2.9 hoạt động gần giống ngôn ngữ Dart 2.8.

* Khi bản patch của SDK được phát hành,
  nó không thể đưa vào tính năng ngôn ngữ mới.
  Ví dụ: Bản phát hành 2.18.3 _vẫn là_ phiên bản ngôn ngữ 2.18.
  Nó phải duy trì tương thích với 2.18.2, 2.18.1 và 2.18.0.

## Chọn phiên bản ngôn ngữ theo library {: #library-override}

Theo mặc định, mọi file Dart trong package dùng cùng phiên bản ngôn ngữ.
Dart xác định phiên bản ngôn ngữ mặc định là
cận dưới của SDK constraint được chỉ định trong file `pubspec.yaml`.
Đôi khi một file Dart có thể cần dùng phiên bản ngôn ngữ cũ hơn.
Ví dụ, bạn có thể không migrate được tất cả file trong package
sang null safety cùng lúc.

Dart hỗ trợ chọn phiên bản ngôn ngữ theo library.
Để chủ động dùng một phiên bản ngôn ngữ khác với
phần còn lại của package, một [Dart library][] phải
chứa comment theo định dạng sau:

```dart
// @dart = <major>.<minor>
```

Ví dụ:

```dart
// Mô tả nội dung trong file này.
// @dart = 2.17
import 'dart:math';

// ...
```

String `@dart` phải nằm trong comment `//` (không phải `///` hoặc `/*`),
và phải xuất hiện trước bất kỳ code Dart nào trong file.
Khoảng trắng (tab và space) không quan trọng,
ngoại trừ bên trong string `@dart` và version.
Như ví dụ trước cho thấy,
comment khác có thể xuất hiện trước comment `@dart`.

[Dart library]: /tools/pub/create-packages#organizing-a-package

<a id="language-versioned-changes"></a>
## Thay đổi được quản lý theo phiên bản ngôn ngữ

Quản lý phiên bản ngôn ngữ giúp Dart và tooling phát triển theo thời gian.
Bên cạnh những thay đổi cú pháp và ngữ nghĩa ngôn ngữ có khả năng gây breaking change,
tính năng mới và thay đổi tooling cũng có thể bị giới hạn theo phiên bản ngôn ngữ.
Ví dụ, `dart format` có thể format code Dart khác nhau tùy vào
phiên bản ngôn ngữ của library mà nó đang format.

Để tìm những thay đổi mà mỗi phiên bản ngôn ngữ đưa vào,
bạn có thể tham khảo [Dart changelog][] có thể lọc.
Để tìm hiểu chi tiết cụ thể về từng phiên bản ngôn ngữ và tính năng,
hãy xem [tài liệu ngôn ngữ][language documentation] và [đặc tả ngôn ngữ][language specification].

:::tip
Để xem lại các tính năng và thay đổi Dart mới đang được
thảo luận, thiết kế và implement,
hãy xem project [Dart language funnel][] trên GitHub.
:::

[Dart changelog]: /changelog
[language documentation]: /language
[language specification]: /resources/language/spec
[Dart language funnel]: https://github.com/orgs/dart-lang/projects/90
