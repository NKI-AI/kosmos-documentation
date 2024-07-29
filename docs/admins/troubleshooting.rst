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


Homefolder is exceeding quota
------------------------------
Homefolder has 300G for homefolder and snapshots. It could be that
the snapshots are taking up too much space. You can see snapshots
on rhea by running the following command:

.. code-block:: bash

    zfs list -t snapshot project-pool/network_homes/<username>

You can prune snapshots by using this code:
:ref:'https://github.com/bahamas10/zfs-prune-snapshots/blob/master/zfs-prune-snapshots<https://github.com/bahamas10/zfs-prune-snapshots/blob/master/zfs-prune-snapshots>'

To delete snapshots starting with monthly, older tahn 6 months in
``project-pool/network_homes``, you can run the following command:
.. code-block:: bash

    zfs-prune-snapshots -p monthly 6M project-pool/network_homes

Use ``zfs send`` and ``zfs receive`` to move the snapshots
to another location. For example:

.. code-block:: bash

        zfs send project-pool/network_homes/${USER}@monthly-2024-02-01 > /mnt/to_tesla/${USER}

See also write_to_tape.sh on rhea. Use this command to rebuild
the filesystem:

.. code-block:: bash

    sudo zfs receive project-pool/projects/eric-zfs-receive-new < /project-pool/projects/eric-zfs-send/snap to rebuild the filesystem
