.. _json-module:

-------------------------------------------------------------------------------
                          Module `json`
-------------------------------------------------------------------------------

===============================================================================
                                   Overview
===============================================================================

The ``json`` module provides JSON manipulation routines. It is based on the
`Lua-CJSON module by Mark Pulford <http://www.kyne.com.au/~mark/software/lua-cjson.php>`_.
For a complete manual on Lua-CJSON please read
`the official documentation <http://www.kyne.com.au/~mark/software/lua-cjson-manual.html>`_.

===============================================================================
                                    Index
===============================================================================

Below is a list of all ``json`` functions and members.

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2

    +--------------------------------------+---------------------------------+
    | Name                                 | Use                             |
    +======================================+=================================+
    | :ref:`json.encode()                  | Convert a Lua object to a JSON  |
    | <json-encode>`                       | string                          |
    +--------------------------------------+---------------------------------+
    | :ref:`json.decode()                  | Convert a JSON string to a Lua  |
    | <json-decode>`                       | object                          |
    +--------------------------------------+---------------------------------+
    | :ref:`json.NULL                      | Analog of Lua's "nil"           |
    | <json-null>`                         |                                 |
    +--------------------------------------+---------------------------------+
    | :ref:`json.cfg()                     | Set global flags                |
    | <json-module_cfg>`                   |                                 |
    +--------------------------------------+---------------------------------+

.. module:: json

.. _json-encode:

.. function:: encode(lua-value [, configuration])

    Convert a Lua object to a JSON string.

    :param lua_value: either a scalar value or a Lua table value.
    :param configuration: see :ref:`json.cfg <json-module_cfg>`
    :return: the original value reformatted as a JSON string.
    :rtype: string

    **Example:**

    .. code-block:: tarantoolsession

        tarantool> json=require('json')
        ---
        ...
        tarantool> json.encode(123)
        ---
        - '123'
        ...
        tarantool> json.encode({123})
        ---
        - '[123]'
        ...
        tarantool> json.encode({123, 234, 345})
        ---
        - '[123,234,345]'
        ...
        tarantool> json.encode({abc = 234, cde = 345})
        ---
        - '{"cde":345,"abc":234}'
        ...
        tarantool> json.encode({hello = {'world'}})
        ---
        - '{"hello":["world"]}'
        ...

.. _json-decode:

.. function:: decode(string [,configuration])

    Convert a JSON string to a Lua object.

    :param string string: a string formatted as JSON.
    :param configuration: see :ref:`json.cfg <json-module_cfg>`
    :return: the original contents formatted as a Lua table.
    :rtype: table

    **Example:**

    .. code-block:: tarantoolsession

        tarantool> json = require('json')
        ---
        ...
        tarantool> json.decode('123')
        ---
        - 123
        ...
        tarantool> json.decode('[123, "hello"]')
        ---
        - [123, 'hello']
        ...
        tarantool> json.decode('{"hello": "world"}').hello
        ---
        - world
        ...

    See the tutorial
    :ref:`Sum a JSON field for all tuples <c_lua_tutorial-sum_a_json_field>`
    to see how ``json.decode()`` can fit in an application.

.. _json-null:

.. data:: NULL

    A value comparable to Lua "nil" which may be useful as a placeholder in a
    tuple.

    **Example:**

    .. code-block:: tarantoolsession

        -- When nil is assigned to a Lua-table field, the field is null
        tarantool> {nil, 'a', 'b'}
        ---
        - - null
          - a
          - b
        ...
        -- When json.NULL is assigned to a Lua-table field, the field is json.NULL
        tarantool> {json.NULL, 'a', 'b'}
        ---
        - - null
          - a
          - b
        ...
        -- When json.NULL is assigned to a JSON field, the field is null
        tarantool> json.encode({field2 = json.NULL, field1 = 'a', field3 = 'c'})
        ---
        - '{"field2":null,"field1":"a","field3":"c"}'
        ...

    The JSON output structure can be specified with ``__serialize``:

    * ``__serialize="seq"`` for an array
    * ``__serialize="map"`` for a map

    Serializing 'A' and 'B' with different ``__serialize`` values causes different
    results:

    .. code-block:: tarantoolsession

        tarantool> json.encode(setmetatable({'A', 'B'}, { __serialize="seq"}))
        ---
        - '["A","B"]'
        ...
        tarantool> json.encode(setmetatable({'A', 'B'}, { __serialize="map"}))
        ---
        - '{"1":"A","2":"B"}'
        ...
        tarantool> json.encode({setmetatable({f1 = 'A', f2 = 'B'}, { __serialize="map"})})
        ---
        - '[{"f2":"B","f1":"A"}]'
        ...
        tarantool> json.encode({setmetatable({f1 = 'A', f2 = 'B'}, { __serialize="seq"})})
        ---
        - '[[]]'
        ...

.. _json-module_cfg:

