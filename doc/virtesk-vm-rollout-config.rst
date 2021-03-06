Configuring virtual rooms
=======================================

The following steps explain the config file of virtesk-vm-rollout,
focusing on how to define and configure virtual rooms.

--------------

Config structure
----------------

Syntax of ``virtesk-vm-rollout.conf``:

::

    # This is a comment
    [section1]
        key1 = value1
        key2 = value2
    [section2]
        [[subsection 1]]
        key3 = "value" # values can optionally be enclosed in double quotes
        [[subsection 2]]
        # empty subsection

For syntax details, please refer to `ConfigObj library
documentation <http://www.voidspace.org.uk/python/configobj.html>`__

A sample config file is available under ``sample_config/virtesk-vm-rollout.conf``.

Structure of ``virtesk-vm-rollout.conf``:

::

    [general]
        # General Tool configuration
        [[connect]]
        # Parameters for connecting to Ovirt REST API   

    [logging]
        # Configuration of logging library

    [room room01]  # Defines a virtual room with name "room01"
        [[student_vms]]
        # Complete definition of all VMs

        [[teacher_vms]]
        # Complete definition of all VMs

        [[other_vms]]
        # Complete definition of all VMs
        
    [room room02]  # Defines a virtual room with name "room02"
        [[vms]]
        # Complete definition of all VMs

All configuration is mandatory. No configuration may be omitted. There
are no default values.

Location of ``virtesk-vm-rollout.conf``: The scripts do look for the
config file in the following locations, first match wins:

-  Optional command line argument
   (``virtesk-virtroom-rollout --config /path/to/virtesk-vm-rollout.conf myroom01``)
-  ``~/.config/virtesk-vdi/virtesk-vm-rollout.conf``
-  ``/etc/virtesk-vdi/virtesk-vm-rollout.conf``

Other configuration files (``Autounattend.xml.template``,
``logging.conf``, ...):

-  Absolute paths are valid
-  all other paths are handled relative to the location of
   ``virtesk-vm-rollout.conf``.

General configuration: section [general]
----------------------------------------

::

    [general]
            sftp_floppy_upload_cmd = "(echo put {0}; echo chmod 666 {1}; echo ls -l {1}) | sftp sftp-floppy-upload@infrastructure-server:/floppy/"
            sftp_floppy_cleanup_cmd = "echo rm {0} | sftp sftp-floppy-upload@infrastructure-server:/floppy/"
            ovirt_worker_floppy_prefix = "/rhev/data-center/mnt/..../floppy"
            [[connect]]
        # ...

Ovirt REST-API connection parameters: section [general][[connect]]
------------------------------------------------------------------

::

    [general]
        # ...
            [[connect]]
            url = "https://ovirt-manager/api"
            username = "admin@internal"
            password = "PASSWORD"
            ca_file = "ca.crt"

-  **url:** URL used to connect to REST-API of your RHEV/oVirt Manager.
-  **username** and **password:** Credentials of some Administrator
   account in Ovirt/RHEV. This account is used for all operations on
   VMs.
-  **ca_file:** Path to CA certificate file. Used for validating
   SSL connection. Can be downloaded from RHEV/Ovirt Manager:
   ``https://ovirt-manager/ca.crt``

Logging: section [logging]
--------------------------

::

    [logging]
    config_file=logging.conf
    log_file=logs/rollout.log

-  **config_file:** Path to ``logging.conf``
-  **log_file:** Path to logfile

Sample ``logging.conf``:

::

    [loggers]
    keys        = root

    [logger_root]
    level       = DEBUG
    handlers    = console,file

    [formatters]
    keys        = simple,complex

    [formatter_simple]
    format      = %(asctime)s - %(name)s - %(levelname)s - %(message)s

    [formatter_complex]
    format      = %(asctime)s - %(name)s - %(levelname)s - %(module)s : %(lineno)d - %(message)s

    [handlers]
    keys        = file,console

    [handler_file]
    class       = FileHandler
    formatter   = complex
    level       = DEBUG
    args        = (r'%(log_file)s', r'%(file_mode)s')

    [handler_console]
    class       = StreamHandler
    formatter   = simple
    level       = DEBUG
    args        = (sys.stdout,)

