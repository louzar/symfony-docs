The Big Picture
===============


Downloading Symfony2
--------------------

First, check that you have installed and configured a Web server (such as
Apache) with PHP 5.3.3 or higher.


Start by downloading the Symfony2 Standard Edition: 

After unpacking the archive under your web server root directory, you should
have a ``Symfony/`` directory that looks like this:

.. code-block:: text

    www/ <- your web root directory
        Symfony/ <- the unpacked archive
            app/
                cache/
                config/
                logs/
                Resources/
            bin/
            src/
                Acme/
                    DemoBundle/
                        Controller/
                        Resources/
                        ...
            vendor/
                symfony/
                doctrine/
                ...
            web/
                app.php
                ...

.. note::

    If you are familiar with Composer, you can download Composer and then
    run the following command instead of downloading the archive:

    .. code-block:: bash

        $ php composer.phar create-project symfony/framework-standard-edition Symfony 2.4.*

.. _`quick-tour-big-picture-built-in-server`:



The URL to your application will be:
http://localhost/Symfony/web/app_dev.php.

.. image:: /images/quick_tour/welcome.png
   :align: center

Checking the Configuration
--------------------------

Symfony2 comes with a visual server configuration tester to help avoid some
headaches that come from Web server or PHP misconfiguration. Use the following
URL to see the diagnostics for your machine: http://localhost/Symfony/web/config.php



Understanding the Fundamentals
------------------------------

One of the main goals of a framework is to ensure Separation of Concerns.
This keeps your code organized and allows your application to evolve easily
over time by avoiding the mixing of database calls, HTML tags, and business
logic in the same script. To achieve this goal with Symfony, you'll first
need to learn a few fundamental concepts and terms.


Routing
~~~~~~~

Symfony2 routes the request to the code that handles it by trying to match the
requested URL, against some configured paths. By default,
these paths (called routes) are defined in the ``app/config/routing.yml`` configuration
file. When you're in the `dev`-
indicated by the app_**dev**.php front controller - the ``app/config/routing_dev.yml``
configuration file is also loaded. In the Standard Edition, the routes to
these "demo" pages are imported from this file:

.. code-block:: yaml

    # app/config/routing_dev.yml
    # ...

    # AcmeDemoBundle routes (to be removed)
    _acme_demo:
        resource: "@AcmeDemoBundle/Resources/config/routing.yml"

This imports a ``routing.yml`` file that lives inside the AcmeDemoBundle:

.. code-block:: yaml

    # src/Acme/DemoBundle/Resources/config/routing.yml
    _welcome:
        path:  /
        defaults: { _controller: AcmeDemoBundle:Welcome:index }

    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

    # ...

The first three lines (after the comment) define the code that is executed
when the user requests the "``/``" resource (i.e. the welcome page you saw
earlier). When requested, the ``AcmeDemoBundle:Welcome:index`` controller
will be executed. In the next section, you'll learn exactly what that means.


Controllers
~~~~~~~~~~~

A controller is a name for a PHP function or method that handles incoming
*requests* and returns *responses* (often HTML code). Instead of using the
PHP global variables and functions (like ``$_GET`` or ``header()``) to manage
these HTTP messages, Symfony uses objects: 
and :. The simplest possible
controller might create the response by hand, based on the request::

    use Symfony\Component\HttpFoundation\Response;

    $name = $request->query->get('name');

    return new Response('Hello '.$name, Response::HTTP_OK, array('Content-Type' => 'text/plain'));

.. versionadded:: 2.4
    Support for HTTP status code constants was added in Symfony 2.4.



Symfony2 chooses the controller based on the ``_controller`` value from the
routing configuration: ``AcmeDemoBundle:Welcome:index``. This string is the
controller *logical name*, and it references the ``indexAction`` method from
the ``Acme\DemoBundle\Controller\WelcomeController`` class::

    // src/Acme/DemoBundle/Controller/WelcomeController.php
    namespace Acme\DemoBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class WelcomeController extends Controller
    {
        public function indexAction()
        {
            return $this->render('AcmeDemoBundle:Welcome:index.html.twig');
        }
    }

.. tip::

    You could have used the full class and method name -
    ``Acme\DemoBundle\Controller\WelcomeController::indexAction`` - for the
    ``_controller`` value. But if you follow some simple conventions, the
    logical name is shorter and allows for more flexibility.

