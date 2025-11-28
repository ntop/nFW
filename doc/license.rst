License Management
==================

nFW is commercial software that requires a valid license to operate. This section explains how to obtain, install, and manage your nFW license.

License Overview
----------------

Types of Licenses
~~~~~~~~~~~~~~~~~

nFW licenses are issued by ntop.org and come in different types:

1. **Evaluation License**: Time-limited license for testing and evaluation
2. **Commercial License**: Full license for production use
3. **OEM License**: For integration into third-party products
4. **Academic License**: For educational and research institutions

License Features
~~~~~~~~~~~~~~~~

Each license specifies:

- **Licensed System**: Hardware identifier (System ID)
- **Expiration Date**: License validity period
- **Maintenance Date**: Support and updates expiration
- **Features**: Enabled functionality (standard or advanced)

Obtaining a License
-------------------

Step 1: Get System ID
~~~~~~~~~~~~~~~~~~~~~~

Run nFW with the ``--show-system-id`` option:

.. code-block:: console

   nfw --show-system-id

**Important**: The System ID is derived from hardware identifiers and must match the system where nFW will run.

Step 2: Request License
~~~~~~~~~~~~~~~~~~~~~~~~

Contact ntop.org to purchase a license:

- **Website**: https://www.ntop.org
- **Email**: sales@ntop.org
- **Information Needed**:
  - System ID
  - License type (evaluation, commercial, etc.)
  - Company/organization details
  - Intended use case

Step 3: Receive License File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ntop.org will provide a ``nfw.license`` file tied to your System ID.

Installing a License
--------------------

License File Locations
~~~~~~~~~~~~~~~~~~~~~~

nFW searches for the license file in the following locations (in order):

1. ``./nfw.license`` (current directory)
2. ``/etc/nfw.license`` (system-wide)

Recommended Installation
~~~~~~~~~~~~~~~~~~~~~~~~

Install the license system-wide:

.. code-block:: console

   # Copy license file
   sudo cp nfw.license /etc/nfw.license

   # Set appropriate permissions
   sudo chmod 644 /etc/nfw.license
   sudo chown root:root /etc/nfw.license

Verifying Installation
~~~~~~~~~~~~~~~~~~~~~~

Check that the license is valid:

.. code-block:: console

   nfw --check-license

Expected output:

.. code-block:: text

   License is valid
   Licensed to: Your Company Name
   Expiration: 2026-12-31
   Maintenance: 2026-12-31

If the license is invalid, you'll see an error message:

.. code-block:: text

   Invalid license: License expired
   Invalid license: System ID mismatch
   Invalid license: License file not found

License Validation
------------------

Checking License Status
~~~~~~~~~~~~~~~~~~~~~~~

**Check License Validity**:

.. code-block:: console

   nfw --check-license

**Check Maintenance Status**:

.. code-block:: console

   nfw --check-maintenance

Output:

.. code-block:: text

   Maintenance valid until: 2026-12-31
   Updates and support available

**Display System ID**:

.. code-block:: console

   nfw --show-system-id

Validation at Startup
~~~~~~~~~~~~~~~~~~~~~~

When nFW starts, it automatically validates the license:

.. code-block:: console

   sudo nfw -q 0 -z tcp://127.0.0.1:1234

If the license is invalid, nFW will not start:

.. code-block:: text

   ERROR: Invalid license
   ERROR: License expired on 2025-12-31
   ERROR: Please contact ntop.org for license renewal

License Expiration
------------------

Grace Period
~~~~~~~~~~~~

Some licenses may include a grace period after expiration, allowing nFW to continue running with warnings.

Renewal
~~~~~~~

To renew an expired license:

1. Contact ntop.org sales team
2. Provide your current System ID
3. Receive updated license file
4. Replace ``/etc/nfw.license`` with the new file
5. Restart nFW (if running)

.. code-block:: console

   sudo cp nfw-new.license /etc/nfw.license
   sudo killall nfw
   sudo nfw -q 0 -z tcp://127.0.0.1:1234

Maintenance Expiration
----------------------

Maintenance Period
~~~~~~~~~~~~~~~~~~

The maintenance period covers:

- Software updates and bug fixes
- nDPI protocol updates
- Technical support
- Security patches

After Maintenance Expires
~~~~~~~~~~~~~~~~~~~~~~~~~

When maintenance expires:

- nFW continues to operate normally
- New updates and features are not available
- Technical support may be limited
- nDPI protocol database becomes outdated

**To restore maintenance**:

Contact ntop.org to renew your maintenance agreement.

Troubleshooting License Issues
-------------------------------

License File Not Found
~~~~~~~~~~~~~~~~~~~~~~

**Error**: ``License file not found``

**Solutions**:

.. code-block:: console

   # Check if license file exists
   ls -la nfw.license /etc/nfw.license

   # If missing, copy license to correct location
   sudo cp nfw.license /etc/nfw.license

   # Verify permissions
   sudo chmod 644 /etc/nfw.license

Invalid License
~~~~~~~~~~~~~~~

