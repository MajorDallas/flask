Debugging Application Errors
============================


In Production
-------------

**Do not run the development server, or enable the built-in debugger, in
a production environment.** The debugger allows executing arbitrary
Python code from the browser. It's protected by a pin, but that should
not be relied on for security.

Use an error logging tool, such as Sentry, as described in
:ref:`error-logging-tools`, or enable logging and notifications as
described in :doc:`/logging`.

If you have access to the server, you could add some code to start an
external debugger if ``request.remote_addr`` matches your IP. Some IDE
debuggers also have a remote mode so breakpoints on the server can be
interacted with locally. Only enable a debugger temporarily.


The Built-In Debugger
---------------------

The built-in Werkzeug development server provides a debugger which shows
an interactive traceback in the browser when an unhandled error occurs
during a request. This debugger should only be used during development.

.. image:: _static/debugger.png
   :align: center
   :class: screenshot
   :alt: screenshot of debugger in action

.. warning::

    The debugger allows executing arbitrary Python code from the
    browser. It is protected by a pin, but still represents a major
    security risk. Do not run the development server or debugger in a
    production environment.

The debugger is enabled by default when the development server is run in debug mode.

.. code-block:: text

    $ flask --app hello run --debug

When running from Python code, passing ``debug=True`` enables debug mode, which is
mostly equivalent.

.. code-block:: python

    app.run(debug=True)

:doc:`/server` and :doc:`/cli` have more information about running the debugger and
debug mode. More information about the debugger can be found in the `Werkzeug
documentation <https://werkzeug.palletsprojects.com/debug/>`__.


External Debuggers
------------------

External debuggers, such as those provided by IDEs, can offer a more
powerful debugging experience than the built-in debugger. They can also
be used to step through code during a request before an error is raised,
or if no error is raised. Some even have a remote mode so you can debug
code running on another machine.

When using an external debugger, the app should still be in debug mode,
but it can be useful to disable the built-in debugger and reloader,
which can interfere.

.. code-block:: text

    $ flask --app hello run --debug --no-debugger --no-reload

When running from Python:

.. code-block:: python

    app.run(debug=True, use_debugger=False, use_reloader=False)

Disabling these isn't required, an external debugger will continue to
work with the following caveats. If the built-in debugger is not
disabled, it will catch unhandled exceptions before the external
debugger can. If the reloader is not disabled, it could cause an
unexpected reload if code changes during debugging.

With Flask's debugger disabled, Werkzeug's built-in exception handler
will catch any exceptions that your application does not. These will
NOT be propagated to your debugger, even if the ``PROPAGATE_EXCEPTIONS``
option is enabled.

If this becomes an obstacle, Werkzeug can be instructed to instead
propagate these exceptions by calling :meth:`Flask.run` with
``passthrough_errors=True``. Note: this will result in the development
server crashing as soon as your debugger allows execution to finish; for
this reason, it is recommended to set breakpoints over setting this option.

An example of setting this for `Debugpy <https://github.com/microsoft/debugpy>`__
running under `Vimspector <https://puremourning.github.io/vimspector/>`__:

.. code-block:: json

    {
        "flask": {
            "adapter": "debugpy",
            "default": true,
            "configuration": {
                "name": "Python Flask",
                "type": "python",
                "request": "launch",
                "python": "${workspaceRoot}/.venv/bin/python",
                "cwd": "${workspaceRoot}/",
                "code": "from my_app import app; app.run(passthrough_errors=True, debug=True, use_debugger=False, use_reloader=False, host='localhost', port=5000)",
                "env": {
                    "FLASK_ENV": "development"
                },
                "justMyCode": false
            }
        }
    }
