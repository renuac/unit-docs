Prepare Unit configuration for your update.  To obtain an initial template,
query the control API:

.. code-block:: console

   # curl --unix-socket :nxt_term:`/path/to/control.unit.sock <Path to Unit control socket in your installation>` \
          :nxt_term:`http://localhost/config/ <Path to config section in Unit API>` > config.json

.. note::

   Control socket path may vary; run :command:`unitd --help` or see
   :ref:`installation-startup` for details.
