=======================================================
Creating and managing storage (home, project, and data)
=======================================================

This guide provides instructions to **create, manage, and control access** to storage directories across `/home`, `/projects`, and `/data` using the provided admin scripts.

Each script handles both **directory structure setup** and **access permissions using ACLs**, ensuring compliance with RHPC data management policies.

------------------------
Where to Run Each Script
------------------------

Use the correct node depending on the type of storage:

- **Run on rhea** for:

  - Home directories (`/home`)
  - Project directories (`/projects`)
  - ACL modifications for home or project folders

- **Run on kronos** for:

  - Data directories (`/data`)
  - ACL modifications for archive/derived folders

---------------------------
Scripts and Their Purpose
---------------------------

create-home
===========

Creates a home directory at `/home/<user>` with private access.

**Run on:** `rhea`

**Usage:**
::

    sudo create-home -u <username>

**Parameters:**

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

create-project
==============

Creates a ZFS-backed project directory in `/project-pool/projects/`.

**Run on:** `rhea`

**Usage:**
::

    sudo create-project -p <project_name> -q <quota> -u <owner> [-a <users>] [-m <mask>]

**Parameters:**

+-------------------------------+-----------+--------------------------------------------------+
| Parameter                     | Required? | Description                                      |
+===============================+===========+==================================================+
| :code:`-p`, :code:`--project` | Yes       | Project name                                     |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-q`, :code:`--quota`   | Yes       | Storage quota (e.g., 100G)                       |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-u`, :code:`--user`    | Yes       | Project owner username                           |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-a`, :code:`--add`     | No        | Additional users to grant access                 |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-m`, :code:`--mask`    | No        | ACL mask value (e.g., :code:`rwx`)               |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-h`, :code:`--help`    | No        | Show help message and exit                       |
+-------------------------------+-----------+--------------------------------------------------+

create-data-dir
===============

Creates paired `archive/` and `derived/` directories under `/data-pool/groups/`, either private or public.

**Run on:** `kronos`

**Usage:**
::

    sudo create-data-dir -u <owner> -d <dataset_name> -g <group> [-a <users>] [-m <mask>] [-p]

**Parameters:**

+-------------------------------+-----------+--------------------------------------------------+
| Parameter                     | Required? | Description                                      |
+===============================+===========+==================================================+
| :code:`-u`, :code:`--user`    | Yes       | Owner of the dataset                             |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-d`, :code:`--dir`     | Yes       | Dataset name                                     |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-g`, :code:`--group`   | Yes       | Group name                                       |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-a`, :code:`--add`     | No        | Additional users to grant access                 |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-m`, :code:`--mask`    | No        | ACL mask value (e.g., :code:`rwx`)               |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-p`, :code:`--public`  | No        | Enables public mode with inherited ACLs          |
+-------------------------------+-----------+--------------------------------------------------+
| :code:`-h`, :code:`--help`    | No        | Show help message and exit                       |
+-------------------------------+-----------+--------------------------------------------------+

delete-project
==============

Deletes a ZFS project dataset from `/project-pool/projects/`.

**Run on:** `rhea`

**Usage:**
::

    sudo delete-project -p <project_name> [--force]

**Parameters:**

+------------------------------+-----------+--------------------------------------------------+
| Parameter                    | Required? | Description                                      |
+==============================+===========+==================================================+
| :code:`-p`, :code:`--project`| Yes       | Project name to delete                           |
+------------------------------+-----------+--------------------------------------------------+
| :code:`-f`, :code:`--force`  | No        | Skip confirmation prompt                         |
+------------------------------+-----------+--------------------------------------------------+
| :code:`-h`, :code:`--help`   | No        | Show help message and exit                       |
+------------------------------+-----------+--------------------------------------------------+

delete-data-dir
===============

Deletes both `archive/` and `derived/` folders for a dataset.

**Run on:** `kronos`

**Usage:**
::

    sudo delete-data-dir -d <path> [--force]

**Parameters:**

+-----------------------------+-----------+--------------------------------------------------------------------------+
| Parameter                   | Required? | Description                                                              |
+=============================+===========+==========================================================================+
| :code:`-d`, :code:`--dir`   | Yes       | Relative or absolute path to :code:`archive/` or :code:`derived/` folder |
+-----------------------------+-----------+--------------------------------------------------------------------------+
| :code:`-f`, :code:`--force` | No        | Skip confirmation prompt                                                 |
+-----------------------------+-----------+--------------------------------------------------------------------------+
| :code:`-h`, :code:`--help`  | No        | Show help message and exit                                               |
+-----------------------------+-----------+--------------------------------------------------------------------------+

modify-acl
==========

Adds or removes user access to any directory using ACLs.

**Run on:** *kronos* or *rhea*

**Usage:**
::

    sudo modify-acl -d <directory> [-a <users>] [-r <users>] [-m <mask>] [--no-counterpart]

**Parameters:**

+-------------------------------+-----------+-----------------------------------------------------------------------+
| Parameter                     | Required? | Description                                                           |
+===============================+===========+=======================================================================+
| :code:`-d`, :code:`--dir`     | Yes       | Path to the directory                                                 |
+-------------------------------+-----------+-----------------------------------------------------------------------+
| :code:`-a`, :code:`--add`     | No        | Users to add to ACL                                                   |
+-------------------------------+-----------+-----------------------------------------------------------------------+
| :code:`-r`, :code:`--remove`  | No        | Users to remove from ACL                                              |
+-------------------------------+-----------+-----------------------------------------------------------------------+
| :code:`-m`, :code:`--mask`    | No        | ACL mask value (e.g., rwx)                                            |
+-------------------------------+-----------+-----------------------------------------------------------------------+
| :code:`-c`, :code:`--check`   | No        | Verify and report current ACL settings                                |
+-------------------------------+-----------+-----------------------------------------------------------------------+
| :code:`--no-counterpart`      | No        | Skip updating paired :code:`archive/` or :code:`derived/` directory   |
+-------------------------------+-----------+-----------------------------------------------------------------------+
| :code:`-h`, :code:`--help`    | No        | Show help message and exit                                            |
+-------------------------------+-----------+-----------------------------------------------------------------------+
