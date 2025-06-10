==========================================
Creating and Managing Storage
==========================================

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

**Creates a home directory at `/home/<user>` with private access.**

**Run on:** `rhea`

**Usage:**
::

    sudo create-home -u <username>

**Parameters:**

+------------+-----------+--------------------------------------------+
| Parameter  | Required? | Description                                |
+============+===========+============================================+
| -u         | Yes       | Username for the home dir                  |
+------------+-----------+--------------------------------------------+
| -h, --help | No        | Show help message and exit                 |
+------------+-----------+--------------------------------------------+

- Initializes the user's personal space
- Sets appropriate ownership and permissions
- Access is private (ACL: owner only)

create-project
==============

**Creates a ZFS-backed project directory in `/project-pool/projects/`.**

**Run on:** `rhea`

**Usage:**
::

    sudo create-project -p <project_name> -q <quota> -u <owner> [-a <users>] [-m <mask>]

**Parameters:**

+--------------+-----------+-------------------------------------------------+
| Parameter    | Required? | Description                                     |
+==============+===========+=================================================+
| -p, --project| Yes       | Project name                                    |
+--------------+-----------+-------------------------------------------------+
| -q, --quota  | Yes       | Storage quota (e.g., 100G)                      |
+--------------+-----------+-------------------------------------------------+
| -u, --user   | Yes       | Project owner username                          |
+--------------+-----------+-------------------------------------------------+
| -a, --add    | No        | Additional users to grant access                |
+--------------+-----------+-------------------------------------------------+
| -m, --mask   | No        | ACL mask value (e.g., rwx)                      |
+--------------+-----------+-------------------------------------------------+
| -h, --help   | No        | Show help message and exit                     |
+--------------+-----------+-------------------------------------------------+

create-data-dir
===============

**Creates paired `archive/` and `derived/` directories under `/data-pool/groups/`, either private or public.**

**Run on:** ``kronos``

**Usage:**
::

    sudo create-data-dir -u <owner> -d <dataset_name> -g <group> [-a <users>] [-m <mask>] [-p]

**Parameters:**

+--------------+-----------+-------------------------------------------------------------+
| Parameter    | Required? | Description                                                 |
+==============+===========+=============================================================+
| -u, --user   | Yes       | Owner of the dataset                                        |
+--------------+-----------+-------------------------------------------------------------+
| -d, --dir    | Yes       | Dataset name                                                |
+--------------+-----------+-------------------------------------------------------------+
| -g, --group  | Yes       | Group name or `public` for open access                      |
+--------------+-----------+-------------------------------------------------------------+
| -a, --add    | No        | Additional users to grant access                            |
+--------------+-----------+-------------------------------------------------------------+
| -m, --mask   | No        | ACL mask value (e.g., rwx)                                  |
+--------------+-----------+-------------------------------------------------------------+
| -p, --public | No        | Enables public mode with inherited ACLs                     |
+--------------+-----------+-------------------------------------------------------------+
| -h, --help   | No        | Show help message and exit                                  |
+--------------+-----------+-------------------------------------------------------------+

delete-project
==============

**Deletes a ZFS project dataset from `/project-pool/projects/`.**

**Run on:** `rhea`

**Usage:**
::

    sudo delete-project -p <project_name> [--force]

**Parameters:**

+--------------+-----------+--------------------------------------------+
| Parameter    | Required? | Description                                |
+==============+===========+============================================+
| -p, --project| Yes       | Project name to delete                     |
+--------------+-----------+--------------------------------------------+
| -f, --force  | No        | Skip confirmation prompt                   |
+--------------+-----------+--------------------------------------------+
| -h, --help   | No        | Show help message and exit                 |
+--------------+-----------+--------------------------------------------+

delete-data-dir
===============

**Deletes both `archive/` and `derived/` folders for a dataset.**

**Run on:** `kronos`

**Usage:**
::

    sudo delete-data-dir -d <path> [--force]

**Parameters:**

+--------------+-----------+---------------------------------------------------------+
| Parameter    | Required? | Description                                             |
+==============+===========+=========================================================+
| -d, --dir    | Yes       | Relative or absolute path to archive or derived folder |
+--------------+-----------+---------------------------------------------------------+
| -f, --force  | No        | Skip confirmation prompt                               |
+--------------+-----------+---------------------------------------------------------+
| -h, --help   | No        | Show help message and exit                             |
+--------------+-----------+---------------------------------------------------------+

modify-acl
==========

**Adds or removes user access to any directory using ACLs.**

**Run on:** *kronos* or *rhea*

**Usage:**
::

    sudo modify-acl -d <directory> [-a <users>] [-r <users>] [-m <mask>] [--no-counterpart]

**Parameters:**

+--------------------+-----------+-------------------------------------------------------------+
| Parameter          | Required? | Description                                                 |
+====================+===========+=============================================================+
| -d, --dir          | Yes       | Path to the directory                                       |
+--------------------+-----------+-------------------------------------------------------------+
| -a, --add          | No        | Users to add to ACL                                         |
+--------------------+-----------+-------------------------------------------------------------+
| -r, --remove       | No        | Users to remove from ACL                                    |
+--------------------+-----------+-------------------------------------------------------------+
| -m, --mask         | No        | ACL mask value (e.g., rwx)                                  |
+--------------------+-----------+-------------------------------------------------------------+
| -c, --check        | No        | Verify and report current ACL settings                      |
+--------------------+-----------+-------------------------------------------------------------+
| --no-counterpart   | No        | Skip updating paired `archive/` or `derived/` directory     |
+--------------------+-----------+-------------------------------------------------------------+
| -h, --help         | No        | Show help message and exit                                  |
+--------------------+-----------+-------------------------------------------------------------+