.. function:: cfg(table)

    Set values affecting behavior of :ref:`json.encode <json-encode>`
    and :ref:`json.decode <json-decode>`.

    The values are all either integers or boolean ``true``/``false`` values.

    .. container:: table

        .. rst-class:: left-align-column-1
        .. rst-class:: center-align-column-2
        .. rst-class:: left-align-column-3

        +---------------------------------+---------+-------------------------------------------+
        | Option                          | Default | Use                                       |
        +=================================+=========+===========================================+
        | ``cfg.encode_max_depth``        |   128   | Set max recursion depth for encoding      |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_deep_as_nil``      |  false  | A flag whether a table with too high      |
        |                                 |         | nest level should be cropped. The         |
        |                                 |         | not-encoded fields are replaced with      |
        |                                 |         | one null. If not set, too high            |
        |                                 |         | nesting is considered an error            |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_invalid_numbers``  |  true   | Enable encoding of NaN and Inf numbers    |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_number_precision`` | 14      | Set point numbers precision               |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_load_metatables``  | true    | Show on ``__serialize`` field in a        |
        |                                 |         | metatable (if exists). See example below  |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_use_tostring``     | false   | Enable ``tostring()`` usage for unknown   |
        |                                 |         | types                                     |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_invalid_as_nil``   |  false  | Use NULL for all unrecognizable types     |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_sparse_convert``   | true    | Handle excessively sparse arrays as maps  |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_sparse_ratio``     |  2      | Permissible number of missing values in   |
        |                                 |         | a sparse array. See example below         |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.encode_sparse_safe``      | 10      | Limit ensuring that small Lua arrays      |
        |                                 |         | are always encoded as sparse arrays       |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.decode_invalid_numbers``  |  true   | Set floating point numbers precision      |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.decode_save_metatables``  |  true   | Save ``__serialize`` meta-value for       |
        |                                 |         | decoded arrays and map                    |
        +---------------------------------+---------+-------------------------------------------+
        | ``cfg.decode_max_depth``        |  128    | Max recursion depth for decoding          |
        +---------------------------------+---------+-------------------------------------------+

    **Example:**

    The following code will encode 0/0 as nan ("not a number")
    and 1/0 as inf ("infinity"), rather than returning nil or an error message:

    .. code-block:: lua

        json = require('json')
        json.cfg{encode_invalid_numbers = true}
        x = 0/0
        y = 1/0
        json.encode({1, x, y, 2})

    The result of the ``json.encode()`` request will look like this:

    .. code-block:: tarantoolsession

        tarantool> json.encode({1, x, y, 2})
        ---
        - '[1,nan,inf,2]
        ...

    To achieve the same effect for only one call to ``json.encode()`` without
    changing the configuration persistently, you can use
    ``json.encode({1, x, y, 2}, {encode_invalid_numbers = true})``.

    The same configuration settings exist for :ref:`MsgPack
    <msgpack-cfg>`, and for :ref:`YAML <yaml-cfg>`.

.. _json-module.cfg_encode_deep_as_nil:

.. NOTE::

    **Behavior change:** Before Tarantool version 1.10.4,
    if a nested structure was deeper than ``cfg.encode_max_depth``,
    the deeper levels were cropped (encoded as nil).

    Now, the result is an error suggesting that ``cfg.encode_max_depth``
    is not deep enough. To return to the old behavior, say
    ``cfg.encode_deep_as_nil = true``.

    This option is ignored for ``YAML``.

.. _Lua-CJSON module by Mark Pulford: http://www.kyne.com.au/~mark/software/lua-cjson.php
.. _the official documentation: http://www.kyne.com.au/~mark/software/lua-cjson-manual.html


    .. * ``cfg.encode_deep_as_nil`` (default is false) -- see :ref:`below <json-module.cfg_encode_deep_as_nil>`
    .. * ``cfg.encode_invalid_as_nil`` (default is false) -- use ``null`` for all
    ..   unrecognizable types
    .. * ``cfg.encode_invalid_numbers`` (default is true) -- enables encoding of NaN and Inf numbers
    .. * ``cfg.encode_load_metatables`` (default is true) -- load metatables
    .. * ``cfg.encode_max_depth`` (default is 128) -- maximum nesting depth in a structure
    .. * ``cfg.encode_number_precision`` (default is 14) -- maximum post-decimal digits
    .. * ``cfg.encode_sparse_convert`` (default is true) -- handle excessively sparse arrays as maps
    .. * ``cfg.encode_sparse_ratio`` (default is 2) -- how sparse an array can be
    .. * ``cfg.encode_sparse_safe`` (default is 10) -- how much can safely be sparse
    .. * ``cfg.encode_use_tostring`` (default is false) -- use ``tostring`` for
    ..   unrecognizable types
    .. * ``cfg.decode_invalid_numbers`` (default is true) -- allow nan and inf
    .. * ``cfg.decode_max_depth`` (default is 128) -- maximum nesting depth in a structure
    .. * ``cfg.decode_save_metatables`` (default is true) -- like ``encode_load_metatables``
