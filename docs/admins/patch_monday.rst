===================
Patch Monday
===================

What to run to bring the cluster back online after patch monday?
-------------------------------------------------------------------

On atlas:

::

    sudo iptables -P INPUT ACCEPT
    sudo iptables -P FORWARD ACCEPT
    sudo iptables -P OUTPUT ACCEPT
    sudo iptables -t nat -F
    sudo iptables -t mangle -F
    sudo iptables -F
    sudo iptables -X

If there are any problems starting slurm, run:

::

    sudo systemctl restart slurmctld
    sudo scontrol reconfigure

If any node doesn't come back, restart slurm on that node:

::

    sudo scontrol reconfigure
    sudo systemctl restart slurmd


Delete reservation

::

    scontrol show reservation
    sudo scontrol delete reservation=<name>

Bring back drained nodes:

::

    sudo scontrol update nodename=<name> state=idle

