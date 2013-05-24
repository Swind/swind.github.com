.. title: iSCSI REST API 設計                                                                                                                                            
.. slug: iscsi_rest_api
.. date: 2013/03/05 10:16:20
.. tags: FreeBSD
.. link: 
.. description: iSCSI REST API 設計 

Create iSCSI Target
-----------------------------------------

This API is to create a new iSCSI target

:HTTP method: POST
:URI Pattern: /iscsi-targets

Parameters
============================================

:name: iSCSI Target name
:target_alias: iSCSI Target alias
:auth_method: iSCSI target auth method
:use digest: use digest for PDU header and data or not

Example. Create iSCSI Target: JSON Request
============================================

.. code-block:: json

    {
        "iscsi_target":{
            "name": "iqn.2012-05.pi-coral.com.disk1",
            "target_alias": "iSCSI Data Disk1",
            "AuthMethod": "Auto",
            "UseDigest": "Auto"
        }
    }
    

Example. Create iSCSI Target:JSON Response
============================================

.. code-block:: json

    {
        "iscsi_target":{
            "id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
            "links": [
                {
                    "href": "http://localhost:8776/v2/0c2eba2c5af04d3f9e9d0d410b371fde/iscsi-targets/5aa119a8-d25b-45a7-8d1b-88e127885635",
                    "ref": "self"
                },
                {
                    "href": "http://localhost:8776/0c2eba2c5af04d3f9e9d0d410b371fde/iscsi-targets/5aa119a8-d25b-45a7-8d1b-88e127885635",
                    "ref": "self"
                }
            ],
            "name": "iqn.2012-05.pi-coral.com"
        }
    }

Update iSCSI Target
--------------------------------------------

:HTTP method: PUT
:URI Pattern: /iscsi-targets/<iscsi target id>

Parameters
=============================================

:max_connextions:
    Default is 1
    Maximum number of connections requested/acceptable.

:first_burst_length:
    Default is 65536 (64 Kbytes)
    Maximum amount in bytes of unsolicited data an iSCSI initiator may send to the target during the execution of a single SCSI Command.

:max_burst_length: 
    Default is 262411 (256 Kbytes)
    Maximum SCSI data payload in bytes in a Data-In or Data-Out PDU with the F bit set to one.

:max_receive_data_segment_length:
    Default is 8192 bytes.
    Maximuxm data segment length in bytes it can receive in an iSCSI PDU.

:portal_group:
    A portal specifies the IP address and port number to be used for iSCSI connections.

:initiator_group:
    Spetified name allow login/discovery

Example. Update iSCSI Target: JSON Request
============================================
 
.. code-block:: json

    {
        "iscsi_target":{
            "name": "iqn.2012-05.pi-coral.com.disk1",
            "target_alias": "iSCSI Data Disk1",
            "AuthMethod": "Auto",
            "UseDigest": "Auto"
        }
    }

Example. Update iSCSI Target: JSON Response
============================================

Create Portal Group
--------------------------------------------

Example. Create Portal Group: JSON Request
============================================
 
Example. Create Portal Group: JSON Response
============================================

Create Initiator Group
--------------------------------------------

Parameters
=============================================

:comment: 
    The description of this initiator group.

:initiator_name: 
    Spetified name allow login/discovery. Special word "ALL" match all of initiators.

:netmask:
    192.168.2.0/24 means the initiators from 192.168.2.0/24 can login and discovery this target.

Example. Create Initiator Group: JSON Request
=====================================================
 
.. code-block:: json

    {
        "initiator_group":{
            "name": "initiator_group1",
            "comment": "All initiators from 192.168.2.0/24",
            "initiator_name": "All",
            "netmask": "192.168.2.0/24"
        }
    }

Example. Create Initiator Group: JSON Response
======================================================

.. code-block:: json

    {
        "initiator_group":{
            "id": "5aa119a8-d25b-45a7-8d1b-88e127885635",
            "links": [
                {
                    "href": "http://localhost:8776/v2/0c2eba2c5af04d3f9e9d0d410b371fde/iscsi-targets/5aa119a8-d25b-45a7-8d1b-88e127885635",
                    "ref": "self"
                },
                {
                    "href": "http://localhost:8776/0c2eba2c5af04d3f9e9d0d410b371fde/iscsi-targets/5aa119a8-d25b-45a7-8d1b-88e127885635",
                    "ref": "self"
                }
            ],
            "name": "initiator_group1"
        }
    }
