``attribute``
=============

Версия:: 1.2
    The ``attribute`` function was added in Twig 1.2.

``attribute`` can be used to access a "dynamic" attribute of a variable:

.. code-block:: jinja

    {{ attribute(object, method) }}
    {{ attribute(object, method, arguments) }}
    {{ attribute(array, item) }}

Заметка:

    The resolution algorithm is the same as the one used for the ``.``
    notation, except that the item can be any valid expression.
