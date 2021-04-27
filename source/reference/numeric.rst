.. index::
   single: integer
   single: number
   single: types; numeric

.. _numeric:

Numeric types
-------------

.. contents:: :local:

JSON Schema包括两种数值类型:  `integer` 和 `number`, 两种数值类型共享相同的校验字段.

.. note::

    JSON 无法表示复数, 因此无法在JSON Schema中对其进行校验.

.. _integer:


integer
'''''''

``integer`` 类型用来表示整数.

.. language_specific::

    --Python
    Python中 "integer" 类似于 ``int``.
    --Ruby
    Ruby中 "integer" 类似于 ``Integer``.

.. schema_example::

    { "type": "integer" }
    --
    42
    --
    -1
    --X
    // 浮点数无法通过校验:
    3.1415926
    --X
    // 整数字符串也无法通过校验:
    "42"

.. warning::

    "integer" 类型的精度处理取决于 JSON Schema 验证的实现. JavaScript(包括JSON)不区分整数和浮点数的, 
    因此没法单独通过 type 来区分 整数和非整数. JSON Schema 规范建议(但不要求) 验证程序实现的时候使用数学值来确定是否为整数,
    而不是通过类型来判断. 在这一点上, 各个验证程序实现之间有一些分歧. 比如 javascript 的验证程序认为 ``1.0`` 是整数, 但是 Python
    验证程序 `jsonschema <https://pypi.python.org/pypi/jsonschema>`__ 不认为 ``1.0`` 是整数.

可以巧妙的使用 ``multipleOf`` 字段(参考 `multiples` )来解决不同验证器之间的差异. 比如,以下案例可能在所有的JSON Schema实现上具有相同的行为:

.. schema_example::

    { "type": "number", "multipleOf": 1.0 }
    --
    42
    --
    42.0
    --X
    3.14156926


.. index::
   single: number

.. _number:

number
''''''

``number`` 类型用于任意数值类型, 包括整数和浮点.

.. language_specific::

    --Python
    Python中 "number" 类似于 ``float`` 类型.
    --Ruby
    Ruby中 "number" 类似于 ``Float`` 类型.

.. schema_example::

    { "type": "number" }
    --
    42
    --
    -1
    --
    // 简单浮点数:
    5.0
    --
    // 指数表示法也可以通过校验:
    2.99792458e8
    --X
    // 数字字符串无法通过校验:
    "42"

.. index::
   single: multipleOf
   single: number; multiple of

.. _multiples:

Multiples
'''''''''

Number 类型可以用 ``multipleOf`` 约束为给定数字的倍数.  ``multipleOf`` 可以被设为任意的正整数.

.. schema_example::

    {
        "type"       : "number",
        "multipleOf" : 10
    }
    --
    0
    --
    10
    --
    20
    --X
    // 非10的倍数:
    23

.. index::
   single: number; range
   single: maximum
   single: exclusiveMaximum
   single: minimum
   single: exclusiveMinimum

Range
'''''

number类型的范围通过 ``minimum`` 和 ``maximum`` (或者 ``exclusiveMinimum`` 和
``exclusiveMaximum`` 表示不包含的范围)来表示.

记待校验的值为 *x* , 只有符合以下关系才能通过校验:

  - *x* ≥ ``minimum``
  - *x* > ``exclusiveMinimum``
  - *x* ≤ ``maximum``
  - *x* < ``exclusiveMaximum``

可以同时指定 ``minimum`` 和 ``exclusiveMinimum`` 或者 ``maximum`` 和 ``exclusiveMaximum``,
虽然这样做并没有什么实质的效果.

.. schema_example::

    {
      "type": "number",
      "minimum": 0,
      "exclusiveMaximum": 100
    }
    --X
    // 小于 ``minimum``:
    -1
    --
    // 包含 ``minimum`` 对应的值, 因此 0 可以通过校验:
    0
    --
    10
    --
    99
    --X
    // 不包含 ``exclusiveMaximum`` 对应的值, 因此 100 无法通过校验:
    100
    --X
    // 超过 ``maximum``:
    101

.. language_specific::

    --Draft 4
    JSON Schema Draft 4规范中,  ``exclusiveMinimum`` 和 ``exclusiveMaximum`` 与 Draft 7不太一样.
    Draft 4 规范中这两个字段都是 boolean 类型, 用来标记 ``minimum`` 和 ``maximum`` 是包含还是不包含. 
    比如:

    - 如果 ``exclusiveMinimum`` 设为 ``false``, 则 *x* ≥ ``minimum``.
    - 如果 ``exclusiveMinimum`` 设为 ``true``, 则  *x* > ``minimum``.

    Draft 7 进行了调整使得字段间具有更好的独立性.

    下面是一个采用 Draft 4规范的案例:

    .. schema_example:: 4

        {
          "type": "number",
          "minimum": 0,
          "maximum": 100,
          "exclusiveMaximum": true
        }
        --X
        // 小于 ``minimum``:
        -1
        --
        // ``exclusiveMinimum`` 没有设为 ``true``, 因此包含0:
        0
        --
        10
        --
        99
        --X
        // ``exclusiveMaximum`` 被设为 ``true``, 因此不包含100:
        100
        --X
        // 超过 ``maximum``:
        101
