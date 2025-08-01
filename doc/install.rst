Installation
============

The nFW installation is pretty straightforward, just install the *nfw* package from the repository, then move to the configuration.

In order to run nFW the following major steps are needed:

1. Create a configuration file /etc/nfw/nfw.conf (copy /etc/nfw/nfw.conf.example)

2. Configure at least LAN and WAN interfaces in /etc/nfw/nfw.conf

3. Start the service

.. code-block:: console

   systemctl start nfw

