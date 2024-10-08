:image: https://raw.githubusercontent.com/Tobi-De/falco/main/assets/og-image.jpg
:description: Virtualenv and dependencies management with a project generated with falco.

Virtualenv and dependencies
===========================

This is mainly handled using ``hatch``, ``hatch-pip-compile``, and the ``pyproject.toml`` file.

The pyproject.toml File
-----------------------

The ``pyproject.toml`` file is a Python standard introduced to unify and simplify Python project packaging and configurations. It was introduced by `PEP 518 <https://www.python.org/dev/peps/pep-0518/>`_ and `PEP 621 <https://www.python.org/dev/peps/pep-0621/>`_.
For more details, check out the `complete specifications <https://packaging.python.org/en/latest/specifications/pyproject-toml/#pyproject-toml-spec>`_.
Many tools in the Python ecosystem, including hatch, support it, and it seems that this is what the Python ecosystem has settled on for the future.

Hatch
-----

The project is set up to use hatch_ for virtual environment management and dependencies management.

   "Hatch is a modern, extensible Python project manager."

   -- Official hatch documentation


.. admonition:: why hatch?
   :class: dropdown note

   Using hatch is a recent switch for me. Previously, I used `poetry <https://python-poetry.org/>`_ as my preferred tool. While poetry is still a great tool, I have chosen hatch for the following reasons:

   1. Backed by the **pypa** (Python Packaging Authority), hatch aligns with the efforts to solve packaging and tooling issues in the Python ecosystem. I believe that if the Python ecosystem ever manages to overcome these challenges, it will be because the pypa has reached a consensus, and I hope that hatch will be the chosen solution. We all hope to see a cargo-like tool for Python someday.

   2. Hatch now has the ability to install and manage Python versions, along with other existing features. This brings it closer to being the all-in-one tool that every Python developer needs.

   3. Hatch is PEP-friendly, making it compatible with other tools in the ecosystem. It adds minimal custom configuration to the ``pyproject.toml`` file and relies on existing standards for project information and dependencies.

   4. In terms of performance, hatch is faster compared to poetry. While poetry is generally not slow, there have been rare instances where it took 30 minutes to install requirements. I have experienced this a few times.


Read the hatch documentation on `environment <https://hatch.pypa.io/latest/environment/>`_ for more information on how to manage virtual environments.
Hatch can do a lot, including `managing Python installations <https://hatch.pypa.io/latest/cli/reference/#hatch-python>`_, but for the context of the project, these are the things you need to know.

.. admonition:: Specify a Python Version
   :class: dropdown note

   If you have multiple Python interpreter versions installed on your computer, you can specify the specific version you want to use for a project
   by setting the ``python`` option in your default environment. Every other environment inherits from the default, so they will use the same version.

   .. code-block:: toml
      :caption: pyproject.toml

      [tool.hatch.envs.default]
      python = "3.12"
      ...

   More information on this can be found `here <https://hatch.pypa.io/latest/plugins/environment/virtual/#pyprojecttoml>`_.


Environments
************

The project comes with three environment configurations: ``default``, ``dev``, and ``docs``.

- The ``default`` environment is activated when you run ``hatch shell``. It is the default environment made available by Hatch and contains all the required dependencies for your project to run in production.
- The ``dev`` environment contains packages used for development and testing, such as ``django-debug-toolbar``, ``pytest``, ``django-pytest``, etc.
- The ``docs`` environment is for documentation. It contains tools such as ``sphinx``, ``furo``, etc.

.. admonition:: Install all dependencies
   :class: dropdown note

   Running ``just bootstrap`` will create all three environments and install the dependencies for each.

Although ``hatch`` comes with an integrated script runner, the project uses `just <https://just.systems/>`_ as the script runner. The main reason is that it is a more universal solution (not limited to the Python ecosystem) and 
I find it more flexible. Paired with the `scripts-to-rule-them-all <https://github.com/github/scripts-to-rule-them-all>`_ pattern, it's an efficient way to standardize a set of commands 
(``setup``, ``server``, ``console``, etc.) across all your projects, whether they are Django, Python, or something else. This way, you don't need to remember a different set of commands for each project.

