``set``
=======

Внутри шаблонов можно создавать переменные с помощью тега ``set``:

.. code-block:: jinja

    {% set foo = 'foo' %}

    {% set foo = [1, 2] %}

    {% set foo = {'foo': 'bar'} %}

    {% set foo = 'foo' ~ 'bar' %}

    {% set foo, bar = 'foo', 'bar' %}

С помощью тега ``set`` в переменную можно записать блок:

.. code-block:: jinja

    {% set foo %}
      <div id="pagination">
        ...
      </div>
    {% endset %}