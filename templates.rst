Twig для дизайнеров
===========================
Этот документ описывает синтаксис и семантику шаблонов. Будет наиболее полезен тем, кто создает Twig шаблоны

Начало
--------
Шаблон - это просто текстовой файл. Из шаблона можно сгенерировать любой текстовой документ(HTML, XML, CSV, LaTeX, и тд.). Twig шаблоны может не прикреплены за каким-либо расширением, хотя обычно это ``.html`` или ``.xml``.

Шаблон может содержать **переменные** или **выражения**, которые заменяются значениями, во время генерации шаблона.
Также в шаблонах существуют **теги**, котролирующие логику сборки шаблона.

Рассмотрим небольшой пример шаблона, В котором показаны некоторые основы создания шаблонов:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>Мой сайт</title>
        </head>
        <body>
            <ul id="navigation">
            {% for item in navigation %}
                <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
            {% endfor %}
            </ul>

            <h1>Моя статья</h1>
            {{ text }}
        </body>
    </html>

Здесь присутвует два вида разделителей:
 - ``{% ... %}`` используется для выполнения выражений: циклов, условий, тегов
 - ``{{ ... }}`` используется для вывода значений выражения в шаблон

Интеграция с IDE
----------------

Множество IDE поддерживают подсветку синтаксиса и автодополнение:

* *Textmate* : `Twig bundle`_
* *Vim* : `Jinja syntax plugin`_
* *Netbeans* : `Twig syntax plugin`_ (до 7.1, встроен с 7.2)
* *PhpStorm* (native as of 2.1)
* *Eclipse* : `Twig plugin`_
* *Sublime Text* : `Twig bundle`_
* *GtkSourceView* : `Twig language definition`_ (используется gedit и другие проекты)
* *Coda* и *SubEthaEdit* : `Twig syntax mode`_
* *Coda 2* : `other Twig syntax mode`_
* *Komodo* и *Komodo Edit* : Twig highlight/syntax check mode
* *Notepad++* : `Notepad++ Twig Highlighter`_
* *Emacs* :  `web-mode.el`_

Переменные
---------

Все переменные, переданные в шаблон можно использовать повторно. Переменные могут включать в себя аттрибуты. Вид переменных зависит от приложения.

Атрибутами переменных являются методы и свойства PHP-объектов, или переменные массивов.
Для доступа к атрибутам можно использовать (``.``), или (``[]``):

.. code-block:: jinja

    {{ foo.bar }}
    {{ foo['bar'] }}

Когда атрибуты содержат спец.символы (например ``-`` воспримется, как математический оператор "минус"), используйте функцию ``attribute``:

