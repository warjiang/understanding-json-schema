.. index::
   single: type
   single: types; basic

.. _type:

Type-specific keywords
======================

``type`` 字段是 JSON Schema 的基石, 描述了 schema 的数据类型.

JSON Schema 定义了以下几种基本类型:

   - `string`
   - `numeric`
   - `object`
   - `array`
   - `boolean`
   - `null`

尽管JSON Schema 和 具体编程语言中 的类型名称不一定相同, 但表达的含义都是类似的.

.. language_specific::

    --Python
    下表中列出了JavaScript中的类型到Python中类型的映射关系:

    +----------+-----------+
    |JavaScript|Python     |
    +----------+-----------+
    |string    |string     |
    |          |[#1]_      |
    +----------+-----------+
    |number    |int/float  |
    |          |[#2]_      |
    +----------+-----------+
    |object    |dict       |
    +----------+-----------+
    |array     |list       |
    +----------+-----------+
    |boolean   |bool       |
    +----------+-----------+
    |null      |None       |
    +----------+-----------+

    .. rubric:: Footnotes

    .. [#1] 由于 JavaScript 中string一直都是unicode, 与 Python 2.x 中的 ``unicode`` 类型
            或者 Python 3.x 中的 ``str`` 类似.

    .. [#2] JavaScript 中不区分 整数和浮点数.


    --Ruby
    下表中列出了JavaScript中的类型到Ruby中类型的映射关系:

    +----------+----------------------+
    |JavaScript|Ruby                  |
    +----------+----------------------+
    |string    |String                |
    +----------+----------------------+
    |number    |Integer/Float         |
    |          |[#3]_                 |
    +----------+----------------------+
    |object    |Hash                  |
    +----------+----------------------+
    |array     |Array                 |
    +----------+----------------------+
    |boolean   |TrueClass/FalseClass  |
    +----------+----------------------+
    |null      |NilClass              |
    +----------+----------------------+

    .. rubric:: Footnotes

    .. [#3] JavaScript 中不区分 整数和浮点数.

``type`` 字段既可以是 string 也可以是 array:

- 如果 ``type`` 是string, 则表示上面基础类型名之一.

- 如果 ``type`` 是array, 则必须为string数组, 数组中每个string都必须要是基础类型名中的一个, 
  同时需要确保数组中的string是唯一的. 这种情况下, 只要给定的JSON数据符合 *任意* 一个类型的要求
  则表示该JSON是有效的.

如下是使用 ``type`` 字段的简单案例:

.. schema_example::

   { "type": "number" }
   --
   42
   --
   42.0
   --X
   // 非数字, 而是数字字符串.
   "42"


下面的案例中, 可以接受string或者number类型的数据,但不支持结构化的数据类型:

.. schema_example::

   { "type": ["number", "string"] }
   --
   42
   --
   "Life, the universe, and everything"
   --X
   ["Life", "the universe", "and everything"]

每种类型都会仅适用于该类型的关键字. 比如 numeric 类型可以某种方式来描述数据的范围, 但这种方式不适用于其他数据类型.
后面的章节中将对这些类型及其校验关键字进行介绍.
