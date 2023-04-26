===================
Cluster maintenance
===================

Node reservations
-----------------

It is convenient if a node has no running jobs when you want to do maintenance on it. For this, we can use slurm reservations. These allow for scheduling jobs as usual, but with the restriction that all newly submitted jobs should have ended at a specific time. We create a reservation with the following command:

.. code-block:: bash
   
   sudo scontrol create reservation starttime=2023-01-01T00:00:00 duration=1-00:00:00 flags=maint,ignore_jobs nodes=ALL


This will create a reservation which starts at 2023-01-01T00:00:00 and lasts for 1 day. The flags indicate that the reservation is for maintenance, and that the reservation can be made even if there are jobs running at the time of making the reservation. The nodes=ALL indicates that all nodes should be reserved. If you want to reserve only a subset of the nodes, you can specify them by their names, e.g. nodes=node[1-10,20-30].
