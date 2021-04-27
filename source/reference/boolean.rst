.. index::
   single: boolean

.. _boolean:

boolean
-------

boolean 类型只有两个特殊的值:  ``true`` and ``false``. 注意那些等价于 ``true`` or ``false``
的值(比如1、0)无法通过schema校验.

.. language_specific::

   --Python
   "boolean" 类似与Python中 ``bool``. 需要注意的是, JSON中的 ``true`` 和 ``false`` 是小写的, 但是 Python是
   首字母大写的 (``True`` and ``False``).
   --Ruby
   "boolean" 类似与Ruby中 ``TrueClass`` and ``FalseClass``. 需要注意的是Ruby中没有 ``Boolean`` 这个类.

.. schema_example::

    { "type": "boolean" }
    --
    true
    --
    false
    --X
    "true"
    --X
    // 数值上等价于 ``true`` or ``false`` 的无法通过校验:
    0
