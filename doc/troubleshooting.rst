Troubleshooting
===============

This section provides solutions to common issues encountered when deploying and operating nEdge Lite.

Startup Issues
--------------

License Validation Failed
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: ``Invalid license`` or ``License file not found``

**Solution**:

.. code-block:: console

   # Check license file location
   ls -la nedgelite.license /etc/nedgelite.license

   # Verify license validity
   nedgelite --check-license

   # Display system ID for license generation
   nedgelite --show-system-id

   # Contact ntop.org for valid license

Unable to Bind to NFQUEUE
~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: ``Unable to open queue 0`` or ``Unable to bind to queue``

**Causes and Solutions**:

1. **Queue already in use**:

   .. code-block:: console

      # Check for other nedgelite instances
      ps aux | grep nedgelite
      sudo killall nedgelite

      # Check for other applications using the queue
      cat /proc/net/netfilter/nfnetlink_queue

2. **Kernel module not loaded**:

   .. code-block:: console

      sudo modprobe nfnetlink_queue
      sudo modprobe nf_conntrack

3. **Insufficient permissions**:

   .. code-block:: console

      # Must run as root
      sudo nedgelite -q 0 -z tcp://127.0.0.1:1234

Permission Denied Errors
~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: ``Permission denied`` when accessing netfilter

**Solution**:

.. code-block:: console

   # Run as root or with sudo
   sudo nedgelite -q 0 -z tcp://127.0.0.1:1234

   # Check capabilities (if using capabilities instead of root)
   sudo setcap cap_net_admin=eip /usr/local/bin/nedgelite

Packet Processing Issues
-------------------------

No Packets Reaching nEdge Lite
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: nEdge Lite starts but shows no activity

**Diagnosis**:

.. code-block:: console

   # Check iptables rules
   sudo iptables -t mangle -L -n -v

   # Verify packets are being queued
   # Look for non-zero packet counts on NFQUEUE rules

   # Check queue statistics
   cat /proc/net/netfilter/nfnetlink_queue

**Solutions**:

1. **iptables rules not configured**:

   .. code-block:: console

      # Run setup script
      sudo /usr/share/nedgelite/scripts/default_setup.sh eth0

2. **Wrong queue number**:

   .. code-block:: console

      # Ensure iptables queue-num matches nedgelite -q option
      sudo iptables -t mangle -L -n -v | grep NFQUEUE
      # Should show: --queue-num 0
      # Must match: nedgelite -q 0

3. **Packets already marked**:

   .. code-block:: console

      # Flush conntrack table to reset marks
      sudo conntrack -F

4. **Traffic not matching rules**:

   .. code-block:: console

      # Add catch-all rule for testing
      sudo iptables -t mangle -A PREROUTING -j NFQUEUE --queue-num 0 --queue-bypass

Packets Dropped Unexpectedly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: Legitimate traffic is being blocked

**Diagnosis**:

.. code-block:: console

   # Run with verbose logging
   sudo nedgelite -q 0 -r /etc/nedgelite/policy.json -v

   # Check conntrack marks
   sudo conntrack -L | grep mark=2

   # View blocked flows in ntopng

**Solutions**:

1. **Overly broad policy rules**:

   Review policy file and adjust category/protocol filters:

   .. code-block:: json

      {
        "policy": {
          "markers": {
            "categories": {
              "SocialNetwork": "drop"  # May block too much
            }
          },
          "default_marker": "pass"  # Change from "drop"
        }
      }

2. **Protocol misdetection**:

   Check nEdge Lite logs to see what protocol was detected. nDPI may occasionally misidentify protocols.

3. **Default marker is "drop"**:

   .. code-block:: json

      "default_marker": "pass"  # More permissive

4. **Testing policies**:

   .. code-block:: console

      # Temporarily use pass-all policy
      sudo nedgelite -q 0 -v  # No -r option

High Packet Loss
~~~~~~~~~~~~~~~~

**Problem**: Packets are being dropped due to queue overflow

**Diagnosis**:

