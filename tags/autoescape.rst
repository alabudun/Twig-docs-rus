``autoescape``
==============

Внезависимости от того, включено экранирование или нет, вы можете включить его для блока:

.. code-block:: jinja

    {# Следующий синтаксис работает для Twig 1.8 -- смотрите примечания для предыдущих версий #}

    {% autoescape %}
        Все будет автоматически экранированно, как HTML-код
    {% endautoescape %}

    {% autoescape 'html' %}
        Все будет автоматически экранированно, как HTML-код
    {% endautoescape %}

    {% autoescape 'js' %}
        Все будет автоматически экранированно, как js-код
    {% endautoescape %}

    {% autoescape false %}
        Экранирование отключено
    {% endautoescape %}

.. Примечание::

    Перед Twig 1.8, синтаксис был другим:

    .. code-block:: jinja

        {% autoescape true %}
            Все будет автоматически экранированно, как HTML-код
        {% endautoescape %}

        {% autoescape false %}
            Экранирование отключено
        {% endautoescape %}

        {% autoescape true js %}
            Все будет автоматически экранированно, как js-код
        {% endautoescape %}

Некоторые переменные экранировать не нужно даже тогда, когда блок помечен тегом ``autoescape``. Для этого можно использовать фильтр :doc:`raw<../filters/raw>`:

.. code-block:: jinja

    {% autoescape %}
        {{ safe_value|raw }}
    {% endautoescape %}

.. Примечание::

    Twig достаточно умен, чтобы не экранировать уже экранированные строки с помощью фильтра :doc:`escape<../filters/escape>`.

.. Примечание::

    Глава :doc:`Twig for Developers<../api>` даст более подробную информацию о экранировании
