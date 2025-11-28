Policy Rules
============

nFW uses JSON-based policy rules to define traffic filtering behavior. Policies can be loaded from files or dynamically updated via ntopng. This section explains the policy format and provides examples for common use cases.

Policy Architecture
-------------------

nFW policies consist of two main components:

1. **Pools**: Define groups of IP addresses and MAC addresses
2. **Policies**: Define filtering rules applied to pools

Policy File Format
------------------

Policy files use newline-delimited JSON format. Each line contains either a pool definition or a policy definition.

Basic Structure
~~~~~~~~~~~~~~~

.. code-block:: json

   {"pool": {...}, "policy": {"id": 1}}
   {"policy": {"id": 1, "name": "...", "markers": {...}, "default_marker": "..."}}

Pool Definitions
----------------

Pools group IP addresses and/or MAC addresses for policy application.

Pool Format
~~~~~~~~~~~

.. code-block:: json

   {
     "pool": {
       "id": <pool_id>,
       "name": "<pool_name>",
       "ip": ["<cidr1>", "<cidr2>", ...],
       "mac": ["<mac1>", "<mac2>", ...]
     },
     "policy": {
       "id": <policy_id>
     }
   }

**Fields**:

- ``id``: Unique pool identifier (integer)
- ``name``: Descriptive pool name (string)
- ``ip``: Array of IP address ranges in CIDR notation
- ``mac``: Array of MAC addresses (format: "AA:BB:CC:DD:EE:FF")
- ``policy.id``: ID of the policy to apply to this pool

Examples
~~~~~~~~

**Default Pool (All Traffic)**:

.. code-block:: json

   {
     "pool": {
       "id": 1,
       "name": "Default",
       "ip": ["0.0.0.0/0"],
       "mac": []
     },
     "policy": {
       "id": 1
     }
   }

**Internal Network Pool**:

.. code-block:: json

   {
     "pool": {
       "id": 2,
       "name": "Internal Network",
       "ip": ["192.168.1.0/24", "10.0.0.0/8"],
       "mac": []
     },
     "policy": {
       "id": 2
     }
   }

**Guest Network Pool**:

.. code-block:: json

   {
     "pool": {
       "id": 3,
       "name": "Guest WiFi",
       "ip": ["192.168.100.0/24"],
       "mac": []
     },
     "policy": {
       "id": 3
     }
   }

**MAC-Based Pool**:

.. code-block:: json

   {
     "pool": {
       "id": 4,
       "name": "Executive Devices",
       "ip": [],
       "mac": ["AA:BB:CC:DD:EE:FF", "11:22:33:44:55:66"]
     },
     "policy": {
       "id": 4
     }
   }

Policy Definitions
------------------

Policies define filtering rules based on protocols, categories, countries, and other criteria.

Policy Format
~~~~~~~~~~~~~

.. code-block:: json

   {
     "policy": {
       "id": <policy_id>,
       "root": <root_policy_id>,
       "name": "<policy_name>",
       "markers": {
         "protocols": {
           "<protocol_name>": "<marker>",
           ...
         },
         "categories": {
           "<category_name>": "<marker>",
           ...
         },
         "countries": {
           "<country_code>": "<marker>",
           ...
         },
         "continents": {
           "<continent_name>": "<marker>",
           ...
         },
         "asn": {
           "<asn_number>": "<marker>",
           ...
         }
       },
       "default_marker": "<marker>"
     }
   }

**Fields**:

- ``id``: Unique policy identifier (integer)
- ``root``: Parent policy ID for inheritance (0 = no parent)
- ``name``: Descriptive policy name (string)
- ``markers``: Object containing filtering rules
- ``default_marker``: Default action for unmatched traffic

**Marker Values**:

- ``"pass"``: Allow traffic (CONNMARK = 1)
- ``"drop"``: Block traffic (CONNMARK = 2)

Filtering Criteria
------------------

Protocols
~~~~~~~~~

Filter by detected application protocol (nDPI protocol names).

**Example**:

.. code-block:: json

   "protocols": {
     "Facebook": "drop",
     "YouTube": "drop",
     "BitTorrent": "drop",
     "SSH": "pass",
     "DNS": "pass"
   }

**Common Protocols**:

- ``Facebook``, ``Instagram``, ``WhatsApp`` (Meta services)
- ``YouTube``, ``Gmail``, ``GoogleDrive`` (Google services)
- ``Netflix``, ``Hulu``, ``Disney+`` (Streaming)
- ``BitTorrent``, ``eDonkey`` (P2P)
- ``SSH``, ``RDP``, ``VNC`` (Remote access)
- ``HTTP``, ``HTTPS``, ``DNS``, ``DHCP`` (Core protocols)