Details: see
https://docs.python.org/2/library/logging.config.html#configuration-file-format

Room definitions: section [room room01]
---------------------------------------

Structure:

::

    [room room01]  # Defines a virtual room with name "room01"
        [[student_vms]]
        # Complete definition of all VMs

        [[teacher_vms]]
        # Complete definition of all VMs

        [[other_vms]]
        # Complete definition of all VMs

    [room room02]  # Defines a virtual room with name "room02"
        [[vms]]
        # Complete definition of all VMs

Room definition:

::

    [room test01]
        [[student_vms]]
            # Numbering and naming
            ids = "range(1,10+1)"
            names = "${roomname}-vd${id}"
        
            # Network
            network_name="TEST-NET-1"
            ip_addresses = "192.0.2.${suffix}"
            ip_addresses_suffix = 1
            netmask_suffix=24
            default_gateway=192.0.2.1

            # VM
            cluster = Default
            template_name = "win7-goldimage-01"
            memory = 4 * 1024 * 1024 * 1024
            os="windows_7x64"
            description = "student VM"
            timezone="W. Europe Standard Time"
            usb = enabled
            stateless = False

            # Windows Autounattend
            autounattend_templatefile = "Autounattend-production.xml.template"
            workaround_os="rhel_7x64"
            workaround_timezone="Etc/GMT"
        
            # Permissions
            tc_user = "ovirt.thinclient@ourdomain"

            # Snapshots
            snapshot_description = "Automatic snapshot after virtesk-vmrollout, IP=${ip}/${netmask_as_suffix}, scripttime=${scripttime}"
            reset_to_snapshot_regex = "Automatic snapshot after virtesk-vmrollout, .*"

            # Starting VM
            rollout_startvm = True
            reset_startvm = Auto

Numbering and naming:

-  **ids:** List of IDs. Will be passed to ``eval()``, e.g. this can be
   any valid python code.

   -  Every VM has an ID. This is required for computing VM names, IP
      Addresses, ...
   -  Example: ``ids = "[1,2,3,4,5]"`` - 5 VMs.
   -  Example:
      ``ids = "range(1,5+1)"`` - same as ``ids = "[1,2,3,4,5]"``. Please dont forget the ``+1``,
      because the python range()-function does not include the end
      itself in the list.
   -  Number of VMs: ``len(ids)``
   -  Requirement: IDs must be unique with the IDs-List itself. However,
      they don't need to be unique inside the room (different
      subsections can have the same IDs). Also, it is quite common to
      have the same set of IDs for different rooms.

-  **names:** Specifies the Ovirt names of the VMs, and also the Windows
   ComputerName that will be configured during the Autounattend phase.

   -  Examle: ``names = "${roomname}-vd${id}"``
   -  Variable substitution: implemented using `Python template
      strings <https://docs.python.org/2/library/string.html#template-strings>`__
   -  Variable **roomname**: name of the virtual room
   -  Variable **id**: id of the VM. Padded to two digits.

Network:

-  **network\_name:** Name of Ovirt network to attach to VM.
-  **ip\_adresses** and **ip\_adresses\_suffix:** Used to define IP
   addresses for the VMs. IPs are computed, but not used, by
   virtesk-vm-rollout. IPs can be used inside Autounattend.xml to
   configure static IP adresses for VMs.

   -  last IP octet = ip\_adresses\_suffix + id - 1
   -  IP = ip\_adresses, with ``$suffix`` replaced by the last IP octet
      computed above.
   -  Example: ``ids=[1,2,3,4]``, ``ip_adresses=192.0.2.$suffix``,
      ``ip_adresses_suffix=11`` ===> VMs will get the IPs 192.0.2.11,
      192.0.2.12, 192.0.2.13, 192.0.2.14.
   -  Example: ``ids=[1,2,3,4]``, ``ip_adresses=192.0.2.$suffix``,
      ``ip_adresses_suffix=100`` ===> VMs will get the IPs 192.0.2.100,
      192.0.2.101, 192.0.2.102, 192.0.2.103.

