.. _data_management:

=====================
Data management guide
=====================

.. contents::

This page explains where and how to store different types of data on the RHPC cluster. It outlines the purpose of each storage location, what should and should not be stored there, and how to request new space or manage access. Following these conventions keeps the system organized, efficient, and maintainable.

Regardless of where data is stored, it is essential that **all data uploaded to the cluster is fully anonymized**. No personal identifiers should be present. Users are responsible for ensuring their data meets anonymization requirements **before** uploading.

For any folder with restricted access (such as private project or data folders), the **initial owner is responsible** for keeping access up to date and ensuring only appropriate users can view or modify the content. See the section **Managing Data Access** for more details.

Folder Types Overview
=====================

There are three primary storage areas available on the cluster:

1. **Home folders** – For personal configurations, tools, and environments
2. **Project folders** – For project-specific code, results, logs, and configurations
3. **Data folders** – For long-term storage of raw and processed datasets shared across projects

Home Folders
^^^^^^^^^^^^^^^

These personal workspaces are created for each user by default and are accessible across all nodes. They are intended for storing user-specific configuration files, environments, and lightweight tools or scripts used across multiple projects.

**Location**:

    ``/home/<user>``

**Folder setup**:
Created by default for each user with a limit of 150G. Access is limited to the user only. Daily and monthly snapshots are available at:

    ``/home/<user>/.zfs/snapshot``

**Examples of appropriate contents**:

    - Configuration files (e.g., ``.bashrc``, ``.gitconfig``)
    - Miniconda or Python virtual environments
    - Personal utility scripts (e.g., job submission helpers)

**What not to store**:

    - Project-specific scripts, data, or outputs
    - Large files or datasets
    - Intermediate job results or temporary outputs

Project Folders
^^^^^^^^^^^^^^^^^^

These locations support the active development and execution of research projects, including source code, logs, models, and other artifacts tied directly to a specific project.

**Location**:

    ``/projects/<project_name>``

**Folder setup**:
Project folders are created upon request and, by default, are accessible only to the requesting user. The folder owner is responsible for managing access for additional users. If multiple users will be collaborating, refer to the *Managing Data Access* section for guidance on setting correct permissions.

To request a new project folder, post in the Slack channel ``#tech-kosmos-requests`` using the following template:

::

    Project name: <project_name>
    Quota: <storage size>  (default: 100G)
    Requires backups: yes / no
    Owner: <username>
    Additional users (optional): <user1>:<permissions>, <user2>:<permissions>, ...
    Reason (optional):
    <brief justification with a rough estimate or calculation if requesting more than 100G>

**Examples of appropriate contents**:

    - Project-specific codebases
    - Input files like CSVs, JSON configs, and annotations tied to the project
    - Output logs, model checkpoints, and performance metrics

**Use of ``_data`` folder**:
If your project involves large project-specific datasets or generates substantial intermediate data, request a dedicated ``_data`` folder (e.g., ``/projects/<project_name>_data``) to keep it separate from your code and configuration files:

::

    Request: Project data folder for <project_name>
    Quota: <storage size>
    Reason: <brief justification with a rough estimate or calculation>

This folder is for **project-specific data only** — data tailored to or generated for your project and not intended for general reuse. Examples include:

    - nnUNet-style configurations and splits specific to this project
    - Project-specific modified versions of datasets from ``/data``
    - Intermediate data created during preprocessing or transformation steps that take too long to regenerate

This separation helps monitor storage usage and simplify cleanup. Ensure your project folder contains the pipeline or scripts required to regenerate the contents of the ``_data`` folder.

**What not to store**:

    - Shared long-term datasets used by multiple projects
    - Personal configuration files or unrelated utilities

Data Folders
^^^^^^^^^^^^^^^

Data folders store long-term datasets intended for use across multiple projects or users. They are meant for data that should be reusable, traceable, and centrally maintained over time.

**Dataset organization**:
Each dataset should be structured with two main subdirectories:

- ``archive/`` — the original, unmodified data exactly as received
- ``derived/`` — cleaned, reformatted, or annotated versions for reuse

This structure promotes reproducibility and allows teams to build reliably on shared datasets. Detailed guidelines for each of these subdirectories are provided in the following sections.

**Dataset access**:
Datasets may be public or private:

- **Public datasets** are open-access and available to all users. These typically come from public repositories or open collaborations.
- **Private datasets** are access-limited to specific users or groups and may include in-house or licensed data.

