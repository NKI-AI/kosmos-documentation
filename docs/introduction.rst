============
Introduction
============

.. contents::

The AI for Oncology group has a compute cluster in the NKI data center named Kosmos.

Resource sharing
================

In a typical setting, multiple users will use the same server simultaneously. Therefore, consider the amount of CPU, GPU, and memory you use. For instance on ptolemaeus there are 128 cores (64 effective), 8 GPUs and 512GB of memory. So if you use 1 GPU, you can use up to 16 cores and 64GB of CPU memory to evenly use the available resources.

Kosmos uses scheduling set by slurm. We have a description on :ref:`how to use slurm<slurm-usage-guide>`.

The :ref:`storage<storage>` page mentions the available storage options. On each machine there is a mounted path ``/mnt/archives`` linking to a file server. This path can be used to store archives which are considered to be *raw data,* for instance, .svs or dicom files. These folders can only be created by admins and are supposed to be read-only. Processed files, or temporary files for which you have a script that converts the raw archive to data used for your model can be stored in your own home directory.

Typically ``/mnt/archives`` is not fast enough to use to train a model. **This can lead to slowdowns for all users.** Therefore, each machine is equipped with a fast SSD storage in RAID-6, mounted in ``/processing``:


* ``/processing`` is the scratch disk, which is local to the machine

* When you login ``/processing/<username>`` is created and has permissions 700 (no other regular users can read this) for your own user.

* The folder is stored in the environment variable ``SCRATCH``, so no need to hardcode this! You can now rsync to ``$SCRATCH``.

* The ``/processing`` disk will be wiped (oldest files first) if it is getting full!

Make sure to not constantly load large files during training from the network storage (this includes home). Not only will this be slow, but will also impact all other users.

Make sure to not store important training artefacts on the ``/processing`` drive as these can be deleted without warning. Use the project data folders for persistent storage.

The processing folders of the GPU nodes are mounted on eratosthenes in ``/mnt/processing/<hostname>`` so you can inspect the folders without needing to schedule a job one a compute node.

Transferring data before training can be done with ``rsync``. For instance, before training you could execute syncing your source with scratch.

.. code-block:: java

   rsync -avv --info=progress2 --delete <SOURCE> $SCRATCH

Hardware
========

Below lists the current available hardware.

GPU nodes
---------

.. list-table::
   :header-rows: 1
   
   * - Hostname
     - GPUs
     - CPUs
     - Memory
     - Scratch
     - Network
     - Installed
     - Remarks
   * - wallace
     - 8x Quadro RTX8000\ :sup:`1` (48GB)
     - 2x Intel Xeon Gold 6262V(24 cores)
     - 384GB
     - ±13TB
     - 10 Gbps
     - May 2020
     - Not part of KOSMOS
   * - aristarchus
     - 8x Quadro A6000\ :sup:`2` (48GB)
     - 2 x AMD EPYC 7542 (2nd gen)(32 cores)
     - 1TB
     - ±21TB
     - 40 Gbps
     - May 2021
     -
   * - ptolemaeus
     - 8x Quadro A6000\ :sup:`2` (48GB)
     - 2 x AMD EPYC 7542 (2nd gen)(32 cores)
     - 1TB
     - ±21TB
     - 40 Gbps
     - May 2021
     -
   * - eudoxus
     - 8x A100\ :sup:`2` (80GB)
     - 2 x AMD EPYC 7543 SP3 (3rd gen)(32 cores)
     - 1TB
     - ±21TB
     - 40 Gbps
     - April 2022
     -
   * - euctemon
     - 8x A100\ :sup:`2` (80GB)
     - 2 x AMD EPYC 7543 SP3 (3rd gen)(32 cores)
     - 1TB
     - ±21TB
     - 40 Gbps
     - September 2022
     -
   * - plato
     - 2x RTX2080Ti (11GB)
     - 1x i9-7920X CPU @ 2.90GHz (12 cores)
     - 120GB
     - ±8TB
     - 1 Gbps
     - August 2022
     -
   * - schrodinger
     - 2x RTX2080Ti (11GB)
     - 1x i9-7920X CPU @ 2.90GHz (12 cores)
     - 120GB
     - ±8TB
     - 1 Gbps
     - August 2022
     -

*1 Requires CUDA >= 10.*

*2 Requires CUDA >= 11, needs capability sm_86 as the Quadro A6000s and A100s use the Ampere architecture.*


Storage nodes
-------------

.. list-table::
   :header-rows: 1

   * - Hostname
     - Storage
     - Network connection
     - Specifications
     - Software stack
     - Backup
     - Installed
   * - storage01
     - ±261TB
     - 10 Gbps
     - 2x Xeon Silver 4208 - 8 core / 192 GB RAM
     - FreeNAS
     - No
     - February 2021
   * - kronos
     - ±400TB
     - 40 Gbps
     - 2x Xeon Silver 4208 - 8 core / 192 GB RAM
     - TrueNAS
     - Yes, for specific folders
     - July 2022
   * - rhea
     - ±400TB
     - 40 Gbps
     - 2x Xeon Silver 4208 - 8 core / 192 GB RAM
     - TrueNAS
     - Yes, for specific folders
     - July 2022


CPU nodes
---------

.. list-table::
   :header-rows: 1

   * - Hostname
     - CPUs
     - Memory
     - Scratch
     - Network
     - Software stack
     - Installed
     - Status
   * - eratosthenes
     - 2 x AMD EPYC 7402 SP3 24-core 2.8GHz
     - 256GB
     - ±11TB
     - 40 Gbps
     - Ubuntu 20.04Docker\ :sup:`3` / Singularity / Enroot
     - April 2022
     - Main login node

*3 Requires root permissions, can be used through slurm with pyxis.*

The CPU nodes can be used for all kinds of tasks which do not require GPUs, such as preprocessing data, running tensorboard, etc.

Hardware or configuration errors
================================

If you encounter a problem which is likely due to configuration or hardware failure, you can check in the Slack channel ``#tech-hpc-cluster`` with others, or if you are sure immediately contact the admins: `rhpc-admin@nki.nl <mailto:rhpc-admin@nki.nl>`_.

Installed software
==================

We use `spack <https://spack.readthedocs.io/en/latest/>`_ for package management on RHPC. This is managed by Jonas Teuwen.

Useful commands
---------------

.. list-table::
   :header-rows: 1

   * - Command w/ Spack
     - Command w/ Module
     - Description
   * - ``spack find``
     - ``module avail``
     - Show available packages
   * - ``spack load <package>``
     - ``module load <package>``
     - Load the specified package

Installed software
------------------

.. list-table::
   :header-rows: 1

   * - General name
     - Specific installed name
     - Description
   * - pixman
     - pixman@0.40.0
     - Dependency for openslide. Previous versions are buggy
   * - cuda
     - cuda@11.3.0
     - GPU communication
   * - openslide
     - openslide-aifo@3.4.1-nki
     - Software to read whole-slide images.