To see all supported protocols:

.. code-block:: console

   nfw -H

Categories
~~~~~~~~~~

Filter by application category (nDPI categories).

**Example**:

.. code-block:: json

   "categories": {
     "SocialNetwork": "drop",
     "Streaming": "drop",
     "Game": "drop",
     "VPN": "drop"
   }

**Common Categories**:

- ``SocialNetwork`` (Facebook, Twitter, Instagram, etc.)
- ``Streaming`` (Netflix, YouTube, Spotify, etc.)
- ``Game`` (Online gaming platforms)
- ``VPN`` (VPN services and tunnels)
- ``FileSharing`` (P2P file sharing)
- ``RemoteAccess`` (SSH, RDP, VNC)
- ``Email`` (SMTP, POP3, IMAP)
- ``Cloud`` (Dropbox, Google Drive, OneDrive)
- ``Messaging`` (WhatsApp, Telegram, Signal)
- ``Collaboration`` (Slack, Teams, Zoom)

Countries
~~~~~~~~~

Filter by source or destination country (ISO 3166-1 alpha-2 codes).

**Example**:

.. code-block:: json

   "countries": {
     "CN": "drop",
     "RU": "drop",
     "KP": "drop",
     "IR": "drop"
   }

**Common Country Codes**:

- ``US`` (United States)
- ``GB`` (United Kingdom)
- ``DE`` (Germany)
- ``FR`` (France)
- ``CN`` (China)
- ``RU`` (Russia)
- ``JP`` (Japan)
- ``KR`` (South Korea)
- ``IN`` (India)
- ``BR`` (Brazil)

Continents
~~~~~~~~~~

Filter by continent.

**Example**:

.. code-block:: json

   "continents": {
     "Asia": "drop",
     "Africa": "drop"
   }

**Valid Continents**:

- ``Africa``
- ``Asia``
- ``Europe``
- ``NorthAmerica``
- ``SouthAmerica``
- ``Oceania``
- ``Antarctica``

ASN (Autonomous System Number)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Filter by Autonomous System Number.

**Example**:

.. code-block:: json

   "asn": {
     "15169": "drop"
   }

``15169`` is Google's ASN. This would block all traffic to/from Google's network.

Complete Policy Examples
------------------------

Corporate Network Policy
~~~~~~~~~~~~~~~~~~~~~~~~

Block social media, streaming, and gaming; allow business applications:

