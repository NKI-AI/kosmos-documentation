.. _slurm-usage-guide:

=================
Slurm usage guide
=================

.. contents::

Introduction
------------

Slurm is an open-source job scheduling system for Linux clusters, most frequently used for high-performance computing (HPC) applications. This guide will cover some of the basics to get started using slurm as a user. For more information, the `Slurm Docs <https://slurm.schedmd.com/documentation.html>`_ are a good place to start.

Slurm is deployed on Kosmos, this means that a slurm daemon should be running on each compute system. Users do not log directly into each compute system to do their work. Instead, they execute slurm commands (ex: ``srun``\ , ``sinfo``\ , ``scancel``\ , ``scontrol``\ , etc) from a slurm login node. These commands communicate with the slurmd daemons on each host to perform work.

The control node is ``kosmos``. You should login to this node to run any job. Only when a job is running, you can login to the compute nodes, but this shouldn’t be necessary.


Node Partition and GPU Configuration
------------------------------------
The following table provides a summary of the node partitions and their corresponding GPU/CPU configurations within the cluster:

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
| cpu          | gaia        | \-                              | 256  | \-   | 490     |
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

SLURM QoS Specifications for Our Cluster
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. _slurm-qos-options:

If you encounter errors upon launching new jobs with an invalid QoS specification, add the following options to your `srun` command:

+---------------------+---------------------+-----------------+
| Partition           | QoS Specification   | Maximum GPUs    |
+=====================+=====================+=================+
| A6000               | --qos=a6000_qos     | 2               |
+---------------------+---------------------+-----------------+
| A100                | --qos=a100_qos      | 2               |
+---------------------+---------------------+-----------------+
| All Other Partitions| --qos=rtx_qos       | 4               |
+---------------------+---------------------+-----------------+

For the A6000 partition, use the `--qos=a6000_qos` option, which allows a maximum of 2 GPUs.
Similarly, for the A100 partition, use `--qos=a100_qos` with a maximum of 2 GPUs.
For all other partitions, including RTX, use `--qos=rtx_qos` and a maximum of 4 GPUs.

These QoS specifications ensure that your jobs are properly assigned to the appropriate partitions with the correct
number of GPUs.

Note: Please be aware that there may exist other QoS specifications specific to certain users or groups.
If you belong to such a user group, please consult the guidelines provided by your system administrators
to determine the appropriate QoS specification to use for your jobs.

SLURM Job Submission Options
----------------------------
.. _slurm-options:

The following is a list of commonly used SLURM arguments for job submissions:

- ``--job-name=<name>``:
    Specifies the name for the job.
- ``--partition=<partition>``:
    Specifies the partition to run the job on.
- ``--nodes=<num>``:
    Specifies the number of nodes to allocate for the job.
- ``--nodelist=<node1,node2,...>``:
    Specifies a comma-separated list of specific nodes to run the job on.
- ``--ntasks=<num>``:
    Specifies the total number of tasks to run for the job.
- ``--ntasks-per-node=<num>``:
    Specifies the number of tasks to run per node.
- ``--cpus-per-task=<num>``:
    Specifies the number of CPUs to allocate per task.
- ``--gpus=<resource>``:
    Specifies GPUs number. This cannot be higher than the specific `--qos` configuration.
- ``--mem=<memory>``:
    Specifies the memory requirement for the job.
- ``--time=<time>``:
    Specifies the maximum run time for the job. Note that in our cluster the maximum allowed time can be 7:00:00 days.
    You can contact a system admin for an increase if necessary.
- ``--output=<file>``:
    Specifies the file to which the standard output will be written.
- ``--error=<file>``:
    Specifies the file to which the standard error will be written.
- ``--account=<account>``:
    Specifies the account to charge the job's resource usage.
- ``--qos=<quality_of_service>``:
    Specifies the Quality of Service for the job, affecting priority or resource allocation.
    In our cluster this would affect the number of maximum gpus that can be allocated in each partition.
    For advice on qos specifications, refer to :ref:`SLURM QoS Specifications for Our Cluster<slurm-qos-options>`


