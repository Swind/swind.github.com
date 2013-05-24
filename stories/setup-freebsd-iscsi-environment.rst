.. title: Setup iSCSI in FreeBSD 
.. slug: setup-iscsi-in-freebsd
.. date: 2013/05/21 18:00:00
.. tags: FreeBSD
.. link: 
.. description: How to setup the iSCSI initiator in FreeBSD 

PiSCSI Config File
=============================================

Global Environment Variables

.. code-block:: kconfig

    version = "1.0"

    global:
    {
        #Global Debug Level: ERROR, WARN, INFO, DEBUG, NONE
        debug_level = "DEBUG";
        socket_buffer = "1048576";
    }

    logical_unit:
    {
        debug_level = "WARN";

        #Log the scsi command but exclude include READ and WRITE.
        log_scsi_command = true;
    }
    
Portal

.. code-block:: kconfig

    portal_groups:
    (
        {
            Name        = "Group1";
            Portals     =
            (
                "192.168.1.157:3260"
            );
        }
    );

Logical Units

.. code-block:: kconfig

    logical_units:
    (
        {
            Description = "Target 1";
            TargetName = "iqn.2007-09.jp.ne.peach.istgt:disk1";
            TargetAlias = "Test Target 1";
            PortalGroup = "Group1";
            Luns = 
            (
                {
                    Id = 1;
                    Path = "/dev/da1";
                    Size = "1024 MB";
                    Type = "Pass";
                }
            );
        }
    )
