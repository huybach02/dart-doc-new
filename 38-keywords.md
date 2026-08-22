---
title: Keywords
description: Các keyword trong Dart.
showToc: false
---

Bảng sau liệt kê những từ
mà ngôn ngữ Dart dành riêng cho mục đích sử dụng của chính nó.
Những từ này không thể được dùng làm identifier trừ khi có ghi chú khác.
Ngay cả khi được phép, việc dùng keyword làm identifier có thể gây nhầm lẫn cho
developer khác đọc code của bạn và nên tránh.
Để tìm hiểu thêm về cách dùng identifier, hãy nhấp vào thuật ngữ.

| | | | |
|---|---|---|---|
| [`abstract`](/language/class-modifiers#abstract)<sup>2</sup> | [`as`](/language/operators#type-test-operators)<sup>2</sup> | [`assert`](/language/error-handling#assert) | [`async`](/language/async)<sup>3</sup> |
| [`await`](/language/async)<sup>1</sup> | [`base`](/language/class-modifiers#base)<sup>3</sup> | [`break`](/language/loops#break-and-continue) | [`case`](/language/branches#switch) |
| [`catch`](/language/error-handling#catch) | [`class`](/language/classes#instance-variables) | [`const`](/language/variables#final-and-const) | [`continue`](/language/loops#break-and-continue) |
| [`covariant`](/language/type-system#covariant-keyword)<sup>2</sup> | [`default`](/language/branches#switch) | [`deferred`](/language/libraries#lazily-loading-a-library)<sup>2</sup> | [`do`](/language/loops#while-and-do-while) |
| [`dynamic`](/language#important-concepts)<sup>2</sup> | [`else`](/language/branches#if) | [`enum`](/language/enums) | [`export`](/tools/pub/create-packages)<sup>2</sup> |
| [`extends`](/language/extend) | [`extension`](/language/extension-methods)<sup>2</sup> | [`external`](/language/functions#external)<sup>2</sup> | [`factory`](/language/constructors#factory-constructors)<sup>2</sup> |
| [`false`](/language/built-in-types#booleans) | [**final (var)**](/language/variables#final-and-const) | [**final (class)**](/language/class-modifiers#final) | [`finally`](/language/error-handling#finally) |
| [`for`](/language/loops#for-loops) | [`Function`](/language/functions)<sup>2</sup> | [`get`](/language/methods#getters-and-setters)<sup>2</sup> | [`hide`](/language/libraries#importing-only-part-of-a-library)<sup>3</sup> |
| [`if`](/language/branches#if) | [`implements`](/language/classes#implicit-interfaces)<sup>2</sup> | [`import`](/language/libraries#using-libraries)<sup>2</sup> | [`in`](/language/loops#for-loops) |
| [`interface`](/language/class-modifiers#interface)<sup>2</sup> | [`is`](/language/operators#type-test-operators) | [`late`](/language/variables#late-variables)<sup>2</sup> | [`library`](/language/libraries)<sup>2</sup> |
| [`mixin`](/language/mixins)<sup>2</sup> | [`new`](/language/classes#using-constructors) | [`null`](/language/variables#default-value) | [`of`](/tools/pub/create-packages#organizing-a-package)<sup>3</sup> |
| [`on`](/language/error-handling#catch)<sup>3</sup> | [`operator`](/language/methods#operators)<sup>2</sup> | [`part`](/tools/pub/create-packages#organizing-a-package)<sup>2</sup> | [`required`](/language/functions#named-parameters)<sup>2</sup> |
| [`rethrow`](/language/error-handling#catch) | [`return`](/language/functions#return-values) | [`sealed`](/language/class-modifiers#sealed)<sup>3</sup> | [`set`](/language/methods#getters-and-setters)<sup>2</sup> |
| [`show`](/language/libraries#importing-only-part-of-a-library)<sup>3</sup> | [`static`](/language/classes#class-variables-and-methods)<sup>2</sup> | [`super`](/language/extend) | [`switch`](/language/branches#switch) |
| [`sync`](/language/functions#generators)<sup>3</sup> | [`this`](/language/constructors) | [`throw`](/language/error-handling#throw) | [`true`](/language/built-in-types#booleans) |
| [`try`](/language/error-handling#catch) | [`type`](/language/extension-types)<sup>2</sup> | [`typedef`](/language/typedefs)<sup>2</sup> | [`var`](/language/variables) |
| [`void`](/language/built-in-types) | [`when`](/language/branches#when)<sup>3</sup> | [`with`](/language/mixins) | [`while`](/language/loops#while-and-do-while) |
| [`yield`](/language/functions#generators)<sup>1</sup> | | | |

<sup>1</sup> Keyword này có thể được dùng làm identifier
        tùy theo **context**.

<sup>2</sup> Keyword này không thể được dùng làm tên của một kiểu
        (class, mixin, enum, extension type hoặc type alias),
        tên của extension hoặc làm import prefix.
        Nó có thể được dùng làm identifier trong mọi trường hợp khác.

<sup>3</sup> Keyword này có thể được dùng làm identifier mà không bị hạn chế.
