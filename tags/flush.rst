``flush``
=========

Версия:: 1.5
    Тег flush добавлен в Twig 1.5.

Тег ``flush`` очищает буфер вывода и отправляет содержимое пользователю:

.. code-block:: jinja

    {% flush %}

Примечание:

    На самом деле Twig использует PHP-функцию `flush`.

.. _`flush`: http://php.net/flush
