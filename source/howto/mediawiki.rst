.. |app| replace:: MediaWiki
.. |mod| replace:: PHP
.. |app-link| replace:: core files
.. _app-link: https://www.mediawiki.org/wiki/Download

#########
MediaWiki
#########

To run the `MediaWiki <https://www.mediawiki.org/>`_ collaboration and
documentation platform using Unit:

#. .. include:: ../include/howto_install_unit.rst

#. .. include:: ../include/howto_install_app.rst

#. .. include:: ../include/howto_change_ownership.rst

#. Next, :ref:`put together <configuration-php>` the |app| configuration for
   Unit:

   .. code-block:: json

      {
          "listeners": {
              "*:80": {
                  "pass": "routes/mediawiki"
              }

          },

          "routes": {
              "mediawiki": [
                  {
                      "match": {
                          ":nxt_term:`uri <Controls access to directories best kept private>`": [
                              "!/tests/qunit/*",
                              "/cache/*",
                              "/includes/*",
                              "/languages/*",
                              "/maintenance/*",
                              "/tests/*",
                              "/vendor/*"
                          ]
                      },

                      "action": {
                          "return": 403
                      }
                  },
                  {
                      "match": {
                          ":nxt_term:`uri <Enables access to application entry points>`": [
                              "/api.php*",
                              "/img_auth.php*",
                              "/index.php*",
                              "/load.php*",
                              "/mw-config/*.php",
                              "/opensearch_desc.php*",
                              "/profileinfo.php*",
                              "/rest.php*",
                              "/tests/qunit/*.php",
                              "/thumb.php*",
                              "/thumb_handler.php*"
                          ]
                      },

                      "action": {
                          "pass": "applications/mw/direct"
                      }
                  },
                  {
                      "match": {
                          ":nxt_term:`uri <Enables static access to specific content locations>`": [
                              "!*.php",
                              "!*.json",
                              "!*.:nxt_term:`htaccess <The negations deny access to file types listed here>`",
                              "/extensions/*",
                              "/images/*",
                              "/resources/assets/*",
                              "/resources/lib/*",
                              "/resources/src/*",
                              "/skins/*"
                          ]
                      },

                      "action": {
                          "share": "/path/to/app/"
                      }
                  },
                  {
                      "action": {
                          "pass": "applications/mw/index"
                      }
                  }
              ]
          },

          "applications": {
              "mw": {
                  "type": "php",
                  "user": ":nxt_term:`unit_user <User and group values must have access to target root directories>`",
                  "group": "unit_group",
                  "targets": {
                      "direct": {
                          "root": "/path/to/app/"
                      },

                      "index": {
                          "root": "/path/to/app/",
                          "script": "index.php"
                      }
                  }
              }
          }
      }

   .. note::

      The difference between the :samp:`pass` targets is their usage of the
      :samp:`script` :ref:`setting <configuration-php>`:

      - The :samp:`direct` target runs the :samp:`.php` script from the URI or
        defaults to :samp:`index.php` if the w omits it.
      - The :samp:`index` target specifies the :samp:`script` that Unit runs
        for *any* URIs the target receives.

#. .. include:: ../include/howto_upload_config.rst

#. Browse to http://localhost/mw-config/index.php and set |app| up using
   the settings noted earlier:

   .. image:: ../images/mw_install.png
      :width: 100%
      :alt: MediaWiki on Unit

   Download the newly generated :file:`LocalSettings.php` file and place it
   `appropriately <https://www.mediawiki.org/wiki/Manual:Config_script>`_:

   .. code-block:: console

      $ mv LocalSettings.php /path/to/app/
      $ chmod 600 /path/to/app/LocalSettings.php
      # chown unit_user /path/to/app/LocalSettings.php

#. After installation, add a match condition to the first step to disable
   access to the :file:`mw-config/` directory:

   .. code-block:: console

      # curl -X POST -d '"/mw-config/*"' --unix-socket \
             /path/to/control.unit.sock                \
             http://localhost/config/routes/mediawiki/0/match/uri/

            {
                "success": "Reconfiguration done."
            }

   Finally, your wiki is ready!

   .. image:: ../images/mw_ready.png
      :width: 100%
      :alt: MediaWiki on Unit
