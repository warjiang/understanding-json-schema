.. index::
   single: structure

.. _structuring:

Structuring a complex schema
============================

.. contents:: :local:

一般开发复杂的程序时, 我们会将程序 "结构化" 成可复用的功能模块, 
而不是到处复制、粘贴重复的代码. JSON Schema也一样, 除了原子化的schema之外, 
我们也会把 schema 拆成可复用的部分. 本章将提供一些实际案例, 
通过JSON Schema提供能力来结构化、复用schema.

Reuse
-----

下面的例子中, 假设我们要定义一个客户记录, 每个客户都可能有收货地址和帐单邮寄地址. 
地址的结构是相同的：都包含街道地址, 城市和州名. 因此我们不想在所有需要存储地址的地方复制一份地址的schema.
这样不仅会使schema变得冗长, 而且会让后续的结构变更变得更困难. 如果我们假想的公司将来要开展国际业务, 
想在所有的地址中添加一个国家/地区字段, 最好在一个地方进行调整而不是在所有使用地方的地方调整.

从定义address的schema开始: ::

    {
      "type": "object",
      "properties": {
        "street_address": { "type": "string" },
        "city":           { "type": "string" },
        "state":          { "type": "string" }
      },
      "required": ["street_address", "city", "state"]
    }

由于我们要复用这个schema, 习惯上(非必须)会将其放在父schema下的 ``definitions`` 字段内::

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
      }
    }

.. index::
    single: $ref

然后我们就可以通过 ``$ref`` 关键字引用该 schema 片段. ``$ref`` 逻辑上等价于其所指向的内容. 因此想要引用上面的schema, 我们可以这样写::

    { "$ref": "#/definitions/address" }

这种写法可以在任何需要 schema 的地方使用. 如果schema对象中写了 ``$ref`` , 校验的时候会忽略 schema 中的其他字段.

``$ref`` 的值是 URI-reference 格式的, ``#`` 后面的部分("fragment" 或者 "named anchor") 采用了 `JSON Pointer
<https://tools.ietf.org/html/rfc6901>`__ 的格式.

.. note::
    JSON Pointer的作用类似于XML中的 `XPath <http://www.w3.org/TR/xpath/>`_ , 但是JSON Pointer比XPath要简单的多.

如果复用的definition在同一个 JSON Schema 文档中, 则 ``$ref`` 的值以井号(#)开头. 井号后面是用斜杠分割的key, 根据这些key遍历 JSON Schema 文档即可得到复用的 schema 片段的定义. 在我们的案例中 ``"#/definitions/address"`` 含义如下:

1) 访问JSON Schema的root节点
2) 找到 ``"definitions"`` 对应的值
3) 在definitions对象内部, 找到 ``"address"`` 对应的值

``$ref`` 也可以解析为引用另一个文件的URI, 因此如果希望把definitions放在独立的文件中, 也可以这么做::

    { "$ref": "definitions.json#/address" }

这种情况下会从与当前schema文件并列的文件中加载address对应的schema.

现在可以把所有的schema放在一起并基于address schema创建customer schema:

.. schema_example::

    {
      "$schema": "http://json-schema.org/draft-07/schema#",

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

      "type": "object",

      "properties": {
        "billing_address": { "$ref": "#/definitions/address" },
        "shipping_address": { "$ref": "#/definitions/address" }
      }
    }
    --
    {
      "shipping_address": {
        "street_address": "1600 Pennsylvania Avenue NW",
        "city": "Washington",
        "state": "DC"
      },
      "billing_address": {
        "street_address": "1st Street SE",
        "city": "Washington",
        "state": "DC"
      }
    }

.. note::

    即使 ``$ref`` 对应的值是 URI-reference, 但它也只是个标识符, 而不一定是网络定位符. 
    也就是说通过URI解析schema不是一个必选项. 是否可以通过URI解析schema取决于验证程序
    如何实现对外部资源的处理, 开发者不应假定验证程序一定会实现通过网络解析  ``$ref`` 对应的schema.

