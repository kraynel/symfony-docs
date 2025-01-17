.. index::
   single: Debug
   single: Error
   single: Exception
   single: Components; ErrorHandler

The ErrorHandler Component
==========================

    The ErrorHandler component provides tools to manage errors and ease debugging PHP code.

Installation
------------

.. code-block:: terminal

    $ composer require symfony/error-handler

.. include:: /components/require_autoload.rst.inc

Usage
-----

The ErrorHandler component provides several tools to help you debug PHP code.
Enable all of them by calling this method::

    use Symfony\Component\ErrorHandler\Debug;

    Debug::enable();

The :method:`Symfony\\Component\\ErrorHandler\\Debug::enable` method registers an
error handler, an exception handler and
:ref:`a special class loader <component-debug-class-loader>`.

Read the following sections for more information about the different available
tools.

.. caution::

    You should never enable the debug tools, except for the error handler, in a
    production environment as they might disclose sensitive information to the user.

Handling PHP Errors and Exceptions
----------------------------------

Enabling the Error Handler
~~~~~~~~~~~~~~~~~~~~~~~~~~

The :class:`Symfony\\Component\\ErrorHandler\\ErrorHandler` class catches PHP
errors and converts them to exceptions (of class :phpclass:`ErrorException` or
:class:`Symfony\\Component\\ErrorHandler\\Exception\\FatalErrorException` for
PHP fatal errors)::

    use Symfony\Component\ErrorHandler\ErrorHandler;

    ErrorHandler::register();

This error handler is enabled by default in the production environment when the
application uses the FrameworkBundle because it generates better error logs.

Enabling the Exception Handler
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The :class:`Symfony\\Component\\ErrorHandler\\ExceptionHandler` class catches
uncaught PHP exceptions and converts them to a nice PHP response. It is useful
in :ref:`debug mode <debug-mode>` to replace the default PHP/XDebug output with
something prettier and more useful::

    use Symfony\Component\ErrorHandler\ExceptionHandler;

    ExceptionHandler::register();

.. note::

    If the :doc:`HttpFoundation component </components/http_foundation>` is
    available, the handler uses a Symfony Response object; if not, it falls
    back to a regular PHP response.

Catches PHP errors and turn them into exceptions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Most PHP core functions were written before exception handling was introduced to the
language and most of this functions do not throw an exception on failure. Instead,
they return ``false`` in case of error.

Let's take the following code example::

    $data = json_decode(file_get_contents($filename), true);
    $data['read_at'] = date($datetimeFormat);
    file_put_contents($filename, json_encode($data));

All these functions ``file_get_contents``, ``json_decode``, ``date``, ``json_encode``
and ``file_put_contents`` will return ``false`` or ``null`` on error, having to
deal with those failures manually::

    $content = @file_get_contents($filename);
    if (false === $content) {
        throw new \RuntimeException('Could not load file.');
    }
    $data = @json_decode($content, true);
    if (null === $data) {
        throw new \RuntimeException('File does not contain valid JSON.');
    }
    $datetime = @date($datetimeFormat);
    if (false === $datetime) {
        throw new \RuntimeException('Invalid datetime format.');
    }
    // ...

.. note::

    Since PHP 7.3 `json_decode`_ function will accept a new ``JSON_THROW_ON_ERROR`` option
    that will let ``json_decode`` throw an exception instead of returning ``null`` on error.
    However, it is not enabled by default, so you will need to explicitly configure it.

To simplify this behavior the :class:`Symfony\\Component\\ErrorHandler\\ErrorHandler` class
provides a :method:`Symfony\\Component\\ErrorHandler\\ErrorHandler::call` method that will
automatically throw an exception when such a failure occurs. This method will accept a ``callable``
parameter and then the arguments needed to call it, returning back the result::

    $content = ErrorHandler::call('file_get_contents', $filename);

This way, you could use a ``\Closure`` function to wrap a portion of code and be sure that it
breaks even if the `@-silencing operator`_ is used::

    $data = ErrorHandler::call(static function () use ($filename, $datetimeFormat) {
        $data = json_decode(file_get_contents($filename), true);
        $data['read_at'] = date($datetimeFormat);
        file_put_contents($filename, json_encode($data));

        return $data;
    });

.. _component-debug-class-loader:

Debugging a Class Loader
------------------------

The :class:`Symfony\\Component\\ErrorHandler\\DebugClassLoader` attempts to
throw more helpful exceptions when a class isn't found by the registered
autoloaders. All autoloaders that implement a ``findFile()`` method are replaced
with a ``DebugClassLoader`` wrapper.

Using the ``DebugClassLoader`` is done by calling its static
:method:`Symfony\\Component\\ErrorHandler\\DebugClassLoader::enable` method::

    use Symfony\Component\ErrorHandler\DebugClassLoader;

    DebugClassLoader::enable();

.. _`@-silencing operator`: https://php.net/manual/en/function.json-decode.php
.. _`json_decode`: https://php.net/manual/en/language.operators.errorcontrol.php
