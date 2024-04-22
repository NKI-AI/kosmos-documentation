==============
Administration
==============

Basic accounting
----------------

Slurm is all about associations. Below I will explain shortly how to create accounts, users
and QoS policies. Note that in slurm 'account' is not a user. (All commands below can only be run
with sudo)

Some basic useful commands to show associations and Qos policies:

.. code-block:: text

   sacctmgr show assoc
   sacctmgr show qos

Creating accounts
^^^^^^^^^^^^^^^^^

Suppose we want to create a new account, under which people are only allowed to use
2 gpus per user. Firstly we have to create the account

.. code-block:: text

   sacctmgr add account two_gpu Cluster=kosmos Description="Allows only 2 gpus"

Creating and modifying qos
^^^^^^^^^^^^^^^^^^^^^^^^^^

Creating a new quality of service:

.. code-block:: text

   sacctmgr add qos twogpu

Now we want to modify the limits within this qos, such that only 2 gpus can be used:

.. code-block:: text

   sacctmgr modify qos twogpu set maxtresperuser=gres/gpu=2

Modifying accounts
^^^^^^^^^^^^^^^^^^

Now let us set the qos of our new account to the new qos:

.. code-block:: text

   sudo sacctmgr modify account two_gpu set qos=twogpu

Adding users to new account
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Show which qos a user currently belongs to:

.. code-block:: text

   sacctmgr show assoc where user=<username> format=user,qos

Now we can add users to the new account. Even if the user already exists in an older
account, we create first a new account by

.. code-block:: text

   sacctmgr add user username account=two_gpu

If we want to make this two_gpu account the default one, where jobs are run from standardly, 
then we modify the user by

.. code-block:: text

   sacctmgr modify user where user=username set defaultaccount=two_gpu

Afterward it is good practice to delete the user from the older account, otherwise,
the user couls still append ``--Account=older_account`` when running jobs to circumvent limits.

.. code-block:: text

   sudo sacctmgr delete user where name=username account=older_account

The user can now only ask for max 2 gpus

Partition QoS
=============

If we want to limit access to a particular partition we can proceed as follows.
Add to the partition you want to limit in the slurm.conf: AllowQos=your_qos . Then only users which have 
access to this qos can submit to this partition.

Then the approach is as described above: make new account, make new qos, set the new qos to the new account, add users which you want to be able to submit on this
partition to the new assoc w/ its qos:

.. code-block:: text

   sudo sacctmgr modify user where name=username account=[radiology,aifo,mann] set qos=[rtx_qos,a6000_qos,four_a6000_qos,a100_qos,four_a100_qos] 

Group limits
============

We can create an account for which there are group limits, instead of user limits. 
Similar to above, we create a new account, and afterward we can add the group limits as:

.. code-block:: text

   sacctmgr modify account test_account set GrpTRES=gres/gpu=3
