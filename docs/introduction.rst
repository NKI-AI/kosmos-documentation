
The AI for Oncology group has a compute cluster in the NKI data center named Kosmos.

/*<![CDATA[*/ div.rbtoc1670582129811 {padding: 0px;} div.rbtoc1670582129811 ul {list-style: disc;margin-left: 0px;} div.rbtoc1670582129811 li {margin-left: 0px;padding-left: 0px;} /*]]>*/


* `Resource sharing <#Computecluster@NKI(Kosmos>`_\ -Resourcesharing)
* `Hardware <#Computecluster@NKI(Kosmos>`_\ -Hardware)

  * `GPU nodes <#Computecluster@NKI(Kosmos>`_\ -GPUnodes)
  * `Storage nodes <#Computecluster@NKI(Kosmos>`_\ -Storagenodes)
  * `CPU nodes <#Computecluster@NKI(Kosmos>`_\ -CPUnodes)

* `Hardware or configuration errors <#Computecluster@NKI(Kosmos>`_\ -Hardwareorconfigurationerrors)
* `Installed software <#Computecluster@NKI(Kosmos>`_\ -Installedsoftware)

  * `Useful commands <#Computecluster@NKI(Kosmos>`_\ -Usefulcommands)
  * `Installed software <#Computecluster@NKI(Kosmos>`_\ -Installedsoftware.1)

Resource sharing
================

In a typical setting, multiple users will use the same server simultaneously. Therefore, consider the amount of CPU, GPU, and memory you use. For instance on ptolemaeus there are 128 cores (64 effective), 8 GPUs and 512GB of memory. So if you use 1 GPU, you can use up to 16 cores and 64GB of CPU memory to evenly use the available resources.

Kosmos uses scheduling set by slurm. We have a description on `how to use slurm. <Slurm-Usage-Guide_2385608707.html>`_

The `storage <Storage_1984299013.html>`_ page mentions the available storage options. On each machine there is a mounted path ``/mnt/archives`` linking to a file server. This path can be used to store archives which are considered to be *raw data,* for instance, .svs or dicom files. These folders can only be created by admins and are supposed to be read-only. Processed files, or temporary files for which you have a script that converts the raw archive to data used for your model can be stored in your own home directory.

Typically\ ``/mnt/archives`` is not fast enough to use to train a model. **This can lead to slowdowns for all users.** Therefore, each machine is equipped with a fast SSD storage in RAID-6, mounted in ``/processing``\ :


* 
  ``/processing``\  is the scratch disk, which is local to the machine

* 
  When you login \ ``/processing/<user name>``\   is created and has permissions 700 (no other regular users can read this) for your own user.

* 
  The folder is stored in the environment variable \ ``SCRATCH``\  , so no need to hardcode this! You can now rsync to \ ``$SCRATCH``\  .

* 
  The ``/processing`` disk will be wiped (oldest files first) if it is getting full!

Make sure to not constantly load large files during training from the network storage (this includes home). Not only will this be slow, but will also impact all other users.

Make sure to not store important training artefacts on the ``/processing`` drive as these can be deleted without warning. Use the project data folders for persistent storage.

The processing folders of the GPU nodes are mounted on eratosthenes in ``/mnt/processing/<hostname>`` so you can inspect the folders without needing to schedule a job one a compute node.

Transferring data before training can be done with ``rsync``. For instance, before training you could execute syncing your source with scratch.

.. code-block:: java

   rsync -avv --info=progress2 --delete <SOURCE> $SCRATCH

Hardware
========

Below lists the current available hardware.

GPU nodes
---------