Private datasets must be stored under the group directory associated with the dataset owner's primary group. The dataset owner is responsible for maintaining up-to-date permissions. Refer to the *Managing Data Access* section for more information.

**Location**:

    - Public datasets:
        ``/data/groups/public/archive/<dataset_name>/``
        ``/data/groups/public/derived/<dataset_name>/``

    - Private datasets:
        ``/data/groups/<group>/archive/<dataset_name>/``
        ``/data/groups/<group>/derived/<dataset_name>/``

**Folder setup**:
Data folders are created upon request via the ``#tech-kosmos-requests`` Slack channel. Public folders are accessible to all users. Private folders are restricted to authorized users as defined by the dataset owner.

To request a new data folder, use the following template:

::

    Dataset name: <dataset_name>
    Private: yes / no
    Owner: <username>
    Additional users (optional): <user1>:<permissions>, <user2>:<permissions>, ...
    Data description: <short description or link to public dataset>


Archive
"""""""

The ``archive/`` directory stores the raw, unaltered form of a dataset — exactly as it was received or downloaded. This content must remain unchanged to preserve the dataset’s provenance.

**Locations**:

    - ``/data/groups/public/archive/<dataset_name>``
    - ``/data/groups/<group>/archive/<dataset_name>``

**Guidelines**:

    - Do not modify any files in ``archive/``
    - Every dataset in ``archive/`` should have a corresponding ``derived/`` directory if used in processing
    - Data must be reproducible — you should be able to re-download it from the original source (e.g., URL, DOI, accession)

**Examples of appropriate contents**:

    - Datasets from public repositories (e.g., TCIA, PhysioNet)
    - Image annotations bundled with the original dataset
    - Raw CSVs, XMLs, or JSON files from collaborators

**What not to store in archive folders**:

    - Cleaned, renamed, or transformed files
    - Project-specific annotations or outputs
    - Converted file formats (e.g., NIfTI copies of DICOMs)

Derived
"""""""

The ``derived/`` directory contains cleaned, reformatted, or annotated versions of datasets originally stored in ``archive/``. These are meant for cross-project analysis, modeling, or standardized workflows.

**Locations**:

    - ``/data/groups/public/derived/<dataset_name>/<subfolder>``
    - ``/data/groups/<group>/derived/<dataset_name>/<subfolder>``

**Guidelines**:

    - Use clear, separate subfolders for distinct processing steps (e.g., ``converted_nifti``)
    - Avoid mixing unrelated outputs
    - Try to include metadata or scripts to make processing reproducible

**Examples of appropriate contents**:

    - Converted file formats (e.g., DICOM to NIfTI)
    - Additional segmentation masks or annotations

**What not to store in derived folders**:

    - Project-specific logs, results, or temporary files
    - Intermediate data not meant for reuse
    - Personal scripts, tools, or environments

Managing Data Access
============================

Access to private datasets is the responsibility of the initial dataset owner. They must:

    - Ensure the correct users have access
    - Request access updates via ``#tech-kosmos-requests``
    - Communicate clearly when permissions need to be removed or changed

All access changes are applied by the admin team. Users must not attempt to modify folder permissions directly.

To check who currently has access to a folder, use the ``getfacl`` command:

::

    getfacl /path/to/folder

This shows permission entries like:

::

    # file: /path/to/folder
    # owner: user1
    # group: group1
    user::rwx
    user:user2:r-x
    group::r-x
    group:group2:r--
    mask::rwx
    other::---

This means:

- ``user::rwx`` — the owner has full access
- ``user:user2:r-x`` — user2 has read and execute access
- ``group::r-x`` — the default group (group1) has read/execute
- ``group:group2:r--`` — users in group2 have read-only access
- ``other::---`` — others have no access

To request an ACL update, post the following in ``#tech-kosmos-requests``:

::

    Request: ACL update for /data/groups/<group>/<dataset_name> or /projects/<project_name>
    Add users (optional): <user1>:<permissions>, <user2>:<permissions>, ...
    Remove users (optional): <user3>:<permissions>, <user4>:<permissions>, ...


**Permissions**

    - Use standard `r` (read), `w` (write), `x` (execute) flags.
    - Combine as needed (e.g., `rw`, `rwx`).
    - If not specified, default is `rwx`.

Need Help?
==========

If you’re unsure where to store your data or whether it should be public or private, feel free to ask in ``#tech-hpc-cluster`` or reach out to the RHPC admin team on Slack.