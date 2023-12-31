Development and Testing
:::::::::::::::::::::::

How to work on and test Factory Djoy, across different versions of Django.


Quick start
-----------

The test framework is already in place and ``tox`` is configured to make use of
it. Therefore the quickest way to get going is to make use of it.

Clone the repository and drop in:

.. code-block:: sh

    git clone git@github.com:jamescooke/factory_djoy.git
    cd factory_djoy

Create a virtual environment. This uses a bare call to ``virtualenv``, you
might prefer to use ``workon``:

.. code-block:: sh

    make venv

Activate the virtual environment and install requirements:

.. code-block:: sh

    . venv/bin/activate
    make install

Run all tests and linting using ``tox`` for all versions of Python and Django:

.. code-block:: sh

    make test

Requirements
------------

Quick info on packages:

* Only requirements for build are ``tox`` and ``tox-gh-actions``, these are
  unpinned and installed in ``build.yml`` GitHub Action config.

* For local development, there are base requirements in ``base.txt``. These are
  built with Python 3.7.

* For building documentation, there are docs requirements in ``docs.txt``.
  These are used by Read The Docs and build with Python 3.10.

Docs and base requirements are split because ``sphinx`` and ``flake8``
currently conflict.

Testing with real Django projects
---------------------------------

``factory_djoy`` is asserted to work with different vanilla versions of Django.
For each version of Django tested, a default project and app exist in the
``test_framework`` folder.

This structure has the following features:

* ``test_framework`` contains a folder for each version of Django. For example,
  the Django 1.11 project is in the ``test_framework/django111`` folder.

* Every project is created with the default ``django-admin startproject``
  command.

* In every project, a test settings file contains all the default settings as
  installed by Django, but adds the ``djoyapp`` app to the list of
  ``INSTALLED_APPS``.

* Every ``djoyapp`` contains a ``models.py`` and provides some models used for
  testing.

* Initial migrations for the models also exist, created using the matching
  version of Django using the default ``./manage.py makemigrations`` command.

* Every project has a ``tests`` folder wired into the Django project root.
  This contains the tests that assert that ``factory_djoy`` behaves as
  expected.

* Every project's tests are executed through the default ``./manage.py test``
  command.


Versioning notes
................

* ``tox`` is used to manage versions of Python and Django installed at test
  time.

* The latest point release from each major Django version is used, excluding
  versions that are no longer supported.


Creating Django test projects for Django version
................................................

In order to add a version of Django to the test run:

* Install the new version of Django into the current virtual environment:

  .. code-block:: sh

      pip install -U django

* Ask the new version of Django to create projects and all ``test_framework``
  structures:

  .. code-block:: sh

      cd test_framework
      make build

  Please note that creating a Django test project will fail if the target
  folder already exists. All ``django*`` folders can be removed with ``make
  clean`` - they can be rebuilt again identically with the ``build`` recipe.

* Add the new Django version to ``tox.ini``. (There's probably a better DRYer
  way to complete this.)

  ::

      django31: Django>=3.1,<3.2

* Remember to add the new Django version to the README and do a release.


Working locally
---------------

If there are multiple tests to run this can become inefficient with ``tox``.
Therefore, you can use the helper local environment configured inside
``test_framework``. This installs Python 3 and latest Django.

Create a new virtual environment in the ``test_framework`` folder and install
the requirements:

.. code-block:: sh

    cd test_framework
    make venv
    . venv/bin/activate
    make install

The test framework means that all the tests can be run on the test models and
factories using the standard ``manage.py`` test command. So, if working with
Django 1.10, after calling ``make build`` to create the app and folder
structure for that Django version, then all tests can be run with:

.. code-block:: sh

    make test


Release process
---------------

Decide the new version number. Semantic versioning is used and it will look
like ``1.2.3``.

* In a Pull Request for the release:

  * Update `CHANGELOG`_ with changes. Update links in footer.

  * Set version number in ``factory_djoy/__about__.py``

  * Ensure Pull Request is GREEN, then merge.

* With the newly merged master:

  * Run tests locally:

    .. code-block:: sh

        make test

  * Clean out any old distributions and make new ones:

    .. code-block:: sh

        make clean dist

  * Test upload with Test PyPI and follow it with an install direct from Test
    PyPI (might need to create a ``~/.pypirc`` file with settings for the test
    server:

    .. code-block:: sh

        make test-upload

        deactivate
        virtualenv /tmp/tenv --python=python3.8
        . /tmp/tenv/bin/activate
        make test-install

  * Tag release branch and push it:

    .. code-block:: sh

        git tag v1.2.3
        git push origin --tags

  * Upload to PyPI:

    .. code-block:: sh

        make upload

All done.

Post release:

* Ensure that link in `CHANGELOG`_ to the new diff works OK on GitHub.

* Check new docs are built on RTD.


Contributing
------------

Please ensure that any provided code:

* Has been developed with "test first" process.

* Can be auto-merged in GitHub.

* Passes `build on GitHub Actions <https://github.com/jamescooke/factory_djoy/actions>`_.


Helper `Makefile` recipes
-------------------------

The root ``Makefile`` has a couple of helper recipes (note this is different to
the ``Makefile`` in ``test_settings``):

* ``dist``: Creates the distribution files.

* ``dist-check``: Uses Twine to check the dist. Will be used to replace
  ``setup.py check``.

* ``upload``: Push generated distribution to PyPI.

* ``bump_reqs``: Update all packages, commit updates to a new
  ``auto/bump-requirements`` branch and push it to origin.

* ``clean``: Remove all compiled Python files, distributions, etc.


.. _CHANGELOG: https://github.com/jamescooke/factory_djoy/blob/master/CHANGELOG.rst
