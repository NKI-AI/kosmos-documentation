================
Getting Started
================

.. contents::
   :depth: 2

This page aims to guide new users through the basics of the Kosmos cluster. The complexity of such a system can sometimes lead to its unintended misuse. Should you find important information missing, or feel that certain rules have not been clearly communicated, please contact us via the #tech-hpc-cluster Slack channel.

General Description
-------------------

The Kosmos cluster is a network of high-performance compute servers, also known as "nodes." These nodes are crucial in the development, training, and testing of innovative AI algorithms. The cluster is primarily used by the AI for Oncology group, led by Jonas Teuwen, and the Radiology group, headed by Regina Beets-Tan.

Equipped with servers featuring high-capacity Graphical Processing Units (GPUs), the cluster excels in deep learning tasks. Additionally, it includes a standard CPU node for data preprocessing and training classical machine learning models.

Resource Sharing and Job Scheduling
------------------------------------

The Kosmos cluster caters to a wide range of users, necessitating the efficient and fair allocation of resources. For this purpose, we use a job scheduler called Slurm. Slurm organizes and allocates computational resources among users and their tasks, queuing jobs and executing them as resources become available. Familiarity with Slurm is vital for effective use of the cluster.

Understanding Slurm Nodes
-------------------------

Understanding the distinction between login nodes and compute nodes in Slurm is crucial. The login node for the Kosmos cluster, named ``kosmos``, is where users submit jobs and manage their tasks. It serves as the access point to the cluster but is not intended for heavy computations. Conversely, compute nodes, like our CPU node ``gaia``, are dedicated to performing intensive computational tasks. Heavy tasks should not be run on the login node to avoid system slowdowns. Misuse of the login node for computationally intensive tasks violates cluster policies and may result in account restrictions or bans. Interactive sessions on compute nodes like ``gaia`` can be requested via Slurm, as detailed in our :ref:`Slurm Usage Guide<slurm-usage-guide>`.

When is a Task "Computationally Intensive"?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The term "computationally intensive" lacks a precise definition, but users are expected to use their judgment. Avoid running tasks such as large data transfers, data preprocessing, or services like VSCode or Tensorboard on the ``kosmos`` login node. While this is not an exhaustive list, activities like copying a handful of small files may be permissible. Use discretion and common sense in these cases. When in doubt: use ``gaia``.

Understanding Local vs. Network Storage
---------------------------------------

On the Kosmos cluster, each user has a personal home folder with a 300GB quota, part of the network storage. This home folder remains consistent across all nodes, meaning its contents are the same regardless of the node you're logged into, achieved by mounting the folders from a networked machine.

The cluster also provides shared network storage locations: `/data` for large, commonly used datasets, and `/projects` for storing and accessing files specific to collaborative projects.

In contrast to network storage, compute nodes on the Kosmos cluster are equipped with "scratch disks" located at `/processing/<username>`. These scratch disks are examples of local storage and are ideal for tasks with heavy data throughput, such as training deep learning models where the same data is repeatedly loaded. Utilizing these scratch disks for such intensive tasks is crucial because relying on network storage for this purpose can overwhelm the network bandwidth, leading to slower performance across all servers for all users. Therefore, it's recommended to use local scratch disks for high-performance computing tasks to maintain optimal network and system efficiency. We refer to :ref:`Cluster best practices`<practices> for a more detailed discussion on when to use local or network storage.

