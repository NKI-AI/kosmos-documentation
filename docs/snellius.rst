.. _snellius:

================
Snellius cluster
================

.. contents::

Introduction
------------

Snellius is the successor of the Lisa cluster, a national shared HPC facility. Snellius provides both CPU and GPU compute nodes and makes use of the slurm scheduling system, just like our Kosmos cluster. All GPU nodes (and there are many) have 8 NVIDIA A100 Tensor Cores, making the Snellius cluster a powerful resource for heavy machine learning tasks. The downside of using Snellius is that it is expensive: 1 hour of usage on 1 GPU core costs roughly 100 Standard Billing Units (SBUs), which amounts to 1 euro. The AI for Oncology group currently has a budget of roughly 3.5 million SBUs, but this is supposed to last us until the end of 2026. Therefore, one should be mindful when allocating resources here, and estimate beforehand how many SBUs a planned job is expected to use up.

Snellius home folders have a 200GB quota. See :ref:`shared-storage` for information on the larger shared storage space available to our group on the Snellius cluster.

We refer to `servicedesk.surf.nl/wiki/display/WIKI/Snellius <https://servicedesk.surf.nl/wiki/display/WIKI/Snellius>`_ for detailed documentation of the Snellius cluster.


.. _shared-storage:

Shared storage
--------------
/projects/0/nksr0630. The purpose of this folder is to be analogous to /data/groups/aiforoncology on kosmos. The total storage is 10TB.


