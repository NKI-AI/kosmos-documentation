==============
Mounting drives
==============

Mount /processing drive to the login node
^^^^^^^^
Procedure to add /processing of <server_name> to /mnt/processing/<server_name>, so it can be accessed from the login node. 

.. code-block:: text

    ssh rhpc-<server_name>
    sudo nano /etc/exports

To give permission to access the /processing, we have to add to /etc/exports the following text: 

``/processing eratosthenes(rw,sync,no_subtree_check)``. 

To export the new file run:

.. code-block:: text
    
    sudo exportfs -rav

Next, we have to access the login node to add the mount

.. code-block:: text

    ssh rhpc-eratosthenes
    mkdir /mnt/processing/<server_name>
    sudo nano /etc/fstab
    
Here we have to add the following just above # BEGIN ANSIBLE MANAGED JOB

``<server_name>:/processing    /mnt/processing/<server_name>        nfs rsize=524288,wsize=524288,timeo=30,intr``

Subsequently mount the drive.

.. code-block:: text

    sudo mount -a
