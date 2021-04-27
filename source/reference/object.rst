.. index::
   single: object

.. _object:

object
------

.. contents:: :local:

JSON 中 Object 是映射类型, 负责将 "key" 映射到 "value", 其中"key" 必须是 string 类型. 
一般情况下称 key-value 为 "property".

.. language_specific::
   --Python
   Python中 "object" 类似于 ``dict`` . 不同的是, JSON 的key只能是 string 类型的,
   但 dicts 可以使用任意可hash的类型作为key.

   不要混淆两个地方的 "object":  Python中用 ``object`` 来表示所有对象的基类, 
   而在JSON中, "object"就是字面意思, 表示string类型的key到value的映射关系.

   --Ruby
   Ruby中 "object" 类似于 ``Hash`` 类型. 不同的是, JSON 的key只能是 string 类型的,
   所有非字符串类型的 key 都要对应的转换成 string 类型.

   不要混淆两个地方的 "object":  Ruby中中用 ``Object`` 来表示所有对象的基类, 
   而在JSON中, "object"就是字面意思, 表示string类型的key到value的映射关系.


.. schema_example::

    { "type": "object" }
    --
    {
       "key"         : "value",
       "another_key" : "another_value"
    }
    --
    {
        "Sun"     : 1.9891e30,
        "Jupiter" : 1.8986e27,
        "Saturn"  : 5.6846e26,
        "Neptune" : 10.243e25,
        "Uranus"  : 8.6810e25,
        "Earth"   : 5.9736e24,
        "Venus"   : 4.8685e24,
        "Mars"    : 6.4185e23,
        "Mercury" : 3.3022e23,
        "Moon"    : 7.349e22,
        "Pluto"   : 1.25e22
    }
    --X
    // JSON中使用非字符串类型的key:
    {
        0.01 : "cm"
        1    : "m",
        1000 : "km"
    }
    --X
    "Not an object"
    --X
    ["An", "array", "not", "an", "object"]


.. index::
   single: object; properties
   single: properties
   single: additionalProperties

.. _additionalProperties:

Properties
''''''''''

schema 中的 ``properties`` 字段用来表示对象的属性(key-value 对), ``properties`` 对应的值是object, object上每个key对应属性的字段名, key对应的value表示校验属性值的 JSON Schema.

比如我们想定义一个address的schema, 包括数字、街道名以及街道类型:

.. schema_example::

    {
      "type": "object",
      "properties": {
        "number":      { "type": "number" },
        "street_name": { "type": "string" },
        "street_type": { "type": "string",
                         "enum": ["Street", "Avenue", "Boulevard"]
                       }
      }
    }
    --
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue" }
    --X
    // 如果给 number 字段设置 string 类型的数据, 校验不通过:
    { "number": "1600", "street_name": "Pennsylvania", "street_type": "Avenue" }
    --
    // 对象中缺少字段可以通过校验. 具体可以参考 `required` 章节.
    { "number": 1600, "street_name": "Pennsylvania" }
    --
    // 空对象也可以通过校验:
    { }
    --
    // 对象中包含其他的属性也可以通过校验:
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue",
      "direction": "NW" }

``additionalProperties`` 字段用来限制其他部分, 也就是那些没有在 ``properties`` 字段中列出来的属性. 默认情况下任意其他属性都能通过校验.

``additionalProperties`` 字段可以是 boolean 也可以是 object 类型. 如果 ``additionalProperties`` 被设置成 ``false``,  表示对象中不能出现其他属性.

沿用前面的例子, 但是这次把 ``additionalProperties`` 设为 ``false``.

.. schema_example::

    {
      "type": "object",
      "properties": {
        "number":      { "type": "number" },
        "street_name": { "type": "string" },
        "street_type": { "type": "string",
                         "enum": ["Street", "Avenue", "Boulevard"]
                       }
      },
      "additionalProperties": false
    }
    --
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue" }
    --X
    // 由于 ``additionalProperties`` 被设置为 ``false``, 额外的 "direction" 导致校验不通过:
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue",
      "direction": "NW" }

如果 ``additionalProperties`` 字段是 object , 则表示校验 ``properties`` 之外的属性的 schema.

如下所示, 表示了一个允许其他属性, 但是其他属性的类型必须都是 string 类型的 schema:

.. schema_example::

    {
      "type": "object",
      "properties": {
        "number":      { "type": "number" },
        "street_name": { "type": "string" },
        "street_type": { "type": "string",
                         "enum": ["Street", "Avenue", "Boulevard"]
                       }
      },
      "additionalProperties": { "type": "string" }
    }
    --
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue" }
    --
    // 这个例子可以通过校验, 因为其他属性 direction 是 string 类型的:
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue",
      "direction": "NW" }
    --X
    // 这个例子不能通过校验, 因为其他属性 office_number 是 number 类型而不是 string 类型:
    { "number": 1600, "street_name": "Pennsylvania", "street_type": "Avenue",
      "office_number": 201  }


.. index::
   single: object; required properties
   single: required

.. _required:

Required Properties
'''''''''''''''''''

一般情况下,  ``properties`` 中描述的字段都是可选的. 但是可以通过 ``required`` 关键字来限制那些字段是必选的.

``required`` 可以接受包含零个或多个string, 但需要保证 string 的唯一性.

.. draft_specific::

   --Draft 4
   Draft 4规范中, ``required`` 字段至少包含一个string.

如下的 schema 描述了 user 对象必须包含name和e-mail字段, address或者telephone字段都是可选的.

.. schema_example::

    {
      "type": "object",
      "properties": {
        "name":      { "type": "string" },
        "email":     { "type": "string" },
        "address":   { "type": "string" },
        "telephone": { "type": "string" }
      },
      "required": ["name", "email"]
    }
    --
    {
      "name": "William Shakespeare",
      "email": "bill@stratford-upon-avon.co.uk"
    }
    --
    // 对象中可以包含schema之外的字段：
    {
      "name": "William Shakespeare",
      "email": "bill@stratford-upon-avon.co.uk",
      "address": "Henley Street, Stratford-upon-Avon, Warwickshire, England",
      "authorship": "in question"
    }
    --X
    // 缺少 "email" 字段导致整个JSON document校验不通过:
    {
      "name": "William Shakespeare",
      "address": "Henley Street, Stratford-upon-Avon, Warwickshire, England",
    }

.. index::
   single: object; property names
   single: propertyNames

Property names
''''''''''''''

|draft6|

JSON Schema 可以在不约束 value 的情况下只约束 key 的命名规则. Property names字段适合那些不希望在properties中列出所有的key, 但希望能够限制 key 的命名规则的情况. 比如可能想确保所有的 key 都是合法的ASCII字符, 这样可以在代码中将这些key用作属性名.

.. schema_example::

    {
      "type": "object",
      "propertyNames": {
        "pattern": "^[A-Za-z_][A-Za-z0-9_]*$"
      }
    }
    --
    {
      "_a_proper_token_001": "value"
    }
    --X
    {
      "001 invalid": "value"
    }

因为 JSON 中所有的 key 都必须是 string 类型的, 也就意味着 ``propertyNames`` 对应的字段默认包含如下的规则::

    { "type": "string" }

.. index::
   single: object; size
   single: minProperties
   single: maxProperties

Size
''''