.. raw:: html

   <table data-layout="full-width" data-local-id="f08e8f8b-1973-4097-afc9-012d7774e1d4" class="confluenceTable"><colgroup><col style="width: 125.0px;"><col style="width: 152.0px;"><col style="width: 248.0px;"><col style="width: 96.0px;"><col style="width: 91.0px;"><col style="width: 140.0px;"><col style="width: 118.0px;"><col style="width: 187.0px;"></colgroup><tbody><tr><th class="confluenceTh"><p><strong>Hostname</strong></p></th><th class="confluenceTh"><p><strong>GPUs</strong></p></th><th class="confluenceTh"><p><strong>CPUs</strong></p></th><th class="confluenceTh"><p><strong>Memory</strong></p></th><th class="confluenceTh"><p><strong>Scratch</strong></p></th><th class="confluenceTh"><p><strong>Network</strong></p></th><th class="confluenceTh"><p><strong>Installed</strong></p></th><th class="confluenceTh"><p><strong>Remarks</strong></p></th></tr><tr><td class="confluenceTd"><p>wallace</p></td><td class="confluenceTd"><p>8x Quadro RTX8000<sup>1</sup></p><p>(48GB)</p></td><td class="confluenceTd"><p>2x Intel Xeon Gold 6262V</p><p>(24 cores)</p></td><td class="confluenceTd"><p>384GB</p></td><td class="confluenceTd"><p>±13TB</p></td><td class="confluenceTd"><p>10 Gbps</p></td><td class="confluenceTd"><p>May 2020</p></td><td class="confluenceTd"><p>Not part of KOSMOS</p></td></tr><tr><td class="confluenceTd"><p>aristarchus</p></td><td class="confluenceTd"><p>8x Quadro A6000<sup>2</sup></p><p>(48GB)</p></td><td class="confluenceTd"><p>2 x AMD EPYC 7542 (2nd gen)</p><p>(32 cores)</p></td><td class="confluenceTd"><p>1TB</p></td><td class="confluenceTd"><p>±21TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>May 2021</p></td><td class="confluenceTd"><p></p></td></tr><tr><td class="confluenceTd"><p>ptolemaeus</p></td><td class="confluenceTd"><p>8x Quadro A6000<sup>2</sup></p><p>(48GB)</p></td><td class="confluenceTd"><p>2 x AMD EPYC 7542 (2nd gen)</p><p>(32 cores)</p></td><td class="confluenceTd"><p>1TB</p></td><td class="confluenceTd"><p>±21TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>May 2021</p></td><td class="confluenceTd"><p></p></td></tr><tr><td class="confluenceTd"><p>eudoxus</p></td><td class="confluenceTd"><p>8x A100<sup>2</sup></p><p>(80GB)</p></td><td class="confluenceTd"><p>2 x AMD EPYC 7543 SP3 (3rd gen)</p><p>(32 cores)</p></td><td class="confluenceTd"><p>1TB</p></td><td class="confluenceTd"><p>±21TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>April 2022</p></td><td class="confluenceTd"><p></p></td></tr><tr><td class="confluenceTd"><p>euctemon</p></td><td class="confluenceTd"><p>8x A100<sup>2</sup></p><p>(80GB)</p></td><td class="confluenceTd"><p>2 x AMD EPYC 7543 SP3 (3rd gen)</p><p>(32 cores)</p></td><td class="confluenceTd"><p>1TB</p></td><td class="confluenceTd"><p>±21TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>September 2022</p></td><td class="confluenceTd"><p></p></td></tr><tr><td class="confluenceTd"><p>plato</p></td><td class="confluenceTd"><p>2x RTX2080Ti (11GB)</p></td><td class="confluenceTd"><p>1x i9-7920X CPU @ 2.90GHz (12 cores)</p></td><td class="confluenceTd"><p>120GB</p></td><td class="confluenceTd"><p>±8TB</p></td><td class="confluenceTd"><p>1 Gbps</p></td><td class="confluenceTd"><p>August 2022</p></td><td class="confluenceTd"><p></p></td></tr><tr><td class="confluenceTd"><p>schrodinger</p></td><td class="confluenceTd"><p>2x RTX2080Ti (11GB)</p></td><td class="confluenceTd"><p>1x i9-7920X CPU @ 2.90GHz (12 cores)</p></td><td class="confluenceTd"><p>120GB</p></td><td class="confluenceTd"><p>±8TB</p></td><td class="confluenceTd"><p>1 Gbps</p></td><td class="confluenceTd"><p>August 2022</p></td><td class="confluenceTd"><p></p></td></tr></tbody></table>


*1 Requires CUDA >= 10.*

_2 Requires CUDA >= 11, needs capability sm_86 as the Quadro A6000s and A100s use the Ampere architecture._

*3 Requires root permissions, can be used through slurm with pyxis*

Storage nodes
-------------


.. raw:: html

   <table data-layout="full-width" data-local-id="c8bc5e38-4947-4382-9e47-1bf3b28091b1" class="confluenceTable"><colgroup><col style="width: 141.0px;"><col style="width: 169.0px;"><col style="width: 140.0px;"><col style="width: 140.0px;"><col style="width: 140.0px;"><col style="width: 148.0px;"><col style="width: 200.0px;"></colgroup><tbody><tr><th class="confluenceTh"><p><strong>Hostname</strong></p></th><th class="confluenceTh"><p><strong>Storage</strong></p></th><th class="confluenceTh"><p><strong>Network connection</strong></p></th><th class="confluenceTh"><p><strong>Specifications</strong></p></th><th class="confluenceTh"><p><strong>Software stack</strong></p></th><th class="confluenceTh"><p><strong>Backup</strong></p></th><th class="confluenceTh"><p><strong>Installed</strong></p></th></tr><tr><td class="confluenceTd"><p>storage01</p></td><td class="confluenceTd"><p>±261TB</p></td><td class="confluenceTd"><p>10 Gbps</p></td><td class="confluenceTd"><p>2x Xeon Silver 4208 - 8 core / 192 GB RAM</p></td><td class="confluenceTd"><p>FreeNAS</p></td><td class="confluenceTd"><p>No</p></td><td class="confluenceTd"><p>February 2021</p></td></tr><tr><td class="confluenceTd"><p>kronos</p></td><td class="confluenceTd"><p>±400TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>2x Xeon Silver 4208 - 8 core / 192 GB RAM</p></td><td class="confluenceTd"><p>TrueNAS</p></td><td class="confluenceTd"><p>Yes, for specific folders</p></td><td class="confluenceTd"><p>July 2022</p></td></tr><tr><td class="confluenceTd"><p>rhea</p></td><td class="confluenceTd"><p>±400TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>2x Xeon Silver 4208 - 8 core / 192 GB RAM</p></td><td class="confluenceTd"><p>TrueNAS</p></td><td class="confluenceTd"><p>Yes, for specific folders</p></td><td class="confluenceTd"><p>July 2022</p></td></tr></tbody></table>