-  **netmask\_suffix:** Prefix length of network mask. Passed directly
   to Autounattend.xml.

   -  ``netmask_suffix=24`` ===> same as netmask 255.255.255.0
   -  ``netmask_suffix=21`` ===> same as netmask 255.255.248.0

-  **default\_gateway**: Passed directly to Autounattend.xml

VM:

-  **cluster**: Ovirt cluster for creating VMs.
-  **template\_name**: Name of Ovirt template (e.g. windows goldimage)
   to use for creating VMs
-  **memory**: RAM of virtual machine in Bytes.
-  **os**: Operating System to assign to this VM after Autounattend
   completed. This is the OS as configured in Ovirt (Edit VM dialog).
-  **timezone**: Timezone to assign to this VM after Autounattend
   completed. This is the Timezone as configured in Ovirt (Edit VM
   dialog). Please note: Ovirt uses different timezone names for Linux
   than for Windows VMs.
-  **description**: VM description
-  **usb**: Shall USB be enabled for the VM? Valid values: *enabled*,
   *disabled*
-  **stateless**: Shall the VM be
   `stateless <stateless_and_snapshot_features.html>`__? Valid values:
   *True*, *False*.

Windows Autounattend:

-  **autounattend\_templatefile**: Path to Mako template for
   `Autounattend.xml <autounattend.html>`__.
-  **workaround\_os**: Workaround Operating System to use during
   Autounattend process. Can be any valid OS available in Ovirt.
-  **workaround\_timezone**: Workaround timezone to use during
   Autounattend process. Must be a timezone that is valid for the
   configured *workaround\_os*.

Permissons:

-  **tc\_user**: Technical account used by TCs to access VMs. After
   rollout, permissions are granted to *tc\_user*. Details are
   documented `here <virtesk-tc-connectspice.html>`__

Snapshots:

-  **snapshot\_description**: If empty, no snapshot is created.
   Otherwise, a `snapshot <stateless_and_snapshot_features.html>`__ with
   the configured description is created after rolling out VMs.
-  **reset\_to\_snapshot\_regex**: Must be a valid python regex. Used by
   virtesk-virtroom-reset to identify the
   `snapshot <stateless_and_snapshot_features.html>`__ to reset the VM to.
   The regex is applied to the snapshot description.

Starting VMs:

-  **rollout\_startvm:** If *True*, VMs will be started after rolling
   out a virtual room.
-  **reset\_startvm:** If *Always*, VMs will always be started after
   resetting a virtual room to their snapshots. If *Auto*, VMs will be
   started if they were running before.

VIM: Tipps and Tricks
---------------------

virtesk-vm-rollout.conf will become large when used in real schools.
The following VIM tricks should help to manage this config file
efficently:

New room: Copy-Paste
~~~~~~~~~~~~~~~~~~~~

::

    vim virtesk-vm-rollout.conf

    (navigate to the beginning of an existing room configuration)

    v             # Press v to enter visual mode

    (navigate to the end of the configuration of this room)
    (Press $ to move to the end of a line if nessecary)

    y             # Press y to yank/copy the selected lines

    G             # Move to the end of the file

    O             # Press O and ESC to 
    ESC           # create a new line at the end of the file

    p             # Press p to insert the lines copied before

You might need to adjust the new room configuration afterwards; usually,
the names, the IDs and the network configuration might need attention.

Switching to a new VM Template:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using vim:

::

    # Make Backup
    cp virtesk-vm-rollout.conf virtesk-vm-rollout.conf-backup-$(date +"%Y%m%d-%H%M%S")

    # Edit File
    vim virtesk-vm-rollout.conf

    # Inside VIM: Apppy regular expression
    :%s/template_name = "vdi-template-009"/template_name = "vdi-template-010"/

    # Inside VIM: Write File, Quit
    :wq

Using sed:

::

    # Backup
    cp virtesk-vm-rollout.conf virtesk-vm-rollout.conf-backup-$(date +"%Y%m%d-%H%M%S")

    # Apply regular expression
    sed -i 's/template_name = "vdi-template-009"/template_name = "vdi-template-010"/' virtesk-vm-rollout.conf

    # Compare difference
    diff -puN virtesk-vm-rollout.conf-backup-20160314-182733 virtesk-vm-rollout.conf