Consult your system's admins or documentation guides for issued with your SLURM configuration.

SLURM Job Submission Commands
-----------------------------

Running an Interactive Job with ``srun``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Especially when developing and experimenting, it's helpful to run an interactive job, which requests a resource
and provides a command prompt as an interface to it.

During interactive mode, the resource is reserved for use until the prompt is exited (as shown above).
Commands can be run in succession, and a debugger, such as PyCharm, can be connected.

Example using SLURM arguments:

.. code-block:: bash

    <user>@kosmos:~$ srun --partition=<partition> --nodes=<num_nodes> --nodelist=<nodes> --ntasks=2 --cpus-per-task=<num_of_cpu_cores> --qos=<node_qos_specification> --pty /bin/bash

Check :ref:`SLURM Job Submission Options <slurm-options>`.

Before starting an interactive session with ``srun``, it may be helpful to create a session on the login node with
a tool like ``tmux`` or ``screen``. This will prevent a user from losing interactive jobs in case of a network
outage or if the terminal is closed.

Running a Non-Interactive Job with ``sbatch``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When running jobs on a cluster, it's often more appropriate to submit non-interactive jobs using the ``sbatch``
command instead of ``srun``. Unlike ``srun``, which provides an interactive prompt,
``sbatch`` allows you to submit a job script or a command directly to the cluster's job scheduler.

By using ``sbatch``, you can benefit from the cluster's scheduling capabilities, job queuing, and automatic job management.
This is particularly useful for longer-running tasks. Additionally, running an ``sbatch`` job would avoid wasting resources
in case there is a bug in your code or your task is finished early.


Example using an .sh file:

1. Create a job script file, e.g., `job_script.sh`, with the following content:

.. code-block:: bash

    #!/bin/bash
    #SBATCH --partition=<partition>
    #SBATCH --nodes=<num_nodes>
    #SBATCH --nodelist=<nodes>
    #SBATCH --ntasks=2
    #SBATCH --cpus-per-task=<num_of_cpu_cores>
    #SBATCH --mem=<cpu_memory>GB
    #SBATCH --qos=<node_qos_specification>
    #SBATCH --time=<D-HH:MM:SS>
    #SBATCH --output=<out_file_name>.out
    #SBATCH --error=<error_file_name>.err

    #SBATCH --other-slurm-arguments

    # Add your commands here
    # spack ...
    # ...
    # python3 main.py

2. Submit the job to the cluster's job scheduler using the following command:

.. code-block:: bash

   <user>@kosmos:~$ sbatch job_script.sh

The above command submits the `job_script.sh` file to the cluster's job scheduler with the specified SLURM arguments,
such as partition, number of nodes, node list, number of tasks, CPU cores per task, and QoS specification.
You should replace `<partition>`, `<num_nodes>`, `<nodes>`, `<num_of_cpu_cores>`, and `<node_qos_specification>` with the
appropriate values for your job. For more information about SLURM job submission options and customizing your
job script, refer to the :ref:`SLURM Job Submission Options <slurm-options>` section.


Using `sbatch` with a job script file provides better flexibility and scalability for running batch jobs,
allowing your tasks to be scheduled and executed efficiently within the cluster's resources.

Other SLURM Commands
--------------------

Cluster State with ``sinfo``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To find information about the cluster and available resources, SSH to the SLURM login node for your cluster
(e.g., 'kosmos') and run the ``sinfo`` command:

.. code-block:: bash

   <user>@kosmos:~$ sinfo
   PARTITION      AVAIL  TIMELIMIT     NODES  STATE     NODELIST
   a6000             up   7-00:00:00        2    mix     aristarchus,galileo
   a6000             up   7-00:00:00        1  alloc     ptolemaeus
   rtx2080ti_sm*     up  14-00:00:00        1    mix     plato
   rtx2080ti_sm*     up  14-00:00:00        2   idle     carlos,schrodinger
   a100              up   7-00:00:00        2    mix     euctemon,eudoxus
   rtx2080ti         up   7-00:00:00        2    mix     alanturing,hamilton
   rtx8000           up   7-00:00:00        1    mix     roentgen
   p6000             up   7-00:00:00        1    mix     mariecurie
   cpu               up  30-00:00:00        1    mix     gaia
   gpu               down   7-00:00:00       1    mix     notus

The state of each node can vary between `mix`, `alloc`, `idle`, and `down`.

- `PARTITION`:
    The name of the partition.
- `AVAIL`:
    Availability of the partition.
- `TIMELIMIT`:
    The maximum time limit for jobs in the partition.
- `NODES`:
    The number of nodes in the partition.
- `STATE`:
    The state of the nodes in the partition.
- `NODELIST`:
    The list of nodes in the partition.

The state of each node can have the following meanings:

- `mix`:
    Nodes that are available for job allocation.
- `alloc`:
    Nodes that have been allocated to a job and are currently in use.
- `idle`:
    Nodes that are idle and available for job allocation but currently not in use.
- `down`:
    Nodes that are currently not operational or unavailable for job allocation.


The ``sinfo`` command provides an overview of the cluster state and availability.
For additional details and options, refer to the `sinfo documentation <https://slurm.schedmd.com/sinfo.html>`_.


Observing running jobs with ``squeue``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To see which jobs are running on the cluster, you can use the ``squeue`` command. It provides information about the jobs, such as their job ID, partition, name, user, state, time, time limit, number of nodes, and the node list.

Example usage:
By monitoring the running jobs with ``squeue``, you can track the progress and resource utilization of jobs in the cluster.
This information helps you manage and prioritize your work effectively.

For more details and available options, refer to the `squeue documentation <https://slurm.schedmd.com/squeue.html>`_.

.. code-block:: bash

    <user>@kosmos:~$ squeue -a -l
    Fri Jun 02 15:45:37 2023
    JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMIT  NODES NODELIST(REASON)
    41706      a100 lire3_fu n.moriak  RUNNING   16:50:37 3-08:00:00      1 euctemon
    40701     a6000    debug j.teuwen  RUNNING 5-20:40:41 7-00:00:00      1 galileo
    41495     a6000     bash  s.doyle  PENDING       0:00 1-16:00:00      1 (QOSMaxGRESPerUser)

The above command displays the currently running jobs on the cluster, including their job ID, partition, name, user,
state, running time, time limit, number of nodes, and the node list.

Additional ``squeue`` commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


*   To see only the running jobs for a particular user, you can use the following command:

    .. code-block:: bash

       <user>@kosmos:~$ squeue -l -u <username>


*   Sometimes, when the cluster experiences a high workload, your job may not start immediately and instead have to
    wait until one of the nodes becomes available. In order to see the estimated start time of your jobs, you can use the
    following command:

    .. code-block:: bash

        <user>@kosmos:~$ squeue -u USERNAME --start
        JOBID PARTITION     NAME     USER ST          START_TIME  NODES SCHEDNODES           NODELIST(REASON)

    This command will display the job ID, partition, name, user, state, start time, number of nodes, and the node list for the jobs.
    The START_TIME column indicates when the job is estimated to start. Please note that SLURM estimates the start
    time based on the time limits of jobs that are currently running. The time limit represents the maximum duration
    before a job is killed. In practice, many jobs finish earlier than their time limit, which means your job might
    start sooner than the initial estimated start time.

*   To monitor your running jobs and check for immediate errors after submitting, you can use the ``watch`` command along with ``squeue``.
    The following command will continuously display the running jobs, updating every 2 seconds:

    .. code-block:: bash

        <user>@kosmos:~$ watch squeue -u <username>


Additionally, you can use advanced options with `squeue` to further filter and customize the output.
Here are a few examples:

*   To see only the pending jobs (not yet running):

    .. code-block:: bash

       <user>@kosmos:~$ squeue -t PD

