Troubleshooting
================
This page describes common problems and possible solutions.

Processing drives are full
---------------------------
When a node is down because processing drives are full, please run the following to
remove all files in processing that have not been touched the last 60 days.

.. code-block:: bash

    find "$DIR" -type f -atime +60 -delete
    find "$DIR" -type d -empty -delete

I/O error when submitting a batch job
--------------------------------------
When you get the following error message when submitting a batch job, it is likely
that the logfile storage is full.

.. code-block:: bash

    sbatch: error: Batch job submission failed: I/O error writing script/environment to file

You can fix this by using ``rsync`` to copy all logs to ``/mnt/archive/logs``
and subsequently deleting all logs on atlas except for today.

Node in drain
-------------
When a node is in drain, you can use the following commands to get it out of drain:

.. code-block:: bash

    sudo scontrol update NodeName=<nodename> state=RESUME