**Error**: ``Invalid license``

**Causes**:

1. **Corrupted file**: Re-download or request a new license file
2. **Wrong format**: Ensure binary format, not text
3. **Tampered file**: License files are cryptographically signed

**Solution**: Re-download the original license file from ntop.org

System ID Mismatch
~~~~~~~~~~~~~~~~~~

**Error**: ``System ID mismatch`` or ``License not valid for this system``

**Causes**:

1. **License installed on wrong system**: License is tied to specific hardware
2. **Hardware changed**: Motherboard, network cards, or other hardware changed
3. **Virtual machine cloned**: VM cloning changes hardware identifiers

**Solutions**:

1. **Verify System ID**:

   .. code-block:: console

      nfw --show-system-id

2. **Contact ntop.org**: Request license transfer or update for new hardware

3. **For VMs**: Use stable hardware identifiers or request VM-friendly license

License Expired
~~~~~~~~~~~~~~~

**Error**: ``License expired on YYYY-MM-DD``

**Solution**: Contact ntop.org to renew your license

.. code-block:: console

   # Check expiration date
   nfw --check-license

Maintenance Expired
~~~~~~~~~~~~~~~~~~~

**Error**: ``Maintenance expired on YYYY-MM-DD``

**Impact**: nFW continues to run, but updates and support are unavailable

**Solution**: Renew maintenance with ntop.org

.. code-block:: console

   # Check maintenance status
   nfw --check-maintenance

License for Virtual Machines
-----------------------------

VM Considerations
~~~~~~~~~~~~~~~~~

Virtual machines present special challenges for hardware-based licensing:

- **VM Cloning**: Cloned VMs may have different hardware IDs
- **VM Migration**: Moving VMs between hosts can change hardware
- **Snapshots**: Restoring snapshots may affect licensing

Best Practices for VMs
~~~~~~~~~~~~~~~~~~~~~~

1. **Use Static Hardware IDs**: Configure VM with stable MAC addresses and disk UUIDs

2. **Request VM-Specific License**: Some licenses can be tied to software identifiers instead of hardware

3. **Document VM Configuration**: Keep records of VM settings for license requests

4. **Test Before Deployment**: Verify license works after VM cloning/migration

License Backup
--------------

It's recommended to keep backup copies of your license:

.. code-block:: console

   # Backup license file
   sudo cp /etc/nfw.license ~/nfw.license.backup

   # Store in version control or secure location
   # IMPORTANT: Do not share license files publicly

**Note**: License files are tied to specific systems and should be kept secure.

Moving License to New Hardware
-------------------------------

If you need to move nFW to new hardware:

Step 1: Get New System ID
~~~~~~~~~~~~~~~~~~~~~~~~~~

On the new system:

.. code-block:: console

   nfw --show-system-id

Step 2: Contact ntop.org
~~~~~~~~~~~~~~~~~~~~~~~~~

Request a license transfer or new license:

- Provide old System ID
- Provide new System ID
- Explain reason for transfer (hardware upgrade, replacement, etc.)

Step 3: Install New License
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once received:

.. code-block:: console

   sudo cp nfw-new.license /etc/nfw.license
   nfw --check-license

License Compliance
------------------

Terms of Use
~~~~~~~~~~~~

nFW licenses typically include:

- **Single System**: License is valid for one system only
- **No Redistribution**: License files cannot be shared or redistributed
- **No Modification**: License files must not be altered
- **Commercial Use**: Requires commercial license (evaluation licenses are for testing only)

Compliance Verification
~~~~~~~~~~~~~~~~~~~~~~~

ntop.org may periodically verify license compliance. Ensure:

- License file is authentic and unmodified
- System ID matches licensed system
- License is not expired
- Usage complies with license terms

Frequently Asked Questions
--------------------------

Can I use one license on multiple systems?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No. Each license is tied to a specific System ID and can only be used on one system.

What happens if my hardware fails?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Contact ntop.org to transfer your license to replacement hardware. Provide both old and new System IDs.

Can I run nFW without a license for testing?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

nFW requires a valid license. Contact ntop.org for an evaluation license for testing purposes.

How long is the license valid?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

License validity periods vary. Check with ``nfw --check-license`` or review your license agreement.

What's the difference between license and maintenance expiration?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **License Expiration**: nFW stops working
- **Maintenance Expiration**: nFW continues working, but no updates or support

Do I need a new license for nFW updates?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No. Your license remains valid for all nFW versions during your maintenance period.

Can I transfer my license to a virtual machine?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Yes, if the VM has a stable System ID. Contact ntop.org if you encounter issues.

Support and Contact
-------------------

For license-related questions or issues:

- **Sales**: sales@ntop.org
- **Support**: support@ntop.org
- **Website**: https://www.ntop.org
- **Phone**: Check website for regional contact numbers

When contacting support, please provide:

- System ID
- License file name
- Error messages
- Output of ``nfw --check-license``

Next Steps
----------

- Complete the :doc:`install` process
- Follow the :doc:`quick_start` guide
- Review :doc:`configuration` options