.. code-block:: json

   {
     "pool": {
       "id": 1,
       "name": "Corporate Network",
       "ip": ["10.0.0.0/8"],
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
       "name": "Corporate Policy",
       "markers": {
         "protocols": {
           "Facebook": "drop",
           "Instagram": "drop",
           "Twitter": "drop",
           "YouTube": "drop",
           "Netflix": "drop",
           "BitTorrent": "drop"
         },
         "categories": {
           "SocialNetwork": "drop",
           "Streaming": "drop",
           "Game": "drop",
           "FileSharing": "drop"
         },
         "countries": {},
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }

Guest WiFi Policy
~~~~~~~~~~~~~~~~~

Restricted access for guest users:

.. code-block:: json

   {
     "pool": {
       "id": 2,
       "name": "Guest WiFi",
       "ip": ["192.168.100.0/24"],
       "mac": []
     },
     "policy": {
       "id": 2
     }
   }
   {
     "policy": {
       "id": 2,
       "root": 0,
       "name": "Guest Policy",
       "markers": {
         "protocols": {
           "BitTorrent": "drop",
           "eDonkey": "drop"
         },
         "categories": {
           "FileSharing": "drop",
           "VPN": "drop",
           "RemoteAccess": "drop"
         },
         "countries": {
           "CN": "drop",
           "RU": "drop",
           "KP": "drop"
         },
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }

Parental Control Policy
~~~~~~~~~~~~~~~~~~~~~~~

Protect children from inappropriate content:

.. code-block:: json

   {
     "pool": {
       "id": 3,
       "name": "Kids Devices",
       "ip": ["192.168.1.100", "192.168.1.101"],
       "mac": []
     },
     "policy": {
       "id": 3
     }
   }
   {
     "policy": {
       "id": 3,
       "root": 0,
       "name": "Parental Control",
       "markers": {
         "protocols": {
           "Facebook": "drop",
           "Instagram": "drop",
           "TikTok": "drop",
           "YouTube": "drop",
           "BitTorrent": "drop"
         },
         "categories": {
           "SocialNetwork": "drop",
           "Game": "drop",
           "FileSharing": "drop",
           "Streaming": "drop"
         },
         "countries": {},
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }

Geographic Restrictions
~~~~~~~~~~~~~~~~~~~~~~~

Block traffic from high-risk regions:

.. code-block:: json

   {
     "pool": {
       "id": 4,
       "name": "All Traffic",
       "ip": ["0.0.0.0/0"],
       "mac": []
     },
     "policy": {
       "id": 4
     }
   }
   {
     "policy": {
       "id": 4,
       "root": 0,
       "name": "Geographic Filter",
       "markers": {
         "protocols": {},
         "categories": {},
         "countries": {
           "CN": "drop",
           "RU": "drop",
           "KP": "drop",
           "IR": "drop",
           "SY": "drop"
         },
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }

Multi-Pool Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

Different policies for different network segments:

.. code-block:: json

   {
     "pool": {
       "id": 1,
       "name": "Employees",
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
       "name": "Employee Policy",
       "markers": {
         "protocols": {},
         "categories": {
           "Game": "drop",
           "FileSharing": "drop"
         },
         "countries": {},
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }
   {
     "pool": {
       "id": 2,
       "name": "Executives",
       "ip": ["192.168.2.0/24"],
       "mac": []
     },
     "policy": {
       "id": 2
     }
   }
   {
     "policy": {
       "id": 2,
       "root": 0,
       "name": "Executive Policy",
       "markers": {
         "protocols": {},
         "categories": {},
         "countries": {},
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }
   {
     "pool": {
       "id": 3,
       "name": "IoT Devices",
       "ip": ["192.168.3.0/24"],
       "mac": []
     },
     "policy": {
       "id": 3
     }
   }
   {
     "policy": {
       "id": 3,
       "root": 0,
       "name": "IoT Policy",
       "markers": {
         "protocols": {},
         "categories": {
           "RemoteAccess": "drop",
           "VPN": "drop"
         },
         "countries": {
           "CN": "drop",
           "RU": "drop"
         },
         "continents": {},
         "asn": {}
       },
       "default_marker": "pass"
     }
   }

Policy Management
-----------------

Static Policies (File-Based)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Load policies from a file:

.. code-block:: console

   sudo nfw -q 0 -r /etc/nfw/policy.json

Reload policies without restarting:

.. code-block:: console

   sudo kill -HUP $(pidof nfw)

Dynamic Policies (ntopng-Based)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Receive policies from ntopng:

.. code-block:: console

   sudo nfw -q 0 -p tcp://ntopng-server:5557

Policies are automatically updated when changed in ntopng's web interface.

Testing Policies
----------------

Verify Policy Application
~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Start nFW with verbose logging**:

   .. code-block:: console

      sudo nfw -q 0 -r /etc/nfw/policy.json -v

2. **Generate test traffic**:

   .. code-block:: console

      curl https://www.facebook.com

3. **Check logs for policy application**:

   Look for messages like:

   .. code-block:: text

      Flow marked as DROP: Facebook (protocol)

4. **Check conntrack marks**:

   .. code-block:: console

      sudo conntrack -L | grep -E "mark=[12]"

   - ``mark=1``: Passed flows
   - ``mark=2``: Dropped flows

Debug Policy Matching
~~~~~~~~~~~~~~~~~~~~~

If a policy isn't working as expected:

1. **Verify protocol detection**:

   Check nFW logs to see which protocol was detected.

2. **Check pool membership**:

   Ensure the source/destination IP is in the expected pool.

3. **Verify marker precedence**:

   Protocol-specific markers override category markers.

4. **Test with default marker**:

   Temporarily set ``"default_marker": "drop"`` and allow specific protocols.

Best Practices
--------------

1. **Start Permissive**: Begin with ``"default_marker": "pass"`` and selectively block
2. **Test Incrementally**: Add rules one at a time and verify behavior
3. **Use Categories**: Categories are easier to maintain than individual protocols
4. **Document Policies**: Use descriptive names and comments (in separate documentation)
5. **Monitor Impact**: Use ntopng to monitor dropped flows and adjust policies
6. **Avoid Over-Blocking**: Be careful with country/continent blocks—may break CDNs
7. **Regular Updates**: Keep nDPI updated for latest protocol detection

Limitations
-----------

- **Protocol Detection Delay**: nDPI may require a few packets to detect protocols. Initial packets may pass before detection completes.
- **Encrypted Traffic**: Some protocols cannot be detected in encrypted traffic.
- **CDN Impact**: Blocking countries/ASNs may break CDN-hosted content.
- **Dynamic IPs**: GeoIP data may be outdated for dynamic IP ranges.

Next Steps
----------

- Set up :doc:`ntopng_integration` for dynamic policy management
- Learn about :doc:`advanced` features
- Review :doc:`troubleshooting` if policies aren't working as expected