Recursion
`````````

可以用 ``$ref`` 创建指向schema本身的递归schema. 比如 ``person`` 这个schema包含一组 ``children``, 
同时每个 ``children`` 又都是 ``person`` 实例.

.. schema_example::

    {
      "$schema": "http://json-schema.org/draft-07/schema#",

      "definitions": {
        "person": {
          "type": "object",
          "properties": {
            "name": { "type": "string" },
            "children": {
              "type": "array",
    *          "items": { "$ref": "#/definitions/person" },
              "default": []
            }
          }
        }
      },

      "type": "object",

      "properties": {
        "person": { "$ref": "#/definitions/person" }
      }
    }
    --
    // 英国王室家族树片段如下
    {
      "person": {
        "name": "Elizabeth",
        "children": [
          {
            "name": "Charles",
            "children": [
              {
                "name": "William",
                "children": [
                  { "name": "George" },
                  { "name": "Charlotte" }
                ]
              },
              {
                "name": "Harry"
              }
            ]
          }
        ]
      }
    }

上面的例子中, 我们创建了一个指向自身的的schema, 从而在验证程序中构造了一个循环, 这种情况是允许的, 且在实际场景中十分有用. 但是需要注意的是 ``$ref`` schema互相引用明确禁止的, 循环引用会导致解析器产生死循环.

.. schema_example::

    {
      "definitions": {
        "alice": {
          "anyOf": [
            { "$ref": "#/definitions/bob" }
          ]
        },
        "bob": {
          "anyOf": [
            { "$ref": "#/definitions/alice" }
          ]
        }
      }
    }

.. index::
    single: $id

.. _id:

The $id property
----------------

``$id`` 属性是一个URI-reference, 主要有两个作用:

- 为schema声明了一个唯一标识符.

- 声明解析 ``$ref`` 对应的URI-references所用的base URI.

最好每个schema的顶层都设置 ``$id`` 属性指向一个绝对的URI(而不是相对引用). 举个例子, 如果您拥有 ``foo.bar`` 域名,
同时有一个address的schema, 此时就是可以把schema的 ``$id`` 设置为:

.. schema_example::

  { "$id": "http://foo.bar/schemas/address.json" }

这么做一方面可以为每个schema提供一个唯一的标志符, 另一方面也可以作为schema的下载地址.

注意到 ``$id`` 属性的第二个用途: 声明解析 ``$ref`` 对应的URI-references所用的base URI.
举个例子, 如果有以下schema:

.. schema_example::

  { "$ref": "person.json" }

在同一个文件内, 即使验证程序从其他地方(比如本地文件系统)加载了 ``address.json``, 同时也支持通过网络 ``http://foo.bar/schemas/person.json`` 加载 ``person.json``. 草案对这块的行为并没有明确地定义，验证程序对于如何加载引用的schema的实现各有不同.


|draft6|

.. draft_specific::

    --Draft 4
    在Draft 4中, ``$id`` 就是 ``id`` (没有$符).

``$id`` 既不应该为空字符串也不应该是空fragment, 因为没有意义.

Using $id with $ref
```````````````````
``$id`` 还提供了一种无须 JSON Pointer 引用子schema的方式. ``$id`` 可以直接通过唯一的名称来引用子schema, 而不是通过子schema在JSON树中的位置来引用.

接着上面的案例, 我们可以给address schema增加一个 ``$id`` 属性, 然后通过 ``$id`` 属性来引用该schema.

.. schema_example::

    {
      "$schema": "http://json-schema.org/draft-07/schema#",

      "definitions": {
        "address": {
          *"$id": "#address",
          "type": "object",
          "properties": {
            "street_address": { "type": "string" },
            "city":           { "type": "string" },
            "state":          { "type": "string" }
          },
          "required": ["street_address", "city", "state"]
        }
      },

      "type": "object",

      "properties": {
        *"billing_address": { "$ref": "#address" },
        *"shipping_address": { "$ref": "#address" }
      }
    }

.. note::

    Python实现的 ``jsonschema`` 库暂时还不支持这个功能.

Extending
---------

和 ``allOf`` 关键字一起使用的时候才能真正发挥 ``$ref`` 作用, ``anyOf`` 和 ``oneOf`` (参考
:ref:`combining` 小结).

接着说回收货地址, 由于送货的方法依赖收货地址的类型, 我们必须要知道该收货地址是居住地址还是公司地址.
但是对于账单地址, 并不需要存储地址类型. 

为了解决这个问题, 我们需要调整收货地址::

    "shipping_address": { "$ref": "#/definitions/address" }

用 ``allOf`` 关键字来组合address schema和地址类型字段::

    "shipping_address": {
      "allOf": [
        // Here, we include our "core" address schema...
        { "$ref": "#/definitions/address" },

        // ...and then extend it with stuff specific to a shipping
        // address
        { "properties": {
            "type": { "enum": [ "residential", "business" ] }
          },
          "required": ["type"]
        }
      ]
    }

综合上面的改动最终的JSON Schema如下,

.. schema_example::

    {
      "$schema": "http://json-schema.org/draft-06/schema#",

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

      "type": "object",

      "properties": {
        "billing_address": { "$ref": "#/definitions/address" },
        "shipping_address": {
          "allOf": [
            { "$ref": "#/definitions/address" },
            { "properties":
              { "type": { "enum": [ "residential", "business" ] } },
              "required": ["type"]
            }
          ]
        }
      }
    }
    --X
    // 由于缺失地址类型会导致校验失败:
    {
      "shipping_address": {
        "street_address": "1600 Pennsylvania Avenue NW",
        "city": "Washington",
        "state": "DC"
      }
    }
    --
    {
      "shipping_address": {
        "street_address": "1600 Pennsylvania Avenue NW",
        "city": "Washington",
        "state": "DC",
        "type": "business"
      }
    }

以上案例中可以看出, 不要重复大量的代码就可以构建出强大的JSON Schema.
