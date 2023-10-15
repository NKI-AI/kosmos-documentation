============
Introduction
============

.. contents::

The AI for Oncology group has a compute cluster in the NKI data center named Kosmos.

Resource sharing
================

In a typical setting, multiple users will use the same server simultaneously. Therefore, consider the amount of CPU, GPU, and memory you use. For instance on ptolemaeus there are 128 cores (64 effective), 8 GPUs and 512GB of memory. So if you use 1 GPU, you can use up to 16 cores and 64GB of CPU memory to evenly use the available resources.

Kosmos uses scheduling set by slurm. We have a description on :ref:`how to use slurm<slurm-usage-guide>`.

The :ref:`storage<storage>` page mentions the available storage options. On each machine there is a mounted path ``/mnt/`` linking to a file server. This path can be used to store archives which are considered to be *raw data,* for instance, .svs or dicom files. These folders can only be created by admins and are supposed to be read-only. Processed files, or temporary files for which you have a script that converts the raw archive to data used for your model can be stored in your own home directory.

The :ref:`Training/ Inference on Cluster Best Practices<practices>` page mentions best practices for training
or performing inference on the cluster.

Hardware
========

Below lists the current available hardware.

.. _gpu-nodes:

GPU nodes
---------

+--------------+-------------+---------------------------------+------+------+---------+
| PARTITION    | NODE        | GPU TYPE                        | CPUS | GPUS | MEM (G) |
+==============+=============+=================================+======+======+=========+
| a100         | euctemon    | NVIDIA A100 80GB                | 128  | 8    | 980     |
+--------------+-------------+---------------------------------+------+------+---------+
| a100         | eudoxus     | NVIDIA A100 80GB                | 128  | 8    | 980     |
+--------------+-------------+---------------------------------+------+------+---------+
| a6000        | aristarchus | NVIDIA RTX A6000 48GB           | 128  | 8    | 980     |
+--------------+-------------+---------------------------------+------+------+---------+
| a6000        | galileo     | NVIDIA RTX A6000 48GB           | 128  | 8    | 490     |
+--------------+-------------+---------------------------------+------+------+---------+
| a6000        | ptolemaeus  | NVIDIA RTX A6000 48GB           | 128  | 8    | 980     |
+--------------+-------------+---------------------------------+------+------+---------+
| p6000        | mariecurie  | NVIDIA Quadro P6000 24GB        | 40   | 4    | 122     |
+--------------+-------------+---------------------------------+------+------+---------+
| rtx2080ti    | alanturing  | NVIDIA GeForce RTX 2080 Ti 11GB | 40   | 8    | 489     |
+--------------+-------------+---------------------------------+------+------+---------+
| rtx2080ti    | hamilton    | NVIDIA GeForce RTX 2080 Ti 11GB | 40   | 8    | 244     |
+--------------+-------------+---------------------------------+------+------+---------+
| rtx2080ti_sm | carlos      | NVIDIA GeForce RTX 2080 Ti 11GB | 24   | 2    | 91      |
+--------------+-------------+---------------------------------+------+------+---------+
| rtx2080ti_sm | plato       | NVIDIA GeForce RTX 2080 Ti 11GB | 24   | 2    | 122     |
+--------------+-------------+---------------------------------+------+------+---------+
| rtx2080ti_sm | schrodinger | NVIDIA GeForce RTX 2080 Ti 11GB | 24   | 2    | 91      |
+--------------+-------------+---------------------------------+------+------+---------+
| rtx8000      | roentgen    | NVIDIA Quadro RTX 8000 48GB     | 40   | 4    | 244     |
+--------------+-------------+---------------------------------+------+------+---------+



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
     - Installed
     - Status
   * - gaia
     - 256 cpu cores
     - 490GB
     - ±11TB
     - 40 Gbps
     - 2023
     - CPU only node. Access through slurm


The CPU nodes can be used for all kinds of tasks which do not require GPUs, such as preprocessing data, running tensorboard, etc.

Useful command for resources
----------------------------

.. list-table::
   :header-rows: 1

   * - Command
     - Possible flags
     - Description
   * - ``nodestat``
     - ``-j``
     - Gives a list of the nodes and the resources they have/ that are available. Check out :ref:`nodestat-command` for more information.
   * - ``myquota``
     - 
     - Gives a list of storage associated with your account and shows how much space is left

Hardware or configuration errors
================================

If you encounter a problem which is likely due to configuration or hardware failure, you can check in the Slack channel ``#tech-hpc-cluster`` with others, or if you are sure immediately contact the admins: `rhpc-admin@nki.nl <mailto:rhpc-admin@nki.nl>`_.

Installed software
==================

We use `spack <https://spack.readthedocs.io/en/latest/>`_ for package management on RHPC. This is managed by Jonas Teuwen.

Useful commands for Spack
-------------------------

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



