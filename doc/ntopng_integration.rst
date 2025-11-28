ntopng Integration
==================

nFW is designed to work seamlessly with `ntopng <https://www.ntop.org/products/traffic-analysis/ntop/>`_, providing centralized monitoring, visualization, and policy management. This section explains how to set up and optimize the integration.

Overview
--------

The nFW-ntopng integration provides:

- **Real-Time Flow Monitoring**: View all flows inspected by nFW in ntopng's web interface
- **Protocol Analytics**: See detailed statistics on detected protocols and applications
- **Dynamic Policy Management**: Configure and update policies through ntopng's GUI
- **Historical Data**: Store and query flow data over time
- **Alerting**: Set up alerts for policy violations or suspicious traffic
- **Multi-Instance Support**: Multiple nFW instances can report to a single ntopng

Integration Architecture
------------------------

Communication Channels
~~~~~~~~~~~~~~~~~~~~~~

nFW and ntopng communicate via two ZeroMQ channels:

1. **Flow Export Channel**: nFW sends flow data to ntopng (ZMQ PUB/SUB)
2. **Policy Update Channel**: ntopng sends policy updates to nFW (ZMQ PUB/SUB)

.. code-block:: text

   ┌─────────┐                    ┌─────────┐
   │   nFW   │──── Flows ────────>│  ntopng │
   │         │<─── Policies ──────│         │
   └─────────┘                    └─────────┘

ZeroMQ Endpoints
~~~~~~~~~~~~~~~~

