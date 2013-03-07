``filter``
==========

C помощью тега ``filter`` можно применить фильтры не к одной переменной, а сразу к блоку кода:

.. code-block:: jinja

    {% filter upper %}
        Весь текст написанный здесь будет выведет в верхнем регистре
    {% endfilter %}

Вы также можете связывать фильтры:

.. code-block:: jinja

    {% filter lower|escape %}
        <strong>ЖИРНЫЙ ТЕКСТ</strong>
    {% endfilter %}

    {# на выходе: "&lt;strong&gt;жирный текст&lt;/strong&gt;" #}
