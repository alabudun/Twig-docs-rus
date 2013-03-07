``if``
======

Условия ``if`` в Twig сравнимы с PHP.

Проще всего проверить ``true``/``false`` выражения:

.. code-block:: jinja

    {% if online == false %}
        <p>Сайт на обслуживании, зайдите позже.</p>
    {% endif %}

Вы также можете проверить, пустой ли массив:

.. code-block:: jinja

    {% if users %}
        <ul>
            {% for user in users %}
                <li>{{ user.username|e }}</li>
            {% endfor %}
        </ul>
    {% endif %}

Примечание:

    Если вы хотите проверить определена ли переменная, используйте выражение ``if users is defined``.

Ветвление условий ``elseif`` и ``else`` Можно использовать также, как в PHP:

.. code-block:: jinja

    {% if kenny.sick %}
        Кенни болен.
    {% elseif kenny.dead %}
        Ты убил Кенни! Ублюдок!!!
    {% else %}
        Кенни в порядке - до сих пор
    {% endif %}
