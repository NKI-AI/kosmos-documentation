===============================================================
Administrator Guide: Managing ACLs for Data and Project Folders
===============================================================

This guide explains how to handle Access Control Lists (ACLs) for data and project
folders using four custom commands: ``create-data-dir``, ``delete-data-dir``,
``create-project`` and ``modify-acl``. These are available on kronos and rhea.
These functions help set and manage permissions efficiently to ensure that
only authorised users have access to private data, maintaining security and
compliance. The owner of each private data and project folder is responsible
for keeping ACLs up to date. If any changes are needed, owners can request
modifications via ``#tech-kosmos-requests``.

Creating Data Directories
^^^^^^^^^^^^^^^^^^^^^^^^^^

To create a new data directory with the appropriate permissions and ACLs, use:

.. code-block:: bash

    create-data-dir -u username -g {radiology|aiforoncology} -d directoryname [-a user1,user2] [-m permission_mask]

Options:

+---------------+---------------------------------------------------------------------+
| ``-u``        | Username of the owner                                               |
+---------------+---------------------------------------------------------------------+
| ``-g``        | Group name (must be radiology or aiforoncology                      |
+---------------+---------------------------------------------------------------------+
| ``-d``        | Name of the directory to create                                     |
+---------------+---------------------------------------------------------------------+
| ``-a``        | (optional) Comma-separated list of additional users to grant access |
+---------------+---------------------------------------------------------------------+
| ``-m``        | (optional) Permission mask (default: rwx if not provided)           |
+---------------+---------------------------------------------------------------------+
| ``-h, help``  + Show help message and exit                                          |
+---------------+---------------------------------------------------------------------+

Examples:

.. code-block:: bash

    # Create a private data directory named projectX for user alice under radiology with default permissions:
    create-data-dir -u alice -g radiology -d projectX

    # Create a directory with specific access for bob and charlie:
    create-data-dir -u alice -g aiforoncology -d projectY -a bob,charlie

    # Create a directory with a custom permission mask:
    create-data-dir -u alice -g radiology -d projectZ -m r-x

Deleting Data Directories
^^^^^^^^^^^^^^^^^^^^^^^^^^

To delete a data directory and its corresponding counterpart directories, use:

.. code-block:: bash

    delete-data-dir -d path_to_directory [-f]

Options:

+--------+----------------------------------------------------------+
| ``-d``            | Absolute or relative path to the directory    |
+--------+----------------------------------------------------------+
| ``-f``            | (optional) Force delete without confimation   |
+--------+----------------------------------------------------------+
| ``-h, --help``    | Show help message and exit                    |
+--------+----------------------------------------------------------+

Examples:

.. code-block:: bash

    # Delete a directory with confirmation:
    delete-data-dir -d /data-pool/groups/beets-tan/archive/projectX

    # Force delete a directory without confirmation:
    delete-data-dir -d /data-pool/groups/aiforoncology/archive/projectY -f

Creating a Project
^^^^^^^^^^^^^^^^^^^

To create a new project by setting up a ZFS dataset with a specified quota, owner, and optional
ACL entries for additional users, use:

.. code-block:: bash

    create-project -p projectname -q quota -u owner [-a additional_users] [-m permission_mask]

Options:

+---------------+------------------------------------------------------------------------+
| ``-p``        | Specify the project name (required)                                    |
+---------------+------------------------------------------------------------------------+
| ``-q``        | Specity the quota for the project (e.g., 10G) (required)               |
+---------------+------------------------------------------------------------------------+
| ``-u``        | Specify the owner user for the project (required)                      |
+---------------+------------------------------------------------------------------------+
| ``-a``        | (optional) Comma-separated list of additional users to add ACL entries |
+---------------+------------------------------------------------------------------------+
| ``-m``        | (optional) Permission mask for additional users (default: rwx)         |
+---------------+------------------------------------------------------------------------+
| ``-h, --help``| Show help message and exit                                             |
+---------------+------------------------------------------------------------------------+

Examples:

.. code-block:: bash

        # Create a project with the required parameters:
        create-project -p projectX -q 10G -u alice

        # Create a project with additional ACL entries for extra users with read and execute permission:
        create-project -p projectY -q 20G -u bob -a alice,carol -m r-x

Modifying ACLs
^^^^^^^^^^^^^^

To modify ACLs for directories, use:

.. code-block:: bash

    modify-acl [OPTIONS]

Options:

+-----------------------+------------------------------------------------------------------------+
| ``-d``                | Set the target directory (required)                                    |
+-----------------------+------------------------------------------------------------------------+
| ``-a``                | Add users (comma-separated) with specific permissions                  |
+-----------------------+------------------------------------------------------------------------+
| ``-r``                | Remove users (comma-separated) from ACL                                |
+-----------------------+------------------------------------------------------------------------+
| ``-m``                | Specify the permission mask (e.g., rwx)                                |
+-----------------------+------------------------------------------------------------------------+
| ``-c``                | Check ACL consistency across counterpart directories                   |
+-----------------------+------------------------------------------------------------------------+
| ``--no-recursive``    | Apply ACL changes without recursion                                    |
+-----------------------+------------------------------------------------------------------------+
| ``-h, --help``        | Show help message and exit                                             |
+-----------------------+------------------------------------------------------------------------+

Examples:

.. code-block:: bash

        # Add users david and eva with full permissions to projectX:
        modify-acl -d /data-pool/groups/beets-tan/archive/projectX -a david,eva -m rwx

        # Remove user frank from ACL of projectY:
        modify-acl -d /data-pool/groups/aiforoncology/archive/projectY -r frank

        # Modify ACL without recursion:
        modify-acl -d /data-pool/groups/beets-tan/archive/projectZ -a george -m r-- --no-recursive

        # Check ACL consistency across archive and derived directories:
        modify-acl -d /data-pool/groups/beets-tan/archive/projectX -c

Note: If the directory is within archive or derived, ACL modifications also apply to its counterpart.