object 中属性的数量可以通过 ``minProperties`` 、 ``maxProperties`` 来约束.但需要保证 ``minProperties`` 、 ``maxProperties`` 都是非负整数.

.. schema_example::

    {
      "type": "object",
      "minProperties": 2,
      "maxProperties": 3
    }
    --X
    {}
    --X
    { "a": 0 }
    --
    { "a": 0, "b": 1 }
    --
    { "a": 0, "b": 1, "c": 2 }
    --X
    { "a": 0, "b": 1, "c": 2, "d": 3 }


.. index::
   single: object; dependencies
   single: dependencies


Dependencies
''''''''''''

.. note::
    Dependencies 是 JSON Schema 中的高级特性.

``dependencies`` 字段允许根据特定的属性衍生出不同的schema.

JSON Schema中主要有两种类型的依赖:

- **Property dependencies** 表示只有属性A存在的情况下属性B才会存在.

- **Schema dependencies** 表示当特定属性存在的情况下, schema 才会发生变化.

Property dependencies
^^^^^^^^^^^^^^^^^^^^^

先从一个简单的 property dependencies 案例开始. 假设有一个表示顾客的 schema. 有卡号的情况下必须要有账单地址. 如果没有卡号,则账单地址也是可选的属性. 可以通过 ``dependencies`` 字段来描述这种属性之间的依赖关系. ``dependencies`` 字段的值是 object , 对象中的 key-value 描述了属性 *p* 到
其依赖属性数组之间的依赖关系.

下面的例子中, 只要 ``credit_card`` 属性出现, 则对象中必须包含 ``billing_address`` 属性.

.. schema_example::

    {
      "type": "object",

      "properties": {
        "name": { "type": "string" },
        "credit_card": { "type": "number" },
        "billing_address": { "type": "string" }
      },

      "required": ["name"],

      "dependencies": {
        "credit_card": ["billing_address"]
      }
    }
    --
    {
      "name": "John Doe",
      "credit_card": 5555555555555555,
      "billing_address": "555 Debtor's Lane"
    }
    --X
    // 对象中包含 ``credit_card`` 字段但是没有 ``billing_address`` 字段.
    {
      "name": "John Doe",
      "credit_card": 5555555555555555
    }
    --
    // 对象中既不包含 ``credit_card`` 字段也不包含 ``billing_address``, 这种情况也是符合要求的.
    {
      "name": "John Doe"
    }
    --
    // 需要注意的是依赖关系不是双向的. 包含billing address字段但不包含credit card字段也是ok的.
    {
      "name": "John Doe",
      "billing_address": "555 Debtor's Lane"
    }

