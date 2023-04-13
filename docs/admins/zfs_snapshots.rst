ZFS snapshots
=============

ZFS snapshots store the state of a filesystem at a given point in time. The snapshots are diff-based, so that snapshots of unchanged filesystems do not take up any disk space. The snapshots can be accessed in the virtual ``.zfs`` directory in the root of the filesystem.

We can recursively create a snapshot of all user home folders with the following command:

.. code-block:: bash

    zfs snapshot -r project-pool/network_homes@<label>

where the ``<label>`` is a name for the snapshot. We can use the current date in the format ``YYYY-MM-DD`` as the snapshot label. For now, I create these snapshots manually every morning, but we will want to automate this in the near future. We can then also implement a FIFO policy, so that daily snapshots are retained for one week, and monthly snapshots are retained for one year.

It seems that typical home folder usage does not involve high data throughput, and as a result the accumulated disk space usage of more than one week's worth of snapshots is still below 100GB. The total snapshot disk usage can be viewed using the following command:

.. code-block:: bash

    zfs list -p -t snapshot -o used | awk 'NR>1 {total+=$1} END {print total}' | numfmt --to=iec

A list of individual snapshot sizes can be found using the command

.. code-block::bash

   zfs list -t snapshot
