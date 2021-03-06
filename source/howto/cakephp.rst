.. |app| replace:: CakePHP
.. |mod| replace:: PHP 7.2+

#######
CakePHP
#######

To run apps based on the `CakePHP <https://cakephp.org>`_ framework using Unit:

#. .. include:: ../include/howto_install_unit.rst

#. `Install
   <https://book.cakephp.org/4/en/installation.html>`_ CakePHP and
   create or deploy your app.  Here, we use CakePHP's `basic template
   <https://book.cakephp.org/4/en/installation.html#create-a-cakephp-project>`_
   and Composer:

   .. code-block:: console

      $ cd /path/to/
      $ composer create-project --prefer-dist cakephp/app:4.* app

   This creates the app's directory tree at :file:`/path/to/app/`.  Its
   :file:`webroot/` subdirectory contains both the root :file:`index.php` and
   the static files; if your app requires additional :file:`.php` scripts, also
   store them here.

#. .. include:: ../include/howto_change_ownership.rst

#. Finally, prepare and upload the app :ref:`configuration <configuration-php>`
   to Unit (note the use of :samp:`uri`, :samp:`share`, and :samp:`fallback`):

   .. code-block:: json

      {
          "listeners": {
              "*:80": {
                  "pass": "routes/cakephp"
              }
          },

          "routes": {
              "cakephp": [
                  {
                      "match": {
                          ":nxt_term:`uri <Handles all direct script-based requests>`": [
                              "*.php",
                              "*.php/*"
                          ]
                      },

                      "action": {
                          "pass": "applications/cakephp/direct"
                      }
                  },
                  {
                      "action": {
                          ":nxt_term:`share <Serves all kinds of static files>`": ":nxt_term:`/path/to/app/webroot/ <Use a real path in your configuration>`",
                          ":nxt_term:`fallback <Uses the index.php at the root as the last resort>`": {
                              "pass": "applications/cakephp/index"
                          }
                      }
                  }
              ]
          },

          "applications": {
              "cakephp": {
                  "type": "php",
                  "user": ":nxt_term:`unit_user <User and group values must have access to the app root directory>`",
                  "group": "unit_group",
                  "targets": {
                      "direct": {
                          "root": ":nxt_term:`/path/to/app/webroot/ <Path to the webroot/ directory>`"
                      },

                      "index": {
                          "root": ":nxt_term:`/path/to/app/webroot/ <Path to the webroot/ directory>`",
                          "script": ":nxt_term:`index.php <All requests are handled by a single script>`"
                      }
                  }
              }
          }
      }

   .. note::

      The difference between the :samp:`pass` targets is their usage of the
      :samp:`script` :ref:`setting <configuration-php>`:

      - The :samp:`direct` target runs the :samp:`.php` script from the URI or
        defaults to :samp:`index.php` if the URI omits it.
      - The :samp:`index` target specifies the :samp:`script` that Unit runs
        for *any* URIs the target receives.

   For a detailed discussion, see `Fire It Up
   <https://book.cakephp.org/4/en/installation.html#fire-it-up>`_ in CakePHP
   docs.

#. .. include:: ../include/howto_upload_config.rst

   .. image:: ../images/cakephp.png
      :width: 100%
      :alt: CakePHP Basic Template App on Unit
