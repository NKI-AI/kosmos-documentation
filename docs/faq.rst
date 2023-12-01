.. _faq:

==========================
Frequently asked questions
==========================

.. contents::

What resources is a slurm job using?
""""""""""""""""""""""""""""""""""""

.. code-block:: bash

   scontrol show job <jobid>

What resources are in use on a slurm node?
""""""""""""""""""""""""""""""""""""""""""

.. code-block:: bash
   
   scontrol show node <nodename>

I'm seeing "Disk quota exceeded" when writing files. What's wrong?
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The file system you're writing to is full. Check the quota of this file system using:

.. code-block:: bash

   df -h .

and subsequently, check what's eating up disk space by running

.. code-block:: bash

   du -h -d1 .

in the root directory of this project or home folder. Now delete stuff accordingly. Due to automatic backups, it may be the case that you'll see a temporary reduction in available disk space when running 

.. code-block:: bash

   df -h .

again. We're phasing out this behavior, but in case you're still running into the issue, please contact #tech-hpc-cluster.


How to view the available qos options for my account?
""""""""""""""""""""""""""""""""""""

.. code-block:: bash

   sacctmgr show assoc where user=<user.id>
