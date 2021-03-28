.. index::
   single: combining schemas

.. _combining:

Combining schemas
=================

.. contents:: :local:

JSON Schema中包含了一些用于组合schema的关键字. 虽然 `structuring` 章节介绍了用于从多个文件或者
JSON树中组合schema的方法, 但是组合schema并不是一个必选项.  组合schema就像增加多个验证条件一样简单.

举个例子, 下面的schema中 ``anyOf`` 关键字表示给定的数据需要匹配给定子schema中任意一条即可. 第一个子schema限制string的长度不超过5. 第二个子schema限制number的最小值为0. 只要给定的数据能够符合其中 *任意* 一条子schema即表示匹配通过组合后的schema.

.. schema_example::

    {
      "anyOf": [
        { "type": "string", "maxLength": 5 },
        { "type": "number", "minimum": 0 }
      ]
    }
    --
    "short"
    --X
    "too long"
    --
    12
    --X
    -5

用于组合shcema的关键字有::

- `allOf`: 匹配 *所有* 子schema
- `anyOf`: 匹配 *任意一个* 子schema
- `oneOf`: 匹配 *有且仅有一个* 子schema

所有用于组合的关键字都必须是是数组类型, 数据内部的元素表示每个独立的子schema.

除此之外还有:

- `not`: 所有的schema都 *不能* 匹配

.. index::
   single: allOf
   single: combining schemas; allOf

.. _allOf:

allOf
-----

给定的数据需要通过所有的子schema校验才能通过 ``allOf`` 校验.

.. schema_example::

    {
      "allOf": [
        { "type": "string" },
        { "maxLength": 5 }
      ]
    }
    --
    "short"
    --X
    "too long"

使用 ``allOf`` 创建的schema很容易出现逻辑上的冲突问题. 下面的例子中就演示了一个永远不可能校验通过的schema
(因为不能存在同时既是string类型又是number类型的变量):

.. schema_example::

    {
      "allOf": [
        { "type": "string" },
        { "type": "number" }
      ]
    }
    --X
    "No way"
    --X
    -1

需要注意的是 `allOf`, `anyOf` 或者 `oneOf` 数组中对应的子schema之间无任何感知.
虽然 `allOf` 可以往schema中增加字段扩展schema, 但是 `allOf` 不能用于表示对象之间的继承关系. 举个例子, 
假定在 ``definitions`` 内有一个address的schema, 现在想扩展address往里面增加一个type字段:

.. schema_example::

   {
     "definitions": {
       "address": {
         "type": "object",
         "properties": {
           "street_address": { "type": "string" },
           "city":           { "type": "string" },
           "state":          { "type": "string" }
         },
         "required": ["street_address", "city", "state"]
       }
     },

     "allOf": [
       { "$ref": "#/definitions/address" },
       { "properties": {
           "type": { "enum": [ "residential", "business" ] }
         }
       }
     ]
   }
   --
   {
      "street_address": "1600 Pennsylvania Avenue NW",
      "city": "Washington",
      "state": "DC",
      "type": "business"
   }

扩展的schema可以正常工作, 但是如果我们想限制不能有额外的字段应该怎么实现呢?
一种思路是加上 ``additionalProperties`` 字段:

.. schema_example::

   {
     "definitions": {
       "address": {
         "type": "object",
         "properties": {
           "street_address": { "type": "string" },
           "city":           { "type": "string" },
           "state":          { "type": "string" }
         },
         "required": ["street_address", "city", "state"]
       }
     },

     "allOf": [
       { "$ref": "#/definitions/address" },
       { "properties": {
           "type": { "enum": [ "residential", "business" ] }
         }
       }
     ],

     *"additionalProperties": false
   }
   --X
   {
      "street_address": "1600 Pennsylvania Avenue NW",
      "city": "Washington",
      "state": "DC",
      "type": "business"
   }

不幸的是, 扩展后的schema无法匹配 *任何输入* .因为 `additionalProperties` 是面向的整个扩展后的schema的. 
但是扩展后的schema不包含任何属性且对 `allOf` 内的schema无感知.

组合模式与面向对象语言的继承表现的不太一直可能是最大的缺点之一了. 在下个版本的JSON schema的规范中也有新的提案用来解决这个问题.

.. index::
   single: anyOf
   single: combining schemas; anyOf

.. _anyOf:

anyOf
-----

``anyOf`` 表示给定的数据能够匹配给定规则中的一条或多条规则。

.. schema_example::

   {
     "anyOf": [
       { "type": "string" },
       { "type": "number" }
     ]
   }
   --
   "Yes"
   --
   42
   --X
   { "Not a": "string or number" }

.. index::
   single: oneOf
   single: combining schemas; oneOf

.. _oneOf:

oneOf
-----

``oneOf`` 表示当且仅当给定的数据能够匹配给定规则中的一条规则：

.. schema_example::

    {
      "oneOf": [
        { "type": "number", "multipleOf": 5 },
        { "type": "number", "multipleOf": 3 }
      ]
    }
    --
    10
    --
    9
    --X
    // 5 或 3 的倍数两条规则均不匹配.
    2
    --X
    // 5 和 3 的倍数两条规则均匹配.
    15

注意可以把子schema中公共的部分提取出来. 下面的schema和上面的schema等价:

.. schema_example::

   {
      "type": "number",
      "oneOf": [
        { "multipleOf": 5 },
        { "multipleOf": 3 }
      ]
    }

.. index::
   single: not
   single: combining schemas; not

.. _not:


not
---

严格来说 ``not`` 不属于是schema组合的模式, 但是 ``not`` 能够和本章的其他的schema组合使用, 
一定程度上调整其他schema的效果. ``not`` 表示所有的给定的子schema都不匹配的模式.

下面的例子表示匹配所有非string的数据:

.. schema_example::

    { "not": { "type": "string" } }
    --
    42
    --
    { "key": "value" }
    --X
    "I am a string"
