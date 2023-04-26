==========
ZFS mounts
==========

User home folders and project folders are physically stored on ``rhea``\ . These are automatically mounted on-demand on the cluster nodes using ``autofs`` \.  The ``autofs`` mounting is configured through Ansible, by the playbook ``playbooks/slurm-cluster/autofs.yml`` in the  ``NKI-AI/kosmos-cluster`` repository. In principle, this playbook needs to be run just once for every machine for the initial configuration, and the ``autofs`` service should automatically restart after a reboot.

The home folders are currently automatically exported on ``rhea`` , and to be honest I don't remember whether I had to configure something specific to achieve this. The ``sharenfs=on`` flag for creating the filesystem (as performed with ``/usr/bin/create-home`` on ``rhea``\ ) likely takes care of this, so let's work on that assumption for the time being.

Project folders are still mounted using the old method, with a convoluted Ansible script. We'll change this during the next patch Monday, so that project folders are also mounted using ``autofs`` and we can get rid of the Ansible-managed block in the ``/etc/exports`` and ``/etc/fstab``.
