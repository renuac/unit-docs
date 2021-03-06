######
Django
######

To run your Django projects and apps in Unit:

#. :ref:`Install Unit <installation-precomp-pkgs>` with the appropriate Python
   language module version.

#. If you haven't already done so, `create your Django project and apps
   <https://docs.djangoproject.com/en/stable/intro/overview/>`_ where you
   usually store them.

#. .. include:: ../include/get-config.rst

   This creates a JSON file with Unit's current settings; update it with your
   project settings as follows.

   Suppose you use a `basic directory structure
   <https://docs.djangoproject.com/en/stable/ref/django-admin/#django-admin-startproject>`_
   for your Django project:

   .. code-block:: none

      /home/django/project/
      |-- manage.py
      |-- app1/
      |   |-- ...
      |-- app2/
      |   |-- ...
      `-- project/
          |-- ...
          `-- wsgi.py

   Edit the JSON, adding a :ref:`listener <configuration-listeners>` entry to
   point to a Unit :ref:`app <configuration-applications>` with your
   *project*'s WSGI module; the project and its apps will run on the listener's
   IP and port.  If you use a `virtual environment
   <https://docs.djangoproject.com/en/stable/intro/contributing/#getting-a-copy-of-django-s-development-version>`_,
   reference it as :samp:`home`.  Finally, you can also set up some environment
   variables that your project relies on:

   .. code-block:: json

      {
          "listeners": {
              "127.0.0.1:8080": {
                  "pass": "applications/django_project"
              }
          },

          "applications": {
              "django_project": {
                  "type": "python 3",
                  "path": "/home/django/project/",
                  "home": ":nxt_term:`/home/django/venv/ <Virtual environment directory>`",
                  "module": "project.wsgi",
                  "environment": {
                      "DJANGO_SETTINGS_MODULE": "project.settings",
                      "DB_ENGINE": "django.db.backends.postgresql",
                      "DB_NAME": "project",
                      "DB_HOST": "127.0.0.1",
                      "DB_PORT": "5432"
                  }
              }
          }
      }

   .. note::

      Mind that Unit will look for an :samp:`application` callable in the WSGI
      module to run the entire project.

   Here, the top-level :file:`project` directory becomes :samp:`path`; its
   child :file:`project` and the :file:`wsgi.py` in it are `imported
   <https://docs.python.org/3/reference/import.html>`_ via :samp:`module`.  If
   you reorder your directories, :ref:`set up <configuration-python>`
   :samp:`path` and :samp:`module` accordingly.

#. If your project uses Django's `static files
   <https://docs.djangoproject.com/en/stable/howto/static-files/>`_, optionally
   add a :ref:`route <configuration-routes>` to :ref:`serve
   <configuration-static>` them with Unit:

   .. code-block:: json

      {
          "listeners": {
              "127.0.0.1:8080": {
                  "pass": "routes"
              }
          },

          "routes": [
              {
                  "match": {
                      "uri": "/static/*"
                  },

                  "action": {
                      ":nxt_term:`share <The resulting static asset path will be /home/django/static/>`": "/home/django/"
                  }
              },
              {
                  "action": {
                      "pass": "applications/django_project"
                  }
              }
          ],

          "applications": {
              "django_project": {
                  "type": "python 3",
                  "path": "/home/django/project/",
                  "home": ":nxt_term:`/home/django/venv/ <Virtual environment directory>`",
                  "module": "project.wsgi",
                  "environment": {
                      "DJANGO_SETTINGS_MODULE": "project.settings",
                      "DB_ENGINE": "django.db.backends.postgresql",
                      "DB_NAME": "project",
                      "DB_HOST": "127.0.0.1",
                      "DB_PORT": "5432"
                  }
              }
          }
      }

#. Upload the updated configuration:

   .. code-block:: console

      # curl -X PUT --data-binary @config.json --unix-socket \
             /path/to/control.unit.sock http://localhost/config

   After a successful update, your project and apps should be available
   on the listener's IP address and port:

   .. code-block:: console

      $ curl 127.0.0.1:8080/admin/
      $ curl 127.0.0.1:8080/app1/