.. code-block:: console

   # Check queue statistics
   cat /proc/net/netfilter/nfnetlink_queue
   # Look for queue_dropped counter

   # Check system logs
   dmesg | grep nf_queue

**Solutions**:

1. **Increase queue size**:

   .. code-block:: console

      sudo iptables -t mangle -A PREROUTING -m mark --mark 0 \
        -j NFQUEUE --queue-num 0 --queue-size 4096

2. **Use multiple queues**:

   .. code-block:: console

      sudo iptables -t mangle -A PREROUTING -m mark --mark 0 \
        -j NFQUEUE --queue-balance 0:3 --queue-cpu-fanout

      sudo nedgelite -q 0:4 -z tcp://127.0.0.1:1234

3. **Add queue-bypass**:

   .. code-block:: console

      sudo iptables -t mangle -A PREROUTING -m mark --mark 0 \
        -j NFQUEUE --queue-num 0 --queue-bypass

   This allows packets to pass if nEdge Lite is overloaded or crashes.

4. **Optimize nEdge Lite performance**:

   - Reduce flow update interval: ``-u 60``
   - Increase conntrack table size
   - Pin nEdge Lite to dedicated CPU cores

Protocol Detection Issues
--------------------------

Protocols Not Being Detected
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: nDPI marks flows as "Unknown"

**Causes**:

- Encrypted traffic (HTTPS, VPN)
- Not enough packets inspected
- New/unsupported protocol
- Fragmented packets

**Solutions**:

1. **Update nDPI library**:

   .. code-block:: console

      cd ~/nDPI
      git pull
      make
      sudo make install

2. **Check nDPI protocol list**:

   .. code-block:: console

      nedgelite -H | grep -i <protocol>

3. **Allow more packets for detection**:

   nDPI may need 10-20 packets for some protocols. Ensure initial packets aren't being dropped.

4. **Custom protocol configuration**:

   Consult nDPI documentation for custom protocol definitions.

Encrypted Traffic
~~~~~~~~~~~~~~~~~

**Problem**: HTTPS/TLS traffic not classified correctly

**Explanation**: nDPI can detect many protocols over HTTPS using:

- TLS SNI (Server Name Indication)
- Certificate analysis
- Behavioral patterns

**Limitations**: Some encrypted protocols cannot be identified without TLS interception (MITM).

Wrong Protocol Detected
~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: nDPI misidentifies the protocol

**Solutions**:

1. **Update nDPI**: Newer versions have improved accuracy

2. **Report to nDPI project**: Submit PCAP files for analysis

3. **Use category-based policies**: Categories are more reliable than individual protocols

4. **Manual override**: Use IP-based rules for specific services

Policy Issues
-------------

Policy Not Applied
~~~~~~~~~~~~~~~~~~

**Problem**: Policy rules are not being enforced

**Diagnosis**:

.. code-block:: console

   # Check if policy file is being loaded
   sudo nedgelite -q 0 -r /etc/nedgelite/policy.json -v
   # Look for "Loading policy" messages

   # Verify JSON syntax
   cat /etc/nedgelite/policy.json | jq .

**Solutions**:

1. **JSON syntax error**:

   .. code-block:: console

      # Validate JSON
      cat /etc/nedgelite/policy.json | jq .
      # Fix any syntax errors

2. **Pool not matching traffic**:

   Verify IP ranges in pool definition:

   .. code-block:: json

      {
        "pool": {
          "ip": ["192.168.1.0/24"],  # Must match actual traffic
          ...
        }
      }

3. **Reload policy**:

   .. code-block:: console

      sudo kill -HUP $(pidof nedgelite)

Policy Changes Not Taking Effect
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: Modifications to policy file have no effect

**Solutions**:

1. **Reload policy**:

   .. code-block:: console

      sudo kill -HUP $(pidof nedgelite)

2. **Restart nEdge Lite**:

   .. code-block:: console

      sudo killall nedgelite
      sudo nedgelite -q 0 -r /etc/nedgelite/policy.json -z tcp://127.0.0.1:1234

