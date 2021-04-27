.. index::
   single: string

.. _string:

string
------

.. contents:: :local:

``string`` 用来表示文本字符串, 可能会包含 unicode 字符.

.. language_specific::

   --Python
   Python中 "string" 类似于 python2.x 中的 ``unicode`` 类型, python3.x 中的 ``str`` 类型.
   --Ruby
   Ruby中, "string" 类似于 ``String`` 类型.

.. schema_example::

    { "type": "string" }
    --
    "This is a string"
    --
    // unicode 字符:
    "Déjà vu"
    --
    ""
    --
    "42"
    --X
    42

.. index::
   single: string; length
   single: maxLength
   single: minLength

Length
''''''

string的长度可以用 ``minLength`` 和 ``maxLength`` 字段进行约束, 这两个字段都必须是非负整数.

.. schema_example::

    {
      "type": "string",
      "minLength": 2,
      "maxLength": 3
    }
    --X
    "A"
    --
    "AB"
    --
    "ABC"
    --X
    "ABCD"

.. index::
   single: string; regular expression
   single: pattern

Regular Expressions
'''''''''''''''''''

.. _pattern:

``pattern`` 字段用于约束字符串符合特定的正则表达式. 正则语法参考JavaScript中的正则语法(`ECMA 262
<http://www.ecma-international.org/publications/standards/Ecma-262.htm>`__
定义). 更多细节参考 `regular-expressions` 章节.

.. note::
    只要正则表达式匹配字符串的子串即可认为该字符串通过校验. 比如, 正则表达式 ``"p"`` 可以匹配任何包含 ``p`` 的字符串, 比如 ``"apple"`` , 但是不能匹配单独的字符串 ``p`` . 这样看起似乎有些令人困惑, 但考虑到可以将正则中包含在 ``^...$`` (比如 ``"^p$"``), 这样也是可以接受的.

下面的例子演示了一个带可选区号的北美号码匹配的案例:

.. schema_example::

   {
      "type": "string",
      "pattern": "^(\\([0-9]{3}\\))?[0-9]{3}-[0-9]{4}$"
   }
   --
   "555-1212"
   --
   "(888)555-1212"
   --X
   "(888)555-1212 ext. 532"
   --X
   "(800)FLOWERS"

.. index::
    single: string; format
    single: format

.. _format:

Format
''''''

``format`` 字段允许对某些内容进行基本的语义化验证. 相较于 JSON Schema 中的其他能力(包括 `regular-expressions` ), format 提供了更高级的约束能力.

.. note::

    具体的JSON Schema实现不要求实现format这部分, 许多约束已经停止使用了.

JSON Schema 规范中关于与网络相关的format更多些, 这可能与JSON Schema脱胎于web技术有关.
当然也可以使用自定义format, 只要JSON文档交互双方都包含自定义format定义即可. 
JSON Schema验证程序会忽略不能识别的format规则.

.. index::
   single: format

Built-in formats
^^^^^^^^^^^^^^^^

以下是JSON Schema规范中列出的 formats.

.. index::
   single: date-time
   single: time
   single: date
   single: format; date-time
   single: format; time
   single: format; date

Dates and times
***************

日期和时间规范可以参考 `RFC 3339, section 5.6
<https://json-schema.org/latest/json-schema-validation.html#RFC3339>`_. 
也是 `ISO8601 format
<https://www.iso.org/iso-8601-date-and-time-format.html>`_ 的子集.

- ``"date-time"``: 同时包含日期、时间, 比如, ``2018-11-13T20:20:39+00:00``.

- ``"time"``: |draft7| 时间, 比如, ``20:20:39+00:00``

- ``"date"``: |draft7| 日期, 比如, ``2018-11-13``.

.. index::
   single: email
   single: idn-email
   single: format; email
   single: format; idn-email

Email addresses
***************

- ``"email"``: email地址, 具体参考 `RFC 5322,
  section 3.4.1 <http://tools.ietf.org/html/rfc5322#section-3.4.1>`_.

