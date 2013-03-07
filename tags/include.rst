``include``
===========

Тег ``include`` подключает другой шаблон. Фактически он заменяет себя обработанным содержимым подключенного шаблона:

.. code-block:: jinja

    {% include 'header.html' %}
        Сверху меня шапка, а снизу подвал
    {% include 'footer.html' %}

Подключенные шаблоны имеют доступ к переменным "подключателя".

Если вы используйте автозагрузчик, то поиск шаблона на подключения будет производится в заданных ему дерикториях.

При подключении с помощью ключа ``with`` вы можете указать переменные специально для подключенного шаблона:

.. code-block:: jinja
    {# template.html получит доступ к переменным "подключателя" и дополнительно объявленным после ключа ``with`` #}
    {% include 'template.html' with {'foo': 'bar'} %}

    {% set vars = {'foo': 'bar'} %}
    {% include 'template.html' with vars %}

Вы можете ограничить доступ к переменным "подключателя" используя ключ ``only``:

.. code-block:: jinja

    {# template.html будет доступна только переменная ``foo`` #}
    {% include 'template.html' with {'foo': 'bar'} only %}

.. code-block:: jinja

    {# template.html не имеет доступа к переменным "подключателя" #}
    {% include 'template.html' only %}

Примечание:

    Когда подключаемый шаблон создан конечным пользователям вам следует

    When including a template created by an end user, you should consider
    sandboxing it. More information in the :doc:`Twig for Developers<../api>`
    chapter and in the :doc:`sandbox<../tags/sandbox>` tag documentation.

The template name can be any valid Twig expression:

.. code-block:: jinja

    {% include some_var %}
    {% include ajax ? 'ajax.html' : 'not_ajax.html' %}

And if the expression evaluates to a ``Twig_Template`` object, Twig will use it
directly::

    // {% include template %}

    $template = $twig->loadTemplate('some_template.twig');

    $twig->loadTemplate('template.twig')->display(array('template' => $template));

Версия:: 1.2
    The ``ignore missing`` feature has been added in Twig 1.2.

You can mark an include with ``ignore missing`` in which case Twig will ignore
the statement if the template to be ignored does not exist. It has to be
placed just after the template name. Here some valid examples:

.. code-block:: jinja

    {% include 'sidebar.html' ignore missing %}
    {% include 'sidebar.html' ignore missing with {'foo': 'bar'} %}
    {% include 'sidebar.html' ignore missing only %}

Версия:: 1.2
    The possibility to pass an array of templates has been added in Twig 1.2.

You can also provide a list of templates that are checked for existence before
inclusion. The first template that exists will be included:

.. code-block:: jinja

    {% include ['page_detailed.html', 'page.html'] %}

If ``ignore missing`` is given, it will fall back to rendering nothing if none
of the templates exist, otherwise it will throw an exception.
