EnvVars
=======

``EnvVars`` is a class that represents an instance of environment variables for a given system.
It is obtained from the generic :ref:`Environment<conan_tools_env_environment_model>` class.

This class is used by other tools like the `conan.tools.gnu` autotools helpers and
the :ref:`VirtualBuildEnv<conan_tools_env_virtualbuildenv>` and :ref:`VirtualRunEnv<conan_tools_env_virtualrunenv>`
generator.


Creating environment files
++++++++++++++++++++++++++

``EnvVars`` object can generate environment files (shell, bat or powershell scripts):

.. code:: python

    def generate(self):
        env1 = Environment()
        env1.define("foo", "var")
        envvars = env1.vars(self)
        envvars.save_script("my_env_file")


Although it potentially could be used in other methods, this functionality is intended to work in the ``generate()``
method.

It will generate automatically a ``my_env_file.bat`` for Windows systems or ``my_env_file.sh`` otherwise.

It is possible to opt-in to generate PowerShell ``.ps1`` scripts instead of ``.bat`` ones, 
by using the configuration ``tools.env.virtualenv:powershell``. This configuration should 
be set with the value corresponding to the desired PowerShell executable: 
``powershell.exe`` for versions up to 5.1, and ``pwsh`` for PowerShell versions starting 
from 7. Note that setting ``tools.env.virtualenv:powershell`` to ``True`` or ``False`` is 
deprecated as of Conan 2.11.0.

You can also include additional arguments in the ``tools.env.virtualenv:powershell``
configuration. For example, you can set the value to ``powershell.exe -NoProfile`` or
``pwsh -NoProfile`` by including the arguments as part of the configuration value. These
arguments will be considered when executing the generated ``.ps1`` launchers.

Also, by default, Conan will automatically append that launcher file path to a list that will be used to
create a ``conanbuild.bat|sh|ps1`` file aggregating all the launchers in order. The ``conanbuild.sh|bat|ps1`` launcher
will be created after the execution of the ``generate()`` method.

The ``scope`` argument (``"build"`` by default) can be used to define different scope of environment files, to
aggregate them separately. For example, using a ``scope="run"``, like the ``VirtualRunEnv`` generator does, will
aggregate and create a ``conanrun.bat|sh|ps1`` script:

.. code:: python

    def generate(self):
        env1 = Environment()
        env1.define("foo", "var")
        envvars = env1.vars(self, scope="run")
        # Will append "my_env_file" to "conanrun.bat|sh|ps1"
        envvars.save_script("my_env_file")


You can also use ``scope=None`` argument to avoid appending the script to the aggregated ``conanbuild.bat|sh|ps1``:

.. code:: python

    env1 = Environment()
    env1.define("foo", "var")
    # Will not append "my_env_file" to "conanbuild.bat|sh|ps1"
    envvars = env1.vars(self, scope=None)
    envvars.save_script("my_env_file")


Running with environment files
++++++++++++++++++++++++++++++

The ``conanbuild.bat|sh|ps1`` launcher will be executed by default before calling every ``self.run()`` command. This
would be typically done in the ``build()`` method.

You can change the default launcher with the ``env`` argument of ``self.run()``:

.. code:: python

    ...
    def build(self):
        # This will automatically wrap the "foo" command with the correct environment:
        # source my_env_file.sh && foo
        # my_env_file.bat && foo
        # powershell my_env_file.ps1 ; cmd c/ foo
        self.run("foo", env=["my_env_file"])


Applying the environment variables
++++++++++++++++++++++++++++++++++

As an alternative to running a command, environments can be applied in the python environment:

.. code:: python

    from conan.tools.env import Environment

    env1 = Environment()
    env1.define("foo", "var")
    envvars = env1.vars(self)
    with envvars.apply():
       # Here os.getenv("foo") == "var"
       ...

Iterating the variables
+++++++++++++++++++++++

You can iterate the environment variables of an ``EnvVars`` object like this:

.. code:: python

    env1 = Environment()
    env1.append("foo", "var")
    env1.append("foo", "var2")
    envvars = env1.vars(self)
    for name, value in envvars.items():
        assert name == "foo":
        assert value == "var var2"


The current value of the environment variable in the system is replaced in the returned
value. This happens when variables are appended or prepended. If a placeholder is desired
instead of the actual value, it is possible to use the ``variable_reference`` argument
with a jinja template syntax, so a string with that resolved template will be returned
instead:

.. code:: python

    env1 = Environment()
    env1.append("foo", "var")
    envvars = env1.vars(self)
    for name, value in envvars.items(variable_reference="$penv{{{name}}}""):
        assert name == "foo":
        assert value == "$penv{{foo}} var"

.. warning::

    In Windows, there is a limit to the size of environment variables, a total of 32K for the whole environment,
    but specifically the PATH variable has a limit of 2048 characters. That means that the above utils could hit
    that limit, for example for large dependency graphs where all packages contribute to the PATH env-var.

    This can be mitigated by:

    - Putting the Conan cache closer to C:/ for shorter paths
    - Better definition of what dependencies can contribute to the PATH env-var
    - Other mechanisms for things like running with many shared libraries dependencies with too many .dlls, like ``deployers``


Reference
---------


.. currentmodule:: conan.tools.env.environment

.. autoclass:: EnvVars
    :members:
