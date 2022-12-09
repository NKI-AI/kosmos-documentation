=================
Slurm usage guide
=================

.. contents::

Introduction
------------

Slurm is an open-source job scheduling system for Linux clusters, most frequently used for high-performance computing (HPC) applications. This guide will cover some of the basics to get started using slurm as a user. For more information, the `Slurm Docs <https://slurm.schedmd.com/documentation.html>`_ are a good place to start.

Slurm is deployed on Kosmos, this means that a slurmd daemon should be running on each compute system. Users do not log directly into each compute system to do their work. Instead, they execute slurm commands (ex: ``srun``\ , ``sinfo``\ , ``scancel``\ , ``scontrol``\ , etc) from a slurm login node. These commands communicate with the slurmd daemons on each host to perform work.

The control node is ``kosmos``. You should login to this node to run any job. Only when a job is running, you can login to the compute nodes, but this shouldn’t be necessary.

Simple Commands
---------------

Cluster state with sinfo
^^^^^^^^^^^^^^^^^^^^^^^^

To "see" the cluster, ssh to the slurm login node for your cluster and run the ``sinfo`` command:

.. code-block:: bash

   $ sinfo
   PARTITION AVAIL  TIMELIMIT   NODES  STATE NODELIST
   batch*       up  7-00:00:00      1    mix aristarchus
   batch*       up  7-00:00:00      1   idle ptolemaeus

There are 2 nodes available on this system, one in\ ``idle`` and one in ``mix`` state, which means the node is partly available. There is a time limit per job of 7 days. This can be changed by an administrator for running jobs. If a node is completely busy, its state will change from ``idle`` to ``alloc``\ :

.. code-block:: bash

   $ sinfo
   PARTITION AVAIL   TIMELIMIT   NODES  STATE NODELIST
   batch*       up   7-00:00:00      1  alloc aristarchus
   batch*       up   7-00:00:00      1   idle ptolemaeus

The ``sinfo`` command can be used to output a lot more information about the cluster. Check out the `sinfo doc for more <https://slurm.schedmd.com/sinfo.html>`_.

Running a job with srun
^^^^^^^^^^^^^^^^^^^^^^^

To run a job, use the ``srun`` command:

.. code-block:: bash

   $ srun hostname
   ptolemaeus

What happened here? With the ``srun`` command we instructed slurm to find the first available node and run ``hostname`` on it. It returned the result in our command prompt. It's just as easy to run a different command that runs a python script or a container using srun.

Most of the time, scheduling on a full system is not necessary and it's better to request only a certain portion of the GPUs:

.. code-block:: bash

   $ srun --gres=gpu:2 env | grep CUDA
   CUDA_VISIBLE_DEVICES=0,1

Or, conversely, sometimes it's necessary to run on multiple systems:

.. code-block:: bash

   $ srun --ntasks 2 -l hostname
   ptolemaeus
   aristarchus

Running an interactive job
^^^^^^^^^^^^^^^^^^^^^^^^^^

Especially when developing and experimenting, it's helpful to run an interactive job, which requests a resource and provides a command prompt as an interface to it:

.. code-block:: bash

   slurm-login:~$ srun --pty /bin/bash
   ptolemaeus~$ hostname
   ptolemaeus
   ptolmaeus:~$ exit

During interactive mode, the resource is being reserved for use until the prompt is exited (as shown above). Commands can be run in succession, and a debugger, e.g. with pycharm can be connected.

Before starting an interactive session with ``srun`` it may be helpful to create a session on the login node with a tool like ``tmux`` or ``screen``. This will prevent a user from losing interactive jobs if there is a network outage or the terminal is closed.

Resource and node management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The default amount of physical CPU memory per CPU code is set to 7000 Megabytes. This value is chosen with having a very memory/CPU intensive task in mind and usually more than the required amount of CPU memory for most tasks. Using this amount (or worse, requesting even more memory per CPU core) can limit the performance of the cluster and deprive the availability of cluster which means higher pending times for all the submitted jobs. Therefore a it is recommended to explicitly request for an amount of the assigned CPU cores and memory that is necessary for the submitted job. A number of 4 to 8 CPU cores per GPU should ideally be enough for most training and inference tasks. Slurm flag ``--cpus-per-task=8`` can be used in order to pass the number of required CPU cores for the submitted job. The flag ``--mem=48G`` can also be used for requesting the amount of memory assigned to the job.

The two cluster nodes ``ptolemaeus`` and ``aristarchus`` have Quadro A6000 GPUs with 48GB of GPU memory. Cluster node ``eudoxus`` has A100 GPUs with 80GB of memory (see `Compute cluster @ NKI (Kosmos) <1984233497.html>`_ for more details). For tasks that do not require a GPUs with more than 48GB of memory, the two former nodes should be used. Slurm command flag ``-w=<list of nodes>`` or ``--nodelist=<list of nodes>`` can be used to make sure Slurm scheduler assigns the job to one of the provided nodes. For example ``--nodelist=ptolemaeus`` will assign the task only to ``ptolemaeus`` node. Alternatively it is also possible to exclude nodes by using ``--exclude=<list of nodes>`` (example: ``--exclude=eudoxus``\ ).

More Advanced Use
-----------------

Run a batch job
^^^^^^^^^^^^^^^

While the ``srun`` command blocks any other execution in the terminal, ``sbatch`` can be run to queue a job for execution once resources are available in the cluster. Also, a batch job will let you queue up several jobs that run as nodes become available. It's therefore good practice to encapsulate everything that needs to be run into a script and then execute with ``sbatch`` vs with ``srun``\ :

.. code-block:: bash

   $ cat script.sh
   #!/bin/bash
   /bin/hostname
   sleep 30
   $ sbatch script.sh

Observing running jobs with ``squeue``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To see which jobs are running in the cluster, use the ``squeue`` command:

.. code-block:: bash

   $ squeue -a -l
   Tue Nov 17 19:08:18 2020
   JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
   9     batch         bash   user01  RUNNING       5:43 UNLIMITED      1 ptolemaeus

To see just the running jobs for a particular user ``USERNAME``\ :

.. code-block:: bash

   $ squeue -l -u USERNAME

Sometimes when the cluster experiences a lot of traffic, your job will not start immediately, but instead it will have to wait until one of the nodes become available. In order to see what is the estimated start time of your jobs, run the following command:

.. code-block:: bash

   $ squeue -u USERNAME --start
   JOBID PARTITION     NAME     USER ST          START_TIME  NODES SCHEDNODES           NODELIST(REASON)
   9527512 gpu_titan     pcam bdolicki PD 2022-06-15T17:25:19      1 r34n4                (Resources)

The ``START_TIME`` column indicates that the job will start on June 15th at 5:25 pm.

SLURM estimates the start time based on the time limits of jobs that are currently running. Given that the time limit is an upper bound after which the job is killed, many jobs might in practice finish long before their time limit which means that your job (which is waiting) might end up starting faster than the initial ``START_TIME``.

Advanced ``squeue`` commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section discusses more advanced ``squeue`` commands which aren't essential for new SLURM users, but can be helpful in certain cases.

Consider this scenario: You just finished writing a job script to train your model for the next 8 hours and submit it to the cluster in the evening, hoping to see its results the next day. However, in the morning you realize there was a small bug in your script that caused the job to fail after 30 seconds and now you have to fix it, resubmit and wait 8 hours again. To prevent such disappointments, it’s good to check if the job is running at least for a couple minutes after submitting to make sure there aren’t any immediate errors. This can be done using the ``watch`` command in Linux that allows to rerun any command in a specified time interval (by default, every 2 seconds). The following will show your running jobs and update every 2 seconds, so if any of your jobs disappears from this view, it means it finished (either completed successfully, or failed):

.. code-block:: bash

   $ watch squeue -u USERNAME
   Every 2.0s: squeue -u b.dolicki                                                                                                                         

                JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
                13015     batch pretrain b.dolick  R   21:01:06      1 ptolemaeus
                13024     batch finetune b.dolick  R   18:55:22      1 aristarchus

When running many jobs it helps to see their names in ``squeue`` to keep track of what you're running. In the view below, the names are truncated. To see full names you can increase the width of particular columns by specifying ``--format``\ :

.. code-block:: bash

   $ squeue -u USERNAME --format="%.18i %.9P %.30j %.8u %.8T %.10M %.15l %.6D %R"             
                JOBID PARTITION                           NAME     USER    STATE       TIME      TIME_LIMIT  NODES NODELIST(REASON)
                13015     batch             pretrain_pcam_moco b.dolick  RUNNING   21:14:58      7-00:00:00      1 ptolemaeus
                13024     batch          finetune_pcam_moco_e2 b.dolick  RUNNING   19:09:14      7-00:00:00      1 aristarchus

To learn more about ``squeue`` check the `official documentation <https://slurm.schedmd.com/squeue.html>`_.

Cancel a job with ``scancel``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To cancel a job, use the ``squeue`` command to look up the JOBID and the ``scancel`` command to cancel it:

.. code-block:: bash

   $ squeue
   $ scancel JOBID

Running an MPI job
^^^^^^^^^^^^^^^^^^

To run a deep learning job with multiple processes, use MPI:

.. code-block:: bash

   $ srun -p PARTITION --pty /bin/bash
   $ singularity pull docker://nvcr.io/nvidia/tensorflow:19.05-py3
   $ singularity run docker://nvcr.io/nvidia/tensorflow:19.05-py3
   $ cd /opt/tensorflow/nvidia-examples/cnn/
   $ mpiexec --allow-run-as-root -np 2 python resnet.py --layers=50 --batch_size=32 --precision=fp16 --num_iter=50

Running using a docker container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This needs to be written, but currently the pyxis is supported, so go ahead and check that out.

Additional Resources
--------------------


* 
  `SchedMD Slurm Quickstart Guide <https://slurm.schedmd.com/quickstart.html>`_

* 
  `LLNL Slurm Quickstart Guide <https://hpc.llnl.gov/banks-jobs/running-jobs/slurm-quick-start-guide>`_
