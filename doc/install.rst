Installation
============

This section provides detailed instructions for installing nFW on your Linux system.

Installation from Packages
---------------------------

Pre-built packages for many Linux distributions are available in the repository.

Installation instructions can be found at http://packages.ntop.org/. 

License Installation
--------------------

nFW requires a valid license file to operate. The license file should be placed in one of these locations:

- ``nfw.license`` (current directory)
- ``/etc/nfw.license`` (system-wide)

License Management Commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   # Verify installation
   nfw --version

   # Display system ID for license generation
   nfw --show-system-id

   # Check license validity
   nfw --check-license

   # Check maintenance status
   nfw --check-maintenance

Contact ntop.org or visit http://shop.ntop.org/ to obtain a commercial license.

Post-Installation Steps
-----------------------

After installing nFW, perform these configuration steps:

1. **Verify Installation**

   .. code-block:: console

      nfw --version
      nfw --help

2. **Configure Netfilter**

   Set up iptables rules (see :doc:`netfilter_setup` for details):

   .. code-block:: console

      # For bridge mode
      sudo /usr/share/nfw/scripts/bridge_setup.sh lan0 wan0

      # For single interface mode
      sudo /usr/share/nfw/scripts/default_setup.sh eth0

3. **Install License**

   Copy your license file:

   .. code-block:: console

      sudo cp nfw.license /etc/nfw.license
      nfw --check-license

4. **Test Basic Operation**

   Run nFW with minimal configuration:

   .. code-block:: console

      sudo nfw -q 0 -v

   Press Ctrl+C to stop. You should see nFW start successfully.

Next Steps
----------

- **Quick Start**: Follow the :doc:`quick_start` guide for your first deployment
- **Netfilter Setup**: Configure netfilter rules in :doc:`netfilter_setup`
- **ntopng Integration**: Set up ntopng integration in :doc:`ntopng_integration`

Uninstallation
--------------

**Ubuntu/Debian:**

.. code-block:: console

   sudo apt-get remove nfw
   sudo apt-get purge nfw  # Also remove configuration files

**CentOS/Rocky:**

.. code-block:: console

   sudo yum remove nfw

Remove Configuration
~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo rm -rf /etc/nfw
   sudo rm -f /etc/nfw.license

