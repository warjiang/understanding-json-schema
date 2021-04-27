.. index::
   single: null

.. _null:

null
----

一般情况 null 类型用来表示缺省值. 如果 schema 指定 ``type`` 为 ``null``, 此时只有 ``null`` 能通过校验.

.. language_specific::

   --Python
   Python 中 ``null`` 类似于 ``None``.
   --Ruby
   Ruby 中 ``null`` 类似于 ``nil``.

.. schema_example::

    { "type": "null" }
    --
    null
    --X
    false
    --X
    0
    --X
    ""