要想实现上面提到的效果(双向依赖关系), 也可以显式的定义两个依赖关系:

.. schema_example::

    {
      "type": "object",

      "properties": {
        "name": { "type": "string" },
        "credit_card": { "type": "number" },
        "billing_address": { "type": "string" }
      },

      "required": ["name"],

      "dependencies": {
        "credit_card": ["billing_address"],
        "billing_address": ["credit_card"]
      }
    }
    --X
    // 对象中包含 ``credit_card`` 但不包含 ``billing_address``.
    {
      "name": "John Doe",
      "credit_card": 5555555555555555
    }
    --X
    // 对象中包含 ``billing_address`` 但不包含 ``credit_card``.
    {
      "name": "John Doe",
      "billing_address": "555 Debtor's Lane"
    }


Schema dependencies
^^^^^^^^^^^^^^^^^^^

Schema dependencies 和 property dependencies 类似, 但schema dependencies不仅可以指定依赖的其他属性, 还能描述其他限制条件.

上面的例子也可以通过 schema dependencies 的形式描述:

.. schema_example::

    {
      "type": "object",

      "properties": {
        "name": { "type": "string" },
        "credit_card": { "type": "number" }
      },

      "required": ["name"],

      "dependencies": {
        "credit_card": {
          "properties": {
            "billing_address": { "type": "string" }
          },
          "required": ["billing_address"]
        }
      }
    }
    --
    {
      "name": "John Doe",
      "credit_card": 5555555555555555,
      "billing_address": "555 Debtor's Lane"
    }
    --X
    // 对象中包含 ``credit_card`` 但不包含 ``billing_address``:
    {
      "name": "John Doe",
      "credit_card": 5555555555555555
    }
    --
    // 这种包含 ``billing_address`` 但不包含 ``credit_card`` 的情况能通过校验, ``billing_address`` 更像是额外属性:
    {
      "name": "John Doe",
      "billing_address": "555 Debtor's Lane"
    }


.. index::
   single: object; regular expression
   single: patternProperties

.. _patternProperties:

Pattern Properties
''''''''''''''''''

正如前面看到的, ``additionalProperties`` 可以限制对象不能包含其他属性或者为其他属性指定schema.
有些情况下 ``additionalProperties`` 还不够，可能还需要限制其他属性的字段名, 或者限制其他属性的value满足特定的schema. ``patternProperties`` 就是为了解决这类需求的: ``patternProperties`` 字段用于映射正则表达式到schema. 如果其他属性的key符合特定的正则表达式，则该属性的value也应该满足对应的schema.

.. note::
    定义正则表达式的时候需要注意的是表达式可以匹配属性名的任意位置. 举个例子, 正则表达式 ``"p"`` 会匹配所有包含 ``p`` 的字段, 比如 ``"apple"`, 但不会匹配属性名为 ``"p"`` 的字段. 这样的话在正则的前后加上 ``^...$`` 能够减少混乱, 比如 ``"^p$"``.

下面的例子中, 以 ``S_`` 开头的属性必须是 string 类型, 以 ``I_`` 开头的属性必须是 integer 类型. 其他属性需要在 properties 中显式定义, 在 properties 中显式定义的属性都是符合要求的, 但是不允许包含不符合正则表达式的其他属性.

.. schema_example::

    {
      "type": "object",
      "patternProperties": {
        "^S_": { "type": "string" },
        "^I_": { "type": "integer" }
      },
      "additionalProperties": false
    }
    --
    { "S_25": "This is a string" }
    --
    { "I_0": 42 }
    --X
    // 以 ``S_`` 开头的字段都必须是 string 类型的
    { "S_0": 42 }
    --X
    // 以 ``I_`` 开头的字段都必须是 integer 类型的
    { "I_42": "This is a string" }
    --X
    // 下面是一个不符合任何正则的key:
    { "keyword": "value" }

``patternProperties`` 会与 ``additionalProperties`` 配合使用.  
这种情况下 ``additionalProperties`` 表示那些既没有显式在 ``properties`` 中申明也不符合 ``patternProperties``的属性. 在上面的例子的基础上, 增加了 ``"builtin"`` 属性, 必须是 number 类型, 同时申明了所有的其他属性(非builtin、不匹配 ``patternProperties`` )必须是 string 类型的:

.. schema_example::

    {
      "type": "object",
      "properties": {
        "builtin": { "type": "number" }
      },
      "patternProperties": {
        "^S_": { "type": "string" },
        "^I_": { "type": "integer" }
      },
      "additionalProperties": { "type": "string" }
    }
    --
    { "builtin": 42 }
    --
    // "keyword" 不符合正则、也不是builtin, 命中 additionalProperties 规则, 保证字段是string类型即可通过校验
    { "keyword": "value" }
    --X
    // "keyword" 必须是 string 类型:
    { "keyword": 42 }