*   To display the jobs sorted by their priority:

    .. code-block:: bash

       <user>@kosmos:~$ squeue --sort -p

*   To show detailed information about a specific job using its job ID:

    .. code-block:: bash

       <user>@kosmos:~$ squeue -j <JOBID> -o "%.18i %.9P %.8j %.8u %.2t %.10M %.6D %.10L %.6R"

*   To display only the jobs from a specific partition:

    .. code-block:: bash

        <user>@kosmos:~$ squeue -p <partition_name>

By utilizing these advanced options, you can gain more insights into the job status and make informed decisions for job management on the cluster.

Please note that SLURM provides various other options and formatting possibilities with `squeue`.
For a comprehensive list and detailed documentation, refer to the `squeue documentation <https://slurm.schedmd.com/squeue.html>`_.


Cancel a job with ``scancel``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To cancel a job, use the ``squeue`` command to look up the JOBID and the ``scancel`` command to cancel it:

.. code-block:: bash

   $ squeue
   $ scancel JOBID

Finding job or node information with ``scontrol``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To see the status of a node or job and its resources run the ``scontrol`` command followed by either ``job <jobid>`` or ``node <nodename`` 


.. code-block:: bash

	$ scontrol show node ptolemaeus
	NodeName=ptolemaeus Arch=x86_64 CoresPerSocket=32
   		CPUAlloc=32 CPUTot=128 CPULoad=11.37
   		AvailableFeatures=(null)
   		ActiveFeatures=(null)
   		Gres=gpu:8(S:0-1)
   		NodeAddr=ptolemaeus NodeHostName=ptolemaeus Version=21.08.8
   		OS=Linux 5.4.0-137-generic #154-Ubuntu SMP Thu Jan 5 17:03:22 UTC 2023
   		RealMemory=980330 AllocMem=210304 FreeMem=298006 Sockets=2 Boards=1
   		State=MIXED ThreadsPerCore=2 TmpDisk=0 Weight=1 Owner=N/A MCS_label=N/A
   		Partitions=a6000
   		BootTime=2023-01-18T17:54:27 SlurmdStartTime=2023-01-18T17:55:08
   		LastBusyTime=2023-01-27T17:44:05
   		CfgTRES=cpu=128,mem=980330M,billing=128,gres/gpu=8
   		AllocTRES=cpu=32,mem=210304M,gres/gpu=3
   		CapWatts=n/a
   		CurrentWatts=0 AveWatts=0
   		ExtSensorsJoules=n/s ExtSensorsWatts=0 ExtSensorsTemp=n/s


This gives us for example the total resources (8 gpus), but also the allocated resources (3 gpus).

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

The ``nodestat`` command
------------------------

The `nodestat` command in our cluster is a utility that provides information about the cluster nodes,
including active jobs, job queues, and total resources.

Usage of ``nodestat``
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    nodestat [-h] [-j] [-m] [-q] [-t]

Options of ``nodestat``
^^^^^^^^^^^^^^^^^^^^^^^
- ``-h, --help``:
    Displays the help message and usage instructions for the ``nodestat`` command.

- ``-j, --jobs``:
    Shows the active jobs running on the nodes. This option provides information about the jobs currently utilizing the cluster resources.

- ``-m, --me``:
    Shows only the jobs belonging to the current user.
    By specifying this option, you can filter the displayed information to show only the jobs associated with your user account.

- ``-q, --queue``:
    Shows the jobs in the queue. This option provides information about the pending jobs waiting to be executed on the cluster nodes.

- ``-t, --total``:
    Shows the total resources available on the cluster. This option displays information about the overall resources,
    such as the total number of nodes, CPU cores, and memory available in the cluster.


Additional Resources
--------------------

* 
  `SchedMD Slurm Quickstart Guide <https://slurm.schedmd.com/quickstart.html>`_

* 
  `LLNL Slurm Quickstart Guide <https://hpc.llnl.gov/banks-jobs/running-jobs/slurm-quick-start-guide>`_
