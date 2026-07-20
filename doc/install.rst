Installation
============

This section provides detailed instructions for installing nEdge Lite on your Linux system.

Installation from Packages
---------------------------

Pre-built packages for many Linux distributions are available in the repository.

1. Install the repository by following instructions at http://packages.ntop.org/
2. Install the **nedgelite** package.

License Installation
--------------------

nEdge Lite requires a valid license file to operate. The license file should be placed in one of these locations:

- ``nedgelite.license`` (current directory)
- ``/etc/nedgelite.license`` (system-wide)

License Management Commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   # Verify installation
   nedgelite --version

   # Display system ID for license generation
   nedgelite --show-system-id

   # Check license validity
   nedgelite --check-license

   # Check maintenance status
   nedgelite --check-maintenance

Contact ntop.org or visit http://shop.ntop.org/ to obtain a commercial license.

Post-Installation Steps
-----------------------

After installing nEdge Lite, perform these configuration steps:

1. **Verify Installation**

   .. code-block:: console

      nedgelite --version
      nedgelite --help

2. **Configure Netfilter**

   Set up iptables rules (see :doc:`netfilter_setup` for details):

   .. code-block:: console

      # For bridge mode
      sudo /usr/share/nedgelite/scripts/bridge_setup.sh lan0 wan0

      # For single interface mode
      sudo /usr/share/nedgelite/scripts/default_setup.sh eth0

3. **Install License**

   Copy your license file:

   .. code-block:: console

      sudo cp nedgelite.license /etc/nedgelite.license
      nedgelite --check-license

4. **Test Basic Operation**

   Run nEdge Lite with minimal configuration:

   .. code-block:: console

      sudo nedgelite -q 0 -v

   Press Ctrl+C to stop. You should see nEdge Lite start successfully.

Next Steps
----------

- **Quick Start**: Follow the :doc:`quick_start` guide for your first deployment
- **Netfilter Setup**: Configure netfilter rules in :doc:`netfilter_setup`
- **ntopng Integration**: Set up ntopng integration in :doc:`ntopng_integration`

Uninstallation
--------------

**Ubuntu/Debian:**

.. code-block:: console

   sudo apt-get remove nedgelite
   sudo apt-get purge nedgelite  # Also remove configuration files

**CentOS/Rocky:**

.. code-block:: console

   sudo yum remove nedgelite

Remove Configuration
~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo rm -rf /etc/nedgelite
   sudo rm -f /etc/nedgelite.license