- **ntopng as Collector**: Endpoint URL must end with ``c`` (e.g., ``tcp://127.0.0.1:1234c``)
- **nFW as Publisher**: Endpoint URL without ``c`` (e.g., ``tcp://127.0.0.1:1234``)

Basic Setup
-----------

Same-Host Deployment
~~~~~~~~~~~~~~~~~~~~

When nFW and ntopng run on the same machine:

**Start ntopng:**

.. code-block:: console

   sudo ntopng -i tcp://127.0.0.1:1234c

**Start nFW:**

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:1234

**Access ntopng Web Interface:**

Open http://localhost:3000 in your browser. You should see flows from nFW appearing in real-time.

Remote Deployment
~~~~~~~~~~~~~~~~~

When nFW and ntopng run on different machines:

**On ntopng host (192.168.1.10):**

.. code-block:: console

   sudo ntopng -i tcp://0.0.0.0:1234c

**On nFW host:**

.. code-block:: console

   sudo nfw -q 0 -z tcp://192.168.1.10:1234

**Firewall Configuration:**

Ensure TCP port 1234 is accessible from the nFW host to ntopng host.

Dynamic Policy Management
--------------------------

To enable bidirectional communication (flow export + policy updates):

Setup with Policy Updates
~~~~~~~~~~~~~~~~~~~~~~~~~~

**On ntopng host:**

.. code-block:: console

   sudo ntopng -i tcp://0.0.0.0:5556c --zmq-publish-events tcp://0.0.0.0:5557

**On nFW host:**

.. code-block:: console

   sudo nfw -q 0 -z tcp://ntopng-host:5556 -p tcp://ntopng-host:5557

**Configure Policies in ntopng:**

1. Open ntopng web interface
2. Navigate to **Settings** → **Policies**
3. Create or modify policies
4. Changes are automatically pushed to nFW

Managing Policies via ntopng
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Creating a Pool:**

1. Go to **Pools** → **Host Pools**
2. Click **Add Pool**
3. Define IP ranges and/or MAC addresses
4. Assign a policy to the pool

**Creating a Policy:**

1. Go to **Policies** → **Traffic Policies**
2. Click **Add Policy**
3. Configure protocol/category filters
4. Set default action (Pass/Drop)
5. Click **Save**

**Viewing Applied Policies:**

1. Go to **Flows** → **All Flows**
2. Look for the **Policy** column
3. Flows show which policy was applied

Multiple nFW Instances
-----------------------

Single ntopng, Multiple nFW Deployments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Deploy nFW on multiple hosts, all reporting to one ntopng:

**On ntopng host:**

.. code-block:: console

   sudo ntopng -i tcp://0.0.0.0:5556c --zmq-publish-events tcp://0.0.0.0:5557

**On nFW host 1:**

.. code-block:: console

   sudo nfw -q 0 -z tcp://ntopng-host:5556 -p tcp://ntopng-host:5557

**On nFW host 2:**

.. code-block:: console

   sudo nfw -q 0 -z tcp://ntopng-host:5556 -p tcp://ntopng-host:5557

**On nFW host 3:**

.. code-block:: console

   sudo nfw -q 0 -z tcp://ntopng-host:5556 -p tcp://ntopng-host:5557

All nFW instances will:

- Send flows to the same ntopng
- Receive policy updates from the same ntopng
- Appear as separate interfaces/collectors in ntopng

High Availability
~~~~~~~~~~~~~~~~~

For redundancy, send flows to multiple ntopng instances:

.. code-block:: console

   sudo nfw -q 0 \
     -z tcp://ntopng1:5556 \
     -z tcp://ntopng2:5556 \
     -p tcp://ntopng1:5557

Encrypted Communication
-----------------------

Secure flow export with ZeroMQ CURVE encryption.

Generate CURVE Keys
~~~~~~~~~~~~~~~~~~~~

On ntopng host, generate a key pair:

.. code-block:: console

   # Generate keys using zmq tools
   curve_keygen

This outputs:

.. code-block:: text

   public-key: Yne@$w-vo<fVvi]a<NY6T1ed:M$fCG*[IaLV{hID
   secret-key: D:)Q[IlAW!ahhC2ac:9*A}h:p?([4%wOTJ%JR%cs

Configure ntopng with Encryption
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo ntopng -i tcp://0.0.0.0:5556c \
     --zmq-encryption-key "Yne@$w-vo<fVvi]a<NY6T1ed:M$fCG*[IaLV{hID:D:)Q[IlAW!ahhC2ac:9*A}h:p?([4%wOTJ%JR%cs"

Configure nFW with Encryption
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   sudo nfw -q 0 \
     -z tcp://ntopng-host:5556 \
     -y "Yne@$w-vo<fVvi]a<NY6T1ed:M$fCG*[IaLV{hID"

**Note**: nFW only needs the public key, ntopng needs both.

Flow Export Formats
-------------------

TLV Format (Default)
~~~~~~~~~~~~~~~~~~~~

Type-Length-Value format is compact and efficient:

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:1234

**Advantages**:

- Compact binary format
- Lower bandwidth usage
- Faster serialization

JSON Format
~~~~~~~~~~~

Human-readable JSON format for debugging:

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:1234 -j

**Advantages**:

- Human-readable
- Easy to debug
- Compatible with custom integrations

**Disadvantages**:

- Higher bandwidth usage
- Slower serialization

Optimizing Flow Export
-----------------------

Flow Update Interval
~~~~~~~~~~~~~~~~~~~~

Adjust how often flows are updated:

.. code-block:: console

   # Update every 10 seconds (more real-time)
   sudo nfw -q 0 -z tcp://127.0.0.1:1234 -u 10

   # Update every 60 seconds (less overhead)
   sudo nfw -q 0 -z tcp://127.0.0.1:1234 -u 60

**Recommendations**:

- **Real-time monitoring**: 5-15 seconds
- **Historical analysis**: 30-60 seconds
- **High-volume environments**: 60-120 seconds

Immediate Flush
~~~~~~~~~~~~~~~

Disable batching for lowest latency:

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:1234 -f

**Use Cases**:

- Debugging and troubleshooting
- Low-traffic environments
- When latency is critical

**Trade-offs**:

- Higher CPU usage
- More ZMQ messages
- Increased bandwidth

Monitoring the Integration
--------------------------

Verify Flow Export
~~~~~~~~~~~~~~~~~~

**In ntopng:**

1. Go to **Interfaces**
2. Look for the ZMQ collector interface
3. Check packet/byte counters

**Check ZMQ Connection:**

.. code-block:: console

   # On ntopng host
   sudo netstat -tnlp | grep 1234

   # On nFW host
   sudo netstat -tn | grep 1234

View Flows in ntopng
~~~~~~~~~~~~~~~~~~~~~

1. **All Flows**: Go to **Flows** → **All Flows**
2. **Top Protocols**: Go to **Dashboard** → **Top Protocols**
3. **Top Applications**: Go to **Dashboard** → **Top Applications**
4. **Alerts**: Go to **Alerts** → **Detected Alerts**

Troubleshooting Integration
----------------------------

No Flows Appearing in ntopng
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Verify ntopng is listening:**

   .. code-block:: console

      sudo netstat -tnlp | grep ntopng

2. **Check nFW is running:**

   .. code-block:: console

      ps aux | grep nfw

3. **Verify ZMQ endpoint configuration:**

   - ntopng endpoint must end with ``c``
   - nFW endpoint must not have ``c``

4. **Check firewall rules:**

   .. code-block:: console

      sudo iptables -L -n -v | grep 1234

5. **Enable verbose logging:**

   .. code-block:: console

      sudo nfw -q 0 -z tcp://127.0.0.1:1234 -v

Policy Updates Not Working
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Verify policy endpoint:**

   .. code-block:: console

      sudo nfw -q 0 -z tcp://127.0.0.1:5556 -p tcp://127.0.0.1:5557 -v

2. **Check ntopng is publishing:**

   .. code-block:: console

      sudo netstat -tnlp | grep 5557

3. **Restart nFW after policy changes:**

   If using static files, send SIGHUP:

   .. code-block:: console

      sudo kill -HUP $(pidof nfw)

High Latency or Packet Loss
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Reduce flow update interval:**

   .. code-block:: console

      sudo nfw -q 0 -z tcp://127.0.0.1:1234 -u 60

2. **Use multiple queues:**

   .. code-block:: console

      sudo nfw -q 0:4 -z tcp://127.0.0.1:1234

3. **Check network bandwidth:**

   .. code-block:: console

      iftop -i eth0

Integration Best Practices
---------------------------

1. **Co-locate when possible**: Run nFW and ntopng on the same host for lowest latency
2. **Use TLV format**: More efficient than JSON for production
3. **Tune update interval**: Balance real-time visibility with performance
4. **Enable encryption**: For remote deployments or sensitive environments
5. **Monitor ZMQ queue**: Watch for backlog indicating performance issues
6. **Use multiple collectors**: For high-availability deployments
7. **Regular backups**: Export ntopng configuration and historical data

Next Steps
----------

- Explore :doc:`advanced` features
- Review :doc:`troubleshooting` for common issues
- Learn about :doc:`configuration` options for optimization