.. code-block:: jinja

    {# Подобное не будет работать foo.data-foo #}
    {{ attribute(foo, 'data-foo') }}

.. note::
    Выжно понимать, что фигурные скобки - *не* часть переменной, а оператор вывода значений переменных в шаблон. Поэтому при использовании переменных в тегах не нужно ставить фигурные скобки.

Если переменная, или атрибут не найден, то в шаблоне это заменится на ``null`` значение. Однако если опция ``strict_variables`` установленна как ``true``, Twig выдаст сообщение об ошибке (см. :ref:`environment options<environment_options>`).

.. sidebar:: Подробнее об устройстве переменных

    Рассмотрим что будет в PHP, когда в Twig ищет переменную ``foo.bar``

    * Проверим: ``foo`` - массив и ``bar`` один из его элементов;
    * Проверим: ``foo`` - объект и ``bar`` одно из его свойств;
    * Проверим: ``foo`` - объект и ``bar`` валидный метод (если ``bar`` конструктор - use ``__construct()``);
    * Проверим: ``foo`` - объект и существует метод ``getBar``;
    * Проверим: ``foo`` - объект и существует метод ``isBar``;
    * В противном случае ``null``.

    ``foo['bar']`` в этом случае используются строго массивы:

    * Проверим: ``foo`` - массив и ``bar`` один из его элементов;
    * В противном случае ``null``.

.. note::

    Если вы хотите получить динамическое свойство объекта, используйте :doc:`attribute<functions/attribute>` функцию вместо этого.

Глобальные переменные
~~~~~~~~~~~~~~~~

Эти переменные всегда доступны в шаблоне:

* ``_self``: ссылается на текущий шаблон;
* ``_context``: ссылается на текущий контекст;
* ``_charset``: ссылается на текущую кодировку.

Установка переменных
~~~~~~~~~~~~~~~~~

Вы можете устанавливать значения переменных в блоках кода для этого используйте тег :doc:`set<tags/set>`:

.. code-block:: jinja

    {% set foo = 'foo' %}
    {% set foo = [1, 2] %}
    {% set foo = {'foo': 'bar'} %}

Фильтры
-------

Переменные можно фильтровать. Фильтры отделяются от переменных прямой чертой (``|``) и могут иметь аргументы внутри. Можно использовать сразу несколько фильтров, фильтры применяются по очереди.

В этом примере из ``name`` удаляются все HTML-теги и title-cases:

.. code-block:: jinja

    {{ name|striptags|title }}

Аргументы фильтров записываются в скобках после названия. В следующем примере к значению ``list`` добавится ``,``:

.. code-block:: jinja

    {{ list|join(', ') }}

Чтобы применить фильтр к блоку кода - оберните его тэгом :doc:`filter<tags/filter>`:

.. code-block:: jinja

    {% filter upper %}
        этот текст будет выведен в верхнем регистре
    {% endfilter %}

Подробнее о фильтрах :doc:`filters<filters/index>`

Функции
---------

Функции можно вызвать для генирации контента. Функции вызываются по их названию, как фильтры, аргументы также вставляются в (``()``).

Для примера функция ``range`` возвращает список целых чисел, аргументами является начальное и конечное число списка

.. code-block:: jinja

    {% for i in range(0, 3) %}
        {{ i }},
    {% endfor %}

Подробнее о функциях :doc:`functions<functions/index>`.

Названия аргументов
---------------

.. versionadded:: 1.12
    Поддержка названий аргументов была добавлена в Twig 1.12.

Аргументы для фильтров и функций могут быть дополнительно названы:

.. code-block:: jinja

    {% for i in range(low=1, high=10, step=2) %}
        {{ i }},
    {% endfor %}

Использование именованных аргументов делает шаблоны более понятными:

.. code-block:: jinja

    {{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}

    {# В сравнении с  #}

    {{ data|convert_encoding(from='iso-2022-jp', to='UTF-8') }}

Также именованные аргументы полезны, когда вам не хочется менять некоторые аргументы по умолчанию, но и записывать их вам тоже не хочется
to change the default value:

.. code-block:: jinja

    {# Первый аргумент - формат даты, который задан в приложении глобально #}
    {{ "now"|date(null, "Europe/Paris") }}

    {# Или можно пропустить ``format``, но указать ``timezone`` #}
    {{ "now"|date(timezone="Europe/Paris") }}

Вы также можете использовать оба варианта записи аргументов, однако это не рекомендуется, потому что приведет к путанице:

.. code-block:: jinja

    {# Оба варианта - рабочие #}
    {{ "now"|date('d/m/Y H:i', timezone="Europe/Paris") }}
    {{ "now"|date(timezone="Europe/Paris", 'd/m/Y H:i') }}

.. tip::

    По каждой функции и каждому фильтру есть страница документации, где указаны какие аргументы доступны и их названия

Управляющие структуры
-----------------

К управляющим структурам относится все условные операторы (такие как  ``if``/``elseif``/``else``), ``for``-loops. Управляющие структуры находятся внутри``{% ... %}`` блоков.

На пример чтобы отобразить список пользователей ``users``, используется тег :doc:`for<tags/for>`:

.. code-block:: jinja

    <h1>Пользователи</h1>
    <ul>
        {% for user in users %}
            <li>{{ user.username|e }}</li>
        {% endfor %}
    </ul>

Тег :doc:`if<tags/if>` может быть использован для проверки:

.. code-block:: jinja

    {% if users|length > 0 %}
        <ul>
            {% for user in users %}
                <li>{{ user.username|e }}</li>
            {% endfor %}
        </ul>
    {% endif %}

Подробнее о тегах :doc:`tags<tags/index>`.

Комментирование
--------------

Для комментирования части кода, или пояснений к нему используйте ``{# ... #}``.

.. code-block:: jinja

    {# Примечание: закоментированно, тк больше не используется
        {% for user in users %}
            ...
        {% endfor %}
    #}

Подключение шаблонов
-------------------------

Тег :doc:`include<tags/include>` возвращает содержимое шаблона из файла:

.. code-block:: jinja

    {% include 'sidebar.html' %}

По умолчанию содержимое подключаемых шаблонов встает в место вызова

В подключенном шаблоне определены переменные родительского шаблона

.. code-block:: jinja

    {% for box in boxes %}
        {% include "render_box.html" %}
    {% endfor %}

Подключенный шаблон ``render_box.html`` имеет доступ к переменной ``box``.

Название файла зависит от загрузчика шаблонов. Например ``Twig_Loader_Filesystem`` позволяет получить шаблон по названию файла. Также можно указать путь до дериктории шаблона, используя слэш ``/``:

.. code-block:: jinja

    {% include "sections/articles/sidebar.html" %}

Наследование шаблонов
--------------------

Самая мощная часть Twig - наследование шаблонов. Наследование шаблонов позволяет задать скелет вашего шаблона, а затем переопределить некоторые блоки.

Звучит сложно, однако стоит просто попробовать.

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

Здесь используется тег :doc:`extends<tags/extends>`. Он сообщает шаблонизатору, что этот шаблон наследуется от другово. *Тег ``extends`` должен быть первым в шаблоне*

Следует учесть, что так как блок ``footer`` не определен, то используется родительский

Возможно отображать значение родительского блока, используя функцию :doc:`parent<functions/parent>`:

.. code-block:: jinja

    {% block sidebar %}
        <h3>Оглавление</h3>
        {{ parent() }}
    {% endblock %}

.. Дополнительно::

    Подробнее про ``extends`` :doc:`extends<tags/extends>`. Описанны интересные возможности использования блоков, такие как динамическое наследование, условное наследование, вложенность и сферы применения.

.. Замечание::

    Twig также поддерживает множественное наследование с использованием тега :doc:`use<tags/use>` tag. Это продвинутая возможность и врятли она понадобится в простых шаблонах.

Экранирование
-------------

Генерируя HTML всегда есть возможность вывести специальные символы, ломающие логику HTML. Здесь есть два варианта - принудительное экранирование, или установленное по умолчанию для всех переменных.

Работа с принудительным экранированием
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If manual escaping is enabled, it is **your** responsibility to escape
variables if needed. What to escape? Any variable you don't trust.

Escaping works by piping the variable through the
:doc:`escape<filters/escape>` or ``e`` filter:

.. code-block:: jinja

    {{ user.username|e }}

By default, the ``escape`` filter uses the ``html`` strategy, but depending on
the escaping context, you might want to explicitly use any other available
strategies:

.. code-block:: jinja

    {{ user.username|e('js') }}
    {{ user.username|e('css') }}
    {{ user.username|e('url') }}
    {{ user.username|e('html_attr') }}

Working with Automatic Escaping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whether automatic escaping is enabled or not, you can mark a section of a
template to be escaped or not by using the :doc:`autoescape<tags/autoescape>`
tag:

.. code-block:: jinja

    {% autoescape %}
        Everything will be automatically escaped in this block (using the HTML strategy)
    {% endautoescape %}

By default, auto-escaping uses the ``html`` escaping strategy. If you output
variables in other contexts, you need to explicitly escape them with the
appropriate escaping strategy:

.. code-block:: jinja

    {% autoescape 'js' %}
        Everything will be automatically escaped in this block (using the JS strategy)
    {% endautoescape %}

Escaping
--------

It is sometimes desirable or even necessary to have Twig ignore parts it would
otherwise handle as variables or blocks. For example if the default syntax is
used and you want to use ``{{`` as raw string in the template and not start a
variable you have to use a trick.

The easiest way is to output the variable delimiter (``{{``) by using a variable
expression:

.. code-block:: jinja

    {{ '{{' }}

For bigger sections it makes sense to mark a block
:doc:`verbatim<tags/verbatim>`.

Macros
------

.. versionadded:: 1.12
    Support for default argument values was added in Twig 1.12.

Macros are comparable with functions in regular programming languages. They
are useful to reuse often used HTML fragments to not repeat yourself.

A macro is defined via the :doc:`macro<tags/macro>` tag. Here is a small example
(subsequently called ``forms.html``) of a macro that renders a form element:

.. code-block:: jinja

    {% macro input(name, value, type, size) %}
        <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
    {% endmacro %}

Macros can be defined in any template, and need to be "imported" via the
:doc:`import<tags/import>` tag before being used:

.. code-block:: jinja

    {% import "forms.html" as forms %}

    <p>{{ forms.input('username') }}</p>

Alternatively, you can import individual macro names from a template into the
current namespace via the :doc:`from<tags/from>` tag and optionally alias them:

.. code-block:: jinja

    {% from 'forms.html' import input as input_field %}

    <dl>
        <dt>Username</dt>
        <dd>{{ input_field('username') }}</dd>
        <dt>Password</dt>
        <dd>{{ input_field('password', '', 'password') }}</dd>
    </dl>

A default value can also be defined for macro arguments when not provided in a
macro call:

.. code-block:: jinja

    {% macro input(name, value = "", type = "text", size = 20) %}
        <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}" size="{{ size }}" />
    {% endmacro %}

Expressions
-----------

Twig allows expressions everywhere. These work very similar to regular PHP and
even if you're not working with PHP you should feel comfortable with it.

.. Замечание::
.. note::

    The operator precedence is as follows, with the lowest-precedence
    operators listed first: ``b-and``, ``b-xor``, ``b-or``, ``or``, ``and``,
    ``==``, ``!=``, ``<``, ``>``, ``>=``, ``<=``, ``in``, ``..``, ``+``,
    ``-``, ``~``, ``*``, ``/``, ``//``, ``%``, ``is``, and ``**``.

Literals
~~~~~~~~

.. versionadded:: 1.5
    Support for hash keys as names and expressions was added in Twig 1.5.

The simplest form of expressions are literals. Literals are representations
for PHP types such as strings, numbers, and arrays. The following literals
exist:

* ``"Hello World"``: Everything between two double or single quotes is a
  string. They are useful whenever you need a string in the template (for
  example as arguments to function calls, filters or just to extend or include
  a template). A string can contain a delimiter if it is preceded by a
  backslash (``\``) -- like in ``'It\'s good'``.

* ``42`` / ``42.23``: Integers and floating point numbers are created by just
  writing the number down. If a dot is present the number is a float,
  otherwise an integer.

* ``["foo", "bar"]``: Arrays are defined by a sequence of expressions
  separated by a comma (``,``) and wrapped with squared brackets (``[]``).

* ``{"foo": "bar"}``: Hashes are defined by a list of keys and values
  separated by a comma (``,``) and wrapped with curly braces (``{}``):

  .. code-block:: jinja

    {# keys as string #}
    { 'foo': 'foo', 'bar': 'bar' }

    {# keys as names (equivalent to the previous hash) -- as of Twig 1.5 #}
    { foo: 'foo', bar: 'bar' }

    {# keys as integer #}
    { 2: 'foo', 4: 'bar' }

    {# keys as expressions (the expression must be enclosed into parentheses) -- as of Twig 1.5 #}
    { (1 + 1): 'foo', (a ~ 'b'): 'bar' }

* ``true`` / ``false``: ``true`` represents the true value, ``false``
  represents the false value.

* ``null``: ``null`` represents no specific value. This is the value returned
  when a variable does not exist. ``none`` is an alias for ``null``.

Arrays and hashes can be nested:

.. code-block:: jinja

    {% set foo = [1, {"foo": "bar"}] %}

.. tip::

    Using double-quoted or single-quoted strings has no impact on performance
    but string interpolation is only supported in double-quoted strings.

Math
~~~~

Twig allows you to calculate with values. This is rarely useful in templates
but exists for completeness' sake. The following operators are supported:

* ``+``: Adds two objects together (the operands are casted to numbers). ``{{
  1 + 1 }}`` is ``2``.

* ``-``: Subtracts the second number from the first one. ``{{ 3 - 2 }}`` is
  ``1``.

* ``/``: Divides two numbers. The returned value will be a floating point
  number. ``{{ 1 / 2 }}`` is ``{{ 0.5 }}``.

* ``%``: Calculates the remainder of an integer division. ``{{ 11 % 7 }}`` is
  ``4``.

* ``//``: Divides two numbers and returns the truncated integer result. ``{{
  20 // 7 }}`` is ``2``.

* ``*``: Multiplies the left operand with the right one. ``{{ 2 * 2 }}`` would
  return ``4``.

* ``**``: Raises the left operand to the power of the right operand. ``{{ 2 **
  3 }}`` would return ``8``.

Logic
~~~~~

You can combine multiple expressions with the following operators:

* ``and``: Returns true if the left and the right operands are both true.

* ``or``: Returns true if the left or the right operand is true.

* ``not``: Negates a statement.

* ``(expr)``: Groups an expression.

.. Замечание::
.. note::

    Twig also support bitwise operators (``b-and``, ``b-xor``, and ``b-or``).

Comparisons
~~~~~~~~~~~

The following comparison operators are supported in any expression: ``==``,
``!=``, ``<``, ``>``, ``>=``, and ``<=``.

Containment Operator
~~~~~~~~~~~~~~~~~~~~

The ``in`` operator performs containment test.

It returns ``true`` if the left operand is contained in the right:

.. code-block:: jinja

    {# returns true #}

    {{ 1 in [1, 2, 3] }}

    {{ 'cd' in 'abcde' }}

.. tip::

    You can use this filter to perform a containment test on strings, arrays,
    or objects implementing the ``Traversable`` interface.

To perform a negative test, use the ``not in`` operator:

.. code-block:: jinja

    {% if 1 not in [1, 2, 3] %}

    {# is equivalent to #}
    {% if not (1 in [1, 2, 3]) %}

Test Operator
~~~~~~~~~~~~~

The ``is`` operator performs tests. Tests can be used to test a variable against
a common expression. The right operand is name of the test:

.. code-block:: jinja

    {# find out if a variable is odd #}

    {{ name is odd }}

Tests can accept arguments too:

.. code-block:: jinja

    {% if loop.index is divisibleby(3) %}

Tests can be negated by using the ``is not`` operator:

.. code-block:: jinja

    {% if loop.index is not divisibleby(3) %}

    {# is equivalent to #}
    {% if not (loop.index is divisibleby(3)) %}

Go to the :doc:`tests<tests/index>` page to learn more about the built-in
tests.

Other Operators
~~~~~~~~~~~~~~~

.. versionadded:: 1.12.0
    Support for the extended ternary operator was added in Twig 1.12.0.

The following operators are very useful but don't fit into any of the other
categories:

* ``..``: Creates a sequence based on the operand before and after the
  operator (this is just syntactic sugar for the :doc:`range<functions/range>`
  function).

* ``|``: Applies a filter.

* ``~``: Converts all operands into strings and concatenates them. ``{{ "Hello
  " ~ name ~ "!" }}`` would return (assuming ``name`` is ``'John'``) ``Hello
  John!``.

* ``.``, ``[]``: Gets an attribute of an object.

* ``?:``: The ternary operator:

  .. code-block:: jinja

      {{ foo ? 'yes' : 'no' }}

      {# as of Twig 1.12.0 #}
      {{ foo ?: 'no' }} == {{ foo ? foo : 'no' }}
      {{ foo ? 'yes' }} == {{ foo ? 'yes' : '' }}

String Interpolation
~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 1.5
    String interpolation was added in Twig 1.5.

String interpolation (`#{expression}`) allows any valid expression to appear
within a *double-quoted string*. The result of evaluating that expression is
inserted into the string:

.. code-block:: jinja

    {{ "foo #{bar} baz" }}
    {{ "foo #{1 + 2} baz" }}

Whitespace Control
------------------

.. versionadded:: 1.1
    Tag level whitespace control was added in Twig 1.1.

The first newline after a template tag is removed automatically (like in PHP.)
Whitespace is not further modified by the template engine, so each whitespace
(spaces, tabs, newlines etc.) is returned unchanged.

Use the ``spaceless`` tag to remove whitespace *between HTML tags*:

.. code-block:: jinja

    {% spaceless %}
        <div>
            <strong>foo</strong>
        </div>
    {% endspaceless %}

    {# output will be <div><strong>foo</strong></div> #}

In addition to the spaceless tag you can also control whitespace on a per tag
level. By using the whitespace control modifier on your tags, you can trim
leading and or trailing whitespace:

.. code-block:: jinja

    {% set value = 'no spaces' %}
    {#- No leading/trailing whitespace -#}
    {%- if true -%}
        {{- value -}}
    {%- endif -%}

    {# output 'no spaces' #}

The above sample shows the default whitespace control modifier, and how you can
use it to remove whitespace around tags.  Trimming space will consume all whitespace
for that side of the tag.  It is possible to use whitespace trimming on one side
of a tag:

.. code-block:: jinja

    {% set value = 'no spaces' %}
    <li>    {{- value }}    </li>

    {# outputs '<li>no spaces    </li>' #}

Extensions
----------

Twig can be easily extended.

If you are looking for new tags, filters, or functions, have a look at the Twig official
`extension repository`_.

If you want to create your own, read the :ref:`Creating an
Extension<creating_extensions>` chapter.

.. _`Twig bundle`:                https://github.com/Anomareh/PHP-Twig.tmbundle
.. _`Jinja syntax plugin`:        http://jinja.pocoo.org/2/documentation/integration
.. _`Twig syntax plugin`:         http://plugins.netbeans.org/plugin/37069/php-twig
.. _`Twig plugin`:                https://github.com/pulse00/Twig-Eclipse-Plugin
.. _`Twig language definition`:   https://github.com/gabrielcorpse/gedit-twig-template-language
.. _`extension repository`:       http://github.com/fabpot/Twig-extensions
.. _`Twig syntax mode`:           https://github.com/bobthecow/Twig-HTML.mode
.. _`other Twig syntax mode`:     https://github.com/muxx/Twig-HTML.mode
.. _`Notepad++ Twig Highlighter`: https://github.com/Banane9/notepadplusplus-twig
.. _`web-mode.el`:                http://web-mode.org/