- ``"idn-email"``: |draft7| 国际化的email地址， 具体参考 `RFC 6531 <https://tools.ietf.org/html/rfc6531>`_.

.. index::
   single: hostname
   single: idn-hostname
   single: format; hostname
   single: format; idn-hostname

Hostnames
*********

- ``"hostname"``: 主机名, 参考 `RFC 1034, section 3.1
  <http://tools.ietf.org/html/rfc1034#section-3.1>`_.

- ``"idn-hostname"``: |draft7| 国际化的主机名, 具体参考
  `RFC5890, section 2.3.2.3
  <https://tools.ietf.org/html/rfc5890#section-2.3.2.3>`_.

.. index::
   single: ipv4
   single: ipv6
   single: format; ipv4
   single: format; ipv6

IP Addresses
************

- ``"ipv4"``: IPv4 地址, 符合 `RFC 2673, section 3.2
  <http://tools.ietf.org/html/rfc2673#section-3.2>`_ 定义的点分四进制ABNF语法.

- ``"ipv6"``: IPv6 地址, 符合 `RFC 2373, section 2.2
  <http://tools.ietf.org/html/rfc2373#section-2.2>`_ 定义.

.. index::
   single: uri
   single: uri-reference
   single: iri
   single: iri-reference
   single: format; uri
   single: format; uri-reference
   single: format; iri
   single: format; iri-reference

Resource identifiers
********************

- ``"uri"``: `RFC3986 <http://tools.ietf.org/html/rfc3986>`_ 定义的统一资源定位符 (URI).

- ``"uri-reference"``: |draft6| `RFC3986, section 4.1
  <http://tools.ietf.org/html/rfc3986#section-4.1>`_ 定义的URI引用(URI或者相对引用).

- ``"iri"``: |draft7| 国际化的 "uri" `RFC3987 <https://tools.ietf.org/html/rfc3987>`_.

- ``"iri-reference"``: |draft7| 国际化的 "uri-reference" `RFC3987 <https://tools.ietf.org/html/rfc3987>`_

如果 schema 中包含相对路径资源(比如网页链接), 一般情况下最好使用 ``"uri-reference"`` (或者 ``"iri-reference"``) 而不是 ``"uri"`` (或者 ``"iri"`` ). 只有绝对路径时才应该使用 ``"uri"`` .

.. draft_specific::

   --Draft 4
   Draft 4中只包含了 ``"uri"``, 不包含 ``"uri-reference"`` . 因此关于 "uri" 是否应该接受相对路径还存在歧义.

.. index::
   single: uri-template
   single: format; uri-template

URI template
************

- ``"uri-template"``: |draft6| `RFC6570 <https://tools.ietf.org/html/rfc6570>`_ 中定义的 URI 模版(任意级别). 如果还不知道URI模版是什么, 大概率也不需要这种类型.

.. index::
   single: json-pointer
   single: relative-json-pointer
   single: format; json-pointer
   single: format; relative-json-pointer

JSON Pointer
************

- ``"json-pointer"``: |draft6| `RFC6901
  <https://tools.ietf.org/html/rfc6901>`_ 定义的JSON Pointer. 对于构造复杂JSON Schema是否应该是用JSON Pointer还有很多讨论. 注意仅当整个字符串中只包含JSON Pointer(比如 ``/foo/bar`` )才能使用JSON Pointer. JSON Pointer的URI片段比如( ``#/foo/bar/`` )应当使用 ``"uri-reference"``.

- ``"relative-json-pointer"``: |draft7| `relative JSON pointer
  <https://tools.ietf.org/html/draft-handrews-relative-json-pointer-01>`_.

.. index::
   single: regex
   single: format; regex

Regular Expressions
*******************

- ``"regex"``: |draft7| 正则表达式, 需要符合 `ECMA 262
  <http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf>`_
  规范.

实践中JSON Schema校验程序只需要实现文档中涉及的 `regular-expressions` 子集即可.

.. TODO: Add some examples for ``format`` here