3. **Clear conntrack**:

   Existing connections retain old marks:

   .. code-block:: console

      sudo conntrack -F

ntopng Integration Issues
--------------------------

No Flows Appearing in ntopng
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: nEdge Lite is running but ntopng shows no flows

**Diagnosis**:

.. code-block:: console

   # Check ZMQ connection
   sudo netstat -tn | grep 1234

   # Verify ntopng endpoint
   ps aux | grep ntopng | grep tcp://

**Solutions**:

1. **Endpoint mismatch**:

   .. code-block:: console

      # ntopng must have trailing 'c'
      sudo ntopng -i tcp://127.0.0.1:1234c

      # nEdge Lite must NOT have trailing 'c'
      sudo nedgelite -q 0 -z tcp://127.0.0.1:1234

2. **Firewall blocking ZMQ**:

   .. code-block:: console

      # Allow TCP port 1234
      sudo iptables -A INPUT -p tcp --dport 1234 -j ACCEPT

3. **ntopng not running**:

   .. code-block:: console

      ps aux | grep ntopng
      # Start ntopng if not running

ZMQ Connection Refused
~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: ``Connection refused`` when nEdge Lite tries to connect to ntopng

**Solutions**:

1. **Start ntopng first**:

   .. code-block:: console

      sudo ntopng -i tcp://0.0.0.0:1234c

2. **Check ntopng is listening**:

   .. code-block:: console

      sudo netstat -tnlp | grep 1234

3. **Verify endpoint address**:

   - Use ``0.0.0.0`` on ntopng to listen on all interfaces
   - Use correct IP address in nEdge Lite ``-z`` option

Policy Updates from ntopng Not Working
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: Changes in ntopng GUI don't affect nEdge Lite

**Solutions**:

1. **Verify policy endpoint**:

   .. code-block:: console

      # ntopng must publish events
      sudo ntopng -i tcp://0.0.0.0:5556c --zmq-publish-events tcp://0.0.0.0:5557

      # nEdge Lite must subscribe
      sudo nedgelite -q 0 -z tcp://ntopng:5556 -p tcp://ntopng:5557

2. **Check ZMQ policy connection**:

   .. code-block:: console

      sudo netstat -tn | grep 5557

3. **Restart both services**:

   .. code-block:: console

      sudo killall ntopng nedgelite
      # Start ntopng first, then nEdge Lite

Performance Issues
------------------

High CPU Usage
~~~~~~~~~~~~~~

**Problem**: nEdge Lite consuming excessive CPU

**Causes**:

- High packet rate
- Complex policies
- Too frequent flow updates

**Solutions**:

1. **Use multiple queues**:

   .. code-block:: console

      sudo nedgelite -q 0:$(nproc) -z tcp://127.0.0.1:1234

2. **Reduce flow update frequency**:

   .. code-block:: console

      sudo nedgelite -q 0 -z tcp://127.0.0.1:1234 -u 60

3. **Optimize policies**:

   - Use category filters instead of many protocol filters
   - Simplify policy logic

4. **System tuning**:

   .. code-block:: console

      # Increase conntrack table
      sudo sysctl -w net.netfilter.nf_conntrack_max=1048576

High Memory Usage
~~~~~~~~~~~~~~~~~

**Problem**: nEdge Lite memory usage growing over time

**Diagnosis**:

.. code-block:: console

   # Monitor memory
   ps aux | grep nedgelite
   pmap $(pidof nedgelite)

**Causes**:

- Large number of concurrent flows
- Flow leaks (not being purged)

**Solutions**:

1. **Normal behavior**: ~2-3KB per active flow is expected

2. **Adjust idle timeout**: If flows aren't being purged, check conntrack timeouts:

   .. code-block:: console

      sudo sysctl -w net.netfilter.nf_conntrack_tcp_timeout_established=3600

3. **Monitor flow count**:

   .. code-block:: console

      sudo conntrack -C  # Show connection count

Network Latency
~~~~~~~~~~~~~~~

**Problem**: Increased latency when nEdge Lite is running

**Solutions**:

1. **Optimize queue processing**:

   - Use ``--queue-cpu-fanout`` in iptables
   - Pin nEdge Lite to dedicated cores

2. **Disable flow batching**:

   .. code-block:: console

      sudo nedgelite -q 0 -z tcp://127.0.0.1:1234 -f

3. **Check queue depth**:

   .. code-block:: console

      cat /proc/net/netfilter/nfnetlink_queue

System Issues
-------------

Kernel Module Not Loading
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem**: ``modprobe: FATAL: Module nfnetlink_queue not found``

**Solutions**:

.. code-block:: console

   # Install kernel modules
   # Ubuntu/Debian
   sudo apt-get install linux-headers-$(uname -r)

   # CentOS/Rocky
   sudo yum install kernel-devel-$(uname -r)

   # Rebuild modules
   sudo depmod -a
   sudo modprobe nfnetlink_queue

Conntrack Table Full
~~~~~~~~~~~~~~~~~~~~~

**Problem**: ``nf_conntrack: table full, dropping packet``

**Solutions**:

.. code-block:: console

   # Increase conntrack table size
   sudo sysctl -w net.netfilter.nf_conntrack_max=1048576

   # Make persistent
   echo "net.netfilter.nf_conntrack_max=1048576" | \
     sudo tee -a /etc/sysctl.conf

   # Verify
   sudo sysctl -p

Out of Memory
~~~~~~~~~~~~~

**Problem**: Kernel OOM killer terminates nEdge Lite

**Solutions**:

1. **Add more RAM**

2. **Reduce flow count**: Implement aggressive timeout policies

3. **Limit nEdge Lite memory**:

   .. code-block:: console

      # Use systemd to limit memory
      sudo systemctl set-property nedgelite.service MemoryMax=1G

Debugging Techniques
--------------------

Verbose Logging
~~~~~~~~~~~~~~~

.. code-block:: console

   sudo nedgelite -q 0 -r /etc/nedgelite/policy.json -z tcp://127.0.0.1:1234 -v

Packet Capture
~~~~~~~~~~~~~~

.. code-block:: console

   # Capture packets going to NFQUEUE
   sudo tcpdump -i any -w /tmp/capture.pcap

   # Analyze with Wireshark
   wireshark /tmp/capture.pcap

Conntrack Monitoring
~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   # Watch connections in real-time
   watch -n 1 'sudo conntrack -L | head -20'

   # Show statistics
   sudo conntrack -S

   # Show connections with marks
   sudo conntrack -L -m

iptables Rule Verification
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   # List mangle table with counters
   sudo iptables -t mangle -L -n -v

   # Trace packet path (requires iptables-extensions)
   sudo iptables -t raw -A PREROUTING -p tcp --dport 80 -j TRACE
   sudo xtables-monitor -t

System Logs
~~~~~~~~~~~

.. code-block:: console

   # Check kernel messages
   sudo dmesg | grep -i "nf_queue\|conntrack\|nedgelite"

   # Check system logs
   sudo journalctl -u nedgelite -f  # If running as systemd service

Getting Help
------------

If you cannot resolve an issue:

1. **Check Documentation**: Review relevant sections of this guide

2. **Search Issues**: Look for similar issues on GitHub

3. **Collect Diagnostics**:

   .. code-block:: console

      # System info
      uname -a
      nedgelite --version
      ntopng --version

      # Configuration
      iptables -t mangle -L -n -v
      cat /etc/nedgelite/policy.json
      ps aux | grep nedgelite

      # Statistics
      conntrack -S
      cat /proc/net/netfilter/nfnetlink_queue

4. **Report Bug**: https://github.com/ntop/nEdge Lite/issues

Include:

- nEdge Lite version
- Operating system and kernel version
- Configuration files
- Log output with ``-v`` flag
- Steps to reproduce

5. **Commercial Support**: Contact ntop.org for professional support

Next Steps
----------

- Review :doc:`configuration` for optimization options
- Learn about :doc:`advanced` features
- Consult :doc:`architecture` for technical details