To see all available scripts or `recipes` as ``just`` calls them, you can run:

.. code-block:: bash

   $ just

The primary environment you'll use during development is the ``dev`` environment. To run a command, you can either run ``just run`` or ``hatch --env dev run``. The first command is basically an alias for the second.

**Examples**

.. code-block:: bash

   $ just run python # launch the Python shell
   $ just run python manage.py dbshell # launch the database shell

There are aliases for most Django commands, such as ``just server`` to run the development server, ``just migrate`` to apply migrations, ``just createsuperuser`` to create a superuser, etc. 
. For any other commands that aren't explicitly aliased, you can run ``just dj <command>`` to run the command in the Django context.

Activate the virtual environment
********************************

To activate an environment for the current shell, run ``hatch shell <env_name>``, so ``hatch shell dev`` will activate the ``dev`` environment. If no specific environment name is provided, the default environment is activated.

.. admonition:: Get the path of the dev environment
   :class: dropdown note

   You can get the full path of the dev environment with ``just env-path`` or ``hatch env-path dev``. This can be useful to specify the interpreter in VSCode or PyCharm, for example.

You don't need to activate your shell to run commands. When running a just script, dependencies will be automatically synced (installed or removed if necessary), since it uses Hatch underneath, and 
the command will be executed in the appropriate virtual environment.


Add / remove a new dependency
*****************************

The default virtual environment includes all the dependencies specified in the ``[project.dependencies]`` section of the ``pyproject.toml`` file. To add a new dependency to your project, simply edit the ``pyproject.toml`` file and add it to the ``[project.dependencies]`` section. 
The next time you run a command using ``just``, such as ``just server``, Hatch (used underneath by the just script) will automatically install the new dependency. The process is the same for removing a dependency.

.. admonition:: Install a specific version of a package
   :class: dropdown note

   You can also explicitly install a dependency after adding or removing it by running ``just install``.

For development, I think this workflow should work quite well. Now, what happens when you need to deploy your app? You could install Hatch on the deployment target machine, but I prefer having a ``requirements.txt`` file that I can use to install dependencies on the deployment machine. That's where ``hatch-pip-compile`` comes in.


hatch-pip-compile
-----------------

The `hatch-pip-compile <https://github.com/juftin/hatch-pip-compile>`_ plugin is used with hatch to automatically generate a
requirements file (lock file) using `pip-tools <https://github.com/jazzband/pip-tools>`_. This file contains the dependencies of your hatch virtual environment with pinned versions.
The default setup generates a ``requirements.txt`` file that can be used for installing dependencies during deployment, as shown in the provided Dockerfile. However, you can customize the plugin to save
locks for all your environments. Refer to the `hatch-pip-compile documentation <https://github.com/juftin/hatch-pip-compile>`_ for more details.

Here is the current configuration in the ``pyproject.toml`` file relevant to hatch-pip-compile:

.. code-block:: toml
   :caption: pyproject.toml

   [tool.hatch.env]
   requires = [
      "hatch-pip-compile>=1.11.2"
   ]

   [tool.hatch.envs.default]
   type = "pip-compile"
   pip-compile-constraint = "default"
   pip-compile-installer = "uv"
   pip-compile-resolver = "uv"
   lock-filename = "requirements.txt"
   ...

Thanks to `hatch-pip-compile <https://juftin.com/hatch-pip-compile/>`_, we can try `uv <https://github.com/astral-sh/uv>`_, which is, and I quote:

   An extremely fast Python package installer and resolver, written in Rust. Designed as a drop-in replacement for pip and pip-compile

   -- Official github

Needless to say, it does make a noticeable difference in speed. If you encounter any issues with ``uv``, you can easily switch back to pip by updating the configurations as below:

.. code-block:: toml
   :caption: pyproject.toml

   [tool.hatch.envs.default]
   type = "pip-compile"
   pip-compile-constraint = "default"
   pip-compile-installer = "pip"
   pip-compile-resolver = "pip-compile"
   lock-filename = "requirements.txt"
   ...


.. _hatch: https://hatch.pypa.io/latest/