CPU nodes
---------


.. raw:: html

   <table data-layout="full-width" data-local-id="cfa54099-1ce6-4883-ba29-6a79120be9ba" class="confluenceTable"><colgroup><col style="width: 147.0px;"><col style="width: 279.0px;"><col style="width: 108.0px;"><col style="width: 101.0px;"><col style="width: 97.0px;"><col style="width: 217.0px;"><col style="width: 139.0px;"><col style="width: 255.0px;"></colgroup><tbody><tr><th class="confluenceTh"><p><strong>Hostname</strong></p></th><th class="confluenceTh"><p><strong>CPUs</strong></p></th><th class="confluenceTh"><p><strong>Memory</strong></p></th><th class="confluenceTh"><p><strong>Scratch</strong></p></th><th class="confluenceTh"><p><strong>Network</strong></p></th><th class="confluenceTh"><p><strong>Software stack</strong></p></th><th class="confluenceTh"><p><strong>Installed</strong></p></th><th class="confluenceTh"><p><strong>Status</strong></p></th></tr><tr><td class="confluenceTd"><p>eratosthenes</p></td><td class="confluenceTd"><p>2 x AMD EPYC 7402 SP3 24-core 2.8GHz</p></td><td class="confluenceTd"><p>256GB</p></td><td class="confluenceTd"><p>±11TB</p></td><td class="confluenceTd"><p>40 Gbps</p></td><td class="confluenceTd"><p>Ubuntu 20.04</p><p>Docker<sup>3</sup> / Singularity / Enroot</p></td><td class="confluenceTd"><p>April 2022</p></td><td class="confluenceTd"><p>Main login node</p></td></tr></tbody></table>


The CPU nodes can be used for all kinds of tasks which do not require GPUs, such as preprocessing data, running tensorboard, etc.

Hardware or configuration errors
================================

If you encounter a problem which is likely due to configuration or hardware failure, you can check in the Slack channel ``#tech-hpc-cluster`` with others, or if you are sure immediately contact the admins: `rhpc-admin@nki.nl <mailto:rhpc-admin@nki.nl>`_.

Installed software
==================

We use `spack <https://spack.readthedocs.io/en/latest/>`_ for package management on RHPC. This is managed by Jonas Teuwen.

Useful commands
---------------


.. raw:: html

   <table data-layout="default" data-local-id="8002b28d-2f21-401b-b4e6-d59d1b32a127" class="confluenceTable"><colgroup><col style="width: 226.67px;"><col style="width: 226.67px;"><col style="width: 226.67px;"></colgroup><tbody><tr><th class="confluenceTh"><p><strong>Command w/ Spack</strong></p></th><th class="confluenceTh"><p><strong>Command w/ Module</strong></p></th><th class="confluenceTh"><p><strong>Description</strong></p></th></tr><tr><td class="confluenceTd"><p><code>spack find</code></p></td><td class="confluenceTd"><p><code>module avail</code></p></td><td class="confluenceTd"><p>Show available packages</p></td></tr><tr><td class="confluenceTd"><p><code>spack load &lt;package&gt;</code></p></td><td class="confluenceTd"><p><code>module load &lt;package&gt;</code></p></td><td class="confluenceTd"><p>Load the specified package</p></td></tr></tbody></table>


Installed software
------------------


.. raw:: html

   <table data-layout="default" data-local-id="d261ae1c-c0d0-4f24-8f7a-44781d9e0528" class="confluenceTable"><colgroup><col style="width: 226.67px;"><col style="width: 226.67px;"><col style="width: 226.67px;"></colgroup><tbody><tr><th class="confluenceTh"><p><strong>General name</strong></p></th><th class="confluenceTh"><p><strong>Specific installed name</strong></p></th><th class="confluenceTh"><p><strong>Description</strong></p></th></tr><tr><td class="confluenceTd"><p>pixman</p></td><td class="confluenceTd"><p><code>pixman@0.40.0</code></p></td><td class="confluenceTd"><p>Dependency for openslide. Previous versions are buggy</p></td></tr><tr><td class="confluenceTd"><p>cuda</p></td><td class="confluenceTd"><p><code>cuda@11.3.0</code></p></td><td class="confluenceTd"><p>GPU communication</p></td></tr><tr><td class="confluenceTd"><p>openslide</p></td><td class="confluenceTd"><p><code>openslide-aifo@3.4.1-nki</code></p></td><td class="confluenceTd"><p>Software to read whole-slide images.</p></td></tr></tbody></table>

