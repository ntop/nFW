Introduction
============

What is nEdge Lite?
--------------------

nEdge Lite is a high-performance, netfilter-based packet filtering and Deep Packet Inspection (DPI) application designed for Linux systems. It provides Layer-7 (application-level) traffic control by inspecting packet contents and applying sophisticated filtering policies based on protocols, applications, geographic locations, and other advanced criteria.

nEdge Lite integrates seamlessly with `ntopng <https://www.ntop.org/products/traffic-analysis/ntop/>`_, enabling centralized monitoring, policy management, and real-time traffic analytics.

Key Features
------------

Layer-7 Deep Packet Inspection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Protocol Detection**: Leverages the powerful `nDPI <https://www.ntop.org/products/deep-packet-inspection/ndpi/>`_ library to identify hundreds of protocols and applications
- **Application Awareness**: Detects applications like Facebook, YouTube, Netflix, Zoom, and thousands more
- **Encrypted Traffic Analysis**: Identifies protocols even within encrypted traffic using behavioral analysis and heuristics

Policy-Based Filtering
~~~~~~~~~~~~~~~~~~~~~~

- **Protocol-Based Rules**: Block or allow traffic based on detected protocols (e.g., block BitTorrent, allow HTTP)
- **Category-Based Rules**: Filter by application categories (Social Networks, Gaming, Streaming, etc.)
- **Geographic Filtering**: Block or allow traffic based on country or continent using GeoIP databases
- **ASN-Based Rules**: Filter traffic by Autonomous System Number
- **Custom Policies**: Define granular policies for specific IP ranges or network segments

Netfilter Integration
~~~~~~~~~~~~~~~~~~~~~

- **NFQUEUE Support**: Leverages Linux netfilter userspace queuing for packet interception
- **Conntrack Integration**: Integrates with connection tracking to mark entire flows
- **Multiple Queue Support**: Distributes packet processing across multiple queues for better performance
- **Bridge and Router Modes**: Works transparently in bridge mode or as a traditional router/firewall

ntopng Integration
~~~~~~~~~~~~~~~~~~

- **Real-Time Flow Export**: Exports detailed flow information to ntopng via ZeroMQ
- **Bidirectional Policy Updates**: Receives policy updates from ntopng in real-time
- **Centralized Management**: Configure and manage policies through ntopng's web interface
- **Flow Analytics**: View detailed statistics, historical data, and alerts in ntopng

Performance and Scalability
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **High Throughput**: Optimized for multi-gigabit traffic processing
- **Multi-Queue Processing**: Distributes workload across CPU cores
- **Efficient Flow Tracking**: Uses hash-based flow tables for fast lookups
- **Memory Efficient**: Automatic idle flow purging and resource management
- **Low Latency**: Minimal packet processing overhead

Flexible Deployment
~~~~~~~~~~~~~~~~~~~

- **Bridge Mode**: Transparent deployment between network segments without IP configuration
- **Single Interface Mode**: Protects traffic on a single network interface
- **Multi-Instance Support**: Run multiple nEdge Lite instances on different hosts, all reporting to the same ntopng

Use Cases
---------

Enterprise Network Security
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Deploy nEdge Lite to enforce corporate acceptable use policies, blocking unauthorized applications and protocols while allowing business-critical traffic. Monitor and control:

- Social media and entertainment sites
- Peer-to-peer file sharing
- Cryptocurrency mining
- Gaming and streaming services

Network Segmentation
~~~~~~~~~~~~~~~~~~~~

Use nEdge Lite to implement micro-segmentation in data centers and cloud environments:

- Control East-West traffic between VLANs or subnets
- Enforce application-level policies between security zones
- Monitor and restrict inter-service communication

Geographic Restrictions
~~~~~~~~~~~~~~~~~~~~~~~

Organizations can enforce geographic access controls:

- Block traffic from high-risk countries
- Comply with data sovereignty regulations
- Prevent access to region-specific content
- Implement sanctions and embargo policies

Educational Institutions
~~~~~~~~~~~~~~~~~~~~~~~~

Schools and universities can use nEdge Lite to:

- Protect students from inappropriate content
- Prevent bandwidth abuse from streaming and gaming
- Enforce lab and classroom network policies
- Monitor network usage for security incidents

How It Works
------------

nEdge Lite operates as a userspace application that receives packets from the Linux kernel via netfilter's NFQUEUE mechanism:

1. **Packet Interception**: iptables rules route packets to an NFQUEUE for inspection
2. **Flow Tracking**: nEdge Lite maintains a hash table of active network flows (5-tuple: protocol, source IP, source port, destination IP, destination port)
3. **DPI Processing**: Each packet is processed through nDPI for protocol detection
4. **Policy Evaluation**: Once a protocol is detected, nEdge Lite applies the configured policy rules
5. **Connection Marking**: The verdict (pass/drop) is applied to the connection using CONNMARK
6. **Packet Verdict**: The packet is returned to the kernel with an accept or drop verdict
7. **Flow Export**: Flow metadata is periodically exported to ntopng via ZeroMQ
8. **Policy Synchronization**: Policy updates from ntopng are received and applied dynamically

This architecture ensures that packets are inspected efficiently with minimal latency, while providing comprehensive visibility and control over network traffic.

Getting Started
---------------

Ready to deploy nEdge Lite? Continue to the :doc:`install` section to set up your system, or jump to the :doc:`quick_start` guide for a hands-on introduction.
