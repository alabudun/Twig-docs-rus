``extends``
===========

Тег ``extends`` используется для наследования шаблонов.

.. Примечание::

    Как и в PHP Twig не допускает множественного наследования. Однако, Twig поддерживает горизонтальное :doc:`reuse<use>`.

Давайте определим базовый шаблон, ``base.html``, для простой страницы с двумя колонками:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            {% block head %}
                <link rel="stylesheet" href="style.css" />
                <title>{% block title %}{% endblock %} - Мой сайт</title>
            {% endblock %}
        </head>
        <body>
            <div id="content">{% block content %}{% endblock %}</div>
            <div id="footer">
                {% block footer %}
                    &copy; Copyright 2013 <a href="http://example.com/">Вы</a>.
                {% endblock %}
            </div>
        </body>
    </html>

В этом примере тегом the :doc:`block<tags/block>` определяется 4 блока, которые мы и заменим. Все теги ``block`` сообщат шаблонизатору, что в последствии их можно будет переопределить

Дочерний шаблон
--------------

Дочерний шаблон может выглядеть так:

.. code-block:: jinja

    {% extends "base.html" %}

    {% block title %}Главная{% endblock %}
    {% block head %}
        {{ parent() }}
        <style type="text/css">
            .important { color: #336699; }
        </style>
    {% endblock %}
    {% block content %}
        <h1>Главная</h1>
        <p class="important">
            Приветсвую на своем потрясном сайте!
        </p>
    {% endblock %}

Тег ``extends`` здесь ключевой, он сообщает шаблонизатору кто "родитель" шаблона и существует ли он. Когда шаблон обрабаотывается, сначала обрабаотывается родительский. Тег наследования ``extends`` должен быть первым в шаблоне.

Следует учесть, что так как блок ``footer`` не определен, то используется родительский

Вы можете постоянно переопределять ``block`` блоком с таким же именем. Такие преобразования возможны потому что блоки работают в обоих направлениях.

Если вы хотите вывести блок несколько раз, вы можете использовать функцию ``block``:

.. code-block:: jinja

    <title>{% block title %}{% endblock %}</title>
    <h1>{{ block('title') }}</h1>
    {% block body %}{% endblock %}

Родительские блоки
-------------

Возможно отображать значение родительского блока, используя функцию :doc:`parent<functions/parent>`:

.. code-block:: jinja

    {% block sidebar %}
        <h3>Заголовок</h3>
        {{ parent() }}
    {% endblock %}

Параметры для закрытия блока
--------------------

Twig позволяет указывать какой именно блок стоит закрыть:

.. code-block:: jinja

    {% block sidebar %}
        {% block inner_sidebar %}
            ...
        {% endblock inner_sidebar %}
    {% endblock sidebar %}

Конечно после тега ``endblock`` слово должно содержать название блока.

Вложенность блоков и области видимости
-----------------------

Блоки могут быть вложены друг в друга. По умолчанию блоки имеют доступ к переменным других областей видимости

.. code-block:: jinja

    {% for item in seq %}
        <li>{% block loop_item %}{{ item }}{% endblock %}</li>
    {% endfor %}

Блочные сокращения
---------------

Для блоков с небольшим содержимым можно использовать сокращения. Следующие конструкции одинаковы:

.. code-block:: jinja

    {% block title %}
        {{ page_title|title }}
    {% endblock %}

.. code-block:: jinja

    {% block title page_title|title %}

Динамическое наследование
-------------------

Twig поддерживает динамическое наследование, используя переменную в качестве названия:

.. code-block:: jinja

    {% extends some_var %}

Если переменная имеет значение объекта ``Twig_Template`` Twig использует это как родительский шаблон::

    // {% extends layout %}

    $layout = $twig->loadTemplate('some_layout_template.twig');

    $twig->display('template.twig', array('layout' => $layout));

.. versionadded:: 1.2
    Возможность проверять шаблоны по массиву добавлена в Twig 1.2.

Вы можете указать массив названий шаблонов. Первый найденный шаблон будет использоваться в качестве родителя:

.. code-block:: jinja

    {% extends ['layout.html', 'base_layout.html'] %}

Условия при установке наследования
-----------------------

В качестве названия может быть использовано любое выражение, на пример:

.. code-block:: jinja

    {% extends standalone ? "minimum.html" : "base.html" %}

В этом примере шаблон будет унаследован "minimum.html", если ``standalone`` вернет ``true``, в противном случае "base.html"

Как устроенны блоки?
----------------

Блоки позволяют менять все, что находится внутри них, но никак не влияют на то, что происходит вокруг них

В следующем примере можно увидеть, как блоки работают, и что самое главное - как они не работают:

.. code-block:: jinja

    {# base.twig #}

    {% for post in posts %}
        {% block post %}
            <h1>{{ post.title }}</h1>
            <p>{{ post.body }}</p>
        {% endblock %}
    {% endfor %}

Если вы напишете такой блок, то при обработке на каждой итерации цикла, содержимое блока ``post`` будет перезаписанно:

.. code-block:: jinja

    {# child.twig #}

    {% extends "base.twig" %}

    {% block post %}
        <article>
            <header>{{ post.title }}</header>
            <section>{{ post.text }}</section>
        </article>
    {% endblock %}

Теперь при обработке дочернего шаблона, "родительский" цикл использует "детское" определение блока и в итоге получится это:

.. code-block:: jinja

    {% for post in posts %}
        <article>
            <header>{{ post.title }}</header>
            <section>{{ post.text }}</section>
        </article>
    {% endfor %}

Давайте попробуем другой пример с ``if`` условием:

.. code-block:: jinja

    {% if posts is empty %}
        {% block head %}
            {{ parent() }}

            <meta name="robots" content="noindex, follow">
        {% endblock head %}
    {% endif %}

Вопреки тому, что вы могли подумать, этот шаблон не определит блок условно, а только сделает переписываемым "дочерний" шаблон и будет выведен, если условие будет выполнено.

Если вы хотите, чтобы вывод блока был определен условно, напишите следующую конструкцию:

.. code-block:: jinja

    {% block head %}
        {{ parent() }}

        {% if posts is empty %}
            <meta name="robots" content="noindex, follow">
        {% endif %}
    {% endblock head %}

.. seealso:: :doc:`block<../functions/block>`, :doc:`block<../tags/block>`, :doc:`parent<../functions/parent>`, :doc:`use<../tags/use>`
