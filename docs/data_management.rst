.. _data_management:

===============
Data management
===============
.. contents::


Home folders
------------
Every user of the RHPC cluster has a personal home directory with a 100GB limit. Additionally, users may request project
folders for project-specific data.

Personal home directories are located at ``/home/<user>``\ . The home folders are backed up daily and monthly. Backups
can be found at ``/home/<user>/.zfs/snapshot``\ . Note that the ``/home/<user>/.zfs``\ folder may not appear when
listing the contents of ``/home/<user>``\ , since it is a virtual directory. Simply running

.. code-block:: bash

    ls /home/<user>/.zfs

will display the contents of this virtual directory. The snapshot directories are read-only, and reflect the state of
the user's home folder at the time of the snapshot.

Project folders
----------------

Project folders are located at ``/projects/<project_name>``\ . New project folders can be requested in the
#tech-kosmos-requests Slack channel, following the predefined template (more information in the #tech-kosmos-requests
channel in Slack.). When requesting project folders with capacities over 100GB, please provide a detailed reason
(ideally involving a rough calculation) for the higher disk space requirements.

By default, no periodic snapshots are made of project folders. Project folder snapshotting is available upon request
--- for example, if you are nearing the end of a project and want to guard against accidental loss of vital project data.
Such requests can be posted in the #tech-kosmos-requests Slack channel, as a reply to your original project folder
request message.

Data folders
------------
Data folders are located at ``/data`` . Data folders are intended for storing public and private datasets, intended for
use on the cluster. Permission is managed by Access Control Lists (ACLs) which provide more fine-grained control over
file and directory permissions, ensuring data is accessible only to authorized individuals. These ACLs can be set for
each file and directory on the server, allowing the owner to specify detailed permissions for users and groups based on
access requirements.

Public data
^^^^^^^^^^^
For directories containing public data, all groups should have default access. New public datasets must be added to
``/data/groups/public/``: use ``archive`` for unaltered datasets and ``derived`` for modified datasets.
Normally, the dataset and its contents are automatically assigned the correct ACLs, allowing access to anyone in the
appropriate group. If there is an issue with this process, use the following command to set the ACLs manually:

.. code-block::

        # Manually set the ACLs for new public data recursively
        setfacl -R -m mask::rwx -m user::rwx -m group::rwx -m
        group:radiology:rwx -m group:teuwen-group:rwx -m other::--- -m
        default:mask::rwx -m default:user::rwx -m default:group::r-x -m
        default:group:radiology:rwx -m default:group:teuwen-group:rwx -m
        default:other::--- /data/groups/public/<archive OR derived>/<new_dataset>

Private data
^^^^^^^^^^^^
For private data directories, access is more complex and governed by IRB guidelines. Private datasets should be added to
the appropriate ``/data/groups/<group>/`` directory, with ACLs set to reflect authorized access.

Unaltered datasets, so raw data as received from the datadesk, should be saved in ``/data/groups/<group>/archive``.

For processed datasets please consider if they need saving or can easily be reprocessed from the original. If a
processed version needs to be saved they should be organized in version-specific directories. These directories are saved
in ``/data/groups/<group>/<project-name>/<version-name>``.


By default, newly added
private datasets should restrict access to the owner only. The owner is responsible for adjusting the ACLs to grant
access to other approved users. It is the directory owner's responsibility to maintain and update the ACL to ensure that
only authorized individuals have access.

Here are some basic commands that may be useful during the process:

- Viewing current ACL with ``getfacl``:

.. code-block::

        getfacl /path/to/directory

- Add or remove user access recursively:

.. code-block::

        setfacl -R -m u:<username>:rwx /path/to/directory
        setfacl -R -x u:<username> /path/to/directory

- Add or remove group access recursively:

.. code-block::

        setfacl -R -m g:<groupname>:rwx /path/to/directory
        setfacl -R -x g:<groupname> /path/to/directory

- Set default ACL that will be inherited by new files created in the directory recursively:

.. code-block::

        setfacl -R -m d:u:<username>:rwx -m d:g:<groupname>:rwx /path/to/directory

Here is an end-to-end example of adding a new private dataset, ensuring that it is accesible by two specific users (in
addition to the owner), and applying the appropriate ACLs:

.. code-block::

        # 1. Create the directory for the new dataset
        mkdir -p /data/groups/<group>/<new_dataset>

        # 2. Upload the dataset
        # Add data in appropriate way

        # 3. Set ACLs to grant access to bob and carol, and restrict access for others, while setting these as the default for new files created in the dataset
        setfacl -R -m u:bob:rwx -m u:carol:rwx -m g::--- -m o::--- -m d:u:bob:rwx -m d:u:carol:rwx -m d:g::--- -m d:o::--- /data/groups/<group>/<new_dataset>

        # 4. Verify the ACL settings
        getfacl /data/groups/<group>/<new_dataset>

The resulting ACL wil indicatie that only the owner, Bob, and Carol have access to the directory and its contents:

.. code-block::

        # file: /data/groups/<group>/<new_dataset>
        # owner: <owner>
        # group: <owner>
        user::rwx
        user:bob:rwx
        user:carol:rwx
        group::---
        mask::rwx
        other::---

        default:user::rwx
        default:user:bob:rwx
        default:user:carol:rwx
        default:group::---
        default:mask::rwx
        default:other::---
