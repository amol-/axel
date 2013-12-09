=================================================
AXEL - The javascript loader that you control
=================================================

A common issue in modern javascript is loading external resources asynchronously,
many tools exist to perform such task in an efficient and reliable way, but most
of them have one common issue: They don't provide collision detection or
they require you to declare modules in a specific way according to AMD or similar solutions.

AXEL inverts the paradigm where files declare their module names, instead it's
the file user that declares the module name of each file and can mark names as
aliases to other modules. This makes possible to use plain javascript files
while detecting when you are loading the same module multiple times even from
different names or paths.

AXEL has born to satisfy a specific need: Improve the simple and small head.js
loader adding the ability to detect when you are importing the same library multiple times.

.. image:: https://raw.github.com/amol-/axel/master/docs/tests.png

Getting Started
================================

To get started with AXEL the only script you need to add to your ``<head>`` is AXEL itself::

    <script src="https://raw.github.com/amol-/axel/master/src/axel.js"></script>

Then you can load any other script directly from your body asynchronously::

    <script>
        axel.register('jquery', 'http://code.jquery.com/jquery-1.9.1.min.js').load('jquery', function() {
            alert('jQuery loaded!');
        });
    </script>

Give a look at the `example.html <https://raw.github.com/amol-/axel/master/docs/example.html>`_ file.

Loading Modules
================================

The first step you need to do to load a module is register its path::

    axel.register('jquery', 'http://code.jquery.com/jquery-1.9.1.min.js');

Then you can load it using::

    axel.load('jquery');

You can even perform a callback when the module has been loaded::

    axel.load('jquery', function() { alert('Hello!'); });

Or you can load multiple modules using::

    axel.load(['jquery', 'jquery.ui'], function() { alert('Both loaded!'); });

Collision Detection
===============================

If a module is registered multiple times it will be loaded only using the first
path registered, this permits to override the paths from which modules are loaded
by external libraries.

If you have a library that requires jQuery it is probably going to register it
and then load it, suppose our library looks for jQuery 1.8 we can upgrade it to
jQuery 1.9 by simply registering jQuery before loading our library::

    axel.register('jquery', 'http://code.jquery.com/jquery-1.9.1.min.js');

    //Third Party Library imports jquery which has already been registered
    axel.register('jquery', 'http://code.jquery.com/jquery-1.8.1.min.js');
    axel.load('jquery'); //jQuery 1.9 will be loaded.

Aliases
===============================

It might happen that libraries you are using are registering a dependency with
a name different from the one you are using. For example you might be loading
jQuery with the 'jquery' name while another library might be registering it wit
'jQuery' name. In such a case to perform collision detection and override
the file loaded by the library you can declare an alias::

    axel.register('jquery', 'http://code.jquery.com/jquery-1.9.1.min.js');
    axel.alias('jquery', 'jQuery');

    //Third Party Library imports jQuery which has been aliased to jquery
    axel.register('jQuery', 'http://code.jquery.com/jquery-1.8.1.min.js');
    axel.load('jQuery'); //jQuery 1.9 will be loaded.

Register module without path
================================

If third party libraries you depend on try to load a module that you don't use
with different names  you can simply register it without a path to tell
AXEL which is the root module that has to be loaded for both libraries instead
of loading it twice::

    //I don't know where jquery-1.9 is, but mark this as the module I want to use
    axel.register('jquery-1.9');
    //Whenever jquery-1.8 is requested use jquery-1.9 instead
    axel.alias('jquery-1.8', 'jquery-1.9');

    //Third Party Library 1 imports jquery
    axel.register('jquery-1.9', 'http://code.jquery.com/jquery-1.9.1.min.js');
    axel.load('jquery-1.9'); //Jquery 1.9 will be loaded

    //Third Party Library 2 imports jquery
    axel.register('jquery-1.8', 'http://code.jquery.com/jquery-1.8.1.min.js');
    axel.load('jquery-1.8'); //Jquery 1.9 will be used

Mixing axel loaded files with <script> loaded files
=====================================================

AXEL provides minimal support for registering files which are loaded using
a ``<script>`` tag instead of using axel itself. This can be useful if you have
some kind of automatic script injection provided by your web framework.

This can be achieved by registering a module as ``axel.Preloaded``::

    if (typeof jQuery === 'undefined') {
        //If jquery has not been already loaded, register it for loading
        axel.register('jquery', 'http://code.jquery.com/jquery.js');
    }
    else {
        //jQuery has already been loaded by a script tag in the head, skip loading
        axel.register('jquery', axel.Preloaded);
    }

    //Register bootstrap which dependens on jQuery
    axel.register('bootstrap', "/javascript/bootstrap.min.js");

    //Load both jquery and boostrap, loading jQuery will do nothing when marked as axel.Preloaded
    axel.load(['jquery', 'bootstrap']);

    //axel.ready will correctly fire both when jQuery was loaded or marked as Preloaded
    axel.ready('jquery', function() { alert('jQuery loaded!'); });

API Reference
=================================

AXEL provides various api, the core API are ``load``, ``register`` and ``alias``
but a few other exist that you might need to use:

    - ``register(name, [path])`` - Registers the given module for the provided path, if no path is provided it just marks it as the root of its aliases
    - ``load(name, [callback])`` - Loads the given module and call callback after it has been loaded
    - ``alias(name, othername)`` - Aliases a module to another one, every time one of the two names will be refered the root one will be used
    - ``ready(name, callback)`` - Whenever the module is loaded call the given callback, at least one path for the module has to be in place for this to work
    - ``resolve(name)`` - Resolves an alias to its root module
    - ``path(name)`` - Returns the root module file path for the given name
    - ``clear()`` - Erase all registered modules and aliases, mostly meant for testing.


