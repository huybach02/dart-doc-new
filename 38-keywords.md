---
title: Keyword
description: Các keyword trong Dart.
showToc: false
---

{% assign ckw = '&nbsp;<sup>1</sup>' %}
{% assign bii = '&nbsp;<sup>2</sup>' %}
{% assign unr = '&nbsp;<sup>3</sup>' %}

Bảng sau liệt kê những từ
mà ngôn ngữ Dart dành riêng cho mục đích sử dụng của chính nó.
Những từ này không thể được dùng làm identifier trừ khi có ghi chú khác.
Ngay cả khi được phép, việc dùng keyword làm identifier có thể gây nhầm lẫn cho
developer khác đọc code của bạn và nên tránh.
Để tìm hiểu thêm về cách dùng identifier, hãy nhấp vào thuật ngữ.

<table class="table table-striped">

{% tablerow keyword in keywords cols: 4 %}
<a href="{{keyword.link}}">{{keyword.term}}</a>
{%- case keyword.type %}
{% when 'bit' %}{{bii}}
{% when 'context' %}{{ckw}}
{% when 'unrestricted' %}{{unr}}
{% endcase %}
{% endtablerow %}
</table>

{{ckw}} Keyword này có thể được dùng làm identifier
        tùy theo **context**.

{{bii}} Keyword này không thể được dùng làm tên của một kiểu
        (class, mixin, enum, extension type hoặc type alias),
        tên của extension hoặc làm import prefix.
        Nó có thể được dùng làm identifier trong mọi trường hợp khác.

{{unr}} Keyword này có thể được dùng làm identifier mà không bị hạn chế.
