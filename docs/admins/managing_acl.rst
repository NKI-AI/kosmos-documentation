=======================================================
Creating and managing storage (home, project, and data)
=======================================================

This guide provides instructions to **create, manage, and control access** to storage directories across ``/home``, ``/projects``, and ``/data`` using the provided admin scripts.

Each script handles both **directory structure setup** and **access permissions using ACLs**, ensuring compliance with RHPC data management policies. All scripts use a centralized ACL management system for consistency and reliability.

------------------------
Where to Run Each Script
------------------------

Use the correct node depending on the type of storage:

- **Run on rhea** for:

  - Home directories (``/home``)
  - Project directories (``/projects``)
  - ACL modifications for home or project folders

- **Run on kronos** for:

  - Data directories (``/data``)
  - ACL modifications for archive/derived folders

---------------------------
Scripts and examples
---------------------------

Create home directory
=========================

Creates a home directory at ``/home/<user>`` with private access.

**Run on:** ``rhea``

**Usage:**
::

    sudo create-home -u <username>

**Options:**

+----------------------------+-----------+--------------------------------------------------+
| Parameter                  | Required? | Description                                      |
+============================+===========+==================================================+
| :code:`-u`                 | Yes       | Username for the home dir                        |
+----------------------------+-----------+--------------------------------------------------+
| :code:`-h`, :code:`--help` | No        | Show help message and exit                       |
+----------------------------+-----------+--------------------------------------------------+

- Initializes the user's personal space
- Sets appropriate ownership and permissions
- Access is private (ACL: owner only)

Create project folder
=======================

Creates a ZFS-backed project directory in ``/project-pool/projects/``.

**Run on:** ``rhea``

**Usage:**
::

    sudo create-project -p projectname -q quota -u owner [OPTIONS]

**Options:**

+----------------------------------+-----------+--------------------------------------------------------------+
| Parameter                        | Required? | Description                                                  |
+==================================+===========+==============================================================+
| :code:`-p`, :code:`--project`    | Yes       | Project name                                                 |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-q`, :code:`--quota`      | Yes       | Storage quota (e.g., 100G)                                   |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-u`, :code:`--user`       | Yes       | Project owner username                                       |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-a`, :code:`--add-users`  | No        | Users with optional permissions (e.g., user1:rwx,user2:r-x)  |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-A`, :code:`--add-groups` | No        | Groups with optional permissions (e.g., group1:rx,group2)    |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-m`, :code:`--mask`       | No        | Default permission mask                                      |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-h`, :code:`--help`       | No        | Show help message and exit                                   |
+----------------------------------+-----------+--------------------------------------------------------------+


**Examples:**

Create a project with the required parameters:

::

    sudo create-project -p projectX -q 10G -u alice

Create a project with additional users using granular permissions:

::

    sudo create-project -p projectY -q 20G -u bob -a alice:rwx,carol:r-x

Create a project with additional groups using shared permissions:

::

    sudo create-project -p projectZ -q 15G -u alice -A researchers,students:rx

Create a project with additional users using global mask:

::

    sudo create-project -p projectW -q 25G -u bob -a alice,carol -m rw-

Deleting project folders
========================

Deletes a ZFS project dataset from `/project-pool/projects/`.

**Run on:** `rhea`

**Usage:**
::

    sudo delete-project -p <project_name> [--force]

**Options:**

+----------------------------------+-----------+--------------------------------------------------------------+
| Parameter                        | Required? | Description                                                  |
+==================================+===========+==============================================================+
| :code:`-p`, :code:`--project`    | Yes       | Project name to delete                                       |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-f`, :code:`--force`      | No        | Skip confirmation prompt                                     |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-h`, :code:`--help`       | No        | Show help message and exit                                   |
+----------------------------------+-----------+--------------------------------------------------------------+


Creating data directories
=========================

Creates paired `archive/` and `derived/` directories under `/data-pool/groups/`, either private or public.

**Run on:** `kronos`

**Usage:**

To create a new data directory with the appropriate permissions and ACLs, use:

::

    create-data-dir -u username -g {radiology|aiforoncology|nkiai} -d directoryname [OPTIONS]

**Options:**

