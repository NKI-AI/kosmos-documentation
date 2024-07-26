===============
Useful commands
===============

Switching to individual users
-----------------------------

Switch to user <username>:

.. code-block:: bash
   
   sudo -i -u <username>

Home folders
------------
On rhea there is the create-home function (which has a --help
function)

To change quotas use:

.. code-block:: bash

    zfs set quota=10G project-pool/network_homes/<username>

To look at quota use:

.. code-block:: bash

    zfs get (ref)quota project-pool/network_homes/<username>"

Updating slurm parameters
-------------------------
To update slurm parameters like which gpu belongs to which qos and
configure the default number of gpus change the slurm.conf on
atlas:/etc/slurm/slurm.conf and reconfigure slurm by running

.. code-block:: bash

   sudo scontrol reconfigure

Cluster management for the whole cluster
----------------------------------------
To do cluster management for the whole cluster use ansible on the teuwen-ansible VM. Here you have installed ansible and
you can run playbooks.

To change cluster wide variables look at the repo in ``kosmos-cluster/config/group_vars``

To run ansible playbook run the following command:

.. code-block:: bash

   ansible-playbook -kK -l gaia playbooks/slurm-cluster/autofs.yml

``-kK`` option specifies password (asks at the beginning, uses for everything),
-l option limits to only gaia, the path is the playbook to run.
