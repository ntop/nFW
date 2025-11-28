Quick Start Guide
=================

This guide will help you get nFW up and running quickly. We'll cover a basic deployment scenario with ntopng integration.

Prerequisites
-------------

Before starting, ensure you have:

1. **Installed nFW**: Follow the :doc:`install` guide if you haven't already
2. **Valid License**: Place your license file at ``/etc/nfw.license``
3. **Root Access**: All commands must be run as root or with sudo
4. **Network Interfaces**: At least one network interface for traffic inspection

Quick Setup (Single Interface)
-------------------------------

This is the simplest deployment for testing or protecting traffic on a single interface.

Step 1: Set Up Netfilter Rules
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Run the setup script for single interface mode:

.. code-block:: console

   sudo /usr/share/nfw/scripts/default_setup.sh eth0

Replace ``eth0`` with your actual interface name. This script configures iptables to route packets to NFQUEUE.

**What this does:**

- Configures iptables mangle table for CONNMARK save/restore
- Routes unmarked packets (mark=0) to NFQUEUE 0
- Drops packets marked with mark=2
- Allows packets marked with mark=1

Step 2: Start ntopng
~~~~~~~~~~~~~~~~~~~~~

On the same host (or a different one), start ntopng with ZMQ collector:

.. code-block:: console

   sudo ntopng -i tcp://127.0.0.1:1234c

**Important**: Note the trailing ``c`` in the endpoint URL. This tells ntopng to act as a ZMQ collector.

ntopng will listen on port 1234 for flow data from nFW.

Step 3: Start nFW
~~~~~~~~~~~~~~~~~~

Start nFW and connect it to ntopng:

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:1234 -v

**Command breakdown:**

- ``-q 0``: Use NFQUEUE ID 0
- ``-z tcp://127.0.0.1:1234``: Send flows to ntopng at this ZMQ endpoint
- ``-v``: Verbose logging

Step 4: Verify Operation
~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Check nFW is running:**

   You should see output indicating nFW has started and is processing packets.

2. **Generate some traffic:**

   .. code-block:: console

      ping 8.8.8.8
      curl http://www.example.com

3. **View flows in ntopng:**

   Open your browser and navigate to http://localhost:3000 (default ntopng web interface).
   You should see flows appearing in real-time.

4. **Check connection tracking:**

   .. code-block:: console

      sudo conntrack -L | head -20

   You should see connection marks being applied.

Quick Setup (Bridge Mode)
--------------------------

Bridge mode allows nFW to inspect traffic transparently between two network segments.

Step 1: Set Up Bridge
~~~~~~~~~~~~~~~~~~~~~~

Run the bridge setup script:

.. code-block:: console

   sudo /usr/share/nfw/scripts/bridge_setup.sh eth0 eth1

Replace ``eth0`` (LAN) and ``eth1`` (WAN) with your actual interface names.

**What this does:**

- Creates a bridge interface ``br0``
- Adds both interfaces to the bridge
- Configures iptables for bridge packet filtering
- Routes packets to NFQUEUE for inspection

Step 2: Start ntopng and nFW
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   # Terminal 1: Start ntopng
   sudo ntopng -i tcp://127.0.0.1:1234c

   # Terminal 2: Start nFW
   sudo nfw -q 0 -z tcp://127.0.0.1:1234 -v

Step 3: Test Connectivity
~~~~~~~~~~~~~~~~~~~~~~~~~~

From a device on the LAN side, test internet connectivity:

.. code-block:: console

   ping 8.8.8.8
   curl http://www.google.com

Traffic should flow through the bridge and be inspected by nFW.

Using Policy Files
------------------

Instead of relying on ntopng for policies, you can use a static JSON policy file.

Step 1: Create a Policy File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create ``/etc/nfw/policy.json``:

.. code-block:: json

   {
     "pool": {
       "id": 1,
       "name": "Default Pool",
       "ip": ["192.168.1.0/24"],
       "mac": []
     },
     "policy": {
       "id": 1
     }
   }
   {
     "policy": {
       "id": 1,
       "root": 0,
       "name": "Default Policy",
       "markers": {
         "protocols": {
           "Facebook": "drop",
           "YouTube": "drop",
           "BitTorrent": "drop"
         },
         "categories": {
           "SocialNetwork": "drop",
           "Streaming": "drop"
         },
         "countries": {},
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }

This policy blocks Facebook, YouTube, BitTorrent, and entire categories like Social Networks and Streaming.

Step 2: Start nFW with Policy File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo nfw -q 0 -r /etc/nfw/policy.json -z tcp://127.0.0.1:1234 -v

**Command breakdown:**

- ``-r /etc/nfw/policy.json``: Load policy rules from this file
- Other options remain the same

Step 3: Test Policy Enforcement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Try accessing Facebook:**

   .. code-block:: console

      curl https://www.facebook.com

   The connection should be blocked (or hang).

2. **Try accessing allowed sites:**

   .. code-block:: console

      curl https://www.wikipedia.org

   This should work normally.

3. **View blocked flows in ntopng:**

   Open ntopng and check for dropped flows.

Dynamic Policy Updates
----------------------

For dynamic policy management, use ntopng to send policy updates to nFW.

Step 1: Start ntopng with ZMQ Publisher
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo ntopng -i tcp://127.0.0.1:5556c --zmq-publish-events tcp://127.0.0.1:5557

**Parameters:**

- ``-i tcp://127.0.0.1:5556c``: ZMQ collector for flows (note the ``c``)
- ``--zmq-publish-events tcp://127.0.0.1:5557``: Publish policy events

Step 2: Start nFW with Policy Collector
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:5556 -p tcp://127.0.0.1:5557 -v

**Parameters:**

- ``-z tcp://127.0.0.1:5556``: Send flows to ntopng
- ``-p tcp://127.0.0.1:5557``: Subscribe to policy updates from ntopng

Step 3: Configure Policies in ntopng
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Open ntopng web interface: http://localhost:3000
2. Navigate to the Policies section
3. Create or modify policies
4. nFW will automatically receive and apply the updates

Reloading Policies
------------------

If using a static policy file (``-r`` option), you can reload the policy without restarting:

.. code-block:: console

   # Find the PID
   ps aux | grep nfw

   # Send SIGHUP
   sudo kill -HUP <PID>

nFW will reload the policy file and apply the new rules.

If you are configuring policies via ntopng instead, they are automatically reloaded when changing the policy in ntopng.

Common Use Cases
----------------

Block Social Media
~~~~~~~~~~~~~~~~~~

.. code-block:: json

   "markers": {
     "categories": {
       "SocialNetwork": "drop"
     }
   }

Block Gaming
~~~~~~~~~~~~

.. code-block:: json

   "markers": {
     "categories": {
       "Game": "drop"
     }
   }

Block Streaming Services
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: json

   "markers": {
     "protocols": {
       "Netflix": "drop",
       "YouTube": "drop",
       "Hulu": "drop",
       "Disney+": "drop"
     }
   }

Block P2P File Sharing
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: json

   "markers": {
     "protocols": {
       "BitTorrent": "drop",
       "eDonkey": "drop"
     }
   }

Block Traffic from Specific Countries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: json

   "markers": {
     "countries": {
       "CN": "drop",
       "RU": "drop",
       "KP": "drop"
     }
   }

Block Entire Continents
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: json

   "markers": {
     "continents": {
       "Asia": "drop",
       "Africa": "drop"
     }
   }

Next Steps
----------

Now that you have nFW running, explore these topics:

- **Netfilter Setup**: Learn more about :doc:`netfilter_setup` for advanced configurations
- **Configuration**: Explore all :doc:`configuration` options
- **Policies**: Deep dive into :doc:`policies` for sophisticated filtering
- **ntopng Integration**: Set up advanced :doc:`ntopng_integration`
- **Advanced Features**: Explore :doc:`advanced` features like multiple queues
- **Troubleshooting**: If you encounter issues, check :doc:`troubleshooting`