+----------------------------------+-----------+--------------------------------------------------------------+
| Parameter                        | Required? | Description                                                  |
+==================================+===========+==============================================================+
| :code:`-u`, :code:`--user`       | Yes       | Username of the owner                                        |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-g`, :code:`--group`      | Yes       | Group name (``radiology``, ``aiforoncology``, or ``nkiai``)  |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-d`, :code:`--dir`        | Yes       | Name of the directory to create                              |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-a`, :code:`--add-users`  | No        | Users with optional permissions (e.g., user1:rwx,user2:r-x)  |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-A`, :code:`--add-groups` | No        | Groups with optional permissions (e.g., group1:rx,group2)    |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-m`, :code:`--mask`       | No        | Default permission mask                                      |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-p`, :code:`--public`     | No        | Place directories under public instead of group subfolder    |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-s`, :code:`--subdir`     | No        | Subdirectory to create between group and directory name      |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-h`, :code:`--help`       | No        | Show help message and exit                                   |
+----------------------------------+-----------+--------------------------------------------------------------+

**Group Logic:**

**Non-public directories:**

- **Default ownership**: ``owner:owner`` (user owns, user's primary group owns)
- **With additional groups (-A)**: First group becomes the group owner, remaining groups get ACL access
- **All groups (ownership + additional)**: Get permissions specified by the mask

**Public directories (-p):**

- **Group ownership**: Uses the ``-g`` group (mapped to actual Linux group)
- **Additional users/groups ignored**: Permissions handled by inheritance from parent directory
- **Subdirectories supported**: Use ``-s subdir`` or include path in directory name (``subdir/dirname``)

**Examples:**

Create a private data directory named ``projectX`` for user ``alice`` under ``radiology``:

::

    sudo create-data-dir -u alice -g radiology -d projectX

Create a directory with additional users using granular permissions:

::

    sudo create-data-dir -u alice -g aiforoncology -d projectY -a bob:rwx,charlie:r-x

Create a directory with additional groups using shared permissions:

::

    sudo create-data-dir -u alice -g radiology -d projectZ -A researchers,students:rx

Create a directory with additional users using global mask:

::

    sudo create-data-dir -u alice -g aiforoncology -d projectW -a bob,charlie -m rw-

Create a directory with subdirectories (two methods):

::

    sudo create-data-dir -u alice -g radiology -d lung-study -s projects
    sudo create-data-dir -u alice -g radiology -d projects/lung-study

Create a public directory:

::

    sudo create-data-dir -u alice -g radiology -d open-data -p

Deleting data directories
==========================

Deletes both archive and derived directories. Supports subdirectories.

**Run on:** `kronos`

**Usage:**

::

    sudo delete-data-dir -d <path_to_directory> [-f]

**Options:**

+----------------------------------+-----------+--------------------------------------------------------------+
| Parameter                        | Required? | Description                                                  |
+==================================+===========+==============================================================+
| :code:`-d`, :code:`--dir`        | Yes       | Absolute or relative path to the directory                   |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-f`, :code:`--force`      | No        | Force delete without confirmation                            |
+----------------------------------+-----------+--------------------------------------------------------------+
| :code:`-h`, :code:`--help`       | No        | Show help message and exit                                   |
+----------------------------------+-----------+--------------------------------------------------------------+

**Examples:**

Delete a directory with confirmation:

::

    sudo delete-data-dir -d /data-pool/groups/radiology/archive/projectX

Delete a directory with subdirectories:

::

    sudo delete-data-dir -d /data-pool/groups/radiology/archive/projects/lung-study

Force delete a directory without confirmation:

::

    sudo delete-data-dir -d /data-pool/groups/aiforoncology/archive/projectY -f

Delete using relative paths:

::

    sudo delete-data-dir -d ./radiology/archive/projectX --force

Modifying ACLs
==================

Modify ACLs for directories, with support for both users and groups.

**Run on:** ``kronos`` or ``rhea``

**Usage:**

::

    sudo modify-acl -d <directory> [OPTIONS]

**Options:**

+-------------------------------------+-----------+------------------------------------------------------------------------+
| Parameter                           | Required? | Description                                                            |
+=====================================+===========+========================================================================+
| :code:`-d`, :code:`--dir`           | Yes       | Target directory to modify ACLs                                        |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-a`, :code:`--add-users`     | No        | Users with optional permissions (e.g., user1:rwx,user2:r-x)            |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-A`, :code:`--add-groups`    | No        | Groups with optional permissions (e.g., group1:rx,group2)              |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-r`, :code:`--remove-users`  | No        | Remove users from ACL                                                  |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-R`, :code:`--remove-groups` | No        | Remove groups from ACL                                                 |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-m`, :code:`--mask`          | No        | Default permission mask                                                |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-c`, :code:`--check`         | No        | Check ACL consistency across counterpart directories                   |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`--no-recursive`              | No        | Apply ACL changes without recursion                                    |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`--no-counterpart`            | No        | Do not apply changes to archive/derived counterpart                    |
+-------------------------------------+-----------+------------------------------------------------------------------------+
| :code:`-h`, :code:`--help`          | No        | Show help message and exit                                             |
+-------------------------------------+-----------+------------------------------------------------------------------------+

**Features:**

- **Centralized ACL management**: All scripts use the same ``modify-acl`` engine for consistent behavior
- **User access warnings**: Automatically checks if added users can actually access the directory
- **Counterpart synchronization**: Same ACL operations applied to both archive and derived directories
- **Group support**: Full support for adding/removing group ACLs with granular permissions
- **Intelligent group ownership**: When adding groups to ``owner:owner`` directories, first group becomes the new owner group
- **Group removal handling**: When removing the owner group, reverts to ``owner:owner`` or promotes next available group
- **Permission normalization**: Automatically normalizes permissions to valid 3-character format (e.g., ``rw`` → ``rw-``)
- **Recursive operations**: All ACL changes applied recursively by default (use ``--no-recursive`` to disable)

**Examples:**

Add users with granular permissions:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectX -a david:rwx,eva:r-x

Add groups with shared permissions:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectX -A researchers,students:rx

Add groups with mixed permissions:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectX -A researchers:rw,students:rx,admins

Add users with global mask:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectX -a david,eva -m rwx

Remove user ``frank`` from ACL:

::

    sudo modify-acl -d /data-pool/groups/aiforoncology/archive/projectY -r frank

Remove groups from ACL:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectX -R researchers,students

Modify ACL without recursion and without affecting counterpart:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectZ -a george:r-- --no-recursive --no-counterpart

Apply ACLs non-recursively (only to specified directory):

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectZ -a alice:rw,bob:r --no-recursive

Check ACL consistency across archive and derived directories:

::

    sudo modify-acl -d /data-pool/groups/radiology/archive/projectX -c

**Note:**

- If the directory is within ``archive`` or ``derived``, ACL modifications also apply to its counterpart unless ``--no-counterpart`` is used.
- The script will warn if added users cannot access the directory due to missing parent directory permissions.
- Both relative and absolute paths are supported.

---------------------------
Permission and Group Logic
---------------------------

**Data Directory Group Behavior:**

**Non-public directories:**

- **Default ownership**: ``owner:owner`` (user owns, user's primary group owns)
- **With additional groups (-A)**: First group becomes the group owner, remaining groups get ACL access
- **Additional users (-a)**: Get ACL access with specified permission mask
- **All groups (ownership + additional)**: Get permissions specified by the mask
- **Modular ACL management**: Uses centralized ``modify-acl`` engine for all ACL operations

**Public directories (-p):**

- **Group ownership**: Uses the ``-g`` group (mapped to actual Linux group)
- **Additional users/groups ignored**: Permissions handled by inheritance from parent directory

**Project Directory Group Behavior:**

- **Default ownership**: ``owner:owner`` (user owns, user's primary group owns)
- **With additional groups (-A)**: First group becomes the group owner, remaining groups get ACL access
- **Additional users (-a)**: Get ACL access with specified permission mask
- **All groups (ownership + additional)**: Get permissions specified by the mask
- **Modular ACL management**: Uses the same ``modify-acl`` engine as data directories for consistent behavior

**Permission Masks:**

- ``rwx`` - Read, write, execute (full access)
- ``rw-`` - Read, write (no execute) - automatically normalized from ``rw``
- ``r-x`` - Read, execute (no write) - automatically normalized from ``rx``
- ``r--`` - Read only - automatically normalized from ``r``
- ``---`` - No access (useful for removing permissions)

**Note:** The system automatically normalizes permissions to the standard 3-character format. For example, ``rw`` becomes ``rw-``, ``r`` becomes ``r--``, etc.

**Subdirectory Support:**

Both ``create-data-dir`` and ``delete-data-dir`` support subdirectories:

- Use ``-s subdir`` flag to specify subdirectory
- Or include path in directory name: ``subdir/dirname``
- Works with both relative and absolute paths for deletion


----------------------------------------
Tape backup for homefolders and projects
----------------------------------------

For homefolders snapshots are taken weekly and monthly, these snapshots are being written to tape in irregular intervals. To initiate a backup ask Ameer first. Previous backup overview:

+---------------------+---------------------------------------------+---------------------------------------+
| Date                | What                                        | Notes                                 |
+=====================+=============================================+=======================================+
| 23-09-2025          | Full backup of homefolders                  | Not confirmed done by Ameer           |
| 04-10-2024          | Snapshots of homefolders and projects < 1TB | Confirmed and written to tape         |
+---------------------+---------------------------------------------+---------------------------------------+
