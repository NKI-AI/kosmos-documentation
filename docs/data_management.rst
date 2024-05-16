.. _data_management:

===============
Data management
===============

.. contents::

Every user of the RHPC cluster has a personal home directory with a 100GB limit. Additionally, users may request project folders for project-specific data.

Personal home directories are located at ``/home/<user>``\ . The home folders are backed up daily and monthly. Backups can be found at ``/home/<user>/.zfs/snapshot``\ . Note that the ``/home/<user>/.zfs``\ folder may not appear when listing the contents of ``/home/<user>``\ , since it is a virtual directory. Simply running 

.. code-block:: bash

    ls /home/<user>/.zfs

will display the contents of this virtual directory. The snapshot directories are read-only, and reflect the state of the user's home folder at the time of the snapshot.

Project folders are located at ``/projects/<project_name>``\ . New project folders can be requested in the #tech-kosmos-requests Slack channel, following the predefined template. When requesting project folders with capacities over 100GB, please provide a detailed reason (ideally involving a rough calculation) for the higher disk space requirements.

By default, no periodic snapshots are made of project folders. Project folder snapshotting is available upon request --- for example, if you are nearing the end of a project and want to guard against accidental loss of vital project data. Such requests can be posted in the #tech-kosmos-requests Slack channel, as a reply to your original project folder request message.

