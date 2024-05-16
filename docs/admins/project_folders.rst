=========================
Creating project folders
=========================

Assessing project folder request
--------------------------------

Project folders are requested by users in the ``#tech-kosmos-requests`` channel using
the following format.

    **Project name:** <project name>

    **Quota:** <desired quota (default 100G)>

    **Does it require backups:** <yes/no (default no)>

    **Username:** <your username>

    Optional:

    **Reason above 100G:** <reason>


Tasks for admins:

    - Is the name sufficiently clear? In other words is it identifiable, recognizable
      and not using unclear abbreviations?
    - If desired quota >100G, is a reasonable explanation provided?
    - Is the backup requirement reasonable?

Creating project folders
-------------------------

Login into rhea, then run:

.. code-block:: bash
   
   sudo create-project -p <folder_name> -u <user> -q <quota>

``create-project`` has a --help function.

Modifying project folders
--------------------------
You can change quotas and permissions for project folders using the following commands:

Changing quotas
^^^^^^^^^^^^^^^^^
To change quotas use

.. code-block:: bash

   zfs set quota=10G project-pool/projects/<project_folder>

Changing permissions
^^^^^^^^^^^^^^^^^^^^^

Change project folder permissions to <group>:

.. code-block:: bash

   getent group <group>
   chgrp <group> /projects/<project_folder>
   chmod -R g+rw /projects/<project_folder>