The ``WelcomeController`` class extends the built-in ``Controller`` class,
which provides useful shortcut methods, like the
 method that loads and renders
a template (``AcmeDemoBundle:Welcome:index.html.twig``). The returned value
is a Response object populated with the rendered content. So, if the need
arises, the Response can be tweaked before it is sent to the browser::

    public function indexAction()
    {
        $response = $this->render('AcmeDemoBundle:Welcome:index.txt.twig');
        $response->headers->set('Content-Type', 'text/plain');

        return $response;
    }

No matter how you do it, the end goal of your controller is always to return
the ``Response`` object that should be delivered back to the user. This ``Response``
object can be populated with HTML code, represent a client redirect, or even
return the contents of a JPG image with a ``Content-Type`` header of ``image/jpg``.


The template name, ``AcmeDemoBundle:Welcome:index.html.twig``, is the template
*logical name* and it references the ``Resources/views/Welcome/index.html.twig``
file inside the AcmeDemoBundle (located at ``src/Acme/DemoBundle``).
The `Bundles`_ section below will explain why this is useful.

Now, take a look at the routing configuration again and find the ``_demo``
key:

.. code-block:: yaml

    # src/Acme/DemoBundle/Resources/config/routing.yml
    # ...
    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo


    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    class DemoController extends Controller
    {
        /**
         * @Route("/hello/{name}", name="_demo_hello")
         * @Template()
         */
        public function helloAction($name)
        {
            return array('name' => $name);
        }

        // ...
    }

The ``@Route()`` annotation defines a new route with a path of
``/hello/{name}`` that executes the ``helloAction`` method when matched. A
string enclosed in curly brackets like ``{name}`` is called a placeholder. As
you can see, its value can be retrieved through the ``$name`` method argument.


If you take a closer look at the controller code, you can see that instead of
rendering a template and returning a ``Response`` object like before, it
just returns an array of parameters. The ``@Template()`` annotation tells
Symfony to render the template for you, passing in each variable of the array
to the template. The name of the template that's rendered follows the name
of the controller. So, in this example, the ``AcmeDemoBundle:Demo:hello.html.twig``
template is rendered (located at ``src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig``).



Templates
~~~~~~~~~

The controller renders the ``src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig``
template (or ``AcmeDemoBundle:Demo:hello.html.twig`` if you use the logical name):

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {% block title "Hello " ~ name %}

    {% block content %}
        <h1>Hello {{ name }}!</h1>
    {% endblock %}

By default, Symfony2 uses `Twig`_ as its template engine but you can also use
traditional PHP templates if you choose. The next chapter will introduce how
templates work in Symfony2.

Bundles
~~~~~~~

You might have wondered why the :term:`Bundle` word is used in many names you
have seen so far. All the code you write for your application is organized in
bundles. In Symfony2 speak, a bundle is a structured set of files (PHP files,
stylesheets, JavaScripts, images, ...) that implements a single feature (a
blog, a forum, ...) and which can be easily shared with other developers. As
of now, you have manipulated one bundle, AcmeDemoBundle. You will learn
more about bundles in the :doc:`last chapter of this tutorial`.

.. _quick-tour-big-picture-environments:

Working with Environments
-------------------------

Now that you have a better understanding of how Symfony2 works, take a closer
look at the bottom of any Symfony2 rendered page. You should notice a small
bar with the Symfony2 logo. This is the "Web Debug Toolbar", and it is a
Symfony2 developer's best friend!

.. image:: /images/quick_tour/web_debug_toolbar.png
   :align: center

But what you see initially is only the tip of the iceberg; click on the
hexadecimal number (the session token) to reveal yet another very useful
Symfony2 debugging tool: the profiler.

.. image:: /images/quick_tour/profiler.png
   :align: center

.. note::

    You can also get more information quickly by hovering over the items
    on the Web Debug Toolbar, or clicking them to go to their respective
    pages in the profiler.

When loaded and enabled (by default in the ``dev``,
the Profiler provides a web interface for a *huge* amount of information recorded
on each request, including logs, a timeline of the request, GET or POST parameters,
security details, database queries and more!

Of course, it would be unwise to have these tools enabled when you deploy
your application, so by default, the profiler is not enabled in the ``prod``
environment.

.. _quick-tour-big-picture-environments-intro:

